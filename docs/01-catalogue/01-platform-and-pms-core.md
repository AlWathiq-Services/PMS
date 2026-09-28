# 01-catalogue / 01 — Platform and PMS core (M01–M08)

**Pack:** MetriStay Hospitality Suite Phase 1 planning pack v0.1 (draft for review) • **Date:** 2026-09-28
**Governing source:** master prompt v3.0 Sections A, B, C (rows M01–M08), D, G, O, P. Conventions: `docs/README.md` §3.
**Status:** specification / design target. No code exists. Nothing here asserts that a partner contract, certification, government API or legal clearance exists; every such dependency carries an honesty label (`unverified-assumption`, `source-cited`, `counsel-reviewed`, `partner-contracted`, `sandbox-tested`, `certified`, `blocked`).

## 0. How to read this file

- Every subfeature (SF) is a Section-L YAML block with all 18 fields. `api` holds one or more `METHOD path` pairs separated by `;`. Every mutating call that moves money, stock, inventory or an external order requires the `Idempotency-Key` header (README §3.5); `(idem)` marks this explicitly.
- Events follow the README envelope and are published through the transactional outbox (SF01.6.6). Consumers dedup on `event_id` in their inbox.
- `phase` is the first implementation phase; `release: R1` means part of Customer Release 1 (Phases 2–6).
- Money = integer minor units + ISO-4217; times are UTC + property IANA zone; `business_date` is a separate field on every operational and financial record (README §3.8).
- Decision ids in this file use **D-101..D-199**.

### 0.1 Shared entity glossary defined by this file (other catalogue files MUST reuse these names)

| Entity | Owner module / context | Meaning |
|---|---|---|
| `tenant`, `property`, `department`, `outlet` | M01 `platform` | Organizational hierarchy. `outlet` is a revenue/cost point (bar, club, restaurant, parking) inside a department. |
| `business_day` | M01 `platform` | One row per property per hotel business date: `{property_id, business_date, state: open/closing/closed/reopened, opened_at, closed_at, night_audit_run_id}`. Exactly one `open` row per property. |
| `feature_flag`, `activation_gate`, `config_version` | M01 | Feature activation, jurisdiction/partner gates, versioned settings. |
| `device_identity`, `secret_ref`, `audit_log`, `outbox_message`, `inbox_message`, `backup_run`, `restore_test`, `migration_run`, `locale_bundle` | M01 | Platform services used by all modules. |
| `principal` (`staff_user`, `guest_account`, `corporate_user`, `vendor_user`, `service_account`), `role`, `role_assignment` (scope), `delegation`, `approval_request`, `privileged_session` | M02 `iam` | Identities and scoped permissions. |
| `consent_record`, `consent_purpose`, `retention_policy`, `legal_hold`, `data_subject_request` | M02 `iam` | Purpose/channel consent, retention and DSR. |
| `building`, `floor`, `room_type`, `room`, `room_attribute`, `room_connection` | M03 `inventory` | Physical and saleable structure. |
| `room_type_inventory_night` | M03 | Sold stock per `{property_id, room_type_id, stay_date}`: `physical, out_of_order, held, sold, allotted, overbook_limit, available (derived)`. The ONLY authority for "can this be sold". |
| `inventory_hold`, `room_status_block` (OOO/OOS), `stop_sell`, `allotment`, `overbooking_policy`, `room_assignment` | M03 | Holds with expiry, room blocks, restrictions, allotments, controlled overbooking, and room-to-reservation assignment (separate from sold stock). |
| `rate_plan`, `rate_price_night`, `rate_rule`, `restriction`, `package`, `promotion`, `quote`, `quote_line_night`, `policy_snapshot`, `price_override` | M04 `rates` | Pricing and quotes. Tax computation is delegated to M38 `tax_rule_version` via port. |
| `reservation`, `reservation_room` (one room-type stay segment), `stay`, `reservation_party` (role: booker/occupant/payer/primary_guest), `guest_profile`, `waitlist_entry`, `registration_record`, `room_move`, `device_entitlement` | M05 `reservations` | Booking and in-house stay. `guest_profile` is the single guest master (M52 enriches it, never duplicates it). |
| `hk_room_status`, `hk_task`, `hk_inspection`, `dnd_status`, `offline_change_set`, `room_ready_estimate`, `minibar_posting_request`, `found_item_ref` | M06 `frontoffice` | Housekeeping and front-desk operational state. |
| `channel_connection`, `channel_mapping`, `ari_message`, `channel_booking_message`, `dead_letter`, `channel_reconciliation_run`, `source_attribution`, `commission_accrual`, `funnel_event` | M07 `distribution` | Distribution and attribution. |
| `folio`, `folio_window`, `folio_line` (type: charge/tax/payment/reversal/transfer_out/transfer_in/adjustment), `routing_instruction`, `deposit_ledger_entry`, `payment_authorization_ref`, `fiscal_document` (invoice/credit_note/receipt), `cashier_shift`, `cash_movement`, `night_audit_run`, `finance_export_batch` | M08 `folio` | Append-only guest/account ledger and cashiering. Card data never enters these tables; only M28 tokens/refs. |

## Module index

| Module | Features | Subfeatures | First phase |
|---|---|---|---|
| M01 Platform/deployment | F01.1–F01.6 | 32 | 2 |
| M02 IAM and consent | F02.1–F02.5 | 27 | 2 |
| M03 Room inventory | F03.1–F03.5 | 26 | 2 |
| M04 Rates and quote engine | F04.1–F04.5 | 26 | 2 |
| M05 Reservations/stays | F05.1–F05.5 | 30 | 2 |
| M06 Front desk/housekeeping | F06.1–F06.4 | 23 | 2 |
| M07 Distribution | F07.1–F07.5 | 26 | 2 |
| M08 Folio/cashiering | F08.1–F08.6 | 32 | 2 |
| **Total** | **41 features** | **222** | |

---

# M01 — Platform/deployment

| Attribute | Value |
|---|---|
| Purpose | Provide the tenant/property/department structure, hotel clock and business date, deployment profiles (SaaS and single-hotel on-prem), configuration and activation gates, backup/restore and migrations, English/Arabic RTL and accessibility foundations, observability, secrets, device identity, audit log and the outbox/inbox messaging backbone every other module depends on. |
| Build phase(s) | 2 (foundation, all SFs); 3–6 hardening (DR drills, site device profiles, performance); 7 multi-property extensions. |
| Release flag | R1 (portfolio/chain features Later). |
| Bounded context | `platform` (schema `platform`). |
| Systems of record owned | `tenant`, `property`, `department`, `outlet`, `business_day`, `feature_flag`, `activation_gate`, `config_version`, `deployment_profile`, `device_identity`, `secret_ref` (metadata only — secret material lives in Vault/KMS), `audit_log`, `outbox_message`, `inbox_message`, `backup_run`, `restore_test`, `migration_run`, `locale_bundle`. |
| Upstream dependencies | None (foundation). Uses M44 activation results (jurisdiction gates) and M02 for authorization of admin actions. |
| Downstream dependents | All modules M02–M68. Business date consumed by M05, M06, M08, M13, M19, M32, M60; outbox by every event producer; M64 extends device/resilience. |
| External dependencies | Keycloak (OIDC/SAML) — technology choice, not partner; Vault/KMS; S3-compatible storage (MinIO on-prem); offsite backup target — `unverified-assumption` until the pilot hotel selects a provider (D-104). |

## F01.1 Tenancy, property and organization structure

```yaml
id: M01.F01.1.SF01.1.1
name: Provision tenant
phase: 2
release: R1
actors: [tenant_admin, it_admin]
screens: [SCR-ADM-tenant-setup, SCR-ADM-tenant-list]
inputs: [tenant_legal_name, tenant_slug, deployment_profile, primary_region, default_locale, billing_contact, idp_realm]
states: [draft, provisioning, active, suspended, offboarding, closed]
api: POST /v1/tenants (idem); PATCH /v1/tenants/{tid}; POST /v1/tenants/{tid}/suspend
events: [TenantProvisioned, TenantSuspended, TenantOffboardingStarted]
data: [tenant, deployment_profile, audit_log]
rules:
  - tenant_slug unique, immutable after active; idp realm created once per tenant (SaaS) or one realm per install (on-prem).
  - Suspension blocks staff/guest writes but never blocks read-only export or required financial retention.
  - Provisioning is a saga; partial failure rolls back realm and schema grants and leaves tenant in draft.
security: platform operator role only (MetriSys staff) in SaaS; on-prem installer uses bootstrap token valid 24h; all actions audited with step-up MFA.
failure_cases: [idp_realm_create_failed, duplicate_slug, region_unavailable, partial_provision_timeout]
finance_report_effect: none directly; tenant is the top reporting boundary for subscription billing (outside hotel GL).
i18n_a11y: tenant default locale en or ar; setup wizard keyboard-operable and RTL-mirrored.
acceptance: Provisioning with a duplicate slug returns 409; a simulated IdP failure mid-saga leaves no orphan realm, schema grant or active tenant row.
dependency: Keycloak admin API (technology choice); ADR on deployment profiles in docs/03.
```

```yaml
id: M01.F01.1.SF01.1.2
name: Create and configure property
phase: 2
release: R1
actors: [tenant_admin, property_admin]
screens: [SCR-ADM-property-profile, SCR-ADM-property-setup-checklist]
inputs: [property_name_en, property_name_ar, legal_entity_id, address, country_code, subdivision_code, municipality_code, iana_time_zone, base_currency, enabled_locales, service_profile, star_category, check_in_time, check_out_time]
states: [draft, configuring, go_live_ready, live, closed_temporarily, decommissioned]
api: POST /v1/properties (idem); PATCH /v1/properties/{pid}; POST /v1/properties/{pid}/go-live
events: [PropertyCreated, PropertyConfigured, PropertyWentLive, PropertyClosedTemporarily]
data: [property, tenant, audit_log]
rules:
  - legal_entity_id and location codes are required and sent to M44 classifier; go-live blocked until M44 returns a classification and each activated feature's gates are resolved or have an approved manual path.
  - iana_time_zone and base_currency are immutable after first business_day opens; change requires a migration project (D-116 related).
  - service_profile (limited/full) seeds default feature flags but never auto-enables a regulated feature.
security: property_admin scoped to own property; tenant_admin for create; field-level audit on legal and tax fields.
failure_cases: [unknown_subdivision, classifier_unavailable, currency_minor_unit_mismatch, go_live_with_unresolved_gate]
finance_report_effect: property is the P&L and tax reporting unit; base_currency drives folio and GL functional currency.
i18n_a11y: bilingual names mandatory for guest-facing fields when ar enabled; address supports RTL entry and Latin transliteration.
acceptance: A property in Oman with OMR base currency stores 3-decimal minor units; go-live is refused while any activated feature has an unresolved M44 gate, and the refusal lists each gate with owner.
dependency: M44 classifier foundation (Phase 2); M38 legal entity registry.
```

```yaml
id: M01.F01.1.SF01.1.3
name: Departments, outlets and cost-center links
phase: 2
release: R1
actors: [property_admin, financial_controller]
screens: [SCR-ADM-departments, SCR-ADM-outlets]
inputs: [department_code, department_name_en, department_name_ar, outlet_code, outlet_type, cost_center_code, revenue_center_flag, operating_hours]
states: [active, inactive]
api: POST /v1/properties/{pid}/departments; POST /v1/properties/{pid}/outlets; PATCH /v1/properties/{pid}/outlets/{oid}
events: [DepartmentChanged, OutletChanged]
data: [department, outlet, audit_log]
rules:
  - Each outlet belongs to exactly one department and maps to one M19 cost center (mapping may be pending before Phase 4 but must exist before finance export).
  - An outlet with posted transactions cannot be deleted, only inactivated.
  - Department is also an authorization scope dimension for M02.
security: financial_controller approves cost-center mapping changes (maker-checker).
failure_cases: [duplicate_code, outlet_without_cost_center_at_export, inactivate_with_open_folio_lines]
finance_report_effect: department/outlet are dimensions on every folio_line and GL line; drive departmental P&L (M32).
i18n_a11y: bilingual labels; codes are locale-neutral.
acceptance: Posting to an inactive outlet is rejected with a clear code; finance export fails visibly listing outlets lacking a cost-center mapping.
dependency: M19 chart of accounts (Phase 4) — interim placeholder mapping flagged estimate.
```

```yaml
id: M01.F01.1.SF01.1.4
name: Property activation profile
phase: 2
release: R1
actors: [property_admin, gm]
screens: [SCR-ADM-activation-profile]
inputs: [enabled_modules, enabled_outlets, enabled_integrations, service_profile]
states: [not_enabled, enabled_pending_gate, enabled, disabled]
api: GET /v1/properties/{pid}/activation; PUT /v1/properties/{pid}/activation/{module_code}
events: [ModuleActivationChanged]
data: [feature_flag, activation_gate, property]
rules:
  - Modules not owned by the hotel are hidden from navigation, search and reports (master prompt A, P1).
  - Enabling a module that depends on a partner or rule pack enters enabled_pending_gate until gate evidence is recorded.
  - Disabling keeps data and required retention; it hides entry points and rejects new writes.
security: property_admin with gm approval for revenue-affecting modules; audit.
failure_cases: [enable_with_missing_dependency_module, disable_with_open_transactions]
finance_report_effect: disabled outlets excluded from P&L layout but historical data retained.
i18n_a11y: status labels bilingual; state not conveyed by color alone.
acceptance: Disabling the club module removes club screens from all staff navigation and returns 403 feature_disabled on club APIs while historical club revenue still appears in prior-period reports.
dependency: none.
```

```yaml
id: M01.F01.1.SF01.1.5
name: Tenant and property isolation enforcement
phase: 2
release: R1
actors: [it_admin, auditor]
screens: [SCR-OPS-isolation-test-report]
inputs: [tenant_id, property_id, request_principal]
states: [pass, fail]
api: none public; enforced in every repository and by Postgres RLS; GET /v1/ops/isolation-tests/latest
events: [IsolationTestFailed]
data: [all property-scoped tables with tenant_id and property_id columns, audit_log]
rules:
  - Every property-scoped table carries tenant_id and property_id NOT NULL with RLS policies keyed to session settings set by the API gateway.
  - Object ids are UUIDv7; authorization checks object ownership, never trusts client-supplied property_id alone (OWASP API1 BOLA).
  - CI runs an automated cross-tenant and cross-property probe suite on every build.
security: RLS plus application checks (defense in depth); background workers set scope explicitly per job.
failure_cases: [missing_rls_policy_on_new_table, worker_without_scope, cache_key_without_tenant]
finance_report_effect: prevents cross-property revenue leakage in reports.
i18n_a11y: none (internal).
acceptance: The isolation suite attempts reads and writes of 100% of property-scoped endpoints with another tenant's and another property's ids and receives 404/403 for all; a migration adding a table without an RLS policy fails CI.
dependency: ADR-00x RLS in docs/03.
```

## F01.2 Hotel time, business date and rollover

```yaml
id: M01.F01.2.SF01.2.1
name: Property clock and time-zone handling
phase: 2
release: R1
actors: [property_admin, it_admin]
screens: [SCR-ADM-property-profile, SCR-OPS-time-sync-health]
inputs: [iana_time_zone, ntp_source, tzdata_version]
states: [in_sync, drift_warning, drift_critical]
api: GET /v1/properties/{pid}/clock
events: [ClockDriftDetected]
data: [property, device_identity]
rules:
  - All timestamps stored UTC; display converts with property zone, not viewer's device zone, on staff screens.
  - DST transitions are handled by tzdata; a stay_date is a calendar date label, never computed by adding 24h.
  - On-prem server drift > 2s warns, > 30s blocks card-present payment and fiscal document numbering until resolved (threshold configurable).
security: NTP source configured by it_admin; time-sync health visible to M64.
failure_cases: [ntp_unreachable, tzdata_outdated, device_clock_skew]
finance_report_effect: correct period cut-off for postings near midnight and DST.
i18n_a11y: times shown in property zone with zone abbreviation; 12/24h per locale; Arabic-Indic digits optional per D-121.
acceptance: A check-in at 01:30 local on a DST fall-back night records two distinct UTC instants correctly and both display with the property zone label.
dependency: none.
```

```yaml
id: M01.F01.2.SF01.2.2
name: Business date register and rollover
phase: 2
release: R1
actors: [night_auditor, front_office_manager, night_audit_worker]
screens: [SCR-FIN-night-audit, SCR-FD-business-date-banner]
inputs: [property_id, current_business_date, night_audit_run_id]
states: [open, closing, closed, reopened]
api: GET /v1/properties/{pid}/business-date; POST /v1/properties/{pid}/business-date/rollover (idem)
events: [BusinessDateClosing, BusinessDateRolled, BusinessDateReopened]
data: [business_day, night_audit_run]
rules:
  - Business date is separate from the calendar date; it advances only when night audit (SF08.6.x) completes or an authorized forced rollover occurs.
  - Exactly one business_day per property is open; rollover is a single transaction closing D and opening D+1.
  - While closing, new postings are queued to D+1 only if explicitly flagged late; otherwise posting UI shows the closing banner.
security: rollover requires night_auditor or front_office_manager; forced rollover requires duty_manager step-up MFA and reason.
failure_cases: [concurrent_rollover_attempt, audit_step_failed, rollover_skipped_days]
finance_report_effect: every folio_line, stock and POS record carries business_date; daily flash (M32) keyed on business_date.
i18n_a11y: persistent business-date banner distinct from calendar date, announced to screen readers when it changes.
acceptance: Two concurrent rollover calls produce one BusinessDateRolled event and one 409; a calendar date of 29 Sep 02:00 with business date 28 Sep posts room charges to 28 Sep.
dependency: M08 F08.6 night audit.
```

```yaml
id: M01.F01.2.SF01.2.3
name: Business date stamping and posting windows
phase: 2
release: R1
actors: [front_desk_agent, cashier, system]
screens: [SCR-FD-folio, SCR-FIN-posting-journal]
inputs: [transaction_type, occurred_at, business_date_override_reason]
states: [stamped, late_posted, back_dated]
api: applied in posting services of M05/M06/M08/M13; GET /v1/properties/{pid}/business-days/{date}
events: [LatePostingRecorded]
data: [business_day, folio_line, audit_log]
rules:
  - Default business_date = current open business_day of the property, not the device date.
  - Posting to a closed business date is forbidden; corrections post on the current date with a reference to the original date.
  - Back-dating inside the open date requires reason; POS offline queues keep their original business_date if still open, else post to current with late flag.
security: no role may write to a closed date; reopening (SF08.6.5) is the only path.
failure_cases: [offline_device_posts_after_rollover, clock_skewed_device]
finance_report_effect: late postings visible in night audit exception report and M60 controls.
i18n_a11y: late flag shown as text badge, not only color.
acceptance: An offline POS charge created on business date D and synced after rollover posts on D+1 with late_posted flag and a link to original occurred_at.
dependency: M13 offline queue (Phase 3).
```

```yaml
id: M01.F01.2.SF01.2.4
name: Stuck-audit alert and forced rollover deadline
phase: 2
release: R1
actors: [duty_manager, night_audit_worker, gm]
screens: [SCR-FIN-night-audit, SCR-GM-attention-inbox]
inputs: [rollover_deadline_local, escalation_contacts]
states: [on_time, overdue_warning, overdue_critical, forced]
api: PUT /v1/properties/{pid}/business-date/policy; POST /v1/properties/{pid}/business-date/force-rollover (idem)
events: [NightAuditOverdue, BusinessDateForced]
data: [business_day, config_version]
rules:
  - If business date is not rolled by the configured deadline (default 06:00 local, D-101), escalate to duty_manager then gm.
  - Forced rollover records unfinished audit steps as open exceptions carried to D+1 and marks the day closed_with_exceptions.
security: step-up MFA; reason mandatory.
failure_cases: [no_contact_ack, forced_with_unposted_room_charges]
finance_report_effect: day flagged estimate in daily flash until exceptions resolved.
i18n_a11y: escalations via push/SMS templates in en/ar.
acceptance: With the deadline passed and audit incomplete, an alert reaches duty_manager within 1 minute; forced rollover leaves a visible exception list and the daily flash for that date shows estimate.
dependency: M63 notification engine (Phase 2 engine).
```

```yaml
id: M01.F01.2.SF01.2.5
name: Business date to accounting period mapping
phase: 2
release: R1
actors: [financial_controller]
screens: [SCR-FIN-period-calendar]
inputs: [fiscal_calendar, period_start, period_end]
states: [open, soft_closed, locked]
api: GET /v1/properties/{pid}/accounting-periods; PUT /v1/properties/{pid}/accounting-periods/{id}
events: [AccountingPeriodLocked]
data: [business_day, accounting_period]
rules:
  - Each business_date maps to exactly one accounting period; the last business date of a month maps to that month even if audit runs after midnight.
  - Locked period rejects any finance export that would alter it; corrections flow to the current period (M19 SF19.1.3).
security: financial_controller only.
failure_cases: [gap_in_calendar, overlapping_periods]
finance_report_effect: defines month-end cut-off for room revenue and all departmental revenue.
i18n_a11y: Gregorian primary; optional Hijri display annotation (D-121).
acceptance: Room revenue for business date 31 Oct audited at 02:10 on 1 Nov lands in October.
dependency: M19 F19.1 (Phase 4), foundation table in Phase 2.
```

## F01.3 Deployment profiles, configuration and activation

```yaml
id: M01.F01.3.SF01.3.1
name: SaaS multi-tenant deployment profile
phase: 2
release: R1
actors: [it_admin, tenant_admin]
screens: [SCR-OPS-deployment-status]
inputs: [region, data_residency_requirement, tenant_tier]
states: [planned, deployed, degraded, maintenance]
api: GET /v1/ops/deployment
events: [DeploymentVersionChanged, MaintenanceWindowScheduled]
data: [deployment_profile, tenant]
rules:
  - Kubernetes, shared Postgres cluster with per-tenant RLS; residency constraint may force dedicated database per D-116.
  - Rolling deploys with expand/contract migrations; no downtime for guest booking web.
security: SOC-style change control; per-tenant encryption keys in KMS (envelope).
failure_cases: [region_outage, failed_rollout_auto_rollback]
finance_report_effect: none.
i18n_a11y: maintenance notices in en/ar.
acceptance: A rolling deploy under synthetic booking load drops zero confirmed bookings and the previous version remains readable until contract step.
dependency: Hosting provider selection `unverified-assumption` (D-116).
```

```yaml
id: M01.F01.3.SF01.3.2
name: On-premises single-hotel deployment profile
phase: 2
release: R1
actors: [it_admin, property_admin]
screens: [SCR-OPS-deployment-status, SCR-OPS-onprem-health]
inputs: [host_spec, network_segments, offsite_backup_target, update_channel]
states: [installed, healthy, degraded, wan_offline, upgrade_pending]
api: GET /v1/ops/deployment; POST /v1/ops/upgrades/{version}/apply
events: [OnPremWanLost, OnPremWanRestored, UpgradeApplied]
data: [deployment_profile, backup_run]
rules:
  - Same code and schema as SaaS; Docker Compose or k3s; local Keycloak, MinIO, Postgres, job queue.
  - When WAN is lost, PMS core continues (reservations, folio, housekeeping); external channel, PSP and messaging calls queue in outbox with visible pending state.
  - Minimum host spec and UPS are documented prerequisites, not assumed.
security: disk encryption, local firewall, remote support only via break-glass session approved on-site (SF01.3.6).
failure_cases: [disk_full, wan_loss_during_channel_push, upgrade_failure_rollback]
finance_report_effect: none directly; ensures continuity of posting.
i18n_a11y: admin console bilingual.
acceptance: With WAN disconnected for 2h, staff check in guests and post charges; on restore, queued ARI and payment calls drain in order with zero duplicates.
dependency: pilot hotel hardware/network profile (D-102, D-104).
```

```yaml
id: M01.F01.3.SF01.3.3
name: Feature flags and activation gates
phase: 2
release: R1
actors: [property_admin, compliance_officer, integration_admin]
screens: [SCR-ADM-feature-flags, SCR-ADM-activation-gates]
inputs: [flag_key, scope, gate_type, gate_evidence_ref, honesty_label]
states: [off, on, gated_blocked, gated_manual_path, gated_passed]
api: GET /v1/properties/{pid}/flags; PUT /v1/properties/{pid}/flags/{key}; POST /v1/properties/{pid}/gates/{gate_id}/evidence
events: [FeatureFlagChanged, ActivationGateChanged]
data: [feature_flag, activation_gate, audit_log]
rules:
  - Gate types = release (R1/Later), jurisdiction (M44 rule pack status), partner (honesty label), legal (counsel-reviewed).
  - A gated feature is enforced server-side (API, jobs, adapters), never only in UI.
  - Only honesty label certified/sandbox-tested/partner-contracted can pass a partner gate for production; blocked shows the manual path.
security: compliance_officer must approve legal/jurisdiction gate evidence; maker-checker.
failure_cases: [flag_enabled_but_gate_blocked, gate_evidence_expired]
finance_report_effect: none directly.
i18n_a11y: gate reason text bilingual.
acceptance: With a channel adapter gate at unverified-assumption, the ARI push job refuses to run for that property and the UI shows the manual extranet path; toggling the UI flag does not bypass it.
dependency: M44 rule pack status (Phase 2 foundation).
```

```yaml
id: M01.F01.3.SF01.3.4
name: Versioned configuration with maker-checker and rollback
phase: 2
release: R1
actors: [property_admin, gm]
screens: [SCR-ADM-config-history, SCR-ADM-config-diff]
inputs: [config_domain, change_set, effective_from, reason]
states: [draft, pending_approval, active, superseded, rolled_back]
api: POST /v1/properties/{pid}/config/{domain}/versions; POST .../versions/{v}/approve; POST .../versions/{v}/rollback
events: [ConfigVersionActivated, ConfigVersionRolledBack]
data: [config_version, audit_log]
rules:
  - Sensitive domains (tax mapping, overbooking policy, cancellation policies, cash limits) need a second approver.
  - Transactions record the config_version they were evaluated under.
security: approver cannot be the maker.
failure_cases: [conflicting_concurrent_drafts, rollback_of_config_referenced_by_open_quotes]
finance_report_effect: audit shows which policy version priced or charged a transaction.
i18n_a11y: side-by-side diff accessible with text markers.
acceptance: Rolling back overbooking policy v3 to v2 keeps existing reservations unaffected and new availability checks use v2 within 5 seconds.
dependency: none.
```

```yaml
id: M01.F01.3.SF01.3.5
name: Schema migrations and data upgrades
phase: 2
release: R1
actors: [it_admin]
screens: [SCR-OPS-migrations]
inputs: [migration_id, target_version, dry_run]
states: [pending, running, succeeded, failed, rolled_back]
api: POST /v1/ops/migrations/{id}/run; GET /v1/ops/migrations
events: [MigrationSucceeded, MigrationFailed]
data: [migration_run]
rules:
  - Expand/contract pattern; destructive steps only after all app versions stopped reading old columns.
  - Ledger tables (folio_line, audit_log, stock ledgers) are never rewritten; only additive columns.
  - Every migration has tested rollback or forward-fix plan and a pre-migration backup (on-prem mandatory).
security: it_admin with change ticket reference.
failure_cases: [lock_timeout, long_running_backfill, version_skew]
finance_report_effect: none; protects ledger integrity.
i18n_a11y: none.
acceptance: A failed migration on on-prem restores the pre-migration backup automatically and the application starts on the prior version with no lost committed transactions.
dependency: docs/09 migration and rollback plan.
```

```yaml
id: M01.F01.3.SF01.3.6
name: Update channel, support access and break-glass
phase: 2
release: R1
actors: [it_admin, gm, auditor]
screens: [SCR-OPS-support-access]
inputs: [support_ticket_id, requested_scope, duration]
states: [requested, approved, active, expired, revoked]
api: POST /v1/ops/support-sessions; POST /v1/ops/support-sessions/{id}/approve; DELETE /v1/ops/support-sessions/{id}
events: [SupportSessionOpened, SupportSessionClosed]
data: [privileged_session, audit_log]
rules:
  - Vendor (MetriSys) support has no standing access to hotel data; time-boxed session approved by the hotel.
  - Break-glass access records every query/screen and notifies gm.
security: MFA, IP allowlist, recorded session.
failure_cases: [session_not_revoked_at_expiry]
finance_report_effect: none.
i18n_a11y: notification bilingual.
acceptance: A support session expires at its end time and subsequent requests with its token return 401; the audit shows each accessed object.
dependency: M02 F02.3 privileged access.
```

## F01.4 Backup, restore and continuity

```yaml
id: M01.F01.4.SF01.4.1
name: Encrypted backups with point-in-time recovery
phase: 2
release: R1
actors: [it_admin, backup_worker]
screens: [SCR-OPS-backups]
inputs: [schedule, retention_days, target, encryption_key_ref]
states: [scheduled, running, succeeded, failed, expired]
api: GET /v1/ops/backups; POST /v1/ops/backups (idem)
events: [BackupSucceeded, BackupFailed]
data: [backup_run]
rules:
  - Postgres base backup plus WAL archiving; object storage versioned; config and secrets metadata included, secret values backed up only via Vault's own mechanism.
  - Offsite copy mandatory for on-prem; failure raises critical alert.
security: backups encrypted with keys not stored on the same host.
failure_cases: [offsite_unreachable, backup_corrupt, key_unavailable]
finance_report_effect: none.
i18n_a11y: none.
acceptance: Two consecutive failed offsite uploads raise a critical alert to it_admin and gm; RPO target per D-104 is displayed and measured.
dependency: offsite storage target `unverified-assumption` (D-104).
```

```yaml
id: M01.F01.4.SF01.4.2
name: Restore test to isolated environment
phase: 2
release: R1
actors: [it_admin, auditor]
screens: [SCR-OPS-restore-tests]
inputs: [backup_run_id, target_env, point_in_time]
states: [requested, restoring, verifying, passed, failed]
api: POST /v1/ops/restore-tests (idem); GET /v1/ops/restore-tests/{id}
events: [RestoreTestPassed, RestoreTestFailed]
data: [restore_test, backup_run]
rules:
  - Monthly scheduled restore; verification runs ledger checksums (folio balance totals, stock ledger totals, audit_log hash chain).
  - Restored environment has outbound adapters disabled to prevent duplicate external calls.
security: restored data subject to same access controls; environment destroyed after test.
failure_cases: [checksum_mismatch, adapters_not_disabled]
finance_report_effect: evidence for continuity and audit.
i18n_a11y: none.
acceptance: A restore to point-in-time T reproduces folio balance totals and audit hash-chain head identical to production at T, and zero outbound calls occur.
dependency: M64 F64.2.
```

```yaml
id: M01.F01.4.SF01.4.3
name: SaaS disaster recovery failover
phase: 3
release: R1
actors: [it_admin]
screens: [SCR-OPS-dr-status]
inputs: [dr_region, rpo_target, rto_target]
states: [standby, failing_over, active_dr, failing_back]
api: POST /v1/ops/dr/failover (idem)
events: [DrFailoverStarted, DrFailoverCompleted]
data: [deployment_profile, backup_run]
rules:
  - Measured RPO/RTO published; outbox messages replayed with idempotency keys after failover.
security: dual authorization for failover.
failure_cases: [split_brain, replication_lag_exceeds_rpo]
finance_report_effect: transactions within RPO window reconciled via PSP/channel reconciliation (M07, M28).
i18n_a11y: status page bilingual.
acceptance: A DR drill meets agreed RTO and all external callbacks received during failover are processed exactly once.
dependency: hosting provider (D-116).
```

```yaml
id: M01.F01.4.SF01.4.4
name: Local survivability and store-and-forward
phase: 2
release: R1
actors: [front_desk_agent, housekeeper, outbox_relay]
screens: [SCR-FD-connectivity-banner, SCR-OPS-queue-depth]
inputs: [queue_name, message]
states: [online, degraded, offline, draining]
api: GET /v1/properties/{pid}/connectivity
events: [ConnectivityDegraded, QueueDrained]
data: [outbox_message, offline_change_set]
rules:
  - External calls are always asynchronous via outbox; the UI shows pending external status instead of spinning.
  - Mobile staff apps keep an offline change set with per-record version for conflict resolution (SF06.3.x).
security: offline data on devices encrypted; wiped on device revocation.
failure_cases: [queue_overflow, poison_message]
finance_report_effect: pending external payment/channel calls appear as unreconciled items.
i18n_a11y: banner announced via aria-live.
acceptance: During a simulated outage, UI displays offline banner within 10 seconds and all queued messages are delivered in order after restoration.
dependency: M64.
```

```yaml
id: M01.F01.4.SF01.4.5
name: Tenant data export and offboarding
phase: 2
release: R1
actors: [tenant_admin, dpo, financial_controller]
screens: [SCR-ADM-data-export]
inputs: [export_scope, format, legal_hold_check]
states: [requested, approved, generating, ready, downloaded, expired]
api: POST /v1/tenants/{tid}/exports (idem); GET /v1/tenants/{tid}/exports/{id}
events: [TenantExportReady]
data: [tenant, legal_hold, audit_log]
rules:
  - Export includes ledgers, fiscal documents and audit logs in open formats (CSV/JSON/PDF).
  - Deletion after offboarding respects retention and legal holds (M02 F02.5).
security: dual approval; download link short-lived, MFA.
failure_cases: [export_too_large, active_legal_hold]
finance_report_effect: preserves statutory records.
i18n_a11y: documents keep original language.
acceptance: Offboarding export contains every fiscal document number with no gaps and the tenant cannot be purged while a legal hold exists.
dependency: M02 F02.5.
```

## F01.5 Localization (English/Arabic RTL) and accessibility

```yaml
id: M01.F01.5.SF01.5.1
name: Locale bundles and RTL layout
phase: 2
release: R1
actors: [content_editor, property_admin, guest, front_desk_agent]
screens: [SCR-ADM-translations, all SCR-* screens]
inputs: [locale, message_key, translation, reviewer]
states: [missing, machine_draft, reviewed, published]
api: GET /v1/locales/{locale}/bundle; PUT /v1/properties/{pid}/translations/{key}
events: [TranslationPublished]
data: [locale_bundle]
rules:
  - en and ar are mandatory initial locales; ar renders dir=rtl with mirrored layout and logical CSS properties.
  - Untranslated guest-facing legal text (policies, consent) may not be published in a locale until reviewed.
  - Language packs for further jurisdictions are additive (P1).
security: content_editor edits, content_approver publishes legal text.
failure_cases: [missing_key_fallback, bidi_mixed_content]
finance_report_effect: none.
i18n_a11y: bidi isolation for guest names, numbers and codes inside Arabic sentences.
acceptance: Every R1 screen passes an automated RTL snapshot test; a missing ar key falls back to en and is logged as a translation gap, while missing ar consent text blocks publication.
dependency: none.
```

```yaml
id: M01.F01.5.SF01.5.2
name: Multilingual master data fields
phase: 2
release: R1
actors: [property_admin, content_editor]
screens: [SCR-ADM-room-types, SCR-ADM-rate-plans, SCR-ADM-policies]
inputs: [field_key, value_by_locale]
states: [complete, incomplete]
api: embedded in owning resources (e.g. PATCH /v1/properties/{pid}/room-types/{id})
events: none
data: [localized_text columns (jsonb by locale) on owning entities]
rules:
  - Guest-facing names/descriptions stored per locale; guest personal names stored as entered plus optional Latin transliteration.
  - Search indexes both scripts and normalizes Arabic letter variants (alef, taa marbuta, diacritics).
security: none beyond owning resource.
failure_cases: [search_miss_due_to_normalization]
finance_report_effect: invoices use the locale chosen per fiscal rule (M38).
i18n_a11y: form fields accept either script and preserve direction.
acceptance: Searching guest "احمد" finds a profile stored as "أحمد", and searching "Ahmed" finds it via transliteration.
dependency: none.
```

```yaml
id: M01.F01.5.SF01.5.3
name: Number, date and currency formatting
phase: 2
release: R1
actors: [guest, front_desk_agent, finance_clerk]
screens: [all monetary and date displays]
inputs: [locale, currency, minor_units]
states: [none]
api: none (shared formatting library); GET /v1/currencies
events: none
data: [currency_reference]
rules:
  - Money is integer minor units; exponent from ISO-4217 (OMR 3, SAR 2, CAD 2, PKR 2, EUR 2).
  - Display rounding never changes stored values; rounding rules for tax come from M38.
  - Digits Western by default; Arabic-Indic optional per locale setting (D-121).
security: none.
failure_cases: [float_arithmetic_introduced, wrong_exponent]
finance_report_effect: prevents rounding drift in folios and reports.
i18n_a11y: currency symbol position per locale; screen readers read full currency name.
acceptance: OMR 12.345 is stored as 12345 and displayed as "12.345 ر.ع." in ar and "OMR 12.345" in en; a lint rule rejects floating-point money types.
dependency: none.
```

```yaml
id: M01.F01.5.SF01.5.4
name: Accessibility baseline (WCAG 2.2 AA)
phase: 2
release: R1
actors: [guest, all staff]
screens: [all SCR-*]
inputs: [component, test_result]
states: [conformant, exception_logged]
api: none
events: none
data: [a11y_test_report]
rules:
  - Shared component library meets WCAG 2.2 AA (keyboard, focus visible, target size, contrast, error identification, accessible authentication without cognitive tests or CAPTCHA).
  - Automated axe checks in CI plus manual screen-reader passes (NVDA, VoiceOver, TalkBack) for guest booking and check-in journeys.
security: none.
failure_cases: [regression_in_ci]
finance_report_effect: none.
i18n_a11y: applies in both en and ar, including RTL focus order.
acceptance: Guest booking and checkout journeys complete using keyboard only and a screen reader in en and ar with zero axe critical violations.
dependency: W3C WCAG 2.2 `source-cited`.
```

```yaml
id: M01.F01.5.SF01.5.5
name: Bilingual document rendering
phase: 2
release: R1
actors: [front_desk_agent, cashier, guest]
screens: [SCR-FD-registration-card, SCR-FIN-invoice-preview]
inputs: [template_id, locale_primary, locale_secondary, data]
states: [draft, rendered, issued]
api: POST /v1/properties/{pid}/documents/render
events: [DocumentRendered]
data: [document_template, fiscal_document]
rules:
  - Templates versioned; bilingual side-by-side or sequential per jurisdiction rule (M38/M44).
  - PDF embeds fonts supporting Arabic shaping; tagged PDF for accessibility.
security: templates editable by property_admin, fiscal templates require compliance_officer approval.
failure_cases: [font_missing_glyphs, template_version_mismatch]
finance_report_effect: fiscal documents reproducible from stored data + template version.
i18n_a11y: tagged PDF with correct reading order in RTL.
acceptance: Re-rendering an issued invoice from stored data and its template version produces identical text content and hash.
dependency: M38 invoice rules (jurisdiction-specific, `unverified-assumption` per market).
```

## F01.6 Observability, secrets, device identity, audit and messaging backbone

```yaml
id: M01.F01.6.SF01.6.1
name: Structured logs and traces with PII redaction
phase: 2
release: R1
actors: [it_admin]
screens: [SCR-OPS-observability]
inputs: [service, log_level, trace_id]
states: [none]
api: none (OpenTelemetry exporters)
events: none
data: [log_stream, trace_store]
rules:
  - Correlation_id from API gateway propagates to events and adapter calls.
  - Redaction filters remove card data, ID numbers, SIN, passport numbers, tokens and free-text guest notes from logs.
security: log access restricted to it_admin; retention per D-104.
failure_cases: [redaction_bypass, exporter_backpressure]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A synthetic request containing a test PAN and passport number produces logs with neither value, verified by an automated scanner.
dependency: none.
```

```yaml
id: M01.F01.6.SF01.6.2
name: Metrics, SLOs and alert routing
phase: 2
release: R1
actors: [it_admin, duty_manager]
screens: [SCR-OPS-health, SCR-GM-attention-inbox]
inputs: [slo_definition, alert_rule, on_call_schedule]
states: [ok, warning, critical, acknowledged]
api: GET /v1/ops/health; POST /v1/ops/alerts/{id}/ack
events: [AlertRaised, AlertAcknowledged]
data: [alert, slo]
rules:
  - Business alerts (overbooking exposure, dead letters, night audit overdue) go to operational owners, technical alerts to it_admin.
  - Unacknowledged critical alert escalates after configured minutes.
security: none special.
failure_cases: [alert_storm, notification_channel_down]
finance_report_effect: none.
i18n_a11y: alerts bilingual; mobile push accessible.
acceptance: A dead-letter spike on channel ARI produces one grouped alert to integration_admin and an escalation to duty_manager if unacknowledged in 15 minutes.
dependency: M63 notification engine.
```

```yaml
id: M01.F01.6.SF01.6.3
name: Secrets management and rotation
phase: 2
release: R1
actors: [integration_admin, it_admin]
screens: [SCR-INT-credentials]
inputs: [secret_name, scope, rotation_period, provider_ref]
states: [active, rotating, expired, revoked]
api: POST /v1/properties/{pid}/secrets (write-only); POST .../secrets/{id}/rotate
events: [SecretRotated, SecretExpiring]
data: [secret_ref]
rules:
  - Secret values written to Vault/KMS only; API never returns a value after creation.
  - Expiry reminders 30/7/1 days before.
security: write-only API; dual control for payment and government credentials.
failure_cases: [rotation_breaks_adapter, expired_certificate]
finance_report_effect: none.
i18n_a11y: none.
acceptance: GET on a secret returns metadata only; a certificate expiring in 7 days raises an alert to integration_admin.
dependency: Vault/KMS deployment.
```

```yaml
id: M01.F01.6.SF01.6.4
name: Device registration and identity
phase: 2
release: R1
actors: [it_admin, housekeeping_supervisor, front_office_manager]
screens: [SCR-OPS-devices, SCR-HK-device-enrol]
inputs: [device_type, serial, assigned_department, enrollment_code]
states: [pending, enrolled, suspended, revoked, wiped]
api: POST /v1/properties/{pid}/devices/enroll (idem); POST /v1/properties/{pid}/devices/{id}/revoke
events: [DeviceEnrolled, DeviceRevoked]
data: [device_identity]
rules:
  - Staff mobile, kiosks, POS terminals and gate controllers get a device certificate (mTLS) bound to property and department.
  - Revocation invalidates offline token and triggers remote wipe of cached PII on next contact.
security: enrollment code single-use, 15-minute expiry.
failure_cases: [lost_device, cloned_certificate]
finance_report_effect: postings record device_id for audit (M60).
i18n_a11y: enrollment flow bilingual.
acceptance: A revoked housekeeping device cannot sync its offline change set and its cached guest names are wiped on next connection.
dependency: M64 device registry extension.
```

```yaml
id: M01.F01.6.SF01.6.5
name: Append-only audit log with hash chain
phase: 2
release: R1
actors: [auditor, compliance_officer]
screens: [SCR-OPS-audit-search]
inputs: [actor, action, object_ref, before, after, reason]
states: [recorded]
api: GET /v1/properties/{pid}/audit?object=...
events: none (audit is a sink)
data: [audit_log]
rules:
  - Every privileged, financial, inventory, consent and configuration change writes an audit record in the same transaction.
  - Records chained with SHA-256 per property per day; daily anchor hash stored offsite.
  - Field-level before/after masks sensitive fields (ID numbers show last 4).
security: no update/delete grants on audit_log for any role; read access scoped.
failure_cases: [hash_chain_break, audit_write_failure_aborts_transaction]
finance_report_effect: evidence for M60 and external audit.
i18n_a11y: audit viewer bilingual labels.
acceptance: Tampering with one audit row in a test database is detected by the daily verification job and reported as a chain break at that row.
dependency: none.
```

```yaml
id: M01.F01.6.SF01.6.6
name: Transactional outbox and inbox
phase: 2
release: R1
actors: [outbox_relay, integration_admin]
screens: [SCR-INT-event-health, SCR-INT-dead-letters]
inputs: [event_envelope, consumer_name]
states: [pending, published, failed_retrying, dead_lettered, replayed]
api: GET /v1/properties/{pid}/events/dead-letters; POST .../dead-letters/{id}/replay (idem)
events: [all domain events]
data: [outbox_message, inbox_message, dead_letter]
rules:
  - Event written in same DB transaction as state change; relay publishes at-least-once; consumers dedup on event_id in inbox table.
  - Ordering guaranteed per aggregate key (e.g. reservation_id, room_type+date).
  - Exponential backoff with max attempts, then dead letter with reason and replay button.
security: replay requires integration_admin; payload PII visible only to authorized roles.
failure_cases: [poison_message, broker_down, consumer_crash_mid_processing]
finance_report_effect: guarantees each posting-triggering event is processed once.
i18n_a11y: none.
acceptance: Killing a consumer after side effect but before ack and redelivering the event produces no duplicate side effect.
dependency: none.
```

### M01 key invariants
1. Exactly one open `business_day` per property; no write to a closed business date.
2. Every property-scoped row has `tenant_id`/`property_id` and RLS; cross-tenant access is impossible even with a known id.
3. Money is never floating point; currency exponent follows ISO-4217.
4. Every state change that others must react to emits its event in the same transaction (outbox); every consumer is idempotent.
5. Gated features are enforced server-side; UI flags never bypass a `blocked` gate.
6. Audit log is append-only and hash-chained; ledger tables are never rewritten by migrations.
7. Restored environments cannot call external partners.

### M01 module-level acceptance
| Test | Maps to | Statement |
|---|---|---|
| AC-M01-1 | AT-G08.1 | Night audit rolls the business date once; postings after calendar midnight but before rollover land on the prior business date and correct accounting period. |
| AC-M01-2 | AT-G09.1 | Five test properties (CA, OM, PK, SA, PT) created with distinct time zones, currencies and classifier outputs; go-live blocked for any feature with an unresolved gate. |
| AC-M01-3 | AT-G20.1 | Network outage: on-prem continues core PMS; queues drain without duplicates; restore test proves ledger checksums. |
| AC-M01-4 | AT-G19.1 | Guest booking journey passes WCAG 2.2 AA checks in en and ar (RTL). |
| AC-M01-5 | AT-G20.2 | Cross-tenant/property isolation probe suite passes 100%. |

### M01 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-101 | Night-audit rollover mode: manual only, or automatic at deadline with exceptions carried forward | Front Office Manager (pilot hotel) | Manual audit; alert at 04:00, forced rollover allowed by duty_manager after 06:00 local. |
| D-102 | On-prem hardware spec, UPS, remote support model and update cadence | IT Admin (pilot) + MetriSys Ops | Single server 16 vCPU/64 GB/2 TB SSD RAID1 + UPS; monthly updates; break-glass support only. |
| D-104 | RPO/RTO targets and offsite backup provider | GM + IT Admin | RPO 15 min, RTO 4 h (SaaS); on-prem RPO 1 h offsite, RTO 8 h; provider `unverified-assumption`. |
| D-116 | SaaS hosting region and data residency per market (Oman, Saudi, Canada, Portugal, Pakistan) | Compliance Officer + counsel per market | Region chosen per tenant; dedicated database option if residency required; no residency claim until `counsel-reviewed`. |
| D-121 | Digit shaping (Arabic-Indic vs Western) and Hijri date display | Product Owner | Western digits default, Arabic-Indic optional; Hijri shown as secondary annotation only. |

---

# M02 — IAM and consent

| Attribute | Value |
|---|---|
| Purpose | One identity and permission model for staff, guests, corporate users, vendors and machine clients; SSO/MFA; tenant/property/department/record/field scopes; delegated approvals; privileged-access audit; guest consent by purpose and channel; retention, export, deletion and legal hold; hard boundaries between employee, vendor, corporate and guest identities. |
| Build phase(s) | 2 (all core); 3 corporate/vendor portals consume it; 5 messaging consent adapters. |
| Release flag | R1. |
| Bounded context | `iam` (schema `iam`); consent/retention sub-context `privacy`. |
| Systems of record owned | `principal` (`staff_user`, `guest_account`, `corporate_user`, `vendor_user`, `service_account`), `role`, `permission`, `role_assignment`, `sod_rule`, `delegation`, `approval_policy`, `approval_request`, `privileged_session`, `access_review`, `consent_purpose`, `consent_record`, `privacy_notice_version`, `retention_policy`, `legal_hold`, `data_subject_request`. Credentials live in Keycloak, not in application tables. |
| Upstream dependencies | M01 (tenant/property/department, audit, secrets); M44 (jurisdiction-specific consent/retention rule packs). |
| Downstream dependents | Every module for authorization; M05/M52/M18/M40/M41 for consent; M27 (joiner/leaver feed); M10/M11 corporate users; M46/M48 vendor users; M33 API clients. |
| External dependencies | Keycloak (technology choice); corporate customer IdPs for SSO — per customer `unverified-assumption`; SMS/WhatsApp OTP providers via M41 — `unverified-assumption` until contracted. Privacy law applicability per market (Canada PIPEDA/provincial, Oman PDPL, Saudi PDPL, Pakistan, Portugal GDPR) — `unverified-assumption` pending `counsel-reviewed` register in docs/07. |

## F02.1 Identity types and lifecycle

```yaml
id: M02.F02.1.SF02.1.1
name: Staff user lifecycle (joiner, mover, leaver)
phase: 2
release: R1
actors: [hr_officer, property_admin, it_admin]
screens: [SCR-ADM-users, SCR-ADM-user-detail]
inputs: [employee_id, email, phone, legal_name, preferred_name, departments, start_date, end_date]
states: [invited, active, suspended, leaver_pending, deactivated]
api: POST /v1/properties/{pid}/staff-users; PATCH .../staff-users/{uid}; POST .../staff-users/{uid}/deactivate
events: [StaffUserActivated, StaffUserRoleChanged, StaffUserDeactivated]
data: [staff_user, role_assignment, audit_log]
rules:
  - A staff_user links to at most one M27 employee record; contractors have a contractor flag and end_date mandatory.
  - Leaver event from M27 deactivates access at end_date 23:59 property time and revokes sessions and device tokens.
  - Mover changes remove old department scopes before granting new ones (no accumulation).
security: property_admin cannot grant roles above own level; all changes audited.
failure_cases: [leaver_feed_missing, duplicate_email, shared_account_attempt]
finance_report_effect: user id stamped on every posting, cash shift and approval.
i18n_a11y: names in Arabic and Latin; invitation email bilingual.
acceptance: A leaver with end_date today cannot log in after 23:59 property time and any open cashier shift is flagged to the front_office_manager.
dependency: M27 employee master (Phase 4); manual HR entry until then.
```

```yaml
id: M02.F02.1.SF02.1.2
name: Guest account and verified contact
phase: 2
release: R1
actors: [guest, booker]
screens: [SCR-GST-sign-in, SCR-GST-account]
inputs: [email, phone, otp_code, locale]
states: [unverified, verified, locked, closed]
api: POST /v1/guest/auth/start; POST /v1/guest/auth/verify; GET /v1/guest/me
events: [GuestAccountCreated, GuestAccountVerified]
data: [guest_account, guest_profile]
rules:
  - Booking does not require an account (guest checkout); account creation is optional and passwordless (email magic link or OTP; SMS/WhatsApp only via approved M41 adapters).
  - guest_account links to one guest_profile after verified contact match; never auto-merge profiles on name alone.
security: OTP single-use 10 min, rate limited; accessible authentication without CAPTCHA puzzles.
failure_cases: [otp_not_delivered, account_takeover_attempt, email_reused_by_other_person]
finance_report_effect: none.
i18n_a11y: OTP message and screens bilingual; paste-friendly OTP field.
acceptance: A guest completes booking without an account; later verifying the same email links past reservations only after OTP success.
dependency: M41 OTP adapters (SMS/WhatsApp `unverified-assumption`); email provider.
```

```yaml
id: M02.F02.1.SF02.1.3
name: Corporate user identity
phase: 3
release: R1
actors: [corporate_admin, sales_manager]
screens: [SCR-CORP-users, SCR-ADM-corporate-accounts]
inputs: [company_id, email, role, cost_centers, sso_connection]
states: [invited, active, suspended, removed]
api: POST /v1/corporate/{cid}/users; PATCH /v1/corporate/{cid}/users/{uid}
events: [CorporateUserChanged]
data: [corporate_user, role_assignment]
rules:
  - Corporate users are scoped to one company (M10) and see only that company's bookings, rates and invoices.
  - corporate_admin manages own users; hotel sales_manager can suspend.
security: no access to hotel staff screens; separate realm/client.
failure_cases: [user_moves_company, sso_domain_conflict]
finance_report_effect: bookings attributed to company and cost center.
i18n_a11y: bilingual portal.
acceptance: A corporate user from company A requesting company B's invoice receives 404.
dependency: M10/M11 (Phase 3).
```

```yaml
id: M02.F02.1.SF02.1.4
name: Vendor user identity
phase: 2
release: R1
actors: [vendor_admin, procurement_officer]
screens: [SCR-VND-team, SCR-ADM-vendor-users]
inputs: [vendor_legal_entity_id, email, phone, vendor_role]
states: [invited, active, suspended, revoked]
api: POST /v1/vendors/{vid}/users; POST /v1/vendors/{vid}/users/{uid}/revoke
events: [VendorUserChanged]
data: [vendor_user, role_assignment]
rules:
  - Vendor users see only their own entity's profile, quotes, POs and assigned jobs (M46 SF46.3.x).
  - Guest data visible only per assigned job, minimum fields, revoked at job close.
  - Vendor identity is never an employee identity even for the same person.
security: MFA mandatory for vendor_admin; app device binding for vendor mobile.
failure_cases: [vendor_suspended_with_active_sessions]
finance_report_effect: none.
i18n_a11y: bilingual vendor app.
acceptance: Suspending a vendor revokes all its users' sessions within 60 seconds and hides its jobs.
dependency: M46 (Phase 2 identity/taxonomy).
```

```yaml
id: M02.F02.1.SF02.1.5
name: Service accounts and API clients
phase: 2
release: R1
actors: [integration_admin]
screens: [SCR-INT-api-clients]
inputs: [client_name, scopes, allowed_ips, expiry]
states: [active, expiring, revoked]
api: POST /v1/properties/{pid}/api-clients; DELETE .../api-clients/{id}
events: [ApiClientCreated, ApiClientRevoked]
data: [service_account, secret_ref]
rules:
  - OAuth2 client credentials with least-privilege scopes; property-bound; expiry mandatory.
  - Each channel/PSP adapter runs under its own service account.
security: secrets write-only; rotation reminders.
failure_cases: [scope_creep_request, leaked_secret]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A client with scope reservations:read calling a folio endpoint receives 403 and the attempt is audited.
dependency: M33.
```

## F02.2 Authentication, SSO and MFA

```yaml
id: M02.F02.2.SF02.2.1
name: SSO for staff and corporate (OIDC/SAML)
phase: 2
release: R1
actors: [it_admin, corporate_admin]
screens: [SCR-ADM-sso-connections]
inputs: [idp_metadata, domains, claim_mapping, jit_provisioning]
states: [draft, testing, active, disabled]
api: POST /v1/tenants/{tid}/sso-connections; POST .../sso-connections/{id}/test
events: [SsoConnectionActivated]
data: [sso_connection]
rules:
  - IdP claims can map to groups but property/department scope is assigned in MetriStay, not trusted blindly from external claims.
  - Local break-glass admin exists for on-prem when IdP unreachable.
security: signed assertions, key rollover support.
failure_cases: [idp_cert_expired, clock_skew_assertion_rejected]
finance_report_effect: none.
i18n_a11y: login pages bilingual.
acceptance: A corporate IdP user without an assigned MetriStay role can sign in but sees only an access-request page.
dependency: customer IdP availability `unverified-assumption`.
```

```yaml
id: M02.F02.2.SF02.2.2
name: MFA and step-up authentication
phase: 2
release: R1
actors: [all staff, corporate_approver, vendor_admin]
screens: [SCR-AUTH-mfa-enrol, SCR-AUTH-step-up]
inputs: [factor_type, action_class]
states: [not_enrolled, enrolled, step_up_required, step_up_passed]
api: POST /v1/auth/step-up
events: [StepUpPassed, StepUpFailed]
data: [staff_user, privileged_session]
rules:
  - MFA required for finance, admin, payment release, refunds above threshold, rate override above guardrail, consent/export and privileged roles.
  - Step-up valid 5 minutes for the specific action class.
security: TOTP/WebAuthn; SMS only as fallback per policy.
failure_cases: [lost_factor, repeated_failures_lockout]
finance_report_effect: refund/override records show step-up evidence.
i18n_a11y: WebAuthn supported for accessible auth; no cognitive puzzles.
acceptance: A refund above threshold without step-up returns 401 step_up_required; with step-up it succeeds and audit shows factor type.
dependency: none.
```

```yaml
id: M02.F02.2.SF02.2.3
name: Shared front-desk terminal and fast user switch
phase: 2
release: R1
actors: [front_desk_agent, cashier]
screens: [SCR-FD-user-switch]
inputs: [device_id, staff_pin, badge_id]
states: [locked, user_active, idle_locked]
api: POST /v1/properties/{pid}/devices/{did}/sessions
events: [TerminalUserSwitched]
data: [device_identity, staff_user]
rules:
  - Enrolled device plus personal PIN/badge gives a scoped session; idle lock after 3 minutes (configurable).
  - Each posting attributed to the active user, never to the device.
security: PIN only valid on enrolled devices; lockout after 5 failures.
failure_cases: [pin_shared_between_staff, device_not_enrolled]
finance_report_effect: correct cashier attribution for shift balancing.
i18n_a11y: large touch targets; PIN pad accessible.
acceptance: Two agents alternating on one terminal produce folio lines attributed to each respectively.
dependency: M01 SF01.6.4.
```

```yaml
id: M02.F02.2.SF02.2.4
name: Account recovery and lockout
phase: 2
release: R1
actors: [staff_user, guest, it_admin]
screens: [SCR-AUTH-recover]
inputs: [identifier, recovery_channel]
states: [requested, verified, reset, denied]
api: POST /v1/auth/recover
events: [AccountRecovered, AccountLocked]
data: [principal]
rules:
  - Responses do not reveal whether an account exists.
  - Privileged staff recovery requires admin verification in person or via manager.
security: rate limits per IP and identifier.
failure_cases: [enumeration_attempt, social_engineering]
finance_report_effect: none.
i18n_a11y: bilingual.
acceptance: Recovery for an unknown email returns the same response and timing profile as for a known one.
dependency: none.
```

## F02.3 Roles and scoped authorization

```yaml
id: M02.F02.3.SF02.3.1
name: Role and permission catalogue
phase: 2
release: R1
actors: [tenant_admin, property_admin]
screens: [SCR-ADM-roles]
inputs: [role_code, permissions, description]
states: [system_role, custom_role, deprecated]
api: GET /v1/roles; POST /v1/properties/{pid}/roles
events: [RoleDefinitionChanged]
data: [role, permission]
rules:
  - System roles mirror README §3.3 actors; custom roles compose permissions but cannot include permissions the creator lacks.
  - Permissions are verb-object (e.g. folio.refund.create) with optional limit attributes (max_amount).
security: role changes audited; high-risk permission sets flagged.
failure_cases: [privilege_escalation_via_custom_role]
finance_report_effect: none.
i18n_a11y: role descriptions bilingual.
acceptance: A property_admin without folio.refund.create cannot create a custom role containing it.
dependency: none.
```

```yaml
id: M02.F02.3.SF02.3.2
name: Scoped role assignment (tenant, property, department, record)
phase: 2
release: R1
actors: [property_admin, tenant_admin]
screens: [SCR-ADM-user-detail]
inputs: [principal_id, role_id, scope_type, scope_ids, valid_from, valid_to]
states: [pending, active, expired, revoked]
api: POST /v1/properties/{pid}/role-assignments; DELETE .../role-assignments/{id}
events: [RoleAssigned, RoleRevoked]
data: [role_assignment]
rules:
  - Scope levels are tenant > property > department > record-set (e.g. assigned rooms for a housekeeper, own jobs for a vendor, own company for corporate).
  - Policy decision point evaluates role + scope + attribute conditions on every request; results cached per request only.
security: time-bounded assignments for temporary cover.
failure_cases: [assignment_without_expiry_for_contractor, stale_cache]
finance_report_effect: report filters inherit scopes (M65).
i18n_a11y: none.
acceptance: A bar supervisor scoped to department F&B/bar can view bar outlet revenue but receives 403 for rooms revenue and front-desk folios.
dependency: none.
```

```yaml
id: M02.F02.3.SF02.3.3
name: Field-level protection and masking
phase: 2
release: R1
actors: [front_desk_agent, dpo, hr_officer]
screens: [SCR-FD-guest-profile, SCR-ADM-field-policies]
inputs: [entity, field, classification, mask_rule]
states: [visible, masked, hidden]
api: applied by serializers; GET /v1/field-policies
events: [SensitiveFieldRevealed]
data: [field_policy, audit_log]
rules:
  - Classifications = public, internal, personal, sensitive (ID numbers, date of birth, nationality, health/accessibility notes), restricted (SIN, salary, biometric).
  - Sensitive fields masked by default; reveal requires permission, reason and is audited.
  - Exports and reports apply the same masking.
security: encryption at rest for sensitive and restricted columns with separate keys.
failure_cases: [unmasked_export, reveal_without_reason]
finance_report_effect: none.
i18n_a11y: masked values announced as masked to screen readers.
acceptance: A front_desk_agent sees passport number as ****1234; revealing requires a reason and creates an audit record; a CSV export by the same user contains the masked value.
dependency: M41 ID field rules per jurisdiction.
```

```yaml
id: M02.F02.3.SF02.3.4
name: Record-level restrictions (VIP, incognito, sensitive stays)
phase: 2
release: R1
actors: [front_office_manager, guest_relations]
screens: [SCR-FD-reservation, SCR-ADM-record-restrictions]
inputs: [record_ref, restriction_type, allowed_roles]
states: [unrestricted, restricted]
api: PUT /v1/properties/{pid}/reservations/{rid}/restriction
events: [RecordRestrictionChanged]
data: [record_restriction]
rules:
  - Incognito guest names hidden from housekeeping boards, switchboard lookups and non-authorized searches.
  - Restriction propagates to derived views (room board, reports).
security: only front_office_manager and above set restrictions.
failure_cases: [leak_via_search_index]
finance_report_effect: none.
i18n_a11y: none.
acceptance: An incognito guest does not appear in guest-name search for a front_desk_agent, and the housekeeping board shows only room number.
dependency: none.
```

```yaml
id: M02.F02.3.SF02.3.5
name: Segregation-of-duties rules
phase: 2
release: R1
actors: [financial_controller, compliance_officer]
screens: [SCR-ADM-sod-rules, SCR-ADM-sod-violations]
inputs: [conflicting_permission_pair, enforcement_mode]
states: [warn, block]
api: GET /v1/properties/{pid}/sod-violations; PUT .../sod-rules/{id}
events: [SodViolationDetected]
data: [sod_rule, role_assignment]
rules:
  - Default blocking pairs are same user creating and approving refund; creating payee and releasing payment; editing rate and approving own override.
  - Single-person small hotels can set warn mode with compensating review (logged).
security: rules change needs financial_controller + gm.
failure_cases: [conflict_via_delegation]
finance_report_effect: control evidence in M60.
i18n_a11y: none.
acceptance: A cashier who created a refund request cannot approve it even when holding both roles; the attempt is logged.
dependency: M60 controls.
```

```yaml
id: M02.F02.3.SF02.3.6
name: Privileged access monitoring and access reviews
phase: 2
release: R1
actors: [auditor, gm, it_admin]
screens: [SCR-ADM-access-review, SCR-OPS-audit-search]
inputs: [review_period, reviewer, decisions]
states: [scheduled, in_review, completed, overdue]
api: POST /v1/properties/{pid}/access-reviews; POST .../access-reviews/{id}/decisions
events: [AccessReviewCompleted, PrivilegedActionPerformed]
data: [access_review, privileged_session, audit_log]
rules:
  - Quarterly review of privileged roles; unreviewed assignments auto-expire after grace period (configurable).
  - Privileged actions (role grants, reveals, exports, forced rollovers, reopenings) listed in a daily digest to gm.
security: reviewers cannot review themselves.
failure_cases: [review_overdue]
finance_report_effect: none.
i18n_a11y: none.
acceptance: An assignment not confirmed within 14 days after the review deadline is revoked and the user notified.
dependency: none.
```

## F02.4 Delegated approvals

```yaml
id: M02.F02.4.SF02.4.1
name: Approval policy definitions
phase: 2
release: R1
actors: [financial_controller, gm]
screens: [SCR-ADM-approval-policies]
inputs: [action_type, thresholds, approver_roles, levels, currency]
states: [draft, active, retired]
api: POST /v1/properties/{pid}/approval-policies; PATCH .../approval-policies/{id}
events: [ApprovalPolicyChanged]
data: [approval_policy, config_version]
rules:
  - Covers rate override, discount, refund, rebate, write-off, reopen business date, overbooking beyond limit, comp, cash variance.
  - Thresholds in property base currency minor units; multi-level for higher bands.
security: maker-checker for policy changes.
failure_cases: [gap_between_thresholds]
finance_report_effect: approvals linked to postings.
i18n_a11y: bilingual policy labels.
acceptance: A refund of OMR 150.000 routes to level-2 approver when level-1 limit is OMR 100.000.
dependency: none.
```

```yaml
id: M02.F02.4.SF02.4.2
name: Time-bounded delegation
phase: 2
release: R1
actors: [gm, financial_controller, front_office_manager]
screens: [SCR-ADM-delegations]
inputs: [delegator, delegate, action_types, max_amount, valid_from, valid_to]
states: [scheduled, active, expired, revoked]
api: POST /v1/properties/{pid}/delegations; DELETE .../delegations/{id}
events: [DelegationActivated, DelegationExpired]
data: [delegation]
rules:
  - Delegate cannot exceed delegator's limits; cannot delegate to self; SoD rules still apply.
  - Actions record "approved by X on behalf of Y".
security: delegation creation requires step-up.
failure_cases: [delegation_chain_attempt]
finance_report_effect: audit shows delegated authority.
i18n_a11y: none.
acceptance: A delegate of a delegate cannot approve; a delegation outside its valid window is rejected.
dependency: none.
```

```yaml
id: M02.F02.4.SF02.4.3
name: Approval request execution
phase: 2
release: R1
actors: [any requester, approver]
screens: [SCR-GM-approvals-inbox, SCR-MOB-approvals]
inputs: [request_type, subject_ref, amount, reason, attachments]
states: [pending, approved, rejected, expired, cancelled]
api: POST /v1/properties/{pid}/approvals; POST .../approvals/{id}/decide (idem)
events: [ApprovalRequested, ApprovalDecided]
data: [approval_request]
rules:
  - Decision binds to the subject version; if subject changes (amount), approval invalidates.
  - Requester cannot approve own request.
security: step-up for money-moving approvals; mobile approval requires enrolled device.
failure_cases: [subject_changed_after_approval, duplicate_decision]
finance_report_effect: posting only proceeds with an approved request id.
i18n_a11y: approvals inbox accessible on mobile.
acceptance: Changing a discount from 10% to 20% after approval invalidates the approval and the discount cannot be posted.
dependency: M63 engine.
```

```yaml
id: M02.F02.4.SF02.4.4
name: Approval escalation and expiry
phase: 2
release: R1
actors: [approval_worker, gm]
screens: [SCR-GM-approvals-inbox]
inputs: [sla_minutes, escalation_path]
states: [pending, escalated, expired]
api: none (worker); GET /v1/properties/{pid}/approvals?state=escalated
events: [ApprovalEscalated, ApprovalExpired]
data: [approval_request]
rules:
  - Unanswered requests escalate per path; expiry leaves subject unchanged.
security: none.
failure_cases: [no_available_approver]
finance_report_effect: none.
i18n_a11y: notifications bilingual.
acceptance: A late-checkout fee waiver pending 30 minutes escalates to duty_manager; after expiry the fee remains.
dependency: M63.
```

## F02.5 Consent, retention and data-subject rights

```yaml
id: M02.F02.5.SF02.5.1
name: Consent purpose catalogue per jurisdiction
phase: 2
release: R1
actors: [dpo, compliance_officer]
screens: [SCR-ADM-consent-purposes]
inputs: [purpose_code, legal_basis_by_market, channels, default_state, notice_text_by_locale]
states: [draft, reviewed, active, retired]
api: POST /v1/properties/{pid}/consent-purposes; PATCH .../consent-purposes/{code}
events: [ConsentPurposeChanged]
data: [consent_purpose, privacy_notice_version]
rules:
  - Purposes include marketing_email, marketing_sms, marketing_whatsapp, profiling_personalization, id_image_processing, biometric_match, analytics_cookies, review_invitation, ai_chat_transcript, travel_data_sharing.
  - Legal basis per market comes from M44 rule pack; status must be counsel-reviewed before a purpose relying on consent goes live in that market; otherwise purpose stays inactive.
  - Service messages (booking confirmation) are separated from marketing and not gated by marketing consent.
security: dpo approval.
failure_cases: [purpose_without_market_basis]
finance_report_effect: none.
i18n_a11y: notice text en/ar reviewed; plain language.
acceptance: A marketing_whatsapp purpose in a market whose rule pack is unverified-assumption cannot be activated, and the UI names the missing review.
dependency: M44 rule packs; docs/07 privacy register (`unverified-assumption` per market).
```

```yaml
id: M02.F02.5.SF02.5.2
name: Capture consent by purpose and channel with evidence
phase: 2
release: R1
actors: [guest, front_desk_agent, booker]
screens: [SCR-GST-checkout, SCR-FD-registration-card, SCR-GST-privacy-center]
inputs: [guest_profile_id, purpose_code, channel, state, source, notice_version, captured_by]
states: [granted, denied, withdrawn, expired]
api: POST /v1/properties/{pid}/guests/{gid}/consents (idem); GET .../guests/{gid}/consents
events: [ConsentGranted, ConsentWithdrawn]
data: [consent_record]
rules:
  - Unticked by default; one record per purpose per channel; stores notice_version, timestamp, source (web, desk, app), IP/device where applicable.
  - A booker cannot grant marketing consent on behalf of an occupant; each person consents for themselves.
  - Records are append-only; current state derived from latest.
security: staff capture requires the guest's in-person confirmation recorded with staff id.
failure_cases: [pre_ticked_box, consent_for_other_person]
finance_report_effect: none.
i18n_a11y: consent controls are native checkboxes with labels; not bundled with terms acceptance.
acceptance: A booker making a reservation for two occupants can only record consent for self; occupants' marketing consent remains unknown until they give it.
dependency: none.
```

```yaml
id: M02.F02.5.SF02.5.3
name: Withdrawal and suppression propagation
phase: 2
release: R1
actors: [guest, marketing_manager, dpo]
screens: [SCR-GST-privacy-center, SCR-ADM-suppression-list]
inputs: [guest_profile_id, purpose_code, channel]
states: [active_suppression]
api: POST /v1/guest/me/consents/{purpose}/withdraw; POST /v1/properties/{pid}/suppressions
events: [ConsentWithdrawn, SuppressionAdded]
data: [consent_record, suppression_entry]
rules:
  - Withdrawal effective within 5 minutes for all sending paths (M52 campaigns, M40 chat follow-ups, M55 inbox marketing).
  - Suppression by contact point survives profile deletion (hashed) to prevent re-contact.
security: none special.
failure_cases: [queued_campaign_after_withdrawal]
finance_report_effect: none.
i18n_a11y: one-click unsubscribe links in both languages.
acceptance: A campaign queued before withdrawal does not send to the withdrawn contact if send time is after withdrawal.
dependency: M52 (Phase 3).
```

```yaml
id: M02.F02.5.SF02.5.4
name: Retention policies by record purpose and country
phase: 2
release: R1
actors: [dpo, financial_controller, compliance_officer]
screens: [SCR-ADM-retention-policies]
inputs: [record_type, purpose, market, retention_period, start_event, action]
states: [draft, reviewed, active]
api: PUT /v1/properties/{pid}/retention-policies/{record_type}
events: [RetentionPolicyChanged, RetentionJobCompleted]
data: [retention_policy]
rules:
  - Separate periods for fiscal records (folio, invoices), registration/ID records, marketing profiles, CCTV pointers, audit logs.
  - Actions = delete, anonymize, archive; fiscal ledgers anonymize personal fields but retain amounts.
  - Periods sourced from M44 rule pack; unreviewed markets use the longest conservative period for fiscal and shortest for ID images with a flag.
security: retention job runs as service account; results audited.
failure_cases: [conflicting_periods, job_failure]
finance_report_effect: ledgers remain reconcilable after anonymization.
i18n_a11y: none.
acceptance: After a registration record's retention expires, ID images are deleted, the folio remains with guest name replaced by a pseudonymous token, and totals are unchanged.
dependency: M44 retention rules (`unverified-assumption` per market, D-118).
```

```yaml
id: M02.F02.5.SF02.5.5
name: Legal hold
phase: 2
release: R1
actors: [compliance_officer, gm]
screens: [SCR-ADM-legal-holds]
inputs: [scope_type (guest/reservation/incident/date_range), scope_ref, reason, authority_ref]
states: [active, released]
api: POST /v1/properties/{pid}/legal-holds; POST .../legal-holds/{id}/release
events: [LegalHoldPlaced, LegalHoldReleased]
data: [legal_hold]
rules:
  - Hold suspends deletion/anonymization for matched records, including DSR deletion.
security: compliance_officer only; dual control for release.
failure_cases: [hold_scope_too_broad]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A deletion request for a guest under legal hold is placed in held state and executed only after release.
dependency: none.
```

```yaml
id: M02.F02.5.SF02.5.6
name: Data-subject access and export
phase: 2
release: R1
actors: [guest, dpo]
screens: [SCR-GST-privacy-center, SCR-ADM-dsr-queue]
inputs: [requester_identity, verification_evidence, scope]
states: [received, verifying, in_progress, delivered, rejected]
api: POST /v1/guest/me/data-requests; POST /v1/properties/{pid}/dsr/{id}/fulfil
events: [DataSubjectRequestReceived, DataSubjectRequestFulfilled]
data: [data_subject_request]
rules:
  - Statutory deadlines per market configured from M44; countdown visible.
  - Export compiles profile, reservations, folios, consents, messages; excludes other persons' data and security-sensitive logs.
security: identity verification before release; download link expiring.
failure_cases: [unverifiable_identity, overdue]
finance_report_effect: none.
i18n_a11y: export readable format with bilingual headers.
acceptance: An access request is fulfilled with a machine-readable file listing all reservations for that profile but no data about co-occupants.
dependency: M44 deadlines (`unverified-assumption`).
```

```yaml
id: M02.F02.5.SF02.5.7
name: Deletion and anonymization respecting mandatory retention
phase: 2
release: R1
actors: [dpo, guest]
screens: [SCR-ADM-dsr-queue]
inputs: [dsr_id, affected_records]
states: [planned, executing, completed, partially_completed]
api: POST /v1/properties/{pid}/dsr/{id}/erase (idem)
events: [PersonalDataErased]
data: [data_subject_request, guest_profile, retention_policy]
rules:
  - Records under mandatory fiscal/registration retention are restricted rather than deleted, with explanation to requester.
  - Erasure propagates to search index, caches, analytics and downstream processors via event.
security: dual approval.
failure_cases: [processor_not_acknowledging, backup_contains_data]
finance_report_effect: amounts preserved.
i18n_a11y: response letter bilingual.
acceptance: After erasure, guest name search returns nothing; the folio still totals correctly and shows a pseudonym; backups roll off per retention schedule documented in the response.
dependency: none.
```

```yaml
id: M02.F02.5.SF02.5.8
name: Privacy notice versions and processing register
phase: 2
release: R1
actors: [dpo]
screens: [SCR-ADM-privacy-notices]
inputs: [notice_text_by_locale, effective_from, processors, purposes]
states: [draft, published, superseded]
api: POST /v1/properties/{pid}/privacy-notices
events: [PrivacyNoticePublished]
data: [privacy_notice_version]
rules:
  - Every consent_record references the notice version shown; material changes may require re-consent per market rule.
  - Register lists processors (PSP, channel manager, messaging) with honesty label and data categories.
security: dpo publish.
failure_cases: [notice_missing_locale]
finance_report_effect: none.
i18n_a11y: bilingual, accessible HTML.
acceptance: Every consent record in a sample of 1,000 resolves to a published notice version in the language shown.
dependency: docs/07 privacy register.
```

### M02 key invariants
1. No request is authorized from client-supplied scope alone; object ownership and property scope checked server-side.
2. Staff, guest, corporate and vendor identities are separate principals; a person holding two gets two accounts with no permission bleed.
3. No self-approval; approval binds to subject version; delegation never exceeds delegator limits.
4. Consent is per person, per purpose, per channel, append-only, and never inferred from booking or payment.
5. Deletion never destroys records under mandatory retention or legal hold; it restricts or anonymizes them.
6. Sensitive field reveals and exports are always audited and masked by default.

### M02 module-level acceptance
| Test | Maps to | Statement |
|---|---|---|
| AC-M02-1 | AT-G05.1 | Individual salary fields invisible to GM role; only payroll roles can reveal (field policy). |
| AC-M02-2 | AT-G10.1 | Vendor users see only own jobs/POs; department search scope respected. |
| AC-M02-3 | AT-G13.1 | Consent for ID processing and messaging captured by purpose/channel before OCR and OTP run. |
| AC-M02-4 | AT-G19.2 | Consented review invitation only; withdrawn guest receives no post-stay marketing. |
| AC-M02-5 | AT-G20.3 | Refund and override approvals enforce SoD and step-up under concurrent requests. |

### M02 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-103 | Staff MFA factors allowed (TOTP, WebAuthn, SMS fallback) and shared-terminal PIN policy | IT Admin + GM | TOTP/WebAuthn mandatory for privileged roles; PIN+device for front desk terminals. |
| D-115 | Guest authentication method (magic link, email OTP, SMS/WhatsApp OTP) per market | Product Owner + DPO | Email magic link/OTP first; SMS/WhatsApp only after M41 adapter contracted. |
| D-118 | Retention periods for ID images, registration cards, folios per market | DPO + counsel per market | ID images deleted 30 days after checkout unless a rule pack requires longer; fiscal records 10 years; all marked `unverified-assumption`. |
| D-123 | Small-hotel SoD: allow warn-mode with compensating review | Financial Controller | Allowed for properties with fewer than 5 finance-capable staff; reviewed weekly. |

---

# M03 — Room inventory

| Attribute | Value |
|---|---|
| Purpose | Model the physical hotel (buildings, floors, room types, attributes, rooms, connecting and accessible rooms) and maintain saleable room-type-by-night stock that is independent from which physical room a guest is assigned; manage OOO/OOS, holds with expiry, stop-sell, allotments and controlled overbooking so the hotel never sells a ghost room. |
| Build phase(s) | 2 (all); 3 group/corporate allotments consume it (M12, M10); 5 revenue recommendations feed overbooking (M53). |
| Release flag | R1. |
| Bounded context | `inventory` (schema `inventory`). Timed non-room resources are M09, not here. |
| Systems of record owned | `building`, `floor`, `room_type`, `room_attribute`, `room`, `room_connection`, `suite_composition`, `room_type_inventory_night`, `inventory_movement` (append-only journal behind the night counters), `inventory_hold`, `room_status_block` (OOO/OOS), `stop_sell`, `allotment`, `overbooking_policy`, `room_assignment`. |
| Upstream dependencies | M01 (property, business date, config); M02 (scopes); M26 maintenance requests OOO; M66 renovation closures. |
| Downstream dependents | M04 (quotes need availability), M05 (reservations consume stock), M06 (room grid), M07 (ARI availability), M12 (blocks), M53 (overbooking risk), M32 (occupancy denominators exclude OOO). |
| External dependencies | None. |

**Stock formula (normative).** For each `{property_id, room_type_id, stay_date}`:
`available = physical − out_of_order − sold − held − allotted_unreleased + overbook_allowance`, where `overbook_allowance = min(policy_limit, approved_extra)` and `available` may be ≤ 0 only when the overbooking policy explicitly allows it. `sold` counts every reservation_room night in state tentative/confirmed/in_house for that sold type. OOS rooms remain in `physical` (sellable, last to assign).

## F03.1 Physical structure

```yaml
id: M03.F03.1.SF03.1.1
name: Buildings, floors and zones
phase: 2
release: R1
actors: [property_admin]
screens: [SCR-ADM-buildings-floors]
inputs: [building_code, name_en, name_ar, floor_number, zone_code, housekeeping_section]
states: [active, closed]
api: POST /v1/properties/{pid}/buildings; POST /v1/properties/{pid}/floors
events: [PropertyStructureChanged]
data: [building, floor]
rules:
  - Floors may have negative or lettered labels; ordering is explicit.
  - Housekeeping sections and zones are assignment units for M06.
security: property_admin only.
failure_cases: [delete_floor_with_rooms]
finance_report_effect: none.
i18n_a11y: bilingual names; floor labels not assumed numeric.
acceptance: A floor labeled "M" (mezzanine) sorts between 1 and 2 when configured so and is usable as a housekeeping section.
dependency: none.
```

```yaml
id: M03.F03.1.SF03.1.2
name: Room types and attributes
phase: 2
release: R1
actors: [property_admin, revenue_manager]
screens: [SCR-ADM-room-types]
inputs: [room_type_code, name_by_locale, description_by_locale, base_occupancy, max_adults, max_children, max_occupancy, bed_configurations, size_sqm, view, accessibility_features, smoking, media_refs]
states: [draft, active, retired]
api: POST /v1/properties/{pid}/room-types; PATCH .../room-types/{rtid}
events: [RoomTypeChanged]
data: [room_type, room_attribute]
rules:
  - Room type code unique per property and immutable after first sale; retire rather than delete.
  - Accessibility features are structured attributes (step-free, roll-in shower, grab bars, visual alarm, hearing loop) not free text, so search can filter by need.
  - Media and descriptions come from M39 approved assets only.
security: revenue_manager and property_admin.
failure_cases: [max_occupancy_below_base, retire_with_future_sales]
finance_report_effect: room type is a revenue dimension.
i18n_a11y: descriptions per locale; accessibility attributes rendered as text lists.
acceptance: A room type cannot be retired while any future reservation_room night references it; the error lists the reservations.
dependency: M39 media (Phase 2–3).
```

```yaml
id: M03.F03.1.SF03.1.3
name: Physical rooms
phase: 2
release: R1
actors: [property_admin, chief_engineer]
screens: [SCR-ADM-rooms, SCR-FD-room-grid]
inputs: [room_number, room_type_id, floor_id, attributes, accessible_flag, connecting_group, key_system_ref, housekeeping_credits]
states: [active, inactive]
api: POST /v1/properties/{pid}/rooms; PATCH .../rooms/{room_id}
events: [RoomChanged]
data: [room, room_attribute]
rules:
  - A room belongs to exactly one room type at a time; type change is effective-dated (SF03.1.5).
  - Room-level attributes (high floor, near lift, balcony) override type defaults for assignment preferences only; they never change stock.
security: property_admin.
failure_cases: [duplicate_room_number]
finance_report_effect: physical count feeds occupancy denominator.
i18n_a11y: room numbers locale-neutral.
acceptance: Adding a room of type DLX increases physical for DLX on all future dates from its effective date by one and emits an ARI availability delta.
dependency: none.
```

```yaml
id: M03.F03.1.SF03.1.4
name: Connecting rooms and composed suites
phase: 2
release: R1
actors: [property_admin, front_desk_agent]
screens: [SCR-ADM-room-connections]
inputs: [room_ids, connection_type, suite_type_id, component_rooms]
states: [active, inactive]
api: POST /v1/properties/{pid}/room-connections; POST .../suite-compositions
events: [RoomConnectionChanged]
data: [room_connection, suite_composition]
rules:
  - Connecting pair is an assignment attribute; guests requesting connecting rooms get a linked assignment request.
  - A composed suite (e.g. two rooms sold as one family suite) consumes stock from its own type and blocks component types on those nights; inventory transfer is atomic.
security: property_admin.
failure_cases: [component_already_sold, circular_composition]
finance_report_effect: occupancy counts component rooms, not the composite twice.
i18n_a11y: none.
acceptance: Selling one composed family suite for a night decrements the suite type by 1 and each component room type by 1; occupancy reports count 2 occupied rooms.
dependency: none.
```

```yaml
id: M03.F03.1.SF03.1.5
name: Effective-dated inventory change
phase: 2
release: R1
actors: [property_admin, gm]
screens: [SCR-ADM-inventory-change]
inputs: [room_id, new_room_type_id, effective_from, reason]
states: [planned, effective, cancelled]
api: POST /v1/properties/{pid}/inventory-changes (idem)
events: [InventoryReclassified]
data: [room, room_type_inventory_night, inventory_movement]
rules:
  - Reclassification is refused if it would drive any future night of the old type below sold (unless gm accepts overbooking exposure explicitly).
  - Renovation-driven changes come from M66 SF66.2.2.
security: gm approval.
failure_cases: [future_oversell_created]
finance_report_effect: occupancy denominators change from effective date; reports show restatement note.
i18n_a11y: none.
acceptance: Reclassifying room 305 from STD to DLX effective 1 Nov leaves October stock unchanged and blocks the change if STD is fully sold on 3 Nov.
dependency: M66 (Phase 4–6).
```

## F03.2 Room-type-by-night stock

```yaml
id: M03.F03.2.SF03.2.1
name: Night stock ledger model
phase: 2
release: R1
actors: [system]
screens: [SCR-REV-inventory-calendar]
inputs: [room_type_id, stay_date, movement_type, quantity, source_ref]
states: [none]
api: none (internal service); GET /v1/properties/{pid}/inventory-nights?from=&to=
events: [RoomTypeAvailabilityChanged]
data: [room_type_inventory_night, inventory_movement]
rules:
  - Counters (physical, out_of_order, sold, held, allotted, overbook_limit) are updated only via inventory_movement rows in the same transaction; counters are a projection of the journal.
  - Horizon materialized for 730 days rolling; extended nightly.
  - Row-level lock ordering by (room_type_id, stay_date) ascending to avoid deadlocks on multi-night bookings.
security: no direct write API.
failure_cases: [horizon_not_extended, deadlock]
finance_report_effect: occupancy, available-rooms and RevPAR denominators.
i18n_a11y: none.
acceptance: Replaying the inventory_movement journal for any date reproduces the counters exactly.
dependency: none.
```

```yaml
id: M03.F03.2.SF03.2.2
name: Multi-night availability query
phase: 2
release: R1
actors: [guest, front_desk_agent, ai_assistant, channel adapter]
screens: [SCR-GST-search, SCR-FD-availability]
inputs: [arrival, departure, adults, children_ages, room_count, accessibility_needs, channel]
states: [none]
api: GET /v1/properties/{pid}/availability?arrival=&departure=&adults=&children=&rooms=
events: [FunnelSearchPerformed (sampled)]
data: [room_type_inventory_night, stop_sell, allotment, room_type]
rules:
  - A room type is available for a stay only if every night has available >= requested rooms after channel-specific stop-sell and allotment visibility.
  - Result includes freshness timestamp; direct web reads the primary, not a stale cache, at checkout.
security: public read rate-limited; no PII.
failure_cases: [stale_cache, oversized_range]
finance_report_effect: search counts feed funnel (M07 F07.5).
i18n_a11y: results accessible; accessible-room filter labeled clearly.
acceptance: A room type with availability 1 on night 2 of a 3-night search and room_count 2 is excluded; with room_count 1 it is included.
dependency: none.
```

```yaml
id: M03.F03.2.SF03.2.3
name: Atomic stock consumption and release
phase: 2
release: R1
actors: [system]
screens: [none]
inputs: [reservation_room_id, room_type_id, nights, operation (consume/release/transfer), idempotency_key]
states: [applied, rejected]
api: internal port InventoryPort.consume/release/transfer (idem)
events: [RoomTypeAvailabilityChanged, OversellDetected]
data: [room_type_inventory_night, inventory_movement]
rules:
  - All nights of a stay consumed in one DB transaction or none; SELECT ... FOR UPDATE per night in key order.
  - Consumption beyond available is rejected unless overbooking policy allows it; channel ingests follow SF07.4.3.
  - Idempotency key prevents double consume on retries.
security: internal only.
failure_cases: [concurrent_last_room, retry_after_timeout]
finance_report_effect: none directly.
i18n_a11y: none.
acceptance: 50 concurrent requests for the last room on a date yield exactly one success and 49 rejections; retrying the successful request with the same key returns the same result without a second decrement.
dependency: none.
```

```yaml
id: M03.F03.2.SF03.2.4
name: Inventory integrity audit and recompute
phase: 2
release: R1
actors: [night_audit_worker, revenue_manager]
screens: [SCR-REV-inventory-audit]
inputs: [date_range]
states: [consistent, mismatch_found, repaired]
api: POST /v1/properties/{pid}/inventory-audits (idem)
events: [InventoryMismatchFound]
data: [room_type_inventory_night, reservation_room, inventory_hold]
rules:
  - Nightly job recomputes sold/held from reservations and holds and compares to counters.
  - Mismatch never silently corrected; it creates an exception with proposed correcting movement for revenue_manager approval.
security: approval for correcting movement.
failure_cases: [mismatch_due_to_lost_event]
finance_report_effect: occupancy statistics flagged if mismatch unresolved.
i18n_a11y: none.
acceptance: A deliberately injected counter drift is detected in the next nightly run and appears in the audit queue with the affected dates.
dependency: M08 night audit sequence.
```

```yaml
id: M03.F03.2.SF03.2.5
name: Inventory calendar view
phase: 2
release: R1
actors: [revenue_manager, front_office_manager, gm]
screens: [SCR-REV-inventory-calendar]
inputs: [date_range, room_types]
states: [none]
api: GET /v1/properties/{pid}/inventory-nights
events: none
data: [room_type_inventory_night]
rules:
  - Shows physical, OOO, sold, held, allotted, overbook limit, available and occupancy % per night with drill to contributing reservations/holds.
security: revenue scope.
failure_cases: [large_range_timeout]
finance_report_effect: none.
i18n_a11y: grid navigable by keyboard; RTL mirrors date direction; numbers not color-only.
acceptance: Clicking a sold count lists exactly the reservations contributing to it.
dependency: none.
```

## F03.3 Out of order, out of service and room blocks

```yaml
id: M03.F03.3.SF03.3.1
name: Out-of-order room block
phase: 2
release: R1
actors: [chief_engineer, front_office_manager, housekeeping_supervisor]
screens: [SCR-FD-room-grid, SCR-ENG-work-order]
inputs: [room_id, from_date, to_date, reason_code, work_order_ref, notes]
states: [requested, active, released, cancelled]
api: POST /v1/properties/{pid}/room-blocks (idem); POST .../room-blocks/{id}/release
events: [RoomOutOfOrder, RoomReturnedToService]
data: [room_status_block, room_type_inventory_night, inventory_movement]
rules:
  - OOO removes the room from physical sellable stock for each night; refused if it would oversell the type unless front_office_manager accepts exposure and relocations are planned.
  - A room with an in-house guest cannot be set OOO for current night without a room move.
security: engineering requests; front_office_manager confirms when conflicts exist.
failure_cases: [conflict_with_assigned_reservation, open_ended_block]
finance_report_effect: OOO rooms excluded from available rooms in occupancy per KPI dictionary (M32 SF32.1.1).
i18n_a11y: reason codes bilingual.
acceptance: Setting room 412 OOO for 3 nights reduces DLX available by 1 on each night, unassigns any future assignment with a notification, and is rejected for tonight while occupied.
dependency: M26 maintenance (Phase 3–4); manual until then.
```

```yaml
id: M03.F03.3.SF03.3.2
name: Out-of-service room status
phase: 2
release: R1
actors: [housekeeping_supervisor, engineer]
screens: [SCR-FD-room-grid, SCR-HK-room-board]
inputs: [room_id, from, to, reason]
states: [active, released]
api: POST /v1/properties/{pid}/room-blocks (type=oos)
events: [RoomOutOfService]
data: [room_status_block]
rules:
  - OOS keeps stock (sellable) but excludes the room from auto-assignment until released; used for minor defects.
  - If OOS rooms of a type exceed unassigned sellable surplus, raise an assignment-risk alert.
security: none special.
failure_cases: [oos_rooms_needed_for_arrivals]
finance_report_effect: counted as available rooms.
i18n_a11y: none.
acceptance: With 2 OOS DLX rooms and DLX sold equal to physical minus 1 for tonight, an assignment-risk alert is raised to front_office_manager.
dependency: none.
```

```yaml
id: M03.F03.3.SF03.3.3
name: Renovation and long-term closure blocks
phase: 4
release: R1
actors: [gm, owner, revenue_manager]
screens: [SCR-ADM-inventory-change]
inputs: [rooms, from_date, to_date, capex_ref]
states: [planned, active, completed]
api: POST /v1/properties/{pid}/room-blocks (type=renovation)
events: [RoomsClosedForRenovation]
data: [room_status_block]
rules:
  - Planned closures publish stock reduction immediately to channels; resale date from M66 inspection.
security: gm approval.
failure_cases: [existing_future_reservations]
finance_report_effect: occupancy denominator exclusion noted in reports.
i18n_a11y: none.
acceptance: Planning closure of floor 3 for next month produces a list of affected reservations requiring relocation before activation.
dependency: M66 SF66.2.2.
```

```yaml
id: M03.F03.3.SF03.3.4
name: Return to service with inspection
phase: 2
release: R1
actors: [housekeeping_supervisor, chief_engineer]
screens: [SCR-HK-inspection, SCR-ENG-work-order]
inputs: [block_id, inspection_result, evidence_photos]
states: [pending_inspection, released, reblocked]
api: POST /v1/properties/{pid}/room-blocks/{id}/release
events: [RoomReturnedToService]
data: [room_status_block, hk_inspection]
rules:
  - Release requires engineering completion and housekeeping inspection pass; releases stock for remaining nights.
security: role-scoped.
failure_cases: [inspection_failed]
finance_report_effect: none.
i18n_a11y: mobile accessible.
acceptance: A room released on day 2 of a 3-night OOO block adds availability back only for nights 2 and 3.
dependency: M26 SF26.1.6.
```

## F03.4 Holds, stop-sell, allotments and controlled overbooking

```yaml
id: M03.F03.4.SF03.4.1
name: Inventory holds with expiry
phase: 2
release: R1
actors: [guest, front_desk_agent, sales_manager, ai_assistant]
screens: [SCR-GST-checkout, SCR-FD-new-reservation]
inputs: [quote_id, room_type_id, nights, room_count, hold_reason, ttl_seconds, idempotency_key]
states: [active, converted, expired, released]
api: POST /v1/properties/{pid}/inventory-holds (idem); DELETE .../inventory-holds/{hid}
events: [InventoryHoldPlaced, InventoryHoldExpired, InventoryHoldConverted]
data: [inventory_hold, room_type_inventory_night, inventory_movement]
rules:
  - Hold increments held for each night atomically; default TTL web checkout 15 min, desk 30 min, sales option 72 h (D-106).
  - Conversion to reservation transfers held to sold in one transaction; an expired hold cannot convert.
  - Payment in progress at expiry extends hold once by 5 minutes if PSP authorization is pending.
security: hold creation rate-limited per session/IP to stop inventory hoarding.
failure_cases: [expiry_during_payment, hold_flood_attack]
finance_report_effect: holds excluded from sold KPIs; hold conversion rate in funnel.
i18n_a11y: countdown timer announced at 2 minutes and 30 seconds remaining; option to extend for users needing more time (WCAG timing adjustable).
acceptance: A hold expiring while PSP authorization is pending is extended once; if authorization then fails, the hold expires and availability is restored within 1 minute.
dependency: M28 payment intent state.
```

```yaml
id: M03.F03.4.SF03.4.2
name: Hold expiry worker
phase: 2
release: R1
actors: [hold_expiry_worker]
screens: [SCR-OPS-jobs]
inputs: [now]
states: [running]
api: none
events: [InventoryHoldExpired]
data: [inventory_hold, inventory_movement]
rules:
  - Runs every 30 seconds; expiry is also enforced lazily on read (expired holds never count).
  - Expiry and conversion race resolved by row lock on the hold.
security: service account.
failure_cases: [worker_down]
finance_report_effect: none.
i18n_a11y: none.
acceptance: With the worker stopped, availability queries still exclude expired holds; on restart counters reconcile.
dependency: none.
```

```yaml
id: M03.F03.4.SF03.4.3
name: Stop-sell and close-out
phase: 2
release: R1
actors: [revenue_manager]
screens: [SCR-REV-restrictions]
inputs: [room_type_ids, date_range, channels, rate_plans, reason]
states: [active, lifted]
api: POST /v1/properties/{pid}/stop-sells; DELETE .../stop-sells/{id}
events: [StopSellChanged]
data: [stop_sell]
rules:
  - Stop-sell hides availability to selected channels/rate plans without changing counters; front desk may still sell with override permission.
  - Changes produce ARI deltas (M07 SF07.3.1).
security: revenue_manager.
failure_cases: [channel_ack_missing]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A stop-sell on STD for OTA channels removes STD from OTA ARI within the propagation SLO and leaves direct web selling unaffected.
dependency: M07.
```

```yaml
id: M03.F03.4.SF03.4.4
name: Allotments with release-back cutoff
phase: 3
release: R1
actors: [sales_manager, revenue_manager]
screens: [SCR-SALES-allotments]
inputs: [holder_type (channel/corporate/group), holder_id, room_type_id, nights, quantity, release_days]
states: [active, partially_released, released, expired]
api: POST /v1/properties/{pid}/allotments (idem); PATCH .../allotments/{id}
events: [AllotmentCreated, AllotmentReleased]
data: [allotment, room_type_inventory_night]
rules:
  - Allotted rooms leave general availability; pickup by holder consumes allotment first.
  - At cutoff (release_days before stay date, property time), unused allotment returns to general stock automatically.
security: sales_manager.
failure_cases: [pickup_after_cutoff, over_pickup]
finance_report_effect: pickup vs allotment report for M12/M32 SF32.1.4.
i18n_a11y: none.
acceptance: A corporate allotment of 10 with 4 picked up returns 6 rooms to general availability at 00:00 property time on the cutoff date and emits ARI deltas.
dependency: M10, M12 (Phase 3).
```

```yaml
id: M03.F03.4.SF03.4.5
name: Controlled overbooking policy
phase: 2
release: R1
actors: [revenue_manager, gm]
screens: [SCR-REV-overbooking]
inputs: [room_type_id, date_range, limit_rooms, limit_percent, channels_allowed, approval_level]
states: [draft, active, suspended]
api: PUT /v1/properties/{pid}/overbooking-policies/{id}
events: [OverbookingPolicyChanged]
data: [overbooking_policy, config_version]
rules:
  - Default overbooking limit is 0 (D-105); limits per room type and date; house-level cap across types.
  - Only channels_allowed may consume overbook allowance; OTA channels default excluded.
  - Any sale into overbook allowance records the risk and requires approval above level.
security: gm approves policies; maker-checker.
failure_cases: [limit_without_walk_plan]
finance_report_effect: walk costs tracked (SF05.3.4).
i18n_a11y: none.
acceptance: With limit 2 on DLX for a date, the third oversale is rejected; each accepted oversale appears on the exposure monitor.
dependency: M53 recommendations (Phase 5) advisory only.
```

```yaml
id: M03.F03.4.SF03.4.6
name: Oversell exposure monitor
phase: 2
release: R1
actors: [front_office_manager, revenue_manager, duty_manager]
screens: [SCR-REV-overbooking, SCR-GM-attention-inbox]
inputs: [date_range]
states: [no_exposure, exposed, walk_planned, resolved]
api: GET /v1/properties/{pid}/oversell-exposure
events: [OversellDetected, OversellResolved]
data: [room_type_inventory_night, overbooking_policy]
rules:
  - Exposure = max(0, sold + held − (physical − OOO)) per type/night and house total; includes channel-forced oversells.
  - Exposure for today/tomorrow raises attention item with owner and deadline.
security: none special.
failure_cases: [exposure_from_late_channel_booking]
finance_report_effect: walk cost and relocation expense estimates.
i18n_a11y: none.
acceptance: A channel booking accepted into a sold-out date appears within 1 minute as exposure 1 with owner front_office_manager.
dependency: M07 SF07.4.3.
```

## F03.5 Room assignment (separate from sold stock)

```yaml
id: M03.F03.5.SF03.5.1
name: Assign physical room to reservation room
phase: 2
release: R1
actors: [front_desk_agent, front_office_manager]
screens: [SCR-FD-room-grid, SCR-FD-reservation]
inputs: [reservation_room_id, room_id, from_night, to_night, lock_flag]
states: [unassigned, assigned, locked, checked_in, released]
api: POST /v1/properties/{pid}/room-assignments (idem); DELETE .../room-assignments/{id}
events: [RoomAssigned, RoomUnassigned]
data: [room_assignment]
rules:
  - A room cannot be assigned to two overlapping reservation nights (exclusion constraint on room_id and night range).
  - Assignment of a room of the sold type does not change stock; assignment of a different type is an upgrade/downgrade and goes through SF03.5.3.
  - Locked assignments (guest-requested room) are not moved by auto-assign.
security: front_desk scope.
failure_cases: [overlap, room_ooo, accessible_room_to_non_need_guest_when_protected]
finance_report_effect: none.
i18n_a11y: room grid keyboard-operable; drag-and-drop has a button alternative.
acceptance: Assigning room 210 to two reservations overlapping one night fails with a conflict naming the other reservation.
dependency: none.
```

```yaml
id: M03.F03.5.SF03.5.2
name: Auto-assignment
phase: 2
release: R1
actors: [front_office_manager, assignment_worker]
screens: [SCR-FD-auto-assign]
inputs: [arrival_date, strategy, preferences]
states: [proposed, applied]
api: POST /v1/properties/{pid}/room-assignments/auto (idem)
events: [AutoAssignmentProposed, AutoAssignmentApplied]
data: [room_assignment, hk_room_status]
rules:
  - Priority = accessibility need, VIP, connecting requests, early arrivals, length of stay continuity (minimize future moves), clean-room readiness.
  - Proposal reviewed before apply by default; never overrides locked assignments.
security: none special.
failure_cases: [insufficient_rooms_of_type]
finance_report_effect: none.
i18n_a11y: explanation per assignment in plain text.
acceptance: For arrivals including one guest needing a roll-in shower, auto-assign assigns a room with that attribute or flags the gap; it never assigns that room to another guest while the need is unmet.
dependency: none.
```

```yaml
id: M03.F03.5.SF03.5.3
name: Upgrade and downgrade inventory transfer
phase: 2
release: R1
actors: [front_desk_agent, front_office_manager]
screens: [SCR-FD-reservation]
inputs: [reservation_room_id, target_room_type_id, upgrade_type (complimentary/paid), reason]
states: [requested, applied, reverted]
api: POST /v1/properties/{pid}/reservation-rooms/{rrid}/room-type-change (idem)
events: [RoomTypeChanged, RoomTypeAvailabilityChanged]
data: [reservation_room, room_type_inventory_night, inventory_movement]
rules:
  - Complimentary upgrade keeps rate and rate room type for revenue, but transfers the stock unit from sold type to assigned type for affected nights atomically; refused if target type has no availability.
  - Paid upgrade changes sold type and reprices through M04 SF04.5.5 with guest acceptance.
  - Reason and approver recorded; complimentary upgrades above policy need approval.
security: approval policy for complimentary upgrades.
failure_cases: [target_type_sold_out_on_later_night]
finance_report_effect: revenue stays in booked type for complimentary upgrade; paid upgrade revenue to M54 upsell code.
i18n_a11y: none.
acceptance: A complimentary STD to DLX upgrade for 2 nights decrements DLX and increments STD availability on both nights and leaves folio room charges unchanged.
dependency: M54 paid upgrades (Phase 3).
```

```yaml
id: M03.F03.5.SF03.5.4
name: Assignability and consistency check
phase: 2
release: R1
actors: [night_audit_worker, front_office_manager]
screens: [SCR-FD-assignment-risk]
inputs: [date]
states: [feasible, infeasible]
api: GET /v1/properties/{pid}/assignment-feasibility?date=
events: [AssignmentInfeasible]
data: [room_assignment, room_type_inventory_night, room_status_block]
rules:
  - For each night, check a perfect matching exists between reservation rooms and physical rooms of the sold (or transferred) type excluding OOO; infeasibility flags risk even if counters look fine (e.g. stays needing a room move).
security: none.
failure_cases: [fragmented_availability_requires_move]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A scenario where total counts fit but no single room covers a 3-night stay without a move is flagged with the proposed move.
dependency: none.
```

```yaml
id: M03.F03.5.SF03.5.5
name: Room grid (rooms by dates)
phase: 2
release: R1
actors: [front_desk_agent, front_office_manager]
screens: [SCR-FD-room-grid]
inputs: [date_range, filters]
states: [none]
api: GET /v1/properties/{pid}/room-grid?from=&to=
events: none
data: [room, room_assignment, room_status_block, hk_room_status]
rules:
  - Shows per room per night assigned reservation, OOO/OOS, housekeeping status and unassigned demand counts per type.
  - Incognito rules applied (SF02.3.4).
security: front office scope.
failure_cases: [grid_slow_on_large_property]
finance_report_effect: none.
i18n_a11y: grid with row/column headers for screen readers; RTL flips time axis.
acceptance: The grid for 200 rooms x 14 days renders in under 2 seconds on reference hardware and exposes an accessible table alternative.
dependency: none.
```

```yaml
id: M03.F03.5.SF03.5.6
name: Accessible room protection
phase: 2
release: R1
actors: [revenue_manager, front_office_manager]
screens: [SCR-REV-overbooking, SCR-FD-room-grid]
inputs: [room_type_id, protect_until_days_before_arrival]
states: [protected, released_to_general]
api: PUT /v1/properties/{pid}/accessible-room-protection
events: [AccessibleRoomProtectionReleased]
data: [room, room_assignment]
rules:
  - Accessible rooms are assigned last to guests without declared need; within protection window only guests with need can be assigned.
  - Booking engine lets guests select accessible rooms explicitly by feature.
security: none.
failure_cases: [only_accessible_room_left]
finance_report_effect: none.
i18n_a11y: accessibility features described in words on booking pages.
acceptance: Seven days before arrival, a guest without a declared need is not auto-assigned the last roll-in shower room; after the window it may be assigned.
dependency: local accessibility law per market `unverified-assumption`.
```

### M03 key invariants
1. No ghost sale: for every room type and night, `sold + held + allotted_unreleased ≤ physical − OOO + overbook_allowance` except channel-forced oversells, which always create a visible exposure record.
2. Stock counters change only through `inventory_movement`; the journal replays to the counters.
3. Room assignment never changes sold stock except through the explicit upgrade/downgrade transfer.
4. No physical room is assigned to overlapping nights for two reservations.
5. Expired holds never count as held, even if the expiry worker is down.
6. OOO removes stock; OOS does not.

### M03 module-level acceptance
| Test | Maps to | Statement |
|---|---|---|
| AC-M03-1 | AT-G02.1 | Ten corporate rooms in a composite booking cannot be double sold by concurrent direct and channel bookings. |
| AC-M03-2 | AT-G20.4 | 50 concurrent bookings for the last room → one success; idempotent retries do not decrement twice. |
| AC-M03-3 | AT-G08.2 | Occupancy excludes OOO rooms per KPI dictionary and reconciles to inventory counters. |
| AC-M03-4 | AT-G19.3 | Optional paid upgrade purchase transfers stock and never oversells the target type. |

### M03 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-105 | Default overbooking limits per room type and allowed channels | Revenue Manager (pilot) | 0 by default; direct/desk only when enabled; house cap 2% of physical. |
| D-106 | Default hold TTLs (web, desk, sales option, AI draft) | Front Office Manager + Revenue Manager | Web 15 min, desk 30 min, sales option 72 h, AI draft 10 min. |
| D-124 | Occupancy KPI treatment of OOO/complimentary/house-use rooms | Financial Controller | OOO excluded from available rooms; complimentary counted occupied at zero revenue, reported separately; house use excluded from sold. |

---

# M04 — Rates and quote engine

| Attribute | Value |
|---|---|
| Purpose | Price every stay deterministically: rate plans and negotiated rates, occupancy/child and length-of-stay rules, restrictions, packages and promotions, taxes/fees/rounding/currency (tax rules delegated to M38), and a quote engine that produces an honest total price with a frozen policy snapshot and expiry, supports amendment repricing and records every manual override. |
| Build phase(s) | 2 (core); 3 corporate negotiated rates and packages with M10/M12/M54; 5 M53 recommendations publish into it. |
| Release flag | R1. |
| Bounded context | `rates` (schema `rates`). |
| Systems of record owned | `rate_plan`, `rate_derivation_rule`, `rate_price_night`, `occupancy_price_rule`, `child_age_band`, `los_price_rule`, `restriction`, `package`, `package_component`, `promotion`, `promo_code`, `fee_rule` (non-tax service charges only), `quote`, `quote_line_night`, `policy_snapshot`, `price_override`, `cancellation_policy`, `deposit_policy`. Tax rule content is owned by M38 (`tax_rule_version`); M04 stores the computed result and version reference. |
| Upstream dependencies | M01, M02, M03 (availability), M38/M44 (tax and levy rules, fiscal rounding), M10 (corporate eligibility, Phase 3), M53 (recommended rates, Phase 5). |
| Downstream dependents | M05 (reservation price), M07 (ARI rates, direct engine), M08 (posting schedule), M11/M12 (corporate/group quotes), M40 (AI quote tool), M54 (upsell pricing), M31 (margin basis). |
| External dependencies | FX rate source for display currency — `unverified-assumption` (D-112). Tax/levy content per market — `unverified-assumption` until M38 rule packs are `counsel-reviewed`. |

**Pricing pipeline (normative order):** eligibility (channel, member, corporate, promo) → restrictions → base nightly price (rate_price_night or derived) → occupancy/extra person/child → LOS rule → package components → promotion (stacking rules) → manual override (if any) → service charges/fees → taxes and levies (M38) → rounding (M38 per jurisdiction) → totals by night, by component and by payer.

## F04.1 Rate plans and negotiated rates

```yaml
id: M04.F04.1.SF04.1.1
name: Rate plan definition
phase: 2
release: R1
actors: [revenue_manager]
screens: [SCR-REV-rate-plans, SCR-REV-rate-plan-detail]
inputs: [rate_code, name_by_locale, type (bar/public/member/corporate/package/group/opaque), room_types, meal_plan, cancellation_policy_id, deposit_policy_id, market_segment, valid_from, valid_to, tax_inclusive_display]
states: [draft, active, suspended, retired]
api: POST /v1/properties/{pid}/rate-plans; PATCH .../rate-plans/{rpid}
events: [RatePlanChanged]
data: [rate_plan, cancellation_policy, deposit_policy]
rules:
  - Rate plan binds default cancellation and deposit policies; policies are versioned and snapshotted at quote time.
  - market_segment mandatory for reporting (M32/M53).
  - Retired plans remain on existing reservations.
security: revenue_manager; approval for new public rate plans per policy.
failure_cases: [missing_segment, policy_version_missing_locale]
finance_report_effect: rate_code and segment dimensions on room revenue.
i18n_a11y: guest-facing names and policy text bilingual.
acceptance: Changing a plan's cancellation policy does not alter the snapshot on existing reservations; new quotes use the new version.
dependency: none.
```

```yaml
id: M04.F04.1.SF04.1.2
name: Derived rates
phase: 2
release: R1
actors: [revenue_manager]
screens: [SCR-REV-rate-plan-detail]
inputs: [parent_rate_plan_id, derivation (percent or amount), direction, rounding_rule, floor_price]
states: [active, broken_parent]
api: PUT /v1/properties/{pid}/rate-plans/{rpid}/derivation
events: [DerivedRatesRecalculated]
data: [rate_derivation_rule, rate_price_night]
rules:
  - Derivations form a DAG; cycles rejected; recalculation on parent change emits ARI deltas.
  - Floor price prevents derived rate below configured minimum.
security: revenue_manager.
failure_cases: [cycle, parent_retired]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A member rate derived at BAR minus 10% updates within 1 minute of a BAR change and never goes below its floor.
dependency: none.
```

```yaml
id: M04.F04.1.SF04.1.3
name: Nightly price grid
phase: 2
release: R1
actors: [revenue_manager]
screens: [SCR-REV-rate-grid]
inputs: [rate_plan_id, room_type_id, date_range, price_by_occupancy, day_of_week_pattern]
states: [draft, published]
api: PUT /v1/properties/{pid}/rate-prices (idem, bulk); GET .../rate-prices?from=&to=
events: [RatePricesPublished]
data: [rate_price_night, config_version]
rules:
  - Prices in base currency minor units; bulk edit validated against guardrails (min/max per room type, D-107/M53 guardrails).
  - Publishing creates a version; rollback restores prior version (M53 SF53.2.5).
security: edits above guardrail require approval.
failure_cases: [missing_price_for_sellable_date, guardrail_breach]
finance_report_effect: price history available for pickup/pace analysis.
i18n_a11y: grid keyboard-navigable; changed cells marked with text marker.
acceptance: Publishing a price outside guardrail without approval is rejected; with approval it publishes and the prior version can be restored in one action.
dependency: M53 guardrails (Phase 5); static guardrails Phase 2.
```

```yaml
id: M04.F04.1.SF04.1.4
name: Negotiated corporate rates and eligibility
phase: 3
release: R1
actors: [sales_manager, revenue_manager, corporate_booker]
screens: [SCR-SALES-agreement-rates, SCR-CORP-search]
inputs: [company_id, agreement_version_id, rate_plan_id, fixed_or_discount, room_types, blackout_dates, lra_flag, valid_from, valid_to]
states: [draft, approved, active, expired]
api: POST /v1/properties/{pid}/corporate-rates; GET /v1/corporate/{cid}/eligible-rates
events: [CorporateRateActivated]
data: [rate_plan, corporate_rate_link]
rules:
  - Only principals scoped to the company (or staff booking on its behalf with company selected) see the rate.
  - Last-room-availability (LRA) flag means the rate is sellable whenever the room type is available; non-LRA rates respect closures.
  - Agreement version from M10 is referenced and snapshotted into quotes.
security: corporate scope enforcement (M02 SF02.1.3).
failure_cases: [expired_agreement, rate_leak_to_public]
finance_report_effect: corporate segment revenue and agreement compliance reporting.
i18n_a11y: none.
acceptance: Company A and company B receive different rates for the same room/date (AT-G scenario of two corporations); a public search sees neither.
dependency: M10 (Phase 3).
```

```yaml
id: M04.F04.1.SF04.1.5
name: Rate distribution scope
phase: 2
release: R1
actors: [revenue_manager, integration_admin]
screens: [SCR-REV-rate-plan-detail, SCR-INT-channel-mapping]
inputs: [rate_plan_id, channels, closed_user_groups, visibility]
states: [visible, hidden]
api: PUT /v1/properties/{pid}/rate-plans/{rpid}/distribution
events: [RateDistributionChanged]
data: [rate_plan, channel_mapping]
rules:
  - A rate plan is only sold on channels listed; opaque/member rates require the relevant login or code.
security: none special.
failure_cases: [mapping_missing_for_enabled_channel]
finance_report_effect: channel mix reporting.
i18n_a11y: none.
acceptance: A member-only rate does not appear in anonymous direct search or in any OTA ARI message.
dependency: M07 F07.2.
```

## F04.2 Occupancy, child, length-of-stay and restrictions

```yaml
id: M04.F04.2.SF04.2.1
name: Occupancy-based pricing
phase: 2
release: R1
actors: [revenue_manager]
screens: [SCR-REV-rate-plan-detail]
inputs: [base_occupancy, extra_adult_price, single_occupancy_discount]
states: [active]
api: PUT /v1/properties/{pid}/rate-plans/{rpid}/occupancy-rules
events: [RatePlanChanged]
data: [occupancy_price_rule]
rules:
  - Price for occupancy n = explicit price if defined for n else base + (n − base) × extra_adult_price; max occupancy from room type enforced.
security: revenue_manager.
failure_cases: [occupancy_exceeds_room_max]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A 3-adult quote in a room with max 2 adults returns no offer for that room type rather than a price.
dependency: none.
```

```yaml
id: M04.F04.2.SF04.2.2
name: Child age bands, extra beds and cots
phase: 2
release: R1
actors: [revenue_manager, guest]
screens: [SCR-REV-child-policy, SCR-GST-search]
inputs: [age_bands, price_per_band, free_child_limit, extra_bed_price, cot_price, age_reference (at arrival)]
states: [active]
api: PUT /v1/properties/{pid}/child-policies
events: [ChildPolicyChanged]
data: [child_age_band, occupancy_price_rule]
rules:
  - Child age evaluated at arrival date; bands configurable per property and rate plan (D-122).
  - Extra bed/cot consumes a limited room-level or house-level inventory counter.
security: none.
failure_cases: [age_missing, cot_inventory_exhausted]
finance_report_effect: extra-bed revenue coded separately.
i18n_a11y: age selection accessible (not only slider).
acceptance: A child aged 11 at arrival priced in the 6–11 band even if turning 12 during the stay.
dependency: none.
```

```yaml
id: M04.F04.2.SF04.2.3
name: Length-of-stay pricing
phase: 2
release: R1
actors: [revenue_manager]
screens: [SCR-REV-rate-plan-detail]
inputs: [los_thresholds, discount_or_fixed, stay_pay_offer (e.g. stay 4 pay 3)]
states: [active]
api: PUT /v1/properties/{pid}/rate-plans/{rpid}/los-rules
events: [RatePlanChanged]
data: [los_price_rule]
rules:
  - LOS rule evaluated on total nights of the reservation_room; the discount is allocated across nights proportionally for posting (no negative night).
  - Amendment that shortens the stay below threshold reprices under SF04.5.5.
security: none.
failure_cases: [shortening_breaks_offer]
finance_report_effect: nightly revenue allocation consistent with folio postings.
i18n_a11y: offer explained in text on quote.
acceptance: Stay-4-pay-3 at 100 per night posts 75 per night over 4 nights, and shortening to 3 nights reprices to 100 per night with guest notice.
dependency: none.
```

```yaml
id: M04.F04.2.SF04.2.4
name: Restrictions evaluation
phase: 2
release: R1
actors: [revenue_manager, system]
screens: [SCR-REV-restrictions]
inputs: [rate_plan_ids, room_type_ids, dates, min_los, max_los, min_los_arrival, closed_to_arrival, closed_to_departure, closed, min_advance_days, max_advance_days]
states: [active, lifted]
api: PUT /v1/properties/{pid}/restrictions (bulk, idem)
events: [RestrictionsChanged]
data: [restriction]
rules:
  - min_los on arrival date vs through-date semantics are explicit per restriction; channels receiving only one semantics get a mapped value (M07).
  - Front desk may override restrictions with permission and reason; overrides audited.
security: revenue_manager; override permission for front_office_manager.
failure_cases: [channel_semantic_mismatch]
finance_report_effect: none.
i18n_a11y: quote shows which restriction blocked an option in plain text.
acceptance: A 2-night search arriving on a date with min_los_arrival 3 returns no offer and explains the restriction; a desk override with reason produces the offer.
dependency: none.
```

## F04.3 Packages and promotions

```yaml
id: M04.F04.3.SF04.3.1
name: Package definition and component allocation
phase: 2
release: R1
actors: [revenue_manager, fnb_manager]
screens: [SCR-REV-packages]
inputs: [package_code, components (breakfast/parking/spa_credit), component_price_or_percent, posting_rhythm (per night/per stay/arrival only), outlet_id, revenue_account_ref]
states: [draft, active, retired]
api: POST /v1/properties/{pid}/packages; PATCH .../packages/{id}
events: [PackageChanged]
data: [package, package_component]
rules:
  - Package price is split into room and component revenue at the component's allocated value; components post to their outlets for departmental revenue.
  - Component capacity (e.g. parking) checked via owning module (M09/M17) at quote time.
  - Tax per component per M38 (different tax treatment for room vs F&B vs parking).
security: fnb_manager approves F&B component values.
failure_cases: [component_capacity_unavailable, allocation_exceeds_price]
finance_report_effect: breakfast component revenue to F&B department, room remainder to rooms.
i18n_a11y: inclusions listed in text.
acceptance: A 120 bed-and-breakfast package with breakfast allocated 15 posts 105 room revenue and 15 F&B revenue per night with respective tax codes.
dependency: M38 product tax treatment; M17 parking capacity (Phase 3).
```

```yaml
id: M04.F04.3.SF04.3.2
name: Promotions and promo codes
phase: 2
release: R1
actors: [marketing_manager, revenue_manager, guest]
screens: [SCR-REV-promotions, SCR-GST-checkout]
inputs: [promo_code, discount_type, value, eligible_rate_plans, stay_window, booking_window, usage_limit, per_guest_limit, channels]
states: [draft, active, exhausted, expired]
api: POST /v1/properties/{pid}/promotions; POST /v1/properties/{pid}/quotes/{qid}/apply-promo
events: [PromotionRedeemed]
data: [promotion, promo_code]
rules:
  - Usage counters decremented atomically at booking confirmation, restored on cancellation within policy.
  - Codes case-insensitive, brute-force rate-limited.
security: marketing_manager creates; revenue_manager approves discount above threshold.
failure_cases: [code_enumeration, usage_race]
finance_report_effect: discount tracked as promotion cost for campaign ROI (M52/M51).
i18n_a11y: error messages explain why a code is not valid.
acceptance: A code with usage_limit 1 applied by two concurrent checkouts results in one redemption and one clear rejection.
dependency: none.
```

```yaml
id: M04.F04.3.SF04.3.3
name: Stacking, exclusions and best-offer selection
phase: 2
release: R1
actors: [revenue_manager]
screens: [SCR-REV-promotions]
inputs: [stacking_policy, exclusion_groups, priority]
states: [active]
api: PUT /v1/properties/{pid}/promotion-policy
events: none
data: [promotion]
rules:
  - Default no stacking; exclusion groups prevent combining corporate and promo discounts.
  - Search results show each eligible offer; "best price" label only on the lowest total with equal policy terms.
security: none.
failure_cases: [misleading_best_label]
finance_report_effect: none.
i18n_a11y: comparison accessible.
acceptance: A corporate rate and a promo code are never combined when in the same exclusion group; the quote shows which one applied.
dependency: none.
```

```yaml
id: M04.F04.3.SF04.3.4
name: Package posting schedule
phase: 2
release: R1
actors: [night_audit_worker]
screens: [SCR-FD-folio]
inputs: [reservation_room_id, package_id]
states: [scheduled, posted, consumed, expired_unconsumed]
api: none (generated at confirmation); GET /v1/properties/{pid}/reservations/{rid}/posting-schedule
events: [PackageComponentPosted]
data: [package_component, folio_line]
rules:
  - Posting schedule generated from quote lines; night audit posts per business date.
  - Unconsumed component policy (e.g. missed breakfast) follows package rule (breakage to package revenue or outlet).
security: none.
failure_cases: [schedule_drift_after_amendment]
finance_report_effect: package breakage accounted per policy.
i18n_a11y: none.
acceptance: A 3-night B&B stay generates 3 room and 3 breakfast postings on the correct business dates; shortening the stay removes the unposted third pair.
dependency: M08 F08.6.
```

## F04.4 Taxes, fees, rounding and currency

```yaml
id: M04.F04.4.SF04.4.1
name: Tax computation through jurisdiction port
phase: 2
release: R1
actors: [system, compliance_officer]
screens: [SCR-GST-checkout, SCR-FD-reservation]
inputs: [property_legal_entity, product_type, stay_date, amount, guest_type, exemption_evidence_ref]
states: [computed, rule_missing, rule_unverified]
api: internal TaxPort.compute
events: [TaxRuleMissing]
data: [quote_line_night, tax_rule_version (M38)]
rules:
  - Tax is computed per night and per component using the rule version effective on the stay date (or supply date as defined per market by M38).
  - If rule status is not verified for the market, quote displays estimate and the property's gate decides whether selling is allowed (M44 SF44.2.6).
  - Tax-inclusive display markets show inclusive price but store net + tax lines.
security: rule content editable only in M38.
failure_cases: [rule_missing, rate_change_mid_stay]
finance_report_effect: tax liability lines by code for M38 remittance.
i18n_a11y: tax names bilingual per M38.
acceptance: A stay spanning a tax-rate change date shows different tax per night according to each night's effective rule version.
dependency: M38 rule packs (`unverified-assumption` per market until reviewed).
```

```yaml
id: M04.F04.4.SF04.4.2
name: Service charges, municipal and tourism levies
phase: 2
release: R1
actors: [financial_controller, compliance_officer]
screens: [SCR-FIN-fee-rules]
inputs: [fee_code, basis (percent/per room night/per person night), applies_to_products, taxable_flag, jurisdiction_rule_ref]
states: [active, retired]
api: PUT /v1/properties/{pid}/fee-rules/{code}
events: [FeeRuleChanged]
data: [fee_rule]
rules:
  - Hotel service charges are M04 fee rules; government levies are M38 rules referenced here, never hard-coded.
  - Order of application (fee before or after tax) defined per rule and market.
security: financial_controller.
failure_cases: [circular_tax_on_fee]
finance_report_effect: fees post to separate revenue/liability accounts.
i18n_a11y: none.
acceptance: A per-person-per-night levy for 2 adults and 3 nights yields 6 units on the quote and 3 nightly folio lines.
dependency: M38.
```

```yaml
id: M04.F04.4.SF04.4.3
name: Rounding rules
phase: 2
release: R1
actors: [financial_controller]
screens: [SCR-FIN-fee-rules]
inputs: [rounding_mode, rounding_level (line/invoice), currency]
states: [active]
api: GET /v1/properties/{pid}/rounding-policy
events: none
data: [config_version]
rules:
  - Rounding level and mode come from M38 per jurisdiction; quote total equals sum of nightly lines exactly (residual allocated to last night).
security: none.
failure_cases: [total_mismatch_with_folio]
finance_report_effect: folio total equals quote total for unchanged stays.
i18n_a11y: none.
acceptance: For 1,000 randomized quotes, folio totals after posting equal quoted totals to the minor unit.
dependency: M38.
```

```yaml
id: M04.F04.4.SF04.4.4
name: Display currency and FX estimate
phase: 2
release: R1
actors: [guest]
screens: [SCR-GST-search, SCR-GST-checkout]
inputs: [display_currency, fx_rate_source, fx_timestamp]
states: [fresh, stale]
api: GET /v1/fx-rates?base=
events: none
data: [fx_rate]
rules:
  - Charges are in property base currency; converted amounts labeled estimate with rate timestamp; card issuer conversion disclaimer shown.
  - Stale FX (> 24 h) hides converted amounts.
security: none.
failure_cases: [fx_feed_down]
finance_report_effect: none.
i18n_a11y: estimate label read by screen readers.
acceptance: With FX feed stale, the booking page shows only base-currency prices.
dependency: FX source `unverified-assumption` (D-112).
```

```yaml
id: M04.F04.4.SF04.4.5
name: All-in total price transparency
phase: 2
release: R1
actors: [guest, corporate_booker]
screens: [SCR-GST-search, SCR-GST-checkout, SCR-CORP-quote]
inputs: [quote]
states: [none]
api: part of quote response
events: none
data: [quote, quote_line_night]
rules:
  - Headline price includes all mandatory fees and taxes where the market requires or the property chooses; optional extras shown separately.
  - The amount shown at checkout equals the amount confirmed; any difference forces a re-confirm step.
security: none.
failure_cases: [hidden_mandatory_fee]
finance_report_effect: reduces disputes/chargebacks.
i18n_a11y: price breakdown as accessible list.
acceptance: No path exists where the confirmation total differs from the last displayed checkout total without an explicit re-acceptance.
dependency: consumer price-display rules per market `unverified-assumption`.
```

## F04.5 Quote engine, policy snapshot, expiry and overrides

```yaml
id: M04.F04.5.SF04.5.1
name: Generate quote
phase: 2
release: R1
actors: [guest, front_desk_agent, corporate_booker, ai_assistant, sales_manager]
screens: [SCR-GST-search, SCR-FD-new-reservation, SCR-CORP-quote]
inputs: [arrival, departure, occupancy_by_room, rate_plan_filter, promo_code, company_id, channel, currency]
states: [draft, offered, held, accepted, expired, superseded]
api: POST /v1/properties/{pid}/quotes
events: [QuoteCreated]
data: [quote, quote_line_night, policy_snapshot]
rules:
  - Applies the normative pipeline; each quote_line_night stores base, adjustments, fees, taxes, rule versions and component split.
  - Quote references availability at creation but does not reserve stock unless a hold is placed (SF03.4.1).
security: public quotes rate-limited; corporate rates only with scope.
failure_cases: [no_availability, tax_rule_unverified, pricing_timeout]
finance_report_effect: quotes feed funnel and conversion KPIs.
i18n_a11y: quote summary accessible and bilingual.
acceptance: A quote response contains per-night lines whose sum equals the total, with rule version ids for price, tax and policy.
dependency: M38 tax port.
```

```yaml
id: M04.F04.5.SF04.5.2
name: Policy snapshot
phase: 2
release: R1
actors: [system]
screens: [SCR-GST-checkout, SCR-FD-reservation]
inputs: [quote_id]
states: [frozen]
api: GET /v1/properties/{pid}/quotes/{qid}/policy
events: none
data: [policy_snapshot]
rules:
  - Snapshot freezes cancellation terms (deadlines in property time), deposit schedule, no-show fee, check-in/out times, child policy, tax/rule versions, and legal text version shown.
  - Snapshot is immutable and copied to the reservation on conversion; later policy edits never change it.
security: none.
failure_cases: [snapshot_missing_locale]
finance_report_effect: cancellation/no-show fees computed only from snapshot.
i18n_a11y: cancellation deadline shown with date, time and time zone in words.
acceptance: After the hotel changes cancellation policy, a reservation made earlier is charged the fee per its snapshot, not the new policy.
dependency: none.
```

```yaml
id: M04.F04.5.SF04.5.3
name: Quote expiry and hold linkage
phase: 2
release: R1
actors: [guest, sales_manager, corporate_booker]
screens: [SCR-GST-checkout, SCR-CORP-quote]
inputs: [quote_id, valid_until, hold_id]
states: [offered, expired]
api: POST /v1/properties/{pid}/quotes/{qid}/extend
events: [QuoteExpired]
data: [quote, inventory_hold]
rules:
  - Every quote has valid_until (default web 15 min matching hold, corporate/sales firm quote per D-107).
  - A firm corporate quote guarantees price until expiry only if a hold exists; without hold it states "price guaranteed, availability not guaranteed".
  - Expired quotes cannot convert; reconversion creates a new quote.
security: extension requires sales permission.
failure_cases: [conversion_after_expiry]
finance_report_effect: quote expiry rate in funnel.
i18n_a11y: expiry shown in property time and viewer's local time.
acceptance: Converting a quote one second after valid_until fails with quote_expired and returns a fresh quote for re-acceptance.
dependency: none.
```

```yaml
id: M04.F04.5.SF04.5.4
name: Convert quote to reservation price
phase: 2
release: R1
actors: [system]
screens: [SCR-GST-checkout]
inputs: [quote_id, hold_id, idempotency_key]
states: [converted]
api: POST /v1/properties/{pid}/reservations (with quote_id, idem)
events: [QuoteAccepted]
data: [quote, reservation_room, policy_snapshot]
rules:
  - Reservation stores quote lines as booked price lines; recompute only for verification—on mismatch (rule changed after quote) the quoted price wins within validity.
security: none.
failure_cases: [double_submit]
finance_report_effect: booked revenue = quoted revenue.
i18n_a11y: none.
acceptance: A double-submitted checkout with the same idempotency key yields one reservation.
dependency: M05 SF05.1.1.
```

```yaml
id: M04.F04.5.SF04.5.5
name: Amendment repricing
phase: 2
release: R1
actors: [front_desk_agent, guest, corporate_booker]
screens: [SCR-FD-reservation-amend, SCR-GST-manage-booking]
inputs: [reservation_room_id, change_set (dates/room type/occupancy/rate plan)]
states: [preview, accepted, rejected]
api: POST /v1/properties/{pid}/reservations/{rid}/amendment-quotes; POST .../amendment-quotes/{aqid}/accept (idem)
events: [ReservationRepriced]
data: [quote, quote_line_night, policy_snapshot]
rules:
  - Unchanged nights keep their original price and policy; added or changed nights priced at current rates (property option to honor original rate for extensions, D-107).
  - Price delta and any new policy terms shown for explicit acceptance before commit.
  - LOS/package offers re-evaluated on total stay.
security: amendments to corporate bookings require corporate scope.
failure_cases: [original_rate_retired, availability_lost_for_new_nights]
finance_report_effect: unposted nights updated; posted nights never altered (corrections via M08).
i18n_a11y: before/after comparison accessible.
acceptance: Extending a 3-night stay by 1 night keeps the first 3 nights' price and prices the 4th at current rate, shown as a delta for acceptance.
dependency: none.
```

```yaml
id: M04.F04.5.SF04.5.6
name: Manual price override with audit
phase: 2
release: R1
actors: [front_desk_agent, front_office_manager, revenue_manager]
screens: [SCR-FD-reservation, SCR-GM-approvals-inbox]
inputs: [quote_or_reservation_ref, nights, new_price, reason_code, comment]
states: [requested, approved, applied, rejected]
api: POST /v1/properties/{pid}/price-overrides (idem)
events: [PriceOverridden]
data: [price_override, approval_request]
rules:
  - Overrides within role limit apply immediately; beyond limit need approval (M02 F02.4).
  - Original computed price retained next to override; reports show override variance.
security: step-up for overrides above threshold; SoD with approval.
failure_cases: [override_below_floor, approval_after_quote_expired]
finance_report_effect: override variance reported to revenue_manager and M60.
i18n_a11y: none.
acceptance: A 40% override by a front_desk_agent with 10% limit is held pending approval and the quote cannot be converted until approved.
dependency: M02 F02.4.
```

```yaml
id: M04.F04.5.SF04.5.7
name: Quote API for channels, corporate portal and AI assistant
phase: 2
release: R1
actors: [ai_assistant, corporate_booker, channel adapter]
screens: [SCR-GST-chat, SCR-CORP-search]
inputs: [search_params, caller_scope]
states: [none]
api: POST /v1/properties/{pid}/quotes (scoped); GET /v1/properties/{pid}/quotes/{qid}
events: [QuoteCreated]
data: [quote]
rules:
  - AI tool receives only the priced offers and policy text with source/freshness; it may create a draft quote/hold but never confirm a booking (M40 SF40.2.2).
  - Same pipeline for all callers; no caller-specific price logic outside eligibility.
security: scoped service accounts; PII not required for quoting.
failure_cases: [tool_abuse_rate_limit]
finance_report_effect: channel attribution per quote.
i18n_a11y: none.
acceptance: The AI assistant's quote for given dates equals the web quote for the same eligibility to the minor unit.
dependency: M40 (Phase 3–5).
```

```yaml
id: M04.F04.5.SF04.5.8
name: Quote reproducibility and audit
phase: 2
release: R1
actors: [auditor, revenue_manager]
screens: [SCR-REV-quote-audit]
inputs: [quote_id]
states: [reproduced, mismatch]
api: POST /v1/properties/{pid}/quotes/{qid}/reproduce
events: none
data: [quote, config_version, rate_price_night]
rules:
  - Given stored rule versions, the engine reproduces a historical quote exactly; mismatch indicates a defect and raises an alert.
security: auditor read-only.
failure_cases: [version_purged]
finance_report_effect: supports dispute resolution and M60.
i18n_a11y: none.
acceptance: Reproducing a 6-month-old quote yields the identical per-night lines and total.
dependency: none.
```

### M04 key invariants
1. Quote total = Σ nightly lines = folio postings for an unchanged stay, to the minor unit.
2. Policy snapshot is immutable; fees for cancellation/no-show derive only from it.
3. Posted nights are never repriced; amendments affect only unposted nights.
4. Tax content comes only from M38; an unverified rule is displayed as estimate and gated per market.
5. Every override stores original price, reason, actor and approval.
6. Corporate/member rates are never visible outside their eligibility scope.

### M04 module-level acceptance
| Test | Maps to | Statement |
|---|---|---|
| AC-M04-1 | AT-G01.1 | Corporate search shows only configurations at the company's contracted rates. |
| AC-M04-2 | AT-G12.1 | Canadian province bundle quote uses the effective-dated tax/levy configuration from M38 (rule status shown). |
| AC-M04-3 | AT-G19.4 | Direct booking with optional upgrade shows a correct all-in total equal to the charged total. |
| AC-M04-4 | AT-G19.5 | Revenue manager's guardrailed rate change is versioned and reversible. |
| AC-M04-5 | AT-G09.2 | Five-market fixtures produce different tax lines per market from distinct rule packs. |

### M04 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-107 | Default quote validity for corporate/sales firm quotes and extension pricing (honor original rate?) | Revenue Manager + Sales Manager | Firm quotes 7 days; extensions priced at current rate unless desk honors original with permission. |
| D-108 | Tax-inclusive vs exclusive display per market | Compliance Officer + counsel per market | Inclusive display in Oman, Saudi, Portugal, Pakistan; exclusive with taxes shown before payment in Canada — all `unverified-assumption`. |
| D-112 | Display-currency FX source and multi-currency settlement | Financial Controller | Display estimate only; settlement in base currency; FX source TBD. |
| D-122 | Child age bands and free-child policy | Revenue Manager | 0–5 free, 6–11 child price, 12+ adult. |

---

# M05 — Reservations/stays

| Attribute | Value |
|---|---|
| Purpose | Own the reservation from creation (direct, desk, walk-in, channel, group) through amendment, cancellation, no-show, walk and waitlist, to arrival, check-in (registration and ID policy), in-stay room moves and extensions, checkout and post-departure charges; distinguish booker, occupants and payer; emit entitlement events for keys, parking and connected devices. |
| Build phase(s) | 2 (direct, desk, walk-in, core lifecycle, check-in/out); 3 channel ingest at scale, groups/corporate, device entitlements; 5 messaging-based pre-arrival; 7 key/UC/HSIA/IPTV consumers. |
| Release flag | R1 (device consumers for UC/HSIA/IPTV: Later). |
| Bounded context | `reservations` (schema `reservations`). |
| Systems of record owned | `reservation`, `reservation_room`, `reservation_night` (booked price lines copied from quote), `stay`, `reservation_party`, `guest_profile` (single guest master), `guarantee`, `special_request`, `waitlist_entry`, `registration_record`, `room_move`, `walk_record`, `device_entitlement`, `reservation_history`. |
| Upstream dependencies | M01, M02 (consent, scopes), M03 (stock, assignment), M04 (quote, snapshot), M28 (payment tokens/preauth), M41 (ID OCR, e-sign, OTP), M44 (guest registration rules), M07 (channel bookings), M10/M12 (corporate/groups, Phase 3). |
| Downstream dependents | M06 (arrivals/departures, HK tasks), M08 (folio creation, postings), M07 (booking acks to channels), M17 parking, M18/M52/M55 guest journey, M30/M31 loyalty/referral qualification, M32 KPIs, M34–M36 device consumers (Later), M43 lost-and-found matching. |
| External dependencies | Guest registration/police reporting interfaces per market — `unverified-assumption` (manual export path until verified); door lock systems — optional adapter, `unverified-assumption` (Phase 7 hardware; manual key encoding path in R1). |

**Reservation states (SM-reservation, detailed in docs/02):** `tentative` (option/hold) → `confirmed` → `in_house` (at least one room checked in) → `checked_out`; side exits `cancelled`, `no_show`, `walked` (relocated), `waitlisted` (no stock consumed). Each `reservation_room` has its own state; the reservation state is derived.

**Party model:** `reservation_party{reservation_id, guest_profile_id, role ∈ (booker, primary_guest, occupant, payer), reservation_room_id?}`. Booker may be a non-staying person, an agent, or a corporate_user; occupants are per room; payer is linked to a folio window (M08). Only occupants are registered and ID-checked; only payers see folio charges routed to them.

## F05.1 Booking creation by source

```yaml
id: M05.F05.1.SF05.1.1
name: Direct web/app booking confirmation
phase: 2
release: R1
actors: [guest, booker]
screens: [SCR-GST-checkout, SCR-GST-confirmation]
inputs: [quote_id, hold_id, booker_details, occupant_details, payment_intent_ref, consents, special_requests, attribution_ref, idempotency_key]
states: [pending_payment, confirmed, failed]
api: POST /v1/properties/{pid}/reservations (idem)
events: [ReservationConfirmed, ReservationFailed]
data: [reservation, reservation_room, reservation_night, reservation_party, guarantee, policy_snapshot]
rules:
  - Confirmation requires valid hold, unexpired quote, guarantee per policy snapshot (preauth, deposit or none) and acceptance of terms version.
  - Payment and booking are a saga; if payment succeeds but booking commit fails, the payment is voided/refunded automatically and incident logged; never a charged guest without a reservation.
  - Confirmation number unique per property, non-sequential for guests (avoid enumeration).
security: guest session or anonymous checkout with email verification link to manage booking.
failure_cases: [payment_declined, hold_expired, psp_timeout, duplicate_submit]
finance_report_effect: booked room nights and revenue on-the-books; deposit liability if collected.
i18n_a11y: confirmation page and email bilingual; accessible confirmation summary.
acceptance: A PSP timeout followed by success callback yields exactly one confirmed reservation and one authorization; a commit failure after authorization triggers a void within 5 minutes.
dependency: M28 PSP (`unverified-assumption` until a PSP is `sandbox-tested`); M07 F07.1 UI.
```

```yaml
id: M05.F05.1.SF05.1.2
name: Desk and phone reservation
phase: 2
release: R1
actors: [front_desk_agent, sales_manager]
screens: [SCR-FD-new-reservation]
inputs: [quote_id, guest_profile_id, booker, occupants, guarantee_type, source_code, segment]
states: [tentative, confirmed]
api: POST /v1/properties/{pid}/reservations (idem)
events: [ReservationCreated, ReservationConfirmed]
data: [reservation, reservation_party, guarantee]
rules:
  - Card details for phone bookings captured only through PSP secure capture (pay-by-link or iframe), never typed into PMS fields.
  - Tentative (option) reservations carry an option date; unconfirmed options auto-release at option date.
security: PCI scope minimized; agent cannot see full PAN.
failure_cases: [option_lapsed, guest_duplicate_suspected]
finance_report_effect: segment/source coded.
i18n_a11y: form keyboard-efficient; bilingual guest name entry.
acceptance: An option reservation not confirmed by its option date is released and its stock returned, with an event to the sales owner.
dependency: M28 pay-by-link.
```

```yaml
id: M05.F05.1.SF05.1.3
name: Walk-in reservation and immediate check-in
phase: 2
release: R1
actors: [front_desk_agent]
screens: [SCR-FD-walk-in]
inputs: [nights, occupancy, room_id, guest_details, payment_or_preauth]
states: [confirmed, in_house]
api: POST /v1/properties/{pid}/walk-ins (idem)
events: [ReservationConfirmed, GuestCheckedIn]
data: [reservation, stay, room_assignment]
rules:
  - Combines quote, hold, reservation and check-in in one guided flow; requires a clean/inspected vacant room.
  - Guarantee mandatory (preauth or cash deposit) per policy.
security: none special.
failure_cases: [no_clean_room, preauth_declined]
finance_report_effect: walk-in source coded.
i18n_a11y: guided steps with progress indicator.
acceptance: A walk-in completes in one flow and the room status becomes occupied; with no clean room the flow offers room-ready ETA instead of checking in.
dependency: M06 SF06.2.5.
```

```yaml
id: M05.F05.1.SF05.1.4
name: Channel booking domain ingest
phase: 2
release: R1
actors: [channel_worker]
screens: [SCR-INT-channel-bookings]
inputs: [channel_booking_message_id, channel_reservation_id, mapped_room_type, mapped_rate_plan, price_lines, guest, payment_model]
states: [received, created, modified, cancelled, needs_attention]
api: internal command from M07 SF07.4.1
events: [ReservationConfirmed, ReservationModified, ReservationCancelled]
data: [reservation, reservation_night, source_attribution]
rules:
  - Channel price is stored as booked; discrepancy against PMS price creates a rate-parity exception but never rejects the booking.
  - Channel bookings consume stock even if oversold (channel already sold it) and create exposure (SF03.4.6).
security: channel service account.
failure_cases: [unmapped_room, modification_for_unknown_booking]
finance_report_effect: commission accrual via M07 SF07.5.2.
i18n_a11y: none.
acceptance: A channel booking with a price differing from PMS is created at the channel price and a parity exception is raised.
dependency: M07 channel manager (`unverified-assumption`).
```

```yaml
id: M05.F05.1.SF05.1.5
name: Group and corporate block pickup
phase: 3
release: R1
actors: [sales_manager, event_organizer, corporate_booker]
screens: [SCR-SALES-block-pickup, SCR-CORP-rooming-list]
inputs: [block_id, rooming_list, billing_instructions]
states: [picked_up, released]
api: POST /v1/properties/{pid}/blocks/{bid}/pickups (idem, bulk)
events: [BlockPickedUp]
data: [reservation, allotment]
rules:
  - Pickup consumes block allotment first; beyond block requires general availability and rate decision.
  - Rooming list import validates occupants and routes charges to master folio per instructions.
security: corporate scope.
failure_cases: [rooming_list_over_block, cutoff_passed]
finance_report_effect: block pickup reporting (M32 SF32.1.4).
i18n_a11y: bulk import error report bilingual.
acceptance: Importing a 10-room rooming list into a 10-room block creates 10 reservations routed to the event master folio for room and tax.
dependency: M12 (Phase 3).
```

```yaml
id: M05.F05.1.SF05.1.6
name: Multi-room reservation and shared itinerary
phase: 2
release: R1
actors: [front_desk_agent, booker]
screens: [SCR-FD-reservation, SCR-GST-manage-booking]
inputs: [reservation_rooms, shared_arrival, linked_reservations]
states: [partially_confirmed, confirmed]
api: POST /v1/properties/{pid}/reservations (multi-room, idem)
events: [ReservationConfirmed]
data: [reservation, reservation_room]
rules:
  - All rooms confirmed atomically unless booker accepts partial; each room has own occupants, state and folio windows.
security: none.
failure_cases: [partial_availability]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A 3-room booking where only 2 rooms are available fails atomically unless the booker explicitly accepts 2.
dependency: none.
```

## F05.2 Booker, occupants, payer and guest profile

```yaml
id: M05.F05.2.SF05.2.1
name: Party roles on a reservation
phase: 2
release: R1
actors: [front_desk_agent, booker, corporate_booker]
screens: [SCR-FD-reservation-parties]
inputs: [reservation_id, guest_profile_id, role, reservation_room_id]
states: [active, removed]
api: POST /v1/properties/{pid}/reservations/{rid}/parties; DELETE .../parties/{party_id}
events: [ReservationPartyChanged]
data: [reservation_party]
rules:
  - Exactly one booker and at least one primary_guest per reservation_room before check-in; payer may be booker, occupant, company or third party.
  - Booker sees booking details and confirmation; booker sees folio charges only if also payer or granted by payer.
  - Occupant personal data (ID, nationality) is not exposed to booker.
security: record-level visibility by role in party.
failure_cases: [checkin_without_primary_guest, booker_requests_occupant_id]
finance_report_effect: payer drives folio window ownership (M08 SF08.2.1).
i18n_a11y: role labels bilingual.
acceptance: A corporate booker who is not payer cannot view the incidental window of an occupant's folio.
dependency: M08.
```

```yaml
id: M05.F05.2.SF05.2.2
name: Guest profile match and create
phase: 2
release: R1
actors: [front_desk_agent, system]
screens: [SCR-FD-guest-search, SCR-FD-guest-profile]
inputs: [names, email, phone, date_of_birth, document_hash]
states: [new, matched, suspected_duplicate, merged]
api: POST /v1/properties/{pid}/guests; GET /v1/properties/{pid}/guests/search; POST .../guests/merge-suggestions
events: [GuestProfileCreated, GuestProfilesMerged]
data: [guest_profile]
rules:
  - Match candidates scored on verified email/phone and document hash; never auto-merge on name only; merges are reviewed and reversible (M52 SF52.1.1 governs cross-channel).
  - Profile carries consent refs, preferences with purpose tags, stay history.
security: search results masked per field policy.
failure_cases: [false_merge, arabic_latin_variants]
finance_report_effect: repeat-guest metrics.
i18n_a11y: dual-script names; transliteration search.
acceptance: Two profiles with same name but different verified emails are not merged; a reviewed merge can be undone restoring both histories.
dependency: M52 (Phase 3) extends.
```

```yaml
id: M05.F05.2.SF05.2.3
name: Guarantee and payer setup
phase: 2
release: R1
actors: [front_desk_agent, booker]
screens: [SCR-FD-reservation-guarantee]
inputs: [guarantee_type (card_token/deposit/corporate_billing/voucher/none), payment_token_ref, company_account_id, deposit_schedule]
states: [pending, guaranteed, failed, released]
api: PUT /v1/properties/{pid}/reservations/{rid}/guarantee (idem)
events: [ReservationGuaranteed, GuaranteeFailed]
data: [guarantee, payment_authorization_ref]
rules:
  - Guarantee type must satisfy policy snapshot; corporate billing requires active AR account with credit (M20) or approved PO.
  - Token stored as PSP reference only; consent for storing card-on-file recorded (M28).
security: no PAN in PMS; token usage scoped to reservation.
failure_cases: [token_expired, company_credit_exceeded]
finance_report_effect: deposit liability or AR exposure.
i18n_a11y: none.
acceptance: A reservation requiring deposit whose deposit is not received by due date moves to guarantee_failed and appears on the front office attention list.
dependency: M28 tokenization; M20 AR (Phase 4; manual credit limit before).
```

```yaml
id: M05.F05.2.SF05.2.4
name: Special requests and accessibility needs routing
phase: 2
release: R1
actors: [guest, front_desk_agent, housekeeping_supervisor]
screens: [SCR-GST-checkout, SCR-FD-reservation, SCR-HK-task-detail]
inputs: [request_type, details, needs_category, consent_for_sensitive]
states: [requested, acknowledged, fulfilled, not_possible]
api: POST /v1/properties/{pid}/reservations/{rid}/requests
events: [SpecialRequestCreated, SpecialRequestFulfilled]
data: [special_request]
rules:
  - Requests are routed to owning team (HK, engineering, F&B) as tasks with due time before arrival.
  - Accessibility and dietary needs are sensitive; stored with purpose limitation and visible only to teams that need them.
  - A not-possible outcome requires guest notification before arrival.
security: sensitive category access limited.
failure_cases: [request_after_cutoff]
finance_report_effect: none.
i18n_a11y: free-text plus structured options; bilingual.
acceptance: A guest's request for a visual fire alarm creates an HK/engineering task due before arrival and blocks auto-assignment to rooms without that feature.
dependency: M55 (Phase 2–3).
```

```yaml
id: M05.F05.2.SF05.2.5
name: Service communications vs marketing
phase: 2
release: R1
actors: [system, guest]
screens: [SCR-GST-manage-booking]
inputs: [reservation_id, message_type, channel]
states: [queued, sent, failed]
api: POST /v1/properties/{pid}/reservations/{rid}/messages
events: [ReservationMessageSent]
data: [message_log, consent_record]
rules:
  - Transactional messages (confirmation, change, cancellation, pre-arrival logistics) sent to booker and, where contact known, primary guest; marketing only per consent (M02).
  - Messaging via email in Phase 2; SMS/WhatsApp only via approved adapters.
security: templates approved.
failure_cases: [bounce, channel_not_approved]
finance_report_effect: none.
i18n_a11y: language by guest preference; RTL email templates.
acceptance: A confirmation email contains no marketing content when marketing consent is absent.
dependency: M41/M52 adapters (`unverified-assumption` for SMS/WhatsApp).
```

## F05.3 Amendments, cancellation, no-show, walk and waitlist

```yaml
id: M05.F05.3.SF05.3.1
name: Amend reservation
phase: 2
release: R1
actors: [front_desk_agent, guest, corporate_booker, channel_worker]
screens: [SCR-FD-reservation-amend, SCR-GST-manage-booking]
inputs: [reservation_room_id, amendment_quote_id, idempotency_key]
states: [confirmed]
api: POST /v1/properties/{pid}/reservations/{rid}/amendments (idem)
events: [ReservationModified]
data: [reservation_room, reservation_night, inventory_movement, reservation_history]
rules:
  - Stock swap for changed nights/types is atomic (release old, consume new) with no window of double sale or loss.
  - Repricing per SF04.5.5; guarantee re-evaluated.
  - Every change produces a history version with actor and diff; channel-originated reservations amended at desk flag channel sync mismatch.
security: scope by source (channel bookings modifiable at desk only with permission).
failure_cases: [new_nights_unavailable, amendment_on_checked_out]
finance_report_effect: on-the-books updated; posting schedule regenerated for unposted nights.
i18n_a11y: diff shown accessibly.
acceptance: Changing room type for nights 2–3 of an in-house stay releases old type and consumes new type in one transaction; a failure leaves both unchanged.
dependency: M03, M04.
```

```yaml
id: M05.F05.3.SF05.3.2
name: Cancel reservation with snapshot fee
phase: 2
release: R1
actors: [guest, front_desk_agent, channel_worker]
screens: [SCR-GST-manage-booking, SCR-FD-reservation]
inputs: [reservation_id, reason_code, waive_fee_request]
states: [cancelled]
api: POST /v1/properties/{pid}/reservations/{rid}/cancel (idem)
events: [ReservationCancelled, CancellationFeeAssessed]
data: [reservation, policy_snapshot, inventory_movement]
rules:
  - Fee computed from snapshot deadline in property time; deposit applied or refunded accordingly (M08 SF08.3.6).
  - Waiver requires approval; stock released immediately.
  - Cancellation number issued; confirmation sent.
security: guest self-cancel only via verified manage-booking link or account.
failure_cases: [fee_charge_declined, cancel_after_arrival]
finance_report_effect: cancellation fee revenue, refund liability, loyalty/referral reversal events (M30/M31).
i18n_a11y: fee explained before confirm.
acceptance: Cancelling 1 hour after the snapshot deadline charges the snapshot fee; cancelling 1 hour before charges none; both release stock immediately.
dependency: M08, M28.
```

```yaml
id: M05.F05.3.SF05.3.3
name: No-show processing
phase: 2
release: R1
actors: [night_auditor, night_audit_worker]
screens: [SCR-FIN-night-audit-noshow]
inputs: [business_date, due_arrivals_not_checked_in]
states: [candidate, no_show, reinstated]
api: POST /v1/properties/{pid}/night-audit/{run}/no-shows (idem)
events: [ReservationNoShow, NoShowFeeAssessed]
data: [reservation, policy_snapshot]
rules:
  - At night audit, guaranteed arrivals not checked in are listed for review; auditor confirms no-show or keeps (late arrival).
  - No-show releases remaining nights' stock per policy (release all or keep first night).
  - Fee per snapshot; charge via token with PSP result recorded.
security: night_auditor.
failure_cases: [late_arrival_after_noshow, token_charge_declined]
finance_report_effect: no-show revenue, cancellation/no-show KPIs (M32 SF32.1.5).
i18n_a11y: none.
acceptance: A guaranteed reservation marked no-show is charged once; a later reinstatement reverses the fee via folio reversal and restores stock if available.
dependency: M08 F08.6.
```

```yaml
id: M05.F05.3.SF05.3.4
name: Walk (relocate) a confirmed guest
phase: 2
release: R1
actors: [front_office_manager, duty_manager]
screens: [SCR-FD-walk-planner]
inputs: [reservation_id, partner_hotel, nights_walked, transport, compensation, guest_consent]
states: [planned, executed, returned]
api: POST /v1/properties/{pid}/reservations/{rid}/walk (idem)
events: [GuestWalked]
data: [walk_record, reservation]
rules:
  - Walk candidates proposed by rule (not VIP, not accessibility need, not repeat guest priority) and approved by duty_manager.
  - Cost of relocation (partner room, transport) recorded as expense; guest charges handled per policy (typically no charge for walked nights).
security: duty_manager approval.
failure_cases: [no_partner_hotel_available]
finance_report_effect: walk cost to rooms department expense; walked nights excluded from revenue.
i18n_a11y: guest letter bilingual.
acceptance: An exposure of 1 on tonight produces a walk plan; executing it reduces sold by 1 for walked nights and records the relocation cost.
dependency: partner hotel agreements (manual, D-119).
```

```yaml
id: M05.F05.3.SF05.3.5
name: Waitlist
phase: 2
release: R1
actors: [front_desk_agent, guest]
screens: [SCR-FD-waitlist]
inputs: [dates, room_type_prefs, priority, contact, expiry]
states: [waiting, offered, accepted, expired, removed]
api: POST /v1/properties/{pid}/waitlist; POST .../waitlist/{id}/offer
events: [WaitlistOfferMade]
data: [waitlist_entry, inventory_hold]
rules:
  - Waitlist consumes no stock; on availability, an offer creates a time-limited hold for the top-priority entry.
security: none.
failure_cases: [offer_not_answered]
finance_report_effect: denied/regret demand for M53.
i18n_a11y: offer message bilingual.
acceptance: When a cancellation frees a room, the first waitlisted guest receives an offer with a 2-hour hold and the room is not visible to others during the hold.
dependency: none.
```

```yaml
id: M05.F05.3.SF05.3.6
name: Reinstate cancelled or no-show reservation
phase: 2
release: R1
actors: [front_office_manager]
screens: [SCR-FD-reservation]
inputs: [reservation_id, reason]
states: [confirmed]
api: POST /v1/properties/{pid}/reservations/{rid}/reinstate (idem)
events: [ReservationReinstated]
data: [reservation, inventory_movement]
rules:
  - Reinstatement re-consumes stock subject to availability and re-quotes if the original quote's policy is still valid; fees previously posted are reversed, not deleted.
security: front_office_manager.
failure_cases: [no_availability]
finance_report_effect: reversal lines.
i18n_a11y: none.
acceptance: Reinstating a cancelled reservation on a sold-out date fails unless overbooking approval is obtained.
dependency: none.
```

## F05.4 Arrival, check-in, registration and ID policy

```yaml
id: M05.F05.4.SF05.4.1
name: Arrival readiness checks
phase: 2
release: R1
actors: [front_desk_agent, front_office_manager]
screens: [SCR-FD-arrivals]
inputs: [business_date]
states: [ready, blocked_room, blocked_payment, blocked_id, blocked_registration]
api: GET /v1/properties/{pid}/arrivals?date=
events: none
data: [reservation, room_assignment, hk_room_status, guarantee, registration_record]
rules:
  - Each arrival shows readiness chips (room clean/inspected, guarantee, pre-registration/ID, deposit, special requests) and the single next action.
security: none.
failure_cases: [stale_hk_status]
finance_report_effect: none.
i18n_a11y: chips have text labels.
acceptance: The arrivals list shows for each guest whether identity, room readiness, payment and registration are complete, answering the Section O front-desk question.
dependency: M06, M41.
```

```yaml
id: M05.F05.4.SF05.4.2
name: Check-in
phase: 2
release: R1
actors: [front_desk_agent, guest]
screens: [SCR-FD-check-in, SCR-GST-online-check-in]
inputs: [reservation_room_id, room_id, registration_record_id, preauth_ref, key_count, idempotency_key]
states: [in_house]
api: POST /v1/properties/{pid}/reservation-rooms/{rrid}/check-in (idem)
events: [GuestCheckedIn]
data: [stay, room_assignment, registration_record, folio]
rules:
  - Preconditions are business date equals arrival (or early check-in approved), room vacant clean or inspected, registration complete per jurisdiction rule, guarantee/preauth valid.
  - Creates stay, opens folio windows per routing (M08), changes room to occupied, emits entitlement events.
  - Offline check-in allowed on on-prem LAN; payment preauth queued with risk flag if PSP unreachable.
security: front desk scope.
failure_cases: [room_not_ready, preauth_declined, psp_unreachable, id_mismatch]
finance_report_effect: occupancy starts; room charges scheduled.
i18n_a11y: guided flow; registration card bilingual.
acceptance: Check-in into a dirty room is blocked with the room-ready ETA shown; check-in with PSP unreachable proceeds only with a manager-approved alternative guarantee recorded.
dependency: M28, M41, M06.
```

```yaml
id: M05.F05.4.SF05.4.3
name: Registration record and ID policy by jurisdiction
phase: 2
release: R1
actors: [front_desk_agent, guest, compliance_officer]
screens: [SCR-FD-registration-card, SCR-GST-online-check-in]
inputs: [occupant_ids, nationality, document_type, document_number, document_expiry, ocr_result_ref, signature_ref, purpose_consents]
states: [draft, completed, verified, rejected]
api: POST /v1/properties/{pid}/stays/{sid}/registration (idem)
events: [RegistrationCompleted]
data: [registration_record, consent_record]
rules:
  - Required fields per occupant come from M44 guest-registration rule pack for the property's jurisdiction; unverified rule packs use a conservative default and label it.
  - ID imagery handled only by M41 (OCR, encryption, timed deletion); registration stores extracted confirmed fields and evidence ref.
  - Guest confirms each OCR field; mismatches go to review; non-biometric manual path always available.
security: sensitive fields encrypted and masked; access audited.
failure_cases: [ocr_wrong_field, expired_document, minor_without_guardian]
finance_report_effect: none.
i18n_a11y: registration card bilingual; e-sign accessible alternative.
acceptance: For the Oman fixture property the required fields differ from the Portugal fixture per their rule packs; a guest correcting an OCR error is recorded with before/after.
dependency: M41 (Phase 2–3), M44 rule packs (`unverified-assumption` per market).
```

```yaml
id: M05.F05.4.SF05.4.4
name: Early check-in and day use
phase: 2
release: R1
actors: [front_desk_agent, guest]
screens: [SCR-FD-check-in]
inputs: [reservation_room_id, early_time, fee_quote_id]
states: [approved, declined]
api: POST /v1/properties/{pid}/reservation-rooms/{rrid}/early-check-in (idem)
events: [EarlyCheckInGranted]
data: [reservation_room, room_type_inventory_night]
rules:
  - Early check-in before a configured hour consumes the previous night's stock (treated as extra night) unless the room is vacant and clean and policy allows fee-based early entry.
  - Day use (arrive and depart same day) uses day-use rate and does not count as a room night in occupancy (per KPI dictionary).
security: none.
failure_cases: [previous_night_sold_out]
finance_report_effect: early check-in fee to M54 ancillary code; day-use revenue separate.
i18n_a11y: none.
acceptance: A 05:00 arrival for a room whose previous night is sold out is refused early entry and offered room-ready ETA.
dependency: M54.
```

```yaml
id: M05.F05.4.SF05.4.5
name: Keys and device entitlement events
phase: 2
release: R1
actors: [front_desk_agent, system]
screens: [SCR-FD-check-in, SCR-FD-keys]
inputs: [stay_id, room_id, valid_from, valid_to, entitlement_types (key/parking/wifi/tv/phone/club)]
states: [issued, updated, revoked]
api: POST /v1/properties/{pid}/stays/{sid}/entitlements (idem)
events: [DeviceEntitlementGranted, DeviceEntitlementChanged, DeviceEntitlementRevoked]
data: [device_entitlement]
rules:
  - Entitlements carry stay window and are updated on room move, extension and checkout; consumers (M17 parking, M34–M36, lock adapters) subscribe.
  - Without a lock adapter, keys are encoded manually and count recorded; lost keys logged.
security: entitlement payload minimal (room, window, guest pseudonym).
failure_cases: [consumer_offline, key_encoder_unavailable]
finance_report_effect: entitlements for chargeable services link to folio.
i18n_a11y: none.
acceptance: A room move emits revoke for the old room and grant for the new room in order; the parking permit (M17) remains valid with updated room reference.
dependency: lock/parking/UC/HSIA/IPTV adapters (M17 Phase 3; M34–M36 Phase 7, Later).
```

```yaml
id: M05.F05.4.SF05.4.6
name: Guest registration reporting to authorities
phase: 3
release: R1
actors: [front_office_manager, compliance_officer, reporting_worker]
screens: [SCR-FD-registration-reporting]
inputs: [business_date, registrations, rule_pack_id]
states: [not_required, pending, exported_manual, submitted, acknowledged, failed]
api: POST /v1/properties/{pid}/registration-reports (idem)
events: [RegistrationReportExported, RegistrationReportAcknowledged]
data: [registration_record, government_submission_ref]
rules:
  - Where a market requires guest reporting, M44 rule pack defines content, deadline and route; only verified routes (M38 SF38.3.x) submit electronically, otherwise a manual export with staff attestation.
  - Never marked submitted without a receipt.
security: minimum data; access audited.
failure_cases: [route_unverified, deadline_missed]
finance_report_effect: none.
i18n_a11y: none.
acceptance: In a fixture market with unverified route, the system produces a manual export and the status reads exported_manual, never submitted.
dependency: M38 F38.3, M44 (`blocked` until verified per market).
```

## F05.5 In-stay changes and departure

```yaml
id: M05.F05.5.SF05.5.1
name: Room move
phase: 2
release: R1
actors: [front_desk_agent, guest, housekeeping_supervisor]
screens: [SCR-FD-room-move]
inputs: [stay_id, from_room, to_room, effective_time, reason, rate_change_flag]
states: [requested, executed]
api: POST /v1/properties/{pid}/stays/{sid}/room-moves (idem)
events: [GuestRoomMoved]
data: [room_move, room_assignment, device_entitlement, hk_task]
rules:
  - Target room must be vacant clean/inspected and free for remaining nights; if type changes, SF03.5.3 transfer applies.
  - Old room becomes vacant dirty with a cleaning task; folio unchanged unless rate change accepted.
security: none.
failure_cases: [target_not_free_for_all_nights, entitlement_consumer_lag]
finance_report_effect: none unless rate change.
i18n_a11y: none.
acceptance: After a room move, the old room shows vacant dirty with a new HK task, the new room occupied, and charges continue on the same folio.
dependency: M06.
```

```yaml
id: M05.F05.5.SF05.5.2
name: Stay extension and early departure
phase: 2
release: R1
actors: [front_desk_agent, guest]
screens: [SCR-FD-reservation-amend, SCR-GST-manage-booking]
inputs: [stay_id, new_departure, amendment_quote_id]
states: [extended, shortened]
api: POST /v1/properties/{pid}/stays/{sid}/change-departure (idem)
events: [StayExtended, StayShortened]
data: [reservation_room, inventory_movement]
rules:
  - Extension checks availability for the same room first, then same type with room move, else offer alternatives.
  - Early departure releases remaining nights; early departure fee per snapshot if any.
security: none.
failure_cases: [same_room_unavailable]
finance_report_effect: revenue schedule adjusted; early departure fee.
i18n_a11y: none.
acceptance: Extending by one night when the same room is sold to an arriving guest offers another room of the same type with a move plan.
dependency: M04 SF04.5.5.
```

```yaml
id: M05.F05.5.SF05.5.3
name: Checkout
phase: 2
release: R1
actors: [front_desk_agent, cashier, guest]
screens: [SCR-FD-check-out, SCR-GST-express-checkout]
inputs: [stay_id, settlement_instructions, invoice_details, idempotency_key]
states: [checked_out]
api: POST /v1/properties/{pid}/stays/{sid}/check-out (idem)
events: [GuestCheckedOut]
data: [stay, folio, fiscal_document, device_entitlement]
rules:
  - All folio windows must have zero balance or be transferred to AR (approved direct bill) before checkout; pending minibar/late charges check prompted.
  - Issues invoice/receipt per M08; revokes entitlements; room becomes vacant dirty.
  - Loyalty/referral qualification events emitted (StayCompleted) with payment status.
security: none.
failure_cases: [balance_outstanding, card_capture_failed]
finance_report_effect: revenue recognized per posted nights; AR transfer.
i18n_a11y: invoice in chosen language; accessible receipt email.
acceptance: Checkout with a non-zero balance and no approved direct-bill is refused; after capture the invoice is issued once and entitlements revoked.
dependency: M08, M28.
```

```yaml
id: M05.F05.5.SF05.5.4
name: Express and late checkout
phase: 2
release: R1
actors: [guest, front_desk_agent]
screens: [SCR-GST-express-checkout, SCR-FD-departures]
inputs: [stay_id, requested_time, fee_quote_id, card_on_file_consent]
states: [requested, approved, declined, completed]
api: POST /v1/properties/{pid}/stays/{sid}/late-checkout (idem); POST .../stays/{sid}/express-checkout (idem)
events: [LateCheckoutGranted, ExpressCheckoutCompleted]
data: [stay, reservation_room]
rules:
  - Late checkout bounded by next arrival's room-ready requirement (M06 SF06.2.5); fee via M54.
  - Express checkout captures against preauth only with folio reviewed by guest (or emailed) and consent.
security: guest verified via app/OTP.
failure_cases: [next_guest_needs_room, preauth_insufficient]
finance_report_effect: late checkout fee revenue.
i18n_a11y: none.
acceptance: A late checkout request conflicting with an early-arrival VIP assigned to the room is declined or triggers a room reassignment proposal.
dependency: M54, M06.
```

```yaml
id: M05.F05.5.SF05.5.5
name: Post-departure (late) charges
phase: 2
release: R1
actors: [cashier, fnb_manager]
screens: [SCR-FD-folio]
inputs: [stay_id, charge, evidence, window]
states: [posted, disputed]
api: POST /v1/properties/{pid}/folios/{fid}/late-charges (idem)
events: [LateChargePosted]
data: [folio_line]
rules:
  - Late charges within configured days post to a reopened window and charge the card on file only if consent covers it; guest notified with evidence.
  - Otherwise transferred to AR/collection.
security: approval above threshold.
failure_cases: [no_card_consent, dispute]
finance_report_effect: revenue on current business date.
i18n_a11y: notification bilingual.
acceptance: A minibar charge found after checkout posts once with photo evidence and the guest receives a revised invoice or credit note chain.
dependency: M08, M56.
```

```yaml
id: M05.F05.5.SF05.5.6
name: Stay state machine and entitlement revocation
phase: 2
release: R1
actors: [system]
screens: [SCR-FD-reservation]
inputs: [stay_id, transition]
states: [due_in, in_house, due_out, checked_out, reopened]
api: none (derived; transitions via commands above)
events: [StayStateChanged]
data: [stay]
rules:
  - Transitions only via defined commands; reopen checkout allowed on same business date for correction, with audit.
  - Checkout or cancellation revokes every device_entitlement exactly once.
security: reopen requires front_office_manager.
failure_cases: [illegal_transition]
finance_report_effect: none.
i18n_a11y: none.
acceptance: Every illegal transition returns 409 with allowed transitions listed; entitlement revocation events are unique per entitlement.
dependency: docs/02 SM-reservation.
```

```yaml
id: M05.F05.5.SF05.5.7
name: Reservation history and audit
phase: 2
release: R1
actors: [front_office_manager, auditor]
screens: [SCR-FD-reservation-history]
inputs: [reservation_id]
states: [none]
api: GET /v1/properties/{pid}/reservations/{rid}/history
events: none
data: [reservation_history, audit_log]
rules:
  - Every change stored as versioned diff with actor, channel, correlation id and reason.
security: masked fields per policy.
failure_cases: [history_gap]
finance_report_effect: supports dispute and M60 duplicate detection.
i18n_a11y: timeline accessible.
acceptance: The history shows creation, each amendment and cancellation with actor and source for a channel-originated reservation.
dependency: none.
```

### M05 key invariants
1. A reservation consumes stock only in tentative/confirmed/in_house states; waitlist consumes none.
2. Never a captured payment without a reservation, nor a confirmed reservation whose required guarantee is missing without an attention item.
3. Booker, occupant and payer are distinct roles; occupant sensitive data is never shown to a booker by default.
4. Check-in requires room readiness, registration per jurisdiction rule and guarantee; overrides are approved and audited.
5. Every entitlement granted is revoked exactly once at checkout/cancel.
6. Registration reports are never marked submitted without a receipt.

### M05 module-level acceptance
| Test | Maps to | Statement |
|---|---|---|
| AC-M05-1 | AT-G03.1 | Guests check in; room charge posts once per night; parking entitlement issued. |
| AC-M05-2 | AT-G13.2 | OCR autofill with guest correction, e-sign registration and OTP-bound booking confirmation after payment. |
| AC-M05-3 | AT-G20.5 | Corporate cancellation releases stock, assesses snapshot fee and reverses loyalty exactly once. |
| AC-M05-4 | AT-G07.1 | Refund/cancellation emits reversal events consumed once by points ledger. |
| AC-M05-5 | AT-G19.6 | Stay followed from booking through checkout with consented review invite. |

### M05 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-110 | Default cancellation/no-show policies and no-show stock release rule | Revenue Manager + GM | Free cancel until 24 h before arrival 14:00 property time; no-show = 1 night; release all remaining nights. |
| D-119 | Walk policy and partner hotel agreements | GM | Manual partner list; hotel pays first night elsewhere + transport. |
| D-125 | Guest registration/police reporting per market | Compliance Officer + counsel | Manual export path everywhere until a route is verified. |
| D-126 | Door-lock system for pilot hotel | IT Admin + Chief Engineer | Manual encoding in R1; lock adapter Later/optional. |

---

# M06 — Front desk/housekeeping

| Attribute | Value |
|---|---|
| Purpose | Give the front desk live arrivals, departures, in-house and guest-history views with a prioritized attention inbox, and give housekeeping a room-state model that keeps occupancy status separate from cleaning status, generates and assigns tasks, runs inspections, handles linen/minibar/lost-and-found intake, respects DND/privacy, estimates room-ready times, and works on mobile including offline with deterministic conflict resolution. |
| Build phase(s) | 2 (front desk boards, room/cleaning state, tasks, inspection, mobile offline); 3 minibar/linen integration with M56, service requests with M55. |
| Release flag | R1. |
| Bounded context | `frontoffice` (schema `frontoffice`); housekeeping sub-context `housekeeping`. |
| Systems of record owned | `hk_room_status`, `hk_task`, `hk_task_template`, `hk_inspection`, `dnd_status`, `offline_change_set`, `room_ready_estimate`, `fo_hk_discrepancy`, `shift_handover_note`, `guest_trace`, `minibar_count` (count evidence; posting via M08), `room_entry_log`. Linen custody ledger is M56; found-item custody is M43. |
| Upstream dependencies | M01 (devices, offline), M02 (scopes, incognito), M03 (rooms, OOO/OOS, assignment), M05 (stays, arrivals/departures). |
| Downstream dependents | M05 (readiness), M08 (minibar postings), M26 (defects), M43 (found items), M55 (service requests), M56 (linen/minibar stock), M32 (HK productivity KPIs), M62 (handover). |
| External dependencies | None mandatory. Optional room sensors/locks for occupancy detection — Later, `unverified-assumption`. |

**Room state model (normative).** Two independent dimensions per room:
- *Front-office occupancy* (derived from M05/M03): `vacant`, `occupied`, `due_out`, `due_in`, `ooo`, `oos`.
- *Housekeeping condition* (owned here): `dirty`, `cleaning_in_progress`, `clean`, `inspected`, `pickup` (light touch-up), `refused_service`.
A room is **ready for arrival** only when `vacant ∧ (inspected ∨ (clean ∧ inspection not required)) ∧ ¬ooo`. HK-reported occupancy (`hk_observed: vacant/occupied/sleep_out/skip`) is compared to front-office occupancy to produce discrepancies.

## F06.1 Front desk operations

```yaml
id: M06.F06.1.SF06.1.1
name: Arrivals board
phase: 2
release: R1
actors: [front_desk_agent, front_office_manager]
screens: [SCR-FD-arrivals]
inputs: [business_date, filters (vip/group/accessibility/unassigned/unpaid)]
states: [due_in, ready, blocked, checked_in]
api: GET /v1/properties/{pid}/arrivals?date=
events: none
data: [reservation, room_assignment, hk_room_status, guarantee, special_request]
rules:
  - Sorted by ETA then priority; each row shows readiness and next action (SF05.4.1).
  - Updates in real time from events (room ready, payment received).
security: incognito masking.
failure_cases: [realtime_feed_lag]
finance_report_effect: none.
i18n_a11y: live updates announced politely; mobile and desktop layouts.
acceptance: When housekeeping inspects a room assigned to an arrival, that arrival's row turns ready within 5 seconds without page reload.
dependency: M05.
```

```yaml
id: M06.F06.1.SF06.1.2
name: Departures board
phase: 2
release: R1
actors: [front_desk_agent, cashier]
screens: [SCR-FD-departures]
inputs: [business_date]
states: [due_out, checked_out, late_checkout, overdue]
api: GET /v1/properties/{pid}/departures?date=
events: [DepartureOverdue]
data: [stay, folio, hk_room_status]
rules:
  - Shows balance, pending charges (minibar count pending), late checkout approvals and overdue departures after checkout time.
security: balances visible only to cashier-capable roles.
failure_cases: [overdue_not_flagged]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A stay not checked out 30 minutes after its checkout time appears as overdue and alerts the front desk.
dependency: M08.
```

```yaml
id: M06.F06.1.SF06.1.3
name: In-house list and guest history lookup
phase: 2
release: R1
actors: [front_desk_agent, concierge, guest_relations]
screens: [SCR-FD-in-house, SCR-FD-guest-profile]
inputs: [search_term, room_number]
states: [none]
api: GET /v1/properties/{pid}/in-house; GET /v1/properties/{pid}/guests/{gid}/history
events: none
data: [stay, guest_profile, reservation]
rules:
  - Guest history shows prior stays, preferences with purpose tags, complaints/recovery links (M55) and consents; sensitive notes masked.
  - Switchboard lookups honor incognito.
security: field masking; history visible per role.
failure_cases: [incognito_leak]
finance_report_effect: none.
i18n_a11y: dual-script search.
acceptance: A lookup by surname for an incognito guest returns no result for a concierge role.
dependency: M02 SF02.3.4.
```

```yaml
id: M06.F06.1.SF06.1.4
name: Front office attention inbox
phase: 2
release: R1
actors: [front_office_manager, duty_manager]
screens: [SCR-FD-attention, SCR-GM-attention-inbox]
inputs: [property_id]
states: [open, owned, resolved, escalated]
api: GET /v1/properties/{pid}/attention-items; POST .../attention-items/{id}/assign
events: [AttentionItemRaised, AttentionItemResolved]
data: [attention_item]
rules:
  - Aggregates exposures (overbooking), unassigned VIPs, accessibility needs unmet, failed guarantees, discrepancies, overdue departures, room defects blocking arrivals.
  - Each item has owner, due time and action; answers "what needs attention, why, by when, who owns it".
security: role-scoped.
failure_cases: [duplicate_items]
finance_report_effect: none.
i18n_a11y: 3–7 top items on home screen (P1).
acceptance: An oversell exposure for tonight appears once with owner and deadline and disappears when the walk is executed.
dependency: M63 engine.
```

```yaml
id: M06.F06.1.SF06.1.5
name: Shift handover log
phase: 2
release: R1
actors: [front_desk_agent, night_auditor, housekeeping_supervisor]
screens: [SCR-FD-handover]
inputs: [shift_id, notes, open_items]
states: [draft, handed_over, acknowledged]
api: POST /v1/properties/{pid}/handovers; POST .../handovers/{id}/ack
events: [ShiftHandedOver]
data: [shift_handover_note, attention_item]
rules:
  - Handover lists open attention items automatically; incoming shift acknowledges.
security: none.
failure_cases: [unacknowledged_handover]
finance_report_effect: none.
i18n_a11y: bilingual.
acceptance: An unacknowledged handover after 30 minutes alerts the front_office_manager.
dependency: M62 SF62.2.1.
```

```yaml
id: M06.F06.1.SF06.1.6
name: Guest traces and messages
phase: 2
release: R1
actors: [front_desk_agent, concierge]
screens: [SCR-FD-traces]
inputs: [reservation_id, trace_text, due_at, department]
states: [open, done]
api: POST /v1/properties/{pid}/traces; POST .../traces/{id}/complete
events: [TraceDue]
data: [guest_trace]
rules:
  - Traces are internal reminders tied to a reservation and department with due time; overdue traces surface in attention inbox.
security: department scope.
failure_cases: [trace_on_cancelled_reservation]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A trace due at 18:00 appears on the concierge list at 18:00 and escalates at 18:30 if open.
dependency: none.
```

## F06.2 Room condition, cleaning tasks, inspection and readiness

```yaml
id: M06.F06.2.SF06.2.1
name: Room condition vs occupancy and discrepancy
phase: 2
release: R1
actors: [housekeeper, housekeeping_supervisor, front_office_manager]
screens: [SCR-HK-room-board, SCR-FD-discrepancies]
inputs: [room_id, hk_condition, hk_observed_occupancy]
states: [dirty, cleaning_in_progress, clean, inspected, pickup, refused_service]
api: PUT /v1/properties/{pid}/rooms/{room_id}/hk-status (idem); GET /v1/properties/{pid}/discrepancies
events: [RoomConditionChanged, RoomDiscrepancyDetected]
data: [hk_room_status, fo_hk_discrepancy]
rules:
  - Condition changes never alter occupancy; checkout sets condition dirty automatically.
  - HK observation "vacant" on an occupied room (skip) or "occupied" on a vacant room (sleep) raises a discrepancy for front desk resolution.
security: housekeeper updates assigned rooms only.
failure_cases: [status_update_on_ooo_room, stale_board]
finance_report_effect: skips may indicate revenue risk (M60).
i18n_a11y: status by text and icon, not color only.
acceptance: A housekeeper reporting a vacant observation for an in-house room creates one discrepancy item; resolving it records outcome (guest left early / error).
dependency: none.
```

```yaml
id: M06.F06.2.SF06.2.2
name: Task generation and priorities
phase: 2
release: R1
actors: [housekeeping_supervisor, hk_task_worker]
screens: [SCR-HK-task-board]
inputs: [business_date, templates (departure/stayover/turndown/deep_clean/pickup), priority_rules]
states: [pending, assigned, in_progress, done, inspected, failed, cancelled]
api: POST /v1/properties/{pid}/hk-tasks/generate (idem); GET .../hk-tasks
events: [HkTaskCreated]
data: [hk_task, hk_task_template]
rules:
  - Tasks generated at business-date start and on events (checkout, room move, early arrival).
  - Priority = early arrivals/VIP/accessibility first, then due-ins, then departures without arrival, then stayovers; DND respected.
  - Task credits/durations from templates for workload.
security: supervisor edits priorities.
failure_cases: [duplicate_generation]
finance_report_effect: labor productivity inputs (M32/M62).
i18n_a11y: none.
acceptance: Regenerating tasks for the same day creates no duplicates; a checkout at 09:00 creates a departure task within 10 seconds.
dependency: M56 SF56.1.1 extends.
```

```yaml
id: M06.F06.2.SF06.2.3
name: Assignment and workload balancing
phase: 2
release: R1
actors: [housekeeping_supervisor]
screens: [SCR-HK-assignment]
inputs: [housekeepers_on_shift, sections, credits_per_attendant]
states: [draft, published]
api: POST /v1/properties/{pid}/hk-assignments (idem)
events: [HkAssignmentPublished]
data: [hk_task]
rules:
  - Balances credits across attendants within sections; minimizes walking between floors.
  - Reassignment of an in-progress task requires supervisor and notifies both attendants.
security: supervisor.
failure_cases: [attendant_absent]
finance_report_effect: none.
i18n_a11y: mobile push bilingual.
acceptance: Publishing assignments pushes each attendant only their own rooms; an absent attendant's tasks can be redistributed in one action.
dependency: M27 roster (Phase 4) optional.
```

```yaml
id: M06.F06.2.SF06.2.4
name: Inspection and reclean
phase: 2
release: R1
actors: [housekeeping_supervisor, housekeeper]
screens: [SCR-HK-inspection]
inputs: [room_id, checklist_results, photos, fail_reasons]
states: [passed, failed]
api: POST /v1/properties/{pid}/rooms/{room_id}/inspections (idem)
events: [RoomInspected, RoomInspectionFailed]
data: [hk_inspection, hk_task]
rules:
  - Fail creates a reclean task to the original attendant with reasons; defects create M26 work orders.
  - Inspection required for VIP/accessibility/after OOO; optional otherwise per policy.
security: inspector cannot inspect own cleaning if policy requires independence.
failure_cases: [inspection_photo_upload_offline]
finance_report_effect: quality metrics.
i18n_a11y: checklist accessible; photo optional alternative text.
acceptance: A failed inspection returns the room to dirty with a reclean task and the arrival stays blocked.
dependency: M26, M61.
```

```yaml
id: M06.F06.2.SF06.2.5
name: Room-ready estimate
phase: 2
release: R1
actors: [front_desk_agent, guest, system]
screens: [SCR-FD-arrivals, SCR-GST-stay]
inputs: [room_id, queue_position, task_durations, attendant_progress]
states: [estimated, ready, delayed]
api: GET /v1/properties/{pid}/rooms/{room_id}/ready-estimate
events: [RoomReadyEstimateChanged, RoomReady]
data: [room_ready_estimate]
rules:
  - Estimate from attendant's queue and historical durations; confidence shown; labeled estimate.
  - Guest notified when ready if consented for service messages.
security: none.
failure_cases: [estimate_wrong_due_to_dnd]
finance_report_effect: turnaround KPI.
i18n_a11y: estimate shown as time and relative minutes.
acceptance: Estimates for a day of tasks have median absolute error recorded and displayed; a room marked inspected triggers RoomReady within 5 seconds.
dependency: none.
```

```yaml
id: M06.F06.2.SF06.2.6
name: DND, privacy and service refusal
phase: 2
release: R1
actors: [guest, housekeeper, housekeeping_supervisor, duty_manager]
screens: [SCR-HK-room-board, SCR-GST-stay]
inputs: [room_id, dnd_on, source (guest app/phone/door indicator), time]
states: [dnd_active, dnd_cleared, welfare_check_due]
api: PUT /v1/properties/{pid}/rooms/{room_id}/dnd
events: [DndChanged, WelfareCheckDue]
data: [dnd_status, room_entry_log]
rules:
  - DND defers service tasks; after configured duration (e.g. 24 h) a welfare check is escalated to duty_manager, not skipped silently.
  - Every staff room entry logged (who, when, reason) for guest privacy.
security: entry log visible to managers only.
failure_cases: [dnd_unknown_to_hk]
finance_report_effect: none.
i18n_a11y: guest app DND toggle accessible.
acceptance: A DND lasting beyond the threshold raises a welfare check to duty_manager with the entry log.
dependency: M42 guest welfare link.
```

## F06.3 Mobile and offline housekeeping

```yaml
id: M06.F06.3.SF06.3.1
name: Housekeeping mobile task flow
phase: 2
release: R1
actors: [housekeeper, housekeeping_supervisor]
screens: [SCR-HK-my-rooms, SCR-HK-task-detail]
inputs: [task_id, start, finish, notes, photos]
states: [assigned, in_progress, done]
api: POST /v1/properties/{pid}/hk-tasks/{tid}/start; POST .../hk-tasks/{tid}/finish (idem)
events: [HkTaskStarted, HkTaskCompleted]
data: [hk_task, hk_room_status]
rules:
  - One-tap start/finish; finishing sets condition clean (or inspected if attendant is certified to self-inspect).
  - Works on low-end Android devices; large targets; minimal text entry.
security: enrolled device; attendant sees only room number, task and permitted guest notes.
failure_cases: [device_offline]
finance_report_effect: none.
i18n_a11y: bilingual; icon plus text; screen reader support.
acceptance: A housekeeper completes a task in two taps; board updates in real time.
dependency: M01 SF01.6.4.
```

```yaml
id: M06.F06.3.SF06.3.2
name: Offline change capture
phase: 2
release: R1
actors: [housekeeper]
screens: [SCR-HK-my-rooms, SCR-HK-sync-status]
inputs: [change_id, entity, field, new_value, base_version, device_time, device_seq, device_id]
states: [queued, synced, conflicted]
api: POST /v1/properties/{pid}/sync/hk (idem per change_id)
events: [OfflineChangesSynced]
data: [offline_change_set]
rules:
  - Device stores last-known assignments and room states; each offline change carries entity base_version and device monotonic sequence.
  - Offline mode clearly indicated; changes that require server authority (minibar charge posting, lost item release) are captured as requests, never as final.
security: offline store encrypted; purge on logout/revoke.
failure_cases: [device_clock_wrong, app_killed_before_sync]
finance_report_effect: none directly.
i18n_a11y: offline banner accessible.
acceptance: A device offline for 2 hours syncs 30 changes in order; server-authority actions appear as pending requests.
dependency: M01 SF01.4.4.
```

```yaml
id: M06.F06.3.SF06.3.3
name: Offline conflict resolution rules
phase: 2
release: R1
actors: [system, housekeeping_supervisor]
screens: [SCR-HK-sync-conflicts]
inputs: [offline_change, server_state]
states: [auto_resolved, needs_review, rejected]
api: GET /v1/properties/{pid}/sync/conflicts; POST .../sync/conflicts/{id}/resolve
events: [SyncConflictDetected, SyncConflictResolved]
data: [offline_change_set, hk_room_status]
rules:
  - Precedence (normative, D-114) 1) front-office facts win over HK (checkout, room move, OOO) — an offline "clean" on a room that became OOO is rejected; 2) later inspection outranks earlier clean; 3) same-field edits from two devices resolved by server receive order only when neither is an inspection, else needs_review; 4) task completion on a reassigned task is accepted as work evidence but credited to the performing attendant and flagged.
  - Never regress inspected to clean from a stale device without review.
security: supervisor resolves reviews.
failure_cases: [conflict_storm_after_outage]
finance_report_effect: none.
i18n_a11y: conflict explanation in plain language.
acceptance: A room inspected online at 10:05 and marked clean offline at 10:00 (synced 10:20) remains inspected and the offline change is logged as superseded; an offline clean on a room set OOO at 09:50 is rejected with reason.
dependency: none.
```

```yaml
id: M06.F06.3.SF06.3.4
name: Sync audit and review queue
phase: 2
release: R1
actors: [housekeeping_supervisor, auditor]
screens: [SCR-HK-sync-conflicts, SCR-OPS-audit-search]
inputs: [date_range, device_id]
states: [open, closed]
api: GET /v1/properties/{pid}/sync/audit
events: none
data: [offline_change_set, audit_log]
rules:
  - Every offline change stored with device time, server receive time and resolution outcome.
security: none.
failure_cases: none
finance_report_effect: none.
i18n_a11y: none.
acceptance: Audit for a device lists every offline change with both timestamps and resolution.
dependency: none.
```

```yaml
id: M06.F06.3.SF06.3.5
name: Front office vs housekeeping discrepancy report
phase: 2
release: R1
actors: [front_office_manager, night_auditor]
screens: [SCR-FD-discrepancies, SCR-FIN-night-audit]
inputs: [business_date]
states: [open, resolved]
api: GET /v1/properties/{pid}/discrepancies?date=
events: [RoomDiscrepancyDetected]
data: [fo_hk_discrepancy]
rules:
  - Unresolved sleep/skip discrepancies block night audit completion unless acknowledged with reason.
security: none.
failure_cases: [unresolved_at_audit]
finance_report_effect: revenue protection evidence (M60 SF60.1.4).
i18n_a11y: none.
acceptance: Night audit refuses to complete with an unresolved skip discrepancy until acknowledged.
dependency: M08 F08.6.
```

## F06.4 In-room items, requests and defects

```yaml
id: M06.F06.4.SF06.4.1
name: Room linen par check
phase: 3
release: R1
actors: [housekeeper, housekeeping_supervisor]
screens: [SCR-HK-task-detail]
inputs: [room_id, linen_items_counted, soiled_removed, damaged]
states: [ok, shortage, damage_reported]
api: POST /v1/properties/{pid}/rooms/{room_id}/linen-counts (idem)
events: [RoomLinenCounted, LinenDamageReported]
data: [linen_room_count (M56 ledger ref)]
rules:
  - Room-level counts feed M56 custody ledger; this module captures, M56 owns balances.
security: none.
failure_cases: [count_without_task]
finance_report_effect: linen loss/damage cost via M56.
i18n_a11y: stepper controls accessible.
acceptance: Recording 2 damaged towels in a room creates one M56 damage movement with room and attendant reference.
dependency: M56 (Phase 3–4).
```

```yaml
id: M06.F06.4.SF06.4.2
name: Minibar count and one-time posting request
phase: 3
release: R1
actors: [housekeeper, cashier]
screens: [SCR-HK-minibar, SCR-FD-folio]
inputs: [room_id, stay_id, items_consumed, count_time, idempotency_key]
states: [counted, posting_requested, posted, disputed, voided]
api: POST /v1/properties/{pid}/rooms/{room_id}/minibar-counts (idem)
events: [MinibarConsumptionCounted, MinibarChargePosted]
data: [minibar_count, minibar_posting_request, folio_line]
rules:
  - Each count references the previous restock baseline; posting request keyed by (room, count_id) so offline resubmits post once.
  - Counts after checkout route to late charges (SF05.5.5); disputes reverse via M08.
  - Restock decrements M56 minibar stock.
security: attendant cannot post price; price from outlet menu.
failure_cases: [duplicate_count_sync, count_after_room_move]
finance_report_effect: minibar revenue to F&B outlet; COGS via M56/M14.
i18n_a11y: item images with text labels.
acceptance: The same minibar count synced twice from an offline device posts one folio charge.
dependency: M56, M08.
```

```yaml
id: M06.F06.4.SF06.4.3
name: Amenity and guest service requests routing
phase: 2
release: R1
actors: [guest, front_desk_agent, housekeeper]
screens: [SCR-GST-requests, SCR-HK-task-board]
inputs: [stay_id, request_type, quantity, note]
states: [requested, accepted, delivered, cancelled]
api: POST /v1/properties/{pid}/stays/{sid}/service-requests
events: [ServiceRequestCreated, ServiceRequestCompleted]
data: [hk_task, service_request (M55 owned)]
rules:
  - Housekeeping-type requests (towels, pillows, cot) become HK tasks with SLA; chargeable items post via M08 once on delivery.
security: guest sees own requests only.
failure_cases: [sla_breach]
finance_report_effect: chargeable amenities revenue.
i18n_a11y: guest app accessible.
acceptance: A guest's extra-pillow request becomes an HK task and the guest sees status delivered after completion.
dependency: M55.
```

```yaml
id: M06.F06.4.SF06.4.4
name: Found item intake from rooms
phase: 2
release: R1
actors: [housekeeper]
screens: [SCR-HK-found-item]
inputs: [room_id, description, photo, found_at, condition]
states: [logged]
api: POST /v1/properties/{pid}/found-items (idem)
events: [FoundItemLogged]
data: [found_item_ref (M43 owned)]
rules:
  - Captured in one screen from the room task; custody and matching are M43; attendant cannot view claimant data.
security: photo stored restricted.
failure_cases: [offline_photo]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A found item logged offline creates exactly one M43 item with room and stay reference after sync.
dependency: M43.
```

```yaml
id: M06.F06.4.SF06.4.5
name: Room defect reporting and sale impact
phase: 2
release: R1
actors: [housekeeper, housekeeping_supervisor, engineer]
screens: [SCR-HK-defect, SCR-ENG-work-order]
inputs: [room_id, defect_category, severity, photo, affects_sale]
states: [reported, work_order_created, resolved]
api: POST /v1/properties/{pid}/rooms/{room_id}/defects (idem)
events: [RoomDefectReported]
data: [room_defect, room_status_block]
rules:
  - Severity critical → OOO request (SF03.3.1) proposal to front_office_manager; minor → OOS or work order only.
  - Guest-reported defects link to M55 recovery case.
security: none.
failure_cases: [defect_in_occupied_room]
finance_report_effect: maintenance cost attributed to room (M26).
i18n_a11y: none.
acceptance: A critical leak reported in a vacant room proposes OOO and blocks it from arrivals assignment immediately pending confirmation.
dependency: M26 (Phase 3–4); simple work order Phase 2.
```

```yaml
id: M06.F06.4.SF06.4.6
name: Staff room entry log
phase: 2
release: R1
actors: [housekeeper, engineer, security_officer, duty_manager]
screens: [SCR-HK-task-detail, SCR-OPS-audit-search]
inputs: [room_id, entry_time, reason, task_ref]
states: [recorded]
api: POST /v1/properties/{pid}/rooms/{room_id}/entries
events: none
data: [room_entry_log]
rules:
  - Entries tied to tasks recorded automatically at task start; ad-hoc entries require reason.
security: log visible to managers and security only.
failure_cases: [entry_without_reason]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A guest complaint about entry can be answered from the log showing staff, time and task.
dependency: none.
```

### M06 key invariants
1. Housekeeping condition never changes occupancy, and occupancy never marks a room clean.
2. A room is offered for check-in only if vacant, not OOO and clean/inspected per policy.
3. Offline changes never overwrite front-office facts or later inspections; conflicts are explicit.
4. Minibar and chargeable amenity postings are idempotent by count/request id — posted once.
5. DND never silently suppresses welfare escalation.
6. Every staff room entry is logged.

### M06 module-level acceptance
| Test | Maps to | Statement |
|---|---|---|
| AC-M06-1 | AT-G19.7 | Linen/housekeeping journey for the traced stay; room-ready estimate and inspection gate arrival. |
| AC-M06-2 | AT-G20.6 | Network outage: housekeeping offline changes sync with deterministic conflict resolution and no duplicate minibar charge. |
| AC-M06-3 | AT-G14.1 | Found item logged from room creates one M43 custody record. |
| AC-M06-4 | AT-G08.3 | Night audit blocked by unresolved FO/HK discrepancy until acknowledged. |

### M06 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-114 | Offline conflict precedence (normative list in SF06.3.3) confirmation | Housekeeping Supervisor + Front Office Manager | As specified in SF06.3.3. |
| D-127 | Inspection mandatory for all rooms or only VIP/accessibility/after OOO; self-inspection certification | Housekeeping Supervisor | Mandatory for VIP/accessibility/after OOO; certified attendants may self-inspect others. |
| D-128 | DND welfare-check threshold | GM + Security | 24 h continuous DND or 2 consecutive days without service. |

---

# M07 — Distribution

| Attribute | Value |
|---|---|
| Purpose | Sell rooms directly through an accessible, low-bandwidth booking engine and indirectly through an approved channel manager: map rooms/rates, publish availability-rates-inventory (ARI) with retries and dead letters, ingest channel bookings/modifications/cancellations idempotently, reconcile PMS vs channel, expose oversell risk, and attribute every booking to its source with commission and funnel measurement. |
| Build phase(s) | 2 (direct booking engine, attribution, channel port + mock adapter, manual extranet path); 3 certified channel adapter, reconciliation, commission; 5 metasearch/campaign measurement with M51/M53. |
| Release flag | R1. |
| Bounded context | `distribution` (schema `distribution`). |
| Systems of record owned | `channel_connection`, `channel_capability`, `channel_mapping`, `ari_message`, `ari_ack`, `channel_booking_message`, `dead_letter` (distribution-scoped view of M01 dead letters), `channel_reconciliation_run`, `reconciliation_item`, `source_code`, `source_attribution`, `commission_rule`, `commission_accrual`, `funnel_event`, `booking_session`. |
| Upstream dependencies | M03 (availability, stop-sell, allotments), M04 (rates, restrictions, quotes), M05 (reservation commands), M28 (payments, virtual cards), M02 (consent for analytics), M51 (website shell/SEO, Phase 2–3). |
| Downstream dependents | M05 (channel bookings), M20/M60 (commission invoices, match), M32/M53 (channel mix, pickup, net contribution), M51/M52 (attribution, abandoned-quote recovery), M31 (referral attribution precedence). |
| External dependencies | Channel manager (e.g. SiteMinder or equivalent) — `unverified-assumption`; no partner is contracted or certified. Until a certified adapter exists the production path is **manual extranet** with the mock adapter for tests (release blocker for hotels requiring channels, per P5). OTA virtual card handling — `unverified-assumption`. |

## F07.1 Direct booking engine

```yaml
id: M07.F07.1.SF07.1.1
name: Direct search and availability
phase: 2
release: R1
actors: [guest, booker]
screens: [SCR-GST-search, SCR-GST-results]
inputs: [arrival, departure, rooms, adults, children_ages, accessibility_filters, promo_code, locale, currency]
states: [searching, results, no_availability]
api: GET /v1/public/properties/{slug}/offers?arrival=&departure=&adults=
events: [FunnelSearchPerformed]
data: [booking_session, funnel_event]
rules:
  - Uses M03 availability and M04 quote pipeline; only sellable room types shown; alternative dates offered on no availability.
  - Accessibility filters match structured room attributes.
security: bot rate-limiting without CAPTCHA puzzles (P1); no PII.
failure_cases: [pricing_timeout, invalid_dates]
finance_report_effect: funnel top-of-stage counts.
i18n_a11y: SSR pages under 200 KB initial for low bandwidth; WCAG 2.2 AA; en/ar RTL.
acceptance: On a throttled 3G profile the results page is interactive within 5 seconds and passes axe with zero critical issues.
dependency: M51 site shell.
```

```yaml
id: M07.F07.1.SF07.1.2
name: Offer display with honest total price
phase: 2
release: R1
actors: [guest]
screens: [SCR-GST-results, SCR-GST-room-detail]
inputs: [quote]
states: [none]
api: part of offers response
events: [FunnelOfferViewed]
data: [quote]
rules:
  - Each offer shows all-in total, per-night breakdown on demand, cancellation terms in plain words with deadline in property time, inclusions and approved media only (M39).
  - No fake scarcity messages; "only N left" shown only if true and configured.
security: none.
failure_cases: [media_unapproved]
finance_report_effect: none.
i18n_a11y: images have approved alt text; price breakdown accessible.
acceptance: A "2 rooms left" message is shown only when available equals 2 for every night of the stay.
dependency: M04 SF04.4.5, M39.
```

```yaml
id: M07.F07.1.SF07.1.3
name: Hold and checkout with payment
phase: 2
release: R1
actors: [guest]
screens: [SCR-GST-checkout]
inputs: [quote_id, guest_details, consents, payment_method, upsell_selections]
states: [hold_placed, payment_pending, confirmed, failed, expired]
api: POST /v1/public/properties/{slug}/holds (idem); POST /v1/public/properties/{slug}/bookings (idem)
events: [FunnelHoldPlaced, FunnelBookingCompleted]
data: [inventory_hold, booking_session]
rules:
  - Hold placed when guest starts checkout; payment via M28 hosted fields/redirect; confirmation via SF05.1.1.
  - Optional upsells (M54) priced in same quote; total re-confirmed.
security: PSP hosted fields (no PAN touches our servers); CSP headers.
failure_cases: [hold_expired, 3ds_abandoned, payment_declined]
finance_report_effect: direct-channel revenue; zero commission.
i18n_a11y: form errors identified and described; timing adjustable.
acceptance: A guest abandoning 3-D Secure has the hold released at expiry and no reservation created; a guest completing it gets one reservation and one confirmation.
dependency: M28 PSP (`unverified-assumption`).
```

```yaml
id: M07.F07.1.SF07.1.4
name: Confirmation and manage-booking
phase: 2
release: R1
actors: [guest, booker]
screens: [SCR-GST-confirmation, SCR-GST-manage-booking]
inputs: [confirmation_number, email, magic_link_token]
states: [viewing, amending, cancelling]
api: GET /v1/public/bookings/{token}; POST /v1/public/bookings/{token}/amend; POST /v1/public/bookings/{token}/cancel
events: [ManageBookingAccessed]
data: [reservation]
rules:
  - Access via signed, expiring link or guest account; confirmation number alone is not sufficient.
  - Amend/cancel use M05 commands and snapshot policies.
security: token binds reservation; rate limited.
failure_cases: [link_forwarded, token_expired]
finance_report_effect: none.
i18n_a11y: bilingual.
acceptance: Guessing confirmation numbers without a valid token never returns booking data.
dependency: M02 SF02.1.2.
```

```yaml
id: M07.F07.1.SF07.1.5
name: Booking engine resilience and low-bandwidth mode
phase: 2
release: R1
actors: [guest]
screens: [SCR-GST-search, SCR-GST-checkout]
inputs: [network_quality]
states: [normal, lite]
api: same endpoints
events: none
data: [none]
rules:
  - Lite mode serves text-first pages with deferred images; checkout works without client-side JS frameworks where PSP allows.
  - On PMS core unavailability, booking engine shows honest "try later / call" message, never accepts unconfirmable bookings.
security: none.
failure_cases: [core_outage]
finance_report_effect: none.
i18n_a11y: meets P1 low-bandwidth requirement.
acceptance: With core API down, the site shows contact options and zero bookings are accepted.
dependency: none.
```

```yaml
id: M07.F07.1.SF07.1.6
name: Abandoned quote capture and bot filtering
phase: 2
release: R1
actors: [marketing_manager, system]
screens: [SCR-MKT-funnel]
inputs: [booking_session, consent_state]
states: [abandoned, recovered, suppressed]
api: GET /v1/properties/{pid}/funnel/abandoned
events: [QuoteAbandoned]
data: [booking_session, funnel_event]
rules:
  - Abandonment recorded anonymously; recovery messages only if the visitor gave contact plus marketing consent (M52 SF52.1.3).
  - Bot/duplicate sessions filtered from funnel metrics.
security: none.
failure_cases: [recovery_without_consent]
finance_report_effect: conversion KPIs.
i18n_a11y: none.
acceptance: An abandoned checkout from a visitor without marketing consent produces no message and is counted in anonymous funnel stats only.
dependency: M52.
```

## F07.2 Channel connection and mapping

```yaml
id: M07.F07.2.SF07.2.1
name: Channel connection and capability registry
phase: 2
release: R1
actors: [integration_admin]
screens: [SCR-INT-channels]
inputs: [provider, credentials_secret_ref, hotel_code, capabilities, honesty_label, environment]
states: [draft, sandbox, certified_live, suspended, manual_only]
api: POST /v1/properties/{pid}/channel-connections; PATCH .../channel-connections/{id}
events: [ChannelConnectionChanged]
data: [channel_connection, channel_capability]
rules:
  - Capabilities flagged (ARI push, booking pull/push, modification, cancel, restrictions semantics, virtual cards); unsupported ones are unavailable, never faked.
  - Production live requires honesty_label certified or partner-contracted plus sandbox-tested and an activation gate pass.
security: credentials in vault; integration_admin only.
failure_cases: [credentials_invalid, label_insufficient]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A connection at unverified-assumption cannot move to certified_live; the UI shows manual_only with extranet instructions.
dependency: channel manager contract `unverified-assumption` (D-109).
```

```yaml
id: M07.F07.2.SF07.2.2
name: Room and rate mapping
phase: 2
release: R1
actors: [integration_admin, revenue_manager]
screens: [SCR-INT-channel-mapping]
inputs: [room_type_id, rate_plan_id, channel_room_code, channel_rate_code, occupancy_mapping, restriction_semantics]
states: [draft, validated, active, broken]
api: PUT /v1/properties/{pid}/channel-connections/{cid}/mappings (idem)
events: [ChannelMappingChanged]
data: [channel_mapping]
rules:
  - One PMS room/rate pair per channel code; validation pulls channel product list when capability exists.
  - Derived rates may be mapped; restriction semantic translation declared explicitly.
security: maker-checker for live mapping changes.
failure_cases: [duplicate_mapping, channel_code_missing]
finance_report_effect: none.
i18n_a11y: none.
acceptance: Mapping two PMS rate plans to the same channel rate code is rejected.
dependency: SF07.2.1.
```

```yaml
id: M07.F07.2.SF07.2.3
name: Unmapped and broken mapping detection
phase: 2
release: R1
actors: [integration_admin]
screens: [SCR-INT-channel-mapping]
inputs: [connection_id]
states: [ok, unmapped_found]
api: GET /v1/properties/{pid}/channel-connections/{cid}/mapping-health
events: [ChannelMappingBroken]
data: [channel_mapping]
rules:
  - Retired room types/rates with active mappings and sellable products without mappings are listed; incoming bookings for unmapped codes go to resolution queue (SF07.4.2).
security: none.
failure_cases: [none]
finance_report_effect: none.
i18n_a11y: none.
acceptance: Retiring a mapped rate plan raises a mapping-health warning before ARI for it stops.
dependency: none.
```

```yaml
id: M07.F07.2.SF07.2.4
name: Manual extranet fallback mode
phase: 2
release: R1
actors: [revenue_manager, front_office_manager]
screens: [SCR-INT-manual-channel-tasks]
inputs: [connection_id, change_events]
states: [task_open, done_attested]
api: GET /v1/properties/{pid}/manual-channel-tasks; POST .../manual-channel-tasks/{id}/attest
events: [ManualChannelTaskCreated]
data: [manual_channel_task]
rules:
  - When a connection is manual_only or down, ARI changes generate tasks ("close STD on OTA X for 12 Oct") and manually keyed bookings are entered with channel reference.
  - Staff attest completion; unattested closures count toward oversell exposure.
security: none.
failure_cases: [task_not_done]
finance_report_effect: none.
i18n_a11y: bilingual task text.
acceptance: Selling the last STD room directly while channel is manual creates a close-out task within 1 minute and exposure shows risk until attested.
dependency: none.
```

## F07.3 ARI publishing

```yaml
id: M07.F07.3.SF07.3.1
name: ARI delta generation
phase: 2
release: R1
actors: [ari_worker]
screens: [SCR-INT-ari-monitor]
inputs: [RoomTypeAvailabilityChanged, RatePricesPublished, RestrictionsChanged, StopSellChanged]
states: [pending, coalesced, sent]
api: none (event consumer)
events: [AriDeltaQueued]
data: [ari_message]
rules:
  - Consumes inventory/rate/restriction events and coalesces per (connection, room, rate, date) within a short window (e.g. 2 s) preserving the latest value.
  - Availability sent = min(M03 available for channel visibility, allotment for that channel), never exceeding real stock unless overbooking policy explicitly allows the channel.
security: none.
failure_cases: [event_burst]
finance_report_effect: none.
i18n_a11y: none.
acceptance: 100 rapid changes to the same date produce at most a few messages and the final channel value equals PMS availability.
dependency: M03, M04.
```

```yaml
id: M07.F07.3.SF07.3.2
name: ARI delivery with retries and rate limits
phase: 3
release: R1
actors: [ari_worker, integration_admin]
screens: [SCR-INT-ari-monitor]
inputs: [ari_message, adapter]
states: [sending, acknowledged, retrying, failed, dead_lettered]
api: adapter port ChannelPort.pushAri
events: [AriAcknowledged, AriDeliveryFailed]
data: [ari_message, ari_ack]
rules:
  - Exponential backoff with jitter; respects provider rate limits; per-connection ordering by sequence number.
  - Non-retryable errors (mapping invalid) dead-letter immediately; retryable errors dead-letter after max attempts or age.
security: signed/authenticated requests per provider.
failure_cases: [provider_timeout, rate_limited, auth_expired]
finance_report_effect: none.
i18n_a11y: none.
acceptance: With the mock adapter returning timeouts for 10 minutes, messages retry and deliver in order afterwards; a mapping error goes straight to dead letters.
dependency: certified adapter (`unverified-assumption`); mock adapter Phase 2.
```

```yaml
id: M07.F07.3.SF07.3.3
name: ARI dead letters and replay
phase: 3
release: R1
actors: [integration_admin, revenue_manager]
screens: [SCR-INT-dead-letters]
inputs: [dead_letter_id, action (replay/discard_with_reason/full_refresh)]
states: [open, replayed, discarded]
api: POST /v1/properties/{pid}/dead-letters/{id}/replay (idem)
events: [DeadLetterReplayed]
data: [dead_letter, ari_message]
rules:
  - Replay sends current PMS value, not the stale payload; discard requires reason and triggers full refresh for the affected range.
  - Open ARI dead letters for dates within 30 days count as oversell risk.
security: integration_admin.
failure_cases: [replay_of_superseded_value]
finance_report_effect: none.
i18n_a11y: none.
acceptance: Replaying a dead-lettered availability message after further sales sends the current availability, not the stale value.
dependency: M01 SF01.6.6.
```

```yaml
id: M07.F07.3.SF07.3.4
name: Full ARI refresh
phase: 3
release: R1
actors: [integration_admin, ari_worker]
screens: [SCR-INT-ari-monitor]
inputs: [connection_id, date_range]
states: [requested, running, completed, failed]
api: POST /v1/properties/{pid}/channel-connections/{cid}/full-refresh (idem)
events: [AriFullRefreshCompleted]
data: [ari_message]
rules:
  - Sends complete ARI for range, throttled; scheduled nightly for the next 365 days (configurable) and after mapping changes.
security: none.
failure_cases: [refresh_exceeds_rate_limit]
finance_report_effect: none.
i18n_a11y: none.
acceptance: After a full refresh, a sampled channel read-back (where capability exists) matches PMS for 100% of sampled cells.
dependency: adapter capability.
```

```yaml
id: M07.F07.3.SF07.3.5
name: ARI staleness and acknowledgment tracking
phase: 3
release: R1
actors: [revenue_manager, integration_admin]
screens: [SCR-INT-ari-monitor, SCR-REV-rate-grid]
inputs: [connection_id]
states: [in_sync, lagging, unknown]
api: GET /v1/properties/{pid}/channel-connections/{cid}/sync-status
events: [AriLagDetected]
data: [ari_ack]
rules:
  - Per connection, oldest unacknowledged change age is displayed; lag over threshold alerts; rate changes show channel acknowledgment for the revenue manager (M53 SF53.2.4).
security: none.
failure_cases: [ack_not_supported]
finance_report_effect: none.
i18n_a11y: none.
acceptance: A rate change shows "acknowledged by channel at hh:mm" or "not acknowledged" per connection within the SLO.
dependency: adapter capability.
```

## F07.4 Channel bookings and reconciliation

```yaml
id: M07.F07.4.SF07.4.1
name: Inbound channel booking ingest
phase: 3
release: R1
actors: [channel_worker]
screens: [SCR-INT-channel-bookings]
inputs: [provider_message_id, channel_reservation_id, type (new/modify/cancel), payload, signature]
states: [received, processed, needs_attention, rejected_duplicate]
api: POST /v1/integrations/channels/{connection_id}/bookings (signed webhook) or poll
events: [ChannelBookingReceived]
data: [channel_booking_message]
rules:
  - Inbox dedup on (connection, provider_message_id) and on (connection, channel_reservation_id, version); out-of-order modifications held until prior version processed.
  - Acknowledge to provider only after durable commit.
  - Raw payload stored encrypted with retention limit.
security: signature/credential verification; IP allowlist where provider supports.
failure_cases: [duplicate_delivery, modify_before_create, bad_signature]
finance_report_effect: commission accrual triggered.
i18n_a11y: Arabic names in payloads preserved.
acceptance: The same booking delivered 3 times creates one reservation; a modification arriving before its create is processed after the create.
dependency: adapter (`unverified-assumption`).
```

```yaml
id: M07.F07.4.SF07.4.2
name: Resolution queue for unmappable bookings
phase: 3
release: R1
actors: [front_office_manager, integration_admin]
screens: [SCR-INT-channel-bookings]
inputs: [channel_booking_message_id, resolution]
states: [open, resolved]
api: POST /v1/properties/{pid}/channel-bookings/{id}/resolve
events: [ChannelBookingResolved]
data: [channel_booking_message]
rules:
  - Unmapped room/rate or invalid data creates attention item with the raw offer; staff map and create the reservation; stock consumed at resolution with exposure if needed.
security: none.
failure_cases: [unresolved_near_arrival]
finance_report_effect: none.
i18n_a11y: none.
acceptance: An unmapped booking arriving 2 days before stay raises a high-priority attention item and is not lost.
dependency: none.
```

```yaml
id: M07.F07.4.SF07.4.3
name: Oversell on ingest
phase: 3
release: R1
actors: [channel_worker, front_office_manager]
screens: [SCR-REV-overbooking]
inputs: [channel_booking]
states: [accepted_with_exposure]
api: internal
events: [OversellDetected]
data: [room_type_inventory_night, reservation]
rules:
  - A channel booking for a sold-out night is accepted (the channel already sold it) and immediately creates exposure, a stop-sell refresh to all channels and a walk/relocation task.
security: none.
failure_cases: [exposure_ignored]
finance_report_effect: walk cost estimate.
i18n_a11y: none.
acceptance: A late-arriving channel booking on a sold-out date results in reservation creation, exposure 1, and an ARI zero-availability push to all connections.
dependency: SF03.4.6.
```

```yaml
id: M07.F07.4.SF07.4.4
name: Channel payment models and virtual cards
phase: 3
release: R1
actors: [cashier, finance_clerk]
screens: [SCR-FD-folio, SCR-FIN-channel-payments]
inputs: [payment_model (hotel_collect/channel_collect/virtual_card), vcc_token_ref, activation_date]
states: [expected, charged, settled, failed]
api: POST /v1/properties/{pid}/reservations/{rid}/channel-payment (idem)
events: [ChannelPaymentCharged]
data: [guarantee, folio_window]
rules:
  - Channel-collect bookings route room charges to a channel AR window; virtual cards charged via M28 only after activation date and only for the permitted amount.
  - VCC data only as PSP token.
security: PCI boundary; no card data in PMS.
failure_cases: [vcc_not_yet_active, amount_exceeds_vcc]
finance_report_effect: channel AR receivable vs guest folio separation.
i18n_a11y: none.
acceptance: Charging a virtual card before its activation date is blocked and scheduled for the activation date.
dependency: M28, OTA VCC support (`unverified-assumption`).
```

```yaml
id: M07.F07.4.SF07.4.5
name: PMS vs channel reconciliation
phase: 3
release: R1
actors: [integration_admin, revenue_manager, reconciliation_worker]
screens: [SCR-INT-reconciliation]
inputs: [connection_id, date_range, channel_report_or_api]
states: [scheduled, running, matched, differences_found, closed]
api: POST /v1/properties/{pid}/channel-connections/{cid}/reconciliations (idem)
events: [ChannelReconciliationCompleted, ChannelReconciliationDifference]
data: [channel_reconciliation_run, reconciliation_item]
rules:
  - Daily compare of reservations (existence, dates, room, price, status) and ARI sample; differences create items with owner.
  - Differences never auto-fixed in the channel; PMS-side corrections go through normal commands.
security: none.
failure_cases: [channel_report_unavailable]
finance_report_effect: commission and revenue accuracy.
i18n_a11y: none.
acceptance: A booking cancelled in the channel but active in PMS appears as a difference item within the next reconciliation run.
dependency: adapter capability or CSV import.
```

## F07.5 Attribution, commission, funnel and exposure

```yaml
id: M07.F07.5.SF07.5.1
name: Source, channel and segment codes
phase: 2
release: R1
actors: [revenue_manager, marketing_manager]
screens: [SCR-REV-source-codes]
inputs: [source_code, channel_type, segment, sub_source]
states: [active, retired]
api: PUT /v1/properties/{pid}/source-codes/{code}
events: none
data: [source_code]
rules:
  - Every reservation has exactly one source code and segment at creation; changes audited.
security: none.
failure_cases: [missing_source]
finance_report_effect: channel mix, segment reporting.
i18n_a11y: none.
acceptance: A reservation cannot be confirmed without source and segment.
dependency: none.
```

```yaml
id: M07.F07.5.SF07.5.2
name: Commission rules and accrual
phase: 3
release: R1
actors: [finance_clerk, revenue_manager]
screens: [SCR-FIN-commissions]
inputs: [channel, basis (room revenue net of tax/gross), percent, fixed, trigger (on stay/on booking)]
states: [estimated, accrued, invoiced, matched, disputed, reversed]
api: PUT /v1/properties/{pid}/commission-rules/{id}; GET .../commission-accruals
events: [CommissionAccrued, CommissionReversed]
data: [commission_rule, commission_accrual]
rules:
  - Accrual created at checkout (or per contract) from realized room revenue; cancellations/no-shows reverse or adjust per contract.
  - Accrual carries rule version.
security: none.
failure_cases: [contract_basis_unknown]
finance_report_effect: channel commission expense accrued to rooms department.
i18n_a11y: none.
acceptance: A 3-night OTA stay at 100 net per night with 15% commission accrues 45 at checkout; a later partial refund of one night reduces accrual to 30.
dependency: M19/M20 posting (Phase 4).
```

```yaml
id: M07.F07.5.SF07.5.3
name: Commission invoice matching
phase: 4
release: R1
actors: [ap_clerk, finance_approver]
screens: [SCR-FIN-commission-match]
inputs: [channel_invoice, accrual_items]
states: [matched, variance, disputed]
api: POST /v1/properties/{pid}/commission-invoices (idem)
events: [CommissionInvoiceMatched]
data: [commission_accrual, supplier_invoice (M20)]
rules:
  - Line-level match on channel reservation id; variances beyond tolerance disputed; payment only through M20 approvals.
security: SoD.
failure_cases: [invoice_for_cancelled_booking]
finance_report_effect: AP and commission expense true-up.
i18n_a11y: none.
acceptance: A channel invoice line for a no-show we did not charge is flagged disputed.
dependency: M20, M60 SF60.2.3.
```

```yaml
id: M07.F07.5.SF07.5.4
name: Sales funnel events
phase: 2
release: R1
actors: [marketing_manager, revenue_manager]
screens: [SCR-MKT-funnel]
inputs: [session, stage]
states: [search, offer_view, checkout_start, hold, payment, booked, abandoned]
api: POST /v1/public/funnel-events (batched); GET /v1/properties/{pid}/funnel
events: [FunnelEventRecorded]
data: [funnel_event, booking_session]
rules:
  - Recorded with analytics consent level; without consent only aggregated, cookieless counts.
  - Bots filtered; stage conversion and quote-to-book reported.
security: no PII in funnel events.
failure_cases: [consent_banner_blocked]
finance_report_effect: quote-to-book conversion KPI (P6).
i18n_a11y: none.
acceptance: Funnel conversion from search to booked for a test day equals booked reservations with source direct divided by filtered sessions.
dependency: M51 SF51.2.4.
```

```yaml
id: M07.F07.5.SF07.5.5
name: Oversell exposure and channel net contribution view
phase: 3
release: R1
actors: [revenue_manager, gm]
screens: [SCR-REV-channel-performance, SCR-REV-overbooking]
inputs: [date_range]
states: [none]
api: GET /v1/properties/{pid}/channel-performance
events: none
data: [reservation, commission_accrual, source_attribution]
rules:
  - Per channel - room nights, revenue, commission, payment fees (M28), cancellations, net contribution; plus oversell exposure contributed by each channel and open ARI dead letters.
  - Figures labeled estimate until commission invoices matched.
security: revenue scope.
failure_cases: [fees_missing]
finance_report_effect: channel net contribution (M32 SF32.1.3).
i18n_a11y: tables with text labels.
acceptance: Net contribution per channel equals revenue minus accrued commission minus attributed payment fees, labeled estimate until matched.
dependency: M28 fees (Phase 5).
```

```yaml
id: M07.F07.5.SF07.5.6
name: Campaign, metasearch and referral attribution tags
phase: 2
release: R1
actors: [marketing_manager]
screens: [SCR-MKT-attribution]
inputs: [utm_params, metasearch_click_id, referral_code, first_touch, last_touch]
states: [attributed, unattributed]
api: part of booking session; GET /v1/properties/{pid}/attribution
events: [BookingAttributed]
data: [source_attribution]
rules:
  - Deterministic precedence per property policy - referral code (M31) > explicit channel > last paid click > last touch; one owner per booking for commission purposes.
  - Attribution data retained per consent and retention policy.
security: no referrer sees guest PII.
failure_cases: [conflicting_codes]
finance_report_effect: marketing cost per completed stay (M51/M52).
i18n_a11y: none.
acceptance: A booking carrying both a referral code and a metasearch click is attributed to one owner per policy and the losing claim is recorded for audit.
dependency: M31, M51 (metasearch partners `unverified-assumption`).
```

### M07 key invariants
1. Availability published to any channel never exceeds real PMS availability except under an explicit overbooking policy for that channel.
2. Every inbound channel message is processed exactly once (inbox dedup) and acknowledged only after commit.
3. Unsupported partner capabilities are unavailable, never simulated; a non-certified connection is manual_only.
4. Dead-letter replay sends current state, never a stale payload.
5. Every booking has exactly one source/segment and at most one commissionable attribution owner.
6. Channel oversells always create visible exposure and a stop-sell refresh.

### M07 module-level acceptance
| Test | Maps to | Statement |
|---|---|---|
| AC-M07-1 | AT-G19.8 | Direct/search/channel visitor attributed; accessible low-bandwidth booking with correct total; net acquisition cost comparable per channel. |
| AC-M07-2 | AT-G19.9 | Revenue manager's rate change shows channel acknowledgment and is reversible. |
| AC-M07-3 | AT-G20.7 | Concurrent direct and channel bookings plus duplicate channel webhook → no double sale, one reservation. |
| AC-M07-4 | AT-G02.2 | Composite corporate booking stock not sellable on channels after confirmation (ARI reflects). |
| AC-M07-5 | AT-G08.4 | Channel fees flow into channel net contribution and GM drill-down. |

### M07 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-109 | Channel manager partner selection and certification path | Revenue Manager + Integration Admin | No partner assumed; mock adapter + manual extranet; production channel automation is a release blocker for hotels needing it. |
| D-120 | Build own booking engine (as specified) vs white-label partner | Product Owner | Own engine (SF07.1.x); partner engine not assumed. |
| D-129 | Attribution precedence and lookback windows | Marketing Manager + Referral Program Admin | Referral code > explicit channel > last paid click (30 days) > last touch. |
| D-130 | Commission accrual trigger and basis per channel contract | Financial Controller | At checkout on net room revenue excluding tax; estimate until invoice match. |

---

# M08 — Folio/cashiering

| Attribute | Value |
|---|---|
| Purpose | Keep an append-only guest and account ledger: charges, taxes, payments, reversals and transfers across multiple windows and payers with routing; deposits and card preauthorizations; invoices, credit notes and receipts per jurisdiction; cashier shifts, cash movements and refunds; night audit that posts room and tax, balances the guest ledger and rolls the business date; controlled reopening; and a finance export to the GL. |
| Build phase(s) | 2 (all core); 4 GL integration hardening (M19), AR transfer to M20; 5 PSP reconciliation depth (M28), e-invoicing adapters where verified (M38). |
| Release flag | R1. |
| Bounded context | `folio` (schema `folio`). |
| Systems of record owned | `folio`, `folio_window`, `folio_line`, `routing_instruction`, `deposit_ledger_entry`, `payment_authorization_ref` (PSP token/auth ids only), `fiscal_document` (`invoice`, `credit_note`, `receipt`, `pro_forma`), `fiscal_sequence`, `cashier_shift`, `cash_movement`, `night_audit_run`, `night_audit_step`, `guest_ledger_balance` (daily snapshot), `finance_export_batch`, `transaction_code` (charge/payment code catalogue with department, tax category and GL mapping ref). |
| Upstream dependencies | M01 (business date, audit, outbox), M02 (approvals, SoD, step-up), M04 (quote lines, package schedule), M05 (stays, reservations), M06 (minibar), M07 (channel payments), M13/M15/M17 (outlet charges, Phase 3), M28 (payments), M38 (tax, fiscal numbering/format), M44 (e-invoice gates). |
| Downstream dependents | M19 (GL journals), M20 (AR city ledger), M30 (points earn/reverse), M31 (referral margin), M32 (revenue KPIs), M60 (cash controls), M54 (voucher liability). |
| External dependencies | PSP via M28 — `unverified-assumption` until a PSP is `sandbox-tested`/`certified`; e-invoicing/fiscal clearance platforms per market (e.g. Saudi e-invoicing, Portugal certified invoicing software rules) — `unverified-assumption`, **must be validated in docs/07; no compliance claimed**; bank for cash deposits — manual. |

**Guest ledger equation (normative).** For each business date: `opening_balance + charges + taxes − payments − reversals ± transfers_net = closing_balance`, summed over all open folios; transfers between folios net to zero; `closing_balance(D) = opening_balance(D+1)`. Night audit fails if the equation does not hold to the minor unit.

**Folio line types:** `charge`, `tax`, `service_charge`, `payment`, `refund`, `reversal` (references original line), `transfer_out`/`transfer_in` (paired, same amount), `allowance` (adjustment with approval), `deposit_applied`. Lines are never updated or deleted.

## F08.1 Append-only folio ledger

```yaml
id: M08.F08.1.SF08.1.1
name: Folio and window creation
phase: 2
release: R1
actors: [system, front_desk_agent]
screens: [SCR-FD-folio]
inputs: [owner_type (stay/reservation/master/house/non_guest), owner_id, windows (1..8), payer_party_ids]
states: [open, settled, closed, reopened]
api: POST /v1/properties/{pid}/folios (idem); POST .../folios/{fid}/windows
events: [FolioOpened, FolioWindowAdded]
data: [folio, folio_window]
rules:
  - A stay folio is created at confirmation (for deposits) and becomes active at check-in; window 1 defaults to primary guest, others per routing.
  - Each window has exactly one payer (guest, company AR account, channel, third party) and currency = property base currency.
security: window visibility limited to payer-related roles and staff.
failure_cases: [payer_missing, window_limit_exceeded]
finance_report_effect: folio is sub-ledger of guest ledger (GL control account via M19).
i18n_a11y: window labels bilingual.
acceptance: A stay with company-paid room and guest-paid incidentals has two windows with distinct payers before first posting.
dependency: M05 SF05.2.1.
```

```yaml
id: M08.F08.1.SF08.1.2
name: Post charge
phase: 2
release: R1
actors: [front_desk_agent, cashier, night_audit_worker, pos_interface_worker]
screens: [SCR-FD-folio-post]
inputs: [folio_id, transaction_code, amount, quantity, outlet_id, description, source_ref, business_date, idempotency_key]
states: [posted]
api: POST /v1/properties/{pid}/folios/{fid}/lines (idem)
events: [FolioChargePosted]
data: [folio_line, transaction_code]
rules:
  - Tax and service charge lines are generated by M04/M38 rules at posting and linked to the charge line; manual tax entry forbidden.
  - Interfaces (POS, parking) must pass source_ref; duplicate (source system, source_ref) rejected.
  - Routing (SF08.2.1) chooses target window at posting; posting to closed folio is rejected.
security: manual charges limited by transaction code permissions.
failure_cases: [duplicate_interface_post, closed_folio, unknown_transaction_code]
finance_report_effect: revenue by department/outlet/transaction code on business_date.
i18n_a11y: none.
acceptance: A POS charge retried 3 times with the same source_ref posts once, with its tax lines, to the routed window.
dependency: M38 tax port; M13 POS (Phase 3).
```

```yaml
id: M08.F08.1.SF08.1.3
name: Reversal (void and correction)
phase: 2
release: R1
actors: [cashier, front_office_manager]
screens: [SCR-FD-folio]
inputs: [original_line_id, reason_code, comment, approval_id]
states: [reversed]
api: POST /v1/properties/{pid}/folios/{fid}/lines/{lid}/reverse (idem)
events: [FolioLineReversed]
data: [folio_line]
rules:
  - A reversal line of equal and opposite amount (including linked tax lines) references the original; the original is never altered.
  - Same-business-date reversal = void category; prior-date reversal = correction category requiring approval above threshold; both visible in audit reports.
  - A line can be reversed at most once (partial reversal via allowance).
security: approval policy and SoD (poster cannot approve own correction above threshold).
failure_cases: [double_reversal, reversal_on_invoiced_line]
finance_report_effect: revenue reduced on reversal business date; void/correction reports to M60.
i18n_a11y: none.
acceptance: Reversing a line twice returns 409; reversing an invoiced line requires a credit note path (SF08.4.2).
dependency: M02 F02.4.
```

```yaml
id: M08.F08.1.SF08.1.4
name: Transfer between windows, folios and accounts
phase: 2
release: R1
actors: [cashier, front_desk_agent]
screens: [SCR-FD-folio-transfer]
inputs: [source_line_ids, target_folio_window, reason]
states: [transferred]
api: POST /v1/properties/{pid}/folios/{fid}/transfers (idem)
events: [FolioLinesTransferred]
data: [folio_line]
rules:
  - Transfer writes paired transfer_out/transfer_in lines with identical amount and original line reference; revenue attribution stays with the original charge.
  - Transfer to AR city ledger requires approved direct-bill account and moves balance to M20 on checkout.
security: target folio must be within property; cross-property later.
failure_cases: [target_closed, ar_account_on_hold]
finance_report_effect: no revenue change; ledger balance moves.
i18n_a11y: none.
acceptance: Transferring a 50 bar charge from guest window to company window leaves bar revenue unchanged and both window balances updated by 50.
dependency: M20 AR (Phase 4; manual AR list before).
```

```yaml
id: M08.F08.1.SF08.1.5
name: Allowance and adjustment with approval
phase: 2
release: R1
actors: [front_office_manager, guest_relations]
screens: [SCR-FD-folio, SCR-GM-approvals-inbox]
inputs: [folio_id, target_line_id, amount, reason_code, recovery_case_id]
states: [requested, approved, posted, rejected]
api: POST /v1/properties/{pid}/folios/{fid}/allowances (idem)
events: [FolioAllowancePosted]
data: [folio_line, approval_request]
rules:
  - Allowances reference the charge or recovery case (M55); tax adjusted proportionally per M38 rules.
  - Caps per role; above cap requires approval.
security: SoD.
failure_cases: [allowance_exceeds_charge]
finance_report_effect: allowance account per department; service recovery cost report (M55 SF55.2.5).
i18n_a11y: none.
acceptance: An allowance larger than the referenced charge is rejected; an approved allowance links to its recovery case.
dependency: M55.
```

```yaml
id: M08.F08.1.SF08.1.6
name: Master, house and non-guest folios
phase: 2
release: R1
actors: [cashier, sales_manager]
screens: [SCR-FD-account-folios]
inputs: [folio_type (group_master/event_master/house/paymaster/non_guest), owner_ref, credit_limit]
states: [open, closed]
api: POST /v1/properties/{pid}/folios (idem)
events: [FolioOpened]
data: [folio]
rules:
  - Paymaster (non-room) folios for events, day guests, walk-in outlet customers and house accounts; house accounts require cost-center mapping.
  - Group/event master receives routed charges from member stays (M12).
security: house account posting restricted.
failure_cases: [house_account_misuse]
finance_report_effect: house use to cost center, not revenue.
i18n_a11y: none.
acceptance: A charge posted to a house account appears as internal cost for the mapped cost center and not in room/outlet revenue.
dependency: M12 (Phase 3), M19.
```

## F08.2 Routing and multiple payers

```yaml
id: M08.F08.2.SF08.2.1
name: Routing instructions
phase: 2
release: R1
actors: [front_desk_agent, sales_manager]
screens: [SCR-FD-routing]
inputs: [source_folio_or_stay, transaction_code_groups, date_range, target_window_or_folio, limit_amount]
states: [active, expired, suspended]
api: POST /v1/properties/{pid}/routing-instructions; PATCH .../routing-instructions/{id}
events: [RoutingInstructionChanged]
data: [routing_instruction]
rules:
  - Matching precedence - specific transaction code > group > default window; date-bounded; optional amount cap after which charges fall back to guest window.
  - Routing to another stay's folio requires that stay's payer consent (e.g. parent paying child's room).
security: none.
failure_cases: [cap_exceeded, conflicting_routes]
finance_report_effect: correct payer receivable.
i18n_a11y: none.
acceptance: With room and tax routed to company up to 300 per night, a 320 night posts 300 to company and 20 to guest window.
dependency: none.
```

```yaml
id: M08.F08.2.SF08.2.2
name: Split billing and corporate direct bill
phase: 3
release: R1
actors: [cashier, ar_clerk]
screens: [SCR-FD-check-out, SCR-FIN-ar-transfer]
inputs: [window_id, company_account_id, po_number, approval_ref]
states: [pending_transfer, transferred_to_ar]
api: POST /v1/properties/{pid}/folios/{fid}/windows/{wid}/transfer-to-ar (idem)
events: [FolioTransferredToAr]
data: [folio_window, ar_invoice (M20)]
rules:
  - Transfer at checkout creates AR item with invoice issued to the company legal entity; credit limit checked (M20 SF20.2.5).
security: ar scope.
failure_cases: [credit_limit_exceeded, po_missing]
finance_report_effect: AR receivable instead of guest ledger.
i18n_a11y: none.
acceptance: A company window of 900 transfers to AR at checkout creating one AR item and an invoice to the company, guest window settled separately.
dependency: M10, M20.
```

```yaml
id: M08.F08.2.SF08.2.3
name: Mid-stay routing change and re-route
phase: 2
release: R1
actors: [front_desk_agent, front_office_manager]
screens: [SCR-FD-routing]
inputs: [routing_instruction_id, new_target, apply_to_existing (bool)]
states: [applied]
api: POST /v1/properties/{pid}/routing-instructions/{id}/apply-retroactively (idem)
events: [FolioLinesTransferred]
data: [routing_instruction, folio_line]
rules:
  - Retroactive application is executed as transfers (SF08.1.4), never as edits.
security: approval if target is company AR.
failure_cases: [lines_already_invoiced]
finance_report_effect: none.
i18n_a11y: none.
acceptance: Applying a new route retroactively creates paired transfer lines for each matched past charge.
dependency: none.
```

```yaml
id: M08.F08.2.SF08.2.4
name: Credit limit and balance monitoring
phase: 2
release: R1
actors: [night_auditor, cashier, front_office_manager]
screens: [SCR-FD-credit-monitor]
inputs: [folio_window, authorized_amount, balance]
states: [within_limit, near_limit, over_limit]
api: GET /v1/properties/{pid}/credit-monitor
events: [FolioOverLimit]
data: [folio_window, payment_authorization_ref]
rules:
  - Balance vs preauth or credit limit monitored per window; over-limit triggers incremental authorization request or front desk contact task.
security: none.
failure_cases: [incremental_auth_declined]
finance_report_effect: bad-debt risk.
i18n_a11y: none.
acceptance: A window reaching 90% of its preauth raises near_limit and requests incremental authorization via M28.
dependency: M28.
```

## F08.3 Deposits, preauthorization, payments and refunds

```yaml
id: M08.F08.3.SF08.3.1
name: Deposit request and schedule
phase: 2
release: R1
actors: [system, front_desk_agent, sales_manager]
screens: [SCR-FD-reservation-guarantee, SCR-FIN-deposits-due]
inputs: [reservation_id, policy_snapshot_deposit_schedule]
states: [scheduled, requested, received, overdue, waived]
api: GET /v1/properties/{pid}/deposits?state=due; POST .../reservations/{rid}/deposit-requests (idem)
events: [DepositRequested, DepositOverdue]
data: [deposit_ledger_entry, policy_snapshot]
rules:
  - Schedule from snapshot (e.g. 30% at booking, balance 7 days prior); pay-by-link via M28.
  - Overdue deposit triggers attention item and optional auto-cancel per policy after notice.
security: none.
failure_cases: [link_expired, partial_payment]
finance_report_effect: none until received.
i18n_a11y: request email/SMS bilingual.
acceptance: A deposit not paid by due date raises overdue and, if policy says auto-cancel after 48 h notice, cancels only after the notice period.
dependency: M28 pay-by-link.
```

```yaml
id: M08.F08.3.SF08.3.2
name: Deposit receipt and liability ledger
phase: 2
release: R1
actors: [cashier, finance_clerk]
screens: [SCR-FIN-deposit-ledger]
inputs: [reservation_id, payment_ref, amount]
states: [held, applied, refunded, forfeited]
api: POST /v1/properties/{pid}/reservations/{rid}/deposits (idem)
events: [DepositReceived]
data: [deposit_ledger_entry, fiscal_document]
rules:
  - Advance deposits are a liability (not revenue) until applied at check-in or forfeited; receipt issued; tax treatment of advance payments per M38 market rule.
  - Deposit ledger balance reconciles to GL deposit liability at night audit.
security: none.
failure_cases: [deposit_on_cancelled_reservation]
finance_report_effect: advance deposit liability; tax point per market (M38, `unverified-assumption`).
i18n_a11y: receipt bilingual.
acceptance: Deposit ledger total equals the sum of held deposits by reservation and matches the GL control account export.
dependency: M38 advance payment tax rule.
```

```yaml
id: M08.F08.3.SF08.3.3
name: Card preauthorization and incidental holds
phase: 2
release: R1
actors: [front_desk_agent, system]
screens: [SCR-FD-check-in, SCR-FD-credit-monitor]
inputs: [folio_window_id, amount, payment_method_token, terminal_id, idempotency_key]
states: [requested, authorized, incremented, partially_captured, captured, released, expired, declined]
api: POST /v1/properties/{pid}/folios/{fid}/windows/{wid}/authorizations (idem)
events: [PreauthAuthorized, PreauthDeclined, PreauthReleased]
data: [payment_authorization_ref]
rules:
  - Amount = room and tax for stay + incidental allowance per night (D-117); card-present via terminal or token.
  - Authorization expiry tracked per scheme/PSP; re-auth before expiry for long stays.
  - Release remaining hold on checkout promptly; guest informed of hold amount.
security: PSP token only; no PAN/CVV in PMS.
failure_cases: [declined, psp_timeout_ambiguous, auth_expired]
finance_report_effect: none until capture.
i18n_a11y: hold amount explained in plain language.
acceptance: A PSP timeout during preauth leaves state requested and an inquiry resolves it; no second authorization is sent before the inquiry result.
dependency: M28 (`unverified-assumption` until PSP chosen).
```

```yaml
id: M08.F08.3.SF08.3.4
name: Payment posting (card, cash, transfer, city ledger)
phase: 2
release: R1
actors: [cashier, front_desk_agent]
screens: [SCR-FD-folio-payment]
inputs: [folio_window_id, tender_type, amount, payment_ref, cashier_shift_id, idempotency_key]
states: [pending, posted, failed]
api: POST /v1/properties/{pid}/folios/{fid}/windows/{wid}/payments (idem)
events: [FolioPaymentPosted]
data: [folio_line, cash_movement, payment_authorization_ref]
rules:
  - Card payments post only on PSP success (capture); cash payments require an open cashier shift; bank transfers post on confirmed receipt or as pending with evidence.
  - Split tender allowed; overpayment creates credit balance requiring refund or transfer.
security: cashier role; step-up for manual payment references.
failure_cases: [capture_failed, cash_without_shift, duplicate_callback]
finance_report_effect: payments by tender; PSP reconciliation in M28.
i18n_a11y: none.
acceptance: A duplicated PSP success webhook posts one payment line.
dependency: M28.
```

```yaml
id: M08.F08.3.SF08.3.5
name: Refunds with approval
phase: 2
release: R1
actors: [cashier, front_office_manager, financial_controller]
screens: [SCR-FD-folio-refund, SCR-GM-approvals-inbox]
inputs: [folio_window_id, original_payment_line_id, amount, reason, refund_method]
states: [requested, approved, submitted, completed, failed, rejected]
api: POST /v1/properties/{pid}/folios/{fid}/refunds (idem)
events: [RefundRequested, RefundCompleted, RefundFailed]
data: [folio_line, approval_request]
rules:
  - Refund to original payment method by default; alternative method requires higher approval.
  - Cumulative refunds cannot exceed original payment; refund lines post only on PSP confirmation (card) or cash out with shift.
  - Refunds trigger loyalty/referral reversals via events.
security: SoD (requester ≠ approver); step-up above threshold.
failure_cases: [refund_exceeds_payment, psp_refund_timeout, double_submit]
finance_report_effect: refund lines; M30/M31 reversal; M60 duplicate refund detection.
i18n_a11y: none.
acceptance: Two concurrent refund requests totaling more than the original payment result in one approval-eligible refund and one rejection; a timed-out PSP refund stays submitted until inquiry.
dependency: M28 SF28.1.6.
```

```yaml
id: M08.F08.3.SF08.3.6
name: Deposit application and forfeiture
phase: 2
release: R1
actors: [system, night_audit_worker]
screens: [SCR-FIN-deposit-ledger]
inputs: [reservation_id, event (check_in/cancel/no_show)]
states: [applied, forfeited, refund_due]
api: internal on GuestCheckedIn, ReservationCancelled, ReservationNoShow
events: [DepositApplied, DepositForfeited]
data: [deposit_ledger_entry, folio_line]
rules:
  - Check-in moves deposit to folio as deposit_applied payment line; cancellation applies fee from snapshot against deposit and refunds excess; forfeiture recognized as revenue per policy.
security: none.
failure_cases: [deposit_exceeds_fee_refund_fail]
finance_report_effect: liability to revenue or refund.
i18n_a11y: none.
acceptance: Cancelling with 100 deposit and 60 fee posts 60 cancellation revenue and a 40 refund request.
dependency: SF05.3.2.
```

## F08.4 Invoices, credit notes and receipts

```yaml
id: M08.F08.4.SF08.4.1
name: Issue invoice
phase: 2
release: R1
actors: [cashier, front_desk_agent, guest]
screens: [SCR-FIN-invoice-preview, SCR-FD-check-out]
inputs: [folio_window_id, bill_to (name/legal entity/tax id/address), locale, template_version]
states: [draft, issued, cancelled_by_credit_note]
api: POST /v1/properties/{pid}/folios/{fid}/windows/{wid}/invoices (idem)
events: [InvoiceIssued]
data: [fiscal_document, fiscal_sequence, folio_line]
rules:
  - Number from gap-free sequence per legal entity, document type and series; allocated in the issuing transaction.
  - Content (mandatory fields, tax breakdown, language, QR/hash where required) from M38 market template; issued invoices immutable and reproducible (SF01.5.5).
  - Lines included are locked; later changes only via credit note.
security: bill_to edits before issue only.
failure_cases: [sequence_gap_on_rollback, missing_buyer_tax_id_required]
finance_report_effect: revenue documentation; tax reporting feed (M38).
i18n_a11y: bilingual per market rule; tagged PDF.
acceptance: 1,000 concurrent invoice issues produce 1,000 consecutive numbers with no gap or duplicate; a rolled-back transaction does not consume a number.
dependency: M38 invoice rules per market (`unverified-assumption`).
```

```yaml
id: M08.F08.4.SF08.4.2
name: Credit note
phase: 2
release: R1
actors: [cashier, front_office_manager]
screens: [SCR-FIN-invoice-detail]
inputs: [invoice_id, lines_or_amount, reason]
states: [issued]
api: POST /v1/properties/{pid}/invoices/{iid}/credit-notes (idem)
events: [CreditNoteIssued]
data: [fiscal_document, folio_line]
rules:
  - References original invoice; cannot exceed original remaining; posts matching reversal lines; own number sequence.
security: approval above threshold.
failure_cases: [credit_exceeds_invoice]
finance_report_effect: revenue and tax reduction on credit note date.
i18n_a11y: bilingual.
acceptance: A credit note for a full invoice followed by a corrected invoice yields net revenue equal to the corrected invoice.
dependency: M38.
```

```yaml
id: M08.F08.4.SF08.4.3
name: Payment receipt
phase: 2
release: R1
actors: [cashier, guest]
screens: [SCR-FD-folio-payment, SCR-GST-receipts]
inputs: [payment_line_id]
states: [issued]
api: POST /v1/properties/{pid}/payments/{lid}/receipts (idem)
events: [ReceiptIssued]
data: [fiscal_document]
rules:
  - Receipts for deposits and payments; card data shown masked from PSP data only.
security: none.
failure_cases: [receipt_for_failed_payment]
finance_report_effect: none.
i18n_a11y: guest can retrieve receipts in app/web.
acceptance: A guest retrieves all receipts for their stay from the guest portal after checkout.
dependency: M18 guest app.
```

```yaml
id: M08.F08.4.SF08.4.4
name: Folio preview and pro-forma
phase: 2
release: R1
actors: [guest, front_desk_agent, corporate_booker]
screens: [SCR-FD-folio, SCR-GST-stay-bill]
inputs: [folio_window_id]
states: [preview]
api: GET /v1/properties/{pid}/folios/{fid}/windows/{wid}/preview
events: none
data: [folio_line]
rules:
  - Pro-forma clearly labeled not a tax invoice; guests see only windows where they are payer.
security: payer scope.
failure_cases: [preview_shows_other_payer_lines]
finance_report_effect: none.
i18n_a11y: accessible statement table.
acceptance: A guest viewing their in-stay bill sees incidental window lines only, not company-routed room charges.
dependency: none.
```

```yaml
id: M08.F08.4.SF08.4.5
name: E-invoicing and fiscal submission gate
phase: 4
release: R1
actors: [compliance_officer, fiscal_worker]
screens: [SCR-FIN-fiscal-submissions]
inputs: [fiscal_document_id, market_route]
states: [not_required, pending, submitted, accepted, rejected, manual_path]
api: POST /v1/properties/{pid}/fiscal-documents/{id}/submit (idem)
events: [FiscalDocumentSubmitted, FiscalDocumentRejected]
data: [fiscal_document, government_submission_ref]
rules:
  - Only markets whose M44 rule pack and M38 connector are verified submit electronically; otherwise manual_path with clear label; never marked accepted without receipt.
  - Rejections create correction workflow (credit note + reissue) per market rules.
security: credentials in vault; M38 SF38.3.x controls.
failure_cases: [route_unverified, platform_outage, rejected_schema]
finance_report_effect: tax compliance status per document.
i18n_a11y: none.
acceptance: In a fixture market whose route is unverified, issuing an invoice sets manual_path and the compliance dashboard counts it as not submitted.
dependency: M38 F38.3, M44 (`blocked` until verified per market).
```

## F08.5 Cashier shifts, cash and refunds

```yaml
id: M08.F08.5.SF08.5.1
name: Open cashier shift with float
phase: 2
release: R1
actors: [cashier, front_desk_agent]
screens: [SCR-FIN-cashier-shift]
inputs: [cashier_id, drawer_id, opening_float_by_denomination, currency]
states: [open, closing, closed, reconciled]
api: POST /v1/properties/{pid}/cashier-shifts (idem)
events: [CashierShiftOpened]
data: [cashier_shift, cash_movement]
rules:
  - One open shift per cashier per drawer; float issued from safe with dual count.
security: cashier identity via personal login (no shared).
failure_cases: [second_open_shift, float_mismatch]
finance_report_effect: cash control (M60 SF60.1.1).
i18n_a11y: none.
acceptance: Opening a second shift for the same cashier fails until the first is closed.
dependency: M60.
```

```yaml
id: M08.F08.5.SF08.5.2
name: Cash movements (paid-out, drop, safe)
phase: 2
release: R1
actors: [cashier, duty_manager]
screens: [SCR-FIN-cashier-shift]
inputs: [shift_id, movement_type (paid_out/drop/float_top_up/petty_cash), amount, evidence, approver]
states: [recorded]
api: POST /v1/properties/{pid}/cashier-shifts/{sid}/movements (idem)
events: [CashMovementRecorded]
data: [cash_movement]
rules:
  - Paid-outs require receipt evidence and approval above limit; drops recorded with bag id and witness.
security: SoD for approval.
failure_cases: [paid_out_without_evidence]
finance_report_effect: petty cash expense to cost center; cash in transit.
i18n_a11y: none.
acceptance: A paid-out above limit without approval is rejected.
dependency: M60 SF60.1.2.
```

```yaml
id: M08.F08.5.SF08.5.3
name: Close shift with blind count and variance
phase: 2
release: R1
actors: [cashier, front_office_manager]
screens: [SCR-FIN-cashier-close]
inputs: [shift_id, counted_by_tender_and_denomination]
states: [closed_balanced, closed_with_variance, reviewed]
api: POST /v1/properties/{pid}/cashier-shifts/{sid}/close (idem)
events: [CashierShiftClosed, CashVarianceDetected]
data: [cashier_shift, cash_movement]
rules:
  - Blind count - cashier enters counts before seeing expected; variance beyond tolerance requires manager review and reason.
  - Card totals compared to terminal batch where available.
security: expected totals hidden until count submitted.
failure_cases: [variance_over_tolerance, terminal_batch_missing]
finance_report_effect: over/short to cash variance account.
i18n_a11y: none.
acceptance: The close screen does not display expected cash until counts are submitted; a 5.000 shortage over 1.000 tolerance requires manager review.
dependency: M60.
```

```yaml
id: M08.F08.5.SF08.5.4
name: Foreign-currency cash acceptance (gated)
phase: 2
release: R1
actors: [cashier, financial_controller]
screens: [SCR-FD-folio-payment]
inputs: [currency, amount, rate, rate_source]
states: [disabled, enabled]
api: POST /v1/properties/{pid}/folios/{fid}/windows/{wid}/payments (tender=fx_cash)
events: [FolioPaymentPosted]
data: [folio_line, cash_movement, fx_rate]
rules:
  - Disabled by default; if enabled, hotel posts base-currency equivalent at published house rate; change given in base currency only; activation requires M44 gate (currency exchange licensing may apply).
security: financial_controller sets rates.
failure_cases: [gate_blocked]
finance_report_effect: FX gain/loss account.
i18n_a11y: none.
acceptance: With the gate blocked, foreign-currency tender is not offered.
dependency: M44 (`unverified-assumption` licensing per market).
```

```yaml
id: M08.F08.5.SF08.5.5
name: Cash refund limits
phase: 2
release: R1
actors: [cashier, duty_manager]
screens: [SCR-FD-folio-refund]
inputs: [refund_request_id, shift_id]
states: [paid_out, rejected]
api: POST /v1/properties/{pid}/refunds/{rid}/pay-cash (idem)
events: [RefundCompleted]
data: [cash_movement, folio_line]
rules:
  - Cash refunds only for cash-originated payments unless approved; per-refund and per-shift caps; guest signature captured.
security: approval and step-up.
failure_cases: [cash_refund_for_card_payment]
finance_report_effect: cash reduction in shift.
i18n_a11y: e-sign accessible alternative.
acceptance: A cash refund of a card payment is blocked without financial_controller approval.
dependency: M41 e-sign (optional).
```

## F08.6 Night audit, reopening and finance export

```yaml
id: M08.F08.6.SF08.6.1
name: Pre-audit checks
phase: 2
release: R1
actors: [night_auditor]
screens: [SCR-FIN-night-audit]
inputs: [business_date]
states: [passed, warnings, blocking_errors]
api: POST /v1/properties/{pid}/night-audit-runs (idem) then GET .../night-audit-runs/{run}/checks
events: [NightAuditStarted]
data: [night_audit_run, night_audit_step]
rules:
  - Checks - arrivals not checked in (no-show candidates), departures not checked out, open cashier shifts, unresolved FO/HK discrepancies, pending interface postings (POS queues), inventory integrity job, dead letters affecting postings.
  - Blocking errors must be resolved or acknowledged by authorized role with reason.
security: night_auditor.
failure_cases: [open_shift, pos_queue_backlog]
finance_report_effect: none.
i18n_a11y: checklist accessible.
acceptance: Audit cannot proceed past checks with an open cashier shift unless the front_office_manager force-closes it with reason.
dependency: M06 SF06.3.5, M13 queues.
```

```yaml
id: M08.F08.6.SF08.6.2
name: Room, package and tax posting run
phase: 2
release: R1
actors: [night_audit_worker]
screens: [SCR-FIN-night-audit]
inputs: [business_date, in_house_stays, posting_schedules]
states: [running, completed, failed]
api: POST /v1/properties/{pid}/night-audit-runs/{run}/post-room-charges (idem)
events: [RoomChargesPosted]
data: [folio_line, reservation_night]
rules:
  - For each in-house stay night equal to business date, post room and package components from reservation_night lines with tax; idempotent per (stay, night).
  - Day-use and complimentary nights post per rules (zero-rated comp with statistic).
security: service account.
failure_cases: [partial_failure_resume]
finance_report_effect: room revenue for the business date; statistics (rooms sold, comp, house).
i18n_a11y: none.
acceptance: Re-running the posting step after a crash midway posts only missing nights; each stay night has exactly one room charge.
dependency: M04 SF04.3.4.
```

```yaml
id: M08.F08.6.SF08.6.3
name: No-show and cancellation fee posting
phase: 2
release: R1
actors: [night_audit_worker, night_auditor]
screens: [SCR-FIN-night-audit-noshow]
inputs: [confirmed_no_shows, late_cancellations]
states: [posted, charge_failed]
api: POST /v1/properties/{pid}/night-audit-runs/{run}/post-fees (idem)
events: [NoShowFeePosted]
data: [folio_line, payment_authorization_ref]
rules:
  - Fees per snapshot posted to reservation folio and charged via token or deposit; failure creates AR/collection item.
security: none.
failure_cases: [token_declined]
finance_report_effect: no-show/cancellation revenue.
i18n_a11y: none.
acceptance: Each confirmed no-show has exactly one fee posting; declined tokens appear in the collections list.
dependency: SF05.3.3.
```

```yaml
id: M08.F08.6.SF08.6.4
name: Guest ledger balancing and audit reports
phase: 2
release: R1
actors: [night_auditor, financial_controller, gm]
screens: [SCR-FIN-night-audit-reports, SCR-GM-daily-flash]
inputs: [business_date]
states: [balanced, out_of_balance]
api: GET /v1/properties/{pid}/night-audit-runs/{run}/reports
events: [GuestLedgerBalanced, GuestLedgerOutOfBalance]
data: [guest_ledger_balance, folio_line, deposit_ledger_entry]
rules:
  - Verify guest ledger equation, deposit ledger vs liability, AR transfers vs M20, statistics (occupancy, ADR, RevPAR per KPI dictionary).
  - Out of balance blocks rollover; emits incident.
  - Reports - trial balance of guest ledger, revenue by department/transaction code, payments by tender, voids/corrections, allowances, overrides, rate variance, in-house list snapshot, cashier summaries.
security: reports scoped.
failure_cases: [equation_fails]
finance_report_effect: daily flash; source for M32 and M19 export.
i18n_a11y: reports exportable PDF/CSV bilingual headers.
acceptance: An injected unbalanced line (test only) makes the balance step fail and blocks rollover; a normal day balances to the minor unit.
dependency: M32 KPI dictionary.
```

```yaml
id: M08.F08.6.SF08.6.5
name: Reopen closed business date
phase: 2
release: R1
actors: [financial_controller, gm]
screens: [SCR-FIN-business-date-reopen]
inputs: [business_date, reason, scope]
states: [requested, approved, reopened, reclosed]
api: POST /v1/properties/{pid}/business-dates/{date}/reopen (idem)
events: [BusinessDateReopened, BusinessDateReclosed]
data: [business_day, night_audit_run]
rules:
  - Only the most recent closed date may be reopened, only if its accounting period is not locked and its finance export not yet accepted by GL (else corrections go to current date).
  - While reopened, only correction postings with reason allowed; reclose reruns balancing and regenerates export batch with version increment.
  - Operational posting continues on the current open date.
security: dual approval + step-up; audited.
failure_cases: [period_locked, export_already_posted]
finance_report_effect: restated daily reports flagged with version.
i18n_a11y: none.
acceptance: Reopening a date whose export was accepted by the GL is refused; reopening the last date allows a correction and a re-export with version 2 replacing version 1.
dependency: SF01.2.5, M19.
```

```yaml
id: M08.F08.6.SF08.6.6
name: Finance export batch to GL
phase: 2
release: R1
actors: [night_audit_worker, finance_clerk, financial_controller]
screens: [SCR-FIN-export-batches]
inputs: [business_date, mapping_version]
states: [generated, sent, accepted, rejected, superseded]
api: POST /v1/properties/{pid}/finance-exports (idem); GET .../finance-exports/{id}
events: [FinanceExportGenerated, FinanceExportAccepted]
data: [finance_export_batch, transaction_code]
rules:
  - Summarized journal by transaction code → GL account/department/tax code; balanced debits/credits; idempotent batch id per business date and version.
  - Phase 2 exports CSV/JSON to external accounting; Phase 4 posts to M19 internally.
  - Unmapped codes block export and list missing mappings.
security: financial_controller approves mapping changes.
failure_cases: [unmapped_code, external_import_rejected]
finance_report_effect: GL revenue, tax liability, deposits, AR, cash.
i18n_a11y: none.
acceptance: The export for a business date balances (debits = credits), maps every transaction code, and re-sending the same batch id does not duplicate GL entries.
dependency: M19 (Phase 4); external accounting system format `unverified-assumption` (D-131).
```

### M08 key invariants
1. Folio lines are append-only; corrections are reversal/transfer/allowance/credit-note lines referencing originals.
2. Guest ledger equation holds to the minor unit every business date; rollover is blocked otherwise.
3. Each external event (POS, PSP webhook, minibar count, room night) posts at most once (idempotency on source reference).
4. Fiscal documents use gap-free, per-legal-entity sequences, are immutable and reproducible; never marked accepted/submitted without a receipt.
5. No PAN/CVV in folio tables; payments post on PSP confirmation only; timeouts resolved by inquiry, never blind retry.
6. Refunds never exceed original payments; requester ≠ approver.
7. Deposits are liabilities until applied or forfeited.
8. A closed business date only reopens under dual approval and never after its period is locked or export accepted.

### M08 module-level acceptance
| Test | Maps to | Statement |
|---|---|---|
| AC-M08-1 | AT-G03.2 | Room and parking charges post once each to the correct routed windows. |
| AC-M08-2 | AT-G06.1 | Guest and corporate payments through the gateway (sandbox) with duplicate webhook produce one payment line. |
| AC-M08-3 | AT-G07.2 | Refund reverses charge and emits one points reversal. |
| AC-M08-4 | AT-G08.5 | Night audit balances guest ledger, reports occupancy/ADR/RevPAR and exports a balanced GL batch with drill-through to folio lines. |
| AC-M08-5 | AT-G02.3 | Corporate composite booking billed via master and individual windows with routing; post-event reconciliation ties to AR. |
| AC-M08-6 | AT-G09.3 | Five fixture markets produce jurisdiction-specific invoice outputs; unverified submission routes show manual_path. |
| AC-M08-7 | AT-G20.8 | Duplicate payment/provider webhook and repeated invoice attempt are surfaced/deduplicated. |

### M08 open decisions
| ID | Decision | Owner | Interim assumption |
|---|---|---|---|
| D-111 | Invoice numbering series, mandatory content and e-invoicing route per market | Financial Controller + counsel per market | One series per legal entity and document type; manual_path everywhere until M38 route verified. |
| D-113 | Night-audit reopen policy (how many days, who approves) | Financial Controller | Only last closed date, dual approval, never after GL acceptance. |
| D-117 | Preauth amount and incidental allowance per night | Front Office Manager + Financial Controller | Room+tax for stay + incidental allowance per night set per property currency. |
| D-131 | External accounting system and export format before M19 exists | Financial Controller | Generic balanced CSV/JSON journal; format adapter per system later. |
| D-132 | Foreign-currency cash acceptance | Financial Controller + counsel | Disabled; base currency cash only. |
| D-133 | Late-charge window after checkout and card-on-file consent wording | Front Office Manager + DPO | 72 h window; charge only with explicit card-on-file consent. |

---

## Cross-module assumptions for other catalogue writers

1. **Entity names** in §0.1 are canonical; do not create parallel `booking`, `guest`, `invoice`, `stock_night` or `charge` tables. Outlet modules post to `folio_line` through the M08 posting API with `source_ref` idempotency; they never write folio tables directly.
2. **Business date:** every operational/financial record carries `business_date` from `business_day`; modules must not derive it from wall-clock time. Offline devices keep their original business date if still open, else post late (SF01.2.3).
3. **Inventory:** only M03 decides saleability of rooms (`room_type_inventory_night`); M09 owns timed non-room resources with the same hold/expiry pattern and a composite-hold saga orchestrated by M12 (Phase 3).
4. **Tax:** M04/M08 call M38 `TaxPort.compute` and render fiscal documents with M38 templates; no module hard-codes rates.
5. **Payments:** all money movement via M28 ports; M08 stores only `payment_authorization_ref`. Timeouts resolved by inquiry.
6. **Consent:** marketing/communication/ID-processing purposes are checked via M02 `consent_record`; service messages are not marketing.
7. **Approvals:** use M02 `approval_request`/`approval_policy`; do not build module-specific approval tables.
8. **Events** named here (e.g. `ReservationConfirmed`, `GuestCheckedIn`, `GuestCheckedOut`, `StayCompleted` semantics carried by `GuestCheckedOut`, `FolioChargePosted`, `FolioLineReversed`, `RefundCompleted`, `RoomTypeAvailabilityChanged`, `BusinessDateRolled`, `DeviceEntitlementGranted/Revoked`) are the contract for M17, M30, M31, M32, M34–M36, M52, M55.
9. **Honesty:** channel manager, PSP, e-invoicing routes, guest-registration routes, lock systems, FX source and SMS/WhatsApp providers are all `unverified-assumption` in this file.

## Decision register for this file (D-101..D-133)

| ID | Module | Short title | Owner |
|---|---|---|---|
| D-101 | M01 | Night-audit rollover mode and deadline | Front Office Manager |
| D-102 | M01 | On-prem hardware/support model | IT Admin + MetriSys Ops |
| D-103 | M02 | Staff MFA factors and terminal PIN policy | IT Admin + GM |
| D-104 | M01 | RPO/RTO and offsite backup provider | GM + IT Admin |
| D-105 | M03 | Default overbooking limits | Revenue Manager |
| D-106 | M03 | Hold TTL defaults | Front Office Manager + Revenue Manager |
| D-107 | M04 | Firm quote validity and extension pricing | Revenue Manager + Sales Manager |
| D-108 | M04 | Tax-inclusive display per market | Compliance Officer + counsel |
| D-109 | M07 | Channel manager partner and certification | Revenue Manager + Integration Admin |
| D-110 | M05 | Cancellation/no-show defaults | Revenue Manager + GM |
| D-111 | M08 | Invoice series and e-invoicing per market | Financial Controller + counsel |
| D-112 | M04 | FX source and multi-currency | Financial Controller |
| D-113 | M08 | Business-date reopen policy | Financial Controller |
| D-114 | M06 | Offline conflict precedence | Housekeeping Supervisor + Front Office Manager |
| D-115 | M02 | Guest authentication method per market | Product Owner + DPO |
| D-116 | M01 | SaaS region and data residency | Compliance Officer + counsel |
| D-117 | M08 | Preauth/incidental amounts | Front Office Manager + Financial Controller |
| D-118 | M02 | Retention periods (ID, registration, fiscal) | DPO + counsel |
| D-119 | M05 | Walk policy and partner hotels | GM |
| D-120 | M07 | Own booking engine vs partner | Product Owner |
| D-121 | M01 | Digit shaping and Hijri display | Product Owner |
| D-122 | M04 | Child age bands | Revenue Manager |
| D-123 | M02 | Small-hotel SoD warn mode | Financial Controller |
| D-124 | M03 | Occupancy treatment of OOO/comp/house | Financial Controller |
| D-125 | M05 | Guest registration reporting routes | Compliance Officer + counsel |
| D-126 | M05 | Door-lock system | IT Admin + Chief Engineer |
| D-127 | M06 | Inspection policy/self-inspection | Housekeeping Supervisor |
| D-128 | M06 | DND welfare-check threshold | GM + Security |
| D-129 | M07 | Attribution precedence/lookback | Marketing Manager + Referral Program Admin |
| D-130 | M07 | Commission accrual trigger/basis | Financial Controller |
| D-131 | M08 | Pre-M19 accounting export format | Financial Controller |
| D-132 | M08 | Foreign-currency cash acceptance | Financial Controller + counsel |
| D-133 | M08 | Late-charge window and card-on-file consent | Front Office Manager + DPO |
