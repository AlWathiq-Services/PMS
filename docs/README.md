# MetriStay Hospitality Suite — Phase 1 Planning Pack

**Product:** MetriStay Hospitality Suite (Metrikingdom • MetriSys technology brand)
**Phase:** 1 — Entire-project planning. **No production code exists and none may be written until the planning gate in `docs/13` is signed off.**
**Pack version:** 0.1 (draft for review) • **Date:** 2026-09-28
**Governing source:** `docs/source/master-prompt-v3.0.md` (Consolidated Phased Master Prompt v3.0, 28 Sep 2026).

> Everything in this pack is a specification or design target. Nothing here claims that working software, a partner contract, a government API, or legal clearance exists. Where proof from a provider or counsel is missing, the document says so and names the gate.

## 1. Repository audit (Section I: "Inspect the repository first")

| Item | Finding (2026-09-28) |
|---|---|
| Repository `AlWathiq-Services/PMS` | Empty: no commits, no source, no schema history, no conventions to preserve. |
| Existing coverage vs M01–M68 | **0 of 68 modules implemented.** Every module is greenfield. |
| The "two original hospitality blueprints" | **Not supplied** with this session. The master prompt states they end in the OTA section and lack compliance/build-order sections. Requirements in this pack are derived from the master prompt only; blueprint ingestion is open decision **D-001** (owner: Product Owner). When supplied, each blueprint idea must be added to the traceability index (`docs/13` §3) with a module/feature link or an explicit exclusion reason. |

Because the repository is empty, the architecture is chosen fresh (see `docs/03-architecture.md` §2, ADR-001..ADR-012).

## 2. Documents

| # | File | Contents |
|---|---|---|
| 00 | `00-executive-scope.md` | Product boundaries, personas, Release 1 definition, success metrics, feasibility, staffing, cost & calendar |
| 01 | `01-module-catalogue.md` + `01-catalogue/*.md` | M01–M68 (+ discovered modules) → features → numbered subfeatures, Section-L schema |
| 02 | `02-journeys-and-states.md` | All required journeys and state machines with exception transitions |
| 03 | `03-architecture.md` | Context, module boundaries, ERDs, systems of record, SaaS/on-prem, offline, outbox/inbox, ledgers, access scoping, ADRs |
| 04 | `04-screens-and-design.md` | Web/mobile screen maps, annotated flows, bilingual/RTL, accessibility, app distribution |
| 05 | `05-integrations.md` | Every external integration contract, auth, idempotency, failure, reconciliation, licensing, partner owner |
| 06 | `06-finance-kpis.md` | Chart of accounts mapping, posting examples, allocation, salary confidentiality, KPI dictionary, profit bridge, close |
| 07 | `07-security-regulatory.md` | Threat model, privacy/retention, PCI boundary, wallet/PSP gating, Oman network-marketing gate, five-market validation register, NIS/SIN decision |
| 08 | `08-phase-backlog.md` | Work packages Phases 2–8, dependencies, estimates, critical path, gates, partner blockers, exclusions |
| 09 | `09-acceptance-and-migration.md` | Integrated scenario tests G1–G20, fixtures, performance/security/restore, pilot criteria, migration/rollback |
| 10 | `10-competitive-gap-and-questions.md` | Benchmark vs OPERA/Mews/Cloudbeds/SiteMinder; owner-question traceability |
| 11 | `11-guest-acquisition-and-service.md` | Website → booking → stay → recovery → return journey (M51–M55, M18, M40) |
| 12 | `12-operations-safety-continuity.md` | Operations, food safety, incident, continuity (M42, M47, M56–M68) |
| 13 | `13-simple-experience-and-acceptance.md` | Role home screens, simplicity rules, **traceability index**, consistency check, decision log, planning-gate checklist |

## 3. Shared conventions (all documents MUST follow)

### 3.1 Identifiers
- Module `Mnn` (M01–M68; discovered modules M69+). Feature `Fnn.k` (e.g. F29.1). Subfeature `SFnn.k.j` (e.g. SF29.1.7). Full id: `M29.F29.1.SF29.1.7`.
- Identifiers given in master-prompt Sections K and Q are **fixed** and must be reused verbatim.
- Journey/state machine `J-xx` / `SM-<name>`; screen `SCR-<app>-<name>`; integration `INT-<name>`; decision `D-nnn`; risk `R-nnn`; acceptance test `AT-<scenario>.<n>` (G1–G20 → `AT-G01.1` …); unit-level acceptance `AC-<SF id>`; work package `WP-<phase>.<n>`; ADR `ADR-nnn`.

### 3.2 Phases and release flags
- Phase 2–6 = **Customer Release 1: Single Hotel** (`R1`). Phase 7–8 = `Later`. No subset of 2–6 is "Release 1".
- Every subfeature carries `phase:` (first implementation phase) and `release: R1|Later`.

### 3.3 Standard actors (roles)
`owner`, `gm`, `duty_manager`, `revenue_manager`, `sales_manager`, `marketing_manager`, `guest_relations`, `front_desk_agent`, `front_office_manager`, `concierge`, `night_auditor`, `cashier`, `housekeeper`, `housekeeping_supervisor`, `laundry_attendant`, `executive_chef`, `shift_chef`, `backup_chef`, `emergency_chef` (external), `fnb_manager`, `bartender`, `server`, `club_host`, `catering_manager`, `storekeeper`, `receiver`, `procurement_officer`, `procurement_approver`, `engineer`, `chief_engineer`, `security_officer`, `parking_attendant`, `finance_clerk`, `ap_clerk`, `ar_clerk`, `finance_approver`, `payment_releaser`, `financial_controller`, `hr_officer`, `payroll_officer`, `payroll_approver`, `employee`, `compliance_officer`, `dpo` (privacy), `it_admin`, `integration_admin`, `property_admin`, `tenant_admin`, `content_editor`, `content_approver`, `referral_program_admin`, `guest`, `booker`, `corporate_admin`, `corporate_booker`, `corporate_approver`, `event_organizer`, `vendor_admin`, `vendor_user`, `referrer`, `auditor` (read-only), system actors `*_worker` (e.g. `billpay_worker`, `outbox_relay`, `night_audit_worker`, `ai_assistant`).

### 3.4 Section-L subfeature schema (mandatory for every SF)
```yaml
id: M29.F29.1.SF29.1.7
name: Verify pending bill payment
phase: 5
release: R1
actors: [finance_approver, billpay_worker]
screens: [SCR-FIN-bill-payment-detail, SCR-OPS-exception-queue]
inputs: [provider_id, biller_id, account_reference, order_id, idempotency_key]
states: [submitted, pending, confirmed, failed, disputed]
api: GET /v1/properties/{pid}/bill-payments/{order_id}/status
events: [BillPaymentStatusChanged, BillPaymentReconciliationNeeded]
data: [bill_payment_order, provider_status_log]
rules: [...]
security: ...
failure_cases: [...]
finance_report_effect: ...
i18n_a11y: ...
acceptance: ...
dependency: ...
```
Where a module group file uses compact tables for brevity, each row must still carry every field above.

### 3.5 API and event style
- REST JSON under `/v1/…`, property-scoped paths `/v1/properties/{pid}/…`; `Idempotency-Key` header mandatory on every mutating call that can move money, stock, inventory or external orders. OpenAPI 3.1 is the contract source.
- Domain events: PascalCase past tense (`ReservationConfirmed`), envelope `{event_id, type, version, tenant_id, property_id, occurred_at, business_date, actor, correlation_id, causation_id, payload}`; published via transactional outbox; consumed via inbox with dedup on `event_id`.
- Commands that touch external partners go through an adapter port (`docs/05`) with capability flags; unsupported operations are *unavailable*, never faked.

### 3.6 Honesty labels
Every integration, rule pack and legal position carries one status: `unverified-assumption`, `source-cited`, `counsel-reviewed`, `partner-contracted`, `sandbox-tested`, `certified`, `blocked`. UI and reports show `estimate` vs `reconciled` vs `certified`.

### 3.7 Baseline technology decisions (detailed in `docs/03` ADRs)
- **Modular monolith** in TypeScript (Node.js LTS, NestJS) with one bounded-context package per module group; PostgreSQL 16 as the single system of record (one schema per bounded context, row-level `tenant_id`/`property_id` scoping + Postgres RLS); transactional outbox/inbox tables; background workers on a Postgres-backed job queue (pg-boss) with Temporal considered for long-running sagas in Phase 3+.
- Web: Next.js (React) apps — admin/staff back office, guest booking website (SSR, low-bandwidth), corporate portal, vendor web fallback. Mobile: React Native (Expo) signed builds — staff app (offline-capable), guest app, corporate app, vendor app.
- Object storage S3-compatible (MinIO on-prem); secrets in Vault/KMS; OpenTelemetry logs/metrics/traces; Keycloak (OIDC/SAML, MFA) as identity provider for SaaS and on-prem.
- Local AI: self-hosted inference (image enhancement pipeline; LLM for guest assistant/follow-ups via a pluggable provider port; on-prem profile can use a locally hosted open-weight model after licence review).
- Deployment profiles: `saas` (multi-tenant, Kubernetes) and `onprem-single-hotel` (single tenant, Docker Compose/k3s on a hotel server with offsite encrypted backup).

### 3.8 Money, time and quantity
Money = integer minor units + ISO-4217 currency (OMR has 3 decimals). Time stored UTC + property IANA time zone; hotel **business date** separate from calendar and accounting period. Quantities = decimal with canonical UOM + conversion table.
