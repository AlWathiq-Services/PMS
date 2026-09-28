# 09 — Acceptance Tests, Fixtures, Non-Functional Tests, Migration and Rollback

**Pack:** Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28 • **Master-prompt sections:** B (phase exits), E (referral non-negotiable tests), G (integrated demonstration 1–20), H.10, M (verification checklist), P (gates and invariants)

> This is a **test plan**, not a test report. No test here has been executed; no partner, hardware or government interface is claimed to exist. A test marked `REAL` cannot be passed with a simulator. Decision ids in this document use **D-901…D-910** (proposed; consolidated in `docs/13`).

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
| `FX-HOTEL-H` | Second property (Later, AT-G21 only) | Same tenant, different legal entity/currency option; used only for Phase 7 multi-property tests | AT-G21 |
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
| `FX-M31-01` | Referral worked-example booking (id used by `docs/01` catalogue; = `docs/06` Ex.6) | Net collected room revenue 100.000, attributable costs 70.000 under `RCF-v1`, rate 20 % → 6.000 | AT-G07.11 |
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

## 4. Integrated scenario tests AT-G01 … AT-G20 (Section G) and AT-G21 … AT-G25 (Later)

Common preconditions for all AT-G tests: `FX-TEN-01`, `FX-HOTEL-G`, `FX-LE-G-01`, `FX-ROOMS-G`, `FX-RATES-G`, business date `2026-10-05`, role accounts per `docs/README` §3.3. **Phase** = first phase in which the case is runnable (normally on `SIM`); `REAL` runs are Phase 6 unless noted. **Numbering is authoritative here**; the "Cross-document id reconciliation" table at the end of this section maps ids that other pack documents used provisionally.

### AT-G01 — Corporate composite search (80 attendees)  ·  M09, M10, M11, M12, M15, M16, M17

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G01.1 | `FX-CORP-ACME` (booker), `FX-SPACE-BALL`, `FX-SPACE-MR2`, `FX-OUT-CLUB`, `FX-PRK-01`; two candidate dates D1, D2 with seeded availability | Booker searches D1 and D2: 80 attendees, classroom, 10 rooms, lunch, hosted bar, club visit, 20 parking passes, AV | Only configurations feasible on each date appear (ballroom classroom 90 ≥ 80; ≥10 rooms at ACME contracted rate; club capacity; 20 passes); total price with TEST-ONLY taxes and expiry | API response, UI screenshots EN/AR, quote snapshot | Every returned option passes all capacity checks when re-validated independently; prices equal expected CSV | 3 | SIM |
| AT-G01.2 | As .1; D2 has only 8 rooms free; MR2 (classroom 60) is the only free space on D2 afternoon; parking passes limited | Search D2 | D2 composite shown "not feasible" with component reasons (rooms 8 < 10; MR2 60 < 80); no bookable option | Response + reason codes | Zero infeasible composite options bookable | 3 | SIM |
| AT-G01.3 | `FX-CORP-ACME`, `FX-CORP-BETA` | Both corporates search same date/room type; BETA tampers agreement id in API | Each sees only its own negotiated rate/eligibility; tampered request denied | Two responses; denied request log | Rates differ per agreements; 0 cross-visibility (Phase 3 exit "two corporations at different rates") | 3 | SIM |
| AT-G01.4 | Signed Android and iOS corporate app builds (M11) on test devices | Repeat .1 on both apps | Same options/prices as web | Device recordings, build signatures | Parity diff = 0; builds signature-verified (store publication not required) | 3 | REAL (devices) |
| AT-G01.5 | Quote from .1 | Wait past quote expiry; attempt to book | Refused "quote expired — re-price"; new quote reflects current rates/availability | Quote snapshot, audit | No booking from expired quote | 3 | SIM |
| AT-G01.6 | 2 attendees need accessible rooms | Search with accessibility requirement | Options include ≥2 ACC rooms or are flagged infeasible | Response | Requirement honoured | 3 | SIM |

### AT-G02 — Approval, deposit, PO; composite booking; no double sale; BEO propagation  ·  M09, M10, M12, M16, M28

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G02.1 | Quote from G01.1; `FX-PSP-SBX`; ACME credit limit | Booker submits; `corporate_approver` approves with PO `ACME-PO-88`; deposit 1,000.000 via pay-by-link | Composite booking `confirmed` atomically (room block, space, catering slots, club reservation, 20 passes, AV); deposit EV-33 posted | Booking, `CompositeBookingConfirmed`, journal | All components or none; journal = `docs/06` Ex.13 step a | 3 | SIM(3) → REAL-RB(6) [PSP] |
| AT-G02.2 | Two sessions (ACME and a sales agent) competing for the ballroom and last 10 rooms on D1 | Submit both within 50 ms | Exactly one succeeds; other gets conflict with alternatives | DB state, logs | No overlapping space booking; room-type-night stock never negative; parking/catering capacity never exceeded | 3 | SIM |
| AT-G02.3 | Confirmed booking; BEO v1 | Organizer changes covers 80→85 day 1 and adds nut allergy; catering_manager approves BEO v2 | Kitchen production plan, ingredient reservation and bar plan update to v2; allergen on KDS; v1 superseded; organizer sees change order | BEO versions, KDS screenshot, ack events | Propagation ≤ 5 min (assumption A-OPS-01); kitchen & bar acks recorded | 3 | SIM |
| AT-G02.4 | Event completed; attendees with personal incidentals | Post charges; route to master vs individual folios; post-event reconciliation of actual vs contracted | Master folio and individual folios per routing; actual-vs-contracted table; balance to city ledger (`docs/06` Ex.13) | Folios, reconciliation, journal | Totals equal expected; change orders only with organizer approval | 3 | SIM |
| AT-G02.5 | Parking capacity exhausted by another hold during commit | Commit composite | Whole composite fails with component reason or offers option without parking requiring re-approval; no orphan holds | Hold table snapshot | Zero orphan holds after 1 min | 3 | SIM |
| AT-G02.6 | Unconfirmed composite hold | Let hold expire | All component holds released together | `HoldExpired` events | Inventory equals pre-hold state | 3 | SIM |
| AT-G02.7 | Confirmed booking | Corporate cancels inside 50 % penalty window | Forfeit/refund per contract (`docs/06` Ex.20); inventory released | Journal, PSP refund | Journal balances; refund once | 3 | SIM(3) → REAL-RB(6) [PSP] |

### AT-G03 — Check-in, LPR gate, single posting, stock depletion, club capacity  ·  M05, M06, M08, M13, M14, M15, M17

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G03.1 | Composite booking from G02; `FX-GUEST-SET` attendees | Front desk checks readiness (ID/registration/payment/room clean-inspected) and checks in 10 guests | Rooms assigned (assignment separate from sold inventory); folios with routing to master/individual | Folios, events | 10 stays in-house; routing correct; not-ready room blocks check-in with reason | 2 | SIM |
| AT-G03.2 | 20 plates registered to ACME passes; `FX-LPR-SIM`; one look-alike low-confidence read | Vehicles arrive at entry lane | Registered high-confidence plate → gate `open`; low-confidence → attendant review; manual open with reason | LPR events, gate command log, override log | Decision ≤ 2 s from observation (A-PERF-06); no automatic open below threshold; overrides audited | 3 | SIM(3) → REAL-RB(6) [LPR camera/server + gate] |
| AT-G03.3 | Sessions from .2; duplicate exit events from two cameras; minibar consumption; bar check charged to room | Vehicles exit; minibar posted; night audit runs | Room, parking, minibar and POS room-charge each post exactly once | Journals EV-01/EV-24/EV-37 | Postings = sessions/nights/items; duplicates ignored (`docs/06` Ex.18) | 3 | SIM |
| AT-G03.4 | `FX-RECIPES`, `FX-STORE-BAR`; event lunch and hosted bar | POS sales; catering production posts | Theoretical depletion per recipe; issues reduce lot balances; COGS per cost model | Stock ledger, variance report | Theoretical depletion = Σ recipe qty × items sold | 3 | SIM |
| AT-G03.5 | `FX-OUT-CLUB` capacity 50; 49 inside | 2 guests attempt entry; one exits; retry | First admitted (50); second denied until one exits; admission charged only on entry; re-entry rules honoured | Entry log, charges | Occupancy never > 50; no charge on denial | 3 | SIM |
| AT-G03.6 | Gate/LPR offline | Vehicles enter/exit | Manual fallback (attendant plate entry/ticket) operates gate; sessions reconciled on recovery | Manual log, reconciliation | 100 % sessions reconciled; no double charge | 3 | SIM → REAL-RB(6) |

### AT-G04 — Procurement and operating costs: cylinders, maintenance, utility bills  ·  M20–M26

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G04.1 | `FX-CYL-L` at reorder point; `FX-VND-GAS-CYL` | Threshold → requisition → PO → exchange (4 empties out, 5 full in incl. 1 new shell) → receipt → invoice → match | Custody and deposits correct; postings per `docs/06` Ex.12 | Custody ledger, GRN, journals | 1310 = shells × deposit; journals = expected | 4 | SIM |
| AT-G04.2 | Work order on `AS-CH-01`; `FX-VND-MAINT` | Requisition materials; quote/PO; service completion with photos; engineer acceptance; invoice | 3-way match; asset/room cost tagged; AP approved | WO, PO, acceptance, invoice | Posted once; asset dimension set | 4 | SIM |
| AT-G04.3 | `FX-MTR-G-PIPE`, `FX-VND-GAS-PIPE` | Import pipeline gas bill; compare to meter; approve | Standing/variable lines; variance within tolerance; payable | Bill record | AP = bill; allocation driver present | 4 | SIM |
| AT-G04.4 | `FX-MTR-E-MAIN` intervals; `FX-VND-UTIL-E` bill PDF/CSV | Import bill (OCR/CSV); compare kWh | Variance flag if > tolerance; engineering review before approval | Variance record | Bill not approvable with open flag without reason | 4 | SIM |
| AT-G04.5 | `FX-MTR-W-MAIN` with leak pattern | Anomaly job; bill import | Leak alert → work order; supply/sewage split; accrual reconciled | Alert, WO, bill | WO ≤ 1 business day | 4 | SIM |
| AT-G04.6 | User holds requester role | Attempt to approve own requisition and release payment | Denied (segregation of duties) | Audit | 0 self-approvals | 4 | SIM |
| AT-G04.7 | Items from .1–.5 | Open operating-cost status view | Each category shows planned/accrued/invoiced/approved/paid/settled separately and ties to bank | Dashboard export | Totals tie to GL control accounts (see AT-G08.6) | 4 | SIM |

### AT-G05 — Payroll, WPS/bank, confidentiality, labor cost  ·  M27, M38

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G05.1 | `FX-EMP-SET`, rosters, approved OT and leave, `RP-TEST-GENERIC-v1` | Gross-to-net preview; payroll_officer submits; payroll_approver approves | Payslips; exceptions listed; aggregate GL journal per `docs/06` Ex.11 | Run record, journal | Net/gross = expected CSV to the minor unit | 4 | SIM |
| AT-G05.2 | Approved run; `FX-BANK-SIM` in WPS mode of `FX-PROP-OM` | Generate & submit salary file | Validates against **bank-provided** spec version (placeholder schema in SIM); status `submitted`; no GL until confirmation | File hash, bank ack | Schema valid | 4 | SIM(4) → REAL-RB(6) [payroll bank/WPS for Oman properties] |
| AT-G05.3 | One employee with invalid account | Bank rejects line | 2200 stays open for that employee; case; corrected resubmission confirmed | Rejection file, case | Paid exactly once | 4 | SIM → REAL-RB(6) |
| AT-G05.4 | GM, dept manager, finance_clerk | View individual pay via UI, API, report, export, search, pivot differencing | All denied/suppressed; attempts logged | Responses, access log | 0 individual pay values exposed to non-payroll roles | 4 | SIM |
| AT-G05.5 | Approved run | Open departmental P&L | Labor by department posted; F&B cell (< 3 employees) shows total but suppresses averages | Report | Matches journal; k-rule applied | 4 | SIM |

### AT-G06 — Guest/corporate payments; bill-pay via authorized adapter or alternate  ·  M28, M29, M20

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G06.1 | `FX-PSP-SBX`; guest folio | Intent, authorize (3DS), capture at checkout, partial refund | Tokenized; journals EV-12/EV-15; no PAN/CVV in PMS tables/logs (DB and log scan) | PSP ids, scan result | Zero PAN patterns; one capture, one refund | 2 | SBX(2) → REAL-RB(6) [PSP certification] |
| AT-G06.2 | ACME city-ledger invoice | Pay-by-link and bank transfer; allocate receipts | AR cleared; unallocated cash in 2500 until matched | AR ledger | Allocation correct | 4 | SIM/SBX |
| AT-G06.3 | Electricity bill approved | If Khedmah/ONEIC contract & sandbox exist → inquire/pay/receipt via `FX-BILLPAY-SBX`; else pay via approved bank path | Contract: provider receipt stored, settlement reconciled. Absent: adapter shown `blocked` as external dependency; bank payment reconciled; AP closed | Adapter status, receipt, reconciliation | No "paid" without receipt/bank confirmation; `blocked` visible to finance & GM | 5 | SIM(5); SBX/REAL-RB(6) **for properties that require electronic bill-pay** |
| AT-G06.4 | `FX-BILLPAY-SIM` timeout then late success, then duplicate callback | Submit payment | `pending`; inquiry before any retry; one confirmation; duplicate ignored | Order log | Exactly one payment and one AP settlement (Section L example) | 5 | SIM |
| AT-G06.5 | PSP settlement file | Import & reconcile | 1040 clears; fees from statement | Reconciliation | Unmatched = 0 or explained | 5 | SBX → REAL-RB(6) |
| AT-G06.6 | Forged and replayed PSP webhooks | Send | Rejected (AT-SEC.5/6) | Logs | 0 state changes | 2 | SIM |

### AT-G07 — Points, vouchers and referral (Section E)  ·  M30, M31, M54

Cases AT-G07.4–AT-G07.12 are the **Section E non-negotiable referral tests**; they are specified in full in §5 and listed here for numbering continuity.

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G07.1 | `FX-POINTS-PROG`, member guest | Eligible room + dinner + ineligible item (tax, excluded product) | Points on eligible net only; pending → available after window | Points ledger | Points = formula; ineligible lines earn 0 | 5 | SIM |
| AT-G07.2 | Member with 1,000 pts | Redeem as partial tender at checkout | Liability released; one card capture for remainder; no earn on redeemed part (`docs/06` Ex.5) | Ledger, journal | Balances match | 5 | SIM |
| AT-G07.3 | Paid stay/dinner with earned points (and one after redemption) | Full refund; duplicate PSP refund webhook; UI retry with same idempotency key | Charge and points reversed exactly once each (`docs/06` Ex.4; D-813 for already-redeemed points) | Journals, points ledger, inbox dedup | One reversal each; duplicates no-op | 5 | SIM(5) → SBX |
| AT-G07.4 – AT-G07.12 | see §5 | | | | | 5 (build/test); 6 (activation gate) | SIM |
| AT-G07.13 | `FX-VOUCHERS` | Issue, partial redeem, expire gift voucher; attempt redemption above balance | Liability ledger per `docs/06` Ex.7; over-redemption refused; breakage only if rule pack permits | Voucher ledger | 2120 = Σ open balances | 4 | SIM |
| AT-G07.14 | Points older than expiry | Expiry run; liability report | Breakage posted; liability reconciles | Report | 2130 = Σ outstanding × valuation | 5 | SIM |
| AT-G07.15 | Member | Attempt points cash-out and peer transfer | Not offered; API has no such operation; cash wallet feature flag off and gated | API catalogue, UI | No cash-out/transfer path exists | 5 | SIM |

### AT-G08 — Night audit, month-end, KPIs, drill-through, incomplete estimate  ·  M08, M19, M32, M60, M65

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G08.1 | Business date with an open cashier shift, open POS check, unresolved no-show, room-status discrepancy | Run night audit | Blocking steps stop the roll; after fixes NA-01…NA-17 (`docs/06` §13) pass; POS day-close journal = Σ checks by revenue center and tender; date rolls and locks | Checklist record, day-close journal | Date never rolls with blocking failure unless audited override | 2 | SIM |
| AT-G08.2 | `FX-PERIOD-2026-09` with expected values | Compute occupancy, ADR, RevPAR, TRevPAR, GOPPAR, food/beverage cost %, utility per occupied room; drill each to ledger lines, allocation run, meter reading, work order/invoice and source events | Values = `expected/kpi-2026-09.csv` using `docs/06` §12 definitions; every drill resolves; payroll drills stop at aggregate | KPI report + definition versions; drill recordings | Exact match (minor units / 2 dp %); no dead-end; no individual salary | 2 (rooms KPIs), 4 (cost KPIs, GOP) | SIM |
| AT-G08.3 | **Incomplete-source rule**: withhold September pipeline-gas bill and gas meter data (and, in a second run, the water bill) | Open UTL, departmental P&L, GOP, GM flash, month-end pack, export | Lines labelled **"incomplete estimate"** listing missing sources and owners; coverage gap shown, never zero-filled; never `certified` | Screens, export file | 0 occurrences of `certified`/`reconciled` on affected lines; PDF banner present | 4 | SIM |
| AT-G08.4 | Open period with all sources | Run month-end MC-01…MC-22: cashier shifts, guest ledger, city ledger/AR aging, AP, GRNI, payroll, PSP, bank, bill-provider, points, vouchers, referral control totals = GL; TB balanced per legal entity; lock period; then post a late event dated in the closed period | Period locks with two staff cost centers in P&L; late event posts to next open period with original business date | Close record, TB, journal | 2990 = 0; all control totals equal | 4 | SIM |
| AT-G08.5 | Allocation run `ALLOC-ELEC-2026-09-v1` (estimate) then v2 (actual) | Compute departmental contribution before/after allocation; supersede v1 | Contribution per dept and property operating result; v2 restates management view with version note; GL unchanged under D-809 | Allocation runs, restatement report | Numbers = `docs/06` §7.3 | 4 | SIM |
| AT-G08.6 | Payables/receivables across all categories | Open reconciliation dashboard | Planned/accrued/invoiced/approved/paid/settled correct for electricity, water, pipeline gas, cylinders, salaries, maintenance, other suppliers, referral, bill-provider | Dashboard export | Column totals tie to GL control accounts | 4 | SIM |

### AT-G09 — Five-market jurisdiction classifier  ·  M44, M38

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G09.1 | `FX-PROP-CA-ON`, `-CA-QC`, `-OM`, `-PK`, `-SA`, `-PT` with legal entities | Classify each property and transaction type (room night, catering, parking, payroll, guest registration) | Distinct rule-pack ids/versions per country/subnational/legal entity/effective date; precedence trace | Classification trace | No cross-country reuse | 2 | SIM |
| AT-G09.2 | Same room + catering bundle; `RP-TEST-CA-ON` v0 → v1 effective mid-stay | Quote and invoice in each property; run a stay across the change date | Tax lines per TEST-ONLY pack components; nights split by effective version; localized invoice outputs (language/fields/numbering series per entity) | Quotes, invoice PDFs, folio lines with rule version | Components and splits as configured; totals correct | 2 (quote), 4 (invoice) | SIM |
| AT-G09.3 | All six properties; `FX-PROP-PT` payroll obligation configured `manual_only` | Open obligation list per property; attempt submission | Submission mode per obligation (API/file/portal/manual/blocked) with reviewer, evidence, status; manual-only shows handoff task, not "submitted" | Obligation register, gate log | Mode honest; no `certified` in fixtures | 2 (register), 4 (filings) | SIM |
| AT-G09.4 | `FX-PROP-PK` pack `draft`; `FX-PROP-PT` one obligation `expired` | Attempt automated filing export and dependent automated selling/payout | Blocked; UI shows reviewer, evidence status and contingency (manual review path) | Gate log, screen | 0 automated actions from draft/expired packs | 2 (gate), 4 (filing) | SIM |
| AT-G09.5 | Property with missing subnational configuration | Price a taxable item | No implicit global fallback; manual/compliance review | Response | Price not auto-finalized | 2 | SIM |
| AT-G09.6 | All six properties | Open coverage/unknowns dashboard | Coverage and unknowns per obligation category; scheduled change alerts | Dashboard | Counts match fixture register | 4 | SIM |

### AT-G10 — Vendor self-registration, verification and departmental discovery  ·  M46, M26, M49

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G10.1 | `FX-VND-MAINT`, `-VEG`, `-UTILCON`, `-AIR`, `-CRUISE`, `-TAXI` | Each self-registers (legal entity, categories, locations, team accounts with MFA) | Category-specific document checklist; status `pending-verification` | Registration records | Missing required doc blocks submission | 2 (identity/taxonomy), 3 (portal) | SIM |
| AT-G10.2 | Registered vendors; taxi permit expiring tomorrow | Maker verifies credentials; same user attempts checker approval; second user approves; advance clock past permit expiry | Self-check denied; approval by different user; expiry reminder then auto-suspension from new orders (retained for audit) | Audit, status history | Maker ≠ checker; suspended vendor absent from order search | 3 | SIM |
| AT-G10.3 | Engineering, kitchen, concierge users | Search providers | Each sees only eligible, approved providers for its service/location | Search results | 0 ineligible results | 3 | SIM |
| AT-G10.4 | Engineering RFQ | Send one RFQ to 3 eligible maintenance vendors | Invitations only to eligible vendors; sealed bids; deadlines | RFQ record | Ineligible cannot be invited | 3 | SIM |
| AT-G10.5 | Award from .4 | Service accepted with evidence; invoice reconciled | 3-way match; vendor performance updated | PO, acceptance, invoice | Match passes; posted once | 4 | SIM |
| AT-G10.6 | Vendor user of `FX-VND-MAINT` | Request another vendor's job/bid/profile by id | 403/404; no data | Responses | BOLA prevented (AT-SEC.3) | 3 | SIM |

### AT-G11 — Concierge travel: taxi, flight, cruise  ·  M45, M46

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G11.1 | Guest consent flow; `FX-VND-TAXI`, `-AIR`, `-CRUISE` | Front desk records consent; requests airport taxi; requests flight and cruise quotes via adapter or manual RFQ with staff evidence | Each offer shows expiry, final price, taxes/fees, cancellation terms, merchant of record | Request/offer records | All fields present; traveler data purpose-limited | 3 (manual), 5 (adapter) | SIM → SBX/REAL-RB(6) if contracted |
| AT-G11.2 | Accepted offer | Book; provider confirmation delayed | Itinerary shows `requested/pending` until confirmed external reference; then `booked` | Order log | Never `booked` without external ref | 3 | SIM |
| AT-G11.3 | Offer past expiry | Attempt to accept | Refused; re-quote forced | Audit | 0 bookings on expired offer | 3 | SIM |
| AT-G11.4 | Duplicate provider callback; booking timeout | Deliver callback twice; inject timeout then late confirmation | One order, one payable/receivable | Inbox log | Idempotent | 5 | SIM |
| AT-G11.5 | Confirmed flight cancelled by supplier | Inject cancellation | Disruption case; guest notified; refund once; commission reversal once; reconciliation | Case, ledger | Refund and reversal exactly once | 5 | SIM |
| AT-G11.6 | Property classified "concierge referrer" (not licensed seller) | Inspect UI and API | No hold/book/issue/pay controls; referral-only path | UI/API | No ticket issuance possible | 3 | SIM |
| AT-G11.7 | Booked taxi transfer | Guest changes pickup time after hours; driver no-show | Change request to provider; after-hours escalation to duty manager; rescue ride; fees per terms | Case timeline | Escalation within SLA | 3 | SIM |

### AT-G12 — Canadian hotel: tax, payroll, SIN, T4 artifact, government adapter  ·  M38, M44, M27

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G12.1 | `FX-PROP-CA-ON` and `FX-PROP-CA-QC` with TEST-ONLY packs | Quote identical room + catering bundle in each; issue invoice | Different component structures/effective dates per province; totals per placeholder values | Quotes, invoices | Correct per fixture. **Production correctness requires a `counsel-reviewed` pack** | 2 | SIM; REAL-RB(6) = counsel-reviewed pack for any Canadian pilot |
| AT-G12.2 | ON and QC employees with synthetic SINs (valid & invalid checksum) | Enter, view, export SIN; run payroll | Checksum validation; encrypted; masked; unmask only payroll roles with step-up; ON → CPP + EI + income-tax structure, QC → QPP + QPIP + EI + provincial structure (fake parameters) | DB inspection, payslips, access log | No plaintext SIN outside payroll; correct program selection | 4 | SIM |
| AT-G12.3 | Year-end | Generate T4-style artifact (and QC slip structure placeholder) | Validates against versioned spec schema in fixtures; audit trail; submission = authorized receipt or labelled manual handoff | Artifact, handoff record | Never "filed" without receipt | 4 | SIM → REAL-RB(6) [authorized filing route or certified payroll provider] |
| AT-G12.4 | `FX-GOV-CA-SIM` auth failure then outage | Attempt submission | Auth error surfaced; outage → queued with manual path; bounded retries | Adapter log | Status honest; ≤ configured retries | 4 | SIM |
| AT-G12.5 | Configuration screens | Attempt to label a Canadian identifier "NIS" | Validation prevents; decision reference (`docs/07`) shown | Screen | Term never used as a Canadian tax id | 4 | SIM |

### AT-G13 — Media, local enhancement, guest AI, ID OCR, e-sign, OTP  ·  M39, M40, M41

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G13.1 | `FX-MEDIA-SET` | Upload photo+video with rights (and one without); approve; publish | Rights-cleared assets transcoded (derivatives, poster, subtitles) and published; rightless asset blocked; guest site/app show approved versions only | Media records, CDN URLs | 0 rightless or unapproved publications | 2 | SIM |
| AT-G13.2 | `FX-LLM-LOCAL` enhancement pipeline | Enhance a copy; compare before/after; approve or reject | Original immutable (hash); provenance metadata; compute cost recorded; no fabricated features | Hashes, job record | Original unchanged; unapproved never served | 3 | SIM (lab) |
| AT-G13.3 | `FX-AI-KB` | Guest asks policy question EN and AR; disputes answer; sends prompt-injection text; asks it to "book and pay" | Answers cite approved article; assistant disclosed; disputed answer → human handoff with context; injection ignored; booking only as draft | Transcript, handoff case | Citation present; expired article unused; 0 unauthorized tool calls | 3 | SIM |
| AT-G13.4 | `FX-ID-SPECIMENS`, `FX-IDOCR-SIM` with one wrong field | Scan ID; autofill; guest corrects; choose non-biometric path | Field-by-field confirmation; correction stored; image deleted on timer | Form audit, deletion log | Correction wins; image purged on schedule | 2 | SIM |
| AT-G13.5 | Registration card | Guest e-signs; alter a copy | Envelope with document hash, intent, timestamp; tamper check fails on altered copy | Signature evidence | Tamper detected | 3 | SIM → SBX(5) |
| AT-G13.6 | Guest with SMS consent, no WhatsApp consent | Request OTP | Sent only on consented channel with approved template; rate-limited; delivery failure → fallback path | Message log | No message on unconsented channel | 3 | SIM → SBX(5) [messaging providers] |
| AT-G13.7 | Verified OTP | Confirm booking | Confirmed only after inventory hold + payment success + verification | Booking record | 0 confirmations without payment | 3 | SIM |
| AT-G13.8 | QR handoff | Scan QR on second device; replay after use; replay after expiry | QR binds same session/device challenge; replay and expired QR rejected; QR alone never authenticates | Auth logs | Replays denied | 3 | SIM |

### AT-G14 — Incident detection/escalation; lost and found  ·  M42, M43

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G14.1 | `FX-BMS-SIM` smoke/gas alert ×3 duplicates; on-call roster | Ingest; no ack within timeout | One incident (dedup) with location/severity; operator confirmation required; playbook; escalation to next on-call and duty_manager | Incident record, escalation log | 1 incident; escalation at configured time ±30 s | 3 | SIM → REAL-RB(6) where BMS/fire integration enabled |
| AT-G14.2 | Internet outage during incident | Continue response | Immutable chronology; local fallback (on-prem alerting/SMS gateway or phone tree); sync later without loss | Chronology | No lost/reordered entries | 3 | SIM (onprem-lab) |
| AT-G14.3 | `FX-LOST-ITEMS` | Intake item; two claimants | Genuine claimant verified and released with custody chain; false claimant denied | Custody log | Chain unbroken; release authorized | 2 | SIM |
| AT-G14.4 | Item past retention | Disposal job | Disposal/donation per policy with approval | Disposal record | Correct action per placeholder policy | 2 | SIM |

### AT-G15 — Chef coverage and emergency callout  ·  M47

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G15.1 | `FX-CHEF-CREW`; lunch BEO | Primary and backup mark absent within meal-window SLA | Uncovered service detected; manager and eligible emergency roster alerted in configured order | Alerts | Detection ≤ 1 min | 3 | SIM |
| AT-G15.2 | Callout open | EC1 accepts (authenticated) | Acceptance reserves candidate under atomic lock; manager approval for paid external chef | Assignment | One authenticated acceptance | 3 | SIM |
| AT-G15.3 | EC3 credential expired | EC3 accepts first; then eligible chef assigned | EC3 rejected; eligible chef's food-safety qualification and shift availability verified; BEO/allergen/menu handed over, access limited to assignment and revoked after shift | Eligibility & access logs | Expired never assigned; 0 post-shift access | 3 | SIM |
| AT-G15.4 | EC1 and EC2 accept within 100 ms | Concurrent acceptance | Exactly one assignment; other told "filled" | Assignment table | 1 assignment | 3 | SIM |
| AT-G15.5 | No acceptance by deadline | — | Escalation to GM; menu contingency/substitution approval; guest/catering risk alert | Case | Escalation fires | 3 | SIM |
| AT-G15.6 | Completed emergency shift | Time and invoice | Agency invoice to AP or internal time to payroll exactly once (`docs/06` Ex.19 d) | Journal | Posted once | 4 | SIM |

### AT-G16 — Vendor mobile catalog, variants, stock and pricing  ·  M48

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G16.1 | `FX-VND-VEG`, `FX-VND-MEAT` on signed vendor apps | Publish fresh/frozen/pulp/powder variants and meat cuts (certificate evidence); dated available quantity; KG/gram/packet prices | Canonical UOM conversions exact; as-of in vendor TZ; missing certificate blocks meat publish | Catalog records | Conversions exact | 3 | SIM + REAL devices (signed builds) |
| AT-G16.2 | `FX-VND-HOSP` | List soap, towels, bedsheets, pillows; printing & custom packing (method, MOQ, setup, lead time, tiers) | Structured attributes captured | Records | All required attributes | 3 | SIM |
| AT-G16.3 | `FX-VND-MAINT` | List electrical/plumbing per-job pricing with scope, exclusions, urgent surcharge | Per-job items searchable | Records | Present | 3 | SIM |
| AT-G16.4 | Offer with stock as-of 30 h old | View and add to requisition | Stale badge; hotel-side hold/confirmation required before commitment; published stock not treated as guaranteed | UI, hold record | Badge correct; no commitment without hold | 3 | SIM |
| AT-G16.5 | Kitchen, housekeeping, engineering staff | Search by department, location, category, freshness | Only approved vendors; competitors' bids never visible to vendors | Search results | 0 unapproved results | 3 | SIM |
| AT-G16.6 | Vendor app offline | Draft offer offline; reconnect; sync twice | Draft saved; cannot confirm offer/quote without server ack; no duplicate publication | Device log | No offline confirmation; 1 publication | 3 | REAL devices |

### AT-G17 — Requisition, samples, quotes, weighted award, PO  ·  M49

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G17.1 | `FX-VND-VEG`, `-VEG2`, `-VEG3`; kitchen requisition 120 KG | Issue RFQ; 3 quotes; repeat with `-VEG3` disabled | 3 eligible quotes proceed; with 2, exception requires reason + higher approver | RFQ, approval | No award below minimum without exception | 3 | SIM |
| AT-G17.2 | Sample images | Upload via secure uploader (malware scan); request new photos; view side-by-side | Stored with ownership/consent; side-by-side comparison linked to bids | Upload log | Scan passes; linkage present | 3 | SIM |
| AT-G17.3 | Property policy start event = RFQ close | Advance clock 89 / 90 / 91 days; one image under approved hold | Purged at 90 days from reference timestamp with deletion proof; held image retained with reason; bid/award/PO/invoice records unaffected | Purge log, hold record | Purge within one job window | 3 | SIM |
| AT-G17.4 | Published weights version | Evaluate normalized landed price, quality, freshness, delivery, history (with sample size), capacity; attempt weight change after bid opening | Scores with locked weights; change blocked | Evaluation record | Normalization = expected; weight version locked | 3 | SIM |
| AT-G17.5 | Evaluation | Request AI summary; AI tool attempts award/weight change | Summary only; award and weight tools unavailable to AI | Tool log | 0 AI awards/weight edits | 3 | SIM |
| AT-G17.6 | Recommendation; approver with declared conflict | Override with reason; sign award; generate PO; vendor acknowledges; rejection path | Versioned PO (SKU/UOM, price, delivery, sample ref); conflicted approver blocked; SoD audit | PO, ack, audit | Ack recorded; override logged | 3 | SIM |

### AT-G18 — AI follow-up, receiving, stock lifecycle, recall  ·  M50, M14

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G18.1 | PO from G17 | AI drafts milestone reminders; vendor replies "late, 14:00" | Summary + proposed ETA with confidence; late ETA to human; AI never marks delivered | Message log | 0 AI-asserted deliveries | 3 | SIM |
| AT-G18.2 | Lot-coded ASN; `FX-SCAN-SIM`, `FX-SCALE-SIM`, `FX-PROBE-SIM` | Gate check-in; scan; weigh; probe | Draft GRN with raw evidence | GRN draft | Draft only until attestation | 3 | SIM → REAL(6) pilot devices |
| AT-G18.3 | High-risk food; 5 KG damaged; one crate temperature breach | Accountable receiver verifies | Short/damaged/breach → restricted quarantine and vendor claim | Quarantine ledger | Not available for issue | 3 | SIM |
| AT-G18.4 | Accepted receipt | Post | One stock-ledger entry per accepted line | Stock ledger | Exactly once | 3 | SIM |
| AT-G18.5 | Invoice arrives | 3-way match | One AP match (`docs/06` Ex.14) | AP ledger | Exactly once | 4 | SIM |
| AT-G18.6 | Issue to BEO | Recipe estimate; actual count; waste; try returning wasted item | Waste ledger; return rejected; intact return accepted with inspector | Ledger | Available never increases from waste | 3 | SIM |
| AT-G18.7 | Recall on lot `L-7781` | Trace | Receipt, stores, issues, events (EV-ACME) listed; remaining qty on hold | Trace report | Complete lot trace | 4 | SIM |
| AT-G18.8 | Barcode scanned 3×; duplicate ASN webhook | Receive | Idempotent | Ledger | Qty unchanged | 3 | SIM |

### AT-G19 — Website to repeat guest; revenue action; GM drill  ·  M51–M56, M18, M32, M53

Business walkthrough: `docs/11` §10 (steps 1–20). This table fixes the executable ids.

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G19.1 | Website published from approved media/content (walkthrough 1–3) | Visits from direct, search campaign tag, channel referral; bot traffic injected | Exactly one source per booking; analytics identifiers only after consent; bots filtered | Attribution records | Correct source per session | 2 | SIM |
| AT-G19.2 | Throttled 3G profile, screen reader; accessible room filter (walkthrough 4–6) | Quote, add paid upgrade, pay via PSP sandbox with duplicate webhook | All-in total incl. TEST-ONLY taxes/fees shown before payment = charged total; policy snapshot frozen; one capture; upgrade inventory never oversold | Recording, price breakdown | Quote price = charge; WCAG checks (AT-A11Y) pass | 2 | SIM/SBX |
| AT-G19.3 | Booking from .2 (walkthrough 7–8, 10–11) | Pre-arrival instructions/ID/sign; check-in; in-room dining complaint and towel request | Each request reaches the correct team with owner and SLA; upgrade posted once | Case/task timeline | SLAs recorded | 3 | SIM |
| AT-G19.4 | Arrival day (walkthrough 9) | Housekeeping prioritization, linen issue from par, inspection, minibar restock & one-time charge | Room ready before ETA or ETA shown; linen custody updated; minibar posted once | Room board, linen ledger, folio | No double minibar post | 2 (room board), 3 (linen/minibar) | SIM |
| AT-G19.5 | Complaint case (walkthrough 12) | Duty manager waives room-service charge (L1) and offers goodwill points or voucher (L2 approval) | Folio reversal and points/voucher entries linked to case; requester ≠ approver at L2; caps enforced | Case, ledger | Within cap; SoD enforced | 4 | SIM |
| AT-G19.6 | Case after remedy (walkthrough 13–15) | Guest confirms resolution; checkout; survey; consented review request; review ingested | Case closed with recovery time; review request only with consent; no incentive for review | Case, survey, consent log | Complaint-to-closure recorded; 0 unconsented requests | 3–4 | SIM |
| AT-G19.7 | Revenue manager (walkthrough 17–18) | Review 90-day forecast/pickup; guardrailed rate change; publish to channel; roll back; attempt out-of-guardrail change | Channel ack before "live"; rollback restores prior rate on web and channel; out-of-guardrail needs approval | ARI logs, rate versions | Ack recorded; rollback exact | 3 (baseline), 5 (recommendations) | SIM → REAL-RB(6) [certified channel manager] |
| AT-G19.8 | Data from .1–.6 (walkthrough 16) | Compute net acquisition cost per source, quote-to-book, recovery time | KPIs per `docs/06` K-18/K-19/K-20 with estimate/reconciled labels | KPI output | = expected | 4 | SIM |
| AT-G19.9 | Guest from .2 (walkthrough 20) | Withdraw marketing consent | Suppressed from pending campaigns immediately; no post-stay marketing | Consent log, campaign audit | 0 messages after withdrawal | 3 | SIM |
| AT-G19.10 | Period data (walkthrough 19) | GM drills from occupancy and profit through channel fee, labor, laundry, food waste, utilities to source events | Each figure resolves to journals and source events; missing source shows incomplete estimate | Drill recording | No dead-end | 4 | SIM |

### AT-G20 — Exception storm and failure injection  ·  cross-module

Cases .1–.10 follow the order of Section G item 20; .11 is the roll-up; .12–.16 are module-specific failure variants referenced by the catalogue.

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G20.1 | 1 room left; also 1 ballroom slot | 50 concurrent bookings (web, channel, front desk, corporate) | Exactly 1 succeeds per resource | DB, logs | No oversell (AT-PERF.1) | 2 | SIM |
| AT-G20.2 | `FX-PSP-SIM`, `FX-BILLPAY-SIM` | Duplicate payment webhook, duplicate provider callback, duplicate invoice issue request | One state change and one ledger effect each | Inbox | Idempotent | 2/5 | SIM |
| AT-G20.3 | `FX-IDOCR-SIM` error | OCR returns wrong date of birth | Guest confirmation catches; mismatch review | Audit | Wrong value never auto-committed | 2 | SIM |
| AT-G20.4 | `FX-MTR-E-MAIN` 20-h gap | Accrual run | `estimate` (interpolated) or `incomplete-estimate` per coverage rule; gap alert | Alert | Label correct | 4 | SIM |
| AT-G20.5 | Bill kWh ≠ meter beyond tolerance | Import | Variance flag; approval blocked until reason | Flag | Blocked | 4 | SIM |
| AT-G20.6 | Supplier ships 110 of 120 | Receive | Short receipt; claim; PO partially open or closed per rule | GRN | Qty correct | 3 | SIM |
| AT-G20.7 | Same invoice twice (PDF + e-mail OCR) | Capture | Second rejected as duplicate | AP log | One payable | 4 | SIM |
| AT-G20.8 | `FX-BANK-SIM` rejects whole payroll file | Submit | Run returns to `approved-unpaid`; case; resubmission | Case | Paid once | 4 | SIM |
| AT-G20.9 | Internet down at hotel (onprem-lab) 2 h | Operate FO, POS, parking gate, night audit | Offline queues; manual fallbacks; sync with conflict queue (AT-OFF) | Sync report | 0 lost transactions; no double posting | 3 | SIM (onprem-lab) |
| AT-G20.10 | ACME cancels event 3 days before | Cancel | Attrition/forfeit per contract; rooms/space/catering/parking/club released; ingredient reservations released | Events, journal | All components released | 3 | SIM |
| AT-G20.11 | After .1–.10 | Open exception dashboard & reconciliation | Every injected exception listed with owner, SLA, state; 0 unexplained reconciliation items | Dashboard export | 100 % of injected exceptions visible | 4 | SIM |
| AT-G20.12 | Travel order in flight; network outage then replay | Provider webhook duplicated during outage replay | Reconciled once; surfaced in exception queue | Inbox, order log | One order state | 5 | SIM |
| AT-G20.13 | Staff app offline | Chef submits absence report offline; reconnect | Queued report syncs; coverage slot state recomputes; callout starts | Device + server logs | No lost report | 3 | SIM |
| AT-G20.14 | Vendor app offline draft | Duplicate sync of the same draft/bid | No duplicate bid or stock publication | Catalog log | One record | 3 | SIM |
| AT-G20.15 | One lot with 10 KG | Two concurrent issues of 8 KG | One succeeds; other rejected/partial | Stock ledger | Lot never negative | 3 | SIM |
| AT-G20.16 | Receiving tablet offline | Confirm delivery offline; reconnect; retry | Offline confirmation syncs once; signed timestamp retained | GRN log | No duplicate receipt | 3 | SIM |

### AT-G21 … AT-G25 — Later-phase scenarios (release: **Later**, Phases 7–8)

Proposed in `docs/08` §7 (WP-7.x/WP-8.x). Not part of Customer Release 1; not runnable before the Phase 7/8 gates. Vendor-certification cases are `REAL-RB` only for properties that enable the capability.

| ID | Release | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|---|
| AT-G21.1 | Later | `FX-HOTEL-G` + second property `FX-HOTEL-H` under one portfolio | Create portfolio hierarchy; assign users with property scopes | Users see only authorized properties; tenant/property isolation holds | Access matrix test | 0 cross-property leakage | 7 | SIM |
| AT-G21.2 | Later | Portfolio | Central rate/content configuration pushed to both properties with local overrides | Versioned central config; local override audited | Config versions | Correct precedence | 7 | SIM |
| AT-G21.3 | Later | Two closed periods | Cross-property reporting and consolidation (per-currency, per-legal entity) | Consolidated KPIs; eliminations where configured; data-locality controls respected | Reports | = expected | 7 | SIM |
| AT-G21.4 | Later | Guest with stays at both properties | Shared identity with per-property consent; merge governance | Consent enforced per property/purpose; merge reviewable/reversible | Profile audit | No unconsented cross-property use | 7 | SIM |
| AT-G21.5 | Later | Golf/beach amenity enabled at one property | Book, sell, post to folio | Amenity hidden where not enabled; capacity enforced | Records | No sale of unavailable amenity | 7 | SIM |
| AT-G22.1 | Later | Named PBX vendor | Check-in/out/room move; DND; call charge; wake-up with escalation; local survivability | Status sync; call charges posted once; missed wake-up escalates | PBX logs | Certified adapter | 7 | REAL-RB [PBX] |
| AT-G22.2 | Later | Named HSIA vendor | Captive portal entitlement; premium tier purchase; checkout revocation | Access revoked at checkout; usage billed once | Network logs | Certified adapter | 7 | REAL-RB [HSIA] |
| AT-G22.3 | Later | Named IPTV vendor & content licence | Pair room device; purchase; checkout reset | Purchase posted once; device reset; casting isolated | Device logs | Certified adapter | 7 | REAL-RB [IPTV] |
| AT-G22.4 | Later | Named lock vendor; kiosk | Issue mobile key; room move; checkout; kiosk check-in | Key revoked on move/checkout; kiosk follows ID/payment rules | Lock audit | Certified adapter | 7 | REAL-RB [locks/kiosk] |
| AT-G23.1 | Later | 12 months history; licensed compset feed | Automated rule execution within guardrails | Actions within guardrails; human override; backtest vs actual | Rate logs | No guardrail breach | 7 | SIM → REAL (licensed data) |
| AT-G23.2 | Later | Staff copilot with bounded tools | Copilot proposes actions; approval required for consequential tools | No unapproved consequential action; evaluation harness scores logged | Tool logs | 0 unapproved actions | 7 | SIM |
| AT-G23.3 | Later | Partner certification kit | Third party runs kit; deprecation of API v1 | Certification report; deprecation notices & sunset honoured | Kit output | Kit passes/fails deterministically | 7 | SIM |
| AT-G24.1 | Later | Marketplace supplier listings (reusing M46 vendors) | Onboard, moderate, publish availability | Only verified suppliers listed; moderation audit | Listing records | 0 unverified listings | 8 | SIM |
| AT-G24.2 | Later | Licensed PSP marketplace-payments product | Supplier payout; dispute | Payout only via licensed PSP with partner KYC; dispute workflow | PSP refs | No self-custody | 8 | REAL-RB [licensed PSP] |
| AT-G24.3 | Later | Multi-property search; dynamic package | Build package; component tax/commission allocation; package-travel review per market | Allocation correct; market gate enforced | Package record | Gate honoured | 8 | SIM |
| AT-G24.4 | Later | Two hotels share supplier discovery | Search suppliers across hotels | Confidential bids/prices not shared across hotels | Search results | 0 confidential leakage | 8 | SIM |
| AT-G24.5 | Later | Fraud scenarios | Inject fraudulent listing/booking/payout | Fraud scoring, dispute resolution, evidence | Case records | All detected or queued | 8 | SIM |
| AT-G25.1 | Later | New legal opinion per market recorded | Enable partner-distribution referral in that market | Still single-tier; one referrer per booking; no recruitment or multi-level payouts; AT-G07.4–.12 re-run green | Gate record, test run | All Section E tests pass | 8 | SIM → activation gate |

### Cross-document id reconciliation (docs/09 is authoritative)

Other pack documents cited provisional sub-ids. The table lists every place where the **meaning** differs from the case above so the owning document can re-point its reference. Ids not listed are consistent (same meaning).

| Id as cited elsewhere | Where | Meaning there | Authoritative case in docs/09 |
|---|---|---|---|
| AT-G01.2 "contracted rate per corporation" / AT-G01.3 "rooms+parking+AV composite" | docs/08 §A1 | swapped | rate per corporation = **AT-G01.3**; composite infeasibility = **AT-G01.2** (docs/02 order kept) |
| AT-G02.4 "master/individual billing & post-event reconciliation" | docs/08 | same | AT-G02.4 (consistent) |
| AT-G03.2 (Folio/POS/Parking posting), AT-G03.3 (Event), AT-G03.4 (POS/stock) | docs/02 §18 | posting and depletion shifted by one | LPR gate = **AT-G03.2**; post-once = **AT-G03.3**; stock depletion = **AT-G03.4**; club capacity = **AT-G03.5** (docs/08 order kept) |
| AT-G06.3/.4 | all | consistent | settlement reconciliation added as **AT-G06.5**, webhook forgery **AT-G06.6** |
| AT-G07.2 "refund reverses (folio/PSP/points/referral)" | docs/02 §18 | refund | refund of charge+points = **AT-G07.3**; commission reversal = **AT-G07.7** |
| AT-G07.3 "SM-GiftVoucher" | docs/02 §18 | voucher | **AT-G07.13** |
| AT-G07.4 "referral non-negotiable tests" / AT-G07.5 "disabled jurisdiction" | docs/08 §A1 | grouped | expanded to **AT-G07.4–AT-G07.12** (catalogue 04 numbering adopted); disabled jurisdiction = **AT-G07.9** |
| AT-G08.2 "occupancy/ADR/RevPAR/TRevPAR" and "drill to evidence" | docs/08, catalogue 01/03 | split in 08 | KPIs **and** drill-through = **AT-G08.2** |
| AT-G08.3 "departmental contribution & allocation" | docs/08 §A1 | allocation | allocation = **AT-G08.5**; AT-G08.3 = incomplete-source rule (as catalogue 03 uses it) |
| AT-G08.4 "drill to evidence" | docs/08 | drill | drill = **AT-G08.2**; AT-G08.4 = month-end close incl. cashier shift and AR/city-ledger control (docs/02 & catalogue 03 meaning) |
| AT-G08.5 "missing source = estimate" | docs/08 | incomplete source | **AT-G08.3** |
| AT-G08.1 "month-end … certified P&L" | catalogue 03 (M19) | month-end | **AT-G08.4** |
| AT-G09.3 "localized invoices" | docs/08 | invoices | localized invoices = **AT-G09.2**; AT-G09.3 = submission mode incl. Portugal `manual_only` (catalogue 04 meaning) |
| AT-G10.4 "RFQ→service acceptance→invoice", AT-G10.5 "own jobs" | docs/08 | split differs | RFQ = **AT-G10.4**; acceptance+invoice = **AT-G10.5**; own jobs = **AT-G10.6** (catalogue 05 numbering) |
| AT-G11.2 "quotes", .3 "external ref", .4 "expiry/dup/cancel", .5 "no unlicensed" | docs/08 | differ | catalogue 05 numbering adopted: quotes in **.1**, external ref **.2**, expiry **.3**, duplicate/timeout **.4**, cancellation/refund **.5**, no unlicensed **.6**; after-hours change **.7** (docs/02 SM-TravelOrder) |
| AT-G12.2 (SIN/CPP/QPP) and AT-G12.3 (program selection) | early draft | merged | **AT-G12.2** covers both; T4 = **AT-G12.3**; adapter = **AT-G12.4**; NIS = **AT-G12.5** |
| AT-G13.3 "guest sees approved only" / .4 AI / .5 OCR / .6 e-sign / .7 OTP | docs/08 | shifted | approved-only folded into **AT-G13.1/.2**; AI **.3**; OCR **.4**; e-sign **.5**; OTP consent **.6**; confirmation after payment **.7**; QR replay **.8** (docs/02 numbering) |
| AT-G13.4 "bound OTP verification" | catalogue 02 (M18) | OTP | **AT-G13.6–AT-G13.8** |
| AT-G14.3 "gas incident chronology" | catalogue 03 (M24) | incident | incident chronology = **AT-G14.2**; AT-G14.3 = lost item |
| AT-G14.3 "lost-and-found" | docs/08 G2-6 | same | consistent |
| AT-G15.3 "qualification/handover", .4 "escalation" | docs/08 | shifted | catalogue 05 numbering: double assignment **.4**, escalation **.5**, posting **.6** |
| AT-G16.1 "vegetable/meat", .4 "approved-only search" | docs/08 | shifted | stale/hold = **AT-G16.4**; approved-only search = **AT-G16.5**; offline = **AT-G16.6** |
| AT-G17.2 "90-day purge", .3 "weighted", .4 "award", .5 "override" | docs/08 | shifted | catalogue 05 numbering: samples upload **.2**, purge **.3**, weighted **.4**, AI limits **.5**, award/PO/override **.6** |
| AT-G18.5 "issue to BEO", .6 "recall & duplicate scan" | docs/08 | shifted | stock once **.4**, AP once **.5**, BEO/waste **.6**, recall **.7**, duplicate scans **.8** (catalogue 05) |
| AT-G19.3 "stay journey", .4 "KPIs", .5 "rate", .6 "GM drill" | docs/08 | shifted | docs/02 numbering: requests **.3**, housekeeping/linen/minibar **.4**, compensation **.5**, complaint closure & review **.6**, rate **.7**, KPIs **.8**, consent **.9**; GM drill added **.10** |
| AT-G19.1–.9 as cited in catalogue 01 (AC-M01…M07) | catalogue 01 | several (WCAG, consent, upgrade stock, total price, rate reversibility…) | WCAG → **AT-A11Y.1** + **AT-G19.2**; consented review → **AT-G19.6**; upgrade stock/total → **AT-G19.2**; rate reversibility → **AT-G19.7**; stay traced → **AT-G19.3/.6**; linen → **AT-G19.4**; attribution + net cost → **AT-G19.1/.8** |
| AT-G19.1–.8 as cited in catalogue 06 | catalogue 06 | various | resolve by subject using the row above |
| AT-G20.1 "duplicate travel webhook during outage" | catalogue 05 (M45) | travel | **AT-G20.12** |
| AT-G20.2 "concurrent stock issues" | catalogue 05 (M50) | stock | **AT-G20.15** |
| AT-G20.3 "short delivery" | catalogue 05 (M50) | receiving | **AT-G20.6** |
| AT-G20.4 "repeated invoice" | catalogue 05 (M46/M49) | AP | **AT-G20.7** |
| AT-G20.5 "staff app offline absence report" | catalogue 05 (M47) | chef | **AT-G20.13** |
| AT-G20.6 "vendor offline duplicate sync" | catalogue 05 (M48) | vendor app | **AT-G20.14** |
| AT-G20.7 "receiving outage offline sync" | catalogue 05 (M50) | receiving | **AT-G20.16** |
| AT-G20.1–.7 as cited in catalogues 01/03/06 | catalogues | various failure types | resolve by subject to .1–.16 above |

---

## 5. Section E referral non-negotiable tests (AT-G07.4 – AT-G07.12) and extended referral suite (AT-REF)

Fixtures: `FX-REF-SET`, `FX-PROP-OM` (`GATE-REF-OM=closed` unless stated), a test-market property `FX-HOTEL-G` sandbox copy with gate `open-test`, contract formula `RCF-v1`, cost version `CPOR-STD-2026-10`, `FX-PSP-SIM`, `FX-BANK-SIM`. All runnable from Phase 5 on `SIM`; **payout activation in Oman additionally requires the documented legal and tax opinion (Phase 6 activation gate)** — it is not something a test can "pass" into existence.

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-G07.4 | Referrers A and B; one booking carrying A's code and B's last-click link | Complete paid stay; run attribution & accrual; attempt manual claim by B | One owner by deterministic precedence, or `disputed` queue; never two accruals | `referral_attribution` rows, accrual ledger | Exactly ≤ 1 commission row per booking; unique constraint holds under concurrent claims | 5 | SIM |
| AT-G07.5 | A "recruits" R2 (R2 enrols via A's invitation link) | R2 enrols; R2 generates bookings | A earns zero from R2's enrolment or bookings; enrolment creates no ledger rows; no referrer→referrer relation stored | Ledger, schema, API | 0 commission to A from R2 | 5 | SIM |
| AT-G07.6 | Guest G1 booked via A's code; later G1 enrols as referrer G1R and refers guest C | Both stays complete | A paid only for G1's eligible booking; C's booking pays G1R (if eligible) and never A | Commission statements | A's statement contains only G1's booking | 5 | SIM |
| AT-G07.7 | Qualified booking with accrued commission | Refund the booking; deliver refund event twice; also case after approval and after payout | Commission reversed exactly once (before approval: Dr 2140/Cr 6210; after payout: referrer receivable, `docs/06` Ex.6b) | Journals | One reversal per commission id | 5 | SIM |
| AT-G07.8 | Booking with net revenue 60.000 and attributable costs 70.000 (and one with exactly 70.000) | Calculate | Commission = 0 in both; statement shows "not eligible — margin ≤ 0" | Calc record | 0 journal lines | 5 | SIM |
| AT-G07.9 | Approved commission; jurisdiction disabled server-side while referrer UI still shows "enabled" (stale cache) | Attempt payout via UI, direct API call, and scheduled payout job | Rejected in all three with gate reason; earned commission remains `approved-held` (not deleted) | API responses, job log | 0 payouts | 5 | SIM |
| AT-G07.10 | Any paid commission | Open audit view | Shows booking id, net revenue, each cost component and source, formula version, cost version, calculation date, approvers, payment reference | Audit export | All fields present and reproducible from stored inputs | 5 | SIM |
| AT-G07.11 | Worked example `FX-M31-01` = `docs/06` Ex.6 | Complete stay; clear payment; pass refund window; accrue | 100.000 − 70.000 = 30.000 × 20 % = **6.000** accrued only after checkout, cleared payment and refund window; not before | Calc record, journal | Exactly 6.000; timing correct | 5 | SIM |
| AT-G07.12 | `FX-PROP-OM`, gate closed; evidence register has no legal/tax opinion | Build/sandbox accrual; attempt approval & payout; then record opinion evidence for final terms, referrer categories and promotion channels and open gate (test tenant only) | Accrual and sandbox work; payout blocked until evidence recorded by `compliance_officer`; gate change audited | Gate log, evidence record | No Oman payout without recorded opinion | 5 (build), 6 (activation) | SIM; activation = REAL-RB for Oman properties using referral |
| AT-REF.1 | Referrer books own stay (same identity/payment instrument/device) | Complete stay | Self-referral flagged; commission not accrued pending fraud review | Fraud case | 0 automatic accrual | 5 | SIM |
| AT-REF.2 | Enrolment flow | Attempt to configure a joining fee or purchase requirement | Not configurable; API rejects; enrolment has no payment step | Config API response | No fee path exists | 5 | SIM |
| AT-REF.3 | Referrer A | Call dashboard/statement APIs; try guest detail endpoints | Only booking id, stay month, status, amounts; no guest name/contact/ID | API responses | 0 guest PII fields | 5 | SIM |
| AT-REF.4 | Booking with and without referral code | Quote both | Guest price identical unless a separately approved promotion applies | Quotes | Prices equal | 5 | SIM |
| AT-REF.5 | Same inputs replayed (code, clicks, manual claim, consent state) | Re-run attribution | Same owner every time; cookie-less path honours consent choice | Attribution trace | Deterministic | 5 | SIM |
| AT-REF.6 | Kill switch by country, hotel, referrer category, campaign | Toggle each | Server/API/jobs enforce within 60 s (assumption); existing earned commissions retained per contract | Gate log | Enforcement everywhere; nothing deleted | 5 | SIM |
| AT-REF.7 | Database schema & API catalogue | Static scan | No referrer-to-referrer relation, rank, team volume, upline/downline fields or endpoints | Schema report | 0 such structures | 5 | SIM |
| AT-REF.8 | Share templates & landing page | Activate campaign without compensation disclosure | Activation blocked; with disclosure allowed | Campaign audit | Disclosure present on all templates | 5 | SIM |
| AT-REF.9 | Partial refund; amended stay length | Recalculate | Delta posted using the same formula/cost versions | Journals | Correct delta, once | 5 | SIM |
| AT-REF.10 | Referrer without verified tax/identity/payment details; withholding rule pack `unverified` | Attempt payout | Blocked with reason | Payout log | 0 payouts | 5 | SIM |

---

## 6. Performance tests (AT-PERF)

**All targets are planning assumptions** (A-PERF-nn) to be replaced by values agreed with the pilot hotel (P.6, D-902). Load model assumption: single hotel up to 500 rooms; SaaS reference tenant pool of 200 hotels on shared infrastructure; peak = 3× average.

| Assumption | Target (assumption) |
|---|---|
| A-PERF-01 | Availability/quote search p95 ≤ 800 ms at 50 req/s per property, 1,000 req/s pool |
| A-PERF-02 | Booking commit p95 ≤ 1.5 s (excluding PSP latency) |
| A-PERF-03 | POS check close p95 ≤ 500 ms; KDS ticket ≤ 2 s |
| A-PERF-04 | Night audit ≤ 10 min (60 rooms), ≤ 30 min (500 rooms) |
| A-PERF-05 | GM flash render ≤ 5 s; month-end P&L ≤ 30 s |
| A-PERF-06 | LPR observation → gate command ≤ 2 s end-to-end (on-site network) |
| A-PERF-07 | Webhook ingest 100/s sustained, 0 loss, dedup correct |
| A-PERF-08 | Outbox lag p99 ≤ 5 s |

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-PERF.1 | `FX-ROOMS-G` with 5 rooms left for a room-type-night | 500 concurrent booking attempts from web, channel sim, front desk, AI tool | Exactly 5 confirmed; others clean conflict; no deadlock timeouts > 1 % | DB counts, latency histogram | 0 oversell; A-PERF-02 met for successes | 2 | SIM (perf) |
| AT-PERF.2 | Timed resources (ballroom, club tables, parking) | 200 concurrent composite holds overlapping | No overlapping allocations; holds expire correctly | DB | 0 double-sold slots | 3 | SIM |
| AT-PERF.3 | Search load | Ramp to A-PERF-01 for 30 min | Latency within target; error rate < 0.1 % | Load report | A-PERF-01 | 2 | SIM |
| AT-PERF.4 | 500-room generated property, 7 outlets | Night audit | Completes within A-PERF-04 with correct totals | Timing log | A-PERF-04 | 2 | SIM |
| AT-PERF.5 | Webhook storm incl. 20 % duplicates | Send 100/s for 15 min | All processed once; outbox lag within A-PERF-08 | Inbox stats | 0 loss, 0 double effect | 2 | SIM |
| AT-PERF.6 | POS peak (bar happy hour) | 30 terminals × 2 checks/min | A-PERF-03 | Load report | Met | 3 | SIM |
| AT-PERF.7 | Month-end with 12 months data | Generate P&L, TB, KPI pack | A-PERF-05 | Timing | Met | 4 | SIM |
| AT-PERF.8 | On-prem reference hardware | Run AT-PERF.1/.4 on on-prem profile | Targets met or documented hardware sizing uplift | Report | Documented | 6 | onprem-lab |

---

## 7. Security tests (AT-SEC)

Aligned to OWASP API Security Top 10 categories and the threat model in `docs/07`. Run in CI (automated subset) and before each phase gate (full, including external penetration test before Phase 6).

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-SEC.1 | Two tenants with overlapping ids | Cross-tenant read/write through API, reports, exports, search, websockets; direct SQL as app role | Denied; Postgres RLS blocks even with forged `tenant_id` | Test log | 0 cross-tenant rows | 2 | SIM |
| AT-SEC.2 | Two properties in one tenant; user scoped to one | Access other property's objects | Denied | Log | 0 leakage | 2 | SIM |
| AT-SEC.3 | Objects: folio, reservation, guest profile, corporate invoice, vendor bid/job, referrer statement, payslip, ID image, incident evidence | Enumerate/alter ids across users of same role (BOLA) | 403/404 for every foreign object | Automated matrix | 0 BOLA findings | 2 | SIM |
| AT-SEC.4 | Vendor, corporate booker, housekeeper tokens | Call finance/admin/payroll functions (BFLA) | Denied | Matrix | 0 findings | 2 | SIM |
| AT-SEC.5 | PSP, bill-pay, channel, LPR, messaging webhooks | Invalid signature, wrong key, algorithm confusion, missing headers, body tampering | Rejected before parsing business payload; alert on repeated forgery | Logs | 0 state changes | 2 | SIM |
| AT-SEC.6 | Valid captured webhook | Replay after timestamp window; replay with same event id inside window | Rejected / deduplicated | Logs | 0 duplicate effects | 2 | SIM |
| AT-SEC.7 | Mutating money/stock endpoint | Reuse `Idempotency-Key` with different payload | 422 conflict; original result unchanged | Response | Correct | 2 | SIM |
| AT-SEC.8 | Privileged actions (payment release, payroll unmask, rule-pack activation, referral gate, manual journal) | Attempt without step-up / with stale MFA | Step-up demanded; action logged | Auth log | 100 % step-up | 2 | SIM |
| AT-SEC.9 | Public/corporate APIs | Mass assignment of price, status, tenant, role fields; excessive data exposure check | Server ignores/rejects; responses minimal | Tests | 0 findings | 2 | SIM |
| AT-SEC.10 | OTP, login, search, AI chat | Brute force / scraping rates | Rate limits and lockouts; no user enumeration | Logs | Limits hold | 2 | SIM |
| AT-SEC.11 | Logs, traces, error pages | Search for secrets, tokens, SIN, PAN, ID numbers | None present; vault rotation works without downtime | Scan report | 0 findings | 2 | SIM |
| AT-SEC.12 | Databases, logs, backups, analytics | PAN/CVV pattern scan; terminal integration mode check | No PAN/CVV anywhere; payment data only tokens | Scan report | 0 findings (PCI boundary, `docs/07`) | 2 | SIM |
| AT-SEC.13 | Devices (LPR, gate, scale, scanner, probe, BMS gateway) | Connect unregistered device / expired certificate | Rejected; alert | Device registry log | 0 accepted | 3 | SIM → REAL(6) |
| AT-SEC.14 | Production tenant | Load `RP-TEST-*` pack; enable fixture flags | Rejected | Config log | 0 test artifacts in prod | 2 | SIM |
| AT-SEC.15 | Guest AI and staff AI tools | Prompt injection to call unauthorized tools, reveal other guests' data, or change prices | Tool permission layer denies; PII masked | Transcript, tool log | 0 unauthorized calls | 3 | SIM |
| AT-SEC.16 | Upload endpoints (media, ID, samples, invoices, vendor docs) | Malware test files, polyglots, oversize, wrong MIME | Quarantined/rejected | Scan log | 0 accepted malicious files | 2 | SIM |
| AT-SEC.17 | URL-accepting fields (webhook targets, media import) | SSRF payloads (metadata IP, internal hosts) | Blocked by egress allow-list | Logs | 0 internal fetches | 2 | SIM |
| AT-SEC.18 | Audit log | Attempt update/delete as app and DBA roles; verify hash chain | Denied; chain verification detects tampering in copy | Verification report | Chain intact | 2 | SIM |

---

## 8. Privacy tests (AT-PRIV) — deletion versus legal hold

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-PRIV.1 | Guest with stays, marketing consent, preferences, reviews, AI transcripts | Submit verified deletion request | Marketing profile, preferences, transcripts erased/anonymized; folios, tax invoices, payment records retained under retention schedule and restricted; guest told which categories are retained and why | Deletion log, retention report | Only legally required records remain; access restricted | 2 | SIM |
| AT-PRIV.2 | Same guest linked to an open incident/chargeback (legal hold) | Deletion request | Held categories not deleted; hold reason, owner, review date shown; deletion completes automatically after hold release | Hold record, job log | No deletion under hold; completes after release | 2 | SIM |
| AT-PRIV.3 | ID images | Advance clock past retention | Deleted from primary storage; backup copies unrecoverable after backup window via per-object key destruction (crypto-shredding) | Key destruction log | Image not restorable after window | 2 | SIM |
| AT-PRIV.4 | Sample photos | = AT-G17.3 | 90-day purge vs approved hold | Purge log | As AT-G17.3 | 3 | SIM |
| AT-PRIV.5 | Employee leaves | Final settlement; access revoked; request erasure | Payroll/tax records retained per rule pack; SIN stays encrypted; non-required HR data deleted at schedule | Retention log | Correct per placeholder schedule | 4 | SIM |
| AT-PRIV.6 | Guest access request | Export | Complete, machine-readable; excludes other people's data and internal fraud notes where lawfully withheld | Export | Completeness checklist | 2 | SIM |
| AT-PRIV.7 | Website visitor declines analytics | Browse and book | No analytics identifiers set; booking works | Browser capture | 0 identifiers | 2 | SIM |
| AT-PRIV.8 | LPR plate observations | Advance past plate retention | Non-incident observations purged; plates linked to incident on hold retained | Purge log | Correct split | 3 | SIM |
| AT-PRIV.9 | Referrer data | Referrer leaves program | Referrer PII minimized after contract/tax retention; commission records retained | Retention log | Correct | 5 | SIM |
| AT-PRIV.10 | Backup restored after deletions | Restore to test env | Deletion replay log re-applies erasures before env is released | Restore log | Deleted data not resurrected | 2 | SIM |

---

## 9. Backup/restore and disaster-recovery drills (AT-DR)

Targets are **assumptions** until agreed with the pilot hotel (D-903). Every drill measures actual RPO/RTO and records them (KPI K-28, `docs/06`).

| Profile | RPO target (assumption) | RTO target (assumption) | Backup model |
|---|---|---|---|
| `saas` | ≤ 5 min (continuous WAL archiving + PITR) | ≤ 4 h full service; ≤ 1 h booking/folio read-only | Encrypted, cross-zone and cross-region copies; immutable retention |
| `onprem-single-hotel` | ≤ 15 min to local replica/NAS; ≤ 24 h to offsite if WAN down | ≤ 8 h with spare server; ≤ 2 h manual/paper mode start | Local + encrypted offsite; payroll schema separately keyed |

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-DR.1 | `int` env with full fixture activity | Destroy env; restore from backups into a clean account | Service restored; measured RPO/RTO recorded | Drill record with timestamps | Within targets or deviation logged with corrective action | 2 | SIM |
| AT-DR.2 | Accidental destructive change | PITR to T−1 min into a side env; extract rows | Recovery without full rollback | Drill record | Data recovered | 2 | SIM |
| AT-DR.3 | on-prem lab server | Power-pull and disk loss; rebuild on spare from offsite | Hotel operating result restored; paper-mode forms used meanwhile; re-entry reconciled | Drill record | Targets met; re-entry reconciled | 2 | onprem-lab |
| AT-DR.4 | SaaS zone failure | Fail over | Promotion of replica; clients reconnect | Drill record | RTO measured | 6 | SIM |
| AT-DR.5 | After any restore | Integrity suite: TB balanced; ledger hash chains; sub-ledger controls = GL; outbox replay produces 0 duplicates; in-flight payments/bill-pay/payroll resolved by **inquiry**, never blind resubmission | Integrity report | 100 % checks pass | 2 | SIM |
| AT-DR.6 | Key management | Restore payroll schema with separate key; simulate lost key procedure (escrow/dual control) | Restore works only with authorized key holders | Key ceremony log | Dual control enforced | 4 | SIM |
| AT-DR.7 | Ransomware scenario | Encrypt primary; attempt to delete backups with compromised admin | Immutable backups survive; restore succeeds | Drill record | Backups intact | 6 | SIM |
| AT-DR.8 | Monthly cadence | Automated restore test | Evidence per month for 3 consecutive months before Release 1 | Records | 3/3 | 6 | SIM |

---

## 10. Offline and outage tests (AT-OFF)

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-OFF.1 | `onprem-single-hotel`, WAN down 4 h | Check-in/out, folio postings, housekeeping, POS cash/room charge, night audit | Local operation continues; cloud-dependent actions queued (PSP online auth, channel ARI, messaging) with visible status | Ops log | 0 lost transactions; queue drains on recovery | 3 | onprem-lab |
| AT-OFF.2 | `saas`, hotel internet down | Staff app offline mode | Cached arrivals/room board readable; housekeeping status and tasks queued; new bookings not confirmable offline (paper fallback form with reference) | Device logs | No offline booking confirmation; queued updates sync | 3 | SIM |
| AT-OFF.3 | POS offline | Sell cash and room-charge; card only if PSP store-and-forward is contractually allowed | Queue sync exactly once; offline card limit enforced | POS logs | Idempotent sync | 3 | SIM → REAL(6) terminal |
| AT-OFF.4 | PSP outage | Checkout with card | Alternative tender or deferred settlement per policy; no double capture after recovery | PSP log | 0 duplicate captures | 2 | SIM |
| AT-OFF.5 | Channel manager outage 2 h | Sell direct meanwhile | ARI queue; oversell-exposure dashboard; reconciliation after recovery | Channel log | Exposure visible; reconciled | 3 | SIM |
| AT-OFF.6 | Bill-provider outage | Pay bill | Remains `pending`/manual path; no blind retry | Order log | 0 duplicates | 5 | SIM |
| AT-OFF.7 | On-prem server power loss | UPS then hard cut | Graceful shutdown on UPS signal; after hard cut, restart consistent (no partial ledger entries) | Integrity check | Consistent | 3 | onprem-lab |
| AT-OFF.8 | Device clock drift 10 min (gate, POS, scale) | Operate | Drift detected and alerted; server time authoritative for ledgers | Alert log | Detected | 3 | SIM |
| AT-OFF.9 | Two housekeepers update the same room offline | Sync | Conflict queue with both versions; supervisor resolves; audit | Conflict record | No silent overwrite | 3 | SIM |
| AT-OFF.10 | AI/LLM provider outage | Guest chat | Fallback message with human contact; no fabricated answers | Transcript | Correct fallback | 3 | SIM |

---

## 11. Accessibility and RTL tests (AT-A11Y)

Target: WCAG 2.2 AA for guest, corporate and vendor web and apps; staff apps meet AA for core flows. English and Arabic (RTL) initial languages; jurisdiction language packs (e.g. French for Québec, Portuguese for Portugal, Urdu if required for Pakistan) are configuration tested by string completeness.

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-A11Y.1 | Booking website EN and AR | Automated scan (axe-core) + manual audit by accessibility tester | No AA violations | Audit report | 0 AA failures | 2 | SIM |
| AT-A11Y.2 | Booking, check-in pre-arrival, corporate search, vendor catalog | Keyboard-only completion | All tasks completable; visible focus; no traps | Recording | 100 % tasks | 2 | SIM |
| AT-A11Y.3 | Same flows | Screen readers NVDA, VoiceOver (iOS/macOS), TalkBack | Labels, roles, live regions for price/availability changes, errors announced | Recording | Tasks completable | 2 | REAL devices |
| AT-A11Y.4 | Arabic UI | Inspect layout mirroring, directional icons, tables, charts, date pickers | Correctly mirrored; charts keep numeric axes readable | Screenshots | Checklist 100 % | 2 | SIM |
| AT-A11Y.5 | Mixed bidi content: plate numbers, amounts (OMR 3 dp), e-mails, room numbers, booking ids, phone numbers | Render in AR screens, PDFs, SMS | No reordering errors; isolates used | Screenshots, PDFs | 0 bidi defects | 2 | SIM |
| AT-A11Y.6 | Arabic invoices/receipts/kitchen tickets | Print on pilot printers and PDF | Correct shaping and fonts; legible | Printouts | Legible | 3 | REAL printers (pilot) |
| AT-A11Y.7 | Hold/quote timers | Timer expiry warnings; extend option | Adjustable/extendable per 2.2.1; CAPTCHA-free bot protection alternatives | Recording | Pass | 2 | SIM |
| AT-A11Y.8 | Staff apps (kitchen/housekeeping/parking) | Large touch targets, contrast in bright/dim conditions, glove use | Targets ≥ 24×24 CSS px (2.5.8), contrast AA | Audit | Pass | 3 | REAL devices |
| AT-A11Y.9 | Localized formats | Dates (Gregorian; optional Hijri display), numerals setting, currency minor units per currency | Consistent and configurable | Screens | Pass | 2 | SIM |
| AT-A11Y.10 | Language packs FR/PT (and others enabled) | String completeness check | 100 % keys translated or fallback flagged before activation | Report | No missing keys in activated packs | 4 | SIM |

---
## 12. Data migration plan (legacy PMS, accounting, HR/payroll, POS)

The pilot hotel's current systems are unknown (Section J; decision D-904, owner: Product Owner + pilot GM). This plan is system-agnostic and is refined once source systems and export capabilities are known. **Migration never writes directly into ledgers**: it loads through the same domain commands/events as live operation (or dedicated, audited migration commands that emit events), so every migrated balance has a source.

### 12.1 Scope and approach per entity

| # | Entity | Legacy source | Method | Transform / rules | Validation (automated) | Owner | Timing |
|---|---|---|---|---|---|---|---|
| 1 | Property setup: room types, rooms, attributes, outlets, spaces, tax/rule-pack assignment | PMS config, manual | Configured fresh; legacy used as reference | Rule packs **never migrated** — created via M44 evidence workflow | Room count = physical inventory walk | property_admin | T−45 |
| 2 | Chart of accounts & mapping | Accounting | Map legacy accounts → `docs/06` template; keep legacy code as attribute | 1:n / n:1 mapping table approved | Every legacy account with balance mapped | financial_controller | T−40 |
| 3 | Guest profiles | PMS/CRM | Extract → cleanse → dedup (M52 rules) → load with consent status | Consent migrated only if lawful basis evidenced; otherwise `unknown` = no marketing; ID images **not** migrated unless lawful and needed | Counts, duplicate rate, consent coverage report | dpo + guest_relations | T−21 (bulk), T−1 (delta) |
| 4 | Corporate accounts & negotiated rates | PMS/sales | Load accounts, contacts, agreements (versioned) | Expired agreements archived | Rate spot-check 100 % of active agreements | sales_manager | T−21 |
| 5 | Reservations on-the-books (future) | PMS | Load as reservations with original confirmation no., rate, guarantee, deposits, routing | Deposits load as liabilities matched to opening 2100 balance; card guarantees re-tokenized via PSP token migration or re-collected (never raw PAN import) | Room-nights by date & revenue on-the-books = legacy report (±0) | front_office_manager | T−7 bulk, T−0 delta |
| 6 | Groups/events/BEOs | Sales & catering | Load blocks, pickup, contracts, BEO latest version, deposits | Historic BEO versions attached as documents | Block/pickup match | catering_manager | T−14 |
| 7 | In-house guests & open folios | PMS at cutover night | Transfer open folio balances as opening charge lines per folio (summary with legacy detail attached) | Night of cutover after legacy night audit | Σ open folios = legacy guest ledger = opening 1100 | night_auditor + financial_controller | T−0 |
| 8 | City ledger / AR | Accounting | Open invoices individually with aging dates | Credit notes/unapplied cash loaded | Aging buckets = legacy aging; Σ = opening 1110 | ar_clerk | T−0 |
| 9 | AP open items | Accounting | Open invoices, credits, GRNI list | Duplicate keys preserved for detection | Σ = opening 2000/2020 | ap_clerk | T−0 |
| 10 | Stock on hand | POS/inventory or count | **Physical count at cutover** is authoritative; legacy valuation used for cost | Lots/expiry captured for food; quarantine separated | Count sheet = loaded qty; value = opening 12xx | storekeeper | T−0 |
| 11 | Cylinder custody & deposits | Supplier statement + count | Load shells by type and deposit | | Shells × deposit = 1310 | storekeeper | T−0 |
| 12 | Employees, contracts, compensation history (current + YTD) | HR/payroll | Load master data, effective-dated compensation, leave balances, YTD payroll totals needed for statutory computation | SIN/national IDs encrypted on load; bank details re-verified | Headcount, YTD totals by statutory bucket = legacy | payroll_officer + hr_officer | T−30 (parallel payroll), T−0 |
| 13 | Loyalty balances / vouchers | Legacy program | Load as opening points/voucher ledger entries with source "migration" | Points valuation version approved | Σ = opening 2130/2120 | referral_program_admin + financial_controller | T−0 |
| 14 | Opening trial balance | Accounting | Load via migration journal against **3900 Opening balance equity** by account; sub-ledger-backed accounts must be matched by their sub-ledger loads | Period = first open period | 3900 = 0 after all sub-ledger loads; TB = legacy TB | financial_controller | T−0 |
| 15 | Vendors | Accounting/procurement | Load legal entity, bank (verified), categories | Unverified vendors → `pending-verification`, cannot be paid | Bank detail change review queue empty | procurement_officer | T−30 |
| 16 | Assets & maintenance history | Engineering records | Asset register, PM schedules, open work orders | | Asset count | chief_engineer | T−30 |
| 17 | Historical statistics (for forecasting/KPIs) | PMS reports | Load daily stats (rooms sold, revenue by segment) 2–3 years as statistics, not transactions | Labelled `imported-history` | Monthly totals = legacy | revenue_manager | T−30 |

### 12.2 Dual-run (parallel operation)
- **Payroll:** at least one full pay cycle in parallel (legacy pays; MetriStay computes); differences per employee explained before cutover (AT-MIG.6).
- **PMS/folio:** 3–7 nights (assumption D-905) shadow night audit: reservations and postings keyed/imported into MetriStay; compare occupancy, ADR, revenue by department, guest ledger, deposits each morning; differences explained.
- **Accounting:** first month-end closed in both systems; TB comparison.
- **POS:** one outlet pilot for 3 service days before all outlets.

### 12.3 Cutover checklist

| When | Step | Owner |
|---|---|---|
| T−60 | Source systems, export formats, contacts confirmed; migration mapping approved; freeze on legacy config changes planned | Product Owner + pilot GM |
| T−45 | Property configured; rule packs at required status for go-live obligations or manual path documented | property_admin, compliance_officer |
| T−30 | Rehearsal 1 (full load into `pilot` sandbox property) + AT-MIG suite; payroll parallel starts | migration lead |
| T−14 | Rehearsal 2 with timing (must fit in cutover window); staff training complete; hardware pilots passed (§13) | migration lead, it_admin |
| T−7 | Go/No-Go 1: rehearsal defects closed; reservations bulk load; PSP token migration or re-collection plan executing | GM, financial_controller |
| T−1 | Delta loads; freeze new legacy bookings at set time; channel manager mapping switched in test mode | front_office_manager |
| T−0 (night) | Legacy final night audit → extract in-house folios, AR, AP, TB, stock count, cylinder count → load → validate (AT-MIG.1–.5) → **Go/No-Go 2** → switch channel connectivity and website booking engine → first MetriStay night audit | migration lead + night_auditor |
| T+1…T+7 | Hypercare; daily reconciliation; legacy read-only | all owners |
| T+30 | First month-end in MetriStay certified; legacy archived per retention | financial_controller |

### 12.4 Rollback (migration)
- **Before Go/No-Go 2:** abort; legacy remains system of record; nothing to reverse.
- **Point of no return** = first external commitment from MetriStay (channel switch + first new booking/payment). Until then rollback = re-enable legacy, discard MetriStay tenant data (keep evidence).
- **After point of no return (within T+72 h):** rollback only by **forward re-entry into legacy** using the MetriStay transaction export (bookings, payments, postings since cutover) — a controlled, reconciled procedure with a named owner; MetriStay ledgers are archived, not deleted. Criteria for invoking: inability to check in/out or take payments for > 2 h with no workaround, or unreconcilable guest ledger difference > threshold (D-906).
- Channel mappings and website switch-back have a scripted procedure tested in rehearsal 2.

### 12.5 Migration acceptance tests (AT-MIG)

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-MIG.1 | `FX-LEGACY-EXPORT` with seeded defects (duplicates, bad dates, orphan folios) | Run full load | Defects rejected to error queue with reasons; nothing partially loaded | Load report | 100 % defects caught | 6 | SIM |
| AT-MIG.2 | Loaded reservations | Compare on-the-books by date/room type/revenue | Equal to legacy | Comparison report | 0 difference | 6 | SIM → REAL (pilot) |
| AT-MIG.3 | Opening balances | TB compare; 3900 = 0 | Equal | TB report | 0 difference | 6 | SIM → REAL |
| AT-MIG.4 | Open folios, AR, AP, deposits, points, vouchers | Sub-ledger vs GL control | Equal | Control report | 0 difference | 6 | SIM → REAL |
| AT-MIG.5 | Stock & cylinders | Count vs loaded | Equal | Count sheets | 0 difference | 6 | REAL |
| AT-MIG.6 | Payroll parallel run | Compare per employee (payroll roles only) | Differences explained | Parallel report | 100 % explained | 6 | REAL |
| AT-MIG.7 | Guest profiles | Consent migration | No marketing to `unknown`-consent guests | Campaign audit | 0 sends | 6 | SIM |
| AT-MIG.8 | Card guarantees | Search migrated data for PAN | None; tokens only | Scan | 0 PAN | 6 | SIM |
| AT-MIG.9 | Rehearsal 2 | Time full cutover | Fits window (assumption ≤ 6 h) | Timing | Within window | 6 | REAL |
| AT-MIG.10 | Rollback rehearsal | Execute rollback before point of no return and a forward re-entry drill | Legacy restored; re-entry reconciled | Drill record | Pass | 6 | REAL |

---

## 13. Pilot hardware and vendor acceptance criteria (AT-HW)

Each device class is accepted **per site** with the actual model, firmware and network. Thresholds are **assumptions to be agreed with the pilot hotel and vendor** (D-907); where a vendor publishes a specification, the site test verifies it rather than trusting it. Failure → the related workflow is a release blocker for that property (`-RB`) unless an approved manual path is accepted by the GM and recorded.

| ID | Device / vendor | Acceptance criteria (assumption values) | Test method | Evidence | Mode |
|---|---|---|---|---|---|
| AT-HW.1 | LPR camera (edge AI) | Plate read accuracy ≥ 98 % day, ≥ 95 % night/rain on `FX-PLATES` real-plate equivalents at site (consented staff vehicles), confidence score exposed, supported plate formats of the market incl. Arabic script, time-synced, signed firmware | ≥ 500 passes over 7 days incl. night; compare to manual ground truth | Read log, ground truth sheet | REAL-RB |
| AT-HW.2 | LPR server / VMS adapter | Event delivery ≤ 500 ms on LAN; duplicate suppression; buffering during WAN loss; authenticated API; no retention beyond policy | Event capture, outage test | Logs | REAL-RB |
| AT-HW.3 | Gate / barrier controller | Command → open ≤ 1 s; fail-safe mode documented (fail-open/closed per fire code); manual override key/button logged; loop detector prevents closing on vehicle | Functional + safety test with vendor present | Test sheet signed by vendor & security_officer | REAL-RB |
| AT-HW.4 | Receiving scales | Legal-for-trade certification where required by jurisdiction; calibration certificate valid; tolerance per class; integration returns stable weight with unit; tamper seal | Test weights at 3 points; integration read | Calibration cert, readings | REAL |
| AT-HW.5 | Barcode/QR scanners (handheld & fixed) | Reads GS1-128/DataMatrix/QR incl. damaged labels ≥ 99 % first-pass on sample set; keyboard-wedge vs SDK mode fixed; duplicate scan handling in app | 1,000-scan sample | Scan log | REAL |
| AT-HW.6 | Temperature probes / cold-chain loggers | Accuracy ±0.5 °C in food range (assumption) with calibration certificate; logging interval configurable; offline buffer; alert on breach | Ice-point/reference check; breach simulation | Cert, readings | REAL |
| AT-HW.7 | BMS / fire / alarm gateway | **Read-only** integration via approved gateway (e.g. BACnet/Modbus/vendor API); point list mapped; alarm latency ≤ 10 s; life-safety system remains independently compliant; loss of integration alerts | Point-to-point test with vendor; alarm simulation in test mode | Point list sign-off | REAL-RB where enabled |
| AT-HW.8 | Meters (electricity/water/gas sub-meters, data loggers) | Interval data completeness ≥ 99 % over 14 days; clock sync; meter ids match utility accounts | Data capture review | Coverage report | REAL |
| AT-HW.9 | Printers (receipt, kitchen, label, A4 invoice) | Arabic shaping correct; kitchen printer failover to KDS/other printer; paper-out alert; fiscal/tax-invoice requirements per rule pack if any | Print suite | Printouts | REAL |
| AT-HW.10 | Payment terminals | PSP-certified model and integration mode (semi-integrated / P2PE preferred so PAN never touches PMS); offline behavior per PSP contract; receipt; refund; tip adjust if allowed | PSP certification script + site test | PSP certification letter | REAL-RB |
| AT-HW.11 | Staff mobile devices & tablets; kiosk (if any) | MDM enrolled; OS versions supported; offline storage encrypted; camera quality for ID/photos | Device checklist | MDM report | REAL |
| AT-HW.12 | On-prem server, UPS, network | Sizing per AT-PERF.8; UPS runtime ≥ 30 min with graceful shutdown; VLAN segmentation for devices/guest/staff; time source | Load + power-pull + network scan | Reports | REAL (on-prem profile) |
| AT-HW.13 | External partners (PSP, channel manager, bank/WPS, bill-provider, messaging, e-sign, travel providers, government routes) | Contract signed; sandbox tests green; production certification or authorized onboarding; named technical contact; reconciliation file tested | Partner checklist per `docs/05` | Certification/onboarding evidence | REAL-RB per required partner |

---

## 14. Release rollback strategy (application releases)

| Topic | Strategy |
|---|---|
| Deployment | SaaS: blue/green or canary per service with automatic rollback on SLO breach (error rate, latency, outbox lag). On-prem: versioned bundle with pre-upgrade snapshot (DB + object store) and one-command revert to previous bundle. |
| Database changes | **Expand → migrate → contract**: additive schema changes ship first; code works with old and new schema (N and N−1); destructive "contract" step only after the release is stable and backups verified. Never roll back by restoring a production database over live transactions. |
| Ledgers | Never rolled back. Any bad posting from a faulty release is corrected with **reversing entries** generated by an audited correction job (P.3), with a release-incident reference. |
| Feature flags | New behavior behind property-scoped flags; rollback = flag off; flags for money/stock/external calls require step-up approval to change. |
| Integrations | Adapter versions pinned per partner; new adapter version runs in shadow/sandbox before switch; rollback = pin previous version; inbound webhooks accepted by both versions during transition. |
| Mobile apps | Server supports app versions N and N−1; minimum-version gate for security fixes; staff app offline queue format versioned so older queued items still sync after a server rollback. |
| Events/contracts | Event schemas versioned; consumers accept N and N−1; no breaking contract change without deprecation window (M33). |
| Rule packs & mapping rules | Versioned with effective dates; rollback = activate previous version for future dates; past periods never re-posted automatically. |
| Decision & communication | Rollback authority: on-call engineering lead + duty manager for property-affecting incidents; status page and in-app banner; post-incident review with AT-RB evidence. |

| ID | Preconditions / fixtures | Steps | Expected result | Evidence | Pass criteria | Phase | Mode |
|---|---|---|---|---|---|---|---|
| AT-RB.1 | Release N+1 with additive migration | Deploy, generate traffic, roll back code to N | N runs on expanded schema; no data loss | Deploy log | 0 errors attributable to schema | 2 | SIM |
| AT-RB.2 | Faulty release posts wrong tax on 20 folios | Detect; run correction job | Reversals + correct postings linked to incident; reports restated | Journals | TB balanced; audit linked | 2 | SIM |
| AT-RB.3 | On-prem upgrade fails mid-way | Revert bundle from snapshot | Hotel back on N within 30 min (assumption) | Drill | Within target | 3 | onprem-lab |
| AT-RB.4 | Mobile app N−1 with queued offline items; server rolled back | Sync | Items processed once | Sync log | Pass | 3 | SIM |
| AT-RB.5 | Feature flag rollback for a money-moving feature | Toggle off with step-up | In-flight operations complete or are safely held; no partial state | Logs | Pass | 2 | SIM |

---

## 15. Phase 1 → Phase 2 planning-gate verification checklist (Section M)

Run by the Phase 1 lead before any production code; results recorded in `docs/13`. Status column starts `open` and is updated only with evidence.

| # | Check (Section M / P.7) | Verification method | Evidence location | Owner | Status |
|---|---|---|---|---|---|
| PG-01 | Count all catalogue entries; M01–M68 all present | Script counts module headers in `docs/01` + `01-catalogue/*` | Count report in `docs/13` | Product analyst | open |
| PG-02 | Each module has ≥1 named feature and ≥1 numbered subfeature | Script: every `Mnn` has `Fnn.k` and `SFnn.k.j` | Same | Product analyst | open |
| PG-03 | Every subfeature has phase (2–8) and `release: R1|Later` | Schema lint over Section-L blocks/tables | Lint output | Product analyst | open |
| PG-04 | Every subfeature has actor, screen, data, API, events, failure cases, test | Schema lint | Lint output | Product analyst | open |
| PG-05 | External dependency status recorded with honesty label (`docs/README` §3.6) | Cross-check `docs/05` integrations and `docs/08` §11 blocker register | Register | Integration lead | open |
| PG-06 | Financial posting and report effect stated for every money/stock subfeature | Lint `finance_report_effect`; cross-check `docs/06` §4 event map covers every financial event type | Coverage table | Financial controller | open |
| PG-07 | Bilingual/accessibility needs stated | Lint `i18n_a11y`; AT-A11Y suite planned | Lint output | UX lead | open |
| PG-08 | User's additional modules each have explicit traceability link (utilities, gas cylinders/pipeline, salaries, maintenance requisitions, bill providers, points wallet, network-marketing legal gate, website/marketing/CRM/reputation/revenue, optional spa/pool/retail, laundry/linen/minibar, service recovery, shift SOPs, safety/quality, cash control, site resilience, owner/brand, sustainability, five-market classifier, chef continuity, vendor apps/daily stock, RFQ samples/weighted award/PO, AI follow-up/receiving/stock/waste, vendor registration/search, cruise/flight/taxi, Canada/SIN/tax/government exchange, media/AI enhancement, AI agent, ID/e-sign/OTP, incident response, lost-and-found) | Traceability index row per item → module/feature/subfeature/phase/test | `docs/13` traceability index | Product analyst | open |
| PG-09 | Every Section O question has a traceability row | `docs/10` columns complete | `docs/10` | Product analyst | open |
| PG-10 | Section H documents 00–09 plus 10–13 exist | File presence + non-empty sections | Repo listing | Phase 1 lead | open |
| PG-11 | Consistency: phase plan (B) vs catalogue vs K/Q breakdown vs UI companion vs acceptance suite | Cross-reference script: every AT id cited in any doc exists in `docs/09`; every SF in K/Q exists in `docs/01`; screen ids exist in `docs/04` | Consistency report (incl. §4 id reconciliation table) | Phase 1 lead | open |
| PG-12 | Each AT-G01…G20 case has phase, mode and `-RB` classification; per-property release-blocker matrix derivable | This document §4; `docs/00` per-property matrix | This doc | QA lead | open |
| PG-13 | Fixture catalogue complete; all rule packs labelled TEST-ONLY | Review §3 | This doc | QA lead + compliance_officer | open |
| PG-14 | Where proof from provider or counsel is missing, it is stated with gate and owner | Review `docs/07` register and `docs/08` §11 | Registers | Compliance officer | open |
| PG-15 | Unresolved decisions have owner, interim assumption and fallback | Decision log in `docs/13` (incl. D-801…D-819, D-901…D-907) | `docs/13` | Phase 1 lead | open |
| PG-16 | No production code in repository | Repo inspection | Commit list | Phase 1 lead | open |
| PG-17 | Estimates, staffing, cost, critical path present | `docs/00`, `docs/08` | Docs | Delivery lead | open |
| PG-18 | Blueprint ingestion status (D-001) recorded | `docs/README` §1 | README | Product Owner | open |

---

## 16. Summary and decisions

### 16.1 Test case counts (this document)

| Suite | Cases |
|---|---|
| AT-G01 … AT-G20 (Release 1 integrated scenario) | 146 (incl. AT-G07.4–.12 specified in §5) |
| AT-G21 … AT-G25 (Later) | 18 |
| AT-REF (extended referral) | 10 |
| AT-PERF | 8 |
| AT-SEC | 18 |
| AT-PRIV | 10 |
| AT-DR | 8 |
| AT-OFF | 10 |
| AT-A11Y | 10 |
| AT-MIG | 10 |
| AT-HW | 13 |
| AT-RB | 5 |
| **Total** | **266** |

### 16.2 Decisions and assumptions raised here

| Id | Decision / assumption | Interim | Owner |
|---|---|---|---|
| D-901 | Evidence bundle retention | Life of R1 + 7 years, subject to jurisdiction rules | QA lead + compliance_officer |
| D-902 | Performance targets | A-PERF-01…08 | Pilot GM + it_admin |
| D-903 | RPO/RTO targets per profile | §9 table | Pilot GM + it_admin |
| D-904 | Pilot legacy systems and export formats | Unknown; generic plan §12 | Product Owner |
| D-905 | Dual-run length for PMS | 3–7 nights | financial_controller + front_office_manager |
| D-906 | Migration rollback trigger thresholds | > 2 h no check-in/payment, or unreconcilable guest-ledger difference | GM |
| D-907 | Hardware acceptance thresholds | §13 assumption values | chief_engineer + security_officer + vendors |
| A-OPS-01 | BEO propagation time | ≤ 5 min | catering_manager |
