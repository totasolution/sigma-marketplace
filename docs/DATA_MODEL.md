# Data Model — Sigma Marketplace

**Version:** 1.0  
**Date:** 2026-05-19

---

## Entity Relationship Overview

```
organizations
    │
    ├── users (role: client_admin, technician)
    │
    ├── sop_templates
    │       └── sop_steps
    │
    ├── job_orders
    │       ├── sop_steps (snapshot at assignment)
    │       ├── step_completions
    │       │       └── step_photos
    │       └── payouts
    │
    └── notifications
```

---

## Schema

### organizations

```sql
CREATE TABLE organizations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            TEXT NOT NULL,
    slug            TEXT NOT NULL UNIQUE,
    status          TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'suspended')),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### users

```sql
CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    email           TEXT NOT NULL UNIQUE,
    full_name       TEXT NOT NULL,
    role            TEXT NOT NULL CHECK (role IN ('super_admin', 'client_admin', 'technician')),
    status          TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'inactive')),
    expo_push_token TEXT,                          -- nullable, set on mobile login
    last_seen_at    TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_users_org ON users(organization_id);
```

### sop_templates

```sql
CREATE TABLE sop_templates (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),  -- NULL = global template
    name            TEXT NOT NULL,
    description     TEXT,
    version         INTEGER NOT NULL DEFAULT 1,
    status          TEXT NOT NULL DEFAULT 'draft' CHECK (status IN ('draft', 'published', 'archived')),
    est_duration_min INTEGER,
    created_by      UUID NOT NULL REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### sop_steps

```sql
CREATE TABLE sop_steps (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sop_template_id UUID NOT NULL REFERENCES sop_templates(id) ON DELETE CASCADE,
    sequence        INTEGER NOT NULL,
    title           TEXT NOT NULL,
    instructions    TEXT,
    payout_rate     NUMERIC(12,2) NOT NULL DEFAULT 0,
    photo_required  BOOLEAN NOT NULL DEFAULT true,
    notes_allowed   BOOLEAN NOT NULL DEFAULT true,
    enforce_order   BOOLEAN NOT NULL DEFAULT true,     -- must complete prev step first
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),

    UNIQUE(sop_template_id, sequence)
);
```

### job_orders

```sql
CREATE TABLE job_orders (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    site_name       TEXT NOT NULL,
    site_address    TEXT,
    site_lat        DECIMAL(10,8),
    site_lng        DECIMAL(11,8),
    technician_id   UUID NOT NULL REFERENCES users(id),
    assigned_by     UUID NOT NULL REFERENCES users(id),
    sop_template_id UUID NOT NULL REFERENCES sop_templates(id),
    sop_version     INTEGER NOT NULL,               -- locked at assignment time
    scheduled_at    TIMESTAMPTZ NOT NULL,
    started_at      TIMESTAMPTZ,
    completed_at    TIMESTAMPTZ,
    status          TEXT NOT NULL DEFAULT 'assigned'
                    CHECK (status IN ('assigned', 'in_progress', 'completed', 'cancelled')),
    notes           TEXT,
    total_payout    NUMERIC(12,2),                  -- computed on completion
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_jobs_org         ON job_orders(organization_id);
CREATE INDEX idx_jobs_technician  ON job_orders(technician_id);
CREATE INDEX idx_jobs_status      ON job_orders(status);
CREATE INDEX idx_jobs_scheduled   ON job_orders(scheduled_at);
```

### job_sop_steps (snapshot)

```sql
-- Snapshot of SOP steps at the time of job assignment.
-- Insulates the job from future SOP edits.
CREATE TABLE job_sop_steps (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_order_id    UUID NOT NULL REFERENCES job_orders(id) ON DELETE CASCADE,
    sop_step_id     UUID NOT NULL REFERENCES sop_steps(id),
    sequence        INTEGER NOT NULL,
    title           TEXT NOT NULL,
    instructions    TEXT,
    payout_rate     NUMERIC(12,2) NOT NULL,
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
    technician_id   UUID NOT NULL REFERENCES users(id),
    completed_at    TIMESTAMPTZ NOT NULL,
    notes           TEXT,
    payout_amount   NUMERIC(12,2) NOT NULL,          -- locked from job_sop_steps.payout_rate
    synced_at       TIMESTAMPTZ,                     -- when server received this record
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),

    UNIQUE(job_order_id, job_sop_step_id)            -- one completion per step per job
);
```

### step_photos

```sql
CREATE TABLE step_photos (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    step_completion_id  UUID NOT NULL REFERENCES step_completions(id),
    r2_key              TEXT NOT NULL,               -- Cloudflare R2 object key
    file_size_bytes     INTEGER,
    captured_at         TIMESTAMPTZ,                 -- from device EXIF
    gps_lat             DECIMAL(10,8),
    gps_lng             DECIMAL(11,8),
    uploaded_at         TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### payouts

```sql
CREATE TABLE payouts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    technician_id   UUID NOT NULL REFERENCES users(id),
    job_order_id    UUID NOT NULL REFERENCES job_orders(id) UNIQUE,
    period          TEXT NOT NULL,                   -- e.g. "2026-05"
    amount          NUMERIC(12,2) NOT NULL,
    status          TEXT NOT NULL DEFAULT 'pending'
                    CHECK (status IN ('pending', 'approved', 'paid')),
    paid_at         TIMESTAMPTZ,
    paid_by         UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_payouts_technician ON payouts(technician_id);
CREATE INDEX idx_payouts_period     ON payouts(period);
CREATE INDEX idx_payouts_status     ON payouts(status);
```

### notifications

```sql
CREATE TABLE notifications (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id),
    type            TEXT NOT NULL,
                    -- JOB_ASSIGNED | JOB_STARTED | JOB_COMPLETED
                    -- STEP_COMPLETED | PAYOUT_PROCESSED | REMINDER
    payload         JSONB NOT NULL DEFAULT '{}',
    sent_at         TIMESTAMPTZ,
    delivered_at    TIMESTAMPTZ,
    read_at         TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
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

```typescript
// db/schema.ts
import { appSchema, tableSchema } from '@nozbe/watermelondb'

export const schema = appSchema({
  version: 1,
  tables: [
    tableSchema({
      name: 'job_orders',
      columns: [
        { name: 'server_id',      type: 'string' },
        { name: 'site_name',      type: 'string' },
        { name: 'site_address',   type: 'string', isOptional: true },
        { name: 'sop_name',       type: 'string' },
        { name: 'scheduled_at',   type: 'number' },   // unix timestamp
        { name: 'started_at',     type: 'number', isOptional: true },
        { name: 'completed_at',   type: 'number', isOptional: true },
        { name: 'status',         type: 'string' },
        { name: 'total_payout',   type: 'number' },
        { name: 'notes',          type: 'string', isOptional: true },
        { name: 'last_synced_at', type: 'number', isOptional: true },
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
        { name: 'payout_rate',    type: 'number' },
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
        { name: 'payout_amount',     type: 'number' },
        { name: 'sync_status',       type: 'string' }, // pending|synced|failed
      ],
    }),
    tableSchema({
      name: 'step_photos',
      columns: [
        { name: 'step_completion_id', type: 'string', isIndexed: true },
        { name: 'local_path',         type: 'string' },
        { name: 'r2_key',             type: 'string', isOptional: true },
        { name: 'upload_status',      type: 'string' }, // pending|uploaded|failed
        { name: 'captured_at',        type: 'number' },
      ],
    }),
  ],
})
```
