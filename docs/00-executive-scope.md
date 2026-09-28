# 00 — Executive Scope, Release 1 Definition, Feasibility, Staffing, Cost and Decisions

**Pack:** MetriStay Hospitality Suite Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Governing source:** master prompt v3.0 (Sections A, B, E, G, J, N–P). **Companion:** work packages, critical path, gates, partner blockers and exclusions are in `08-phase-backlog.md`; acceptance tests in `09-acceptance-and-migration.md`; regulatory register in `07-security-regulatory.md`.

> Everything below is a plan or a planning assumption. No partner contract, government interface, certification, legal clearance or target KPI exists yet. Estimates are **planning-grade, low confidence** until decisions D-001, D-002, D-023 and D-035 are resolved.

---

## 1. Executive summary

| Topic | Position |
|---|---|
| What | **MetriStay Hospitality Suite** — one integrated hotel operating platform (PMS, distribution, corporate/MICE, F&B/club/catering, parking/LPR, engineering/utilities/gas, procurement/stores, HR/payroll, finance/GL/AP/AR, payments, bill-provider interfaces, loyalty, referral, CRM/revenue, guest AI, identity/e-sign, incident, lost-and-found, five-market jurisdiction classifier) from **Metrikingdom**, on the **MetriSys** technology platform. |
| First customer release | **Customer Release 1: Single Hotel** = all of Phases 2–6. No subset is "Release 1". Phases 2–5 exits are internal milestones. |
| Realistic calendar | Release 1 **~24–36 months after the planning gate** (likely ~28–30), not weeks. Phase 1 remaining: 1–3 months. |
| Effort | Release 1 **~460–1,050 person-months** (likely ~685) incl. 30 % programme overhead; Phases 7–8 a further ~95–230 PM. Confidence **low** (−30 % / +50 %). |
| Team | Peak ~23–34 FTE in Phase 3, organised as 8 stream-aligned bounded-context teams + platform + enabling functions (§12). |
| Dominant risks | Scope breadth (R-001), partner/certification lead times (R-003–R-010), five-market regulatory uncertainty (R-007, R-020), missing blueprints/pilot hotel (R-002, R-023). |
| Honesty rule | A required real workflow that only has a mock is a **release blocker for any property that requires it** (§6.3). |

---

## 2. Product boundaries

### 2.1 In scope (Release 1 unless marked Later)

| Domain | Modules | Release |
|---|---|---|
| Platform, identity, consent, workflow, data governance, devices/resilience | M01, M02, M33 (foundation→mature), M63, M64, M65 | R1 (M33 maturity and M64 UC/HSIA/IPTV parts Later) |
| Rooms, rates, reservations, front desk, housekeeping, distribution, website | M03–M07, M51, M53, M54, M55, M56 | R1 (M53 advanced automation, M55 key/kiosk hardware Later) |
| Folio, cashiering, revenue protection | M08, M60 | R1 |
| Timed facilities, corporate, portal/apps, groups/MICE, club, catering, amenities | M09–M12, M15, M16, M58 | R1 (M58 amenity-specific activation only when owned; golf/beach etc. Later) |
| F&B POS, restaurant, inventory, food assurance, hygiene | M13, M14, M57, M61 | R1 |
| Parking/ANPR | M17 | R1 |
| Guest engagement, CRM, reputation, guest AI, ID/e-sign/OTP, media | M18, M39, M40, M41, M52 | R1 |
| Finance, GL, AP/AR/treasury, BI, owner, sustainability, risk | M19, M20, M32, M66, M67, M68 | R1 (M66 portfolio Later) |
| Procurement, vendors, vendor apps, RFQ/award/PO, receiving/stores, maintenance | M21, M26, M46, M48, M49, M50 | R1 |
| Utilities and gas | M22–M25 | R1 |
| HR, workforce, payroll, staff learning, chef continuity, fleet | M27, M47, M59, M62 | R1 |
| Payments, bill-provider gateway, loyalty points, single-tier referral | M28–M31 | R1 (referral payout activation per market gated; partner distribution expansion Later) |
| Tax, government exchange, five-market classifier | M38, M44 | R1 (certified workflows only where authorized) |
| Incident, lost-and-found | M42, M43 | R1 |
| Travel concierge (air/cruise/taxi) | M45 | R1 (contracted provider exchange only; referral/manual mode otherwise) |
| UC, HSIA, IPTV, smart locks, chains, advanced revenue AI, extended AI | M34, M35, M36, M37 (AI), M53 (advanced) | **Later (Phase 7)** |
| Marketplace, supplier payouts, OTA-like search, dynamic packages, cross-hotel supplier marketplace | M37 (marketplace), M31 (expansion) | **Later (Phase 8)** |

### 2.2 Out of scope (all phases) — summary; authoritative list is `08` §12 (X-01…X-20)
Multi-level/recruiting/downline payouts; self-custodied cash wallet; unlicensed ticket issuance; scraping government portals or CAPTCHA bypass; storing PAN/CVV; reimplementing card networks, bank rails, utility billing systems, camera analytics or payroll clearing; biometric processing without lawful basis; AI autonomous award/delivery confirmation/life-safety triage; unverified green claims; global compliance toggle.

### 2.3 System of record vs external partner ("one software", Section A)

MetriStay owns **one property data and permission model, unified transaction IDs, coordinated workflows and consolidated reporting**. External partners remain system-of-record (SoR) for their regulated service; MetriStay holds a **mirror + reconciliation record** with visible status.

| Record / capability | MetriStay SoR | External SoR (MetriStay mirrors/reconciles) | Adapter status label |
|---|---|---|---|
| Property, rooms, inventory, rates, restrictions, reservations, folios, invoices | **Yes** | Channel manager/OTAs hold their copies; MetriStay is master for ARI | channel: `unverified-assumption` until D-006 |
| Guest profile, consent, preferences | **Yes** | Messaging provider holds delivery logs only | — |
| Card data, authorization, capture, settlement, chargeback | No (tokens, references, status) | **PSP/acquirer/card networks** | D-024 |
| Bank balance, transfers, payouts | No (payment instructions, status, statements imported) | **Bank / licensed PSP** | D-024, D-027 |
| Salary payment clearing / WPS | Payroll calculation, payslips, file generation = **MetriStay**; clearing = bank | **Bank + Ministry WPS system** | D-027 |
| Utility bills and bill payment | Bill record, meter data, approval, AP = **MetriStay** | **Utility provider; Khedmah/ONEIC** for bill-pay orders | D-025, D-030 |
| Loyalty points (non-cash) | **Yes** (append-only ledger) | — | — |
| Cash/stored-value wallet | No | **Licensed bank/PSP** only if D-039 = pursue | Excluded from R1 |
| Referral attribution & commission ledger | **Yes** | Payout execution via bank/PSP | D-033 |
| Video, camera analytics, LPR reads | Observations, decisions, sessions, audit = **MetriStay** | **Camera/LPR server vendor** (video, recognition model) | D-028 |
| Fire alarm, BMS, life-safety | Incident record only | **Fire panel/BMS** remain independent and compliant | D-029 |
| Government filings, tax/payroll submissions | Prepared data, submission record, receipt | **Government authority** (portal/API/file) | D-019 |
| Airline/cruise inventory, tickets, vouchers | Request, consent, quote evidence, external reference, reconciliation | **Airline/aggregator/cruise line/taxi provider** | D-011, D-012 |
| Identity documents, OCR, e-signature evidence | Confirmed fields, signature envelope hash/receipt, retention | ID/e-sign provider if external | D-037 |
| Vendor catalog, daily stock | **Yes** (as published by vendor, with as-of time; not a guarantee) | Vendor ERP if synced | — |
| GL, AP, AR, stock ledgers, fixed assets | **Yes** (single-hotel) | External finance system only if D-023 retains one (export) | — |

---

## 3. Brand architecture (Section E)

| Layer | Name | Use | Clearance |
|---|---|---|---|
| Company | **Metrikingdom** | Legal/commercial entity, contracting party for referral commissions and SaaS | — |
| Technology brand | **MetriSys** | Platform, developer platform (M33), deployment | trademark check (D-004) |
| Product | **MetriStay Hospitality Suite** | Hotel product (all modules) | D-004 |
| Guest & referral ecosystem | **MetriStay Network** | Non-hierarchical guest and commercial partner network — **not** a network-marketing scheme | D-004, D-033 |
| Guest loyalty | **MetriStay Rewards** | Non-cash points (M30) | D-004, D-032 |
| Direct referrer dashboard | **MetriStay Partner Hub** | Single-tier referral statements (M31) | D-004, D-033 |
| Corporate portal/apps | **MetriStay Business** | M11 web + Android/iOS | D-004, D-031 |
| Proposed app names (to clear) | MetriStay Staff, MetriStay Guest, MetriStay Business, MetriStay Vendor | Signed app distribution | D-004, D-031 |

Rule: external names are used only after trademark, app-store and domain clearance (B-25). Until then builds use neutral internal identifiers. Hotel-facing guest channels show **the hotel's brand**; MetriStay branding is secondary ("powered by") per hotel contract.

---

## 4. Assumptions (A-nnn)

Each assumption is `unverified-assumption` until the linked decision closes it.

| ID | Assumption | Linked decision | If false |
|---|---|---|---|
| A-001 | Requirements come from master prompt v3.0 only until the two blueprints are supplied. | D-001 | Traceability delta; re-estimate affected WPs. |
| A-002 | Release 1 is proven with **one pilot hotel** in one market; the other four markets are proven with test fixtures (AT-G09). | D-002, D-010 | Additional pilot = additional counsel, partner and hardware cost. |
| A-003 | Pilot hotel is full-service with rooms, one bar, one club/lounge, one catering kitchen, meeting space and one parking facility (Section G baseline). | D-002, D-007 | Modules not owned stay specified + fixture-tested but not site-accepted. |
| A-004 | Pilot size 80–300 rooms; ≤ 5 outlets; ≤ 300 employees. | D-002 | Performance/capacity and on-prem sizing revised. |
| A-005 | Greenfield repository; stack per README §3.7 (TypeScript modular monolith, PostgreSQL 16, Next.js, React Native/Expo, Keycloak). | — (ADR-001) | — |
| A-006 | Both deployment profiles (`saas`, `onprem-single-hotel`) are built from the same artefacts; the pilot uses one of them. | D-034 | — |
| A-007 | English and Arabic (RTL) are the R1 languages; French (Canada/Québec) and Portuguese are jurisdiction language packs required before activating a property in those markets; Urdu optional for Pakistan staff UIs. | D-010 | Localization effort added to WP-6.5. |
| A-008 | One PSP and one acquiring bank are certified for R1; others follow the same port. | D-024 | — |
| A-009 | One channel manager is certified for R1; direct OTA connections are not built. | D-006 | — |
| A-010 | Khedmah/ONEIC merchant API access is **not** assumed; bill-pay automation ships as provider-neutral port + simulator; manual/bank path is the default. | D-030 | — |
| A-011 | Payroll statutory parameters are configured per market and remain `draft` until adviser sign-off; Canadian payroll may use a certified provider instead of native calculation. | D-018, D-022 | — |
| A-012 | Government submissions default to **export file + manual handoff** unless an authorized API/file route is confirmed per obligation. | D-019 | — |
| A-013 | Travel concierge operates **referral/manual RFQ** mode unless the hotel/Metrikingdom holds the required licence/accreditation for a market. | D-011 | — |
| A-014 | Loyalty points are non-cash, closed-loop, redeemable only on eligible hotel purchases. | D-032 | — |
| A-015 | Referral feature is built and tested in Phase 5 but **payout disabled in every market** until market-specific legal/tax opinion (Oman: Decision 105/2021 review). | D-033 | — |
| A-016 | No cash/stored-value wallet in R1. | D-039 | — |
| A-017 | Guest AI and follow-up AI use a provider port; on-prem profile may use a locally hosted open-weight model after licence review; SaaS may use a hosted model under DPA. | D-038 | — |
| A-018 | Image enhancement uses a locally deployed, permissively licensed or non-generative pipeline — no per-image external fee; compute/storage cost still applies. | D-038 | — |
| A-019 | ID OCR runs through a pluggable port; biometric match/liveness is **off** in R1 unless lawful basis is confirmed per market. | D-021, D-037 | — |
| A-020 | LPR integration via an on-prem **site edge connector** talking to a vendor LPR server/camera and a gate controller with dry-contact or API. | D-028 | — |
| A-021 | Meter data arrives by BMS/API or CSV; where absent, manual reads + bill import are used and labelled `estimate`. | D-025 | — |
| A-022 | Pilot site has a wired LAN, segregated VLANs for devices, and a primary + backup internet link; offline mode covers front desk, housekeeping, POS and gate for a stated duration. | D-009 | Offline scope reduced or hardware added. |
| A-023 | Chart of accounts follows a USALI-style departmental structure configurable to local statutory reporting. | D-040 | Mapping rework in WP-4.1. |
| A-024 | Incumbent systems (PMS/accounting/HR/POS) exist at the pilot and provide exports for migration. | D-023 | Manual data entry at go-live. |
| A-025 | Corporate customers authenticate via MetriStay Business accounts; SSO federation for large corporates is optional. | D-042 | — |
| A-026 | Delivery team works in a blended onshore/offshore model at a blended loaded rate set by D-035; estimates in person-months are location-neutral. | D-035 | Cost figures change; PM do not. |
| A-027 | Sample-photo 90-day retention starts at **RFQ close** by default (configurable to award approval). | D-016 | Configuration change only. |
| A-028 | RFQ default minimum = 3 eligible responsive quotes per category/value threshold, waiver requires higher approver. | D-017 | Configuration change only. |
| A-029 | "NIS" in user requests is treated as **unresolved terminology**; Canadian employee identifier = SIN; no field named NIS is created until D-018 closes. | D-018 | — |
| A-030 | App-store publication is possible under Metrikingdom developer accounts; private/MDM distribution is the fallback for staff/vendor apps. | D-031 | — |
| A-031 | A pilot hotel commits operational staff for acceptance testing, baseline measurement and two month-end close rehearsals. | D-041 | Release acceptance cannot be evidenced. |

---

## 5. Personas (all README §3.3 roles)

Home-screen tasks follow P.1: 3–7 highest-priority tasks with unified inbox and one authoritative status. Screen ids are defined in `docs/04`; journeys in `docs/02`.

### 5.1 Ownership and leadership

| Role | Top goals | Home-screen tasks (3–7) |
|---|---|---|
| `owner` | Know true profitability after all costs; cash/obligations; capex payback | 1 Property operating result vs budget (estimate vs reconciled) • 2 Cash & obligations due (AP, tax, payroll, debt, insurance) • 3 Department/channel/event margin drill • 4 Capex requests awaiting decision • 5 Change since yesterday/month/year |
| `gm` | Run the day safely and profitably; every exception owned | 1 Today: arrivals/departures/VIP/accessibility • 2 Open exceptions by owner & due time • 3 Staffing/chef coverage gaps • 4 Daily flash (occupancy, ADR, RevPAR, TRevPAR) • 5 Approvals (comps, overrides, payables) • 6 Incidents & outages |
| `duty_manager` | Resolve live issues on shift | 1 Live incidents & escalations • 2 Overbooking/walk decisions • 3 Guest complaints SLA • 4 Room defects / OOO • 5 Shift handover notes |

### 5.2 Commercial

| Role | Top goals | Home-screen tasks |
|---|---|---|
| `revenue_manager` | Right price/restriction per date & segment; net contribution | 1 Pace/pickup vs forecast • 2 Recommended rate actions to approve • 3 Channel parity/ack failures • 4 Overbooking risk • 5 Group wash/cutoffs |
| `sales_manager` | Win feasible, profitable corporate/MICE business | 1 RFQ feasibility queue • 2 Proposals expiring • 3 Block pickup vs contract • 4 Corporate credit/PO issues • 5 Pipeline stage changes |
| `marketing_manager` | Profitable direct demand; lawful consented campaigns | 1 Direct funnel & abandonment • 2 Campaigns awaiting approval • 3 Net acquisition cost by channel • 4 Reviews needing response • 5 Consent/suppression alerts |
| `content_editor` | Accurate, rights-cleared media and content | 1 Media awaiting captions/tags • 2 Enhancement jobs to review • 3 Expiring rights • 4 Draft pages |
| `content_approver` | Nothing misleading is published | 1 Approval queue (before/after) • 2 Takedown requests • 3 Syndication failures |
| `referral_program_admin` | Lawful single-tier programme | 1 Referrer applications • 2 Attribution disputes • 3 Pending commissions for approval • 4 Market/kill-switch status • 5 Fraud flags |

### 5.3 Front office and guest service

| Role | Top goals | Home-screen tasks |
|---|---|---|
| `front_office_manager` | Smooth arrivals, accurate folios | 1 Arrivals readiness (room/ID/payment/registration) • 2 Overbooking & room moves • 3 Unsettled folios • 4 Guest requests SLA • 5 Night audit exceptions |
| `front_desk_agent` | Check guests in/out correctly and fast | 1 Search/create reservation • 2 Check-in wizard • 3 Check-out & receipt • 4 Room move/extend • 5 Parking permit • 6 Lost item intake |
| `guest_relations` | Close complaints and recover guests | 1 Open cases by SLA • 2 Compensation approvals • 3 VIP/accessibility pre-arrivals • 4 Post-stay feedback |
| `concierge` | Arrange services with real confirmations | 1 Travel/taxi requests • 2 Quotes expiring • 3 Awaiting supplier reference • 4 Disruptions/refunds • 5 Guest messages |
| `night_auditor` | Close business date accurately | 1 Pre-audit checklist • 2 Run night audit • 3 Difference queue • 4 No-show processing • 5 Audit reports |
| `cashier` | Balanced shift | 1 Open/close shift • 2 Post payments/refunds • 3 Cash drop • 4 Variance explanation |

### 5.4 Housekeeping and laundry

| Role | Top goals | Home-screen tasks |
|---|---|---|
| `housekeeping_supervisor` | Rooms ready on time; linen/amenities sufficient | 1 Room board by priority • 2 Assign attendants • 3 Inspections/recleans • 4 Linen par vs arrivals • 5 Minibar exceptions |
| `housekeeper` | Clear, offline-safe task list | 1 My rooms (priority/DND) • 2 Start/finish/report defect • 3 Minibar count • 4 Found item |
| `laundry_attendant` | Linen custody accurate | 1 Pickup/drop by floor • 2 Vendor dispatch/return weights • 3 Damage/loss record |

### 5.5 F&B, kitchen, club and catering

| Role | Top goals | Home-screen tasks |
|---|---|---|
| `fnb_manager` | Outlet margin, service quality | 1 Outlet sales vs covers • 2 Void/comp approvals • 3 Stock variance • 4 Staffing gaps • 5 Guest complaints |
| `executive_chef` | Safe, covered, cost-controlled kitchen | 1 Chef/backup coverage & gaps • 2 BEO production & allergens • 3 Requisitions/RFQ status • 4 Receiving exceptions/quarantine • 5 Waste & recipe variance • 6 Recall lookup |
| `shift_chef` | Execute today's production | 1 Tickets/production plan • 2 Lot/temperature checks • 3 Issue/return/waste entries • 4 Handover |
| `backup_chef` | Be ready to cover | 1 Standby assignments • 2 Accept/decline cover • 3 Handover pack |
| `emergency_chef` (external) | Accept paid cover safely | 1 Callout offers (accept within deadline) • 2 Assignment details (scoped BEO/allergens) • 3 Time & invoice |
| `bartender` | Fast accurate service | 1 Open tabs • 2 Orders/KDS • 3 Charge to room/event • 4 Shift close |
| `server` | Serve and charge correctly | 1 Tables/orders • 2 Allergen flags • 3 Split/settle • 4 Room-service deliveries |
| `club_host` | Capacity and access respected | 1 Capacity & entries • 2 Reservations • 3 Member/pass check • 4 Minimum spend status |
| `catering_manager` | Events delivered as contracted | 1 BEO versions & changes • 2 Covers/guarantees due • 3 Dispatch/returns • 4 Actual vs contract invoice |

### 5.6 Supply chain

| Role | Top goals | Home-screen tasks |
|---|---|---|
| `procurement_officer` | Competitive, compliant, on-time purchasing | 1 Requisitions to source • 2 RFQs closing / below quote minimum • 3 Comparisons to recommend • 4 POs unacknowledged • 5 Late deliveries (AI follow-up queue) |
| `procurement_approver` | Approve with evidence and SoD | 1 Awards/overrides awaiting approval • 2 Waivers • 3 Budget encumbrance breaches |
| `storekeeper` | Accurate, safe stock | 1 Expected deliveries • 2 Quarantine queue • 3 Issue requests • 4 Expiring lots (FEFO) • 5 Stocktake |
| `receiver` | Low-touch accountable receipt | 1 Arriving now (ASN) • 2 Scan/weigh/temperature • 3 Discrepancies • 4 Attest GRN |

### 5.7 Engineering, security, parking

| Role | Top goals | Home-screen tasks |
|---|---|---|
| `chief_engineer` | Assets safe, rooms returned to sale, utility costs controlled | 1 Critical faults & SLA • 2 OOO rooms & return dates • 3 PM schedule compliance • 4 Meter anomalies/bill variances • 5 Vendor jobs awaiting acceptance • 6 Cylinder stock |
| `engineer` | Do assigned work with evidence | 1 My work orders • 2 Parts requisition • 3 Photos/completion • 4 Meter reads |
| `security_officer` | Detect, confirm, escalate | 1 Live alerts to confirm • 2 Incident chronology • 3 Gate overrides • 4 Lost-and-found custody |
| `parking_attendant` | Correct entry/exit and charges | 1 Low-confidence plate reviews • 2 Manual gate events • 3 Occupancy • 4 Permit lookup |

### 5.8 Finance

| Role | Top goals | Home-screen tasks |
|---|---|---|
| `finance_clerk` | Clean postings and exports | 1 Posting exceptions • 2 Bank lines to match • 3 Folio-to-GL differences |
| `ap_clerk` | Pay right invoices once | 1 Invoices to capture • 2 Match exceptions (2/3-way) • 3 Duplicates flagged • 4 Disputes |
| `ar_clerk` | Collect corporate receivables | 1 Aging buckets • 2 Invoices to issue • 3 Receipts to allocate • 4 Credit-limit breaches |
| `finance_approver` | Approve payables/credit within limits | 1 Payables for approval • 2 Write-offs • 3 Commission approvals |
| `payment_releaser` | Release authorized payments safely | 1 Batches awaiting release (dual control) • 2 Rejected/failed payments • 3 Pending bill-pay orders |
| `financial_controller` | Close on time with reconciled numbers | 1 Close checklist • 2 Reconciliation dashboard (planned/accrued/invoiced/approved/paid/settled) • 3 Allocation versions • 4 Tax calendar • 5 Unreconciled items age |

### 5.9 HR and workforce

| Role | Top goals | Home-screen tasks |
|---|---|---|
| `hr_officer` | Accurate employee records & compliance | 1 Joiners/leavers • 2 Expiring certifications/permits • 3 Leave approvals • 4 Contract changes |
| `payroll_officer` | Correct pay on time | 1 Payroll run status & exceptions • 2 WPS/bank file status • 3 Rejections to resubmit • 4 Statutory parameter status |
| `payroll_approver` | Confidential maker-checker | 1 Payroll for approval (aggregate + drill) • 2 Variances vs last period |
| `employee` | See roster/pay; request leave | 1 My roster & swaps • 2 Clock in/out • 3 Leave request • 4 Payslips • 5 Training due |

### 5.10 Governance and technology

| Role | Top goals | Home-screen tasks |
|---|---|---|
| `compliance_officer` | Only verified rules drive automation | 1 Rule-pack status per market (draft/verified/expired) • 2 Blocked automations • 3 Counsel questions open • 4 Upcoming rule changes • 5 Filing receipts |
| `dpo` | Lawful, minimal personal data | 1 DSAR queue • 2 Consent anomalies • 3 Retention/deletion jobs • 4 ID-image deletion proof • 5 DPIA actions |
| `it_admin` | Systems healthy and recoverable | 1 Device/cert expiry • 2 Backup/restore status • 3 Outage/degraded queues • 4 Patch approvals |
| `integration_admin` | Partners connected and reconciled | 1 Connector health • 2 Dead letters • 3 Reconciliation breaks • 4 Credential rotation due |
| `property_admin` | Property configured correctly | 1 Config changes pending • 2 Outlets/facilities enablement • 3 User access requests |
| `tenant_admin` | Tenant-level governance | 1 Properties & licences • 2 Admin access review • 3 Feature flags |
| `auditor` (read-only) | Evidence without modification | 1 Audit trail search • 2 Ledger drill • 3 Export evidence packs |

### 5.11 External: guests, corporate, vendors, referrers

| Role | Top goals | Home-screen tasks |
|---|---|---|
| `guest` | Honest price, easy stay, help when needed | 1 My booking & pre-arrival • 2 Check-in (ID/sign/verify) • 3 Requests & chat • 4 Folio/receipts/pay • 5 Points • 6 Parking vehicle |
| `booker` | Book for others correctly | 1 Search/quote • 2 My bookings • 3 Occupant details • 4 Payment/receipts |
| `referrer` | Transparent single-tier earnings | 1 My link/code & disclosure • 2 Attributed completed bookings • 3 Commission statement (pending/approved/paid/reversed) • 4 Disputes |
| `corporate_admin` | Control company use | 1 Users/roles • 2 Contracted rates • 3 Invoices/credit • 4 Analytics |
| `corporate_booker` | Book rooms/events within policy | 1 Search facilities by date/attendees • 2 Compare/hold • 3 Rooming lists • 4 Itinerary |
| `corporate_approver` | Approve spend | 1 Pending approvals • 2 Budget vs spend |
| `event_organizer` | Event delivered as agreed | 1 Confirmed vs proposed • 2 BEO/changes • 3 Attendee check-in • 4 Bill |
| `vendor_admin` | Stay approved and win business | 1 Verification/document expiry • 2 Catalog & daily stock • 3 RFQs to answer • 4 POs to acknowledge • 5 Invoices/payments |
| `vendor_user` | Fulfil correctly | 1 Assigned jobs/POs • 2 Dispatch/ASN • 3 Job evidence • 4 Messages |

### 5.12 System actors
`outbox_relay`, `night_audit_worker`, `billpay_worker`, `ai_assistant` (bounded tools only), plus `channel_worker`, `payroll_worker`, `purge_worker` (sample-photo/ID deletion), `followup_worker` (AI delivery follow-up), `lpr_edge_worker`. None may approve, award, release payment or confirm delivery.

---

## 6. Customer Release 1: Single Hotel — definition

### 6.1 Definition
Release 1 is the complete single-hotel system delivered by **Phases 2, 3, 4, 5 and 6 together**, covering every Section C module marked R1 in §2.1 and every Section I Release-1 capability. Internal milestone demonstrations and pilots of Phase 2–5 slices are allowed and must be labelled "internal milestone — not Release 1".

### 6.2 What "done" means (all must be true)
| # | Criterion |
|---|---|
| DN-1 | All AT-G01…AT-G20 pass on a production-like environment configured for the pilot hotel, with working integrations (not mock UIs) for every workflow the pilot requires. |
| DN-2 | Two consecutive month-end closes reconcile folio ↔ PSP ↔ bank ↔ AR/AP ↔ GL; departmental P&L signed off by the pilot financial controller. |
| DN-3 | Every WP in Phases 2–6 meets the vertical-slice DoD (V1–V8, `08` §1.1). |
| DN-4 | Security: external pen test with no open critical/high; tenant isolation suite; PCI scope documented; DPIA per activated market. |
| DN-5 | Restore drill with measured RPO/RTO; internet-outage drill; cyber tabletop. |
| DN-6 | EN/AR/RTL reviewed; WCAG 2.2 AA audit of booking site, guest app and critical staff flows with no blocking issues. |
| DN-7 | Five-market classifier coverage report complete; every feature activated for the pilot has rule-pack evidence at `counsel-reviewed` or higher; unknowns block dependent automation. |
| DN-8 | Runbooks, training and support model accepted by the pilot. |
| DN-9 | Success-metric baselines captured and targets agreed with the pilot (§8). |
| DN-10 | Per-property blocker matrix (§6.3) published; no blocker hidden. |

### 6.3 Per-property release blockers (P.5: "Release 1 is blocked for a property when a required real workflow has only a mock")

| If the property requires… | Required proof | If only a mock/simulator exists |
|---|---|---|
| OTA/channel distribution | Channel manager `certified` (B-01) | **Blocked** for channel sales; may go live direct-only if the hotel accepts in writing |
| Card acceptance in MetriStay | PSP `certified` (B-02) | **Blocked**; standalone terminal + manual folio posting is an interim operating mode, not R1 compliance |
| Automated supplier/salary payouts | Bank/PSP file/API `sandbox-tested`→`certified` (B-03, B-04) | Manual bank-portal upload of generated files; automation blocked |
| Oman WPS | Bank WPS spec + successful test submission (B-04) | **Blocked** for Oman payroll payment |
| Electronic utility bill-pay | Khedmah/ONEIC or other provider contract + tested settlement (B-05) | Automation `blocked`; manual/bank payment reconciled (allowed, Section G.6) |
| LPR/gate automation | Installed device passes AT-G03.2 on site (B-06) | **Blocked** for automated gate; manual attendant mode |
| BMS/fire signal ingest | Protocol tested read-only (B-07) | Manual incident intake only |
| SMS/WhatsApp OTP | Provider + sender/template approval (B-08) | Email/in-person verification |
| Government e-invoicing / filing mandated in market | Authorized route certified (B-11…B-15) | **Blocked** for operation in that market if mandatory (e.g. if e-invoicing integration is confirmed mandatory) |
| Canadian payroll | Native parameters adviser-signed or provider certified (B-28) | **Blocked** for Canadian payroll |
| Air/cruise booking by hotel | Licence/accreditation + contracted adapter (B-16, B-17) | Referral/manual RFQ mode only |
| Downloadable corporate/vendor apps | Store or signed distribution approved (B-19) | Web fallback only; "downloadable app" claim blocked |
| Referral payouts in a market | Written legal/tax opinion (B-21…) | Payouts disabled (attribution may run in test) |
| Local AI (media/assistant) | Model licence review + capacity test (B-26) | Feature disabled or classical pipeline |

---

## 7. Phase 7–8 scope (Later)

| Phase | Scope | Exit (Section B) | Preconditions |
|---|---|---|---|
| 7 — Connected & multi-property | Chains/central config, cross-property guest governance, UC/wake-up (M34), HSIA (M35), IPTV (M36), smart locks/digital key/kiosk, advanced revenue optimization (M53), extended bounded AI automations (M37 AI), developer platform maturity, remaining amenity businesses | Named vendor certifications, locality/data controls, cross-property reporting, operational acceptance | R1 live and stable; named hardware vendors; data-residency decisions per market |
| 8 — Marketplace & jurisdiction-gated growth | Multi-supplier hotel/experience marketplace, supplier payouts via licensed PSP, OTA-like search, dynamic packages, cross-hotel supplier marketplace, direct commercial referral expansion where legally reviewed; **no multi-level payouts** | Supplier/traveller/payment compliance, fraud/dispute resolution, lawful marketing model, independent profitability | Phase 7 gate; licensed marketplace-payments partner; package-travel and consumer-law review per market |

---

## 8. Success metrics (Section P.6)

Targets are **agreed with the pilot hotel in Phase 1 (D-005, D-041)** and recorded in `docs/13`; none are universal. Baselines are measured for **at least 8–12 weeks** before go-live (or from 12 months of historical data where the incumbent system allows), and definitions live in the KPI dictionary (M32/M65, `docs/06`).

| ID | Metric | Definition (summary) | Baseline method | Data source after go-live | Target |
|---|---|---|---|---|---|
| MET-01 | Guest quote-to-book conversion | Paid bookings ÷ quotes issued (direct web, voice, desk), by channel | Incumbent booking-engine analytics + call log sampling; if absent, 4-week manual tally | M51/M04 events | Agreed with pilot hotel |
| MET-02 | Direct net acquisition cost | (Marketing spend + payment fees + commissions attributable) ÷ completed direct stays | Last 12 months invoices for ads/metasearch/commissions | M51/M52/M20 | Agreed with pilot hotel |
| MET-03 | Occupancy and net RevPAR | Occupancy excl. OOO per definition; RevPAR net of channel commissions and payment fees | Incumbent PMS night-audit reports (12 months) | M32 | Agreed with pilot hotel |
| MET-04 | Complaint time-to-closure | Median/P90 hours from case open to guest-confirmed closure | Sample of logbook/email cases over 8 weeks | M55 | Agreed with pilot hotel |
| MET-05 | Housekeeping turnaround | Minutes from departure to inspected-ready, by room type | 2-week timed observation / incumbent HK timestamps | M56 | Agreed with pilot hotel |
| MET-06 | Vendor on-time-in-full (OTIF) | Deliveries on time and complete ÷ deliveries | 8-week receiving log sample | M50 | Agreed with pilot hotel |
| MET-07 | Food waste | Waste cost (and kg) ÷ food cost (or ÷ covers) | Waste log or 2-week weighed audit | M50 F50.3.5 | Agreed with pilot hotel |
| MET-08 | Average stock age | Value-weighted days on hand per store | Stocktake + purchase dates | M14/M50 | Agreed with pilot hotel |
| MET-09 | Utility cost per occupied room | Electricity + water + gas cost ÷ occupied room nights (occupancy- and weather-context noted) | 12 months bills ÷ occupied rooms | M22–M25, M67 | Agreed with pilot hotel |
| MET-10 | Outstanding AR | Corporate AR balance and DSO by aging bucket | Incumbent AR aging at 3 month-ends | M20 | Agreed with pilot hotel |
| MET-11 | Payroll errors | Corrections/off-cycle payments ÷ payslips per run | Last 6 payroll runs' correction records | M27 | Agreed with pilot hotel |
| MET-12 | Payment disputes | Chargebacks + guest billing disputes ÷ payment transactions | PSP/bank reports 12 months | M28/M60 | Agreed with pilot hotel |
| MET-13 | Incident acknowledgment time | Median/P90 seconds from alert/report to accountable acknowledgment | Drill measurement pre-go-live | M42 | Agreed with pilot hotel |
| MET-14 | Uptime | Availability of critical services (booking, front desk, POS, gate) per SLO definition | Incumbent incident log (if any) | M64 monitoring | Agreed with pilot hotel (and contract SLA) |
| MET-15 | Recovery | Measured RTO/RPO in restore/outage drills | First drill in WP-2.20 | M64 | Agreed with pilot hotel |

---

## 9. Feasibility assessment

| Dimension | Assessment | Key conditions | Rating |
|---|---|---|---|
| **Technical** | Feasible with a disciplined modular monolith: all capabilities are known patterns (inventory invariants, double-entry ledgers, outbox, RLS, offline queues). Complexity lies in breadth (68 modules), correctness of ledgers/invariants, offline sync, and bilingual/accessible UX across 5 app surfaces. | Early ledger/event contracts; strong test automation; stable team; scope control (R-001, R-017, R-021). | **Feasible — high effort** |
| **Partner** | Each external dependency has multi-month lead time outside our control (PSP, channel manager, banks/WPS, Khedmah/ONEIC, hardware, messaging, app stores, travel providers). Khedmah/ONEIC merchant APIs are unconfirmed; airline ticketing requires authority we do not have. | Start commercial workstreams during Phase 1; simulators for all; honest blockers (R-003–R-010, R-016, R-027). | **Partly feasible — schedule risk** |
| **Regulatory** | Five markets with distinct tax, invoicing, payroll, guest-registration, privacy, payments and travel rules; some markets may mandate certified invoicing/e-invoicing integration; referral model requires Oman legal opinion; biometric/ID processing needs lawful basis. None can be assumed. | Local counsel per market (B-20…B-24); rule packs gated; pilot in one market first (R-007, R-011, R-013, R-020). | **Feasible for one pilot market in R1; others via fixtures + gated activation** |
| **Commercial** | ~685 PM likely (up to ~1,050) for R1 plus vendor licences, hardware, counsel and certification fees; revenue before R1 only from pilot/design-partner arrangements. Competes with established suites (OPERA, Mews, Cloudbeds, SiteMinder) — differentiation is integrated ERP/procurement/utilities/payroll for single hotels in the target markets. | Funding envelope (D-035); pilot commitment (D-041); phased commercial pilots of internal milestones allowed if labelled (R-001, R-019, R-023). | **Viable only with committed multi-year funding and a pilot partner** |

---

## 10. Decisions (Section J, converted) — D-001…D-042

Owner = accountable role (programme roles or README §3.3 roles). "Interim assumption" is used for design until closed; "Simulator/test path" keeps work unblocked; "Release dependency" is what cannot be claimed without the decision. All **open** as of 2026-09-28; target close = planning gate unless noted.

| ID | Decision | Owner role | Interim interface assumption | Simulator / test path | Release dependency created |
|---|---|---|---|---|---|
| D-001 | Supply the two original hospitality blueprints | Product Owner | Master prompt v3.0 is sole source (A-001) | Traceability index has "blueprint delta" column | Traceability completeness at planning gate; re-estimate |
| D-002 | Pilot hotel: country/province/municipality, legal entity, size, outlets, facilities | Product Owner + pilot `owner` | Section G baseline hotel (A-003/A-004), market TBD | Five fixture properties (AT-G09) | Which rule packs, partners and hardware become R1 blockers |
| D-003 | Guest acquisition baseline: website, search/maps, OTA/GDS, corporate pipeline | `marketing_manager` (pilot) | Direct web + 1 channel manager + corporate direct | Attribution fixtures | MET-01/02 baselines; WP-3.24/5.9 scope |
| D-004 | Brand/domain/trademark/app names; privacy & analytics preferences | Metrikingdom brand owner + `dpo` | Neutral internal names; consent-first analytics, no third-party trackers by default | — | Public naming, app-store listing (B-25) |
| D-005 | Target ADR/occupancy/profit, allowable rate bounds, KPI targets | pilot `owner` + `revenue_manager` | Guardrails: manual approval for every rate change | Backtest fixtures | WP-5.8 guardrails; §8 targets |
| D-006 | Channel manager selection, agreements, review-site access | `revenue_manager` + Partner Manager | Generic channel port (ARI push, booking pull/push, ack) | Channel simulator | B-01; channel sales at R1 |
| D-007 | Enabled amenities (spa/pool/gym/golf/retail), laundry model (in-house/outsourced), minibar | pilot `gm` | Spa+pool+gym simple timed bookings; outsourced laundry; minibar manual count | Fixture amenities | WP-3.8/4.19 site acceptance scope |
| D-008 | Cash handling and loss-control policy (floats, drops, limits, safe) | `financial_controller` | Blind drop, dual count over threshold, variance approval | Cash shift fixtures | WP-2.10 configuration; AT-G08 |
| D-009 | Network/device/outage profile; site network availability | pilot `it_admin` | A-022 (dual WAN, VLANs, offline for FD/HK/POS/gate) | Outage injection | Offline scope claims; WP-6.9 |
| D-010 | Five-market launch sequence | Metrikingdom CEO + Product Owner | Pilot market first; others fixture-only | AT-G09 | Language packs, counsel engagement order |
| D-011 | Travel operating model per market: licensed seller/agency vs referral-only | `compliance_officer` + counsel | Referral/manual RFQ only (A-013) | Travel simulator with capability flags | B-16; ticketing claims |
| D-012 | Flight, cruise, taxi provider contracts and API coverage | Partner Manager | Provider-neutral travel port | Travel simulator; manual RFQ | B-16–B-18; WP-5.7/6.10 |
| D-013 | Vendor category taxonomy and verification owners per category | Procurement lead (`procurement_approver`) | Seed taxonomy from Section C/K categories | Fixture vendors (AT-G10) | WP-2.16/3.12 approval flows |
| D-014 | Chef/backup qualifications, emergency roster sources and agreements | `executive_chef` + `hr_officer` | Food-safety credential mandatory; roster from agency agreements | Callout simulator (AT-G15) | WP-3.14 live callout |
| D-015 | Receiving hardware (scanners, scales, probes) and cold-chain controls | `storekeeper` lead + `chief_engineer` | Mobile camera barcode + manual weight/temperature entry | Device simulators | B-27; WP-6.7 |
| D-016 | Sample-photo purge start event | Procurement lead + `dpo` | RFQ close (A-027) | Purge job with clock fixtures | AT-G17.2 configuration |
| D-017 | RFQ minimum quotes, weights, override policy | `financial_controller` + Procurement lead | 3 quotes; published weights; higher-approver waiver (A-028) | AT-G17 fixtures | WP-3.15 policy config |
| D-018 | **NIS/SIN terminology:** what "NIS" meant, in which jurisdiction, and whether a Canadian SIN was intended | Product Owner + `compliance_officer` (with Canadian payroll adviser) | Canada uses **SIN** (restricted, encrypted, masked); "NIS" stored nowhere and never treated as Canadian tax identifier; if NIS is found to mean another national identifier (e.g. a social-security or national insurance number in another market), it becomes a jurisdiction-specific field in that market's pack | SIN validation fixtures; no NIS field | WP-4.12; Canadian payroll release; catalogue terminology |
| D-019 | Permitted government filing method per country/obligation (API/file/portal/manual) | `compliance_officer` | Export + manual handoff (A-012) | Government adapter simulator incl. outage | B-11–B-15; WP-5.6 |
| D-020 | Jurisdiction tax counsel engaged per market | CFO + `compliance_officer` | Rule packs `draft` | Rule-pack fixtures | B-20–B-24; DN-7 |
| D-021 | Privacy and e-signature counsel; biometric position per market | `dpo` | No biometrics; consent-based ID OCR with manual alternative; e-sign as evidence with paper fallback | DPIA templates | WP-2.14, WP-3.20, WP-6.4 |
| D-022 | Canadian payroll: native engine vs certified payroll provider | Payroll lead + Product Owner | Native engine with provider port | Parameter fixtures vs published CRA tables | B-28; WP-4.12 |
| D-023 | Current PMS/accounting/HR/POS at pilot and migration scope | pilot `it_admin` + `financial_controller` | Exports available (A-024) | Synthetic migration datasets | WP-6.2; go-live |
| D-024 | Bank and PSP (acquirer, terminals, pay-by-link, payouts) | `financial_controller` | Generic PSP port; hosted fields/redirect | PSP simulator | B-02, B-03; WP-5.1/5.2 |
| D-025 | Utility suppliers, account formats, meter/BMS data access | `chief_engineer` | CSV/manual meter + bill import | Meter data generator incl. gaps/resets | WP-4.6/4.7 automation level |
| D-026 | Gas supplier, pipeline meter and cylinder practices (sizes, deposits, IDs) | `chief_engineer` + `fnb_manager` | Cylinders tracked by type+count, serial optional | Cylinder custody fixtures | WP-4.8 |
| D-027 | Payroll/WPS bank and salary-file channel | `payroll_officer` + `financial_controller` | Generic WPS salary-information file per published spec, versioned | File-format validator + bank test | B-04; Oman payroll payment |
| D-028 | Camera/LPR/gate models and parking edge topology | `chief_engineer` + `security_officer` | Edge connector + vendor LPR API + gate relay (A-020) | LPR/gate simulator with confidence | B-06; WP-3.10/6.7 |
| D-029 | BMS/fire/alarm integration scope | `chief_engineer` + safety lead | Read-only signal ingest; life-safety independent | Signal simulator | B-07; WP-3.23 |
| D-030 | Khedmah/ONEIC commercial and API access | Partner Manager + `financial_controller` | Provider-neutral bill-pay port (A-010) | Bill-pay simulator incl. timeout/duplicate | B-05; AT-G06.3 mode |
| D-031 | App-store developer accounts; public vs private/MDM distribution | `it_admin` + Metrikingdom | Org accounts; internal builds until approved (A-030) | EAS internal distribution | B-19; WP-6.12 |
| D-032 | Loyalty terms and accounting policy (earn/burn, expiry, liability/breakage) | `financial_controller` + counsel | Liability accrued at issue; expiry configurable | Points ledger fixtures | WP-5.4; MetriStay Rewards launch |
| D-033 | Referral terms (commission rate, margin formula, allowed deductions, referrer categories, channels) and Oman legal/tax opinion | Metrikingdom CEO + counsel | Payout disabled everywhere (A-015) | Non-negotiable referral tests | B-21; WP-5.5/6.6; payout activation |
| D-034 | Deployment location: SaaS region/data residency; pilot on SaaS vs on-prem | CTO + `dpo` | Both profiles built (A-006) | Both profiles in CI | Hosting contracts; residency claims |
| D-035 | Programme budget, funding envelope, blended rate, sourcing model | CEO/CFO | Person-month plan (§13) | — | Staffing plan; calendar commitment |
| D-036 | SMS/WhatsApp providers, sender IDs and templates per country | `integration_admin` | Messaging port | Messaging simulator | B-08; WP-5.10 |
| D-037 | ID/OCR and e-signature provider(s) or self-hosted | `dpo` + CTO | Local OCR engine candidate; self-hosted signature evidence service | OCR error fixtures | B-09, B-10 |
| D-038 | Local AI models and licences (image enhancement, guest assistant LLM, follow-up drafting); GPU capacity | CTO + counsel | Provider port; classical enhancement fallback (A-017/A-018) | Evaluation harness (hallucination/prompt-injection) | B-26; WP-3.19/3.20 |
| D-039 | Cash/stored-value wallet: pursue via licensed partner or not | CEO + counsel | **Not in R1** (A-016) | — | Only a new product decision reopens |
| D-040 | Accounting framework and chart-of-accounts template | `financial_controller` | USALI-style departmental template (A-023) | Posting fixtures | WP-4.1 |
| D-041 | Pilot hotel commitment: acceptance participation, baseline window, close rehearsals | Product Owner + pilot `owner` | A-031 | — | DN-1, DN-2, DN-9 |
| D-042 | Corporate SSO requirements (SAML/OIDC federation) for pilot corporate customer | `sales_manager` + `it_admin` | Local accounts + optional federation (A-025) | IdP simulator | WP-3.3 scope |

---

## 11. Risks (R-001…R-027)

Likelihood/impact: H/M/L. Owners are accountable roles.

| ID | Risk | L | I | Mitigation | Owner |
|---|---|---|---|---|---|
| R-001 | Release 1 scope breadth (≈60 R1 modules) exceeds funding/calendar tolerance | H | H | Honest estimates; phase gates; configurable activation; re-estimate per gate; no subset called R1 | Product Owner |
| R-002 | Blueprints missing → requirement gaps discovered late | M | M | D-001; traceability delta column; change control | Product Owner |
| R-003 | Channel manager certification queue/lead time | M | H | Select in Phase 1; sandbox in Phase 2; B-01 blocker | Partner Manager |
| R-004 | PSP onboarding/certification and PCI scope creep | M | H | Hosted fields/redirect; D-024 early; SAQ documented | `financial_controller` |
| R-005 | Khedmah/ONEIC merchant API unavailable | H | M | Provider-neutral port; manual/bank path; blocked label | Partner Manager |
| R-006 | WPS bank spec/test access delayed | M | H | Early bank engagement; file validator | `payroll_officer` |
| R-007 | Mandatory certified invoicing/e-invoicing or filing integrations in some markets (to verify) | M | H | Counsel register; pilot market first; per-market blocker | `compliance_officer` |
| R-008 | Canadian payroll complexity (federal/provincial/QC) and yearly parameter changes | M | H | D-022 provider option; adviser sign-off; yearly update runbook | Payroll lead |
| R-009 | Hardware (LPR/gate/scales/BMS) integration and site install delays | M | M | Simulators; pilot-site install windows; manual fallbacks | `chief_engineer` |
| R-010 | App-store review rejection or account delays | M | M | Apply in Phase 2; private/MDM fallback | `it_admin` |
| R-011 | Referral model judged as network marketing or needing licences (Oman/others) | M | H | Single-tier only; payout disabled until opinion; kill switch | CEO + counsel |
| R-012 | Pressure to add cash wallet without licence | L | H | Excluded (X-03); D-039 | CEO |
| R-013 | ID image/biometric/signature processing lacks lawful basis in a market | M | H | No biometrics; manual path; DPIA; timed deletion | `dpo` |
| R-014 | AI hallucination/prompt injection (guest assistant, follow-ups) | M | H | Bounded tools, citations, eval harness, human handoff, no autonomous commitments | ML lead |
| R-015 | Local AI model licence restrictions or insufficient on-prem GPU/CPU | M | M | Licence review; classical pipeline fallback; cost metering | CTO |
| R-016 | Travel intermediation licensing (ticketing/package travel) | H | M | Referral/manual mode default; D-011 | `compliance_officer` |
| R-017 | Offline sync conflicts corrupt room/stock/POS state | M | H | Server-acknowledged commits; conflict policies; drills | Platform lead |
| R-018 | Data migration from incumbent systems incomplete/dirty | M | M | Two rehearsals; reconciliation reports | Data lead |
| R-019 | Hiring/retaining hotel-finance, payroll and hospitality domain expertise | M | H | Domain BAs early; external advisers; documentation | Delivery Manager |
| R-020 | Counsel cost/time across five markets exceeds plan | M | M | Market sequencing (D-010); fixture-only for non-pilot markets | CFO |
| R-021 | Invariant defects (double sale, double capture, duplicate receipt) under load | L | H | Property-based concurrency tests; idempotency everywhere; AT-G20 | Architect |
| R-022 | Tenant/property data leak (BOLA) | L | H | RLS + policy engine + isolation suite + pen test | Security lead |
| R-023 | Pilot hotel unavailable or withdraws | M | H | D-041 contract; secondary pilot candidate | Product Owner |
| R-024 | Low vendor adoption of mobile catalog/daily stock | M | M | Web fallback, CSV/API sync, onboarding support | Procurement lead |
| R-025 | Arabic/RTL and accessibility quality insufficient | M | M | RTL from S2-00; native reviewers; axe in CI; WCAG audit | UX lead |
| R-026 | Scope creep from Phase 7–8 into R1 | M | M | Change control; Later flag enforced in catalogue | Product Owner |
| R-027 | Messaging sender-ID/WhatsApp template approvals vary by country | M | M | Early registration; email/in-person fallback | `integration_admin` |

---

## 12. Staffing model

### 12.1 Team topology by bounded context

| Team | Type | Bounded contexts / modules owned | Most active phases |
|---|---|---|---|
| T0 Platform & Identity | Platform | M01, M02, M33, M63 (engine), M64, M65 (foundation), outbox/audit, deployment profiles, staff-mobile shell | 2 (lead), all |
| T1 Rooms & Distribution | Stream | M03, M04, M05, M06 (front desk), M07, M51, M53, M54 | 2–3, 5 |
| T2 Folio, Payments & Rewards | Stream | M08, M28, M29, M30, M31, M60 | 2, 5 |
| T3 Corporate, Events & Venues | Stream | M09, M10, M11, M12, M15, M16, M58 | 3 |
| T4 F&B, Supply & Stores | Stream | M13, M14, M21, M46, M47, M48, M49, M50, M57 | 3–4 |
| T5 Finance, Workforce & Tax | Stream | M19, M20, M27, M32, M38, M44, M66 | 2 (M44/M38 fdn), 4–5 |
| T6 Operations, Engineering & Safety | Stream | M06/M56 housekeeping & linen, M17, M22–M26, M42, M43, M59, M61, M62, M67, M68 | 3–4 |
| T7 Guest Experience, Media & AI | Stream | M18, M39, M40, M41, M45, M52, M55 | 2–3, 5 |
| Enabling | Enabling | UX/a11y/localization, QA automation & E2E, SRE, security/privacy, data/BI, compliance coordination, partner management, documentation/training | all |

Teams are introduced progressively: Phase 2 runs T0, T1, T2, T5 (foundation) and a combined T6/T7 squad; Phase 3 stands up T3, T4, T6, T7 fully; Phase 4 shifts capacity from T1/T3 to T5/T6; Phase 5 grows T2 and integration engineers; Phase 6 is cross-team hardening.

### 12.2 Roles and FTE ranges per phase (low–high)

| Role | P2 | P3 | P4 | P5 | P6 | P7 | P8 |
|---|---|---|---|---|---|---|---|
| Product owner / product managers | 1–1.5 | 2 | 1.5–2 | 1.5–2 | 1–1.5 | 1 | 1 |
| Domain business analysts (front office, F&B, finance, payroll, procurement) | 1–2 | 2–3 | 2–3 | 1–2 | 1–1.5 | 1 | 1 |
| Solution architect | 1 | 1–1.5 | 1 | 1 | 0.5–1 | 1 | 0.5–1 |
| Backend engineers | 4–6 | 5–8 | 5–8 | 4–6 | 2–4 | 3–5 | 2–4 |
| Web frontend engineers | 2–3 | 2–4 | 2–3 | 1.5–2 | 1–2 | 1–2 | 1–2 |
| Mobile engineers (React Native) | 1–1.5 | 2–3 | 0.5–1 | 1 | 1 | 0.5–1 | 0–0.5 |
| Integration engineers (partner adapters) | 0–0.5 | 1–2 | 1–2 | 3–4 | 1–1.5 | 2–3 | 1–1.5 |
| QA / test automation | 2–3 | 2.5–4 | 2–3 | 2–3 | 2.5–4 | 1–2 | 1–1.5 |
| UX / accessibility / localization | 1–1.5 | 1–2 | 0.5–1 | 0.5–1 | 1–1.5 | 0.5 | 0.5 |
| SRE / DevOps | 1–1.5 | 1–1.5 | 1 | 1 | 1–1.5 | 0.5–1 | 0.5 |
| Security / privacy engineer | 0.5 | 0.5 | 0.5 | 1 | 1–1.5 | 0.5 | 0.5 |
| Data / BI engineer | 0.5 | 0.5 | 1–2 | 0.5–1 | 0.5 | 0–0.5 | 0–0.5 |
| ML / AI engineer | 0–0.5 | 1 | 0–0.5 | 1 | 0–0.5 | 0.5–1 | 0–0.5 |
| Compliance coordinator (in-house; counsel external) | 0.5 | 0.5 | 1 | 1–1.5 | 1 | 0.5 | 1 |
| Partner / alliance manager | 0.5 | 0.5 | 0.5 | 1 | 0.5 | 0.5 | 0.5 |
| Technical writer / trainer | 0 | 0 | 0–0.5 | 0–0.5 | 1–1.5 | 0 | 0 |
| Delivery / release manager | 0–0.5 | 0.5 | 0.5 | 0.5 | 0.5 | 0–0.5 | 0–0.5 |
| **Total FTE** | **16–24** | **23–34** | **20–30** | **21–29** | **16–25** | **13–21** | **10–17** |

Phase 1 (remaining): 4–7 FTE (product owner, architect, 2 BAs, UX lead, compliance coordinator, partner manager) for 1–3 months.

---

## 13. Calendar and cost estimate

### 13.1 Calendar (months, from planning-gate sign-off) — confidence **low**

| Phase | Duration range | Likely | Overlap allowed | Driver of length |
|---|---|---|---|---|
| 1 (remaining) | 1–3 | 2 | — | Decisions D-002, D-010, D-035; pack review |
| 2 | 6–9 | 7.5 | — | Ledger/invariant foundations; team ramp-up |
| 3 | 7–10 | 8.5 | Phase 4 finance/HR teams start in last 1–3 months | Breadth (25 WPs), channel certification, hardware |
| 4 | 5–8 | 6.5 | Phase 5 PSP work starts on D-024 signature | GL/AP/payroll correctness; adviser sign-offs |
| 5 | 4–7 | 5.5 | — | PSP certification, government adapters, partner contracts |
| 6 | 4–6 | 5 | Hypercare 1–1.5 months included | Site pilots, two close rehearsals, pen test, migration |
| **Release 1 (2–6)** | **~24–36** | **~28–30** | net of overlaps (sequential sum 26–40) | |
| 7 | 5–8 | 6.5 | after R1 stabilisation | Named vendor certifications |
| 8 | 4–7 | 5.5 | after P7 gate | Legal reviews; licensed payout partner |

"Six phases" is **not** six weeks, six sprints or six months.

### 13.2 Effort (person-months) — sums of `08` work packages ×1.3 programme overhead

| Phase | Low | Likely | High | Confidence |
|---|---|---|---|---|
| 1 (remaining) | 4 | 10 | 20 | medium |
| 2 | 90 | 135 | 207 | low |
| 3 | 145 | 214 | 324 | low |
| 4 | 104 | 154 | 237 | low |
| 5 | 69 | 103 | 163 | low (partner-driven) |
| 6 | 50 | 78 | 121 | low |
| **Release 1 (2–6)** | **458** | **684** | **1,053** | **low (−30 % / +50 %)** |
| 7 | 55 | 83 | 131 | very low |
| 8 | 41 | 62 | 96 | very low |

### 13.3 Cost model (no invented prices)

**Labour cost = person-months × blended loaded monthly rate (BR).** BR depends on sourcing location and mix and is set by D-035; it is not assumed here. Example formula: R1 labour = 458–1,053 PM × BR (likely 684 × BR).

| Planning assumption (PA) — to be validated | Value | Validate by |
|---|---|---|
| PA-01 Programme overhead on WP effort | +25–35 % (plan uses 30 %) | Actuals after Phase 2 |
| PA-02 Contingency on labour | 20–30 % held by sponsor, not allocated to teams | D-035 |
| PA-03 Non-labour cost as share of labour | Unknown — build bottom-up from quotes per category below; do not use a percentage until at least three vendor quotes exist | Procurement quotes in Phase 1–2 |
| PA-04 On-prem single-hotel server sizing (pilot ≤ 300 rooms) | App/DB: 2 nodes each 16–32 vCPU, 64–128 GB RAM, 1–4 TB NVMe (RAID) + offsite encrypted backup; optional GPU node for local AI | WP-6.3 load test |
| PA-05 Site edge node (LPR/gate/scale connectors) | 1 small industrial PC per site cluster | WP-3.10 pilot |

**Non-labour cost categories (priced only from real quotes):**

| Category | Items |
|---|---|
| Cloud/SaaS hosting | Kubernetes, managed Postgres or VMs, object storage, CDN, backups, monitoring, per region per D-034 |
| Software licences/subscriptions | Identity (if not self-hosted), observability, error tracking, CI minutes, EAS/app build service, design tools, test devices farm |
| Payment | PSP setup/certification fees, per-transaction fees, terminals, PCI assessment (QSA/SAQ support) |
| Distribution | Channel manager subscription/certification fees, metasearch/advertising spend (hotel-borne) |
| Banking | WPS/bank file channel fees, payout fees |
| Bill-pay | Khedmah/ONEIC commercial terms (if any) |
| Messaging | SMS per message by country, WhatsApp Business conversation fees, sender-ID registration |
| Identity & signature | OCR/ID verification licences or self-hosted compute; e-signature provider fees |
| AI | GPU hardware or hosted inference, model licence review, evaluation tooling |
| Hardware (per pilot site) | LPR cameras/server, gate controllers, barcode/QR scanners, scales, temperature probes, staff mobile devices, kiosk (Later), network equipment |
| Travel providers | Aggregator/API access fees, accreditation costs if D-011 = licensed |
| Government/compliance | Certification of invoicing software where mandated, digital certificates, filing credentials |
| Legal & advisory | Local counsel ×5 markets, tax advisers, payroll advisers (Canada/QC, Oman), privacy counsel/DPIA, trademark clearance |
| Security assurance | External penetration tests (web/API/mobile), cyber insurance |
| App distribution | Apple/Google developer programmes, MDM (if private distribution) |
| Pilot operations | Training, hypercare travel, data migration support |

---

## 14. Deployment profiles summary (detail in `docs/03`)

| Aspect | `saas` | `onprem-single-hotel` |
|---|---|---|
| Tenancy | Multi-tenant; row-level `tenant_id`/`property_id` + RLS | Single tenant, same schema and code |
| Runtime | Kubernetes; managed or self-run Postgres 16 with PITR; S3-compatible storage; Keycloak | Docker Compose or k3s on hotel servers (PA-04); MinIO; Keycloak |
| Updates | Continuous delivery with feature flags; per-tenant migration waves | Signed release bundles; maintenance windows; rollback bundle |
| Backups/DR | Cross-zone replicas, PITR, cross-region encrypted backup; restore drills | Local snapshots + **offsite encrypted backup** (to SaaS region or hotel-chosen storage); restore drills |
| Offline | Site edge node + staff-app offline queues cover internet loss for front desk/HK/POS/gate within stated limits | Core runs locally; internet loss affects only partner integrations (queued via outbox) |
| Integrations | Partner adapters in cloud; device connectors on site edge | Adapters on-prem; outbound allow-list to partners |
| AI | Hosted or self-hosted inference per D-038 | Local model on optional GPU node; classical fallback |
| Data residency | Region chosen per D-034 and market rules | On hotel premises; offsite backup location per D-034 |
| Support | Metrikingdom operated SRE | Hotel IT + remote support with privileged-access audit |

Both profiles must pass the same acceptance suite (AT-G01…G20) and restore drill before R1 for whichever profile the pilot adopts.
