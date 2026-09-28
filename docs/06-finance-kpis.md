# 06 — Finance, Chart of Accounts, Postings, Allocation and KPIs

**Pack:** Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28 • **Modules:** M19, M20, M27 (F27.4), M28, M29, M30, M31, M32, M54, M60, M65, M66, M67 • **Master-prompt sections:** D, E, G.8, K (F19/F20/F27/F32), P.3, P.6

> **Status of this document.** Everything here is a specification/design target. The chart of accounts is a **configurable template inspired by the structure of the Uniform System of Accounts for the Lodging Industry (USALI)** — it is *not* a licensed copy of USALI, not an endorsement by its publishers, and not a statutory chart for any of the five markets. Each property's financial controller adopts, renames or extends it. **No tax, levy, payroll-contribution or withholding rate in this document is a legal value.** Every rate shown in a posting example is an arithmetic placeholder labelled `TEST-ONLY`; real rates come only from a jurisdiction rule pack in status `counsel-reviewed` or better (see `docs/07`, M38/M44).

Decision ids in this document use the range **D-801…D-899** (proposed; consolidated into the decision log in `docs/13`).

---

## 1. Accounting principles the platform enforces

| # | Principle | Mechanism |
|---|---|---|
| P1 | Double entry, always balanced | Every `journal_entry` has ≥2 lines, Σdebit = Σcredit per currency and per legal entity; DB constraint + posting service assertion (SF19.2.1). |
| P2 | Append-only | Posted lines are immutable. Corrections are **reversing entries** linked by `reverses_entry_id` plus a new correct entry (SF19.2.2, P.3). No UPDATE/DELETE grants on ledger tables. |
| P3 | Every line has a source | `source_type` + `source_id` + `source_event_id` (outbox event) + `correlation_id` (SF19.2.3). A line without a source document is rejected; manual journals have `source_type=manual_journal` with attachment and maker-checker. |
| P4 | Exactly-once from events | Posting consumer uses the inbox pattern: `(event_id)` unique; a mapping rule is keyed `(event_type, event_version, rule_version)`; replays produce zero new lines. |
| P5 | Business date ≠ accounting date | Each entry stores `business_date` (hotel operating day from night audit), `accounting_date` (period assignment) and `posted_at` (UTC). A late event for a closed period posts to the first open period with `original_business_date` retained (SF19.1.3). |
| P6 | Money | Integer minor units + ISO-4217; OMR 3 dp, CAD/SAR/PKR/EUR 2 dp (PKR display policy per rule pack). Rounding per tax rule pack; rounding residue posts to `7690 Rounding differences` and is reported. |
| P7 | Dimensions, not account explosion | Natural account × department × cost center × outlet × jurisdiction tag × project/event × counterparty. |
| P8 | Honest status | Every reported amount carries `evidence_status ∈ {planned, estimate, incomplete-estimate, reconciled, certified}` (see §11). |
| P9 | Approval ≠ money movement | AP approval, payment authorization and provider/bank confirmation are separate states and separate GL events (Section D). |
| P10 | Unknown mapping never silently drops | An event with no active mapping rule posts to `2990 Unmapped posting suspense` and raises `GlMappingMissing`; period close is blocked while 2990 ≠ 0. |

---

## 2. Chart-of-accounts template (USALI-inspired, configurable)

### 2.1 Account string

`<legal_entity>-<natural_account>-<department>-<cost_center>[-<outlet>][-<jurisdiction_tag>]` with optional analytic dimensions `event_id`, `counterparty_id`, `project_id`.

- **Legal entity** — from M44 classifier (`LE-…`). One trial balance per legal entity; one property may have one entity in Release 1.
- **Natural account** — 4-digit template below; properties may add 2-digit sub-accounts (`4000.01`).
- **Jurisdiction tag** — mandatory on tax, levy, payroll-statutory and withholding accounts (e.g. `2300.CA-ON.GST_HST`, `2300.OM.VAT`), generated from the active rule pack, never typed by users.

### 2.2 Department codes (operated / undistributed / non-operating)

| Code | Department | Type | Revenue? | Notes |
|---|---|---|---|---|
| `RMS` | Rooms | Operated | Yes | Includes front office, housekeeping, laundry (if in-house, else sub-dept `RMS-LDY`), reservations, guest-facing concierge. |
| `FNB-<outlet>` | Food & beverage outlets (restaurant, room service, minibar, bar) | Operated | Yes | One code per outlet, e.g. `FNB-REST`, `FNB-IRD`, `FNB-MINI`. |
| `BAR` | Bar (if managed as its own department) | Operated | Yes | Property can instead map bar as `FNB-BAR`; D-801 chooses. |
| `CLB` | Club (lounge/nightlife/pool/fitness venue, M15) | Operated | Yes | Admission, membership, minimum spend, hosted bar. |
| `CAT` | Catering / Banquets & conference (M12/M16) | Operated | Yes | Venue hire, AV, catering food/beverage, off-site catering, setup labor fees. |
| `PRK` | Parking (M17) | Operated | Yes | May be "minor operated" in small hotels. |
| `OOD-<code>` | Other operated departments (spa, retail, transport fleet, guest laundry, day-pass, concierge travel fees) | Operated | Yes | Created only when M58/M59/M45 activated for the property. |
| `MIS` | Miscellaneous income | Revenue only | Yes | Cancellation/attrition fees, breakage, commissions received. |
| `AGN` | Administrative & General | Undistributed | No | Finance, purchasing office, PSP fees, bad debt, audit, licences. |
| `SMK` | Sales & Marketing | Undistributed | No | Channel commissions (policy D-802), referral commissions, loyalty program cost, campaigns, website. |
| `POM` | Property Operation & Maintenance / Engineering | Undistributed | No | Engineering labor, maintenance contracts, materials, grounds, cylinder losses (policy). |
| `UTL` | Utilities | Undistributed | No | Electricity, water/sewage, pipeline gas, cylinder gas not charged to outlets (policy D-803). |
| `ITC` | Information & Telecommunications systems | Undistributed | No | Software/SaaS, hardware maintenance, connectivity. |
| `HRM` | Human Resources | Undistributed | No | Recruitment, training (M62), staff welfare. |
| `NOP` | Non-operating (fixed charges) | Below GOP | No | Management/franchise fees, rent, property insurance, property taxes, depreciation, interest, income tax. |
| `BAL` | Balance sheet (no P&L department) | — | — | Used on balance-sheet lines. |

**Cost centers** sit beneath departments (e.g. `RMS-FO`, `RMS-HK`, `FNB-KIT-MAIN`, `CAT-KIT`, `POM-ENG`). The acceptance reference hotel uses two staff cost centers (`CC-RMS-OPS`, `CC-FNB-OPS`), see `docs/09` fixtures.

### 2.3 Balance sheet accounts

| Account | Name | Normal | Sub-ledger / control | Notes |
|---|---|---|---|---|
| 1000 | Cash on hand – house bank/safe | Dr | Cash shift (M60) | Per-cashier floats as sub-accounts. |
| 1010 | Cashier floats | Dr | Cash shift | |
| 1020 | Bank – operating (per bank account) | Dr | Bank statement (SF20.3.1) | |
| 1030 | Bank – payroll/WPS | Dr | Payroll payment batch | Optional separate account. |
| 1040 | PSP clearing – receivable from PSP | Dr | PSP transaction/settlement | Captures not yet settled; cleared by settlement file. |
| 1045 | Card/terminal batch in transit | Dr | Terminal batch | For terminals settling by batch. |
| 1050 | Bill-provider clearing | Dr/Cr | Bill-payment order (M29) | Funds sent to provider awaiting receipt/settlement match. |
| 1060 | Payments in transit – bank transfers out | Cr | Payment batch | Optional where bank debits later than release. |
| 1100 | **Guest ledger** (in-house folios, incl. event master folios) | Dr | Folio (M08) | Must equal Σ open folio balances at night audit. |
| 1110 | **City ledger / AR – corporate & direct bill** | Dr | AR (F20.2) | Per corporate account; aging. |
| 1112 | AR – travel/channel collect (OTA virtual cards, agency) | Dr | AR | |
| 1115 | AR – other operated (off-site catering, club members) | Dr | AR | |
| 1118 | Referrer/partner receivable (commission clawback after payout) | Dr | Referral ledger (M31) | §6 Ex.6b. |
| 1120 | Chargebacks & disputed card receivables | Dr | Dispute case (SF28.1.6) | |
| 1130 | Allowance for doubtful accounts | Cr | AR | Contra. |
| 1200 | Inventory – food (per store) | Dr | Stock ledger (M14/M50) | Lot-level valuation. |
| 1210 | Inventory – beverage | Dr | Stock ledger | |
| 1220 | Inventory – club/bar supplies | Dr | Stock ledger | |
| 1230 | Inventory – housekeeping, linen in store, amenities | Dr | Stock ledger (M56) | Linen in circulation per policy D-804. |
| 1240 | Inventory – engineering spares | Dr | Stock ledger (M26) | |
| 1250 | Inventory – gas in cylinders (full cylinders) | Dr | Cylinder ledger (M25) | Gas content only; shells are supplier property. |
| 1260 | Inventory – quarantine (non-sellable, pending decision) | Dr | Quarantine ledger | Never available for issue (SF50.3.5). |
| 1300 | Prepaid expenses | Dr | Prepayment schedule (SF19.1.4) | Insurance, SaaS, licences. |
| 1310 | **Cylinder deposits paid** (with gas supplier) | Dr | Cylinder custody ledger | Count × deposit per cylinder type; §6 Ex.12. |
| 1320 | Supplier advances / prepayments | Dr | AP | |
| 1325 | Staff advances / loans receivable | Dr | Payroll (confidential) | |
| 1330.`<jur>` | Input tax recoverable (per jurisdiction/tax code) | Dr | Tax engine | Only where rule pack allows recovery. |
| 1400–1480 | Property, plant & equipment (by class) | Dr | Asset register (M26/M66) | |
| 1490 | Accumulated depreciation | Cr | Asset register | |
| 2000 | **AP – trade** | Cr | AP invoice (F20.1) | |
| 2010 | AP – utilities & bill providers | Cr | Utility bill (M22–M24) | |
| 2020 | **GRNI – goods/services received not invoiced** | Cr | GRN / service acceptance (SF21.2) | Must be explained line-by-line at close. |
| 2030 | Accrued utilities (estimate) | Cr | Utility accrual run | Carries `evidence_status`. |
| 2040 | Accrued expenses – other | Cr | Accrual schedule | |
| 2100 | **Advance deposits – guest reservations** | Cr | Deposit ledger (M08) | |
| 2105 | Advance deposits – groups, events, catering | Cr | Event billing (M12/M16) | |
| 2107 | Deferred revenue – club memberships/passes | Cr | Membership (M15) | Recognized over entitlement period. |
| 2110 | Guest ledger credit balances | Cr | Folio | Reclass at close if folios in credit. |
| 2120 | **Gift voucher liability** | Cr | Voucher ledger (M54) | Issued − redeemed − breakage recognized. |
| 2130 | **Loyalty points liability** (deferred revenue or cost accrual per D-805) | Cr | Points ledger (M30) | Reconciles to Σ outstanding points × valuation version. |
| 2140 | **Referral commission accrual (pending)** | Cr | Referral commission ledger (M31) | Pending = earned per formula, not yet approved. |
| 2145 | Referral commission payable (approved) | Cr | Referral commission ledger | |
| 2150 | Tips payable to staff | Cr | POS tips (M13) | Pass-through; paid via payroll or cash-out per D-806. |
| 2155 | Service charge distribution payable | Cr | POS / folio | Only when service charge is distributed (D-806). |
| 2160 | Travel supplier payable (concierge orders as agent) | Cr | Travel order (M45) | Only where hotel collects on behalf. |
| 2200 | **Net salaries payable** | Cr | Payroll run (M27) | Cleared only by bank/WPS confirmation. |
| 2210.`<jur>` | Employee statutory withholdings payable (income tax, social insurance, CPP/QPP/EI/QPIP where applicable) | Cr | Payroll | Per jurisdiction/program. |
| 2220.`<jur>` | Employer statutory contributions payable | Cr | Payroll | |
| 2230 | Accrued leave, end-of-service/gratuity provisions | Cr | Payroll provisions | Method per rule pack/counsel. |
| 2240 | Payroll clearing (rejected/returned salary payments) | Cr | WPS/bank rejection | Must be zero or explained at close. |
| 2250 | Other payroll deductions payable (unions, garnishments, benefits) | Cr | Payroll | |
| 2300.`<jur>.<code>` | **Output tax payable (per jurisdiction & tax code)** | Cr | Tax engine (M38) | e.g. `2300.OM.VAT`, `2300.CA.GST_HST`, `2300.CA-QC.QST`, `2300.PT.IVA`, `2300.SA.VAT`, `2300.PK-<prov>.ST` — codes are placeholders until rule pack verified. |
| 2310.`<jur>.<code>` | Accommodation / tourism / municipal levies payable | Cr | Tax engine | |
| 2320.`<jur>` | Withholding tax payable (on supplier/referrer payments) | Cr | Tax engine | Applies only per verified rule. |
| 2330 | Tax settlement clearing | Dr/Cr | Filing/remittance (SF38.1.5) | |
| 2400 | Customer cylinder deposits held | Cr | Cylinder custody | Only if hotel lends cylinders to off-site catering clients (rare). |
| 2500 | Bank/PSP reconciliation suspense | Dr/Cr | Reconciliation (F20.3) | Unmatched statement lines; must be aged. |
| 2600 | Intercompany / owner current account | Dr/Cr | M66 | |
| 2700–2790 | Loans, leases (liability) | Cr | M66 | Authorized source only (SF66.1.4). |
| 2990 | **Unmapped posting suspense** | — | Mapping engine | Close blocked while ≠ 0. |
| 3000 | Owner capital | Cr | — | |
| 3100 | Retained earnings | Cr | — | |
| 3200 | Current-year result | Cr | Year-end close | |
| 3900 | Opening balance equity (migration only) | — | Migration | Must be zero after migration sign-off (`docs/09` §12). |

### 2.4 Revenue accounts

| Account | Name | Default dept | Notes |
|---|---|---|---|
| 4000 | Room revenue – transient | RMS | Segment dimension (direct, OTA, walk-in). |
| 4010 | Room revenue – group/event block | RMS | |
| 4020 | Room revenue – corporate negotiated | RMS | |
| 4030 | Day-use / timed-room revenue | RMS | |
| 4040 | Upgrades, early arrival, late checkout (M54) | RMS | |
| 4050 | No-show & late-cancellation revenue | RMS | Tax treatment per rule pack. |
| 4090 | Rooms allowances & rebates (contra) | RMS | Service-recovery credits (M55). |
| 4100 | Food revenue | FNB-`<outlet>` | |
| 4110 | Beverage revenue | FNB-`<outlet>` / BAR | |
| 4120 | In-room dining revenue | FNB-IRD | |
| 4130 | Minibar revenue | FNB-MINI or RMS (D-807) | |
| 4150 | Service charge retained (only when policy = retained revenue) | Outlet | D-806. |
| 4190 | F&B allowances (contra) | Outlet | |
| 4300 | Club admission revenue | CLB | |
| 4310 | Club membership revenue (released from 2107) | CLB | |
| 4320 | Club minimum-spend shortfall revenue | CLB | |
| 4330 | Club beverage/food revenue | CLB | Or FNB account with outlet=CLB (D-801). |
| 4400 | Catering/banquet food revenue | CAT | |
| 4410 | Catering/banquet beverage revenue (incl. hosted bar) | CAT | |
| 4420 | Venue/room hire | CAT | |
| 4430 | AV & equipment rental | CAT | |
| 4440 | Setup/labor/service fees (not service charge) | CAT | |
| 4450 | Off-site catering revenue | CAT | |
| 4460 | Attrition / cancellation fees (events) | CAT or MIS (D-808) | |
| 4500 | Parking revenue – transient | PRK | |
| 4510 | Parking revenue – permits/passes | PRK | |
| 4600 | Other operated revenue | OOD-`<code>` | Spa, retail, transport, guest laundry. |
| 4610 | Concierge travel service fees / commissions (agent model) | OOD-TRV | Only where licensed model (M45 gate). |
| 4700 | Gift voucher breakage | MIS | Only where rule pack permits (unclaimed-property rules). |
| 4710 | Points breakage (expiry) | MIS or contra-SMK (D-805) | |
| 4720 | Commissions received | MIS | |
| 4790 | Other miscellaneous income | MIS | |

### 2.5 Cost of sales and expense accounts (department supplied by dimension)

| Account | Name | Typical departments |
|---|---|---|
| 5000 | Cost of food sold | FNB-*, CAT, CLB |
| 5010 | Cost of beverage sold | FNB-*, BAR, CAT, CLB |
| 5020 | Cost of other goods sold (retail, minibar goods) | OOD, FNB-MINI |
| 5030 | Complimentary food & beverage (staff meal, comps) — reclass | FNB → receiving dept |
| 5090 | Purchase price variance | FNB-*, CAT, POM |
| 6000 | Salaries & wages – regular | all |
| 6010 | Overtime & holiday premium | all |
| 6020 | Employer statutory contributions | all |
| 6030 | Employee benefits, allowances, staff meals | all |
| 6040 | Leave and end-of-service provision expense | all |
| 6050 | External/contract labor (agency, emergency chef M47) | FNB/CAT/RMS/POM |
| 6100 | Operating supplies & guest supplies | RMS, FNB, CLB, CAT |
| 6110 | Linen (replacement, loss, damage) | RMS, FNB, CAT |
| 6120 | Laundry & dry cleaning (outsourced) | RMS, FNB, CAT |
| 6130 | Cleaning supplies | RMS, FNB |
| 6140 | Decorations, menus, printing | FNB, CAT |
| 6150 | Cylinder gas consumed (outlet-charged policy) | FNB, CAT, CLB, RMS-LDY |
| 6160 | Cylinder losses & deposit forfeits | POM or consuming outlet (D-803) |
| 6200 | Commissions – OTA/channel/travel agent | RMS (USALI-style) or SMK (D-802) |
| 6210 | Referral commission expense (MetriStay Network, marketing expense) | SMK |
| 6220 | Loyalty program cost (cost-accrual model only) | SMK |
| 6230 | PSP / card processing fees and chargeback fees | AGN |
| 6240 | Bill-provider convenience fees | AGN |
| 6250 | Bad debt expense & chargeback losses | AGN |
| 6260 | Cash over/short | AGN |
| 6300 | Electricity | UTL |
| 6310 | Water & sewage | UTL |
| 6320 | Pipeline gas | UTL |
| 6330 | Cylinder gas (undistributed policy) | UTL |
| 6340 | Other fuel | UTL |
| 6400 | Maintenance contracts & contractor services | POM |
| 6410 | Maintenance materials & spares consumed | POM |
| 6420 | Grounds, pest control, waste removal | POM |
| 6500 | Software, SaaS, licences, connectivity | ITC |
| 6510 | IT hardware maintenance | ITC |
| 6600 | Advertising, campaigns, website, media production | SMK |
| 6610 | Metasearch/search advertising | SMK |
| 6700 | Inventory waste & spoilage (approved) | consuming dept |
| 6710 | Inventory shrinkage (unexplained count variance) | store owner dept |
| 6720 | Recall write-off | FNB/CAT |
| 6800 | Professional fees (audit, legal, tax, counsel) | AGN |
| 6810 | Licences, permits, inspections | AGN/POM |
| 6820 | Recruitment, training, staff learning | HRM |
| 6900 | Allocated shared cost (optional GL allocation policy only) | receiving dept (§7.4) |
| 6901 | Allocated shared cost – contra (pool) | pool dept |
| 7000 | Management fees (base/incentive) | NOP |
| 7010 | Franchise/brand fees | NOP |
| 7100 | Rent & leases | NOP |
| 7200 | Property & liability insurance | NOP |
| 7300 | Property/municipal taxes (non-income) | NOP |
| 7400 | Depreciation & amortization | NOP |
| 7500 | Interest expense | NOP |
| 7600 | Realized FX gain/loss | NOP |
| 7610 | Unrealized FX gain/loss (revaluation) | NOP |
| 7690 | Rounding differences | AGN |
| 7700 | Income tax expense | NOP |
| 7800 | Gain/loss on asset disposal | NOP |

**Configurable policy decisions (owner: `financial_controller`, all logged D-801…D-810):** bar as own department vs F&B outlet (D-801); channel commission in Rooms vs S&M (D-802); cylinder gas charged to outlet vs undistributed utilities (D-803); linen in circulation capitalized vs expensed (D-804); loyalty accounting model — *deferred revenue* (relative standalone-selling-price allocation to points, default assumption) vs *cost accrual* (D-805, requires auditor/counsel sign-off per legal entity); tips and service charge — pass-through liability vs retained revenue, per jurisdiction rule pack and employment contracts (D-806); minibar under Rooms vs F&B (D-807); event attrition fees in CAT vs MIS (D-808); GL-posted allocation vs management-layer allocation (D-809, default management layer); voucher breakage recognition method — proportional to redemption vs at expiry, subject to unclaimed-property rules (D-810).

---

## 3. Posting engine design (M19 F19.1/F19.2)

- **Mapping rule:** `gl_mapping_rule(id, event_type, event_version, condition_expr, line_templates[], rule_version, effective_from, effective_to, approved_by, status)`. Conditions test dimensions (department, outlet, product class, tax code, jurisdiction, payer type). Line templates reference amount components from the event payload (`net`, `tax[code]`, `levy[code]`, `service_charge`, `tip`, `fee`, `allocation[component]`).
- **Rule change control:** new rule version is maker-checker approved, dry-run against the last 30 business days of events with a diff report, activated at a future accounting date; never retroactively re-posts closed periods.
- **Posting timing:** real-time for cash-affecting and folio events; batched at night audit for room & tax auto-posting; at period close for accruals, revaluation, allocation (if GL-posted), provisions.
- **Sub-ledger to GL control accounts:** Guest ledger (1100), City ledger (1110), Advance deposits (2100/2105), Points (2130), Vouchers (2120), Referral (2140/2145), AP (2000/2010), GRNI (2020), Payroll (2200–2250), Inventory (12xx), Cylinder deposits (1310). Each has a **daily control-total check**: Σ sub-ledger = GL control balance; difference raises `SubledgerOutOfBalance` into the close checklist.
- **Tax lines:** produced only by the tax engine (M38) from the rule pack version effective on the tax point date; the journal line stores `rule_pack_id`, `rule_version`, `tax_code`, `taxable_base`, `rate_ref` (not the rate as free text). If the rule pack is `unverified-assumption`/`draft`, lines post with `evidence_status=estimate` and the tax filing export for that jurisdiction is blocked (M44 gate).

---

## 4. Event → GL mapping table

Legend: **Dr/Cr** show the default template; department in brackets. "GL: none" means sub-ledger/memo only (still audited). All events carry `idempotency_key`/`event_id`; "Reversal" states how corrections post. `Tx` = output tax per tax code; `Lv` = levy per code.

### 4.1 Rooms, folio, deposits and payments (M04, M05, M08, M28)

| # | Event (domain event) | Trigger / source | Debit | Credit | Timing & reversal |
|---|---|---|---|---|---|
| EV-01 | Room & tax posting (`RoomChargePosted`) | Night audit per occupied stay-night, or on day-use check-out | 1100 Guest ledger | 4000/4010/4020/4030 [RMS] net; 2300.`<jur>` Tx; 2310.`<jur>` Lv | Business date of the stay night. Correction = `FolioChargeReversed` (mirror) + re-post. |
| EV-02 | Package allocation (`PackageChargePosted`) | Package rate night | 1100 | Components by allocation version (rooms 4000, food 4100 [FNB-outlet], parking 4510 [PRK], etc.) + tax per component | Allocation by relative standalone selling price (default) or fixed component amount (D-811); allocation version stored per line. |
| EV-03 | Ancillary folio charge (`FolioChargePosted`) | Upsell, minibar, laundry, AV, misc | 1100 | Revenue by product class + Tx | Reversal mirror with reason code & approval above threshold. |
| EV-04 | Folio allowance / rebate (`FolioAllowancePosted`) | Service recovery (M55), rate adjustment | 4090/4190 contra [dept] + 2300 Tx (reduction, if rule pack allows) | 1100 | Approval per cap (SF55.2.3). |
| EV-05 | Folio transfer (`FolioTransferred`) | Routing between windows/folios/master | 1100 (target) | 1100 (source) | Net zero in GL; sub-ledger audit only. |
| EV-06 | Transfer to city ledger (`FolioTransferredToAR`) | Checkout with direct bill | 1110 City ledger [counterparty] | 1100 | Credit limit check (SF20.2.5). |
| EV-07 | Advance deposit received (`DepositReceived`) | Reservation/booking engine payment | 1040 PSP clearing / 1020 Bank / 1000 Cash | 2100 (or 2105 for groups/catering) | Tax on deposits only if rule pack says (then 2100 net + Tx). |
| EV-08 | Deposit applied (`DepositApplied`) | Check-in or event billing | 2100/2105 | 1100 | |
| EV-09 | Deposit forfeited (`DepositForfeited`) | Cancellation outside policy | 2100/2105 | 4050 / 4460 + Tx per rule pack | |
| EV-10 | Deposit refunded (`DepositRefunded`) | Cancellation within policy | 2100/2105 | 1040 / 1020 | Once per refund id; see EV-15. |
| EV-11 | Pre-authorization (`PaymentAuthorized`) | Card guarantee/incidentals | GL: none (memo `payment_authorization` with amount/expiry) | — | Expiry/void releases memo. |
| EV-12 | Capture / settlement tender (`PaymentCaptured`) | Checkout or pay-now | 1040 (card/PSP), 1000 (cash), 1020 (transfer) | 1100 (or 1110 when paying AR) | Idempotent on PSP `capture_id`. |
| EV-13 | Void before capture (`PaymentVoided`) | Cancel auth | GL: none | — | |
| EV-14 | PSP settlement (`PspSettlementReconciled`) | Settlement file/statement | 1020 Bank (net), 6230 PSP fees [AGN] | 1040 PSP clearing (gross) | Unmatched lines → 2500 suspense. |
| EV-15 | Refund (`PaymentRefunded`) | Guest refund | 1100 (restores folio credit) — or directly the revenue reversal via EV-03 reversal | 1040 / 1020 / 1000 | Refund id unique; duplicate webhook = no posting. Linked points reversal EV-50 and referral EV-62 fire from the same causation. |
| EV-16 | Chargeback received (`ChargebackOpened`) | PSP dispute notice | 1120 Chargebacks receivable; 6230 fee | 1040 | |
| EV-17 | Chargeback won (`ChargebackWon`) | PSP decision | 1040 | 1120 | |
| EV-18 | Chargeback lost (`ChargebackLost`) | PSP decision | 6250 [AGN] (or revenue reversal if policy) | 1120 | Triggers points/referral reversal where the stay was the basis. |
| EV-19 | No-show / late cancel fee (`NoShowCharged`) | Night audit | 1100 | 4050 + Tx per rule pack | Captured against guarantee via EV-12. |
| EV-20 | Cash shift over/short (`CashShiftClosed`) | Cashier close (M60) | 6260 or 1000 | 1000 or 6260 | Above tolerance → manager approval. |
| EV-21 | Corporate AR receipt (`ArReceiptAllocated`) | Bank import allocation (SF20.2.4) | 1020 | 1110 | Unallocated cash → 2500 until matched. |
| EV-22 | AR write-off (`ArWrittenOff`) | Approved write-off | 1130 / 6250 | 1110 | Maker-checker (SF20.2.5). |
| EV-23 | Guest ledger credit reclass (`GuestCreditReclassified`) | Period close | 1100 | 2110 | Auto-reversed first day of next period. |

### 4.2 F&B, bar, club, catering, parking (M12–M17, M54, M57)

| # | Event | Trigger | Debit | Credit | Timing & reversal |
|---|---|---|---|---|---|
| EV-24 | POS sale settled (`PosCheckClosed`) | Check closed (tender cash/card/room/corporate/event) | 1000 / 1040 / 1100 (room charge) / 1110 (direct bill) | 4100/4110 [outlet] net; 2300 Tx; 2155 service charge (pass-through) **or** 4150 (retained); 2150 tips | Business date = outlet business date. |
| EV-25 | POS void (`PosItemVoided`) | Before check close | GL: none (audit + void report, SF60.2.2) | — | |
| EV-26 | POS post-close correction (`PosCheckReversed`) | After close; approval | Mirror of EV-24 | Mirror | Reason code; outlet manager approval. |
| EV-27 | POS comp/discount (`PosDiscountApplied`) | Approved comp | 4190 contra (discount) or 5030 (comp, cost-based) | 4100/4110 or inventory | Policy D-812. |
| EV-28 | Tips paid out (`TipsDistributed`) | Payroll or cash-out | 2150 | 2200 (via payroll) or 1000 | Distribution rules per D-806. |
| EV-29 | Club admission (`ClubAdmissionCharged`) | Entry scan/sale | 1100/1040/1000 | 4300 [CLB] + Tx | Capacity check precedes sale (no charge if denied). |
| EV-30 | Club membership sold (`MembershipSold`) | Membership sale | 1040/1110 | 2107 deferred + Tx at invoice point per rule pack | Released monthly EV-31. |
| EV-31 | Membership revenue release (`MembershipRevenueRecognized`) | Period job | 2107 | 4310 [CLB] | Straight-line over entitlement. |
| EV-32 | Minimum-spend shortfall (`MinimumSpendSettled`) | Table/booking close | 1100/1110 | 4320 [CLB] + Tx per rule pack | Shortfall = max(0, min spend − eligible net spend). |
| EV-33 | Catering/event deposit (`EventDepositReceived`) | Contract signature | 1040/1020 | 2105 | |
| EV-34 | Event charge – contracted item (`EventChargePosted`) | BEO billing line at event date | 1100 (event master folio) | 4400/4410/4420/4430/4440 [CAT], 4010 [RMS], 4300 [CLB], 4510 [PRK] + Tx | Billed qty = max(guarantee, actual) per contract clause; both stored. |
| EV-35 | Event actual-vs-contracted adjustment (`EventActualsReconciled`) | Post-event reconciliation (M12) | 1100 (increase) or revenue contra (decrease) | Revenue (increase) or 1100 (decrease) | Change-order id required; corporate approval evidence. |
| EV-36 | Event attrition/cancellation fee (`EventAttritionCharged`) | Contract clause | 1110 or 2105 | 4460 + Tx per rule pack | |
| EV-37 | Parking session charge (`ParkingSessionCharged`) | LPR/gate exit or night audit for permits | 1100 / 1040 / 1000 / 1110 | 4500/4510 [PRK] + Tx | Unique `parking_session_id`; duplicate LPR events = no posting. Free/complimentary session → GL: none, statistics only. |
| EV-38 | Gift voucher issued (`VoucherIssued`) | Sale | 1040/1000 | 2120 (+ Tx at issue only if single-purpose voucher per rule pack) | |
| EV-39 | Gift voucher redeemed (`VoucherRedeemed`) | Tender on folio/POS | 2120 | 1100 / POS tender clearing | Revenue already posted by the underlying sale. |
| EV-40 | Voucher breakage (`VoucherExpired` / breakage run) | Expiry or proportional method | 2120 | 4700 [MIS] | Blocked if rule pack flags unclaimed-property remittance (then Cr 2040/escheat payable). |

### 4.3 Loyalty points and referral (M30, M31)

| # | Event | Trigger | Debit | Credit | Timing & reversal |
|---|---|---|---|---|---|
| EV-50 | Points earned – pending (`PointsEarnedPending`) | Eligible paid transaction | Deferred-revenue model: revenue account of the source sale (allocation) | 2130 | Amount = points × valuation version (standalone selling price × expected redemption). Cost model: Dr 6220 [SMK] Cr 2130. |
| EV-51 | Points become available (`PointsReleased`) | After stay/refund window | GL: none | — | Status change only. |
| EV-52 | Points redeemed (`PointsRedeemed`) | Tender on eligible purchase | 2130 | 1100 / POS tender clearing | Tax on redemption per rule pack. |
| EV-53 | Points expired (`PointsExpired`) | Expiry job | 2130 | 4710 [MIS] (deferred-revenue model) or 6220 contra (cost model) | If breakage is estimated up front, only true-up posts. |
| EV-54 | Points reversed (`PointsReversed`) | Refund/chargeback/cancellation of source | 2130 | Source revenue account (restores allocation) | Exactly once per `(source_txn_id, reversal_reason)`; if points already redeemed → negative balance handling D-813. |
| EV-55 | Points valuation change (`PointsValuationRevised`) | Approved new valuation version | 2130 or source contra | 4710 / 6220 | Prospective; restatement report shows effect. |
| EV-60 | Referral commission accrued (`ReferralCommissionAccrued`) | After checkout + cleared payment + refund window; margin formula version | 6210 [SMK] | 2140 | Commission = max(0, eligible margin) × contract rate. Zero margin → no entry (statement line "0 – not eligible"). |
| EV-61 | Referral commission approved (`ReferralCommissionApproved`) | Finance + compliance approval; jurisdiction gate `open` | 2140 | 2145 (net) + 2320.`<jur>` withholding if verified rule applies | Blocked if jurisdiction/referrer category gate is closed. |
| EV-62 | Referral clawback before payout (`ReferralCommissionReversed`) | Refund/chargeback/amendment | 2140 or 2145 | 6210 | Once per commission id. |
| EV-63 | Referral payout (`ReferralPayoutConfirmed`) | Authorized bank/PSP confirmation | 2145 | 1020 / 1040 | Server-side gate re-check at release **and** at confirmation. |
| EV-64 | Clawback after payout (`ReferralClawbackRaised`) | Reversal after payout | 1118 Referrer receivable | 6210 | Offset against future commission or collection per contract. |

### 4.4 Procurement, AP, utilities, cylinders, maintenance (M20–M26, M49, M50)

| # | Event | Trigger | Debit | Credit | Timing & reversal |
|---|---|---|---|---|---|
| EV-70 | PO issued / budget encumbrance (`PurchaseOrderIssued`) | Award → PO | GL: none (commitment ledger/encumbrance) | — | Encumbrance released on receipt/cancel. |
| EV-71 | Goods received – stock item (`GoodsReceiptAccepted`) | Accepted quantity at PO price | 12xx inventory [store] | 2020 GRNI | One entry per accepted receipt line; duplicate scans idempotent (SF50.2.7). Quarantined qty: Dr 1260 Cr 2020 until accepted/rejected. |
| EV-72 | Goods received – direct expense (`GoodsReceiptAccepted`, non-stock) | Non-stock PO | Expense [dept] | 2020 | |
| EV-73 | Service accepted (`ServiceAccepted`) | Maintenance/service completion evidence | 6400/6050/6120 [dept] or asset 14xx (capitalize decision SF21.2.4) | 2020 | |
| EV-74 | Supplier invoice matched (`SupplierInvoiceApproved`) | 3-way match within tolerance or approved variance | 2020 GRNI (matched value); 5090 price variance; 1330 input tax | 2000 AP | Duplicate invoice key blocks posting. |
| EV-75 | Non-PO invoice (`SupplierInvoiceApproved`, 2-way) | Utilities, subscriptions, rent | Expense/prepaid/accrual clearance; 1330 | 2000/2010 | |
| EV-76 | Supplier credit note (`SupplierCreditNoteApproved`) | Short/damaged/return | 2000 | 12xx / GRNI / expense; 1330 reversal | |
| EV-77 | Return to vendor (`GoodsReturnedToVendor`) | Rejected/quarantined goods | 2020 (if not invoiced) or 1118/2000 | 1260/12xx | |
| EV-78 | Payment approved (`PayablePaymentApproved`) | AP approval | GL: none (payment proposal state) | — | Separation: approver ≠ releaser. |
| EV-79 | Payment released to bank (`PaymentBatchReleased`) | Payment releaser | GL: none or 2000 → 1060 in-transit (policy D-814) | — | |
| EV-80 | Payment confirmed by bank (`SupplierPaymentConfirmed`) | Bank statement / confirmation | 2000/2010 | 1020 (or 1060) | Partially paid invoices remain open for remainder. |
| EV-81 | Bank rejection (`SupplierPaymentRejected`) | Bank return | 1060 reversal (if used) | — | Invoice returns to `approved-unpaid`. |
| EV-82 | Utility accrual – estimate (`UtilityAccrualPosted`) | Period close, meter/estimate | 6300/6310/6320 [UTL] | 2030 | `evidence_status=estimate` or `incomplete-estimate` (§11). |
| EV-83 | Utility bill posted (`UtilityBillApproved`) | Bill import & variance check | 2030 (accrued amount for bill period) + 63xx (true-up difference, ±) + 1330 | 2010 | True-up posts in the current open period tagged `relates_to_period`. |
| EV-84 | Bill-pay pending (`BillPaymentSubmitted` / `BillPaymentPending`) | Provider submission | GL: none (payment-in-flight lock prevents re-payment) | — | Timeout never re-submits without inquiry (Section E). |
| EV-85 | Bill-pay confirmed (`BillPaymentConfirmed`) | Provider final status + receipt no. | 2010 AP utilities; 6240 convenience fee [AGN] | 1050 Bill-provider clearing (or 1020 if paid via bank) | Receipt number mandatory. |
| EV-86 | Bill-provider settlement (`BillProviderSettlementReconciled`) | Provider/bank statement | 1050 | 1020 | Differences → 2500. |
| EV-87 | Bill-pay failed/reversed (`BillPaymentFailed` / `Reversed`) | Provider status | 1050 (if funds returned) | 2010 re-opened | |
| EV-88 | Cylinder delivered full (`CylinderReceived`) | Exchange/receipt | 1250 gas content (fill price) | 2020 GRNI | Custody count ↑ full. |
| EV-89 | New cylinder deposit paid (`CylinderDepositPaid`) | First issue of shell | 1310 | 2020/2000 | |
| EV-90 | Empty returned in exchange (`CylinderReturned`) | Exchange | GL: none (custody count; deposit carried) | — | |
| EV-91 | Deposit refunded (`CylinderDepositRefunded`) | Shell returned without replacement | 1020/2000 offset | 1310 | |
| EV-92 | Cylinder lost/damaged (`CylinderLost`) | Custody variance | 6160 [dept per D-803] | 1310 | Incident link if safety-related. |
| EV-93 | Cylinder consumption issue (`CylinderIssued`) | Connected to outlet/event | 6150 [outlet] or 6330 [UTL] | 1250 | |
| EV-94 | Pipeline gas bill (`UtilityBillApproved` type gas) | as EV-83 | as EV-83 with 6320 | 2010 | Standing/variable components kept as lines for allocation. |

### 4.5 Inventory and COGS (M14, M50, M56)

| # | Event | Trigger | Debit | Credit | Notes |
|---|---|---|---|---|---|
| EV-100 | Store transfer (`StockTransferred`) | Store → store | 12xx [dest store] | 12xx [source store] | Same valuation; GL: none if same account & dept (dimension move only). |
| EV-101 | Issue to consuming outlet (`StockIssued`) | Issue to kitchen/bar/BEO/housekeeping | 5000/5010 [dept/outlet/event] or 6100/6410 | 12xx | Default "cost at issue" model (D-815). Alternative "periodic COGS" model: issues are location moves; COGS = opening + purchases − closing at stocktake. |
| EV-102 | Intact return to store (`StockReturnedIntact`) | Inspector-approved return with original lot/expiry | 12xx | 5000/5010 [dept] | Only intact items; never waste. |
| EV-103 | Waste/spoilage recorded (`StockWasted`) | Approved waste transaction (reason/photo/lot) | 6700 [dept] | 12xx or 1260 | Quantity moves to waste bin, never back to available. |
| EV-104 | Quarantine (`StockQuarantined`) | Receipt/issue exception | 1260 | 12xx | Status change; no P&L. |
| EV-105 | Stocktake shrinkage (`StocktakeVariancePosted`) | Blind count vs book | 6710 [store dept] (loss) | 12xx (or reverse for gain) | Approval above tolerance. |
| EV-106 | Recall write-off (`RecallWriteOff`) | Recall case | 6720 | 12xx/1260 | Supplier claim → 1118/2000 credit. |
| EV-107 | Theoretical POS depletion (`TheoreticalDepletionComputed`) | POS sale × recipe BOM | GL: none (variance analytics only) | — | Theory vs actual variance report (SF50.3.7). |
| EV-108 | Staff meal / comp reclass (`StaffMealRecorded`) | Staff meal issue | 6030 [receiving dept] | 5000 | |

### 4.6 Payroll (M27, M38) — aggregate postings only

| # | Event | Trigger | Debit | Credit | Notes |
|---|---|---|---|---|---|
| EV-110 | Payroll run approved (`PayrollRunApproved`) | Maker-checker approval | 6000/6010/6030 [dept, cost center] gross; 6020 employer contributions | 2200 net; 2210.`<jur>` withholdings; 2220.`<jur>` employer contributions; 2250 other deductions; 1325 advance recovery | **Journal lines aggregate by department × cost center × natural account.** Individual pay stays in encrypted payroll sub-ledger (§8). |
| EV-111 | Payroll accrual at period end (`PayrollAccrued`) | Days worked not yet paid | 6000/6010 [dept] | 2040 | Auto-reverse next period. |
| EV-112 | Provision movements (`PayrollProvisionPosted`) | Leave / end-of-service | 6040 [dept] | 2230 | Method per jurisdiction rule pack. |
| EV-113 | WPS/bank file submitted (`SalaryPaymentFileSubmitted`) | File exchange | GL: none (batch pending) | — | |
| EV-114 | Salary payment confirmed (`SalaryPaymentConfirmed`) | Bank/WPS confirmation per employee batch | 2200 | 1030/1020 | Only confirmed items. |
| EV-115 | Salary payment rejected (`SalaryPaymentRejected`) | Bank return/rejection | 2200 stays open (no posting) or 1030 → 2240 if bank debited then returned | — | Case to `payroll_officer`; resubmission creates new file id. |
| EV-116 | Statutory remittance paid (`PayrollRemittanceConfirmed`) | Authority payment receipt | 2210/2220 | 1020 | Receipt reference required. |
| EV-117 | External/emergency chef invoice (`ServiceAccepted` M47) | Shift completion | 6050 [FNB/CAT] | 2020 → 2000 | Not payroll; AP flow. |

### 4.7 FX, tax, night audit, period close (M19, M38)

| # | Event | Trigger | Debit | Credit | Notes |
|---|---|---|---|---|---|
| EV-120 | Foreign-currency transaction (`*` with `txn_currency ≠ functional`) | Any | Functional-currency amounts at rate version `fx_rate_id` | | Both currencies stored per line. |
| EV-121 | Realized FX (`FxRealized`) | Settlement at different rate | 7600 (loss) | 7600 (gain) | |
| EV-122 | Unrealized FX revaluation (`FxRevaluationPosted`) | Period close for open monetary items | 1110/2000/1020 or 7610 | 7610 or … | Auto-reverse next period. |
| EV-123 | Tax filing/remittance (`TaxRemittanceConfirmed`) | Return filed & paid with receipt | 2300/2310 − 1330 | 1020 / 2330 | Only for rule packs `counsel-reviewed`+ and submission receipt or labelled manual handoff. |
| EV-124 | Night audit roll (`BusinessDateClosed`) | Night audit completed | Triggers EV-01/EV-19/EV-37 batches; GL: none itself | — | Locks business date for operational postings (reopen = controlled, audited). |
| EV-125 | Period close (`AccountingPeriodClosed`) | Close checklist complete | Accrual/reversal/reval/allocation jobs | | Period locked; later corrections in next open period. |
| EV-126 | Year-end close (`FiscalYearClosed`) | Year-end | P&L accounts | 3100 | |
| EV-127 | Manual journal (`ManualJournalPosted`) | Controller | per entry | per entry | Maker-checker, attachment, reason; never on sub-ledger control accounts (blocked). |

---

## 5. Conventions for the worked examples

- Currency OMR (3 decimals) unless stated. Functional currency = OMR for the reference legal entity `LE-G-01` (acceptance hotel `FX-HOTEL-G`, `docs/09` §3).
- `TT` = **TEST-ONLY output tax at 10 %**; `LV` = **TEST-ONLY accommodation levy at 2 %**; `ESD` = **TEST-ONLY employee statutory deduction 5 %**; `ERC` = **TEST-ONLY employer contribution 8 %**. These are arithmetic placeholders from rule pack `RP-TEST-GENERIC-v1` and **are not legal values for any jurisdiction**. Whether a given item (deposit, no-show fee, service charge, redemption, deposit on cylinder shells) is taxable is decided by the verified rule pack, not by these examples.
- Every table balances; the "Check" row shows Σ Dr = Σ Cr.
- Department in brackets, e.g. `4000 [RMS]`.

## 6. Worked double-entry posting examples (20)

### Ex.1 — Room night with tax and levy (EV-01)

Stay night 2026-10-05, corporate negotiated rate OMR 50.000 net.

| Line | Account | Dr | Cr |
|---|---|---:|---:|
| 1 | 1100 Guest ledger (folio F-1001) | 56.000 | |
| 2 | 4020 Room revenue – corporate [RMS] | | 50.000 |
| 3 | 2300.TEST.TT Output tax payable | | 5.000 |
| 4 | 2310.TEST.LV Levy payable | | 1.000 |
| | **Check** | **56.000** | **56.000** |

Stored per line: `rule_pack=RP-TEST-GENERIC-v1`, `tax_code`, `taxable_base=50.000`, `business_date=2026-10-05`, `source=room_charge:RC-…`, `event_id`.

### Ex.2 — Package allocation, room + breakfast (EV-02)

Package "Bed & Breakfast" OMR 60.000 net per night. Allocation version `ALLOC-PKG-BB-v3` uses relative standalone selling prices (room 55.000, breakfast 8.000; Σ 63.000).

| Component | SSP | Allocated = 60 × SSP / 63 |
|---|---:|---:|
| Room | 55.000 | 52.381 |
| Breakfast | 8.000 | 7.619 |
| **Total** | 63.000 | 60.000 |

| Line | Account | Dr | Cr |
|---|---|---:|---:|
| 1 | 1100 Guest ledger | 66.000 | |
| 2 | 4000 Room revenue [RMS] | | 52.381 |
| 3 | 4100 Food revenue [FNB-REST] | | 7.619 |
| 4 | 2300.TEST.TT (10 % of 60.000; components share one TEST-ONLY rate) | | 6.000 |
| | **Check** | **66.000** | **66.000** |

Rounding: last component absorbs the residue (52.381 + 7.619 = 60.000). If components carry different tax codes, the tax engine computes per component. Levy on the room component would add a line when the rule pack applies it (omitted here: TEST-ONLY pack exempts packages — illustrative only).

### Ex.3 — Deposit, pre-authorization, capture, PSP fee and settlement (EV-07/08/11/12/14)

| Step | Event | Account | Dr | Cr |
|---|---|---|---:|---:|
| a | Deposit via PSP pay-by-link | 1040 PSP clearing | 30.000 | |
| | | 2100 Advance deposits | | 30.000 |
| b | Pre-auth OMR 100.000 at check-in | *memo only — no GL* | — | — |
| c | Deposit applied at check-in | 2100 Advance deposits | 30.000 | |
| | | 1100 Guest ledger | | 30.000 |
| d | Stay charges (Ex.1) posted | (see Ex.1: 1100 Dr 56.000) | | |
| e | Capture balance at checkout (26.000; pre-auth remainder released) | 1040 PSP clearing | 26.000 | |
| | | 1100 Guest ledger | | 26.000 |
| f | PSP settlement: gross 56.000, fee 1.120 (fee from PSP statement, not computed) | 1020 Bank | 54.880 | |
| | | 6230 PSP fees [AGN] | 1.120 | |
| | | 1040 PSP clearing | | 56.000 |
| | **Check (all steps)** | | **142.000** | **142.000** |

Folio F-1001 closes at zero (56 − 30 − 26). 1040 clears to zero after step f. A settlement line that does not match a capture posts to 2500 and opens a reconciliation item.

### Ex.4 — Refund reverses the charge and the points exactly once (EV-24/EV-50/EV-15/EV-54) — Section G.7

Dinner charged to room: net 40.000, TT 4.000. MetriStay Rewards earns 10 pts per OMR 1 net = 400 pts. Points valuation version `PTS-VAL-2026-Q4` = 0.005 OMR/pt (standalone selling price × expected redemption; deferred-revenue model D-805) → 2.000 allocated to points.

| Step | Event | Account | Dr | Cr |
|---|---|---|---:|---:|
| a | Dinner posted to folio | 1100 Guest ledger | 44.000 | |
| | | 4100 Food revenue [FNB-REST] | | 40.000 |
| | | 2300.TEST.TT | | 4.000 |
| b | Points earned (pending, 400 pts) | 4100 Food revenue [FNB-REST] | 2.000 | |
| | | 2130 Points liability | | 2.000 |
| c | Paid by card at checkout | 1040 PSP clearing | 44.000 | |
| | | 1100 Guest ledger | | 44.000 |
| d | Complaint upheld → charge reversal (`FolioChargeReversed`, reason `SR-QUALITY`, approval `APR-…`) | 4100 Food revenue [FNB-REST] | 40.000 | |
| | | 2300.TEST.TT | 4.000 | |
| | | 1100 Guest ledger | | 44.000 |
| e | Linked points reversal (`PointsReversed`, causation = reversal id, −400 pts) | 2130 Points liability | 2.000 | |
| | | 4100 Food revenue [FNB-REST] | | 2.000 |
| f | Refund to card (`PaymentRefunded`, refund id `RF-77`) | 1100 Guest ledger | 44.000 | |
| | | 1040 PSP clearing | | 44.000 |
| g | **Duplicate PSP refund webhook** (same `RF-77`, new delivery id) | *inbox dedup — no posting, no second points reversal* | — | — |
| h | Operator retries refund from UI with same idempotency key | *returns original result — no posting* | — | — |
| | **Check (a–f)** | | **180.000** | **180.000** |

Net effect after f: 4100 = 0, 2300 = 0, 2130 = 0, 1100 = 0, 1040 = 0 (the refund is netted in the next PSP settlement). Points ledger shows +400 pending then −400 reversed, available balance unchanged. **Edge rule D-813:** if the 400 pts had already been redeemed, the reversal still posts once; the account goes negative in `points_debt` status, future earn offsets it first, and no cash is demanded from the guest unless the program terms (legal-reviewed) say so.

### Ex.5 — Points redemption and expiry (EV-52/EV-53)

Guest holds 1,000 available pts (carried at 5.000). Lunch net 20.000, TT 2.000 (TEST-ONLY assumption: tax on the full price; the real treatment of redemption is a rule-pack decision).

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Sale | 1100 Guest ledger | 22.000 | |
| | 4100 Food revenue [FNB-REST] | | 20.000 |
| | 2300.TEST.TT | | 2.000 |
| b Redeem 1,000 pts as tender | 2130 Points liability | 5.000 | |
| | 1100 Guest ledger | | 5.000 |
| c Card for balance | 1040 PSP clearing | 17.000 | |
| | 1100 Guest ledger | | 17.000 |
| d Expiry job: 600 pts of another member expire (3.000) | 2130 Points liability | 3.000 | |
| | 4710 Points breakage [MIS] | | 3.000 |
| **Check** | | **47.000** | **47.000** |

No points are earned on the redeemed portion (campaign rule `EARN-EXCL-REDEEM`). If the program estimates breakage up front (D-805 variant), step d posts only the true-up versus the estimate.

### Ex.6 — Section E referral: OMR 100 / 70 / 30 → commission OMR 6 (EV-60/61/63, EV-62/64)

Booking `B-5501` referred by direct referrer **A** (code `A-7Q2`), property in a jurisdiction whose referral gate is **open for test** (Oman payout requires the documented legal/tax opinion — gate `GATE-REF-OM` must be `open` for step c to execute; in acceptance this runs in sandbox only). Contract formula version `RCF-v1`, cost version `CPOR-STD-2026-10` (frozen at calculation).

**Margin calculation (stored as `referral_commission_calc` with every component):**

| Component | Source | OMR |
|---|---|---:|
| Net collected room revenue (excl. tax, after discounts, after any partial refund) | Folio + PSP settlement | 100.000 |
| − PSP fee actually charged | PSP settlement line | 2.000 |
| − Rooms variable labor (standard cost per occupied room-night × nights) | CPOR-STD version | 22.000 |
| − Laundry & linen (standard) | CPOR-STD version | 8.000 |
| − Guest supplies/amenities (standard) | CPOR-STD version | 5.000 |
| − Utilities per occupied room (standard) | UPOR-STD version | 10.000 |
| − Distribution/booking-engine cost (contract-defined) | Contract schedule | 8.000 |
| − Maintenance allowance (contract-defined) | Contract schedule | 5.000 |
| − A&G contract allowance | Contract schedule | 10.000 |
| **= Attributable costs** | | **70.000** |
| **Eligible margin = 100 − 70** | | **30.000** |
| Commission rate (business decision, contract `RA-A-2026`) | | 20 % |
| **Commission = max(0, 30.000) × 20 %** | | **6.000** |

*The cost components are illustrative; only costs listed in the signed contract version may be deducted.*

| Step | Event | Account | Dr | Cr |
|---|---|---|---:|---:|
| a | After checkout + cleared payment + refund window (qualification `QUAL-OK`) → accrue | 6210 Referral commission [SMK] | 6.000 | |
| | | 2140 Referral commission accrual (pending) | | 6.000 |
| b | Finance + compliance approval; tax/withholding: rule pack `unverified` → **no withholding line, payout blocked until verified**; in the test pack WHT = 0 | 2140 | 6.000 | |
| | | 2145 Referral commission payable | | 6.000 |
| c | Payout confirmed by authorized bank/PSP (`payment_ref PAY-…`); gate re-checked at release and at confirmation | 2145 | 6.000 | |
| | | 1020 Bank | | 6.000 |
| | **Check** | | **18.000** | **18.000** |

**Ex.6b — clawback after payout.** Guest raises a chargeback 20 days after payout and it is lost: `ReferralClawbackRaised` → Dr 1118 Referrer receivable 6.000 / Cr 6210 [SMK] 6.000. Offset against A's next approved commission (Dr 2145 / Cr 1118) or collected per contract. Posted once per `commission_id`.

**Ex.6c — reversal before approval.** Full refund before approval → Dr 2140 6.000 / Cr 6210 6.000; partial refund → recalculated with same `RCF-v1`/cost version, delta posted.

**Ex.6d — zero/negative margin.** Net revenue 60.000, attributable costs 70.000 → eligible margin −10.000 → commission `max(0, −10) × 20 % = 0`. No journal; statement line shows "not eligible — margin ≤ 0" with components visible to A only in privacy-preserving form (booking id, stay month, revenue band not guest identity).

**Controls asserted:** only one referrer per booking (`referral_attribution` unique on `booking_id`); no table relates referrers to referrers; C's booking referred by guest G (who was A's referee) earns **G** a commission only if G is itself an enrolled direct referrer and C used G's code — A earns zero from it.

### Ex.7 — Gift voucher issue, partial redemption, breakage (EV-38/39/40)

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Voucher `GV-3001` sold for 50.000 (multi-purpose voucher; TEST-ONLY pack: no tax at issue) | 1040 PSP clearing | 50.000 | |
| | 2120 Gift voucher liability | | 50.000 |
| b Dinner net 30.000 + TT 3.000 | POS tender clearing / 1100 | 33.000 | |
| | 4100 Food revenue [FNB-REST] | | 30.000 |
| | 2300.TEST.TT | | 3.000 |
| c Voucher tender 30.000 | 2120 | 30.000 | |
| | POS tender clearing / 1100 | | 30.000 |
| d Card 3.000 | 1040 | 3.000 | |
| | POS tender clearing / 1100 | | 3.000 |
| e Voucher expires with 20.000 unused; rule pack flag `unclaimed_property=false` | 2120 | 20.000 | |
| | 4700 Voucher breakage [MIS] | | 20.000 |
| **Check** | | **136.000** | **136.000** |

If the rule pack flags unclaimed-property remittance, step e credits `2040 Escheat payable` instead and 4700 is not used.

### Ex.8 — Bar POS: service charge, tip, void, COGS (EV-24/25/101)

Service charge policy D-806 = **pass-through to staff pool at 10 % (TEST-ONLY policy value)**; TT applies to drinks + service charge in the TEST-ONLY pack.

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Item "cocktail 4.000" voided before check close (bartender, reason `WRONG-ITEM`) | *no GL; void log & SF60.2.2 outlier stats* | — | — |
| b Check closed: drinks 20.000, SC 2.000, TT 2.200, tip 3.000 → card 27.200 | 1040 PSP clearing | 27.200 | |
| | 4110 Beverage revenue [BAR] | | 20.000 |
| | 2155 Service charge distribution payable | | 2.000 |
| | 2300.TEST.TT | | 2.200 |
| | 2150 Tips payable | | 3.000 |
| c Bar par replenishment issued from store (cost-at-issue model D-815) | 5010 Cost of beverage [BAR] | 5.600 | |
| | 1210 Inventory – beverage | | 5.600 |
| d Tips and SC distributed via payroll | 2150 | 3.000 | |
| | 2155 | 2.000 | |
| | 2200 Net salaries payable (tips/SC lines, withholding per rule pack) | | 5.000 |
| **Check** | | **37.800** | **37.800** |

Theoretical depletion from recipes (EV-107) for the 20.000 of drinks = 5.100 → theory-vs-actual variance 0.500 reported in beverage cost % analytics, not posted.

### Ex.9 — Club admission, deposit and minimum-spend shortfall (EV-29/32)

Table for 2 non-resident guests: deposit 100.000 paid at reservation; admission 10.000 each; minimum spend 200.000 on eligible beverage/food (admission excluded); actual spend 150.000.

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Deposit at reservation | 1040 | 100.000 | |
| | 2100 Advance deposits | | 100.000 |
| b Close tab: admission 20.000, spend 150.000, shortfall 50.000, TT 22.000; deposit applied, card 142.000 | 2100 Advance deposits | 100.000 | |
| | 1040 PSP clearing | 142.000 | |
| | 4300 Club admission [CLB] | | 20.000 |
| | 4330 Club beverage/food [CLB] | | 150.000 |
| | 4320 Minimum-spend shortfall [CLB] | | 50.000 |
| | 2300.TEST.TT (TEST-ONLY: shortfall taxable) | | 22.000 |
| **Check** | | **342.000** | **342.000** |

Admission is posted only after the capacity check succeeds (`ClubEntryGranted`); a denied entry produces no charge.

### Ex.10 — Electricity: estimate accrual → actual bill true-up → bill-pay pending/confirmed (EV-82/83/84/85/86)

September 2026. Main meter `FX-MTR-E-MAIN` intervals 97 % complete (one 20-hour gap interpolated). Estimated 40,000 kWh × tariff version `TAR-E-2026-01` placeholder 0.025 = 1,000.000.

| Step | Date / period | Account | Dr | Cr | Status |
|---|---|---|---:|---:|---|
| a Accrual | 30 Sep (P09) | 6300 Electricity [UTL] | 1,000.000 | | `incomplete-estimate` (gap) |
| | | 2030 Accrued utilities | | 1,000.000 | |
| b Bill arrives 12 Oct: 43,200 kWh, net 1,080.000, TT 108.000; P09 closed → true-up in P10 tagged `relates_to_period=P09` | 12 Oct (P10) | 2030 Accrued utilities | 1,000.000 | | |
| | | 6300 Electricity [UTL] (true-up) | 80.000 | | `reconciled` for P09 view |
| | | 1330.TEST Input tax recoverable | 108.000 | | |
| | | 2010 AP – utilities | | 1,188.000 | |
| c Bill-pay submitted via provider adapter; response timeout | 20 Oct | *no GL — order `BPO-9` pending, payment-in-flight lock; inquiry scheduled, no blind retry* | — | — | pending |
| d Inquiry returns `confirmed`, receipt `RCPT-…`; convenience fee 0.500 | 20 Oct | 2010 AP – utilities | 1,188.000 | | |
| | | 6240 Bill-provider fees [AGN] | 0.500 | | |
| | | 1050 Bill-provider clearing | | 1,188.500 | |
| e Provider settlement / bank debit matched | 21 Oct | 1050 Bill-provider clearing | 1,188.500 | | settled |
| | | 1020 Bank | | 1,188.500 | |
| | **Check** | | **4,565.000** | **4,565.000** | |

Variance check: bill kWh 43,200 vs metered 41,900 (incl. interpolation) = 3.1 % > tolerance 2 % (policy placeholder) → `UtilityBillVarianceFlagged`, engineering review before AP approval; approved with reason "meter gap + provider reading date differs". Allocation of the actual cost is shown in §7.3. If Khedmah/ONEIC is not contracted, steps c–e run through the approved bank-transfer path (EV-80) and the adapter is shown `blocked`.

### Ex.11 — Payroll with confidential detail, WPS rejection (EV-110/114/115)

Payroll run `PR-2026-09`, 5 employees, TEST-ONLY rates ESD 5 %, ERC 8 %.

**Restricted payroll sub-ledger** (visible only to `payroll_officer`, `payroll_approver`, `hr_officer` with purpose; encrypted separately; see §8):

| Emp (synthetic) | Cost center | Gross base | OT | ESD 5 % | Advance recovery | Net | ERC 8 % |
|---|---|---:|---:|---:|---:|---:|---:|
| E1 | CC-RMS-OPS | 600.000 | 0 | 30.000 | 0 | 570.000 | 48.000 |
| E2 | CC-RMS-OPS | 500.000 | 0 | 25.000 | 50.000 | 425.000 | 40.000 |
| E3 | CC-RMS-OPS | 400.000 | 100.000 | 25.000 | 0 | 475.000 | 40.000 |
| E4 | CC-FNB-OPS | 550.000 | 0 | 27.500 | 0 | 522.500 | 44.000 |
| E5 | CC-FNB-OPS | 450.000 | 0 | 22.500 | 0 | 427.500 | 36.000 |
| **Total** | | 2,500.000 | 100.000 | 130.000 | 50.000 | 2,420.000 | 208.000 |

**GL journal (what finance/GM can see — aggregated, no employee identifiers):**

| Line | Account | Dr | Cr |
|---|---|---:|---:|
| 1 | 6000 Salaries [RMS / CC-RMS-OPS] | 1,500.000 | |
| 2 | 6010 Overtime [RMS / CC-RMS-OPS] | 100.000 | |
| 3 | 6000 Salaries [FNB-REST / CC-FNB-OPS] | 1,000.000 | |
| 4 | 6020 Employer contributions [RMS] | 128.000 | |
| 5 | 6020 Employer contributions [FNB-REST] | 80.000 | |
| 6 | 2200 Net salaries payable | | 2,420.000 |
| 7 | 2210.TEST Employee statutory withholding payable | | 130.000 |
| 8 | 2220.TEST Employer contributions payable | | 208.000 |
| 9 | 1325 Staff advances receivable | | 50.000 |
| | **Check** | **2,808.000** | **2,808.000** |

**WPS/bank file** `WPS-2026-09-01` (format per selected bank, `docs/05`): bank rejects E4 (invalid account). Confirmed 1,897.500 → Dr 2200 1,897.500 / Cr 1030 1,897.500. 2200 keeps 522.500 open; case `PAY-REJ-…` to `payroll_officer`; corrected file `WPS-2026-09-02` confirmed → Dr 2200 522.500 / Cr 1030 522.500. GM report shows "Payroll Sep: approved 2,420.000 • paid 2,420.000 • 1 rejection resolved" without names. **F&B has 2 employees (< k = 3)**: the department labor total appears in the P&L (required), but average pay, headcount-cost splits and sub-cost-center breakdowns for F&B are suppressed in generic reports (§8.3).

### Ex.12 — Gas cylinder exchange with deposit (EV-88/89/90/92/93)

Opening custody for cylinder type `CYL-L` (placeholder size): 6 deposit-paid shells at the hotel (1310 = 6 × 20.000 = 120.000). Exchange delivery `ASN-C-44`: 4 empties returned, 4 full received, **plus** 1 additional new shell to raise par (deposit 20.000). Refill price 12.000 per cylinder. TT on refills only (TEST-ONLY; deposit tax treatment per rule pack).

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Receipt (GRN, 5 full) | 1250 Inventory – gas in cylinders | 60.000 | |
| | 1310 Cylinder deposits paid (1 new shell) | 20.000 | |
| | 2020 GRNI | | 80.000 |
| b Invoice matched (5 refills 60.000 + deposit 20.000 + TT 6.000) | 2020 GRNI | 80.000 | |
| | 1330.TEST Input tax | 6.000 | |
| | 2000 AP – trade | | 86.000 |
| c Issue: 3 connected in main kitchen, 1 in catering kitchen | 6150 Cylinder gas [FNB-KIT] | 36.000 | |
| | 6150 Cylinder gas [CAT] | 12.000 | |
| | 1250 | | 48.000 |
| d Month-end count finds 1 shell missing → deposit forfeited (D-803: POM) | 6160 Cylinder losses [POM] | 20.000 | |
| | 1310 Cylinder deposits paid | | 20.000 |
| **Check** | | **234.000** | **234.000** |

**Custody reconciliation:** shells 6 − 4 (empties out) + 4 (full in) + 1 (new) − 1 (lost) = 6 → 1310 = 120.000 ✓. Full-cylinder gas on hand 1 × 12.000 = 1250 balance 12.000 ✓. Remaining gas in connected cylinders is estimated by scale weight at stocktake (optional adjustment EV-105 style, flagged `estimate`).

### Ex.13 — 80-person corporate event composite bill (EV-33/34/35/08/06) — Section G.1–G.3

Corporate `FX-CORP-ACME`, 2-day event, contract `EVC-ACME-01` v3, guarantee 80 covers per lunch, hosted bar cap 500.000, deposit 1,000.000 paid at contract.

**Actual vs contracted:**

| Item | Contracted | Actual | Billed basis | Net OMR |
|---|---|---|---|---:|
| Meeting room classroom, 2 days @ 300 | 2 days | 2 days | contract | 600.000 |
| Lunch @ 8.000 | 80 + 80 | 80 + 76 | max(guarantee, actual) → 160 covers | 1,280.000 |
| Hosted bar | cap 500.000 | 450.000 consumed (POS, event tab) | actual ≤ cap | 450.000 |
| Club visit admission @ 5.000 | 80 | 78 entered | contracted per-head package | 400.000 |
| Parking 20 passes × 2 days @ 2.000 | 40 pass-days | 36 used (LPR) | passes billed | 80.000 |
| AV package | 1 | 1 | contract | 150.000 |
| Rooms 10 × 2 nights @ 45.000 | 20 RN | 20 RN | block pickup 100 % | 900.000 |
| **Net total** | | | | **3,860.000** |

TT 10 % on 3,860.000 = 386.000; LV 2 % on rooms 900.000 = 18.000 (TEST-ONLY).

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Deposit at contract | 1040 / 1020 | 1,000.000 | |
| | 2105 Advance deposits – events | | 1,000.000 |
| b Charges to event master folio `F-EV-ACME` | 1100 Guest ledger (master) | 4,264.000 | |
| | 4420 Venue hire [CAT] | | 600.000 |
| | 4400 Catering food [CAT] | | 1,280.000 |
| | 4410 Catering beverage – hosted bar [CAT] | | 450.000 |
| | 4300 Club admission [CLB] | | 400.000 |
| | 4510 Parking passes [PRK] | | 80.000 |
| | 4430 AV [CAT] | | 150.000 |
| | 4010 Room revenue – group [RMS] | | 900.000 |
| | 2300.TEST.TT | | 386.000 |
| | 2310.TEST.LV | | 18.000 |
| c Deposit applied | 2105 | 1,000.000 | |
| | 1100 | | 1,000.000 |
| d Balance to city ledger (credit approved, PO `ACME-PO-88`) | 1110 City ledger [ACME] | 3,264.000 | |
| | 1100 | | 3,264.000 |
| **Check** | | **9,528.000** | **9,528.000** |

Attendees' personal incidentals route to individual folios by routing rules (EV-05). Catering ingredients issued to the event carry `event_id=EV-ACME` (Ex.15) so event contribution = event revenue − event COGS − attributed event labor (SF27.2.5) − direct costs. Post-event change order `CO-3` (extra coffee break 76 × 1.500 = 114.000 net) posts via EV-35 only with the organizer's approval evidence.

### Ex.14 — Supplier invoice: GRNI, short delivery, price variance, 3-way match (EV-71/74/77)

PO `PO-VEG-120` (v2): 120 KG vegetables @ 0.400 = 48.000. Delivery: 120 KG scanned; 5 KG damaged → quarantine; 115 KG accepted. Invoice first arrives for 120 KG @ 0.410.

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Accepted 115 KG | 1200 Inventory – food [store MAIN] | 46.000 | |
| | 2020 GRNI | | 46.000 |
| b Quarantined 5 KG | 1260 Inventory – quarantine | 2.000 | |
| | 2020 GRNI | | 2.000 |
| c Quarantine rejected, returned to vendor | 2020 GRNI | 2.000 | |
| | 1260 | | 2.000 |
| d Invoice 120 KG → **match fails** (qty) → hold; supplier sends corrected invoice 115 KG @ 0.410 = 47.150 + TT 4.715. Price variance 1.150 (2.5 %) > tolerance 2 % → `procurement_approver` approves variance | 2020 GRNI | 46.000 | |
| | 5090 Purchase price variance [FNB-KIT] | 1.150 | |
| | 1330.TEST Input tax | 4.715 | |
| | 2000 AP – trade | | 51.865 |
| e Same invoice re-submitted by e-mail OCR (same supplier + invoice no. + amount) | *duplicate key → rejected, no posting* | — | — |
| **Check** | | **101.865** | **101.865** |

### Ex.15 — Store issue to an event, intact return, waste, shrinkage (EV-101/102/103/105)

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Issue 30 KG vegetables (lot `L-7781`) to BEO `EV-ACME` | 5000 Cost of food [CAT, event EV-ACME] | 12.000 | |
| | 1200 Inventory – food | | 12.000 |
| b Intact return 3 KG, inspector-approved, same lot/expiry | 1200 | 1.200 | |
| | 5000 [CAT, EV-ACME] | | 1.200 |
| c 2 KG trimmed/spoiled in kitchen after issue — reclass for waste % | 6700 Waste [CAT] | 0.800 | |
| | 5000 [CAT, EV-ACME] | | 0.800 |
| d 1.5 KG spoiled in store (photo, reason `EXPIRED`), moved to waste bin | 6700 Waste [FNB-KIT store] | 0.600 | |
| | 1200 | | 0.600 |
| e Blind count: book 50.5 KG, counted 49.5 KG → shrinkage 1 KG (approved) | 6710 Shrinkage [FNB-KIT] | 0.400 | |
| | 1200 | | 0.400 |
| **Check** | | **15.000** | **15.000** |

Wasted quantities (c, d) are recorded in the waste ledger with lot; they **never** return to available stock (P.3). A later "return" of the 2 KG is rejected by the stock service.

### Ex.16 — Chargeback lost, with linked points and referral effects (EV-16/18)

Room folio 56.000 settled earlier (Ex.3).

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Chargeback notice | 1120 Chargebacks receivable | 56.000 | |
| | 1040 PSP clearing | | 56.000 |
| b Chargeback fee (PSP statement) | 6230 PSP fees [AGN] | 5.000 | |
| | 1040 | | 5.000 |
| c Evidence submitted; dispute lost | 6250 Chargeback losses [AGN] | 56.000 | |
| | 1120 | | 56.000 |
| **Check** | | **117.000** | **117.000** |

Linked, same causation id: points earned on the stay reversed once (EV-54); referral commission reversed (EV-62) or clawed back (EV-64, Ex.6b).

### Ex.17 — Foreign-currency corporate invoice (EV-120/121/122)

Invoice USD 1,000.00 at rate `FX-USD-OMR-2026-10-01` 0.38500 → 385.000 OMR (illustrative rates).

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Invoice | 1110 City ledger (USD 1,000.00) | 385.000 | |
| | 4420 Venue hire [CAT] | | 385.000 |
| b Month-end revaluation at 0.38450 (auto-reverse 1st of next period) | 7610 Unrealized FX | 0.500 | |
| | 1110 | | 0.500 |
| c Reversal of b | 1110 | 0.500 | |
| | 7610 | | 0.500 |
| d Receipt USD 1,000.00 at 0.38480 | 1020 Bank (USD account, OMR equivalent) | 384.800 | |
| | 7600 Realized FX | 0.200 | |
| | 1110 | | 385.000 |
| **Check** | | **771.000** | **771.000** |

Tax lines omitted for clarity (TEST-ONLY: venue hire to a non-resident treated per rule pack; not decided here).

### Ex.18 — Night audit: no-show fee and LPR parking posted once (EV-19/37)

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a No-show, guaranteed booking, 1 night fee | 1100 (no-show folio) | 55.000 | |
| | 4050 No-show revenue [RMS] | | 50.000 |
| | 2300.TEST.TT (TEST-ONLY: taxable) | | 5.000 |
| b Captured against guarantee token | 1040 | 55.000 | |
| | 1100 | | 55.000 |
| c LPR session `PS-221` (2 days @ 2.000) at exit | 1100 (in-house guest folio) | 4.400 | |
| | 4500 Parking revenue [PRK] | | 4.000 |
| | 2300.TEST.TT | | 0.400 |
| d Duplicate exit observation from second camera (same plate, 4 s later) | *session already closed — no posting* | — | — |
| **Check** | | **114.400** | **114.400** |

### Ex.19 — Maintenance service with materials, and emergency chef (EV-73/74/80/117)

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Chiller repair accepted (WO `WO-512`, photos, engineer sign-off); not capitalized (below threshold) | 6400 Contractor services [POM, asset AS-CH-01] | 250.000 | |
| | 6410 Materials [POM, asset AS-CH-01] | 80.000 | |
| | 2020 GRNI | | 330.000 |
| b Invoice matched | 2020 | 330.000 | |
| | 1330.TEST | 33.000 | |
| | 2000 AP | | 363.000 |
| c Payment confirmed by bank (approved by `finance_approver`, released by `payment_releaser`) | 2000 | 363.000 | |
| | 1020 | | 363.000 |
| d Emergency chef shift accepted (M47 callout `CO-CHEF-9`) | 6050 External labor [CAT] | 60.000 | |
| | 2020 GRNI | | 60.000 |
| **Check** | | **1,116.000** | **1,116.000** |

### Ex.20 — Event cancellation: partial deposit forfeiture and refund (EV-09/10)

Deposit 1,000.000 in 2105; cancellation inside the 50 % penalty window.

| Step | Account | Dr | Cr |
|---|---|---:|---:|
| a Forfeit 50 % | 2105 | 500.000 | |
| | 4460 Attrition/cancellation fees [CAT] | | 500.000 |
| b Refund 50 % | 2105 | 500.000 | |
| | 1040 | | 500.000 |
| **Check** | | **1,000.000** | **1,000.000** |

No tax line: in `RP-TEST-GENERIC-v1` cancellation compensation is out of scope — **placeholder only**; the verified rule pack decides.

---

## 7. Shared-cost allocation (SF19.1, SF32.3.1, F22.2, F23.1, F24.1, F27.2)

### 7.1 Model

- **Allocation run** `ALLOC-<pool>-<period>-v<n>`: pool (source cost), driver, driver source, driver values per receiving department, method, `evidence_status` of the pool cost **and** of the driver, approver, created_at, supersedes.
- **States:** `draft → estimate → actual → superseded`. A run is `actual` only when the pool cost is `reconciled` (bill/payroll/invoice posted) **and** the driver data coverage ≥ the configured threshold (default assumption 95 %; D-816). Otherwise it is `estimate` or `incomplete-estimate` (§11).
- **Default posting policy (D-809):** allocations are computed in the **management reporting layer** (departmental P&L, profit bridge); USALI-style undistributed accounts remain unchanged in the GL. Optional policy: post allocation journals to 6900/6901 in the open period only.
- **Restatement:** when an `actual` run supersedes an `estimate` for a closed period, the management view restates that period (versioned; old version retained, reproducible, SF65.2.2) and the variance is shown; GL-posted allocations adjust in the current open period.

### 7.2 Drivers

| Pool | Primary driver | Source | Fallback when source missing | Default evidence |
|---|---|---|---|---|
| Electricity | Sub-meter kWh per department | Sub-meters/BMS (M22) | Engineering load schedule (connected kW × hours) version | actual if sub-meters ≥ 95 % coverage; else estimate |
| Water & sewage | Sub-meter m³ | Sub-meters (M23) | Occupied room-nights (rooms), covers (F&B), laundry kg | estimate unless metered |
| Pipeline gas | Gas sub-meter or burner-hours | Meter (M24) / kitchen logs | Covers produced by kitchen, laundry kg | estimate |
| Cylinder gas (undistributed policy) | Cylinders connected per outlet | Cylinder ledger (M25) | Covers | actual (custody-based) |
| Labor (shared staff, e.g. stewarding, security) | Scheduled or punched hours per department | Roster/time (M27 F27.2) | Rostered hours | actual when punches approved |
| Event labor attribution | Hours clocked against event code | Time capture with event tag | BEO labor plan | actual/estimate |
| A&G (for event/outlet contribution views only) | Revenue share or transaction count (policy) | GL + ops stats | — | policy-based (always labelled "allocated") |
| IT/ITC | Users/devices per department | Device registry (M64) | Headcount | policy-based |
| Laundry (in-house) | Kg processed per department | Laundry logs (M56) | Linen issues | actual/estimate |

### 7.3 Example: allocation of the September electricity bill (Ex.10)

Run `ALLOC-ELEC-2026-09-v2` (actual; supersedes v1 estimate based on 1,000.000 accrual). Pool 1,080.000.

| Receiving dept | Driver kWh | Share | v1 estimate | v2 actual | Δ |
|---|---:|---:|---:|---:|---:|
| RMS (guest floors) | 23,760 | 55 % | 550.000 | 594.000 | +44.000 |
| FNB-KIT (kitchen) | 6,480 | 15 % | 150.000 | 162.000 | +12.000 |
| RMS-LDY (laundry) | 4,320 | 10 % | 100.000 | 108.000 | +8.000 |
| CLB | 3,456 | 8 % | 80.000 | 86.400 | +6.400 |
| PRK | 864 | 2 % | 20.000 | 21.600 | +1.600 |
| AGN/common areas | 4,320 | 10 % | 100.000 | 108.000 | +8.000 |
| **Total** | **43,200** | 100 % | **1,000.000** | **1,080.000** | **+80.000** |

Driver evidence: sub-meters RMS/FNB-KIT/CLB 100 % intervals; PRK and laundry sub-meters 96 %; common area = residual. Coverage ≥ 95 % → run `actual`. Utility per occupied room (KPI K-17) recomputes for September with a restatement note.

### 7.4 Versioning rules

1. A run never mutates; new versions supersede. 2. Driver definitions have effective dates and approver. 3. A run using a fallback driver is labelled `estimate` even if the pool cost is reconciled. 4. Missing pool cost for a period (e.g. no gas bill yet and no meter) → run is `incomplete-estimate` and every downstream KPI inherits that label. 5. The referral margin (Ex.6) uses *standard cost versions* frozen by contract, not live allocation runs — so later restatements do not change earned commissions.

---

## 8. Salary confidentiality model (F27.4, Section D "never expose individual salary")

### 8.1 Data separation

- Payroll sub-ledger (employee-level gross/deductions/net, bank details, SIN/ID numbers) lives in the `payroll` schema with a **separate encryption key** (envelope encryption, KMS key `kms/payroll`), separate backup encryption, separate export controls and retention (SF27.1.3, SF27.1.5).
- GL receives only **aggregated journals** by department × cost center × natural account (Ex.11). The `source_id` of a payroll journal points to the payroll run, not to employees.
- Canadian SIN and other national IDs: encrypted field, masked display (last 3 digits), access only with purpose and step-up auth (SF38.2.1).

### 8.2 Role access matrix

| Data | payroll_officer | payroll_approver | hr_officer | financial_controller | gm / owner | dept manager | employee | auditor |
|---|---|---|---|---|---|---|---|---|
| Own payslip | — | — | — | — | — | — | ✔ own only | — |
| Individual gross/net/deductions | ✔ | ✔ | ✔ (purpose-logged) | ✖ (aggregate) — break-glass with 2nd approver | ✖ | ✖ | own only | sample, time-boxed grant |
| Individual bank details / SIN | ✔ masked; unmask with step-up | masked | masked | ✖ | ✖ | ✖ | own masked | ✖ |
| Department labor totals (P&L) | ✔ | ✔ | ✔ | ✔ | ✔ | own dept | ✖ | ✔ |
| Labor cost %, hours, overtime hours | ✔ | ✔ | ✔ | ✔ | ✔ | own dept | ✖ | ✔ |
| Per-employee hours/attendance (non-pay) | ✔ | ✔ | ✔ | ✖ | ✖ | own team | own | ✖ |
| Payroll GL journal | ✔ | ✔ | ✖ | ✔ | ✔ (read) | ✖ | ✖ | ✔ |
| WPS/bank file | create | approve | ✖ | status only | status only | ✖ | ✖ | metadata |

### 8.3 Inference protection

- **Minimum cell size k = 3 (assumption D-817):** any generic report cell derived from pay (average pay, cost per head, sub-cost-center labor, event labor with < 3 people) is suppressed or rolled up to the next level ("combined"). Department P&L totals are always shown because statutory/management P&L requires them; the UI states "contains fewer than 3 employees".
- **Differencing guard:** consecutive-period or filter-combination queries that would isolate one employee (e.g. total minus total-excluding-one) are blocked in pivots/custom reports (SF32.4.8) by applying the same k rule to every filter intersection.
- **Exports:** payroll-derived columns excluded from generic exports; payroll exports require payroll role, watermark with requester and time, and are logged.
- **Emergency chef / external labor** is AP spend (vendor invoice), visible to procurement/finance per contract — it is not salary.
- **Audit:** every access to individual pay emits `ConfidentialRecordViewed` (who, why, which record); monthly review by `hr_officer` + `financial_controller` of access log.

---

## 9. Profit bridge and result definitions (F32.3, SF32.3.2)

| Level | Definition | Includes | Excludes |
|---|---|---|---|
| **Total operating revenue** | Σ revenue accounts 4000–4790 net of allowances (4090/4190) | Rooms, F&B outlets, bar, club, catering, parking, other operated, misc income | Taxes/levies collected (liabilities), tips and pass-through service charge, deposits not yet earned, voucher/points liabilities, travel supplier pass-through (agent model) |
| **Departmental expenses** | Cost of sales (5xxx) + payroll & related (6000–6050) + other direct expenses charged to operated departments | Direct costs only, including channel commission if D-802 = Rooms | Undistributed and non-operating costs |
| **Departmental profit** (per operated dept) | Dept revenue − dept expenses | | Allocated shared costs (shown separately as "after allocation" view) |
| **Total departmental profit** | Σ departmental profit | | |
| **Undistributed operating expenses** | AGN + SMK + POM + UTL + ITC + HRM | PSP fees, referral commission, loyalty cost (cost model), utilities, maintenance, IT | |
| **Gross operating profit (GOP)** | Total departmental profit − undistributed operating expenses | | |
| **Management/brand fees** | 7000/7010 | | |
| **Income before non-operating income and expenses** | GOP − management/brand fees | | |
| **EBITDA-like operating result** ("Property operating result") | GOP − management/brand fees − non-operating fixed charges excluding depreciation, interest and income tax (rent 7100, insurance 7200, property taxes 7300) | | Depreciation, interest, income tax, FX, disposals |
| **Net income** | Operating result − depreciation/amortization (7400) − interest (7500) ± FX (7600/7610) ± disposals (7800) − income tax (7700) | | |
| **Allocated view (management only)** | Departmental profit after allocation of UTL, shared labor, and optionally AGN/ITC using §7 runs | Allocation version per line | Never replaces GAAP/statutory view |

**Bridge presentation (GM flash and month-end):** Revenue → − Cost of sales → − Payroll (aggregated) → − Other direct → **Departmental profit** by dept → − Undistributed (with UTL split into electricity/water/pipeline gas/cylinders and drill to meter/bill) → **GOP** → − Fees → − Fixed charges → **Operating result** → − D&A − Interest ± FX − Tax → **Net income**. Each bar carries `evidence_status` and coverage (§11); clicking drills to journal lines → source events → documents (G.8, G.19).

"EBITDA-like" is used deliberately: the platform does not assert conformance to any statutory EBITDA definition; the owner report states the formula used.

---

## 10. Reconciliation dashboard: planned / accrued / invoiced / approved / paid / settled (Section D)

One row per payable/receivable object; the dashboard aggregates by cost category (electricity, water, pipeline gas, cylinders, salaries, maintenance, other suppliers, referral commissions, bill-provider orders, guest/corporate collections).

| Column | Meaning | Source of truth | GL relation |
|---|---|---|---|
| **Planned** | Budget, PO commitment, payroll forecast, contracted event revenue | Budget/PO/encumbrance (EV-70), event contract | No GL (commitment) |
| **Accrued** | Recognized expense without invoice: GRNI, utility accrual, payroll accrual, referral pending | 2020, 2030, 2040, 2140 | GL |
| **Invoiced** | Supplier/provider invoice captured & matched (or AR invoice issued) | AP/AR sub-ledger | 2000/2010/1110 |
| **Approved** | Payment approved (maker-checker), not yet released | Payment proposal | No GL change |
| **Paid** | Released and confirmed by bank/provider/PSP (receipt) | Payment confirmation | 2000 → 1020/1050 |
| **Settled** | Matched to bank/provider/PSP settlement statement; reconciliation closed | Reconciliation item | 1050/1040 → 1020 cleared |

**Exception columns:** pending-unknown (timeout, awaiting inquiry), rejected, disputed, partially paid, duplicate blocked, variance over tolerance, aged > N days per stage (N configurable). **Invariants shown as tiles:** Σ confirmed payments = Σ bank debits matched; no object in *paid* without receipt reference; no *settled* without statement line; 2500 suspense ageing; 2990 = 0. **Drill:** row → document → evidence (meter reading, GRN, service photos, payroll run summary, provider receipt).

---

## 11. Evidence status and the incomplete-source rule (G.8, P.3, SF32.3.4)

| Status | Rule |
|---|---|
| `planned` | Budget/forecast/commitment; no actual event yet. |
| `estimate` | Actual event occurred; amount from a model, standard cost, meter estimate, fallback driver, or unverified rule pack; all required sources present. |
| `incomplete-estimate` | **At least one required cost source for the scope is missing** (e.g. no gas bill and no gas meter data for the period; payroll run not approved; channel commission statement missing; a meter gap > threshold; unmapped events in 2990; sub-ledger out of balance). |
| `reconciled` | All sources present, posted, and matched to external evidence (bill, bank, PSP, count); period may still be open. |
| `certified` | `reconciled` **and** period closed **and** financial controller sign-off **and** (for tax/filing values) rule pack `counsel-reviewed`+ with submission receipt. |

**Rule (non-negotiable):** a KPI, P&L line, bridge bar or report total **inherits the weakest status of any required input**. If any required cost source is missing, the value is displayed as **"incomplete estimate"** with the list of missing sources, owners and due dates — **never "certified actual"**, and never silently zero-filled. Required sources per scope are configured in `source_coverage_rule` (e.g. UTL for a month requires: electricity bill or ≥95 % metered kWh with tariff; water bill or meter; pipeline gas bill or meter; cylinder custody count). Exports carry the status column; PDF reports print a banner. Acceptance: `AT-G08.3` in `docs/09`.

---
## 12. KPI dictionary (M32, M65 SF65.1.2, P.6)

Conventions: *available rooms* = physical rooms × nights **minus** rooms out-of-order (OOO) for the whole night (OOS rooms remain available; policy D-818 lets a property exclude long-term OOO per its management-company standard — the rule used is printed on the report). *Occupied rooms* = room-nights sold and occupied or charged, including day-use counted by policy (D-819, default: excluded from occupancy, reported separately), complimentary rooms counted in occupancy **but** excluded from ADR revenue denominator only if flagged `comp` (both variants shown). House-use rooms excluded from both. Targets are **agreed with the pilot hotel** (P.6) and are not set here. Every KPI shows `evidence_status` (§11), period, business-date cutoff and last refresh time.

| ID | KPI | Formula | Numerator: includes / excludes | Denominator: includes / excludes | Source | Freshness | Owner |
|---|---|---|---|---|---|---|---|
| K-01 | Occupancy % | occupied room-nights ÷ available room-nights | Incl: paying + comp occupied; no-show not occupied. Excl: house use, day-use (reported separately) | Incl: all physical rooms × nights. Excl: OOO full-night rooms | Room inventory (M03), stays (M05), night audit | Intraday provisional; final at night audit | revenue_manager |
| K-02 | ADR | room revenue ÷ paid occupied room-nights | Incl: 4000–4040 net of allowances 4090 and package allocations to other depts. Excl: tax/levy, no-show revenue 4050, F&B in packages | Incl: paid occupied room-nights. Excl: comp, house use | GL 40xx + stays | Night audit | revenue_manager |
| K-03 | RevPAR | room revenue ÷ available room-nights (= Occ × ADR) | As K-02 | As K-01 | GL + inventory | Night audit | revenue_manager |
| K-04 | TRevPAR | total operating revenue ÷ available room-nights | Incl: all operated dept revenue + MIS. Excl: taxes, tips, pass-through service charge, deposits, voucher/points liabilities, travel pass-through | As K-01 | GL | Night audit (provisional), month-end (final) | gm |
| K-05 | GOPPAR | GOP ÷ available room-nights | GOP per §9 | As K-01 | GL + allocation | Month-end; daily flash as `estimate` | financial_controller |
| K-06 | Net RevPAR | (room revenue − channel/OTA commissions − PSP/transaction fees attributable to rooms − loyalty/referral cost attributable to room bookings) ÷ available room-nights | Incl: room revenue per K-02 numerator. Deductions: 6200 [RMS/SMK room bookings], 6230 share by payment, 6210, 6220/points allocation. Excl: fixed marketing | As K-01 | GL + channel statements + PSP | Daily `estimate`; month-end `reconciled` when commission statements match | revenue_manager |
| K-07 | CPOR (cost per occupied room) | Rooms dept expenses ÷ occupied room-nights | Incl: RMS payroll (aggregated), laundry, linen, guest supplies, cleaning, RMS allocated utilities (allocated view only). Excl: undistributed unless "allocated view" | Occupied room-nights incl. comp | GL + allocation + stays | Month-end (daily estimate) | front_office_manager + housekeeping_supervisor |
| K-08 | Labor cost % | payroll & related (6000–6050) ÷ total operating revenue (or dept revenue for dept view) | Incl: gross, OT, employer contributions, benefits, external/emergency labor 6050 (shown split). Excl: tips/SC pass-through | Revenue per K-04 numerator or dept revenue | Payroll GL journals (aggregate), GL revenue | Per payroll run + daily accrual estimate | financial_controller (dept managers read own) |
| K-09 | Food cost % | cost of food sold ÷ food revenue | Incl: 5000 net of intact returns, staff meal reclass out, PPV 5090 (policy). Excl: waste 6700 (reported separately), comps reclassified | Incl: 4100, 4120, 4400, food share of packages & 4330. Excl: tax, SC | Stock ledger + GL | Daily (cost-at-issue model) / at stocktake (periodic model) | executive_chef + fnb_manager |
| K-10 | Beverage cost % | cost of beverage sold ÷ beverage revenue | Incl: 5010. Excl: waste, comps | Incl: 4110, 4410, beverage share of 4330 | Stock ledger + GL + POS | Daily / stocktake | fnb_manager |
| K-11 | Utility cost per occupied room | (electricity + water + pipeline gas + cylinder gas cost) ÷ occupied room-nights; also kWh/m³/gas units per occupied room | Incl: 6300–6330 + 6150 (policy). Excl: taxes recoverable | Occupied room-nights | Utility bills, accruals, meters (M22–M25), allocation §7 | Daily from meters (`estimate`), month-end with bills | chief_engineer |
| K-12 | OTIF (vendor on-time-in-full) | PO lines delivered within delivery window **and** accepted qty ≥ ordered qty (within tolerance) ÷ PO lines due | Incl: accepted qty after quarantine decisions. Excl: lines cancelled by hotel before dispatch | PO lines with due date in period | PO/ASN/GRN (M49/M50) | Real-time | procurement_officer |
| K-13 | Fill rate | Σ accepted qty ÷ Σ ordered qty (by value and by units) | Incl: accepted; substitutions only if approved as equivalent. Excl: rejected/quarantined-then-rejected | Ordered qty on PO lines due | GRN, PO | Real-time | procurement_officer |
| K-14 | Average stock age (days) | Σ(qty on hand × days since receipt) ÷ Σ qty on hand, per store/category; plus % stock beyond shelf-life threshold | Incl: available + reserved. Excl: quarantine/waste bins (reported separately) | — | Lot-level stock ledger | Real-time | storekeeper |
| K-15 | Waste % | waste & spoilage cost (6700 + 6720) ÷ (food + beverage cost issued) — also by weight | Incl: approved waste transactions, recall write-offs. Excl: intact returns, shrinkage (separate K-15b) | Incl: 5000 + 5010 + waste | Waste ledger, stock ledger | Daily | executive_chef |
| K-15b | Shrinkage % | stocktake variance loss (6710) ÷ inventory issued value | Unexplained count variance | Issued value | Stocktake | Per count | storekeeper |
| K-16 | DSO & AR aging | DSO = (AR balance 1110+1112+1115 ÷ credit revenue last 90 days) × 90; aging buckets 0–30/31–60/61–90/>90 by invoice date | Incl: open invoices, unallocated credits shown separately. Excl: guest ledger in-house | Credit sales (direct bill) | AR sub-ledger | Daily | ar_clerk / financial_controller |
| K-17 | Payroll error rate | payslips requiring correction after approval (off-cycle adjustments, rejected for data error) ÷ payslips issued | Incl: corrections due to hotel data errors, calculation errors. Excl: bank-side rejections not caused by hotel data (reported separately as K-17b) | Payslips in run | Payroll runs, correction log | Per run | payroll_approver (aggregate only) |
| K-18 | Quote-to-book conversion | paid bookings ÷ priced quotes presented (unique session/guest per stay date), by channel | Incl: web, AI assistant, front desk, corporate quotes. Excl: bot-flagged sessions (SF51.2.6), internal test traffic | Quotes with price shown | Booking engine, M51 attribution | Hourly | marketing_manager |
| K-19 | Direct net acquisition cost | (direct marketing spend + metasearch/ads + booking-engine fees + PSP fees on direct bookings + referral commissions + loyalty cost of direct bookings) ÷ direct paid room-nights (or ÷ direct room revenue as %) | Incl: 6600/6610 attributed, 6210, points cost allocation. Excl: brand-wide fixed marketing not attributable (shown separately) | Direct paid stays completed (not cancelled) | GL, attribution, referral ledger | Weekly `estimate`; month-end `reconciled` | marketing_manager |
| K-20 | Complaint-to-closure time | median and P90 of (case closed-confirmed-by-guest time − case opened time) | Incl: all guest complaints (all channels). Excl: duplicates merged; reopened cases measured to final closure | Closed cases in period | Service recovery cases (M55) | Real-time | guest_relations |
| K-21 | Housekeeping turnaround | median/P90 of (room inspected-ready time − departure/checkout time) for departure rooms; also stayover service time | Incl: departures cleaned same business day. Excl: DND time, OOO rooms, rooms waiting for maintenance (reported separately) | Departure rooms cleaned | Room board (M06/M56) | Real-time | housekeeping_supervisor |
| K-22 | Incident acknowledgment time | median/P90 of (first accountable human acknowledgment − incident creation/alert ingestion) by severity | Incl: confirmed and false-alarm incidents (split). Excl: test drills (reported separately) | Incidents in period | Incident log (M42) | Real-time | security_officer / duty_manager |
| K-23 | Platform uptime | (minutes in period − minutes of user-facing unavailability of critical journeys) ÷ minutes in period, per journey (booking, check-in, folio/payment, POS, gate) | Incl: unplanned outages; degraded-mode minutes counted separately. Excl: announced maintenance windows (reported separately) | Minutes in period | Synthetic probes + OTel (M01/M64) | 1-minute | it_admin |
| K-24 | Department contribution | dept revenue − dept direct costs (and after-allocation variant) | per §9 | — | GL + allocation | Daily flash `estimate`, month-end | gm |
| K-25 | Points liability & breakage rate | outstanding points × valuation; breakage = expired ÷ (redeemed + expired) | Excl: pending points in liability view shown separately | — | Points ledger | Daily | referral_program_admin / financial_controller |
| K-26 | Referral cost per referred stay | Σ 6210 ÷ referred completed stays; clawback rate | Excl: reversed | Referred stays completed | Referral ledger | Weekly | referral_program_admin |
| K-27 | Payment dispute rate | chargebacks opened ÷ card captures (count and value) | | Captures | PSP | Daily | financial_controller |
| K-28 | Recovery time (DR drills) | measured RPO / RTO per drill | | | DR drill evidence (`docs/09` §9) | Per drill | it_admin |

KPI definitions are versioned (`kpi_definition` with version/effective date/approver); any change restates history only in a new report version.

---

## 13. Night audit checklist (M08, SF60.1.4, J night-audit state machine)

Run by `night_auditor` via `night_audit_worker`; each step records pass/fail/override with reason. Business date cannot roll while a **blocking** step fails unless `duty_manager` overrides with reason (logged, flagged on the GM flash).

| # | Step | Blocking | Evidence |
|---|---|---|---|
| NA-01 | Confirm all cashier shifts for the business date closed; cash over/short posted and approved above tolerance | Yes | Shift reports |
| NA-02 | Arrivals not checked in: mark no-show per policy; guaranteed no-show fees calculated (not yet captured if policy requires review) | Yes | No-show list |
| NA-03 | Departures not checked out: extend, late-checkout charge or flag | Yes | Due-out list |
| NA-04 | Room status reconciliation: front desk occupied vs housekeeping status discrepancies (sleepers/skips) resolved or flagged | No (flag) | Discrepancy report |
| NA-05 | POS outlets closed; open checks transferred to folio or carried with approval; offline POS queues synced (no pending offline transactions) | Yes | Outlet close report, sync status |
| NA-06 | Parking sessions: in-house permits posted; open sessions with unmatched plates listed for review | No (flag) | LPR exception list |
| NA-07 | Club and event tabs posted for the date; event master folios reviewed | Yes | Tab report |
| NA-08 | Post room & tax (EV-01), packages (EV-02), fixed charges; verify count = occupied rooms | Yes | Room-and-tax report |
| NA-09 | Rate check: rate overrides vs rate plan, zero-rate rooms, comps with approval | No (flag) | Rate variance report |
| NA-10 | Payment/PSP: pre-auths near expiry, declined captures, pending refunds, webhook dead letters | No (flag) | PSP exception queue |
| NA-11 | Guest ledger control: Σ open folios = GL 1100; advance deposits sub-ledger = 2100/2105 | Yes | Control-total report |
| NA-12 | Credit limit exceptions on folios and city ledger | No (flag) | Credit report |
| NA-13 | Duplicate detection: duplicate charges/payments/refunds (SF60.2.1) | No (flag, case) | Duplicate report |
| NA-14 | Unmapped posting suspense 2990 = 0 for the date | Yes | Mapping report |
| NA-15 | Produce daily statistics: occupancy, ADR, RevPAR, TRevPAR, dept revenue (K-01–K-04, K-24) with status | Yes | Daily flash |
| NA-16 | Roll business date; lock postings for the closed date; publish `BusinessDateClosed` | — | Audit log |
| NA-17 | Backup checkpoint verified (on-prem profile) | No (alert) | Backup job id |

Reopen of a closed business date: `financial_controller` + reason; only reversing/adjusting postings allowed; reports re-versioned.

## 14. Month-end close checklist (F19.1.5, SF32.3.4)

| # | Step | Owner | Blocking for `certified` |
|---|---|---|---|
| MC-01 | All business dates of the period night-audited | night_auditor | Yes |
| MC-02 | AR: invoices issued for all city-ledger transfers; receipts allocated; aging reviewed; allowance updated | ar_clerk | Yes |
| MC-03 | AP: invoices captured; 3-way match exceptions resolved or accrued; duplicates reviewed | ap_clerk | Yes |
| MC-04 | GRNI (2020) line-by-line review; stale GRNI > 60 days investigated | ap_clerk | Yes |
| MC-05 | Utilities: bills imported or accrued per meter; variance flags resolved; `source_coverage_rule` evaluated | chief_engineer + finance_clerk | Yes |
| MC-06 | Cylinder custody count; deposit reconciliation 1310 = shells × deposit | storekeeper | Yes |
| MC-07 | Stocktake (blind count) for all stores; shrinkage approved; quarantine bin reviewed; waste approvals complete | storekeeper + executive_chef | Yes |
| MC-08 | Payroll: all runs approved; accrual for unpaid days; provisions; 2200/2240 open items explained; statutory liabilities reconcile to payroll reports | payroll_officer | Yes |
| MC-09 | PSP: all settlements matched; 1040 residual explained; chargebacks updated | finance_clerk | Yes |
| MC-10 | Bank reconciliation for every account; 2500 suspense aged and explained | finance_clerk | Yes |
| MC-11 | Bill-provider orders: no `pending-unknown`; 1050 cleared or explained | finance_clerk | Yes |
| MC-12 | Points liability reconciles to points ledger × valuation; breakage run | financial_controller | Yes |
| MC-13 | Voucher liability reconciles; breakage per D-810 | financial_controller | Yes |
| MC-14 | Referral: pending accruals reviewed; gate status per jurisdiction; no payout while gate closed | referral_program_admin + compliance_officer | Yes |
| MC-15 | Channel/OTA commission statements matched (SF60.2.3) | revenue_manager | Yes (for net RevPAR `reconciled`) |
| MC-16 | Prepayments amortized; accruals posted; depreciation run | financial_controller | Yes |
| MC-17 | FX revaluation of open monetary items | financial_controller | Yes |
| MC-18 | Tax: output/input tax reports per jurisdiction; rule-pack status checked; filing export or manual handoff; remittance scheduled | compliance_officer | Yes for tax values |
| MC-19 | Allocation runs executed; estimate vs actual status reviewed | financial_controller | No (management view) |
| MC-20 | Sub-ledger control totals = GL for all control accounts; 2990 = 0; trial balance balanced per legal entity | financial_controller | Yes |
| MC-21 | Flux analysis vs budget/prior period; commentary | financial_controller | No |
| MC-22 | Lock period; produce owner/management pack; evidence status per line | financial_controller | — |

## 15. Reporting catalogue (F32.4) and role access

Legend: ✔ full • D own department only • A aggregate only (no individual pay/guest PII) • — none. All reports: PDF/CSV/XLSX export where permitted, scheduled delivery with recipient consent (SF65.2.4), point-in-time version (SF65.2.2), evidence status column.

| SF | Report | owner/gm | fin. controller | revenue/sales | dept managers | payroll/HR | auditor | Notes |
|---|---|---|---|---|---|---|---|---|
| SF32.4.1 | **GM daily flash**: occupancy, ADR, RevPAR, TRevPAR, dept revenue & contribution (estimate), cash position, AP due 7 days, payroll summary (A), utilities today vs baseline, open exceptions/SLAs | ✔ | ✔ | ✔ | D | A | ✔ | Shown on owner/GM home (Section F). |
| SF32.4.2 | **Finance & cash**: trial balance, P&L (dept & USALI-style), balance sheet, cash-flow (indirect), cash forecast, AP/AR aging, reconciliation dashboard (§10), close status | ✔ | ✔ | — | — | — | ✔ | |
| SF32.4.3 | **Occupancy & revenue**: pace/pickup, segment/channel mix, net RevPAR, cancellations/no-shows, corporate block pickup, forecast vs actual | ✔ | ✔ | ✔ | Rooms D | — | ✔ | |
| SF32.4.4 | **Utilities/energy/water/gas**: consumption vs bill, estimate vs actual, per occupied room, anomalies, allocation runs, cylinder custody & deposit, sustainability intensity (M67) | ✔ | ✔ | — | engineering ✔; others D (allocated share) | — | ✔ | |
| SF32.4.5 | **Purchasing/payroll/vendor**: spend by category/vendor, RFQ competition & exceptions, OTIF/fill rate, GRNI, price variance; payroll cost by dept (A), overtime hours, payroll error rate; vendor scorecards | ✔ (A for payroll) | ✔ (A) | — | D | ✔ (individual in payroll module only) | ✔ (A) | Salary rules §8. |
| SF32.4.6 | **Corporate/events/outlets**: event P&L actual vs contracted, BEO change orders, corporate account profitability & AR, outlet P&L, food/beverage cost %, waste %, club capacity & minimum spend, parking utilization | ✔ | ✔ | ✔ (corporate/events) | D | — | ✔ | |
| SF32.4.7 | **Audit & data quality**: journal audit trail, manual journals, reversals, overrides (rates, voids, comps, gate), duplicates, unmapped events, stale feeds, meter gaps, source coverage, confidential-access log (restricted), rule-pack status | ✔ | ✔ | — | — | access log: HR ✔ | ✔ | |
| SF32.4.8 | **Authorized custom pivots & scheduled exports**: semantic layer with row/field security, k-suppression, watermarking | per role | per role | per role | D | payroll data in payroll workspace only | read | No raw PAN, no ID images, no individual pay outside payroll roles. |

Additional regulated outputs (per jurisdiction rule pack, M38): tax invoices/credit notes, tax return exports, payroll statutory reports (e.g. Canadian T4 artifacts), WPS file — produced in their modules with receipts, referenced from SF32.4.2/SF32.4.7.

## 16. Decisions, assumptions and traceability

| Id | Decision / assumption | Default in this document | Owner |
|---|---|---|---|
| D-801 | Bar as own department vs F&B outlet | `BAR` own dept for acceptance hotel | financial_controller |
| D-802 | Channel commission: Rooms vs S&M | Rooms (direct cost) | financial_controller |
| D-803 | Cylinder gas & losses: outlet vs UTL/POM | Outlet consumption; losses POM | financial_controller + chief_engineer |
| D-804 | Linen in circulation | Expensed on issue to circulation | financial_controller |
| D-805 | Loyalty accounting model | Deferred revenue (relative SSP) — requires auditor confirmation per legal entity | financial_controller |
| D-806 | Tips & service charge treatment | Pass-through liabilities; per rule pack & contracts | financial_controller + hr_officer + counsel |
| D-807 | Minibar department | FNB-MINI | financial_controller |
| D-808 | Event attrition fees | CAT | financial_controller |
| D-809 | Allocation GL-posted vs management layer | Management layer | financial_controller |
| D-810 | Voucher breakage method | At expiry, subject to unclaimed-property rule | financial_controller + counsel |
| D-811 | Package allocation method | Relative SSP | revenue_manager + financial_controller |
| D-812 | Comps: discount vs cost reclass | Cost reclass for comps, contra for discounts | fnb_manager |
| D-813 | Points reversal after redemption | Negative `points_debt`, offset future earn | referral_program_admin + counsel |
| D-814 | Payments in transit account | Not used unless bank debit lags | financial_controller |
| D-815 | Inventory COGS model | Cost at issue | financial_controller |
| D-816 | Driver coverage threshold for `actual` | 95 % | financial_controller |
| D-817 | Salary suppression k | 3 | hr_officer + dpo |
| D-818 | Long-term OOO in availability | Excluded (reported) | revenue_manager |
| D-819 | Day-use in occupancy | Excluded, reported separately | revenue_manager |

**Traceability:** F19.1/F19.2 → §1–§4, §13–§14; F20.1–F20.3 → §4.1, §4.4, §10; F22.2/F23.1/F24.1/F25.1 → §4.4, Ex.10, Ex.12, §7; F27.3/F27.4 → §4.6, Ex.11, §8; F28.1/F28.2 → Ex.3, Ex.4, Ex.16; F29.1 → Ex.10; F30.1 → Ex.4, Ex.5; F31.1/F31.2 → Ex.6; F32.1–F32.4 → §9, §11, §12, §15; F54.2 → Ex.7; F60.1/F60.2 → §13; Section D → §10; Section G.8 → §11, `AT-G08.*`. Acceptance tests for this document: `AT-G07.*`, `AT-G08.*`, `AT-REF.*`, `AT-G05.*`, `AT-G04.*` in `docs/09`.
