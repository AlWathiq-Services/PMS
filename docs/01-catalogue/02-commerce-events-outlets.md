# 01-catalogue / 02 — Commerce, events and outlets (M09–M18)

**Pack:** MetriStay Hospitality Suite — Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Scope:** M09 Timed facility inventory, M10 Corporate accounts, M11 Corporate portal/apps, M12 Group/events/MICE, M13 Bar/F&B POS, M14 Bar/kitchen inventory, M15 Club/membership, M16 Catering, M17 Parking/ANPR, M18 Guest engagement.
**Governing source:** master prompt v3.0, Section C rows M09–M18, with Sections D, F, G, O and P. Conventions are in `docs/README.md` §3 (IDs, actors, Section-L schema, API/event style, honesty labels, money/time/quantity).

> This file is a specification. Nothing in it claims that software, a device integration, an app-store listing, a partner contract or a certification exists. Every device and partner capability carries an honesty label (`docs/README.md` §3.6). No camera, LPR server, gate controller, POS terminal, KDS or printer vendor is presented as certified.

## 0. How to read this file

### 0.1 Subfeature notation
Each subfeature is one Section-L YAML block. For density, blocks use YAML flow sequences; every field is present and none is empty (`none` is written when a field does not apply). `phase` is the first implementation phase; `release: R1` means Customer Release 1 (Phases 2–6). `api` names the primary endpoint; related endpoints are listed after `;`. Every mutating call that moves money, stock, inventory, capacity or an external order requires an `Idempotency-Key` header (README §3.5), so the header is not repeated in each block.

### 0.2 Screen-ID prefixes used here
| Prefix | Application (Section F) |
|---|---|
| `SCR-ADM-` | Property admin/configuration (web back office) |
| `SCR-SALES-` | Sales and catering office (web) |
| `SCR-EVT-` | Events/banquets operations (web + staff mobile) |
| `SCR-CORP-` | MetriStay Business corporate web portal |
| `SCR-CAPP-` | MetriStay Business signed Android/iOS app |
| `SCR-FD-` | Front desk (web) |
| `SCR-POS-` / `SCR-KDS-` | Outlet POS (tablet/terminal) / kitchen and bar display |
| `SCR-INV-` | Stores, stock and recipes (web + staff mobile) |
| `SCR-CLUB-` | Club host and door (tablet/staff mobile) |
| `SCR-CAT-` | Catering production and dispatch (web + staff mobile) |
| `SCR-PARK-` | Parking/security console and lane (web + staff mobile) |
| `SCR-GST-` | Guest web/app (MetriStay guest) |
| `SCR-STF-` | Staff mobile app (offline-capable) |
| `SCR-FIN-` / `SCR-OPS-` / `SCR-MGT-` | Finance, operations exception queue, management reporting |

### 0.3 Canonical entities defined in this file (other writers MUST reuse these names)
| Entity | Owner module / bounded context | Meaning |
|---|---|---|
| `timed_resource` | M09 `facility-scheduling` | Anything sold or reserved by time interval: meeting/banquet room, partition, restaurant/club table or zone, day-use room slot pool, parking pool, kitchen production slot, equipment pool, labor pool. |
| `resource_link` | M09 | Parent/child and conflict edges between resources (combined hall ↔ partitions; table ↔ zone). |
| `space_layout` | M09 | Layout type per space with seated/standing capacity, accessibility capacity and setup/teardown minutes. |
| `resource_allocation` | M09 | One interval × quantity claim on a `timed_resource` (states `held`, `confirmed`, `released`, `consumed`). The only thing counted by the non-double-sale invariant. |
| `composite_hold` / `composite_hold_component` | M09 | Atomic, all-or-nothing hold across room-type nights (M03 `inventory_hold`), spaces, equipment, labor, kitchen slots, club capacity and parking capacity. |
| `corporate_account`, `corporate_site`, `corporate_contact`, `cost_center_ref`, `corporate_agreement`, `agreement_version`, `rate_eligibility_rule`, `corporate_credit_line`, `customer_po`, `approval_policy`, `approval_request`, `corporate_budget`, `sales_opportunity`, `corporate_rfq`, `proposal`, `contract_version` | M10 `corporate-commercial` | Corporate customer master, negotiated terms, credit and sales pipeline. `customer_po` is the corporate buyer's PO and is distinct from M49 `purchase_order` (hotel's own purchasing). |
| `app_device_registration`, `app_release` | M11 `corporate-channel` | Signed app build/release register and device enrolment. |
| `room_block`, `room_block_night`, `block_pickup`, `rooming_list`, `event_booking`, `function_booking`, `beo`, `beo_version`, `beo_line`, `change_order`, `attendee`, `master_account`, `routing_rule_ref`, `event_reconciliation` | M12 `events-mice` | Groups/MICE. `master_account` is a folio-type account owned by M08 but opened and routed by M12. |
| `menu`, `menu_item`, `modifier_group`, `price_level`, `pos_check`, `pos_check_line`, `pos_tender`, `pos_shift`, `pos_offline_batch`, `kitchen_ticket`, `pos_adjustment` | M13 `outlet-pos` | Outlet sales. |
| `stock_item`, `uom`, `uom_conversion`, `store`, `store_bin`, `stock_lot`, `stock_ledger_entry`, `stock_balance` (projection), `recipe`, `recipe_line`, `stock_count`, `stock_count_line`, `recall_case`, `waste_record` | M14 `stock` (shared canonical ledger with M50) | **Canonical stock model for the whole suite.** M50 receiving/issue/return/waste, M56 linen/minibar, M25 cylinders and M16 catering post into `stock_ledger_entry`; none keeps a second quantity source of truth. |
| `club_venue`, `club_zone`, `membership_plan`, `membership`, `club_pass`, `access_event`, `club_session`, `minimum_spend_commitment` | M15 `club` | Club/membership. |
| `catering_package`, `catering_order`, `cover_guarantee`, `production_plan`, `production_batch`, `ingredient_reservation`, `dispatch_manifest`, `catering_actual` | M16 `catering` | Catering production and off-site logistics. |
| `parking_facility`, `parking_zone`, `parking_lane`, `parking_permit`, `plate_registration`, `lpr_device`, `plate_observation`, `observation_review`, `gate_decision`, `gate_command`, `parking_session`, `parking_tariff` | M17 `parking` | Parking and ANPR. |
| `guest_app_session`, `service_request_type`, `service_request`, `message_thread`, `message`, `feedback_response` | M18 `guest-engagement` | Guest-facing engagement channel. |

**Referenced, not owned here** (names assumed from sibling catalogue files; reconcile at the consistency check in `docs/13`): M02 `consent_record`, `identity`, `role_scope`; M03 `room_type_night_inventory`, `inventory_hold`; M04 `rate_plan`, `quote`, `policy_snapshot`; M05 `reservation`, `stay`, `guest_profile`; M08 `folio`, `folio_window`, `folio_charge`, `invoice`, `credit_note`; M19 `journal_entry`; M20 `ar_account`; M21/M49 `purchase_requisition`, `purchase_order`; M26 `work_order`; M28 `payment_intent`; M30 `points_ledger_entry`; M44 `rule_pack`; M47 `chef_assignment`; M50 `goods_receipt`, `asn`; M52 `guest_contact`; M55 `guest_case`; M63 `task`.

### 0.4 Suite-level invariants owned by this file
- **INV-CAP-1 (non-double-sale):** for every `timed_resource` r and every instant t, Σ quantity of `resource_allocation` in state `held|confirmed` overlapping t (expanded by setup/teardown buffers, and propagated along `resource_link` conflict edges) ≤ available capacity(r, t). Enforced in PostgreSQL by a GiST exclusion constraint (unit resources) or a serialized capacity counter row per (resource, slot) with `SELECT … FOR UPDATE` (quantity pools). Room-type nights remain enforced by M03; a composite hold takes both locks in a fixed global order (M03 → M09 by resource_id → M17 → M15) inside one transaction.
- **INV-STK-1:** `stock_ledger_entry` is append-only; balances are projections; a correction is a reversing entry with `reverses_entry_id`.
- **INV-STK-2:** quantity with `stock_status in (quarantine, waste, recalled)` never increases `available` for sale or issue except through an approved `quarantine_release` of **intact, never-issued, in-date** stock; any lot recorded as discarded, served, plated, returned-from-service or temperature-breached is terminal (`waste`) and cannot be released. "No discarded food returns to sellable stock."
- **INV-FOL-1:** a POS check, parking session, club charge or event line posts to a folio/master account at most once; the posting carries `source_type + source_id` as a unique key in M08.

---

## M09 — Timed facility inventory

| Item | Value |
|---|---|
| Purpose | One capacity engine for everything sold by time interval (spaces, partitions, tables, day use, parking pools, kitchen production slots, equipment, labor) and for atomic composite holds that span rooms, spaces, catering, club and parking without double sale. |
| Build phases | 2 (registry, single-resource allocation, invariant) • 3 (composite holds, diary, layouts, partitions, parking/kitchen/club pools) |
| Release | R1 |
| Bounded context | `facility-scheduling` (schema `fac`) |
| Systems of record owned | `timed_resource`, `resource_link`, `space_layout`, `resource_calendar`, `resource_allocation`, `composite_hold`, `composite_hold_component`, `changeover_rule` |
| Upstream | M01 (property, business date, time zone), M02 (scopes), M03 (room-type-night holds for composite), M26 (out-of-service windows), M44 (occupancy/licensing rule pack references) |
| Downstream | M04 (priced space/equipment), M11 (search), M12 (function diary, events), M13/M57 (tables), M15 (club capacity), M16 (kitchen slots), M17 (parking pools), M54/M58 (amenity slots), M32 (space utilization) |

### F09.1 Resource registry and capacity model

```yaml
- id: M09.F09.1.SF09.1.1
  name: Register timed resources
  phase: 2
  release: R1
  actors: [property_admin, sales_manager, fnb_manager]
  screens: [SCR-ADM-resource-list, SCR-ADM-resource-detail]
  inputs: [resource_type, name_en, name_ar, outlet_or_department, capacity_mode, unit_capacity, pool_quantity, bookable_from, bookable_to, sell_channels, accessibility_features, photo_media_ids]
  states: [draft, active, suspended, retired]
  api: POST /v1/properties/{pid}/timed-resources ; PATCH /v1/properties/{pid}/timed-resources/{rid}
  events: [TimedResourceCreated, TimedResourceChanged, TimedResourceRetired]
  data: [timed_resource, resource_calendar]
  rules: ["resource_type in meeting_room|banquet_hall|partition|restaurant_table|club_zone|club_table|day_use_pool|parking_pool|kitchen_production_slot|equipment_pool|labor_pool", "capacity_mode unit (one booking at a time) or quantity (pool)", "retire only when no future held/confirmed allocation exists", "every resource belongs to exactly one property and one owning department"]
  security: property_admin scope; field changes audited with before/after; no guest data
  failure_cases: ["duplicate name in property -> 409", "retire with future allocations -> 422 with list", "capacity reduced below confirmed future use -> blocked, conflict report"]
  finance_report_effect: Resource department and revenue-center mapping feed M19 mapping and M32 space utilization denominators.
  i18n_a11y: EN/AR names mandatory; RTL form; step-free/hearing-loop/accessible-toilet flags exposed to guest search.
  acceptance: "Creating a hall, 3 partitions, 12 AV kits pool and 20-space parking pool succeeds; reducing pool to 5 with 10 confirmed future passes is rejected listing the affected bookings."
  dependency: M01 property/department model; M39 media for photos.

- id: M09.F09.1.SF09.1.2
  name: Partitions and combinable spaces
  phase: 3
  release: R1
  actors: [property_admin, sales_manager]
  screens: [SCR-ADM-space-combination-editor]
  inputs: [parent_resource_id, child_resource_ids, combination_name, combined_layout_capacities]
  states: [draft, active, retired]
  api: PUT /v1/properties/{pid}/timed-resources/{rid}/links
  events: [ResourceLinkChanged]
  data: [resource_link, timed_resource]
  rules: ["booking a combined space allocates all constituent partitions", "booking any partition blocks the combined space for the overlapping interval", "links form a DAG; cycles rejected", "link change cannot invalidate confirmed allocations"]
  security: property_admin only; audited
  failure_cases: ["cycle -> 422", "link edit conflicting with confirmed booking -> blocked with conflict list"]
  finance_report_effect: Utilization reports attribute combined bookings to constituent partitions pro rata by area.
  i18n_a11y: Visual editor also offers a list/table mode for keyboard and screen-reader users.
  acceptance: "Ballroom = A+B+C. With A confirmed 09:00-12:00, search for Ballroom 10:00-11:00 returns unavailable; B remains bookable."
  dependency: SF09.1.1.

- id: M09.F09.1.SF09.1.3
  name: Layout capacities
  phase: 3
  release: R1
  actors: [property_admin, sales_manager]
  screens: [SCR-ADM-space-layouts]
  inputs: [resource_id, layout_type, max_persons, wheelchair_positions, stage_or_dancefloor_reduction, setup_minutes, teardown_minutes, fire_occupancy_limit, evidence_document_id]
  states: [draft, active, retired]
  api: PUT /v1/properties/{pid}/timed-resources/{rid}/layouts/{layout_type}
  events: [SpaceLayoutChanged]
  data: [space_layout]
  rules: ["layout_type in theatre|classroom|banquet|cabaret|u_shape|boardroom|reception|hollow_square|custom", "max_persons <= fire_occupancy_limit", "fire_occupancy_limit requires evidence document or is labelled unverified-assumption and blocks sale above 50 percent of stated capacity until verified (D-203)", "feasibility uses layout capacity not room area"]
  security: property_admin; evidence document retained per M44 rule pack
  failure_cases: ["layout without capacity -> not searchable", "occupancy limit missing -> unverified badge and cap"]
  finance_report_effect: Capacity denominators for space yield (revenue per available space-hour) in M32.
  i18n_a11y: Layout icons have text labels; capacities read as numbers not only diagrams.
  acceptance: "Hall A classroom 80 with setup 90 min is feasible for 80 attendees; 81 attendees classroom returns no Hall A option."
  dependency: M44 local occupancy rule pack reference; D-203.

- id: M09.F09.1.SF09.1.4
  name: Setup, teardown and changeover buffers
  phase: 3
  release: R1
  actors: [property_admin, sales_manager, housekeeping_supervisor]
  screens: [SCR-ADM-changeover-matrix]
  inputs: [resource_id, from_layout, to_layout, changeover_minutes, labor_pool_id, labor_units]
  states: [active, retired]
  api: PUT /v1/properties/{pid}/changeover-rules
  events: [ChangeoverRuleChanged]
  data: [changeover_rule, space_layout]
  rules: ["allocation interval = [start - setup, end + teardown] and includes changeover from previous layout", "setup/teardown labor claims the labor pool for the buffer interval", "buffers can be shortened only by sales_manager with reason, never below configured minimum"]
  security: override audited with actor/reason
  failure_cases: ["back-to-back bookings violating changeover -> infeasible", "labor pool exhausted -> infeasible with reason"]
  finance_report_effect: Setup labor hours attributed to event labor cost in M12 event P&L.
  i18n_a11y: Times shown in property time zone with 24h/12h per locale.
  acceptance: "Classroom->banquet changeover 120 min: a banquet starting 60 min after a classroom session ends is rejected with reason changeover."
  dependency: SF09.1.3; M27 labor pool linkage optional.

- id: M09.F09.1.SF09.1.5
  name: Operating calendars, blackouts and out-of-service
  phase: 2
  release: R1
  actors: [property_admin, chief_engineer, fnb_manager]
  screens: [SCR-ADM-resource-calendar]
  inputs: [resource_id, opening_hours, blackout_ranges, oos_range, oos_reason, work_order_id]
  states: [open, closed, out_of_service]
  api: POST /v1/properties/{pid}/timed-resources/{rid}/calendar-blocks
  events: [ResourceOutOfServiceScheduled, ResourceReturnedToService]
  data: [resource_calendar, resource_allocation]
  rules: ["OOS is itself a resource_allocation of type block", "OOS overlapping confirmed bookings is allowed only with explicit displacement list and triggers re-accommodation task (SF09.4.3)", "M26 work order can request OOS; release requires inspection"]
  security: engineering and property_admin; displacement requires duty_manager approval
  failure_cases: ["OOS on booked space without approval -> 403", "work order closed without inspection -> resource stays OOS"]
  finance_report_effect: OOS hours excluded from available space-hours; displacement cost tracked to M12 event.
  i18n_a11y: Calendar has list view; colour plus text status.
  acceptance: "Scheduling OOS on Hall A with one confirmed event creates a re-accommodation task and emits ResourceOutOfServiceScheduled; the event is not silently cancelled."
  dependency: M26 work orders; M63 tasks.

- id: M09.F09.1.SF09.1.6
  name: Equipment and labor pools
  phase: 3
  release: R1
  actors: [property_admin, sales_manager, fnb_manager]
  screens: [SCR-ADM-equipment-pools]
  inputs: [pool_id, item_type, quantity_owned, quantity_oos, rental_supplier_id, rental_lead_time_h, labor_role, headcount_by_shift]
  states: [active, suspended]
  api: PUT /v1/properties/{pid}/timed-resources/{rid}/pool
  events: [ResourcePoolChanged]
  data: [timed_resource, resource_calendar]
  rules: ["AV kits, projectors, staging, tables, chairs and staff counts are quantity pools", "rental supplier capacity counts as available only when marked contracted with lead time; otherwise shown as 'on request'", "labor pool capacity derives from published roster where M27 is live, else configured headcount (estimate)"]
  security: sales and F&B managers; supplier data read-only from M46
  failure_cases: ["rental supplier not approved in M46 -> excluded", "roster missing -> labor shown as estimate"]
  finance_report_effect: Equipment rental cost becomes event direct cost via PO; owned equipment utilization reported.
  i18n_a11y: Quantity steppers keyboard operable.
  acceptance: "With 4 projectors owned and 4 held, a fifth projector request is infeasible unless a contracted rental supplier with lead time is configured."
  dependency: M46 approved vendors; M27 roster (Phase 4) else estimate.
```

### F09.2 Availability and feasibility search

```yaml
- id: M09.F09.2.SF09.2.1
  name: Space feasibility search
  phase: 3
  release: R1
  actors: [sales_manager, corporate_booker, event_organizer]
  screens: [SCR-SALES-feasibility-search, SCR-CORP-facility-search, SCR-CAPP-facility-search]
  inputs: [dates_or_date_range, start_time, end_time, attendees, layout_type, accessibility_needs, equipment_list, catering_required, breakout_count]
  states: [none]
  api: POST /v1/properties/{pid}/facility-search
  events: [FacilitySearchPerformed]
  data: [timed_resource, space_layout, resource_allocation, changeover_rule]
  rules: ["return only configurations where every requested component is feasible at the same time", "feasibility checks layout capacity, buffers, partitions, equipment, labor and kitchen slot", "results carry freshness timestamp and are not a hold", "multi-date search evaluates each date independently"]
  security: corporate users see only spaces enabled for their agreement; no other customers' booking names exposed (show 'unavailable' only)
  failure_cases: ["no feasible configuration -> alternatives (other date/layout/split rooms) with reasons", "search timeout > 3s -> partial results flagged"]
  finance_report_effect: Search-to-hold conversion metric for M32/M53.
  i18n_a11y: Results announced via aria-live; filters keyboard operable; RTL date pickers.
  acceptance: "AT-G01.1 - two dates, 80 attendees classroom, AV, lunch: date 1 returns Hall A+B; date 2 where Hall A is booked returns only feasible alternatives, never Hall A."
  dependency: SF09.1.2-1.6; M16 kitchen slot pool.

- id: M09.F09.2.SF09.2.2
  name: Restaurant and club table availability
  phase: 3
  release: R1
  actors: [server, club_host, guest, front_desk_agent]
  screens: [SCR-POS-table-plan, SCR-CLUB-floor, SCR-GST-dining-booking]
  inputs: [outlet_id, date, time, party_size, zone_preference, accessibility_needs, duration_minutes]
  states: [available, held, seated, released]
  api: GET /v1/properties/{pid}/outlets/{oid}/table-availability
  events: [TableAvailabilityQueried]
  data: [timed_resource, resource_allocation, resource_link]
  rules: ["turn time per outlet/party size", "table combinations via resource_link", "walk-in capacity reserve configurable per slot", "club tables also count against club zone capacity (M15)"]
  security: guest channel returns slots only, never other diners
  failure_cases: ["stale table plan offline -> seat allowed locally but reconciled; conflict raises host alert"]
  finance_report_effect: Covers and seat utilization for outlet KPIs.
  i18n_a11y: Accessible table flag; text list alternative to floor map.
  acceptance: "Party of 6 with 90 min turn time cannot be booked where two 4-tops are free but not combinable."
  dependency: M57 restaurant reservations consume this API; M15 zone capacity.

- id: M09.F09.2.SF09.2.3
  name: Day-use room slots
  phase: 3
  release: R1
  actors: [front_desk_agent, guest, revenue_manager]
  screens: [SCR-FD-day-use, SCR-GST-day-use]
  inputs: [room_type_id, date, slot_start, slot_end]
  states: [held, confirmed, released]
  api: POST /v1/properties/{pid}/day-use/holds
  events: [DayUseHeld, DayUseConfirmed]
  data: [timed_resource, resource_allocation]
  rules: ["day-use pool per room type is carved from M03 night inventory only for rooms free that night or with confirmed check-out before slot and cleaning ETA before next arrival", "day-use never reduces overnight sellable inventory without M03 hold", "cleaning buffer applied after slot"]
  security: front desk scope
  failure_cases: ["overnight arrival assigned to same room -> day-use slot must end + cleaning before arrival ETA else infeasible"]
  finance_report_effect: Day-use revenue reported separately; excluded from occupancy room-nights per KPI dictionary (docs/06).
  i18n_a11y: Slot times in local format.
  acceptance: "A day-use slot 10:00-16:00 on a room type fully sold overnight is offered only if a departing room is ready by 10:00 and 2h cleaning ends before 16:00 arrival cut."
  dependency: M03 inventory_hold; M06 cleaning ETA; D-202.

- id: M09.F09.2.SF09.2.4
  name: Kitchen production slot capacity
  phase: 3
  release: R1
  actors: [executive_chef, catering_manager]
  screens: [SCR-CAT-kitchen-capacity]
  inputs: [kitchen_id, slot_start, slot_end, cover_capacity, station_capacity, menu_complexity_factor]
  states: [open, reserved, closed]
  api: PUT /v1/properties/{pid}/kitchens/{kid}/production-slots
  events: [KitchenCapacityChanged]
  data: [timed_resource, resource_allocation]
  rules: ["event catering reserves covers x complexity factor in the production slot before service", "a la carte baseline reserve protected", "chef coverage gap (M47) marks slot at-risk, not unavailable"]
  security: kitchen management scope
  failure_cases: ["capacity exceeded -> infeasible with alternative time or off-site production", "chef coverage missing -> at-risk warning on quote"]
  finance_report_effect: Kitchen utilization and overtime drivers for M32/M27.
  i18n_a11y: none beyond standard form accessibility.
  acceptance: "Two 80-cover lunches in the same 11:00-13:00 production slot with capacity 120 covers: second is infeasible."
  dependency: M16; M47.

- id: M09.F09.2.SF09.2.5
  name: Parking and club capacity windows
  phase: 3
  release: R1
  actors: [sales_manager, parking_attendant, club_host]
  screens: [SCR-SALES-feasibility-search, SCR-PARK-capacity]
  inputs: [pool_resource_id, date, start_time, end_time, quantity]
  states: [available, held, confirmed]
  api: GET /v1/properties/{pid}/timed-resources/{rid}/capacity
  events: [none]
  data: [timed_resource, resource_allocation]
  rules: ["parking pool bookable quantity = zone capacity - reserved permits - walk-in reserve (M17)", "club pool = commercial capacity per session (M15) never above safety occupancy"]
  security: read-only for corporate search; exact counts hidden, only feasible yes/no
  failure_cases: ["LPR-derived live occupancy unavailable -> planning capacity used, labelled estimate"]
  finance_report_effect: none directly; feeds pass sales.
  i18n_a11y: Standard.
  acceptance: "Requesting 20 parking passes where pool has 18 unreserved returns infeasible and suggests 18 plus alternative zone."
  dependency: M17 SF17.1.1; M15 SF15.1.2.
```

### F09.3 Composite holds

```yaml
- id: M09.F09.3.SF09.3.1
  name: Create composite hold atomically
  phase: 3
  release: R1
  actors: [sales_manager, corporate_booker, event_organizer]
  screens: [SCR-SALES-composite-hold, SCR-CORP-quote-hold, SCR-CAPP-quote-hold]
  inputs: [quote_id, components(type/resource_or_room_type/interval/quantity/layout), hold_expiry, option_rank, customer_ref]
  states: [requested, held, partially_failed_rolled_back, expired, converted, released]
  api: POST /v1/properties/{pid}/composite-holds
  events: [CompositeHoldPlaced, CompositeHoldRejected]
  data: [composite_hold, composite_hold_component, resource_allocation, inventory_hold]
  rules: ["all components succeed or none; single DB transaction across fac, inv (M03), park (M17) and club (M15) schemas", "locks taken in global order M03 room_type_night asc -> M09 resource_id asc -> M17 -> M15 to avoid deadlock", "hold expiry default 72h (D-201); max set by policy", "hold stores quote/policy snapshot id from M04"]
  security: corporate caller limited to own account and contracted components; rate from agreement only
  failure_cases: ["any component infeasible -> CompositeHoldRejected listing component reasons, zero allocations remain", "lock timeout -> retry with backoff then 409", "duplicate request same idempotency key -> original result"]
  finance_report_effect: Holds are not revenue; pipeline value reported as tentative in M32 sales forecast.
  i18n_a11y: Component failure reasons localized and linked to fields.
  acceptance: "AT-G02.1 - a hold for 10 rooms x 2 nights, Hall A classroom, lunch 80, hosted bar, club visit 80, 20 parking passes, AV is placed atomically; forcing parking to fail leaves no room, space or kitchen allocation."
  dependency: M03 inventory_hold API; M04 quote; M15, M16, M17 pools.

- id: M09.F09.3.SF09.3.2
  name: Hold expiry, extension and option ranking
  phase: 3
  release: R1
  actors: [sales_manager, hold_expiry_worker]
  screens: [SCR-SALES-hold-list]
  inputs: [composite_hold_id, new_expiry, reason, option_rank]
  states: [held, expiring, expired, extended]
  api: POST /v1/properties/{pid}/composite-holds/{hid}/extend
  events: [CompositeHoldExpiring, CompositeHoldExpired, CompositeHoldExtended]
  data: [composite_hold, resource_allocation]
  rules: ["expiry releases every component in one transaction", "second-option holds are waitlisted claims and do not reduce capacity; they auto-promote on first-option release with customer notice and new expiry", "extensions beyond policy need revenue_manager approval"]
  security: extension audited; corporate users can request not grant
  failure_cases: ["worker crash mid-expiry -> idempotent re-run; no partial release", "promotion races with new search -> promotion wins by queue order"]
  finance_report_effect: Expired-hold rate KPI; lost-business log.
  i18n_a11y: Expiry shown with time zone and relative time.
  acceptance: "An expired hold frees all 7 components within one worker cycle (< 60 s) and the second-option hold is promoted exactly once."
  dependency: M63 timers.

- id: M09.F09.3.SF09.3.3
  name: Convert hold to confirmed allocations
  phase: 3
  release: R1
  actors: [sales_manager, corporate_approver, finance_approver]
  screens: [SCR-SALES-composite-hold, SCR-CORP-approval]
  inputs: [composite_hold_id, confirmation_basis(deposit_payment_id|customer_po_id|credit_approval_id), contract_version_id]
  states: [held, confirming, confirmed, confirmation_failed]
  api: POST /v1/properties/{pid}/composite-holds/{hid}/confirm
  events: [CompositeBookingConfirmed]
  data: [composite_hold, resource_allocation, room_block, event_booking]
  rules: ["confirmation requires one satisfied basis per agreement policy (deposit captured, PO accepted, or credit approved)", "conversion re-validates the hold is still held and unexpired; no re-search", "creates room_block (M12) and function_booking records in same transaction via outbox"]
  security: approver must be distinct from requester when policy requires; step-up auth for confirmations above threshold
  failure_cases: ["deposit pending -> stays held; expiry paused only if policy allows", "hold expired before approval -> 409 and re-search offered"]
  finance_report_effect: Moves pipeline from tentative to definite; deposit recorded as liability by M08/M19.
  i18n_a11y: Status stepper accessible.
  acceptance: "AT-G02.2 - approval + deposit converts the hold; room inventory, space schedule, catering slot and parking pool show the booking as confirmed and concurrent attempts to sell the same capacity fail."
  dependency: M10 approvals/credit; M28 deposit; M12.

- id: M09.F09.3.SF09.3.4
  name: Amend composite booking components
  phase: 3
  release: R1
  actors: [sales_manager, event_organizer]
  screens: [SCR-SALES-composite-hold, SCR-EVT-change-order]
  inputs: [composite_id, add_components, remove_components, change_order_id]
  states: [amend_pending, amended, amend_rejected]
  api: POST /v1/properties/{pid}/composite-holds/{hid}/amendments
  events: [CompositeBookingAmended]
  data: [composite_hold_component, resource_allocation, change_order]
  rules: ["increase components acquire new allocations atomically; decrease releases after change order approval", "original allocations untouched if amendment fails", "price delta flows to change order (M12)"]
  security: same scope as creation; amendments after cutoff need events manager approval
  failure_cases: ["partial capacity for increase -> reject whole amendment with feasible max"]
  finance_report_effect: Change order revenue delta; attrition tracking.
  i18n_a11y: Diff view readable by screen reader.
  acceptance: "Increasing parking passes from 20 to 25 when only 3 spare fails atomically and offers 23."
  dependency: M12 change orders.

- id: M09.F09.3.SF09.3.5
  name: Release and cancel composite booking
  phase: 3
  release: R1
  actors: [sales_manager, corporate_admin]
  screens: [SCR-SALES-composite-hold, SCR-CORP-booking-detail]
  inputs: [composite_id, reason_code, cancellation_policy_snapshot]
  states: [cancel_requested, cancelled]
  api: POST /v1/properties/{pid}/composite-holds/{hid}/cancel
  events: [CompositeBookingCancelled]
  data: [composite_hold, resource_allocation]
  rules: ["release all capacity in one transaction", "cancellation fee computed from snapshot policy and contract, posted by M12/M08", "downstream: BEO voided, catering ingredient reservations released, parking permits revoked"]
  security: corporate_admin may cancel own bookings; fees require sales_manager confirmation
  failure_cases: ["downstream consumer failure -> retried via inbox; capacity release not blocked"]
  finance_report_effect: Cancellation fee revenue; lost business report.
  i18n_a11y: Confirmation dialog accessible and explicit about fees.
  acceptance: "AT-G20.10 - corporate cancellation releases all components, revokes 20 parking permits and posts one cancellation fee."
  dependency: M12, M16, M17, M08.
```

### F09.4 Non-double-sale invariant, diary and reconciliation

```yaml
- id: M09.F09.4.SF09.4.1
  name: Enforce non-double-sale invariant
  phase: 2
  release: R1
  actors: [system]
  screens: [none]
  inputs: [resource_id, interval, quantity, allocation_state]
  states: [held, confirmed, released, consumed]
  api: internal port AllocationService.allocate(resource, interval, qty, idempotency_key)
  events: [ResourceAllocated, ResourceAllocationReleased]
  data: [resource_allocation, timed_resource, resource_link]
  rules: ["INV-CAP-1", "unit resources use EXCLUDE USING gist (resource_id WITH =, buffered_range WITH &&) WHERE state in (held,confirmed)", "quantity pools use per-slot capacity counter row with FOR UPDATE", "linked resources propagate claims to ancestors and descendants"]
  security: only via service layer; direct table writes denied by RLS/role
  failure_cases: ["serialization failure -> bounded retry", "on-prem offline staff app -> cannot allocate spaces offline; tables allowed with reconciliation (SF09.2.2)"]
  finance_report_effect: none directly; prevents oversold revenue.
  i18n_a11y: none (system).
  acceptance: "AT-G20.1 - 50 concurrent requests for the last Hall A slot yield exactly one confirmed allocation; property test over random partition bookings never violates INV-CAP-1."
  dependency: ADR on PostgreSQL 16 (docs/03).

- id: M09.F09.4.SF09.4.2
  name: Function diary and resource timeline
  phase: 3
  release: R1
  actors: [sales_manager, catering_manager, fnb_manager, duty_manager]
  screens: [SCR-SALES-function-diary, SCR-STF-diary]
  inputs: [date_range, resource_filter, status_filter]
  states: [prospect, tentative, definite, in_progress, closed, cancelled]
  api: GET /v1/properties/{pid}/function-diary
  events: [none]
  data: [resource_allocation, function_booking, composite_hold]
  rules: ["show holds, options, definite, OOS and buffers distinctly", "customer names masked for roles without sales scope", "freshness timestamp shown"]
  security: role-based field masking
  failure_cases: ["projection lag > 30 s -> stale banner"]
  finance_report_effect: none.
  i18n_a11y: Grid has accessible table alternative; RTL mirrors time axis.
  acceptance: "Diary shows Hall A with setup buffer shaded, option-2 hold marked 'waitlisted', and housekeeping user sees 'Private event' not customer name."
  dependency: M12 function_booking.

- id: M09.F09.4.SF09.4.3
  name: Displacement and re-accommodation
  phase: 3
  release: R1
  actors: [duty_manager, sales_manager]
  screens: [SCR-OPS-exception-queue, SCR-SALES-reaccommodate]
  inputs: [allocation_id, cause(oos|capacity_change|safety), proposed_alternative]
  states: [displaced, alternative_offered, accepted, compensated, cancelled]
  api: POST /v1/properties/{pid}/allocations/{aid}/reaccommodate
  events: [AllocationDisplaced, AllocationReaccommodated]
  data: [resource_allocation, change_order]
  rules: ["displacement never silently deletes allocation", "alternative must pass full feasibility", "customer notified; compensation via M55 approval caps"]
  security: duty_manager
  failure_cases: ["no alternative -> escalate to GM within SLA"]
  finance_report_effect: Compensation cost to event P&L.
  i18n_a11y: Notifications localized.
  acceptance: "OOS on Hall A displaces event X; task created; accepting Hall C creates new allocation and releases old in one transaction."
  dependency: M55, M63.

- id: M09.F09.4.SF09.4.4
  name: Controlled overbooking policy for pools
  phase: 3
  release: R1
  actors: [revenue_manager]
  screens: [SCR-ADM-overbooking-policy]
  inputs: [resource_id, overbook_limit_qty, valid_dates, rationale]
  states: [draft, approved, expired]
  api: PUT /v1/properties/{pid}/timed-resources/{rid}/overbooking
  events: [OverbookingPolicyChanged]
  data: [timed_resource]
  rules: ["unit spaces, partitions, kitchen slots and club safety capacity can never be overbooked", "parking pools and restaurant tables may have explicit small overbook limit with approval", "composite holds for events never use overbook allowance"]
  security: revenue_manager + gm approval
  failure_cases: ["attempt on unit resource -> 422"]
  finance_report_effect: Overbook exposure report.
  i18n_a11y: Standard.
  acceptance: "Setting overbook on Hall A is rejected; parking pool overbook 2 approved allows 2 walk-in permits only."
  dependency: D-204.

- id: M09.F09.4.SF09.4.5
  name: Capacity reconciliation job
  phase: 3
  release: R1
  actors: [integration_admin, capacity_recon_worker]
  screens: [SCR-OPS-exception-queue]
  inputs: [business_date]
  states: [scheduled, running, clean, exceptions_found]
  api: POST /v1/properties/{pid}/capacity-reconciliations
  events: [CapacityReconciliationCompleted, CapacityExceptionRaised]
  data: [resource_allocation, composite_hold, room_block, parking_permit, club_pass]
  rules: ["nightly compare allocations vs owning records (blocks, permits, passes, BEO)", "orphan allocations flagged not auto-deleted", "any INV-CAP-1 breach is Sev-1"]
  security: read-only job; exceptions visible to duty_manager
  failure_cases: ["job overrun -> alert; partial result not marked clean"]
  finance_report_effect: Data-quality indicator in M65.
  i18n_a11y: none.
  acceptance: "Injected orphan allocation is reported next run with owner and age; no breach on clean fixture."
  dependency: M63, M65.
```

**M09 key invariants:** INV-CAP-1; composite holds are all-or-nothing; holds are not revenue; unit spaces/partitions/kitchen slots/club safety capacity are never overbooked; displacement never deletes an allocation silently.

**M09 module acceptance:** AT-G01.1 (feasible-only configurations), AT-G02.1–G02.2 (composite hold and confirmation without double sale), AT-G20.1 (concurrent booking), AT-G20.10 (corporate cancellation release).

**M09 open decisions**
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-201 | Default composite-hold expiry and maximum extension | sales_manager (pilot hotel) | 72 h default, 14 days max with revenue_manager approval |
| D-202 | Whether day-use counts toward occupancy KPIs | revenue_manager / financial_controller | Excluded from room-nights; reported separately |
| D-203 | Source of fire-occupancy limits per space and verification owner | chief_engineer / compliance_officer | Unverified limits cap sale at 50% of stated capacity |
| D-204 | Parking/table overbook allowance | revenue_manager | 0 by default; up to 5% of pool with GM approval |
| D-205 | Temporal adoption for long-running hold/expiry sagas vs pg-boss timers | it_admin (ADR owner) | pg-boss timers with idempotent workers in Phase 3 |

---

## M10 — Corporate accounts

| Item | Value |
|---|---|
| Purpose | Corporate customer master, negotiated agreements that determine who may book what at which price, corporate credit/PO/approval/budget controls, and the sales pipeline from RFQ to signed contract. |
| Build phases | 3 (M20 AR integration completes in 4) |
| Release | R1 |
| Bounded context | `corporate-commercial` (schema `corp`) |
| Systems of record owned | `corporate_account`, `corporate_site`, `corporate_contact`, `cost_center_ref`, `corporate_agreement`, `agreement_version`, `rate_eligibility_rule`, `corporate_credit_line`, `customer_po`, `approval_policy`, `approval_request`, `corporate_budget`, `sales_opportunity`, `corporate_rfq`, `proposal`, `contract_version` |
| Upstream | M02 (corporate identities, SSO federation), M04 (rate plans; negotiated rates are published into M04), M44 (tax ID/invoice rules per jurisdiction), M41 (e-signature), M20 (AR balance/aging) |
| Downstream | M11 (portal), M12 (events), M05 (reservation eligibility), M08 (direct-bill routing), M20 (AR credit), M31 (corporate collaboration reporting only), M32 (corporate margin), M53 (segment forecasting) |

### F10.1 Account structure

```yaml
- id: M10.F10.1.SF10.1.1
  name: Corporate legal entities and branches
  phase: 3
  release: R1
  actors: [sales_manager, ar_clerk, corporate_admin]
  screens: [SCR-SALES-account-detail, SCR-CORP-company-profile]
  inputs: [legal_name, trade_name, registration_no, tax_id, country, subdivision, billing_address, parent_account_id, site_list, industry, preferred_language]
  states: [prospect, active, on_hold, inactive, merged]
  api: POST /v1/corporate-accounts ; PATCH /v1/corporate-accounts/{cid}
  events: [CorporateAccountCreated, CorporateAccountChanged, CorporateAccountStatusChanged]
  data: [corporate_account, corporate_site]
  rules: ["tenant-level record, linked to one or more properties via agreements", "tax_id format validated by M44 rule pack for country; unknown pack -> accepted as unverified and invoice shows manual review flag", "hierarchy max depth 4; child inherits agreements only if explicitly flagged"]
  security: sales scope for property-linked accounts; corporate_admin edits only profile fields, not status or credit
  failure_cases: ["duplicate registration_no in same country -> merge suggestion", "tax ID invalid -> blocks tax invoice issuance (M08) until corrected"]
  finance_report_effect: Account is the AR customer (M20) and the corporate dimension in M32.
  i18n_a11y: Legal name in Latin and Arabic script fields; RTL addresses.
  acceptance: "Two corporations (Acme OM, Beta CA) created with Oman and Ontario tax identifiers pass their rule-pack validation; an invalid Canadian BN is flagged."
  dependency: M44 rule packs.

- id: M10.F10.1.SF10.1.2
  name: Corporate contacts, users and roles
  phase: 3
  release: R1
  actors: [corporate_admin, sales_manager]
  screens: [SCR-CORP-user-admin, SCR-CAPP-user-admin, SCR-SALES-account-contacts]
  inputs: [email, name, phone, role(corporate_admin|corporate_booker|corporate_approver|traveler), site_ids, cost_center_ids, sso_subject]
  states: [invited, active, suspended, removed]
  api: POST /v1/corporate-accounts/{cid}/users
  events: [CorporateUserInvited, CorporateUserActivated, CorporateUserSuspended]
  data: [corporate_contact, identity, role_scope]
  rules: ["corporate identities are separate from staff and vendor identities (M02)", "a user may hold approver and booker roles but may not approve own request", "SSO-provisioned users get roles from mapped IdP groups"]
  security: MFA or corporate SSO mandatory for approver and admin; invitation links single-use, 72h
  failure_cases: ["invite to existing staff email -> separate corporate identity, no privilege merge", "last corporate_admin removal -> blocked"]
  finance_report_effect: none.
  i18n_a11y: Invitation email EN/AR.
  acceptance: "A booker cannot approve their own booking; removing the last admin is rejected."
  dependency: M02 identity/SSO.

- id: M10.F10.1.SF10.1.3
  name: Cost centers and traveler profiles
  phase: 3
  release: R1
  actors: [corporate_admin, corporate_booker]
  screens: [SCR-CORP-cost-centers]
  inputs: [cost_center_code, name, budget_owner_contact_id, traveler_guest_profile_id, default_cost_center]
  states: [active, closed]
  api: POST /v1/corporate-accounts/{cid}/cost-centers
  events: [CorporateCostCenterChanged]
  data: [cost_center_ref, guest_profile]
  rules: ["cost center code is the customer's reference and appears on invoices", "traveler link to guest_profile requires traveler consent for profile sharing with employer (M02 purpose corporate_travel)"]
  security: traveler preferences visible to employer only where consented; no loyalty balance exposure
  failure_cases: ["closed cost center used -> booking blocked with message"]
  finance_report_effect: Invoice lines and corporate analytics grouped by cost center.
  i18n_a11y: Standard.
  acceptance: "Invoice for a stay booked on CC-100 shows CC-100; traveler without consent shows name only to employer."
  dependency: M02 consent; M18 profile.

- id: M10.F10.1.SF10.1.4
  name: Account deduplication and merge
  phase: 3
  release: R1
  actors: [sales_manager, ar_clerk]
  screens: [SCR-SALES-account-merge]
  inputs: [survivor_account_id, merged_account_id, reason]
  states: [proposed, approved, merged, rejected]
  api: POST /v1/corporate-accounts/{cid}/merge
  events: [CorporateAccountMerged]
  data: [corporate_account]
  rules: ["merge requires maker-checker when either account has open AR", "merged account kept as redirect; historical invoices keep original legal name", "agreements do not auto-combine"]
  security: sales_manager proposes, financial_controller approves when AR exists
  failure_cases: ["conflicting tax IDs -> merge blocked"]
  finance_report_effect: AR balances transferred by M20 journal, not edited.
  i18n_a11y: Side-by-side compare accessible table.
  acceptance: "Merging two accounts with open AR requires a second approver and preserves historical invoice names."
  dependency: M20; M65 duplicate policy.
```

### F10.2 Negotiated agreements and rate eligibility

```yaml
- id: M10.F10.2.SF10.2.1
  name: Negotiated agreement versions
  phase: 3
  release: R1
  actors: [sales_manager, revenue_manager]
  screens: [SCR-SALES-agreement-editor]
  inputs: [account_id, property_ids, valid_from, valid_to, room_rates_by_type_season, rate_basis(fixed|discount_from_bar), lra_flag, space_rental_rates, fnb_package_prices, parking_rates, club_access_terms, payment_terms, cancellation_terms, commission_terms]
  states: [draft, pending_approval, active, superseded, expired, terminated]
  api: POST /v1/corporate-accounts/{cid}/agreements ; POST /v1/corporate-accounts/{cid}/agreements/{aid}/versions
  events: [CorporateAgreementVersionActivated, CorporateAgreementExpired]
  data: [corporate_agreement, agreement_version]
  rules: ["versions are immutable once active; changes create a new version with effective date", "bookings and quotes store the agreement_version id used", "discount-from-BAR rates must define floor and rounding", "LRA (last room availability) flag controls whether restrictions apply"]
  security: revenue_manager approval required for rates below floor; audit on every version
  failure_cases: ["overlapping active versions -> rejected", "property not enabled for account -> rejected"]
  finance_report_effect: Negotiated revenue vs BAR dilution reported in M32/M53.
  i18n_a11y: Agreement PDF bilingual.
  acceptance: "AT-G01.2 - Acme sees 45.000 OMR and Beta sees 52.000 OMR for the same room type/date; each booking stores its agreement_version."
  dependency: M04 rate plans.

- id: M10.F10.2.SF10.2.2
  name: Rate eligibility rules
  phase: 3
  release: R1
  actors: [revenue_manager, system]
  screens: [SCR-SALES-agreement-eligibility]
  inputs: [agreement_version_id, eligible_sites, eligible_user_roles, email_domains, booking_channels, blackout_dates, min_max_los, room_type_list, max_rooms_per_booking]
  states: [active, inactive]
  api: POST /v1/properties/{pid}/rate-eligibility/evaluate
  events: [RateEligibilityEvaluated]
  data: [rate_eligibility_rule, agreement_version]
  rules: ["eligibility is evaluated server-side on every quote and booking, never trusted from client", "channel bookings with corporate code are eligible only if channel mapping lists the code (M07)", "email-domain eligibility requires verified email"]
  security: result excludes other accounts' rates
  failure_cases: ["ineligible -> BAR offered with explanation 'not eligible for negotiated rate'"]
  finance_report_effect: Leakage report of ineligible corporate-code attempts.
  i18n_a11y: Explanation text localized.
  acceptance: "A Beta user cannot obtain Acme rate by passing Acme code; blackout date returns BAR with reason."
  dependency: M04; M07.

- id: M10.F10.2.SF10.2.3
  name: Volume commitments and production tracking
  phase: 3
  release: R1
  actors: [sales_manager, revenue_manager]
  screens: [SCR-SALES-account-production]
  inputs: [agreement_version_id, committed_room_nights, committed_revenue, review_dates]
  states: [on_track, at_risk, missed, exceeded]
  api: GET /v1/corporate-accounts/{cid}/production
  events: [CorporateProductionThresholdCrossed]
  data: [agreement_version, reservation, event_booking]
  rules: ["production counts consumed (checked-out) room nights and realized revenue, not bookings", "channel-booked stays count only when code captured"]
  security: sales scope
  failure_cases: ["missing folio close -> figures labelled estimate"]
  finance_report_effect: Account revenue and contribution in M32 corporate margin.
  i18n_a11y: Charts have table fallback.
  acceptance: "After 3 checked-out stays, production shows 6 room nights reconciled and 2 future stays as pipeline, separately."
  dependency: M32.

- id: M10.F10.2.SF10.2.4
  name: Agreement approval and publishing to rate engine
  phase: 3
  release: R1
  actors: [sales_manager, revenue_manager, gm]
  screens: [SCR-SALES-agreement-approval]
  inputs: [agreement_version_id, approver_comments]
  states: [pending_approval, approved, rejected, published, publish_failed]
  api: POST /v1/corporate-accounts/{cid}/agreements/{aid}/versions/{vid}/approve
  events: [CorporateAgreementApproved, NegotiatedRatePublished]
  data: [agreement_version, rate_plan]
  rules: ["approval thresholds by discount depth (D-207)", "publishing creates negotiated rate plans in M04 and channel mappings if enabled", "maker != checker"]
  security: step-up auth for approval
  failure_cases: ["channel publish failure -> version active for direct/portal; channel status shows failed with retry"]
  finance_report_effect: none until bookings.
  i18n_a11y: Standard.
  acceptance: "An agreement 25% below BAR needs GM approval; after approval the negotiated rate appears in portal quotes within 1 minute."
  dependency: M04, M07.

- id: M10.F10.2.SF10.2.5
  name: Agreement renewal and expiry
  phase: 3
  release: R1
  actors: [sales_manager, agreement_expiry_worker]
  screens: [SCR-SALES-renewals]
  inputs: [agreement_id, renewal_terms]
  states: [expiring, renewed, expired]
  api: POST /v1/corporate-accounts/{cid}/agreements/{aid}/renew
  events: [CorporateAgreementExpiring, CorporateAgreementExpired]
  data: [corporate_agreement, agreement_version]
  rules: ["alerts at 90/30/7 days", "bookings made before expiry for stays after expiry honor booked rate", "expired agreement removes eligibility for new quotes"]
  security: sales scope
  failure_cases: ["renewal not approved by expiry -> BAR for new bookings"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "A booking made on the last valid day for a stay next month keeps the negotiated rate; next-day quote shows BAR."
  dependency: M63 timers.
```

### F10.3 Credit, PO, approvals and budgets

```yaml
- id: M10.F10.3.SF10.3.1
  name: Corporate credit line
  phase: 3
  release: R1
  actors: [ar_clerk, financial_controller, sales_manager]
  screens: [SCR-FIN-credit-line]
  inputs: [account_id, credit_limit, currency, payment_terms_days, guarantor_docs, review_date]
  states: [applied, approved, active, on_hold, revoked]
  api: POST /v1/corporate-accounts/{cid}/credit-lines
  events: [CorporateCreditApproved, CorporateCreditHold]
  data: [corporate_credit_line, ar_account]
  rules: ["exposure = open AR + uninvoiced confirmed charges + pending confirmed bookings deposit shortfall", "exposure > limit blocks direct-bill confirmation and prompts deposit/card", "auto hold at configured overdue days"]
  security: finance roles only set limits; sales sees available/used status, not other financials
  failure_cases: ["M20 unavailable (pre-Phase 4) -> exposure from M08 open folios labelled estimate"]
  finance_report_effect: Credit exposure and aging in M20; bad-debt risk.
  i18n_a11y: Currency with minor units per ISO 4217.
  acceptance: "With limit 5,000 OMR and exposure 4,800, a 400 OMR direct-bill booking is blocked and a deposit request is offered."
  dependency: M20 SF20.2.1.

- id: M10.F10.3.SF10.3.2
  name: Customer purchase orders
  phase: 3
  release: R1
  actors: [corporate_booker, sales_manager, ar_clerk]
  screens: [SCR-CORP-po-list, SCR-SALES-booking-po]
  inputs: [customer_po_number, amount_ceiling, currency, valid_to, cost_center_code, attachment_id]
  states: [received, accepted, consumed, exhausted, expired, rejected]
  api: POST /v1/corporate-accounts/{cid}/customer-pos
  events: [CustomerPoAccepted, CustomerPoExhausted]
  data: [customer_po]
  rules: ["PO required per agreement policy; bookings and invoices reference it", "consumption tracked by invoiced amount; exceeding ceiling needs PO amendment or approval", "customer_po is distinct from hotel purchase_order (M49)"]
  security: attachment malware-scanned; corporate users see own POs
  failure_cases: ["PO exhausted mid-event -> change orders flagged; invoicing held for AR review"]
  finance_report_effect: Invoice header carries PO; AR matching key.
  i18n_a11y: Standard.
  acceptance: "AT-G02.3 - event confirmed on PO-778 (8,000 OMR); final invoice 8,450 raises an exception requiring PO amendment before issuance."
  dependency: M08, M20.

- id: M10.F10.3.SF10.3.3
  name: Corporate approval policies
  phase: 3
  release: R1
  actors: [corporate_admin, corporate_approver, corporate_booker]
  screens: [SCR-CORP-approval-policy, SCR-CORP-approvals, SCR-CAPP-approvals]
  inputs: [policy_rules(amount_threshold/room_count/event_size/out_of_policy_rate), approver_chain, delegation, timeout_hours]
  states: [pending, approved, rejected, expired, delegated]
  api: POST /v1/corporate-accounts/{cid}/approval-requests/{arid}/decision
  events: [CorporateApprovalRequested, CorporateApprovalDecided]
  data: [approval_policy, approval_request]
  rules: ["approval request pauses hold expiry only if hotel policy allows", "requester cannot approve own request", "delegation time-bounded and audited", "hotel-side approvals (credit, discount) remain separate"]
  security: step-up MFA for approvals over threshold; push notification contains no amounts on lock screen
  failure_cases: ["approver unavailable -> escalation to delegate after timeout", "hold expires during approval -> booker notified to re-hold"]
  finance_report_effect: none.
  i18n_a11y: Approve/reject buttons labelled; mobile parity.
  acceptance: "An 80-attendee event over 5,000 OMR routes to two approvers in sequence; mobile approval with step-up succeeds; self-approval rejected."
  dependency: M11, M02.

- id: M10.F10.3.SF10.3.4
  name: Corporate budgets by cost center
  phase: 3
  release: R1
  actors: [corporate_admin, corporate_approver]
  screens: [SCR-CORP-budgets]
  inputs: [cost_center_id, period, budget_amount, warn_percent, hard_stop]
  states: [open, warning, exhausted, closed]
  api: PUT /v1/corporate-accounts/{cid}/budgets/{bid}
  events: [CorporateBudgetThresholdReached]
  data: [corporate_budget]
  rules: ["committed = confirmed bookings estimated total; actual = invoiced", "hard_stop blocks booking; soft routes to approver"]
  security: corporate-side data; hotel staff read-only
  failure_cases: ["currency differs -> converted at booking-date rate, labelled estimate"]
  finance_report_effect: none on hotel ledgers.
  i18n_a11y: Standard.
  acceptance: "A booking exceeding remaining CC-200 budget with hard_stop is blocked; with soft it creates an approval request."
  dependency: M11.

- id: M10.F10.3.SF10.3.5
  name: Deposit and payment schedules
  phase: 3
  release: R1
  actors: [sales_manager, ar_clerk, corporate_booker]
  screens: [SCR-SALES-payment-schedule, SCR-CORP-payments]
  inputs: [contract_version_id, milestones(due_date/percent/amount), method]
  states: [scheduled, requested, paid, overdue, waived]
  api: POST /v1/properties/{pid}/events/{eid}/payment-schedule
  events: [DepositRequested, DepositReceived, DepositOverdue]
  data: [contract_version, payment_intent, folio]
  rules: ["deposit is received into the event master account as advance deposit (liability)", "overdue deposit triggers hold-release warning per contract", "waiver requires financial_controller"]
  security: pay-by-link via M28 only; no card data in M10
  failure_cases: ["gateway callback duplicated -> single deposit (M28 idempotency)"]
  finance_report_effect: Advance deposit liability; applied at final invoice.
  i18n_a11y: Pay link page accessible.
  acceptance: "A 30% deposit paid via link appears once on the master account as deposit liability; duplicate webhook creates no second entry."
  dependency: M28, M08.
```

### F10.4 Sales pipeline, RFQ, proposal and contract

```yaml
- id: M10.F10.4.SF10.4.1
  name: Leads and opportunities
  phase: 3
  release: R1
  actors: [sales_manager]
  screens: [SCR-SALES-pipeline]
  inputs: [account_id, contact_id, source, stage, expected_dates, rooms, attendees, estimated_value, probability, owner]
  states: [lead, qualified, proposal, negotiation, won, lost]
  api: POST /v1/properties/{pid}/opportunities
  events: [OpportunityStageChanged, OpportunityWon, OpportunityLost]
  data: [sales_opportunity]
  rules: ["won requires signed contract_version", "lost requires reason code", "stage probabilities configurable"]
  security: record scope to owner and sales team
  failure_cases: ["duplicate opportunity same account/dates -> warning"]
  finance_report_effect: Weighted pipeline in sales forecast; not revenue.
  i18n_a11y: Kanban has list alternative.
  acceptance: "Marking won without a signed contract is rejected."
  dependency: none.

- id: M10.F10.4.SF10.4.2
  name: Corporate RFQ intake
  phase: 3
  release: R1
  actors: [corporate_booker, event_organizer, sales_manager]
  screens: [SCR-CORP-rfq-new, SCR-CAPP-rfq-new, SCR-SALES-rfq-inbox]
  inputs: [dates_options, attendees, layout, rooms_per_night, fnb_requirements, parking_count, club_visit, av_needs, budget, decision_date]
  states: [submitted, acknowledged, in_review, proposal_sent, declined, converted]
  api: POST /v1/corporate-accounts/{cid}/rfqs
  events: [CorporateRfqSubmitted, CorporateRfqAcknowledged]
  data: [corporate_rfq, sales_opportunity]
  rules: ["auto-run feasibility (SF09.2.1) and attach result", "response SLA per property (D-208)", "corporate RFQ is a sales inquiry, distinct from M49 procurement RFQ"]
  security: attachments scanned
  failure_cases: ["SLA breach -> escalation to sales lead"]
  finance_report_effect: RFQ-to-win conversion KPI.
  i18n_a11y: Form usable on mobile; EN/AR.
  acceptance: "AT-G01.3 - an RFQ for the G1 bundle is acknowledged automatically with feasibility for both dates."
  dependency: M09.

- id: M10.F10.4.SF10.4.3
  name: Proposal generation
  phase: 3
  release: R1
  actors: [sales_manager]
  screens: [SCR-SALES-proposal-editor]
  inputs: [opportunity_id, composite_hold_id, template_id, line_items, validity_date, terms]
  states: [draft, sent, viewed, accepted, declined, expired, superseded]
  api: POST /v1/properties/{pid}/proposals
  events: [ProposalSent, ProposalAccepted]
  data: [proposal, quote, composite_hold]
  rules: ["prices from M04 quote with agreement version; manual price changes need discount approval", "proposal validity cannot exceed hold expiry unless hold extended", "total includes taxes/service per M44 rule pack"]
  security: proposal link tokenized, expiring, no PII beyond account
  failure_cases: ["tax rule pack unverified -> proposal shows 'tax estimate' label"]
  finance_report_effect: none.
  i18n_a11y: PDF tagged for accessibility, EN/AR.
  acceptance: "A proposal shows rooms, space, F&B, parking and club lines with taxes and a validity equal to the hold expiry."
  dependency: M04, M44.

- id: M10.F10.4.SF10.4.4
  name: Contract versions and signature
  phase: 3
  release: R1
  actors: [sales_manager, corporate_admin, gm]
  screens: [SCR-SALES-contract, SCR-CORP-contract-sign]
  inputs: [proposal_id, clauses(attrition/cancellation/cutoff/f&b_minimum/deposit), signatory]
  states: [draft, internal_review, sent_for_signature, signed, countersigned, amended, void]
  api: POST /v1/properties/{pid}/contracts/{ctid}/send-for-signature
  events: [ContractSigned, ContractAmended]
  data: [contract_version]
  rules: ["each amendment is a new contract_version with diff", "signature via M41 envelope with document hash", "non-standard clauses need gm approval"]
  security: signed PDFs immutable; access restricted to account and sales
  failure_cases: ["signature provider unavailable -> wet-signature upload path with evidence label"]
  finance_report_effect: Contract terms drive attrition/cancellation fee postings.
  i18n_a11y: Accessible signing flow per M41.
  acceptance: "Amending attendance creates contract v2 linked to v1; both hashes retained."
  dependency: M41.

- id: M10.F10.4.SF10.4.5
  name: Win/loss and pipeline analytics
  phase: 3
  release: R1
  actors: [sales_manager, gm, revenue_manager]
  screens: [SCR-MGT-sales-pipeline]
  inputs: [period, segment, owner]
  states: [none]
  api: GET /v1/properties/{pid}/reports/sales-pipeline
  events: [none]
  data: [sales_opportunity, proposal, event_booking]
  rules: ["actual vs pipeline separated", "lost reasons standardized"]
  security: report row scope
  failure_cases: ["incomplete folios -> estimate label"]
  finance_report_effect: Feeds M32 corporate/events report (SF32.4.6).
  i18n_a11y: Table alternative for charts.
  acceptance: "Report shows win rate and lost-reason breakdown reconciling to opportunity count."
  dependency: M32.
```

**M10 key invariants:** agreement versions are immutable and referenced by every quote/booking; eligibility is evaluated server-side; exposure > credit limit never auto-confirms direct bill; requester ≠ approver.

**M10 module acceptance:** AT-G01.2 (two corporations at different rates, Phase 3 exit), AT-G02.3 (PO/approval), AT-G20.10 (corporate cancellation fee from contract).

**M10 open decisions**
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-206 | Is the corporate account tenant-wide or property-scoped in single-hotel R1 | product_owner | Tenant-wide record, property-linked agreements |
| D-207 | Discount-depth approval thresholds | revenue_manager / gm | >15% below BAR needs revenue_manager; >25% GM |
| D-208 | Corporate RFQ response SLA | sales_manager | 24 business hours |
| D-209 | Whether sales pipeline uses built-in CRM (M52) or an external CRM | sales_manager / it_admin | Built-in; external CRM out of scope for R1 |
| D-210 | Credit exposure formula components pre-Phase 4 | financial_controller | Open folios + confirmed uninvoiced estimate, labelled estimate |

---

## M11 — Corporate portal and apps (MetriStay Business)

| Item | Value |
|---|---|
| Purpose | Self-service corporate channel on web plus signed downloadable Android/iOS apps with SSO: search contracted facilities/rates by date and attendees, compare, quote, hold, approve, book, manage rooming lists and itineraries, message the hotel, pay and analyse invoices. |
| Build phases | 3 (payments via M28 complete in 5) |
| Release | R1 (public app-store listing is a release dependency, not a feature claim) |
| Bounded context | `corporate-channel` (BFF over `corporate-commercial`, `facility-scheduling`, `events-mice`); owns no commercial SoR |
| Systems of record owned | `app_release`, `app_device_registration`, `portal_saved_search`, `portal_notification_pref` |
| Upstream | M02 (identity, SSO federation, MFA), M09, M10, M12, M04, M08, M28, M18 messaging service |
| Downstream | M32/M65 (portal funnel analytics), M52 (corporate contact engagement, consented) |

**Distribution honesty:** Android/iOS builds are produced and signed by the hotel operator's (or Metrikingdom's) developer accounts. Public Google Play / Apple App Store publication status is `blocked` until developer accounts and store review exist (D-212). Private distribution (Apple Business Manager custom app, managed Google Play, or MDM) is the fallback path; each is `unverified-assumption` until tested.

### F11.1 Access, SSO and signed distribution

```yaml
- id: M11.F11.1.SF11.1.1
  name: Corporate web portal shell
  phase: 3
  release: R1
  actors: [corporate_admin, corporate_booker, corporate_approver, event_organizer]
  screens: [SCR-CORP-home, SCR-CORP-inbox]
  inputs: [none]
  states: [signed_out, signed_in, session_expired]
  api: GET /v1/corporate/me/home
  events: [CorporatePortalSignedIn]
  data: [corporate_contact, approval_request, event_booking]
  rules: ["home shows 3-7 tasks - pending approvals, expiring holds, rooming list deadlines, unpaid invoices, messages", "only accounts/properties the user is scoped to", "Next.js SSR low-bandwidth mode"]
  security: OIDC session; CSRF protection; session idle timeout 30 min; BOLA tests per OWASP API1
  failure_cases: ["backend degraded -> read-only banner with last-sync time"]
  finance_report_effect: none.
  i18n_a11y: WCAG 2.2 AA; EN/AR with RTL; keyboard navigation of all tasks.
  acceptance: "A booker of Acme never sees Beta data via crafted IDs (automated BOLA test suite passes)."
  dependency: M02.

- id: M11.F11.1.SF11.1.2
  name: Signed Android and iOS app builds and distribution
  phase: 3
  release: R1
  actors: [it_admin, corporate_admin]
  screens: [SCR-ADM-app-releases, SCR-CAPP-about]
  inputs: [build_number, platform, signing_key_ref, min_supported_version, distribution_channel(public_store|private_store|mdm|internal_test), release_notes]
  states: [built, signed, internal_testing, submitted, approved, released, deprecated, blocked]
  api: POST /v1/admin/app-releases
  events: [AppReleaseSigned, AppReleasePublished, AppVersionDeprecated]
  data: [app_release]
  rules: ["release builds signed with keys in KMS/HSM-backed store; signing key never in repo", "store publication status reflects actual store response; 'blocked' until developer accounts exist", "min_supported_version enforced at API with upgrade prompt", "SBOM and dependency scan attached to each release"]
  security: signing restricted to release pipeline identity; two-person approval for production release
  failure_cases: ["store rejection -> status rejected with reason; private channel remains", "forced upgrade while offline -> app read-only until upgrade"]
  finance_report_effect: none.
  i18n_a11y: Store listing and in-app strings EN/AR.
  acceptance: "Build pipeline produces signed APK/AAB and IPA with verifiable signatures; API rejects a version below min_supported_version with an upgrade response."
  dependency: Developer accounts (D-212); Expo EAS or equivalent build service licence review.

- id: M11.F11.1.SF11.1.3
  name: Corporate SSO and MFA
  phase: 3
  release: R1
  actors: [corporate_admin, it_admin]
  screens: [SCR-CORP-sso-setup, SCR-CAPP-login]
  inputs: [idp_type(oidc|saml), metadata_url, client_id, group_to_role_mapping, allowed_domains, jit_provisioning]
  states: [draft, testing, active, disabled]
  api: PUT /v1/corporate-accounts/{cid}/sso
  events: [CorporateSsoConfigured, CorporateSsoDisabled]
  data: [corporate_account, identity]
  rules: ["federation brokered by Keycloak per tenant realm", "mobile uses system browser with PKCE, never embedded webview credentials", "fallback local login with MFA only if corporate_admin enables it", "deprovisioned IdP user loses access at next token refresh (max 15 min)"]
  security: signed SAML assertions; certificate rotation alerts; tokens stored in platform keystore
  failure_cases: ["IdP outage -> local MFA fallback if enabled, else clear error", "clock skew -> tolerance 2 min"]
  finance_report_effect: none.
  i18n_a11y: Login accessible; no CAPTCHA.
  acceptance: "A user disabled in the corporate IdP cannot call the API after 15 minutes; login on iOS uses ASWebAuthenticationSession with PKCE."
  dependency: M02; Keycloak ADR.

- id: M11.F11.1.SF11.1.4
  name: Device registration and remote revoke
  phase: 3
  release: R1
  actors: [corporate_admin, corporate_booker]
  screens: [SCR-CORP-devices, SCR-CAPP-settings]
  inputs: [device_id, platform, push_token, app_version]
  states: [registered, active, revoked]
  api: POST /v1/corporate/me/devices ; DELETE /v1/corporate/me/devices/{did}
  events: [CorporateDeviceRegistered, CorporateDeviceRevoked]
  data: [app_device_registration]
  rules: ["revocation invalidates refresh tokens for that device", "push payloads carry no amounts or guest names"]
  security: device-bound refresh token; jailbreak/root detection warns only
  failure_cases: ["push token invalid -> fallback to email notification"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Revoking a lost phone ends its session on next call."
  dependency: M02.

- id: M11.F11.1.SF11.1.5
  name: Web/mobile core-journey parity
  phase: 3
  release: R1
  actors: [it_admin, product_owner]
  screens: [SCR-ADM-parity-matrix]
  inputs: [journey_id, web_status, android_status, ios_status, test_run_id]
  states: [planned, implemented, tested, released]
  api: GET /v1/admin/app-parity
  events: [none]
  data: [app_release]
  rules: ["core journey = search -> compare -> quote -> hold -> approve -> book -> rooming list -> itinerary -> invoice -> pay; must pass on web, Android and iOS before release", "non-core features may lag with visible 'use web' link"]
  security: none beyond standard
  failure_cases: ["parity test fails on one platform -> release blocked for that platform"]
  finance_report_effect: none.
  i18n_a11y: Accessibility tests (TalkBack/VoiceOver) part of parity.
  acceptance: "End-to-end parity suite passes for the core journey on web, Android and iOS test devices."
  dependency: docs/09 test plan.
```

### F11.2 Search and compare

```yaml
- id: M11.F11.2.SF11.2.1
  name: Contracted facility, room and package search
  phase: 3
  release: R1
  actors: [corporate_booker, event_organizer]
  screens: [SCR-CORP-facility-search, SCR-CAPP-facility-search]
  inputs: [date_options(1-3), attendees, layout, rooms_per_night, room_types, meals, hosted_bar, club_visit, parking_passes, av_list, accessibility_needs]
  states: [none]
  api: POST /v1/corporate/properties/{pid}/search
  events: [CorporateSearchPerformed]
  data: [timed_resource, room_type_night_inventory, agreement_version, quote]
  rules: ["only feasible configurations appear (SF09.2.1 + M03 availability)", "prices from caller's agreement; BAR shown only if agreement allows fallback", "totals include taxes/fees per M44; unverified pack labelled estimate"]
  security: agreement scope; rate isolation between corporations
  failure_cases: ["no feasible config -> alternatives with reasons", "rate engine unavailable -> no prices shown, not stale prices"]
  finance_report_effect: Search funnel metrics.
  i18n_a11y: Results list screen-reader friendly; currency formatting per locale.
  acceptance: "AT-G01.1 - G1 bundle search on two dates returns only feasible configurations at Acme contracted rates; Beta sees its own rates."
  dependency: M09, M03, M04, M10.

- id: M11.F11.2.SF11.2.2
  name: Compare room blocks and packages
  phase: 3
  release: R1
  actors: [corporate_booker, event_organizer]
  screens: [SCR-CORP-compare, SCR-CAPP-compare]
  inputs: [result_ids(max 4)]
  states: [none]
  api: POST /v1/corporate/properties/{pid}/compare
  events: [none]
  data: [quote]
  rules: ["side-by-side date, spaces, capacity headroom, price components, cancellation/attrition terms", "no hidden fees; mandatory charges in total"]
  security: same as search
  failure_cases: ["result expired -> re-price prompt"]
  finance_report_effect: none.
  i18n_a11y: Comparison table with row headers; mobile stacked cards.
  acceptance: "Comparing two dates shows per-component totals that equal the quote totals."
  dependency: SF11.2.1.

- id: M11.F11.2.SF11.2.3
  name: Saved searches and availability alerts
  phase: 3
  release: R1
  actors: [corporate_booker]
  screens: [SCR-CORP-saved-searches]
  inputs: [search_criteria, alert_enabled]
  states: [saved, alerting, disabled]
  api: POST /v1/corporate/saved-searches
  events: [SavedSearchMatched]
  data: [portal_saved_search]
  rules: ["alerts are notifications, not holds", "rate-limited to 1 per search per day"]
  security: own searches only
  failure_cases: ["criteria outdated -> auto-disable after 90 days"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Alert fires when a previously infeasible date becomes feasible and does not reserve capacity."
  dependency: M63.

- id: M11.F11.2.SF11.2.4
  name: Individual traveler booking under agreement
  phase: 3
  release: R1
  actors: [corporate_booker, traveler]
  screens: [SCR-CORP-book-stay, SCR-CAPP-book-stay]
  inputs: [traveler_profile_id, dates, room_type, cost_center, payer(direct_bill|traveler_card)]
  states: [quoted, booked, cancelled]
  api: POST /v1/corporate/properties/{pid}/reservations
  events: [ReservationConfirmed]
  data: [reservation, quote, agreement_version]
  rules: ["eligibility and credit exposure checked server-side", "booker vs occupant vs payer recorded (M05)", "approval policy applied"]
  security: traveler PII limited to booking needs
  failure_cases: ["credit hold -> card guarantee path"]
  finance_report_effect: Corporate segment revenue.
  i18n_a11y: Standard.
  acceptance: "Acme booker books a traveler on direct bill with CC-100; invoice routes to Acme AR."
  dependency: M05, M08.
```

### F11.3 Quote, hold, approval and booking

```yaml
- id: M11.F11.3.SF11.3.1
  name: Quote with expiry and policy snapshot
  phase: 3
  release: R1
  actors: [corporate_booker, event_organizer]
  screens: [SCR-CORP-quote, SCR-CAPP-quote]
  inputs: [search_result_id]
  states: [issued, expired, accepted, superseded]
  api: POST /v1/corporate/properties/{pid}/quotes
  events: [QuoteIssued]
  data: [quote, policy_snapshot]
  rules: ["firm expiry timestamp shown", "quote is not a hold", "quote PDF bilingual"]
  security: quote bound to account
  failure_cases: ["expired quote accept -> reprice"]
  finance_report_effect: Quote conversion KPI.
  i18n_a11y: Expiry with time zone.
  acceptance: "Quote shows firm expiry; accepting after expiry forces re-price."
  dependency: M04.

- id: M11.F11.3.SF11.3.2
  name: Request composite hold from portal
  phase: 3
  release: R1
  actors: [corporate_booker, event_organizer]
  screens: [SCR-CORP-quote-hold, SCR-CAPP-quote-hold]
  inputs: [quote_id, option_rank]
  states: [held, rejected, expired]
  api: POST /v1/corporate/properties/{pid}/quotes/{qid}/hold
  events: [CompositeHoldPlaced]
  data: [composite_hold]
  rules: ["delegates to SF09.3.1", "portal self-hold allowed up to policy size (D-211); larger goes to sales review"]
  security: account scope
  failure_cases: ["component failed -> reasons shown, nothing held"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Hold for G1 bundle from iOS app succeeds atomically and is visible on web."
  dependency: M09.

- id: M11.F11.3.SF11.3.3
  name: Internal approval in portal and app
  phase: 3
  release: R1
  actors: [corporate_approver]
  screens: [SCR-CORP-approvals, SCR-CAPP-approvals]
  inputs: [approval_request_id, decision, comment]
  states: [pending, approved, rejected]
  api: POST /v1/corporate-accounts/{cid}/approval-requests/{arid}/decision
  events: [CorporateApprovalDecided]
  data: [approval_request]
  rules: ["see SF10.3.3"]
  security: step-up MFA
  failure_cases: ["hold expired during approval -> approval recorded but booking requires re-hold"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Approving on Android triggers confirmation path when deposit/PO is satisfied."
  dependency: M10.

- id: M11.F11.3.SF11.3.4
  name: Confirm booking with deposit or PO
  phase: 3
  release: R1
  actors: [corporate_booker]
  screens: [SCR-CORP-confirm, SCR-CAPP-confirm]
  inputs: [composite_hold_id, customer_po_id_or_payment_method, contract_acceptance]
  states: [confirming, confirmed, failed]
  api: POST /v1/corporate/properties/{pid}/composite-holds/{hid}/confirm
  events: [CompositeBookingConfirmed]
  data: [composite_hold, contract_version, payment_intent]
  rules: ["delegates to SF09.3.3", "contract signature via M41 before confirmation when required"]
  security: payment via M28 hosted fields/pay link; no card data in portal
  failure_cases: ["payment declined -> hold kept until expiry"]
  finance_report_effect: Deposit liability.
  i18n_a11y: Standard.
  acceptance: "AT-G02.2 - corporate confirms via PO; all inventories show confirmed."
  dependency: M09, M28, M41.
```

### F11.4 Rooming list, itinerary and messaging

```yaml
- id: M11.F11.4.SF11.4.1
  name: Rooming list upload and edit
  phase: 3
  release: R1
  actors: [event_organizer, corporate_booker]
  screens: [SCR-CORP-rooming-list, SCR-CAPP-rooming-list]
  inputs: [room_block_id, file_csv_xlsx_or_rows, guest_name, arrival, departure, room_type, sharer, payer(master|individual), special_requests]
  states: [draft, submitted, validated, partially_invalid, applied]
  api: POST /v1/properties/{pid}/room-blocks/{bid}/rooming-list
  events: [RoomingListSubmitted, RoomingListApplied]
  data: [rooming_list, reservation, block_pickup]
  rules: ["rows validated against block inventory by type/night", "each valid row creates or updates a reservation picked up from block", "after cutoff, changes need hotel approval and may reprice"]
  security: guest PII minimal; file scanned; organizer sees only own block
  failure_cases: ["row exceeds block -> row rejected with reason, others applied", "duplicate guest rows -> flagged"]
  finance_report_effect: Pickup metrics feed M32 SF32.1.4.
  i18n_a11y: Template EN/AR; Arabic names supported; accessible table editor.
  acceptance: "Uploading 12 rows to a 10-room block applies 10 and rejects 2 with 'exceeds block' reasons."
  dependency: M12 SF12.1.2.

- id: M11.F11.4.SF11.4.2
  name: Event itinerary view
  phase: 3
  release: R1
  actors: [event_organizer, corporate_booker]
  screens: [SCR-CORP-itinerary, SCR-CAPP-itinerary]
  inputs: [event_booking_id]
  states: [proposed, confirmed, changed, cancelled]
  api: GET /v1/corporate/events/{eid}/itinerary
  events: [none]
  data: [event_booking, beo_version, room_block, parking_permit, club_pass]
  rules: ["each item labelled confirmed vs proposed", "BEO customer view excludes internal cost/staff notes"]
  security: organizer scope
  failure_cases: ["projection stale -> timestamp shown"]
  finance_report_effect: none.
  i18n_a11y: Chronological list accessible; add-to-calendar export.
  acceptance: "After a BEO revision, itinerary shows changed item with 'pending your acceptance' label."
  dependency: M12.

- id: M11.F11.4.SF11.4.3
  name: Corporate-hotel messaging
  phase: 3
  release: R1
  actors: [event_organizer, sales_manager, catering_manager]
  screens: [SCR-CORP-messages, SCR-CAPP-messages, SCR-SALES-messages]
  inputs: [thread_context(event|rfq|invoice), body, attachments]
  states: [open, awaiting_hotel, awaiting_customer, closed]
  api: POST /v1/corporate/threads/{tid}/messages
  events: [CorporateMessagePosted]
  data: [message_thread, message]
  rules: ["uses M18 messaging service with corporate context", "response SLA tracked", "messages are not change orders; hotel converts request to change order explicitly"]
  security: attachments scanned; retention per M02
  failure_cases: ["push fails -> email notification"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "A message requesting 5 more lunches creates no charge until a change order is issued and accepted."
  dependency: M18.

- id: M11.F11.4.SF11.4.4
  name: Attendee parking and club passes
  phase: 3
  release: R1
  actors: [event_organizer]
  screens: [SCR-CORP-passes, SCR-CAPP-passes]
  inputs: [event_booking_id, attendee_name, plate_number, pass_type(parking|club), valid_window]
  states: [allocated, issued, used, revoked]
  api: POST /v1/corporate/events/{eid}/passes
  events: [ParkingPermitIssued, ClubPassIssued]
  data: [parking_permit, club_pass, plate_registration]
  rules: ["passes drawn from confirmed composite allocation quantity only", "plate capture requires notice per M17 privacy rule"]
  security: plate data visible to organizer and parking roles only
  failure_cases: ["allocation exhausted -> change order path"]
  finance_report_effect: Pass value billed per contract to master account.
  i18n_a11y: Plate input supports Arabic and Latin characters.
  acceptance: "Organizer issues 20 parking passes with plates; the 21st is rejected."
  dependency: M17, M15.
```

### F11.5 Invoices, payments and analytics

```yaml
- id: M11.F11.5.SF11.5.1
  name: Corporate invoices and statements
  phase: 3
  release: R1
  actors: [corporate_admin, corporate_approver]
  screens: [SCR-CORP-invoices, SCR-CAPP-invoices]
  inputs: [period, status_filter, cost_center]
  states: [issued, partially_paid, paid, disputed, credited]
  api: GET /v1/corporate-accounts/{cid}/invoices
  events: [none]
  data: [invoice, credit_note, ar_account]
  rules: ["invoices are M08/M20 documents; portal is read-only view", "dispute creates AR case"]
  security: account scope; downloads logged
  failure_cases: ["AR not yet live -> folio-based invoice list"]
  finance_report_effect: none (view).
  i18n_a11y: Tax invoice PDF bilingual per M44.
  acceptance: "Organizer downloads final event invoice with PO number and cost center lines."
  dependency: M08, M20.

- id: M11.F11.5.SF11.5.2
  name: Online invoice payment
  phase: 5
  release: R1
  actors: [corporate_admin]
  screens: [SCR-CORP-pay-invoice, SCR-CAPP-pay-invoice]
  inputs: [invoice_ids, amount, method]
  states: [initiated, authorized, captured, failed]
  api: POST /v1/corporate-accounts/{cid}/payments
  events: [CorporatePaymentReceived]
  data: [payment_intent, ar_account]
  rules: ["partial payments allocate oldest-first unless chosen", "receipt only after PSP confirmation"]
  security: M28 tokenized; no PAN storage
  failure_cases: ["duplicate callback -> single allocation"]
  finance_report_effect: AR receipt allocation.
  i18n_a11y: Standard.
  acceptance: "Payment captured once and allocated; duplicate webhook ignored."
  dependency: M28 certified gateway.

- id: M11.F11.5.SF11.5.3
  name: Corporate spend and compliance analytics
  phase: 3
  release: R1
  actors: [corporate_admin, corporate_approver]
  screens: [SCR-CORP-analytics, SCR-CAPP-analytics]
  inputs: [period, cost_center, traveler, category]
  states: [none]
  api: GET /v1/corporate-accounts/{cid}/analytics
  events: [none]
  data: [invoice, reservation, event_booking]
  rules: ["actual (invoiced) vs committed separated", "policy compliance - out-of-policy bookings, approval lead time"]
  security: aggregate; traveler-level detail only for corporate_admin
  failure_cases: ["data freshness shown"]
  finance_report_effect: none.
  i18n_a11y: Charts with table alternatives.
  acceptance: "Totals reconcile with invoice list for the period."
  dependency: M32.

- id: M11.F11.5.SF11.5.4
  name: Export of bookings and invoices
  phase: 3
  release: R1
  actors: [corporate_admin]
  screens: [SCR-CORP-exports]
  inputs: [dataset, period, format(csv|xlsx|pdf)]
  states: [requested, ready, expired]
  api: POST /v1/corporate-accounts/{cid}/exports
  events: [CorporateExportGenerated]
  data: [invoice, reservation]
  rules: ["export links expire in 24h", "no guest data beyond account's own travelers"]
  security: download audited
  failure_cases: ["large export -> async job"]
  finance_report_effect: none.
  i18n_a11y: CSV UTF-8 with BOM for Arabic.
  acceptance: "XLSX export of Q3 invoices matches the portal list."
  dependency: none.
```

**M11 key invariants:** portal never owns commercial truth (it calls M09/M10/M12/M08); rate isolation between corporations; public store status reflects actual store response; core journey parity across web/Android/iOS before release.

**M11 module acceptance:** AT-G01.1 (search from portal and app), AT-G02.2 (confirm), Phase 3 exit "two corporations at different rates".

**M11 open decisions**
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-211 | Max composite size a corporate user may self-hold without sales review | sales_manager | 15 rooms / 100 attendees |
| D-212 | Developer-account ownership (Metrikingdom vs hotel) and public vs private app distribution | product_owner / it_admin | Metrikingdom accounts; private distribution until store approval; status `blocked` |
| D-213 | White-label per hotel vs single MetriStay Business app | product_owner / marketing_manager | Single app with property selection |
| D-214 | Minimum supported OS versions | it_admin | Android 10+, iOS 16+ |

---

## M12 — Group, events and MICE

| Item | Value |
|---|---|
| Purpose | Turn a confirmed composite booking into an operated event: room blocks with pickup and cutoff, function diary entries, BEOs whose revisions reach kitchen/bar/AV/parking/club, change orders, attendee check-in, master vs individual billing and post-event reconciliation. |
| Build phases | 3 |
| Release | R1 |
| Bounded context | `events-mice` (schema `evt`) |
| Systems of record owned | `event_booking`, `function_booking`, `room_block`, `room_block_night`, `block_pickup`, `rooming_list`, `beo`, `beo_version`, `beo_line`, `beo_acknowledgment`, `change_order`, `attendee`, `event_reconciliation` |
| Upstream | M09 (composite allocations), M10 (contract, PO, credit), M03 (block inventory), M04 (group rates), M08 (master account, routing), M16 (catering), M13 (hosted bar), M15 (club), M17 (parking), M47 (chef coverage) |
| Downstream | M05 (reservations from pickup), M08, M16, M13, M14 (issues to BEO), M17, M15, M32 (event P&L), M53 (group wash), M20 (AR) |

### F12.1 Room blocks, pickup and cutoff

```yaml
- id: M12.F12.1.SF12.1.1
  name: Create room block from composite booking
  phase: 3
  release: R1
  actors: [sales_manager, system]
  screens: [SCR-EVT-room-block]
  inputs: [composite_hold_id, room_type_nights, group_rate_plan_id, cutoff_date, attrition_percent, comp_policy]
  states: [tentative, definite, released, cancelled, actualized]
  api: POST /v1/properties/{pid}/room-blocks
  events: [RoomBlockCreated, RoomBlockDefinite]
  data: [room_block, room_block_night, inventory_hold]
  rules: ["block quantities are M03 holds of type group_block; they reduce sellable inventory", "block and its M03 hold share composite hold transaction", "group rate from agreement or contract"]
  security: sales scope
  failure_cases: ["M03 inventory insufficient -> block not created (composite rollback)"]
  finance_report_effect: Blocked rooms appear in forecast as group segment; not revenue until stay.
  i18n_a11y: Grid by night with accessible table.
  acceptance: "10 rooms x 2 nights block reduces sellable inventory by 10 per night; channels see reduced availability."
  dependency: M03, M09.

- id: M12.F12.1.SF12.1.2
  name: Pickup tracking
  phase: 3
  release: R1
  actors: [sales_manager, front_desk_agent, event_organizer]
  screens: [SCR-EVT-pickup, SCR-CORP-rooming-list]
  inputs: [room_block_id, reservation_id]
  states: [unpicked, picked_up, cancelled_returned]
  api: POST /v1/properties/{pid}/room-blocks/{bid}/pickups
  events: [RoomBlockPickedUp, RoomBlockPickupCancelled]
  data: [block_pickup, reservation]
  rules: ["reservation from block consumes block hold, not general inventory", "cancelled pickup returns room to block before cutoff and to general inventory after cutoff", "pickup percentage per night calculated"]
  security: organizer sees own block only
  failure_cases: ["pickup exceeding block by type -> rejected unless sales overrides with general inventory check"]
  finance_report_effect: Pickup % in M32 SF32.1.4.
  i18n_a11y: Standard.
  acceptance: "Picking up 8 of 10 shows 80% and 2 remaining per night; cancelling one after cutoff releases it to general inventory."
  dependency: M05.

- id: M12.F12.1.SF12.1.3
  name: Cutoff and automatic release
  phase: 3
  release: R1
  actors: [block_cutoff_worker, sales_manager]
  screens: [SCR-EVT-cutoff-queue]
  inputs: [room_block_id, cutoff_date, extension_request]
  states: [pre_cutoff, cutoff_warning, released, extended]
  api: POST /v1/properties/{pid}/room-blocks/{bid}/cutoff
  events: [RoomBlockCutoffWarning, RoomBlockReleased]
  data: [room_block, room_block_night, inventory_hold]
  rules: ["at cutoff (property time zone) unpicked rooms return to general inventory in one transaction", "warnings at 7 and 2 days to organizer and sales", "extension needs revenue_manager approval"]
  security: worker identity audited
  failure_cases: ["worker late -> runs on recovery idempotently", "extension after release -> re-hold subject to availability"]
  finance_report_effect: Released rooms become sellable; attrition calculated later.
  i18n_a11y: Notifications localized.
  acceptance: "At cutoff 2 unpicked rooms/night are released exactly once and channel ARI updates."
  dependency: M07 ARI.

- id: M12.F12.1.SF12.1.4
  name: Attrition, wash and penalties
  phase: 3
  release: R1
  actors: [sales_manager, ar_clerk]
  screens: [SCR-EVT-attrition]
  inputs: [room_block_id, contracted_room_nights, actual_room_nights, attrition_allowance]
  states: [calculated, waived, invoiced]
  api: POST /v1/properties/{pid}/room-blocks/{bid}/attrition
  events: [AttritionCalculated]
  data: [room_block, contract_version, folio_charge]
  rules: ["penalty = max(0, contracted x (1 - allowance) - actual) x group rate, per contract clause", "resale credit applied if clause present", "waiver needs gm"]
  security: sales + finance
  failure_cases: ["contract missing clause -> no automatic penalty; manual review"]
  finance_report_effect: Attrition revenue line in event P&L; wash feeds M53 group wash.
  i18n_a11y: Standard.
  acceptance: "20 contracted, 80% allowance, 14 actual -> 2 room nights penalty posted once to master account."
  dependency: M10 contract.

- id: M12.F12.1.SF12.1.5
  name: Group rates and comp policy
  phase: 3
  release: R1
  actors: [sales_manager, revenue_manager]
  screens: [SCR-EVT-room-block]
  inputs: [comp_ratio(1 per N), comp_room_type, staff_rate]
  states: [active]
  api: PUT /v1/properties/{pid}/room-blocks/{bid}/pricing
  events: [RoomBlockPricingChanged]
  data: [room_block, rate_plan]
  rules: ["comps earned on actualized room nights", "comp value credited on master account, not by deleting charges"]
  security: pricing change after definite needs revenue_manager
  failure_cases: ["comp computed on pickup not actuals -> prevented"]
  finance_report_effect: Comp allowance reported as discount, revenue gross.
  i18n_a11y: Standard.
  acceptance: "1 per 20 comp on 40 actual room nights credits 2 room nights on the master account."
  dependency: M08.
```

### F12.2 Function diary, proposals and resources

```yaml
- id: M12.F12.2.SF12.2.1
  name: Event and function bookings
  phase: 3
  release: R1
  actors: [sales_manager, catering_manager]
  screens: [SCR-EVT-event-detail, SCR-SALES-function-diary]
  inputs: [event_name, account_id, organizer_contact, functions(space/layout/start/end/type/attendees)]
  states: [prospect, tentative, definite, in_progress, completed, reconciled, closed, cancelled]
  api: POST /v1/properties/{pid}/events ; POST /v1/properties/{pid}/events/{eid}/functions
  events: [EventBookingStatusChanged, FunctionBookingChanged]
  data: [event_booking, function_booking, resource_allocation]
  rules: ["each function holds its resource_allocation via M09", "definite requires contract signed + deposit/PO/credit", "closed only after reconciliation and final invoice"]
  security: sales/catering scope
  failure_cases: ["status skip -> rejected by state machine SM-event (docs/02)"]
  finance_report_effect: Event is the P&L object in M32.
  i18n_a11y: Standard.
  acceptance: "Event cannot move to definite without signed contract and satisfied confirmation basis."
  dependency: M09, M10.

- id: M12.F12.2.SF12.2.2
  name: AV, equipment and staff reservation
  phase: 3
  release: R1
  actors: [catering_manager, sales_manager]
  screens: [SCR-EVT-resources]
  inputs: [function_id, equipment_pool_id, quantity, staff_role, headcount, interval]
  states: [requested, allocated, released]
  api: POST /v1/properties/{pid}/events/{eid}/resource-requests
  events: [EventResourceAllocated]
  data: [resource_allocation, function_booking]
  rules: ["uses M09 pools; outsourced AV creates M21 requisition", "staff headcount becomes labor demand for M27/M62 roster"]
  security: standard
  failure_cases: ["pool exhausted -> rental option or infeasible"]
  finance_report_effect: Equipment/labor direct cost to event.
  i18n_a11y: Standard.
  acceptance: "Reserving 2 projectors and 4 servers for Lunch reduces pools and creates labor demand."
  dependency: M09, M21, M27.

- id: M12.F12.2.SF12.2.3
  name: Event proposal and contract link
  phase: 3
  release: R1
  actors: [sales_manager]
  screens: [SCR-SALES-proposal-editor]
  inputs: [event_id, proposal_id, contract_version_id]
  states: [linked]
  api: PUT /v1/properties/{pid}/events/{eid}/contract
  events: [EventContractLinked]
  data: [event_booking, contract_version, proposal]
  rules: ["event pricing baseline = accepted proposal; deltas only via change orders", "contract clauses (F&B minimum, attrition, cutoff) copied as structured terms"]
  security: standard
  failure_cases: ["contract amended -> event baseline version incremented"]
  finance_report_effect: Contracted baseline for actual-vs-contracted (SF12.5.3).
  i18n_a11y: Standard.
  acceptance: "Event shows contracted baseline totals equal to accepted proposal."
  dependency: M10.
```

### F12.3 BEO, revisions and change orders

```yaml
- id: M12.F12.3.SF12.3.1
  name: Create BEO
  phase: 3
  release: R1
  actors: [catering_manager, sales_manager]
  screens: [SCR-EVT-beo-editor]
  inputs: [function_id, menu_items_or_packages, covers, service_style, bar_setup(hosted|cash|consumption), timings, layout, av, signage, parking_instructions, club_visit, dietary_counts, allergen_notes, billing_instructions]
  states: [draft, issued, customer_accepted, locked, executed, superseded]
  api: POST /v1/properties/{pid}/events/{eid}/beos
  events: [BeoIssued]
  data: [beo, beo_version, beo_line]
  rules: ["BEO v1 created from contract baseline", "customer view excludes cost and internal notes", "allergen notes structured (EU 14 / GCC list per M44 rule pack) plus free text"]
  security: allergen/dietary data about individuals limited to counts unless consented
  failure_cases: ["missing mandatory sections -> cannot issue"]
  finance_report_effect: BEO lines are the priced basis for event charges.
  i18n_a11y: BEO printable EN/AR; kitchen print large type option.
  acceptance: "BEO v1 for G1 lists lunch 80 covers (3 vegan, 2 nut-allergy), hosted bar, club visit, AV and parking instructions."
  dependency: M16, M13, M15, M17.

- id: M12.F12.3.SF12.3.2
  name: BEO revision and versioning
  phase: 3
  release: R1
  actors: [catering_manager, event_organizer]
  screens: [SCR-EVT-beo-editor, SCR-EVT-beo-diff, SCR-CORP-itinerary]
  inputs: [beo_id, changes, reason, change_order_id]
  states: [draft_revision, issued, customer_accepted, superseded]
  api: POST /v1/properties/{pid}/beos/{beoid}/versions
  events: [BeoRevised]
  data: [beo_version, beo_line, change_order]
  rules: ["each revision is new immutable beo_version with diff", "priced changes require linked change order", "revisions after lock (D-216) need events manager approval"]
  security: audit actor/time/diff
  failure_cases: ["concurrent edit -> optimistic lock conflict with merge view"]
  finance_report_effect: Revised priced basis.
  i18n_a11y: Diff readable (added/removed labelled in text, not colour only).
  acceptance: "Revising covers 80->85 produces v2 with diff and pending change order."
  dependency: SF12.3.4.

- id: M12.F12.3.SF12.3.3
  name: Propagate BEO revisions to kitchen, bar and departments
  phase: 3
  release: R1
  actors: [executive_chef, fnb_manager, bartender, parking_attendant, club_host, housekeeping_supervisor]
  screens: [SCR-KDS-event-orders, SCR-CAT-production-plan, SCR-STF-beo-inbox]
  inputs: [beo_version_id, department]
  states: [sent, delivered, acknowledged, overdue]
  api: POST /v1/properties/{pid}/beo-versions/{vid}/acknowledgments
  events: [BeoVersionDispatched, BeoVersionAcknowledged, BeoAcknowledgmentOverdue]
  data: [beo_acknowledgment, production_plan, ingredient_reservation]
  rules: ["revision updates M16 production plan and ingredient reservations, M13 hosted-bar tab limits, M17 permit counts, M15 pass counts automatically", "each department must acknowledge within SLA; overdue escalates to duty_manager", "kitchen sees only latest version with changes highlighted; older versions read-only"]
  security: departments see their sections only
  failure_cases: ["downstream consumer error -> inbox retry and dead-letter to exception queue; BEO marked 'not propagated'"]
  finance_report_effect: none directly; drives cost reservations.
  i18n_a11y: Kitchen screens large text; audible alert optional.
  acceptance: "AT-G02.4 - BEO v2 (85 covers, bar extended 1h) updates the production plan, ingredient reservations and hosted-bar limit and requires chef and bar acknowledgments; a missing acknowledgment escalates after SLA."
  dependency: M16, M13, M14, M17, M15.

- id: M12.F12.3.SF12.3.4
  name: Event change orders
  phase: 3
  release: R1
  actors: [sales_manager, catering_manager, event_organizer, corporate_approver]
  screens: [SCR-EVT-change-order, SCR-CORP-change-order]
  inputs: [event_id, lines(add/remove/modify), price_delta, tax_delta, capacity_components, customer_acceptance]
  states: [draft, sent, accepted, rejected, applied, void]
  api: POST /v1/properties/{pid}/events/{eid}/change-orders
  events: [ChangeOrderAccepted, ChangeOrderApplied]
  data: [change_order, beo_version, composite_hold_component]
  rules: ["capacity changes amend composite atomically (SF09.3.4)", "customer acceptance (portal or signature) required for priced changes unless on-site authorized organizer", "PO ceiling checked"]
  security: step-up for acceptance above threshold
  failure_cases: ["capacity unavailable -> change order rejected before sending", "PO exceeded -> approval path"]
  finance_report_effect: Delta added to contracted baseline; reconciliation shows each change order.
  i18n_a11y: Standard.
  acceptance: "Change order +5 lunches and +5 parking accepted in portal updates BEO, capacity and baseline totals once."
  dependency: M09, M10.

- id: M12.F12.3.SF12.3.5
  name: Final guarantee and dietary aggregation
  phase: 3
  release: R1
  actors: [event_organizer, catering_manager]
  screens: [SCR-CORP-guarantee, SCR-EVT-guarantee]
  inputs: [function_id, guaranteed_covers, dietary_counts]
  states: [open, guaranteed, locked]
  api: PUT /v1/properties/{pid}/functions/{fid}/guarantee
  events: [GuaranteeSubmitted]
  data: [cover_guarantee, beo_version]
  rules: ["guarantee cutoff per contract (e.g. 72h); after cutoff decreases not accepted, increases subject to capacity", "missing guarantee at cutoff -> last expected covers become guarantee"]
  security: organizer scope
  failure_cases: ["late decrease -> recorded as request, billed at guarantee"]
  finance_report_effect: Guarantee is billing floor (SF16.5.2).
  i18n_a11y: Standard.
  acceptance: "Decrease after cutoff is not applied to billing floor."
  dependency: M16 SF16.2.2.
```

### F12.4 Event day

```yaml
- id: M12.F12.4.SF12.4.1
  name: Attendee registration and check-in
  phase: 3
  release: R1
  actors: [event_organizer, front_desk_agent]
  screens: [SCR-EVT-attendee-checkin, SCR-STF-attendee-scan]
  inputs: [attendee_name, email_optional, badge_code, dietary_need_optional, consent_flags]
  states: [registered, checked_in, no_show]
  api: POST /v1/properties/{pid}/events/{eid}/attendees/{atid}/check-in
  events: [AttendeeCheckedIn]
  data: [attendee]
  rules: ["badge QR is an event pass, not authentication for any guest account", "attendee count feeds actuals", "offline scan queued with dedup by badge_code"]
  security: attendee PII minimal; retention per M02 event purpose
  failure_cases: ["duplicate scan -> idempotent", "offline -> local list with later sync"]
  finance_report_effect: Actual attendance for per-person charges.
  i18n_a11y: Scanner has manual search fallback.
  acceptance: "Scanning the same badge twice counts one attendee; offline scans sync without duplicates."
  dependency: M18 offline sync patterns.

- id: M12.F12.4.SF12.4.2
  name: Hosted bar tab linkage
  phase: 3
  release: R1
  actors: [bartender, catering_manager]
  screens: [SCR-POS-hosted-tab]
  inputs: [function_id, tab_limit_amount_or_time, eligible_items, overflow_rule(stop|cash|approve)]
  states: [open, limit_reached, closed]
  api: POST /v1/properties/{pid}/functions/{fid}/hosted-tab
  events: [HostedTabOpened, HostedTabLimitReached, HostedTabClosed]
  data: [pos_check, beo_version]
  rules: ["tab limits from latest BEO", "consumption posts to event master account once at close (INV-FOL-1)", "overflow rule enforced by POS"]
  security: bartender cannot raise limit; manager approval
  failure_cases: ["POS offline -> tab continues locally with limit enforcement on cached limit"]
  finance_report_effect: Beverage revenue to event; stock depletion via M14.
  i18n_a11y: Standard.
  acceptance: "AT-G03.3 - hosted bar with 600 OMR limit stops at limit, posts once to master account and depletes recipe stock."
  dependency: M13, M14.

- id: M12.F12.4.SF12.4.3
  name: Event-day live consumption and incidents
  phase: 3
  release: R1
  actors: [catering_manager, duty_manager]
  screens: [SCR-STF-event-live]
  inputs: [function_id, consumption_items, incident_type]
  states: [recorded]
  api: POST /v1/properties/{pid}/functions/{fid}/consumptions
  events: [EventConsumptionRecorded]
  data: [catering_actual, change_order]
  rules: ["on-site additions require organizer authorization captured (signature or authorized contact)", "incidents link to M42/M55"]
  security: staff scope
  failure_cases: ["no authorization -> charge pending approval"]
  finance_report_effect: Actuals for reconciliation.
  i18n_a11y: Mobile large touch targets.
  acceptance: "An extra coffee break authorized on-site appears in reconciliation with authorizer."
  dependency: M16.
```

### F12.5 Billing and post-event reconciliation

```yaml
- id: M12.F12.5.SF12.5.1
  name: Master account and routing rules
  phase: 3
  release: R1
  actors: [sales_manager, front_desk_agent, cashier]
  screens: [SCR-EVT-billing-setup, SCR-FD-folio]
  inputs: [event_id, master_payer, routing_rules(charge_type/date_range/cap -> master|individual)]
  states: [open, closed]
  api: PUT /v1/properties/{pid}/events/{eid}/billing
  events: [MasterAccountOpened, RoutingRuleApplied]
  data: [master_account, routing_rule_ref, folio]
  rules: ["common setup - room and tax to master, incidentals to individual", "routing by rule at posting time; rule changes apply forward only", "attendee sees only individual folio"]
  security: routing changes audited
  failure_cases: ["charge without matching rule -> individual folio by default and flagged"]
  finance_report_effect: Determines AR (master) vs guest settlement.
  i18n_a11y: Standard.
  acceptance: "Room nights post to master; minibar posts to individual; parking per contract posts to master once."
  dependency: M08.

- id: M12.F12.5.SF12.5.2
  name: Pro-forma invoice and deposits application
  phase: 3
  release: R1
  actors: [ar_clerk, sales_manager]
  screens: [SCR-EVT-proforma]
  inputs: [event_id]
  states: [draft, issued]
  api: POST /v1/properties/{pid}/events/{eid}/proforma
  events: [ProformaIssued]
  data: [event_booking, beo_version, change_order]
  rules: ["pro-forma is not a tax invoice", "shows deposits received and balance"]
  security: standard
  failure_cases: ["tax pack unverified -> estimate label"]
  finance_report_effect: none.
  i18n_a11y: Bilingual PDF.
  acceptance: "Pro-forma equals baseline plus accepted change orders minus deposits."
  dependency: M08.

- id: M12.F12.5.SF12.5.3
  name: Post-event reconciliation actual vs contracted
  phase: 3
  release: R1
  actors: [catering_manager, ar_clerk, event_organizer]
  screens: [SCR-EVT-reconciliation, SCR-CORP-event-reconciliation]
  inputs: [event_id, actual_covers, actual_consumption, attrition_result, disputes]
  states: [draft, sent_to_customer, agreed, disputed, finalized]
  api: POST /v1/properties/{pid}/events/{eid}/reconciliation
  events: [EventReconciliationFinalized]
  data: [event_reconciliation, catering_actual, change_order, folio_charge]
  rules: ["line billing = per contract - F&B billed at max(guarantee, actual); hosted bar actual; attrition per SF12.1.4", "every variance explained with source", "finalize posts adjustments as new charges/credits, never edits"]
  security: finalize requires ar_clerk + catering_manager
  failure_cases: ["open POS checks for event -> cannot finalize"]
  finance_report_effect: Final event revenue; variance report.
  i18n_a11y: Standard.
  acceptance: "AT-G02.5 - reconciliation for G1 shows contracted vs actual per line; billing floor applied; totals match master account."
  dependency: M16, M13, M17, M15.

- id: M12.F12.5.SF12.5.4
  name: Final invoice and transfer to AR
  phase: 3
  release: R1
  actors: [ar_clerk]
  screens: [SCR-FIN-invoice-issue]
  inputs: [master_account_id, customer_po_id]
  states: [ready, issued, transferred_to_ar]
  api: POST /v1/properties/{pid}/master-accounts/{mid}/invoice
  events: [InvoiceIssued, FolioTransferredToAr]
  data: [invoice, master_account, ar_account]
  rules: ["tax invoice per M44/M38 rule pack", "PO number mandatory if agreement requires", "credit notes for corrections"]
  security: finance scope
  failure_cases: ["tax invoice rule pack unverified -> blocked or manual review per M44 gate"]
  finance_report_effect: AR receivable; revenue recognized per posting date.
  i18n_a11y: Bilingual.
  acceptance: "Final invoice issued once and appears in AR aging and corporate portal."
  dependency: M08, M20, M38.

- id: M12.F12.5.SF12.5.5
  name: Event P&L
  phase: 4
  release: R1
  actors: [catering_manager, financial_controller, gm]
  screens: [SCR-MGT-event-pnl]
  inputs: [event_id]
  states: [estimate, reconciled]
  api: GET /v1/properties/{pid}/events/{eid}/pnl
  events: [none]
  data: [event_reconciliation, stock_ledger_entry, journal_entry]
  rules: ["revenue by department; food/bev cost from actual issues to BEO; labor from attributed hours; commissions", "estimate until period close and costs posted"]
  security: margin visible to management roles
  failure_cases: ["labor not attributed -> estimate label"]
  finance_report_effect: Feeds M32 SF32.4.6 event margin.
  i18n_a11y: Table alternative.
  acceptance: "G1 event P&L drills from margin to stock issues and change orders."
  dependency: M19, M27, M14.
```

**M12 key invariants:** block hold is M03 inventory (no parallel counts); each BEO revision is immutable and must be acknowledged by every affected department; priced changes only via change orders; event charges post once via routing; reconciliation adjusts with new entries.

**M12 module acceptance:** AT-G02.1–G02.5, AT-G03.3, AT-G20.10.

**M12 open decisions**
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-215 | Allergen list standard per market (EU 14, GCC, Canada priority allergens) | executive_chef / compliance_officer | Use M44 rule pack; default union list with free text |
| D-216 | BEO lock time before event | catering_manager | 72 h before first function |
| D-217 | BEO acknowledgment SLA and escalation route | fnb_manager | 4 business hours; escalate to duty_manager |
| D-218 | Default routing template (master vs individual) | financial_controller | Room+tax+event F&B+parking to master; incidentals individual |

---

## M13 — Bar and F&B POS

| Item | Value |
|---|---|
| Purpose | Multi-outlet point of sale for bars, restaurant, club and banquet bars: menus/modifiers/happy hour, tables and tabs, room/corporate/event charges, kitchen and bar display, split tender/tips/service charge, discount/void/comp approvals, shift close and an offline queue that never double-posts. |
| Build phases | 3 |
| Release | R1 |
| Bounded context | `outlet-pos` (schema `pos`) |
| Systems of record owned | `outlet`, `menu`, `menu_item`, `modifier_group`, `price_level`, `happy_hour_rule`, `pos_check`, `pos_check_line`, `pos_tender`, `pos_adjustment`, `pos_shift`, `pos_offline_batch`, `kitchen_ticket`, `pos_terminal` |
| Upstream | M01 (device identity), M02 (staff PIN/badge + role), M04/M44 (tax/service charge rules), M09 (tables), M12 (hosted tabs), M14 (recipe mapping), M28 (terminals/tender), M05 (in-house guest lookup), M30 (points redemption), M57 (restaurant reservations, allergens) |
| Downstream | M08 (room/master charges), M14 (theoretical depletion), M19 (sales/tender journal), M27 (tip distribution to payroll), M60 (void/comp controls), M32 (outlet P&L), M30 (earn) |

**Device honesty:** POS runs as the MetriStay staff app on tablets/terminals. Card-present payment terminals, receipt/kitchen printers and cash drawers are connected through M28/M64 adapters; every model is `unverified-assumption` until a pilot device passes the test plan in `docs/09`. No terminal vendor is claimed as certified. Fiscal-device requirements, where a market has them, come from M44 (D-221).

### F13.1 Menus, modifiers and pricing

```yaml
- id: M13.F13.1.SF13.1.1
  name: Multi-outlet menus and items
  phase: 3
  release: R1
  actors: [fnb_manager, property_admin]
  screens: [SCR-ADM-menu-editor]
  inputs: [outlet_id, menu_name, categories, item_name_en_ar, price_by_price_level, tax_category, revenue_center, recipe_id, allergen_tags, image_media_id, availability_schedule]
  states: [draft, published, archived]
  api: POST /v1/properties/{pid}/outlets/{oid}/menus ; POST /v1/properties/{pid}/outlets/{oid}/menus/{mid}/publish
  events: [MenuPublished]
  data: [outlet, menu, menu_item, price_level]
  rules: ["publish creates immutable menu version; checks reference menu_version_id", "items without recipe mapping allowed but flagged for depletion coverage", "tax category mandatory"]
  security: fnb_manager; price change audit
  failure_cases: ["publish while terminals offline -> they keep prior version until sync, labelled"]
  finance_report_effect: Revenue center mapping to GL (M19).
  i18n_a11y: Item names EN/AR; allergen icons with text.
  acceptance: "Publishing bar menu v3 reaches online terminals within 60 s; offline terminal shows v2 and version badge."
  dependency: M14 recipes; M39 media.

- id: M13.F13.1.SF13.1.2
  name: Modifiers and combos
  phase: 3
  release: R1
  actors: [fnb_manager]
  screens: [SCR-ADM-modifier-editor]
  inputs: [modifier_group, min_select, max_select, price_delta, recipe_delta_lines]
  states: [active, archived]
  api: POST /v1/properties/{pid}/outlets/{oid}/modifier-groups
  events: [ModifierGroupChanged]
  data: [modifier_group, recipe_line]
  rules: ["modifiers can adjust recipe depletion (e.g. double shot)", "required modifier groups enforced at order entry"]
  security: standard
  failure_cases: ["missing required modifier -> cannot send"]
  finance_report_effect: Modifier revenue and depletion.
  i18n_a11y: Standard.
  acceptance: "A 'double' modifier depletes 60 ml instead of 30 ml of the spirit."
  dependency: M14.

- id: M13.F13.1.SF13.1.3
  name: Happy hour and time-based pricing
  phase: 3
  release: R1
  actors: [fnb_manager, revenue_manager]
  screens: [SCR-ADM-happy-hour]
  inputs: [outlet_id, days, start_time, end_time, item_filter, price_rule(percent|fixed)]
  states: [scheduled, active, expired]
  api: PUT /v1/properties/{pid}/outlets/{oid}/happy-hours
  events: [HappyHourRuleChanged]
  data: [happy_hour_rule]
  rules: ["price determined at line order time in property time zone", "not stackable with manual discount unless configured", "M44 rule pack may restrict alcohol promotions -> rule blocked when pack says so or unverified in that market"]
  security: standard
  failure_cases: ["terminal clock skew -> server time authoritative on sync; discrepancies flagged"]
  finance_report_effect: Promo discount reported separately from gross.
  i18n_a11y: Standard.
  acceptance: "Line rung at 17:59 is happy-hour priced; 18:01 is regular; alcohol promotion in a market where the pack forbids it cannot be activated."
  dependency: M44.

- id: M13.F13.1.SF13.1.4
  name: Tax and service charge configuration
  phase: 3
  release: R1
  actors: [financial_controller, property_admin]
  screens: [SCR-ADM-outlet-tax]
  inputs: [outlet_id, tax_category_map, service_charge_percent, service_charge_taxable, inclusive_or_exclusive]
  states: [draft, active]
  api: PUT /v1/properties/{pid}/outlets/{oid}/charges-config
  events: [OutletChargesConfigChanged]
  data: [outlet, rule_pack]
  rules: ["tax engine from M44/M38, effective-dated", "service charge distinct from tip", "rounding per currency minor units (OMR 3)"]
  security: finance roles only
  failure_cases: ["rule pack unverified -> receipts labelled and filing blocked per M44"]
  finance_report_effect: Tax liability and service charge revenue/liability per policy.
  i18n_a11y: Receipt shows tax labels localized.
  acceptance: "A 10.000 OMR bill with 10% service and 5% VAT computes to jurisdiction fixture totals."
  dependency: M44, M38.

- id: M13.F13.1.SF13.1.5
  name: Item availability and 86 list
  phase: 3
  release: R1
  actors: [shift_chef, bartender, fnb_manager]
  screens: [SCR-KDS-86-list, SCR-POS-order]
  inputs: [menu_item_id, status(available|limited|86), remaining_count]
  states: [available, limited, unavailable]
  api: PUT /v1/properties/{pid}/outlets/{oid}/items/{iid}/availability
  events: [MenuItemAvailabilityChanged]
  data: [menu_item]
  rules: ["86 propagates to POS, guest in-room dining and guest app menus", "limited count decrements on send"]
  security: kitchen/bar roles
  failure_cases: ["offline terminal sells 86 item -> flagged at sync for manager"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Marking an item 86 on KDS removes it from guest app ordering within 30 s."
  dependency: M18, M57.
```

### F13.2 Orders, tables, tabs and kitchen display

```yaml
- id: M13.F13.2.SF13.2.1
  name: Open table or tab
  phase: 3
  release: R1
  actors: [server, bartender, club_host]
  screens: [SCR-POS-table-plan, SCR-POS-tabs]
  inputs: [outlet_id, table_id_or_tab_name, covers, guest_link(reservation_id|member_id|event_function_id), preauth_payment_token]
  states: [open, sent, partially_paid, paid, closed, void]
  api: POST /v1/properties/{pid}/outlets/{oid}/checks
  events: [PosCheckOpened]
  data: [pos_check, resource_allocation]
  rules: ["check id is client-generated UUIDv7 for offline safety", "bar tabs may hold card preauth via M28 token", "linking in-house guest verifies name+room and charge privilege"]
  security: staff PIN/badge + device identity; server sees own checks unless manager
  failure_cases: ["guest charge privilege off -> room charge tender disabled"]
  finance_report_effect: none until closed.
  i18n_a11y: Large touch targets; RTL layout.
  acceptance: "Opening a tab offline yields a UUID check that syncs without collision."
  dependency: M09, M05.

- id: M13.F13.2.SF13.2.2
  name: Order entry and course firing
  phase: 3
  release: R1
  actors: [server, bartender]
  screens: [SCR-POS-order]
  inputs: [check_id, items, modifiers, seat, course, allergen_flag_per_line, notes]
  states: [pending, sent, prepared, served]
  api: POST /v1/properties/{pid}/checks/{chk}/lines
  events: [PosLinesSent]
  data: [pos_check_line, kitchen_ticket]
  rules: ["allergen flag forces kitchen acknowledgment on ticket (M57 SF57.1.4)", "sent lines immutable; changes are voids", "responsible-service gate check (SF13.2.5)"]
  security: server scope
  failure_cases: ["printer/KDS offline -> route to backup station, alert"]
  finance_report_effect: none until close.
  i18n_a11y: Standard.
  acceptance: "Line flagged nut allergy shows on KDS with required acknowledgment before bump."
  dependency: M57.

- id: M13.F13.2.SF13.2.3
  name: Kitchen and bar display
  phase: 3
  release: R1
  actors: [shift_chef, bartender]
  screens: [SCR-KDS-station]
  inputs: [station_id, ticket_id, action(start|bump|recall)]
  states: [new, in_progress, ready, bumped, recalled]
  api: POST /v1/properties/{pid}/kitchen-tickets/{tid}/actions
  events: [KitchenTicketBumped]
  data: [kitchen_ticket]
  rules: ["ticket timing KPIs recorded", "event/BEO orders appear with BEO version badge", "local LAN relay when internet down (on-prem or edge relay)"]
  security: station device identity
  failure_cases: ["KDS failure -> printer fallback; manual ticket mode"]
  finance_report_effect: Ticket times for M57 analytics.
  i18n_a11y: High-contrast, large fonts; bilingual item names.
  acceptance: "With internet down, tickets still flow from POS to KDS on local network and times sync later."
  dependency: M64 local relay.

- id: M13.F13.2.SF13.2.4
  name: Transfer, merge and split checks
  phase: 3
  release: R1
  actors: [server, bartender, fnb_manager]
  screens: [SCR-POS-split]
  inputs: [check_ids, split_mode(by_seat|by_item|evenly|amount), target_table]
  states: [split, merged, transferred]
  api: POST /v1/properties/{pid}/checks/{chk}/split
  events: [PosCheckSplit, PosCheckTransferred]
  data: [pos_check, pos_check_line]
  rules: ["sum of split checks equals original to minor unit; rounding remainder on last check", "transfer between servers audited"]
  security: transfer to other server requires both PINs or manager
  failure_cases: ["partially paid check split -> only unpaid lines"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Even split of 10.001 OMR into 3 yields 3.334+3.334+3.333."
  dependency: none.

- id: M13.F13.2.SF13.2.5
  name: Age and responsible-service gate
  phase: 3
  release: R1
  actors: [bartender, server, fnb_manager]
  screens: [SCR-POS-order]
  inputs: [check_id, age_verified_flag, refusal_reason]
  states: [not_required, verified, refused]
  api: POST /v1/properties/{pid}/checks/{chk}/age-verification
  events: [ResponsibleServiceRefusal]
  data: [pos_check]
  rules: ["alcohol items require verification per outlet and M44 rule pack; where alcohol sale is not permitted for the property, alcohol items cannot be published", "refusal logged without storing ID image"]
  security: no ID images in POS
  failure_cases: ["pack unverified -> conservative - require verification"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Alcohol line cannot be sent until the age prompt is answered; refusal logged."
  dependency: M44, M57 SF57.1.5.
```

### F13.3 Settlement and charge destinations

```yaml
- id: M13.F13.3.SF13.3.1
  name: Split tender
  phase: 3
  release: R1
  actors: [server, bartender, cashier]
  screens: [SCR-POS-payment]
  inputs: [check_id, tenders(type/amount/reference)]
  states: [open, partially_paid, paid]
  api: POST /v1/properties/{pid}/checks/{chk}/tenders
  events: [PosTenderApplied, PosCheckClosed]
  data: [pos_tender, pos_check]
  rules: ["tender types - cash, card (M28), room charge, master/event, corporate direct bill, points (M30), voucher (M54)", "sum tenders = check total + tip; overtender only cash with change", "each tender idempotent by client tender_id"]
  security: card via M28 terminal/token only; cash tender tied to open shift
  failure_cases: ["card approved but POS crash -> terminal reconciliation matches by tender_id", "points service unavailable -> points tender disabled"]
  finance_report_effect: Tender journal; points liability movement.
  i18n_a11y: Standard.
  acceptance: "A check paid 5 cash + 7 card + 3 room charge closes once; replaying the card callback creates no duplicate."
  dependency: M28, M30, M54.

- id: M13.F13.3.SF13.3.2
  name: Room charge to guest folio
  phase: 3
  release: R1
  actors: [server, bartender]
  screens: [SCR-POS-room-charge]
  inputs: [check_id, room_number, guest_last_name, signature_capture]
  states: [requested, posted, rejected]
  api: POST /v1/properties/{pid}/folios/{fid}/charges
  events: [FolioChargePosted]
  data: [folio_charge, pos_check]
  rules: ["post once with unique (source_type=pos_check, source_id) (INV-FOL-1)", "requires in-house status and charge privilege", "offline room charge queued with credit-limit risk flag"]
  security: name match required; signature image stored with check
  failure_cases: ["guest checked out while offline -> charge goes to exception queue for late-charge handling"]
  finance_report_effect: Outlet revenue recognized; guest ledger debit.
  i18n_a11y: Standard.
  acceptance: "AT-G03.4 - bar charge to room posts exactly once even after offline replay."
  dependency: M08.

- id: M13.F13.3.SF13.3.3
  name: Corporate and event master charges
  phase: 3
  release: R1
  actors: [bartender, catering_manager]
  screens: [SCR-POS-master-charge]
  inputs: [check_id, master_account_id, function_id, authorized_contact]
  states: [posted, rejected]
  api: POST /v1/properties/{pid}/master-accounts/{mid}/charges
  events: [FolioChargePosted]
  data: [folio_charge, master_account]
  rules: ["only for events/accounts with active routing permitting outlet charges", "hosted tab closes to master (SF12.4.2)"]
  security: authorized contact recorded
  failure_cases: ["master account closed -> rejected"]
  finance_report_effect: Event revenue.
  i18n_a11y: Standard.
  acceptance: "Hosted bar close posts one master account charge with function reference."
  dependency: M12.

- id: M13.F13.3.SF13.3.4
  name: Tips and service charge distribution
  phase: 3
  release: R1
  actors: [fnb_manager, payroll_officer]
  screens: [SCR-POS-tips, SCR-FIN-tip-pool]
  inputs: [shift_id, tip_amounts, pool_rules, service_charge_distribution_policy]
  states: [collected, pooled, approved, exported_to_payroll]
  api: POST /v1/properties/{pid}/tip-pools/{tpid}/approve
  events: [TipPoolApproved]
  data: [pos_tender, pos_shift]
  rules: ["tips are liabilities to staff, not revenue", "distribution per policy and M44/M38 payroll rules", "card tips paid via payroll or cash per policy"]
  security: individual tip amounts visible to employee, manager and payroll only
  failure_cases: ["policy unverified in market -> export blocked for review"]
  finance_report_effect: Tip liability then payroll (M27).
  i18n_a11y: Standard.
  acceptance: "Shift tips 25.000 OMR pooled by hours and exported to payroll once."
  dependency: M27.

- id: M13.F13.3.SF13.3.5
  name: Receipts and fiscal output
  phase: 3
  release: R1
  actors: [server, guest]
  screens: [SCR-POS-receipt, SCR-GST-receipts]
  inputs: [check_id, delivery(print|email|app)]
  states: [issued, reissued]
  api: POST /v1/properties/{pid}/checks/{chk}/receipt
  events: [PosReceiptIssued]
  data: [pos_check]
  rules: ["receipt/tax-invoice format per M44 rule pack", "fiscal device integration only where required and certified (D-221)", "reprint marked copy"]
  security: email requires consent for marketing, not for transactional receipt
  failure_cases: ["fiscal device offline -> sale blocked or queued per market rule"]
  finance_report_effect: none.
  i18n_a11y: Bilingual receipts.
  acceptance: "Reprint shows 'COPY' and same totals."
  dependency: M44.
```

### F13.4 Discounts, voids, comps and refunds

```yaml
- id: M13.F13.4.SF13.4.1
  name: Discounts with limits
  phase: 3
  release: R1
  actors: [server, fnb_manager]
  screens: [SCR-POS-discount]
  inputs: [check_or_line_id, discount_code, percent_or_amount, reason]
  states: [requested, approved, applied, rejected]
  api: POST /v1/properties/{pid}/checks/{chk}/adjustments
  events: [PosAdjustmentApplied]
  data: [pos_adjustment]
  rules: ["role-based limits per outlet; above limit needs manager PIN", "discount types mapped to GL contra-revenue"]
  security: approver != requester
  failure_cases: ["offline approval -> manager PIN cached hash with 24h validity"]
  finance_report_effect: Discounts report; M60 outliers.
  i18n_a11y: Standard.
  acceptance: "Server limited to 10%; 20% requires manager PIN and logs both identities."
  dependency: M60.

- id: M13.F13.4.SF13.4.2
  name: Void before and after send
  phase: 3
  release: R1
  actors: [server, fnb_manager]
  screens: [SCR-POS-void]
  inputs: [line_id, reason_code, waste_flag]
  states: [voided]
  api: POST /v1/properties/{pid}/checks/{chk}/lines/{lid}/void
  events: [PosLineVoided]
  data: [pos_adjustment, pos_check_line]
  rules: ["pre-send void by server; post-send void needs manager", "post-send void asks whether item was prepared - prepared -> M14 waste entry, not stock return", "void on closed check is refund (SF13.4.4)"]
  security: approval PIN
  failure_cases: ["reason missing -> blocked"]
  finance_report_effect: Void report; waste cost.
  i18n_a11y: Standard.
  acceptance: "Post-send void of prepared cocktail creates waste ledger entry and no stock credit."
  dependency: M14 SF14.3.6.

- id: M13.F13.4.SF13.4.3
  name: Comps with approval
  phase: 3
  release: R1
  actors: [fnb_manager, duty_manager]
  screens: [SCR-POS-comp]
  inputs: [line_or_check, comp_reason(service_recovery|marketing|staff|owner), guest_case_id]
  states: [requested, approved, applied]
  api: POST /v1/properties/{pid}/checks/{chk}/comps
  events: [PosCompApplied]
  data: [pos_adjustment]
  rules: ["comp retains cost depletion; revenue zero with comp expense account", "service recovery comps link to M55 case and caps"]
  security: approval by role limit
  failure_cases: ["cap exceeded -> escalate"]
  finance_report_effect: Comp expense by reason.
  i18n_a11y: Standard.
  acceptance: "Service-recovery comp is linked to the guest case and reported in recovery cost."
  dependency: M55, M60.

- id: M13.F13.4.SF13.4.4
  name: Refunds on closed checks
  phase: 3
  release: R1
  actors: [fnb_manager, cashier]
  screens: [SCR-POS-refund]
  inputs: [check_id, lines, tender_to_refund, reason]
  states: [requested, approved, refunded, failed]
  api: POST /v1/properties/{pid}/checks/{chk}/refunds
  events: [PosRefundIssued]
  data: [pos_adjustment, pos_tender]
  rules: ["refund to original tender; room charge refund = folio reversal", "card refund via M28 idempotent", "points reversal via M30"]
  security: manager approval; dual control above threshold
  failure_cases: ["PSP refund pending -> status pending, not paid"]
  finance_report_effect: Reverses revenue/tax; refund journal.
  i18n_a11y: Standard.
  acceptance: "AT-G07.2 - refund of a points-earning check reverses charge and points once."
  dependency: M28, M30.

- id: M13.F13.4.SF13.4.5
  name: Adjustment exception feed to revenue protection
  phase: 3
  release: R1
  actors: [financial_controller, pos_control_worker]
  screens: [SCR-OPS-exception-queue]
  inputs: [business_date]
  states: [none]
  api: GET /v1/properties/{pid}/pos/adjustments
  events: [PosAdjustmentOutlierDetected]
  data: [pos_adjustment]
  rules: ["feeds M60 SF60.2.2 outlier rules", "no automated disciplinary action"]
  security: controller scope; employee privacy
  failure_cases: ["none beyond feed lag"]
  finance_report_effect: Control report.
  i18n_a11y: Standard.
  acceptance: "A server with 5x median voids is flagged for review with evidence."
  dependency: M60.
```

### F13.5 Shifts, day close and offline queue

```yaml
- id: M13.F13.5.SF13.5.1
  name: Shift open and float
  phase: 3
  release: R1
  actors: [bartender, cashier, fnb_manager]
  screens: [SCR-POS-shift]
  inputs: [terminal_id, drawer_id, opening_float_by_denomination]
  states: [open, suspended, closing, closed]
  api: POST /v1/properties/{pid}/pos-shifts
  events: [PosShiftOpened]
  data: [pos_shift]
  rules: ["one open shift per drawer", "float counted and confirmed by two people if policy"]
  security: PIN/badge
  failure_cases: ["drawer already open -> blocked"]
  finance_report_effect: Float movement from safe (M60).
  i18n_a11y: Standard.
  acceptance: "Opening second shift on same drawer is rejected."
  dependency: M60 SF60.1.1.

- id: M13.F13.5.SF13.5.2
  name: Blind shift close
  phase: 3
  release: R1
  actors: [bartender, cashier, fnb_manager]
  screens: [SCR-POS-shift-close]
  inputs: [shift_id, declared_cash_by_denomination, declared_card_batch_total]
  states: [closing, closed_balanced, closed_variance, reviewed]
  api: POST /v1/properties/{pid}/pos-shifts/{sid}/close
  events: [PosShiftClosed, PosShiftVarianceRaised]
  data: [pos_shift, pos_tender]
  rules: ["declaration before expected totals are shown (blind)", "variance beyond tolerance requires manager note", "open checks must be transferred before close"]
  security: expected totals hidden from closer until declared
  failure_cases: ["unsynced offline batch -> close pending sync"]
  finance_report_effect: Cash over/short account.
  i18n_a11y: Denomination entry keyboard friendly.
  acceptance: "Declaring 98.500 vs expected 100.000 creates variance -1.500 requiring manager review."
  dependency: M60.

- id: M13.F13.5.SF13.5.3
  name: Offline queue and sync
  phase: 3
  release: R1
  actors: [pos_sync_worker, bartender]
  screens: [SCR-POS-sync-status]
  inputs: [pos_offline_batch(checks/lines/tenders/adjustments with client ids and device sequence)]
  states: [queued, syncing, synced, conflict, rejected]
  api: POST /v1/properties/{pid}/pos/offline-batches
  events: [PosOfflineBatchSynced, PosOfflineConflictRaised]
  data: [pos_offline_batch, pos_check, pos_tender]
  rules: ["local encrypted store on device; queue bounded by max offline duration and amount (D-219)", "server applies by client ids idempotently; device sequence detects gaps", "card payments offline only via terminal store-and-forward if supported by PSP, else card tender disabled", "room charges offline queued with risk flag"]
  security: device keys; queue encrypted; remote wipe
  failure_cases: ["device lost before sync -> sequence gap alert and manual reconstruction", "server rejects line (menu version missing) -> conflict queue"]
  finance_report_effect: Late-synced sales posted to their original business date if day not closed; else to current with audit link.
  i18n_a11y: Offline banner and queue count visible.
  acceptance: "AT-G20.9 - 2 h network outage with 40 checks; after reconnection each check, tender and room charge posts exactly once; replaying the batch changes nothing."
  dependency: M64 offline policy.

- id: M13.F13.5.SF13.5.4
  name: Outlet day close and GL export
  phase: 3
  release: R1
  actors: [fnb_manager, night_auditor]
  screens: [SCR-POS-day-close]
  inputs: [outlet_id, business_date]
  states: [open, closing, closed, reopened]
  api: POST /v1/properties/{pid}/outlets/{oid}/day-close
  events: [OutletDayClosed]
  data: [pos_check, pos_shift, journal_entry]
  rules: ["all shifts closed, no open checks (or transferred), all offline batches synced or explicitly waived", "summarized sales/tax/tender/discount journal per M19 mapping", "reopen needs financial_controller and produces delta journal"]
  security: night_auditor
  failure_cases: ["device never synced -> close with waiver and exception case"]
  finance_report_effect: Outlet revenue, tax, tender journals; feeds night audit M08.
  i18n_a11y: Standard.
  acceptance: "AT-G08.1 - day close journal equals sum of checks by revenue center and tender."
  dependency: M08 night audit, M19.
```

**M13 key invariants:** a check posts to a folio once (INV-FOL-1); sent lines are never edited (void/refund only); prepared-then-voided items go to waste, never back to stock; tips are staff liabilities; offline replay is idempotent by client IDs.

**M13 module acceptance:** AT-G03.3–G03.4, AT-G07.2, AT-G08.1, AT-G20.9.

**M13 open decisions**
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-219 | Max offline duration/amount before POS locks card/room charge | fnb_manager / financial_controller | 8 h and 2,000 OMR-equivalent per terminal |
| D-220 | Pilot payment terminal and store-and-forward support | it_admin / PSP | Card tender disabled offline until PSP confirms |
| D-221 | Fiscal device/e-invoice requirements per market (e.g. Saudi e-invoicing, Portugal certified billing software) | compliance_officer | Treated as M44 rule-pack gate; `unverified-assumption` |
| D-222 | Tip pooling policy and payroll treatment | hr_officer / payroll_officer | Pool by hours worked; payroll payout |

---

## M14 — Bar and kitchen inventory (canonical stock ledger, shared with M50)

| Item | Value |
|---|---|
| Purpose | Item/SKU master with unit conversion, recipes/BOM/yield, stores and bins, lots with expiry and recall, the append-only stock ledger for every movement (receipt, transfer, issue, intact return, quarantine, waste, count adjustment, depletion), theoretical POS depletion vs blind count, shrinkage and cost/margin reporting. Guarantees that discarded food never returns to sellable stock. |
| Build phases | 3 (item master, recipes, ledger, depletion, counts) • 4 (costing/COGS GL, full M50 low-touch receiving integration) |
| Release | R1 |
| Bounded context | `stock` (schema `stk`) — **single canonical stock ledger for the suite**; M50 (receiving/issue/return/waste workflows), M16, M25, M56 and M58 write only through the `StockLedger` port |
| Systems of record owned | `stock_item`, `uom`, `uom_conversion`, `store`, `store_bin`, `stock_lot`, `stock_ledger_entry`, `recipe`, `recipe_line`, `stock_count`, `stock_count_line`, `recall_case`, `waste_record`, `par_level`; projection `stock_balance` |
| Upstream | M48 (vendor SKU → `stock_item` crosswalk), M21/M49 (PO), M50 (goods receipt, ASN evidence, issue/return/waste workflows), M13 (sales for depletion), M16/M12 (BEO issues), M44 (food-trace retention rules), M61 (food-safety checks) |
| Downstream | M19 (inventory, COGS, waste, shrinkage journals), M20 (3-way match quantities), M32 (food/bev cost %, outlet margin), M57 (allergens, batch trace), M61 (recall), M67 (food-waste measurement) |

### Canonical stock entities (other writers MUST reuse)

| Entity | Key fields | Notes |
|---|---|---|
| `stock_item` | `item_id, property_id, sku_code, name_en, name_ar, category, base_uom, stock_uom, purchase_uoms[], perishable, lot_tracked, expiry_tracked, allergen_tags[], storage_condition(ambient/chilled/frozen), costing_method, gs1_gtin?` | Hotel master item; vendor SKUs map via M48 crosswalk. |
| `uom` / `uom_conversion` | `uom_code, dimension(mass/volume/count/length)`; `item_id?, from_uom, to_uom, factor(decimal), source(standard/item_specific/vendor_pack), valid_from` | Item-specific conversions (1 case = 24 × 330 ml) override standard. |
| `store` | `store_id, property_id, department, type(main/outlet/kitchen/bar/catering/housekeeping/engineering/quarantine/waste), cost_center, is_sellable_source` | `quarantine` and `waste` stores have `is_sellable_source = false`. |
| `store_bin` | `bin_id, store_id, code, storage_condition, status(active/blocked)` | Physical location; FEFO picking by bin. |
| `stock_lot` | `lot_id, item_id, supplier_id, supplier_lot_code, internal_lot_code, received_at, manufacture_date?, expiry_date?, origin?, goods_receipt_id, status(available/on_hold/quarantine/recalled/expired/consumed/waste), temperature_evidence_ref?` | Created by receipt (M50) or production batch (M16/M57). Terminal statuses: `waste`, `consumed`. |
| `stock_ledger_entry` | `entry_id (UUIDv7), property_id, business_date, occurred_at, item_id, lot_id?, store_id, bin_id?, quantity (signed decimal, base_uom), stock_status(available/reserved/quarantine/waste/recalled), movement_type, unit_cost_minor, currency, source_type, source_id, source_line_id, idempotency_key (unique), reverses_entry_id?, reason_code?, evidence_refs[], actor_id, approved_by?` | Append-only; unique `idempotency_key`; `movement_type ∈ receipt, receipt_reversal, transfer_out, transfer_in, issue, intact_return, quarantine_in, quarantine_release, waste, rtv (return to vendor), count_adjustment, pos_depletion, production_consume, production_output, recall_hold, recall_release, cost_revaluation`. |
| `stock_balance` | `(item, lot, store, bin, stock_status) → quantity, value` | Projection rebuilt from ledger; never written directly. |

### F14.1 Item master, units and stores

```yaml
- id: M14.F14.1.SF14.1.1
  name: Stock item master
  phase: 3
  release: R1
  actors: [storekeeper, executive_chef, fnb_manager, procurement_officer]
  screens: [SCR-INV-item-list, SCR-INV-item-detail]
  inputs: [sku_code, name_en, name_ar, category, base_uom, stock_uom, perishable, lot_tracked, expiry_tracked, allergen_tags, storage_condition, par_min, par_max, gs1_gtin, vendor_sku_links]
  states: [draft, active, blocked, discontinued]
  api: POST /v1/properties/{pid}/stock-items
  events: [StockItemCreated, StockItemChanged]
  data: [stock_item, par_level]
  rules: ["perishable food items must be lot_tracked and expiry_tracked", "base_uom immutable once ledger entries exist (change via new item + transfer)", "vendor SKU mapping through M48 crosswalk; one vendor SKU maps to one stock_item"]
  security: storekeeper edits; cost fields finance-only
  failure_cases: ["duplicate GTIN -> blocked", "discontinue with balance -> blocked"]
  finance_report_effect: Category drives GL inventory/COGS accounts.
  i18n_a11y: EN/AR names; allergen text labels.
  acceptance: "Creating fresh tomatoes as perishable without lot tracking is rejected."
  dependency: M48 crosswalk.

- id: M14.F14.1.SF14.1.2
  name: Units of measure and conversions
  phase: 3
  release: R1
  actors: [storekeeper, fnb_manager]
  screens: [SCR-INV-uom]
  inputs: [item_id, from_uom, to_uom, factor, source]
  states: [active, superseded]
  api: POST /v1/properties/{pid}/stock-items/{iid}/uom-conversions
  events: [UomConversionChanged]
  data: [uom, uom_conversion]
  rules: ["cross-dimension conversion (count->mass) only item-specific", "factor change is effective-dated, not retroactive", "all ledger quantities stored in base_uom with original entered uom recorded"]
  security: audit
  failure_cases: ["circular or inconsistent conversions -> rejected"]
  finance_report_effect: Unit cost normalization.
  i18n_a11y: Localized unit labels (kg/كغ).
  acceptance: "Receiving 5 cases (24x330 ml) posts 39.6 L in base uom; recipe 30 ml depletes correctly."
  dependency: M48 pack data.

- id: M14.F14.1.SF14.1.3
  name: Stores and bins
  phase: 3
  release: R1
  actors: [storekeeper, property_admin]
  screens: [SCR-INV-stores]
  inputs: [store_name, type, department, cost_center, is_sellable_source, bins(code/condition)]
  states: [active, blocked, closed]
  api: POST /v1/properties/{pid}/stores ; POST /v1/properties/{pid}/stores/{sid}/bins
  events: [StoreChanged]
  data: [store, store_bin]
  rules: ["each property has at least one quarantine store and one waste store (non-sellable)", "closing a store requires zero balance"]
  security: property_admin
  failure_cases: ["configure waste store as sellable -> rejected"]
  finance_report_effect: Store cost center mapping.
  i18n_a11y: Standard.
  acceptance: "Attempt to set waste store is_sellable_source=true fails."
  dependency: none.

- id: M14.F14.1.SF14.1.4
  name: Par levels and reorder suggestions
  phase: 3
  release: R1
  actors: [storekeeper, fnb_manager, reorder_worker]
  screens: [SCR-INV-reorder]
  inputs: [item_id, store_id, par_min, par_max, lead_time_days, forecast_source(beo|pos_history)]
  states: [ok, below_min, suggested, requisitioned]
  api: GET /v1/properties/{pid}/stores/{sid}/reorder-suggestions
  events: [ReorderSuggested]
  data: [par_level, stock_balance]
  rules: ["available = available status minus reservations", "BEO ingredient reservations (M16) reduce available", "suggestion becomes M21/M49 requisition by user action"]
  security: standard
  failure_cases: ["stale count -> confidence low label"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Reserving ingredients for BEO drops available below min and suggests reorder."
  dependency: M21, M49.

- id: M14.F14.1.SF14.1.5
  name: Costing method and valuation
  phase: 4
  release: R1
  actors: [financial_controller]
  screens: [SCR-FIN-inventory-valuation]
  inputs: [costing_method(moving_average|fifo_by_lot), valuation_date]
  states: [configured, locked]
  api: GET /v1/properties/{pid}/inventory-valuation
  events: [InventoryValuationComputed]
  data: [stock_ledger_entry, stock_balance]
  rules: ["receipt cost = PO price adjusted by invoice match (cost_revaluation entry)", "method change only at period start with controller approval (D-223)"]
  security: finance only
  failure_cases: ["uninvoiced receipts -> valued at PO price flagged estimate"]
  finance_report_effect: Inventory asset balance and COGS basis for M19.
  i18n_a11y: Standard.
  acceptance: "Valuation equals sum of ledger quantity x cost and ties to GL inventory account."
  dependency: M19, M20.
```

### F14.2 Recipes, BOM and yield

```yaml
- id: M14.F14.2.SF14.2.1
  name: Recipes and BOM with sub-recipes
  phase: 3
  release: R1
  actors: [executive_chef, fnb_manager]
  screens: [SCR-INV-recipe-editor]
  inputs: [recipe_name, output_item_or_menu_item, portions, recipe_lines(item/qty/uom/sub_recipe), method_notes, version]
  states: [draft, approved, active, superseded]
  api: POST /v1/properties/{pid}/recipes ; POST /v1/properties/{pid}/recipes/{rid}/approve
  events: [RecipeApproved]
  data: [recipe, recipe_line]
  rules: ["versions immutable once active; checks and BEOs reference recipe version", "sub-recipe depth max 5; no cycles", "approval by executive_chef"]
  security: recipes confidential to F&B and finance
  failure_cases: ["missing conversion for line uom -> cannot approve"]
  finance_report_effect: Theoretical cost per portion.
  i18n_a11y: Standard.
  acceptance: "Mojito recipe v2 activated; checks after activation deplete v2 quantities."
  dependency: SF14.1.2.

- id: M14.F14.2.SF14.2.2
  name: Yield and trim factors
  phase: 3
  release: R1
  actors: [executive_chef]
  screens: [SCR-INV-yield-test]
  inputs: [item_id, as_purchased_qty, edible_qty, test_date, evidence_photo]
  states: [recorded, applied]
  api: POST /v1/properties/{pid}/stock-items/{iid}/yield-tests
  events: [YieldFactorUpdated]
  data: [recipe_line, stock_item]
  rules: ["recipe consumption uses as-purchased qty = edible qty / yield", "yield change versioned and dated"]
  security: standard
  failure_cases: ["yield > 1 -> rejected"]
  finance_report_effect: More accurate theoretical cost.
  i18n_a11y: Standard.
  acceptance: "Yield 0.8 on onions depletes 125 g for a 100 g recipe line."
  dependency: none.

- id: M14.F14.2.SF14.2.3
  name: POS item to recipe mapping
  phase: 3
  release: R1
  actors: [fnb_manager]
  screens: [SCR-INV-pos-mapping]
  inputs: [menu_item_id, modifier_id, recipe_id]
  states: [mapped, unmapped]
  api: PUT /v1/properties/{pid}/menu-items/{miid}/recipe
  events: [MenuItemRecipeMapped]
  data: [menu_item, recipe]
  rules: ["unmapped sold items reported as depletion coverage gap", "mapping change applies to future sales only"]
  security: standard
  failure_cases: ["none"]
  finance_report_effect: Coverage % shown on variance reports.
  i18n_a11y: Standard.
  acceptance: "Coverage report lists 3 unmapped items with sales quantity."
  dependency: M13.

- id: M14.F14.2.SF14.2.4
  name: Recipe costing and menu margin
  phase: 3
  release: R1
  actors: [executive_chef, fnb_manager, financial_controller]
  screens: [SCR-INV-recipe-cost]
  inputs: [recipe_id, cost_basis(latest|average)]
  states: [none]
  api: GET /v1/properties/{pid}/recipes/{rid}/cost
  events: [none]
  data: [recipe, recipe_line, stock_ledger_entry]
  rules: ["cost per portion and food-cost % vs menu price net of tax", "alert when cost % exceeds target"]
  security: cost visible to F&B management and finance
  failure_cases: ["missing cost -> estimate"]
  finance_report_effect: Menu engineering input.
  i18n_a11y: Standard.
  acceptance: "Recipe cost recalculates after a price revaluation."
  dependency: SF14.1.5.

- id: M14.F14.2.SF14.2.5
  name: Allergen roll-up
  phase: 3
  release: R1
  actors: [executive_chef]
  screens: [SCR-INV-recipe-editor]
  inputs: [recipe_id]
  states: [computed, chef_confirmed]
  api: GET /v1/properties/{pid}/recipes/{rid}/allergens
  events: [RecipeAllergensConfirmed]
  data: [recipe, stock_item]
  rules: ["allergens union of ingredients and sub-recipes; chef confirmation required before menu publish", "substitutions re-trigger confirmation (M57 SF57.2.3)"]
  security: standard
  failure_cases: ["ingredient allergen unknown -> 'may contain' warning"]
  finance_report_effect: none.
  i18n_a11y: Allergen labels bilingual.
  acceptance: "Adding pesto sub-recipe adds 'nuts' to the dish allergen list and blocks publish until confirmed."
  dependency: M57, M13.
```

### F14.3 Stock ledger and movements

```yaml
- id: M14.F14.3.SF14.3.1
  name: Append-only stock ledger port
  phase: 3
  release: R1
  actors: [stock_ledger_worker]
  screens: [SCR-INV-ledger]
  inputs: [movement_type, item_id, lot_id, store_id, bin_id, quantity, uom, stock_status, unit_cost, source_type, source_id, idempotency_key, evidence_refs]
  states: [posted, reversed]
  api: internal port StockLedger.post(entries[]) ; GET /v1/properties/{pid}/stock-ledger
  events: [StockLedgerEntriesPosted]
  data: [stock_ledger_entry, stock_balance, stock_lot]
  rules: ["INV-STK-1 and INV-STK-2", "entries in one call post atomically", "negative available balance per lot/store rejected (no negative food lot)", "duplicate idempotency_key returns original entries", "corrections only via reversing entry"]
  security: only service identities write; UPDATE/DELETE revoked at DB role level
  failure_cases: ["duplicate scan/webhook -> no-op", "insufficient lot quantity -> reject with available", "projection lag -> balance read shows as-of"]
  finance_report_effect: Source of inventory, COGS, waste and shrinkage journals.
  i18n_a11y: Ledger view accessible table.
  acceptance: "AT-G18.4 - replaying the same receipt event creates one ledger entry; attempting to issue more than lot balance is rejected."
  dependency: M50 uses this port.

- id: M14.F14.3.SF14.3.2
  name: Receipt posting interface
  phase: 3
  release: R1
  actors: [receiver, storekeeper]
  screens: [SCR-INV-receipt-post]
  inputs: [goods_receipt_id, lines(item/lot/qty/uom/cost/accepted_status)]
  states: [accepted_available, accepted_quarantine]
  api: POST /v1/properties/{pid}/stock-receipts
  events: [StockReceived]
  data: [stock_ledger_entry, stock_lot]
  rules: ["accepted qty -> receipt entry available; discrepant/temperature-breach qty -> quarantine store", "one ledger entry per accepted GR line (idempotency_key = gr_line_id)", "receipt workflow and evidence owned by M50 SF50.2.x"]
  security: receiver attestation required
  failure_cases: ["GR line reversed -> receipt_reversal entry"]
  finance_report_effect: Inventory debit / GRNI credit.
  i18n_a11y: Standard.
  acceptance: "Receipt of 120 kg vegetables with 5 kg damaged posts 115 available + 5 quarantine."
  dependency: M50 SF50.2.7.

- id: M14.F14.3.SF14.3.3
  name: Store transfers
  phase: 3
  release: R1
  actors: [storekeeper, bartender, shift_chef]
  screens: [SCR-INV-transfer, SCR-STF-transfer-scan]
  inputs: [from_store, to_store, lines(item/lot/qty)]
  states: [requested, dispatched, received, discrepancy]
  api: POST /v1/properties/{pid}/stock-transfers
  events: [StockTransferred]
  data: [stock_ledger_entry]
  rules: ["transfer_out and transfer_in paired; in-transit visible", "receiving store confirms qty; difference -> discrepancy case", "FEFO lot proposed"]
  security: both stores' roles
  failure_cases: ["receiving never confirmed -> ages in exception queue"]
  finance_report_effect: Cost center transfer.
  i18n_a11y: Scan with manual entry fallback.
  acceptance: "Main store -> bar transfer of 12 bottles creates paired entries; bar confirms 11 -> discrepancy 1."
  dependency: none.

- id: M14.F14.3.SF14.3.4
  name: Issue to outlet, kitchen or BEO
  phase: 3
  release: R1
  actors: [storekeeper, shift_chef, catering_manager]
  screens: [SCR-INV-issue, SCR-STF-issue-scan]
  inputs: [requesting_department, beo_version_id_or_job, lines(item/lot/qty)]
  states: [requested, picked, issued]
  api: POST /v1/properties/{pid}/stock-issues
  events: [StockIssued]
  data: [stock_ledger_entry, ingredient_reservation]
  rules: ["issue consumes ingredient_reservation for BEO first", "FEFO enforced unless override reason", "expired/on_hold/recalled lots cannot be issued"]
  security: requester and issuer recorded
  failure_cases: ["reserved lot recalled -> alternative lot proposed"]
  finance_report_effect: Cost moves to department/event cost.
  i18n_a11y: Standard.
  acceptance: "AT-G18.5 - issue for BEO G1 lunch consumes reservation and posts cost to event."
  dependency: M16, M50 SF50.3.2.

- id: M14.F14.3.SF14.3.5
  name: Intact return and quarantine return
  phase: 3
  release: R1
  actors: [shift_chef, storekeeper, executive_chef]
  screens: [SCR-INV-return, SCR-STF-return]
  inputs: [issue_ref, item_id, lot_id, qty, condition(intact_sealed|opened|served|temperature_unknown), photo, inspector_id]
  states: [returned_quarantine, inspected, released_available, moved_to_waste]
  api: POST /v1/properties/{pid}/stock-returns
  events: [StockReturnedToQuarantine, QuarantineReleased, QuarantineWasted]
  data: [stock_ledger_entry, stock_lot, waste_record]
  rules: ["every return from service areas lands in quarantine first", "release to available only if condition=intact_sealed, never issued to guest service, in date, cold chain evidenced, inspector != returner", "any other condition -> waste (terminal)", "original lot and expiry retained"]
  security: inspector role (executive_chef or designated) with step-up
  failure_cases: ["inspector same as returner -> blocked", "attempt to release opened item -> rejected"]
  finance_report_effect: Intact return reverses cost to department; waste posts waste expense.
  i18n_a11y: Condition choices with text + icons.
  acceptance: "AT-G18.6 - sealed unused cream returns via quarantine and is released by a different inspector; an opened tray returned from buffet is forced to waste and available stock does not increase."
  dependency: M50 SF50.3.4-3.5, M61.

- id: M14.F14.3.SF14.3.6
  name: Waste and disposal (no return to sellable stock)
  phase: 3
  release: R1
  actors: [shift_chef, bartender, storekeeper, fnb_manager]
  screens: [SCR-INV-waste, SCR-STF-waste]
  inputs: [item_id, lot_id, qty, store_id, reason(spoilage|breakage|expiry|overproduction|plate_waste|prepared_void|sampling|staff_meal|recall|temperature_breach), photo, disposal_method]
  states: [recorded, approved, disposed]
  api: POST /v1/properties/{pid}/waste-records
  events: [StockWasted, WasteDisposed]
  data: [waste_record, stock_ledger_entry]
  rules: ["waste moves qty to waste store with status waste; waste store is non-sellable", "no movement type exists from waste to available; reversal of a waste entry allowed only as error correction by financial_controller with evidence and within same business date, producing a reversing entry, never a release", "approval above value threshold", "food waste weight feeds M67"]
  security: approval PIN; photo evidence for high value
  failure_cases: ["API request of quarantine_release on waste lot -> 422 INV-STK-2", "duplicate waste scan -> idempotent"]
  finance_report_effect: Waste expense by reason and department; food waste KPI.
  i18n_a11y: Standard.
  acceptance: "AT-G18.7 - property-based test - no sequence of API calls increases available quantity of a lot after a waste entry, except a same-day reversing correction by controller that is fully audited."
  dependency: M50 SF50.3.5-3.9, M67.
```

### F14.4 Lots, expiry and recall

```yaml
- id: M14.F14.4.SF14.4.1
  name: Lot capture and traceability
  phase: 3
  release: R1
  actors: [receiver, storekeeper, shift_chef]
  screens: [SCR-INV-lot-detail]
  inputs: [supplier_lot_code, internal_lot_code, gs1_ai_data, manufacture_date, expiry_date, production_batch_inputs]
  states: [available, on_hold, quarantine, recalled, expired, consumed, waste]
  api: GET /v1/properties/{pid}/stock-lots/{lid}/trace
  events: [StockLotCreated]
  data: [stock_lot, stock_ledger_entry, production_batch]
  rules: ["production batches (M16/M57) record input lots -> output lot (one-up/one-down trace)", "trace query returns forward (events, checks, guests where feasible) and backward (supplier, GR)"]
  security: guest-level trace restricted to compliance_officer and executive_chef
  failure_cases: ["missing supplier lot -> internal lot with 'unverified supplier lot' flag"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Trace of chicken lot L-77 lists GR, 3 production batches, BEO G1 lunch and 2 restaurant days."
  dependency: M50 SF50.2.10, M57.

- id: M14.F14.4.SF14.4.2
  name: FEFO and expiry alerts
  phase: 3
  release: R1
  actors: [storekeeper, expiry_worker]
  screens: [SCR-INV-expiry-board]
  inputs: [days_ahead]
  states: [ok, near_expiry, expired]
  api: GET /v1/properties/{pid}/stock-lots?expiring_within_days=
  events: [StockLotNearExpiry, StockLotExpired]
  data: [stock_lot, stock_balance]
  rules: ["expired lot auto-set on_hold then waste on disposal", "FEFO pick list default"]
  security: standard
  failure_cases: ["missing expiry on perishable -> blocked at receipt"]
  finance_report_effect: Expiry waste forecast.
  i18n_a11y: Standard.
  acceptance: "A lot passing expiry at midnight property time becomes non-issuable."
  dependency: none.

- id: M14.F14.4.SF14.4.3
  name: Recall hold and case
  phase: 3
  release: R1
  actors: [compliance_officer, executive_chef, storekeeper]
  screens: [SCR-INV-recall-case]
  inputs: [supplier_id, supplier_lot_codes, item_ids, source(supplier|authority|internal), notice_document]
  states: [opened, holds_applied, traced, disposed_or_returned, closed]
  api: POST /v1/properties/{pid}/recall-cases
  events: [RecallCaseOpened, RecallHoldApplied, RecallCaseClosed]
  data: [recall_case, stock_lot, stock_ledger_entry]
  rules: ["recall_hold entries move all matching balances to recalled status across stores within one transaction", "POS/menu items using affected lots flagged; M61 notified", "closure requires disposal or RTV evidence"]
  security: compliance scope; step-up
  failure_cases: ["lot codes fuzzy match -> manual confirm list"]
  finance_report_effect: Recall loss / vendor claim receivable.
  i18n_a11y: Standard.
  acceptance: "AT-G18.8 - simulated supplier recall holds all balances of the lot in 1 transaction; they cannot be issued; trace lists affected events."
  dependency: M61, M50 SF50.2.10.

- id: M14.F14.4.SF14.4.4
  name: Quarantine decision queue
  phase: 3
  release: R1
  actors: [executive_chef, storekeeper, procurement_officer]
  screens: [SCR-INV-quarantine-queue]
  inputs: [quarantine_entry_ids, decision(release|waste|rtv), evidence]
  states: [pending, decided]
  api: POST /v1/properties/{pid}/quarantine/decisions
  events: [QuarantineDecided]
  data: [stock_ledger_entry, waste_record]
  rules: ["release only for receipt-discrepancy quarantine resolved in favour (e.g. count error) or intact-return rules; never for waste status", "RTV creates vendor claim (M50 SF50.2.9)"]
  security: segregation - decider not the receiver
  failure_cases: ["aging > SLA -> escalation"]
  finance_report_effect: Waste or vendor credit.
  i18n_a11y: Standard.
  acceptance: "Temperature-breach quarantine can be RTV or waste; release option not offered."
  dependency: M50.
```

### F14.5 Consumption, counts and reporting

```yaml
- id: M14.F14.5.SF14.5.1
  name: Theoretical POS depletion
  phase: 3
  release: R1
  actors: [depletion_worker]
  screens: [SCR-INV-depletion-log]
  inputs: [PosCheckClosed events, recipe versions]
  states: [pending, posted, coverage_gap]
  api: internal consumer of PosCheckClosed ; GET /v1/properties/{pid}/depletions
  events: [TheoreticalDepletionPosted]
  data: [stock_ledger_entry, recipe, pos_check_line]
  rules: ["pos_depletion entries per closed check line x recipe version incl. modifiers, from outlet store", "idempotency_key = check_line_id + recipe_version", "voided unprepared lines not depleted; prepared voids go to waste via SF14.3.6", "depletion may drive theoretical balance negative only in a separate theoretical projection, never the physical ledger availability used for issues (D-224)"]
  security: service identity
  failure_cases: ["recipe missing -> coverage gap report", "replayed check event -> no duplicate"]
  finance_report_effect: Theoretical COGS for outlet.
  i18n_a11y: none.
  acceptance: "AT-G03.5 - 10 mojitos sold deplete 300 ml rum and 100 g mint from bar store once."
  dependency: M13.

- id: M14.F14.5.SF14.5.2
  name: Approximate consumption for batches and events
  phase: 3
  release: R1
  actors: [catering_manager, executive_chef]
  screens: [SCR-INV-event-consumption]
  inputs: [beo_version_id, actual_covers, production_batches]
  states: [estimated, confirmed]
  api: GET /v1/properties/{pid}/events/{eid}/consumption
  events: [EventConsumptionEstimated]
  data: [recipe, stock_ledger_entry, catering_actual]
  rules: ["estimated consumption = recipe x produced portions; compared with issued minus intact returns minus waste", "labelled estimate until counts"]
  security: standard
  failure_cases: ["missing production count -> uses guarantee, labelled"]
  finance_report_effect: Event food cost estimate vs actual.
  i18n_a11y: Standard.
  acceptance: "G1 lunch shows issued 42 kg, estimated 38 kg, intact return 2 kg, waste 1.5 kg, unexplained 0.5 kg."
  dependency: M16.

- id: M14.F14.5.SF14.5.3
  name: Blind stock count
  phase: 3
  release: R1
  actors: [storekeeper, fnb_manager, auditor]
  screens: [SCR-INV-count-sheet, SCR-STF-count-scan]
  inputs: [store_id, count_scope(full|cycle|item_set), counted_qty_by_bin_lot, counter_ids]
  states: [planned, frozen, counting, submitted, reviewed, posted]
  api: POST /v1/properties/{pid}/stock-counts ; POST /v1/properties/{pid}/stock-counts/{cid}/submit
  events: [StockCountPosted]
  data: [stock_count, stock_count_line, stock_ledger_entry]
  rules: ["counters never see system quantity (blind)", "movements during count captured by snapshot timestamp", "recount required above tolerance", "posting creates count_adjustment entries"]
  security: counter != approver
  failure_cases: ["offline count -> synced with timestamps", "movement after freeze -> adjusted automatically"]
  finance_report_effect: Shrinkage entries.
  i18n_a11y: Scanner with manual entry; large numeric keypad.
  acceptance: "Counter UI never exposes expected qty; a 15% variance forces a recount before posting."
  dependency: none.

- id: M14.F14.5.SF14.5.4
  name: Variance and shrinkage analysis
  phase: 3
  release: R1
  actors: [fnb_manager, financial_controller]
  screens: [SCR-INV-variance-report]
  inputs: [store_id, period]
  states: [none]
  api: GET /v1/properties/{pid}/reports/stock-variance
  events: [StockVarianceAboveThreshold]
  data: [stock_ledger_entry, stock_count]
  rules: ["actual usage = opening + receipts + transfers in - transfers out - closing; theoretical from depletions; variance = actual - theoretical - recorded waste", "coverage % shown", "above-threshold feeds M60 case"]
  security: controller scope
  failure_cases: ["incomplete count -> no variance, reason shown"]
  finance_report_effect: Shrinkage GL; M32 cost.
  i18n_a11y: Table alternative.
  acceptance: "Variance report ties opening/closing values to the ledger for the period."
  dependency: M60.

- id: M14.F14.5.SF14.5.5
  name: Cost, margin and COGS posting
  phase: 4
  release: R1
  actors: [financial_controller, gl_posting_worker]
  screens: [SCR-FIN-cogs, SCR-MGT-outlet-margin]
  inputs: [period, outlet_id]
  states: [draft, posted]
  api: POST /v1/properties/{pid}/stock-postings
  events: [StockJournalPosted]
  data: [stock_ledger_entry, journal_entry]
  rules: ["summarized journals by movement type/store/cost center into M19", "food and beverage cost % = COGS / net revenue per KPI dictionary", "posting idempotent per period + batch"]
  security: finance
  failure_cases: ["period closed -> post to next with adjustment note"]
  finance_report_effect: COGS, waste, shrinkage, inventory asset; departmental P&L.
  i18n_a11y: Standard.
  acceptance: "AT-G08.2 - bar cost % in management report drills to ledger entries."
  dependency: M19, M32.
```

**M14 key invariants:** INV-STK-1, INV-STK-2; one ledger for all departments (no second quantity source); no negative available per lot/store; expired/recalled/quarantined lots are not issuable; blind counts; receipt accepted once per GR line.

**M14 module acceptance:** AT-G03.5, AT-G08.2, AT-G18.4–G18.8, AT-G20.6 (supplier short delivery → quarantine and partial receipt, jointly with M50).

**M14 open decisions**
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-223 | Costing method (moving average vs FIFO by lot) | financial_controller | Moving average; lot cost retained for trace |
| D-224 | Allow physical depletion to create negative balances when counts lag | fnb_manager / financial_controller | No; depletion posts from outlet store and shortfall goes to a 'depletion shortfall' exception, physical availability never negative |
| D-225 | Intact-return eligibility categories (sealed beverages yes; any prepared food never) | executive_chef / compliance_officer | Only sealed, never-in-guest-service, in-date, cold-chain-evidenced items |
| D-226 | GS1 barcode adoption for suppliers without GS1 | procurement_officer | Internal lot labels printed at receipt |

---

## M15 — Club and membership

| Item | Value |
|---|---|
| Purpose | Configurable venue module (lounge, nightlife, pool club, fitness) with safety occupancy and commercial capacity per session, member plans/benefits/renewals, passes and access control, reservations, entry and re-entry, club events, hosted bar and minimum-spend billing. Hidden when the hotel has no club. |
| Build phases | 3 |
| Release | R1 (activated per property profile) |
| Bounded context | `club` (schema `club`) |
| Systems of record owned | `club_venue`, `club_zone`, `club_session`, `membership_plan`, `membership`, `club_pass`, `access_event`, `club_reservation` (backed by M09 `resource_allocation`), `minimum_spend_commitment` |
| Upstream | M09 (zone/table capacity), M02 (member identity, consent), M05 (in-house guest entitlement), M12 (event club visits), M28 (membership payments), M44 (licensing/age rules), M64 (door devices) |
| Downstream | M13 (club POS, minimum spend), M08 (charges), M30 (points), M32 (club P&L), M42 (incidents) |

**Device honesty:** turnstiles/door readers, if used, connect via an M64 access-control adapter labelled `unverified-assumption`; the default R1 path is QR/NFC scan on the staff app with a manual counter.

### F15.1 Venue, zones and capacity

```yaml
- id: M15.F15.1.SF15.1.1
  name: Configure club venue
  phase: 3
  release: R1
  actors: [property_admin, fnb_manager]
  screens: [SCR-ADM-club-venue]
  inputs: [venue_type(lounge|nightlife|pool|fitness|other), name_en_ar, opening_sessions, age_rule, dress_code_text, guest_eligibility, license_ref]
  states: [draft, active, suspended]
  api: POST /v1/properties/{pid}/club-venues
  events: [ClubVenueConfigured]
  data: [club_venue]
  rules: ["module hidden when no active venue", "age/licensing rules from M44; unverified pack -> conservative default and admin warning"]
  security: property_admin
  failure_cases: ["activation without safety occupancy -> blocked"]
  finance_report_effect: Club revenue center and cost center.
  i18n_a11y: EN/AR names; accessible venue info to guests.
  acceptance: "Venue cannot activate without a safety occupancy value."
  dependency: M44.

- id: M15.F15.1.SF15.1.2
  name: Safety occupancy vs commercial capacity
  phase: 3
  release: R1
  actors: [property_admin, security_officer, chief_engineer]
  screens: [SCR-ADM-club-capacity]
  inputs: [venue_id, zone_id, safety_occupancy_limit, evidence_doc, commercial_capacity_by_session, member_reserve, walk_in_reserve]
  states: [configured]
  api: PUT /v1/properties/{pid}/club-venues/{vid}/capacity
  events: [ClubCapacityChanged]
  data: [club_zone, club_session, timed_resource]
  rules: ["commercial capacity <= safety occupancy; pools in M09 derived from commercial capacity", "live headcount (SF15.3.3) can never exceed safety occupancy - entry denied at limit regardless of passes"]
  security: safety limit editable by chief_engineer/security with evidence
  failure_cases: ["evidence missing -> unverified label and cap (D-203 policy)"]
  finance_report_effect: Capacity denominators.
  i18n_a11y: Standard.
  acceptance: "Setting commercial capacity 250 with safety 200 is rejected."
  dependency: M09, D-203.

- id: M15.F15.1.SF15.1.3
  name: Sessions and seating
  phase: 3
  release: R1
  actors: [fnb_manager, club_host]
  screens: [SCR-CLUB-sessions, SCR-CLUB-floor]
  inputs: [session_date, start, end, zones, tables_cabanas, pricing]
  states: [scheduled, open, closed, cancelled]
  api: POST /v1/properties/{pid}/club-venues/{vid}/sessions
  events: [ClubSessionOpened, ClubSessionClosed]
  data: [club_session, timed_resource]
  rules: ["tables/cabanas are M09 unit resources linked to zone pools", "session close requires all tabs closed or transferred"]
  security: standard
  failure_cases: ["cancel session with reservations -> notify and refund per policy"]
  finance_report_effect: Session revenue reporting.
  i18n_a11y: Floor map with list alternative.
  acceptance: "Cancelling a session with 10 reservations generates 10 notifications and refund tasks."
  dependency: M09.
```

### F15.2 Membership plans

```yaml
- id: M15.F15.2.SF15.2.1
  name: Membership plans and benefits
  phase: 3
  release: R1
  actors: [fnb_manager, marketing_manager, financial_controller]
  screens: [SCR-ADM-membership-plans]
  inputs: [plan_name, term(monthly|annual|custom), price, joining_fee, benefits(access_sessions/guest_passes/discount_percent/priority_booking), max_members, terms_version]
  states: [draft, active, closed_to_new, retired]
  api: POST /v1/properties/{pid}/membership-plans
  events: [MembershipPlanChanged]
  data: [membership_plan]
  rules: ["benefit changes versioned; existing members keep version until renewal", "discount benefits applied via POS price level/discount rule", "membership is not a stored-value wallet; prepaid credit not permitted unless licensed (M30 SF30.2.6)"]
  security: finance approves pricing
  failure_cases: ["plan with stored value -> rejected"]
  finance_report_effect: Deferred membership revenue recognized over term (docs/06).
  i18n_a11y: Bilingual terms.
  acceptance: "Annual plan revenue is deferred and recognized monthly in the GL export."
  dependency: M19.

- id: M15.F15.2.SF15.2.2
  name: Enrollment and payment
  phase: 3
  release: R1
  actors: [club_host, guest, cashier]
  screens: [SCR-CLUB-enroll, SCR-GST-membership]
  inputs: [guest_profile_id, plan_id, start_date, payment_method, consent_marketing, photo_optional, terms_acceptance]
  states: [pending_payment, active, payment_failed]
  api: POST /v1/properties/{pid}/memberships
  events: [MembershipActivated]
  data: [membership, payment_intent, consent_record]
  rules: ["activation only after payment confirmation (M28) or approved invoice", "member photo optional, consented, not used for biometric matching"]
  security: member PII scoped to club roles
  failure_cases: ["payment pending -> membership pending"]
  finance_report_effect: Deferred revenue; tax per M44.
  i18n_a11y: Accessible enrollment.
  acceptance: "Membership stays pending until PSP confirmation; duplicate callback activates once."
  dependency: M28.

- id: M15.F15.2.SF15.2.3
  name: Renewals, freeze and cancellation
  phase: 3
  release: R1
  actors: [club_host, guest, membership_worker]
  screens: [SCR-CLUB-member-detail, SCR-GST-membership]
  inputs: [membership_id, action(renew|freeze|cancel), effective_date, reason]
  states: [active, grace, frozen, expired, cancelled]
  api: POST /v1/properties/{pid}/memberships/{mid}/actions
  events: [MembershipRenewed, MembershipExpired, MembershipCancelled]
  data: [membership]
  rules: ["auto-renew only with explicit consent and saved token", "renewal reminders 30/7 days", "refunds per terms and consumer law rule pack"]
  security: standard
  failure_cases: ["renewal charge fails -> grace period then expire"]
  finance_report_effect: Deferred revenue adjustments.
  i18n_a11y: Standard.
  acceptance: "Failed renewal moves to grace for 7 days and then expired; access denied after expiry."
  dependency: M28.

- id: M15.F15.2.SF15.2.4
  name: In-house guest and corporate entitlements
  phase: 3
  release: R1
  actors: [front_desk_agent, club_host, sales_manager]
  screens: [SCR-CLUB-entitlements]
  inputs: [rate_plan_or_package_id, agreement_version_id, entitlement(sessions/guests/fee)]
  states: [active]
  api: PUT /v1/properties/{pid}/club-venues/{vid}/entitlements
  events: [ClubEntitlementChanged]
  data: [club_pass, agreement_version]
  rules: ["stay with entitlement issues club_pass valid for stay dates on check-in", "passes revoked on checkout or room move to non-entitled rate"]
  security: standard
  failure_cases: ["reservation without check-in -> no pass"]
  finance_report_effect: Package allocation to club revenue (M54 SF54.2.3).
  i18n_a11y: Standard.
  acceptance: "Check-in on package 'Stay & Lounge' issues a pass; checkout revokes it."
  dependency: M05, M54.
```

### F15.3 Reservations, passes and access

```yaml
- id: M15.F15.3.SF15.3.1
  name: Club reservations
  phase: 3
  release: R1
  actors: [club_host, guest, event_organizer]
  screens: [SCR-CLUB-reservations, SCR-GST-club-booking]
  inputs: [session_id, party_size, table_or_zone, min_spend_tier, deposit]
  states: [held, confirmed, arrived, no_show, cancelled]
  api: POST /v1/properties/{pid}/club-sessions/{sid}/reservations
  events: [ClubReservationConfirmed]
  data: [club_reservation, resource_allocation, minimum_spend_commitment]
  rules: ["allocation via M09 (zone pool + table)", "event club visit from composite hold converts to reservation", "no-show fee per terms"]
  security: standard
  failure_cases: ["capacity exceeded -> waitlist"]
  finance_report_effect: Deposit liability; no-show revenue.
  i18n_a11y: Standard.
  acceptance: "Composite G1 club visit for 80 appears as a confirmed reservation consuming zone capacity 80."
  dependency: M09, M12.

- id: M15.F15.3.SF15.3.2
  name: Passes and credentials
  phase: 3
  release: R1
  actors: [club_host, event_organizer, guest]
  screens: [SCR-CLUB-passes, SCR-GST-wallet-pass]
  inputs: [holder_ref, pass_type(member|stay|event|day), valid_from, valid_to, max_entries, reentry_allowed]
  states: [issued, active, used_up, revoked, expired]
  api: POST /v1/properties/{pid}/club-passes
  events: [ClubPassIssued, ClubPassRevoked]
  data: [club_pass]
  rules: ["pass token is signed, rotating QR (TOTP-style) in app; printed static QR only for event passes with single-entry limit", "pass is an access credential, not account authentication"]
  security: signing keys in vault; screenshots of rotating code expire within 30 s
  failure_cases: ["forged/expired token -> deny with reason"]
  finance_report_effect: none.
  i18n_a11y: Pass shows name and validity in text for staff read-out.
  acceptance: "A screenshot of a rotating pass older than 30 s is denied."
  dependency: M02 key management.

- id: M15.F15.3.SF15.3.3
  name: Entry and exit counting
  phase: 3
  release: R1
  actors: [club_host, security_officer]
  screens: [SCR-CLUB-door]
  inputs: [pass_token_or_manual_lookup, direction(in|out), device_id]
  states: [admitted, denied, exited]
  api: POST /v1/properties/{pid}/club-venues/{vid}/access-events
  events: [ClubEntryAdmitted, ClubEntryDenied, ClubExitRecorded, ClubCapacityReached]
  data: [access_event, club_pass, club_session]
  rules: ["live headcount = admitted - exited per zone; deny at safety occupancy with no override except emergency services", "commercial capacity reached -> only reserved/member admits per reserve rules", "multiple door devices serialize counts server-side; offline device uses allotted quota"]
  security: door device identity; host PIN
  failure_cases: ["device offline -> local quota slice (e.g. 10% of remaining) then block", "exit not scanned -> end-of-session reconciliation with manual count"]
  finance_report_effect: Cover entry fees posting.
  i18n_a11y: Visual + audible accept/deny; colour-blind safe.
  acceptance: "AT-G03.6 - with safety limit 200 and 200 inside, the next valid pass is denied; after 1 exit, 1 admit succeeds."
  dependency: M64.

- id: M15.F15.3.SF15.3.4
  name: Re-entry rules
  phase: 3
  release: R1
  actors: [club_host]
  screens: [SCR-CLUB-door]
  inputs: [pass_id, reentry_window_minutes, max_reentries]
  states: [inside, outside_reentry_allowed, reentry_expired]
  api: POST /v1/properties/{pid}/club-venues/{vid}/access-events
  events: [ClubReentryAdmitted, ClubReentryDenied]
  data: [access_event, club_pass]
  rules: ["re-entry counts against capacity like first entry", "anti-passback - pass already inside cannot enter again until exit recorded", "event passes single-entry unless BEO says otherwise"]
  security: standard
  failure_cases: ["missing exit record -> host override with reason and supervisor PIN"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "A pass inside cannot be used by a second person at the door (anti-passback)."
  dependency: none.

- id: M15.F15.3.SF15.3.5
  name: Denied entry and override log
  phase: 3
  release: R1
  actors: [club_host, duty_manager]
  screens: [SCR-CLUB-door, SCR-OPS-exception-queue]
  inputs: [access_event_id, override_reason, approver]
  states: [denied, overridden]
  api: POST /v1/properties/{pid}/access-events/{aeid}/override
  events: [ClubAccessOverridden]
  data: [access_event]
  rules: ["override permitted for commercial reasons only below safety occupancy", "age refusal cannot be overridden", "incidents link to M42"]
  security: supervisor PIN; audit
  failure_cases: ["override at safety limit -> impossible"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Override attempt at safety limit fails; below limit succeeds with reason logged."
  dependency: M42.
```

### F15.4 Spend, hosted bar and billing

```yaml
- id: M15.F15.4.SF15.4.1
  name: Minimum spend commitments
  phase: 3
  release: R1
  actors: [club_host, bartender, fnb_manager]
  screens: [SCR-CLUB-min-spend, SCR-POS-tabs]
  inputs: [reservation_id, min_spend_amount, inclusive_of_tax_service, deposit_applied]
  states: [open, met, shortfall, settled]
  api: POST /v1/properties/{pid}/club-reservations/{crid}/settle-min-spend
  events: [MinimumSpendSettled]
  data: [minimum_spend_commitment, pos_check]
  rules: ["tracked against linked POS checks", "shortfall posted as separate line 'minimum spend adjustment' taxed per M44 rule pack", "deposit applied first"]
  security: waiver needs manager
  failure_cases: ["checks not linked -> shortfall overstated; host can link before settlement"]
  finance_report_effect: Shortfall revenue line.
  i18n_a11y: Standard.
  acceptance: "Min spend 300, consumption 260 -> adjustment 40 posted once with correct tax."
  dependency: M13.

- id: M15.F15.4.SF15.4.2
  name: Hosted bar at club
  phase: 3
  release: R1
  actors: [bartender, catering_manager]
  screens: [SCR-POS-hosted-tab]
  inputs: [function_id, limit]
  states: [open, limit_reached, closed]
  api: POST /v1/properties/{pid}/functions/{fid}/hosted-tab
  events: [HostedTabClosed]
  data: [pos_check, beo_version]
  rules: ["same as SF12.4.2 with club outlet"]
  security: standard
  failure_cases: ["see SF12.4.2"]
  finance_report_effect: Event beverage revenue.
  i18n_a11y: Standard.
  acceptance: "G1 club visit hosted tab closes to master account once."
  dependency: M12.

- id: M15.F15.4.SF15.4.3
  name: Club billing destinations
  phase: 3
  release: R1
  actors: [club_host, cashier]
  screens: [SCR-CLUB-billing]
  inputs: [charge_type(entry|membership|min_spend|no_show), destination(folio|master|card|member_invoice)]
  states: [posted]
  api: POST /v1/properties/{pid}/club-charges
  events: [FolioChargePosted]
  data: [folio_charge]
  rules: ["INV-FOL-1 unique source", "member invoices for monthly fees via M08/M20"]
  security: standard
  failure_cases: ["duplicate door event -> no duplicate entry fee"]
  finance_report_effect: Club revenue lines.
  i18n_a11y: Standard.
  acceptance: "Entry fee posts once even if door scan retried."
  dependency: M08.

- id: M15.F15.4.SF15.4.4
  name: Club events and ticketing
  phase: 3
  release: R1
  actors: [fnb_manager, marketing_manager, guest]
  screens: [SCR-CLUB-events, SCR-GST-club-events]
  inputs: [session_id, ticket_types, price, quantity, sale_window]
  states: [draft, on_sale, sold_out, closed]
  api: POST /v1/properties/{pid}/club-sessions/{sid}/ticket-types
  events: [ClubTicketSold]
  data: [club_pass, resource_allocation]
  rules: ["tickets consume session commercial capacity via M09", "refund per policy"]
  security: standard
  failure_cases: ["concurrent last ticket -> one sale"]
  finance_report_effect: Ticket revenue; deferred until event date.
  i18n_a11y: Accessible purchase flow.
  acceptance: "Last ticket purchased concurrently by two users yields one sale."
  dependency: M09, M28.
```

**M15 key invariants:** live headcount never exceeds safety occupancy (no override); commercial capacity ≤ safety occupancy; passes are access credentials not identity; membership is not stored value; each club charge posts once.

**M15 module acceptance:** AT-G03.6 (club admission respects capacity), AT-G02.1 (club component in composite hold).

**M15 open decisions**
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-227 | Does the pilot hotel operate a club and which venue type | gm | One lounge/pool club |
| D-228 | Door hardware (turnstile/reader) vs staff-app scanning | security_officer / it_admin | Staff-app scanning; hardware `unverified-assumption` |
| D-229 | Offline door quota slice | security_officer | 10% of remaining commercial capacity per device, never above safety |

---

## M16 — Catering

| Item | Value |
|---|---|
| Purpose | In-hotel and off-site catering: menus/packages/tiers, guaranteed covers and cutoffs, allergen flags, BEO version consumption, kitchen prep/production, ingredient reservation and purchasing, staff/equipment/vehicle, dispatch/returns and actual-vs-contracted invoicing. |
| Build phases | 3 (orders, guarantees, production plan, reservations) • 4 (purchasing/costing integration, off-site vehicle cost) |
| Release | R1 |
| Bounded context | `catering` (schema `cat`) |
| Systems of record owned | `catering_package`, `catering_order`, `cover_guarantee`, `production_plan`, `production_batch`, `ingredient_reservation`, `dispatch_manifest`, `catering_actual` |
| Upstream | M12 (BEO versions, functions), M09 (kitchen production slots), M14 (recipes, stock, lots), M47 (chef coverage), M21/M49 (purchasing), M59 (vehicles/drivers), M61 (food-safety checks), M44 (allergen/food rules) |
| Downstream | M14 (issues, production output/consume, waste), M12 (actuals for reconciliation), M08 (charges), M32 (event food cost), M50 (receiving of event-specific purchases) |

### F16.1 Menus, packages and tiers

```yaml
- id: M16.F16.1.SF16.1.1
  name: Catering menus and packages
  phase: 3
  release: R1
  actors: [catering_manager, executive_chef]
  screens: [SCR-CAT-package-editor]
  inputs: [package_name_en_ar, tier(standard|premium|deluxe), components(menu_items/recipes/beverage_rules), price_per_cover, min_covers, service_style, off_site_eligible, lead_time_hours, valid_from_to]
  states: [draft, active, retired]
  api: POST /v1/properties/{pid}/catering-packages
  events: [CateringPackageChanged]
  data: [catering_package, recipe]
  rules: ["every food component links to a recipe version for costing, allergens and depletion", "price changes versioned; confirmed events keep contracted version"]
  security: cost visible to catering management/finance only
  failure_cases: ["component without recipe -> cannot activate"]
  finance_report_effect: Theoretical food cost % per package.
  i18n_a11y: Bilingual menu cards; allergen text.
  acceptance: "Premium lunch package shows cost per cover and food-cost % and cannot activate with an unmapped dish."
  dependency: M14.

- id: M16.F16.1.SF16.1.2
  name: Tiered and per-cover pricing
  phase: 3
  release: R1
  actors: [catering_manager, revenue_manager]
  screens: [SCR-CAT-pricing]
  inputs: [package_id, cover_bands, child_price, agreement_overrides, service_charge]
  states: [active]
  api: PUT /v1/properties/{pid}/catering-packages/{cpid}/pricing
  events: [CateringPricingChanged]
  data: [catering_package, agreement_version]
  rules: ["agreement prices take precedence when eligible", "taxes per M44 product treatment (F&B vs catering vs off-site delivery)"]
  security: standard
  failure_cases: ["tax pack unverified for off-site -> estimate label, filing gate"]
  finance_report_effect: Revenue by package.
  i18n_a11y: Standard.
  acceptance: "Acme sees contracted lunch price; Beta sees list price."
  dependency: M10, M44.

- id: M16.F16.1.SF16.1.3
  name: Allergen and diet matrix
  phase: 3
  release: R1
  actors: [executive_chef]
  screens: [SCR-CAT-allergen-matrix]
  inputs: [package_id]
  states: [computed, chef_confirmed]
  api: GET /v1/properties/{pid}/catering-packages/{cpid}/allergens
  events: [CateringAllergenMatrixConfirmed]
  data: [recipe, catering_package]
  rules: ["derived from M14 roll-up; chef confirmation required", "diet labels (vegan, halal where certified evidence exists, gluten-free) only with evidence; 'halal' claim requires supplier certificate evidence in M46/M48"]
  security: standard
  failure_cases: ["claim without evidence -> blocked"]
  finance_report_effect: none.
  i18n_a11y: Matrix accessible table.
  acceptance: "Package cannot be labelled halal without linked supplier evidence."
  dependency: M14, M48.

- id: M16.F16.1.SF16.1.4
  name: Off-site eligibility and service radius
  phase: 3
  release: R1
  actors: [catering_manager]
  screens: [SCR-CAT-offsite-settings]
  inputs: [max_distance_km, max_transit_minutes, hot_hold_capable, cold_chain_equipment, permit_refs]
  states: [active, suspended]
  api: PUT /v1/properties/{pid}/catering/offsite-settings
  events: [OffsiteSettingsChanged]
  data: [catering_package]
  rules: ["off-site orders blocked where M44 requires a permit not evidenced", "transit time limits derived from food-safety rule pack"]
  security: standard
  failure_cases: ["permit expired -> off-site sale disabled"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Expired off-site catering permit disables off-site ordering."
  dependency: M44, M61.
```

### F16.2 Orders, guarantees and cutoffs

```yaml
- id: M16.F16.2.SF16.2.1
  name: Catering order from BEO
  phase: 3
  release: R1
  actors: [catering_manager, system]
  screens: [SCR-CAT-order-detail]
  inputs: [beo_version_id, function_id, location(in_hotel|off_site_address), service_time, packages, expected_covers]
  states: [draft, confirmed, in_production, dispatched, served, reconciled, cancelled]
  api: POST /v1/properties/{pid}/catering-orders
  events: [CateringOrderConfirmed, CateringOrderChanged]
  data: [catering_order, beo_version, resource_allocation]
  rules: ["one catering_order per function; always bound to latest acknowledged BEO version", "kitchen production slot allocation via M09 held in composite hold", "stand-alone catering (no rooms) also creates event_booking in M12"]
  security: standard
  failure_cases: ["BEO revision while in production -> change flagged to chef with delta"]
  finance_report_effect: Revenue on service date via event master account.
  i18n_a11y: Standard.
  acceptance: "BEO v2 updates the catering order and increments its version reference."
  dependency: M12, M09.

- id: M16.F16.2.SF16.2.2
  name: Guaranteed covers and cutoffs
  phase: 3
  release: R1
  actors: [catering_manager, event_organizer, guarantee_worker]
  screens: [SCR-CAT-guarantees, SCR-CORP-guarantee]
  inputs: [catering_order_id, guarantee_cutoff, guaranteed_covers]
  states: [expected, guaranteed, locked]
  api: PUT /v1/properties/{pid}/catering-orders/{coid}/guarantee
  events: [CoverGuaranteeLocked]
  data: [cover_guarantee]
  rules: ["guarantee locks at cutoff; if none submitted, expected covers become guarantee", "post-cutoff increases accepted only within kitchen slot capacity and stock; decreases not billed-down"]
  security: organizer + catering
  failure_cases: ["cutoff worker missed -> run idempotently on recovery with original timestamp"]
  finance_report_effect: Billing floor.
  i18n_a11y: Countdown shown with time zone.
  acceptance: "No guarantee at cutoff locks expected covers 80; organizer's later decrease to 70 is recorded but floor remains 80."
  dependency: M12 SF12.3.5.

- id: M16.F16.2.SF16.2.3
  name: Overset and production quantity
  phase: 3
  release: R1
  actors: [executive_chef]
  screens: [SCR-CAT-production-plan]
  inputs: [guaranteed_covers, overset_percent]
  states: [computed]
  api: GET /v1/properties/{pid}/catering-orders/{coid}/production-quantities
  events: [none]
  data: [cover_guarantee, production_plan]
  rules: ["prepare = guarantee x (1 + overset) where overset default 3-5% (D-231)", "overset not billed; its cost is event food cost"]
  security: standard
  failure_cases: ["none"]
  finance_report_effect: Overset cost visible in event P&L.
  i18n_a11y: Standard.
  acceptance: "Guarantee 80 with 5% overset plans 84 portions."
  dependency: none.

- id: M16.F16.2.SF16.2.4
  name: Allergen flags and special meals
  phase: 3
  release: R1
  actors: [catering_manager, executive_chef, event_organizer]
  screens: [SCR-CAT-special-meals, SCR-CORP-guarantee]
  inputs: [allergen_counts, named_special_meals(consented), seat_or_badge_ref]
  states: [requested, acknowledged_by_kitchen, prepared, served]
  api: POST /v1/properties/{pid}/catering-orders/{coid}/special-meals
  events: [SpecialMealAcknowledged]
  data: [catering_order, attendee]
  rules: ["each allergen special meal requires kitchen acknowledgment and separate labelled prep", "named data only with attendee consent; otherwise counts plus seat/badge ref", "late allergen request after lock needs chef acceptance"]
  security: health-related data minimized and access-limited
  failure_cases: ["unacknowledged allergen meal 2 h before service -> escalation to executive_chef and duty_manager"]
  finance_report_effect: none.
  i18n_a11y: Special meal labels bilingual, large type.
  acceptance: "2 nut-allergy meals require acknowledgment; missing acknowledgment escalates."
  dependency: M57 SF57.1.4.
```

### F16.3 Production and purchasing

```yaml
- id: M16.F16.3.SF16.3.1
  name: Production plan and prep lists
  phase: 3
  release: R1
  actors: [executive_chef, shift_chef]
  screens: [SCR-CAT-production-plan, SCR-KDS-prep-list]
  inputs: [service_date, catering_orders, recipe_versions]
  states: [draft, published, in_progress, completed]
  api: POST /v1/properties/{pid}/production-plans
  events: [ProductionPlanPublished, ProductionPlanChanged]
  data: [production_plan, recipe]
  rules: ["aggregates all orders by recipe and station with timeline backwards from service time", "BEO revision regenerates plan diff and requires chef acknowledgment"]
  security: kitchen scope
  failure_cases: ["plan change after start -> delta list only"]
  finance_report_effect: Planned food cost.
  i18n_a11y: Prep list printable large font; bilingual.
  acceptance: "Adding 5 covers produces a delta prep list for affected stations only."
  dependency: M12 SF12.3.3.

- id: M16.F16.3.SF16.3.2
  name: Ingredient reservation against stock
  phase: 3
  release: R1
  actors: [executive_chef, storekeeper, reservation_worker]
  screens: [SCR-INV-reservations]
  inputs: [production_plan_id]
  states: [reserved, partially_reserved, shortage, released, consumed]
  api: POST /v1/properties/{pid}/ingredient-reservations
  events: [IngredientsReserved, IngredientShortageDetected]
  data: [ingredient_reservation, stock_balance, stock_lot]
  rules: ["reserve by item (lot chosen FEFO at issue) against available status only", "reservations reduce available-to-promise but are not ledger movements", "released on cancellation or plan reduction"]
  security: standard
  failure_cases: ["shortage -> requisition suggestion (SF16.3.3)", "lot recalled after reservation -> re-reserve alternative"]
  finance_report_effect: none until issue.
  i18n_a11y: Standard.
  acceptance: "G1 lunch reserves 40 kg chicken; available-to-promise for others drops by 40 kg."
  dependency: M14.

- id: M16.F16.3.SF16.3.3
  name: Shortage to requisition
  phase: 3
  release: R1
  actors: [executive_chef, procurement_officer]
  screens: [SCR-CAT-shortages]
  inputs: [shortage_lines, need_by_datetime, event_ref]
  states: [suggested, requisitioned, ordered, received]
  api: POST /v1/properties/{pid}/purchase-requisitions
  events: [PurchaseRequisitionCreated]
  data: [purchase_requisition, ingredient_reservation]
  rules: ["requisition carries event/BEO reference and delivery deadline", "RFQ/award via M49 thresholds; emergency path M21 SF21.1.6", "M50 alerts catering on late ETA (SF50.1.6)"]
  security: requisition approval per M21
  failure_cases: ["no supplier can meet deadline -> menu substitution approval path (M47 SF47.2.6 style)"]
  finance_report_effect: Event-coded purchases.
  i18n_a11y: Standard.
  acceptance: "AT-G17.1 - kitchen requisition for 120 kg vegetables for an event carries event ref into RFQ."
  dependency: M21, M49, M50.

- id: M16.F16.3.SF16.3.4
  name: Kitchen slot and chef coverage check
  phase: 3
  release: R1
  actors: [executive_chef, duty_manager]
  screens: [SCR-CAT-production-plan]
  inputs: [production_plan_id]
  states: [covered, at_risk, uncovered]
  api: GET /v1/properties/{pid}/production-plans/{ppid}/coverage
  events: [ProductionCoverageAtRisk]
  data: [production_plan, chef_assignment, resource_allocation]
  rules: ["each plan requires named chef and backup (M47)", "uncovered -> M47 callout"]
  security: standard
  failure_cases: ["chef and backup absent -> emergency roster (AT-G15)"]
  finance_report_effect: Emergency chef cost to event.
  i18n_a11y: Standard.
  acceptance: "AT-G15.1 - plan shows uncovered when both chef and backup mark absence and triggers M47 callout."
  dependency: M47.

- id: M16.F16.3.SF16.3.5
  name: Production batches and lot linkage
  phase: 3
  release: R1
  actors: [shift_chef]
  screens: [SCR-KDS-batch, SCR-STF-batch]
  inputs: [recipe_version, input_lots_and_qty, output_qty, produced_at, hold_temperature_checks]
  states: [started, completed, held, served, discarded]
  api: POST /v1/properties/{pid}/production-batches
  events: [ProductionBatchCompleted]
  data: [production_batch, stock_lot, stock_ledger_entry]
  rules: ["production_consume entries for inputs and production_output lot for output (if stored)", "hot/cold holding checks per M61/M57 rule pack", "batch served to guests is terminal - leftovers go to waste, never back to stock"]
  security: standard
  failure_cases: ["temperature check missed -> batch held; release needs chef decision or waste"]
  finance_report_effect: Actual production cost.
  i18n_a11y: Standard.
  acceptance: "Leftover served buffet batch can only be recorded as waste."
  dependency: M14, M57, M61.
```

### F16.4 Off-site logistics

```yaml
- id: M16.F16.4.SF16.4.1
  name: Staff, equipment and vehicle scheduling
  phase: 3
  release: R1
  actors: [catering_manager]
  screens: [SCR-CAT-logistics]
  inputs: [catering_order_id, staff_roles_counts, equipment_pool_items, vehicle_id_or_external, driver]
  states: [planned, confirmed]
  api: POST /v1/properties/{pid}/catering-orders/{coid}/logistics
  events: [CateringLogisticsConfirmed]
  data: [resource_allocation, catering_order]
  rules: ["equipment and staff via M09 pools", "vehicles via M59 fleet or external provider from M46", "vehicle must have valid cold-chain equipment where required"]
  security: standard
  failure_cases: ["vehicle unavailable -> external provider RFQ"]
  finance_report_effect: Logistics cost to event.
  i18n_a11y: Standard.
  acceptance: "Off-site order cannot confirm without a vehicle with valid permit and cold box."
  dependency: M59, M09.

- id: M16.F16.4.SF16.4.2
  name: Dispatch manifest and cold chain
  phase: 3
  release: R1
  actors: [catering_manager, shift_chef, driver_via_staff_app]
  screens: [SCR-CAT-dispatch, SCR-STF-dispatch]
  inputs: [items(batch/qty/container), equipment_list, departure_temp_readings, departure_time]
  states: [loading, dispatched, in_transit, delivered]
  api: POST /v1/properties/{pid}/dispatch-manifests
  events: [CateringDispatched]
  data: [dispatch_manifest, production_batch]
  rules: ["each container linked to batch", "temperature at load and arrival recorded; out-of-range -> hold at venue and chef decision"]
  security: standard
  failure_cases: ["driver app offline -> readings queued with timestamps"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Manifest lists 6 containers with batch ids and load temperatures."
  dependency: M61.

- id: M16.F16.4.SF16.4.3
  name: Delivery confirmation at venue
  phase: 3
  release: R1
  actors: [catering_manager, event_organizer]
  screens: [SCR-STF-dispatch]
  inputs: [manifest_id, arrival_temp_readings, recipient_signature, photos]
  states: [delivered, disputed]
  api: POST /v1/properties/{pid}/dispatch-manifests/{dmid}/deliver
  events: [CateringDelivered]
  data: [dispatch_manifest]
  rules: ["recipient signature via M41 lightweight signature or name+photo evidence", "delivery time feeds on-time KPI"]
  security: standard
  failure_cases: ["recipient absent -> photo + GPS + callback"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Delivery recorded with arrival temperatures and signature."
  dependency: M41.

- id: M16.F16.4.SF16.4.4
  name: Returns of equipment and food
  phase: 3
  release: R1
  actors: [catering_manager, storekeeper]
  screens: [SCR-CAT-returns]
  inputs: [manifest_id, equipment_returned, food_returned(qty/condition)]
  states: [returned, missing, waste]
  api: POST /v1/properties/{pid}/dispatch-manifests/{dmid}/returns
  events: [CateringReturnsRecorded]
  data: [dispatch_manifest, waste_record, stock_ledger_entry]
  rules: ["food returned from off-site service -> waste (terminal); sealed unopened beverages may go to quarantine for inspection (SF14.3.5)", "missing equipment -> loss charge per contract or write-off"]
  security: standard
  failure_cases: ["unrecorded returns -> manifest cannot close"]
  finance_report_effect: Waste cost; equipment loss.
  i18n_a11y: Standard.
  acceptance: "Returned trays are posted as waste; 12 sealed water bottles go to quarantine not directly to available."
  dependency: M14.
```

### F16.5 Actuals and settlement

```yaml
- id: M16.F16.5.SF16.5.1
  name: Capture actual covers and consumption
  phase: 3
  release: R1
  actors: [catering_manager, server]
  screens: [SCR-CAT-actuals, SCR-STF-event-live]
  inputs: [catering_order_id, actual_covers(count_method), extra_items, beverage_actuals]
  states: [draft, confirmed_by_organizer, disputed]
  api: POST /v1/properties/{pid}/catering-orders/{coid}/actuals
  events: [CateringActualsRecorded]
  data: [catering_actual]
  rules: ["count method recorded (headcount, plates, badge scans)", "organizer confirmation on site or via portal"]
  security: standard
  failure_cases: ["dispute -> reconciliation flag"]
  finance_report_effect: Basis for billing.
  i18n_a11y: Standard.
  acceptance: "Actual 76 covers confirmed by organizer."
  dependency: M12.

- id: M16.F16.5.SF16.5.2
  name: Actual vs contracted invoice lines
  phase: 3
  release: R1
  actors: [catering_manager, ar_clerk]
  screens: [SCR-EVT-reconciliation]
  inputs: [catering_order_id]
  states: [computed, posted]
  api: POST /v1/properties/{pid}/catering-orders/{coid}/bill
  events: [CateringBilled]
  data: [catering_actual, cover_guarantee, folio_charge]
  rules: ["billable covers = max(guarantee, actual) unless contract says otherwise", "F&B minimum shortfall per contract as separate line", "post once to master account (INV-FOL-1)"]
  security: standard
  failure_cases: ["contract clause missing -> manual review"]
  finance_report_effect: Catering revenue.
  i18n_a11y: Standard.
  acceptance: "AT-G02.5 - guarantee 80, actual 76 bills 80; actual 85 bills 85."
  dependency: M12 SF12.5.3.

- id: M16.F16.5.SF16.5.3
  name: Event food cost and margin
  phase: 4
  release: R1
  actors: [catering_manager, financial_controller]
  screens: [SCR-MGT-event-pnl]
  inputs: [catering_order_id]
  states: [estimate, reconciled]
  api: GET /v1/properties/{pid}/catering-orders/{coid}/margin
  events: [none]
  data: [stock_ledger_entry, production_batch, catering_actual]
  rules: ["cost = issues - intact returns + waste attributable + logistics + emergency chef", "reconciled after counts and invoices matched"]
  security: management
  failure_cases: ["missing invoice -> estimate"]
  finance_report_effect: Feeds M12 event P&L.
  i18n_a11y: Standard.
  acceptance: "Catering margin drills to issues, waste and logistics cost lines."
  dependency: M14, M19.
```

**M16 key invariants:** every dish has a recipe version; guarantees lock at cutoff; ingredient reservations are not ledger movements; served or off-site-returned food is terminal waste; billable covers follow the contract rule.

**M16 module acceptance:** AT-G02.4–G02.5, AT-G03.5 (catering depletes stock), AT-G15.1, AT-G17.1, AT-G18.5–G18.7.

**M16 open decisions**
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-230 | Guarantee cutoff default | catering_manager | 72 h before service |
| D-231 | Default overset percentage | executive_chef | 5% |
| D-232 | Off-site catering permits and transit limits per pilot market | compliance_officer | Off-site disabled until permit evidence exists |
| D-233 | Halal/diet claims evidence standard | executive_chef / compliance_officer | Claim only with valid supplier certificate on file |

---

## M17 — Parking and ANPR

| Item | Value |
|---|---|
| Purpose | Parking facilities/zones/capacity, guest/corporate/staff/event permits, a consented plate registry, vendor-neutral adapters for edge AI cameras or LPR servers, plate observations with confidence thresholds and low-confidence human review, lane/gate decisions, entry/exit sessions, tariffs, one-time folio posting, manual fallback and reconciliation. |
| Build phases | 3 |
| Release | R1 (pilot hardware certification is a release dependency per property) |
| Bounded context | `parking` (schema `park`) |
| Systems of record owned | `parking_facility`, `parking_zone`, `parking_lane`, `parking_permit`, `plate_registration`, `lpr_device`, `plate_observation`, `observation_review`, `gate_decision`, `gate_command`, `parking_session`, `parking_tariff` |
| Upstream | M09 (parking pools for composite holds), M05 (stay dates, in-house status), M10/M12 (corporate/event entitlements), M02 (consent, retention), M44 (privacy/plate retention rules per market), M64 (device registry, certificates, health), M28 (pay-on-exit) |
| Downstream | M08 (parking charges), M18 (guest vehicle registration), M32 (parking P&L, SF32.2.3), M42 (security incidents), M60 (override audit) |

**Hardware and partner honesty:** MetriStay defines an `LprAdapter` port (`docs/05` `INT-LPR`) with capability flags: `push_observations`, `pull_observations`, `plate_image`, `vehicle_attributes`, `confidence_score`, `lane_direction`, `gate_relay_control`, `heartbeat`. Candidate integration styles are (a) edge AI camera posting signed webhooks, (b) on-prem LPR server with REST/event stream, (c) gate controller relay via I/O module. **No camera, LPR server, or gate vendor is certified**; each pilot device starts as `unverified-assumption`, becomes `sandbox-tested` after the `docs/09` bench test, and `certified` only after site acceptance at the pilot property. A property that needs automated gates cannot go live on a mock adapter (Section P5).

### F17.1 Facilities, permits and plates

```yaml
- id: M17.F17.1.SF17.1.1
  name: Facility, zones, lanes and capacity
  phase: 3
  release: R1
  actors: [property_admin, security_officer]
  screens: [SCR-ADM-parking-facility]
  inputs: [facility_name, zones(name/capacity/accessible_bays/ev_bays/reserved_types), lanes(direction/device_ids/gate_id), walk_in_reserve, operating_hours]
  states: [draft, active, suspended]
  api: POST /v1/properties/{pid}/parking-facilities
  events: [ParkingFacilityChanged]
  data: [parking_facility, parking_zone, parking_lane, timed_resource]
  rules: ["each zone creates an M09 parking_pool resource", "accessible bays reserved for permit holders with accessibility flag", "capacity reduction below active permits blocked"]
  security: property_admin
  failure_cases: ["lane without device -> manual-only lane flag"]
  finance_report_effect: Parking revenue/cost centers.
  i18n_a11y: Bilingual signage text export.
  acceptance: "Zone P1 capacity 60 with 4 accessible bays creates an M09 pool of 60."
  dependency: M09.

- id: M17.F17.1.SF17.1.2
  name: Permits for guest, corporate, staff and event
  phase: 3
  release: R1
  actors: [front_desk_agent, parking_attendant, event_organizer, hr_officer]
  screens: [SCR-FD-parking-permit, SCR-PARK-permits, SCR-CORP-passes]
  inputs: [permit_type(guest|corporate|staff|event|visitor|valet), holder_ref(reservation/event/employee), plates, zone, valid_from, valid_to, tariff_id, billing(free|folio|master|pay_on_exit), max_vehicles]
  states: [reserved, active, suspended, expired, revoked]
  api: POST /v1/properties/{pid}/parking-permits
  events: [ParkingPermitIssued, ParkingPermitRevoked, ParkingPermitExpired]
  data: [parking_permit, resource_allocation]
  rules: ["event permits drawn from composite hold allocation", "guest permits activate at check-in and expire at checkout + grace (D-236)", "staff permits do not bill guests; linked to employee record without payroll data"]
  security: plate data visible to parking/security/front desk; corporate sees own event permits
  failure_cases: ["zone capacity exhausted -> alternate zone or waitlist", "checkout while vehicle inside -> exit grace then pay-on-exit"]
  finance_report_effect: Permit tariff drives charges.
  i18n_a11y: Standard.
  acceptance: "AT-G02.6 - 20 event permits reserved via composite; the 21st is refused; room move keeps the guest permit."
  dependency: M09, M05, M12.

- id: M17.F17.1.SF17.1.3
  name: Plate registry and privacy
  phase: 3
  release: R1
  actors: [guest, front_desk_agent, dpo]
  screens: [SCR-GST-vehicle, SCR-FD-parking-permit]
  inputs: [plate_text, country_region, plate_script(latin|arabic|dual), vehicle_desc_optional, consent_notice_version]
  states: [registered, verified_by_observation, removed]
  api: POST /v1/properties/{pid}/plate-registrations
  events: [PlateRegistered, PlateRemoved]
  data: [plate_registration, consent_record]
  rules: ["plates normalized - uppercase, Arabic-Indic to Western digits map, Arabic letters transliteration table per country, strip separators", "notice shown at registration and site signage; lawful basis per M44 rule pack", "retention - registrations deleted N days after permit end (D-235), sessions kept per finance retention without images"]
  security: plate is personal data; field-level access; export logged
  failure_cases: ["duplicate plate active on two holders -> conflict flag for staff"]
  finance_report_effect: none.
  i18n_a11y: Input supports Arabic keyboard; normalized preview shown.
  acceptance: "Plate entered as 'أ ب ج ١٢٣٤' and observed as 'ABJ1234' normalize to the same key per Oman fixture table."
  dependency: M44, M02.

- id: M17.F17.1.SF17.1.4
  name: Tariffs
  phase: 3
  release: R1
  actors: [revenue_manager, financial_controller]
  screens: [SCR-ADM-parking-tariffs]
  inputs: [tariff_name, basis(per_entry|per_hour|per_day|per_night|flat), bands, grace_minutes, daily_cap, tax_category, valid_from]
  states: [draft, active, superseded]
  api: POST /v1/properties/{pid}/parking-tariffs
  events: [ParkingTariffChanged]
  data: [parking_tariff]
  rules: ["effective-dated; session priced with tariff at entry", "tax per M44 product treatment for parking"]
  security: finance approval
  failure_cases: ["overlapping versions -> rejected"]
  finance_report_effect: Parking revenue and tax.
  i18n_a11y: Standard.
  acceptance: "A session spanning a tariff change uses the entry tariff version."
  dependency: M44.
```

### F17.2 Devices, observations and confidence

```yaml
- id: M17.F17.2.SF17.2.1
  name: LPR device registry and adapter binding
  phase: 3
  release: R1
  actors: [integration_admin, it_admin]
  screens: [SCR-ADM-lpr-devices]
  inputs: [device_type(edge_camera|lpr_server|gate_controller), vendor_model, firmware, lane_id, adapter_id, capability_flags, auth(mtls_cert|hmac_key_ref), honesty_label]
  states: [registered, sandbox_tested, certified, degraded, offline, retired]
  api: POST /v1/properties/{pid}/lpr-devices
  events: [LprDeviceRegistered, LprDeviceStatusChanged]
  data: [lpr_device]
  rules: ["device identity via M64 certificate or HMAC key in vault", "unsupported capabilities are unavailable, never simulated", "honesty label progression requires evidence (bench test id, site test id)"]
  security: device credentials rotated; network segment isolated (M64)
  failure_cases: ["cert expiry -> degraded and alert 30 days prior"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "A device without gate_relay_control shows lane as 'manual gate' and no auto-open control."
  dependency: M64, docs/05 INT-LPR.

- id: M17.F17.2.SF17.2.2
  name: Observation ingest
  phase: 3
  release: R1
  actors: [lpr_ingest_worker]
  screens: [SCR-PARK-lane-live]
  inputs: [device_id, lane_id, direction, observed_at, plate_text_raw, plate_region, confidence(0-1), image_ref, vendor_event_id, signature]
  states: [received, normalized, matched, unmatched, duplicate, rejected]
  api: POST /v1/properties/{pid}/lpr/observations
  events: [PlateObserved]
  data: [plate_observation]
  rules: ["verify signature/mTLS; reject unsigned", "dedup by (device_id, vendor_event_id) and by same normalized plate in same lane within debounce window (D-237)", "clock skew beyond 5 s -> use server receive time and flag", "images stored in object storage with short retention; observation keeps pointer"]
  security: images access-restricted to security roles; not used for AI training
  failure_cases: ["burst of duplicates -> single observation", "invalid signature -> rejected and security alert"]
  finance_report_effect: none directly.
  i18n_a11y: none (system).
  acceptance: "Same vendor_event_id posted 3 times creates 1 observation."
  dependency: SF17.2.1.

- id: M17.F17.2.SF17.2.3
  name: Confidence thresholds and matching
  phase: 3
  release: R1
  actors: [integration_admin, security_officer]
  screens: [SCR-ADM-lpr-thresholds]
  inputs: [lane_id, auto_accept_threshold, review_threshold, fuzzy_match_rules(max_edit_distance/confusable_pairs)]
  states: [active]
  api: PUT /v1/properties/{pid}/parking-lanes/{lid}/thresholds
  events: [LprThresholdChanged]
  data: [parking_lane, plate_observation]
  rules: ["confidence >= auto_accept and exact match to active permit -> auto decision", "review_threshold <= confidence < auto_accept or fuzzy match (e.g. O/0, B/8) -> human review", "below review_threshold -> treated as no-read -> ticket/manual path", "thresholds tuned from pilot data; defaults labelled unverified-assumption (D-234)"]
  security: threshold changes audited, security_officer approval
  failure_cases: ["fuzzy match to two permits -> human review, never auto"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Observation 'A8C123' at 0.93 vs permit 'ABC123' routes to human review, not auto-open."
  dependency: pilot bench data.

- id: M17.F17.2.SF17.2.4
  name: Low-confidence human review queue
  phase: 3
  release: R1
  actors: [parking_attendant, security_officer]
  screens: [SCR-PARK-review-queue, SCR-STF-parking-review]
  inputs: [observation_id, image_crop, candidate_permits, decision(confirm_permit|correct_plate|deny|visitor_ticket), corrected_plate]
  states: [pending, decided, timed_out]
  api: POST /v1/properties/{pid}/lpr/observations/{oid}/review
  events: [ObservationReviewed]
  data: [observation_review, plate_observation, gate_decision]
  rules: ["lane review SLA seconds (D-237); on timeout lane falls back to ticket/intercom manual path", "reviewer sees image crop and candidate permits only", "corrections stored as labelled data for threshold tuning within privacy rules"]
  security: reviewer identity logged; image access logged
  failure_cases: ["no reviewer online -> manual fallback lane mode"]
  finance_report_effect: none.
  i18n_a11y: Keyboard shortcuts for confirm/deny; image alt text 'plate image'.
  acceptance: "AT-G03.2 - a low-confidence read is confirmed by an attendant within SLA, opening the gate once with reviewer recorded."
  dependency: M64 staff app.

- id: M17.F17.2.SF17.2.5
  name: Device health and outage mode
  phase: 3
  release: R1
  actors: [it_admin, security_officer, device_health_worker]
  screens: [SCR-PARK-device-health, SCR-OPS-exception-queue]
  inputs: [heartbeat, last_observation_at, error_codes]
  states: [healthy, degraded, offline]
  api: GET /v1/properties/{pid}/lpr-devices/health
  events: [LprDeviceOffline, LprDeviceRecovered]
  data: [lpr_device]
  rules: ["missing heartbeat > 60 s -> offline; lane switches to manual fallback (SF17.3.4)", "on-prem profile keeps lane decisions local when internet is down (edge decision cache of active permits signed and time-limited)"]
  security: standard
  failure_cases: ["cloud outage in SaaS profile -> edge cache valid for configured hours then manual"]
  finance_report_effect: Outage sessions flagged for reconciliation.
  i18n_a11y: Standard.
  acceptance: "Unplugging the camera flips lane to manual within 60 s and alerts security."
  dependency: M64 SF64.1.3, SF64.1.5.
```

### F17.3 Gate decisions and sessions

```yaml
- id: M17.F17.3.SF17.3.1
  name: Gate decision engine
  phase: 3
  release: R1
  actors: [gate_decision_worker]
  screens: [SCR-PARK-lane-live]
  inputs: [observation_id, lane_id, direction, permits, zone_occupancy, blacklist]
  states: [allow, deny, review, ticket]
  api: internal ; GET /v1/properties/{pid}/gate-decisions/{gdid}
  events: [GateDecisionMade]
  data: [gate_decision, parking_permit, parking_session]
  rules: ["entry allow if active permit for plate, zone not full (except reserved permit bays), not already inside (anti-passback)", "exit allow if session paid/covered or billing to folio/master allowed", "decision latency target < 1 s p95 locally", "every decision stores inputs snapshot and rule version"]
  security: decision logic server/edge only; devices cannot self-authorize
  failure_cases: ["permit lookup unavailable -> edge cache; else review/manual"]
  finance_report_effect: none directly.
  i18n_a11y: Lane display messages bilingual.
  acceptance: "Guest with active permit and clear read gets 'allow' in < 1 s on the bench fixture."
  dependency: SF17.2.3.

- id: M17.F17.3.SF17.3.2
  name: Gate command and acknowledgment
  phase: 3
  release: R1
  actors: [gate_decision_worker, parking_attendant]
  screens: [SCR-PARK-lane-live]
  inputs: [gate_decision_id, gate_id, command(open|hold_open|close), command_id]
  states: [sent, acknowledged, failed, timed_out]
  api: POST /v1/properties/{pid}/gates/{gid}/commands
  events: [GateCommandSent, GateCommandAcknowledged, GateCommandFailed]
  data: [gate_command]
  rules: ["one command per decision, idempotent by command_id", "no ack within 3 s -> retry once then alert attendant; never loop", "gate controller vendor protocol via adapter"]
  security: command channel authenticated; attendant manual open is a separate audited command
  failure_cases: ["relay stuck -> alert and manual"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "AT-G03.1 - tested gate opens once for an authorized vehicle; duplicate decision does not send a second open."
  dependency: Gate controller adapter (unverified-assumption until pilot).

- id: M17.F17.3.SF17.3.3
  name: Entry/exit session pairing
  phase: 3
  release: R1
  actors: [parking_session_worker]
  screens: [SCR-PARK-sessions]
  inputs: [entry_decision_id, exit_decision_id, plate_key]
  states: [open, closed, orphan_entry, orphan_exit, manually_closed]
  api: GET /v1/properties/{pid}/parking-sessions
  events: [ParkingSessionOpened, ParkingSessionClosed, ParkingSessionOrphaned]
  data: [parking_session]
  rules: ["one open session per plate per facility", "exit without entry -> orphan_exit with review", "entry without exit beyond permit end + grace -> orphan_entry exception"]
  security: standard
  failure_cases: ["misread at exit -> review and manual pairing"]
  finance_report_effect: Duration basis for charges.
  i18n_a11y: Standard.
  acceptance: "Entry and exit of the same plate create one closed session with duration."
  dependency: none.

- id: M17.F17.3.SF17.3.4
  name: Manual fallback and attendant override
  phase: 3
  release: R1
  actors: [parking_attendant, security_officer, duty_manager]
  screens: [SCR-PARK-manual-lane, SCR-STF-parking-manual]
  inputs: [lane_id, plate_typed, permit_lookup, reason(device_offline|misread|emergency|vip|other), photo_optional]
  states: [manual_entry, manual_exit, override_open]
  api: POST /v1/properties/{pid}/parking-lanes/{lid}/manual-events
  events: [ParkingManualEvent, GateOverrideOpened]
  data: [parking_session, gate_command, gate_decision]
  rules: ["manual events create the same sessions and charges as automated ones", "override open requires reason; emergency open has no delay but is reviewed after", "override frequency reported to M60"]
  security: attendant PIN; supervisor approval for non-emergency overrides beyond limit
  failure_cases: ["offline staff app -> local log synced later with dedup"]
  finance_report_effect: Manual sessions billed identically; override report.
  i18n_a11y: Large controls usable with gloves; bilingual.
  acceptance: "With camera offline, attendant manual entry creates a session that bills to folio once at exit."
  dependency: M60, M64 SF64.1.5.

- id: M17.F17.3.SF17.3.5
  name: Live occupancy and anti-passback
  phase: 3
  release: R1
  actors: [parking_attendant, security_officer]
  screens: [SCR-PARK-occupancy]
  inputs: [facility_id]
  states: [available, near_full, full]
  api: GET /v1/properties/{pid}/parking-facilities/{fid}/occupancy
  events: [ParkingZoneFull]
  data: [parking_session, parking_zone]
  rules: ["occupancy = open sessions per zone; corrected by periodic manual count", "full zone denies walk-ins but admits reserved permits into reserved bays"]
  security: standard
  failure_cases: ["drift between count and sessions -> reconciliation task"]
  finance_report_effect: Utilization KPI.
  i18n_a11y: Standard.
  acceptance: "Manual count 55 vs sessions 58 creates reconciliation task listing 3 oldest open sessions."
  dependency: none.
```

### F17.4 Charging and reconciliation

```yaml
- id: M17.F17.4.SF17.4.1
  name: Tariff calculation
  phase: 3
  release: R1
  actors: [parking_session_worker]
  screens: [SCR-PARK-sessions]
  inputs: [parking_session_id, tariff_version, permit_billing]
  states: [priced]
  api: GET /v1/properties/{pid}/parking-sessions/{psid}/price
  events: [ParkingSessionPriced]
  data: [parking_session, parking_tariff]
  rules: ["grace, bands and caps applied deterministically", "free entitlements price at zero but still record session"]
  security: standard
  failure_cases: ["missing tariff -> manual price with approval"]
  finance_report_effect: Parking revenue amount.
  i18n_a11y: Standard.
  acceptance: "3 h 10 min with 15 min grace and hourly band charges 3 hours."
  dependency: SF17.1.4.

- id: M17.F17.4.SF17.4.2
  name: One-time folio or master posting
  phase: 3
  release: R1
  actors: [parking_posting_worker]
  screens: [SCR-FD-folio]
  inputs: [parking_session_id_or_nightly_permit_charge, folio_or_master_id]
  states: [pending, posted, exception]
  api: POST /v1/properties/{pid}/folios/{fid}/charges
  events: [FolioChargePosted, ParkingPostingException]
  data: [folio_charge, parking_session, parking_permit]
  rules: ["per-night permits post nightly in night audit; per-session tariffs post at session close", "unique key (source_type=parking_session|parking_permit_night, source_id) (INV-FOL-1)", "guest checked out -> late charge queue, not silent drop"]
  security: service identity
  failure_cases: ["folio closed -> exception queue", "retry after timeout -> no duplicate"]
  finance_report_effect: Parking revenue and tax; guest ledger.
  i18n_a11y: Folio line bilingual.
  acceptance: "AT-G03.4 - room and parking charges each post once across night audit rerun and replayed session-closed events."
  dependency: M08.

- id: M17.F17.4.SF17.4.3
  name: Corporate and event billing and free entitlements
  phase: 3
  release: R1
  actors: [ar_clerk, sales_manager]
  screens: [SCR-EVT-billing-setup]
  inputs: [event_id, permit_ids, billing_rule]
  states: [posted]
  api: POST /v1/properties/{pid}/master-accounts/{mid}/charges
  events: [FolioChargePosted]
  data: [folio_charge, parking_permit]
  rules: ["event permits billed per contract - per pass or per actual use", "unused passes billed if contract says so"]
  security: standard
  failure_cases: ["contract ambiguity -> manual review"]
  finance_report_effect: Event parking revenue.
  i18n_a11y: Standard.
  acceptance: "20 passes billed per pass post one aggregated line with 20 source refs."
  dependency: M12.

- id: M17.F17.4.SF17.4.4
  name: Pay on exit
  phase: 5
  release: R1
  actors: [guest, parking_attendant]
  screens: [SCR-PARK-pay-station, SCR-GST-parking-pay]
  inputs: [plate_or_ticket, amount, payment_method]
  states: [due, paid, failed]
  api: POST /v1/properties/{pid}/parking-sessions/{psid}/payments
  events: [ParkingSessionPaid]
  data: [payment_intent, parking_session]
  rules: ["payment via M28 (link/QR or terminal)", "exit decision allows after confirmed payment", "cash at attendant booth tied to cashier shift"]
  security: no card data in parking system
  failure_cases: ["payment timeout -> attendant assist; no double charge on retry"]
  finance_report_effect: Parking revenue; tender.
  i18n_a11y: Pay page accessible, bilingual.
  acceptance: "Visitor pays via QR link; exit gate opens once after PSP confirmation."
  dependency: M28 certified gateway.

- id: M17.F17.4.SF17.4.5
  name: Daily parking reconciliation
  phase: 3
  release: R1
  actors: [night_auditor, security_officer, parking_recon_worker]
  screens: [SCR-PARK-reconciliation, SCR-OPS-exception-queue]
  inputs: [business_date]
  states: [clean, exceptions]
  api: POST /v1/properties/{pid}/parking-reconciliations
  events: [ParkingReconciliationCompleted]
  data: [parking_session, gate_decision, gate_command, folio_charge]
  rules: ["checks - sessions without charge, charges without session, orphan entries/exits, overrides, device outage windows, manual count vs sessions", "each exception gets owner and SLA"]
  security: standard
  failure_cases: ["job failure -> night audit warning"]
  finance_report_effect: Revenue leakage metric.
  i18n_a11y: Standard.
  acceptance: "AT-G20.9 - after a simulated outage, reconciliation lists every manual session and confirms each billed once."
  dependency: M08 night audit.

- id: M17.F17.4.SF17.4.6
  name: Plate data and image retention
  phase: 3
  release: R1
  actors: [dpo, retention_worker]
  screens: [SCR-ADM-retention-policies]
  inputs: [image_retention_days, observation_retention_days, session_retention_years]
  states: [active]
  api: PUT /v1/properties/{pid}/parking/retention
  events: [ParkingDataPurged]
  data: [plate_observation, parking_session, plate_registration]
  rules: ["images purged first (short), observations next, sessions kept for finance retention without image", "legal hold for incidents (M42) suspends purge for linked records", "purge proof logged"]
  security: dpo approval to change
  failure_cases: ["purge job failure -> alert; retries"]
  finance_report_effect: none.
  i18n_a11y: Standard.
  acceptance: "Images older than policy are deleted with proof; an incident-linked image under legal hold remains."
  dependency: M02, M44, M42.
```

**M17 key invariants:** a device never authorizes itself; unsupported device capabilities are unavailable; a low-confidence or ambiguous read is never auto-opened; one open session per plate per facility; each session/permit-night posts once; manual fallback yields the same sessions and charges.

**M17 module acceptance:** AT-G02.6 (20 passes cannot double sell), AT-G03.1–G03.2 (tested gate, low-confidence review), AT-G03.4 (charges post once), AT-G20.9 (outage reconciliation).

**M17 open decisions**
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-234 | Default auto-accept and review confidence thresholds | security_officer / integration_admin | auto ≥ 0.95 exact match; review 0.70–0.95 or fuzzy; tuned on pilot data |
| D-235 | Plate and image retention per market | dpo / compliance_officer | Images 30 days; observations 90 days; sessions per finance retention without images |
| D-236 | Guest permit grace after checkout | front_office_manager | 2 h |
| D-237 | Debounce window and lane review SLA | security_officer | 10 s debounce; 20 s review SLA then manual |
| D-238 | Pilot camera/LPR server and gate controller models | it_admin / gm | None selected; `unverified-assumption`; mock adapter for development only |
