# Grand Design — Sigma Marketplace

**Version:** 1.1  
**Date:** 2026-06-30  
**Note:** updated for the v2.0 Projects model — see [PROJECTS_DESIGN.md](PROJECTS_DESIGN.md).

---

## 1. System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SIGMA MARKETPLACE                            │
│                                                                     │
│  ┌──────────────────┐    ┌──────────────────┐   ┌───────────────┐  │
│  │  Worker App  │    │ Client Dashboard │   │ Admin Panel   │  │
│  │ (React Native +  │    │   (Next.js)      │   │  (Next.js)    │  │
│  │   Expo)          │    │                  │   │               │  │
│  └────────┬─────────┘    └────────┬─────────┘   └───────┬───────┘  │
│           │                       │                     │           │
│           └───────────────────────┴─────────────────────┘           │
│                                   │                                 │
│                          ┌────────▼────────┐                        │
│                          │   Go REST API   │                        │
│                          │  (sigma-api)    │                        │
│                          └────────┬────────┘                        │
│                                   │                                 │
│          ┌────────────────────────┼────────────────────┐            │
│          │                        │                    │            │
│  ┌───────▼──────┐     ┌───────────▼──────┐  ┌─────────▼──────┐    │
│  │  PostgreSQL  │     │  Cloudflare R2   │  │  Upstash Redis │    │
│  │  (Supabase)  │     │  (Photo Storage) │  │  (Notif Queue) │    │
│  └──────────────┘     └──────────────────┘  └────────────────┘    │
│                                                                     │
│                        ┌──────────────────┐                         │
│                        │   Expo Push API  │                         │
│                        │  (APNs + FCM)    │                         │
│                        └──────────────────┘                         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Component Architecture

### 2.1 sigma-api (Go Backend)

Single deployable Go binary. Monolith architecture at MVP — split into services only when a clear bottleneck is identified.

```
sigma-api/
├── cmd/
│   └── server/          # entrypoint
├── internal/
│   ├── auth/            # JWT issue/validate, refresh tokens
│   ├── organization/    # org & user management
│   ├── sop/             # SOP templates, steps, versioning
│   ├── project/         # projects, agreements, milestones, worker assignment (v2.0)
│   ├── outcome/         # outcome reporting, verification, disputes (v2.0)
│   ├── worker/          # worker profiles, skills, capacity (v2.0)
│   ├── job/             # legacy job orders — adapter over project/outcome during migration
│   ├── sync/            # WatermelonDB sync protocol endpoints
│   ├── photo/           # R2 presigned URL generation
│   ├── payout/          # accruals, statements, reports (v2.0)
│   ├── scheduler/       # auto-confirm, retainer generation, statement roll-up (v2.0)
│   └── notification/    # push dispatch, delivery tracking
├── pkg/
│   ├── middleware/      # JWT auth, rate limit, CORS
│   ├── db/              # PostgreSQL connection, migrations
│   ├── r2/              # Cloudflare R2 client
│   ├── push/            # Expo Push API client
│   └── redis/           # Upstash Redis client
└── migrations/          # SQL migration files
```

**Key patterns:**
- Repository pattern — all DB queries behind interfaces (testable)
- Service layer — business logic isolated from transport
- Chi router — lightweight, idiomatic Go
- sqlc — type-safe SQL query generation
- golang-migrate — schema versioning

### 2.2 sigma-mobile (React Native + Expo)

Offline-first. All job data stored locally in WatermelonDB (SQLite). Syncs with Go API when connectivity exists.

```
sigma-mobile/
├── app/                 # Expo Router file-based routes
│   ├── (auth)/          # login, register screens
│   ├── (tabs)/
│   │   ├── jobs/        # job list, job detail, SOP execution
│   │   ├── earnings/    # payout history
│   │   └── profile/     # settings, notifications
├── components/          # reusable UI components
├── db/
│   ├── schema.ts        # WatermelonDB schema
│   ├── models/          # Job, SopStep, StepCompletion, SyncQueue
│   └── sync/            # pull/push sync adapter
├── services/
│   ├── api.ts           # Go API client (axios)
│   ├── sync.ts          # sync orchestrator
│   ├── photo.ts         # capture, compress, queue upload
│   └── notifications.ts # Expo push token, handlers
├── store/               # Zustand global state (auth, connectivity)
└── tasks/               # Expo TaskManager background jobs
```

**Key patterns:**
- WatermelonDB with custom sync adapter for Go backend
- NetInfo listener → triggers sync on reconnect
- Expo TaskManager → background sync triggered by silent push
- Expo FileSystem → persist photos before upload
- Direct R2 upload via presigned URLs

### 2.3 sigma-web (Next.js)

Two logical apps in one Next.js codebase, separated by route groups.

```
sigma-web/
├── app/
│   ├── (auth)/          # login page
│   ├── (client)/        # client admin area
│   │   ├── dashboard/   # overview metrics
│   │   ├── jobs/        # job list, create, detail
│   │   ├── workers/ # team management
│   │   ├── sop/         # SOP builder
│   │   └── payouts/     # earnings reports
│   └── (admin)/         # super admin area
│       ├── organizations/
│       ├── sop-library/
│       └── payouts/
├── components/
│   ├── ui/              # shadcn/ui base components
│   ├── job/             # job-specific components
│   └── sop/             # SOP builder components
├── lib/
│   ├── api.ts           # Go API client
│   └── supabase.ts      # Supabase Realtime client
└── hooks/               # useJobRealtime, useNotifications, etc.
```

---

## 3. Data Flow

### 3.1 Job Assignment Flow

```
Client Admin (Web)                 Go API              Worker (Mobile)
       │                              │                        │
       │── POST /job-orders ─────────►│                        │
       │                              │── Save to PostgreSQL   │
       │                              │── Publish to Redis     │
       │                              │       notif queue      │
       │                              │                        │
       │                              │◄─ Notif worker picks up│
       │                              │── Expo Push API ──────►│ (silent push)
       │                              │                        │── Wake app
       │                              │                        │── GET /sync/pull
       │                              │◄───────────────────────│
       │                              │── Return new job ─────►│
       │                              │                        │── Store in SQLite
       │                              │── Expo Push API ──────►│ (visible push)
       │                              │                        │ "New job: Site XYZ"
```

### 3.2 Offline Task Completion & Sync Flow

```
Worker (Mobile — No Internet)
       │
       │── Open job from local SQLite
       │── Execute SOP steps
       │── Mark step complete → write to local SQLite
       │── Capture photo → save to device filesystem
       │── Add to sync_queue: { type: "step_completion", ... }
       │── Add to sync_queue: { type: "photo_upload", ... }
       │
       [Internet restored — NetInfo event]
       │
       │── Start sync cycle
       │   ├── PUSH: POST /sync/push (step completions)
       │   ├── GET presigned URL from /photos/presign
       │   ├── PUT photo directly to Cloudflare R2
       │   ├── PATCH /step-completions/:id { photoKey }
       │   └── PULL: GET /sync/pull?lastPulledAt=xxx
       │
       │── Update lastPulledAt in SQLite
       │── Delete uploaded photos from device filesystem
```

### 3.3 Payout Accrual Flow (v2.0)

```
Outcome reported (sale/visit/milestone/flat)         Retainer period (scheduler)
       │  +evidence                                          │ auto-generated
       ▼                                                     ▼ auto-confirmed
Verification: client monitors; dispute = exception          │
       │  auto-confirm after audit_window_days               │
       ▼                                                     │
Outcome CONFIRMED ◄──────────────────────────────────────────┘
       │  amount = pricing rule (unit_rate×qty | retainer | flat | milestone.amount)
       ▼
payout_accrual { project, outcome, worker, period, amount }
       │  (period close, scheduler)
       ▼
payout_statement per (vendor, worker, period)   status: pending
       │
       ▼
Client/Admin: export CSV · approve · mark paid (manual transfer, MVP)
```

---

## 4. Sync Protocol Detail

The mobile app implements the WatermelonDB sync protocol against two Go endpoints.

### Pull endpoint

```
GET /sync/pull?lastPulledAt=<unix_timestamp>

Response 200:
{
  "changes": {
    "job_orders": {
      "created": [ { ...jobFields } ],
      "updated": [ { ...jobFields } ],
      "deleted": [ "id1", "id2" ]
    },
    "sop_steps": {
      "created": [...],
      "updated": [...],
      "deleted": []
    },
    "step_completions": {
      "created": [...],   // completions from other devices (future multi-device)
      "updated": [...],
      "deleted": []
    }
  },
  "timestamp": 1716134000
}
```

### Push endpoint

```
POST /sync/push?lastPulledAt=<unix_timestamp>

Body:
{
  "changes": {
    "outcomes": {
      "created": [ { id, project_id, type, quantity, occurred_at, evidence_keys } ],
      "updated": [], "deleted": []
    },
    "step_completions": {
      "created": [ { id, outcome_id, sop_step_id, completed_at, note, photo_key } ],
      "updated": [],
      "deleted": []
    }
  }
}

Response 200: {}
Response 409: { "error": "conflict", "conflicts": [...] }
```

### Conflict resolution matrix

| Entity | Winner | Rationale |
|---|---|---|
| projects / pricing | Server | Client/admin controls the engagement |
| sop_steps | Server | Read-only on device |
| step_completions | Device | Worker is source of truth |
| outcomes (worker-reported) | Device | Worker is source of truth |
| outcome status: CONFIRMED | Most advanced | Cannot regress a confirmed/paid outcome |

---

## 5. Multi-Tenancy Design

All tables include `organization_id`. Enforced at two levels:

1. **API middleware** — JWT claims carry `organization_id`; injected into every query
2. **PostgreSQL RLS** — Row Level Security policies as defense-in-depth

```sql
-- Example RLS policy (a project is visible to its client or any participating vendor)
CREATE POLICY org_isolation ON projects
  USING (client_org_id = current_setting('app.organization_id')::uuid
      OR id IN (SELECT project_id FROM project_vendors
                WHERE vendor_org_id = current_setting('app.organization_id')::uuid));
-- worker_profiles / worker_skills: vendor-only (clients excluded) — see PROJECTS_DESIGN §8.1.
```

Super Admin role bypasses RLS via a separate DB role.

---

## 6. Push Notification Architecture

```
Business Event (job assigned, step complete, deadline approaching)
          │
          ▼
Go notification.Service.Publish(event)
          │
          ▼
Upstash Redis Queue (LPUSH notifications:queue)
          │
          ▼
Notification Worker (goroutine, BRPOP)
  ├── Look up user's expo_push_token
  ├── Build Expo push message
  ├── POST to Expo Push API
  ├── Handle receipt:
  │     ├── ok      → mark notification delivered
  │     ├── DeviceNotRegistered → delete token, retry with fallback
  │     └── error   → exponential backoff retry (max 3 attempts)
  └── Save to notifications table (in-app center)
```

**Silent push for background sync:**
- Sent immediately after job assignment
- Wakes Expo TaskManager on device
- App pulls new data before user sees visible notification
- Visible push follows 2–3 seconds later

---

## 7. Infrastructure & Deployment

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     Vercel      │    │    Railway      │    │    Supabase     │
│  (sigma-web)    │    │  (sigma-api)    │    │  (PostgreSQL)   │
│  Next.js SSR    │    │  Go binary      │    │  + Auth         │
│  Free tier      │    │  ~$7/month      │    │  Free tier      │
└────────┬────────┘    └────────┬────────┘    └────────┬────────┘
         │                      │                      │
         └──────────────────────┴──────────────────────┘
                                │
                    ┌───────────┼───────────┐
                    │                       │
         ┌──────────▼──────┐    ┌───────────▼──────┐
         │  Cloudflare R2  │    │  Upstash Redis   │
         │  Photo storage  │    │  Notif queue     │
         │  Free 10GB/mo   │    │  Free tier       │
         └─────────────────┘    └──────────────────┘
```

**CI/CD:**
- GitHub Actions — test + build on PR
- Vercel auto-deploy on push to `main`
- Railway auto-deploy via GitHub integration
- EAS Build for mobile (GitHub Actions trigger)

---

## 8. Security Considerations

| Concern | Mitigation |
|---|---|
| Tenant data isolation | organization_id on all tables + RLS |
| JWT security | Short-lived access tokens (1h) + secure refresh |
| Photo access | R2 objects not publicly accessible — served via signed URLs |
| API rate limiting | Per-IP and per-user rate limit via Redis |
| Input validation | All API inputs validated with Go validator |
| SQL injection | sqlc generates type-safe queries — no string interpolation |
| Mobile token storage | Expo SecureStore (Keychain on iOS, Keystore on Android) |

---

## 9. Monitoring & Observability

| Tool | Purpose | Cost |
|---|---|---|
| Railway built-in metrics | CPU, memory, request rate | Included |
| Sentry | Error tracking (API + mobile) | Free tier |
| Upstash Redis metrics | Queue depth, processing rate | Included |
| Supabase dashboard | Query performance, DB size | Included |
| GitHub Actions | CI pass/fail | Included |
