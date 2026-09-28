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

