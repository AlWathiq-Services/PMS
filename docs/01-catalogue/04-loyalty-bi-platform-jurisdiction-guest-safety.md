# Module Catalogue — Group 04: Loyalty, Referral, BI, Integration Platform, Connected Room (Later), Marketplace, Jurisdiction, Media, Guest AI, Identity, Safety

**Pack:** Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28 • **Parent index:** `docs/01-module-catalogue.md` • **Conventions:** `docs/README.md` §3 (identifiers, Section-L schema, API/event style, honesty labels, money/time).
**Governing source:** master prompt v3.0 Sections A, B, C (rows M30–M44), E (two balances; MetriStay Network), G (AT-G07, G08, G09, G12, G13, G14, G19, G20), K (fixed IDs for M30, M31, M32, M38–M44), L (schema), P (invariants).

> Everything in this file is a specification. It does not claim that code, a partner contract, a licensed wallet, a government API, a legal opinion or a certification exists. Rule packs, integrations and legal positions carry a status from README §3.6; where the status is below `counsel-reviewed` / `certified`, the dependent automated action is **blocked** and a manual path is named.

## 0. Scope, reading guide and counts

| Module | Name | First build phase | Release | Features | Subfeatures | Section K fixed? |
|---|---|---|---|---|---|---|
| M30 | Loyalty points wallet (MetriStay Rewards) | 5 | R1 (cash wallet: separately gated, not R1) | 2 | 13 | Yes |
| M31 | Referrals/growth (MetriStay Network, MetriStay Partner Hub) | 5 build/test; 6 activation gate; 8 partner expansion | R1 (build + gate); payout per market only after gate | 2 | 14 | Yes |
| M32 | Management BI | 2 foundation; 4–6 complete | R1 | 4 | 24 | Yes |
| M33 | Integration developer platform | 2 foundation; 5–7 mature | R1 (public self-service portal Later) | 3 | 13 | No (created here) |
| M34 | UC/wake-up | 7 | Later | 2 | 7 | No (created here) |
| M35 | HSIA | 7 | Later | 2 | 7 | No (created here) |
| M36 | IPTV | 7 | Later | 2 | 7 | No (created here) |
| M37 | Marketplace/AI | 7–8 | Later | 3 | 10 | No (created here) |
| M38 | Jurisdiction, tax and government exchange | 2 foundation; 4–6 certified workflows | R1 | 3 | 20 | Yes |
| M39 | Property media and AI enhancement | 2–3 | R1 | 2 | 13 | Yes |
| M40 | Guest AI assistant | 3–5 | R1 | 2 | 13 | Yes |
| M41 | Assisted identity, signature and verification | 2–3; 5 messaging adapters | R1 | 2 | 15 | Yes |
| M42 | Emergency and incident response | 3–4 | R1 | 2 | 12 | Yes |
| M43 | Lost and found | 2–3 | R1 | 1 | 8 | Yes |
| M44 | Five-market jurisdiction classifier | 2 foundation; 4–6 rules/verification | R1 | 2 | 14 | Yes |
| **Total** | | | | **34** | **190** | |

Each subfeature below is a Section-L block (README §3.4) in compact YAML flow style with all 18 fields. `acceptance` is the unit-level `AC-<SF id>`; module acceptance maps to integrated scenario tests `AT-Gnn.k` owned by `docs/09`. Open decisions for this group use **D-401..D-499**; they are summarised again in §16 for the decision log in `docs/13`.

Screen prefixes used here: `SCR-GUEST-*` (guest web/app), `SCR-HUB-*` (MetriStay Partner Hub), `SCR-FD-*` (front desk), `SCR-FIN-*` (finance), `SCR-MGMT-*` (owner/GM/BI), `SCR-HR-*`, `SCR-COMP-*` (compliance/jurisdiction), `SCR-ADMIN-*` (property/integration admin), `SCR-DEV-*` (developer platform), `SCR-CONTENT-*` (media), `SCR-SAFETY-*` (incident/lost-found), `SCR-STAFF-*` (staff mobile), `SCR-POS-*` (outlet POS), `SCR-MKT-*` (marketplace, Later), `SCR-SUP-*` (marketplace supplier, Later).

---

## 1. M30 — Loyalty points wallet (MetriStay Rewards)

| Header | Value |
|---|---|
| Purpose | Closed-loop, **non-cash** points for genuine own purchases at the hotel: earn, pending→available, redeem on eligible hotel purchases, expire, reverse; tier and balance; guest visibility; financial liability and breakage reporting; fraud caps and audit. Required in Release 1 (Section E balance #1). |
| Second balance | **Cash/stored-value wallet is a different product** (Section E balance #2): only via a licensed bank/PSP wallet with ledger mirroring, KYC/AML, safeguarding, top-up/refund/settlement handled by the partner; separately gated (SF30.2.6), **not** required for R1 points, and never implemented as a self-custodied database balance. |
| Phases / release | Phase 5 (Section B "non-cash rewards wallet"), R1. Profile linkage uses M18/M52 profile from Phase 3. |
| Bounded context | `loyalty` |
| System-of-record entities | `loyalty_program`, `loyalty_account`, `loyalty_tier`, `points_campaign_version` (earn/redeem/expiry rules, versioned), `points_ledger_entry` (append-only), `points_lot` (per-earn bucket for FIFO expiry), `points_redemption`, `points_hold`, `points_dispute`, `loyalty_fraud_review`, `loyalty_liability_snapshot`, `external_wallet_link` (gated, cash wallet mirror pointer only) |
| Referenced (not owned) | `guest_profile` (M18/M52), `reservation`/`stay` (M05), `folio_line` (M08), `pos_check` (M13), `catering_order` (M16), `club_membership` (M15), `payment` (M28), `consent_record` (M02), `gl_journal` (M19), `jurisdiction_profile`/`rule_pack` (M44) |
| Dependencies | M02 consent/identity, M05/M08 settlement facts, M13/M15/M16 charge facts, M19 GL posting, M28 refund/chargeback events, M44 rule pack for consumer-law terms (expiry notices, tax treatment), M63 notifications |
| Publishes | `PointsEarnedPending`, `PointsReleased`, `PointsRedeemed`, `PointsRedemptionReversed`, `PointsReversed`, `PointsExpired`, `PointsAdjusted`, `LoyaltyTierChanged`, `LoyaltyFraudFlagged`, `LoyaltyLiabilitySnapshotted` |

### F30.1 Points rules

```yaml
- id: M30.F30.1.SF30.1.1
  name: "eligibility by service/product/company"
  phase: 5
  release: R1
  actors: [marketing_manager, financial_controller, property_admin, guest, loyalty_worker]
  screens: [SCR-ADMIN-loyalty-program, SCR-ADMIN-loyalty-eligibility-matrix, SCR-GUEST-points-how-to-earn]
  inputs: [program_id, service_category, product_or_rate_plan_ids, outlet_ids, corporate_account_ids, channel_codes, guest_residency_rule, effective_from, effective_to]
  states: [draft, pending_approval, active, retired]
  api: "PUT /v1/properties/{pid}/loyalty/programs/{program_id}/eligibility (versioned; GET for guest-facing summary)"
  events: [LoyaltyEligibilityVersionActivated]
  data: [loyalty_program, points_campaign_version, eligibility_rule]
  rules: ["Only settled own-purchase revenue of the member (room, F&B, catering, club, parking, eligible ancillaries) can qualify; taxes, service charges, deposits, gift-voucher purchases, points-paid portions and third-party pass-through (e.g. M45 airline tickets) are excluded by default", "Corporate-billed stays earn only if the corporate agreement (M10) permits and names the traveller as earning member", "OTA/channel bookings earn only where channel contract allows (flag per channel)", "No earning for referring others; referral commission is M31 and never converts to points", "Rules are effective-dated versions; a transaction uses the version valid at its business date"]
  security: "maker-checker activation (marketing_manager proposes, financial_controller approves); property scope; change audit with before/after"
  failure_cases: [overlapping_active_versions, category_not_mapped_to_gl, corporate_contract_conflict, rule_pack_missing_for_market]
  finance_report_effect: "Determines which revenue lines create points liability; eligibility version is a dimension on liability and cost-of-redemption reports"
  i18n_a11y: "Earning rules rendered in English/Arabic RTL from the same version; plain-language summary, screen-reader tables, no colour-only eligibility cues"
  acceptance: "AC-SF30.1.1: With version v2 excluding parking, a folio with room OMR 50.000 + parking OMR 5.000 + VAT earns points only on OMR 50.000; a charge dated before v2 activation uses v1; activation without a second approver is rejected (403)"
  dependency: "M10 corporate agreement flags, M07 channel contract flags, D-402 earn policy, D-404 consumer-law terms per market"
- id: M30.F30.1.SF30.1.2
  name: "earn rate/tier/promotion"
  phase: 5
  release: R1
  actors: [marketing_manager, financial_controller, loyalty_worker]
  screens: [SCR-ADMIN-loyalty-earn-rates, SCR-ADMIN-loyalty-tiers, SCR-ADMIN-loyalty-campaigns, SCR-GUEST-points-tier-status]
  inputs: [base_rate_points_per_minor_unit, currency, tier_multipliers, tier_thresholds, qualification_period, promotion_id, promotion_multiplier_or_bonus, budget_cap_points, start_end_dates]
  states: [draft, pending_approval, scheduled, active, paused, ended]
  api: "POST /v1/properties/{pid}/loyalty/campaigns; POST /v1/properties/{pid}/loyalty/campaigns/{id}/approve (Idempotency-Key)"
  events: [PointsCampaignActivated, PointsCampaignBudgetExhausted, LoyaltyTierChanged]
  data: [points_campaign_version, loyalty_tier, tier_qualification_counter]
  rules: ["Points = floor(eligible_net_amount_minor x rate x tier_multiplier) + promotion bonus, computed once per settled source line and stored with the campaign version id", "Promotions have a points budget cap; when exhausted the promotion stops earning and the base rate continues", "Tier qualification counts only completed, non-reversed stays/spend; reversals re-evaluate tier with a grace rule (D-402)", "No earn for buying points, transferring points or referral activity"]
  security: "maker-checker; campaigns above a points-budget threshold need financial_controller; audit"
  failure_cases: [rate_misconfiguration_outlier, budget_cap_race, tier_recalc_after_reversal, currency_without_rate]
  finance_report_effect: "Estimated liability per campaign (points x cost-per-point assumption) shown before approval; campaign cost appears in marketing cost and loyalty liability reports"
  i18n_a11y: "Tier names localisable; numerals per locale with Western/Arabic-Indic option; tier progress bar has text equivalent"
  acceptance: "AC-SF30.1.2: Gold member (x1.5) with base 1 pt per OMR 1.000 settles OMR 120.000 eligible spend during a 500-pt bonus campaign -> 680 pts pending, one ledger entry per source line; a second campaign activation that would exceed budget cap stops at cap exactly"
  dependency: "D-401 cost-per-point/accounting policy; D-402 tier design"
- id: M30.F30.1.SF30.1.3
  name: "pending vs available"
  phase: 5
  release: R1
  actors: [loyalty_worker, guest, guest_relations]
  screens: [SCR-GUEST-points-wallet, SCR-FD-guest-loyalty-panel]
  inputs: [source_type, source_id, member_id, points, release_condition, release_after_business_date]
  states: [pending, available, held, reversed, expired, redeemed]
  api: "POST /v1/properties/{pid}/loyalty/earn (internal, event-driven, Idempotency-Key = source_type:source_id:line_id); GET /v1/loyalty/accounts/{account_id}/balances"
  events: [PointsEarnedPending, PointsReleased]
  data: [points_ledger_entry, points_lot, loyalty_account]
  rules: ["Earn is created as pending on checkout/settled charge and released to available only after payment is cleared and the refund/chargeback window configured per tender type has passed", "Balances are derived from the ledger: available, pending, held, expired, lifetime; never stored as a mutable single number without ledger backing", "Release job is idempotent and re-checks source state (not refunded, not disputed) before releasing"]
  security: "earn only via internal service identity from domain events; no staff UI to create earn without the adjustment flow (SF30.2.3)"
  failure_cases: [source_event_duplicate, source_refunded_before_release, payment_status_unknown, release_job_outage]
  finance_report_effect: "Pending points are disclosed separately from available in liability report; release changes liability classification per D-401"
  i18n_a11y: "Guest sees 'pending until <date>' with localized date and explanation; status not conveyed by colour only"
  acceptance: "AC-SF30.1.3: Checkout with card payment creates 300 pending; replaying the same event creates no new entry; after the configured window with no refund, 300 become available once; if refunded before release, pending is reversed and never becomes available"
  dependency: "M28 payment cleared/refund events, M08 settlement, D-402 pending window policy"
- id: M30.F30.1.SF30.1.4
  name: "expiry"
  phase: 5
  release: R1
  actors: [loyalty_worker, guest, marketing_manager]
  screens: [SCR-GUEST-points-wallet, SCR-GUEST-points-expiry-notice, SCR-ADMIN-loyalty-expiry-policy]
  inputs: [expiry_policy (fixed_months_from_earn | inactivity_months), notice_days, jurisdiction_profile_id]
  states: [available, expiring_notice_sent, expired]
  api: "Job: loyalty.expire-lots daily per property business date; GET /v1/loyalty/accounts/{id}/expiring"
  events: [PointsExpiryNoticeSent, PointsExpired]
  data: [points_lot, points_ledger_entry, notification_record]
  rules: ["Expiry consumes lots FIFO; an expiry is a negative ledger entry referencing the lot, never deletion", "Notice sent per consent and market rule pack before expiry; if the market rule pack is unverified, expiry is suspended (points stay available) and a compliance task opens", "Tier-status or activity extensions are recorded as policy decisions, not silent edits"]
  security: "policy change maker-checker; job runs under service identity; audit"
  failure_cases: [notice_delivery_failed, rule_pack_unverified_for_market, job_rerun_same_day, timezone_boundary]
  finance_report_effect: "Expired points recognised as breakage per accounting policy (D-401); breakage report by campaign/month"
  i18n_a11y: "Expiry notices localized EN/AR; accessible email/SMS templates; dates in property time zone with explicit label"
  acceptance: "AC-SF30.1.4: Lot of 200 earned 2025-01-10 with 18-month policy expires on business date 2026-07-10 with one PointsExpired entry; rerunning the job creates no second entry; with the Portugal rule pack in draft, the job skips expiry for Portugal-resident accounts and opens a compliance task"
  dependency: "M44 rule pack (consumer terms), M52 consent/channel rules, D-404"
- id: M30.F30.1.SF30.1.5
  name: "refund reversal"
  phase: 5
  release: R1
  actors: [loyalty_worker, cashier, finance_clerk]
  screens: [SCR-FD-folio-refund, SCR-FIN-loyalty-reversal-queue, SCR-GUEST-points-history]
  inputs: [refund_id, source_line_ids, refund_amount_minor, chargeback_id]
  states: [requested, reversed, negative_balance_recovery, written_off]
  api: "Consumer of RefundCompleted / ChargebackOpened / ReservationCancelled; POST /v1/properties/{pid}/loyalty/reversals (internal, Idempotency-Key = refund_id)"
  events: [PointsReversed, LoyaltyNegativeBalanceRaised]
  data: [points_ledger_entry, points_lot, reversal_link]
  rules: ["A refund reverses the points earned on the refunded portion exactly once (proportional for partial refund, rounding down in the member's disfavour is not allowed: round to nearest with ties toward member)", "If earned points were already redeemed, reversal may create a negative available balance that is netted against future earn; it is never collected as cash", "A refund of a points-paid portion re-credits the redeemed points (see SF30.1.6) as a separate linked entry", "Reversal links to the original earn entry id; a second event for the same refund is a no-op"]
  security: "service identity only; manual reversal requires SF30.2.3 adjustment approval"
  failure_cases: [duplicate_refund_event, partial_refund_rounding, earn_already_expired, chargeback_then_representment_win]
  finance_report_effect: "Liability reduced by reversed points; if the reversal lands after period close it posts in the current open period with link to source"
  i18n_a11y: "History line shows 'reversed – refund <receipt no.>' localized; accessible table"
  acceptance: "AC-SF30.1.5 (AT-G07): A OMR 100.000 stay earning 100 pts is 40% refunded -> exactly -40 pts once; duplicate refund webhook produces no second reversal; folio refund and points reversal share the refund_id in audit"
  dependency: "M28 refund/chargeback events, M08 folio refund lines"
- id: M30.F30.1.SF30.1.6
  name: "redemption limits, exclusion and partial tender"
  phase: 5
  release: R1
  actors: [guest, front_desk_agent, cashier, server, loyalty_worker]
  screens: [SCR-GUEST-checkout-pay-with-points, SCR-FD-folio-tender-points, SCR-POS-tender-points]
  inputs: [account_id, points_to_redeem, target (folio_window_id | pos_check_id | booking_quote_id), step_up_token, idempotency_key]
  states: [quoted, held, captured, released, reversed]
  api: "POST /v1/properties/{pid}/loyalty/redemptions (hold) -> POST .../redemptions/{id}/capture | /release (Idempotency-Key)"
  events: [PointsRedemptionHeld, PointsRedeemed, PointsRedemptionReleased, PointsRedemptionReversed]
  data: [points_redemption, points_hold, points_ledger_entry, folio_line]
  rules: ["Redeem only against eligible hotel purchases as a partial or full tender; never for cash, bank transfer, gift voucher purchase, peer transfer or third-party pass-through", "Min/max points per transaction and per day, blackout and excluded items per campaign version", "Hold then capture; hold expires (default 30 min) and auto-releases; capture posts a tender line on the folio/POS with points and monetary equivalent", "Remaining balance is paid by another tender via M28; points tender is not a payment method in PSP scope"]
  security: "guest step-up (OTP/app session) above threshold; staff-initiated redemption needs guest confirmation on device or signed consent; property scope"
  failure_cases: [insufficient_available, hold_expired_before_capture, concurrent_redemption_race, excluded_item_in_basket, pos_offline]
  finance_report_effect: "Redemption reduces liability and records redemption cost/revenue allocation per D-401 on the revenue department where consumed"
  i18n_a11y: "Points slider with numeric input alternative; live region announces remaining amount; RTL layout; offline POS shows 'points unavailable offline' rather than queuing a redemption"
  acceptance: "AC-SF30.1.6: Guest with 1,000 available redeems 600 against a OMR 30.000 dinner (rate 20 pts/OMR 1.000) + pays OMR 0.000 remainder; two concurrent redemptions of 600 result in one capture and one insufficient_available; excluded minibar item cannot be paid with points"
  dependency: "M08 folio tender types, M13 POS tender integration, M41 OTP for step-up"
- id: M30.F30.1.SF30.1.7
  name: "liability/breakage report"
  phase: 5
  release: R1
  actors: [financial_controller, finance_clerk, gm, owner, auditor]
  screens: [SCR-FIN-loyalty-liability, SCR-MGMT-loyalty-dashboard]
  inputs: [period, property_id, campaign_id, valuation_method_version, breakage_rate_assumption]
  states: [draft, reviewed, posted, restated]
  api: "GET /v1/properties/{pid}/reports/loyalty-liability?period=; POST /v1/properties/{pid}/loyalty/liability-snapshots/{id}/post (Idempotency-Key)"
  events: [LoyaltyLiabilitySnapshotted, LoyaltyLiabilityPosted]
  data: [loyalty_liability_snapshot, points_ledger_entry, gl_journal]
  rules: ["Snapshot reproduces from ledger at period end: opening + issued - redeemed - expired - reversed +/- adjustments = closing (must tie to zero difference)", "Valuation (deferred revenue / provision) and breakage method are policy versions approved by financial_controller (D-401); report states method and whether estimate or reconciled", "Management view shows issuance, breakage and redemption cost by campaign and outlet"]
  security: "finance roles; export watermarking; auditor read-only"
  failure_cases: [ledger_rollforward_mismatch, policy_version_missing, late_reversal_after_close]
  finance_report_effect: "Posts liability journal to M19; feeds M32 SF32.2.x department P&L (redemption cost) and GM flash"
  i18n_a11y: "Report in EN/AR with numeric alignment; CSV/XLSX export; accessible charts with data table"
  acceptance: "AC-SF30.1.7: Fixture month with 10,000 issued, 3,000 redeemed, 500 expired, 200 reversed produces closing 6,300 matching ledger sum; posting twice with the same key posts one journal"
  dependency: "M19 GL mapping, D-401"
```

### F30.2 Wallet security

```yaml
- id: M30.F30.2.SF30.2.1
  name: "append-only points ledger and idempotency"
  phase: 5
  release: R1
  actors: [loyalty_worker, auditor, it_admin]
  screens: [SCR-FIN-loyalty-ledger-explorer, SCR-ADMIN-loyalty-integrity]
  inputs: [entry_type, account_id, points, lot_id, source_type, source_id, campaign_version_id, idempotency_key, correlation_id]
  states: [posted]
  api: "Internal ledger port LoyaltyLedger.post(entry) — no UPDATE/DELETE; GET /v1/loyalty/accounts/{id}/ledger (paged)"
  events: [PointsLedgerEntryPosted]
  data: [points_ledger_entry, points_lot, ledger_idempotency_key]
  rules: ["Entries are immutable; corrections are new reversing/adjusting entries linked to the corrected entry", "Unique constraint on (tenant_id, idempotency_key); a replay returns the original entry", "Balance by type = SUM(points) over entries; nightly integrity job compares derived balances to cached projections and alerts on any difference", "DB role for the app has no UPDATE/DELETE grant on the ledger table"]
  security: "Postgres RLS by tenant/property; append-only enforced by grants and trigger; hash chain per account for tamper evidence"
  failure_cases: [duplicate_key_different_payload, projection_drift, clock_skew, partial_transaction]
  finance_report_effect: "Single source for liability, breakage and redemption cost; drill-through from M32 to entries"
  i18n_a11y: "Explorer is staff-only; accessible data grid with keyboard navigation"
  acceptance: "AC-SF30.2.1: Posting the same idempotency key with identical payload returns the first entry; with different payload returns 409; an attempted UPDATE by the app role fails; integrity job on fixture reports zero drift"
  dependency: "M01 platform ledger pattern (ADR on append-only ledgers)"
- id: M30.F30.2.SF30.2.2
  name: "account/device security"
  phase: 5
  release: R1
  actors: [guest, guest_relations, dpo, loyalty_worker]
  screens: [SCR-GUEST-account-security, SCR-GUEST-points-wallet, SCR-FD-guest-loyalty-panel]
  inputs: [guest_identity_id, email_or_phone, mfa_factor, device_id, session_id]
  states: [unverified, active, locked, merged, closed]
  api: "POST /v1/loyalty/accounts (enroll); POST /v1/loyalty/accounts/{id}/lock; POST /v1/loyalty/accounts/{id}/merge-requests"
  events: [LoyaltyAccountEnrolled, LoyaltyAccountLocked, LoyaltyAccountMerged]
  data: [loyalty_account, guest_identity (M02), device_binding]
  rules: ["One loyalty account per verified guest identity; enrollment requires verified email or phone (OTP)", "Contact-detail change and redemption above threshold require step-up and notify old channel", "Account merge is staff-reviewed and moves balances via paired ledger entries, never by editing", "Closing an account forfeits or pays out nothing in cash; remaining points handled per terms"]
  security: "guest auth via Keycloak; rate limiting on login/OTP; staff cannot view full contact data without role; no points visible to referrers"
  failure_cases: [account_takeover_attempt, duplicate_enrollment, sim_swap_suspected, merge_conflict]
  finance_report_effect: "Merges and closures appear as adjustments in liability roll-forward"
  i18n_a11y: "Accessible MFA (no CAPTCHA-only), RTL forms, error messages associated to fields"
  acceptance: "AC-SF30.2.2: Changing phone number without step-up is rejected; merge of two accounts with 300 and 200 creates -300/+300 paired entries and closing balance 500 on survivor"
  dependency: "M02 IAM/guest identity, M41 OTP adapter"
- id: M30.F30.2.SF30.2.3
  name: "anti-fraud velocity and review"
  phase: 5
  release: R1
  actors: [loyalty_worker, guest_relations, financial_controller, compliance_officer]
  screens: [SCR-FIN-loyalty-fraud-queue, SCR-ADMIN-loyalty-adjustment]
  inputs: [velocity_rules (earn/redeem per day, per device, per staff), manual_adjustment_reason, evidence_ref, approver_id]
  states: [flagged, under_review, cleared, confirmed_fraud, adjusted]
  api: "GET /v1/properties/{pid}/loyalty/fraud-reviews; POST /v1/properties/{pid}/loyalty/adjustments (maker-checker, Idempotency-Key)"
  events: [LoyaltyFraudFlagged, LoyaltyFraudReviewClosed, PointsAdjusted]
  data: [loyalty_fraud_review, points_adjustment_request, points_ledger_entry]
  rules: ["Velocity breaches hold further redemption (not earn) pending review", "Manual adjustments require reason code + evidence + approver different from requester; staff cannot adjust own/related accounts", "Staff-posted earn on folio lines they themselves created is flagged", "Confirmed fraud reverses affected entries with links; account may be locked"]
  security: "segregation of duties; step-up for approver; full audit; conflict-of-interest check against staff relationships"
  failure_cases: [false_positive_customer_block, approver_equals_requester, review_sla_breach]
  finance_report_effect: "Adjustments reported separately in liability roll-forward; fraud losses reported to M60 revenue protection"
  i18n_a11y: "Queue accessible; reason codes localized"
  acceptance: "AC-SF30.2.3: 6 redemptions within 10 minutes on one account (limit 5) flags and holds the 6th; a manual +500 adjustment approved by the same user is rejected; approved adjustment creates one ledger entry"
  dependency: "M60 anomaly case management, M02 staff relationship data"
- id: M30.F30.2.SF30.2.4
  name: "self-service history/dispute"
  phase: 5
  release: R1
  actors: [guest, guest_relations, loyalty_worker]
  screens: [SCR-GUEST-points-history, SCR-GUEST-points-dispute, SCR-FD-loyalty-dispute-queue]
  inputs: [account_id, entry_id_or_stay_id, dispute_reason, attachment_ref]
  states: [open, investigating, resolved_credit, resolved_no_change, escalated]
  api: "GET /v1/loyalty/accounts/{id}/ledger; POST /v1/loyalty/accounts/{id}/disputes"
  events: [PointsDisputeOpened, PointsDisputeResolved]
  data: [points_dispute, points_ledger_entry]
  rules: ["Guest sees every entry with date, source (booking/receipt number), campaign, pending/available/expired status", "Missing-points claim for a stay verifies the stay is settled and eligible before credit via SF30.2.3 adjustment", "SLA with escalation via M63"]
  security: "guest sees own account only (object-level authorization test); attachments malware-scanned"
  failure_cases: [claim_for_other_guest_stay, duplicate_dispute, attachment_malware]
  finance_report_effect: "Resolved credits flow as adjustments; dispute volume in guest-service reports"
  i18n_a11y: "History export PDF/CSV; accessible table; EN/AR"
  acceptance: "AC-SF30.2.4: Guest A requesting ledger of account B receives 404; a missing-points claim for a settled eligible stay results in one adjustment entry linked to the dispute"
  dependency: "M55 case/SLA, M63 workflow"
- id: M30.F30.2.SF30.2.5
  name: "service-specific consent"
  phase: 5
  release: R1
  actors: [guest, dpo, marketing_manager]
  screens: [SCR-GUEST-loyalty-enrol-consent, SCR-GUEST-privacy-preferences]
  inputs: [consent_purpose (programme_membership | marketing_email | marketing_sms | profiling_for_offers), channel, jurisdiction_profile_id, terms_version]
  states: [not_given, given, withdrawn]
  api: "POST /v1/guests/{gid}/consents (M02 port); GET /v1/loyalty/terms/{version}"
  events: [ConsentGiven, ConsentWithdrawn, LoyaltyTermsAccepted]
  data: [consent_record (M02), loyalty_terms_version]
  rules: ["Programme membership consent (terms) is separate from marketing consent; declining marketing does not block earning", "Withdrawal of profiling consent stops personalised offers but not ledger processing required for the contract", "Terms version is stored on the account and re-acceptance is required on material change"]
  security: "consent records immutable with timestamp, channel, text version"
  failure_cases: [terms_version_missing_for_language, withdrawal_during_campaign]
  finance_report_effect: "None directly; consent status is a filter in campaign reporting (M52)"
  i18n_a11y: "Unbundled checkboxes, not pre-ticked; EN/AR terms with same version id; screen-reader labels"
  acceptance: "AC-SF30.2.5: Enrolling with marketing unchecked succeeds and earns points; a campaign send excludes the account; withdrawing consent is effective for the next scheduled send"
  dependency: "M02 consent service, M52 suppression"
- id: M30.F30.2.SF30.2.6
  name: "cash wallet PSP boundary and regulatory gate"
  phase: 5
  release: R1
  actors: [compliance_officer, financial_controller, tenant_admin, guest]
  screens: [SCR-ADMIN-feature-gates, SCR-COMP-wallet-gate-evidence, SCR-GUEST-points-wallet]
  inputs: [market, psp_partner_id, licence_evidence_ref, legal_opinion_ref, kyc_model, safeguarding_model, gate_status]
  states: [not_offered, evaluating, partner_contracted, sandbox_tested, enabled, suspended]
  api: "GET /v1/tenants/{tid}/feature-gates/cash-wallet; POST /v1/tenants/{tid}/feature-gates/cash-wallet/decisions (dual approval)"
  events: [CashWalletGateChanged]
  data: [feature_activation_gate (M44), external_wallet_link, partner_contract_record]
  rules: ["Points are never convertible to cash, never transferable peer-to-peer and never topped up with money", "No MetriStay table holds a customer cash balance; a cash/stored-value wallet exists only as a licensed bank/PSP product and MetriStay stores only a mirrored reference and partner statement lines for reconciliation", "Gate default = not_offered in every market; enabling requires partner licence evidence, legal opinion for the market (e.g. CBO PSP policy in Oman) and dual approval", "Guest UI must never label points as money or 'wallet balance in OMR'"]
  security: "gate change requires compliance_officer + financial_controller; enforcement in API and jobs, not UI only"
  failure_cases: [gate_enabled_without_evidence, partner_licence_expired, ui_shows_cash_wording]
  finance_report_effect: "When enabled later, partner-held balances are off-balance-sheet references reconciled to partner statements; none in R1"
  i18n_a11y: "Wording review in EN/AR to avoid monetary terms for points"
  acceptance: "AC-SF30.2.6: Schema check finds no cash-balance column for guests; API call to any cash-wallet endpoint returns 403 gate_disabled in all five fixture markets; UI copy test finds no currency-denominated balance for points"
  dependency: "D-403 cash wallet decision; licensed PSP partner (none contracted); M28 PSP port"
```

### M30 key invariants
1. Points balance = sum of immutable `points_ledger_entry` rows; no mutable balance without ledger backing.
2. A source line earns at most once; a refund reverses at most once (`idempotency_key` = source/refund id).
3. Points are redeemable only against eligible hotel purchases; no cash-out, top-up, peer transfer or referral-to-points conversion.
4. Cash/stored-value wallet is disabled everywhere until the SF30.2.6 gate is passed with licensed-partner and counsel evidence.
5. Liability roll-forward reconciles to the ledger with zero difference each period.

### M30 module acceptance (Section G)
| Test | Scenario | Pass condition |
|---|---|---|
| AT-G07.1 | Guest earns on eligible room + F&B, not on excluded items | Pending points equal formula; ineligible lines earn zero |
| AT-G07.2 | Guest redeems points as partial tender at checkout | One capture, folio shows points tender + card remainder |
| AT-G07.3 | Refund of the stay after redemption | Points and charge reversed exactly once each; duplicate refund webhook is a no-op |
| AT-G08.x | Month-end liability | Liability snapshot ties to ledger; breakage and redemption cost shown by department |
| AT-G20.x | Duplicate provider webhook / network outage | No duplicate earn/redeem; POS offline shows points unavailable |

### M30 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-401 | Accounting treatment of points (deferred revenue/material right vs provision), cost-per-point and breakage method | Financial Controller + external auditor | Deferred-revenue model with stand-alone selling price per point; breakage estimated from history, labelled `estimate` until approved |
| D-402 | Earn rates, tiers, pending window, expiry period, tier grace | Marketing Manager + GM | 1 pt per 1 major currency unit eligible net spend; 2 tiers; pending until checkout + 14 days; 18-month expiry from earn |
| D-403 | Whether to offer a cash/stored-value wallet at all, in which market and with which licensed PSP/bank | Product Owner + Legal Counsel (Oman, CBO PSP policy) + Treasury | Not offered in R1; gate stays `not_offered` |
| D-404 | Consumer-law terms for points per market (expiry notice, unfair terms, tax on redemption) | Local counsel per market via M44 | Expiry suspended for any market whose loyalty rule pack is not `verified` |

---

## 2. M31 — Referrals/growth (MetriStay Network, MetriStay Partner Hub)

| Header | Value |
|---|---|
| Purpose | **Single-tier direct booking referral.** An enrolled, approved referrer may receive one contractual commission only when a person it directly referred books a hotel/property through its attributable code/link, completes the paid stay, and the transaction is not reversed. Commission = contract rate x **realized contribution margin of that specific booking**, paid by the company as a marketing expense. No recruiting rewards, no downline/upline, no team volume, no ranks, no joining fee, no purchase requirement. The guest can book without joining and pays the same price unless a separately approved promotion applies. |
| Phases / release | Phase 5: single-tier build, sandbox and test (R1). Phase 6: market-specific activation gate (R1). Phase 8: partner distribution expansion (Later, see M37 F37.3). **Oman payout activation requires a documented legal and tax opinion** on final terms, referrer categories, promotion channels and licensing (Decision 105/2021 assessment, social-media marketing licence, promotional permit, travel/tourism licence, VAT/withholding). Each other market needs its own review. |
| Bounded context | `referral` (finance-owned ledger inside it; payouts executed by M28 F28.2) |
| System-of-record entities | `referral_program_policy` (per market/legal entity/category/campaign, includes kill switch), `referrer` (individual or approved company; **no parent/sponsor field**), `referral_agreement` (versioned contract acceptance), `referral_code` (code/link), `referral_touch` (click/code-entry evidence, privacy-limited), `referral_attribution` (**at most one per booking**), `referral_dispute`, `margin_formula_version`, `booking_margin_calculation`, `commission_ledger_entry` (append-only: pending/approved/paid/reversed), `commission_payout` (links to M28 disbursement), `referrer_tax_profile` |
| Explicitly absent (schema lint enforced) | Any referrer-to-referrer relationship: no `sponsor_id`, `parent_referrer_id`, `upline`, `downline`, `team`, `rank`, `level` column or table; no recursive CTE over referrers; no commission rows whose source is another referrer's activity. |
| Referenced | `reservation`/`stay` (M05), `folio`/`invoice` (M08), `payment`/`refund`/`chargeback` (M28), channel/OTA fee (M07), cost facts (M19/M32 allocation), `jurisdiction_profile`/`rule_pack`/`feature_activation_gate` (M44), `consent_record` (M02), `supplier_payable`/`disbursement` (M20/M28) |
| Dependencies | M05, M07, M08, M19, M20, M28 F28.2, M32 cost versions, M44 gates, M02 consent/cookies (M51 SF51.2.4), M60 anomaly cases |
| Publishes | `ReferrerApproved`, `ReferralCodeIssued`, `ReferralTouchRecorded`, `ReferralAttributed`, `ReferralAttributionDisputed`, `ReferralQualified`, `BookingMarginCalculated`, `CommissionAccrued`, `CommissionApproved`, `CommissionPaid`, `CommissionReversed`, `ReferralPolicyKillSwitchChanged` |

**Worked example (fixture FX-M31-01, OMR, 3 decimals):** net collected room revenue OMR 100.000 (100000 baisa) − contractually defined attributable costs OMR 70.000 (70000 baisa: e.g. VAT excluded upfront, OTA/gateway fee, variable fulfilment cost per formula version) = eligible margin OMR 30.000; contract rate 20% → commission **OMR 6.000** (6000 baisa), accrued as pending only after checkout, cleared payment and the refund window. The 20% is a business decision recorded on the agreement, not a legal safe harbour.

### F31.1 Single-tier direct booking

```yaml
- id: M31.F31.1.SF31.1.1
  name: "approved referrer agreement/link/code"
  phase: 5
  release: R1
  actors: [referrer, referral_program_admin, compliance_officer, referral_worker]
  screens: [SCR-HUB-apply, SCR-HUB-agreement-sign, SCR-HUB-my-code, SCR-ADMIN-referrer-approvals]
  inputs: [referrer_type (individual | company), legal_name, country, legal_entity_scope, permitted_channels, agreement_version_id, signature_envelope_id, disclosure_text_version]
  states: [applied, kyc_pending, approved, suspended, terminated]
  api: "POST /v1/referral/referrers; POST /v1/referral/referrers/{rid}/approve (maker-checker); POST /v1/referral/referrers/{rid}/codes"
  events: [ReferrerApplied, ReferrerApproved, ReferralCodeIssued, ReferrerSuspended]
  data: [referrer, referral_agreement, referral_code, referrer_tax_profile, signature_envelope (M41)]
  rules: ["No joining fee, purchase, subscription or quota is ever required; any fee field is absent from the schema", "Agreement is versioned and scoped to country, hotel legal entity and permitted promotional channels; code works only inside that scope", "Referrer record has no reference to another referrer; the application form has no 'invited by' field that affects compensation", "Codes are unguessable (>=64-bit random) and optionally vanity after review; a code maps to exactly one referrer"]
  security: "referrer identity via Partner Hub login with MFA; approval by referral_program_admin + compliance_officer; agreement signed via M41 envelope"
  failure_cases: [market_gate_disabled, kyc_incomplete, vanity_code_collision, agreement_version_retired]
  finance_report_effect: "None until commission accrues; referrer count by market in programme report"
  i18n_a11y: "Agreement and compensation disclosure in EN/AR (plus market language pack); accessible signing"
  acceptance: "AC-SF31.1.1: Application with any fee or 'sponsor' parameter is rejected by schema; code issued in Oman-scope does not attribute a Portugal legal-entity booking; approval by a single user is rejected"
  dependency: "M41 signature envelope, M44 feature gate per market, D-407 referrer categories"
- id: M31.F31.1.SF31.1.2
  name: "guest consent and deterministic one-owner attribution"
  phase: 5
  release: R1
  actors: [guest, booker, referral_worker, referral_program_admin]
  screens: [SCR-GUEST-booking-referral-code-field, SCR-GUEST-cookie-preferences, SCR-ADMIN-referral-attribution-disputes]
  inputs: [reservation_id, touches (code_entered | link_click with timestamp), consent_state, manual_assignment_request, channel]
  states: [none, attributed, disputed, rejected, locked]
  api: "Consumer of ReservationConfirmed; POST /v1/properties/{pid}/referral/attributions/{reservation_id}/resolve (internal); POST .../disputes"
  events: [ReferralTouchRecorded, ReferralAttributed, ReferralAttributionDisputed, ReferralAttributionRejected]
  data: [referral_touch, referral_attribution (unique on reservation_id), referral_dispute]
  rules: ["At most ONE referral_attribution row per booking (unique constraint); precedence: explicit code entered at booking > last qualifying link click within window (D-408) > approved manual assignment; ties or conflicting claims -> disputed, never split and never both paid", "Self-referral (referrer = booker/guest/payer, same payment instrument token, same household/device fingerprint where lawfully used) is rejected", "Link-click touches recorded only with cookie/analytics consent or as first-party code entry; referrer never sees guest PII", "Channel/OTA bookings are ineligible unless the policy explicitly allows the channel", "No recursive graph: attribution reads only the code->referrer map"]
  security: "attribution computed server-side; guest PII segregated from referrer views; dispute resolution by program admin with audit"
  failure_cases: [two_codes_claimed, code_after_booking, cookie_withdrawn, self_referral, bot_clicks]
  finance_report_effect: "Attributed bookings counted in acquisition channel reports (M51/M32) as 'referral' source"
  i18n_a11y: "Optional code field clearly labelled, not required to book; consent banner accessible and RTL"
  acceptance: "AC-SF31.1.2 (non-negotiable): Referrers A and B both claim reservation R -> either one attribution or state disputed; commission ledger for R never contains entries for both; a booking without any code books normally at the same price"
  dependency: "M51 cookie/consent (SF51.2.4), D-408 attribution window"
- id: M31.F31.1.SF31.1.3
  name: "completed paid stay qualification"
  phase: 5
  release: R1
  actors: [referral_worker, finance_clerk]
  screens: [SCR-FIN-referral-qualification-queue]
  inputs: [reservation_id, stay_status, folio_settlement_status, payment_cleared_at, refund_window_days, chargeback_status]
  states: [awaiting_stay, awaiting_settlement, in_refund_window, qualified, disqualified]
  api: "Job: referral.qualify daily; GET /v1/properties/{pid}/referral/qualifications?status="
  events: [ReferralQualified, ReferralDisqualified]
  data: [referral_attribution, qualification_check]
  rules: ["Qualifies only after checkout, fully cleared payment (not preauth), and end of refund window (D-409); cancelled, no-show and walked bookings never qualify", "Amendments re-run qualification against the final stay", "Corporate-credit stays qualify only once the AR invoice is paid"]
  security: "service identity; read-only for finance clerks"
  failure_cases: [payment_status_unknown, stay_extended_after_checkout, late_chargeback]
  finance_report_effect: "Qualified count and value drive accrual estimate visibility"
  i18n_a11y: "Staff queue accessible; statuses localized"
  acceptance: "AC-SF31.1.3: A no-show booking with referral code never qualifies; a stay paid by corporate AR qualifies only after the receipt is allocated; a qualified booking re-evaluates after amendment"
  dependency: "M05 stay status, M08/M20 settlement, M28 cleared status, D-409"
- id: M31.F31.1.SF31.1.4
  name: "booking-specific margin formula/version"
  phase: 5
  release: R1
  actors: [financial_controller, referral_program_admin, referral_worker, auditor]
  screens: [SCR-FIN-margin-formula-versions, SCR-FIN-booking-margin-calculation]
  inputs: [formula_version_id, net_collected_revenue_components, allowed_deductions (tax, refunds, ota_fee, gateway_fee, variable_fulfilment_cost, excluded_services), multi_room_rule, rounding_rule, rate_percent]
  states: [draft, approved, active, superseded]
  api: "POST /v1/referral/margin-formulas (maker-checker); POST /v1/properties/{pid}/referral/margin-calculations/{reservation_id} (internal, Idempotency-Key)"
  events: [MarginFormulaApproved, BookingMarginCalculated]
  data: [margin_formula_version, booking_margin_calculation]
  rules: ["eligible_margin = net_collected_revenue - sum(allowed attributable costs per formula version); commission = max(0, eligible_margin) x rate; zero or negative margin earns zero", "Formula version, all components, cost sources and rate stored on the calculation (reproducible)", "Multi-room bookings computed per booking total unless contract says per room; partial refunds recalculate", "No earning estimate is displayed to referrers until an approved formula version with defined deductions exists", "Worked example FX-M31-01: 100.000 - 70.000 = 30.000; 20% -> 6.000 OMR"]
  security: "formula maker-checker (financial_controller approves); calculation immutable; recalculation creates a new calculation row"
  failure_cases: [cost_source_missing, formula_not_active_at_checkout_date, currency_mismatch, rounding_dispute]
  finance_report_effect: "Commission expense recognised as marketing expense (M19 mapping); margin inputs drill to M32 cost versions"
  i18n_a11y: "Statement shows components with localized currency formatting (OMR 3 decimals)"
  acceptance: "AC-SF31.1.4 (worked example): FX-M31-01 yields commission exactly 6000 baisa; a booking with revenue 100.000 and costs 100.000 or 105.000 yields 0, never negative; changing formula after calculation does not change the stored result"
  dependency: "M32 cost allocation versions, M07 channel fees, M28 gateway fees, D-405, D-412"
- id: M31.F31.1.SF31.1.5
  name: "pending/approved/paid/reversed commission"
  phase: 5
  release: R1
  actors: [finance_approver, payment_releaser, referral_worker, referrer]
  screens: [SCR-FIN-commission-ledger, SCR-FIN-commission-approval, SCR-HUB-statement]
  inputs: [booking_margin_calculation_id, approval_decision, payout_batch_id, reversal_reason]
  states: [pending, approved, paid, reversed, held]
  api: "POST /v1/referral/commissions/{id}/approve; POST /v1/referral/commissions/{id}/reverse (internal, Idempotency-Key = source event); GET /v1/referral/referrers/{rid}/commissions"
  events: [CommissionAccrued, CommissionApproved, CommissionPaid, CommissionReversed]
  data: [commission_ledger_entry, commission_payout]
  rules: ["One pending entry per qualified attribution; unique (reservation_id, entry_type) prevents double accrual", "Cancellation, chargeback, no-show conversion or refund after accrual reverses once (Idempotency-Key = refund/chargeback id); partial refund recalculates and posts the difference", "Reversal after payment creates a receivable/offset against future commission per contract, never silent deletion", "Only the direct referrer of that booking can hold entries for it"]
  security: "approval by finance_approver; payment release by different payment_releaser; step-up MFA"
  failure_cases: [duplicate_refund_event, reversal_after_paid, approver_equals_releaser]
  finance_report_effect: "Accrued commission liability, paid commission expense and clawback receivable posted to GL; referral ROI by market"
  i18n_a11y: "Statement accessible; status text plus icon"
  acceptance: "AC-SF31.1.5 (non-negotiable): Refund of a qualified booking reverses its commission exactly once, even with duplicate refund webhooks; a referral whose referred guest later refers guest C yields no entry for A on C's booking"
  dependency: "M28 F28.2 disbursement, M19 posting"
- id: M31.F31.1.SF31.1.6
  name: "tax/disclosure/payout"
  phase: 5
  release: R1
  actors: [finance_approver, payment_releaser, compliance_officer, referrer]
  screens: [SCR-HUB-tax-and-payout-details, SCR-FIN-referral-payout-batch, SCR-HUB-disclosure-kit]
  inputs: [referrer_tax_profile, tax_residency, withholding_rule_pack_id, invoice_or_self_billing_doc, payout_method_token, disclosure_template_version]
  states: [tax_profile_incomplete, ready, batched, submitted, settled, failed]
  api: "POST /v1/referral/payout-batches (dual approval) -> M28 F28.2 disbursement port"
  events: [ReferralPayoutBatched, CommissionPaid, ReferralPayoutFailed]
  data: [referrer_tax_profile, commission_payout, disbursement (M28), tax_document]
  rules: ["Payout only via authorized bank/PSP channel through M28 F28.2; bank details verified, change requires review", "Withholding/VAT/income-tax treatment from verified M44 rule pack; unverified -> payout blocked", "Referrer must use the approved compensation disclosure in every permitted channel (FTC-style clear disclosure where applicable)", "Guest price unchanged by referral"]
  security: "payout tokens vaulted; dual approval; referrer sees masked account"
  failure_cases: [rule_pack_unverified, bank_rejection, sanctions_hit, missing_tax_document]
  finance_report_effect: "Payout settles liability; withholding posted as tax payable; tax report per market"
  i18n_a11y: "Disclosure kit in EN/AR and market language; accessible forms"
  acceptance: "AC-SF31.1.6: Payout batch containing an Oman referrer with withholding rule pack in draft is rejected for that line; approved batch settles and posts once"
  dependency: "M28 F28.2 payout partner, M44 tax rule packs, D-406, D-410"
- id: M31.F31.1.SF31.1.7
  name: "Oman legal activation gate and no recruitment reward"
  phase: 6
  release: R1
  actors: [compliance_officer, referral_program_admin, financial_controller, tenant_admin]
  screens: [SCR-COMP-referral-market-gate, SCR-ADMIN-feature-gates]
  inputs: [market (OM), legal_opinion_ref, tax_opinion_ref, approved_terms_version, approved_referrer_categories, approved_promotion_channels, licence_refs (social_media_marketing, promotional_permit, tourism_agency if applicable)]
  states: [build_only, sandbox_testing, awaiting_opinion, approved_for_payout, suspended]
  api: "POST /v1/tenants/{tid}/feature-gates/referral-payout/markets/OM/decision (dual approval, evidence required)"
  events: [ReferralMarketGateChanged]
  data: [feature_activation_gate (M44), referral_program_policy, evidence_source]
  rules: ["Oman gate cannot move to approved_for_payout without both a documented legal opinion and tax opinion covering the exact terms, categories and channels", "No recruitment reward of any kind exists: signups, referrer invitations and team size generate zero value", "Product is never represented as network/pyramid marketing or 'approved' by any authority", "Build and sandbox testing may proceed while gate is awaiting_opinion; payouts and live attribution earnings display remain blocked"]
  security: "gate change: compliance_officer + financial_controller; evidence stored immutably; enforced in API and all jobs"
  failure_cases: [opinion_expired_or_terms_changed, attempt_to_enable_without_evidence, terms_version_mismatch]
  finance_report_effect: "While blocked, no commission accrual posts to GL for the market; sandbox figures labelled test"
  i18n_a11y: "Gate screen accessible; evidence list with dates"
  acceptance: "AC-SF31.1.7 (non-negotiable): A referrer inviting another referrer who enrols and books nothing earns zero; enrolment of 10 referrers creates zero ledger entries; enabling Oman payout without both opinion references returns 422"
  dependency: "D-406 Oman legal/tax opinion; M44 gates"
```

### F31.2 MetriStay Partner Hub

```yaml
- id: M31.F31.2.SF31.2.1
  name: "approved jurisdiction/contract and referrer category"
  phase: 5
  release: R1
  actors: [referral_program_admin, compliance_officer]
  screens: [SCR-ADMIN-referral-policies, SCR-COMP-referral-market-matrix]
  inputs: [market, legal_entity_id, referrer_category (guest_individual | travel_professional | corporate_partner | influencer), contract_template_version, channels_allowed, rate_percent, effective_dates]
  states: [draft, pending_review, active, suspended, retired]
  api: "PUT /v1/referral/policies/{policy_id} (versioned, maker-checker)"
  events: [ReferralPolicyActivated, ReferralPolicySuspended]
  data: [referral_program_policy, referral_agreement_template, rule_pack (M44)]
  rules: ["Each (market, legal entity, category, campaign) combination is a separate policy with its own verified rule-pack references", "A category disallowed by counsel for a market cannot be selected at onboarding", "Policies never define multi-level rates; schema has a single rate field"]
  security: "maker-checker; audit"
  failure_cases: [policy_without_rule_pack, overlapping_versions, category_disallowed]
  finance_report_effect: "Policy id is a dimension on commission expense"
  i18n_a11y: "Admin forms EN/AR"
  acceptance: "AC-SF31.2.1: Attempt to add a second-level rate field via API is rejected by schema validation; a policy for Saudi Arabia without verified rule pack cannot be activated"
  dependency: "M44 rule packs; D-407, D-411"
- id: M31.F31.2.SF31.2.2
  name: "direct referrer onboarding"
  phase: 5
  release: R1
  actors: [referrer, referral_program_admin, compliance_officer]
  screens: [SCR-HUB-onboarding-wizard, SCR-HUB-identity-and-tax, SCR-ADMIN-referrer-approvals]
  inputs: [identity_verification_ref, sanctions_screen_ref, tax_id_where_required, payout_details_token, accepted_terms_version]
  states: [started, identity_pending, tax_pending, review, approved, rejected]
  api: "POST /v1/referral/referrers/{rid}/onboarding-steps/{step}"
  events: [ReferrerOnboardingStepCompleted, ReferrerApproved, ReferrerRejected]
  data: [referrer, referrer_tax_profile, identity_verification_check (M41)]
  rules: ["Onboarding is free and not tied to any purchase", "Company referrers verified via M46 legal-entity checks where applicable", "Staff of the hotel/legal entity are excluded or flagged per policy"]
  security: "MFA; PII minimisation; tax IDs encrypted and masked"
  failure_cases: [sanctions_match, staff_conflict, document_expired]
  finance_report_effect: "None"
  i18n_a11y: "Wizard accessible, resumable, EN/AR"
  acceptance: "AC-SF31.2.2: Onboarding completes without any payment step; a hotel employee applying is flagged for conflict review"
  dependency: "M41 identity check, M46 entity verification"
- id: M31.F31.2.SF31.2.3
  name: "own verified paid-and-completed booking"
  phase: 5
  release: R1
  actors: [referrer, referral_worker]
  screens: [SCR-HUB-my-referred-bookings]
  inputs: [referrer_id, period]
  states: [attributed, qualifying, qualified, disqualified]
  api: "GET /v1/referral/referrers/{rid}/bookings"
  events: [ReferralQualified]
  data: [referral_attribution, booking_margin_calculation]
  rules: ["Referrer sees only bookings directly attributed to its own code: booking reference token, stay month, status and commission; no guest name, contact, room or price detail beyond what the contract allows", "No view of any other referrer's activity, count or earnings"]
  security: "object-level authorization on referrer_id; privacy-preserving pseudonymous booking token"
  failure_cases: [bola_attempt, stale_status]
  finance_report_effect: "None"
  i18n_a11y: "Accessible list with status text"
  acceptance: "AC-SF31.2.3: Referrer A querying B's bookings gets 404; A's list shows no guest PII fields in API response schema"
  dependency: "SF31.1.2, SF31.1.3"
- id: M31.F31.2.SF31.2.4
  name: "contribution-margin commission and clawback"
  phase: 5
  release: R1
  actors: [referral_worker, finance_approver, referrer]
  screens: [SCR-HUB-statement, SCR-FIN-commission-ledger]
  inputs: [calculation_id, refund_or_chargeback_event, contract_clawback_terms]
  states: [pending, approved, paid, reversed, offset_pending]
  api: "Internal; GET /v1/referral/referrers/{rid}/statement?period="
  events: [CommissionAccrued, CommissionReversed]
  data: [commission_ledger_entry, booking_margin_calculation]
  rules: ["Statement shows per booking token: revenue and cost components as permitted by contract, formula version, rate, commission, status", "Clawback after payment offsets future commissions per contract; unrecovered amounts escalate to finance, never auto-debit a bank account without mandate"]
  security: "statement generated server-side; immutable monthly statement PDF with hash"
  failure_cases: [clawback_exceeds_future_earnings, contract_silent_on_clawback]
  finance_report_effect: "Clawback receivable and write-off tracked"
  i18n_a11y: "Statement PDF tagged for accessibility, EN/AR"
  acceptance: "AC-SF31.2.4: After paying OMR 6.000 on FX-M31-01, a full refund creates one -6.000 reversal and an offset_pending balance of 6.000 that nets against the next accrual"
  dependency: "SF31.1.4, SF31.1.5"
- id: M31.F31.2.SF31.2.5
  name: "tax/payout reconciliation and privacy-limited dashboard"
  phase: 5
  release: R1
  actors: [referrer, finance_clerk, financial_controller]
  screens: [SCR-HUB-dashboard, SCR-FIN-referral-reconciliation]
  inputs: [period, payout_batch_id, bank_statement_lines]
  states: [unreconciled, matched, exception]
  api: "GET /v1/referral/referrers/{rid}/dashboard; GET /v1/referral/reconciliation?period="
  events: [ReferralPayoutReconciled, ReferralPayoutException]
  data: [commission_payout, bank_statement_line (M20), commission_ledger_entry]
  rules: ["Dashboard totals: pending, approved, paid, reversed for own code only; no leaderboard, rank or team metric", "Payout reconciled to bank/PSP confirmation before marking paid (settled)"]
  security: "referrer scope; finance scope for reconciliation"
  failure_cases: [bank_confirmation_missing, amount_mismatch]
  finance_report_effect: "Referral programme cost, ROI and payout reconciliation in M32 marketing reports"
  i18n_a11y: "Dashboard charts with text tables; RTL"
  acceptance: "AC-SF31.2.5: Dashboard API has no fields for other referrers or rankings; a payout without bank confirmation remains 'submitted' not 'paid'"
  dependency: "M20 bank import, M28 status"
- id: M31.F31.2.SF31.2.6
  name: "no recruiting/downline reward"
  phase: 5
  release: R1
  actors: [referral_program_admin, auditor, it_admin]
  screens: [SCR-COMP-referral-structure-audit]
  inputs: [schema_snapshot, commission_ledger_sample]
  states: [compliant, violation_detected]
  api: "CI check: schema-lint referral context; GET /v1/referral/audit/structure"
  events: [ReferralStructureAuditRun]
  data: [referrer, referral_attribution, commission_ledger_entry]
  rules: ["CI fails if any referral table has a self-referencing FK, sponsor/parent/upline/downline/level/rank/team column, or a recursive query on referrers", "Every commission entry's source booking's attribution.referrer_id equals the entry's referrer_id (data invariant check nightly)", "Marketing templates reviewed to exclude recruitment messaging"]
  security: "audit read-only; violations page compliance_officer"
  failure_cases: [schema_regression, data_invariant_violation]
  finance_report_effect: "Violation freezes payouts for affected entries"
  i18n_a11y: "Audit report accessible"
  acceptance: "AC-SF31.2.6 (non-negotiable): Adding a parent_referrer_id column in a migration fails CI; fixture where guest B (referred by A) refers guest C produces commission for B (if enrolled) on C and none for A"
  dependency: "M33 CI tooling"
- id: M31.F31.2.SF31.2.7
  name: "kill switch and compliance audit"
  phase: 5
  release: R1
  actors: [referral_program_admin, compliance_officer, referral_worker, auditor]
  screens: [SCR-ADMIN-referral-kill-switch, SCR-COMP-referral-audit-trail]
  inputs: [scope (country | legal_entity | property | referrer_category | campaign | referrer), action (disable | enable), reason]
  states: [enabled, disabled]
  api: "POST /v1/referral/kill-switch (step-up; takes effect immediately); GET /v1/referral/audit?booking_id="
  events: [ReferralPolicyKillSwitchChanged]
  data: [referral_program_policy, feature_activation_gate, audit_log]
  rules: ["Disable takes effect in API, attribution, qualification, accrual and payout jobs on next read (policy cache TTL <= 60 s, payout job re-checks at execution)", "A disabled jurisdiction rejects payout regardless of UI state or pre-approved batch", "Existing earned commissions remain governed by contract: held, not deleted", "Audit per booking shows booking, revenue, cost components, formula version, authorization and payment reference"]
  security: "step-up MFA; audit immutable"
  failure_cases: [cache_staleness, batch_already_submitted_to_bank]
  finance_report_effect: "Held commissions disclosed as held liability"
  i18n_a11y: "Big clear status, confirmation dialog accessible"
  acceptance: "AC-SF31.2.7 (non-negotiable): After disabling Oman, a payout batch approved earlier is rejected by the job and API for Oman lines even when invoked directly; the audit for FX-M31-01 displays booking id, revenue 100.000, cost components totalling 70.000, formula version, approver and payment reference"
  dependency: "M44 feature gates"
```

### M31 key invariants
1. `referral_attribution` is unique per booking; at most one direct referrer; conflicts are `disputed`, never split or double paid.
2. No referrer-to-referrer relationship exists in schema, data or code; recruitment generates zero value.
3. Commission = max(0, eligible_margin) × rate for the specific booking, only after checkout, cleared payment and refund window; reversal happens once per refund/chargeback event.
4. Kill switch and market gates are enforced server-side and in every background job; Oman payouts require the documented legal+tax opinion.
5. Referrer never sees guest PII; guest price is unchanged by referral.

### M31 module acceptance — Section E non-negotiable tests (AT-G07)
| Test | Section E requirement | Pass condition |
|---|---|---|
| AT-G07.4 | Two referrers claiming a booking | One attribution or `disputed`; never two payable entries |
| AT-G07.5 | A referrer recruiting another referrer | Recruiter earns zero; no ledger rows created by enrolment |
| AT-G07.6 | Guest B booked through A's code later refers guest C | A is paid only for B's eligible booking, never for C's |
| AT-G07.7 | Refund of a qualified booking | Commission reversed exactly once (duplicate refund events are no-ops) |
| AT-G07.8 | Booking with zero or negative eligible margin | Commission = 0 |
| AT-G07.9 | Disabled jurisdiction | Payout rejected regardless of UI state, via API and background job |
| AT-G07.10 | Audit | Shows booking, revenue, cost components, formula version, authorization and payment reference |
| AT-G07.11 | Worked example FX-M31-01 | OMR 100.000 − 70.000 = 30.000 × 20% = OMR 6.000 accrued after checkout/cleared payment/refund window |
| AT-G07.12 | Oman legal gate | Build/sandbox works; Oman payout blocked until legal + tax opinion evidence recorded |

### M31 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-405 | Commission rate(s) and contractual allowed deductions list, multi-room and partial-refund rules | Commercial Director + Financial Controller + Legal | 20% placeholder in fixtures only; deductions = tax, refunds, OTA/gateway fees, variable fulfilment cost, excluded ancillaries |
| D-406 | Oman legal opinion (Decision 105/2021, social-media marketing licence, promotional permit, tourism/travel agency licence) and tax opinion (VAT, withholding, income treatment) | Oman external counsel + tax adviser; accountable: Compliance Officer | Oman gate = `awaiting_opinion`; no Oman payout |
| D-407 | Permitted referrer categories per market (individual guest, travel professional, company, influencer) and KYC/tax documents | Compliance Officer | Individuals and companies allowed in sandbox; live only per verified policy |
| D-408 | Attribution window, cookie policy, device-signal use for self-referral detection | Product Owner + DPO | Code at booking wins; last click within 30 days with consent; no fingerprinting without DPO approval |
| D-409 | Refund window length and commission calculation date | Financial Controller | Checkout + 30 days, calculation on day 31 using formula version active at checkout |
| D-410 | Payout channel/partner and payout frequency | Treasury / Financial Controller | Monthly bank transfer via M28 F28.2 authorized partner; manual bank file fallback |
| D-411 | Referral review for Canada, Pakistan, Saudi Arabia, Portugal (consumer, advertising, tourism, privacy, tax) | Local counsel per market | All four markets `build_only` |
| D-412 | Rounding of commission to currency minor units | Financial Controller | Round half-even at currency minor unit after multiplication |

---

## 3. M32 — Management BI (management reports and profitability)

| Header | Value |
|---|---|
| Purpose | Governed KPI dictionary and management reporting from ledgers and operational facts: occupancy/ADR/RevPAR/TRevPAR, pickup/pace, department P&L (rooms, bar, club, catering, parking, utilities, maintenance, fees), shared-cost allocation, operating result vs net income, budget/forecast/prior year, source coverage and close status, drill-through, GM flash, custom pivots and scheduled exports. Every figure is labelled `provisional`, `estimate`, `reconciled` or `certified` with source freshness. |
| Phases / release | Phase 2 foundation (KPI dictionary, room commerce, GM flash basic); Phase 3 pickup/corporate; Phases 4–6 department P&L, allocation, property result and full suite. R1. |
| Bounded context | `analytics` (read models; M65 owns governance definitions and lineage, M19 owns GL) |
| System-of-record entities | `kpi_definition` (versioned; shared glossary with M65), `report_definition`, `report_version`, `report_run`, `report_schedule`, `allocation_policy_version`, `allocation_run`, `budget_version`, `budget_line`, `data_freshness_status`, `restatement_record`; read models `fact_room_night`, `fact_revenue_line`, `fact_cost_line`, `fact_labor_cost` (aggregated, salary-confidential), `fact_utility_usage` |
| Referenced | M03/M05 inventory and stays, M07 channel fees, M08 folio, M13–M17 outlet facts, M19 GL journals/periods, M20 AP/AR, M21/M50 purchasing and stock, M22–M25 utilities, M26 maintenance, M27 payroll aggregates, M28 PSP fees, M30/M31 loyalty/referral costs, M53 `forecast_version`, M65 lineage |
| Dependencies | M19 period close, M65 KPI governance, M02 field/row permissions, M63 scheduler/notifications |
| Publishes | `KpiDefinitionVersioned`, `KpiSnapshotPublished`, `GmFlashPublished`, `AllocationRunCompleted`, `ReportRunCompleted`, `ScheduledReportDelivered`, `PeriodRestated`, `DataFreshnessBreached` |

### F32.1 Room commerce

```yaml
- id: M32.F32.1.SF32.1.1
  name: "occupancy and inventory exclusions"
  phase: 2
  release: R1
  actors: [gm, revenue_manager, owner, night_audit_worker]
  screens: [SCR-MGMT-occupancy, SCR-MGMT-kpi-dictionary]
  inputs: [business_date_range, room_type, exclusion_policy (ooo_excluded | oos_included | comp_counted | house_use_counted), kpi_version]
  states: [provisional, audited, restated]
  api: "GET /v1/properties/{pid}/kpis/occupancy?from=&to=&kpi_version="
  events: [KpiDefinitionVersioned, KpiSnapshotPublished]
  data: [kpi_definition, fact_room_night, room_inventory_snapshot]
  rules: ["Occupancy = occupied room nights / available room nights; available excludes OOO per policy and includes OOS; comp and house-use shown in and out per KPI version", "Figures before night audit are labelled provisional", "Definitions are versioned; changing a definition never rewrites published reports without a restatement record"]
  security: "property scope; role-based KPI visibility"
  failure_cases: [night_audit_not_run, room_count_changed_mid_period, missing_ooo_reason]
  finance_report_effect: "Denominator for RevPAR/TRevPAR and per-occupied-room cost metrics"
  i18n_a11y: "Numbers and dates localized; charts have data table and text summary; RTL mirrored axes"
  acceptance: "AC-SF32.1.1: Fixture 100 rooms, 5 OOO, 80 occupied incl. 2 comp -> occupancy 84.21% (80/95) with comp-included flag; before night audit the badge reads provisional"
  dependency: "M03 inventory, M06/M08 night audit, M65 definitions, D-413"
- id: M32.F32.1.SF32.1.2
  name: "ADR/RevPAR/TRevPAR"
  phase: 2
  release: R1
  actors: [gm, revenue_manager, owner]
  screens: [SCR-MGMT-room-kpis, SCR-MGMT-gm-flash]
  inputs: [period, net_room_revenue, total_operating_revenue, available_rooms, occupied_rooms_sold]
  states: [provisional, audited, restated]
  api: "GET /v1/properties/{pid}/kpis/room-commerce?from=&to="
  events: [KpiSnapshotPublished]
  data: [fact_revenue_line, fact_room_night, kpi_definition]
  rules: ["ADR = net room revenue / occupied rooms sold; RevPAR = net room revenue / available rooms; TRevPAR = total net operating revenue / available rooms", "Taxes, levies and pass-through excluded; package components allocated per M54 allocation", "Multi-currency converted at policy rate with rate source shown"]
  security: "property scope"
  failure_cases: [package_allocation_missing, fx_rate_missing, revenue_posted_after_audit]
  finance_report_effect: "Ties to M19 room revenue accounts at month-end (reconciliation check)"
  i18n_a11y: "Currency formatting per ISO minor units (OMR 3 decimals); EN/AR"
  acceptance: "AC-SF32.1.2 (AT-G08): Fixture day room revenue OMR 4,000.000, 80 sold, 95 available, total revenue 6,000.000 -> ADR 50.000, RevPAR 42.105, TRevPAR 63.158; month total ties to GL room revenue with zero difference"
  dependency: "M08 revenue postings, M19 mapping"
- id: M32.F32.1.SF32.1.3
  name: "pickup/pace/channel net contribution"
  phase: 3
  release: R1
  actors: [revenue_manager, sales_manager, gm]
  screens: [SCR-MGMT-pickup-pace, SCR-MGMT-channel-contribution]
  inputs: [stay_date_range, as_of_booking_date, segment, channel, fee_sources (ota_commission, gateway_fee, referral_commission, marketing_cost)]
  states: [estimate, reconciled]
  api: "GET /v1/properties/{pid}/reports/pace?stay_from=&stay_to=&as_of="
  events: [ReportRunCompleted]
  data: [reservation_snapshot, fact_revenue_line, fact_cost_line]
  rules: ["Pace compares on-the-books at the same days-before-arrival vs prior year and forecast", "Channel net contribution = net room revenue - channel commission - payment fees - referral commission - attributable marketing; unreconciled fees labelled estimate"]
  security: "revenue/sales roles"
  failure_cases: [channel_fee_not_invoiced_yet, snapshot_gap]
  finance_report_effect: "Feeds M53 decisions and M51 acquisition-cost comparison"
  i18n_a11y: "Accessible line charts with tabular alternative"
  acceptance: "AC-SF32.1.3 (AT-G19): Channel report shows direct vs OTA net contribution with OTA fee labelled estimate until the channel invoice is matched, then reconciled"
  dependency: "M07 commissions, M28 fees, M31 commission, M53"
- id: M32.F32.1.SF32.1.4
  name: "corporate room-block pickup"
  phase: 3
  release: R1
  actors: [sales_manager, revenue_manager, corporate_admin]
  screens: [SCR-MGMT-block-pickup, SCR-CORP-block-status]
  inputs: [block_id, cutoff_date, contracted_nights, picked_up_nights, attrition_terms]
  states: [open, cutoff_passed, released, closed]
  api: "GET /v1/properties/{pid}/reports/block-pickup?block_id="
  events: [ReportRunCompleted]
  data: [room_block (M12), fact_room_night]
  rules: ["Pickup % = picked-up nights / contracted nights; attrition exposure computed per contract version", "Corporate users see only their own blocks"]
  security: "corporate row scope"
  failure_cases: [rooming_list_late, contract_version_changed]
  finance_report_effect: "Attrition revenue estimate flagged for AR"
  i18n_a11y: "Accessible tables, EN/AR"
  acceptance: "AC-SF32.1.4: Block of 10 rooms x 2 nights with 16 nights picked up shows 80% and contractual attrition exposure; corporate A cannot see corporate B's block (404)"
  dependency: "M12 blocks, M10 contracts"
- id: M32.F32.1.SF32.1.5
  name: "cancellation/no-show"
  phase: 2
  release: R1
  actors: [revenue_manager, front_office_manager, gm]
  screens: [SCR-MGMT-cancellations-no-shows]
  inputs: [period, channel, segment, lead_time_bucket]
  states: [provisional, audited]
  api: "GET /v1/properties/{pid}/reports/cancellations?from=&to="
  events: [ReportRunCompleted]
  data: [reservation_event_log, fact_revenue_line]
  rules: ["Rates by channel/segment; retained cancellation/no-show fees vs lost revenue", "Counts dedupe amended reservations"]
  security: "property scope"
  failure_cases: [channel_cancellation_arrives_late, amended_booking_double_count]
  finance_report_effect: "No-show and cancellation fee revenue reconciled to folio postings"
  i18n_a11y: "Accessible tables and charts"
  acceptance: "AC-SF32.1.5: Fixture with 10 cancellations (2 with fees) reports rate and fee revenue equal to folio postings"
  dependency: "M05 lifecycle events"
```

### F32.2 Department P&L

```yaml
- id: M32.F32.2.SF32.2.1
  name: "room revenue/direct cost"
  phase: 4
  release: R1
  actors: [financial_controller, gm, owner, front_office_manager]
  screens: [SCR-MGMT-dept-pnl-rooms]
  inputs: [period, cost_center, gl_accounts, allocation_policy_version]
  states: [estimate, reconciled, certified]
  api: "GET /v1/properties/{pid}/reports/department-pnl?dept=rooms&period="
  events: [ReportRunCompleted]
  data: [gl_journal (M19), fact_cost_line, fact_labor_cost]
  rules: ["Rooms direct cost: housekeeping labor (aggregate), linen/laundry, amenities, commissions, loyalty redemption cost, guest supplies", "Status certified only after period close with complete source coverage"]
  security: "department managers see own department; salary aggregated only"
  failure_cases: [period_open, cost_not_mapped]
  finance_report_effect: "Rooms departmental contribution (layout per D-413)"
  i18n_a11y: "Statement layout accessible, EN/AR labels"
  acceptance: "AC-SF32.2.1 (AT-G08): Rooms P&L revenue equals GL rooms revenue; labor shows the department total with no employee rows"
  dependency: "M19, M27 aggregates, M56"
- id: M32.F32.2.SF32.2.2
  name: "bar/club/catering recipe and labor cost"
  phase: 4
  release: R1
  actors: [fnb_manager, executive_chef, club_host, catering_manager, financial_controller]
  screens: [SCR-MGMT-outlet-pnl, SCR-MGMT-recipe-variance]
  inputs: [outlet_id, event_id, recipe_version, stock_ledger_period, labor_allocation]
  states: [theoretical, actual, reconciled]
  api: "GET /v1/properties/{pid}/reports/outlet-pnl?outlet_id=&period="
  events: [ReportRunCompleted]
  data: [fact_revenue_line, stock_ledger_entry (M50), recipe_version (M14), fact_labor_cost]
  rules: ["COGS = actual stock consumption (issues minus intact returns) including waste; theoretical from POS/BEO recipes; variance shown", "Event margin per BEO uses actual consumption where lot-linked"]
  security: "outlet scope"
  failure_cases: [stocktake_missing, recipe_unmapped_item]
  finance_report_effect: "Outlet and event contribution to GOP"
  i18n_a11y: "Accessible tables; EN/AR"
  acceptance: "AC-SF32.2.2 (AT-G08): A catering event shows revenue, actual ingredient cost from the stock ledger, labor allocation and margin; a missing stocktake labels COGS as estimate"
  dependency: "M13, M14, M15, M16, M50"
- id: M32.F32.2.SF32.2.3
  name: "parking cost"
  phase: 4
  release: R1
  actors: [gm, financial_controller, security_officer]
  screens: [SCR-MGMT-parking-pnl]
  inputs: [facility_id, period, session_revenue, equipment_service_cost, allocated_utilities, staff_cost]
  states: [estimate, reconciled]
  api: "GET /v1/properties/{pid}/reports/department-pnl?dept=parking&period="
  events: [ReportRunCompleted]
  data: [parking_session (M17), fact_cost_line]
  rules: ["Complimentary sessions reported as volume with zero revenue; lost-ticket and manual override counts shown"]
  security: "property scope"
  failure_cases: [gate_offline_sessions_unreconciled]
  finance_report_effect: "Parking contribution"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.2.3: Parking revenue equals posted folio/cash parking revenue; LPR camera maintenance appears as direct cost"
  dependency: "M17, M26"
- id: M32.F32.2.SF32.2.4
  name: "utilities by meter/allocation"
  phase: 4
  release: R1
  actors: [chief_engineer, financial_controller, gm]
  screens: [SCR-MGMT-utility-cost-allocation]
  inputs: [meter_ids, bill_ids, allocation_driver (submeter | area | occupied_rooms | covers | kg_laundry), allocation_policy_version]
  states: [estimated_from_meter, billed, reconciled]
  api: "GET /v1/properties/{pid}/reports/utility-allocation?period="
  events: [AllocationRunCompleted]
  data: [fact_utility_usage, utility_bill (M22-M24), allocation_run]
  rules: ["Submetered usage is direct; remainder allocated by versioned driver; missing meter interval yields estimate label", "Accrual shown when bill not yet received"]
  security: "property scope"
  failure_cases: [meter_gap, bill_meter_variance]
  finance_report_effect: "Utility cost per department and per occupied room"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.2.4 (AT-G08): A missing electricity bill shows an accrued estimate, never certified actual; after bill import the variance is shown"
  dependency: "M22, M23, M24, M25, D-414"
- id: M32.F32.2.SF32.2.5
  name: "maintenance vendor/parts"
  phase: 4
  release: R1
  actors: [chief_engineer, financial_controller]
  screens: [SCR-MGMT-maintenance-cost]
  inputs: [work_order_ids, vendor_invoices, parts_issues, asset_ids]
  states: [committed, invoiced, paid]
  api: "GET /v1/properties/{pid}/reports/maintenance-cost?period="
  events: [ReportRunCompleted]
  data: [work_order (M26), supplier_invoice (M20), stock_ledger_entry]
  rules: ["Cost by asset, room and department; capex vs opex split per M66 policy"]
  security: "engineering/finance roles"
  failure_cases: [invoice_without_work_order]
  finance_report_effect: "Maintenance in department P&L and asset cost"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.2.5: A room repair cost drills to work order, PO, receipt and invoice"
  dependency: "M26, M21, M20"
- id: M32.F32.2.SF32.2.6
  name: "payment/channel fees"
  phase: 4
  release: R1
  actors: [financial_controller, revenue_manager]
  screens: [SCR-MGMT-fees]
  inputs: [psp_settlement_fees, channel_commission_invoices, referral_commissions, bill_provider_fees]
  states: [estimated, invoiced, reconciled]
  api: "GET /v1/properties/{pid}/reports/fees?period="
  events: [ReportRunCompleted]
  data: [psp_settlement_line (M28), channel_commission (M07), commission_ledger_entry (M31)]
  rules: ["Fees attributed to source booking/outlet where available, otherwise allocated with label"]
  security: "finance roles"
  failure_cases: [settlement_file_missing]
  finance_report_effect: "Distribution-cost line in profit bridge"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.2.6 (AT-G19): A booking's channel fee drills to the channel commission record and its PSP fee to the settlement line"
  dependency: "M07, M28, M31"
```

### F32.3 Property result

```yaml
- id: M32.F32.3.SF32.3.1
  name: "shared expense allocation and version"
  phase: 4
  release: R1
  actors: [financial_controller, gm, owner]
  screens: [SCR-FIN-allocation-policies, SCR-FIN-allocation-run]
  inputs: [cost_pool, drivers, weights, effective_period, policy_version]
  states: [draft, approved, run, reversed]
  api: "POST /v1/properties/{pid}/allocation-policies (maker-checker); POST /v1/properties/{pid}/allocation-runs (Idempotency-Key)"
  events: [AllocationPolicyApproved, AllocationRunCompleted]
  data: [allocation_policy_version, allocation_run]
  rules: ["Allocations sum to 100% of the pool (no leakage); a run is reproducible from its version; a rerun reverses the previous run first"]
  security: "maker-checker"
  failure_cases: [driver_data_missing, overlapping_versions]
  finance_report_effect: "Management allocations (memo or GL per D-414)"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.3.1: Energy pool OMR 1,000.000 allocated by driver sums exactly to 1,000.000 across departments; a rerun produces identical figures"
  dependency: "D-414"
- id: M32.F32.3.SF32.3.2
  name: "operating result vs net income definition"
  phase: 4
  release: R1
  actors: [owner, financial_controller, gm]
  screens: [SCR-MGMT-profit-bridge]
  inputs: [period, definition_version]
  states: [estimate, reconciled, certified]
  api: "GET /v1/properties/{pid}/reports/profit-bridge?period="
  events: [ReportRunCompleted]
  data: [gl_journal, kpi_definition]
  rules: ["Bridge: total revenue -> departmental profit -> undistributed expenses -> GOP -> management fees/fixed charges (M66) -> EBITDA -> depreciation/interest/tax -> net income, each step labelled with source", "Lines lacking an authoritative source (e.g. debt interest) show 'not available', never zero"]
  security: "owner/controller scope"
  failure_cases: [missing_fixed_charges_source]
  finance_report_effect: "Primary owner profitability answer"
  i18n_a11y: "Waterfall chart with table alternative"
  acceptance: "AC-SF32.3.2 (AT-G08): Bridge totals tie to the trial balance; a missing depreciation source shows 'not available' with owner"
  dependency: "M19, M66, D-416"
- id: M32.F32.3.SF32.3.3
  name: "actual/estimate/budget/prior year"
  phase: 4
  release: R1
  actors: [financial_controller, gm, owner]
  screens: [SCR-MGMT-budget-vs-actual, SCR-FIN-budget-import]
  inputs: [budget_version, forecast_version, prior_year_period]
  states: [draft, approved, locked]
  api: "POST /v1/properties/{pid}/budgets (import); GET /v1/properties/{pid}/reports/variance?period="
  events: [BudgetVersionApproved]
  data: [budget_version, budget_line, forecast_version (M53)]
  rules: ["Columns never mix estimate and actual silently; each cell carries status"]
  security: "budget edit restricted to finance"
  failure_cases: [budget_account_unmapped]
  finance_report_effect: "Variance analysis"
  i18n_a11y: "Status icon plus text; accessible"
  acceptance: "AC-SF32.3.3: Variance report shows actual (reconciled), estimate and budget in separate columns with status text"
  dependency: "M53 forecast"
- id: M32.F32.3.SF32.3.4
  name: "source coverage/close status"
  phase: 4
  release: R1
  actors: [financial_controller, gm]
  screens: [SCR-MGMT-data-coverage]
  inputs: [period, expected_sources (bank, psp, utilities, payroll, ap, pos, channel)]
  states: [missing, partial, complete, closed]
  api: "GET /v1/properties/{pid}/reports/coverage?period="
  events: [DataFreshnessBreached]
  data: [data_freshness_status, period_close (M19)]
  rules: ["Every KPI shows last update and source; a missing source yields an 'incomplete estimate' badge"]
  security: "property scope"
  failure_cases: [feed_stale]
  finance_report_effect: "Gates the certified label"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.3.4 (AT-G08): With payroll not imported, labor lines and GOP show incomplete estimate"
  dependency: "M65"
- id: M32.F32.3.SF32.3.5
  name: "drill-through and period restatement"
  phase: 4
  release: R1
  actors: [gm, owner, financial_controller, auditor]
  screens: [SCR-MGMT-drill-through, SCR-FIN-restatements]
  inputs: [kpi_cell_ref, restatement_reason]
  states: [published, restated]
  api: "GET /v1/properties/{pid}/reports/{report_id}/cells/{cell}/lineage"
  events: [PeriodRestated]
  data: [report_version, restatement_record, lineage (M65)]
  rules: ["Drill path KPI -> journal -> source document/event, honouring field permissions (salary aggregated)", "Restated periods keep the prior version accessible with reason"]
  security: "field-level filtering"
  failure_cases: [lineage_broken]
  finance_report_effect: "Audit trail for all figures"
  i18n_a11y: "Keyboard-navigable drill path with breadcrumbs"
  acceptance: "AC-SF32.3.5 (AT-G19): GM drills occupancy -> profit -> channel fee -> labor -> laundry -> food waste -> utilities to source events; GM cannot open an individual payslip"
  dependency: "M65 lineage"
```

### F32.4 Report suite

```yaml
- id: M32.F32.4.SF32.4.1
  name: "GM flash"
  phase: 2
  release: R1
  actors: [gm, duty_manager, owner]
  screens: [SCR-MGMT-gm-flash]
  inputs: [business_date]
  states: [provisional, audited]
  api: "GET /v1/properties/{pid}/reports/gm-flash?date="
  events: [GmFlashPublished]
  data: [kpi_snapshot, data_freshness_status]
  rules: ["3-7 headline items with source/freshness plus exceptions needing action (owner, due time)"]
  security: "gm/owner scope"
  failure_cases: [night_audit_late]
  finance_report_effect: "Daily summary of revenue and cash"
  i18n_a11y: "Mobile-first, EN/AR, screen-reader summary"
  acceptance: "AC-SF32.4.1: Flash for a business date shows occupancy, ADR, RevPAR, revenue by department, cash and open incidents, labelled provisional before audit"
  dependency: "M06/M08 night audit"
- id: M32.F32.4.SF32.4.2
  name: "finance and cash"
  phase: 4
  release: R1
  actors: [financial_controller, finance_clerk]
  screens: [SCR-FIN-cash-position, SCR-FIN-ar-ap-summary]
  inputs: [period]
  states: [planned, accrued, invoiced, approved, paid, settled]
  api: "GET /v1/properties/{pid}/reports/cash?period="
  events: [ReportRunCompleted]
  data: [bank_statement_line, payable, receivable]
  rules: ["Planned/accrued/invoiced/approved/paid/settled shown separately (Section D)"]
  security: "finance roles"
  failure_cases: [bank_import_missing]
  finance_report_effect: "Cash reporting"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.4.2: Report shows each Section D status bucket; totals tie to AP/AR sub-ledgers"
  dependency: "M20"
- id: M32.F32.4.SF32.4.3
  name: "occupancy/revenue"
  phase: 2
  release: R1
  actors: [revenue_manager, gm]
  screens: [SCR-MGMT-revenue-report]
  inputs: [period, dimensions (segment, channel, rate_plan, room_type)]
  states: [provisional, audited]
  api: "GET /v1/properties/{pid}/reports/revenue?from=&to="
  events: [ReportRunCompleted]
  data: [fact_revenue_line, fact_room_night]
  rules: ["Dimension totals always sum to the property total (unmapped bucket shown)"]
  security: "property scope"
  failure_cases: [unmapped_segment]
  finance_report_effect: "Revenue analysis"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.4.3: Revenue by segment sums to total revenue including an explicit unmapped bucket"
  dependency: "F32.1"
- id: M32.F32.4.SF32.4.4
  name: "utility/energy/water/gas"
  phase: 4
  release: R1
  actors: [chief_engineer, gm]
  screens: [SCR-MGMT-utilities-report]
  inputs: [period, meters]
  states: [estimated, verified]
  api: "GET /v1/properties/{pid}/reports/utilities?period="
  events: [ReportRunCompleted]
  data: [fact_utility_usage, utility_bill]
  rules: ["Intensity per occupied room and per cover; pipeline gas and cylinders reported separately"]
  security: "property scope"
  failure_cases: [meter_gap]
  finance_report_effect: "Utility cost KPIs"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.4.4: kWh per occupied room is computed with a missed interval flagged as estimate"
  dependency: "M22-M25, M67"
- id: M32.F32.4.SF32.4.5
  name: "purchasing/payroll/vendor"
  phase: 4
  release: R1
  actors: [procurement_approver, financial_controller, hr_officer]
  screens: [SCR-MGMT-purchasing-report, SCR-MGMT-labor-cost]
  inputs: [period, department]
  states: [estimate, reconciled]
  api: "GET /v1/properties/{pid}/reports/purchasing?period=; GET /v1/properties/{pid}/reports/labor-cost?period="
  events: [ReportRunCompleted]
  data: [purchase_order, fact_labor_cost, vendor_scorecard]
  rules: ["Labor cost aggregated by department/period; individual pay only for payroll roles (F27.4)", "Groups below 3 employees suppressed for non-payroll roles", "Vendor OTIF and spend"]
  security: "salary confidentiality enforced by row/field policy"
  failure_cases: [small_group_reidentification]
  finance_report_effect: "Labor and purchasing analysis"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.4.5 (AT-G05): A department with 2 employees shows suppressed labor detail to non-payroll roles"
  dependency: "M27, M21, M49"
- id: M32.F32.4.SF32.4.6
  name: "corporate/events/outlets"
  phase: 4
  release: R1
  actors: [sales_manager, catering_manager, fnb_manager, gm]
  screens: [SCR-MGMT-corporate-report, SCR-MGMT-event-report]
  inputs: [corporate_id, event_id, outlet_id]
  states: [proposed, actual, reconciled]
  api: "GET /v1/properties/{pid}/reports/corporate?period="
  events: [ReportRunCompleted]
  data: [corporate_account (M10), event (M12), fact_revenue_line]
  rules: ["Contracted vs actual per event; corporate AR aging"]
  security: "corporate row scope for corporate users"
  failure_cases: [event_not_closed]
  finance_report_effect: "Event and corporate margin"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.4.6: The 80-person event report shows rooms, F&B, club and parking revenue and cost with contracted vs actual"
  dependency: "M10, M12, M16"
- id: M32.F32.4.SF32.4.7
  name: "audit/data quality"
  phase: 4
  release: R1
  actors: [auditor, financial_controller, it_admin]
  screens: [SCR-MGMT-data-quality]
  inputs: [check_set]
  states: [pass, warn, fail]
  api: "GET /v1/properties/{pid}/reports/data-quality"
  events: [DataQualityCheckFailed]
  data: [data_quality_result (M65)]
  rules: ["Duplicate, orphan, unmapped, stale-feed and ledger roll-forward checks run nightly"]
  security: "read-only"
  failure_cases: [check_timeout]
  finance_report_effect: "Close readiness"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF32.4.7 (AT-G20): Every simulated G20 exception appears in the data-quality or exception report"
  dependency: "M65"
- id: M32.F32.4.SF32.4.8
  name: "authorized custom pivots and scheduled PDF/CSV/XLSX exports"
  phase: 5
  release: R1
  actors: [gm, financial_controller, revenue_manager, owner]
  screens: [SCR-MGMT-pivot-builder, SCR-MGMT-report-schedules]
  inputs: [dataset, dimensions, measures, filters, schedule_cron, recipients, format]
  states: [draft, saved, scheduled, paused]
  api: "POST /v1/properties/{pid}/report-definitions; POST /v1/properties/{pid}/report-schedules"
  events: [ScheduledReportDelivered, ScheduledReportFailed]
  data: [report_definition, report_schedule, report_run]
  rules: ["Pivots only over governed datasets; permissions applied at execution time for each recipient", "Exports watermarked with user/time; recipients must be authorized users"]
  security: "row/field security on export; no salary or ID data in pivot datasets"
  failure_cases: [recipient_lost_permission, export_size_limit]
  finance_report_effect: "Report distribution audit"
  i18n_a11y: "Tagged PDF; XLSX header rows; EN/AR"
  acceptance: "AC-SF32.4.8: A scheduled report to a user whose role was revoked is not delivered and logs failure; exported CSV excludes restricted fields"
  dependency: "M02, M65, D-415"
```

### M32 key invariants
1. Every KPI cell carries definition version, source, freshness and status; missing sources never render as actual.
2. Month-end revenue and cost figures tie to the M19 trial balance.
3. Individual salary and ID data never appear in management datasets; small groups are suppressed.
4. Allocations are versioned, sum to 100% and are reproducible.

### M32 module acceptance (Section G)
| Test | Scenario | Pass condition |
|---|---|---|
| AT-G08.1 | Night audit and month-end | Occupancy, ADR, RevPAR, TRevPAR, department contribution and operating result shown and tied to GL |
| AT-G08.2 | Shared energy/labor allocation | Allocation versioned and drills to evidence |
| AT-G08.3 | Missing cost source | Incomplete estimate, never certified actual |
| AT-G19.x | GM drill-through | Occupancy → profit → channel fee, labor, laundry, food waste, utilities → source events |
| AT-G05.x | Salary confidentiality | Department aggregate only for non-payroll roles |

### M32 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-413 | KPI definitions and statement layout (USALI-style alignment; comp/house-use treatment) | Financial Controller + GM | USALI-style departmental layout; comp included in occupancy with toggle |
| D-414 | Allocation drivers per cost pool; whether allocations post to GL or remain management memo | Financial Controller | Management memo allocations; GL untouched |
| D-415 | BI tooling: in-app governed reports only vs embedded BI engine | Solution Architect | In-app governed reports + CSV/XLSX; embedded engine evaluated in Phase 4 |
| D-416 | Net-income line sources (depreciation, interest, owner costs) | Owner + Financial Controller | Show 'not available' until an M66/M19 source exists |

---

## 4. M33 — Integration developer platform

| Header | Value |
|---|---|
| Purpose | Make every MetriStay capability consumable by approved partners and internal apps through versioned OpenAPI 3.1 contracts, domain events and signed webhooks, with scoped credentials, sandbox tenants, SDK/docs, deprecation policy, event replay/monitoring, and a vendor certification kit with audit. It is the contract layer; each external partner adapter (docs/05) still sits behind its own port. |
| Features (created from Section C row) | F33.1 API contracts and access; F33.2 Events and webhooks; F33.3 Sandbox, SDK and certification |
| Phases / release | Phase 2 foundation (contract registry, client credentials, outbox event catalogue); Phases 5–6 mature (webhooks, replay, sandbox, certification kit) — R1; Phase 7 public self-service developer portal — Later. |
| Bounded context | `integration-platform` |
| System-of-record entities | `api_contract_version`, `api_client`, `api_scope_grant`, `api_rate_limit_policy`, `event_schema_version`, `webhook_subscription`, `webhook_delivery`, `dead_letter_item`, `event_replay_request`, `sandbox_tenant`, `sdk_release`, `certification_run`, `partner_access_review`, `deprecation_notice` |
| Referenced | M01 tenant/property, M02 identity (Keycloak clients), outbox/inbox tables of every context, M64 device/connector registry |
| Dependencies | M01, M02, M64; consumed by every integration-heavy module (M07, M17, M28, M29, M38, M41, M45, M48) |
| Publishes | `ApiClientRegistered`, `ApiScopeGranted`, `ApiVersionDeprecated`, `WebhookDeliveryFailed`, `WebhookSubscriptionSuspended`, `EventReplayCompleted`, `CertificationRunCompleted` |

### F33.1 API contracts and access

```yaml
- id: M33.F33.1.SF33.1.1
  name: "OpenAPI contract registry and versioning"
  phase: 2
  release: R1
  actors: [integration_admin, it_admin, auditor]
  screens: [SCR-DEV-api-catalogue, SCR-DEV-contract-diff]
  inputs: [openapi_document, semver, owning_context, compatibility_class (additive | breaking)]
  states: [draft, published, deprecated, retired]
  api: "GET /v1/platform/contracts; POST /v1/platform/contracts (CI pipeline identity only)"
  events: [ApiContractPublished]
  data: [api_contract_version]
  rules: ["OpenAPI 3.1 is the source of truth; CI blocks merges whose implementation diverges from the contract", "Breaking change requires a new major path version (/v2) and a deprecation notice for the old one", "Every mutating money/stock/inventory/external-order operation declares Idempotency-Key required"]
  security: "publish only from CI with signed commits; catalogue readable by authenticated partners per scope"
  failure_cases: [breaking_change_without_major, contract_drift, missing_idempotency_declaration]
  finance_report_effect: "None directly; contract ids referenced in audit of financial API calls"
  i18n_a11y: "Error payloads carry machine code + localizable message key; docs site accessible"
  acceptance: "AC-SF33.1.1: CI rejects a PR that removes a response field without major version bump; a POST on /payments without Idempotency-Key declaration fails lint"
  dependency: "ADR API style (docs/03)"
- id: M33.F33.1.SF33.1.2
  name: "API client registration and scoped credentials"
  phase: 2
  release: R1
  actors: [integration_admin, tenant_admin, vendor_admin]
  screens: [SCR-DEV-clients, SCR-DEV-client-scopes]
  inputs: [client_name, owner_org, grant_type (client_credentials | authorization_code_pkce | mtls), scopes, property_ids, ip_allowlist, expiry]
  states: [requested, approved, active, suspended, revoked]
  api: "POST /v1/platform/clients (maker-checker); POST /v1/platform/clients/{id}/rotate-secret"
  events: [ApiClientRegistered, ApiScopeGranted, ApiClientRevoked]
  data: [api_client, api_scope_grant]
  rules: ["Least privilege: scopes are resource:action:property (e.g. reservations:read:{pid}); no wildcard tenant scope for partners", "Secrets shown once, stored hashed/vaulted; rotation with overlap window; mandatory expiry", "Money-moving scopes require dual approval and mTLS or signed requests"]
  security: "Keycloak clients; audit of every grant; OWASP API object-level authorization tests on each scope"
  failure_cases: [scope_escalation_attempt, leaked_secret, expired_client_in_use]
  finance_report_effect: "None"
  i18n_a11y: "Admin screens accessible"
  acceptance: "AC-SF33.1.2: A client scoped to property P1 receives 404 on P2 objects; a revoked client is rejected within 60 s; money scope without second approver cannot be granted"
  dependency: "M02 Keycloak"
- id: M33.F33.1.SF33.1.3
  name: "versioning and deprecation policy"
  phase: 5
  release: R1
  actors: [integration_admin, vendor_admin]
  screens: [SCR-DEV-deprecations]
  inputs: [contract_version, sunset_date, migration_guide_ref]
  states: [announced, warning_headers, sunset]
  api: "Responses include Deprecation and Sunset headers; GET /v1/platform/deprecations"
  events: [ApiVersionDeprecated]
  data: [deprecation_notice, api_client usage stats]
  rules: ["Minimum support window per D-418 before sunset; clients still calling deprecated endpoints are notified", "No silent removal"]
  security: "notifications to registered client owners only"
  failure_cases: [active_client_on_sunset_day]
  finance_report_effect: "None"
  i18n_a11y: "Notices EN/AR"
  acceptance: "AC-SF33.1.3: Calls to a deprecated endpoint return Deprecation/Sunset headers; usage report lists clients still calling it 30 days before sunset"
  dependency: "D-418"
- id: M33.F33.1.SF33.1.4
  name: "rate limits and quotas"
  phase: 5
  release: R1
  actors: [integration_admin, it_admin]
  screens: [SCR-DEV-rate-limits]
  inputs: [client_id, limits_per_minute, burst, daily_quota]
  states: [within_limit, throttled, blocked]
  api: "429 with Retry-After; GET /v1/platform/clients/{id}/usage"
  events: [ApiClientThrottled]
  data: [api_rate_limit_policy, usage_counter]
  rules: ["Per-client and per-property limits; front-desk/staff apps have reserved capacity so partner bursts cannot starve operations"]
  security: "limits enforced at gateway"
  failure_cases: [distributed_counter_drift, legitimate_burst_during_sync]
  finance_report_effect: "None"
  i18n_a11y: "Error messages localizable"
  acceptance: "AC-SF33.1.4: A partner exceeding its limit gets 429 while front-desk API latency stays within SLO in load test"
  dependency: "M64 gateway"
```

### F33.2 Events and webhooks

```yaml
- id: M33.F33.2.SF33.2.1
  name: "event catalogue and schema registry"
  phase: 2
  release: R1
  actors: [integration_admin, it_admin]
  screens: [SCR-DEV-event-catalogue]
  inputs: [event_type, version, json_schema, owning_context, pii_classification]
  states: [draft, published, deprecated]
  api: "GET /v1/platform/events/types"
  events: [EventSchemaPublished]
  data: [event_schema_version]
  rules: ["Envelope per README 3.5; payload schema versioned; PII fields classified and minimised for external subscribers", "Consumers dedupe on event_id (inbox)"]
  security: "PII-classified fields only in scopes allowing them"
  failure_cases: [schema_incompatible_publish, unclassified_pii_field]
  finance_report_effect: "Event lineage used by M65/M32 drill-through"
  i18n_a11y: "Docs accessible"
  acceptance: "AC-SF33.2.1: Publishing an event that fails its schema is rejected by the outbox relay test; catalogue shows PII classification per field"
  dependency: "outbox/inbox ADR"
- id: M33.F33.2.SF33.2.2
  name: "webhook subscriptions and signed delivery"
  phase: 5
  release: R1
  actors: [integration_admin, vendor_admin, webhook_worker]
  screens: [SCR-DEV-webhooks]
  inputs: [client_id, event_types, target_url, secret, property_ids]
  states: [active, failing, suspended, deleted]
  api: "POST /v1/platform/webhook-subscriptions; deliveries signed HMAC-SHA256 with timestamp header"
  events: [WebhookDeliverySucceeded, WebhookDeliveryFailed, WebhookSubscriptionSuspended]
  data: [webhook_subscription, webhook_delivery]
  rules: ["At-least-once delivery with event_id for receiver dedupe; timestamp tolerance 5 min against replay", "HTTPS only; target URL validated against SSRF (no private ranges)", "Subscription limited to client's scopes and properties"]
  security: "signing secret vaulted; rotation; SSRF protection"
  failure_cases: [receiver_down, slow_receiver, ssrf_target, signature_rotation_overlap]
  finance_report_effect: "None"
  i18n_a11y: "Admin UI accessible"
  acceptance: "AC-SF33.2.2: A delivery to a 10.0.0.0/8 URL is refused; receiver verifying signature accepts valid and rejects replayed (old timestamp) delivery"
  dependency: "SF33.1.2"
- id: M33.F33.2.SF33.2.3
  name: "retry, dead letter and replay"
  phase: 5
  release: R1
  actors: [integration_admin, it_admin, webhook_worker]
  screens: [SCR-DEV-dead-letters, SCR-ADMIN-integration-health]
  inputs: [delivery_id, replay_from, replay_to, event_types]
  states: [retrying, dead_lettered, replay_requested, replayed]
  api: "POST /v1/platform/event-replays (Idempotency-Key); POST /v1/platform/dead-letters/{id}/redeliver"
  events: [EventReplayCompleted, DeadLetterCreated]
  data: [dead_letter_item, event_replay_request]
  rules: ["Exponential backoff for 24 h then dead-letter; replay re-sends original event_id so receivers can dedupe", "Replay limited to client's scope and retention window"]
  security: "replay requires integration_admin; audit"
  failure_cases: [replay_storm, retention_expired]
  finance_report_effect: "Dead letters touching money flows surface in M32 SF32.4.7 exception report"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF33.2.3 (AT-G20): Duplicate provider webhook and a replay of 100 events produce zero duplicate side effects in the consuming inbox"
  dependency: "inbox dedupe in each context"
- id: M33.F33.2.SF33.2.4
  name: "integration monitoring"
  phase: 5
  release: R1
  actors: [it_admin, integration_admin, duty_manager]
  screens: [SCR-ADMIN-integration-health]
  inputs: [connector_id, metrics (latency, error_rate, lag, dead_letters)]
  states: [healthy, degraded, down]
  api: "GET /v1/platform/health/connectors"
  events: [ConnectorHealthChanged]
  data: [connector_health_snapshot]
  rules: ["Each connector shows honesty label (README 3.6), last success, backlog and manual fallback link"]
  security: "admin roles"
  failure_cases: [monitoring_blind_spot]
  finance_report_effect: "Feeds data freshness (SF32.3.4)"
  i18n_a11y: "Status text plus colour; accessible"
  acceptance: "AC-SF33.2.4: Stopping the PSP sandbox marks the connector down within 2 minutes and shows the manual path"
  dependency: "M64 monitoring"
```

### F33.3 Sandbox, SDK and certification

```yaml
- id: M33.F33.3.SF33.3.1
  name: "sandbox tenant with synthetic data"
  phase: 5
  release: R1
  actors: [integration_admin, vendor_admin]
  screens: [SCR-DEV-sandbox]
  inputs: [fixture_set (single_hotel | five_market), partner_client_id]
  states: [provisioning, ready, reset, expired]
  api: "POST /v1/platform/sandboxes; POST /v1/platform/sandboxes/{id}/reset"
  events: [SandboxProvisioned]
  data: [sandbox_tenant]
  rules: ["Synthetic data only; no production data copy", "Mock adapters for PSP/bill provider/government mark every response as simulated"]
  security: "isolated tenant; separate credentials"
  failure_cases: [production_data_leak_attempt]
  finance_report_effect: "Sandbox figures excluded from all reports"
  i18n_a11y: "Fixtures include Arabic names/addresses for RTL testing"
  acceptance: "AC-SF33.3.1: A sandbox client cannot reach production endpoints; fixture five-market dataset loads in < 10 minutes"
  dependency: "docs/09 fixtures"
- id: M33.F33.3.SF33.3.2
  name: "SDK and developer documentation"
  phase: 5
  release: R1
  actors: [integration_admin, vendor_admin]
  screens: [SCR-DEV-docs]
  inputs: [contract_versions]
  states: [generated, published]
  api: "Static docs site generated from contracts; SDK packages (TypeScript first)"
  events: [SdkReleased]
  data: [sdk_release]
  rules: ["SDKs generated from OpenAPI; examples use sandbox only", "Docs state capability flags and unavailable operations honestly"]
  security: "docs behind partner login until public portal (SF33.3.5)"
  failure_cases: [sdk_contract_mismatch]
  finance_report_effect: "None"
  i18n_a11y: "Docs WCAG 2.2 AA; code samples copyable"
  acceptance: "AC-SF33.3.2: Generated SDK compiles and passes contract tests against sandbox"
  dependency: "SF33.1.1"
- id: M33.F33.3.SF33.3.3
  name: "vendor certification kit"
  phase: 6
  release: R1
  actors: [integration_admin, vendor_admin, compliance_officer]
  screens: [SCR-DEV-certification]
  inputs: [partner_id, test_suite_version, evidence]
  states: [not_started, in_progress, passed, failed, expired]
  api: "POST /v1/platform/certification-runs"
  events: [CertificationRunCompleted]
  data: [certification_run]
  rules: ["Suite covers auth, idempotency, signature, timeout, duplicate callback, outage, reconciliation", "Passing the MetriStay kit is not a claim of third-party or government certification"]
  security: "evidence immutable"
  failure_cases: [flaky_partner_sandbox]
  finance_report_effect: "Status drives connector honesty label (sandbox-tested/certified)"
  i18n_a11y: "Reports accessible"
  acceptance: "AC-SF33.3.3: A partner adapter failing the duplicate-callback case cannot be labelled sandbox-tested"
  dependency: "docs/05 per-integration contracts"
- id: M33.F33.3.SF33.3.4
  name: "partner access audit and review"
  phase: 6
  release: R1
  actors: [compliance_officer, dpo, auditor, integration_admin]
  screens: [SCR-DEV-access-reviews]
  inputs: [review_period, clients, scopes, data_categories]
  states: [scheduled, in_review, attested, remediation]
  api: "GET /v1/platform/access-reviews; POST .../{id}/attest"
  events: [PartnerAccessReviewed]
  data: [partner_access_review, audit_log]
  rules: ["Quarterly review of all partner scopes; unused scopes > 90 days revoked", "Audit log of every partner call touching PII or money retained per policy"]
  security: "attestation by compliance_officer"
  failure_cases: [review_overdue]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF33.3.4: A scope unused for 91 days is flagged and revoked after attestation"
  dependency: "M02 audit"
- id: M33.F33.3.SF33.3.5
  name: "public self-service developer portal"
  phase: 7
  release: Later
  actors: [integration_admin, vendor_admin]
  screens: [SCR-DEV-public-portal]
  inputs: [developer_signup, terms_acceptance]
  states: [applied, approved, active]
  api: "Public portal (Later)"
  events: [DeveloperSignedUp]
  data: [developer_account]
  rules: ["Self-service signup grants sandbox only; production access still maker-checker"]
  security: "bot protection without CAPTCHA-only barrier"
  failure_cases: [abuse_signups]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF33.3.5: Self-signed developer can reach sandbox but not production"
  dependency: "D-417"
```

### M33 key invariants / acceptance / decisions
- Invariants: contract-first; object-level authorization per property; at-least-once events with consumer dedupe; SSRF-safe signed webhooks; certification label never implies third-party/government certification.
- Module acceptance: AT-G20.x (duplicate webhook, network outage — no duplicate side effects); AT-G06.x/AT-G12.x connectors show honest status and manual path; AT-G09.x five-market sandbox fixtures.

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-417 | Whether/when to open a public developer portal and partner commercial terms | Product Owner | Invite-only partners through R1 |
| D-418 | API deprecation support window | Solution Architect | 12 months minimum; 6 months for sandbox-only APIs |
| D-419 | Webhook signing standard (HMAC vs asymmetric JWS) | Security Lead | HMAC-SHA256 with timestamp; JWS evaluated Phase 5 |

---

## 5. M34 — UC/wake-up (Later)

| Header | Value |
|---|---|
| Purpose | Integrate hotel telephony (PBX/UC): extension ↔ room mapping, guest name/status and class-of-service on check-in/out/room move, DND and message waiting, call accounting charge posting, wake-up call execution with acknowledgment and human escalation, and emergency-calling local survivability. |
| Features (created from Section C) | F34.1 PBX integration; F34.2 Wake-up and emergency calling |
| Phases / release | Phase 7, **Later**. Specified at planning depth; R1 uses manual wake-up task lists (M63) and no PBX automation. |
| Bounded context | `uc` |
| System-of-record entities | `pbx_system`, `pbx_extension`, `extension_room_map`, `wake_up_call`, `wake_up_attempt`, `call_charge_record` (raw CDR reference), `uc_sync_log` |
| Dependencies | M05 stay events, M06 DND, M08 folio posting, M64 device registry, M42 emergency, M33 adapter framework; vendor PBX interface (D-420) |
| Publishes | `PbxGuestStatusSynced`, `WakeUpScheduled`, `WakeUpAcknowledged`, `WakeUpEscalated`, `CallChargePosted` |

```yaml
- id: M34.F34.1.SF34.1.1
  name: "PBX and extension registry with adapter"
  phase: 7
  release: Later
  actors: [it_admin, integration_admin]
  screens: [SCR-ADMIN-pbx-extensions]
  inputs: [pbx_vendor, protocol, extension_number, room_id, device_type]
  states: [unmapped, mapped, faulty, retired]
  api: "PUT /v1/properties/{pid}/uc/extensions/{ext}"
  events: [ExtensionMapped]
  data: [pbx_system, pbx_extension, extension_room_map]
  rules: ["One active room per guest extension; capability flags per PBX adapter; unsupported operations unavailable"]
  security: "adapter credentials vaulted; network segmentation (M64)"
  failure_cases: [pbx_unreachable, mapping_conflict]
  finance_report_effect: "None"
  i18n_a11y: "Admin UI accessible"
  acceptance: "AC-SF34.1.1: Mapping an extension already mapped to another room is rejected"
  dependency: "D-420 PBX vendor/protocol"
- id: M34.F34.1.SF34.1.2
  name: "check-in/out/room-move guest status and class of service"
  phase: 7
  release: Later
  actors: [front_desk_agent, uc_worker]
  screens: [SCR-FD-room-move, SCR-ADMIN-uc-sync-log]
  inputs: [stay_id, room_id, guest_display_name_consented, language, class_of_service]
  states: [pending_sync, synced, failed]
  api: "Consumer of GuestCheckedIn/GuestCheckedOut/RoomMoved; adapter command SetGuestStatus"
  events: [PbxGuestStatusSynced, PbxSyncFailed]
  data: [uc_sync_log]
  rules: ["Checkout bars outgoing chargeable calls; room move transfers settings and messages", "Guest name shown on staff phones only if consented/policy allows"]
  security: "minimum guest data to PBX"
  failure_cases: [sync_failed_retry, out_of_order_events]
  finance_report_effect: "Prevents unbillable calls after checkout"
  i18n_a11y: "Language code passed for voice prompts where supported"
  acceptance: "AC-SF34.1.2: Room move sends one status change; failed sync retries and appears in exception queue"
  dependency: "M05 events"
- id: M34.F34.1.SF34.1.3
  name: "DND and message waiting"
  phase: 7
  release: Later
  actors: [guest, front_desk_agent, uc_worker]
  screens: [SCR-FD-guest-messages]
  inputs: [room_id, dnd_flag, message_waiting]
  states: [off, on]
  api: "Adapter commands SetDnd, SetMessageWaiting"
  events: [DndChanged]
  data: [uc_sync_log]
  rules: ["DND in PMS and PBX kept consistent; emergency calls bypass DND"]
  security: "staff scope"
  failure_cases: [state_divergence]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF34.1.3: DND set on phone updates M06 housekeeping DND once"
  dependency: "M06"
- id: M34.F34.1.SF34.1.4
  name: "call accounting charge posting"
  phase: 7
  release: Later
  actors: [night_auditor, uc_worker]
  screens: [SCR-FIN-call-charges]
  inputs: [cdr_id, extension, duration, destination_class, tariff_version]
  states: [received, rated, posted, disputed]
  api: "Adapter CDR feed; POST folio charge via M08 (Idempotency-Key = cdr_id)"
  events: [CallChargePosted]
  data: [call_charge_record, folio_line]
  rules: ["Each CDR posts at most once; calls after checkout to routing folio or house"]
  security: "destination numbers masked in guest views"
  failure_cases: [duplicate_cdr, unknown_extension]
  finance_report_effect: "Telephone revenue and cost"
  i18n_a11y: "Folio line localized"
  acceptance: "AC-SF34.1.4: Duplicate CDR produces one folio line"
  dependency: "M08"
- id: M34.F34.2.SF34.2.1
  name: "wake-up call scheduling"
  phase: 7
  release: Later
  actors: [guest, front_desk_agent]
  screens: [SCR-GUEST-wake-up, SCR-FD-wake-up-list]
  inputs: [room_id, local_time, repeat, language]
  states: [scheduled, cancelled]
  api: "POST /v1/properties/{pid}/uc/wake-up-calls"
  events: [WakeUpScheduled]
  data: [wake_up_call]
  rules: ["Stored in property time zone; cancelled on checkout/room move re-targeted"]
  security: "guest own room only"
  failure_cases: [dst_transition]
  finance_report_effect: "None"
  i18n_a11y: "Accessible time picker; EN/AR"
  acceptance: "AC-SF34.2.1: Room move re-targets the scheduled wake-up to the new extension"
  dependency: "SF34.1.2"
- id: M34.F34.2.SF34.2.2
  name: "wake-up execution, acknowledgment and human escalation"
  phase: 7
  release: Later
  actors: [uc_worker, front_desk_agent, duty_manager]
  screens: [SCR-FD-wake-up-exceptions]
  inputs: [wake_up_call_id, attempt_results]
  states: [due, attempting, acknowledged, unanswered_escalated, completed]
  api: "Adapter command PlaceWakeUpCall; callback WakeUpResult"
  events: [WakeUpAcknowledged, WakeUpEscalated]
  data: [wake_up_attempt]
  rules: ["N retries then human task (room visit per SOP); PBX outage switches to manual list immediately"]
  security: "audit"
  failure_cases: [pbx_outage, no_answer]
  finance_report_effect: "None"
  i18n_a11y: "Voice prompt language per guest"
  acceptance: "AC-SF34.2.2: Unanswered after 3 attempts creates an escalation task within 1 minute"
  dependency: "M63"
- id: M34.F34.2.SF34.2.3
  name: "emergency calling and local survivability"
  phase: 7
  release: Later
  actors: [it_admin, security_officer, compliance_officer]
  screens: [SCR-ADMIN-uc-emergency-config]
  inputs: [emergency_numbers, dispatchable_location_per_extension, notification_targets]
  states: [configured, tested, failed_test]
  api: "Configuration only; test log POST /v1/properties/{pid}/uc/emergency-tests"
  events: [EmergencyCallPlaced (from PBX), EmergencyCallTestLogged]
  data: [pbx_extension, emergency_test_log]
  rules: ["Emergency dialing works without prefix and without MetriStay availability (PBX local survivability)", "Emergency call notifies security desk and opens an M42 signal", "Local legal emergency-calling obligations per D-421"]
  security: "configuration change dual approval"
  failure_cases: [location_missing, wan_down]
  finance_report_effect: "None"
  i18n_a11y: "Room signage accessible"
  acceptance: "AC-SF34.2.3: With MetriStay offline, test emergency call still routes by PBX; notification creates M42 signal when restored"
  dependency: "D-421, M42"
```

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-420 | PBX/UC vendor and interface protocol for pilot | IT Manager | None selected; manual wake-up list in R1 |
| D-421 | Emergency-calling legal obligations per market (dispatchable location, direct dial) | Local counsel + Security Lead | Treat as PBX vendor responsibility; verify before Phase 7 |

M34 acceptance maps to future Phase 7 acceptance (not in Section G R1 set); invariants: CDR posts once; emergency calling never depends on MetriStay.

---

## 6. M35 — HSIA (Later)

| Header | Value |
|---|---|
| Purpose | Guest and staff internet access: captive portal authentication by stay, entitlements by stay/corporate/event, device/session policies, premium tiers with folio posting, revocation on room move/checkout, telemetry and billable usage. |
| Features (created from Section C) | F35.1 Access and entitlements; F35.2 Lifecycle, telemetry and billing |
| Phases / release | Phase 7, **Later**. R1 relies on the hotel's existing Wi-Fi without MetriStay integration. |
| Bounded context | `hsia` |
| System-of-record entities | `hsia_plan`, `hsia_entitlement`, `hsia_session` (reference to controller session), `hsia_usage_record`, `hsia_voucher` |
| Dependencies | M05 stays, M08 folio, M12 events, M02 consent, M64 network segmentation, D-422 vendor |
| Publishes | `HsiaEntitlementGranted`, `HsiaEntitlementRevoked`, `HsiaPremiumPurchased`, `HsiaUsageRecorded` |

```yaml
- id: M35.F35.1.SF35.1.1
  name: "captive portal authentication"
  phase: 7
  release: Later
  actors: [guest, hsia_worker]
  screens: [SCR-GUEST-wifi-portal]
  inputs: [room_number, surname_or_access_code, device_mac_hash, terms_version]
  states: [unauthenticated, authenticated, denied]
  api: "POST /v1/properties/{pid}/hsia/portal/login (controller RADIUS/API adapter)"
  events: [HsiaSessionStarted]
  data: [hsia_session, hsia_entitlement]
  rules: ["Throttle guessing (room+surname); vouchers for non-resident/event users"]
  security: "rate-limit; MAC stored hashed; no guest PII to network vendor beyond necessity"
  failure_cases: [controller_unreachable, brute_force]
  finance_report_effect: "None"
  i18n_a11y: "Portal WCAG 2.2 AA, EN/AR, no CAPTCHA-only"
  acceptance: "AC-SF35.1.1: 10 wrong attempts lock the room code for 15 minutes"
  dependency: "D-422"
- id: M35.F35.1.SF35.1.2
  name: "guest/staff/corporate/event entitlements"
  phase: 7
  release: Later
  actors: [front_desk_agent, sales_manager, it_admin]
  screens: [SCR-ADMIN-hsia-plans]
  inputs: [plan_id, subject (stay | staff | event | corporate), devices_max, bandwidth]
  states: [granted, active, revoked, expired]
  api: "POST /v1/properties/{pid}/hsia/entitlements"
  events: [HsiaEntitlementGranted]
  data: [hsia_plan, hsia_entitlement]
  rules: ["Staff network segregated from guest network", "Event entitlements tied to BEO dates"]
  security: "VLAN segregation (M64)"
  failure_cases: [overlapping_entitlements]
  finance_report_effect: "Included vs chargeable tracked"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF35.1.2: Event voucher works only within event dates"
  dependency: "M12"
- id: M35.F35.1.SF35.1.3
  name: "device and session policies"
  phase: 7
  release: Later
  actors: [it_admin]
  screens: [SCR-ADMIN-hsia-policies]
  inputs: [max_devices, session_timeout, content_filter_policy_ref]
  states: [active, superseded]
  api: "PUT /v1/properties/{pid}/hsia/policies"
  events: [HsiaPolicyChanged]
  data: [hsia_plan]
  rules: ["Content filtering only per local law/hotel policy; logged decisions"]
  security: "admin only"
  failure_cases: [policy_push_failed]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF35.1.3: Fourth device on a 3-device plan is denied with clear message"
  dependency: "D-422"
- id: M35.F35.1.SF35.1.4
  name: "premium tier purchase and folio posting"
  phase: 7
  release: Later
  actors: [guest, hsia_worker]
  screens: [SCR-GUEST-wifi-upgrade]
  inputs: [plan_id, stay_id, confirmation]
  states: [offered, purchased, posted, refunded]
  api: "POST /v1/properties/{pid}/hsia/purchases (Idempotency-Key)"
  events: [HsiaPremiumPurchased]
  data: [hsia_entitlement, folio_line]
  rules: ["Posts once to folio; tax via M38"]
  security: "guest own stay"
  failure_cases: [double_click_purchase]
  finance_report_effect: "HSIA revenue"
  i18n_a11y: "Accessible price display incl. tax"
  acceptance: "AC-SF35.1.4: Double submission posts one charge"
  dependency: "M08, M38"
- id: M35.F35.2.SF35.2.1
  name: "room-move and checkout revocation"
  phase: 7
  release: Later
  actors: [hsia_worker]
  screens: [SCR-ADMIN-hsia-sessions]
  inputs: [stay_event]
  states: [active, revoked]
  api: "Consumer of GuestCheckedOut/RoomMoved"
  events: [HsiaEntitlementRevoked]
  data: [hsia_entitlement, hsia_session]
  rules: ["Checkout revokes after grace period; room move transfers entitlement"]
  security: "audit"
  failure_cases: [controller_offline_revocation_delayed]
  finance_report_effect: "None"
  i18n_a11y: "Notice page localized"
  acceptance: "AC-SF35.2.1: After checkout + grace, session is disconnected"
  dependency: "M05"
- id: M35.F35.2.SF35.2.2
  name: "network telemetry and capacity"
  phase: 7
  release: Later
  actors: [it_admin]
  screens: [SCR-ADMIN-hsia-telemetry]
  inputs: [ap_metrics, utilisation]
  states: [normal, degraded]
  api: "GET /v1/properties/{pid}/hsia/telemetry"
  events: [HsiaDegraded]
  data: [hsia_usage_record]
  rules: ["Aggregate only; no browsing content stored"]
  security: "network log retention per D-422 and local law"
  failure_cases: [telemetry_gap]
  finance_report_effect: "Feeds M64 health"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF35.2.2: Telemetry store contains no URL/content fields"
  dependency: "M64"
- id: M35.F35.2.SF35.2.3
  name: "billable usage reconciliation"
  phase: 7
  release: Later
  actors: [night_auditor, finance_clerk]
  screens: [SCR-FIN-hsia-reconciliation]
  inputs: [controller_usage_export, posted_charges]
  states: [matched, exception]
  api: "GET /v1/properties/{pid}/hsia/reconciliation"
  events: [HsiaReconciliationException]
  data: [hsia_usage_record, folio_line]
  rules: ["Every premium entitlement has one charge or a documented comp"]
  security: "finance"
  failure_cases: [unmatched_entitlement]
  finance_report_effect: "Revenue assurance"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF35.2.3: Unposted premium entitlement appears as exception"
  dependency: "M60"
```

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-422 | HSIA controller vendor, and network-log retention/lawful-access obligations per market | IT Manager + DPO + local counsel | Not integrated in R1; no network logs held by MetriStay |

M35 invariants: guest/staff network segregation; no browsing content stored; premium posts once. Acceptance: Phase 7 suite (Later).

---

## 7. M36 — IPTV (Later)

| Header | Value |
|---|---|
| Purpose | In-room TV integration: room/device pairing and fleet registry, licensed content and EPG, casting isolation per room, guest services and folio view, purchase posting, checkout reset. |
| Features (created from Section C) | F36.1 Device, content and privacy; F36.2 Guest services and fleet |
| Phases / release | Phase 7, **Later**. |
| Bounded context | `iptv` |
| System-of-record entities | `iptv_device`, `iptv_pairing`, `content_licence_ref`, `iptv_purchase`, `iptv_reset_log` |
| Dependencies | M05, M08, M64, M18 guest services, D-423 vendor |
| Publishes | `IptvDevicePaired`, `IptvReset`, `IptvPurchasePosted` |

```yaml
- id: M36.F36.1.SF36.1.1
  name: "room/device pairing and fleet registry"
  phase: 7
  release: Later
  actors: [it_admin, engineer]
  screens: [SCR-ADMIN-iptv-devices]
  inputs: [device_serial, room_id, firmware, vendor]
  states: [unpaired, paired, faulty, retired]
  api: "PUT /v1/properties/{pid}/iptv/devices/{serial}"
  events: [IptvDevicePaired]
  data: [iptv_device, iptv_pairing]
  rules: ["One device per room slot; firmware inventory in M64"]
  security: "device identity certificates"
  failure_cases: [duplicate_pairing]
  finance_report_effect: "Asset register link (M26)"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF36.1.1: Pairing a device to a second room is rejected"
  dependency: "D-423"
- id: M36.F36.1.SF36.1.2
  name: "licensed content and EPG"
  phase: 7
  release: Later
  actors: [it_admin, compliance_officer]
  screens: [SCR-ADMIN-iptv-content]
  inputs: [channel_lineup, licence_ref, expiry]
  states: [licensed, expiring, expired]
  api: "PUT /v1/properties/{pid}/iptv/lineup"
  events: [ContentLicenceExpiring]
  data: [content_licence_ref]
  rules: ["No channel shown without a current licence reference"]
  security: "admin"
  failure_cases: [licence_lapsed]
  finance_report_effect: "Licence fees as cost"
  i18n_a11y: "EPG supports Arabic and subtitles where provided"
  acceptance: "AC-SF36.1.2: Expired licence removes channel from lineup"
  dependency: "D-423"
- id: M36.F36.1.SF36.1.3
  name: "casting isolation"
  phase: 7
  release: Later
  actors: [guest, it_admin]
  screens: [SCR-GUEST-tv-cast-pairing]
  inputs: [room_id, pairing_code]
  states: [unpaired, paired, expired]
  api: "Vendor casting gateway; MetriStay issues room-scoped pairing token"
  events: [CastPaired]
  data: [iptv_pairing]
  rules: ["Guest devices can cast only to own room TV; pairing expires at checkout"]
  security: "client isolation on network (M64)"
  failure_cases: [cross_room_cast]
  finance_report_effect: "None"
  i18n_a11y: "Instructions accessible"
  acceptance: "AC-SF36.1.3: A device in room 101 cannot discover room 102 TV"
  dependency: "M35"
- id: M36.F36.1.SF36.1.4
  name: "checkout reset"
  phase: 7
  release: Later
  actors: [iptv_worker]
  screens: [SCR-ADMIN-iptv-reset-log]
  inputs: [checkout_event]
  states: [pending, reset, failed]
  api: "Consumer of GuestCheckedOut -> adapter ResetDevice"
  events: [IptvReset]
  data: [iptv_reset_log]
  rules: ["Clears app logins, casting pairings and history"]
  security: "audit"
  failure_cases: [device_offline]
  finance_report_effect: "None"
  i18n_a11y: "Welcome screen language reset to property default"
  acceptance: "AC-SF36.1.4: Offline device reset retried and housekeeping task created if still failing"
  dependency: "M05"
- id: M36.F36.2.SF36.2.1
  name: "guest services and folio view on TV"
  phase: 7
  release: Later
  actors: [guest]
  screens: [SCR-TV-guest-services]
  inputs: [stay_id]
  states: [available, unavailable]
  api: "GET /v1/properties/{pid}/iptv/rooms/{room}/services"
  events: [TvServiceRequestCreated]
  data: [service_request (M18)]
  rules: ["Folio shown only after guest PIN; no PII on idle screen"]
  security: "room-scoped token"
  failure_cases: [pin_bruteforce]
  finance_report_effect: "None"
  i18n_a11y: "Remote-navigable, high contrast, EN/AR"
  acceptance: "AC-SF36.2.1: Folio requires PIN; idle screen shows no guest name"
  dependency: "M18"
- id: M36.F36.2.SF36.2.2
  name: "purchase posting"
  phase: 7
  release: Later
  actors: [guest, iptv_worker]
  screens: [SCR-TV-purchase-confirm]
  inputs: [item_id, price, stay_id]
  states: [confirmed, posted, refunded]
  api: "POST folio charge via M08 (Idempotency-Key = vendor purchase id)"
  events: [IptvPurchasePosted]
  data: [iptv_purchase, folio_line]
  rules: ["Posts once; checked-out room cannot purchase"]
  security: "PIN for purchase"
  failure_cases: [duplicate_vendor_callback]
  finance_report_effect: "In-room entertainment revenue"
  i18n_a11y: "Price incl. tax announced"
  acceptance: "AC-SF36.2.2: Duplicate vendor purchase callback posts once"
  dependency: "M08"
- id: M36.F36.2.SF36.2.3
  name: "hardware fleet health and firmware"
  phase: 7
  release: Later
  actors: [it_admin, engineer]
  screens: [SCR-ADMIN-iptv-fleet]
  inputs: [heartbeat, firmware_version]
  states: [online, offline, outdated]
  api: "GET /v1/properties/{pid}/iptv/fleet"
  events: [IptvDeviceOffline]
  data: [iptv_device]
  rules: ["Offline > threshold creates M26 work order"]
  security: "firmware from signed sources"
  failure_cases: [mass_offline]
  finance_report_effect: "Maintenance cost via M26"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF36.2.3: Device offline 30 min opens a work order"
  dependency: "M26, M64"
```

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-423 | IPTV vendor and content licensing model | IT Manager + GM | Not integrated in R1 |

M36 invariants: casting isolated per room; reset on checkout; purchases post once. Acceptance: Phase 7 suite (Later).

---

## 8. M37 — Marketplace/AI (Later)

| Header | Value |
|---|---|
| Purpose | Phase 8 multi-supplier hotel/experience marketplace (listings, OTA-type discovery, dynamic packages, orders, supplier payouts via licensed partner, disputes/fraud) plus Phase 7 extended supervised AI automations beyond R1 guest chat (tool permission registry, approvals, evaluation) and jurisdiction-gated direct commercial partner incentives (single-tier only). |
| Features (created from Section C) | F37.1 Supplier marketplace; F37.2 Supervised AI automations; F37.3 Jurisdiction-gated partner incentives |
| Phases / release | Phase 7 (F37.2), Phase 8 (F37.1, F37.3). **Later.** No multi-level payouts under any circumstance. |
| Bounded context | `marketplace` (F37.1, F37.3), `ai-governance` (F37.2) |
| System-of-record entities | `marketplace_listing`, `marketplace_supplier` (extends M46 vendor), `marketplace_order`, `merchant_of_record_decision`, `supplier_payout`, `marketplace_dispute`, `ai_tool_grant`, `ai_action_approval`, `ai_eval_run`, `partner_incentive_agreement` |
| Dependencies | M46 vendors, M28 payouts, M40 AI stack, M31 single-tier model, M44 gates, D-424, D-425 |
| Publishes | `ListingPublished`, `MarketplaceOrderPlaced`, `SupplierPayoutReleased`, `AiToolGranted`, `AiActionApproved`, `PartnerIncentiveAccrued` |

```yaml
- id: M37.F37.1.SF37.1.1
  name: "supplier listing onboarding and content"
  phase: 8
  release: Later
  actors: [vendor_admin, content_approver, compliance_officer]
  screens: [SCR-SUP-listings, SCR-MKT-listing-review]
  inputs: [supplier_id, listing_type (hotel | experience | transfer), content, licences, prices]
  states: [draft, in_review, published, suspended]
  api: "POST /v1/marketplace/listings"
  events: [ListingPublished]
  data: [marketplace_listing, marketplace_supplier]
  rules: ["Only M46-verified suppliers with valid licences; content accuracy review; no fabricated amenities"]
  security: "supplier sees own listings only"
  failure_cases: [licence_expired, misleading_content]
  finance_report_effect: "None until orders"
  i18n_a11y: "Listing content EN/AR, accessibility attributes required"
  acceptance: "AC-SF37.1.1: Listing from supplier with expired licence cannot publish"
  dependency: "M46"
- id: M37.F37.1.SF37.1.2
  name: "OTA-type discovery and search"
  phase: 8
  release: Later
  actors: [guest, booker]
  screens: [SCR-MKT-search]
  inputs: [destination, dates, party, filters]
  states: [results, no_results]
  api: "GET /v1/marketplace/search"
  events: [MarketplaceSearchPerformed]
  data: [marketplace_listing, availability_snapshot]
  rules: ["Ranking factors disclosed; paid placement labelled; only bookable live inventory shown"]
  security: "public with rate limits"
  failure_cases: [stale_availability]
  finance_report_effect: "Funnel metrics"
  i18n_a11y: "WCAG 2.2 AA search"
  acceptance: "AC-SF37.1.2: Sponsored results carry visible label"
  dependency: "M51"
- id: M37.F37.1.SF37.1.3
  name: "dynamic packages"
  phase: 8
  release: Later
  actors: [guest, revenue_manager]
  screens: [SCR-MKT-package-builder]
  inputs: [components, dates]
  states: [quoted, held, booked]
  api: "POST /v1/marketplace/packages/quotes"
  events: [PackageQuoted]
  data: [package_quote]
  rules: ["Package travel law per market via M44 before selling"]
  security: "gate per market"
  failure_cases: [component_unavailable]
  finance_report_effect: "Component revenue allocation"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF37.1.3: Package sale blocked in market without verified package-travel rule pack"
  dependency: "M44, M54"
- id: M37.F37.1.SF37.1.4
  name: "orders, merchant of record and supplier payout via licensed partner"
  phase: 8
  release: Later
  actors: [finance_approver, payment_releaser, vendor_admin]
  screens: [SCR-FIN-marketplace-payouts]
  inputs: [order_id, mor_model, commission, payout_schedule]
  states: [ordered, fulfilled, payable, paid, reversed]
  api: "POST /v1/marketplace/orders; payouts via M28 F28.2"
  events: [MarketplaceOrderPlaced, SupplierPayoutReleased]
  data: [marketplace_order, merchant_of_record_decision, supplier_payout]
  rules: ["Funds flow only through licensed PSP split/escrow; no self-custody", "Payout after fulfilment evidence"]
  security: "dual approval"
  failure_cases: [supplier_no_show, chargeback]
  finance_report_effect: "Marketplace commission revenue and payables"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF37.1.4: Payout without fulfilment evidence is blocked"
  dependency: "D-424, M28"
- id: M37.F37.1.SF37.1.5
  name: "disputes, fraud and refunds"
  phase: 8
  release: Later
  actors: [guest_relations, compliance_officer]
  screens: [SCR-MKT-disputes]
  inputs: [order_id, claim]
  states: [open, resolved, escalated]
  api: "POST /v1/marketplace/disputes"
  events: [MarketplaceDisputeOpened]
  data: [marketplace_dispute]
  rules: ["Refund reverses commission and payout once"]
  security: "evidence restricted"
  failure_cases: [supplier_unresponsive]
  finance_report_effect: "Dispute cost"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF37.1.5: Refund reverses supplier payable exactly once"
  dependency: "M28"
- id: M37.F37.2.SF37.2.1
  name: "AI tool permission registry and approval"
  phase: 7
  release: Later
  actors: [it_admin, compliance_officer, dpo]
  screens: [SCR-ADMIN-ai-tools]
  inputs: [tool_id, scope, risk_class, approval_required]
  states: [proposed, approved, active, revoked]
  api: "POST /v1/platform/ai/tools (maker-checker)"
  events: [AiToolGranted]
  data: [ai_tool_grant]
  rules: ["Each AI tool bounded, logged, property-scoped; consequential actions require human approval"]
  security: "risk review before activation"
  failure_cases: [tool_scope_creep]
  finance_report_effect: "AI cost metering"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF37.2.1: AI agent cannot invoke an unregistered tool"
  dependency: "M40, D-425"
- id: M37.F37.2.SF37.2.2
  name: "supervised AI actions beyond guest chat"
  phase: 7
  release: Later
  actors: [duty_manager, revenue_manager, ai_assistant]
  screens: [SCR-STAFF-ai-approvals]
  inputs: [proposed_action, rationale, evidence]
  states: [proposed, approved, rejected, executed]
  api: "POST /v1/platform/ai/actions/{id}/approve"
  events: [AiActionApproved]
  data: [ai_action_approval]
  rules: ["No autonomous money, price, award or safety action"]
  security: "approver role per action class"
  failure_cases: [approval_timeout]
  finance_report_effect: "None directly"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF37.2.2: Rate change proposed by AI executes only after approval"
  dependency: "M53, M63"
- id: M37.F37.2.SF37.2.3
  name: "AI evaluation and audit"
  phase: 7
  release: Later
  actors: [it_admin, auditor]
  screens: [SCR-ADMIN-ai-evals]
  inputs: [eval_suite, model_version]
  states: [running, passed, failed]
  api: "POST /v1/platform/ai/eval-runs"
  events: [AiEvalCompleted]
  data: [ai_eval_run]
  rules: ["Model/prompt change requires passing eval including prompt-injection suite"]
  security: "audit"
  failure_cases: [regression]
  finance_report_effect: "None"
  i18n_a11y: "Evals include Arabic"
  acceptance: "AC-SF37.2.3: Failing injection eval blocks deployment"
  dependency: "SF40.1.5"
- id: M37.F37.3.SF37.3.1
  name: "direct commercial partner incentives (single-tier, legally reviewed)"
  phase: 8
  release: Later
  actors: [referral_program_admin, compliance_officer]
  screens: [SCR-ADMIN-partner-incentives]
  inputs: [partner_id, market, agreement, rate]
  states: [draft, gated, active, disabled]
  api: "POST /v1/referral/partner-incentives"
  events: [PartnerIncentiveAccrued]
  data: [partner_incentive_agreement, commission_ledger_entry]
  rules: ["Reuses M31 single-tier model; no multi-level payouts; activation per market only with counsel review"]
  security: "kill switch shared with M31"
  failure_cases: [market_gate_disabled]
  finance_report_effect: "Commission expense"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF37.3.1: Schema has no multi-level fields; disabled market rejects accrual"
  dependency: "M31, M44"
- id: M37.F37.3.SF37.3.2
  name: "cross-hotel supplier marketplace"
  phase: 8
  release: Later
  actors: [procurement_officer, vendor_admin]
  screens: [SCR-MKT-supplier-directory]
  inputs: [category, location]
  states: [listed, unlisted]
  api: "GET /v1/marketplace/suppliers"
  events: [SupplierDiscovered]
  data: [marketplace_supplier]
  rules: ["Cross-property discovery only with supplier consent; bids remain confidential per property"]
  security: "tenant isolation"
  failure_cases: [confidential_bid_leak]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF37.3.2: Property A cannot see property B's bids"
  dependency: "M46, M49"
```

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-424 | Marketplace merchant-of-record, travel/package licensing and payout partner | Product Owner + Legal | Not in R1 |
| D-425 | Scope of AI automations beyond guest chat and approval classes | Product Owner + Compliance | Only M40 bounded chat and M50 drafting in R1 |

M37 invariants: no self-custody of funds; no multi-level payouts; AI never takes autonomous money/price/award/safety action. Acceptance: Phase 7–8 suites (Later).

---

## 9. M38 — Jurisdiction, tax and government exchange

| Header | Value |
|---|---|
| Purpose | Apply M44-classified, **verified** rule packs to money and people: property legal entity and location, effective-dated tax engine and tax invoice/credit note, product-specific treatment (room, F&B, catering, parking), rounding, filing/remittance calendar and reconciliation, adviser sign-off; Canadian payroll (SIN restricted/encrypted/masked, federal/provincial income tax, CPP/EI, Québec QPP/QPIP, T4 and provincial slips, remittance, amendments); and a registry of **authorized** government portal/file/API connectors with credentials, receipts and a manual path. Oman VAT/WPS/local reporting are configured the same way (WPS file mechanics owned by M27 SF27.3.6). |
| Honesty rules | Never guess a tax rate, levy or payroll parameter: every numeric parameter comes from a rule-pack version with source, effective date and reviewer. Never assert that a government API exists; each route is `api`, `certified_software_file`, `portal_upload` or `manual` with a status label (README §3.6). No scraping, no CAPTCHA bypass, no implied government certification. "NIS" is an **open terminology decision (D-426)** and is never treated as a Canadian identifier; Canada uses SIN. |
| Phases / release | Phase 2 foundation (legal entity, tax engine, invoice); Phase 4 payroll and filing calendar; Phases 5–6 permitted filing adapters and certification. R1. |
| Bounded context | `tax-compliance` (consumes `jurisdiction` context M44) |
| System-of-record entities | `property_legal_entity` (jurisdiction attributes; the legal entity master is M01/M66), `tax_code`, `tax_rule_version` (derived from rule pack), `tax_exemption_evidence`, `tax_determination` (per transaction line, immutable), `tax_invoice_document` (numbering; the folio/invoice record is M08), `filing_obligation`, `filing_calendar_entry`, `tax_return_workpaper`, `remittance`, `adviser_signoff`, `employee_sin_record` (restricted vault), `payroll_statutory_parameter_set` (per year/province), `payroll_statutory_deduction_line`, `tax_slip` (T4/RL-1 etc.), `government_connector`, `government_credential_ref`, `government_submission`, `submission_receipt` |
| Referenced | `jurisdiction_profile`, `rule_pack`, `classification_decision`, `feature_activation_gate` (M44); `folio_line`/`invoice` (M08); `payroll_run` (M27); `gl_journal` (M19) |
| Dependencies | M44 (must be `verified` for automation), M08, M19, M27, M33 connector framework, M02 restricted-field access, secrets vault |
| Publishes | `TaxDetermined`, `TaxInvoiceIssued`, `CreditNoteIssued`, `FilingDue`, `ReturnPrepared`, `AdviserSignedOff`, `GovernmentSubmissionSent`, `GovernmentReceiptRecorded`, `GovernmentSubmissionFailed`, `SinAccessed`, `TaxSlipGenerated`, `RemittanceReconciled` |

**Candidate authorities and routes to verify (all `unverified-assumption` until counsel/partner evidence; recorded in `docs/07` validation register):** Canada — CRA (GST/HST returns; payroll remittances; T4 via internet file transfer/web forms per CRA guidance cited in master prompt), Revenu Québec (QST, QPP, QPIP, RL-1), provincial ministries and municipalities for accommodation taxes/levies; Oman — Tax Authority (VAT, withholding), Ministry of Labour WPS via bank file, municipal/tourism levies; Pakistan — federal and provincial revenue authorities (sales tax on goods vs provincial sales tax on services), provincial tourism/bed tax; Saudi Arabia — ZATCA (VAT, e-invoicing requirements), municipal/tourism fees; Portugal — AT (VAT/IVA, invoicing software and communication requirements, SAF-T), municipal tourist tax. Whether any of these accept a direct API from MetriStay is **unknown** and must be proven per SF38.3.1.

### F38.1 Jurisdiction rules

```yaml
- id: M38.F38.1.SF38.1.1
  name: "property legal entity and country/province/municipality"
  phase: 2
  release: R1
  actors: [property_admin, compliance_officer, financial_controller]
  screens: [SCR-COMP-legal-entity-jurisdiction, SCR-ADMIN-property-setup]
  inputs: [legal_entity_id, registration_numbers (per country), tax_registration_ids, country_iso, subdivision_code, municipality_code, physical_address, effective_from]
  states: [draft, pending_verification, verified, superseded]
  api: "PUT /v1/properties/{pid}/jurisdiction (maker-checker) -> M44 classification"
  events: [PropertyJurisdictionVerified]
  data: [property_legal_entity, jurisdiction_profile (M44)]
  rules: ["Property must have exactly one verified jurisdiction profile per effective period before any taxable sale", "Registration IDs validated by format only; authenticity via evidence document", "Change of municipality/entity is effective-dated, never overwriting history"]
  security: "compliance_officer + financial_controller approval; audit"
  failure_cases: [missing_municipality, conflicting_registration, effective_gap]
  finance_report_effect: "Legal entity is a GL dimension and invoice header"
  i18n_a11y: "Address formats per country; Arabic legal name field; RTL"
  acceptance: "AC-SF38.1.1 (AT-G09): Five fixture properties (CA-ON or CA-QC, OM-MA, PK-PB, SA-01, PT-11) each have one verified profile; a sale on a property with unverified profile is blocked with review path"
  dependency: "M44 SF44.1.1, D-448 launch sequence"
- id: M38.F38.1.SF38.1.2
  name: "tax effective dates and versioned exemption evidence"
  phase: 2
  release: R1
  actors: [compliance_officer, financial_controller, front_desk_agent, ar_clerk]
  screens: [SCR-COMP-tax-rules, SCR-FD-tax-exemption-capture]
  inputs: [tax_code, rule_pack_version_id, effective_from, effective_to, exemption_type, exemption_certificate_ref, guest_or_company_id]
  states: [draft, verified, expired, rejected]
  api: "GET /v1/properties/{pid}/tax/rules?at=; POST /v1/properties/{pid}/tax/exemptions"
  events: [TaxRuleVersionActivated, TaxExemptionRecorded]
  data: [tax_code, tax_rule_version, tax_exemption_evidence]
  rules: ["Rates and thresholds are copied from a verified rule-pack version, never typed ad hoc", "Tax date (supply/stay date per rule) selects the version; stays spanning a change split per night", "Exemption requires evidence document and expiry; expired evidence reverts to taxable with notice"]
  security: "exemption evidence restricted; audit"
  failure_cases: [no_verified_version_for_date, exemption_evidence_expired, retroactive_rate_change]
  finance_report_effect: "Tax payable accounts by code; exemption register"
  i18n_a11y: "Exemption form EN/AR and market language"
  acceptance: "AC-SF38.1.2: A 3-night stay across a fixture rate-change date taxes night 1-2 on v1 and night 3 on v2; with no verified version the quote returns tax_rule_unverified and routes to manual review"
  dependency: "M44 rule packs, D-428 advisers"
- id: M38.F38.1.SF38.1.3
  name: "product-specific hotel room, F&B, catering and parking treatment"
  phase: 2
  release: R1
  actors: [compliance_officer, financial_controller]
  screens: [SCR-COMP-product-tax-matrix]
  inputs: [product_category, jurisdiction_profile_id, tax_codes, levy_codes, place_of_supply_rule]
  states: [draft, verified, expired]
  api: "GET /v1/properties/{pid}/tax/determine (internal, pure function with rule-pack version)"
  events: [TaxDetermined]
  data: [tax_determination, tax_code]
  rules: ["Each sellable category maps to tax and levy codes per jurisdiction; unmapped category cannot be sold", "Packages allocate components before tax (M54)", "Accommodation levies (provincial/municipal) are separate codes with own remittance targets"]
  security: "matrix change maker-checker"
  failure_cases: [unmapped_category, package_allocation_missing, levy_per_night_vs_percent]
  finance_report_effect: "Correct tax/levy liability lines per GL"
  i18n_a11y: "Tax labels localized per invoice language rules"
  acceptance: "AC-SF38.1.3 (AT-G12): Same room+catering bundle quoted in a Canadian province fixture and in Oman fixture produces different tax/levy lines from different verified rule packs; unmapped parking category blocks sale"
  dependency: "M04 quote engine, M54"
- id: M38.F38.1.SF38.1.4
  name: "tax rounding and invoice/credit note"
  phase: 2
  release: R1
  actors: [cashier, ar_clerk, financial_controller]
  screens: [SCR-FIN-tax-invoice, SCR-FIN-credit-note]
  inputs: [folio_or_invoice_id, rounding_rule (line | invoice), numbering_series, language, required_fields_per_jurisdiction]
  states: [draft, issued, credited, voided_not_allowed]
  api: "POST /v1/properties/{pid}/invoices/{id}/issue (Idempotency-Key); POST .../credit-notes"
  events: [TaxInvoiceIssued, CreditNoteIssued]
  data: [tax_invoice_document, tax_determination]
  rules: ["Issued tax invoices are immutable; corrections via credit note referencing the original", "Numbering gap-free per series per legal entity where rule pack requires", "Required fields and QR/e-invoice elements come from the verified rule pack (e.g. where a market mandates e-invoicing clearance the invoice is not final until the clearance route is certified or the manual process applies)"]
  security: "issuing restricted; numbering server-side only"
  failure_cases: [numbering_gap, clearance_route_unavailable, rounding_difference]
  finance_report_effect: "Tax register ties to GL tax accounts"
  i18n_a11y: "Bilingual invoice templates (EN/AR; FR for Québec; PT for Portugal) accessible PDF"
  acceptance: "AC-SF38.1.4 (AT-G09): Localized invoice outputs differ per market fixture; credit note references original and reverses tax once; attempt to edit issued invoice fails"
  dependency: "M08, D-431 rounding"
- id: M38.F38.1.SF38.1.5
  name: "filing/remittance calendar and reconciliation"
  phase: 4
  release: R1
  actors: [financial_controller, finance_clerk, compliance_officer]
  screens: [SCR-FIN-filing-calendar, SCR-FIN-tax-reconciliation]
  inputs: [obligation, period, due_date_rule, gl_accounts, submission_route]
  states: [upcoming, prepared, signed_off, submitted, receipt_recorded, paid, reconciled, overdue]
  api: "GET /v1/properties/{pid}/filings?period=; POST .../filings/{id}/prepare"
  events: [FilingDue, ReturnPrepared, RemittanceReconciled]
  data: [filing_obligation, filing_calendar_entry, tax_return_workpaper, remittance]
  rules: ["Return workpaper = tax determinations for the period reconciled to GL tax accounts; difference must be zero or explained", "Status 'submitted' only with receipt (API/portal reference or uploaded acknowledgment); never inferred"]
  security: "finance scope; adviser read access"
  failure_cases: [gl_tax_difference, missed_due_date, receipt_missing]
  finance_report_effect: "Tax payable roll-forward; overdue alerts to GM flash"
  i18n_a11y: "Calendar accessible; dates in property time zone"
  acceptance: "AC-SF38.1.5: A fixture GST/HST period workpaper ties to GL; marking submitted without receipt is rejected"
  dependency: "M19, SF38.3.x"
- id: M38.F38.1.SF38.1.6
  name: "audit trail and adviser signoff for each location"
  phase: 4
  release: R1
  actors: [compliance_officer, financial_controller, auditor]
  screens: [SCR-COMP-adviser-signoffs]
  inputs: [location, scope, adviser_id, evidence_refs, valid_until]
  states: [requested, signed, expired, withdrawn]
  api: "POST /v1/properties/{pid}/tax/adviser-signoffs"
  events: [AdviserSignedOff]
  data: [adviser_signoff, evidence_source (M44)]
  rules: ["Each property location needs a current adviser signoff for its tax configuration before automated filing is enabled", "Expiry triggers re-review and blocks automation on lapse"]
  security: "adviser external identity with limited scope"
  failure_cases: [signoff_expired]
  finance_report_effect: "Compliance status on filing dashboard"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF38.1.6: With signoff expired, automated submission is disabled and manual path displayed"
  dependency: "D-428"
```

### F38.2 Canada payroll

```yaml
- id: M38.F38.2.SF38.2.1
  name: "validate and encrypt SIN with restricted HR access and masked display"
  phase: 4
  release: R1
  actors: [hr_officer, payroll_officer, employee, dpo, auditor]
  screens: [SCR-HR-employee-identity, SCR-HR-sin-reveal]
  inputs: [employee_id, sin, sin_expiry_if_temporary (9-series), purpose]
  states: [captured, validated, expired_temporary, revoked]
  api: "PUT /v1/hr/employees/{eid}/sin (step-up); POST /v1/hr/employees/{eid}/sin/reveal (reason required)"
  events: [SinRecorded, SinAccessed]
  data: [employee_sin_record]
  rules: ["Luhn check-digit validation; temporary SIN expiry tracked", "Stored field-level encrypted with separate key; display masked (***-***-123); reveal logged with reason", "Never in analytics, BI, AI prompts, exports except T4/remittance filing payloads", "Label is SIN; 'NIS' is not used for Canada (D-426)"]
  security: "payroll/HR roles only; step-up MFA; access log reviewed monthly"
  failure_cases: [invalid_check_digit, temporary_sin_expired, unauthorized_reveal]
  finance_report_effect: "None; enables statutory filings"
  i18n_a11y: "EN/FR labels for Canada"
  acceptance: "AC-SF38.2.1 (AT-G12): Invalid SIN rejected; GM role gets masked value only; reveal by payroll_officer writes SinAccessed with reason; SIN absent from report datasets"
  dependency: "M27 employee master, KMS"
- id: M38.F38.2.SF38.2.2
  name: "federal/provincial income tax and CPP/EI with yearly parameters"
  phase: 4
  release: R1
  actors: [payroll_officer, payroll_approver]
  screens: [SCR-HR-payroll-statutory-preview]
  inputs: [tax_year, province_of_employment, pay_period_type, td1_claims, parameter_set_version]
  states: [parameters_draft, parameters_verified, calculated, approved]
  api: "POST /v1/hr/payroll-runs/{id}/statutory-calc (internal)"
  events: [PayrollStatutoryCalculated]
  data: [payroll_statutory_parameter_set, payroll_statutory_deduction_line]
  rules: ["Parameters (rates, maximums, exemptions, brackets) loaded per year from CRA-published guidance into a verified parameter set; no hardcoded figures", "Results validated against CRA-published calculation examples/tools for the year before activation", "Unverified parameter set blocks payroll finalisation for Canadian employees"]
  security: "payroll roles"
  failure_cases: [parameter_set_unverified, mid_year_change, province_change]
  finance_report_effect: "Employer CPP/EI expense and liabilities posted to GL"
  i18n_a11y: "EN/FR payslip"
  acceptance: "AC-SF38.2.2 (AT-G12): Fixture employee in Ontario produces deductions matching the verified test vector; with parameter set in draft the run cannot be approved"
  dependency: "D-427 payroll engine vs provider, M27"
- id: M38.F38.2.SF38.2.3
  name: "Québec QPP/QPIP and provincial requirements when selected"
  phase: 4
  release: R1
  actors: [payroll_officer, payroll_approver]
  screens: [SCR-HR-payroll-statutory-preview]
  inputs: [province = QC, qpp_parameters, qpip_parameters, quebec_income_tax_parameters]
  states: [parameters_draft, parameters_verified, calculated]
  api: "Same as SF38.2.2 with QC rule pack"
  events: [PayrollStatutoryCalculated]
  data: [payroll_statutory_parameter_set]
  rules: ["QC employment uses QPP instead of CPP, QPIP plus reduced EI rate, Québec income tax, per verified Revenu Québec parameters", "Other provinces' specific items (e.g. health levies) via their rule packs"]
  security: "payroll roles"
  failure_cases: [province_misclassified]
  finance_report_effect: "QPP/QPIP liabilities"
  i18n_a11y: "French payslip for QC"
  acceptance: "AC-SF38.2.3 (AT-G12): QC fixture employee has QPP and QPIP lines and no CPP line"
  dependency: "D-430"
- id: M38.F38.2.SF38.2.4
  name: "T4 and applicable provincial slips/export"
  phase: 4
  release: R1
  actors: [payroll_officer, payroll_approver, employee]
  screens: [SCR-HR-year-end-slips, SCR-STAFF-my-tax-slips]
  inputs: [tax_year, employer_account_number, slip_type (T4 | RL-1), filing_route]
  states: [draft, validated, approved, filed_with_receipt, manual_handoff, amended]
  api: "POST /v1/hr/tax-slips/generate?year=; GET .../export?format=xml"
  events: [TaxSlipGenerated]
  data: [tax_slip, government_submission]
  rules: ["Slip totals reconcile to payroll ledger for the year", "Output format per verified CRA/RQ specification for the year; filing route per SF38.3 (certified file/portal or manual handoff clearly labelled)"]
  security: "slips contain SIN; encrypted at rest and in export"
  failure_cases: [schema_validation_error, totals_mismatch]
  finance_report_effect: "Year-end reconciliation"
  i18n_a11y: "Employee slip view accessible, EN/FR"
  acceptance: "AC-SF38.2.4 (AT-G12): Auditable T4 artifact generated for fixture year with totals equal to ledger; status shows 'receipt recorded' or 'manual handoff', never 'filed' without receipt"
  dependency: "SF38.3.x"
- id: M38.F38.2.SF38.2.5
  name: "employer and employee remittance liability and reconciliation"
  phase: 4
  release: R1
  actors: [payroll_officer, financial_controller, payment_releaser]
  screens: [SCR-FIN-payroll-remittances]
  inputs: [remitter_type, period, amounts, payment_route]
  states: [accrued, due, paid, reconciled]
  api: "POST /v1/hr/remittances (dual approval) -> M28 F28.2"
  events: [RemittanceReconciled]
  data: [remittance]
  rules: ["Remittance = employee withholdings + employer contributions for period; due dates from verified remitter-type rule"]
  security: "dual approval"
  failure_cases: [late_remittance, bank_rejection]
  finance_report_effect: "Liability cleared on bank confirmation"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF38.2.5: Remittance equals ledger liability; bank rejection keeps it due"
  dependency: "M28"
- id: M38.F38.2.SF38.2.6
  name: "correction/amendment and current-year verification"
  phase: 4
  release: R1
  actors: [payroll_officer, payroll_approver, auditor]
  screens: [SCR-HR-payroll-corrections]
  inputs: [original_run_id, correction_reason, amended_slip]
  states: [requested, approved, applied, amended_filed]
  api: "POST /v1/hr/payroll-corrections"
  events: [PayrollCorrectionApplied]
  data: [payroll_statutory_deduction_line, tax_slip]
  rules: ["Corrections are reversing entries; amended slips reference originals", "Annual parameter verification checklist before first payroll of each year"]
  security: "maker-checker"
  failure_cases: [closed_year_correction]
  finance_report_effect: "Adjusting journals"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF38.2.6: Correction produces reversal + new lines; January run blocked until new-year parameter set verified"
  dependency: "M27"
```

### F38.3 Government connector registry

```yaml
- id: M38.F38.3.SF38.3.1
  name: "agency/obligation/protocol eligibility and authorization"
  phase: 2
  release: R1
  actors: [compliance_officer, integration_admin]
  screens: [SCR-COMP-government-connectors]
  inputs: [agency, obligation, route (api | certified_software_file | portal_upload | manual), eligibility_evidence, authorization_ref, status_label]
  states: [unverified-assumption, source-cited, counsel-reviewed, partner-contracted, sandbox-tested, certified, blocked]
  api: "PUT /v1/platform/government-connectors/{id}"
  events: [GovernmentConnectorStatusChanged]
  data: [government_connector]
  rules: ["Registry lists each obligation's route with evidence; api route only after documented eligibility (e.g. software vendor registration) and authorization", "UI shows the honest status label"]
  security: "compliance approval"
  failure_cases: [route_assumed_without_evidence]
  finance_report_effect: "Filing dashboard status"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF38.3.1 (AT-G09): Every obligation in five fixtures has a route and status; none shows api/certified without evidence"
  dependency: "D-429"
- id: M38.F38.3.SF38.3.2
  name: "sandbox/test certificate and credentials in vault"
  phase: 5
  release: R1
  actors: [integration_admin, it_admin]
  screens: [SCR-ADMIN-connector-credentials]
  inputs: [connector_id, environment, credential_type, certificate]
  states: [pending, active, expiring, revoked]
  api: "POST /v1/platform/government-connectors/{id}/credentials (vault write-only)"
  events: [GovernmentCredentialRotated]
  data: [government_credential_ref]
  rules: ["Credentials never in DB/logs; only vault references", "Test and production strictly separated"]
  security: "vault; dual control"
  failure_cases: [certificate_expiry]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF38.3.2: Log scan finds no credential material; expiring cert alerts 30 days ahead"
  dependency: "Vault/KMS"
- id: M38.F38.3.SF38.3.3
  name: "scoped OAuth or mTLS/signing as prescribed by the agency, least privilege, rotation and network restrictions"
  phase: 5
  release: R1
  actors: [integration_admin, it_admin]
  screens: [SCR-ADMIN-connector-security]
  inputs: [auth_scheme, scopes, egress_allowlist, rotation_period]
  states: [configured, verified]
  api: "Adapter configuration"
  events: [GovernmentConnectorSecurityVerified]
  data: [government_connector]
  rules: ["Use only the agency-prescribed scheme; egress allowlist to agency endpoints"]
  security: "network restrictions; rotation"
  failure_cases: [scheme_mismatch]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF38.3.3: Adapter cannot reach non-allowlisted hosts"
  dependency: "M64"
- id: M38.F38.3.SF38.3.4
  name: "request schema/version/idempotency/rate-limit"
  phase: 5
  release: R1
  actors: [gov_filing_worker]
  screens: [SCR-ADMIN-connector-health]
  inputs: [schema_version, submission_key, rate_limit]
  states: [valid, invalid]
  api: "Adapter port GovernmentFiling.submit(payload, idempotency_key)"
  events: [GovernmentSubmissionSent]
  data: [government_submission]
  rules: ["Payload validated against the year's agency schema before send; one submission per key; inquire before retry"]
  security: "audit"
  failure_cases: [schema_year_mismatch, timeout_ambiguous]
  finance_report_effect: "None"
  i18n_a11y: "N/A (machine interface); error messages localized in UI"
  acceptance: "AC-SF38.3.4: After a timeout, adapter queries status before any resend; no duplicate submission"
  dependency: "Agency specs"
- id: M38.F38.3.SF38.3.5
  name: "secure filing payload and minimum data"
  phase: 5
  release: R1
  actors: [payroll_officer, finance_clerk, dpo]
  screens: [SCR-FIN-submission-preview]
  inputs: [return_or_slip_id]
  states: [previewed, approved]
  api: "GET /v1/filings/{id}/payload-preview (masked)"
  events: [FilingPayloadApproved]
  data: [government_submission]
  rules: ["Only fields required by the obligation; encrypted at rest; preview masks identifiers"]
  security: "restricted roles"
  failure_cases: [extra_pii_included]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF38.3.5: Payload lint fails if a non-required PII field is present"
  dependency: "DPO review"
- id: M38.F38.3.SF38.3.6
  name: "submission status/receipt/retry/reversal if supported"
  phase: 5
  release: R1
  actors: [gov_filing_worker, finance_clerk]
  screens: [SCR-FIN-submission-status]
  inputs: [submission_id]
  states: [prepared, sent, acknowledged, accepted, rejected, receipt_recorded, amended]
  api: "GET /v1/filings/{id}/status"
  events: [GovernmentReceiptRecorded, GovernmentSubmissionFailed]
  data: [government_submission, submission_receipt]
  rules: ["'Submitted' only with receipt; reversal only if agency supports, else amendment route"]
  security: "audit"
  failure_cases: [agency_outage, rejection]
  finance_report_effect: "Filing status"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF38.3.6 (AT-G12): Simulated agency outage keeps status 'sent/pending' and alerts; manual handoff path available"
  dependency: "Agency capability"
- id: M38.F38.3.SF38.3.7
  name: "audit, incident notification and retention"
  phase: 5
  release: R1
  actors: [compliance_officer, dpo, it_admin]
  screens: [SCR-COMP-filing-audit]
  inputs: [event_log, retention_policy]
  states: [retained, purge_eligible, legal_hold]
  api: "GET /v1/filings/audit"
  events: [FilingSecurityIncident]
  data: [audit_log, government_submission]
  rules: ["Retain filings per jurisdiction retention rule; security incidents follow M64 process"]
  security: "immutable logs"
  failure_cases: [retention_conflict]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF38.3.7: Submission records cannot be purged before rule-pack retention period"
  dependency: "M44 retention rules"
- id: M38.F38.3.SF38.3.8
  name: "approved portal/file/manual path when API unavailable"
  phase: 4
  release: R1
  actors: [finance_clerk, payroll_officer, compliance_officer]
  screens: [SCR-FIN-manual-filing-handoff]
  inputs: [filing_id, generated_file, portal_reference, uploaded_acknowledgment]
  states: [file_generated, handed_off, acknowledgment_uploaded]
  api: "POST /v1/filings/{id}/manual-receipt"
  events: [GovernmentReceiptRecorded]
  data: [government_submission, submission_receipt]
  rules: ["No scraping, CAPTCHA bypass or robotic portal login", "UI states 'manual handoff' until acknowledgment uploaded"]
  security: "acknowledgment malware-scanned"
  failure_cases: [acknowledgment_missing]
  finance_report_effect: "Filing status"
  i18n_a11y: "Step-by-step accessible instructions"
  acceptance: "AC-SF38.3.8 (AT-G09/G12): Obligation with no API shows manual handoff with generated file; status changes only after acknowledgment upload"
  dependency: "None"
```

### M38 key invariants
1. No tax/levy/payroll parameter without a verified rule-pack version; unverified blocks the dependent automated action.
2. Issued invoices immutable; corrections by credit note.
3. "Submitted" requires a receipt; no scraping or CAPTCHA bypass; no implied government certification.
4. SIN is encrypted, masked, reveal-logged and excluded from analytics/AI; "NIS" never treated as Canadian.

### M38 module acceptance (Section G)
| Test | Scenario | Pass condition |
|---|---|---|
| AT-G09.1 | Five properties across CA/OM/PK/SA/PT | Different effective-dated tax/payroll/document rule packs and localized invoices |
| AT-G09.2 | Unknown/unreviewed obligation | Dependent automated filing blocked; reviewer, evidence and contingency shown |
| AT-G12.1 | Canadian province bundle quote | Correct verified tax/levy lines |
| AT-G12.2 | Canadian payroll | Restricted SIN; CPP or QPP, EI, QPIP, tax deductions |
| AT-G12.3 | T4 artifact | Auditable; receipt or clearly labelled manual handoff; adapter auth and outage behaviour verified |

### M38 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-426 | Meaning of "NIS": intended jurisdiction and identifier (candidates to confirm with the user: Portugal social-security number NISS, UK-style National Insurance number, another national ID); whether a Canadian SIN was intended | Product Owner with the requesting user + HR/Payroll adviser | Canada uses SIN; "NIS" field not created; any jurisdiction-specific employee identifier is modelled generically per rule pack |
| D-427 | Canadian payroll: in-house engine vs certified payroll provider integration | HR Director + Financial Controller | In-house calculation engine with verified yearly parameter sets and test vectors; provider adapter kept possible |
| D-428 | Tax advisers per location and signoff cadence | Financial Controller | Annual signoff per property plus on each rule change |
| D-429 | Permitted filing routes per country/obligation (API vs certified file vs portal vs manual) | Compliance Officer + local advisers | All routes `manual` until evidence; exports produced |
| D-430 | Source and maintenance of Québec (QPP/QPIP/QST/RL-1) parameters | Payroll adviser (Québec) | Parameter sets entered from Revenu Québec publications and verified by adviser |
| D-431 | Tax rounding rule (per line vs per invoice) per market | Tax adviser per market | Per rule pack; default per-line half-up at currency minor unit pending advice |

---

## 10. M44 — Five-market jurisdiction classifier

| Header | Value |
|---|---|
| Purpose | Classify every property and transaction context (country → subnational → municipality → legal entity → transaction place/service/delivery, tax date) and map obligations (tax, invoices, payroll, guest registration, ID/consent, privacy, payments, tourism/travel intermediation, permits, retention, government exchange) to **independent, versioned rule packs** for Canada, Oman, Pakistan, Saudi Arabia and Portugal with evidence, reviewer, status and confidence; drive feature activation gates and exceptions; report coverage and unknowns. |
| Phases / release | Phase 2 foundation (classifier, registry, gates); Phases 4–6 rules and verification. R1. |
| Bounded context | `jurisdiction` (SoR for classification and rule packs; consumed by M30, M31, M38, M41, M43, M45, M47, M50, M52, M57, M61) |
| System-of-record entities | `jurisdiction_node` (ISO 3166-1/-2 plus configurable municipality), `jurisdiction_profile` (property/legal-entity level, effective-dated), `classification_decision` (per transaction context, immutable), `rule_pack` (country × obligation category), `rule_pack_version` (states draft/verified/expired/rejected), `obligation`, `evidence_source`, `rule_test_fixture`, `feature_activation_gate`, `compliance_exception`, `rule_change_alert` |
| Dependencies | M01 property/legal entity, M02 approvals, M33 connectors; counsel/adviser inputs (D-447) |
| Publishes | `JurisdictionClassified`, `RulePackVersionVerified`, `RulePackVersionExpired`, `RulePackVersionRejected`, `FeatureGateChanged`, `ComplianceExceptionApproved`, `RuleChangeAlertRaised` |

### F44.1 Classify

```yaml
- id: M44.F44.1.SF44.1.1
  name: "legal-entity domicile, hotel physical location and subnational authority"
  phase: 2
  release: R1
  actors: [compliance_officer, property_admin]
  screens: [SCR-COMP-jurisdiction-profile]
  inputs: [legal_entity_id, domicile_country, property_address, iso_3166_2, municipality_node_id, effective_from]
  states: [draft, verified, superseded]
  api: "PUT /v1/properties/{pid}/jurisdiction-profile (maker-checker)"
  events: [JurisdictionProfileVerified]
  data: [jurisdiction_profile, jurisdiction_node]
  rules: ["Profile must resolve to a leaf node (municipality where configured) or record explicit 'no municipal authority' evidence", "Entity domicile and property location stored separately"]
  security: "compliance roles"
  failure_cases: [node_missing, ambiguous_boundary]
  finance_report_effect: "Dimension for tax and compliance reports"
  i18n_a11y: "Node names EN/AR plus local language"
  acceptance: "AC-SF44.1.1 (AT-G09): Each of five fixture properties resolves to a distinct profile with subnational code"
  dependency: "D-448"
- id: M44.F44.1.SF44.1.2
  name: "guest residency versus place of supply, service location and supplier jurisdiction"
  phase: 2
  release: R1
  actors: [jurisdiction_worker]
  screens: [SCR-COMP-classification-explorer]
  inputs: [guest_residency, customer_type (b2c | b2b), service_location, supplier_jurisdiction, delivery_context]
  states: [classified, unresolved]
  api: "POST /v1/jurisdiction/classify (pure; returns decision + rule pack versions)"
  events: [JurisdictionClassified]
  data: [classification_decision]
  rules: ["Place of supply determined by the verified rule pack, not by guest residency by default", "Unresolved -> no automated action; manual review"]
  security: "service identity"
  failure_cases: [missing_input, conflicting_rules]
  finance_report_effect: "Drives tax determination"
  i18n_a11y: "Explorer accessible"
  acceptance: "AC-SF44.1.2: Foreign-resident guest at Oman fixture property classified per OM rule pack, not guest country"
  dependency: "Rule packs"
- id: M44.F44.1.SF44.1.3
  name: "effective date and transaction type"
  phase: 2
  release: R1
  actors: [jurisdiction_worker]
  screens: [SCR-COMP-classification-explorer]
  inputs: [tax_point_date, business_date, transaction_type]
  states: [classified]
  api: "Part of POST /v1/jurisdiction/classify"
  events: [JurisdictionClassified]
  data: [classification_decision]
  rules: ["Version chosen by the date rule defined in the pack (e.g. per night of stay)", "Decision stores rule-pack version ids for audit"]
  security: "immutable decisions"
  failure_cases: [date_before_any_version]
  finance_report_effect: "Reproducible tax"
  i18n_a11y: "Dates in property time zone"
  acceptance: "AC-SF44.1.3: Re-classifying a past transaction returns the same versions as stored"
  dependency: "None"
- id: M44.F44.1.SF44.1.4
  name: "deterministic precedence/conflict resolution and audit"
  phase: 2
  release: R1
  actors: [compliance_officer, auditor]
  screens: [SCR-COMP-conflicts]
  inputs: [rules_matched]
  states: [resolved, conflict_escalated]
  api: "GET /v1/jurisdiction/decisions/{id}/explain"
  events: [JurisdictionConflictEscalated]
  data: [classification_decision]
  rules: ["Precedence: municipality > subnational > national within the same obligation unless the pack declares cumulative; equal-precedence conflict escalates, never picks arbitrarily"]
  security: "audit"
  failure_cases: [circular_precedence]
  finance_report_effect: "None"
  i18n_a11y: "Explanation readable"
  acceptance: "AC-SF44.1.4: Two equal-precedence rules produce conflict_escalated and block automation"
  dependency: "None"
- id: M44.F44.1.SF44.1.5
  name: "canonical ISO country/currency and configurable administrative divisions"
  phase: 2
  release: R1
  actors: [compliance_officer, property_admin]
  screens: [SCR-COMP-jurisdiction-nodes]
  inputs: [iso_3166, iso_4217, division_hierarchy]
  states: [active, retired]
  api: "PUT /v1/jurisdiction/nodes/{id}"
  events: [JurisdictionNodeChanged]
  data: [jurisdiction_node]
  rules: ["Currency minor units from ISO 4217 (OMR 3)", "Divisions configurable per country"]
  security: "admin"
  failure_cases: [code_retired]
  finance_report_effect: "Currency precision"
  i18n_a11y: "Localized names"
  acceptance: "AC-SF44.1.5: OMR amounts carry 3 decimals end-to-end"
  dependency: "None"
- id: M44.F44.1.SF44.1.6
  name: "no implicit global fallback for sensitive rules"
  phase: 2
  release: R1
  actors: [jurisdiction_worker, compliance_officer]
  screens: [SCR-COMP-coverage-dashboard]
  inputs: [obligation_category, jurisdiction]
  states: [covered, unknown]
  api: "Classify returns unknown when no verified pack"
  events: [UnknownObligationEncountered]
  data: [classification_decision, compliance_exception]
  rules: ["No default/global pack for tax, payroll, ID, privacy, payments, travel; unknown blocks dependent automation", "Canada or Oman rules never copied to other markets"]
  security: "enforced server-side"
  failure_cases: [developer_default_attempt]
  finance_report_effect: "Unknowns listed on dashboard"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF44.1.6 (AT-G09): Pakistan fixture with payroll pack in draft blocks payroll finalisation and shows reviewer and contingency"
  dependency: "None"
```

### F44.2 Compliance evidence

```yaml
- id: M44.F44.2.SF44.2.1
  name: "independent Canada/Oman/Pakistan/Saudi/Portugal rule-pack registry"
  phase: 2
  release: R1
  actors: [compliance_officer]
  screens: [SCR-COMP-rule-packs]
  inputs: [country, obligation_category, pack_version]
  states: [draft, verified, expired, rejected]
  api: "GET/POST /v1/jurisdiction/rule-packs"
  events: [RulePackVersionCreated]
  data: [rule_pack, rule_pack_version]
  rules: ["Packs are per country and obligation; no cross-country inheritance"]
  security: "compliance roles"
  failure_cases: [copied_pack_detected]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF44.2.1: Registry holds separate packs for 5 countries; cloning between countries is not an API operation"
  dependency: "D-447"
- id: M44.F44.2.SF44.2.2
  name: "obligation category tax/payroll/tourism/ID/privacy/payment/travel"
  phase: 2
  release: R1
  actors: [compliance_officer]
  screens: [SCR-COMP-obligations]
  inputs: [category, description, authority_candidate, applies_to_features]
  states: [identified, sourced, verified]
  api: "POST /v1/jurisdiction/obligations"
  events: [ObligationIdentified]
  data: [obligation]
  rules: ["Each obligation lists dependent features (gate targets)"]
  security: "compliance"
  failure_cases: [unmapped_feature]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF44.2.2: Every gated feature has at least one obligation mapping"
  dependency: "docs/07 register"
- id: M44.F44.2.SF44.2.3
  name: "primary source, effective date, reviewer and counsel/partner validation"
  phase: 4
  release: R1
  actors: [compliance_officer, dpo, auditor]
  screens: [SCR-COMP-evidence]
  inputs: [source_url_or_doc, retrieved_date, effective_date, reviewer, honesty_label]
  states: [unverified-assumption, source-cited, counsel-reviewed]
  api: "POST /v1/jurisdiction/evidence"
  events: [EvidenceRecorded]
  data: [evidence_source]
  rules: ["Verified requires at least source-cited plus named reviewer; payout/filing-critical packs require counsel-reviewed"]
  security: "evidence immutable"
  failure_cases: [source_link_dead]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF44.2.3: Verification without reviewer is rejected"
  dependency: "D-447"
- id: M44.F44.2.SF44.2.4
  name: "draft/verified/expired/rejected states and test fixtures"
  phase: 4
  release: R1
  actors: [compliance_officer, it_admin]
  screens: [SCR-COMP-rule-pack-version]
  inputs: [version_id, fixtures, transition]
  states: [draft, verified, expired, rejected]
  api: "POST /v1/jurisdiction/rule-pack-versions/{id}/transition (maker-checker)"
  events: [RulePackVersionVerified, RulePackVersionExpired, RulePackVersionRejected]
  data: [rule_pack_version, rule_test_fixture]
  rules: ["draft -> verified only if all fixtures pass and evidence complete; verified -> expired at valid_until or on source change; rejected is terminal", "Dependent automated filing/selling/payout requires verified at the transaction date"]
  security: "maker-checker"
  failure_cases: [fixture_failure, expiry_unnoticed]
  finance_report_effect: "Gate status"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF44.2.4 (AT-G09): Expired pack immediately blocks dependent automation; failing fixture blocks verification"
  dependency: "None"
- id: M44.F44.2.SF44.2.5
  name: "local government API, authorized file/portal/manual path"
  phase: 4
  release: R1
  actors: [compliance_officer]
  screens: [SCR-COMP-government-routes]
  inputs: [obligation_id, route, connector_id]
  states: [manual, file, portal, api]
  api: "Link to M38 government_connector"
  events: [GovernmentRouteLinked]
  data: [obligation, government_connector (M38)]
  rules: ["Route claims require M38 SF38.3.1 evidence"]
  security: "compliance"
  failure_cases: [route_without_evidence]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF44.2.5: Obligation route 'api' without connector evidence is rejected"
  dependency: "M38"
- id: M44.F44.2.SF44.2.6
  name: "feature flags and exception approvals scoped by property/jurisdiction"
  phase: 2
  release: R1
  actors: [compliance_officer, gm, tenant_admin]
  screens: [SCR-ADMIN-feature-gates, SCR-COMP-exceptions]
  inputs: [feature_key, scope, required_obligations, exception_reason, expiry]
  states: [blocked, enabled, exception_approved, suspended]
  api: "GET /v1/properties/{pid}/feature-gates; POST /v1/properties/{pid}/compliance-exceptions (dual approval)"
  events: [FeatureGateChanged, ComplianceExceptionApproved]
  data: [feature_activation_gate, compliance_exception]
  rules: ["Gates evaluated in API and background jobs", "Exceptions time-bound, cannot enable automated filing or payout", "Gates cover referral payout, cash wallet, biometric, e-invoicing, travel selling"]
  security: "dual approval"
  failure_cases: [expired_exception]
  finance_report_effect: "None"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF44.2.6: An exception cannot enable referral payout; job re-check honours gate"
  dependency: "None"
- id: M44.F44.2.SF44.2.7
  name: "scheduled change alert and retroactive adjustment"
  phase: 5
  release: R1
  actors: [compliance_officer, financial_controller]
  screens: [SCR-COMP-change-alerts]
  inputs: [review_schedule, source_change, retroactive_effective_date]
  states: [scheduled, alerted, assessed, adjusted]
  api: "POST /v1/jurisdiction/rule-change-alerts"
  events: [RuleChangeAlertRaised]
  data: [rule_change_alert]
  rules: ["Retroactive rule change produces an impact list and adjustment proposals (credit notes/corrections), never silent recalculation"]
  security: "approval"
  failure_cases: [closed_period_impact]
  finance_report_effect: "Adjustments via M38 credit notes / M19 corrections"
  i18n_a11y: "Accessible"
  acceptance: "AC-SF44.2.7: Retroactive fixture change lists affected invoices and proposes credit notes"
  dependency: "M38, M19"
- id: M44.F44.2.SF44.2.8
  name: "five-country coverage/unknowns dashboard"
  phase: 4
  release: R1
  actors: [compliance_officer, gm, owner]
  screens: [SCR-COMP-coverage-dashboard]
  inputs: [countries, categories]
  states: [covered, partial, unknown]
  api: "GET /v1/jurisdiction/coverage"
  events: [CoverageReportGenerated]
  data: [rule_pack_version, obligation]
  rules: ["Matrix countries x categories with status, confidence, reviewer, next review"]
  security: "read roles"
  failure_cases: [stale_matrix]
  finance_report_effect: "Compliance readiness"
  i18n_a11y: "Matrix accessible table"
  acceptance: "AC-SF44.2.8 (AT-G09): Dashboard shows unknowns with owner and contingency"
  dependency: "None"
```

### M44 key invariants
1. Rule packs are independent per country; no global fallback for sensitive categories.
2. Only `verified` versions at the transaction date permit dependent automation; `draft`, `expired`, `rejected` or unknown block it with a manual path.
3. Classification decisions are immutable and reproducible.

### M44 module acceptance (Section G)
| Test | Scenario | Pass condition |
|---|---|---|
| AT-G09.1–3 | Five-market fixtures | Distinct profiles, packs, invoices, submission modes; unknowns blocked with reviewer/evidence/contingency |
| AT-G12.x | Canada | Province/municipality classification drives tax and payroll packs |
| AT-G07.x | Referral gate | Oman/other market gates enforced in jobs |

### M44 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-447 | Named local counsel/adviser per country and obligation category | Compliance Officer | All packs `draft` until assigned |
| D-448 | Five-market launch sequence and pilot property location | Product Owner + Owner | Pilot market decided in Phase 1 decision log; all five seeded as fixtures |
| D-449 | Confidence scoring method for classifications | Compliance Officer | Qualitative: high (counsel-reviewed), medium (source-cited), low (assumption) |
| D-450 | Retroactive adjustment policy for closed periods | Financial Controller | Adjust in current period with reference; no reopen without controller approval |

---
