# Architecture and data foundation

## Decisions

- Multi-tenant modular monolith, PostgreSQL as system of record, HTTP API, background jobs for notifications/accrual/recalculation; object storage for documents.
- Every tenant-owned row carries `tenant_id`; authorization enforces tenant and record scope at the API/service layer. Add PostgreSQL row-level security where practical as defense in depth. Composite tenant-aware foreign keys and indexes prevent cross-tenant references.
- UUID primary keys, UTC timestamps plus tenant timezone for local dates, `created_at`/`updated_at`, actor attribution and effective dates on employment and policy changes. Store money as integer minor units with currency; never floating point.
- Separate authentication user from employee: an invited user may have no employee record, and an employee may have no login. Platform operator privileges never imply access to tenant employee data.
- Encrypt sensitive fields where applicable, keep document bytes outside the database, issue short-lived signed download URLs, audit reads and writes of sensitive records. Define retention and deletion rules before production.
- API validation and authorization are server-side. Requests that mutate balances, approvals or payroll run in database transactions with idempotency keys and optimistic concurrency where needed. Do not derive payroll from mutable live inputs after a run is locked.
- Design system is created from new tokens, typography, spacing and accessible patterns; no hrtion demo colors, layouts or components are imported.

## Domain modules and ownership

| Module | Responsibilities | First delivery |
| --- | --- | --- |
| Identity & tenant | Tenant provisioning, invitations, sessions, roles, scoped permissions | Phase 1 |
| Organization | Legal entities, locations, departments, designations, reporting lines | Phase 1 |
| People | Employee master, employment history, contact data, documents, lifecycle | Phase 2 |
| Workflow | Definitions, approver resolution, decisions, unified inbox, audit | Phase 2 core |
| Time | Holiday calendars, shifts, rosters, raw punches, daily attendance, exceptions | Phase 3 |
| Leave | Policies, credits/debits, requests, approvals, balances | Phase 3 |
| Payroll | Salary components, effective-dated assignments, input snapshot, runs, payslips, compliance adapters | Phase 4 |
| Employee finance | Expenses, reimbursements, advances and benefits | Phase 5 |
| Talent | Requisitions, candidates, offers, onboarding, goals and reviews | Phase 6 |
| Projects & services | Project allocation, timesheets, assets, helpdesk and travel | Phase 7 |
| Platform | Reports, notifications, integrations, audit, exports | Cross-cutting; deepen in Phase 8 |

## Core entities (proposed initial schema)

| Area | Tables / entities | Key relations and constraints |
| --- | --- | --- |
| Tenant | `tenants`, `legal_entities`, `locations` | Tenant slug unique; location belongs to tenant/entity |
| Identity | `users`, `memberships`, `roles`, `permissions`, `role_grants`, `sessions` | Email unique for login identity; membership connects user to tenant and scoped grants |
| Organization | `departments`, `designations`, `cost_centers` | Tenant-local codes unique; parent department must share tenant |
| People | `employees`, `employment_terms`, `employee_contacts`, `employee_documents`, `employee_events` | Employee number unique per tenant; optional user_id; manager_id same tenant; employment terms effective-dated without overlap |
| Workflow | `workflow_definitions`, `workflow_steps`, `approval_requests`, `approval_decisions` | Request references source type/id and policy version; approver snapshot retained |
| Time | `holiday_calendars`, `holidays`, `shifts`, `shift_assignments`, `attendance_punches`, `attendance_days`, `regularization_requests` | Raw punches immutable; daily result recomputable; uniqueness on employee/date for current result |
| Leave | `leave_types`, `leave_policies`, `leave_assignments`, `leave_ledger`, `leave_requests` | Ledger is append-only; balance is sum of posted entries; overlap validation and atomic reservation |
| Payroll later | `salary_components`, `salary_structures`, `salary_assignments`, `payroll_periods`, `payroll_inputs`, `payroll_runs`, `payroll_lines`, `payslips` | Effective-dated assignments; frozen input snapshot and explicit lock/finalize states |
| Shared | `audit_events`, `outbox_events`, `notification_deliveries`, `file_objects` | Actor, tenant, target, action, timestamp and correlation ID; retryable outbox delivery |

Implement only tables needed by the current phase. Keep personal and bank/statutory data in restricted tables; do not put them in generic audit payloads. Avoid a universal `settings` JSON blob for policy rules that require queries or historical replay.

## Invariants and workflows

1. Employee creation assigns a tenant-local employee number, department, location and optional manager. Manager cannot be self or form a cycle. Effective-dated job changes preserve history.
2. Scope: platform operator, tenant admin, HR admin, manager and employee are distinct grants. Managers see their reporting subtree only where policy allows; employees see their own sensitive records. Test two-tenant denial for every endpoint.
3. Approval flow: request submitted → definition/version selected → approvers resolved and snapshotted → sequential or parallel decisions → source module commits outcome once → audit/outbox event. Delegation and reassignment require an audit reason.
4. Leave: policy accrues ledger credits; request validates eligibility, date overlap, holiday/week-off treatment and available balance; approval posts debit exactly once; cancellation reverses with a compensating entry.
5. Attendance: preserve raw timestamp/source/device; calculate local workday using shift, holiday and timezone; regularization creates a request and approved correction, without rewriting raw punches.
6. Payroll: take immutable period input snapshot after attendance/leave lock; validate, calculate, review, finalize, then publish payslips. Jurisdiction-specific India PF/ESI/PT/LWF/TDS rules must be versioned and reviewed by a qualified payroll specialist before use.

## API and events

Use `/api/v1/tenants/:tenantId/...` or an authenticated tenant context consistently; never trust a tenant ID without membership validation. Expose resources for organizations, employees, workflow inbox, attendance, and leave. Cursor pagination, filtering, stable error codes and audit correlation IDs are required. Publish transactional outbox events such as `employee.created`, `leave.approved` and `attendance.regularized`; consumers must be idempotent. No direct cross-module table writes outside the owning module's service contract.

## Quality gates

Migration up/down and fresh database bootstrap; tenant and role isolation tests; leave concurrent approval/balance tests; attendance timezone/overnight-shift tests; audit integrity; accessibility checks for critical forms; backup restore rehearsal before production. Keep the initial deploy simple with separate web, API, worker, PostgreSQL and private object storage.
