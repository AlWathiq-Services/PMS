# Catalogue 03 — Finance, procurement policy, utilities, workforce and payments (M19–M29)

**Pack:** MetriStay Hospitality Suite Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Governing source:** master prompt v3.0 Sections C (rows M19–M29), D (shared financial workflow), E (gateway and bill-provider APIs), G (acceptance), K (fixed features/subfeatures), L (schema), P (invariants).
**Conventions:** `docs/README.md` §3. IDs from Section K are reused verbatim; features/subfeatures marked **(added)** are required by Sections C, D, E, G or P but not enumerated in Section K.

> Specification only. No code, bank channel, PSP, WPS route, Khedmah/ONEIC API or government interface described here exists or is contracted today. Every external dependency carries an honesty label (README §3.6). Where a label is below `partner-contracted`, the production path is **blocked** and the manual/bank path is the operating route.

## 0. Scope, boundaries and cross-references

| Module | Bounded context (package) | Owns (system of record) | Does **not** own — reference instead |
|---|---|---|---|
| M19 GL and cost centers | `finance.ledger` | chart, cost centers, mappings, periods, journals, accruals, allocations, statements | business date (M01), tax codes/rates (M38/M44), KPI definitions (M32/M65), capex/depreciation policy (M66) |
| M20 AP/AR and treasury | `finance.payables`, `finance.receivables`, `finance.treasury` | supplier invoices, payables, AR invoices/receipts, bank statements, budgets, commitments, forecasts, cost-status snapshots | vendor legal identity and verification (M46), purchase order (M49), goods receipt (M50), guest folio (M08), cash shifts (M60) |
| M21 Procurement | `procurement.policy` | requisition, procurement policy, thresholds, blanket agreements, replenishment rules, SoD rules, emergency purchases, procurement disputes, service acceptance | RFQ, bids, award, PO (M49.F49.1–F49.3); ASN, receiving, stock ledger (M50.F50.1–F50.3); vendor catalog (M48) |
| M22–M25 Utilities and gas | `utilities` | utility accounts, connections, meters, readings, tariffs, utility bills, usage reconciliation, allocations, cylinders and their custody/deposit ledger | work orders (M26), incidents (M42), inspections (M61), sustainability reporting (M67), payment execution (M28/M29) |
| M26 Maintenance/assets/vendors | `engineering` | assets, PM schedules, work orders, SLA, inspections, vendor job assignments, work evidence, warranties, scorecards | vendor onboarding/credentials master (M46), room OOO/OOS status (M03), spare-part stock ledger (M50), depreciation (M66) |
| M27 HR/workforce/payroll | `workforce`, `payroll` (separate encrypted schema) | employee, contract, compensation, roster, time, leave, payroll run/results, payslips, WPS files, salary access log | Canadian SIN/CPP/EI/QPP/QPIP/T4 rules (M38.F38.2), jurisdiction rule packs (M44), chef coverage (M47), training (M62) |
| M28 Payment orchestration | `payments` | payment intents/attempts, tokens (references only), refunds, chargebacks, PSP settlements/fees, disbursement payment orders, payout batches, webhook inbox | folio (M08), corporate AR (M20), points (M30), referral payouts policy (M31) |
| M29 Bill-provider gateway | `payments.billpay` | bill providers, billers, bill accounts, inquiries, quotes, bill payment orders, provider status log, receipts, settlements, disputes, adapter certification records | utility bill and payable (M22–M25, M20), bank payment (M28.F28.2) |

**Canonical entity names defined in this file** are listed in §13 for the architecture ERD (`docs/03`).

## 1. Shared payable backbone (Section D) — normative for M20–M27

Every hotel cost uses one backbone. A module may add evidence types but may not skip a stage or create a second payable ledger.

```
request/usage -> evidence -> vendor invoice -> validation -> budget approval -> payable
   -> authorized payment -> provider/bank confirmation -> GL and cost center -> reconciliation/report
```

| Stage | Canonical record | Owner | Exit condition (enforced server-side) |
|---|---|---|---|
| request/usage | `purchase_requisition` (M21), `work_order` (M26), `meter_reading`/`utility_bill` period (M22–M24), `cylinder_custody_entry` (M25), `timesheet` (M27), `recurring_payable_template` (M20) | originating department | cost center + budget line resolved (SF21.1.2 / SF20.4.2) |
| evidence | `goods_receipt` (M50), `service_acceptance` (M21), `work_evidence` (M26), `meter_reading`, `usage_reconciliation`, approved `timesheet` | receiver / engineer / manager | accountable attestation recorded; raw evidence immutable |
| vendor invoice | `supplier_invoice`, `utility_bill`, `payroll_run` (salary "invoice" equivalent) | AP clerk / payroll officer | captured, supplier resolved, unique invoice key passed (SF20.1.3) |
| validation | `invoice_match` (2/3-way or usage-vs-bill) | AP clerk | within tolerance or exception approved (SF21.2.5, SF22.1.6) |
| budget approval | `payable` approval steps per delegation of authority | finance_approver / budget owner | approvals by distinct users (SoD, SF21.3.4) |
| payable | `payable` (open, due date, amount outstanding) | AP | posted to GL control account (AP liability) |
| authorized payment | `payment_order` (M28.F28.2) or `bill_payment_order` (M29) inside a `payout_batch` | payment_releaser (≠ approver) | dual authorization + step-up MFA |
| provider/bank confirmation | `payment_order` status `confirmed` with bank/provider reference; `provider_receipt` | treasury worker | external reference stored; timeout never equals success |
| GL and cost center | `journal_entry` / `journal_line` with cost center | ledger | balanced, source-linked |
| reconciliation/report | `reconciliation_match` to `bank_statement_line` / `provider_settlement` | treasury | matched → `settled`; unmatched → exception queue |

**Approval is not money transfer.** `payable.status = approved` authorizes the hotel's intent to pay. Money moves only when a `payment_order` is released by a `payment_releaser` to an **authorized** bank channel, PSP payout service or contracted bill provider, and is `paid` only on that party's confirmation. MetriStay records, authorizes and reconciles; it never holds funds, pools money or acts as a payment institution (see D-341, `docs/07` CBO/PSP gate).

**Controls (Section D) and where they are specified:** unique supplier invoice key and duplicate detection (SF20.1.3); PO/receipt/invoice match tolerance (SF21.2.5, SF20.1.4); segregation of requisition/approval/payment release (SF21.3.4, SF28.2.2); spend limits (SF20.5.3); FX and tax (SF19.2.4, SF20.5.4); partially paid invoices (SF20.5.4); disputed bills (SF20.1.5, SF21.3.5, SF29.1.9); bank rejection (SF28.2.4, SF27.3.7); attachments and signed audit (SF19.2.3, SF20.1.2).

**Cost-status dashboard (SF20.5.2).** Six separate, non-netted measures per cost center, cost category, supplier and period:

| Measure | Definition (source) |
|---|---|
| planned | open budget line + open commitments (`encumbrance` from approved requisitions/PO/blanket call-offs) + recurring forecast (`expense_forecast`) not yet incurred |
| accrued | incurred, not yet invoiced: goods received not invoiced, estimated utility usage, payroll accrual, prepayment amortization (accrual `journal_entry` with `estimate` label) |
| invoiced | validated `supplier_invoice`/`utility_bill`/`payroll_run` gross not yet approved |
| approved | `payable` approved and not yet confirmed paid (includes `payment_order` states `released`/`submitted`/`pending` shown as "in flight") |
| paid | bank/provider confirmation reference received |
| settled | matched to bank statement line or provider settlement file and GL reconciliation closed |

A cost in `accrued` is labelled **estimate**; `settled` is **reconciled**. No measure is computed by subtraction of another (each is queried from its own ledger state), so a missing source shows as a coverage gap, not a zero (AT-G08.3).

## 2. Common rules for this file

- Money: integer minor units + ISO-4217 (OMR 3 decimals). Every mutating money call requires `Idempotency-Key`.
- All financial ledgers (`journal_line`, `payable` events, `cylinder_custody_entry`, `payroll_result`, `provider_status_log`) are append-only; corrections are reversing entries with `reverses_id`.
- Salary data lives in the `payroll` schema with its own KMS key; nothing outside M27 stores an individual pay amount (F27.4).
- Raw PAN/CVV never touches MetriStay services, logs or databases; only PSP tokens and last-4/brand/expiry month-year (F28.3).
- External timeouts are `pending`, never `failed` or `paid`; the only exit is inquiry, signed callback, settlement file or approved manual evidence (F22.3, SF28.2.4, SF29.1.7).
- Abbreviations in `screens:` — FIN finance web, GM owner/GM, HR HR web, EMP employee mobile, ENG engineering web/mobile, VEN vendor app/portal, PROC procurement, STORE stores/receiving, FD front desk, GST guest web/app, CORP corporate portal, ADM integrations/admin, OPS cross-department exception queue.
- `AT-Gnn.n` numbers below are **proposed** for `docs/09`; `docs/09` owns final numbering.

---

## 3. M19 — GL and cost centers

| Header | Value |
|---|---|
| Purpose | Single authoritative double-entry ledger per property legal entity: chart of accounts, department/cost centers, event-to-GL mapping, immutable journals and reversing corrections, business date vs accounting period, accruals/prepayments, allocation policy, period close, trial balance and P&L / cash-flow / balance-sheet export. |
| Phases | Mapping keys and finance export stub in Phase 2 (M08 night-audit export); full ledger Phase 4; certified close/restatement hardening Phase 6. |
| Release | R1 |
| Bounded context | `finance.ledger` |
| System of record | `legal_entity_book`, `ledger_account`, `cost_center`, `cost_center_hierarchy_version`, `account_mapping_rule`, `accounting_period`, `journal_entry`, `journal_line`, `accrual_schedule`, `prepayment_schedule`, `fx_rate`, `allocation_policy`, `allocation_driver`, `allocation_run`, `period_close_checklist`, `trial_balance_snapshot`, `financial_statement_export`, `source_coverage_status` |
| Dependencies | M01 (property, department, business date), M02 (roles, step-up), M08 (folio/night audit), M20/M21/M22–M28 (posting sources), M32/M65 (reports, lineage), M38/M44 (tax codes, rule packs), M66 (capex/depreciation), `docs/06` (CoA template) |

### F19.1 Chart and periods
Story: As financial controller I maintain one chart of accounts and cost centers per legal entity so every revenue and cost event lands in the right account, department and period.

```yaml
- id: M19.F19.1.SF19.1.1
  name: property and department accounts
  phase: 4
  release: R1
  actors: [financial_controller, finance_clerk, property_admin, auditor]
  screens: [SCR-FIN-gl-accounts, SCR-FIN-cost-centers]
  inputs: [legal_entity_id, account_code, account_name_en, account_name_ar, account_type, normal_balance, is_posting, parent_code, cost_center_code, department_id, effective_from]
  states: [draft, active, blocked_for_posting, inactive]
  api: POST /v1/properties/{pid}/gl/accounts; PATCH /v1/properties/{pid}/gl/accounts/{code}; POST /v1/properties/{pid}/gl/cost-centers; GET /v1/properties/{pid}/gl/cost-center-hierarchy?as_of=
  events: [LedgerAccountCreated, LedgerAccountStatusChanged, CostCenterCreated, CostCenterHierarchyVersioned]
  data: [ledger_account, cost_center, cost_center_hierarchy_version, legal_entity_book]
  rules: ["Account code unique per legal_entity_book", "Only posting accounts accept journal lines; header accounts aggregate", "Revenue and expense lines require a cost center; balance-sheet lines may not carry one unless the account allows it", "An account with postings can be deactivated but never deleted or retyped", "Cost center hierarchy is effective-dated and versioned so historic reports reproduce", "At least two operating cost centers plus an undistributed/admin center are seeded (Section G)"]
  security: financial_controller maintains; finance_clerk read; changes need maker-checker; property-scoped RLS; audit of every change
  failure_cases: [duplicate_code, retype_with_postings, cost_center_missing_on_pnl_line, hierarchy_cycle, deactivate_with_open_balance]
  finance_report_effect: Defines the dimensions of every P&L, balance sheet and department report in M32; hierarchy version is stamped on each report run.
  i18n_a11y: Bilingual EN/AR account names with RTL tree view; keyboard-navigable tree with aria-level; codes remain LTR inside RTL layout.
  acceptance: AC-SF19.1.1 — posting an expense line without cost center is rejected with GL_COST_CENTER_REQUIRED; deactivating an account with balance returns GL_ACCOUNT_HAS_BALANCE; a report run as of last month uses the prior hierarchy version.
  dependency: CoA template decision D-301; M01 department registry.
- id: M19.F19.1.SF19.1.2
  name: account mapping from each revenue/cost event
  phase: 4
  release: R1
  actors: [financial_controller, finance_clerk, ledger_worker]
  screens: [SCR-FIN-account-mapping, SCR-OPS-exception-queue]
  inputs: [source_module, event_type, charge_or_item_code, tax_code, department_id, tender_type, debit_account, credit_account, cost_center_rule, effective_from]
  states: [draft, active, superseded]
  api: PUT /v1/properties/{pid}/gl/mapping-rules/{rule_id}; POST /v1/properties/{pid}/gl/mapping-rules:simulate
  events: [AccountMappingPublished, PostingMappingMissing, SuspensePostingCreated]
  data: [account_mapping_rule, journal_entry, journal_line]
  rules: ["Every posting source event (folio charge, POS sale, GRN, supplier invoice, utility bill, cylinder issue, payroll run, PSP fee, bill-provider fee, points liability) resolves through a mapping rule", "Most specific rule wins by deterministic precedence item > charge code > department > default", "Unmapped event posts to the suspense account and raises an exception; it is never dropped", "Mapping changes are effective-dated and never rewrite past postings", "Simulation shows resulting lines before publish"]
  security: publish requires financial_controller plus second approver; ledger_worker service identity only posts via rules
  failure_cases: [unmapped_event, ambiguous_rule, mapping_to_inactive_account, retroactive_change_attempt]
  finance_report_effect: Guarantees every operational event reaches GL once; suspense balance is a close blocker in SF19.3.1.
  i18n_a11y: Rule table sortable and screen-reader labelled; simulation output readable as a table, not only color.
  acceptance: AC-SF19.1.2 — a fixture of 40 event types from M08, M13, M20, M22–M28, M29, M30, M50 posts balanced entries; one unmapped charge code lands in suspense with PostingMappingMissing within the same business date and clears via reversal plus repost after mapping.
  dependency: Event catalogue of source modules; tax codes from M38.F38.1.
- id: M19.F19.1.SF19.1.3
  name: hotel business date vs accounting period
  phase: 4
  release: R1
  actors: [financial_controller, night_auditor, night_audit_worker]
  screens: [SCR-FIN-periods, SCR-FIN-night-audit-summary]
  inputs: [fiscal_calendar_id, period_code, start_date, end_date, business_date, posting_date_rule]
  states: [future, open, soft_closed, closed, locked]
  api: POST /v1/properties/{pid}/gl/fiscal-calendars; POST /v1/properties/{pid}/gl/periods/{period}:transition
  events: [AccountingPeriodOpened, AccountingPeriodSoftClosed, AccountingPeriodClosed, AccountingPeriodLocked]
  data: [accounting_period, journal_entry]
  rules: ["accounting_date derives from the hotel business_date (M01), not wall-clock UTC", "A transaction after midnight but before night audit belongs to the prior business date", "soft_closed accepts only close-adjustment journals by finance roles", "closed and locked periods reject postings; late source events post to the first open period with original business_date retained", "Business date roll and period close are separate actions"]
  security: transition rights by role; lock requires financial_controller with step-up MFA
  failure_cases: [posting_into_closed_period, business_date_not_rolled, overlapping_periods, timezone_misconfiguration]
  finance_report_effect: Daily flash uses business date; P&L uses accounting period; both are shown with their basis label.
  i18n_a11y: Dates rendered in property locale with Gregorian primary and optional Hijri display; period status conveyed by text plus icon.
  acceptance: AC-SF19.1.3 — a POS sale at 00:40 before night audit posts to the prior business date; a folio charge arriving after period close posts to the next open period with original business_date and a late-posting flag.
  dependency: M01 business date and night audit (M08, M60.F60.1).
- id: M19.F19.1.SF19.1.4
  name: accrual and prepayment schedule
  phase: 4
  release: R1
  actors: [finance_clerk, financial_controller, ledger_worker]
  screens: [SCR-FIN-accruals, SCR-FIN-prepayments]
  inputs: [source_type, source_id, amount, currency, account, cost_center, start_period, end_period, method, auto_reverse]
  states: [draft, scheduled, posting, completed, cancelled]
  api: POST /v1/properties/{pid}/gl/accrual-schedules; POST /v1/properties/{pid}/gl/prepayment-schedules; POST /v1/properties/{pid}/gl/accrual-schedules/{id}:cancel
  events: [AccrualPosted, AccrualReversed, PrepaymentAmortized]
  data: [accrual_schedule, prepayment_schedule, journal_entry]
  rules: ["Period-end accruals auto-reverse on the first day of the next period", "System accrual sources are goods-received-not-invoiced (SF20.5.5), estimated utility usage (SF22.2.3), payroll accrual (SF27.3.8) and cylinder consumption (SF25.1.6)", "Prepayments (insurance, rent, subscriptions, annual contracts) amortize straight-line or by schedule and are linked to the payable", "Accrual lines carry estimate label and source link", "Cancelling a schedule stops future postings only"]
  security: finance roles; manual accruals above threshold need finance_approver
  failure_cases: [double_accrual_same_source, reversal_missing, prepayment_without_invoice, schedule_past_locked_period]
  finance_report_effect: Populates the accrued measure of the cost-status dashboard and makes monthly P&L complete before invoices arrive.
  i18n_a11y: Schedule grid readable by screen reader with period headers; amounts formatted per currency minor units.
  acceptance: AC-SF19.1.4 — an unbilled electricity estimate of OMR 1,250.000 accrues on 31 Aug and reverses on 1 Sep; when the actual bill posts, net September expense equals the bill; a second accrual for the same source is rejected.
  dependency: SF20.5.5, SF22.2.3, SF27.3.8.
- id: M19.F19.1.SF19.1.5
  name: trial-balance validation and locked period
  phase: 4
  release: R1
  actors: [financial_controller, auditor, ledger_worker]
  screens: [SCR-FIN-trial-balance, SCR-FIN-period-close]
  inputs: [period, legal_entity_id, currency_basis]
  states: [not_run, running, balanced, out_of_balance, subledger_mismatch, locked]
  api: POST /v1/properties/{pid}/gl/trial-balances; GET /v1/properties/{pid}/gl/trial-balances/{id}
  events: [TrialBalanceProduced, SubledgerMismatchDetected, AccountingPeriodLocked]
  data: [trial_balance_snapshot, journal_line, accounting_period]
  rules: ["Sum of debits equals sum of credits per legal entity in base and each transaction currency", "Control accounts reconcile to subledgers (guest ledger, city ledger/AR, AP, inventory, payroll liabilities, PSP clearing, bill-pay clearing, cylinder deposits)", "Lock requires balanced TB, zero suspense or approved carry-forward, and completed close checklist", "Snapshot is immutable and hash-stamped"]
  security: lock by financial_controller with step-up MFA; auditor read-only access to snapshots
  failure_cases: [out_of_balance, subledger_mismatch, suspense_not_cleared, concurrent_posting_during_lock]
  finance_report_effect: Locked TB is the only source for certified period statements; unlocked period reports show draft.
  i18n_a11y: TB table exportable and navigable with row/column headers; mismatch conveyed in text.
  acceptance: AC-SF19.1.5 — injecting a subledger mismatch of one cent in AP blocks lock with SubledgerMismatchDetected; after correction TB locks and later posting to that period is rejected.
  dependency: SF19.3.1 checklist; subledgers M08, M20, M27, M28, M29, M50.
```

### F19.2 Journal ledger
Story: As auditor I can trust that every posted amount is balanced, immutable, source-linked and correctly converted and taxed.

```yaml
- id: M19.F19.2.SF19.2.1
  name: debit/credit invariant
  phase: 4
  release: R1
  actors: [ledger_worker, finance_clerk]
  screens: [SCR-FIN-journal-entry]
  inputs: [journal_entry_id, lines, currency, base_currency, rate_id]
  states: [draft, validated, posted, rejected]
  api: POST /v1/properties/{pid}/gl/journal-entries (Idempotency-Key)
  events: [JournalEntryPosted, JournalEntryRejected]
  data: [journal_entry, journal_line]
  rules: ["At least two lines; each line has exactly one of debit or credit, positive integer minor units", "Entry balances in transaction currency and base currency; rounding difference only to a configured FX rounding account", "Database constraint validates balance at commit, not only application code", "Idempotency key plus source reference prevents double posting"]
  security: only ledger service role inserts; no direct SQL write grants to app users
  failure_cases: [unbalanced_entry, zero_amount_line, duplicate_idempotency_key, currency_mismatch]
  finance_report_effect: Guarantees TB balance by construction.
  i18n_a11y: Entry form shows running difference as text; totals announced to screen readers.
  acceptance: AC-SF19.2.1 — an unbalanced entry is rejected at database commit even if the API check is bypassed in test; replaying the same Idempotency-Key returns the original entry.
  dependency: ADR on ledger constraints in docs/03.
- id: M19.F19.2.SF19.2.2
  name: immutable posted entry and reversing correction
  phase: 4
  release: R1
  actors: [finance_clerk, finance_approver, auditor]
  screens: [SCR-FIN-journal-entry, SCR-FIN-journal-history]
  inputs: [journal_entry_id, reversal_reason, reversal_date, replacement_lines]
  states: [posted, reversed, partially_superseded]
  api: POST /v1/properties/{pid}/gl/journal-entries/{id}:reverse
  events: [JournalEntryReversed, JournalEntryCorrected]
  data: [journal_entry, journal_line, ledger_hash_chain]
  rules: ["No UPDATE or DELETE on posted journal rows", "Correction equals reversal entry linked by reverses_id plus new entry; reason mandatory", "An entry can be reversed once; reversing a reversal is rejected", "Hash chain per legal entity detects tampering", "Reversal date defaults to the first open period"]
  security: reversal needs maker-checker above threshold; all reversals audited with actor and reason
  failure_cases: [double_reversal, reversal_into_locked_period, tamper_detected]
  finance_report_effect: History preserved; restated reports show original and correction lines.
  i18n_a11y: Reversal reason field bilingual; linked entries navigable by keyboard.
  acceptance: AC-SF19.2.2 — attempting UPDATE on a posted line fails; reversing twice returns GL_ALREADY_REVERSED; hash verification job flags a manually altered test row.
  dependency: SF19.2.1.
- id: M19.F19.2.SF19.2.3
  name: source-document link
  phase: 4
  release: R1
  actors: [finance_clerk, auditor, gm, owner]
  screens: [SCR-FIN-journal-entry, SCR-GM-drill-through]
  inputs: [source_type, source_id, evidence_refs, correlation_id]
  states: [linked, evidence_missing]
  api: GET /v1/properties/{pid}/gl/journal-entries/{id}/sources
  events: [JournalSourceLinked]
  data: [journal_entry, document_attachment, journal_source_link]
  rules: ["Every system entry carries source_type and source_id (folio, invoice, payable, payment_order, bill_payment_order, payroll_run, utility_bill, goods_receipt, allocation_run)", "Manual entries require an attachment or written rationale", "Drill-through respects the viewer's permissions; payroll sources expose run-level, not employee-level, detail to non-HR roles", "Attachments are stored immutable with SHA-256"]
  security: evidence access follows source-module scopes; field-level masking for salary and identity
  failure_cases: [source_deleted_or_purged, permission_denied_on_source, attachment_missing]
  finance_report_effect: Enables GM drill from KPI to evidence (AT-G08).
  i18n_a11y: Drill-through breadcrumbs announced; attachment viewer accessible with alt text.
  acceptance: AC-SF19.2.3 — from department P&L utilities line, a user drills to allocation_run, utility_bill, meter_reading and PDF; a GM drilling payroll expense sees payroll_run total only.
  dependency: M32 drill-through, F27.4.
- id: M19.F19.2.SF19.2.4
  name: FX/tax lines
  phase: 4
  release: R1
  actors: [finance_clerk, financial_controller, ledger_worker]
  screens: [SCR-FIN-fx-rates, SCR-FIN-journal-entry]
  inputs: [transaction_currency, base_currency, rate_source, rate_date, rate, tax_code, tax_amount, taxable_amount]
  states: [rate_pending, rate_applied, revalued]
  api: POST /v1/properties/{pid}/gl/fx-rates; POST /v1/properties/{pid}/gl/fx-revaluations
  events: [FxRateRecorded, FxRevaluationPosted, RealizedFxPosted]
  data: [fx_rate, journal_line, tax_line_ref]
  rules: ["Rate source and date stored on each line; no silent default rate", "Realized FX difference posts when a foreign-currency payable or receivable is settled", "Period-end unrealized revaluation posts and auto-reverses", "Tax lines use tax_code from M38 with effective date; ledger does not compute tax rates itself", "Recoverable versus non-recoverable input tax go to separate accounts"]
  security: rate maintenance by finance roles; automated feed credentials in vault
  failure_cases: [missing_rate, stale_rate, tax_code_unverified, rounding_difference]
  finance_report_effect: Correct base-currency P&L and tax control accounts for M38 returns.
  i18n_a11y: Currency codes and minor units per ISO-4217; numerals per locale with Western digits option.
  acceptance: AC-SF19.2.4 — a EUR supplier invoice booked at 0.420 and paid at 0.425 OMR posts realized FX loss exactly; a tax code with rule-pack status unverified blocks automated tax return export but allows posting with a warning.
  dependency: D-303; M38.F38.1; M44 rule-pack status.
- id: M19.F19.2.SF19.2.5
  name: approvals and export
  phase: 4
  release: R1
  actors: [finance_clerk, finance_approver, financial_controller, integration_admin]
  screens: [SCR-FIN-manual-journal-approval, SCR-FIN-gl-export]
  inputs: [journal_entry_id, approver_id, export_target, period, format]
  states: [draft, submitted, approved, rejected, posted, exported, export_failed]
  api: POST /v1/properties/{pid}/gl/journal-entries/{id}:submit; POST /v1/properties/{pid}/gl/journal-entries/{id}:approve; POST /v1/properties/{pid}/gl/exports (Idempotency-Key)
  events: [ManualJournalSubmitted, ManualJournalApproved, GlExportCompleted, GlExportFailed]
  data: [journal_entry, approval_step, gl_export_batch]
  rules: ["Manual and recurring journals require maker-checker; maker cannot approve", "Export batch is idempotent and records exported entry range; re-export produces identical file hash", "External GL (if any) is downstream only; MetriStay remains system of record for sub-ledgers"]
  security: step-up MFA for approval; export files encrypted at rest and access-logged
  failure_cases: [self_approval, export_target_unreachable, partial_export, duplicate_export]
  finance_report_effect: Only approved journals affect reports; export status visible on close checklist.
  i18n_a11y: Approval inbox accessible; export formats CSV/XLSX with UTF-8 BOM for Arabic.
  acceptance: AC-SF19.2.5 — maker self-approval returns SOD_VIOLATION; exporting August twice yields one batch and identical hash.
  dependency: D-305 external accounting system.
```

### F19.3 Period close and allocation (added — Section C "period close, allocation policy, P&L/cashflow/balance-sheet export")
Story: As financial controller I close a month with a checklist, allocate shared costs by a versioned policy and publish statements whose estimate/actual status is explicit.

```yaml
- id: M19.F19.3.SF19.3.1
  name: period close checklist and subledger cutoff
  phase: 4
  release: R1
  actors: [financial_controller, finance_clerk, ap_clerk, ar_clerk, payroll_officer]
  screens: [SCR-FIN-period-close]
  inputs: [period, checklist_template_id, task_id, completion_evidence]
  states: [not_started, in_progress, blocked, ready_to_close, closed]
  api: GET /v1/properties/{pid}/gl/periods/{period}/close-checklist; POST /v1/properties/{pid}/gl/periods/{period}/close-checklist/{task}:complete
  events: [CloseTaskCompleted, CloseBlocked, AccountingPeriodClosed]
  data: [period_close_checklist, accounting_period, source_coverage_status]
  rules: ["Checklist includes night audits complete, cash shifts closed, GRNI accrual, utility accruals, payroll accrual, bank reconciliation, PSP and bill-provider settlement, allocation run, suspense cleared, TB balanced", "Automatic tasks are verified by the system, not ticked manually", "Subledger cutoff freezes posting into the period for non-finance sources"]
  security: tasks assigned by role; completion audited
  failure_cases: [task_evidence_missing, source_feed_stale, night_audit_missing]
  finance_report_effect: Close status drives certified vs draft labelling in M32.
  i18n_a11y: Checklist is an accessible list with status text; progress not color-only.
  acceptance: AC-SF19.3.1 — with one missing night audit and an unreconciled bank line, close shows two blockers and cannot transition to closed.
  dependency: M60.F60.1, SF20.3.2, SF27.3.8.
- id: M19.F19.3.SF19.3.2
  name: allocation policy and driver versions
  phase: 4
  release: R1
  actors: [financial_controller, gm, owner]
  screens: [SCR-FIN-allocation-policy]
  inputs: [policy_version, source_account_or_pool, driver_type, driver_source, target_cost_centers, effective_from]
  states: [draft, approved, active, superseded]
  api: POST /v1/properties/{pid}/gl/allocation-policies; POST /v1/properties/{pid}/gl/allocation-policies/{id}:approve
  events: [AllocationPolicyApproved]
  data: [allocation_policy, allocation_driver]
  rules: ["Driver types include submeter share, floor area, occupied room nights, covers, labor hours, kg laundry, fixed percentage", "Metered actual drivers take precedence over estimated drivers; estimated drivers are labelled", "Percentages sum to 100 per pool", "Policy version is stamped on every allocation run and report"]
  security: approval by financial_controller; owner/gm read
  failure_cases: [percent_not_100, driver_source_missing, overlapping_versions]
  finance_report_effect: Determines shared energy, labor and overhead in department P&L (M32.F32.3 SF32.3.1).
  i18n_a11y: Driver explanation text in EN/AR; charts have table alternative.
  acceptance: AC-SF19.3.2 — a policy splitting electricity 45/30/15/10 fails approval if it sums to 99; a submeter-based driver overrides area share where submeter data exists.
  dependency: D-304; F22.1 submeters.
- id: M19.F19.3.SF19.3.3
  name: allocation run, review and reversal
  phase: 4
  release: R1
  actors: [finance_clerk, financial_controller, ledger_worker]
  screens: [SCR-FIN-allocation-run]
  inputs: [period, policy_version, pools]
  states: [simulated, reviewed, posted, reversed]
  api: POST /v1/properties/{pid}/gl/allocation-runs:simulate; POST /v1/properties/{pid}/gl/allocation-runs/{id}:post (Idempotency-Key); POST /v1/properties/{pid}/gl/allocation-runs/{id}:reverse
  events: [AllocationRunPosted, AllocationRunReversed]
  data: [allocation_run, journal_entry]
  rules: ["One posted run per pool per period; rerun requires reversal first", "Run is reproducible from stored driver snapshot", "Allocation lines net to zero for the pool"]
  security: post by financial_controller; simulation by finance_clerk
  failure_cases: [driver_data_changed_after_simulation, duplicate_run, locked_period]
  finance_report_effect: Department contribution and property operating result include allocated shared costs with version reference.
  i18n_a11y: Simulation diff table accessible.
  acceptance: AC-SF19.3.3 — posting the same run twice returns the first run; reversal followed by rerun yields identical totals when drivers unchanged.
  dependency: SF19.3.2.
- id: M19.F19.3.SF19.3.4
  name: financial statements and export
  phase: 4
  release: R1
  actors: [financial_controller, owner, gm, auditor]
  screens: [SCR-FIN-statements, SCR-GM-home]
  inputs: [period, statement_type, comparison_basis, format]
  states: [draft, certified, superseded]
  api: GET /v1/properties/{pid}/gl/statements?type=pnl|balance_sheet|cash_flow&period=; POST /v1/properties/{pid}/gl/statement-exports
  events: [FinancialStatementPublished]
  data: [financial_statement_export, trial_balance_snapshot]
  rules: ["P&L, balance sheet and indirect cash flow generated from locked TB for certified output", "Unlocked period statements carry DRAFT and estimate share", "Statement layout maps accounts via versioned report mapping (USALI-style departmental layout per D-301)"]
  security: owner/gm see property totals; salary detail never itemized per employee
  failure_cases: [period_not_locked, mapping_gap, currency_basis_missing]
  finance_report_effect: Primary financial outputs; feeds M32 GOP/net-profit bridge and M66 owner statement.
  i18n_a11y: PDF/XLSX exports tagged for accessibility; Arabic RTL layout for AR statements.
  acceptance: AC-SF19.3.4 — the August P&L export from a locked period is marked certified and reproduces byte-identical on rerun; an open period export shows DRAFT and estimate percentage.
  dependency: SF19.1.5, D-301.
- id: M19.F19.3.SF19.3.5
  name: period reopen and restatement
  phase: 4
  release: R1
  actors: [financial_controller, owner, auditor]
  screens: [SCR-FIN-period-close]
  inputs: [period, reason, approver_ids]
  states: [locked, reopen_requested, reopened, relocked]
  api: POST /v1/properties/{pid}/gl/periods/{period}:reopen-request; POST /v1/properties/{pid}/gl/periods/{period}:reopen-approve
  events: [PeriodReopenRequested, PeriodReopened, StatementRestated]
  data: [accounting_period, financial_statement_export]
  rules: ["Reopen requires two approvers including financial_controller and a reason", "Previously certified statements are marked superseded, never overwritten", "Subscribers of scheduled reports are notified of restatement"]
  security: step-up MFA; immutable audit
  failure_cases: [single_approver, reopen_after_external_filing]
  finance_report_effect: Restatement trail visible in M32.F32.3 SF32.3.5.
  i18n_a11y: Notification text bilingual.
  acceptance: AC-SF19.3.5 — reopen with one approver fails; after approved reopen and correction, the prior certified P&L remains retrievable as superseded.
  dependency: M65.F65.2.
- id: M19.F19.3.SF19.3.6
  name: source coverage and estimate/actual status
  phase: 4
  release: R1
  actors: [gm, owner, financial_controller]
  screens: [SCR-GM-home, SCR-FIN-cost-status-dashboard]
  inputs: [period, cost_category, expected_sources]
  states: [complete, estimate, missing_source]
  api: GET /v1/properties/{pid}/gl/source-coverage?period=
  events: [SourceCoverageChanged]
  data: [source_coverage_status]
  rules: ["Each expected cost source (electricity, water, gas, cylinders, payroll, maintenance, PSP fees, rent) has a coverage state per period", "A missing source is shown as incomplete estimate, never certified actual (Section G step 8)", "Coverage percentage shown beside profit figures"]
  security: property-scoped read
  failure_cases: [expected_source_not_configured, stale_feed]
  finance_report_effect: Profit figures carry estimate vs reconciled vs certified label (README §3.6).
  i18n_a11y: Status uses text labels and patterns, not color alone.
  acceptance: AC-SF19.3.6 — with no water bill imported for September, the property result shows water as missing_source and the report header reads incomplete estimate.
  dependency: M32.F32.3 SF32.3.4, M65.F65.1 SF65.1.5.
```

**M19 key invariants:** balanced entries at DB commit; posted lines immutable; every system entry source-linked; no posting to closed/locked periods; suspense is a close blocker; allocations versioned and reproducible; missing source never reported as actual.

**M19 module acceptance (Section G):** AT-G08.1 month-end for the Section G hotel shows balanced TB, locked period and certified P&L with two staff cost centers; AT-G08.2 GM drills from department contribution to journal, allocation run and source document; AT-G08.3 withholding the water bill makes the result "incomplete estimate"; AT-G20.1 duplicate provider webhook and repeated invoice produce exactly one journal each.

**M19 open decisions**

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-301 | Chart-of-accounts template and statement layout (USALI 11th edition departmental vs local statutory layout per country) | financial_controller (pilot hotel) | USALI-style departmental management layout plus a statutory mapping per legal entity; both driven from one ledger. |
| D-302 | Fiscal calendar (calendar months vs 4-4-5) and fiscal year start | financial_controller | Calendar months, fiscal year = calendar year. |
| D-303 | Functional currency and FX rate source per legal entity | financial_controller | Functional = local currency of the legal entity; rates entered daily by finance or from a central-bank feed once licensed. |
| D-304 | Shared-cost allocation drivers (energy, water, labor, overhead) | financial_controller + gm | Submeter where available, else floor area; labor by timesheet department; overhead left undistributed per USALI. |
| D-305 | External accounting system (none, Xero, QuickBooks, SAP, local package) and export format | financial_controller + integration_admin | MetriStay is full GL; optional CSV journal export per batch. |

---
## 4. M20 — AP/AR and treasury

| Header | Value |
|---|---|
| Purpose | Operate the shared payable backbone (§1): supplier invoices/credits/debits, duplicate detection, 2/3-way match, approvals, due dates and payment status; corporate AR, aging, credit and collections; bank import and reconciliation; budgets, commitments, cash and expense forecasting; planned/accrued/invoiced/approved/paid/settled dashboard. |
| Phases | Corporate credit account/PO basics Phase 3 (with M10/M12); AP/AR/treasury/budget Phase 4; PSP/bill-provider settlement matching Phase 5; hardening Phase 6. |
| Release | R1 |
| Bounded context | `finance.payables`, `finance.receivables`, `finance.treasury` |
| System of record | `supplier_payment_profile`, `supplier_invoice`, `supplier_invoice_line`, `supplier_credit_note`, `invoice_match`, `payable`, `payable_hold`, `payable_approval_step`, `recurring_payable_template`, `corporate_credit_account`, `ar_invoice`, `ar_adjustment`, `ar_receipt`, `receipt_allocation`, `collection_case`, `write_off_request`, `bank_account`, `bank_statement`, `bank_statement_line`, `reconciliation_match`, `cash_forecast`, `budget_version`, `budget_line`, `encumbrance`, `expense_forecast`, `delegation_of_authority`, `cost_status_snapshot` |
| Dependencies | M19 (posting), M21/M49/M50 (PO, receipts), M22–M27 (payable sources), M28 (payment orders, PSP settlement), M29 (bill payment orders), M10/M11/M12 (corporate accounts, event billing), M08 (city-ledger transfer), M46 (vendor identity/bank verification), M38 (tax, e-invoice), M60 (duplicate detection cases) |

### F20.1 Supplier payables
Story: As AP clerk I capture every supplier bill once, match it to evidence, route it for approval and see its payment status through settlement.

```yaml
- id: M20.F20.1.SF20.1.1
  name: supplier onboarding and payment details
  phase: 4
  release: R1
  actors: [ap_clerk, finance_approver, vendor_admin, compliance_officer]
  screens: [SCR-FIN-supplier-payment-profile, SCR-VEN-bank-details]
  inputs: [vendor_id, legal_entity_id, payment_terms, currency, bank_account_iban_or_local, bank_proof_document, tax_registration_ref, withholding_flag]
  states: [pending_verification, verified, change_pending, suspended]
  api: POST /v1/properties/{pid}/suppliers/{vendor_id}/payment-profile; POST /v1/properties/{pid}/suppliers/{vendor_id}/payment-profile/bank-change-requests
  events: [SupplierPaymentProfileVerified, SupplierBankChangeRequested, SupplierBankChangeApproved]
  data: [supplier_payment_profile, vendor, document_attachment]
  rules: ["Vendor legal identity and sanctions checks come from M46; M20 holds only payment terms and bank instructions", "Bank detail change requires out-of-band callback verification and a second approver; payments to the new account are held 72h configurable", "Payee name must match verified legal entity", "Withholding and tax registration flags come from M38/M44 rule packs"]
  security: bank fields encrypted and masked (last 4); change visible to vendor only for its own profile; step-up MFA
  failure_cases: [bank_change_fraud_attempt, name_mismatch, unverified_vendor, expired_tax_document]
  finance_report_effect: Payment eligibility gate; suspended profile blocks payment orders.
  i18n_a11y: Bilingual form; IBAN input formatted in LTR within RTL page; error messages specific and announced.
  acceptance: AC-SF20.1.1 — a vendor-submitted bank change is held until callback plus second approval; a payment order created during the hold is blocked with SUPPLIER_BANK_CHANGE_PENDING.
  dependency: M46.F46.1 SF46.1.4; bank payout channel D-341.
- id: M20.F20.1.SF20.1.2
  name: invoice capture/manual/import
  phase: 4
  release: R1
  actors: [ap_clerk, vendor_user, invoice_ocr_worker]
  screens: [SCR-FIN-ap-invoice-inbox, SCR-FIN-ap-invoice-detail, SCR-VEN-invoice-submit]
  inputs: [vendor_id, invoice_number, invoice_date, currency, lines, tax_lines, po_id, pdf_or_einvoice_file, source_channel]
  states: [received, ocr_draft, captured, validation_failed, validated]
  api: POST /v1/properties/{pid}/supplier-invoices (Idempotency-Key); POST /v1/properties/{pid}/supplier-invoices:import; POST /v1/vendor/invoices
  events: [SupplierInvoiceReceived, SupplierInvoiceCaptured, SupplierInvoiceValidationFailed]
  data: [supplier_invoice, supplier_invoice_line, document_attachment]
  rules: ["Channels: vendor portal/app, email inbox, OCR, structured e-invoice (where jurisdiction supports), manual entry", "OCR output is a draft; a human confirms low-confidence fields", "Original file stored immutable with hash; e-invoice XML retained per M44 retention", "Tax lines validated against M38 tax codes; debit notes captured as separate document type"]
  security: vendor sees only own invoices; malware scan on upload; AP role scope by property
  failure_cases: [ocr_low_confidence, unreadable_file, unknown_vendor, currency_not_allowed, einvoice_signature_invalid]
  finance_report_effect: Moves amount into invoiced measure once validated.
  i18n_a11y: Side-by-side PDF and fields; Arabic invoice text supported by OCR where engine allows; keyboard field navigation.
  acceptance: AC-SF20.1.2 — an OCR-captured invoice with confidence below threshold cannot be validated until a user confirms flagged fields; the stored file hash matches on retrieval.
  dependency: D-311 OCR provider; D-312 e-invoicing per jurisdiction.
- id: M20.F20.1.SF20.1.3
  name: duplicate detection
  phase: 4
  release: R1
  actors: [ap_clerk, finance_approver, ap_worker]
  screens: [SCR-FIN-ap-invoice-detail, SCR-OPS-exception-queue]
  inputs: [vendor_id, normalized_invoice_number, invoice_date, gross_amount, currency, file_hash]
  states: [unique, suspected_duplicate, confirmed_duplicate, cleared]
  api: POST /v1/properties/{pid}/supplier-invoices/{id}:duplicate-review
  events: [SupplierInvoiceDuplicateSuspected, SupplierInvoiceDuplicateCleared]
  data: [supplier_invoice, duplicate_check_result]
  rules: ["Hard unique key (legal_entity, vendor_id, normalized_invoice_number) enforced by database", "Soft checks flag same vendor+amount+date window, same file hash, or same amount across vendors sharing bank account", "Clearing a soft duplicate needs a reason and a second person", "Confirmed duplicates are voided, never paid; case raised to M60 where pattern repeats"]
  security: reviewer cannot be the capturer
  failure_cases: [invoice_number_formatting_variant, credit_note_mistaken_as_duplicate, cross_property_duplicate]
  finance_report_effect: Prevents double expense and double payment.
  i18n_a11y: Duplicate comparison view accessible as table.
  acceptance: AC-SF20.1.3 — re-submitting invoice INV-001 as inv 001 from the same vendor is rejected; a same-amount same-day invoice with different number is flagged and needs a second reviewer (AT-G20.4).
  dependency: M60.F60.2 SF60.2.1.
- id: M20.F20.1.SF20.1.4
  name: PO/receipt/service matching
  phase: 4
  release: R1
  actors: [ap_clerk, procurement_officer, receiver, ap_worker]
  screens: [SCR-FIN-match-workbench]
  inputs: [supplier_invoice_id, po_id, po_version, goods_receipt_ids, service_acceptance_ids, tolerance_profile]
  states: [unmatched, matched, within_tolerance, exception, overridden]
  api: POST /v1/properties/{pid}/supplier-invoices/{id}:match; POST /v1/properties/{pid}/invoice-matches/{id}:override
  events: [InvoiceMatched, InvoiceMatchException, InvoiceMatchOverridden]
  data: [invoice_match, purchase_order, goods_receipt, service_acceptance]
  rules: ["Two-way (PO-invoice) for approved non-stock services with acceptance; three-way (PO-receipt-invoice) for goods", "Match on quantity accepted, unit price, tax and freight per line; quantity invoiced cannot exceed accepted minus previously invoiced", "Tolerance from SF21.2.5; exceptions route to procurement and budget owner", "A PO alone never authorizes payment (M49 SF49.3.7)", "Override needs reason and finance_approver distinct from capturer"]
  security: procurement and AP scopes; overrides audited
  failure_cases: [receipt_missing, price_variance, quantity_over_receipt, po_version_changed, partial_receipt]
  finance_report_effect: Matched quantity clears GRNI accrual and moves to invoiced/approved.
  i18n_a11y: Variance column uses text and sign; screen readers announce exception reason.
  acceptance: AC-SF20.1.4 — invoice for 120 kg against 110 kg accepted matches 110 and raises quantity exception for 10 kg; matching the same receipt twice is rejected (AT-G18.4).
  dependency: M49.F49.3 SF49.3.6; M50.F50.2 SF50.2.7.
- id: M20.F20.1.SF20.1.5
  name: hold/dispute/approval
  phase: 4
  release: R1
  actors: [ap_clerk, finance_approver, gm, procurement_approver, vendor_user]
  screens: [SCR-FIN-payable-approval, SCR-GM-approvals, SCR-VEN-invoice-status]
  inputs: [payable_id, hold_reason, dispute_reason, disputed_amount, approval_decision, comment]
  states: [pending_approval, approved, rejected, on_hold, disputed, partially_disputed]
  api: POST /v1/properties/{pid}/payables/{id}:approve; POST /v1/properties/{pid}/payables/{id}:hold; POST /v1/properties/{pid}/payables/{id}:dispute
  events: [PayableCreated, PayableApproved, PayableHeld, PayableDisputed, PayableDisputeResolved]
  data: [payable, payable_hold, payable_approval_step, procurement_dispute]
  rules: ["Payable is created from a validated, matched or approved-exception invoice", "Approval chain from delegation_of_authority by amount, category and cost center (SF20.5.3)", "Approver cannot be requester, capturer or payment releaser for the same payable", "Disputed amount is excluded from payment; undisputed portion may be paid", "Holds stop payment selection but not accrual"]
  security: step-up MFA above threshold; vendor sees status and dispute messages only
  failure_cases: [approver_out_of_office, delegation_expired, dispute_without_evidence, approval_after_payment_released]
  finance_report_effect: Moves amount from invoiced to approved; disputed amounts tracked separately in dashboard.
  i18n_a11y: Mobile approval card with large tap targets and screen-reader summary of amount, vendor and evidence.
  acceptance: AC-SF20.1.5 — an OMR 900 invoice partly disputed for OMR 100 allows payment of OMR 800 only; approver equal to capturer is rejected with SOD_VIOLATION.
  dependency: SF20.5.3, SF21.3.4; D-306.
- id: M20.F20.1.SF20.1.6
  name: payment batch/status/reconciliation
  phase: 4
  release: R1
  actors: [ap_clerk, payment_releaser, finance_approver, treasury_worker]
  screens: [SCR-FIN-payment-batch, SCR-FIN-cost-status-dashboard]
  inputs: [payable_ids, value_date, bank_account_id, channel, batch_id]
  states: [proposed, approved, released, submitted, partially_confirmed, confirmed, settled, rejected]
  api: POST /v1/properties/{pid}/payout-batches (Idempotency-Key); GET /v1/properties/{pid}/payables/{id}/payment-status
  events: [PayoutBatchProposed, PaymentOrderConfirmed, PayableSettled, PaymentOrderRejected]
  data: [payout_batch, payment_order, payable, reconciliation_match]
  rules: ["Payment selection only from approved, unheld, undisputed payables with verified payment profile", "Batch execution is delegated to M28.F28.2; M20 tracks payable status from payment_order events", "Payable becomes paid on bank/provider confirmation and settled when matched to bank statement line", "Bank rejection reopens the payable to approved with reason"]
  security: releaser distinct from approver; batches over limit need dual release
  failure_cases: [bank_rejection, partial_batch_failure, payment_to_suspended_vendor, value_date_holiday]
  finance_report_effect: Moves approved to paid to settled; credits AP control and debits bank clearing then bank.
  i18n_a11y: Status timeline with text for each stage.
  acceptance: AC-SF20.1.6 — a batch of five payables where the bank rejects one shows four paid, one approved-with-rejection; matching the bank statement moves the four to settled (AT-G20.7).
  dependency: M28.F28.2; D-307, D-341.
```

### F20.2 Corporate receivables
Story: As AR clerk I bill corporate accounts correctly, collect and allocate receipts and control credit exposure.

```yaml
- id: M20.F20.2.SF20.2.1
  name: credit account and PO
  phase: 3
  release: R1
  actors: [ar_clerk, sales_manager, financial_controller, corporate_admin]
  screens: [SCR-FIN-ar-accounts, SCR-CORP-billing-profile]
  inputs: [corporate_account_id, legal_entity_billing_details, tax_registration, credit_limit, payment_terms, po_required, customer_po_number, po_amount, po_expiry]
  states: [applied, approved, active, on_stop, closed]
  api: POST /v1/properties/{pid}/ar/credit-accounts; POST /v1/properties/{pid}/ar/credit-accounts/{id}/purchase-orders
  events: [CorporateCreditAccountApproved, CorporateAccountOnStop, CustomerPoRegistered]
  data: [corporate_credit_account, customer_purchase_order, corporate_account]
  rules: ["Credit account links to M10 corporate account; one billing legal entity per account", "Direct-bill (city ledger) transfer from folio allowed only for active accounts within available credit", "Customer PO required flag blocks invoicing without a valid PO number and remaining PO amount", "Credit limit approval by financial_controller"]
  security: corporate users see their own account statements only; hotel roles property-scoped
  failure_cases: [credit_limit_exceeded, po_expired, account_on_stop, missing_tax_registration]
  finance_report_effect: Determines city-ledger exposure; drives AR aging.
  i18n_a11y: Corporate portal pages bilingual and WCAG 2.2 AA.
  acceptance: AC-SF20.2.1 — transfer of an event folio exceeding available credit is blocked and routed for approval; an invoice for a PO-required account without PO is rejected.
  dependency: M10.F10.x corporate accounts; M08 folio transfer.
- id: M20.F20.2.SF20.2.2
  name: invoice and adjustment
  phase: 4
  release: R1
  actors: [ar_clerk, finance_approver, corporate_approver]
  screens: [SCR-FIN-ar-invoice, SCR-CORP-invoices]
  inputs: [corporate_account_id, folio_transfer_ids, event_id, lines, tax_lines, adjustment_reason, credit_note_ref]
  states: [draft, issued, partially_paid, paid, credited, disputed]
  api: POST /v1/properties/{pid}/ar/invoices (Idempotency-Key); POST /v1/properties/{pid}/ar/invoices/{id}/adjustments
  events: [ArInvoiceIssued, ArAdjustmentPosted, ArInvoiceDisputed]
  data: [ar_invoice, ar_adjustment, tax_invoice_ref]
  rules: ["Tax invoice numbering and format come from M38 per jurisdiction", "Issued invoices are immutable; corrections via credit note or adjustment with approval", "Event master and individual billing reconcile to M12 event totals", "Dispute on a line excludes that amount from dunning"]
  security: adjustment approval above threshold; audit
  failure_cases: [tax_rule_unverified, duplicate_invoice_for_folio, adjustment_exceeds_invoice]
  finance_report_effect: Revenue already posted from folio; AR invoice moves balance from guest ledger to city ledger.
  i18n_a11y: Bilingual invoice PDF with Arabic RTL; accessible tagged PDF.
  acceptance: AC-SF20.2.2 — the 80-person event (Section G) produces one master invoice whose total equals event folio transfers; a credit note reduces it without altering the original.
  dependency: M38.F38.1 SF38.1.4; M12 billing.
- id: M20.F20.2.SF20.2.3
  name: aging/collections
  phase: 4
  release: R1
  actors: [ar_clerk, financial_controller, sales_manager]
  screens: [SCR-FIN-ar-aging, SCR-FIN-collections]
  inputs: [as_of_date, bucket_definition, collection_action, promise_to_pay_date]
  states: [current, overdue, in_collection, promised, escalated, resolved]
  api: GET /v1/properties/{pid}/ar/aging?as_of=; POST /v1/properties/{pid}/ar/collection-cases
  events: [ArInvoiceOverdue, CollectionActionLogged]
  data: [collection_case, ar_invoice]
  rules: ["Aging by invoice date and due date buckets 0-30/31-60/61-90/90+ configurable", "Dunning messages only to billing contacts with consent/contract basis", "Accounts beyond policy go on_stop automatically with sales notification"]
  security: sales sees account status, not other accounts' details
  failure_cases: [contact_missing, disputed_invoice_dunned]
  finance_report_effect: Outstanding AR KPI and bad-debt provisioning input.
  i18n_a11y: Aging table accessible; dunning templates EN/AR.
  acceptance: AC-SF20.2.3 — an invoice 61 days past due appears in 61-90 bucket and triggers on_stop per policy; disputed lines are excluded from dunning totals.
  dependency: M52 consent rules for contacts where applicable.
- id: M20.F20.2.SF20.2.4
  name: allocation of bank receipts
  phase: 4
  release: R1
  actors: [ar_clerk, treasury_worker]
  screens: [SCR-FIN-receipt-allocation, SCR-FIN-bank-reconciliation]
  inputs: [bank_statement_line_id, corporate_account_id, invoice_ids, amounts, remittance_reference]
  states: [unapplied, partially_applied, applied, refunded]
  api: POST /v1/properties/{pid}/ar/receipts (Idempotency-Key); POST /v1/properties/{pid}/ar/receipts/{id}/allocations
  events: [ArReceiptRecorded, ArReceiptAllocated]
  data: [ar_receipt, receipt_allocation, bank_statement_line]
  rules: ["Receipt originates from a bank statement line, PSP settlement or cashier tender; never created twice for one source", "Auto-suggest allocation by remittance reference then amount", "Unapplied cash remains a liability until allocated or refunded", "Short payment leaves remainder open; bank charges posted to fee account with approval"]
  security: AR roles; refunds of unapplied cash go through M28 disbursement with dual approval
  failure_cases: [overpayment, unidentified_payer, fx_difference, same_line_allocated_twice]
  finance_report_effect: Reduces AR; realized FX via SF19.2.4.
  i18n_a11y: Allocation grid accessible with totals announced.
  acceptance: AC-SF20.2.4 — one bank credit paying two invoices partially is allocated once; re-importing the same statement does not create another receipt.
  dependency: SF20.3.1, SF20.3.2.
- id: M20.F20.2.SF20.2.5
  name: credit limit and write-off approval
  phase: 4
  release: R1
  actors: [financial_controller, gm, owner, ar_clerk]
  screens: [SCR-FIN-write-off, SCR-GM-approvals]
  inputs: [corporate_account_id, new_limit, write_off_amount, reason, evidence]
  states: [requested, approved, rejected, posted]
  api: POST /v1/properties/{pid}/ar/credit-accounts/{id}:limit-change; POST /v1/properties/{pid}/ar/write-off-requests
  events: [CreditLimitChanged, ArWriteOffPosted]
  data: [write_off_request, corporate_credit_account, journal_entry]
  rules: ["Limit increases and write-offs follow delegation of authority", "Write-off posts to bad-debt expense and keeps the receivable history; later recovery reverses", "Tax treatment of write-off from M38 where bad-debt relief exists"]
  security: step-up MFA; requester cannot approve
  failure_cases: [write_off_exceeds_balance, approval_chain_missing]
  finance_report_effect: Bad-debt expense in A&G cost center.
  i18n_a11y: Approval card accessible.
  acceptance: AC-SF20.2.5 — a write-off above the controller limit requires owner approval; posting writes one balanced journal.
  dependency: SF20.5.3.
```

### F20.3 Treasury
Story: As financial controller I know which money has actually moved, what is unmatched and what cash I will have.

```yaml
- id: M20.F20.3.SF20.3.1
  name: bank statement import
  phase: 4
  release: R1
  actors: [finance_clerk, treasury_worker, integration_admin]
  screens: [SCR-FIN-bank-statements]
  inputs: [bank_account_id, file_or_feed, format, statement_date, opening_balance, closing_balance]
  states: [received, parsed, validated, rejected]
  api: POST /v1/properties/{pid}/bank-statements:import (Idempotency-Key)
  events: [BankStatementImported, BankStatementRejected]
  data: [bank_account, bank_statement, bank_statement_line]
  rules: ["Formats CAMT.053, MT940, BAI2 or bank CSV via versioned parser", "Opening balance must equal previous closing balance; gaps flagged", "Duplicate line detection by bank reference plus amount plus date", "Raw file retained immutable"]
  security: file upload scanned; API feed credentials in vault; finance-only access
  failure_cases: [balance_discontinuity, duplicate_file, unknown_format, missing_day]
  finance_report_effect: Source of truth for paid-to-settled transition and cash position.
  i18n_a11y: Import result summary readable; Arabic narrative text preserved in UTF-8.
  acceptance: AC-SF20.3.1 — importing the same CAMT file twice creates no new lines; a statement whose opening balance mismatches the previous close is rejected with a gap warning.
  dependency: D-307 bank and format.
- id: M20.F20.3.SF20.3.2
  name: transaction matching
  phase: 4
  release: R1
  actors: [finance_clerk, treasury_worker, financial_controller]
  screens: [SCR-FIN-bank-reconciliation]
  inputs: [bank_statement_line_id, candidate_type, candidate_id, match_rule_id]
  states: [unmatched, suggested, matched, manually_matched, written_off]
  api: POST /v1/properties/{pid}/reconciliation/matches; POST /v1/properties/{pid}/reconciliation:auto-run
  events: [BankLineMatched, BankLineUnmatchedAged]
  data: [reconciliation_match, bank_statement_line, payment_order, psp_settlement_batch, provider_settlement, ar_receipt, cash_deposit]
  rules: ["Candidates include payment orders, payout batches, PSP settlement batches net of fees, bill-provider settlements, AR receipts, cash deposits from M60, payroll batches", "One-to-one, one-to-many and many-to-one matches allowed only when totals equal", "A match settles its sources; unmatch reverses that status with audit", "Small differences post only to an approved difference account within tolerance"]
  security: manual matches over threshold need second reviewer
  failure_cases: [amount_difference, multiple_candidates, missing_settlement_file, bank_fee_netting]
  finance_report_effect: Drives settled measure and close task bank reconciled.
  i18n_a11y: Two-pane matching usable by keyboard; totals announced.
  acceptance: AC-SF20.3.2 — a PSP settlement of 10 captures minus fees matches one bank credit with fee lines posted; the payroll bulk debit matches the payroll payout batch without exposing employee lines to finance_clerk.
  dependency: M28.F28.1 SF28.1.7; M29.F29.1 SF29.1.8; F27.4.
- id: M20.F20.3.SF20.3.3
  name: cash forecast
  phase: 4
  release: R1
  actors: [financial_controller, owner, gm]
  screens: [SCR-FIN-cash-forecast, SCR-GM-home]
  inputs: [horizon_days, scenario, include_sources]
  states: [generated, reviewed, superseded]
  api: POST /v1/properties/{pid}/treasury/cash-forecasts
  events: [CashForecastGenerated]
  data: [cash_forecast, payable, ar_invoice, expense_forecast, payroll_calendar_ref]
  rules: ["Inflows from AR due dates, on-the-books deposits and PSP settlement lag; outflows from approved payables, payroll calendar totals, utility due dates, tax remittance calendar (M38), loan/lease inputs (M66)", "Forecast lines labelled committed vs estimated", "Payroll outflow appears as a total only"]
  security: owner/gm/financial_controller read
  failure_cases: [missing_bank_balance, stale_forecast]
  finance_report_effect: Cash KPI on owner/GM home; not a posting.
  i18n_a11y: Chart with data table alternative.
  acceptance: AC-SF20.3.3 — a 13-week forecast includes the next payroll total and the electricity due date, each tagged with source and confidence.
  dependency: SF20.4.3; M38.F38.1 SF38.1.5.
- id: M20.F20.3.SF20.3.4
  name: vendor payout approval
  phase: 4
  release: R1
  actors: [finance_approver, payment_releaser, financial_controller]
  screens: [SCR-FIN-payout-approval]
  inputs: [payout_batch_id, approver_decision, release_decision, mfa_assertion]
  states: [proposed, approved, released, cancelled]
  api: POST /v1/properties/{pid}/payout-batches/{id}:approve; POST /v1/properties/{pid}/payout-batches/{id}:release
  events: [PayoutBatchApproved, PayoutBatchReleased, PayoutBatchCancelled]
  data: [payout_batch, payment_approval]
  rules: ["Approval of a batch confirms content; release sends to execution; separate people", "Any content change after approval invalidates approvals", "Release window and cutoff per bank; release outside window queues"]
  security: step-up MFA on approve and release; hardware key recommended for releasers
  failure_cases: [batch_changed_after_approval, releaser_equals_approver, mfa_failure]
  finance_report_effect: Payables in released batch shown as approved-in-flight.
  i18n_a11y: Clear confirmation screen summarizing count, total and currency.
  acceptance: AC-SF20.3.4 — editing a payee amount after approval resets approval status; the same user cannot approve and release.
  dependency: M28.F28.2 SF28.2.2.
- id: M20.F20.3.SF20.3.5
  name: unmatched items and chargeback/dispute evidence
  phase: 4
  release: R1
  actors: [finance_clerk, financial_controller, front_office_manager]
  screens: [SCR-FIN-unmatched-items, SCR-OPS-exception-queue]
  inputs: [item_id, owner, due_by, evidence_files, resolution]
  states: [open, assigned, evidence_collected, submitted, resolved, written_off]
  api: POST /v1/properties/{pid}/treasury/unmatched-items/{id}:assign; POST /v1/properties/{pid}/treasury/unmatched-items/{id}:resolve
  events: [UnmatchedItemAged, DisputeEvidenceSubmitted]
  data: [unmatched_item, chargeback, document_attachment]
  rules: ["Every unmatched bank, PSP or provider item has owner and SLA", "Chargeback evidence pack assembles folio, registration signature, ID check record reference and communications within scheme deadline", "Write-off follows delegation of authority"]
  security: evidence excludes raw ID images unless lawful and necessary; access logged
  failure_cases: [deadline_missed, evidence_incomplete]
  finance_report_effect: Aged unmatched value is a close blocker above threshold.
  i18n_a11y: Queue sortable and accessible.
  acceptance: AC-SF20.3.5 — an unmatched PSP debit older than 5 days escalates to financial_controller; a chargeback pack is generated with folio and signature evidence links.
  dependency: M28.F28.1 SF28.1.6; M41 signature evidence.
```

### F20.4 Budget, commitment and expense forecasting (added — Section C "cash/budget and expense forecasting", Section D "budget approval")
Story: As department head I can see how much of my budget is planned, committed and spent before approving new spend.

```yaml
- id: M20.F20.4.SF20.4.1
  name: budget versions by account, cost center and month
  phase: 4
  release: R1
  actors: [financial_controller, gm, owner, fnb_manager, chief_engineer]
  screens: [SCR-FIN-budget]
  inputs: [fiscal_year, version_name, account, cost_center, month, amount, driver_assumptions]
  states: [draft, submitted, approved, active, superseded]
  api: POST /v1/properties/{pid}/budgets; POST /v1/properties/{pid}/budgets/{id}:approve; POST /v1/properties/{pid}/budgets/{id}:import
  events: [BudgetApproved, BudgetRevised]
  data: [budget_version, budget_line]
  rules: ["One active approved version per year; forecasts are separate versions", "Payroll budget is held at department level, never per employee outside M27", "Revisions create a new version with reason"]
  security: department heads edit own cost centers in draft; owner approves
  failure_cases: [unmapped_account, import_format_error]
  finance_report_effect: Budget column in M32 budget/actual.
  i18n_a11y: Spreadsheet-like grid with keyboard entry and screen-reader headers.
  acceptance: AC-SF20.4.1 — importing a 12-month XLSX budget for two cost centers creates one draft version; approval makes it active and the previous version superseded.
  dependency: D-313.
- id: M20.F20.4.SF20.4.2
  name: budget availability check and encumbrance ledger
  phase: 4
  release: R1
  actors: [procurement_officer, procurement_approver, budget_worker]
  screens: [SCR-PROC-requisition, SCR-FIN-budget]
  inputs: [cost_center, account, period, amount, source_type, source_id]
  states: [available, insufficient, encumbered, relieved, released]
  api: POST /v1/properties/{pid}/budgets:check; GET /v1/properties/{pid}/encumbrances?source_id=
  events: [BudgetEncumbered, EncumbranceRelieved, BudgetExceeded]
  data: [encumbrance, budget_line]
  rules: ["Approved requisition creates a pre-encumbrance; PO converts it to encumbrance; invoice relieves it; cancellation releases it", "Insufficient budget blocks unless an over-budget approver authorizes", "Encumbrance ledger is append-only"]
  security: over-budget approval role distinct from requester
  failure_cases: [race_two_requisitions_same_budget, currency_conversion, cancelled_po_not_released]
  finance_report_effect: Planned measure on the cost-status dashboard.
  i18n_a11y: Budget remaining displayed as text with value.
  acceptance: AC-SF20.4.2 — two concurrent requisitions exceeding remaining budget cannot both pass without over-budget approval; PO cancellation releases the encumbrance exactly once.
  dependency: SF21.1.2; M49.F49.3 SF49.3.1.
- id: M20.F20.4.SF20.4.3
  name: recurring expense forecast
  phase: 4
  release: R1
  actors: [financial_controller, finance_clerk, forecast_worker]
  screens: [SCR-FIN-expense-forecast]
  inputs: [category, method, occupancy_forecast_ref, tariff_version, contract_ref, horizon]
  states: [generated, adjusted, approved]
  api: POST /v1/properties/{pid}/expense-forecasts
  events: [ExpenseForecastGenerated]
  data: [expense_forecast]
  rules: ["Utilities from consumption baseline times occupancy forecast times tariff", "Payroll from roster and compensation totals supplied by M27 at department level", "Contracts and subscriptions from recurring_payable_template", "Every line states method and confidence"]
  security: finance roles; payroll lines aggregated
  failure_cases: [missing_occupancy_forecast, tariff_expired]
  finance_report_effect: Feeds planned measure and cash forecast.
  i18n_a11y: Method explanation in EN/AR.
  acceptance: AC-SF20.4.3 — the electricity forecast for next month changes when M53 occupancy forecast changes and records both versions.
  dependency: M53.F53.1; F22.1; F27.3.
- id: M20.F20.4.SF20.4.4
  name: budget vs actual vs forecast variance and alerts
  phase: 4
  release: R1
  actors: [gm, owner, financial_controller, department_heads]
  screens: [SCR-GM-home, SCR-FIN-budget-variance]
  inputs: [period, cost_center, threshold_percent]
  states: [within_threshold, warning, breached]
  api: GET /v1/properties/{pid}/budgets/variance?period=&cost_center=
  events: [BudgetVarianceBreached]
  data: [budget_line, journal_line, expense_forecast, encumbrance]
  rules: ["Variance computed on actual plus accrual versus budget, with committed shown separately", "Alerts to cost center owner and gm at threshold", "Estimate share disclosed"]
  security: department heads see own cost centers
  failure_cases: [period_not_closed, allocation_not_run]
  finance_report_effect: Budget/actual in M32 department P&L.
  i18n_a11y: Variance with signed numbers and words; not color-only.
  acceptance: AC-SF20.4.4 — kitchen gas cost exceeding budget by 12 percent with a 10 percent threshold alerts fnb_manager and gm with drill-through.
  dependency: M32.F32.3 SF32.3.3.
```

### F20.5 Payable backbone and cost-status reconciliation (added — Section D)
Story: As owner I see every hotel cost move through one backbone and can tell planned from accrued from invoiced from approved from paid from settled.

```yaml
- id: M20.F20.5.SF20.5.1
  name: unified payable lifecycle backbone
  phase: 4
  release: R1
  actors: [ap_clerk, finance_approver, payment_releaser, ap_worker]
  screens: [SCR-FIN-payable-detail, SCR-OPS-exception-queue]
  inputs: [source_type, source_id, evidence_refs, invoice_ref, cost_center, due_date]
  states: [requested, evidenced, invoiced, validated, approved, scheduled, released, paid, settled, on_hold, disputed, void]
  api: GET /v1/properties/{pid}/payables/{id}/backbone; GET /v1/properties/{pid}/payables?stage=
  events: [PayableStageChanged, PayableStageSkippedAttempt]
  data: [payable, invoice_match, payment_order, bill_payment_order, reconciliation_match, journal_entry]
  rules: ["Backbone stages are request/usage, evidence, vendor invoice, validation, budget approval, payable, authorized payment, provider/bank confirmation, GL and cost center, reconciliation/report", "Every payable type (utilities, cylinders, salaries, maintenance, other suppliers) uses this state machine SM-payable", "Stage skip attempts are rejected and logged", "Approval never triggers money movement by itself", "Salaries appear as payroll_run payables with department totals"]
  security: stage transitions by role; audit each transition with actor and evidence
  failure_cases: [evidence_missing, stage_skip, source_cancelled_after_approval, payment_without_confirmation]
  finance_report_effect: Single ledger for all six dashboard measures.
  i18n_a11y: Stage stepper with text labels and aria-current.
  acceptance: AC-SF20.5.1 — attempting to create a payment order for a payable without evidence returns BACKBONE_STAGE_MISSING; each of the Section D categories (electricity, water, pipeline gas, cylinders, salaries, maintenance, other supplier) traverses all stages in fixture AT-G04.1.
  dependency: SM-payable in docs/02; M21, M22–M27, M28, M29.
- id: M20.F20.5.SF20.5.2
  name: planned/accrued/invoiced/approved/paid/settled dashboard
  phase: 4
  release: R1
  actors: [owner, gm, financial_controller, ap_clerk]
  screens: [SCR-FIN-cost-status-dashboard, SCR-GM-home]
  inputs: [period, cost_center, cost_category, supplier_id, as_of]
  states: [current, stale_source]
  api: GET /v1/properties/{pid}/cost-status?period=&group_by=
  events: [CostStatusSnapshotTaken]
  data: [cost_status_snapshot, encumbrance, accrual_schedule, supplier_invoice, payable, payment_order, reconciliation_match]
  rules: ["Six measures shown separately and each queried from its own state, never derived by subtraction", "Accrued labelled estimate; settled labelled reconciled", "Drill from every cell to underlying records", "Payroll row shows department totals only for non-HR roles", "Snapshot at close is immutable for comparison"]
  security: role-scoped rows; salary masking per F27.4
  failure_cases: [source_feed_stale, currency_mixing, snapshot_missing]
  finance_report_effect: Owner view of obligations; feeds M32.F32.4 SF32.4.2 finance and cash.
  i18n_a11y: Table first with optional chart; column headers explicit; RTL mirrored.
  acceptance: AC-SF20.5.2 — for the Section G month, electricity shows planned (forecast), accrued (estimate), invoiced, approved, paid and settled as six distinct values and the settled total equals matched bank lines (AT-G04.4).
  dependency: SF20.4.2, SF20.5.5, SF20.3.2.
- id: M20.F20.5.SF20.5.3
  name: delegation of authority and spend limits
  phase: 4
  release: R1
  actors: [owner, financial_controller, gm, property_admin]
  screens: [SCR-FIN-delegation-of-authority]
  inputs: [role_or_user, category, cost_center, amount_limit, currency, valid_from, valid_to, delegate_id]
  states: [draft, active, expired, revoked]
  api: POST /v1/properties/{pid}/delegation-of-authority; POST /v1/properties/{pid}/delegation-of-authority/{id}:revoke
  events: [DelegationOfAuthorityChanged]
  data: [delegation_of_authority]
  rules: ["Limits by action (requisition approve, PO approve, invoice approve, payment release, write-off, refund) and category", "Temporary delegation has end date and cannot exceed delegator limit", "Split purchases to evade limits are detected by M60 SF60.2.4", "Changes require owner or financial_controller plus second approver"]
  security: step-up MFA; full audit
  failure_cases: [delegate_exceeds_delegator, expired_delegation_used, no_approver_available]
  finance_report_effect: Control evidence for audit.
  i18n_a11y: Matrix view accessible as table.
  acceptance: AC-SF20.5.3 — a delegate cannot approve beyond the delegator's limit; three invoices of OMR 490 from one vendor on one day under a 500 limit raise a split-order alert.
  dependency: M02 delegated approvals; M60.F60.2; D-306.
- id: M20.F20.5.SF20.5.4
  name: partial payment, credit application and FX settlement
  phase: 4
  release: R1
  actors: [ap_clerk, finance_approver]
  screens: [SCR-FIN-payable-detail]
  inputs: [payable_id, amount, credit_note_id, fx_rate_id]
  states: [open, partially_paid, paid, credited]
  api: POST /v1/properties/{pid}/payables/{id}:apply-credit; POST /v1/properties/{pid}/payables/{id}/installments
  events: [SupplierCreditApplied, PayablePartiallyPaid]
  data: [payable, supplier_credit_note, payment_order]
  rules: ["Outstanding amount equals approved amount minus confirmed payments minus applied credits", "A payment order can never exceed outstanding amount", "Foreign-currency payables settle with realized FX posting", "Withholding tax where applicable is deducted and recorded as liability per M38"]
  security: AP roles; audit
  failure_cases: [overpayment_attempt, credit_exceeds_balance, fx_rate_missing]
  finance_report_effect: Correct AP balance and FX gain/loss.
  i18n_a11y: Balance breakdown readable.
  acceptance: AC-SF20.5.4 — two installments of 60 and 40 percent close the payable; a third order is rejected; a credit note applied reduces outstanding once.
  dependency: SF19.2.4.
- id: M20.F20.5.SF20.5.5
  name: received-not-invoiced and usage accruals
  phase: 4
  release: R1
  actors: [finance_clerk, accrual_worker]
  screens: [SCR-FIN-accruals]
  inputs: [period, goods_receipt_ids, service_acceptance_ids, usage_estimates, payroll_accrual_total]
  states: [computed, posted, reversed]
  api: POST /v1/properties/{pid}/accruals:compute?period=
  events: [GrniAccrualPosted, UsageAccrualPosted]
  data: [accrual_schedule, goods_receipt, service_acceptance, usage_reconciliation]
  rules: ["Accepted receipts and services without matched invoice at period end accrue at PO price", "Utility usage without bill accrues at tariff times metered or estimated usage", "Payroll earned but unpaid accrues by department", "All accruals auto-reverse next period"]
  security: finance roles
  failure_cases: [po_price_missing, meter_gap_estimate]
  finance_report_effect: Accrued measure and complete monthly P&L.
  i18n_a11y: Accrual listing accessible.
  acceptance: AC-SF20.5.5 — a maintenance service accepted on 30 Sep and invoiced on 5 Oct accrues in September and reverses in October with no double expense.
  dependency: SF19.1.4.
- id: M20.F20.5.SF20.5.6
  name: non-PO recurring payables
  phase: 4
  release: R1
  actors: [ap_clerk, financial_controller]
  screens: [SCR-FIN-recurring-payables]
  inputs: [vendor_id, category, amount_or_formula, schedule, contract_ref, cost_center, prepayment_flag]
  states: [active, paused, ended]
  api: POST /v1/properties/{pid}/recurring-payables
  events: [RecurringPayableGenerated]
  data: [recurring_payable_template, payable, prepayment_schedule]
  rules: ["Rent, insurance, subscriptions, licences and service contracts generate expected payables that still require invoice and approval", "Annual prepayments route to prepayment schedule", "Contract end date alerts 60 days before"]
  security: AP roles
  failure_cases: [invoice_differs_from_template, contract_expired]
  finance_report_effect: Planned measure and prepayment amortization.
  i18n_a11y: Schedule accessible.
  acceptance: AC-SF20.5.6 — an insurance premium of 12 months creates a prepayment schedule amortizing monthly after invoice approval.
  dependency: M68.F68.1 SF68.1.2.
```

**M20 key invariants:** one payable per unique supplier invoice key; no payment order without approved payable and verified payment profile; approver ≠ releaser; payable paid only on external confirmation; settled only after bank/provider match; the six dashboard measures are independently sourced; receipts cannot be allocated twice.

**M20 module acceptance (Section G):** AT-G04.1 every Section D cost category completes the backbone for the fixture hotel; AT-G04.4 cost-status dashboard shows six separate measures reconciled to bank; AT-G08.4 AR aging and city ledger reconcile to GL control; AT-G02.3 corporate deposit/PO confirms composite booking within credit; AT-G20.4 repeated invoice detected and paid once; AT-G20.7 bank rejection surfaces and resubmits once.

**M20 open decisions**

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-306 | Delegation-of-authority matrix amounts | owner | Three tiers: department head ≤ 500, GM ≤ 5,000, owner above (base currency). |
| D-307 | Bank(s), accounts and statement format/feed per pilot legal entity | financial_controller | CAMT.053 or bank CSV file upload daily; no direct bank API until contracted (`unverified-assumption`). |
| D-308 | Invoice match tolerances (price/quantity %, absolute cap) | financial_controller + procurement_approver | ±2% price, 0% quantity over receipt, absolute cap OMR 5.000 equivalent. |
| D-309 | Corporate credit policy, payment terms, on-stop and write-off thresholds | financial_controller | Net 30, on-stop at 60 days overdue, write-off > 90 days with owner approval. |
| D-310 | Supplier payment-run cadence and default terms | financial_controller | Weekly payment run; default supplier terms net 30 unless contract says otherwise. |
| D-311 | Invoice OCR engine (local vs cloud) and data residency | it_admin + dpo | Local open-source OCR for on-prem; cloud OCR only with DPA and residency review. |
| D-312 | Structured e-invoicing obligations per market (e.g. Saudi e-invoicing, Portugal certified invoicing, Oman plans) for AP intake and AR issue | compliance_officer via M38/M44 | AR issues through M38 per verified rule pack; AP accepts PDF and structured files; unverified markets block automated e-invoice submission. |
| D-313 | Budget ownership, cadence and reforecast frequency | gm + financial_controller | Annual budget approved by owner; quarterly reforecast. |

---
## 5. M21 — Procurement (policy and backbone)

| Header | Value |
|---|---|
| Purpose | Source-to-pay policy spine: `requisition -> threshold and RFQ policy -> supplier/sample/stock comparison -> weighted selection and documented override -> award -> versioned PO -> AI-monitored fulfillment -> low-touch goods receipt/service acceptance -> invoice match -> payout`, plus budgets, blanket contracts, replenishment, emergency purchase, disputes and segregation of duties. **Detailed RFQ/sample/award/PO behavior is specified in M49 (F49.1–F49.3); ASN/receiving/stock ledger in M50 (F50.1–F50.3).** M21 owns the rules those engines must obey and the handoff to M20. |
| Phases | Requisition, thresholds and PO handoff Phase 3; budget, blanket, replenishment, SoD, matching Phase 4; hardening Phase 6. |
| Release | R1 |
| Bounded context | `procurement.policy` |
| System of record | `purchase_requisition`, `purchase_requisition_line`, `procurement_policy`, `approval_threshold`, `blanket_agreement`, `blanket_call_off`, `replenishment_rule`, `sod_rule`, `sod_violation`, `emergency_purchase`, `service_acceptance`, `receipt_acceptance_policy`, `procurement_dispute`, `capitalization_decision` |
| Dependencies | M49 (RFQ, `purchase_order`), M50 (`goods_receipt`, stock ledger), M46 (eligible vendors), M48 (catalog/price), M20 (budget, encumbrance, match, payable), M26 (maintenance triggers), M25 (cylinder reorder), M14/M56 (stock par), M19 (GL), M60 (split-order/conflict detection), M66 (capex) |

### F21.1 Request-to-order
Story: As department head I raise a requisition from a stock or maintenance need; policy tells me how many quotes, which approvals and which budget apply before a PO can be issued.

```yaml
- id: M21.F21.1.SF21.1.1
  name: stock or maintenance trigger
  phase: 3
  release: R1
  actors: [storekeeper, chief_engineer, executive_chef, housekeeping_supervisor, replenishment_worker]
  screens: [SCR-PROC-requisition, SCR-ENG-work-order-detail, SCR-STORE-reorder-suggestions]
  inputs: [trigger_type, source_id, item_id_or_service_category, quantity, uom, need_by, delivery_location, department_id]
  states: [suggested, drafted, dismissed]
  api: POST /v1/properties/{pid}/requisitions:from-trigger (Idempotency-Key)
  events: [RequisitionSuggested, RequisitionDrafted]
  data: [purchase_requisition, purchase_requisition_line, replenishment_rule, work_order]
  rules: ["Triggers are reorder point breach (SF21.3.3), cylinder threshold (SF25.1.3), work order parts or contractor need (SF26.3.1), BEO/catering demand (M16), manual need", "One open suggestion per item per store at a time", "Trigger keeps source link for audit and drill"]
  security: department-scoped creation; vendor has no access
  failure_cases: [duplicate_trigger, item_not_in_master, uom_conversion_missing]
  finance_report_effect: None until approved; pre-encumbrance on approval.
  i18n_a11y: Mobile-friendly requisition with bilingual item names and large touch targets.
  acceptance: AC-SF21.1.1 — a gas-cylinder count below reorder point creates exactly one suggested requisition linked to the cylinder stock record; a second breach event the same day does not duplicate it (AT-G04.2).
  dependency: M14/M50 stock balances; F25.1; F26.1.
- id: M21.F21.1.SF21.1.2
  name: budget/cost-center validation
  phase: 4
  release: R1
  actors: [procurement_officer, budget_worker, procurement_approver]
  screens: [SCR-PROC-requisition]
  inputs: [cost_center, account, estimated_amount, currency, period]
  states: [not_checked, within_budget, over_budget_pending, over_budget_approved, rejected]
  api: POST /v1/properties/{pid}/requisitions/{id}:budget-check
  events: [RequisitionBudgetChecked, BudgetExceeded]
  data: [purchase_requisition, encumbrance, budget_line]
  rules: ["Cost center and account derive from department and item category mapping; override needs reason", "Budget check uses SF20.4.2 and creates pre-encumbrance on approval", "Over-budget requires over-budget approver distinct from requester"]
  security: requester sees own budget remaining only
  failure_cases: [no_active_budget, cost_center_inactive, fx_estimate_missing]
  finance_report_effect: Planned measure increases on approval.
  i18n_a11y: Budget remaining stated in text.
  acceptance: AC-SF21.1.2 — a requisition exceeding remaining maintenance budget is routed to the over-budget approver; its pre-encumbrance appears in planned.
  dependency: SF20.4.2.
- id: M21.F21.1.SF21.1.3
  name: request and approval threshold
  phase: 3
  release: R1
  actors: [department_heads, procurement_approver, gm, financial_controller]
  screens: [SCR-PROC-requisition, SCR-GM-approvals]
  inputs: [requisition_id, amount, category, urgency, approver_decision]
  states: [draft, submitted, approved, rejected, returned, cancelled]
  api: POST /v1/properties/{pid}/requisitions/{id}:submit; POST /v1/properties/{pid}/requisitions/{id}:approve
  events: [RequisitionSubmitted, RequisitionApproved, RequisitionRejected]
  data: [purchase_requisition, approval_threshold, delegation_of_authority]
  rules: ["Approval chain by value band, category and cost center from approval_threshold and delegation_of_authority", "Threshold also determines sourcing route - direct from blanket/catalog, quotes, or formal RFQ in M49", "Requester cannot approve own requisition"]
  security: step-up above configured value
  failure_cases: [no_eligible_approver, approver_conflict_of_interest, split_requisition]
  finance_report_effect: Pre-encumbrance created.
  i18n_a11y: Approval notification bilingual; accessible approve/reject with reason.
  acceptance: AC-SF21.1.3 — a requisition above the formal-RFQ threshold cannot convert directly to PO and is routed to M49; self-approval returns SOD_VIOLATION.
  dependency: SF21.3.1; D-314.
- id: M21.F21.1.SF21.1.4
  name: multi-supplier RFQ/quote comparison
  phase: 3
  release: R1
  actors: [procurement_officer, procurement_approver]
  screens: [SCR-PROC-rfq-comparison]
  inputs: [requisition_id, rfq_policy_id, eligible_vendor_ids]
  states: [policy_resolved, rfq_in_m49, award_returned, waiver_pending]
  api: POST /v1/properties/{pid}/requisitions/{id}:source
  events: [RequisitionSourcingStarted, AwardReceivedForRequisition]
  data: [purchase_requisition, procurement_policy, rfq_ref]
  rules: ["M21 resolves the minimum quote count, weights profile and sealed/open rule from policy and passes them to M49.F49.1 SF49.1.5 and M49.F49.2", "M21 accepts only an award with documented comparison or approved waiver (M49 SF49.1.6)", "Eligible vendors come from M46 for the category and location only"]
  security: bidders never see competitor bids (M48 SF48.3.7)
  failure_cases: [insufficient_eligible_vendors, waiver_rejected, rfq_expired]
  finance_report_effect: None directly; award price feeds encumbrance update.
  i18n_a11y: Comparison screen defined in M49.
  acceptance: AC-SF21.1.4 — for 120 kg vegetables with policy minimum three quotes, only two responsive quotes force the exception path with higher approver (AT-G17.1, executed in M49).
  dependency: M49.F49.1, M49.F49.2.
- id: M21.F21.1.SF21.1.5
  name: PO issue/change/cancel
  phase: 3
  release: R1
  actors: [procurement_officer, procurement_approver, vendor_user]
  screens: [SCR-PROC-po-detail, SCR-VEN-po-inbox]
  inputs: [award_id_or_call_off_id, po_version, change_reason, cancel_reason]
  states: [po_requested, po_issued_in_m49, changed, cancelled, closed]
  api: POST /v1/properties/{pid}/requisitions/{id}:request-po
  events: [PurchaseOrderRequested, PurchaseOrderStatusMirrored]
  data: [purchase_requisition, purchase_order, encumbrance]
  rules: ["The versioned purchase_order is owned by M49 SF49.3.4/SF49.3.5; M21 enforces that a PO exists only for approved requisitions, awards, blanket call-offs or emergency purchases", "PO value increase beyond tolerance re-enters approval", "Cancellation releases encumbrance (SF20.4.2)"]
  security: vendor acknowledges via M49; changes audited
  failure_cases: [po_without_approved_source, change_after_receipt, cancel_after_partial_receipt]
  finance_report_effect: Encumbrance created on PO issue, adjusted on change, released on cancel.
  i18n_a11y: PO PDF bilingual.
  acceptance: AC-SF21.1.5 — attempting a PO without approved requisition or call-off is rejected; increasing PO value by 15 percent re-triggers approval.
  dependency: M49.F49.3.
- id: M21.F21.1.SF21.1.6
  name: emergency retrospective approval
  phase: 3
  release: R1
  actors: [duty_manager, chief_engineer, executive_chef, gm, financial_controller]
  screens: [SCR-PROC-emergency-purchase, SCR-GM-approvals]
  inputs: [reason, safety_or_guest_impact, vendor_id, estimated_amount, evidence, incident_id]
  states: [declared, ordered, retrospective_pending, ratified, rejected_investigation]
  api: POST /v1/properties/{pid}/emergency-purchases (Idempotency-Key); POST /v1/properties/{pid}/emergency-purchases/{id}:ratify
  events: [EmergencyPurchaseDeclared, EmergencyPurchaseRatified, EmergencyPurchaseRejected]
  data: [emergency_purchase, purchase_order, incident_ref]
  rules: ["Allowed only for defined reasons (safety, guest-impacting outage, food service continuity) up to a cap", "Retrospective approval required within configured hours; unratified emergency purchase blocks payment", "Frequency per requester monitored by M60"]
  security: declared by named roles only; audit and reason mandatory
  failure_cases: [cap_exceeded, ratification_overdue, misuse_pattern]
  finance_report_effect: Payable held until ratified; flagged in purchasing report.
  i18n_a11y: One-screen mobile declaration usable offline with queued submit.
  acceptance: AC-SF21.1.6 — an emergency pump repair at 02:00 issues a PO; payment is blocked until GM ratifies within 24 hours; overdue ratification escalates.
  dependency: D-317; M42 incident link.
```

### F21.2 Receive/accept
Story: As receiver or engineer I accept only what arrived or was done, with evidence, so AP pays exactly once for accepted quantities.

```yaml
- id: M21.F21.2.SF21.2.1
  name: goods quantity/condition/batch
  phase: 3
  release: R1
  actors: [receiver, storekeeper, executive_chef]
  screens: [SCR-STORE-receiving]
  inputs: [po_id, asn_id, lines_received, lot, expiry, condition, temperature, photos]
  states: [expected, received_draft, accepted, partially_accepted, quarantined, rejected]
  api: POST /v1/properties/{pid}/goods-receipts (defined in M50)
  events: [GoodsReceiptAccepted, GoodsReceiptQuarantined]
  data: [goods_receipt, receipt_acceptance_policy]
  rules: ["goods_receipt and stock-ledger posting are owned by M50.F50.2", "M21 sets receipt_acceptance_policy per category (food high-risk requires physical verification; low-risk straight-through allowed per M50 SF50.2.5)", "Only accepted quantity is matchable in SF21.2.5"]
  security: receiver identity and attestation required
  failure_cases: [asn_mismatch, temperature_breach, lot_missing]
  finance_report_effect: Accepted quantity creates GRNI accrual basis and stock value (M50).
  i18n_a11y: Defined in M50 screens.
  acceptance: AC-SF21.2.1 — a food receipt with temperature breach cannot be accepted and is quarantined; accepted quantity is the only quantity offered to invoice match.
  dependency: M50.F50.2.
- id: M21.F21.2.SF21.2.2
  name: service milestone/evidence
  phase: 3
  release: R1
  actors: [chief_engineer, engineer, department_heads, vendor_user]
  screens: [SCR-ENG-service-acceptance, SCR-VEN-job-evidence]
  inputs: [po_id, work_order_id, milestone_id, evidence_photos, checklist_results, inspector_id]
  states: [pending, submitted_by_vendor, accepted, rejected, partially_accepted]
  api: POST /v1/properties/{pid}/service-acceptances (Idempotency-Key)
  events: [ServiceAccepted, ServiceRejected]
  data: [service_acceptance, work_evidence, purchase_order]
  rules: ["Vendor submission is not acceptance; a hotel inspector distinct from the vendor accepts", "Milestone value accepted cannot exceed PO milestone value", "Room release after maintenance requires SF26.1.6 inspection"]
  security: vendor sees own jobs only; inspector role required
  failure_cases: [evidence_missing, inspector_is_requester_for_high_value, milestone_overclaim]
  finance_report_effect: Acceptance is the evidence stage; triggers GRNI accrual for services.
  i18n_a11y: Photo evidence with captions; checklist accessible.
  acceptance: AC-SF21.2.2 — vendor-marked complete job remains unpayable until engineer accepts; partial acceptance of 60 percent limits invoice match to 60 percent.
  dependency: M26.F26.2 SF26.2.4.
- id: M21.F21.2.SF21.2.3
  name: short receipt/return
  phase: 3
  release: R1
  actors: [receiver, procurement_officer, vendor_user]
  screens: [SCR-STORE-receiving, SCR-PROC-supplier-claims]
  inputs: [goods_receipt_id, shortage_qty, return_qty, reason, evidence]
  states: [short_recorded, return_requested, returned, credit_expected, credit_received, closed]
  api: POST /v1/properties/{pid}/goods-receipts/{id}/returns (M50); POST /v1/properties/{pid}/procurement-disputes
  events: [ShortReceiptRecorded, ReturnToVendorCreated, SupplierCreditExpected]
  data: [goods_receipt, procurement_dispute, supplier_credit_note]
  rules: ["Short or damaged quantity never enters available stock", "Return-to-vendor creates expected credit tracked to closure", "Backorder or cancel remainder is an explicit decision"]
  security: procurement roles
  failure_cases: [vendor_denies_shortage, credit_not_received]
  finance_report_effect: Reduces matchable quantity; expected credit tracked in AP.
  i18n_a11y: Defined in M50 screens.
  acceptance: AC-SF21.2.3 — a supplier short delivery of 10 kg produces a supplier claim and the invoice for full quantity raises a match exception (AT-G20.5).
  dependency: M50.F50.2 SF50.2.6, SF50.2.9.
- id: M21.F21.2.SF21.2.4
  name: stock/asset capitalization decision
  phase: 4
  release: R1
  actors: [financial_controller, chief_engineer, procurement_officer]
  screens: [SCR-PROC-po-detail, SCR-FIN-capitalization]
  inputs: [po_line_id, item_category, unit_value, useful_life_estimate, asset_class]
  states: [expense, inventory, capex_pending, capitalized]
  api: POST /v1/properties/{pid}/po-lines/{id}:capitalization-decision
  events: [CapitalizationDecided, AssetCreatedFromReceipt]
  data: [capitalization_decision, asset]
  rules: ["Decision at PO line using category defaults and capitalization threshold", "Capitalized items create or update an asset (M26) and hand off depreciation to M66", "Inventory items post to stock (M50); others expense to cost center"]
  security: financial_controller approves capex classification
  failure_cases: [threshold_ambiguous, asset_class_missing]
  finance_report_effect: Determines P&L vs balance-sheet treatment.
  i18n_a11y: Decision options explained in plain language.
  acceptance: AC-SF21.2.4 — a replacement chiller compressor above threshold becomes capex_pending then capitalized with asset link; linen below threshold expenses.
  dependency: D-318; M66.F66.2 SF66.2.3.
- id: M21.F21.2.SF21.2.5
  name: invoice three-way match and tolerance
  phase: 4
  release: R1
  actors: [ap_clerk, procurement_officer, ap_worker]
  screens: [SCR-FIN-match-workbench]
  inputs: [tolerance_profile_id, category, price_pct, qty_pct, absolute_cap]
  states: [configured, active, superseded]
  api: PUT /v1/properties/{pid}/match-tolerances/{profile_id}
  events: [MatchToleranceChanged]
  data: [invoice_match, tolerance_profile]
  rules: ["Tolerance profiles by category; zero tolerance for quantity above accepted", "Match execution in SF20.1.4 uses the active profile at invoice date", "Tolerance change is approved and versioned"]
  security: financial_controller approves changes
  failure_cases: [profile_missing, tolerance_too_loose_warning]
  finance_report_effect: Within-tolerance variances post to purchase price variance account.
  i18n_a11y: Settings form accessible.
  acceptance: AC-SF21.2.5 — a 1.5 percent price variance under a 2 percent profile auto-matches and posts variance; 3 percent raises an exception; stock ledger and AP match occur once (AT-G18.4).
  dependency: D-308; SF20.1.4.
```

### F21.3 Procurement policy and controls (added — Section C "budgets, blanket contracts, replenishment, emergency purchase, disputes and segregation of duties")
Story: As financial controller I configure procurement rules once and the system enforces them in every department.

```yaml
- id: M21.F21.3.SF21.3.1
  name: procurement policy and threshold configuration
  phase: 3
  release: R1
  actors: [financial_controller, procurement_approver, property_admin]
  screens: [SCR-PROC-policy]
  inputs: [category, value_band, min_quotes, sourcing_route, weights_profile_id, sealed_bid_flag, jurisdiction_rule_ref, sample_required, effective_from]
  states: [draft, approved, active, superseded]
  api: PUT /v1/properties/{pid}/procurement-policies/{id}; POST /v1/properties/{pid}/procurement-policies/{id}:approve
  events: [ProcurementPolicyActivated]
  data: [procurement_policy, approval_threshold]
  rules: ["Policy resolves sourcing route per category and value band - catalog or blanket call-off, minimum N quotes, formal RFQ", "Min quotes and weights are consumed by M49; version locked at RFQ open", "Jurisdiction constraints from M44 may add requirements"]
  security: maker-checker for policy changes
  failure_cases: [overlapping_bands, missing_category_default]
  finance_report_effect: Audit basis for quote competition report (M50 SF50.4.2).
  i18n_a11y: Policy editor with plain-language summary.
  acceptance: AC-SF21.3.1 — policy for food above 200 OMR requires three quotes; RFQ opened under version 3 keeps version 3 rules after policy changes to version 4.
  dependency: D-314; M49.F49.1 SF49.1.5.
- id: M21.F21.3.SF21.3.2
  name: blanket contracts and call-offs
  phase: 4
  release: R1
  actors: [procurement_officer, procurement_approver, storekeeper, vendor_user]
  screens: [SCR-PROC-blanket-agreements]
  inputs: [vendor_id, items_or_services, price_schedule, max_value, max_quantity, valid_from, valid_to, call_off_quantity]
  states: [draft, approved, active, exhausted, expired, terminated]
  api: POST /v1/properties/{pid}/blanket-agreements; POST /v1/properties/{pid}/blanket-agreements/{id}/call-offs (Idempotency-Key)
  events: [BlanketAgreementActivated, BlanketCallOffCreated, BlanketAgreementNearLimit]
  data: [blanket_agreement, blanket_call_off, purchase_order]
  rules: ["Blanket agreement is itself awarded through M49 or approved waiver", "Call-offs use contract price and create PO releases without new RFQ within limits", "Cumulative call-offs cannot exceed max value or quantity; alert at 80 percent", "Expired agreement blocks call-offs"]
  security: call-off by authorized requesters; vendor sees own agreement
  failure_cases: [limit_exceeded, price_changed_by_vendor, agreement_expired]
  finance_report_effect: Call-off creates encumbrance; utilization report.
  i18n_a11y: Utilization shown as numbers with text.
  acceptance: AC-SF21.3.2 — cylinder refill call-offs beyond the agreement quantity are blocked; price on PO equals contract price.
  dependency: D-315; M49.F49.3.
- id: M21.F21.3.SF21.3.3
  name: replenishment suggestions (par/min-max)
  phase: 4
  release: R1
  actors: [storekeeper, executive_chef, housekeeping_supervisor, replenishment_worker]
  screens: [SCR-STORE-reorder-suggestions]
  inputs: [store_id, item_id, min_qty, max_qty, par_level, lead_time_days, forecast_ref]
  states: [configured, suggestion_open, converted, dismissed]
  api: PUT /v1/properties/{pid}/replenishment-rules/{id}; GET /v1/properties/{pid}/replenishment-suggestions
  events: [ReplenishmentSuggested]
  data: [replenishment_rule, purchase_requisition]
  rules: ["Suggestion equals max minus available (excluding quarantined) minus open POs", "Event/BEO demand and occupancy forecast may raise suggested quantity", "Stale stock data blocks auto-suggestion"]
  security: store-scoped
  failure_cases: [stock_data_stale, lead_time_missing]
  finance_report_effect: None until requisition approved.
  i18n_a11y: Suggestions list accessible.
  acceptance: AC-SF21.3.3 — with available 4, open PO 3 and max 12, the suggestion is 5; quarantined units are not counted as available.
  dependency: M50.F50.3 SF50.3.1.
- id: M21.F21.3.SF21.3.4
  name: segregation-of-duties matrix and conflict detection
  phase: 3
  release: R1
  actors: [financial_controller, compliance_officer, auditor]
  screens: [SCR-FIN-sod-matrix, SCR-OPS-exception-queue]
  inputs: [action_pair, severity, compensating_control, exception_expiry]
  states: [active_rule, violation_blocked, exception_granted, exception_expired]
  api: PUT /v1/properties/{pid}/sod-rules/{id}; GET /v1/properties/{pid}/sod-violations
  events: [SodViolationBlocked, SodExceptionGranted]
  data: [sod_rule, sod_violation]
  rules: ["Blocking pairs include requisition vs approve, approve vs payment release, vendor bank change vs payment release, receive vs invoice approve for same PO, create vendor vs pay vendor", "Conflict-of-interest declarations from M46 block related approvers", "Small hotels may grant time-bound exceptions with compensating owner review"]
  security: rules enforced server-side on every action; exceptions need owner approval
  failure_cases: [same_person_multiple_roles, exception_expired_during_workflow]
  finance_report_effect: Control evidence; violations reported to owner.
  i18n_a11y: Violation messages explain which role conflict.
  acceptance: AC-SF21.3.4 — the user who approved a payable cannot release its payment; an expired SoD exception stops approval immediately.
  dependency: D-316; M02.
- id: M21.F21.3.SF21.3.5
  name: procurement disputes and supplier claims
  phase: 4
  release: R1
  actors: [procurement_officer, ap_clerk, vendor_user, procurement_approver]
  screens: [SCR-PROC-supplier-claims, SCR-VEN-disputes]
  inputs: [dispute_type, po_id, invoice_id, goods_receipt_id, claimed_amount, evidence, resolution]
  states: [opened, vendor_responded, escalated, resolved_credit, resolved_no_credit, withdrawn]
  api: POST /v1/properties/{pid}/procurement-disputes; POST /v1/properties/{pid}/procurement-disputes/{id}:resolve
  events: [ProcurementDisputeOpened, ProcurementDisputeResolved]
  data: [procurement_dispute, supplier_credit_note, payable_hold]
  rules: ["Dispute links to PO, receipt and invoice; disputed amount placed on payable hold", "Resolution updates vendor performance (M26 SF26.2.7, M50 SF50.4.1)", "Vendor communication kept in case thread"]
  security: vendor sees own disputes only
  failure_cases: [no_vendor_response, evidence_disputed]
  finance_report_effect: Disputed amounts shown separately; credits reduce cost.
  i18n_a11y: Messaging thread accessible and bilingual.
  acceptance: AC-SF21.3.5 — a damaged-goods dispute holds OMR 40 of the invoice while the rest is payable; resolution with credit note releases the hold.
  dependency: SF20.1.5.
- id: M21.F21.3.SF21.3.6
  name: evidence-to-payout gate
  phase: 4
  release: R1
  actors: [ap_worker, payment_releaser, auditor]
  screens: [SCR-FIN-payable-detail]
  inputs: [payable_id]
  states: [gate_passed, gate_failed]
  api: GET /v1/properties/{pid}/payables/{id}/payout-gate
  events: [PayoutGateFailed]
  data: [payable, invoice_match, service_acceptance, goods_receipt, emergency_purchase]
  rules: ["Payment requires approved source (requisition/award/call-off/ratified emergency), accepted evidence, matched invoice or approved exception, and approved payable", "No invoice paid solely because a PO exists (M49 SF49.3.7)", "Gate result stored with the payment order"]
  security: evaluated server-side at payment order creation and release
  failure_cases: [evidence_revoked_after_approval, emergency_not_ratified]
  finance_report_effect: Prevents pay-without-approved-evidence (Section P invariant).
  i18n_a11y: Gate failure explains missing item.
  acceptance: AC-SF21.3.6 — a payable whose service acceptance is reversed after approval fails the gate at release.
  dependency: SF20.5.1.
```

**M21 key invariants:** no PO without approved source; min-quote policy version locked at RFQ open; SoD pairs enforced server-side; emergency purchases capped and ratified before payment; pay only accepted, matched quantities; RFQ/PO/receipt mechanics are not duplicated outside M49/M50.

**M21 module acceptance (Section G):** AT-G04.2 gas-cylinder exchange and maintenance materials flow requisition → PO → receipt/return/service → invoice → AP; AT-G10.4 RFQ to several eligible suppliers, service acceptance and invoice reconciliation (with M46/M49); AT-G17.1 minimum-quote exception path; AT-G18.4 three-way match once only; AT-G20.5 short delivery surfaced as claim.

**M21 open decisions**

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-314 | Procurement value bands, minimum quotes and sourcing routes per category (shared with M49) | financial_controller + procurement_approver | < 50: catalog/direct; 50–2,000: 3 quotes; > 2,000: formal RFQ (base currency); food and cylinders via blanket where available. |
| D-315 | Use of blanket agreements for recurring supplies (cylinders, linen, cleaning, produce) | procurement_approver | Allowed for recurring categories with 12-month maximum and value cap. |
| D-316 | SoD for small hotels with few finance staff | owner + financial_controller | Time-bound exceptions allowed with weekly owner review of all overlapped transactions. |
| D-317 | Emergency purchase cap and ratification window | gm + financial_controller | Cap 1,000 base currency; ratification within 24 hours. |
| D-318 | Capitalization threshold and asset classes (shared with M66) | financial_controller | Items ≥ 500 base currency with useful life > 1 year capitalized. |

---
## 6. M22 — Electricity

| Header | Value |
|---|---|
| Purpose | Electricity accounts, master and submeters, BMS/API/CSV ingestion with interval, demand and quality handling, tariffs and peak windows, bill capture and usage-vs-bill reconciliation, allocation and estimates, AP approval, authorized provider/bank payment, settlement and late-fee exceptions, anomaly monitoring. Hosts the **cross-utility guard** (F22.3) that M23, M24 and M25 also obey. |
| Phases | Accounts, meters, bills, allocation and AP Phase 4; provider bill-pay via M29 Phase 5; site pilots Phase 6. |
| Release | R1 |
| Bounded context | `utilities` |
| System of record | `utility_account`, `service_connection`, `meter`, `meter_register`, `meter_reading`, `meter_interval`, `meter_event`, `tariff_version`, `tariff_component`, `utility_bill`, `utility_bill_line`, `usage_reconciliation`, `utility_allocation`, `consumption_baseline`, `consumption_anomaly` |
| Dependencies | M46 (utility provider as vendor), M20 (payable), M28/M29 (payment), M19 (accrual/allocation), M26 (anomaly work orders), M64 (BMS device registry), M67 (sustainability), M32 (utility reports), M38 (tax on bills) |

### F22.1 Electricity data
Story: As chief engineer I trust meter data because gaps, resets and time zones are handled explicitly and I can compare usage with the bill.

```yaml
- id: M22.F22.1.SF22.1.1
  name: utility account/master/submeter registry
  phase: 4
  release: R1
  actors: [chief_engineer, finance_clerk, property_admin]
  screens: [SCR-ENG-meter-map, SCR-FIN-utility-accounts]
  inputs: [utility_type, provider_vendor_id, account_number, premise_id, service_address, meter_serial, meter_role, parent_meter_id, multiplier, unit, served_cost_centers, install_date]
  states: [planned, active, faulty, replaced, retired]
  api: POST /v1/properties/{pid}/utility-accounts; POST /v1/properties/{pid}/meters; PATCH /v1/properties/{pid}/meters/{id}
  events: [UtilityAccountRegistered, MeterRegistered, MeterReplaced]
  data: [utility_account, service_connection, meter, meter_register]
  rules: ["Utility types electricity, water, wastewater, pipeline_gas share this registry", "Meter hierarchy master > submeters; sum of submeters vs master shows unmetered/common load", "Meter replacement closes old register with final read and opens new with initial read", "Account number unique per provider"]
  security: engineering edits meters; finance edits account/payment fields; audit
  failure_cases: [duplicate_account, hierarchy_cycle, replacement_without_final_read]
  finance_report_effect: Defines cost-center mapping for allocation drivers.
  i18n_a11y: Meter map has list alternative; bilingual labels.
  acceptance: AC-SF22.1.1 — registering a master meter with three submeters shows unmetered share; replacing a submeter preserves continuity of consumption across registers.
  dependency: D-319 supplier/account format; M64 device registry.
- id: M22.F22.1.SF22.1.2
  name: BMS/API/CSV import
  phase: 4
  release: R1
  actors: [integration_admin, chief_engineer, meter_ingest_worker]
  screens: [SCR-ADM-integrations, SCR-ENG-meter-readings]
  inputs: [source_type, meter_id, timestamp, value, unit, quality_flag, file, mapping_profile]
  states: [received, parsed, validated, rejected, quarantined]
  api: POST /v1/properties/{pid}/meter-readings:batch (Idempotency-Key); POST /v1/properties/{pid}/meter-readings:import-csv
  events: [MeterReadingsIngested, MeterReadingRejected]
  data: [meter_reading, meter_interval, ingest_batch]
  rules: ["Adapters: BMS gateway push, provider/smart-meter API where contracted, CSV upload, manual mobile read with photo", "Dedup on meter_id plus timestamp plus register", "Unit conversion to canonical kWh and kW", "Raw payload retained for audit"]
  security: device identity and mTLS for gateway push; CSV scanned; integration role
  failure_cases: [unknown_meter, unit_mismatch, duplicate_batch, clock_skew]
  finance_report_effect: Feeds usage reconciliation and allocation drivers.
  i18n_a11y: Import errors listed with row numbers and plain-language reason.
  acceptance: AC-SF22.1.2 — importing the same CSV twice creates no duplicate readings; a manual read with photo is accepted from offline mobile after sync.
  dependency: D-320 BMS/meter protocol; M64.F64.1.
- id: M22.F22.1.SF22.1.3
  name: cumulative and interval kWh, kW demand and quality
  phase: 4
  release: R1
  actors: [chief_engineer, meter_ingest_worker]
  screens: [SCR-ENG-meter-readings]
  inputs: [register_type, cumulative_value, interval_start, interval_end, demand_kw, power_factor, quality_flag]
  states: [actual, estimated, substituted, suspect]
  api: GET /v1/properties/{pid}/meters/{id}/consumption?from=&to=&interval=
  events: [ConsumptionComputed, MeterDataQualityFlagged]
  data: [meter_reading, meter_interval]
  rules: ["Consumption derived from cumulative deltas times multiplier; interval data validated against cumulative", "Peak demand per billing period computed from interval kW", "Every value carries quality actual/estimated/substituted/suspect", "Negative delta without reset event is suspect"]
  security: read by engineering and finance
  failure_cases: [negative_delta, spike_outlier, interval_sum_mismatch]
  finance_report_effect: Quality flag propagates to estimate label in reports.
  i18n_a11y: Charts with tabular alternative and units spoken.
  acceptance: AC-SF22.1.3 — interval sum within 1 percent of cumulative delta passes; a 10x spike is flagged suspect and excluded from allocation until reviewed.
  dependency: SF22.1.2.
- id: M22.F22.1.SF22.1.4
  name: reset/gap/time-zone handling
  phase: 4
  release: R1
  actors: [chief_engineer, meter_ingest_worker]
  screens: [SCR-ENG-meter-readings, SCR-OPS-exception-queue]
  inputs: [meter_id, event_type, rollover_max, gap_start, gap_end, estimation_method]
  states: [gap_open, estimated, filled_by_actual, reset_confirmed]
  api: POST /v1/properties/{pid}/meters/{id}/events; POST /v1/properties/{pid}/meters/{id}/gaps/{gap}:estimate
  events: [MeterGapDetected, MeterResetRecorded, MeterGapFilled]
  data: [meter_event, meter_interval]
  rules: ["Timestamps stored UTC and displayed in property time zone; DST transitions do not double count or drop intervals", "Rollover detected when delta negative near register max", "Missing intervals raise gap; estimation by linear or profile method is labelled estimated and replaced when actual arrives"]
  security: estimation actions audited
  failure_cases: [missed_interval, meter_reset_unannounced, dst_duplicate_hour]
  finance_report_effect: Estimated consumption flows to accrual with estimate label.
  i18n_a11y: Gap timeline accessible.
  acceptance: AC-SF22.1.4 — a missed 6-hour interval raises MeterGapDetected, is estimated, and is replaced by actual when late data arrives (AT-G20.3).
  dependency: SF22.1.3.
- id: M22.F22.1.SF22.1.5
  name: tariff version and peak windows
  phase: 4
  release: R1
  actors: [finance_clerk, financial_controller, chief_engineer]
  screens: [SCR-FIN-tariffs]
  inputs: [provider_id, tariff_code, components, energy_rate_bands, demand_charge, fixed_charge, peak_windows, season, tax_code, effective_from, source_document]
  states: [draft, verified, active, superseded]
  api: POST /v1/properties/{pid}/tariffs; POST /v1/properties/{pid}/tariffs/{id}:verify
  events: [TariffVersionActivated]
  data: [tariff_version, tariff_component]
  rules: ["Tariff entered from published schedule or contract with source document and verifier", "Unverified tariff produces estimates only", "Peak windows in property local time with weekday/season rules", "Version effective-dated; bills compare against the version valid in the bill period"]
  security: maker-checker on verification
  failure_cases: [overlapping_versions, missing_source, tariff_change_mid_period]
  finance_report_effect: Basis of expected-bill and accrual values.
  i18n_a11y: Tariff table accessible.
  acceptance: AC-SF22.1.5 — a tariff change mid-period prorates expected cost by days; an unverified tariff marks expected bill as estimate.
  dependency: D-321.
- id: M22.F22.1.SF22.1.6
  name: usage vs invoice reconciliation
  phase: 4
  release: R1
  actors: [finance_clerk, chief_engineer, recon_worker]
  screens: [SCR-FIN-utility-bill-detail, SCR-OPS-exception-queue]
  inputs: [utility_bill_id, meter_ids, period, tolerance_profile]
  states: [pending, matched, variance_within_tolerance, variance_exception, explained]
  api: POST /v1/properties/{pid}/utility-bills/{id}:reconcile-usage
  events: [UsageReconciled, UtilityBillVarianceDetected]
  data: [usage_reconciliation, utility_bill, meter_interval, tariff_version]
  rules: ["Compare billed kWh and kW to metered for same period and account", "Compare billed amount to tariff-computed expected amount", "Variance above tolerance routes to exception with chief_engineer and finance", "Explained variance requires reason (estimated provider read, tariff change, meter fault)"]
  security: finance and engineering roles
  failure_cases: [period_misaligned, provider_estimated_read, meter_fault]
  finance_report_effect: Validation stage of the backbone; variance reported in utility report.
  i18n_a11y: Variance explanation fields bilingual.
  acceptance: AC-SF22.1.6 — a bill 8 percent above metered expected with 3 percent tolerance creates UtilityBillVarianceDetected and blocks auto-approval (AT-G20.3).
  dependency: D-322.
```

### F22.2 Electricity finance
Story: As AP clerk I process the electricity bill through the backbone and pay only via an authorized channel.

```yaml
- id: M22.F22.2.SF22.2.1
  name: PDF/invoice capture and supplier ID
  phase: 4
  release: R1
  actors: [ap_clerk, invoice_ocr_worker]
  screens: [SCR-FIN-utility-bill-inbox]
  inputs: [pdf_or_structured_bill, provider_vendor_id, account_number, bill_number, period_start, period_end, readings_on_bill]
  states: [received, captured, validated, rejected]
  api: POST /v1/properties/{pid}/utility-bills (Idempotency-Key)
  events: [UtilityBillCaptured]
  data: [utility_bill, utility_bill_line, document_attachment]
  rules: ["Bill resolves to one utility_account; unknown account raises exception", "Unique key provider plus account plus bill number", "Bill lines typed as energy, demand, fixed, tax, arrears, late fee, adjustment"]
  security: AP scope
  failure_cases: [unknown_account, duplicate_bill, period_overlap]
  finance_report_effect: Invoiced measure once validated.
  i18n_a11y: Arabic bill OCR supported where engine allows; manual entry fallback accessible.
  acceptance: AC-SF22.2.1 — an imported bill for an unregistered account is rejected into exception; a repeated bill number is rejected.
  dependency: SF20.1.2, SF20.1.3.
- id: M22.F22.2.SF22.2.2
  name: tax and due date
  phase: 4
  release: R1
  actors: [ap_clerk, finance_clerk]
  screens: [SCR-FIN-utility-bill-detail]
  inputs: [tax_lines, due_date, late_fee_terms, arrears_amount]
  states: [due, due_soon, overdue, paid]
  api: PATCH /v1/properties/{pid}/utility-bills/{id}
  events: [UtilityBillDueSoon, UtilityBillOverdue]
  data: [utility_bill, payable]
  rules: ["Tax codes validated against M38; recoverable input tax separated", "Due-date reminders at configurable lead times", "Arrears on a bill are reconciled to prior unpaid bills, not treated as new expense"]
  security: AP scope
  failure_cases: [tax_code_unverified, arrears_double_count]
  finance_report_effect: Correct expense vs input tax; cash forecast outflow.
  i18n_a11y: Due-date alerts bilingual.
  acceptance: AC-SF22.2.2 — a bill showing arrears equal to last month's unpaid bill does not create additional expense.
  dependency: M38.F38.1.
- id: M22.F22.2.SF22.2.3
  name: cost allocation/estimate
  phase: 4
  release: R1
  actors: [finance_clerk, financial_controller, ledger_worker]
  screens: [SCR-FIN-allocation-run, SCR-FIN-accruals]
  inputs: [period, allocation_policy_version, submeter_consumption, estimated_usage, tariff_version]
  states: [estimated, allocated, trued_up]
  api: POST /v1/properties/{pid}/utility-allocations:run?period=
  events: [UtilityCostAllocated, UtilityAccrualPosted]
  data: [utility_allocation, allocation_run, accrual_schedule]
  rules: ["Before bill arrives accrue metered or estimated usage times tariff", "On bill posting, true-up difference and reverse accrual", "Allocation to cost centers by submeter share then policy driver for common load"]
  security: finance roles
  failure_cases: [bill_period_spans_months, submeter_gap]
  finance_report_effect: Department utility cost in P&L; estimate vs actual label.
  i18n_a11y: Allocation table accessible.
  acceptance: AC-SF22.2.3 — kitchen submeter 30 percent, laundry 20 percent, remainder by area; P&L shows kitchen electricity with source drill to meter (AT-G08.2).
  dependency: SF19.3.2, SF19.1.4.
- id: M22.F22.2.SF22.2.4
  name: AP approval
  phase: 4
  release: R1
  actors: [finance_approver, chief_engineer]
  screens: [SCR-FIN-payable-approval]
  inputs: [utility_bill_id, reconciliation_status, approver_decision]
  states: [pending_approval, approved, held]
  api: POST /v1/properties/{pid}/payables/{id}:approve
  events: [PayableApproved]
  data: [payable, usage_reconciliation]
  rules: ["Approval requires usage reconciliation matched or variance explained", "Engineering concurrence for variance above tolerance", "Delegation of authority applies"]
  security: SoD
  failure_cases: [variance_unexplained]
  finance_report_effect: Moves to approved.
  i18n_a11y: Approval card shows metered vs billed in text.
  acceptance: AC-SF22.2.4 — an unexplained variance bill cannot be approved.
  dependency: SF20.1.5.
- id: M22.F22.2.SF22.2.5
  name: authorized provider/bank payment
  phase: 4
  release: R1
  actors: [payment_releaser, finance_approver, billpay_worker]
  screens: [SCR-FIN-payment-batch, SCR-FIN-bill-payment-detail]
  inputs: [payable_id, channel, bill_account_id, provider_id]
  states: [channel_selected, ordered, pending, confirmed, failed]
  api: POST /v1/properties/{pid}/payables/{id}:pay (routes to M28 payment_order or M29 bill_payment_order)
  events: [UtilityPaymentOrdered, UtilityPaymentConfirmed]
  data: [payment_order, bill_payment_order, payable]
  rules: ["Channel is bank transfer via M28.F28.2 (Phase 4) or contracted bill provider via M29 (Phase 5)", "Khedmah/ONEIC channel selectable only when adapter status is partner-contracted or better; otherwise shown as blocked", "Cross-utility guard F22.3 applies"]
  security: releaser distinct from approver; step-up MFA
  failure_cases: [provider_blocked, bank_rejection, timeout]
  finance_report_effect: Approved to paid on confirmation.
  i18n_a11y: Channel status labels explicit (available, blocked - no contract).
  acceptance: AC-SF22.2.5 — with no Khedmah contract, the provider channel is shown blocked and the bill is paid by bank transfer with reference recorded (AT-G06.3).
  dependency: M28.F28.2; M29; D-346, D-347.
- id: M22.F22.2.SF22.2.6
  name: settlement/late fee and exception
  phase: 4
  release: R1
  actors: [finance_clerk, financial_controller]
  screens: [SCR-FIN-bank-reconciliation, SCR-OPS-exception-queue]
  inputs: [payment_ref, provider_receipt, next_bill_arrears, late_fee_line]
  states: [awaiting_settlement, settled, late_fee_disputed, closed]
  api: POST /v1/properties/{pid}/utility-bills/{id}:settlement-check
  events: [UtilityBillSettled, LateFeeDetected]
  data: [utility_bill, reconciliation_match, provider_receipt]
  rules: ["Settled when bank/provider settlement matched and next bill shows no arrears for that amount", "Late fee on next bill links to payment timing; if paid on time with evidence, dispute raised", "Late fees post to a distinct account for reporting"]
  security: finance roles
  failure_cases: [payment_not_applied_by_provider, late_fee_after_timely_payment]
  finance_report_effect: Settled measure; late-fee KPI.
  i18n_a11y: Exception text clear.
  acceptance: AC-SF22.2.6 — a late fee on the next bill after on-time payment opens a provider dispute with payment evidence attached.
  dependency: F22.3.
```

### F22.3 Cross-utility guard (Section K cross-utility guard — applies to M22, M23, M24, M25 and M29)
Story: As financial controller I never mark a utility paid unless the provider or bank confirms it against the exact account and bill.

```yaml
- id: M22.F22.3.SF22.3.1
  name: provider confirmation and bill binding
  phase: 4
  release: R1
  actors: [billpay_worker, treasury_worker, finance_clerk]
  screens: [SCR-FIN-bill-payment-detail]
  inputs: [utility_bill_id, bill_account_id, provider_bill_ref, payment_ref, confirmation_ref]
  states: [unconfirmed, confirmed, confirmation_mismatch]
  api: POST /v1/properties/{pid}/utility-payments/{id}/confirmations
  events: [UtilityPaymentConfirmationRecorded, UtilityPaymentConfirmationMismatch]
  data: [utility_payment_link, provider_receipt, payment_order, bill_payment_order]
  rules: ["Payment success cannot be inferred from a gateway charge, card capture or bank debit alone", "Confirmation must carry provider/bank reference and the same account and bill identifiers", "Mismatch of account or amount opens exception and keeps payable unpaid"]
  security: confirmations only from signed callbacks, settlement files or approved manual evidence
  failure_cases: [gateway_success_provider_unknown, account_mismatch, amount_mismatch]
  finance_report_effect: Guards the paid measure for all utilities.
  i18n_a11y: Confirmation details shown as labelled fields.
  acceptance: AC-SF22.3.1 — a card capture for a water bill without provider receipt leaves the bill approved-in-flight, not paid.
  dependency: M29.F29.1 SF29.1.8.
- id: M22.F22.3.SF22.3.2
  name: timeout stays pending until inquiry
  phase: 4
  release: R1
  actors: [billpay_worker, treasury_worker, finance_approver]
  screens: [SCR-FIN-bill-payment-detail, SCR-OPS-exception-queue]
  inputs: [order_id, idempotency_key, inquiry_schedule]
  states: [pending, inquiry_in_progress, confirmed, failed_confirmed, manual_review]
  api: POST /v1/properties/{pid}/utility-payments/{id}:inquire
  events: [UtilityPaymentPendingAged, UtilityPaymentInquiryCompleted]
  data: [bill_payment_order, payment_order, provider_status_log]
  rules: ["A timeout sets pending; no new payment attempt may be created for the same bill while pending", "Inquiry by idempotency key or order reference precedes any retry", "Failed only when provider or bank affirmatively reports failure", "Pending beyond SLA escalates to manual review"]
  security: retry action disabled in UI and API while pending
  failure_cases: [lost_callback, provider_outage, ambiguous_status]
  finance_report_effect: Prevents double payment and false paid status.
  i18n_a11y: Pending state explained in plain language with next check time.
  acceptance: AC-SF22.3.2 — after simulated timeout, a user retry is rejected with PAYMENT_PENDING_INQUIRY_REQUIRED; inquiry returns success and the bill is paid once (AT-G06.4).
  dependency: M29.F29.1 SF29.1.7; M28.F28.2 SF28.2.4.
- id: M22.F22.3.SF22.3.3
  name: approved alternate payment evidence
  phase: 4
  release: R1
  actors: [finance_approver, payment_releaser, auditor]
  screens: [SCR-FIN-bill-payment-detail]
  inputs: [utility_bill_id, alternate_channel, bank_reference, receipt_file, evidence_approver]
  states: [manual_pending, evidence_submitted, evidence_approved, rejected]
  api: POST /v1/properties/{pid}/utility-payments/{id}:manual-evidence
  events: [ManualUtilityPaymentEvidenceApproved]
  data: [utility_payment_link, document_attachment]
  rules: ["Manual/bank path used when provider adapter is blocked or down", "Evidence approver distinct from submitter", "Manual paid still requires settlement match to become settled"]
  security: evidence immutable; audit
  failure_cases: [evidence_forged_suspected, duplicate_manual_and_provider_payment]
  finance_report_effect: Paid on approved evidence; settled after bank match.
  i18n_a11y: Upload accessible with file type guidance.
  acceptance: AC-SF22.3.3 — a bank-paid electricity bill with receipt approved by second user becomes paid and settles on statement import.
  dependency: SF29.3.2.
- id: M22.F22.3.SF22.3.4
  name: utility payment reconciliation across provider, bank and GL
  phase: 5
  release: R1
  actors: [finance_clerk, recon_worker, financial_controller]
  screens: [SCR-FIN-utility-reconciliation]
  inputs: [period, provider_settlement_files, bank_statement_lines, gl_accounts]
  states: [open, matched, exception, closed]
  api: POST /v1/properties/{pid}/utility-reconciliations?period=
  events: [UtilityReconciliationCompleted, UtilityReconciliationException]
  data: [utility_bill, bill_payment_order, provider_settlement, reconciliation_match, journal_line]
  rules: ["Four-way tie out: utility bill, payment order/bill payment order, provider/bank settlement, GL clearing account", "Clearing accounts must be zero or explained at close"]
  security: finance roles
  failure_cases: [settlement_file_missing, clearing_not_zero]
  finance_report_effect: Close checklist item for utilities.
  i18n_a11y: Reconciliation table accessible.
  acceptance: AC-SF22.3.4 — month-end for electricity, water and gas shows zero clearing balance and every bill settled or listed as exception.
  dependency: SF29.3.3; SF20.3.2.
```

### F22.4 Energy monitoring and anomalies (added — Section C "monitoring/anomalies")
Story: As chief engineer I get alerted to abnormal energy use and can act on it.

```yaml
- id: M22.F22.4.SF22.4.1
  name: consumption baseline and anomaly detection
  phase: 4
  release: R1
  actors: [chief_engineer, anomaly_worker, gm]
  screens: [SCR-ENG-energy-monitor, SCR-GM-home]
  inputs: [meter_id, baseline_method, occupancy_ref, weather_ref, threshold]
  states: [normal, anomaly_open, acknowledged, resolved, false_positive]
  api: GET /v1/properties/{pid}/consumption-anomalies; POST /v1/properties/{pid}/consumption-anomalies/{id}:acknowledge
  events: [ConsumptionAnomalyDetected, ConsumptionAnomalyResolved]
  data: [consumption_baseline, consumption_anomaly]
  rules: ["Baseline normalized by occupied rooms and optional weather degree-days", "Anomaly requires sustained deviation, not a single suspect reading", "Applies to electricity, water and pipeline gas meters", "Statistical, explainable method; no autonomous equipment control"]
  security: engineering scope
  failure_cases: [insufficient_history, data_quality_suspect]
  finance_report_effect: Utility per occupied room KPI (M32 SF32.4.4, M67).
  i18n_a11y: Alerts bilingual; chart with table.
  acceptance: AC-SF22.4.1 — a 40 percent overnight rise sustained 3 hours on the kitchen submeter raises one anomaly, not one per interval.
  dependency: M67.F67.1.
- id: M22.F22.4.SF22.4.2
  name: anomaly work order and sustainability feed
  phase: 4
  release: R1
  actors: [chief_engineer, engineer]
  screens: [SCR-ENG-work-orders]
  inputs: [anomaly_id, work_order_template]
  states: [linked, work_order_closed]
  api: POST /v1/properties/{pid}/consumption-anomalies/{id}:create-work-order
  events: [AnomalyWorkOrderCreated]
  data: [consumption_anomaly, work_order]
  rules: ["One work order per anomaly; closure evidence closes anomaly", "Resolved anomalies and savings estimates feed M67 SF67.2.2"]
  security: engineering scope
  failure_cases: [duplicate_work_order]
  finance_report_effect: Maintenance cost links to anomaly for savings measurement.
  i18n_a11y: Standard work-order accessibility.
  acceptance: AC-SF22.4.2 — creating a work order twice for one anomaly returns the existing one.
  dependency: F26.1; M67.F67.2.
```

**M22 key invariants:** readings deduplicated and quality-flagged; estimates labelled and replaced by actuals; unverified tariffs produce estimates only; bill cannot be approved with unexplained variance; utility paid only with provider/bank confirmation bound to account and bill; timeout never triggers a blind retry.

**M22 module acceptance (Section G):** AT-G04.3 electricity bill imported, metered, reconciled and approved; AT-G06.3 bill paid via authorized adapter or approved alternate with partner API marked blocked; AT-G06.4 timeout plus duplicate callback settles exactly one payable; AT-G08.2 shared energy allocation drills to meter; AT-G20.3 missed meter interval and utility bill mismatch surfaced.

**M22 open decisions**

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-319 | Pilot electricity supplier, account/bill format and whether structured bills are available | financial_controller | PDF bill with manual/OCR capture; one account per premise. |
| D-320 | BMS/meter models and protocol (Modbus/BACnet via gateway, vendor API, CSV) | chief_engineer + it_admin | CSV export or gateway push of 15-minute intervals; manual monthly read fallback. |
| D-321 | Tariff source and verification owner per provider | financial_controller | Finance enters published tariff with source document; chief_engineer verifies. |
| D-322 | Usage-vs-bill variance tolerance per utility | financial_controller + chief_engineer | 3% quantity, 2% amount. |

---

## 7. M23 — Water

| Header | Value |
|---|---|
| Purpose | Water supply and wastewater accounts, master/submeters or bill import, supply/sewage components, usage/period/rates, leak and abnormal use to work order, due dates, provider inquiry/payment/reconciliation, departmental allocation. Reuses the M22 registry, ingestion, tariff and anomaly services with `utility_type = water / wastewater`. |
| Phases | 4 (bills, meters, allocation, AP); 5 (provider inquiry/payment via M29). |
| Release | R1 |
| Bounded context | `utilities` |
| System of record | shares M22 entities; adds `wastewater_basis` on `tariff_component` and `leak_case` |
| Dependencies | M22 (shared services, F22.3 guard), M26 (leak work orders), M42 (flood incident), M61 (water safety checks), M20, M28, M29, M67 |

### F23.1 Water

```yaml
- id: M23.F23.1.SF23.1.1
  name: account and service address
  phase: 4
  release: R1
  actors: [finance_clerk, chief_engineer]
  screens: [SCR-FIN-utility-accounts]
  inputs: [provider_vendor_id, account_number, service_address, premise_id, supply_type, wastewater_included]
  states: [active, suspended, closed]
  api: POST /v1/properties/{pid}/utility-accounts (utility_type=water)
  events: [UtilityAccountRegistered]
  data: [utility_account, service_connection]
  rules: ["Supply and wastewater may be one account or separate providers", "Account number format validated per provider pattern where known", "Bill-pay registration of this account in M29 is scoped to hotel operating bills"]
  security: finance and engineering roles
  failure_cases: [duplicate_account, address_mismatch]
  finance_report_effect: Account-to-cost-center mapping.
  i18n_a11y: Arabic addresses supported; RTL form.
  acceptance: AC-SF23.1.1 — a water account with separate wastewater provider produces two linked accounts on one service connection.
  dependency: D-323.
- id: M23.F23.1.SF23.1.2
  name: meter/manual/provider bill
  phase: 4
  release: R1
  actors: [engineer, ap_clerk, meter_ingest_worker]
  screens: [SCR-ENG-meter-readings, SCR-FIN-utility-bill-inbox]
  inputs: [meter_id, reading_value, reading_photo, bill_file, bill_number, period]
  states: [read, bill_captured, validated]
  api: POST /v1/properties/{pid}/meter-readings:batch; POST /v1/properties/{pid}/utility-bills
  events: [MeterReadingsIngested, UtilityBillCaptured]
  data: [meter_reading, utility_bill]
  rules: ["Where no hotel meter exists, bill consumption is the usage source and labelled provider-read", "Manual readings require photo and reader identity", "Same dedup and uniqueness as M22"]
  security: engineering and AP scopes
  failure_cases: [unreadable_photo, provider_estimated_bill]
  finance_report_effect: Usage and invoiced amounts.
  i18n_a11y: Mobile reading form with camera guidance and large controls.
  acceptance: AC-SF23.1.2 — a manual water read with photo syncs after offline capture; a bill without hotel meter shows usage source provider-read.
  dependency: F22.1.
- id: M23.F23.1.SF23.1.3
  name: supply/wastewater components
  phase: 4
  release: R1
  actors: [finance_clerk]
  screens: [SCR-FIN-utility-bill-detail, SCR-FIN-tariffs]
  inputs: [supply_volume, supply_rate, wastewater_basis, wastewater_rate, fixed_charges, tax_lines]
  states: [captured, validated]
  api: PATCH /v1/properties/{pid}/utility-bills/{id}/lines
  events: [UtilityBillLinesValidated]
  data: [utility_bill_line, tariff_component]
  rules: ["Wastewater may be billed as percent of supply volume or separately metered; basis stored on tariff", "Lines map to separate GL accounts for supply and sewage", "Expected amount computed from tariff for comparison"]
  security: finance scope
  failure_cases: [basis_unknown, component_missing]
  finance_report_effect: Separate water and sewage expense lines.
  i18n_a11y: Line table accessible.
  acceptance: AC-SF23.1.3 — wastewater at 80 percent of supply volume computes expected sewage charge and matches bill within tolerance.
  dependency: SF22.1.5.
- id: M23.F23.1.SF23.1.4
  name: leak/anomaly work order
  phase: 4
  release: R1
  actors: [chief_engineer, engineer, duty_manager, anomaly_worker]
  screens: [SCR-ENG-energy-monitor, SCR-ENG-work-orders]
  inputs: [meter_id, night_flow_threshold, anomaly_id]
  states: [suspected_leak, work_order_open, confirmed_leak, resolved, false_positive]
  api: POST /v1/properties/{pid}/leak-cases
  events: [LeakSuspected, LeakConfirmed, LeakResolved]
  data: [leak_case, consumption_anomaly, work_order]
  rules: ["Minimum night flow above threshold for configured hours signals suspected leak", "Suspected leak creates work order; confirmed major leak can open M42 incident", "Leak allowance claims to provider tracked with evidence"]
  security: engineering scope; incident escalation per M42
  failure_cases: [no_night_baseline, alarm_fatigue]
  finance_report_effect: Leak volume cost reported separately.
  i18n_a11y: Alert bilingual and audible on staff app where enabled.
  acceptance: AC-SF23.1.4 — continuous flow of 0.5 m3/h from 01:00 to 05:00 creates one leak case and work order.
  dependency: D-324; F26.1; M42.
- id: M23.F23.1.SF23.1.5
  name: estimate/actual reconciliation
  phase: 4
  release: R1
  actors: [finance_clerk, recon_worker]
  screens: [SCR-FIN-utility-bill-detail]
  inputs: [period, accrual_id, utility_bill_id]
  states: [estimated, actual_received, trued_up]
  api: POST /v1/properties/{pid}/utility-bills/{id}:reconcile-usage
  events: [UsageReconciled, UtilityAccrualTrueUp]
  data: [usage_reconciliation, accrual_schedule]
  rules: ["Provider estimated bills are flagged and later corrected bills reconcile the difference", "Hotel accrual trued up on actual"]
  security: finance scope
  failure_cases: [provider_estimate_corrected_later]
  finance_report_effect: Estimate vs actual label transitions.
  i18n_a11y: Label text explicit.
  acceptance: AC-SF23.1.5 — two provider-estimated months followed by an actual bill produce a true-up equal to cumulative difference.
  dependency: SF22.1.6.
- id: M23.F23.1.SF23.1.6
  name: payable/payment and departmental allocation
  phase: 4
  release: R1
  actors: [ap_clerk, finance_approver, payment_releaser]
  screens: [SCR-FIN-payable-approval, SCR-FIN-bill-payment-detail]
  inputs: [utility_bill_id, channel, allocation_policy_version]
  states: [approved, pending, paid, settled]
  api: POST /v1/properties/{pid}/payables/{id}:pay
  events: [UtilityPaymentOrdered, UtilityCostAllocated]
  data: [payable, bill_payment_order, payment_order, utility_allocation]
  rules: ["Follows backbone and F22.3 guard", "Provider inquiry/payment via M29 only where contracted; otherwise bank path", "Allocation to rooms/laundry/kitchen/pool by submeter or policy"]
  security: SoD
  failure_cases: [provider_blocked, timeout]
  finance_report_effect: Department water cost in P&L.
  i18n_a11y: Standard.
  acceptance: AC-SF23.1.6 — water bill inquiry via mock adapter returns amount; production path uses bank payment while adapter is blocked; allocation shows laundry share.
  dependency: M29; F22.3.
```

**M23 key invariants:** as M22; wastewater basis explicit; provider-estimated bills never labelled actual.
**M23 module acceptance (Section G):** AT-G04.3 (water bill imported/approved), AT-G06.3, AT-G20.3.

**M23 open decisions**

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-323 | Water/wastewater provider structure and billing basis at pilot | financial_controller | Single provider, wastewater as a percentage of supply volume. |
| D-324 | Leak detection thresholds and night window | chief_engineer | Night flow > 20% of daytime average for 3 consecutive hours. |

---

## 8. M24 — Pipeline gas

| Header | Value |
|---|---|
| Purpose | Pipeline gas supplier/account/connection, meter or billed consumption, tariff with standing, minimum and variable fees, invoice variance, kitchen/laundry cost drivers, payable/payment, and safety-related fault/maintenance reference. |
| Phases | 4 |
| Release | R1 (enabled only if the property has a piped gas supply) |
| Bounded context | `utilities` |
| System of record | shares M22 entities (`utility_type = pipeline_gas`, units m3/kWh/MMBtu with calorific conversion) plus `gas_safety_reference`, `gas_connection_certificate` |
| Dependencies | M22 (F22.3 guard), M26 (maintenance), M42 (gas incident playbook), M61 (inspection), M20, M28, M29 |

### F24.1 Pipeline gas

```yaml
- id: M24.F24.1.SF24.1.1
  name: supplier/account/connection
  phase: 4
  release: R1
  actors: [finance_clerk, chief_engineer]
  screens: [SCR-FIN-utility-accounts, SCR-ENG-meter-map]
  inputs: [provider_vendor_id, account_number, connection_id, pressure_class, served_areas, contract_ref]
  states: [active, isolated, closed]
  api: POST /v1/properties/{pid}/utility-accounts (utility_type=pipeline_gas)
  events: [UtilityAccountRegistered, GasConnectionIsolated]
  data: [utility_account, service_connection]
  rules: ["Connection records served areas (kitchen, laundry, boilers) for allocation", "Isolation state set only via safety reference (SF24.2.1)"]
  security: engineering scope for connection; finance for account
  failure_cases: [duplicate_account]
  finance_report_effect: Allocation basis.
  i18n_a11y: Standard bilingual forms.
  acceptance: AC-SF24.1.1 — a gas connection serving kitchen and laundry appears with both served areas and allocation defaults.
  dependency: D-325.
- id: M24.F24.1.SF24.1.2
  name: meter or billed consumption
  phase: 4
  release: R1
  actors: [engineer, meter_ingest_worker, ap_clerk]
  screens: [SCR-ENG-meter-readings]
  inputs: [meter_id, volume, unit, calorific_value, correction_factor, bill_consumption]
  states: [actual, estimated, provider_read]
  api: POST /v1/properties/{pid}/meter-readings:batch
  events: [MeterReadingsIngested]
  data: [meter_reading, meter_interval]
  rules: ["Volume converted to energy using calorific value and correction factor from bill or contract", "Without hotel meter, billed consumption is the source labelled provider-read"]
  security: engineering scope
  failure_cases: [calorific_value_missing, unit_mismatch]
  finance_report_effect: Consumption basis for variance and allocation.
  i18n_a11y: Units spelled out for screen readers.
  acceptance: AC-SF24.1.2 — 1,000 m3 at calorific value 38.5 MJ/m3 converts to the correct kWh and matches bill energy within tolerance.
  dependency: F22.1.
- id: M24.F24.1.SF24.1.3
  name: tariff/standing and variable fees
  phase: 4
  release: R1
  actors: [finance_clerk, financial_controller]
  screens: [SCR-FIN-tariffs]
  inputs: [standing_charge, minimum_charge, variable_rate, capacity_charge, tax_code, effective_from]
  states: [draft, verified, active, superseded]
  api: POST /v1/properties/{pid}/tariffs (utility_type=pipeline_gas)
  events: [TariffVersionActivated]
  data: [tariff_version, tariff_component]
  rules: ["Minimum charge applies when consumption charge below minimum", "Standing charge prorated for partial periods"]
  security: maker-checker
  failure_cases: [minimum_charge_misapplied]
  finance_report_effect: Expected bill and accrual basis.
  i18n_a11y: Standard.
  acceptance: AC-SF24.1.3 — a low-use month bills the minimum charge and expected amount equals minimum plus standing plus tax.
  dependency: SF22.1.5.
- id: M24.F24.1.SF24.1.4
  name: invoice variance
  phase: 4
  release: R1
  actors: [finance_clerk, chief_engineer]
  screens: [SCR-FIN-utility-bill-detail]
  inputs: [utility_bill_id, tolerance_profile]
  states: [matched, variance_exception, explained]
  api: POST /v1/properties/{pid}/utility-bills/{id}:reconcile-usage
  events: [UtilityBillVarianceDetected]
  data: [usage_reconciliation]
  rules: ["Same variance workflow as SF22.1.6", "Unexplained sudden increase also checks for leak via SF24.2.1"]
  security: finance and engineering
  failure_cases: [period_misaligned]
  finance_report_effect: Validation stage.
  i18n_a11y: Standard.
  acceptance: AC-SF24.1.4 — a 25 percent billed increase with flat meter consumption raises a variance exception.
  dependency: SF22.1.6.
- id: M24.F24.1.SF24.1.5
  name: kitchen/laundry cost driver
  phase: 4
  release: R1
  actors: [financial_controller, fnb_manager]
  screens: [SCR-FIN-allocation-policy]
  inputs: [submeter_share, covers, laundry_kg, policy_version]
  states: [configured, allocated]
  api: POST /v1/properties/{pid}/utility-allocations:run?period=
  events: [UtilityCostAllocated]
  data: [utility_allocation, allocation_driver]
  rules: ["Submeter first; otherwise covers for kitchen and kg processed for laundry per policy", "Event/catering gas usage attributable to event when submetered or estimated with label"]
  security: finance
  failure_cases: [driver_data_missing]
  finance_report_effect: Kitchen and laundry gas cost in department P&L.
  i18n_a11y: Standard.
  acceptance: AC-SF24.1.5 — without submeters, gas cost splits by covers and laundry kg per policy and is labelled allocated-estimate.
  dependency: SF19.3.2.
- id: M24.F24.1.SF24.1.6
  name: payable/payment/maintenance alert
  phase: 4
  release: R1
  actors: [ap_clerk, finance_approver, payment_releaser, chief_engineer]
  screens: [SCR-FIN-payable-approval, SCR-ENG-work-orders]
  inputs: [utility_bill_id, channel, maintenance_flag]
  states: [approved, pending, paid, settled]
  api: POST /v1/properties/{pid}/payables/{id}:pay
  events: [UtilityPaymentOrdered, GasMaintenanceAlertRaised]
  data: [payable, payment_order, work_order]
  rules: ["Backbone and F22.3 guard apply", "Bill notices of meter inspection or maintenance create an engineering task"]
  security: SoD
  failure_cases: [bank_rejection]
  finance_report_effect: Paid/settled measures.
  i18n_a11y: Standard.
  acceptance: AC-SF24.1.6 — a bill with provider inspection notice creates one engineering task; payment follows bank path with confirmation reference.
  dependency: F22.3; F26.1.
```

### F24.2 Gas safety reference (added — Section C "safety-related fault/maintenance reference")

```yaml
- id: M24.F24.2.SF24.2.1
  name: safety fault reference and supply isolation record
  phase: 4
  release: R1
  actors: [chief_engineer, security_officer, duty_manager, engineer]
  screens: [SCR-ENG-gas-safety, SCR-OPS-incident]
  inputs: [connection_id, fault_type, detector_alarm_ref, incident_id, isolation_time, restoration_time, provider_emergency_ref]
  states: [reported, isolated, provider_notified, repaired, pressure_tested, restored]
  api: POST /v1/properties/{pid}/gas-safety-references; POST /v1/properties/{pid}/gas-safety-references/{id}:restore
  events: [GasSafetyFaultReported, GasSupplyIsolated, GasSupplyRestored]
  data: [gas_safety_reference, service_connection, work_order, incident_ref]
  rules: ["MetriStay records and links; life-safety response follows M42 playbook and local emergency procedures, never software alone", "Restoration requires competent-person test evidence", "Linked to work order and incident for insurance evidence (M68)"]
  security: restricted to engineering and duty management; immutable chronology
  failure_cases: [restoration_without_test, incident_not_linked]
  finance_report_effect: Repair costs link to connection asset; downtime reported.
  i18n_a11y: High-contrast emergency UI; bilingual instructions.
  acceptance: AC-SF24.2.1 — a gas leak alarm links incident, isolation and work order; restoration without test evidence is rejected.
  dependency: M42.F42.2 SF42.2.2; D-326.
- id: M24.F24.2.SF24.2.2
  name: connection inspection certificate and expiry
  phase: 4
  release: R1
  actors: [chief_engineer, compliance_officer]
  screens: [SCR-ENG-gas-safety]
  inputs: [certificate_type, issuer, issue_date, expiry_date, document]
  states: [valid, expiring, expired]
  api: POST /v1/properties/{pid}/gas-connection-certificates
  events: [GasCertificateExpiring, GasCertificateExpired]
  data: [gas_connection_certificate]
  rules: ["Certificate requirements per jurisdiction from M44/M61", "Expiry alerts at 60/30/7 days; expired certificate raises M61 nonconformance"]
  security: engineering and compliance
  failure_cases: [requirement_unverified]
  finance_report_effect: None direct; inspection cost via maintenance.
  i18n_a11y: Standard.
  acceptance: AC-SF24.2.2 — an expiring certificate alerts at 60 days and creates a nonconformance on expiry.
  dependency: M61.F61.1 SF61.1.4.
```

**M24 key invariants:** as M22; safety isolation/restoration always evidence-backed and linked to incident; software never replaces life-safety systems.
**M24 module acceptance (Section G):** AT-G04.3 (pipeline gas bill imported, approved), AT-G14.3 gas incident chronology (with M42), AT-G08.2 kitchen/laundry allocation drill.

**M24 open decisions**

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-325 | Whether pilot hotel has piped gas; supplier tariff/minimum-charge terms | chief_engineer + financial_controller | Module configurable but disabled until confirmed. |
| D-326 | Gas safety fault procedure owner and competent-person evidence required per jurisdiction | chief_engineer + compliance_officer | Hotel engineering lead owns; licensed contractor certificate required for restoration. |

---

## 9. M25 — Gas cylinders

| Header | Value |
|---|---|
| Purpose | Cylinder types/capacities/identifiers and safety status, full/empty/custody/deposit ledger, par/reorder to requisition/PO, delivery/return exchange and loss, refill invoice and price variance, usage by outlet/event and stocktake, deposit accounting and storage safety. |
| Phases | 4 |
| Release | R1 |
| Bounded context | `utilities` (cylinder sub-ledger) |
| System of record | `cylinder_type`, `cylinder`, `cylinder_location`, `cylinder_custody_entry`, `cylinder_deposit_entry`, `cylinder_stocktake`, `cylinder_storage_rule` |
| Dependencies | M21 (requisition/blanket), M49 (PO), M50 (receiving ledger integration), M20 (invoice match/payable), M28, M13/M16/M57 (outlet/event usage), M61 (safety inspections), M42 |

### F25.1 Cylinder lifecycle

```yaml
- id: M25.F25.1.SF25.1.1
  name: cylinder sizes, identifiers and safety status
  phase: 4
  release: R1
  actors: [storekeeper, chief_engineer]
  screens: [SCR-FIN-cylinder-ledger, SCR-STORE-cylinder-register]
  inputs: [cylinder_type_code, gas_type, capacity_kg, serial_or_barcode, owner, test_due_date, valve_type, safety_status]
  states: [in_service, test_due, quarantined, condemned, returned_to_owner]
  api: POST /v1/properties/{pid}/cylinder-types; POST /v1/properties/{pid}/cylinders
  events: [CylinderRegistered, CylinderSafetyStatusChanged]
  data: [cylinder_type, cylinder]
  rules: ["Serial tracking optional per type; count-only mode tracks quantities per type and location", "Cylinder past test due date or damaged is quarantined and cannot be issued", "Supplier-owned cylinders never capitalized"]
  security: store and engineering roles
  failure_cases: [duplicate_serial, test_date_missing]
  finance_report_effect: Deposit and stock valuation basis.
  i18n_a11y: Barcode scan with manual entry alternative.
  acceptance: AC-SF25.1.1 — a cylinder with expired test date cannot be issued to kitchen; count-only mode works without serials.
  dependency: D-327.
- id: M25.F25.1.SF25.1.2
  name: full/empty/custody/deposit ledger
  phase: 4
  release: R1
  actors: [storekeeper, receiver, shift_chef, bartender, catering_manager]
  screens: [SCR-FIN-cylinder-ledger, SCR-OPS-cylinder-exchange]
  inputs: [movement_type, cylinder_type, cylinder_id, quantity, from_location, to_location, fill_state, custody_holder, deposit_amount]
  states: [full_in_store, issued_in_use, empty_awaiting_return, returned_to_supplier, lost]
  api: POST /v1/properties/{pid}/cylinder-movements (Idempotency-Key)
  events: [CylinderMoved, CylinderDepositRecorded]
  data: [cylinder_custody_entry, cylinder_deposit_entry]
  rules: ["Append-only ledger; balance per type, location and fill state never negative", "Every movement records custody holder and signature/acknowledgment", "Deposit paid or refunded is a separate deposit ledger entry (SF25.2.1)"]
  security: store-scoped; outlet staff record issue/return for their outlet only
  failure_cases: [negative_balance, duplicate_scan, custody_unacknowledged]
  finance_report_effect: Stock value and deposit asset.
  i18n_a11y: Offline-capable mobile exchange screen with large buttons.
  acceptance: AC-SF25.1.2 — issuing a full cylinder to the bar decrements store full count and creates custody with bartender acknowledgment; a duplicate scan does not double move.
  dependency: M50 ledger patterns.
- id: M25.F25.1.SF25.1.3
  name: threshold/requisition/PO
  phase: 4
  release: R1
  actors: [storekeeper, procurement_officer, replenishment_worker]
  screens: [SCR-STORE-reorder-suggestions]
  inputs: [cylinder_type, reorder_point, par_level, blanket_agreement_id]
  states: [above_reorder, below_reorder, requisitioned, ordered]
  api: PUT /v1/properties/{pid}/replenishment-rules/{id}
  events: [ReplenishmentSuggested, RequisitionDrafted]
  data: [replenishment_rule, purchase_requisition, blanket_call_off]
  rules: ["Full count plus open PO below reorder point triggers SF21.1.1", "Exchange orders call off blanket agreement where active", "Storage limit (SF25.2.2) caps order quantity"]
  security: store and procurement roles
  failure_cases: [order_exceeds_storage_limit]
  finance_report_effect: Planned measure via encumbrance.
  i18n_a11y: Standard.
  acceptance: AC-SF25.1.3 — full count 3 with reorder 4 and par 10 suggests 7 unless storage limit is 8 total, in which case suggestion is capped (AT-G04.2).
  dependency: F21.1, SF21.3.2.
- id: M25.F25.1.SF25.1.4
  name: delivery/return exchange and loss
  phase: 4
  release: R1
  actors: [receiver, vendor_user, storekeeper]
  screens: [SCR-OPS-cylinder-exchange, SCR-STORE-receiving]
  inputs: [po_id, full_received, empties_returned, serials, damaged, lost_count, driver_signature]
  states: [expected, exchanged, partially_exchanged, loss_recorded]
  api: POST /v1/properties/{pid}/cylinder-exchanges (Idempotency-Key)
  events: [CylinderExchangeCompleted, CylinderLossRecorded]
  data: [cylinder_custody_entry, goods_receipt, cylinder_deposit_entry]
  rules: ["One exchange records fulls in and empties out atomically", "Empties returned must exist in empty_awaiting_return", "Missing empties create loss with deposit forfeiture or charge", "Received fulls pass quality check (seal, leak test) before full_in_store"]
  security: receiver attestation; vendor confirms counts in vendor app
  failure_cases: [empties_count_mismatch, seal_broken, vendor_disputes_count]
  finance_report_effect: Receipt enables invoice match; loss expense or deposit adjustment.
  i18n_a11y: Counts entered with steppers; confirmation summary read aloud.
  acceptance: AC-SF25.1.4 — delivery of 6 fulls with 5 empties returned leaves 1 empty outstanding with the supplier and a deposit adjustment line (AT-G04.2).
  dependency: M50.F50.2.
- id: M25.F25.1.SF25.1.5
  name: invoice/refill/price variance
  phase: 4
  release: R1
  actors: [ap_clerk, procurement_officer]
  screens: [SCR-FIN-match-workbench]
  inputs: [supplier_invoice_id, refill_qty, unit_price, deposit_lines, delivery_fee]
  states: [matched, price_variance, qty_variance]
  api: POST /v1/properties/{pid}/supplier-invoices/{id}:match
  events: [InvoiceMatched, InvoiceMatchException]
  data: [invoice_match, cylinder_custody_entry]
  rules: ["Invoice refill quantity matched to exchanged fulls; deposits matched to deposit ledger", "Price checked against blanket or PO price"]
  security: AP scope
  failure_cases: [invoice_includes_unreturned_empties_charge, price_above_contract]
  finance_report_effect: Refill cost to inventory; price variance to PPV.
  i18n_a11y: Standard.
  acceptance: AC-SF25.1.5 — an invoice for 6 refills at contract price matches the exchange; a 7th refill line raises a quantity exception.
  dependency: SF20.1.4.
- id: M25.F25.1.SF25.1.6
  name: kitchen/bar/event usage and stocktake
  phase: 4
  release: R1
  actors: [storekeeper, executive_chef, catering_manager, financial_controller]
  screens: [SCR-FIN-cylinder-ledger, SCR-STORE-stocktake]
  inputs: [outlet_id, event_id, issued_count, returned_empty_count, stocktake_counts]
  states: [counted, variance_open, variance_approved]
  api: POST /v1/properties/{pid}/cylinder-stocktakes
  events: [CylinderStocktakeCompleted, CylinderVarianceApproved]
  data: [cylinder_stocktake, cylinder_custody_entry]
  rules: ["Consumption expensed on issue to outlet or event cost center", "Stocktake variance requires approval and reason", "Event usage attributed to event for margin reporting"]
  security: store scope; variance approval by financial_controller
  failure_cases: [unexplained_variance]
  finance_report_effect: Outlet/event gas cost; shrinkage.
  i18n_a11y: Standard.
  acceptance: AC-SF25.1.6 — two cylinders issued to the catering event post cost to the event and the stocktake variance of one missing cylinder requires approval.
  dependency: M16, M12 event costing.
```

### F25.2 Cylinder deposit and storage safety (added — Section D "deposit", Section P safety)

```yaml
- id: M25.F25.2.SF25.2.1
  name: deposit ledger accounting and refund reconciliation
  phase: 4
  release: R1
  actors: [finance_clerk, storekeeper]
  screens: [SCR-FIN-cylinder-ledger]
  inputs: [supplier_id, deposit_amount, cylinders_held, refund_amount]
  states: [deposit_paid, deposit_held, refund_due, refunded, forfeited]
  api: GET /v1/properties/{pid}/cylinder-deposits?supplier_id=
  events: [CylinderDepositRefunded, CylinderDepositForfeited]
  data: [cylinder_deposit_entry, journal_entry]
  rules: ["Deposits recorded as receivable/asset, not expense (subject to D-328)", "Deposit balance per supplier equals held supplier-owned cylinders times deposit rate", "Reconciled to supplier statement quarterly"]
  security: finance scope
  failure_cases: [supplier_statement_mismatch]
  finance_report_effect: Deposit asset account reconciled at close.
  i18n_a11y: Standard.
  acceptance: AC-SF25.2.1 — holding 12 supplier cylinders at deposit 10 shows deposit balance 120 matching GL.
  dependency: D-328.
- id: M25.F25.2.SF25.2.2
  name: storage limit and safety segregation
  phase: 4
  release: R1
  actors: [chief_engineer, compliance_officer, storekeeper]
  screens: [SCR-STORE-cylinder-register]
  inputs: [location_id, max_full, max_total_kg, segregation_rule, jurisdiction_rule_ref]
  states: [within_limit, near_limit, over_limit]
  api: PUT /v1/properties/{pid}/cylinder-storage-rules/{location_id}
  events: [CylinderStorageLimitBreached]
  data: [cylinder_storage_rule, cylinder_location]
  rules: ["Limits per location from verified jurisdiction/civil-defence rule or insurer requirement; unverified rule shows warning", "Movement that would exceed limit is blocked unless safety override with reason", "Breach raises M61 nonconformance"]
  security: override by chief_engineer only
  failure_cases: [rule_unverified, limit_breach]
  finance_report_effect: None direct.
  i18n_a11y: Warning bilingual.
  acceptance: AC-SF25.2.2 — receiving fulls beyond location max is blocked without chief_engineer override.
  dependency: D-329; M61.
```

**M25 key invariants:** no negative cylinder balance per type/location/fill state; an exchange is atomic; unsafe cylinders never issued; deposit ledger reconciles to GL; storage limits enforced.
**M25 module acceptance (Section G):** AT-G04.2 gas-cylinder exchange requisition → PO → delivered full/returned empty/deposit → receipt → invoice match → payment → consumption and cost center; AT-G20.5 short delivery.

**M25 open decisions**

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-327 | Per-serial tracking vs count-only by cylinder type | storekeeper + chief_engineer | Count-only default; serial tracking for large (≥ 40 kg) cylinders. |
| D-328 | Deposit accounting (asset vs expense) | financial_controller | Refundable deposits as current asset. |
| D-329 | Storage limits and segregation per jurisdiction/civil defence and insurer | compliance_officer | Configurable per location; warning status until verified (`unverified-assumption`). |

---
## 10. M26 — Maintenance, assets and vendors

| Header | Value |
|---|---|
| Purpose | Asset registry and criticality, preventive and reactive work with SLA, guest/IoT/meter-triggered faults, room out-of-order and return-to-service, assigned vendor portal (credentials, jobs, site access, labor/parts/photos, quotes, invoices, callbacks, scorecard), spare parts, warranty and asset cost. Maintenance follows Section D: `work order -> quote -> requisition/PO -> material receipt/service completion -> inspection/warranty -> invoice match -> approved payment -> asset and room cost`. |
| Phases | Work orders, room OOO link and vendor assigned-job view Phase 3; procurement, invoice, scorecard, spare parts, capitalization Phase 4. |
| Release | R1 |
| Bounded context | `engineering` |
| System of record | `asset`, `asset_class`, `asset_location`, `pm_schedule`, `work_order`, `work_order_task`, `sla_policy`, `maintenance_block`, `inspection`, `vendor_assignment`, `site_visit`, `work_evidence`, `warranty`, `warranty_claim`, `vendor_scorecard`, `asset_cost_entry` |
| Dependencies | M46 (vendor master, credentials), M03 (OOO/OOS room status and sellable inventory), M06/M56 (housekeeping/guest faults), M22–M24 (meter alerts), M21/M49/M50 (requisition, PO, parts stock), M20 (invoice/payable), M42 (incidents), M61 (inspections), M64 (devices), M66 (capex/depreciation), M55 (guest complaints) |

### F26.1 Maintenance
Story: As chief engineer I see every asset, what is due, what is broken and which rooms are out of order, and I close work with inspected evidence.

```yaml
- id: M26.F26.1.SF26.1.1
  name: asset hierarchy and criticality
  phase: 3
  release: R1
  actors: [chief_engineer, engineer, financial_controller]
  screens: [SCR-ENG-asset-registry]
  inputs: [asset_tag, asset_class, parent_asset_id, location_id, room_id, manufacturer, model, serial, install_date, criticality, owner_type, capitalized_ref]
  states: [planned, in_service, degraded, out_of_service, disposed]
  api: POST /v1/properties/{pid}/assets; PATCH /v1/properties/{pid}/assets/{id}
  events: [AssetRegistered, AssetStatusChanged, AssetDisposed]
  data: [asset, asset_class, asset_location]
  rules: ["Hierarchy site > building > floor > room/area > system > component", "Criticality A/B/C drives SLA and PM frequency", "Room-linked assets can affect room sellability via SF26.1.5", "Disposal requires reason and finance link if capitalized"]
  security: engineering edit; finance view of capitalized link
  failure_cases: [duplicate_tag, orphan_asset, disposal_of_capitalized_without_finance]
  finance_report_effect: Maintenance cost roll-up per asset and room (SF26.3.3).
  i18n_a11y: Bilingual names; QR tag scan with manual fallback.
  acceptance: AC-SF26.1.1 — scanning an asset QR opens its record offline; disposing a capitalized asset requires financial_controller confirmation.
  dependency: D-330.
- id: M26.F26.1.SF26.1.2
  name: preventive schedule
  phase: 3
  release: R1
  actors: [chief_engineer, engineer, pm_worker]
  screens: [SCR-ENG-pm-calendar]
  inputs: [asset_id_or_class, frequency, meter_based_trigger, checklist_template, assigned_team_or_vendor, lead_days]
  states: [scheduled, generated, overdue, completed, skipped_with_reason]
  api: POST /v1/properties/{pid}/pm-schedules; POST /v1/properties/{pid}/pm-schedules:generate
  events: [PreventiveWorkOrderGenerated, PreventiveMaintenanceOverdue]
  data: [pm_schedule, work_order]
  rules: ["Calendar and runtime/meter triggers", "Generation is idempotent per schedule and due date", "Room PM scheduled against low-occupancy dates where possible and blocks via SF26.1.5 only when needed"]
  security: engineering scope
  failure_cases: [duplicate_generation, vendor_contract_expired]
  finance_report_effect: Planned maintenance cost forecast.
  i18n_a11y: Calendar with list alternative.
  acceptance: AC-SF26.1.2 — running generation twice creates one work order per due item; overdue PM escalates to chief_engineer.
  dependency: M46 contract validity.
- id: M26.F26.1.SF26.1.3
  name: guest fault/IoT/meter alert
  phase: 3
  release: R1
  actors: [front_desk_agent, housekeeper, guest, engineer, anomaly_worker]
  screens: [SCR-ENG-work-orders, SCR-FD-service-request, SCR-GST-service-request]
  inputs: [source, room_id, asset_id, description, photo, severity, alert_ref]
  states: [reported, triaged, work_order_open, duplicate_merged]
  api: POST /v1/properties/{pid}/work-orders (Idempotency-Key)
  events: [WorkOrderCreated, WorkOrderMerged]
  data: [work_order, service_request_ref, consumption_anomaly]
  rules: ["Sources guest request (M18/M55), housekeeping (M56), BMS/IoT (M64), meter anomaly (F22.4), incident (M42)", "Duplicate reports for the same asset merge", "Guest sees status only, not internal notes"]
  security: guest-submitted content sanitized; staff scope by department
  failure_cases: [duplicate_report, unknown_asset]
  finance_report_effect: Reactive maintenance cost.
  i18n_a11y: Guest form accessible and bilingual; photo optional.
  acceptance: AC-SF26.1.3 — two guests reporting the same corridor light create one work order with two linked requests.
  dependency: M55.F55.1, M64.F64.1.
- id: M26.F26.1.SF26.1.4
  name: SLA dispatch
  phase: 3
  release: R1
  actors: [chief_engineer, engineer, vendor_user, dispatch_worker]
  screens: [SCR-ENG-dispatch-board, SCR-VEN-assigned-jobs]
  inputs: [work_order_id, priority, sla_policy_id, assignee_type, assignee_id]
  states: [unassigned, assigned, accepted, en_route, in_progress, on_hold_parts, completed, sla_breached]
  api: POST /v1/properties/{pid}/work-orders/{id}:assign; POST /v1/properties/{pid}/work-orders/{id}:status
  events: [WorkOrderAssigned, WorkOrderAccepted, WorkOrderSlaBreached]
  data: [work_order, sla_policy, vendor_assignment]
  rules: ["SLA response and resolution by priority and criticality", "External vendor assignment only to M46-approved vendors with valid credentials for category", "Escalation on breach through M63"]
  security: vendor sees only assigned jobs (SF26.2.2)
  failure_cases: [no_available_technician, vendor_credentials_expired, assignment_declined]
  finance_report_effect: SLA KPI; vendor cost when external.
  i18n_a11y: Dispatch board keyboard operable; mobile push bilingual.
  acceptance: AC-SF26.1.4 — assigning a job to a vendor with expired insurance is blocked; SLA breach escalates at configured time.
  dependency: D-331; M46.F46.1 SF46.1.6.
- id: M26.F26.1.SF26.1.5
  name: room out-of-order
  phase: 3
  release: R1
  actors: [chief_engineer, front_office_manager, housekeeping_supervisor]
  screens: [SCR-ENG-work-order-detail, SCR-FD-room-status]
  inputs: [work_order_id, room_id, block_type, from_date, expected_to_date, reason]
  states: [requested, conflict_check, blocked_ooo, blocked_oos, released]
  api: POST /v1/properties/{pid}/maintenance-blocks
  events: [MaintenanceBlockRequested, RoomBlockedForMaintenance]
  data: [maintenance_block, work_order]
  rules: ["OOO removes room from sellable inventory via M03; OOS keeps it sellable but not assignable", "Conflict with existing reservation assigned to room requires room move before block", "M03 remains system of record for room status"]
  security: engineering requests; front office manager approves when it affects arrivals
  failure_cases: [reservation_conflict, oversell_risk]
  finance_report_effect: OOO excluded from occupancy denominator per KPI definition.
  i18n_a11y: Room status conveyed with text.
  acceptance: AC-SF26.1.5 — requesting OOO for a room with an arriving guest forces a room move or rejection; OOO reduces room-type availability exactly once.
  dependency: M03 OOO/OOS.
- id: M26.F26.1.SF26.1.6
  name: inspection/return to service
  phase: 3
  release: R1
  actors: [chief_engineer, housekeeping_supervisor, engineer]
  screens: [SCR-ENG-inspection]
  inputs: [work_order_id, checklist_results, photos, inspector_id]
  states: [awaiting_inspection, passed, failed_rework, returned_to_service]
  api: POST /v1/properties/{pid}/work-orders/{id}/inspections
  events: [InspectionPassed, InspectionFailed, RoomReturnedToService]
  data: [inspection, maintenance_block]
  rules: ["Inspector distinct from technician for criticality A", "Room returns to service only after maintenance inspection and housekeeping clean", "Failed inspection reopens work order"]
  security: role-based inspection rights
  failure_cases: [inspector_equals_technician, housekeeping_not_done]
  finance_report_effect: Service acceptance evidence for vendor invoice (SF21.2.2).
  i18n_a11y: Checklist accessible offline.
  acceptance: AC-SF26.1.6 — room stays OOO until inspection passes and housekeeping marks clean; failed inspection reopens the job.
  dependency: M56.F56.1 SF56.1.4.
```

### F26.2 Vendor portal
Story: As a maintenance contractor I see only my assigned jobs, record arrival, evidence and invoices, and know when I will be paid.

```yaml
- id: M26.F26.2.SF26.2.1
  name: credentials/insurance/warranty
  phase: 3
  release: R1
  actors: [vendor_admin, compliance_officer, chief_engineer]
  screens: [SCR-VEN-credentials, SCR-ENG-vendor-profile]
  inputs: [license_type, license_number, insurance_policy, coverage_amount, expiry, trade_certificates, warranty_terms]
  states: [submitted, verified, expiring, expired, rejected]
  api: GET /v1/properties/{pid}/vendors/{id}/credentials (M46 master); POST /v1/vendor/credentials
  events: [VendorCredentialExpiring, VendorCredentialExpired]
  data: [vendor_credential_ref, warranty]
  rules: ["Credentials are mastered in M46 SF46.1.3/SF46.1.6; M26 reads eligibility per maintenance category", "Warranty terms offered by vendor stored for job-level warranty (SF26.3.2)"]
  security: vendor manages own; verification by compliance
  failure_cases: [expired_credential_on_open_job]
  finance_report_effect: None direct.
  i18n_a11y: Vendor app bilingual.
  acceptance: AC-SF26.2.1 — expiry of insurance mid-job alerts chief_engineer and blocks new assignments.
  dependency: M46.F46.1.
- id: M26.F26.2.SF26.2.2
  name: assigned-job view only
  phase: 3
  release: R1
  actors: [vendor_user, vendor_admin]
  screens: [SCR-VEN-assigned-jobs]
  inputs: [vendor_id, user_id]
  states: [visible, revoked]
  api: GET /v1/vendor/jobs
  events: [VendorJobAccessRevoked]
  data: [vendor_assignment, work_order]
  rules: ["Vendor sees only work orders assigned to its organization and only fields needed (location, scope, access window)", "Guest names hidden unless strictly needed and approved", "Access revoked at job close plus configured period"]
  security: object-level authorization tested against OWASP API BOLA; vendor tenant isolation
  failure_cases: [id_enumeration_attempt, stale_access]
  finance_report_effect: None.
  i18n_a11y: Vendor app WCAG 2.2 AA.
  acceptance: AC-SF26.2.2 — vendor A requesting vendor B's job id receives 404; closed jobs disappear after the configured period (AT-G10.5).
  dependency: M02; M46.F46.3 SF46.3.6.
- id: M26.F26.2.SF26.2.3
  name: site access/arrival
  phase: 3
  release: R1
  actors: [vendor_user, security_officer, engineer]
  screens: [SCR-VEN-job-check-in, SCR-ENG-site-visits]
  inputs: [work_order_id, arrival_time, personnel_names, id_check_ref, permit_to_work, briefing_ack]
  states: [scheduled, checked_in, briefed, working, checked_out, no_show]
  api: POST /v1/properties/{pid}/site-visits; POST /v1/properties/{pid}/site-visits/{id}:check-out
  events: [VendorCheckedIn, VendorCheckedOut, VendorNoShow]
  data: [site_visit]
  rules: ["Check-in validates scheduled window and permit-to-work for hot work, electrical or gas", "Contractor site briefing from M62 SF62.1.4 required before work", "Time on site feeds labor verification"]
  security: security officer confirms physical ID; minimal personal data retained
  failure_cases: [arrival_outside_window, permit_missing, no_show]
  finance_report_effect: Labor hours evidence for invoice match.
  i18n_a11y: QR check-in with manual alternative.
  acceptance: AC-SF26.2.3 — electrical work without a permit-to-work cannot move to working; a no-show updates the scorecard.
  dependency: M62.F62.1 SF62.1.4.
- id: M26.F26.2.SF26.2.4
  name: labor/parts/photos
  phase: 3
  release: R1
  actors: [vendor_user, engineer]
  screens: [SCR-VEN-job-evidence]
  inputs: [work_order_id, labor_hours, technicians, parts_used, before_photos, after_photos, notes]
  states: [draft, submitted, accepted, rejected]
  api: POST /v1/vendor/jobs/{id}/evidence (Idempotency-Key)
  events: [WorkEvidenceSubmitted]
  data: [work_evidence]
  rules: ["Photos timestamped and stored immutable; EXIF location stripped from guest-area photos except property zone tag", "Parts from hotel stock issued via M50; vendor-supplied parts listed for invoice", "Submission leads to SF21.2.2 acceptance"]
  security: vendor own jobs; malware scan; privacy review of guest-room photos
  failure_cases: [photo_missing, hours_exceed_site_visit]
  finance_report_effect: Basis for invoice validation.
  i18n_a11y: Photo upload with captions.
  acceptance: AC-SF26.2.4 — claimed hours exceeding site-visit duration are flagged at acceptance.
  dependency: SF21.2.2.
- id: M26.F26.2.SF26.2.5
  name: quote and invoice
  phase: 4
  release: R1
  actors: [vendor_user, chief_engineer, procurement_officer, ap_clerk]
  screens: [SCR-VEN-quotes, SCR-VEN-invoice-submit]
  inputs: [work_order_id, rfq_id, quote_lines, validity, invoice_file, po_id]
  states: [quote_requested, quoted, awarded, invoiced]
  api: POST /v1/vendor/quotes (via M49 RFQ); POST /v1/vendor/invoices
  events: [VendorQuoteSubmitted, SupplierInvoiceReceived]
  data: [vendor_quote_ref, supplier_invoice]
  rules: ["Quotes above threshold go through M49 RFQ; below threshold direct quote per policy", "Invoice must reference PO and accepted job; captured by SF20.1.2"]
  security: vendor own jobs
  failure_cases: [invoice_without_po, quote_expired]
  finance_report_effect: Invoiced measure.
  i18n_a11y: Standard.
  acceptance: AC-SF26.2.5 — a vendor invoice without PO reference is rejected at submission with guidance.
  dependency: M49; SF20.1.2.
- id: M26.F26.2.SF26.2.6
  name: callbacks/disputes
  phase: 4
  release: R1
  actors: [chief_engineer, vendor_user]
  screens: [SCR-ENG-work-order-detail, SCR-VEN-disputes]
  inputs: [original_work_order_id, callback_reason, within_warranty, dispute_reason]
  states: [callback_open, resolved_no_charge, charged, disputed]
  api: POST /v1/properties/{pid}/work-orders/{id}:callback
  events: [VendorCallbackRaised, VendorCallbackResolved]
  data: [work_order, warranty_claim, procurement_dispute]
  rules: ["Repeat failure within warranty creates no-charge callback", "Callback counts in scorecard", "Disputes use SF21.3.5"]
  security: vendor own jobs
  failure_cases: [warranty_period_disputed]
  finance_report_effect: Avoided cost tracked.
  i18n_a11y: Standard.
  acceptance: AC-SF26.2.6 — the same AC unit failing 10 days after repair under 30-day warranty creates a no-charge callback.
  dependency: SF26.3.2.
- id: M26.F26.2.SF26.2.7
  name: supplier scorecard and payment visibility
  phase: 4
  release: R1
  actors: [vendor_admin, chief_engineer, procurement_officer]
  screens: [SCR-VEN-performance, SCR-ENG-vendor-profile]
  inputs: [vendor_id, period]
  states: [computed, disputed_by_vendor, final]
  api: GET /v1/properties/{pid}/vendors/{id}/scorecard; GET /v1/vendor/payments
  events: [VendorScorecardPublished]
  data: [vendor_scorecard, payable]
  rules: ["Metrics response time, SLA met, first-time fix, callbacks, no-shows, invoice accuracy with sample size", "Vendor may contest a metric through fair-review process (M46 SF46.2.6)", "Vendor sees own payable status (approved, paid date, reference) not internal approvals"]
  security: vendor own data only
  failure_cases: [small_sample_misleading]
  finance_report_effect: Feeds M49 previous-history weighting.
  i18n_a11y: Scorecard numbers with text descriptions.
  acceptance: AC-SF26.2.7 — scorecard shows sample size; vendor sees paid status with reference once payment confirmed.
  dependency: D-332; M49.F49.2 SF49.2.2.
```

### F26.3 Spares, warranty and asset cost (added — Section C "spare parts and requisitions", Section D "asset and room cost")

```yaml
- id: M26.F26.3.SF26.3.1
  name: spare parts and work-order material requisition
  phase: 4
  release: R1
  actors: [engineer, storekeeper, chief_engineer]
  screens: [SCR-ENG-work-order-detail, SCR-STORE-issue]
  inputs: [work_order_id, item_id, quantity, store_id]
  states: [requested, reserved, issued, returned, requisitioned]
  api: POST /v1/properties/{pid}/work-orders/{id}/materials
  events: [WorkOrderMaterialIssued, WorkOrderMaterialRequisitioned]
  data: [work_order, stock_ledger_ref, purchase_requisition]
  rules: ["Parts issued from store via M50 ledger to the work order", "Out-of-stock parts create requisition (SF21.1.1) linked to work order", "Unused parts returned intact to store"]
  security: engineering and store roles
  failure_cases: [part_out_of_stock, issue_without_work_order]
  finance_report_effect: Parts cost charged to work order, asset and cost center.
  i18n_a11y: Scan-to-issue with manual option.
  acceptance: AC-SF26.3.1 — issuing two filters to a work order reduces stock and adds cost to the asset; unavailable part creates linked requisition.
  dependency: M50.F50.3 SF50.3.2.
- id: M26.F26.3.SF26.3.2
  name: warranty and warranty claim
  phase: 4
  release: R1
  actors: [chief_engineer, procurement_officer]
  screens: [SCR-ENG-warranty]
  inputs: [asset_id_or_work_order_id, warranty_provider, start, end, terms, claim_reason]
  states: [active, expiring, expired, claim_open, claim_accepted, claim_rejected]
  api: POST /v1/properties/{pid}/warranties; POST /v1/properties/{pid}/warranty-claims
  events: [WarrantyClaimOpened, WarrantyClaimResolved]
  data: [warranty, warranty_claim]
  rules: ["New work order on an asset under warranty prompts claim before paid repair", "Claim outcome recorded with evidence"]
  security: engineering scope
  failure_cases: [warranty_ignored_paid_repair]
  finance_report_effect: Avoided cost and recoveries.
  i18n_a11y: Standard.
  acceptance: AC-SF26.3.2 — creating a paid work order for an asset under warranty shows a claim prompt and requires override reason to proceed.
  dependency: SF26.2.6.
- id: M26.F26.3.SF26.3.3
  name: asset and room maintenance cost roll-up
  phase: 4
  release: R1
  actors: [chief_engineer, financial_controller, gm]
  screens: [SCR-ENG-asset-cost, SCR-GM-home]
  inputs: [asset_id, room_id, period]
  states: [computed]
  api: GET /v1/properties/{pid}/assets/{id}/cost?period=
  events: [AssetCostEntryRecorded]
  data: [asset_cost_entry, journal_line]
  rules: ["Labor (internal hours valued at department average rate, not individual salary), parts and vendor invoices roll to asset and room", "Totals reconcile to maintenance GL accounts"]
  security: individual salaries never exposed; averages only
  failure_cases: [unlinked_invoice]
  finance_report_effect: M32 SF32.2.5 maintenance vendor/parts and room cost.
  i18n_a11y: Table with totals.
  acceptance: AC-SF26.3.3 — room 204 annual cost shows vendor, parts and internal labor lines reconciling to GL.
  dependency: F27.4 department averages.
- id: M26.F26.3.SF26.3.4
  name: capitalization and depreciation hand-off
  phase: 4
  release: R1
  actors: [financial_controller, chief_engineer]
  screens: [SCR-FIN-capitalization]
  inputs: [asset_id, capitalization_decision_id, cost, useful_life]
  states: [capex_pending, capitalized, handed_to_m66]
  api: POST /v1/properties/{pid}/assets/{id}:capitalize
  events: [AssetCapitalized]
  data: [asset, capitalization_decision]
  rules: ["Capitalized assets get finance link and depreciation schedule owned by M66", "Major repair capitalization follows D-318"]
  security: financial_controller
  failure_cases: [asset_without_cost]
  finance_report_effect: Balance-sheet asset; depreciation via M66.
  i18n_a11y: Standard.
  acceptance: AC-SF26.3.4 — a capitalized chiller has cost, useful life and M66 depreciation schedule reference.
  dependency: SF21.2.4; M66.F66.2 SF66.2.3.
```

**M26 key invariants:** vendor sees only assigned jobs; no assignment to vendor with expired credentials; room returns to service only after inspection and clean; M03 remains the room-status SoR; maintenance cost rolls to asset/room without individual salary exposure.

**M26 module acceptance (Section G):** AT-G04.2 maintenance materials requisition and service completion enter procurement/AP; AT-G10.5 vendor restricted to own jobs; AT-G10.4 service accepted and invoice reconciled; AT-G08.2 maintenance cost drills to work order and invoice.

**M26 open decisions**

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-330 | Asset register import scope and tagging standard (QR vs barcode) | chief_engineer | Import critical A/B assets at go-live; QR tags. |
| D-331 | SLA matrix by priority/criticality | chief_engineer + gm | P1 respond 15 min/resolve 4 h; P2 1 h/24 h; P3 next day/7 days. |
| D-332 | Vendor scorecard metrics and weights (aligned with M49 history weight) | procurement_approver | Equal weights across SLA, first-time fix, callbacks, invoice accuracy; minimum 5 jobs for display. |

---

## 11. M27 — HR, workforce and payroll

| Header | Value |
|---|---|
| Purpose | Employee master and contracts, compensation with effective dates, protected bank/identity data, joining/leaving and final settlement; rosters, time capture, leave, overtime and labor attribution; payroll gross-to-net with configurable statutory rules, maker-checker, bank/WPS files, rejection handling, payslips and salary cost journal; strict salary confidentiality. Salaries follow Section D: `approved attendance/leave/overtime -> gross-to-net payroll -> confidential manager/finance approvals -> WPS/bank file/payment status -> salary expense/accrual and payslips`. |
| Phases | Employee/roster basis Phase 2 (M47 kitchen roster); time/leave Phase 3; payroll, WPS, labor allocation Phase 4; Canadian payroll via M38 Phase 4–6; validation Phase 6. |
| Release | R1 |
| Bounded context | `workforce` (roster/time/leave) and `payroll` (separate schema, separate KMS key, restricted service role) |
| System of record | `employee`, `employment_contract`, `position`, `compensation_record`, `employee_bank_account`, `employee_identity_document`, `final_settlement`, `roster`, `shift`, `shift_swap`, `time_punch`, `timesheet`, `leave_type`, `leave_request`, `leave_balance`, `overtime_approval`, `labor_allocation`, `payroll_calendar`, `payroll_run`, `payroll_result`, `pay_element`, `salary_advance`, `statutory_rule_binding`, `payslip`, `wps_file`, `wps_submission`, `salary_access_log`, `payroll_payment_batch` |
| Dependencies | M01/M02 (identity, roles, step-up), M38.F38.2 (Canadian SIN/CPP/EI/QPP/QPIP/T4), M44 (payroll rule packs and status), M28.F28.2 (salary disbursement batch), M20 (payable/cash forecast totals), M19 (salary journal), M47 (chef rosters), M62 (training/labor forecast), M12/M16 (event labor attribution), M32 (labor cost reports) |

### F27.1 Employee master
Story: As HR officer I keep one accurate, protected record per employee with effective-dated contracts and pay.

```yaml
- id: M27.F27.1.SF27.1.1
  name: contract/role/work location
  phase: 3
  release: R1
  actors: [hr_officer, gm, employee]
  screens: [SCR-HR-employee, SCR-EMP-profile]
  inputs: [employee_number, legal_name, preferred_name, legal_entity_id, position_id, department_id, cost_center, work_location, contract_type, start_date, probation_end, working_hours_pattern, jurisdiction_ref]
  states: [candidate, active, on_leave, suspended, terminated]
  api: POST /v1/properties/{pid}/employees; POST /v1/properties/{pid}/employees/{id}/contracts
  events: [EmployeeHired, EmploymentContractChanged, EmployeeStatusChanged]
  data: [employee, employment_contract, position]
  rules: ["One active contract per employee per legal entity at a time; effective-dated changes", "Work jurisdiction selects payroll rule binding (SF27.5.5/SF27.5.6)", "Contract document stored encrypted with e-signature evidence where used (M41)"]
  security: HR scope; managers see name, role, roster only; employee sees own
  failure_cases: [overlapping_contracts, jurisdiction_rule_unverified]
  finance_report_effect: Headcount by department for labor KPIs.
  i18n_a11y: Bilingual names (Arabic and Latin), RTL form.
  acceptance: AC-SF27.1.1 — a promotion effective next month keeps current role until that date; a manager cannot view the contract document.
  dependency: M44 jurisdiction; M41 signature.
- id: M27.F27.1.SF27.1.2
  name: compensation effective dates
  phase: 4
  release: R1
  actors: [hr_officer, payroll_officer, payroll_approver]
  screens: [SCR-HR-compensation]
  inputs: [employee_id, pay_basis, base_amount, currency, allowances, effective_from, reason]
  states: [proposed, approved, active, superseded]
  api: POST /v1/properties/{pid}/payroll/compensation-records
  events: [CompensationChangeApproved]
  data: [compensation_record]
  rules: ["Changes require maker-checker", "Retroactive changes generate retro pay in next run with explicit lines", "Stored in payroll schema only"]
  security: payroll schema encryption; access logged in salary_access_log
  failure_cases: [retro_across_closed_year, self_approval]
  finance_report_effect: Payroll cost forecast totals by department.
  i18n_a11y: Currency formatting per locale.
  acceptance: AC-SF27.1.2 — a raise backdated one month produces a retro line in the next run and the access is logged.
  dependency: F27.4.
- id: M27.F27.1.SF27.1.3
  name: bank and identity protection
  phase: 4
  release: R1
  actors: [hr_officer, payroll_officer, employee, dpo]
  screens: [SCR-HR-employee-sensitive, SCR-EMP-bank-details]
  inputs: [bank_name, iban_or_local_account, account_holder, bank_proof, national_id_type, national_id_number, expiry]
  states: [pending_verification, verified, change_pending]
  api: POST /v1/properties/{pid}/employees/{id}/bank-accounts; POST /v1/properties/{pid}/employees/{id}/identity-documents
  events: [EmployeeBankChangeRequested, EmployeeBankVerified]
  data: [employee_bank_account, employee_identity_document]
  rules: ["Identifiers (civil ID, SIN via M38 SF38.2.1, others) field-encrypted and masked", "Bank change by employee requires verification before next run cutoff; changes within cutoff apply next run", "Minimum identity data per jurisdiction"]
  security: field-level encryption; step-up MFA to unmask; purpose logged
  failure_cases: [bank_change_fraud, id_expired]
  finance_report_effect: None direct; prevents misdirected pay.
  i18n_a11y: Masked values announced as masked.
  acceptance: AC-SF27.1.3 — an employee bank change after cutoff is paid to the old account this run with notice; unmasking a civil ID writes an access log entry.
  dependency: M38.F38.2 SF38.2.1; M02.
- id: M27.F27.1.SF27.1.4
  name: joining/leaving and final settlement
  phase: 4
  release: R1
  actors: [hr_officer, payroll_officer, payroll_approver, gm]
  screens: [SCR-HR-offboarding]
  inputs: [employee_id, last_working_day, reason, leave_encashment, end_of_service_basis, recoveries, advances_outstanding]
  states: [initiated, calculated, approved, paid, closed]
  api: POST /v1/properties/{pid}/employees/{id}/final-settlements
  events: [FinalSettlementCalculated, FinalSettlementApproved, EmployeeOffboarded]
  data: [final_settlement, payroll_result, salary_advance]
  rules: ["Final settlement uses jurisdiction rule binding (e.g. Oman end-of-service rules pending D-338)", "Outstanding advances recovered within legal limits", "Access revoked across M02 at last working day", "Unverified rule pack requires manual calculation evidence and approval"]
  security: HR and payroll only
  failure_cases: [rule_unverified, negative_net]
  finance_report_effect: End-of-service expense/provision release.
  i18n_a11y: Settlement statement bilingual.
  acceptance: AC-SF27.1.4 — offboarding revokes system access on last day and produces a settlement with advance recovery; unverified rule forces manual evidence.
  dependency: D-338.
- id: M27.F27.1.SF27.1.5
  name: confidential access audit
  phase: 4
  release: R1
  actors: [dpo, auditor, hr_officer]
  screens: [SCR-HR-access-log]
  inputs: [employee_id, accessor_id, period]
  states: [logged, reviewed, flagged]
  api: GET /v1/properties/{pid}/payroll/access-log
  events: [SalaryRecordAccessed, SalaryAccessAnomalyFlagged]
  data: [salary_access_log]
  rules: ["Every read of individual pay, bank or identity data is logged with purpose", "Unusual access patterns flagged for review", "Log is append-only and retained per D-340"]
  security: log readable by dpo and auditor only
  failure_cases: [bulk_export_attempt, access_outside_role]
  finance_report_effect: None.
  i18n_a11y: Log table accessible.
  acceptance: AC-SF27.1.5 — viewing ten employees' pay within one minute by a payroll officer is flagged for review.
  dependency: F27.4.
```

### F27.2 Time and scheduling
Story: As department head I publish rosters, staff clock in, and only approved time and leave reach payroll.

```yaml
- id: M27.F27.2.SF27.2.1
  name: rosters and shift swap
  phase: 3
  release: R1
  actors: [department_heads, employee, executive_chef, housekeeping_supervisor]
  screens: [SCR-HR-roster, SCR-EMP-schedule]
  inputs: [department_id, week, shifts, required_skills, employee_ids, swap_request]
  states: [draft, published, swap_requested, swap_approved, locked]
  api: POST /v1/properties/{pid}/rosters; POST /v1/properties/{pid}/shift-swaps
  events: [RosterPublished, ShiftSwapApproved]
  data: [roster, shift, shift_swap]
  rules: ["Swaps require qualification match (e.g. food-safety credential for kitchen via M62) and manager approval", "Rest-period and maximum-hours rules from jurisdiction binding warn or block", "Chef coverage rules from M47 consume this roster"]
  security: department-scoped
  failure_cases: [rest_period_violation, unqualified_swap]
  finance_report_effect: Labor forecast vs budget (M62 SF62.2.2).
  i18n_a11y: Schedule readable on mobile and by screen reader.
  acceptance: AC-SF27.2.1 — a swap to an employee lacking food-safety certificate is blocked for a kitchen shift.
  dependency: M47.F47.1; M62.F62.1.
- id: M27.F27.2.SF27.2.2
  name: punch/time capture
  phase: 3
  release: R1
  actors: [employee, department_heads, time_worker]
  screens: [SCR-EMP-time, SCR-HR-timesheets]
  inputs: [employee_id, punch_type, timestamp, device_id, location_method, offline_flag]
  states: [recorded, exception, corrected, approved]
  api: POST /v1/properties/{pid}/time-punches (Idempotency-Key)
  events: [TimePunchRecorded, TimeExceptionRaised]
  data: [time_punch, timesheet]
  rules: ["Methods registered kiosk PIN, staff app with property geofence, or supervisor entry; biometric capture only with verified lawful basis and alternative", "Offline punches sync with device time and server reconciliation", "Corrections by manager with reason; original retained"]
  security: device identity; location used only at punch time
  failure_cases: [missing_punch, duplicate_punch, clock_skew, biometric_not_permitted]
  finance_report_effect: Hours basis for payroll and labor cost.
  i18n_a11y: One-tap punch with large targets; audio/haptic confirmation.
  acceptance: AC-SF27.2.2 — an offline punch syncs once; a missing out-punch raises an exception to the manager before payroll cutoff.
  dependency: D-335.
- id: M27.F27.2.SF27.2.3
  name: leave/absence
  phase: 3
  release: R1
  actors: [employee, department_heads, hr_officer]
  screens: [SCR-EMP-leave, SCR-HR-leave]
  inputs: [leave_type, from, to, partial_day, attachment, reason]
  states: [requested, approved, rejected, cancelled, taken]
  api: POST /v1/properties/{pid}/leave-requests; POST /v1/properties/{pid}/leave-requests/{id}:decide
  events: [LeaveRequested, LeaveApproved, AbsenceRecorded]
  data: [leave_request, leave_balance, leave_type]
  rules: ["Leave types and accrual rules from jurisdiction binding plus contract", "Sick-note attachments visible to HR only", "Approved leave updates roster and chef coverage alerts (M47)"]
  security: medical details HR-only
  failure_cases: [insufficient_balance, overlapping_request]
  finance_report_effect: Leave liability accrual.
  i18n_a11y: Calendar picker accessible.
  acceptance: AC-SF27.2.3 — approving the head chef's leave triggers an M47 coverage check.
  dependency: M47.F47.1 SF47.1.2.
- id: M27.F27.2.SF27.2.4
  name: approved overtime/holiday
  phase: 4
  release: R1
  actors: [department_heads, gm, payroll_officer]
  screens: [SCR-HR-timesheets]
  inputs: [employee_id, date, hours, overtime_type, public_holiday_flag, pre_approval_ref]
  states: [requested, approved, rejected, paid]
  api: POST /v1/properties/{pid}/overtime-approvals
  events: [OvertimeApproved]
  data: [overtime_approval, timesheet]
  rules: ["Only approved overtime flows to payroll", "Multipliers from jurisdiction binding and contract", "Public holiday calendar per jurisdiction"]
  security: department-scoped approvals
  failure_cases: [unapproved_overtime_at_cutoff]
  finance_report_effect: Overtime cost KPI by department.
  i18n_a11y: Standard.
  acceptance: AC-SF27.2.4 — unapproved overtime hours at cutoff are excluded from the run and listed as exceptions (AT-G05.1).
  dependency: SF27.3.3.
- id: M27.F27.2.SF27.2.5
  name: department split and project/event labor attribution
  phase: 4
  release: R1
  actors: [department_heads, catering_manager, payroll_officer]
  screens: [SCR-HR-labor-allocation]
  inputs: [timesheet_id, cost_center_splits, event_id, work_order_id]
  states: [unallocated, allocated, locked]
  api: POST /v1/properties/{pid}/labor-allocations
  events: [LaborAllocated]
  data: [labor_allocation]
  rules: ["Hours split across cost centers and events; cost valued at run time in payroll", "Event labor reported as totals per event, never per person outside HR"]
  security: managers see hours; cost visible aggregated only
  failure_cases: [split_not_100]
  finance_report_effect: Departmental and event labor cost in P&L.
  i18n_a11y: Standard.
  acceptance: AC-SF27.2.5 — 8 hours split 6 banquet and 2 bar produce department cost totals in the salary journal without individual lines.
  dependency: SF27.3.8.
```

### F27.3 Payroll
Story: As payroll officer I run gross-to-net for the period, resolve exceptions, get independent approval, pay through the approved bank/WPS route and post one aggregated salary journal.

```yaml
- id: M27.F27.3.SF27.3.1
  name: base salary
  phase: 4
  release: R1
  actors: [payroll_officer, payroll_worker]
  screens: [SCR-HR-payroll-run]
  inputs: [payroll_calendar_id, period, compensation_records, days_worked, proration_method]
  states: [pending, calculated]
  api: POST /v1/properties/{pid}/payroll/runs (Idempotency-Key)
  events: [PayrollRunCreated]
  data: [payroll_run, payroll_result, payroll_calendar]
  rules: ["Base pay prorated for joiners/leavers and unpaid leave per contract method", "One regular run per calendar period per legal entity; off-cycle runs explicitly typed"]
  security: payroll roles only
  failure_cases: [duplicate_run, missing_compensation]
  finance_report_effect: Gross pay basis.
  i18n_a11y: Standard.
  acceptance: AC-SF27.3.1 — a mid-month joiner is prorated by calendar days per configuration; a second regular run for the period is rejected.
  dependency: D-336.
- id: M27.F27.3.SF27.3.2
  name: allowance/deduction and advances
  phase: 4
  release: R1
  actors: [payroll_officer, hr_officer, employee]
  screens: [SCR-HR-pay-elements, SCR-EMP-advance-request]
  inputs: [pay_element_code, amount_or_formula, recurring_flag, advance_amount, recovery_schedule]
  states: [active, ended, advance_requested, advance_approved, advance_recovering, advance_closed]
  api: POST /v1/properties/{pid}/payroll/pay-elements; POST /v1/properties/{pid}/payroll/salary-advances
  events: [SalaryAdvanceApproved, PayElementAssigned]
  data: [pay_element, salary_advance]
  rules: ["Housing, transport, meal, service-charge distribution and other allowances configurable per contract", "Deductions capped per jurisdiction limits where configured", "Advances disbursed via M28 payment order and recovered in installments"]
  security: payroll roles; employee sees own advances
  failure_cases: [deduction_exceeds_limit, advance_to_leaver]
  finance_report_effect: Advance receivable; allowance expense by department.
  i18n_a11y: Employee advance form accessible.
  acceptance: AC-SF27.3.2 — an advance of 300 with 3 installments reduces net pay by 100 for three runs and closes.
  dependency: M28.F28.2.
- id: M27.F27.3.SF27.3.3
  name: configurable statutory rules with local validation
  phase: 4
  release: R1
  actors: [payroll_officer, compliance_officer, payroll_worker]
  screens: [SCR-HR-statutory-rules]
  inputs: [jurisdiction_rule_pack_id, rule_version, parameters, validation_status]
  states: [draft, source_cited, counsel_reviewed, verified, expired, blocked]
  api: GET /v1/properties/{pid}/payroll/statutory-bindings
  events: [StatutoryRuleBindingChanged, StatutoryRuleBlocked]
  data: [statutory_rule_binding]
  rules: ["Payroll consumes rule packs from M44; status must be verified or counsel-reviewed to run automated statutory calculation", "Oman social insurance contributions for eligible employees and other Oman rules come from the Oman pack (status unverified until validated)", "Canadian federal/provincial tax, CPP/QPP, EI/QPIP are calculated by M38.F38.2 and returned as results", "Unverified pack forces manual statutory line entry with evidence and approver"]
  security: rule-pack changes by compliance with maker-checker
  failure_cases: [rule_pack_unverified, rule_pack_expired, rate_year_missing]
  finance_report_effect: Employer contribution expense and statutory liabilities.
  i18n_a11y: Status labels in text.
  acceptance: AC-SF27.3.3 — a run for an employee under an unverified pack cannot compute statutory lines automatically and requires manual evidence; a Canadian employee's deductions appear from M38 with parameter year stamped (AT-G12.2).
  dependency: M44.F44.2; M38.F38.2 SF38.2.2, SF38.2.3.
- id: M27.F27.3.SF27.3.4
  name: gross-to-net preview/exception
  phase: 4
  release: R1
  actors: [payroll_officer, payroll_worker]
  screens: [SCR-HR-payroll-run]
  inputs: [payroll_run_id]
  states: [calculated, exceptions_open, exceptions_resolved, ready_for_approval]
  api: POST /v1/properties/{pid}/payroll/runs/{id}:calculate; GET /v1/properties/{pid}/payroll/runs/{id}/exceptions
  events: [PayrollRunCalculated, PayrollExceptionRaised]
  data: [payroll_run, payroll_result]
  rules: ["Exceptions include negative net, net change above threshold vs prior run, missing bank, unapproved time, unverified rule pack, missing ID for WPS", "Preview shows totals by department and variance to prior period", "Recalculation is deterministic and versioned"]
  security: payroll roles
  failure_cases: [negative_net, missing_bank_account, variance_spike]
  finance_report_effect: Accrual basis at month end.
  i18n_a11y: Exception list accessible.
  acceptance: AC-SF27.3.4 — an employee net change above 30 percent raises an exception that must be resolved before approval.
  dependency: SF27.3.1–SF27.3.3.
- id: M27.F27.3.SF27.3.5
  name: maker-checker authorization
  phase: 4
  release: R1
  actors: [payroll_officer, payroll_approver, financial_controller, gm]
  screens: [SCR-HR-payroll-approval, SCR-GM-approvals]
  inputs: [payroll_run_id, approval_decision, mfa_assertion]
  states: [ready_for_approval, hr_approved, finance_approved, rejected]
  api: POST /v1/properties/{pid}/payroll/runs/{id}:approve
  events: [PayrollRunApproved, PayrollRunRejected]
  data: [payroll_run, approval_step]
  rules: ["Payroll approver distinct from preparer", "Finance approval sees department totals, headcount and variance, not individual pay", "Any change after approval invalidates approvals", "Approval does not move money; payment release is SF27.3.6 via M28"]
  security: step-up MFA; GM approval view aggregated
  failure_cases: [self_approval, change_after_approval]
  finance_report_effect: Payroll payable created for net pay and statutory liabilities.
  i18n_a11y: Approval summary accessible.
  acceptance: AC-SF27.3.5 — the GM approval screen shows department totals only; preparer self-approval returns SOD_VIOLATION (AT-G05.2).
  dependency: F27.4.
- id: M27.F27.3.SF27.3.6
  name: WPS/bank file
  phase: 4
  release: R1
  actors: [payroll_officer, payment_releaser, payroll_worker]
  screens: [SCR-HR-wps-export, SCR-FIN-payout-approval]
  inputs: [payroll_run_id, channel, bank_format_version, wps_format_version, value_date]
  states: [generated, validated, released, submitted, accepted_by_bank, partially_rejected, rejected, confirmed]
  api: POST /v1/properties/{pid}/payroll/runs/{id}/payment-files (Idempotency-Key); POST /v1/properties/{pid}/payroll/payment-batches/{id}:release
  events: [PayrollPaymentFileGenerated, PayrollPaymentReleased, PayrollPaymentConfirmed]
  data: [wps_file, payroll_payment_batch, payment_order]
  rules: ["Oman WPS salary information file generated to the bank-specified format version; status unverified until bank and Ministry validation (D-333)", "Non-Oman bank bulk file per bank format", "File hash recorded; regeneration of a released file blocked", "Release via M28.F28.2 by payment_releaser distinct from payroll approver", "Finance sees batch total only"]
  security: file encrypted at rest and in transit; download access logged; delivered by bank channel or secure manual upload
  failure_cases: [format_rejected, value_date_holiday, file_regeneration_after_release]
  finance_report_effect: Moves payroll payable to paid on bank confirmation.
  i18n_a11y: Arabic names transliteration as bank requires.
  acceptance: AC-SF27.3.6 — WPS file validates against the stored format fixture, is released by a second person and becomes confirmed only on bank acknowledgment (AT-G05.3).
  dependency: D-333; SF27.5.1–SF27.5.3; M28.F28.2.
- id: M27.F27.3.SF27.3.7
  name: bank rejection/resubmission
  phase: 4
  release: R1
  actors: [payroll_officer, payment_releaser, hr_officer]
  screens: [SCR-HR-wps-export, SCR-OPS-exception-queue]
  inputs: [rejection_file, rejected_employee_refs, reason_codes, corrected_bank_details]
  states: [rejected_lines_open, corrected, resubmitted, confirmed]
  api: POST /v1/properties/{pid}/payroll/payment-batches/{id}/rejections; POST /v1/properties/{pid}/payroll/payment-batches/{id}:resubmit-rejected (Idempotency-Key)
  events: [PayrollPaymentRejected, PayrollPaymentResubmitted]
  data: [payroll_payment_batch, wps_submission, payment_order]
  rules: ["Only rejected lines are resubmitted; accepted lines never repaid", "Resubmission requires corrected, verified bank details and a new release", "Rejection with unknown status triggers inquiry before resubmission"]
  security: same as SF27.3.6
  failure_cases: [status_unknown, repeated_rejection]
  finance_report_effect: Rejected amounts remain payable until confirmed.
  i18n_a11y: Employee notified in preferred language.
  acceptance: AC-SF27.3.7 — one of twenty lines rejected for invalid IBAN is corrected and resubmitted alone; total paid equals net payroll exactly once (AT-G20.8).
  dependency: SF28.2.4.
- id: M27.F27.3.SF27.3.8
  name: payslip and salary cost journal
  phase: 4
  release: R1
  actors: [payroll_worker, payroll_officer, employee, ledger_worker]
  screens: [SCR-EMP-payslip, SCR-FIN-journal-entry]
  inputs: [payroll_run_id]
  states: [payslips_generated, published, journal_posted]
  api: POST /v1/properties/{pid}/payroll/runs/{id}:post-journal (Idempotency-Key); GET /v1/me/payslips
  events: [PayslipsPublished, PayrollJournalPosted, PayrollAccrualPosted]
  data: [payslip, journal_entry]
  rules: ["Salary journal aggregates by cost center and pay element group; no employee-level GL lines", "Liabilities for net pay, social insurance, tax and other deductions posted to control accounts", "Month-end accrual if pay period and accounting period differ", "Payslip available only to the employee and HR/payroll"]
  security: payslip access authenticated with MFA option; PDF watermarked with employee id
  failure_cases: [journal_duplicate, payslip_generation_failure]
  finance_report_effect: Labor cost by department in P&L; payroll liabilities reconcile at close.
  i18n_a11y: Payslip bilingual EN/AR, accessible tagged PDF and in-app HTML.
  acceptance: AC-SF27.3.8 — the salary journal has lines per cost center only and equals run totals; the GM P&L shows department labor while individual pay stays hidden (AT-G05.4).
  dependency: F27.4; SF19.1.4.
```

### F27.4 Manager privacy
Section K text (verbatim): *Reports aggregate employee cost by department and period; only HR/payroll roles can drill into individual pay. Payroll records have independent encryption, retention and export controls.*

```yaml
- id: M27.F27.4.SF27.4.1
  name: aggregated labor cost reporting with small-group suppression
  phase: 4
  release: R1
  actors: [gm, owner, department_heads, financial_controller]
  screens: [SCR-GM-home, SCR-FIN-statements]
  inputs: [period, cost_center, min_group_size]
  states: [shown, suppressed_small_group]
  api: GET /v1/properties/{pid}/labor-cost?period=&group_by=department
  events: [LaborCostReportGenerated]
  data: [labor_cost_aggregate]
  rules: ["Non-HR reports show totals by department and period only", "Groups smaller than the minimum size are merged or suppressed so individual pay cannot be inferred", "Generic management reports and pivots in M32 cannot select payroll_result fields"]
  security: aggregation performed inside payroll service; only aggregates leave the schema
  failure_cases: [differencing_attack_via_filters, single_person_department]
  finance_report_effect: Labor cost lines in M32 department P&L.
  i18n_a11y: Suppression explained in text.
  acceptance: AC-SF27.4.1 — a department with one employee is merged into its parent group in GM reports; a pivot request for payroll_result fields by a GM returns 403.
  dependency: D-337; M32.F32.4 SF32.4.8.
- id: M27.F27.4.SF27.4.2
  name: HR/payroll-only individual drill with purpose
  phase: 4
  release: R1
  actors: [hr_officer, payroll_officer, payroll_approver]
  screens: [SCR-HR-payroll-run, SCR-HR-employee-sensitive]
  inputs: [employee_id, purpose_code]
  states: [granted, denied]
  api: GET /v1/properties/{pid}/payroll/results/{employee_id}?period=
  events: [SalaryRecordAccessed]
  data: [payroll_result, salary_access_log]
  rules: ["Individual pay visible only to HR/payroll roles scoped to the legal entity", "Purpose code mandatory and logged", "Managers never see peers' or reports' pay unless granted HR role"]
  security: RLS plus service-level check; step-up MFA per session
  failure_cases: [role_escalation_attempt]
  finance_report_effect: None.
  i18n_a11y: Standard.
  acceptance: AC-SF27.4.2 — a financial_controller without payroll role receives 403 on an individual result and the denial is logged.
  dependency: M02.
- id: M27.F27.4.SF27.4.3
  name: independent payroll encryption
  phase: 4
  release: R1
  actors: [it_admin, dpo]
  screens: [SCR-ADM-key-management]
  inputs: [kms_key_id, rotation_schedule]
  states: [active, rotating, retired]
  api: POST /v1/admin/payroll-keys:rotate
  events: [PayrollKeyRotated]
  data: [payroll_key_ref]
  rules: ["Payroll schema encrypted with a dedicated key separate from general application keys", "Backups of payroll schema encrypted with that key; restore test proves decryptability", "Database administrators without payroll role cannot read decrypted pay fields"]
  security: KMS/Vault with separation of duties
  failure_cases: [key_unavailable, restore_without_key]
  finance_report_effect: None.
  i18n_a11y: Not user-facing beyond admin console accessibility.
  acceptance: AC-SF27.4.3 — a general DB read-only role querying payroll_result sees ciphertext only; key rotation completes without data loss.
  dependency: docs/03 ADR on secrets; M64.F64.2.
- id: M27.F27.4.SF27.4.4
  name: payroll export and retention controls
  phase: 4
  release: R1
  actors: [payroll_officer, dpo, auditor]
  screens: [SCR-HR-exports]
  inputs: [export_type, scope, recipient, justification]
  states: [requested, approved, generated, expired, purged]
  api: POST /v1/properties/{pid}/payroll/exports
  events: [PayrollExportGenerated, PayrollRecordsPurged]
  data: [payroll_export, retention_policy_ref]
  rules: ["Individual-level exports require second approval and expire after download window", "Retention per jurisdiction and record type from M44 (D-340); legal hold supported", "Purge produces proof without deleting required accounting totals"]
  security: watermarked, encrypted exports; access logged
  failure_cases: [export_without_approval, purge_under_legal_hold]
  finance_report_effect: None.
  i18n_a11y: Standard.
  acceptance: AC-SF27.4.4 — an individual export link expires after 24 hours; records under legal hold are skipped by purge.
  dependency: D-340; M02 retention.
- id: M27.F27.4.SF27.4.5
  name: auditor and break-glass access
  phase: 4
  release: R1
  actors: [auditor, dpo, owner]
  screens: [SCR-HR-access-log]
  inputs: [request_reason, scope, time_limit, approver_ids]
  states: [requested, approved, active, expired]
  api: POST /v1/properties/{pid}/payroll/break-glass
  events: [BreakGlassGranted, BreakGlassExpired]
  data: [break_glass_grant, salary_access_log]
  rules: ["Time-limited read-only access with two approvals", "All actions recorded and reviewed after expiry"]
  security: step-up MFA; automatic expiry
  failure_cases: [grant_not_expired]
  finance_report_effect: None.
  i18n_a11y: Standard.
  acceptance: AC-SF27.4.5 — an external auditor gets 48-hour read-only access after two approvals and loses it automatically.
  dependency: M02.
```

### F27.5 WPS, payslip self-service and jurisdiction payroll binding (added — Sections A, C, G steps 5 and 12)

```yaml
- id: M27.F27.5.SF27.5.1
  name: Oman WPS establishment and employee registration data
  phase: 4
  release: R1
  actors: [hr_officer, payroll_officer, compliance_officer]
  screens: [SCR-HR-wps-setup]
  inputs: [employer_establishment_id, employer_bank_code, employee_wps_identifiers, civil_id_or_labour_card_ref, bank_routing]
  states: [incomplete, complete_unverified, bank_validated, ministry_validated]
  api: PUT /v1/properties/{pid}/payroll/wps-profile
  events: [WpsProfileUpdated]
  data: [wps_profile, employee_identity_document]
  rules: ["Fields per the selected bank's WPS specification; status unverified until bank and Ministry validation", "Employees missing required identifiers flagged before cutoff", "No assumption that the same format applies outside Oman"]
  security: HR and payroll only; identifiers encrypted
  failure_cases: [missing_identifier, bank_code_invalid]
  finance_report_effect: None direct.
  i18n_a11y: Bilingual labels matching Ministry terminology once confirmed.
  acceptance: AC-SF27.5.1 — an employee lacking a required WPS identifier appears in pre-run exceptions and the profile shows complete_unverified until bank validation.
  dependency: D-333; Ministry of Labour WPS guidance (source-cited, unverified).
- id: M27.F27.5.SF27.5.2
  name: WPS file format version and validation fixtures
  phase: 4
  release: R1
  actors: [payroll_officer, integration_admin]
  screens: [SCR-ADM-integrations, SCR-HR-wps-export]
  inputs: [format_version, bank_spec_document, fixture_files]
  states: [draft, bank_sample_accepted, production_approved, deprecated]
  api: POST /v1/admin/wps-formats
  events: [WpsFormatApproved]
  data: [wps_format_version]
  rules: ["Format specified from bank-provided spec, versioned, with golden fixtures", "Production use only after bank sample acceptance recorded", "Format change requires regression on fixtures"]
  security: integration admin with maker-checker
  failure_cases: [spec_changed_by_bank, fixture_regression]
  finance_report_effect: None.
  i18n_a11y: Not user-facing.
  acceptance: AC-SF27.5.2 — generating a file with an unapproved format version is blocked for production and allowed in sandbox with watermark.
  dependency: D-333.
- id: M27.F27.5.SF27.5.3
  name: WPS compliance monitor and bank/Ministry feedback
  phase: 4
  release: R1
  actors: [payroll_officer, hr_officer, gm]
  screens: [SCR-HR-wps-status]
  inputs: [wps_submission_id, bank_ack, ministry_status_file_or_portal_evidence]
  states: [submitted, bank_acknowledged, compliant, noncompliant_flagged, resolved]
  api: POST /v1/properties/{pid}/payroll/wps-submissions/{id}/feedback
  events: [WpsSubmissionAcknowledged, WpsNoncomplianceFlagged]
  data: [wps_submission]
  rules: ["Track salary payment within required timelines per verified rule", "Feedback captured via bank file or manual portal evidence; no scraping", "Noncompliance escalates to gm and hr_officer"]
  security: HR and payroll
  failure_cases: [feedback_missing, late_payment]
  finance_report_effect: None direct; late payment risk shown.
  i18n_a11y: Status text bilingual.
  acceptance: AC-SF27.5.3 — a submission without bank acknowledgment by T+2 days escalates; uploaded portal evidence moves it to compliant.
  dependency: D-333.
- id: M27.F27.5.SF27.5.4
  name: payslip self-service
  phase: 4
  release: R1
  actors: [employee]
  screens: [SCR-EMP-payslip]
  inputs: [employee_id, period]
  states: [available, viewed, disputed]
  api: GET /v1/me/payslips; POST /v1/me/payslips/{id}:query
  events: [PayslipViewed, PayslipQueryRaised]
  data: [payslip]
  rules: ["Employees see only own payslips, including after leaving for the retention period", "Queries route to payroll officer with SLA", "Payslip content follows jurisdiction requirements where verified"]
  security: MFA option; session timeout; no payslip in push notification body
  failure_cases: [former_employee_access]
  finance_report_effect: None.
  i18n_a11y: EN/AR payslip, screen-reader friendly HTML, large text.
  acceptance: AC-SF27.5.4 — a push notification says a payslip is available without amounts; a former employee can still download within retention.
  dependency: SF27.3.8.
- id: M27.F27.5.SF27.5.5
  name: Canadian payroll delegation to M38.F38.2
  phase: 4
  release: R1
  actors: [payroll_officer, payroll_worker]
  screens: [SCR-HR-payroll-run]
  inputs: [employee_id, province, sin_token_ref, earnings]
  states: [requested, calculated_by_m38, failed_validation]
  api: POST /v1/internal/m38/canada-payroll:calculate
  events: [CanadianPayrollCalculated]
  data: [payroll_result, statutory_rule_binding]
  rules: ["M27 supplies earnings and province; M38.F38.2 returns federal/provincial tax, CPP or QPP, EI or QPIP and year parameters", "SIN handled only per M38 SF38.2.1; M27 stores a token reference", "T4/RL slips and remittance produced by M38 SF38.2.4/SF38.2.5 from M27 results"]
  security: internal service call with scoped service identity
  failure_cases: [parameters_year_missing, province_not_configured]
  finance_report_effect: Canadian statutory liabilities to control accounts.
  i18n_a11y: English and French considered for Québec per M44 language packs.
  acceptance: AC-SF27.5.5 — a Québec employee run returns QPP and QPIP lines from M38 and a T4/RL artifact is produced by M38 (AT-G12.2).
  dependency: M38.F38.2.
- id: M27.F27.5.SF27.5.6
  name: other-market payroll activation gate
  phase: 4
  release: R1
  actors: [compliance_officer, payroll_officer]
  screens: [SCR-HR-statutory-rules]
  inputs: [jurisdiction, rule_pack_status, manual_path_flag]
  states: [blocked, manual_only, automated]
  api: GET /v1/properties/{pid}/payroll/activation-status
  events: [PayrollJurisdictionActivated]
  data: [statutory_rule_binding]
  rules: ["Pakistan, Saudi Arabia and Portugal payroll rules are not copied from Oman or Canada", "Until verified, payroll for these jurisdictions runs manual_only with external provider or adviser figures entered with evidence", "Automated filing is blocked"]
  security: activation by compliance_officer with counsel evidence
  failure_cases: [activation_without_evidence]
  finance_report_effect: Manual statutory lines labelled.
  i18n_a11y: Status text.
  acceptance: AC-SF27.5.6 — a Portugal fixture property shows payroll manual_only with reviewer and evidence fields (AT-G09.3).
  dependency: D-339; M44.F44.2.
```

**M27 key invariants:** only approved time/leave/overtime enter payroll; preparer ≠ approver ≠ releaser; approval never moves money; bank/WPS confirmation required for paid; rejected lines resubmitted alone after inquiry; salary journal aggregated by cost center; individual pay visible only to HR/payroll roles and every access logged; payroll data under an independent key; unverified rule packs never compute or file automatically.

**M27 module acceptance (Section G):** AT-G05.1 shifts, overtime and leave produce payroll; AT-G05.2 maker-checker with confidential approvals; AT-G05.3 WPS/bank file executed via approved channel or tested file exchange; AT-G05.4 departmental labor posts to P&L while individual salaries stay role-confidential; AT-G12.2 Canadian payroll via M38; AT-G20.8 payroll rejection surfaced and resubmitted once.

**M27 open decisions**

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-333 | Oman payroll bank, WPS file format/version and submission channel | payroll_officer + financial_controller | File exchange via bank portal upload; format from bank spec; status `unverified-assumption` until bank sample accepted and Ministry route confirmed. |
| D-334 | Payroll engine: in-house per jurisdiction vs certified third-party provider (especially Canada) | financial_controller + compliance_officer | In-house for Oman with verified pack; Canada via M38 in-house calculation with CRA parameters or certified provider adapter (decision shared with M38). |
| D-335 | Time capture method (kiosk PIN, mobile geofence, biometric) | hr_officer + dpo | Kiosk PIN and staff app geofence; biometric disabled until lawful basis confirmed. |
| D-336 | Pay frequency, cutoff and value dates | payroll_officer | Monthly, cutoff 25th, value date last working day. |
| D-337 | Minimum group size for aggregated labor reports | dpo + financial_controller | 3 employees. |
| D-338 | Oman end-of-service, leave accrual and social insurance parameters (current labour law) | compliance_officer + local counsel | Configurable parameters; manual evidence required until counsel-reviewed. |
| D-339 | Payroll activation plan for Pakistan, Saudi Arabia, Portugal | compliance_officer | Manual-only with external provider figures until rule packs verified. |
| D-340 | Payroll and HR record retention per jurisdiction | dpo + compliance_officer | Retain payroll registers 10 years unless verified rule differs; identity images shortest lawful period. |

---
