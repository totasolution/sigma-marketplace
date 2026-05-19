# Product Requirements Document
# Sigma Marketplace

**Version:** 1.1  
**Date:** 2026-05-19  
**Status:** Draft  
**Owner:** Totasolution

---

## 1. Executive Summary

Sigma Marketplace is a B2B field service platform that connects **clients** (organizations that need field work done) with **vendors** (businesses that manage worker workforces). Clients create job orders for a specific service category, select a worker from a partner vendor, and track execution in real time. Workers follow structured SOPs defined per service category and complete tasks with photo evidence — even offline.

The core value proposition:
- **For clients:** full visibility into field operations with structured, auditable SOP execution per service type
- **For vendors:** centralized worker management, clear job dispatch, and automated payout tracking per job
- **For workers:** clear task checklists, offline-capable mobile app, and transparent job history

---

## 2. Problem Statement

Field service operations today are fragmented:
- Job assignments happen over WhatsApp/email — no audit trail
- SOPs exist as PDF documents — compliance is unchecked
- Worker pay is manually calculated each month from spreadsheets
- Clients have zero real-time visibility into site work progress
- Evidence (photos) are stored in personal WhatsApp or Google Drive with no structure
- No clear separation between who requests work (client) and who supplies the workforce (vendor)

Sigma replaces this with a structured, trackable, mobile-first platform with a clean client–vendor separation.

---

## 3. Target Users

### 3.1 Primary Users

| Role | Organization | Description | Platform |
|---|---|---|---|
| **Worker** | Vendor | Field worker executing site tasks | Mobile (primary), offline-capable |
| **Vendor Admin** | Vendor | Manages worker roster, views job assignments, tracks payouts | Web dashboard |
| **Client Admin** | Client | Creates job orders, assigns workers, monitors progress | Web dashboard |
| **Super Admin** | Platform | Manages all orgs, service categories, SOP library, platform-wide reports | Web admin panel |

### 3.2 User Personas

**Budi — Network Worker (Vendor)**
- 28 years old, field tech for 4 years, employed by PT. Sigma Teknik (vendor)
- Works at 3–5 client sites per week
- Limited connectivity at remote sites
- Needs: simple task checklist, photo upload, view of assigned jobs

**Andi — Vendor Operations Manager**
- 33 years old, runs a team of 15 workers at PT. Sigma Teknik
- Needs to see all workers' assignments, track completion rates, reconcile payouts monthly
- Wants: worker roster, job history per tech, payout summary

**Rina — Client Operations Manager**
- 35 years old, IT infrastructure manager at a telecom company (client)
- Creates job orders for network installations and site surveys
- Wants: assign jobs to trusted vendor workers, real-time progress, photo evidence

**Platform Admin**
- Internal Totasolution staff
- Manages client and vendor onboarding, global service category library, SOP management

---

## 4. Scope

### In Scope (MVP)

- Organization management: clients and vendors as distinct org types
- Client–vendor relationship management
- User management: client_admin, vendor_admin, worker roles
- Service category management (platform-level)
- SOP template management: per service category, versioned, with ordered steps
- Job order creation by clients, assignment to vendor workers
- Mobile app: offline task execution with photo evidence per SOP step
- Real-time job progress on client and vendor web dashboards
- Per-job payout tracking (flat amount per job order)
- Push notifications (job assigned, task updates, completion)
- Offline sync with conflict resolution

### Out of Scope (Post-MVP)

- Direct payment disbursement (bank transfer, e-wallet) — Phase 2
- Open marketplace (vendors/workers discoverable by any client) — Phase 3
- IoT/sensor integration — Phase 3
- AI-based SOP compliance checking — Future
- Worker rating/review system — Phase 2

---

## 5. Core Features & User Stories

### 5.1 Organization & User Management

**US-001** As a Super Admin, I can create and manage client organizations and vendor organizations independently so that each party has appropriate access.

**US-002** As a Super Admin, I can establish a client–vendor relationship so that a client can assign jobs to workers from a specific vendor.

**US-003** As a Vendor Admin, I can invite and manage workers within my vendor organization so they can receive job assignments.

**US-004** As a Vendor Admin, I can deactivate a worker without losing their historical job records.

**US-005** As any user, I can log in with email/password and receive a JWT so I can access the platform securely.

### 5.2 Service Categories & SOP Management

**US-010** As a Super Admin, I can create service categories (e.g. "LAN Installation", "Fiber Optic Splicing", "Site Survey") so that SOPs are organized by type of work.

**US-011** As a Super Admin, I can create and publish SOP templates under a service category, with an ordered list of steps, photo requirements, and order enforcement rules.

**US-012** As a Super Admin, I can version SOPs — published versions are immutable; a new edit creates the next version. Active jobs always reference the version at assignment time.

**US-013** As a Client Admin, I can view all published SOPs for a service category when creating a job order so I know what the worker will execute.

### 5.3 Job Order Management

**US-020** As a Client Admin, I can create a Job Order specifying: site name, location, service category, SOP template version, assigned worker (from a partner vendor), scheduled date, and payout amount for the job.

**US-021** As a Client Admin, I can view all my job orders in a filterable list (by status, vendor, worker, date range, service category).

**US-022** As a Vendor Admin, I can view all job orders assigned to my workers across all clients.

**US-023** As a Client Admin, I receive a notification when a worker starts or completes a job.

**US-024** As a Vendor Admin, I receive a notification when one of my workers is assigned a new job by a client.

**US-025** As a Super Admin, I can view all job orders across all organizations.

### 5.4 Mobile — Task Execution

**US-030** As a Worker, I can view all my assigned jobs for today and upcoming days without internet connection (offline-first).

**US-031** As a Worker, I can open a job and see all SOP steps in order with their instructions.

**US-032** As a Worker, I can mark a step as complete and upload photo evidence — even when offline.

**US-033** As a Worker, I can add a text note to any step for exceptions or observations.

**US-034** As a Worker, I cannot skip mandatory steps — the app enforces SOP order where configured by the step.

**US-035** As a Worker, when connectivity is restored, all offline completions sync automatically in the background.

### 5.5 Payouts

**US-040** As a Client Admin, I can set a payout amount when creating a job order so the vendor knows what will be paid on completion.

**US-041** As a Vendor Admin, I can see all payouts (pending and paid) for jobs completed by my workers.

**US-042** As a Super Admin, I can export payout reports as CSV per vendor per period for reconciliation.

**US-043** As a Client Admin, I can mark a job payout as approved once I am satisfied with the completed work.

**US-044** Payout amounts are locked once a job is created — they cannot be changed retroactively after the job starts.

### 5.6 Push Notifications

**US-050** As a Worker, I receive a push notification when a new job is assigned to me.

**US-051** As a Worker, I receive a reminder notification 1 hour before a scheduled job.

**US-052** As a Client Admin, I receive a notification when a worker completes a job.

**US-053** As a Vendor Admin, I receive a notification when a client assigns a job to one of my workers.

**US-054** All notifications are stored in an in-app notification center as fallback.

---

## 6. Functional Requirements

### 6.1 Authentication & Authorization

- JWT-based authentication (access token 1h, refresh token 30d)
- Roles: `super_admin`, `client_admin`, `vendor_admin`, `worker`
- Data isolation: clients see only their own jobs; vendors see only their workers' jobs
- Workers see only jobs assigned to them
- All API endpoints require a valid JWT except `/auth/login` and `/auth/refresh`

### 6.2 Service Category & SOP Rules

- Service categories are platform-level (created by super_admin only)
- Each SOP template belongs to exactly one service category
- Multiple SOP versions can exist; only one can be `published` at a time per category
- When a job is created, the active SOP version's steps are snapshotted into the job — future SOP edits do not affect it

### 6.3 Sync Requirements

- Mobile app must be fully functional with no internet for up to 7 days
- Sync must resume automatically when connectivity is restored
- Photos must survive app restarts before successful upload
- Conflict resolution: server wins for jobs/SOPs, device wins for step completions
- Sync must complete within 10 seconds on 3G connectivity for a standard job payload

### 6.4 Photo Evidence

- Accepted formats: JPG, PNG
- Max size: 10MB per photo
- Up to 5 photos per SOP step
- Photos stored with metadata: worker ID, step ID, GPS coordinates, timestamp
- Direct upload to Cloudflare R2 via presigned URL (bypasses API server)

### 6.5 Performance

- API response time: < 300ms p95 for non-file endpoints
- Dashboard loads within 2 seconds on standard broadband
- Mobile app cold start: < 3 seconds
- Push notification delivery: < 5 seconds after trigger event

---

## 7. Non-Functional Requirements

| Requirement | Target |
|---|---|
| Availability | 99.5% uptime (MVP) |
| Data retention | Job records retained indefinitely |
| Photo retention | 2 years |
| Data privacy | Clients cannot access vendor-internal data and vice versa |
| Mobile OS support | iOS 14+, Android 10+ |
| Web browser support | Chrome, Safari, Firefox (last 2 versions) |

---

## 8. Success Metrics (MVP)

| Metric | Target (90 days post-launch) |
|---|---|
| Client orgs onboarded | 3 paying clients |
| Vendor orgs onboarded | 5 vendors |
| Jobs completed on platform | 500+ |
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
