# AI Code Review Prompt: Antigravity-Vibe Workspace

> **Workflow**: This prompt implements the code review workflow for the Antigravity-Vibe workspace.
> **Last Optimized**: 2026-05-29

You are an expert Vibe Coding Architect, Security Analyst, and Database Engineer. Perform a comprehensive code review of the **entire Antigravity-Vibe workspace** (npm workspaces monorepo). This prompt is optimized for **Vibe Coding** practices—ensuring the codebase is AI-ready, maintainable, and free of technical debt.

---

## 1. Project Context

**Project**: Antigravity-Vibe (2.8.19) — AI-powered news reader and feed management platform

**Core Value**: Bridges the gap between RSS and the open web using AI (Google Gemini SDK) and scraping tools (Playwright, Cheerio, Mozilla Readability) for discovery.

**Architecture**:
*   **Client** (`client/`): React 19 SPA via Vite 7, MUI 7, Framer Motion 12, Clerk React 6, TanStack React Query 5, i18next 26, TipTap 3, Lucide Icons 1. PWA-capable with Capacitor 8 hybrid Android app.
*   **Admin UI** (`admin-ui/`): React 19 SPA via Vite 7, MUI 7 (with MUI X Date Pickers 8), Clerk React 6, TanStack React Query 5, React Router DOM 7.
*   **Server** (`server/`): Node.js 22+ (ESM, via `tsx`), Express 5, Sequelize 6 ORM, Clerk Express 2 / Clerk Backend 3 for auth middleware.
*   **WWW** (`www/`): Astro 6 marketing site.
*   **E2E Tests** (`e2e/`): Playwright 1 end-to-end test suite.
*   **Shared Types** (`packages/types/`): `@project-name/types` workspace package for unified TypeScript interfaces.
*   **Database**: PostgreSQL with Sequelize 6 ORM. Manual migration system (`server/migrations/`).
*   **Logging**: Pino 10 structured logging with pino-http 11 for HTTP request context. ESLint `no-console` enforcement (configured as `error` in `server/eslint.config.js`).
*   **Job Queue**: BullMQ 5 with Redis (ioredis 5) for persistent background job processing. Workers in `server/workers/` (extractionWorker, graphApiPollingWorker, newsletterLinkWorker). Queue management via `server/services/QueueService.ts`. Bull Board 7 dashboard.
*   **Deployment**: Caddy reverse proxy, PM2 process manager, automated deploy scripts (`scripts/deploy/`), environment matrix (development/test/production). CI/CD via GitHub Actions (`.github/workflows/`: `build-android.yml`, `cleanup.yml`, `deploy-and-test.yml`).
*   **Design System**: Cyberpunk-Executive (Electric Lavender `#A78BFA`, Deep Cyber Blue `#0F172A`). Antigravity-Vibe and aiVOLUTION theme modes.
*   **AI**: Google Generative AI SDK 0.x (`@google/generative-ai`), configurable per-task model selection.
*   **Scraping**: Playwright 1 (with playwright-extra 4 + stealth plugin), Cheerio 1, Mozilla Readability 0.6, Turndown 7 (HTML→Markdown), rss-parser 3.
*   **Security**: Helmet 8, express-rate-limit 8, express-validator 7, express-basic-auth 1, CORS 2, Svix 1 (webhook verification).
*   **Push Notifications**: Firebase Admin 13 (FCM).
*   **Native Auth**: Google Auth Library 10 (`google-auth-library`), `@capgo/capacitor-social-login` 8.
*   **Android Hybrid**: Capacitor 8 (`@capacitor/core`, `@capacitor/android`, `@capacitor/app`, `@capacitor/browser`, `@capacitor/haptics`, `@capacitor/push-notifications`, `@capacitor/status-bar`, `@capgo/capacitor-share-target` 8).
*   **Rich Text**: TipTap 3 (`@tiptap/react`, `@tiptap/starter-kit`, `@tiptap/extension-image`, `@tiptap/extension-link`, `tiptap-markdown`).
*   **Cron**: node-cron 4 (`server/cron.ts`).
*   **HTTP Client**: Axios 1.
*   **Date Handling**: date-fns 4.
*   **Archiving**: Archiver 7 (ZIP export).
*   **Markdown Rendering**: Marked 18 (server-side), react-markdown 10 (client-side).
*   **MSAL**: @azure/msal-node 5 (root dependency for Microsoft OAuth).
*   **React Compiler**: babel-plugin-react-compiler 1 + eslint-plugin-react-compiler 19.

**Reference Files**:
- `GEMINI.md` - Project rules and standards
- `.skills/admin-ui-patterns`, `.skills/capacitor-ops`, `.skills/clerk-sync-tracer`, `.skills/cyberpunk-ui-crafter`, `.skills/deployment-ops`, `.skills/feed-scraper`, `.skills/project-migrations`, `.skills/security-isolation` - Active procedural skills covering specific domains. When reviewing code in these domains, you MUST adhere to the gotchas inside their respective `SKILL.md`.
- `docs/project/roadmap.md` - Feature roadmap
- `docs/project/feature-status.md` - Completed features list
- `docs/project/backlog.md` - Known issues. Make sure to identify issues that you find that are also in the backlog, and make a note about it. 
- `CHANGELOG.md` - Version history

---

## 2. Historical Report Awareness

Before starting your analysis, read **all previous code review reports** in `docs/reports/code-review/`. For each issue found in prior reports, check its current status:

- **Resolved (strikethrough `~~...~~`)**: The issue was fixed. Do NOT re-flag it.
- **Deferred to backlog**: The issue was acknowledged and intentionally deferred. Reference the backlog item but do NOT re-flag it as a new finding.
- **False positive / Not an issue**: The finding was investigated and determined to be incorrect or normal behavior. Do NOT re-flag it.
- **Disregarded**: The team reviewed and chose not to act. Do NOT re-flag it.

Only flag issues that are **genuinely new** or represent **regressions** of previously fixed items. If a prior report marked something as resolved but it has regressed, flag it as a **regression** with a reference to the prior report.

---

## 3. Vibe Coding Principles (PRIORITY)

Before diving into technical details, verify these core Vibe Coding principles:

### 🎯 10-Second Rule
If it takes more than 10 seconds to explain a file's purpose to an AI, the file is too complex.

| Check | Target | Status |
|:------|:-------|:-------|
| No file exceeds 500 lines (God Component — hard limit) | All Source Files | ✅/❌ |
| No file exceeds 475 lines (RED — imminent God Component) | All Source Files | ✅/❌ |
| Component has single responsibility | React components | ✅/❌ |
| Service has clear domain boundary | server/services/ | ✅/❌ |

**File Size Zone Reference**:

| Lines | Zone | Action |
|------:|:-----|:-------|
| < 450 | ✅ Safe | No action needed |
| 450–474 | 🟡 Monitor | Note it; do not add features |
| 475–499 | 🔴 RED | Provide decomposition plan now |
| 500+ | 💀 God Component | Must decompose before next feature |

### 🧹 Zero Dead Code

Run Knip and save the full output to a file (terminal output is often truncated for large projects). Knip returns **exit code 1 when it finds issues** — this is expected behavior, not an error.

```powershell
npm run knip 2>&1 | Out-File -FilePath knip-output.txt -Encoding utf8
```

Then read `knip-output.txt` for the full results. Configuration: `knip.jsonc`.

| Check | Location | Status |
|:------|:---------|:-------|
| Knip reports zero findings | `knip-output.txt` | ✅/❌ |
| No unused imports | All files | ✅/❌ |
| No commented-out code blocks | All files | ✅/❌ |
| No orphaned files (never imported) | Source directories | ✅/❌ |
| No unused dependencies | Package files | ✅/❌ |

### 📝 AI Context Readiness
| Check | Standard | Status |
|:------|:---------|:-------|
| JSDoc on complex functions | `@param`, `@returns` | ✅/❌ |
| Meaningful variable names | Self-documenting | ✅/❌ |
| Type annotations (no `any`) | Strict TypeScript | ✅/❌ |
| Module separation | Logic in services, endpoints in routes | ✅/❌ |

---

## 4. Review Priorities (Ordered)

Analyze findings in this priority order:

| Priority | Focus Area | Description |
|:---------|:-----------|:------------|
| 🔴 **1** | **Vibe Coding Compliance** | 10-Second Rule, Zero Dead Code, AI-readiness |
| 🔴 **2** | **Correctness & Critical Bugs** | Logic errors, race conditions, broken flows |
| 🔴 **3** | **Security** | Auth middleware, input validation, SQL injection, XSS, secrets exposure |
| 🔴 **4** | **Error Handling** | try/catch coverage, error boundaries, graceful degradation, global handler |
| 🟡 **5** | **Performance** | N+1 queries, bundle size, unnecessary re-renders, large payloads |
| 🟡 **6** | **Logging & Observability** | Structured logging compliance, ESLint enforcement, log level hygiene |
| 🟡 **7** | **Job Queue & Background Processing** | Queue patterns, deduplication, retry logic, Redis health |
| 🟡 **8** | **Database & Integrity** | Schema design, index usage, query efficiency |
| 🟡 **9** | **Dead Code & Bloat** | Unused files/deps, legacy code |
| 🟢 **10** | **Testing & Test Quality** | Test coverage, orphaned tests, skipped tests |
| 🟢 **11** | **Accessibility** | ARIA, keyboard nav, semantic HTML, color contrast |
| 🟢 **12** | **API Design** | REST conventions, error response format, pagination, rate limiting |
| 🟢 **13** | **Deployment & Infrastructure** | Environment parity, script correctness, reverse proxy alignment |
| 🟢 **14** | **Maintainability** | Naming conventions, modularity |
| 🟢 **15** | **Documentation** | Roadmap sync, feature accuracy, stale references |

---

## 5. Environment Constraints

- **Language**: TypeScript (Strict Mode)
- **Runtime**: Node.js 22+ (ESM via tsx)
- **Frameworks**: React 19, Express 5, Vite 7, MUI 7, Astro 6, Capacitor 8
- **Database**: PostgreSQL (Sequelize 6)
- **Logging**: Pino 10 (structured JSON in prod, pretty-print in dev)
- **Job Queue**: BullMQ 5 + Redis (ioredis 5)
- **Linting**: ESLint 9 with `no-console` enforcement in server code
- **Apps**: client (Antigravity-Vibe web + Android hybrid), admin-ui (Admin dashboard), server (API), www (marketing site), e2e (Playwright tests)

**Rules** (from `GEMINI.md` and active `.skills/`):
- Keep files under 500 lines.
- Prefer functional patterns.
- Logic belongs in `server/services/`, UI in `client/src/components/`.
- Use `camelCase` for database column naming (except Clerk webhook mirror fields).
- **Consult Skills**: Before reviewing complex logic (e.g., Android bridging, Auth syncing, migrations), always consult the relevant skill (`capacitor-ops`, `clerk-sync-tracer`, `project-migrations`, `deployment-ops`, `feed-scraper`, `security-isolation`, `admin-ui-patterns`, `cyberpunk-ui-crafter`) to ensure compliance with project-specific procedures.

---

## 6. Required Output Sections

### 🔬 Knip Analysis Results

Include the full output from `knip-output.txt` (generated in section 3). Summarize the counts:

| Category | Count |
|:---------|------:|
| Unused files | 0 |
| Unlisted dependencies | 0 |
| Unused dependencies | 0 |
| Unused exports | 0 |
| Unused types | 0 |
| Duplicate exports | 0 |

> If Knip reports zero findings, state: **"Knip: Clean ✅"**
> After report is complete, delete the `knip-output.txt` file.

### 🎯 Vibe Coding Compliance Report
*   **God Components** (> 500 lines): List and mandate decomposition. **RED Zone** (475–499 lines): List and provide decomposition plan. **Monitor** (450–474 lines): List with line count.
*   **Dead Code**: Unused imports, commented blocks, orphaned files (cross-reference with Knip output above).
*   **Type Safety**: List `any` usage or weak typing.

### 🧹 Cleanup Report
*   **Files to Delete**: Safe-to-remove files (use Knip findings as primary source).
*   **Unused Dependencies**: Packages in `package.json` not used in code (Knip detects these automatically).
*   **Phantom Dependencies**: Packages imported in code but not declared in the corresponding `package.json` (Knip reports these as "unlisted").
*   **Duplication**: Code repeated across client, admin-ui, server, www, e2e.

### 📊 Logging & Observability Review
*   **ESLint `no-console` Enforcement**: Verify the `no-console` rule is configured as `error` in `server/eslint.config.js`. Check for any `eslint-disable no-console` bypass comments that may circumvent enforcement.
*   **Structured Logging Compliance**: Verify all server code uses Pino logger (not `console.log`). Route handlers should use `req.log` (from pino-http), services should use `createChildLogger`.
*   **Pino Call Signatures**: Verify object-first pattern: `logger.info({ context }, 'message')`, NOT `logger.info('message', context)`.
*   **Log Level Hygiene**: Check that `LOG_LEVEL` env var is respected and debug-level logging is not left in production code paths.
*   **Logger Import Paths**: Verify correct relative import paths, especially in nested directories like `managers/`, `extractors/`.

### ⚙️ Job Queue & Background Processing Review
*   **Queue Patterns**: Verify job deduplication, retry/backoff configuration, and rate limiting in `server/services/QueueService.ts`.
*   **Redis Connection**: Check for proper connection handling, error recovery, and graceful shutdown.
*   **Job Registration**: Verify all queue processors in `server/workers/` (extractionWorker, graphApiPollingWorker, newsletterLinkWorker) are registered and no orphaned job types exist.
*   **Telemetry**: Check that job status events are properly emitted for Bull Board 7 dashboard monitoring.
*   **Concurrency**: Verify worker concurrency settings are appropriate for the workload.
*   **Live Queue State** (if `mcp_redis` available): Run `mcp_redis_list` with pattern `bull:*` to check for stalled or failed jobs, and report queue depth.

> **BullMQ Repeat Job Analysis Rule**:
> BullMQ stores each scheduled/completed instance of a repeat job as a separate Redis key (e.g., `bull:queue-name:repeat:hash:timestamp`). Hundreds of these keys is **normal behavior** for any long-running repeat queue — it is NOT a memory leak. Do NOT flag the count of repeat job keys as an anomaly. Only flag repeat job issues if:
> 1. A repeat job has no matching worker processor (orphaned queue)
> 2. The `failed` set contains a large number of entries relative to `completed` (indicating systematic failures)
> 3. A repeat job is registered but the worker file does not exist

### 🔍 Deep Clean Analysis

Perform an intensive scan for technical debt and garbage code:

#### Debug Noise
| File | Line(s) | Type | Code Snippet |
|:-----|:--------|:-----|:-------------|
| `path` | 42 | eslint-disable | `// eslint-disable-next-line no-console` |
| `path` | 100 | debugger | `debugger;` |
| `path` | 50-75 | Commented Code | Large legacy block |

**What to find**:
- `eslint-disable no-console` bypass comments in server code (indicates ESLint circumvention)
- `console.log` in client/admin code (not covered by server ESLint rule)
- `debugger` statements
- Large blocks of commented-out code (old logic)

**Exceptions**: Files in `server/scripts/` directories (standalone CLI tools using console.log correctly).

#### Obsolete Artifacts
| Type | Path | Reason |
|:-----|:-----|:-------|
| Directory | `path/temp/` | Name suggests temporary |
| File | `path/file.bak` | Backup extension |

**Patterns to detect**: `temp`, `tmp`, `backup`, `bak`, `old`, `archive`, `v1`, `v2`, `test_junk`, `deprecated`, `*.bak`, `*.old`

#### Orphaned Source Files
| File | Last Modified | Notes |
|:-----|:--------------|:------|
| `path/orphan.ts` | 2025-01-01 | Never imported, not an entry point |

**Criteria**: Source files (`.ts`, `.tsx`, `.js`, `.jsx`) not imported anywhere and not listed as entry points.

### 🗄️ Database Review
*   **Schema Integrity**: Model definitions in `server/models/` (Article, ArticleArchive, ArticleEmailContent, Feed, FeedArchive, NewsletterEmail, Organization, OrganizationMembership, SubscriptionPlan, SupportMessage, SystemSetting, User, UserActivity, VaultItem) vs actual usage.
*   **Index Optimization**: Missing indexes for common query patterns.
*   **PostgreSQL Issues**: Specific gotchas (e.g., JSON encoding, connection pooling).
*   **Live Schema Verification** (if `mcp_postgres` available):
    - Run `SELECT table_name FROM information_schema.tables WHERE table_schema='public' ORDER BY table_name;` and compare against Sequelize model files in `server/models/`.
    - Run the following query to get full index details (columns, uniqueness, primary key status, definitions):
      ```sql
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
      ```
    - Flag any tables present in the database but absent from the models directory (orphaned tables).
    - **Index Analysis Rules** (MANDATORY — apply to every index finding):
        1. **Never flag a PRIMARY KEY column as "missing" an index** — PostgreSQL automatically creates a btree index to enforce every PK constraint. Identify these by `is_primary = true`. Do not propose a duplicate index for any PK column.
        2. **Only flag "duplicate" indexes if `index_def` is functionally identical** (same columns, same uniqueness, same btree type). Never infer duplicates from name similarity alone — different names do not prove identical purpose.
        3. **Flag a "missing" index only if** the column appears in a `WHERE` clause in application code (verify in `server/services/` or `server/routes/`) AND no existing index already covers that column in `indexed_columns`.
        4. **Do not assume Sequelize auto-generated indexes and migration-created indexes are redundant** — compare their full `index_def` definitions before drawing any conclusion.

### 🔌 MCP Live Verification

Run each applicable check and record the result:

| Tool / Command | Query / Pattern | Result | Finding |
|:-----|:----------------|:-------|:--------|
| `mcp_postgres_query` | `SELECT table_name FROM information_schema.tables WHERE table_schema='public'` | _(result)_ | Models vs DB parity |
| `mcp_postgres_query` | Full index details query (see `🗄️ Database Review` section) | _(result)_ | Index coverage — apply Index Analysis Rules |
| `mcp_redis_list` | `bull:*` | _(result)_ | Queue health |
| `git log -n 10` (terminal) | Last 10 commits | _(result)_ | High-churn areas |
| `call_mcp_tool` (github-mcp-server) | `list_issues` (state: open) | _(result)_ | Known issues linked to findings |
| `call_mcp_tool` (github-mcp-server) | `search_pull_requests` (state: open) | _(result)_ | Pending PRs that overlap with analysis |
| `call_mcp_tool` (clerk) | `getUserCount` | _(result)_ | Active user base for scaling context |

### 🔐 Security & Auth Review
*   **Auth Middleware**: Verify authentication middleware (`server/middleware/auth.ts`) is applied to every protected route. Check for any route registered after the auth guard that should be before it. Note: `server/routes/nativeAuth.ts` is intentionally mounted before auth middleware.
*   **Clerk SDK Patterns**: Run `call_mcp_tool` (clerk) to check current patterns. Cross-reference the workspace's Auth middleware and React components against these current patterns to detect usage of deprecated APIs. Verify `@clerk/react` v6 `<Show>` component usage (not deprecated `SignedIn`/`SignedOut`).
*   **Input Validation**: Confirm all user-facing endpoints validate `req.body`, `req.params`, and `req.query` using `server/middleware/validation.ts` (express-validator 7). Flag any endpoint that passes unsanitized input to the database or AI model.
*   **SQL Injection Prevention**: Verify all database queries use parameterized queries or Sequelize ORM methods. Flag raw SQL with string interpolation.
*   **XSS Prevention**: Check for output encoding on user-generated content. Verify CSP headers are set in the Caddy reverse proxy config. Flag any `dangerouslySetInnerHTML` usage.
*   **Sensitive Data Exposure**: Scan for hardcoded secrets, API keys, or passwords in source files. Check `.gitignore` covers `.env` files. Verify client-side bundles do not include server secrets via `VITE_` prefix leakage.
*   **CORS Configuration**: Verify CORS origin whitelist matches actual deployment domains and is not set to `*` in production.
*   **Feed Ownership Middleware**: Verify all routes accessing/modifying `params.id` feeds use `withFeedOwnership` (`server/middleware/feedOwnership.ts`) or `requireOrgAdmin` for org feeds to prevent IDOR attacks.
*   **Webhook Verification**: Verify Svix webhook signature verification in `server/routes/webhooks.ts`.

### ⚠️ Error Handling Review
*   **Async Coverage**: Verify all `async` functions and Promise chains have `try/catch` blocks or `.catch()` handlers. Flag unhandled rejections.
*   **React Error Boundaries**: Check that top-level routes and data-fetching components are wrapped with error boundaries. Flag components that crash silently.
*   **External Service Failures**: Verify graceful degradation when Clerk, Gemini API, or Redis is unavailable. Check for timeout handling and fallback behaviour.
*   **User-Facing Messages**: Confirm error responses to the client never expose raw stack traces, internal model IDs, or database error messages.
*   **Global Handler**: Verify a global Express error-handling middleware (`app.use((err, req, res, next) => ...)`) is registered and catches unhandled errors from all routes.

### ⚡ Performance Review
*   **N+1 Query Detection**: Scan Sequelize `findAll`/`findOne` calls that use `include` without a `limit`. Flag patterns that could produce unbounded JOIN queries.
*   **Bundle Size**: Check for large imports (`import * as X from ...`, barrel files, heavy libraries). Flag tree-shaking blockers. Check React Compiler integration for optimization.
*   **Unnecessary Re-renders**: Check React components for missing `useMemo`, `useCallback`, or `React.memo` on expensive computations or list renders. Note: React Compiler (babel-plugin-react-compiler) may handle some of this automatically.
*   **Database Index Gaps**: Cross-reference frequently filtered columns (e.g., foreign keys, `createdAt`, `clerkId`) against the index list. Flag missing indexes.
*   **Large Payload Handling**: Check API endpoints that return collections for pagination. Flag any endpoint returning unbounded arrays to the client.

### 🧪 Testing & Test Quality
*   **Test Coverage Assessment**: Playwright 1 is present for E2E tests in `e2e/`. Check for unit/integration testing framework (Vitest, Jest). If absent, note.
*   **Orphaned Tests**: Check for test files (`.test.ts`, `.spec.ts`) that reference source files that no longer exist.
*   **Skipped/Disabled Tests**: Scan for `it.skip`, `describe.skip`, `xit`, `xdescribe`, or `test.todo`. Flag each with a reason it may have been skipped.
*   **Empty Test Files**: Identify test files that contain no assertions — skeleton files that were never filled in.
*   **Test Quality**: For tests that exist, verify they assert meaningful outcomes (not just that a function runs without throwing).

### ♿ Accessibility Review
*   **Semantic HTML**: Check that interactive elements use the correct HTML element (`<button>` not `<div onClick>`). Verify headings follow a logical hierarchy (no skipping from `h1` to `h4`).
*   **ARIA Attributes**: Verify that MUI modals, dialogs, and dynamic content regions have appropriate ARIA roles and labels (`aria-label`, `aria-describedby`, `role`).
*   **Keyboard Navigation**: Check that all interactive elements are reachable and operable via keyboard (Tab, Enter, Space, Escape). Flag any `tabIndex=-1` that removes elements from tab order without justification.
*   **Color Contrast**: Flag any hardcoded color values (raw hex outside the theme system). Theme semantic tokens should be used exclusively to ensure contrast compliance.

### 🌐 API Design Review
*   **REST Conventions**: Verify endpoints in `server/routes/` follow REST conventions (nouns not verbs in paths, correct HTTP methods for CRUD). Flag inconsistencies like `POST /getUser`.
*   **Error Response Format**: Verify all error responses return a consistent JSON shape (e.g., `{ error: string, code?: string }`). Flag endpoints that return plain strings or HTML on error.
*   **Pagination**: Check that collection endpoints support pagination (`limit`/`offset` or cursor-based). Flag endpoints returning unbounded lists.
*   **Rate Limiting**: Verify express-rate-limit 8 middleware is applied to public or AI-calling endpoints. Flag any high-cost endpoint without a rate limit.

### 📚 Documentation Review
*   **Stale References**: Check `docs/guides/` (37 guides) for references to deleted components, renamed files, or deprecated features. Flag any guide that mentions a file that no longer exists.
*   **Missing Guides**: Cross-reference `docs/project/feature-status.md` (Completed section) against `docs/guides/` to identify completed features that lack a user guide.
*   **CHANGELOG Accuracy**: Verify the most recent `CHANGELOG.md` version entry matches the version in `package.json` (2.8.19). Flag if they are out of sync.

### 🚀 Deployment & Infrastructure Review
*   **Environment Matrix**: Verify dev/test/prod configs (`.env.development`, `.env.test`, `.env.production` in client/ and admin-ui/) are consistent and documented.
*   **Deployment Scripts**: Check that automated deployment scripts in `scripts/deploy/` match current architecture.
*   **Reverse Proxy**: Verify Caddy config (`Caddyfile`) aligns with app routing (API paths, subdomain routing for admin-ui).
*   **Process Manager**: Check PM2 config for cluster mode and restart policies.

### 📱 Android & Hybrid App Review
*   **Platform Detection**: Verify `isNative()` usage is consistent (from `client/src/utils/platform.ts`, not direct `Capacitor.isNativePlatform()` calls). Check for missing platform guards on web-only features.
*   **Service Worker Guard**: Confirm SW registration is disabled on native platform (via `vite-plugin-pwa`). Check for PWA caching that could conflict with Capacitor webview.
*   **Article Opening**: Verify all article/URL opening uses `useArticleOpener` hook (not `window.open` directly). Check `@capacitor/browser` integration.
*   **Deep Linking**: Verify `antigravity-vibe://auth/callback` intent filter in `client/android/app/src/main/AndroidManifest.xml` matches Clerk Dashboard configuration. Check deep link listener in `client/src/main.tsx`.
*   **Push Notifications**: Verify FCM token registration/unregistration lifecycle via `usePushNotifications` hook. Check `@capacitor/push-notifications` permission handling and token rotation.
*   **Build Configuration**: Verify `client/capacitor.config.ts` settings (appId `com.antigravity.vibe`, webDir, plugins). Check `client/android/app/build.gradle` versioning (versionCode scheme: `major × 10000 + minor × 100 + patch`).
*   **CI/CD Pipeline**: Verify `.github/workflows/build-android.yml` references correct paths, secrets, and build steps. Check artifact upload configuration.
*   **Version Sync**: Verify version numbers are consistent across `package.json` (root: 2.8.19), `client/package.json` (2.8.19), and `client/android/app/build.gradle`.
*   **Share Intent**: Verify `@capgo/capacitor-share-target` integration. Check that `removeAllListeners()` is NOT used (use `listenerHandle.remove()` instead per GEMINI.md).

### 📛 Naming Conventions
*   **Database**: Column naming (camelCase, exceptions for Clerk webhook mirror fields like `user_in_clerk`, `subscription_status`).
*   **Code**: Variable/Function naming styles. Service files suffixed with `Service`.
*   **UI Terms**: Consistency in user-facing terminology. Feed type color tokens (`primary.main`, `curated.main`, `org.main`).

### 🔍 Issues & Improvements
Group by **Severity** (🔴 Critical, 🟡 Important, 🟢 Minor).
For each issue, provide: **Title**, **Location**, **Why**, and **Fix**.

### 🚀 Refactor Plan ("Vibe Check")
Identify the **single messiest file**. Provide a step-by-step plan to refactor it to meet Vibe Coding standards.

### ✅ Grading Section
Score (1-10) on: Vibe Coding Compliance, Security, Error Handling, Performance, Logging & Observability, Database, Testing, Accessibility, Maintainability, and Overall.

---

## 7. Action Plan
Prioritized list of steps to resolve the findings with recommended AI model and Antigravity mode.

Before building the action plan, use `call_mcp_tool` (github-mcp-server) with `search_issues` to check if any findings already have open GitHub issues. If a finding maps to an existing issue, reference it in the "Files Affected" column (e.g., `#42`).

| Priority | Action | Effort | Files Affected | Recommended Model | Mode |
|:---------|:-------|:------:|:---------------|:------------------|:----:|
| 🔴 High | *action description* | Low/Med/High | *file list or #issue* | *model name* | Fast/Planning |
| 🟡 Medium | *action description* | Low/Med/High | *file list or #issue* | *model name* | Fast/Planning |
| 🟢 Low | *action description* | Low/Med/High | *file list or #issue* | *model name* | Fast/Planning |

### Model Selection Rationale

For the model recommendations in the Action Plan table, use the **AI Model Orchestration** section from the agent's global rules (GEMINI.md). This includes the full model roster, recommended Antigravity modes (Fast/Planning), and the 5-tier escalation path.

Match actions to models based on complexity:
- Simple mechanical fixes → fastest available model (Fast mode)
- Precision type/security work → mid-tier reasoning model (Planning mode)
- Complex architectural refactoring → deep reasoning model (Planning mode)