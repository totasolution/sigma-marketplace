# Data Model — Sigma Marketplace

**Version:** 1.1  
**Date:** 2026-05-19

---

## Entity Relationship Overview

```
organizations (type: client | vendor)
    │
    ├── [client] ──── client_vendor_relationships ──── [vendor]
    │                                                      │
    ├── [client] job_orders ────────────────────── worker_id (vendor user)
    │               │                                      │
    │               ├── job_sop_steps (snapshot)    users (role: vendor_admin | worker)
    │               ├── step_completions
    │               │       └── step_photos
    │               └── payouts
    │
    ├── [vendor] users (role: vendor_admin | worker)
    │
service_categories (platform-level)
    └── sop_templates
            └── sop_steps
```

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

### job_orders

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

### payouts

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

### notifications

```sql
CREATE TABLE notifications (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID NOT NULL REFERENCES users(id),
    type        TEXT NOT NULL,
                -- JOB_ASSIGNED | JOB_STARTED | JOB_COMPLETED
                -- STEP_COMPLETED | PAYOUT_APPROVED | REMINDER
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

```typescript
// db/schema.ts
import { appSchema, tableSchema } from '@nozbe/watermelondb'

export const schema = appSchema({
  version: 2,
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
