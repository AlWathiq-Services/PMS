# 01 — Module Catalogue (index)

**Pack:** MetriStay Hospitality Suite Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Status:** specification / design target — no code exists. Conventions: `docs/README.md` §3.

The module catalogue is the Section H item 2 deliverable: every module M01–M68 expanded as **module → features → numbered subfeatures**, each subfeature a complete Section-L block (id, name, phase, release, actors, screens, inputs, states, api, events, data, rules, security, failure_cases, finance_report_effect, i18n_a11y, acceptance, dependency). Because of its size it is split into six files:

| File | Modules | Theme |
|---|---|---|
| [`01-catalogue/01-platform-and-pms-core.md`](01-catalogue/01-platform-and-pms-core.md) | M01–M08 | Platform, IAM/consent, room inventory, rates/quotes, reservations, front desk/housekeeping, distribution, folio |
| [`01-catalogue/02-commerce-events-outlets.md`](01-catalogue/02-commerce-events-outlets.md) | M09–M18 | Timed facilities, corporate accounts/apps, MICE, bar/POS, stock ledger, club, catering, parking/ANPR, guest engagement |
| [`01-catalogue/03-finance-procurement-utilities-workforce-payments.md`](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) | M19–M29 | GL, AP/AR/treasury, procurement policy, electricity/water/gas/cylinders, maintenance, HR/payroll/WPS, payments, bill-pay |
| [`01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md`](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) | M30–M44 | Points, single-tier referral, BI, developer platform, UC/HSIA/IPTV/marketplace (Later), tax & government exchange, media, guest AI, ID/e-sign/OTP, incidents, lost & found, jurisdiction classifier |
| [`01-catalogue/05-travel-vendors-chef-sourcing-receiving.md`](01-catalogue/05-travel-vendors-chef-sourcing-receiving.md) | M45–M50 | Travel concierge, vendor registration/search, chef continuity, vendor mobile catalog, RFQ→award→PO, AI follow-up/receiving/stores |
| [`01-catalogue/06-growth-operations-governance.md`](01-catalogue/06-growth-operations-governance.md) | M51–M68 | Website, CRM/reputation, revenue mgmt, offers, guest journey, housekeeping/laundry, food assurance, amenities, transport, revenue protection, hygiene, staff, workflow, devices/resilience, data governance, owner/brand, sustainability, risk/continuity |

Every file opens with a shared-entity glossary and closes each module with key invariants, module acceptance mapped to Section G (`AT-Gnn.m`, defined in `docs/09`) and its open decisions (consolidated in `docs/13a-decision-register.generated.md`). Screen ids in the catalogue resolve to canonical `docs/04` screens through `docs/04a-screen-crosswalk.md`.

## Verification (Section M)

Run `python3 tools/check_catalogue.py`. Result at pack v0.1:

- **68 of 68 modules** represented; **210 features**; **1,150 numbered subfeatures**, every one with all 18 Section-L fields non-empty and a unique id.
- **1,113 R1** (Phases 2–6) and **37 Later** (Phases 7–8) subfeatures.
- All **584 subfeature ids fixed by master-prompt Sections K and Q** are present with their original names; features for M01–M18 and M33–M37 (no Section K/Q detail) were derived from Section C.
- Full per-subfeature list with phase, release and acceptance: `docs/13b-subfeature-index.generated.md`.

## Module summary

| Module | Name | Section C build phase | Features | Subfeatures | Catalogue phases | Release | File |
|---|---|---|---|---|---|---|---|
| M01 | Platform/deployment | 2 | F01.1–F01.6 (6) | 32 | 2, 3 | R1 | [01](01-catalogue/01-platform-and-pms-core.md) |
| M02 | IAM and consent | 2 | F02.1–F02.5 (5) | 27 | 2, 3 | R1 | [01](01-catalogue/01-platform-and-pms-core.md) |
| M03 | Room inventory | 2 | F03.1–F03.5 (5) | 26 | 2, 3, 4 | R1 | [01](01-catalogue/01-platform-and-pms-core.md) |
| M04 | Rates and quote engine | 2 | F04.1–F04.5 (5) | 26 | 2, 3 | R1 | [01](01-catalogue/01-platform-and-pms-core.md) |
| M05 | Reservations/stays | 2–3 | F05.1–F05.5 (5) | 30 | 2, 3 | R1 | [01](01-catalogue/01-platform-and-pms-core.md) |
| M06 | Front desk/housekeeping | 2–3 | F06.1–F06.4 (4) | 23 | 2, 3 | R1 | [01](01-catalogue/01-platform-and-pms-core.md) |
| M07 | Distribution | 2–3 | F07.1–F07.5 (5) | 26 | 2, 3, 4 | R1 | [01](01-catalogue/01-platform-and-pms-core.md) |
| M08 | Folio/cashiering | 2 | F08.1–F08.6 (6) | 32 | 2, 3, 4 | R1 | [01](01-catalogue/01-platform-and-pms-core.md) |
| M09 | Timed facility inventory | 2–3 | F09.1–F09.4 (4) | 21 | 2, 3 | R1 | [02](01-catalogue/02-commerce-events-outlets.md) |
| M10 | Corporate accounts | 3 | F10.1–F10.4 (4) | 19 | 3 | R1 | [02](01-catalogue/02-commerce-events-outlets.md) |
| M11 | Corporate portal/apps | 3 | F11.1–F11.5 (5) | 21 | 3, 5 | R1 | [02](01-catalogue/02-commerce-events-outlets.md) |
| M12 | Group/events/MICE | 3 | F12.1–F12.5 (5) | 21 | 3, 4 | R1 | [02](01-catalogue/02-commerce-events-outlets.md) |
| M13 | Bar/F&B POS | 3 | F13.1–F13.5 (5) | 24 | 3 | R1 | [02](01-catalogue/02-commerce-events-outlets.md) |
| M14 | Bar/kitchen inventory | 3–4 | F14.1–F14.5 (5) | 25 | 3, 4 | R1 | [02](01-catalogue/02-commerce-events-outlets.md) |
| M15 | Club/membership | 3 | F15.1–F15.4 (4) | 16 | 3 | R1 | [02](01-catalogue/02-commerce-events-outlets.md) |
| M16 | Catering | 3–4 | F16.1–F16.5 (5) | 20 | 3, 4 | R1 | [02](01-catalogue/02-commerce-events-outlets.md) |
| M17 | Parking/ANPR | 3 | F17.1–F17.4 (4) | 20 | 3, 5 | R1 | [02](01-catalogue/02-commerce-events-outlets.md) |
| M18 | Guest engagement | 3–5 | F18.1–F18.5 (5) | 18 | 3, 5 | R1 | [02](01-catalogue/02-commerce-events-outlets.md) |
| M19 | GL and cost centers | 4 | F19.1–F19.3 (3) | 16 | 4 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M20 | AP/AR and treasury | 4 | F20.1–F20.5 (5) | 26 | 3, 4 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M21 | Procurement | 3–4 | F21.1–F21.3 (3) | 17 | 3, 4 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M22 | Electricity | 4–5 | F22.1–F22.4 (4) | 18 | 4, 5 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M23 | Water | 4–5 | F23.1–F23.1 (1) | 6 | 4 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M24 | Pipeline gas | 4 | F24.1–F24.2 (2) | 8 | 4 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M25 | Gas cylinders | 4 | F25.1–F25.2 (2) | 8 | 4 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M26 | Maintenance/assets/vendors | 3–4 | F26.1–F26.3 (3) | 17 | 3, 4 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M27 | HR/workforce/payroll | 4 | F27.1–F27.5 (5) | 29 | 3, 4 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M28 | Payment orchestration | 2 and 5 | F28.1–F28.3 (3) | 18 | 2, 3, 4, 5 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M29 | Bill-provider gateway | 5 | F29.1–F29.3 (3) | 19 | 4, 5 | R1 | [03](01-catalogue/03-finance-procurement-utilities-workforce-payments.md) |
| M30 | Loyalty points wallet | 5 | F30.1–F30.2 (2) | 13 | 5 | R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M31 | Referrals/growth | 5 (single-tier build and testing); 6 (market-specific activation gate); 8 (partner distribution expansion) | F31.1–F31.2 (2) | 14 | 5, 6 | R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M32 | Management BI | 2 foundation; 4–6 complete | F32.1–F32.4 (4) | 24 | 2, 3, 4, 5 | R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M33 | Integration developer platform | 2 foundation; 5–7 mature | F33.1–F33.3 (3) | 13 | 2, 5, 6, 7 | Later/R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M34 | UC/wake-up | 7 | F34.1–F34.2 (2) | 7 | 7 | Later | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M35 | HSIA | 7 | F35.1–F35.2 (2) | 7 | 7 | Later | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M36 | IPTV | 7 | F36.1–F36.2 (2) | 7 | 7 | Later | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M37 | Marketplace/AI | 7–8 | F37.1–F37.3 (3) | 10 | 7, 8 | Later | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M38 | Jurisdiction, tax and government exchange | 2 foundation; 4–6 certified workflows | F38.1–F38.3 (3) | 20 | 2, 4, 5 | R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M39 | Property media and AI enhancement | 2–3 | F39.1–F39.2 (2) | 13 | 2, 3 | R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M40 | Guest AI assistant | 3–5 | F40.1–F40.2 (2) | 13 | 3, 5 | R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M41 | Assisted identity, signature and verification | 2–3; 5 messaging adapters | F41.1–F41.2 (2) | 15 | 2, 3, 5 | R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M42 | Emergency and incident response | 3–4 | F42.1–F42.2 (2) | 12 | 3, 4 | R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M43 | Lost and found | 2–3 | F43.1–F43.1 (1) | 8 | 2, 3 | R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M44 | Five-market jurisdiction classifier | 2 foundation; 4–6 rules/verification | F44.1–F44.2 (2) | 14 | 2, 4, 5 | R1 | [04](01-catalogue/04-loyalty-bi-platform-jurisdiction-guest-safety.md) |
| M45 | Hotel-initiated travel and mobility concierge | 3 request/case and taxi; 5 contracted provider exchange; 6 pilot certification | F45.1–F45.3 (3) | 23 | 3, 5 | R1 | [05](01-catalogue/05-travel-vendors-chef-sourcing-receiving.md) |
| M46 | All-service vendor registration and department discovery | 2 identity/taxonomy; 3 portal/search; 4 procurement integration; 5 travel-provider adapters | F46.1–F46.3 (3) | 20 | 2, 3, 4 | R1 | [05](01-catalogue/05-travel-vendors-chef-sourcing-receiving.md) |
| M47 | Kitchen continuity and emergency chefs | 2 roster basis; 3 kitchen coverage; 4 payroll/procurement integration | F47.1–F47.2 (2) | 14 | 2, 3, 4 | R1 | [05](01-catalogue/05-travel-vendors-chef-sourcing-receiving.md) |
| M48 | Vendor mobile catalog, prices and availability | 3 apps/catalog; 4 ERP and stock integrations | F48.1–F48.3 (3) | 20 | 3 | R1 | [05](01-catalogue/05-travel-vendors-chef-sourcing-receiving.md) |
| M49 | RFQ-to-award and purchase order engine | 3 requisition/RFQ/PO; 4 accounting and audit | F49.1–F49.3 (3) | 24 | 3, 4 | R1 | [05](01-catalogue/05-travel-vendors-chef-sourcing-receiving.md) |
| M50 | Fulfillment, automated follow-up and low-touch stores | 3 dispatch and scan pilots; 4 ledgers and finance; 6 site acceptance | F50.1–F50.4 (4) | 31 | 3, 4 | R1 | [05](01-catalogue/05-travel-vendors-chef-sourcing-receiving.md) |
| M51 | Hotel website and customer acquisition | 2 booking website; 3–5 marketing/channel integrations | F51.1–F51.2 (2) | 14 | 2, 3 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M52 | Guest CRM, marketing and reputation | 3 profile/messaging/reviews; 5 advanced campaigns | F52.1–F52.2 (2) | 14 | 3, 4, 5 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M53 | Revenue and demand management | 3 baseline; 5 recommendations; 7 advanced automation | F53.1–F53.2 (2) | 13 | 3, 5, 7 | Later/R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M54 | Offers, upsells, vouchers and packages | 3 core ancillary; 4–5 accounting/optimization | F54.1–F54.2 (2) | 12 | 3, 4, 5 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M55 | Guest journey and service recovery | 2–3 guest desk; 4 service recovery; 7 key/kiosk hardware | F55.1–F55.3 (3) | 15 | 2, 3, 4, 7 | Later/R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M56 | Housekeeping, laundry, linen and minibar | 2 room board; 3–4 linen/minibar | F56.1–F56.2 (2) | 12 | 2, 3, 4 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M57 | Restaurant, room service and food assurance | 3 ordering; 4 food controls | F57.1–F57.2 (2) | 13 | 3, 4 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M58 | Optional amenity businesses | 3 simple timed bookings; 4–7 amenity-specific activation | F58.1–F58.2 (2) | 11 | 3, 4 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M59 | Local transport, fleet and dispatch | 3 concierge; 4 fleet where operated | F59.1–F59.2 (2) | 11 | 3, 4 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M60 | Revenue protection and control | 2 core cash/night audit; 4–5 controls | F60.1–F60.2 (2) | 12 | 2, 3, 4, 5 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M61 | Hygiene, safety and inspections | 3 food/room checks; 4 risk workflows | F61.1–F61.2 (2) | 11 | 3, 4 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M62 | Staff enablement and quality | 3 shift/SOP; 4 learning/workforce | F62.1–F62.2 (2) | 11 | 3, 4, 5 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M63 | Workflow automation and service desk | 2 engine; 3–6 templates | F63.1–F63.2 (2) | 11 | 2, 3, 4 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M64 | Technology, site devices and resilience | 2 foundation; 3–6 tested profiles; 7 optional UC/HSIA/IPTV | F64.1–F64.2 (2) | 14 | 2, 3, 4, 6, 7 | Later/R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M65 | Data governance and decision intelligence | 2 definitions; 4–6 governed analytics | F65.1–F65.2 (2) | 12 | 2, 4, 5 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M66 | Owner, brand and property lifecycle | 4–6 single-property owner; 7 portfolio | F66.1–F66.2 (2) | 11 | 4, 5, 6, 7 | Later/R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M67 | Sustainability and resource performance | 4 utilities/waste; 5–6 dashboards | F67.1–F67.2 (2) | 11 | 4, 5, 6 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |
| M68 | Enterprise risk, insurance and continuity | 4 incident linkage; 6 operational drills | F68.1–F68.2 (2) | 11 | 4, 5, 6 | R1 | [06](01-catalogue/06-growth-operations-governance.md) |

Subfeature phases later than the Section C build phase are extensions ("later extensions may continue", Section C). Three modules schedule a few subfeatures **earlier** than Section C; these are deliberate and recorded in `docs/13` §4.2.

## Discovered modules

No additional top-level module was required: every discovered need attached to an existing module as an added feature or subfeature, marked `(added: reason)` or `# ADDED` in the catalogue. Examples: F19.3 period close/allocation, F20.4–F20.5 budgets and payable backbone, F22.3 cross-utility guard, F28.3 tokenization/fraud, F29.3 manual/bank path, F55.3 digital key/kiosk (Phase 7), SF45.2.9–10, SF47.2.8, SF48.3.8, SF49.3.8. The blueprints (D-001) may still add modules M69+.
