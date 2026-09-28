# 11 — Guest acquisition, booking, stay and service recovery

**Pack:** MetriStay Hospitality Suite — Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Governing source:** master prompt v3.0 Section P.2 (canonical commercial journey), Sections C and Q (M39, M40, M41, M51–M55, M18), K (M28, M30, M31, M39–M41), G19 (integrated scenario) and O (guest, marketing, revenue, front-desk questions). Conventions: `docs/README.md` §3.
**Status:** specification only. No website, channel connection, PSP, messaging provider, review platform or AI model is claimed to exist or be certified. SLA and budget numbers are **configurable defaults labelled `assumption`**, to be agreed with the pilot hotel (Section P.6); they are not industry benchmarks.

**Modules covered:** M39 media, M51 website/acquisition, M52 CRM/reputation, M53 revenue, M54 offers/vouchers, M55 guest journey/recovery, M18 guest engagement, M40 guest AI, M41 identity/signature/OTP, M07 distribution, M04 rates/quotes, M05 reservations/stays, M28 payments, M30 loyalty points, M31 referrals. Supporting: M02 consent, M03 inventory, M08 folio, M32/M65 reporting, M44 jurisdiction rule packs, M56 housekeeping, M57 room service, M63 workflow.

---

## 1. The canonical journey at a glance

```
S01 Publish ──► S02 Attribute ──► S03 Search/quote ──► S04 Total price & policy ──► S05 Hold
 (M39,M51,M07)   (M51,M02,M31)     (M51,M03,M04,M40)    (M04,M44,M38)                 (M03,M09)
                                                                                         │
S12 Consented return ◄── S11 Survey/recovery ◄── S10 Checkout ◄── S09 Stay/ancillaries ◄── S08 Pre-arrival ◄── S07 Booking ◄── S06 Payment
 (M52,M30,M31,M51)       (M52,M55)               (M05,M08,M28,M30) (M55,M54,M56,M57,M18,M40) (M55,M41,M54,M18) (M05,M07,M41) (M28)
```

Channel (OTA/GDS) bookings enter at **S07** through M07 with their channel attribution already set; they skip S03–S06 on MetriStay but share S07–S12. Assisted bookings (phone, desk, email) run S03–S07 through staff screens with `booking_channel=assisted` and the same quote/hold/payment rules.

**Journey invariants** (from Section P.3, applied here):
- **J-INV-1** No ghost sale: S07 cannot commit without an active S05 hold (or channel-accepted inventory decrement) and an S06 payment result or an approved pay-later policy.
- **J-INV-2** Price honesty: the total shown at S04 equals the total confirmed at S07 unless the guest explicitly accepts a new quote; policy snapshot is frozen at S04.
- **J-INV-3** Once-only: a guest task (pre-arrival form, request, ancillary purchase, survey) completed on any channel is complete on all channels; each folio posting carries a unique source key (INV-FOL-1).
- **J-INV-4** Consent first: no marketing message, abandoned-quote recovery or review solicitation without a matching `consent_record` for purpose + channel + jurisdiction (M02, SF52.1.3).
- **J-INV-5** Attribution is single-owner: at most one marketing source and at most one direct referrer per booking (SF31.1.2); disputes go to a queue, never double credit.
- **J-INV-6** AI never commits: the assistant may quote and draft; confirmation, payment and compensation are human- or guest-confirmed actions (SF40.2.2).

---

## 2. Stage specifications

Each stage lists owner, the authoritative timestamp (event + field), failure recovery, KPI, consent rule and data sources. Events follow README §3.5 envelope; `occurred_at` is UTC, `business_date` from M01.

### S01 — Property media, SEO and distribution publish

| Aspect | Specification |
|---|---|
| Purpose | Make accurate, rights-cleared room/venue/amenity content discoverable on the hotel website, search/maps and approved channels. |
| Owner | `content_approver` (media and copy); `marketing_manager` (SEO/listings); `revenue_manager` (channel room/rate mapping) |
| Subfeatures | SF39.1.1–SF39.1.7, SF39.2.3–SF39.2.4 (enhanced copy never replaces original; no fabricated facilities/views); SF51.1.1–SF51.1.5; M07 mapping/ARI (F-id per docs/01) |
| Timestamps | `MediaAssetApproved.approved_at`, `WebContentPublished.published_at`, `ChannelMappingActivated.activated_at`, `AriPushAcknowledged.ack_at` |
| Failure recovery | Rights expired → `MediaAssetUnpublished` + syndication takedown (SF39.1.7). CDN failure → serve last good version, alert `it_admin`. Structured-data validation error → page published without the invalid block, task to `content_editor`. ARI not acknowledged within timeout (default 5 min, `assumption`) → retry with backoff, then dead letter + `revenue_manager` task; affected dates flagged "channel state unknown" and stop-sell considered (M07). |
| KPI | Listing accuracy defects; assets with expiring rights; page performance budget pass rate (§7); ARI ack latency; channel mapping errors |
| Consent rule | Image rights/model releases (SF39.1.2). No guest images without documented consent. |
| Data sources | M39 `media_asset`, `media_rendition`, rights record; M51 page/content versions; M03 room attributes (accessibility features must match physical room data); M07 mappings |

### S02 — Consented traffic attribution

| Aspect | Specification |
|---|---|
| Purpose | Know where a visitor came from so booking cost and contribution can be measured, without over-collecting. |
| Owner | `marketing_manager` (campaign tags, analytics choices); `dpo` (cookie/consent configuration) |
| Subfeatures | SF51.2.3, SF51.2.4, SF51.2.6; SF31.1.1–SF31.1.2 (referral code/link); M02 consent |
| Timestamps | `VisitStarted.occurred_at` (server-side, minimal); `AttributionTouchRecorded.recorded_at`; `ConsentChoiceRecorded.recorded_at` |
| Model | Server-side first-party session with **strictly necessary** data only until consent. Touch types: `referral_code` > `paid_campaign` > `metasearch` > `organic_search` > `direct` (deterministic precedence, configurable per property; last non-direct touch within window, default 30 days `assumption`). Referral precedence follows Section E: code/last-click/manual with dispute queue. |
| Failure recovery | Consent banner fails to load → treat as "no analytics consent" (fail closed). Bot/duplicate traffic → filtered and reported separately (SF51.2.6). Conflicting referral claims → `ReferralAttributionDisputed` to `referral_program_admin`. |
| KPI | Consent rate by choice; share of bookings with known source; bot-filtered share |
| Consent rule | Analytics/advertising identifiers only with the choice required by the guest's jurisdiction rule pack (M44 privacy category). Refusal must not degrade booking function. Referrer never sees guest PII (Section E step 2). |
| Data sources | M51 session/touch log (pseudonymous); M02 `consent_record`; M31 referral link registry |

### S03 — Live search and quote

| Aspect | Specification |
|---|---|
| Purpose | Show only truly sellable rooms/facilities for the requested dates, party and accessibility needs. |
| Owner | System (`quote_service`); business owner `revenue_manager`; assisted path `front_desk_agent` |
| Subfeatures | SF51.2.1; M03 room-type night stock; M04 rate plans/restrictions/quote; M09 timed resources; SF40.2.1 (assistant uses the same quote tool) |
| Timestamps | `QuoteRequested.occurred_at`; `QuoteIssued.issued_at` + `expires_at` (default 20 min `assumption`) |
| Failure recovery | Inventory service slow → return "availability temporarily unavailable, call/email us" with assisted contact (never cached availability shown as live). No results → alternative dates, nearby room types, waitlist (M05). Accessibility need unavailable → show clearly and offer assisted contact, never a non-accessible substitute silently. |
| KPI | Search-to-quote rate; zero-result rate; p95 quote latency; accessibility-filter zero-result rate |
| Consent rule | Search parameters stored pseudonymously; if guest later books or consents, linked to profile. Abandoned-quote recovery only with prior consent (S12). |
| Data sources | M03 `room_type_night_inventory`, `inventory_hold`; M04 `rate_plan`, `quote`; M09 `resource_allocation`; M51 search log |

### S04 — Policy and total cost

| Aspect | Specification |
|---|---|
| Purpose | Present the honest total: room, taxes, levies, mandatory fees, deposits, cancellation and no-show terms before any payment. |
| Owner | `revenue_manager` (fees/policies); `compliance_officer` (tax/levy rule packs) |
| Subfeatures | SF51.2.2; M04 taxes/fees/rounding/currency, policy snapshot; M44.F44.1.SF44.1.2–SF44.1.3; M38.F38.1.SF38.1.3 |
| Timestamps | `PolicySnapshotCreated.created_at` (frozen, hashed) |
| Failure recovery | Rule pack for the property is `draft`/`expired` → quote shows "tax estimate — to be confirmed" label and routes booking to assisted confirmation; automated tax invoice blocked (Section A). Currency conversion display only; charge currency stated explicitly. |
| KPI | Total-price complaints (target 0); share of quotes with verified rule pack |
| Consent rule | Terms acceptance recorded (not marketing consent). |
| Data sources | M04 `policy_snapshot`; M44 `rule_pack` version; M38 tax lines |

### S05 — Inventory hold

| Aspect | Specification |
|---|---|
| Purpose | Reserve the exact inventory while the guest pays, without letting holds leak into oversell or permanent blockage. |
| Owner | System (`inventory_service`); monitor `front_office_manager` |
| Subfeatures | M03 holds/expiry; M09 composite hold (INV-CAP-1); SF41.2.1 |
| Timestamps | `InventoryHeld.held_at`, `expires_at` (default 15 min `assumption`); `InventoryHoldReleased.released_at` |
| Failure recovery | Hold expires during payment → payment intent is **not** captured; guest offered re-quote (price may change, J-INV-2). Concurrent last-room race → one hold succeeds (serialized lock), other gets alternatives. |
| KPI | Hold-to-booking conversion; hold-expiry rate; oversell incidents (0) |
| Consent rule | n/a |
| Data sources | M03 `inventory_hold`; M09 `composite_hold` |

### S06 — Approved payment

| Aspect | Specification |
|---|---|
| Purpose | Collect deposit/prepayment/guarantee per policy using a certified PSP; never store raw PAN/CVV. |
| Owner | System (`payment_service`); `cashier`/`finance_approver` for exceptions |
| Subfeatures | SF28.1.1, SF28.1.2, SF28.1.4, SF28.1.5; SF30.1.6 (partial points tender) ; SF54.2.1 (voucher redemption) |
| Timestamps | `PaymentIntentCreated.created_at`; `PaymentAuthorized.authorized_at`; `PaymentCaptured.captured_at` (PSP-confirmed) |
| Failure recovery | Decline → retry with another method; soft decline/3-DS challenge → guest-completed challenge. PSP timeout → status inquiry before any retry (idempotency key), booking stays `payment_pending` with hold extension once (default +10 min `assumption`). Duplicate webhook → deduped by `event_id`/PSP id (inbox). PSP outage → "pay at hotel" only if policy allows, else assisted booking with pay-by-link later (SF28.1.4). |
| KPI | Payment success rate; payment-pending duration; duplicate capture (0) |
| Consent rule | Card-on-file tokenization only with explicit consent (Section E, SF30.2.5 pattern). |
| Data sources | M28 `payment_intent`, PSP status log; M08 folio deposit |

### S07 — Booking (direct, assisted or channel)

| Aspect | Specification |
|---|---|
| Purpose | Convert hold + payment into a confirmed reservation with booker/occupant/payer separated. |
| Owner | System; `front_desk_agent` for assisted; `revenue_manager` for channel exceptions |
| Subfeatures | M05 reservation/booker-occupant-payer (F-id per docs/01); SF41.2.4–SF41.2.6 (OTP/QR where configured); M07 channel booking ingest; SF31.1.2 referral attribution; SF52.1.1 profile match |
| Timestamps | `ReservationConfirmed.confirmed_at`; channel: `ChannelBookingReceived.received_at` + `ChannelBookingAcknowledged.ack_at` |
| Failure recovery | Payment captured but reservation commit fails → automatic void/refund saga (M28) and incident task; never money without booking. Channel booking for sold-out inventory (oversell) → `OversellDetected` to `front_office_manager` with walk/relocation workflow. Confirmation message fails → retry, then alternative channel; guest can always retrieve via manage-booking link. |
| KPI | Quote-to-book conversion; channel booking ack latency; oversell count; confirmation delivery rate |
| Consent rule | Transactional messages under contract/legitimate basis per rule pack; marketing opt-in captured separately and unticked by default. |
| Data sources | M05 `reservation`, `stay`; M07 channel message log; M52 `guest_contact`; M31 attribution |

### S08 — Pre-arrival

| Aspect | Specification |
|---|---|
| Purpose | Complete registration data, ID (where required), preferences, accessibility needs, arrival time and optional upsells before arrival — once. |
| Owner | `guest_relations` (tasks); `front_office_manager` (readiness) |
| Subfeatures | SF55.1.2, SF55.1.3; SF41.1.1–SF41.1.7, SF41.2.2–SF41.2.3; SF54.1.1–SF54.1.4; M18 guest app |
| Timestamps | `PreArrivalInvited.sent_at` (default T-72h `assumption`); `PreArrivalCompleted.completed_at`; `UpsellOffered/UpsellAccepted.accepted_at` |
| Failure recovery | Guest does not complete → desk completes at arrival (assisted path). OCR error → field-by-field correction (SF41.1.5). Non-biometric path always available (SF41.1.6). Upgrade no longer available at arrival → auto-void of unpaid upgrade and refund of paid one (SF54.1.5). |
| KPI | Pre-arrival completion rate; OCR correction rate; upsell take rate and net revenue |
| Consent rule | ID images: purpose-limited, short-lived, excluded from analytics/AI training, timed deletion (SF41.1.7). Upsell messages are service messages only when tied to the booking and permitted by rule pack; otherwise require marketing consent. |
| Data sources | M41 ID extraction (transient); M55 tasks; M54 offers; M05 stay |

### S09 — Stay and ancillaries

| Aspect | Specification |
|---|---|
| Purpose | Deliver requests, room readiness, dining, parking and experiences with visible ownership and SLA. |
| Owner | Owning department per request type (`housekeeping_supervisor`, `fnb_manager`, `chief_engineer`, `concierge`), overseen by `duty_manager` |
| Subfeatures | SF55.1.4; SF56.1.1–SF56.1.5, SF56.2.4; SF57.1.2–SF57.1.4; SF54.1.3; SF40.2.3, SF40.2.6; M18 requests/messaging |
| Timestamps | `ServiceRequestCreated.created_at`; `ServiceRequestAcknowledged.ack_at`; `ServiceRequestCompleted.completed_at`; `GuestConfirmedResolution.confirmed_at` |
| Failure recovery | SLA breach → M63 escalation; request turns into complaint case (S11 recovery) if guest dissatisfied. Device outage → request by phone/desk into same record later (SF55.1.6). |
| KPI | Request acknowledgment/resolution time; room-ready-by-arrival rate; in-room dining promise kept |
| Consent rule | Preferences stored with purpose boundary (SF52.1.2). |
| Data sources | M18 `service_request`, `message_thread`; M56 tasks; M57 orders; M08 postings |

### S10 — Checkout

| Aspect | Specification |
|---|---|
| Purpose | Final folio, settlement, receipt/tax invoice, points earn, deposit release. |
| Owner | `front_desk_agent`/`cashier`; express checkout by guest |
| Subfeatures | SF55.1.5; M08 invoice/receipt; SF28.1.2, SF28.1.6; SF30.1.3; M38.F38.1.SF38.1.4 |
| Timestamps | `FolioSettled.settled_at`; `StayCheckedOut.checked_out_at`; `PointsEarnedPending.created_at` |
| Failure recovery | Disputed charge → hold that line, settle the rest, case to `guest_relations` (SF56.2.6 for minibar). Card fails → pay-by-link with AR follow-up. Invoice format unverified for jurisdiction → provisional receipt, formal invoice later. |
| KPI | Checkout duration; post-checkout adjustments; folio disputes |
| Consent rule | Receipt delivery channel as chosen; points require loyalty enrolment consent. |
| Data sources | M08 `folio`, `invoice`; M28; M30 `points_ledger_entry` |

### S11 — Survey and recovery

| Aspect | Specification |
|---|---|
| Purpose | Learn, close open issues and repair relationships; respond to public reviews. |
| Owner | `guest_relations`; approvals per §9 |
| Subfeatures | SF52.2.1–SF52.2.6; SF55.2.1–SF55.2.6 |
| Timestamps | `SurveySent.sent_at` (default checkout+24h `assumption`); `SurveyAnswered.answered_at`; `GuestCaseOpened/Closed`; `ReviewIngested.ingested_at`; `ReviewResponsePublished.published_at` |
| Failure recovery | Review platform API unavailable → manual copy of public review into case with URL. Guest unreachable → case closes "no response" with reason after timer. |
| KPI | Survey response rate; complaint-to-closure time; recovery closure rate; recovery cost; review response time |
| Consent rule | Survey = service message where permitted, else consent. **No incentives for positive reviews; no suppression of negative ones** (SF52.2.6). |
| Data sources | M52 survey/review; M55 `guest_case`; M54 voucher ledger; M08 reversals |

### S12 — Consented return visit

| Aspect | Specification |
|---|---|
| Purpose | Invite return business only from guests who agreed, and measure it. |
| Owner | `marketing_manager`; `referral_program_admin` for referral |
| Subfeatures | SF52.1.3–SF52.1.6; SF30.1.1–SF30.1.4; SF31.1.1–SF31.1.7 (market-gated); SF51.2.2 abandoned-quote recovery where consented |
| Timestamps | `CampaignMessageSent.sent_at`; `ConsentWithdrawn.withdrawn_at` (suppression effective immediately); `RepeatBookingConfirmed` |
| Failure recovery | Consent withdrawn mid-campaign → queued messages cancelled. Deletion request → profile erased except legally retained accounting/safety records (SF65.2.6). |
| KPI | Repeat rate; campaign cost per completed stay; opt-out rate; referral commissions reversed |
| Consent rule | Purpose + channel + jurisdiction consent; frequency caps (SF52.1.4); Oman referral payout gate (SF31.1.7). |
| Data sources | M52 segments; M30; M31; M51 attribution |

---

## 3. State machine summary (booking-to-return)

`docs/02` owns the full state machines; this is the journey-level view that the stages above must honour.

| State | Entered by | Exit to | Timer / owner |
|---|---|---|---|
| `quoted` | S03 | `held`, `expired` | quote expiry / system |
| `held` | S05 | `payment_pending`, `released` | hold expiry / system |
| `payment_pending` | S06 | `confirmed`, `payment_failed`, `released` | one extension / system → cashier |
| `confirmed` | S07 | `pre_arrival_complete`, `amended`, `cancelled`, `no_show` | T-72h invite / guest_relations |
| `pre_arrival_complete` | S08 | `in_house` | arrival day / front desk |
| `in_house` | check-in | `checked_out` | — |
| `checked_out` | S10 | `survey_sent`, `case_open` | checkout+24h / guest_relations |
| `case_open` | complaint | `case_resolved` → `case_closed`, `case_reopened` | severity SLA / case owner |
| `returnable` | consent present | `repeat_booked`, `suppressed` | frequency cap / marketing |

---

## 4. Direct versus channel net contribution

### 4.1 Definition (per booking, realized)

`net_contribution = net_room_revenue + net_ancillary_margin − channel_commission − PSP_fees − marketing_cost_allocated − referral_commission − loyalty_cost − cost_to_serve_variable`

| Component | Source (system of record) | Estimate → reconciled |
|---|---|---|
| Net room revenue (excl. taxes/levies collected for authorities) | M08 folio, M38 tax lines | at checkout → after refund window |
| Net ancillary margin (upgrade, parking, F&B, spa) | M54 allocations, M13/M14 COGS | at posting → month-end COGS |
| Channel commission | M07 booking commission rate → OTA invoice/statement (M20) | booking time estimate → invoice matched (SF60.2.3) |
| PSP fees | M28 settlement file | capture time estimate → settlement reconciled (SF28.1.7) |
| Marketing cost allocated | M52 campaign spend / M51 metasearch cost, allocated by attributed completed stays | spend accrual → invoice |
| Referral commission | M31 ledger (Section E formula) | pending → approved/paid/reversed |
| Loyalty cost | M30 earn liability and redemption cost | earn → breakage/redemption per policy |
| Variable cost-to-serve | M56 room turnover labor/linen, amenities, laundry (allocation version M32) | standard cost → actual allocation |

Reporting rules:
- Each figure shows `estimate` or `reconciled` (README §3.6). A booking is "reconciled" only when commission, PSP fee and refunds are settled.
- Compare by **source** (direct organic, direct paid, metasearch, each OTA, GDS, corporate, referral, assisted) per room night and per booking, with length of stay and cancellation rate alongside (a cheaper channel with high cancellations is not automatically better).
- Illustrative arithmetic only (not a benchmark, OMR): direct paid booking — room net 100.000, PSP fee 2.000, marketing allocated 6.000, cost-to-serve 15.000 → 77.000; OTA booking — room net 100.000, commission 15.000, PSP 0 (hotel-collect varies), cost-to-serve 15.000 → 70.000. Real inputs come from the pilot hotel's contracts.
- Channel commissions and marketing spend are reconciled monthly to realized contribution (SF51.2.5, SF53.1.3, SF32.1.3, SF60.2.3); unmatched commission invoices appear in the M60 case queue.

### 4.2 Decisions supported
Revenue manager: which channel to close on high-demand dates (SF53.2.1) — recommendation shows net contribution, not ADR alone. Marketing manager: campaign continuation by cost per completed stay (Q-MKT-4). GM/owner: channel mix trend in profit bridge (Q-OWN-2).

---

## 5. Attribution and referral boundaries

| Rule | Detail |
|---|---|
| One marketing source per booking | Deterministic precedence (S02); ties → most recent within window |
| One direct referrer per booking | SF31.1.2; code entered at checkout beats link cookie; manual assignment only by `referral_program_admin` with reason |
| No recursive graph | Data model has no referrer-of-referrer relation (Section E) |
| Self-referral and fraud | Same payer/identity/device heuristics → review queue, not auto-reject of the booking |
| Guest price unchanged | Referral never alters guest price unless a separately approved promotion applies |
| Market gate | Payout blocked where jurisdiction disabled (SF31.1.7); attribution may still be recorded for measurement if lawful |

---

## 6. Guest AI assistant boundaries (M40)

| May do | Must not do |
|---|---|
| Answer property-policy questions from the approved multilingual KB with cited source and "last reviewed" date (SF40.1.1–SF40.1.2) | Answer outside the KB as fact; invent facilities, prices, availability or policies |
| Disclose it is an AI assistant at start and on request (SF40.1.3) | Pretend to be a human or a named staff member |
| Call read-only availability/quote tool with live inventory (SF40.2.1) | Hold inventory beyond the standard hold, confirm, pay, cancel, refund, or grant compensation |
| Create a **draft** booking the guest must confirm and pay in the booking UI (SF40.2.2) | Collect card data in chat; take ID images in chat |
| Hand off to a human with case context; after hours give callback promise and time (SF40.2.3) | Keep a guest in a loop — a request for a human always triggers handoff |
| Escalate safety/emergency words to a human and show local emergency numbers (SF40.2.6) | Give medical, legal or emergency advice beyond the approved safety script |
| Mask PII in transcripts; follow transcript retention (SF40.2.4–SF40.2.5) | Use transcripts or ID data for model training without separate lawful basis |
| Degrade to "contact us" form + phone when provider is down or budget cap hit (SF40.2.7) | Fail silently |

Controls: tool permissions scoped to property and read-only by default; prompt-injection and hallucination regression suite (SF40.1.5) run before each KB or model change; per-conversation and monthly cost caps; containment is measured but **never optimized at the cost of blocked handoff** (KPI pair: containment rate and handoff wait time). Arabic and English at launch; additional languages require KB translation review by owners.

---

## 7. Accessibility and low-bandwidth budget

Target: WCAG 2.2 AA (Section P.1). The following budgets are **design targets labelled `assumption`**; they are verified in Phase 6 lab tests on a throttled profile and adjusted with the pilot hotel.

| Budget item | Target (assumption) | Test |
|---|---|---|
| Booking path pages usable without client JS for search → quote → guest details (progressive enhancement; SSR) | Yes | JS-disabled run of G19 booking |
| Critical-path transfer size per booking-step page (HTML+CSS+critical JS, compressed) | ≤ 200 KB; images lazy and responsive | Build-time budget check |
| Hero/gallery images | Responsive derivatives (SF39.1.4), ≤ 100 KB for first viewport image on mobile | Lighthouse/field sample |
| Time to usable search form on a slow 3G-class profile (~400 kbps, 400 ms RTT) | ≤ 5 s | Throttled lab test |
| Quote API p95 server latency | ≤ 1.5 s | Load test |
| Payment step | PSP-hosted fields/redirect; fallback pay-by-link if PSP script fails | PSP outage simulation |
| Total booking-flow steps from dates to payment | ≤ 5 screens | UX review |
| Video | Never autoplay with sound; captions/subtitles required (SF39.1.4) | Content check |

Accessibility requirements (non-exhaustive, all tested with keyboard and at least one screen reader in EN and AR/RTL):
- Keyboard operable date picker with text-entry alternative; visible focus (WCAG 2.4.11 focus not obscured); target size ≥ 24×24 CSS px (2.5.8).
- Errors identified in text with correction suggestions; no information lost on error (3.3.1, 3.3.3); redundant entry avoided — guest details reused (3.3.7).
- **Accessible authentication:** no cognitive-function test; no CAPTCHA. Bot defence uses rate limits, server heuristics and honeypots; OTP entry allows paste (3.3.8).
- Accessibility room features described factually from M03 attributes (e.g. roll-in shower, door width), searchable as filters; photos have alt text (SF39.1.3).
- Price and total announced as text, currency stated; no colour-only status.
- Session/hold timers announced with option to extend once (2.2.1).
- Assisted path always visible: phone, email and "request help booking" form creating an M55 inbox item with SLA.

---

## 8. Consent matrix

Consent requirements come from the M44 privacy rule pack for the guest's relevant jurisdiction; where the pack is unverified the stricter default (opt-in) applies.

| Purpose | Channel(s) | Default basis (subject to rule pack) | Where captured | Withdrawal effect |
|---|---|---|---|---|
| Booking confirmation, pre-arrival, in-stay service messages | email, SMS, approved WhatsApp template, app push | Contract/service | Booking | Cannot disable confirmation; can switch channel |
| Analytics cookies/identifiers | web | Consent (opt-in) | Banner | Stop setting; delete identifiers |
| Advertising/remarketing | web, ad platforms | Consent (opt-in) | Banner | Stop; audience sync removal |
| Marketing campaigns | email, SMS, WhatsApp | Consent (opt-in), per channel | Booking/profile | Immediate suppression |
| Abandoned-quote recovery | email | Consent obtained before abandonment | Guest-details step | Stop |
| Post-stay survey | email/app | Service where permitted; else consent | Booking | Stop |
| ID image processing | web/app/desk | Legal obligation or consent per market; non-biometric alternative | Pre-arrival/desk | Delete per SF41.1.7 |
| Biometric match | app/desk | Explicit consent only where lawful (SF41.1.6) | Pre-arrival | Stop; manual path |
| Loyalty membership | all | Contract (enrolment) | Enrolment | Close account; liability handling |
| Referral attribution | web | Transparent notice; referrer compensation disclosed | Link landing | Attribution kept for accounting only |
| AI assistant transcript | chat | Notice + retention policy; consent where required | Chat start | Transcript deleted per policy |

---

## 9. Service recovery and compensation approval matrix (M55, M54, M08, M30, M28)

### 9.1 Severity and SLA defaults (all `assumption`, configurable per property)

| Severity | Examples | Acknowledge | Resolve target | Case owner | Escalation |
|---|---|---|---|---|---|
| S1 Safety / vulnerable guest | Allergic reaction, fall, security threat, accessibility failure leaving guest unable to use room | Immediate (≤ 5 min) | Continuous until safe | `duty_manager` | GM + M42 incident; M61 link for food safety |
| S2 Major | Room unusable (no AC/water), wrong room type, overbooking walk, billing error > configured amount | ≤ 15 min | ≤ 2 h or relocation | `duty_manager` | GM |
| S3 Moderate | Slow room service, cleanliness defect, noisy room | ≤ 30 min | ≤ 4 h | Department head | Duty manager |
| S4 Minor / feedback | Amenity missing, suggestion | ≤ 4 h | ≤ 24 h | `guest_relations` | Department head |

Vulnerable-guest and accessibility cases are escalated by rule, never deprioritized by automated scoring; no automated decision uses protected characteristics (SF55.2.6).

### 9.2 Compensation approval matrix

Caps are **parameters** configured per property (values below are placeholders expressed relative to the guest's own charges, `assumption`, not recommendations). Requester ≠ approver above Level 1. Every compensation is a ledger entry: folio reversal (M08, reversing entry), voucher issue (M54 liability), points (M30 ledger), or refund (M28 to original tender).

| Compensation type | Level 1 (front-line: `front_desk_agent`, `guest_relations`, department supervisor) | Level 2 (`duty_manager`, `front_office_manager`, department head) | Level 3 (`gm`) | Level 4 (`gm` + `financial_controller`) | Accounting treatment |
|---|---|---|---|---|---|
| Apology amenity (drink, dessert, fruit) | ≤ configured amenity list | — | — | — | Outlet comp → M13 comp with reason; cost to recovery cost center |
| Waive specific ancillary charge (minibar item, room service delivery fee, parking day) | ≤ 100% of the affected charge up to `L1_cap` | ≤ `L2_cap` | above | — | M08 reversing posting linked to case |
| Room-night discount | — | ≤ 50% of one affected night | ≤ 100% of affected nights | > affected nights | M08 allowance posting (revenue reduction), case link |
| Loyalty points goodwill | ≤ `pts_L1` | ≤ `pts_L2` | ≤ `pts_L3` | above | M30 `goodwill` entry; liability per SF30.1.7 |
| Future-stay voucher | — | ≤ one night value | ≤ `voucher_L3` | above | M54 voucher liability with expiry (SF54.2.1) |
| Refund of paid amount | — | ≤ `refund_L2` | ≤ `refund_L3` | above; dual approval | M28 refund once to original tender; M08 reversal |
| Relocation (walk) at hotel cost | — | Approve per walk policy | Exceptions | — | M05 walk record; cost to rooms division |
| Cash payment | Not permitted | Not permitted | Only with `financial_controller` | Yes, documented | Discouraged; M60 cash control |

Controls: per-staff daily compensation limit and anomaly detection (M60 SF60.2.2 pattern); compensation cannot exceed guest's charges except relocation; recovery cost reported by cause and department (SF55.2.5); guest must confirm resolution or the case can be reopened (SF55.2.4).

### 9.3 Recovery workflow

`complaint received (any channel) → case (severity, owner, SLA) → link to work order / HK task / F&B order / incident → remedy → compensation (matrix) → guest confirmation → close → root cause tag → repeat-issue report`. Public review mentioning the stay is linked to the case; response approved by `guest_relations` lead with privacy check (no stay details disclosed publicly).

---

## 10. AT-G19 walkthrough (integrated scenario G19)

`docs/09` owns the executable test and numbering. This walkthrough fixes the business assertions this document requires. Fixture: pilot property with verified or clearly labelled rule pack, 2 room types (one with accessible attributes), upgrade offer, one OTA mapping via sandbox channel manager, sandbox PSP, website published.

| Step | Actor | Action | System evidence | Assertion |
|---|---|---|---|---|
| 1 | content_approver | Approve room photos (original + enhanced copy) and accessible-room description; publish website | `MediaAssetApproved`, `WebContentPublished` | Only approved renditions visible; alt text present; structured data valid |
| 2 | marketing_manager | Create campaign tag; configure metasearch/partner link (sandbox) | Campaign record | Tag resolvable in attribution |
| 3 | guest (three sessions) | Arrive via direct, search (campaign tag) and channel | `VisitStarted`, `AttributionTouchRecorded` | Each booking gets exactly one source; analytics identifiers only after consent |
| 4 | guest | On throttled mobile profile with screen reader, search dates, filter accessible room | Quote log | Budget §7 met; only sellable inventory; accessible room listed with features |
| 5 | guest | Add paid upgrade option; review total price | `PolicySnapshotCreated` | Total incl. taxes/fees equals final charge; upgrade priced; policy frozen |
| 6 | guest | Pay via sandbox PSP; simulate duplicate webhook | `PaymentCaptured` ×1 | One capture, one folio deposit, one reservation |
| 7 | channel (sandbox) | OTA booking arrives for other session | `ChannelBookingReceived/Acknowledged` | Inventory decremented once; commission recorded as estimate |
| 8 | guest | Pre-arrival: ID OCR with one deliberate error corrected; non-biometric path; sign; OTP | M41 events | Corrected field saved; ID image deleted per timer; signature evidence hashed |
| 9 | housekeeping_supervisor, laundry_attendant | Room prioritized for arrival; linen issued from par; inspection passed | `RoomInspected`, linen transfer | Room-ready before arrival ETA or ETA shown to desk |
| 10 | front_desk_agent | Check in; upgrade applied; posted once | Folio posting with source key | No double post; upgrade inventory decremented |
| 11 | guest | Order room service; delivery late and item wrong → complaint in app | `ServiceRequestCreated`, `GuestCaseOpened` (S3) | Case owner assigned; SLA timer started; F&B order linked |
| 12 | duty_manager | Remedy + compensation: waive room-service charge (L1) and offer points (L2 approval) | M08 reversal, M30 goodwill entry | Requester ≠ approver for L2; ledger entries linked to case |
| 13 | guest | Confirms resolution | `GuestConfirmedResolution` | Case closed; recovery time recorded |
| 14 | guest, cashier | Checkout; receipt/tax invoice; points pending | `FolioSettled`, `StayCheckedOut` | Folio zero balance; points pending until refund window |
| 15 | guest | Survey answered; consents to review request; review ingested (sandbox/manual) | `SurveyAnswered`, `ReviewIngested` | No incentive offered for review; response approved |
| 16 | marketing_manager | Report | Funnel + contribution report | Net acquisition cost per source, quote conversion and guest recovery time shown with estimate/reconciled labels |
| 17 | revenue_manager | Open 90-day forecast/pickup; approve guardrailed rate change for one date | `RateActionApproved`, `AriPushAcknowledged` | Change within guardrail; channel ack received; discrepancy queue empty |
| 18 | revenue_manager | Roll back the change | `RateActionRolledBack`, second ack | Previous rate restored on website and channel |
| 19 | gm | Drill from occupancy and profit to channel fee, labor, laundry, food waste and utilities | Profit bridge drill-through | Each figure drills to source events; missing source shows incomplete estimate |
| 20 | guest | Withdraws marketing consent | `ConsentWithdrawn` | Guest suppressed from pending campaign immediately |

Negative checks bundled with G19: hold expiry during payment (no capture); PSP timeout (inquiry before retry); assistant asked to "book and pay" (produces draft only); consent absent (no recovery email).

---

## 11. KPI set for the journey

Quote-to-book conversion; abandonment by step and reason; share of bookings with known source; net acquisition cost per completed stay by source; net contribution per room night by source; ARI ack latency; payment success; pre-arrival completion; room-ready by arrival; request resolution time; complaint-to-closure time; recovery closure rate and cost; survey response; review response time; repeat rate; opt-out rate; assistant containment and handoff wait; accessibility defect count. Definitions and denominators live in the `docs/06` KPI dictionary (SF65.1.2). Targets: agreed with pilot hotel (D-111).

## 12. Open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-111 | Pilot KPI baselines/targets | Pilot GM | Measure 30-day baseline |
| D-112 | Attribution window and precedence | Marketing Manager | 30 days, precedence in S02 |
| D-113 | Compensation cap values (`L1_cap` … `refund_L3`) | GM + Financial Controller | Placeholders in §9.2 |
| D-114 | Hold/quote timers | Revenue Manager | 20 min quote, 15 min hold, one 10 min extension |
| D-115 | Messaging providers and WhatsApp templates per market | Integration Admin | Email + SMS; WhatsApp gated |
| D-116 | Review sources with approved API access | Guest Relations | Manual ingestion |
| D-117 | AI provider/local model and cost caps | IT Admin + DPO | Pluggable port; caps set before pilot |

Decision ids D-111–D-119 are reserved for this file; `docs/13` is authoritative.
