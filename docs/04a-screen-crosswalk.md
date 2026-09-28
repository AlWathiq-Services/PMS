# 04a — Screen Crosswalk (catalogue references → canonical screen ids)

**Pack:** MetriStay Hospitality Suite Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Companion to:** `docs/04-screens-and-design.md` (canonical screen ids, column legend §1.2, codes §2) • **Resolves:** screen references in `docs/01-catalogue/*.md` that are not defined in `docs/04`.
**Status:** design target. No screen exists.

> This file closes the consistency gap between the six catalogue files and `docs/04`. The catalogue used 1120 screen references with placeholder prefixes (FD, STF, ADMIN, REV, OWN, GUEST, MGMT, MKT, VND, COMP, INV, SALES, HK, POS, WEB, SAFETY, CONTENT, CAT, EVT, DEV, CLUB, PARK, INT, HUB, KIT, EMP, PROC, STR, RCV, …) or non-canonical names under canonical prefixes. Every one of them is resolved below: **1039** as aliases of existing `docs/04` screens and **81** onto **37** new canonical screens defined in §2.

---

## 1. Rules

1. **Catalogue references resolve through this crosswalk.** A `screens:` entry in `docs/01-catalogue/*.md` that is not a `docs/04` id is read as the canonical id given for it in §3. The catalogue text is not rewritten; this file is the authoritative resolution.
2. **Canonical ids are authoritative for implementation.** Routes, components, permissions, tests (`AT-*`, `AC-*`), analytics events and traceability (`docs/13` §3) use only canonical ids: those in `docs/04` §4 plus the new screens in §2 of this file. A catalogue alias never becomes a route or a component name.
3. **Alias = same purpose.** An `alias` row means the catalogue subfeature is served by the named canonical screen; where the note names a tab, panel, mode, step or filter, that element lives inside the canonical screen and inherits its actor, permission, audit, notification, accessibility and mobile attributes unless the subfeature adds a stricter rule.
4. **New = canonical from now on.** §2 screens carry the same eleven attributes as `docs/04` §4 (legend `docs/04` §1.2, codes `docs/04` §2). They are to be merged into the `docs/04` app tables and counts (§12.1) at the next pack revision; until then this file is their source of definition. Screens for M34–M37 use the ids reserved in `docs/04` §12.2 (`SCR-ADM-uc-*`, `SCR-ADM-hsia-*`, `SCR-ADM-iptv-*`, `SCR-GST-marketplace-*`) or clearly marked Later equivalents and stay `release: Later`.
5. **Phase 2 slices must use canonical ids.** Every Phase 2+ work package, story, OpenAPI tag and test fixture cites the canonical id; the Phase 2 doc-lint (`docs/04` §12.1 note) fails a build that introduces a screen id defined neither in `docs/04` nor in §2 of this file, and warns on any use of a catalogue alias outside `docs/01-catalogue`.
6. **Adding screens.** A new catalogue need is first checked against `docs/04` and this file; only when no canonical screen reasonably covers the purpose is a new id added here (or in `docs/04`) with all eleven attributes, and then referenced.

---

## 2. New canonical screens

37 new screens. Column legend and codes as `docs/04` §1.2 and §2. The last line under each table lists the catalogue references that resolve to each screen.

### 2.1 GM — owner, GM, revenue and sales office

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-GM-composite-holds | Hotel-side list and detail of composite holds (rooms + space + F&B + parking + club) with firm expiry, extension, option ranking and conversion to booking (M09/M10/M12) | sales_manager, catering_manager | hold id, account, components per resource, expiry, option rank, extension requests, displacement value, linked RFQ/proposal | held, extension_requested, extended, converted, released, expired | EL-L; expiring holds pinned with countdown | com.hold.read@property, com.hold.extend, com.hold.convert `+limit` | AU-W | N-in, N-esc before expiry; N-email corporate on extension decision | A-grid, A-time | ML-D |
| SCR-GM-promotions | Promotions and promo codes: eligibility, exclusions, stacking, capacity caps, best-offer selection (M04/M54) | revenue_manager | promo code, linked rate plans, stay/booking windows, eligibility, exclusions, stacking rule, cap/used, jurisdiction review status | draft, pending_approval, active, paused, expired | EL-L; EL-F in editor | inv.promo.write; activation `+mc` above discount limit | AU-W | N-in on cap reached | A-grid, A-form | ML-D |

- `SCR-GM-composite-holds` ← `SCR-SALES-composite-hold`, `SCR-SALES-hold-list`
- `SCR-GM-promotions` ← `SCR-REV-promo-rules`, `SCR-REV-promotions`

### 2.2 FO — front desk and housekeeping

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-FO-keys | Issue, duplicate and revoke room keys and digital keys via authorised lock adapter; entitlement events on check-in/move/checkout (M05/M55) | front_desk_agent, front_office_manager | room, key type (card/mobile), count, validity, adapter status, entitlement events | issued, active, revoked, failed, adapter_unavailable | EL-M (lock adapter command); unavailable adapter → manual key log | res.key.issue, res.key.revoke | AU-W | N-push guest on digital key issue/revoke | A-form, A-live | ML-T |
| SCR-FO-registration-reporting | Guest registration reporting to authorities per JUR rule pack: batch, submit via connector or manual portal, receipts (M05/M38/M41) | front_office_manager, night_auditor, compliance_officer | business date, guests due, minimum data set, route (API/file/portal/manual), submission ref, receipt, rejections | pending, submitted, acknowledged, rejected, manual_evidence | EL-M; route `blocked` shows manual path | res.registration_report.submit `+stepup` | AU-P + AU-E | N-esc when not submitted by JUR deadline | A-grid | ML-D |
| SCR-FO-wake-up-calls | Schedule wake-up calls and track execution, acknowledgement and human escalation (M34, Later) | front_desk_agent, guest (via GST request) | room, guest, time, recurrence, language, attempt log, acknowledgement | scheduled, attempting, acknowledged, failed, escalated, cancelled | EL-L; PBX adapter down → manual call list | res.wakeup.write | AU-W | N-esc on unanswered call; N-in desk | A-grid, A-live | ML-C |

- `SCR-FO-keys` ← `SCR-FD-keys`, `SCR-STF-key-status`
- `SCR-FO-registration-reporting` ← `SCR-FD-registration-reporting`
- `SCR-FO-wake-up-calls` ← `SCR-FD-wake-up-exceptions`, `SCR-FD-wake-up-list`

### 2.3 FIN — finance

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-FIN-deposits | Deposit schedule and liability ledger: deposits due, receipts, application at check-in/event, forfeiture (M08/M10/M12) | cashier, ar_clerk, finance_clerk | reservation/event/account, due date, amount/currency, received, applied, forfeited, balance | scheduled, due, received, applied, forfeited, refunded | EL-L; EL-M for refund/forfeit | fin.deposit.read, fin.deposit.apply; forfeit/refund `+mc` over limit | AU-L | N-email/N-wa payer reminders; N-esc overdue | A-grid | ML-D |
| SCR-FIN-channel-commissions | Channel/OTA commission rules and accrual, commission invoice matching, channel payment models and virtual cards (M07) | finance_clerk, ap_clerk | channel, rule version, booking ref, accrued commission, invoiced amount, variance, VCC status | accrued, invoiced, matched, disputed, paid | EL-L | fin.channel_commission.read, .match | AU-L | N-in on variance above tolerance | A-grid | ML-D |
| SCR-FIN-tips-commissions | Tips and service charge distribution pools and amenity/practitioner commissions with payroll hand-off (M13/M58) | finance_clerk, payroll_officer, fnb_manager | outlet/amenity, period, pool total, distribution rule version, eligible staff (no pay rates shown), commission basis | open, calculated, approved, sent_to_payroll | EL-L; EL-M on approval | fin.tip_pool.calculate, .approve `+mc` | AU-L | N-in payroll on approval | A-grid | ML-D |
| SCR-FIN-marketplace-payouts | Marketplace orders, merchant-of-record split and supplier payouts via licensed provider; disputes, fraud and refunds (M37, Later) | finance_approver, payment_releaser | order, supplier, gross, fees, supplier share, provider payout ref, dispute status | pending, eligible, paid, held, disputed, refunded | EL-M | fin.marketplace.payout `+mc` `+stepup` | AU-P + AU-L | N-email supplier remittance | A-grid | ML-D |

- `SCR-FIN-deposits` ← `SCR-FIN-deposit-ledger`, `SCR-FIN-deposits-due`, `SCR-SALES-payment-schedule`
- `SCR-FIN-channel-commissions` ← `SCR-FIN-channel-payments`, `SCR-FIN-commission-match`, `SCR-FIN-commissions`
- `SCR-FIN-tips-commissions` ← `SCR-FIN-amenity-commissions`, `SCR-FIN-tip-pool`, `SCR-POS-tips`
- `SCR-FIN-marketplace-payouts` ← `SCR-FIN-marketplace-payouts`, `SCR-MKT-disputes`

### 2.4 HR — HR and employee self-service

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-HR-ess-profile | My profile: contract summary, role, work location, contacts; bank details masked with change request routed to SCR-FIN-payee-bank-change | employee | name, role, location, contacts, bank (masked), pending change requests | active, change_pending | EL-O | wrk.ess.profile@own; bank change `+stepup` | AU-W; bank view AU-R | N-push on change decision | A-form | ML-N |
| SCR-HR-ess-advance-request | Request salary advance or allowance within policy; see approval and payroll recovery schedule | employee | amount, reason, policy limit, recovery schedule, approval status | draft, requested, approved, rejected, recovered | EL-O | wrk.advance.request@own | AU-W | N-push decision | A-form | ML-N |

- `SCR-HR-ess-profile` ← `SCR-EMP-bank-details`, `SCR-EMP-profile`
- `SCR-HR-ess-advance-request` ← `SCR-EMP-advance-request`

### 2.5 CON — concierge, travel desk, transport

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-CON-transport-tariffs | Transport tariffs, gratuity rules and default payer for hotel fleet and contracted taxi/transfer (M59) | concierge, fnb_manager, finance_clerk | route/zone, vehicle type, tariff version, gratuity rule, payer default, tax category | draft, active, expired | EL-F | trv.tariff.write | AU-W | — | A-form | ML-D |

- `SCR-CON-transport-tariffs` ← `SCR-ADM-transport-tariffs`

### 2.6 FNB — bar, kitchen, club, catering, events, amenities

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-FNB-club-events | Club events and ticketing: capacity, ticket types, entitlements, door list (M15) | club_host, fnb_manager | event, date, capacity (safety vs commercial), ticket types/prices, sold, entitlements, door list | draft, on_sale, sold_out, live, closed | EL-L | com.club_event.write | AU-W | N-email ticket holders on change | A-grid | ML-C |

- `SCR-FNB-club-events` ← `SCR-CLUB-events`

### 2.7 GST — guest website, guest app and Partner Hub

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-GST-profile | Guest profile: verified contacts, stay preferences and accessibility needs, saved payment methods (PSP tokens), notification settings, account and device security | guest | name, verified email/phone, preferences, accessibility needs, saved methods (masked), devices/sessions | active, unverified_contact | EL-F | gst.profile@own | AU-W | N-email on security change | A-form, A-auth | ML-P |
| SCR-GST-club | Guest club/lounge: reservations, events and tickets, membership enrolment/renewal, entitlements (M15) | guest | venue, date/session, party, minimum spend, ticket, membership plan, entitlement balance | available, booked, ticketed, member_active, expired | EL-P; EL-M for payment | gst.club@own | AU-W | N-email/N-push confirmations | A-form, A-time | ML-P |
| SCR-GST-transfer | Request airport/taxi transfer, see quote with tariff and gratuity, give traveler-data consent, approve offer and track live trip (M45/M59) | guest | pickup/drop-off, time, flight/ship no, pax, luggage, accessibility, quote, consent, driver/vehicle, live status | draft, quoted, approved, booked, en_route, completed, cancelled | EL-P; EL-M on paid booking | gst.transfer@own | AU-W | N-push/N-sms driver status | A-form, A-help, A-live | ML-P |
| SCR-GST-gift-voucher | Buy gift voucher on the website: amount/experience, recipient, message, delivery, PSP payment (M54) | guest, booker | voucher type, amount/currency, recipient, delivery date, terms, payment | draft, paid, delivered, redeemed_partial, expired | EL-P; EL-M on payment | public; purchase via PSP | AU-W | N-email recipient/purchaser | A-form, A-help | ML-P |
| SCR-GST-kiosk-check-in | Self-service kiosk check-in/out via authorised adapter: find booking, ID, registration, payment, key; staff assist call | guest | booking lookup, ID step, signature, payment guarantee, key dispense, assist request | idle, in_session, assist_requested, completed, failed | EL-P; device fault → staff-assisted path at desk | device realm + guest OTP | AU-W; ID steps AU-E | N-in desk on assist request | A-help, A-sig, A-cam, A-time | ML-K |
| SCR-GST-wifi-portal | Captive portal authentication for in-room/venue Wi-Fi and premium tier purchase with folio posting (M35, Later) | guest | room/surname or code, plan, device count, premium tier price, folio posting | unauthenticated, active, upgraded, expired | EL-P | gst.wifi@own | AU-W | — | A-auth, A-form | ML-P |
| SCR-GST-iptv-home | In-room TV guest services: folio view, service requests, purchase confirmation and phone cast pairing isolated per stay (M36, Later) | guest | stay, folio summary, services menu, purchase item/price, cast pairing code | idle, paired, purchase_pending, posted | EL-P | gst.tv@room-stay | AU-W | — | A-media, A-std (remote-control navigation) | ML-K |
| SCR-GST-marketplace-search | OTA-type discovery and search across marketplace suppliers with dynamic packages and honest total price (M37, Later) | guest | destination, dates, party, supplier, package components, total price, merchant of record | search, results, quoted | EL-P | public | AU-0 | — | A-form, A-help | ML-P |

- `SCR-GST-profile` ← `SCR-GST-account`, `SCR-GST-my-preferences`, `SCR-GST-payment-methods`, `SCR-GST-preferences`, `SCR-GST-profile`, `SCR-GST-settings`, `SCR-GUEST-account-security`
- `SCR-GST-club` ← `SCR-GST-club-booking`, `SCR-GST-club-events`, `SCR-GST-membership`
- `SCR-GST-transfer` ← `SCR-GST-transfer-quote`, `SCR-GST-transfer-request`, `SCR-GST-transfer-status`, `SCR-GST-travel-consent`
- `SCR-GST-gift-voucher` ← `SCR-WEB-gift-voucher`
- `SCR-GST-kiosk-check-in` ← `SCR-KSK-check-in`
- `SCR-GST-wifi-portal` ← `SCR-GUEST-wifi-portal`, `SCR-GUEST-wifi-upgrade`
- `SCR-GST-iptv-home` ← `SCR-GUEST-tv-cast-pairing`, `SCR-TV-guest-services`, `SCR-TV-purchase-confirm`
- `SCR-GST-marketplace-search` ← `SCR-MKT-search`

### 2.8 CORP — corporate web portal

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-CORP-company-profile | Company profile: legal entities and branches, billing profile (credit account, PO rule), SSO/MFA settings, registered devices with remote revoke | corporate_admin | legal entities, branches, tax ids, billing profile, IdP metadata, device list | active, pending_verification | EL-F | corp.company.write `+stepup` for SSO/device revoke | AU-W | N-email admins on SSO change | A-form | ML-D |

- `SCR-CORP-company-profile` ← `SCR-CORP-billing-profile`, `SCR-CORP-company-profile`, `SCR-CORP-devices`, `SCR-CORP-sso-setup`

### 2.9 PRK — parking and security

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-PRK-lpr-settings | LPR confidence thresholds, plate normalisation and matching rules per lane/zone | property_admin, security_officer | lane, auto-open threshold, review threshold, region rules, matching window, anti-passback | draft, active | EL-F | com.lpr_settings.write `+mc` | AU-P | N-in security on change | A-form | ML-D |
| SCR-PRK-pay-station | Pay-on-exit station: plate/ticket lookup, tariff, pay via PSP or room charge, exit authorisation | guest, parking_attendant (assist) | plate/ticket, session duration, tariff, amount, payment method, room link | idle, amount_due, paying, paid, failed, assist | EL-M; device fault → manual gate SCR-PRK-manual-gate | device realm | AU-L | N-in attendant on assist | A-help, A-time | ML-K |
| SCR-PRK-valet-queue | Valet requests and vehicle movements: request, park, retrieve, handover with ticket and plate | parking_attendant | ticket, plate, guest/room, location slot, requested time, status, damage photos | requested, parked, retrieving, ready, handed_over | EL-R | com.valet.write | AU-W | N-sms/N-push guest when ready | A-live, A-grid | ML-C |

- `SCR-PRK-lpr-settings` ← `SCR-ADM-lpr-thresholds`
- `SCR-PRK-pay-station` ← `SCR-PARK-pay-station`
- `SCR-PRK-valet-queue` ← `SCR-PARK-valet-queue`

### 2.10 ADM — integrations, platform and regulatory administration

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-ADM-config-versions | Versioned configuration change sets with diff, maker-checker approval, scheduled effect and rollback | property_admin, tenant_admin | change set, entity, before/after diff, maker, checker, effective time, rollback link | draft, pending_approval, applied, rolled_back, rejected | EL-F | plt.config.change `+mc` | AU-P | N-in approver | A-grid | ML-D |
| SCR-ADM-sso-connections | SSO connections (OIDC/SAML) for staff and corporate realms: metadata, claim mapping, MFA policy, test login | it_admin, tenant_admin | realm, protocol, IdP metadata, certificate expiry, claim → role mapping, MFA policy | draft, testing, active, disabled | EL-F | iam.sso.write `+stepup` `+mc` | AU-P | N-email on certificate expiry | A-form | ML-D |
| SCR-ADM-channel-mapping | Channel room/rate mapping and rate distribution scope per channel | revenue_manager, integration_admin | channel, channel room/rate codes, hotel room type/rate plan, scope, unmapped items | unmapped, mapped, invalid | EL-L | dst.mapping.write | AU-W | N-in on unmapped inbound booking | A-grid | ML-D |
| SCR-ADM-observability | Platform health: metrics and SLOs, alert routing, logs/traces with PII redaction, store-and-forward queue depth, time sync, tenant isolation tests, on-prem profile | it_admin | SLO, error budget, alerts, queue depth, clock drift, isolation test result, profile (saas/onprem) | healthy, degraded, breached | EL-D, EL-R | plt.observability.read | AU-0 | N-push/N-esc on SLO breach | A-chart, A-live | ML-D |
| SCR-ADM-developer-portal | Public self-service developer portal: API/SDK docs, sandbox tenant with synthetic data, certification kit and status | integration_admin, external developer | API docs versions, SDKs, sandbox credentials, certification checklist, status | registered, sandbox, certifying, certified | EL-P | public; sandbox `dev.sandbox@own` | AU-W | N-email certification result | A-std | ML-D |
| SCR-ADM-uc-extensions | PBX/UC extension registry with adapter binding, class of service and PMS status sync log (M34, Later) | it_admin, integration_admin | extension, room, class of service, adapter, last sync event, errors | active, out_of_sync, disabled | EL-L | plt.uc.write | AU-W | N-in on sync failure | A-grid | ML-D |
| SCR-ADM-uc-emergency | Emergency calling configuration and local survivability tests for UC (M34, Later) | it_admin, security_officer | emergency number routing, location data, survivable gateway, last test | configured, test_due, failed | EL-F | plt.uc.emergency `+mc` | AU-P | N-esc on failed test | A-form | ML-D |
| SCR-ADM-hsia-plans | HSIA plans and entitlements (guest/staff/corporate/event) with device and session policies (M35, Later) | it_admin | plan, bandwidth, device limit, price/entitlement, audience, session policy | draft, active, retired | EL-F | plt.hsia.write | AU-W | — | A-form | ML-D |
| SCR-ADM-hsia-sessions | HSIA sessions: room-move/checkout revocation, billable usage reconciliation, network telemetry and capacity (M35, Later) | it_admin, finance_clerk | session, room/stay, device, plan, usage, folio posting, AP load/capacity | active, revoked, expired, posted | EL-R | plt.hsia.read | AU-W | N-in on revocation failure | A-grid, A-chart | ML-D |
| SCR-ADM-iptv-content | Licensed IPTV content and EPG with licence expiry (M36, Later) | it_admin | channel/content, licence, territory, expiry, EPG source | active, expiring, expired | EL-L | plt.iptv.write | AU-W | N-in on licence expiry | A-grid, A-media | ML-D |
| SCR-ADM-iptv-devices | IPTV room/device pairing, fleet health and firmware, checkout reset log (M36, Later) | it_admin, engineer | device, room, firmware, last heartbeat, pairing, last reset | online, offline, reset_pending, reset_done | EL-R | plt.iptv.devices | AU-W | N-in on reset failure | A-grid | ML-D |

- `SCR-ADM-config-versions` ← `SCR-ADM-config-diff`, `SCR-ADM-config-history`
- `SCR-ADM-sso-connections` ← `SCR-ADM-sso-connections`
- `SCR-ADM-channel-mapping` ← `SCR-INT-channel-mapping`
- `SCR-ADM-observability` ← `SCR-OPS-health`, `SCR-OPS-isolation-test-report`, `SCR-OPS-observability`, `SCR-OPS-onprem-health`, `SCR-OPS-queue-depth`, `SCR-OPS-time-sync-health`
- `SCR-ADM-developer-portal` ← `SCR-DEV-certification`, `SCR-DEV-docs`, `SCR-DEV-public-portal`, `SCR-DEV-sandbox`
- `SCR-ADM-uc-extensions` ← `SCR-ADMIN-pbx-extensions`, `SCR-ADMIN-uc-sync-log`
- `SCR-ADM-uc-emergency` ← `SCR-ADMIN-uc-emergency-config`
- `SCR-ADM-hsia-plans` ← `SCR-ADMIN-hsia-plans`, `SCR-ADMIN-hsia-policies`
- `SCR-ADM-hsia-sessions` ← `SCR-ADMIN-hsia-sessions`, `SCR-ADMIN-hsia-telemetry`, `SCR-FIN-hsia-reconciliation`
- `SCR-ADM-iptv-content` ← `SCR-ADMIN-iptv-content`
- `SCR-ADM-iptv-devices` ← `SCR-ADMIN-iptv-devices`, `SCR-ADMIN-iptv-fleet`, `SCR-ADMIN-iptv-reset-log`

### 2.11 MED — property content, website and marketing

| ID | Purpose | Actor | Key data fields | States | E/L/E | Perm | Audit | Notif | A11y | Mobile |
|---|---|---|---|---|---|---|---|---|---|---|
| SCR-MED-marketplace-listings | Review marketplace supplier listings: content, rights, pricing, category, publication (M37, Later) | marketing_manager, content_approver | supplier, listing, media, terms, price, category, rights status | submitted, in_review, approved, rejected, withdrawn | EL-L | med.marketplace.review | AU-W | N-email supplier decision | A-grid | ML-D |

- `SCR-MED-marketplace-listings` ← `SCR-MKT-listing-review`

### 2.12 New screens per application

| App | New screens |
|---|---|
| GM | 2 |
| FO | 3 |
| FIN | 4 |
| HR | 2 |
| CON | 1 |
| FNB | 1 |
| GST | 8 |
| CORP | 1 |
| PRK | 3 |
| ADM | 11 |
| MED | 1 |
| **Total** | **37** |

---

## 3. Alias table

1120 rows, sorted by catalogue reference. `alias` = existing `docs/04` screen; `new` = screen defined in §2. Subfeatures that cite each reference are listed in the last column (up to six; ids are `docs/01-catalogue` full ids).

| catalogue ref | resolution (alias/new) | canonical id | tab/panel or note |
|---|---|---|---|
| SCR-ADM-activation-profile | alias | SCR-ADM-property-profile | activation profile section — used by: M01.F01.1.SF01.1.4 |
| SCR-ADM-amenity-activation | alias | SCR-FNB-amenity-config | amenity enablement tab — used by: M58.F58.1.SF58.1.1 |
| SCR-ADM-amenity-activation-checklist | alias | SCR-ADM-activation-gates | amenity gate and pilot evidence — used by: M58.F58.1.SF58.1.5 |
| SCR-ADM-amenity-capacity | alias | SCR-FNB-amenity-config | resources and capacity tab — used by: M58.F58.1.SF58.1.2 |
| SCR-ADM-amenity-rates | alias | SCR-FNB-amenity-config | rates tab (membership/day-pass/guest/corporate) — used by: M58.F58.1.SF58.1.3 |
| SCR-ADM-analytics-settings | alias | SCR-ADM-consent-purposes | cookie and analytics purposes tab — used by: M51.F51.2.SF51.2.4 |
| SCR-ADM-app-releases | alias | SCR-ADM-app-distribution | same purpose — used by: M11.F11.1.SF11.1.2,M18.F18.1.SF18.1.3 |
| SCR-ADM-approval-policies | alias | SCR-ADM-roles-permissions | approval policies and limits tab — used by: M02.F02.4.SF02.4.1 |
| SCR-ADM-attribution-models | alias | SCR-ADM-kpi-dictionary | profitability/attribution model versions tab — used by: M65.F65.2.SF65.2.7 |
| SCR-ADM-bill-providers | alias | SCR-ADM-connectors | bill-pay provider connectors and allowed billers — used by: M29.F29.1.SF29.1.1,M29.F29.2.SF29.2.1,M29.F29.3.SF29.3.1 |
| SCR-ADM-bot-rules | alias | SCR-MED-attribution | bot/duplicate filtering rules panel — used by: M51.F51.2.SF51.2.6 |
| SCR-ADM-buildings-floors | alias | SCR-ADM-room-setup | buildings, floors and zones tab — used by: M03.F03.1.SF03.1.1 |
| SCR-ADM-certifications | alias | SCR-HR-certifications | same purpose — used by: M62.F62.1.SF62.1.2 |
| SCR-ADM-changeover-matrix | alias | SCR-ADM-timed-resource-setup | buffers and changeover tab — used by: M09.F09.1.SF09.1.4 |
| SCR-ADM-club-capacity | alias | SCR-ADM-timed-resource-setup | club zones: safety vs commercial capacity — used by: M15.F15.1.SF15.1.2 |
| SCR-ADM-club-venue | alias | SCR-ADM-outlet-setup | club/lounge outlet type — used by: M15.F15.1.SF15.1.1 |
| SCR-ADM-config-diff | new | SCR-ADM-config-versions | diff view — used by: M01.F01.3.SF01.3.4 |
| SCR-ADM-config-history | new | SCR-ADM-config-versions | version history tab — used by: M01.F01.3.SF01.3.4 |
| SCR-ADM-connector-permissions | alias | SCR-ADM-connector-detail | network zone and permission tab — used by: M64.F64.1.SF64.1.2 |
| SCR-ADM-content-approval-queue | alias | SCR-MED-approval | same purpose — used by: M51.F51.1.SF51.1.5 |
| SCR-ADM-corporate-accounts | alias | SCR-GM-corporate-account-detail | users tab (corporate identity) — used by: M02.F02.1.SF02.1.3 |
| SCR-ADM-data-export | alias | SCR-ADM-tenant-properties | data export and offboarding tab — used by: M01.F01.4.SF01.4.5 |
| SCR-ADM-data-freshness | alias | SCR-GM-data-coverage | same purpose — used by: M65.F65.1.SF65.1.5 |
| SCR-ADM-delegations | alias | SCR-ADM-users | delegations tab (time-bounded) — used by: M02.F02.4.SF02.4.2 |
| SCR-ADM-destination-content | alias | SCR-MED-website-pages | destination and amenity content sections — used by: M51.F51.1.SF51.1.6 |
| SCR-ADM-dsr-queue | alias | SCR-ADM-dsr-requests | same purpose — used by: M02.F02.5.SF02.5.6,M02.F02.5.SF02.5.7,M52.F52.1.SF52.1.8 |
| SCR-ADM-emission-factors | alias | SCR-ENG-sustainability-baseline | emission factors tab — used by: M67.F67.1.SF67.1.5 |
| SCR-ADM-equipment-pools | alias | SCR-ADM-timed-resource-setup | equipment and labour pools tab — used by: M09.F09.1.SF09.1.6 |
| SCR-ADM-external-access | alias | SCR-SAF-claim-file | adjuster/vendor access scope panel — used by: M68.F68.1.SF68.1.4 |
| SCR-ADM-field-policies | alias | SCR-ADM-roles-permissions | field policies tab — used by: M02.F02.3.SF02.3.3 |
| SCR-ADM-gated-permissions | alias | SCR-HR-certifications | revocation-on-lapse rules panel — used by: M62.F62.1.SF62.1.5 |
| SCR-ADM-happy-hour | alias | SCR-FNB-happy-hour-pricing | same purpose — used by: M13.F13.1.SF13.1.3 |
| SCR-ADM-inspection-templates | alias | SCR-SAF-inspections | checklist templates tab — used by: M61.F61.1.SF61.1.1 |
| SCR-ADM-integrations | alias | SCR-ADM-connectors | same purpose — used by: M22.F22.1.SF22.1.2,M27.F27.5.SF27.5.2,M28.F28.1.SF28.1.5,M29.F29.2.SF29.2.2,M29.F29.2.SF29.2.3,M29.F29.2.SF29.2.4 |
| SCR-ADM-inventory-change | alias | SCR-ADM-room-setup | effective-dated inventory changes tab — used by: M03.F03.1.SF03.1.5,M03.F03.3.SF03.3.3 |
| SCR-ADM-key-management | alias | SCR-ADM-secrets-certs | payroll encryption keys — used by: M27.F27.4.SF27.4.3 |
| SCR-ADM-lpr-devices | alias | SCR-ADM-devices | LPR device filter and adapter binding — used by: M17.F17.2.SF17.2.1 |
| SCR-ADM-lpr-thresholds | new | SCR-PRK-lpr-settings | defined in §2 — used by: M17.F17.2.SF17.2.3 |
| SCR-ADM-master-data-quality | alias | SCR-ADM-data-quality | same purpose — used by: M65.F65.1.SF65.1.1 |
| SCR-ADM-membership-plans | alias | SCR-FNB-club-memberships | plans and benefits tab — used by: M15.F15.2.SF15.2.1 |
| SCR-ADM-menu-editor | alias | SCR-FNB-menu-editor | same purpose — used by: M13.F13.1.SF13.1.1 |
| SCR-ADM-metric-dictionary | alias | SCR-ADM-kpi-dictionary | same purpose — used by: M65.F65.1.SF65.1.2 |
| SCR-ADM-modifier-editor | alias | SCR-FNB-menu-editor | modifiers and combos tab — used by: M13.F13.1.SF13.1.2 |
| SCR-ADM-notification-rules | alias | SCR-ADM-notification-templates | routing and quiet-hours tab — used by: M63.F63.1.SF63.1.6 |
| SCR-ADM-on-call | alias | SCR-SAF-on-call | same purpose — used by: M63.F63.1.SF63.1.6 |
| SCR-ADM-outlet-tax | alias | SCR-ADM-outlet-setup | tax and service charge tab — used by: M13.F13.1.SF13.1.4 |
| SCR-ADM-outlets | alias | SCR-ADM-departments | outlets and cost-center links tab — used by: M01.F01.1.SF01.1.3 |
| SCR-ADM-overbooking-policy | alias | SCR-GM-overbooking-control | same purpose — used by: M09.F09.4.SF09.4.4 |
| SCR-ADM-parity-matrix | alias | SCR-ADM-app-distribution | core-journey parity tab — used by: M11.F11.1.SF11.1.5 |
| SCR-ADM-parking-facility | alias | SCR-ADM-timed-resource-setup | parking zones, lanes and capacity tab — used by: M17.F17.1.SF17.1.1 |
| SCR-ADM-parking-tariffs | alias | SCR-PRK-tariffs | same purpose — used by: M17.F17.1.SF17.1.4 |
| SCR-ADM-policies | alias | SCR-ADM-rate-plan-setup | policies tab (multilingual fields) — used by: M01.F01.5.SF01.5.2 |
| SCR-ADM-policy-content | alias | SCR-MED-website-pages | location, accessibility and policy content — used by: M51.F51.1.SF51.1.3 |
| SCR-ADM-practitioners | alias | SCR-FNB-amenity-config | practitioners and sanitation tab — used by: M58.F58.2.SF58.2.3 |
| SCR-ADM-privacy-notices | alias | SCR-ADM-consent-purposes | notice versions and processing register tab — used by: M02.F02.5.SF02.5.8 |
| SCR-ADM-property-changes | alias | SCR-ADM-property-profile | configuration change requests — used by: M66.F66.2.SF66.2.4 |
| SCR-ADM-property-setup-checklist | alias | SCR-ADM-tenant-properties | property setup checklist — used by: M01.F01.1.SF01.1.2 |
| SCR-ADM-rate-plans | alias | SCR-ADM-rate-plan-setup | same purpose — used by: M01.F01.5.SF01.5.2 |
| SCR-ADM-record-restrictions | alias | SCR-ADM-roles-permissions | record restrictions tab (VIP/incognito/sensitive) — used by: M02.F02.3.SF02.3.4 |
| SCR-ADM-redirects | alias | SCR-MED-seo-structured-data | redirects tab — used by: M51.F51.1.SF51.1.4 |
| SCR-ADM-request-catalogue | alias | SCR-ADM-workflow-templates | service request catalogue tab — used by: M18.F18.3.SF18.3.1 |
| SCR-ADM-resource-calendar | alias | SCR-ADM-timed-resource-setup | operating calendars and blackouts tab — used by: M09.F09.1.SF09.1.5 |
| SCR-ADM-resource-detail | alias | SCR-ADM-timed-resource-setup | resource detail — used by: M09.F09.1.SF09.1.1 |
| SCR-ADM-resource-list | alias | SCR-ADM-timed-resource-setup | resource list — used by: M09.F09.1.SF09.1.1 |
| SCR-ADM-responsible-persons | alias | SCR-SAF-permit-calendar | responsible persons tab — used by: M61.F61.1.SF61.1.6 |
| SCR-ADM-retention-map | alias | SCR-ADM-retention-policies | same purpose — used by: M65.F65.2.SF65.2.6 |
| SCR-ADM-role-training-matrix | alias | SCR-HR-training-matrix | same purpose — used by: M62.F62.1.SF62.1.1 |
| SCR-ADM-roles | alias | SCR-ADM-roles-permissions | same purpose — used by: M02.F02.3.SF02.3.1 |
| SCR-ADM-room-connections | alias | SCR-ADM-room-setup | connecting rooms and composed suites tab — used by: M03.F03.1.SF03.1.4 |
| SCR-ADM-room-types | alias | SCR-ADM-room-setup | room types tab — used by: M01.F01.5.SF01.5.2,M03.F03.1.SF03.1.2 |
| SCR-ADM-rooms | alias | SCR-ADM-room-setup | rooms tab — used by: M03.F03.1.SF03.1.3 |
| SCR-ADM-seo-metadata | alias | SCR-MED-seo-structured-data | same purpose — used by: M51.F51.1.SF51.1.4 |
| SCR-ADM-sod-rules | alias | SCR-ADM-roles-permissions | segregation-of-duties rules tab — used by: M02.F02.3.SF02.3.5 |
| SCR-ADM-sod-violations | alias | SCR-ADM-access-review | SoD violations tab — used by: M02.F02.3.SF02.3.5 |
| SCR-ADM-sop-library | alias | SCR-HR-sop-library | same purpose — used by: M62.F62.1.SF62.1.6 |
| SCR-ADM-space-combination-editor | alias | SCR-ADM-timed-resource-setup | partitions and combinations tab — used by: M09.F09.1.SF09.1.2 |
| SCR-ADM-space-layouts | alias | SCR-ADM-timed-resource-setup | layouts and capacities tab — used by: M09.F09.1.SF09.1.3 |
| SCR-ADM-sso-connections | new | SCR-ADM-sso-connections | defined in §2 — used by: M02.F02.2.SF02.2.1 |
| SCR-ADM-status-page | alias | SCR-ADM-deployment-status | status page and support ownership panel — used by: M64.F64.2.SF64.2.6 |
| SCR-ADM-suppression-list | alias | SCR-ADM-consent-purposes | suppression list tab — used by: M02.F02.5.SF02.5.3 |
| SCR-ADM-takedown | alias | SCR-MED-takedown | same purpose — used by: M51.F51.1.SF51.1.5 |
| SCR-ADM-tenant-list | alias | SCR-ADM-tenant-properties | same purpose — used by: M01.F01.1.SF01.1.1 |
| SCR-ADM-tenant-setup | alias | SCR-ADM-tenant-properties | tenant provisioning — used by: M01.F01.1.SF01.1.1 |
| SCR-ADM-translations | alias | SCR-ADM-localization | same purpose — used by: M01.F01.5.SF01.5.1 |
| SCR-ADM-transport-privacy | alias | SCR-ADM-retention-policies | transport record class (data minimisation) — used by: M59.F59.2.SF59.2.2 |
| SCR-ADM-transport-tariffs | new | SCR-CON-transport-tariffs | defined in §2 — used by: M59.F59.2.SF59.2.3 |
| SCR-ADM-travel-market-gate | alias | SCR-CON-provider-eligibility | market gate (referrer vs licensed seller) — used by: M45.F45.3.SF45.3.1,M45.F45.3.SF45.3.2 |
| SCR-ADM-travel-provider-capabilities | alias | SCR-CON-provider-eligibility | provider capability matrix — used by: M45.F45.2.SF45.2.2,M45.F45.3.SF45.3.3,M45.F45.3.SF45.3.4 |
| SCR-ADM-user-detail | alias | SCR-ADM-users | user detail (lifecycle, scoped roles) — used by: M02.F02.1.SF02.1.1,M02.F02.3.SF02.3.2 |
| SCR-ADM-vendor-users | alias | SCR-VEN-team-users | same purpose — used by: M02.F02.1.SF02.1.4 |
| SCR-ADM-web-performance | alias | SCR-MED-seo-structured-data | performance panel — used by: M51.F51.1.SF51.1.4 |
| SCR-ADM-website-page-editor | alias | SCR-MED-website-pages | same purpose — used by: M51.F51.1.SF51.1.2 |
| SCR-ADM-website-settings | alias | SCR-MED-website-pages | site settings/domain tab — used by: M51.F51.1.SF51.1.1 |
| SCR-ADM-website-theme | alias | SCR-MED-website-pages | theme tab — used by: M51.F51.1.SF51.1.1 |
| SCR-ADM-workflow-audit | alias | SCR-ADM-workflow-templates | audit and performance tab — used by: M63.F63.2.SF63.2.5 |
| SCR-ADM-workflow-data-policy | alias | SCR-ADM-workflow-templates | data minimisation policy tab — used by: M63.F63.2.SF63.2.3 |
| SCR-ADM-workflow-editor | alias | SCR-ADM-workflow-templates | same purpose — used by: M63.F63.1.SF63.1.1,M63.F63.2.SF63.2.4 |
| SCR-ADM-workflow-permissions | alias | SCR-ADM-workflow-templates | configuration permissions tab — used by: M63.F63.2.SF63.2.1 |
| SCR-ADM-workflow-runs | alias | SCR-ADM-job-monitor | workflow runs filter (retry/dedup/ack) — used by: M63.F63.1.SF63.1.3 |
| SCR-ADMIN-ai-evals | alias | SCR-AI-evaluation | staff AI actions evaluation set — used by: M37.F37.2.SF37.2.3 |
| SCR-ADMIN-ai-tools | alias | SCR-AI-settings | tool permission registry tab (supervised staff AI actions) — used by: M37.F37.2.SF37.2.1 |
| SCR-ADMIN-assistant-channels | alias | SCR-AI-settings | channels, throttling and outage fallback tab — used by: M40.F40.2.SF40.2.7 |
| SCR-ADMIN-assistant-evals | alias | SCR-AI-evaluation | same purpose — used by: M40.F40.1.SF40.1.5 |
| SCR-ADMIN-assistant-retention | alias | SCR-ADM-retention-policies | AI transcript record class — used by: M40.F40.2.SF40.2.4 |
| SCR-ADMIN-assistant-tools | alias | SCR-AI-settings | tools and PII masking tab — used by: M40.F40.2.SF40.2.5 |
| SCR-ADMIN-connector-credentials | alias | SCR-ADM-connector-detail | credentials and certificates (write-only) — used by: M38.F38.3.SF38.3.2 |
| SCR-ADMIN-connector-health | alias | SCR-ADM-integration-health | same purpose — used by: M38.F38.3.SF38.3.4 |
| SCR-ADMIN-connector-security | alias | SCR-ADM-gov-connectors | authentication/security tab — used by: M38.F38.3.SF38.3.3 |
| SCR-ADMIN-enhancement-capacity | alias | SCR-MED-enhancement-queue | capacity and compute cost panel — used by: M39.F39.2.SF39.2.5 |
| SCR-ADMIN-enhancement-models | alias | SCR-MED-enhancement-queue | model/licence registry tab — used by: M39.F39.2.SF39.2.1 |
| SCR-ADMIN-feature-gates | alias | SCR-ADM-activation-gates | wallet and referral gates — used by: M30.F30.2.SF30.2.6,M31.F31.1.SF31.1.7,M44.F44.2.SF44.2.6 |
| SCR-ADMIN-hsia-plans | new | SCR-ADM-hsia-plans | defined in §2 — used by: M35.F35.1.SF35.1.2 |
| SCR-ADMIN-hsia-policies | new | SCR-ADM-hsia-plans | device and session policies tab — used by: M35.F35.1.SF35.1.3 |
| SCR-ADMIN-hsia-sessions | new | SCR-ADM-hsia-sessions | defined in §2 — used by: M35.F35.2.SF35.2.1 |
| SCR-ADMIN-hsia-telemetry | new | SCR-ADM-hsia-sessions | network telemetry and capacity tab — used by: M35.F35.2.SF35.2.2 |
| SCR-ADMIN-id-retention | alias | SCR-ADM-retention-policies | ID imagery record class — used by: M41.F41.1.SF41.1.7 |
| SCR-ADMIN-integration-health | alias | SCR-ADM-integration-health | same purpose — used by: M33.F33.2.SF33.2.3,M33.F33.2.SF33.2.4 |
| SCR-ADMIN-iptv-content | new | SCR-ADM-iptv-content | defined in §2 — used by: M36.F36.1.SF36.1.2 |
| SCR-ADMIN-iptv-devices | new | SCR-ADM-iptv-devices | defined in §2 — used by: M36.F36.1.SF36.1.1 |
| SCR-ADMIN-iptv-fleet | new | SCR-ADM-iptv-devices | fleet health and firmware tab — used by: M36.F36.2.SF36.2.3 |
| SCR-ADMIN-iptv-reset-log | new | SCR-ADM-iptv-devices | checkout reset log tab — used by: M36.F36.1.SF36.1.4 |
| SCR-ADMIN-loyalty-adjustment | alias | SCR-ADM-loyalty-program | fraud rules and manual adjustment review tab — used by: M30.F30.2.SF30.2.3 |
| SCR-ADMIN-loyalty-campaigns | alias | SCR-ADM-loyalty-program | promotions tab — used by: M30.F30.1.SF30.1.2 |
| SCR-ADMIN-loyalty-earn-rates | alias | SCR-ADM-loyalty-program | earn rules tab — used by: M30.F30.1.SF30.1.2 |
| SCR-ADMIN-loyalty-eligibility-matrix | alias | SCR-ADM-loyalty-program | eligibility matrix tab — used by: M30.F30.1.SF30.1.1 |
| SCR-ADMIN-loyalty-expiry-policy | alias | SCR-ADM-loyalty-program | expiry tab — used by: M30.F30.1.SF30.1.4 |
| SCR-ADMIN-loyalty-integrity | alias | SCR-FIN-loyalty-liability | ledger integrity checks panel — used by: M30.F30.2.SF30.2.1 |
| SCR-ADMIN-loyalty-program | alias | SCR-ADM-loyalty-program | same purpose — used by: M30.F30.1.SF30.1.1 |
| SCR-ADMIN-loyalty-tiers | alias | SCR-ADM-loyalty-program | tiers tab — used by: M30.F30.1.SF30.1.2 |
| SCR-ADMIN-partner-incentives | alias | SCR-ADM-referral-program | direct partner incentives tab (single-tier) — used by: M37.F37.3.SF37.3.1 |
| SCR-ADMIN-pbx-extensions | new | SCR-ADM-uc-extensions | defined in §2 — used by: M34.F34.1.SF34.1.1 |
| SCR-ADMIN-playbooks | alias | SCR-SAF-playbooks | same purpose — used by: M42.F42.2.SF42.2.2 |
| SCR-ADMIN-property-setup | alias | SCR-ADM-legal-entities | property legal entity and jurisdiction mapping — used by: M38.F38.1.SF38.1.1 |
| SCR-ADMIN-referral-attribution-disputes | alias | SCR-ADM-referral-program | attribution disputes tab — used by: M31.F31.1.SF31.1.2 |
| SCR-ADMIN-referral-kill-switch | alias | SCR-ADM-referral-program | market kill switch — used by: M31.F31.2.SF31.2.7 |
| SCR-ADMIN-referral-policies | alias | SCR-ADM-referral-program | same purpose — used by: M31.F31.2.SF31.2.1 |
| SCR-ADMIN-referrer-approvals | alias | SCR-ADM-referral-program | referrer approvals tab — used by: M31.F31.1.SF31.1.1,M31.F31.2.SF31.2.2 |
| SCR-ADMIN-safety-integrations | alias | SCR-ADM-connectors | fire/BMS/camera adapter filter — used by: M42.F42.1.SF42.1.2 |
| SCR-ADMIN-uc-emergency-config | new | SCR-ADM-uc-emergency | defined in §2 — used by: M34.F34.2.SF34.2.3 |
| SCR-ADMIN-uc-sync-log | new | SCR-ADM-uc-extensions | PMS status sync log tab — used by: M34.F34.1.SF34.1.2 |
| SCR-AUTH-mfa-enrol | alias | SCR-OPS-login | MFA enrolment step — used by: M02.F02.2.SF02.2.2 |
| SCR-AUTH-recover | alias | SCR-OPS-login | account recovery flow — used by: M02.F02.2.SF02.2.4 |
| SCR-AUTH-step-up | alias | SCR-OPS-step-up | same purpose — used by: M02.F02.2.SF02.2.2 |
| SCR-CAPP-about | alias | SCR-CAPP-home | about/version panel — used by: M11.F11.1.SF11.1.2 |
| SCR-CAPP-analytics | alias | SCR-CORP-reports | web hand-off from CAPP — used by: M11.F11.5.SF11.5.3 |
| SCR-CAPP-book-stay | alias | SCR-CORP-room-search | web hand-off from CAPP — used by: M11.F11.2.SF11.2.4 |
| SCR-CAPP-confirm | alias | SCR-CAPP-booking-confirm | same purpose — used by: M11.F11.3.SF11.3.4 |
| SCR-CAPP-facility-search | alias | SCR-CAPP-event-search | same purpose — used by: M09.F09.2.SF09.2.1,M11.F11.2.SF11.2.1 |
| SCR-CAPP-login | alias | SCR-CAPP-sign-in | same purpose — used by: M11.F11.1.SF11.1.3 |
| SCR-CAPP-passes | alias | SCR-CAPP-parking-passes | parking and club passes — used by: M11.F11.4.SF11.4.4 |
| SCR-CAPP-pay-invoice | alias | SCR-CAPP-invoices | pay action — used by: M11.F11.5.SF11.5.2 |
| SCR-CAPP-quote | alias | SCR-CAPP-compare | selected-option quote with expiry — used by: M11.F11.3.SF11.3.1 |
| SCR-CAPP-quote-hold | alias | SCR-CAPP-hold | same purpose — used by: M09.F09.3.SF09.3.1,M11.F11.3.SF11.3.2 |
| SCR-CAPP-rfq-new | alias | SCR-CORP-rfq-request | web hand-off from CAPP — used by: M10.F10.4.SF10.4.2 |
| SCR-CAPP-settings | alias | SCR-CAPP-notifications | settings: preferences and device registration — used by: M11.F11.1.SF11.1.4 |
| SCR-CAPP-user-admin | alias | SCR-CORP-users-roles | web hand-off from CAPP — used by: M10.F10.1.SF10.1.2 |
| SCR-CAT-actuals | alias | SCR-FNB-event-actuals-settlement | catering actuals — used by: M16.F16.5.SF16.5.1 |
| SCR-CAT-allergen-matrix | alias | SCR-FNB-allergen-matrix | catering filter — used by: M16.F16.1.SF16.1.3 |
| SCR-CAT-dispatch | alias | SCR-FNB-catering-dispatch | same purpose — used by: M16.F16.4.SF16.4.2 |
| SCR-CAT-guarantees | alias | SCR-FNB-catering-order | guarantees and cutoffs tab — used by: M16.F16.2.SF16.2.2 |
| SCR-CAT-kitchen-capacity | alias | SCR-FNB-production-plan | production slot capacity panel — used by: M09.F09.2.SF09.2.4 |
| SCR-CAT-logistics | alias | SCR-FNB-catering-dispatch | staff, equipment and vehicle schedule tab — used by: M16.F16.4.SF16.4.1 |
| SCR-CAT-offsite-settings | alias | SCR-ADM-outlet-setup | catering outlet off-site eligibility and radius — used by: M16.F16.1.SF16.1.4 |
| SCR-CAT-order-detail | alias | SCR-FNB-catering-order | same purpose — used by: M16.F16.2.SF16.2.1 |
| SCR-CAT-package-editor | alias | SCR-FNB-menu-editor | catering menus and packages — used by: M16.F16.1.SF16.1.1 |
| SCR-CAT-pricing | alias | SCR-FNB-menu-editor | tiered and per-cover pricing tab — used by: M16.F16.1.SF16.1.2 |
| SCR-CAT-production-plan | alias | SCR-FNB-production-plan | same purpose — used by: M12.F12.3.SF12.3.3,M16.F16.2.SF16.2.3,M16.F16.3.SF16.3.1,M16.F16.3.SF16.3.4 |
| SCR-CAT-returns | alias | SCR-FNB-catering-dispatch | returns tab — used by: M16.F16.4.SF16.4.4 |
| SCR-CAT-shortages | alias | SCR-PRC-requisition-new | raised from catering shortage — used by: M16.F16.3.SF16.3.3 |
| SCR-CAT-special-meals | alias | SCR-FNB-catering-order | allergen flags and special meals tab — used by: M16.F16.2.SF16.2.4 |
| SCR-CLUB-billing | alias | SCR-FNB-club-memberships | billing destinations tab — used by: M15.F15.4.SF15.4.3 |
| SCR-CLUB-door | alias | SCR-FNB-club-entry | same purpose — used by: M15.F15.3.SF15.3.3,M15.F15.3.SF15.3.4,M15.F15.3.SF15.3.5 |
| SCR-CLUB-enroll | alias | SCR-FNB-club-memberships | enrolment and payment — used by: M15.F15.2.SF15.2.2 |
| SCR-CLUB-entitlements | alias | SCR-FNB-club-memberships | guest and corporate entitlements tab — used by: M15.F15.2.SF15.2.4 |
| SCR-CLUB-events | new | SCR-FNB-club-events | defined in §2 — used by: M15.F15.4.SF15.4.4 |
| SCR-CLUB-floor | alias | SCR-FNB-pos-tables | club floor/seating view — used by: M09.F09.2.SF09.2.2,M15.F15.1.SF15.1.3 |
| SCR-CLUB-member-detail | alias | SCR-FNB-club-memberships | member detail (renew/freeze/cancel) — used by: M15.F15.2.SF15.2.3 |
| SCR-CLUB-min-spend | alias | SCR-FNB-club-reservations | minimum spend panel — used by: M15.F15.4.SF15.4.1 |
| SCR-CLUB-passes | alias | SCR-FNB-club-memberships | passes and credentials tab — used by: M15.F15.3.SF15.3.2 |
| SCR-CLUB-reservations | alias | SCR-FNB-club-reservations | same purpose — used by: M15.F15.3.SF15.3.1 |
| SCR-CLUB-sessions | alias | SCR-FNB-club-reservations | sessions and seating tab — used by: M15.F15.1.SF15.1.3 |
| SCR-COMP-adviser-signoffs | alias | SCR-ADM-rule-version-review | adviser sign-off panel — used by: M38.F38.1.SF38.1.6 |
| SCR-COMP-change-alerts | alias | SCR-ADM-rule-packs | change alerts tab — used by: M44.F44.2.SF44.2.7 |
| SCR-COMP-classification-explorer | alias | SCR-ADM-classifier-test | same purpose — used by: M44.F44.1.SF44.1.2,M44.F44.1.SF44.1.3 |
| SCR-COMP-conflicts | alias | SCR-ADM-rule-version-review | precedence/conflict panel — used by: M44.F44.1.SF44.1.4 |
| SCR-COMP-coverage-dashboard | alias | SCR-ADM-coverage-dashboard | same purpose — used by: M44.F44.1.SF44.1.6,M44.F44.2.SF44.2.8 |
| SCR-COMP-evidence | alias | SCR-ADM-rule-version-review | sources and evidence tab — used by: M44.F44.2.SF44.2.3 |
| SCR-COMP-exceptions | alias | SCR-ADM-activation-gates | exception approvals tab — used by: M44.F44.2.SF44.2.6 |
| SCR-COMP-filing-audit | alias | SCR-ADM-filing-submission | audit trail tab — used by: M38.F38.3.SF38.3.7 |
| SCR-COMP-government-connectors | alias | SCR-ADM-gov-connectors | same purpose — used by: M38.F38.3.SF38.3.1 |
| SCR-COMP-government-routes | alias | SCR-ADM-gov-connectors | route (API/file/portal/manual) tab — used by: M44.F44.2.SF44.2.5 |
| SCR-COMP-jurisdiction-nodes | alias | SCR-ADM-jurisdiction-tree | same purpose — used by: M44.F44.1.SF44.1.5 |
| SCR-COMP-jurisdiction-profile | alias | SCR-ADM-legal-entities | jurisdiction profile tab — used by: M44.F44.1.SF44.1.1 |
| SCR-COMP-legal-entity-jurisdiction | alias | SCR-ADM-legal-entities | same purpose — used by: M38.F38.1.SF38.1.1 |
| SCR-COMP-obligations | alias | SCR-ADM-filing-calendar | obligations register tab — used by: M44.F44.2.SF44.2.2 |
| SCR-COMP-product-tax-matrix | alias | SCR-ADM-tax-config | same purpose — used by: M38.F38.1.SF38.1.3 |
| SCR-COMP-referral-audit-trail | alias | SCR-ADM-referral-program | compliance audit tab — used by: M31.F31.2.SF31.2.7 |
| SCR-COMP-referral-market-gate | alias | SCR-ADM-activation-gates | referral market gate — used by: M31.F31.1.SF31.1.7 |
| SCR-COMP-referral-market-matrix | alias | SCR-ADM-referral-program | market matrix tab — used by: M31.F31.2.SF31.2.1 |
| SCR-COMP-referral-structure-audit | alias | SCR-ADM-referral-program | structure audit (no downline) tab — used by: M31.F31.2.SF31.2.6 |
| SCR-COMP-rule-pack-version | alias | SCR-ADM-rule-version-review | same purpose — used by: M44.F44.2.SF44.2.4 |
| SCR-COMP-rule-packs | alias | SCR-ADM-rule-packs | same purpose — used by: M44.F44.2.SF44.2.1 |
| SCR-COMP-signature-requirements | alias | SCR-ADM-rule-packs | signature requirement category — used by: M41.F41.2.SF41.2.2 |
| SCR-COMP-tax-rules | alias | SCR-ADM-tax-config | effective dates and exemption evidence tab — used by: M38.F38.1.SF38.1.2 |
| SCR-COMP-wallet-gate-evidence | alias | SCR-ADM-activation-gates | wallet/PSP gate evidence — used by: M30.F30.2.SF30.2.6 |
| SCR-CON-disruption-queue | alias | SCR-CON-disruption-refund | same purpose — used by: M45.F45.2.SF45.2.7,M45.F45.2.SF45.2.8,M45.F45.2.SF45.2.9 |
| SCR-CON-travel-desk | alias | SCR-CON-request-inbox | same purpose — used by: M45.F45.1.SF45.1.1 |
| SCR-CON-travel-request-form | alias | SCR-CON-request-new | same purpose — used by: M45.F45.1.SF45.1.1,M45.F45.1.SF45.1.2,M45.F45.1.SF45.1.3,M45.F45.1.SF45.1.4,M45.F45.1.SF45.1.5,M45.F45.1.SF45.1.6 |
| SCR-CONTENT-approval-queue | alias | SCR-MED-approval | same purpose — used by: M39.F39.1.SF39.1.5,M39.F39.2.SF39.2.4 |
| SCR-CONTENT-before-after | alias | SCR-MED-enhance-compare | same purpose — used by: M39.F39.2.SF39.2.3 |
| SCR-CONTENT-derivatives | alias | SCR-MED-asset-detail | derivatives tab — used by: M39.F39.1.SF39.1.4 |
| SCR-CONTENT-enhance | alias | SCR-MED-enhancement-queue | same purpose — used by: M39.F39.2.SF39.2.2 |
| SCR-CONTENT-job-history | alias | SCR-MED-enhancement-queue | job history and provenance tab — used by: M39.F39.2.SF39.2.6 |
| SCR-CONTENT-kb-articles | alias | SCR-MED-kb-articles | same purpose — used by: M40.F40.1.SF40.1.1 |
| SCR-CONTENT-kb-review-queue | alias | SCR-MED-kb-articles | review-due queue — used by: M40.F40.1.SF40.1.1 |
| SCR-CONTENT-media-library | alias | SCR-MED-library | same purpose — used by: M39.F39.1.SF39.1.1 |
| SCR-CONTENT-media-tagging | alias | SCR-MED-asset-detail | tags, alt text and captions tab — used by: M39.F39.1.SF39.1.3 |
| SCR-CONTENT-preview | alias | SCR-MED-listing-preview | same purpose — used by: M39.F39.1.SF39.1.6 |
| SCR-CONTENT-rights | alias | SCR-MED-asset-detail | rights tab — used by: M39.F39.1.SF39.1.2 |
| SCR-CONTENT-syndication | alias | SCR-MED-publish-status | same purpose — used by: M39.F39.1.SF39.1.6 |
| SCR-CONTENT-takedowns | alias | SCR-MED-takedown | same purpose — used by: M39.F39.1.SF39.1.7 |
| SCR-CONTENT-upload | alias | SCR-MED-upload | same purpose — used by: M39.F39.1.SF39.1.1 |
| SCR-CONTENT-version-history | alias | SCR-MED-asset-detail | versions tab — used by: M39.F39.1.SF39.1.5 |
| SCR-CORP-analytics | alias | SCR-CORP-reports | same purpose — used by: M11.F11.5.SF11.5.3 |
| SCR-CORP-approval | alias | SCR-CORP-hold-detail | convert hold to confirmed allocations — used by: M09.F09.3.SF09.3.3 |
| SCR-CORP-approval-policy | alias | SCR-CORP-cost-centers | approval routing tab — used by: M10.F10.3.SF10.3.3 |
| SCR-CORP-approvals | alias | SCR-CORP-approval-queue | same purpose — used by: M10.F10.3.SF10.3.3,M11.F11.3.SF11.3.3,M28.F28.3.SF28.3.4 |
| SCR-CORP-billing-profile | new | SCR-CORP-company-profile | billing profile tab (credit account, PO rule) — used by: M20.F20.2.SF20.2.1 |
| SCR-CORP-block-status | alias | SCR-CORP-rooming-list | block pickup panel — used by: M32.F32.1.SF32.1.4 |
| SCR-CORP-book-stay | alias | SCR-CORP-room-search | same purpose — used by: M11.F11.2.SF11.2.4 |
| SCR-CORP-booking-detail | alias | SCR-CORP-hold-detail | booking detail after conversion (release/cancel) — used by: M09.F09.3.SF09.3.5 |
| SCR-CORP-budgets | alias | SCR-CORP-budget | same purpose — used by: M10.F10.3.SF10.3.4 |
| SCR-CORP-change-order | alias | SCR-CORP-change-request | same purpose — used by: M12.F12.3.SF12.3.4 |
| SCR-CORP-company-profile | new | SCR-CORP-company-profile | defined in §2 — used by: M10.F10.1.SF10.1.1 |
| SCR-CORP-compare | alias | SCR-CORP-event-compare | same purpose — used by: M11.F11.2.SF11.2.2 |
| SCR-CORP-confirm | alias | SCR-CORP-booking-confirm | same purpose — used by: M11.F11.3.SF11.3.4 |
| SCR-CORP-contract-sign | alias | SCR-CORP-agreement-view | contract versions and signature panel — used by: M10.F10.4.SF10.4.4 |
| SCR-CORP-devices | new | SCR-CORP-company-profile | devices tab (registration, remote revoke) — used by: M11.F11.1.SF11.1.4 |
| SCR-CORP-event-reconciliation | alias | SCR-CORP-invoices | post-event reconciliation statement — used by: M12.F12.5.SF12.5.3 |
| SCR-CORP-exports | alias | SCR-CORP-reports | exports tab — used by: M11.F11.5.SF11.5.4 |
| SCR-CORP-facility-search | alias | SCR-CORP-event-search | same purpose — used by: M09.F09.2.SF09.2.1,M11.F11.2.SF11.2.1 |
| SCR-CORP-guarantee | alias | SCR-CORP-beo-review | final guarantee and dietary aggregation panel — used by: M12.F12.3.SF12.3.5,M16.F16.2.SF16.2.2,M16.F16.2.SF16.2.4 |
| SCR-CORP-inbox | alias | SCR-CORP-home | portal shell inbox — used by: M11.F11.1.SF11.1.1 |
| SCR-CORP-passes | alias | SCR-CORP-parking-passes | parking and club passes — used by: M11.F11.4.SF11.4.4,M17.F17.1.SF17.1.2 |
| SCR-CORP-pay-invoice | alias | SCR-CORP-invoices | pay action — used by: M11.F11.5.SF11.5.2 |
| SCR-CORP-payments | alias | SCR-CORP-invoices | deposit and payment schedule tab — used by: M10.F10.3.SF10.3.5 |
| SCR-CORP-po-list | alias | SCR-CORP-cost-centers | purchase orders tab — used by: M10.F10.3.SF10.3.2 |
| SCR-CORP-quote | alias | SCR-CORP-package-builder | all-in quote summary — used by: M04.F04.4.SF04.4.5,M04.F04.5.SF04.5.1,M04.F04.5.SF04.5.3,M11.F11.3.SF11.3.1 |
| SCR-CORP-quote-hold | alias | SCR-CORP-hold-detail | same purpose — used by: M09.F09.3.SF09.3.1,M11.F11.3.SF11.3.2 |
| SCR-CORP-rfq-new | alias | SCR-CORP-rfq-request | same purpose — used by: M10.F10.4.SF10.4.2 |
| SCR-CORP-saved-searches | alias | SCR-CORP-event-search | saved searches and availability alerts panel — used by: M11.F11.2.SF11.2.3 |
| SCR-CORP-search | alias | SCR-CORP-room-search | same purpose — used by: M04.F04.1.SF04.1.4,M04.F04.5.SF04.5.7 |
| SCR-CORP-sso-setup | new | SCR-CORP-company-profile | SSO/MFA tab — used by: M11.F11.1.SF11.1.3 |
| SCR-CORP-travel-request | alias | SCR-CORP-travel-requests | same purpose — used by: M45.F45.1.SF45.1.1,M45.F45.1.SF45.1.3,M45.F45.1.SF45.1.6 |
| SCR-CORP-user-admin | alias | SCR-CORP-users-roles | same purpose — used by: M10.F10.1.SF10.1.2 |
| SCR-CORP-users | alias | SCR-CORP-users-roles | same purpose — used by: M02.F02.1.SF02.1.3 |
| SCR-DEV-access-reviews | alias | SCR-ADM-access-review | API client/partner scope — used by: M33.F33.3.SF33.3.4 |
| SCR-DEV-api-catalogue | alias | SCR-ADM-webhooks-api-keys | API catalogue and versions tab — used by: M33.F33.1.SF33.1.1 |
| SCR-DEV-certification | new | SCR-ADM-developer-portal | certification kit section — used by: M33.F33.3.SF33.3.3 |
| SCR-DEV-client-scopes | alias | SCR-ADM-webhooks-api-keys | client scopes — used by: M33.F33.1.SF33.1.2 |
| SCR-DEV-clients | alias | SCR-ADM-webhooks-api-keys | API clients tab — used by: M33.F33.1.SF33.1.2 |
| SCR-DEV-contract-diff | alias | SCR-ADM-webhooks-api-keys | contract diff view — used by: M33.F33.1.SF33.1.1 |
| SCR-DEV-dead-letters | alias | SCR-ADM-dead-letters | same purpose — used by: M33.F33.2.SF33.2.3 |
| SCR-DEV-deprecations | alias | SCR-ADM-webhooks-api-keys | versions and deprecations tab — used by: M33.F33.1.SF33.1.3 |
| SCR-DEV-docs | new | SCR-ADM-developer-portal | docs and SDK section — used by: M33.F33.3.SF33.3.2 |
| SCR-DEV-event-catalogue | alias | SCR-ADM-event-monitor | event catalogue/schema registry tab — used by: M33.F33.2.SF33.2.1 |
| SCR-DEV-public-portal | new | SCR-ADM-developer-portal | defined in §2 — used by: M33.F33.3.SF33.3.5 |
| SCR-DEV-rate-limits | alias | SCR-ADM-webhooks-api-keys | rate limits and quotas tab — used by: M33.F33.1.SF33.1.4 |
| SCR-DEV-sandbox | new | SCR-ADM-developer-portal | sandbox tenant section — used by: M33.F33.3.SF33.3.1 |
| SCR-DEV-webhooks | alias | SCR-ADM-webhooks-api-keys | webhook subscriptions tab — used by: M33.F33.2.SF33.2.2 |
| SCR-EMP-advance-request | new | SCR-HR-ess-advance-request | defined in §2 — used by: M27.F27.3.SF27.3.2 |
| SCR-EMP-bank-details | new | SCR-HR-ess-profile | bank details (masked; change via SCR-FIN-payee-bank-change) — used by: M27.F27.1.SF27.1.3 |
| SCR-EMP-leave | alias | SCR-HR-ess-leave | same purpose — used by: M27.F27.2.SF27.2.3 |
| SCR-EMP-payslip | alias | SCR-HR-ess-payslip | same purpose — used by: M27.F27.3.SF27.3.8,M27.F27.5.SF27.5.4 |
| SCR-EMP-profile | new | SCR-HR-ess-profile | defined in §2 — used by: M27.F27.1.SF27.1.1 |
| SCR-EMP-schedule | alias | SCR-HR-ess-schedule | same purpose — used by: M27.F27.2.SF27.2.1 |
| SCR-EMP-time | alias | SCR-HR-ess-clock | same purpose — used by: M27.F27.2.SF27.2.2 |
| SCR-ENG-asset-cost | alias | SCR-ENG-asset-detail | maintenance cost roll-up tab — used by: M26.F26.3.SF26.3.3 |
| SCR-ENG-backups | alias | SCR-ADM-backup-restore | same purpose — used by: M64.F64.2.SF64.2.1 |
| SCR-ENG-certificates | alias | SCR-ADM-secrets-certs | same purpose — used by: M64.F64.1.SF64.1.1 |
| SCR-ENG-change-requests | alias | SCR-ADM-deployment-status | change approvals and rollback — used by: M64.F64.2.SF64.2.5 |
| SCR-ENG-contractor-access | alias | SCR-ENG-contractor-assignments | site briefing and access panel — used by: M62.F62.1.SF62.1.4 |
| SCR-ENG-cyber-incident | alias | SCR-SAF-incident-command | cyber incident playbook (isolation, log preservation) — used by: M64.F64.2.SF64.2.3 |
| SCR-ENG-deployment-profile | alias | SCR-ADM-deployment-status | deployment profile and time sync panel — used by: M64.F64.1.SF64.1.6 |
| SCR-ENG-device-registry | alias | SCR-ADM-devices | same purpose — used by: M64.F64.1.SF64.1.1,M64.F64.1.SF64.1.7 |
| SCR-ENG-dispatch-board | alias | SCR-ENG-work-order-board | same purpose — used by: M26.F26.1.SF26.1.4 |
| SCR-ENG-dr-tests | alias | SCR-ADM-backup-restore | DR failover/failback tests tab — used by: M64.F64.2.SF64.2.7 |
| SCR-ENG-energy-monitor | alias | SCR-ENG-consumption-anomalies | same purpose — used by: M22.F22.4.SF22.4.1,M23.F23.1.SF23.1.4 |
| SCR-ENG-fallback-tests | alias | SCR-SAF-drills | manual fallback tests (gate/key/POS) — used by: M64.F64.1.SF64.1.5 |
| SCR-ENG-fleet-validity | alias | SCR-CON-vehicle-register | same purpose — used by: M59.F59.2.SF59.2.1 |
| SCR-ENG-gas-safety | alias | SCR-ENG-gas-pipeline | safety faults and connection certificates tab — used by: M24.F24.2.SF24.2.1,M24.F24.2.SF24.2.2 |
| SCR-ENG-health | alias | SCR-ADM-integration-health | degraded services queue — used by: M64.F64.1.SF64.1.3 |
| SCR-ENG-inspection | alias | SCR-ENG-room-release-inspection | same purpose — used by: M26.F26.1.SF26.1.6 |
| SCR-ENG-maintenance-windows | alias | SCR-ADM-deployment-status | maintenance windows tab — used by: M64.F64.1.SF64.1.4 |
| SCR-ENG-meter-map | alias | SCR-ENG-asset-meter-map | same purpose — used by: M22.F22.1.SF22.1.1,M24.F24.1.SF24.1.1 |
| SCR-ENG-meter-quality | alias | SCR-ENG-meter-readings | data quality score panel — used by: M67.F67.1.SF67.1.6 |
| SCR-ENG-network-zones | alias | SCR-ADM-devices | network zones tab — used by: M64.F64.1.SF64.1.2 |
| SCR-ENG-pm-calendar | alias | SCR-ENG-preventive-schedule | same purpose — used by: M26.F26.1.SF26.1.2 |
| SCR-ENG-recovery-objectives | alias | SCR-ADM-backup-restore | RPO/RTO measured vs target — used by: M64.F64.2.SF64.2.4 |
| SCR-ENG-resource-anomalies | alias | SCR-ENG-consumption-anomalies | same purpose — used by: M67.F67.2.SF67.2.2 |
| SCR-ENG-resource-data-status | alias | SCR-ENG-sustainability-baseline | data status (missing/estimated/verified) — used by: M67.F67.1.SF67.1.2 |
| SCR-ENG-resource-sources | alias | SCR-ENG-utility-accounts | utility and waste sources — used by: M67.F67.1.SF67.1.1 |
| SCR-ENG-restore-tests | alias | SCR-ADM-backup-restore | restore test evidence — used by: M64.F64.2.SF64.2.1 |
| SCR-ENG-site-visits | alias | SCR-ENG-contractor-assignments | site visits tab — used by: M26.F26.2.SF26.2.3 |
| SCR-ENG-support-desk | alias | SCR-ADM-deployment-status | support ownership panel — used by: M64.F64.2.SF64.2.6 |
| SCR-ENG-vendor-profile | alias | SCR-PRC-vendor-detail | same purpose — used by: M26.F26.2.SF26.2.1,M26.F26.2.SF26.2.7 |
| SCR-ENG-work-order | alias | SCR-ENG-work-order-detail | OOO-linked work order — used by: M03.F03.3.SF03.3.1,M03.F03.3.SF03.3.4,M06.F06.4.SF06.4.5,M55.F55.2.SF55.2.2 |
| SCR-ENG-work-orders | alias | SCR-ENG-work-order-board | same purpose — used by: M22.F22.4.SF22.4.2,M23.F23.1.SF23.1.4,M24.F24.1.SF24.1.6,M26.F26.1.SF26.1.3 |
| SCR-EVT-attendee-checkin | alias | SCR-FNB-event-detail | attendees and check-in tab — used by: M12.F12.4.SF12.4.1 |
| SCR-EVT-attrition | alias | SCR-GM-group-block | attrition, wash and penalties panel — used by: M12.F12.1.SF12.1.4 |
| SCR-EVT-beo-diff | alias | SCR-FNB-beo-editor | version compare view — used by: M12.F12.3.SF12.3.2 |
| SCR-EVT-beo-editor | alias | SCR-FNB-beo-editor | same purpose — used by: M12.F12.3.SF12.3.1,M12.F12.3.SF12.3.2 |
| SCR-EVT-billing-setup | alias | SCR-FNB-event-detail | billing tab (master account, routing, entitlements) — used by: M12.F12.5.SF12.5.1,M17.F17.4.SF17.4.3 |
| SCR-EVT-change-order | alias | SCR-FNB-event-change-order | same purpose — used by: M09.F09.3.SF09.3.4,M12.F12.3.SF12.3.4 |
| SCR-EVT-cutoff-queue | alias | SCR-GM-group-block | cutoff and auto-release queue — used by: M12.F12.1.SF12.1.3 |
| SCR-EVT-event-detail | alias | SCR-FNB-event-detail | same purpose — used by: M12.F12.2.SF12.2.1 |
| SCR-EVT-guarantee | alias | SCR-FNB-event-detail | final guarantee tab — used by: M12.F12.3.SF12.3.5 |
| SCR-EVT-pickup | alias | SCR-GM-group-block | pickup tracking — used by: M12.F12.1.SF12.1.2 |
| SCR-EVT-proforma | alias | SCR-FIN-ar-invoice | pro-forma and deposit application — used by: M12.F12.5.SF12.5.2 |
| SCR-EVT-reconciliation | alias | SCR-FNB-event-actuals-settlement | same purpose — used by: M12.F12.5.SF12.5.3,M16.F16.5.SF16.5.2 |
| SCR-EVT-resources | alias | SCR-FNB-event-detail | AV, equipment and staff resources tab — used by: M12.F12.2.SF12.2.2 |
| SCR-EVT-room-block | alias | SCR-GM-group-block | same purpose — used by: M12.F12.1.SF12.1.1,M12.F12.1.SF12.1.5 |
| SCR-FD-account-folios | alias | SCR-FO-folio | master, house and non-guest folio types — used by: M08.F08.1.SF08.1.6 |
| SCR-FD-arrivals | alias | SCR-FO-arrivals | same purpose — used by: M05.F05.4.SF05.4.1,M06.F06.1.SF06.1.1,M06.F06.2.SF06.2.5 |
| SCR-FD-assignment-risk | alias | SCR-FO-room-assignment | assignability and consistency check panel — used by: M03.F03.5.SF03.5.4 |
| SCR-FD-attention | alias | SCR-FO-home | attention inbox panel — used by: M06.F06.1.SF06.1.4 |
| SCR-FD-auto-assign | alias | SCR-FO-room-assignment | auto-assign action — used by: M03.F03.5.SF03.5.2 |
| SCR-FD-availability | alias | SCR-FO-availability-grid | same purpose — used by: M03.F03.2.SF03.2.2 |
| SCR-FD-business-date-banner | alias | SCR-FO-home | business-date banner (shell component on all FO screens) — used by: M01.F01.2.SF01.2.2 |
| SCR-FD-check-in | alias | SCR-FO-check-in | same purpose — used by: M05.F05.4.SF05.4.2,M05.F05.4.SF05.4.4,M05.F05.4.SF05.4.5,M08.F08.3.SF08.3.3 |
| SCR-FD-check-out | alias | SCR-FO-checkout | same purpose — used by: M05.F05.5.SF05.5.3,M08.F08.2.SF08.2.2,M08.F08.4.SF08.4.1 |
| SCR-FD-checkout | alias | SCR-FO-checkout | partial and multi-tender settlement step — used by: M28.F28.1.SF28.1.3,M28.F28.3.SF28.3.4 |
| SCR-FD-connectivity-banner | alias | SCR-OPS-outage-mode | connectivity banner component — used by: M01.F01.4.SF01.4.4 |
| SCR-FD-credit-monitor | alias | SCR-FO-deposit-preauth | credit limit and balance monitor tab — used by: M08.F08.2.SF08.2.4,M08.F08.3.SF08.3.3 |
| SCR-FD-day-use | alias | SCR-FO-reservation-new | day-use slot mode — used by: M09.F09.2.SF09.2.3 |
| SCR-FD-departures | alias | SCR-FO-departures | same purpose — used by: M05.F05.5.SF05.5.4,M06.F06.1.SF06.1.2 |
| SCR-FD-discrepancies | alias | SCR-FO-hk-room-board | FO vs HK discrepancy filter — used by: M06.F06.2.SF06.2.1,M06.F06.3.SF06.3.5 |
| SCR-FD-folio | alias | SCR-FO-folio | same purpose — used by: M01.F01.2.SF01.2.3,M04.F04.3.SF04.3.4,M05.F05.5.SF05.5.5,M06.F06.4.SF06.4.2,M07.F07.4.SF07.4.4,M08.F08.1.SF08.1.1 |
| SCR-FD-folio-payment | alias | SCR-FO-payment-take | same purpose — used by: M08.F08.3.SF08.3.4,M08.F08.4.SF08.4.3,M08.F08.5.SF08.5.4,M28.F28.1.SF28.1.1,M28.F28.1.SF28.1.2,M28.F28.1.SF28.1.3 |
| SCR-FD-folio-post | alias | SCR-FO-folio | post charge action — used by: M08.F08.1.SF08.1.2 |
| SCR-FD-folio-refund | alias | SCR-FO-payment-take | refund mode (approval via SCR-FIN-refund-approval) — used by: M08.F08.3.SF08.3.5,M08.F08.5.SF08.5.5,M30.F30.1.SF30.1.5 |
| SCR-FD-folio-tender-points | alias | SCR-FO-payment-take | points tender — used by: M30.F30.1.SF30.1.6 |
| SCR-FD-folio-transfer | alias | SCR-FO-folio-adjust | transfer between windows/folios/accounts — used by: M08.F08.1.SF08.1.4 |
| SCR-FD-guest-loyalty-panel | alias | SCR-FO-guest-profile | loyalty panel — used by: M30.F30.1.SF30.1.3,M30.F30.2.SF30.2.2 |
| SCR-FD-guest-messages | alias | SCR-FO-guest-inbox | message-waiting/DND panel (M34, Later) — used by: M34.F34.1.SF34.1.3 |
| SCR-FD-guest-profile | alias | SCR-FO-guest-profile | same purpose — used by: M02.F02.3.SF02.3.3,M05.F05.2.SF05.2.2,M06.F06.1.SF06.1.3,M18.F18.2.SF18.2.2 |
| SCR-FD-guest-search | alias | SCR-FO-guest-profile | match-and-create search — used by: M05.F05.2.SF05.2.2 |
| SCR-FD-handover | alias | SCR-OPS-shift-handover | same purpose — used by: M06.F06.1.SF06.1.5 |
| SCR-FD-id-mismatch-queue | alias | SCR-SAF-id-mismatch-review | same purpose — used by: M41.F41.1.SF41.1.5 |
| SCR-FD-id-scan | alias | SCR-SAF-id-intake | same purpose — used by: M41.F41.1.SF41.1.2 |
| SCR-FD-in-house | alias | SCR-FO-in-house | same purpose — used by: M06.F06.1.SF06.1.3 |
| SCR-FD-inbox | alias | SCR-FO-guest-inbox | same purpose — used by: M18.F18.4.SF18.4.1 |
| SCR-FD-keys | new | SCR-FO-keys | defined in §2 — used by: M05.F05.4.SF05.4.5 |
| SCR-FD-loyalty-dispute-queue | alias | SCR-FO-guest-case | loyalty points dispute case type — used by: M30.F30.2.SF30.2.4 |
| SCR-FD-manual-id-check | alias | SCR-SAF-id-mismatch-review | manual verification decision — used by: M41.F41.1.SF41.1.6 |
| SCR-FD-new-reservation | alias | SCR-FO-reservation-new | same purpose — used by: M03.F03.4.SF03.4.1,M04.F04.5.SF04.5.1,M05.F05.1.SF05.1.2 |
| SCR-FD-parking-permit | alias | SCR-FO-parking-permits | same purpose — used by: M17.F17.1.SF17.1.2,M17.F17.1.SF17.1.3 |
| SCR-FD-payment-link | alias | SCR-FO-payment-take | pay-by-link — used by: M28.F28.1.SF28.1.4 |
| SCR-FD-qr-handoff | alias | SCR-SAF-otp-qr-verify | QR handoff attempts — used by: M41.F41.2.SF41.2.6 |
| SCR-FD-refund | alias | SCR-FO-payment-take | refund mode — used by: M28.F28.1.SF28.1.6 |
| SCR-FD-registration | alias | SCR-SAF-id-intake | jurisdiction/document-type check — used by: M41.F41.1.SF41.1.1 |
| SCR-FD-registration-card | alias | SCR-FO-registration-card | same purpose — used by: M01.F01.5.SF01.5.5,M02.F02.5.SF02.5.2,M05.F05.4.SF05.4.3 |
| SCR-FD-registration-reporting | new | SCR-FO-registration-reporting | defined in §2 — used by: M05.F05.4.SF05.4.6 |
| SCR-FD-requests | alias | SCR-FO-service-requests | same purpose — used by: M18.F18.3.SF18.3.2 |
| SCR-FD-reservation | alias | SCR-FO-reservation-detail | same purpose — used by: M02.F02.3.SF02.3.4,M03.F03.5.SF03.5.1,M03.F03.5.SF03.5.3,M04.F04.4.SF04.4.1,M04.F04.5.SF04.5.2,M04.F04.5.SF04.5.6 |
| SCR-FD-reservation-amend | alias | SCR-FO-reservation-amend | same purpose — used by: M04.F04.5.SF04.5.5,M05.F05.3.SF05.3.1,M05.F05.5.SF05.5.2 |
| SCR-FD-reservation-guarantee | alias | SCR-FO-reservation-detail | guarantee, payer and deposit schedule tab — used by: M05.F05.2.SF05.2.3,M08.F08.3.SF08.3.1 |
| SCR-FD-reservation-history | alias | SCR-FO-reservation-detail | history tab — used by: M05.F05.5.SF05.5.7 |
| SCR-FD-reservation-parties | alias | SCR-FO-reservation-detail | parties tab — used by: M05.F05.2.SF05.2.1 |
| SCR-FD-room-grid | alias | SCR-FO-room-rack | same purpose — used by: M03.F03.1.SF03.1.3,M03.F03.3.SF03.3.1,M03.F03.3.SF03.3.2,M03.F03.5.SF03.5.1,M03.F03.5.SF03.5.5,M03.F03.5.SF03.5.6 |
| SCR-FD-room-move | alias | SCR-FO-room-move | same purpose — used by: M05.F05.5.SF05.5.1,M34.F34.1.SF34.1.2 |
| SCR-FD-room-status | alias | SCR-FO-hk-room-board | room status view — used by: M26.F26.1.SF26.1.5,M42.F42.2.SF42.2.5 |
| SCR-FD-routing | alias | SCR-FO-folio-routing | same purpose — used by: M08.F08.2.SF08.2.1,M08.F08.2.SF08.2.3 |
| SCR-FD-service-request | alias | SCR-FO-service-requests | fault/IoT/meter alert request type — used by: M26.F26.1.SF26.1.3 |
| SCR-FD-signature-record | alias | SCR-SAF-signature-record | same purpose — used by: M41.F41.2.SF41.2.3 |
| SCR-FD-tax-exemption-capture | alias | SCR-FO-reservation-detail | tax exemption evidence panel — used by: M38.F38.1.SF38.1.2 |
| SCR-FD-terminal | alias | SCR-FO-payment-take | card terminal — used by: M28.F28.1.SF28.1.4 |
| SCR-FD-traces | alias | SCR-FO-reservation-detail | traces tab — used by: M06.F06.1.SF06.1.6 |
| SCR-FD-unified-inbox | alias | SCR-AI-agent-console | same purpose — used by: M40.F40.2.SF40.2.3 |
| SCR-FD-user-switch | alias | SCR-OPS-login | fast user switch on shared terminal — used by: M02.F02.2.SF02.2.3 |
| SCR-FD-verification-fallback | alias | SCR-SAF-otp-qr-verify | manual in-person fallback — used by: M41.F41.2.SF41.2.7 |
| SCR-FD-waitlist | alias | SCR-FO-waitlist | same purpose — used by: M05.F05.3.SF05.3.5 |
| SCR-FD-wake-up-exceptions | new | SCR-FO-wake-up-calls | exceptions and escalation tab — used by: M34.F34.2.SF34.2.2 |
| SCR-FD-wake-up-list | new | SCR-FO-wake-up-calls | defined in §2 — used by: M34.F34.2.SF34.2.1 |
| SCR-FD-walk-in | alias | SCR-FO-reservation-new | walk-in mode (continues to SCR-FO-check-in) — used by: M05.F05.1.SF05.1.3 |
| SCR-FD-walk-planner | alias | SCR-GM-overbooking-control | walk plan — used by: M05.F05.3.SF05.3.4 |
| SCR-FIN-account-mapping | alias | SCR-FIN-posting-rules | account mapping — used by: M19.F19.1.SF19.1.2 |
| SCR-FIN-accruals | alias | SCR-FIN-accruals-prepayments | same purpose — used by: M19.F19.1.SF19.1.4,M20.F20.5.SF20.5.5,M22.F22.2.SF22.2.3 |
| SCR-FIN-allocation-policies | alias | SCR-FIN-allocation-rules | same purpose — used by: M32.F32.3.SF32.3.1 |
| SCR-FIN-allocation-policy | alias | SCR-FIN-allocation-rules | same purpose — used by: M19.F19.3.SF19.3.2,M24.F24.1.SF24.1.5 |
| SCR-FIN-allocation-run | alias | SCR-FIN-allocation-rules | allocation run and reversal tab — used by: M19.F19.3.SF19.3.3,M22.F22.2.SF22.2.3,M32.F32.3.SF32.3.1 |
| SCR-FIN-amenity-commissions | new | SCR-FIN-tips-commissions | amenity/practitioner commissions tab — used by: M58.F58.2.SF58.2.4 |
| SCR-FIN-ancillary-allocation | alias | SCR-FIN-posting-rules | package and ancillary allocation tab — used by: M54.F54.1.SF54.1.5 |
| SCR-FIN-ap-invoice-detail | alias | SCR-FIN-supplier-invoice-detail | same purpose — used by: M20.F20.1.SF20.1.2,M20.F20.1.SF20.1.3 |
| SCR-FIN-ap-invoice-inbox | alias | SCR-FIN-ap-inbox | same purpose — used by: M20.F20.1.SF20.1.2 |
| SCR-FIN-ap-match | alias | SCR-FIN-three-way-match | same purpose — used by: M47.F47.2.SF47.2.7,M49.F49.3.SF49.3.6,M49.F49.3.SF49.3.7,M50.F50.2.SF50.2.7 |
| SCR-FIN-ar-aging | alias | SCR-FIN-ar-aging-collections | same purpose — used by: M20.F20.2.SF20.2.3 |
| SCR-FIN-ar-ap-summary | alias | SCR-GM-cash-position | AR/AP summary panel — used by: M32.F32.4.SF32.4.2 |
| SCR-FIN-ar-transfer | alias | SCR-FIN-ar-accounts | transfer from folio to city ledger — used by: M08.F08.2.SF08.2.2 |
| SCR-FIN-bill-accounts | alias | SCR-ENG-utility-accounts | bill-pay account registration/validation — used by: M29.F29.1.SF29.1.2,M29.F29.3.SF29.3.1 |
| SCR-FIN-bill-disputes | alias | SCR-FIN-bill-payment-detail | dispute/reversal — used by: M29.F29.1.SF29.1.9 |
| SCR-FIN-bill-payment-confirm | alias | SCR-FIN-bill-payment-detail | confirm step — used by: M29.F29.1.SF29.1.5 |
| SCR-FIN-booking-margin-calculation | alias | SCR-FIN-commission-payouts | booking margin calculation panel — used by: M31.F31.1.SF31.1.4 |
| SCR-FIN-budget | alias | SCR-FIN-budget-editor | same purpose — used by: M20.F20.4.SF20.4.1,M20.F20.4.SF20.4.2 |
| SCR-FIN-budget-import | alias | SCR-FIN-budget-editor | import — used by: M32.F32.3.SF32.3.3 |
| SCR-FIN-budget-variance | alias | SCR-GM-budget-vs-actual | same purpose — used by: M20.F20.4.SF20.4.4 |
| SCR-FIN-business-date-reopen | alias | SCR-FO-night-audit | reopen business date (privileged) — used by: M08.F08.6.SF08.6.5 |
| SCR-FIN-call-charges | alias | SCR-FO-folio | call-accounting postings (M34, Later) — used by: M34.F34.1.SF34.1.4 |
| SCR-FIN-capitalization | alias | SCR-FIN-fixed-assets | same purpose — used by: M21.F21.2.SF21.2.4,M26.F26.3.SF26.3.4,M66.F66.2.SF66.2.3 |
| SCR-FIN-cash-drop | alias | SCR-FIN-cash-drop-safe | same purpose — used by: M60.F60.1.SF60.1.2 |
| SCR-FIN-cash-position | alias | SCR-GM-cash-position | same purpose — used by: M32.F32.4.SF32.4.2 |
| SCR-FIN-cashier-close | alias | SCR-FIN-cashier-shift | close with blind count — used by: M08.F08.5.SF08.5.3 |
| SCR-FIN-channel-payments | new | SCR-FIN-channel-commissions | channel payment models and virtual cards tab — used by: M07.F07.4.SF07.4.4 |
| SCR-FIN-chargebacks | alias | SCR-FIN-chargeback-case | same purpose — used by: M28.F28.1.SF28.1.6 |
| SCR-FIN-cogs | alias | SCR-FNB-theoretical-vs-actual | COGS posting tab — used by: M14.F14.5.SF14.5.5,M50.F50.3.SF50.3.7,M50.F50.4.SF50.4.3 |
| SCR-FIN-collections | alias | SCR-FIN-ar-aging-collections | same purpose — used by: M20.F20.2.SF20.2.3 |
| SCR-FIN-commission-approval | alias | SCR-FIN-commission-payouts | same purpose — used by: M31.F31.1.SF31.1.5 |
| SCR-FIN-commission-ledger | alias | SCR-FIN-commission-payouts | commission ledger tab — used by: M31.F31.1.SF31.1.5,M31.F31.2.SF31.2.4 |
| SCR-FIN-commission-match | new | SCR-FIN-channel-commissions | invoice match tab — used by: M07.F07.5.SF07.5.3,M60.F60.2.SF60.2.3 |
| SCR-FIN-commissions | new | SCR-FIN-channel-commissions | defined in §2 — used by: M07.F07.5.SF07.5.2 |
| SCR-FIN-comp-report | alias | SCR-FIN-revenue-protection | compensation/comp report — used by: M55.F55.2.SF55.2.3 |
| SCR-FIN-control-limits | alias | SCR-ADM-roles-permissions | void/comp/discount/refund limits tab — used by: M60.F60.2.SF60.2.7 |
| SCR-FIN-corrections | alias | SCR-FIN-journal-detail | reversing correction — used by: M60.F60.1.SF60.1.5 |
| SCR-FIN-cost-centers | alias | SCR-FIN-chart-of-accounts | cost centers tab — used by: M19.F19.1.SF19.1.1 |
| SCR-FIN-cost-status-dashboard | alias | SCR-FIN-cost-reconciliation | same purpose — used by: M19.F19.3.SF19.3.6,M20.F20.1.SF20.1.6,M20.F20.5.SF20.5.2 |
| SCR-FIN-credit-line | alias | SCR-FIN-ar-accounts | credit line — used by: M10.F10.3.SF10.3.1 |
| SCR-FIN-credit-note | alias | SCR-FO-invoice-print | credit note — used by: M38.F38.1.SF38.1.4 |
| SCR-FIN-cylinder-ledger | alias | SCR-FIN-gas-cylinder-ledger | same purpose — used by: M25.F25.1.SF25.1.1,M25.F25.1.SF25.1.2,M25.F25.1.SF25.1.6,M25.F25.2.SF25.2.1 |
| SCR-FIN-date-mapping | alias | SCR-FIN-period-close | business-date to accounting-period mapping tab — used by: M65.F65.1.SF65.1.4 |
| SCR-FIN-delegation-of-authority | alias | SCR-ADM-roles-permissions | approval limits (delegation of authority) tab — used by: M20.F20.5.SF20.5.3 |
| SCR-FIN-deposit-ledger | new | SCR-FIN-deposits | liability ledger tab — used by: M08.F08.3.SF08.3.2,M08.F08.3.SF08.3.6 |
| SCR-FIN-deposits-due | new | SCR-FIN-deposits | defined in §2 — used by: M08.F08.3.SF08.3.1 |
| SCR-FIN-expense-forecast | alias | SCR-FIN-cash-forecast | recurring expense lines — used by: M20.F20.4.SF20.4.3 |
| SCR-FIN-export-batches | alias | SCR-FIN-journal-list | export batches tab — used by: M08.F08.6.SF08.6.6 |
| SCR-FIN-external-obligations | alias | SCR-FIN-cash-forecast | external obligations (lease/debt) inputs tab — used by: M66.F66.1.SF66.1.4 |
| SCR-FIN-fee-rules | alias | SCR-ADM-tax-config | service charges, levies and rounding — used by: M04.F04.4.SF04.4.2,M04.F04.4.SF04.4.3 |
| SCR-FIN-fee-schedules | alias | SCR-FIN-owner-fees | same purpose — used by: M66.F66.1.SF66.1.2 |
| SCR-FIN-filing-calendar | alias | SCR-ADM-filing-calendar | same purpose — used by: M38.F38.1.SF38.1.5 |
| SCR-FIN-fiscal-submissions | alias | SCR-ADM-filing-submission | e-invoicing/fiscal submissions — used by: M08.F08.4.SF08.4.5 |
| SCR-FIN-fraud-review | alias | SCR-FIN-anomaly-case | payment fraud review — used by: M28.F28.3.SF28.3.5 |
| SCR-FIN-fx-rates | alias | SCR-FIN-chart-of-accounts | currencies and FX rates tab — used by: M19.F19.2.SF19.2.4 |
| SCR-FIN-gl-accounts | alias | SCR-FIN-chart-of-accounts | same purpose — used by: M19.F19.1.SF19.1.1 |
| SCR-FIN-gl-export | alias | SCR-FIN-journal-list | export batches tab — used by: M19.F19.2.SF19.2.5 |
| SCR-FIN-hsia-reconciliation | new | SCR-ADM-hsia-sessions | billable usage reconciliation tab — used by: M35.F35.2.SF35.2.3 |
| SCR-FIN-insurance-claims | alias | SCR-SAF-claim-file | same purpose — used by: M68.F68.1.SF68.1.3 |
| SCR-FIN-insurance-premiums | alias | SCR-SAF-insurance-register | premiums tab — used by: M68.F68.1.SF68.1.2 |
| SCR-FIN-insurance-register | alias | SCR-SAF-insurance-register | same purpose — used by: M68.F68.1.SF68.1.1 |
| SCR-FIN-inventory-valuation | alias | SCR-PRC-stock-balances | valuation view — used by: M14.F14.1.SF14.1.5 |
| SCR-FIN-invoice-detail | alias | SCR-FO-invoice-print | same purpose — used by: M08.F08.4.SF08.4.2 |
| SCR-FIN-invoice-issue | alias | SCR-FIN-ar-invoice | final invoice — used by: M12.F12.5.SF12.5.4 |
| SCR-FIN-invoice-preview | alias | SCR-FO-invoice-print | same purpose — used by: M01.F01.5.SF01.5.5,M08.F08.4.SF08.4.1 |
| SCR-FIN-journal-entry | alias | SCR-FIN-journal-detail | same purpose — used by: M19.F19.2.SF19.2.1,M19.F19.2.SF19.2.2,M19.F19.2.SF19.2.3,M19.F19.2.SF19.2.4,M27.F27.3.SF27.3.8 |
| SCR-FIN-journal-history | alias | SCR-FIN-journal-list | same purpose — used by: M19.F19.2.SF19.2.2 |
| SCR-FIN-lineage | alias | SCR-GM-flash-drilldown | source-to-ledger lineage — used by: M65.F65.1.SF65.1.3 |
| SCR-FIN-loyalty-fraud-queue | alias | SCR-FIN-revenue-protection | loyalty velocity anomalies — used by: M30.F30.2.SF30.2.3 |
| SCR-FIN-loyalty-ledger-explorer | alias | SCR-FIN-loyalty-liability | ledger explorer tab — used by: M30.F30.2.SF30.2.1 |
| SCR-FIN-loyalty-reversal-queue | alias | SCR-FIN-loyalty-liability | refund reversal queue — used by: M30.F30.1.SF30.1.5 |
| SCR-FIN-manual-filing-handoff | alias | SCR-ADM-filing-submission | manual portal/file path — used by: M38.F38.3.SF38.3.8 |
| SCR-FIN-manual-journal-approval | alias | SCR-OPS-approval-detail | manual journal approval type — used by: M19.F19.2.SF19.2.5 |
| SCR-FIN-margin-formula-versions | alias | SCR-FIN-commission-payouts | formula versions tab — used by: M31.F31.1.SF31.1.4 |
| SCR-FIN-marketplace-payouts | new | SCR-FIN-marketplace-payouts | defined in §2 — used by: M37.F37.1.SF37.1.4 |
| SCR-FIN-match-workbench | alias | SCR-FIN-three-way-match | same purpose — used by: M20.F20.1.SF20.1.4,M21.F21.2.SF21.2.5,M25.F25.1.SF25.1.5 |
| SCR-FIN-night-audit | alias | SCR-FO-night-audit | same purpose — used by: M01.F01.2.SF01.2.2,M01.F01.2.SF01.2.4,M06.F06.3.SF06.3.5,M08.F08.6.SF08.6.1,M08.F08.6.SF08.6.2,M60.F60.1.SF60.1.4 |
| SCR-FIN-night-audit-noshow | alias | SCR-FO-no-show-processing | same purpose — used by: M05.F05.3.SF05.3.3,M08.F08.6.SF08.6.3 |
| SCR-FIN-night-audit-reports | alias | SCR-FO-night-audit | audit reports tab — used by: M08.F08.6.SF08.6.4 |
| SCR-FIN-night-audit-summary | alias | SCR-FIN-night-audit-review | same purpose — used by: M19.F19.1.SF19.1.3 |
| SCR-FIN-package-allocation | alias | SCR-FIN-posting-rules | package and ancillary allocation tab — used by: M54.F54.2.SF54.2.3 |
| SCR-FIN-payable-approval | alias | SCR-FIN-supplier-invoice-detail | hold/dispute/approve — used by: M20.F20.1.SF20.1.5,M22.F22.2.SF22.2.4,M23.F23.1.SF23.1.6,M24.F24.1.SF24.1.6 |
| SCR-FIN-payable-detail | alias | SCR-FIN-payables-list | payable detail — used by: M20.F20.5.SF20.5.1,M20.F20.5.SF20.5.4,M21.F21.3.SF21.3.6 |
| SCR-FIN-payments | alias | SCR-FIN-psp-reconciliation | payment transactions tab — used by: M28.F28.1.SF28.1.2 |
| SCR-FIN-payout-approval | alias | SCR-FIN-payment-release | same purpose — used by: M20.F20.3.SF20.3.4,M27.F27.3.SF27.3.6,M28.F28.2.SF28.2.2 |
| SCR-FIN-payroll-remittances | alias | SCR-FIN-tax-returns | payroll remittance liabilities tab — used by: M38.F38.2.SF38.2.5 |
| SCR-FIN-period-calendar | alias | SCR-FIN-period-close | period calendar tab — used by: M01.F01.2.SF01.2.5 |
| SCR-FIN-periods | alias | SCR-FIN-period-close | same purpose — used by: M19.F19.1.SF19.1.3 |
| SCR-FIN-posting-journal | alias | SCR-FIN-journal-list | posting windows by business date — used by: M01.F01.2.SF01.2.3 |
| SCR-FIN-prepayments | alias | SCR-FIN-accruals-prepayments | same purpose — used by: M19.F19.1.SF19.1.4 |
| SCR-FIN-receipt-allocation | alias | SCR-FIN-bank-reconciliation | receipt allocation — used by: M20.F20.2.SF20.2.4,M28.F28.3.SF28.3.3 |
| SCR-FIN-recurring-payables | alias | SCR-FIN-payables-list | recurring (non-PO) payables tab — used by: M20.F20.5.SF20.5.6 |
| SCR-FIN-referral-payout-batch | alias | SCR-FIN-commission-payouts | payout batch — used by: M31.F31.1.SF31.1.6 |
| SCR-FIN-referral-qualification-queue | alias | SCR-FIN-commission-payouts | qualification queue — used by: M31.F31.1.SF31.1.3 |
| SCR-FIN-referral-reconciliation | alias | SCR-FIN-commission-payouts | reconciliation tab — used by: M31.F31.2.SF31.2.5 |
| SCR-FIN-report-snapshots | alias | SCR-GM-report-catalogue | point-in-time snapshots — used by: M65.F65.2.SF65.2.2 |
| SCR-FIN-reports | alias | SCR-OPS-report-viewer | same purpose — used by: M65.F65.2.SF65.2.1 |
| SCR-FIN-restatements | alias | SCR-FIN-period-close | restatements tab — used by: M32.F32.3.SF32.3.5 |
| SCR-FIN-safe-register | alias | SCR-FIN-cash-drop-safe | same purpose — used by: M60.F60.1.SF60.1.3 |
| SCR-FIN-shift-close | alias | SCR-FIN-cashier-shift | same purpose — used by: M60.F60.1.SF60.1.3 |
| SCR-FIN-sod-matrix | alias | SCR-ADM-roles-permissions | segregation-of-duties rules tab — used by: M21.F21.3.SF21.3.4 |
| SCR-FIN-statements | alias | SCR-FIN-financial-statements | same purpose — used by: M19.F19.3.SF19.3.4,M27.F27.4.SF27.4.1 |
| SCR-FIN-submission-preview | alias | SCR-ADM-filing-submission | payload preview — used by: M38.F38.3.SF38.3.5 |
| SCR-FIN-submission-status | alias | SCR-ADM-filing-submission | status and receipt — used by: M38.F38.3.SF38.3.6 |
| SCR-FIN-supplier-payment-profile | alias | SCR-PRC-vendor-detail | payment details (bank change via SCR-FIN-payee-bank-change) — used by: M20.F20.1.SF20.1.1 |
| SCR-FIN-tariffs | alias | SCR-ENG-tariffs | same purpose — used by: M22.F22.1.SF22.1.5,M23.F23.1.SF23.1.3,M24.F24.1.SF24.1.3 |
| SCR-FIN-tax-invoice | alias | SCR-FO-invoice-print | tax invoice — used by: M38.F38.1.SF38.1.4 |
| SCR-FIN-tax-reconciliation | alias | SCR-FIN-tax-returns | reconciliation tab — used by: M38.F38.1.SF38.1.5 |
| SCR-FIN-till-open | alias | SCR-FIN-cashier-shift | open with float — used by: M60.F60.1.SF60.1.1 |
| SCR-FIN-tip-pool | new | SCR-FIN-tips-commissions | defined in §2 — used by: M13.F13.3.SF13.3.4 |
| SCR-FIN-transport-settlement | alias | SCR-FIN-travel-reconciliation | transport vendor settlement — used by: M59.F59.2.SF59.2.4 |
| SCR-FIN-travel-settlement | alias | SCR-FIN-travel-reconciliation | same purpose — used by: M45.F45.2.SF45.2.8,M45.F45.2.SF45.2.10 |
| SCR-FIN-unmatched-items | alias | SCR-FIN-bank-reconciliation | unmatched items tab — used by: M20.F20.3.SF20.3.5 |
| SCR-FIN-utility-accounts | alias | SCR-ENG-utility-accounts | same purpose — used by: M22.F22.1.SF22.1.1,M23.F23.1.SF23.1.1,M24.F24.1.SF24.1.1 |
| SCR-FIN-utility-reconciliation | alias | SCR-FIN-billpay-orders | payment reconciliation tab — used by: M22.F22.3.SF22.3.4,M29.F29.1.SF29.1.8,M29.F29.2.SF29.2.7,M29.F29.3.SF29.3.3 |
| SCR-FIN-vendor-bank-change | alias | SCR-FIN-payee-bank-change | same purpose — used by: M46.F46.1.SF46.1.4 |
| SCR-FIN-voucher-reconciliation | alias | SCR-FIN-voucher-liability | reconciliation tab — used by: M54.F54.2.SF54.2.5 |
| SCR-FIN-write-off | alias | SCR-FIN-credit-limit-writeoff | same purpose — used by: M20.F20.2.SF20.2.5 |
| SCR-FIN-year-end-pack | alias | SCR-FIN-financial-statements | year-end export pack — used by: M66.F66.1.SF66.1.5 |
| SCR-FNB-age-check | alias | SCR-FNB-club-entry | age/licensing gate — used by: M57.F57.1.SF57.1.5 |
| SCR-FNB-amenity-pos | alias | SCR-FNB-pos-order | amenity treatment/retail outlet mode — used by: M58.F58.2.SF58.2.1 |
| SCR-FNB-check-settle | alias | SCR-FNB-pos-payment | same purpose — used by: M57.F57.1.SF57.1.3 |
| SCR-FNB-food-checks | alias | SCR-FNB-haccp-checks | same purpose — used by: M57.F57.2.SF57.2.2 |
| SCR-FNB-food-incident | alias | SCR-SAF-incident-report | food-safety incident type — used by: M57.F57.2.SF57.2.6 |
| SCR-FNB-ird-board | alias | SCR-FNB-in-room-dining | same purpose — used by: M57.F57.1.SF57.1.2 |
| SCR-FNB-kds-allergen | alias | SCR-FNB-kds | allergen banner and acknowledgement — used by: M57.F57.1.SF57.1.4 |
| SCR-FNB-outlet-performance | alias | SCR-GM-department-pnl | outlet food cost and satisfaction view — used by: M57.F57.1.SF57.1.7 |
| SCR-FNB-production-batch | alias | SCR-FNB-production-plan | batch with recipe yield and ingredient lots — used by: M57.F57.2.SF57.2.1 |
| SCR-FNB-reservation-book | alias | SCR-FNB-restaurant-reservations | same purpose — used by: M57.F57.1.SF57.1.1 |
| SCR-FNB-service-rules | alias | SCR-ADM-outlet-setup | licensing, age and responsible-service rules tab — used by: M57.F57.1.SF57.1.5 |
| SCR-FNB-substitution-review | alias | SCR-FNB-recipe-bom | substitution and allergen impact review — used by: M57.F57.2.SF57.2.3 |
| SCR-FNB-ticket-times | alias | SCR-FNB-kds | ticket timing and pacing panel — used by: M57.F57.1.SF57.1.6 |
| SCR-FNB-trace | alias | SCR-FNB-lot-trace | same purpose — used by: M57.F57.2.SF57.2.4 |
| SCR-FNB-void-approval | alias | SCR-FNB-pos-void-comp-approval | same purpose — used by: M57.F57.1.SF57.1.3 |
| SCR-FNB-waitlist | alias | SCR-FNB-restaurant-reservations | waitlist tab — used by: M57.F57.1.SF57.1.1 |
| SCR-GM-approvals | alias | SCR-OPS-approvals-queue | same purpose — used by: M20.F20.1.SF20.1.5,M20.F20.2.SF20.2.5,M21.F21.1.SF21.1.3,M21.F21.1.SF21.1.6,M27.F27.3.SF27.3.5 |
| SCR-GM-approvals-inbox | alias | SCR-OPS-approvals-queue | same purpose — used by: M02.F02.4.SF02.4.3,M02.F02.4.SF02.4.4,M04.F04.5.SF04.5.6,M08.F08.1.SF08.1.5,M08.F08.3.SF08.3.5 |
| SCR-GM-attention-inbox | alias | SCR-GM-home | attention panel — used by: M01.F01.2.SF01.2.4,M01.F01.6.SF01.6.2,M03.F03.4.SF03.4.6,M06.F06.1.SF06.1.4 |
| SCR-GM-daily-flash | alias | SCR-GM-flash | same purpose — used by: M08.F08.6.SF08.6.4 |
| SCR-GM-drill-through | alias | SCR-GM-flash-drilldown | same purpose — used by: M19.F19.2.SF19.2.3 |
| SCR-GST-account | new | SCR-GST-profile | defined in §2 — used by: M02.F02.1.SF02.1.2 |
| SCR-GST-add-ons | alias | SCR-GST-extras | same purpose — used by: M54.F54.1.SF54.1.3,M58.F58.2.SF58.2.6 |
| SCR-GST-case-status | alias | SCR-GST-request-detail | same purpose — used by: M55.F55.2.SF55.2.4 |
| SCR-GST-chat | alias | SCR-GST-messages | same purpose — used by: M04.F04.5.SF04.5.7,M18.F18.4.SF18.4.1 |
| SCR-GST-checkout | alias | SCR-GST-payment | booking review, consent and hold step — used by: M02.F02.5.SF02.5.2,M03.F03.4.SF03.4.1,M04.F04.3.SF04.3.2,M04.F04.4.SF04.4.1,M04.F04.4.SF04.4.4,M04.F04.4.SF04.4.5 |
| SCR-GST-club-booking | new | SCR-GST-club | reservations tab — used by: M15.F15.3.SF15.3.1 |
| SCR-GST-club-events | new | SCR-GST-club | events and tickets tab — used by: M15.F15.4.SF15.4.4 |
| SCR-GST-communication-preferences | alias | SCR-GST-privacy-consent | same purpose — used by: M52.F52.1.SF52.1.3 |
| SCR-GST-day-pass | alias | SCR-GST-amenity-booking | day pass — used by: M58.F58.1.SF58.1.3 |
| SCR-GST-day-use | alias | SCR-GST-facility-search | day-use rooms — used by: M09.F09.2.SF09.2.3 |
| SCR-GST-digital-key | alias | SCR-GST-digital-pass | digital key panel — used by: M55.F55.3.SF55.3.1 |
| SCR-GST-dining-booking | alias | SCR-GST-facility-search | restaurant tables — used by: M09.F09.2.SF09.2.2,M57.F57.1.SF57.1.1 |
| SCR-GST-early-late | alias | SCR-GST-extras | early arrival/late checkout — used by: M54.F54.1.SF54.1.2 |
| SCR-GST-express-checkout | alias | SCR-GST-checkout-express | same purpose — used by: M05.F05.5.SF05.5.3,M05.F05.5.SF05.5.4 |
| SCR-GST-feedback-pulse | alias | SCR-GST-feedback-survey | in-stay pulse mode — used by: M18.F18.4.SF18.4.2 |
| SCR-GST-find-booking | alias | SCR-GST-sign-in | find booking (reservation-bound access) — used by: M18.F18.1.SF18.1.1 |
| SCR-GST-membership | new | SCR-GST-club | memberships tab — used by: M15.F15.2.SF15.2.2,M15.F15.2.SF15.2.3 |
| SCR-GST-menu-allergens | alias | SCR-GST-in-room-dining | allergen information panel — used by: M57.F57.1.SF57.1.4 |
| SCR-GST-my-extras | alias | SCR-GST-manage-booking | booked extras tab — used by: M54.F54.1.SF54.1.5 |
| SCR-GST-my-preferences | new | SCR-GST-profile | preferences tab — used by: M52.F52.1.SF52.1.2 |
| SCR-GST-online-check-in | alias | SCR-GST-precheckin | same purpose — used by: M05.F05.4.SF05.4.2,M05.F05.4.SF05.4.3 |
| SCR-GST-parking-pay | alias | SCR-GST-parking-vehicle | pay on exit — used by: M17.F17.4.SF17.4.4 |
| SCR-GST-pay | alias | SCR-GST-pay-balance | same purpose — used by: M28.F28.1.SF28.1.1,M28.F28.1.SF28.1.4,M28.F28.3.SF28.3.1 |
| SCR-GST-payment-methods | new | SCR-GST-profile | saved payment methods (PSP tokens) tab — used by: M28.F28.3.SF28.3.6 |
| SCR-GST-pre-arrival | alias | SCR-GST-precheckin | same purpose — used by: M55.F55.1.SF55.1.2 |
| SCR-GST-preferences | new | SCR-GST-profile | preferences and accessibility tab — used by: M18.F18.2.SF18.2.2 |
| SCR-GST-privacy-center | alias | SCR-GST-privacy-consent | same purpose — used by: M02.F02.5.SF02.5.2,M02.F02.5.SF02.5.3,M02.F02.5.SF02.5.6 |
| SCR-GST-privacy-requests | alias | SCR-GST-data-request | same purpose — used by: M52.F52.1.SF52.1.8 |
| SCR-GST-privacy-settings | alias | SCR-GST-privacy-consent | same purpose — used by: M18.F18.5.SF18.5.1,M18.F18.5.SF18.5.2 |
| SCR-GST-profile | new | SCR-GST-profile | defined in §2 — used by: M18.F18.2.SF18.2.1 |
| SCR-GST-request | alias | SCR-GST-service-requests | same purpose — used by: M55.F55.1.SF55.1.3 |
| SCR-GST-request-new | alias | SCR-GST-service-requests | new request — used by: M18.F18.3.SF18.3.2,M18.F18.3.SF18.3.4 |
| SCR-GST-request-status | alias | SCR-GST-request-detail | same purpose — used by: M55.F55.1.SF55.1.4 |
| SCR-GST-requests | alias | SCR-GST-service-requests | same purpose — used by: M06.F06.4.SF06.4.3,M18.F18.3.SF18.3.3 |
| SCR-GST-results | alias | SCR-GST-room-results | same purpose — used by: M07.F07.1.SF07.1.1,M07.F07.1.SF07.1.2 |
| SCR-GST-search | alias | SCR-GST-room-search | same purpose — used by: M03.F03.2.SF03.2.2,M04.F04.2.SF04.2.2,M04.F04.4.SF04.4.4,M04.F04.4.SF04.4.5,M04.F04.5.SF04.5.1,M07.F07.1.SF07.1.1 |
| SCR-GST-service-request | alias | SCR-GST-service-requests | fault report type — used by: M26.F26.1.SF26.1.3 |
| SCR-GST-settings | new | SCR-GST-profile | notification settings tab — used by: M18.F18.1.SF18.1.4 |
| SCR-GST-spa-intake | alias | SCR-GST-amenity-booking | intake and consent form — used by: M58.F58.2.SF58.2.2 |
| SCR-GST-stay | alias | SCR-GST-my-trips | current-stay card (room-ready ETA, DND) — used by: M06.F06.2.SF06.2.5,M06.F06.2.SF06.2.6 |
| SCR-GST-stay-bill | alias | SCR-GST-purchases | same purpose — used by: M08.F08.4.SF08.4.4 |
| SCR-GST-stay-home | alias | SCR-GST-my-trips | in-stay dashboard — used by: M18.F18.1.SF18.1.2 |
| SCR-GST-survey | alias | SCR-GST-feedback-survey | same purpose — used by: M18.F18.4.SF18.4.3,M52.F52.2.SF52.2.1 |
| SCR-GST-transfer-quote | new | SCR-GST-transfer | quote step — used by: M59.F59.2.SF59.2.3 |
| SCR-GST-transfer-request | new | SCR-GST-transfer | defined in §2 — used by: M59.F59.1.SF59.1.2 |
| SCR-GST-transfer-status | new | SCR-GST-transfer | live trip status — used by: M59.F59.1.SF59.1.4 |
| SCR-GST-travel-consent | new | SCR-GST-transfer | traveler data consent and offer approval step — used by: M45.F45.1.SF45.1.2,M45.F45.1.SF45.1.5,M45.F45.1.SF45.1.7,M45.F45.2.SF45.2.5,M45.F45.2.SF45.2.6 |
| SCR-GST-upgrade-offer | alias | SCR-GST-extras | upgrade offer — used by: M54.F54.1.SF54.1.1 |
| SCR-GST-vehicle | alias | SCR-GST-parking-vehicle | same purpose — used by: M17.F17.1.SF17.1.3,M18.F18.3.SF18.3.5 |
| SCR-GST-wallet-pass | alias | SCR-GST-digital-pass | same purpose — used by: M15.F15.3.SF15.3.2 |
| SCR-GUEST-account-security | new | SCR-GST-profile | account and device security tab — used by: M30.F30.2.SF30.2.2 |
| SCR-GUEST-biometric-consent | alias | SCR-GST-id-capture | liveness consent step — used by: M41.F41.1.SF41.1.6 |
| SCR-GUEST-booking-referral-code-field | alias | SCR-GST-guest-details | referral code field — used by: M31.F31.1.SF31.1.2 |
| SCR-GUEST-booking-review | alias | SCR-AI-availability-card | draft booking review — used by: M40.F40.2.SF40.2.2,M41.F41.2.SF41.2.1 |
| SCR-GUEST-chat | alias | SCR-AI-chat-widget | same purpose — used by: M40.F40.1.SF40.1.2,M40.F40.1.SF40.1.3,M40.F40.1.SF40.1.4,M40.F40.2.SF40.2.1,M40.F40.2.SF40.2.3,M40.F40.2.SF40.2.6 |
| SCR-GUEST-chat-privacy | alias | SCR-AI-transcript-controls | same purpose — used by: M40.F40.2.SF40.2.4 |
| SCR-GUEST-checkout-pay-with-points | alias | SCR-GST-payment | points tender — used by: M30.F30.1.SF30.1.6 |
| SCR-GUEST-cookie-preferences | alias | SCR-GST-cookie-consent | same purpose — used by: M31.F31.1.SF31.1.2 |
| SCR-GUEST-id-capture | alias | SCR-GST-id-capture | same purpose — used by: M41.F41.1.SF41.1.2,M41.F41.1.SF41.1.3 |
| SCR-GUEST-id-consent | alias | SCR-GST-id-capture | consent step — used by: M41.F41.2.SF41.2.8 |
| SCR-GUEST-id-review | alias | SCR-GST-id-confirm | same purpose — used by: M41.F41.1.SF41.1.4,M41.F41.1.SF41.1.5 |
| SCR-GUEST-id-start | alias | SCR-GST-precheckin | ID step start (document-type check) — used by: M41.F41.1.SF41.1.1 |
| SCR-GUEST-lost-item-inquiry | alias | SCR-GST-lost-item-inquiry | same purpose — used by: M43.F43.1.SF43.1.4 |
| SCR-GUEST-loyalty-enrol-consent | alias | SCR-GST-points | enrolment consent — used by: M30.F30.2.SF30.2.5 |
| SCR-GUEST-otp | alias | SCR-GST-otp-verify | same purpose — used by: M41.F41.2.SF41.2.5 |
| SCR-GUEST-payment | alias | SCR-GST-payment | same purpose — used by: M41.F41.2.SF41.2.4 |
| SCR-GUEST-points-dispute | alias | SCR-GST-points | dispute action — used by: M30.F30.2.SF30.2.4 |
| SCR-GUEST-points-expiry-notice | alias | SCR-GST-points | expiry notice — used by: M30.F30.1.SF30.1.4 |
| SCR-GUEST-points-history | alias | SCR-GST-points | history tab — used by: M30.F30.1.SF30.1.5,M30.F30.2.SF30.2.4 |
| SCR-GUEST-points-how-to-earn | alias | SCR-GST-points | how-to-earn panel — used by: M30.F30.1.SF30.1.1 |
| SCR-GUEST-points-tier-status | alias | SCR-GST-points | tier status panel — used by: M30.F30.1.SF30.1.2 |
| SCR-GUEST-points-wallet | alias | SCR-GST-points | same purpose — used by: M30.F30.1.SF30.1.3,M30.F30.1.SF30.1.4,M30.F30.2.SF30.2.2,M30.F30.2.SF30.2.6 |
| SCR-GUEST-privacy-preferences | alias | SCR-GST-privacy-consent | same purpose — used by: M30.F30.2.SF30.2.5 |
| SCR-GUEST-qr-continue | alias | SCR-GST-qr-handoff | same purpose — used by: M41.F41.2.SF41.2.6 |
| SCR-GUEST-quote-card | alias | SCR-AI-availability-card | same purpose — used by: M40.F40.2.SF40.2.1 |
| SCR-GUEST-report-issue | alias | SCR-GST-service-requests | safety issue type (routes to SCR-SAF-incident-report) — used by: M42.F42.1.SF42.1.1 |
| SCR-GUEST-sign-registration | alias | SCR-GST-registration-sign | same purpose — used by: M41.F41.2.SF41.2.3 |
| SCR-GUEST-tv-cast-pairing | new | SCR-GST-iptv-home | cast pairing panel — used by: M36.F36.1.SF36.1.3 |
| SCR-GUEST-wake-up | alias | SCR-GST-service-requests | wake-up call request type (M34, Later) — used by: M34.F34.2.SF34.2.1 |
| SCR-GUEST-wifi-portal | new | SCR-GST-wifi-portal | defined in §2 — used by: M35.F35.1.SF35.1.1 |
| SCR-GUEST-wifi-upgrade | new | SCR-GST-wifi-portal | premium tier purchase panel — used by: M35.F35.1.SF35.1.4 |
| SCR-HK-amenity-par | alias | SCR-FO-hk-linen-par | amenity par tab — used by: M56.F56.2.SF56.2.5 |
| SCR-HK-assignment | alias | SCR-FO-hk-task-assign | same purpose — used by: M06.F06.2.SF06.2.3,M56.F56.1.SF56.1.2 |
| SCR-HK-conflicts | alias | SCR-OPS-sync-queue | housekeeping conflicts — used by: M56.F56.1.SF56.1.5 |
| SCR-HK-damage-loss | alias | SCR-FO-hk-linen-discrepancy | damage/loss replacement cost — used by: M56.F56.2.SF56.2.3 |
| SCR-HK-deep-clean-plan | alias | SCR-FO-hk-task-assign | deep-clean rotation tab — used by: M56.F56.1.SF56.1.6 |
| SCR-HK-defect | alias | SCR-FO-hk-fault-report | same purpose — used by: M06.F06.4.SF06.4.5 |
| SCR-HK-device-enrol | alias | SCR-OPS-device-enrollment | same purpose — used by: M01.F01.6.SF01.6.4 |
| SCR-HK-found-item | alias | SCR-SAF-lost-intake | same purpose — used by: M06.F06.4.SF06.4.4 |
| SCR-HK-inspection | alias | SCR-FO-hk-inspection | same purpose — used by: M03.F03.3.SF03.3.4,M06.F06.2.SF06.2.4,M56.F56.1.SF56.1.3 |
| SCR-HK-laundry-batches | alias | SCR-FO-hk-laundry-dispatch | same purpose — used by: M56.F56.2.SF56.2.2 |
| SCR-HK-linen-par | alias | SCR-FO-hk-linen-par | same purpose — used by: M56.F56.2.SF56.2.1 |
| SCR-HK-minibar | alias | SCR-FO-hk-minibar-count | same purpose — used by: M06.F06.4.SF06.4.2 |
| SCR-HK-minibar-exceptions | alias | SCR-FO-hk-minibar-count | late charge and exception review — used by: M56.F56.2.SF56.2.4 |
| SCR-HK-my-rooms | alias | SCR-FO-hk-my-rooms | same purpose — used by: M06.F06.3.SF06.3.1,M06.F06.3.SF06.3.2 |
| SCR-HK-release-gate | alias | SCR-FO-hk-inspection | release gate — used by: M56.F56.1.SF56.1.4 |
| SCR-HK-room-board | alias | SCR-FO-hk-room-board | same purpose — used by: M03.F03.3.SF03.3.2,M06.F06.2.SF06.2.1,M06.F06.2.SF06.2.6,M56.F56.1.SF56.1.1 |
| SCR-HK-sync-conflicts | alias | SCR-OPS-sync-queue | same purpose — used by: M06.F06.3.SF06.3.3,M06.F06.3.SF06.3.4 |
| SCR-HK-sync-status | alias | SCR-OPS-sync-queue | same purpose — used by: M06.F06.3.SF06.3.2 |
| SCR-HK-task-board | alias | SCR-FO-hk-task-assign | task generation and priorities — used by: M06.F06.2.SF06.2.2,M06.F06.4.SF06.4.3 |
| SCR-HK-task-detail | alias | SCR-FO-hk-room-task | same purpose — used by: M05.F05.2.SF05.2.4,M06.F06.3.SF06.3.1,M06.F06.4.SF06.4.1,M06.F06.4.SF06.4.6 |
| SCR-HK-variance | alias | SCR-FO-hk-minibar-count | variance/dispute review — used by: M56.F56.2.SF56.2.6 |
| SCR-HR-access-log | alias | SCR-ADM-audit-log | HR sensitive access filter — used by: M27.F27.1.SF27.1.5,M27.F27.4.SF27.4.5 |
| SCR-HR-compensation | alias | SCR-HR-contract-compensation | same purpose — used by: M27.F27.1.SF27.1.2 |
| SCR-HR-employee | alias | SCR-HR-employee-profile | same purpose — used by: M27.F27.1.SF27.1.1 |
| SCR-HR-employee-identity | alias | SCR-HR-employee-sensitive | same purpose — used by: M38.F38.2.SF38.2.1 |
| SCR-HR-exports | alias | SCR-HR-payroll-run | exports and retention tab — used by: M27.F27.4.SF27.4.4 |
| SCR-HR-labor-allocation | alias | SCR-HR-labor-cost | department/event allocation — used by: M27.F27.2.SF27.2.5 |
| SCR-HR-leave | alias | SCR-HR-leave-requests | same purpose — used by: M27.F27.2.SF27.2.3 |
| SCR-HR-offboarding | alias | SCR-HR-onboarding-offboarding | same purpose — used by: M27.F27.1.SF27.1.4 |
| SCR-HR-pay-elements | alias | SCR-HR-contract-compensation | allowances, deductions and advances tab — used by: M27.F27.3.SF27.3.2 |
| SCR-HR-payroll-approval | alias | SCR-FIN-payroll-approval | same purpose — used by: M27.F27.3.SF27.3.5 |
| SCR-HR-payroll-corrections | alias | SCR-HR-payroll-exceptions | corrections and amendments — used by: M38.F38.2.SF38.2.6 |
| SCR-HR-payroll-statutory-preview | alias | SCR-HR-payroll-preview | statutory deductions view — used by: M38.F38.2.SF38.2.2,M38.F38.2.SF38.2.3 |
| SCR-HR-roster | alias | SCR-HR-roster-planner | same purpose — used by: M27.F27.2.SF27.2.1 |
| SCR-HR-sin-reveal | alias | SCR-HR-employee-sensitive | same purpose — used by: M38.F38.2.SF38.2.1 |
| SCR-HR-statutory-rules | alias | SCR-ADM-rule-packs | payroll category — used by: M27.F27.3.SF27.3.3,M27.F27.5.SF27.5.6 |
| SCR-HR-timesheets | alias | SCR-HR-time-attendance | same purpose — used by: M27.F27.2.SF27.2.2,M27.F27.2.SF27.2.4 |
| SCR-HR-wps-export | alias | SCR-HR-wps-bank-file | same purpose — used by: M27.F27.3.SF27.3.6,M27.F27.3.SF27.3.7,M27.F27.5.SF27.5.2 |
| SCR-HR-wps-setup | alias | SCR-HR-wps-bank-file | WPS establishment setup tab — used by: M27.F27.5.SF27.5.1 |
| SCR-HR-wps-status | alias | SCR-HR-payment-status | WPS/bank feedback — used by: M27.F27.5.SF27.5.3 |
| SCR-HR-year-end-slips | alias | SCR-HR-statutory-forms | same purpose — used by: M38.F38.2.SF38.2.4 |
| SCR-HUB-agreement-sign | alias | SCR-GST-partner-hub-onboarding | agreement step — used by: M31.F31.1.SF31.1.1 |
| SCR-HUB-apply | alias | SCR-GST-partner-hub-onboarding | same purpose — used by: M31.F31.1.SF31.1.1 |
| SCR-HUB-dashboard | alias | SCR-GST-referral-status | same purpose — used by: M31.F31.2.SF31.2.5 |
| SCR-HUB-disclosure-kit | alias | SCR-GST-referral-status | disclosure text panel — used by: M31.F31.1.SF31.1.6 |
| SCR-HUB-identity-and-tax | alias | SCR-GST-partner-hub-onboarding | identity and tax step — used by: M31.F31.2.SF31.2.2 |
| SCR-HUB-my-code | alias | SCR-GST-referral-status | code and link panel — used by: M31.F31.1.SF31.1.1 |
| SCR-HUB-my-referred-bookings | alias | SCR-GST-referral-status | attributed bookings list — used by: M31.F31.2.SF31.2.3 |
| SCR-HUB-onboarding-wizard | alias | SCR-GST-partner-hub-onboarding | same purpose — used by: M31.F31.2.SF31.2.2 |
| SCR-HUB-statement | alias | SCR-GST-partner-hub-statement | same purpose — used by: M31.F31.1.SF31.1.5,M31.F31.2.SF31.2.4 |
| SCR-HUB-tax-and-payout-details | alias | SCR-GST-partner-hub-onboarding | payment details step — used by: M31.F31.1.SF31.1.6 |
| SCR-INT-api-clients | alias | SCR-ADM-webhooks-api-keys | same purpose — used by: M02.F02.1.SF02.1.5 |
| SCR-INT-ari-monitor | alias | SCR-GM-rate-publish-status | same purpose — used by: M07.F07.3.SF07.3.1,M07.F07.3.SF07.3.2,M07.F07.3.SF07.3.4,M07.F07.3.SF07.3.5 |
| SCR-INT-channel-bookings | alias | SCR-ADM-integration-health | channel booking ingest queue — used by: M05.F05.1.SF05.1.4,M07.F07.4.SF07.4.1,M07.F07.4.SF07.4.2 |
| SCR-INT-channel-mapping | new | SCR-ADM-channel-mapping | defined in §2 — used by: M04.F04.1.SF04.1.5,M07.F07.2.SF07.2.2,M07.F07.2.SF07.2.3 |
| SCR-INT-channels | alias | SCR-ADM-connectors | channel connections — used by: M07.F07.2.SF07.2.1 |
| SCR-INT-credentials | alias | SCR-ADM-secrets-certs | same purpose — used by: M01.F01.6.SF01.6.3 |
| SCR-INT-dead-letters | alias | SCR-ADM-dead-letters | same purpose — used by: M01.F01.6.SF01.6.6,M07.F07.3.SF07.3.3 |
| SCR-INT-event-health | alias | SCR-ADM-event-monitor | same purpose — used by: M01.F01.6.SF01.6.6 |
| SCR-INT-manual-channel-tasks | alias | SCR-OPS-unified-inbox | manual extranet task type — used by: M07.F07.2.SF07.2.4 |
| SCR-INT-reconciliation | alias | SCR-ADM-reconciliation-hub | channel tab — used by: M07.F07.4.SF07.4.5 |
| SCR-INV-count-sheet | alias | SCR-PRC-blind-count | count sheet — used by: M14.F14.5.SF14.5.3 |
| SCR-INV-depletion-log | alias | SCR-FNB-theoretical-vs-actual | theoretical depletion log tab — used by: M14.F14.5.SF14.5.1 |
| SCR-INV-event-consumption | alias | SCR-FNB-theoretical-vs-actual | batch and event consumption tab — used by: M14.F14.5.SF14.5.2 |
| SCR-INV-expiry-board | alias | SCR-PRC-stock-balances | FEFO and expiry view — used by: M14.F14.4.SF14.4.2 |
| SCR-INV-issue | alias | SCR-PRC-store-issue | same purpose — used by: M14.F14.3.SF14.3.4 |
| SCR-INV-item-detail | alias | SCR-PRC-item-master | item detail — used by: M14.F14.1.SF14.1.1 |
| SCR-INV-item-list | alias | SCR-PRC-item-master | same purpose — used by: M14.F14.1.SF14.1.1 |
| SCR-INV-ledger | alias | SCR-PRC-stock-balances | stock ledger tab — used by: M14.F14.3.SF14.3.1 |
| SCR-INV-lot-detail | alias | SCR-PRC-stock-balances | lot detail drawer — used by: M14.F14.4.SF14.4.1 |
| SCR-INV-pos-mapping | alias | SCR-FNB-recipe-bom | POS item mapping tab — used by: M14.F14.2.SF14.2.3 |
| SCR-INV-quarantine-queue | alias | SCR-PRC-quarantine | same purpose — used by: M14.F14.4.SF14.4.4 |
| SCR-INV-recall-case | alias | SCR-PRC-recall-lookup | same purpose — used by: M14.F14.4.SF14.4.3 |
| SCR-INV-receipt-post | alias | SCR-PRC-receiving | stock posting on GRN — used by: M14.F14.3.SF14.3.2 |
| SCR-INV-recipe-cost | alias | SCR-FNB-recipe-bom | costing and menu margin tab — used by: M14.F14.2.SF14.2.4 |
| SCR-INV-recipe-editor | alias | SCR-FNB-recipe-bom | same purpose — used by: M14.F14.2.SF14.2.1,M14.F14.2.SF14.2.5 |
| SCR-INV-reorder | alias | SCR-PRC-replenishment | same purpose — used by: M14.F14.1.SF14.1.4 |
| SCR-INV-reservations | alias | SCR-FNB-production-plan | ingredient reservations tab — used by: M16.F16.3.SF16.3.2 |
| SCR-INV-return | alias | SCR-PRC-store-return | same purpose — used by: M14.F14.3.SF14.3.5 |
| SCR-INV-stores | alias | SCR-PRC-stock-balances | stores and bins setup tab — used by: M14.F14.1.SF14.1.3 |
| SCR-INV-transfer | alias | SCR-PRC-transfers | same purpose — used by: M14.F14.3.SF14.3.3 |
| SCR-INV-uom | alias | SCR-PRC-item-master | UOM conversions tab — used by: M14.F14.1.SF14.1.2 |
| SCR-INV-variance-report | alias | SCR-FNB-theoretical-vs-actual | variance and shrinkage analysis — used by: M14.F14.5.SF14.5.4 |
| SCR-INV-waste | alias | SCR-FNB-waste-log | same purpose — used by: M14.F14.3.SF14.3.6 |
| SCR-INV-yield-test | alias | SCR-FNB-recipe-bom | yield and trim factors tab — used by: M14.F14.2.SF14.2.2 |
| SCR-KDS-86-list | alias | SCR-FNB-kds | 86 list panel — used by: M13.F13.1.SF13.1.5 |
| SCR-KDS-batch | alias | SCR-FNB-production-plan | production batch and lot linkage — used by: M16.F16.3.SF16.3.5 |
| SCR-KDS-event-orders | alias | SCR-FNB-beo-kitchen-view | same purpose — used by: M12.F12.3.SF12.3.3 |
| SCR-KDS-prep-list | alias | SCR-FNB-production-plan | prep lists — used by: M16.F16.3.SF16.3.1 |
| SCR-KDS-station | alias | SCR-FNB-kds | same purpose — used by: M13.F13.2.SF13.2.3 |
| SCR-KIT-callout-console | alias | SCR-FNB-chef-callout | same purpose — used by: M47.F47.1.SF47.1.6,M47.F47.2.SF47.2.1,M47.F47.2.SF47.2.2,M47.F47.2.SF47.2.4,M47.F47.2.SF47.2.6,M47.F47.2.SF47.2.7 |
| SCR-KIT-chef-coverage-board | alias | SCR-FNB-chef-coverage-plan | same purpose — used by: M47.F47.1.SF47.1.1,M47.F47.1.SF47.1.3,M47.F47.1.SF47.1.4,M50.F50.1.SF50.1.6 |
| SCR-KIT-consumption-variance | alias | SCR-FNB-theoretical-vs-actual | same purpose — used by: M50.F50.3.SF50.3.3,M50.F50.3.SF50.3.4,M50.F50.3.SF50.3.7 |
| SCR-KIT-emergency-roster | alias | SCR-FNB-emergency-roster | same purpose — used by: M47.F47.1.SF47.1.5 |
| SCR-KIT-handover-packet | alias | SCR-FNB-callout-handover | same purpose — used by: M47.F47.2.SF47.2.5 |
| SCR-KIT-requisition-quick | alias | SCR-PRC-requisition-new | quick mode from kitchen/job/event — used by: M49.F49.1.SF49.1.1,M50.F50.3.SF50.3.2 |
| SCR-KIT-skills-matrix | alias | SCR-FNB-chef-roster | skills matrix tab — used by: M47.F47.1.SF47.1.2 |
| SCR-KSK-check-in | new | SCR-GST-kiosk-check-in | defined in §2 — used by: M55.F55.3.SF55.3.2 |
| SCR-MGMT-assistant-quality | alias | SCR-AI-cost-usage | containment and quality metrics — used by: M40.F40.1.SF40.1.6 |
| SCR-MGMT-block-pickup | alias | SCR-GM-group-block | pickup report — used by: M32.F32.1.SF32.1.4 |
| SCR-MGMT-budget-vs-actual | alias | SCR-GM-budget-vs-actual | same purpose — used by: M32.F32.3.SF32.3.3 |
| SCR-MGMT-cancellations-no-shows | alias | SCR-GM-report-catalogue | cancellation/no-show report — used by: M32.F32.1.SF32.1.5 |
| SCR-MGMT-channel-contribution | alias | SCR-GM-channel-performance | same purpose — used by: M32.F32.1.SF32.1.3 |
| SCR-MGMT-corporate-report | alias | SCR-GM-report-catalogue | corporate report — used by: M32.F32.4.SF32.4.6 |
| SCR-MGMT-data-coverage | alias | SCR-GM-data-coverage | same purpose — used by: M32.F32.3.SF32.3.4 |
| SCR-MGMT-data-quality | alias | SCR-ADM-data-quality | same purpose — used by: M32.F32.4.SF32.4.7 |
| SCR-MGMT-dept-pnl-rooms | alias | SCR-GM-department-pnl | rooms department — used by: M32.F32.2.SF32.2.1 |
| SCR-MGMT-drill-through | alias | SCR-GM-flash-drilldown | same purpose — used by: M32.F32.3.SF32.3.5 |
| SCR-MGMT-event-report | alias | SCR-GM-report-catalogue | events report — used by: M32.F32.4.SF32.4.6 |
| SCR-MGMT-fees | alias | SCR-GM-channel-performance | payment and channel fees view — used by: M32.F32.2.SF32.2.6 |
| SCR-MGMT-gm-flash | alias | SCR-GM-flash | same purpose — used by: M32.F32.1.SF32.1.2,M32.F32.4.SF32.4.1 |
| SCR-MGMT-kpi-dictionary | alias | SCR-ADM-kpi-dictionary | same purpose — used by: M32.F32.1.SF32.1.1 |
| SCR-MGMT-labor-cost | alias | SCR-GM-payroll-summary | same purpose — used by: M32.F32.4.SF32.4.5 |
| SCR-MGMT-loyalty-dashboard | alias | SCR-FIN-loyalty-liability | liability and breakage dashboard — used by: M30.F30.1.SF30.1.7 |
| SCR-MGMT-maintenance-cost | alias | SCR-GM-maintenance-sla | maintenance cost tab — used by: M32.F32.2.SF32.2.5 |
| SCR-MGMT-occupancy | alias | SCR-GM-occupancy-forecast | actuals tab — used by: M32.F32.1.SF32.1.1 |
| SCR-MGMT-outlet-pnl | alias | SCR-GM-department-pnl | outlets (bar/club/catering) — used by: M32.F32.2.SF32.2.2 |
| SCR-MGMT-parking-pnl | alias | SCR-GM-department-pnl | parking department — used by: M32.F32.2.SF32.2.3 |
| SCR-MGMT-pickup-pace | alias | SCR-GM-revenue-pickup-pace | same purpose — used by: M32.F32.1.SF32.1.3 |
| SCR-MGMT-pivot-builder | alias | SCR-GM-custom-pivot | same purpose — used by: M32.F32.4.SF32.4.8 |
| SCR-MGMT-profit-bridge | alias | SCR-GM-profit-bridge | same purpose — used by: M32.F32.3.SF32.3.2 |
| SCR-MGMT-purchasing-report | alias | SCR-PRC-procurement-reports | same purpose — used by: M32.F32.4.SF32.4.5 |
| SCR-MGMT-recipe-variance | alias | SCR-FNB-theoretical-vs-actual | same purpose — used by: M32.F32.2.SF32.2.2 |
| SCR-MGMT-report-schedules | alias | SCR-GM-scheduled-reports | same purpose — used by: M32.F32.4.SF32.4.8 |
| SCR-MGMT-revenue-report | alias | SCR-GM-report-catalogue | revenue report — used by: M32.F32.4.SF32.4.3 |
| SCR-MGMT-room-kpis | alias | SCR-GM-flash | room KPIs (ADR/RevPAR/TRevPAR) — used by: M32.F32.1.SF32.1.2 |
| SCR-MGMT-utilities-report | alias | SCR-GM-utilities-overview | same purpose — used by: M32.F32.4.SF32.4.4 |
| SCR-MGMT-utility-cost-allocation | alias | SCR-GM-utilities-overview | allocation by meter tab — used by: M32.F32.2.SF32.2.4 |
| SCR-MGR-alerts | alias | SCR-OPS-unified-inbox | manager escalation items — used by: M45.F45.2.SF45.2.9,M46.F46.1.SF46.1.6,M50.F50.1.SF50.1.6 |
| SCR-MGR-coverage-alert | alias | SCR-FNB-chef-coverage-plan | coverage gap alert — used by: M47.F47.1.SF47.1.3,M47.F47.2.SF47.2.4,M47.F47.2.SF47.2.6,M47.F47.2.SF47.2.8 |
| SCR-MGR-procurement-dashboard | alias | SCR-PRC-procurement-reports | same purpose — used by: M50.F50.4.SF50.4.2,M50.F50.4.SF50.4.3 |
| SCR-MGT-event-pnl | alias | SCR-GM-department-pnl | events view — used by: M12.F12.5.SF12.5.5,M16.F16.5.SF16.5.3 |
| SCR-MGT-outlet-margin | alias | SCR-GM-department-pnl | outlet margin view — used by: M14.F14.5.SF14.5.5 |
| SCR-MGT-sales-pipeline | alias | SCR-GM-sales-pipeline | same purpose — used by: M10.F10.4.SF10.4.5 |
| SCR-MKT-abandonment-recovery | alias | SCR-MED-campaign-editor | abandoned-checkout recovery journey — used by: M51.F51.2.SF51.2.2 |
| SCR-MKT-acquisition-sources | alias | SCR-MED-attribution | acquisition sources tab — used by: M51.F51.2.SF51.2.3 |
| SCR-MKT-attribution | alias | SCR-MED-attribution | same purpose — used by: M07.F07.5.SF07.5.6 |
| SCR-MKT-attribution-report | alias | SCR-MED-attribution | same purpose — used by: M51.F51.2.SF51.2.5 |
| SCR-MKT-campaign-results | alias | SCR-MED-campaign-results | same purpose — used by: M52.F52.1.SF52.1.6 |
| SCR-MKT-campaign-send | alias | SCR-MED-campaign-editor | send step — used by: M52.F52.1.SF52.1.5 |
| SCR-MKT-claim-review | alias | SCR-MED-approval | sustainability claim check — used by: M67.F67.2.SF67.2.4 |
| SCR-MKT-delivery-log | alias | SCR-MED-campaign-results | delivery log tab — used by: M52.F52.1.SF52.1.5 |
| SCR-MKT-disputes | new | SCR-FIN-marketplace-payouts | disputes, fraud and refunds tab — used by: M37.F37.1.SF37.1.5 |
| SCR-MKT-frequency-caps | alias | SCR-MED-campaign-editor | frequency caps — used by: M52.F52.1.SF52.1.4 |
| SCR-MKT-funnel | alias | SCR-MED-attribution | funnel tab — used by: M07.F07.1.SF07.1.6,M07.F07.5.SF07.5.4,M51.F51.2.SF51.2.8 |
| SCR-MKT-guest-360 | alias | SCR-FO-guest-profile | marketing 360 view — used by: M52.F52.1.SF52.1.1,M52.F52.1.SF52.1.2 |
| SCR-MKT-listing-review | new | SCR-MED-marketplace-listings | defined in §2 — used by: M37.F37.1.SF37.1.1 |
| SCR-MKT-merge-review | alias | SCR-FO-guest-merge-review | same purpose — used by: M52.F52.1.SF52.1.1 |
| SCR-MKT-offer-targeting | alias | SCR-MED-offers-vouchers | targeting tab — used by: M54.F54.1.SF54.1.4 |
| SCR-MKT-package-builder | alias | SCR-MED-offers-vouchers | dynamic packages (M37, Later) — used by: M37.F37.1.SF37.1.3 |
| SCR-MKT-recovery-follow-up | alias | SCR-FO-guest-case | follow-up and closure — used by: M52.F52.2.SF52.2.4 |
| SCR-MKT-repeat-guests | alias | SCR-MED-segments | repeat guest and lifetime value view — used by: M52.F52.1.SF52.1.7 |
| SCR-MKT-reputation-trends | alias | SCR-MED-reviews-inbox | trends tab — used by: M52.F52.2.SF52.2.5 |
| SCR-MKT-review-inbox | alias | SCR-MED-reviews-inbox | same purpose — used by: M52.F52.2.SF52.2.2 |
| SCR-MKT-review-policy | alias | SCR-MED-review-response | review policy panel — used by: M52.F52.2.SF52.2.6 |
| SCR-MKT-review-response | alias | SCR-MED-review-response | same purpose — used by: M52.F52.2.SF52.2.3 |
| SCR-MKT-search | new | SCR-GST-marketplace-search | defined in §2 — used by: M37.F37.1.SF37.1.2 |
| SCR-MKT-segment-builder | alias | SCR-MED-segments | same purpose — used by: M52.F52.1.SF52.1.4 |
| SCR-MKT-supplier-directory | alias | SCR-PRC-vendor-directory | cross-hotel marketplace scope (M37, Later) — used by: M37.F37.3.SF37.3.2 |
| SCR-MKT-suppression | alias | SCR-ADM-consent-purposes | suppression list tab — used by: M52.F52.1.SF52.1.3 |
| SCR-MKT-survey-results | alias | SCR-MED-surveys | same purpose — used by: M52.F52.2.SF52.2.1 |
| SCR-MKT-templates | alias | SCR-ADM-notification-templates | marketing templates — used by: M52.F52.1.SF52.1.5 |
| SCR-MKT-traffic-quality | alias | SCR-MED-attribution | bot filtering tab — used by: M51.F51.2.SF51.2.6 |
| SCR-MOB-approvals | alias | SCR-OPS-approvals-queue | same purpose — used by: M02.F02.4.SF02.4.3 |
| SCR-OPS-amenity-closure | alias | SCR-FNB-amenity-bookings | closure and re-accommodation action — used by: M58.F58.2.SF58.2.5 |
| SCR-OPS-anomaly-feedback | alias | SCR-FIN-anomaly-case | false-positive feedback — used by: M60.F60.2.SF60.2.6 |
| SCR-OPS-anomaly-queue | alias | SCR-FIN-revenue-protection | same purpose — used by: M60.F60.2.SF60.2.1,M60.F60.2.SF60.2.2,M60.F60.2.SF60.2.4 |
| SCR-OPS-approval-queue | alias | SCR-OPS-approvals-queue | same purpose — used by: M60.F60.1.SF60.1.2,M63.F63.1.SF63.1.4 |
| SCR-OPS-arrivals | alias | SCR-FO-arrivals | pre-arrival and assisted check-in readiness — used by: M55.F55.1.SF55.1.2 |
| SCR-OPS-audit-differences | alias | SCR-FO-night-audit-exceptions | same purpose — used by: M60.F60.1.SF60.1.4 |
| SCR-OPS-audit-search | alias | SCR-ADM-audit-log | same purpose — used by: M01.F01.6.SF01.6.5,M02.F02.3.SF02.3.6,M06.F06.3.SF06.3.4,M06.F06.4.SF06.4.6 |
| SCR-OPS-backups | alias | SCR-ADM-backup-restore | same purpose — used by: M01.F01.4.SF01.4.1 |
| SCR-OPS-compensation | alias | SCR-FO-guest-case | compensation options tab — used by: M55.F55.2.SF55.2.3 |
| SCR-OPS-containment | alias | SCR-SAF-nonconformance | containment actions — used by: M61.F61.2.SF61.2.2 |
| SCR-OPS-corrective-actions | alias | SCR-SAF-nonconformance | corrective actions tab — used by: M61.F61.2.SF61.2.3 |
| SCR-OPS-coverage | alias | SCR-OPS-shift-handover | coverage panel — used by: M62.F62.2.SF62.2.1 |
| SCR-OPS-cylinder-exchange | alias | SCR-PRC-cylinder-exchange | same purpose — used by: M25.F25.1.SF25.1.2,M25.F25.1.SF25.1.4 |
| SCR-OPS-dead-letters | alias | SCR-ADM-dead-letters | workflow dead letters — used by: M63.F63.1.SF63.1.3 |
| SCR-OPS-deployment-status | alias | SCR-ADM-deployment-status | same purpose — used by: M01.F01.3.SF01.3.1,M01.F01.3.SF01.3.2 |
| SCR-OPS-devices | alias | SCR-ADM-devices | same purpose — used by: M01.F01.6.SF01.6.4 |
| SCR-OPS-dr-status | alias | SCR-ADM-backup-restore | DR status tab — used by: M01.F01.4.SF01.4.3 |
| SCR-OPS-escalations | alias | SCR-OPS-case-timeline | escalate/pause/cancel actions — used by: M63.F63.1.SF63.1.5 |
| SCR-OPS-findings | alias | SCR-SAF-nonconformance | same purpose — used by: M61.F61.2.SF61.2.1,M61.F61.2.SF61.2.5 |
| SCR-OPS-guest-cases | alias | SCR-FO-guest-case | same purpose — used by: M52.F52.2.SF52.2.4,M55.F55.2.SF55.2.1,M55.F55.2.SF55.2.4,M56.F56.2.SF56.2.6 |
| SCR-OPS-guest-inbox | alias | SCR-FO-guest-inbox | same purpose — used by: M55.F55.1.SF55.1.1,M55.F55.1.SF55.1.7 |
| SCR-OPS-guest-welfare | alias | SCR-SAF-continuity-plans | guest welfare and relocation tab — used by: M68.F68.2.SF68.2.3 |
| SCR-OPS-health | new | SCR-ADM-observability | defined in §2 — used by: M01.F01.6.SF01.6.2 |
| SCR-OPS-incident | alias | SCR-SAF-incident-report | same purpose — used by: M24.F24.2.SF24.2.1,M57.F57.2.SF57.2.6,M61.F61.2.SF61.2.5,M68.F68.1.SF68.1.3 |
| SCR-OPS-inspection-calendar | alias | SCR-SAF-inspections | same purpose — used by: M61.F61.1.SF61.1.2 |
| SCR-OPS-inspection-export | alias | SCR-SAF-inspections | inspector-ready export — used by: M61.F61.1.SF61.1.5 |
| SCR-OPS-investigations | alias | SCR-FIN-anomaly-case | same purpose — used by: M60.F60.2.SF60.2.5 |
| SCR-OPS-isolation-test-report | new | SCR-ADM-observability | tenant isolation test tab — used by: M01.F01.1.SF01.1.5 |
| SCR-OPS-jobs | alias | SCR-ADM-job-monitor | same purpose — used by: M03.F03.4.SF03.4.2 |
| SCR-OPS-labor-demand | alias | SCR-HR-labor-forecast | same purpose — used by: M62.F62.2.SF62.2.2 |
| SCR-OPS-leads | alias | SCR-GM-sales-pipeline | inquiry leads stage — used by: M55.F55.1.SF55.1.7 |
| SCR-OPS-manual-procedures | alias | SCR-OPS-outage-mode | same purpose — used by: M64.F64.1.SF64.1.5 |
| SCR-OPS-migrations | alias | SCR-ADM-deployment-status | migrations tab — used by: M01.F01.3.SF01.3.5 |
| SCR-OPS-my-work | alias | SCR-OPS-unified-inbox | same purpose — used by: M63.F63.1.SF63.1.2 |
| SCR-OPS-observability | new | SCR-ADM-observability | logs and traces tab — used by: M01.F01.6.SF01.6.1 |
| SCR-OPS-offer-accuracy | alias | SCR-MED-listing-preview | live offer accuracy check panel — used by: M51.F51.1.SF51.1.6 |
| SCR-OPS-onprem-health | new | SCR-ADM-observability | on-prem profile view — used by: M01.F01.3.SF01.3.2 |
| SCR-OPS-permit-calendar | alias | SCR-SAF-permit-calendar | same purpose — used by: M61.F61.1.SF61.1.4 |
| SCR-OPS-price-discrepancies | alias | SCR-GM-rate-publish-status | price discrepancy queue — used by: M53.F53.2.SF53.2.5 |
| SCR-OPS-quality-samples | alias | SCR-HR-quality-sampling | same purpose — used by: M62.F62.2.SF62.2.3 |
| SCR-OPS-queue-depth | new | SCR-ADM-observability | store-and-forward queue depth — used by: M01.F01.4.SF01.4.4 |
| SCR-OPS-recall | alias | SCR-PRC-recall-lookup | same purpose — used by: M57.F57.2.SF57.2.4 |
| SCR-OPS-recovery-report | alias | SCR-GM-report-catalogue | service recovery report — used by: M55.F55.2.SF55.2.5 |
| SCR-OPS-release | alias | SCR-SAF-nonconformance | retest and independent release — used by: M61.F61.2.SF61.2.4 |
| SCR-OPS-relocation | alias | SCR-SAF-continuity-plans | guest relocation tab — used by: M68.F68.2.SF68.2.3 |
| SCR-OPS-reorder-suggestions | alias | SCR-PRC-replenishment | same purpose — used by: M56.F56.2.SF56.2.5 |
| SCR-OPS-request-board | alias | SCR-FO-service-requests | same purpose — used by: M55.F55.1.SF55.1.3,M55.F55.1.SF55.1.4 |
| SCR-OPS-restore-tests | alias | SCR-ADM-backup-restore | restore test evidence — used by: M01.F01.4.SF01.4.2 |
| SCR-OPS-retail-stock | alias | SCR-FNB-outlet-stock | amenity retail outlet — used by: M58.F58.2.SF58.2.1 |
| SCR-OPS-supplier-evidence | alias | SCR-PRC-vendor-detail | sustainability evidence tab — used by: M67.F67.2.SF67.2.5 |
| SCR-OPS-support-access | alias | SCR-ADM-break-glass | vendor support access — used by: M01.F01.3.SF01.3.6 |
| SCR-OPS-sync-conflicts | alias | SCR-OPS-sync-queue | same purpose — used by: M64.F64.2.SF64.2.2 |
| SCR-OPS-time-sync-health | new | SCR-ADM-observability | time sync panel — used by: M01.F01.2.SF01.2.1 |
| SCR-OPS-transport-dispatch | alias | SCR-CON-fleet-dispatch | same purpose — used by: M59.F59.1.SF59.1.1,M59.F59.1.SF59.1.2,M59.F59.2.SF59.2.1 |
| SCR-OPS-transport-exceptions | alias | SCR-CON-fleet-dispatch | missed pickup/disruption exceptions — used by: M59.F59.1.SF59.1.5 |
| SCR-OPS-transport-incident | alias | SCR-SAF-incident-detail | transport incident type — used by: M59.F59.2.SF59.2.5 |
| SCR-OPS-trip-manifest | alias | SCR-CON-trip-manifest | same purpose — used by: M59.F59.1.SF59.1.3 |
| SCR-OPS-welfare-escalations | alias | SCR-FO-guest-case | welfare/accessibility escalation type — used by: M55.F55.2.SF55.2.6 |
| SCR-OWN-acquisition-cost | alias | SCR-GM-channel-performance | acquisition net cost view — used by: M51.F51.2.SF51.2.5 |
| SCR-OWN-amenity-contribution | alias | SCR-GM-department-pnl | amenities contribution — used by: M58.F58.2.SF58.2.5 |
| SCR-OWN-bia | alias | SCR-SAF-continuity-plans | business impact analysis tab — used by: M68.F68.2.SF68.2.1 |
| SCR-OWN-brand-standards | alias | SCR-SAF-inspections | brand standards checklist — used by: M66.F66.2.SF66.2.5 |
| SCR-OWN-budget-vs-actual | alias | SCR-GM-budget-vs-actual | same purpose — used by: M65.F65.2.SF65.2.5 |
| SCR-OWN-capex-requests | alias | SCR-GM-capex-requests | same purpose — used by: M66.F66.2.SF66.2.1 |
| SCR-OWN-contact-plan-tests | alias | SCR-SAF-drills | contact reachability tests — used by: M68.F68.2.SF68.2.6 |
| SCR-OWN-continuity-plans | alias | SCR-SAF-continuity-plans | same purpose — used by: M68.F68.2.SF68.2.4 |
| SCR-OWN-crisis-roles | alias | SCR-SAF-continuity-plans | crisis role tree tab — used by: M68.F68.2.SF68.2.2 |
| SCR-OWN-daily-flash | alias | SCR-GM-flash | same purpose — used by: M65.F65.2.SF65.2.3 |
| SCR-OWN-data-health | alias | SCR-GM-data-coverage | same purpose — used by: M65.F65.1.SF65.1.5 |
| SCR-OWN-drilldown | alias | SCR-GM-flash-drilldown | same purpose — used by: M65.F65.1.SF65.1.3,M65.F65.2.SF65.2.3 |
| SCR-OWN-drills | alias | SCR-SAF-drills | same purpose — used by: M68.F68.2.SF68.2.4 |
| SCR-OWN-exceptions-overview | alias | SCR-GM-owner-home | exceptions panel — used by: M63.F63.1.SF63.1.2,M63.F63.2.SF63.2.5 |
| SCR-OWN-forecast-summary | alias | SCR-GM-occupancy-forecast | same purpose — used by: M53.F53.1.SF53.1.1,M53.F53.1.SF53.1.5 |
| SCR-OWN-guest-value | alias | SCR-MED-segments | repeat guest and lifetime value view — used by: M52.F52.1.SF52.1.7 |
| SCR-OWN-insurance-summary | alias | SCR-SAF-insurance-register | deductible/reserve/settlement summary — used by: M68.F68.1.SF68.1.5 |
| SCR-OWN-owner-statement | alias | SCR-GM-owner-statement | same purpose — used by: M66.F66.1.SF66.1.3 |
| SCR-OWN-portfolio | alias | SCR-GM-owner-home | portfolio view — used by: M66.F66.2.SF66.2.6 |
| SCR-OWN-recovery-evidence | alias | SCR-SAF-drills | recovery evidence — used by: M68.F68.2.SF68.2.5 |
| SCR-OWN-relationships | alias | SCR-GM-owner-statement | legal relationships tab — used by: M66.F66.1.SF66.1.1 |
| SCR-OWN-renovation-plan | alias | SCR-ENG-capex-project | renovation closures — used by: M66.F66.2.SF66.2.2 |
| SCR-OWN-reopening-checklist | alias | SCR-ENG-capex-project | reopening checklist tab — used by: M66.F66.2.SF66.2.5 |
| SCR-OWN-reports | alias | SCR-GM-report-catalogue | same purpose — used by: M65.F65.2.SF65.2.1 |
| SCR-OWN-reputation | alias | SCR-MED-reviews-inbox | trends tab (read-only) — used by: M52.F52.2.SF52.2.5 |
| SCR-OWN-resource-intensity | alias | SCR-GM-sustainability-dashboard | same purpose — used by: M67.F67.1.SF67.1.3 |
| SCR-OWN-savings-measurement | alias | SCR-GM-sustainability-dashboard | capex savings measurement tab — used by: M67.F67.2.SF67.2.3 |
| SCR-OWN-scheduled-reports | alias | SCR-GM-scheduled-reports | same purpose — used by: M65.F65.2.SF65.2.4 |
| SCR-OWN-service-quality | alias | SCR-GM-report-catalogue | service quality report — used by: M55.F55.2.SF55.2.5,M62.F62.2.SF62.2.5 |
| SCR-OWN-sustainability-export | alias | SCR-GM-sustainability-dashboard | assurance export — used by: M67.F67.2.SF67.2.4 |
| SCR-OWN-sustainability-targets | alias | SCR-ENG-sustainability-baseline | targets tab — used by: M67.F67.2.SF67.2.1 |
| SCR-OWN-transport-margin | alias | SCR-GM-department-pnl | transport department — used by: M59.F59.2.SF59.2.4 |
| SCR-OWN-waste-laundry | alias | SCR-GM-sustainability-dashboard | food waste and laundry tab — used by: M67.F67.1.SF67.1.4 |
| SCR-PARK-capacity | alias | SCR-PRK-occupancy | capacity windows — used by: M09.F09.2.SF09.2.5 |
| SCR-PARK-device-health | alias | SCR-PRK-device-status | same purpose — used by: M17.F17.2.SF17.2.5 |
| SCR-PARK-lane-live | alias | SCR-PRK-lane-monitor | same purpose — used by: M17.F17.2.SF17.2.2,M17.F17.3.SF17.3.1,M17.F17.3.SF17.3.2 |
| SCR-PARK-manual-lane | alias | SCR-PRK-manual-gate | same purpose — used by: M17.F17.3.SF17.3.4 |
| SCR-PARK-occupancy | alias | SCR-PRK-occupancy | same purpose — used by: M17.F17.3.SF17.3.5 |
| SCR-PARK-pay-station | new | SCR-PRK-pay-station | defined in §2 — used by: M17.F17.4.SF17.4.4 |
| SCR-PARK-permits | alias | SCR-PRK-permits | same purpose — used by: M17.F17.1.SF17.1.2 |
| SCR-PARK-reconciliation | alias | SCR-PRK-reconciliation | same purpose — used by: M17.F17.4.SF17.4.5 |
| SCR-PARK-review-queue | alias | SCR-PRK-lane-review | same purpose — used by: M17.F17.2.SF17.2.4 |
| SCR-PARK-sessions | alias | SCR-PRK-sessions | same purpose — used by: M17.F17.3.SF17.3.3,M17.F17.4.SF17.4.1 |
| SCR-PARK-valet-queue | new | SCR-PRK-valet-queue | defined in §2 — used by: M18.F18.3.SF18.3.5 |
| SCR-POS-comp | alias | SCR-FNB-pos-void-comp-approval | comp — used by: M13.F13.4.SF13.4.3 |
| SCR-POS-day-close | alias | SCR-FNB-pos-shift-close | outlet day close and GL export — used by: M13.F13.5.SF13.5.4 |
| SCR-POS-discount | alias | SCR-FNB-pos-order | discount action (over-limit via SCR-FNB-pos-void-comp-approval) — used by: M13.F13.4.SF13.4.1 |
| SCR-POS-hosted-tab | alias | SCR-FNB-pos-tab | hosted bar tab — used by: M12.F12.4.SF12.4.2,M15.F15.4.SF15.4.2 |
| SCR-POS-master-charge | alias | SCR-FNB-pos-payment | corporate/event master charge — used by: M13.F13.3.SF13.3.3 |
| SCR-POS-order | alias | SCR-FNB-pos-order | same purpose — used by: M13.F13.1.SF13.1.5,M13.F13.2.SF13.2.2,M13.F13.2.SF13.2.5 |
| SCR-POS-payment | alias | SCR-FNB-pos-payment | same purpose — used by: M13.F13.3.SF13.3.1 |
| SCR-POS-receipt | alias | SCR-FNB-pos-payment | receipt and fiscal output — used by: M13.F13.3.SF13.3.5 |
| SCR-POS-refund | alias | SCR-FNB-pos-payment | refund on closed check — used by: M13.F13.4.SF13.4.4 |
| SCR-POS-room-charge | alias | SCR-FNB-pos-payment | room charge — used by: M13.F13.3.SF13.3.2 |
| SCR-POS-shift | alias | SCR-FNB-pos-shift-close | shift open and float — used by: M13.F13.5.SF13.5.1 |
| SCR-POS-shift-close | alias | SCR-FNB-pos-shift-close | same purpose — used by: M13.F13.5.SF13.5.2 |
| SCR-POS-split | alias | SCR-FNB-pos-order | transfer, merge and split checks — used by: M13.F13.2.SF13.2.4 |
| SCR-POS-sync-status | alias | SCR-OPS-sync-queue | POS store-and-forward — used by: M13.F13.5.SF13.5.3 |
| SCR-POS-table-plan | alias | SCR-FNB-pos-tables | same purpose — used by: M09.F09.2.SF09.2.2,M13.F13.2.SF13.2.1 |
| SCR-POS-tabs | alias | SCR-FNB-pos-tab | same purpose — used by: M13.F13.2.SF13.2.1,M15.F15.4.SF15.4.1 |
| SCR-POS-tender-points | alias | SCR-FNB-pos-payment | points tender — used by: M30.F30.1.SF30.1.6 |
| SCR-POS-tips | new | SCR-FIN-tips-commissions | tips and service charge distribution — used by: M13.F13.3.SF13.3.4 |
| SCR-POS-void | alias | SCR-FNB-pos-void-comp-approval | void — used by: M13.F13.4.SF13.4.2 |
| SCR-PRC-bid-compare | alias | SCR-PRC-rfq-comparison | same purpose — used by: M49.F49.2.SF49.2.1,M49.F49.2.SF49.2.2,M49.F49.2.SF49.2.4,M49.F49.2.SF49.2.5,M49.F49.2.SF49.2.6 |
| SCR-PRC-catalog-search | alias | SCR-FNB-ingredient-catalog-search | same purpose — used by: M48.F48.2.SF48.2.7,M48.F48.3.SF48.3.1,M48.F48.3.SF48.3.2,M48.F48.3.SF48.3.4,M48.F48.3.SF48.3.6,M48.F48.3.SF48.3.8 |
| SCR-PRC-crosswalk | alias | SCR-PRC-item-master | vendor SKU crosswalk tab — used by: M48.F48.2.SF48.2.1 |
| SCR-PRC-exception-queue | alias | SCR-PRC-ai-followup-queue | exceptions tab (no response, late, stock-out) — used by: M50.F50.1.SF50.1.5 |
| SCR-PRC-followup-queue | alias | SCR-PRC-ai-followup-queue | same purpose — used by: M50.F50.1.SF50.1.1,M50.F50.1.SF50.1.3,M50.F50.1.SF50.1.4,M50.F50.1.SF50.1.7 |
| SCR-PRC-policy-config | alias | SCR-ADM-rfq-policy | same purpose — used by: M49.F49.2.SF49.2.2 |
| SCR-PRC-requisition | alias | SCR-PRC-requisition-new | same purpose — used by: M49.F49.1.SF49.1.1,M49.F49.1.SF49.1.2,M49.F49.1.SF49.1.8 |
| SCR-PRC-retention-console | alias | SCR-PRC-sample-gallery | retention and purge proof — used by: M49.F49.1.SF49.1.4,M49.F49.3.SF49.3.8,M50.F50.4.SF50.4.5 |
| SCR-PRC-shortlist | alias | SCR-PRC-vendor-directory | shortlist — used by: M46.F46.2.SF46.2.4 |
| SCR-PRC-taxonomy-admin | alias | SCR-ADM-service-taxonomy | same purpose — used by: M46.F46.1.SF46.1.2 |
| SCR-PRC-vendor-approval-queue | alias | SCR-PRC-vendor-review | same purpose — used by: M46.F46.1.SF46.1.4,M46.F46.1.SF46.1.7 |
| SCR-PRC-vendor-profile | alias | SCR-PRC-vendor-detail | same purpose — used by: M45.F45.3.SF45.3.2,M45.F45.3.SF45.3.5,M46.F46.1.SF46.1.1,M46.F46.1.SF46.1.3,M46.F46.1.SF46.1.6,M46.F46.2.SF46.2.3 |
| SCR-PRC-waiver | alias | SCR-PRC-rfq-waiver | same purpose — used by: M49.F49.1.SF49.1.6 |
| SCR-PROC-blanket-agreements | alias | SCR-PRC-contracts-blanket | same purpose — used by: M21.F21.3.SF21.3.2 |
| SCR-PROC-emergency-purchase | alias | SCR-PRC-rfq-waiver | emergency retrospective approval — used by: M21.F21.1.SF21.1.6 |
| SCR-PROC-po-detail | alias | SCR-PRC-po-detail | same purpose — used by: M21.F21.1.SF21.1.5,M21.F21.2.SF21.2.4 |
| SCR-PROC-policy | alias | SCR-ADM-rfq-policy | procurement thresholds — used by: M21.F21.3.SF21.3.1 |
| SCR-PROC-requisition | alias | SCR-PRC-requisition-approval | budget check and encumbrance — used by: M20.F20.4.SF20.4.2,M21.F21.1.SF21.1.1,M21.F21.1.SF21.1.2,M21.F21.1.SF21.1.3 |
| SCR-PROC-rfq-comparison | alias | SCR-PRC-rfq-comparison | same purpose — used by: M21.F21.1.SF21.1.4 |
| SCR-PROC-supplier-claims | alias | SCR-PRC-return-to-vendor | supplier claims and disputes — used by: M21.F21.2.SF21.2.3,M21.F21.3.SF21.3.5 |
| SCR-RCV-dock-scan | alias | SCR-PRC-receiving | same purpose — used by: M50.F50.2.SF50.2.2 |
| SCR-RCV-draft-grn | alias | SCR-PRC-receiving | draft GRN — used by: M50.F50.2.SF50.2.3,M50.F50.2.SF50.2.5,M50.F50.2.SF50.2.7,M50.F50.2.SF50.2.8 |
| SCR-RCV-food-verification | alias | SCR-PRC-receiving-verify | same purpose — used by: M50.F50.2.SF50.2.4 |
| SCR-RCV-gate-checkin | alias | SCR-PRC-receiving | gate check-in — used by: M50.F50.2.SF50.2.1,M50.F50.2.SF50.2.2 |
| SCR-RCV-quarantine | alias | SCR-PRC-quarantine | same purpose — used by: M50.F50.2.SF50.2.6 |
| SCR-RCV-rtv | alias | SCR-PRC-return-to-vendor | same purpose — used by: M50.F50.2.SF50.2.9 |
| SCR-REV-abandonment-reasons | alias | SCR-MED-attribution | quote abandonment funnel — used by: M51.F51.2.SF51.2.8 |
| SCR-REV-approval-queue | alias | SCR-GM-rate-recommendation | approval queue — used by: M53.F53.2.SF53.2.2 |
| SCR-REV-automation-settings | alias | SCR-GM-rate-recommendation | auto-apply guardrail settings — used by: M53.F53.2.SF53.2.7 |
| SCR-REV-backtest | alias | SCR-GM-revenue-forecast | backtest tab — used by: M53.F53.2.SF53.2.6 |
| SCR-REV-channel-net-contribution | alias | SCR-GM-channel-performance | same purpose — used by: M51.F51.2.SF51.2.5,M53.F53.1.SF53.1.3 |
| SCR-REV-channel-performance | alias | SCR-GM-channel-performance | same purpose — used by: M07.F07.5.SF07.5.5 |
| SCR-REV-child-policy | alias | SCR-ADM-rate-plan-setup | child and occupancy rules — used by: M04.F04.2.SF04.2.2 |
| SCR-REV-demand-calendar | alias | SCR-GM-revenue-forecast | demand calendar — used by: M53.F53.1.SF53.1.4 |
| SCR-REV-forecast | alias | SCR-GM-revenue-forecast | same purpose — used by: M53.F53.1.SF53.1.5 |
| SCR-REV-guardrails | alias | SCR-GM-rate-recommendation | guardrails — used by: M53.F53.2.SF53.2.2 |
| SCR-REV-inventory-audit | alias | SCR-FO-availability-grid | integrity audit and recompute — used by: M03.F03.2.SF03.2.4 |
| SCR-REV-inventory-calendar | alias | SCR-FO-availability-grid | same purpose — used by: M03.F03.2.SF03.2.1,M03.F03.2.SF03.2.5 |
| SCR-REV-market-inputs | alias | SCR-GM-revenue-forecast | market inputs tab — used by: M53.F53.1.SF53.1.4 |
| SCR-REV-offer-compliance | alias | SCR-MED-offers-vouchers | compliance review — used by: M54.F54.2.SF54.2.4 |
| SCR-REV-offer-performance | alias | SCR-MED-offers-vouchers | performance tab — used by: M54.F54.1.SF54.1.6 |
| SCR-REV-overbooking | alias | SCR-GM-overbooking-control | same purpose — used by: M03.F03.4.SF03.4.5,M03.F03.4.SF03.4.6,M03.F03.5.SF03.5.6,M07.F07.4.SF07.4.3,M07.F07.5.SF07.5.5 |
| SCR-REV-overbooking-risk | alias | SCR-GM-overbooking-control | risk estimate — used by: M53.F53.1.SF53.1.6 |
| SCR-REV-pace-grid | alias | SCR-GM-revenue-pickup-pace | same purpose — used by: M53.F53.1.SF53.1.1 |
| SCR-REV-package-builder | alias | SCR-ADM-rate-plan-setup | packages tab — used by: M54.F54.2.SF54.2.2 |
| SCR-REV-packages | alias | SCR-ADM-rate-plan-setup | packages tab — used by: M04.F04.3.SF04.3.1 |
| SCR-REV-pickup-report | alias | SCR-GM-revenue-pickup-pace | same purpose — used by: M53.F53.1.SF53.1.1 |
| SCR-REV-promo-rules | new | SCR-GM-promotions | eligibility and caps tab — used by: M54.F54.2.SF54.2.6 |
| SCR-REV-promotions | new | SCR-GM-promotions | defined in §2 — used by: M04.F04.3.SF04.3.2,M04.F04.3.SF04.3.3 |
| SCR-REV-publication-status | alias | SCR-GM-rate-publish-status | same purpose — used by: M53.F53.2.SF53.2.4 |
| SCR-REV-quote-audit | alias | SCR-OPS-record-history | quote snapshot audit — used by: M04.F04.5.SF04.5.8 |
| SCR-REV-rate-grid | alias | SCR-GM-rate-grid | same purpose — used by: M04.F04.1.SF04.1.3,M07.F07.3.SF07.3.5 |
| SCR-REV-rate-history | alias | SCR-GM-rate-grid | rate history and rollback — used by: M53.F53.2.SF53.2.5 |
| SCR-REV-rate-plan-detail | alias | SCR-ADM-rate-plan-setup | same purpose — used by: M04.F04.1.SF04.1.1,M04.F04.1.SF04.1.2,M04.F04.1.SF04.1.5,M04.F04.2.SF04.2.1,M04.F04.2.SF04.2.3 |
| SCR-REV-rate-plans | alias | SCR-ADM-rate-plan-setup | same purpose — used by: M04.F04.1.SF04.1.1 |
| SCR-REV-recommendations | alias | SCR-GM-rate-recommendation | same purpose — used by: M53.F53.2.SF53.2.1 |
| SCR-REV-renovation-impact | alias | SCR-GM-revenue-forecast | renovation displacement — used by: M66.F66.2.SF66.2.2 |
| SCR-REV-restrictions | alias | SCR-GM-restrictions-calendar | same purpose — used by: M03.F03.4.SF03.4.3,M04.F04.2.SF04.2.4 |
| SCR-REV-simulator | alias | SCR-GM-rate-recommendation | simulate — used by: M53.F53.2.SF53.2.3 |
| SCR-REV-source-codes | alias | SCR-ADM-rate-plan-setup | source, channel and segment codes tab — used by: M07.F07.5.SF07.5.1 |
| SCR-REV-supply-and-wash | alias | SCR-GM-revenue-forecast | supply and wash tab — used by: M53.F53.1.SF53.1.2 |
| SCR-SAFETY-claim-verification | alias | SCR-SAF-lost-claims | same purpose — used by: M43.F43.1.SF43.1.5 |
| SCR-SAFETY-dispatch | alias | SCR-SAF-incident-command | dispatches and acknowledgements — used by: M42.F42.2.SF42.2.3,M42.F42.2.SF42.2.4 |
| SCR-SAFETY-disposal-queue | alias | SCR-SAF-lost-register | disposal/donation queue — used by: M43.F43.1.SF43.1.7 |
| SCR-SAFETY-drills | alias | SCR-SAF-drills | same purpose — used by: M42.F42.2.SF42.2.7 |
| SCR-SAFETY-incident-board | alias | SCR-SAF-home | same purpose — used by: M40.F40.2.SF40.2.6,M42.F42.1.SF42.1.1,M42.F42.2.SF42.2.1 |
| SCR-SAFETY-incident-detail | alias | SCR-SAF-incident-detail | same purpose — used by: M42.F42.1.SF42.1.4 |
| SCR-SAFETY-incident-timeline | alias | SCR-SAF-incident-detail | chronology — used by: M42.F42.2.SF42.2.6 |
| SCR-SAFETY-life-safety-register | alias | SCR-ENG-asset-registry | life-safety equipment class — used by: M42.F42.1.SF42.1.5 |
| SCR-SAFETY-lost-found-report | alias | SCR-SAF-lost-register | reports tab — used by: M43.F43.1.SF43.1.8 |
| SCR-SAFETY-lost-item-custody | alias | SCR-SAF-lost-custody | same purpose — used by: M43.F43.1.SF43.1.3 |
| SCR-SAFETY-matching | alias | SCR-SAF-lost-claims | same purpose — used by: M43.F43.1.SF43.1.4 |
| SCR-SAFETY-playbook-runner | alias | SCR-SAF-incident-command | same purpose — used by: M42.F42.2.SF42.2.2 |
| SCR-SAFETY-release | alias | SCR-SAF-lost-release | same purpose — used by: M43.F43.1.SF43.1.6 |
| SCR-SAFETY-reviews | alias | SCR-SAF-post-incident-review | same purpose — used by: M42.F42.2.SF42.2.7 |
| SCR-SAFETY-signal-review | alias | SCR-SAF-sensor-alert-review | same purpose — used by: M42.F42.1.SF42.1.3 |
| SCR-SAFETY-welfare | alias | SCR-SAF-incident-command | guest welfare panel — used by: M42.F42.2.SF42.2.5 |
| SCR-SALES-account-contacts | alias | SCR-GM-corporate-account-detail | contacts tab — used by: M10.F10.1.SF10.1.2 |
| SCR-SALES-account-detail | alias | SCR-GM-corporate-account-detail | same purpose — used by: M10.F10.1.SF10.1.1 |
| SCR-SALES-account-merge | alias | SCR-GM-corporate-account-detail | merge action — used by: M10.F10.1.SF10.1.4 |
| SCR-SALES-account-production | alias | SCR-GM-corporate-account-detail | production tab — used by: M10.F10.2.SF10.2.3 |
| SCR-SALES-agreement-approval | alias | SCR-GM-corporate-agreement-editor | approval and publish — used by: M10.F10.2.SF10.2.4 |
| SCR-SALES-agreement-editor | alias | SCR-GM-corporate-agreement-editor | same purpose — used by: M10.F10.2.SF10.2.1 |
| SCR-SALES-agreement-eligibility | alias | SCR-GM-corporate-agreement-editor | eligibility tab — used by: M10.F10.2.SF10.2.2 |
| SCR-SALES-agreement-rates | alias | SCR-GM-corporate-agreement-editor | rates tab — used by: M04.F04.1.SF04.1.4 |
| SCR-SALES-allotments | alias | SCR-GM-group-block | allotments with release-back — used by: M03.F03.4.SF03.4.4 |
| SCR-SALES-block-pickup | alias | SCR-GM-group-block | same purpose — used by: M05.F05.1.SF05.1.5 |
| SCR-SALES-booking-po | alias | SCR-GM-corporate-account-detail | purchase orders tab — used by: M10.F10.3.SF10.3.2 |
| SCR-SALES-composite-hold | new | SCR-GM-composite-holds | defined in §2 — used by: M09.F09.3.SF09.3.1,M09.F09.3.SF09.3.3,M09.F09.3.SF09.3.4,M09.F09.3.SF09.3.5 |
| SCR-SALES-contract | alias | SCR-GM-corporate-agreement-editor | contract and signature — used by: M10.F10.4.SF10.4.4 |
| SCR-SALES-feasibility-search | alias | SCR-GM-corporate-rfq-response | feasibility search panel — used by: M09.F09.2.SF09.2.1,M09.F09.2.SF09.2.5 |
| SCR-SALES-function-diary | alias | SCR-FNB-function-diary | same purpose — used by: M09.F09.4.SF09.4.2,M12.F12.2.SF12.2.1 |
| SCR-SALES-hold-list | new | SCR-GM-composite-holds | defined in §2 — used by: M09.F09.3.SF09.3.2 |
| SCR-SALES-messages | alias | SCR-GM-corporate-account-detail | messages tab — used by: M11.F11.4.SF11.4.3 |
| SCR-SALES-payment-schedule | new | SCR-FIN-deposits | deposit and payment schedule — used by: M10.F10.3.SF10.3.5 |
| SCR-SALES-pipeline | alias | SCR-GM-sales-pipeline | same purpose — used by: M10.F10.4.SF10.4.1 |
| SCR-SALES-proposal-editor | alias | SCR-GM-corporate-rfq-response | proposal — used by: M10.F10.4.SF10.4.3,M12.F12.2.SF12.2.3 |
| SCR-SALES-reaccommodate | alias | SCR-FNB-function-diary | displacement and re-accommodation — used by: M09.F09.4.SF09.4.3 |
| SCR-SALES-renewals | alias | SCR-GM-corporate-agreement-editor | renewals — used by: M10.F10.2.SF10.2.5 |
| SCR-SALES-rfq-inbox | alias | SCR-GM-corporate-rfq-response | RFQ inbox list — used by: M10.F10.4.SF10.4.2 |
| SCR-STAFF-ai-approvals | alias | SCR-OPS-approvals-queue | supervised AI action approvals (M37, Later) — used by: M37.F37.2.SF37.2.2 |
| SCR-STAFF-found-item-intake | alias | SCR-SAF-lost-intake | same purpose — used by: M43.F43.1.SF43.1.1,M43.F43.1.SF43.1.2 |
| SCR-STAFF-my-tax-slips | alias | SCR-HR-ess-payslip | tax slips tab — used by: M38.F38.2.SF38.2.4 |
| SCR-STAFF-report-incident | alias | SCR-SAF-incident-report | same purpose — used by: M42.F42.1.SF42.1.1 |
| SCR-STF-amenity-checkin | alias | SCR-FNB-amenity-bookings | check-in with entitlement check — used by: M58.F58.1.SF58.1.4 |
| SCR-STF-amenity-schedule | alias | SCR-FNB-amenity-bookings | practitioner/resource schedule view — used by: M58.F58.1.SF58.1.2 |
| SCR-STF-approvals | alias | SCR-OPS-approvals-queue | same purpose — used by: M63.F63.1.SF63.1.4 |
| SCR-STF-arrival-checklist | alias | SCR-FO-arrivals | readiness checklist — used by: M55.F55.1.SF55.1.2 |
| SCR-STF-attendee-scan | alias | SCR-FNB-event-detail | attendees tab, scan mode — used by: M12.F12.4.SF12.4.1 |
| SCR-STF-attestation | alias | SCR-HR-ess-training | SOP attestation — used by: M62.F62.1.SF62.1.3 |
| SCR-STF-batch | alias | SCR-FNB-production-plan | production batch — used by: M16.F16.3.SF16.3.5 |
| SCR-STF-beo-inbox | alias | SCR-FNB-beo-kitchen-view | same purpose — used by: M12.F12.3.SF12.3.3 |
| SCR-STF-callout-offer | alias | SCR-OPS-task-detail | internal callout offer task (accept/decline) — used by: M47.F47.2.SF47.2.1,M47.F47.2.SF47.2.2,M47.F47.2.SF47.2.3 |
| SCR-STF-case-detail | alias | SCR-FO-guest-case | same purpose — used by: M55.F55.2.SF55.2.1,M55.F55.2.SF55.2.2 |
| SCR-STF-cashier-shift | alias | SCR-FIN-cashier-shift | same purpose — used by: M60.F60.1.SF60.1.1 |
| SCR-STF-count-scan | alias | SCR-FNB-stock-count | same purpose — used by: M14.F14.5.SF14.5.3 |
| SCR-STF-course-player | alias | SCR-HR-ess-training | course player — used by: M62.F62.1.SF62.1.3 |
| SCR-STF-departures | alias | SCR-FO-departures | same purpose — used by: M55.F55.1.SF55.1.5 |
| SCR-STF-diary | alias | SCR-FNB-function-diary | same purpose — used by: M09.F09.4.SF09.4.2 |
| SCR-STF-dispatch | alias | SCR-FNB-catering-dispatch | same purpose — used by: M16.F16.4.SF16.4.2,M16.F16.4.SF16.4.3 |
| SCR-STF-driver-handover | alias | SCR-CON-driver-trip | passenger handover step — used by: M59.F59.1.SF59.1.6 |
| SCR-STF-driver-manifest | alias | SCR-CON-trip-manifest | same purpose — used by: M59.F59.1.SF59.1.3 |
| SCR-STF-driver-trips | alias | SCR-CON-driver-trip | same purpose — used by: M59.F59.1.SF59.1.1,M59.F59.1.SF59.1.4 |
| SCR-STF-early-late-requests | alias | SCR-FO-upsell-offers | early arrival/late checkout requests — used by: M54.F54.1.SF54.1.2 |
| SCR-STF-event-live | alias | SCR-FNB-event-detail | event-day live tab (consumption, incidents) — used by: M12.F12.4.SF12.4.3,M16.F16.5.SF16.5.1 |
| SCR-STF-guest-inbox | alias | SCR-FO-guest-inbox | same purpose — used by: M55.F55.1.SF55.1.1 |
| SCR-STF-guest-preferences | alias | SCR-FO-guest-profile | preferences tab — used by: M52.F52.1.SF52.1.2 |
| SCR-STF-inspection-checklist | alias | SCR-FO-hk-inspection | same purpose — used by: M56.F56.1.SF56.1.3 |
| SCR-STF-inspection-run | alias | SCR-SAF-inspection-run | same purpose — used by: M61.F61.1.SF61.1.3 |
| SCR-STF-intake-review | alias | SCR-FNB-amenity-bookings | intake/consent review — used by: M58.F58.2.SF58.2.2 |
| SCR-STF-ird-delivery | alias | SCR-FNB-in-room-dining | delivery confirmation — used by: M57.F57.1.SF57.1.2 |
| SCR-STF-issue-scan | alias | SCR-PRC-store-issue | same purpose — used by: M14.F14.3.SF14.3.4 |
| SCR-STF-key-status | new | SCR-FO-keys | digital key status — used by: M55.F55.3.SF55.3.1 |
| SCR-STF-kiosk-monitor | alias | SCR-FO-arrivals | kiosk sessions panel — used by: M55.F55.3.SF55.3.2 |
| SCR-STF-linen-move | alias | SCR-FO-hk-linen-par | linen custody moves — used by: M56.F56.2.SF56.2.1 |
| SCR-STF-minibar-count | alias | SCR-FO-hk-minibar-count | same purpose — used by: M56.F56.2.SF56.2.4 |
| SCR-STF-my-coaching | alias | SCR-HR-ess-training | my coaching feedback — used by: M62.F62.2.SF62.2.3,M62.F62.2.SF62.2.4 |
| SCR-STF-my-learning | alias | SCR-HR-ess-training | same purpose — used by: M62.F62.1.SF62.1.2 |
| SCR-STF-my-rooms | alias | SCR-FO-hk-my-rooms | same purpose — used by: M56.F56.1.SF56.1.1,M56.F56.1.SF56.1.2 |
| SCR-STF-my-shifts | alias | SCR-HR-ess-schedule | same purpose — used by: M47.F47.1.SF47.1.2 |
| SCR-STF-my-tasks | alias | SCR-OPS-unified-inbox | same purpose — used by: M63.F63.1.SF63.1.2 |
| SCR-STF-notification-settings | alias | SCR-OPS-notification-preferences | same purpose — used by: M63.F63.1.SF63.1.6 |
| SCR-STF-offline-arrivals | alias | SCR-FO-outage-registration | offline arrivals list — used by: M55.F55.1.SF55.1.6 |
| SCR-STF-package-entitlements | alias | SCR-FO-reservation-detail | package entitlements tab — used by: M54.F54.2.SF54.2.2 |
| SCR-STF-parking-manual | alias | SCR-PRK-manual-gate | same purpose — used by: M17.F17.3.SF17.3.4 |
| SCR-STF-parking-review | alias | SCR-PRK-lane-review | same purpose — used by: M17.F17.2.SF17.2.4 |
| SCR-STF-report-fault | alias | SCR-FO-hk-fault-report | same purpose — used by: M56.F56.1.SF56.1.4 |
| SCR-STF-report-found-item | alias | SCR-SAF-lost-intake | same purpose — used by: M56.F56.1.SF56.1.4 |
| SCR-STF-report-it-issue | alias | SCR-OPS-help-sop | report IT issue action — used by: M64.F64.2.SF64.2.6 |
| SCR-STF-requests | alias | SCR-FO-service-requests | same purpose — used by: M55.F55.1.SF55.1.3 |
| SCR-STF-return | alias | SCR-PRC-store-return | same purpose — used by: M14.F14.3.SF14.3.5 |
| SCR-STF-sanitation-check | alias | SCR-SAF-inspection-run | treatment-room sanitation checklist — used by: M58.F58.2.SF58.2.3 |
| SCR-STF-shift-handover | alias | SCR-OPS-shift-handover | same purpose — used by: M62.F62.2.SF62.2.1 |
| SCR-STF-sop-viewer | alias | SCR-OPS-help-sop | same purpose — used by: M62.F62.1.SF62.1.6 |
| SCR-STF-sync-status | alias | SCR-OPS-sync-queue | same purpose — used by: M56.F56.1.SF56.1.5,M64.F64.2.SF64.2.2 |
| SCR-STF-task-inbox | alias | SCR-OPS-unified-inbox | same purpose — used by: M18.F18.3.SF18.3.3 |
| SCR-STF-temperature-log | alias | SCR-FNB-haccp-checks | same purpose — used by: M57.F57.2.SF57.2.2 |
| SCR-STF-transfer-scan | alias | SCR-PRC-transfers | same purpose — used by: M14.F14.3.SF14.3.3 |
| SCR-STF-upgrade-at-desk | alias | SCR-FO-upsell-offers | same purpose — used by: M54.F54.1.SF54.1.1 |
| SCR-STF-voucher-redeem | alias | SCR-FO-payment-take | voucher tender — used by: M54.F54.2.SF54.2.1 |
| SCR-STF-waste | alias | SCR-FNB-waste-log | same purpose — used by: M14.F14.3.SF14.3.6 |
| SCR-STORE-cylinder-register | alias | SCR-ENG-cylinder-stock | storage limits and segregation — used by: M25.F25.1.SF25.1.1,M25.F25.2.SF25.2.2 |
| SCR-STORE-issue | alias | SCR-PRC-store-issue | spare parts to work order — used by: M26.F26.3.SF26.3.1 |
| SCR-STORE-receiving | alias | SCR-PRC-receiving | same purpose — used by: M21.F21.2.SF21.2.1,M21.F21.2.SF21.2.3,M25.F25.1.SF25.1.4 |
| SCR-STORE-reorder-suggestions | alias | SCR-PRC-replenishment | same purpose — used by: M21.F21.1.SF21.1.1,M21.F21.3.SF21.3.3,M25.F25.1.SF25.1.3 |
| SCR-STORE-stocktake | alias | SCR-PRC-blind-count | same purpose — used by: M25.F25.1.SF25.1.6 |
| SCR-STR-balances | alias | SCR-PRC-stock-balances | same purpose — used by: M50.F50.3.SF50.3.1,M50.F50.3.SF50.3.9 |
| SCR-STR-issue | alias | SCR-PRC-store-issue | same purpose — used by: M50.F50.3.SF50.3.2,M50.F50.3.SF50.3.8 |
| SCR-STR-recall | alias | SCR-PRC-recall-lookup | same purpose — used by: M50.F50.2.SF50.2.10 |
| SCR-STR-return | alias | SCR-PRC-store-return | same purpose — used by: M50.F50.3.SF50.3.4 |
| SCR-STR-stocktake | alias | SCR-PRC-blind-count | same purpose — used by: M50.F50.3.SF50.3.8 |
| SCR-STR-waste | alias | SCR-PRC-store-return | waste record — used by: M50.F50.3.SF50.3.5,M50.F50.3.SF50.3.6 |
| SCR-SUP-listings | alias | SCR-VEN-catalog | marketplace listings (M37, Later) — used by: M37.F37.1.SF37.1.1 |
| SCR-TV-guest-services | new | SCR-GST-iptv-home | defined in §2 — used by: M36.F36.2.SF36.2.1 |
| SCR-TV-purchase-confirm | new | SCR-GST-iptv-home | purchase confirmation panel — used by: M36.F36.2.SF36.2.2 |
| SCR-VEN-bank-details | alias | SCR-VEN-profile-locations | bank details (change via SCR-FIN-payee-bank-change) — used by: M20.F20.1.SF20.1.1 |
| SCR-VEN-credentials | alias | SCR-VEN-document-upload | same purpose — used by: M26.F26.2.SF26.2.1 |
| SCR-VEN-disputes | alias | SCR-VEN-messages-disputes | same purpose — used by: M21.F21.3.SF21.3.5,M26.F26.2.SF26.2.6 |
| SCR-VEN-invoice-status | alias | SCR-VEN-payment-status | same purpose — used by: M20.F20.1.SF20.1.5 |
| SCR-VEN-job-check-in | alias | SCR-VEN-job-evidence | on-site check-in — used by: M26.F26.2.SF26.2.3 |
| SCR-VEN-payments | alias | SCR-VEN-payment-status | same purpose — used by: M28.F28.2.SF28.2.5 |
| SCR-VEN-po-inbox | alias | SCR-VEN-po-list | same purpose — used by: M21.F21.1.SF21.1.5 |
| SCR-VEN-quotes | alias | SCR-VEN-bid-submit | service job quote — used by: M26.F26.2.SF26.2.5 |
| SCR-VND-bid-editor | alias | SCR-VEN-bid-submit | same purpose — used by: M46.F46.3.SF46.3.2,M48.F48.1.SF48.1.5,M49.F49.1.SF49.1.3 |
| SCR-VND-bulk-import | alias | SCR-VEN-bulk-import | same purpose — used by: M48.F48.3.SF48.3.5 |
| SCR-VND-catalog | alias | SCR-VEN-catalog | same purpose — used by: M48.F48.2.SF48.2.2,M48.F48.3.SF48.3.7 |
| SCR-VND-claim-workspace | alias | SCR-SAF-claim-file | external vendor/adjuster access scope — used by: M68.F68.1.SF68.1.4 |
| SCR-VND-corrective-action | alias | SCR-VEN-assigned-jobs | corrective action job — used by: M61.F61.2.SF61.2.3 |
| SCR-VND-dispatch-asn | alias | SCR-VEN-asn-dispatch | same purpose — used by: M46.F46.3.SF46.3.4,M50.F50.1.SF50.1.2,M50.F50.2.SF50.2.1 |
| SCR-VND-documents | alias | SCR-VEN-document-upload | same purpose — used by: M46.F46.1.SF46.1.3,M46.F46.1.SF46.1.6,M46.F46.3.SF46.3.1,M48.F48.1.SF48.1.4 |
| SCR-VND-home | alias | SCR-VEN-home | same purpose — used by: M48.F48.1.SF48.1.1 |
| SCR-VND-invoices | alias | SCR-VEN-invoice-submit | same purpose — used by: M46.F46.3.SF46.3.4 |
| SCR-VND-item-editor | alias | SCR-VEN-catalog-item-editor | same purpose — used by: M48.F48.2.SF48.2.1,M48.F48.2.SF48.2.2,M48.F48.2.SF48.2.3,M48.F48.2.SF48.2.4,M48.F48.2.SF48.2.5,M48.F48.2.SF48.2.6 |
| SCR-VND-jobs | alias | SCR-VEN-assigned-jobs | same purpose — used by: M46.F46.3.SF46.3.3,M46.F46.3.SF46.3.6 |
| SCR-VND-laundry-batch | alias | SCR-VEN-assigned-jobs | laundry batch job (counts/weights) — used by: M56.F56.2.SF56.2.2 |
| SCR-VND-messages | alias | SCR-VEN-messages-disputes | same purpose — used by: M46.F46.3.SF46.3.5,M49.F49.2.SF49.2.8,M50.F50.2.SF50.2.6,M50.F50.2.SF50.2.9 |
| SCR-VND-performance | alias | SCR-VEN-performance | same purpose — used by: M46.F46.2.SF46.2.6,M50.F50.4.SF50.4.1 |
| SCR-VND-po-inbox | alias | SCR-VEN-po-list | same purpose — used by: M46.F46.3.SF46.3.2,M48.F48.3.SF48.3.6,M49.F49.3.SF49.3.3,M49.F49.3.SF49.3.5,M50.F50.1.SF50.1.1 |
| SCR-VND-price-list | alias | SCR-VEN-price-list | same purpose — used by: M46.F46.2.SF46.2.3,M48.F48.3.SF48.3.2,M48.F48.3.SF48.3.3 |
| SCR-VND-profile | alias | SCR-VEN-profile-locations | same purpose — used by: M46.F46.3.SF46.3.1,M48.F48.1.SF48.1.3 |
| SCR-VND-register | alias | SCR-VEN-registration-wizard | same purpose — used by: M46.F46.1.SF46.1.1,M46.F46.1.SF46.1.2,M46.F46.1.SF46.1.5,M48.F48.1.SF48.1.2 |
| SCR-VND-rfq-inbox | alias | SCR-VEN-travel-order-queue | same purpose — used by: M45.F45.2.SF45.2.3,M46.F46.3.SF46.3.2,M48.F48.3.SF48.3.7,M49.F49.1.SF49.1.5,M49.F49.1.SF49.1.7,M49.F49.2.SF49.2.3 |
| SCR-VND-schedule | alias | SCR-VEN-emergency-chef-availability | availability and capacity schedule — used by: M46.F46.3.SF46.3.3,M47.F47.1.SF47.1.5,M48.F48.1.SF48.1.4,M48.F48.3.SF48.3.4 |
| SCR-VND-settings | alias | SCR-VAPP-notifications | preferences and offline drafts — used by: M48.F48.1.SF48.1.5 |
| SCR-VND-site-briefing | alias | SCR-VEN-assigned-jobs | site briefing acknowledgement — used by: M62.F62.1.SF62.1.4 |
| SCR-VND-stock-today | alias | SCR-VEN-daily-stock | same purpose — used by: M48.F48.3.SF48.3.1 |
| SCR-VND-sustainability-documents | alias | SCR-VEN-document-upload | sustainability evidence — used by: M67.F67.2.SF67.2.5 |
| SCR-VND-team | alias | SCR-VEN-team-users | same purpose — used by: M02.F02.1.SF02.1.4,M46.F46.1.SF46.1.5,M48.F48.1.SF48.1.3 |
| SCR-VND-trip-offer | alias | SCR-VEN-travel-order-queue | transport trip offer — used by: M59.F59.1.SF59.1.1 |
| SCR-WEB-accessibility | alias | SCR-GST-content-pages | accessibility statement — used by: M51.F51.1.SF51.1.3 |
| SCR-WEB-amenities | alias | SCR-GST-home | facilities section — used by: M51.F51.1.SF51.1.6,M58.F58.2.SF58.2.6 |
| SCR-WEB-checkout | alias | SCR-GST-payment | same purpose — used by: M51.F51.2.SF51.2.2 |
| SCR-WEB-confirmation | alias | SCR-GST-confirmation | same purpose — used by: M51.F51.2.SF51.2.7 |
| SCR-WEB-consent-banner | alias | SCR-GST-cookie-consent | same purpose — used by: M51.F51.2.SF51.2.4 |
| SCR-WEB-extras | alias | SCR-GST-extras | same purpose — used by: M51.F51.2.SF51.2.7,M54.F54.1.SF54.1.1,M54.F54.1.SF54.1.3 |
| SCR-WEB-gift-voucher | new | SCR-GST-gift-voucher | defined in §2 — used by: M54.F54.2.SF54.2.1 |
| SCR-WEB-guest-details | alias | SCR-GST-guest-details | same purpose — used by: M51.F51.2.SF51.2.7 |
| SCR-WEB-home | alias | SCR-GST-home | same purpose — used by: M51.F51.1.SF51.1.1 |
| SCR-WEB-location | alias | SCR-GST-content-pages | location and directions — used by: M51.F51.1.SF51.1.3 |
| SCR-WEB-payment | alias | SCR-GST-payment | same purpose — used by: M51.F51.2.SF51.2.7 |
| SCR-WEB-policies | alias | SCR-GST-content-pages | policies — used by: M51.F51.1.SF51.1.3 |
| SCR-WEB-privacy-preferences | alias | SCR-GST-cookie-consent | same purpose — used by: M51.F51.2.SF51.2.4 |
| SCR-WEB-quote-summary | alias | SCR-GST-payment | price review summary — used by: M51.F51.2.SF51.2.2 |
| SCR-WEB-results | alias | SCR-GST-room-results | same purpose — used by: M51.F51.2.SF51.2.1 |
| SCR-WEB-room-type | alias | SCR-GST-room-detail | same purpose — used by: M51.F51.1.SF51.1.2 |
| SCR-WEB-search | alias | SCR-GST-room-search | same purpose — used by: M51.F51.2.SF51.2.1 |
| SCR-WEB-unsubscribe | alias | SCR-GST-privacy-consent | one-click unsubscribe landing — used by: M52.F52.1.SF52.1.3 |
| SCR-WEB-venue | alias | SCR-GST-facility-search | venue detail — used by: M51.F51.1.SF51.1.2 |
