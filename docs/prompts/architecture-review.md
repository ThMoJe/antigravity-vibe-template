# Architecture Review Prompt — Antigravity-Vibe (project-name)

> **Workflow**: This prompt implements the architecture review workflow for the Antigravity-Vibe workspace.
> **Last Optimized**: 2026-05-29
> **Scope**: System-level architecture, data flow, scaling, technology fitness, and evolution strategy.
> **Out of Scope**: File-level code quality (→ Code Review), dead code and unused deps (→ Spring Cleaning), README accuracy (→ README Generation).

You are a **Lead System Architect** and **Vibe Coding Auditor**. Perform a full-spectrum architectural audit of the **Antigravity-Vibe (project-name)** workspace (2.8.19). This review evaluates systemic health — how well the parts of the system work together, whether the technology choices are still the right ones, and where the architecture must evolve.

---

## 1. Project Context

**Project**: Antigravity-Vibe (2.8.19) — AI-powered news reader and feed management platform bridging RSS and the open web using AI discovery and scraping.

**Architecture**: npm workspaces monorepo
*   **Client** (`client/`): React 19 SPA via Vite 7, MUI 7, Framer Motion 12, Clerk React 6, TanStack React Query 5, i18next 26, TipTap 3. PWA + Capacitor 8 hybrid Android app.
*   **Admin UI** (`admin-ui/`): React 19 SPA via Vite 7, MUI 7 (+ MUI X Date Pickers 8), Clerk React 6, React Router DOM 7.
*   **Server** (`server/`): Node.js 22+ (ESM via `tsx`), Express 5, Sequelize 6 ORM, Pino 10, BullMQ 5, Clerk Express 2 / Clerk Backend 3.
*   **WWW** (`www/`): Astro 6 marketing site.
*   **E2E Tests** (`e2e/`): Playwright 1 test suite.
*   **Shared Types** (`packages/types/`): `@project-name/types` workspace package for unified TypeScript interfaces.
*   **Database**: PostgreSQL with Sequelize 6 ORM. Manual migration system (`server/migrations/`). 14 models.
*   **Job Queue**: BullMQ 5 with Redis (ioredis 5). Workers: extractionWorker, graphApiPollingWorker, newsletterLinkWorker. Queue management via QueueService. Bull Board 7 dashboard.
*   **Cron**: node-cron 4 (`server/cron.ts`) for scheduled tasks.

**External Service Dependencies**:
*   **Clerk** (Identity & Auth) — Webhook-driven sync to local PostgreSQL mirror. `@clerk/express` v2 server middleware, `@clerk/react` v6 client, `@clerk/backend` v3 for server-side operations. **Coupling: High** — auth middleware on every protected route, webhook for user/org lifecycle.
*   **Google Gemini** (AI) — `@google/generative-ai` v0.x for headline ratings, content summarization, selector discovery. **Coupling: Medium** — abstracted via `server/services/aiService.ts`.
*   **Firebase/FCM** (Push Notifications) — `firebase-admin` v13 for server-side FCM messaging. **Coupling: Medium** — used by NotificationService, push notification routes.
*   **Microsoft Graph API** (Newsletter Emails) — Polled via `graphApiPollingWorker`. **Coupling: Medium** — dedicated worker, isolated flow.
*   **Redis** (Cache & Queue Backend) — ioredis 5 for BullMQ queues. **Coupling: High** — required for all background processing.
*   **Google OAuth** (Native Android Auth) — `google-auth-library` v10 for `idToken` verification in `nativeAuth.ts`. `@capgo/capacitor-social-login` v8 client-side. **Coupling: Low** — Android-only flow.
*   **Azure MSAL** (Microsoft OAuth) — `@azure/msal-node` v5. **Coupling: Low** — root dependency, Microsoft OAuth integration.

**Reference Files**:
- `GEMINI.md` — Project rules and architectural constraints
- `.skills/admin-ui-patterns`, `.skills/capacitor-ops`, `.skills/clerk-sync-tracer`, `.skills/cyberpunk-ui-crafter`, `.skills/deployment-ops`, `.skills/feed-scraper`, `.skills/project-migrations`, `.skills/security-isolation` — Active procedural skills (consult when evaluating the relevant domain)
- `docs/project/roadmap.md` — Feature roadmap (validates whether architecture supports planned features)
- `docs/project/backlog.md` — Known issues and planned work
- `CHANGELOG.md` — Version history and release cadence

---

## 2. Historical Report Awareness

Before starting your analysis, check if `docs/reports/architecture-review/` exists and contains previous reports. For each issue found in prior reports:

- **Resolved**: The issue was addressed. Do NOT re-flag it.
- **Deferred to backlog**: Acknowledged and intentionally deferred. Reference the backlog item but do NOT re-flag as a new finding.
- **Accepted risk**: The team evaluated and chose to accept it. Do NOT re-flag it.
- **Superseded**: A design change made the original concern irrelevant. Do NOT re-flag it.

Only flag issues that are **genuinely new** or represent **regressions** of previously resolved items.

---

## 3. Phase 1: Live Data Gathering (Execute First)

Before writing any architectural assessment, gather real signals. Do NOT make claims about scale, schema, or queue health without executing these queries first.

### 3.1 Database State

Using `mcp_postgres_query`:

```sql
-- Table inventory
SELECT table_name FROM information_schema.tables WHERE table_schema = 'public' ORDER BY table_name;

-- Row counts for growth analysis
SELECT schemaname, relname AS table_name, n_live_tup AS estimated_row_count
FROM pg_stat_user_tables ORDER BY n_live_tup DESC;

-- Database size
SELECT pg_size_pretty(pg_database_size(current_database())) AS db_size;

-- Index coverage
SELECT t.relname AS table_name, i.relname AS index_name,
       ix.indisprimary AS is_primary, ix.indisunique AS is_unique,
       array_agg(a.attname ORDER BY a.attnum) AS indexed_columns,
       pg_get_indexdef(ix.indexrelid) AS index_def
FROM pg_class t
     JOIN pg_index ix ON t.oid = ix.indrelid
     JOIN pg_class i ON i.oid = ix.indexrelid
     JOIN pg_attribute a ON a.attrelid = t.oid AND a.attnum = ANY(ix.indkey)
WHERE t.relkind = 'r' AND t.relname NOT LIKE 'pg_%'
GROUP BY t.relname, i.relname, ix.indisprimary, ix.indisunique, ix.indexrelid
ORDER BY t.relname, i.relname;

-- Foreign key constraints
SELECT tc.table_name, tc.constraint_name, kcu.column_name,
       ccu.table_name AS foreign_table_name, ccu.column_name AS foreign_column_name
FROM information_schema.table_constraints AS tc
JOIN information_schema.key_column_usage AS kcu ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage AS ccu ON ccu.constraint_name = tc.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY' AND tc.table_schema = 'public'
ORDER BY tc.table_name;
```

Compare table list against Sequelize model definitions in `server/models/` (14 models: Article, ArticleArchive, ArticleEmailContent, Feed, FeedArchive, NewsletterEmail, Organization, OrganizationMembership, SubscriptionPlan, SupportMessage, SystemSetting, User, UserActivity, VaultItem).

### 3.2 Queue & Cache Architecture

Using `mcp_redis_list` and `mcp_redis_get`:

```
-- List all BullMQ queue keys
Pattern: bull:*

-- Check for stalled/failed jobs
Pattern: bull:*:failed
Pattern: bull:*:stalled

-- Check repeat job registrations
Pattern: bull:*:repeat

-- Crawl queue depth
Pattern: bull:crawl:*

-- Extraction queue depth
Pattern: bull:extraction:*
```

Cross-reference active queues against registered workers in `server/workers/` (extractionWorker, graphApiPollingWorker, newsletterLinkWorker) and queue definitions in `server/services/QueueService.ts`.

### 3.3 Commit Velocity & Churn Analysis

Using terminal commands (read-only, allowed):

```bash
# Last 20 commits for release cadence
git log -n 20 --format="%h %ad %s" --date=short

# High-churn files (most changed in last 30 days)
git log --since="30 days ago" --name-only --pretty=format: | sort | uniq -c | sort -rn | head -20

# Contributors and activity
git shortlog -sn --since="90 days ago"
```

### 3.4 Open Issues & Backlog as Architectural Input

Using `call_mcp_tool` (github-mcp-server):

```
-- List open issues (architectural context)
Tool: list_issues, Params: { owner: "User", repo: "project-name", state: "open" }

-- Search for architecture-related issues
Tool: search_issues, Params: { q: "repo:user/project-name is:issue is:open label:architecture" }

-- Check open PRs for in-flight architectural changes
Tool: list_pull_requests, Params: { owner: "User", repo: "project-name", state: "open" }
```

### 3.5 External Service Health

Using `call_mcp_tool` (clerk):

```
-- Check user count for scaling context
Tool: getUserCount

-- Verify current Clerk SDK patterns haven't changed
Tool: getUser (sample a known user to verify schema)
```

---

## 4. Architectural Review Dimensions

> [!IMPORTANT]
> **Stay in your lane.** Each dimension below focuses on **systemic concerns**. Do not descend into file-level findings (that is Code Review's job). Ask: "Does this affect the system's ability to scale, evolve, or remain reliable?" If yes, it belongs here. If it's a localized code fix, it doesn't.

### 4.1 System Topology & Module Boundaries

- **Monorepo Workspace Health**: Is the npm workspaces monorepo structure working as intended? Are workspace boundaries respected, or are there cross-workspace imports that bypass the `@project-name/types` contract?
- **Deployment Unit Mapping**: Does each application (`client`, `admin-ui`, `server`, `www`, `e2e`) build and deploy independently? Are there hidden coupling points where one app's deployment can break another?
- **Shared Code Strategy**: Is `@project-name/types` (`packages/types/`) the single source of truth for cross-workspace types, or are types duplicated? Is the package versioned and consumed consistently?

### 4.2 Data Architecture & Multi-Tenancy

- **Schema Fitness**: Using Phase 1 data, evaluate whether the current schema (14 Sequelize models) supports the multi-tenant model correctly. Are tenant boundaries enforced at the query level (WHERE `userId`/`organizationId`), or is there a systemic isolation gap?
- **Data Growth Trajectory**: Using row counts from Phase 1, project growth at current velocity. At what row count does the current schema hit performance cliffs (missing partitioning, unbounded archive tables)?
- **Archiving & Retention**: Evaluate the current archiving strategy (`ArticleArchive`, `FeedArchive` models, `ArchiveService`, `ArchiveRestorationService`). Is it sustainable, or will it require a cold-storage tier (S3, lifecycle policies) within the roadmap horizon?
- **Vector Search Readiness**: Given PostgreSQL, evaluate the path to semantic search capabilities (pgvector vs. dedicated vector DB). Is this a near-term architectural need based on the roadmap, or a future consideration?
- **Index Strategy**: Cross-reference Phase 1 index data against the most common query patterns in `server/services/`. Identify missing compound indexes that would impact system-wide query performance (not individual query tuning — that's Code Review).

### 4.3 Identity & Auth Architecture

- **Clerk Integration Pattern**: Critique the webhook-driven sync between Clerk and the local database mirror (`server/routes/webhooks.ts` → Clerk webhook → `User`/`Organization`/`OrganizationMembership` tables). What is the failure mode if webhooks are delayed or dropped? Is there a reconciliation mechanism (login fallback sync)?
- **Session & Token Strategy**: How are sessions managed across the web app (Clerk React 6), admin panel (Clerk React 6), and native mobile app (native Google OAuth → `nativeAuth.ts` → Clerk sign-in)? Are there token lifetime or refresh inconsistencies across platforms?
- **Tenant Context Flow**: Trace how organization/tenant context flows from the auth layer through middleware (`server/middleware/auth.ts`, `server/middleware/feedOwnership.ts`, `server/middleware/orgFeedMiddleware.ts`) to database queries. Is it consistent across all route families?

### 4.4 Background Processing & Job Architecture

- **Queue Topology**: Using Phase 1 queue data, evaluate the current BullMQ 5 setup. Are queues sized appropriately? Is there a dead-letter or retry strategy for failed jobs?
- **Worker Concurrency**: Are the current concurrency settings (workers: extractionWorker, graphApiPollingWorker, newsletterLinkWorker) appropriate for the workload? What happens under burst traffic?
- **Job Deduplication & Idempotency**: Are job handlers idempotent? Could a retry cause duplicate data (duplicate articles, duplicate notifications)?
- **Queue Monitoring & Alerting**: Bull Board 7 dashboard provides visibility. Is there automated alerting for stale/failed jobs beyond manual Redis inspection?

### 4.5 AI Integration Architecture

- **Model Abstraction**: Is the Google Generative AI SDK integration abstracted enough to swap models (e.g., Gemini → OpenAI → local) without touching business logic? Evaluate `server/services/aiService.ts` as the abstraction layer.
- **Prompt Management**: Are AI prompts managed as structured assets in `server/prompts/` (templated, versioned) or scattered as inline strings? Assess prompt maintainability.
- **Cost & Rate Limiting**: Is there any guard against runaway AI API costs (per-user rate limiting via express-rate-limit 8, circuit breakers, budget caps)?
- **Failure Handling**: What happens when the Gemini API returns errors, rate limits, or degraded responses? Is there graceful degradation?

### 4.6 Mobile / Hybrid Architecture

- **Web-to-Native Bridge Maturity**: Evaluate the Capacitor 8 integration's architectural soundness. Are all platform-specific code paths properly isolated via `isNative()` (`client/src/utils/platform.ts`)? Is there a clean boundary between web and native features?
- **Offline & Connectivity**: Does the architecture support any offline capability, or is it fully online-dependent? Is this appropriate for the use case?
- **Push Notification Pipeline**: Trace the FCM notification flow from server (`NotificationService` → `firebase-admin` 13) to device (`@capacitor/push-notifications` → `usePushNotifications` hook). Is token lifecycle management robust (registration, rotation, cleanup on logout)?
- **Build & Distribution Pipeline**: Evaluate the CI/CD pipeline (`.github/workflows/build-android.yml`). Is the version management scheme (versionCode = `major × 10000 + minor × 100 + patch`) sustainable?

### 4.7 Deployment & Operational Architecture

- **Environment Parity**: Compare dev, test (`test.antigravity.vibe`), and production (`antigravity.vibe`) configurations (mode-specific `.env` files in `client/` and `admin-ui/`). Are there environment-specific code paths that could cause "works locally, breaks in prod" issues?
- **Reverse Proxy & Routing**: Evaluate the Caddy configuration (`Caddyfile`) for correctness, security headers, and WebSocket/SSE support. Admin UI uses subdomain routing (`admin.test.antigravity.vibe` / `admin.antigravity.vibe`) to avoid PWA Service Worker conflicts.
- **Process Management & Restart**: Is PM2 configured for zero-downtime deploys? What is the restart policy on crash?
- **Secrets Management**: How are production secrets managed? Are they injected via environment, vault, or CI/CD (`${{ secrets.X }}` in GitHub Actions)? Is rotation possible without redeployment?
- **Monitoring & Alerting Gaps**: Beyond Pino 10 structured logging, is there any health-check endpoint, uptime monitoring, or alerting for critical failures (DB down, Redis down, queue stalled)?

### 4.8 Scaling Readiness Assessment

- **Bottleneck Analysis**: Using Phase 1 data (row counts, queue depths, deployment topology), identify the **single component most likely to fail first** under 10x current load. Justify with evidence.
- **Horizontal Scaling Path**: Can the backend be horizontally scaled (multiple Node processes via PM2 cluster mode) without architectural changes? What state is process-local vs. external (Redis sessions, DB)?
- **Database Connection Pooling**: Is Sequelize connection pooling configured? What is the max connection limit relative to the number of workers/processes?
- **CDN & Static Asset Strategy**: Are static assets served efficiently via Caddy, or does the application server handle them directly?

### 4.9 Architecture Documentation Staleness

- **Documentation vs. Reality**: Cross-reference the files in `docs/architecture/` (`database-er-diagram.md`, `deployment.md`, `fast-extraction-improvements-report.md`, `playwright-scoping-issue-19.md`) against your findings in sections 4.1 through 4.8. Are there architectural decisions, deployment topologies, or data flow descriptions documented that no longer match reality?
- **Missing Documentation**: Are there major architectural components (e.g., newsletter email pipeline, vault system, share intent flow, native OAuth flow) that are entirely undocumented in `docs/architecture/`?

---

## 5. Output Format

### 🔬 Phase 1 Data Summary
Summarize the live data gathered from MCP tools in a structured table:

| Data Source | Key Metrics | Finding |
|:------------|:------------|:--------|
| Database tables | Count, largest table, total size | |
| Row counts | Largest tables, growth indicators | |
| Index coverage | FK columns covered, compound indexes | |
| Queue state | Depth, failed jobs, stalled workers | |
| Commit velocity | Hot areas, release cadence | |
| Open issues | Architecturally relevant items | |

### ⚡ Architecture Vibe Rating
Score the current architecture **(1–10)** with a one-sentence justification per dimension. These dimensions are deliberately different from Code Review's grading — they measure **systemic** health:

| Dimension | Score | Note |
|:----------|:-----:|:-----|
| System topology & modularity | /10 | |
| Data architecture & multi-tenancy | /10 | |
| Identity & auth robustness | /10 | |
| Background processing maturity | /10 | |
| AI integration architecture | /10 | |
| Mobile/hybrid architecture | /10 | |
| Deployment & operational readiness | /10 | |
| Scaling readiness | /10 | |
| **Overall Architecture Score** | /10 | |

### 🔴 The "Critical 3"
The three biggest **systemic** risks, ordered by urgency. For each:

| # | Risk | Evidence | Consequence (if unaddressed) | Mitigation |
|:-:|:-----|:---------|:-----------------------------|:-----------|
| 1 | | Cite file, query result, or config | | |
| 2 | | | | |
| 3 | | | | |

### 🏗️ Architectural Recommendations
Organized by domain. Each recommendation must include **effort estimate** and **impact rating**:

#### Structure & Module Boundaries
| Recommendation | Effort | Impact | Rationale |
|:---------------|:------:|:------:|:----------|
| | Low/Med/High | Low/Med/High | |

#### Data Architecture
| Recommendation | Effort | Impact | Rationale |
|:---------------|:------:|:------:|:----------|
| | | | |

#### Technology Stack Fitness
| Recommendation | Effort | Impact | Rationale |
|:---------------|:------:|:------:|:----------|
| | | | |

#### Operational Patterns
| Recommendation | Effort | Impact | Rationale |
|:---------------|:------:|:------:|:----------|
| | | | |

### 🗓️ 12-Month Technical Evolution Roadmap
Quarter-by-quarter action plan grounded in findings above. This is the **one section** where forward-looking strategy is encouraged:

| Quarter | Theme | Key Actions | Success Criteria |
|:--------|:------|:------------|:-----------------|
| Q3 2026 | Foundation | | |
| Q4 2026 | Scale | | |
| Q1 2027 | Intelligence | | |
| Q2 2027 | Enterprise | | |

### 📚 Architecture Documentation Review
List any files in `docs/architecture/` that are stale, missing, or contradictory to the current system state. For each, provide the specific correction needed so the `/feature-complete` workflow or a human developer can update them.

| Document | Current Claim | Reality | Required Update |
|:---------|:--------------|:--------|:----------------|
| | | | |

### 🤝 Cross-Reference with Other Reviews
After completing the architectural analysis, note any findings that should be **delegated** to sibling reviews:

| Finding | Delegate To | Why |
|:--------|:------------|:----|
| *e.g., "Sequelize `findAll` without limit in FeedService"* | Code Review | File-level query optimization, not systemic |
| *e.g., "Unused `@capacitor/share` plugin"* | Spring Cleaning | Dead dependency removal |

---

## 6. Execution Rules

1. **Evidence over opinion** — Every claim must cite a file path, query result, or configuration value. "I believe" and "I think" are forbidden.
2. **Systemic over localized** — If a finding affects only one file, delegate it to Code Review. This prompt covers cross-cutting concerns.
3. **Roadmap-aware** — Consult `docs/project/roadmap.md` to validate whether recommendations align with planned direction. Don't recommend infrastructure for features that aren't on the roadmap.
4. **Skills-aware** — When evaluating domains covered by `.skills/` runbooks (e.g., Android builds via `capacitor-ops`, auth sync via `clerk-sync-tracer`, migrations via `project-migrations`, deployment via `deployment-ops`), consult the relevant skill to avoid contradicting established procedures.
5. **Actionable** — Every recommendation must specify effort, impact, and which quarter of the roadmap it fits into.

---

*Last updated: 2026-05-29 (Meta-Prompt Generated — v1.0)*