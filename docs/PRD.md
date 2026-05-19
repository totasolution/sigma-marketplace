# Product Requirements Document
# Sigma Marketplace

**Version:** 1.0  
**Date:** 2026-05-19  
**Status:** Draft  
**Owner:** Totasolution

---

## 1. Executive Summary

Sigma Marketplace is a B2B field service platform that digitizes the end-to-end workflow of dispatching managed technicians to client sites. Organizations onboard their technician workforce, define SOPs per service type, assign jobs to sites, and pay technicians per completed task — all within a single platform.

The core value proposition:
- **For clients:** full visibility and control over field operations with structured, auditable SOP execution
- **For technician managers:** streamlined job dispatch, real-time progress tracking, and automated payout calculation
- **For technicians:** clear task lists, offline-capable mobile app, transparent earnings

---

## 2. Problem Statement

Field service operations today are fragmented:
- Job assignments happen over WhatsApp/email — no audit trail
- SOPs exist as PDF documents — compliance is unchecked
- Technician pay is manually calculated each month from spreadsheets
- Clients have zero real-time visibility into site work progress
- Evidence (photos) are stored in personal WhatsApp or Google Drive with no structure

Sigma replaces this with a structured, trackable, mobile-first platform.

---

## 3. Target Users

### 3.1 Primary Users

| Role | Description | Platform |
|---|---|---|
| **Technician** | Field worker executing site tasks | Mobile (primary), offline-capable |
| **Client Admin** | Manages technicians, creates jobs, reviews progress | Web dashboard |
| **Super Admin** | Platform operator, manages orgs, SOPs, payouts | Web admin panel |

### 3.2 User Personas

**Budi — Network Technician**
- 28 years old, field tech for 4 years
- Works at 3–5 sites per week
- Limited connectivity at remote sites
- Needs: simple task checklist, photo upload, clear daily earnings

**Rina — Client Operations Manager**
- 35 years old, manages 20 technicians
- Needs to assign jobs, monitor progress, approve payouts
- Wants: dashboard overview, exception alerts, monthly reports

**Platform Admin**
- Internal Totasolution staff
- Manages client onboarding, global SOP library, payment reconciliation

---

## 4. Scope

### In Scope (MVP)

- Organization & user management (multi-tenant)
- SOP template builder with ordered steps
- Job order creation & technician assignment
- Mobile app: offline task execution with photo evidence
- Real-time job progress on client web dashboard
- Per-task payout rate configuration
- Earnings report per technician per period
- Push notifications (job assigned, task updates)
- Offline sync with conflict resolution

### Out of Scope (Post-MVP)

- Direct payment disbursement (bank transfer, e-wallet) — Phase 2
- Customer-facing marketplace (public technician discovery) — Phase 3
- IoT/sensor integration — Phase 3
- AI-based SOP compliance checking — Future

---

## 5. Core Features & User Stories

### 5.1 Organization & User Management

**US-001** As a Super Admin, I can create and manage client organizations so that each B2B client is isolated in their own workspace.

**US-002** As a Client Admin, I can invite technicians to my organization with a role of `technician` so they can receive job assignments.

**US-003** As a Client Admin, I can deactivate a technician account without losing their historical job records.

**US-004** As any user, I can log in with email/password and receive a JWT so I can access the platform securely.

### 5.2 SOP Template Management

**US-010** As a Super Admin, I can create SOP templates with an ordered list of steps so clients can reuse standard procedures.

**US-011** As a Client Admin, I can clone a global SOP template and customize it for my organization.

**US-012** As a Client Admin, I can configure a per-step payout rate so technicians are compensated accurately for each task.

**US-013** As a Super Admin, I can version SOPs — active jobs reference the version at assignment time, not the latest.

### 5.3 Job Order Management

**US-020** As a Client Admin, I can create a Job Order specifying: site name, location, assigned technician, SOP template, and scheduled date.

**US-021** As a Client Admin, I can view all job orders in a filterable list (by status, technician, date range).

**US-022** As a Super Admin, I can view all job orders across all organizations.

**US-023** As a Client Admin, I receive a notification when a technician starts or completes a job.

### 5.4 Mobile — Task Execution

**US-030** As a Technician, I can view my assigned jobs for today and upcoming days without internet connection (offline-first).

**US-031** As a Technician, I can open a job and see all SOP steps in order with their payout value per step.

**US-032** As a Technician, I can mark a step as complete and upload photo evidence — even when offline.

**US-033** As a Technician, I can add a text note to any step for exceptions or observations.

**US-034** As a Technician, I cannot skip mandatory steps — the app enforces SOP order where configured.

**US-035** As a Technician, when connectivity is restored, all offline completions sync automatically in the background.

### 5.5 Earnings & Payouts

**US-040** As a Technician, I can see my total earnings for the current week and month broken down by job.

**US-041** As a Client Admin, I can see a payout report per technician for a given period.

**US-042** As a Super Admin, I can export payout reports as CSV for manual payment processing.

**US-043** Payout amounts are locked once a job is marked complete — retroactive SOP rate changes don't affect past jobs.

### 5.6 Push Notifications

**US-050** As a Technician, I receive a push notification when a new job is assigned to me.

**US-051** As a Technician, I receive a reminder notification 1 hour before a scheduled job.

**US-052** As a Client Admin, I receive a push/email notification when a technician completes a job.

**US-053** All notifications are stored in an in-app notification center as fallback.

---

## 6. Functional Requirements

### 6.1 Authentication & Authorization

- JWT-based authentication (access token 1h, refresh token 30d)
- Roles: `super_admin`, `client_admin`, `technician`
- Row-level data isolation per organization
- All API endpoints require a valid JWT except `/auth/login` and `/auth/refresh`

### 6.2 Sync Requirements

- Mobile app must be fully functional with no internet for up to 7 days
- Sync must resume automatically when connectivity is restored
- Photos must survive app restarts before successful upload
- Conflict resolution: server wins for jobs/SOPs, device wins for step completions
- Sync must complete within 10 seconds on 3G connectivity for a standard job payload

### 6.3 Photo Evidence

- Accepted formats: JPG, PNG
- Max size: 10MB per photo
- Up to 5 photos per SOP step
- Photos stored with metadata: technician ID, step ID, GPS coordinates, timestamp
- Direct upload to Cloudflare R2 via presigned URL (bypasses API server)

### 6.4 Performance

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
| GDPR / data privacy | PII minimized, org-scoped data isolation |
| Mobile OS support | iOS 14+, Android 10+ |
| Web browser support | Chrome, Safari, Firefox (last 2 versions) |

---

## 8. Success Metrics (MVP)

| Metric | Target (90 days post-launch) |
|---|---|
| Orgs onboarded | 3 paying clients |
| Jobs completed on platform | 500+ |
| Sync failure rate | < 1% |
| Push notification delivery rate | > 95% |
| Technician app crash rate | < 0.5% |
| Payout calculation accuracy | 100% |

---

## 9. Release Milestones

| Milestone | Scope | Target |
|---|---|---|
| M1 — Foundation | Auth, org management, user roles | Week 2 |
| M2 — SOP Engine | SOP builder, versioning, payout rates | Week 4 |
| M3 — Job Dispatch | Job orders, assignment, notifications | Week 6 |
| M4 — Mobile MVP | Offline task execution, photo upload, sync | Week 10 |
| M5 — Dashboard | Client web dashboard, real-time updates | Week 12 |
| M6 — Payouts | Earnings reports, CSV export | Week 14 |
| M7 — Hardening | Performance, monitoring, security audit | Week 16 |
