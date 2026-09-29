# Kamal HRMS

Greenfield, multi-tenant HRMS foundation. The product design and visual language are independent of the existing hrtion demo. This repository starts with architecture and delivery documents; no application code has been implemented yet.

## Product boundary

One canonical employee record powers organization, attendance, leave, payroll, talent, projects and employee services. Tenant administrators configure their company; HR operates permitted workforce data; managers act within their team; employees use self-service. Platform operators are separate from tenant admins.

## Proposed repository structure

```text
apps/
  web/                 # Employee, manager and HR web UI
  api/                 # Versioned HTTP API and background job entry points
packages/
  domain/              # Framework-independent rules, types and authorization policies
  db/                  # PostgreSQL schema, migrations and seed fixtures
  contracts/           # API schemas and generated client types
  ui/                  # New design tokens and accessible components
  config/              # Shared lint, TypeScript and build settings
docs/
  ARCHITECTURE.md      # Modules, entities, boundaries and invariants
  BACKLOG.md           # Ordered implementation tickets and acceptance checks
infra/
  local/               # Local database and object-storage setup
  deploy/              # Deployment manifests and environment templates
tests/
  e2e/                 # Critical cross-module journeys
.github/workflows/      # Lint, typecheck, tests and migration checks
```

Create folders when implementation starts; empty directories are not tracked by Git. Start with a modular monolith and PostgreSQL, a single API and a web app. Keep domain boundaries in code so modules can be split only if scale demands it. Stack suggestion: TypeScript, React, Node.js, PostgreSQL; confirm the developer team's operational fit before choosing frameworks.

## Build order

1. Tenant setup, identity, RBAC and organization.
2. Employee master, lifecycle, documents and self-service.
3. Holidays, shifts, attendance and leave with shared approvals.
4. Payroll and India-specific statutory rules after policy review.
5. Expenses, hiring/onboarding, performance, timesheets and employee services.
6. Reports, integrations, mobile channels and advanced workflows.

See [architecture](docs/ARCHITECTURE.md) and [first backlog](docs/BACKLOG.md). Each phase requires tenant isolation, auditability and role-scoped access before release.

## Local development contract

Add `.env.example` with dummy values only, a one-command local setup, idempotent seed data, migration commands, CI checks and a documented backup/restore procedure with the first code milestone. Never store secrets, real employee information or production documents in Git.
