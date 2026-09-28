# 07 — Security, Privacy and Regulatory Register

**Pack:** MetriStay Hospitality Suite — Phase 1 planning pack v0.1 • **Date:** 2026-09-28 • **Document owner:** `compliance_officer` (regulatory), `dpo` (privacy), Security Lead (to be named, D-071)
**Covers:** master prompt Section H item 8, Section A "Regulatory and partner facts to verify", Section E (wallet, referral legal gate), M02, M28, M30, M31, M38, M40, M41, M42, M44, M64, Section P.4.
**Related:** `docs/03-architecture.md` (access scoping, RLS, ledgers), `docs/05-integrations.md` (per-connector auth/failure), `docs/08-phase-backlog.md` (gates), `docs/09-acceptance-and-migration.md` (security/restore tests).

> **Legal status of this document.** This is an engineering and compliance *planning* register, not legal advice and not a clearance. No statement here has been reviewed by counsel (`counsel-reviewed` count = 0). Facts are marked `source-cited` only where a primary official source was actually retrieved on **2026-09-28**; statements drawn from secondary sources are marked as such; everything else is `unverified-assumption` or a counsel question. **No tax rate or threshold is stated as fact.** Where a retrieved primary page itself showed a figure, it is quoted with "(page states; to be confirmed by counsel)".

---

## 1. Security objectives and scope

1. No cross-tenant or cross-property data exposure (SaaS and on-prem).
2. No unauthorized movement of money, stock, inventory or external orders; every such action is idempotent, authorized, audited and reversible only by compensating entries.
3. Sensitive personal data (ID images, biometrics, SIN/payroll, payment tokens, CCTV/LPR, signatures) is collected by purpose, minimized, encrypted and deleted on schedule, except where accounting/legal evidence or legal hold requires retention.
4. Regulated services (card payments, wallets, bill payment, payroll clearing, government filing, ticketing) stay with licensed partners; MetriStay never implies a licence or certification it lacks.
5. AI components cannot take consequential actions autonomously, cannot award, and cannot assert delivery/booking facts without system-of-record evidence.

---

## 2. Trust boundaries

| TB | Boundary | Crossing actors / data |
|---|---|---|
| TB1 | Public internet → guest website, guest app, booking API | `guest`, `booker`, bots; search, quotes, PII, payment redirects, OTP |
| TB2 | Internet → corporate portal/apps | `corporate_*` users; negotiated rates, rooming lists, invoices |
| TB3 | Internet → vendor app/portal | `vendor_admin`, `vendor_user`; catalogs, bids, POs, ASNs, invoices, bank details |
| TB4 | Staff devices (mobile, offline) → staff API | all staff roles; offline queue, room/guest data, stock movements |
| TB5 | Privileged admin plane | `tenant_admin`, `property_admin`, `it_admin`, `integration_admin`, `payroll_*`, `payment_releaser`; config, credentials, rule packs |
| TB6 | Platform ↔ external partners (outbound calls, inbound webhooks) | PSP, bank, channel, messaging, bill-pay, travel, LLM, OCR (`docs/05`) |
| TB7 | Hotel site OT network ↔ edge agent ↔ platform | LPR cameras, gates, BMS, scales, scanners, POS terminals, locks |
| TB8 | Platform ↔ AI inference and tool gateway | guest/vendor text (untrusted), retrieved knowledge, tool calls |
| TB9 | Tenant ↔ tenant (SaaS shared infrastructure) | row-level `tenant_id`/`property_id`, shared workers, caches, object storage |
| TB10 | Production ↔ operations (backups, logs, support access, CI/CD) | backups, telemetry, break-glass |
| TB11 | Platform ↔ government portals/files (human-mediated by default) | tax, payroll, guest-registration artifacts |

---

## 3. STRIDE threat model per trust boundary

Legend: **S**poofing, **T**ampering, **R**epudiation, **I**nformation disclosure, **D**enial of service, **E**levation of privilege. Each mitigation maps to a test in §16.

### TB1 — Guest web/app and booking API
| STRIDE | Threat | Mitigation |
|---|---|---|
| S | Account takeover; OTP interception; QR link forwarded to attacker | OIDC via Keycloak, MFA optional for guests / step-up for payments & profile export; OTP bound to session+device challenge, 5-min expiry, single use, attempt limit (SF41.2.6); QR = handoff only, requires independent authentication |
| T | Price/quote tampering in client; manipulated hidden fields | Server-side quote snapshot with signed quote id and expiry (M04); totals recomputed at hold and payment |
| R | Guest disputes booking/consent | Consent ledger (purpose, channel, version, timestamp, IP hash); e-sign evidence bundle (§17) |
| I | Enumeration of reservations by confirmation number; PII in URLs | Lookup requires confirmation no. + surname + OTP; no PII in query strings; generic errors |
| D | Search scraping, inventory-hold exhaustion, SMS pumping | Rate limits per IP/device/account; hold TTL and per-session hold caps; bot mitigation without inaccessible CAPTCHA (accessible alternative per Section P.1); OTP country/number velocity caps |
| E | Guest accessing staff/corporate endpoints | Separate audiences/clients per app; function-level authorization deny-by-default |

### TB2 — Corporate portal/apps
| STRIDE | Threat | Mitigation |
|---|---|---|
| S | Shared corporate logins; SSO misconfiguration | Named users, SSO (SAML/OIDC) with domain binding, MFA for approvers |
| T | Editing negotiated rate or approval state | Rates read-only to corporate; approvals are signed state transitions with maker-checker |
| R | Corporate approver denies approval | Approval audit with step-up MFA evidence |
| I | Company A seeing Company B's rates/attendees | Object-level checks on `corporate_account_id`; tenant-isolation tests (§4.2) |
| D | Bulk rooming-list uploads | File size/row caps, async processing |
| E | Booker self-granting approver role | Role changes only by `corporate_admin` + hotel `sales_manager` confirmation |

### TB3 — Vendor app/portal
| STRIDE | Threat | Mitigation |
|---|---|---|
| S | Fake vendor registration; impersonation of an approved vendor | M46 verification (INT-VENDORCHK or manual), maker-checker approval, MFA for vendor admins |
| T | Changing bank details to divert payment; editing bid after close | Bank change = dual approval + out-of-band callback; sealed bids with server-side close timestamp and version lock (SF49.2.3) |
| R | Vendor denies PO acknowledgment or delivery claim | Signed acknowledgment evidence; ASN/receipt linked to scan/scale evidence |
| I | Vendor sees competitor bids or guest data | Vendor scope = own records only (SF48.3.7, SF46.3.6); minimum guest data per job |
| D | Catalog flooding, image upload abuse | Quotas, malware scan, async processing |
| E | Vendor user promoting self to admin; accessing other properties | Vendor roles scoped to vendor + approved properties; function-level authz |

### TB4 — Staff devices and offline sync
| STRIDE | Threat | Mitigation |
|---|---|---|
| S | Lost/stolen device used by another person | Device registration, short session, biometric/PIN unlock of app (device-local, not stored by us), remote wipe/revoke |
| T | Replayed or edited offline queue | Offline commands signed with device key, monotonic sequence, server validates against current state; conflicts routed to resolution UI |
| R | Staff denies void/comp/stock adjustment | Per-user audit, reason codes, approvals |
| I | Cached guest data on device | Encrypted local store, minimal cache scope by role/shift, TTL purge |
| D | Sync storm after outage | Backpressure, batch limits |
| E | Housekeeper viewing payroll/ID data | Field-level permissions; sensitive fields never synced to non-authorized roles |

### TB5 — Privileged admin plane
| STRIDE | Threat | Mitigation |
|---|---|---|
| S | Phished admin | Phishing-resistant MFA (WebAuthn) for privileged roles; IP/device posture policy for SaaS admin |
| T | Unauthorized rule-pack or tax change; feature flag toggling referral payout | Rule-pack change requires reviewer + evidence; jurisdiction gates enforced server-side; config change audit with diff |
| R | Admin denies configuration change | Immutable admin audit log shipped to append-only store |
| I | Credential exposure in UI/logs | Credentials write-only (Vault refs), never displayed |
| D | Mass disable of connectors | Change approval for connector deactivation in live |
| E | Self-granting `payment_releaser` | Segregation of duties matrix; role grants require second admin; privileged access time-bound (JIT) |

### TB6 — External partners and webhooks
| STRIDE | Threat | Mitigation |
|---|---|---|
| S | Forged webhook (fake "payment succeeded") | Signature verification on raw body, timestamp tolerance, provider status confirmation before money state change (`docs/05` §1.3) |
| T | Tampered payload in transit | TLS 1.2+ (1.3 preferred), mTLS where supported, payload signatures |
| R | Partner denies receiving an instruction | Stored request/response hashes, idempotency keys, provider references |
| I | Oversharing PII with partners | Per-connector data minimization (`docs/05` payload sketches), DPAs |
| D | Partner outage cascades | Circuit breakers, queues, manual fallback per connector |
| E | Compromised partner credentials used to act beyond scope | Least-privilege partner keys, per-property credentials, rotation, egress allow-lists |
| — | Unsafe consumption of partner data (API10) | Schema validation, size limits, no deserialization of untrusted types, SSRF-safe fetchers |

### TB7 — Site OT (LPR, gates, BMS, scales, POS terminals, locks)
| STRIDE | Threat | Mitigation |
|---|---|---|
| S | Spoofed camera event opening gate; plate cloning | Edge agent mTLS; camera on isolated VLAN; low-confidence/duplicate-plate anomalies to attendant; permit + plate + time-window rules |
| T | Manipulated scale readings | Scale serial/calibration record, stable-reading requirement, tolerance checks vs ASN |
| R | Gate override without trace | Override requires attendant auth + reason; logged |
| I | CCTV/LPR images exposed | Image refs only, short retention, access logged |
| D | Network loss at gate | Cached allow-list with TTL, manual fallback, fail-safe per fire code |
| E | Pivot from OT network to platform | Outbound-only edge agent, no inbound ports, segmentation (SF64.1.2) |
| — | Life-safety interference | **No write/command path to fire/life-safety systems** (INT-BMS read-only) |

### TB8 — AI inference and tool gateway
| STRIDE | Threat | Mitigation |
|---|---|---|
| S | Model output impersonating staff; guest claiming to be staff via prompt | Assistant identity disclosure; role derived from auth, never from conversation text |
| T | Prompt injection via guest chat, vendor replies, reviews, uploaded docs, retrieved KB | Treat all content as data; tool gateway enforces caller scopes; allow-listed tools per surface; output validation; see §14 |
| R | Disputed AI answer | Transcript with cited sources and KB version |
| I | Model leaking other guests' data | Retrieval filtered by caller's authorization before prompt construction; PII masking; no cross-session memory |
| D | Cost exhaustion | Per-property/day token caps, rate limits |
| E | Model invoking payment/award/booking | Consequential tools not exposed; drafts only; human confirmation (SF40.2.2, SF49.2.6, SF50.1.7) |

### TB9 — Tenant isolation (SaaS)
| STRIDE | Threat | Mitigation |
|---|---|---|
| I/E | Missing `tenant_id` filter; cache key collisions; object storage path traversal; background job running with wrong tenant | Postgres RLS on every tenant table (`SET app.tenant_id` per transaction, RLS `FORCE`), tenant-prefixed cache keys, per-tenant storage prefixes with signed URLs, job envelopes carry tenant and re-establish RLS context; tests §4.2 |
| D | Noisy neighbour | Per-tenant quotas, queue fairness |
| T | Cross-tenant event delivery | Outbox events carry tenant; inbox consumer asserts tenant match |

### TB10 — Operations (backups, logs, support, CI/CD)
| STRIDE | Threat | Mitigation |
|---|---|---|
| S | Supply-chain compromise | Signed commits/builds, SBOM, dependency pinning and scanning, provenance attestations |
| T | Backup tampering | Encrypted, immutable (object-lock) offsite backups; restore tests (SF64.2.1) |
| R | Support engineer access denial | Break-glass access with ticket id, time-bound, recorded |
| I | PII in logs/telemetry | Structured logging with redaction; sensitive-field allow-list |
| D | Ransomware | Immutable backups, isolated restore environment, RPO/RTO targets in `docs/09` |
| E | CI secrets abuse | OIDC-based short-lived deploy credentials, environment protection rules |

### TB11 — Government artifacts
| STRIDE | Threat | Mitigation |
|---|---|---|
| S | Unauthorized filing on behalf of the employer | Filings performed by authorized human with own government credentials; MetriStay never stores them |
| T | Artifact altered after approval | Artifact hash recorded at approval; submission compares hash |
| R | "We filed" without proof | Receipt/confirmation number mandatory to reach `acknowledged` |
| I | SIN/ID in exported files | Encrypted artifacts, download logging, auto-expiry of download links |
| E | Filing from unverified rule pack | Rule-pack `verified` state required (M44 gate) |

---

## 4. OWASP API Security Top 10 (2023) mapping and tenant-isolation tests

Category list retrieved 2026-09-28 from api-security.owasp.org (`source-cited`).

### 4.1 Mapping
| OWASP 2023 | MetriStay exposure | Controls | Test ids |
|---|---|---|---|
| API1:2023 Broken Object Level Authorization | Reservations, folios, invoices, POs, bids, payslips, ID images addressed by id | UUIDv7 ids (not a control on its own); every repository call takes `(tenant_id, property_id, principal)`; policy engine check per object; RLS as defence-in-depth | SEC-BOLA-01..20 |
| API2:2023 Broken Authentication | Guest OTP, vendor login, device tokens, partner webhooks | Keycloak, MFA/WebAuthn for privileged, OTP binding, token lifetime ≤ 15 min access / rotating refresh, webhook signatures | SEC-AUTH-01..12 |
| API3:2023 Broken Object Property Level Authorization | Mass assignment (e.g. `status`, `rate`, `approved_by`), over-exposed fields (SIN, salary, bank) | Explicit DTO allow-lists for input and output per role; field-level masking; response schemas in OpenAPI tested | SEC-BOPLA-01..10 |
| API4:2023 Unrestricted Resource Consumption | Availability search, OTP send, OCR, LLM, media upload, report export | Rate limits, quotas, pagination caps, upload limits, cost caps | SEC-RES-01..08 |
| API5:2023 Broken Function Level Authorization | Admin/finance functions reachable by staff/vendor tokens | Deny-by-default route guards, role/permission matrix generated from catalogue, separate admin API audience | SEC-BFLA-01..10 |
| API6:2023 Unrestricted Access to Sensitive Business Flows | Inventory holds, promo/voucher abuse, referral self-referral, points farming, bid sniping | Hold caps, velocity rules, fraud checks (SF30.2.3, M31 step 3), sealed bids | SEC-FLOW-01..10 |
| API7:2023 Server Side Request Forgery | Media URL import, webhook URL config, vendor catalog URL sync, OCR fetch | Egress proxy with allow-list, block link-local/metadata IPs, no user-supplied URL fetch without validation | SEC-SSRF-01..05 |
| API8:2023 Security Misconfiguration | On-prem installs, CORS, default creds, verbose errors | Hardened images, config baseline checks, CIS benchmarks, secure headers, on-prem installer security checklist | SEC-CFG-01..08 |
| API9:2023 Improper Inventory Management | Deprecated API versions, sandbox endpoints exposed in prod | OpenAPI registry, versioning/deprecation (M33), environment separation, mock adapters refused in live | SEC-INV-01..05 |
| API10:2023 Unsafe Consumption of APIs | Partner responses (PSP, channel, OCR, LLM) trusted blindly | Schema validation, signature checks, status confirmation, output encoding, LLM output treated as untrusted | SEC-UCA-01..08 |

### 4.2 Tenant- and property-isolation test suite (automated, every build)
| Test | Assertion |
|---|---|
| ISO-01 | For every REST endpoint in OpenAPI, a token for tenant A requesting tenant B object ids returns 404 (not 403, to avoid existence leaks) |
| ISO-02 | Same as ISO-01 across properties within one tenant for property-scoped roles |
| ISO-03 | Direct SQL as app role without `app.tenant_id` set returns zero rows on every RLS table (RLS `FORCE` verified by catalog query) |
| ISO-04 | Background jobs: enqueue job for tenant A with payload referencing tenant B id → job fails closed |
| ISO-05 | Outbox/inbox: event with mismatched tenant rejected by consumer |
| ISO-06 | Cache: identical keys for two tenants never collide (tenant prefix enforced by wrapper; lint rule) |
| ISO-07 | Object storage: signed URL for tenant A object cannot be reused for another key; path traversal blocked |
| ISO-08 | Search index: queries always include tenant/property filter; fuzz test with missing filter fails build |
| ISO-09 | Reports/exports: row-, column- and field-level filters applied (salary never in generic reports, F27.4) |
| ISO-10 | Vendor tokens can only read own bids/POs across all procurement endpoints |
| ISO-11 | Corporate tokens can only read own account's bookings/invoices/rates |
| ISO-12 | AI tool gateway: tool call with object id outside caller scope rejected |
| ISO-13 | Webhook for connector bound to property P cannot mutate objects in property Q |
| ISO-14 | Tenant deletion/export job touches only that tenant (row counts verified) |

---

## 5. Privacy by purpose and retention schedule

### 5.1 Principles
- Every personal-data field is tagged with `data_class` and one or more `purpose` codes; processing is allowed only for listed purposes (M02 consent by purpose/channel).
- Retention periods below are **proposed policy defaults for design and testing, not legal retention periods**. Each jurisdiction rule pack (M44) overrides them after counsel review; until then the rule-pack row is `unverified-assumption` and the stricter of (a) shortest period compatible with the purpose and (b) any legal hold applies.
- Deletion is implemented by **crypto-shredding** (per-record or per-subject data keys) for blobs and field-level encryption, plus hard delete/anonymization for rows, with a deletion receipt (`deletion_event`: class, object ids hash, rule version, executed_at, executor).

### 5.2 Retention matrix
| Data class | Purpose(s) | Lawful-basis category (to confirm per market) | Proposed default retention (policy, not law) | Start event | Deletion method | Legal hold? | Accounting / legal-evidence carve-out | Owner |
|---|---|---|---|---|---|---|---|---|
| Guest profile (name, contacts, preferences, stay history) | Reservation performance, service, CRM (consented) | Contract; consent for marketing | 3 years after last stay for profile; marketing data until consent withdrawn + suppression record | Last checkout / consent withdrawal | Anonymize profile; keep suppression hash | Yes | Folio/invoice copies keep name/billing fields per accounting retention (separate class) | `dpo` |
| Guest registration record (fields required by guest-registration/police rules) | Legal obligation where applicable | Legal obligation (market-specific) | Per jurisdiction rule — **unknown in all five markets until counsel review** | Checkout | Hard delete at rule expiry | Yes | n/a | `compliance_officer` |
| ID document images | Identity verification at check-in, OCR autofill | Legal obligation or consent (market-specific) | Delete within 24 h after verification/check-in completion; never used for analytics/AI training (SF41.1.7); if a market requires a copy, retain only per that rule | Verification complete | Crypto-shred blob; keep verification outcome + doc type + masked number | Only via incident/legal hold with `dpo` approval | Not accounting evidence | `dpo` |
| Biometric data (face templates, liveness signals) | Optional face match | Explicit consent / authorization where lawful | **Not persisted.** Templates processed transiently, deleted immediately after match result; only `match_result, score_band, consent_ref` retained | Match complete | In-memory only; vendor contract requires zero retention | No (never stored) | n/a | `dpo` |
| Signatures and e-sign evidence | Proof of agreement (registration card, contracts) | Contract / legal obligation | Same as the signed document's retention class | Document finalization | With document | Yes | Contracts/registration cards follow document class | `compliance_officer` |
| OTP / verification logs | Security, fraud investigation | Legitimate interest / security | 90 days; OTP values never stored in clear (hashed challenge only) | Challenge created | Hard delete | Yes (security incident) | n/a | Security Lead |
| CCTV images/clips (pointers; footage stays in VMS) | Security, incident evidence | Legitimate interest / legal (signage) | Per VMS policy (proposed ≤ 30 days) unless incident-linked | Recording | VMS overwrite; MetriStay keeps pointer only | Yes | Incident evidence class | `security_officer` |
| LPR plates and plate images | Parking access, billing | Contract / legitimate interest | Plate images 7 days; plate text on parking sessions 90 days, then hashed; permits until expiry + 30 days | Session exit / permit expiry | Delete images; hash plates | Yes | Parking charge on folio retained as accounting record without plate | `security_officer` |
| Payroll records, SIN and national IDs of employees | Payroll, tax, social security, WPS | Legal obligation | Per payroll/tax books-and-records rules per market — **unverified; counsel to confirm** (proposed: retain until verified period expires, never less than the statutory period once confirmed) | Tax year end / employment end | Hard delete + key shred after period | Yes | Payroll journals are accounting evidence | `payroll_officer` |
| Sample photos (RFQ) | Supplier sample comparison | Contract / legitimate interest | **90 days** (product requirement, SF49.1.4) after configured reference timestamp (RFQ close or approval — D-072) | Configured reference event | Automatic purge job with deletion proof | Yes — narrow, approved hold only | Separated from PO/invoice/food-trace records which follow their own class | `procurement_officer` |
| Bids, awards, POs, supplier invoices, GRNs | Procurement, AP, audit | Legal obligation / contract | Per accounting/tax retention per market — **unverified; counsel** | Fiscal year end | Hard delete after period | Yes | These *are* accounting evidence | `financial_controller` |
| Food traceability records (lot in/out) | Food safety, recall | Legal obligation | Canada: CFIA page states records kept "two years after the day on which the food was provided" (page states; to be confirmed by counsel). Other markets unverified | Food provided | Hard delete | Yes | Linked to GRN (accounting) | `executive_chef` |
| Incident evidence and chronology | Safety, insurance, legal claims | Legal obligation / legitimate interest | Until case closure + limitation period per market — **counsel** | Case closure | Delete or anonymize | Yes (default on for injury/police cases) | Insurance claims evidence | `security_officer` |
| AI transcripts (guest chat) | Service quality, dispute resolution | Consent / legitimate interest | 90 days full transcript (PII-masked), then aggregated metrics only | Conversation end | Hard delete | Yes | If transcript contains a booking/complaint, the case record (not the transcript) is retained | `guest_relations` |
| Integration raw payloads | Debugging, reconciliation | Legitimate interest | 90 days, then hashed summary | Receipt | Delete | Yes | Money-related provider references kept as accounting evidence | `integration_admin` |
| Lost-and-found item records/photos | Reuniting property | Legitimate interest | Per disposal policy by jurisdiction (SF43.1.7) + 1 year | Release/disposal | Delete photos; keep custody log summary | Yes | Fee receipts | `front_office_manager` |
| Referrer identity/tax/payment data | Commission payout, tax | Contract / legal obligation | Contract term + accounting retention | Contract end | Hard delete after period | Yes | Commission payouts are accounting evidence | `referral_program_admin` |

### 5.3 Legal hold and deletion-vs-accounting rules
1. **Legal hold** (`legal_hold`: scope, reason, requested_by, approved_by `dpo`+`compliance_officer`, review_date ≤ 180 days) suspends deletion for listed objects only; it cannot be applied silently or indefinitely; review reminders escalate.
2. **Erasure requests** delete or anonymize profile/marketing/ID/biometric data, but **do not delete accounting evidence** (invoices, folios, payments, payroll journals, tax records). Those records are *restricted* (access limited to finance/audit) and minimized to fields required by accounting law; the guest is told which records are retained and why.
3. Financial ledgers are append-only: deletion never removes a ledger row; personal fields referenced by ledgers live in a separate, crypto-shreddable table referenced by pseudonymous id, so the ledger remains balanced after erasure.
4. Sample photos are stored in a separate bucket/class from purchase and food-trace records so the 90-day purge cannot delete evidence, and evidence retention cannot extend sample retention (SF49.1.4).
5. Every scheduled deletion writes a deletion receipt; the M65 dashboard shows due/overdue deletions and holds.

---

## 6. PCI DSS boundary

- **Current standard:** PCI SSC document library lists **PCI DSS v4.0.1** (retrieved 2026-09-28, `source-cited`).
- **Design rule:** MetriStay never receives, processes, stores or transmits PAN, CVV/CVC, track data or PIN. Card capture happens only in (a) PSP hosted fields/iframes or redirect pages, (b) PSP-provided semi-integrated terminals (P2PE where offered), (c) channel virtual cards delivered as PSP tokens (INT-CHANNEL), never raw.
- **Stored card data:** token reference, brand, last4, expiry month/year (optional), PSP customer id. No PAN in general PMS tables, logs, notes, emails, chat transcripts, OCR (a PAN detector blocks and redacts free text and uploads).
- **SAQ scope assumption (unverified, to validate with PSP/QSA):** e-commerce pages using PSP iframe/hosted fields → target the SAQ applicable to fully outsourced e-commerce, with awareness that script-integrity requirements in v4.x apply to the page hosting the iframe; card-present via semi-integrated/P2PE terminals → target the terminal-specific SAQ. Final SAQ type determined by the acquirer/PSP and, if required, a QSA (D-073). Because on-prem hotel deployments host payment pages differently, each profile is assessed separately.
- **Controls still in scope:** payment page script inventory and integrity monitoring, CSP, TLS, change control, vulnerability scanning of internet-facing assets, access control for staff using terminals, MOTO handling policy (staff key-entry only into PSP virtual terminal, never into MetriStay).
- **Test:** PCI-01 scan database, logs and object storage in staging for PAN patterns (Luhn) — zero findings required; PCI-02 PAN pasted into any free-text field is rejected/redacted.

---

## 7. CBO/PSP and wallet gating (Oman and other markets)

- **Source status:** CBO PSP licensing policy PDF (URL from master prompt) **could not be retrieved** on 2026-09-28 (HTTP 503) → content `unverified`. Master prompt states CBO publishes a PSP licensing policy and that a cash/stored-value wallet or money transfer may require licensing.
- **Design decisions:**
  1. **MetriStay Rewards points wallet** (M30) is a closed-loop, non-cash points ledger: no cash-out, no peer transfer, redemption only on eligible hotel purchases. Treated as a loyalty liability, not stored value. Counsel question per market: confirm points programme is outside payment/e-money licensing (Q-OM-PAY-02 etc. in §12).
  2. **Cash/stored-value wallet** is **disabled** in all markets (`feature.cash_wallet=false`, server-enforced). Activation requires: licensed bank/PSP partner contract, legal opinion per market, KYC/AML responsibility allocation, safeguarding, limits and dispute process (SF30.2.6).
  3. **Bill payment** (M29) is initiated through a licensed provider; MetriStay does not hold funds.
  4. **Referral payouts** are made by the hotel/company through its bank/PSP (INT-BANK), not from a MetriStay balance.
  5. Gift vouchers (M54) are hotel-issued prepaid service vouchers; counsel question whether any market treats them as stored value.
- **Gate:** `GATE-PAY-WALLET-<country>` (states: `off` → `legal_opinion_filed` → `partner_contracted` → `sandbox_tested` → `live`); the API rejects wallet operations unless `live`.

---

## 8. Oman Ministerial Decision 105/2021 and the single-tier referral activation gate

### 8.1 Source status
- Decision text via qanoon.om (URL given in master prompt; a legal database reproducing the Official Gazette) retrieved 2026-09-28: Decision **105/2021**, dated 26 July 2021, published in Official Gazette No. 1401 on 1 August 2021, effective the day after publication; prohibits sale, purchase, trading, advertising or promotion of goods, products or services through network or pyramid marketing by any means. Article 1 definition (English summary of Arabic text): a method by which a supplier or advertiser invites consumers to select lists of other consumers to purchase goods or receive services in exchange for benefits, arranging them in hierarchical or networked groups to collect money from the largest number of participants. Penalty summarized as an administrative fine of OMR 5,000, doubling on repetition (page states; to be confirmed by counsel).
- Official Ministry of Justice listing (mjla.gov.om) **could not be retrieved** (HTTP 503) → the official text is `unverified` against the primary publisher; status `source-cited (secondary legal database)`.

### 8.2 Assessment (inference, not clearance)
| Element of the definition | MetriStay single-tier design | Inference |
|---|---|---|
| Consumers invited to recruit other consumers | Referrer shares a booking link; referred guest is **not** invited to become a referrer as a condition or for any reward to the referrer | Appears different |
| Hierarchical or networked grouping | No upline/downline table; attribution stores at most one `referrer_id` per booking; no recursive relationships (schema test) | Appears different |
| Benefit in exchange for bringing participants | Commission only on a completed, paid, non-reversed stay booked by the directly referred guest; zero for recruiting | Appears different |
| Collecting money from participants | No joining fee, no purchase requirement, guest price unchanged | Appears different |

**Conclusion (engineering inference only):** the specified one-tier, direct-sale commission appears materially different from the defined network/pyramid model. **This is not legal clearance.** Counsel must assess actual terms, promotion method, referrer categories (individual consumers vs companies), whether consumer referrers could be organized as a marketing network in practice, social-media marketing licence requirements (gov.om service page retrieved: Ministry of Commerce, Industry and Investment Promotion licenses commercial companies to market on social media), promotional-offer permits (gov.om page retrieved: MoCIIP permit for promotional offers/discounts/marketing cards), whether referral activity constitutes travel-agency activity (gov.om page retrieved: Ministry of Heritage and Tourism travel & tourism agency licence), and tax treatment including withholding on payments to non-resident referrers (Tax Authority page retrieved lists service fees among payment categories subject to withholding for foreign persons without a PE; the page states a 10% rate and remittance by the 14th of the following month — page states; to be confirmed by counsel).

### 8.3 Activation gate `GATE-REF-OM`
| Step | Evidence required | Approver |
|---|---|---|
| 1 Build & sandbox test (Phase 5) | Non-negotiable tests from master prompt Section E pass (double claim, recruiting earns zero, second-level earns zero, refund reverses once, zero-margin earns zero, disabled jurisdiction rejects payout) | Engineering + `referral_program_admin` |
| 2 Final commercial terms | Commission formula version, allowed deductions, referrer categories, promotion channels, disclosure text | Product Owner |
| 3 Omani legal opinion | Written opinion on MD 105/2021, consumer protection, social-media licence, promotional permit, tourism licence | Omani counsel → `compliance_officer` files as `counsel-reviewed` |
| 4 Tax opinion | VAT, withholding, income treatment for each referrer category | Omani tax adviser |
| 5 Activation | Gate set per country + legal entity + referrer category + channel; payouts server-enforced | `compliance_officer` + `gm` (dual) |
| Kill switch | Instant disable by country/hotel/category/campaign; earned commissions governed by contract, not deleted | `referral_program_admin` |

Other markets: separate `GATE-REF-<country>` with that market's consumer, advertising (endorsement disclosure — FTC guidance retrieved as a disclosure benchmark only; not an Oman/CA/PK/SA/PT law), tourism/agency, privacy, payment and tax review. **No multi-level compensation anywhere** (M31, Section E).

---

## 9. Oman Wage Protection System (WPS)

- **Retrieved (2026-09-28, `source-cited`):** MoL "About WPS" describes an electronic system shared between the Ministry of Labour and the Central Bank of Oman, implementing Article 87 of the Labour Law (wages transferred to licensed local financial institutions), using a standardized wage file. MoL FAQ describes a CSV "Wage Information File" with naming conventions (commercial registration number, bank code, date, sequence), phased compliance dates (large/medium enterprises 50% by 9 Nov 2023 and 100% by 9 Jan 2024; small enterprises by 9 Mar 2024), coverage of Omani and non-Omani private-sector workers, detection of discrepancies against a legally mandated payment window, and penalties including warnings, suspension of e-services, an administrative fine (FAQ states OMR 50, doubling on repetition) and judicial referral (page states; to be confirmed by counsel).
- **Unverified:** exact file columns, bank-specific upload workflow, response-file format, treatment of exceptions (leave without pay, final settlement).
- **System design:** INT-WPS-OM (`docs/05` §3.3); payroll run cannot be marked `paid` until bank confirmation; WPS file generation restricted to `payroll_officer`/`payroll_approver`; file stored encrypted; individual salaries never shown outside payroll roles (F27.4).
- **Gate:** `GATE-OM-WPS`: spec version obtained from bank + test file accepted by bank → live.

---

## 10. Canada: SIN, CPP/EI, QPP/QPIP, T4 and GST/HST/provincial/municipal levies

### 10.1 Payroll
- **Retrieved (`source-cited`):** CRA "Calculate payroll deductions and contributions" — employers determine whether to deduct CPP, EI and income tax, including for non-regular payments and special situations; CRA provides CPP contribution tables, EI premium tables, claim codes, income-tax tables and an online calculator. CRA information-return e-filing overview — **Web Forms** and **Internet File Transfer (XML)** are the electronic filing methods, with published XML specifications/schemas; authorized representatives may file. The CRA T4 slip page was retrieved but its filing-method details are on a linked page.
- **Unverified:** Québec QPP/QPIP/RL-1 details (Revenu Québec page returned HTTP 403); provincial payroll specifics; current-year parameters (no rates or maximums are stated in this pack).
- **SIN handling:** field-level encryption with a payroll-only key; masked display (last 3 digits) in UI; full value decrypted only in T4/RL artifact generation and payroll-provider export; access logged and reviewed; SIN never used as an internal identifier or login; validation by format/checksum only.
- **Design:** yearly parameter tables are data in a `verified` rule pack with CRA/RQ source links and reviewer; engine blocks payroll finalization if the tax-year pack is not verified; option to delegate calculation to a Canadian payroll provider (D-056 in `docs/05`).

### 10.2 Sales tax and accommodation levies
- **Retrieved (`source-cited`):** CRA GST/HST travel/convention industry page — short-term accommodation is generally taxable with stated exceptions (page mentions low-priced accommodation and long-term residential stays; exact thresholds to be confirmed by counsel); accommodation-related services (room service, pay-per-view, telephone etc.) are taxable; tour packages require allocation among differently taxed elements; travel-agency commissions are generally taxable unless the underlying service is zero-rated. CRA GST/HST Internet File Transfer accepts a `.tax` format from CRA-certified software.
- **Unverified / counsel:** rate by province (GST vs HST provinces), provincial sales tax and provincial accommodation taxes, municipal accommodation taxes/levies (collected via municipality or designated entity), place-of-supply rules for packages and events, exemptions (e.g. diplomatic, First Nations), invoice content requirements.
- **Design:** jurisdiction classifier requires province + municipality for each Canadian property; each levy is a separate effective-dated tax component; tax invoice lists each component; returns prepared as worksheets; filing route per `docs/05` INT-GOV-CA.

---

## 11. "NIS" terminology decision

| Item | Content |
|---|---|
| Problem | The user's term "NIS" is ambiguous. In Canada the employee identifier is the **Social Insurance Number (SIN)** (French: *numéro d'assurance sociale*, NAS). "NIS" is not a Canadian tax identifier. |
| Plausible intended meanings (inference, to confirm with user) | (a) a typo/transposition for Canadian **SIN**; (b) Portugal's social security number (*Número de Identificação da Segurança Social*, commonly "NISS"); (c) another market's national insurance/identity scheme; (d) an internal abbreviation (e.g. "National ID Scheme"). None verified. |
| Decision **D-074** | (1) The data model has **no field named "NIS"**. (2) Employee identifiers are modelled as `employee_identifier(type, jurisdiction, value_encrypted)` with a controlled vocabulary per rule pack: `CA_SIN`, `PT_NIF`, `PT_NISS`, `OM_CIVIL_ID`, `SA_NATIONAL_ID`/`SA_IQAMA`, `PK_CNIC` — each vocabulary entry `unverified-assumption` until the market pack is reviewed. (3) Canadian payroll uses `CA_SIN` only. (4) The user must confirm the meaning of "NIS"; answer recorded in `docs/13` decision log. |
| Owner | Product Owner (question to user) + `payroll_officer` |
| Gate | Payroll rule pack cannot reach `verified` while any required identifier type in that market is unconfirmed. |

---

## 12. Five-market validation register

### 12.1 How to read
- **Source status** values: `retrieved 2026-09-28` (primary page opened and content read), `retrieval failed 2026-09-28 (<reason>) – unverified`, `secondary only – unverified`, `not attempted – unverified`. Retrieval never implies counsel review.
- **Confirmed / Assumption / Counsel Q:** **C** = confirmed from a retrieved primary source (still "to be confirmed by counsel" for legal effect); **A** = assumption; **Q** = counsel question.
- **Filing route:** `API` only where a primary official source says so; otherwise `file`, `portal`, `manual`, or `unknown`.
- **System gate:** the M44 activation gate that blocks the dependent feature while the row is not `counsel-reviewed`/`verified`. Gate names: `GATE-<CC>-<CAT>`.
- Owner roles: `compliance_officer` (CO), `financial_controller` (FC), `payroll_officer` (PO), `dpo`, `front_office_manager` (FOM), `referral_program_admin` (RPA), `executive_chef` (EC), `concierge` (CON), plus external local counsel/adviser (LC).

### 12.2 Canada (federal + province + municipality)
| ID | Category | Obligation | Candidate authority | Primary source URL | Source status | C / A / Q | Filing route | Owner | System gate |
|---|---|---|---|---|---|---|---|---|---|
| CA-01 | Tax (federal) | GST/HST on short-term accommodation, F&B, related services; package allocation | Canada Revenue Agency (CRA) | https://www.canada.ca/en/revenue-agency/services/tax/businesses/topics/gst-hst-businesses/charge-collect-specific-situations/gst-hst-information-travel-convention-industry.html | retrieved 2026-09-28 | C: taxability principles on page. Q: rates, thresholds, exemptions, place of supply per property | portal (My Business Account/NETFILE) or file (IFT via CRA-certified software) | FC + LC | GATE-CA-TAX |
| CA-02 | Tax (provincial) | Provincial sales tax / HST component / provincial accommodation tax per province | Provincial finance ministry/revenue agency (per province; e.g. Revenu Québec for QST) | not identified — per province | not attempted – unverified | A: provincial component exists in some provinces. Q: which applies to pilot province | unknown | FC + LC | GATE-CA-TAX-<prov> |
| CA-03 | Tax (municipal) | Municipal accommodation tax / destination levy | Municipality or designated tourism entity | not identified — per municipality | not attempted – unverified | A: some municipalities levy; Q: pilot municipality rules, remitter, return format | manual/portal | FC + LC | GATE-CA-TAX-<muni> |
| CA-04 | E-invoicing | Tax invoice content requirements (no national e-invoicing mandate assumed) | CRA | (GST/HST invoice requirements page — not retrieved) | not attempted – unverified | A: no clearance e-invoicing; Q: invoice information requirements by amount | n/a | FC | GATE-CA-INV |
| CA-05 | Payroll (federal) | CPP, EI, income tax withholding; remittance; T4 | CRA | https://www.canada.ca/en/revenue-agency/services/tax/businesses/topics/payroll/calculating-deductions.html ; https://www.canada.ca/en/revenue-agency/services/tax/businesses/topics/payroll/completing-filing-information-returns/t4-information-employers/t4-slip.html | retrieved 2026-09-28 (both) | C: employer calculates CPP/EI/income tax using CRA tables. Q: yearly parameters; provider vs native | T4: file (IFT XML) or portal (Web Forms) — C from e-filing overview; remittance via bank | PO + LC | GATE-CA-PAY |
| CA-06 | Payroll (Québec) | QPP, QPIP, provincial income tax, RL-1, other Québec employer contributions | Revenu Québec | https://www.revenuquebec.ca/en/businesses/source-deductions-and-employer-contributions/ | retrieval failed 2026-09-28 (HTTP 403) – unverified | A: QPP replaces CPP and QPIP applies in Québec (master prompt). Q: full list and filing route | unknown | PO + LC | GATE-CA-PAY-QC |
| CA-07 | Guest registration | Guest register / police reporting | Provincial/municipal (if any) | none identified | not attempted – unverified | A: no general federal guest-reporting duty; Q: provincial/municipal hotel register rules | manual | FOM + LC | GATE-CA-GREG |
| CA-08 | ID / biometrics / privacy | Consent, purpose limitation, biometrics, identification (PIPEDA + provincial laws such as Québec, Alberta, BC) | Office of the Privacy Commissioner of Canada; provincial commissioners | https://www.priv.gc.ca/en/privacy-topics/health-information-genetics-biometrics/biometrics/bio-tips_org/ ; https://www.priv.gc.ca/en/privacy-topics/identities/identification-and-authentication/auth_061013/ | retrieved 2026-09-28 (both) | C: OPC guidance — legitimate purpose, express consent for biometrics, minimization, safeguards; collect minimum ID data; authentication proportional to risk. Q: provincial law applicable to pilot; ID copy retention | n/a | dpo + LC | GATE-CA-PRIV / GATE-CA-BIO |
| CA-09 | Payments | Card acceptance via acquirer; no wallet licensing assumed needed for points | Acquirer/PSP; FINTRAC/Bank of Canada for payment services (Q) | not retrieved | not attempted – unverified | A: points ledger outside payment regulation; Q: gift voucher and cash-wallet rules | n/a | FC + LC | GATE-PAY-WALLET-CA |
| CA-10 | Tourism / travel intermediation | Travel-agent registration where selling travel (provincial, e.g. Ontario/Québec/BC) | Provincial travel regulators | not retrieved | not attempted – unverified | Q: whether hotel concierge booking flights/cruises requires registration | n/a | CON + LC | GATE-CA-TRAVEL |
| CA-11 | Marketing / referral | Anti-spam consent for electronic messages; competition/deceptive marketing; referral commission disclosure | CRTC / Competition Bureau | not retrieved | not attempted – unverified | Q: CASL consent for referral links and campaigns; disclosure | n/a | RPA + dpo + LC | GATE-REF-CA / GATE-CA-MKT |
| CA-12 | Food safety | Traceability (one step back/forward), licensing where applicable | Canadian Food Inspection Agency; provincial/municipal health units | https://inspection.canada.ca/en/food-safety-industry/traceability/traceability | retrieved 2026-09-28 | C: page states traceability records one step back/forward retained two years (to be confirmed by counsel for hotel kitchens, which may fall under provincial rules). Q: provincial food premises rules | n/a | EC + LC | GATE-CA-FOOD |
| CA-13 | Retention | Books and records retention (tax/payroll) | CRA | not retrieved | not attempted – unverified | Q: retention period and electronic records rules | n/a | FC + LC | GATE-CA-RET |
| CA-14 | Government exchange method | GST/HST IFT requires CRA-certified software; information returns via Web Forms or IFT XML | CRA | https://www.canada.ca/en/revenue-agency/services/tax/businesses/topics/gst-hst-businesses/calculate-prepare-report/software-gst-hst-internet-file-transfer.html ; https://www.canada.ca/en/revenue-agency/services/e-services/filing-information-returns-electronically-t4-t5-other-types-returns-overview.html | retrieved 2026-09-28 (both) | C: file-based channels listed; no general public "API" found. Q: pursue IFT certification? | file / portal | CO | GATE-CA-GOVX |

### 12.3 Oman
| ID | Category | Obligation | Candidate authority | Primary source URL | Source status | C / A / Q | Filing route | Owner | System gate |
|---|---|---|---|---|---|---|---|---|---|
| OM-01 | Tax | VAT on hotel services; tourism/municipality fees on hotel bills | Oman Tax Authority; municipality / Ministry of Heritage and Tourism (fees) | https://tms.taxoman.gov.om/portal/ | retrieved 2026-09-28 (portal: VAT services exist) | C: VAT administered via portal. Q: rate, hotel-specific fees, invoice rules | portal | FC + LC | GATE-OM-TAX |
| OM-02 | Tax (withholding) | WHT on payments to foreign persons without PE (incl. services, software, management fees) — relevant to referrers/vendors abroad | Oman Tax Authority | https://tms.taxoman.gov.om/portal/withholding-tax | retrieved 2026-09-28 | C: categories listed; page states 10% and remittance by 14th of following month (to be confirmed by counsel). Q: applicability to referral commissions/SaaS fees | portal | FC + LC | GATE-OM-WHT |
| OM-03 | E-invoicing | Fawtara e-invoicing programme | Oman Tax Authority | https://tms.taxoman.gov.om/portal/web/taxportal/fawtara | retrieved 2026-09-28 (content was a service-provider terms document referencing onboarding to an "Oman SMP") | A: e-invoicing programme exists. Q: scope, timeline, model, technical spec | unknown | FC + CO | GATE-OM-EINV |
| OM-04 | Payroll / social security | WPS wage file via bank; social protection contributions | Ministry of Labour; Central Bank of Oman; Social Protection Fund | https://mol.gov.om/pages/ABOUT-WPS ; https://www.mol.gov.om/FAQ | retrieved 2026-09-28 (both) | C: WPS, standardized CSV wage file, phased dates, penalties (see §9). Q: file columns, SPF contribution rules | file (via bank) | PO + LC | GATE-OM-WPS / GATE-OM-PAY |
| OM-05 | Guest registration | Guest data reporting to police/tourism authority | Royal Oman Police; Ministry of Heritage and Tourism | none identified | not attempted – unverified | Q: whether and how hotels report guest data | unknown | FOM + LC | GATE-OM-GREG |
| OM-06 | ID / biometrics / privacy | Personal Data Protection Law (RD 6/2022): explicit consent, sensitive data (incl. biometric) requires ministry authorization, breach notification, DPO | Ministry of Transport, Communications and IT (Q: current competent body) | https://qanoon.om/p/2022/rd2022006/ | retrieved 2026-09-28 (secondary legal database) | C (secondary): obligations summarized. Q: executive regulation, permit process for biometric processing, cross-border transfer | n/a | dpo + LC | GATE-OM-PRIV / GATE-OM-BIO |
| OM-07 | Payments | PSP licensing; stored-value wallet | Central Bank of Oman | https://cbo.gov.om/sites/assets/FintechCompulsoryDocs/Circular%20BM%201192-%20PSP%20policy.pdf | retrieval failed 2026-09-28 (HTTP 503) – unverified | A: wallet/transfer may require licence (master prompt). Q: points and voucher treatment | n/a | FC + LC | GATE-PAY-WALLET-OM |
| OM-08 | Tourism / travel intermediation | Travel & tourism agency licence for selling travel | Ministry of Heritage and Tourism | https://gov.om/en/w/get-travel-and-tourism-agencies-license | retrieved 2026-09-28 | C: licence service exists (MHT). Q: whether hotel concierge booking or referral requires it | n/a | CON + LC | GATE-OM-TRAVEL |
| OM-09 | Marketing / referral | MD 105/2021 network/pyramid marketing ban; social-media marketing licence; promotional permit | Ministry of Commerce, Industry and Investment Promotion | https://qanoon.om/p/2021/mociip20210105/ ; https://mjla.gov.om/decisions/ar/1088/show/1454 ; https://gov.om/en/w/get-a-license-to-practice-marketing-and-promotion-on-social-media-platforms ; https://gov.om/en/w/get-a-permit-for-organizing-promotional-offers-discounts-and-marketing-cards | qanoon + both gov.om pages retrieved 2026-09-28; mjla retrieval failed (HTTP 503) | C (secondary for decision text; primary for licence/permit services). Q: full §8 assessment | n/a | RPA + LC | GATE-REF-OM |
| OM-10 | Food safety | Food establishment licensing, hygiene, traceability | Municipality; Food Safety and Quality Center (Q) | none identified | not attempted – unverified | Q: requirements for hotel kitchens | n/a | EC + LC | GATE-OM-FOOD |
| OM-11 | Retention | Tax, labour and PDPL retention periods | Tax Authority; MoL; PDPL regulator | none identified | not attempted – unverified | Q: periods | n/a | FC + dpo + LC | GATE-OM-RET |
| OM-12 | Government exchange method | Tax portal; WPS via bank file; Fawtara unknown | Tax Authority; MoL/CBO | as OM-01, OM-03, OM-04 | see above | C: portal and bank file routes; Q: any API | portal / file / unknown | CO | GATE-OM-GOVX |

### 12.4 Pakistan (federal vs provincial)
| ID | Category | Obligation | Candidate authority | Primary source URL | Source status | C / A / Q | Filing route | Owner | System gate |
|---|---|---|---|---|---|---|---|---|---|
| PK-01 | Tax (federal) | Sales tax on goods (e.g. retail items, minibar stock purchases) and federal returns | Federal Board of Revenue (FBR) | https://www.fbr.gov.pk/ | retrieved 2026-09-28 | C: FBR administers sales tax, returns, IRIS portal. Q: which hotel supplies fall under federal sales tax | portal (IRIS) | FC + LC | GATE-PK-TAX-FED |
| PK-02 | Tax (provincial services) | Provincial sales tax on services (hotels, restaurants, events) — by province | Punjab Revenue Authority; Sindh Revenue Board; KP Revenue Authority; Balochistan Revenue Authority | https://pra.punjab.gov.pk/ ; https://www.srb.gos.pk/ (KPRA/BRA not attempted) | retrieval failed 2026-09-28 (HTTP 503, both) – unverified | A: services taxed provincially (constitutional split — to verify). Q: rates, registration, returns, hotel/restaurant specifics per province | unknown | FC + LC | GATE-PK-TAX-<prov> |
| PK-03 | Tax (income) | Income tax withholding on payments (vendors, salaries, commissions) | FBR | https://www.fbr.gov.pk/ | retrieved 2026-09-28 (home only) | Q: withholding categories and rates | portal (IRIS) | FC + LC | GATE-PK-WHT |
| PK-04 | E-invoicing / fiscal POS | POS integration under Sales Tax Rules 2006 Ch. XIV (Tier-1 retailers); FBR Digital Invoicing System; provincial POS schemes | FBR; provincial authorities | https://fbr.gov.pk/pos-legal-provisions/163085/163086 | retrieved 2026-09-28 | C: POS integration rules exist for Tier-1 retailers with per-invoice service charge (S.R.O. 1279(I)/2021) and 2025 amendments. Q: applicability to hotels/restaurants; provincial e-invoicing; technical spec | unknown (API indicated by FBR materials; spec not retrieved) | FC + CO | GATE-PK-EINV |
| PK-05 | Payroll / social security | EOBI, provincial social security, income tax on salaries | EOBI; provincial social security institutions; FBR | https://eobi.gov.pk/ | retrieval failed 2026-09-28 (HTTP 503) – unverified | Q: contributions, registration, returns | unknown | PO + LC | GATE-PK-PAY |
| PK-06 | Guest registration | Guest reporting to police (provincial/local) | Provincial police / local administration | none identified | not attempted – unverified | Q: requirement and route (form/portal) | unknown | FOM + LC | GATE-PK-GREG |
| PK-07 | ID / biometrics / privacy | Data protection (status of federal data protection law to verify); CNIC handling | Ministry of IT & Telecom; NADRA (ID verification services) | none identified | not attempted – unverified | Q: enacted law status, ID copy and biometric rules | n/a | dpo + LC | GATE-PK-PRIV / GATE-PK-BIO |
| PK-08 | Payments | Acquiring; e-money/wallet licensing | State Bank of Pakistan | none identified | not attempted – unverified | Q: points/voucher treatment | n/a | FC + LC | GATE-PAY-WALLET-PK |
| PK-09 | Tourism / travel intermediation | Travel agency registration; hotel registration/grading | Department of Tourist Services (federal/provincial) | none identified | not attempted – unverified | Q | n/a | CON + LC | GATE-PK-TRAVEL |
| PK-10 | Marketing / referral | Consumer protection (provincial), advertising, referral commissions | Provincial consumer authorities; Competition Commission of Pakistan | none identified | not attempted – unverified | Q | n/a | RPA + LC | GATE-REF-PK |
| PK-11 | Food safety | Provincial food authorities (licensing, hygiene) | e.g. Punjab Food Authority, Sindh Food Authority | none identified | not attempted – unverified | Q | n/a | EC + LC | GATE-PK-FOOD |
| PK-12 | Retention | Tax record retention | FBR / provincial | none identified | not attempted – unverified | Q | n/a | FC + LC | GATE-PK-RET |
| PK-13 | Government exchange method | IRIS portal (federal); POS integration; provincial portals | FBR; provincial authorities | https://www.fbr.gov.pk/ | retrieved 2026-09-28 | C: portal exists; Q: API/POS spec | portal / unknown | CO | GATE-PK-GOVX |

### 12.5 Saudi Arabia
| ID | Category | Obligation | Candidate authority | Primary source URL | Source status | C / A / Q | Filing route | Owner | System gate |
|---|---|---|---|---|---|---|---|---|---|
| SA-01 | Tax | VAT on hotel services; municipal/tourism fees (Q) | Zakat, Tax and Customs Authority (ZATCA); municipality / Ministry of Tourism | https://zatca.gov.sa/en/E-Invoicing/Pages/default.aspx (VAT page not retrieved) | retrieved 2026-09-28 (e-invoicing page) | Q: rates, fees, invoice rules | portal | FC + LC | GATE-SA-TAX |
| SA-02 | E-invoicing | FATOORA Phase 1 (from 4 Dec 2021) and Phase 2 integration (from 1 Jan 2023, in waves); specifications, XML and security standards; qualified solution providers list | ZATCA | https://zatca.gov.sa/en/E-Invoicing/Pages/default.aspx ; https://zatca.gov.sa/en/E-Invoicing/SystemsDevelopers/Pages/default.aspx | retrieved 2026-09-28; developer portal sandbox.zatca.gov.sa retrieval failed (HTTP 403) | C: programme phases and developer portal existence. Q: wave applicable to entity, onboarding/certificates, API details | API indicated by ZATCA developer portal reference (details unverified) | FC + CO | GATE-SA-EINV |
| SA-03 | Payroll / social security | GOSI contributions; wage protection (Mudad) (Q) | GOSI; Ministry of Human Resources | https://www.gosi.gov.sa/ | retrieval failed 2026-09-28 (timeout) – unverified | Q: all | unknown | PO + LC | GATE-SA-PAY |
| SA-04 | Guest registration | Guest data registration with Ministry of Interior (Shomoos); tourism occupancy reporting | Ministry of Interior (National Information Center); Ministry of Tourism | none primary (secondary vendor pages found via search) | secondary only – unverified | A (secondary): registration mandatory for licensed facilities. Q: primary legal basis, access method, certification of PMS | unknown | FOM + CO | GATE-SA-GREG |
| SA-05 | ID / biometrics / privacy | Personal Data Protection Law (PDPL) and regulations; sensitive/biometric data; cross-border transfer | Saudi Data & AI Authority (SDAIA) | https://sdaia.gov.sa/en/SDAIA/about/Documents/Personal%20Data%20English%20V2-23April2023-%20Reviewed-.pdf | retrieval failed 2026-09-28 (request rejected) – unverified | Q: all | n/a | dpo + LC | GATE-SA-PRIV / GATE-SA-BIO |
| SA-06 | Payments | Acquiring (local scheme acceptance); e-wallet licensing | Saudi Central Bank (SAMA) | none identified | not attempted – unverified | Q | n/a | FC + LC | GATE-PAY-WALLET-SA |
| SA-07 | Tourism / travel intermediation | Hospitality facility licence; travel agency licensing | Ministry of Tourism | https://mt.gov.sa/ | retrieval failed 2026-09-28 (HTTP 403) – unverified | Q | n/a | CON + LC | GATE-SA-TRAVEL |
| SA-08 | Marketing / referral | Commercial/e-commerce advertising rules; referral commissions | Ministry of Commerce (Q) | none identified | not attempted – unverified | Q | n/a | RPA + LC | GATE-REF-SA |
| SA-09 | Food safety | Food establishment requirements | Saudi Food and Drug Authority; municipalities | none identified | not attempted – unverified | Q | n/a | EC + LC | GATE-SA-FOOD |
| SA-10 | Retention | Tax/e-invoice archiving; PDPL retention | ZATCA; SDAIA | none identified | not attempted – unverified | Q | n/a | FC + dpo + LC | GATE-SA-RET |
| SA-11 | Government exchange method | ZATCA e-invoicing (API indicated); portals; guest registration route unknown | ZATCA; MoI | as SA-02, SA-04 | see above | Q | API (indicated, ZATCA only) / portal / unknown | CO | GATE-SA-GOVX |

### 12.6 Portugal (incl. EU/GDPR)
| ID | Category | Obligation | Candidate authority | Primary source URL | Source status | C / A / Q | Filing route | Owner | System gate |
|---|---|---|---|---|---|---|---|---|---|
| PT-01 | Tax | VAT (IVA) on accommodation and F&B; municipal tourist tax (taxa turística) by municipality | Autoridade Tributária e Aduaneira (AT); municipalities | https://www.portaldasfinancas.gov.pt/ | retrieved 2026-09-28 (home only) | A: tourist tax is municipal. Q: rates, exemptions, per-municipality rules | portal | FC + LC | GATE-PT-TAX / GATE-PT-TAX-<muni> |
| PT-02 | E-invoicing / invoicing software | Certified invoicing software, invoice communication, ATCUD and QR code, SAF-T (PT) | AT | https://info.portaldasfinancas.gov.pt/pt/apoio_contribuinte/Faturacao/Paginas/default.aspx | retrieval failed 2026-09-28 (HTTP 404) – unverified | A: all listed items (not verified). Q: certification process, timeline for EU e-invoicing | unknown | FC + CO | GATE-PT-EINV |
| PT-03 | Payroll / social security | Social security contributions and monthly remuneration declarations; IRS withholding | Instituto da Segurança Social; AT | https://www.seg-social.pt/ | retrieval partial 2026-09-28 (only portal shell) – unverified | Q: all; NISS identifier (see §11) | unknown | PO + LC | GATE-PT-PAY |
| PT-04 | Guest registration | Communication of foreign guest stays (historically SIBA) | AIMA / successor authority to SEF (Q) | https://siba.sef.pt/ | retrieval failed 2026-09-28 (HTTP 503) – unverified | Q: current authority, route, deadlines | unknown | FOM + LC | GATE-PT-GREG |
| PT-05 | ID / biometrics / privacy (EU) | GDPR incl. special-category biometric data; national implementing law; supervisory authority | CNPD; EU | https://www.cnpd.pt/ ; https://eur-lex.europa.eu/eli/reg/2016/679/oj | CNPD retrieved 2026-09-28; EUR-Lex retrieval returned no content – unverified | C: CNPD is the Portuguese DPA supervising GDPR. Q: lawful basis for ID copies; biometrics; DPIA requirements | n/a | dpo + LC | GATE-PT-PRIV / GATE-PT-BIO |
| PT-06 | Payments | PSD2/SCA for card payments; e-money licensing | Banco de Portugal; EU | none identified | not attempted – unverified | A: SCA applies via PSP (3DS). Q: vouchers/points | n/a | FC + LC | GATE-PAY-WALLET-PT |
| PT-07 | Tourism / travel intermediation | Tourism establishment registration (RNT/RJET); travel agency registration (RNAVT); package travel rules | Turismo de Portugal | https://business.turismodeportugal.pt/ | retrieved 2026-09-28 | C: RNT and SI-RJET registries exist. Q: whether concierge travel booking constitutes travel-agency/package activity | portal | CON + LC | GATE-PT-TRAVEL |
| PT-08 | Marketing / referral | Unfair commercial practices (DL 57/2008, incl. pyramid promotion); e-privacy/marketing consent | Direção-Geral do Consumidor; ASAE; CNPD | https://diariodarepublica.pt/dr/detalhe/decreto-lei/57-2008-246504 | retrieval returned no content 2026-09-28 – unverified | A (master prompt): EU/PT pyramid rules focus on payments for recruitment. Q: referral disclosure, consent | n/a | RPA + dpo + LC | GATE-REF-PT |
| PT-09 | Food safety | EU hygiene/HACCP and traceability; national enforcement | ASAE; DGAV; EU | none identified | not attempted – unverified | Q | n/a | EC + LC | GATE-PT-FOOD |
| PT-10 | Retention | Accounting/tax document retention; GDPR storage limitation | AT; CNPD | none identified | not attempted – unverified | Q | n/a | FC + dpo + LC | GATE-PT-RET |
| PT-11 | Government exchange method | Invoice communication, SAF-T, guest reporting, social security declarations | AT; AIMA; Segurança Social | as PT-02 to PT-04 | see above | Q: official web-service routes (none asserted) | unknown | CO | GATE-PT-GOVX |

**Register count: 61 rows** (Canada 14, Oman 12, Pakistan 13, Saudi Arabia 11, Portugal 11). All rows are `unverified-assumption` or `source-cited`; none is `counsel-reviewed`.

### 12.7 Gate behaviour (M44)
- A gate in state other than `verified` blocks: automated filing/submission, live tax-invoice issuance where e-invoicing is mandatory, referral payout, wallet operations, travel ordering, biometric processing, and live guest-registration submission for that property/entity. It never blocks manual operations with evidence capture.
- The coverage dashboard (SF44.2.8) lists every row with owner, status and next action; expired reviews (default 12 months or on source change alert) revert the gate to `draft`.
- Acceptance: AT-G09 (five-market fixture) asserts that each of the five fixtures produces distinct rule packs and that unknown rows block dependent automation with reviewer/evidence/contingency displayed.

---

## 13. Source retrieval log (2026-09-28)

| Source | Result |
|---|---|
| CRA payroll deductions page | Retrieved |
| CRA T4 slip page | Retrieved (filing methods on linked page) |
| CRA information returns e-filing overview | Retrieved |
| CRA GST/HST travel & convention industry | Retrieved |
| CRA GST/HST Internet File Transfer | Retrieved |
| OPC biometrics tips for organizations | Retrieved |
| OPC identification and authentication | Retrieved |
| CFIA traceability | Retrieved |
| qanoon.om MD 105/2021 | Retrieved (secondary legal database) |
| qanoon.om RD 6/2022 (Oman PDPL) | Retrieved (secondary legal database) |
| MoL Oman About WPS | Retrieved |
| MoL Oman FAQ | Retrieved |
| Khedmah | Retrieved (no merchant API mentioned) |
| ONEIC Pay | Retrieved (no merchant API mentioned) |
| gov.om social-media marketing licence | Retrieved |
| gov.om promotional-offers permit | Retrieved |
| gov.om travel & tourism agency licence | Retrieved |
| Oman Tax Authority portal | Retrieved |
| Oman Tax Authority withholding-tax page | Retrieved |
| Oman Tax Authority Fawtara page | Retrieved (content was a service-provider terms document) |
| ZATCA e-invoicing page | Retrieved |
| ZATCA Systems Developers page | Retrieved |
| FBR home page | Retrieved |
| FBR POS legal provisions | Retrieved |
| Portal das Finanças home | Retrieved |
| CNPD home | Retrieved |
| Turismo de Portugal business portal | Retrieved |
| FTC endorsement guides FAQ | Retrieved |
| IATA NDC | Retrieved |
| OWASP API Security Top 10 2023 | Retrieved (after redirect) |
| PCI SSC document library | Retrieved |
| CBO PSP policy PDF | **Failed** (HTTP 503) |
| MJLA decision listing (Oman MoJ) | **Failed** (HTTP 503) |
| Diário da República DL 57/2008 | **Failed** (no content returned) |
| Punjab Revenue Authority | **Failed** (HTTP 503) |
| Sindh Revenue Board | **Failed** (HTTP 503) |
| AT invoicing information page | **Failed** (HTTP 404) |
| SIBA | **Failed** (HTTP 503) |
| ZATCA developer sandbox | **Failed** (HTTP 403) |
| SDAIA PDPL PDF | **Failed** (request rejected) |
| Revenu Québec source deductions | **Failed** (HTTP 403) |
| Saudi Ministry of Tourism | **Failed** (HTTP 403) |
| GOSI | **Failed** (timeout) |
| EOBI | **Failed** (HTTP 503) |
| EUR-Lex GDPR | **Failed** (no content returned) |
| Segurança Social | Partial (portal shell only) — treated as failed |
| Shomoos | Secondary vendor sources only (web search) — not primary |

Retrieval was done with automated fetching and summarization; the Compliance Lead must re-open each source, archive a dated copy in the source-evidence register (`docs/13`), and confirm each quoted statement before counsel review.

---

## 14. AI governance

| Control | Specification |
|---|---|
| Scope | Guest AI (M40), vendor follow-up drafting/parsing (M50), bid evidence summaries (M49), staff summaries; image enhancement (M39) under its own controls. |
| Identity & disclosure | Assistant always discloses it is an AI (SF40.1.3); never impersonates staff; hands off to humans on request. |
| Prompt injection | All user, vendor, review, email, document and retrieved content is **data**. System prompts are not secret-dependent. Tools are allow-listed per surface; tool arguments validated against schemas; object ids re-authorized against the *caller's* scopes by the tool gateway; outputs rendered as text (no HTML/markdown execution); links to external domains stripped or allow-listed. Red-team corpus (direct, indirect via KB/review/vendor email, multilingual Arabic/English, encoding tricks) in CI (SF40.1.5). |
| Tool scoping | Guest assistant tools: `search_availability`, `get_quote`, `get_policy`, `create_draft_booking` (draft only), `create_handoff`. No payment, refund, cancellation, folio, profile export or admin tools. Vendor follow-up tools: `get_po_milestones`, `draft_message`, `propose_status_update` (requires human confirmation for consequential changes, SF50.1.4). |
| No autonomous award | AI may summarize bids and flag anomalies; it cannot change weights, disqualify, rank-finalize or award (SF49.2.6). Award requires human approvers with segregation of duties. |
| No delivery/booking claims | AI cannot mark delivery occurred from chat, GPS ping or invoice (SF50.1.7); cannot tell a guest "booked" without system-of-record confirmation (and external reference for travel). |
| Safety escalation | Emergency keywords/intent trigger immediate human/local-services instruction (SF40.2.6); critical safety alerts never rely solely on AI (M42). |
| Data | PII masking before inference; no ID images, payment, payroll/SIN data in prompts; vendor models under zero-retention/DPA; local model option for on-prem. |
| Quality | Citations to KB article/version; hallucination tests; containment/handoff metrics; monthly sampled review by `guest_relations`. |
| Cost & availability | Per-property budgets, throttling, graceful fallback to human inbox. |
| Model licence | Open-weight model licences reviewed before on-prem deployment; image enhancement never fabricates facilities (SF39.2.4). |
| Jurisdiction | AI transcript retention and cross-border inference assessed per market privacy row (§12). |

---

## 15. Key management and secrets

- **Hierarchy:** cloud KMS/HSM (SaaS) or Vault with auto-unseal via hardware/TPM or offsite KMS (on-prem) → per-tenant key-encryption keys → per-data-class data keys (e.g. `payroll`, `id-image`, `guest-pii`, `bank-details`) → per-record data keys for crypto-shredding where required.
- **Separation:** payroll/SIN keys accessible only to payroll service identity; ID-image keys only to identity service; application DB role cannot decrypt payroll fields without service-level token.
- **Rotation:** data keys rotated annually or on incident (envelope re-wrap); partner API credentials per partner schedule (default 90 days where supported); webhook secrets support dual-secret overlap; TLS certificates automated (ACME) with 30-day renewal alerts; device certificates (edge agents, scales, gates) lifetime ≤ 1 year with registry (SF64.1.1).
- **Access:** no human access to production keys; break-glass requires two approvers and is logged; secrets never in code, images, logs or tickets (secret scanning in CI).
- **Backups:** encrypted with separate backup keys; key escrow procedure documented so on-prem hotels can restore after hardware loss.

---

## 16. Security testing plan

| Layer | Activity | Frequency / phase |
|---|---|---|
| Design | Threat model review per bounded context (this §3 extended per module) | Phase 1; update each phase |
| Code | SAST, dependency/SCA, secret scanning, IaC scanning, lint rules for tenant scoping | Every PR |
| API | Automated OWASP API tests SEC-* and ISO-* (§4) against OpenAPI; schema fuzzing | Every build |
| Money invariants | Replay/duplicate webhook, timeout-then-success, concurrent capture/refund, double payout, double stock receipt (G20) | Every build (mock adapters) + sandbox per connector |
| DAST | Authenticated scans of web apps and APIs in staging | Weekly from Phase 2 |
| Mobile | MASVS-aligned checks (storage, network, tamper), offline queue signing | Each release candidate |
| AI | Prompt-injection and data-leak corpus; tool-scope tests | Every build touching AI; monthly corpus refresh |
| Privacy | Retention job tests (sample photos purge at day 90, ID image purge, legal hold respected), erasure-vs-ledger test | Every build |
| PCI | PAN discovery scans in staging; payment-page script integrity | Monthly + before go-live |
| Infrastructure | CIS benchmark checks, container image scanning, K8s/Compose hardening, on-prem installer checklist | Every release |
| Penetration test | Independent third-party test of external surfaces, tenant isolation and payment flows | Before pilot (Phase 5/6) and annually |
| Restore | Full restore to isolated environment with integrity checks and RPO/RTO measurement (SF64.2.1) | Quarterly; mandatory in Phase 6 |
| Incident drills | Tabletop (ransomware, data breach, payment fraud, gate failure) | Phase 6 and semi-annually |

---

## 17. Biometric/ID consent and e-signature evidence

### 17.1 ID and biometric consent flow (M41)
1. Jurisdiction check determines: is ID required, which document types, may images be captured, may copies be retained, is biometric processing permitted (gate `GATE-<CC>-BIO`).
2. Guest sees purpose, retention, and the **non-biometric alternative** (staff visual check) before capture; biometric match is opt-in, separate from registration consent, and refusal has no service penalty (OPC guidance: express consent for biometrics, minimization — retrieved 2026-09-28).
3. Consent record: `{purpose, version, channel, language, timestamp, method (tap/signature), withdrawal_path}`; withdrawal stops processing and triggers deletion where no legal obligation applies.
4. Images: short-lived encrypted upload, OCR, guest field-by-field confirmation, deletion per §5.2; no analytics/AI training use.
5. Where Oman PDPL requires ministry authorization for biometric processing (secondary-source summary), `GATE-OM-BIO` stays off until authorization evidence is filed.

### 17.2 E-signature evidence bundle
`{document_id, document_version, sha256(document), signer identity evidence (authenticated session id, OTP verification ref, ID verification ref if any), intent statement shown, consent to electronic signing, signature image/vector or click-to-sign event, timestamp (trusted TSA token where configured), device/IP hash, geolocation only if lawful and disclosed, audit trail of views, final PDF with embedded signature and evidence page, bundle hash}`. Bundle stored immutably (WORM) with the document's retention class; verification endpoint recomputes hashes. Signature level per document/market (simple/advanced/qualified) is a counsel question (D-066 in `docs/05`).

---

## 18. Incident response

| Stage | Specification |
|---|---|
| Classification | Security incident (breach, account takeover, ransomware), privacy incident (misdirected data, over-retention), payment incident (fraud, double charge), safety-linked cyber incident (gate/BMS), AI incident (harmful output, data leak) |
| Detection | SIEM alerts from audit logs, anomaly rules (M60), partner notifications, user reports |
| Roles | Incident commander (Security Lead), `dpo` (privacy assessment and regulator/data-subject notifications), `financial_controller` (payments), `it_admin` (containment), comms owner, legal counsel |
| Containment | Revoke tokens/keys, disable connectors (kill switch per INT), isolate tenant, preserve logs (legal hold), switch to manual operations playbooks (M64) |
| Notification | Market-specific breach notification duties and deadlines (e.g. Oman PDPL requires notifying ministry and data subjects — secondary summary; GDPR supervisory authority notification for Portugal; Canadian breach reporting) — **deadlines to be confirmed per market by counsel** and encoded in rule packs; PSP/acquirer notification per contract |
| Evidence | Immutable incident chronology (M42 SF42.2.6), chain of custody for forensic images |
| Recovery | Restore from immutable backups, credential rotation, integrity checks of ledgers (trial balance, stock ledger invariants) |
| Post-incident | Postmortem within 10 business days, corrective work orders, register and threat-model updates |

---

## 19. Decisions and risks (reserved block 071–090)

| ID | Item | Owner | Default until resolved |
|---|---|---|---|
| D-071 | Name Security Lead | Product Owner | Architect acts |
| D-072 | Sample-photo purge start event (RFQ close vs approval) | `procurement_officer` | RFQ close |
| D-073 | SAQ type per deployment profile with PSP/QSA | `financial_controller` | Hosted fields + P2PE terminals |
| D-074 | "NIS" meaning (§11) | Product Owner | No NIS field; typed identifiers |
| D-075 | Retain counsel per market (tax, privacy, labour, tourism, consumer) | `compliance_officer` | All gates `draft` |
| D-076 | Pilot hotel country/province/municipality | Product Owner | Oman fixture + Canadian fixture |
| D-077 | Guest-registration submission route per market | `front_office_manager` | Manual/portal with evidence |
| D-078 | Biometric features in R1 at all? | `dpo` | Off in all markets |
| R-071 | Official sources unavailable/blocked from automated retrieval → evidence gaps | `compliance_officer` | Manual retrieval and archiving |
| R-072 | Mandatory e-invoicing (SA, possibly OM/PK/PT) discovered to require certification not achievable in timeline | `financial_controller` | Partner-certified solution option |
| R-073 | Referral programme reinterpreted as network marketing | `compliance_officer` | Gate off in Oman until opinion |
| R-074 | Cross-border inference/hosting conflicts with local data-transfer rules | `dpo` | On-prem/local-region profile |
| R-075 | Gate/LPR fail-safe conflicts with fire code | `security_officer` | Site-specific configuration reviewed by fire engineer |
