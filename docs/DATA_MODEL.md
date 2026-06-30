# Data Model — Sigma Marketplace

**Version:** 2.0  
**Date:** 2026-06-30  
**Note:** v2.0 introduces the **Projects** engagement model ([PROJECTS_DESIGN.md](PROJECTS_DESIGN.md)). `job_orders` and `payouts` are migrated to `projects` / `outcomes` / `payout_accruals` / `payout_statements`; they remain below for migration reference.

---

## Entity Relationship Overview

```
organizations (type: client | vendor)
    │
    ├── [client] ──── client_vendor_relationships ──── [vendor]
    │
    ├── [client] projects ──── project_vendors ──── [vendor]  (many-to-many)
    │               ├── project_milestones            (milestone model)
    │               └── outcomes (visit|sale|milestone|flat|retainer_period · per vendor)
    │                      ├── job_sop_steps (snapshot, type=visit)
    │                      ├── step_completions → step_photos   (evidence)
    │                      ├── disputes (client raises; super_admin arbitrates)
    │                      └── payout_accruals ──► payout_statements
    │                                              (per VENDOR/period: pending→approved→paid)
    │
    ├── [vendor] users (vendor_admin | worker)
    │               └── worker_profiles, worker_skills          (vendor-internal roster)
    │
service_categories (platform-level)
    └── sop_templates → sop_steps
```

> **Legacy (v1):** `job_orders` → `projects`+`outcomes`; `payouts` → `payout_accruals`+`payout_statements`. Kept below for migration reference.

---

## Schema

### organizations

```sql
CREATE TABLE organizations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            TEXT NOT NULL,
    slug            TEXT NOT NULL UNIQUE,
    type            TEXT NOT NULL CHECK (type IN ('client', 'vendor')),
    status          TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'suspended')),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### client_vendor_relationships

```sql
-- Controls which vendors can serve which clients.
-- A client can only assign jobs to workers from approved vendors.
CREATE TABLE client_vendor_relationships (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    client_org_id   UUID NOT NULL REFERENCES organizations(id),
    vendor_org_id   UUID NOT NULL REFERENCES organizations(id),
    status          TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'inactive')),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),

    UNIQUE(client_org_id, vendor_org_id),
    CHECK (client_org_id <> vendor_org_id)
);

CREATE INDEX idx_cvr_client ON client_vendor_relationships(client_org_id);
CREATE INDEX idx_cvr_vendor ON client_vendor_relationships(vendor_org_id);
```

### users

```sql
CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    email           TEXT NOT NULL UNIQUE,
    full_name       TEXT NOT NULL,
    role            TEXT NOT NULL CHECK (role IN ('super_admin', 'client_admin', 'vendor_admin', 'worker')),
    status          TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'inactive')),
    expo_push_token TEXT,
    last_seen_at    TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_users_org ON users(organization_id);
```

### service_categories

```sql
-- Platform-level. Managed by super_admin only.
-- Each category defines the type of work (e.g. "LAN Installation", "Fiber Optic Splicing").
CREATE TABLE service_categories (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name        TEXT NOT NULL UNIQUE,
    description TEXT,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### sop_templates

```sql
-- Each SOP belongs to a service category.
-- Multiple versions exist per category; only one is published at a time.
CREATE TABLE sop_templates (
    id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    service_category_id  UUID NOT NULL REFERENCES service_categories(id),
    name                 TEXT NOT NULL,
    description          TEXT,
    version              INTEGER NOT NULL DEFAULT 1,
    status               TEXT NOT NULL DEFAULT 'draft' CHECK (status IN ('draft', 'published', 'archived')),
    est_duration_min     INTEGER,
    created_by           UUID NOT NULL REFERENCES users(id),
    created_at           TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at           TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_sop_category ON sop_templates(service_category_id);
```

### sop_steps

```sql
CREATE TABLE sop_steps (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sop_template_id UUID NOT NULL REFERENCES sop_templates(id) ON DELETE CASCADE,
    sequence        INTEGER NOT NULL,
    title           TEXT NOT NULL,
    instructions    TEXT,
    photo_required  BOOLEAN NOT NULL DEFAULT true,
    notes_allowed   BOOLEAN NOT NULL DEFAULT true,
    enforce_order   BOOLEAN NOT NULL DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),

    UNIQUE(sop_template_id, sequence)
);
```

### job_orders  *(v1 — MIGRATED to `projects` + `outcomes` in v2.0; retained for migration reference)*

```sql
-- Created by client_admin. Assigned to a worker from a partner vendor.
-- Payout is a flat amount per job, set at creation time and locked.
CREATE TABLE job_orders (
    id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    client_org_id        UUID NOT NULL REFERENCES organizations(id),
    vendor_org_id        UUID NOT NULL REFERENCES organizations(id),
    service_category_id  UUID NOT NULL REFERENCES service_categories(id),
    sop_template_id      UUID NOT NULL REFERENCES sop_templates(id),
    sop_version          INTEGER NOT NULL,
    worker_id        UUID NOT NULL REFERENCES users(id),
    assigned_by          UUID NOT NULL REFERENCES users(id),
    site_name            TEXT NOT NULL,
    site_address         TEXT,
    site_lat             DECIMAL(10,8),
    site_lng             DECIMAL(11,8),
    scheduled_at         TIMESTAMPTZ NOT NULL,
    started_at           TIMESTAMPTZ,
    completed_at         TIMESTAMPTZ,
    status               TEXT NOT NULL DEFAULT 'assigned'
                         CHECK (status IN ('assigned', 'in_progress', 'completed', 'cancelled')),
    payout_amount        NUMERIC(12,2) NOT NULL,       -- flat per-job amount, locked at creation
    notes                TEXT,
    created_at           TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at           TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_jobs_client      ON job_orders(client_org_id);
CREATE INDEX idx_jobs_vendor      ON job_orders(vendor_org_id);
CREATE INDEX idx_jobs_worker  ON job_orders(worker_id);
CREATE INDEX idx_jobs_status      ON job_orders(status);
CREATE INDEX idx_jobs_scheduled   ON job_orders(scheduled_at);
CREATE INDEX idx_jobs_category    ON job_orders(service_category_id);
```

### job_sop_steps (snapshot)

```sql
-- Snapshot of SOP steps at the time of job creation.
-- Insulates the job from future SOP edits.
CREATE TABLE job_sop_steps (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_order_id    UUID NOT NULL REFERENCES job_orders(id) ON DELETE CASCADE,
    sop_step_id     UUID NOT NULL REFERENCES sop_steps(id),
    sequence        INTEGER NOT NULL,
    title           TEXT NOT NULL,
    instructions    TEXT,
    photo_required  BOOLEAN NOT NULL,
    notes_allowed   BOOLEAN NOT NULL,
    enforce_order   BOOLEAN NOT NULL,

    UNIQUE(job_order_id, sequence)
);
```

### step_completions

```sql
CREATE TABLE step_completions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_order_id    UUID NOT NULL REFERENCES job_orders(id),
    job_sop_step_id UUID NOT NULL REFERENCES job_sop_steps(id),
    worker_id   UUID NOT NULL REFERENCES users(id),
    completed_at    TIMESTAMPTZ NOT NULL,
    notes           TEXT,
    synced_at       TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),

    UNIQUE(job_order_id, job_sop_step_id)
);
```

### step_photos

```sql
CREATE TABLE step_photos (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    step_completion_id  UUID NOT NULL REFERENCES step_completions(id),
    r2_key              TEXT NOT NULL,
    file_size_bytes     INTEGER,
    captured_at         TIMESTAMPTZ,
    gps_lat             DECIMAL(10,8),
    gps_lng             DECIMAL(11,8),
    uploaded_at         TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### payouts  *(v1 — REPLACED by `payout_accruals` + `payout_statements` in v2.0)*

```sql
-- One payout record per job_order.
-- Amount mirrors job_orders.payout_amount — locked at job creation.
CREATE TABLE payouts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_order_id    UUID NOT NULL REFERENCES job_orders(id) UNIQUE,
    client_org_id   UUID NOT NULL REFERENCES organizations(id),
    vendor_org_id   UUID NOT NULL REFERENCES organizations(id),
    worker_id   UUID NOT NULL REFERENCES users(id),
    period          TEXT NOT NULL,                   -- e.g. "2026-05"
    amount          NUMERIC(12,2) NOT NULL,           -- copied from job_orders.payout_amount
    status          TEXT NOT NULL DEFAULT 'pending'
                    CHECK (status IN ('pending', 'approved', 'paid')),
    approved_at     TIMESTAMPTZ,
    approved_by     UUID REFERENCES users(id),        -- client_admin who approved
    paid_at         TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_payouts_vendor     ON payouts(vendor_org_id);
CREATE INDEX idx_payouts_worker ON payouts(worker_id);
CREATE INDEX idx_payouts_period     ON payouts(period);
CREATE INDEX idx_payouts_status     ON payouts(status);
```

---

## v2.0 — Projects Model

Replaces `job_orders`/`payouts`. Design rationale: [PROJECTS_DESIGN.md](PROJECTS_DESIGN.md).

### projects

```sql
CREATE TABLE projects (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    client_org_id       UUID NOT NULL REFERENCES organizations(id),
    -- vendors attached via project_vendors (many-to-many)
    service_category_id UUID NOT NULL REFERENCES service_categories(id),
    title               TEXT NOT NULL,
    description         TEXT,
    pricing_model       TEXT NOT NULL CHECK (pricing_model IN ('per_unit','retainer','flat','milestone')),
    unit_rate           NUMERIC(14,2),         -- per_unit
    retainer_amount     NUMERIC(14,2),         -- retainer (per worker per period)
    retainer_period     TEXT CHECK (retainer_period IN ('weekly','monthly')),
    flat_amount         NUMERIC(14,2),         -- flat
    evidence_photo_required BOOLEAN NOT NULL DEFAULT true,
    evidence_min_photos     INTEGER NOT NULL DEFAULT 1,
    evidence_require_gps    BOOLEAN NOT NULL DEFAULT true,
    evidence_require_ref    BOOLEAN NOT NULL DEFAULT false,
    audit_window_days   INTEGER NOT NULL DEFAULT 7,
    sop_template_id     UUID REFERENCES sop_templates(id),
    sop_version         INTEGER,
    status              TEXT NOT NULL DEFAULT 'draft'
                        CHECK (status IN ('draft','active','paused','closed','archived')),
    starts_at           TIMESTAMPTZ,
    ends_at             TIMESTAMPTZ,
    created_by          UUID NOT NULL REFERENCES users(id),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_projects_client ON projects(client_org_id);
CREATE INDEX idx_projects_status ON projects(status);
```

Integrity (enforced in app / triggers): each `project_vendors` pairing must have an active
`client_vendor_relationships` with the client; pricing fields required by `pricing_model` must be set; pricing is
uniform across a project's vendors (v1) and immutable once the project leaves `draft`; an outcome's `vendor_org_id`
must be one of the project's participating vendors.

### project_vendors

```sql
-- Many-to-many: a project may engage several vendors. Each pairing needs an active client_vendor_relationship.
CREATE TABLE project_vendors (
    id                 UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id         UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    vendor_org_id      UUID NOT NULL REFERENCES organizations(id),
    retainer_headcount INTEGER,    -- retainer projects: placed headcount the vendor declares
    status             TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active','inactive')),
    created_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE(project_id, vendor_org_id)
);
CREATE INDEX idx_project_vendors_vendor ON project_vendors(vendor_org_id);
```

### project_milestones

```sql
CREATE TABLE project_milestones (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id  UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    sequence    INTEGER NOT NULL,
    title       TEXT NOT NULL,
    amount      NUMERIC(14,2) NOT NULL,
    UNIQUE(project_id, sequence)
);
```

### worker_profiles / worker_skills (vendor-internal roster — decoupled from projects; vendor + super_admin only)

```sql
CREATE TABLE worker_profiles (
    user_id              UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,  -- role='worker'
    phone                TEXT,
    region               TEXT,
    base_rate            NUMERIC(14,2),
    employment_attested  BOOLEAN NOT NULL DEFAULT false,  -- vendor attests worker is its employee
    employment_ref       TEXT,
    availability_status  TEXT NOT NULL DEFAULT 'available'
                         CHECK (availability_status IN ('available','limited','on_leave')),
    max_concurrent       INTEGER NOT NULL DEFAULT 5,
    notes                TEXT,
    created_at           TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at           TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE worker_skills (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    service_category_id UUID NOT NULL REFERENCES service_categories(id),
    certification       TEXT,
    certified_at        DATE,
    UNIQUE(user_id, service_category_id)
);
CREATE INDEX idx_worker_skills_user ON worker_skills(user_id);
```

### outcomes

```sql
-- A unit of work that may earn payout. Polymorphic by `type`.
CREATE TABLE outcomes (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id    UUID NOT NULL REFERENCES projects(id),
    vendor_org_id UUID NOT NULL REFERENCES organizations(id),   -- payout attribution (per vendor)
    type          TEXT NOT NULL CHECK (type IN ('visit','sale','milestone','flat','retainer_period')),
    quantity      INTEGER NOT NULL DEFAULT 1,
    amount        NUMERIC(14,2) NOT NULL DEFAULT 0,        -- computed accrual at confirmation
    milestone_id  UUID REFERENCES project_milestones(id),  -- type='milestone'
    period        TEXT,                                    -- type='retainer_period' e.g. '2026-07'
    occurred_at   TIMESTAMPTZ NOT NULL,
    status        TEXT NOT NULL DEFAULT 'reported'
                  CHECK (status IN ('reported','confirmed','disputed','rejected')),
    reported_by   UUID REFERENCES users(id),               -- operational reporter; NULL = system (retainer_period)
    confirmed_at  TIMESTAMPTZ,
    site_name     TEXT, site_address TEXT, site_lat DECIMAL(10,8), site_lng DECIMAL(11,8),
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_outcomes_project ON outcomes(project_id);
CREATE INDEX idx_outcomes_vendor  ON outcomes(vendor_org_id);
CREATE INDEX idx_outcomes_status  ON outcomes(status);
```

> **Re-point:** `job_sop_steps`, `step_completions`, `step_photos` now reference `outcomes(id)` (via a new
> `outcome_id`) instead of `job_order_id`, for `type='visit'` outcomes. Backfill maps each old job → one outcome.

### disputes

```sql
CREATE TABLE disputes (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    outcome_id  UUID NOT NULL REFERENCES outcomes(id),
    raised_by   UUID NOT NULL REFERENCES users(id),    -- client_admin
    reason      TEXT NOT NULL,
    status      TEXT NOT NULL DEFAULT 'open'
                CHECK (status IN ('open','resolved_upheld','resolved_rejected','withdrawn')),
    arbiter_id  UUID REFERENCES users(id),             -- super_admin
    resolution  TEXT,
    resolved_at TIMESTAMPTZ,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_disputes_outcome ON disputes(outcome_id);
```

### payout_statements / payout_accruals

```sql
-- Per-(vendor, worker, period) settlement. Replaces the old per-job `payouts`.
CREATE TABLE payout_statements (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vendor_org_id UUID NOT NULL REFERENCES organizations(id),
    period        TEXT NOT NULL,
    total_amount  NUMERIC(16,2) NOT NULL DEFAULT 0,
    status        TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending','approved','paid')),
    approved_by   UUID REFERENCES users(id),
    approved_at   TIMESTAMPTZ,
    paid_at       TIMESTAMPTZ,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE(vendor_org_id, period)
);
CREATE INDEX idx_statements_status ON payout_statements(status);

-- One accrual per confirmed outcome.
CREATE TABLE payout_accruals (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id    UUID NOT NULL REFERENCES projects(id),
    outcome_id    UUID NOT NULL REFERENCES outcomes(id) UNIQUE,
    statement_id  UUID REFERENCES payout_statements(id),
    client_org_id UUID NOT NULL REFERENCES organizations(id),
    vendor_org_id UUID NOT NULL REFERENCES organizations(id),
    period        TEXT NOT NULL,
    amount        NUMERIC(14,2) NOT NULL,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_accruals_statement ON payout_accruals(statement_id);
CREATE INDEX idx_accruals_vendor    ON payout_accruals(vendor_org_id);
```

---

### notifications

```sql
CREATE TABLE notifications (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID NOT NULL REFERENCES users(id),
    type        TEXT NOT NULL,
                -- v2.0: PROJECT_ASSIGNED | OUTCOME_REPORTED | OUTCOME_DISPUTED
                --       OUTCOME_CONFIRMED | DISPUTE_RESOLVED | RETAINER_ACCRUED
                --       PAYOUT_APPROVED | REMINDER
                -- legacy: JOB_ASSIGNED | JOB_STARTED | JOB_COMPLETED | STEP_COMPLETED
    payload     JSONB NOT NULL DEFAULT '{}',
    sent_at     TIMESTAMPTZ,
    delivered_at TIMESTAMPTZ,
    read_at     TIMESTAMPTZ,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_notif_user ON notifications(user_id);
```

### refresh_tokens

```sql
CREATE TABLE refresh_tokens (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID NOT NULL REFERENCES users(id),
    token_hash  TEXT NOT NULL UNIQUE,
    expires_at  TIMESTAMPTZ NOT NULL,
    revoked_at  TIMESTAMPTZ,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## WatermelonDB Local Schema (Mobile)

> v2.0: `job_orders` becomes `projects`; add `outcomes` for offline sale/visit reporting. `step_completions` /
> `step_photos` hang off an outcome. (Migration bumps the schema version + a WatermelonDB migration.)

```typescript
// db/schema.ts
import { appSchema, tableSchema } from '@nozbe/watermelondb'

export const schema = appSchema({
  version: 3, // v2.0 Projects model
  tables: [
    tableSchema({
      name: 'job_orders',
      columns: [
        { name: 'server_id',            type: 'string' },
        { name: 'client_name',          type: 'string' },
        { name: 'service_category',     type: 'string' },
        { name: 'site_name',            type: 'string' },
        { name: 'site_address',         type: 'string', isOptional: true },
        { name: 'sop_name',             type: 'string' },
        { name: 'scheduled_at',         type: 'number' },
        { name: 'started_at',           type: 'number', isOptional: true },
        { name: 'completed_at',         type: 'number', isOptional: true },
        { name: 'status',               type: 'string' },
        { name: 'notes',                type: 'string', isOptional: true },
        { name: 'last_synced_at',       type: 'number', isOptional: true },
      ],
    }),
    // v2.0: repeatable outcomes (sales/visits) reported offline by the worker.
    tableSchema({
      name: 'outcomes',
      columns: [
        { name: 'server_id',   type: 'string', isOptional: true },
        { name: 'project_id',  type: 'string', isIndexed: true },
        { name: 'type',        type: 'string' },   // visit | sale
        { name: 'quantity',    type: 'number' },
        { name: 'occurred_at', type: 'number' },
        { name: 'status',      type: 'string' },    // reported | confirmed | disputed | rejected
        { name: 'sync_status', type: 'string' },    // pending | synced | failed
      ],
    }),
    tableSchema({
      name: 'job_sop_steps',
      columns: [
        { name: 'server_id',      type: 'string' },
        { name: 'job_order_id',   type: 'string', isIndexed: true },
        { name: 'sequence',       type: 'number' },
        { name: 'title',          type: 'string' },
        { name: 'instructions',   type: 'string', isOptional: true },
        { name: 'photo_required', type: 'boolean' },
        { name: 'enforce_order',  type: 'boolean' },
      ],
    }),
    tableSchema({
      name: 'step_completions',
      columns: [
        { name: 'server_id',         type: 'string', isOptional: true },
        { name: 'job_order_id',      type: 'string', isIndexed: true },
        { name: 'job_sop_step_id',   type: 'string' },
        { name: 'completed_at',      type: 'number' },
        { name: 'notes',             type: 'string', isOptional: true },
        { name: 'sync_status',       type: 'string' },  // pending|synced|failed
      ],
    }),
    tableSchema({
      name: 'step_photos',
      columns: [
        { name: 'step_completion_id', type: 'string', isIndexed: true },
        { name: 'local_path',         type: 'string' },
        { name: 'r2_key',             type: 'string', isOptional: true },
        { name: 'upload_status',      type: 'string' },  // pending|uploaded|failed
        { name: 'captured_at',        type: 'number' },
      ],
    }),
  ],
})
```
