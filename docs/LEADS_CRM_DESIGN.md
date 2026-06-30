# Design — Leads & Field-Sales CRM

**Version:** 0.1 (design, for review)
**Date:** 2026-06-30
**Status:** Proposed — not yet approved for build
**Related:** [PROJECTS_DESIGN.md](PROJECTS_DESIGN.md) (sales projects, outcomes, commission)

> **The idea.** Not a rep-entered CRM — the platform **provides leads** to the field team to *accelerate output*.
> A client posts a **Sales Project** (e.g. "soundbox"); Sigma matches **relevant lead contacts** from its pool and
> hands the vendor's reps a curated, mapped worklist. Reps work leads → close → (later) commission accrues.
> This is a core **selling point**: "post a sales project, we hand your field team a qualified target list."

---

## 1. Concept & Flow

```
Client posts Sales Project (product=soundbox, category, region, target count, Rp/sale)
        │
        ▼  match: category↔product + geo + segment
Platform LEAD POOL ──────────────► matched lead list ──► distribute to vendors
   (curated + rep-sourced)                                      │ (territory / round-robin)
                                                                ▼
                                          Rep mobile worklist (mapped)
                                          assigned → contacted → visited → won / lost
                                                                │ +visit evidence (GPS/photo)
                                                                ▼
                                          (Phase 2) Won → sale outcome → commission accrual
```

**Acceleration =** reps don't cold-prospect; they get a relevant, mapped list + light guidance (route, last-contacted hint).

---

## 2. Goals / Non-Goals

**Goals**
- A platform **lead pool** sourced from a **curated data asset** + **rep field-sourced** entries.
- **Match & distribute** leads to a sales project's reps by product category + geography.
- A rep **mobile worklist** (map + pipeline stages + visit evidence), offline-first.
- Client + platform **funnel/conversion analytics** (the selling-point view).

**Non-Goals (v1)**
- **No lead locking/exclusivity** (owner decision) — a lead may be worked by multiple projects/reps. *(Risk + soft mitigation in §7.)*
- **No outcome/commission link yet** — standalone CRM in P1; wire Won→`sale` outcome in P2.
- **Client-provided lists** and **third-party enrichment** — out (not chosen).
- Lead scoring, territories, automations — later.

---

## 3. Decisions (owner, 2026-06-30)

- **Sources:** platform **curated data asset** + **rep field-sourced**. (No client uploads, no 3rd-party enrichment in v1.)
- **No locking:** leads can appear across projects/reps simultaneously. Accepted risk: collision / merchant fatigue.
- **Standalone in P1:** `project_leads.outcome_id` stays null until P2.
- **Access:** vendors see leads **assigned to them** (their workers see them in the field app); clients see their project's leads; super-admin curates the pool; **DNC** (`leads.status='dnc'`) honored everywhere.

---

## 4. Data Model (sketch)

```sql
-- Platform-owned lead pool. Reused across projects (no exclusivity).
CREATE TABLE leads (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name          TEXT NOT NULL,                 -- merchant / outlet
    category      TEXT,                          -- maps to service_categories / product fit
    segment       TEXT,                          -- e.g. F&B, retail, UMKM tier
    address       TEXT,
    region        TEXT,
    lat           DECIMAL(10,8), lng DECIMAL(11,8),
    contact_name  TEXT,
    contact_phone TEXT,                          -- access-gated by role
    source        TEXT NOT NULL CHECK (source IN ('curated','field')),
    added_by      UUID REFERENCES users(id),     -- rep, if field-sourced
    status        TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active','dnc')),  -- do-not-contact
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_leads_category ON leads(category);
CREATE INDEX idx_leads_region   ON leads(region);
-- geo index (PostGIS or lat/lng box) for "near me" matching.

-- A lead being worked within a project. No-locking ⇒ many rows per lead allowed.
CREATE TABLE project_leads (
    id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id       UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    lead_id          UUID NOT NULL REFERENCES leads(id),
    assigned_vendor_org_id UUID REFERENCES organizations(id),   -- distributed to a vendor; vendor assigns workers internally
    stage            TEXT NOT NULL DEFAULT 'assigned'
                     CHECK (stage IN ('assigned','contacted','visited','won','lost')),
    last_activity_at TIMESTAMPTZ,
    outcome_id       UUID REFERENCES outcomes(id),   -- NULL in P1; set on Won in P2 (vendor-attributed)
    created_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE(project_id, lead_id, assigned_vendor_org_id)
);
CREATE INDEX idx_project_leads_project ON project_leads(project_id);
CREATE INDEX idx_project_leads_vendor  ON project_leads(assigned_vendor_org_id);

-- Activity timeline on a worked lead (visit carries evidence, like outcomes).
CREATE TABLE lead_activities (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_lead_id UUID NOT NULL REFERENCES project_leads(id) ON DELETE CASCADE,
    type            TEXT NOT NULL CHECK (type IN ('note','call','visit','stage_change')),
    body            TEXT,
    by_user         UUID REFERENCES users(id),
    occurred_at     TIMESTAMPTZ NOT NULL,
    lat DECIMAL(10,8), lng DECIMAL(11,8), photo_key TEXT,   -- visit evidence
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Matching criteria live on the **sales project** (reuse `projects`): target `service_category`, region(s),
target count, optional segment filter. A nightly/triggered **match job** proposes leads; distribution creates
`project_leads` rows per vendor.

---

## 5. Matching & Distribution

- **Match:** leads where `category` fits the project's product + within target region/geo + `status='active'`
  (+ a "last contacted N days" hint, since no hard locking). Ranked by proximity / freshness.
- **Distribute:** assign matched leads to the project's **vendors** — round-robin or by **territory** (vendor region).
  Each vendor's workers then work them (vendor-internal who-does-what). v1 = simple round-robin; territory = later.
- **Field-sourced top-up:** a rep can add a new merchant in the field → enters `leads` (`source='field'`) and is
  immediately a `project_lead` for them; reusable by future projects.

---

## 6. Roles & Screens

- **Super admin (web)** — own the **data asset**: import/curate leads, categorize, set DNC, define match rules;
  cross-client **funnel analytics** (extends the [admin CRM](../wireframes/admin-crm.html)).
- **Client admin (web)** — per project: review/approve the **matched lead list**, watch the **pipeline (Kanban) +
  conversion funnel**, rep performance.
- **Rep / worker (mobile)** — **"My leads"**: a mapped worklist; tap to advance stage; **log a visit** (GPS/photo);
  **add a field lead**. Offline-first, syncs like outcomes.

---

## 7. No-Locking: risk & soft mitigation

A lead can be worked by multiple reps/projects at once → risk of **collisions** and **merchant fatigue** (same
outlet pitched twice). Accepted by owner. Soft mitigations (non-blocking):
- Show a **"last contacted X days ago"** badge on a lead (cross-project, read-only) so reps self-avoid.
- Optional **cool-down hint** (e.g. flag if contacted < 7 days) — advisory only.
- Revisit hard locking later if fatigue shows up in the data.

---

## 8. Phasing

- **P1 — standalone leads CRM:** lead pool (import + field-add) · match/assign to a project · rep mobile worklist + stages + visit evidence · client pipeline/funnel. *No outcome link.*
- **P2 — wire to sales:** Won `project_lead` → `sale` outcome → commission; conversion→GMV analytics; auto-matching.
- **P3 — scale:** territories, lead scoring, dedup, data partnerships/enrichment, consent management.

---

## 9. Open Decisions

- **Match trigger** — auto on project activation, or client requests a batch? (proposed: client requests, admin approves)
- **Distribution** — round-robin (v1) vs territory-based (later) — confirm v1 is fine.
- **Geo source** — how the curated pool gets lat/lng (maps/POI provider) — sourcing + cost.

---

## 10. Proposed Next Step

Wireframes to make it tangible: **lead-pool admin** (super-admin), **project lead-matching + pipeline/funnel**
(client), and **mobile "My leads" worklist + log-visit + add-lead** (rep). Then fold the data model into
[DATA_MODEL.md](DATA_MODEL.md) on approval.
