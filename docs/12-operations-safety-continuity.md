# 12 — Operations, safety and business continuity

**Pack:** MetriStay Hospitality Suite — Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Governing source:** master prompt v3.0 Sections C/Q (M42, M43, M47, M56–M64, M67, M68), K (M42, M43, M47, M50), G14/G15/G18/G20, O and P. Conventions: `docs/README.md` §3.
**Status:** specification only. Every SLA, timer, RTO/RPO, frequency and threshold in this file is a **configurable default labelled `assumption`** that the pilot hotel confirms or replaces (Section P.6). No food-safety temperature, holding time, inspection frequency, retention period or legal limit is stated as law: those values come **only** from a verified M44 jurisdiction rule pack or from the hotel's own documented food-safety plan, and are labelled by source (§9). Life-safety systems (fire alarm, sprinklers, emergency lighting, gas detection) remain independently certified and operable; MetriStay receives signals from them where approved but never replaces them (SF42.1.5).

**Modules:** M56 housekeeping/laundry/linen/minibar • M57 restaurant/room service/food assurance • M47 chef continuity • M14 stock • M50 receiving/stores • M61 hygiene/inspections • M42 incidents • M43 lost-and-found • M59 transport/fleet • M60 revenue protection • M62 staff enablement • M63 workflow automation • M64 devices/network/cyber • M68 risk/insurance/continuity • M67 sustainability.

**Shared mechanics.** Every escalation in this file is an M63 workflow (`task` with owner, SLA timer, escalation chain, pause/cancel with reason — SF63.1.2–SF63.1.5), versioned and dry-run before activation (SF63.2.2). Every manual fallback produces a record that is back-entered into the same source of truth with `capture_mode=manual`, original paper reference and the back-entering user, then reconciled (SF64.2.2).

---

## 1. Operating model principles

1. **One authoritative status per object** (room, lot, task, incident, vehicle) — departments act on it; they do not keep side lists.
2. **Shift = unit of accountability.** Each department shift opens with a checklist, carries a handover (SF62.2.1) and closes with open-item transfer; nothing is "owned by nobody" at shift change (M63 auto-reassigns to the incoming shift lead).
3. **Evidence at the point of work** — photo, scan, scale or sensor reading captured by the staff mobile app (offline-capable), not re-typed later.
4. **Exceptions over reports** — staff home screens show 3–7 current exceptions (Section P.1); managers see SLA breaches first.
5. **Manual first-class path** — every automated or device-dependent step has a documented manual route with an accountable owner (§5).
6. **Safety beats speed** — no workflow allows a food, fire, gas, pool or vehicle safety check to be skipped to meet an SLA; the SLA breaches visibly instead.

Assumed shift pattern for defaults (`assumption`): Morning 07:00–15:00, Evening 15:00–23:00, Night 23:00–07:00 hotel local time; kitchen split shifts per meal period; business-date rollover at night audit.

---

## 2. Daily and shift operating rhythms

Times are hotel local defaults (`assumption`). "System" lines are automated M63 triggers.

### 2.1 Housekeeping, laundry, linen and minibar (M56)

| When | Who | What | Subfeatures / evidence |
|---|---|---|---|
| Night (≈05:30) | System | Build day board: departures, arrivals with ETA/VIP/accessibility, stay-overs, DND carry-over, OOO; compute priority and task durations | SF56.1.1, SF56.1.2 |
| Shift open 07:00 | housekeeping_supervisor | Briefing; assign by capacity; linen par check against arrivals; confirm laundry pickup/return schedule | SF56.1.2, SF56.2.1; SF62.2.1 handover read |
| 07:30–14:30 | housekeeper | Clean, report faults (→ M26), found items (→ M43), minibar count and post once | SF56.1.4, SF56.2.4 |
| Rolling | housekeeping_supervisor | Inspect; reclean; release to front desk; update room-ready ETA | SF56.1.3, SF56.1.5 |
| 10:00 & 14:00 | System | DND check: rooms DND beyond threshold (default: no service for 24 h / 2nd day, `assumption`) → welfare-check task to duty manager (two-person, logged) | SF56.1.1; M42 link if concern |
| Laundry pickup/return | laundry_attendant + vendor | Soiled count/weight out; clean count/weight in; stain/damage quarantine; custody signature | SF56.2.2, SF56.2.3 |
| Shift close | housekeeping_supervisor | Unfinished rooms handed over; linen/amenity variance; minibar disputes | SF56.2.5, SF56.2.6 |
| Weekly | housekeeping_supervisor | Linen stocktake vs par; loss/replacement request (→ M49) | SF56.2.1, SF56.2.3 |
| Monthly | financial_controller | Laundry cost per occupied room, linen loss, amenity cost allocation | SF56.2.5; M32 |

### 2.2 Kitchen, room service and food assurance (M57, M47, M14, M50)

| When | Who | What | Subfeatures / evidence |
|---|---|---|---|
| D-1 afternoon | executive_chef | Confirm tomorrow's coverage (primary/backup per meal period/outlet/event); open gaps trigger callout | SF47.1.1, SF47.1.3 |
| D-1 | System | Production plan from covers, BEOs (with allergens/diets), room-service forecast; ingredient reservations; FEFO pick lists | SF57.2.1, SF50.3.3, SF50.3.8 |
| Shift open | shift_chef | Pre-service checklist from active rule pack / hotel food-safety plan (equipment, holding units, sanitation, pest signs); staff fitness-to-work declaration where policy requires | SF61.1.1, SF57.2.2 |
| Before each service | shift_chef | Allergen briefing; BEO version check; kitchen acknowledgment of guest allergy/diet notes | SF57.1.4, SF57.2.3 |
| During service | kitchen staff | Batch-linked preparation (lot capture by scan), hold/cool/reheat checks per plan, ticket timing | SF57.2.1, SF57.2.2 |
| Receiving windows | receiver | Low-touch receipt with food verification (temperature/condition/expiry), quarantine exceptions | SF50.2.3–SF50.2.6 |
| After service | shift_chef | Waste/returned portions recorded; intact unused stock return with inspection; discarded to waste (never sellable) | SF50.3.4, SF50.3.5, SF57.2.5 |
| Shift close | shift_chef | Cleaning log, temperature log completeness, handover of open issues | SF61.1.3, SF62.2.1 |
| Daily | storekeeper | Expiry sweep, quarantine review, recall-hold list | SF50.3.8 |
| Weekly | executive_chef + fnb_manager | Recipe theory vs actual, food cost, waste by reason | SF50.3.7, SF57.2.5 |
| Monthly | compliance_officer | Food-safety plan review, rule-pack change alerts, mock trace (one lot forward/back) | SF44.2.7, SF57.2.4 |

### 2.3 Stores and receiving (M50, M14)

Daily: ASN review and dock schedule; receiving; issue to departments; returns and quarantine decisions (within 24 h default, `assumption`); duplicate-scan review. Weekly: cycle counts (blind count, M14), stock age review. Monthly: full count of high-value/controlled items, shrinkage report.

### 2.4 Engineering and maintenance (M26, feeding M61/M64/M67)

Daily: overnight fault review, OOO room plan with front office, planned-maintenance tasks, meter read or feed-gap check (M22–M24), gas cylinder custody check (M25). Weekly: life-safety system logs reviewed (records of certified provider tests, not MetriStay tests), pool/water plant checks per plan, contractor schedule and briefings (SF62.1.4). Monthly: asset criticality review, utility anomalies to work orders (SF67.2.2).

### 2.5 Security, incidents and lost-and-found (M42, M43)

Each shift: patrol checklist; incident log review; key/master-key custody; CCTV health (M64); lost-and-found intake log, sealed storage check. Daily: open incidents review with duty manager; claims queue. Weekly: storage deadline sweep (disposal/donation per rule pack, SF43.1.7). Quarterly: drill (§8).

### 2.6 Transport and fleet (M59)

Day before: manifest from arrivals/departures with flight/ship numbers and accessibility needs; driver/vehicle assignment; permit/insurance/vehicle validity check (SF59.2.1). Each trip: driver acknowledgment, live status, passenger handover. Daily: missed pickups review, fuel/labor capture. Monthly: third-party settlement vs owned-fleet cost (SF59.2.4).

### 2.7 Front office cash and night audit (M60)

Each cashier shift: till opening float, drops, refunds/comps with approval, closing count (SF60.1.1–SF60.1.3). Night: night audit with difference queue (SF60.1.4), business-date roll. Daily: integrity alerts review — duplicates, void/discount outliers, commission mismatches (SF60.2.1–SF60.2.3). Monthly: false-positive feedback and access-log review (SF60.2.6).

### 2.8 Staff enablement (M62) and workflow ops (M63)

Each shift: handover read/acknowledged; coverage vs labor forecast (SF62.2.2). Weekly: certification expiries (30/14/7-day reminders, `assumption`) and auto-revocation on lapse (SF62.1.5); quality sampling (SF62.2.3). Monthly: SOP/training attestations; workflow performance (SLA breach heat map, SF63.2.5).

### 2.9 Technology (M64)

Daily: health board, backup job success, certificate expiry, dead letters. Weekly: patch window review, device firmware report. Monthly: restore test to isolated environment (SF64.2.1), access review. Quarterly: DR failover exercise (§8).

### 2.10 Sustainability (M67) and risk (M68)

Monthly: utility and waste data quality (missing/estimated/verified), intensity per occupied room/guest/cover (SF67.1.2–SF67.1.4). Quarterly: target review, anomaly-to-work-order closure. Annually: insurance renewal, business-impact assessment refresh, crisis contact tree verification (SF68.1.1, SF68.2.1–SF68.2.2).

---

## 3. Department operating models (controls and evidence)

### 3.1 Housekeeping, laundry, linen, minibar (M56)
- **Room state vs cleaning state** are separate: sold/occupied/vacant (M03/M05) vs dirty/clean/inspected/OOO (M56). Release to sale requires `inspected` (or `clean` where property policy allows without inspection) and no open room-blocking fault (SF56.1.4).
- **DND and privacy:** DND suppresses service but not welfare; welfare checks are two-person, logged, with duty manager authorization (§4.1).
- **Linen custody ledger:** SKU by type/size, states `clean_store → floor → in_room → soiled → vendor → returned_clean | rejected_stained | lost`; each transfer has count or weight and accepting person (SF56.2.1–SF56.2.2). Laundry vendor mismatch beyond tolerance (default 2% by count, `assumption`) → vendor claim.
- **Minibar/amenities:** count-based posting with unique source key; late-found consumption after checkout posts to folio only with guest notice per policy; disputes reverse once (SF56.2.4, SF56.2.6). Consumption decrements M14 stock once.
- **Offline:** staff app queues status changes; server resolves conflicts (latest valid transition wins for cleaning state; room release requires online confirmation to avoid releasing an OOO room) (SF56.1.5).

### 3.2 Food assurance and kitchen continuity (M57, M47, M14, M50)
- Production is **batch-linked**: each production batch records input lots (scan) and output portions/destinations (outlet, event, room-service orders) (SF57.2.1). Where lot capture is impossible (e.g. bulk staples), the record names the store lot range and marks trace precision "coarse".
- Allergen control: recipe allergen profile (M14) → menu/BEO flags → guest declaration → kitchen acknowledgment → substitution requires allergen impact review (SF57.1.4, SF57.2.3).
- **Chef continuity:** coverage slots per meal service/outlet/event; no-show thresholds (default: not clocked-in 30 min before service start, `assumption`) trigger primary→backup→emergency roster callout with atomic single acceptance; food-handler credentials must be valid before assignment; if no chef by cutoff, manager is alerted with menu-contingency options (reduced menu, catering partner, guest notification) (SF47.1.3–SF47.2.7, G15).
- **Stock integrity:** INV-STK-1/2 (defined in `docs/01-catalogue/02` §0.4) — discarded, served, plated or temperature-breached items are terminal waste.

### 3.3 Hygiene and inspections (M61)
Checklists are **templates bound to a rule pack version or a hotel policy version** (SF61.1.1). Each check has a responsible credentialed person (SF61.1.2), evidence mode (sensor/manual/photo) (SF61.1.3), and on failure creates a nonconformance with severity: *critical* → service stoppage/quarantine/room block/recall hold (SF61.2.1–SF61.2.2); *major* → corrective action within SLA; *minor* → next-shift fix. Release after critical requires independent retest by someone other than the fixer (SF61.2.4). Permits/inspections calendar escalates expiry (SF61.1.4). Inspector-ready export bundles checks, nonconformances and closures (SF61.1.5).

### 3.4 Incidents (M42) and lost-and-found (M43)
- Intake from staff/guest/security and approved sensor adapters; dedupe; operator confirmation before paging for non-life-safety sensor alerts; **life-safety alarms are never gated on AI or confirmation** — the building's certified alarm system acts independently; MetriStay notifies in parallel (SF42.1.2–SF42.1.5).
- Playbooks by type (fire, medical, security, gas, flood, outage) with acknowledgment timers (§4.3), guest-welfare roster and room/asset impact (SF42.2.2–SF42.2.5). Chronology is append-only; evidence pointers to CCTV are access-restricted (SF42.2.6).
- Lost-and-found: unique ID, sealed bag, custody transfers, privacy-limited matching (staff search shows category/location/date, not full descriptions, until claim verification), release with identity evidence, shipping at owner's cost where policy, disposal/donation after rule-pack or hotel-policy retention (SF43.1.1–SF43.1.8). Valuables/ID documents/cash follow a higher-custody path (two-person, safe).

### 3.5 Transport and fleet (M59)
Owned-fleet trips require valid driver licence/permit, vehicle insurance and inspection on the trip date (SF59.2.1); external taxi via M45/M46 approved providers. Passenger data minimized to name, pickup, contact for the trip (SF59.2.2). Accessible vehicle requests are matched or escalated before confirming to the guest (SF59.1.3). Missed pickup → rescue workflow (§4.5). Incidents create M42 records and evidence (SF59.2.5).

### 3.6 Revenue protection (M60)
Maker-checker on refunds, comps, voids above thresholds; cash drop and safe custody; night audit difference queue; anomaly cases with **employee-privacy controls** — investigations restricted to authorized roles, evidence access logged, false-positive feedback improves rules (SF60.2.5–SF60.2.6). No automated disciplinary action.

### 3.7 Staff enablement (M62)
Role/SOP/language matrix (EN/AR at launch, more per property); onboarding and certification expiry with automatic revocation of assignments that require the credential (e.g. food handler, pool operator, driver) (SF62.1.2, SF62.1.5); contractor site briefing before site access (SF62.1.4); shift handover (SF62.2.1); sampled quality reviews with confidentiality and dispute/correction (SF62.2.3–SF62.2.4).

### 3.8 Workflow automation (M63)
Only roles with `workflow_designer` scope may configure; templates are versioned, dry-run in a simulator against recorded events, and activated with approval (SF63.2.1–SF63.2.2). Guest-sensitive fields are minimized in task payloads (SF63.2.3). Rollback to prior version is one action (SF63.2.4). Retries are idempotent and deduplicated (SF63.1.3). A workflow **cannot** disable a safety check, approve money movement, or change a rule pack.

### 3.9 Devices, network and cyber (M64)
Device registry with firmware/certificate inventory and supported connector versions (SF64.1.1); network segmentation — guest Wi-Fi, staff, payment terminals, building systems/OT and servers on separate segments with connector-specific permissions (SF64.1.2); health and degraded-service queue (SF64.1.3); vendor maintenance windows (SF64.1.4); tested manual alternatives (SF64.1.5); time sync via NTP with drift alarms (SF64.1.6). Recovery: encrypted backups and independent restore tests (SF64.2.1), offline conflict policy (SF64.2.2), cyber isolation and log preservation (SF64.2.3), measured recovery (SF64.2.4), change approval/rollback (SF64.2.5), status page and support ownership (SF64.2.6).

### 3.10 Risk, insurance, continuity (M68) and sustainability (M67)
Insurance register with insured assets, coverage, expiry and premium accruals (SF68.1.1–SF68.1.2); incident evidence packets for claims with adjuster scoped access (SF68.1.3–SF68.1.4); deductible/reserve/settlement report (SF68.1.5). Continuity plans per §6 with crisis role tree (SF68.2.2). Sustainability: measured values labelled missing/estimated/verified; emission factors only with a reviewed source/version; **no unverified green claims** in any guest-facing text (SF67.1.5, SF67.2.4).

---

## 4. Escalation matrices (SLA defaults — all `assumption`, configurable per property)

Columns: trigger → first owner and acknowledgment SLA → escalation if not acknowledged/resolved → final escalation → evidence. Notification channels default to staff app push + SMS for L2/L3; voice call for life-safety.

### 4.1 Housekeeping / laundry / minibar

| Trigger | L1 owner (ack / resolve) | L2 after | L3 after | Evidence |
|---|---|---|---|---|
| Arrival room not ready at guest ETA − 60 min | housekeeping_supervisor (10 min / by ETA) | front_office_manager at ETA − 30 | duty_manager at ETA | Room-state history |
| Inspection failed | housekeeping_supervisor (15 / 45 min reclean) | housekeeping_supervisor (lead) 60 min | duty_manager 120 min | Inspection photos |
| DND beyond threshold / no-response | duty_manager (30 min) | security_officer accompanies welfare check | gm | Welfare-check log (two names) |
| Linen par below arrivals need | housekeeping_supervisor (30 min) | procurement_officer / laundry vendor 2 h | gm 4 h | Par report |
| Laundry return mismatch > tolerance | laundry_attendant (at receipt) | housekeeping_supervisor same day | procurement_officer vendor claim 48 h | Count/weight record |
| Minibar charge dispute | front_desk_agent (at checkout) | front_office_manager 30 min | guest_relations case | Count record, posting id |

### 4.2 Kitchen / food assurance

| Trigger | L1 | L2 | L3 | Evidence |
|---|---|---|---|---|
| Chef no-show (not clocked in at service − 30 min) | backup_chef auto-called (10 min to accept) | emergency roster callout (ordered sequence, 10 min each) | fnb_manager + gm; menu contingency at service − 0 | Callout attempts, acceptance lock |
| Critical control check failed (per plan) | shift_chef immediate: stop/hold product | executive_chef 15 min | compliance_officer + gm 60 min | Check record, quarantine id |
| Temperature sensor out of range (holding/cold store) | shift_chef/storekeeper (15 min) | chief_engineer (equipment) 30 min | executive_chef decision on product 60 min | Sensor series, product disposition |
| Guest allergic reaction / suspected foodborne illness | duty_manager immediate (S1, M42 medical playbook) | executive_chef + compliance_officer 15 min (lot trace, recall hold) | gm; authority notification per rule pack | Incident chronology, lot trace |
| Supplier recall notice | storekeeper (30 min: hold lots) | executive_chef 1 h (menus/events impacted) | compliance_officer; guest notification decision | Recall case, affected lots/events |
| Delivery late for event-critical items | procurement_officer (at ETA miss) | executive_chef alt sourcing 1 h | catering_manager guest/organizer notice | PO milestones |

### 4.3 Incidents / security / lost-and-found

| Trigger | L1 | L2 | L3 | Evidence |
|---|---|---|---|---|
| Fire/gas alarm signal (from approved panel interface) | security_officer + duty_manager **immediate** (ack 2 min) — building alarm acts independently | gm + chief_engineer 5 min; emergency services per local procedure | owner/crisis lead | Signal log, ack chain |
| Water leak/flood sensor | engineer (10 min) | chief_engineer 20 min | duty_manager; room moves | Sensor, photos, work order |
| Medical emergency | first-aider/security (immediate) | duty_manager 5 min; emergency services per local procedure | gm | Chronology |
| Security threat | security_officer immediate | duty_manager 5 min; police per local procedure | gm/owner | Restricted evidence |
| Lost item claim with ID/valuables | security_officer (24 h) | front_office_manager 48 h | gm dispute | Custody chain |

### 4.4 Engineering / devices

| Trigger | L1 | L2 | L3 | Evidence |
|---|---|---|---|---|
| Guest-impacting fault in occupied room | engineer (15 / 60 min) | chief_engineer 60 min; room move offer | duty_manager 120 min | Work order |
| Critical asset down (chiller, boiler, lift, kitchen gas) | chief_engineer (15 min) | approved contractor per SLA | gm | Asset log |
| Device/integration degraded (§5) | it_admin (15 min business hours / 30 min after hours) | vendor support per contract | gm if guest-facing > 60 min | Health log |
| Suspected cyber incident | it_admin immediate isolate | security lead/DPO 30 min; external IR retainer | gm/owner; regulator/breach notice per rule pack | Preserved logs |

### 4.5 Transport, revenue protection, staff

| Trigger | L1 | L2 | L3 | Evidence |
|---|---|---|---|---|
| Driver not acknowledged 30 min before pickup | concierge (10 min) | alternate driver / approved taxi | duty_manager | Trip log |
| Missed pickup / guest stranded | concierge immediate rescue | duty_manager | gm; recovery case (`docs/11` §9) | Trip log, case |
| Cash shift variance > threshold | front_office_manager (end of shift) | financial_controller next day | gm | Count sheets |
| Void/discount outlier | fnb_manager (daily) | financial_controller | revenue-protection case | POS log |
| Certification lapsed for assigned staff | department head (auto-unassign) | hr_officer 24 h | gm | Certification record |

---

## 5. Manual fallbacks for every device and integration outage

Each row: how the outage is detected, the manual route, who owns it, what is captured, and how it is reconciled. Paper/offline packs are printed/refreshed every night audit and at shift start (default, `assumption`) and stored at front desk, security and kitchen.

| Device / integration | Detection | Manual route | Owner | Capture | Resync / reconciliation |
|---|---|---|---|---|---|
| SaaS platform unreachable (internet or provider) | Staff app offline banner; heartbeat | Staff app offline mode for room status, tasks, POS queue; **offline pack**: arrivals/departures/in-house list, room status, folio balances, emergency contacts | duty_manager | Offline queue + paper forms numbered | Queue replay with idempotency; conflicts to exception queue |
| On-prem server failure | Health monitor | Failover to standby/restore (§7); same offline pack | it_admin | As above | Restore + replay |
| Internet link (site) | Router/ISP monitor | Secondary link (4G/5G) if provisioned; else offline mode | it_admin | — | Auto |
| Staff mobile devices | MDM | Shared spare devices; paper task sheets | department head | Paper sheet | Back-entry |
| POS terminals | Terminal health | Offline POS queue on device; paper checks with sequential numbers when device dead | fnb_manager | Paper check no. | Back-entry; duplicate check by check no. |
| Kitchen display (KDS) | Heartbeat | Printed tickets; runner | shift_chef | Printer tickets | none (operational) |
| Receipt/ticket printers | Error | Alternate printer; handwritten ticket | outlet lead | — | — |
| Card terminal / PSP | Payment failures, PSP status | Alternate terminal; pay-by-link later; guarantee by policy; **no manual card imprint storing PAN** | cashier | Pending-payment list | Pay-by-link / collect at checkout; AR follow-up |
| Channel manager / OTA link | ARI ack failures, dead letters | Revenue manager updates extranets directly for critical dates; tight stop-sell on uncertainty | revenue_manager | Change log | Full ARI refresh + reservation reconciliation on restore |
| Website / booking engine | Synthetic check | Banner "call/email to book"; assisted booking by phone/email | marketing_manager | Assisted bookings | Normal (same system when back) |
| Door lock encoder / lock system | Encoder error | Spare encoder; physical master under security custody; room change; locksmith | security_officer | Key log | Reissue keys; audit trail from lock system |
| LPR camera / server / gate | Observation gaps, low confidence | Manual lane review; attendant opens gate with reason code; paper/permit list | parking_attendant | Override log (SF64.1.5) | Session reconstruction; charges posted once |
| Fire/BMS interface to MetriStay | Heartbeat from interface | **Life-safety system operates independently**; security monitors panel directly; incidents logged manually | security_officer | Incident log | Back-entry of chronology |
| CCTV / VMS | Health | Increased patrols; note coverage gap | security_officer | Patrol log | Evidence gap noted in incidents |
| PBX / phones | Line test | Mobile phones list; radios | it_admin | — | — |
| Guest Wi-Fi | Monitor | Notice; front-desk help | it_admin | — | — |
| SMS / WhatsApp provider | Delivery failures | Email; in-person OTP alternative at desk (SF41.2.7) | integration_admin | — | Resend queue |
| Email service | Bounce/queue | SMS where consented; desk | integration_admin | — | Resend |
| E-signature provider | Error | Paper registration card signed, scanned, hashed | front_desk_agent | Scan | Link to reservation |
| ID OCR service | Error/timeout | Manual entry with document check by agent | front_desk_agent | Manual fields | none |
| Guest AI assistant / LLM provider | Error/budget cap | Contact form + phone; chat shows fallback | guest_relations | Inbox items | none |
| Receiving scanner / scale | Device error | Manual count/weight on paper GRN; photo of delivery note; **no stock posting until verified** | receiver | Paper GRN | Back-entry; duplicate-scan protection |
| Temperature sensors | Missing readings | Manual probe readings at plan frequency | shift_chef / storekeeper | Manual log | Mark data manual |
| Vendor mobile app / vendor API | Sync failures | Vendor web fallback; phone/email order with PO reference | procurement_officer | PO notes | Catalog stale badge |
| Bill-pay provider (M29) | Timeout/pending | **No blind retry**; inquiry; bank payment if pending unresolved by due date with approval | ap_clerk | Pending log | Provider receipt vs bank reconciliation |
| Bank / WPS channel | File rejection | Corrected resubmission; manual bank portal upload by authorized signatories | payroll_officer | Rejection file | Status reconciliation (SF27.3.7) |
| Government filing portal/API | Error/outage | Approved portal/file/manual path (SF38.3.8); deadline tracker | compliance_officer | Receipt | Attach receipt |
| Utility meter feeds / BMS data | Gaps | Manual meter read with photo | engineer | Photo + value | Gap flagged; bill reconciliation |
| Time clock | Offline | Supervisor-attested manual attendance | department head | Sheet | Payroll approval flags manual entries |
| Laundry vendor portal | Down | Paper custody form | laundry_attendant | Form | Back-entry |
| Taxi/transfer provider API | Error | Phone booking with confirmation reference; never "booked" without reference | concierge | Reference | Settlement match |
| Accounting export / external finance | Error | Retry; manual journal upload by controller | financial_controller | — | Balance check |

---

## 6. Business continuity scenarios

RTO = time to restore the named capability; RPO = maximum acceptable data loss. All values are **assumptions** to be confirmed in the Phase 1 business-impact assessment (SF68.2.1) and proven in drills (SF68.2.5). "Tier" refers to §7.

### 6.1 Fire

| Aspect | Plan |
|---|---|
| Trigger | Certified fire alarm activation (independent of MetriStay); MetriStay receives interface signal where approved |
| Immediate | Building evacuation per local fire plan; security/duty manager take crisis roles (SF68.2.2); emergency services per local procedure |
| MetriStay role | Print/offline **in-house list with room numbers, accessibility/assistance flags and guest count** for roll call (pre-generated offline pack, refreshed hourly on a local device, `assumption`); incident chronology; staff roll call from attendance |
| Guest welfare | Assisted-evacuation list (SF68.2.3); relocation to partner hotels; medication/ID retrieval coordination; communication templates |
| Systems | If server room affected: DR per §7; front desk switches to SaaS/standby or offline pack |
| Recovery | Room inventory OOO en bloc; reaccommodation bookings; insurance evidence packet (SF68.1.3) |
| RTO / RPO | Guest roll-call list: available at T+0 (offline); core PMS: Tier 1 |
| Drill | Semi-annual evacuation drill run by fire-safety responsible person (legal frequency per rule pack); MetriStay measures time to produce roll-call list and reconcile guests; tabletop for relocation annually |

### 6.2 Flood / water ingress

| Aspect | Plan |
|---|---|
| Trigger | Leak sensor, staff report, weather warning |
| Immediate | Isolate water/electric per engineering SOP; move guests; protect server/electrical rooms |
| MetriStay role | Room OOO batch with reason; relocation bookings; asset damage log with photos; vendor emergency call-off (M46) |
| Guest welfare | Room moves, belongings care, compensation per `docs/11` §9 |
| Recovery | Drying/remediation work orders; hygiene re-inspection before resale (SF61.2.4) |
| RTO / RPO | Room status accurate within 15 min of event (`assumption`); data Tier per §7 |
| Drill | Annual tabletop; quarterly leak-sensor test (where sensors exist) |

### 6.3 Cyber incident (ransomware, account compromise, data breach)

| Aspect | Plan |
|---|---|
| Trigger | EDR/SIEM alert, abnormal access, ransom note, partner notice |
| Immediate | Isolate affected segments/accounts (SF64.2.3); preserve logs; revoke tokens/keys; engage IR retainer; decide operation in offline mode |
| MetriStay role | Tenant/property isolation; immutable backups; restore to clean environment; forced credential rotation; audit export |
| Guest welfare | Continue service on offline packs; payment only via standalone terminals not on affected network |
| Notification | DPO assesses breach notification duties per market rule pack (M44 privacy); PSP/card-brand notification per PSP contract |
| Recovery | Restore from last clean backup; replay offline queue; reconcile payments/channel bookings |
| RTO / RPO | Tier 1 services from clean backup: RTO 4 h / RPO 15 min (SaaS), RTO 8 h / RPO 1 h (on-prem) — `assumption` |
| Drill | Semi-annual tabletop; annual restore-from-immutable-backup exercise with measured time |

### 6.4 Power failure

| Aspect | Plan |
|---|---|
| Trigger | Grid loss, generator start |
| Immediate | Life-safety on emergency power (certified systems); check lifts for trapped persons; kitchen gas and cold-chain status |
| MetriStay role | On-prem server/network on UPS + generator circuit (requirement for on-prem profile); staff devices battery; outage mode; cold-store temperature monitoring and food disposition decisions logged |
| Food safety | Cold-chain breach decisions per food-safety plan (§9); product quarantined until disposition |
| RTO / RPO | Front-desk operations continue on UPS ≥ 30 min (`assumption`) or offline pack |
| Drill | Quarterly generator test (engineering); annual MetriStay "power-off" functional test of offline mode |

### 6.5 Staff shortage (illness, strike, weather, visa delays)

| Aspect | Plan |
|---|---|
| Trigger | Absence above threshold (default > 20% of shift roster, `assumption`), chef no-show, housekeeping capacity below required room count |
| Immediate | Coverage gap view; callout (chefs via M47; other roles via on-call lists); cross-trained staff from M62 skills; approved agency vendors (M46) |
| Service reduction options | Stay-over service on request only; reduced menu; limited outlet hours; restrict arrivals/stop-sell high-effort rooms (revenue manager) |
| Guest communication | Templates with honest service levels |
| Safety | Never assign uncredentialed staff to food handling, pool, driving, electrical work |
| Drill | Annual tabletop; monthly chef-callout test on a low-risk shift (G15) |

### 6.6 Internet loss (site)

| Aspect | Plan |
|---|---|
| SaaS profile | Staff app offline mode; secondary cellular link if provisioned; offline packs; payment via standalone terminals with own connectivity; channel updates paused (ARI ack unknown → revenue manager decides stop-sell via extranet on mobile network) |
| On-prem profile | Core PMS continues on LAN; channel, PSP, messaging, bill-pay queue until link returns |
| RTO | Secondary link switchover ≤ 5 min where provisioned (`assumption`); otherwise ISP SLA |
| Drill | Quarterly: pull the WAN link for 30 min in a quiet period and measure queue replay |

### 6.7 PSP outage

| Aspect | Plan |
|---|---|
| Detection | Authorization error rate / PSP status |
| Actions | Switch to secondary PSP only if contracted and certified; else guarantee-by-policy, pay-by-link later, collect at checkout; booking engine may accept "pay at hotel" rates only if configured |
| Controls | No storage of card numbers on paper or in notes; no double capture on recovery (idempotent intents, status inquiry) |
| Reconciliation | Pending-payment list worked by cashier; settlement reconciliation next day |
| Drill | Semi-annual sandbox PSP outage simulation (G20) |

### 6.8 Channel outage (channel manager or OTA)

| Aspect | Plan |
|---|---|
| Detection | ARI ack timeouts; reservation feed silence beyond expected (default 60 min at > 50% occupancy, `assumption`) |
| Actions | Revenue manager updates key OTA extranets directly; stop-sell on uncertain inventory for near-term dates; direct website remains live |
| Recovery | Full ARI refresh; pull missed reservations; compare OTA extranet reservation list to M05; oversell → walk workflow |
| Drill | Semi-annual in sandbox; annual live "extranet fallback" tabletop |

### 6.9 Additional scenario: food contamination / recall (required by M57/M61)
Supplier or authority recall or suspected outbreak → recall case, lot hold, forward trace to menus/events/guests where feasible (SF50.2.10, SF57.2.4), authority liaison per rule pack, guest communication decision by GM with counsel. Drill: quarterly mock trace of one lot (target time agreed with pilot, `assumption`).

---

## 7. Recovery tiers, RTO/RPO assumptions

| Tier | Capabilities | SaaS RTO / RPO (`assumption`) | On-prem single-hotel RTO / RPO (`assumption`) | Offline continuity |
|---|---|---|---|---|
| 1 | Reservations/stays, room status, folio posting, POS, payments orchestration, incident log | 1 h / 5 min (regional failover) | 4 h / 15 min (warm standby or restore) | Staff app offline + packs |
| 2 | Housekeeping board, stores/receiving, procurement, guest messaging, website booking | 4 h / 15 min | 8 h / 1 h | Paper forms |
| 3 | Finance/GL, payroll, reporting, CRM campaigns, sustainability | 24 h / 1 h | 24 h / 4 h | Deferred |
| 4 | Analytics exports, AI enhancement queues, archives | 72 h / 24 h | 72 h / 24 h | Deferred |

Backup requirements: encrypted, immutable/offsite copy (on-prem → offsite encrypted per README §3.7), daily full + continuous WAL/log shipping for Tier 1–2; restore tests monthly to an isolated environment with measured time recorded as RTO evidence (SF64.2.1, SF64.2.4).

---

## 8. Drill plan

| Drill | Frequency (`assumption`) | Type | Participants | Success criteria (measured, SF68.2.5) |
|---|---|---|---|---|
| Evacuation roll-call list | Semi-annual (with hotel fire drill) | Functional | security, front desk, duty manager | List produced from offline pack; all in-house guests and on-duty staff accounted; time recorded |
| Restore from backup | Monthly | Technical | it_admin | Restore completes within tier RTO; data within RPO; checksum/row-count reconciliation |
| DR failover (SaaS region or on-prem standby) | Annual | Technical + functional | it_admin, front desk | Front-desk flows run on recovered system; offline queue replayed without duplicates |
| Internet loss | Quarterly (30 min) | Functional | front desk, F&B, it_admin | Offline transactions replayed; zero duplicates; conflicts resolved |
| PSP outage | Semi-annual (sandbox) | Functional | cashier, finance | No double capture; pending list cleared |
| Channel outage | Semi-annual (sandbox) + annual tabletop | Functional | revenue manager | Extranet fallback executed; no oversell or oversell walked per policy |
| Chef callout | Monthly (low-risk shift) | Functional | kitchen, fnb_manager | Acceptance within target; credentials checked; single assignment |
| Mock recall trace | Quarterly | Functional | storekeeper, chef, compliance | Lot traced forward/back; recall hold effective |
| Cyber tabletop | Semi-annual | Tabletop | gm, it_admin, dpo, finance | Decisions logged; notification decisions per rule pack |
| Flood/power tabletop | Annual | Tabletop | engineering, duty managers | Roles and contacts verified; gaps to corrective work orders |
| Staff shortage tabletop | Annual | Tabletop | department heads, HR | Service-reduction playbook agreed |

Every drill produces: participants, scenario, start/end timestamps, measured recovery time, deviations, corrective actions with owners (M63 tasks) and closure. Results feed M68 evidence and insurance renewals.

---

## 9. Food safety and traceability rules (jurisdiction registry driven)

### 9.1 Source of every numeric control
| Value type | Allowed sources | Label shown | Not allowed |
|---|---|---|---|
| Temperatures (cold/hot holding, cooling, reheating, delivery acceptance) | (a) M44 rule pack for the property's jurisdiction with status `verified`; (b) the hotel's documented food-safety plan (HACCP-style) approved by the named responsible person | `legal: <rule pack id vX>` or `hotel policy: <plan id vY>` | Hard-coded defaults presented as law; values copied from another country's pack |
| Holding/shelf-life times, date marking | Same as above; supplier specification for product-specific shelf life | as above + `supplier spec` | Guessing |
| Record retention (temperature logs, trace records) | Rule pack; else hotel policy with counsel review flag | as above | Deleting before rule-pack period; sample-photo 90-day rule (SF49.1.4) does **not** apply to trace records |
| Inspection/check frequency | Rule pack or plan | as above | — |
| Allergen list to declare | Rule pack (market list) + hotel policy additions | as above | Silent omission |
| Food-handler credential requirements | Rule pack | as above | Assigning uncredentialed staff |

If a property's food rule pack is `draft`, `expired` or `unknown`, checklists still run from the hotel plan, but the compliance dashboard shows **"legal basis unverified"** for that property and inspector-ready exports carry that label (SF44.2.4, SF44.2.8). No automated authority submission occurs.

### 9.2 Candidate authorities to validate (unverified — for M44 rule-pack research, not legal advice)
Canada: CFIA (federal traceability guidance, cited in Section K) plus provincial/municipal public-health food premises rules. Oman, Pakistan (provincial food authorities), Saudi Arabia and Portugal: competent national/municipal food-safety authorities to be identified and cited by the compliance workstream (`docs/07` validation register). Each entry needs source URL, effective date, reviewer and counsel/partner validation before status `verified`.

### 9.3 Traceability rules (system invariants)
1. **One step back, one step forward:** every received food lot records supplier, product, lot/batch, quantity, expiry/production date where supplied, receipt time, temperature/condition evidence (SF50.2.1–SF50.2.4); every issue and production batch records the lots consumed and the destinations (outlet/event/room-service order) (SF50.3.2, SF57.2.1).
2. Use GS1 identifiers when suppliers provide them; otherwise internal lot IDs with supplier reference.
3. **No negative lots, no silent restoration:** quantities never go negative; discarded/served/plated/temperature-breached items are terminal waste (INV-STK-2).
4. **Quarantine by default on doubt:** short/over/damaged/substituted/temperature-breach receipts go to restricted quarantine (SF50.2.6); release requires authorized inspection of intact, never-issued, in-date stock.
5. **Recall hold is immediate and global in the property:** a recall case blocks issue, sale and production of affected lots across all stores and outlets (SF50.3.8, SF57.2.4, SF61.2.2).
6. **Allergen chain:** recipe allergen profile → menu/BEO → order/guest declaration → kitchen acknowledgment; substitution recalculates allergens and requires review (SF57.1.4, SF57.2.3).
7. **Emergency chefs** cannot be assigned without valid food-handling credentials and a handover packet containing BEO, allergens and production plan (SF47.1.6, SF47.2.5).
8. **Evidence retention** for trace and temperature records follows the rule pack/plan, independent of procurement sample-photo purge.
9. **Trace precision is reported honestly:** where lot capture was coarse, trace reports say so.
10. **Food-safety incidents** link guest case, lots, staff on shift and corrective actions (SF57.2.6, SF61.2.5).

### 9.4 Acceptance hooks
G18 (receiving/issue/waste/recall/duplicate scans), G15 (chef callout with credential check), AC-SF57.2.2 (checks run against the active rule-pack/plan version and label it), AC-SF61.2.4 (independent release after critical nonconformance).

---

## 10. KPIs for operations and continuity

Housekeeping turnaround and room-ready-by-arrival; inspection pass rate; linen loss and laundry cost per occupied room; minibar disputes; chef callout time-to-fill; food waste cost and recipe variance; average stock age; OTIF; check completion rate and critical nonconformances; incident acknowledgment time; lost-item match/return rate; pickups on time; cash variance; void/discount outliers; certification compliance; workflow SLA breach rate; uptime and measured RTO/RPO; drill completion and corrective-action closure; utility per occupied room with data-quality share. Definitions: `docs/06`. Targets: pilot hotel (Section P.6).

## 11. Open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-721 | Confirm shift pattern and all SLA defaults in §4 | GM of pilot hotel | Values in this file |
| D-722 | RTO/RPO per tier and budget for standby/secondary link | Owner + IT Admin | §7 values |
| D-723 | Food-safety plan owner and plan values per outlet | Executive Chef + Compliance Officer | Hotel plan required before Phase 4 food controls go live |
| D-724 | Food rule-pack authorities and validation for pilot jurisdiction | Compliance Officer | `unverified-assumption` |
| D-725 | Laundry model (in-house vs outsourced) and tolerance | Housekeeping lead | Outsourced, 2% tolerance |
| D-726 | Fleet ownership (owned shuttle vs taxi only) | GM | Taxi via approved providers; owned fleet only if pilot operates one |
| D-727 | Life-safety interface (panel model, approval, certification) | Chief Engineer | No interface; manual incident logging |
| D-728 | IR retainer, cyber insurance, breach-notification counsel | Owner + DPO | To procure before Phase 6 |
| D-729 | Lost-and-found retention/disposal per market | Compliance Officer | Hotel policy flagged "unverified" |

Decision ids D-721–D-739 are reserved for this file; `docs/13` is authoritative.
