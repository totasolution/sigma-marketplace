# Tech Stack & Cost Analysis — Sigma Marketplace

**Version:** 1.0  
**Date:** 2026-05-19

---

## Stack Summary

| Layer | Technology | Rationale |
|---|---|---|
| API | Go + Chi router | Low memory, fast, existing team expertise |
| ORM / SQL | sqlc + golang-migrate | Type-safe queries, no runtime magic |
| Mobile | React Native + Expo | Single codebase iOS + Android |
| Local DB (mobile) | WatermelonDB (SQLite) | Offline-first, built-in sync protocol |
| Web | Next.js 15 (App Router) | SSR + RSC, deploys free on Vercel |
| UI Components | shadcn/ui + Tailwind | No license cost, full control |
| Database | PostgreSQL via Supabase | Managed, free tier, RLS support |
| Auth | Supabase Auth (JWT) | Free, handles token refresh, secure |
| File Storage | Cloudflare R2 | Free 10GB, S3-compatible, no egress fee |
| Notification Queue | Upstash Redis | Serverless Redis, free tier, pay-per-use |
| Push Notifications | Expo Push API | Handles APNs + FCM, free |
| Realtime (web) | Supabase Realtime | Free, WebSocket-based, zero extra infra |
| API Hosting | Railway | $5/mo, auto-deploy, simple config |
| Web Hosting | Vercel | Free tier, auto-deploy |
| Mobile Build | EAS Build | 30 free builds/month |
| Error Tracking | Sentry | Free tier (5k events/day) |
| CI/CD | GitHub Actions | Free for public repos, 2000 min/mo private |

---

## Cost Analysis

### MVP Phase (0–100 users, 0–500 jobs/month)

| Service | Free Tier Limit | Monthly Cost |
|---|---|---|
| Supabase (PostgreSQL + Auth + Realtime) | 500MB DB, 50k MAU | **$0** |
| Cloudflare R2 | 10GB storage, 1M reads/mo | **$0** |
| Upstash Redis | 10k commands/day | **$0** |
| Vercel (Next.js) | 100GB bandwidth | **$0** |
| Railway (Go API) | $5 credit/mo on Starter | **$5** |
| Expo Push API | Unlimited | **$0** |
| EAS Build | 30 builds/month | **$0** |
| Sentry | 5k events/day | **$0** |
| GitHub Actions | 2000 min/month | **$0** |
| **TOTAL** | | **~$5/month** |

### Growth Phase (100–1,000 users, 500–5,000 jobs/month)

| Service | Expected Usage | Monthly Cost |
|---|---|---|
| Supabase Pro | 8GB DB, unlimited MAU | $25 |
| Cloudflare R2 | ~50GB storage | $0.75 |
| Upstash Redis | ~500k commands/day | $10 |
| Vercel Pro | Higher bandwidth | $20 |
| Railway | 2x replicas | $15 |
| EAS Build | 100+ builds/month | $99/year = $8/mo |
| Sentry Team | Higher volume | $26 |
| **TOTAL** | | **~$105/month** |

### Scale Phase (1,000+ users, 5,000+ jobs/month)
At this point, optimize based on actual bottlenecks. Likely moves:
- Migrate Railway to dedicated VPS (Hetzner ~$12/mo for 4vCPU/8GB RAM)
- Add read replica for PostgreSQL
- R2 costs scale linearly and remain cheap

---

## Key Technology Decisions & Tradeoffs

### Go for API (vs Node.js / Python)
**Pros:** 10–20x lower memory than Node.js equivalent → cheaper hosting. Single binary deployment. Strong concurrency for sync workloads.  
**Cons:** Smaller ecosystem for some integrations. Slightly more verbose.  
**Decision:** Go is the right call for a long-running service with high concurrency requirements (sync + notification workers).

### WatermelonDB (vs plain Expo SQLite / AsyncStorage)
**Pros:** Built-in sync protocol that maps directly to our Go endpoints. Lazy loading prevents mobile performance issues with large datasets. Proven at scale (used by Nozbe, Hey).  
**Cons:** Learning curve. Slightly heavy for simple apps.  
**Decision:** The sync protocol alone is worth it — saves 2–3 weeks of custom SQLite sync implementation.

### Supabase (vs plain PostgreSQL on Railway)
**Pros:** Free managed PostgreSQL. Auth built-in (saves implementing JWT refresh + token storage). Realtime for free. Row Level Security support.  
**Cons:** Vendor lock-in for RLS and Auth. Free tier has limits.  
**Decision:** Use Supabase for PostgreSQL and Auth only. Don't use Supabase Edge Functions or Storage (use R2 instead). Easy to migrate to plain Postgres later.

### Cloudflare R2 (vs AWS S3 / Supabase Storage)
**Pros:** Zero egress fees (vs S3's $0.09/GB). 10GB free storage. S3-compatible API.  
**Cons:** Fewer CDN edge locations than CloudFront.  
**Decision:** R2 is significantly cheaper at any scale. No egress fee is a major win for photo-heavy workloads.

### Expo Push (vs direct FCM/APNs)
**Pros:** Single API for both iOS and Android. Handles certificate rotation. Receipt checking built-in. Free.  
**Cons:** Expo's servers are a middleman — if they're down, pushes fail.  
**Decision:** Acceptable for MVP. Add direct FCM fallback in Phase 2 if reliability is a concern.

---

## Repository Structure

```
github.com/totasolution/
├── sigma-marketplace    (this repo — PRD, docs, project tracking)
├── sigma-api            (Go backend)
├── sigma-mobile         (React Native + Expo)
└── sigma-web            (Next.js)
```

## Development Environment

| Tool | Purpose |
|---|---|
| Docker Compose | Local PostgreSQL + Redis |
| golangci-lint | Go linting |
| ESLint + Prettier | JS/TS linting |
| Maestro | Mobile E2E testing |
| Playwright | Web E2E testing |
