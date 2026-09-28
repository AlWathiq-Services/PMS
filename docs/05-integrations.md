# 05 — External Integrations and Adapter Contracts

**Pack:** MetriStay Hospitality Suite — Phase 1 planning pack v0.1 • **Date:** 2026-09-28 • **Owner of this document:** `integration_admin` (Integration Lead), with `compliance_officer` for regulated connectors
**Covers:** master prompt Section H item 6, Section E (gateway and bill-provider interfaces), M28/M29 F29.2, M38 F38.3, M44 F44.2, M45 F45.2–F45.3, M50 F50.2 and every module that crosses the platform boundary.
**Related:** `docs/03-architecture.md` (outbox/inbox, adapter boundary ADRs), `docs/07-security-regulatory.md` (threat model, PCI, five-market register), `docs/08-phase-backlog.md` (work packages and partner blockers), `docs/09-acceptance-and-migration.md` (G1–G20).

> **Honesty statement.** No partner contract, sandbox credential, certification or government API has been obtained for MetriStay as of 2026-09-28. Every integration below is a **specification**. Provider names appear only as *examples to validate*; none is selected. Status labels follow `docs/README.md` §3.6. Where a primary official source was actually opened on 2026-09-28 it is cited as `source-cited`; everything else is `unverified-assumption`. A connector reaches `partner-contracted`, `sandbox-tested` or `certified` only with filed evidence (contract ID, sandbox run log, certificate) in the connector registry (SF38.3.1).

---

## 1. Common adapter model (applies to every INT-*)

### 1.1 Port and adapter pattern
Each external dependency is reached only through a **port** (a versioned TypeScript interface owned by the consuming bounded context) implemented by one or more **adapters** (provider-specific, `mock`, `file`, `manual`). Adapters live in `integrations/<int-id>/<provider>`; domain code never imports a provider SDK. A property binds a port to exactly one active adapter per legal entity/purpose via the **connector registry** (entity `connector_binding`: `int_id, property_id, legal_entity_id, adapter, environment(sandbox|live), status_label, capability_set, credential_ref, contract_ref, activated_by, activated_at, expires_at`).

### 1.2 Capability flags
Every adapter publishes a static `capabilities()` manifest. The UI, API and workers read the same manifest; an operation whose flag is `false` is rendered **unavailable** with the manual path, never simulated in live mode (README §3.5).

Common flag vocabulary: `supports_sandbox`, `supports_webhook`, `webhook_signature` (`hmac_sha256|jws|mtls|none`), `supports_status_inquiry`, `supports_idempotency_key` (provider-side), `supports_void`, `supports_partial_refund`, `supports_reversal`, `supports_settlement_file`, `supports_batch`, `supports_cancel`, `supports_amend`, `max_rps`, `data_residency` (region code or `unknown`), `live_certified` (bool, requires evidence id).

### 1.3 Money-safe execution rules (non-negotiable)
1. **Every mutating external call carries our idempotency key**: `sha256(tenant_id|property_id|int_id|business_object_id|operation|attempt_group)`; stored in `external_request` before the call (transactional outbox).
2. **No blind retry for money, stock or external orders.** On timeout / 5xx / connection reset after the request may have left our network, the request moves to `unknown` → worker runs **status inquiry** (by our key or provider reference) → only a proven `not_found`/`failed` allows a new attempt with the *same* key where the provider honours keys, otherwise with a new attempt_group after human approval (`finance_approver`). If the provider has no inquiry capability, the item goes to the exception queue (SCR-OPS-exception-queue) and is settled only from provider statement/settlement evidence.
3. Non-money reads (availability, catalog, maps) may retry with exponential backoff + jitter (base 500 ms, max 5 attempts, cap 30 s) and circuit breaker (open after 50% failures over 20 calls in 60 s; half-open probe after 60 s).
4. **Inbound callbacks** land in `inbox` with dedup on `(int_id, provider_event_id)`; if the provider gives no event id, on `sha256(raw_body)`. Signature verified **before** parse; timestamp tolerance 300 s; unknown event types stored and ignored; out-of-order events reconciled by provider status inquiry, never by arrival order.
5. **A callback never directly marks a business object final** where money is concerned: it triggers a status fetch or a signature-verified terminal state, then the domain command applies with the inbox event as causation.

### 1.4 Standard failure states
`pending → submitted → (acknowledged) → confirmed | failed | unknown → reconciled | disputed | reversed`. `unknown` is a first-class state shown to users as "Waiting for provider confirmation — do not repeat".

### 1.5 Reconciliation baseline
Each money/stock/order integration has a reconciliation job producing `recon_run` and `recon_item(match_status: matched|amount_mismatch|missing_internal|missing_external|duplicate|timing)`. Unmatched items create owner-assigned exceptions with SLA; reports show `estimate` / `reconciled` / `certified` (README §3.6).

### 1.6 Credentials and transport
Secrets in Vault/KMS (`credential_ref` only in DB); per-property, per-environment credentials; rotation schedule recorded per connector; outbound egress allow-list per adapter; mTLS where the partner supports it; inbound webhooks on a dedicated ingress host with WAF, rate limit, and IP allow-list where the partner publishes ranges. See `docs/07` §15 key management.

### 1.7 Simulator/mock standard
Every port ships a deterministic **simulator adapter** (`adapter: mock`) usable in CI and demo tenants, driven by scenario scripts (`fixtures/integrations/<int-id>/*.yaml`) that can inject: success, decline, timeout-then-success, timeout-then-failure, duplicate callback, out-of-order callback, bad signature, partial settlement, provider outage, rate limit (429), schema drift. Mock adapters are **refused in `live` environment** by the connector registry (hard check), and every screen shows a `SIMULATED` badge when a mock is bound.

### 1.8 Data minimization baseline
Send only fields listed in each payload sketch; tokenise or pseudonymise where the partner permits; log payloads with field-level redaction (PAN, CVV never present; ID numbers, SIN, phone, email, plates masked); raw partner payloads retained per `docs/07` §5 retention class `integration-raw` (proposed 90 days, then hashed summary) unless they are accounting evidence.

### 1.9 Status labels used here
All entries are `unverified-assumption` unless a primary source retrieved on 2026-09-28 supports a factual statement, in which case the statement (not the integration) is `source-cited`. Source retrieval evidence: `docs/07` §13.

---

## 2. Integration register (summary)

| # | INT-id | Integration | Main modules | First phase | Status | Release dependency created |
|---|---|---|---|---|---|---|
| 1 | INT-PSP | Payment gateway / terminal / pay-by-link | M28, M08, M13, M54, M30 | 2 (abstraction), 5 (live) | unverified-assumption | One certified PSP per launch property (G-scenario 6); R1 blocker if property takes cards |
| 2 | INT-BANK | Bank statements, supplier/commission payouts | M20, M28 F28.2, M31 | 4 | unverified-assumption | Statement format + payout channel agreement per bank |
| 3 | INT-WPS-OM | Oman Wage Protection System salary file via bank | M27, M38 | 4 | source-cited (file-based model) | Bank WPS onboarding; file spec from bank/MoL |
| 4 | INT-GOV-CA | Canada CRA / Revenu Québec filing and exchange | M38, M44, M27 | 4–6 | source-cited (IFT/Web Forms exist) | Certified-software/IFT eligibility, transmitter number, counsel sign-off |
| 5 | INT-GOV-OM | Oman Tax Authority (VAT, Fawtara e-invoicing), guest reporting | M38, M44, M05 | 4–6 | unverified-assumption (Fawtara section exists) | Tax Authority onboarding/route confirmation |
| 6 | INT-GOV-PK | Pakistan FBR (federal) and provincial revenue authorities | M38, M44, M13 | 4–6 | source-cited (FBR POS integration rules exist) | Applicability to hotels; provincial authority route |
| 7 | INT-GOV-SA | Saudi ZATCA e-invoicing; MoI guest registration; tourism platform | M38, M44, M05 | 4–6 | source-cited (ZATCA developer portal referenced) | ZATCA onboarding/CSID; guest registration access |
| 8 | INT-GOV-PT | Portugal AT invoicing/SAF-T, guest-stay reporting, Social Security | M38, M44, M27, M05 | 4–6 | unverified-assumption | Certified invoicing software status; guest reporting access |
| 9 | INT-UTIL | Utility meters (AMR/CSV) and utility bill import | M22–M24, M67 | 4 | unverified-assumption | Meter hardware and utility bill formats per supplier |
| 10 | INT-BMS | Building management / fire / sensor alerts | M22, M42, M64 | 3–4 | unverified-assumption | Named BMS model + protocol gateway; life-safety isolation |
| 11 | INT-BILLPAY | Bill-provider gateway (Khedmah/ONEIC candidates) | M29, M22, M23 | 5 | unverified-assumption (no public merchant API found) | Partner contract + sandbox, else blocked with manual/bank path |
| 12 | INT-CHANNEL | Channel manager (ARI, reservations) | M07, M03, M04, M53 | 3 | unverified-assumption | Certified channel adapter for pilot hotel |
| 13 | INT-POS | POS: internal MetriStay POS vs external POS | M13, M08, M14, M57 | 3 | unverified-assumption | External POS interface certification if hotel keeps legacy POS; fiscal POS rules (PK, PT) |
| 14 | INT-LPR | AI camera / LPR server / gate controller | M17, M42 | 3 | unverified-assumption | Named camera/LPR/gate models, site pilot |
| 15 | INT-MSG | SMS and WhatsApp Business messaging (guest, OTP) | M41, M18, M52, M55 | 3 (SMS), 5 (WhatsApp) | unverified-assumption | Sender IDs/templates approved per country |
| 16 | INT-IDOCR | ID document OCR / authenticity / optional liveness | M41 | 2–3 | unverified-assumption | Provider or local model licence; lawful-basis gate per market |
| 17 | INT-ESIGN | Electronic signature / timestamp provider | M41, M10, M12, M31, M49 | 2–3 | unverified-assumption | Signature-level requirement per document/jurisdiction (counsel) |
| 18 | INT-MEDIA | Media storage, transcode, CDN, syndication | M39, M51 | 2–3 | unverified-assumption | CDN contract (SaaS); on-prem uses local storage |
| 19 | INT-SEARCH | Search/maps/metasearch listings and hotel ads feeds | M51, M53 | 3–5 | unverified-assumption | Partner program acceptance per metasearch |
| 20 | INT-ANALYTICS | Website analytics and consent | M51, M65 | 2 | unverified-assumption | Consent-mode configuration per market |
| 21 | INT-REVIEW | Review sources and review-response messaging | M52 | 3 | unverified-assumption | Approved API access per review source |
| 22 | INT-LOCK | Optional smart lock / digital key | M55, M05 | 7 (optional earlier pilot) | unverified-assumption | Lock vendor certification (Later) |
| 23 | INT-KIOSK | Optional self-service kiosk | M55, M41 | 7 (optional earlier pilot) | unverified-assumption | Kiosk hardware + PSP terminal certification (Later) |
| 24 | INT-AIR | Airline distribution / agency (NDC, GDS, consolidator) | M45, M46 | 5 | source-cited (NDC is a standard) | Licensed agency/consolidator agreement or referral-only mode |
| 25 | INT-CRUISE | Cruise operator / aggregator | M45, M46 | 5 | unverified-assumption | Signed supplier agreement |
| 26 | INT-TAXI | Taxi / transfer partner | M45, M59 | 3 (manual), 5 (API) | unverified-assumption | Partner agreement per city |
| 27 | INT-VENDORCHK | Vendor verification (registries, sanctions, bank-account) | M46, M20 | 3–4 | unverified-assumption | Data licence for sanctions list/registry access per market |
| 28 | INT-PUSH | Mobile push and vendor SMS/WhatsApp notification routing | M48, M47, M50, M18 | 3 | unverified-assumption | App-store developer accounts, push certificates |
| 29 | INT-SCAN | Barcode / QR / GS1 scanners and weighing scales | M50, M14, M25, M43 | 3 | unverified-assumption | Hardware pilot at receiving dock |
| 30 | INT-EDI | ASN / EDI / supplier API catalog sync | M48, M50, M21 | 4 | unverified-assumption | Supplier-by-supplier onboarding |
| 31 | INT-INVOCR | Invoice / packing-slip / utility bill OCR | M20, M22–M24, M50 | 4 | unverified-assumption | Model/provider licence; data residency |
| 32 | INT-EXTFIN | External accounting / finance system export | M19, M20, M27 | 4 | unverified-assumption | Target system selection by hotel (D-051) |
| 33 | INT-LLM | LLM / AI inference (guest AI, follow-up drafting, summaries) | M40, M50, M49, M37 | 3 | unverified-assumption | Model licence review; hosted vs local decision |
| 34 | INT-EMAIL | Transactional and campaign email | M18, M52, M50, M02 | 2 | unverified-assumption | Sending domain authentication (SPF/DKIM/DMARC) |

**Count: 34 integration contracts** (the four non-Canadian government exchanges are split into four INT-ids; bank and WPS, analytics and search, ID/OCR and e-sign, smart-lock and kiosk are split for separate gating).

Partner-owner roles used below: `integration_admin` (technical owner), `financial_controller` (money partners), `payroll_officer`, `compliance_officer`, `dpo`, `revenue_manager`, `marketing_manager`, `chief_engineer`, `security_officer`, `procurement_officer`, `concierge`/`front_office_manager`, `it_admin`. "Commercial owner" = the role that signs/maintains the partner agreement on the hotel side; MetriStay-side vendor management is owned by the MetriSys Partnerships role (to be named, D-052).

---

## 3. Money and payroll integrations

### 3.1 INT-PSP — Payment gateway, terminal and pay-by-link
| Field | Specification |
|---|---|
| Purpose & modules | Guest/corporate collections: deposits, pre-auth, capture at checkout, POS tenders, pay-by-link, ancillary/upsell and voucher sales, refunds, chargebacks. M28 F28.1, M08, M13, M15, M54, M30 (points never pass through PSP), M12 deposits. |
| Port `PaymentGatewayPort` | `createIntent(amount_minor, currency, purpose, folio_ref, customer_ref?)` • `authorize` • `capture(partial?)` • `void` • `refund(partial?)` • `createPaymentLink(expiry)` • `tokenizeViaHostedFields → token_ref` (consent-bound) • `chargeToken` • `getStatus(our_key | psp_ref)` • `fetchSettlement(date)` • `fetchDisputes(since)` • `submitDisputeEvidence` • `terminalPurchase(terminal_id)` (semi-integrated, card data stays in terminal). Flags: `supports_partial_capture`, `supports_incremental_auth`, `supports_card_on_file`, `supports_3ds`, `supports_terminal`, `supports_payment_link`, `supports_settlement_file`, `supports_dispute_api`, `currencies[]`. |
| Auth model | Server-side secret API key or OAuth client credentials in Vault, scoped per merchant account (per legal entity/property); publishable key only for hosted fields; terminal pairing by PSP-issued device credential. |
| Payload sketch | `{idempotency_key, amount_minor, currency, capture_mode, metadata:{tenant_id, property_id, folio_id|invoice_id|order_id, business_date}, return_url, customer:{token_ref?}}` → `{psp_payment_id, status, auth_code?, three_ds_status, card_brand, last4, expiry_mm_yy?, fee_minor?}`. **No PAN/CVV ever enters MetriStay** (hosted fields/redirect/terminal). |
| Idempotency | Our key sent as provider idempotency header where `supports_idempotency_key`; otherwise stored mapping and inquiry-before-retry. One `payment_attempt` per key; capture/refund each have their own key tied to parent. |
| Callbacks | Signed webhook (`hmac_sha256` or `jws` per provider) verified on raw body; event types mapped to `PaymentStatusChanged`; dedup on provider event id; terminal state confirmed by `getStatus` before folio posting. |
| Failure & timeout | Timeout → `unknown`, inquiry job at 30 s, 2 min, 10 min, 1 h; UI blocks second charge on same folio line while `unknown`. Decline → reason code mapped to guest-safe message. PSP outage → offline options: manual imprint is **not** offered; approved alternatives are cash, bank transfer, later pay-by-link, city-ledger per policy. |
| Reconciliation | Daily: PSP settlement file/API vs internal captures/refunds vs bank credit (INT-BANK), fees posted to GL fee account, chargebacks to dispute queue. Report columns *authorized, captured, settled, fee, net, banked*. |
| Data minimization | Store `psp_payment_id`, brand, last4, expiry month/year only if needed for guest recognition, token_ref; never full PAN, CVV, track data, PIN. Customer email/phone only if PSP requires for 3DS/receipt. |
| Licensing/certification | MetriStay does not hold funds. PSP must be licensed/authorised in the collecting market (e.g. CBO-licensed or bank-sponsored in Oman — see `docs/07` §7). PCI DSS v4.0.1 is the current standard listed by PCI SSC (retrieved 2026-09-28); intended SAQ scope assumption in `docs/07` §6. PSP-specific integration certification where required. |
| Simulator | `mock-psp`: test cards by scenario (approve, decline, 3DS challenge, timeout-then-captured, duplicate webhook, chargeback on day+10, partial settlement, fee variance). |
| Manual fallback | Cash shift, bank transfer with remittance reference, standalone terminal with manual folio entry by `cashier` + second-person verification and daily terminal batch report reconciliation. |
| Partner owner | Commercial: `financial_controller`; technical: `integration_admin`. |
| Examples to validate | International and regional acquirers/PSPs operating in each market (e.g. global PSPs with hosted fields and terminals; Omani bank acquirers; Saudi mada-capable acquirers; Pakistani bank acquirers; Canadian and Portuguese acquirers). None selected. |
| Status | `unverified-assumption` |
| Release dependency | **R1 blocker** for any property accepting cards: one PSP `sandbox-tested` then `certified` (G-scenario 6, AT-G06). Decision D-053 (PSP per launch market). |

### 3.2 INT-BANK — Bank statements, payment files and payouts
| Field | Specification |
|---|---|
| Purpose & modules | Import bank statements (M20 F20.3), export payment batches for supplier payables (F28.2), referral commission payouts (M31), refunds by transfer, corporate receipts allocation (F20.2.4). |
| Port `BankPort` | `importStatement(file|api, account_id, period)` • `exportPaymentBatch(batch_id, format)` • `submitPaymentBatch` (only if bank offers host-to-host/API) • `getBatchStatus` • `importRejections`. Flags: `statement_format` (`camt053|mt940|bai2|csv_bank_specific`), `payment_format` (`pain001|bank_csv|portal_manual`), `supports_h2h`, `supports_status_report` (`pain002`). |
| Auth model | Host-to-host: bank-issued certificates/SFTP keys or API OAuth; else **file download/upload by `payment_releaser` in the bank portal** (default). |
| Payload sketch | Batch header (debtor account, execution date, batch id, control sum) + lines `{end_to_end_id = payable_id, creditor name, IBAN/account, bank code, amount_minor, currency, remittance}`. |
| Idempotency | `end_to_end_id` unique per payable-payment; batch hash stored; resubmission of same batch blocked; bank rejection creates new attempt only after approval. |
| Callbacks | Usually none; status via statement or status report import. |
| Failure & timeout | Upload ambiguous → batch `unknown`; confirm via bank portal/statement before any resubmission. Rejected lines → payable back to `approved_unpaid` with reason. |
| Reconciliation | Statement lines matched by end_to_end_id, amount, date window; unmatched receipts to AR allocation queue; bank fees to GL. |
| Data minimization | Beneficiary bank details encrypted, masked in UI, change requires dual approval + out-of-band vendor confirmation (fraud control, M46 SF46.1.4). |
| Licensing | Payment initiation by the hotel through its own bank channel; MetriStay is not a payment institution. Any API payment initiation requires the bank's product agreement. |
| Simulator | `mock-bank`: generates camt.053/CSV statements, pain.002 rejections, duplicate-line and late-credit cases. |
| Manual fallback | Portal upload/download by `payment_releaser`, evidence file attached to batch. |
| Partner owner | `financial_controller`; `payment_releaser` operationally. |
| Examples to validate | Pilot hotel's operating banks; ISO 20022 support to be confirmed per bank. |
| Status | `unverified-assumption` |
| Release dependency | Statement format per bank before M20 reconciliation acceptance (AT-G08/AT-G20). |

### 3.3 INT-WPS-OM — Oman Wage Protection System salary file
| Field | Specification |
|---|---|
| Purpose & modules | Produce the standardized wage information file for Oman payroll and track bank processing (M27 SF27.3.6–SF27.3.7, M38). |
| Facts retrieved | Ministry of Labour pages (retrieved 2026-09-28) describe WPS as an electronic system shared between MoL and the Central Bank of Oman that monitors wage transfer to workers' bank accounts, requiring a **standardized wage file**; the FAQ refers to a CSV "Wage Information File" with naming conventions including commercial registration number, bank code, date and sequence. `source-cited`; exact column spec **not retrieved** and must come from the employer's bank/MoL. |
| Port `WageFilePort` | `generateWageFile(payroll_run_id, bank_id) → file + checksum` • `validateAgainstSpec(version)` • `recordSubmission(evidence)` • `importBankResponse(file)` • `recordRejection(line, reason)`. Flags: `spec_version`, `supports_bank_h2h` (default false), `supports_response_file`. |
| Auth model | File uploaded by `payroll_approver`/`payment_releaser` in the bank's corporate portal (default). H2H only under bank agreement. |
| Payload sketch | Per employee: employer CR number, employee ID type/number (civil ID/resident card), bank/IBAN, salary components as the spec requires, working days, deductions. Exact fields per bank-provided spec version. |
| Idempotency | One file per `payroll_run_id + bank_id + sequence`; regeneration increments sequence and voids the previous file only if not submitted. |
| Failure & timeout | Bank rejection lines → resubmission workflow (SF27.3.7) with maker-checker; salary not marked paid until bank confirmation/statement match. |
| Reconciliation | Payroll net pay vs bank debit lines vs WPS response; variances to `payroll_officer` queue. |
| Data minimization | File generated in memory, stored encrypted in payroll vault, access restricted to payroll roles; generic reports aggregate only (F27.4). |
| Licensing | None for software; employer obligation. Penalty facts per FAQ in `docs/07` §9. |
| Simulator | `mock-wps`: validates against a synthetic spec, emits accept/reject response files. |
| Manual fallback | Bank-prepared file from exported CSV; evidence upload. |
| Partner owner | `payroll_officer` (commercial with bank: `financial_controller`). |
| Status | `source-cited` (model), `unverified-assumption` (format and bank workflow). |
| Release dependency | Oman launch property: bank WPS spec + test file acceptance (AT-G05). |

### 3.4 INT-BILLPAY — Bill-provider gateway (Khedmah / ONEIC candidates)
| Field | Specification |
|---|---|
| Purpose & modules | Provider-neutral bill inquiry and payment for hotel operating utility bills (M29 F29.1/F29.2; M22, M23); guest bill-pay is **out of R1 scope** unless separately approved. |
| Facts retrieved | Khedmah and ONEIC Pay public sites (retrieved 2026-09-28) advertise electricity/water and other bill payments; **neither page mentions a merchant or developer API.** Consistent with master prompt: consumer sites do not establish API availability. |
| Port `BillProviderPort` | `listBillers()` • `validateAccount(biller_id, account_ref)` • `inquireBill → {bill_id, amount_minor, due, expiry, fee_minor}` • `quote` • `pay(order_id, idempotency_key)` • `getStatus(order_id)` • `reverse(order_id)` (flag) • `getReceipt` • `fetchSettlement(date)`. Flags: `supports_account_validation`, `supports_fee_quote`, `supports_reversal`, `supports_webhook`, `allowed_billers[]`. |
| Auth model | To be defined by partner contract (expected: OAuth2 client credentials or mTLS + request signing). Keys in Vault, rotation per SF29.2.3. |
| Payload sketch | `{order_id, idempotency_key, biller_id, account_ref, bill_id, amount_minor, currency:"OMR", payer_legal_entity, approval_ref}` → `{provider_txn_id, status, receipt_no?, settled_at?}`. |
| Idempotency | Mandatory; inquiry before retry; never second payment on timeout (Section L example). |
| Callbacks | Signed webhook or safe polling; confirmation required to mark paid. |
| Failure & timeout | `unknown` until inquiry/settlement; biller outage → bill remains payable via bank; dispute workflow SF29.1.9. |
| Reconciliation | Provider order ↔ gateway charge (if card-funded) ↔ bank debit ↔ AP payable ↔ GL; utility bill account/period matched. |
| Data minimization | Account refs and bill amounts only; no guest data. |
| Licensing | Partner holds payment licence; MetriStay only initiates approved payments under hotel's contract. Data-sharing agreement required. |
| Simulator | `mock-billpay` with timeouts, duplicate callbacks, biller outage, reversal-not-supported. |
| Manual fallback | Pay utility by bank transfer or provider counter; upload receipt; AP marks paid on evidence (G-scenario 6). |
| Partner owner | `financial_controller`; `integration_admin` technical. |
| Examples to validate | Khedmah, ONEIC Pay (Oman) — candidate partners, **not presumed accessible APIs**; equivalents in other markets only after jurisdiction review. |
| Status | `unverified-assumption` → set `blocked` for live automated payment until contract + sandbox (decision D-054). |
| Release dependency | Automatic bill payment is an **external blocker**; R1 passes with manual/bank path if documented (master prompt I). |

---

## 4. Government and regulatory exchange

Common port `GovFilingPort` (M38 F38.3): `listObligations(jurisdiction)` • `prepare(filing_type, period) → artifact` • `validate(schema_version)` • `submit(artifact, channel)` (only when channel=`api|file_transfer` and connector certified) • `getStatus` • `recordReceipt(evidence)` • `amend`. Channel enum: `api | file_transfer | portal_upload | web_form_manual | paper`. **Default channel for every obligation is `web_form_manual` or `portal_upload` performed by an authorised human, with the generated artifact and receipt stored.** No scraping, CAPTCHA bypass or robotic portal automation (SF38.3.8). Filing state machine: `draft → validated → approved(maker-checker) → submitted → acknowledged/receipt → accepted | rejected → amended`. A rule pack not in `verified` state blocks `approved` (M44 gate; `docs/07` §12).

### 4.1 INT-GOV-CA — Canada (CRA, Revenu Québec, provincial/municipal)
| Field | Specification |
|---|---|
| Purpose & modules | GST/HST returns; payroll remittances; T4 information returns; Québec RL-1 and QPP/QPIP where applicable; provincial accommodation tax/municipal lodging levy returns (M38 F38.1/F38.2, M27, M44). |
| Facts retrieved (2026-09-28) | (a) CRA "GST/HST Internet File Transfer" accepts a specific `.tax` file format created by **CRA-certified third-party software**; certification indicates compatibility, not endorsement. (b) CRA information-return pages list **Web Forms** and **Internet File Transfer (XML)** as electronic filing methods, with XML specifications/schemas, and allow authorised representatives. (c) CRA payroll pages describe CPP/EI/income-tax deduction tables and an online calculator. All `source-cited`; no general "government API" was found. |
| Port / capabilities | `GST_HST_RETURN`: channel `file_transfer` **only if** MetriStay becomes CRA-certified for GST/HST IFT (flag `ift_certified=false` now) else `web_form_manual` via My Business Account/NETFILE by an authorised person using the MetriStay worksheet. `T4`: channel `file_transfer` (XML generated to CRA schema, uploaded by the employer/representative via IFT) or `web_form_manual`. `RL-1` (Québec): route **unverified** (Revenu Québec page returned 403). Provincial/municipal levies: `portal_upload`/`manual` until each authority confirmed. Payroll remittance payment: through the employer's bank (INT-BANK), not an API. |
| Auth model | Human credentials of the employer/representative (CRA sign-in/business account; transmitter/web access code as CRA assigns) — never stored in MetriStay; XML generated and downloaded under `payroll_approver` step-up MFA. |
| Payload sketch | T4 XML: employer BN/payroll account, employee name, SIN (decrypted only at export in the payroll vault), box amounts per CRA spec version for the tax year. GST/HST worksheet: line values from the tax ledger with drill-down. |
| Idempotency | One filing artifact per `(obligation, period, version)`; amendments create new versions referencing the original; submission recorded once with receipt/confirmation number. |
| Callbacks | None assumed; acknowledgement recorded manually or from IFT confirmation. |
| Failure & timeout | Schema validation failure blocks approval; portal outage → filing calendar escalation to `compliance_officer`; late-filing risk shown with due dates from verified rule pack. |
| Reconciliation | Filed values ↔ GL tax liability ↔ remittance payments (bank) ↔ CRA statement of account (manual import). |
| Data minimization | SIN only in T4/RL artifacts and payroll vault; artifacts encrypted; download logged. |
| Licensing/cert | CRA software certification for GST/HST IFT (application process to be investigated, D-055). Payroll: decision whether to integrate a **Canadian payroll provider** instead of native calculation (master prompt J; D-056). |
| Simulator | `mock-cra`: schema validation against locally stored XSD version, synthetic acknowledgements, rejection codes. |
| Manual fallback | Default route (web form/portal by authorised person) with stored artifact and receipt. |
| Partner owner | `payroll_officer` (T4/RL), `financial_controller` (GST/HST, levies), `compliance_officer` (rule packs); external: Canadian tax counsel/accountant (named in D-057). |
| Status | `source-cited` (routes above), `unverified-assumption` (Québec, provincial, municipal routes). |
| Release dependency | Canadian launch property: verified rule pack + counsel sign-off + tested artifact (AT-G12); IFT certification optional (manual route acceptable if labelled). |

### 4.2 INT-GOV-OM — Oman (Tax Authority, VAT/Fawtara, guest reporting, municipality)
| Field | Specification |
|---|---|
| Purpose | VAT returns and tax invoices; e-invoicing (Fawtara) if/when in scope for the entity; withholding tax on qualifying payments to non-residents; guest registration/police reporting; municipal/tourism fees. |
| Facts retrieved | Tax Authority portal (tms.taxoman.gov.om) retrieved: VAT services and a dedicated **Fawtara e-invoicing** section exist. The Fawtara page content retrieved was a service-provider terms document referencing onboarding to an "Oman SMP" via an authorised service provider; the architecture (clearance vs exchange network) and timeline were **not** established. Withholding-tax page lists categories of payments to foreign persons without a PE (royalties, R&D, software, management fees, dividends, interest, services) — see `docs/07` register. |
| Channel | VAT return: `web_form_manual` (portal). Fawtara: `unknown` — to be determined from Tax Authority technical specifications; no API asserted. Guest reporting: `unknown` (ROP/tourism authority requirement to be confirmed). WHT: `web_form_manual`. |
| Auth / payload / idempotency | Human portal credentials; artifacts = VAT return worksheet, invoice XML only once spec obtained. Same filing state machine. |
| Simulator | `mock-om-tax` schema placeholder; guest-report CSV mock. |
| Manual fallback | Portal filing by authorised person with receipt upload. |
| Partner owner | `financial_controller`, `compliance_officer`; Omani tax adviser (D-057). |
| Status | `unverified-assumption` (routes), `source-cited` (existence of VAT/Fawtara sections). |
| Release dependency | Oman pilot: Fawtara applicability determination before invoice design freeze (D-058). |

### 4.3 INT-GOV-PK — Pakistan (FBR federal; PRA/SRB/KPRA/BRA provincial)
| Field | Specification |
|---|---|
| Purpose | Federal sales tax on goods and income-tax withholding (FBR); **provincial sales tax on services** (e.g. hotels/restaurants typically fall under provincial services tax regimes — assumption to be confirmed per province); POS/e-invoice integration where mandated. |
| Facts retrieved | FBR homepage references **POS Integration**, a new **Digital Invoicing System**, sales tax services and the IRIS portal. FBR POS legal-provisions page states integration is governed by Sales Tax Rules, 2006 Chapter XIV and refers to **Tier-1 retailers**, a per-invoice service charge (S.R.O. 1279(I)/2021) and amendments in 2025 SROs. Whether hotels/restaurants are within FBR POS scope was **not** established. PRA (Punjab) and SRB (Sindh) sites returned HTTP 503. |
| Channel | FBR POS/digital invoicing: API indicated by FBR materials but **technical spec not retrieved** → `unknown`; provincial services tax returns: `web_form_manual`; provincial POS/e-invoice integration: `unknown`. |
| Simulator | `mock-pk-fiscal`: returns synthetic invoice numbers/QR strings to exercise print layouts; never used in live. |
| Manual fallback | Portal filing; if a fiscal-POS mandate applies and is uncertified, **block POS go-live** for that outlet (fiscal requirement cannot be simulated). |
| Partner owner | `financial_controller`, `compliance_officer`; Pakistani tax adviser per province. |
| Status | `source-cited` (FBR POS rules exist), `unverified-assumption` (applicability, provincial routes). |
| Release dependency | Pakistan property: provincial authority identification and POS mandate determination (D-059). |

### 4.4 INT-GOV-SA — Saudi Arabia (ZATCA, MoI guest registration, tourism monitoring, GOSI)
| Field | Specification |
|---|---|
| Purpose | VAT tax invoices and e-invoicing (FATOORA) phases; guest registration with Ministry of Interior (Shomoos); tourism occupancy reporting; GOSI payroll contributions. |
| Facts retrieved | ZATCA e-invoicing page (retrieved): Phase 1 from 4 Dec 2021, Phase 2 from 1 Jan 2023 (rollout in waves); ZATCA publishes e-invoice specifications, XML implementation and security standards, and a list of qualified solution providers; ZATCA "Systems Developers" page links to an **E-Invoicing Developer Portal (sandbox.zatca.gov.sa)** which returned HTTP 403 to us. Shomoos information found **only in secondary vendor sources** (not primary). GOSI site timed out. |
| Channel | ZATCA Phase 2: `api` **indicated** by ZATCA's own developer portal reference; endpoints, onboarding (device/solution certificates) and wave applicability **unverified** → build behind `zatca_phase2_verified=false`. Shomoos: `unknown` (primary source not retrieved). GOSI: `web_form_manual`. |
| Auth | Expected certificate-based onboarding per ZATCA specs (to be verified from primary technical documents). |
| Idempotency | Invoice UUID + invoice counter/hash chain as ZATCA specs require (to verify); resubmission uses same UUID. |
| Simulator | `mock-zatca`: produces XML/QR placeholders and synthetic clearance/reporting responses; marked SIMULATED on printed invoices. |
| Manual fallback | If not integrated for a wave that applies: **block tax-invoice issuance in live mode** for that entity (cannot be manually substituted if e-invoicing is mandatory); if not yet in scope, standard invoice with Phase 1 requirements. |
| Partner owner | `financial_controller`, `compliance_officer`; Saudi tax adviser; possibly a ZATCA-qualified solution provider as partner (example to validate). |
| Status | `source-cited` (program and developer portal existence), `unverified-assumption` (details). |
| Release dependency | Saudi property cannot go live without verified e-invoicing path (D-060). |

### 4.5 INT-GOV-PT — Portugal (AT invoicing/SAF-T, guest-stay reporting, Social Security, tourism registry)
| Field | Specification |
|---|---|
| Purpose | Invoicing obligations (certified invoicing software, invoice communication, ATCUD/QR, SAF-T — **all assumptions, not retrieved**); foreign-guest stay reporting (historically SIBA); Social Security remuneration declarations; tourism registry (RNT/RNET). |
| Facts retrieved | Portal das Finanças home retrieved (Autoridade Tributária e Aduaneira) but AT invoicing page returned 404; SIBA returned 503; Segurança Social returned only a cookie page; Turismo de Portugal business portal retrieved and references RNT and SI-RJET registries. |
| Channel | All `unknown` until primary sources are read; invoicing-software certification is expected to be a **licensing precondition** for issuing invoices from MetriStay in Portugal (assumption, D-061). |
| Simulator | `mock-pt-at` SAF-T schema placeholder; guest-report mock. |
| Manual fallback | Use an already-certified invoicing system as system-of-record via INT-EXTFIN until MetriStay certification (option to validate). |
| Partner owner | `financial_controller`, `compliance_officer`, `dpo`; Portuguese counsel. |
| Status | `unverified-assumption` |
| Release dependency | Portugal property: invoicing certification route decision (D-061). |

---

## 5. Property operations and site devices

### 5.1 INT-UTIL — Utility meters and bill import
| Field | Specification |
|---|---|
| Purpose & modules | Interval/cumulative readings for electricity, water, gas (M22 F22.1, M23, M24, M67); utility invoice import (F22.2) feeding AP. |
| Port `MeterReadingPort` | `listMeters` • `pullReadings(meter_id, from, to)` • `pushReadings` (inbound CSV/API) • `getMeterHealth`. Flags: `protocol` (`modbus_tcp|bacnet|mbus|pulse|csv|vendor_api`), `interval_minutes`, `supports_push`, `cumulative|interval`. `UtilityBillPort`: `importBill(pdf|csv|edi)` → OCR via INT-INVOCR → structured bill. |
| Auth | Site gateway device identity (mTLS client cert from device registry M64); vendor API keys in Vault. |
| Payload sketch | `{meter_id, ts_utc, value, unit, quality: actual|estimated|reset|gap, source}`; bill: `{supplier, account_no, period_from/to, lines[], tax, due_date, total_minor}`. |
| Idempotency | Readings keyed `(meter_id, ts_utc)`; re-sent values upsert only if quality improves; bills keyed `(supplier, account_no, bill_no)` → duplicate detection (M20 SF20.1.3). |
| Failure | Gap detection, meter reset handling (SF22.1.4), stale-feed alert after 2× interval; bill parse failures → manual entry. |
| Reconciliation | Metered kWh/m³ vs billed quantity per period with tolerance; variance exception. |
| Data minimization | No personal data expected; room-level submeters treated as occupancy-inferring → aggregated for reporting. |
| Licensing | None; utility supplier data export terms. |
| Simulator | `mock-meter` generating curves with gaps/resets/spikes; sample bill PDFs. |
| Manual fallback | Manual meter reading on staff app with photo evidence; manual bill entry. |
| Partner owner | `chief_engineer` (meters), `ap_clerk` (bills). |
| Examples to validate | Site energy gateways supporting Modbus/BACnet; utility supplier e-bill formats per market. |
| Status | `unverified-assumption` |
| Release dependency | Meter model/protocol confirmation at pilot hotel (D-062); R1 acceptable with manual readings. |

### 5.2 INT-BMS — Building management, fire and sensor alerts
| Field | Specification |
|---|---|
| Purpose | Read-only ingest of BMS points and alarms (energy, leaks, HVAC faults) and approved fire/security alert **signals** for incident awareness (M42 F42.1, M22, M26 SF26.1.3, M64). |
| Port `BuildingSignalPort` | `subscribeAlarms` • `readPoints` • `ackAlarm` (flag, default false) • `health`. **No write/command capability** to life-safety systems in R1 (`supports_command=false`). |
| Auth | On-prem connector in segmented OT VLAN, outbound-only to MetriStay, device certificate. |
| Payload | `{source_system, point_id, location_ref, severity, alarm_type, ts, value, state: active|cleared}`. |
| Idempotency | Dedup on `(source_system, alarm_id, state, ts)`; incident dedup rules M42 SF42.1.3. |
| Failure | Connector heartbeat; loss → "BMS feed down" incident to `chief_engineer`/`security_officer`; life-safety panels remain independently compliant (SF42.1.5). |
| Reconciliation | Daily alarm count vs BMS log export sample. |
| Data minimization | No CCTV imagery ingested; footage pointer only. |
| Licensing | Integrator authorisation; fire-panel interface may require certified installer (market-specific). |
| Simulator | `mock-bms` scenario scripts (fire alarm, leak, false alarm, flapping). |
| Manual fallback | Staff/guest intake and phone escalation (playbooks). |
| Owner | `chief_engineer`, `security_officer`. |
| Status | `unverified-assumption` |
| Release dependency | Named BMS model at pilot site (D-062); AT-G14 simulated signal acceptable, real feed per hotel. |

### 5.3 INT-LPR — AI camera / LPR server / gate controller
| Field | Specification |
|---|---|
| Purpose | Plate observations, lane decisions and gate actuation for parking (M17), incident evidence pointers (M42). |
| Port `PlateObservationPort` (inbound) + `GateControlPort` (outbound) | Inbound: `{camera_id, lane_id, direction, plate_text, plate_country?, confidence, ts, image_ref?}`. Outbound: `openGate(lane_id, decision_id)`, `getGateState`. Flags: `edge_decision` (camera decides locally from allow-list), `supports_allowlist_push`, `supports_image_ref`, `gate_protocol` (`relay_io|vendor_api`). |
| Auth | On-prem; camera/LPR server → local MetriStay edge agent over mTLS; gate controller commands signed by edge agent. |
| Idempotency | Observation dedup by `(camera_id, ts, plate_hash)` window 10 s; gate command keyed by `decision_id` (one actuation). |
| Failure | Low confidence (< property threshold) → attendant review (SCR parking queue); network outage → edge allow-list cached with TTL; gate fail-safe mode per fire code (open on alarm) configured by site. |
| Reconciliation | Entry/exit session pairing; unmatched sessions nightly; parking charges post once to folio (AT-G03). |
| Data minimization | Plates hashed for matching where possible; images retained per `docs/07` retention class `lpr-image` (short); no face analytics. |
| Licensing | Camera analytics licences; signage/notice obligations per privacy law (`docs/07`). |
| Simulator | `mock-lpr` replaying plate streams incl. misreads, duplicates, tailgating. |
| Manual fallback | Attendant ticket/permit check and manual gate with logged override. |
| Owner | `security_officer`; `parking_attendant` ops. |
| Examples to validate | ANPR-capable IP cameras or LPR servers exposing HTTP event push; barrier controllers with dry-contact I/O. |
| Status | `unverified-assumption` |
| Release dependency | G-scenario 3 requires tested camera/LPR/gate at pilot site (hardware pilot WP in `docs/08`). |

### 5.4 INT-POS — Point of sale (internal vs external)
| Field | Specification |
|---|---|
| Decision | **Default: MetriStay internal POS (M13)** — no external integration; card tenders via INT-PSP terminals. **Option: external POS** retained by the hotel → integrate via `ExternalPOSPort`. |
| Port `ExternalPOSPort` | Inbound `postCheck {pos_check_id, outlet, items[{sku, qty, price, tax}], tenders[], room_or_account_ref, server_id, closed_at}`; outbound `lookupGuest(room, name) → masked match`, `validateChargeAuthority`, `menuSync`. Flags: `supports_room_charge_inquiry`, `supports_item_level`, `supports_void_sync`. |
| Auth | API key per outlet or on-prem interface (serial/TCP legacy protocols via edge agent). |
| Idempotency | `(pos_system, outlet, pos_check_id, revision)`; voids as reversing entries. |
| Failure | Offline queue both sides; room-charge authority cached per in-house list with expiry; post-offline reconciliation. |
| Reconciliation | Daily POS Z-report totals vs folio postings vs PSP terminal batch vs stock depletion (M14). |
| Fiscal note | Pakistan FBR POS-integration rules and any Portuguese certified-software/fiscal rules may apply to POS invoices — see INT-GOV-PK/PT; internal POS must then carry the fiscal integration itself. |
| Simulator | `mock-pos` check stream incl. duplicate/void/offline burst. |
| Manual fallback | Manual charge voucher with signature and later entry. |
| Owner | `fnb_manager`, `integration_admin`. |
| Status | `unverified-assumption` |
| Release dependency | Only if pilot keeps an external POS (D-063). |

### 5.5 INT-LOCK — Optional smart lock / digital key (Later)
Purpose: key encoding at check-in, mobile key, revocation at checkout/room move (M55, M05 linked device entitlement events). Port `KeyPort`: `issueKey(room, valid_from/to, guest_ref)`, `revokeKey`, `listEncoders`, flags `supports_mobile_key`, `supports_remote_revoke`. Auth: vendor cloud API or on-prem lock server via edge agent. Idempotency: key per `(stay_id, room, version)`. Failure: encoder offline → manual key encoding at lock server; revocation failure escalates to security. Reconciliation: active keys vs in-house stays nightly. Data: guest first name/room/dates only. Licensing: lock vendor partner program certification. Simulator: `mock-lock`. Manual fallback: native lock-vendor software. Owner: `front_office_manager`/`it_admin`. Examples to validate: major hotel lock vendors' integration programs. Status `unverified-assumption`. Release dependency: **Phase 7 (Later)**; not an R1 blocker unless pilot mandates (then separate certification WP).

### 5.6 INT-KIOSK — Optional self-service kiosk (Later)
Purpose: self check-in/out, ID scan (INT-IDOCR), payment on attached terminal (INT-PSP), key dispense (INT-LOCK). Port: kiosk is a **MetriStay client app** using public APIs with a device identity; peripheral drivers are local. Auth: device certificate + kiosk role with least privilege; no staff session on kiosk. Idempotency: same as guest web. Failure: any peripheral failure → hand-off to front desk. Data: no local storage of ID images; session wipe. Licensing: PCI PTS-approved terminal via PSP; accessibility (reach ranges) per market. Simulator: kiosk emulator. Owner: `front_office_manager`. Status `unverified-assumption`. Release dependency: Phase 7 (Later).

### 5.7 INT-SCAN — Barcode / QR / GS1 scanners and weighing scales
| Field | Specification |
|---|---|
| Purpose | Receiving, issue, stocktake, cylinder serials, lost-and-found tags (M50 F50.2/F50.3, M14, M25, M43). |
| Port `ScanPort` / `ScalePort` | Scans via mobile camera or HID/Bluetooth scanners in staff app; GS1-128/DataMatrix parsing (GTIN, lot, expiry, SSCC) — GS1 traceability standard referenced by master prompt (not re-retrieved). Scale: `readWeight(scale_id) → {gross, tare, net, unit, stable, ts, scale_serial}` via USB/serial/network edge agent. Flags `supports_gs1_ai`, `scale_legal_for_trade` (evidence id). |
| Auth | Device registry identity for fixed scales; user session for handhelds. |
| Idempotency | Scan event id generated on device (UUIDv7) + `(receipt_line, sscc/lot)` business key; duplicate scans rejected (SF50.2.7). |
| Failure | Offline capture queued with device timestamp; server acknowledgment required before stock posts. |
| Reconciliation | Scanned vs ASN vs PO; weight tolerance. |
| Licensing | Legal-for-trade scale certification where weight is the invoice basis (market metrology rules — `docs/07` register food/other). |
| Simulator | Barcode fixture set + `mock-scale` with unstable readings. |
| Manual fallback | Manual quantity entry with receiver attestation and photo. |
| Owner | `storekeeper`/`receiver`. |
| Status | `unverified-assumption` |
| Release dependency | Receiving-dock hardware pilot (AT-G18). |

---

## 6. Distribution, marketing and guest communications

### 6.1 INT-CHANNEL — Channel manager
| Field | Specification |
|---|---|
| Purpose | ARI (availability, rates, restrictions) push, reservation/modification/cancellation pull or push, OTA commissions, mapping (M07, M03, M04, M53 SF53.2.4). |
| Port `ChannelPort` | `pushARI(delta[])` • `receiveReservation` • `ackReservation` • `pullReservations(since)` (flag) • `getMapping` • `fullSync`. Flags: `ari_granularity` (room type × rate plan × date), `supports_los_pricing`, `supports_occupancy_pricing`, `supports_modify`, `supports_virtual_card` (VCC via PSP token, never raw), `max_update_rps`. |
| Auth | Channel-manager API credentials per hotel; webhook signing where supported. |
| Payload | ARI delta `{room_type_code, rate_plan_code, date, avail, rate_minor/currency, min_los, cta, ctd, stop_sell}`; reservation `{channel_res_id, channel, status, stay, guests, amounts, commission, payment_model}`. |
| Idempotency | ARI message sequence per `(property, room_type, rate_plan, date)`; reservations keyed `(channel, channel_res_id, revision)`. |
| Failure | ARI push failures retried (non-money) with dead-letter; oversell exposure monitor; reservation import failure → manual entry with channel reference; ack only after local commit. |
| Reconciliation | Daily reservation list vs channel extranet export; commission invoice match (M60 SF60.2.3). |
| Data minimization | Guest PII only as delivered; VCC handled by PSP tokenization. |
| Licensing | Channel-manager certification of MetriStay as a PMS connectivity partner. |
| Simulator | `mock-channel` with overbooking race, modification after cancellation, duplicate delivery. |
| Manual fallback | Close-out via extranet; manual reservation entry. |
| Owner | `revenue_manager`; `integration_admin`. |
| Examples to validate | Established channel managers' PMS partner programs; OpenTravel/HTNG message conventions. |
| Status | `unverified-assumption` |
| Release dependency | **R1**: certified channel adapter (Phase 3 exit) for pilot hotel (D-064). |

### 6.2 INT-SEARCH — Search, maps and metasearch
Purpose: property listings (business profile, map pin), hotel price/availability feeds and booking links for metasearch; campaign tags (M51 F51.2.3, M53). Port `ListingFeedPort`: `publishPropertyData`, `publishPriceFeed(itineraries)`, `receiveClick(attribution)`, flags `feed_type` (`xml_pull|push_api`), `supports_live_pricing_query`. Auth: partner-issued credentials/feed URL with token. Payload: property static data; price points `{checkin, los, room, total_price_incl_taxes, currency, url}` — **price must equal bookable total** (M51 accuracy). Idempotency: feed snapshots versioned. Failure: price-accuracy mismatch alerts → suspend feed. Reconciliation: click → booking attribution; partner cost invoices vs clicks/bookings. Data minimization: no guest data sent; server-side attribution ids only. Licensing: partner program acceptance/terms. Simulator: `mock-meta` crawler hitting price endpoint. Manual fallback: static listing without prices. Owner: `marketing_manager`. Examples to validate: major search/maps business-profile programs and hotel metasearch programs. Status `unverified-assumption`. Release dependency: Phase 3–5; not blocking core R1 but required for AT-G19 attribution (at least one channel or direct/search visit).

### 6.3 INT-ANALYTICS — Website analytics and consent
Purpose: funnel analytics, conversion attribution (M51 SF51.2.4–SF51.2.6, M65). Port `AnalyticsPort`: server-side event forwarding `{event, session_pseudonym, consent_state, page, value_minor}`; `ConsentPort` for CMP. Default: **first-party, server-side, consent-gated**; third-party tags only after consent per market (GDPR/ePrivacy for Portugal; other markets per register). Idempotency: event_id. Failure: drop analytics, never block booking. Data minimization: IP truncation, no PII, no ID/payment data. Simulator: local collector. Owner: `marketing_manager` + `dpo`. Status `unverified-assumption`. Release dependency: consent configuration verified per market before launch.

### 6.4 INT-REVIEW — Review sources and review messaging
Purpose: ingest public reviews via approved APIs, post approved responses, send post-stay review invitations (M52 F52.2). Port `ReviewPort`: `fetchReviews(since)`, `postResponse(review_id, text)` (flag), `sendInvitation` (via INT-EMAIL/INT-MSG). Auth: OAuth delegated by the property's account on each platform. Idempotency: `(source, review_id)`; response keyed by approval id. Failure: API disabled → manual copy with link. Data: reviewer display name only; no linkage to guest profile without lawful basis. Controls: no incentivised/fabricated reviews (SF52.2.6); response approval (`guest_relations` → `gm`). Simulator: `mock-reviews`. Owner: `guest_relations`. Examples to validate: platforms offering business review APIs. Status `unverified-assumption`. Release dependency: Phase 3; manual fallback acceptable.

### 6.5 INT-MSG — SMS and WhatsApp messaging (guest and OTP)
| Field | Specification |
|---|---|
| Purpose | OTP (M41 SF41.2.5), transactional guest messages, two-way guest chat (M55 inbox, M40 channel), marketing only with consent (M52). |
| Port `MessagingPort` | `sendTemplate(channel, to, template_id, locale, vars, purpose)` • `sendSession(text)` (WhatsApp 24h window — verify per provider) • `receive` (webhook) • `deliveryStatus`. Flags: `channels[sms,whatsapp]`, `supports_sender_id`, `supports_templates`, `supports_delivery_receipt`, `country_coverage[]`. |
| Auth | Provider API token in Vault; webhook signature verification. |
| Payload | `{message_id, to_e164, template, vars (no secrets except OTP), purpose: otp|transactional|marketing, consent_ref}`. |
| Idempotency | `message_id` per business intent; OTP resend creates new challenge, invalidates previous. |
| Failure | Delivery failure → fallback channel (SMS→email→in-person) per SF41.2.7; rate limits per recipient (anti-SMS-pumping: per-number, per-IP, per-country caps). |
| Reconciliation | Monthly usage invoice vs sent log. |
| Data minimization | Message bodies for OTP never logged in plain text; phone numbers masked in logs; OTP log retention per `docs/07` §5. |
| Licensing | Sender registration/template approval per country and platform rules; marketing consent laws per market. |
| Simulator | `mock-msg` in-app inbox + delivery failure injection. |
| Manual fallback | Front-desk verification in person; email. |
| Owner | `guest_relations` (content), `integration_admin`, `dpo`. |
| Examples to validate | CPaaS providers with coverage in all five markets; WhatsApp Business Platform via official or approved business solution providers. |
| Status | `unverified-assumption` |
| Release dependency | AT-G13 requires one bound SMS/WhatsApp verification in sandbox at minimum; live per-country sender approval. |

### 6.6 INT-EMAIL — Transactional and campaign email
Purpose: confirmations, invoices/receipts, password flows, vendor RFQ/PO notices, campaigns (M18, M52, M49, M50, M02). Port `EmailPort`: `send(template, to, locale, attachments_refs, purpose)`, `receiveInbound` (vendor replies parsed for M50 SF50.1.4), `bounceWebhook`. Auth: provider API key; SPF/DKIM/DMARC on hotel/MetriStay domains; on-prem SMTP relay option. Idempotency: message_id. Failure: queue with retry (non-money); bounces suppress. Data: invoices sent as secure links for sensitive docs. Licensing: anti-spam/consent laws per market. Simulator: local mail catcher. Manual fallback: print. Owner: `it_admin`. Examples to validate: transactional email services; hotel's own mail server. Status `unverified-assumption`. Release dependency: domain authentication before go-live.

### 6.7 INT-PUSH — Mobile push and vendor/staff notification routing
Purpose: vendor app PO/RFQ alerts (M48), emergency chef callout (M47 SF47.2.2), AI delivery reminders (M50), staff tasks, guest app. Port `NotificationPort`: `notify(user, purpose, priority, payload_ref)` → routes push (platform push services), SMS/WhatsApp (INT-MSG), email, voice (flag) by consented preferences and **approved channel per purpose**. Auth: platform push certificates/keys per app bundle. Idempotency: notification id; callout acceptance via authenticated app action with atomic reservation (SF47.2.3), never by SMS keyword alone for binding actions. Failure: escalation ladder with deadlines. Data: payload contains reference id only; content fetched after auth. Licensing: app-store developer accounts, platform policies. Simulator: `mock-push`. Owner: `integration_admin`. Status `unverified-assumption`. Release dependency: signed app distribution accounts (D-065).

### 6.8 INT-MEDIA — Media storage, transcode, CDN and syndication
Purpose: property photos/videos, derivatives, captions, CDN delivery, syndication to channels/metasearch (M39 F39.1, M51). Port `MediaDeliveryPort`: `store(original)` (immutable, S3-compatible), `transcode(profile)`, `publish(asset_version, targets[])`, `purge(asset_id)`, `syndicate(channel)`. Local AI enhancement runs on internal workers (M39 F39.2) — not an external integration unless a hosted provider is chosen. Auth: storage IAM roles; CDN API token. Idempotency: content hash. Failure: CDN failure → origin serving; syndication failures per channel queue. Data: EXIF GPS stripped; people-in-image consent/rights metadata. Licensing: rights/model releases; model/weight licence for enhancement. Simulator: local storage + no-op CDN. Manual fallback: direct upload to channel extranets. Owner: `content_approver`. Examples to validate: S3-compatible stores, CDNs, open-source transcoders (licence review). Status `unverified-assumption`. Release dependency: none blocking for on-prem; CDN contract for SaaS.

### 6.9 INT-IDOCR — ID document OCR, authenticity cues and optional liveness
| Field | Specification |
|---|---|
| Purpose | Assisted guest registration (M41 F41.1): MRZ/VIZ extraction, document expiry, authenticity cues; optional face match/liveness **only** behind lawful-basis gate. |
| Port `IdDocumentPort` | `extract(image_refs, doc_type_hint, country) → fields + confidence` • `authenticityChecks` (flag) • `faceMatch(selfie_ref, doc_ref)` (flag `biometric`, default **disabled**) • `deleteArtifacts(job_id)`. Flags: `processing_location` (`on_device|on_prem|vendor_cloud`), `retains_images` (must be false or bounded by contract), `supports_mrz`, `supports_nfc_chip`. |
| Auth | Vendor API key or local model; short-lived signed upload URLs. |
| Payload | Encrypted image refs (TTL ≤ 15 min), `{job_id, country, doc_type}` → `{fields{name, doc_no (masked in logs), dob, nationality, expiry}, confidence per field}`. |
| Idempotency | job_id; re-extraction allowed (read-only). |
| Failure | Low confidence → guest corrects field-by-field (SF41.1.5); vendor down → manual entry by staff viewing document. |
| Reconciliation | n/a (no money); audit counts for deletion proof. |
| Data minimization | Preference for **on-device or on-prem OCR**; no ID images in analytics/AI training (SF41.1.7); images deleted per retention class `id-image`; biometric templates never persisted unless jurisdiction pack allows and consent recorded. |
| Licensing | Model licence or vendor DPA; biometric processing permissions (e.g. Oman PDPL sensitive-data authorisation — `docs/07` §11). |
| Simulator | `mock-idocr` with specimen (synthetic) documents only; **no real ID images in test fixtures**. |
| Manual fallback | Staff visual check and typed entry; non-biometric path always available. |
| Owner | `dpo` + `front_office_manager`. |
| Examples to validate | On-device MRZ libraries (licence review), commercial IDV vendors with regional processing. |
| Status | `unverified-assumption` |
| Release dependency | AT-G13 OCR in sandbox; biometric features **not** in R1 unless market gate passed. |

### 6.10 INT-ESIGN — Electronic signature and trusted timestamp
Purpose: guest registration card, corporate contracts/BEO sign-off, referrer agreements, vendor award acknowledgment (M41 F41.2, M10, M12, M31, M49 SF49.3.5). Port `SignaturePort`: `createEnvelope(doc_hash, signers, level: simple|advanced|qualified)`, `captureInApp(signature_evidence)` (native simple e-signature), `getEvidence`, `timestamp(doc_hash)` (RFC 3161 TSA flag). Default R1: **native simple electronic signature** with evidence bundle (see `docs/07` §17); external provider only where a document/jurisdiction requires advanced/qualified level (e.g. eIDAS levels in Portugal — counsel question). Auth: provider OAuth. Idempotency: envelope per `(document_version_hash)`. Failure: provider down → wet signature scan path. Data: minimal signer identity. Licensing: qualified trust service provider for qualified signatures (EU). Simulator: `mock-esign`. Owner: `dpo`/`compliance_officer`. Status `unverified-assumption`. Release dependency: counsel determination of required signature level per document per market (D-066).

---

## 7. Travel concierge partners (M45, M46, M59)

Common rule: order/payment controls **hidden until authorized** (SF45.3.6); itinerary shows `booked` only with confirmed external reference (G-scenario 11). Hotel operating mode per market: `referral_only | agent_of_licensed_seller | licensed_seller` (M45 F45.3; `docs/07` register tourism rows). Port `TravelProviderPort` with capability flags per provider: `search`, `quote`, `hold`, `book`, `issue`, `status`, `change`, `cancel`, `refund`, `webhook`.

### 7.1 INT-AIR — Airline distribution / agency (NDC, GDS, consolidator)
| Field | Specification |
|---|---|
| Purpose | Flight quotes and, only where authorised, bookings/ticketing for guests/corporate travellers. |
| Facts retrieved | IATA describes NDC as an XML-based data transmission standard for offer/order management — a **standard, not inventory access** (retrieved 2026-09-28, `source-cited`). Ticketing requires accreditation or an accredited partner (IATA accreditation page not retrieved). |
| Port | `searchOffers`, `priceOffer(expiry)`, `createOrder` (flag), `issueTicket` (flag, false unless licensed seller/partner issues), `getOrder`, `cancel/refund` (flags). |
| Auth | Aggregator/consolidator API credentials under the hotel's or licensed agency's contract. |
| Payload | Passenger data minimized to what the airline order requires (names as on passport, DOB if required, contact); consent reference. |
| Idempotency | `order_request_id`; duplicate callback handling; price-change reconfirmation. |
| Failure | Offer expiry → requote; timeout on createOrder → inquiry (no duplicate booking); schedule change events to concierge queue. |
| Reconciliation | Supplier invoice/ADM vs orders; commission/service fee; guest folio charge vs supplier payable. |
| Licensing | Travel-agency licence in market (e.g. Oman Ministry of Heritage and Tourism travel & tourism agency licence exists — gov.om page retrieved) or operate referral-only. |
| Simulator | `mock-air` with expiry, fare change, cancellation, refund. |
| Manual fallback | Auditable manual RFQ to a contracted agency by email/portal with evidence (SF45.2.3). |
| Owner | `concierge`/`front_office_manager`; commercial `gm`. |
| Examples to validate | Self-service flight APIs, consolidators, GDS agency programs, airline NDC programs (Amadeus developer guide cited by master prompt as example only). |
| Status | `source-cited` (NDC nature), `unverified-assumption` (access). |
| Release dependency | Phase 5 exchange only with contract; R1 passes with referral/manual RFQ path labelled (D-067). |

### 7.2 INT-CRUISE — Cruise operator / aggregator
Purpose: cruise itinerary/cabin quotes, holds and bookings, shore transfer coordination. Port as common, flags mostly `false` until signed supplier agreement (SF45.3.4). Auth per agreement. Payload: occupants, cabin category, dates, consent. Idempotency: request id. Failure: hold expiry, inquiry-before-retry. Reconciliation: supplier statements/commission. Licensing: agency/package-travel obligations per market (Portugal package travel rules — counsel). Simulator `mock-cruise`. Manual fallback: email RFQ to operator/agency with evidence. Owner `concierge`. Examples to validate: cruise line agency portals, cruise aggregators. Status `unverified-assumption`. Release dependency: Phase 5 contract; manual path for R1.

### 7.3 INT-TAXI — Taxi / airport transfer partner
Purpose: book guest rides/airport transfers, flight-linked ETA, status (M45 SF45.1.5, M59). Port flags: `quote`, `book`, `track`, `cancel`, `driver_details`, `webhook`. Auth: partner API token; guest phone shared only with consent and only for trip. Payload `{pickup, dropoff, time, pax, luggage, accessibility, guest_contact(masked relay if supported), payer}`. Idempotency: trip request id; duplicate callbacks deduped. Failure: no driver assigned by T-minus threshold → escalate to hotel fleet/alternate partner. Reconciliation: trip receipts vs folio charges vs partner invoice. Licensing: driver/vehicle licensing is the provider's (SF45.3.5) verified via M46 documents. Simulator `mock-taxi`. Manual fallback: phone booking with confirmation reference logged. Owner `concierge`. Examples to validate: ride-hailing guest-ride APIs (master prompt cites one as example), local taxi fleets. Status `unverified-assumption`. Release dependency: Phase 3 manual case required; API Phase 5 per city.

---

## 8. Procurement, vendor and finance integrations

### 8.1 INT-VENDORCHK — Vendor verification (registries, sanctions, bank account)
Purpose: onboarding checks for M46 SF46.1.3–SF46.1.4: company registry lookup, tax registration validation, sanctions/PEP screening, bank-account ownership confirmation. Port `VendorVerificationPort`: `lookupCompany(country, reg_no)`, `validateTaxId`, `screenSanctions(entity, persons[])`, `verifyBankAccount` (flag). Auth: data-provider licence keys. Payload: legal name, reg no, country, beneficial owner names (minimal). Idempotency: check id; results versioned with list version date. Failure: provider unavailable → manual document review with maker-checker; screening "possible match" → compliance review, never auto-reject. Reconciliation: periodic re-screen on list updates. Data minimization: store result + evidence ref, not full dossiers. Licensing: sanctions list licences; official public lists (UN/regional/national) as free source options to validate. Simulator `mock-vendorchk` (hit/near-hit/clear). Manual fallback: upload of certificates, manual check of official registry websites by `procurement_officer` with screenshot evidence. Owner `procurement_officer` + `compliance_officer`. Status `unverified-assumption`. Release dependency: Phase 3 manual path; automated screening licence optional.

### 8.2 INT-EDI — ASN / EDI / supplier catalog API sync
Purpose: supplier PO transmission, acknowledgments, ASNs with lot/expiry/SSCC, catalog/stock/price sync (M48 SF48.3.5, M50 SF50.1.2, M21). Port `SupplierExchangePort`: `sendPO`, `receivePOAck`, `receiveASN`, `receiveInvoice` (→ AP), `syncCatalog`, `syncStock`. Formats: MetriStay JSON API (primary), CSV upload (vendor app), optional EDI (e.g. EDIFACT/X12/GS1 XML) via translator — examples to validate per supplier. Auth: vendor API keys scoped to own records; mTLS for EDI VAN/AS2 if used. Idempotency: `(vendor, doc_type, doc_no, revision)`. Failure: ASN mismatch → receiving exception; stale stock → badge (SF48.3.1). Reconciliation: PO ↔ ASN ↔ GRN ↔ invoice three-way. Data: vendor sees only own POs. Simulator `mock-supplier`. Manual fallback: vendor app/web entry; email PDF. Owner `procurement_officer`. Status `unverified-assumption`. Release dependency: per-supplier onboarding; not blocking R1 (vendor app is primary).

### 8.3 INT-INVOCR — Invoice, packing-slip and utility bill OCR
Purpose: extract supplier invoices, delivery notes, utility bills (M20 SF20.1.2, M22 SF22.2.1, M50 SF50.2.2). Port `DocumentExtractionPort`: `extract(doc_ref, doc_type) → fields + confidence + bbox`. Default **local model** option (licence review) with hosted provider as alternative. Idempotency: doc hash. Failure: low confidence → human verification; never auto-approve payables on OCR alone (SF49.3.7). Data: supplier docs may contain personal data of sole traders — processed under vendor DPA. Simulator: synthetic invoice set incl. duplicates and tax errors. Manual fallback: manual keying. Owner `ap_clerk`. Status `unverified-assumption`. Release dependency: none blocking; accuracy benchmark on pilot documents before straight-through use.

### 8.4 INT-EXTFIN — External accounting / finance system
Purpose: for hotels keeping an external GL/ERP/payroll, export journals, AP/AR subledgers, payroll journals; import chart of accounts and payment status (M19 SF19.2.5, M20, M27). Port `FinanceExportPort`: `exportJournals(period, format)`, `exportSubledger`, `importCoA`, `importPaymentStatus` (flag). Formats: CSV/XLSX, target-system API (examples to validate: common SME/enterprise accounting packages; Canadian CRA-certified GST/HST software list includes several), SAF-T-like exports per market. Auth: OAuth per target. Idempotency: journal batch id + hash; re-export flagged. Failure: partial import → batch reconciliation report. Reconciliation: trial balance tie-out between systems. Data: salary journals aggregated by department (F27.4). Simulator: file drop. Manual fallback: file export/import. Owner `financial_controller`. Status `unverified-assumption`. Release dependency: target system decision (D-051).

### 8.5 INT-LLM — LLM / AI inference
| Field | Specification |
|---|---|
| Purpose | Guest AI assistant answers and bounded tool calls (M40), vendor follow-up drafting and reply parsing (M50 SF50.1.3–SF50.1.4), bid evidence summaries (M49 SF49.2.6), staff summaries. |
| Port `InferencePort` | `complete(messages, tools_allowed[], max_tokens, data_class)` • `embed(texts)` • `moderate(text)`. Flags: `hosting` (`local_open_weight|vendor_api`), `data_retention` (vendor zero-retention or not), `region`, `supports_tool_calls`, `context_limit`. |
| Auth | Vendor API key in Vault or local inference endpoint on internal network. |
| Payload | Redacted prompts (PII masking SF40.2.5); tool calls executed by MetriStay tool gateway with the **caller's** scopes, never the model's. |
| Idempotency | Not money-moving; tool actions that would create orders/bookings produce **drafts only** requiring human/guest confirmation (SF40.2.2). |
| Failure | Timeout/outage → "assistant unavailable, connecting you to staff" (SF40.2.7); cost cap per property/day. |
| Reconciliation | Token/cost metering vs invoice; quality sampling. |
| Data minimization | No ID images, payment data, SIN/payroll into prompts; transcripts per retention class `ai-transcript`. |
| Licensing | Model/weight licence review for local models; vendor DPA and data-transfer assessment per market (`docs/07` §14). |
| Simulator | `mock-llm` deterministic responses incl. prompt-injection test corpus. |
| Manual fallback | Human staff inbox. |
| Owner | `integration_admin` + `dpo`; content owner `guest_relations`. |
| Examples to validate | Hosted frontier-model APIs; permissively licensed open-weight models for on-prem. |
| Status | `unverified-assumption` |
| Release dependency | Model choice and licence review (D-068); AI governance tests (`docs/07` §14). |

---

## 9. Decisions and risks raised by this document

| ID | Decision / risk | Owner | Default assumption until resolved |
|---|---|---|---|
| D-051 | Target external accounting/payroll system at pilot hotel | `financial_controller` | MetriStay is GL of record; CSV export available |
| D-052 | Name MetriSys Partnerships owner for partner contracts | Product Owner | Integration Lead acts |
| D-053 | PSP per launch market/legal entity | `financial_controller` | Hosted-fields + terminal PSP; mock in CI |
| D-054 | Khedmah/ONEIC commercial access | `financial_controller` | `blocked`; manual/bank path |
| D-055 | Apply for CRA GST/HST IFT software certification? | `compliance_officer` | No; manual web filing with worksheet |
| D-056 | Native Canadian payroll vs Canadian payroll provider | `payroll_officer` | Provider integration evaluated first; native engine behind gate |
| D-057 | Name tax counsel/adviser per market | `compliance_officer` | Rule packs stay `draft` |
| D-058 | Oman Fawtara applicability and technical route | `financial_controller` | Unknown; invoice design keeps structured data |
| D-059 | Pakistan province + POS/e-invoice mandate for hotel/restaurant | `compliance_officer` | Unknown; block fiscal POS go-live |
| D-060 | Saudi ZATCA phase/wave for entity; qualified provider vs native | `financial_controller` | Native behind gate; partner option |
| D-061 | Portugal certified invoicing software route | `financial_controller` | External certified system of record |
| D-062 | Meter/BMS models and protocols at pilot | `chief_engineer` | CSV/manual readings |
| D-063 | Internal vs external POS at pilot | `fnb_manager` | Internal POS |
| D-064 | Channel manager selection | `revenue_manager` | Mock + one certification target |
| D-065 | App-store developer accounts and signing ownership | `it_admin` | Enterprise/internal distribution for pilots |
| D-066 | Required e-signature level per document/market | `compliance_officer` | Simple e-signature with evidence bundle |
| D-067 | Travel operating mode per market | `gm` + counsel | `referral_only` |
| D-068 | LLM hosting model (local vs vendor) | `integration_admin` + `dpo` | Vendor API with zero retention for SaaS; local for on-prem |
| R-051 | A partner exposes no inquiry API → double-payment risk | `financial_controller` | Exception queue + settlement-only confirmation |
| R-052 | Government portal changes break manual worksheets | `compliance_officer` | Rule-pack change alerts (SF44.2.7) |
| R-053 | Fiscal/e-invoicing mandate discovered late blocks go-live | `compliance_officer` | Register rows in `docs/07` §12 resolved before Phase 5 |

*(D-/R- numbers 051–068 are reserved for this document; `docs/13` decision log consolidates them.)*
