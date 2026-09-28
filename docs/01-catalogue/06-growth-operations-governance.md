# 01-catalogue/06 — Growth, Operations and Governance (M51–M68)

**Pack:** MetriStay Hospitality Suite — Phase 1 Planning Pack v0.1 (draft for review) • **Date:** 2026-09-28
**Governing source:** master prompt v3.0 Section C rows M51–M68, Section Q (F51.1–F68.2, fixed IDs reused verbatim), Sections G, L, N, O, P.
**Conventions:** `docs/README.md` §3 (identifiers, phases/release flags, standard actors, Section-L schema, API/event style, honesty labels, money/time/quantity).

> Everything below is a specification or design target. No code, partner contract, licensed data feed, certification or legal clearance exists. Where proof from a provider or counsel is missing the entry names the gate and the manual path.

## 0. How to read this file

- Every subfeature is a Section-L YAML block with all 18 fields (`id, name, phase, release, actors, screens, inputs, states, api, events, data, rules, security, failure_cases, finance_report_effect, i18n_a11y, acceptance, dependency`). `acceptance` is the unit-level `AC-<SF id>` test.
- Subfeatures fixed by Section Q keep their ID and name verbatim. Subfeatures **added** in this catalogue to cover Section C/G/O/P requirements carry a trailing `# ADDED — <reason>` comment and continue the numbering of their feature.
- `phase` is the first implementation phase (2–8). `release: R1` = Customer Release 1: Single Hotel (Phases 2–6); `Later` = Phase 7–8.
- Screens use `SCR-<app>-<name>` with apps: `WEB` public website, `KSK` self-service kiosk (Phase 7), `GST` guest app/web, `ADM` back-office admin, `STF` staff mobile (offline-capable), `REV` revenue, `MKT` marketing/CRM, `HK` housekeeping/laundry, `FNB` restaurant/kitchen, `ENG` engineering/IT, `FIN` finance, `OPS` exception/operations, `OWN` owner/GM, `VND` vendor app/web, `CORP` corporate portal/app.
- Actors are the standard roles of `docs/README.md` §3.3 plus two role specializations of `employee` used here: `driver` (M59) and `practitioner` (M58); system actors follow the `*_worker` convention.
- Integrations referenced as `INT-<name>` are specified in `docs/05-integrations.md`; acceptance scenarios `AT-Gnn.n` in `docs/09-acceptance-and-migration.md`.

## 1. Source-of-truth boundaries (no duplicates)

Section Q forbids duplicate guest, room, stock, vendor or finance sources of truth. Modules in this file **own only the entities listed in their header** and reference everything else by module ID.

| Shared record | Owning module (SoR) | How M51–M68 use it |
|---|---|---|
| Staff/guest/vendor identity, roles, scopes, **consent by purpose/channel**, retention/erasure/legal hold | M02 | M51 analytics/cookie choices, M52 campaigns, M55 inbox, M59 passenger data read and write consent **through M02 APIs**; M65 retention maps call M02 deletion. |
| Physical rooms, room-type-by-night stock, OOO/OOS, holds, stop-sell, overbooking limits | M03 | M51 search, M53 restrictions/overbooking, M54 upgrades, M56 release gate, M61 room block, M66 renovation closures. |
| Rate plans, packages, promotions, taxes/fees, quotes and policy snapshots | M04 | M51 total price, M53 publishes rate actions into M04, M54 package pricing and promo eligibility. |
| Reservations/stays, booker/occupant/payer | M05 | M51 booking, M54 ancillary lines, M55 journey tasks, M59 transfers. |
| Room cleaning status (state machine) | M06 | M56 executes tasks and inspections that transition M06 status; no second room-status table. |
| Channel mapping, ARI, commissions, source attribution of channel bookings | M07 | M51 attribution joins M07 source codes; M53 publication acknowledgments come from M07. |
| Folio charges, reversals, cashier shifts, night audit | M08 | M54, M56 minibar, M57, M58, M59 post charges; M60 controls M08 shifts/audits; M55 compensation reverses via M08. |
| Timed resources (tables, spa rooms, courts, vehicles as slots) and composite holds | M09 | M57 table capacity, M58 sessions, M59 vehicle slots. |
| POS checks, tenders, voids/comps, tips | M13 | M57, M58 retail, M60 void/discount outliers. |
| Item/SKU/recipe/BOM master | M14 | M56 linen/minibar/amenity SKUs, M57 recipes/allergens, M58 retail items. |
| **Stock ledger** (receipt/issue/return/waste/quarantine/recall hold) | M14/M50 | M56 linen custody and minibar restock, M57 lots, M61 recall, M67 waste quantities post to or read from M50 `stock_ledger_entry`. |
| GL journals, cost centers, periods, allocations | M19 | Every financial effect below is an M19 posting via the mapping of SF19.1.2; no module keeps a private ledger. |
| AP/AR, payments, payouts | M20/M28 | M52 campaign spend, M59 taxi settlement, M66 fees, M68 premiums flow through M20. |
| Assets, work orders, vendor jobs | M26 | M56 room faults, M59 vehicles, M64 devices, M66 capex assets, M67 anomalies create M26 work orders. |
| Employees, rosters, time, payroll | M27 | M59 drivers, M62 labor demand and training, M60 employee privacy. |
| KPI dictionary rendering, daily flash, report suite | M32 | Consumes M65 `metric_definition`; see D-651. |
| Property media and rights | M39 | M51 pages, M52 templates, M58 amenity listings reference `media_asset` versions only. |
| Guest AI assistant | M40 | M55 inbox handoff target; M51 chat entry point. |
| Incidents and emergency command | M42 | M55, M57, M59, M61, M64, M68 link to `incident`. |
| Lost and found custody | M43 | M56 intake hands off to M43 `found_item`. |
| Jurisdiction rule packs | M44 | M52 consent rules, M57 food rules, M58 intake legality, M61 checklists, M67 factors. |
| Vendors, catalogs, RFQ/PO, receiving | M46/M48/M49/M50 | M56 laundry contracts, M59 taxi providers, M61 pest/water vendors, M67 supplier evidence. |
| Guest profile and preferences | M18 (identity in M02) | M52 merges/segments and M55 routes needs using `guest_profile`; see D-605. |
| Generic task/SLA/escalation/notification engine | **M63** (this file) | All modules in this file create `work_item`s through M63. |

## 2. Module index

| Module | Features | Subfeatures (Section Q + added) | First phase | Release |
|---|---|---|---|---|
| M51 Hotel website and customer acquisition | F51.1–F51.2 | 11 + 3 | 2 | R1 |
| M52 Guest CRM, marketing and reputation | F52.1–F52.2 | 12 + 2 | 3 | R1 |
| M53 Revenue and demand management | F53.1–F53.2 | 11 + 2 | 3 | R1 (+ Later auto) |
| M54 Offers, upsells, vouchers and packages | F54.1–F54.2 | 10 + 2 | 3 | R1 |
| M55 Guest journey and service recovery | F55.1–F55.3 | 12 + 3 | 2 | R1 (+ Later key/kiosk) |
| M56 Housekeeping, laundry, linen and minibar | F56.1–F56.2 | 11 + 1 | 2 | R1 |
| M57 Restaurant, room service and food assurance | F57.1–F57.2 | 11 + 2 | 3 | R1 |
| M58 Optional amenity businesses | F58.1–F58.2 | 10 + 1 | 3 | R1 (hidden unless enabled) |
| M59 Local transport, fleet and dispatch | F59.1–F59.2 | 10 + 1 | 3 | R1 |
| M60 Revenue protection and control | F60.1–F60.2 | 11 + 1 | 2 | R1 |
| M61 Hygiene, safety and inspections | F61.1–F61.2 | 10 + 1 | 3 | R1 |
| M62 Staff enablement and quality | F62.1–F62.2 | 10 + 1 | 3 | R1 |
| M63 Workflow automation and service desk | F63.1–F63.2 | 10 + 1 | 2 | R1 |
| M64 Technology, site devices and resilience | F64.1–F64.2 | 12 + 2 | 2 | R1 (+ Later UC/HSIA/IPTV) |
| M65 Data governance and decision intelligence | F65.1–F65.2 | 11 + 1 | 2 | R1 |
| M66 Owner, brand and property lifecycle | F66.1–F66.2 | 10 + 1 | 4 | R1 (+ Later portfolio) |
| M67 Sustainability and resource performance | F67.1–F67.2 | 10 + 1 | 4 | R1 |
| M68 Enterprise risk, insurance and continuity | F68.1–F68.2 | 10 + 1 | 4 | R1 |

Totals and the decision register are in §20 at the end of this file.

---

## M51 — Hotel website and customer acquisition

| Field | Value |
|---|---|
| Purpose | Fast, multilingual, accessible property website that shows accurate live offers and converts attributable visitors into paid direct stays, while measuring net acquisition cost without violating analytics/privacy choices. |
| Phases | 2 booking website, attribution foundation; 3–5 search/maps/metasearch/advertising integrations and campaign measurement. |
| Release flag | R1 (all subfeatures). |
| Bounded context | `acquisition` |
| SoR entities (owned) | `website_site`, `website_domain`, `website_takedown`, `site_page`, `site_page_version`, `redirect_rule`, `structured_data_snapshot`, `destination_content_item`, `web_search_session`, `quote_funnel_event`, `abandonment_reason`, `acquisition_source`, `campaign_tag`, `attribution_touch`, `attribution_assignment`, `bot_filter_rule`, `acquisition_cost_line` (analytical allocation referencing M20/M19 postings, not a ledger). |
| Referenced (not owned) | M39 `media_asset`/rights; M03 sellable inventory; M04 quote/policy snapshot; M05 reservation; M07 channel source/commission; M28 payment intent; M02 `consent_record` (cookie/analytics/marketing); M40 chat; M44 privacy rule pack; M19/M20 marketing spend. |
| Dependencies | M01 (CDN, flags, locales), M02, M03, M04, M05, M07, M28, M39, M44; INT-cdn, INT-search-console, INT-maps-listing, INT-metasearch, INT-analytics (first-party). |

### F51.1 Publish

```yaml
- id: M51.F51.1.SF51.1.1
  name: responsive property website/theme/domain
  phase: 2
  release: R1
  actors: [property_admin, content_editor, content_approver, it_admin, guest]
  screens: [SCR-ADM-website-settings, SCR-ADM-website-theme, SCR-WEB-home]
  inputs: [property_id, theme_id, brand_tokens, domain_name, dns_txt_token, tls_cert_ref, default_locale, enabled_locales]
  states: [draft, domain_pending_verification, staged, live, suspended]
  api: ["PUT /v1/properties/{pid}/website", "POST /v1/properties/{pid}/website/domains", "POST /v1/properties/{pid}/website/domains/{did}/verify"]
  events: [WebsiteConfigured, WebsiteDomainVerified, WebsiteWentLive, WebsiteSuspended]
  data: [website_site, website_domain]
  rules: ["One live website_site per property in R1; chain sites are Phase 7.", "Domain goes live only after DNS TXT verification and valid TLS certificate.", "Theme uses design tokens only; no custom script injection outside the approved tag allowlist.", "Mobile-first layout; server-side rendered pages must work without client JavaScript for search, quote and booking."]
  security: "Only property_admin/it_admin change domain and TLS; step-up MFA for domain change; CSP, HSTS and SRI enforced; admin API property-scoped."
  failure_cases: [dns_not_verified, tls_expired, cdn_outage_origin_fallback, theme_token_contrast_fail]
  finance_report_effect: "None directly; site hosting cost is an AP expense in M20 allocated to marketing cost center in M19."
  i18n_a11y: "English and Arabic with RTL mirroring per locale; theme tokens validated for WCAG 2.2 AA contrast; skip links and landmark regions required."
  acceptance: "AC-SF51.1.1: With an unverified domain the site stays staged; after TXT verification and TLS issuance it is live; Lighthouse-equivalent audit on a 3G profile shows first contentful paint under the agreed pilot budget and zero AA contrast failures in both locales."
  dependency: "M01 CDN/TLS automation, M02 admin roles; INT-cdn; pilot domain ownership (D-601)."

- id: M51.F51.1.SF51.1.2
  name: room/venue media and facilities from approved M39 CMS
  phase: 2
  release: R1
  actors: [content_editor, content_approver, guest]
  screens: [SCR-ADM-website-page-editor, SCR-WEB-room-type, SCR-WEB-venue]
  inputs: [room_type_id, venue_id, media_asset_version_ids, facility_attribute_ids, display_order]
  states: [unlinked, linked_pending_approval, published, withdrawn]
  api: ["PUT /v1/properties/{pid}/website/pages/{page_id}/media-links", "GET /v1/public/properties/{slug}/room-types/{rtid}"]
  events: [WebsiteMediaLinked, WebsiteMediaWithdrawn]
  data: [site_page_version, media_asset (M39), room_type (M03), facility_attribute (M03/M09)]
  rules: ["Only M39 media_asset versions in status approved with unexpired usage rights may render.", "Room size, bed type, view and accessibility attributes come from M03/M09 records, never free text typed on the page.", "Rights expiry or M39 takedown removes the asset from the site within the CDN purge SLA.", "Enhanced images must carry the M39 provenance flag; no image may show a facility the property does not have."]
  security: "Public read of published data only; editors can link but cannot publish without content_approver."
  failure_cases: [media_rights_expired, asset_takedown, missing_alt_text, attribute_conflict_with_M03]
  finance_report_effect: "None."
  i18n_a11y: "Alt text and captions required per enabled locale from M39; video has captions/subtitles; lazy-loaded responsive derivatives for low bandwidth."
  acceptance: "AC-SF51.1.2: An asset whose M39 rights expire at T disappears from the public room page by T plus purge SLA; an unapproved enhanced copy never renders (AT-G13.1)."
  dependency: "M39 SF39.1.2, SF39.1.5, SF39.2.4; M03 room-type attributes."

- id: M51.F51.1.SF51.1.3
  name: location, accessibility and policy content with owner/version
  phase: 2
  release: R1
  actors: [content_editor, content_approver, front_office_manager, compliance_officer, guest]
  screens: [SCR-ADM-policy-content, SCR-WEB-location, SCR-WEB-accessibility, SCR-WEB-policies]
  inputs: [content_type, locale, body, owner_user_id, review_due_date, effective_from, linked_policy_snapshot_id]
  states: [draft, in_review, approved, published, review_overdue, retired]
  api: ["POST /v1/properties/{pid}/website/content", "POST /v1/properties/{pid}/website/content/{cid}/versions/{v}/approve"]
  events: [WebsiteContentApproved, WebsiteContentReviewOverdue]
  data: [site_page, site_page_version]
  rules: ["Each accessibility, cancellation, deposit, child, pet and ID policy item has a named owner and review date.", "Cancellation/deposit wording must match the M04 policy snapshot applied in quotes; mismatch blocks publish.", "Accessibility content describes measured features (door widths, step-free route, roll-in shower) sourced from M03 accessible-room attributes.", "Overdue review raises an M63 work_item to the owner; content stays live but flagged internally."]
  security: "Policy content requires content_approver plus compliance_officer for legal policy types."
  failure_cases: [policy_snapshot_mismatch, owner_left_company, translation_missing]
  finance_report_effect: "None."
  i18n_a11y: "All policy content in every enabled locale before publish; plain-language summary plus full text; map has text directions alternative."
  acceptance: "AC-SF51.1.3: Changing the M04 cancellation window without updating website policy blocks the next policy publish and raises a mismatch task; accessibility page lists only attributes present on M03 accessible rooms."
  dependency: "M04 policy snapshot; M03 accessible attributes; M63 tasks; D-602 map provider."

- id: M51.F51.1.SF51.1.4
  name: multilingual pages, metadata/structured data, redirects/sitemap and page performance
  phase: 2
  release: R1
  actors: [content_editor, marketing_manager, it_admin]
  screens: [SCR-ADM-seo-metadata, SCR-ADM-redirects, SCR-ADM-web-performance]
  inputs: [page_id, locale, title, meta_description, canonical_url, hreflang_map, schema_type, redirect_from, redirect_to, perf_budget]
  states: [valid, warning, invalid_blocked]
  api: ["PUT /v1/properties/{pid}/website/pages/{page_id}/seo", "POST /v1/properties/{pid}/website/redirects", "GET /v1/public/properties/{slug}/sitemap.xml"]
  events: [WebsiteSeoUpdated, WebsiteSitemapRegenerated, WebsitePerfBudgetBreached]
  data: [structured_data_snapshot, redirect_rule, site_page_version]
  rules: ["Structured data (Hotel, HotelRoom, Offer, Place) is generated from SoR records; offers in markup must equal a live M04 quote for the stated dates or be omitted.", "No review/rating markup unless sourced from M52 genuine first-party reviews under the approved policy.", "Redirect chains longer than one hop and loops are rejected.", "Performance budget per page type; breach raises an M63 task and blocks theme releases that worsen it."]
  security: "Redirect targets restricted to owned domains or allowlisted partners (open-redirect prevention)."
  failure_cases: [redirect_loop, invalid_schema, stale_offer_markup, perf_budget_breach]
  finance_report_effect: "None."
  i18n_a11y: "Locale-specific URLs with hreflang; Arabic pages dir=rtl with lang attribute; metadata translated, not machine-duplicated without review."
  acceptance: "AC-SF51.1.4: Schema validator passes for all published pages; an Offer markup price differing from the live quote fails the nightly check and is removed; a redirect loop cannot be saved."
  dependency: "M04 quote API; M52 SF52.2.6 review policy; INT-search-console (Phase 3, optional)."

- id: M51.F51.1.SF51.1.5
  name: content approval/takedown
  phase: 2
  release: R1
  actors: [content_editor, content_approver, compliance_officer, dpo]
  screens: [SCR-ADM-content-approval-queue, SCR-ADM-takedown]
  inputs: [content_version_id, decision, reason_code, takedown_scope, legal_request_ref]
  states: [submitted, approved, rejected, scheduled, published, taken_down]
  api: ["POST /v1/properties/{pid}/website/content/{cid}/versions/{v}/decision", "POST /v1/properties/{pid}/website/takedowns"]
  events: [WebsiteContentPublished, WebsiteContentTakenDown]
  data: [site_page_version, website_takedown]
  rules: ["Maker-checker: editor cannot approve own version.", "Takedown is immediate, purges CDN and logs requester, reason and scope; restoring requires a new approval.", "Scheduled publish respects property time zone.", "Every published version is retained for audit and rollback."]
  security: "Takedown available to content_approver, compliance_officer, dpo; all decisions audited with actor and timestamp."
  failure_cases: [cdn_purge_failed_retry, approver_unavailable_escalation, conflicting_scheduled_versions]
  finance_report_effect: "None."
  i18n_a11y: "Approval shows side-by-side locales; RTL preview mandatory for Arabic."
  acceptance: "AC-SF51.1.5: Editor self-approval is rejected (403); a takedown removes the page from CDN within the SLA and leaves an immutable audit row; rollback to prior version requires approval."
  dependency: "M01 CDN purge; M02 roles; M63 approvals."

- id: M51.F51.1.SF51.1.6  # ADDED — Section C 'destination and amenity content' and 'accurate real-time offers'; Section P.2
  name: Destination and amenity content with live offer accuracy check
  phase: 3
  release: R1
  actors: [content_editor, marketing_manager, fnb_manager, guest]
  screens: [SCR-ADM-destination-content, SCR-WEB-amenities, SCR-OPS-offer-accuracy]
  inputs: [amenity_ref, outlet_ref, opening_hours, destination_item, source_ref, claim_type, check_frequency]
  states: [accurate, stale, mismatch, suppressed]
  api: ["POST /v1/properties/{pid}/website/destination-items", "GET /v1/properties/{pid}/website/accuracy-checks"]
  events: [WebsiteOfferMismatchDetected, WebsiteAmenitySuppressed]
  data: [destination_content_item, site_page_version]
  rules: ["Amenity pages render only outlets/amenities enabled in M01 feature flags and M58; disabled amenity is hidden, not shown as closed-soon.", "Opening hours and prices shown are read from M09/M13/M58 at render time or labelled 'from' with as-of date.", "Nightly check compares every priced claim on the site with live M04/M54 offers; mismatch suppresses the claim and creates a task.", "Third-party destination facts carry source and review date; no distance or travel-time claims without a source."]
  security: "Public read only; accuracy report internal to marketing_manager and gm."
  failure_cases: [amenity_disabled_but_linked, price_claim_mismatch, source_expired]
  finance_report_effect: "None."
  i18n_a11y: "Opening hours localized with property time zone; accessible table markup."
  acceptance: "AC-SF51.1.6: Disabling the spa in M58 removes spa pages and offers on the next render; a stale priced claim is suppressed by the nightly check and an owner task is created."
  dependency: "M58 SF58.1.1, M54 offers, M04 quotes, M63."
```

### F51.2 Convert and measure

```yaml
- id: M51.F51.2.SF51.2.1
  name: live date/party/accessibility search with true sellable inventory
  phase: 2
  release: R1
  actors: [guest, booker, front_desk_agent]
  screens: [SCR-WEB-search, SCR-WEB-results, SCR-GST-search]
  inputs: [arrival_date, departure_date, adults, children_ages, rooms, accessibility_needs, promo_code, locale, currency]
  states: [searching, results, no_availability, alternative_dates, error_degraded]
  api: ["GET /v1/public/properties/{slug}/availability?arrival&departure&adults&children&access"]
  events: [WebSearchPerformed]
  data: [web_search_session, room_type_night_stock (M03), quote (M04)]
  rules: ["Availability is read from M03 sellable stock net of holds, OOO/OOS and stop-sell; no cached count older than the configured freshness window is shown as bookable.", "Accessibility filter matches M03 accessible-room attributes and only returns room types with an assignable accessible room.", "Occupancy/child rules come from M04; ineligible combinations are explained, not silently hidden.", "When no availability, show alternative dates and a contact path, never a fabricated scarcity message."]
  security: "Anonymous rate-limited endpoint with bot throttling; no PII required to search."
  failure_cases: [inventory_service_timeout, stale_cache, invalid_date_range, rate_limit_exceeded]
  finance_report_effect: "Search counts feed the funnel denominator in M65 metric_definition 'quote_conversion'."
  i18n_a11y: "Date pickers keyboard-operable with typed-date alternative; Hijri display optional in Arabic locale while storing Gregorian ISO dates; results announced via live region."
  acceptance: "AC-SF51.2.1: With one accessible room OOO the accessible filter returns no bookable accessible option; concurrent searches never return a room type whose M03 stock is zero (AT-G19.1, AT-G20.1)."
  dependency: "M03 availability API, M04 quote engine."

- id: M51.F51.2.SF51.2.2
  name: total taxes/fees/policy and abandoned checkout recovery where consented
  phase: 2
  release: R1
  actors: [guest, booker, marketing_manager, outbox_relay]
  screens: [SCR-WEB-quote-summary, SCR-WEB-checkout, SCR-MKT-abandonment-recovery]
  inputs: [quote_id, rate_plan_id, tax_breakdown, fees, policy_snapshot_id, email_or_phone, recovery_consent_flag]
  states: [quoted, checkout_started, abandoned, recovery_eligible, recovery_sent, recovered, expired]
  api: ["POST /v1/public/properties/{slug}/quotes", "POST /v1/public/properties/{slug}/checkouts", "POST /v1/properties/{pid}/abandonment-recovery/{checkout_id}/send"]
  events: [QuoteIssued, CheckoutStarted, CheckoutAbandoned, AbandonmentRecoverySent, CheckoutRecovered]
  data: [quote (M04), quote_funnel_event, consent_record (M02)]
  rules: ["The first displayed price for a stay is the all-in total including mandatory taxes and fees from M04/M44; optional extras are separately labelled.", "Quote shows expiry and the policy snapshot; price at payment equals the quote unless expired, in which case the guest sees the re-quote difference before paying.", "Recovery message only if M02 consent for that purpose and channel exists and M44 rule pack permits; one reminder per checkout by default; unsubscribe honored.", "Recovery sending activates in Phase 3 when M52 SF52.1.5 is live; Phase 2 records abandonment only."]
  security: "Checkout contact data encrypted at rest; recovery link is single-use, short-lived and does not reveal PII in the URL."
  failure_cases: [quote_expired_at_payment, tax_rule_unverified_block, consent_missing, message_delivery_failed]
  finance_report_effect: "Tax/fee components are those later posted to M08 folio; funnel events feed conversion and recovered-revenue metrics (estimate until stay is paid)."
  i18n_a11y: "Currency formatting per locale with correct minor units (OMR 3 decimals); price breakdown readable by screen readers as a list."
  acceptance: "AC-SF51.2.2: For a Canadian and an Omani fixture property the displayed total equals the M04 quote including levies; a guest without marketing consent never receives a recovery message; an expired quote forces visible re-quote (AT-G19.2, AT-G09.1)."
  dependency: "M04 tax/fee engine, M44 rule packs (unverified pack blocks sale), M02 consent, M52 messaging."

- id: M51.F51.2.SF51.2.3
  name: approved search/maps/metasearch/OTA/GDS referrals and campaign tags
  phase: 3
  release: R1
  actors: [marketing_manager, integration_admin, revenue_manager]
  screens: [SCR-MKT-acquisition-sources, SCR-ADM-integration-health]
  inputs: [source_type, partner_id, contract_status, campaign_tag, utm_params, click_id, landing_url, cost_model]
  states: [proposed, contracted, sandbox_tested, active, paused, terminated, blocked]
  api: ["POST /v1/properties/{pid}/acquisition-sources", "PUT /v1/properties/{pid}/acquisition-sources/{sid}/status", "POST /v1/properties/{pid}/campaign-tags"]
  events: [AcquisitionSourceActivated, AcquisitionSourcePaused, CampaignTagCreated]
  data: [acquisition_source, campaign_tag]
  rules: ["A source may send priced traffic only after contract status partner-contracted and a sandbox test; else status blocked with manual listing path.", "Metasearch price feeds use the same M04 quote logic as the website (no feed-only lower price unless an approved M04 promotion).", "OTA/GDS bookings are attributed via M07 source codes, not website tags.", "Campaign tags are unique per property and immutable once used."]
  security: "Partner credentials in vault; integration_admin only; honesty label shown on every source."
  failure_cases: [feed_rejected_by_partner, price_parity_mismatch, partner_credentials_expired]
  finance_report_effect: "Each source carries a cost model (CPC, commission, fixed) used to compute net acquisition cost with actual AP/commission postings from M20/M07."
  i18n_a11y: "Admin screens bilingual; no guest-facing UI."
  acceptance: "AC-SF51.2.3: An uncontracted metasearch source cannot be activated; a feed price mismatch vs M04 is flagged and the feed paused within one cycle."
  dependency: "INT-metasearch, INT-maps-listing (partner contracts, D-602); M07 source codes."

- id: M51.F51.2.SF51.2.4
  name: cookie/analytics choices
  phase: 2
  release: R1
  actors: [guest, dpo, marketing_manager]
  screens: [SCR-WEB-consent-banner, SCR-WEB-privacy-preferences, SCR-ADM-analytics-settings]
  inputs: [visitor_pseudonymous_id, consent_categories, jurisdiction_code, consent_version, timestamp]
  states: [not_asked, accepted_all, rejected_non_essential, customized, withdrawn]
  api: ["POST /v1/public/consents", "GET /v1/public/consents/{visitor_id}"]
  events: [AnalyticsConsentRecorded, AnalyticsConsentWithdrawn]
  data: [consent_record (M02), website_site]
  rules: ["Consent is stored in M02 by purpose; M51 owns only the banner configuration.", "No non-essential cookie, pixel or third-party tag fires before a matching choice; reject is as easy as accept.", "Banner text and defaults come from the M44 privacy rule pack of the visitor-facing jurisdiction configured for the site.", "Withdrawal takes effect on the next page load and deletes non-essential identifiers."]
  security: "Pseudonymous ID only; no IP stored beyond security logs; tag allowlist enforced by CSP."
  failure_cases: [rule_pack_unverified_default_strict, tag_fires_without_consent_detected, consent_service_unavailable_default_deny]
  finance_report_effect: "Attribution coverage metric reports the share of sessions without analytics consent (reported as unknown, not modelled as a fact)."
  i18n_a11y: "Banner keyboard-trappable only until choice; focus returned; bilingual; not a blocking CAPTCHA."
  acceptance: "AC-SF51.2.4: Automated browser test shows zero third-party requests before consent and after reject; withdrawal removes identifiers; unverified rule pack defaults to strict opt-in."
  dependency: "M02 consent API, M44 privacy rule packs, D-603 analytics tool choice."

- id: M51.F51.2.SF51.2.5
  name: quote->paid-stay attribution and channel net cost
  phase: 2
  release: R1
  actors: [marketing_manager, revenue_manager, gm, owner]
  screens: [SCR-MKT-attribution-report, SCR-REV-channel-net-contribution, SCR-OWN-acquisition-cost]
  inputs: [attribution_touch_ids, reservation_id, source_code, precedence_policy_version, commission, payment_fees, marketing_spend_period]
  states: [touch_recorded, assigned, stay_completed, paid, reversed, net_cost_estimated, net_cost_reconciled]
  api: ["GET /v1/properties/{pid}/acquisition/attribution?from&to", "GET /v1/properties/{pid}/acquisition/net-cost?period"]
  events: [AttributionAssigned, AttributionReversed, AcquisitionNetCostComputed]
  data: [attribution_touch, attribution_assignment, acquisition_cost_line]
  rules: ["Exactly one attribution_assignment per reservation using the versioned precedence policy (M07 channel source > referral code M31 > last consented click > direct).", "Revenue counts only when the stay is completed and paid; cancellations/refunds reverse the assignment.", "Net cost = commission + payment fees + allocated campaign spend per completed stay; each component labelled estimate until M19 period posted, then reconciled.", "M31 referral attribution remains the M31 SoR; M51 reads it and never double-counts."]
  security: "Reports aggregate; guest-level drill only for roles with guest-data scope; no PII sent to ad partners."
  failure_cases: [conflicting_sources, late_channel_booking_import, refund_after_period_close, missing_spend_invoice]
  finance_report_effect: "Feeds M32 channel net contribution and M65 profitability attribution; restated values follow M19 period rules."
  i18n_a11y: "Charts have tabular alternative and screen-reader summaries."
  acceptance: "AC-SF51.2.5: A booking from a metasearch click then cancelled shows zero realized revenue and one reversal; net acquisition cost per completed stay shows estimate vs reconciled labels (AT-G19.3, AT-G08.1)."
  dependency: "M05, M07 source/commission, M28 fees, M20/M19 postings, M31 attribution."

- id: M51.F51.2.SF51.2.6
  name: bot/duplicate attribution filtering and privacy-safe report
  phase: 3
  release: R1
  actors: [marketing_manager, dpo, it_admin]
  screens: [SCR-MKT-traffic-quality, SCR-ADM-bot-rules]
  inputs: [session_signals, user_agent_class, rate_pattern, click_id, dedup_window, rule_version]
  states: [clean, suspected_bot, duplicate, excluded, restored]
  api: ["POST /v1/properties/{pid}/acquisition/bot-rules", "GET /v1/properties/{pid}/acquisition/traffic-quality"]
  events: [TrafficExcluded, BotRuleActivated]
  data: [bot_filter_rule, attribution_touch]
  rules: ["Duplicate click IDs within the window count once.", "Excluded traffic is retained with reason for audit and can be restored by rule rollback.", "Reports show counts with minimum aggregation thresholds so no individual visitor can be re-identified.", "Bot filtering never blocks a real booking; it affects attribution only."]
  security: "Raw signals retained per M65 retention map then deleted; access limited to it_admin and dpo."
  failure_cases: [false_positive_real_guest, rule_regression, signal_unavailable]
  finance_report_effect: "Paid-click spend disputes with partners use excluded-traffic evidence."
  i18n_a11y: "Report bilingual with accessible tables."
  acceptance: "AC-SF51.2.6: Replaying the same click ID five times yields one touch; a suppressed cohort under threshold is shown as '<k' not exact."
  dependency: "M65 retention map; INT-analytics."

- id: M51.F51.2.SF51.2.7  # ADDED — Section G19 accessible low-bandwidth booking with optional upgrade; Section P.1/P.2 canonical journey
  name: Accessible low-bandwidth direct booking checkout with optional upgrade
  phase: 2
  release: R1
  actors: [guest, booker, payment_worker]
  screens: [SCR-WEB-guest-details, SCR-WEB-extras, SCR-WEB-payment, SCR-WEB-confirmation]
  inputs: [quote_id, guest_details, occupants, special_requests, selected_offer_ids, payment_method_token, policy_acceptance, idempotency_key]
  states: [details, extras, hold_placed, payment_pending, confirmed, payment_failed, hold_expired]
  api: ["POST /v1/public/properties/{slug}/bookings", "POST /v1/public/properties/{slug}/bookings/{bid}/payments"]
  events: [BookingHoldPlaced, BookingConfirmed, BookingPaymentFailed, BookingHoldExpired]
  data: [reservation (M05), inventory_hold (M03), payment_intent (M28), ancillary_order (M54)]
  rules: ["Sequence: M03 hold -> M04 quote validation -> M28 payment -> M05 confirmation; confirmation page only after payment success or policy-permitted guarantee.", "Optional upgrade offers come from M54 with live inventory and are added to the same quote total before payment.", "Idempotency-Key required; a double submit yields one reservation and one capture.", "Pages under the low-bandwidth budget and functional without JavaScript; no CAPTCHA, use rate limits and accessible challenge alternatives."]
  security: "Card data only in PSP-hosted fields (PCI SAQ-A boundary); tokenized method; booking reference not guessable."
  failure_cases: [hold_expired_before_payment, payment_declined, duplicate_submit, psp_timeout_pending]
  finance_report_effect: "Deposit/prepayment posts to M08 folio and M19 guest deposit liability; upgrade revenue allocated per M54 SF54.1.5."
  i18n_a11y: "WCAG 2.2 AA: error summary with links to fields, no timeouts under 20 s without extension, accessible authentication; RTL forms; name fields accept Arabic script."
  acceptance: "AC-SF51.2.7: On a throttled 3G profile a screen-reader user completes room plus upgrade purchase; total equals quote; a double click creates one booking and one capture (AT-G19.2, AT-G20.2)."
  dependency: "M03, M04, M05, M28 certified PSP, M54."

- id: M51.F51.2.SF51.2.8  # ADDED — Section O revenue/sales 'how many direct visitors abandon a quote and why'
  name: Quote abandonment reasons and funnel diagnostics
  phase: 3
  release: R1
  actors: [revenue_manager, marketing_manager, gm]
  screens: [SCR-MKT-funnel, SCR-REV-abandonment-reasons]
  inputs: [funnel_step, exit_step, error_code, price_shown, availability_state, optional_exit_survey_answer]
  states: [collected, classified, reviewed]
  api: ["GET /v1/properties/{pid}/acquisition/funnel?from&to&segment", "POST /v1/public/funnel-feedback"]
  events: [FunnelStepRecorded, AbandonmentClassified]
  data: [quote_funnel_event, abandonment_reason]
  rules: ["Reasons are classified from observable facts (no availability, price shown, payment error, validation error, timeout) plus optional voluntary survey; no inferred personal traits.", "Funnel counts only sessions with analytics consent; non-consented sessions are reported as a coverage gap.", "Reason taxonomy is versioned in M65."]
  security: "Aggregated reporting; free-text survey answers screened for PII and retained per M65."
  failure_cases: [low_coverage_warning, taxonomy_version_change]
  finance_report_effect: "Feeds quote-to-book conversion KPI (Section P.6 baseline)."
  i18n_a11y: "Survey optional, one question, accessible and bilingual."
  acceptance: "AC-SF51.2.8: Simulated payment declines and no-availability exits appear as distinct reasons with counts matching test fixtures; coverage percentage is shown."
  dependency: "SF51.2.4 consent, M65 SF65.1.2."
```

### M51 key invariants

1. The site never offers a room, amenity or price that M03/M04/M09/M58 cannot sell at that moment (Section P.3 "no ghost room sale").
2. Exactly one attribution per reservation; revenue attributed only on completed paid stays; reversals on refund.
3. No non-essential tracking before consent; consent SoR is M02.
4. All media rendered is an approved, rights-valid M39 version; no fabricated facilities.
5. Displayed first price is the all-in mandatory total.

### M51 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF51.1.2, AC-SF51.1.6 | AT-G13 (approved media only), AT-G19 (accurate website) | Marketing: "Can guests discover accurate room, event and amenity listings? Which photos/videos are current and rights-cleared?" |
| AC-SF51.2.1, AC-SF51.2.7 | AT-G19 (accessible low-bandwidth purchase), AT-G20 (concurrent bookings) | Guest: "Can I find a suitable accessible room at an honest total price?" |
| AC-SF51.2.2 | AT-G09, AT-G12 (jurisdiction totals) | Front desk: "Which rate and taxes apply?" |
| AC-SF51.2.5 | AT-G19 (net acquisition cost), AT-G08 (drill to source) | Revenue: "Which OTA or campaign drives profitable stays?"; Marketing: "What did it cost per completed stay?" |
| AC-SF51.2.8 | AT-G19 (quote conversion) | Revenue: "How many direct visitors abandon a quote and why?" |

### M51 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-601 | Website brand, domain and hosting (SaaS CDN vs hotel-owned domain on on-prem profile). | Product Owner + pilot GM | Hotel-owned domain pointed at MetriStay SaaS CDN; on-prem profile serves website from SaaS edge only. |
| D-602 | Search/maps/metasearch partners and contract owners. | Marketing Manager (pilot) | None contracted; all sources `blocked` with manual listing path; mock metasearch feed for tests. |
| D-603 | First-party analytics tool (self-hosted vs vendor) and data residency. | DPO + IT Admin | Self-hosted first-party analytics in the property's deployment region; no third-party ad pixels in R1 until D-603 closes. |
| D-604 | Attribution precedence and lookback window per market. | Revenue Manager + DPO | Channel source > referral code > last consented click within 30 days > direct; versioned in M65. |

---

## M52 — Guest CRM, marketing and reputation

| Field | Value |
|---|---|
| Purpose | Permissioned single view of each guest relationship for service and lawful marketing: merge/dedup with uncertainty, consent-aware segmented messaging, post-stay surveys, approved-channel review handling, and repeat-guest/lifetime-value reporting. |
| Phases | 3 profile merge, messaging, surveys and reviews; 5 advanced campaigns, measurement and lifetime value. |
| Release flag | R1. |
| Bounded context | `crm` |
| SoR entities (owned) | `guest_merge_candidate`, `guest_merge_decision`, `guest_identity_link`, `send_eligibility_decision`, `review_policy`, `subject_tag`, `segment_definition`, `segment_snapshot`, `message_template`, `campaign`, `campaign_send`, `message_delivery`, `frequency_cap_policy`, `survey_definition`, `survey_response`, `external_review`, `review_response`, `guest_value_snapshot`. |
| Referenced (not owned) | M18 `guest_profile`/preferences; M02 `consent_record`, suppression, erasure; M05 stays; M08 spend; M55 `guest_case`; M39 media; M44 marketing/privacy rules; M30 loyalty; M63 notifications; M20 campaign spend. |
| Dependencies | M02, M05, M08, M18, M39, M44, M55, M63; INT-sms, INT-whatsapp (approved templates), INT-email, INT-review-sources. |

### F52.1 Guest relationship

```yaml
- id: M52.F52.1.SF52.1.1
  name: permissioned merged guest/contact with identity uncertainty and correction
  phase: 3
  release: R1
  actors: [guest_relations, front_office_manager, dpo, crm_worker]
  screens: [SCR-MKT-merge-review, SCR-MKT-guest-360]
  inputs: [guest_profile_ids, match_signals, match_score, reviewer_decision, split_reason]
  states: [candidate, auto_linked, pending_review, merged, rejected, split]
  api: ["GET /v1/properties/{pid}/crm/merge-candidates", "POST /v1/properties/{pid}/crm/merge-candidates/{cid}/decision", "POST /v1/properties/{pid}/crm/guests/{gid}/split"]
  events: [GuestMergeProposed, GuestProfilesMerged, GuestProfileSplit]
  data: [guest_merge_candidate, guest_merge_decision, guest_identity_link, guest_profile (M18)]
  rules: ["Merge creates a link record over M18 profiles; the surviving profile ID is kept and source IDs remain resolvable (no destructive overwrite).", "Auto-link only on verified exact identifiers (verified email plus verified phone); fuzzy matches go to human review.", "ID-document numbers from M41 are never used as match signals.", "Split fully reverses a merge and re-points stays, cases and consents to the original profiles."]
  security: "Merge/split restricted to guest_relations and front_office_manager; guest-360 fields filtered by role and purpose; all views logged."
  failure_cases: [false_merge, conflicting_consents_on_merge, corporate_shared_email, deleted_profile_in_candidate]
  finance_report_effect: "Repeat-guest and lifetime-value metrics recalculate after merge/split; folios remain on original reservations."
  i18n_a11y: "Name matching handles Arabic/Latin transliteration variants as signals only; review screen compares fields side by side, screen-reader friendly."
  acceptance: "AC-SF52.1.1: Two profiles with the same name but different verified emails are not auto-merged; a merged pair can be split with all stays and consents restored; conflicting consents resolve to the most restrictive."
  dependency: "M18 guest_profile, M02 consent, M65 SF65.1.1 governance."

- id: M52.F52.1.SF52.1.2
  name: preferences and service history with purpose boundaries
  phase: 3
  release: R1
  actors: [guest_relations, front_desk_agent, housekeeper, guest]
  screens: [SCR-MKT-guest-360, SCR-STF-guest-preferences, SCR-GST-my-preferences]
  inputs: [preference_type, value, source, purpose_tag, sensitivity_class]
  states: [active, guest_hidden, expired, deleted]
  api: ["GET /v1/properties/{pid}/guests/{gid}/preferences?purpose", "PUT /v1/properties/{pid}/guests/{gid}/preferences/{prefid}"]
  events: [GuestPreferenceUpdated]
  data: [guest_profile (M18), guest_preference (M18)]
  rules: ["Every preference carries a purpose tag (service, marketing); marketing use requires the matching M02 consent.", "Sensitive categories (health, dietary for medical reasons, accessibility, religion inferred from diet) are service-only, never used for segmentation.", "Guests can view and correct their preferences in the guest app.", "Housekeeping sees only room-service-relevant preferences."]
  security: "Field-level scopes by role; sensitive fields encrypted and access-logged."
  failure_cases: [sensitive_field_used_in_segment_blocked, stale_preference, guest_deletion_request]
  finance_report_effect: "None."
  i18n_a11y: "Preference labels localized; guest correction form accessible."
  acceptance: "AC-SF52.1.2: A segment definition referencing an allergy or accessibility field is rejected at save; a housekeeper cannot read marketing preferences (403)."
  dependency: "M18 preferences, M02 purpose consents."

- id: M52.F52.1.SF52.1.3
  name: country/channel-specific consent/opt-out and suppression
  phase: 3
  release: R1
  actors: [guest, marketing_manager, dpo, crm_worker]
  screens: [SCR-GST-communication-preferences, SCR-MKT-suppression, SCR-WEB-unsubscribe]
  inputs: [guest_id, channel, purpose, jurisdiction_code, opt_in_evidence, opt_out_source]
  states: [no_consent, opted_in, soft_opt_in_permitted, opted_out, suppressed_legal, suppressed_bounce]
  api: ["GET /v1/properties/{pid}/crm/send-eligibility?guest_id&channel&purpose", "POST /v1/public/unsubscribe/{token}"]
  events: [MarketingOptOutRecorded, SuppressionApplied]
  data: [consent_record (M02), suppression_entry (M02), send_eligibility_decision]
  rules: ["Eligibility evaluates M02 consent plus M44 channel rules for the guest's applicable jurisdiction; unknown jurisdiction defaults to explicit opt-in.", "Opt-out via any channel suppresses that purpose within the rule-pack deadline (default immediate).", "Transactional service messages (confirmation, pre-arrival instructions) are separated from marketing and never carry marketing content.", "Suppression survives profile merge and erasure (hashed suppression token)."]
  security: "Consent evidence immutable; DPO-only override with reason; unsubscribe token single-purpose."
  failure_cases: [rule_pack_unverified, whatsapp_template_not_approved, opt_out_race_with_send]
  finance_report_effect: "None."
  i18n_a11y: "Unsubscribe in one step without login, bilingual, screen-reader accessible."
  acceptance: "AC-SF52.1.3: An opt-out recorded one second before a batch send excludes the guest; a Portugal fixture guest without explicit opt-in is ineligible while a transactional confirmation still sends."
  dependency: "M02 consent SoR, M44 marketing rule packs (Canada CASL, Portugal/EU, Oman, Saudi, Pakistan: counsel review D-606)."

- id: M52.F52.1.SF52.1.4
  name: segment by permitted criteria, lifecycle and frequency cap
  phase: 3
  release: R1
  actors: [marketing_manager, dpo]
  screens: [SCR-MKT-segment-builder, SCR-MKT-frequency-caps]
  inputs: [criteria_expression, lifecycle_stage, purpose, cap_per_period, snapshot_time]
  states: [draft, validated, approved, snapshot_taken, archived]
  api: ["POST /v1/properties/{pid}/crm/segments", "POST /v1/properties/{pid}/crm/segments/{sid}/snapshot"]
  events: [SegmentApproved, SegmentSnapshotTaken]
  data: [segment_definition, segment_snapshot, frequency_cap_policy]
  rules: ["Criteria are limited to an allowlist (stay history, spend band, lifecycle, language, consented interests); protected characteristics and sensitive fields are not selectable.", "Snapshot is frozen at send time for audit; membership recomputed per campaign.", "Frequency cap applies across all campaigns per guest and channel.", "Segments below a minimum size cannot be exported."]
  security: "Segment export disabled by default; enabled per campaign with DPO approval."
  failure_cases: [prohibited_criterion, empty_segment, cap_exceeded]
  finance_report_effect: "None."
  i18n_a11y: "Builder has non-drag keyboard alternative."
  acceptance: "AC-SF52.1.4: Nationality or religion cannot be selected as criteria; a guest already at cap is excluded and counted in the send report."
  dependency: "SF52.1.2, SF52.1.3."

- id: M52.F52.1.SF52.1.5
  name: template/approval/send/delivery/failure
  phase: 3
  release: R1
  actors: [marketing_manager, content_approver, crm_worker, outbox_relay]
  screens: [SCR-MKT-templates, SCR-MKT-campaign-send, SCR-MKT-delivery-log]
  inputs: [template_id, locale_variants, channel, media_asset_ids, segment_snapshot_id, schedule_at, idempotency_key]
  states: [draft, approved, scheduled, sending, sent, partially_failed, cancelled]
  api: ["POST /v1/properties/{pid}/crm/templates", "POST /v1/properties/{pid}/crm/campaigns/{cid}/send", "POST /v1/integrations/messaging/callbacks"]
  events: [CampaignScheduled, CampaignMessageSent, MessageDeliveryFailed, CampaignCompleted]
  data: [message_template, campaign, campaign_send, message_delivery]
  rules: ["Maker-checker on template and campaign; approved WhatsApp templates only on that channel.", "Eligibility re-checked per recipient at send time.", "Idempotent send per recipient per campaign; provider callbacks deduplicated by provider message ID.", "Send windows respect guest local time and quiet hours; failures retried with backoff then marked failed."]
  security: "Provider keys in vault; message content scanned for PII placeholders not resolved; links signed."
  failure_cases: [provider_outage, template_rejected, duplicate_callback, bounce_to_suppression]
  finance_report_effect: "Messaging provider cost accrued per campaign to marketing cost center (M20/M19)."
  i18n_a11y: "Template variants per locale with RTL preview; plain-text and accessible HTML email."
  acceptance: "AC-SF52.1.5: A replayed provider callback does not double-count delivery; a crash mid-send resumes without duplicate messages."
  dependency: "INT-email, INT-sms, INT-whatsapp; M63 notifications; M39 media."

- id: M52.F52.1.SF52.1.6
  name: campaign conversion and cost
  phase: 5
  release: R1
  actors: [marketing_manager, revenue_manager, gm]
  screens: [SCR-MKT-campaign-results]
  inputs: [campaign_id, attribution_window, holdout_percent, spend_lines]
  states: [collecting, provisional, final]
  api: ["GET /v1/properties/{pid}/crm/campaigns/{cid}/results"]
  events: [CampaignResultsFinalized]
  data: [campaign, attribution_assignment (M51), acquisition_cost_line (M51)]
  rules: ["Conversions count only completed paid stays attributed per M51 precedence.", "Where holdout is configured, report incremental effect with confidence interval; without holdout, report attributed not incremental.", "Cost includes messaging cost, voucher/discount cost from M54 and creative cost lines."]
  security: "Aggregates only."
  failure_cases: [insufficient_sample, late_refunds_restate]
  finance_report_effect: "Campaign ROI uses M19-posted costs once period closes (estimate before)."
  i18n_a11y: "Accessible charts with tables."
  acceptance: "AC-SF52.1.6: A campaign with a refunded conversion shows it reversed; report labels 'attributed' vs 'incremental' correctly."
  dependency: "M51 SF51.2.5, M54, M19."

- id: M52.F52.1.SF52.1.7  # ADDED — Section C 'repeat-guest and lifetime-value reporting'; Section N 'bring guests back'
  name: Repeat-guest and lifetime-value reporting
  phase: 5
  release: R1
  actors: [marketing_manager, gm, owner]
  screens: [SCR-MKT-repeat-guests, SCR-OWN-guest-value]
  inputs: [period, merge_state, revenue_components, cost_components]
  states: [computed, restated]
  api: ["GET /v1/properties/{pid}/crm/guest-value?period"]
  events: [GuestValueSnapshotComputed]
  data: [guest_value_snapshot]
  rules: ["Lifetime value = realized net revenue minus direct acquisition cost per merged guest; forecasted LTV labelled estimate.", "Repeat rate denominator defined in M65.", "Individual-level values visible only to roles with guest-data scope; never used to deny service."]
  security: "Aggregates default; drill to guest requires scope."
  failure_cases: [merge_split_restatement]
  finance_report_effect: "Consumes M08/M19 revenue; no postings."
  i18n_a11y: "Accessible tables."
  acceptance: "AC-SF52.1.7: Splitting a merged guest restates repeat rate for affected periods with a restatement note."
  dependency: "SF52.1.1, M65 metric definitions."

- id: M52.F52.1.SF52.1.8  # ADDED — Section O technology/compliance 'delete data by purpose'; Section P.4
  name: Data-subject request propagation across CRM
  phase: 3
  release: R1
  actors: [dpo, guest]
  screens: [SCR-ADM-dsr-queue, SCR-GST-privacy-requests]
  inputs: [dsr_id, request_type, verified_identity_ref, scope]
  states: [received, verified, executing, completed, partially_retained]
  api: ["POST /v1/properties/{pid}/crm/dsr/{dsr_id}/execute"]
  events: [CrmDsrExecuted]
  data: [dsr (M02), segment_snapshot, survey_response, message_delivery]
  rules: ["Erasure removes marketing data, survey free text and segments; retains legally required folio, tax, safety evidence per M02 legal-hold and records the retained-by-purpose reason.", "Suppression token kept to honor opt-out.", "Export includes CRM data in machine-readable form."]
  security: "DPO-only execution; identity verified via M02."
  failure_cases: [legal_hold_conflict, merged_profile_scope]
  finance_report_effect: "None; financial records unaffected."
  i18n_a11y: "Guest request form accessible and bilingual."
  acceptance: "AC-SF52.1.8: After erasure, CRM search returns no guest data while the folio remains with pseudonymized name per retention rule; the guest stays suppressed."
  dependency: "M02 DSR workflow, M65 SF65.2.6."
```

### F52.2 Reputation

```yaml
- id: M52.F52.2.SF52.2.1
  name: post-stay survey and internal case
  phase: 3
  release: R1
  actors: [guest, guest_relations, crm_worker]
  screens: [SCR-GST-survey, SCR-MKT-survey-results]
  inputs: [reservation_id, survey_definition_id, answers, score, free_text, contact_permission]
  states: [scheduled, sent, responded, case_created, closed, expired]
  api: ["POST /v1/public/surveys/{token}/responses", "GET /v1/properties/{pid}/crm/surveys/responses"]
  events: [SurveySent, SurveyResponded, SurveyLowScoreCaseOpened]
  data: [survey_definition, survey_response, guest_case (M55)]
  rules: ["Survey is a transactional service message sent once per stay after checkout; no incentive tied to rating.", "Scores below threshold or complaint keywords create an M55 guest_case with owner and SLA.", "Survey invitation never asks for or gates public reviews on positive scores (no review gating)."]
  security: "Survey token single-use, bound to reservation; free text PII-screened."
  failure_cases: [token_reused, delivery_failed, guest_opted_out_of_surveys]
  finance_report_effect: "None."
  i18n_a11y: "Accessible form with scale labels; bilingual."
  acceptance: "AC-SF52.2.1: A score of 2/10 opens an M55 case within one minute; all respondents receive the same public-review link regardless of score."
  dependency: "M55 SF55.2.1, M05 checkout event."

- id: M52.F52.2.SF52.2.2
  name: external review ingestion only via approved channel
  phase: 3
  release: R1
  actors: [integration_admin, guest_relations, review_worker]
  screens: [SCR-MKT-review-inbox, SCR-ADM-integration-health]
  inputs: [source_id, external_review_id, rating, text, published_at, reviewer_display_name, language]
  states: [ingested, matched_to_stay, unmatched, response_needed, responded, removed_by_source]
  api: ["POST /v1/integrations/reviews/{source}/webhook", "GET /v1/properties/{pid}/crm/reviews"]
  events: [ExternalReviewIngested, ExternalReviewRemoved]
  data: [external_review]
  rules: ["Ingest only through contracted API or source-permitted export; no scraping.", "Reviews are stored as received and never edited; matching to a stay is optional, internal and probabilistic with confidence label.", "Source status honesty label shown."]
  security: "Source credentials in vault; matched stay link visible only to guest_relations."
  failure_cases: [source_api_unavailable, duplicate_ingest, terms_change]
  finance_report_effect: "None."
  i18n_a11y: "Original language kept; optional machine translation labelled as such."
  acceptance: "AC-SF52.2.2: Replaying the same webhook stores one review; a source without contract cannot be enabled."
  dependency: "INT-review-sources (contract per source, D-607)."

- id: M52.F52.2.SF52.2.3
  name: response queue/approval, privacy and anti-retaliation
  phase: 3
  release: R1
  actors: [guest_relations, marketing_manager, gm, dpo]
  screens: [SCR-MKT-review-response]
  inputs: [review_id, draft_text, ai_draft_flag, approver_id]
  states: [draft, pending_approval, approved, published, publish_failed]
  api: ["POST /v1/properties/{pid}/crm/reviews/{rid}/responses", "POST /v1/properties/{pid}/crm/reviews/{rid}/responses/{resp}/approve"]
  events: [ReviewResponsePublished]
  data: [review_response]
  rules: ["Public responses never disclose stay details, room numbers, names or health information.", "AI-drafted responses are labelled internally and require human approval.", "Negative reviewers cannot be flagged, blacklisted or have rates changed because of the review (anti-retaliation); the system has no such action.", "Response SLA tracked via M63."]
  security: "Publish requires approver distinct from drafter."
  failure_cases: [pii_detected_block, source_publish_api_unavailable_manual_path]
  finance_report_effect: "None."
  i18n_a11y: "Response in reviewer's language where possible; RTL editor."
  acceptance: "AC-SF52.2.3: A draft containing a room number is blocked by the PII check; no endpoint exists to tag a guest based on a review."
  dependency: "M40 drafting (optional), M63 SLA."

- id: M52.F52.2.SF52.2.4
  name: guest follow-up and recovery closure
  phase: 4
  release: R1
  actors: [guest_relations, duty_manager, guest]
  screens: [SCR-MKT-recovery-follow-up, SCR-OPS-guest-cases]
  inputs: [guest_case_id, follow_up_message, guest_confirmation]
  states: [follow_up_due, contacted, guest_confirmed_resolved, reopened, closed_no_response]
  api: ["POST /v1/properties/{pid}/guest-cases/{case_id}/follow-ups"]
  events: [GuestFollowUpSent, GuestCaseClosureConfirmed]
  data: [guest_case (M55)]
  rules: ["Follow-up is a service message under the case, not marketing.", "Closure requires guest confirmation or documented no-response after the configured attempts.", "Case lifecycle SoR stays in M55."]
  security: "Case data scoped to guest_relations and management."
  failure_cases: [guest_unreachable, reopen_after_closure]
  finance_report_effect: "Recovery cost from M55 SF55.2.3 attached to case outcome."
  i18n_a11y: "Messages localized; accessible reply."
  acceptance: "AC-SF52.2.4: A case cannot move to closed without guest confirmation or three logged attempts."
  dependency: "M55 F55.2."

- id: M52.F52.2.SF52.2.5
  name: source/rating/subject trends
  phase: 5
  release: R1
  actors: [gm, marketing_manager, guest_relations]
  screens: [SCR-MKT-reputation-trends, SCR-OWN-reputation]
  inputs: [period, source, subject_taxonomy_version]
  states: [computed]
  api: ["GET /v1/properties/{pid}/crm/reputation/trends"]
  events: [ReputationTrendsComputed]
  data: [external_review, survey_response, subject_tag]
  rules: ["Subject tagging (cleanliness, staff, food, noise) by versioned taxonomy; AI tagging shows confidence and can be corrected.", "Scores are never blended across sources without showing each source separately."]
  security: "Aggregates."
  failure_cases: [taxonomy_change_restatement]
  finance_report_effect: "None."
  i18n_a11y: "Accessible charts."
  acceptance: "AC-SF52.2.5: Trend report reproduces fixture counts by source and subject; corrected tags update counts."
  dependency: "SF52.2.1, SF52.2.2, M65."

- id: M52.F52.2.SF52.2.6
  name: no fabricated or incentivized deceptive reviews
  phase: 3
  release: R1
  actors: [marketing_manager, compliance_officer, auditor]
  screens: [SCR-MKT-review-policy, SCR-ADM-audit-log]
  inputs: [review_policy_version, incentive_request]
  states: [policy_active]
  api: ["GET /v1/properties/{pid}/crm/review-policy"]
  events: [ReviewPolicyViolationBlocked]
  data: [review_policy]
  rules: ["The product has no function to create, import or edit reviews attributed to guests, or to post reviews on third-party sites.", "Incentives may not be conditioned on a review or its rating; any M54 voucher linked to a survey is blocked.", "Website review markup only from genuine M52 survey responses with published methodology; otherwise omitted.", "Staff-authored content is always labelled as hotel content."]
  security: "Attempts to create review records outside ingestion/survey paths are rejected and logged."
  failure_cases: [attempt_to_link_voucher_to_review]
  finance_report_effect: "None."
  i18n_a11y: "Policy text bilingual."
  acceptance: "AC-SF52.2.6: API has no create-review endpoint; linking an M54 voucher to a survey completion condition fails validation with a policy error."
  dependency: "M54 offer rules, M51 SF51.1.4."
```

### M52 key invariants

1. No marketing message without M02 consent evaluated against the M44 rule pack at send time; opt-out wins races.
2. Merges are reversible links over M18 profiles; no destructive merge.
3. Sensitive/protected attributes are never segmentation criteria.
4. No fabricated, gated or incentivized reviews; no retaliation actions exist.

### M52 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF52.1.3, AC-SF52.1.4, AC-SF52.1.5 | AT-G19 (consented review), AT-G20 (duplicate webhook) | Marketing: "Which campaign can contact which consenting guests?" |
| AC-SF52.1.6, AC-SF52.1.7 | AT-G19 (net acquisition cost) | Marketing: "What did it cost per completed stay? Which guests return?" |
| AC-SF52.2.1, AC-SF52.2.3, AC-SF52.2.4 | AT-G19 (complaint -> recovery -> consented review) | Marketing: "What complaints or public reviews need response? What service recovery actually closed the case?" |
| AC-SF52.1.8 | AT-G20 (exceptions visible) | Technology/compliance: "Can we delete data by purpose without deleting required accounting and safety evidence?" |

### M52 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-605 | Guest profile SoR boundary between M18 and M52 (who executes merges). | Lead Architect | M18 owns `guest_profile`; M52 owns merge candidates/decisions/links and calls M18 to re-point. |
| D-606 | Per-market marketing consent model (CASL, EU/Portugal ePrivacy/GDPR, Oman, KSA PDPL, Pakistan). | DPO + local counsel | Explicit opt-in per channel in all five markets until counsel-reviewed. |
| D-607 | Review sources to contract (and whether response publishing via API is allowed). | Marketing Manager | No source contracted; manual copy-in by staff flagged as manual ingest; responses posted manually. |
| D-608 | Messaging providers (email/SMS/WhatsApp BSP) per market. | IT Admin + Procurement | Provider-neutral port with mock; WhatsApp only with approved templates. |

---

## M53 — Revenue and demand management

| Field | Value |
|---|---|
| Purpose | Evidence-based demand forecast and pricing/restriction decisions measured on net contribution, with explainable recommendations, role guardrails, human approval, acknowledged publication, rollback and honest backtests. |
| Phases | 3 baseline (pace/pickup, manual guardrailed rate change, publish/ack/rollback); 5 forecast confidence, recommendations, simulation, backtest; 7 advanced bounded automation. |
| Release flag | R1 (Phases 3–5); `Later` for SF53.2.7. |
| Bounded context | `revenue` |
| SoR entities (owned) | `demand_snapshot`, `pace_curve`, `forecast_run`, `forecast_value`, `market_input`, `demand_event_calendar`, `rate_recommendation`, `rate_action`, `guardrail_policy`, `rate_publication_ack`, `price_discrepancy_item`, `backtest_result`, `overbooking_recommendation`. |
| Referenced (not owned) | M03 inventory, OOO/OOS, overbooking limits; M04 rate plans/restrictions (the published price SoR); M05 reservations/cancellations; M07 channel ARI and acknowledgments; M10/M12 corporate contracts and group blocks; M08/M19 realized revenue and fees; M51 funnel; M65 metric definitions. |
| Dependencies | M03, M04, M05, M07, M10, M12, M19, M32, M65; INT-channel-manager; INT-market-data (licensed, optional). |

### F53.1 Forecast

```yaml
- id: M53.F53.1.SF53.1.1
  name: pace/pickup by stay date and booking date
  phase: 3
  release: R1
  actors: [revenue_manager, gm, revenue_worker]
  screens: [SCR-REV-pace-grid, SCR-REV-pickup-report, SCR-OWN-forecast-summary]
  inputs: [stay_date_range, snapshot_date, segment, channel, room_type, horizon_days]
  states: [snapshot_pending, snapshot_taken, snapshot_late]
  api: ["GET /v1/properties/{pid}/revenue/pace?from&to&as_of", "GET /v1/properties/{pid}/revenue/pickup?days"]
  events: [DemandSnapshotTaken, DemandSnapshotLate]
  data: [demand_snapshot, pace_curve]
  rules: ["A demand snapshot of on-the-books room nights and revenue by stay date is taken at each business-date close (M08 night audit) and is immutable.", "Pickup is the difference between snapshots, never recomputed from current reservations.", "Horizon default 365 days; 90-day view is the default screen (Section G19).", "Group blocks show contracted, picked-up and remaining separately (from M12)."]
  security: "revenue_manager, gm, owner read; no guest PII in snapshots."
  failure_cases: [night_audit_delayed_snapshot_late, backdated_reservation_change, missing_prior_year]
  finance_report_effect: "On-the-books revenue is an estimate; realized revenue comes from M08/M19 after stay."
  i18n_a11y: "Grid keyboard-navigable with row/column headers; numbers localized; RTL mirrored grid."
  acceptance: "AC-SF53.1.1: After three simulated business dates with fixture bookings the 90-day pickup equals the fixture difference; a backdated amendment appears in the next snapshot only (AT-G19.4)."
  dependency: "M08 night audit event, M05 reservations, M12 blocks."

- id: M53.F53.1.SF53.1.2
  name: cancellations/no-shows/group wash/OOO supply
  phase: 3
  release: R1
  actors: [revenue_manager, sales_manager]
  screens: [SCR-REV-supply-and-wash]
  inputs: [stay_date, cancellation_history, no_show_history, block_id, ooo_room_nights]
  states: [computed, overridden]
  api: ["GET /v1/properties/{pid}/revenue/wash-and-supply?from&to"]
  events: [WashFactorsComputed]
  data: [forecast_value, demand_snapshot]
  rules: ["Sellable supply = physical rooms minus M03 OOO (OOS counted separately, per M65 occupancy definition).", "Wash factors derived from property history by segment and lead time; manual override needs reason and expiry.", "Group wash applied to unpicked block rooms only."]
  security: "Overrides audited."
  failure_cases: [insufficient_history_default_to_zero_wash, ooo_backlog_uncertain]
  finance_report_effect: "Feeds forecast, not ledgers."
  i18n_a11y: "Accessible tables."
  acceptance: "AC-SF53.1.2: Placing 5 rooms OOO reduces sellable supply by 5 for those dates on the next computation; an override without reason is rejected."
  dependency: "M03 OOO/OOS, M12 pickup."

- id: M53.F53.1.SF53.1.3
  name: segment/channel contribution after fees
  phase: 3
  release: R1
  actors: [revenue_manager, gm, owner]
  screens: [SCR-REV-channel-net-contribution]
  inputs: [period, segment, channel, commission, payment_fees, acquisition_cost]
  states: [estimated, reconciled]
  api: ["GET /v1/properties/{pid}/revenue/net-contribution?period"]
  events: [NetContributionComputed]
  data: [forecast_value, acquisition_cost_line (M51)]
  rules: ["Net room revenue = room revenue net of tax minus channel commission (M07), payment fees (M28) and attributable acquisition cost (M51).", "Values labelled estimate until M19 period close then reconciled.", "Definitions from M65 metric_definition 'net_contribution'."]
  security: "Aggregates; commission contracts visible to revenue and finance only."
  failure_cases: [commission_invoice_missing, fee_file_late]
  finance_report_effect: "Feeds M32 SF32.1.3 and owner profit bridge."
  i18n_a11y: "Currency-correct formatting."
  acceptance: "AC-SF53.1.3: For fixture OTA and direct bookings with identical ADR the report ranks direct higher after commission and shows estimate labels until period close."
  dependency: "M07, M28, M51, M19, M65."

- id: M53.F53.1.SF53.1.4
  name: holiday/event and optional licensed competitor inputs
  phase: 5
  release: R1
  actors: [revenue_manager, integration_admin]
  screens: [SCR-REV-demand-calendar, SCR-REV-market-inputs]
  inputs: [event_name, date_range, impact_estimate, source, license_ref, compset_rates_feed]
  states: [proposed, active, expired, license_blocked]
  api: ["POST /v1/properties/{pid}/revenue/demand-events", "POST /v1/integrations/market-data/{provider}/import"]
  events: [DemandEventAdded, MarketInputImported]
  data: [demand_event_calendar, market_input]
  rules: ["Competitor rates only from a licensed provider with contract terms stored; no scraping.", "Market inputs carry as-of timestamp and are ignored by the forecast after staleness limit.", "Public holidays per jurisdiction from M44 calendars where verified."]
  security: "License terms enforce display/retention limits; provider keys in vault."
  failure_cases: [feed_stale, license_expired_purge, holiday_calendar_unverified]
  finance_report_effect: "Data licence cost as AP expense to revenue cost center."
  i18n_a11y: "Calendar supports Gregorian storage with Hijri display for Arabic locale."
  acceptance: "AC-SF53.1.4: A market feed older than the staleness limit is excluded and the recommendation shows 'market input unavailable'; without a licence the import endpoint returns blocked."
  dependency: "INT-market-data (D-609), M44 holiday calendars."

- id: M53.F53.1.SF53.1.5
  name: confidence, data coverage and prior-year comparison
  phase: 5
  release: R1
  actors: [revenue_manager, gm, revenue_worker]
  screens: [SCR-REV-forecast, SCR-OWN-forecast-summary]
  inputs: [forecast_method_version, training_window, stay_date, horizon]
  states: [running, completed, low_confidence, failed]
  api: ["POST /v1/properties/{pid}/revenue/forecast-runs", "GET /v1/properties/{pid}/revenue/forecast-runs/{rid}"]
  events: [ForecastRunCompleted, ForecastLowConfidence]
  data: [forecast_run, forecast_value]
  rules: ["Every forecast value has a point estimate plus prediction interval and a data-coverage score (history length, missing days, snapshot completeness).", "Prior-year comparison aligns by day-of-week and flags calendar shifts.", "Forecast method and parameters are versioned; runs are reproducible from stored inputs.", "New hotel with under the minimum history shows 'insufficient history' rather than a confident number."]
  security: "Read restricted to revenue roles and management."
  failure_cases: [insufficient_history, run_timeout, data_gap]
  finance_report_effect: "Forecast feeds M32 budget/forecast views labelled forecast."
  i18n_a11y: "Intervals described in text for screen readers."
  acceptance: "AC-SF53.1.5: With 20 days of history the 90-day forecast is flagged low confidence; re-running a stored run ID reproduces identical values."
  dependency: "SF53.1.1, SF53.1.2, M65 lineage."

- id: M53.F53.1.SF53.1.6  # ADDED — Section C 'overbooking risk'
  name: Overbooking risk estimate within M03 limits
  phase: 5
  release: R1
  actors: [revenue_manager, front_office_manager]
  screens: [SCR-REV-overbooking-risk]
  inputs: [stay_date, forecast_wash, current_overbook_limit, walk_cost_estimate]
  states: [computed, proposed, approved, rejected]
  api: ["GET /v1/properties/{pid}/revenue/overbooking-risk", "POST /v1/properties/{pid}/revenue/overbooking-recommendations/{rid}/decision"]
  events: [OverbookingRecommendationDecided]
  data: [overbooking_recommendation, overbooking_limit (M03)]
  rules: ["Recommendation is advisory; the enforced overbooking limit remains an M03 setting changed only by an authorized human.", "Show probability of walk and expected walk cost; accessible rooms and VIP-flagged commitments are never included in overbooking capacity.", "Limit cannot exceed the M03 policy maximum."]
  security: "Approval by front_office_manager or gm."
  failure_cases: [low_confidence_no_recommendation]
  finance_report_effect: "Walk costs realized in M08/M20 are compared with estimates in backtest."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF53.1.6: Approving a recommendation above the M03 maximum is rejected; accessible room types are excluded from overbooking capacity."
  dependency: "M03 controlled overbooking, SF53.1.5."
```

### F53.2 Rate action

```yaml
- id: M53.F53.2.SF53.2.1
  name: candidate rate/length/stay/close-to-arrival rule with rationale
  phase: 5
  release: R1
  actors: [revenue_manager, revenue_worker]
  screens: [SCR-REV-recommendations]
  inputs: [stay_date, room_type, rate_plan, proposed_price, restriction_type, rationale_factors, forecast_run_id]
  states: [generated, under_review, accepted, rejected, expired]
  api: ["GET /v1/properties/{pid}/revenue/recommendations", "POST /v1/properties/{pid}/revenue/recommendations/{rid}/decision"]
  events: [RateRecommendationGenerated, RateRecommendationDecided]
  data: [rate_recommendation]
  rules: ["Each recommendation lists the drivers (pace vs curve, forecast, remaining supply, event, market input) with values and the forecast run ID.", "Recommendations never use guest personal attributes; prices are set per stay date/room type/rate plan, not per individual.", "Expired recommendations cannot be accepted.", "Corporate contracted rates (M10) are never recommended for change."]
  security: "Generated by system; decision by revenue_manager."
  failure_cases: [forecast_low_confidence_suppressed, rate_plan_locked_by_contract]
  finance_report_effect: "None until published."
  i18n_a11y: "Rationale readable text, not chart only."
  acceptance: "AC-SF53.2.1: Each generated recommendation contains at least one driver with value and a forecast run ID; recommendations for a contracted corporate rate are not generated."
  dependency: "SF53.1.5, M04, M10."

- id: M53.F53.2.SF53.2.2
  name: guardrails and approval based on role
  phase: 3
  release: R1
  actors: [revenue_manager, gm, owner]
  screens: [SCR-REV-guardrails, SCR-REV-approval-queue]
  inputs: [rate_plan, floor, ceiling, max_change_pct, approval_tier, effective_from]
  states: [draft, active, superseded]
  api: ["PUT /v1/properties/{pid}/revenue/guardrails", "POST /v1/properties/{pid}/revenue/rate-actions/{aid}/approve"]
  events: [GuardrailPolicyActivated, RateActionApproved, RateActionRejected]
  data: [guardrail_policy, rate_action]
  rules: ["Any rate action (manual or recommended) outside floor/ceiling or above max change % needs the next approval tier.", "Guardrail policy changes require gm or owner approval (maker-checker).", "Floor must be greater than zero and not below contracted corporate rate commitments where parity clauses exist."]
  security: "Step-up MFA for guardrail change; audit of every approval."
  failure_cases: [approver_unavailable_escalation, conflicting_policies]
  finance_report_effect: "None directly."
  i18n_a11y: "Accessible forms."
  acceptance: "AC-SF53.2.2: A 30% increase when max change is 15% routes to gm; revenue_manager self-approval above tier is rejected (AT-G19.4)."
  dependency: "M02 approvals, M63 escalation, D-610."

- id: M53.F53.2.SF53.2.3
  name: simulation against occupancy/net yield and corporate contract
  phase: 5
  release: R1
  actors: [revenue_manager]
  screens: [SCR-REV-simulator]
  inputs: [candidate_actions, forecast_run_id, elasticity_assumption, contract_constraints]
  states: [simulated]
  api: ["POST /v1/properties/{pid}/revenue/simulations"]
  events: [RateSimulationRun]
  data: [rate_action, forecast_value]
  rules: ["Simulation outputs projected occupancy, ADR, net contribution with intervals and the assumptions used; clearly labelled projection.", "Flags any corporate LRA/last-room-availability or parity conflict from M10 contracts."]
  security: "Read only; no side effects."
  failure_cases: [elasticity_unknown_default_flat]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF53.2.3: Simulating a close-to-arrival on a date with an LRA corporate contract shows a contract conflict warning."
  dependency: "M10 contracts, SF53.1.5."

- id: M53.F53.2.SF53.2.4
  name: publish to rate engine/channel with acknowledgment
  phase: 3
  release: R1
  actors: [revenue_manager, channel_worker]
  screens: [SCR-REV-publication-status, SCR-ADM-integration-health]
  inputs: [rate_action_id, idempotency_key, target_channels]
  states: [approved, applied_to_rate_engine, sent_to_channels, acknowledged, partially_acknowledged, failed]
  api: ["POST /v1/properties/{pid}/revenue/rate-actions/{aid}/publish"]
  events: [RateActionApplied, RatePublicationAcknowledged, RatePublicationFailed]
  data: [rate_action, rate_publication_ack, rate_plan_price (M04)]
  rules: ["M04 is updated first in one transaction; M07 pushes ARI via outbox; status is 'acknowledged' only on channel manager acknowledgment per channel.", "Unacknowledged after timeout creates a price_discrepancy_item.", "Idempotent: republish of the same action does not create a second version."]
  security: "Publish only approved actions; channel credentials in M07."
  failure_cases: [channel_timeout, partial_ack, mapping_missing]
  finance_report_effect: "Future revenue at new rates; no posting."
  i18n_a11y: "Status icons have text labels."
  acceptance: "AC-SF53.2.4: A published change shows per-channel acknowledgment; a simulated channel timeout produces a discrepancy item, not 'published' (AT-G19.4)."
  dependency: "M04, M07 SF (ARI/ack), INT-channel-manager."

- id: M53.F53.2.SF53.2.5
  name: versioned rollback and price-discrepancy queue
  phase: 3
  release: R1
  actors: [revenue_manager, integration_admin]
  screens: [SCR-REV-rate-history, SCR-OPS-price-discrepancies]
  inputs: [rate_action_id, rollback_to_version, discrepancy_id, resolution]
  states: [open, investigating, resolved, rolled_back]
  api: ["POST /v1/properties/{pid}/revenue/rate-actions/{aid}/rollback", "GET /v1/properties/{pid}/revenue/price-discrepancies"]
  events: [RateActionRolledBack, PriceDiscrepancyOpened, PriceDiscrepancyResolved]
  data: [rate_action, price_discrepancy_item]
  rules: ["Rollback creates a new rate_action restoring the prior version (append-only history) and re-publishes with acknowledgment.", "Existing reservations keep booked prices; rollback affects future sales only.", "Discrepancy detection compares website quote, M04 and channel-reported rates on sampled dates."]
  security: "Rollback subject to the same guardrail approvals."
  failure_cases: [rollback_during_publication, channel_rejects_rollback]
  finance_report_effect: "None on existing bookings."
  i18n_a11y: "Accessible history table."
  acceptance: "AC-SF53.2.5: Rolling back returns channel and website rates to the previous version with acknowledgment; confirmed bookings retain their prices (AT-G19.4 reversibility)."
  dependency: "SF53.2.4, M51 quotes."

- id: M53.F53.2.SF53.2.6
  name: backtest versus actual, with no asserted guaranteed uplift
  phase: 5
  release: R1
  actors: [revenue_manager, gm, owner]
  screens: [SCR-REV-backtest]
  inputs: [forecast_run_ids, period, metrics]
  states: [computed]
  api: ["GET /v1/properties/{pid}/revenue/backtests?period"]
  events: [BacktestComputed]
  data: [backtest_result]
  rules: ["Report forecast error (MAPE/bias) by horizon and recommendation acceptance outcomes versus actuals.", "No UI text, report or export may claim guaranteed or promised revenue uplift; comparisons are labelled observational unless a controlled test was run.", "Actuals from M19-closed periods only."]
  security: "Management read."
  failure_cases: [period_not_closed_provisional]
  finance_report_effect: "Uses M19 actuals."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF53.2.6: Backtest over fixture data reproduces known MAPE; a content lint over UI strings finds no 'guaranteed uplift' phrasing."
  dependency: "M19 close, M65 SF65.2.5."

- id: M53.F53.2.SF53.2.7  # ADDED — Section C 'M53 7 advanced automation'
  name: Bounded auto-apply of recommendations within guardrails
  phase: 7
  release: Later
  actors: [revenue_manager, gm, revenue_worker]
  screens: [SCR-REV-automation-settings]
  inputs: [auto_apply_scope, max_daily_changes, confidence_threshold, kill_switch]
  states: [disabled, shadow_mode, enabled, suspended]
  api: ["PUT /v1/properties/{pid}/revenue/automation"]
  events: [RateAutomationApplied, RateAutomationSuspended]
  data: [rate_action, guardrail_policy]
  rules: ["Must run in shadow mode with backtest evidence before enablement.", "Auto-applies only within guardrails and above confidence threshold; everything else stays human-approved.", "Kill switch immediately stops automation; all auto actions reversible via SF53.2.5."]
  security: "Enablement by gm plus owner approval."
  failure_cases: [anomaly_auto_suspend, channel_failure]
  finance_report_effect: "As SF53.2.4."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF53.2.7: With confidence below threshold no auto action occurs; kill switch halts within one cycle."
  dependency: "SF53.2.6 backtest evidence; Phase 7 gate."
```

### M53 key invariants

1. M04 is the only published-price SoR; M53 writes via approved `rate_action`s, append-only and reversible.
2. No price is "published" without channel acknowledgment; unacknowledged becomes a discrepancy.
3. Pricing never keys off individual guest personal attributes (no discriminatory automation).
4. No guaranteed-uplift claims; forecasts show confidence and coverage.

### M53 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF53.1.1, AC-SF53.1.5 | AT-G19 (90-day forecast/pickup) | Revenue: "Why is demand changing and how certain is the forecast? What is pickup to contract versus forecast?" |
| AC-SF53.2.2, AC-SF53.2.4, AC-SF53.2.5 | AT-G19 (guardrailed rate change, channel ack, reversibility) | Revenue: "Which dates and segments need a rate/restriction change?" |
| AC-SF53.1.3 | AT-G08, AT-G19 | Revenue: "Which OTA or campaign drives profitable stays?" |
| AC-SF53.1.6 | AT-G20 (concurrent bookings) | GM: "What is sold, blocked...? overbookings" |

### M53 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-609 | Licensed market/compset data provider (if any). | Revenue Manager | None; forecast from hotel evidence only; market input shown as unavailable. |
| D-610 | Guardrail floor/ceiling/max-change and approval tiers for pilot hotel. | GM + Owner | Max 15% change per action by revenue_manager; above requires gm; floors per rate plan set at go-live. |
| D-611 | Baseline forecast method (pickup-additive vs exponential smoothing) for Phase 5. | Lead Data Engineer | Additive pickup model plus seasonal naive benchmark; method versioned. |
| D-612 | Overbooking tolerance policy and walk procedure. | Front Office Manager | Zero overbooking until policy agreed; recommendations display only. |

---

## M54 — Offers, upsells, vouchers and packages

| Field | Value |
|---|---|
| Purpose | Sell ancillary value (upgrades, early/late, parking, dining, spa, transport, experiences, bundles, gift vouchers) only when capacity exists, at a correct total, with lawful targeting, correct revenue allocation and liability accounting. |
| Phases | 3 core ancillary offers and bundles; 4 voucher liability and accounting allocation; 5 reconciliation and optimization. |
| Release flag | R1. |
| Bounded context | `offers` |
| SoR entities (owned) | `offer_definition`, `offer_eligibility_rule`, `offer_presentation`, `ancillary_order`, `package_component`, `component_consumption`, `gift_voucher`, `voucher_ledger_entry`, `offer_performance_snapshot`. |
| Referenced (not owned) | M03 room inventory/upgrades; M04 package rate plans, promotions and taxes; M05 reservation; M06 room status; M08 folio; M09 timed resources; M13 POS; M17 parking; M58 amenities; M59 transport; M19 GL (liability, allocation); M28 payments; M44 promotional permit rules; M02 consent. |
| Dependencies | M03, M04, M05, M08, M09, M19, M28, M44, M58, M59; M52 messaging for targeted offers. |

### F54.1 Upsells

```yaml
- id: M54.F54.1.SF54.1.1
  name: eligible room upgrade and inventory check
  phase: 3
  release: R1
  actors: [guest, front_desk_agent, revenue_manager]
  screens: [SCR-WEB-extras, SCR-GST-upgrade-offer, SCR-STF-upgrade-at-desk]
  inputs: [reservation_id, from_room_type, to_room_type, stay_dates, price_per_night, attribute_basis, idempotency_key]
  states: [offered, accepted_pending_inventory, confirmed, declined, expired, failed_no_inventory]
  api: ["GET /v1/properties/{pid}/reservations/{rid}/upgrade-offers", "POST /v1/properties/{pid}/reservations/{rid}/upgrades"]
  events: [UpgradeOffered, UpgradeConfirmed, UpgradeFailed]
  data: [offer_definition, ancillary_order, room_type_night_stock (M03)]
  rules: ["Offer shown only if target room type has sellable stock for every night; acceptance atomically moves inventory in M03 (release source, consume target).", "Attribute-based upgrades (view, floor, balcony) require an assignable room with that attribute for all nights.", "Price comes from M04 upgrade rate; staff discount beyond limit needs M60 approval.", "Accessible rooms are not offered as upgrades to guests who did not request accessibility while accessible demand is pending."]
  security: "Guest can upgrade only own reservation (object-level authorization)."
  failure_cases: [inventory_race, price_changed, reservation_modified_concurrently]
  finance_report_effect: "Upgrade revenue posts as room revenue with upgrade analysis code to M08/M19."
  i18n_a11y: "Before/after comparison accessible; media from M39 with alt text."
  acceptance: "AC-SF54.1.1: Two guests accepting the last suite concurrently yield one confirmation and one failed_no_inventory with no charge (AT-G19.2, AT-G20.1)."
  dependency: "M03 atomic moves, M04 pricing, M05."

- id: M54.F54.1.SF54.1.2
  name: early arrival/late checkout bounded by cleaning and room sale
  phase: 3
  release: R1
  actors: [guest, front_desk_agent, housekeeping_supervisor]
  screens: [SCR-GST-early-late, SCR-STF-early-late-requests]
  inputs: [reservation_id, requested_time, room_id, next_arrival, cleaning_duration]
  states: [requested, feasible_offered, confirmed, waitlisted, declined, revoked_with_refund]
  api: ["POST /v1/properties/{pid}/reservations/{rid}/early-late-requests"]
  events: [EarlyLateConfirmed, EarlyLateRevoked]
  data: [ancillary_order, hk_task (M56)]
  rules: ["Feasibility checks M06/M56 cleaning ETA and any same-day arrival assigned to the room.", "Confirmed late checkout creates a timed hold on the room so it cannot be assigned to an earlier arrival.", "If later infeasible (room fault), revoke with full refund and notify guest."]
  security: "Guest own reservation only."
  failure_cases: [cleaning_delay, room_fault, overbooked_day]
  finance_report_effect: "Fee posts to M08 under ancillary room revenue; refunds reverse once."
  i18n_a11y: "Times shown in property time zone with explicit label."
  acceptance: "AC-SF54.1.2: Confirming a 15:00 late checkout prevents assignment of that room to a 13:00 arrival; revocation reverses the charge exactly once."
  dependency: "M06, M56 SF56.1.2, M08."

- id: M54.F54.1.SF54.1.3
  name: dining/parking/spa/transport offer and live capacity
  phase: 3
  release: R1
  actors: [guest, concierge, fnb_manager]
  screens: [SCR-GST-add-ons, SCR-WEB-extras]
  inputs: [offer_id, resource_type, slot, quantity, reservation_id]
  states: [available, held, booked, capacity_exhausted, cancelled]
  api: ["GET /v1/properties/{pid}/offers?reservation_id", "POST /v1/properties/{pid}/ancillary-orders"]
  events: [AncillaryOrderBooked, AncillaryOrderCancelled]
  data: [ancillary_order, timed_resource_hold (M09)]
  rules: ["Capacity is held in the owning module (M09 tables/spa slots, M17 parking, M59 trips); M54 never keeps its own counters.", "Offers for disabled amenities are not generated (M58 flag).", "Composite holds release on payment failure or expiry."]
  security: "Guest object scope; staff by department."
  failure_cases: [capacity_race, amenity_disabled, hold_expired]
  finance_report_effect: "Revenue posts to owning department's revenue account via M08."
  i18n_a11y: "Slot pickers accessible."
  acceptance: "AC-SF54.1.3: Selling the last parking pass via M54 decrements M17 capacity; a concurrent sale fails cleanly."
  dependency: "M09, M17, M58, M59."

- id: M54.F54.1.SF54.1.4
  name: targeted timing, expiry and consent
  phase: 3
  release: R1
  actors: [marketing_manager, revenue_manager, crm_worker]
  screens: [SCR-MKT-offer-targeting]
  inputs: [offer_id, trigger_moment, audience_rule, expiry, channel]
  states: [draft, approved, active, paused, expired]
  api: ["PUT /v1/properties/{pid}/offers/{oid}/targeting"]
  events: [OfferTargetingActivated]
  data: [offer_presentation, offer_eligibility_rule]
  rules: ["Pre-arrival offers inside the booking journey are service content; separate marketing sends require M02 consent via M52.", "Targeting criteria use the M52 allowlist; no pricing by protected characteristics.", "Every offer has an expiry shown to the guest."]
  security: "Maker-checker on activation."
  failure_cases: [consent_missing, expired_offer_clicked]
  finance_report_effect: "None until order."
  i18n_a11y: "Offer copy localized; expiry in property time zone."
  acceptance: "AC-SF54.1.4: A pre-arrival email offer is not sent to a guest without marketing consent, while the in-app offer tile still appears in the booking view."
  dependency: "M52 SF52.1.3, SF52.1.4."

- id: M54.F54.1.SF54.1.5
  name: booked price, cancellation/refund and folio/revenue allocation
  phase: 3
  release: R1
  actors: [guest, front_desk_agent, finance_clerk]
  screens: [SCR-GST-my-extras, SCR-FIN-ancillary-allocation]
  inputs: [ancillary_order_id, booked_price, tax_lines, cancellation_policy, refund_amount, idempotency_key]
  states: [booked, consumed, cancelled_free, cancelled_charged, refunded, no_show]
  api: ["POST /v1/properties/{pid}/ancillary-orders/{oid}/cancel", "POST /v1/properties/{pid}/ancillary-orders/{oid}/refund"]
  events: [AncillaryOrderConsumed, AncillaryOrderRefunded]
  data: [ancillary_order, folio_line (M08)]
  rules: ["Price and policy snapshot frozen at booking.", "Revenue recognized on consumption date; prepaid unconsumed amounts are deferred revenue.", "Refund reverses via M08/M28 exactly once with idempotency."]
  security: "Refund above limit requires M60 approval."
  failure_cases: [double_refund_attempt, psp_refund_pending]
  finance_report_effect: "Deferred revenue until consumption; department revenue mapping per SF19.1.2."
  i18n_a11y: "Receipts localized."
  acceptance: "AC-SF54.1.5: Refunding the same order twice produces one refund; revenue appears on consumption date not booking date."
  dependency: "M08, M19, M28, M60."

- id: M54.F54.1.SF54.1.6  # ADDED — Section C 'offer tracking'; Section N upsells benchmark
  name: Offer performance tracking
  phase: 5
  release: R1
  actors: [revenue_manager, marketing_manager, gm]
  screens: [SCR-REV-offer-performance]
  inputs: [offer_id, period]
  states: [computed]
  api: ["GET /v1/properties/{pid}/offers/performance?period"]
  events: [OfferPerformanceComputed]
  data: [offer_performance_snapshot]
  rules: ["Report presentations, acceptances, consumed, refunded and net revenue after cost of delivery.", "No claim of incremental uplift without holdout."]
  security: "Aggregates."
  failure_cases: [low_sample]
  finance_report_effect: "Consumes M19 revenue and M50 cost."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF54.1.6: Refunded upgrades are excluded from net revenue in the report."
  dependency: "SF54.1.5, M65."
```

### F54.2 Gift/package

```yaml
- id: M54.F54.2.SF54.2.1
  name: gift voucher issue/redemption/partial balance with accounting liability and fraud checks
  phase: 4
  release: R1
  actors: [guest, cashier, front_desk_agent, finance_clerk, voucher_worker]
  screens: [SCR-WEB-gift-voucher, SCR-STF-voucher-redeem, SCR-FIN-voucher-liability]
  inputs: [voucher_value, currency, purchaser, recipient_name, expiry, redemption_amount, voucher_code, idempotency_key]
  states: [issued, active, partially_redeemed, fully_redeemed, expired, voided, refunded]
  api: ["POST /v1/properties/{pid}/gift-vouchers", "POST /v1/properties/{pid}/gift-vouchers/{code}/redemptions", "GET /v1/properties/{pid}/gift-vouchers/{code}/balance"]
  events: [GiftVoucherIssued, GiftVoucherRedeemed, GiftVoucherExpired, GiftVoucherVoided]
  data: [gift_voucher, voucher_ledger_entry]
  rules: ["Balance = sum of append-only voucher_ledger_entry; no direct balance edits.", "Voucher codes high-entropy, checked with rate limiting; lockout after failed attempts.", "Redemption only on eligible hotel purchases; no cash-out unless required by jurisdiction rule pack.", "Expiry per M44 rule pack (some markets restrict expiry); unverified pack blocks sale."]
  security: "Code stored hashed; staff redemption requires authenticated session and shows partial code only."
  failure_cases: [brute_force_attempts, concurrent_redemption, rule_pack_unverified]
  finance_report_effect: "Issue: Dr cash/receivable, Cr voucher liability; redemption: Dr liability, Cr revenue; breakage per approved policy (D-613)."
  i18n_a11y: "Printable/PDF voucher accessible; bilingual."
  acceptance: "AC-SF54.2.1: Two simultaneous redemptions exceeding balance result in one success; liability report equals issued minus redeemed minus breakage."
  dependency: "M19, M28, M44 (D-613)."

- id: M54.F54.2.SF54.2.2
  name: bundle components and consumption dates
  phase: 3
  release: R1
  actors: [revenue_manager, front_desk_agent, fnb_manager]
  screens: [SCR-REV-package-builder, SCR-STF-package-entitlements]
  inputs: [package_rate_plan_id (M04), components, per_night_or_per_stay, consumption_date_rule]
  states: [draft, active, retired]
  api: ["PUT /v1/properties/{pid}/packages/{pkg}/components", "GET /v1/properties/{pid}/reservations/{rid}/entitlements"]
  events: [PackageComponentsDefined, EntitlementConsumed]
  data: [package_component, component_consumption]
  rules: ["The package price lives in M04; M54 defines components and entitlements only.", "Each entitlement is consumed once (breakfast day 2 cannot be used twice).", "Components requiring capacity create holds at booking via owning modules."]
  security: "Outlet staff see entitlements only for the guest in front of them."
  failure_cases: [double_consumption, component_unavailable]
  finance_report_effect: "Consumption triggers revenue allocation per SF54.2.3."
  i18n_a11y: "Entitlements listed in guest app."
  acceptance: "AC-SF54.2.2: Scanning the same breakfast entitlement twice rejects the second with reason."
  dependency: "M04, M13 POS lookup."

- id: M54.F54.2.SF54.2.3
  name: component tax, FX, commission and package allocation
  phase: 4
  release: R1
  actors: [finance_clerk, financial_controller]
  screens: [SCR-FIN-package-allocation]
  inputs: [package_price, component_standalone_prices, tax_codes, fx_rate, commission]
  states: [configured, approved]
  api: ["PUT /v1/properties/{pid}/packages/{pkg}/allocation"]
  events: [PackageAllocationApproved]
  data: [package_component]
  rules: ["Allocation by relative standalone selling price unless finance policy approves otherwise (D-614).", "Each component carries its own tax code from M44/M04; allocation sums exactly to package price with deterministic rounding.", "Commission on package allocated proportionally."]
  security: "Finance approval required."
  failure_cases: [rounding_residue, missing_tax_code]
  finance_report_effect: "Revenue per department (rooms, F&B, spa) correct in M19 and M32 departmental P&L."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF54.2.3: For a fixture package the allocated components sum to the price to the minor unit, including OMR 3 decimals."
  dependency: "M19, M44 tax, D-614."

- id: M54.F54.2.SF54.2.4
  name: blackout/expiry and jurisdiction review
  phase: 4
  release: R1
  actors: [revenue_manager, compliance_officer]
  screens: [SCR-REV-offer-compliance]
  inputs: [offer_id, blackout_dates, expiry, market, permit_ref]
  states: [pending_review, approved, blocked]
  api: ["POST /v1/properties/{pid}/offers/{oid}/compliance-review"]
  events: [OfferComplianceApproved, OfferComplianceBlocked]
  data: [offer_definition]
  rules: ["Promotions/discount offers in markets requiring a permit (e.g. Oman promotional offers permit) cannot activate without a recorded permit or counsel exemption.", "Blackouts shown before purchase."]
  security: "compliance_officer approval."
  failure_cases: [permit_expired]
  finance_report_effect: "None."
  i18n_a11y: "Terms bilingual."
  acceptance: "AC-SF54.2.4: An Oman fixture discount offer without permit reference stays blocked."
  dependency: "M44 rule packs, D-616."

- id: M54.F54.2.SF54.2.5
  name: transfer/refund/chargeback reconciliation
  phase: 5
  release: R1
  actors: [finance_clerk, guest]
  screens: [SCR-FIN-voucher-reconciliation]
  inputs: [voucher_code, transfer_to, refund_request, chargeback_ref]
  states: [transfer_requested, transferred, refund_pending, refunded, chargeback_open, chargeback_lost]
  api: ["POST /v1/properties/{pid}/gift-vouchers/{code}/transfer", "POST /v1/properties/{pid}/gift-vouchers/{code}/refund"]
  events: [GiftVoucherTransferred, GiftVoucherChargebackApplied]
  data: [voucher_ledger_entry]
  rules: ["Chargeback on purchase voids remaining balance; redeemed portion becomes receivable/loss case.", "Transfer allowed per terms; recorded without exposing purchaser data to recipient."]
  security: "Finance only."
  failure_cases: [chargeback_after_redemption]
  finance_report_effect: "Liability and loss postings to M19."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF54.2.5: A chargeback after partial redemption voids the remainder and creates one loss entry."
  dependency: "M28 chargebacks, M60 case."

- id: M54.F54.2.SF54.2.6  # ADDED — Section C 'promo eligibility, exclusions/capacity'
  name: Promo eligibility, exclusions and capacity caps
  phase: 3
  release: R1
  actors: [revenue_manager, guest]
  screens: [SCR-REV-promo-rules]
  inputs: [promo_code, eligibility_rules, stackability, max_redemptions, exclusions]
  states: [active, exhausted, expired]
  api: ["POST /v1/properties/{pid}/offers/promo-rules", "POST /v1/public/properties/{slug}/promo-validation"]
  events: [PromoRedemptionCounted]
  data: [offer_eligibility_rule]
  rules: ["Discount math is applied by M04 promotions; M54 decides eligibility, stacking and caps.", "Cap counters decremented atomically; cancellation returns capacity.", "Invalid code gives a generic message (no enumeration)."]
  security: "Rate-limited validation."
  failure_cases: [cap_race, stacking_conflict]
  finance_report_effect: "Discount cost reported per promo."
  i18n_a11y: "Error text accessible."
  acceptance: "AC-SF54.2.6: A cap of 10 accepts exactly 10 concurrent redemptions."
  dependency: "M04 promotions."
```

### M54 key invariants

1. Capacity for every ancillary lives in its owning module; M54 never double-sells (Section P.3).
2. Voucher balance is an append-only ledger; liability equals outstanding balance.
3. Package allocations sum exactly to price; revenue recognized on consumption.
4. Refunds/chargebacks reverse once.

### M54 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF54.1.1, AC-SF54.1.3 | AT-G19 (optional upgrade purchase, correct total), AT-G20 | Guest: "honest total price"; Front desk: "Can I move or extend the stay without double-selling?" |
| AC-SF54.1.5, AC-SF54.2.1, AC-SF54.2.3 | AT-G07, AT-G08 | Finance: "Are all charges, refunds... recognized once?" |
| AC-SF54.2.4 | AT-G09 | Technology/compliance: unverified rules |

### M54 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-613 | Voucher liability, breakage recognition and permitted expiry per market. | Financial Controller + counsel | No breakage recognized; minimum expiry per strictest known rule; sale blocked where rule pack unverified. |
| D-614 | Package revenue allocation method. | Financial Controller | Relative standalone selling price. |
| D-615 | Who may grant complimentary upgrades and limits. | Front Office Manager | Front office manager only, logged via M60 comp approval. |
| D-616 | Oman promotional-offer permit process and other markets' promotion rules. | Compliance Officer | Discount promotions blocked in Oman until permit recorded; other markets require counsel note. |

---

## M55 — Guest journey and service recovery

| Field | Value |
|---|---|
| Purpose | One omnichannel guest desk from inquiry through post-stay: every guest request or complaint has one owner, SLA, status and closure; recovery compensation is capped, approved and posted correctly; accessibility needs are escalated by humans, never scored by automation; optional key/kiosk hardware only through authorized adapters. |
| Phases | 2–3 guest desk (pre-arrival, needs routing, departure, outage fallback, inbox); 4 service recovery (compensation, reporting); 7 digital key/kiosk hardware. |
| Release flag | R1; `Later` for F55.3. |
| Bounded context | `guest-journey` |
| SoR entities (owned) | `guest_conversation`, `conversation_message`, `inquiry_lead`, `journey_task`, `guest_request`, `guest_case`, `case_action`, `compensation_grant`, `guest_confirmation`, `key_credential_request` (Later), `kiosk_session` (Later). |
| Referenced (not owned) | M18 guest app/profile (submission channel for `guest_request`, see D-617); M02 consent; M05 reservation/check-in; M06 room status; M08 folio reversal; M26 work orders; M40 AI assistant handoff; M41 ID/e-sign/OTP; M42 incidents; M52 surveys/follow-up; M54 voucher ledger; M56 housekeeping tasks; M63 work items/SLA; M64 outage mode. |
| Dependencies | M02, M05, M06, M08, M18, M26, M40, M41, M42, M54, M56, M63, M64; INT-email, INT-sms, INT-whatsapp, INT-telephony (optional), INT-smart-lock, INT-kiosk (Phase 7). |

### F55.1 Assistance

```yaml
- id: M55.F55.1.SF55.1.1
  name: unified phone/email/web/approved messaging inbox with consent
  phase: 3
  release: R1
  actors: [guest_relations, front_desk_agent, concierge, guest, ai_assistant]
  screens: [SCR-OPS-guest-inbox, SCR-STF-guest-inbox, SCR-GST-messages]
  inputs: [channel, sender_identifier, message_body, attachments, reservation_link, consent_ref, assignee]
  states: [new, assigned, awaiting_guest, awaiting_staff, resolved, closed, spam]
  api: ["GET /v1/properties/{pid}/inbox/conversations", "POST /v1/properties/{pid}/inbox/conversations/{cid}/messages", "POST /v1/integrations/messaging/{provider}/inbound"]
  events: [GuestMessageReceived, GuestConversationAssigned, GuestConversationResolved]
  data: [guest_conversation, conversation_message]
  rules: ["All channels thread into one conversation per guest/reservation where identity is verified; unverified senders get a separate thread until linked.", "Outbound messaging on WhatsApp/SMS only within consent and provider template/session rules; phone calls logged as notes (recording only if lawful and disclosed).", "M40 AI handoffs arrive with transcript and case context; AI identity disclosed to guest.", "Every conversation has one owner; unassigned beyond SLA escalates via M63."]
  security: "Staff see conversations for their property and department scope; attachments malware-scanned; PII redaction in previews."
  failure_cases: [provider_outage, duplicate_inbound_webhook, misrouted_identity, attachment_malware]
  finance_report_effect: "Messaging costs accrued to guest services cost center."
  i18n_a11y: "Bilingual UI; message composer RTL-aware; guest side accessible with screen readers; optional translation labelled."
  acceptance: "AC-SF55.1.1: A guest message via WhatsApp and a later email from the same verified guest appear in one thread; a replayed webhook creates one message; an unassigned thread escalates at SLA."
  dependency: "M02 consent, M40 handoff, M63 SLA, INT-messaging providers (D-619)."

- id: M55.F55.1.SF55.1.2
  name: pre-arrival instructions and accessible/assisted check-in
  phase: 2
  release: R1
  actors: [guest, front_desk_agent, front_office_manager]
  screens: [SCR-GST-pre-arrival, SCR-STF-arrival-checklist, SCR-OPS-arrivals]
  inputs: [reservation_id, eta, id_status, registration_status, payment_status, accessibility_needs, assisted_path_flag]
  states: [pre_arrival_sent, pre_checked_in, ready_for_arrival, blocked_needs_action, checked_in]
  api: ["GET /v1/properties/{pid}/reservations/{rid}/arrival-readiness", "POST /v1/properties/{pid}/reservations/{rid}/pre-arrival"]
  events: [PreArrivalCompleted, ArrivalReadinessChanged]
  data: [journey_task, reservation (M05)]
  rules: ["Readiness shows four independent statuses: identity (M41), registration signature (M41), payment/guarantee (M28/M08), room ready (M06); check-in allowed only when policy satisfied.", "Guest can choose assisted desk path and non-biometric ID path at any time.", "Pre-arrival message is transactional; sent per M02 channel preference."]
  security: "Pre-arrival link bound to reservation with OTP per M41; no ID images in message channels."
  failure_cases: [id_ocr_error_manual_correction, payment_declined, room_not_ready]
  finance_report_effect: "None directly."
  i18n_a11y: "Accessible forms, WCAG 2.2 AA, CAPTCHA-free; large-text and assisted options noted on arrival screen."
  acceptance: "AC-SF55.1.2: Arrival screen shows 'blocked: payment' when guarantee fails even if ID and room are ready; guest choosing assisted path is not forced through online ID capture (AT-G13.2, AT-G19.5)."
  dependency: "M41, M28, M06, M05."

- id: M55.F55.1.SF55.1.3
  name: room/amenity/diet/accessibility need routed to owning team
  phase: 2
  release: R1
  actors: [guest, front_desk_agent, housekeeping_supervisor, engineer, fnb_manager, executive_chef]
  screens: [SCR-GST-request, SCR-STF-requests, SCR-OPS-request-board]
  inputs: [request_type, room_id, reservation_id, detail, due_by, sensitivity_class]
  states: [submitted, routed, accepted, in_progress, done, cannot_fulfil]
  api: ["POST /v1/properties/{pid}/guest-requests", "PUT /v1/properties/{pid}/guest-requests/{req}/status"]
  events: [GuestRequestSubmitted, GuestRequestRouted, GuestRequestCompleted]
  data: [guest_request, work_item (M63)]
  rules: ["Routing table by request type: amenities -> M56, faults -> M26 work order, diet/allergen -> M57 kitchen acknowledgment, accessibility equipment -> front office.", "Diet/medical/accessibility details are shared only with the fulfilling team and only the needed fields.", "cannot_fulfil requires reason and guest notification."]
  security: "Sensitive request details encrypted and access-logged."
  failure_cases: [no_owner_for_type_fallback_to_duty_manager, team_offline_queue]
  finance_report_effect: "Chargeable requests post via owning module to M08."
  i18n_a11y: "Guest request picker with icons plus text; accessible."
  acceptance: "AC-SF55.1.3: A 'leaking tap' request creates one M26 work order; an allergy note reaches kitchen acknowledgment but is not visible to housekeeping."
  dependency: "M26, M56, M57, M63."

- id: M55.F55.1.SF55.1.4
  name: in-stay service, status, handoff and urgent escalation
  phase: 3
  release: R1
  actors: [guest, guest_relations, duty_manager, security_officer]
  screens: [SCR-GST-request-status, SCR-OPS-request-board]
  inputs: [request_id, status_update, handoff_to, urgency]
  states: [in_progress, handed_off, escalated, done]
  api: ["POST /v1/properties/{pid}/guest-requests/{req}/handoff", "POST /v1/properties/{pid}/guest-requests/{req}/escalate"]
  events: [GuestRequestHandedOff, GuestRequestEscalated]
  data: [guest_request, work_item (M63)]
  rules: ["Guest sees live status and ETA; staff handoff preserves history and requires receiving acknowledgment.", "Safety keywords or 'urgent' choices route immediately to duty_manager and, if safety-related, create an M42 incident; AI never handles safety alone.", "Shift change hands off open requests via M62 SF62.2.1."]
  security: "Guest sees own requests only."
  failure_cases: [handoff_not_acknowledged_escalation, duplicate_request]
  finance_report_effect: "None."
  i18n_a11y: "Status changes announced; push/SMS per preference."
  acceptance: "AC-SF55.1.4: A request marked 'feel unsafe' creates an M42 incident and pages duty_manager within one minute."
  dependency: "M42, M63, M62."

- id: M55.F55.1.SF55.1.5
  name: departure/receipt/follow-up
  phase: 2
  release: R1
  actors: [guest, front_desk_agent, cashier]
  screens: [SCR-GST-checkout, SCR-STF-departures]
  inputs: [reservation_id, folio_balance, express_checkout_consent, receipt_channel]
  states: [departure_due, express_requested, settled, checked_out, balance_dispute]
  api: ["POST /v1/properties/{pid}/reservations/{rid}/express-checkout"]
  events: [ExpressCheckoutRequested, ReceiptSent]
  data: [journey_task, folio (M08)]
  rules: ["Express checkout only when folio settled or authorized card on file covers balance; disputes open an M55 case and stop auto-charge.", "Receipt/invoice from M08 with jurisdiction format (M38/M44).", "Post-stay survey trigger delegated to M52 SF52.2.1."]
  security: "Receipt link authenticated and time-limited."
  failure_cases: [minibar_late_post, card_capture_failed, disputed_charge]
  finance_report_effect: "Settlement per M08."
  i18n_a11y: "Receipt accessible PDF and HTML; bilingual."
  acceptance: "AC-SF55.1.5: A guest disputing a charge during express checkout gets a case and no automatic capture of the disputed line."
  dependency: "M08, M28, M52."

- id: M55.F55.1.SF55.1.6
  name: outage-assisted fallback
  phase: 2
  release: R1
  actors: [front_desk_agent, duty_manager, it_admin]
  screens: [SCR-STF-offline-arrivals, SCR-OPS-outage-mode]
  inputs: [outage_mode_flag, cached_arrival_list, manual_registration_form_ref]
  states: [normal, degraded, offline, resync_pending, reconciled]
  api: ["POST /v1/properties/{pid}/outage-mode", "POST /v1/properties/{pid}/sync/replay"]
  events: [OutageModeEntered, OfflineActionsReplayed, OfflineConflictDetected]
  data: [journey_task, offline_action_log (M64)]
  rules: ["Staff app keeps a signed, encrypted cache of today's arrivals/departures and open requests.", "Offline check-in records registration on paper or device with later scan; no new online payments offline, only pre-authorized guarantees or cash receipt per M60.", "Replay on reconnect detects conflicts (room double-assigned) and routes to duty_manager."]
  security: "Cache expires; device must be enrolled (M64)."
  failure_cases: [cache_stale, replay_conflict, device_lost]
  finance_report_effect: "Offline cash receipts reconcile in M60 shift close."
  i18n_a11y: "Printable bilingual fallback forms."
  acceptance: "AC-SF55.1.6: In a simulated network outage two agents assign the same room offline; replay flags one conflict with no silent overwrite (AT-G20.5, AT-G14.2)."
  dependency: "M64 SF64.2.2, M06, M60."

- id: M55.F55.1.SF55.1.7  # ADDED — Section C 'inquiry-to-booking omnichannel inbox'; Section H.3 web-lead journey
  name: Inquiry-to-booking lead handling
  phase: 3
  release: R1
  actors: [guest_relations, sales_manager, front_desk_agent, guest]
  screens: [SCR-OPS-leads, SCR-OPS-guest-inbox]
  inputs: [conversation_id, stay_interest, dates, party, quote_id, follow_up_due]
  states: [new_lead, qualified, quoted, booked, lost, expired]
  api: ["POST /v1/properties/{pid}/inquiry-leads", "POST /v1/properties/{pid}/inquiry-leads/{lid}/quote"]
  events: [InquiryLeadCreated, InquiryLeadConverted, InquiryLeadLost]
  data: [inquiry_lead, quote (M04)]
  rules: ["Quotes are M04 quotes with expiry; conversion links reservation to the lead for attribution in M51.", "Lost reason mandatory; group/corporate inquiries route to M10/M12 pipeline instead."]
  security: "Lead data minimal; retention per M65."
  failure_cases: [quote_expired, duplicate_lead]
  finance_report_effect: "Conversion counts feed M51 funnel."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF55.1.7: A lead quoted and booked shows 'booked' and the reservation carries the lead attribution touch."
  dependency: "M04, M51, M10/M12."
```

### F55.2 Recovery

```yaml
- id: M55.F55.2.SF55.2.1
  name: complaint severity/case owner/SLA
  phase: 3
  release: R1
  actors: [guest_relations, duty_manager, front_office_manager, guest]
  screens: [SCR-OPS-guest-cases, SCR-STF-case-detail]
  inputs: [source, category, severity, description, reservation_id, owner, sla_policy]
  states: [open, acknowledged, in_recovery, awaiting_guest, resolved, closed, reopened]
  api: ["POST /v1/properties/{pid}/guest-cases", "PUT /v1/properties/{pid}/guest-cases/{case_id}"]
  events: [GuestCaseOpened, GuestCaseAcknowledged, GuestCaseSlaBreached]
  data: [guest_case, case_action]
  rules: ["Severity set by staff (AI may suggest with confidence); critical cases page duty_manager.", "One owner at all times; reassignment logged.", "SLA timers via M63 with acknowledgment and resolution targets per severity."]
  security: "Case notes scoped; guest sees status and public notes only."
  failure_cases: [owner_off_shift, duplicate_case_same_issue]
  finance_report_effect: "None directly."
  i18n_a11y: "Accessible case forms."
  acceptance: "AC-SF55.2.1: A room-service complaint opens a case with owner and SLA; breach escalates to front_office_manager (AT-G19.6)."
  dependency: "M63, M52 SF52.2.1 case creation."

- id: M55.F55.2.SF55.2.2
  name: housekeeping/maintenance incident link and guest privacy
  phase: 3
  release: R1
  actors: [guest_relations, housekeeping_supervisor, engineer]
  screens: [SCR-STF-case-detail, SCR-ENG-work-order]
  inputs: [case_id, linked_work_order_id, linked_hk_task_id, linked_incident_id]
  states: [linked, root_cause_recorded]
  api: ["POST /v1/properties/{pid}/guest-cases/{case_id}/links"]
  events: [GuestCaseLinked]
  data: [guest_case, work_order (M26), hk_task (M56), incident (M42)]
  rules: ["Operational teams see the task and room, not the guest complaint text unless needed.", "Case resolution waits for linked task completion or explicit decoupling with reason."]
  security: "Need-to-know field filtering."
  failure_cases: [linked_task_cancelled]
  finance_report_effect: "Repair costs attach to room/asset in M26."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF55.2.2: Engineer's work order shows fault and room but not the guest's name or complaint text."
  dependency: "M26, M56, M42."

- id: M55.F55.2.SF55.2.3
  name: compensation options with cap/approval, folio reversal or voucher ledger
  phase: 4
  release: R1
  actors: [guest_relations, duty_manager, gm, cashier]
  screens: [SCR-OPS-compensation, SCR-FIN-comp-report]
  inputs: [case_id, compensation_type, amount_or_points, reason, approver_id, idempotency_key]
  states: [proposed, pending_approval, approved, applied, rejected, reversed]
  api: ["POST /v1/properties/{pid}/guest-cases/{case_id}/compensations", "POST /v1/properties/{pid}/compensations/{cid}/approve"]
  events: [CompensationApproved, CompensationApplied]
  data: [compensation_grant, folio_line (M08), voucher_ledger_entry (M54), points_ledger_entry (M30)]
  rules: ["Options: folio allowance (M08 reversal/adjustment), M54 voucher, M30 points, complimentary service; no cash payouts outside M08 refund flow.", "Caps per role and per case; above cap requires next approver; approver distinct from requester.", "Applied exactly once (idempotent)."]
  security: "Step-up for high-value compensation; audited in M60."
  failure_cases: [folio_closed_use_refund_flow, approval_timeout]
  finance_report_effect: "Allowances reduce department revenue with reason code; vouchers create liability; cost of recovery reported per case."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF55.2.3: A 60 OMR allowance with a 50 cap for guest_relations routes to duty_manager; applied once to folio as reversal line (AT-G19.6)."
  dependency: "M08, M54, M30, M60, D-618."

- id: M55.F55.2.SF55.2.4
  name: guest confirmation and reopen
  phase: 4
  release: R1
  actors: [guest, guest_relations]
  screens: [SCR-GST-case-status, SCR-OPS-guest-cases]
  inputs: [case_id, guest_response, reopen_reason]
  states: [resolution_proposed, guest_confirmed, guest_rejected, reopened, closed]
  api: ["POST /v1/public/guest-cases/{token}/confirmation"]
  events: [GuestCaseConfirmed, GuestCaseReopened]
  data: [guest_case, guest_confirmation]
  rules: ["Guest can confirm or reject resolution; rejection reopens with same owner.", "Reopen allowed within configured window after closure."]
  security: "Token-bound to case."
  failure_cases: [token_expired]
  finance_report_effect: "None."
  i18n_a11y: "Accessible one-tap confirm."
  acceptance: "AC-SF55.2.4: Guest rejection reopens the case and restarts SLA."
  dependency: "M52 SF52.2.4."

- id: M55.F55.2.SF55.2.5
  name: repeat issue and recovery cost/outcome report
  phase: 4
  release: R1
  actors: [gm, front_office_manager, guest_relations]
  screens: [SCR-OPS-recovery-report, SCR-OWN-service-quality]
  inputs: [period, category, room_id, asset_id]
  states: [computed]
  api: ["GET /v1/properties/{pid}/guest-cases/reports?period"]
  events: [RecoveryReportComputed]
  data: [guest_case, compensation_grant]
  rules: ["Report time-to-acknowledge, time-to-resolve, guest-confirmed rate, recovery cost, repeat issues by room/asset.", "Metrics per M65 definitions."]
  security: "Aggregates; drill by role."
  failure_cases: [none_blocking]
  finance_report_effect: "Recovery cost by department to M32."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF55.2.5: Fixture with three cases on one room flags it as repeat issue and computes guest recovery time (AT-G19.7)."
  dependency: "M65, M32."

- id: M55.F55.2.SF55.2.6
  name: vulnerable-guest/accessibility escalation without discriminatory automation
  phase: 4
  release: R1
  actors: [duty_manager, guest_relations, front_office_manager]
  screens: [SCR-OPS-welfare-escalations]
  inputs: [case_id, welfare_flag_by_staff, guest_stated_need, escalation_level]
  states: [flagged, reviewed, supported, closed]
  api: ["POST /v1/properties/{pid}/guest-cases/{case_id}/welfare-escalation"]
  events: [WelfareEscalationRaised]
  data: [guest_case]
  rules: ["Welfare/vulnerability flags are raised by staff or the guest's own stated need, never inferred by algorithms from age, nationality, disability, gender or other protected traits.", "Automation may prioritize by stated accessibility need and safety keywords only; it may not deprioritize or deny service.", "Flag expires at checkout unless the guest asks to keep the preference."]
  security: "Highly restricted field; access logged; excluded from marketing and analytics exports."
  failure_cases: [flag_misuse_audit]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF55.2.6: No rule builder in M63/M55 allows conditions on protected attributes; the welfare flag is absent from segment and report exports."
  dependency: "M63 SF63.2.3, M52 SF52.1.4."
```

### F55.3 Self-service hardware (optional authorized adapters) — ADDED feature

```yaml
- id: M55.F55.3.SF55.3.1  # ADDED — Section C 'digital key ... optional authorized adapters', Phase 7
  name: Digital key issue/revoke via authorized lock adapter
  phase: 7
  release: Later
  actors: [guest, front_desk_agent, it_admin, lock_worker]
  screens: [SCR-GST-digital-key, SCR-STF-key-status]
  inputs: [reservation_id, room_id, validity_window, device_binding, lock_vendor_id]
  states: [not_eligible, requested, issued, revoked, failed_fallback_physical]
  api: ["POST /v1/properties/{pid}/reservations/{rid}/digital-keys", "DELETE /v1/properties/{pid}/digital-keys/{kid}"]
  events: [DigitalKeyIssued, DigitalKeyRevoked]
  data: [key_credential_request]
  rules: ["Available only with a certified lock vendor adapter; otherwise feature hidden.", "Issue only after check-in readiness; revoke on checkout/room move via M05 events.", "Physical key fallback always available."]
  security: "Credentials handled by lock vendor SDK; no key material stored in PMS."
  failure_cases: [adapter_down, revoke_failed_escalate_security]
  finance_report_effect: "None."
  i18n_a11y: "Accessible alternative to phone-based keys."
  acceptance: "AC-SF55.3.1: Room move revokes the old key and issues a new one; failed revoke raises a security task."
  dependency: "INT-smart-lock certification (D-620), M64."

- id: M55.F55.3.SF55.3.2  # ADDED — Section C 'kiosk optional authorized adapters', Phase 7
  name: Self-service kiosk check-in/out via authorized adapter
  phase: 7
  release: Later
  actors: [guest, front_desk_agent, kiosk_worker]
  screens: [SCR-KSK-check-in, SCR-STF-kiosk-monitor]
  inputs: [reservation_lookup, id_capture_ref, payment_terminal_ref, key_encoder_ref]
  states: [idle, in_session, handed_to_staff, completed, out_of_service]
  api: ["POST /v1/properties/{pid}/kiosk-sessions"]
  events: [KioskSessionCompleted, KioskHandedToStaff]
  data: [kiosk_session]
  rules: ["Same readiness rules as SF55.1.2; any exception hands to staff.", "Kiosk accessible height/screen-reader or staff-assisted alternative."]
  security: "Kiosk device enrolled in M64; session timeout and data wipe."
  failure_cases: [terminal_fail, key_encoder_fail]
  finance_report_effect: "Payments via M28 terminal."
  i18n_a11y: "Bilingual, accessible mode."
  acceptance: "AC-SF55.3.2: Failed payment at kiosk hands off to staff with context and no duplicate charge."
  dependency: "INT-kiosk (D-620), M28, M41."
```

### M55 key invariants

1. Every guest request/case has exactly one owner and an SLA timer; nothing is closed without resolution evidence.
2. Compensation applies once, within caps, via M08/M54/M30 ledgers only.
3. Safety and welfare escalate to humans; no automation keyed to protected attributes.
4. Guest can always choose assisted/manual paths; outage mode never double-assigns silently.

### M55 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF55.1.2, AC-SF55.1.6 | AT-G13, AT-G14, AT-G19, AT-G20 | Front desk: "Can this guest check in now...? What if internet is down?"; GM: "Can we operate if internet or a provider fails?" |
| AC-SF55.1.3, AC-SF55.1.4 | AT-G19 (in-stay) | GM: "Which arrivals... accessibility requests... need action now? Who owns every unresolved exception?" |
| AC-SF55.2.1, AC-SF55.2.3, AC-SF55.2.5 | AT-G19 (complaint, supervised recovery, guest recovery time) | Marketing/guest relations: "What service recovery actually closed the case?" |
| AC-SF55.2.6 | AT-G19 | Guest: "reach a human" |

### M55 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-617 | `guest_request` SoR: M55 (this file) vs M18 catalogue. | Lead Architect | M55 owns `guest_request`; M18 guest app/web is a channel that creates it through the M55 API. |
| D-618 | Compensation caps per role and case. | GM | guest_relations 50 OMR-equivalent, duty_manager 200, above -> gm. |
| D-619 | Telephony/voice integration and call-recording lawfulness. | IT Admin + DPO | No recording; calls logged manually as notes. |
| D-620 | Digital key and kiosk vendors (Phase 7). | GM + IT Admin | Physical keys only in R1; features hidden. |

---

## M56 — Housekeeping, laundry, linen and minibar

| Field | Value |
|---|---|
| Purpose | Prioritized room turnaround with inspection and release gate, offline-safe mobile work, and custody of linen/amenities/minibar with a single posting to stock ledger and folio. |
| Phases | 2 room board, priority, inspection, release gate, offline; 3 linen custody, outsourced laundry, minibar; 4 damage/loss, reorder/cost allocation, disputes. |
| Release flag | R1. |
| Bounded context | `housekeeping` |
| SoR entities (owned) | `hk_task`, `hk_assignment`, `hk_inspection`, `hk_duration_standard`, `deep_clean_schedule`, `linen_par_policy`, `laundry_batch`, `laundry_batch_line`, `linen_damage_record`, `minibar_count`, `amenity_par_policy`. |
| Referenced (not owned) | M06 `room_cleaning_status` state machine (see D-621); M03 room/OOO; M05 arrivals/departures/DND flags; M14 SKU master (linen, amenities, minibar items); M50 `stock_ledger_entry` (all quantity movements); M08 folio (minibar/damage charges); M26 room faults; M43 found items; M46/M49 laundry vendor contracts/PO; M20 AP; M63 tasks; M64 offline sync. |
| Dependencies | M03, M05, M06, M08, M14, M26, M43, M46, M49, M50, M63, M64. |

### F56.1 Room turnaround

```yaml
- id: M56.F56.1.SF56.1.1
  name: priority from arrival/departure/VIP and DND
  phase: 2
  release: R1
  actors: [housekeeping_supervisor, housekeeper, front_office_manager]
  screens: [SCR-HK-room-board, SCR-STF-my-rooms]
  inputs: [room_id, departure_status, arrival_eta, vip_flag, dnd_flag, early_checkin_flag]
  states: [not_due, due, priority, dnd_hold, in_progress, done]
  api: ["GET /v1/properties/{pid}/housekeeping/board?date", "PUT /v1/properties/{pid}/rooms/{room_id}/dnd"]
  events: [HkPriorityRecalculated, RoomDndSet, RoomDndCleared]
  data: [hk_task, room_cleaning_status (M06)]
  rules: ["Priority order: confirmed early check-in/VIP arrivals with ETA, then due-out arrivals, then stayovers; recomputed on M05 events.", "DND respected; after configured DND duration a welfare check task goes to supervisor (not forced entry by housekeeper).", "Accessible-room arrivals prioritized when assigned."]
  security: "Housekeepers see room numbers and task, not guest names unless VIP handling requires."
  failure_cases: [eta_unknown, dnd_extended_welfare_check]
  finance_report_effect: "None."
  i18n_a11y: "Board color plus text/icon labels; staff app bilingual and large-touch."
  acceptance: "AC-SF56.1.1: A VIP arrival with ETA 11:00 ranks above standard stayovers; DND over threshold creates a supervisor welfare check task (AT-G19.5)."
  dependency: "M05 events, M06 status, M63."

- id: M56.F56.1.SF56.1.2
  name: task durations and assignment capacity
  phase: 2
  release: R1
  actors: [housekeeping_supervisor, housekeeper]
  screens: [SCR-HK-assignment, SCR-STF-my-rooms]
  inputs: [task_type, room_type, standard_minutes, attendant_shift, credits_capacity]
  states: [unassigned, assigned, started, paused, completed]
  api: ["POST /v1/properties/{pid}/housekeeping/assignments", "PUT /v1/properties/{pid}/housekeeping/tasks/{tid}/status"]
  events: [HkTaskAssigned, HkTaskStarted, HkTaskCompleted]
  data: [hk_task, hk_assignment, hk_duration_standard]
  rules: ["Assignment respects shift length from M27 roster and standard durations; over-capacity warns supervisor.", "Room-ready ETA = queue position x durations; shown to front desk.", "Task time captured for labor metrics, not individual surveillance beyond shift reporting (M62 privacy)."]
  security: "Housekeeper updates own tasks only."
  failure_cases: [attendant_absent_reassign, duration_missing_default]
  finance_report_effect: "Labor minutes per occupied room feeds M32 rooms department labor cost."
  i18n_a11y: "Offline-capable mobile, bilingual."
  acceptance: "AC-SF56.1.2: Assigning 20 checkouts to an 8-hour shift with 30-minute standards warns over capacity; front desk sees ETA."
  dependency: "M27 roster, M06."

- id: M56.F56.1.SF56.1.3
  name: inspection/reclean
  phase: 2
  release: R1
  actors: [housekeeping_supervisor, housekeeper]
  screens: [SCR-HK-inspection, SCR-STF-inspection-checklist]
  inputs: [room_id, checklist_version, item_results, photos, pass_fail]
  states: [awaiting_inspection, passed, failed_reclean, reinspection]
  api: ["POST /v1/properties/{pid}/housekeeping/inspections"]
  events: [RoomInspectionPassed, RoomInspectionFailed]
  data: [hk_inspection, inspection_template (M61)]
  rules: ["Checklist from M61 template versioned; failure creates reclean task to same or other attendant.", "Inspector cannot inspect a room they cleaned when policy requires separation.", "Only passed inspection (or configured skip for stayovers) sets M06 status to inspected/ready."]
  security: "Inspection by supervisor role."
  failure_cases: [photo_upload_offline_queue]
  finance_report_effect: "None."
  i18n_a11y: "Checklist accessible with large controls."
  acceptance: "AC-SF56.1.3: A failed inspection keeps room not-ready and creates a reclean task; self-inspection is blocked when separation is on."
  dependency: "M61 templates, M06."

- id: M56.F56.1.SF56.1.4
  name: lost/found, room fault and release gate
  phase: 2
  release: R1
  actors: [housekeeper, housekeeping_supervisor, engineer, front_office_manager]
  screens: [SCR-STF-report-found-item, SCR-STF-report-fault, SCR-HK-release-gate]
  inputs: [room_id, found_item_details, fault_type, severity, photos]
  states: [ready_blocked_fault, ready_blocked_inspection, released]
  api: ["POST /v1/properties/{pid}/housekeeping/rooms/{room_id}/fault", "POST /v1/properties/{pid}/lost-found/items"]
  events: [RoomFaultReported, RoomReleased]
  data: [hk_task, found_item (M43), work_order (M26)]
  rules: ["Found items handed to M43 intake immediately with custody start; housekeeping keeps no parallel record.", "A severity-blocking fault creates an M26 work order and sets M03 OOO/OOS per policy; room cannot be released until M26 closes and inspection passes.", "Release gate conditions: cleaned, inspected, no blocking fault, no active M61 hold."]
  security: "Found-item photos visible to M43 custody roles only."
  failure_cases: [fault_closed_without_inspection, m43_intake_offline]
  finance_report_effect: "OOO nights excluded from sellable supply per M65 definition."
  i18n_a11y: "Photo capture with text description alternative."
  acceptance: "AC-SF56.1.4: A room with an open blocking work order cannot be marked ready; a found wallet creates one M43 item with custody chain (AT-G14.3)."
  dependency: "M26, M43, M03, M61."

- id: M56.F56.1.SF56.1.5
  name: offline conflict/audit and room-ready ETA
  phase: 2
  release: R1
  actors: [housekeeper, housekeeping_supervisor, front_desk_agent]
  screens: [SCR-STF-sync-status, SCR-HK-conflicts]
  inputs: [device_id, offline_actions, client_timestamps, server_version]
  states: [synced, pending_sync, conflict, resolved]
  api: ["POST /v1/properties/{pid}/sync/housekeeping"]
  events: [HkOfflineConflictDetected, HkOfflineActionsApplied]
  data: [hk_task, offline_action_log (M64)]
  rules: ["Offline actions carry client timestamp and base version; server applies if no conflicting change; otherwise conflict queue.", "A room already assigned to a guest cannot be reverted to dirty silently by stale offline data.", "Front desk sees 'last synced' age on room status."]
  security: "Enrolled devices only; action log immutable."
  failure_cases: [clock_skew, device_lost_mid_shift]
  finance_report_effect: "None."
  i18n_a11y: "Sync state announced accessibly."
  acceptance: "AC-SF56.1.5: A stale offline 'dirty' update after the room was inspected and checked-in goes to conflict queue, not applied (AT-G20.5)."
  dependency: "M64 SF64.2.2."

- id: M56.F56.1.SF56.1.6  # ADDED — Section C 'cleaning schedules'
  name: Periodic deep-clean and rotation schedules
  phase: 3
  release: R1
  actors: [housekeeping_supervisor, housekeeper]
  screens: [SCR-HK-deep-clean-plan]
  inputs: [room_id, task_type, frequency, last_done, blackout_on_occupancy]
  states: [scheduled, due, done, overdue]
  api: ["POST /v1/properties/{pid}/housekeeping/deep-clean-schedules"]
  events: [DeepCleanDue, DeepCleanOverdue]
  data: [deep_clean_schedule, hk_task]
  rules: ["Deep cleans scheduled on vacant nights using M03 forecast occupancy; if requiring room off sale, create OOS hold via M03.", "Overdue beyond tolerance escalates via M63."]
  security: "Supervisor role."
  failure_cases: [no_vacant_window]
  finance_report_effect: "Chemicals issued via M50 to housekeeping cost center."
  i18n_a11y: "Accessible calendar."
  acceptance: "AC-SF56.1.6: A deep clean due on a fully booked week is proposed on the next vacant night and escalates if overdue."
  dependency: "M03, M63."
```

### F56.2 Linen and minibar

```yaml
- id: M56.F56.2.SF56.2.1
  name: clean/soiled linen SKU, par and custody by room/floor/vendor
  phase: 3
  release: R1
  actors: [laundry_attendant, housekeeping_supervisor, storekeeper]
  screens: [SCR-HK-linen-par, SCR-STF-linen-move]
  inputs: [sku_id (M14), location_id, condition, quantity, par_level, custody_holder]
  states: [clean_in_store, clean_on_floor, in_use, soiled, at_laundry, returned_clean, rejected, lost]
  api: ["PUT /v1/properties/{pid}/housekeeping/linen-par", "POST /v1/properties/{pid}/stock/movements"]
  events: [LinenMoved, LinenParBreached]
  data: [linen_par_policy, stock_ledger_entry (M50), stock_location (M50)]
  rules: ["All linen quantity movements are M50 stock_ledger_entries between locations (store, floor pantry, laundry vendor, in-use, rejected); M56 owns par policies only.", "Par check = clean available vs forecast arrivals x par per room; shortfall alerts supervisor before arrivals.", "Custody handoffs require receiving party acknowledgment."]
  security: "Scoped to housekeeping/laundry/store roles."
  failure_cases: [negative_balance_blocked, par_shortfall]
  finance_report_effect: "Linen in circulation valued per M19 policy; losses expensed via SF56.2.3."
  i18n_a11y: "Barcode/RFID optional; manual count accessible."
  acceptance: "AC-SF56.2.1: Forecast 80 arrivals with par 3 and 200 clean sheets triggers a shortfall alert; no location can go negative (AT-G19.5)."
  dependency: "M14 SKUs, M50 ledger, M53 forecast (arrivals)."

- id: M56.F56.2.SF56.2.2
  name: outsourced laundry pickup/weight/return/mismatch
  phase: 3
  release: R1
  actors: [laundry_attendant, vendor_user, housekeeping_supervisor, ap_clerk]
  screens: [SCR-HK-laundry-batches, SCR-VND-laundry-batch]
  inputs: [laundry_vendor_id (M46), po_or_contract_ref (M49), pickup_counts_by_sku, pickup_weight_kg, return_counts, return_weight_kg]
  states: [prepared, picked_up, returned_partial, returned_complete, mismatch, closed]
  api: ["POST /v1/properties/{pid}/laundry-batches", "POST /v1/properties/{pid}/laundry-batches/{bid}/returns"]
  events: [LaundryBatchPickedUp, LaundryBatchReturned, LaundryMismatchDetected]
  data: [laundry_batch, laundry_batch_line, stock_ledger_entry (M50)]
  rules: ["Pickup and return each signed by hotel and vendor; counts by SKU plus weight when billed by weight.", "Return quantity differences beyond tolerance create a mismatch case with the vendor and hold invoice matching.", "Invoice matched in M49/M20 against returned accepted quantities, not pickup."]
  security: "Vendor sees only own batches."
  failure_cases: [vendor_count_dispute, weight_scale_unavailable_manual]
  finance_report_effect: "Laundry cost per occupied room to rooms department in M19/M32."
  i18n_a11y: "Vendor app bilingual."
  acceptance: "AC-SF56.2.2: A batch of 100 sheets returning 96 opens a mismatch and the invoice for 100 fails the match (AT-G19.8)."
  dependency: "M46 vendor, M49 PO/contract, M50, M20."

- id: M56.F56.2.SF56.2.3
  name: damage/loss and replacement cost
  phase: 4
  release: R1
  actors: [housekeeping_supervisor, front_office_manager, finance_clerk]
  screens: [SCR-HK-damage-loss]
  inputs: [sku_id, quantity, cause_code, room_id, reservation_id, photos]
  states: [reported, assessed, charged_to_guest, claimed_from_vendor, written_off]
  api: ["POST /v1/properties/{pid}/housekeeping/linen-damage"]
  events: [LinenDamageRecorded, LinenWrittenOff]
  data: [linen_damage_record, stock_ledger_entry (M50)]
  rules: ["cause_code is stain, tear, missing, guest_damage or vendor_damage.", "Write-off moves stock to waste via M50 with reason and photo.", "Guest charge only per disclosed policy and with manager approval; vendor damage creates vendor claim.", "Stained items to rewash are a location move, not a write-off."]
  security: "Guest charges require front_office_manager approval."
  failure_cases: [guest_dispute]
  finance_report_effect: "Write-off to linen replacement expense; guest charge to other income via M08."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF56.2.3: A written-off towel reduces M50 in-circulation stock and posts replacement expense once."
  dependency: "M50, M08, M49 claims."

- id: M56.F56.2.SF56.2.4
  name: minibar and amenity stocking/count/consumption with one-time folio post
  phase: 3
  release: R1
  actors: [housekeeper, housekeeping_supervisor, front_desk_agent]
  screens: [SCR-STF-minibar-count, SCR-HK-minibar-exceptions]
  inputs: [room_id, reservation_id, item_counts, count_time, idempotency_key]
  states: [counted, posted, not_chargeable, disputed, reversed]
  api: ["POST /v1/properties/{pid}/minibar-counts"]
  events: [MinibarConsumptionPosted, MinibarCountRecorded]
  data: [minibar_count, folio_line (M08), stock_ledger_entry (M50)]
  rules: ["Consumption = previous restock count minus current count; posted once per count with idempotency; restock posts M50 issue from store to room location.", "Counts after checkout post to the departed folio if within late-charge window, else to a late-charge case.", "Complimentary amenities issue stock but post no folio charge."]
  security: "Housekeeper creates counts; price from M13/M04 item price list."
  failure_cases: [duplicate_count_submission, late_charge_after_checkout, offline_count_replay]
  finance_report_effect: "Minibar revenue to F&B (or rooms per mapping) and COGS via M50."
  i18n_a11y: "Item images with text labels; bilingual."
  acceptance: "AC-SF56.2.4: Submitting the same count twice (offline replay) posts one folio charge; restock reduces store stock once."
  dependency: "M08, M50, M13 price list."

- id: M56.F56.2.SF56.2.5
  name: low-stock reorder and cost allocation
  phase: 4
  release: R1
  actors: [storekeeper, housekeeping_supervisor, procurement_officer]
  screens: [SCR-HK-amenity-par, SCR-OPS-reorder-suggestions]
  inputs: [sku_id, amenity_par_policy, on_hand, forecast_usage]
  states: [ok, below_reorder, requisition_created]
  api: ["GET /v1/properties/{pid}/housekeeping/reorder-suggestions"]
  events: [HkReorderSuggested]
  data: [amenity_par_policy, requisition (M49)]
  rules: ["Suggestions create M49 requisitions; M56 does not create POs.", "Consumables issued to housekeeping cost center per room/night driver."]
  security: "Requisition approval per M49."
  failure_cases: [vendor_unavailable]
  finance_report_effect: "Consumables cost per occupied room in M32."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF56.2.5: Crossing reorder point creates exactly one draft requisition."
  dependency: "M49, M50."

- id: M56.F56.2.SF56.2.6
  name: disputed charge/stock variance
  phase: 4
  release: R1
  actors: [front_office_manager, housekeeping_supervisor, storekeeper]
  screens: [SCR-HK-variance, SCR-OPS-guest-cases]
  inputs: [minibar_count_id, dispute_reason, stocktake_result]
  states: [disputed, reversed, upheld, variance_investigated]
  api: ["POST /v1/properties/{pid}/minibar-counts/{mid}/dispute"]
  events: [MinibarChargeReversed, StockVarianceRecorded]
  data: [minibar_count, folio_line (M08), stock_ledger_entry (M50)]
  rules: ["Upheld dispute reverses folio line via M08 reversal; stock variance recorded in M50 as shrinkage.", "Repeated variance by room/attendant goes to M60 as anomaly, with privacy safeguards."]
  security: "Approval by front_office_manager."
  failure_cases: [already_settled_refund_path]
  finance_report_effect: "Reversal reduces revenue; shrinkage to COGS variance."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF56.2.6: Reversing a disputed minibar charge reverses once and records a stock variance entry."
  dependency: "M08, M50, M60."
```

### M56 key invariants

1. Room readiness is the single M06 status; M56 transitions it only via inspection/release gate.
2. Every linen/amenity/minibar quantity movement is one M50 ledger entry; no negative balances.
3. Minibar consumption posts to folio exactly once per count.
4. Found items go to M43 custody immediately.

### M56 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF56.1.1, AC-SF56.1.2, AC-SF56.1.3 | AT-G19 (linen/housekeeping in the stay) | Housekeeping: "What rooms are due, prioritized or DND? Which tasks or inspections failed and what is the ETA?" |
| AC-SF56.2.1, AC-SF56.2.2, AC-SF56.2.3 | AT-G19, AT-G08 (laundry cost drill) | Housekeeping: "Do we have enough clean linen...? Where did linen go, what was lost or stained, and who accepted its return?" |
| AC-SF56.2.4, AC-SF56.2.6 | AT-G20 (duplicates) | Housekeeping: "Has minibar consumption been posted once?" |
| AC-SF56.1.4, AC-SF56.1.5 | AT-G14, AT-G20 | GM: "What is ... clean, in service or out of order?" |

### M56 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-621 | M06 vs M56 boundary for housekeeping tasks. | Lead Architect | M06 owns `room_cleaning_status`; M56 owns tasks, assignments, inspections, linen/minibar. |
| D-622 | Pilot laundry model (in-house, outsourced by piece or by weight). | Housekeeping Supervisor + Procurement | Outsourced, billed per piece with weight recorded. |
| D-623 | Minibar model (manual count, sensor, none). | F&B Manager | Manual count; sensor minibars out of R1 scope. |
| D-624 | Linen par levels and RFID usage. | Housekeeping Supervisor | Par 3 per bed position, barcode on bundles, no RFID. |

---

## M57 — Restaurant, room service and food assurance

| Field | Value |
|---|---|
| Purpose | Guest meal operations (table reservations, queue, in-room dining with kitchen promise and delivery confirmation, correct charging) joined to food assurance (allergen acknowledgment, lot-linked production, verified-rule-pack checks, recall and waste) without duplicating POS, recipe or stock ledgers. |
| Phases | 3 ordering, reservations, IRD, allergen acknowledgment, licensing gates; 4 food controls (lots, checks, recall, waste, reporting). |
| Release flag | R1. |
| Bounded context | `fnb-service` |
| SoR entities (owned) | `dining_reservation`, `waitlist_entry`, `ird_delivery`, `kitchen_ticket_timing`, `allergen_acknowledgment`, `production_batch`, `production_batch_lot_link`, `food_check_record`, `substitution_review`, `portion_waste_record`, `outlet_service_rule`. |
| Referenced (not owned) | M09 table/timed capacity; M13 POS checks, tenders, tips, voids; M14 recipes/BOM/allergens/yield; M50 lots, stock issue/waste/recall hold; M08 folio; M12/M16 BEO; M47 chef assignment; M61 checklist templates and corrective actions; M42 incidents; M44 food/alcohol rule packs; M52 surveys. |
| Dependencies | M08, M09, M12, M13, M14, M16, M42, M44, M47, M50, M61, M63. |

### F57.1 Guest meal

```yaml
- id: M57.F57.1.SF57.1.1
  name: table reservation/party/turn time and capacity
  phase: 3
  release: R1
  actors: [guest, server, fnb_manager, front_desk_agent]
  screens: [SCR-GST-dining-booking, SCR-FNB-reservation-book, SCR-FNB-waitlist]
  inputs: [outlet_id, date_time, party_size, special_requests, allergen_notice_flag, reservation_id_optional, turn_time_minutes]
  states: [requested, confirmed, waitlisted, seated, completed, no_show, cancelled]
  api: ["GET /v1/properties/{pid}/outlets/{oid}/availability", "POST /v1/properties/{pid}/outlets/{oid}/dining-reservations", "POST /v1/properties/{pid}/outlets/{oid}/waitlist"]
  events: [DiningReservationConfirmed, DiningGuestSeated, DiningNoShow]
  data: [dining_reservation, waitlist_entry, timed_resource_hold (M09)]
  rules: ["Capacity (tables, covers per slot, turn time) is held in M09; M57 never keeps a separate table count.", "Walk-in queue gives estimated wait from live table state; no fabricated wait times.", "Allergen notice flag on reservation creates a pre-service kitchen note."]
  security: "Guest sees own reservations; staff by outlet scope."
  failure_cases: [slot_race, outlet_closed_by_m61_hold, no_show_policy_dispute]
  finance_report_effect: "No-show fee only if disclosed and charged via M08/M28."
  i18n_a11y: "Accessible booking; seating accessibility need captured."
  acceptance: "AC-SF57.1.1: Two concurrent bookings for the last 4-top at 20:00 yield one confirmation and one waitlist offer."
  dependency: "M09 composite holds."

- id: M57.F57.1.SF57.1.2
  name: in-room dining order, kitchen promise and delivery confirmation
  phase: 3
  release: R1
  actors: [guest, server, shift_chef, front_desk_agent]
  screens: [SCR-GST-in-room-dining, SCR-FNB-ird-board, SCR-STF-ird-delivery]
  inputs: [room_id, reservation_id, menu_items, modifiers, allergen_declaration, requested_time, promised_time, delivered_at, recipient_confirmation]
  states: [placed, accepted, in_preparation, ready, out_for_delivery, delivered, delivery_failed, cancelled]
  api: ["POST /v1/properties/{pid}/ird-orders", "PUT /v1/properties/{pid}/ird-orders/{oid}/status", "POST /v1/properties/{pid}/ird-orders/{oid}/delivery-confirmation"]
  events: [IrdOrderPlaced, IrdPromiseSet, IrdDelivered, IrdDeliveryFailed]
  data: [ird_delivery, pos_check (M13)]
  rules: ["Order is an M13 POS check; M57 owns the delivery promise and confirmation only.", "Promised time from kitchen load at acceptance; breach of promise by configured minutes alerts fnb_manager and creates an M55 service note.", "Delivery confirmation by room occupant (verbal/tap) recorded by server; failed delivery returns food to waste record, never back to sale."]
  security: "Guest orders to own room only; check posted to verified in-house reservation."
  failure_cases: [kitchen_overload_promise_extended, room_dnd_on_delivery, wrong_room]
  finance_report_effect: "Revenue and service charge via M13 -> M08 -> M19; failed delivery portion to waste cost."
  i18n_a11y: "Menu accessible with allergen icons plus text; bilingual."
  acceptance: "AC-SF57.1.2: A room-service order delivered 25 minutes after promise triggers a delay alert and appears in the M55 complaint case when the guest complains (AT-G19.6)."
  dependency: "M13, M55, M08."

- id: M57.F57.1.SF57.1.3
  name: charge destination/tip/void/refund
  phase: 3
  release: R1
  actors: [server, cashier, fnb_manager]
  screens: [SCR-FNB-check-settle, SCR-FNB-void-approval]
  inputs: [pos_check_id, charge_destination, tip_amount, void_reason, approver_id]
  states: [open, settled, voided, refunded]
  api: ["POST /v1/properties/{pid}/pos-checks/{cid}/settle", "POST /v1/properties/{pid}/pos-checks/{cid}/void"]
  events: [PosCheckSettled, PosCheckVoided]
  data: [pos_check (M13), folio_line (M08)]
  rules: ["charge_destination is room folio, corporate account, event master, card or cash.", "Room charge only to in-house guest with charge privileges; event master only to active M12 account.", "Void/comp/discount above role limit requires M60 approval with reason.", "Tips allocated per M27/M13 policy; service charge treatment per M44 tax/labor rule pack."]
  security: "Server cannot approve own void."
  failure_cases: [charge_privileges_off, folio_closed, offline_pos_queue]
  finance_report_effect: "Revenue, tips payable and service charge liabilities to M19."
  i18n_a11y: "POS bilingual."
  acceptance: "AC-SF57.1.3: A room charge to a checked-out reservation is rejected; void above limit awaits approval."
  dependency: "M13, M08, M60."

- id: M57.F57.1.SF57.1.4
  name: allergy/diet disclosure and kitchen acknowledgment
  phase: 3
  release: R1
  actors: [guest, server, shift_chef, executive_chef]
  screens: [SCR-FNB-kds-allergen, SCR-GST-menu-allergens]
  inputs: [pos_check_line_id, declared_allergens, diet_type, recipe_allergen_profile (M14), chef_ack_user]
  states: [declared, conflict_flagged, acknowledged, refused_unsafe]
  api: ["POST /v1/properties/{pid}/allergen-acknowledgments"]
  events: [AllergenConflictFlagged, AllergenAcknowledged]
  data: [allergen_acknowledgment]
  rules: ["Item with declared allergen present in recipe profile is flagged at order; kitchen must acknowledge before firing.", "Menu allergen info derived from M14 recipe profile; manual edits are blocked.", "Kitchen may refuse unsafe orders with guest notification."]
  security: "Allergen declarations are health data: kept only with the check and deleted per retention; not in CRM segments."
  failure_cases: [recipe_profile_missing_block_item, ack_timeout_escalation]
  finance_report_effect: "None."
  i18n_a11y: "Allergen names localized; icons with text."
  acceptance: "AC-SF57.1.4: Ordering a nut dish with a nut allergy declared blocks firing until chef acknowledgment or substitution."
  dependency: "M14 allergen profile, M47 chef on duty."

- id: M57.F57.1.SF57.1.5
  name: club/bar licensing, age and responsible service gates by location where relevant
  phase: 3
  release: R1
  actors: [bartender, club_host, fnb_manager, compliance_officer]
  screens: [SCR-FNB-service-rules, SCR-FNB-age-check]
  inputs: [outlet_id, licence_ref, licence_expiry, permitted_hours, min_age, id_check_method]
  states: [licensed_active, licence_expiring, licence_expired_blocked, not_applicable]
  api: ["PUT /v1/properties/{pid}/outlets/{oid}/service-rules"]
  events: [OutletLicenceExpiring, AlcoholSaleBlocked]
  data: [outlet_service_rule]
  rules: ["Alcohol items unsellable in an outlet without active licence and within permitted hours per M44 rule pack; markets or properties where alcohol is not sold hide these items.", "Age check prompt recorded as a yes/no attestation, no ID image stored.", "Responsible-service refusal logged without guest-identifying detail in reports."]
  security: "Rule changes by compliance_officer."
  failure_cases: [licence_expired, rule_pack_unverified]
  finance_report_effect: "None."
  i18n_a11y: "Prompts bilingual."
  acceptance: "AC-SF57.1.5: After licence expiry, alcohol items are rejected by the POS API for that outlet."
  dependency: "M44, M13, D-626."

- id: M57.F57.1.SF57.1.6  # ADDED — Section C 'kitchen ticket timing'
  name: Kitchen ticket timing and course pacing
  phase: 3
  release: R1
  actors: [shift_chef, server, fnb_manager]
  screens: [SCR-FNB-kds, SCR-FNB-ticket-times]
  inputs: [ticket_id, fired_at, bumped_at, station, target_minutes]
  states: [queued, fired, late, bumped, recalled]
  api: ["GET /v1/properties/{pid}/kitchen/ticket-times"]
  events: [KitchenTicketLate]
  data: [kitchen_ticket_timing]
  rules: ["Ticket times captured from KDS bumps (M13); late tickets alert expediter.", "Reports are per station/period, not individual cooks."]
  security: "F&B roles."
  failure_cases: [kds_offline_paper_fallback]
  finance_report_effect: "None."
  i18n_a11y: "KDS high contrast."
  acceptance: "AC-SF57.1.6: A ticket exceeding target is shown late and counted in the station report."
  dependency: "M13 KDS."

- id: M57.F57.1.SF57.1.7  # ADDED — Section C 'food cost ... and guest satisfaction'
  name: Outlet food cost and guest satisfaction report
  phase: 4
  release: R1
  actors: [fnb_manager, executive_chef, gm]
  screens: [SCR-FNB-outlet-performance]
  inputs: [outlet_id, period]
  states: [provisional, closed]
  api: ["GET /v1/properties/{pid}/outlets/{oid}/performance?period"]
  events: [OutletPerformanceComputed]
  data: [pos_check (M13), stock_ledger_entry (M50), survey_response (M52)]
  rules: ["Food cost % = COGS from M50 issues/variance / net food revenue; theoretical vs actual shown.", "Satisfaction from M52 surveys mentioning the outlet; sample size shown."]
  security: "Management."
  failure_cases: [stocktake_missing_estimate_label]
  finance_report_effect: "Feeds M32 outlet margin."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF57.1.7: Missing stocktake shows food cost as estimate."
  dependency: "M50 SF50.3.7, M52."
```

### F57.2 Production assurance

```yaml
- id: M57.F57.2.SF57.2.1
  name: recipe yield and ingredient lot
  phase: 4
  release: R1
  actors: [shift_chef, executive_chef, storekeeper]
  screens: [SCR-FNB-production-batch]
  inputs: [recipe_id (M14), planned_portions, actual_yield, issued_lot_ids (M50), prep_time, beo_id_optional]
  states: [planned, in_production, produced, held, used, discarded]
  api: ["POST /v1/properties/{pid}/production-batches", "PUT /v1/properties/{pid}/production-batches/{bid}"]
  events: [ProductionBatchProduced, ProductionBatchDiscarded]
  data: [production_batch, production_batch_lot_link]
  rules: ["Batch links to M50 lots issued to the kitchen; lot links are immutable once produced.", "Yield variance vs M14 standard beyond tolerance prompts reason.", "BEO-linked batches carry the event ID for trace."]
  security: "Kitchen roles."
  failure_cases: [lot_not_issued, yield_outlier]
  finance_report_effect: "Theoretical consumption for M50 variance."
  i18n_a11y: "Kitchen tablet high-contrast bilingual."
  acceptance: "AC-SF57.2.1: A batch for a BEO links to the issued vegetable lot and the event (AT-G18.3)."
  dependency: "M14, M50, M12/M16."

- id: M57.F57.2.SF57.2.2
  name: production/holding/cold-chain checks according to verified local rule pack
  phase: 4
  release: R1
  actors: [shift_chef, executive_chef, compliance_officer]
  screens: [SCR-FNB-food-checks, SCR-STF-temperature-log]
  inputs: [check_template_id (M61), batch_id, measurement, unit, sensor_id_optional, time]
  states: [due, recorded_pass, recorded_fail, missed]
  api: ["POST /v1/properties/{pid}/food-checks"]
  events: [FoodCheckFailed, FoodCheckMissed]
  data: [food_check_record, inspection_template (M61)]
  rules: ["Critical limits and frequencies come from the M44/M61 rule pack; unverified pack shows 'hotel policy, not verified law'.", "Failed check forces corrective action (reheat, discard, hold) recorded before batch release.", "Missed check escalates via M63."]
  security: "Records immutable; corrections as new entries."
  failure_cases: [sensor_offline_manual_entry, rule_pack_unverified]
  finance_report_effect: "Discards to waste cost."
  i18n_a11y: "Units localized (C/F)."
  acceptance: "AC-SF57.2.2: A hot-holding reading below the limit blocks batch release until corrective action is recorded."
  dependency: "M61, M44, D-625."

- id: M57.F57.2.SF57.2.3
  name: substitution and allergen impact review
  phase: 4
  release: R1
  actors: [executive_chef, shift_chef, catering_manager]
  screens: [SCR-FNB-substitution-review]
  inputs: [recipe_id, original_item, substitute_item, allergen_delta, affected_orders_or_beos]
  states: [proposed, approved, rejected]
  api: ["POST /v1/properties/{pid}/substitution-reviews"]
  events: [SubstitutionApproved]
  data: [substitution_review]
  rules: ["Any substitution changing the allergen profile requires chef approval and updates menu/BEO allergen info before service.", "Affected event organizers notified via M12 change order when BEO items change."]
  security: "Chef role."
  failure_cases: [late_substitution_after_service_started]
  finance_report_effect: "Cost change reflected in recipe cost."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF57.2.3: Replacing sunflower oil with peanut oil flags new allergen and updates BEO allergen summary."
  dependency: "M14, M12, M50 substitutions."

- id: M57.F57.2.SF57.2.4
  name: guest/event lot trace and recall hold
  phase: 4
  release: R1
  actors: [executive_chef, storekeeper, compliance_officer, gm]
  screens: [SCR-FNB-trace, SCR-OPS-recall]
  inputs: [supplier_lot_id, recall_notice_ref, date_range]
  states: [trace_requested, trace_complete, hold_active, released, disposed]
  api: ["GET /v1/properties/{pid}/trace?lot_id", "POST /v1/properties/{pid}/recalls"]
  events: [RecallHoldPlaced, TraceCompleted]
  data: [production_batch_lot_link, recall_hold (M50)]
  rules: ["Trace returns batches, outlets, events and (where recorded) reservations/checks linked to the lot.", "Recall hold is placed in M50 and blocks issue/sale of remaining lot and derived batches.", "Guest notification decisions are human, via M55/M42."]
  security: "Guest identities in trace visible to compliance_officer and gm only."
  failure_cases: [incomplete_lot_capture_labelled]
  finance_report_effect: "Recall waste and supplier claims."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF57.2.4: Simulated recall of a lot lists the BEO and blocks remaining stock issue (AT-G18.4)."
  dependency: "M50 SF50.2.10, SF50.3.8."

- id: M57.F57.2.SF57.2.5
  name: wasted/returned portion and cost
  phase: 4
  release: R1
  actors: [shift_chef, server, fnb_manager]
  screens: [SCR-FNB-waste-log]
  inputs: [batch_id_or_check_line, quantity, waste_reason_code, photo]
  states: [recorded, approved]
  api: ["POST /v1/properties/{pid}/portion-waste"]
  events: [PortionWasteRecorded]
  data: [portion_waste_record, stock_ledger_entry (M50)]
  rules: ["waste_reason_code is overproduction, returned_by_guest, spoiled or dropped.", "Plate returns and discarded portions never return to sellable stock.", "Waste entries post to M50 waste transaction with reason; above threshold needs approval."]
  security: "Kitchen roles."
  failure_cases: [duplicate_waste_entry]
  finance_report_effect: "Waste cost to F&B COGS; feeds M67 food-waste metric."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF57.2.5: Recording returned plates increases waste and never available stock (AT-G18.5)."
  dependency: "M50 SF50.3.5, M67."

- id: M57.F57.2.SF57.2.6
  name: food-safety incident link
  phase: 4
  release: R1
  actors: [executive_chef, duty_manager, compliance_officer]
  screens: [SCR-FNB-food-incident, SCR-OPS-incident]
  inputs: [guest_case_id_or_report, suspected_items, batch_ids, symptoms_summary]
  states: [reported, investigating, hold_placed, closed]
  api: ["POST /v1/properties/{pid}/food-safety-incidents"]
  events: [FoodSafetyIncidentReported]
  data: [incident (M42), production_batch]
  rules: ["Creates an M42 incident of type food-safety, links batches and triggers trace; any suspected batch put on hold.", "Health details minimal and restricted; external reporting per jurisdiction via human decision."]
  security: "Restricted health data."
  failure_cases: [no_batch_link_manual_investigation]
  finance_report_effect: "Potential claim linkage to M68."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF57.2.6: Reporting suspected illness linked to a batch places the batch on hold and creates one M42 incident."
  dependency: "M42, M68, M61."
```

### M57 key invariants

1. No duplicate table/POS/recipe/stock records; M57 links M09/M13/M14/M50.
2. Allergen conflicts block firing until acknowledged.
3. Discarded or returned food never returns to sellable stock; recall holds block issue.
4. Food-check rules cite a verified rule pack or are labelled hotel policy.

### M57 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF57.1.2, AC-SF57.1.3 | AT-G19 (room-service complaint), AT-G03 | Chef/F&B: "Which tips/voids/waste require approval?" |
| AC-SF57.1.4, AC-SF57.2.3 | AT-G15 (BEO/allergen handover), AT-G18 | Chef/F&B: "What covers and BEO portions are due, with dietary/allergen restrictions?" |
| AC-SF57.2.1, AC-SF57.2.4, AC-SF57.2.5 | AT-G18 (lot, recall, waste) | Chef/F&B: "Which lot went into which event and can it be recalled? What should recipe use versus actual?" |

### M57 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-625 | Food safety critical limits and record-keeping per market (CFIA/provincial, Oman, KSA SFDA, Pakistan provincial food authorities, Portugal/EU). | Compliance Officer + local counsel | HACCP-style hotel policy template labelled `unverified-assumption` until rule pack verified. |
| D-626 | Alcohol licensing and age verification per market/outlet. | Compliance Officer | Alcohol items disabled unless outlet licence recorded. |
| D-627 | IRD delivery confirmation method (tap on guest app vs server attestation). | F&B Manager | Server attestation with timestamp; guest app tap optional. |

---

## M58 — Optional amenity businesses

| Field | Value |
|---|---|
| Purpose | Property-configured spa/wellness/pool/gym/beach/golf/retail operations with capacity, entitlements, safety/consent, POS/stock/folio and contribution, **hidden entirely when the hotel has no such business**. |
| Phases | 3 simple timed bookings; 4–7 amenity-specific activation (spa intake, retail stock, golf/beach specifics) as enabled for a pilot. |
| Release flag | R1 (hidden unless enabled via M01 feature flags). |
| Bounded context | `amenities` |
| SoR entities (owned) | `amenity_business_profile`, `amenity_service`, `amenity_booking`, `amenity_entitlement_use`, `practitioner_assignment`, `contraindication_intake`, `amenity_sanitation_record`, `amenity_closure`. |
| Referenced (not owned) | M01 feature flags; M09 timed resources/slots and composite holds; M15 membership plans/passes/access; M13 POS; M14/M50 retail stock; M08 folio; M27 practitioners (employees) and M46 contractors; M62 certifications; M61 checklists; M42 incidents; M44 rule packs; M19 department mapping. |
| Dependencies | M01, M08, M09, M13, M14, M15, M27, M42, M44, M46, M50, M61, M62. |

### F58.1 Capacity

```yaml
- id: M58.F58.1.SF58.1.1
  name: enable owned spa/pool/gym/beach/golf/retail modules independently
  phase: 3
  release: R1
  actors: [property_admin, gm]
  screens: [SCR-ADM-amenity-activation]
  inputs: [amenity_type, enabled_flag, operating_model, department_code, activation_evidence]
  states: [not_offered, configured, enabled, suspended, retired]
  api: ["PUT /v1/properties/{pid}/amenities/{type}"]
  events: [AmenityEnabled, AmenityDisabled]
  data: [amenity_business_profile]
  rules: ["operating_model is owned, leased or concession.", "Default is not_offered: navigation, website, guest app, offers, reports and APIs for that amenity are hidden/404.", "Enable requires department/GL mapping and at least one bookable service or POS outlet.", "Concession-run amenities can be listed but revenue flows per contract (commission only)."]
  security: "property_admin with gm approval."
  failure_cases: [enable_without_mapping_blocked]
  finance_report_effect: "Creates department in M19/M32 when enabled."
  i18n_a11y: "Admin bilingual."
  acceptance: "AC-SF58.1.1: With spa not_offered the guest app, website and staff navigation show no spa entries and spa API returns 404; enabling creates the department mapping."
  dependency: "M01 flags, M19 mapping, D-628."

- id: M58.F58.1.SF58.1.2
  name: specialist/therapist/equipment/time-slot and safety capacity
  phase: 3
  release: R1
  actors: [gm, practitioner, front_desk_agent, guest]
  screens: [SCR-ADM-amenity-capacity, SCR-STF-amenity-schedule]
  inputs: [service_id, duration, resource_requirements, safety_max_occupancy, buffer_minutes]
  states: [open, full, closed, safety_capacity_reached]
  api: ["PUT /v1/properties/{pid}/amenities/{type}/services/{sid}", "GET /v1/properties/{pid}/amenities/{type}/availability"]
  events: [AmenityCapacityChanged]
  data: [amenity_service, timed_resource (M09)]
  rules: ["A booking must reserve all required resources atomically via M09 composite hold.", "Safety maximum occupancy (pool, gym, beach zone) caps concurrent entries regardless of bookings.", "Practitioner availability from M27 roster or M46 contractor schedule."]
  security: "Amenity staff scope."
  failure_cases: [practitioner_absent_reschedule, equipment_out_of_service]
  finance_report_effect: "None directly."
  i18n_a11y: "Accessible scheduling."
  acceptance: "AC-SF58.1.2: A massage requiring room plus therapist cannot be booked when either is taken; pool entries stop at safety max."
  dependency: "M09, M27, M46."

- id: M58.F58.1.SF58.1.3
  name: membership/day-pass/guest/corporate rate
  phase: 3
  release: R1
  actors: [guest, corporate_booker, front_desk_agent, revenue_manager]
  screens: [SCR-ADM-amenity-rates, SCR-GST-day-pass]
  inputs: [price_list, audience_type, membership_plan_id (M15), corporate_agreement_id (M10)]
  states: [draft, active, retired]
  api: ["PUT /v1/properties/{pid}/amenities/{type}/rates"]
  events: [AmenityRateActivated]
  data: [amenity_service, membership_plan (M15)]
  rules: ["audience_type is in_house, day_guest, member or corporate.", "Memberships use M15 plans; no second membership store.", "Day-pass sales to non-residents create a guest profile minimal record in M18 and pass in M15.", "Corporate rates only for eligible M10 agreements."]
  security: "Rate changes by revenue_manager."
  failure_cases: [expired_membership]
  finance_report_effect: "Membership deferred revenue per M15/M19."
  i18n_a11y: "Prices localized."
  acceptance: "AC-SF58.1.3: An expired member is priced at day-pass rate."
  dependency: "M15, M10, M18."

- id: M58.F58.1.SF58.1.4
  name: room/guest entitlement and double-booking protection
  phase: 3
  release: R1
  actors: [guest, front_desk_agent]
  screens: [SCR-GST-amenity-booking, SCR-STF-amenity-checkin]
  inputs: [reservation_id, entitlement_ref (M54 package component), slot, idempotency_key]
  states: [held, booked, checked_in, completed, no_show, cancelled]
  api: ["POST /v1/properties/{pid}/amenities/{type}/bookings"]
  events: [AmenityBooked, AmenityEntitlementUsed]
  data: [amenity_booking, amenity_entitlement_use]
  rules: ["Entitlements (e.g. one spa treatment per stay) are consumed once via M54 component_consumption.", "Same guest cannot hold overlapping bookings in the same amenity.", "Composite hold release on payment failure."]
  security: "Guest own bookings."
  failure_cases: [double_submit, entitlement_already_used]
  finance_report_effect: "Package allocation per M54 SF54.2.3."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF58.1.4: A double-submitted booking creates one booking; a used entitlement cannot be reused."
  dependency: "M54, M09."

- id: M58.F58.1.SF58.1.5  # ADDED — Section C '4–7 amenity-specific activation'; Section P.5 specialist businesses only when enabled
  name: Amenity-specific activation gate and pilot evidence
  phase: 4
  release: R1
  actors: [gm, compliance_officer, property_admin]
  screens: [SCR-ADM-amenity-activation-checklist]
  inputs: [amenity_type, required_permits, insurance_ref (M68), safety_checklist (M61), staff_certifications (M62), test_fixture_results]
  states: [checklist_incomplete, ready_for_activation, activated, blocked]
  api: ["GET /v1/properties/{pid}/amenities/{type}/activation-checklist"]
  events: [AmenityActivationBlocked, AmenityActivationCompleted]
  data: [amenity_business_profile]
  rules: ["Enabling guest sales requires permits recorded, insurance on file, safety checklist configured and certified staff; missing item keeps status blocked.", "Golf/beach/advanced spa features ship as specified-and-tested fixtures but activate only for a pilot hotel that owns them (Phase 4–7)."]
  security: "gm approval."
  failure_cases: [permit_missing]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF58.1.5: Pool activation without lifeguard certification records stays blocked."
  dependency: "M61, M62, M68, M44."
```

### F58.2 Revenue/safety

```yaml
- id: M58.F58.2.SF58.2.1
  name: treatment/retail POS and stock
  phase: 4
  release: R1
  actors: [cashier, storekeeper, fnb_manager]
  screens: [SCR-FNB-amenity-pos, SCR-OPS-retail-stock]
  inputs: [pos_outlet_id (M13), retail_sku (M14), quantity, treatment_product_usage]
  states: [sold, returned, consumed]
  api: ["POST /v1/properties/{pid}/pos-checks"]
  events: [AmenityRetailSold]
  data: [pos_check (M13), stock_ledger_entry (M50)]
  rules: ["Retail and back-bar products are M14 SKUs moving through M50; treatment consumption via service BOM.", "Returns to sellable stock only if unopened and inspected."]
  security: "Outlet scope."
  failure_cases: [negative_stock_block]
  finance_report_effect: "Retail revenue/COGS to amenity department."
  i18n_a11y: "POS bilingual."
  acceptance: "AC-SF58.2.1: Selling a retail item decrements M50 once; opened returns go to waste."
  dependency: "M13, M14, M50."

- id: M58.F58.2.SF58.2.2
  name: intake/contraindication consent where locally permitted
  phase: 4
  release: R1
  actors: [guest, practitioner, dpo]
  screens: [SCR-GST-spa-intake, SCR-STF-intake-review]
  inputs: [booking_id, intake_form_version, answers, consent_signature_ref (M41), retention_period]
  states: [not_required, pending, completed, practitioner_reviewed, declined_service]
  api: ["POST /v1/properties/{pid}/amenities/spa/bookings/{bid}/intake"]
  events: [IntakeCompleted, ServiceDeclinedForSafety]
  data: [contraindication_intake]
  rules: ["Form only where M44 privacy pack and counsel permit health-data processing; otherwise verbal screening with no stored answers.", "Answers visible only to assigned practitioner; deleted after retention period.", "Never used for marketing, pricing or analytics."]
  security: "Special-category data encrypted, field-level access, access log."
  failure_cases: [rule_pack_unverified_verbal_only]
  finance_report_effect: "None."
  i18n_a11y: "Accessible bilingual forms; plain language."
  acceptance: "AC-SF58.2.2: A therapist not assigned to the booking cannot open the intake (403); data is purged after retention."
  dependency: "M41 e-sign, M44, M02, D-629."

- id: M58.F58.2.SF58.2.3
  name: practitioner qualification and room sanitation
  phase: 4
  release: R1
  actors: [gm, hr_officer, housekeeping_supervisor]
  screens: [SCR-ADM-practitioners, SCR-STF-sanitation-check]
  inputs: [practitioner_id, certification_ids (M62), expiry, room_id, sanitation_checklist (M61)]
  states: [qualified, expiring, unqualified, room_sanitized, room_pending]
  api: ["GET /v1/properties/{pid}/amenities/practitioners/{id}/eligibility"]
  events: [PractitionerQualificationLapsed, AmenityRoomSanitized]
  data: [practitioner_assignment, amenity_sanitation_record]
  rules: ["Booking assignment blocked for practitioners with lapsed required certificates (M62 SF62.1.5).", "Treatment room must have a completed sanitation check between sessions."]
  security: "Certificates visible to hr_officer and managers."
  failure_cases: [lapse_mid_schedule_reassign]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF58.2.3: A therapist whose certificate expired yesterday cannot be assigned today."
  dependency: "M62, M61."

- id: M58.F58.2.SF58.2.4
  name: commission/tips/folio/invoice
  phase: 4
  release: R1
  actors: [cashier, payroll_officer, finance_clerk]
  screens: [SCR-FIN-amenity-commissions]
  inputs: [booking_id, service_price, commission_rule, tip_amount, payer_destination]
  states: [accrued, approved, paid]
  api: ["GET /v1/properties/{pid}/amenities/commissions?period"]
  events: [AmenityCommissionAccrued]
  data: [practitioner_assignment, folio_line (M08)]
  rules: ["Employee commissions flow to M27 payroll; contractor commissions to M20 AP; tips per M27 policy.", "Charges post to folio/POS once."]
  security: "Individual commissions visible to payroll roles only."
  failure_cases: [refund_after_commission_clawback]
  finance_report_effect: "Commission expense and tips liability to M19."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF58.2.4: A refunded treatment claws back commission once."
  dependency: "M27, M20, M08, D-630."

- id: M58.F58.2.SF58.2.5
  name: incident/closure and department contribution
  phase: 4
  release: R1
  actors: [gm, duty_manager]
  screens: [SCR-OPS-amenity-closure, SCR-OWN-amenity-contribution]
  inputs: [amenity_type, closure_reason, incident_id, period]
  states: [open, closed_temporarily, reopened]
  api: ["POST /v1/properties/{pid}/amenities/{type}/closures"]
  events: [AmenityClosed, AmenityReopened]
  data: [amenity_closure, incident (M42)]
  rules: ["Closure cancels/reschedules bookings with guest notification and refunds via M08.", "Contribution = revenue - direct labor - products - allocated utilities (M32 policy)."]
  security: "Duty manager."
  failure_cases: [closure_with_active_sessions]
  finance_report_effect: "Department contribution in M32."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF58.2.5: Closing the pool for an incident cancels bookings in the window and shows reduced contribution."
  dependency: "M42, M32."

- id: M58.F58.2.SF58.2.6
  name: unavailable amenity never sold as available
  phase: 3
  release: R1
  actors: [guest, amenity_worker]
  screens: [SCR-WEB-amenities, SCR-GST-add-ons]
  inputs: [amenity_status, closure_windows, capacity]
  states: [sellable, not_sellable]
  api: ["GET /v1/public/properties/{slug}/amenities"]
  events: [AmenitySaleRejected]
  data: [amenity_business_profile, amenity_closure]
  rules: ["Any channel (website, app, M54 offers, package) checks live status at hold time; closed/disabled/at-capacity rejects.", "Package components for closed amenities trigger substitution or refund workflow."]
  security: "Public read minimal."
  failure_cases: [status_cache_stale]
  finance_report_effect: "Refunds for affected packages."
  i18n_a11y: "Clear unavailable messaging."
  acceptance: "AC-SF58.2.6: After closure, a website add-on attempt is rejected and no charge occurs."
  dependency: "M51, M54, M09."
```

### M58 key invariants

1. Not offered = invisible everywhere (UI, API, website, offers, reports).
2. All capacity via M09, memberships via M15, stock via M50, charges via M08.
3. Health intake only where lawful, visible only to assigned practitioner.

### M58 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF58.1.1, AC-SF58.2.6 | AT-G19 (accurate offers), AT-G02 (no double sale) | Marketing: "accurate amenity listings"; Guest: "honest total price" |
| AC-SF58.1.2, AC-SF58.1.4 | AT-G20 | GM: "What is sold, blocked, in service?" |
| AC-SF58.2.5 | AT-G08 | Owner: "Which departments... earn or lose money?" |

### M58 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-628 | Pilot hotel's enabled amenities. | GM | Pool and gym simple timed access only; spa/golf/beach/retail specified, tested on fixtures, disabled. |
| D-629 | Legality of storing spa intake health answers per market. | DPO + counsel | Verbal screening only; no stored answers. |
| D-630 | Commission/tip treatment for practitioners. | Financial Controller + HR | Employee commissions via payroll; contractors via AP; tips pass-through. |

---

## M59 — Local transport, fleet and dispatch

| Field | Value |
|---|---|
| Purpose | Dispatch hotel shuttle/driver/vehicle trips and external taxi assignments for airport/port pickups and local transfers, linked to flight/ship ETA, with manifests, accessibility, handover proof, permits/insurance validity, tariffs, settlement and trip cost. |
| Phases | 3 concierge dispatch (external taxi via M45/M46 providers, manual hotel shuttle); 4 owned fleet controls, cost and settlement. |
| Release flag | R1. |
| Bounded context | `transport` |
| SoR entities (owned) | `transport_trip`, `trip_leg`, `trip_manifest`, `trip_status_event`, `trip_handover`, `transport_tariff`, `fleet_vehicle_profile` (operational attributes; physical asset in M26), `driver_eligibility`, `trip_cost_line`. |
| Referenced (not owned) | M45 `travel_request` (guest consent, external provider transaction — see D-633); M46 taxi/transfer providers and credentials; M26 vehicle asset and maintenance; M27 drivers (employees), roster and time; M05 reservation/arrival; M08 folio; M20 AP settlement; M42 incidents; M68 insurance; M02 consent; M09 vehicle slots. |
| Dependencies | M02, M05, M08, M09, M20, M26, M27, M42, M45, M46, M68; INT-flight-status (optional licensed), INT-taxi-partner. |

Additional actor used in this module: `driver` (a hotel `employee` role specialization; external drivers act as `vendor_user`).

### F59.1 Dispatch

```yaml
- id: M59.F59.1.SF59.1.1
  name: hotel shuttle/driver/vehicle or external taxi assignment
  phase: 3
  release: R1
  actors: [concierge, front_desk_agent, driver, vendor_user]
  screens: [SCR-OPS-transport-dispatch, SCR-STF-driver-trips, SCR-VND-trip-offer]
  inputs: [trip_type, pickup_time, pickup_location, dropoff_location, passengers, vehicle_class, dispatch_mode, provider_id, idempotency_key]
  states: [requested, assigned, accepted, en_route, on_site, in_progress, completed, cancelled, failed]
  api: ["POST /v1/properties/{pid}/transport/trips", "POST /v1/properties/{pid}/transport/trips/{tid}/assign"]
  events: [TransportTripRequested, TransportTripAssigned, TransportTripCompleted]
  data: [transport_trip, trip_leg, timed_resource_hold (M09)]
  rules: ["dispatch_mode is owned_fleet or external_provider.", "Owned-fleet assignment reserves vehicle and driver slots atomically (M09); one vehicle cannot be double-assigned in overlapping windows.", "External taxi trips are created from an M45 travel_request with guest consent and an eligible M46 provider; status 'booked' only on provider confirmation reference.", "Only eligible drivers (SF59.2.1) and vehicles appear in assignment lists."]
  security: "Dispatch roles; vendor sees only own assigned trips."
  failure_cases: [no_vehicle_available, provider_no_confirmation, double_assignment_attempt]
  finance_report_effect: "Trip cost lines per SF59.2.4."
  i18n_a11y: "Dispatch board bilingual; driver app large controls."
  acceptance: "AC-SF59.1.1: Assigning a van already on an overlapping trip is rejected; an external taxi shows 'requested' until provider reference arrives (AT-G11.1)."
  dependency: "M09, M45, M46, M27."

- id: M59.F59.1.SF59.1.2
  name: pickup, flight/ship ETA and guest consent
  phase: 3
  release: R1
  actors: [concierge, guest, transport_worker]
  screens: [SCR-OPS-transport-dispatch, SCR-GST-transfer-request]
  inputs: [flight_number_or_ship_call, scheduled_arrival, eta_source, consent_ref, pickup_buffer]
  states: [scheduled, eta_updated, delayed, arrived, cancelled_by_carrier]
  api: ["POST /v1/properties/{pid}/transport/trips/{tid}/eta-link"]
  events: [TransportEtaUpdated, TransportCarrierDelay]
  data: [transport_trip, trip_status_event, consent_record (M02)]
  rules: ["Flight/ship ETA used only with guest consent for the transfer purpose; source and as-of shown.", "ETA changes beyond threshold re-plan pickup time and notify driver/provider; no automatic cancellation.", "Without a licensed status feed, staff update ETA manually (honesty label)."]
  security: "Flight numbers stored minimal and purged after trip per M65."
  failure_cases: [eta_feed_unavailable_manual, carrier_cancellation]
  finance_report_effect: "Waiting-time charges per tariff."
  i18n_a11y: "Times in local time zone with label."
  acceptance: "AC-SF59.1.2: A 90-minute flight delay updates pickup time and notifies the assigned driver once."
  dependency: "INT-flight-status (D-632), M02."

- id: M59.F59.1.SF59.1.3
  name: manifest, luggage and accessible vehicle
  phase: 3
  release: R1
  actors: [concierge, driver]
  screens: [SCR-OPS-trip-manifest, SCR-STF-driver-manifest]
  inputs: [passenger_count, passenger_display_names, luggage_count, wheelchair_accessible_required, child_seat_count]
  states: [draft, final, changed]
  api: ["PUT /v1/properties/{pid}/transport/trips/{tid}/manifest"]
  events: [TripManifestFinalized]
  data: [trip_manifest]
  rules: ["Vehicle capacity (seats, luggage, wheelchair) must satisfy manifest before assignment.", "Accessible requirement cannot be downgraded without guest confirmation."]
  security: "Driver sees display name and pickup details only."
  failure_cases: [capacity_exceeded_split_trip]
  finance_report_effect: "None."
  i18n_a11y: "Accessibility need icon plus text."
  acceptance: "AC-SF59.1.3: A manifest requiring wheelchair access cannot be assigned to a non-accessible vehicle."
  dependency: "SF59.1.1."

- id: M59.F59.1.SF59.1.4
  name: driver acknowledgment/trip live status
  phase: 3
  release: R1
  actors: [driver, vendor_user, concierge, guest]
  screens: [SCR-STF-driver-trips, SCR-GST-transfer-status]
  inputs: [trip_id, ack, status, location_ping_optional]
  states: [awaiting_ack, acknowledged, en_route, on_site, passenger_on_board, completed]
  api: ["POST /v1/properties/{pid}/transport/trips/{tid}/status"]
  events: [TripAcknowledged, TripStatusChanged]
  data: [trip_status_event]
  rules: ["No acknowledgment within threshold escalates to concierge.", "Location pings only during active trip, not stored beyond trip completion plus retention."]
  security: "Driver authenticated device (M64)."
  failure_cases: [driver_app_offline_sms_fallback]
  finance_report_effect: "None."
  i18n_a11y: "Guest status accessible."
  acceptance: "AC-SF59.1.4: A trip without driver acknowledgment 30 minutes before pickup escalates."
  dependency: "M63, M64."

- id: M59.F59.1.SF59.1.5
  name: missed pickup/disruption and rescue
  phase: 3
  release: R1
  actors: [concierge, duty_manager, guest]
  screens: [SCR-OPS-transport-exceptions]
  inputs: [trip_id, exception_type, rescue_option, guest_contact]
  states: [exception_open, rescue_dispatched, resolved, compensated]
  api: ["POST /v1/properties/{pid}/transport/trips/{tid}/exceptions"]
  events: [TransportExceptionOpened, TransportRescueDispatched]
  data: [transport_trip, guest_case (M55)]
  rules: ["Missed pickup creates an M55 case and a rescue option (backup vehicle/provider) within SLA.", "Guest no-show after carrier arrival is recorded with evidence before charging."]
  security: "Concierge/duty manager."
  failure_cases: [no_backup_available]
  finance_report_effect: "Compensation via M55; provider penalties via M20."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF59.1.5: A missed pickup opens one M55 case and dispatches a backup provider (AT-G11.3)."
  dependency: "M55, M46."

- id: M59.F59.1.SF59.1.6  # ADDED — Section C 'passenger handover'
  name: Passenger handover confirmation
  phase: 3
  release: R1
  actors: [driver, vendor_user, front_desk_agent]
  screens: [SCR-STF-driver-handover]
  inputs: [trip_id, handover_point, confirmed_by, timestamp]
  states: [pending, confirmed, disputed]
  api: ["POST /v1/properties/{pid}/transport/trips/{tid}/handover"]
  events: [PassengerHandoverConfirmed]
  data: [trip_handover]
  rules: ["Trip completion requires handover confirmation at drop-off (front desk or passenger tap/verbal attestation).", "Unaccompanied minors or assisted passengers require named receiving adult/staff."]
  security: "Minimal data."
  failure_cases: [recipient_absent]
  finance_report_effect: "Completion triggers billing."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF59.1.6: A trip cannot be billed as completed without handover confirmation."
  dependency: "SF59.1.4."
```

### F59.2 Controls

```yaml
- id: M59.F59.2.SF59.2.1
  name: permit/insurance/vehicle condition/maintenance validity
  phase: 4
  release: R1
  actors: [chief_engineer, hr_officer, concierge]
  screens: [SCR-ENG-fleet-validity, SCR-OPS-transport-dispatch]
  inputs: [vehicle_id, registration_expiry, insurance_policy_ref, inspection_due, driver_licence_expiry, driver_training_ref]
  states: [valid, expiring, invalid_blocked]
  api: ["GET /v1/properties/{pid}/transport/eligibility"]
  events: [VehicleIneligible, DriverIneligible]
  data: [fleet_vehicle_profile, driver_eligibility, asset (M26), insurance_policy (M68), certification_record (M62)]
  rules: ["Expired registration/insurance/inspection or overdue critical maintenance blocks vehicle assignment.", "Driver licence expiry and required training block driver assignment.", "External providers: validity asserted by M46 credentials; hotel cannot assign an unverified provider."]
  security: "Licence data restricted to hr_officer."
  failure_cases: [expiry_during_trip_allow_complete_flag]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF59.2.1: A vehicle with expired insurance disappears from assignment options."
  dependency: "M26, M46, M62, M68."

- id: M59.F59.2.SF59.2.2
  name: route and passenger data minimization
  phase: 3
  release: R1
  actors: [dpo, concierge]
  screens: [SCR-ADM-transport-privacy]
  inputs: [retention_days, fields_shared_with_provider]
  states: [configured]
  api: ["PUT /v1/properties/{pid}/transport/privacy-settings"]
  events: [TransportDataPurged]
  data: [transport_trip, trip_manifest]
  rules: ["Share with provider only name for sign, pickup point, time, contact number masked where supported.", "Purge location pings and flight numbers after retention; keep trip financial records."]
  security: "DPO controls."
  failure_cases: [provider_requires_more_data_escalate_dpo]
  finance_report_effect: "None."
  i18n_a11y: "Bilingual."
  acceptance: "AC-SF59.2.2: Provider payload excludes passport/ID and room number; purge job removes pings after retention."
  dependency: "M02, M65."

- id: M59.F59.2.SF59.2.3
  name: tariff, gratuity and payer
  phase: 3
  release: R1
  actors: [concierge, revenue_manager, guest]
  screens: [SCR-ADM-transport-tariffs, SCR-GST-transfer-quote]
  inputs: [tariff_id, zone, vehicle_class, waiting_fee, gratuity_policy, payer_type]
  states: [quoted, accepted, charged]
  api: ["PUT /v1/properties/{pid}/transport/tariffs", "POST /v1/properties/{pid}/transport/trips/{tid}/quote"]
  events: [TransportQuoteAccepted, TransportCharged]
  data: [transport_tariff, folio_line (M08)]
  rules: ["payer_type is guest_folio, corporate_account, package or complimentary.", "Quote shows total and payer before booking; tax per M44.", "Gratuity optional and never auto-added unless disclosed."]
  security: "Tariff by revenue_manager."
  failure_cases: [external_price_changed_requote]
  finance_report_effect: "Transport revenue to M08/M19."
  i18n_a11y: "Prices localized."
  acceptance: "AC-SF59.2.3: A corporate-paid transfer posts to the corporate account, not the guest folio."
  dependency: "M08, M10, M44."

- id: M59.F59.2.SF59.2.4
  name: external vendor settlement vs owned-fleet labor/fuel cost
  phase: 4
  release: R1
  actors: [ap_clerk, finance_clerk, chief_engineer]
  screens: [SCR-FIN-transport-settlement, SCR-OWN-transport-margin]
  inputs: [trip_ids, provider_invoice, driver_hours, fuel_receipts, vehicle_depreciation]
  states: [unmatched, matched, disputed, settled]
  api: ["GET /v1/properties/{pid}/transport/settlement?period"]
  events: [TransportSettlementMatched]
  data: [trip_cost_line, supplier_invoice (M20), time_record (M27)]
  rules: ["Provider invoices match completed trips with handover; unmatched lines disputed.", "Owned-fleet trip cost = allocated driver labor + fuel + maintenance + depreciation, labelled estimate until period close."]
  security: "Finance roles."
  failure_cases: [duplicate_invoice_line]
  finance_report_effect: "Transport department margin in M32."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF59.2.4: An invoice line for a trip without handover is flagged; margin report shows owned vs external."
  dependency: "M20, M27, M26, M19."

- id: M59.F59.2.SF59.2.5
  name: incident evidence and reimbursement
  phase: 4
  release: R1
  actors: [driver, duty_manager, security_officer, finance_clerk]
  screens: [SCR-OPS-transport-incident]
  inputs: [trip_id, incident_type, photos, police_report_ref, third_party_details]
  states: [reported, under_review, claim_filed, reimbursed, closed]
  api: ["POST /v1/properties/{pid}/transport/trips/{tid}/incidents"]
  events: [TransportIncidentReported]
  data: [incident (M42), insurance_claim (M68)]
  rules: ["Creates M42 incident; evidence packet available to M68 claim.", "Guest reimbursements via M55 compensation or M08 refund."]
  security: "Evidence restricted."
  failure_cases: [evidence_missing]
  finance_report_effect: "Claims and reimbursements in M19 via M68/M08."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF59.2.5: A vehicle incident produces an M42 incident and an evidence packet usable by M68."
  dependency: "M42, M68."
```

### M59 key invariants

1. No vehicle or driver double-assignment; ineligible vehicles/drivers never offered.
2. External trips never shown as booked without provider confirmation (Section P.3).
3. Minimal passenger data shared; location pings purged.

### M59 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF59.1.1, AC-SF59.1.5 | AT-G11 (taxi request with confirmed external reference, disruption) | Front desk: "Can I arrange taxi... with guest approval and an actual supplier confirmation?" |
| AC-SF59.2.1 | AT-G10 (eligible providers only) | Maintenance/security: "Which contractor can work safely?" |
| AC-SF59.2.4 | AT-G08 | Owner: "Which departments earn or lose money?" |

### M59 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-631 | Does the pilot hotel operate its own fleet? | GM | No owned fleet; external taxi via M45/M46 plus manual shuttle records. |
| D-632 | Flight/ship status data source and licence. | Concierge lead + Procurement | Manual ETA updates. |
| D-633 | Boundary M45 `travel_request` vs M59 `transport_trip`. | Lead Architect | M45 owns request/consent/provider transaction; M59 owns dispatch execution, manifest, status, handover, cost. |

---

## M60 — Revenue protection and control

| Field | Value |
|---|---|
| Purpose | Control framework over money-handling and revenue integrity: till/float/drops/safe, approval limits for refunds/voids/comps/discounts, night-audit difference queue, immutable corrections, anomaly detection (duplicates, outliers, commission mismatch, purchasing conflicts), and privacy-respecting investigations. |
| Phases | 2 core cash/night audit; 3 POS limits; 4–5 integrity analytics and cases. |
| Release flag | R1. |
| Bounded context | `revenue-control` |
| SoR entities (owned) | `control_limit_policy`, `till_session_control`, `cash_drop`, `safe_movement`, `audit_difference_item`, `control_approval`, `anomaly_rule`, `anomaly_alert`, `investigation_case`, `investigation_evidence`, `alert_feedback`. |
| Referenced (not owned) | M08 cashier shifts, folio, receipts, night audit run; M13 POS voids/discounts; M05 reservations; M07 commissions; M20 invoices/payments; M28 payments/refunds; M49 POs/awards; M46 vendor conflict declarations; M27 employees; M19 journals (corrections); M02 privileged audit. |
| Dependencies | M02, M05, M07, M08, M13, M19, M20, M28, M46, M49, M63. |

### F60.1 Shift and close

```yaml
- id: M60.F60.1.SF60.1.1
  name: till opening/float
  phase: 2
  release: R1
  actors: [cashier, front_office_manager]
  screens: [SCR-FIN-till-open, SCR-STF-cashier-shift]
  inputs: [cashier_shift_id, till_id, float_amount_by_denomination, currency, witness_id]
  states: [not_opened, opened, float_disputed]
  api: ["POST /v1/properties/{pid}/tills/{till_id}/open"]
  events: [TillOpened, FloatDisputed]
  data: [till_session_control, cashier_shift (M08)]
  rules: ["A cashier can hold one open till at a time; float counted and signed by cashier and witness/issuer.", "Multi-currency tills count per currency."]
  security: "Cashier authenticated; witness distinct user."
  failure_cases: [float_mismatch, till_already_open]
  finance_report_effect: "Float is a transfer between safe and till (M19 cash sub-accounts)."
  i18n_a11y: "Denomination entry accessible; OMR 3 decimals."
  acceptance: "AC-SF60.1.1: Opening a second till for the same cashier is rejected; a float mismatch is recorded, not overwritten."
  dependency: "M08 cashier shift."

- id: M60.F60.1.SF60.1.2
  name: cash drop/refund/comp approval
  phase: 2
  release: R1
  actors: [cashier, front_office_manager, duty_manager, fnb_manager]
  screens: [SCR-FIN-cash-drop, SCR-OPS-approval-queue]
  inputs: [amount, reason_code, related_folio_or_check, approver_id, idempotency_key]
  states: [requested, approved, rejected, executed]
  api: ["POST /v1/properties/{pid}/control-approvals", "POST /v1/properties/{pid}/cash-drops"]
  events: [ControlApprovalGranted, CashDropRecorded]
  data: [control_approval, cash_drop]
  rules: ["Refunds, comps and allowances above role limit (SF60.2.7) need an approver distinct from the requester.", "Cash refunds only up to till balance and only for cash-origin payments unless approved.", "Drops sealed and logged with bag ID; two-person verification where configured."]
  security: "Step-up MFA for high-value approvals."
  failure_cases: [approver_unavailable, duplicate_submission]
  finance_report_effect: "Drops move cash from till to safe; refunds reverse revenue via M08."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.1.2: A cashier cannot approve own refund above limit; duplicate drop submission records once."
  dependency: "M08, M28."

- id: M60.F60.1.SF60.1.3
  name: shift closing and safe/bank movement
  phase: 2
  release: R1
  actors: [cashier, front_office_manager, finance_clerk]
  screens: [SCR-FIN-shift-close, SCR-FIN-safe-register]
  inputs: [counted_by_denomination, expected_by_tender, variance, explanation, bank_deposit_ref]
  states: [counting, balanced, variance_pending, closed, deposited]
  api: ["POST /v1/properties/{pid}/tills/{till_id}/close", "POST /v1/properties/{pid}/safe-movements"]
  events: [ShiftClosed, CashVarianceRecorded, BankDepositRecorded]
  data: [till_session_control, safe_movement]
  rules: ["Blind count: cashier enters count before seeing expected amount.", "Variance beyond tolerance requires manager review and explanation; variance posts to over/short account.", "Bank deposit matched later in M20 bank reconciliation."]
  security: "Expected totals hidden until count submitted."
  failure_cases: [unclosed_shift_at_night_audit, deposit_not_matched]
  finance_report_effect: "Cash over/short to M19; deposits to bank clearing."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.1.3: Expected total not revealed before count; a variance above tolerance blocks closure until reviewed."
  dependency: "M08, M20 bank reconciliation."

- id: M60.F60.1.SF60.1.4
  name: night audit with difference queue
  phase: 2
  release: R1
  actors: [night_auditor, front_office_manager, finance_clerk, night_audit_worker]
  screens: [SCR-FIN-night-audit, SCR-OPS-audit-differences]
  inputs: [business_date, room_status_vs_folio_checks, unposted_charges, open_shifts, rate_variances]
  states: [pre_checks, differences_open, differences_resolved, audit_closed, reopened]
  api: ["GET /v1/properties/{pid}/night-audit/{date}/differences", "POST /v1/properties/{pid}/night-audit/{date}/differences/{did}/resolve"]
  events: [AuditDifferenceOpened, AuditDifferenceResolved]
  data: [audit_difference_item, night_audit_run (M08)]
  rules: ["Checks: occupied room without room charge, charge on vacant room, rate differing from reservation without override record, open shifts, unposted POS checks.", "Critical differences block business-date roll unless overridden by front_office_manager with reason (M08 run owns the roll).", "Unresolved items carry to next day with age."]
  security: "night_auditor; overrides audited."
  failure_cases: [pos_interface_down_manual_post]
  finance_report_effect: "Ensures room revenue integrity for M32 occupancy/ADR."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.1.4: An occupied room without room charge appears as a critical difference and blocks the roll until resolved or overridden (AT-G08.2)."
  dependency: "M08 night audit, M13."

- id: M60.F60.1.SF60.1.5
  name: immutable financial correction
  phase: 2
  release: R1
  actors: [front_office_manager, financial_controller, auditor]
  screens: [SCR-FIN-corrections]
  inputs: [original_entry_ref, correction_type, reason, approver_id]
  states: [requested, approved, posted]
  api: ["POST /v1/properties/{pid}/corrections"]
  events: [FinancialCorrectionPosted]
  data: [control_approval, folio_line (M08), journal_entry (M19)]
  rules: ["No update/delete on posted folio lines or journals; corrections are reversing plus new entries linked to the original.", "Closed-period corrections post in current period with reference unless period reopened per M19."]
  security: "Database role denies update/delete on ledger tables."
  failure_cases: [period_locked]
  finance_report_effect: "Reversal and repost visible in audit trail."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.1.5: A direct UPDATE on a posted folio line fails at DB level; correction shows original, reversal and new line linked."
  dependency: "M08, M19 SF19.2.2."
```

### F60.2 Integrity

```yaml
- id: M60.F60.2.SF60.2.1
  name: duplicate booking/invoice/payment detection
  phase: 4
  release: R1
  actors: [finance_clerk, ap_clerk, front_office_manager, control_worker]
  screens: [SCR-OPS-anomaly-queue]
  inputs: [match_keys, window, entity_type]
  states: [detected, confirmed_duplicate, not_duplicate, resolved]
  api: ["GET /v1/properties/{pid}/anomalies?type=duplicate"]
  events: [DuplicateSuspected, DuplicateResolved]
  data: [anomaly_alert, anomaly_rule]
  rules: ["Detect same guest/dates/room type reservations, same supplier invoice key/amount, same card/amount/time payments and duplicate refunds.", "Detection raises alerts; preventive idempotency remains in M05/M20/M28.", "Confirmed duplicates corrected via owning module reversal."]
  security: "Finance/front office scope."
  failure_cases: [legit_repeat_booking_false_positive]
  finance_report_effect: "Prevents double revenue/expense."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.2.1: A repeated supplier invoice and a duplicate refund attempt are both alerted (AT-G20.6)."
  dependency: "M05, M20, M28."

- id: M60.F60.2.SF60.2.2
  name: POS void/discount outlier
  phase: 4
  release: R1
  actors: [fnb_manager, financial_controller, control_worker]
  screens: [SCR-OPS-anomaly-queue]
  inputs: [pos_events, baseline_window, threshold]
  states: [detected, reviewed, explained, escalated]
  api: ["GET /v1/properties/{pid}/anomalies?type=pos_outlier"]
  events: [PosOutlierDetected]
  data: [anomaly_alert, pos_check (M13)]
  rules: ["Outliers computed per outlet/shift role against peer baseline; alert shows transactions, not a guilt score.", "Review by manager before any HR action; no automated sanction."]
  security: "Employee identities visible to managers with investigation scope only."
  failure_cases: [small_sample]
  finance_report_effect: "Void/discount cost reporting."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.2.2: A fixture server with voids 5x peer median is alerted with underlying checks listed."
  dependency: "M13."

- id: M60.F60.2.SF60.2.3
  name: channel commission and payout match
  phase: 5
  release: R1
  actors: [finance_clerk, revenue_manager]
  screens: [SCR-FIN-commission-match]
  inputs: [channel_statement, reservation_ids, commission_rates, virtual_card_payouts]
  states: [imported, matched, mismatch, disputed, settled]
  api: ["POST /v1/properties/{pid}/channel-statements", "GET /v1/properties/{pid}/channel-statements/{sid}/matches"]
  events: [CommissionMismatchDetected]
  data: [anomaly_alert, commission_statement_line (M07)]
  rules: ["Commission billed only on completed stays at contracted rate; cancellations/no-shows per contract.", "Mismatches open disputes with evidence."]
  security: "Finance."
  failure_cases: [statement_format_change]
  finance_report_effect: "Commission expense accuracy to M19; channel net contribution."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.2.3: Commission billed on a cancelled booking is flagged as mismatch."
  dependency: "M07, M20."

- id: M60.F60.2.SF60.2.4
  name: purchasing conflict or split-order alert
  phase: 4
  release: R1
  actors: [procurement_approver, financial_controller, compliance_officer]
  screens: [SCR-OPS-anomaly-queue]
  inputs: [requisition_ids, po_ids, vendor_bank_detail_hash, employee_bank_detail_hash, conflict_declarations]
  states: [detected, reviewed, cleared, escalated]
  api: ["GET /v1/properties/{pid}/anomalies?type=procurement"]
  events: [ProcurementConflictSuspected, SplitOrderSuspected]
  data: [anomaly_alert, purchase_order (M49), vendor (M46)]
  rules: ["Flag multiple POs to same vendor just under approval threshold within a window, requester-vendor declared relationships, and vendor bank account matching an employee (hash compare only).", "Alerts route to independent reviewer outside the requester's line."]
  security: "Hash comparison; no raw bank data exposed."
  failure_cases: [legit_recurring_orders]
  finance_report_effect: "None directly."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.2.4: Three POs of 990 against a 1000 threshold within a week are flagged as split orders."
  dependency: "M49, M46, M27."

- id: M60.F60.2.SF60.2.5
  name: authorized investigation case/evidence and employee privacy
  phase: 4
  release: R1
  actors: [financial_controller, gm, hr_officer, compliance_officer]
  screens: [SCR-OPS-investigations]
  inputs: [alert_ids, case_scope, authorized_by, evidence_items]
  states: [opened, authorized, evidence_collected, concluded, closed]
  api: ["POST /v1/properties/{pid}/investigations", "POST /v1/properties/{pid}/investigations/{iid}/evidence"]
  events: [InvestigationOpened, InvestigationConcluded]
  data: [investigation_case, investigation_evidence]
  rules: ["Case needs authorization by two roles (e.g. financial_controller + hr_officer); scope limits which records can be viewed.", "Evidence is hashed and chain-of-custody logged; employees' rights per M44 labor/privacy rules.", "Outcome recorded; no automatic disciplinary action."]
  security: "Case access restricted to named members; access log reviewed."
  failure_cases: [scope_creep_blocked]
  finance_report_effect: "Losses/recoveries recorded via M19."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.2.5: A case member cannot open records outside authorized scope; evidence hashes verify."
  dependency: "M02, M44, D-635."

- id: M60.F60.2.SF60.2.6
  name: false-positive feedback and access log
  phase: 4
  release: R1
  actors: [financial_controller, auditor]
  screens: [SCR-OPS-anomaly-feedback, SCR-ADM-audit-log]
  inputs: [alert_id, feedback_label, note]
  states: [labelled]
  api: ["POST /v1/properties/{pid}/anomalies/{aid}/feedback"]
  events: [AnomalyFeedbackRecorded, AnomalyRuleTuned]
  data: [alert_feedback, anomaly_rule]
  rules: ["feedback_label is true_positive or false_positive.", "Rule precision reported; thresholds changed only via versioned rule change with approval.", "All views of alerts and cases are logged for auditor review."]
  security: "Auditor read-only."
  failure_cases: [label_conflict_between_reviewers]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.2.6: Labelled alerts update rule precision metrics; every alert view is in the access log."
  dependency: "M02 audit."

- id: M60.F60.2.SF60.2.7  # ADDED — Section C 'POS void/comp/discount limits'
  name: Void/comp/discount/refund limit matrix
  phase: 3
  release: R1
  actors: [financial_controller, gm]
  screens: [SCR-FIN-control-limits]
  inputs: [role, transaction_type, per_item_limit, per_shift_limit, currency, effective_from]
  states: [draft, approved, active, superseded]
  api: ["PUT /v1/properties/{pid}/control-limits"]
  events: [ControlLimitPolicyActivated]
  data: [control_limit_policy]
  rules: ["Every refund/void/comp/discount/allowance API in M08/M13/M54/M55 checks this matrix server-side.", "Changes require gm approval; history retained."]
  security: "Step-up MFA."
  failure_cases: [missing_limit_defaults_to_zero]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF60.2.7: A role without a configured limit cannot void anything without approval."
  dependency: "M08, M13, M54, M55."
```

### M60 key invariants

1. Blind counts; variances recorded, never overwritten.
2. Corrections only by reversing entries (Section P.3).
3. Anomalies alert humans; no automated sanctions; investigations scoped and logged.
4. Limit checks enforced server-side for every money-reducing action.

### M60 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF60.1.3, AC-SF60.1.4 | AT-G08 (night audit), AT-G20 | Finance: "Are cash shifts and night audits closed?" |
| AC-SF60.2.1, AC-SF60.2.3 | AT-G20 (repeated invoice, duplicate webhook) | Finance: "Are all charges, refunds... recognized once?" |
| AC-SF60.2.4 | AT-G17 (conflict override) | Procurement: "Which weighted bid won and why?" |

### M60 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-634 | Cash variance tolerance and blind-count policy. | Financial Controller | Any non-zero variance requires manager review. |
| D-635 | Lawful employee-monitoring scope per market (anomaly analytics). | DPO + HR + counsel | Transaction-level analytics only; no behavioural monitoring; notices in staff policy. |
| D-636 | Safe custody and bank deposit process. | Financial Controller | Dual-control safe; daily deposit with bag ID. |

---

## M61 — Hygiene, safety and inspections

| Field | Value |
|---|---|
| Purpose | Jurisdiction-specific planned checks (food, pool, room, pest, fire, water) by credentialed people, with evidence, permit calendar, nonconformance and corrective action through retest and independent release, linked to guest safety incidents and vendors. |
| Phases | 3 food/room checks; 4 risk workflows (permits, corrective action, recall/room block, vendor corrective action). |
| Release flag | R1. |
| Bounded context | `safety-inspection` |
| SoR entities (owned) | `inspection_template`, `inspection_schedule`, `inspection_record`, `inspection_finding`, `corrective_action`, `permit_record`, `responsible_person_assignment`, `service_stoppage`, `release_decision`. |
| Referenced (not owned) | M44 rule packs; M62 certifications; M03 room block (OOO/OOS); M50 quarantine/recall hold; M26 work orders/assets/sensors; M46 vendors (pest control, water testing); M42 incidents; M63 tasks; M57 food checks and M56 room inspections use these templates. |
| Dependencies | M03, M26, M42, M44, M46, M50, M62, M63. |

### F61.1 Planned checks

```yaml
- id: M61.F61.1.SF61.1.1
  name: jurisdiction-specific food, pool, room, pest, fire and water checklist
  phase: 3
  release: R1
  actors: [compliance_officer, chief_engineer, executive_chef, housekeeping_supervisor]
  screens: [SCR-ADM-inspection-templates]
  inputs: [template_type, rule_pack_ref, items, critical_limits, evidence_required, version]
  states: [draft, pending_verification, active_verified, active_hotel_policy, retired]
  api: ["POST /v1/properties/{pid}/inspection-templates", "POST /v1/properties/{pid}/inspection-templates/{tid}/activate"]
  events: [InspectionTemplateActivated]
  data: [inspection_template, rule_pack (M44)]
  rules: ["Each template cites its M44 rule pack and status; without a verified pack it activates as 'hotel policy (unverified)' and is labelled so on every export.", "Templates versioned; records keep the version used.", "Fire/life-safety equipment checks do not replace statutory inspections by licensed parties."]
  security: "compliance_officer publishes."
  failure_cases: [rule_pack_expired, template_without_owner]
  finance_report_effect: "None."
  i18n_a11y: "Bilingual checklist items."
  acceptance: "AC-SF61.1.1: A template without a verified pack shows the unverified label in UI and export."
  dependency: "M44, D-637."

- id: M61.F61.1.SF61.1.2
  name: credentialed assessor/owner and schedule
  phase: 3
  release: R1
  actors: [compliance_officer, chief_engineer, hr_officer]
  screens: [SCR-OPS-inspection-calendar]
  inputs: [template_id, area_or_asset, frequency, assessor_role, required_certification]
  states: [scheduled, due, in_progress, completed, overdue]
  api: ["POST /v1/properties/{pid}/inspection-schedules"]
  events: [InspectionDue, InspectionOverdue]
  data: [inspection_schedule, certification_record (M62)]
  rules: ["Only users with current required certification can record the inspection.", "Overdue escalates via M63."]
  security: "Scoped by department."
  failure_cases: [no_certified_assessor]
  finance_report_effect: "None."
  i18n_a11y: "Accessible calendar."
  acceptance: "AC-SF61.1.2: A user with a lapsed certificate cannot complete a pool inspection."
  dependency: "M62."

- id: M61.F61.1.SF61.1.3
  name: sensor or manual evidence and exception
  phase: 3
  release: R1
  actors: [engineer, shift_chef, housekeeping_supervisor]
  screens: [SCR-STF-inspection-run]
  inputs: [inspection_schedule_id, item_results, sensor_readings, photos, notes]
  states: [recorded, exception_raised]
  api: ["POST /v1/properties/{pid}/inspection-records"]
  events: [InspectionRecorded, InspectionExceptionRaised]
  data: [inspection_record, inspection_finding, device (M64)]
  rules: ["Sensor readings stored with device ID and quality flag; manual entries marked manual.", "Out-of-limit results create a finding automatically.", "Records immutable; corrections appended."]
  security: "Device identity per M64."
  failure_cases: [sensor_offline_manual, photo_offline_queue]
  finance_report_effect: "None."
  i18n_a11y: "Offline-capable mobile."
  acceptance: "AC-SF61.1.3: A chlorine reading out of range creates a finding linked to the record."
  dependency: "M26 sensors, M64."

- id: M61.F61.1.SF61.1.4
  name: expired permit/task escalation
  phase: 4
  release: R1
  actors: [compliance_officer, gm]
  screens: [SCR-OPS-permit-calendar]
  inputs: [permit_type, authority, number, issue_date, expiry_date, document_ref]
  states: [valid, expiring, expired, renewed]
  api: ["POST /v1/properties/{pid}/permits"]
  events: [PermitExpiring, PermitExpired]
  data: [permit_record]
  rules: ["Alerts at configured lead times; expiry of a permit that gates an operation (pool, kitchen, alcohol) triggers the gate in the owning module.", "Renewal requires document upload."]
  security: "compliance_officer."
  failure_cases: [authority_delay]
  finance_report_effect: "Permit fees via AP."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF61.1.4: Pool operating permit expiry blocks M58 pool sales."
  dependency: "M58, M57, M44."

- id: M61.F61.1.SF61.1.5
  name: inspector-ready evidence/export
  phase: 4
  release: R1
  actors: [compliance_officer, auditor]
  screens: [SCR-OPS-inspection-export]
  inputs: [period, areas, template_types]
  states: [generated]
  api: ["POST /v1/properties/{pid}/inspection-exports"]
  events: [InspectionExportGenerated]
  data: [inspection_record, corrective_action]
  rules: ["Export includes records, template version, assessor credentials, findings and closures with hashes.", "Export is read-only and logged."]
  security: "Export access logged."
  failure_cases: [large_export_async]
  finance_report_effect: "None."
  i18n_a11y: "Accessible PDF; bilingual."
  acceptance: "AC-SF61.1.5: Export for a month reproduces all records with verifiable hashes."
  dependency: "M65 SF65.2.2."

- id: M61.F61.1.SF61.1.6  # ADDED — Section C 'responsible-person record'
  name: Responsible-person register
  phase: 3
  release: R1
  actors: [gm, compliance_officer]
  screens: [SCR-ADM-responsible-persons]
  inputs: [safety_domain, person_id, deputy_id, certification_ref, effective_from]
  states: [assigned, vacant, expired]
  api: ["PUT /v1/properties/{pid}/responsible-persons/{domain}"]
  events: [ResponsiblePersonVacant]
  data: [responsible_person_assignment]
  rules: ["safety_domain is one of food_safety, fire, pool, water, health_safety.", "Each regulated domain has a named responsible person and deputy; vacancy escalates to gm.", "Certification expiry makes the assignment expired."]
  security: "gm."
  failure_cases: [person_left]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF61.1.6: Terminating the responsible person in M27 marks the domain vacant and alerts gm."
  dependency: "M27, M62."
```

### F61.2 Corrective action

```yaml
- id: M61.F61.2.SF61.2.1
  name: severity and service stoppage
  phase: 4
  release: R1
  actors: [compliance_officer, duty_manager, gm]
  screens: [SCR-OPS-findings]
  inputs: [finding_id, severity, stoppage_scope_type, stoppage_scope_ref]
  states: [open, stoppage_active, contained]
  api: ["POST /v1/properties/{pid}/findings/{fid}/stoppage"]
  events: [ServiceStoppageActivated]
  data: [inspection_finding, service_stoppage]
  rules: ["stoppage_scope_type is area, outlet, room or amenity.", "Critical severity requires a stoppage decision within minutes; stoppage propagates to owning modules (M03 OOS, M58 closure, M13 outlet closed).", "Stoppage cannot be lifted without SF61.2.4 release."]
  security: "duty_manager or above."
  failure_cases: [propagation_failure_manual_notice]
  finance_report_effect: "Lost revenue visible via closures."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF61.2.1: Stoppage of the pool closes M58 sales and cannot be lifted without release."
  dependency: "M03, M13, M58."

- id: M61.F61.2.SF61.2.2
  name: quarantine/room block/food recall
  phase: 4
  release: R1
  actors: [storekeeper, executive_chef, housekeeping_supervisor]
  screens: [SCR-OPS-containment]
  inputs: [finding_id, lot_ids, room_ids, containment_type]
  states: [requested, applied, verified]
  api: ["POST /v1/properties/{pid}/findings/{fid}/containment"]
  events: [ContainmentApplied]
  data: [corrective_action, recall_hold (M50), room_block (M03)]
  rules: ["Food quarantine/recall via M50 holds; room blocks via M03; M61 records the link only.", "Containment verified by a second person."]
  security: "Role scoped."
  failure_cases: [lot_unknown_broad_hold]
  finance_report_effect: "Quarantine value tracked in M50."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF61.2.2: A pest finding blocks affected rooms in M03 and the room cannot be sold."
  dependency: "M50, M03."

- id: M61.F61.2.SF61.2.3
  name: responsible manager/vendor assignment
  phase: 4
  release: R1
  actors: [compliance_officer, chief_engineer, vendor_user]
  screens: [SCR-OPS-corrective-actions, SCR-VND-corrective-action]
  inputs: [finding_id, owner_id, vendor_id, due_date, work_order_id]
  states: [assigned, in_progress, completed, overdue]
  api: ["POST /v1/properties/{pid}/corrective-actions"]
  events: [CorrectiveActionAssigned, CorrectiveActionOverdue]
  data: [corrective_action, vendor (M46), work_order (M26)]
  rules: ["Vendor corrective action requires an eligible M46 vendor for that category; vendor sees only own actions.", "Overdue escalates."]
  security: "Vendor scope."
  failure_cases: [vendor_unavailable]
  finance_report_effect: "Vendor cost via M26/M49/M20."
  i18n_a11y: "Vendor app bilingual."
  acceptance: "AC-SF61.2.3: The pest vendor sees only its corrective action, not other findings."
  dependency: "M46, M26."

- id: M61.F61.2.SF61.2.4
  name: retest/independent release
  phase: 4
  release: R1
  actors: [compliance_officer, gm]
  screens: [SCR-OPS-release]
  inputs: [finding_id, retest_record_id, releaser_id]
  states: [retest_pending, retest_passed, retest_failed, released]
  api: ["POST /v1/properties/{pid}/findings/{fid}/release"]
  events: [ServiceReleased]
  data: [release_decision]
  rules: ["Releaser must differ from the corrective-action owner.", "Critical findings require a passing retest record before release."]
  security: "Step-up MFA."
  failure_cases: [retest_failed_loop]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF61.2.4: The engineer who fixed the issue cannot release it; release without passing retest is rejected."
  dependency: "SF61.1.3, D-638."

- id: M61.F61.2.SF61.2.5
  name: guest safety incident link and audit
  phase: 4
  release: R1
  actors: [duty_manager, compliance_officer, auditor]
  screens: [SCR-OPS-incident, SCR-OPS-findings]
  inputs: [incident_id, finding_ids]
  states: [linked]
  api: ["POST /v1/properties/{pid}/findings/{fid}/incident-link"]
  events: [FindingIncidentLinked]
  data: [inspection_finding, incident (M42)]
  rules: ["Guest injury/illness incidents link to related findings and inspection history for claim evidence (M68).", "Audit trail complete and exportable."]
  security: "Restricted."
  failure_cases: [incident_location_ambiguous]
  finance_report_effect: "Claims via M68."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF61.2.5: A slip incident at the pool links to the last pool inspection and appears in the M68 evidence packet."
  dependency: "M42, M68."
```

### M61 key invariants

1. Templates show whether they are verified law or hotel policy.
2. Stoppages propagate to owning modules and lift only by independent release after retest.
3. Containment actions execute in M50/M03 ledgers, never duplicated.

### M61 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF61.1.1, AC-SF61.1.3 | AT-G09 (unverified rules labelled), AT-G18 | Technology/compliance: "Where are... unverified tax rules?" (and unverified safety rules) |
| AC-SF61.2.1, AC-SF61.2.2, AC-SF61.2.4 | AT-G18 (quarantine/recall), AT-G14 | Maintenance/security: "Which room must be removed from sale?" |
| AC-SF61.2.5 | AT-G14 | Maintenance/security: "What evidence preserves an incident or insurance claim?" |

### M61 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-637 | Per-market mandatory inspection types, frequencies and records. | Compliance Officer + counsel | Hotel-policy templates labelled unverified. |
| D-638 | Independent release authority by domain. | GM | compliance_officer or gm, never the fixer. |
| D-639 | Water/legionella testing vendor and frequency. | Chief Engineer | Quarterly external test via an M46 vendor. |

---

## M62 — Staff enablement and quality

| Field | Value |
|---|---|
| Purpose | Role-based SOPs and multilingual training with attestation, certification expiry and automatic revocation of gated permissions, contractor site briefings, shift handover, labor demand vs roster, and privacy-bounded service-quality sampling and coaching. |
| Phases | 3 shift handover and SOPs, contractor briefing; 4 learning, certifications, workforce and quality; 5 outcome measures. |
| Release flag | R1. |
| Bounded context | `workforce-enablement` |
| SoR entities (owned) | `sop_document`, `sop_version`, `role_training_matrix`, `training_course`, `training_assignment`, `training_attestation`, `assessment_result`, `certification_record`, `contractor_briefing`, `shift_handover`, `labor_demand_forecast`, `quality_sample`, `coaching_note`. |
| Referenced (not owned) | M27 employee master, roster, time, overtime; M02 roles/permissions (revocation executes there); M46 contractor/vendor users; M47 chef credentials (reads `certification_record`); M53 occupancy forecast; M55 guest cases; M52 surveys; M32 labor cost; M44 labor/privacy rules. |
| Dependencies | M02, M27, M44, M46, M47, M53, M55, M63. |

### F62.1 Learning

```yaml
- id: M62.F62.1.SF62.1.1
  name: role/SOP and language matrix
  phase: 3
  release: R1
  actors: [hr_officer, gm, fnb_manager, housekeeping_supervisor, front_office_manager]
  screens: [SCR-ADM-role-training-matrix]
  inputs: [role, department, required_sops, required_courses, required_certifications, languages_available]
  states: [draft, active]
  api: ["PUT /v1/properties/{pid}/role-training-matrix/{role}"]
  events: [RoleTrainingMatrixUpdated]
  data: [role_training_matrix]
  rules: ["Each role lists required SOPs/courses/certs and which gate a permission (e.g. food_handler gates kitchen production actions).", "Content must exist in at least one language each assignee reads; gaps flagged."]
  security: "hr_officer and department heads."
  failure_cases: [language_gap]
  finance_report_effect: "None."
  i18n_a11y: "English/Arabic plus property-configured staff languages."
  acceptance: "AC-SF62.1.1: A role requiring a course only in English flags a gap for an Urdu-only reader in the assignee language profile."
  dependency: "M27 employee languages, M02 permissions."

- id: M62.F62.1.SF62.1.2
  name: onboarding and certification expiry
  phase: 4
  release: R1
  actors: [hr_officer, employee]
  screens: [SCR-ADM-certifications, SCR-STF-my-learning]
  inputs: [employee_id, certification_type, issuer, number, issue_date, expiry_date, document_ref]
  states: [pending_verification, valid, expiring, expired, rejected]
  api: ["POST /v1/properties/{pid}/certifications", "POST /v1/properties/{pid}/certifications/{cid}/verify"]
  events: [CertificationVerified, CertificationExpiring, CertificationExpired]
  data: [certification_record, training_assignment]
  rules: ["New hires get onboarding assignments from the matrix.", "Uploaded certificates verified by hr_officer before counting.", "Expiry alerts to employee and manager at lead times."]
  security: "Documents visible to hr_officer and the employee."
  failure_cases: [unverifiable_certificate]
  finance_report_effect: "Training costs via AP/payroll."
  i18n_a11y: "Accessible mobile."
  acceptance: "AC-SF62.1.2: An unverified certificate does not satisfy a gate; expiry alerts fire at 30 and 7 days."
  dependency: "M27."

- id: M62.F62.1.SF62.1.3
  name: training attestation and assessment
  phase: 4
  release: R1
  actors: [employee, hr_officer]
  screens: [SCR-STF-course-player, SCR-STF-attestation]
  inputs: [course_id, sop_version_id, completion, assessment_answers, attestation_signature]
  states: [assigned, in_progress, completed, assessed_pass, assessed_fail, attested]
  api: ["POST /v1/properties/{pid}/training-assignments/{aid}/complete", "POST /v1/properties/{pid}/training-assignments/{aid}/attest"]
  events: [TrainingCompleted, TrainingAttested]
  data: [training_attestation, assessment_result]
  rules: ["Attestation binds to the exact SOP version; new major versions require re-attestation.", "Assessment pass mark configurable; retries logged."]
  security: "Employee sees own results; managers see team completion, not answers."
  failure_cases: [offline_completion_sync]
  finance_report_effect: "Training hours may be paid time via M27."
  i18n_a11y: "Accessible course player with captions and screen-reader support."
  acceptance: "AC-SF62.1.3: Publishing a major SOP version reopens attestation for affected roles."
  dependency: "SF62.1.6."

- id: M62.F62.1.SF62.1.4
  name: contractor site briefing
  phase: 3
  release: R1
  actors: [chief_engineer, security_officer, vendor_user]
  screens: [SCR-VND-site-briefing, SCR-ENG-contractor-access]
  inputs: [vendor_id, worker_names, work_order_id, briefing_version, acknowledgment, site_hazards]
  states: [required, acknowledged, expired]
  api: ["POST /v1/properties/{pid}/contractor-briefings"]
  events: [ContractorBriefingAcknowledged]
  data: [contractor_briefing]
  rules: ["Contractor site access for an M26 job requires a current briefing acknowledgment per worker.", "Briefing covers hazards, permits-to-work, emergency procedures, guest privacy."]
  security: "Vendor sees only own briefings."
  failure_cases: [unbriefed_worker_at_gate]
  finance_report_effect: "None."
  i18n_a11y: "Bilingual briefing."
  acceptance: "AC-SF62.1.4: An unbriefed contractor worker cannot be checked in to site for the job."
  dependency: "M26 SF26.2.3, M46."

- id: M62.F62.1.SF62.1.5
  name: revocation on lapse
  phase: 4
  release: R1
  actors: [hr_officer, it_admin, enablement_worker]
  screens: [SCR-ADM-gated-permissions]
  inputs: [certification_record_id, gated_permission, grace_policy]
  states: [active, grace, revoked, restored]
  api: ["GET /v1/properties/{pid}/gated-permissions"]
  events: [GatedPermissionRevoked, GatedPermissionRestored]
  data: [certification_record, role_training_matrix]
  rules: ["When a gating certification expires, M02 removes the gated permission (e.g. food production, pool lifeguard duty, driving) at expiry unless a grace policy approved by gm applies.", "Restoration automatic on renewal verification.", "Roster (M27) and assignment engines (M47, M58, M59, M61) exclude revoked staff."]
  security: "Revocation audit."
  failure_cases: [revocation_mid_shift_notify_manager]
  finance_report_effect: "None."
  i18n_a11y: "Notifications bilingual."
  acceptance: "AC-SF62.1.5: A chef whose food-handler certificate expires cannot be assigned by M47 the next day (AT-G15.2)."
  dependency: "M02, M27, M47."

- id: M62.F62.1.SF62.1.6  # ADDED — Section C 'Department SOPs'
  name: SOP authoring, versioning and publication
  phase: 3
  release: R1
  actors: [gm, fnb_manager, housekeeping_supervisor, front_office_manager, content_approver]
  screens: [SCR-ADM-sop-library, SCR-STF-sop-viewer]
  inputs: [sop_title, department, body, attachments, version_type, approver_id]
  states: [draft, in_review, published, superseded, retired]
  api: ["POST /v1/properties/{pid}/sops", "POST /v1/properties/{pid}/sops/{sid}/versions/{v}/publish"]
  events: [SopPublished]
  data: [sop_document, sop_version]
  rules: ["Maker-checker publish; minor vs major version determines re-attestation.", "Staff app shows current version offline."]
  security: "Department scope."
  failure_cases: [translation_missing]
  finance_report_effect: "None."
  i18n_a11y: "Bilingual SOPs with accessible formatting."
  acceptance: "AC-SF62.1.6: A published SOP is readable offline on the staff app and a superseded version shows a banner."
  dependency: "M64 offline cache."
```

### F62.2 Service quality

```yaml
- id: M62.F62.2.SF62.2.1
  name: shift handover/coverage
  phase: 3
  release: R1
  actors: [front_office_manager, duty_manager, housekeeping_supervisor, shift_chef]
  screens: [SCR-STF-shift-handover, SCR-OPS-coverage]
  inputs: [department, shift_id, open_items, notes, incoming_owner, coverage_gaps]
  states: [drafted, handed_over, acknowledged, unacknowledged_escalated]
  api: ["POST /v1/properties/{pid}/shift-handovers", "POST /v1/properties/{pid}/shift-handovers/{hid}/acknowledge"]
  events: [ShiftHandedOver, ShiftHandoverUnacknowledged]
  data: [shift_handover, work_item (M63)]
  rules: ["Open work items auto-listed; incoming owner must acknowledge; unacknowledged escalates.", "Coverage gaps from M27 roster and M47 chef coverage shown."]
  security: "Department scope."
  failure_cases: [incoming_absent]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF62.2.1: Open guest cases transfer ownership on acknowledgment; an unacknowledged handover escalates after 15 minutes."
  dependency: "M63, M27, M47."

- id: M62.F62.2.SF62.2.2
  name: labor forecast versus roster and overtime
  phase: 4
  release: R1
  actors: [gm, front_office_manager, housekeeping_supervisor, fnb_manager, hr_officer]
  screens: [SCR-OPS-labor-demand]
  inputs: [occupancy_forecast (M53), covers_forecast, productivity_standards, roster (M27)]
  states: [computed, gap_flagged]
  api: ["GET /v1/properties/{pid}/labor-demand?from&to"]
  events: [LaborGapFlagged]
  data: [labor_demand_forecast]
  rules: ["Demand hours = drivers x standards per department; compared with rostered hours; gaps and projected overtime flagged.", "Recommendations only; roster changes in M27."]
  security: "Aggregate hours; no individual pay."
  failure_cases: [forecast_low_confidence]
  finance_report_effect: "Projected labor cost (estimate) per department."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF62.2.2: A forecast of 90% occupancy with a housekeeping roster covering 70% of required hours flags a gap."
  dependency: "M53, M27."

- id: M62.F62.2.SF62.2.3
  name: sampled quality review and coaching
  phase: 4
  release: R1
  actors: [front_office_manager, housekeeping_supervisor, fnb_manager, employee]
  screens: [SCR-OPS-quality-samples, SCR-STF-my-coaching]
  inputs: [sample_type, sampled_record_ref, rubric_version, score, coaching_note]
  states: [sampled, reviewed, coached, acknowledged]
  api: ["POST /v1/properties/{pid}/quality-samples"]
  events: [QualitySampleReviewed, CoachingNoteShared]
  data: [quality_sample, coaching_note]
  rules: ["Random sampling from completed work (inspections, conversations, checks) with published rubric; staff are informed of the program.", "No covert audio/video recording; call recordings only if D-619 permits.", "Coaching notes shared with the employee."]
  security: "Individual results visible to the employee, direct manager and HR only."
  failure_cases: [sample_bias_review]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF62.2.3: An employee can view own samples and notes; a peer cannot (403)."
  dependency: "M44 labor/privacy, D-642."

- id: M62.F62.2.SF62.2.4
  name: dispute/correction and confidentiality
  phase: 4
  release: R1
  actors: [employee, hr_officer]
  screens: [SCR-STF-my-coaching]
  inputs: [quality_sample_id, dispute_text, outcome]
  states: [disputed, upheld, amended]
  api: ["POST /v1/properties/{pid}/quality-samples/{qid}/dispute"]
  events: [QualityDisputeResolved]
  data: [quality_sample]
  rules: ["Employee may dispute a score; HR decides; amended scores keep history.", "Quality data retained per M65 and not used as sole basis for dismissal."]
  security: "Confidential."
  failure_cases: [reviewer_conflict]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF62.2.4: An upheld dispute amends the score with history visible to HR."
  dependency: "M65."

- id: M62.F62.2.SF62.2.5
  name: guest outcome and staff-cost measure
  phase: 5
  release: R1
  actors: [gm, owner]
  screens: [SCR-OWN-service-quality]
  inputs: [period, department]
  states: [computed]
  api: ["GET /v1/properties/{pid}/service-quality/outcomes?period"]
  events: [ServiceOutcomeComputed]
  data: [quality_sample, guest_case (M55), survey_response (M52)]
  rules: ["Report department-level labor cost per occupied room/cover alongside complaint rate, recovery time and survey scores; correlation not causation language.", "No individual salaries (M27 confidentiality)."]
  security: "Aggregates only."
  failure_cases: [small_sample]
  finance_report_effect: "Uses M32 labor cost."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF62.2.5: Report shows department aggregates only; no individual pay appears."
  dependency: "M32, M55, M52."
```

### M62 key invariants

1. Gated permissions are revoked at certificate lapse and excluded from all assignment engines.
2. Attestations bind to exact SOP versions.
3. Quality sampling is transparent, confidential and disputable; no covert monitoring; no individual pay in reports.

### M62 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF62.1.5 | AT-G15 (emergency chef food-safety qualification) | Chef: "Which chef and backup will work?"; Maintenance: "Which contractor can work safely?" (with AC-SF62.1.4) |
| AC-SF62.2.1, AC-SF62.2.2 | AT-G05 (shifts), AT-G15 | GM: "Which shift/team/chef coverage is missing?" |
| AC-SF62.2.5 | AT-G08 | Owner: "Is this property profitable after payroll...?" |

### M62 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-640 | Build in-product learning vs integrate an external LMS. | HR + Product Owner | In-product lightweight courses and attestations; SCORM import deferred. |
| D-641 | Required certifications per role per market (food handler, lifeguard, first aid, fire warden, driver). | HR + Compliance Officer | Hotel-defined list labelled unverified until rule pack verified. |
| D-642 | Quality sampling program scope and staff notice. | HR + DPO | Record-based sampling only; written notice to staff; no recordings. |

---

## M63 — Workflow automation and service desk

| Field | Value |
|---|---|
| Purpose | The shared task/SLA/approval/notification engine: reusable, versioned, property-specific templates with triggers, timers, escalations, maker-checker, cross-department handoffs, safe retries, bounded no-code configuration, dry-run and rollback, and audit. It is the SoR for `work_item`; domain modules own their domain records. |
| Phases | 2 engine (tasks, SLA, approvals, escalations, notifications); 3–6 templates, governance, simulator and performance. |
| Release flag | R1. |
| Bounded context | `workflow` |
| SoR entities (owned) | `workflow_template`, `workflow_version`, `workflow_trigger`, `workflow_instance`, `work_item`, `sla_timer`, `escalation_step`, `approval_request`, `approval_decision`, `notification_rule`, `notification_delivery`, `on_call_schedule`, `workflow_change_audit`. |
| Referenced (not owned) | M01 outbox/inbox event bus and job queue; M02 roles/scopes/delegations; M27 roster (on-call); every domain module's events as triggers; M64 device push registration; M65 performance metrics. |
| Dependencies | M01, M02, M27, M64, M65; INT-push, INT-sms (staff alerts). |

### F63.1 Task

```yaml
- id: M63.F63.1.SF63.1.1
  name: versioned trigger/template
  phase: 2
  release: R1
  actors: [property_admin, gm, workflow_worker]
  screens: [SCR-ADM-workflow-templates, SCR-ADM-workflow-editor]
  inputs: [template_key, trigger_event_type, filter_expression, steps, sla_policy, version_notes]
  states: [draft, validated, active, deprecated, retired]
  api: ["POST /v1/properties/{pid}/workflows", "POST /v1/properties/{pid}/workflows/{wid}/versions", "POST /v1/properties/{pid}/workflows/{wid}/versions/{v}/activate"]
  events: [WorkflowVersionActivated, WorkflowInstanceStarted]
  data: [workflow_template, workflow_version, workflow_trigger, workflow_instance]
  rules: ["Triggers subscribe to domain events via the M01 inbox (dedup on event_id); filter expressions use a bounded, side-effect-free expression language.", "Each instance pins the version active at start; new versions affect new instances only.", "Seed templates ship for Release 1 flows (arrival readiness, complaint SLA, chef callout, permit expiry, delivery follow-up, approval chains)."]
  security: "Template editing limited per SF63.2.1; instances inherit property scope."
  failure_cases: [invalid_filter_expression, trigger_event_schema_version_change]
  finance_report_effect: "None directly; approvals gate financial actions in owning modules."
  i18n_a11y: "Template labels bilingual; editor keyboard-accessible (no drag-only)."
  acceptance: "AC-SF63.1.1: Activating v2 leaves running v1 instances unchanged; a replayed trigger event starts one instance only."
  dependency: "M01 outbox/inbox, D-643."

- id: M63.F63.1.SF63.1.2
  name: owner, property/department and SLA
  phase: 2
  release: R1
  actors: [duty_manager, front_office_manager, housekeeping_supervisor, workflow_worker]
  screens: [SCR-OPS-my-work, SCR-STF-my-tasks, SCR-OWN-exceptions-overview]
  inputs: [work_item_type, owner_user_or_role, property_id, department_id, due_at, priority, source_ref]
  states: [open, assigned, acknowledged, in_progress, waiting, done, cancelled]
  api: ["GET /v1/properties/{pid}/work-items?owner=me", "PUT /v1/properties/{pid}/work-items/{wid}"]
  events: [WorkItemCreated, WorkItemAssigned, WorkItemCompleted, SlaBreached]
  data: [work_item, sla_timer]
  rules: ["Every work_item has exactly one accountable owner (user or role queue), property, department, due time and source record link.", "SLA clocks respect property time zone and optional business hours; pauses recorded.", "Role home screens show top 3–7 items by priority/due (Section P.1)."]
  security: "Users see items in their scopes; source-record fields filtered by scope."
  failure_cases: [owner_deactivated_reassign, clock_skew]
  finance_report_effect: "None."
  i18n_a11y: "Unified inbox accessible, bilingual, mobile-friendly."
  acceptance: "AC-SF63.1.2: Every open exception in a fixture property has an owner and due time; the GM overview lists overdue items by owner (Section O GM question)."
  dependency: "M02 scopes."

- id: M63.F63.1.SF63.1.3
  name: safe retry/dedup and acknowledgment
  phase: 2
  release: R1
  actors: [workflow_worker, it_admin]
  screens: [SCR-ADM-workflow-runs, SCR-OPS-dead-letters]
  inputs: [step_id, attempt, idempotency_key, ack_required]
  states: [pending, running, succeeded, retrying, failed_dead_letter, acknowledged]
  api: ["POST /v1/properties/{pid}/workflow-instances/{iid}/steps/{sid}/retry"]
  events: [WorkflowStepFailed, WorkflowStepDeadLettered]
  data: [workflow_instance]
  rules: ["Automated steps are idempotent and retried with exponential backoff up to a limit then dead-lettered with alert.", "Steps calling money/stock/external commands never retry blindly: they call the owning module command with Idempotency-Key and inquire status on timeout.", "Notifications requiring acknowledgment re-notify then escalate."]
  security: "Dead-letter replay by it_admin with audit."
  failure_cases: [downstream_timeout_unknown_status, poison_message]
  finance_report_effect: "Prevents duplicate postings."
  i18n_a11y: "Accessible admin."
  acceptance: "AC-SF63.1.3: Simulated timeout on a payable-approval step results in status inquiry, not a second command; poison message lands in dead letter with alert (AT-G20.2)."
  dependency: "M01 job queue."

- id: M63.F63.1.SF63.1.4
  name: human action/approval with reason
  phase: 2
  release: R1
  actors: [procurement_approver, finance_approver, duty_manager, gm]
  screens: [SCR-OPS-approval-queue, SCR-STF-approvals]
  inputs: [approval_request_id, decision, reason, delegated_from, step_up_token]
  states: [pending, approved, rejected, expired, delegated]
  api: ["POST /v1/properties/{pid}/approvals/{aid}/decision"]
  events: [ApprovalGranted, ApprovalRejected, ApprovalExpired]
  data: [approval_request, approval_decision]
  rules: ["Maker-checker: requester cannot approve own request; delegation per M02 with validity window.", "Reason mandatory for reject and for overrides.", "Approval is recorded here; the owning module executes the action and re-checks authorization server-side."]
  security: "Step-up MFA for thresholds configured by module."
  failure_cases: [approver_absent_delegate, concurrent_decisions]
  finance_report_effect: "Approval evidence linked to financial documents."
  i18n_a11y: "Accessible approval cards."
  acceptance: "AC-SF63.1.4: Two approvers clicking simultaneously produce one decision; self-approval is rejected."
  dependency: "M02 delegated approvals."

- id: M63.F63.1.SF63.1.5
  name: escalation, pause and cancel
  phase: 2
  release: R1
  actors: [duty_manager, gm, workflow_worker]
  screens: [SCR-OPS-escalations]
  inputs: [work_item_id, escalation_policy, pause_reason, cancel_reason]
  states: [on_track, escalated_l1, escalated_l2, paused, cancelled]
  api: ["POST /v1/properties/{pid}/work-items/{wid}/pause", "POST /v1/properties/{pid}/work-items/{wid}/cancel"]
  events: [WorkItemEscalated, WorkItemPaused, WorkItemCancelled]
  data: [escalation_step, work_item]
  rules: ["Escalation ladder by severity (owner -> supervisor -> duty_manager -> gm) with on-call from SF63.1.6.", "Pause requires reason and resume time; cancel requires reason and keeps history.", "Life-safety escalations also go through M42, not only M63."]
  security: "Scoped."
  failure_cases: [nobody_on_call]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF63.1.5: When nobody acknowledges an L2 escalation, the gm is paged and the item shows 'unowned risk' on the GM screen (AT-G15.3)."
  dependency: "SF63.1.6, M42."

- id: M63.F63.1.SF63.1.6  # ADDED — Section C 'notifications'; Section O GM 'who owns every unresolved exception'
  name: Notification routing, channels, quiet hours and on-call
  phase: 2
  release: R1
  actors: [property_admin, employee, duty_manager]
  screens: [SCR-ADM-notification-rules, SCR-STF-notification-settings, SCR-ADM-on-call]
  inputs: [event_type, recipients_rule, channels, quiet_hours, on_call_rota, critical_flag]
  states: [queued, sent, delivered, failed, acknowledged]
  api: ["PUT /v1/properties/{pid}/notification-rules", "PUT /v1/properties/{pid}/on-call"]
  events: [NotificationSent, NotificationFailed, NotificationAcknowledged]
  data: [notification_rule, notification_delivery, on_call_schedule]
  rules: ["Staff channels: in-app/push first, SMS fallback for critical; staff WhatsApp only via approved business account and staff consent.", "Quiet hours never suppress critical/safety alerts.", "On-call rota derived from M27 roster with manual overrides."]
  security: "Staff phone numbers from M27 visible only to messaging service."
  failure_cases: [push_token_expired, sms_provider_outage]
  finance_report_effect: "Messaging costs to IT/admin cost center."
  i18n_a11y: "Notifications in the recipient's preferred language."
  acceptance: "AC-SF63.1.6: A critical alert during quiet hours is delivered and falls back to SMS when push fails."
  dependency: "M27, M64 devices, D-645."
```

### F63.2 Governance

```yaml
- id: M63.F63.2.SF63.2.1
  name: permitted configuration by role
  phase: 3
  release: R1
  actors: [tenant_admin, property_admin, gm]
  screens: [SCR-ADM-workflow-permissions]
  inputs: [role, allowed_template_categories, allowed_step_types, max_sla_change]
  states: [configured]
  api: ["PUT /v1/properties/{pid}/workflow-permissions"]
  events: [WorkflowPermissionsChanged]
  data: [workflow_template]
  rules: ["No-code editing limited to allowed step types (notify, assign, wait, approve, create task, call whitelisted module command); no arbitrary code or external HTTP calls.", "Financial and safety templates editable only by designated roles and require approval to activate."]
  security: "Changes audited."
  failure_cases: [privilege_escalation_attempt_blocked]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF63.2.1: A department manager cannot add a step calling an unlisted module command or edit the payment approval template."
  dependency: "M02, D-644."

- id: M63.F63.2.SF63.2.2
  name: dry-run/simulator before activation
  phase: 3
  release: R1
  actors: [property_admin, gm]
  screens: [SCR-ADM-workflow-simulator]
  inputs: [workflow_version_id, sample_events, time_acceleration]
  states: [simulated_pass, simulated_warnings, simulated_fail]
  api: ["POST /v1/properties/{pid}/workflows/{wid}/versions/{v}/simulate"]
  events: [WorkflowSimulated]
  data: [workflow_version]
  rules: ["Activation requires a passed simulation on recorded or synthetic events within the last 24 h.", "Simulator has no side effects (sandboxed commands)."]
  security: "No real notifications sent."
  failure_cases: [unreachable_step, infinite_loop_detected]
  finance_report_effect: "None."
  i18n_a11y: "Accessible results."
  acceptance: "AC-SF63.2.2: A version with an unreachable step fails simulation and cannot be activated."
  dependency: "SF63.1.1."

- id: M63.F63.2.SF63.2.3
  name: guest-sensitive data minimization
  phase: 3
  release: R1
  actors: [dpo, property_admin]
  screens: [SCR-ADM-workflow-data-policy]
  inputs: [field_classification, allowed_fields_per_step, protected_attribute_blocklist]
  states: [configured]
  api: ["PUT /v1/properties/{pid}/workflow-data-policy"]
  events: [WorkflowDataPolicyChanged]
  data: [workflow_version]
  rules: ["Steps receive only fields needed; notification text cannot embed health, ID or payment data.", "Conditions on protected attributes (nationality, religion, gender, age, disability, ethnicity) are rejected at validation; accessibility/diet needs usable only when stated by the guest for fulfillment routing."]
  security: "DPO owns blocklist."
  failure_cases: [template_references_blocked_field]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF63.2.3: A condition 'nationality = X' fails validation; a notification template containing an ID number placeholder is rejected."
  dependency: "M02, M55 SF55.2.6."

- id: M63.F63.2.SF63.2.4
  name: workflow version rollback
  phase: 3
  release: R1
  actors: [property_admin, gm]
  screens: [SCR-ADM-workflow-editor]
  inputs: [workflow_id, target_version]
  states: [rolled_back]
  api: ["POST /v1/properties/{pid}/workflows/{wid}/rollback"]
  events: [WorkflowRolledBack]
  data: [workflow_version]
  rules: ["Rollback activates a prior version as a new activation record; running instances continue on their pinned version unless admin migrates them explicitly."]
  security: "Same approval as activation."
  failure_cases: [prior_version_incompatible_trigger_schema]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF63.2.4: Rolling back restores prior behaviour for new instances within one minute."
  dependency: "SF63.1.1."

- id: M63.F63.2.SF63.2.5
  name: cross-department audit and performance
  phase: 4
  release: R1
  actors: [gm, auditor, property_admin]
  screens: [SCR-OWN-exceptions-overview, SCR-ADM-workflow-audit]
  inputs: [period, department, template_key]
  states: [computed]
  api: ["GET /v1/properties/{pid}/work-items/performance?period"]
  events: [WorkflowPerformanceComputed]
  data: [workflow_change_audit, work_item, sla_timer]
  rules: ["Report on-time %, median time-to-acknowledge/resolve, escalations and handoffs by department and template.", "All template changes, activations and overrides in immutable audit."]
  security: "Aggregates for management; auditor read-only."
  failure_cases: [none_blocking]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF63.2.5: Fixture work items produce on-time % matching expected; audit lists every activation with actor."
  dependency: "M65."
```

### M63 key invariants

1. One accountable owner per work item; SLA always visible.
2. No blind retries of money/stock/external commands; idempotency plus status inquiry.
3. No-code is bounded: no arbitrary code, no protected-attribute conditions, versions pinned per instance.

### M63 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF63.1.2, AC-SF63.1.5 | AT-G15 (escalation when nobody accepts), AT-G19 | GM: "Who owns every unresolved exception and by when?" |
| AC-SF63.1.3 | AT-G20 (duplicate webhook, outage) | Technology: "Where are... failed callbacks?" |
| AC-SF63.2.3 | AT-G19 | Guest safety/fairness (no discriminatory automation) |

### M63 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-643 | Workflow runtime (pg-boss only vs Temporal for long sagas). | Lead Architect | pg-boss plus state tables in Phase 2; Temporal evaluated in Phase 3 for sagas over 24 h. |
| D-644 | Which roles may edit which template categories. | GM + Tenant Admin | property_admin edits operational templates; financial/safety templates need gm plus financial_controller or compliance_officer. |
| D-645 | Staff alert channels (push/SMS/WhatsApp) and on-call rules. | IT Admin + HR | Push first, SMS fallback for critical; no staff WhatsApp until approved. |

---

## M64 — Technology, site devices and resilience

| Field | Value |
|---|---|
| Purpose | Know and keep healthy every connected site device and connector (PMS clients, POS, locks, gates/LPR, phones, Wi-Fi, BMS, printers, scales, kiosks), operate degraded/offline with tested manual alternatives, and prove recovery (backups, restore, DR failover, cyber incident isolation, patch/change control, support desk, status page). |
| Phases | 2 foundation (device registry, health, offline sync, backups, time sync); 3–6 tested profiles, change/patch, support, cyber, measured recovery; 7 optional UC/HSIA/IPTV profiles. |
| Release flag | R1; `Later` for SF64.1.7. |
| Bounded context | `site-ops` |
| SoR entities (owned) | `device`, `device_firmware_record`, `certificate_record`, `connector_profile`, `network_zone`, `device_health_sample`, `degraded_queue_item`, `maintenance_window`, `manual_fallback_procedure`, `fallback_test_record`, `backup_job`, `restore_test`, `recovery_objective`, `offline_action_log`, `cyber_containment_record`, `change_request`, `support_ticket`, `status_page_entry`, `dr_failover_test`. |
| Referenced (not owned) | M01 deployment profiles, secrets, observability; M02 device identity and privileged access; M26 physical asset records (device linked to asset); M33 integration adapters/event replay; M42 incidents (cyber incident type); M17 gates/LPR, M13 POS, M34–M36 UC/HSIA/IPTV; M68 continuity plans. |
| Dependencies | M01, M02, M13, M17, M26, M33, M42, M68; INT-mdm (optional), INT-status-page. |

### F64.1 Connected site

```yaml
- id: M64.F64.1.SF64.1.1
  name: device/firmware/cert inventory
  phase: 2
  release: R1
  actors: [it_admin, integration_admin, chief_engineer]
  screens: [SCR-ENG-device-registry, SCR-ENG-certificates]
  inputs: [device_type, vendor, model, serial, asset_id, location, firmware_version, certificate_expiry, connector_profile_id, enrollment_token]
  states: [registered, enrolled, active, degraded, retired, revoked]
  api: ["POST /v1/properties/{pid}/devices", "POST /v1/properties/{pid}/devices/{did}/enroll", "GET /v1/properties/{pid}/certificates?expiring_within"]
  events: [DeviceEnrolled, DeviceRevoked, CertificateExpiring, FirmwareOutdated]
  data: [device, device_firmware_record, certificate_record, asset (M26)]
  rules: ["Each device links to one M26 asset; the physical asset record is not duplicated.", "Staff mobile devices enroll with device identity (M02) and can be remotely revoked/wiped of cached data.", "Certificates tracked with expiry alerts at 30/7/1 days; supported connector list per device model with honesty label."]
  security: "Enrollment tokens single-use; device keys in secure hardware where available."
  failure_cases: [unknown_device_on_network, cert_expired_connector_down]
  finance_report_effect: "Device costs via M26/M19 asset or expense."
  i18n_a11y: "Admin bilingual."
  acceptance: "AC-SF64.1.1: A revoked staff device can no longer sync and its cache key is invalidated; certificate expiring in 7 days raises an alert."
  dependency: "M02, M26."

- id: M64.F64.1.SF64.1.2
  name: network segmentation/connector permission
  phase: 3
  release: R1
  actors: [it_admin, integration_admin]
  screens: [SCR-ENG-network-zones, SCR-ADM-connector-permissions]
  inputs: [zone_name, devices, allowed_flows, connector_scopes]
  states: [designed, verified, drifted]
  api: ["PUT /v1/properties/{pid}/network-zones/{zid}", "PUT /v1/properties/{pid}/connectors/{cid}/scopes"]
  events: [NetworkPolicyDriftDetected]
  data: [network_zone, connector_profile]
  rules: ["Reference design separates guest Wi-Fi, staff, POS/payment (PCI scope), building systems/BMS/cameras, and servers.", "Each connector has least-privilege scopes to the platform API; no shared admin tokens.", "Segmentation verified at site acceptance and re-checked after changes."]
  security: "PCI scope minimization; camera streams never routed to guest networks."
  failure_cases: [flat_network_site_blocker]
  finance_report_effect: "None."
  i18n_a11y: "Admin bilingual."
  acceptance: "AC-SF64.1.2: A connector token with parking scope cannot call folio endpoints (403); site checklist records segmentation evidence."
  dependency: "D-646, M17, M13."

- id: M64.F64.1.SF64.1.3
  name: health and degraded service queue
  phase: 2
  release: R1
  actors: [it_admin, duty_manager, site_worker]
  screens: [SCR-ENG-health, SCR-OPS-outage-mode]
  inputs: [heartbeat, error_rate, queue_depth, last_success_at]
  states: [healthy, degraded, down, recovering]
  api: ["GET /v1/properties/{pid}/health", "GET /v1/properties/{pid}/degraded-queue"]
  events: [DeviceHealthDegraded, ConnectorDown, DegradedQueueDrained]
  data: [device_health_sample, degraded_queue_item]
  rules: ["When a connector is down, commands queue in degraded_queue with TTL and are replayed in order when healthy; money/external commands inquire status before replay.", "Users see 'degraded' banner with the manual procedure (SF64.1.5).", "Health alerts route through M63 on-call."]
  security: "Queue items encrypted at rest."
  failure_cases: [queue_ttl_expired_manual_action, replay_conflict]
  finance_report_effect: "None."
  i18n_a11y: "Banner accessible and bilingual."
  acceptance: "AC-SF64.1.3: With the POS interface down, room charges queue and post once on recovery; a queue item past TTL becomes a manual task (AT-G20.5)."
  dependency: "M63, M33."

- id: M64.F64.1.SF64.1.4
  name: vendor maintenance window
  phase: 3
  release: R1
  actors: [it_admin, vendor_user, duty_manager]
  screens: [SCR-ENG-maintenance-windows]
  inputs: [device_or_connector, vendor_id, start, end, impact, fallback_procedure_id]
  states: [requested, approved, in_progress, completed, overrun]
  api: ["POST /v1/properties/{pid}/maintenance-windows"]
  events: [MaintenanceWindowStarted, MaintenanceWindowOverrun]
  data: [maintenance_window]
  rules: ["Windows avoid peak check-in/out by default and need duty_manager approval.", "Vendor remote access time-boxed and logged (M02 privileged access)."]
  security: "Just-in-time vendor access."
  failure_cases: [overrun_escalation]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF64.1.4: Vendor remote access outside the approved window is denied."
  dependency: "M02, M46."

- id: M64.F64.1.SF64.1.5
  name: tested manual gate/key/POS alternatives
  phase: 3
  release: R1
  actors: [duty_manager, security_officer, parking_attendant, cashier, front_desk_agent]
  screens: [SCR-OPS-manual-procedures, SCR-ENG-fallback-tests]
  inputs: [procedure_type, steps, printable_forms, last_test_date, test_result]
  states: [defined, tested_pass, tested_fail, overdue_test]
  api: ["POST /v1/properties/{pid}/fallback-procedures", "POST /v1/properties/{pid}/fallback-procedures/{fid}/tests"]
  events: [FallbackTestRecorded, FallbackTestOverdue]
  data: [manual_fallback_procedure, fallback_test_record]
  rules: ["Each critical device class (gate/LPR, locks, POS, payment terminal, PMS client) has a documented manual procedure with reconciliation steps.", "Procedures tested at least on the configured frequency; overdue tests escalate.", "Manual gate overrides are logged in M17 with reason and later reconciled."]
  security: "Procedures available offline on staff devices and printed."
  failure_cases: [test_failed_corrective_action]
  finance_report_effect: "Manual sales reconciled via M60."
  i18n_a11y: "Printable bilingual procedures."
  acceptance: "AC-SF64.1.5: A simulated gate outage uses the manual override procedure; each manual entry appears in the M17 reconciliation (AT-G03.2, AT-G14.2)."
  dependency: "M17, M13, M60."

- id: M64.F64.1.SF64.1.6
  name: on-prem/cloud behavior and time synchronization
  phase: 2
  release: R1
  actors: [it_admin, tenant_admin]
  screens: [SCR-ENG-deployment-profile]
  inputs: [deployment_profile, ntp_sources, clock_drift_threshold, offsite_link_status]
  states: [in_sync, drift_warning, drift_critical]
  api: ["GET /v1/properties/{pid}/deployment-status"]
  events: [ClockDriftDetected, OffsiteLinkDown]
  data: [device_health_sample]
  rules: ["Servers and devices sync to approved NTP; drift beyond threshold blocks signing events (e-signatures, audit) until corrected.", "On-prem profile continues local operations when internet is down; cloud-only features (website, OTA, PSP) show degraded status.", "Business date comes from M01, never device clock."]
  security: "Authenticated NTP where supported."
  failure_cases: [ntp_unreachable, offsite_backup_link_down]
  finance_report_effect: "Correct timestamps for audit and tax."
  i18n_a11y: "Admin bilingual."
  acceptance: "AC-SF64.1.6: Injected 5-minute drift on a device blocks its signature capture and raises an alert."
  dependency: "M01 deployment profiles."

- id: M64.F64.1.SF64.1.7  # ADDED — Section C 'M64 7 optional UC/HSIA/IPTV'
  name: Optional UC/HSIA/IPTV device profiles
  phase: 7
  release: Later
  actors: [it_admin, integration_admin]
  screens: [SCR-ENG-device-registry]
  inputs: [pbx_model, hsia_gateway, iptv_platform, certification_status]
  states: [not_enabled, certified, active]
  api: ["POST /v1/properties/{pid}/devices"]
  events: [DeviceProfileCertified]
  data: [device, connector_profile]
  rules: ["Profiles for M34/M35/M36 appear only after vendor certification; otherwise hidden."]
  security: "Same as SF64.1.1."
  failure_cases: [certification_missing]
  finance_report_effect: "None."
  i18n_a11y: "Admin bilingual."
  acceptance: "AC-SF64.1.7: Without certification the UC/HSIA/IPTV device types are not selectable."
  dependency: "M34, M35, M36 (Phase 7)."
```

### F64.2 Recover

```yaml
- id: M64.F64.2.SF64.2.1
  name: encrypted backups and independent restore tests
  phase: 2
  release: R1
  actors: [it_admin, auditor, backup_worker]
  screens: [SCR-ENG-backups, SCR-ENG-restore-tests]
  inputs: [backup_policy, schedule, retention, target_location, encryption_key_ref, restore_environment]
  states: [scheduled, running, succeeded, failed, restore_tested_pass, restore_tested_fail]
  api: ["GET /v1/properties/{pid}/backups", "POST /v1/properties/{pid}/restore-tests"]
  events: [BackupSucceeded, BackupFailed, RestoreTestPassed, RestoreTestFailed]
  data: [backup_job, restore_test]
  rules: ["Database PITR plus object storage backups encrypted with keys held separately; offsite copy for on-prem.", "Restore tests to an isolated environment at least monthly; test verifies row counts, ledger balances and a sample reservation-to-folio check.", "Failed backup or missing restore test escalates."]
  security: "Backup access separate from production admin (separation of duties)."
  failure_cases: [offsite_upload_failed, key_unavailable]
  finance_report_effect: "Backup storage cost to IT."
  i18n_a11y: "Admin bilingual."
  acceptance: "AC-SF64.2.1: Monthly restore test restores to an isolated environment and trial balance equals production snapshot (AT-G20 restore, Section O technology question)."
  dependency: "M01, D-648."

- id: M64.F64.2.SF64.2.2
  name: offline booking/payment conflict policy and eventual sync
  phase: 2
  release: R1
  actors: [front_desk_agent, housekeeper, cashier, it_admin, sync_worker]
  screens: [SCR-STF-sync-status, SCR-OPS-sync-conflicts]
  inputs: [offline_actions, base_versions, device_id, action_types_allowed_offline]
  states: [queued_offline, replaying, applied, conflict, rejected]
  api: ["POST /v1/properties/{pid}/sync/replay", "GET /v1/properties/{pid}/sync/conflicts"]
  events: [OfflineActionsReplayed, OfflineConflictDetected]
  data: [offline_action_log]
  rules: ["Allowed offline: room status updates, task completion, minibar counts, notes, check-in of pre-assigned rooms, cash receipts. Not allowed offline: new reservations that consume inventory, card payments without terminal offline mode approved by PSP, refunds.", "Replay uses optimistic concurrency; conflicts never silently overwrite; money actions replay with Idempotency-Key.", "Offline window limit configurable; beyond it, device requires online revalidation."]
  security: "Offline cache encrypted and bound to enrolled device and user."
  failure_cases: [replay_conflict, device_lost]
  finance_report_effect: "Offline cash reconciled at M60 shift close."
  i18n_a11y: "Sync state accessible."
  acceptance: "AC-SF64.2.2: An offline attempt to create a new reservation is refused locally; replayed minibar counts post once (AT-G20.5)."
  dependency: "M05, M56, M60."

- id: M64.F64.2.SF64.2.3
  name: cyber incident isolation and log preservation
  phase: 4
  release: R1
  actors: [it_admin, gm, dpo, security_officer]
  screens: [SCR-ENG-cyber-incident]
  inputs: [incident_id, affected_devices, isolation_actions, log_snapshot_refs, notification_assessment]
  states: [detected, contained, eradicated, recovered, closed]
  api: ["POST /v1/properties/{pid}/cyber-incidents/{iid}/containment"]
  events: [CyberIncidentContained, LogsPreserved]
  data: [cyber_containment_record, incident (M42)]
  rules: ["Cyber incidents are M42 incidents with type cyber; M64 records technical containment.", "Isolation actions: revoke device/credentials, rotate secrets, block connector; each logged.", "Logs snapshotted to write-once storage with hash; breach-notification assessment by DPO per jurisdiction."]
  security: "Break-glass access with dual approval."
  failure_cases: [logs_tampered_detected, containment_breaks_operations_fallback]
  finance_report_effect: "Incident costs tracked for M68 claim."
  i18n_a11y: "Admin bilingual."
  acceptance: "AC-SF64.2.3: Tabletop injection of a compromised staff device results in revocation, secret rotation and a hashed log snapshot within the target time (AT-G14)."
  dependency: "M42, M68, M01 secrets."

- id: M64.F64.2.SF64.2.4
  name: measured recovery objectives
  phase: 6
  release: R1
  actors: [it_admin, gm, owner]
  screens: [SCR-ENG-recovery-objectives]
  inputs: [service, rpo_target, rto_target, measured_rpo, measured_rto, test_ref]
  states: [defined, met, not_met]
  api: ["GET /v1/properties/{pid}/recovery-objectives"]
  events: [RecoveryObjectiveMissed]
  data: [recovery_objective]
  rules: ["Targets agreed with the pilot hotel (Section P.6), not asserted universally.", "Measured values come only from restore/failover tests or real incidents; untested services show 'unmeasured'."]
  security: "Management read."
  failure_cases: [target_not_met_release_blocker]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF64.2.4: A service without a test in the period shows 'unmeasured' rather than meeting target."
  dependency: "SF64.2.1, SF64.2.7, D-647."

- id: M64.F64.2.SF64.2.5
  name: patch/change approval/rollback and operator alert
  phase: 3
  release: R1
  actors: [it_admin, gm, integration_admin]
  screens: [SCR-ENG-change-requests]
  inputs: [change_type, affected_components, risk, rollback_plan, window, approver]
  states: [requested, approved, scheduled, implemented, rolled_back, failed]
  api: ["POST /v1/properties/{pid}/change-requests"]
  events: [ChangeImplemented, ChangeRolledBack]
  data: [change_request]
  rules: ["Every change has a rollback plan and approval; emergency security patches allowed with retrospective approval within 24 h.", "Operators alerted before/after; failed change auto-alerts on-call."]
  security: "Separation: implementer differs from approver for high-risk changes."
  failure_cases: [rollback_failed_dr_invoke]
  finance_report_effect: "None."
  i18n_a11y: "Admin bilingual."
  acceptance: "AC-SF64.2.5: A high-risk change without a rollback plan cannot be approved."
  dependency: "M63 approvals."

- id: M64.F64.2.SF64.2.6
  name: support ownership and status page
  phase: 3
  release: R1
  actors: [it_admin, gm, employee, vendor_user]
  screens: [SCR-ENG-support-desk, SCR-STF-report-it-issue, SCR-ADM-status-page]
  inputs: [issue_description, component, severity, owner_tier, status_message]
  states: [new, triaged, in_progress, waiting_vendor, resolved, closed]
  api: ["POST /v1/properties/{pid}/support-tickets", "POST /v1/properties/{pid}/status-page/entries"]
  events: [SupportTicketOpened, StatusPageUpdated]
  data: [support_ticket, status_page_entry]
  rules: ["Each component has a support owner (hotel IT, MetriSys, named vendor) and SLA.", "Status page shows component status to staff; guest-facing notices only for guest-impacting outages and approved text."]
  security: "Ticket attachments scanned; no guest PII in tickets."
  failure_cases: [owner_unknown_route_to_it_admin]
  finance_report_effect: "None."
  i18n_a11y: "Bilingual, accessible."
  acceptance: "AC-SF64.2.6: A ticket for the LPR camera routes to the named vendor owner with SLA."
  dependency: "D-649."

- id: M64.F64.2.SF64.2.7  # ADDED — Section C 'DR failover'
  name: DR failover and failback test
  phase: 6
  release: R1
  actors: [it_admin, gm]
  screens: [SCR-ENG-dr-tests]
  inputs: [dr_plan_version, failover_target, test_scope, observed_data_loss, observed_duration]
  states: [planned, executing, failed_over, failed_back, passed, failed]
  api: ["POST /v1/properties/{pid}/dr-tests"]
  events: [DrFailoverTested]
  data: [dr_failover_test]
  rules: ["SaaS: region/zone failover; on-prem: restore to standby host or cloud standby from offsite backup.", "Failback verified with ledger integrity checks; results feed SF64.2.4."]
  security: "DR credentials in separate vault scope."
  failure_cases: [standby_outdated]
  finance_report_effect: "None."
  i18n_a11y: "Admin bilingual."
  acceptance: "AC-SF64.2.7: A DR test records measured RPO/RTO and ledger balances equal before and after failback."
  dependency: "SF64.2.1, D-648."
```

### M64 key invariants

1. Every device is enrolled, owned and linked to one M26 asset; revocation is immediate.
2. Degraded mode never double-executes money/external commands; unsafe actions are refused offline.
3. Recovery claims are only measured values from tests or incidents.

### M64 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF64.1.3, AC-SF64.2.2 | AT-G20 (network outage), AT-G14 (outage fallback) | GM: "Can we operate if internet or a provider fails?"; Front desk: "What if... internet is down?" |
| AC-SF64.1.1, AC-SF64.1.2 | AT-G03 (tested gate) | Technology: "Which integrations, connected devices, certificates... are healthy?" |
| AC-SF64.2.1, AC-SF64.2.4, AC-SF64.2.7 | AT-G20 (restore), Section B Phase 6 exit | Technology: "Can we restore to a test environment and prove recovery?" |
| AC-SF64.1.5 | AT-G03, AT-G14 | Maintenance/security: "Is the parking gate override auditable?" |

### M64 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-646 | Pilot site network profile and segmentation capability. | IT Admin (pilot) | Reference five-zone design; flat network is a site blocker for payment devices. |
| D-647 | RPO/RTO targets per service. | GM + Owner + Lead Architect | PMS core RPO 15 min / RTO 4 h (to be agreed); guest website RTO 1 h on SaaS. |
| D-648 | On-prem hardware and offsite backup location/residency. | IT Admin + DPO | Hotel server plus encrypted offsite copy in same-country cloud region where required. |
| D-649 | Support model (hotel IT L1, MetriSys L2/L3, vendor L3). | Product Owner | Tiered model with named owners per component. |

---

## M65 — Data governance and decision intelligence

| Field | Value |
|---|---|
| Purpose | Make every number trustworthy and traceable: canonical entities and duplicate review, governed metric definitions with denominators, event/source-to-ledger lineage, business-vs-accounting date mapping, freshness/coverage alerts, scoped reports and exports, point-in-time reproducibility, forecast/budget/actual with corrections, and purpose-based retention. |
| Phases | 2 definitions, scopes, date mapping, canonical registry; 4–6 governed analytics (lineage, freshness, reproducibility, flash-to-detail, scheduled exports, retention). |
| Release flag | R1. |
| Bounded context | `data-governance` |
| SoR entities (owned) | `canonical_entity_registry`, `master_data_issue`, `metric_definition`, `metric_version`, `lineage_edge`, `dataset_freshness_status`, `date_mapping_rule`, `report_definition`, `report_version`, `report_snapshot`, `scheduled_report`, `export_job`, `purpose_retention_map`, `attribution_model_version`. |
| Referenced (not owned) | M32 report rendering and suite (consumes `metric_definition`, see D-651); M19 GL periods/journals; M08 business date/night audit; M02 retention execution, legal hold, erasure; M52 guest merge execution; every module's events (lineage sources); M51 attribution; M53 forecasts; budgets in M19/M32. |
| Dependencies | M01, M02, M08, M19, M32, M51, M52, M53. |

### F65.1 Trust

```yaml
- id: M65.F65.1.SF65.1.1
  name: canonical entities and duplicate merge review
  phase: 2
  release: R1
  actors: [property_admin, finance_clerk, procurement_officer, guest_relations]
  screens: [SCR-ADM-master-data-quality]
  inputs: [entity_type, record_ids, issue_type, proposed_action, reviewer]
  states: [detected, under_review, merged_in_owner_module, rejected, fixed]
  api: ["GET /v1/properties/{pid}/master-data/issues", "POST /v1/properties/{pid}/master-data/issues/{iid}/decision"]
  events: [MasterDataIssueDetected, MasterDataIssueResolved]
  data: [canonical_entity_registry, master_data_issue]
  rules: ["Registry names the single owning module and table for each canonical entity (guest_profile M18, room M03, item M14, vendor M46, employee M27, account M19).", "Duplicate detection raises issues; the merge itself executes in the owning module (e.g. M52 for guests, M46 for vendors) with reversible links.", "Vendor duplicates by tax ID/bank details are high priority (fraud risk)."]
  security: "Reviewers only see entity types in their scope."
  failure_cases: [owner_module_rejects_merge, cross_scope_duplicate]
  finance_report_effect: "Prevents double-counting in reports and duplicate payments."
  i18n_a11y: "Accessible review UI."
  acceptance: "AC-SF65.1.1: Two vendors with the same tax ID raise one issue routed to M46; no merge happens in M65 tables."
  dependency: "M18, M46, M52, M14."

- id: M65.F65.1.SF65.1.2
  name: metric definitions/denominators and data coverage
  phase: 2
  release: R1
  actors: [financial_controller, revenue_manager, gm]
  screens: [SCR-ADM-metric-dictionary]
  inputs: [metric_key, formula, numerator, denominator, inclusions_exclusions, grain, owner, version]
  states: [draft, approved, active, superseded]
  api: ["GET /v1/metric-definitions", "POST /v1/properties/{pid}/metric-definitions/{key}/versions"]
  events: [MetricDefinitionActivated]
  data: [metric_definition, metric_version]
  rules: ["Every KPI shown anywhere references a metric_definition version (occupancy with OOO/comp/house-use treatment, ADR, RevPAR, TRevPAR, net contribution, quote conversion, recovery time, utility per occupied room, food waste per cover).", "Reports show coverage (share of source data present) with each value.", "Definition change creates a new version; historical reports keep their version unless restated."]
  security: "Approval by financial_controller for financial metrics."
  failure_cases: [conflicting_definitions_blocked]
  finance_report_effect: "Defines all M32 KPIs."
  i18n_a11y: "Definitions bilingual; tooltip accessible."
  acceptance: "AC-SF65.1.2: Changing the occupancy denominator to exclude OOO creates v2; a report run for last month still states v1 unless restated."
  dependency: "M32 SF32.1.1, D-651."

- id: M65.F65.1.SF65.1.3
  name: source-to-ledger and event lineage
  phase: 4
  release: R1
  actors: [financial_controller, auditor, owner]
  screens: [SCR-OWN-drilldown, SCR-FIN-lineage]
  inputs: [report_cell_ref, journal_entry_id, source_event_id]
  states: [traced, partial_trace, untraceable]
  api: ["GET /v1/properties/{pid}/lineage?journal_entry_id", "GET /v1/properties/{pid}/lineage?report_cell"]
  events: [LineageGapDetected]
  data: [lineage_edge]
  rules: ["Every M19 journal line links to its source document/event (folio line, GRN, invoice, payroll run, meter bill) and correlation_id.", "Report cells drill to journal lines then to source evidence; gaps flagged as untraceable.", "Lineage derived from event envelopes (causation_id/correlation_id) plus posting maps."]
  security: "Drill respects field scopes (salary lines aggregate unless payroll role)."
  failure_cases: [manual_journal_without_evidence_flagged]
  finance_report_effect: "Enables GM drill from profit to source events (AT-G19, AT-G08)."
  i18n_a11y: "Drill path accessible with breadcrumbs."
  acceptance: "AC-SF65.1.3: From the P&L laundry cost line the GM reaches the laundry invoice and M56 batch evidence; salary drill stops at department level for gm (AT-G08.3, AT-G19.8)."
  dependency: "M19 SF19.2.3, M01 event envelope."

- id: M65.F65.1.SF65.1.4
  name: business-date/accounting-date mapping
  phase: 2
  release: R1
  actors: [financial_controller, night_auditor]
  screens: [SCR-FIN-date-mapping]
  inputs: [business_date, accounting_period, cutoff_rules, late_posting_policy]
  states: [open, closed, reopened]
  api: ["GET /v1/properties/{pid}/date-mapping?business_date"]
  events: [DateMappingApplied]
  data: [date_mapping_rule]
  rules: ["Each transaction carries business_date (M08) and accounting_date (M19); rules map late postings after period close to next open period with reference.", "Reports declare which date basis they use."]
  security: "financial_controller."
  failure_cases: [period_closed_late_charge]
  finance_report_effect: "Consistent period reporting."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF65.1.4: A minibar late charge after month close posts to the next period with a link and appears in the correct business-date operational report."
  dependency: "M08, M19 SF19.1.3."

- id: M65.F65.1.SF65.1.5
  name: missing-data and stale-feed alert
  phase: 4
  release: R1
  actors: [it_admin, financial_controller, gm, governance_worker]
  screens: [SCR-OWN-data-health, SCR-ADM-data-freshness]
  inputs: [dataset, expected_cadence, last_received_at, expected_count, received_count]
  states: [fresh, late, stale, missing]
  api: ["GET /v1/properties/{pid}/data-freshness"]
  events: [DatasetStale, DatasetMissing]
  data: [dataset_freshness_status]
  rules: ["Each feed (meter intervals, utility bills, PSP settlement, channel statements, POS, payroll) has expected cadence; late/missing raises alert and marks dependent KPIs 'incomplete estimate'.", "Missing cost source never shows as certified actual (AT-G08)."]
  security: "Aggregates."
  failure_cases: [cadence_misconfigured]
  finance_report_effect: "KPI labels estimate/reconciled per Section 3.6."
  i18n_a11y: "Status icons with text."
  acceptance: "AC-SF65.1.5: A missed meter interval marks utility per occupied room as incomplete estimate for that day (AT-G20.4, AT-G08.4)."
  dependency: "M22–M25, M28, M07."
```

### F65.2 Answers

```yaml
- id: M65.F65.2.SF65.2.1
  name: role/row/field filters on reports and exports
  phase: 2
  release: R1
  actors: [gm, owner, financial_controller, payroll_officer, auditor]
  screens: [SCR-OWN-reports, SCR-FIN-reports]
  inputs: [report_definition_id, user_scopes, field_classifications]
  states: [rendered, redacted]
  api: ["GET /v1/properties/{pid}/reports/{rid}"]
  events: [ReportViewed, ReportFieldRedacted]
  data: [report_definition]
  rules: ["Row filters by tenant/property/department; field filters for salary, ID, health, card data.", "Exports apply the same filters; small-cell suppression for individual inference where configured."]
  security: "Server-side enforcement; tested against OWASP API object-level authorization."
  failure_cases: [cached_report_scope_leak_prevented]
  finance_report_effect: "Salary confidentiality (Section D)."
  i18n_a11y: "Accessible tables."
  acceptance: "AC-SF65.2.1: GM export of labor cost contains department totals only; payroll_officer export contains individual lines (AT-G05.2)."
  dependency: "M02 scopes, M27."

- id: M65.F65.2.SF65.2.2
  name: point-in-time reproducible version
  phase: 4
  release: R1
  actors: [financial_controller, auditor, owner]
  screens: [SCR-FIN-report-snapshots]
  inputs: [report_definition_version, parameters, as_of_timestamp]
  states: [generated, restated]
  api: ["POST /v1/properties/{pid}/reports/{rid}/snapshots", "GET /v1/properties/{pid}/report-snapshots/{sid}"]
  events: [ReportSnapshotCreated, ReportRestated]
  data: [report_version, report_snapshot]
  rules: ["Snapshots store definition version, metric versions, parameters, data as-of and a content hash.", "Restatements create new snapshots referencing the old with reason."]
  security: "Snapshots immutable."
  failure_cases: [source_purged_by_retention_note]
  finance_report_effect: "Owner and audit statements reproducible."
  i18n_a11y: "Accessible PDF."
  acceptance: "AC-SF65.2.2: Regenerating a closed-month snapshot yields the same hash; a restatement shows both versions."
  dependency: "SF65.1.2, M19 close."

- id: M65.F65.2.SF65.2.3
  name: daily flash to detail
  phase: 4
  release: R1
  actors: [gm, owner, duty_manager]
  screens: [SCR-OWN-daily-flash, SCR-OWN-drilldown]
  inputs: [business_date, comparison_basis]
  states: [provisional, final_after_audit]
  api: ["GET /v1/properties/{pid}/flash?business_date"]
  events: [DailyFlashPublished]
  data: [report_snapshot]
  rules: ["Flash shows occupancy, ADR, RevPAR, TRevPAR, revenue by department, key costs, cash, open exceptions, each with source/freshness badge and drill link.", "Provisional before night audit; final after."]
  security: "Scoped."
  failure_cases: [night_audit_late_provisional]
  finance_report_effect: "Consumes M32/M19."
  i18n_a11y: "Mobile-friendly, accessible, bilingual."
  acceptance: "AC-SF65.2.3: Each flash tile links to its drill path and shows freshness; before audit it is labelled provisional (AT-G19.8)."
  dependency: "M32 SF32.4.1, SF65.1.3."

- id: M65.F65.2.SF65.2.4
  name: scheduled report/export/recipient consent
  phase: 4
  release: R1
  actors: [gm, financial_controller, owner]
  screens: [SCR-OWN-scheduled-reports]
  inputs: [report_id, schedule, format, recipients, delivery_channel]
  states: [active, paused, failed_delivery]
  api: ["POST /v1/properties/{pid}/scheduled-reports"]
  events: [ScheduledReportDelivered, ScheduledReportFailed]
  data: [scheduled_report, export_job]
  rules: ["Recipients must be platform users with scope for the report content, or external recipients approved with data-sharing note (e.g. owner's accountant).", "Links expire; attachments only for non-sensitive reports."]
  security: "Signed expiring links; no salary/ID data to external recipients."
  failure_cases: [recipient_scope_removed_auto_pause]
  finance_report_effect: "None."
  i18n_a11y: "PDF/CSV/XLSX accessible, bilingual headers."
  acceptance: "AC-SF65.2.4: Removing a recipient's scope pauses their subscription automatically."
  dependency: "M32 SF32.4.8."

- id: M65.F65.2.SF65.2.5
  name: forecast vs budget vs actual and correction
  phase: 5
  release: R1
  actors: [financial_controller, gm, owner]
  screens: [SCR-OWN-budget-vs-actual]
  inputs: [period, budget_version, forecast_run_id, actuals_basis]
  states: [computed, restated]
  api: ["GET /v1/properties/{pid}/variance?period"]
  events: [VarianceReportComputed]
  data: [report_snapshot]
  rules: ["Actual (closed-period M19), estimate (open period), forecast (M53) and budget each labelled; variances explained by drill.", "Corrections to actuals appear as restatements with reason."]
  security: "Scoped."
  failure_cases: [budget_version_missing]
  finance_report_effect: "Owner budget/actual reporting."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF65.2.5: An open-period value is labelled estimate and cannot be exported as certified actual."
  dependency: "M19, M53, M32 SF32.3.3."

- id: M65.F65.2.SF65.2.6
  name: retention and deletion by purpose
  phase: 4
  release: R1
  actors: [dpo, financial_controller, compliance_officer]
  screens: [SCR-ADM-retention-map]
  inputs: [record_type, purpose, jurisdiction, retention_period, legal_basis, deletion_method]
  states: [draft, approved, active]
  api: ["PUT /v1/properties/{pid}/retention-map/{record_type}"]
  events: [RetentionRuleActivated, PurposeDeletionExecuted]
  data: [purpose_retention_map]
  rules: ["Each record type maps purposes to retention per jurisdiction (M44); accounting, tax, safety and incident evidence retained per law; marketing and analytics data deleted earlier.", "Deletion executed by M02 jobs; M65 verifies completion and records proof; legal hold overrides.", "Analytics aggregates are anonymized before retention of personal data expires."]
  security: "DPO approval."
  failure_cases: [conflicting_retention_rules_longest_lawful_wins_with_dpo_review]
  finance_report_effect: "Financial records retained; reports remain reproducible via aggregates."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF65.2.6: Purpose-based deletion removes marketing data while folio and incident evidence remain (Section O technology question)."
  dependency: "M02, M44, D-652."

- id: M65.F65.2.SF65.2.7  # ADDED — Section C 'profitability and attribution'
  name: Profitability and attribution model governance
  phase: 5
  release: R1
  actors: [financial_controller, revenue_manager, owner]
  screens: [SCR-ADM-attribution-models]
  inputs: [model_type, allocation_drivers, attribution_precedence, version, approver]
  states: [draft, approved, active, superseded]
  api: ["POST /v1/properties/{pid}/attribution-models"]
  events: [AttributionModelActivated]
  data: [attribution_model_version]
  rules: ["Channel/campaign attribution precedence (M51) and shared-cost allocation drivers (M19/M32) are versioned models with owner and approval.", "Reports state model version; changing a model restates only on request."]
  security: "financial_controller approval."
  failure_cases: [model_change_mid_period]
  finance_report_effect: "Departmental and channel profitability consistency."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF65.2.7: Channel net contribution report cites the active attribution model version."
  dependency: "M51 SF51.2.5, M32 SF32.3.1."
```

### M65 key invariants

1. One canonical owner per entity; M65 never stores a second copy to merge.
2. Every KPI cites a metric version, coverage and freshness; missing sources show as incomplete estimates.
3. Reports are reproducible (hash) and scope-filtered, including exports.
4. Deletion is purpose-based and never destroys legally required financial/safety evidence.

### M65 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF65.1.3, AC-SF65.2.3 | AT-G08, AT-G19 (GM drill from occupancy and profit to source events) | Owner: "What changed since yesterday/month/year? What is accrued versus settled?" |
| AC-SF65.1.5, AC-SF65.2.5 | AT-G08, AT-G20 (missed meter interval) | Owner: "Is this property profitable after payroll, energy...?" (honest estimates) |
| AC-SF65.2.1 | AT-G05 | Finance/HR: "Who can see salary and ID?" |
| AC-SF65.2.6 | AT-G20 | Technology/compliance: "Can we delete data by purpose without deleting required accounting and safety evidence?" |

### M65 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-650 | Analytics store (Postgres reporting schema/replica vs separate warehouse). | Lead Architect | Postgres reporting schema with materialized views in R1. |
| D-651 | KPI dictionary ownership M32 vs M65. | Lead Architect + Financial Controller | M65 owns `metric_definition`; M32 renders. |
| D-652 | Retention schedule per record purpose per market. | DPO + counsel | Longest plausible statutory period for finance/tax (10 years) pending counsel; marketing 24 months after last stay. |

---

## M66 — Owner, brand and property lifecycle

| Field | Value |
|---|---|
| Purpose | Represent ownership and management economics (owner/manager/franchise relationships, fee schedules, owner statements, lease/debt inputs), and property change (capex case, renovation closures, capitalization/depreciation, new outlets/rooms, brand standards and reopening) for a single property in R1, portfolio later. |
| Phases | 4–6 single-property owner reporting and lifecycle; 7 portfolio. |
| Release flag | R1; `Later` for SF66.2.6. |
| Bounded context | `owner-lifecycle` |
| SoR entities (owned) | `ownership_relationship`, `management_agreement`, `fee_schedule`, `fee_calculation`, `owner_statement`, `external_obligation_input`, `capex_request`, `investment_case`, `renovation_project`, `capitalization_decision`, `brand_standard`, `brand_audit_evidence`, `reopening_checklist`. |
| Referenced (not owned) | M01 legal entity/property; M19 GL, fixed-asset postings and depreciation journals (see D-654); M20 AP/payments; M26 physical assets, warranties and work orders; M03 room inventory and OOO; M09/M13 outlets/facilities; M21/M49 procurement; M32 P&L; M65 snapshots; M61 inspections; M68 insurance. |
| Dependencies | M01, M03, M19, M20, M26, M32, M49, M61, M65. |

### F66.1 Contract economics

```yaml
- id: M66.F66.1.SF66.1.1
  name: owner/manager/franchise legal relationship
  phase: 4
  release: R1
  actors: [owner, financial_controller, tenant_admin]
  screens: [SCR-OWN-relationships]
  inputs: [legal_entity_id, relationship_type, counterparty, effective_from, effective_to, contract_document_ref]
  states: [draft, active, expired, terminated]
  api: ["POST /v1/properties/{pid}/ownership-relationships"]
  events: [OwnershipRelationshipActivated]
  data: [ownership_relationship, management_agreement]
  rules: ["relationship_type is owner_operated, management_contract, franchise or lease.", "Exactly one active owner of record per property per date; managers/franchisors as counterparties.", "Contract documents stored with restricted access."]
  security: "owner and financial_controller only."
  failure_cases: [overlapping_relationships]
  finance_report_effect: "Determines which fees and statements apply."
  i18n_a11y: "Bilingual."
  acceptance: "AC-SF66.1.1: Two overlapping active owners of record for the same dates are rejected."
  dependency: "M01 legal entity, D-653."

- id: M66.F66.1.SF66.1.2
  name: management/brand fee schedule and approval
  phase: 4
  release: R1
  actors: [financial_controller, owner]
  screens: [SCR-FIN-fee-schedules]
  inputs: [agreement_id, fee_type, basis_metric, rate, thresholds, calculation_period, approver]
  states: [draft, approved, calculated, invoiced, paid]
  api: ["POST /v1/properties/{pid}/fee-schedules", "POST /v1/properties/{pid}/fee-calculations"]
  events: [FeeCalculated, FeeApproved]
  data: [fee_schedule, fee_calculation]
  rules: ["fee_type examples: base management fee on total revenue, incentive fee on GOP, royalty on rooms revenue, marketing fund, reservation fee.", "Basis metric references M65 metric versions; calculation shows inputs and is approved before AP/accrual.", "Estimates in open periods labelled."]
  security: "Owner/finance only."
  failure_cases: [basis_restated]
  finance_report_effect: "Fee accrual and payable via M19/M20; management fees below GOP per profit bridge."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF66.1.2: A 3% base fee on fixture total revenue calculates to the minor unit and posts an accrual after approval."
  dependency: "M65, M19, M20."

- id: M66.F66.1.SF66.1.3
  name: owner statement and restricted access
  phase: 5
  release: R1
  actors: [owner, gm, financial_controller]
  screens: [SCR-OWN-owner-statement]
  inputs: [period, statement_template, snapshot_id]
  states: [draft, reviewed, issued, restated]
  api: ["POST /v1/properties/{pid}/owner-statements", "GET /v1/properties/{pid}/owner-statements/{sid}"]
  events: [OwnerStatementIssued]
  data: [owner_statement, report_snapshot (M65)]
  rules: ["Statement generated from a reproducible M65 snapshot with estimate vs reconciled labels and profit bridge (operating result vs net income).", "Access limited to owner, gm, financial_controller; management company reporting variant for managed hotels."]
  security: "Row/field scopes; no individual salaries."
  failure_cases: [period_not_closed_draft_only]
  finance_report_effect: "Owner reporting."
  i18n_a11y: "Accessible PDF; bilingual."
  acceptance: "AC-SF66.1.3: An owner statement cannot be issued for an open period except as a labelled draft (AT-G08.5)."
  dependency: "M65 SF65.2.2, M32 SF32.3.2."

- id: M66.F66.1.SF66.1.4
  name: lease or debt inputs only from authorized source
  phase: 4
  release: R1
  actors: [financial_controller, owner]
  screens: [SCR-FIN-external-obligations]
  inputs: [obligation_type, counterparty, schedule, source_document, entered_by, approved_by]
  states: [entered, approved, active, ended]
  api: ["POST /v1/properties/{pid}/external-obligations"]
  events: [ExternalObligationApproved]
  data: [external_obligation_input]
  rules: ["Lease/debt schedules entered only from signed documents or lender/lessor statements with maker-checker.", "The system does not calculate lease accounting beyond approved schedules unless finance policy configures it."]
  security: "Finance only."
  failure_cases: [schedule_mismatch_with_statement]
  finance_report_effect: "Debt service and lease payments shown in owner cash view (Section O owner: 'what cash, debt... obligations are due')."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF66.1.4: An obligation without source document cannot be approved."
  dependency: "M19, M20."

- id: M66.F66.1.SF66.1.5
  name: audit and year-end export
  phase: 6
  release: R1
  actors: [financial_controller, auditor, owner]
  screens: [SCR-FIN-year-end-pack]
  inputs: [fiscal_year, pack_contents]
  states: [assembling, ready, delivered]
  api: ["POST /v1/properties/{pid}/year-end-packs"]
  events: [YearEndPackReady]
  data: [report_snapshot (M65), owner_statement]
  rules: ["Pack includes trial balance, fee calculations, fixed-asset register, owner statements, and evidence index with hashes."]
  security: "Auditor read-only access link, expiring."
  failure_cases: [unclosed_period]
  finance_report_effect: "External audit support."
  i18n_a11y: "Accessible exports."
  acceptance: "AC-SF66.1.5: Year-end pack generation fails with a list of open periods if any month is unclosed."
  dependency: "M19 period close."
```

### F66.2 Changes

```yaml
- id: M66.F66.2.SF66.2.1
  name: capex request, budget and investment case
  phase: 4
  release: R1
  actors: [gm, chief_engineer, financial_controller, owner]
  screens: [SCR-OWN-capex-requests]
  inputs: [title, scope, estimated_cost, quotes (M49), expected_benefits, payback_assumptions, risk, budget_line]
  states: [draft, submitted, approved, rejected, in_execution, completed, post_review]
  api: ["POST /v1/properties/{pid}/capex-requests", "POST /v1/properties/{pid}/capex-requests/{cid}/decision"]
  events: [CapexApproved, CapexPostReviewed]
  data: [capex_request, investment_case]
  rules: ["Investment case lists assumptions and a sensitivity range; savings/uplift are projections, not guarantees.", "Approval thresholds by amount (gm, owner); procurement through M21/M49.", "Post-completion review compares projected vs measured (M67 for energy savings)."]
  security: "Owner/finance/gm."
  failure_cases: [budget_exceeded_reapproval]
  finance_report_effect: "Capex budget encumbrance; capitalization via SF66.2.3."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF66.2.1: Capex above gm threshold routes to owner; post-review shows projected vs measured with labels (Section O owner: 'Which capex will pay back?')."
  dependency: "M49, M67 SF67.2.3, D-655."

- id: M66.F66.2.SF66.2.2
  name: renovation room closure/resale date
  phase: 4
  release: R1
  actors: [gm, revenue_manager, front_office_manager]
  screens: [SCR-OWN-renovation-plan, SCR-REV-renovation-impact]
  inputs: [project_id, room_ids, closure_start, planned_resale_date, affected_reservations]
  states: [planned, rooms_blocked, in_progress, delayed, reopened]
  api: ["POST /v1/properties/{pid}/renovation-projects/{rid}/room-closures"]
  events: [RenovationRoomsClosed, RenovationResaleDateChanged]
  data: [renovation_project, room_block (M03)]
  rules: ["Closures create M03 OOO blocks with reason renovation; conflicting reservations listed for relocation before confirmation.", "Resale date change updates M03 blocks and M53 supply."]
  security: "gm approval."
  failure_cases: [reservation_conflict, delay]
  finance_report_effect: "OOO rooms excluded per occupancy definition; displacement estimate shown."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF66.2.2: Closing 10 rooms with 3 conflicting reservations blocks confirmation until relocations are handled."
  dependency: "M03, M53."

- id: M66.F66.2.SF66.2.3
  name: asset capitalization/depreciation and warranty
  phase: 4
  release: R1
  actors: [financial_controller, chief_engineer]
  screens: [SCR-FIN-capitalization]
  inputs: [invoice_lines, asset_id (M26), capitalize_flag, useful_life, method, warranty_terms]
  states: [pending_decision, capitalized, expensed, depreciating, disposed]
  api: ["POST /v1/properties/{pid}/capitalization-decisions"]
  events: [AssetCapitalized, DepreciationPosted]
  data: [capitalization_decision, asset (M26), journal_entry (M19)]
  rules: ["Capitalization decision per policy threshold; depreciation journals generated monthly via M19.", "Warranty terms stored on M26 asset; claims routed via M26."]
  security: "financial_controller."
  failure_cases: [policy_threshold_missing]
  finance_report_effect: "Fixed-asset register and depreciation below GOP."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF66.2.3: A capitalized renovation invoice generates monthly depreciation journals that sum to cost minus residual over the life."
  dependency: "M19, M26, M21 SF21.2.4, D-654."

- id: M66.F66.2.SF66.2.4
  name: new outlet/room/facility configuration
  phase: 4
  release: R1
  actors: [property_admin, gm, financial_controller]
  screens: [SCR-ADM-property-changes]
  inputs: [change_type, room_type_or_outlet_details, effective_date, gl_mapping, feature_flags]
  states: [planned, configured, tested, live]
  api: ["POST /v1/properties/{pid}/property-changes"]
  events: [PropertyChangeLive]
  data: [reopening_checklist]
  rules: ["New rooms/outlets are created in owning modules (M03/M09/M13) with effective date; M66 tracks the change and checklist.", "Go-live requires GL mapping, tax setup (M44), inventory and media (M39)."]
  security: "property_admin with gm approval."
  failure_cases: [missing_gl_mapping]
  finance_report_effect: "New department/outlet reporting from effective date."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF66.2.4: A new outlet cannot go live without GL mapping and tax setup."
  dependency: "M03, M09, M13, M44, M39."

- id: M66.F66.2.SF66.2.5
  name: brand standards evidence and reopening test
  phase: 5
  release: R1
  actors: [gm, content_approver, compliance_officer]
  screens: [SCR-OWN-brand-standards, SCR-OWN-reopening-checklist]
  inputs: [standard_id, evidence_items, audit_result, reopening_checks]
  states: [compliant, noncompliant, waived, reopening_ready]
  api: ["POST /v1/properties/{pid}/brand-audits", "POST /v1/properties/{pid}/reopening-checklists/{cid}/complete"]
  events: [BrandAuditRecorded, ReopeningApproved]
  data: [brand_standard, brand_audit_evidence, reopening_checklist]
  rules: ["Brand standards apply only if a franchise/brand relationship exists; evidence captured with photos and dates.", "Reopening requires M61 inspections passed, M64 devices tested, M03 blocks released and staff briefed (M62)."]
  security: "gm."
  failure_cases: [inspection_failed]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF66.2.5: Reopening is blocked while any M61 critical finding for the area is open."
  dependency: "M61, M64, M62, M03."

- id: M66.F66.2.SF66.2.6  # ADDED — Section C 'multi-property extension later'; Phase 7 portfolio
  name: Portfolio owner reporting across properties
  phase: 7
  release: Later
  actors: [owner, tenant_admin]
  screens: [SCR-OWN-portfolio]
  inputs: [property_ids, period, currency_basis]
  states: [computed]
  api: ["GET /v1/tenants/{tid}/portfolio/owner-summary"]
  events: [PortfolioSummaryComputed]
  data: [owner_statement, report_snapshot (M65)]
  rules: ["Consolidation only across properties the owner holds; FX basis declared.", "Property-level scopes preserved."]
  security: "Tenant-level owner scope."
  failure_cases: [fx_missing]
  finance_report_effect: "Portfolio view."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF66.2.6: Owner of two properties sees consolidated view; a user scoped to one property cannot access it."
  dependency: "Phase 7 multi-property."
```

### M66 key invariants

1. One owner of record per property per date; fees computed from versioned metrics with approval.
2. Renovation closures are M03 blocks; asset records live in M26 and journals in M19.
3. Investment cases state projections, never guaranteed payback.

### M66 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF66.1.2, AC-SF66.1.3 | AT-G08 | Owner: "Is this property profitable...? What is accrued versus settled?" |
| AC-SF66.1.4 | AT-G08 | Owner: "What cash, debt, insurance and tax obligations are due?" |
| AC-SF66.2.1 | AT-G08 | Owner: "Which capex/renovation will pay back?" |

### M66 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-653 | Pilot ownership structure (owner-operated, managed, franchised). | Owner | Owner-operated; fee schedules specified and tested on fixtures. |
| D-654 | Fixed-asset register ownership (M19 vs M26 vs M66) and depreciation method. | Financial Controller + Lead Architect | M26 physical asset; M66 capitalization decision; M19 depreciation journals; straight-line. |
| D-655 | Capex approval thresholds and payback method. | Owner + GM | gm up to 5,000 OMR-equivalent; above -> owner; simple payback and NPV shown as projections. |

---

## M67 — Sustainability and resource performance

| Field | Value |
|---|---|
| Purpose | Measure energy, water, gas, laundry and food waste from traceable sources with verified/estimated/missing status, normalize by occupancy/guests/covers, set targets against baselines, turn anomalies into work orders, measure capex savings with context, and export evidence **without any unverified green claim**. |
| Phases | 4 utilities/waste measurement and anomalies; 5–6 dashboards, targets, emission factors, savings, assurance export. |
| Release flag | R1. |
| Bounded context | `sustainability` |
| SoR entities (owned) | `resource_metric_series`, `metric_data_status`, `meter_quality_score`, `intensity_snapshot`, `emission_factor_set`, `sustainability_target`, `sustainability_baseline`, `savings_measurement`, `sustainability_export`, `claim_review`, `supplier_sustainability_evidence`. |
| Referenced (not owned) | M22 electricity, M23 water, M24 pipeline gas, M25 cylinders (meters, intervals, bills — the utility SoR); M56 laundry batches/weights; M50/M57 waste transactions; M03/M05/M13 occupied rooms, guests, covers; M26 work orders; M66 capex; M46 vendor documents; M44 jurisdiction rules; M65 metric definitions/freshness. |
| Dependencies | M22, M23, M24, M25, M26, M46, M50, M56, M57, M65, M66. |

### F67.1 Measure

```yaml
- id: M67.F67.1.SF67.1.1
  name: utility and waste source meter/vendor
  phase: 4
  release: R1
  actors: [chief_engineer, financial_controller, sustainability_worker]
  screens: [SCR-ENG-resource-sources]
  inputs: [resource_type, source_ref, meter_id_or_bill_account, waste_vendor_id, unit, cadence]
  states: [mapped, unmapped, retired]
  api: ["GET /v1/properties/{pid}/sustainability/sources", "PUT /v1/properties/{pid}/sustainability/sources/{sid}"]
  events: [ResourceSourceMapped]
  data: [resource_metric_series, meter (M22/M23/M24), waste_ledger_view (M50)]
  rules: ["Series are derived views over M22–M25 readings/bills and M50 waste transactions; M67 stores no independent readings.", "Waste haulage weights from vendor tickets via M46/M49 evidence.", "Unmapped consumption sources are listed as coverage gaps."]
  security: "Engineering/finance scope."
  failure_cases: [meter_replaced_series_break]
  finance_report_effect: "None; costs remain in M19 via utility modules."
  i18n_a11y: "Units localized; accessible tables."
  acceptance: "AC-SF67.1.1: A kitchen submeter reading in M22 appears in the M67 series without duplication; an unmapped building meter shows as a coverage gap."
  dependency: "M22–M25, M50."

- id: M67.F67.1.SF67.1.2
  name: missing/estimated/verified value
  phase: 4
  release: R1
  actors: [chief_engineer, financial_controller]
  screens: [SCR-ENG-resource-data-status]
  inputs: [period, series_id, value, status, estimation_method]
  states: [missing, estimated, measured, bill_verified]
  api: ["GET /v1/properties/{pid}/sustainability/series/{sid}/status"]
  events: [ResourceValueEstimated, ResourceValueVerified]
  data: [metric_data_status]
  rules: ["Every period value carries one status; estimates state method (interpolation, bill pro-rata) and are replaced when data arrives.", "Bill-verified = meter total reconciled to supplier bill within tolerance (M22 SF22.1.6)."]
  security: "Read scoped."
  failure_cases: [bill_meter_mismatch]
  finance_report_effect: "Shares status with M65 freshness."
  i18n_a11y: "Status text plus icon."
  acceptance: "AC-SF67.1.2: A missed interval shows 'estimated (interpolation)' until the reading arrives, then 'measured' (AT-G20.4)."
  dependency: "M65 SF65.1.5."

- id: M67.F67.1.SF67.1.3
  name: per occupied room/guest/cover intensity
  phase: 4
  release: R1
  actors: [gm, owner, chief_engineer]
  screens: [SCR-OWN-resource-intensity]
  inputs: [period, resource, denominator_metric]
  states: [computed]
  api: ["GET /v1/properties/{pid}/sustainability/intensity?period"]
  events: [IntensityComputed]
  data: [intensity_snapshot]
  rules: ["Denominators from M65 metric definitions (occupied rooms, guest nights, covers); status of numerator propagates.", "Utility cost per occupied room shown next to consumption intensity."]
  security: "Aggregates."
  failure_cases: [zero_occupancy_period]
  finance_report_effect: "Feeds Section P.6 'utility per occupied room'."
  i18n_a11y: "Accessible charts."
  acceptance: "AC-SF67.1.3: kWh per occupied room for a fixture month equals meter total / occupied rooms and inherits 'estimated' if any interval is estimated (AT-G19.8)."
  dependency: "M65."

- id: M67.F67.1.SF67.1.4
  name: food waste and laundry usage
  phase: 4
  release: R1
  actors: [executive_chef, housekeeping_supervisor, gm]
  screens: [SCR-OWN-waste-laundry]
  inputs: [period, waste_reasons, laundry_kg, covers, occupied_rooms]
  states: [computed]
  api: ["GET /v1/properties/{pid}/sustainability/waste-laundry?period"]
  events: [WasteLaundryComputed]
  data: [resource_metric_series]
  rules: ["Food waste from M50/M57 waste transactions by reason (spoilage, overproduction, plate return) per cover; laundry kg per occupied room from M56 batches.", "Weights estimated from counts are labelled estimated."]
  security: "Aggregates."
  failure_cases: [waste_not_weighed]
  finance_report_effect: "Waste cost from M50 alongside quantities."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF67.1.4: Waste per cover and laundry kg per occupied room reproduce fixture values (AT-G19.8 GM drill)."
  dependency: "M50, M56, M57, D-658."

- id: M67.F67.1.SF67.1.5
  name: emission-factor source/version only after review
  phase: 5
  release: R1
  actors: [compliance_officer, financial_controller]
  screens: [SCR-ADM-emission-factors]
  inputs: [factor_set_name, source_citation, geography, year, values, reviewer, review_date]
  states: [draft, reviewed, active, superseded, rejected]
  api: ["POST /v1/properties/{pid}/sustainability/emission-factor-sets"]
  events: [EmissionFactorSetActivated]
  data: [emission_factor_set]
  rules: ["No emissions figures computed until a factor set with cited source and named reviewer is active.", "Each emissions value states factor set version; changes create restatement notes."]
  security: "compliance_officer approval."
  failure_cases: [no_reviewed_factor_set_emissions_hidden]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF67.1.5: Without an active reviewed factor set, emissions tiles show 'not calculated: no reviewed factor set'."
  dependency: "D-656."

- id: M67.F67.1.SF67.1.6  # ADDED — Section C 'meter quality'
  name: Meter data quality score
  phase: 4
  release: R1
  actors: [chief_engineer]
  screens: [SCR-ENG-meter-quality]
  inputs: [meter_id, gaps, resets, outliers, calibration_date]
  states: [good, fair, poor]
  api: ["GET /v1/properties/{pid}/sustainability/meter-quality"]
  events: [MeterQualityDegraded]
  data: [meter_quality_score]
  rules: ["Score from gap %, reset count, outlier count and calibration age; poor meters flagged and excluded from targets until fixed.", "Degradation creates an M26 work order suggestion."]
  security: "Engineering."
  failure_cases: [calibration_unknown]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF67.1.6: A meter with 20% gaps is scored poor and excluded from target tracking with a visible note."
  dependency: "M22 SF22.1.4, M26."
```

### F67.2 Improve

```yaml
- id: M67.F67.2.SF67.2.1
  name: target and baseline
  phase: 5
  release: R1
  actors: [owner, gm, chief_engineer]
  screens: [SCR-OWN-sustainability-targets]
  inputs: [resource, baseline_period, normalization, target_value, target_date, approver]
  states: [draft, approved, active, achieved, missed]
  api: ["POST /v1/properties/{pid}/sustainability/targets"]
  events: [SustainabilityTargetActivated]
  data: [sustainability_target, sustainability_baseline]
  rules: ["Baseline requires minimum data coverage and verified status share; otherwise target is 'provisional'.", "Targets are internal goals; external publication follows SF67.2.4."]
  security: "Owner/gm approval."
  failure_cases: [insufficient_baseline_data]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF67.2.1: A baseline with under 80% verified coverage marks the target provisional."
  dependency: "D-657."

- id: M67.F67.2.SF67.2.2
  name: anomaly-to-work-order
  phase: 4
  release: R1
  actors: [chief_engineer, engineer, sustainability_worker]
  screens: [SCR-ENG-resource-anomalies]
  inputs: [series_id, anomaly_rule, detected_value, expected_range]
  states: [detected, work_order_created, dismissed, resolved]
  api: ["GET /v1/properties/{pid}/sustainability/anomalies", "POST /v1/properties/{pid}/sustainability/anomalies/{aid}/work-order"]
  events: [ResourceAnomalyDetected]
  data: [resource_metric_series, work_order (M26)]
  rules: ["Night-time water flow above threshold or kWh spike relative to occupancy-adjusted expectation creates an anomaly; engineer converts to M26 work order or dismisses with reason.", "Water leak anomalies reuse M23 SF23.1.4 path."]
  security: "Engineering."
  failure_cases: [false_positive_dismissed]
  finance_report_effect: "Avoided cost estimated, labelled estimate."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF67.2.2: A simulated 03:00 water flow spike creates an anomaly and one M26 work order when accepted."
  dependency: "M23, M26."

- id: M67.F67.2.SF67.2.3
  name: capex savings measurement with occupancy/weather context
  phase: 5
  release: R1
  actors: [chief_engineer, financial_controller, owner]
  screens: [SCR-OWN-savings-measurement]
  inputs: [capex_request_id (M66), pre_period, post_period, normalization_variables, weather_source]
  states: [planned, measuring, measured, inconclusive]
  api: ["POST /v1/properties/{pid}/sustainability/savings-measurements"]
  events: [SavingsMeasured]
  data: [savings_measurement]
  rules: ["Savings = normalized baseline minus measured post-period consumption, with occupancy and (where a licensed/official weather source exists) degree-day adjustment; result with uncertainty.", "Results reported as measured savings with method, never as guaranteed; inconclusive when uncertainty exceeds effect."]
  security: "Management."
  failure_cases: [weather_data_unavailable_occupancy_only]
  finance_report_effect: "Feeds M66 capex post-review."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF67.2.3: A LED retrofit fixture with 3% effect and 5% uncertainty is reported inconclusive."
  dependency: "M66 SF66.2.1."

- id: M67.F67.2.SF67.2.4
  name: assurance/export and no unverified green claim
  phase: 6
  release: R1
  actors: [compliance_officer, marketing_manager, gm]
  screens: [SCR-OWN-sustainability-export, SCR-MKT-claim-review]
  inputs: [export_scope, period, claim_text, evidence_refs, reviewer]
  states: [draft, evidence_linked, approved, rejected, published]
  api: ["POST /v1/properties/{pid}/sustainability/exports", "POST /v1/properties/{pid}/sustainability/claims/{cid}/review"]
  events: [SustainabilityExportGenerated, SustainabilityClaimApproved, SustainabilityClaimRejected]
  data: [sustainability_export, claim_review]
  rules: ["Exports include values, statuses, methods, factor versions and evidence index.", "Any public sustainability statement (website M51, campaigns M52) requires a claim_review with evidence and compliance approval; claims like 'carbon neutral', 'eco-friendly', 'green hotel' are blocked unless backed by approved evidence/certification.", "No certification logo displayed without a certificate record and expiry."]
  security: "compliance_officer approval."
  failure_cases: [claim_without_evidence_blocked, certificate_expired_logo_removed]
  finance_report_effect: "None."
  i18n_a11y: "Accessible export; bilingual."
  acceptance: "AC-SF67.2.4: A website content version containing 'carbon neutral' without an approved claim_review fails publish validation in M51."
  dependency: "M51 SF51.1.5, M52 SF52.1.5."

- id: M67.F67.2.SF67.2.5
  name: supplier sustainability evidence and expiry
  phase: 5
  release: R1
  actors: [procurement_officer, vendor_user, compliance_officer]
  screens: [SCR-VND-sustainability-documents, SCR-OPS-supplier-evidence]
  inputs: [vendor_id, evidence_type, document, issuer, expiry]
  states: [submitted, verified, expired, rejected]
  api: ["POST /v1/vendors/{vid}/sustainability-evidence"]
  events: [SupplierEvidenceVerified, SupplierEvidenceExpired]
  data: [supplier_sustainability_evidence, vendor_document (M46)]
  rules: ["Stored as M46 vendor documents typed 'sustainability'; M67 tracks verification and use.", "Expired evidence cannot support claims or RFQ weighting."]
  security: "Vendor sees own documents."
  failure_cases: [unverifiable_issuer]
  finance_report_effect: "None."
  i18n_a11y: "Vendor app bilingual."
  acceptance: "AC-SF67.2.5: Expired supplier certificate removes that supplier from any claim evidence and RFQ sustainability criterion."
  dependency: "M46, M49 SF49.2.2."
```

### M67 key invariants

1. No independent resource readings: series derive from M22–M25/M50/M56 SoR.
2. Every value carries missing/estimated/measured/verified status.
3. No emissions without a reviewed factor set; no public green claim without approved evidence.
4. Savings are measured with method and uncertainty, never guaranteed.

### M67 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF67.1.3, AC-SF67.1.4 | AT-G19 (GM drill: laundry, food waste, utilities), AT-G08 | Owner: "Is this property profitable after... energy...?"; Chef: "what should recipe use versus actual" (waste) |
| AC-SF67.1.2 | AT-G20 (missed meter interval) | Technology: "Where are outages...?" |
| AC-SF67.2.3 | AT-G08 | Owner: "Which capex/renovation will pay back?" |
| AC-SF67.2.4 | AT-G19 (accurate website) | Marketing: "accurate listings" |

### M67 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-656 | Emission-factor sources and any reporting framework per market. | Compliance Officer | Emissions not calculated in R1 until a reviewed factor set exists. |
| D-657 | Baseline year and normalization variables. | Chief Engineer + GM | First 12 months of verified data; occupancy normalization only. |
| D-658 | Food-waste measurement method (weighing vs estimate). | Executive Chef | Weighed bins at kitchen for spoilage/overproduction; plate returns estimated by portion. |

---

## M68 — Enterprise risk, insurance and continuity

| Field | Value |
|---|---|
| Purpose | Insurance register (policies, insured assets/risks, premiums, claims with evidence packets, deductibles/reserves) and business continuity (impact assessment for fire, flood, weather, cyber, power, staff shortage; crisis roles/contacts; guest welfare/relocation; manual workflows and supplier backup; drills; recovery targets vs actual). |
| Phases | 4 incident linkage, policy/claim register, BIA, crisis roles, welfare; 6 operational drills and recovery evidence. |
| Release flag | R1. |
| Bounded context | `risk-continuity` |
| SoR entities (owned) | `insurance_policy`, `insured_item`, `premium_schedule`, `insurance_claim`, `claim_evidence_packet`, `claim_party_access`, `bia_scenario`, `crisis_role`, `crisis_contact_plan`, `continuity_plan`, `guest_relocation_record`, `drill_record`, `recovery_measurement`. |
| Referenced (not owned) | M42 incidents, chronology and evidence; M26 assets; M20 AP (premiums), AR/receipts (claim settlements); M19 GL (prepaid insurance, reserves); M64 recovery objectives/backups; M61 findings; M55 guest cases; M05 in-house guests; M46 backup suppliers; M27 staff contacts/roster; M02 external access scopes. |
| Dependencies | M02, M05, M19, M20, M26, M27, M42, M46, M55, M61, M63, M64. |

### F68.1 Coverage

```yaml
- id: M68.F68.1.SF68.1.1
  name: policy, insured asset/risk and coverage/expiry
  phase: 4
  release: R1
  actors: [financial_controller, gm, owner]
  screens: [SCR-FIN-insurance-register]
  inputs: [insurer, broker, policy_number, coverage_type, limits, deductibles, period_start, period_end, insured_items, document_ref]
  states: [draft, active, expiring, expired, renewed, cancelled]
  api: ["POST /v1/properties/{pid}/insurance-policies", "POST /v1/properties/{pid}/insurance-policies/{pol}/insured-items"]
  events: [InsurancePolicyActivated, InsurancePolicyExpiring]
  data: [insurance_policy, insured_item, asset (M26)]
  rules: ["Coverage types: property, business interruption, liability, cyber, vehicle, employer; insured items link to M26 assets or risk descriptions.", "Expiry alerts at 60/30/7 days; lapse of a required policy (e.g. vehicle) triggers gates in M59/M58.", "Policy document stored with restricted access."]
  security: "Finance/owner scope."
  failure_cases: [coverage_gap_detected]
  finance_report_effect: "Prepaid insurance and expense amortization via M19."
  i18n_a11y: "Bilingual."
  acceptance: "AC-SF68.1.1: A vehicle policy expiring removes linked vehicles from M59 assignment at expiry."
  dependency: "M26, M59, D-659."

- id: M68.F68.1.SF68.1.2
  name: premium payable/accrual
  phase: 4
  release: R1
  actors: [ap_clerk, financial_controller]
  screens: [SCR-FIN-insurance-premiums]
  inputs: [policy_id, premium_amount, installments, tax, due_dates]
  states: [scheduled, invoiced, approved, paid]
  api: ["POST /v1/properties/{pid}/insurance-policies/{pol}/premium-schedule"]
  events: [PremiumInvoiced, PremiumPaid]
  data: [premium_schedule, supplier_invoice (M20)]
  rules: ["Premium invoices go through M20 AP backbone (Section D); prepaid amortized monthly via M19.", "Allocation to departments by approved driver."]
  security: "AP segregation of duties."
  failure_cases: [duplicate_premium_invoice]
  finance_report_effect: "Insurance expense below GOP or per profit bridge policy."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF68.1.2: An annual premium paid upfront amortizes 1/12 per month in M19."
  dependency: "M20, M19."

- id: M68.F68.1.SF68.1.3
  name: incident evidence packet and claim status
  phase: 4
  release: R1
  actors: [financial_controller, gm, security_officer]
  screens: [SCR-FIN-insurance-claims, SCR-OPS-incident]
  inputs: [incident_ids (M42), policy_id, loss_description, loss_estimate, evidence_selection]
  states: [draft, notified, submitted, under_assessment, approved, partially_approved, denied, settled, closed]
  api: ["POST /v1/properties/{pid}/insurance-claims", "POST /v1/properties/{pid}/insurance-claims/{cid}/evidence-packets"]
  events: [InsuranceClaimSubmitted, InsuranceClaimStatusChanged]
  data: [insurance_claim, claim_evidence_packet]
  rules: ["Evidence packet assembles hashed copies/pointers of M42 chronology, photos, M61 inspection history, M26 asset records, repair quotes and invoices; original evidence remains in source modules.", "Notification deadlines per policy tracked via M63."]
  security: "Guest PII redacted unless required and approved by DPO."
  failure_cases: [notification_deadline_missed_alert, evidence_hash_mismatch]
  finance_report_effect: "Claim receivable recognized only when insurer confirms (not on submission)."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF68.1.3: A fire incident produces a packet whose hashes match the M42 evidence; receivable is not posted until approval (AT-G14.4)."
  dependency: "M42, M61, M26."

- id: M68.F68.1.SF68.1.4
  name: adjuster/vendor access scope
  phase: 4
  release: R1
  actors: [financial_controller, vendor_user, it_admin]
  screens: [SCR-ADM-external-access, SCR-VND-claim-workspace]
  inputs: [claim_id, external_party, access_scope, expiry]
  states: [granted, active, expired, revoked]
  api: ["POST /v1/properties/{pid}/insurance-claims/{cid}/access-grants"]
  events: [ClaimAccessGranted, ClaimAccessRevoked]
  data: [claim_party_access]
  rules: ["Adjusters/loss assessors get time-limited read access to the specific packet only, via M02 external identity with MFA.", "Downloads watermarked and logged."]
  security: "Least privilege; auto-expiry."
  failure_cases: [access_after_expiry_denied]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF68.1.4: An adjuster cannot open another claim or any other record; access ends at expiry."
  dependency: "M02, M46."

- id: M68.F68.1.SF68.1.5
  name: deductible/reserve/settlement report
  phase: 5
  release: R1
  actors: [financial_controller, owner]
  screens: [SCR-OWN-insurance-summary]
  inputs: [period, claims, reserves, settlements]
  states: [computed]
  api: ["GET /v1/properties/{pid}/insurance/summary?period"]
  events: [InsuranceSummaryComputed]
  data: [insurance_claim]
  rules: ["Shows claimed, reserved (internal estimate, labelled), approved, received, deductible borne, and uninsured loss per incident.", "Settlements matched to bank receipts (M20)."]
  security: "Owner/finance."
  failure_cases: [settlement_unmatched]
  finance_report_effect: "Loss and recovery in M19; owner report input."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF68.1.5: A settled claim shows received amount matched to bank receipt and deductible borne."
  dependency: "M20, M19."
```

### F68.2 Continuity

```yaml
- id: M68.F68.2.SF68.2.1
  name: scenario impact (fire, flood, cyber, power, staff shortage)
  phase: 4
  release: R1
  actors: [gm, chief_engineer, it_admin, hr_officer]
  screens: [SCR-OWN-bia]
  inputs: [scenario_type, affected_services, dependencies, max_tolerable_downtime, financial_impact_estimate, guest_impact]
  states: [draft, reviewed, approved, due_for_review]
  api: ["POST /v1/properties/{pid}/bia-scenarios"]
  events: [BiaScenarioApproved]
  data: [bia_scenario]
  rules: ["Scenarios include fire, flood, severe weather, cyber, power/utility outage, key supplier failure, staff shortage (including chef coverage via M47).", "Each lists dependent modules/devices (from M64 registry) and manual workaround.", "Annual review."]
  security: "Management."
  failure_cases: [dependency_unknown]
  finance_report_effect: "Estimates labelled; feeds insurance adequacy review."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF68.2.1: A power outage scenario lists PMS, POS, locks and gate dependencies from M64 and their manual procedures."
  dependency: "M64, M47."

- id: M68.F68.2.SF68.2.2
  name: crisis role tree and contacts
  phase: 4
  release: R1
  actors: [gm, duty_manager, hr_officer]
  screens: [SCR-OWN-crisis-roles]
  inputs: [role_name, primary_person, deputies, contact_channels, external_contacts]
  states: [assigned, gap, verified]
  api: ["PUT /v1/properties/{pid}/crisis-roles/{rid}"]
  events: [CrisisRoleGap]
  data: [crisis_role, crisis_contact_plan]
  rules: ["Every crisis role has primary and at least one deputy; gaps (leaver, leave) flagged from M27.", "External contacts (fire, police, utility emergency lines, insurer hotline) per M44 local numbers, verified periodically."]
  security: "Contacts available offline to crisis role holders."
  failure_cases: [contact_unverified]
  finance_report_effect: "None."
  i18n_a11y: "Offline printable bilingual card."
  acceptance: "AC-SF68.2.2: Deactivating the primary incident commander in M27 flags a crisis-role gap to the gm."
  dependency: "M27, M42, M44."

- id: M68.F68.2.SF68.2.3
  name: guest welfare/relocation
  phase: 4
  release: R1
  actors: [duty_manager, front_office_manager, guest_relations]
  screens: [SCR-OPS-guest-welfare, SCR-OPS-relocation]
  inputs: [incident_id, in_house_list (M05), accessibility_needs, relocation_partner, transport_needs]
  states: [assessed, relocating, relocated, returned, closed]
  api: ["GET /v1/properties/{pid}/incidents/{iid}/guest-welfare", "POST /v1/properties/{pid}/guest-relocations"]
  events: [GuestRelocated]
  data: [guest_relocation_record, reservation (M05)]
  rules: ["Welfare list from M05 in-house guests with stated accessibility needs prioritized; list available offline.", "Relocation to partner hotels recorded with transport (M59) and cost; guest folios adjusted via M08."]
  security: "Guest data minimal; accessibility info visible to crisis roles only during incident."
  failure_cases: [partner_hotel_full]
  finance_report_effect: "Relocation costs to incident cost and potential claim."
  i18n_a11y: "Accessible; bilingual guest notices."
  acceptance: "AC-SF68.2.3: During a simulated evacuation the welfare list shows guests with stated mobility needs first and works offline (AT-G14.2)."
  dependency: "M05, M59, M08, M42, D-660."

- id: M68.F68.2.SF68.2.4
  name: safe manual workflow, supplier backup and drill
  phase: 6
  release: R1
  actors: [gm, duty_manager, procurement_officer, it_admin]
  screens: [SCR-OWN-continuity-plans, SCR-OWN-drills]
  inputs: [plan_id, manual_procedures (M64), backup_suppliers (M46), drill_type, participants, observations]
  states: [planned, executed, findings_open, closed]
  api: ["POST /v1/properties/{pid}/continuity-plans", "POST /v1/properties/{pid}/drills"]
  events: [DrillExecuted, DrillFindingOpened]
  data: [continuity_plan, drill_record]
  rules: ["Each critical supply (gas, food, linen, water) has a backup supplier from M46 or documented alternative.", "Tabletop and live drills per schedule; findings become M63 work items.", "Drills never trigger real emergency services unless planned with them."]
  security: "Management."
  failure_cases: [backup_supplier_expired_credentials]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF68.2.4: A tabletop drill records timeline, findings and follow-up tasks; a backup supplier with expired permit is flagged."
  dependency: "M46, M64, M63, D-661."

- id: M68.F68.2.SF68.2.5
  name: recovery target, actual duration and corrective action
  phase: 6
  release: R1
  actors: [gm, owner, it_admin]
  screens: [SCR-OWN-recovery-evidence]
  inputs: [scenario_id, target_duration, actual_start, actual_end, source (drill or incident)]
  states: [measured, target_met, target_missed, corrective_open, corrective_closed]
  api: ["POST /v1/properties/{pid}/recovery-measurements"]
  events: [RecoveryMeasured, RecoveryTargetMissed]
  data: [recovery_measurement, recovery_objective (M64)]
  rules: ["Only drills or real incidents produce measurements; missed targets require corrective action with owner.", "Recovery evidence reported to owner."]
  security: "Management."
  failure_cases: [no_measurement_in_period]
  finance_report_effect: "None."
  i18n_a11y: "Accessible."
  acceptance: "AC-SF68.2.5: A drill exceeding the target opens a corrective action and shows target_missed on the owner screen."
  dependency: "M64 SF64.2.4."

- id: M68.F68.2.SF68.2.6  # ADDED — Section C 'emergency contact plans'; Section O GM 'did a fire/gas/water alarm reach an accountable person'
  name: Emergency contact plan publication and reachability test
  phase: 4
  release: R1
  actors: [gm, duty_manager, security_officer]
  screens: [SCR-OWN-contact-plan-tests]
  inputs: [contact_plan_id, test_message, recipients, ack_deadline]
  states: [scheduled, sent, acknowledged_all, partial, failed]
  api: ["POST /v1/properties/{pid}/crisis-contact-plans/{cid}/tests"]
  events: [ContactPlanTested]
  data: [crisis_contact_plan]
  rules: ["Periodic test pages crisis roles via M63 channels and records acknowledgment times.", "Unreachable contacts create correction tasks."]
  security: "Test messages clearly labelled TEST."
  failure_cases: [channel_outage]
  finance_report_effect: "None."
  i18n_a11y: "Bilingual."
  acceptance: "AC-SF68.2.6: A monthly test records acknowledgment times and flags one unreachable deputy."
  dependency: "M63 SF63.1.6, M42."
```

### M68 key invariants

1. Evidence packets reference hashed originals in source modules; no evidence copies edited.
2. Claim receivables only on insurer confirmation.
3. Recovery performance is measured from drills or incidents, never asserted.
4. Every crisis role has a deputy; gaps are visible.

### M68 module acceptance

| AC | Section G | Section O question answered |
|---|---|---|
| AC-SF68.1.3, AC-SF68.1.4 | AT-G14 (incident chronology/evidence) | Maintenance/security: "What evidence preserves an incident or insurance claim?" |
| AC-SF68.2.3, AC-SF68.2.6 | AT-G14 (escalation, outage fallback) | Maintenance/security: "Did a fire/gas/water alarm reach an accountable person?" |
| AC-SF68.2.4, AC-SF68.2.5 | Section B Phase 6 exit (cyber/physical continuity) | GM: "Can we operate if internet or a provider fails?" |
| AC-SF68.1.1, AC-SF68.1.5 | AT-G08 | Owner: "What cash, debt, insurance and tax obligations are due?" |

### M68 open decisions

| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-659 | Insurance broker/insurer data exchange (manual vs portal/API). | Financial Controller | Manual entry and document upload; no insurer API. |
| D-660 | Relocation partner hotels and transport arrangements. | GM | Two nearby partner hotels by written agreement; manual booking. |
| D-661 | Drill types and frequency (tabletop, evacuation, cyber, power). | GM + Security Officer | Quarterly tabletop, annual evacuation with local authority coordination, semi-annual cyber tabletop. |

---

## 20. Totals, decision register and canonical entities

### 20.1 Counts

| Measure | Count |
|---|---|
| Modules | 18 (M51–M68) |
| Features | 37 (36 fixed by Section Q + 1 added: F55.3) |
| Subfeatures fixed by Section Q (IDs verbatim) | 192 |
| Subfeatures added (marked `# ADDED`) | 27 |
| Total Section-L subfeature blocks | 219 |
| Subfeatures `release: R1` | 214 |
| Subfeatures `release: Later` (Phase 7) | 5 (SF53.2.7, SF55.3.1, SF55.3.2, SF64.1.7, SF66.2.6) |
| Unit acceptance tests (`AC-<SF id>`) | 219 |
| Open decisions | 61 (D-601–D-661) |

First-phase distribution and the per-module add list are reproducible by counting `phase:` and `# ADDED` in this file.

### 20.2 Open decision register (summary)

| Range | Module | Decisions |
|---|---|---|
| D-601–D-604 | M51 | domain/hosting, search/metasearch partners, analytics tool, attribution precedence |
| D-605–D-608 | M52 | guest profile SoR boundary, per-market consent, review sources, messaging providers |
| D-609–D-612 | M53 | market data licence, guardrails/tiers, forecast method, overbooking policy |
| D-613–D-616 | M54 | voucher liability/expiry, package allocation, comp upgrades, promotional permits |
| D-617–D-620 | M55 | guest_request SoR, compensation caps, telephony/recording, key/kiosk vendors |
| D-621–D-624 | M56 | M06/M56 boundary, laundry model, minibar model, linen par/RFID |
| D-625–D-627 | M57 | food-safety rule packs, alcohol licensing/age, IRD confirmation |
| D-628–D-630 | M58 | enabled amenities, spa intake legality, practitioner commission/tips |
| D-631–D-633 | M59 | owned fleet, flight/ship status source, M45/M59 boundary |
| D-634–D-636 | M60 | cash variance tolerance, employee-monitoring scope, safe/deposit process |
| D-637–D-639 | M61 | inspection rules per market, release authority, water testing |
| D-640–D-642 | M62 | LMS approach, certifications per role/market, quality sampling notice |
| D-643–D-645 | M63 | workflow runtime, template edit rights, staff alert channels |
| D-646–D-649 | M64 | site network, RPO/RTO, on-prem/offsite location, support model |
| D-650–D-652 | M65 | analytics store, KPI dictionary ownership, retention schedule |
| D-653–D-655 | M66 | ownership structure, fixed-asset register ownership, capex thresholds |
| D-656–D-658 | M67 | emission factors, baseline, food-waste measurement |
| D-659–D-661 | M68 | insurer data exchange, relocation partners, drill frequency |

Cross-file alignment decisions that other catalogue authors must confirm: **D-605** (M18/M52 guest profile), **D-617** (M18/M55 `guest_request`), **D-621** (M06/M56), **D-633** (M45/M59), **D-651** (M32/M65 KPI dictionary), **D-654** (M19/M26/M66 fixed assets).

### 20.3 Canonical entity names defined in this file (owned SoR)

- **M51** `website_site`, `website_domain`, `website_takedown`, `site_page`, `site_page_version`, `redirect_rule`, `structured_data_snapshot`, `destination_content_item`, `web_search_session`, `quote_funnel_event`, `abandonment_reason`, `acquisition_source`, `campaign_tag`, `attribution_touch`, `attribution_assignment`, `bot_filter_rule`, `acquisition_cost_line`
- **M52** `guest_merge_candidate`, `guest_merge_decision`, `guest_identity_link`, `send_eligibility_decision`, `segment_definition`, `segment_snapshot`, `frequency_cap_policy`, `message_template`, `campaign`, `campaign_send`, `message_delivery`, `guest_value_snapshot`, `survey_definition`, `survey_response`, `external_review`, `review_response`, `review_policy`, `subject_tag`
- **M53** `demand_snapshot`, `pace_curve`, `forecast_run`, `forecast_value`, `market_input`, `demand_event_calendar`, `overbooking_recommendation`, `rate_recommendation`, `rate_action`, `guardrail_policy`, `rate_publication_ack`, `price_discrepancy_item`, `backtest_result`
- **M54** `offer_definition`, `offer_eligibility_rule`, `offer_presentation`, `ancillary_order`, `offer_performance_snapshot`, `package_component`, `component_consumption`, `gift_voucher`, `voucher_ledger_entry`
- **M55** `guest_conversation`, `conversation_message`, `inquiry_lead`, `journey_task`, `guest_request`, `guest_case`, `case_action`, `compensation_grant`, `guest_confirmation`, `key_credential_request` (Later), `kiosk_session` (Later)
- **M56** `hk_task`, `hk_assignment`, `hk_inspection`, `hk_duration_standard`, `deep_clean_schedule`, `linen_par_policy`, `laundry_batch`, `laundry_batch_line`, `linen_damage_record`, `minibar_count`, `amenity_par_policy`
- **M57** `dining_reservation`, `waitlist_entry`, `ird_delivery`, `kitchen_ticket_timing`, `allergen_acknowledgment`, `production_batch`, `production_batch_lot_link`, `food_check_record`, `substitution_review`, `portion_waste_record`, `outlet_service_rule`
- **M58** `amenity_business_profile`, `amenity_service`, `amenity_booking`, `amenity_entitlement_use`, `practitioner_assignment`, `contraindication_intake`, `amenity_sanitation_record`, `amenity_closure`
- **M59** `transport_trip`, `trip_leg`, `trip_manifest`, `trip_status_event`, `trip_handover`, `transport_tariff`, `fleet_vehicle_profile`, `driver_eligibility`, `trip_cost_line`
- **M60** `control_limit_policy`, `till_session_control`, `cash_drop`, `safe_movement`, `audit_difference_item`, `control_approval`, `anomaly_rule`, `anomaly_alert`, `investigation_case`, `investigation_evidence`, `alert_feedback`
- **M61** `inspection_template`, `inspection_schedule`, `inspection_record`, `inspection_finding`, `corrective_action`, `permit_record`, `responsible_person_assignment`, `service_stoppage`, `release_decision`
- **M62** `sop_document`, `sop_version`, `role_training_matrix`, `training_course`, `training_assignment`, `training_attestation`, `assessment_result`, `certification_record`, `contractor_briefing`, `shift_handover`, `labor_demand_forecast`, `quality_sample`, `coaching_note`
- **M63** `workflow_template`, `workflow_version`, `workflow_trigger`, `workflow_instance`, `work_item`, `sla_timer`, `escalation_step`, `approval_request`, `approval_decision`, `notification_rule`, `notification_delivery`, `on_call_schedule`, `workflow_change_audit`
- **M64** `device`, `device_firmware_record`, `certificate_record`, `connector_profile`, `network_zone`, `device_health_sample`, `degraded_queue_item`, `maintenance_window`, `manual_fallback_procedure`, `fallback_test_record`, `backup_job`, `restore_test`, `recovery_objective`, `offline_action_log`, `cyber_containment_record`, `change_request`, `support_ticket`, `status_page_entry`, `dr_failover_test`
- **M65** `canonical_entity_registry`, `master_data_issue`, `metric_definition`, `metric_version`, `lineage_edge`, `dataset_freshness_status`, `date_mapping_rule`, `report_definition`, `report_version`, `report_snapshot`, `scheduled_report`, `export_job`, `purpose_retention_map`, `attribution_model_version`
- **M66** `ownership_relationship`, `management_agreement`, `fee_schedule`, `fee_calculation`, `owner_statement`, `external_obligation_input`, `capex_request`, `investment_case`, `renovation_project`, `capitalization_decision`, `brand_standard`, `brand_audit_evidence`, `reopening_checklist`
- **M67** `resource_metric_series`, `metric_data_status`, `meter_quality_score`, `intensity_snapshot`, `emission_factor_set`, `sustainability_target`, `sustainability_baseline`, `savings_measurement`, `sustainability_export`, `claim_review`, `supplier_sustainability_evidence`
- **M68** `insurance_policy`, `insured_item`, `premium_schedule`, `insurance_claim`, `claim_evidence_packet`, `claim_party_access`, `bia_scenario`, `crisis_role`, `crisis_contact_plan`, `continuity_plan`, `guest_relocation_record`, `drill_record`, `recovery_measurement`

Entities written as `name (Mnn)` inside `data:` fields are **references** to the named owning module and are defined in that module's catalogue file, not here.
