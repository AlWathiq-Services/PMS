# 09 — Acceptance Tests, Fixtures, Non-Functional Tests, Migration and Rollback

**Pack:** Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28 • **Master-prompt sections:** B (phase exits), E (referral non-negotiable tests), G (integrated demonstration 1–20), H.10, M (verification checklist), P (gates and invariants)

> This is a **test plan**, not a test report. No test here has been executed; no partner, hardware or government interface is claimed to exist. A test marked `REAL` cannot be passed with a simulator. Decision ids in this document use **D-901…D-999** (proposed; consolidated in `docs/13`).

---

## 1. Conventions

### 1.1 Test identifiers
- Integrated scenario tests: `AT-Gnn.m` — `nn` = Section G item 1–20, `m` = case.
- Other suites: `AT-REF.n` (Section E referral), `AT-PERF.n`, `AT-SEC.n`, `AT-PRIV.n`, `AT-DR.n`, `AT-OFF.n`, `AT-A11Y.n`, `AT-HW.n` (pilot hardware/vendor), `AT-MIG.n` (migration), `AT-RB.n` (release rollback).
- Unit/feature acceptance (`AC-<SF id>`) lives in `docs/01`; this document references SF ids for traceability.

### 1.2 Execution mode (column **Mode**)

| Mode | Meaning | Can satisfy Release 1 acceptance? |
|---|---|---|
| `SIM` | Provider-neutral simulator/mock adapter owned by us, deterministic, runs in CI | Only for workflows the property does **not** require from a real partner, and for all internal logic |
| `SBX` | Partner's official sandbox with our test credentials | Proves contract conformance; not production certification |
| `REAL` | Contracted partner production/certification environment or physical device at the pilot site | Yes |
| `…-RB` | **Release blocker per property**: if the property's configuration requires this real workflow, Release 1 cannot be declared for that property until the `REAL` run passes (P.5). If the property does not use it, the test is `N/A` with the reason recorded. | — |

Example: `SIM(3) → REAL-RB(6) [PSP]` = runnable on simulator from Phase 3; real PSP certification run in Phase 6 is a per-property release blocker.

### 1.3 Evidence standard
Every executed case stores an **evidence bundle** `EVB-<test id>-<run id>`: build/commit hash, environment/profile (`saas` / `onprem-single-hotel`), fixture version, UTC start/end, actor accounts used, API request/response logs (secrets redacted), emitted domain events (from outbox), journal entries (ids + lines), screenshots/screen recordings for UI steps (both EN and AR where UI is tested), device logs for hardware, partner reference numbers, and a signed pass/fail verdict by the named tester and reviewer (two different people for `REAL` runs). Bundles are immutable and retained for the life of Release 1 + 7 years (assumption D-901, subject to per-jurisdiction record rules).

### 1.4 Common pass rules (apply to every case unless stated)
1. No ledger invariant broken (P.3): Σ Dr = Σ Cr; no duplicate stock receipt/charge/refund/payout; `2990` suspense = 0 at test end (`docs/06` §1).
2. Every injected failure appears in a named exception queue with owner and SLA; nothing fails silently.
3. Authorization: all steps run with the least-privileged role named; the same step attempted with an unauthorized role is denied (sampled in each scenario, exhaustively in `AT-SEC`).
4. Audit log contains actor, time, before/after, reason for every privileged action.
5. Reports touched by the scenario show correct `evidence_status` (`docs/06` §11).

---

## 2. Test environments

| Env | Profile | Data | Partners | Purpose |
|---|---|---|---|---|
| `ci` | ephemeral containers | fixtures only | SIM | Every commit: unit, contract, `SIM` scenario subsets |
| `int` | saas (k8s) | fixtures | SIM + SBX | Nightly full G-suite on simulators/sandboxes |
| `onprem-lab` | onprem-single-hotel (Docker Compose/k3s on reference hardware) | fixtures | SIM + SBX; lab hardware (camera, gate controller, scale, scanner, probes, printer, terminal) | Offline/outage, device, DR drills |
| `perf` | saas, production-sized | generated volume fixtures | SIM with latency injection | `AT-PERF` |
| `pilot` | profile chosen for pilot hotel | migrated pilot data + fixtures in a sandbox property | REAL | `REAL` and `-RB` runs, dual-run |

---

## 3. Synthetic fixture catalogue

**Rules:** all people, companies, plates, IDs and bank accounts are synthetic; ID-document images are vendor-provided specimen images or generated test documents carrying a visible "SPECIMEN" mark, never real persons' documents. **Rule packs used by fixtures are placeholders labelled `TEST-ONLY, not legal values`**; they carry structure (tax codes, components, effective dates, filing modes) but their rates are arbitrary round numbers and must never be copied into a production pack. Production packs are created only through the M44 evidence workflow (`docs/07`). Fixture versions are semver'd (`fixtures@1.0.0`); a test records the version it used.

### 3.1 Reference hotel ("the G hotel") and organisation

| Fixture id | Description | Key attributes | Used by |
|---|---|---|---|
| `FX-TEN-01` | Tenant "MetriStay Test Tenant" | SaaS tenant; also exported as single-hotel on-prem bundle | all |
| `FX-HOTEL-G` | Reference single hotel "Hotel Gamma (TEST)" | Full-service, 60 rooms, time zone Asia/Muscat, business date independent of calendar, EN/AR, currency OMR (3 dp); jurisdiction fixture = Oman placeholder pack `RP-TEST-OM-v0` | AT-G01…G20 |
| `FX-LE-G-01` | Legal entity of FX-HOTEL-G | Functional currency OMR; chart of accounts from `docs/06` template | G08, finance |
| `FX-ROOMS-G` | Room inventory | 40 STD, 16 DLX, 4 ACC (accessible, 2 connecting pairs among STD); 2 rooms pre-set OOO for tests | G01–G03, G19, G20, PERF |
| `FX-RATES-G` | Rate plans | BAR, B&B package (`ALLOC-PKG-BB-v3`), corporate ACME/BETA negotiated, group block rate, upgrade offers | G01, G19 |
| `FX-SPACE-BALL` | Ballroom with 2 partitions | classroom 90 / theatre 160 / banquet 120; setup 2 h, teardown 1 h | G01, G02 |
| `FX-SPACE-MR2` | Meeting room 2 | classroom 60 (deliberately < 80 for infeasibility) | G01.2 |
| `FX-CORP-ACME` | Corporate customer (primary G scenario) | 2 branches, cost centers, corporate_admin/booker/approver users, credit limit 10,000.000, negotiated rate agreement v3, PO required | G01, G02, G06 |
| `FX-CORP-BETA` | Second corporate (Phase 3 exit "two corporations at different rates") | Different negotiated rate, no event entitlement | G01.3, SEC |
| `FX-CC-RMS-OPS` | Staff cost center 1 | Rooms operations (front office + housekeeping) | G05, G08 |
| `FX-CC-FNB-OPS` | Staff cost center 2 | F&B operations (kitchen, bar, club, catering staff) | G05, G08 |
| `FX-OUT-REST` | Restaurant outlet | POS menus, recipes, in-room dining | G03, G19 |
| `FX-OUT-BAR` | Bar outlet | Happy hour, tabs, service charge policy (TEST-ONLY 10 %), tips | G03, G20 |
| `FX-OUT-CLUB` | Club venue | Capacity 50, admission, member plan, minimum-spend tables, hosted bar | G01, G03 |
| `FX-OUT-CATK` | Catering kitchen | Production slots, BEO consumption, cylinder-connected burners | G02, G15, G17, G18 |
| `FX-STORE-MAIN` / `FX-STORE-BAR` | Stores with bins & lots | Food, beverage, housekeeping, spares; FEFO | G03, G18 |
| `FX-RECIPES` | Recipes/BOM/yield | Lunch menu for 80, cocktails, allergens | G03, G18 |
| `FX-PRK-01` | Parking facility | 80 spaces, zones guest/corporate/staff, 2 lanes (entry/exit), tariffs, 20 corporate passes | G01, G03 |
| `FX-PLATES` | Synthetic plate set | 60 plates incl. Latin/Arabic-script formats, look-alike characters, dirty/partial plate images | G03, HW |
| `FX-MTR-E-MAIN` | Electricity main meter | 15-min intervals; one injectable gap; tariff version `TAR-E-TEST` | G04, G08, G20 |
| `FX-MTR-E-SUB` | Electricity sub-meters RMS, KIT, LDY, CLB, PRK | For allocation `ALLOC-ELEC` | G08 |
| `FX-MTR-W-MAIN` | Water master meter | Supply + sewage components; leak pattern generator | G04 |
| `FX-MTR-G-PIPE` | Pipeline gas meter | Standing + variable tariff placeholders | G04 |
| `FX-CYL-L` | Gas cylinder type | Deposit 20.000 (placeholder), par 6, reorder point 3, 6 serialized shells | G04 |
| `FX-ASSET-SET` | Assets | Chiller `AS-CH-01`, kitchen extraction, gate controller, BMS points | G04, G14 |
| `FX-EMP-SET` | 14 synthetic employees | E1–E14 across both cost centers; contracts, rosters, leave, OT; bank accounts incl. 1 invalid (for rejection) | G05, G15 |
| `FX-CHEF-CREW` | Chef continuity | Primary chef, backup chef (both in FX-EMP-SET), 3 emergency chefs `EC1–EC3` (EC3 with expired food-safety credential) via `FX-VND-CHEF-AGENCY` | G15 |
| `FX-GUEST-SET` | 25 synthetic guests | Mixed languages, 2 accessibility needs, 1 VIP, loyalty members, consent variations, one duplicate profile pair | G03, G07, G13, G19 |
| `FX-REF-SET` | Referrers & referral chain for negative tests | Referrer `A` (individual), `B` (company), guest `G1` referred by A who later enrols as referrer `G1R`, guest `C` referred by `G1R`; referrer `R2` recruited by A | AT-REF |
| `FX-POINTS-PROG` | MetriStay Rewards test program | Earn 10 pts/OMR, valuation `PTS-VAL-TEST`, expiry 12 months, exclusions | G07 |
| `FX-VOUCHERS` | Gift vouchers | Issued, partially redeemed, expired | G07, G19 |
| `FX-PERIOD-2026-09` | A closed financial period with seeded transactions | For KPI hand-calculation & close tests (expected values in `fixtures/expected/*.csv`) | G08 |
| `FX-MEDIA-SET` | Media | 10 photos, 1 video with rights metadata; 1 photo without rights (negative) | G13 |
| `FX-AI-KB` | Approved knowledge base | 30 policy articles EN/AR with owners/review dates; 1 expired article | G13 |
| `FX-ID-SPECIMENS` | ID document specimens | Passport/national-ID specimen images per market incl. low-quality & glare variants | G13 |
| `FX-LOST-ITEMS` | Lost & found items | 5 items, 2 claimants (1 genuine, 1 false) | G14 |

### 3.2 Vendors and partners (simulators and sandboxes)

| Fixture id | Description | Mode | Used by |
|---|---|---|---|
| `FX-VND-MAINT` | Maintenance supplier (electrical/plumbing per-job pricing) | SIM vendor account | G04, G10, G16 |
| `FX-VND-VEG` | Vegetable supplier (fresh/frozen/pulp/powder; KG/gram/packet) | SIM | G16, G17, G18 |
| `FX-VND-VEG2`, `FX-VND-VEG3` | Additional vegetable bidders (VEG3 can be disabled to force 2-quote exception) | SIM | G17 |
| `FX-VND-MEAT` | Meat supplier (fresh/frozen, certificate evidence) | SIM | G16 |
| `FX-VND-HOSP` | Hospitality utilities supplier (soap, towels, bedsheets, pillows, printing, custom packing) | SIM | G16 |
| `FX-VND-UTIL-E` | Electricity supplier account + bill feed (CSV/PDF) | SIM | G04, G06, G08 |
| `FX-VND-UTIL-W` | Water supplier account + bills | SIM | G04 |
| `FX-VND-GAS-PIPE` | Pipeline gas supplier | SIM | G04 |
| `FX-VND-GAS-CYL` | Cylinder gas supplier (exchange, deposits) | SIM | G04 |
| `FX-VND-UTILCON` | Utility contractor (self-registering) | SIM | G10 |
| `FX-VND-AIR` | Airline agency (licensed seller fixture) | SIM; SBX if contracted | G10, G11 |
| `FX-VND-CRUISE` | Cruise operator | SIM; SBX if contracted | G10, G11 |
| `FX-VND-TAXI` | Taxi/transfer fleet | SIM; SBX if contracted | G10, G11 |
| `FX-VND-CHEF-AGENCY` | Emergency chef agency | SIM | G15 |
| `FX-PSP-SBX` | Payment service provider sandbox (one gateway, cards incl. 3DS, refunds, chargebacks, settlement file) | SBX (provider TBD, `docs/05`) → REAL-RB | G02, G06, G07, G20 |
| `FX-PSP-SIM` | Provider-neutral PSP simulator (fault injection: timeouts, duplicate webhooks, forged signatures) | SIM | G06, G20, SEC |
| `FX-BILLPAY-SIM` | Provider-neutral bill-pay simulator implementing inquire→quote→authorize→pay→status→reverse→receipt→settlement with timeout/pending injection | SIM | G06, G20 |
| `FX-BILLPAY-SBX` | Khedmah/ONEIC sandbox **only if** contract and sandbox granted (currently `blocked`, `docs/05`) | SBX/REAL-RB | G06.4 |
| `FX-BANK-SIM` | Bank/WPS file exchange simulator (accept, reject per line, returns) | SIM | G05, G20 |
| `FX-CHAN-SIM` | Channel manager simulator (ARI, bookings, cancellations, ack delays) | SIM → REAL-RB (certified channel) | G19, G20 |
| `FX-GOV-CA-SIM` | Canadian government filing simulator (file generation + portal/manual handoff, auth failure/outage injection) | SIM | G12 |
| `FX-SMS-SIM` / `FX-WA-SIM` | SMS / WhatsApp template OTP simulators | SIM → SBX | G13 |
| `FX-ESIGN-SIM` | E-signature provider simulator | SIM → SBX | G13 |
| `FX-IDOCR-SIM` | ID OCR engine stub returning controlled errors | SIM | G13, G20 |
| `FX-LPR-SIM` / `FX-GATE-SIM` | LPR server/edge camera event simulator and gate controller simulator (confidence, duplicates, outage) | SIM → REAL-RB (pilot hardware) | G03, HW |
| `FX-BMS-SIM` | BMS/fire-panel/camera alert simulator (read-only ingest) | SIM → REAL-RB where used | G14, HW |
| `FX-SCALE-SIM`, `FX-SCAN-SIM`, `FX-PROBE-SIM` | Scale, barcode/QR scanner, temperature probe simulators | SIM → REAL (pilot) | G18, HW |
| `FX-LLM-LOCAL` | Locally deployed guest-assistant model + enhancement pipeline test build | SIM (lab GPU/CPU) | G13 |
| `FX-LEGACY-EXPORT` | Synthetic legacy PMS/accounting/HR/POS export set (CSV/XLSX) with seeded defects | SIM | AT-MIG |

### 3.3 Five-market fixture properties and placeholder rule packs

All packs below are **`TEST-ONLY, not legal values`**. They exercise classifier logic, effective dating, component structure, invoice-template selection, filing-mode selection and gating. Rates are arbitrary; component names are generic (`TAX_A`, `TAX_B`, `LEVY_M`) except where a program name is needed to prove structure (e.g. CPP vs QPP), and even then the parameters are fake.

| Fixture id | Jurisdiction path | Legal entity | Rule pack (TEST-ONLY) | Pack status in fixture | Purpose |
|---|---|---|---|---|---|
| `FX-PROP-CA-ON` | Canada → Ontario → municipality `MUNI-TEST-ON` | `FX-LE-CA-01` | `RP-TEST-CA-ON-v0` (federal/provincial sales-tax component placeholders, municipal accommodation levy placeholder; payroll: CPP/EI/income-tax structure, fake parameters) | `verified-test` | G09, G12 |
| `FX-PROP-CA-QC` | Canada → Québec → `MUNI-TEST-QC` | `FX-LE-CA-02` | `RP-TEST-CA-QC-v0` (QPP/QPIP structure, provincial sales-tax component placeholder) | `verified-test` | G09, G12 |
| `FX-PROP-OM` | Oman → governorate `GOV-TEST` | `FX-LE-OM-01` | `RP-TEST-OM-v0` (VAT-like component placeholder, WPS file mode, referral gate `GATE-REF-OM=closed`) | `verified-test` | G05, G09, REF |
| `FX-PROP-PK` | Pakistan → province `PROV-TEST-PK` | `FX-LE-PK-01` | `RP-TEST-PK-v0` (provincial services-tax placeholder) | **`draft`** — deliberately unreviewed | G09.4 (blocking test) |
| `FX-PROP-SA` | Saudi Arabia → region `REG-TEST-SA` | `FX-LE-SA-01` | `RP-TEST-SA-v0` (e-invoice mode placeholder: "structured-invoice-required=true") | `verified-test` | G09 |
| `FX-PROP-PT` | Portugal → municipality `MUNI-TEST-PT` | `FX-LE-PT-01` | `RP-TEST-PT-v0` (tourist-levy placeholder, invoice-series/certified-software placeholder) | `verified-test`; one obligation `expired` | G09.4 |
| `FX-RP-GENERIC` | none | — | `RP-TEST-GENERIC-v1` (TT 10 %, LV 2 %, ESD 5 %, ERC 8 %) used by `docs/06` examples | `verified-test` | G08, finance |

`verified-test` is a **fixture-only status** that the rule engine accepts solely when `tenant.is_test = true`; a production tenant rejects any pack whose id begins with `RP-TEST-` (tested by `AT-SEC.14`).

---

## 4. Integrated scenario tests AT-G01 … AT-G20 (Section G)

Common preconditions for all AT-G tests: `FX-TEN-01`, `FX-HOTEL-G`, `FX-LE-G-01`, `FX-ROOMS-G`, `FX-RATES-G`, business date `2026-10-05`, role accounts per `docs/README` §3.3. "Phase" = first phase in which the case is runnable (normally on `SIM`); `REAL` runs are Phase 6 unless noted.

### AT-G01 — Corporate composite search (80 attendees)  ·  M09, M10, M11, M12, M15, M16, M17

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G01.1 | `FX-CORP-ACME` (booker), `FX-SPACE-BALL`, `FX-SPACE-MR2`, `FX-OUT-CLUB`, `FX-PRK-01`; two candidate dates D1, D2 with seeded availability | Booker searches D1 and D2: 80 attendees, classroom, 10 rooms, lunch, hosted bar, club visit, 20 parking passes, AV | Only configurations feasible on each date appear (ballroom classroom 90 ≥ 80; ≥10 rooms at ACME rate; club capacity; 20 passes); total price with TEST-ONLY taxes and expiry | API response, UI screenshots EN/AR, quote snapshot record | Every returned option passes all capacity checks when re-validated by an independent query; prices equal hand-computed expected CSV | 3 | SIM |
| AT-G01.2 | As .1; D2 has only 8 rooms free and MR2 is the only free space on a half-day | Search D2 | D2 shows "not feasible" with reasons (rooms 8 < 10; MR2 classroom 60 < 80); no bookable option offered | Response + reason codes | Zero infeasible options bookable; reasons listed | 3 | SIM |
| AT-G01.3 | `FX-CORP-ACME`, `FX-CORP-BETA` | Both corporates search same date/room type | Each sees only its own negotiated rate; BETA cannot see ACME rate or event entitlements (also via API id tampering) | Two responses; denied tampered requests | Rates differ as per agreements; 0 cross-visibility | 3 | SIM |
| AT-G01.4 | Signed Android and iOS corporate app builds (M11) installed on test devices | Repeat .1 on both apps | Same options/prices as web; accessibility labels present | Device recordings, build signatures | Parity diff = 0; builds signature-verified | 3 | REAL (devices); store publication not required |
| AT-G01.5 | Quote from .1 | Wait past quote expiry; attempt to book | Booking refused with "quote expired — re-price"; new quote reflects current rates/availability | Quote snapshot + audit | No booking from expired quote | 3 | SIM |
| AT-G01.6 | 2 attendees need accessible rooms | Search with accessibility filter | Options include ≥2 ACC rooms or are flagged infeasible | Response | Filter honoured | 3 | SIM |

### AT-G02 — Approval, deposit, PO; composite booking; no double sale; BEO propagation  ·  M09, M12, M16, M28, M10

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G02.1 | Quote from G01.1; `FX-PSP-SBX` | Booker submits; `corporate_approver` approves with PO `ACME-PO-88`; deposit 1,000.000 paid by pay-by-link | Composite booking `confirmed` atomically: room block, space schedule, catering production slots, club reservation, 20 parking passes; deposit EV-33 posted | Booking record, events `CompositeBookingConfirmed`, journal | All components confirmed in one transaction or none; deposit journal matches `docs/06` Ex.13 step a | 3 | SIM(3) → REAL-RB(6) [PSP] |
| AT-G02.2 | Two sessions (ACME and a walk-in sales agent) competing for the ballroom and last 10 rooms on D1 | Submit both within 50 ms | Exactly one succeeds; the other gets a conflict with alternatives | DB state, logs | No overlapping space booking; room-type-night stock never negative | 3 | SIM |
| AT-G02.3 | Parking capacity exhausted by another hold during composite commit | Commit composite | Whole composite fails with explicit component reason, or offers option without parking requiring re-approval; no orphan holds | Hold table snapshot | Zero orphan holds after 1 min | 3 | SIM |
| AT-G02.4 | Unconfirmed composite hold | Let hold expire | All component holds released together; inventory returns | Events `HoldExpired` | Inventory equals pre-hold state | 3 | SIM |
| AT-G02.5 | Confirmed booking; BEO v1 | Organizer changes covers 80→85 day 1 and adds nut allergy; catering_manager approves BEO v2 | Kitchen production plan, ingredient reservation and bar/hosted-bar plan update to v2; allergen flag visible on kitchen display; v1 marked superseded; organizer sees change order | BEO versions, KDS screenshot, ack events | Propagation ≤ 5 min (assumption A-OPS-01); kitchen & bar ack recorded | 3 | SIM |
| AT-G02.6 | Confirmed booking | Corporate cancels inside 50 % penalty window | Forfeit/refund per contract (`docs/06` Ex.20); all inventory released | Journal, PSP refund | Journal balances; refund once | 3 | SIM(3) → REAL-RB(6) [PSP] |

### AT-G03 — Check-in, LPR gate, single posting, stock depletion, club capacity  ·  M05, M06, M08, M13, M14, M15, M17

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G03.1 | Composite booking from G02; `FX-GUEST-SET` attendees | Front desk checks in 10 guests (ID policy per `RP-TEST-OM-v0`) | Rooms assigned (inventory separate from assignment), folios opened with routing to master/individual | Folios, events | 10 stays in-house; routing correct | 2 | SIM |
| AT-G03.2 | 20 plates registered to ACME passes; `FX-LPR-SIM` high-confidence read | Vehicle arrives at entry lane | Gate decision `open` for registered plate; session opened | LPR event, gate command log | Decision ≤ 2 s from observation (A-PERF-06) | 3 | SIM(3) → REAL-RB(6) [LPR/gate hardware] |
| AT-G03.3 | Low-confidence read (look-alike characters) | Vehicle arrives | Routed to attendant review; manual open with reason; audit | Review queue, override log | No automatic open below threshold; override audited | 3 | SIM → REAL-RB(6) |
| AT-G03.4 | Sessions from .2; duplicate exit events from 2 cameras | Vehicles exit; night audit runs | Parking charge and room charge post exactly once each | Journals EV-01, EV-37 | Count of postings = count of sessions/nights; duplicates ignored (`docs/06` Ex.18) | 3 | SIM |
| AT-G03.5 | `FX-RECIPES`, `FX-STORE-BAR`; event lunch and hosted bar | POS sales at bar; catering production posts | Stock depletion (theoretical) per recipe; issues reduce lot balances; COGS per cost model | Stock ledger, variance report | Theoretical depletion = Σ recipe qty × items sold | 3 | SIM |
| AT-G03.6 | `FX-OUT-CLUB` capacity 50; 49 inside | 2 guests attempt entry; one exits; retry | First admitted (50), second denied until one exits; admission charged only on entry; re-entry rules honoured | Entry log, charges | Occupancy never > 50; no charge on denial | 3 | SIM |
| AT-G03.7 | Gate/LPR offline | Vehicles enter/exit | Manual fallback (attendant ticket/plate entry) opens gate; sessions reconciled when LPR returns | Manual log, reconciliation | 100 % sessions reconciled; no double charge | 3 | SIM → REAL-RB(6) |

### AT-G04 — Procurement: cylinders, maintenance, utility bills  ·  M21, M22, M23, M24, M25, M26, M20

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G04.1 | `FX-CYL-L` with 3 full shells (= reorder point), `FX-VND-GAS-CYL` | Threshold triggers requisition → PO → supplier exchange (4 empty out, 5 full in incl. 1 new shell) → receipt → invoice → match | Custody ledger and deposits correct; postings per `docs/06` Ex.12 | Custody ledger, GRN, journals | 1310 = shells × deposit; journals equal expected | 4 | SIM |
| AT-G04.2 | Work order on `AS-CH-01`; `FX-VND-MAINT` | Requisition materials; vendor quote/PO; service completion with photos; engineer acceptance; invoice | 3-way match; room/asset cost tagged; AP approved | WO, PO, acceptance, invoice | Posted once; asset dimension set | 4 | SIM |
| AT-G04.3 | `FX-MTR-G-PIPE`, `FX-VND-GAS-PIPE` | Import pipeline gas bill; compare to meter; approve | Standing/variable lines captured; variance within tolerance; payable | Bill record | AP entry matches bill; allocation driver present | 4 | SIM |
| AT-G04.4 | `FX-MTR-E-MAIN` with intervals; `FX-VND-UTIL-E` bill PDF/CSV | Import bill (OCR/CSV); compare kWh | Variance flag if > tolerance; engineering review before approval | Variance record | Bill cannot be approved while flag open without reason | 4 | SIM |
| AT-G04.5 | `FX-MTR-W-MAIN` with injected leak pattern | Nightly anomaly job; bill import | Leak alert → work order; bill supply/sewage split; accrual reconciled | Alert, WO, bill | WO created ≤ 1 business day | 4 | SIM |
| AT-G04.6 | Same user holds requester role | Attempt to approve own requisition and release payment | Denied (segregation of duties) | Audit | 0 self-approvals | 4 | SIM |

### AT-G05 — Payroll, WPS/bank, confidentiality, labor cost  ·  M27, M38

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G05.1 | `FX-EMP-SET`, rosters, approved OT and leave, `RP-TEST-GENERIC-v1` | Compute gross-to-net preview; payroll_officer submits; payroll_approver approves | Payslips computed; exceptions listed (missing punches); GL aggregate journal per `docs/06` Ex.11 | Run record, journal | Net/gross match expected CSV to the minor unit | 4 | SIM |
| AT-G05.2 | Approved run; `FX-BANK-SIM` in WPS file mode of `FX-PROP-OM` | Generate & submit salary file | File validates against the **bank-provided** spec version (placeholder schema in SIM); status `submitted` | File hash, bank ack | File schema valid; no GL until confirmation | 4 | SIM(4) → REAL-RB(6) [payroll bank/WPS for Oman properties] |
| AT-G05.3 | One employee with invalid account | Bank rejects line | 2200 remains open for that employee; case to payroll_officer; corrected resubmission confirmed | Rejection file, case | Paid exactly once after resubmission | 4 | SIM → REAL-RB(6) |
| AT-G05.4 | GM, dept manager, finance_clerk accounts | Try to view individual pay via UI, API (`/employees/{id}/compensation`), report, export, global search, pivot differencing | All denied or suppressed; access attempts logged | Responses, access log | 0 individual pay values exposed to non-payroll roles | 4 | SIM |
| AT-G05.5 | Approved run | Open departmental P&L | Labor cost by department posted; F&B cell with < 3 employees shows total but suppresses averages | Report | Matches journal; k-rule applied | 4 | SIM |

### AT-G06 — Guest/corporate payments; bill-pay via authorized adapter or alternate  ·  M28, M29, M20

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G06.1 | `FX-PSP-SBX`; guest folio | Create intent, authorize (3DS), capture at checkout | Tokenized; no PAN/CVV stored in PMS tables (DB scan); journals EV-12 | PSP ids, DB scan result | Zero PAN patterns in DB/logs | 2 | SBX(2) → REAL-RB(6) [PSP certification] |
| AT-G06.2 | ACME city-ledger invoice | Pay-by-link and bank transfer receipt allocation | AR cleared; unallocated amount parked in 2500 until matched | AR ledger | Allocation correct | 4 | SIM/SBX |
| AT-G06.3 | Settlement file from PSP | Import & reconcile | 1040 clears; fees posted from statement | Recon report | Unmatched = 0 or explained | 5 | SBX → REAL-RB(6) |
| AT-G06.4 | Electricity bill approved; bill-pay adapter status | If Khedmah/ONEIC contract & sandbox exist → inquire/pay/receipt via `FX-BILLPAY-SBX`; else pay via approved bank path | Contract present: provider receipt stored, settlement reconciled. Absent: adapter shown `blocked` with external dependency, bank payment reconciled, AP closed | Adapter status screen, receipt, recon | No "paid" without receipt/bank confirmation; `blocked` visible to finance & GM | 5 | SIM(5); SBX/REAL-RB(6) **for properties that require electronic bill-pay** |
| AT-G06.5 | `FX-BILLPAY-SIM` injects timeout then late success, then duplicate callback | Submit payment | Order stays `pending`; inquiry before any retry; one confirmation; duplicate callback ignored | Order log | Exactly one payment and one AP settlement (Section L example) | 5 | SIM |
| AT-G06.6 | PSP webhook forged/replayed | Send | Rejected (see AT-SEC.5/6) | Logs | 0 state changes | 2 | SIM |

### AT-G07 — Points earn/redeem/refund; referral legal gate  ·  M30, M31

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G07.1 | `FX-POINTS-PROG`, member guest | Eligible dinner + ineligible item (tax, excluded product) | Points on eligible net only; pending → available after window | Points ledger | Points = expected | 5 | SIM |
| AT-G07.2 | Member with 1,000 pts | Redeem as partial tender | Liability released; no earn on redeemed part (`docs/06` Ex.5) | Ledger, journal | Balances match | 5 | SIM |
| AT-G07.3 | Paid dinner with earned points | Full refund; then duplicate PSP refund webhook; then UI retry with same idempotency key | Charge and points reversed exactly once (`docs/06` Ex.4) | Journals, points ledger, inbox dedup log | One reversal entry each; balances zero | 5 | SIM(5) → SBX |
| AT-G07.4 | Points older than expiry | Run expiry; liability report | Expired points leave liability; breakage posted; report reconciles | Report | 2130 = Σ outstanding × valuation | 5 | SIM |
| AT-G07.5 | `FX-PROP-OM` with `GATE-REF-OM=closed` | Referral accrual for a completed stay; attempt payout via UI, API and batch job | Accrual allowed (pending); payout rejected everywhere with gate reason | API responses, job log | 0 payouts; see AT-REF.6 | 5 | SIM |
| AT-G07.6 | Member | Attempt points cash-out and peer transfer | Not offered; API returns unsupported | Response | No such operation exists | 5 | SIM |

### AT-G08 — Night audit, month-end, KPIs, drill-through, incomplete estimate  ·  M08, M19, M32, M65

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G08.1 | Business date with open shift, open POS check, unmatched no-show | Run night audit | Blocking steps stop roll; after fixes, checklist NA-01…NA-17 (`docs/06` §13) passes; date rolls | Checklist record | Date cannot roll with blocking failure unless audited override | 2 | SIM |
| AT-G08.2 | `FX-PERIOD-2026-09` with expected values | Compute occupancy, ADR, RevPAR, TRevPAR, GOPPAR | Values equal hand-calculated `expected/kpi-2026-09.csv` using `docs/06` §12 definitions | KPI report + definition versions | Exact match (minor units / 2 dp %) | 2 (rooms KPIs), 4 (GOP) | SIM |
| AT-G08.3 | Allocation run `ALLOC-ELEC-2026-09-v2`, payroll journal, invoices | GM drills: dept contribution → UTL electricity → allocation → sub-meter readings → bill; labor → payroll aggregate (no individuals); AP → invoice → GRN | Every drill level resolves to source evidence | Drill path screenshots | No dead-end; salary not exposed | 4 | SIM |
| AT-G08.4 | **Incomplete-source rule**: remove September pipeline-gas bill and gas meter data | Open UTL, GOP, GM flash, month-end pack | Values labelled "incomplete estimate" with missing source "pipeline gas Sep — owner chief_engineer"; never "certified"; export carries status | Screens, export file | 0 occurrences of `certified`/`reconciled` for affected lines; banner on PDF | 4 | SIM |
| AT-G08.5 | Open period with all sources | Run month-end MC-01…MC-22; then post a late event dated in the closed period | Period locks; late event posts to next open period with original business date | Close record, journal | 2990 = 0; control totals equal | 4 | SIM |
| AT-G08.6 | Payables across all categories | Open reconciliation dashboard | Planned/accrued/invoiced/approved/paid/settled columns correct for electricity, water, gas, cylinders, salaries, maintenance, other suppliers, referral, bill-provider | Dashboard export | Column totals tie to GL control accounts | 4 | SIM |

### AT-G09 — Five-market jurisdiction classifier  ·  M44, M38

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G09.1 | `FX-PROP-CA-ON`, `-CA-QC`, `-OM`, `-PK`, `-SA`, `-PT` and legal entities | Classify each property + transaction (room night, catering, parking, payroll) | Distinct rule-pack ids/versions resolved per country/subnational/legal entity/effective date; precedence trace | Classification trace | Each resolves to its own pack; no cross-country reuse | 2 | SIM |
| AT-G09.2 | Same room + catering bundle | Quote & invoice in each property | Tax lines per TEST-ONLY pack components; invoice template/language per market (EN/AR/FR/PT as configured); invoice numbering series per entity | Quote, invoice PDFs | Components differ as configured; totals correct | 2 (quote), 4 (invoice) | SIM |
| AT-G09.3 | `RP-TEST-CA-ON` v0 → v1 effective mid-stay | Stay spanning change date | Nights before/after use respective versions | Folio lines with rule version | Split correct | 2 | SIM |
| AT-G09.4 | `FX-PROP-PK` pack `draft`; `FX-PROP-PT` one obligation `expired` | Attempt automated filing export & any dependent automated selling/payout | Blocked; UI shows reviewer, evidence status, contingency (manual review path) | Gate log, screen | 0 automated filings from draft/expired packs | 4 | SIM |
| AT-G09.5 | Property with missing subnational configuration | Price a taxable item | No implicit global fallback; item goes to manual/compliance review | Response | Price not auto-finalized | 2 | SIM |
| AT-G09.6 | All six properties | Open coverage/unknowns dashboard | Submission mode per obligation (API/file/portal/manual/blocked) with honest status labels | Dashboard | No obligation shown `certified` in fixtures | 4 | SIM |

### AT-G10 — Vendor self-registration, verification and departmental discovery  ·  M46, M26, M49

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G10.1 | `FX-VND-MAINT`, `-VEG`, `-UTILCON`, `-AIR`, `-CRUISE`, `-TAXI` | Each self-registers with category documents (licence, insurance, travel/transport permit as category requires) | Category-specific document checklists enforced; status `pending-verification` | Registration records | Missing required doc blocks submission | 3 | SIM |
| AT-G10.2 | Registered vendors | Maker verifies; same user attempts checker approval; different checker approves | Self-check denied; approval by second user | Audit | Maker ≠ checker | 3 | SIM |
| AT-G10.3 | Taxi fleet permit expiring tomorrow | Advance clock | Reminder then auto-suspension from new orders; retained for audit | Status history | Suspended vendor absent from order search | 3 | SIM |
| AT-G10.4 | Engineering, kitchen, concierge users | Search providers | Each sees only eligible providers for its service/location | Search results | 0 ineligible results | 3 | SIM |
| AT-G10.5 | Engineering RFQ to 3 eligible maintenance vendors | Invite, receive quotes, award, service acceptance, invoice | Reconciled invoice; vendor performance updated | RFQ, PO, invoice | 3-way match passes | 4 | SIM |
| AT-G10.6 | Vendor user of `FX-VND-MAINT` | Request another vendor's job/bid by id | 404/403; no data | Responses | BOLA prevented (see AT-SEC.3) | 3 | SIM |

### AT-G11 — Concierge travel: taxi, flight, cruise  ·  M45, M46

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G11.1 | Guest consent flow; `FX-VND-TAXI` | Front desk requests airport ride | Request sent; itinerary shows `requested` until provider confirmation ref; then `booked` | Order log | Never `booked` without external ref | 3 | SIM → SBX/REAL-RB(6) if contracted |
| AT-G11.2 | `FX-VND-AIR`, `FX-VND-CRUISE` | Request flight & cruise quotes via adapter or manual RFQ with staff evidence | Quote shows expiry, final price, taxes/fees, cancellation terms, merchant of record | Quote records | All fields present | 3 (manual), 5 (adapter) | SIM |
| AT-G11.3 | Quote past expiry | Attempt to book | Refused; re-quote | Audit | 0 bookings on expired quote | 3 | SIM |
| AT-G11.4 | Duplicate provider callback | Deliver twice | One order state change | Inbox log | Idempotent | 5 | SIM |
| AT-G11.5 | Confirmed flight canceled by supplier | Inject cancellation | Disruption case; guest notified; refund tracked to supplier receivable/payable; reconciliation | Case, ledger | Refund reconciled | 5 | SIM |
| AT-G11.6 | Property classified "concierge referrer" (not licensed seller) | Look for ticket issue/payment controls | Hidden/unavailable; referral-only path | UI/API | No ticket issuance possible | 3 | SIM |

### AT-G12 — Canadian hotel: tax, payroll, SIN, T4 artifact, government adapter  ·  M38, M44, M27

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G12.1 | `FX-PROP-CA-ON` and `FX-PROP-CA-QC` with TEST-ONLY packs | Quote identical room + catering bundle in each | Different component structures/effective dates per province; totals per placeholder values | Quotes | Correct per fixture; **production correctness requires `counsel-reviewed` pack** | 2 | SIM; REAL-RB(6) = counsel-reviewed pack for any Canadian pilot |
| AT-G12.2 | Canadian employees with synthetic SINs (valid & invalid checksum) | Enter, view, export | Checksum validation; encrypted at rest; masked display; unmask only payroll roles with step-up; access logged | DB inspection, screens, access log | No plaintext SIN in DB/logs/exports to non-payroll | 4 | SIM |
| AT-G12.3 | ON employee and QC employee | Run payroll | ON: CPP + EI + income-tax structure; QC: QPP + QPIP + EI (reduced-rate structure) + provincial structure — parameters fake | Payslips | Correct program selection by province | 4 | SIM |
| AT-G12.4 | Year-end | Generate T4-style artifact (and RL-slip structure for QC placeholder) | Artifact validates against the versioned spec schema held in fixtures; audit trail; submission = authorized receipt or labelled manual handoff | Artifact, handoff record | Never shows "filed" without receipt | 4 | SIM → REAL-RB(6) [authorized filing route or certified payroll provider] |
| AT-G12.5 | `FX-GOV-CA-SIM` auth failure then outage | Attempt submission | Auth error surfaced; outage → queued with manual path; no retry storm | Adapter log | Status honest; ≤ configured retries | 4 | SIM |
| AT-G12.6 | Config screens | Attempt to label a Canadian identifier "NIS" | Validation prevents; decision reference shown (`docs/07`) | Screen | Term not used as Canadian tax id | 4 | SIM |

### AT-G13 — Media, local enhancement, guest AI, ID OCR, e-sign, OTP  ·  M39, M40, M41

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G13.1 | `FX-MEDIA-SET` | Upload photo+video with rights; one without rights | Rights-cleared assets transcoded (derivatives, poster, subtitles); rightless asset blocked from publish | Media records | 0 rightless publications | 2 | SIM |
| AT-G13.2 | `FX-LLM-LOCAL` enhancement pipeline | Enhance a copy; compare; approve | Original immutable; provenance metadata; guest site shows approved version only; compute cost recorded | Hashes, CDN URLs | Original hash unchanged; unapproved never served | 3 | SIM (lab) |
| AT-G13.3 | `FX-AI-KB` | Guest asks policy question EN and AR | Answer cites approved article; assistant identity disclosed | Transcript | Citation present; expired article not used | 3 | SIM |
| AT-G13.4 | Disputed answer / prompt-injection attempt | Guest disputes; injection text in message | Human handoff with context; injection ignored; no tool misuse | Transcript, handoff case | Handoff ≤ SLA; 0 unauthorized tool calls | 3 | SIM |
| AT-G13.5 | `FX-ID-SPECIMENS`, `FX-IDOCR-SIM` with one wrong field | Scan ID; autofill; guest corrects | Field-by-field confirmation; correction stored; non-biometric path available; image deleted per timer | Form audit, deletion log | Correction wins; image purged on schedule | 2 | SIM |
| AT-G13.6 | Registration card | Guest e-signs | Envelope with document hash, intent, timestamp; tamper check fails on altered copy | Signature evidence | Tamper detected | 3 | SIM → SBX(5) |
| AT-G13.7 | OTP by SMS/WhatsApp; QR handoff | Complete verification; replay QR on second device; confirm booking | OTP bound to session; replay rejected; booking confirmed only after inventory hold + payment success | Auth logs, booking | 0 confirmations without payment; replay denied | 3 | SIM → SBX(5) [messaging providers] |

### AT-G14 — Incident detection/escalation; lost and found  ·  M42, M43

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G14.1 | `FX-BMS-SIM` smoke alert ×3 duplicates | Ingest | One incident (dedup), location/severity, operator confirmation required; playbook opened | Incident record | 1 incident; human confirmation logged | 3 | SIM → REAL-RB(6) where BMS/fire integration is enabled |
| AT-G14.2 | On-call roster | No ack within timeout | Escalation to next on-call and duty_manager | Escalation log | Escalation at configured time ±30 s | 3 | SIM |
| AT-G14.3 | Internet outage during incident | Continue response | Local fallback (on-prem alerts/SMS gateway or phone tree) and chronology continues; sync later | Chronology | No lost entries; immutable order | 3 | SIM (onprem-lab) |
| AT-G14.4 | `FX-LOST-ITEMS` | Intake item; two claimants | Genuine claimant verified and item released with custody chain; false claimant denied | Custody log | Chain unbroken; release authorized | 2 | SIM |
| AT-G14.5 | Item past retention | Run disposal job | Disposal/donation per policy with approval | Disposal record | Correct action per `RP-TEST-OM-v0` placeholder | 2 | SIM |

### AT-G15 — Chef coverage and emergency callout  ·  M47

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G15.1 | `FX-CHEF-CREW`; lunch service BEO | Primary and backup mark absent within meal-window SLA | Uncovered service detected; manager + eligible emergency roster alerted in configured order | Alerts | Detection ≤ 1 min | 3 | SIM |
| AT-G15.2 | EC1 and EC2 accept within 100 ms | Concurrent acceptance | Exactly one assignment (atomic lock); other told "filled" | Assignment table | 1 assignment | 3 | SIM |
| AT-G15.3 | EC3 credential expired | EC3 accepts first | Rejected as ineligible; next eligible offered | Eligibility log | Expired never assigned | 3 | SIM |
| AT-G15.4 | Assigned chef | Access BEO/allergen/menu; after shift try again | Access limited to assignment; revoked after | Access log | 0 post-shift access | 3 | SIM |
| AT-G15.5 | No acceptance by deadline | — | Escalation to GM; menu contingency/substitution approval workflow; guest/catering risk alert | Case | Escalation fires | 3 | SIM |
| AT-G15.6 | Completed emergency shift | Time and invoice | Agency invoice to AP (`docs/06` Ex.19 d); no double assignment | Journal | Posted once | 4 | SIM |

### AT-G16 — Vendor mobile catalog, variants, stock and pricing  ·  M48

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G16.1 | `FX-VND-VEG` on vendor app | Publish tomato fresh/frozen/pulp/powder; stock as-of; price per KG/gram/packet | Canonical UOM conversion correct; as-of timestamp in vendor TZ | Catalog records | Conversions exact | 3 | SIM + REAL devices (signed builds) |
| AT-G16.2 | `FX-VND-MEAT` | Publish fresh/frozen cuts with certificate evidence | Certificate required per category; expiry tracked | Records | Missing evidence blocks publish | 3 | SIM |
| AT-G16.3 | `FX-VND-HOSP` | List soap, towels, bedsheets, pillows; printing & custom packing options (MOQ, setup, lead time, tiers) | Structured attributes captured | Records | All required attributes | 3 | SIM |
| AT-G16.4 | `FX-VND-MAINT` | List electrical/plumbing per-job pricing with scope/exclusions/urgent surcharge | Per-job items searchable | Records | Present | 3 | SIM |
| AT-G16.5 | Kitchen, housekeeping, engineering staff | Search by department, location, category, freshness | Only approved vendors; stale stock badge after threshold; stock not treated as guaranteed | Search results | 0 unapproved; badge correct | 3 | SIM |
| AT-G16.6 | Vendor app offline | Draft offer offline; reconnect | Draft saved; cannot confirm sale/quote without server ack | Device log | No confirmation offline | 3 | REAL devices |

### AT-G17 — Requisition, samples, quotes, weighted award, PO  ·  M49

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G17.1 | `FX-VND-VEG`, `-VEG2`, `-VEG3`; kitchen requisition 120 KG with sample images | Issue RFQ; receive 3 quotes | Comparison normalizes landed price (pack/UOM/tax/freight/FX) | Comparison sheet | Normalization equals expected | 3 | SIM |
| AT-G17.2 | `-VEG3` disabled | Only 2 quotes | Exception requires reason + higher approver | Approval record | Cannot award without exception approval | 3 | SIM |
| AT-G17.3 | Sample images; property policy start event = RFQ close | Advance clock 89 d / 90 d / 91 d; one image under approved hold | Purged at 90 d from reference timestamp with deletion proof; held image retained with hold reason; bid/award/PO/invoice records unaffected (separate schedules) | Purge log, hold record | Purge on schedule ±1 day job window | 3 | SIM |
| AT-G17.4 | Published weights version | Evaluate; AI summary requested; attempt to change weights after bid opening | Scores computed with locked weights; AI summary only; weight change blocked | Evaluation record | Weight version locked | 3 | SIM |
| AT-G17.5 | Recommendation | Award signed; PO generated; vendor acknowledges | Versioned PO with SKU/UOM, price, delivery, sample ref | PO, ack | Ack recorded | 3 | SIM |
| AT-G17.6 | Approver overrides recommendation; conflict-of-interest declared | Override | Reason, SoD audit; conflict blocks the conflicted approver | Audit | Logged; conflicted user blocked | 3 | SIM |

### AT-G18 — AI follow-up, receiving, stock lifecycle, recall  ·  M50, M14

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G18.1 | PO from G17 | Milestones: AI drafts reminders; vendor replies "arriving late 14:00" | Summary + proposed ETA with confidence; late ETA flagged to human; AI never marks delivered | Message log | 0 AI-asserted deliveries | 3 | SIM |
| AT-G18.2 | Vendor ASN with lot/expiry; `FX-SCAN-SIM`, `FX-SCALE-SIM`, `FX-PROBE-SIM` | Gate check-in; scan; weigh; probe | Draft GRN with evidence; high-risk food requires accountable receiver attestation | GRN draft, evidence | Draft only until attestation | 3 | SIM → REAL(6) pilot devices |
| AT-G18.3 | 5 KG damaged, temperature breach on one crate | Receive | Short/damaged/breach → quarantine; vendor claim | Quarantine ledger | Not available for issue | 3 | SIM |
| AT-G18.4 | Accepted receipt | Invoice arrives | One stock-ledger entry, one AP match (`docs/06` Ex.14) | Ledgers | Exactly once | 4 | SIM |
| AT-G18.5 | Same barcode scanned 3×; duplicate ASN webhook | Receive | Idempotent; no extra stock | Ledger | Qty unchanged | 3 | SIM |
| AT-G18.6 | Issue to BEO | Recipe estimate; actual count; waste; attempt to return wasted item to stock | Waste ledger; return rejected; intact return accepted with inspector | Ledger | Available never increases from waste | 3 | SIM |
| AT-G18.7 | Recall on lot `L-7781` | Trace | Lists receipt, stores, issues, events (EV-ACME), remaining qty placed on hold | Trace report | Complete lot trace | 4 | SIM |

### AT-G19 — Website to repeat guest; revenue action; GM drill  ·  M51–M55, M53, M56, M32

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G19.1 | Website published from approved media/content | Visits from direct, search campaign tag, and channel referral | Attribution recorded per consent choices; bot sessions filtered | Attribution records | Correct source per session | 2 | SIM |
| AT-G19.2 | Low-bandwidth profile (throttled 3G), screen reader | Book room + optional upgrade | Correct total incl. TEST-ONLY taxes/fees before payment; WCAG checks pass (AT-A11Y) | Recording, price breakdown | Price at quote = price charged | 2 | SIM |
| AT-G19.3 | Booking from .2 | Pre-arrival message; linen/housekeeping tasks; room-service complaint; supervised recovery with capped compensation; checkout; consented review request | Each step owned, timed, closed; compensation within cap/approval; review request only with consent | Case timeline | All SLAs recorded; no request without consent | 3–4 | SIM |
| AT-G19.4 | Data from .1–.3 | Compute direct net acquisition cost, quote-to-book, recovery time | KPIs per `docs/06` K-18/K-19/K-20 | KPI output | Equals expected | 4 | SIM |
| AT-G19.5 | Revenue manager | Review 90-day forecast/pickup; change rate within guardrails; publish to channel; rollback | Channel ack recorded; rollback restores prior version; out-of-guardrail change requires approval | ARI logs | Ack before "live"; rollback exact | 3 (baseline), 5 (recommendations) | SIM → REAL-RB(6) [certified channel manager] |
| AT-G19.6 | Period data | GM drills from occupancy/profit through channel fee, labor, laundry, food waste, utilities to source events | Each resolves to journals and source events | Drill recording | No dead-end | 4 | SIM |

### AT-G20 — Exception storm  ·  cross-module

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G20.1 | 1 room left | 50 concurrent bookings (web, channel, front desk) | Exactly 1 succeeds | DB, logs | No oversell (see AT-PERF.1) | 2 | SIM |
| AT-G20.2 | `FX-PSP-SIM`, `FX-BILLPAY-SIM` | Duplicate payment webhook and provider callback | One state change each | Inbox | Idempotent | 2/5 | SIM |
| AT-G20.3 | `FX-IDOCR-SIM` error | OCR returns wrong DOB | Guest confirmation catches; mismatch review | Audit | Wrong value never auto-committed | 2 | SIM |
| AT-G20.4 | `FX-MTR-E-MAIN` gap 20 h | Run accrual | `incomplete-estimate` or interpolated `estimate` per coverage rule; gap alert | Alert | Label correct | 4 | SIM |
| AT-G20.5 | Bill kWh ≠ meter beyond tolerance | Import | Variance flag; approval blocked until reason | Flag | Blocked | 4 | SIM |
| AT-G20.6 | Supplier ships 110 of 120 | Receive | Short receipt; PO remains partially open or closed per rule; claim | GRN | Qty correct | 3 | SIM |
| AT-G20.7 | Same invoice twice (PDF + e-mail OCR) | Capture | Second rejected as duplicate | AP log | One payable | 4 | SIM |
| AT-G20.8 | `FX-BANK-SIM` rejects payroll file entirely | Submit | Run returns to `approved-unpaid`; case; resubmission | Case | Paid once | 4 | SIM |
| AT-G20.9 | Internet down at hotel (onprem-lab) for 2 h | Operate FO, POS, gate | Offline queues; manual fallbacks; sync with conflict queue on reconnection (see AT-OFF) | Sync report | 0 lost transactions | 3 | SIM (onprem-lab) |
| AT-G20.10 | ACME cancels the event 3 days before | Cancel | Attrition/forfeit per contract; inventory released; kitchen/bar/parking/club notified; ingredient reservations released | Events, journal | All components released | 3 | SIM |
| AT-G20.11 | After .1–.10 | Open exception dashboard & reconciliation | Every injected exception listed with owner, SLA, state; reconciliation shows 0 unexplained items | Dashboard export | 100 % of injected exceptions visible | 4 | SIM |

