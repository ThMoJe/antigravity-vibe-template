# Architecture Review Report (2026-05-29)

## 🔬 Phase 1 Data Summary

| Data Source | Key Metrics | Finding |
|:------------|:------------|:--------|
| Database tables | 14 core tables + 1 SequelizeMeta | The schema matches `database-er-diagram.md` perfectly with entities properly segmented. |
| Row counts | `Articles` (172), `UserActivities` (138), `ArticleArchives` (105) | Database size is ~11 MB. `Articles` and `UserActivities` represent the highest volume tables, indicating high event ingestion. |
| Index coverage | Comprehensive FK coverage + compound indexes | Critical compound indexes (e.g., `Articles(feed_id, published_date)`) are present. Multi-tenant boundaries (`userId`, `organizationId`) are correctly indexed. |
| Queue state | 4 distinct BullMQ queues | `extraction`, `scan`, `graph-api-polling`, and `newsletter-link-queue` active. No stalled/failed keys returned in current snapshot. |
| Commit velocity | High churn in configuration | `backlog.md`, `package.json`, and deployment scripts (`ci-update.sh`) are hot files. Stable core backend. |
| Open issues | 0 open architecture issues | No major architectural debt items tracked in current issues. |

## ⚡ Architecture Vibe Rating

| Dimension | Score | Note |
|:----------|:-----:|:-----|
| System topology & modularity | 9/10 | Clean workspace separation. `packages/types` successfully handles cross-boundary contracts. |
| Data architecture & multi-tenancy | 8/10 | Excellent tenant isolation (`userId`/`organizationId`). Archiving model works but lacks cold-storage transition for scale. |
| Identity & auth robustness | 8/10 | Clerk + Native OAuth is well-integrated. Webhook-only sync poses a minor single-point-of-failure risk without a polling fallback. |
| Background processing maturity | 9/10 | BullMQ handles scale gracefully. Separation of concerns across the 4 queues is highly mature. |
| AI integration architecture | 8/10 | `aiService.ts` abstraction is solid. Ranked discovery pipelines (`FEED.contentSelectorsRanked`) represent excellent agentic design. |
| Mobile/hybrid architecture | 8/10 | Capacitor 8 bridge is strong. Recent Native Android OAuth (v2.8.16) resolved significant WebView pain points. |
| Deployment & operational readiness | 9/10 | Automated DB backups, 6-job CI/CD pipeline, and Better Stack monitoring (v2.8.7) provide excellent operational safety. |
| Scaling readiness | 8/10 | PM2 allows for horizontal scaling. Single Postgres instance is the inevitable bottleneck but has huge runway left. |
| **Overall Architecture Score** | **8.4/10** | Production-ready architecture. The system is extremely mature for v2.8.x with strong foundational observability and isolation. |

## 🔴 The "Critical 3"

| # | Risk | Evidence | Consequence (if unaddressed) | Mitigation |
|:-:|:-----|:---------|:-----------------------------|:-----------|
| 1 | Unbounded `UserActivity` Growth | `UserActivities` is already the second largest table (138 rows for 20 users). | Rapid DB bloat as active users generate hundreds of interaction events daily, degrading index performance. | Implement a retention policy (e.g., 90 days) or transition historical analytics to an OLAP sink. |
| 2 | Clerk Webhook Dependency | Identity heavily relies on `server/routes/webhooks.ts`. | If the webhook endpoint drops payloads or Caddy fails to route them, local DB becomes desynced from Clerk identity. | Implement a daily fallback cron sync or on-login drift check against Clerk's backend API. |
| 3 | AI Extraction Fallback Limits | `FEED.fallbackEngine` relies on `firecrawl`. | If Playwright fails and Firecrawl rate-limits/errors, feed extraction completely stalls. | Introduce a circuit breaker and exponential backoff in the extraction queue to prevent spamming broken targets. |

## 🏗️ Architectural Recommendations

#### Structure & Module Boundaries
| Recommendation | Effort | Impact | Rationale |
|:---------------|:------:|:------:|:----------|
| Enforce strict API boundary testing | Med | Med | E2E Playwright tests exist, but lower-level contract testing between `client/` and `server/` using the shared types would prevent regressions. |

#### Data Architecture
| Recommendation | Effort | Impact | Rationale |
|:---------------|:------:|:------:|:----------|
| `UserActivity` Retention Job | Low | High | A simple node-cron job to purge or aggregate `UserActivities` older than X days will prevent the table from becoming the largest bottleneck. |
| pgvector evaluation | Med | High | The roadmap dictates "Dynamic Categorization" and "Vølve Feed" (Phase 1/2). Incorporating `pgvector` into the DB stack now ensures readiness. |

#### Technology Stack Fitness
| Recommendation | Effort | Impact | Rationale |
|:---------------|:------:|:------:|:----------|
| Circuit Breakers on External APIs | Low | Med | Protecting Microsoft Graph API and Firecrawl calls with `opossum` or similar circuit breakers improves queue worker resilience. |

#### Operational Patterns
| Recommendation | Effort | Impact | Rationale |
|:---------------|:------:|:------:|:----------|
| Consolidate PM2 Ecosystem | Low | Med | Replacing ad-hoc PM2 commands with a dedicated `ecosystem.config.js` will standardize environment configurations per the roadmap plan. |

## 🗓️ 12-Month Technical Evolution Roadmap

| Quarter | Theme | Key Actions | Success Criteria |
|:--------|:------|:------------|:-----------------|
| Q3 2026 | App Launch & Stability | Finalize Play Store launch. Consolidate PM2. Add fallback identity sync. | 0 critical downtime incidents. App live on Play Store. |
| Q4 2026 | Intelligence Baseline | Introduce `pgvector`. Implement Vibe Tags and Dynamic Categorization. | Semantic similarity search operational in DB. |
| Q1 2027 | Proactive Assistant | Build "Morning Briefing" synthesis. Implement Vølve (For You) Feed. | Push notifications delivering personalized daily briefings. |
| Q2 2027 | Enterprise Scale | Build Gated Content Scraping. Scale out BullMQ workers horizontally. | Multi-node worker deployment successfully processing B2B feeds. |

## 📚 Architecture Documentation Review

| Document | Current Claim | Reality | Required Update |
|:---------|:--------------|:--------|:----------------|
| `database-er-diagram.md` | Detailed and accurate mapping of v2.8 schema. | Matches local DB state perfectly (14 models + Meta). | Keep as-is. High quality. |
| `deployment.md` | Outlines the production topology. | Mostly accurate, but `ecosystem.config.js` is still planned (not fully enforced). | Add specific section detailing the new Better Stack monitoring integration (v2.8.7). |

## 🤝 Cross-Reference with Other Reviews

| Finding | Delegate To | Why |
|:--------|:------------|:----|
| Ensure `UserActivity` cleanup is scheduled. | Code Review / Feature | Requires a specific cron job script to be written. |
| Stale configurations in `pm2` command scripts. | Spring Cleaning | Scripts can be replaced by a unified `ecosystem.config.js`. |
