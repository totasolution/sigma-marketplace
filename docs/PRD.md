# Product Requirements Document
# Sigma Marketplace

**Version:** 2.0
**Date:** 2026-06-30
**Status:** Draft — Projects/Engagement model (see [docs/PROJECTS_DESIGN.md](PROJECTS_DESIGN.md))
**Owner:** Totasolution

> **What changed in v2.0.** Driven by **Permenaker No. 7 Tahun 2026** (a company's essential/core roles can no
> longer be outsourced), the platform's unit of work becomes the **Project** — a client contracts an
> *outcome-based engagement* from **one or more vendors**, fulfilled by **vendor-employed** workers. Projects support four pricing
> models (per-unit commission, recurring retainer, flat fee, milestone), accrual-based payouts, and a
> vendor-reports / client-monitors (with dispute) verification flow. The legacy **Job Order** becomes one project type (a scheduled
> site visit). Full design and data model: [docs/PROJECTS_DESIGN.md](PROJECTS_DESIGN.md).
>
> *Legal note: the regulatory effect is taken as described by the product owner and must be validated with counsel
> before launch.*

---

## 1. Executive Summary

Sigma Marketplace is a B2B field service platform that connects **clients** (organizations that need field work done) with **vendors** (businesses that manage worker workforces). Clients create **projects** for a specific service category, vendors assign their workers, and execution is tracked in real time. Workers follow structured SOPs and report outcomes with photo evidence — even offline.

The core value proposition:
- **For clients:** full visibility into field operations with structured, auditable outcome reporting per engagement, in a way that respects the regulatory boundary between contracting work and employing labor
- **For vendors:** centralized worker management, clear project dispatch, and automated payout accrual tracking
- **For workers:** clear task checklists, offline-capable mobile app, and transparent earnings history

**Engagement model.** A **Project** is the primary unit of work: a client contracts an outcome-based engagement
from **one or more vendors** (project↔vendor is many-to-many), fulfilled by those vendors' workers (who remain
vendor-employed — preserving the compliance boundary). Each project carries one **pricing model** (uniform across
its vendors in v1) and accrues payouts from **verified outcomes**.

---

## 2. Problem Statement

Field service operations today are fragmented:
- Job assignments happen over WhatsApp/email — no audit trail
- SOPs exist as PDF documents — compliance is unchecked
- Worker pay is manually calculated each month from spreadsheets
- Clients have zero real-time visibility into site work progress
- Evidence (photos) are stored in personal WhatsApp or Google Drive with no structure
- No clear separation between who requests work (client) and who supplies the workforce (vendor)
- New regulation (Permenaker 7/2026) bars outsourcing core roles — companies need a compliant way to engage
  outcome-based project work from vendors rather than directly outsourcing labor

Sigma replaces this with a structured, trackable, mobile-first platform with a clean client–vendor separation.

---

## 3. Target Users

### 3.1 Primary Users

| Role | Organization | Description | Platform |
|---|---|---|---|
| **Worker** | Vendor | Field worker executing project work | Mobile (primary), offline-capable |
| **Vendor Admin** | Vendor | Manages worker roster, assigns workers to projects, reports outcomes, tracks accruals | Web dashboard |
| **Client Admin** | Client | Creates projects, monitors progress, disputes outcomes (exception) | Web dashboard |
| **Super Admin** | Platform | Manages all orgs, service categories, SOP library, arbitrates disputes, platform-wide reports | Web admin panel |

### 3.2 User Personas

**Budi — Network Worker (Vendor)**
- 28 years old, field tech for 4 years, employed by PT. Sigma Teknik (vendor)
- Works at 3–5 client sites per week
- Limited connectivity at remote sites
- Needs: simple task checklist, photo upload, view of assigned work, log outcomes

**Andi — Vendor Operations Manager**
- 33 years old, runs a team of 15 workers at PT. Sigma Teknik
- Needs to see all workers' project assignments, report outcomes, reconcile accruals monthly
- Wants: worker roster, project history per tech, accrual summary

**Rina — Client Operations Manager**
- 35 years old, IT infrastructure manager at a telecom company (client)
- Creates projects (site work, sales campaigns, placed roles)
- Wants: contract trusted vendors, real-time progress, evidence on tap, dispute only when something's off

**Platform Admin**
- Internal Totasolution staff
- Manages client and vendor onboarding, global service category library, SOP management, dispute arbitration

---

## 4. Scope

### In Scope (MVP)

- Organization management: clients and vendors as distinct org types
- Client–vendor relationship management
- User management: client_admin, vendor_admin, worker roles
- **Worker management (vendor-internal):** the vendor's own roster — profiles, skills, availability (not linked to projects/payouts)
- Service category management (platform-level)
- SOP template management: per service category, versioned, with ordered steps
- **Project creation by clients with four pricing models** (per-unit commission, recurring retainer, flat fee, milestone); a project may engage **one or more vendors** (many-to-many), each of whom assigns its workers
- **Outcome reporting** by vendors/workers (visits, sales, milestones; retainer periods accrue automatically) with mandatory evidence
- **Client monitoring + dispute** (exception path) of outcomes, with a 7-day auto-confirm window
- Mobile app: offline task execution and **offline outcome logging** with photo evidence
- Real-time project progress on client and vendor web dashboards
- **Accrual-based payout tracking** (per worker, per period), aggregated from verified outcomes
- Push notifications (project/outcome events)
- Offline sync with conflict resolution

### Out of Scope (Post-MVP)

- Direct payment disbursement (bank transfer, e-wallet) — Phase 2
- Budget caps / capped incentive pools — later phase (only if a client needs to bound commission spend)
- Open marketplace (vendors/workers discoverable by any client) — Phase 3
- IoT/sensor integration — Phase 3
- AI-based SOP / fraud compliance checking — Future
- Worker rating/review system & performance analytics — Phase 2
- Worker employment/compliance document management (contracts, BPJS) — later phase
- Client visibility into individual workers — out (compliance: clients see the vendor, not workers)

---

## 5. Core Features & User Stories

### 5.1 Organization & User Management

**US-001** As a Super Admin, I can create and manage client organizations and vendor organizations independently so that each party has appropriate access.

**US-002** As a Super Admin, I can establish a client–vendor relationship so that a client can engage a specific vendor on projects.

**US-003** As a Vendor Admin, I can invite and manage workers within my vendor organization so they can be assigned to projects.

**US-004** As a Vendor Admin, I can deactivate a worker without losing their historical records.

**US-005** As any user, I can log in with email/password and receive a JWT so I can access the platform securely.

### 5.2 Service Categories & SOP Management

**US-010** As a Super Admin, I can create service categories (e.g. "LAN Installation", "Fiber Optic Splicing", "Site Survey", "Outlet Sales") so that SOPs and projects are organized by type of work.

**US-011** As a Super Admin, I can create and publish SOP templates under a service category, with an ordered list of steps, photo requirements, and order enforcement rules.

**US-012** As a Super Admin, I can version SOPs — published versions are immutable; a new edit creates the next version. Active work always references the version at assignment time.

**US-013** As a Client Admin, I can view all published SOPs for a service category when creating a project so I know what the worker will execute.

### 5.3 Project Management

> Replaces Job Orders. A *Job Order* is now a Project of type "site visit". See [PROJECTS_DESIGN.md](PROJECTS_DESIGN.md).

**US-020** As a Client Admin, I can create a Project specifying: title, location/scope, service category, **one or more fulfilling vendors**, scheduled dates, and a **pricing model** — one of per-unit commission (with unit rate), recurring retainer (amount per worker per period), flat fee, or milestone (with milestone amounts).

**US-021** As a Client Admin, I can attach an optional SOP checklist and evidence requirements to a project's outcomes (e.g. a sale requires a receipt photo, serial number, and GPS).

**US-022** As a Client Admin, I can view all my projects in a filterable list (by status, vendor, pricing model, date range, service category) and drill into per-vendor outcomes and accrued payout.

**US-023** As a Vendor Admin, I can see projects my organization is engaged on and assign my workers to them.

**US-024** As a Client or Vendor Admin, I receive notifications for project lifecycle and outcome events (assigned, reported, disputed, confirmed).

**US-025** As a Super Admin, I can view all projects across all organizations.

**US-026** As a Client Admin, I can view a **per-project Sales Report** — units sold, sales value (GMV), conversion funnel, sales-over-time, and breakdowns by **vendor / region / category** — and **export** it (PDF/CSV) for stakeholders.

### 5.4 Mobile — Task Execution & Outcome Reporting

**US-030** As a Worker, I can view all my assigned work for today and upcoming days without internet connection (offline-first).

**US-031** As a Worker, I can open a project visit and see all SOP steps in order with their instructions.

**US-032** As a Worker, I can mark a step as complete and upload photo evidence — even when offline.

**US-033** As a Worker, I can add a text note to any step for exceptions or observations.

**US-034** As a Worker, I cannot skip mandatory steps — the app enforces SOP order where configured by the step.

**US-035** As a Worker, when connectivity is restored, all offline completions sync automatically in the background.

**US-036** As a Worker on a commission or milestone project, I can log a repeatable outcome (e.g. a sale) with its required evidence — offline — and it syncs like a step completion (device wins on conflict).

### 5.5 Payouts & Verification

**US-040** A vendor's **worker** reports `visit`/`sale` outcomes (mobile) and the **Vendor Admin** reports `milestone`/`flat` (web), each with required evidence; `retainer_period` accrues automatically. Each confirmed outcome creates a payout accrual **for the vendor** (the reporter is operational; money attributes to the vendor).

**US-041** As a Client Admin, I **monitor** reported outcomes and may **dispute** one (with a reason) within the **7-day** window; outcomes I don't action **auto-confirm**. Monitoring is the norm — there is no per-outcome approval step; dispute is the exception.

**US-042** As a Vendor Admin, I can see my organization's accruals (reported / confirmed / disputed / paid), grouped by project and period.

**US-043** As a Super Admin, I export payout reports as CSV per vendor per period (accrual line items) for reconciliation, and I am the **final arbiter** of disputes (resolution SLA: 3 business days).

**US-044** Payout statements aggregate confirmed accruals **per vendor per period** and move through pending → approved → paid; the **vendor pays its workers internally**.

**US-045** Pricing terms (unit rate, retainer, flat, milestone amounts) are **locked at project creation**; changes apply only to future outcomes — never retroactively to already-accrued ones.

### 5.6 Push Notifications

**US-050** As a Worker, I receive a push notification when I am assigned to a project.

**US-051** As a Worker, I receive a reminder notification 1 hour before a scheduled visit.

**US-052** As a Client Admin, I receive a notification when an outcome is reported for my project.

**US-053** As a Vendor Admin, I receive a notification when a client disputes an outcome.

**US-054** All notifications are stored in an in-app notification center as fallback.

### 5.7 Worker Management

> Worker records are **vendor-internal** (vendor + super_admin only; never clients). Workers are **not rostered to projects** and not a platform payout unit — the vendor uses this roster for its own ops and the mobile field app.

**US-060** As a Vendor Admin, I can maintain a worker's profile — contact, region, pay rate, and **skills (service categories)** — for my own roster.

**US-061** As a Vendor Admin, I can find and filter my workers by **skill and availability** for my own scheduling.

**US-062** As a Vendor Admin, I can set each worker's **availability** (available / limited / on leave) and keep notes.

**US-063** As a Vendor Admin, I assign my workers to field work and leads **internally** — the platform does not manage worker↔project assignment or pay workers.

**US-064** As a Super Admin, I can view worker records across vendors for oversight; **Client Admins cannot** view individual worker records.

---

## 6. Functional Requirements

### 6.1 Authentication & Authorization

- JWT-based authentication (access token 1h, refresh token 30d)
- Roles: `super_admin`, `client_admin`, `vendor_admin`, `worker`
- Data isolation: clients see only their own projects; vendors see only their workers' work
- Workers see only work assigned to them
- All API endpoints require a valid JWT except `/auth/login` and `/auth/refresh`

### 6.2 Service Category & SOP Rules

- Service categories are platform-level (created by super_admin only)
- Each SOP template belongs to exactly one service category
- Multiple SOP versions can exist; only one is `published` at a time per category
- When a project (or visit outcome) is created, the active SOP version's steps are snapshotted — future SOP edits do not affect it

### 6.3 Sync Requirements

- Mobile app must be fully functional with no internet for up to 7 days
- Sync must resume automatically when connectivity is restored
- Photos must survive app restarts before successful upload
- Conflict resolution: server wins for projects/SOPs/pricing, device wins for step completions and worker-reported outcomes
- Sync must complete within 10 seconds on 3G connectivity for a standard payload

### 6.4 Photo Evidence

- Accepted formats: JPG, PNG
- Max size: 10MB per photo
- Up to 5 photos per SOP step / outcome
- Photos stored with metadata: worker ID, step/outcome ID, GPS coordinates, timestamp
- Direct upload to Cloudflare R2 via presigned URL (bypasses API server)

### 6.5 Performance

- API response time: < 300ms p95 for non-file endpoints
- Dashboard loads within 2 seconds on standard broadband
- Mobile app cold start: < 3 seconds
- Push notification delivery: < 5 seconds after trigger event

### 6.6 Project, Pricing & Verification Rules

- A project has exactly **one pricing model**: `per_unit | retainer | flat | milestone`, **uniform across its vendors**.
- A project is `client_org → one or more vendor_orgs` (many-to-many). Workers are **vendor-internal** — not rostered to projects and not a platform payout unit; **outcomes & payouts attribute to the vendor**.
- Outcome reporters by type: `visit`/`sale` → vendor's worker (mobile); `milestone`/`flat` → vendor admin (web); `retainer_period` → system. The reporter is operational; money attributes to the vendor.
- Outcomes are **self-reported** with mandatory **per-project evidence rules**. Clients **monitor**; an outcome **auto-confirms after 7 days** unless **disputed** — no per-outcome client approval.
- Confirmed outcomes accrue payout to the **vendor**, rolled up **per vendor per period**. **Retainer** periods accrue **unconditionally** (the vendor's declared headcount; no timesheet gating in v1).
- No budget cap or alert in v1; the financial control is **verification** (evidence + audit/dispute). A capped incentive pool is a possible later phase.
- Disputes are arbitrated by **super_admin**, who has final say (resolution SLA: 3 business days).
- Pricing terms are locked at project creation; edits never apply retroactively to accrued outcomes.

### 6.7 Worker & Staffing Rules

- Worker records (profile, skills, availability) are **vendor-internal** — a roster for the vendor's own ops and to run the mobile field app; visible to the vendor + super_admin only, never to clients.
- Workers are **not rostered to projects** and are **not a platform payout unit**. The vendor distributes work and **pays its workers internally** (off-platform).
- In client-facing views, outcomes are attributed to the **vendor** — clients never see individual workers.
- **Out of scope (v1):** employment/compliance document management, per-worker platform payouts, performance analytics — later.

### 6.8 Scheduled Jobs

- **Auto-confirm:** outcomes past the audit window with no open dispute are auto-confirmed and accrued.
- **Retainer generation:** each period, one `retainer_period` outcome per active retainer project; amount = `retainer_amount × the vendor's placed headcount` (unconditional).
- **Statement roll-up:** confirmed accruals roll into per-(vendor, period) statements at period close.
- **Visit reminders:** 1 hour before scheduled visits (US-051).

---

## 7. Non-Functional Requirements

| Requirement | Target |
|---|---|
| Availability | 99.5% uptime (MVP) |
| Data retention | Project & outcome records retained indefinitely |
| Evidence (photo) retention | 2 months (v1) — covers the 7-day audit window; ⚠ purged before later audits |
| Data privacy | Clients cannot access vendor-internal data and vice versa |
| Audit trail | Outcome status transitions (report/confirm/dispute) are append-only |
| Mobile OS support | iOS 14+, Android 10+ |
| Web browser support | Chrome, Safari, Firefox (last 2 versions) |

---

## 8. Success Metrics (MVP)

| Metric | Target (90 days post-launch) |
|---|---|
| Client orgs onboarded | 3 paying clients |
| Vendor orgs onboarded | 5 vendors |
| Projects active on platform | 20+ |
| Outcomes reported & confirmed | 500+ |
| Sync failure rate | < 1% |
| Push notification delivery rate | > 95% |
| Worker app crash rate | < 0.5% |

---

## 9. Release Milestones

| Milestone | Scope | Target |
|---|---|---|
| M1 — Foundation | Auth, client & vendor org management, user roles | Week 2 |
| M2 — SOP Engine | Service categories, SOP builder, versioning | Week 4 |
| M3 — Job Dispatch | Job orders, assignment, per-job payout, notifications | Week 6 |
| M4 — Mobile MVP | Offline task execution, photo upload, sync | Week 10 |
| M5 — Dashboard | Client & vendor web dashboards, real-time updates | Week 12 |
| M6 — Payouts | Payout tracking, approval flow, CSV export | Week 14 |
| M7 — Hardening | Performance, monitoring, security audit | Week 16 |
| **M8a — Projects: schema + backfill** | Additive tables; jobs→projects backfill; legacy REST + sync adapters (no new UI) | Post-MVP |
| **M8b — Projects: per-unit + flat** | Project builder, evidence rules, report/monitor/dispute, per-vendor accrual payouts + statements, scheduler | Post-MVP |
| **M8c — Projects: retainer + milestone** | Remaining pricing models; deprecate legacy job orders | Post-MVP |

> M1–M7 are the existing MVP (job-order model, already built/in-progress). **M8** evolves it to the Projects model — phased per [PROJECTS_DESIGN.md](PROJECTS_DESIGN.md) §11.
