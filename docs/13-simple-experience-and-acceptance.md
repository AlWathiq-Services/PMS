# 13 — Simple Experience, Traceability and Planning Gate

**Pack:** MetriStay Hospitality Suite Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Status:** specification / design target. **Phase 2 may not start until the gate in §7 is signed.**
**Generated companions:** `13a-decision-register.generated.md` (all decisions), `13b-subfeature-index.generated.md` (all 1,150 subfeatures). Regenerate with `python3 tools/build_registers.py`.

---

## 1. Simple experience rules (Section P.1)

These rules bind every screen in `docs/04` / `docs/04a` and every Phase 2+ slice.

| # | Rule | Where specified | Test |
|---|---|---|---|
| UX-1 | **Answer five questions on every work item:** what needs attention, why, by when, who owns it, what can I do — and after acting, *did it work?* (explicit success/failed/pending state, never silent). | `docs/04` §2 shared behaviour codes; M63 work items | AT-A11Y.*, AT-G19.* |
| UX-2 | **Role home = 3–7 priority tasks + one unified inbox** filtered to the role; everything else by search or progressive disclosure. | `docs/04` role homes (≈40 roles) | Usability test per persona (`docs/09` pilot criteria) |
| UX-3 | **Hotel profile drives navigation.** Limited/full service profile; only outlets, integrations and regulations the hotel actually uses are enabled; disabled modules disappear from menus, API scopes, website and reports (M01 feature flags, M44 activation gates, M58 "hidden when not owned"). | M01.F01.x, M44.F44.2.SF44.2.6, M58.F58.1.SF58.1.1 | AT-G09.*, AC-SF58.1.1 |
| UX-4 | **One authoritative status per object** (from its owning module's state machine in `docs/02`); other modules display it, never re-derive it. | `docs/02`, `docs/03` §4 SoR table | Consistency review §4.5 |
| UX-5 | **Source and freshness on every KPI** (`estimate` / `reconciled` / `certified`, as-of time, coverage). Missing cost source ⇒ "incomplete estimate". | `docs/06` §11, M32, M65 | AT-G08.3 |
| UX-6 | **Honest external status.** A partner/government/device capability that is not certified shows its honesty label and the manual path; never a green tick for a mock. | README §3.6, `docs/05` | AT-G06.*, AT-G12.*, `docs/09` REAL-RB cases |
| UX-7 | **Bilingual and accessible by default:** English/Arabic with full RTL; WCAG 2.2 AA; keyboard, screen reader, error correction, CAPTCHA-free alternatives; jurisdiction language packs per `docs/07`. | `docs/04` RTL + WCAG checklists | AT-A11Y.1–10 |
| UX-8 | **Low-bandwidth guest path and assisted channel:** guest booking usable on low-bandwidth mobile and through front-desk-assisted booking with the same rules and totals. | `docs/04` low-bandwidth budget; `docs/11` | AT-G19.* |
| UX-9 | **Offline-safe staff workflows** that queue actions but never confirm a sale, capture a payment or mark a delivery without server acknowledgment. | `docs/03` ADR-008 | AT-OFF.1–10 |
| UX-10 | **Owner drill-down:** any KPI drills to the source ledger/event with the same numbers (point-in-time reproducible). | M32.F32.3.SF32.3.5, M65.F65.2 | AT-G08.2 |

---

## 2. Planning pack contents (Section H + P.7)

| Required output | File | Status v0.1 |
|---|---|---|
| H1 executive scope | `00-executive-scope.md` | drafted |
| H2 module catalogue | `01-module-catalogue.md` + `01-catalogue/01..06` | drafted; checker passes (§4.1) |
| H3 journeys and state machines | `02-journeys-and-states.md` (58 SM, 21 journeys) | drafted |
| H4 architecture | `03-architecture.md` (21 ADRs, ERDs) | drafted |
| H5 screens and design | `04-screens-and-design.md` (578 screens) + `04a-screen-crosswalk.md` | drafted; crosswalk see §4.3 |
| H6 integrations | `05-integrations.md` (34 contracts) | drafted |
| H7 finance and KPIs | `06-finance-kpis.md` (127 event mappings, 20 posting examples, 28 KPIs) | drafted |
| H8 security and regulatory | `07-security-regulatory.md` (61-row five-market register) | drafted; no row counsel-reviewed |
| H9 phase backlog | `08-phase-backlog.md` (109 WPs) | drafted |
| H10 acceptance and migration | `09-acceptance-and-migration.md` (266 test cases, fixtures) | drafted |
| N competitive gap and questions | `10-competitive-gap-and-questions.md` (86 question rows) | drafted |
| P.7 guest acquisition and service | `11-guest-acquisition-and-service.md` | drafted |
| P.7 operations, safety, continuity | `12-operations-safety-continuity.md` | drafted |
| P.7 simple experience and acceptance | this file | drafted |

---

## 3. Traceability index

### 3.1 Every user request → module → feature → phase → test (Sections A and M)

Each item in the master prompt's mission (Section A) and the Section M "additional modules" list has an explicit link; none is folded into a generic "ERP" heading. Full subfeature-level detail: `13b-subfeature-index.generated.md`. Blueprint delta column is empty until D-001 supplies the blueprints.

| # | User request (Section A / M) | Modules → key features | First phase | Release | Acceptance |
|---|---|---|---|---|---|
| T-01 | PMS: rooms, rates, reservations, front desk, folio, night audit | M03, M04, M05, M06, M08 (F08.x night audit), M60.F60.1 | 2 | R1 | AT-G01, G03, G08, G20 |
| T-02 | Direct and channel distribution | M07, M51.F51.2, M53.F53.2.SF53.2.4 | 2–3 | R1 | AT-G19, G20 |
| T-03 | Corporate bookings, corporate web + signed Android/iOS apps | M10, M11 | 3 | R1 | AT-G01, G02 |
| T-04 | Function spaces, events/MICE, BEO | M09, M12 | 2–3 | R1 | AT-G01, G02 |
| T-05 | Catering | M16, M57.F57.2 | 3 | R1 | AT-G02, G03, G18 |
| T-06 | Bar / F&B POS | M13, M57.F57.1 | 3 | R1 | AT-G03 |
| T-07 | Club / membership | M15 | 3 | R1 | AT-G01, G03 |
| T-08 | Parking with AI camera or LPR server and gate | M17 | 3 | R1 | AT-G03 (REAL-RB hardware) |
| T-09 | Engineering / maintenance requisitions | M26, M21, M49 | 3–4 | R1 | AT-G04, G10 |
| T-10 | Electricity | M22 (F22.1–F22.4) | 4 | R1 | AT-G04, G08, G20 |
| T-11 | Water | M23.F23.1 | 4 | R1 | AT-G04 |
| T-12 | Pipeline gas | M24.F24.1–F24.2 | 4 | R1 | AT-G04 |
| T-13 | Gas cylinders | M25.F25.1–F25.2 | 4 | R1 | AT-G04 |
| T-14 | Purchasing, inventory across departments | M21, M14, M49, M50 | 3–4 | R1 | AT-G04, G17, G18 |
| T-15 | HR / salaries / payroll / WPS | M27 (F27.1–F27.5) | 3–4 | R1 | AT-G05 (WPS REAL-RB) |
| T-16 | Finance / AP / AR / GL | M19, M20 | 4 | R1 | AT-G04, G05, G08 |
| T-17 | Payment gateway | M28 | 2 and 5 | R1 | AT-G06, G07, G20 (one certified PSP REAL-RB) |
| T-18 | Bill-provider interfaces (Khedmah / ONEIC) | M29 (F29.1–F29.3) | 5 | R1 (adapter **blocked** until contract) | AT-G06 |
| T-19 | Guest loyalty — non-cash points wallet | M30 | 5 | R1 | AT-G07 |
| T-20 | Network-marketing legal gate → single-tier direct referral (MetriStay Network) | M31 (F31.1–F31.2), `docs/07` Oman gate | 5 build; 6 activation | R1 (Oman payout gated) | AT-G07.4–G07.12 |
| T-21 | Cash / stored-value wallet | M30.F30.2.SF30.2.6 | — | **Excluded** from R1 unless licensed PSP (D-039) | AT-G07 (negative test) |
| T-22 | Management profitability | M32, M65, M66, `docs/06` | 2 → 4–6 | R1 | AT-G08, G19 |
| T-23 | Hotel-initiated flight / cruise / taxi | M45 (F45.1–F45.3), M59 | 3 / 5 / 6 | R1 (market-gated) | AT-G11 |
| T-24 | Searchable service-provider registry for every department | M46 | 2–4 | R1 | AT-G10 |
| T-25 | Chef staffing continuity / emergency chefs | M47 | 2–4 | R1 | AT-G15 |
| T-26 | Vendor mobile marketplace / catalog / daily stock and rates | M48, M46.F46.3 | 3–4 | R1 | AT-G16 |
| T-27 | Source-to-pay: RFQ, 90-day samples, weighted award, PO | M49, M21 | 3–4 | R1 | AT-G17 |
| T-28 | AI delivery follow-up, low-touch receiving, issue/return/waste | M50, M14 | 3–4, 6 site acceptance | R1 | AT-G18 |
| T-29 | Web admin, guest web/mobile, staff mobile | `docs/04` apps OPS/GM/FO/…/GST, ADR-009/010 | 2–3 | R1 | AT-A11Y, AT-OFF |
| T-30 | Property image/video publishing + locally deployable AI enhancement | M39 | 2–3 | R1 | AT-G13 |
| T-31 | Guarded customer AI agent | M40 | 3–5 | R1 | AT-G13 |
| T-32 | Identity-assisted check-in, e-signature, SMS/WhatsApp verification | M41 | 2–3; 5 adapters | R1 | AT-G13 |
| T-33 | Incident response | M42 | 3–4 | R1 | AT-G14 |
| T-34 | Lost and found | M43 | 2–3 | R1 | AT-G14 |
| T-35 | Five-market jurisdiction classifier (CA, OM, PK, SA, PT) | M44, M38 | 2 → 4–6 | R1 | AT-G09 |
| T-36 | Canada: SIN, CPP/EI, QPP/QPIP, T4, GST/HST + provincial/municipal levies | M38.F38.1–F38.2, M27 | 4–6 | R1 (filing REAL-RB) | AT-G12 |
| T-37 | Government filing / API boundaries | M38.F38.3, `docs/05` INT-GOV-* | 4–6 | R1 (manual path until authorized) | AT-G09, G12 |
| T-38 | Booking website / marketing / CRM / reputation | M51, M52 | 2–5 | R1 | AT-G19 |
| T-39 | Revenue management | M53 | 3 / 5 / 7 | R1 (automation Later) | AT-G19 |
| T-40 | Upsells, vouchers, packages | M54 | 3–5 | R1 | AT-G19 |
| T-41 | Guest journey and service recovery | M55, M18 | 2–4 | R1 (key/kiosk Later) | AT-G19 |
| T-42 | Optional spa / pool / retail / golf | M58 | 3–7 | R1 when enabled | AC-SF58.* |
| T-43 | Laundry / linen / minibar | M56.F56.2 | 2–4 | R1 | AT-G19 |
| T-44 | Food safety / hygiene / inspections | M57.F57.2, M61 | 3–4 | R1 | AT-G18 |
| T-45 | Shift SOPs, staff learning | M62 | 3–4 | R1 | AC-SF62.* |
| T-46 | Cash controls / revenue protection | M60 | 2–5 | R1 | AT-G08, G20 |
| T-47 | Site devices, network, cyber, backup/restore | M64, M01 | 2–6 | R1 | AT-DR.*, AT-OFF.* |
| T-48 | Owner / brand / capex | M66 | 4–6 | R1 (portfolio Later) | AC-SF66.* |
| T-49 | Sustainability | M67 | 4–6 | R1 | AC-SF67.* |
| T-50 | Risk, insurance, business continuity | M68 | 4–6 | R1 | AT-DR.*, AC-SF68.* |
| T-51 | Workflow automation / service desk | M63 | 2–6 | R1 | used by all |
| T-52 | Integration developer platform | M33 | 2 → 5–7 | R1 (public portal Later) | AT-SEC.* |
| T-53 | Multi-property, UC/wake-up, HSIA, IPTV, smart locks | M34, M35, M36, M55.F55.3, M64 (Later) | 7 | Later | AT-G21–G24 |
| T-54 | Marketplace, OTA-like search, supplier payouts | M37 | 7–8 | Later | AT-G25 |

### 3.2 Section G integrated scenarios → journeys → tests

| Scenario | Journey (`docs/02`) | Primary modules | Tests (`docs/09`) | Real partner/hardware release blockers |
|---|---|---|---|---|
| G1 corporate search 80 attendees | J-01 | M09, M10, M11, M12, M04 | AT-G01.x | — |
| G2 composite booking, no double sale, BEO propagation | J-02 | M09, M12, M16, M13, M17 | AT-G02.x | — |
| G3 check-in, LPR gate, single posting, stock depletion, club capacity | J-03 | M05, M17, M08, M13, M14, M15 | AT-G03.x | LPR camera/server + gate |
| G4 gas cylinder & maintenance procurement; utility bills | J-04 | M21, M25, M26, M22–M24, M20 | AT-G04.x | — |
| G5 payroll and WPS | J-05 | M27, M19 | AT-G05.x | WPS bank channel |
| G6 gateway payments, bill-pay or alternate | J-06 | M28, M29 | AT-G06.x | Certified PSP; Khedmah/ONEIC if required |
| G7 points and referral | J-07 | M30, M31 | AT-G07.x | Oman legal/tax opinion for payout |
| G8 night audit, month-end, drill-through | J-08 | M08, M19, M32, M65 | AT-G08.x | — |
| G9 five-market classifier | J-09 | M44, M38 | AT-G09.x | Counsel per market for activation |
| G10 vendor self-registration and department search | J-10 | M46, M49, M26 | AT-G10.x | — |
| G11 taxi / flight / cruise | J-11 | M45, M59, M46 | AT-G11.x | Contracted providers / licensed seller |
| G12 Canadian hotel tax and payroll | J-12 | M38, M27, M44 | AT-G12.x | CRA-authorized route / payroll provider |
| G13 media, AI, ID OCR, e-sign, OTP | J-13 | M39, M40, M41 | AT-G13.x | SMS/WhatsApp, e-sign provider |
| G14 incident and lost-and-found | J-14 | M42, M43 | AT-G14.x | Approved BMS/camera interface |
| G15 chef and backup absent | J-15 | M47, M27, M12 | AT-G15.x | — |
| G16 vendor app catalogs | J-16 | M48, M46 | AT-G16.x | App-store developer accounts |
| G17 120 KG vegetables RFQ | J-17 | M49, M21, M48 | AT-G17.x | — |
| G18 AI follow-up, receiving, stores, recall | J-18 | M50, M14, M20 | AT-G18.x | Scales/scanners/probes (site pilot) |
| G19 website → stay → recovery → revenue | J-19, J-21 | M51–M55, M53, M56, M32 | AT-G19.x | Channel manager |
| G20 concurrency and failure injection | J-20 | all | AT-G20.x | — |

### 3.3 Section O owner questions

All 86 question rows (76 from Section O plus 10 discovered so that every module M01–M68 has one) are in `docs/10` §§ with the required columns *owner question → actor → needed action → source-of-truth → module → subfeature → phase → screen/API → exception → acceptance test → KPI*, and the role × {normal, busy, low staffing, outage, dispute, refund, assisted, audit} cross-cut table.

### 3.4 Blueprint delta

The two original hospitality blueprints were **not supplied** (D-001). When provided, every blueprint idea gets a row here: *blueprint § → T-row or module/SF → accepted / deferred (phase) / excluded (reason)*. Until then, the master prompt is the sole requirements source (A-001 in `docs/00`).

---

## 4. Consistency check (Section P.7: catalogue vs phase plan vs breakdown vs UI vs acceptance)

### 4.1 Automated catalogue checks — **PASS**

`python3 tools/check_catalogue.py`: 68/68 modules; 210 features; 1,150 subfeatures; all 18 Section-L fields present and non-empty; ids unique and well-formed; releases R1 (1,113) / Later (37); all 584 Section K/Q subfeature ids present with their original names.

### 4.2 Phase plan vs catalogue — **PASS with 3 recorded deviations**

Every module's earliest subfeature phase equals its Section C build phase, except these deliberate pull-forwards (Section C phases are "first implementation phase"; pulling a narrow enabler forward does not move the module's completion):

| Subfeature(s) | Section C | Catalogue | Reason |
|---|---|---|---|
| SF20.2.1 corporate credit account and PO | M20: 4 | 3 | G2 (Phase 3 exit) needs deposit/PO/credit approval before AR exists; implemented as M10 credit line, posted to AR in Phase 4. |
| SF27.1.1, SF27.2.1–SF27.2.3 employee contract/roster/time/leave | M27: 4 | 3 | M47 chef coverage and M62 shift SOPs are Phase 3 and need rosters, attendance and leave; payroll remains Phase 4. |
| SF29.3.2 manual/bank alternate path and blocked labelling | M29: 5 | 4 | Utility bills are approved and paid in Phase 4 (G4); the honest manual/bank path must exist before any provider adapter. |

Extensions after the Section C phase (e.g. M11 app parity in 5, M17 reconciliation in 5) are permitted and match `docs/08` work packages.

### 4.3 Catalogue screens vs `docs/04` — **resolved via crosswalk**

The catalogue references 1,194 distinct screen ids; 74 match `docs/04` directly and 1,120 were written with placeholder app prefixes before `docs/04` fixed its codes. `docs/04a-screen-crosswalk.md` maps each placeholder to a canonical `docs/04` screen (alias/tab) or defines a new canonical screen with the full `docs/04` column set. **Rule:** implementation uses canonical ids only; catalogue ids resolve through the crosswalk. Re-verify when the catalogue changes.

### 4.4 Acceptance ids — **PASS**

164 distinct `AT-Gnn.m` ids are referenced across `docs/00`–`13` and the catalogue; all 164 are defined in `docs/09`. Where writers gave the same id different meanings, `docs/09` §4 "Cross-document id reconciliation" records the authoritative meaning; the catalogue's own `AC-<SF id>` criteria remain valid unit-level acceptance.

### 4.5 Shared entity names — **canonical owner decided here**

The six catalogue files were written in parallel; where two files define the same concept, the canonical name and owner below win. Other modules reference the canonical entity (Section Q: no duplicate sources of truth).

| Concept | Names found | Canonical entity → owner | Notes |
|---|---|---|---|
| Guest master | `guest_profile` (M05; M18/M52 references) | `guest_profile` → **M05** | M52 enriches/merges (`guest_merge_decision`), M18 app reads. Resolves D-605 in favour of M05. |
| Purchase request | `requisition` (M49), `purchase_requisition` (M21, M14 refs) | `requisition` + `requisition_line` → **M49** | M21 owns policy (`procurement_policy`, thresholds, SoD) applied to M49 requisitions. |
| Stock item | `stock_item` (M14), `item` (M48/M50 refs) | `stock_item` → **M14** | M48 `item_crosswalk` maps vendor SKU → `stock_item`. |
| Stock ledger, lots, bins, waste, recall | M14 and M50 both list `waste_record`, `recall_case` | `stock_ledger_entry`, `stock_lot`, `store_bin`, `waste_record`, `recall_case` → **M14** | M50 runs receiving/issue/recall workflows and writes only through the M14 `StockLedger` port. |
| Approvals | `approval_request` (M02, M10, M63), `approval_policy` (M02, M10) | policy & delegation of authority → **M02**; runtime `approval_request`/`approval_decision` → **M63** workflow engine | M10 corporate buyer approvals use the same engine with corporate-realm principals. |
| Housekeeping | `hk_task`, `hk_inspection`, `minibar_count` (M06 and M56) | `hk_room_status` → **M06**; `hk_task`, `hk_inspection`, `minibar_count`, linen → **M56** | M06 screens present M56 tasks at the front desk. Resolves D-621. |
| Production batch | `production_batch` (M16, M57) | `production_batch` → **M57** | M16 catering creates batches via M57; lot links via M14. |
| Budgets | `budget_version` (M20, M32, M66 refs) | `budget_version`, `budget_line` → **M20** | M32 reads for budget vs actual. |
| Cost allocation | `allocation_policy`/`allocation_run` (M19), `allocation_policy_version`/`allocation_run` (M32) | `allocation_policy` (versioned), `allocation_run` → **M19** | M32 displays; estimate vs actual flags per `docs/06`. |
| Metric definitions | `kpi_definition` (M32), `metric_definition` (M65), `report_definition` (M32, M65) | `metric_definition`, `report_definition` → **M65** | M32 renders. Resolves D-651. |
| Capitalization | `capitalization_decision` (M21, M66) | → **M21** | M66 capex/investment case links to it; M19 posts. D-654 asset split stands. |
| Backup/restore | `backup_run`/`restore_test` (M01), `backup_job`/`restore_test` (M64) | → **M01** runtime records; **M64** owns DR drills, `recovery_objective`, `dr_failover_test` | |
| Waitlists | `waitlist_entry` (M05 rooms, M57 dining) | M05 keeps `waitlist_entry`; M57 renames to **`dining_waitlist_entry`** | Different concepts. |
| Dead letters | `dead_letter` (M01, M07), `dead_letter_item` (M33) | `dead_letter` → **M01** | M07/M33 are views/replay tooling over it. |
| Vendor performance | `vendor_scorecard` (M26), `vendor_performance_record` (M46) | `vendor_performance_record` → **M46** | M26 scorecard is a maintenance-filtered view. |
| Travel vs transport | `travel_request` (M45), `transport_trip` (M59) | both kept | M45 = external provider orders; M59 = hotel fleet/dispatch; a taxi order in M45 may create a linked `transport_trip` for tracking. Resolves D-633. |

These are **documentation-level** decisions; the catalogue files keep their original wording for traceability and are aligned during Phase 2 schema design (WP-2.1 in `docs/08`). The Phase 2 schema review must check every table name against this list.

### 4.6 Decision ids — **renumbered, no collisions**

Parallel writers reserved overlapping ranges. Final allocation (no id is defined in two files; verified by `tools/build_registers.py`):

| Range | Source | Note |
|---|---|---|
| D-001–D-042 | `docs/00` (Section J decisions) | Programme-level; **governing** for their subject |
| D-051–D-068 | `docs/05` | |
| D-071–D-078 | `docs/07` | |
| D-101–D-133 | `01-catalogue/01` | |
| D-201–D-241 | `01-catalogue/02` | |
| D-301–D-350 | `01-catalogue/03` | |
| D-401–D-450 | `01-catalogue/04` | |
| D-501–D-537 | `01-catalogue/05` | |
| D-601–D-661 | `01-catalogue/06` | |
| D-701–D-709, D-711–D-717, D-721–D-729 | `docs/10`, `11`, `12` | renumbered from D-101…/D-111…/D-121… |
| D-801–D-819 | `docs/06` | renumbered from D-601… |
| D-901–D-907 | `docs/09` | |
| D-911–D-916 | `docs/02` | renumbered from D-JS-01…06 |
| D-921–D-940 | `docs/03` | renumbered from D-301… |
| D-971–D-980 | `docs/04` | renumbered from D-401… |

Total: **427 decisions**, listed in `13a-decision-register.generated.md`.

### 4.7 Decision clusters (same subject decided at several levels)

The programme decision in `docs/00` governs; module decisions refine it and must not contradict it. Resolve the cluster together.

| Subject | Governing | Refinements |
|---|---|---|
| Blueprints supplied | D-001 | — |
| Meaning of "NIS" vs Canadian SIN | D-018 | D-074, D-426 (interim: no NIS field; typed national ids per market, e.g. `CA_SIN`, `PT_NISS`) |
| Sample-photo 90-day clock start | D-016 | D-520, D-911 (interim: RFQ close; award approval optional per property) |
| Minimum quotes and weights | D-017 | D-314, D-521, D-522, D-523, D-913 (interim: 3; 2 with waiver; 1 single-source with GM approval) |
| Travel operating model per market | D-011 | D-501, D-914 (interim: flights/cruises referral-only; taxi via contracted provider) |
| Oman referral payout | D-033 | D-406, D-915 |
| Channel manager | D-006 | D-109 |
| PSP / PCI | D-024 | D-342, D-343 |
| Khedmah / ONEIC | D-030 | D-346, D-347, D-348 |
| WPS bank / format | D-027 | D-333 |
| Payroll engine build vs provider (Canada) | D-022 | D-334, D-427 |
| Chart of accounts | D-040 | D-301 |
| Camera / LPR / gate | D-028 | D-234–D-238 |
| SMS / WhatsApp providers | D-036 | D-241, D-436, D-441 |
| ID/OCR, e-sign | D-037 | D-438, D-440 |
| Local AI models and GPU | D-038 | D-432, D-433, D-435, D-528, D-940 |
| App-store accounts | D-031 | D-212, D-516 |
| DND welfare-check threshold | D-128 | D-916 |
| Hold / quote / OTP timeouts | D-106 | D-107, D-201, D-912 |
| Retention periods | D-118 | D-340, D-437, D-504, D-524 |

---

## 5. Where proof from a provider or counsel is missing

Nothing in this pack is `counsel-reviewed`, `partner-contracted`, `sandbox-tested` or `certified`. The following are therefore **release blockers for any property that requires them** (full list `docs/08` B-01…B-28; legal register `docs/07`):

- **Legal/regulatory (all five markets):** tax and e-invoicing treatment, guest registration/police reporting, ID/biometric lawful basis, payroll/social security, travel intermediation licensing, marketing/referral rules, food safety and retention. The `docs/07` register has 61 rows; about 20 primary sources could not be retrieved on 2026-09-28 (e.g. CBO PSP policy PDF, the official MoJ listing of Decision 105/2021, Pakistani provincial revenue authorities, several Saudi and Portuguese authority pages) and are marked `unverified`.
- **Oman single-tier referral:** built and sandbox-tested in Phase 5; **payout activation** needs a documented legal and tax opinion on final terms, referrer categories and promotion channels. The platform never presents it as a network/pyramid scheme and contains no multi-level structure.
- **Cash/stored-value wallet:** not in Release 1; only via a licensed PSP/bank after legal advice (D-039).
- **Partners:** channel manager, PSP (one certified gateway), bank/WPS, Khedmah/ONEIC (candidates, API not established), LPR camera/gate, BMS/fire, SMS/WhatsApp, e-sign, ID/OCR, airline/cruise/taxi providers, app-store developer accounts, government filing routes per market (CRA certified software/file formats; ZATCA developer portal indicated but not validated).
- For each, `docs/05` specifies a simulator adapter and a manual operating path so independent work proceeds.

---

## 6. Top decisions needed before or early in Phase 2

| Id | Decision | Why it gates |
|---|---|---|
| D-001 | Supply the two blueprints | Completes requirements traceability (§3.4) |
| D-002 / D-041 | Pilot hotel: country/province/municipality, size, outlets; commitment | Drives jurisdiction rule packs, enabled modules, hardware and acceptance targets |
| D-010 | Five-market launch sequence | Orders counsel engagement and rule-pack verification |
| D-018 | Meaning of "NIS" | Payroll data model per market |
| D-024 | Bank and PSP | Longest certification lead time (Phase 5 gate) |
| D-006 | Channel manager | Phase 3 certification |
| D-034 | Deployment location and data residency | SaaS region vs on-prem profile |
| D-035 | Programme budget and blended rate | Converts `docs/00` person-months into cost |
| D-040 | Chart of accounts | Phase 2 finance export and Phase 4 GL |
| D-020 / D-021 | Tax, privacy and signature counsel per market | Every activation gate depends on it |

---

## 7. Planning gate checklist (Sections H, M, P.7)

| # | Check | Evidence | Status v0.1 |
|---|---|---|---|
| G-01 | All Section H and P.7 documents exist | §2 | **met** |
| G-02 | M01–M68 each with named feature and numbered subfeature | §4.1 checker | **met** |
| G-03 | Every subfeature has phase, release flag, actor/screen/data/API/events/failure/test, dependency, finance/report effect, i18n/a11y | §4.1 checker | **met** |
| G-04 | Section K/Q ids retained verbatim | §4.1 (584/584) | **met** |
| G-05 | Every user request has explicit traceability | §3.1 (T-01…T-54) | **met** (blueprints pending D-001) |
| G-06 | Every Section O scenario has a traceability row | `docs/10` (86 rows, 68/68 modules) | **met** |
| G-07 | Phase plan, catalogue, detailed breakdown consistent | §4.2 | **met** with 3 recorded deviations |
| G-08 | Catalogue vs UI companion consistent | §4.3 crosswalk | **met** when `docs/04a` verification passes |
| G-09 | Catalogue vs acceptance suite consistent | §4.4 | **met** |
| G-10 | Unresolved decisions have owner, interim assumption and fallback | `13a` (427) | **met** |
| G-11 | Partner/counsel proof gaps stated with gate and manual path | §5, `docs/05`, `docs/07`, `docs/08` | **met** |
| G-12 | Estimates, staffing, critical path, exclusions | `docs/00` §12–13, `docs/08` | **met** (low confidence; budget D-035 open) |
| G-13 | Shared entity names reconciled | §4.5 | **met** at documentation level; enforced in WP-2.1 schema review |
| G-14 | `docs/09` PG-01…PG-18 planning-gate verification | `docs/09` | to be run at sign-off |
| G-15 | **Human sign-off** of the pack by Product Owner, Architect, Finance lead, Compliance lead | — | **open** |

**Gate rule:** Phase 2 production coding begins after G-15. Per Section H, ordinary design choices do not need approval, and work that does not depend on a missing partner or counsel answer continues against simulators. The first executable slice is `docs/08` S2-00 (walking skeleton: tenancy, IAM, business date, outbox) followed by S2-01…S2-12.
