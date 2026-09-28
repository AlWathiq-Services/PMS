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

---

## 4. Screen maps by application

Each subsection opens with the navigation map (tree), followed by the screen table. Column legend §1.2; codes §2.

### 4.1 OPS — cross-cutting staff shell

```
Shell (every staff app)
├─ Role home (per app, §3) ── Unified inbox ── Task detail ── Approval detail
├─ Global search ── record pages (any app)
├─ Exceptions ── exception detail ── case timeline
├─ Notifications ── preferences
├─ Shift handover
├─ Sync queue (mobile) ── conflict card
└─ Account: login, step-up, property switch, profile, device enrolment, help/SOP, app update, outage mode
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-OPS-login | Sign in to staff realm with MFA/passkey; realm-specific branding | all staff | username/email, passkey, OTP, language toggle EN/AR | signed_out, mfa_required, locked, password_expired | EL-F; lockout message without account enumeration | public | AU-W (auth events) | N-email on new device | A-auth, A-form | ML-P |
| SCR-OPS-step-up | Re-authenticate for privileged action (acr=2) | privileged roles | action summary, method (passkey/OTP), reason (when required) | prompt, verified, failed, expired | EL-F; failure keeps pending action intact | any `+stepup` | AU-P | — | A-auth, A-time | ML-P |
| SCR-OPS-property-switcher | Switch active property/department scope | multi-property staff | property list, role per property, business date per property | — | EL-L | iam.scope.use | AU-W | — | A-std | ML-P |
| SCR-OPS-unified-inbox | Single queue of tasks, approvals, exceptions, messages for role scope with owner/due/why | all staff | type, title, why, due_at, SLA timer, owner, severity, source record link, primary action | new, acknowledged, in_progress, snoozed, escalated, done | EL-L; empty = "Nothing needs you now" + last cleared time | ops.inbox.read@own/dept | AU-W (claim/reassign) | receives N-in, N-esc; emits N-push on assignment | A-grid, A-live | ML-C |
| SCR-OPS-task-detail | Work one task: context, SOP, checklist, action buttons, history | task owner | task template version, checklist items, attachments, linked records, comments, SLA | open, in_progress, blocked, done, cancelled | EL-F | ops.task.update@own | AU-W | N-esc on breach | A-form | ML-P |
| SCR-OPS-approvals-queue | All approvals for the user (maker-checker) grouped by type and value | approvers | request type, amount/currency, requester, reason, evidence count, due | pending, approved, rejected, expired, recalled | EL-L | ops.approval.decide (+limit) | AU-P | N-in, N-push, N-esc | A-grid | ML-C |
| SCR-OPS-approval-detail | Decide one approval with evidence; approve/reject/return with reason | approvers | before/after diff, evidence, policy rule, limit, SoD check result, requester ≠ approver | pending, decided | EL-M for money approvals | per type `+mc`, `+stepup` for money/privileged | AU-P | N-in to requester | A-form (3.3.4 confirm) | ML-P |
| SCR-OPS-exception-queue | Cross-module exceptions (integration failures, reconciliation breaks, ledger holds) with owner and next action | dept managers, finance, integration_admin | exception type, source, value at risk, age, owner, suggested action, manual path | open, assigned, waiting_external, resolved, dismissed_with_reason | EL-L | ops.exception.read@dept, .resolve | AU-W / AU-P (dismiss money) | N-in, N-esc | A-grid | ML-C |
| SCR-OPS-exception-detail | Resolve one exception: evidence, retry/inquire, manual evidence upload, link to record | owner | raw status log, provider refs, idempotency key, related ledger entries, resolution notes | open → resolved | EL-M where money | ops.exception.resolve | AU-P | N-in | A-form | ML-P |
| SCR-OPS-global-search | Search across permitted entities; recent items | all staff | query, entity type chips, results with scope badges | — | EL-L; "no results — check spelling/Arabic variants" | per entity read | AU-R for restricted hits (none returned for restricted fields) | — | A-std, A-live (result count) | ML-P |
| SCR-OPS-notification-center | Chronological notifications with read state | all staff | title, source, time, link | unread, read | EL-L | own | AU-0 | receives all | A-live | ML-P |
| SCR-OPS-notification-preferences | Choose channels/quiet hours; critical ones locked | all staff | per category channel toggles, quiet hours, language | — | EL-F | own | AU-W | — | A-form | ML-P |
| SCR-OPS-shift-handover | Record and acknowledge shift handover (open items, risks) | duty_manager, FO, HK, kitchen, security | shift, department, open items (auto-pulled), notes, acknowledged_by | draft, submitted, acknowledged | EL-F | ops.handover.write@dept | AU-W | N-in to incoming shift | A-form | ML-P |
| SCR-OPS-sync-queue | Show queued offline commands, results, conflicts; resolve | staff app users | command, entity, client time, status, server reason, suggested resolution | queued, sending, accepted, merged, rejected, conflict | EL-O | own | AU-W | N-push on rejection | A-live | ML-N |
| SCR-OPS-device-enrollment | Enrol shared device (POS, tablet, KDS, lane) with device identity | it_admin | device type, serial, property, role profile, certificate status | pending, enrolled, revoked | EL-F | plt.device.enroll `+stepup` | AU-P | N-email to admin | A-form | ML-T |
| SCR-OPS-my-profile | Language, digits (Latin/Arabic-Indic), Hijri display, theme, accessibility prefs | all staff | locale, calendar display, number system, text size, reduced motion | — | EL-F | own | AU-W | — | A-form | ML-P |
| SCR-OPS-help-sop | Contextual SOP/training viewer from current screen | all staff | SOP version, language, steps, media | current, superseded | EL-L | ops.sop.read | AU-0 (attestation in HR) | — | A-media | ML-P |
| SCR-OPS-app-update-required | Block unsupported app versions with update path | app users | current/min version, store/MDM link | — | EL-F | public | AU-0 | — | A-std | ML-N |
| SCR-OPS-outage-mode | Guide staff during outage: what works, manual forms, what's queued, what's blocked | all staff | connectivity state, affected integrations, manual procedures, printable forms, queue counts | normal, degraded, offline, recovering | EL-R | ops.outage.read | AU-W (manual records) | N-push broadcast | A-live | ML-P |
| SCR-OPS-case-timeline | Cross-department case/workflow timeline (M63) | case owners, managers | steps, owners, SLA per step, handoffs, approvals | running, paused, escalated, completed, cancelled | EL-L | ops.workflow.read@dept | AU-W | N-esc | A-live | ML-C |
| SCR-OPS-report-viewer | Render any report with filters, source/freshness, export | report users | parameters, version, as-of, coverage flags, export format | draft_params, running, ready, failed | EL-D | bi.report.read (+ row/field filters) | AU-R for exports with PII | N-email for scheduled | A-grid, A-chart | ML-D |
| SCR-OPS-record-history | Drawer showing audit history of any record | managers, auditor | who/when/what, reason, approvals, correlation id | — | EL-L | iam.audit.read@dept | AU-R | — | A-grid | ML-C |

### 4.2 GM — owner, GM, revenue and sales office

```
GM app
├─ Owner home ── owner statement, capex, profit bridge
├─ GM home ── live operations, approvals, flash ── flash drill-down ── source ledger
├─ Performance: department P&L, budget vs actual, cash, AP due, payroll summary, utilities, maintenance SLA, sustainability
├─ Reports: catalogue, custom pivot, scheduled, data coverage
├─ Revenue: home, pickup/pace, forecast, rate grid, recommendations, restrictions, overbooking, channel performance, publish status
└─ Sales: home, pipeline, corporate account, agreement editor, RFQ response/proposal, group block
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-GM-owner-home | Owner/asset manager priority view (§3) | owner | flash KPIs, GOP, cash, obligations due, approvals ≥ GM limit, incomplete-source warnings | — | EL-D | bi.owner.read | AU-0 | N-email weekly digest | A-chart | ML-C |
| SCR-GM-home | GM role home (§3) | gm | tasks, inbox, KPI strip, incidents, coverage gaps | — | EL-D | bi.gm.read | AU-0 | N-in, N-esc | A-chart, A-live | ML-C |
| SCR-GM-duty-manager-home | Shift-level control for duty manager | duty_manager | incidents, cases at risk, arrivals blocked, device/outage status, handover | — | EL-R | ops.duty.read | AU-0 | N-push critical | A-live | ML-C |
| SCR-GM-live-operations | Live board: arrivals/departures, in-house, rooms by state, events today, incidents | gm, duty_manager | counts with drill links, room state matrix, VIP/accessibility flags | — | EL-R | res.board.read | AU-0 | N-rt | A-live, A-map | ML-C |
| SCR-GM-flash | Daily flash (SF32.4.1) for business date | owner, gm, financial_controller | occupancy, ADR, RevPAR, TRevPAR, revenue by dept, labor, energy, cash, AR, variances vs budget/LY, close status | draft (day open), provisional, closed (after night audit), restated | EL-D (incomplete estimate labels) | bi.flash.read | AU-0 | N-email scheduled | A-chart, A-grid | ML-C |
| SCR-GM-flash-drilldown | Drill any KPI to components and source ledger entries (critical flow W10) | owner, gm | KPI definition/denominator, components, allocation version, source events, ledger lines, evidence | estimate, reconciled, certified | EL-D | bi.flash.drill (+ field filters: no individual salary) | AU-R for PII-bearing drill | — | A-grid, A-chart | ML-D |
| SCR-GM-occupancy-forecast | Occupancy/revenue forecast summary for GM | gm | 90-day forecast, confidence band, OTB, group pickup | — | EL-D | dst.forecast.read | AU-0 | — | A-chart | ML-C |
| SCR-GM-budget-vs-actual | Budget vs actual vs forecast by department/account | gm, financial_controller | budget version, actual, estimate, variance, commentary | open period, closed | EL-D | fin.budget.read@dept | AU-W (commentary) | — | A-grid | ML-D |
| SCR-GM-department-pnl | Departmental P&L (rooms, F&B outlets, club, catering, parking, amenities) | gm, owner, dept heads (own dept) | revenue, COGS, labor (aggregate), utilities allocated, maintenance, fees, contribution | provisional, closed, restated | EL-D | bi.pnl.read@dept | AU-0 | — | A-grid, A-chart | ML-D |
| SCR-GM-profit-bridge | GOP → operating result → net income bridge (SF32.3.2) | owner, gm | revenue, departmental costs, undistributed, fees, fixed charges, allocation versions, coverage flags | provisional, closed | EL-D | bi.pnl.read | AU-0 | — | A-chart (waterfall + table) | ML-D |
| SCR-GM-cash-position | Cash, bank balances, PSP in transit, forecast 30 d | owner, gm, financial_controller | bank balances (as-of), unsettled PSP, AP due, AR expected, payroll dates | — | EL-D | fin.cash.read | AU-0 | — | A-chart | ML-C |
| SCR-GM-ap-due | Payables due by week/category; approval status | gm, financial_controller | due date, vendor, amount, status planned→settled | — | EL-L | fin.payable.read | AU-0 | — | A-grid | ML-C |
| SCR-GM-payroll-summary | Labor cost aggregates by department (no individual pay) | gm, owner | dept, headcount, hours, OT, labor cost, % revenue | provisional, approved | EL-D | wrk.payroll.read_aggregate | AU-0 | — | A-chart | ML-C |
| SCR-GM-utilities-overview | Electricity/water/gas usage, cost, intensity, anomalies, bill status | gm, chief_engineer | kWh/m3/gas per occupied room, bill vs meter variance, due bills | metered, estimated | EL-D | eng.utility.read | AU-0 | — | A-chart | ML-C |
| SCR-GM-maintenance-sla | SLA performance, OOO room-nights, critical open WOs | gm, chief_engineer | WO counts by priority, breaches, MTTR, OOO rooms and return dates | — | EL-D | eng.wo.read | AU-0 | — | A-chart | ML-C |
| SCR-GM-sustainability-dashboard | Resource intensity vs baseline/targets; evidence status (M67) | gm, owner | intensity metrics, baseline, target, verified/estimated flags, emission factor version | — | EL-D | eng.sustainability.read | AU-0 | — | A-chart | ML-D |
| SCR-GM-report-catalogue | Browse governed reports (SF32.4) by area | managers | report name, owner, definition version, schedule, permissions | — | EL-L | bi.report.list | AU-0 | — | A-grid | ML-C |
| SCR-GM-custom-pivot | Authorised custom pivot over governed datasets | gm, financial_controller, revenue_manager | dataset, dimensions, measures (KPI dictionary), filters, row/field security applied | draft, saved | EL-D | bi.pivot.create | AU-R (PII exports) | — | A-grid | ML-D |
| SCR-GM-scheduled-reports | Schedule PDF/CSV/XLSX delivery to consenting recipients | managers | report, cadence, recipients, format, last run status | active, paused, failed | EL-L | bi.schedule.write | AU-W | N-email | A-form | ML-D |
| SCR-GM-data-coverage | Freshness and close status per source (M65) | gm, financial_controller | source, last event, expected cadence, stale flag, unposted items, close status | fresh, stale, missing | EL-D | bi.coverage.read | AU-0 | N-in stale feed | A-grid | ML-C |
| SCR-GM-owner-statement | Owner statement and management/franchise fee view (M66) | owner, financial_controller | period, owner-relevant P&L, fees, distributions, notes | draft, issued | EL-D | fin.owner_statement.read | AU-R | N-email | A-grid | ML-D |
| SCR-GM-capex-requests | Capex requests, investment case, approval, renovation closures | gm, owner, chief_engineer | request, budget, payback, rooms affected & dates, status | draft, submitted, approved, rejected, in_progress, closed | EL-F | fin.capex.request / .approve `+mc` | AU-P | N-in | A-form | ML-D |
| SCR-GM-revenue-home | Revenue manager role home (§3) | revenue_manager | recommendations, publish failures, pace alerts, overbooking exposure | — | EL-D | dst.revenue.read | AU-0 | N-in | A-chart | ML-C |
| SCR-GM-revenue-pickup-pace | Pickup/pace by stay date and booking date, by segment/channel | revenue_manager | OTB, pickup 1/7/30, LY/STLY, cancellations, group wash | — | EL-D | dst.pace.read | AU-0 | — | A-chart, A-grid | ML-D |
| SCR-GM-revenue-forecast | Demand forecast with confidence and data coverage (F53.1) | revenue_manager | forecast occ/ADR, band, drivers, events calendar, licensed comp input flag | — | EL-D | dst.forecast.read | AU-0 | — | A-chart | ML-D |
| SCR-GM-rate-grid | Rate plan × room type × date grid with overrides | revenue_manager | BAR, derived plans, negotiated plans (read-only), overrides, guardrails | draft changes, published, pending_ack | EL-L | inv.rate.write | AU-W | N-in on publish failure | A-grid | ML-D |
| SCR-GM-rate-recommendation | Review AI/rule rate recommendations with explanation; approve/reject; simulate | revenue_manager, gm (above guardrail) | current vs recommended, rationale, simulated occupancy/net yield, corporate contract impact, guardrail | proposed, approved, rejected, published, rolled_back | EL-F | dst.recommendation.decide (+ `+mc` beyond guardrail) | AU-P | N-in | A-chart, A-form | ML-C |
| SCR-GM-restrictions-calendar | LOS, CTA/CTD, stop-sell by date/room type/channel | revenue_manager | restriction matrix, source, effective dates | draft, published | EL-L | inv.restriction.write | AU-W | — | A-grid, A-drag | ML-D |
| SCR-GM-overbooking-control | Controlled overbooking limits and walk plan | revenue_manager, front_office_manager | limit per night/type, exposure, walk cost, partner hotels | proposed, approved | EL-F | inv.overbook.set `+mc` | AU-P | N-in to FO | A-form | ML-C |
| SCR-GM-channel-performance | Channel/segment net contribution after commission/fees (SF53.1.3) | revenue_manager, marketing_manager | room nights, gross, commission, payment fees, net ADR, cancellation rate | — | EL-D | dst.channel.read | AU-0 | — | A-chart | ML-D |
| SCR-GM-rate-publish-status | ARI publish acknowledgments, discrepancies, rollback | revenue_manager, integration_admin | message id, channel, payload summary, ack/error, discrepancy, rollback version | sent, acked, failed, discrepancy, rolled_back | EL-L | dst.publish.read / .rollback | AU-W | N-in on failure | A-grid | ML-C |
| SCR-GM-sales-home | Sales role home (§3) | sales_manager | RFQs, holds expiring, renewals, cutoffs | — | EL-D | pty.sales.read | AU-0 | N-in | A-std | ML-C |
| SCR-GM-sales-pipeline | Corporate/group opportunities pipeline | sales_manager | account, opportunity, stage, value, room nights, event dates, probability, next step | lead, qualified, proposal, negotiation, won, lost | EL-L | pty.opportunity.write | AU-W | N-in | A-grid, A-drag | ML-C |
| SCR-GM-corporate-account-detail | Corporate account: legal entities, contacts, users, credit, agreements, activity | sales_manager, ar_clerk | legal name, branches, cost centers, credit limit, AR status, agreements, portal users | prospect, active, on_hold, closed | EL-F | pty.corporate.read / .write | AU-W | — | A-form | ML-C |
| SCR-GM-corporate-agreement-editor | Versioned negotiated rates, eligibility, facilities, cancellation, credit terms | sales_manager, gm (approve) | version, valid dates, rate plans, room types, event packages, parking/club benefits | draft, pending_approval, approved, expired | EL-F | pty.agreement.write, .approve `+mc` | AU-P | N-email to corporate admin | A-form | ML-D |
| SCR-GM-corporate-rfq-response | Answer corporate RFQ with feasible proposal (rooms+space+F&B+parking) | sales_manager, catering_manager | request, feasible configurations, prices, hold expiry, proposal document version | received, drafting, sent, accepted, declined, expired | EL-F | com.proposal.write | AU-W | N-email/N-in to corporate | A-form | ML-D |
| SCR-GM-group-block | Room block by night/type, pickup, cutoff, wash | sales_manager, revenue_manager | block nights, picked-up, cutoff date, release rules, rooming list status | tentative, definite, cutoff_passed, released | EL-L | com.block.write | AU-W | N-in cutoff reminders | A-grid | ML-D |

### 4.3 FO — front desk and housekeeping

```
Front desk (web/tablet)                           Housekeeping (staff app, offline)
├─ Home ── arrivals ── check-in wizard            ├─ HK home (attendant) ── my rooms ── room task
│           ├─ room assignment                    │                          ├─ minibar count
│           ├─ ID intake (SAF) / signature / OTP  │                          ├─ fault report (→ENG) / lost item (→SAF)
│           └─ registration card                  ├─ Supervisor home ── room board ── assign ── inspection
├─ Availability grid, room rack                    ├─ Linen par ── laundry dispatch ── discrepancy
├─ Reservations: search, new, detail, amend,      └─ DND log, room-ready ETA
│   cancel, waitlist, no-show, group rooming
├─ Guest: profile, merge review, inbox, cases
├─ In-house: room move, extension, upsell, parking permit
├─ Departures ── checkout ── folio ── payment / invoice
└─ Night audit ── exceptions; outage registration
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-FO-home | Front desk agent home (§3) | front_desk_agent | arrivals ready/not ready, check-ins in progress with missing steps, departures with balances, messages, moves, permits | — | EL-R | res.frontdesk.read | AU-0 | N-in, N-rt | A-live | ML-T |
| SCR-FO-fo-manager-home | FO manager home (§3) | front_office_manager | overbooking/walk decisions, unassigned arrivals, case SLA risk, shift variances, audit readiness | — | EL-R | res.frontdesk.manage | AU-0 | N-esc | A-live | ML-T |
| SCR-FO-availability-grid | Room-type availability by date incl. holds, OOO, overbook limit | front_desk_agent, reservations, revenue_manager | per night: physical, OOO, sold, held, available, overbook remaining, stop-sell | — | EL-L | inv.availability.read | AU-0 | N-rt | A-grid | ML-C |
| SCR-FO-room-rack | Physical room assignment timeline (tape chart) | front_desk_agent, front_office_manager | rooms × nights, assignments, HK/service state, connecting/accessible flags | — | EL-R; conflicts highlighted (exclusion violation prevented) | res.assignment.write | AU-W | N-rt | A-grid, A-drag (menu "Move to room…") | ML-D |
| SCR-FO-reservation-search | Find reservations by name, conf no, phone, company, channel ref, dates | front desk, reservations | query, filters, results with status and balance | — | EL-L | res.reservation.read | AU-0 | — | A-grid | ML-C |
| SCR-FO-reservation-new | Quote and book (phone/walk-in/direct) with policy snapshot and hold | front_desk_agent, reservations | dates, party, room type, rate plan, corporate eligibility, total with taxes/fees (JUR), deposit policy, guest, payer, hold expiry | quoting, held, confirmed, failed | EL-M (confirm needs server ack; hold timer) | res.reservation.create | AU-W | N-email/N-sms confirmation | A-form, A-time | ML-C |
| SCR-FO-reservation-detail | Full reservation: rooms, occupants, booker vs payer, rates, policy snapshot, folio link, history | front desk | confirmation no, status, rooms, nightly rates, deposits, preferences, requests, channel ref, referral attribution (masked) | draft, held, confirmed, in_house, checked_out, cancelled, no_show | EL-F | res.reservation.read | AU-W on change | — | A-form | ML-C |
| SCR-FO-reservation-amend | Amend dates/rooms/occupants with repricing and override audit | front desk, reservations | old vs new price, policy impact, override reason, inventory result | draft, repriced, confirmed, rejected | EL-M | res.reservation.amend (+ override `+mc` beyond limit) | AU-P for overrides | N-email amended confirmation | A-form (3.3.4) | ML-C |
| SCR-FO-reservation-cancel | Cancel with policy fee, refund/points/commission reversal preview | front desk | cancellation policy, fee, refund amount, points to reverse, referral commission reversal | confirmed → cancelled | EL-M | res.reservation.cancel; refund `fin.refund.create +limit` | AU-P | N-email | A-form (3.3.4) | ML-C |
| SCR-FO-waitlist | Waitlisted requests and auto-offer when inventory frees | reservations | request, dates, priority, offer expiry | waiting, offered, accepted, expired | EL-L | res.waitlist.write | AU-W | N-sms/N-email offer | A-grid, A-time | ML-C |
| SCR-FO-guest-profile | Guest profile with preferences, history, consents, loyalty, cases | front desk, guest_relations | name(s) EN/AR, contacts, preferences (service), accessibility flag, stays, spend, consents, points tier | active, merged | EL-F | pty.guest.read / .write; field:health | AU-W; AU-R for restricted | — | A-form | ML-C |
| SCR-FO-guest-merge-review | Review suspected duplicates; merge/undo with evidence | guest_relations, front_office_manager | match score, identifiers, stays, differences | suggested, merged, rejected, undone | EL-L | pty.guest.merge `+mc` for high-value | AU-P | — | A-grid | ML-D |
| SCR-FO-arrivals | Today's/selected-date arrivals with readiness (room, payment, ID, registration) | front_desk_agent | ETA, VIP, accessibility, room assigned, HK state, deposit/guarantee, pre-check-in progress | expected, ready, partially_ready, checked_in, no_show_candidate | EL-R | res.arrival.read | AU-0 | N-rt | A-grid, A-live | ML-C |
| SCR-FO-check-in | Check-in wizard: verify reservation → room → payment guarantee → ID (SCR-SAF-id-intake) → registration sign → OTP → keys → welcome (critical flow W2) | front_desk_agent | reservation, room, guarantee method, ID status, signature status, OTP status, parking plate, key count | not_started, in_progress (per step), blocked (reason), completed | EL-M (completion needs server ack) | res.checkin.perform | AU-W + AU-E (signature) | N-sms/N-wa OTP; N-rt HK | A-form, A-cam, A-sig, A-time | ML-T |
| SCR-FO-room-assignment | Assign physical room honouring preferences/accessibility/connecting | front_desk_agent | candidate rooms, state, features, conflicts | unassigned, assigned, upgraded | EL-L | res.assignment.write | AU-W | N-rt HK rush | A-grid | ML-C |
| SCR-FO-registration-card | Show/print/e-sign registration card per JUR rule | front_desk_agent, guest | guest snapshot, required fields per jurisdiction, policy text, signature, document hash | draft, signed, void | EL-F | res.registration.write | AU-E | — | A-sig, A-form | ML-T |
| SCR-FO-in-house | In-house guests list with balance, alerts, requests | front desk | room, guest, departure date, balance vs limit, alerts | — | EL-L | res.stay.read | AU-0 | — | A-grid | ML-C |
| SCR-FO-room-move | Move stay to another room; inventory and rate impact; HK/parking/device events | front_desk_agent | from/to room, reason, rate change, charge, effective time | requested, completed, rejected | EL-M | res.stay.move | AU-W | N-rt HK, ENG | A-form | ML-C |
| SCR-FO-stay-extension | Extend/shorten stay with availability and repricing | front_desk_agent | new departure, availability, price, deposit top-up | requested, confirmed, rejected | EL-M | res.stay.extend | AU-W | N-email | A-form | ML-C |
| SCR-FO-departures | Departures with balances, express checkout status | front_desk_agent | room, balance, payment on file, express status, late checkout | expected, checked_out, late | EL-R | res.departure.read | AU-0 | — | A-grid | ML-C |
| SCR-FO-checkout | Checkout: review folio → settle → invoice → points → feedback (critical flow W3) | front_desk_agent, cashier | folio windows, balance per payer, tenders, invoice legal fields, points earn preview | open, settling, settled, checked_out | EL-M | res.checkout.perform; fin.payment.capture | AU-L | N-email invoice/receipt | A-form (3.3.4) | ML-T |
| SCR-FO-folio | Folio windows with append-only entries, routing, transfers, reversals | front desk, cashier, night_auditor | entries (date, business date, code, amount, tax lines, source), windows, balances | open, settled, closed, reopened | EL-L | fin.folio.read / .post | AU-L | — | A-grid | ML-D |
| SCR-FO-folio-routing | Define payer windows and routing rules (room to company, extras to guest) | front desk, sales | window, payer, revenue codes routed, limits | draft, active | EL-F | fin.folio.route | AU-W | — | A-form | ML-D |
| SCR-FO-folio-adjust | Post reversal/transfer/adjustment with reason and approval over limit | cashier, front_office_manager | original entry, reversal amount, reason code, approver | draft, pending_approval, posted | EL-M | fin.folio.adjust (+limit, `+mc`) | AU-L + AU-P | N-in approver | A-form (3.3.4) | ML-D |
| SCR-FO-payment-take | Take payment/deposit: card terminal, pay-by-link, cash, bank transfer, points, voucher, corporate credit | cashier, front_desk_agent | amount, currency, tender, terminal id, link status, idempotency ref | created, pending, succeeded, failed, unknown | EL-M | fin.payment.create | AU-L | N-sms/N-email pay link | A-form, A-time | ML-T |
| SCR-FO-deposit-preauth | Pre-authorisation and incremental auth for stay | cashier | auth amount, expiry, card token (last4), increments | authorized, incremented, captured, released, expired | EL-M | fin.payment.authorize | AU-L | — | A-form | ML-T |
| SCR-FO-invoice-print | Preview/issue fiscal invoice, receipt, credit note per JUR rules | cashier | fiscal number series, legal entity, tax breakdown, language(s), Hijri display where required | draft, issued, credited | EL-F | fin.invoice.issue | AU-L | N-email | A-std | ML-D |
| SCR-FO-no-show-processing | Process no-shows: fee, release inventory, commission/points reversal | front_office_manager, night_auditor | candidates, policy fee, guarantee, decisions | candidate, no_show, waived | EL-M | res.noshow.process | AU-P for waive | N-email | A-grid | ML-D |
| SCR-FO-corporate-guests | In-house/arriving corporate guests by company with billing instructions | front desk, sales | company, guest, rate agreement, routing, PO number | — | EL-L | res.corporate.read | AU-0 | — | A-grid | ML-C |
| SCR-FO-group-rooming-list | Import/edit rooming list against block, assign rooms | front desk, sales, event_organizer (via CORP) | block, names, sharing, arrival/departure, special needs, billing | draft, submitted, validated, applied | EL-L | res.rooming.write | AU-W | N-in on import | A-grid | ML-D |
| SCR-FO-parking-permits | Issue/modify guest parking permit linked to stay and plate | front_desk_agent | plate, region, validity, zone, tariff/free, stay link | issued, active, expired, revoked | EL-F | com.parking_permit.write | AU-W | N-rt gateway | A-form | ML-C |
| SCR-FO-service-requests | Desk view of guest requests/cases routed to owning teams | front desk, guest_relations | request, room, category, owner team, SLA, status | new, assigned, in_progress, done, reopened | EL-L | res.case.read / .write | AU-W | N-esc | A-grid, A-live | ML-C |
| SCR-FO-guest-case | Complaint/recovery case with compensation options and approval (F55.2) | guest_relations, front_office_manager | severity, owner, SLA, linked HK/ENG/F&B records, compensation (folio reversal/voucher/points) with cap | open, in_progress, awaiting_guest, resolved, reopened | EL-F; compensation EL-M | res.case.write; compensation +limit `+mc` | AU-P for compensation | N-in, N-esc, guest N-email/N-wa | A-form | ML-C |
| SCR-FO-guest-inbox | Omnichannel guest messaging (email, SMS, approved WhatsApp, web chat handoffs) | front desk, guest_relations, concierge | conversation, channel, consent state, templates, attachments, AI handoff context | open, waiting_guest, closed | EL-L | res.message.write (consent check) | AU-W | N-in | A-live | ML-C |
| SCR-FO-upsell-offers | Offer eligible upgrades/early-late/extras at desk with live capacity | front_desk_agent | offers, price, capacity, guest eligibility | offered, accepted, declined | EL-M | com.offer.sell | AU-W | — | A-form | ML-C |
| SCR-FO-night-audit | Night audit run: pre-checks, posting room/tax, balance, close business date, reports (SM-night-audit) | night_auditor | business date, checklist, open checks/shifts, unposted charges, room/tax posting summary, reports | ready, blocked, running, completed, failed, reopened | EL-M | fin.night_audit.run `+stepup`; reopen `+mc` | AU-L + AU-P | N-email reports | A-live | ML-D |
| SCR-FO-night-audit-exceptions | Items blocking audit: open POS checks, unbalanced windows, pending payments, no-shows | night_auditor, front_office_manager | exception, source, owner, action | open, resolved, overridden | EL-L | fin.night_audit.resolve | AU-P for override | N-in | A-grid | ML-D |
| SCR-FO-outage-registration | Reconcile paper/manual check-ins, charges and payments captured during outage | front_office_manager | manual record ref, scanned form, entered data, matched reservation, posting | captured, entered, reconciled | EL-F | res.outage.reconcile | AU-E | N-in | A-form, A-cam | ML-D |
| SCR-FO-hk-home | Housekeeper home (§3) | housekeeper | my rooms by priority, rush, redo, minibar, report buttons | — | EL-O | res.hk.task@own | AU-0 | N-push new rush | A-live | ML-N |
| SCR-FO-hk-supervisor-home | HK supervisor home (§3) | housekeeping_supervisor | assignment balance, inspections due, ready ETA vs arrivals, linen shortfall | — | EL-R | res.hk.supervise | AU-0 | N-esc | A-live | ML-T |
| SCR-FO-hk-room-board | All rooms: occupancy × HK × service state, DND, priority, attendant | housekeeping_supervisor, front desk | room, occupancy, HK state, service state, priority, DND, assigned attendant, ETA | dirty, in_progress, clean, inspected, pickup, ooo, oos | EL-R | res.hk.read | AU-0 | N-rt | A-grid, A-map | ML-T |
| SCR-FO-hk-task-assign | Assign/rebalance rooms by credits/time and zones | housekeeping_supervisor | attendants, capacity, rooms, estimated minutes | draft, published | EL-L | res.hk.assign | AU-W | N-push to attendants | A-grid, A-drag | ML-T |
| SCR-FO-hk-my-rooms | Attendant's ordered room list, offline | housekeeper | room, type of clean, priority, DND, notes, status | queued, in_progress, done, dnd_skipped | EL-O | res.hk.task@own | AU-W | N-push | A-live | ML-N |
| SCR-FO-hk-room-task | Clean checklist, start/finish, photos, linen used, amenities, fault/lost-item shortcuts | housekeeper | checklist version, start/end time, linen counts, amenity counts, notes | in_progress, done, failed_inspection | EL-O | res.hk.task@own | AU-W | N-rt to board | A-form, A-cam | ML-N |
| SCR-FO-hk-inspection | Supervisor inspection pass/fail with reasons, reclean | housekeeping_supervisor | checklist, failed items, photos, reclean assignment | pending, passed, failed | EL-O | res.hk.inspect | AU-W | N-push reclean | A-form | ML-N |
| SCR-FO-hk-dnd-log | Record DND/privacy refusal and escalation rules (welfare check) | housekeeper, duty_manager | room, time, DND duration, welfare escalation rule | logged, escalated, resolved | EL-O | res.hk.dnd | AU-W | N-esc after threshold | A-form | ML-N |
| SCR-FO-hk-minibar-count | Count minibar/amenities; one-time folio post; late charge review after checkout | housekeeper, minibar attendant | items, par, counted, consumed, price, count_id | draft, posted, late_charge_review, disputed | EL-O (posting on sync, dedup by count_id) | stk.minibar.count; fin.folio.post (system) | AU-L | N-in late charges | A-form | ML-N |
| SCR-FO-hk-linen-par | Clean/soiled linen par by floor/store, shortages for arrivals | housekeeping_supervisor, laundry_attendant | SKU, par, on hand clean, soiled, at vendor, shortfall | — | EL-L | stk.linen.read | AU-0 | N-in shortfall | A-grid | ML-C |
| SCR-FO-hk-laundry-dispatch | Outsourced laundry pickup/return with counts/weight and mismatch | laundry_attendant | batch, SKU counts, weight, vendor, signatures, returned counts | dispatched, returned, mismatch | EL-O | stk.linen.move | AU-L + AU-E (signature) | N-in mismatch | A-form, A-sig | ML-N |
| SCR-FO-hk-linen-discrepancy | Investigate linen loss/stain/damage; replacement cost; vendor claim | housekeeping_supervisor | discrepancy, batch, cost, responsibility, claim | open, claimed, written_off | EL-F | stk.linen.adjust `+mc` over limit | AU-L | N-in | A-form | ML-C |
| SCR-FO-hk-room-ready-eta | Predicted ready times vs arrival ETAs; prioritisation | housekeeping_supervisor, front desk | room, arrival ETA, predicted ready, attendant, risk | on_track, at_risk, late | EL-R | res.hk.read | AU-0 | N-in at risk | A-live | ML-T |
| SCR-FO-hk-fault-report | Quick fault report from room to engineering (creates WO, optional OOO request) | housekeeper, front desk | room, asset/category, photo, urgency, guest impact | submitted, queued | EL-O | eng.wo.create | AU-W | N-push to engineer | A-cam, A-form | ML-N |

### 4.4 FIN — finance, cashiering, AP/AR, reconciliation, utility bills, payroll approval

```
Finance
├─ Home (clerk/cashier) · Controller home
├─ Cashiering: cashier shift, cash drop/safe, refund approval, night audit review
├─ GL: chart of accounts, posting rules, posting exceptions, journals (list/detail/manual), accruals, allocations, period close, trial balance, statements, budget editor, fixed assets
├─ AP: AP inbox (capture/OCR), supplier invoice detail, 3-way match, payables, payment batch, payment release, payee bank change
├─ AR: city-ledger accounts, AR invoice, aging & collections, credit limit/write-off
├─ Treasury & reconciliation: bank statements, bank reconciliation, PSP reconciliation, chargeback case, cash forecast, cost reconciliation dashboard
├─ Operating costs: utility bill inbox/detail, bill-pay orders/detail, gas cylinder ledger
├─ Payroll approval
├─ Liabilities: loyalty liability, voucher liability, referral commission payouts, travel reconciliation
├─ Tax: invoice series, tax returns (→ ADM filing)
└─ Controls: revenue protection, anomaly case; owner fees
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-FIN-home | Finance clerk/cashier home (§3) | finance_clerk, cashier | shift status, posting exceptions, unmatched items, refund approvals, invoices to capture | — | EL-D | fin.home.read | AU-0 | N-in | A-std | ML-C |
| SCR-FIN-controller-home | Controller home (§3) | financial_controller | close checklist, approvals, recon breaks, release queue, incomplete sources | — | EL-D | fin.controller.read | AU-0 | N-esc | A-chart | ML-C |
| SCR-FIN-cashier-shift | Open/close drawer with float, blind count, variance | cashier | drawer, opening float, movements, expected vs counted by tender, variance reason | open, closing, closed, variance_review | EL-M | fin.cash_shift.operate@own | AU-L | N-in variance > threshold | A-form | ML-T |
| SCR-FIN-cash-drop-safe | Record cash drops, safe movements, bank deposit bags | cashier, financial_controller | bag number, amount, witness, safe balance | recorded, deposited, confirmed | EL-M | fin.cash.move (witness `+mc`) | AU-L + AU-E | — | A-form | ML-T |
| SCR-FIN-refund-approval | Approve refunds above limit; see original payment/capture and reason | financial_controller, finance_approver | original txn, captured, refunded so far, requested, reason, requester | pending, approved, rejected, executed, unknown | EL-M | fin.refund.approve `+mc` `+stepup` | AU-P + AU-L | N-in requester | A-form (3.3.4) | ML-C |
| SCR-FIN-night-audit-review | Finance review of audit output: revenue, tax, deposits, guest ledger | financial_controller | business date, revenue by code, tax totals, guest ledger movement, exceptions | pending_review, reviewed | EL-D | fin.night_audit.review | AU-W | — | A-grid | ML-D |
| SCR-FIN-chart-of-accounts | Maintain accounts, departments, cost centers, mappings | financial_controller | code, name EN/AR, type, control flag, department, active | draft, active, inactive | EL-L | fin.coa.write `+mc` | AU-P | — | A-grid | ML-D |
| SCR-FIN-posting-rules | Versioned mapping from events to journal lines (SF19.1.2) | financial_controller | event type, version, conditions, debit/credit accounts, cost center rule, tax lines | draft, tested, active, superseded | EL-F (test with sample event) | fin.posting_rule.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-FIN-posting-exceptions | Events not posted (missing rule, locked period, invalid account) | finance_clerk, financial_controller | event, source, amount, reason, age | open, reposted, resolved | EL-L | fin.posting.resolve | AU-W | N-in | A-grid | ML-D |
| SCR-FIN-journal-list | Search journals by date, source, account, status | finance roles, auditor | journal no, dates, source, amount, reversed flag | posted, reversed | EL-L | fin.journal.read | AU-0 | — | A-grid | ML-D |
| SCR-FIN-journal-detail | View balanced lines, source document, reversal chain; create reversal | financial_controller | lines, FX, tax rule versions, source links, reversal link | posted, reversed | EL-F; reversal EL-M | fin.journal.reverse `+mc` | AU-L + AU-P | — | A-grid | ML-D |
| SCR-FIN-manual-journal | Create manual journal with evidence; debit=credit enforced live | financial_controller, finance_clerk | lines, accounts, cost centers, amounts, evidence, reason | draft, pending_approval, posted | EL-M | fin.journal.create `+mc` | AU-P + AU-L | N-in approver | A-form, A-grid | ML-D |
| SCR-FIN-accruals-prepayments | Accrual/prepayment schedules and auto-reversals (SF19.1.4) | financial_controller | schedule, basis estimate/actual, periods, reversal date | active, completed | EL-L | fin.accrual.write | AU-L | — | A-grid | ML-D |
| SCR-FIN-allocation-rules | Shared cost allocation drivers and versions (energy, labor, laundry) | financial_controller | driver, source metric, departments, weights, version | draft, active, superseded | EL-F | fin.allocation.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-FIN-period-close | Close checklist: sub-ledgers, recon, accruals, lock | financial_controller | tasks, owners, status, blockers, lock actions | open, soft_closed, locked | EL-M | fin.period.lock `+stepup` `+mc` | AU-P | N-in task owners | A-live | ML-D |
| SCR-FIN-trial-balance | Trial balance by period/department with drill | financial_controller, auditor | account, opening, debits, credits, closing, functional currency | provisional, closed | EL-D | fin.tb.read | AU-0 | — | A-grid | ML-D |
| SCR-FIN-financial-statements | P&L, balance sheet, cash flow export | financial_controller, owner | statements, comparatives, restatement flags | provisional, final | EL-D | fin.statements.read | AU-0 | N-email scheduled | A-grid | ML-D |
| SCR-FIN-budget-editor | Budget and forecast versions by account/department/month | financial_controller, gm | version, lines, drivers, approval | draft, approved, locked | EL-F | fin.budget.write `+mc` | AU-P | — | A-grid | ML-D |
| SCR-FIN-fixed-assets | Capitalization, depreciation, disposal (M66) | financial_controller | asset, cost, life, method, depreciation schedule, linked capex/PO | active, disposed | EL-L | fin.asset.write | AU-L | — | A-grid | ML-D |
| SCR-FIN-ap-inbox | Capture supplier invoices (upload/email/EDI/OCR) and queue for match | ap_clerk | source file, OCR fields with confidence, vendor, invoice no, dates, totals, duplicate flag | captured, needs_review, ready_to_match, duplicate_suspected | EL-L, EL-C | fin.ap.capture | AU-E (source evidence) | N-in duplicates | A-grid, A-form | ML-D |
| SCR-FIN-supplier-invoice-detail | Review invoice, match to PO/GRN/service acceptance, hold/dispute/approve | ap_clerk, finance_approver | header, lines, PO/receipt matches, tolerance results, tax, attachments | unmatched, matched, tolerance_exception, on_hold, disputed, approved | EL-F | fin.ap.match, .approve (+limit) | AU-P for approve | N-in approver/vendor | A-form, A-grid | ML-D |
| SCR-FIN-three-way-match | Side-by-side PO vs receipt vs invoice lines with tolerances | ap_clerk | qty/price/tax per line, variances, tolerance rule | matched, variance | EL-L | fin.ap.match | AU-W | — | A-grid | ML-D |
| SCR-FIN-payables-list | All payables by status: planned, accrued, invoiced, approved, scheduled, paid, settled | ap_clerk, financial_controller | payable, category (utility/gas/payroll/maintenance/supplier/commission), due, status, evidence | per §D states | EL-L | fin.payable.read | AU-0 | — | A-grid | ML-D |
| SCR-FIN-payment-batch | Build payment batch from approved payables | ap_clerk | payables, totals by bank/currency, payee account status (verified/frozen) | draft, submitted, approved, released, sent, partially_rejected, confirmed | EL-M | fin.batch.create | AU-P | N-in approver | A-grid | ML-D |
| SCR-FIN-payment-release | Approve and release batch (distinct approver and releaser, step-up) | finance_approver, payment_releaser | batch, totals, SoD check, bank channel honesty label | approved, released, unknown | EL-M | fin.batch.approve `+mc`; fin.batch.release `+stepup` | AU-P + AU-L | N-in, N-email vendor on payment | A-form (3.3.4) | ML-C |
| SCR-FIN-payee-bank-change | Maker-checker change of vendor/employee payee bank details; payout freeze | ap_clerk, finance_approver | old/new (masked), evidence, callback verification, freeze until | requested, verified, approved, rejected | EL-F | fin.payee.change `+mc` `+stepup` | AU-P + AU-R | N-email out-of-band to vendor admin | A-form | ML-D |
| SCR-FIN-ar-accounts | Corporate/city-ledger accounts: balance, limit, aging | ar_clerk | account, credit limit, balance, overdue, hold flag | active, on_hold | EL-L | fin.ar.read | AU-0 | — | A-grid | ML-D |
| SCR-FIN-ar-invoice | Issue AR invoice/statement from folios/events; adjustments | ar_clerk | invoice, lines, PO no, cost center, tax, attachments | draft, issued, partially_paid, paid, credited | EL-F | fin.ar.invoice | AU-L | N-email corporate | A-form | ML-D |
| SCR-FIN-ar-aging-collections | Aging buckets, dunning actions, promises to pay | ar_clerk | account, buckets, last contact, next action | — | EL-L | fin.ar.collect | AU-W | N-email dunning | A-grid | ML-D |
| SCR-FIN-credit-limit-writeoff | Credit limit change and write-off approvals | financial_controller | account, current/new limit, write-off amount, reason | requested, approved, rejected | EL-M | fin.ar.writeoff `+mc` `+stepup` | AU-P + AU-L | N-in | A-form | ML-D |
| SCR-FIN-bank-statements | Import bank statements (file/API), view lines | finance_clerk | account, statement date, lines, import source, duplicates | imported, partially_matched, matched | EL-L, EL-C | fin.bank.import | AU-W | N-in import failures | A-grid | ML-D |
| SCR-FIN-bank-reconciliation | Match bank lines to receipts/disbursements/PSP settlements | finance_clerk, financial_controller | line, candidates, rule, manual match, unmatched age | unmatched, matched, exception | EL-L | fin.recon.match | AU-W | N-in unmatched aged | A-grid | ML-D |
| SCR-FIN-psp-reconciliation | PSP settlement files vs captures/refunds/fees/chargebacks | finance_clerk | settlement batch, gross, fees, net, per-txn match, missing items | pending, matched, exception | EL-L | fin.recon.psp | AU-W | N-in | A-grid | ML-D |
| SCR-FIN-chargeback-case | Chargeback evidence and response | finance_clerk, financial_controller | txn, reason code, deadline, evidence (folio, signature, ID-free receipt) | received, evidence_submitted, won, lost | EL-F, A-time | fin.chargeback.respond | AU-E | N-esc deadline | A-form | ML-D |
| SCR-FIN-cash-forecast | 13-week cash forecast from AR, AP, payroll, taxes | financial_controller | inflows/outflows by week, scenario | draft, published | EL-D | fin.cash_forecast.read | AU-0 | — | A-chart | ML-D |
| SCR-FIN-cost-reconciliation | Reconciliation dashboard showing planned/accrued/invoiced/approved/paid/settled per cost category (§D) | financial_controller, gm | category, amounts per stage, breaks, evidence coverage | — | EL-D | fin.recon.read | AU-0 | N-in breaks | A-chart, A-grid | ML-D |
| SCR-FIN-utility-bill-inbox | Utility bills (electricity/water/gas) inbox with meter variance and due dates (critical flow W9) | ap_clerk, chief_engineer | provider, account (masked), period, total, due, kWh/m3 billed vs metered, variance %, tariff version, duplicate flag, status | captured, validated, variance_review, sent_to_ap, approved, paid, settled | EL-L, EL-C | fin.utility_bill.review | AU-E | N-in variance/due soon, N-esc | A-grid | ML-D |
| SCR-FIN-utility-bill-detail | One bill: lines (supply/sewage/standing), meter reconciliation, allocation preview, approve to AP | ap_clerk, chief_engineer, finance_approver | bill lines, readings, estimated/actual, allocation by dept, evidence PDF | validated, variance_review, approved | EL-F | fin.utility_bill.approve (+limit) | AU-P | N-in | A-grid, A-form | ML-D |
| SCR-FIN-billpay-orders | List bill-provider orders (Khedmah/ONEIC or bank) with status | finance_clerk | provider, biller, account, amount, fee, status, provider ref, honesty label | quoted, submitted, pending, confirmed, failed, disputed | EL-L | fin.billpay.read | AU-0 | N-in pending > SLA | A-grid | ML-D |
| SCR-FIN-bill-payment-detail | Inquire → quote → confirm → pay → status; manual evidence path when adapter blocked (SF29.1.x) | finance_approver, billpay_worker | biller, account ref, bill amount/expiry, fee, idempotency key, status log, receipt no, manual evidence | quoted, authorized, submitted, pending, confirmed, failed, disputed, reversed | EL-M (inquire before retry) | fin.billpay.pay `+mc` `+stepup` | AU-P + AU-L | N-in status change | A-form (3.3.4), A-time | ML-C |
| SCR-FIN-gas-cylinder-ledger | Cylinder custody ledger: full/empty/with vendor/deposit/lost with PO/invoice links | storekeeper, finance_clerk | serial/type, state, location, deposit, movements, PO, invoice | per custody state | EL-L | stk.cylinder.read; adjust `+mc` | AU-L | N-in reorder point | A-grid | ML-C |
| SCR-FIN-payroll-approval | Finance approval of payroll run totals and release of bank/WPS file (aggregates; individual lines only for payroll roles) | finance_approver, payroll_approver | run, totals by dept, employer liabilities, exceptions count, bank file hash, honesty label of bank channel | approved_hr, approved_finance, released | EL-M | wrk.payroll.approve_finance `+mc` `+stepup` | AU-P | N-in payroll_officer | A-form | ML-C |
| SCR-FIN-loyalty-liability | Points issuance, redemption, breakage, liability postings (SF30.1.7) | financial_controller | points by state, valuation rate, liability, breakage estimate | — | EL-D | fin.loyalty.read | AU-0 | — | A-chart | ML-D |
| SCR-FIN-voucher-liability | Gift voucher liability, redemption, expiry | financial_controller | vouchers issued/redeemed/expired, balances | — | EL-D | fin.voucher.read | AU-0 | — | A-grid | ML-D |
| SCR-FIN-commission-payouts | Referral commission approvals and payouts per market gate (F31.1) | finance_approver, compliance_officer, referral_program_admin | referrer, booking id (masked), formula version, margin components, commission, gate status, tax docs | pending, approved, blocked_by_gate, paid, reversed | EL-M | loy.commission.approve `+mc`; payout via batch | AU-P + AU-L | N-email referrer statement | A-grid | ML-D |
| SCR-FIN-travel-reconciliation | Travel orders: supplier payable/receivable, commission, refunds (SF45.2.8) | finance_clerk | order, provider, guest charge, supplier cost, commission, refund status | open, reconciled, exception | EL-L | fin.travel.recon | AU-W | — | A-grid | ML-D |
| SCR-FIN-invoice-series | Fiscal numbering series per legal entity/JUR rule | financial_controller | series, prefix, next no, rule version, device/branch | active, closed | EL-F | fin.invoice_series.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-FIN-tax-returns | Tax period summaries per JUR rule pack; hand-off to filing (SCR-ADM-filing-submission) | financial_controller, compliance_officer | tax period, output/input tax, levies, rule versions, reconciliation to GL | draft, reconciled, approved, filed (link) | EL-D | fin.tax.prepare | AU-P | N-in due | A-grid | ML-D |
| SCR-FIN-revenue-protection | Anomaly dashboard: duplicates, void/comp outliers, commission mismatch, split orders (M60) | financial_controller, auditor | rule, hits, value, false-positive rate | — | EL-D | fin.control.read | AU-R | N-in | A-chart | ML-D |
| SCR-FIN-anomaly-case | Investigate anomaly case with evidence and privacy limits | financial_controller | case, evidence, involved staff (restricted), outcome, feedback | open, investigating, closed_confirmed, closed_false_positive | EL-F | fin.control.investigate | AU-P + AU-R | N-in | A-form | ML-D |
| SCR-FIN-owner-fees | Management/brand/franchise fee calculation and approval (M66) | financial_controller | fee schedule version, base, fee, approval | calculated, approved, invoiced | EL-F | fin.owner_fee.approve `+mc` | AU-P | — | A-form | ML-D |

### 4.5 HR — HR, payroll and employee self-service

```
HR (web)                                    Employee self-service (staff app)
├─ HR home · Payroll home                   ├─ ESS home ── schedule ── clock
├─ Employees: list, profile, sensitive,     ├─ leave ── payslips ── training
│   contract & compensation, joiners/leavers
├─ Time: roster planner, shift swap, attendance, leave, overtime, labor forecast
├─ Payroll: run, preview, exceptions, WPS/bank file, rejections, payment status, payslips, statutory forms, labor cost
└─ Enablement: training matrix, certifications, SOP library, quality sampling
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-HR-home | HR officer home (§3) | hr_officer | joiners/leavers, expiring contracts/certs, leave approvals, roster gaps | — | EL-D | wrk.hr.read | AU-0 | N-in | A-std | ML-C |
| SCR-HR-payroll-home | Payroll home (§3) | payroll_officer, payroll_approver | run status, exceptions, approvals, rejections, filings due | — | EL-D | wrk.payroll.read | AU-0 | N-in | A-std | ML-C |
| SCR-HR-employee-list | Employee directory (HR view) | hr_officer | name, employee no, department, role, status, contract end | active, leaving, left | EL-L | wrk.employee.read@dept | AU-0 | — | A-grid | ML-C |
| SCR-HR-employee-profile | Employee master: contract, role, location, documents (non-sensitive) | hr_officer, dept manager (limited) | personal (non-restricted), department, position, work jurisdiction, contacts | active, leaving, left | EL-F | wrk.employee.write | AU-W | — | A-form | ML-C |
| SCR-HR-employee-sensitive | Reveal/edit SIN/national ID/bank/DOB with reason (F27.1.3, SF38.2.1) | hr_officer, payroll_officer | masked values, reveal button, reason, validation (e.g. SIN checksum), jurisdiction id type | masked, revealed (timed), edited | EL-F | wrk.employee.sensitive `+stepup`; field:national_id, field:bank | AU-R + AU-P | N-email to DPO on bulk | A-form, A-time | ML-D |
| SCR-HR-contract-compensation | Effective-dated contracts, compensation, allowances | hr_officer, payroll_officer | contract type, dates, base pay (encrypted), allowances, work jurisdiction (e.g. CA-QC) | draft, approved, active, ended | EL-F | wrk.compensation.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-HR-onboarding-offboarding | Joiner/leaver checklist incl. access provisioning/revocation, final settlement | hr_officer | tasks, IT access, uniform/keys, final pay items | open, completed | EL-L | wrk.lifecycle.manage | AU-W | N-in IT/security | A-form | ML-C |
| SCR-HR-roster-planner | Build rosters by department/outlet vs labor demand | dept managers, hr_officer | shifts, roles, skills, cost center, event attribution, demand forecast, overlaps prevented | draft, published | EL-L | wrk.roster.write@dept | AU-W | N-push published shifts | A-grid, A-drag | ML-T |
| SCR-HR-shift-swap | Request/approve shift swaps with skill/hour rules | employee, dept manager | shift, counterpart, rule checks | requested, approved, rejected | EL-F | wrk.roster.swap | AU-W | N-push | A-form | ML-P |
| SCR-HR-time-attendance | Punches vs roster, exceptions, corrections | dept manager, hr_officer | punches, source, skew flag, variance, corrections with reason | pending_review, approved | EL-L | wrk.time.approve@dept | AU-W | N-in exceptions | A-grid | ML-C |
| SCR-HR-leave-requests | Leave/absence approvals with balance and coverage impact | dept manager, hr_officer | type, dates, balance, coverage gap | requested, approved, rejected, cancelled | EL-L | wrk.leave.approve@dept | AU-W | N-push employee | A-grid | ML-C |
| SCR-HR-overtime-approval | Approve overtime/holiday premiums | dept manager | employee, hours, reason, rate type | requested, approved, rejected | EL-L | wrk.overtime.approve@dept | AU-W | N-push | A-grid | ML-C |
| SCR-HR-labor-forecast | Labor demand vs roster and OT risk (F62.2) | dept manager, gm | forecast demand by hour, scheduled hours, gap, cost | — | EL-D | wrk.labor.read@dept | AU-0 | — | A-chart | ML-D |
| SCR-HR-payroll-run | Create/calculate run for period with statutory rule versions (JUR) | payroll_officer | period, population, rule versions (CPP/QPP/EI/QPIP/income tax or other market), status | draft, calculated, exceptions, approved_hr, approved_finance, file_generated, sent, partially_rejected, paid, closed | EL-M | wrk.payroll.run | AU-P | N-in approvers | A-live | ML-D |
| SCR-HR-payroll-preview | Gross-to-net preview per employee and totals | payroll_officer, payroll_approver | earnings, deductions, employer contributions, net, variance vs last period | calculated | EL-L | wrk.payroll.view_individual; field:salary | AU-R | — | A-grid | ML-D |
| SCR-HR-payroll-exceptions | Exceptions (missing bank, negative net, rule unknown/blocked, large variance) | payroll_officer | exception, employee, rule, action | open, resolved | EL-L | wrk.payroll.resolve | AU-W | N-in | A-grid | ML-D |
| SCR-HR-wps-bank-file | Generate/export WPS/bank salary file; hash; upload/API status (honesty label) | payroll_officer | format, file hash, totals, channel (API/portal/manual), submission evidence | generated, submitted, acknowledged, rejected | EL-M | wrk.payroll.file `+stepup` | AU-P + AU-E | N-in | A-std | ML-D |
| SCR-HR-bank-rejections | Handle bank/WPS rejections and resubmission (SF27.3.7) | payroll_officer | rejected employees, reason, corrected details, resubmission file | open, resubmitted, resolved | EL-M | wrk.payroll.resubmit `+mc` | AU-P | N-in, N-esc | A-grid | ML-D |
| SCR-HR-payment-status | Salary payment status per employee/batch (approved/sent/confirmed) | payroll_officer, payroll_approver | batch, employee (restricted), status, bank ref | sent, confirmed, rejected | EL-L | wrk.payroll.status | AU-R | N-push employee when paid | A-grid | ML-D |
| SCR-HR-payslips | Generate/publish payslips; access log | payroll_officer | run, payslip PDF (EN/AR), publication status | generated, published | EL-L | wrk.payslip.publish | AU-R | N-push employee | A-std | ML-D |
| SCR-HR-statutory-forms | Year-end/statutory artefacts (e.g. T4/RL-1, other market forms) and filing hand-off | payroll_officer, compliance_officer | form type, year, employees, validation, file, submission mode, receipt | draft, validated, filed, amended | EL-M | wrk.statutory.prepare; filing via ADM | AU-P | N-in due | A-grid | ML-D |
| SCR-HR-labor-cost | Labor cost by department/cost center/event (aggregates only) | hr_officer, financial_controller | hours, cost, OT %, allocation | provisional, approved | EL-D | wrk.labor_cost.read | AU-0 | — | A-chart | ML-D |
| SCR-HR-training-matrix | Role × SOP/training requirements, completion, language (F62.1) | hr_officer, dept manager | role, requirement, due, completion, language | assigned, completed, overdue | EL-L | wrk.training.read@dept | AU-0 | N-push due | A-grid | ML-C |
| SCR-HR-certifications | Certifications (food safety, first aid, driver) with expiry and revocation on lapse | hr_officer | cert type, number, expiry, evidence | valid, expiring, expired, revoked | EL-L | wrk.cert.write | AU-W | N-in expiry, N-push holder | A-grid | ML-C |
| SCR-HR-sop-library | Versioned SOPs by department/language with attestation tracking | hr_officer, dept managers | SOP, version, languages, owner, review date | draft, approved, superseded | EL-L | wrk.sop.write | AU-W | N-push new version to roles | A-media | ML-C |
| SCR-HR-quality-sampling | Service quality sampling and coaching with privacy boundaries (F62.2) | dept manager | sample, criteria, score, coaching note (restricted), dispute | draft, shared, disputed, closed | EL-F | wrk.quality.write@dept | AU-R | N-push employee | A-form | ML-C |
| SCR-HR-ess-home | Employee self-service home (§3) | employee | next shifts, clock, leave balance, payslips, training due | — | EL-O | wrk.ess@own | AU-0 | N-push | A-std | ML-N |
| SCR-HR-ess-schedule | My schedule; swap request | employee | shifts, location, role | published | EL-O | wrk.ess@own | AU-0 | N-push | A-std | ML-N |
| SCR-HR-ess-clock | Clock in/out/break (geofence/device optional, offline queue) | employee | time, source, location check result | clocked_out, clocked_in, on_break, queued | EL-O | wrk.time.punch@own | AU-W | — | A-live | ML-N |
| SCR-HR-ess-leave | Request leave; balances | employee | type, dates, balance | requested, approved, rejected | EL-O | wrk.leave.request@own | AU-W | N-push decision | A-form | ML-N |
| SCR-HR-ess-payslip | View/download own payslips (step-up) | employee | period, gross, net, lines, PDF | published | EL-F | wrk.payslip.read@own `+stepup` | AU-R | — | A-std | ML-N |
| SCR-HR-ess-training | My training, SOP attestation, certification upload | employee | assigned SOPs, quizzes, attest button, cert upload | assigned, completed | EL-O | wrk.training@own | AU-W | N-push | A-media, A-form | ML-N |

### 4.6 ENG — engineering, assets, meters, utilities, contractors

```
Engineering
├─ Home (engineer, mobile offline) · Chief engineer home
├─ Assets: registry, asset detail, asset/meter map, warranty, capex project
├─ Work: WO board, WO detail, my jobs, preventive schedule, room OOO request, room release inspection, requisition
├─ Contractors: assignments, service acceptance, vendor invoice status
└─ Utilities: meter readings, meter import, utility accounts, tariffs, anomalies, pipeline gas, cylinder stock, sustainability baseline
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-ENG-home | Engineer home (§3) | engineer | my jobs by SLA, emergencies, PM due, rooms awaiting inspection, reads due | — | EL-O | eng.home.read@own | AU-0 | N-push | A-live | ML-N |
| SCR-ENG-chief-home | Chief engineer home (§3) | chief_engineer | SLA breaches, OOO rooms, contractor acceptance queue, anomalies, parts | — | EL-D | eng.chief.read | AU-0 | N-esc | A-chart | ML-C |
| SCR-ENG-asset-registry | Asset hierarchy and criticality (SF26.1.1) | chief_engineer | asset, location, parent, criticality, warranty, vendor | active, retired | EL-L | eng.asset.write | AU-W | — | A-grid | ML-C |
| SCR-ENG-asset-detail | Asset history, WOs, parts, warranty, documents, meters | engineer, chief_engineer | specs, WO history, cost to date, warranty expiry | active, under_repair, retired | EL-F | eng.asset.read | AU-W | — | A-form | ML-C |
| SCR-ENG-asset-meter-map | Floor/site map of assets and meters with status overlays | chief_engineer | map layers, asset/meter pins, alerts | — | EL-R | eng.map.read | AU-0 | N-rt | A-map (list alternative) | ML-T |
| SCR-ENG-warranty | Warranty claims and expiries | chief_engineer | asset, vendor, expiry, claim status | valid, expiring, claimed | EL-L | eng.warranty.write | AU-W | N-in expiry | A-grid | ML-C |
| SCR-ENG-capex-project | Renovation/capex project tracking linked to rooms and assets (SF66.2.x) | chief_engineer, gm | project, budget, rooms closed/resale dates, POs, progress | planned, in_progress, completed | EL-F | eng.capex.write | AU-W | N-in FO on room dates | A-form | ML-D |
| SCR-ENG-work-order-board | All WOs by priority/SLA/status; dispatch | chief_engineer, engineer | WO, origin (guest/PM/IoT/meter/inspection), priority, SLA due, assignee, room impact | open, assigned, in_progress, awaiting_parts, done, inspected, closed | EL-R | eng.wo.dispatch | AU-W | N-push assignee, N-esc | A-grid, A-drag | ML-T |
| SCR-ENG-work-order-detail | Work a WO: diagnosis, parts, labor, photos, guest impact, vendor handoff | engineer | description, photos, parts used (stock issue), labor time, notes, completion evidence | per WO states | EL-O | eng.wo.update@own | AU-W | N-rt FO when room affected | A-form, A-cam | ML-N |
| SCR-ENG-my-jobs | Mobile list of assigned jobs, offline | engineer | job, room/asset, SLA countdown, status | per WO states | EL-O | eng.wo@own | AU-W | N-push | A-live | ML-N |
| SCR-ENG-preventive-schedule | PM plans, frequencies, generated tasks (SF26.1.2) | chief_engineer | plan, asset, frequency, next due, checklist | active, paused | EL-L | eng.pm.write | AU-W | N-in due | A-grid | ML-D |
| SCR-ENG-room-ooo-request | Request room out-of-order/out-of-service with dates; FO/revenue sees impact | engineer, chief_engineer | room, reason, from/to, OOO vs OOS, inventory impact | requested, approved, active, released | EL-M (inventory change needs server) | inv.room.ooo_request; approve by FO manager | AU-W | N-in FO manager, revenue | A-form | ML-C |
| SCR-ENG-room-release-inspection | Inspect and return room to service (SF26.1.6) | engineer, housekeeping_supervisor | checklist, photos, test results | pending, passed, failed | EL-O | eng.room.release | AU-W | N-rt FO/HK | A-form | ML-N |
| SCR-ENG-requisition | Raise maintenance materials/service requisition (→ PRC) | engineer, chief_engineer | items/services, qty, WO link, cost center, urgency | draft, submitted | EL-F | stk.requisition.create@dept | AU-W | N-in procurement | A-form | ML-C |
| SCR-ENG-contractor-assignments | Assign vendor jobs (eligible vendors only), track site visit | chief_engineer | WO, vendor (eligible), quote, PO, visit schedule, site access | proposed, assigned, on_site, completed | EL-L | eng.vendor_job.assign | AU-W | N-push vendor (VAPP) | A-grid | ML-C |
| SCR-ENG-service-acceptance | Review contractor evidence (photos, labor, parts) and accept/reject service (SF21.2.2) | chief_engineer | evidence, checklist, warranty, variance vs quote | submitted, accepted, rejected, disputed | EL-F | eng.service.accept (≠ requester) | AU-E | N-push vendor | A-form, A-cam | ML-C |
| SCR-ENG-vendor-invoice-status | Status of contractor invoices through AP | chief_engineer | invoice, PO, match status, payment status | per AP states | EL-L | fin.ap.read@dept | AU-0 | — | A-grid | ML-C |
| SCR-ENG-meter-readings | Readings by meter with quality flags; manual read entry | engineer, chief_engineer | meter, read_at, value, quality (actual/estimated/reset/gap), source | — | EL-O (manual read) | eng.meter.read / .write | AU-W | N-in gaps | A-grid, A-chart | ML-C |
| SCR-ENG-meter-import | Import BMS/API/CSV readings; mapping; gap/reset handling (SF22.1.2–4) | chief_engineer, integration_admin | file/source, mapping, rows, rejected rows, timezone handling | uploaded, validated, imported, failed | EL-C, EL-L | eng.meter.import | AU-W | N-in failures | A-grid | ML-D |
| SCR-ENG-utility-accounts | Utility accounts, service addresses, meters served, providers | chief_engineer, ap_clerk | provider, account (masked), meters, billing cycle | active, closed | EL-L | eng.utility_account.write | AU-W | — | A-grid | ML-D |
| SCR-ENG-tariffs | Tariff versions, peak windows, standing charges | chief_engineer, financial_controller | tariff, bands, valid dates, source evidence | draft, active, superseded | EL-F | eng.tariff.write | AU-W | — | A-form | ML-D |
| SCR-ENG-consumption-anomalies | Leaks/abnormal use detection → WO (SF23.1.4, SF67.2.2) | chief_engineer | meter, expected vs actual, confidence, linked WO | open, work_ordered, dismissed | EL-L | eng.anomaly.triage | AU-W | N-in, N-push | A-chart | ML-C |
| SCR-ENG-gas-pipeline | Pipeline gas accounts, meter/billed consumption, safety faults (M24) | chief_engineer | account, consumption, tariff, fault refs, allocation to kitchen/laundry | — | EL-D | eng.gas.read | AU-0 | N-push safety fault | A-chart | ML-C |
| SCR-ENG-cylinder-stock | Cylinder custody view for engineering safety (full/empty/in use/with vendor) | chief_engineer, storekeeper | serial, size, state, location, safety status | per custody state | EL-L | stk.cylinder.read | AU-0 | N-in low stock | A-grid | ML-C |
| SCR-ENG-sustainability-baseline | Baselines, targets and evidence for energy/water/gas/waste (M67) | chief_engineer, gm | metric, baseline period, target, normalisation, evidence status | draft, approved | EL-F | eng.sustainability.write | AU-W | — | A-form, A-chart | ML-D |

### 4.7 VEN — vendor self-service web

```
Vendor web (vendor realm; same capabilities as VAPP, desktop-friendly for bulk work)
├─ Register: landing ── wizard (legal entity, categories, locations) ── documents ── verification status
├─ Home ── team users ── profile & locations ── categories
├─ Catalog: list ── item/variant editor ── bulk import ── daily stock ── price list ── contracts & SLA
├─ Sourcing: RFQ inbox ── bid submit ── Q&A
├─ Orders: PO list ── PO detail (ack/change) ── ASN dispatch ── invoice submit ── payment status
├─ Jobs (maintenance/services): assigned jobs ── job evidence
├─ Emergency chef: availability ── callout offer
├─ Travel providers: order queue (manual RFQ response)
└─ Performance ── messages & disputes
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-VEN-landing-register | Public entry: what hotel needs, eligibility, start registration or accept invitation | prospective vendor | invitation code, country, category interest, privacy notice | — | EL-P | public | AU-0 | — | A-std, A-help | ML-P |
| SCR-VEN-registration-wizard | Legal entity, owners/contacts, categories/subcategories, locations/service radius, tax id, consent | vendor_admin | legal name, registration no, country, tax id, categories, coverage, contacts, conflict-of-interest declaration | draft, submitted | EL-F (autosave) | vnd.registration@own | AU-W | N-email confirmation | A-form (3.3.7 no re-entry) | ML-P |
| SCR-VEN-document-upload | Upload category-required credentials with expiry (SF46.1.3) | vendor_admin | doc type, file, number, issuer, expiry, jurisdiction | missing, uploaded, scanning, verified, rejected, expiring, expired | EL-C | vnd.credential.upload@own | AU-E | N-email expiry reminders | A-form | ML-P |
| SCR-VEN-verification-status | Track verification/approval per category and property with reasons | vendor_admin | category, property, status, reviewer notes (vendor-safe), next steps | pending, under_review, approved, rejected, suspended | EL-L | vnd.status.read@own | AU-0 | N-email/N-push status change | A-live | ML-P |
| SCR-VEN-home | Vendor home (§3) | vendor_admin, vendor_user | tasks: stock update due, RFQs, POs to ack, dispatches, expiring docs, payments | — | EL-D | vnd.home@own | AU-0 | N-in | A-std | ML-P |
| SCR-VEN-team-users | Manage team accounts, roles, MFA | vendor_admin | users, roles, MFA status, last login | invited, active, disabled | EL-L | vnd.user.manage@own `+stepup` | AU-P | N-email invites | A-grid | ML-C |
| SCR-VEN-profile-locations | Company profile, depots, delivery zones and slots, hours, languages | vendor_admin | addresses, geo coverage, slots, capacity, languages | draft, published | EL-F | vnd.profile.write@own | AU-W | — | A-form, A-map | ML-P |
| SCR-VEN-categories | Request additional categories; see requirements per category | vendor_admin | category, required documents, status | requested, approved, rejected | EL-L | vnd.category.request@own | AU-W | N-email | A-std | ML-P |
| SCR-VEN-catalog | Catalog items and variants with approval status and crosswalk hints | vendor_admin, vendor_user | item, category, variants count, price basis, status | draft, submitted, approved, rejected, inactive | EL-L | vnd.catalog.read@own | AU-0 | — | A-grid | ML-C |
| SCR-VEN-catalog-item-editor | Item/service with variants (fresh/frozen/pulp/powder; cuts; linen specs; printing/custom packing; per-job maintenance) (F48.2) | vendor_user | name EN/AR, category, variant attributes, pack & UOM conversion, allergens, origin, images, spec sheets, printing options (method, setup charge, MOQ, lead time), job scope/exclusions/urgent surcharge | draft, submitted, approved, rejected | EL-F, EL-C | vnd.catalog.write@own | AU-W | N-in hotel reviewer | A-form | ML-P |
| SCR-VEN-bulk-import | CSV/API bulk import of catalog, prices and stock with validation report (SF48.3.5) | vendor_admin | file, mapping, errors per row, change preview | uploaded, validated, applied, failed | EL-C, EL-L | vnd.catalog.import@own | AU-W | N-email result | A-grid | ML-D |
| SCR-VEN-daily-stock | Update today's available stock per variant with as-of time; stale warning | vendor_user | variant, on hand, reserved, available today, as-of (vendor TZ), delivery slots | fresh, stale | EL-F | vnd.stock.write@own | AU-W (history) | N-push daily reminder | A-form, A-grid | ML-P |
| SCR-VEN-price-list | Per-unit/tier/contract prices, tax, freight, deposit, validity | vendor_admin | variant, unit, tiers, currency, valid from/to, contract ref | draft, published, expired | EL-L | vnd.price.write@own | AU-W | — | A-grid | ML-C |
| SCR-VEN-contracts-sla | View contracts, SLAs, rate cards and blackout dates | vendor_admin | contract, validity, SLA terms, rate card | active, expiring, expired | EL-L | vnd.contract.read@own | AU-0 | N-email renewal | A-std | ML-C |
| SCR-VEN-rfq-inbox | RFQs invited to, with deadlines and requirements; no competitor data | vendor_admin, vendor_user | RFQ, items/spec, qty, delivery, deadline, sealed flag, sample request | invited, viewed, bid_submitted, no_bid, closed, awarded, not_awarded | EL-L | vnd.rfq.read@own | AU-0 | N-push/N-email invite and reminders | A-grid, A-time | ML-C |
| SCR-VEN-bid-submit | Submit/revise bid with prices, lead time, alternatives and sample photos | vendor_admin | line prices, UOM, pack, lead time, validity, alternatives, sample upload, terms acceptance | draft, submitted, revised, withdrawn | EL-M (server ack required; offline draft only) | vnd.bid.submit@own `+stepup` for vendor_admin policy | AU-W + AU-E (samples) | N-email receipt | A-form (3.3.4), A-time | ML-P |
| SCR-VEN-rfq-qa | Clarification questions and versioned addenda | vendor_user | question, answer, addendum version (shared to all invitees anonymised) | open, answered | EL-L | vnd.rfq.qa@own | AU-W | N-email addenda | A-std | ML-P |
| SCR-VEN-po-list | Purchase orders with status and milestones | vendor_admin, vendor_user | PO no, date, value, delivery window, status, milestone due | issued, acknowledged, changed, partially_received, received, closed, cancelled | EL-L | vnd.po.read@own | AU-0 | N-push new PO | A-grid | ML-C |
| SCR-VEN-po-detail | Acknowledge/reject PO, request change, view version history | vendor_admin | PO version, lines, delivery/quality/temperature requirements, sample ref, payment terms | issued, acknowledged, change_requested, rejected | EL-M | vnd.po.acknowledge@own | AU-W | N-in procurement | A-form | ML-P |
| SCR-VEN-asn-dispatch | Create ASN: packs, qty, lots, expiry, vehicle, dispatch temperature, labels (SF50.1.2) | vendor_user | lines, lot codes, expiry, weights, vehicle/driver, ETA, cold-chain evidence, printable PO/ASN labels (QR/GS1) | draft, dispatched, arrived, received | EL-F, EL-C | vnd.asn.create@own | AU-E | N-in receiver (ETA) | A-form | ML-P |
| SCR-VEN-invoice-submit | Submit invoice against PO/GRN/job with PDF | vendor_admin | invoice no, date, lines from GRN, tax, attachment | draft, submitted, matched, disputed, approved, paid | EL-M | vnd.invoice.submit@own | AU-E | N-email status | A-form | ML-P |
| SCR-VEN-payment-status | Payment status per invoice (approved/scheduled/paid/settled) and remittance | vendor_admin | invoice, amount, status, payment date, remittance ref | per AP states | EL-L | vnd.payment.read@own | AU-0 | N-email on payment | A-grid | ML-C |
| SCR-VEN-assigned-jobs | Maintenance/service jobs assigned to vendor only (SF26.2.2) | vendor_user (contractor) | job, site, access window, scope, contact (minimum guest data) | assigned, accepted, on_site, completed, disputed | EL-L | vnd.job.read@own | AU-0 | N-push new job | A-grid | ML-C |
| SCR-VEN-job-evidence | Arrival, labor, parts, photos, completion sign-off request | vendor_user | arrival time, labor hours, parts, before/after photos, notes | in_progress, submitted | EL-C | vnd.job.evidence@own | AU-E | N-in chief engineer | A-form, A-cam | ML-P |
| SCR-VEN-emergency-chef-availability | Emergency chef provider maintains availability, response time, credentials | vendor_user (emergency_chef) | availability calendar, radius, response minutes, food safety cert expiry, rate | available, unavailable | EL-F | vnd.availability.write@own | AU-W | N-push reminders | A-form, A-drag | ML-P |
| SCR-VEN-callout-offer | View callout offer and accept/decline before deadline (first valid acceptance wins) | emergency_chef | service, outlet, time window, cuisine/allergen notes (minimal), rate, deadline | offered, accepted, declined, filled_by_other, expired | EL-M (atomic accept) | vnd.callout.respond@own | AU-W | N-push/N-sms/N-wa/N-voice offer | A-time, A-live | ML-P |
| SCR-VEN-travel-order-queue | Travel/transport providers answer manual RFQs, confirm bookings with references | vendor_user (travel provider) | request (minimum traveler data), quote fields, expiry, booking reference, voucher upload | quote_requested, quoted, booked, changed, cancelled | EL-F | vnd.travel.respond@own | AU-E | N-email/N-push | A-form, A-time | ML-P |
| SCR-VEN-performance | Own scorecard: OTIF, defects, response time, ratings with explanation and dispute | vendor_admin | metrics, sample size, confidence, disputes | — | EL-D | vnd.performance.read@own | AU-0 | — | A-chart | ML-C |
| SCR-VEN-messages-disputes | Messages with hotel, disputes on rejections/deductions | vendor_admin, vendor_user | thread, related PO/invoice/bid, attachments | open, resolved | EL-L | vnd.message@own | AU-W | N-push | A-std | ML-C |

### 4.8 VAPP — vendor Android/iOS app

Parity principle: every vendor journey in VEN is available in VAPP except bulk CSV import (web only). Offline: drafts only; submit/accept/confirm require server acknowledgement (SF48.1.5, `docs/03` §8.2).

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-VAPP-onboarding | Language choice, value explanation, sign in / register / accept invitation | vendor users | locale, invitation link | — | EL-P | public | AU-0 | — | A-std | ML-N |
| SCR-VAPP-sign-in | Vendor realm sign-in with MFA (vendor_admin mandatory) | vendor users | email/phone, passkey/OTP | signed_out, mfa_required, locked | EL-F | public | AU-W | — | A-auth | ML-N |
| SCR-VAPP-home | Vendor home (§3) with tasks and inbox | vendor users | tasks, deadlines, alerts, offline draft count | — | EL-O | vnd.home@own | AU-0 | N-push | A-live | ML-N |
| SCR-VAPP-registration | Registration wizard (mirrors SCR-VEN-registration-wizard) | vendor_admin | as VEN | draft, submitted | EL-O (draft) | vnd.registration@own | AU-W | N-push | A-form | ML-N |
| SCR-VAPP-documents | Capture/upload credential documents with camera; expiry | vendor_admin | doc type, photo/PDF, expiry | as VEN | EL-C | vnd.credential.upload@own | AU-E | N-push expiry | A-cam | ML-N |
| SCR-VAPP-category-status | Category/property approval status and reasons | vendor_admin | as VEN verification | as VEN | EL-L | vnd.status.read@own | AU-0 | N-push | A-live | ML-N |
| SCR-VAPP-catalog | Browse own items/variants | vendor users | as VEN | as VEN | EL-O | vnd.catalog.read@own | AU-0 | — | A-std | ML-N |
| SCR-VAPP-item-editor | Add/edit item & variants (fresh/frozen/pulp/powder, cuts, pack/unit, printing/custom packing, per-job) with photos | vendor_user | as SCR-VEN-catalog-item-editor | draft, submitted | EL-O, EL-C | vnd.catalog.write@own | AU-W | — | A-form, A-cam | ML-N |
| SCR-VAPP-daily-stock | Quick daily stock update with as-of time and per-unit rates (KG/gram/packet) | vendor_user | variant, available qty, unit, price per unit, as-of | fresh, stale, queued_draft | EL-O (submit requires connection) | vnd.stock.write@own | AU-W | N-push reminder | A-form | ML-N |
| SCR-VAPP-rates | Per-unit and tier rates, validity, tax/freight | vendor_admin | as VEN price list | as VEN | EL-O | vnd.price.write@own | AU-W | — | A-form | ML-N |
| SCR-VAPP-delivery-coverage | Delivery zones, slots, capacity, partial-fill rule | vendor_admin | zones, slots, capacity, minimum order | draft, published | EL-O | vnd.profile.write@own | AU-W | — | A-map | ML-N |
| SCR-VAPP-rfq-list | RFQ invitations with deadlines | vendor users | as VEN | as VEN | EL-O | vnd.rfq.read@own | AU-0 | N-push | A-time | ML-N |
| SCR-VAPP-bid | Prepare/submit bid with sample photos from camera | vendor_admin | as VEN bid | draft (offline ok), submitted (online) | EL-M | vnd.bid.submit@own | AU-W + AU-E | N-push receipt | A-form, A-cam | ML-N |
| SCR-VAPP-po-accept | Review and acknowledge PO / request change | vendor_admin | as VEN PO detail | as VEN | EL-M | vnd.po.acknowledge@own | AU-W | N-push | A-form | ML-N |
| SCR-VAPP-dispatch-asn | Create ASN at dispatch: lots, expiry, temperature photo, vehicle; show QR label | vendor_user | as VEN ASN | draft, dispatched | EL-O (dispatch requires online) | vnd.asn.create@own | AU-E | N-in receiver | A-form, A-cam | ML-N |
| SCR-VAPP-invoice | Submit invoice from GRN with photo/PDF | vendor_admin | as VEN invoice | as VEN | EL-M | vnd.invoice.submit@own | AU-E | N-push status | A-form | ML-N |
| SCR-VAPP-payments | Payment status and remittances | vendor_admin | as VEN | as VEN | EL-L | vnd.payment.read@own | AU-0 | N-push paid | A-std | ML-N |
| SCR-VAPP-performance | Own scorecard | vendor_admin | as VEN | — | EL-D | vnd.performance.read@own | AU-0 | — | A-chart | ML-N |
| SCR-VAPP-assigned-jobs | Contractor job list, navigation to site, access window | vendor_user | as VEN | as VEN | EL-O | vnd.job.read@own | AU-0 | N-push | A-std | ML-N |
| SCR-VAPP-job-evidence | Check-in on site, photos, labor, parts, completion | vendor_user | as VEN | in_progress (offline ok), submitted (online) | EL-O, EL-C | vnd.job.evidence@own | AU-E | N-in | A-cam | ML-N |
| SCR-VAPP-callout-accept | Emergency chef callout: one-tap accept/decline with deadline countdown | emergency_chef | offer details, deadline, rate, location | offered, accepted, filled_by_other, expired | EL-M | vnd.callout.respond@own | AU-W | N-push/N-sms/N-wa/N-voice | A-time, A-live, A-target | ML-N |
| SCR-VAPP-notifications | Notification inbox and preferences | vendor users | items, channels | — | EL-L | own | AU-0 | all | A-live | ML-N |
| SCR-VAPP-offline-drafts | Drafts saved offline and what still needs server acknowledgement | vendor users | draft type, last edit, blocked reason | draft, ready_to_submit, submitted, rejected | EL-O | own | AU-W | — | A-live | ML-N |

### 4.9 CON — concierge, travel desk, transport and fleet

```
Concierge
├─ Home ── request inbox ── new request (guest/corporate) ── traveler consent
├─ Inquiry: flight · cruise · taxi/transfer ── offer compare ── guest approval ── order status ── itinerary
├─ Manual RFQ evidence · provider eligibility (market gate)
├─ Disruption/refund · after-hours handoff
└─ Fleet: dispatch board ── trip manifest ── driver trip (mobile) ── vehicle register
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-CON-home | Concierge home (§3) | concierge | new requests, approvals pending, expiring offers, unconfirmed orders, disruptions, pickups | — | EL-R | trv.home.read | AU-0 | N-in, N-esc | A-live | ML-C |
| SCR-CON-request-inbox | All travel/transport requests by status/owner | concierge, front desk, sales | request, guest/corporate, kind, dates, owner, status, SLA | per SM travel request | EL-L | trv.request.read | AU-0 | N-in | A-grid | ML-C |
| SCR-CON-request-new | Create request for guest/corporate itinerary with purpose, payer, cost center | concierge, front_desk_agent, sales_manager | requester, stay/corporate link, kind, dates, pax, special assistance, payer, visibility | draft, submitted | EL-F | trv.request.create | AU-W | — | A-form | ML-C |
| SCR-CON-traveler-consent | Capture guest permission and purpose-limited traveler data (SF45.1.2) | concierge, guest (via link) | consent text version, traveler details (encrypted), channel, evidence | pending, granted, withdrawn | EL-F | trv.traveler.write; field:passport | AU-R + AU-E | N-sms/N-wa consent link | A-form | ML-C |
| SCR-CON-flight-inquiry | Flight route/date/pax/class/luggage/assistance search via eligible provider or manual RFQ | concierge | origin/destination, dates, pax types, cabin, bags, assistance, provider capability flags | draft, searching, results, no_results, provider_unavailable | EL-L (capability unavailable → manual RFQ) | trv.flight.inquire | AU-W | — | A-form | ML-C |
| SCR-CON-cruise-inquiry | Cruise itinerary/date/cabin/occupants, transfers | concierge | itinerary, sailing date, cabin type, occupants, transfers | as flight | EL-L | trv.cruise.inquire | AU-W | — | A-form | ML-C |
| SCR-CON-taxi-transfer | Taxi/airport transfer request with pickup, flight monitoring, vehicle/accessibility | concierge, front_desk_agent | pickup/drop-off, time, pax, luggage, vehicle type, accessible, flight no, provider or hotel fleet | draft, requested, offered, booked, en_route, completed, no_show, cancelled | EL-M | trv.taxi.request | AU-W | N-sms/N-wa guest updates | A-form | ML-C |
| SCR-CON-offer-compare | Compare offers: price, taxes, fees, commission, FX, validity, cancellation, merchant of record | concierge | provider, total, breakdown, expiry countdown, terms, honesty label | valid, expiring, expired | EL-L | trv.offer.read | AU-0 | N-in expiring | A-grid, A-time | ML-C |
| SCR-CON-guest-approval | Send selected offer to guest for explicit approval with MoR disclosure (SF45.2.5) | concierge | offer summary, final price, terms, approval channel, evidence | sent, approved, rejected, expired | EL-M | trv.approval.send | AU-E | N-sms/N-wa/N-email | A-form (3.3.4), A-time | ML-C |
| SCR-CON-order-status | Order lifecycle; "booked" only with provider reference (SF45.2.6) | concierge | order, provider ref, ticket/voucher, status log, duplicate callback flags | submitted, pending, booked, ticketed, changed, cancelled, refunded, failed, unknown | EL-M | trv.order.read / .submit | AU-L (status log) | N-in change, guest N-email | A-live | ML-C |
| SCR-CON-manual-rfq-evidence | Record manual quotes/confirmations from email/portal with evidence upload (SF45.2.3) | concierge | provider, quote/confirmation text, file, reference, received time | recorded, verified | EL-C | trv.manual.record | AU-E | — | A-form | ML-C |
| SCR-CON-itinerary | Guest/corporate itinerary combining stay, event, transport, flights (confirmed vs proposed) | concierge, guest (shared), corporate | segments, status per segment, references, contacts | proposed, confirmed, changed | EL-L | trv.itinerary.read | AU-0 | N-email share | A-std | ML-P |
| SCR-CON-disruption-refund | Handle cancellations, delays, no-shows, refunds and supplier receivable (SF45.2.8) | concierge, finance_clerk | disruption, options, refund amount, supplier claim, guest communication | open, rebooked, refunded, closed | EL-M | trv.disruption.handle | AU-W | N-esc | A-form | ML-C |
| SCR-CON-handoff | After-hours escalation and staff handoff with full context | concierge, duty_manager | open items, contacts, deadlines | open, acknowledged | EL-L | trv.handoff | AU-W | N-push on-call | A-live | ML-C |
| SCR-CON-provider-eligibility | Which providers/capabilities are permitted for this property/market (F45.3) | concierge, compliance_officer | provider, category, licence status, capabilities (quote/hold/book/issue), gate state | enabled, manual_only, blocked | EL-L | trv.eligibility.read | AU-0 | — | A-grid | ML-C |
| SCR-CON-fleet-dispatch | Hotel shuttle/driver dispatch board (F59.1) | concierge, transport coordinator | trips, vehicles, drivers, times, flight ETAs, status | planned, assigned, en_route, completed, missed | EL-R | trv.fleet.dispatch | AU-W | N-push driver | A-grid, A-drag | ML-T |
| SCR-CON-trip-manifest | Manifest with passengers (minimum data), luggage, accessibility | concierge, driver | passengers, pickup points, bags, assistance | draft, final | EL-L | trv.manifest.read | AU-R | — | A-grid | ML-C |
| SCR-CON-driver-trip | Driver mobile: trip steps, passenger handover, incident | driver (employee) | trip, route, passenger check-off, status buttons, incident report | assigned, en_route, arrived, onboard, completed | EL-O | trv.trip@own | AU-W | N-push | A-target, A-live | ML-N |
| SCR-CON-vehicle-register | Vehicles/drivers with permits, insurance, maintenance validity (F59.2) | concierge, chief_engineer | vehicle, plate, permit/insurance expiry, service status, driver licence expiry | valid, expiring, blocked | EL-L | trv.fleet.write | AU-W | N-in expiry | A-grid | ML-C |

### 4.10 FNB — bar, kitchen, club, catering, restaurant, events/BEO, amenities

```
F&B / events
├─ Homes: F&B manager · chef · bartender/server
├─ POS (terminal): tables ── order ── tab ── payment ── void/comp approval ── shift close ── KDS
├─ Menu: editor, happy hour pricing, recipe/BOM, allergen matrix
├─ Stock (outlet): outlet stock, count, theoretical vs actual, waste log, ingredient/vendor catalog search
├─ Kitchen continuity: coverage plan, chef roster, emergency roster, callout (W8) ── handover
├─ Production: production plan, HACCP checks, lot trace
├─ Events: function diary ── event detail ── BEO editor ── BEO kitchen view ── change order ── actuals & settlement
├─ Catering: order ── dispatch/returns
├─ Club: reservations ── door entry ── memberships
├─ Restaurant: reservations, in-room dining
└─ Amenities (if enabled): bookings, configuration
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-FNB-home | F&B manager home (§3) | fnb_manager | approvals, shift closes, variance, staffing, complaints | — | EL-D | com.fnb.read | AU-0 | N-in | A-chart | ML-C |
| SCR-FNB-chef-home | Chef home (§3) | executive_chef, shift_chef | coverage next services, BEOs with allergens, callouts, lot alerts/recalls, waste approvals, HACCP due | — | EL-R | com.kitchen.read | AU-0 | N-push, N-esc | A-live | ML-T |
| SCR-FNB-bartender-home | Bartender/server home (§3) | bartender, server | open tabs, ready orders, low bar stock, shift close | — | EL-R | com.pos.use | AU-0 | N-rt | A-live | ML-K |
| SCR-FNB-pos-tables | Table map/list with status and covers | server | tables, status, covers, time seated, server | free, seated, ordered, bill_requested, paid | EL-R | com.pos.use | AU-0 | N-rt | A-map (list alternative), A-drag | ML-K |
| SCR-FNB-pos-order | Enter items/modifiers, courses, allergens, send to KDS; offline store-and-forward | bartender, server | items, modifiers, seat, course, allergen flags, happy-hour price, client_check_id | open, sent, closed, voided, queued | EL-O (POS offline rules) | com.pos.order | AU-W | N-rt KDS | A-target, A-live | ML-K |
| SCR-FNB-pos-tab | Bar tab: open/transfer/close, pre-auth, room charge eligibility check | bartender | tab, guest/room, limit, items | open, transferred, closed | EL-O | com.pos.tab | AU-W | — | A-target | ML-K |
| SCR-FNB-pos-payment | Split tender, tips/service charge, room/corporate/event charge (in-house validation), card terminal | bartender, server, cashier | tenders, split method, tip, service charge, charge target, idempotency ref | pending, paid, partially_paid, failed, unknown | EL-M (room charge requires online validation; offline → pending folio charge) | com.pos.settle; fin.folio.post (system) | AU-L | N-in failures | A-form, A-target | ML-K |
| SCR-FNB-pos-void-comp-approval | Manager approval for void/comp/discount beyond limit | fnb_manager, duty_manager | item, reason, amount, requester, limit | requested, approved, rejected | EL-M | com.pos.void_approve (+limit) `+mc` | AU-P | N-push manager | A-form | ML-P |
| SCR-FNB-pos-shift-close | Close POS shift: tenders, tips, open checks, variance | bartender, cashier, fnb_manager | expected vs counted, tips pool, open checks | open, closing, closed, variance_review | EL-M | com.pos.shift_close | AU-L | N-in variance | A-form | ML-K |
| SCR-FNB-kds | Kitchen display: tickets by station, timers, allergen banners, bump | shift_chef, cooks | ticket, items, modifiers, allergens (prominent), course, timer, table/room | new, in_progress, ready, served, recalled | EL-R (site gateway fallback) | com.kds.use | AU-W (bump log) | N-rt | A-live (assertive for allergen), high contrast mode | ML-K |
| SCR-FNB-menu-editor | Menus, items, modifiers, prices, outlets, EN/AR names, allergens, availability | fnb_manager, executive_chef | item, recipe link, price, tax category (JUR), outlets, allergens, images | draft, published, archived | EL-F | com.menu.write | AU-W | — | A-form | ML-D |
| SCR-FNB-happy-hour-pricing | Time-based pricing rules | fnb_manager | rule, items, time windows, days, price/discount | draft, active | EL-F | com.pricing.write `+mc` | AU-W | — | A-form | ML-D |
| SCR-FNB-recipe-bom | Recipe BOM, yield, portion, cost, allergen roll-up | executive_chef | ingredients (item, qty, UOM), yield %, portion, cost, allergens derived | draft, approved | EL-F | com.recipe.write | AU-W | — | A-grid | ML-D |
| SCR-FNB-allergen-matrix | Menu × allergen matrix per outlet/language for guests and staff | executive_chef, server | items, 14+ allergen columns (configurable per JUR), diet tags, last verified | current, needs_review | EL-L | com.allergen.read | AU-W on change | N-in on recipe change | A-grid (not colour only) | ML-C |
| SCR-FNB-outlet-stock | Outlet store balances and par, reorder suggestions | fnb_manager, bartender | item, on hand, par, reserved, expiring | — | EL-L | stk.balance.read@dept | AU-0 | N-in below par | A-grid | ML-C |
| SCR-FNB-stock-count | Outlet blind count (bar/kitchen) with offline capture | bartender, shift_chef | count session, items, counted qty, UOM, lot | open, submitted, approved | EL-O | stk.count.enter@dept | AU-L | N-in variance | A-form | ML-N |
| SCR-FNB-theoretical-vs-actual | POS depletion (theoretical) vs actual consumption; variance by item | fnb_manager, financial_controller | item, theoretical, actual, variance qty/value, waste recorded | — | EL-D | stk.variance.read@dept | AU-0 | — | A-grid, A-chart | ML-D |
| SCR-FNB-waste-log | Record waste/spoilage/breakage/staff meal with reason, photo, lot; approval | shift_chef, bartender, fnb_manager | item, lot, qty, reason, photo, cost | recorded, approved, rejected | EL-O | stk.waste.record@dept; approve `+mc` over limit | AU-L | N-in approval | A-form, A-cam | ML-N |
| SCR-FNB-ingredient-catalog-search | Search approved vendor catalog by ingredient, form, unit, freshness, location | executive_chef, procurement_officer | ingredient, form, UOM price normalised, available today, as-of/stale badge, vendor rating | — | EL-L | vnd.catalog.search@dept | AU-0 | — | A-grid | ML-C |
| SCR-FNB-chef-coverage-plan | Primary/backup chef per meal service/outlet/event; gaps before cutoff (F47.1) | executive_chef, fnb_manager | service, window, primary, backup, status, skills/allergen training match | covered, at_risk, uncovered, filled_emergency, contingency | EL-R | wrk.coverage.write@dept | AU-W | N-esc on uncovered | A-grid, A-live | ML-T |
| SCR-FNB-chef-roster | Chef shift schedule with skills and food-safety credentials | executive_chef | chef, shifts, cuisine skills, cert expiry, leave | draft, published | EL-L | wrk.roster.write@dept | AU-W | N-push | A-grid, A-drag | ML-T |
| SCR-FNB-emergency-roster | Approved emergency chefs with availability, response time, contract/rate, eligibility | executive_chef, procurement_officer | chef/vendor, distance, response minutes, cert validity, contract, rate, priority order | eligible, ineligible (reason) | EL-L | wrk.emergency_roster.write `+mc` | AU-W | N-in expiry | A-grid | ML-C |
| SCR-FNB-chef-callout | Run callout: status of primary/backup, ordered contact sequence, deadlines, acceptance, approval (critical flow W8) | executive_chef, fnb_manager, duty_manager | slot, attempts (channel, time, reply), deadline countdown, accepted candidate, eligibility check, approval for paid external, contingency plan | open, attempting, accepted_pending_approval, confirmed, escalated, contingency, failed | EL-M (atomic accept), EL-R | wrk.callout.run; approve paid `+mc` | AU-W | N-push/N-sms/N-wa/N-voice to candidates; N-esc manager/GM | A-live, A-time | ML-P |
| SCR-FNB-callout-handover | Share BEO/allergen/production plan to assigned emergency chef only for the assignment window | executive_chef | assignment, documents shared, access expiry, acknowledgment | shared, acknowledged, revoked | EL-F | wrk.handover.share | AU-R | N-push chef | A-std | ML-P |
| SCR-FNB-production-plan | Kitchen production by service/event from covers/BEO; ingredient reservation | executive_chef, shift_chef | dishes, portions, prep times, ingredients required vs reserved, lots | draft, released, in_progress, completed | EL-L | com.production.write | AU-W | N-in shortages | A-grid | ML-T |
| SCR-FNB-haccp-checks | Temperature/holding/cold-chain checklists per verified rule pack (F57.2) | shift_chef | checklist, readings, sensor/manual, corrective action | due, completed, failed, overdue | EL-O | com.haccp.record | AU-E | N-esc overdue/failed | A-form | ML-N |
| SCR-FNB-lot-trace | Which lots went into which dish/event/guest batch; recall hold | executive_chef, compliance_officer | lot, supplier, dishes, events, dates, affected guests (restricted), hold status | none, hold, recalled | EL-L | stk.trace.read | AU-R | N-esc recall | A-grid | ML-D |
| SCR-FNB-function-diary | Function space calendar with holds, definite events, setup/teardown | catering_manager, sales_manager | spaces/partitions, time blocks incl. buffers, status, organizer | tentative, definite, held (expiry) | EL-R (exclusion prevents double booking) | com.event.read | AU-0 | N-rt | A-grid, A-drag, A-map | ML-D |
| SCR-FNB-event-detail | Event master: dates, attendees, spaces, room block, F&B, AV, parking, club, billing | catering_manager | event, organizer, composite hold, BEO version, master folio, deposits, status | inquiry, tentative, definite, in_progress, closed, cancelled | EL-F | com.event.write | AU-W | N-in departments | A-form | ML-D |
| SCR-FNB-beo-editor | Versioned BEO lines by department; send for signature | catering_manager | lines (menu, qty, allergens, AV, setup, labor, parking), prices, notes, version | draft, sent, signed, superseded | EL-F | com.beo.write | AU-W + AU-E (signature) | N-email organizer; N-in departments | A-grid, A-form | ML-D |
| SCR-FNB-beo-kitchen-view | Kitchen/bar view of current signed BEO with change highlights | executive_chef, bartender | current version, changes since last view, allergens, guaranteed covers, timings | current, superseded | EL-R | com.beo.read@dept | AU-0 | N-push on revision | A-live | ML-T |
| SCR-FNB-event-change-order | Post-contract change with price delta and approval | catering_manager, corporate approver (via CORP) | from/to version, delta, capacity recheck, approval | requested, approved, rejected | EL-M | com.change_order.write `+mc` above limit | AU-P | N-email organizer | A-form | ML-D |
| SCR-FNB-event-actuals-settlement | Actual vs contracted (covers, consumption, extras) → invoice, master/individual billing | catering_manager, ar_clerk | contracted, actual, variances, charges to master folio, AR invoice | open, reconciled, invoiced | EL-M | com.event.settle | AU-L | N-email invoice | A-grid | ML-D |
| SCR-FNB-catering-order | Off-site/in-hotel catering order: menu tiers, guarantees, cutoffs, allergens, staff/equipment/vehicle | catering_manager | order, menus, covers, guarantee cutoff, allergens, resources | draft, confirmed, guaranteed, dispatched, completed, cancelled | EL-F | com.catering.write | AU-W | N-in kitchen | A-form | ML-D |
| SCR-FNB-catering-dispatch | Dispatch and returns (equipment, food, containers) | catering_manager, driver | load list, vehicle, temperatures, returned items | loading, dispatched, returned, discrepancy | EL-O | com.catering.dispatch | AU-E | N-in discrepancy | A-form | ML-N |
| SCR-FNB-club-reservations | Club/lounge/pool reservations with capacity and minimum spend | club_host | date, party, area, minimum spend, capacity slot | requested, confirmed, seated, no_show, cancelled | EL-L | com.club.reserve | AU-W | N-sms guest | A-grid | ML-C |
| SCR-FNB-club-entry | Door scan of pass/membership/room key; capacity; re-entry; age/licensing gates where configured | club_host, security_officer | pass, holder, entitlement, capacity now, re-entry flag | admitted, denied (reason), exited | EL-R | com.club.entry | AU-W | N-rt capacity | A-live, A-target | ML-K |
| SCR-FNB-club-memberships | Plans, members, renewals, benefits, billing | club_host, fnb_manager | plan, member, validity, benefits used, billing | active, expiring, lapsed, cancelled | EL-L | com.membership.write | AU-W | N-email renewals | A-grid | ML-C |
| SCR-FNB-restaurant-reservations | Table reservations, waitlist, turn times | server, host | party, time, table, preferences, allergy note | booked, seated, no_show, cancelled | EL-L | com.restaurant.reserve | AU-W | N-sms guest | A-grid, A-drag | ML-C |
| SCR-FNB-in-room-dining | In-room dining orders, promise time, delivery confirmation | server, room service | order, room (in-house check), items, allergens, promised/delivered time, charge | received, preparing, out_for_delivery, delivered, cancelled | EL-R | com.ird.manage | AU-W | N-rt, guest N-push | A-live | ML-T |
| SCR-FNB-amenity-bookings | Spa/pool/gym/beach/golf/retail slots with practitioner/equipment capacity (M58, if enabled) | amenity staff | slot, resource, practitioner, guest, consent form status, price | booked, checked_in, completed, no_show, cancelled | EL-L | com.amenity.book | AU-W | N-sms guest | A-grid | ML-C |
| SCR-FNB-amenity-config | Enable amenity business, resources, practitioners, safety/consent rules | property_admin, amenity manager | amenity type, resources, qualifications, commissions, consent templates | disabled, enabled | EL-F | com.amenity.configure | AU-W | — | A-form | ML-D |

### 4.11 PRC — procurement, vendor directory, receiving and stores

```
Procurement & stores
├─ Homes: procurement officer/approver · storekeeper · receiver
├─ Vendors (hotel side): directory (department-scoped) ── vendor detail ── review/approval (maker-checker) ── credential expiry ── scorecard ── contracts/blanket
├─ Requisitions: new ── list ── approval
├─ RFQ: builder ── monitor (quote minimum) ── waiver ── sample gallery (90-day) ── comparison (W6) ── award
├─ PO: detail ── change ── AI follow-up queue ── delivery schedule
├─ Receiving: receiving (W7) ── high-risk verify ── quarantine ── return to vendor
├─ Stores: issue ── return (intact/waste) ── transfers ── balances ── blind count ── recall lookup ── item master ── replenishment ── cylinder exchange
└─ Reports
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-PRC-home | Procurement home (§3) | procurement_officer, procurement_approver | requisitions, RFQs below minimum, awards ready, unacknowledged/late POs, vendor approvals | — | EL-D | stk.procurement.read | AU-0 | N-in, N-esc | A-std | ML-C |
| SCR-PRC-storekeeper-home | Storekeeper home (§3) | storekeeper | issue requests, returns to inspect, quarantine, counts due, expiring lots | — | EL-R | stk.store.read | AU-0 | N-in | A-live | ML-T |
| SCR-PRC-receiver-home | Receiver home (§3) | receiver | expected deliveries, arrivals at gate, high-risk verifications, discrepancies | — | EL-R | stk.receiving.read | AU-0 | N-push arrival | A-live | ML-T |
| SCR-PRC-vendor-directory | Department-scoped search of eligible approved vendors by category, location, stock freshness, rating (F46.2) | dept requesters, procurement_officer, concierge, chief_engineer | vendor, categories, coverage, credentials valid, available today (as-of), rating (sample size), contract | eligible, ineligible (hidden from ordering, shown in audit mode) | EL-L | vnd.directory.search@dept | AU-0 | — | A-grid | ML-C |
| SCR-PRC-vendor-detail | Vendor profile, credentials, categories/properties approved, contracts, performance, incidents | procurement_officer | profile, docs with expiry, approvals, scorecard, complaints, payment history (FIN summary) | per vendor status | EL-F | vnd.vendor.read | AU-0 | — | A-form | ML-C |
| SCR-PRC-vendor-review | Verify documents/sanctions/duplicates/bank and approve per category/property (maker-checker, conflict declaration) | procurement_officer (maker), procurement_approver (checker) | submitted data, document verification results, duplicate matches, sanctions result, conflict declarations, decision reason | submitted, under_review, approved, rejected, suspended | EL-F | vnd.vendor.approve `+mc` | AU-P | N-email/N-push vendor | A-form | ML-D |
| SCR-PRC-credential-expiry | Expiring/expired vendor credentials; auto-suspend rules | procurement_officer | vendor, doc, expiry, auto-action | expiring, expired, renewed | EL-L | vnd.credential.read | AU-W | N-in, vendor N-email | A-grid | ML-C |
| SCR-PRC-vendor-scorecard | Vendor OTIF, defects, response, food-safety incidents with confidence and fair-review (F46.2.6) | procurement_officer | metrics, sample size, disputes, trend | — | EL-D | vnd.performance.read | AU-0 | — | A-chart | ML-D |
| SCR-PRC-contracts-blanket | Contracts, blanket orders, rate cards, SLAs, validity | procurement_officer | contract, vendor, validity, price list, SLA, consumption vs cap | draft, active, expiring, expired | EL-L | stk.contract.write `+mc` | AU-W | N-in renewal | A-grid | ML-D |
| SCR-PRC-requisition-new | Department requisition with spec, UOM, quantity, need-by, sample photo request, cost center (F49.1) | dept requesters (chef, engineer, housekeeping) | items/spec, form/grade, UOM, qty, delivery date/location, allowed alternatives, sample request, BEO/WO link, budget check | draft, submitted | EL-F, EL-C | stk.requisition.create@dept | AU-W | N-in approver | A-form | ML-P |
| SCR-PRC-requisition-list | Requisitions by status/department | requesters, procurement_officer | req, dept, value, need-by, status | draft, submitted, approved, tendering, ordered, closed, rejected | EL-L | stk.requisition.read@dept | AU-0 | — | A-grid | ML-C |
| SCR-PRC-requisition-approval | Budget/threshold approval routing | dept head, procurement_approver | budget remaining, threshold rule, emergency flag | pending, approved, rejected, returned | EL-F | stk.requisition.approve (+limit) `+mc` | AU-P | N-in requester | A-form | ML-C |
| SCR-PRC-rfq-builder | Build RFQ from requisition: invitees (eligible only), deadline, sealed/open, weights version, minimum quotes | procurement_officer | lines, invitees, min quotes policy, weights (locked at opening), sample requirements, Q&A | draft, open | EL-F | stk.rfq.create | AU-W | N-push/N-email vendors | A-form | ML-D |
| SCR-PRC-rfq-monitor | Track responses vs minimum, reminders, no-bids, late bids | procurement_officer | invitees, responses, time left, minimum met flag | open, below_minimum, closed | EL-R | stk.rfq.read | AU-0 | N-in below minimum at deadline | A-grid, A-time | ML-C |
| SCR-PRC-rfq-waiver | Single-source/urgent/insufficient-quotes exception with higher approver (SF49.1.6) | procurement_officer, procurement_approver | reason, evidence, value, approver | requested, approved, rejected | EL-F | stk.rfq.waive `+mc` `+stepup` | AU-P | N-in approver | A-form | ML-C |
| SCR-PRC-sample-gallery | Sample photos per RFQ/bid side-by-side with retention countdown, purge proof, holds (SF49.1.4) | procurement_officer, executive_chef | image, bid, uploaded at, retention start event, retention_until, hold, purged_at, proof hash | retained, on_hold, purged | EL-L, EL-C | stk.sample.read; hold `+mc` | AU-E | N-in 7 days before purge | A-media (alt text), A-grid | ML-C |
| SCR-PRC-rfq-comparison | Normalised landed-cost and weighted comparison; disqualifications; AI evidence summary (non-deciding) (critical flow W6) | procurement_officer, evaluators | per bid: normalised unit price, landed cost, quality/freshness, availability, lead time, OTIF, defects, food safety, history (n, confidence), weighted score, disqualify reason; AI summary labelled | evaluating, recommended | EL-L | stk.rfq.evaluate (conflict check) | AU-W | — | A-grid (sortable, row summaries), A-chart | ML-D |
| SCR-PRC-award | Sign award: recommended vs chosen, override reason, SoD, budget encumbrance, split award, notifications | procurement_approver | recommendation, selection, override justification, encumbrance, runner-up notice | pending, awarded, rejected | EL-M | stk.award.approve `+mc` `+stepup` (≠ requester) | AU-P | N-email/N-push awarded and not-awarded vendors | A-form (3.3.4) | ML-C |
| SCR-PRC-po-detail | Versioned PO with milestones, vendor acknowledgment, receipts, invoices | procurement_officer | PO version, lines, delivery window, quality/temperature spec, sample ref, terms, milestones, ack status | issued, acknowledged, changed, partially_received, received, closed, cancelled | EL-F | stk.po.read / .issue | AU-W | N-push vendor | A-grid | ML-C |
| SCR-PRC-po-change | Change/cancel PO with new version and vendor acceptance | procurement_officer | changes, reason, encumbrance delta | requested, vendor_accepted, rejected | EL-M | stk.po.change `+mc` over limit | AU-P | N-push vendor | A-form | ML-C |
| SCR-PRC-ai-followup-queue | AI-drafted vendor follow-ups and parsed replies needing human confirmation (F50.1) | procurement_officer | PO, milestone, due, draft message, vendor reply, parsed ETA with confidence, proposed status | draft_ready, sent, reply_parsed, needs_confirmation, confirmed, escalated | EL-L | stk.followup.approve | AU-W (AI label, model version) | N-in late ETA, N-esc | A-grid | ML-C |
| SCR-PRC-delivery-schedule | Dock calendar of expected deliveries (from PO/ASN ETA) | receiver, storekeeper | vendor, PO/ASN, ETA, slot, cold-chain flag | expected, arrived, received, late | EL-R | stk.receiving.read | AU-0 | N-push arrival | A-grid | ML-T |
| SCR-PRC-receiving | Gate/dock receiving: scan PO/ASN QR, barcode/scale/temperature capture, OCR packing slip, expected vs scanned, draft GRN, attest (critical flow W7) | receiver | PO/ASN, lines expected, scanned SKU/lot/expiry, weight, temperature, photos, OCR result, tolerance result, risk class | draft, straight_through_ready, verification_required, posted, partially_rejected | EL-O (draft), EL-M (post), EL-C | stk.receipt.draft / .post | AU-E + AU-L | N-in quarantine, AP | A-form, A-cam | ML-T |
| SCR-PRC-receiving-verify | Physical verification for food/high-value/discrepancy lines | receiver (accountable), storekeeper | line, checks (temperature, condition, expiry, packaging), photos, decision | pending, accepted, rejected, quarantined | EL-O | stk.receipt.verify | AU-E | N-in | A-form, A-cam | ML-N |
| SCR-PRC-quarantine | Quarantined stock with disposition: return, credit, destroy, concession | storekeeper, procurement_officer, executive_chef | lot, qty, reason, photos, vendor claim, disposition | quarantined, return_pending, credited, destroyed, released_with_concession (not for food safety failures) | EL-L | stk.quarantine.decide `+mc` | AU-L | N-in vendor claim | A-grid | ML-C |
| SCR-PRC-return-to-vendor | Return/rejection with credit memo request | storekeeper | lines, reason, vehicle, signature, credit note ref | draft, shipped, credited | EL-M | stk.rtv.create | AU-L + AU-E | N-email vendor | A-form, A-sig | ML-C |
| SCR-PRC-store-issue | Issue to kitchen/bar/catering/housekeeping/maintenance/job/BEO via barcode, FEFO pick | storekeeper | request, item, lot (FEFO suggested), qty, destination, BEO/WO link | requested, picked, issued, partially_issued | EL-O (request), EL-M (issue) | stk.issue.post | AU-L | N-in requester | A-form | ML-N |
| SCR-PRC-store-return | Return intact stock (inspector approval, original lot/expiry) or record waste/quarantine (never back to available) | storekeeper, executive_chef | item, lot, qty, condition, reason, photo, inspector | intact_pending_inspection, returned_available, waste, quarantine | EL-O | stk.return.post; inspector `+mc` | AU-L | N-in | A-form, A-cam | ML-N |
| SCR-PRC-transfers | Store-to-store transfers with in-transit state | storekeeper | from/to store, items, lots, qty | draft, in_transit, received | EL-L | stk.transfer.post | AU-L | N-in receiving store | A-grid | ML-C |
| SCR-PRC-stock-balances | Balances by store/bin/lot/state with value and age | storekeeper, financial_controller | item, lot, bin, available/reserved/quarantine/waste, expiry, value, age | — | EL-L | stk.balance.read | AU-0 | N-in expiring | A-grid | ML-C |
| SCR-PRC-blind-count | Blind count session with variance approval | storekeeper, finance_clerk | session, bins, counted, variance (after submit), approval | open, submitted, approved, posted | EL-O | stk.count.post `+mc` | AU-L | N-in variance | A-form | ML-N |
| SCR-PRC-recall-lookup | Trace by supplier/lot to stock, issues, dishes/events and hold all remaining | storekeeper, executive_chef, compliance_officer | lot, locations, issued to, consumed in events/dishes, hold actions | none, hold, recalled | EL-L | stk.recall.manage | AU-P | N-esc recall | A-grid | ML-D |
| SCR-PRC-item-master | Hotel master items, UOM conversions, lot tracking, food-safety risk, vendor crosswalk approval | procurement_officer, storekeeper | SKU, name EN/AR, base UOM, conversions, category, risk class, allergens, crosswalk | draft, active, inactive | EL-F | stk.item.write | AU-W | — | A-form | ML-D |
| SCR-PRC-replenishment | Par/reorder suggestions → requisitions | storekeeper, procurement_officer | item, on hand, par, reorder point, suggested qty, lead time | suggested, requisitioned, dismissed | EL-L | stk.replenish | AU-W | N-in | A-grid | ML-C |
| SCR-PRC-cylinder-exchange | Gas cylinder exchange: full in, empty out, deposit, reorder point (F25.1) | storekeeper | cylinders delivered (serial/size), empties returned, deposits, PO link, safety check | scheduled, exchanged, discrepancy | EL-O, EL-M (post) | stk.cylinder.exchange | AU-L | N-in reorder point | A-form | ML-N |
| SCR-PRC-procurement-reports | Competition/exception/award-weight audit, OTIF, savings, stock age, waste cost (F50.4) | procurement_approver, auditor | report selector, results, drill-through | — | EL-D | stk.report.read | AU-0 | N-email scheduled | A-grid, A-chart | ML-D |

### 4.12 GST — guest website, guest app and Partner Hub

```
Guest website (SSR, low-bandwidth)                  Guest app (native) / signed-in web
├─ Home / content pages / cookie choices            ├─ My trips ── manage ── cancel
├─ Room search ── results ── room detail            ├─ Pre-check-in ── ID capture ── ID confirm ── registration sign ── OTP ── (QR hand-off)
│   ── extras ── guest details ── payment           ├─ Digital pass ── service requests ── messages ── AI chat
│   ── confirmation | quote expired (W1)            ├─ Parking vehicle ── in-room dining ── purchases ── pay balance ── receipts ── express checkout
├─ Facility search ── amenity booking               ├─ Points ── redeem
├─ Sign in                                          ├─ Privacy & consent ── data request
└─ Assisted contact (every step)                    └─ Lost item inquiry ── feedback survey
Partner Hub (referrers; gated per market): onboarding ── status ── statement
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-GST-home | Property home: accurate media, rooms, facilities, location, accessibility, search entry | guest | hero media (approved only), highlights, search box, policies links | — | EL-P | public | AU-0 | — | A-media, A-help | ML-P |
| SCR-GST-content-pages | Policies, accessibility statement, location/directions, FAQs (owner/version) | guest | page content EN/AR, last reviewed | published | EL-P | public | AU-0 | — | A-std | ML-P |
| SCR-GST-cookie-consent | Cookie/analytics choices; equal-prominence accept/reject (SF51.2.4) | guest | categories, choices, policy link | unset, set | EL-P | public | AU-W (consent record) | — | A-std, keyboard trap-free | ML-P |
| SCR-GST-room-search | Dates, party (adults/children ages), accessibility needs, promo/corporate code | guest, booker | check-in/out, rooms, occupancy, accessible features filter, code | — | EL-P | public | AU-0 | — | A-form (date picker with text input) | ML-P |
| SCR-GST-room-results | Sellable room types with **total price incl. taxes/fees**, policies, availability (live) | guest | room type, photos, features, nightly & total price, taxes/fees, cancellation, rate plans, remaining count (optional) | available, sold_out, restricted | EL-P (no-JS list works) | public | AU-0 | — | A-std | ML-P |
| SCR-GST-room-detail | Room details: media, accessible features, bed types, size (verified), policies | guest | gallery (approved), alt text, accessibility details, amenities | — | EL-P | public | AU-0 | — | A-media | ML-P |
| SCR-GST-extras | Optional upgrades/early-late/parking/dining/packages with live capacity | guest | offers, price, capacity, terms | — | EL-P | public | AU-0 | — | A-form | ML-P |
| SCR-GST-guest-details | Booker and occupants, contact, special requests, consent choices (marketing separate, unticked) | guest, booker | names (EN/AR), email, phone, country, arrival time, requests, accessibility notes (purpose-limited), referral code, consents | — | EL-P (entered data preserved) | public / guest | AU-W | — | A-form (autocomplete, 3.3.7) | ML-P |
| SCR-GST-payment | Review total and policies; PSP-hosted fields/redirect; hold timer; points/voucher tender | guest | summary, total, deposit now vs later, policy acceptance, PSP hosted component, idempotency | reviewing, paying, requires_action (3DS), succeeded, failed, unknown | EL-M (status unknown → checking; never double charge) | public / guest | AU-L (payment) | N-email receipt | A-form (3.3.4 review/confirm), A-time, A-auth (3DS accessible) | ML-P |
| SCR-GST-confirmation | Booking confirmed only after inventory + payment + policy checks; next steps | guest | confirmation no, dates, total paid/due, policies, pre-check-in CTA, add to calendar | confirmed | EL-P | guest (token link) | AU-0 | N-email/N-sms/N-wa (consented) | A-std | ML-P |
| SCR-GST-quote-expired | Hold/quote expired or price changed: show new price, re-hold option | guest | old vs new price, availability | expired, repriced | EL-P | public | AU-0 | — | A-live | ML-P |
| SCR-GST-facility-search | Search meeting rooms/day use/restaurant tables by date/time/party | guest | date, time, attendees/party, layout | — | EL-P | public | AU-0 | — | A-form | ML-P |
| SCR-GST-amenity-booking | Book spa/pool/golf/day pass slots (if enabled) with consent/safety forms | guest | slot, practitioner/resource, price, consent/contraindication form (where lawful) | available, booked | EL-M | guest | AU-W | N-email | A-form, A-time | ML-P |
| SCR-GST-sign-in | Passwordless sign-in (email/SMS OTP, passkey); link bookings | guest | email/phone, OTP, passkey | signed_out, otp_sent, verified | EL-F | public | AU-W | N-sms/N-email OTP | A-auth | ML-P |
| SCR-GST-my-trips | Guest home: upcoming/past stays and tasks (§3) | guest | stays, pre-check-in progress, requests, balance, points | — | EL-L | gst.trips@own | AU-0 | N-push reminders | A-std | ML-P |
| SCR-GST-manage-booking | Modify dates/room/extras with repricing; add occupants | guest, booker | current booking, change options, price difference, policy | viewing, change_quoted, changed | EL-M | gst.booking.amend@own | AU-W | N-email | A-form (3.3.4) | ML-P |
| SCR-GST-cancel-booking | Cancel with fee/refund preview | guest | policy, fee, refund, points reversal | confirmed → cancelled | EL-M | gst.booking.cancel@own | AU-W | N-email | A-form (3.3.4) | ML-P |
| SCR-GST-precheckin | Pre-arrival: arrival time, preferences, ID/registration/verification steps with non-biometric/manual options | guest | ETA, requests, required steps per JUR, progress | not_started, partial, complete | EL-P | gst.precheckin@own | AU-W | N-email/N-wa reminder | A-form | ML-P |
| SCR-GST-id-capture | Capture ID document with guidance; alternatives: upload, desk check, manual entry (SF41.1.2) | guest | document type (per JUR), images (encrypted, short-lived), quality checks | capturing, uploaded, scanning, extracted, failed | EL-C | gst.idv@own | AU-E | — | A-cam (manual/desk alternative), A-time | ML-P |
| SCR-GST-id-confirm | Field-by-field confirmation/correction of OCR values; mismatch flagged for staff review | guest | extracted fields with confidence, editable values, corrections flagged | pending_confirmation, confirmed, mismatch_review | EL-F | gst.idv@own | AU-E | — | A-form | ML-P |
| SCR-GST-registration-sign | Review registration card and sign (typed name + intent checkbox or drawn), receipt with hash | guest | card fields, policy text, signature method, timestamp, document hash | unsigned, signed | EL-M | gst.sign@own | AU-E | N-email signed copy | A-sig, A-form (3.3.4) | ML-P |
| SCR-GST-otp-verify | SMS/approved WhatsApp OTP bound to session/device with expiry and resend limits | guest | channel, masked number, code input, resend timer, attempts left | sent, verified, expired, locked, delivery_failed (→ alternative) | EL-F | gst.otp@own | AU-E | N-sms/N-wa | A-auth (one-time-code autofill), A-time | ML-P |
| SCR-GST-qr-handoff | QR to continue on phone; resumes same session and still requires OTP/verification | guest | QR (short expiry), instructions, fallback link by SMS | active, consumed, expired | EL-P | gst.session@own | AU-E | — | A-cam (text link alternative), A-time | ML-P |
| SCR-GST-digital-pass | Stay pass: room, dates, Wi-Fi info, parking plate, QR for club/facility entry, directions | guest | stay summary, pass QR, entitlements | upcoming, in_house, checked_out | EL-O (read-only cache) | gst.stay@own | AU-0 | N-push | A-std (QR has text code) | ML-N |
| SCR-GST-service-requests | Request items/services/maintenance/accessibility support; track status | guest | category, details, preferred time, status, ETA | submitted, acknowledged, in_progress, done, reopened | EL-L | gst.request@own | AU-W | N-push status | A-form, A-live | ML-P |
| SCR-GST-request-detail | Request thread and outcome; reopen | guest | messages, status history, rating | as above | EL-F | gst.request@own | AU-W | N-push | A-live | ML-P |
| SCR-GST-messages | Messaging with hotel (and AI assistant, clearly labelled) | guest | thread, channel, agent identity (human/AI), attachments | open, closed | EL-L | gst.message@own | AU-W | N-push | A-live | ML-P |
| SCR-GST-parking-vehicle | Register/change vehicle plate(s) for stay; see parking entitlement/charges | guest | plate, region, dates, entitlement, tariff | registered, active, expired | EL-F | gst.parking@own | AU-W | N-push entry/exit (optional) | A-form | ML-P |
| SCR-GST-in-room-dining | Order food/drinks to room with allergen info and promise time | guest | menu (EN/AR), allergens, diet filters, delivery time, charge to room | cart, placed, preparing, delivered | EL-M | gst.ird@own (in-house) | AU-W | N-push | A-form | ML-P |
| SCR-GST-purchases | Folio view: charges, payments, pending items | guest | entries (guest-visible windows only), balance | open, settled | EL-L | gst.folio.read@own | AU-0 | — | A-grid | ML-P |
| SCR-GST-pay-balance | Pay balance/deposit via PSP (pay-by-link target) | guest | amount, currency, tender, hosted fields | pending, succeeded, failed, unknown | EL-M | gst.pay@own or link token | AU-L | N-email receipt | A-form (3.3.4), A-auth | ML-P |
| SCR-GST-receipts | Invoices/receipts/credit notes download per JUR format | guest | documents, language, fiscal number | issued | EL-L | gst.documents@own | AU-0 | — | A-std (accessible PDF/HTML) | ML-P |
| SCR-GST-checkout-express | Express checkout: review folio, settle, invoice, feedback | guest | folio summary, payment on file, invoice details (company name/tax id) | eligible, requested, completed, needs_desk | EL-M | gst.checkout@own | AU-L | N-email invoice | A-form (3.3.4) | ML-P |
| SCR-GST-points | Points balance (available/pending/expiring) and history | guest | balance by state, transactions, expiry dates, tier | — | EL-L | loy.member.read@own | AU-0 | N-push earned | A-grid | ML-P |
| SCR-GST-points-redeem | Redeem points on eligible hotel purchases (no cash-out/transfer) | guest | eligible items, points required, partial tender | quoted, held, captured, released | EL-M | loy.redeem@own | AU-L | N-email | A-form (3.3.4) | ML-P |
| SCR-GST-referral-status | Partner Hub dashboard: own code/link, disclosure text, directly attributed completed bookings (privacy-preserving), commissions pending/approved/paid/reversed | referrer | code, link, disclosure, booking ids (masked), amounts, statuses | market_enabled, market_blocked (shows reason) | EL-L | loy.referrer@own | AU-0 | N-email statements | A-grid | ML-P |
| SCR-GST-partner-hub-onboarding | Referrer agreement (no fee), identity/tax/payment details, permitted channels, disclosure training | referrer | agreement version, country/legal entity, category, tax docs, payout details, channel list | draft, submitted, approved, rejected, suspended | EL-F | loy.referrer.onboard@own | AU-E (agreement) | N-email | A-form | ML-P |
| SCR-GST-partner-hub-statement | Commission statement with formula version and components per booking (privacy-limited) | referrer | booking id (masked), net revenue, allowed deductions, margin, rate, commission, status | per commission ledger | EL-L | loy.statement@own | AU-0 | N-email | A-grid | ML-P |
| SCR-GST-privacy-consent | Manage consents by purpose/channel; withdraw; see policy versions | guest | purposes (marketing email/SMS/WhatsApp, ID OCR, biometric, AI transcript, analytics), state, history | granted, withdrawn | EL-F | gst.consent@own | AU-W (consent record) | N-email confirmation | A-form | ML-P |
| SCR-GST-data-request | Request export/deletion/correction; see status and legal-retention explanations | guest | request type, identity verification, status, retained-record reasons | submitted, verifying, in_progress, completed, partially_completed | EL-F | gst.dsr@own | AU-P (on staff side) | N-email | A-form | ML-P |
| SCR-GST-lost-item-inquiry | Report lost item; privacy-limited matching outcome | guest | description, stay, location, contact, photo (optional) | submitted, possible_match (staff verifies), matched, not_found, closed | EL-F | gst.lost@own or public | AU-W | N-email | A-form | ML-P |
| SCR-GST-feedback-survey | Post-stay survey; low score opens recovery case; public review invite only after, never incentivised deceptively | guest | ratings, comments, consent to contact, review invite | not_started, submitted | EL-P | gst.survey (token) | AU-W | N-email invite | A-form | ML-P |
| SCR-GST-assisted-contact | Consistent help: call, approved WhatsApp, email, chat; hours and emergency guidance | guest | contact options, hours, current wait | — | EL-P | public | AU-0 | — | A-help | ML-P |

### 4.13 AI — guest AI assistant and staff handoff/QA

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-AI-chat-widget | Guest chat with transparent assistant identity, language auto/choose, consent for transcript, emergency guidance | guest, ai_assistant | disclosure banner, messages, language, consent state, human handoff button | idle, conversing, waiting_human, handed_off, outage_fallback | EL-P (outage → contact options) | public / guest | AU-W (transcript per consent) | — | A-live, A-help, keyboard/screen-reader operable | ML-P |
| SCR-AI-chat-source-panel | Show sources and freshness for each answer (policy page, KB article, live inventory time) | guest | citation title, version/date, link | — | EL-P | public | AU-0 | — | A-std | ML-P |
| SCR-AI-availability-card | Live availability/quote card from bounded tool; "draft only" until guest confirms in booking flow | guest | dates, room, total incl. taxes, quote expiry, "continue to book" | quoted, expired | EL-P | public | AU-W (tool call log) | — | A-time | ML-P |
| SCR-AI-handoff-request | Request human; collect contact preference; business-hours fallback | guest | reason, contact channel, consent, expected wait | requested, accepted, queued_after_hours | EL-P | public | AU-W | N-in staff | A-form | ML-P |
| SCR-AI-transcript-controls | Download/delete transcript; see retention | guest | transcript, retention date, delete action | retained, deleted | EL-F | gst.transcript@own | AU-W | — | A-std | ML-P |
| SCR-AI-agent-console | Staff inbox for AI handoffs with full context, suggested replies (labelled), PII masked | guest_relations, front_desk_agent, concierge | conversation, AI summary, tool calls made, guest identity match (uncertain), SLA | new, claimed, replied, resolved | EL-L | aia.handoff.handle | AU-W | N-in, N-esc | A-live | ML-C |
| SCR-AI-conversation-review | QA review: sampled conversations, disputed answers, hallucination/injection flags, corrective KB action | guest_relations, content_approver | conversation, rating, issue type, correction, KB article link | to_review, reviewed, corrected | EL-L | aia.review | AU-R | — | A-grid | ML-D |
| SCR-AI-kb-coverage | Unanswered/low-confidence questions → KB tasks | content_editor | question cluster, frequency, language, suggested article | open, article_created, dismissed | EL-L | aia.kb_gap.read | AU-0 | N-in weekly | A-grid | ML-D |
| SCR-AI-evaluation | Evaluation sets (EN/AR), prompt-injection tests, release gate results per model version | ai lead, it_admin | test set, pass rate, failures, model id/version | running, passed, failed | EL-D | aia.eval.run | AU-W | N-email results | A-grid | ML-D |
| SCR-AI-settings | Tools allowed, channels, languages, business hours, cost caps, outage fallback text, provider (local/remote) | it_admin, guest_relations | tool allow-list, channel toggles, caps, fallback, provider adapter & honesty label | draft, active | EL-F | aia.settings.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-AI-cost-usage | Conversations, tokens/compute, cost vs cap, containment rate | it_admin, gm | usage by channel/day, cost, containment, handoff rate | — | EL-D | aia.usage.read | AU-0 | N-in cap reached | A-chart | ML-D |

### 4.14 CORP — corporate web portal (MetriStay Business)

```
Corporate portal
├─ Sign in (SSO/MFA) ── Home
├─ Admin: users & roles ── cost centers ── budget ── agreement view (contracted rates, facilities)
├─ Book: room search ── event search (dates × attendees × layout) ── compare ── package builder ── RFQ to hotel
│        ── hold detail (expiry) ── approval queue ── booking confirm (deposit/PO)
├─ Manage: rooming list ── BEO review ── change request ── itinerary ── attendee check-in ── club/bar/parking entitlements ── parking passes ── travel requests
└─ Finance & insight: invoices ── credit statement ── reports ── messages
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-CORP-sign-in | Corporate realm sign-in via company SSO or MFA | corporate users | company domain → IdP, credentials | signed_out, sso_redirect, mfa | EL-F | public | AU-W | — | A-auth | ML-P |
| SCR-CORP-home | Corporate home (§3) | corporate_booker, corporate_admin, corporate_approver | tasks: holds expiring, approvals, rooming lists due, invoices due; spend vs budget | — | EL-D | corp.home@company | AU-0 | N-email digest | A-std | ML-C |
| SCR-CORP-users-roles | Manage company users, roles (booker/approver/admin/organizer), approval limits | corporate_admin | users, roles, cost centers, limits | invited, active, disabled | EL-L | corp.user.manage `+stepup` | AU-P | N-email invites | A-grid | ML-C |
| SCR-CORP-cost-centers | Cost centers, PO requirement, approval routing | corporate_admin | cost center, budget owner, PO required, approvers | active, inactive | EL-L | corp.costcenter.write | AU-W | — | A-grid | ML-C |
| SCR-CORP-budget | Budget vs spend by cost center | corporate_admin, corporate_approver | budget, committed, invoiced, remaining | — | EL-D | corp.budget.read | AU-0 | N-email threshold | A-chart | ML-C |
| SCR-CORP-agreement-view | Contracted rates, eligible facilities/packages, validity, cancellation and credit terms | corporate users | agreement version, rates, facilities, terms | active, expiring | EL-L | corp.agreement.read | AU-0 | N-email renewal | A-grid | ML-C |
| SCR-CORP-room-search | Search rooms at contracted rates for travelers | corporate_booker | dates, travelers, room type, rate (contracted only), total | — | EL-L | corp.search | AU-0 | — | A-form | ML-P |
| SCR-CORP-event-search | Search feasible configurations by dates × attendees × layout × rooms × F&B × club × parking × AV (critical flow W4) | corporate_booker, event_organizer | dates (multiple), attendees, layout, room nights, meal periods, hosted bar, club visit, parking passes, AV | — | EL-L (only feasible configs at contracted rates appear) | corp.event.search | AU-0 | — | A-form, A-grid | ML-C |
| SCR-CORP-event-compare | Side-by-side comparison of feasible options (price, capacity, space, date) | corporate_booker | option, spaces, capacity per layout, rooms available, totals incl. taxes, hold expiry if held | — | EL-L | corp.event.search | AU-0 | — | A-grid (row/column headers) | ML-C |
| SCR-CORP-package-builder | Adjust package components (menus, AV, parking, club) and see total recalc | event_organizer | components, quantities, prices, dietary counts | draft, priced | EL-F | corp.package.build | AU-W | — | A-form | ML-C |
| SCR-CORP-rfq-request | Send RFQ to hotel sales when outside contracted scope | event_organizer | requirements, budget, dates, notes | submitted, responded, accepted, declined | EL-F | corp.rfq.create | AU-W | N-in sales, N-email | A-form | ML-P |
| SCR-CORP-hold-detail | Composite hold with firm expiry; extend request; convert to booking | corporate_booker | hold components, expiry countdown, deposit required, approval status | held, extension_requested, expired, converting, confirmed | EL-M | corp.hold.manage | AU-W | N-email/N-push 24 h and 2 h before expiry | A-time, A-live | ML-P |
| SCR-CORP-approval-queue | Corporate approvers approve bookings/changes per policy | corporate_approver | request, cost center, amount, policy result | pending, approved, rejected | EL-M | corp.approval.decide (+limit) | AU-P | N-email/N-push | A-form | ML-P |
| SCR-CORP-booking-confirm | Confirm with deposit (PSP), PO number or credit; confirmation only after server ack | corporate_booker | payment method, PO, terms acceptance, total | confirming, confirmed, failed | EL-M | corp.booking.confirm | AU-L | N-email confirmation | A-form (3.3.4) | ML-P |
| SCR-CORP-rooming-list | Upload/edit rooming list against block; deadlines | event_organizer, corporate_booker | names, room type, sharing, dates, special needs, billing | draft, submitted, validated, applied | EL-L, EL-C | corp.rooming.write | AU-W | N-email deadline | A-grid | ML-C |
| SCR-CORP-beo-review | Review and sign BEO version; see changes | event_organizer | BEO version, changes highlighted, totals | sent, signed, superseded | EL-M | corp.beo.sign | AU-E | N-email new version | A-sig, A-grid | ML-C |
| SCR-CORP-change-request | Request event/booking changes; see price delta and approval | event_organizer | change, delta, capacity result | requested, approved, rejected | EL-M | corp.change.request | AU-W | N-email | A-form | ML-P |
| SCR-CORP-itinerary | Event and travelers itinerary (confirmed vs proposed) | event_organizer, travelers | segments, status, references | proposed, confirmed | EL-L | corp.itinerary.read | AU-0 | N-email updates | A-std | ML-P |
| SCR-CORP-attendee-checkin | Attendee list and on-site check-in status | event_organizer | attendees, badge, arrival, room link | expected, checked_in, no_show | EL-R | corp.attendee.manage | AU-W | — | A-grid | ML-P |
| SCR-CORP-club-bar-parking | Entitlements: hosted bar limits, club visit, parking passes consumed | event_organizer | entitlement, allowance, used, remaining | active, exhausted | EL-D | corp.entitlement.read | AU-0 | N-email threshold | A-chart | ML-P |
| SCR-CORP-parking-passes | Assign parking passes/plates to attendees | event_organizer | plate, attendee, validity, zone | issued, active, expired | EL-L | corp.parking.assign | AU-W | N-email/N-sms attendee | A-form | ML-P |
| SCR-CORP-travel-requests | Request airport transfer/taxi/flight quotes via hotel concierge where permitted | corporate_booker | request, travelers, consent, offers, status | as CON | EL-L | corp.travel.request | AU-W | N-email | A-form | ML-P |
| SCR-CORP-invoices | Invoices, credit notes, payment status, download | corporate_admin, corporate_approver | invoice, PO, amount, due, status | issued, partially_paid, paid, disputed | EL-L | corp.invoice.read | AU-0 | N-email | A-grid | ML-C |
| SCR-CORP-credit-statement | Credit limit, balance, aging, statement | corporate_admin | limit, balance, buckets | — | EL-D | corp.credit.read | AU-0 | N-email monthly | A-grid | ML-C |
| SCR-CORP-reports | Spend, room nights, events, compliance to policy | corporate_admin | reports, exports | — | EL-D | corp.report.read | AU-0 | N-email scheduled | A-chart, A-grid | ML-D |
| SCR-CORP-messages | Messages with hotel sales/events | corporate users | threads by booking/event | open, closed | EL-L | corp.message | AU-W | N-email/N-push | A-live | ML-P |

### 4.15 CAPP — corporate Android/iOS app (core-journey parity)

Parity rule (§F): the core journey — sign in → search (dates × attendees) → compare → hold → approve → confirm → rooming list → itinerary → invoices — is fully available on the app. Admin-heavy screens (users/roles, cost centers, reports) are web-only with a read summary in-app.

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-CAPP-sign-in | SSO/MFA sign-in with biometrics unlock of stored session (device-level, not hotel biometrics) | corporate users | company, SSO, device unlock | as CORP | EL-F | public | AU-W | — | A-auth | ML-N |
| SCR-CAPP-home | Corporate home tasks | corporate users | as CORP home | — | EL-O (read cache) | corp.home | AU-0 | N-push | A-std | ML-N |
| SCR-CAPP-event-search | Search feasible configurations | corporate_booker | as CORP event search | — | EL-L | corp.event.search | AU-0 | — | A-form | ML-N |
| SCR-CAPP-compare | Compare options (swipeable cards + table view toggle) | corporate_booker | as CORP compare | — | EL-L | corp.event.search | AU-0 | — | A-grid (table view) | ML-N |
| SCR-CAPP-hold | Hold with expiry countdown and extension request | corporate_booker | as CORP hold | as CORP | EL-M | corp.hold.manage | AU-W | N-push expiry | A-time | ML-N |
| SCR-CAPP-approvals | Approve/reject requests | corporate_approver | as CORP approvals | as CORP | EL-M | corp.approval.decide | AU-P | N-push | A-form | ML-N |
| SCR-CAPP-booking-confirm | Confirm booking with deposit/PO | corporate_booker | as CORP confirm | as CORP | EL-M | corp.booking.confirm | AU-L | N-push | A-form | ML-N |
| SCR-CAPP-rooming-list | Edit rooming list entries | event_organizer | as CORP rooming list | as CORP | EL-L | corp.rooming.write | AU-W | N-push deadline | A-form | ML-N |
| SCR-CAPP-beo-review | Review/sign BEO | event_organizer | as CORP BEO | as CORP | EL-M | corp.beo.sign | AU-E | N-push | A-sig | ML-N |
| SCR-CAPP-itinerary | Itinerary with offline availability | event_organizer, travelers | as CORP itinerary | as CORP | EL-O | corp.itinerary.read | AU-0 | N-push changes | A-std | ML-N |
| SCR-CAPP-attendee-checkin | On-site attendee check-in (scan badge/QR) | event_organizer | as CORP | as CORP | EL-R | corp.attendee.manage | AU-W | — | A-cam (manual search alternative) | ML-N |
| SCR-CAPP-parking-passes | Assign/view parking passes | event_organizer | as CORP | as CORP | EL-L | corp.parking.assign | AU-W | N-push | A-form | ML-N |
| SCR-CAPP-invoices | Invoices and statement | corporate_admin | as CORP | as CORP | EL-L | corp.invoice.read | AU-0 | N-push | A-std | ML-N |
| SCR-CAPP-messages | Messages with hotel | corporate users | as CORP | as CORP | EL-L | corp.message | AU-W | N-push | A-live | ML-N |
| SCR-CAPP-notifications | Notification inbox/preferences | corporate users | items, channels | — | EL-L | own | AU-0 | all | A-live | ML-N |

