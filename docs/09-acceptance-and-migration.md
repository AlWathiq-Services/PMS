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

