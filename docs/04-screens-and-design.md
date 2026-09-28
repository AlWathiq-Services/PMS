# 04 — Screens, Flows and Design System

**Pack:** MetriStay Hospitality Suite Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Governing source:** master prompt v3.0 §F (application families), §P.1 (simplicity, role home, WCAG 2.2 AA, low bandwidth), §E, §G, §K, §Q • **Conventions:** `docs/README.md` §3 (screen id `SCR-<app>-<name>`, actors §3.3, honesty labels §3.6). Architecture references: `docs/03-architecture.md`.
**Status:** design target. No screen exists. Wireframes are low-fidelity and annotate behaviour, not visual design.

> Decision ids in this document use the reserved range **D-971..D-999** (UX/design), registered in `docs/13`. Catalogue writers (`docs/01`) reference screens by the ids defined here; an id not in this document is a defect to be added here first.

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
| SCR-FIN-chargeback-case | Chargeback evidence and response | finance_clerk, financial_controller | txn, reason code, deadline, evidence (folio, signature, ID-free receipt) | received, evidence_submitted, won, lost | EL-F | fin.chargeback.respond | AU-E | N-esc deadline | A-form, A-time | ML-D |
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

### 4.16 PRK — parking and security

```
Parking/security
├─ Homes: parking attendant · security officer
├─ Lanes: lane monitor ── low-confidence review (W5) ── manual gate / override
├─ Permits ── plate registry ── sessions ── occupancy ── tariffs
├─ Exceptions ── reconciliation
└─ Device status (cameras, LPR server, gate controllers)
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-PRK-home | Parking attendant home (§3) | parking_attendant | review queue count, gate exceptions, occupancy, device health | — | EL-R | com.parking.operate | AU-0 | N-push review needed | A-live | ML-T |
| SCR-PRK-security-home | Security officer home: incidents, patrol tasks, alarms, parking exceptions | security_officer | incidents, patrol checklist, alarm feed, lost items to log | — | EL-R | saf.security.read | AU-0 | N-push critical (assertive) | A-live | ML-N |
| SCR-PRK-lane-monitor | Live lanes: latest observations, decision, gate state | parking_attendant | lane, plate read, confidence, matched permit, decision, gate state, camera snapshot (privacy-limited) | idle, vehicle_present, decided, gate_open, fault | EL-R (device offline → SCR-PRK-manual-gate) | com.parking.monitor | AU-0 | N-rt | A-live | ML-K |
| SCR-PRK-lane-review | Low-confidence/unmatched read review: compare image vs candidates, decide open/deny, correct plate (critical flow W5) | parking_attendant | snapshot, read value, confidence, candidate plates (fuzzy), permit/stay details (minimum), timer | pending, decided_open, decided_deny, corrected, escalated | EL-R | com.parking.review | AU-W (decision with reason) | N-push queue | A-live, A-time, keyboard shortcuts | ML-K |
| SCR-PRK-manual-gate | Manual gate open/close and override with reason; used during device/network failure | parking_attendant, security_officer | lane, reason code, plate (typed), permit lookup, photo (optional) | closed, opened_manual | EL-M (local site gateway command) | com.gate.override `+limit`; repeated overrides alert | AU-P | N-in supervisor on override | A-form, A-target | ML-K |
| SCR-PRK-permits | Permits for guests/corporate/staff/events; validity, zones | parking_attendant, front desk | holder type, plate, zone, validity, tariff/free, source (stay/event) | issued, active, expired, revoked | EL-L | com.parking_permit.write | AU-W | N-rt gateway | A-grid | ML-C |
| SCR-PRK-plate-registry | Registered plates with normalisation per region, linked holders | parking_attendant | plate, region, normalized, holders, notes | active, blocked | EL-L | com.plate.write | AU-W | — | A-grid | ML-C |
| SCR-PRK-sessions | Entry/exit sessions, duration, tariff, folio posting status | parking_attendant, cashier | session, entry/exit evidence, duration, tariff, posted flag | open, closed, posted, disputed | EL-L | com.parking.session.read | AU-L (posting) | — | A-grid | ML-C |
| SCR-PRK-occupancy | Zone occupancy vs capacity; reserved event capacity | parking_attendant, gm | zone, capacity, occupied, reserved, trend | normal, near_full, full | EL-R | com.parking.read | AU-0 | N-in near full | A-chart, A-live | ML-C |
| SCR-PRK-exceptions | Unmatched exits, tailgating, duplicate observations, disputes | parking_attendant, security_officer | exception, evidence, suggested action | open, resolved | EL-L | com.parking.exception | AU-W | N-in | A-grid | ML-C |
| SCR-PRK-tariffs | Tariff rules, free periods, guest/corporate/staff/event rates, tax category | property_admin, fnb/parking manager | rule, periods, prices, caps, tax | draft, active | EL-F | com.parking.tariff.write `+mc` | AU-W | — | A-form | ML-D |
| SCR-PRK-reconciliation | Sessions vs postings vs payments; manual overrides review | finance_clerk, security manager | sessions, charges, payments, overrides, variances | open, reconciled | EL-L | com.parking.reconcile | AU-W | — | A-grid | ML-D |
| SCR-PRK-device-status | Cameras/LPR server/gate controller health, certificates, last heartbeat | it_admin, security_officer | device, heartbeat, firmware, cert expiry, error rate | healthy, degraded, offline | EL-R | plt.device.read | AU-0 | N-push offline | A-live | ML-C |

### 4.17 ADM — integrations, platform and regulatory administration

```
Admin
├─ Home
├─ Organisation: tenant & properties ── property profile (outlets/modules enabled) ── feature flags ── departments
├─ Access: users ── roles & permissions ── access review ── break-glass
├─ Setup: rooms ── rate plans ── timed resources ── outlets ── service taxonomy ── RFQ policy ── workflow templates ── workflow simulator ── notification templates ── loyalty program ── referral program ── KPI dictionary ── app distribution
├─ Integrations: connectors ── connector detail ── integration health ── event monitor ── dead letters ── job monitor ── webhooks & API keys ── reconciliation hub
├─ Platform: secrets & certificates ── devices ── backup & restore ── deployment status ── localization ── data quality
├─ Jurisdiction: tree ── legal entities ── rule packs ── rule version review ── classifier test ── coverage dashboard ── activation gates ── tax config ── filing calendar ── filing submission ── government connectors
└─ Privacy & audit: consent purposes ── retention policies ── legal holds ── data subject requests ── audit log
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-ADM-home | IT/integration admin home (§3) | it_admin, integration_admin | connector health, dead letters, expiring secrets, backups, devices, access reviews | — | EL-D | plt.admin.read | AU-0 | N-in, N-push critical | A-live | ML-C |
| SCR-ADM-tenant-properties | Tenant and properties, time zones, business-date settings, profile (saas/on-prem) | tenant_admin | property, legal entity, IANA TZ, currency, business-date rollover, deployment profile | active, onboarding, suspended | EL-L | plt.property.write `+stepup` | AU-P | — | A-grid | ML-D |
| SCR-ADM-property-profile | Product packaging: limited/full service, enabled outlets/modules/amenities, languages | property_admin | service level, modules toggles, outlets, amenities, languages, markets | draft, active | EL-F | plt.profile.write `+mc` | AU-P | N-in affected managers | A-form | ML-D |
| SCR-ADM-feature-flags | Feature flags and activation per property/jurisdiction (with gate link) | property_admin, compliance_officer | flag, scope, state, gate dependency, change history | off, on, gated | EL-L | plt.flag.write `+mc` | AU-P | — | A-grid | ML-D |
| SCR-ADM-departments | Departments, cost centers link, managers | property_admin | department, cost center, head, parent | active, inactive | EL-L | plt.department.write | AU-W | — | A-grid | ML-D |
| SCR-ADM-users | Staff users, roles per property/department, MFA status, devices | property_admin, it_admin | user, roles, scopes, MFA, last login, status | invited, active, suspended, left | EL-L | iam.user.manage `+stepup` | AU-P | N-email invite | A-grid | ML-C |
| SCR-ADM-roles-permissions | Role definitions, permission sets, limits, field policies | tenant_admin | role, permissions, limits, field policies (mask/hide/decrypt) | draft, active | EL-F | iam.role.write `+mc` `+stepup` | AU-P | — | A-grid | ML-D |
| SCR-ADM-access-review | Periodic access review/recertification | dept heads, tenant_admin | user, role, last used, reviewer decision | pending, kept, revoked | EL-L | iam.review.decide | AU-P | N-email reviewers | A-grid | ML-D |
| SCR-ADM-break-glass | Emergency access with auto-expiry and post-review | privileged staff, dpo | reason, scope, expiry, review outcome | requested, active, expired, reviewed | EL-M | iam.breakglass `+stepup` | AU-P | N-push dpo/tenant_admin | A-form | ML-C |
| SCR-ADM-room-setup | Buildings, floors, room types, rooms, attributes, accessible/connecting | property_admin | type, room, features, floor, accessible, connecting | draft, active, retired | EL-L | inv.room.configure | AU-W | — | A-grid | ML-D |
| SCR-ADM-rate-plan-setup | Rate plans, derivations, occupancy/child rules, packages, policies | revenue_manager, property_admin | plan, base/derived, restrictions, cancellation/deposit, tax inclusion | draft, active | EL-F | inv.rateplan.configure | AU-W | — | A-form | ML-D |
| SCR-ADM-timed-resource-setup | Spaces/partitions, layouts/capacities, buffers, tables, parking zones, kitchen slots, club venues | property_admin | resource, mode (exclusive/pooled), components, capacities per layout, buffers | draft, active | EL-F | inv.resource.configure | AU-W | — | A-form, A-map | ML-D |
| SCR-ADM-outlet-setup | Outlets, POS terminals, KDS stations, printers, tax categories | property_admin, fnb_manager | outlet, type, devices, service charge, tip rules | draft, active | EL-F | com.outlet.configure | AU-W | — | A-form | ML-D |
| SCR-ADM-service-taxonomy | Vendor service categories, required credentials per category/jurisdiction, owning department | procurement_approver, property_admin | category tree, credential requirements, default search scope | draft, active | EL-F | vnd.taxonomy.write `+mc` | AU-P | — | A-grid | ML-D |
| SCR-ADM-rfq-policy | Minimum quotes per category/value/jurisdiction, weight templates, sealed rules, sample retention start event | procurement_approver | thresholds, min quotes, weights, sealed flag, retention start event (D-978), approval matrix | draft, active | EL-F | stk.policy.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-ADM-workflow-templates | Versioned workflow/task templates, triggers, SLAs, escalations (M63) | property_admin, dept heads | template, trigger, steps, owners, SLA, escalation, version | draft, simulated, active, retired | EL-F | plt.workflow.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-ADM-workflow-simulator | Dry-run a template against sample events before activation (SF63.2.2) | property_admin | template version, sample events, resulting tasks/notifications | run, passed, failed | EL-D | plt.workflow.simulate | AU-W | — | A-grid | ML-D |
| SCR-ADM-notification-templates | Message templates per channel/language, WhatsApp template approval status | property_admin, marketing_manager | template, channel, languages, variables, BSP approval | draft, submitted, approved, rejected | EL-F | plt.template.write | AU-W | — | A-form | ML-D |
| SCR-ADM-loyalty-program | Points rules versions, eligibility, caps, expiry, tiers, fraud rules | referral_program_admin, financial_controller | rule version, earn/burn, caps, expiry, liability rate | draft, active, superseded | EL-F | loy.program.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-ADM-referral-program | Referral program per market: gate status, referrer categories, formula versions, kill switch, disputes, fraud flags (F31.2) | referral_program_admin, compliance_officer | market, gate, categories, formula version, disclosures, kill switch, disputes | disabled, gated, enabled | EL-F | loy.referral.configure `+mc` `+stepup`; kill switch single-actor allowed | AU-P | N-in compliance | A-form | ML-D |
| SCR-ADM-kpi-dictionary | KPI definitions/denominators/versions (M65) | financial_controller, data owner | KPI, formula, denominator, source, owner, version | draft, active | EL-F | bi.kpi.write `+mc` | AU-W | — | A-form | ML-D |
| SCR-ADM-app-distribution | App builds, channels (store/MDM/enterprise), versions, min supported, rollout % | it_admin | app, platform, build, signing identity, channel, min version, rollout | building, in_review, released, halted | EL-L | plt.app.release `+mc` | AU-P | N-email | A-grid | ML-D |
| SCR-ADM-connectors | All adapters/connectors with honesty label, capabilities, owner | integration_admin | connector, port, adapter, label, capabilities summary, last success | per label | EL-L | plt.connector.read | AU-0 | — | A-grid | ML-C |
| SCR-ADM-connector-detail | Capability manifest, credentials (write-only), environment, contract/sandbox/cert evidence, manual fallback | integration_admin | manifest, secret refs (never shown), endpoints, rate limits, evidence docs, label history | configured, testing, active, suspended | EL-F | plt.connector.write `+mc` `+stepup` | AU-P | N-in label change | A-form | ML-D |
| SCR-ADM-integration-health | Health per connector: error rate, latency, circuit state, queued items | integration_admin, it_admin | metrics, circuit breaker, last error, queue depth | healthy, degraded, down | EL-R | plt.health.read | AU-0 | N-push down | A-chart, A-live | ML-C |
| SCR-ADM-event-monitor | Outbox lag, event throughput, consumer status, replay | integration_admin | event type, lag, consumers, failures | — | EL-R | plt.event.read | AU-0 | N-in lag breach | A-chart | ML-D |
| SCR-ADM-dead-letters | Dead-lettered events/webhooks with replay/skip (skip money/stock needs second approver) | integration_admin | event, consumer, error, attempts, payload (redacted) | dead, replayed, skipped | EL-L | plt.deadletter.replay; skip `+mc` | AU-P | N-in | A-grid | ML-D |
| SCR-ADM-job-monitor | Background jobs/sagas: state, deadlines, retries, re-drive | integration_admin | job/saga, state, next run, attempts, error | scheduled, running, failed, completed | EL-L | plt.job.manage | AU-W | N-in failures | A-grid | ML-D |
| SCR-ADM-webhooks-api-keys | Developer platform: API clients, scopes, webhook subscriptions, versions, sandbox (M33) | integration_admin | client, scopes, keys (hashed), subscriptions, versions, deprecation notices | active, rotated, revoked | EL-L | plt.api_client.manage `+stepup` | AU-P | N-email integrators on deprecation | A-grid | ML-D |
| SCR-ADM-reconciliation-hub | Cross-partner reconciliation status (PSP, bank, channel, bill-pay, travel, POS, parking) | integration_admin, financial_controller | partner, period, matched %, breaks, value at risk | open, reconciled, exception | EL-D | plt.recon.read | AU-0 | N-in breaks | A-grid | ML-D |
| SCR-ADM-secrets-certs | Secret/certificate inventory, expiry, rotation (values never displayed) | it_admin | secret ref, owner, expiry, rotation date, device certs | valid, expiring, expired, rotated | EL-L | plt.secret.manage `+stepup` | AU-P | N-push expiring | A-grid | ML-D |
| SCR-ADM-devices | Device registry: POS, KDS, tablets, lanes, gateways, BMS connectors; firmware; health (M64) | it_admin | device, type, location, firmware, cert, heartbeat, owner | enrolled, healthy, degraded, offline, revoked | EL-R | plt.device.manage | AU-W | N-push offline | A-grid | ML-C |
| SCR-ADM-backup-restore | Backup status, restore test evidence, RPO/RTO measured vs target | it_admin | last backup, PITR window, restore test date/result, verification checks | ok, warning, failed | EL-D | plt.backup.read; restore test `+stepup` | AU-P | N-push failure | A-grid | ML-D |
| SCR-ADM-deployment-status | Versions, migrations, maintenance windows, change approvals, rollback | it_admin | version, migration state, window, approvals | planned, in_progress, completed, rolled_back | EL-D | plt.deploy.read / .approve `+mc` | AU-P | N-email staff notice | A-std | ML-D |
| SCR-ADM-localization | Translation keys and content translations, coverage per language, RTL preview | property_admin, content_editor | key, EN, AR (+packs), status, reviewer | missing, translated, approved | EL-L | plt.i18n.write | AU-W | — | A-grid | ML-D |
| SCR-ADM-data-quality | Guest duplicates, stale feeds, missing mappings, orphan records (M65) | data owner, it_admin | issue type, count, examples, owner | open, resolved | EL-L | bi.dq.read | AU-R | N-in | A-grid | ML-D |
| SCR-ADM-jurisdiction-tree | Country → subnational → municipality nodes (ISO codes, EN/AR names) | compliance_officer | node, level, code, parent, names | draft, active | EL-L | jur.node.write `+mc` | AU-P | — | A-grid (tree) | ML-D |
| SCR-ADM-legal-entities | Legal entities, tax registrations, functional currency, property mapping | compliance_officer, financial_controller | entity, domicile node, registrations (masked), properties | draft, active | EL-F | jur.entity.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-ADM-rule-packs | Rule packs per country and category with versions, labels and status (F44.2) | compliance_officer | pack, category, versions, effective ranges, status, label, confidence | per version states | EL-L | jur.pack.read | AU-0 | N-in expiring | A-grid | ML-D |
| SCR-ADM-rule-version-review | Review a rule version: sources, rules, fixtures, reviewer/counsel sign-off (maker-checker) | compliance_officer, counsel (external reviewer role) | rules, sources (URL, accessed date), fixtures results, review comments, decision | draft, in_review, verified, rejected, expired | EL-F | jur.version.verify `+mc` `+stepup` | AU-P | N-in reviewer | A-form | ML-D |
| SCR-ADM-classifier-test | Simulate classification for entity/site/service/guest residency/tax date; show precedence trace | compliance_officer, financial_controller | inputs, resolved versions, trace, unknowns | — | EL-F | jur.classify.simulate | AU-W | — | A-form | ML-D |
| SCR-ADM-coverage-dashboard | Five-market coverage/unknowns and blocked features (SF44.2.8); compliance home | compliance_officer, dpo | market × category matrix: verified/draft/expired/unknown, gates blocked, filings due | — | EL-D | jur.coverage.read | AU-0 | N-in changes | A-grid (not colour only), A-chart | ML-D |
| SCR-ADM-activation-gates | Feature activation gates per property/jurisdiction with exception approvals | compliance_officer | feature key, obligation, state, exception approver, expiry | blocked, manual_only, enabled | EL-L | jur.gate.write `+mc` `+stepup` | AU-P | N-in owners | A-grid | ML-D |
| SCR-ADM-tax-config | Tax categories and product mapping (room, F&B, catering, parking, fees) to rule versions | compliance_officer, financial_controller | product/revenue code → tax category → rule version preview | draft, active | EL-F | jur.tax.map `+mc` | AU-P | — | A-grid | ML-D |
| SCR-ADM-filing-calendar | Filing/remittance calendar per obligation and entity | compliance_officer, financial_controller, payroll_officer | obligation, period, due, submission mode, status | upcoming, due, overdue, filed | EL-L | jur.filing.read | AU-0 | N-in, N-esc due | A-grid | ML-D |
| SCR-ADM-filing-submission | Prepare/approve/submit filing via connector or manual portal with receipt upload; never "submitted" without receipt (SF38.3.x) | compliance_officer, financial_controller | artefact, hash, approvals, mode (API/file/portal/manual), receipt no, evidence | prepared, approved, submitted, accepted, rejected, manual_filed | EL-M | jur.filing.submit `+mc` `+stepup` | AU-P + AU-E | N-in status | A-form (3.3.4) | ML-D |
| SCR-ADM-gov-connectors | Government connector registry: agency, protocol, credentials, certification status (no scraping) | integration_admin, compliance_officer | agency, obligation, protocol, sandbox status, cert expiry, label | registered, sandbox_tested, certified, blocked | EL-L | jur.connector.write `+mc` `+stepup` | AU-P | N-in expiry | A-grid | ML-D |
| SCR-ADM-consent-purposes | Consent purposes, wording versions, channels, jurisdictions | dpo | purpose, wording EN/AR, version, lawful basis note, channels | draft, active, retired | EL-F | iam.consent_purpose.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-ADM-retention-policies | Retention per record class and jurisdiction (ID images, samples, transcripts, payroll, folios) | dpo, compliance_officer | record class, keep for, start event, rule version, purge job status | draft, active | EL-F | iam.retention.write `+mc` | AU-P | N-in purge failures | A-grid | ML-D |
| SCR-ADM-legal-holds | Place/release legal holds overriding retention | dpo, financial_controller | scope, reason, approver, released_at | active, released | EL-F | iam.hold.write `+mc` `+stepup` | AU-P | — | A-form | ML-D |
| SCR-ADM-dsr-requests | Data subject requests: verify identity, collect, redact, respond; explain retained records | dpo | request, subject, systems searched, retained reasons, response | received, verifying, in_progress, completed | EL-L | iam.dsr.handle | AU-P + AU-R | N-email subject | A-grid | ML-D |
| SCR-ADM-audit-log | Search audit events and sensitive access log; export for auditors | auditor, dpo, tenant_admin | actor, action, resource, time, reason, approvals, correlation | — | EL-L | iam.audit.read | AU-R (export) | — | A-grid | ML-D |

### 4.18 MED — property content, website and marketing/CRM

```
Content
├─ Home ── library ── upload ── asset detail (rights, captions, tags) ── rights expiry
├─ Enhancement: queue ── enhance compare (original vs enhanced) ── approval
├─ Publishing: listing preview ── publish status (CDN/syndication) ── takedown
├─ Website: pages ── SEO & structured data ── KB articles (AI)
└─ Marketing/CRM: home ── segments ── campaign editor ── campaign results ── reviews inbox ── review response ── surveys ── attribution ── offers & vouchers
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-MED-home | Content home (§3) | content_editor, content_approver | approvals, comparisons, rights expiring, publish failures, KB reviews | — | EL-D | med.home.read | AU-0 | N-in | A-std | ML-C |
| SCR-MED-library | Media library with filters (room/facility tags, rights, status, language of captions) | content_editor | thumbnail, title, tags, status, rights expiry, alt text coverage | uploaded, scanned, tagged, in_review, approved, published, withdrawn | EL-L | med.asset.read | AU-0 | — | A-grid (alt text shown) | ML-C |
| SCR-MED-upload | Multi-file upload with malware scan, type checks, rights declaration | content_editor | files, owner/rights, model/property releases, tags | uploading, quarantined_scanning, scanned, rejected | EL-C | med.asset.upload | AU-E | N-in scan failures | A-form | ML-C |
| SCR-MED-asset-detail | Rights, captions/alt text EN/AR, tags (room/venue), crops, derivatives, versions | content_editor | rights (owner, expiry, usage), alt EN/AR, captions/subtitles, tags, crops, renditions | as library | EL-F | med.asset.write | AU-W | — | A-media, A-form | ML-D |
| SCR-MED-rights-expiry | Assets with expiring/expired rights; auto-withdraw schedule | content_editor, content_approver | asset, expiry, channels published, action | expiring, expired, renewed, withdrawn | EL-L | med.rights.read | AU-W | N-in 30/7 days | A-grid | ML-C |
| SCR-MED-enhancement-queue | Local AI enhancement jobs: preset, model/licence, compute cost, status | content_editor | job, preset, model version, licence ref, runtime, cost, retries | queued, running, done, failed | EL-L | med.enhance.run | AU-W | N-in done/failed | A-grid | ML-D |
| SCR-MED-enhance-compare | Original vs enhanced side-by-side/slider; checks: no fabricated features/views/dimensions (SF39.2.4) | content_approver | original, enhanced, diff overlay, preset, provenance metadata, checklist | pending, approved, rejected | EL-F | med.enhance.approve (≠ uploader) | AU-P | N-in editor | A-drag (slider has buttons), A-media | ML-D |
| SCR-MED-approval | Approve assets/pages for publication with schedule | content_approver | asset/page, checklist (rights, alt text, accuracy), schedule | pending, approved, rejected, scheduled | EL-F | med.publish.approve `+mc` | AU-P | N-in editor | A-form | ML-D |
| SCR-MED-listing-preview | Preview property/room/facility listing as guest sees it (web/app/channel) in EN/AR, mobile/desktop | content_editor, content_approver | rendered preview, locale, device, bandwidth simulation | — | EL-P | med.preview | AU-0 | — | A-std (RTL preview) | ML-D |
| SCR-MED-publish-status | CDN/website/channel/AI-KB publication status per asset/version; rollback | content_editor | channel, version, status, last sync, error | pending, published, failed, rolled_back | EL-L | med.publish.read | AU-W | N-in failures | A-grid | ML-C |
| SCR-MED-takedown | Remove content everywhere (rights issue, complaint) with access logs | content_approver, dpo | asset, reason, channels, completion evidence | requested, in_progress, completed | EL-M | med.takedown `+stepup` | AU-P | N-in | A-form | ML-C |
| SCR-MED-website-pages | Website pages/sections with owner/version, multilingual, accessibility content | content_editor | page, blocks, languages, owner, review date | draft, in_review, published | EL-F | med.page.write | AU-W | — | A-form | ML-D |
| SCR-MED-seo-structured-data | Metadata, structured data (hotel/room/offer), redirects, sitemap, performance budget report | content_editor, marketing_manager | meta per page/locale, schema validation result, redirects, page weight/LCP | valid, warnings, errors | EL-L | med.seo.write | AU-W | — | A-grid | ML-D |
| SCR-MED-kb-articles | Knowledge base for AI and staff with owners/review dates, EN/AR (SF40.1.1) | content_editor, dept owners | article, languages, owner, review due, sources, AI-enabled flag | draft, approved, expired | EL-F | med.kb.write | AU-W | N-in review due | A-form | ML-D |
| SCR-MED-marketing-home | Marketing/guest relations home (§3) | marketing_manager, guest_relations | reviews to respond, campaigns awaiting approval, recovery cases, detractors, attribution anomalies | — | EL-D | pty.marketing.read | AU-0 | N-in | A-chart | ML-C |
| SCR-MED-segments | Segments using permitted criteria; consent-eligible counts | marketing_manager | criteria, size, eligible by channel, exclusions (suppression) | draft, active | EL-L | pty.segment.write | AU-W | — | A-form | ML-D |
| SCR-MED-campaign-editor | Campaign: segment, channel, template, schedule, frequency cap, approval | marketing_manager | segment, channel, template/version, schedule, cap, approval | draft, pending_approval, scheduled, sending, completed, cancelled | EL-F | pty.campaign.write; approve `+mc` | AU-P | N-in approver | A-form | ML-D |
| SCR-MED-campaign-results | Delivery, conversion to paid stays, cost, suppression/opt-outs | marketing_manager | sent, delivered, failed, conversions, revenue, cost per stay | — | EL-D | pty.campaign.read | AU-0 | — | A-chart | ML-D |
| SCR-MED-reviews-inbox | External reviews via approved channels + internal surveys, subject tagging | guest_relations, marketing_manager | source, rating, text, subject tags, linked stay (if matched), response status | new, drafted, approved, published | EL-L | pty.review.read | AU-0 | N-in new low rating | A-grid | ML-C |
| SCR-MED-review-response | Draft/approve public response; privacy and anti-retaliation checks | guest_relations, gm (approver) | review, draft (AI-assisted labelled), approval, publish status | draft, pending_approval, published | EL-F | pty.review.respond `+mc` | AU-P | — | A-form | ML-C |
| SCR-MED-surveys | Survey templates and results; detractor → recovery case | guest_relations | template, responses, score trend, cases opened | active, archived | EL-L | pty.survey.write | AU-W | N-in detractor | A-chart | ML-D |
| SCR-MED-attribution | Visit → quote → paid stay attribution by source/campaign; bot filtering; channel net cost (F51.2) | marketing_manager, revenue_manager | sessions (consented), quotes, bookings, paid stays, cost, net contribution | — | EL-D | dst.attribution.read | AU-0 | — | A-chart, A-grid | ML-D |
| SCR-MED-offers-vouchers | Configure upsell offers, packages, gift vouchers (terms, capacity, accounting treatment) | marketing_manager, revenue_manager, financial_controller | offer, eligibility, price, capacity link, tax, liability account, expiry | draft, active, expired | EL-F | com.offer.configure `+mc` for vouchers | AU-P | — | A-form | ML-D |

### 4.19 SAF — safety, incidents, lost & found, guest verification, inspections, risk

```
Safety
├─ Home ── incident command ── incident detail ── post-incident review ── drills
├─ Intake: quick incident report (mobile) ── sensor alert review ── playbooks ── on-call
├─ Lost & found: intake ── register ── custody ── claims ── release
├─ Guest verification: ID intake (staff-assisted) ── OCR correction ── mismatch review ── signature record ── OTP/QR verify ── consent record
├─ Inspections: schedule ── run ── nonconformance ── permit calendar
└─ Risk: insurance register ── claim file ── continuity plans
```

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-SAF-home | Safety home: active incidents, alerts to triage, inspections due, lost items, continuity status | duty_manager, security_officer | incidents by severity, unconfirmed alerts, due inspections, open claims | — | EL-R | saf.home.read | AU-0 | N-push critical | A-live (assertive for critical) | ML-C |
| SCR-SAF-incident-report | Quick report by staff (mobile, offline-capable) with location, type, photo; call prompt | all staff, guest (via GST/AI escalation) | type, location (room/area), description, photo, injuries?, reporter | submitted, queued | EL-O (queued + "call now" button) | saf.incident.report | AU-E | N-push on-call | A-form, A-cam, A-target | ML-N |
| SCR-SAF-sensor-alert-review | Review camera/BMS/fire alerts: confidence, dedup, footage pointer; confirm or dismiss (human) | security_officer, duty_manager | source, device, confidence, location, related signals, pointer | new, confirmed (→ incident), false_alarm, duplicate | EL-R | saf.alert.triage | AU-E | N-push/N-voice critical | A-live, A-time | ML-C |
| SCR-SAF-incident-command | Command board: incident, playbook steps, dispatches/acks, timers, guest welfare, room/asset impact, emergency services contact | duty_manager (commander), security_officer | incident, playbook version, step status, dispatch list, ack timers, affected rooms/guests, external services contacted | reported, triaged, confirmed, responding, contained, closed | EL-R (offline → printed playbook + phone tree in SCR-OPS-outage-mode) | saf.incident.command | AU-E (hash-chained log) | N-push/N-sms/N-voice dispatch; N-esc on ack timeout | A-live (assertive), A-time | ML-T |
| SCR-SAF-incident-detail | Chronology, evidence pointers (restricted), corrective actions, claim link | duty_manager, security manager | log entries, evidence refs, actions, insurer claim | open, closed | EL-F | saf.incident.read; evidence field:restricted | AU-E + AU-R | — | A-grid | ML-C |
| SCR-SAF-playbooks | Versioned playbooks by incident type (fire, medical, security, gas, flood, outage, food safety) | security manager, gm | steps, roles, timeouts, contacts, local emergency numbers | draft, approved, superseded | EL-F | saf.playbook.write `+mc` | AU-P | — | A-form | ML-D |
| SCR-SAF-on-call | On-call roster and contact order for incidents | duty_manager, security manager | role, person, time window, channels, backup | scheduled, active | EL-L | saf.oncall.write | AU-W | N-push shift start | A-grid | ML-C |
| SCR-SAF-post-incident-review | Post-incident review, root cause, corrective work orders, lessons | gm, security manager | timeline summary, findings, actions (WO links), owners, due dates | draft, approved, actions_closed | EL-F | saf.review.write | AU-W | N-in action owners | A-form | ML-D |
| SCR-SAF-drills | Drill planning and recovery-time evidence (M68) | security manager | scenario, date, participants, measured times, gaps | planned, conducted, reviewed | EL-L | saf.drill.write | AU-W | N-push participants | A-grid | ML-C |
| SCR-SAF-lost-intake | Log found item: finder, location, time, discreet description, photo, barcode label, sealed bag no | housekeeper, security_officer, front desk | item category, description (restricted fields), photo, location, finder, seal no | logged (temp id offline), stored | EL-O, EL-C | saf.lost.intake | AU-E | — | A-form, A-cam | ML-N |
| SCR-SAF-lost-register | Search/manage lost items with privacy-limited staff search | security_officer, front desk | item no, category, date, location, status, dispose-after | logged, stored, matched, released, shipped, disposed, donated | EL-L | saf.lost.read (restricted description for security role) | AU-R | N-in disposal due | A-grid | ML-C |
| SCR-SAF-lost-custody | Chain-of-custody transfers with signatures | security_officer | from, to, time, seal no, signature | per custody event | EL-O | saf.lost.custody | AU-E | — | A-sig | ML-N |
| SCR-SAF-lost-claims | Match guest inquiries to items; claimant verification questions | security_officer, guest_relations | inquiry, candidate items (not shown to claimant), verification answers | open, matched, rejected | EL-L | saf.lost.match | AU-R | N-email claimant | A-grid | ML-C |
| SCR-SAF-lost-release | Authorised release/courier with ID check and fees | security_officer (maker), duty_manager (checker for high value) | claim, identity check, release method, courier ref, fee, signature | approved, released, shipped | EL-M | saf.lost.release `+mc` high value | AU-E + AU-P | N-email guest | A-sig, A-form | ML-C |
| SCR-SAF-id-intake | Staff-assisted ID capture at desk (scanner/camera) per JUR document rules; manual entry alternative | front_desk_agent | document type, images (restricted bucket), jurisdiction rule version, path (OCR/manual) | started, captured, extracted, failed | EL-C | idv.capture; field:id_document | AU-E + AU-R | — | A-cam (manual path) | ML-T |
| SCR-SAF-ocr-correction | Field-by-field OCR result with confidence; staff/guest corrections flagged | front_desk_agent | fields, confidence, original vs corrected, who corrected | extracted, confirmed, mismatch_review | EL-F | idv.confirm | AU-E | — | A-form | ML-T |
| SCR-SAF-id-mismatch-review | Review mismatches/authenticity cues; decide accept/manual verification/reject (no automated denial on biometrics) | front_office_manager, duty_manager | mismatch details, authenticity cues (if enabled), guest explanation, decision | pending, accepted, manual_verified, rejected | EL-F | idv.review `+mc` for reject | AU-P + AU-R | N-in FO manager | A-form | ML-C |
| SCR-SAF-signature-record | Signature envelope evidence: document hash, intent text, timestamp, provider ref, verification | front desk, dpo, auditor | envelope, hashes, signer, time, method, verification result | signed, verified, revoked (withdrawal recorded, not deleted) | EL-F | idv.signature.read | AU-E + AU-R | — | A-std | ML-D |
| SCR-SAF-otp-qr-verify | Staff view of OTP/QR verification attempts, delivery failures, manual in-person alternative | front_desk_agent | challenge, channel, delivery status, attempts, binding, expiry | sent, verified, expired, locked, delivery_failed | EL-F | idv.otp.read; manual verify `+mc` | AU-E | N-sms/N-wa resend | A-time | ML-T |
| SCR-SAF-consent-record | Guest consent records relevant to ID/biometric/signature/messaging with wording versions | front_desk_agent, dpo | purpose, action, wording version, evidence, time | granted, withdrawn | EL-L | iam.consent.read | AU-R | — | A-grid | ML-C |
| SCR-SAF-inspections | Inspection schedule by area (room, kitchen, pool, fire, pest, water) per JUR checklist | security manager, executive_chef, chief_engineer | template, area, assessor, due, last result | scheduled, due, overdue, completed | EL-L | saf.inspection.read | AU-0 | N-in due, N-esc overdue | A-grid | ML-C |
| SCR-SAF-inspection-run | Execute checklist with readings/photos; offline | credentialed assessor | items, readings, photos, pass/fail, comments | in_progress, completed | EL-O | saf.inspection.perform | AU-E | N-in failures | A-form, A-cam | ML-N |
| SCR-SAF-nonconformance | Findings: severity, service stoppage, quarantine/room block/recall, owner, retest and independent release | security manager, dept heads | finding, severity, stop flag, linked holds, actions, retest result | open, contained, corrected, verified_closed | EL-F | saf.nc.manage; release `+mc` | AU-P | N-esc | A-form | ML-C |
| SCR-SAF-permit-calendar | Permits/licences/inspections calendar with expiry and evidence | compliance_officer, security manager | permit, authority, expiry, evidence, owner | valid, expiring, expired | EL-L | saf.permit.read | AU-W | N-in 60/30/7 days | A-grid | ML-C |
| SCR-SAF-insurance-register | Policies, coverage, premiums, renewals (F68.1) | financial_controller, gm | insurer, policy, coverage, deductible, expiry, premium payable link | active, expiring, lapsed | EL-L | saf.insurance.read | AU-0 | N-in renewal | A-grid | ML-D |
| SCR-SAF-claim-file | Claim evidence packet from incidents, adjuster access scope, status, settlement | financial_controller, security manager | claim, incidents, evidence refs, adjuster access, reserve, settlement | draft, submitted, under_review, settled, denied | EL-F | saf.claim.write | AU-E | N-in status | A-form | ML-D |
| SCR-SAF-continuity-plans | Business impact scenarios, crisis role tree, guest welfare/relocation, supplier backups (F68.2) | gm, security manager | scenario, impact, roles/contacts, manual workflows, recovery target, last drill | draft, approved, tested | EL-F | saf.continuity.write | AU-W | — | A-form | ML-D |

---

## 5. Low-fidelity annotated wireframes — 10 critical flows

Conventions: `[Button]`, `( ) radio`, `[x] checkbox`, `<field>`, `{state/annotation ref}`, `▸` drill link. Numbers ① ② … refer to the annotation list under each frame. Frames are shown LTR; RTL mirrors layout per §6 (numbers, codes and plates stay LTR-isolated).

### W1 — Guest booking checkout (SCR-GST-room-results → extras → guest-details → payment → confirmation)

```
┌─ SCR-GST-room-results ──────────────────────────────── EN | ع ─ [Help ☎]─┐
│ Muscat Bay Hotel   12–14 Nov · 2 adults · 1 room   [Change search]          │
│ Filters: [Accessible room] [Breakfast] [Free cancellation]   Sort: Price ▾ │
│ ┌──────────────────────────────────────────────────────────────────────────┐│
│ │ [img: Deluxe King, alt="Deluxe king room with sea-facing balcony"]       ││
│ │ Deluxe King · 32 m² · sea view · roll-in shower available ①            ││
│ │  ( ) Room only   Flexible — free cancel until 10 Nov 18:00              ││
│ │                  OMR 84.000 total for 2 nights ② incl. VAT & fees ▸     ││
│ │  ( ) Breakfast   Non-refundable     OMR 92.500 total ...                 ││
│ │                                                  [Select]                ││
│ └──────────────────────────────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────────────────────────┘
        │ Select → server creates INVENTORY_HOLD + QUOTE (policy snapshot) ③
        ▼
┌─ SCR-GST-extras ─────────────────────────── Held for 14:52 [Need more time?] ④┐
│ Add to your stay (optional)                                                    │
│ [ ] Upgrade to Junior Suite +OMR 25.000/night   (2 left)                       │
│ [ ] Parking, 2 nights  +OMR 6.000                                              │
│ [ ] Late checkout 15:00 +OMR 10.000  (confirmed on arrival day if clean ready)⑤│
│                                            Total OMR 84.000   [Continue]       │
└────────────────────────────────────────────────────────────────────────────────┘
        ▼
┌─ SCR-GST-guest-details ────────────────────────────────────────────────────────┐
│ Booker  <Full name>  <Email>  <Mobile +968 ▾>  <Country ▾>                     │
│ Guests  [ ] I am staying   <Guest 2 name (optional)>                           │
│ Arrival time <▾>   Requests <textarea>   Accessibility needs <textarea> ⑥       │
│ Referral / promo code <      >  (disclosed: referrer may be paid) ⑦            │
│ [ ] Send me offers by email (optional, unticked) ⑧                             │
│                                                            [Review & pay]      │
└────────────────────────────────────────────────────────────────────────────────┘
        ▼
┌─ SCR-GST-payment ──────────────────────────────────────────────────────────────┐
│ Review: Deluxe King · 12–14 Nov · 2 adults                                     │
│   Room 2 nights ............. OMR 74.000                                       │
│   VAT ........................ OMR  3.700   (rule: OM VAT v2026-01, verified) ⑨│
│   Tourism levy / fees ........ OMR  6.300                                      │
│   Total ...................... OMR 84.000   Pay now: OMR 84.000               │
│ Policies ▸  [x] I accept the cancellation and hotel policies                   │
│ ┌ PSP hosted card fields (iframe) ⑩ ┐  or  [Apple Pay] [Pay at hotel*]         │
│ └───────────────────────────────────┘      *if rate allows                      │
│ [Pay OMR 84.000]  ← disabled after click; shows "Processing…" ⑪                │
└────────────────────────────────────────────────────────────────────────────────┘
        ▼ server: capture/authorize OK + hold→sold conversion OK + policy OK ⑫
┌─ SCR-GST-confirmation ─────────────────────────────────────────────────────────┐
│ ✓ Booking confirmed  MS-7K4Q2P                                                 │
│ Next: [Pre-check-in (2 min)]  [Add to calendar]  [Manage booking]              │
└────────────────────────────────────────────────────────────────────────────────┘
```
① Accessible features come from verified room attributes, never marketing text. ② **Total price including taxes/fees on results** (SF51.2.2); per-night breakdown in drill. ③ Hold created on select, not on page view; bot filtering applies. ④ Countdown announced to screen readers at 5/1 min; "Need more time" extends once if inventory policy allows (A-time). ⑤ Upsells never promise what cleaning capacity cannot deliver (SF54.1.2). ⑥ Accessibility notes are purpose-limited (field:health). ⑦ Referral disclosure shown; attribution only via code/link (`docs/03` §5.8). ⑧ Marketing consent separate and unticked. ⑨ Tax line shows rule version; if JUR rule is `unknown`, the rate is not sellable and the page shows an assisted booking path. ⑩ Card data never touches MetriStay (PCI boundary). ⑪ **EL-M**: timeout → "We're confirming your payment…" with automatic status inquiry; no second charge. ⑫ Confirmation only after inventory + payment + policy checks succeed; if payment succeeded but conversion failed (hold expired and room sold), auto-void/refund and apology with alternatives — tested in `docs/09` G20. Low-bandwidth: every step works without client JS (form posts), budget §9.

### W2 — Check-in with ID OCR + registration signature + OTP (SCR-FO-check-in with SCR-SAF-id-intake, SCR-SAF-ocr-correction, SCR-FO-registration-card, SCR-SAF-otp-qr-verify)

```
┌─ SCR-FO-check-in · Res MS-7K4Q2P · Ms. A. Al-Harthy · 12–14 Nov ────────────────────────┐
│ Steps: ①Reservation ✓  ②Room ✓ 1208  ③Guarantee ✓  ④ID ●  ⑤Register ○  ⑥Verify ○  ⑦Keys ○ │
├───────────────────────────────────────────────────────────────────────────────────────────┤
│ ④ Identity document  (rule: OM guest registration v3, verified) ⓐ                         │
│ Document type ( ) Omani ID  (•) Passport  ( ) GCC ID  ( ) Other                            │
│ ┌──────── Scanner / camera ────────┐   Path: (•) OCR assist  ( ) Manual entry ⓑ            │
│ │  [ place document ]              │   [ ] Biometric match — not enabled for this property ⓒ│
│ └──────────────────────────────────┘                                                       │
│ OCR result (confidence)        Value              Guest confirmed                          │
│  Surname            0.98   [AL-HARTHY      ]      ✓                                        │
│  Given names        0.91   [AMAL           ]      ✓                                        │
│  Document no.       0.62 ⚠ [P1234S67       ]  ← corrected from P1234567 ⓓ                  │
│  Nationality        0.99   [OMN            ]      ✓                                        │
│  Expiry             0.97   [2031-04-30     ]      ✓                                        │
│ [Retake]  [Send to mismatch review]                          [Confirm ID]                  │
├───────────────────────────────────────────────────────────────────────────────────────────┤
│ ⑤ Registration card  (guest-facing tablet or guest phone via QR ⓔ)                        │
│  Fields per rule pack · policy text EN/AR · [x] I confirm the details are correct         │
│  Signature: [ Draw ]  or  [ Type full name ] ⓕ     Document hash: 9f2c…a1                 │
├───────────────────────────────────────────────────────────────────────────────────────────┤
│ ⑥ Verification  Send code via (•) SMS +968 •••• 4412  ( ) WhatsApp (approved template)     │
│  Code sent 12:03 · expires 12:08 · attempts left 3   [Resend in 0:45]                     │
│  Guest enters code on tablet/phone → ✓ Verified (bound to this session/device) ⓖ           │
│  Delivery failed? [Verify in person with staff] (reason required) ⓗ                         │
├───────────────────────────────────────────────────────────────────────────────────────────┤
│ Blocked reasons shown here if any: e.g. "Deposit authorisation pending" ⓘ                  │
│                                                 [Complete check-in] (needs server ack)     │
└───────────────────────────────────────────────────────────────────────────────────────────┘
```
ⓐ Required document types/fields come from the JUR rule version for the property; `unknown` → manual registration path with compliance flag. ⓑ Manual entry is always available (A-cam; lawful non-OCR alternative). ⓒ Biometric match appears only if lawful basis + consent + property activation gate (SF41.1.6). ⓓ Low-confidence fields are highlighted, corrections recorded as `guest_corrected`/staff-corrected (AU-E); ID image stored in restricted bucket with TTL and never used for analytics/AI training. ⓔ QR hand-off resumes the same session on the guest's phone; QR alone is not authentication (SF41.2.6). ⓕ Typed-name + intent checkbox is an equal alternative to drawing (A-sig). ⓖ OTP single-use, bound to session and device fingerprint; anti-replay. ⓗ Manual in-person verification requires reason and is audited. ⓘ Completion requires server ack of room assignment, guarantee, ID/registration rule evaluation — never offline (`docs/03` §8.2).

### W3 — Folio and checkout (SCR-FO-folio / SCR-FO-checkout)

```
┌─ SCR-FO-checkout · Room 1208 · Al-Harthy · Dep today 12:00 · Business date 14 Nov ───────┐
│ Windows: [1 Guest] [2 Company: Acme LLC]  [+ Route…]                                        │
│ ┌ Window 1 — Guest ─────────────────────────────────────────────────────────────────────┐   │
│ │ Date   BizDate  Code        Description            Amount     Src       ⋯              │   │
│ │ 12 Nov 12 Nov   ROOM        Deluxe King            37.000     night-aud                │   │
│ │ 12 Nov 12 Nov   VAT         VAT 5%                  1.850     rule v3                  │   │
│ │ 13 Nov 13 Nov   BAR         Lobby bar chk 5531     12.400     POS ▸ ⓐ                  │   │
│ │ 13 Nov 13 Nov   BAR-REV     Reversal of 5531 line 2 −3.200    ↩ link ⓑ                 │   │
│ │ 13 Nov 13 Nov   PARK        Parking 1 day           3.000     LPR session ▸            │   │
│ │ 14 Nov 14 Nov   MINIBAR     Count #MB-8812          4.500     HK count ▸ ⓒ             │   │
│ │ 12 Nov          PAYMENT     Deposit card ••4412   −40.000     PSP ref ▸                │   │
│ │                                          Balance     15.550 OMR                        │   │
│ └───────────────────────────────────────────────────────────────────────────────────────┘   │
│ [Post charge] [Adjust/Reverse…] ⓓ [Transfer to window…]                                     │
│ Settle window 1:  Tender ( ) Card on file ••4412  (•) Terminal T2  ( ) Cash  ( ) Points ⓔ   │
│ Invoice to:  (•) Guest  ( ) Company name + tax id <     >   Language [EN+AR ▾] ⓕ            │
│ Points to earn: 84 pts (pending until stay complete) ⓖ                                      │
│ [Settle 15.550 OMR]   → then [Check out]   {EL-M}                                           │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```
ⓐ Every entry links to its source (POS check, LPR session, HK count) — one posting per source key. ⓑ Corrections are reversal entries with link; no edit/delete. ⓒ Minibar posted once by `count_id`; late counts after checkout go to late-charge review. ⓓ Adjust over limit requires maker-checker. ⓔ Points tender uses hold→capture so failures do not lose points. ⓕ Invoice fields/language/fiscal number follow JUR invoice rules. ⓖ Points pending until the stay is complete and payment cleared; refund reverses points exactly once.

### W4 — Corporate event search / compare / hold (SCR-CORP-event-search → SCR-CORP-event-compare → SCR-CORP-hold-detail)

```
┌─ SCR-CORP-event-search · Acme LLC (agreement v4, valid to 31 Dec 2026) ──────────────────┐
│ Dates:  [18 Nov] [19 Nov] [+ add alternative date]   Attendees <80>   Layout [Classroom ▾] │
│ Rooms   <10> nights <1>   Meals [x] Lunch  [ ] Dinner   Hosted bar [x] 2h                  │
│ Club visit [x] 20:00–22:00   Parking passes <20>   AV [x] Projector [x] 2 mics             │
│                                                   [Find feasible options]                  │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│ 3 feasible options at your contracted rates ⓐ     (2 others hidden: not feasible ▸ why) ⓑ   │
└────────────────────────────────────────────────────────────────────────────────────────────┘
        ▼
┌─ SCR-CORP-event-compare ─────────────────────────────────────────────────────────────────┐
│                     Option A          Option B          Option C                           │
│ Date                18 Nov            18 Nov            19 Nov                             │
│ Space               Ballroom A        Ballroom A+B      Majlis Hall                        │
│ Classroom capacity  90                150               84                                 │
│ Rooms (10×Deluxe)   ✓ contracted 45   ✓ 45              ✓ 45                               │
│ Lunch/bar/club      ✓                 ✓                 ✓ club at 21:00 (20:00 full) ⓒ     │
│ Parking 20          ✓ Zone B          ✓ Zone B          ✓ Zone A                           │
│ Total incl. tax     OMR 4,860.000     OMR 5,420.000     OMR 4,790.000                      │
│ Deposit             30%               30%               30%                                │
│                     [Hold A]          [Hold B]          [Hold C]                           │
└────────────────────────────────────────────────────────────────────────────────────────────┘
        ▼  composite hold: rooms + space components + capacity slots in ONE transaction ⓓ
┌─ SCR-CORP-hold-detail · HOLD-2231 ─────────────── Expires 20 Nov 17:00 (47:59:12) ⓔ ──────┐
│ Components: Ballroom A 18 Nov 07:00–18:00 (incl. setup/teardown) · 10 rooms · lunch 80 ·   │
│ bar 2h · club 20 · parking 20 Zone B · AV                                                  │
│ Approval: required (policy: > OMR 3,000) → Approver: J. Smith  [Request approval] ⓕ       │
│ Payment: ( ) Deposit by card  (•) PO number <PO-7781>  ( ) Credit account                  │
│ [Request extension]                               [Confirm booking] (after approval)       │
└────────────────────────────────────────────────────────────────────────────────────────────┘
```
ⓐ Only configurations feasible at contracted rates appear (G.1). ⓑ "Why not" explains non-feasible options (capacity, space conflict) without revealing other clients. ⓒ Partial feasibility shows the nearest feasible alternative explicitly. ⓓ Exclusion constraints and capacity slots guarantee no double sale (`docs/03` §5.2). ⓔ Countdown with notifications at 24 h/2 h; expiry releases all components atomically. ⓕ Corporate approval routing per cost center; confirmation needs server ack and deposit/PO/credit check.

### W5 — Parking low-confidence review (SCR-PRK-lane-review)

```
┌─ SCR-PRK-lane-review · Lane 2 ENTRY · 21:14:07 ── queue: 2 waiting ─── Gate: CLOSED ──────┐
│ ┌── snapshot (plate crop, privacy-limited) ──┐  Read:  "8 4 2 1 7 ?  A B"  conf 0.61 ⓐ   │
│ │  [ plate image ]                            │  Region: OM (guess 0.8)                    │
│ └─────────────────────────────────────────────┘                                            │
│ Candidates (fuzzy match on active permits):                                                │
│   (•) 84217 AB  · Guest permit · Room 1208 · valid to 14 Nov 12:00  ⓑ                     │
│   ( ) 84211 AB  · Staff permit · valid                                                     │
│   ( ) None of these → type plate <        >                                                │
│ Decision:  [Open gate (O)]   [Deny (D)]   [Escalate to security (E)] ⓒ                      │
│ Reason (required for override/none) <▾ select>                                             │
│ Auto-deny timer 60 s → then route to intercom ⓓ   [Pause timer]                            │
└────────────────────────────────────────────────────────────────────────────────────────────┘
```
ⓐ Below threshold (configurable, default 0.85) always goes to a human; AI/LPR never opens on low confidence. ⓑ Candidate shows minimum personal data; selecting corrects the plate on the observation (audit keeps original read). ⓒ Keyboard shortcuts for speed; each decision audited with reason (AU-W); repeated overrides alert supervisor. ⓓ Timer adjustable (A-time); network loss → site gateway local decision for cached permits only; manual gate screen as fallback.

### W6 — RFQ comparison and award (SCR-PRC-rfq-comparison → SCR-PRC-award)

```
┌─ SCR-PRC-rfq-comparison · RFQ-0912 · 120 KG tomatoes (fresh, grade A) · need 20 Nov 06:00 ─┐
│ Quotes: 3 of min 3 ✓ ⓐ   Weights v2 (locked at opening 17 Nov 10:00) ⓑ                      │
│ Normalised to OMR per KG landed (incl. freight, tax) ⓒ                                      │
│ Vendor        Unit/KG  Landed  Qual/Fresh  Avail  Lead  OTIF(n)    Defects  FoodSafe  Score │
│ Green Farms   0.420    0.451   4.5 [📷3]   150kg  12h   96%(41)    1.2%     ✓ valid   86.4 │
│ Oasis Fresh   0.395    0.447   3.8 [📷2]   120kg  18h   81%(9) low-n 3.9%   ✓ valid   78.9 │
│ Sun Produce   0.380    0.462*  4.0 [📷3]   200kg  24h   — (0) no history ⓓ   ✓        74.2 │
│ * freight added: vendor quoted ex-works                                                    │
│ Disqualified: none   [View samples side-by-side ▸ SCR-PRC-sample-gallery]                   │
│ AI summary (assistive, not a decision) ⓔ: "Oasis cheapest landed by 0.004; Green Farms     │
│   higher OTIF with larger sample…" [sources]                                               │
│ Recommendation: Green Farms (highest score)          [Proceed to award]                    │
└────────────────────────────────────────────────────────────────────────────────────────────┘
        ▼
┌─ SCR-PRC-award ────────────────────────────────────────────────────────────────────────────┐
│ Selected: ( ) Recommended Green Farms  (•) Other: Oasis Fresh  → Override reason required ⓕ│
│ Reason <▾ price priority for banquet> <details…>                                            │
│ Split award? [ ] 80 KG Green / 40 KG Oasis                                                 │
│ Budget: F&B Kitchen CC-410 · encumber OMR 53.640 · remaining 1,204.000                      │
│ Conflict declarations: evaluators ✓ none   SoD: requester ≠ approver ✓                      │
│ [Sign award] (step-up) → PO v1 generated → vendor acknowledgment required ⓖ                 │
└────────────────────────────────────────────────────────────────────────────────────────────┘
```
ⓐ If fewer than minimum eligible responsive quotes: banner + waiver path (SCR-PRC-rfq-waiver), award blocked. ⓑ Weight version locked at opening; changing weights after opening is impossible (SF49.2.3). ⓒ Normalisation to canonical UOM and landed cost prevents apples-to-oranges. ⓓ History shown with sample size/confidence; no-history is neutral per policy, not penalised silently. ⓔ AI summary labelled, cites evidence, cannot change weights or award (SF49.2.6). ⓕ Override requires reason + higher approver where policy says; recorded (SF49.2.7). ⓖ Not-awarded vendors receive limited notice; samples retained until `retention_until` (90 days).

### W7 — Receiving with quarantine (SCR-PRC-receiving → SCR-PRC-receiving-verify → SCR-PRC-quarantine)

```
┌─ SCR-PRC-receiving · Dock 1 · PO-5530 v2 / ASN GF-778 · Green Farms · arrived 05:42 ───────┐
│ Scan: [📷 PO/ASN QR] ✓   Vehicle temp log ✓ 3.9 °C  ⓐ                                      │
│ Line  Item              Expected   Scanned/Weighed   Lot     Expiry   Temp   Result        │
│ 1     Tomatoes fresh A  80 KG      78.6 KG (scale S1) L-1102 22 Nov   6.8°C  ⚠ temp > 5°C ⓑ│
│ 2     Tomatoes fresh A  40 KG      40.2 KG            L-1103 22 Nov   4.1°C  ✓ within tol  │
│ 3     Basil 100 g pk    10 pk      8 pk (barcode)     L-88   19 Nov   —      ⚠ short 2     │
│ Packing slip OCR: matches ASN ✓ (raw image kept) ⓒ                                          │
│ Risk class: FOOD (high) → physical verification required ⓓ                                 │
│ [Verify lines ▸]      Draft GRN (not posted)     Offline: queued draft ● ⓔ                  │
└────────────────────────────────────────────────────────────────────────────────────────────┘
        ▼
┌─ SCR-PRC-receiving-verify · Line 1 ──────────────────────────────────────────────────────┐
│ Checks: Temperature 6.8 °C (limit 5 °C) ✗   Condition ✓   Packaging ✓   Photo [📷 2]       │
│ Decision: ( ) Accept  (•) Quarantine  ( ) Reject/return now                               │
│ Accountable receiver: R. Das (attest) [Sign]                                              │
└────────────────────────────────────────────────────────────────────────────────────────────┘
        ▼ [Post GRN] (server) → line 2 accepted 40.2 KG → STOCK_LEDGER receipt (available) ×1 ⓕ
                               line 1 78.6 KG → STOCK_LEDGER receipt (quarantine) ×1
                               line 3 8 pk accepted; short 2 → vendor claim
┌─ SCR-PRC-quarantine ─────────────────────────────────────────────────────────────────────┐
│ L-1102 Tomatoes 78.6 KG · reason temp breach · photos · vendor notified                   │
│ Disposition: ( ) Return to vendor → credit memo  ( ) Destroy (waste)  [Concession ✗ not    │
│ allowed for food-safety failure] ⓖ           [Submit for approval]                         │
└────────────────────────────────────────────────────────────────────────────────────────────┘
```
ⓐ Cold-chain evidence captured from ASN/vehicle where supplied. ⓑ Tolerance and temperature rules from PO spec + JUR food rule pack. ⓒ Raw evidence immutable (AU-E). ⓓ Low-risk items may be straight-through with attestation; food/high-value/discrepancy require physical verification (SF50.2.5). ⓔ Draft can be captured offline; **posting requires server** and is idempotent — duplicate scans create one movement. ⓕ Accepted quantity creates exactly one stock movement and one payable match input. ⓖ Quarantined/waste stock never becomes available (`docs/03` §5.3).

### W8 — Chef callout (SCR-FNB-chef-callout)

```
┌─ SCR-FNB-chef-callout · Dinner service · Main kitchen · 14 Nov 17:00–23:00 ── UNCOVERED ──┐
│ Primary: Chef K. Rao — sick call 13:10 ✗     Backup: Chef M. Ali — no answer (2 tries) ✗ ⓐ │
│ Cutoff to cover: 15:30 (1:48:22 left)                                                     │
│ Callout sequence (eligible only ⓑ)                         Channel     Sent   Reply  Due    │
│ 1 S. Nair (emergency roster) 4.8 km · 25 min · cert ✓     push+SMS    13:42  —      13:57  │
│ 2 FreshChef Staffing (vendor) · 40 min · contract ✓       push+WA     13:42  —      13:57  │
│ 3 J. Costa (off-duty staff) · OT rules ✓                   SMS+voice  queued               │
│ Live: 13:49 S. Nair ACCEPTED ✓ (first valid acceptance; others notified "filled") ⓒ         │
│ Assignment: needs manager approval (paid external)   [Approve assignment] (F&B mgr) ⓓ      │
│ Handover pack: BEO #EV-311 (80 covers, 6 nut-free, 2 coeliac), menu, allergen matrix       │
│  → shared until 23:30, access logged [Share] ⓔ                                             │
│ If no acceptance by 15:00 → escalate GM + contingency menu approval ⓕ                      │
└────────────────────────────────────────────────────────────────────────────────────────────┘
```
ⓐ Primary/backup statuses from roster/attendance; no-show thresholds configurable. ⓑ Only candidates with valid food-safety credentials, availability and no overlapping assignment are contacted (SF47.1.6). ⓒ Atomic accept (unique constraint): simultaneous acceptances resolve to one; others see "filled by another" (G.15). ⓓ Paid external chef requires manager approval (SF47.2.4). ⓔ BEO/allergen access limited to assignment window (SF47.2.5). ⓕ Escalation and contingency when nobody accepts (SF47.2.6); all attempts logged for response KPI.

### W9 — Utility bill inbox (SCR-FIN-utility-bill-inbox → SCR-FIN-utility-bill-detail → SCR-FIN-bill-payment-detail)

```
┌─ SCR-FIN-utility-bill-inbox ─────────── Filters: [Electricity][Water][Gas] Due ≤ 14 d ▾ ──┐
│ Provider         Account   Period     Billed kWh/m³  Metered   Var %  Amount     Due   St.  │
│ Nama Electricity ••2291    Oct 2026   182,400 kWh    176,950   +3.1   OMR 4,120  02 Dec VAL│
│ Nama Water       ••1180    Oct 2026   5,210 m³       3,960     +31.6⚠ OMR 1,030  28 Nov REV│ⓐ
│ City Gas Co.     ••7712    Oct 2026   —              —(no meter) n/a  OMR   640  05 Dec CAP│ⓑ
│ (duplicate suspected) Nama Water ••1180 Oct 2026 · same no. as above  [Review] ⓒ          │
└────────────────────────────────────────────────────────────────────────────────────────────┘
        ▼
┌─ SCR-FIN-utility-bill-detail · Nama Water ••1180 · Oct 2026 ─────────────────────────────┐
│ Lines: supply 3,870 · wastewater 1,000 · standing 160   Tariff v2026-07 (source ▸)          │
│ Meter reconciliation: master WM-01 3,960 m³ (actual) · gap 03–05 Oct (estimated) ⓓ          │
│ Variance +31.6 % > 10 % tolerance → [Open leak work order ▸ ENG] [Dispute with provider]   │
│ Allocation preview (driver: submeters v3): Rooms 62% · Laundry 24% · Kitchen 11% · Pool 3% │
│ Status: variance_review  → [Approve to AP] (after resolution, finance_approver)             │
└────────────────────────────────────────────────────────────────────────────────────────────┘
        ▼ payable approved → pay
┌─ SCR-FIN-bill-payment-detail ────────────────────────────────────────────────────────────┐
│ Route: (•) Bank transfer (manual evidence)  ( ) Khedmah adapter — BLOCKED: no contract ⓔ  │
│ Amount OMR 1,030.000 · due 28 Nov · idempotency billpay:PAY-8812:1                         │
│ Status: submitted → pending … [Check status] (inquiry) ⓕ                                    │
│ Evidence: bank ref <     > [Upload receipt]      Marked paid only with evidence            │
└────────────────────────────────────────────────────────────────────────────────────────────┘
```
ⓐ Variance beyond tolerance forces review. ⓑ No meter → bill-only evidence labelled `estimate` in allocation. ⓒ Duplicate detection on provider+account+period+number (unique dup_key). ⓓ Missed intervals shown as gaps/estimated (SF22.1.4). ⓔ Adapter honesty label shown; blocked capabilities are unavailable, manual path offered (§E). ⓕ Timeout → pending until inquiry/reconciliation proves status; no blind retry (cross-utility guard).

### W10 — GM flash drill-down (SCR-GM-flash → SCR-GM-flash-drilldown)

```
┌─ SCR-GM-flash · Business date 14 Nov 2026 · CLOSED 02:41 ── vs Budget ▾ vs LY ▾ ─────────┐
│ Occupancy 78.4% (+2.1 vs bud) reconciled   ADR OMR 71.20   RevPAR 55.82   TRevPAR 92.10     │
│ GOP month-to-date OMR 212,400  ⚠ Incomplete estimate: gas invoice missing, laundry accrual ⓐ│
│ Dept contribution: Rooms 68% · F&B 21% · Club 12% · Parking 71% · Catering 18%              │
│ [Table view] ⓑ                                                                               │
└────────────────────────────────────────────────────────────────────────────────────────────┘
        ▼ click GOP
┌─ SCR-GM-flash-drilldown · GOP MTD ───────────────────────────────────────────────────────┐
│ Definition: GOP v3 (KPI dictionary ▸) · allocation v5 · as of 02:41                        │
│ Revenue 612,300 ▸  − Dept costs 301,900 ▸  − Undistributed 98,000 ▸  = GOP 212,400          │
│   Undistributed ▸ Utilities 31,200 (electricity metered ✓, water ✓, gas ESTIMATE ⚠) ▸     │
│      Gas ▸ accrual 2,100 basis: last 3 bills avg · awaiting invoice from City Gas Co.       │
│          ▸ source: accrual journal J-2026-11-0412 → evidence: allocation rule v5          │
│   Dept costs ▸ Labor 142,800 (aggregate by dept — individual salaries not available) ⓒ    │
│   Channel fees ▸ 18,450 → by channel → booking ▸ commission line ▸ channel statement ⓓ     │
│ [Export] [Open source ledger ▸]                                                             │
└────────────────────────────────────────────────────────────────────────────────────────────┘
```
ⓐ Any missing or estimated source flags the tile **Incomplete estimate**, never certified (G.8). ⓑ Every chart has a table view (A-chart). ⓒ Salary confidentiality: GM sees department aggregates only (F27.4). ⓓ Drill path ends at ledger entries and evidence (SF32.3.5, M65 lineage).

---

## 6. Bilingual and RTL rules

### 6.1 Rules
1. **Direction:** `dir` set on `<html>` from locale; all layout uses logical properties (`margin-inline-start`, `inset-inline-end`, `text-align: start`); flex/grid order follows direction automatically. No hard-coded left/right in components.
2. **Mirroring:** mirror directional icons (back/forward, chevrons, progress arrows, send), sliders and steppers; **do not mirror** media controls' play icon, clocks, checkmarks, logos, charts' time axes (time flows left→right in both; D-971 to confirm with Arabic-speaking users), phone keypads, barcode/QR.
3. **Bidi isolation:** confirmation numbers, room numbers, plates, phone numbers, IBAN, emails, SKUs and codes wrapped in `<bdi>`/`dir="ltr"` spans; mixed-script names isolated. Currency amounts keep their internal order.
4. **Numerals:** Latin digits default in Arabic UI with a per-user option for Arabic-Indic (D-937 in `docs/03`); identifiers always Latin digits.
5. **Fonts:** Arabic-capable UI font with matching weights and line height ≥ 1.5 for Arabic; no letter-spacing on Arabic text; no italic for Arabic emphasis (use weight).
6. **Text expansion/contraction:** allow +35 % EN→AR width variance; no fixed-width buttons; truncation only with tooltip/full text on focus.
7. **Forms:** labels above fields (avoids alignment issues); input direction auto for free text; email/URL/phone fields `dir="ltr"`; validation messages localised via ICU.
8. **Dates/calendars:** Gregorian primary; optional Hijri display (ADR-019); date pickers accept typed dates; week start per locale.
9. **Documents:** invoices, registration cards, BEOs and payslips render bilingual when the JUR rule or property setting requires; PDF generation supports Arabic shaping and RTL tables.
10. **Content:** each translatable content field has its own approval status; untranslated content falls back to the default language with a visible language tag (`lang` attribute set on the fallback fragment).
11. **Search:** Arabic normalisation (alef variants, ta marbuta, hamza, diacritics) and transliteration-tolerant guest name search.
12. **Messages:** SMS/WhatsApp templates exist per language; WhatsApp templates require BSP approval per language before use.

### 6.2 Bilingual/RTL acceptance checklist (per screen)
- [ ] All user-visible strings come from ICU catalogs; no concatenation; plurals correct for Arabic (zero/one/two/few/many/other).
- [ ] Screen renders in `ar` with correct `dir="rtl"`, logical layout, mirrored directional icons only.
- [ ] Identifiers, amounts, plates, phones, emails display correctly in RTL (bidi-isolated).
- [ ] No truncation/overlap at 200 % zoom in both languages; pseudo-locale (+40 %) passes.
- [ ] Numbers/currency/date formats follow CLDR for `en` and `ar`; OMR shows 3 decimals.
- [ ] Hijri display (if enabled) shows alongside Gregorian and never replaces stored/legal date.
- [ ] Screen reader reads Arabic content with correct language (`lang="ar"`), including mixed fragments.
- [ ] Generated documents (PDF/email/SMS) verified in both languages.
- [ ] Keyboard focus order follows visual order in RTL.
- [ ] Charts: axes/labels readable in RTL; table alternative localised.

---

## 7. Cross-screen interaction rules

1. **One primary action per screen/card**; secondary actions in an overflow menu with the same labels everywhere (e.g. "Reverse", never "Delete", for ledger entries).
2. **Irreversible or money-moving actions** use a review-and-confirm step (WCAG 3.3.4) that restates amount, currency, target and consequence; the confirm button repeats the amount ("Pay OMR 84.000").
3. **Maker-checker** flows show both principals and forbid self-approval in the UI and server; the checker sees the maker's evidence and reason.
4. **Step-up** prompts appear only at the moment of the privileged action and preserve all entered data.
5. **Bulk actions** (assign rooms, approve invoices, publish media) show a preview count and per-item result list; partial failures are listed, never hidden.
6. **Undo** is offered only for reversible non-ledger UI actions (e.g. dismiss a task within 10 s); ledger corrections are always explicit reversals.
7. **Keyboard shortcuts** exist for high-volume desks (front desk, POS, lane review, receiving) and are discoverable (`?`), remappable and disabled by default for single-key shortcuts outside focused panels (WCAG 2.1.4).
8. **Printing/export**: every list and document supports print-friendly view and CSV/XLSX/PDF export subject to field security; exports with personal data are logged (AU-R).
9. **Freshness**: every screen that shows externally sourced or projected data shows "as of" time; stale data never silently looks current.
10. **Scope indicator**: the active property/department and business date are always visible in the staff shell header.

---

## 8. WCAG 2.2 AA acceptance checklist

Applies to every web and native screen (native mapped to platform accessibility APIs). A screen is accepted only when all applicable items pass automated (axe-core / React Native accessibility lint) **and** manual checks (keyboard, NVDA+Firefox/Chrome, VoiceOver iOS/macOS, TalkBack Android) in EN and AR.

| # | Criterion (WCAG 2.2) | Acceptance check |
|---|---|---|
| 1 | 1.1.1 Non-text content | all images have EN/AR alt text or are marked decorative; property media alt text required before publish (SCR-MED-approval) |
| 2 | 1.2.2/1.2.3/1.2.5 Captions, audio description | published videos have captions EN/AR and audio description or text alternative |
| 3 | 1.3.1 Info and relationships | headings hierarchy, landmarks, tables with headers, form groups with legends |
| 4 | 1.3.2 Meaningful sequence | DOM order = visual order in LTR and RTL |
| 5 | 1.3.4 Orientation | no orientation lock except kiosk/POS devices (documented exception) |
| 6 | 1.3.5 Identify input purpose | `autocomplete` tokens on personal data fields |
| 7 | 1.4.1 Use of colour | status never colour-only (icons + text) — room board, coverage dashboards, charts |
| 8 | 1.4.3 / 1.4.11 Contrast | text ≥ 4.5:1 (≥ 3:1 large), UI components/graphics ≥ 3:1 in light and dark themes (tokens §10 validated) |
| 9 | 1.4.4 / 1.4.10 Resize and reflow | 200 % zoom without loss; reflow at 320 CSS px without horizontal scroll (except data grids with documented 2-D scroll) |
| 10 | 1.4.12 Text spacing | no clipping with increased line/letter/word spacing |
| 11 | 1.4.13 Content on hover/focus | tooltips dismissible, hoverable, persistent |
| 12 | 2.1.1 / 2.1.2 Keyboard, no trap | all functions keyboard operable incl. room rack, function diary, roster (A-drag alternatives); modals return focus |
| 13 | 2.1.4 Character key shortcuts | single-key shortcuts (PRK lane review) can be turned off/remapped and are only active when focus is in the panel |
| 14 | 2.2.1 Timing adjustable | holds/OTP/quotes/timers warn and allow extension where business rules allow; else explain and preserve data |
| 15 | 2.2.2 Pause, stop, hide | auto-updating boards can be paused; carousels paused by default |
| 16 | 2.3.1 Flashes | no flashing > 3/s (alarm UIs use non-flashing emphasis) |
| 17 | 2.4.1 Bypass blocks | skip links; landmarks |
| 18 | 2.4.2 / 2.4.6 Titles, headings, labels | unique page titles incl. record ids; descriptive labels |
| 19 | 2.4.3 Focus order | logical; wizard steps move focus to step heading |
| 20 | 2.4.7 / 2.4.11 Focus visible, not obscured | focus ring token ≥ 3:1; sticky headers/footers never cover focused element |
| 21 | 2.5.3 Label in name | visible label included in accessible name |
| 22 | 2.5.7 Dragging movements | every drag has a single-pointer alternative |
| 23 | 2.5.8 Target size (minimum) | ≥ 24×24 CSS px; touch apps ≥ 44×44 pt |
| 24 | 3.1.1 / 3.1.2 Language | `lang` on page and on mixed-language parts |
| 25 | 3.2.1/3.2.2 On focus/input | no context change on focus/selection without warning |
| 26 | 3.2.6 Consistent help | help/assisted contact in same place on all guest steps |
| 27 | 3.3.1/3.3.3 Error identification/suggestion | errors in text, linked to field, with fix suggestion; summary receives focus |
| 28 | 3.3.4 Error prevention (legal, financial, data) | review-confirm step and/or reversal for payments, bookings, signatures, awards, releases |
| 29 | 3.3.7 Redundant entry | data already entered in the flow is prefilled/selectable |
| 30 | 3.3.8 Accessible authentication (minimum) | no cognitive function test; passkeys, OTP autofill, paste allowed; no puzzle CAPTCHA |
| 31 | 4.1.2 Name, role, value | custom components expose correct roles/states (grid, tabs, combobox, switch) |
| 32 | 4.1.3 Status messages | aria-live for async results (payment status, sync, hold timers, KDS/incident alerts) |
| 33 | Native parity | Dynamic Type/font scale to 200 %, TalkBack/VoiceOver labels, reduce motion honoured |
| 34 | Documents | generated PDFs tagged (invoices, registration card, BEO, payslip) with reading order and language |

Evidence: automated reports stored per build; manual test scripts in `tests/a11y` (`docs/03` §16); pilot includes users of screen readers in EN and AR (D-972).

---

## 9. Low-bandwidth guest budget (booking website and guest web flows)

Target network: "Slow 4G / 3G" profile (≈ 400 kbps down, 400 ms RTT) and mid-range Android device.

| Metric (per page, p75 field + lab) | Budget |
|---|---|
| HTML (compressed) first response | ≤ 50 KB |
| Critical CSS inline | ≤ 14 KB; total CSS ≤ 40 KB |
| JavaScript (compressed) on search/results/details | ≤ 70 KB initial; payment step ≤ 120 KB excluding PSP-hosted component |
| Web fonts | ≤ 2 families, subsetted (Latin + Arabic), `font-display: swap`, ≤ 80 KB total |
| Images above the fold | ≤ 150 KB total; responsive AVIF/WebP with width descriptors; lazy-load below fold |
| Total page weight (first view) | ≤ 500 KB home/results; ≤ 350 KB checkout steps |
| Requests (first view) | ≤ 25 |
| LCP | ≤ 2.5 s (p75, slow 4G lab ≤ 4 s) |
| INP | ≤ 200 ms |
| CLS | ≤ 0.1 |
| TTFB (server) | ≤ 600 ms p75 |
| Availability API (server) | p95 ≤ 500 ms |

Rules: SSR HTML works without JS for search → results → select → details → review (form posts; PSP step may require PSP script — offer pay-at-hotel/pay-link alternatives where rate allows); no third-party tags without consent; analytics loaded after consent and ≤ 10 KB; video never autoplays on mobile data and uses poster + HLS on demand; "Lite mode" toggle removes galleries; offline guest app shows cached confirmation/pass. Budgets enforced in CI (Lighthouse CI/bundle size check) and in `docs/09` performance tests.

---

## 10. Design-system tokens

Tokens live in `packages/ui-tokens` (`docs/03` §16) as W3C Design Tokens JSON → CSS custom properties and RN theme. Brand palette is a placeholder until MetriStay brand assets are approved (D-973); semantic tokens are the contract used by components.

### 10.1 Foundations

| Group | Tokens (name → value, light / dark) |
|---|---|
| Spacing (4-pt) | `space.0`=0, `.1`=4, `.2`=8, `.3`=12, `.4`=16, `.5`=24, `.6`=32, `.7`=48, `.8`=64 |
| Radius | `radius.sm`=4, `.md`=8, `.lg`=12, `.pill`=999 |
| Elevation | `elev.0` none, `.1` 0 1 2 rgba(0,0,0,.12), `.2` 0 4 12 rgba(0,0,0,.16) (dark: borders instead of shadows) |
| Typography | `font.family.latin` = Inter-class sans (placeholder), `font.family.arabic` = Noto Sans Arabic / IBM Plex Sans Arabic class (licence check D-974), `font.family.mono` = mono for codes; sizes `12/14/16/18/20/24/30/36`; base 16 (web) / 17 (iOS) ; line-height 1.5 (Latin) / 1.6 (Arabic); weights 400/500/600/700 |
| Breakpoints | `bp.sm` 360, `bp.md` 768, `bp.lg` 1024, `bp.xl` 1440 |
| Touch target | `target.min` 44 (touch), 24 (pointer min) |
| Motion | `motion.fast` 120 ms, `motion.base` 200 ms, `motion.slow` 320 ms; `prefers-reduced-motion` → 0 ms, no parallax |
| Z-index | `z.base` 0, `z.sticky` 100, `z.drawer` 200, `z.modal` 300, `z.toast` 400 |

### 10.2 Semantic colour tokens (contrast validated ≥ 4.5:1 text / ≥ 3:1 UI in both themes)

| Token | Purpose | Light | Dark |
|---|---|---|---|
| `color.bg.canvas` | page background | #FFFFFF | #0F1115 |
| `color.bg.surface` | cards/panels | #F6F7F9 | #171A21 |
| `color.bg.raised` | menus/modals | #FFFFFF | #1E222B |
| `color.text.primary` | body text | #14171F | #E9ECF2 |
| `color.text.secondary` | secondary | #4A5160 | #B3BAC7 |
| `color.border.default` | borders/dividers (≥ 3:1 where it conveys boundary) | #8A92A3 | #5B6475 |
| `color.action.primary` | primary buttons/links | #0B5CAD | #6FB1FF |
| `color.action.primary.text` | text on primary | #FFFFFF | #0B1320 |
| `color.focus.ring` | focus indicator | #0B5CAD 2px + 2px offset | #9CCBFF |
| `color.status.success` | success/ok | #1B7F3B | #5BD08A |
| `color.status.warning` | warning (always with icon) | #8A5A00 | #F2C14E |
| `color.status.danger` | error/critical | #B3261E | #FF8A80 |
| `color.status.info` | info | #0B5CAD | #6FB1FF |
| `color.honesty.estimate` | "estimate" badge | #6B4FA3 | #C3A8FF |
| `color.honesty.simulator` | "SIMULATOR" badge | #9C27B0 | #E1A3F0 |
| `color.honesty.blocked` | partner blocked | #5F6368 | #A0A6AD |
| `color.room.dirty` / `.clean` / `.inspected` / `.ooo` | HK states (paired with icons + text) | #B3261E / #0B5CAD / #1B7F3B / #3C4043 | dark equivalents |
| `color.data.1..8` | categorical chart palette (colour-blind safe, validated order) | placeholder set | placeholder set |

### 10.3 Component contracts (selected)
Button (primary/secondary/tertiary/danger; loading state disables and announces), TextField (label, hint, error, `dir=auto`), MoneyInput (currency-aware decimals), DateRangePicker (typed input + calendar, Hijri secondary display), DataGrid (A-grid, virtualised, column pinning, RTL), StatusBadge (icon + text + colour), HonestyBadge (§2.8), CountdownTimer (A-time, extend), Stepper/Wizard (focus management), Drawer/BottomSheet, Toast (aria-live polite; critical alerts use modal banner assertive), EmptyState, ErrorBanner (correlation id copy), OfflineBanner/QueueChip, SignaturePad (typed alternative), CameraCapture (manual alternative), ScanInput (keyboard-wedge + camera), KPI Tile (source, as-of, estimate label, table view), Chart wrapper (table + summary).

---

## 11. App distribution plan (corporate, vendor, staff, guest)

### 11.1 Distribution matrix

| App | Audience | Channel (Release 1) | Signing / identity | Update policy | Dependencies |
|---|---|---|---|---|---|
| Guest app (mobile-guest) | public guests | **Public stores**: Apple App Store, Google Play; web remains fully functional (app optional) | Apple Distribution certificate under MetriSys developer account; Google Play App Signing (upload key held in Vault) | store releases; OTA JS updates only for non-native, non-payment/ID changes and within store policy | Apple Developer Program (Organization) + D-U-N-S number; Google Play Console (organization) account; app name/trademark clearance "MetriStay" (§E brand note); privacy labels/data safety forms; per-hotel white-label decision D-975 |
| Corporate app (mobile-corporate, "MetriStay Business") | corporate customers' employees | **Public stores** (unlisted/closed-testing option where supported) **plus** enterprise-friendly delivery: Apple Business Manager Custom App for specific corporates; Managed Google Play private app for corporates using MDM | same developer accounts; per-corporate distribution via ABM/Managed Play when requested | store releases; min supported version gate | developer accounts; corporate MDM cooperation; §F: "installable signed builds with parity for the core journey; public store publication depends on developer accounts and approval" |
| Vendor app (mobile-vendor) | verified suppliers (external businesses) | **Public stores** (listed, login-gated) — vendors do not have hotel MDM; fallback: vendor web (VEN) always available | same developer accounts | store releases; forced update for security fixes | store review of login-gated apps requires demo accounts (simulator tenant); vendor devices unmanaged → app-level security (MFA, SQLCipher, jailbreak/root signals as risk flag, not block) |
| Staff app (mobile-staff) | hotel employees | **Private**: Apple Business Manager Custom App / Apple Developer Enterprise Program only if eligible (D-976), Managed Google Play private app; for hotels without MDM: store-listed but realm-restricted build with device enrolment (SCR-OPS-device-enrollment) | device realm certificates (mTLS) + staff SSO | MDM-pushed updates; min version gate | hotel MDM (Intune/Jamf/other) or lightweight enrolment; shared-device mode for housekeeping; D-977 which MDM for pilot |
| POS/KDS/kiosk shell | outlets, kitchens, lanes | device-provisioned (Android dedicated devices / managed tablets / web kiosk mode) | device identity certs | managed updates in maintenance windows | hardware selection per pilot (docs/05, docs/09) |

### 11.2 Build and release pipeline
1. EAS Build (or equivalent CI runners with macOS for iOS) produces signed builds; signing keys/certificates stored in Vault and CI secret store; no keys on developer laptops.
2. Tracks: `internal` (QA, simulator tenant) → `pilot` (TestFlight external testing / Play closed testing; pilot hotel and selected vendors/corporates) → `production` (phased rollout 10 % → 50 % → 100 % with crash/error gates).
3. Every build carries version, build number, git SHA and SBOM; release notes EN/AR.
4. Store compliance: privacy nutrition labels / data safety forms per app; account deletion in-app (store requirement) mapped to DSR (SCR-GST-data-request); login-gated review accounts use the simulator tenant with fictitious data only.
5. Kill switch & min version: server-side `min_supported_version` per app (SCR-OPS-app-update-required); remote config to disable a feature (e.g. referral in a market) without a release.
6. Certificate/profile expiry monitored (SCR-ADM-secrets-certs), 30-day alert.

### 11.3 External dependencies (release blockers if missing)

| Dependency | Needed for | Owner | Status label |
|---|---|---|---|
| Apple Developer Program (Organization) account for Metrikingdom/MetriSys (D-U-N-S) | guest, corporate, vendor apps on iOS; ABM custom apps | Metrikingdom admin | `unverified-assumption` |
| Google Play Console organization account | Android apps; Managed Google Play private apps | Metrikingdom admin | `unverified-assumption` |
| Apple Developer Enterprise Program eligibility (optional) | in-house staff app distribution | Metrikingdom admin + counsel | `unverified-assumption` (D-976) |
| Trademark/app-name clearance for "MetriStay", "MetriStay Business", "MetriStay Rewards", "MetriStay Partner Hub" | store listing names | Legal | `unverified-assumption` |
| Pilot hotel MDM or enrolment approach | staff app | Pilot hotel IT | D-977 |
| WhatsApp Business (BSP) account and template approvals EN/AR | OTP, notifications | Product/partner | see `docs/05` |
| Push notification credentials (APNs key, FCM) | all apps | Platform | `unverified-assumption` |

---

## 12. Screen index, counts and traceability

### 12.1 Counts by application

| App | Screens | App | Screens |
|---|---|---|---|
| OPS | 22 | PRC | 36 |
| GM | 37 | GST | 43 |
| FO | 52 | AI | 11 |
| FIN | 51 | CORP | 26 |
| HR | 32 | CAPP | 15 |
| ENG | 25 | PRK | 13 |
| VEN | 29 | ADM | 52 |
| VAPP | 23 | MED | 23 |
| CON | 19 | SAF | 27 |
| FNB | 42 | **Total** | **578** |

*(Counts are generated from the tables above; the CI doc-lint in Phase 2 recounts and fails on duplicate or undefined ids.)*

### 12.2 Module → primary screens (for catalogue writers)

| Module | Primary screens |
|---|---|
| M01 | SCR-ADM-tenant-properties, SCR-ADM-property-profile, SCR-ADM-feature-flags, SCR-ADM-backup-restore, SCR-ADM-deployment-status, SCR-ADM-localization, SCR-OPS-my-profile |
| M02 | SCR-OPS-login, SCR-OPS-step-up, SCR-ADM-users, SCR-ADM-roles-permissions, SCR-ADM-access-review, SCR-ADM-break-glass, SCR-ADM-consent-purposes, SCR-ADM-retention-policies, SCR-ADM-legal-holds, SCR-ADM-dsr-requests, SCR-GST-privacy-consent, SCR-GST-data-request |
| M03 | SCR-ADM-room-setup, SCR-FO-availability-grid, SCR-FO-room-rack, SCR-GM-overbooking-control, SCR-ENG-room-ooo-request |
| M04 | SCR-ADM-rate-plan-setup, SCR-GM-rate-grid, SCR-GM-restrictions-calendar, SCR-FO-reservation-new |
| M05 | SCR-FO-reservation-*, SCR-FO-arrivals, SCR-FO-check-in, SCR-FO-room-move, SCR-FO-stay-extension, SCR-FO-checkout, SCR-FO-no-show-processing, SCR-FO-waitlist, SCR-GST-manage-booking |
| M06 | SCR-FO-home, SCR-FO-hk-room-board, SCR-FO-hk-*, SCR-FO-in-house, SCR-FO-departures |
| M07 | SCR-GM-rate-publish-status, SCR-GM-channel-performance, SCR-ADM-connectors |
| M08 | SCR-FO-folio, SCR-FO-folio-routing, SCR-FO-folio-adjust, SCR-FO-payment-take, SCR-FO-invoice-print, SCR-FIN-cashier-shift, SCR-FO-night-audit |
| M09 | SCR-ADM-timed-resource-setup, SCR-FNB-function-diary, SCR-CORP-event-search |
| M10 | SCR-GM-sales-pipeline, SCR-GM-corporate-account-detail, SCR-GM-corporate-agreement-editor |
| M11 | SCR-CORP-*, SCR-CAPP-* |
| M12 | SCR-GM-group-block, SCR-GM-corporate-rfq-response, SCR-FNB-event-detail, SCR-FNB-beo-editor, SCR-FNB-event-change-order, SCR-FNB-event-actuals-settlement, SCR-FO-group-rooming-list |
| M13 | SCR-FNB-pos-*, SCR-FNB-kds, SCR-FNB-menu-editor, SCR-FNB-happy-hour-pricing |
| M14 | SCR-FNB-recipe-bom, SCR-FNB-outlet-stock, SCR-FNB-stock-count, SCR-FNB-theoretical-vs-actual, SCR-FNB-waste-log, SCR-PRC-stock-balances |
| M15 | SCR-FNB-club-reservations, SCR-FNB-club-entry, SCR-FNB-club-memberships |
| M16 | SCR-FNB-catering-order, SCR-FNB-catering-dispatch, SCR-FNB-production-plan |
| M17 | SCR-PRK-*, SCR-FO-parking-permits, SCR-GST-parking-vehicle, SCR-CORP-parking-passes |
| M18 | SCR-GST-my-trips, SCR-GST-service-requests, SCR-GST-messages, SCR-FO-guest-profile |
| M19 | SCR-FIN-chart-of-accounts, SCR-FIN-posting-rules, SCR-FIN-journal-*, SCR-FIN-manual-journal, SCR-FIN-period-close, SCR-FIN-trial-balance, SCR-FIN-financial-statements, SCR-FIN-accruals-prepayments, SCR-FIN-allocation-rules |
| M20 | SCR-FIN-ap-inbox, SCR-FIN-supplier-invoice-detail, SCR-FIN-three-way-match, SCR-FIN-payables-list, SCR-FIN-payment-batch, SCR-FIN-payment-release, SCR-FIN-ar-*, SCR-FIN-bank-*, SCR-FIN-cash-forecast |
| M21 | SCR-PRC-requisition-*, SCR-PRC-po-*, SCR-PRC-contracts-blanket, SCR-ENG-requisition |
| M22–M24 | SCR-ENG-meter-readings, SCR-ENG-meter-import, SCR-ENG-utility-accounts, SCR-ENG-tariffs, SCR-ENG-consumption-anomalies, SCR-ENG-gas-pipeline, SCR-FIN-utility-bill-inbox, SCR-FIN-utility-bill-detail, SCR-GM-utilities-overview |
| M25 | SCR-PRC-cylinder-exchange, SCR-FIN-gas-cylinder-ledger, SCR-ENG-cylinder-stock |
| M26 | SCR-ENG-*, SCR-VEN-assigned-jobs, SCR-VEN-job-evidence, SCR-VAPP-assigned-jobs, SCR-VAPP-job-evidence |
| M27 | SCR-HR-* |
| M28 | SCR-FO-payment-take, SCR-FO-deposit-preauth, SCR-GST-payment, SCR-GST-pay-balance, SCR-FIN-psp-reconciliation, SCR-FIN-chargeback-case, SCR-FIN-refund-approval, SCR-FIN-payment-release |
| M29 | SCR-FIN-billpay-orders, SCR-FIN-bill-payment-detail |
| M30 | SCR-GST-points, SCR-GST-points-redeem, SCR-ADM-loyalty-program, SCR-FIN-loyalty-liability |
| M31 | SCR-GST-referral-status, SCR-GST-partner-hub-onboarding, SCR-GST-partner-hub-statement, SCR-ADM-referral-program, SCR-FIN-commission-payouts |
| M32 | SCR-GM-flash, SCR-GM-flash-drilldown, SCR-GM-department-pnl, SCR-GM-profit-bridge, SCR-GM-budget-vs-actual, SCR-GM-report-catalogue, SCR-GM-custom-pivot, SCR-GM-scheduled-reports, SCR-OPS-report-viewer |
| M33 | SCR-ADM-webhooks-api-keys, SCR-ADM-event-monitor, SCR-ADM-connectors |
| M34–M37 | Later (Phase 7–8): screens to be added under ADM/GST/FO with ids reserved `SCR-ADM-uc-*`, `SCR-ADM-hsia-*`, `SCR-ADM-iptv-*`, `SCR-GST-marketplace-*` |
| M38 | SCR-ADM-tax-config, SCR-ADM-filing-calendar, SCR-ADM-filing-submission, SCR-ADM-gov-connectors, SCR-FIN-tax-returns, SCR-FIN-invoice-series, SCR-HR-statutory-forms, SCR-HR-employee-sensitive |
| M39 | SCR-MED-library, SCR-MED-upload, SCR-MED-asset-detail, SCR-MED-enhancement-queue, SCR-MED-enhance-compare, SCR-MED-approval, SCR-MED-publish-status, SCR-MED-takedown, SCR-MED-rights-expiry, SCR-MED-listing-preview |
| M40 | SCR-AI-* |
| M41 | SCR-GST-id-capture, SCR-GST-id-confirm, SCR-GST-registration-sign, SCR-GST-otp-verify, SCR-GST-qr-handoff, SCR-SAF-id-intake, SCR-SAF-ocr-correction, SCR-SAF-id-mismatch-review, SCR-SAF-signature-record, SCR-SAF-otp-qr-verify, SCR-SAF-consent-record |
| M42 | SCR-SAF-incident-*, SCR-SAF-sensor-alert-review, SCR-SAF-playbooks, SCR-SAF-on-call, SCR-SAF-post-incident-review |
| M43 | SCR-SAF-lost-*, SCR-GST-lost-item-inquiry |
| M44 | SCR-ADM-jurisdiction-tree, SCR-ADM-legal-entities, SCR-ADM-rule-packs, SCR-ADM-rule-version-review, SCR-ADM-classifier-test, SCR-ADM-coverage-dashboard, SCR-ADM-activation-gates |
| M45 | SCR-CON-* (except fleet), SCR-CORP-travel-requests, SCR-VEN-travel-order-queue, SCR-FIN-travel-reconciliation |
| M46 | SCR-VEN-registration-wizard, SCR-VEN-document-upload, SCR-VEN-verification-status, SCR-PRC-vendor-directory, SCR-PRC-vendor-review, SCR-PRC-vendor-detail, SCR-PRC-credential-expiry, SCR-ADM-service-taxonomy |
| M47 | SCR-FNB-chef-coverage-plan, SCR-FNB-chef-roster, SCR-FNB-emergency-roster, SCR-FNB-chef-callout, SCR-FNB-callout-handover, SCR-VEN-emergency-chef-availability, SCR-VEN-callout-offer, SCR-VAPP-callout-accept |
| M48 | SCR-VAPP-*, SCR-VEN-catalog, SCR-VEN-catalog-item-editor, SCR-VEN-bulk-import, SCR-VEN-daily-stock, SCR-VEN-price-list, SCR-FNB-ingredient-catalog-search |
| M49 | SCR-PRC-rfq-builder, SCR-PRC-rfq-monitor, SCR-PRC-rfq-waiver, SCR-PRC-sample-gallery, SCR-PRC-rfq-comparison, SCR-PRC-award, SCR-VEN-rfq-inbox, SCR-VEN-bid-submit, SCR-ADM-rfq-policy |
| M50 | SCR-PRC-ai-followup-queue, SCR-PRC-delivery-schedule, SCR-PRC-receiving, SCR-PRC-receiving-verify, SCR-PRC-quarantine, SCR-PRC-return-to-vendor, SCR-PRC-store-issue, SCR-PRC-store-return, SCR-PRC-recall-lookup, SCR-VEN-asn-dispatch, SCR-PRC-procurement-reports |
| M51 | SCR-GST-home, SCR-GST-room-search, SCR-GST-room-results, SCR-GST-room-detail, SCR-GST-content-pages, SCR-GST-cookie-consent, SCR-MED-website-pages, SCR-MED-seo-structured-data, SCR-MED-attribution |
| M52 | SCR-MED-marketing-home, SCR-MED-segments, SCR-MED-campaign-editor, SCR-MED-campaign-results, SCR-MED-reviews-inbox, SCR-MED-review-response, SCR-MED-surveys, SCR-FO-guest-merge-review |
| M53 | SCR-GM-revenue-home, SCR-GM-revenue-pickup-pace, SCR-GM-revenue-forecast, SCR-GM-rate-recommendation, SCR-GM-occupancy-forecast |
| M54 | SCR-GST-extras, SCR-FO-upsell-offers, SCR-MED-offers-vouchers, SCR-FIN-voucher-liability |
| M55 | SCR-FO-guest-inbox, SCR-FO-service-requests, SCR-FO-guest-case, SCR-GST-precheckin, SCR-GST-assisted-contact, SCR-GST-feedback-survey |
| M56 | SCR-FO-hk-*, SCR-FO-hk-minibar-count, SCR-FO-hk-linen-par, SCR-FO-hk-laundry-dispatch, SCR-FO-hk-linen-discrepancy |
| M57 | SCR-FNB-restaurant-reservations, SCR-FNB-in-room-dining, SCR-FNB-haccp-checks, SCR-FNB-lot-trace, SCR-FNB-allergen-matrix, SCR-GST-in-room-dining |
| M58 | SCR-FNB-amenity-bookings, SCR-FNB-amenity-config, SCR-GST-amenity-booking |
| M59 | SCR-CON-fleet-dispatch, SCR-CON-trip-manifest, SCR-CON-driver-trip, SCR-CON-vehicle-register, SCR-CON-taxi-transfer |
| M60 | SCR-FIN-cashier-shift, SCR-FIN-cash-drop-safe, SCR-FIN-revenue-protection, SCR-FIN-anomaly-case, SCR-FO-night-audit-exceptions |
| M61 | SCR-SAF-inspections, SCR-SAF-inspection-run, SCR-SAF-nonconformance, SCR-SAF-permit-calendar, SCR-FNB-haccp-checks |
| M62 | SCR-HR-training-matrix, SCR-HR-certifications, SCR-HR-sop-library, SCR-HR-quality-sampling, SCR-HR-labor-forecast, SCR-OPS-shift-handover, SCR-OPS-help-sop |
| M63 | SCR-ADM-workflow-templates, SCR-ADM-workflow-simulator, SCR-OPS-unified-inbox, SCR-OPS-task-detail, SCR-OPS-case-timeline, SCR-OPS-approvals-queue |
| M64 | SCR-ADM-devices, SCR-ADM-integration-health, SCR-ADM-secrets-certs, SCR-ADM-backup-restore, SCR-OPS-outage-mode, SCR-OPS-sync-queue, SCR-PRK-device-status |
| M65 | SCR-GM-data-coverage, SCR-ADM-data-quality, SCR-ADM-kpi-dictionary |
| M66 | SCR-GM-owner-statement, SCR-GM-capex-requests, SCR-FIN-owner-fees, SCR-FIN-fixed-assets, SCR-ENG-capex-project |
| M67 | SCR-GM-sustainability-dashboard, SCR-ENG-sustainability-baseline |
| M68 | SCR-SAF-insurance-register, SCR-SAF-claim-file, SCR-SAF-continuity-plans, SCR-SAF-drills |

---

## 13. UX/design decisions requiring owners (registered in `docs/13`)

| Id | Decision | Default assumption | Owner |
|---|---|---|---|
| D-971 | Chart time-axis direction in Arabic UI | left→right time axis in both languages; validate with pilot users | UX lead |
| D-972 | Accessibility user testing panel (EN/AR screen-reader users) | recruit ≥ 5 users per language before Phase 6 | UX lead |
| D-973 | MetriStay brand palette, logo and typography | placeholder tokens §10 | Metrikingdom marketing |
| D-974 | Arabic/Latin font licences for web, apps and PDFs | open-licence families (OFL) | UX lead + legal |
| D-975 | Guest app: single MetriStay app vs per-hotel white-label builds | single app with property selection; white-label later | Product owner |
| D-976 | Apple Enterprise Program eligibility vs ABM custom apps for staff app | ABM custom app + Managed Google Play | IT lead + legal |
| D-977 | Pilot hotel MDM product and shared-device mode | hotel's existing MDM; fallback enrolment screen | Pilot hotel IT |
| D-978 | Sample-photo retention start event (RFQ close vs approval) — UI default in SCR-ADM-rfq-policy | RFQ close (per §J open decision) | Procurement owner |
| D-979 | LPR confidence threshold for automatic gate decisions | 0.85 | Security manager + pilot hotel |
| D-980 | Low-bandwidth "Lite mode" default on slow connections | auto-suggest when Save-Data or slow ECT detected | UX lead |
