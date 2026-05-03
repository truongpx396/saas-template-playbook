# 🚀 The SaaS Template Playbook

> A comprehensive, opinionated, actionable guide for building a **professional, reusable SaaS template** that you can fork and reskin for any vertical (CRM, project management, analytics, internal tooling, vertical SaaS, etc.).
>
> If you read only one section first, read **§3 The 12 Pillars** and **§5 Multi-Tenancy** — those two ideas dictate every other decision in this document.

---

## 📋 Table of Contents

1. [🧐 What "SaaS Template" Actually Means](#1-what-saas-template-actually-means)
2. [⚡ The 30-Second Mental Model](#2-the-30-second-mental-model)
3. [🏛️ The 12 Pillars of a Production SaaS](#3-the-12-pillars-of-a-production-saas)
4. [🏗️ Reference Architecture](#4-reference-architecture)
5. [🏢 Multi-Tenancy — the Keystone Decision](#5-multi-tenancy--the-keystone-decision)
6. [🔐 Authentication & Authorization](#6-authentication--authorization)
7. [👥 Accounts, Organizations, Workspaces, Teams](#7-accounts-organizations-workspaces-teams)
8. [🚪 Onboarding & Activation](#8-onboarding--activation)
9. [💳 Billing, Subscriptions & Metering](#9-billing-subscriptions--metering)
10. [🗄️ Database Design Patterns](#10-database-design-patterns)
11. [🌐 API Design](#11-api-design)
12. [⚙️ Background Jobs, Queues & Schedulers](#12-background-jobs-queues--schedulers)
13. [📡 Real-time & Eventing](#13-real-time--eventing)
14. [📨 Email, Notifications & Inbox](#14-email-notifications--inbox)
15. [📦 File Storage, Uploads & CDN](#15-file-storage-uploads--cdn)
16. [🔎 Search (Full-Text + Semantic)](#16-search-full-text--semantic)
17. [🚩 Feature Flags & Experiments](#17-feature-flags--experiments)
18. [📊 Audit Logs, Activity Feeds & Telemetry](#18-audit-logs-activity-feeds--telemetry)
19. [🛡️ Security, Compliance & Privacy](#19-security-compliance--privacy)
20. [⚡ Performance, Caching & Scaling](#20-performance-caching--scaling)
21. [📈 Observability — Logs, Metrics, Traces, Errors](#21-observability--logs-metrics-traces-errors)
22. [🎨 Frontend Architecture](#22-frontend-architecture)
23. [🌍 Internationalization & Accessibility](#23-internationalization--accessibility)
24. [🔧 Admin & Internal Tooling](#24-admin--internal-tooling)
25. [📝 Marketing Site, Docs & SEO](#25-marketing-site-docs--seo)
26. [🚢 CI/CD, Environments & Release Strategy](#26-cicd-environments--release-strategy)
27. [🧰 Developer Experience (DX)](#27-developer-experience-dx)
28. [🧪 Testing Strategy](#28-testing-strategy)
29. [💰 Pricing, Plans & Packaging Strategy](#29-pricing-plans--packaging-strategy)
30. [🎯 Product Analytics & Growth](#30-product-analytics--growth)
31. [🤝 Customer Support & Success](#31-customer-support--success)
32. [📦 Reusability — How to Make This a Template](#32-reusability--how-to-make-this-a-template)
33. [🗺️ The 14-Phase Build Plan](#33-the-14-phase-build-plan)
34. [⚠️ Common Pitfalls & Hard-Won Guardrails](#34-common-pitfalls--hard-won-guardrails)
35. [📋 Cheat Sheet](#35-cheat-sheet)

---

## 1. 🧐 What "SaaS Template" Actually Means

A reusable SaaS template is **the boring 80% you'd otherwise rebuild for every product**:

- Sign-up, login, password reset, SSO, MFA
- Organizations / workspaces / teams / invites
- Roles + permissions
- Billing, subscriptions, plans, usage metering, invoices
- Email + notifications + in-app inbox
- Audit logs + activity feeds
- Admin panel
- Feature flags
- Background jobs, scheduled jobs, webhooks
- File uploads + CDN
- API keys + rate limiting
- Observability + error tracking
- CI/CD + multi-environment deploys
- Marketing landing page + docs site

**It is NOT:**
- Your product's domain logic — that's the unique 20% you build on top.
- A no-code platform — it's a code starter.
- A magic SaaS-in-a-box — you still need product judgment.

The right mental model: **infrastructure for the parts every SaaS has, with clean seams where your domain plugs in.**

---

## 2. ⚡ The 30-Second Mental Model

```plaintext
                ┌─────────────────────────────────────┐
                │  Marketing Site  +  Docs  +  Status │
                └─────────────────────┬───────────────┘
                                      │
                ┌─────────────────────▼───────────────┐
                │            Web App (SPA)            │
                │       + (optional) Mobile/Desktop   │
                └────────┬─────────────────┬──────────┘
                         │ REST/GraphQL    │ WS/SSE
                ┌────────▼─────────────────▼──────────┐
                │  Edge / API Gateway                 │
                │   (auth, rate limit, CORS, WAF)     │
                └────────┬────────────────────────────┘
                         │
       ┌─────────────────┼─────────────────────────────┐
       ▼                 ▼                             ▼
  ┌────────┐       ┌──────────┐                 ┌──────────┐
  │ App API│ ◄───► │Worker(s) │                 │ Webhooks │
  │  (BFF) │       │+ Cron    │                 │ Out/In   │
  └───┬────┘       └────┬─────┘                 └────┬─────┘
      │                 │                            │
      ▼                 ▼                            ▼
  ┌─────────────────────────────────────────────────────┐
  │  Postgres (core)  •  Redis (cache+queue)            │
  │  Object Storage (S3)  •  Search (PG/Meili/Elastic)  │
  │  Time-series / Analytics (ClickHouse / DuckDB)      │
  └─────────────────────────────────────────────────────┘
                                  │
                  ┌───────────────┼─────────────────────┐
                  ▼               ▼                     ▼
              Stripe          Email (Resend)        Auth (Clerk/
              (billing)       SMS (Twilio)          WorkOS) [opt]
              Sentry          Segment/PostHog       OpenAI/etc.
```

**Three deployable surfaces, one source of truth:**

| Surface | Built from | Where it runs |
|---|---|---|
| Marketing + docs | Next.js static / Astro | CDN (Vercel / Cloudflare Pages) |
| Web app | React SPA (Vite) or Next.js | CDN + edge |
| API + workers | Go / Python / Node | Container platform (Fly / Railway / ECS / k8s) |

---

## 3. 🏛️ The 12 Pillars of a Production SaaS

Every SaaS template needs all twelve. Skip one, and you eat scope creep later.

| # | Pillar | What "done" looks like |
|---|---|---|
| 1 | **Identity** | Email/password, OAuth (Google/GitHub), magic link, MFA, SSO (SAML/OIDC), session + token model. |
| 2 | **Tenancy** | Org/workspace boundary, every query filtered by `workspace_id`, RBAC + (optional) ABAC. |
| 3 | **Billing** | Stripe wired, plans configurable, trials, dunning, usage metering, invoice portal. |
| 4 | **Lifecycle** | Onboarding flow, email verification, invites, offboarding, account deletion (GDPR-clean). |
| 5 | **Eventing** | In-process bus → outbox → workers → webhooks. Idempotent. |
| 6 | **Observability** | Structured logs + traces + metrics + error tracker, all correlated by `request_id` + `tenant_id`. |
| 7 | **Audit** | Append-only audit log of every privileged action, queryable by tenant. |
| 8 | **Notifications** | Transactional email + in-app inbox + (opt) SMS/push, all with per-user preferences. |
| 9 | **Files** | Direct-to-S3 uploads via signed URLs; never proxy bytes through your API. |
| 10 | **Admin** | Internal dashboard for support: impersonate, refund, suspend, inspect tenant. |
| 11 | **Flags** | Feature flags per environment + per tenant + per user. Kill-switch culture. |
| 12 | **DX** | One command to dev (`make dev`), seed data, fast tests, docs that don't lie. |

---

## 4. 🏗️ Reference Architecture

### 4.1 The Spine

```plaintext
          [Browser / Mobile / Desktop]
                       │
                       ▼
              [CDN / Edge Cache]
                       │
                       ▼
            [Reverse Proxy / WAF]   ← TLS terminates here
            (Caddy: automatic HTTPS via Let's Encrypt,
             or Traefik: dynamic routing from Docker/K8s labels)
                       │
            ┌──────────┼───────────┐
            ▼          ▼           ▼
     [API Gateway] [WebSocket]  [Static Assets]
            │          │
            ▼          ▼
       [App API (stateless, horizontally scalable)]
            │
   ┌────────┼─────────────┬─────────────┐
   ▼        ▼             ▼             ▼
 [DB]   [Cache]      [Queue]       [Object Store]
Postgres  Redis      Redis/SQS         S3
   │        │             │             │
   ▼        ▼             ▼             ▼
[Read    [Pub/Sub   [Workers +     [CDN signed
 replica] for WS]    cron]          URLs]
```

### 4.2 What lives where

| Concern | Where |
|---|---|
| Source of truth | Postgres |
| Hot reads, sessions, idempotency keys, rate-limit counters | Redis |
| Heavy/slow work, retries, scheduled work | Workers consuming a queue |
| Real-time fanout to clients | WS hub backed by Redis pub/sub (multi-node) |
| Bulk analytics & reporting | ClickHouse / BigQuery / DuckDB (mirrored from Postgres) |
| Static UI | CDN |
| User-uploaded files | S3 + CDN with signed URLs |
| Secrets | Env (dev) / SSM / Vault / Doppler (prod) |

### 4.3 Suggested tech stack (opinionated, swappable)

| Layer | Default | Why |
|---|---|---|
| API (Go) | **chi + sqlc + pgx** (lean) **or** **Gin + GORM** (batteries-included) | Fast, predictable, low-overhead. Gin/GORM is the path-of-least-resistance combo most Go SaaS teams ship on. |
| API (Node) | Hono / Fastify + Prisma | Edge-friendly, ergonomic |
| ML / heavy compute | **Python** (FastAPI + uv + pydantic v2 + **structlog**) | Ecosystem advantage; structlog gives you JSON logs out of the box |
| Web | **React 19 + TypeScript + Vite + TanStack Query + Zustand + Tailwind** | Boring, excellent, zero magic |
| DB | **Postgres 16+** (with `pgvector`, `pg_trgm`) | One DB to do 90% of jobs |
| Cache | **Redis 7** | Battle-tested |
| Queue / Eventing | **Redis** (simple) → **NATS JetStream** (durable streams, replay, KV, multi-tenant subjects) | NATS is the right answer when you need at-least-once delivery, replay, or fan-out across services without standing up Kafka. |
| Search | **Postgres FTS** (start) → Meilisearch / Typesense (scale) | Cheap → fast |
| Object store | **S3** / **Cloudflare R2** (no egress) / **Supabase Storage** (if you're already on Supabase) | Standard |
| Email | **Resend** or **Postmark** | Reliable transactional, simple SDKs |
| Auth (managed SaaS) | **Clerk** (fast UX), **WorkOS** (enterprise SSO/SCIM), **Supabase Auth** (if you want auth + DB + storage in one) | Saves weeks; pick by where the rest of your stack lives. |
| Auth (self-hosted OSS) | **Ory Kratos** (identity) + **Ory Hydra** (OIDC) + **Ory Keto** (permissions) — pure API, no UI bundled. **Casdoor** — full-stack IAM with built-in admin UI, OIDC/SAML, RBAC, MFA. | Own your identity layer without writing it. Kratos = composable primitives; Casdoor = drop-in IAM. |
| Auth (DIY) | Lucia / Auth.js / your own JWT + refresh | Maximum ownership, maximum maintenance |
| Billing | **Stripe** (default) / **Paddle** or **LemonSqueezy** (Merchant-of-Record, global tax) / **PayPal** (add as a secondary payment method when you have non-card markets — LATAM, parts of EU, gamer/creator audiences) | Stripe owns card-first markets; PayPal is the second checkout option customers ask for. |
| Logging (Go) | **zerolog** (zero-allocation JSON) or `slog` (stdlib, 1.21+) | zerolog is the production default for Go SaaS — fast, structured, contextual. |
| Logging (Python) | **structlog** + `orjson` renderer | Structured, contextvars-aware, async-safe |
| Background jobs | **Asynq** (Go, Redis) / **River** (Go, Postgres) / **BullMQ** (Node) / **Celery / Arq** (Python) / **NATS JetStream consumers** (cross-language) | Match language, or use NATS if you already have it for eventing. |
| Reverse proxy / TLS | **Caddy** (automatic HTTPS, simplest config) **or** **Traefik** (dynamic config, great with Docker/K8s/labels) — **nginx** if you have a reason. | Caddy = "it just works" for VMs. Traefik = service-discovery-driven for containerized stacks. |
| Observability | **OpenTelemetry → Grafana / Honeycomb / Datadog** | Vendor-neutral export |
| Errors | **Sentry** | Best-in-class |
| Analytics | **PostHog** (self-host or cloud) | Product + flags + session replay in one |
| CI/CD | **GitHub Actions** | Where your code already is |
| Infra (PaaS, fastest start) | **Fly.io / Railway / Render** | Push-to-deploy, no ops |
| Infra (cheap VMs, more control) | **Hetzner** (best €/CPU in the market — €4–€40/mo dedicated cores) **or** **Digital Ocean** (polished UX, managed PG/Redis, App Platform) | Most bootstrapped SaaS run profitably on a Hetzner box + DO managed Postgres. Pair with Caddy/Traefik. |
| Infra (hyperscaler, when you have to) | **AWS / GCP / Azure** | Compliance, region breadth, enterprise procurement |

> **Two reference stacks to pick from on day one:**
> - **"Bootstrapped solo / small team":** Go (Gin + GORM + zerolog) + Postgres + NATS JetStream + Caddy on a single Hetzner box, Casdoor or Ory Kratos for auth, Stripe + PayPal for payments. ~€30/mo, scales to thousands of paying customers.
> - **"Funded / enterprise-ready":** Go (chi + sqlc) + managed Postgres + Redis + NATS cluster behind Traefik on Digital Ocean App Platform / Kubernetes, WorkOS or Supabase Auth, Stripe Billing, OTel → Grafana Cloud.

### 4.4 Cross-cutting building blocks (the glossary)

These are the load-bearing concepts every later section assumes. Define them once here; deeper coverage is in the linked sections.

#### 🧱 The middleware chain

A request flows through a fixed stack of middleware before any handler runs. **Order is load-bearing — wire it once in `main.go` and don't rearrange.**

```plaintext
Request
  │
  ▼
[1] Recovery        — catch panics, return 500 + Sentry capture
[2] RequestID       — generate or accept X-Request-ID header
[3] Logger          — bind request_id to ctx logger (zerolog/structlog)
[4] Tracing         — OTel span for the request
[5] CORS            — allowlist origins
[6] RateLimit       — Redis token bucket per IP / API key (§11.7)
[7] Auth            — verify session/JWT/API key → set Actor in ctx (§6)
[8] Tenant          — resolve workspace_id → set in ctx + SET LOCAL app.workspace_id (§5)
[9] CSRF            — cookie endpoints only
[10] Idempotency    — POSTs with Idempotency-Key header (§11.6)
  │
  ▼
Handler → Service → Repository
  │
  ▼
Response
  │
  ▼
[Logger middleware closes the span, emits access log line]
```

Auth comes **before** Tenant (you need an actor before resolving their workspace). Recovery is outermost so a panic anywhere still produces a clean 500. RateLimit goes before Auth so unauthenticated abuse hits the limiter first.

#### 📦 What `ctx` carries

`context.Context` is the request-scoped envelope. Everything below is bound by middleware and read by handlers/services/repos.

| Key | Set by | Read by |
|---|---|---|
| `request_id` | RequestID middleware | logs, error responses, traces |
| `logger` | Logger middleware | every layer (`log.Ctx(ctx)`) |
| `actor` | Auth middleware | permission checks, audit log |
| `workspace_id` | Tenant middleware | every repo query, RLS GUC |
| `trace_id` / `span` | OTel middleware | downstream HTTP/DB instrumentation |
| `db` (per-request handle with GUCs set) | Tenant middleware | repos |

**Rule:** if a function needs any of these, it takes `ctx context.Context` as the first argument. No globals. No `req.Context()` 3 layers deep — pass `ctx` explicitly.

#### 🎭 The `Actor` type (polymorphic identity)

Every action in the system is performed by something — a human, an API key, or the system itself. Don't model "user" everywhere; model `Actor`.

```go
type Actor struct {
    Type ActorType // user | api_key | system
    ID   uuid.UUID
    // for users: cached membership in current workspace
    Role        Role     // owner | admin | member | viewer
    Permissions []string // resolved at auth time
}

func (a *Actor) Can(action string, resource Resource) bool { /* §6.3 */ }
```

This pairs with the polymorphic-actor DB pattern (`created_by_type`, `created_by_id` — see §35) so audit logs, activity feeds, and `created_by` fields handle integrations and humans uniformly.

#### 🏛️ Layered architecture (handler → service → repo)

Each layer has a strict allowed-imports list. Violations are caught by `golangci-lint` `depguard` rules (or equivalent in other languages).

| Layer | Knows about | Forbidden |
|---|---|---|
| **Handler** | HTTP, Service interfaces, request/response DTOs | DB, SQL, third-party SDKs |
| **Service** | Domain logic, other Services, Repository interfaces, the `Bus` | HTTP types (`http.Request`, `gin.Context`) |
| **Repository** | DB driver, SQL, models | HTTP, business rules, other repos |

A handler **never** touches the DB. A repo **never** decides whether an action is allowed. This is what makes services testable without a server and repos swappable.

#### 🔌 The kernel interfaces (the seams)

Every cross-cutting capability is a Go interface (or TS type) defined in `kernel/`. The product imports the interface; wiring picks the implementation at startup. **These are the seams that keep the template reusable.**

```go
type Auth interface {                         // §6
    Authenticate(ctx, token) (*Actor, error)
    Issue(ctx, user *User) (Token, error)
}

type Bus interface {                          // §13
    Publish(ctx, subject string, payload []byte) error
    Subscribe(ctx, subject string, h Handler) (Subscription, error)
}

type Storage interface {                      // §15
    PresignPut(ctx, key string, opts PutOpts) (string, error)
    PresignGet(ctx, key string, ttl time.Duration) (string, error)
}

type Mailer interface {                       // §14
    Send(ctx, msg Message) error
}

type Meter interface {                        // §9.6
    Increment(ctx, workspaceID uuid.UUID, metric string, n int64) error
}

type Flags interface {                        // §17
    IsEnabled(ctx, key string, scope FlagScope) bool
}

type Cache interface {                        // §20
    Get(ctx, key string) ([]byte, bool, error)
    Set(ctx, key string, val []byte, ttl time.Duration) error
    Bump(ctx, tag string) error // tag-based invalidation
}
```

Implementations: `casdoor.Auth`, `workos.Auth`, `kratos.Auth` / `nats.Bus`, `redis.Bus`, `inproc.Bus` / `s3.Storage`, `r2.Storage`, `supabase.Storage` / `resend.Mailer`, `postmark.Mailer` / etc. Swapping providers = changing one line in `main.go`.

#### 🔒 Transactions: the `WithTx` pattern

Don't manually `Begin/Commit/Rollback` — it leaks on panics and confuses nested calls. Use a closure helper that the repo layer owns:

```go
func (r *Repo) WithTx(ctx context.Context, fn func(tx *Repo) error) error {
    return r.db.Transaction(func(db *gorm.DB) error {
        return fn(&Repo{db: db})
    })
}

// Service:
err := repo.WithTx(ctx, func(tx *Repo) error {
    if err := tx.Orders().Create(ctx, order); err != nil { return err }
    return tx.Outbox().Append(ctx, "order.created", order) // §12.4
})
```

Two rules:
- **Never hold a transaction across a network call** (HTTP, Stripe, S3). Read first, do external work, then write fast inside the tx.
- **DB writes + event emission live in the same tx** via the outbox pattern (§12.4). Anything else is eventually-inconsistent in failure modes.

#### 🔁 Idempotency (everywhere, not just §11.6)

Three places idempotency shows up; same idea, different keys:

| Surface | Key | Storage |
|---|---|---|
| Public API `POST` | `Idempotency-Key` header (§11.6) | Redis, 24h TTL, scoped by `(workspace_id, key)` |
| Stripe/PayPal webhooks | `event.id` (§9.3) | Redis, 7-day TTL |
| Background jobs | `(job_type, dedup_key)` (§12.3) | Postgres unique index, or Redis SETNX |

The shape is always: check if you've seen this key → if yes, return cached result / no-op → else do work, then record the key.

#### 🆔 ID conventions

- **`UUID v7`** for all primary keys — sortable by time, single column for PK + chronology, no `created_at` index needed for ordering.
- **Prefixed display IDs** in API responses for human-readable references: `proj_01HMZ...`, `inv_01HMZ...`. The DB stores the raw UUID; the API serializer adds the prefix. Saves debugging time when a customer pastes an ID into a ticket.

#### 🌍 The standard handler shape

Every handler in the codebase looks the same. Deviation = reviewer flag.

```go
func (h *ProjectHandler) Create(c *gin.Context) {
    ctx := c.Request.Context()
    actor := auth.ActorFrom(ctx)            // set by Auth middleware
    workspaceID := tenant.IDFrom(ctx)       // set by Tenant middleware

    var req CreateProjectRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        respondError(c, errs.Validation(err)); return
    }

    project, err := h.svc.Create(ctx, actor, workspaceID, req)
    if err != nil {
        respondError(c, err); return         // single error envelope (§11.5)
    }

    c.JSON(201, project)
}
```

Five lines of mechanical work, then one line of actual business logic delegated to the service. If a handler grows past 20 lines, push the logic down a layer.

The single most consequential architectural choice. Decide at day one and **enforce in code**.

### 5.1 The three models

| Model | Description | When to use |
|---|---|---|
| **Pool (shared)** | One DB, every row tagged `workspace_id` (or `org_id`). | Default for B2B SaaS. Best ops/cost. |
| **Bridge (silo schema)** | One DB, one schema per tenant. | Mid-enterprise; per-tenant migrations possible. |
| **Silo (isolated DB)** | One DB per tenant. | Regulated tenants (banks, healthcare), VIP customers. |

**Recommendation:** Start with **Pool**. Add Silo later as an enterprise tier. Don't try to do all three on day one.

### 5.2 Hard rules for the Pool model

1. **Every tenant-owned table has `workspace_id` (or `org_id`) NOT NULL.**
2. **Every query filters by `workspace_id` — no exceptions.** Enforce via:
   - Repository methods that *require* `workspaceID` as a typed argument.
   - Postgres Row-Level Security (RLS) as a belt-and-suspenders defense.
3. **The active tenant is resolved once per request from the auth token** and stored in `context.Context` / request-local state.
4. **Cross-tenant queries (admin, analytics) go through a separate, audited code path.** Never inside the user request handler.

### 5.3 Postgres RLS as defense-in-depth

```sql
ALTER TABLE issue ENABLE ROW LEVEL SECURITY;

CREATE POLICY issue_tenant_isolation ON issue
    USING (workspace_id = current_setting('app.workspace_id')::uuid);
```

In your handler middleware:

```go
tx.Exec(`SET LOCAL app.workspace_id = $1`, workspaceID)
```

Even if a developer forgets a `WHERE workspace_id = ?`, RLS blocks the leak.

### 5.4 The "two-actor" rule for queries

Every query has two implicit parameters:
- `actor_user_id` (who's asking)
- `tenant_id` (which tenant they're acting in)

Don't accept "logged-in user" alone. The same user can belong to multiple workspaces.

### 5.5 Tenant resolution

Either:
- **Subdomain**: `acme.app.yourtool.com` → `acme` → workspace lookup.
- **Path**: `app.yourtool.com/w/acme/...`
- **Header**: `X-Workspace-ID: <uuid>` (good for APIs, but UI needs a workspace switcher).

Most SaaS pick subdomain or path — pick one and stick with it.

---

## 6. 🔐 Authentication & Authorization

### 6.1 Auth methods you must support

- **Email + password** (always — even if SSO available).
- **Magic link** (best UX for low-stakes products).
- **OAuth**: Google + GitHub minimum. Apple if iOS app.
- **MFA**: TOTP (Authenticator apps) — easy to add, big trust signal.
- **Passkeys (WebAuthn)** — increasingly expected.
- **SSO (SAML 2.0 + OIDC)** — gate behind enterprise plan; outsource to **WorkOS** or **Clerk** unless you want to own the support burden.
- **API keys** — per-workspace, scoped, revocable, hashed at rest (`sha256`).
- **Personal access tokens (PATs)** — for CLIs, with rotation.

### 6.2 Sessions vs JWTs — pick a hybrid

| Use case | Mechanism |
|---|---|
| Browser session | **HttpOnly secure cookie** with opaque session ID → server-side session in Redis. Easy revocation. |
| Mobile / desktop / CLI | **Short-lived JWT (15 min) + refresh token** stored securely. |
| Public API | **API key** (long-lived, scoped, revocable). |
| Service-to-service | **mTLS** or signed JWT with short TTL. |

**Rule:** JWT *or* server-side session — pick per surface. Don't mix-and-match within one surface.

### 6.3 Authorization — RBAC, then ABAC if needed

Start with **role-based access control (RBAC)**:

```plaintext
Workspace roles: owner | admin | member | viewer
Resource permissions derived from role
```

Only add **attribute-based access control (ABAC)** (e.g., "user X can edit only resources where `assignee_id = user.id`") when RBAC alone produces unmaintainable conditionals.

```go
// Permission helper signature
func Can(actor *Actor, action string, resource Resource) bool
```

Centralize all permission logic in one package. Never inline `if user.Role == "admin"` checks in handlers.

### 6.4 Open-source policy engines

- **Casbin** — Go, lightweight, RBAC + ABAC.
- **OPA (Open Policy Agent)** — sidecar, enterprise-grade.
- **Oso** — embedded, declarative.
- **Ory Keto** — Google Zanzibar–style relationship-based access control as a service.

For a template, hand-rolled `Can()` is fine until you hit ~20 permission rules.

### 6.5 Don't-build-it-yourself: managed & self-hostable identity

Auth is a tarpit. Ship a real identity service before you ship your second feature. Pick by where you want the trust boundary:

| Option | Type | Sweet spot | Watch out for |
|---|---|---|---|
| **Clerk** | Managed SaaS | B2C/PLG products that want pre-built React components and great DX. | Per-MAU pricing scales painfully past ~50k actives. |
| **WorkOS** | Managed SaaS | B2B selling into mid-market/enterprise — SSO (SAML/OIDC), SCIM, directory sync, audit log API. | Light on consumer-style password/magic-link flows; pair with Clerk or your own for those. |
| **Supabase Auth** (GoTrue) | Managed *or* self-hosted | You're already using Supabase Postgres + Storage; auth comes "free" with RLS hooks wired in. | You're now Supabase-shaped; migrating off later isn't trivial. |
| **Casdoor** | Self-hosted OSS | Single binary IAM with a built-in admin UI. OIDC/OAuth2/SAML/CAS providers, RBAC/ABAC, MFA, social logins, webhooks. | UI is functional, not premium — usually fine since admins use it, not end users. |
| **Ory Kratos** + **Hydra** + **Keto** | Self-hosted OSS | API-first, headless, composable. Kratos = identity + flows, Hydra = OIDC/OAuth2 server, Keto = permissions. You bring your own UI. | More moving parts; budget a week to wire flows + UI. |
| **Authentik / Zitadel / Keycloak** | Self-hosted OSS | Alternatives in the same shape as Casdoor — pick on UX preference and language affinity. | Keycloak is JVM-heavy; Authentik/Zitadel are lighter. |

**Template recommendation by audience:**

- **Solo / bootstrapped:** start with **Casdoor** (one container, admin UI, OIDC works in 30 minutes) or **Supabase Auth** if you want DB + auth co-located.
- **Funded B2B:** **WorkOS** for SSO/SCIM + your own password/magic-link, or **Ory Kratos** if you must self-host for compliance.
- **Consumer-facing PLG:** **Clerk** for the fastest path to a polished sign-in experience.

Your app should talk to identity through a thin **`auth` package interface** (`Authenticate(token) → Actor`, `Issue(ctx, user) → token`). Swapping Casdoor for WorkOS later is then a ~1-day adapter change, not a rewrite.

### 6.6 Auth security checklist

- [ ] Passwords hashed with `argon2id` (or bcrypt cost 12+).
- [ ] Email enumeration defended (same response for "email not found" and "wrong password").
- [ ] Rate limiting on `/login` (5/min/IP + 10/hr/email).
- [ ] Lockout after N failed attempts, with email notification.
- [ ] CSRF protection on cookie-auth endpoints.
- [ ] Session fixation defense: rotate session ID on login.
- [ ] Logout invalidates server-side session.
- [ ] Refresh tokens rotated on use; revoke entire family on reuse-detection.
- [ ] Password reset tokens are single-use, expire in 1h, are sent to verified email only.
- [ ] MFA backup codes generated, shown once, hashed at rest.

---

## 7. 👥 Accounts, Organizations, Workspaces, Teams

### 7.1 The canonical hierarchy

```plaintext
User  ─┬─►  Membership  ─►  Workspace (tenant)
       │                       │
       │                       ├── Teams (subgroups)
       │                       ├── Resources (projects, issues, …)
       │                       ├── Subscription (Stripe)
       │                       └── Settings (branding, SSO, etc.)
       │
       └─►  Personal account (optional — for solo plans)
```

A `User` is a global identity. A `Membership` ties a user to a workspace with a role.

### 7.2 Required tables (minimum)

```sql
user (id, email, password_hash, email_verified_at, mfa_enabled, created_at, ...)
workspace (id, slug, name, plan, owner_user_id, created_at, ...)
membership (id, user_id, workspace_id, role, status, invited_by, joined_at)
invite (id, workspace_id, email, role, token_hash, expires_at, accepted_at)
team (id, workspace_id, name, parent_team_id NULL)
team_membership (id, team_id, user_id, role)
api_key (id, workspace_id, name, prefix, hash, scopes JSONB, created_by, last_used_at, revoked_at)
```

### 7.3 Invites

- Email a single-use signed token (expires in 7 days).
- Accepting creates the `membership` row.
- **Critical:** if invitee already has an account, just attach a membership — don't force a separate signup flow.

### 7.4 Workspace switcher UI

A persistent UI element (sidebar dropdown or top nav) that:
- Shows current workspace.
- Lets user switch (changes URL: `/w/<slug>/...`).
- Lets user create a new workspace.
- Cache the active workspace ID per-user in a cookie/localStorage so it survives reloads.

### 7.5 Offboarding & deletion

- **Delete account**: GDPR right-to-be-forgotten. Anonymize PII, retain audit log entries with `user_id = NULL` + `display_name = "Deleted user"`.
- **Leave workspace**: just removes the membership row.
- **Delete workspace**: 30-day soft-delete with restore option. Hard-delete after grace period via cron.

---

## 8. 🚪 Onboarding & Activation

The 5-minute window between sign-up and first value is the highest-leverage UX you'll ever build.

### 8.1 The signup flow

```plaintext
1. /signup → email + password (or OAuth)
2. Send verification email immediately (but don't block app entry on it)
3. Land in "create your workspace" step
4. Land in product with one-time guided tour
5. Trigger first-aha-moment within ≤ 3 clicks
```

### 8.2 Activation events

Define **the activation event** — the action that predicts retention. Examples:
- Slack: send 2,000 team messages
- Dropbox: upload 1 file
- Linear: create 3 issues
- Figma: invite 1 collaborator

Track this as `activated_at` on the workspace, fire it from your event bus, and **trigger lifecycle emails** off it.

### 8.3 Email verification — required vs optional

- **Required for sensitive actions** (billing, inviting users, API keys).
- **Optional for read-only browsing**.
- Show a banner ("Verify your email — we sent a link to alice@…") and a one-click resend button.

### 8.4 Sample data / templates

For B2B SaaS, ship with a **demo workspace** that's pre-populated. Lets new users explore before they set up their own data.

### 8.5 Empty states are product surface

Every list view (`/issues`, `/projects`, …) needs an empty state with:
- One sentence of context ("No issues yet — issues are how you track work").
- A primary CTA button.
- An optional "import from CSV / Linear / Jira" hook.

---

## 9. 💳 Billing, Subscriptions & Metering

### 9.1 Use Stripe. (Or Paddle / LemonSqueezy if you want them to handle global tax.)

Don't build billing yourself. Stripe has solved every edge case you'd hit in year three.

**On PayPal:** Stripe is the default subscription engine. **PayPal is a checkout option, not a billing system.** A meaningful slice of customers — LATAM, parts of Asia/EU, freelancer/creator markets, B2C audiences who don't want to hand over a card — will bounce if PayPal isn't there. The right shape is:

- **Subscriptions ledger lives in your DB.** Plan, status, period, seats — your tables, your truth.
- **Stripe** for cards / Apple Pay / Google Pay / SEPA / ACH (subscription billing via Stripe Billing).
- **PayPal Subscriptions API** wired as a *parallel* payment provider — same `subscription` row, different `payment_provider` column.
- **One webhook handler per provider** writing into the same idempotent state machine. Don't try to unify webhooks; unify the *resulting* state.

```sql
subscription (
    id UUID PK,
    workspace_id UUID,
    plan_id UUID,
    status TEXT,                    -- trialing | active | past_due | canceled
    payment_provider TEXT,          -- 'stripe' | 'paypal' | 'manual'
    provider_subscription_id TEXT,  -- stripe sub_… / paypal I-…
    provider_customer_id TEXT,
    current_period_end TIMESTAMPTZ,
    cancel_at TIMESTAMPTZ NULL,
    ...
)
```

Skip PayPal until a real customer asks for it twice. Then add it behind a feature flag and offer it only on the plan-selection page.

### 9.2 Required Stripe surfaces

| Surface | Stripe product |
|---|---|
| Plan selection at signup | **Stripe Checkout** (hosted) |
| In-app upgrade/downgrade | **Stripe Billing Portal** (hosted) — or build your own using the API |
| Usage-based billing | **Metered prices** |
| Trials | Set `trial_period_days` on subscription |
| Discounts / coupons | Stripe coupons + promotion codes |
| Invoices, payment methods, receipts | Customer Portal handles all this for free |

### 9.3 The webhook contract

Subscribe to (at minimum):
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.paid`
- `invoice.payment_failed`
- `customer.updated`
- `checkout.session.completed`

**Idempotency rule:** every webhook handler must be idempotent. Stripe will retry. Use the `event.id` as a dedup key.

### 9.4 Plan model

```sql
plan (id, name, stripe_price_id, monthly_price_cents, yearly_price_cents, features JSONB, limits JSONB)
subscription (id, workspace_id, stripe_subscription_id, stripe_customer_id, plan_id, status, current_period_end, cancel_at, ...)
usage_record (id, workspace_id, metric, quantity, recorded_at, billed_at)
```

`features` and `limits` should be JSONB so you can add new feature gates without migrations:

```json
{
  "features": { "sso": false, "audit_log_export": false, "custom_domains": false },
  "limits":   { "members": 10, "projects": 5, "ai_credits_per_month": 1000 }
}
```

### 9.5 Feature gating

```ts
// Single helper, used everywhere
if (!can(workspace, "feature.sso")) {
  return upgradePrompt("SSO is available on the Team plan and above");
}
```

Every paywall is a `can()` check + a UI prompt. **Never silently 403.**

### 9.6 Metering

For usage-based pricing (AI credits, API calls, storage GB, …):

```go
// In the request path, fast and non-blocking:
meter.Increment(ctx, workspaceID, "ai.tokens", n)
```

`meter.Increment` writes to Redis (incr counter) + buffers writes to Postgres / Stripe in the worker. **Never call Stripe synchronously in the request path.**

### 9.7 Dunning (failed payments)

- 1st failure: email "We couldn't charge your card."
- 3rd failure (~7 days): downgrade to free + email.
- 30 days unpaid: suspend workspace (read-only) + email.
- 60 days: hard-delete or hand to collections.

Stripe handles the retry schedule (Smart Retries) — you handle the in-app messaging.

### 9.8 Trials done right

- **Length:** 14 days is the cultural norm. Don't overthink it.
- **Card upfront vs not:** card-up-front filters tire-kickers (lower volume, higher conversion); no-card maximizes top-of-funnel. **For B2B SaaS template, default to no-card with trial countdown banners.**
- **Trial extension:** offer once, free, no questions. ("Need more time? Extend 7 days.")
- **Trial expiration UX:** read-only mode + upgrade banner. Don't delete data.

### 9.9 When you'd outgrow Stripe-direct: Merchant-of-Record platforms

Stripe leaves *you* responsible for global tax (VAT, GST, US state sales tax). Below ~$1M ARR or with US-only customers, that's fine. Beyond that, or if you sell into the EU/UK as a non-resident, the compliance overhead becomes a real cost — at which point a Merchant-of-Record (MoR) sells the product *to* the customer and *from* you, taking the tax problem off your plate.

| Option | Type | Sweet spot | Watch out for |
|---|---|---|---|
| **Paddle** | Managed MoR | Established (15+ years), broad payment-method coverage, good for B2B SaaS selling globally. | Higher fees than raw Stripe (~5% all-in vs ~2.9% + 30¢); less granular control over the checkout. |
| **LemonSqueezy** | Managed MoR (Stripe-owned since 2024) | Indie/SMB-friendly, simple pricing, good license-key + digital-product support. | Acquired by Stripe — long-term roadmap may converge with Stripe Tax. |
| **Polar** | OSS + managed MoR | Open-source, developer-focused, optimized for indie hackers and dev-tool SaaS. Native usage-based billing, GitHub integration, customer benefits/perks built in. The right pick when you want MoR + a tool that feels native to a dev-first product. | Younger than Paddle/LMSqueezy; smaller ecosystem of integrations. Verify supported regions/payment methods match your market. |
| **Stripe Tax** (add-on, not MoR) | Managed | You stay the merchant of record but Stripe calculates and (in some jurisdictions) files tax for you. The middle ground. | Doesn't solve "non-resident seller of digital services in the EU" — you're still the entity registered for VAT. |

**Decision rule:** stay on raw Stripe until tax compliance starts costing you 1+ engineer-week per quarter. Then go MoR. Polar is the right default for **indie / dev-tool / open-core** SaaS; Paddle/LemonSqueezy for **broader B2B**.

The same pattern as PayPal (§9.1): your `subscription` table is provider-agnostic — `payment_provider TEXT` distinguishes `stripe` / `paypal` / `polar` / `paddle`. Switching MoRs later is a webhook-handler swap, not a rewrite.

---

## 10. 🗄️ Database Design Patterns

### 10.1 Conventions

- Singular table names (`user`, `issue`) — matches Go struct naming.
- Every table has: `id` (UUID v7 — sortable), `created_at`, `updated_at`, and `workspace_id` (if tenant-scoped).
- UUID v7 is sortable by time → primary key + chronological order in one column.
- Soft delete: `deleted_at TIMESTAMPTZ NULL` with a partial unique index where `deleted_at IS NULL`.
- Append-only history tables for things that need provenance (audit log, billing events, webhooks).

### 10.2 Migrations

- **Always forward.** Never edit an applied migration. Create a new one to fix mistakes.
- Use `goose` or **`golang-migrate`** (Go — both fine; `golang-migrate` ships a CLI + library + Docker image and supports many DB drivers, `goose` has nicer Go-based migrations) / `alembic` (Python) / `prisma migrate` / `drizzle-kit` / **Atlas** (declarative, language-agnostic).
- Number them sequentially: `001_init.up.sql`, `002_add_invites.up.sql`, ….
- Run automatically on deploy (with a deploy gate / dry-run for prod).
- **Online migrations**: never block writes on a hot table. Add column nullable → backfill in batches → add NOT NULL in a later migration.

### 10.3 Indexes that pay rent

- Every foreign key.
- Every `WHERE` clause column you actually filter on (run `EXPLAIN ANALYZE`).
- `(workspace_id, status, created_at DESC)` for typical "list X for tenant" queries.
- Partial indexes for soft delete: `WHERE deleted_at IS NULL`.

### 10.4 Transactions

- Wrap every multi-write operation in a transaction.
- **Use the outbox pattern** for cross-service events (see §13.3).
- Don't hold transactions open across HTTP/RPC calls. Read first, do external work, write fast.

### 10.5 Ergonomics

- Use **sqlc** (Go) / **Prisma** (TS) / **SQLAlchemy 2.0 + Alembic** (Python). Skip ORMs that hide SQL.
- Co-locate migrations and queries in the repo; check them in.
- Seed scripts for local dev that create realistic data (`make seed`).

---

## 11. 🌐 API Design

### 11.1 REST is the default; GraphQL is the exception

- **REST + JSON** for 90% of endpoints. Predictable, cacheable, debuggable.
- **GraphQL** if you have a complex, deeply-nested data graph and many client surfaces. Otherwise it's overhead.
- **gRPC** for service-to-service inside your infra.

### 11.2 Resource conventions

```plaintext
GET    /api/v1/projects                 list
POST   /api/v1/projects                 create
GET    /api/v1/projects/:id             read
PATCH  /api/v1/projects/:id             partial update (preferred over PUT)
DELETE /api/v1/projects/:id             delete
GET    /api/v1/projects/:id/issues      sub-collection
POST   /api/v1/projects/:id/issues      create in sub-collection
```

### 11.3 Pagination

- Cursor-based (`?cursor=<opaque>&limit=50`) — not offset. Offsets break under concurrent inserts.
- Return `{ items: [], next_cursor, has_more }`.
- Cap `limit` at 100.

### 11.4 Filtering & sorting

```http
?status=open&priority=high&sort=-created_at&limit=50
```

Document supported filters per endpoint. Reject unknown query params (don't silently ignore — typos won't surface).

### 11.5 Error envelope (one shape, everywhere)

```json
{
  "error": {
    "code": "validation_error",
    "message": "Title is required",
    "fields": { "title": "must not be empty" },
    "request_id": "req_01HMZ..."
  }
}
```

Include `request_id` in every response (header + body) so support can grep your logs.

### 11.6 Idempotency

- For `POST` endpoints that create resources or trigger side effects, accept an `Idempotency-Key` header.
- Cache `(workspace_id, idempotency_key) → response` in Redis for 24h.
- Return the cached response on retry. Stripe's the canonical example.

### 11.7 Rate limiting

- Per API key + per IP + per workspace.
- Token bucket in Redis (`INCR` + `EXPIRE`).
- Return `429` with `Retry-After` header.
- Document limits in your API docs and surface them in the response headers (`X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`).

### 11.8 Versioning

- URL versioning (`/api/v1/`, `/api/v2/`) — boring, works.
- Or header-based (`Accept: application/vnd.yourtool.v2+json`) — fancy, more work.
- Never break v1 once published. Add v2 alongside.

### 11.9 OpenAPI

- Maintain a hand-written or generated OpenAPI 3.1 spec.
- Generate client SDKs from it (`openapi-generator`, `oapi-codegen`).
- Render docs with Stoplight / Redoc / Mintlify.

### 11.10 Webhooks (outgoing)

- Per-workspace endpoints registered in settings.
- Sign every payload: `X-Signature: sha256=<hmac(body, secret)>`.
- Include `X-Event-Id` (idempotency) and `X-Timestamp` (replay defense).
- Retry with exponential backoff (1m, 5m, 30m, 2h, 12h) — fail and notify after final retry.

---

## 12. ⚙️ Background Jobs, Queues & Schedulers

### 12.1 Three job categories

| Category | Examples | Constraint |
|---|---|---|
| Async (fire-and-forget) | Send email, post to webhook, sync to CRM | Must be retried on failure |
| Scheduled | Daily reports, dunning emails, data exports | Must run within window, not on hot path |
| Long-running | Imports, AI batch jobs, video transcode | Need progress tracking + cancellation |

### 12.2 Job system

- Pick one library per language and stick to it.
- Go: **River** (Postgres-backed, transactional) or **Asynq** (Redis-backed).
- Python: **Arq** (asyncio + Redis) or **Celery** (mature, heavy).
- Node: **BullMQ**.

### 12.3 Idempotency

Every handler must tolerate being called twice. Use a `(job_type, dedup_key)` unique key, or check-then-act inside a transaction.

### 12.4 Outbox pattern

When you need "DB write + event emission" to be transactional:

```sql
INSERT INTO order ...;
INSERT INTO outbox (event_type, payload) VALUES ('order.created', '...');
COMMIT;
```

A separate worker polls `outbox`, fires the event (queue / webhook / Stripe sync), marks it done.

### 12.5 Cron / scheduled jobs

- Use a single, *deduplicated* scheduler — not `cron` per box (you'll get duplicate runs on multi-instance deploys).
- Postgres-backed `pg_cron` or library-level (`robfig/cron` + leader election) work fine.
- Every scheduled job logs its run + duration to a `cron_run` table for visibility.

### 12.6 Long-running progress

For jobs the user can see ("Importing 50,000 contacts…"):
- Persist a `job` row with `status`, `progress_pct`, `total`, `current`, `result`, `error`.
- Worker updates progress every N items / N seconds.
- UI polls `GET /jobs/:id` or subscribes via WS.

### 12.7 The tier above queues: durable execution engines

A queue (Asynq, BullMQ) gives you "run this function later, retry on failure." That's enough for 80% of SaaS work. But once your jobs become **multi-step workflows that can pause for hours, fan-out and join, survive worker crashes mid-step, and need exactly-once guarantees end-to-end** (think: subscription onboarding flow, multi-day customer pipeline, agent runs that pause for human approval), a queue starts to bend. You end up rebuilding state machines, sagas, and resumability on top of it. That's the signal to step up to a *durable execution* engine.

| Tool | Type | Sweet spot | Watch out for |
|---|---|---|---|
| **Temporal** | OSS, self-host or Temporal Cloud (managed) | The category leader. Workflows-as-code in Go/TS/Python/Java/.NET, deterministic replay, built-in retries/timeouts/heartbeats/sagas/signals/queries. The right pick for serious multi-step orchestration (billing flows, KYC, ETL pipelines, long-running agents §18 of the AI playbook). | Operationally non-trivial — Temporal cluster needs Cassandra/PostgreSQL + history service + matching service. Use Temporal Cloud (~$200/mo starter) until you have a reason not to. Workflow code must be deterministic — surprising at first. |
| **Hatchet** | OSS, Postgres-backed | Temporal-shaped (durable workflows, retries, fan-out, human-in-the-loop) but runs on **just Postgres** — no separate cluster. Excellent fit for teams that already have Postgres and don't want to operate Temporal. Python and TS SDKs, Go in progress. | Younger project, smaller ecosystem. Postgres becomes a hot bottleneck at very high workflow volume — fine for thousands/sec, not millions. |
| **Inngest** | Managed (OSS dev tools) | Step-functions-style workflows in TS/Python, focused on developer ergonomics and event-driven triggers. Best for serverless/Vercel-shaped stacks. | Less control if you self-host; managed pricing scales with executions. |
| **Restate** | OSS, single binary | Newer durable execution runtime focused on simplicity (single binary, deterministic) with TS/Java/Kotlin/Python/Go/Rust SDKs. Worth watching. | Smaller community than Temporal/Hatchet today. |

**When to pick a durable execution engine over a queue:**

- A workflow has ≥3 steps, any of which can be retried independently.
- A workflow needs to pause and wait — for an external webhook, a human approval, a timer measured in hours/days.
- "If the worker crashes mid-step, the work must continue from exactly where it left off" is a real requirement, not a nice-to-have.
- You're writing your fourth state-machine table this quarter.

**Recommendation by stage:**

- **Day one of the template:** stick with the queue from §12.2. Don't import Temporal complexity before you need it.
- **Year one, indie/bootstrapped:** if you cross the threshold above, **Hatchet** is the path of least resistance — it slots into your existing Postgres.
- **Year two, funded / enterprise:** **Temporal Cloud** is the safe pick — battle-tested, audited, used by Uber/Snap/Netflix, deep tooling. The managed offering removes the operational pain.

The same `Bus` / `Worker` interface pattern from §4.4 applies: workflows are **invoked** through a thin adapter so swapping queues for Temporal later is a worker rewrite, not an API rewrite. AI agents in particular (long pause, human-in-the-loop, hours-long runs) are the canonical fit — see the AI playbook §18.

---

## 13. 📡 Real-time & Eventing

### 13.1 In-process event bus (the spine)

A simple synchronous publisher with topic-based listeners:

```go
bus.Publish(ctx, "issue.created", IssueCreated{ID: ..., WorkspaceID: ...})
```

Listeners write derived state, enqueue jobs, and broadcast over WS.

**Important:** subscribers register **before** publishers. Document the order in `main.go`. Order is load-bearing.

### 13.2 WebSocket vs SSE

| Need | Use |
|---|---|
| Bidirectional (chat, collaborative editing) | **WebSocket** |
| Server → client only (live dashboards, notifications) | **SSE** (simpler, plays nice with HTTP/2) |

For most SaaS, SSE is enough. WebSocket only if you have meaningful client→server messaging beyond auth handshake.

### 13.3 Multi-node fanout

Single API node: in-memory hub.
Multi-node: backend hub publishes to a pub/sub bus, every node subscribes and forwards to its connected clients.

| Bus | When to pick it |
|---|---|
| **Redis pub/sub** | You already have Redis. Fire-and-forget. No durability — a disconnected node misses messages. |
| **Redis Streams** | Same Redis, but with replay + consumer groups. Good middle ground. |
| **NATS JetStream** | The right answer for any SaaS that's growing into multiple services. Persistent streams, replay, exactly-once-on-ack consumers, KV + object store, per-tenant subjects (`ws.<workspace_id>.>`), works as eventing backbone *and* WS fan-out *and* job queue. Cheap to self-host (single binary), clusters trivially. |
| **Kafka / Redpanda** | You have a data team and analytics pipelines. Overkill as a starting point. |

```plaintext
[Browser] ─WS─► [API node A] ─pub─► [NATS JetStream] ─sub─► [API node B] ─WS─► [Browser]
                                          │
                                          └─► [Worker pool] (durable consumers, replay on crash)
```

**Why NATS JetStream is the recommended template default once you outgrow single-node:**

- One binary replaces Redis pub/sub + a job queue + an event log.
- Per-tenant subject hierarchy (`tenant.<workspace_id>.events.>`) maps cleanly to multi-tenancy.
- Durable consumers give you the outbox-pattern guarantees (§12.4) without an outbox table for cross-service events.
- KV bucket for ephemeral state (presence, rate-limit counters) — you can drop Redis in some deployments.

Don't make any of this required for the dev/single-node experience. Single-node self-host should run on Postgres alone, with the bus interface no-op'd to an in-memory channel.

```go
// Bus abstraction — same interface, different backends.
type Bus interface {
    Publish(ctx context.Context, subject string, payload []byte) error
    Subscribe(ctx context.Context, subject string, h Handler) (Subscription, error)
}
// inproc.NewBus() | redis.NewBus(rdb) | nats.NewJetStreamBus(js)
```

### 13.4 Realtime ↔ Cache invalidation rule

> **WS events invalidate Query cache. They never write directly to client stores.**

Why: WS messages can arrive out of order, can be dropped, can be replayed. Cache invalidation is idempotent; direct writes are not.

```ts
ws.on("issue.updated", ({ id }) => {
  queryClient.invalidateQueries(["issue", id])
})
```

---

## 14. 📨 Email, Notifications & Inbox

### 14.1 Three notification surfaces

| Surface | Provider | Use for |
|---|---|---|
| Transactional email | Resend / Postmark / SES | Verify, reset, invite, receipts, dunning |
| In-app inbox | Your own DB | Mentions, comments, status changes, system messages |
| Push / SMS | Twilio / OneSignal / APNS | Mobile-only critical alerts |

### 14.2 Templates

- Use **MJML** or **React Email** for transactional templates. Renders to bulletproof HTML across clients.
- Keep one template per email type. Centralize a "layout" component.
- Plain-text fallback always.

### 14.3 Per-user preferences

```sql
notification_preference (
    user_id, workspace_id, channel TEXT, event_type TEXT, enabled BOOL
)
```

Every email and in-app alert checks preferences before sending. Default new events to "on" — but always allow opt-out with one click.

### 14.4 Unsubscribe link

- Every transactional email *except* security/billing has a `List-Unsubscribe` header + footer link.
- One-click unsubscribe (`mailto:` + URL).
- Persist the opt-out, don't re-send on bounce-back-then-recreate.

### 14.5 In-app inbox

Same data shape as email events. Render a bell icon with unread count + a list view. Keys:
- `notification` rows: `user_id`, `workspace_id`, `kind`, `payload JSONB`, `read_at`.
- WS push for live updates.
- Mark-all-read endpoint.

### 14.6 Digesting / batching

For high-volume events (chat mentions, comment replies):
- Real-time push if user is online.
- Otherwise, batch into a digest email (hourly/daily), configurable per user.

---

## 15. 📦 File Storage, Uploads & CDN

### 15.1 The cardinal rule

> **Never proxy file bytes through your API server.** Client uploads directly to S3 via signed URL.

```plaintext
[Client] ──GET /upload-url──► [API] ──signed PUT URL──► [Client]
[Client] ──PUT───────────────────────────────────────► [S3]
[Client] ──POST /confirm──► [API] (records metadata)
```

### 15.2 Server-issued signed URLs

```go
url := s3.PresignPutObject(ctx, bucket, key, ttl=15min, contentType=..., maxSize=...)
```

Always set:
- TTL (15 min usually).
- `Content-Type` constraint.
- `Content-Length` max (defense against unbounded uploads).
- Tenant-scoped key prefix: `s3://your-bucket/<workspace_id>/<file_id>`.

### 15.3 File metadata

```sql
file (
    id UUID PK,
    workspace_id UUID,
    uploader_user_id UUID,
    filename TEXT,
    mime_type TEXT,
    size_bytes BIGINT,
    s3_key TEXT,
    sha256 TEXT,
    status TEXT,  -- pending | uploaded | scanned | quarantined
    created_at TIMESTAMPTZ
)
```

### 15.4 Virus / content scanning

- For user-uploaded files, scan on upload (S3 event → Lambda / worker → ClamAV / proprietary).
- Until scanned, mark `status = pending` and refuse to serve.

### 15.5 Serving private files

- Generate signed GET URLs (5–60 min TTL), or
- Stream from server with auth check (only for small / sensitive files).

### 15.6 CDN

- Cloudflare or CloudFront in front of S3.
- Use signed CloudFront URLs for private content.
- Public assets (avatars, public docs) get a permanent path with cache-busting via content hash.

---

## 16. 🔎 Search (Full-Text + Semantic)

### 16.1 Start with Postgres

```sql
CREATE INDEX idx_issue_search ON issue
    USING GIN (to_tsvector('english', title || ' ' || coalesce(content, '')));
```

`pg_trgm` adds typo tolerance:

```sql
CREATE INDEX idx_issue_title_trgm ON issue USING GIN (title gin_trgm_ops);
```

This carries you to ~10M rows easily.

### 16.2 Move to a search engine when you need

- Fuzzy search across many fields with relevance tuning → **Meilisearch** or **Typesense** (both excellent DX).
- Massive scale + analytics → **Elasticsearch** / **OpenSearch**.
- Replicate from Postgres via CDC (Debezium) or write-on-write triggers.

### 16.3 Vector / semantic search

```sql
CREATE EXTENSION vector;
ALTER TABLE document ADD COLUMN embedding vector(1536);
CREATE INDEX ON document USING hnsw (embedding vector_cosine_ops);
```

Generate embeddings via OpenAI / local model in a worker after content changes. Don't generate them in the request path.

### 16.4 Hybrid search

Combine BM25 (keyword) and vector (semantic) with reciprocal rank fusion:

```plaintext
score(doc) = 1/(k + rank_bm25) + 1/(k + rank_vector)
```

This dramatically beats either alone for product search.

---

## 17. 🚩 Feature Flags & Experiments

### 17.1 Three flag scopes

```plaintext
flag → environment (dev/staging/prod)
     → workspace (tenant-level rollout)
     → user (individual override)
```

Every flag check resolves: env default → workspace override → user override.

### 17.2 Use a service

- **Self-host:** PostHog, Unleash, GrowthBook.
- **Hosted:** LaunchDarkly, Statsig.
- **DIY:** simple `flag` table + Redis cache → fine for ≤ 50 flags.

### 17.3 The kill-switch culture

Every risky new feature ships behind a flag. Rule: **"if it's not behind a flag, it can't ship."**

```go
if flags.IsEnabled(ctx, "new_billing_engine", workspaceID) {
    return newPath()
}
return oldPath()
```

After 2 weeks of stable rollout: clean up the flag and the dead branch.

### 17.4 Experiments / A-B tests

Ship via the same flag system with a randomized assignment. Log assignment + outcome to your analytics warehouse. Decide significance with a stats library or PostHog's experiment view — don't eyeball.

---

## 18. 📊 Audit Logs, Activity Feeds & Telemetry

### 18.1 Three different things, often confused

| Concept | Audience | Retention | Mutability |
|---|---|---|---|
| **Audit log** | Compliance / security teams | Years | Immutable, append-only |
| **Activity feed** | End users ("Alice changed the title") | Months | Mutable summaries OK |
| **Telemetry / analytics** | Your team (product/eng) | Months–years | Aggregated, anonymized |

Don't try to use one table for all three.

### 18.2 Audit log table

```sql
audit_log (
    id UUID PK,
    workspace_id UUID,
    actor_user_id UUID NULL,
    actor_type TEXT,          -- user | api_key | system
    action TEXT,              -- "issue.delete", "billing.plan.change", "auth.login"
    target_type TEXT,
    target_id UUID,
    metadata JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- never UPDATE or DELETE this table; partition by month
```

Log every privileged action: settings change, role change, billing change, member invite/remove, file deletion, login, password change, MFA enable/disable.

### 18.3 Activity feed

For end-user "what happened to my project":

```sql
activity (
    id, workspace_id, actor_user_id, verb, object_type, object_id, metadata, created_at
)
```

Render with templates: `"{actor} {verb} {object}"`.

### 18.4 Export

Enterprise plan users want audit log export (CSV / JSON / Splunk-compatible). Build the endpoint behind a feature flag.

---

## 19. 🛡️ Security, Compliance & Privacy

### 19.1 The OWASP non-negotiables

- Parameterized queries (no string-concatenated SQL ever).
- Input validation at every boundary (use Zod / pydantic / typed structs).
- Output encoding (React handles this; be careful in raw HTML / PDF generation).
- CSRF tokens on cookie-auth state-changing endpoints.
- CSP headers (`Content-Security-Policy: default-src 'self'`).
- HSTS (`Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`).
- Cookie attributes: `Secure; HttpOnly; SameSite=Lax`.
- File upload type + size + MIME validation.

### 19.2 Secrets management

- **Never commit secrets.** Pre-commit hook with `gitleaks` / `detect-secrets`.
- Local: `.env` (gitignored).
- Prod: AWS Secrets Manager / Doppler / Vault / Infisical.
- Rotate on personnel changes and on any leak suspicion.

### 19.3 Data classification

Tag every data field by sensitivity:
- **Public** — workspace name.
- **Private** — email, IP, billing address.
- **Sensitive** — password hash, OAuth tokens, API keys.
- **Restricted** — payment data (PCI), health data (HIPAA), kid data (COPPA) — generally avoid storing if you can.

Sensitive data: encrypt at rest with KMS-managed key. Restricted data: outsource to a compliant provider (Stripe for cards, etc.).

### 19.4 Compliance by tier

| Compliance | Effort | When you need it |
|---|---|---|
| **GDPR** (EU privacy) | Mandatory if you have any EU users | Day one |
| **CCPA** (California privacy) | Mostly overlaps with GDPR | Day one for US |
| **SOC 2 Type I → Type II** | 3–6 months prep + audit | When enterprise prospects ask |
| **HIPAA** | Significant; needs BAA with all subprocessors | Healthcare verticals only |
| **ISO 27001** | 6–12 months | International enterprise |
| **PCI-DSS** | High; outsource to Stripe and you're SAQ-A | If you touch card data |

For a template: bake in **GDPR-ready primitives** (data export endpoint, account deletion, consent log, data residency tag). Defer SOC 2 until you have $$$ on the line.

### 19.5 Key GDPR primitives

- **Export my data** endpoint: zip of every user-owned row in JSON.
- **Delete my account** endpoint: anonymize PII, retain audit logs with `user_id = NULL`.
- **Consent log**: `consent (user_id, type, version, granted_at, ip)`.
- **DPA (Data Processing Agreement)**: signed with every paid customer, downloadable PDF.
- **Subprocessor list**: public page listing every third party that touches customer data.
- **Data residency**: support EU-only deployments by tagging tenants and routing.

### 19.6 Penetration testing & bug bounty

- DIY scanning: OWASP ZAP / Burp / Nuclei / Trivy on every release.
- Third-party pentest: annually for SOC 2.
- Public bug bounty: HackerOne / Intigriti once you have something worth attacking.

---

## 20. ⚡ Performance, Caching & Scaling

### 20.1 Latency budget

A user-facing API request should complete in < 500 ms p95. Set this as a hard budget. Anything over needs optimization or async-ification.

### 20.2 Cache layers

```plaintext
[CDN]            — public assets, public docs, marketing pages
   ↓
[App-level]      — Redis (hot reads, computed views, rate-limit counters)
   ↓
[DB query cache] — Postgres shared buffers; no client-side query cache
   ↓
[DB read replica]— route read-heavy endpoints (e.g., search) to a replica
```

### 20.3 Rules

- **Cache invalidation > cache duration.** Always know how a cached value gets invalidated. Never set a long TTL "just in case."
- **Tag-based invalidation:** key the cache with `(workspace_id, kind, version)`. Bump version on writes.
- **Don't cache user-specific data with long TTLs.** Personalization defeats CDN caching anyway.

### 20.4 N+1 prevention

- Use `EXPLAIN ANALYZE` on hot endpoints.
- Use dataloaders in GraphQL.
- Prefer joins to per-row lookups.
- Add a CI check: log slow queries with `pg_stat_statements` and assert <5 over a benchmark.

### 20.5 Scaling Postgres

Order of operations:
1. **Indexes** — fix the missing ones first. 90% of Postgres "slow" is "no index."
2. **Connection pooling** — PgBouncer in transaction mode. Postgres can't handle 1000 connections; PgBouncer can.
3. **Read replicas** — route read-heavy reports.
4. **Partitioning** — by `workspace_id` or `created_at` for huge tables (audit log, events).
5. **Vertical scaling** — bigger box. Surprisingly far you can go.
6. **Sharding** — only when you have a reason. Last resort.

### 20.6 Background work moves the latency

If something can be async, it should be. Email, webhooks, audit log fanout, search indexing, analytics events — all queue-driven. Keep the request path lean.

---

## 21. 📈 Observability — Logs, Metrics, Traces, Errors

### 21.1 The four signals (correlated)

| Signal | Tool | Question it answers |
|---|---|---|
| **Logs** | Loki / Datadog / CloudWatch | What happened? |
| **Metrics** | Prometheus / Grafana | How much, how fast, how often? |
| **Traces** | Jaeger / Tempo / Honeycomb / Datadog APM | Where is time spent? |
| **Errors** | Sentry | What broke, and how do I reproduce? |

All four should share **`request_id`** and **`tenant_id`** so you can pivot from one to another.

### 21.2 Structured logging

**Go: `slog` (stdlib) or `zerolog`.** zerolog is the production default for Go SaaS — zero allocations on the hot path, fluent API, JSON-native, contextual loggers attach to `context.Context`.

```go
// zerolog — fluent, zero-alloc, context-aware
logger := log.With().
    Str("request_id", reqID).
    Str("workspace_id", wsID.String()).
    Str("user_id", userID.String()).
    Logger()

logger.Info().
    Str("issue_id", issue.ID.String()).
    Int64("duration_ms", elapsed.Milliseconds()).
    Msg("issue.created")
```

Equivalent with `slog`:

```go
slog.InfoContext(ctx, "issue.created",
    "request_id", reqID,
    "workspace_id", wsID,
    "user_id", userID,
    "issue_id", issue.ID,
    "duration_ms", elapsed.Milliseconds())
```

JSON in production, pretty-printed (zerolog's `ConsoleWriter`, or `tint` / `lmittmann` for slog) in dev. Never `fmt.Println`.

**Python: `structlog`.** The right answer for any FastAPI/async service — contextvars-aware, fast (with `orjson`), composable processors. `logging`-only is a dead end the moment you need request-scoped context.

```python
import structlog

structlog.configure(
    processors=[
        structlog.contextvars.merge_contextvars,   # request_id, workspace_id flow automatically
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer(serializer=orjson.dumps),
    ],
)

log = structlog.get_logger()

# In a middleware:
structlog.contextvars.bind_contextvars(
    request_id=req_id, workspace_id=ws_id, user_id=user_id,
)

# Anywhere downstream — context is automatic:
log.info("embedding.generated", document_id=doc.id, dim=1536, duration_ms=elapsed)
```

**Both languages, same rules:** one event per log line, snake_case keys, every log inside a request carries `request_id`, `workspace_id`, `user_id`. No interpolated strings (`f"user {id} did X"`) — that defeats structured search.

### 21.3 OpenTelemetry-first

Instrument with **OTel SDK** in every language. Export to whichever vendor — switching is then a config change, not a rewrite.

### 21.4 The four golden signals (per service)

- **Latency** — p50, p95, p99.
- **Traffic** — requests/sec.
- **Errors** — error rate (5xx + key 4xx).
- **Saturation** — CPU, memory, DB pool, queue depth.

Alert on anomalies, not absolute thresholds. Rate-of-change > p99 latency.

### 21.5 SLO + error budget

Define one or two SLOs and stick to them.

```plaintext
SLO: 99.9% of API requests < 500ms over 30-day window
     → error budget = 43 minutes/month
```

If you burn the budget, freeze feature work and fix reliability. This is the engineering culture lever.

### 21.6 On-call & runbooks

- Every alert has a **runbook URL** in the alert text.
- Runbooks live in the repo (`docs/runbooks/<alert>.md`), not Confluence.
- Post-mortems for every Sev-1 / 2: blameless, in-repo, indexed.

---

## 22. 🎨 Frontend Architecture

### 22.1 Strict state separation

| State type | Tool | Rule |
|---|---|---|
| **Server state** | TanStack Query | Everything from the API. Never duplicate into a client store. |
| **Client UI state** | Zustand (or React state) | Selection, modals, drafts, presence. |
| **URL state** | TanStack Router / Next.js | Filters, tabs, pagination — anything shareable. |
| **Form state** | React Hook Form + Zod | Validation co-located with schema. |

### 22.2 Package boundaries

For monorepo:

```plaintext
packages/
  core/       headless logic — stores, hooks, api client, types
              ZERO react-dom, ZERO localStorage (use adapter), ZERO process.env
  ui/         atomic primitives (shadcn-style)
              ZERO @core imports, ZERO business logic
  views/      business components & pages
              ZERO next/*, ZERO routing-library imports (use adapter)
apps/
  web/        Next.js wiring + adapters
  desktop/    Electron wiring + adapters
  mobile/     React Native wiring + adapters
```

Internal packages export **raw `.ts` / `.tsx`**, no build step. Consumer's bundler compiles. Fast HMR, real go-to-definition.

### 22.3 Design system

- **Tailwind** for atomic styling. No CSS-in-JS in 2026 — Tailwind v4 is faster and cleaner.
- **shadcn/ui** as base primitives — copy-paste, then own them.
- **Radix UI** under the hood for accessibility.
- One token file (`design-tokens.ts`) for colors, spacing, radii.
- One typography scale.
- **Storybook** (or **Ladle** if you want a faster, lighter alternative) for component dev. One story per component covering default + edge states (loading, error, empty, long-text). Doubles as living documentation for designers and as the surface for visual regression tools (Chromatic, Percy, Playwright snapshots) and `axe-core` a11y checks in CI.

### 22.4 Routing

- Next.js app router (RSC + streaming) if you want SEO-able marketing + app in one stack.
- Vite + TanStack Router if you want an SPA with type-safe routing.
- Avoid mixing two routers in one app.

### 22.5 Forms

```ts
const schema = z.object({ title: z.string().min(1).max(120) })
type FormValues = z.infer<typeof schema>

const form = useForm<FormValues>({ resolver: zodResolver(schema) })
```

Same Zod schema is reused for API validation server-side. Single source of truth.

### 22.6 Loading states + suspense

- Skeleton screens for any fetch > 200ms.
- Optimistic updates for user-triggered actions (TanStack Query mutations).
- Error boundaries at route level — never let an error nuke the whole app.

### 22.7 Critical UX details

- Keyboard shortcuts (Cmd-K, Cmd-Enter, /).
- Toast system (one provider, `toast.success(...)`).
- Global confirm modal helper.
- Date formatting via one utility (`formatDate(d, "short")`) — never raw `toLocaleString`.
- `<Link>` everywhere — never raw `<a>` for internal nav.

---

## 23. 🌍 Internationalization & Accessibility

### 23.1 i18n from day one — even if you ship English-only

Defer language additions; don't defer the **plumbing**.

- Wrap every user-facing string in `t("key.name")`.
- Use **i18next** / **next-intl** / **format.js**.
- Keep translations in `locales/<lang>.json`.
- Use ICU MessageFormat for plurals/genders.
- Avoid string concatenation — translators need full sentences.

### 23.2 Locale-aware formatting

- Dates: `Intl.DateTimeFormat`.
- Numbers / currency: `Intl.NumberFormat`.
- Pluralization: ICU select.
- Time zones: store UTC, render local.

### 23.3 Accessibility (WCAG 2.2 AA)

- Every interactive element keyboard-reachable.
- Visible focus states (don't `outline: none` without a replacement).
- ARIA labels on icon-only buttons.
- Semantic HTML — `<button>` not `<div onClick>`.
- Color contrast ≥ 4.5:1 for body text.
- Test with `axe-core` in CI.

---

## 24. 🔧 Admin & Internal Tooling

### 24.1 Build it day one. Do not skip.

You'll be on support-debug duty all year. An admin panel pays for itself in week two.

### 24.2 What goes in it

| Capability | Why |
|---|---|
| Search any user / workspace | Triage support tickets. |
| Impersonate user (read-only by default) | "It works on my machine" reproduction. |
| Suspend / unsuspend workspace | Abuse handling. |
| Force-verify email | Lost-access support flow. |
| Refund / credit | Billing support. |
| Adjust plan / quota | Sales overrides. |
| Re-send webhook | Customer integration debug. |
| Replay failed jobs | Ops. |
| Inspect Stripe customer | Without leaving your tool. |
| Feature flag override per tenant | Beta access requests. |

### 24.3 Implementation

- Same codebase, gated behind `is_internal_admin` claim.
- Separate hostname (`admin.yourtool.com`) and route group.
- Every action audit-logged with `actor_user_id` (the staff member, not the impersonated user).
- IP-allowlist optional; **MFA mandatory**.
- Time-boxed sessions (re-auth every 30 min).

### 24.4 Don't overthink

You don't need React-Admin or Retool. A plain set of pages with tables and confirm modals is fine. Internal users will accept worse UX than customers.

### 24.5 BI for the business team

Sales/CS/finance/leadership will ask the same kind of questions every week — "MRR by plan?", "trial-to-paid by signup source?", "top 50 workspaces by API usage?". Without a self-serve tool, every one of those becomes a Slack message to engineering. Stand up a BI dashboard against a **read replica** (or a warehouse mirror — see §4.2) on day one of having paying customers.

| Tool | License | Sweet spot | Watch out for |
|---|---|---|---|
| **Apache Superset** | Apache 2.0 | **Default recommendation.** Clean license, powerful SQL Lab, rich chart library (incl. geospatial via deck.gl), scales to large orgs. The right pick when your data team is comfortable in SQL. | Steeper UX for non-technical users; more ops overhead than Metabase. |
| **Metabase (Community)** | **AGPLv3** | Easier UX than Superset for non-technical users — point-and-click query builder genuinely works for sales/CS. Setup in 10 minutes. | **License gotcha:** AGPL is usually fine for internal-only BI but a hard block for embedded analytics in your customer-facing product (need Metabase Enterprise for embedding rights). Many corporate legal policies blanket-ban AGPL — verify with counsel. |
| **Lightdash** | MIT | dbt-native — your dbt models *are* the metrics layer. Best fit if you're already on dbt for transformations. | Smaller community; assumes a dbt workflow. |
| **Evidence.dev** | MIT | Code-as-config (Markdown + SQL → static dashboards in git). Versioned reports as a developer-friendly alternative to clicky dashboard tools. | Not interactive ad-hoc exploration — built for *publishing* recurring reports, not slicing-and-dicing. |
| **Redash** (Databricks-owned) | BSD-2-Clause | Lightweight SQL-first dashboarding. Mature, simple, low-touch. | Lower velocity since the Databricks acquisition; community pace has slowed. |
| **Hex / Mode / Hashboard** | Managed (commercial) | Polished hosted experiences with notebook-style data exploration; pay-per-seat. | Per-seat pricing scales with the team that uses it most. |

**Template recommendation:**

- **Default:** Apache Superset against a Postgres read replica — Apache 2.0 license keeps your options open, and the SQL Lab covers 90% of business questions.
- **If your team is mostly non-technical and AGPL is acceptable:** Metabase is the better UX. Just confirm with legal first, especially if you might want to embed dashboards in your product later.
- **If you already run dbt:** Lightdash, since "the metric layer is your dbt models" is genuinely a better workflow than maintaining metrics in two places.

Run BI **only against a read replica or warehouse mirror**, never your primary OLTP database. A finance team running a "everything joined to everything" query will lock your prod app. Same auth gate as the admin panel (§24.3): SSO + MFA, IP-allowlist optional, time-boxed sessions.

---

## 25. 📝 Marketing Site, Docs & SEO

### 25.1 Three separate surfaces, often conflated

| Surface | Stack | URL |
|---|---|---|
| Marketing site | Next.js (or Astro) | `yourtool.com` |
| Product docs | Mintlify / Docusaurus / Nextra | `yourtool.com/docs` |
| API reference | Stoplight / Redoc / Mintlify | `yourtool.com/docs/api` |
| Status page | StatusPage.io / Instatus | `status.yourtool.com` |
| Changelog | Markdown in repo + RSS | `yourtool.com/changelog` |

Don't try to put marketing + app + docs in one Next.js app on day one. Build separately, deploy separately, link liberally.

### 25.2 SEO basics

- Server-render marketing + docs (RSC, static generation).
- Per-page `<title>` and `<meta description>`.
- Open Graph + Twitter card tags + share image generator.
- `sitemap.xml` + `robots.txt`.
- JSON-LD schema for product/company.
- Page speed: lighthouse ≥ 95 on every marketing page.

### 25.3 Conversion essentials

- Clear pricing page with comparison table + FAQ.
- Public roadmap (or at least a changelog).
- Customer logos / case studies (after you have any).
- Contact + sales form that goes to a real human in < 24h.

---

## 26. 🚢 CI/CD, Environments & Release Strategy

### 26.1 Environment ladder

```plaintext
dev (laptop)  →  ephemeral preview (per-PR)  →  staging  →  production
```

- **Preview environments** per PR: each PR gets its own deployed URL with a seeded DB. Vercel / Render / Fly do this natively.
- **Staging** mirrors prod config + tools but with a separate DB. For E2E tests + final smoke.
- **Production** is the only environment paying customers see.

### 26.2 CI pipeline (keep < 10 min)

```plaintext
1. Install deps (cache aggressively)
2. Lint  (parallel)
3. Typecheck  (parallel)
4. Unit tests  (parallel)
5. Build artifacts
6. Integration tests (real Postgres + Redis as services)
7. E2E tests (Playwright against built artifacts) — only on main + tags
8. Deploy preview (PR) / staging (main) / prod (tag)
```

Fail fast: lint + typecheck before tests. Cache `node_modules` and `~/go/pkg/mod`.

### 26.3 Database migrations on deploy

- Migrations run **automatically** on deploy, before app code.
- Always backwards-compatible: app version N+1 must work against DB at version N (briefly, during rollout).
- For destructive migrations (drop column), use a 2-deploy dance: stop reading → deploy → drop column.

### 26.4 Release strategy

- **Blue-green** or **rolling** deploys. Never stop-the-world.
- **Canary** for risky changes: 1% → 10% → 50% → 100% with metrics gates.
- **Feature flags** decouple deploy from release. Deploy whenever; release when ready.
- **Tag-driven releases** for the CLI / desktop apps via GoReleaser / electron-builder.

### 26.5 Rollback

- Every release is a single immutable artifact (container image with sha256 tag).
- `make rollback` reverts to the previous artifact in < 60 seconds.
- DB migrations are forward-only; rollback means *not running the new migration yet*, not undoing it.

### 26.6 Where to host (and when to switch)

| Stage | Host | Why |
|---|---|---|
| Local dev | Docker Compose | Single command, identical to prod shape. |
| First production deploy | **Fly.io** / **Railway** / **Render** | Push-to-deploy, managed Postgres, zero ops. Cost: $20–$100/mo until you have traction. |
| Profitability stage | **Hetzner** (Cloud or dedicated) + **Caddy** front door | Best price-to-performance in the industry. A €20/mo CCX dedicated-vCPU box runs the API + workers comfortably for thousands of paying customers. Pair with managed Postgres elsewhere or run it yourself with daily off-site backups. |
| Polished IaaS | **Digital Ocean** (Droplets + Managed PG/Redis + Spaces + App Platform) | Better dashboard than Hetzner, managed databases included, predictable billing. ~2× the cost of Hetzner for similar specs but you get the managed pieces. |
| Enterprise / compliance | **AWS / GCP / Azure** | Region breadth, BAAs, customer procurement requirements. |

**Reverse proxy on VM-style hosts (Hetzner, DO Droplets, bare metal):**

- **Caddy** — single binary, automatic HTTPS via Let's Encrypt/ZeroSSL, config in a Caddyfile. The right default for "I have one or two boxes."

  ```caddyfile
  app.yourtool.com {
      reverse_proxy api-1:8080 api-2:8080 {
          health_uri /healthz
      }
      encode gzip zstd
      log
  }
  ```

- **Traefik** — pulls config from Docker labels, K8s ingress objects, or a key-value store. The right default when you have a containerized fleet that scales horizontally and you want zero manual proxy config.

  ```yaml
  # docker-compose.yml
  api:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`app.yourtool.com`)"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
  ```

Don't run nginx unless you have a specific reason — Caddy and Traefik handle TLS, HTTP/3, and modern defaults without the config gymnastics.

### 26.7 The bootstrapped reference deployment

A surprising number of profitable SaaS run on:

```plaintext
[Cloudflare] (CDN, WAF, DNS, Turnstile, R2 for files)
     │
     ▼
[Hetzner CCX dedicated-vCPU box, €20–€60/mo]
     │
     ├── Caddy (TLS, reverse proxy)
     ├── Go API (Gin + GORM + zerolog)
     ├── Worker (Asynq or NATS JetStream consumer)
     ├── NATS JetStream (single node, file-backed)
     ├── Postgres 16 (with WAL-G off-site backups to R2)
     └── Casdoor (auth, separate container)
```

Total infra cost: **€30–€80/month** all-in. Capable of serving thousands of paying customers before you need a second box. Move to Digital Ocean managed Postgres the day you stop wanting to be the on-call DBA.

---

## 27. 🧰 Developer Experience (DX)

### 27.1 The "one command to dev" rule

```bash
make dev
```

Should:
1. Boot Postgres + Redis (Docker Compose).
2. Run migrations.
3. Seed data.
4. Start API + workers + frontend with hot reload.
5. Print URLs for app, docs, mailcatcher, DB UI.

If a new engineer can't `git clone && make dev` and reach the running app in 10 minutes, fix the gap.

### 27.2 Seed data

Realistic, idempotent, reproducible:
- 5 workspaces with different plans.
- 20 users, with at least one in each role.
- 100 representative resources (issues / projects / etc.).
- 1 demo workspace anyone can browse.

### 27.3 Mail in dev

Run **MailHog** / **Mailpit** in Compose. All transactional emails route there. Open the UI to read them.

### 27.4 DB UI in dev

Embed **pgweb** / **Adminer** in Compose at `localhost:8081`. Saves "where's the user table" Slack messages.

### 27.5 Repo conventions

- `Makefile` is the entry point for every workflow (`make dev`, `make test`, `make migrate-up`, `make seed`).
- `.env.example` checked in; `.env` gitignored.
- `CONTRIBUTING.md` with the 5 commands a new dev needs.
- `docs/decisions/` for ADRs (Architecture Decision Records).

### 27.6 Codegen, not boilerplate

- API clients **generated** from OpenAPI.
- DB types **generated** by sqlc / Prisma.
- Translation keys **type-checked**.
- Routes **type-safe** (TanStack Router / Next).
- If you find yourself writing the same thing in three places, generate it.

### 27.7 Pick one Go stack and standardize on it

Two viable shapes. Don't mix them within one service.

| Shape | Stack | When to pick |
|---|---|---|
| **Lean / SQL-first** | `chi` (router) + `sqlc` (codegen) + `pgx` (driver) + `slog` or `zerolog` | You want explicit SQL, zero ORM magic, maximum performance. Code reads like a database textbook. |
| **Batteries-included** | `Gin` (router + middleware ecosystem) + `GORM` (ORM, migrations, hooks) + `zerolog` | You want to ship features faster and trade some control for ergonomics. Most Go SaaS teams pick this. |

**For the template, default to Gin + GORM + zerolog** unless your team has a strong preference. It's the path with the most tutorials, middleware, and Stack Overflow answers — which matters when onboarding new engineers.

```go
// Gin + GORM + zerolog skeleton
r := gin.New()
r.Use(
    requestid.New(),
    ginzerolog.Logger("api"),     // structured access logs
    gin.Recovery(),
    middleware.Auth(authProvider), // verifies session/JWT, sets actor in ctx
    middleware.Tenant(),           // resolves workspace_id, sets app.workspace_id GUC
)

r.POST("/api/v1/projects", handlers.CreateProject(db))

// db is *gorm.DB with logger plugged into zerolog
```

GORM gotchas to know up front: callbacks fire on every save (use them for audit-log fan-out, not business logic), `Preload` is N+1's disguise (prefer explicit joins for hot paths), and `AutoMigrate` is fine for dev but **never run it in prod** — use `goose`, `golang-migrate`, or Atlas for versioned production migrations.

---

## 28. 🧪 Testing Strategy

### 28.1 The pyramid

```plaintext
       /\      E2E (Playwright)         5–10%   slow, valuable
      /  \
     /----\    Integration (real DB)    20–30%  most leverage
    /------\
   /--------\  Unit                     60–70%  fast feedback
```

### 28.2 Rules

- **Unit tests** are co-located with source: `foo.go` + `foo_test.go`, `Button.tsx` + `Button.test.tsx`.
- **Integration tests** spin up a real Postgres + Redis (testcontainers, or services in CI).
- **E2E tests** run against the full Compose stack on tagged releases + main.
- **Fast tests** in pre-commit / on file save. Full suite in CI.

### 28.3 Critical user-facing flows to E2E

1. Sign up → verify email → create workspace → first activation event.
2. Invite teammate → teammate accepts → both see the same data.
3. Upgrade plan → feature unlocks immediately.
4. Cancel plan → downgrade scheduled at period end.
5. Forgotten password → reset → log back in.

If any of these break, the whole product is broken. E2E them.

### 28.4 Snapshot tests

- Useful for emails (rendered HTML) and API responses (response schema).
- **Avoid for UI** — too much false-positive noise. Visual regression tools (Chromatic / Percy) are better.

### 28.5 Property-based tests

For pure logic (validation, pricing math, date calculations) — `fast-check` (TS) / `hypothesis` (Python) / `gopter` (Go) catch the cases you didn't think of.

### 28.6 Don't skip coverage; don't worship it

Aim for ~70% line coverage on logic-heavy packages. Below that = gaps. Above 90% = you're testing trivial getters.

---

## 29. 💰 Pricing, Plans & Packaging Strategy

### 29.1 The three SaaS pricing axes

1. **Per-seat** — works for collaboration (Slack, Linear, Figma). Predictable, scales with customer.
2. **Usage-based** — works for backend infra & AI (Stripe, OpenAI, Vercel). Aligns with value, but harder to budget.
3. **Per-feature tier** — works for breadth (HubSpot, Zendesk). Lets enterprise sales upsell.

Most SaaS combine all three: per-seat × tier + usage-based add-ons.

### 29.2 Recommended starting tiers

```plaintext
Free / Hobby     — 1 user, X resources, limited features    → top of funnel
Starter / Pro    — N users, full features, $/seat/month     → SMB / individual paid
Team / Business  — unlimited users, advanced features       → mid-market
Enterprise       — SSO, audit export, custom DPA, support   → contact sales
```

Don't ship 6 tiers on day one. Ship 3.

### 29.3 What goes behind the paywall

- **Free:** the core value prop, scoped (e.g., "10 issues, 1 user").
- **Pro/Team:** depth (advanced fields, automations, API).
- **Enterprise:** trust (SSO, SCIM, audit log export, custom contract, SLA, support).

### 29.4 Annual discount

Standard: ~20% off vs monthly. Locks in cash flow + reduces churn.

### 29.5 Free trial vs freemium — pick one

- **Trial** (14 days, full features) — high commercial pressure, faster decision.
- **Freemium** (free forever, limited) — top-of-funnel volume, harder conversion.

For a vertical/B2B SaaS template: default to **trial**. For PLG products targeting individuals: freemium.

### 29.6 Discounting & overrides

- Coupons in Stripe with promotion codes for marketing.
- Sales-set discounts via admin panel (audit-logged).
- Annual prepay discounts handled by Stripe automatically.

---

## 30. 🎯 Product Analytics & Growth

### 30.1 Two analytics stacks

| Stack | Tool | Purpose |
|---|---|---|
| Product | **PostHog** / Mixpanel / Amplitude | "Did the user activate? Convert? Churn?" |
| Engineering | OpenTelemetry → Grafana | "Is the system healthy?" |

PostHog is the recommended default — it bundles analytics, session replay, feature flags, and A/B tests in one tool.

### 30.2 The events you must track

From day one:
- `signed_up` (workspace_id, user_id, source)
- `activated` (workspace_id) — your activation event
- `<core_action>_created` — whatever your "noun" is
- `invited_member`, `member_accepted`
- `upgraded_plan`, `downgraded_plan`, `cancelled_subscription`
- `viewed_paywall`, `clicked_upgrade`

Every event has `workspace_id` and `user_id`. Don't track per-user without per-tenant.

### 30.3 The funnels you must measure

- Sign-up → email-verified → workspace-created → activated.
- Activation → invite teammate → second user activated.
- Free → paywall view → upgrade.
- Subscribed → renewal (LTV / churn).

### 30.4 Cohort retention

Plot retention by signup-week cohort. Healthy SaaS shows a "smile" — short-term decline, long-term flat or up. If your retention curves go to zero, no amount of marketing fixes the product.

### 30.5 NPS / CSAT

In-app survey (Delighted / built-in PostHog) at 30 days post-signup and quarterly. NPS > 30 is good, > 50 great.

---

## 31. 🤝 Customer Support & Success

### 31.1 Day-one support stack

- **Email:** `support@yourtool.com` → ticketing system (Pylon, Plain, HelpScout, or just Front).
- **In-app chat:** Intercom / Crisp / Pylon. Gate by plan if costly.
- **Docs:** searchable, with embedded video.
- **Status page:** automatic incident updates from your monitors.
- **Community:** Slack / Discord / Discourse — only if you have bandwidth to keep it active.

### 31.2 Build support hooks into the product

- "Get help" button opens chat with current page URL pre-filled.
- "Copy debug info" button: workspace_id, user_id, browser, version, request_id of last error.
- Per-error pages include `request_id` + a "contact support" link.

### 31.3 Customer success vs support

- **Support** reacts: ticket comes in, response goes out.
- **Customer success** is proactive: usage drops, success manager reaches out.

You don't need CS until you have customers worth saving. But instrument the data day one.

---

## 32. 📦 Reusability — How to Make This a Template

If the goal is a **template** you fork per product, the architecture must keep domain-specific code clean.

### 32.1 The "kernel + product" split

```plaintext
kernel/          — every SaaS has this
  auth, tenancy, billing, notifications, audit, admin, files, search,
  flags, analytics, infra, observability

product/         — your domain
  models, services, handlers, UI, jobs
```

### 32.2 Hard rules

- `kernel/` **never imports `product/`**. One-way dependency.
- `product/` extends kernel through hooks/interfaces, never by editing kernel.
- New tenant-scoped tables follow the same conventions: `id`, `workspace_id`, `created_at`, RLS policy.
- Domain events publish on the same in-process bus.
- Domain UI uses the same design system + permission helpers.

### 32.3 Configuration over code

Most "per-product" customizations should be config:

```yaml
# product.config.yaml
brand:
  name: "MyApp"
  primary_color: "#5B5BD6"
features:
  audit_log_export: true
  custom_domains: false
plans:
  - name: starter
    price_cents: 1900
    limits: { members: 5 }
```

Logo, name, palette, plan structure — all configurable without touching kernel code.

### 32.4 Domain plug-points

Predefine extension points in the kernel:

| Hook | Example use |
|---|---|
| `OnSignup(user, workspace)` | Auto-create demo project. |
| `OnActivated(workspace)` | Send welcome email + slack notification. |
| `BeforeRequest(ctx)` | Inject tenant-specific data. |
| `MeterEvent(name, qty)` | Custom usage metering for your domain. |
| `RenderEmail(template, data)` | Domain-specific transactional emails. |

Each is a Go interface or TS function imported from `kernel`, implemented in `product`.

### 32.5 Reskin checklist (minutes, not days)

- [ ] Update `product.config.yaml`.
- [ ] Replace logo, favicon, OG images.
- [ ] Update `tailwind.config.ts` colors.
- [ ] Update marketing copy in `apps/marketing/content/`.
- [ ] Configure Stripe products + prices, paste IDs into config.
- [ ] Add domain models to `product/`.
- [ ] Wire domain routes / pages.
- [ ] Update `seed.go` with domain-relevant demo data.

### 32.6 Versioning the template

Treat the template as its own project with a version. When kernel improves, projects forked from it can pull updates by:

1. Adding the template repo as a `template-upstream` remote.
2. Cherry-picking kernel commits.
3. Or running a custom `bin/upgrade-kernel` that copies non-product paths.

---

## 33. 🗺️ The 14-Phase Build Plan

Each phase is shippable. **Don't skip ahead.** Most failures here come from doing phase 7 before phase 3 is solid.

### 🌱 Phase 1 — Skeleton (2 days)

- Monorepo: `apps/web`, `apps/api`, `packages/{core,ui,views}`, `infra/`.
- Docker Compose: Postgres + Redis + Mailpit + pgweb.
- `make dev` brings up the stack with hot reload.
- Health endpoints, structured logging, request ID middleware.
- One CI job: lint + typecheck + unit tests.

**Done when:** `git clone && make dev` and an empty app loads with no auth.

### 🔐 Phase 2 — Auth (2 days)

- Email + password + magic link.
- Email verification.
- Google OAuth.
- Password reset.
- Session via cookie (browser) and JWT (API).
- Rate limit on `/login`.

**Done when:** new user can sign up, verify, log out, log in, reset password.

### 🏢 Phase 3 — Tenancy (2 days)

- `workspace`, `membership`, `invite` tables.
- Workspace creation flow.
- Workspace switcher UI.
- Subdomain or path-based routing.
- RLS policies on every tenant-scoped table.
- Permission helper `Can(user, action, resource)`.
- Roles: owner, admin, member.

**Done when:** invited teammates only see the workspaces they belong to. Cross-tenant DB access is blocked at the RLS layer.

### 📨 Phase 4 — Notifications & Email (1 day)

- Resend / Postmark integration.
- React Email templates: verify, reset, invite, billing failure.
- In-app inbox table + WS push.
- Notification preferences.

**Done when:** invite emails arrive in Mailpit (dev) and real inbox (prod), and the in-app bell shows new mentions.

### 💳 Phase 5 — Billing (3 days)

- Stripe integration: Checkout + Customer Portal.
- Plans table + `subscription` table + webhook handler.
- Trial logic.
- Feature gating helper.
- Dunning emails on failed payments.
- Admin override for plan/quota.

**Done when:** users can pick a plan, pay, see their plan, upgrade, downgrade, and a failed payment triggers correct UX.

### ⚙️ Phase 6 — Background Jobs & Cron (1 day)

- Job queue (Asynq / River / BullMQ).
- Worker process running in Compose.
- Job examples: send email, sync to Stripe, expire trial.
- Cron scheduler with leader election or Postgres-backed.
- Outbox pattern for transactional events.

**Done when:** a 10-second job runs in the worker, the API stays fast, and a daily cron fires once across N replicas.

### 📦 Phase 7 — Files (1 day)

- S3 / R2 bucket per environment.
- Signed-URL upload endpoint.
- Confirm endpoint storing metadata.
- Avatar upload as the canonical example.
- CDN with signed cookies for private files.

**Done when:** a user can upload an avatar and serve it via CDN, without bytes touching the API.

### 🔎 Phase 8 — Search & Search-Adjacent (1 day)

- Postgres FTS index on the main domain entity.
- Generic `searchable` interface.
- Hybrid (BM25 + trigram) ranking.
- (Optional) pgvector + embedding worker.

**Done when:** typing in the search bar returns relevant results in < 200ms.

### 📡 Phase 9 — Real-time (1 day)

- WebSocket endpoint with auth + origin check.
- In-process hub + (optional) Redis pub/sub for multi-node.
- Client subscribes, server invalidates Query cache via WS event.
- Presence (online/offline indicators).

**Done when:** two browser windows show the same data update simultaneously.

### 📊 Phase 10 — Audit, Activity, Telemetry (1 day)

- `audit_log` table with privileged-action logging.
- `activity` table for user-facing feeds.
- PostHog (or equivalent) wired with the canonical events.
- Workspace activation event + retention dashboard.

**Done when:** every privileged action is in the audit log and every signup is tracked in PostHog.

### 🚩 Phase 11 — Feature Flags & Admin Panel (2 days)

- Self-hosted PostHog or DIY flag table.
- Per-env / per-workspace / per-user flag resolution.
- Admin panel: user search, workspace search, impersonate (read-only), suspend, override flags.
- Admin actions audit-logged with staff actor.

**Done when:** support can resolve a "I can't see X" ticket in < 5 minutes via admin tools.

### 🛡️ Phase 12 — Security & Compliance Foundation (1 day)

- CSP, HSTS, secure cookies, CSRF.
- `gitleaks` pre-commit + CI.
- GDPR primitives: data export endpoint, account deletion endpoint, consent log.
- DPA template + subprocessor list page.
- Pen-test scan via OWASP ZAP in CI.

**Done when:** a security review can pass the OWASP Top 10 checklist without changes.

### 📈 Phase 13 — Observability (1 day)

- OpenTelemetry SDK in API + workers.
- Logs, metrics, traces all tagged with `request_id` + `tenant_id`.
- Sentry for errors.
- Basic Grafana dashboard with golden signals.
- Status page (Instatus or self-hosted).
- One SLO defined + alerted.

**Done when:** clicking an error in Sentry takes you to the trace, which links to the logs, which contain the request.

### 📦 Phase 14 — Package, Document, Reskin (2 days)

- `kernel/` ↔ `product/` separation.
- `product.config.yaml` and reskin guide.
- Marketing landing page template.
- Docs site template (Mintlify / Nextra).
- README + CONTRIBUTING + ADRs.
- One full reskin pass to verify the template works.

**Done when:** a new engineer can fork, run `bin/reskin --name AcmeApp --color "#FF5C5C"`, and have a custom-branded skeleton in 30 minutes.

---

**Total: ~21 working days for a single experienced engineer to build an MVP-quality SaaS template. ~6–8 weeks calendar with reviews, polish, and docs.**

---

## 34. ⚠️ Common Pitfalls & Hard-Won Guardrails

| Pitfall | Guardrail |
|---|---|
| Forgetting `WHERE workspace_id = ?` somewhere | RLS policies on every tenant table; CI grep for missing filters. |
| Stripe webhook handler is non-idempotent | Use `event.id` as a dedup key in Redis with 7-day TTL. |
| Long-running job blocks request path | Move to a queue; never call third parties synchronously. |
| Admin actions not audit-logged | Wrap every admin handler in middleware that writes to audit log. |
| Email enumeration on signup/login | Same response and timing for "exists" vs "not exists". |
| Migration breaks rolling deploy | Two-phase migrations; never drop+rename in one shot. |
| WS message updates client store directly | Rule: WS invalidates Query cache only, never writes to stores. |
| Cookie auth without CSRF | `SameSite=Lax` + CSRF token on state-changing endpoints. |
| Secrets committed to git | `gitleaks` pre-commit + CI fail. |
| Free tier abuse (signup farming) | Rate limit signups per IP + email-domain block list + Cloudflare Turnstile. |
| Plan change inconsistencies (paid down to free with paid resources still active) | Plan change handler: enforce limits, archive overflow, email user. |
| Trial expires while user has 50 issues | Read-only mode + upgrade banner; do not delete data. |
| Hot N+1 query in detail page | `EXPLAIN ANALYZE` in CI for top endpoints. |
| Cache that never invalidates | Tag-based invalidation; never set TTL > 1 hour without invalidation hook. |
| Tenant data exposed via search index | Search index keys include `workspace_id` and the search query filters by it. |
| Misconfigured CORS opens API to malicious origins | Allowlist origins explicitly; reject `*` with credentials. |
| User can delete their own audit log entries | Audit log is append-only; no user-facing endpoint to mutate. |
| One slow query takes down the API | Statement-level timeouts (`SET LOCAL statement_timeout = '5s'`). |
| Background worker silently fails forever | Dead-letter queue + alert on DLQ depth. |
| Subdomain takeover via stale CNAME | Audit DNS regularly; deactivate orphan subdomains. |
| Test data leaks into prod | Distinct connection strings; loud banner in non-prod environments. |
| "Forgot password" reveals if email exists | Generic response: "If an account exists, we've sent a reset link." |
| No consent log → GDPR audit fails | `consent` table with version + timestamp + IP from day one. |
| Customer asks for a feature already on roadmap | Public roadmap so they can upvote instead of opening a ticket. |

---

## 35. 📋 Cheat Sheet

### 📖 First files / decisions to lock down

1. **Multi-tenancy model** — pool, all queries filter by `workspace_id`, RLS as defense.
2. **Auth model** — cookie session for browser, JWT for mobile/API, API keys for integrations.
3. **Permissions** — single `Can(actor, action, resource)` helper, RBAC roles.
4. **Billing** — Stripe Checkout + Customer Portal; metered prices for usage.
5. **Event bus** — in-process publisher → outbox → workers.
6. **API shape** — REST + JSON, cursor pagination, single error envelope, idempotency keys.
7. **Frontend state** — TanStack Query for server state, Zustand for UI, never mix.

### ⚙️ Default config defaults

| Setting | Default |
|---|---|
| Session TTL (cookie) | 14 days, sliding |
| JWT access token TTL | 15 min |
| Refresh token TTL | 30 days |
| API rate limit | 100 req/min/IP, 1000 req/min/workspace |
| File upload max | 100 MB |
| Idempotency cache TTL | 24 h |
| Trial length | 14 days |
| Soft-delete grace period | 30 days |
| Audit log retention | 7 years |
| Activity feed retention | 6 months |
| GDPR data export TTL | 7 days from generation |
| Workspace slug regex | `[a-z0-9-]{3,40}` |
| Password min length | 12 chars (or zxcvbn score ≥ 3) |

### 🚫 Hard rules (non-negotiable)

- Every tenant-scoped query filters by `workspace_id`.
- Every privileged action writes to `audit_log`.
- Every email obeys per-user notification preferences.
- Every webhook handler is idempotent.
- Every form input is validated server-side (Zod / pydantic / typed structs).
- Every secret is in a secrets manager, not in env in prod.
- Every public endpoint has a rate limit.
- Every payment side effect goes through Stripe webhooks, not the request path.
- Every long-running task is in a job queue.
- WS events invalidate Query cache; they never write directly to stores.
- Migrations are append-only.
- Admin actions are audit-logged with the staff member as actor.
- Feature flags wrap any risky new behavior.
- File uploads bypass the API server (signed S3 URLs).
- No `WHERE` clause in SQL is built via string concatenation.
- New tables follow the convention: `id`, `workspace_id`, `created_at`, `updated_at`.

### 📐 The canonical resource shape (REST)

```json
{
  "id": "01HMZQ...",
  "workspace_id": "01HMW1...",
  "name": "Project Alpha",
  "status": "active",
  "created_at": "2026-04-30T10:00:00Z",
  "updated_at": "2026-04-30T10:00:00Z",
  "created_by": { "type": "user", "id": "01HM..." }
}
```

### 🎭 The polymorphic-actor pattern

```sql
created_by_type TEXT CHECK (created_by_type IN ('user','api_key','system')),
created_by_id   UUID
```

Use this on every "actor" field. It lets you treat agents, integrations, and humans uniformly without parallel schemas.

### 🔑 Environment variables baseline

```properties
APP_ENV=production            # dev | staging | production
APP_URL=https://app.yourtool.com
PUBLIC_URL=https://yourtool.com

DATABASE_URL=postgres://...
REDIS_URL=redis://...

JWT_SECRET=<32-byte-random>
SESSION_SECRET=<32-byte-random>
COOKIE_DOMAIN=.yourtool.com

STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
PAYPAL_CLIENT_ID=...                   # optional, secondary payment method
PAYPAL_CLIENT_SECRET=...
PAYPAL_WEBHOOK_ID=...

# Object storage (S3 / Cloudflare R2 / Supabase Storage — pick one)
S3_BUCKET=...
S3_REGION=...
S3_ENDPOINT=...                        # set for R2 / Supabase / MinIO
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...

# Auth (pick the block matching your provider)
# --- Casdoor (self-hosted IAM)
CASDOOR_ENDPOINT=https://auth.yourtool.com
CASDOOR_CLIENT_ID=...
CASDOOR_CLIENT_SECRET=...
CASDOOR_ORG=yourtool
CASDOOR_APP=app
# --- Ory Kratos (self-hosted)
KRATOS_PUBLIC_URL=https://auth.yourtool.com
KRATOS_ADMIN_URL=http://kratos:4434
# --- Supabase Auth
SUPABASE_URL=https://xyz.supabase.co
SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=...
# --- WorkOS / Clerk
WORKOS_API_KEY=...
CLERK_SECRET_KEY=...

# Eventing
NATS_URL=nats://nats:4222              # if using NATS JetStream
NATS_STREAM=app-events

RESEND_API_KEY=...
EMAIL_FROM="YourTool <hi@yourtool.com>"

SENTRY_DSN=...
POSTHOG_KEY=...
POSTHOG_HOST=https://app.posthog.com

OPENAI_API_KEY=...           # optional, if you have AI features
```

### 🎯 KPIs to track from day one

- Sign-ups / week
- Activation rate (signed up → activated)
- Free → paid conversion rate
- MRR / ARR
- Net revenue retention (NRR)
- Logo churn
- DAU / WAU / MAU
- p95 API latency
- Error rate
- NPS

---

## 💭 Closing Thought

A great SaaS template is **opinionated about everything that doesn't matter to the customer, and flexible about everything that does**.

- Auth, billing, tenancy, observability, admin → **opinionated**, baked-in.
- Domain models, UI flows, branding, pricing → **flexible**, configurable.

The discipline: every time you find yourself solving the *same* infrastructure problem in a *new* product, that solution belongs in the template. Every time you find yourself solving a *different* domain problem, that work belongs in `product/`.

If you internalize **§5 (Multi-Tenancy)**, **§9 (Billing)**, **§19 (Security)**, and the **§32 kernel/product split**, the rest of this playbook becomes a detailed checklist you can execute over 6–8 weeks to ship a real, professional, reusable SaaS foundation.

**Now go build.**

---
> If you found this helpful, let me know by leaving a ⭐!, or if you think this post could help someone, feel free to share it! Thank you very much! 😃

