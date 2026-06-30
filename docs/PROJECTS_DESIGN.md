# Design — Projects & Engagement Model

**Version:** 1.0 (locked)
**Date:** 2026-06-30
**Status:** ✅ Locked — approved for build (Phase A + B next)
**Supersedes on approval:** parts of [PRD.md](PRD.md) §5.3/§5.5 and [DATA_MODEL.md](DATA_MODEL.md) `job_orders`/`payouts`

> **v0.3 — simplification.** The platform's commercial relationship is now strictly **client ↔ vendor**:
> outcomes, payouts, and leads attribute to the **vendor** (per-vendor). **Agreements** and the **project↔worker
> roster** are removed. Workers are the vendor's field executors (they still run the mobile app to report work),
> but worker pay and roster are **vendor-internal** — the platform doesn't track per-worker money. (Earlier v0.2
> had per-worker payouts + per-vendor agreements; both dropped.)

---

## 1. Context & Driver

**Permenaker No. 7 Tahun 2026** restricts outsourcing of a company's *essential/core* roles. Companies can
contract **outcome-based projects** from a vendor, where workers remain **employed by the vendor**, not the client.

Today's platform models a **Job Order** = one site task, one worker, one flat fee. Too narrow for ongoing
engagements, commission, or placed-role retainers. **This design makes the `Project` the primary unit, replacing
the Job Order**, and keeps the commercial relationship cleanly **client → vendor** (workers stay inside the vendor).

> Legal note: the regulatory effect is as described by the product owner; validate specifics with counsel.

---

## 2. Goals / Non-Goals

**Goals**
- A `Project` engagement: a client contracts **one or more vendors** (many-to-many) under one **pricing model**
  (per-unit commission, recurring retainer, flat fee, milestone).
- **Per-vendor** outcomes, accrual payouts, and lead distribution.
- Migrate the site-task + SOP + photo-evidence flow as one project type.
- Vendor self-reports outcomes; client **monitors** (dispute is the exception); auto-confirm.
- Per-project client views: **Monitor · Leads · Sales Report**.

**Non-Goals (this version)**
- **Per-worker** payout / attribution / project rostering — the platform settles with the **vendor**; the vendor
  pays and rosters its own workers (vendor-internal).
- **Agreements / acceptance gating** — removed; the project record itself is the engagement.
- Budget caps/alerts; direct payment disbursement (manual/CSV); open marketplace discovery.

---

## 3. Core Concept — The Engagement Model

```
Client ──contracts──► Project ──fulfilled by──► one or more Vendors  (project_vendors, m:n)
                         │                              │
                         ▼                         (vendor's own workers
                    Pricing Model                   execute in the field —
                  (how the vendor earns)            vendor-internal)
                         │
                         ▼
                     Outcomes  ── confirmed ──►  Payout Accruals ──► Payout Statements
              (visit | sale | milestone |         (per outcome,         (per VENDOR, per period:
               flat | retainer period)             per vendor)           pending→approved→paid)
                         │
                  evidence + verification
              (vendor reports → client monitors / disputes → auto-confirm)
```

- **Project** — a client's engagement with **one or more vendors**. Carries the **pricing model** and lifecycle.
- **Outcome** — a unit of work that earns payout, **attributed to a vendor** (a visit, sale, milestone, flat
  completion, or retainer period). Reported by the vendor (its field worker via mobile, or its admin via web) —
  the reporter is operational only; **money attributes to the vendor**.
- **Payout** — accrues **per vendor per period**; the platform/client settles with the vendor, who pays its own
  workers off-platform.
- **Workers** — the vendor's field executors. They use the mobile app to report outcomes and work leads, but are
  **not rostered to projects** and **not a payout unit** here (see §8.1, vendor-internal).

A Job Order maps in as a **Project** (`flat`) with a single **visit/flat** outcome carrying the SOP snapshot.

---

## 4. Pricing Models

| Model | How the vendor earns | Outcome type | Reported by |
|---|---|---|---|
| **Per-unit commission** | `unit_rate × confirmed quantity` | `sale` | vendor's worker (mobile) |
| **Recurring retainer** | `retainer_amount × placed headcount` per period | `retainer_period` | system (vendor declares headcount) |
| **Flat fee** | one fixed `flat_amount` on delivery | `flat` | vendor admin (web) |
| **Milestone** | sum of completed `milestone.amount` | `milestone` | vendor admin (web) |
| *(any model)* | site task w/ checklist | `visit` | vendor's worker (mobile) |

One pricing model per project, **uniform across the project's vendors** (per-vendor rates = future). Any project
may attach an SOP checklist + evidence rules to its outcomes.

---

## 5. Domain Lifecycle

**Project status:** `draft → active → (paused) → closed → archived`
- `draft`: client configuring; pricing editable here only.
- `active`: vendors attached (`project_vendors`); outcomes can be logged. *(No agreement gating.)*
- `paused`: no new outcomes; existing accruals stand. `closed`: settled; no new outcomes.

**Outcome status:** `reported → confirmed` (+ `disputed` / `rejected`)
- `reported`: logged with required evidence; attributed to a vendor. `retainer_period` is system-generated.
- `confirmed`: client accepted, or audit window elapsed (auto-confirm) → produces a **payout accrual**.
- `disputed` → vendor edits/withdraws or super_admin arbitrates → `confirmed` / `rejected`.

**Client posture = monitor-first.** Default is **auto-confirm after the audit window** (7 days, per-project).
Dispute is the exception; the financial control is **evidence quality + auto-confirm**.

---

## 6. Verification & Dispute

Vendor self-reports with **mandatory per-project evidence rules** (e.g. a `sale` needs receipt photo + ref + GPS).
Client **monitors**; may **dispute** within the window; otherwise it **auto-confirms**. Disputes (raised by client,
arbitrated by super_admin) are recorded for the audit trail. No per-outcome client approval step.

---

## 7. Payout / Accrual Engine (per vendor)

- Each `confirmed` outcome → a **`payout_accrual`** `{ project, outcome, vendor, period, amount }`.
- At period close, accruals roll up into a **`payout_statement`** per **`(vendor, period)`** (`pending → approved
  → paid`). The platform/client pays the vendor; **the vendor pays its workers internally** (off-platform).
- CSV export = accrual line items grouped by vendor/period.
- **No budget cap.** Commission is self-funding; other models deterministic. Control = verification.

### 7.1 Scheduled / background jobs
- **Auto-confirm** — `reported → confirmed` past the audit window (no open dispute) → write accrual.
- **Retainer generation** — each period, one `retainer_period` outcome per active retainer project; `quantity` =
  the vendor's declared placed headcount; `amount = retainer_amount × quantity`, attributed to the vendor.
- **Statement roll-up** — group confirmed accruals into `payout_statements` per (vendor, period).
- **Visit reminders** (PRD US-051).

---

## 8. Data Model (sketch)

`job_orders` migrates into `projects` + `outcomes`. Canonical schema graduates into [DATA_MODEL.md](DATA_MODEL.md).

```sql
CREATE TABLE projects (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    client_org_id       UUID NOT NULL REFERENCES organizations(id),
    -- vendors attached via project_vendors (many-to-many)
    service_category_id UUID NOT NULL REFERENCES service_categories(id),
    title               TEXT NOT NULL,
    description         TEXT,
    pricing_model       TEXT NOT NULL CHECK (pricing_model IN ('per_unit','retainer','flat','milestone')),
    unit_rate           NUMERIC(14,2),   retainer_amount NUMERIC(14,2),   retainer_period TEXT,
    flat_amount         NUMERIC(14,2),
    evidence_photo_required BOOLEAN NOT NULL DEFAULT true,
    evidence_min_photos     INTEGER NOT NULL DEFAULT 1,
    evidence_require_gps    BOOLEAN NOT NULL DEFAULT true,
    evidence_require_ref    BOOLEAN NOT NULL DEFAULT false,
    audit_window_days   INTEGER NOT NULL DEFAULT 7,
    sop_template_id     UUID REFERENCES sop_templates(id),  sop_version INTEGER,
    status              TEXT NOT NULL DEFAULT 'draft'
                        CHECK (status IN ('draft','active','paused','closed','archived')),
    starts_at TIMESTAMPTZ, ends_at TIMESTAMPTZ,
    created_by UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(), updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Participating vendors (many-to-many). Each pairing needs an active client_vendor_relationship.
CREATE TABLE project_vendors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id    UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    vendor_org_id UUID NOT NULL REFERENCES organizations(id),
    retainer_headcount INTEGER,    -- for retainer projects: how many the vendor placed
    status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active','inactive')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE(project_id, vendor_org_id)
);

CREATE TABLE project_milestones (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    sequence INTEGER NOT NULL, title TEXT NOT NULL, amount NUMERIC(14,2) NOT NULL,
    UNIQUE(project_id, sequence)
);

-- A unit of work that earns payout, attributed to a VENDOR (no per-worker payout link).
CREATE TABLE outcomes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id    UUID NOT NULL REFERENCES projects(id),
    vendor_org_id UUID NOT NULL REFERENCES organizations(id),   -- payout attribution
    type     TEXT NOT NULL CHECK (type IN ('visit','sale','milestone','flat','retainer_period')),
    quantity INTEGER NOT NULL DEFAULT 1,    -- units (per_unit) / headcount (retainer)
    amount   NUMERIC(14,2) NOT NULL DEFAULT 0,
    milestone_id UUID REFERENCES project_milestones(id),  period TEXT,
    occurred_at  TIMESTAMPTZ NOT NULL,
    status TEXT NOT NULL DEFAULT 'reported' CHECK (status IN ('reported','confirmed','disputed','rejected')),
    reported_by UUID REFERENCES users(id),   -- operational reporter (vendor admin/worker); NULL = system
    confirmed_at TIMESTAMPTZ,
    site_name TEXT, site_address TEXT, site_lat DECIMAL(10,8), site_lng DECIMAL(11,8),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(), updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_outcomes_project ON outcomes(project_id);
CREATE INDEX idx_outcomes_vendor  ON outcomes(vendor_org_id);

CREATE TABLE disputes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    outcome_id UUID NOT NULL REFERENCES outcomes(id),
    raised_by UUID NOT NULL REFERENCES users(id), reason TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'open' CHECK (status IN ('open','resolved_upheld','resolved_rejected','withdrawn')),
    arbiter_id UUID REFERENCES users(id), resolution TEXT, resolved_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Settlement per (vendor, period). Replaces the old per-job `payouts`.
CREATE TABLE payout_statements (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vendor_org_id UUID NOT NULL REFERENCES organizations(id),
    period TEXT NOT NULL, total_amount NUMERIC(16,2) NOT NULL DEFAULT 0,
    status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending','approved','paid')),
    approved_by UUID REFERENCES users(id), approved_at TIMESTAMPTZ, paid_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE(vendor_org_id, period)
);

CREATE TABLE payout_accruals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id),
    outcome_id UUID NOT NULL REFERENCES outcomes(id) UNIQUE,
    statement_id UUID REFERENCES payout_statements(id),
    client_org_id UUID NOT NULL REFERENCES organizations(id),
    vendor_org_id UUID NOT NULL REFERENCES organizations(id),
    period TEXT NOT NULL, amount NUMERIC(14,2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Evidence (`job_sop_steps`/`step_completions`/`step_photos`) re-points to `outcomes(id)` for `visit` types.

### 8.1 Workers (vendor-internal)

Workers are **decoupled from projects/outcomes/payouts** here. The vendor keeps an internal roster
(`worker_profiles`, `worker_skills`) for its own ops and to run the **mobile field app** (reporting outcomes,
working leads). The platform does **not** assign workers to projects or pay workers — the vendor distributes work
and pays staff internally. (Capacity = the vendor's `availability_status`; no project-load.) These tables stay
vendor-owned and are never visible to clients.

### 8.2 Integrity rules
- Each `project_vendors` pairing must have an **active `client_vendor_relationships`** with the client.
- Pricing fields required by `pricing_model` must be set; pricing uniform across vendors; immutable after `draft`.
- An outcome's `vendor_org_id` must be one of the project's participating vendors.

---

## 9. API Surface (high level)
- `projects`: CRUD + `activate`/`pause`/`close`; nested `vendors`, `milestones`.
- `outcomes`: report (vendor/worker) · dispute (client) · super_admin resolve; auto-confirm via scheduler.
- `payouts`: accruals, **per-vendor** statements, approve/mark-paid, CSV.
- `leads`: pool + match/assign **to a vendor** on a project (see [LEADS_CRM_DESIGN.md](LEADS_CRM_DESIGN.md)).
- `workers`: vendor-scoped roster (vendor-internal; not client-exposed).
- Legacy `job-orders`: thin adapter over `projects`+`outcomes` during migration.

---

## 10. Mobile Impact (offline-first)
- The SOP checklist becomes the **`visit` outcome**; **`sale`** is the repeatable log-outcome flow — both reported
  by the vendor's worker (operational), attributed to the vendor. Offline; device-wins on worker-reported data.
- `milestone`/`flat` reported by vendor admin (web); `retainer_period` system-generated.
- **Earnings on mobile is vendor-internal** — the platform settles with the vendor, so any per-worker earnings the
  worker sees come from the vendor's internal payroll, not platform money.

---

## 11. Migration & Rollout (phased)
Job Orders are live, so this migrates a running system:
1. **A — additive schema** alongside `job_orders`.
2. **B — backfill** each job → a `project` (`flat`) + one outcome (attributed to its vendor); re-point evidence; migrate `payouts` → per-vendor accruals/statements.
3. **C — legacy adapter** (REST + mobile sync) so existing apps keep working.
4. **D — new UI + pricing models**: builder, vendor reporting, client Monitor/Leads/Report, per-vendor payouts, mobile. Pricing phasing: `per_unit`+`flat` first, then `retainer`+`milestone`.
5. **E — deprecate** legacy.

---

## 12. Compliance & Audit (Permenaker)
- **Separation:** projects are `client_org → vendor_org(s)`; workers are entirely **vendor-internal** (the platform
  never tracks them as a client/platform payout unit) — arguably the cleanest separation.
- **Immutable audit trail:** outcome transitions + disputes/arbitration + evidence are append-only.
- **Worker privacy:** clients never see workers; outcomes show the **vendor**.
- **Evidence retention: 2 months** (owner) — covers the 7-day audit window; purged thereafter.

---

## 13. Decisions & Remaining Risks

**Resolved (owner):**
- **Per-vendor attribution** — outcomes, payouts, leads attribute to the **vendor**; per-worker is vendor-internal.
- **Agreements removed**; **workers not rostered to projects**.
- Multi-vendor (`project_vendors`, m:n); pricing uniform across a project's vendors.
- Monitor / Leads / Sales Report kept as per-project client views.
- Client posture monitor-first; audit window 7 days; retainer accrues unconditionally (per declared headcount).
- No budget cap; evidence retention 2 months; disputes kept (super_admin arbiter); pricing phasing.
- **No clawback** — payment is **post-confirmation** (after the 7-day audit/dispute window); a paid statement is **final**. Residual: fraud found *after* payment isn't recoverable in-platform (accepted).

- **Dispute-resolution SLA = 3 business days.**
- **Retainer headcount** = vendor-declared on `project_vendors`.

**✅ All decisions locked (v1.0).** Sole accepted residual: fraud found *after* payment isn't recoverable in-platform.

---

## 14. Doc Impacts
- **PRD → v2.x:** per-vendor payouts; drop agreement story; worker mgmt = vendor-internal.
- **DATA_MODEL → v2.0:** remove `project_agreements`/`project_workers`; outcomes `vendor_org_id`; per-vendor statements/accruals; leads → vendor.
- **GRAND_DESIGN → v1.1:** per-vendor payout flow; modules.
- **WIREFRAMES:** remove agreement panels; vendor-level attribution; per-vendor payouts.

---

## 15. Proposed Next Step
On approval, build **Phase A + B** (additive schema + backfill, no UI) → implementation plan (migrations, sqlc, repo/service).
