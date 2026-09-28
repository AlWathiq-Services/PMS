# 04 — Screens, Flows and Design System

**Pack:** MetriStay Hospitality Suite Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Governing source:** master prompt v3.0 §F (application families), §P.1 (simplicity, role home, WCAG 2.2 AA, low bandwidth), §E, §G, §K, §Q • **Conventions:** `docs/README.md` §3 (screen id `SCR-<app>-<name>`, actors §3.3, honesty labels §3.6). Architecture references: `docs/03-architecture.md`.
**Status:** design target. No screen exists. Wireframes are low-fidelity and annotate behaviour, not visual design.

> Decision ids in this document use the reserved range **D-401..D-429** (UX/design), registered in `docs/13`. Catalogue writers (`docs/01`) reference screens by the ids defined here; an id not in this document is a defect to be added here first.

---

## 1. How to read this document

### 1.1 Application codes

| Code | Application family (§F) | Platforms | Identity realm (`docs/03` §11.2) | Primary modules |
|---|---|---|---|---|
| OPS | Cross-cutting staff shell: inbox, approvals, exceptions, search, sync, handover | web-staff, staff app | staff | M63, M64, M02 |
| GM | Owner/GM, revenue and sales office (hotel-side commercial) | web-staff (desktop/tablet), staff app (read + approve) | staff | M32, M53, M10, M12 (blocks), M65, M66, M67 |
| FO | Front desk and housekeeping | web-staff, tablet, staff app (housekeeping offline) | staff | M03–M06, M08 (desk), M55, M56, M17 (permits) |
| FIN | Finance: cashier, night audit, GL, AP/AR, reconciliation, utility bills, payroll approval | web-staff | staff | M08, M19, M20, M22–M25 (finance), M28, M29, M30 liability, M31 payouts, M60, M66 |
| HR | HR, payroll and employee self-service | web-staff, staff app (ESS) | staff | M27, M38 (payroll), M62 |
| ENG | Engineering, assets, meters, utilities, contractors | web-staff, staff app (offline) | staff | M22–M26, M64 (site), M67 |
| VEN | Vendor self-service web (fallback to the app) | web-vendor | vendor | M46, M48, M49, M50, M26 (vendor portal), M47 (emergency chef) |
| VAPP | Vendor Android/iOS app | mobile-vendor | vendor | M48, M46, M49, M50, M47 |
| CON | Concierge, travel desk, transport and fleet | web-staff, staff app (driver) | staff | M45, M59 |
| FNB | Bar, kitchen, club, catering, restaurant, events/BEO, amenities | web-staff, POS/KDS terminals, tablet | staff, device | M13–M16, M12 (BEO), M47, M57, M58, M54 |
| PRC | Procurement, vendor directory/approval, receiving, stores | web-staff, staff app (receiving offline draft), dock tablet | staff | M21, M46 (hotel side), M49, M50, M14, M25 |
| GST | Guest website, guest app, referral Partner Hub | web-guest (SSR), mobile-guest, web-partner-hub | guest | M51, M05 (guest), M18, M41 (guest), M30, M31, M54, M55, M43 (inquiry) |
| CORP | Corporate web portal (MetriStay Business) | web-corporate | corporate | M11, M10, M12, M20 (AR view) |
| CAPP | Corporate Android/iOS app (core-journey parity) | mobile-corporate | corporate | M11 |
| PRK | Parking and security | web-staff, lane tablet, staff app | staff, device | M17, M42 (security link) |
| ADM | Integrations, platform and regulatory administration | web-staff (desktop) | staff | M01, M02, M33, M38, M44, M63, M64, M65, config of all |
| MED | Property content, website and marketing/CRM | web-staff | staff | M39, M51, M52, M54 (offer setup) |
| SAF | Safety, incidents, lost & found, guest verification, inspections, risk | web-staff, staff app | staff | M41 (staff side), M42, M43, M61, M68 |
| AI | Guest AI assistant (guest widget) and staff handoff/QA | web-guest, mobile-guest, approved messaging, web-staff | guest, staff | M40 |

### 1.2 Column legend for screen tables

Every screen row carries all ten attributes required by §F. Codes below expand to full behaviour; the row adds anything specific.

| Column | Meaning |
|---|---|
| **ID** | `SCR-<APP>-<name>` — stable id for catalogue, tests and traceability |
| **Purpose** | the decision or task the screen serves |
| **Actor** | primary actor(s) from README §3.3 |
| **Key data fields** | principal fields shown or captured (not exhaustive) |
| **States** | screen/record states that change what is shown or allowed |
| **E/L/E** | empty / loading / error behaviour pattern code (§2.2) + specifics |
| **Perm** | permission string(s) `<ctx>.<resource>.<action>` (+ scope) |
| **Audit** | audit level (§2.4) |
| **Notif** | notifications emitted or received (§2.5) |
| **A11y** | accessibility tags beyond the baseline `A-std` (§2.6) |
| **Mobile** | mobile layout code (§2.7) |

---

## 2. Global screen standards

### 2.1 Common state vocabulary
`draft`, `pending`, `in_review`, `approved`, `rejected`, `active`, `expired`, `blocked`, `unknown` (external outcome not yet proven), `queued` (offline), `conflict`, `read_only` (period/business date locked or permission). Record states come from state machines in `docs/02`; screens never invent a state not in the machine.

### 2.2 Empty / loading / error patterns

| Code | Pattern | Empty | Loading | Error |
|---|---|---|---|---|
| EL-L | List / grid | Explains why empty (no data vs filters vs permission) + primary action; never a blank table | skeleton rows (≤ 300 ms delay before showing to avoid flicker) | inline banner with Retry, plain-language cause, `correlation_id`; last good data kept with *stale since hh:mm* badge |
| EL-F | Form / detail | n/a; "not found or no access" page with search (never reveals existence across tenants) | field skeletons; submit button shows progress, disabled to prevent double submit | field-level errors linked to inputs + error summary at top (focus moves to it); `409` edit conflict → compare/merge dialog; unsaved-changes guard; local draft autosave |
| EL-M | Money / irreversible action | n/a | "Processing — do not close" with idempotency reference; no optimistic UI | timeout → **Status unknown — checking with provider** state; automatic status inquiry; *no automatic retry*; manual evidence path; decline shows provider reason code mapped to plain language |
| EL-D | Dashboard / KPI | tile says "No data for this period" + setup link if source not configured | per-tile skeleton; page usable while tiles load | per-tile error; tile shows **Incomplete estimate** when any source missing or unreconciled; each tile shows source, as-of time and `estimate/reconciled/certified` label |
| EL-R | Realtime board / device feed | "No items right now" + last event time | connecting indicator | connection lost → banner "Live updates paused, showing data as of hh:mm", auto-reconnect with backoff; device offline → link to manual fallback screen |
| EL-O | Offline-capable mobile | cached snapshot with as-of time | cached first, refresh in background | offline: queued badge, conflict cards (SCR-OPS-sync-queue); never-offline actions disabled with reason (`docs/03` §8.2) |
| EL-P | Public guest page | helpful alternatives (other dates, contact) | SSR content first, minimal JS; progressive enhancement | plain-language message, entered data preserved, assisted path (phone / approved WhatsApp / front desk); no dead ends |
| EL-C | Capture (camera, scanner, scale, upload) | n/a | capture guidance, resumable upload with progress | permission denied / device missing → manual entry path; poor quality → guidance and retake; malware-scan pending state; file type/size errors specific |

### 2.3 Permission notation
`ctx.resource.action@scope` where scope ∈ `property` (default), `dept`, `own` (record ownership/assignment), `tenant`. `+stepup` = requires step-up (`acr=2`, `docs/03` §11.3). `+mc` = maker-checker (a second, different principal approves). `+limit` = value limit on the grant. Field protection `field:<class>` refers to `docs/03` §11.6.

### 2.4 Audit levels

| Code | Meaning |
|---|---|
| AU-0 | access logging only (public or non-sensitive read) |
| AU-R | sensitive read logged per record (`iam.sensitive_access_log`): ID, salary, bank, health, guest PII bulk export |
| AU-W | change audit: who/when/what (field diff), reason when configured |
| AU-P | privileged: reason mandatory, step-up and/or maker-checker, both principals recorded |
| AU-L | ledger: immutable entry, reversal link, source document link (`docs/03` §10) |
| AU-E | evidence: hashed artefacts, chain of custody/hash-chained log (incident, custody, signature, receiving) |

### 2.5 Notification codes
`N-in` in-app unified inbox item • `N-push` mobile push • `N-sms` SMS • `N-wa` approved WhatsApp template • `N-email` • `N-voice` automated call (callouts, critical incidents) • `N-esc` escalation timer (inbox item escalates to next owner on SLA breach) • `N-rt` realtime board update. Marketing messages always check consent (`iam.consent_record`); operational messages follow the notification preferences screen but critical safety/payroll-failure notifications cannot be muted.

### 2.6 Accessibility tags (WCAG 2.2 AA; baseline `A-std` applies to every screen, §8)

| Tag | Adds |
|---|---|
| A-std | keyboard operable, visible focus (2.4.7) not obscured by sticky UI (2.4.11), labels/names, contrast (1.4.3/1.4.11), reflow at 320 CSS px (1.4.10), text spacing (1.4.12), target size ≥ 24 px (2.5.8; we use 44 px on touch), language of page/parts (3.1.1/3.1.2), RTL mirroring, reduced motion |
| A-grid | data grid semantics (role=grid/table, header association, row summaries for screen readers, keyboard cell navigation, sortable header state announced) |
| A-chart | every chart has a data table alternative and a one-sentence text summary; colour not the only encoding (1.4.1) |
| A-form | programmatic labels, `autocomplete` tokens (1.3.5), error identification/suggestion (3.3.1/3.3.3), no redundant entry within a flow (3.3.7), error prevention for legal/financial (3.3.4: review + confirm + reversible) |
| A-auth | accessible authentication (3.3.8): no cognitive test; paste allowed; passkey/magic link/OTP autofill (`autocomplete="one-time-code"`); no CAPTCHA puzzles — use invisible risk scoring with accessible alternative |
| A-time | time limits adjustable/extendable (2.2.1) — holds, OTP, quotes show countdown with "extend" where business rules allow and warn ≥ 60 s before expiry |
| A-drag | single-pointer alternative to drag (2.5.7) — e.g. room rack moves, roster, function diary |
| A-cam | non-camera alternative (manual entry, staff-assisted path); instructions not solely visual |
| A-live | status changes announced via `aria-live` (polite for updates, assertive for critical alarms) (4.1.3) |
| A-map | list/table alternative for floor plans, meter maps, seat maps |
| A-media | captions/subtitles, transcripts, audio description for promotional video (1.2.x), alt text EN/AR |
| A-sig | signature alternative: typed name + checkbox intent, or staff-assisted; no fine-motor dependency |
| A-help | consistent help location (3.2.6) — assisted contact on every guest step |

### 2.7 Mobile layout codes

| Code | Meaning |
|---|---|
| ML-P | full parity on phone; single column; primary action in thumb zone (bottom, start-aligned in LTR, end-aligned mirrored in RTL) |
| ML-C | card stack on phone; filters in bottom sheet; swipe actions have button alternatives |
| ML-T | tablet-first (landscape) — used at desks, docks, kitchens; phone shows reduced read + key actions |
| ML-D | desktop-first dense screen; phone shows read-only summary and "continue on desktop" hand-off link |
| ML-K | fixed terminal/kiosk layout (POS, KDS, lane tablet, door scanner) |
| ML-N | native app screen (VAPP/CAPP/staff app) — see app's parity table |

### 2.8 Honesty and freshness badges (README §3.6)
Any value from an external or estimated source displays one of: `SIMULATOR`, `estimate`, `reconciled`, `certified`, `stale (as of …)`, `manual evidence`, `partner blocked`. Money actions for adapters labelled below `sandbox-tested` are not shown in production (`docs/03` §12).

---

## 3. Navigation model and role home screens (§P.1)

**Rules.** (1) Navigation is task-based: each app shows a role home with **3–7 priority tasks** (cards ordered by due time and severity) and a **unified inbox** (SCR-OPS-unified-inbox) filtered to the role's scope. (2) A global search (SCR-OPS-global-search) finds guests, reservations, rooms, POs, vendors, invoices, plates, lost items and incidents within permission. (3) Modules not enabled for the property (e.g. no club, no spa) are absent from navigation, search and inbox. (4) Every task card answers: *what needs attention, why, by when, who owns it, what can I do, did it work* — with one primary action and the authoritative status. (5) Progressive disclosure: detail and configuration live one level down. (6) Each home shows connectivity/outage state and the offline queue count on mobile.

| Role(s) | Home screen | Priority tasks (3–7, in default order) | Inbox filter (default) | KPI strip (source-labelled) |
|---|---|---|---|---|
| owner | SCR-GM-owner-home | 1 Review daily flash vs budget; 2 Approve items above GM limit (capex, write-offs); 3 Review profit bridge & incomplete sources; 4 Cash/tax/insurance obligations due ≤ 14 d; 5 Owner statement | approvals ≥ GM limit; owner reports | GOP, NOI proxy, cash, occupancy, RevPAR |
| gm | SCR-GM-home | 1 Exceptions breaching SLA (all depts); 2 Pending approvals; 3 Today: VIP/accessibility arrivals, overbooking risk; 4 Coverage gaps (chef/shift); 5 Open incidents; 6 Flash drill-down | all escalations ≥ dept manager | occupancy, ADR, RevPAR, TRevPAR, complaints open, labor % |
| duty_manager | SCR-GM-duty-manager-home | 1 Active incidents; 2 Guest cases at risk of SLA; 3 Rooms blocking arrivals; 4 Outage/devices status; 5 Handover notes | escalations for current shift | arrivals pending, rooms ready %, open cases |
| revenue_manager | SCR-GM-revenue-home | 1 Rate recommendations awaiting decision; 2 Rate publish failures/discrepancies; 3 Dates with pace below forecast; 4 Overbooking exposure; 5 Group blocks near cutoff | revenue exceptions | pickup 7/30/90, forecast occ, net ADR |
| sales_manager | SCR-GM-sales-home | 1 Corporate RFQs to answer; 2 Holds expiring ≤ 48 h; 3 Contracts to renew; 4 Blocks near cutoff; 5 Event change orders | sales tasks | pipeline value, conversion, pickup vs block |
| front_desk_agent | SCR-FO-home | 1 Arrivals ready/not ready; 2 Check-ins in progress (ID/sign/OTP pending); 3 Departures & balances; 4 Guest messages/requests; 5 Room moves requested; 6 Parking permits to issue | FO tasks, own queue | arrivals, in-house, departures, rooms ready |
| front_office_manager | SCR-FO-fo-manager-home | 1 Overbooking/walk decisions; 2 Unassigned arrivals; 3 Case SLA risks; 4 Cash shift variances; 5 Night audit readiness | FO escalations | arrivals by ETA, no-show risk |
| night_auditor | SCR-FO-night-audit | 1 Pre-audit checklist; 2 Exceptions (unposted, unbalanced, open checks); 3 Run audit; 4 Reports distribution | audit exceptions | business date, balances |
| housekeeper | SCR-FO-hk-home | 1 My rooms in priority order; 2 Rush rooms for arrivals; 3 Inspection failures to redo; 4 Minibar counts; 5 Report fault/lost item | own tasks | rooms done/remaining |
| housekeeping_supervisor | SCR-FO-hk-supervisor-home | 1 Assign/rebalance rooms; 2 Inspections due; 3 Room-ready ETA vs arrivals; 4 Linen par shortfalls; 5 Laundry returns mismatches | HK exceptions | ready %, turnaround time |
| cashier / finance_clerk | SCR-FIN-home | 1 Open/close shift; 2 Posting exceptions; 3 Unmatched payments/settlements; 4 Refunds awaiting approval; 5 Supplier invoices to capture | FIN tasks | unposted count, unreconciled value |
| financial_controller | SCR-FIN-controller-home | 1 Period close checklist; 2 Approvals (payables, write-offs, reopen); 3 Reconciliation breaks; 4 Payment batch release queue; 5 Incomplete cost sources | FIN escalations | trial balance status, AR > 60 d, AP due 7 d |
| ap_clerk | SCR-FIN-ap-inbox | 1 Invoices to capture/OCR-verify; 2 Match exceptions; 3 Duplicates flagged; 4 Utility bill variances; 5 Payables due | AP tasks | invoices pending, value due |
| hr_officer | SCR-HR-home | 1 Joiners/leavers; 2 Expiring contracts/certifications; 3 Leave approvals; 4 Roster gaps; 5 Sensitive-data requests | HR tasks | headcount, open positions |
| payroll_officer / payroll_approver | SCR-HR-payroll-home | 1 Current run status; 2 Exceptions; 3 Approvals; 4 Bank/WPS rejections; 5 Statutory filings due | payroll tasks | run progress, rejected payments |
| employee | SCR-HR-ess-home | 1 My next shifts; 2 Clock in/out; 3 Leave request; 4 Payslips; 5 Required training | own | — |
| engineer | SCR-ENG-home | 1 My jobs by SLA; 2 Emergency jobs; 3 PM tasks due today; 4 Rooms awaiting release inspection; 5 Meter reads due | own jobs | open WOs, SLA breaches |
| chief_engineer | SCR-ENG-chief-home | 1 SLA breaches; 2 OOO rooms & return dates; 3 Contractor jobs awaiting acceptance; 4 Consumption anomalies; 5 Parts requisitions | ENG escalations | MTTR, OOO room-nights, kWh/occupied room |
| procurement_officer / approver | SCR-PRC-home | 1 Requisitions to tender; 2 RFQs below quote minimum; 3 Comparisons ready to award; 4 POs unacknowledged / late ETA; 5 Vendor approvals/expiring credentials | PRC tasks | open RFQs, OTIF, savings |
| storekeeper | SCR-PRC-storekeeper-home | 1 Issue requests; 2 Returns to inspect (intact vs waste); 3 Quarantine decisions; 4 Counts due; 5 Expiring lots (FEFO) | store tasks | stock value, expiring |
| receiver | SCR-PRC-receiver-home | 1 Expected deliveries today (dock slots); 2 Arrived at gate; 3 High-risk items to verify; 4 Discrepancies to record | receiving | deliveries due/arrived |
| executive_chef / shift_chef | SCR-FNB-chef-home | 1 Coverage status for next services; 2 BEOs/production due with allergens; 3 Callout in progress; 4 Stock/lot alerts & recalls; 5 Waste to approve; 6 HACCP checks due | kitchen tasks | covers today, food cost %, waste |
| fnb_manager | SCR-FNB-home | 1 Void/comp approvals; 2 Shift closes; 3 Theoretical vs actual variance; 4 Outlet staffing; 5 Guest complaints F&B | F&B escalations | covers, avg check, bev cost % |
| bartender / server | SCR-FNB-bartender-home | 1 Open tabs; 2 Orders ready; 3 Low stock at bar; 4 Shift close | own | — |
| catering_manager | SCR-FNB-function-diary | 1 Events next 72 h; 2 BEO revisions unsigned; 3 Guarantee cutoffs; 4 Change orders; 5 Post-event settlement | events | events, covers, margin |
| club_host | SCR-FNB-club-entry | 1 Door scan; 2 Capacity now; 3 Reservations tonight; 4 Incidents | club | capacity used |
| concierge | SCR-CON-home | 1 New requests; 2 Offers awaiting guest approval; 3 Offers expiring; 4 Orders pending provider confirmation; 5 Disruptions; 6 Pickups next 3 h | travel/transport | pending confirmations |
| parking_attendant / security_officer | SCR-PRK-home / SCR-PRK-security-home | 1 Low-confidence lane reviews; 2 Gate exceptions; 3 Occupancy alerts; 4 Incidents/patrol; 5 Lost items to log | security | occupancy, open reviews |
| it_admin / integration_admin | SCR-ADM-home | 1 Connector health red/amber; 2 Dead letters; 3 Certificates/secrets expiring; 4 Backup/restore status; 5 Device health; 6 Pending access reviews | platform | uptime, outbox lag |
| compliance_officer / dpo | SCR-ADM-coverage-dashboard | 1 Rule versions to review/expiring; 2 Blocked activation gates; 3 Filings due/unsubmitted; 4 Data subject requests; 5 Legal holds | compliance | coverage %, unknowns |
| content_editor / content_approver | SCR-MED-home | 1 Assets awaiting approval; 2 Enhancement comparisons; 3 Rights expiring; 4 Publish failures; 5 KB articles due review | content | published assets, alt-text coverage |
| marketing_manager / guest_relations | SCR-MED-marketing-home | 1 Reviews to respond; 2 Campaigns awaiting approval; 3 Recovery cases open; 4 Survey detractors; 5 Attribution anomalies | CRM | NPS/score, response time |
| referral_program_admin | SCR-ADM-referral-program | 1 Attribution disputes; 2 Commissions to approve; 3 Market gate status; 4 Fraud flags | referral | pending/approved/paid |
| vendor_admin / vendor_user | SCR-VEN-home / SCR-VAPP-home | 1 Update today's stock & prices; 2 Open RFQs (deadline); 3 POs to acknowledge; 4 Deliveries to dispatch (ASN); 5 Documents expiring; 6 Invoices/payments | vendor inbox | OTIF, rating, payments due |
| emergency_chef | SCR-VAPP-callout-accept | 1 Active callout offers; 2 My availability; 3 Assigned shift handover | callouts | — |
| corporate_booker / admin / approver | SCR-CORP-home / SCR-CAPP-home | 1 Search & hold; 2 Holds expiring; 3 Approvals; 4 Rooming lists due; 5 Invoices due | corporate | spend vs budget |
| guest | SCR-GST-my-trips | 1 Upcoming stay & pre-check-in; 2 Requests; 3 Pay/receipts; 4 Points | messages | — |
| auditor | SCR-ADM-audit-log | read-only: audit log, ledgers, reports | none | — |

