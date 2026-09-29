# Delivery phases and first implementation backlog

This is an ordered handoff, not a claim that features have been built. Estimate after the developer chooses frameworks and hosting. Treat each ID as one issue or split it when implementation size becomes clear.

## Phase gates

| Phase | Outcome | Exit check |
| --- | --- | --- |
| 0 — engineering setup | Reproducible workspace and CI | Fresh clone boots web/API/database with seed data; checks run on PR |
| 1 — tenant and access | Company setup, identity, organization | Two tenants cannot read or mutate each other's data; scoped roles verified |
| 2 — people and workflow | Employee master, lifecycle and reusable approvals | HR creates employee; manager/employee see only allowed data; decision audited |
| 3 — time and leave | Shifts, holidays, punches, leave ledger | Cross-midnight/timezone cases and concurrent requests verified |
| 4 — payroll | Configurable pay and India compliance | Specialist-approved rules; reconciliation, locking and payslip review |
| 5 — finance and talent | Expenses, hiring, onboarding, performance | Candidate-to-employee and expense approval journeys complete |
| 6 — projects and services | Timesheets, assets, helpdesk, travel | Scoped approvals and operational reports complete |
| 7 — platform depth | Integrations, exports, mobile API, analytics | Monitored jobs, backup restore, retention, performance and security review |

## First implementation backlog (execute in order)

| ID | Priority | Task | Acceptance criteria |
| --- | --- | --- | --- |
| KAM-001 | P0 | Initialize monorepo and local services | `apps/web`, `apps/api`, `packages/domain`, `packages/db`, `packages/contracts`, `packages/ui` build; one command starts local PostgreSQL and apps; README commands work on fresh clone |
| KAM-002 | P0 | CI and configuration hygiene | PR runs lint, typecheck, unit and migration checks; `.env.example` has placeholders; secrets and local data ignored; dependency lockfile committed |
| KAM-003 | P0 | Migration and seed framework | Empty DB migrates forward; deterministic demo seed makes two tenants, admin/manager/employee users and org units; rollback strategy documented |
| KAM-004 | P0 | Authentication and tenant membership | Invite, accept, login/logout, password reset or managed identity flow; inactive membership denied; sessions revoked on role change; no hard-coded admin bypass |
| KAM-005 | P0 | RBAC and record scope policy | Tenant admin, HR admin, manager, employee permissions encoded centrally; API tests prove cross-tenant denial, own-record access and manager subtree boundaries |
| KAM-006 | P0 | Tenant and organization CRUD | Tenant config, legal entity, location, department and designation validated; duplicate tenant-local codes rejected; parent references cannot cross tenants |
| KAM-007 | P0 | Audit and outbox foundation | Write actions record tenant, actor, target, time and correlation ID; transactional outbox retries safely; sensitive values redacted |
| KAM-008 | P1 | Employee master API and history | Create/update/list employee with tenant-local number, status, dates, department, location, designation, manager; manager cycles and overlapping employment terms rejected; lifecycle changes preserved |
| KAM-009 | P1 | Employee UI and self-service | New visual system; HR directory/profile/edit; employee own profile; manager scoped team list; keyboard and screen reader usable; sensitive fields hidden by role |
| KAM-010 | P1 | Private document service | File metadata and employee link; size/type validation, malware scanning plan, private object storage, short-lived access; access logged and cross-tenant fetch denied |
| KAM-011 | P1 | Shared approval engine v1 | Versioned definition for manager approval, request state machine, inbox, approve/reject and comments; duplicate decisions idempotent and audited |
| KAM-012 | P1 | Holiday/shift configuration | Location calendar, holiday dates, fixed and overnight shift, dated assignment; local date/timezone conversion tested |
| KAM-013 | P1 | Attendance ingestion and day result | Check-in/out creates immutable punches; day calculation tracks source, missing punch and status; repeated ingestion idempotent |
| KAM-014 | P1 | Attendance correction | Employee submits regularization; manager approves via shared workflow; derived day recalculates; raw punch remains unchanged |
| KAM-015 | P1 | Leave policy and ledger | Types, eligibility, accrual and append-only balance entries; tenant-specific policy assignment; two simultaneous requests cannot overspend balance |
| KAM-016 | P1 | Leave request journey | Apply/cancel, date overlap and holiday handling, manager approval, calendar view; debit/reversal posted once and audit visible |
| KAM-017 | P1 | Production readiness checkpoint | End-to-end joiner → attendance → leave approval; tenant isolation suite; backup/restore rehearsal; error monitoring and access review documented |

## Definition of done for every ticket

- Schema/API contract and migration reviewed; tenant ID and authorization checked server-side.
- Happy path and failure/permission tests; meaningful boundary cases for dates and concurrent writes.
- Audit and privacy behavior stated; no production personal data in test fixtures or logs.
- UI uses the new design system and accessible form states when a UI is involved.
- Developer notes explain local setup, configuration and operational impact.

## Product decisions to confirm before payroll

Target customer segments and tenancy model; country/state rollout; leave accrual and carry-forward policies; attendance channels (web/biometric/GPS); payroll frequency and statutory treatment; hosting/data residency; identity provider; required integrations. Keep these configurable, and defer legal/tax calculations until policies are validated. No design decision depends on the hrtion demo's visual theme.
