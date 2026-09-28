# 10 — Competitive gap audit and owner-question traceability

**Pack:** MetriStay Hospitality Suite — Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Governing source:** master prompt v3.0 Sections N (benchmark), O (question-first playbook), P (simplicity/data/release gates), with feature/subfeature identifiers from Sections C, K and Q. Conventions: `docs/README.md` §3.
**Status:** specification only. Competitor statements below are **official vendor capability descriptions (vendor claims)**, not independently measured performance, and not evidence that any competitor lacks a capability that its visited page did not mention. Nothing here claims MetriStay parity with any product: parity may be asserted only after the named acceptance test passes against a working build.

## 0. How to read this file

| Section | Purpose |
|---|---|
| §1 | Source register: every competitor URL, visit date (2026-09-28), retrieval method and result |
| §2 | What each vendor claims (summary of retrieved text) |
| §3 | Capability matrix by hotel outcome (Section N rows), with MetriStay design requirement and proof |
| §4 | MetriStay scope that the benchmark pages do not cover (reason it exists, not a superiority claim) |
| §5 | Deliberate exclusions and deferrals, with reasons |
| §6 | Substitute partner strategy (where MetriStay integrates rather than builds) |
| §7 | Migration comparison (from each benchmark product) |
| §8 | Operating-cost comparison (qualitative, no prices) |
| §9 | **Owner-question traceability table** — every Section O question, one row each, `Q-<role>-n` |
| §10 | Cross-cut table: each Section O role × eight operating conditions |
| §11 | Module coverage check M01–M68 |
| §12 | Proof gaps, open decisions and owners |

**Identifier notes.**
- Feature and subfeature ids from Sections K and Q are used verbatim (`M53.F53.2.SF53.2.2`).
- Where Section K/Q defines a feature but no subfeature (F27.4 manager privacy; the M22–M25 cross-utility guard), the row cites the `Fnn.k` level and says so.
- Modules M01–M18 and M33–M37 have **no feature ids in the master prompt**. Rows cite them as `Mnn⟨feature phrase from Section C⟩`; the numeric `Fnn.k/SFnn.k.j` is assigned in `docs/01-module-catalogue.md` and reconciled in the `docs/13` consistency check. This is marked **(F-id per docs/01)** where it first matters.
- Acceptance: `AC-<SF id>` is the subfeature-level acceptance defined with each SF in `docs/01` (README §3.1). `Gnn` names the Section G integrated scenario in `docs/09` that exercises it. No `AT-Gnn.n` sub-number is invented here; `docs/09` owns that numbering.
- Phases: first implementation phase; `R1` = Phases 2–6; `Later` = 7–8.
- KPIs name entries in the `docs/06` KPI dictionary. Targets are agreed with the pilot hotel (Section P.6); none is asserted here.

**Screen-ID prefixes added by this file** (in addition to those declared in `docs/01-catalogue/02` §0.2 and `/05` §0.1): `SCR-MGR-` (owner/GM/duty-manager), `SCR-REV-` (revenue/sales analytics), `SCR-MKT-` (marketing, CRM, reputation, content), `SCR-HK-` (housekeeping/laundry web + staff mobile), `SCR-ENG-` (engineering/maintenance), `SCR-SEC-` (security/incident/lost-and-found), `SCR-HR-` (HR/payroll), `SCR-CMP-` (compliance/privacy), `SCR-WEB-` (public hotel website). These are proposals for `docs/04` to confirm.

---

## 1. Source register

Visit date for every row: **2026-09-28**. Retrieval was attempted first with the agent's WebFetch tool; where that was refused, a second plain HTTPS request (curl with a browser user agent) was attempted from the same session. Content is treated as **vendor claims** (honesty label `source-cited`, claim type `vendor-marketing`).

| Ref | Vendor | URL (from Section N unless noted) | Result | Method / notes |
|---|---|---|---|---|
| SRC-C01 | Oracle Hospitality (OPERA Cloud) | https://www.oracle.com/hospitality/hotel-property-management/ | **Retrieved (partial path)** | WebFetch: HTTP 403. curl: HTTP 200, but the URL **redirected to** `https://www.oracle.com/hospitality/` (general hospitality landing page). Headings/text extracted: property management, dining, upselling, sales and event management, distribution, central functions, loyalty and marketing, finance, human capital management; OPERA Cloud Digital Assistant; mobile check-in with merchandising; payments incl. contactless/mobile/kiosk; Oracle Hospitality Integration Platform (APIs for distribution channels). |
| SRC-C02 | Oracle Hospitality (OPERA Cloud Sales and Event Management) | https://www.oracle.com/hospitality/opera-sales-event-management/ | **Retrieved** | WebFetch: HTTP 403. curl: HTTP 200. Text extracted: Block Presentation, Event Presentation, Function Diary, Manage Resources (menus/items), group blocks (corporate/social/FIT) in one system, event templates, catering packages, item inventory and menu management, BEO detailing, multi-property availability on one screen, mobile access for site inspections, three subscription options by property type. |
| SRC-C03 | Mews | https://www.mews.com/en/products | **Retrieved** | WebFetch: HTTP 200. Product list: PMS, reservation and channel management, booking engine, guest intelligence, online check-in, self check-in kiosks, digital keys, guest self check-out, RMS/dynamic pricing/demand forecasting and controls, upsells, housekeeping, POS/ePOS, embedded payments, tokenization, multicurrency, payment terminals, automated reconciliation, accounting and billing, accounts receivable, financing partnership, BI/data and reporting, Mews Marketplace ("1,000+ integrations"), open API, Mews AI. |
| SRC-C04 | Cloudbeds | https://www.cloudbeds.com/hospitality-platform/ | **Retrieved** | WebFetch: HTTP 200. Product list: PMS, channel manager, booking engine, payments, guest experience (communication and digital check-in), RMS, guest marketing CRM, digital marketing, websites, reputation management, insights and reporting, "Signals" (AI foundation model); claims unified data model, multi-property, "450+" integration partners, PCI-DSS Level 1, SSO. |
| SRC-C05 | SiteMinder | https://www.siteminder.com/ | **Not retrieved directly** | WebFetch: HTTP 403. curl: HTTP 403 ("Sorry, you have been blocked" — bot-protection page). |
| SRC-C05a | SiteMinder (indirect) | Search-engine result snippets of official siteminder.com pages: `/channel-manager/`, `/hotel-booking-engine/`, `/integrations/`, `/hotel-software/`, `/find-the-right-guests/`, `/groupsandchains/`, `/pricing/` | **Indirect only** | WebSearch restricted to `siteminder.com` returned titles/snippets: "world's largest open hotel commerce platform"; channel manager connecting to "over 450 distribution channels, including the GDS"; booking engine integrated with own website or SiteMinder website builder; metasearch (Google Hotel Ads, Trivago, Tripadvisor) with a managed bidding team; GDS connectivity; website builder; payments. **Snippets are secondary evidence**; re-verify from a normal browser before any external use (action on D-701). |

**Not used as evidence:** analyst rankings, review sites, customer case studies' performance numbers, or any price. Customer-success headlines on SRC-C01 were seen but are not relied on.

---

## 2. Vendor capability summary (vendor claims)

| Vendor | Positioning claimed on visited pages | Capability families claimed (abbrev.) |
|---|---|---|
| Oracle OPERA Cloud | Enterprise hospitality suite connecting event sales, rooms, management and POS; integration platform for partners | PMS; mobile check-in with embedded merchandising; upselling; sales & event management (blocks, function diary, BEO, menus, resources); distribution via integration-platform APIs; central functions; loyalty and marketing; finance; HCM; dining/F&B POS; payments incl. contactless/kiosk; digital assistant |
| Mews | Cloud PMS with embedded payments and a large integration marketplace | PMS; channel/reservation management; booking engine; online check-in, kiosk, digital keys, self check-out; RMS/dynamic pricing/forecasting; upsells; housekeeping; POS; embedded payments/tokenization/terminals/automated reconciliation; accounting/billing/AR; BI; marketplace + open API; AI |
| Cloudbeds | Unified-data hospitality platform for independents and groups | PMS; channel manager; booking engine; websites; payments; guest communication/digital check-in; RMS; guest marketing CRM; digital marketing; reputation; insights; AI ("Signals"); multi-property; integrations |
| SiteMinder (indirect) | Hotel commerce/distribution platform | Channel manager (OTA + GDS); booking engine; website builder; metasearch (managed); GDS; payments; integrations; groups & chains |

**Observation (not a gap claim):** none of the retrieved pages describes utility bill-pay, gas-cylinder custody, vendor mobile catalogs with daily stock, chef emergency callout, sample-photo retention in RFQs, low-touch receiving with lot trace, five-market jurisdiction rule packs or hotel-initiated flight/cruise booking. These areas may exist in other products or other pages; MetriStay treats them as *its own required scope* (§4), not as a differentiator to advertise.

---

## 3. Capability matrix by hotel outcome

Legend for vendor columns: **C** = claimed on a visited page; **C(p)** = claimed via partner/integration; **—** = not stated on the visited page (absence of evidence only); **I** = indirect (search snippet). MetriStay column is a design requirement; "Proof" names the acceptance that must pass before any comparison is made.

### 3.1 Get discovered and sell direct

| Capability | OPERA | Mews | Cloudbeds | SiteMinder | MetriStay design requirement | Proof (must pass) | Modules / phase |
|---|---|---|---|---|---|---|---|
| Hotel website | — | — | C | I | Fast, multilingual (EN/AR RTL), WCAG 2.2 AA website with owner-versioned content and approved media only | AC-SF51.1.1, AC-SF51.1.4, AC-SF39.1.5; G13, G19 | M51, M39 / 2 |
| Booking engine with live inventory | C(p) | C | C | I | Search only true sellable inventory (M03 room-type night stock minus holds, OOO) and timed resources (M09 INV-CAP-1) | AC-SF51.2.1; G19, G20 concurrency | M51, M03, M04 / 2 |
| Search/maps/metasearch | — | — | C (digital marketing) | I (managed bidding) | Approved search/maps/metasearch referral **via partner**; campaign tags; no scraping | AC-SF51.2.3; partner sandbox | M51, M07 / 3–5 |
| OTA/GDS channel distribution | C | C | C | I | Certified channel-manager adapter (M07), ARI push with acknowledgment, dead letters, oversell exposure | AC-M07 ARI ack (F-id per docs/01); AC-SF53.2.4; G19, G20 | M07 / 3 |
| Total-price quote | — | — | — | — | Taxes/fees/policy snapshot from M04 + M44 rule pack shown before payment; quote expiry | AC-SF51.2.2; G12, G19 | M04, M44 / 2 |
| Attribution and channel net cost | — | — | C (insights) | — | Quote → paid-stay attribution; net contribution after commission, PSP fee, marketing spend | AC-SF51.2.5, AC-SF53.1.3, AC-SF32.1.3; G19 | M51, M53, M32 / 2–5 |
| Accessible, low-bandwidth booking | — | — | — | — | Page-weight/latency budget (`docs/11` §7); assisted booking via front desk/phone | `docs/11` AT-G19 walkthrough | M51, M55 / 2 |

### 3.2 Price and forecast profitably

| Capability | OPERA | Mews | Cloudbeds | SiteMinder | MetriStay design requirement | Proof | Modules / phase |
|---|---|---|---|---|---|---|---|
| Pickup/pace | — | C | C | — | Pace by stay date × booking date; group wash; OOO supply | AC-SF53.1.1, AC-SF53.1.2 | M53 / 3 |
| Demand forecast | — | C | C | — | Forecast from hotel evidence with confidence band and data-coverage flag; prior-year comparison | AC-SF53.1.5; G19 (90-day check) | M53 / 5 |
| Restrictions (LOS, CTA) | — | C | C | I | Candidate restriction with rationale; guardrails by role | AC-SF53.2.1, AC-SF53.2.2 | M53, M04 / 5 |
| Recommended/dynamic rate | — | C | C | — | Recommendation with explainability, simulation vs contracts, human approval, publish ack, rollback; **no guaranteed uplift** | AC-SF53.2.3–SF53.2.6; G19 | M53 / 5 (advanced automation 7) |
| Upsells | C | C | — | — | Attribute-based upgrade/early/late bounded by cleaning & sale; one-time folio post | AC-SF54.1.1, AC-SF54.1.2, AC-SF54.1.5; G19 | M54 / 3 |
| Net contribution (not only ADR) | — | — | — | — | Segment/channel contribution after fees and cost-to-serve | AC-SF53.1.3, AC-SF32.1.3 | M53, M32 / 3–5 |

### 3.3 Run a smooth stay

| Capability | OPERA | Mews | Cloudbeds | SiteMinder | MetriStay design requirement | Proof | Modules / phase |
|---|---|---|---|---|---|---|---|
| Online/mobile check-in | C | C | C | — | Pre-arrival task done once across web/app/desk; ID OCR with field confirmation; non-biometric path | AC-SF41.1.5, AC-SF41.1.6, AC-SF55.1.2; G13 | M41, M55, M05 / 2–3 |
| Kiosk / digital key | C (kiosk) | C | — | — | **Deferred** to Phase 7 authorized adapters (§5) | n/a in R1 | M55 / 7 |
| Front desk and room status | C | C | C | — | Room state vs cleaning state; room-ready ETA; offline conflict | AC-SF56.1.5; G19 | M06, M56 / 2 |
| Housekeeping | C | C | — | — | Priority from arrivals/VIP/DND; inspection/reclean; linen custody; minibar one-time post | AC-SF56.1.1–SF56.2.6; G19 | M56 / 2–4 |
| Payments/tokenization/terminals | C | C | C | I | Payment orchestration via certified PSP; tokens only; idempotent webhooks; MetriStay is **not a PSP** | AC-SF28.1.5, AC-SF28.1.6; G06, G20 | M28 / 2, 5 |
| Guest messaging/requests | C | — | C | — | Unified consented inbox; request routed to owning team with SLA and outage fallback | AC-SF55.1.1, AC-SF55.1.4, AC-SF55.1.6 | M55, M18 / 2–3 |
| AI assistant | C (staff digital assistant) | C (AI) | C (Signals) | — | Guest assistant bounded to approved KB and read-only quote tools; draft booking only; human handoff | AC-SF40.1.5, AC-SF40.2.2, AC-SF40.2.3; G13 | M40 / 3–5 |
| Manual recovery on device failure | — | — | — | — | Every device/integration has a manual route and accountable owner (`docs/12` §5) | AC-SF64.1.5, AC-SF55.1.6; G20 | M64, M55 / 2–6 |

### 3.4 Serve corporate and events

| Capability | OPERA | Mews | Cloudbeds | SiteMinder | MetriStay design requirement | Proof | Modules / phase |
|---|---|---|---|---|---|---|---|
| Negotiated/corporate rates | C | — | — | I (GDS corporate) | Corporate agreement versions and rate eligibility (M10) | G01, G02 | M10, M04 / 3 |
| Room blocks and pickup | C | — | — | I (groups & chains) | Block/pickup/cutoff; pickup vs contract vs forecast | AC-SF32.1.4; G01 | M12, M53 / 3 |
| Function diary/space | C | — | — | — | Timed-resource non-double-sale (INV-CAP-1), composite hold | G01, G02, G20 | M09, M12 / 2–3 |
| BEO, menus, catering resources | C | — | — | — | Versioned BEO propagating to kitchen/bar; allergen flags; production plan; chef coverage | AC-SF47.1.1, AC-SF57.1.4; G02, G15 | M12, M16, M47, M57 / 3 |
| Corporate self-service portal/apps | — | — | — | — | Web + signed Android/iOS with SSO, compare, hold/approve, invoices | G01 | M11 / 3 |
| One itinerary and bill (rooms, F&B, parking, transport) | C (partly: rooms+events) | — | — | — | One versioned itinerary and master bill incl. parking (M17) and transport (M59/M45) | G01–G03, G11 | M12, M08, M17, M59 / 3 |

### 3.5 Keep costs and safety visible

| Capability | OPERA | Mews | Cloudbeds | SiteMinder | MetriStay design requirement | Proof | Modules / phase |
|---|---|---|---|---|---|---|---|
| Finance/accounting | C (finance) | C (accounting & billing, AR) | — | — | Full GL/AP/AR/treasury, append-only journals | AC-SF19.2.1, AC-SF20.1.4; G08 | M19, M20 / 4 |
| Workforce/HCM | C | — | — | — | Roster, payroll, WPS/Canadian payroll; salary confidentiality | AC-SF27.3.5, F27.4 (no SF); G05, G12 | M27, M38 / 4 |
| Inventory/purchasing | C (item inventory in S&E) | — | — | — | Canonical stock ledger (M14/M50), RFQ-to-award, low-touch receiving | G16–G18 | M14, M21, M48–M50 / 3–4 |
| Utilities and gas | — | — | — | — | Meter/bill reconciliation, cylinder custody, authorized bill-pay | G04, G06 | M22–M25, M29 / 4–5 |
| Food safety and trace | — | — | — | — | Lot-linked production, recall hold, rule-pack-driven checks | AC-SF57.2.4, AC-SF50.2.10; G18 | M57, M50, M61 / 4 |
| Owner profit with estimate vs actual | C (finance planning) | C (BI) | C (insights) | — | Property result with source coverage and estimate/reconciled labels | AC-SF32.3.4, AC-SF65.1.5; G08, G19 | M32, M65 / 2–6 |

### 3.6 Bring guests back

| Capability | OPERA | Mews | Cloudbeds | SiteMinder | MetriStay design requirement | Proof | Modules / phase |
|---|---|---|---|---|---|---|---|
| CRM and segmentation | C (loyalty & marketing) | C (guest intelligence) | C (guest marketing CRM) | — | Permissioned merged profile; purpose-bound preferences; consent/suppression by country/channel | AC-SF52.1.1, AC-SF52.1.3 | M52, M02 / 3–5 |
| Loyalty | C | — | — | — | Non-cash points ledger, liability/breakage; no cash wallet without licence | AC-SF30.1.5, AC-SF30.2.6; G07 | M30 / 5 |
| Reputation | — | — | C | — | Approved-channel review ingestion; response approval; no incentivized deceptive reviews | AC-SF52.2.2, AC-SF52.2.6 | M52 / 3 |
| Service recovery | — | — | — | — | Case owner/SLA, compensation approval matrix, guest confirmation/reopen, recovery cost | AC-SF55.2.1–SF55.2.5; G19 | M55 / 4 |
| Referrals | — | — | — | — | Single-tier direct referral, per-booking contribution-margin commission; Oman legal gate | AC-SF31.1.7; G07 | M31 / 5–6 |

---

## 4. Scope MetriStay carries beyond the visited benchmark pages

Listed because the master prompt requires it; each item must still prove itself by acceptance test.

| Area | Why it is in scope | Modules | Proof |
|---|---|---|---|
| Utilities (electricity, water, pipeline gas) and gas cylinders | Section A/D: every hotel cost in one AP/GL backbone | M22–M25 | G04, G06, G20 |
| Provider-neutral bill-pay (Khedmah/ONEIC candidates) | Section E; **blocked until contract** | M29 | AC-SF29.1.7; G06 |
| Vendor registry, mobile catalog with daily stock, RFQ samples/weighted award | Sections K M46–M49 | M46, M48, M49 | G10, G16, G17 |
| Low-touch receiving, lot trace, waste/quarantine | Section K M50 | M50, M14 | G18 |
| Chef coverage and emergency callout | Section K M47 | M47 | G15 |
| Five-market jurisdiction classifier | Sections A, K M44 | M44, M38 | G09, G12 |
| Hotel-initiated flight/cruise/taxi | Section K M45; market-gated | M45, M59 | G11 |
| Parking ANPR, club, catering in one bill | Section C | M17, M15, M16 | G01–G03 |
| Incident command and lost-and-found | Section K M42/M43 | M42, M43 | G14 |
| Business continuity with measured recovery | Section Q M64/M68 | M64, M68 | `docs/12` §7 drills |

---

## 5. Deliberate exclusions and deferrals

| Competitor capability (vendor claim) | Decision | Reason | Revisit trigger / owner |
|---|---|---|---|
| Self check-in kiosks, digital keys (Mews, OPERA kiosk) | **Deferred — Phase 7** (`M55` key/kiosk; `M34–M36` connected room) | Requires certified lock/kiosk vendor adapters and hardware pilots; R1 keeps staffed + mobile pre-arrival path | Pilot hotel requests lock integration → D-702 (owner: Product Owner) |
| Casino/gaming, cruise-ship operations (Oracle sectors) | **Excluded** | Not hotel PMS scope; cruise appears only as a guest travel request (M45) | None planned |
| Centralized multi-property/chain ("central functions", Cloudbeds multi-property, SiteMinder groups & chains) | **Deferred — Phase 7** | Section B: single-hotel R1; data model is tenant/property-scoped so it can extend | Phase 7 gate |
| Managed metasearch bidding service (SiteMinder, indirect) | **Excluded as a service; substitute partner** | MetriStay is software, not an ad agency; integrate an approved metasearch/channel partner for bidding | D-703 (owner: Marketing Manager) |
| Direct GDS connectivity | **Via channel-manager partner** | GDS access requires commercial agreements/certification; build one certified channel adapter (M07) | Channel partner selection D-704 |
| Embedded payments as merchant/PSP (Mews "embedded payments", "financing partnership") | **Excluded** | MetriStay orchestrates a certified PSP; it does not hold funds, underwrite or lend (Section E, P.4) | Never without licensing decision |
| Proprietary AI foundation model (Cloudbeds "Signals") | **Excluded** | Pluggable LLM/provider port and local option (README §3.7); bounded tools only | ADR in `docs/03` |
| Marketplace with 1,000+/450+ integrations | **Deferred — M33 matures in Phases 5–7; M37 marketplace Phase 8** | R1 needs a small set of certified adapters with honest capability flags, not breadth | Integration demand log |
| Autonomous dynamic pricing without approval | **Deferred — Phase 7 advanced automation** | Section Q SF53.2.2 requires human approval/guardrails in R1 | Backtest evidence SF53.2.6 |
| Loyalty cash/stored-value wallet | **Gated** (SF30.2.6) | Requires licensed bank/PSP; CBO policy | Legal opinion D-705 |
| Multi-level/network referral schemes | **Excluded permanently from this plan** | Oman Decision 105/2021; Section A/E | n/a |
| UC/wake-up, HSIA, IPTV | **Deferred — Phase 7** (M34–M36) | Section B | Pilot demand |
| Biometric liveness as default | **Excluded as default; optional gated** | SF41.1.6 lawful basis per market | Counsel per market |

---

## 6. Substitute partner strategy

Honesty label for every row today: `unverified-assumption` (no partner is contracted). Candidate names are examples to evaluate, not selections.

| Capability | MetriStay builds | Partner provides (category) | Example candidates to evaluate | Contract/certification gate | Manual route if absent |
|---|---|---|---|---|---|
| OTA/GDS distribution | M07 adapter, mapping, ARI queue, reconciliation | Certified channel manager | SiteMinder, Cloudbeds channel manager, others offering a PMS-connectivity programme | Partner certification of M07 adapter (D-704) | Extranet updates by revenue manager with dual-entry checklist; stop-sell on uncertainty |
| Metasearch/search ads | Offer feed, attribution tags (SF51.2.3) | Metasearch connectivity/bidding partner | Via channel partner or direct hotel-ads programmes | Commercial access | Organic listing only |
| Card payments | M28 orchestration, token refs, reconciliation | Certified PSP/acquirer + terminals | Pilot-market PSPs (D-706) | PSP certification; PCI scope assessment | Standalone terminal + manual folio posting with reference |
| Competitive rate data | Input slot SF53.1.4 | Licensed rate-shopping provider | TBD | Licence terms | Forecast without compset; flag lower confidence |
| Review ingestion | M52 queue | Review platform APIs/aggregator | TBD | API terms | Manual copy of public review into case with link |
| SMS/WhatsApp | M41/M52 templates | Messaging provider/BSP | TBD per market | Template approval | Email/in-person verification (SF41.2.7) |
| E-signature | Evidence envelope SF41.2.3 | Signature provider where legal effect requires | TBD | Counsel per market | Wet signature scanned with hash |
| Door locks/kiosk | Phase 7 port | Lock/kiosk vendors | TBD | Vendor certification | Physical keys/encoders per `docs/12` §5 |
| Bill-pay | M29 provider-neutral port | Khedmah/ONEIC (candidates) | — | SF29.2.1 | Bank payment with evidence |
| Travel (air/cruise/taxi) | M45 case/offer/order | Licensed agency/supplier APIs | Examples in Section K M45 | SF45.3.x market gate | Referral-only with manual RFQ |

---

## 7. Migration comparison

MetriStay R1 migration principle (`docs/09` owns execution detail): **open balances and future business migrate; history migrates as read-only reference; no financial history is re-posted as new transactions.**

| Source system | Likely export route (to verify per contract) | Objects to migrate | MetriStay target | Specific risks | Coexistence option |
|---|---|---|---|---|---|
| Oracle OPERA (on-prem or Cloud) | Vendor/partner APIs via integration platform (contract needed) or file exports | Profiles, future reservations, blocks, events/BEOs, rate codes, AR ledger, deposits | M05, M12, M04, M10, M20, M08 | Complex rate-code and routing structures; package components; BEO fidelity | Rarely: sales & events could remain temporarily with interface — not planned for R1 |
| Mews | Open API (claimed) / exports | Profiles, reservations, rates, bills/AR, product catalog | M05, M04, M08, M20, M54 | Embedded-payment tokens are PSP-bound: **card tokens generally cannot be moved** without PSP-to-PSP token migration | Keep PSP if certifiable with MetriStay M28 |
| Cloudbeds | API/exports | Profiles, reservations, rates, website content, CRM consent | M05, M04, M51, M52 | Consent evidence must migrate with source/timestamp or be re-collected | Keep Cloudbeds channel manager if certified |
| SiteMinder | Not a PMS; channel mappings and booking engine settings | Room/rate mapping, channel list, direct-booking attribution | M07, M51 | Mapping errors cause oversell; double-feed during cutover | **Likely coexist** as channel manager partner |
| Spreadsheets/legacy | CSV templates | Vendors, items/UOM, assets, employees, meters | M46, M14, M26, M27, M22 | Duplicate masters, missing UOM conversions | n/a |

Cutover controls (all sources): freeze window with stop-sell on channels; ARI full refresh + acknowledgment; reservation count and room-night reconciliation per stay date; deposit and AR balance tie-out to GL; consent import report; rollback = re-enable source with delta log (`docs/09`).

---

## 8. Operating-cost comparison (qualitative)

No prices are stated or implied. "Driver" means a cost the hotel should expect to evaluate; direction is relative to a typical all-SaaS PMS + separate channel manager stack and is a hypothesis for the pilot business case (`docs/00`).

| Cost driver | Typical SaaS PMS stack (benchmarks) | MetriStay SaaS profile | MetriStay on-prem single-hotel profile | Notes / evidence needed |
|---|---|---|---|---|
| Software subscription | Per room/property subscriptions, add-on modules priced separately | One suite; hotel enables only owned outlets | Licence + support | Pricing model is a Metrikingdom decision (D-707) |
| Channel/OTA commission | Charged by OTAs; unchanged by PMS | Unchanged; M53/M51 measure net contribution to shift mix | Same | Savings only if direct mix improves — measure, don't assume |
| Payment fees | PSP fees; embedded-payment vendors may bundle | PSP fees paid to certified PSP | Same | Reconciliation labour reduced only if SF28.1.7 works |
| Integrations | Marketplace connectors, sometimes per-connector fees | Fewer third-party modules needed (ERP, procurement, utilities in-suite) | Same | Each certified adapter still has partner cost |
| Hardware | Kiosks/keys/terminals | Terminals, scanners/scales, LPR cameras, staff devices | + hotel server, UPS, offsite backup | `docs/12` §5 device list |
| Hosting/ops | Included in SaaS | Included | Hotel IT or MSP labour, patching, restore drills | On-prem needs documented RTO evidence |
| AI compute | Bundled or metered | Metered per provider; local model option | Local GPU/CPU capacity | SF39.2.5, SF40.2.7 cost metering |
| Training/change | Per product | One suite, role home screens | Same | `docs/13` simplicity rules |
| Compliance/legal | Hotel's responsibility | Rule packs require counsel validation per market | Same | M44 evidence costs are real |

---

## 9. Owner-question traceability table (Section O)

Every Section O question has its own row. Compound Section O questions that ask about independently testable outcomes are split into separately numbered rows (e.g. the guest "modify, pay, check in, request help, get a receipt, claim points and reach a human" question becomes Q-GST-4 … Q-GST-10), and the split is noted. Rows marked **(discovered)** are Phase 1 additions required by Section O ("expand the catalogue with site-, facility- and jurisdiction-specific questions") so that every module has a traceable question (§11).

Abbreviations: `…/` = `/v1/properties/{pid}/`. "F-id per docs/01" = module has no Section K/Q feature id (see §0). Every row's decidable workflow (inputs, preconditions, result, evidence, escalation) is specified in the cited subfeature's Section-L block in `docs/01`; this table is the index.

Role codes: OWN owner/asset manager • GM GM/duty manager • REV revenue/sales • MKT marketing/guest relations • FD front desk/concierge • HK housekeeping/laundry • FNB chef/F&B/club • PRC procurement/storekeeper/vendor • MNT maintenance/security • FIN finance/HR • GST guest/corporate buyer • TEC technology/compliance.

### 9.1 Owner / asset manager

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-OWN-1** Is this property profitable after payroll, energy, maintenance, channel, payment and corporate credit costs? | owner | Open profit bridge; drill any line to journal and source event; see estimate vs reconciled label per line | M19 `journal_entry`; M32 projections; M65 lineage | M32, M19, M65, M27, M22, M26, M28, M20 | M32.F32.3.SF32.3.2; M32.F32.3.SF32.3.4 | 4 (foundation 2) | SCR-MGR-profit-bridge; `GET …/reports/profit-bridge?period=` | Missing cost source → line shows "incomplete estimate", never "actual"; period not closed → "provisional" | AC-SF32.3.4; G08, G19 | GOP; net operating result; source-coverage % |
| **Q-OWN-2** Which departments, rooms, events and channels earn or lose money? | owner, financial_controller | Rank contribution by department, room type, event, channel; drill to cost drivers | M32 department P&L; M53 channel contribution; M12 event reconciliation | M32, M53, M12 | M32.F32.2.SF32.2.1–SF32.2.6; M53.F53.1.SF53.1.3 | 4 | SCR-MGR-contribution; `GET …/reports/contribution?dimension=` | Allocation version changed → restated figures carry version label (SF32.3.1) | AC-SF32.3.1; G08, G19 | departmental contribution; channel net contribution per room night |
| **Q-OWN-3** What is accrued versus settled? | owner, financial_controller | View lifecycle columns planned/accrued/invoiced/approved/paid/settled (Section D) | M19 accruals; M20 AP/AR; M28 settlement | M19, M20, M28 | M19.F19.1.SF19.1.4; M20.F20.3.SF20.3.2 | 4 | SCR-FIN-reconciliation-dashboard; `GET …/finance/lifecycle-status` | PSP settlement file late → item stays "paid – unsettled" with age | AC-SF20.3.2; G08 | accrued-not-invoiced value; unsettled aging |
| **Q-OWN-4** Which capex/renovation will pay back? | owner | Compare approved investment case with measured post-project savings/revenue | M66 capex case; M67 measured savings; M32 | M66, M67, M32 | M66.F66.2.SF66.2.1; M67.F67.2.SF67.2.3 | 4–6 | SCR-MGR-capex; `POST …/capex-requests` | No baseline → payback "not measurable", not estimated | AC-SF66.2.1; AC-SF67.2.3 | measured payback period; savings vs baseline |
| **Q-OWN-5** What changed since yesterday/month/year? | owner, gm | Open daily flash with day/MTD/YTD variance and reproducible report version | M65 point-in-time versions; M32 | M65, M32 | M65.F65.2.SF65.2.3; M65.F65.2.SF65.2.2 | 2 foundation; 4–6 | SCR-MGR-flash; `GET …/reports/flash?compare=dod,mtd,ytd` | Late posting after close → restatement banner with reason | AC-SF65.2.2; G08 | occupancy, ADR, RevPAR, TRevPAR, GOP deltas |
| **Q-OWN-6** What cash, debt, insurance and tax obligations are due? | owner, financial_controller | Review 90-day obligations calendar and approval status | M20 treasury forecast; M66 debt inputs; M68 premiums; M38 remittance calendar | M20, M66, M68, M38 | M20.F20.3.SF20.3.3; M66.F66.1.SF66.1.4; M68.F68.1.SF68.1.2; M38.F38.1.SF38.1.5 | 4 | SCR-MGR-obligations; `GET …/obligations?horizon=90d` | Rule pack unverified → filing obligation shows "manual/compliance review" | AC-SF20.3.3; AC-SF38.1.5; G09 | cash runway days; overdue obligations |
| **Q-OWN-7** (discovered) Which management/brand fees and owner statements are due and approved? | owner | Approve fee calculation; publish restricted owner statement | M66 fee schedule and statement | M66 | M66.F66.1.SF66.1.2; M66.F66.1.SF66.1.3 | 4–6 | SCR-MGR-owner-statement | Fee basis unreconciled → statement held | AC-SF66.1.3 | statement on-time; fee corrections |
| **Q-OWN-8** (discovered) Is energy/water/waste per occupied room improving on verified data? | owner, chief_engineer | Review intensity with missing/estimated/verified flags | M67 measures; M22/M23 meters | M67, M22, M23 | M67.F67.1.SF67.1.2; M67.F67.1.SF67.1.3 | 4–5 | SCR-MGR-sustainability | Meter gap → estimated value flagged; no "green" claim (SF67.2.4) | AC-SF67.2.4 | utility cost per occupied room; kWh per occupied room |

### 9.2 GM / duty manager

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-GM-1** Which arrivals, departures, VIPs, accessibility requests, room defects, overbookings and incidents need action now? | gm, duty_manager | Work the 3–7-item prioritized inbox; assign owner and due time | M05 stays; M55 cases; M26 work orders; M42 incidents; M03 overbooking position | M55, M05, M03, M26, M42, M63 | M55.F55.1.SF55.1.3; M63.F63.1.SF63.1.2 | 2–3 | SCR-MGR-today; `GET …/inbox?role=duty_manager` | Oversold beyond controlled limit → walk workflow with relocation (M05, SF68.2.3) | AC-SF55.1.3; G19 | open exceptions past SLA; arrivals with unresolved needs |
| **Q-GM-2** Which shift/team/chef coverage is missing? | gm, executive_chef | See uncovered shifts before cutoff; trigger callout | M27 roster; M47 coverage slots | M62, M47, M27 | M62.F62.2.SF62.2.1; M47.F47.1.SF47.1.3 | 3 | SCR-MGR-coverage; `GET …/coverage-gaps?date=` | Nobody accepts callout → manager escalation and menu contingency (SF47.2.6) | AC-SF47.1.3; G15 | uncovered shift-hours; callout time-to-fill |
| **Q-GM-3** What is sold, blocked, clean, in service or out of order? | gm, front_office_manager | Read one authoritative room/space status grid | M03 inventory; M56 cleaning state; M26 OOO | M03, M06, M56, M26, M09 | M56.F56.1.SF56.1.5; M26.F26.1.SF26.1.5; M03⟨OOO/OOS, holds⟩ (F-id per docs/01) | 2 | SCR-FD-room-grid; `GET …/room-status` | Offline device edit conflicts → conflict queue, server state authoritative | AC-SF56.1.5; G19 | sellable rooms; OOO room nights |
| **Q-GM-4** Who owns every unresolved exception and by when? | gm, duty_manager | Unified exception queue sorted by SLA; reassign | M63 `task` | M63 | M63.F63.1.SF63.1.2; M63.F63.1.SF63.1.5 | 2 | SCR-OPS-exception-queue; `GET …/tasks?status=open` | Owner off shift → auto-reassign to duty manager | AC-SF63.1.5; G20 | exceptions without owner (target 0); SLA breach rate |
| **Q-GM-5** Can we operate if internet or a provider fails? | gm, it_admin | Check degraded-service status; invoke manual runbook | M64 health; M68 continuity plan | M64, M68 | M64.F64.1.SF64.1.3; M64.F64.1.SF64.1.5; M68.F68.2.SF68.2.4 | 2–6 | SCR-MGR-continuity-status; `GET …/health/degraded-services` | On-prem node down → paper packs and offline staff app (`docs/12` §5–6) | AC-SF64.1.5; G20 | minutes in degraded mode; manual items pending sync |
| **Q-GM-6** (discovered) Are optional amenities (spa/pool/gym) safe, staffed and not oversold? | gm | Check capacity, practitioner qualification and sanitation | M58 | M58 | M58.F58.1.SF58.1.2; M58.F58.2.SF58.2.6 | 3–7 | SCR-MGR-amenities | Practitioner credential expired → slot unsellable | AC-SF58.2.6 | amenity utilization; amenity incidents |
| **Q-GM-7** (discovered) Is the club/bar within capacity and licensing rules? | gm, club_host | Monitor occupancy, age and responsible-service gates | M15 access events; M57 gates | M15, M57 | M15⟨capacity, entry/re-entry⟩ (F-id per docs/01); M57.F57.1.SF57.1.5 | 3 | SCR-CLUB-door; `GET …/club/occupancy` | Capacity reached → entry denied, waitlist | AC-SF57.1.5; G03 | peak occupancy vs limit |

### 9.3 Revenue / sales

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-REV-1** Which dates and segments need a rate/restriction change? | revenue_manager | Review candidate actions; approve within role guardrails | M53 recommendation | M53, M04 | M53.F53.2.SF53.2.1; M53.F53.2.SF53.2.2 | 5 (baseline 3) | SCR-REV-rate-actions; `POST …/rate-actions/{id}/approve` | Outside guardrail → higher approver; channel ack missing → discrepancy queue (SF53.2.5) | AC-SF53.2.2; G19 | net RevPAR; forecast accuracy |
| **Q-REV-2** Why is demand changing and how certain is the forecast? | revenue_manager | Inspect drivers, confidence band, data coverage | M53 forecast | M53 | M53.F53.1.SF53.1.5; M53.F53.1.SF53.1.4 | 5 | SCR-REV-forecast; `GET …/forecast?horizon=90` | Low coverage → "low confidence", no recommendation issued | AC-SF53.1.5; G19 | forecast error by horizon |
| **Q-REV-3** Which OTA or campaign drives profitable stays? | revenue_manager, marketing_manager | Compare net contribution by channel/campaign | M53, M51 attribution; M07 commissions | M53, M51, M07 | M53.F53.1.SF53.1.3; M51.F51.2.SF51.2.5 | 3–5 | SCR-REV-channel-mix; `GET …/reports/channel-contribution` | Commission invoice unmatched → contribution shown as estimate | AC-SF51.2.5; G19 | net contribution per booking by source; acquisition cost |
| **Q-REV-4** How many direct visitors abandon a quote and why? | marketing_manager, revenue_manager | Read funnel by step and reason; start consented recovery | M51 funnel events | M51 | M51.F51.2.SF51.2.2; M51.F51.2.SF51.2.6 | 2–5 | SCR-MKT-funnel; `GET …/funnel?from=&to=` | Bot/duplicate sessions → filtered and reported separately | AC-SF51.2.6; G19 | quote-to-book conversion; abandonment reasons |
| **Q-REV-5** Which corporate RFQ is feasible with attendee count, rooms, venues, catering and credit? | sales_manager | Run feasibility; place composite hold; check credit | M09 allocations; M03 inventory; M16 capacity; M10/M20 credit | M10, M09, M12, M16, M20 | M10⟨RFQ, credit⟩ (F-id per docs/01); M20.F20.2.SF20.2.1 | 3 | SCR-SALES-rfq-feasibility; `POST …/composite-holds` | Partial feasibility → alternative dates/layouts; credit exceeded → approval | AC-SF20.2.1; G01 | RFQ response time; win rate |
| **Q-REV-6** What is pickup to contract versus forecast? | sales_manager, revenue_manager | Compare block pickup to contract and forecast; act before cutoff | M12 blocks; M53 | M32, M53, M12 | M32.F32.1.SF32.1.4; M53.F53.1.SF53.1.2 | 3 | SCR-REV-block-pickup | Attrition trigger → sales task; cutoff → release to general inventory | AC-SF32.1.4; G01 | block pickup %; group wash % |

### 9.4 Marketing / guest relations

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-MKT-1** Can guests discover accurate room, event and amenity listings? | marketing_manager, content_editor | Run listing-consistency audit vs inventory attributes | M51 CMS; M39 media; M03 attributes | M51, M39, M03 | M51.F51.1.SF51.1.2; M51.F51.1.SF51.1.3 | 2 | SCR-MKT-listing-audit; `GET …/content/consistency-report` | Attribute mismatch (e.g. accessible feature) → publish blocked | AC-SF51.1.3; G19 | listing accuracy defects |
| **Q-MKT-2** Which photos/videos are current and rights-cleared? | content_approver | Review rights/expiry; take down | M39 | M39 | M39.F39.1.SF39.1.2; M39.F39.1.SF39.1.7 | 2 | SCR-MKT-media-library; `GET …/media?rights=expiring` | Rights expired → auto-unpublish and syndication takedown | AC-SF39.1.7; G13 | assets with expiring rights |
| **Q-MKT-3** Which campaign can contact which consenting guests? | marketing_manager | Build segment; system applies consent/suppression | M02 `consent_record`; M52 | M52, M02 | M52.F52.1.SF52.1.3; M52.F52.1.SF52.1.4 | 3–5 | SCR-MKT-segment-builder; `POST …/segments/preview` | Consent unknown for channel/country → excluded | AC-SF52.1.3 | reachable consented audience; opt-out rate |
| **Q-MKT-4** What did it cost per completed stay? | marketing_manager | Attribute campaign spend to completed paid stays | M52 campaigns; M51 attribution; M20 spend | M52, M51, M20 | M52.F52.1.SF52.1.6; M51.F51.2.SF51.2.5 | 5 | SCR-MKT-campaign-roi | Spend invoice missing → estimate label | AC-SF52.1.6 | cost per completed stay |
| **Q-MKT-5** What complaints or public reviews need response? | guest_relations | Work case/review queue; send approved response | M55 cases; M52 reviews | M52, M55 | M52.F52.2.SF52.2.3; M55.F55.2.SF55.2.1 | 3–4 | SCR-MKT-reputation-queue; `GET …/reviews?status=needs_response` | Review contains guest PII → redaction before reply | AC-SF52.2.3; G19 | review response time; cases past SLA |
| **Q-MKT-6** Which guests return, and what service recovery actually closed the case? | guest_relations, marketing_manager | Repeat-guest report; recovery closure evidence | M52 profiles; M55 cases | M52, M55 | M55.F55.2.SF55.2.4; M55.F55.2.SF55.2.5; M52.F52.2.SF52.2.4 | 4 | SCR-MKT-recovery-outcomes | Guest does not confirm → "awaiting confirmation", closes with reason after timer | AC-SF55.2.4; G19 | repeat rate; recovery closure rate; recovery cost |
| **Q-MKT-7** (discovered) Is the guest AI assistant answering from approved sources and handing off correctly? | guest_relations, content_approver | Review sampled transcripts; KB freshness | M40 KB and transcripts | M40 | M40.F40.1.SF40.1.1; M40.F40.1.SF40.1.6; M40.F40.2.SF40.2.3 | 3–5 | SCR-MKT-assistant-quality | Stale KB article → answer suppressed, handoff | AC-SF40.1.5; G13 | containment; handoff wait; incorrect-answer rate |

### 9.5 Front desk / concierge

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-FD-1** Can this guest check in now; is identity, room readiness, payment and registration complete? | front_desk_agent | Open readiness card; clear each blocker | M05 `stay`; M41; M56 room state; M28 | M05, M41, M56, M28, M06 | M41.F41.1.SF41.1.5; M41.F41.2.SF41.2.1–SF41.2.4; M56.F56.1.SF56.1.4 | 2–3 | SCR-FD-checkin; `POST …/stays/{id}/check-in` | ID mismatch → manual review; room not released → ETA and offer | AC-SF41.2.4; G13, G19 | check-in duration; first-time-right check-ins |
| **Q-FD-2** Which rate and taxes apply? | front_desk_agent | Show rate plan, policy snapshot and tax lines | M04 `quote`/`policy_snapshot`; M44 rule pack | M04, M38, M44 | M38.F38.1.SF38.1.3; M44.F44.1.SF44.1.3 | 2 | SCR-FD-rate-detail; `GET …/quotes/{id}` | Rule pack unverified → quote flagged, supervisor review | AC-SF38.1.3; G12 | post-stay tax adjustments |
| **Q-FD-3** Can I move or extend the stay without double-selling? | front_desk_agent | Check inventory; move/extend atomically | M03 room-type nights; M05 | M03, M05, M54 | M03⟨holds, assignment⟩ (F-id per docs/01); M54.F54.1.SF54.1.2 | 2 | SCR-FD-move-extend; `POST …/stays/{id}/extend` | No inventory → alternatives/waitlist | AC-SF54.1.2; G20 | oversell incidents (target 0) |
| **Q-FD-4** What if a card is declined or internet is down? | front_desk_agent, cashier | Offer alternative tender; switch to offline mode | M28; M64 offline queue | M28, M64, M55 | M28.F28.1.SF28.1.3; M28.F28.1.SF28.1.4; M64.F64.2.SF64.2.2; M55.F55.1.SF55.1.6 | 2 | SCR-FD-payment; `POST …/payment-intents` | Offline → no card capture except standalone terminal with reference; later sync | AC-SF64.2.2; G20 | payment recovery rate |
| **Q-FD-5** Can I arrange taxi/flight/cruise with guest approval and an actual supplier confirmation? | concierge | Create request; capture consent; quote; record external reference | M45 `travel_order`; M59 trips | M45, M59 | M45.F45.1.SF45.1.2; M45.F45.2.SF45.2.5; M45.F45.2.SF45.2.6; M59.F59.1.SF59.1.1 | 3–5 | SCR-CON-travel-request; `POST …/travel-requests` | Quote expired / duplicate callback → never "booked" without external reference | AC-SF45.2.6; G11 | confirmed vs requested; disruption resolution time |
| **Q-FD-6** Is a lost item matched safely to this guest? | front_desk_agent, security_officer | Privacy-limited match; verify claim; release | M43 custody | M43 | M43.F43.1.SF43.1.4; M43.F43.1.SF43.1.5; M43.F43.1.SF43.1.6 | 2–3 | SCR-SEC-lost-found; `POST …/lost-items/{id}/release` | Insufficient evidence → hold, supervisor | AC-SF43.1.5; G14 | match rate; custody exceptions |
| **Q-FD-7** (discovered) Can this guest's vehicle enter and is parking billed once? | parking_attendant, front_desk_agent | Register permit/plate; confirm gate decision and folio post | M17 `parking_session`; M08 | M17, M08 | M17⟨permits, gate decision, folio posting⟩ (F-id per docs/01) | 3 | SCR-PARK-lane; `POST …/parking/gate-decisions` | Low-confidence read → manual review; override audited | AC-M17 per docs/01; G03 | duplicate parking charges (0) |

### 9.6 Housekeeping / laundry

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-HK-1** What rooms are due, prioritized or DND? | housekeeping_supervisor | Open priority board; assign by capacity | M56 tasks | M56, M06 | M56.F56.1.SF56.1.1; M56.F56.1.SF56.1.2 | 2 | SCR-HK-board; `GET …/hk/tasks?date=` | DND beyond policy → welfare check (`docs/12` §3.1) | AC-SF56.1.1; G19 | rooms ready by arrival |
| **Q-HK-2** Which tasks or inspections failed and what is the ETA? | housekeeping_supervisor | Assign reclean; update ETA | M56 inspections | M56 | M56.F56.1.SF56.1.3; M56.F56.1.SF56.1.5 | 2 | SCR-HK-inspections | Repeat failure → coaching (SF62.2.3) | AC-SF56.1.3 | inspection pass rate; turnaround minutes |
| **Q-HK-3** Do we have enough clean linen and amenities for arrivals? | housekeeping_supervisor, laundry_attendant | Compare par to forecast; reorder | M56 par; M14 stock | M56, M14 | M56.F56.2.SF56.2.1; M56.F56.2.SF56.2.5 | 3–4 | SCR-HK-linen-par | Laundry late → emergency par release | AC-SF56.2.5 | par coverage days |
| **Q-HK-4** Where did linen go, what was lost or stained, and who accepted its return? | laundry_attendant | Record custody transfers, weights and damage | M56 custody ledger | M56 | M56.F56.2.SF56.2.2; M56.F56.2.SF56.2.3 | 3–4 | SCR-HK-laundry-custody; `POST …/linen/transfers` | Weight/count mismatch → vendor claim | AC-SF56.2.2; G19 | linen loss rate; laundry cost per occupied room |
| **Q-HK-5** Has minibar consumption been posted once? | housekeeper, cashier | Count; post; handle dispute | M56 count; M08 folio | M56, M08 | M56.F56.2.SF56.2.4; M56.F56.2.SF56.2.6 | 3 | SCR-HK-minibar; `POST …/minibar/counts` | Duplicate post → rejected by source key (INV-FOL-1) | AC-SF56.2.4 | minibar disputes; duplicate posts (0) |

### 9.7 Chef / F&B / club

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-FNB-1** What covers and BEO portions are due, with dietary/allergen restrictions? | executive_chef | Open production plan with allergen flags | M12 `beo_version`; M16 `production_plan` | M16, M57, M12 | M57.F57.1.SF57.1.4; M16⟨allergen flags, production⟩ (F-id per docs/01) | 3 | SCR-KIT-production; `GET …/production-plans?date=` | BEO change after cutoff → change-order approval | AC-SF57.1.4; G02 | on-time production; allergen incidents |
| **Q-FNB-2** Which chef and backup will work, and who is the approved emergency replacement? | executive_chef, fnb_manager | Review coverage plan; trigger callout | M47 | M47 | M47.F47.1.SF47.1.1; M47.F47.1.SF47.1.6; M47.F47.2.SF47.2.3 | 2–3 | SCR-KIT-chef-coverage | No acceptance → contingency (SF47.2.6) | AC-SF47.2.3; G15 | callout time-to-fill |
| **Q-FNB-3** What batches and stock are safe and available? | executive_chef, storekeeper | View lot status by store/bin | M14/M50 `stock_ledger_entry` | M50, M14 | M50.F50.3.SF50.3.1; M50.F50.3.SF50.3.8 | 3–4 | SCR-STR-lot-status | Expired/quarantined lot → unavailable | AC-SF50.3.9; G18 | expired stock value; average stock age |
| **Q-FNB-4** What should recipe use versus actual? | executive_chef, fnb_manager | Review theoretical vs actual consumption | M50 estimate; M14 recipes; M13 sales | M50, M14, M13 | M50.F50.3.SF50.3.3; M50.F50.3.SF50.3.7 | 4 | SCR-STR-variance | Variance above threshold → investigation task | AC-SF50.3.7; G18 | recipe variance %; food cost % |
| **Q-FNB-5** Which lot went into which event and can it be recalled? | executive_chef, compliance_officer | Trace; place recall hold | M57 batch links; M50 trace | M57, M50, M61 | M57.F57.2.SF57.2.4; M50.F50.2.SF50.2.10; M61.F61.2.SF61.2.2 | 4 | SCR-STR-recall; `POST …/recall-cases` | Lot not captured → trace gap reported | AC-SF57.2.4; G18 | trace completeness; recall hold time |
| **Q-FNB-6** Which tips/voids/waste require approval? | fnb_manager | Approve queue with SoD | M13 adjustments; M60; M50 waste | M60, M50, M13, M57 | M60.F60.2.SF60.2.2; M50.F50.3.SF50.3.6; M57.F57.1.SF57.1.3 | 3–4 | SCR-POS-approvals | Approver = requester → blocked | AC-SF60.2.2 | void %; waste cost |

### 9.8 Procurement / storekeeper / vendor

Section O's "Has PO been accepted, delivery occurred, goods passed inspection, and invoice matched?" is split into Q-PRC-6 … Q-PRC-9.

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-PRC-1** Which SKU or job is needed, when and at what price? | procurement_officer, storekeeper | Raise requisition from reorder or job | M49 `requisition`; M21 policy | M49, M21 | M49.F49.1.SF49.1.1; M49.F49.1.SF49.1.2; M21.F21.1.SF21.1.1 | 3 | SCR-PRC-requisition; `POST …/requisitions` | Budget exceeded → approval | AC-SF49.1.2; G17 | requisition cycle time |
| **Q-PRC-2** Which vendors are licensed, stocked, available and historically reliable? | procurement_officer | Department-scoped search with freshness badge | M46 vendor; M48 stock snapshot | M46, M48 | M46.F46.2.SF46.2.1; M46.F46.2.SF46.2.2; M48.F48.3.SF48.3.1 | 3 | SCR-PRC-vendor-search | Stale stock → not guaranteed; unapproved → excluded | AC-SF46.2.7; G10, G16 | eligible vendors per category; OTIF |
| **Q-PRC-3** Are enough comparable quotations present? | procurement_officer | RFQ with configured minimum; waiver path | M49 `rfq`/`bid` | M49 | M49.F49.1.SF49.1.5; M49.F49.1.SF49.1.6 | 3 | SCR-PRC-rfq | Only two quotes → waiver with higher approver | AC-SF49.1.5; G17 | competitive-quote compliance |
| **Q-PRC-4** Did samples remain for exactly the approved retention period? | procurement_officer, dpo | Check retention clock and purge proof | M49 `sample_retention_clock`, `purge_proof` | M49 | M49.F49.1.SF49.1.4 | 3 | SCR-PRC-sample-retention | Approved hold → purge suspended with reason | AC-SF49.1.4; G17 | purge-on-time rate |
| **Q-PRC-5** Which weighted bid won and why? | procurement_approver | Review evaluation and award justification | M49 `bid_evaluation`, `award` | M49 | M49.F49.2.SF49.2.2; M49.F49.2.SF49.2.7 | 3 | SCR-PRC-bid-compare | Override → reason and SoD audit | AC-SF49.2.7; G17 | override rate |
| **Q-PRC-6** Has the PO been accepted? | procurement_officer | Track supplier acknowledgment | M49 `po_acknowledgment` | M49 | M49.F49.3.SF49.3.5 | 3 | SCR-PRC-po; `GET …/purchase-orders/{id}` | No ack by deadline → follow-up (SF50.1.3) | AC-SF49.3.5; G17 | acknowledgment time |
| **Q-PRC-7** Has delivery occurred? | receiver | Confirm milestones; delivery only from receipt evidence | M50 `po_milestone`, `goods_receipt` | M50 | M50.F50.1.SF50.1.1; M50.F50.1.SF50.1.7 | 3 | SCR-RCV-dock | Chat/GPS/invoice alone → not delivered | AC-SF50.1.7; G18 | OTIF |
| **Q-PRC-8** Did goods pass inspection? | receiver | Temperature/condition checks; quarantine | M50 `receiving_evidence` | M50 | M50.F50.2.SF50.2.4; M50.F50.2.SF50.2.6 | 3–4 | SCR-RCV-inspection | Breach → quarantine and vendor claim | AC-SF50.2.6; G18 | rejection rate |
| **Q-PRC-9** Has the invoice matched? | ap_clerk | Three-way match within tolerance | M20 `invoice_match` | M20, M49 | M20.F20.1.SF20.1.4; M49.F49.3.SF49.3.6 | 4 | SCR-FIN-invoice-match | Tolerance exceeded → hold; PO alone never pays (SF49.3.7) | AC-SF49.3.7; G18 | first-pass match rate |
| **Q-PRC-10** What stock is issued, intact-returned, quarantined or disposed? | storekeeper | Read store ledger by status | M14/M50 ledger | M50, M14 | M50.F50.3.SF50.3.2; M50.F50.3.SF50.3.4; M50.F50.3.SF50.3.5 | 3–4 | SCR-STR-ledger | Discarded item to sellable → blocked (INV-STK-2) | AC-SF50.3.9; G18 | waste cost; shrinkage |

### 9.9 Maintenance / security

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-MNT-1** Which guest/asset fault is critical, which room must be removed from sale, and what is the repair SLA? | chief_engineer, engineer | Triage; set OOO; dispatch with SLA | M26 `work_order` | M26, M03 | M26.F26.1.SF26.1.3; M26.F26.1.SF26.1.4; M26.F26.1.SF26.1.5 | 3 | SCR-ENG-work-orders; `POST …/work-orders` | Occupied/sold room OOO → relocation workflow | AC-SF26.1.5; G19 | SLA compliance; OOO room nights |
| **Q-MNT-2** Which contractor can work safely? | chief_engineer | Check credentials, briefing, access | M46 documents; M62 briefing | M26, M46, M62 | M26.F26.2.SF26.2.1; M26.F26.2.SF26.2.3; M62.F62.1.SF62.1.4 | 3–4 | SCR-ENG-contractor-access | Insurance expired → access denied | AC-SF62.1.5; G10 | contractor compliance |
| **Q-MNT-3** Did a fire/gas/water alarm reach an accountable person? | security_officer | Verify acknowledgment chain | M42 incident chronology | M42 | M42.F42.2.SF42.2.3; M42.F42.2.SF42.2.4 | 3–4 | SCR-SEC-incident-command | No ack before timeout → escalate + phone tree | AC-SF42.2.4; G14 | time to acknowledge |
| **Q-MNT-4** Is the parking gate override auditable? | security_officer, parking_attendant | Override with reason; review log | M17 `gate_command`; M64 | M17, M64 | M17⟨manual fallback, override⟩ (F-id per docs/01); M64.F64.1.SF64.1.5 | 3 | SCR-PARK-override-log | Override without reason → not allowed | AC-SF64.1.5; G03 | overrides per 100 sessions |
| **Q-MNT-5** What evidence preserves an incident or insurance claim? | security_officer, financial_controller | Assemble evidence packet | M42 chronology; M68 claim | M42, M68 | M42.F42.2.SF42.2.6; M68.F68.1.SF68.1.3 | 3–4 | SCR-SEC-evidence-packet | Footage retention expiring → legal hold | AC-SF68.1.3; G14 | claim evidence completeness |

### 9.10 Finance / HR

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-FIN-1** Do guest folio, bank/PSP, POS, corporate AR, AP and GL reconcile? | financial_controller | Work reconciliation dashboard | M08, M28, M13, M20, M19 | M20, M28, M19, M08, M13 | M20.F20.3.SF20.3.2; M20.F20.3.SF20.3.5; M28.F28.1.SF28.1.7 | 4–5 | SCR-FIN-reconciliation | Unmatched → exception queue with age | AC-SF28.1.7; G08, G20 | unmatched items aging |
| **Q-FIN-2** Are all charges, refunds, tips, wages, utilities, invoices and taxes recognized once? | financial_controller | Review duplicate detection and GL mapping | M19 mapping; M60 integrity | M19, M60 | M19.F19.1.SF19.1.2; M60.F60.2.SF60.2.1 | 4 | SCR-FIN-integrity | Duplicate → investigation case | AC-SF60.2.1; G20 | duplicates detected/resolved |
| **Q-FIN-3** Who can see salary and ID? | hr_officer, dpo | Review access audit | M27; M02; M41 | M27, M02, M41 | M27.F27.4 (Section K gives no SF; F-level cited); M27.F27.1.SF27.1.5; M41.F41.1.SF41.1.7 | 4 | SCR-HR-access-audit | Unauthorized attempt → alert | AC-SF27.1.5; G05 | privileged access exceptions |
| **Q-FIN-4** Are cash shifts and night audits closed? | night_auditor, cashier | Close shifts; run night audit | M60; M08 | M60, M08 | M60.F60.1.SF60.1.3; M60.F60.1.SF60.1.4 | 2 | SCR-FIN-night-audit | Difference → queue | AC-SF60.1.4; G08 | cash variance |
| **Q-FIN-5** Which country/location filing is verified and where is its receipt? | compliance_officer | Open filing registry | M38; M44 | M38, M44 | M38.F38.3.SF38.3.6; M38.F38.3.SF38.3.8; M44.F44.2.SF44.2.3 | 4–6 | SCR-CMP-filings | Unverified pack → manual route, never "submitted" | AC-SF38.3.6; G09, G12 | filings with receipt % |
| **Q-FIN-6** What happens on failed payroll/bill payment? | payroll_officer, finance_approver | Resubmit rejection; inquire pending | M27; M29; M28 | M27, M29, M28 | M27.F27.3.SF27.3.7; M29.F29.1.SF29.1.7; M28.F28.2.SF28.2.4 | 4–5 | SCR-FIN-payment-exceptions | Timeout → inquiry before any retry | AC-SF29.1.7; G05, G06, G20 | payroll errors; stuck payments |
| **Q-FIN-7** (discovered) Are loyalty liability and referral commissions accrued correctly and paid only when eligible? | financial_controller, referral_program_admin | Review liability; approve commissions | M30; M31 ledgers | M30, M31 | M30.F30.1.SF30.1.7; M31.F31.1.SF31.1.5; M31.F31.1.SF31.1.7 | 5–6 | SCR-FIN-loyalty-liability | Disabled jurisdiction → payout rejected | AC-SF31.1.7; G07 | points liability; commission reversals |
| **Q-FIN-8** (discovered) Was each utility bill paid via an authorized channel with provider confirmation? | ap_clerk | Work bill inbox; pay; confirm | M22–M25 bills; M29 | M22, M23, M24, M25, M29 | M22.F22.2.SF22.2.5; M22.F22.2.SF22.2.6; M23.F23.1.SF23.1.6; M24.F24.1.SF24.1.6; M25.F25.1.SF25.1.5; cross-utility guard (F-level, no SF) | 4–5 | SCR-FIN-utility-inbox | Bill-pay partner not contracted → bank payment with evidence | AC-SF22.2.6; G04, G06 | bill variance; late fees |

### 9.11 Guest / corporate buyer

Section O's "Can I modify, pay, check in, request help, get a receipt, claim points and reach a human?" is split into Q-GST-4 … Q-GST-10.

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-GST-1** Can I find a suitable accessible room/facility at an honest total price? | guest, booker | Search with accessibility filter; see total price | M03, M04, M51 | M51, M04, M03 | M51.F51.2.SF51.2.1; M51.F51.2.SF51.2.2; M51.F51.1.SF51.1.3 | 2 | SCR-WEB-search; `GET /v1/public/properties/{pid}/availability` | No accessible room → assisted contact | AC-SF51.2.2; G19 | quote-to-book conversion; total-price complaints (0) |
| **Q-GST-2** Can I compare attendee capacity/rate on actual dates and receive a firm expiry? | corporate_booker | Compare; hold with expiry | M09; M04 | M11, M09, M04 | M11⟨search/compare/hold⟩ (F-id per docs/01); M04⟨quote expiry⟩ | 3 | SCR-CORP-compare; `POST …/composite-holds` | Hold expiry → automatic release | AC-M11 per docs/01; G01 | RFQ-to-hold time |
| **Q-GST-3** Can I correct ID OCR, choose a non-biometric path and sign/verify safely? | guest | Confirm fields; e-sign; OTP/QR | M41 | M41 | M41.F41.1.SF41.1.5; M41.F41.1.SF41.1.6; M41.F41.2.SF41.2.3; M41.F41.2.SF41.2.5; M41.F41.2.SF41.2.6 | 2–3 | SCR-GST-id-confirm | OTP not delivered → in-person (SF41.2.7) | AC-SF41.2.6; G13 | OCR correction rate |
| **Q-GST-4** Can I modify my booking? | guest | Amend with repricing and acceptance | M05; M04 | M05, M04 | M04⟨amendment/repricing⟩; M05⟨amendments⟩ (F-id per docs/01) | 2 | SCR-GST-booking-manage; `PATCH …/reservations/{id}` | Price change → explicit acceptance | AC-M05 per docs/01; G19 | self-service modification rate |
| **Q-GST-5** Can I pay? | guest | Pay via intent or link | M28 | M28 | M28.F28.1.SF28.1.1; M28.F28.1.SF28.1.4 | 2 | SCR-GST-pay | Declined → alternative method | AC-SF28.1.5; G06 | payment success rate |
| **Q-GST-6** Can I check in? | guest | Complete pre-arrival; arrive | M55; M05 | M55, M05 | M55.F55.1.SF55.1.2 | 2–3 | SCR-GST-pre-arrival | Room not ready → ETA notification | AC-SF55.1.2; G19 | pre-arrival completion |
| **Q-GST-7** Can I request help? | guest | Submit service request; see status | M18 `service_request`; M55 | M55, M18 | M55.F55.1.SF55.1.4 | 2–3 | SCR-GST-requests | SLA breach → escalation | AC-SF55.1.4; G19 | request resolution time |
| **Q-GST-8** Can I get a receipt? | guest | Download receipt/tax invoice | M08; M38 | M08, M38 | M55.F55.1.SF55.1.5; M38.F38.1.SF38.1.4 | 2 | SCR-GST-receipts | Invoice format unverified → provisional receipt label | AC-SF55.1.5 | receipt requests to staff |
| **Q-GST-9** Can I claim points? | guest | View history; claim missing points | M30 | M30 | M30.F30.1.SF30.1.3; M30.F30.2.SF30.2.4 | 5 | SCR-GST-points | Missing points → dispute | AC-SF30.2.4; G07 | points disputes |
| **Q-GST-10** Can I reach a human? | guest | Request handoff | M40; M55 | M40, M55 | M40.F40.2.SF40.2.3; M55.F55.1.SF55.1.1 | 3 | SCR-GST-chat | After hours → callback promise | AC-SF40.2.3; G13 | handoff wait time |
| **Q-GST-11** For an event, what is confirmed versus proposed and who pays? | event_organizer | View itinerary status and billing routing | M12 `beo_version`, `change_order`; M08 routing | M12, M11, M08 | M12⟨BEO revision, master billing⟩ (F-id per docs/01) | 3 | SCR-CORP-itinerary | Pending change order → labelled "proposed" | AC-M12 per docs/01; G01, G02 | disputed event invoices |

### 9.12 Technology / compliance

| Owner question | Actor | Needed action | Source-of-truth | Module | Subfeature | Phase | Screen/API | Exception | Acceptance test | KPI |
|---|---|---|---|---|---|---|---|---|---|---|
| **Q-TEC-1** Which integrations, connected devices, certificates, rule packs and rate mappings are healthy? | it_admin, integration_admin | Read health board | M64 registry; M33 monitoring; M07 mappings; M44 | M64, M33, M07, M44 | M64.F64.1.SF64.1.1; M64.F64.1.SF64.1.3; M44.F44.2.SF44.2.8 | 2–6 | SCR-ADM-integration-health | Certificate expiring → alert | AC-SF64.1.3 | connector uptime; expiring certificates |
| **Q-TEC-2** Where are outages, failed callbacks, missing consent, expiring supplier permits or unverified tax rules? | integration_admin, compliance_officer | Work exception queues | M33 event replay; M46; M44; M02 | M64, M46, M44, M65, M02 | M46.F46.1.SF46.1.6; M44.F44.2.SF44.2.4; M65.F65.1.SF65.1.5 | 2–6 | SCR-ADM-dead-letters | Dead letter → replay with dedup | AC-SF65.1.5; G20 | dead letters aging |
| **Q-TEC-3** Can we restore to a test environment and prove recovery? | it_admin | Run restore test; record measured recovery | M64; M01 | M64, M01 | M64.F64.2.SF64.2.1; M64.F64.2.SF64.2.4 | 2–6 | SCR-ADM-restore-tests | Restore fails → incident | AC-SF64.2.4 | measured RTO/RPO |
| **Q-TEC-4** Can we delete data by purpose without deleting required accounting and safety evidence? | dpo | Purpose-based deletion with legal hold | M65; M02 | M65, M02 | M65.F65.2.SF65.2.6; M02⟨retention, legal hold⟩ (F-id per docs/01) | 2–4 | SCR-CMP-retention | Conflicting retention → keep minimal lawful record, redact rest | AC-SF65.2.6 | deletion SLA |
| **Q-TEC-5** (discovered) Which API versions, webhooks and partner certifications are current? | integration_admin | Review developer-platform register | M33 | M33 | M33⟨scopes/versioning/deprecation, certification kit⟩ (F-id per docs/01) | 2–7 | SCR-ADM-api-versions | Deprecated version in use → notice to partner | AC-M33 per docs/01 | partners on deprecated versions |
| **Q-TEC-6** (discovered) Which Later-phase connected services (UC, HSIA, IPTV, marketplace) are disabled and why? | it_admin, property_admin | Review feature-flag register | M01 flags | M01, M34, M35, M36, M37 | M01⟨feature flags⟩ (F-id per docs/01); M34–M37 are Later | 7–8 (Later) | SCR-ADM-feature-flags | Enable without certification → blocked | AC-M01 per docs/01 | n/a (status question) |

**Row count:** 86 rows = 76 Section O questions (after splitting 2 compound questions into 4 and 7 rows) + 10 discovered questions.

---

## 10. Cross-cut table: role × operating condition

Required behavior for each role under the eight Section O conditions. Every cell is a design rule the role's workflows must honour; `docs/12` gives the continuity detail and `docs/11` the guest journey detail.

| Role | Normal | Busy / high-season | Low staffing | Outage | Guest dispute | Refund / cancellation | Accessible / assisted path | Audit / regulator request |
|---|---|---|---|---|---|---|---|---|
| Owner / asset manager | Daily flash with estimate vs reconciled labels | Pace vs budget and oversell exposure surface first | Labor cost vs roster variance shown (SF62.2.2) | Report banner "data stale since t"; no silent zeros | Dispute cost visible in recovery report | Refund/cancel impact on revenue and commission reversal shown | Screen-reader accessible reports; exports in CSV | Point-in-time report version (SF65.2.2) with lineage export |
| GM / duty manager | 3–7 prioritized tasks | Arrivals/departures sorted by risk; overbooking walk plan pre-staged | Coverage gaps and callout status on top (M47/M62) | Continuity runbook launcher; degraded-service list (SF64.1.3) | Case owner + compensation approval within cap matrix (`docs/11` §9) | Approve above-threshold refunds; SoD enforced | Accessibility requests owned with SLA (SF55.2.6) | Incident chronology and approval audit export |
| Revenue / sales | Approve recommendations within guardrails | Tighter guardrails; closed-to-arrival only with approval; no auto-publish | Recommendations queue can defer; no auto-apply | Rate publish paused; channel stop-sell if ARI ack unknown | Price-discrepancy queue (SF53.2.5) | Cancellation pace feeds forecast (SF53.1.2) | Corporate RFQ capture by phone/email into same record | Rate change log with approver, version and rollback |
| Marketing / guest relations | Consent-filtered campaigns; review queue | Frequency caps enforced; complaint SLA shortened by config | Automated survey continues; responses queue with SLA | Campaign sends paused; website read-only banner if booking engine down | Public response approval; no retaliation (SF52.2.3) | Voucher/compensation ledger entries | Alt-text/captions required before publish (SF39.1.3) | Consent evidence per contact; campaign audit |
| Front desk / concierge | Readiness card check-in | Pre-arrival completion pushed; express checkout | Assisted kiosk-free queue triage; tasks auto-routed | Offline staff app + paper registration pack; standalone terminal | Escalate to duty manager; folio adjustment by approval | Refund via original tender only; policy snapshot governs | Assisted check-in path, non-biometric ID, large-print receipts | Registration record, ID-retention proof, travel consent log |
| Housekeeping / laundry | Board by priority | Early-arrival priority; reclean SLA | Capacity-based reassignment; supervisor inspects sample | Paper room-status sheet; sync conflict queue | Minibar/damage charge dispute workflow (SF56.2.6) | Reversal of minibar/linen charges posts once | Accessible-room prep checklist (grab bars, etc. per room attributes) | Linen custody and inspection history |
| Chef / F&B / club | Production plan with allergens | Batch production; capacity gates | Backup chef/emergency callout (M47) | Paper KOT and allergen sheets; POS offline queue | Allergen/food complaint → food-safety incident link (SF57.2.6) | Void/refund approvals; waste logged | Dietary needs acknowledged by kitchen (SF57.1.4) | Lot trace, temperature logs, HACCP-style records per rule pack |
| Procurement / storekeeper / vendor | RFQ with minimum quotes | Emergency purchase with retrospective approval (SF21.1.6) | Low-risk straight-through GRN with attestation (SF50.2.5) | Paper GRN + later scan; no stock posting until verified | Vendor claim / dispute (SF49.2.8) | PO cancel, return-to-vendor, credit memo | Vendor web fallback for app-less suppliers | Bid/award/PO evidence; sample purge proof |
| Maintenance / security | SLA-based work orders | Prioritize guest-impacting faults; rooms OOO only with approval | Contractor call-off from approved vendors | Manual gate/key procedures; life-safety systems independent (SF42.1.5) | Guest-damage vs wear evidence | Vendor warranty/credit on failed repair | Accessible-room faults priority | Incident chronology; gate override log; claim packet |
| Finance / HR | Daily reconciliation | Higher-volume night audit with exception-first view | Maker-checker remains mandatory; no self-approval | Payments pending, no blind retry; manual receipts numbered | Chargeback evidence (SF20.3.5) | Refund once, reversal entries, points/commission reversal | Payslips accessible; employee self-service | Filing receipts, GL audit trail, salary access log |
| Guest / corporate buyer | Self-service booking and stay | Clear availability; waitlist; no hidden fees | Response-time notice in messaging | Assisted phone/desk booking; confirmation later by email | Reach a human; case reference | Policy snapshot shown; refund status visible | WCAG 2.2 AA, low-bandwidth, assisted channel | Access/export of own data on request |
| Technology / compliance | Health board green | Capacity scaling (SaaS) / load alerts (on-prem) | On-call rota and vendor support contacts | Incident command; status page (SF64.2.6) | Evidence preservation for disputes | n/a (supports finance) | Accessibility test evidence | Rule-pack evidence, DPIA, restore-test reports |

---

## 11. Module coverage check (M01–M68)

Generated from the Module column of §9 and the benchmark sections. Every module has at least one owner-question row; benchmark column shows where the module is compared or excluded ("—" = no direct competitor claim observed; module is still in scope).

| Module | Name | Question rows (§9) | Benchmark section |
|---|---|---|---|
| M01 | Platform/deployment | Q-TEC-3, Q-TEC-6 | §5 (flags) |
| M02 | IAM and consent | Q-MKT-3, Q-FIN-3, Q-TEC-2, Q-TEC-4 | 3.6 |
| M03 | Room inventory | Q-GM-1, Q-GM-3, Q-MKT-1, Q-FD-3, Q-MNT-1, Q-GST-1 | 3.1 |
| M04 | Rates and quote engine | Q-REV-1, Q-FD-2, Q-GST-1, Q-GST-2, Q-GST-4 | 3.1 |
| M05 | Reservations/stays | Q-GM-1, Q-FD-1, Q-FD-3, Q-GST-4, Q-GST-6 | 3.1 (via M03/M04) |
| M06 | Front desk/housekeeping | Q-GM-3, Q-FD-1, Q-HK-1 | 3.3 |
| M07 | Distribution | Q-REV-3, Q-TEC-1 | 3.1 |
| M08 | Folio/cashiering | Q-FD-7, Q-HK-5, Q-FIN-1, Q-FIN-4, Q-GST-8, Q-GST-11 | 3.4 |
| M09 | Timed facility inventory | Q-GM-3, Q-REV-5, Q-GST-2 | 3.4 |
| M10 | Corporate accounts | Q-REV-5 | 3.4 |
| M11 | Corporate portal/apps | Q-GST-2, Q-GST-11 | 3.4 |
| M12 | Group/events/MICE | Q-OWN-2, Q-REV-5, Q-REV-6, Q-FNB-1, Q-GST-11 | 3.4 |
| M13 | Bar/F&B POS | Q-FNB-4, Q-FNB-6, Q-FIN-1 | — |
| M14 | Bar/kitchen inventory | Q-HK-3, Q-FNB-3, Q-FNB-4, Q-PRC-10 | 3.5 |
| M15 | Club/membership | Q-GM-7 | §4 |
| M16 | Catering | Q-REV-5, Q-FNB-1 | 3.4 |
| M17 | Parking/ANPR | Q-FD-7, Q-MNT-4 | 3.4 |
| M18 | Guest engagement | Q-GST-7 | 3.3 |
| M19 | GL and cost centers | Q-OWN-1, Q-OWN-3, Q-FIN-1, Q-FIN-2 | 3.5 |
| M20 | AP/AR and treasury | Q-OWN-1, Q-OWN-3, Q-OWN-6, Q-REV-5, Q-MKT-4, Q-PRC-9, Q-FIN-1 | 3.5 |
| M21 | Procurement | Q-PRC-1 | 3.5 |
| M22 | Electricity | Q-OWN-1, Q-OWN-8, Q-FIN-8 | 3.5 |
| M23 | Water | Q-OWN-8, Q-FIN-8 | 3.5 |
| M24 | Pipeline gas | Q-FIN-8 | 3.5 |
| M25 | Gas cylinders | Q-FIN-8 | 3.5 |
| M26 | Maintenance/assets/vendors | Q-OWN-1, Q-GM-1, Q-GM-3, Q-MNT-1, Q-MNT-2 | — |
| M27 | HR/workforce/payroll | Q-OWN-1, Q-GM-2, Q-FIN-3, Q-FIN-6 | 3.5 |
| M28 | Payment orchestration | Q-OWN-1, Q-OWN-3, Q-FD-1, Q-FD-4, Q-FIN-1, Q-FIN-6, Q-GST-5 | 3.3 |
| M29 | Bill-provider gateway | Q-FIN-6, Q-FIN-8 | 3.5 |
| M30 | Loyalty points wallet | Q-FIN-7, Q-GST-9 | 3.6 |
| M31 | Referrals/growth | Q-FIN-7 | 3.6 |
| M32 | Management BI | Q-OWN-1, Q-OWN-2, Q-OWN-4, Q-OWN-5, Q-REV-6 | 3.2, 3.5 |
| M33 | Integration developer platform | Q-TEC-1, Q-TEC-5 | §5 |
| M34 | UC/wake-up | Q-TEC-6 | §5 |
| M35 | HSIA | Q-TEC-6 | §5 |
| M36 | IPTV | Q-TEC-6 | §5 |
| M37 | Marketplace/AI | Q-TEC-6 | §5 |
| M38 | Jurisdiction, tax and government exchange | Q-OWN-6, Q-FD-2, Q-FIN-5, Q-GST-8 | 3.5 |
| M39 | Property media and AI enhancement | Q-MKT-1, Q-MKT-2 | 3.1 |
| M40 | Guest AI assistant | Q-MKT-7, Q-GST-10 | 3.3 |
| M41 | Assisted identity, signature and verification | Q-FD-1, Q-FIN-3, Q-GST-3 | 3.3 |
| M42 | Emergency and incident response | Q-GM-1, Q-MNT-3, Q-MNT-5 | §4 |
| M43 | Lost and found | Q-FD-6 | §4 |
| M44 | Five-market jurisdiction classifier | Q-FD-2, Q-FIN-5, Q-TEC-1, Q-TEC-2 | §4 |
| M45 | Travel and mobility concierge | Q-FD-5 | §4 |
| M46 | All-service vendor registration | Q-PRC-2, Q-MNT-2, Q-TEC-2 | §4 |
| M47 | Kitchen continuity and emergency chefs | Q-GM-2, Q-FNB-2 | 3.4 |
| M48 | Vendor mobile catalog | Q-PRC-2 | 3.5 |
| M49 | RFQ-to-award and PO engine | Q-PRC-1, Q-PRC-3, Q-PRC-4, Q-PRC-5, Q-PRC-6, Q-PRC-9 | 3.5 |
| M50 | Fulfillment, follow-up and low-touch stores | Q-FNB-3, Q-FNB-4, Q-FNB-5, Q-FNB-6, Q-PRC-7, Q-PRC-8, Q-PRC-10 | 3.5 |
| M51 | Hotel website and acquisition | Q-REV-3, Q-REV-4, Q-MKT-1, Q-MKT-4, Q-GST-1 | 3.1 |
| M52 | Guest CRM, marketing and reputation | Q-MKT-3, Q-MKT-4, Q-MKT-5, Q-MKT-6 | 3.6 |
| M53 | Revenue and demand management | Q-OWN-2, Q-REV-1, Q-REV-2, Q-REV-3, Q-REV-6 | 3.2 |
| M54 | Offers, upsells, vouchers | Q-FD-3 | 3.2 |
| M55 | Guest journey and service recovery | Q-GM-1, Q-MKT-5, Q-MKT-6, Q-FD-4, Q-GST-6, Q-GST-7, Q-GST-10 | 3.3, 3.6 |
| M56 | Housekeeping, laundry, linen, minibar | Q-GM-3, Q-FD-1, Q-HK-1, Q-HK-2, Q-HK-3, Q-HK-4, Q-HK-5 | 3.3 |
| M57 | Restaurant, room service, food assurance | Q-GM-7, Q-FNB-1, Q-FNB-5, Q-FNB-6 | 3.4, 3.5 |
| M58 | Optional amenity businesses | Q-GM-6 | — |
| M59 | Local transport and fleet | Q-FD-5 | 3.4 |
| M60 | Revenue protection | Q-FNB-6, Q-FIN-2, Q-FIN-4 | — |
| M61 | Hygiene, safety and inspections | Q-FNB-5 | 3.5 |
| M62 | Staff enablement | Q-GM-2, Q-MNT-2 | — |
| M63 | Workflow automation | Q-GM-1, Q-GM-4 | — |
| M64 | Technology, devices and resilience | Q-GM-5, Q-FD-4, Q-MNT-4, Q-TEC-1, Q-TEC-2, Q-TEC-3 | 3.3 |
| M65 | Data governance | Q-OWN-1, Q-OWN-5, Q-TEC-2, Q-TEC-4 | 3.5 |
| M66 | Owner, brand and lifecycle | Q-OWN-4, Q-OWN-6, Q-OWN-7 | — |
| M67 | Sustainability | Q-OWN-4, Q-OWN-8 | — |
| M68 | Risk, insurance and continuity | Q-OWN-6, Q-GM-5, Q-MNT-5 | §4 |

**Result:** 68 of 68 modules traced to at least one question row; 0 modules missing. Discovered modules M69+ (if `docs/01` adds any) must be added here at the `docs/13` consistency check.
---

## 12. Proof gaps, open decisions and owners

Decision ids D-701–D-709 are reserved for this file (docs/11 uses D-711–D-719, docs/12 uses D-721–D-739); the `docs/13` decision log is authoritative and may renumber on collision.

| ID | Gap / decision | Why it matters | Owner | Interim assumption | Gate |
|---|---|---|---|---|---|
| D-701 | SiteMinder pages not retrievable directly (bot protection); only search snippets used | Benchmark for distribution relies on secondary evidence | Product Owner | Treat SiteMinder claims as `unverified-assumption` until re-read in a browser | Before any external use of §3 |
| D-702 | Digital key / kiosk demand at pilot hotel | Deferred to Phase 7 in this plan | Product Owner | Staffed + mobile pre-arrival path | Pilot hotel profile (Section J) |
| D-703 | Metasearch partner model | We will not run bidding ourselves | Marketing Manager | Partner-provided | Commercial agreement |
| D-704 | Channel-manager partner selection and certification | Required for OTA/GDS reach in R1 | Revenue Manager + Integration Admin | One certified adapter; manual extranet path documented | M07 certification before go-live |
| D-705 | Cash/stored-value wallet | Licensing | Legal/Payments workstream | Not offered | Legal opinion |
| D-706 | PSP for pilot market | Payment certification | Finance + Integration Admin | Mock PSP adapter in Phases 2–4 | PSP certification |
| D-707 | MetriStay commercial pricing model | Needed for §8 business case | Metrikingdom commercial lead | Not modelled | `docs/00` business case |
| D-708 | Feature ids for M01–M18 and M33–M37 | This file cites `Mnn⟨phrase⟩` | Catalogue editor (`docs/01`) | Reconcile at `docs/13` consistency check | Planning gate |
| D-709 | Pilot-hotel KPI baselines and targets (Section P.6) | KPIs in §9 have no targets | GM of pilot hotel | Measure baseline in first 30 days of pilot | Phase 6 exit |

**Claims discipline:** no row in this document may be quoted as "MetriStay matches/exceeds X" until the cited acceptance tests pass on a working build and the competitor capability has been observed in a real evaluation, not only a marketing page.
