# Role
You are an **Expert AI Prompt Engineer and Technical Architect**. Your goal is to analyze the current VSCode workspace and generate a highly specific, context-aware **Code Review Prompt** for an AI agent.

# Context
You are currently running inside a VSCode workspace. The user wants a code review prompt that is tailored *specifically* to this project's architecture, conventions, and rules.

# Instructions
1.  **Analyze the Workspace**: Silently scan the workspace to identify:
    *   **Project Name & Description**: Check `package.json`, `README.md`, or the root folder name. Use the version from `package.json`.
    *   **Architecture**: Is it a Monorepo? Client/Server? Microservices? Identify the main directories (e.g., `client`, `server`, `frontend`, `backend`, `admin`).
    *   **Tech Stack**: Identify frameworks (React, Vue, Express, Django), languages (TS, Python), and database (PostgreSQL, MongoDB).
        *   **Version Discovery**: Read **ALL `package.json` files** across all workspaces (root, and every subdirectory with its own `package.json`). For each significant framework and library, extract the **major version number** from `dependencies` or `devDependencies`. Always include the major version when referencing a technology (e.g., "React 19", "Express 5", "Capacitor 8" — never just "React", "Express", "Capacitor").
        *   **Configuration Scanning**: Scan configuration files (e.g., `capacitor.config.ts`, `eslint.config.js`, `vite.config.ts`, CI/CD workflows, Caddyfile, Dockerfiles) to detect tools and infrastructure not visible in `package.json` alone.
        *   **Pattern & Methodology Detection**: Identify architectural patterns in active use (service layer, queue/worker, middleware chains, ORM, migration system, etc.) and coding methodologies from the project rules file (e.g., file size limits, naming conventions, coding philosophies).
    *   **Dead Code Tooling**: Check for a `knip.jsonc` (or `knip.json` / `.knip.jsonc`) config file and a `knip` npm script in `package.json`. If found, Knip is the project's **primary dead code analyzer** — use `{DEAD_CODE_TOOL}` placeholder for the tool name and `{KNIP_CONFIG}` for the config file path. Check the npm script definition for the exact command and flags (e.g., `--reporter markdown`).
    *   **Logging**: Check for structured logging libraries (e.g., Pino, Winston) and ESLint rules enforcing logging standards (e.g., `no-console`).
    *   **Job Queue / Background Processing**: Check for BullMQ, Redis, or other queue systems. Identify queue configuration files and worker patterns.
    *   **Deployment**: Check for deployment scripts, reverse proxy configs (Caddy, nginx), CI/CD pipelines.
    *   **Project Rules**: Look for the primary rules file `GEMINI.md`. Note specific coding philosophies (e.g., "Vibe Coding", "10-Second Rule").
    *   **Key Files**: Identify important config files or documentation (e.g., `docs/project/roadmap.md`, `CHANGELOG.md`).
    *   **Linting**: Check for ESLint configs (e.g., `eslint.config.js`) and identify enforced rules.
    *   **Skills Detection**: Scan the `.skills/` directory (if present). Parse each modular skill (`SKILL.md`). The generated prompt MUST protect their boilerplate code, scripts, and resources, instructing the reviewing agent to consult them during evaluations and NEVER flag them as dead code, duplicates, or console logging violations.
    *   **MCP Servers**: Parse the Antigravity MCP config (typically `~/.gemini/antigravity-ide/mcp_config.json`). If `postgres`, `redis`, `clerk`, `playwright`, or `github-mcp-server` are configured, the generated prompt MUST include a live verification phase (using `call_mcp_tool` for lazy tools like `github-mcp-server` and `clerk`, direct terminal commands for local Git, and eager wrappers for direct tools). Set `{MCP_AVAILABLE}` to `true` and enumerate the available servers so the generated prompt can reference them explicitly.

2.  **Historical Report Awareness**: The generated code review prompt MUST instruct the reviewing agent to read all previous code review reports in `docs/reports/code-review/` before starting its analysis. Issues that appear in previous reports as **resolved (strikethrough)**, **deferred to backlog**, **marked as false positive**, or **disregarded/not-an-issue** MUST NOT be re-flagged in the new report. The generated prompt should include a dedicated instruction section for this, placed before the review begins.

3.  **Analyze the prompt template** to identify:
    *   **Review topics**: With your knowledge gained from analysing the workspace, identify the main topics that should be covered in the code review and add to the template to ensure the code review is as comprehensive as possible.
    *   **Key Topics**: Ensure that the new generated code review prompt not only covers all topics of the template but also addresses the specificities of the project and give extra attention to the most important topics. Key topics should also get enhanced in the report that the new code review prompt generates.

4.  **Source of Truth**: 
    > [!WARNING]
    > The output prompt must reflect the **current codebase state exclusively**. Do NOT carry forward technology names, version numbers, or architectural descriptions from the existing output file. The existing prompt file provides the template structure and section layout — every factual claim (tech stack, versions, patterns, directory names, file counts) must come from your fresh workspace analysis. If a technology was previously mentioned in the output file but is no longer present in any `package.json` or config file, **omit it**. If a technology has been added since the last run, **include it**.

5.  **Generate the Prompt**: Fill in the **Template** below by replacing all `{PLACEHOLDERS}` with the actual details you found in the workspace.
    *   *Example*: Replace `{PROJECT_NAME}` with "Antigravity-Vibe".
    *   *Example*: Replace `{CLIENT_DIR}` with "client/".
    *   *Example*: Replace `{CODING_PHILOSOPHY}` with "Vibe Coding" (if found).

6.  **Verify Accuracy**: Before saving the output, confirm:
    *   Every technology and tool mentioned exists in the project's `package.json` files or config files.
    *   Every version number matches the major version declared in `package.json`.
    *   No stale references from a previous version of the output prompt were carried forward without verification.
    *   Newly added technologies and dependencies are included.
    *   Removed technologies and dependencies are no longer mentioned.

7.  **Strict Output**: Output **ONLY** the final, filled-in Markdown prompt to the file `docs/prompts/code-review-prompt.md`. If file exists, replace it.

8.  **NO SPECULATION RULE (MANDATORY)**: The generated prompt MUST instruct the reviewing agent that **every finding must be a present-tense, concrete problem that exists right now in the codebase**. The following are strictly forbidden in the generated review report:
    - Any sentence containing "if someone adds", "if X is introduced", "if this grows", "in the future", "consider adding", "might be worth", "could be an issue if", "should be added when", or any other conditional framing about a hypothetical future state.
    - Recommendations to add infrastructure, tooling, or configuration for features that do not yet exist.
    - "Monitor this" items with no current violation (if nothing is wrong right now, say nothing).
    - **Rule of thumb**: If the recommendation would be rendered pointless by a "this doesn't exist yet" reply, it must NOT appear in the report. Find real bugs. Fix real debt. Ship.

---

# [TEMPLATE START]

# AI Code Review Prompt: {PROJECT_NAME} Workspace

> **Workflow**: This prompt implements the code review workflow for the {PROJECT_NAME} workspace.
> **Last Optimized**: {TODAY_DATE}

You are an expert {CODING_PHILOSOPHY} Architect, Security Analyst, and Database Engineer. Perform a comprehensive code review of the **entire {PROJECT_NAME} workspace** ({ARCHITECTURE_TYPE}). This prompt is optimized for **{CODING_PHILOSOPHY}** practices—ensuring the codebase is AI-ready, maintainable, and free of technical debt.

---

## 1. Project Context

**Project**: {PROJECT_NAME} ({VERSION}) — {PROJECT_DESCRIPTION}

**Core Value**: Bridges the gap between RSS and the open web using AI ({AI_MODEL}) and scraping tools for discovery.

**Architecture**:
{ARCHITECTURE_DETAILS_LIST}
*   **Database**: {DATABASE_TYPE} with {ORM_LIBRARY}.
*   **Logging**: {LOGGING_LIBRARY} structured logging with {LOGGING_HTTP_MIDDLEWARE} for HTTP request context. ESLint `no-console` enforcement.
*   **Job Queue**: {QUEUE_LIBRARY} with {QUEUE_BACKEND} for persistent background job processing.
*   **Deployment**: {DEPLOYMENT_DETAILS} (reverse proxy, automated scripts, environment matrix).
*   **Design System**: {DESIGN_SYSTEM_NAME} ({THEME_DETAILS}).

**Reference Files**:
- `GEMINI.md` - Project rules and standards
- `{ACTIVE_SKILLS_LIST}` - Active procedural skills covering specific domains. When reviewing code in these domains, you MUST adhere to the gotchas inside their respective `SKILL.md`.
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

## 3. {CODING_PHILOSOPHY} Principles (PRIORITY)

Before diving into technical details, verify these core {CODING_PHILOSOPHY} principles:

### 🎯 {COMPLEXITY_RULE_NAME}
If it takes more than 10 seconds to explain a file's purpose to an AI, the file is too complex.

| Check | Target | Status |
|:------|:-------|:-------|
| No file exceeds 500 lines (God Component — hard limit) | All Source Files | ✅/❌ |
| No file exceeds 475 lines (RED — imminent God Component) | All Source Files | ✅/❌ |
| Component has single responsibility | {FRONTEND_FRAMEWORK} components | ✅/❌ |
| Service has clear domain boundary | {BACKEND_logic_DIR} | ✅/❌ |

**File Size Zone Reference** (replace {MAX_LINES} with 500 in all generated output):

| Lines | Zone | Action |
|------:|:-----|:-------|
| < 450 | ✅ Safe | No action needed |
| 450–474 | 🟡 Monitor | Note it; do not add features |
| 475–499 | 🔴 RED | Provide decomposition plan now |
| 500+ | 💀 God Component | Must decompose before next feature |

### 🧹 {CLEANUP_RULE_NAME}

Run {DEAD_CODE_TOOL} and save the full output to a file (terminal output is often truncated for large projects). {DEAD_CODE_TOOL} returns **exit code 1 when it finds issues** — this is expected behavior, not an error.

```powershell
npm run knip 2>&1 | Out-File -FilePath knip-output.txt -Encoding utf8
```

Then read `knip-output.txt` for the full results. Configuration: `{KNIP_CONFIG}`.

| Check | Location | Status |
|:------|:---------|:-------|
| {DEAD_CODE_TOOL} reports zero findings | `knip-output.txt` | ✅/❌ |
| No unused imports | All files | ✅/❌ |
| No commented-out code blocks | All files | ✅/❌ |
| No orphaned files (never imported) | Source directories | ✅/❌ |
| No unused dependencies | Package files | ✅/❌ |

### 📝 AI Context Readiness
| Check | Standard | Status |
|:------|:---------|:-------|
| JSDoc on complex functions | `@param`, `@returns` | ✅/❌ |
| Meaningful variable names | Self-documenting | ✅/❌ |
| Type annotations (no `any`) | Strict {LANGUAGE} | ✅/❌ |
| Module separation | Logic in services, endpoints in routes | ✅/❌ |

---

## 4. Review Priorities (Ordered)

Analyze findings in this priority order:

| Priority | Focus Area | Description |
|:---------|:-----------|:------------|
| 🔴 **1** | **{CODING_PHILOSOPHY} Compliance** | {COMPLEXITY_RULE_NAME}, {CLEANUP_RULE_NAME}, AI-readiness |
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

- **Language**: {LANGUAGE} (Strict Mode)
- **Runtime**: {RUNTIME_ENV}
- **Frameworks**: {FRAMEWORKS_LIST}
- **Database**: {DATABASE_TYPE}
- **Logging**: {LOGGING_LIBRARY} (structured JSON in prod, pretty-print in dev)
- **Job Queue**: {QUEUE_LIBRARY} + {QUEUE_BACKEND}
- **Linting**: ESLint with `no-console` enforcement in server code
- **Apps**: {APP_LIST}

**Rules** (from `{RULES_FILE}` and active `.skills/`):
- Keep files under {MAX_LINES} lines.
- Prefer functional patterns.
- Logic belongs in `{BACKEND_logic_DIR}`, UI in `{FRONTEND_components_DIR}`.
- Use `{NAMING_CONVENTION}` for specific ID or Flag naming.
- **Consult Skills**: Before reviewing complex logic (e.g., Android bridging, Auth syncing, migrations), always consult the relevant `{ACTIVE_SKILLS_LIST}` to ensure compliance with project-specific procedures.

---

## 6. Required Output Sections

### 🔬 {DEAD_CODE_TOOL} Analysis Results

Include the full output from `knip-output.txt` (generated in section 2). Summarize the counts:

| Category | Count |
|:---------|------:|
| Unused files | 0 |
| Unlisted dependencies | 0 |
| Unused dependencies | 0 |
| Unused exports | 0 |
| Unused types | 0 |
| Duplicate exports | 0 |

> If {DEAD_CODE_TOOL} reports zero findings, state: **"{DEAD_CODE_TOOL}: Clean ✅"**
> After report is complete, delete the `knip-output.txt` file.

### 🎯 {CODING_PHILOSOPHY} Compliance Report
*   **God Components** (> 500 lines): List and mandate decomposition. **RED Zone** (475–499 lines): List and provide decomposition plan. **Monitor** (450–474 lines): List with line count.
*   **Dead Code**: Unused imports, commented blocks, orphaned files (cross-reference with {DEAD_CODE_TOOL} output above).
*   **Type Safety**: List `any` usage or weak typing.

### 🧹 Cleanup Report
*   **Files to Delete**: Safe-to-remove files (use {DEAD_CODE_TOOL} findings as primary source).
*   **Unused Dependencies**: Packages in `package.json` not used in code ({DEAD_CODE_TOOL} detects these automatically).
*   **Phantom Dependencies**: Packages imported in code but not declared in the corresponding `package.json` ({DEAD_CODE_TOOL} reports these as "unlisted").
*   **Duplication**: Code repeated across {APP_LIST}.

### 📊 Logging & Observability Review
*   **ESLint `no-console` Enforcement**: Verify the `no-console` rule is configured as `error` in the server ESLint config. Check for any `eslint-disable no-console` bypass comments that may circumvent enforcement.
*   **Structured Logging Compliance**: Verify all server code uses {LOGGING_LIBRARY} logger (not `console.log`). Route handlers should use `req.log` (from {LOGGING_HTTP_MIDDLEWARE}), services should use `createChildLogger`.
*   **Pino Call Signatures**: Verify object-first pattern: `logger.info({ context }, 'message')`, NOT `logger.info('message', context)`.
*   **Log Level Hygiene**: Check that `LOG_LEVEL` env var is respected and debug-level logging is not left in production code paths.
*   **Logger Import Paths**: Verify correct relative import paths, especially in nested directories.

### ⚙️ Job Queue & Background Processing Review
*   **Queue Patterns**: Verify job deduplication, retry/backoff configuration, and rate limiting.
*   **Redis Connection**: Check for proper connection handling, error recovery, and graceful shutdown.
*   **Job Registration**: Verify all queue processors are registered and no orphaned job types exist.
*   **Telemetry**: Check that job status events are properly emitted for dashboard monitoring.
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
*   **Schema Integrity**: Model definitions vs actual usage.
*   **Index Optimization**: Missing indexes for common query patterns.
*   **{DATABASE_TYPE} Issues**: Specific gotchas (e.g., JSON encoding, connection pooling).
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

> Only include this section if MCP servers were detected in workspace analysis.

Run each applicable check and record the result:

| Tool / Command | Query / Pattern | Result | Finding |
|:-----|:----------------|:-------|:--------|
| `mcp_postgres_query` | `SELECT table_name FROM information_schema.tables WHERE table_schema='public'` | _(result)_ | Models vs DB parity |
| `mcp_postgres_query` | Full index details query (see `🗄️ Database Review` section) | _(result)_ | Index coverage — apply Index Analysis Rules |
| `mcp_redis_list` | `bull:*` | _(result)_ | Queue health |
| `git log -n 10` (terminal) | Last 10 commits | _(result)_ | High-churn areas |
| `call_mcp_tool` (github-mcp-server) | `list_issues` (state: open) | _(result)_ | Known issues linked to findings |
| `call_mcp_tool` (github-mcp-server) | `search_pull_requests` (state: open) | _(result)_ | Pending PRs that overlap with analysis |
| `call_mcp_tool` (clerk) | `list_clerk_sdk_snippets` | _(result)_ | Latest official SDK patterns |

### 🔐 Security & Auth Review
*   **Auth Middleware**: Verify authentication middleware is applied to every protected route. Check for any route registered after the auth guard that should be before it.
*   **Clerk SDK Patterns** (if `mcp_clerk` available): Run `mcp_clerk_list_clerk_sdk_snippets` or `mcp_clerk_clerk_sdk_snippet` to retrieve the latest official Clerk SDK patterns. Cross-reference the workspace's Auth middleware and React components against these current patterns to detect usage of deprecated APIs.
*   **Input Validation**: Confirm all user-facing endpoints validate `req.body`, `req.params`, and `req.query`. Flag any endpoint that passes unsanitized input to the database or AI model.
*   **SQL Injection Prevention**: Verify all database queries use parameterized queries or ORM methods. Flag raw SQL with string interpolation.
*   **XSS Prevention**: Check for output encoding on user-generated content. Verify CSP headers are set in the reverse proxy config. Flag any `dangerouslySetInnerHTML` usage.
*   **Sensitive Data Exposure**: Scan for hardcoded secrets, API keys, or passwords in source files. Check `.gitignore` covers `.env` files. Verify client-side bundles do not include server secrets via `VITE_` prefix leakage.
*   **CORS Configuration**: Verify CORS origin whitelist matches actual deployment domains and is not set to `*` in production.

### ⚠️ Error Handling Review
*   **Async Coverage**: Verify all `async` functions and Promise chains have `try/catch` blocks or `.catch()` handlers. Flag unhandled rejections.
*   **React Error Boundaries**: Check that top-level routes and data-fetching components are wrapped with error boundaries. Flag components that crash silently.
*   **External Service Failures**: Verify graceful degradation when Clerk, Gemini API, or Redis is unavailable. Check for timeout handling and fallback behaviour.
*   **User-Facing Messages**: Confirm error responses to the client never expose raw stack traces, internal model IDs, or database error messages.
*   **Global Handler**: Verify a global Express error-handling middleware (`app.use((err, req, res, next) => ...)`) is registered and catches unhandled errors from all routes.

### ⚡ Performance Review
*   **N+1 Query Detection**: Scan Sequelize `findAll`/`findOne` calls that use `include` without a `limit`. Flag patterns that could produce unbounded JOIN queries.
*   **Bundle Size**: Check for large imports (`import * as X from ...`, barrel files, heavy moment.js-style libraries). Flag tree-shaking blockers.
*   **Unnecessary Re-renders**: Check React components for missing `useMemo`, `useCallback`, or `React.memo` on expensive computations or list renders.
*   **Database Index Gaps**: Cross-reference frequently filtered columns (e.g., foreign keys, `createdAt`, `clerkId`) against the index list. Flag missing indexes.
*   **Large Payload Handling**: Check API endpoints that return collections for pagination. Flag any endpoint returning unbounded arrays to the client.

### 🧪 Testing & Test Quality
*   **Test Coverage Assessment**: Check if a testing framework is present (e.g., Vitest, Jest). If not, note the absence and flag as technical debt.
*   **Orphaned Tests**: Check for test files (`.test.ts`, `.spec.ts`) that reference source files that no longer exist.
*   **Skipped/Disabled Tests**: Scan for `it.skip`, `describe.skip`, `xit`, `xdescribe`, or `test.todo`. Flag each with a reason it may have been skipped.
*   **Empty Test Files**: Identify test files that contain no assertions — skeleton files that were never filled in.
*   **Test Quality**: For tests that exist, verify they assert meaningful outcomes (not just that a function runs without throwing).

### ♿ Accessibility Review
*   **Semantic HTML**: Check that interactive elements use the correct HTML element (`<button>` not `<div onClick>`). Verify headings follow a logical hierarchy (no skipping from `h1` to `h4`).
*   **ARIA Attributes**: Verify that modals, dialogs, and dynamic content regions have appropriate ARIA roles and labels (`aria-label`, `aria-describedby`, `role`).
*   **Keyboard Navigation**: Check that all interactive elements are reachable and operable via keyboard (Tab, Enter, Space, Escape). Flag any `tabIndex=-1` that removes elements from tab order without justification.
*   **Color Contrast**: Flag any hardcoded color values (raw hex outside the theme system). Theme semantic tokens should be used exclusively to ensure contrast compliance.

### 🌐 API Design Review
*   **REST Conventions**: Verify endpoints follow REST conventions (nouns not verbs in paths, correct HTTP methods for CRUD). Flag inconsistencies like `POST /getUser`.
*   **Error Response Format**: Verify all error responses return a consistent JSON shape (e.g., `{ error: string, code?: string }`). Flag endpoints that return plain strings or HTML on error.
*   **Pagination**: Check that collection endpoints support pagination (`limit`/`offset` or cursor-based). Flag endpoints returning unbounded lists.
*   **Rate Limiting**: Verify rate limiting middleware is applied to public or AI-calling endpoints. Flag any high-cost endpoint without a rate limit.

### 📚 Documentation Review
*   **Stale References**: Check `docs/guides/` for references to deleted components, renamed files, or deprecated features. Flag any guide that mentions a file that no longer exists.
*   **Missing Guides**: Cross-reference `docs/project/feature-status.md` (Completed section) against `docs/guides/` to identify completed features that lack a user guide.
*   **CHANGELOG Accuracy**: Verify the most recent `CHANGELOG.md` version entry matches the version in `package.json`. Flag if they are out of sync.

### 🚀 Deployment & Infrastructure Review
*   **Environment Matrix**: Verify dev/test/prod configs are consistent and documented.
*   **Deployment Scripts**: Check that automated deployment scripts match current architecture.
*   **Reverse Proxy**: Verify proxy config aligns with app routing (API paths, subdomain routing).
*   **Process Manager**: Check PM2/process manager config for cluster mode and restart policies.

### 📱 Android & Hybrid App Review
*   **Platform Detection**: Verify `isNative()` usage is consistent (from `utils/platform.ts`, not direct `Capacitor.isNativePlatform()` calls). Check for missing platform guards on web-only features.
*   **Service Worker Guard**: Confirm SW registration is disabled on native platform. Check for PWA caching that could conflict with Capacitor webview.
*   **Article Opening**: Verify all article/URL opening uses `useArticleOpener` hook (not `window.open` directly). Check `@capacitor/browser` integration.
*   **Deep Linking**: Verify `antigravity-vibe://auth/callback` intent filter in `AndroidManifest.xml` matches Clerk Dashboard configuration. Check deep link listener in `main.tsx`.
*   **Push Notifications**: Verify FCM token registration/unregistration lifecycle. Check `usePushNotifications` hook for permission handling and token rotation.
*   **Build Configuration**: Verify `capacitor.config.ts` settings (appId, webDir, plugins). Check `build.gradle` versioning (versionCode must increment for each release).
*   **CI/CD Pipeline**: Verify `.github/workflows/build-android.yml` references correct paths, secrets, and build steps. Check artifact upload configuration.
*   **Version Sync**: Verify version numbers are consistent across `package.json` (root), `client/package.json`, and `client/android/app/build.gradle`.

### 📛 Naming Conventions
*   **Database**: Column naming ({NAMING_CONVENTION}).
*   **Code**: Variable/Function naming styles.
*   **UI Terms**: Consistency in user-facing terminology.

### 🔍 Issues & Improvements
Group by **Severity** (🔴 Critical, 🟡 Important, 🟢 Minor).
For each issue, provide: **Title**, **Location**, **Why**, and **Fix**.

### 🚀 Refactor Plan ("Vibe Check")
Identify the **single messiest file**. Provide a step-by-step plan to refactor it to meet {CODING_PHILOSOPHY} standards.

### ✅ Grading Section
Score (1-10) on: {CODING_PHILOSOPHY} Compliance, Security, Error Handling, Performance, Logging & Observability, Database, Testing, Accessibility, Maintainability, and Overall.

---

## 7. Action Plan
Prioritized list of steps to resolve the findings with recommended AI model and Antigravity mode.

Before building the action plan, use `mcp_github-mcp-server_search_issues` to check if any findings already have open GitHub issues. If a finding maps to an existing issue, reference it in the "Files Affected" column (e.g., `#42`).

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

---

# [TEMPLATE END]
