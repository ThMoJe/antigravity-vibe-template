# Spring Cleaning Prompt — Antigravity-Vibe

> **Workflow**: Optimized for Vibe Coding practices
> **Last Optimized**: 2026-05-29

You are a **Vibe Coding** expert and **Code Quality Engineer**. Perform a comprehensive "Spring Cleaning" of the Antigravity-Vibe workspace (2.8.19) to eliminate technical debt and ensure AI-readiness.

---

## Project Context

**Architecture**:
*   **Client** (`client/`): React 19 SPA via Vite 7, MUI 7, Framer Motion 12, Clerk React 6, TanStack React Query 5, i18next 26, TipTap 3, Lucide Icons 1. PWA with Capacitor 8 hybrid Android app.
*   **Admin UI** (`admin-ui/`): React 19 SPA via Vite 7, MUI 7 (with MUI X Date Pickers 8), Clerk React 6, TanStack React Query 5, React Router DOM 7.
*   **Server** (`server/`): Node.js 22+ (ESM via `tsx`), Express 5, Sequelize 6 ORM, Pino 10 structured logging, BullMQ 5 + Redis (ioredis 5), Clerk Express 2 / Clerk Backend 3.
*   **WWW** (`www/`): Astro 6 marketing site.
*   **E2E Tests** (`e2e/`): Playwright 1 end-to-end test suite.
*   **Shared Types** (`packages/types/`): `@project-name/types` workspace package.

**Entry Points**:
*   `client/src/main.tsx` — Client app bootstrap
*   `client/src/App.tsx` — Client app root component
*   `admin-ui/src/main.tsx` — Admin UI bootstrap
*   `server/index.ts` — Express server entry
*   `server/cron.ts` — Scheduled cron jobs
*   `server/workers/extractionWorker.ts` — Content extraction worker
*   `server/workers/graphApiPollingWorker.ts` — Microsoft Graph API polling worker
*   `server/workers/newsletterLinkWorker.ts` — Newsletter link processing worker
*   `www/src/pages/` — Astro page entry points
*   `e2e/tests/` — Playwright test files

**Key Directories**:
*   `client/src/components/` — React UI components
*   `client/src/hooks/` — Custom React hooks
*   `client/src/context/` — React context providers
*   `client/src/theme/` — Theme extensions
*   `client/src/utils/` — Utility functions
*   `client/src/i18n/` — Internationalization resources
*   `server/services/` — Business logic services (28 files + subdirs)
*   `server/routes/` — Express route handlers (18 files + subdirs)
*   `server/models/` — Sequelize model definitions (14 models + index.ts)
*   `server/middleware/` — Express middleware (auth, validation, feedOwnership, orgFeed, admin)
*   `server/workers/` — BullMQ worker processors (3 workers)
*   `server/migrations/` — Database migration scripts
*   `server/scripts/` — Standalone CLI debug/utility scripts (23 files)
*   `server/prompts/` — AI prompt templates
*   `server/templates/` — Email/HTML templates
*   `server/utils/` — Server utilities
*   `docs/guides/` — User guides (37 files)
*   `docs/reports/` — Generated review reports
*   `.skills/` — Modular AI operational skills (8 skills)

---

## Historical Report Awareness

Before starting your analysis, read **all previous spring cleaning reports** in `docs/reports/spring-cleaning/`. For each issue found in prior reports, check its current status:

- **Resolved (strikethrough `~~...~~`)**: The issue was fixed. Do NOT re-flag it.
- **Deferred to backlog**: The issue was acknowledged and intentionally deferred. Reference the backlog item but do NOT re-flag it as a new finding.
- **False positive / Not an issue**: The finding was investigated and determined to be incorrect or normal behavior. Do NOT re-flag it.
- **Disregarded**: The team reviewed and chose not to act. Do NOT re-flag it.

Only flag issues that are **genuinely new** or represent **regressions** of previously fixed items. If a prior report marked something as resolved but it has regressed, flag it as a **regression** with a reference to the prior report.

---

## Analysis Tasks

### 0. 🔬 Knip — Automated Dead Code Analysis

Run the Knip dead code analyzer **first** and save the full output to a file. Terminal output is often truncated for large projects, so always capture to a file. Knip returns **exit code 1 when it finds issues** — this is expected behavior, not an error.

```powershell
npm run knip 2>&1 | Out-File -FilePath knip-output.txt -Encoding utf8
```

Then read `knip-output.txt` to get the full, untruncated results.

**Configuration**: `knip.jsonc` (workspace-aware, all known exceptions pre-configured).

Include the full Knip output (from `knip-output.txt`) in the report. Use the results as the starting point for Tasks 1–3 below — Knip findings are high-confidence and should be reported directly. The manual checks below serve as a **verification layer** for edge cases Knip cannot detect (dynamic imports, runtime-loaded files, etc.).

> **Maintaining `knip.jsonc`**: If you discover new legitimate exceptions during analysis (e.g., a new runtime-only dependency or dynamically loaded file), add them to `knip.jsonc` rather than documenting them only in this prompt. The goal is zero false positives from `npm run knip`.

### 1. 🗂️ Orphaned Files

**Primary tool**: Knip (reports unused files automatically).

**Manual verification** — check for edge cases Knip may miss:
*   Files loaded dynamically via `fs.readFile` or `fs.readFileSync` (e.g., `server/prompts/`, `server/templates/`, `server/data/`)
*   Translation JSON files loaded by i18next at runtime (`client/src/i18n/`)
*   Static assets in `client/public/`, `admin-ui/public/`, `www/public/`
*   Capacitor-generated files in `client/android/`
*   Migration scripts in `server/migrations/` (executed manually, not imported)
*   Files inside `.skills/` directories (runbooks, scripts, resources — never flag as orphaned)

**MCP & Terminal-enhanced confidence** (if available): 
- Run standard read-only local Git terminal commands (`git log -1 --format="%cd" -- <file>` or `git blame <file>`) to check when the suspected orphan was last modified. Files not touched in 90+ days are high-confidence deletion candidates.
- **CRITICAL**: Use `call_mcp_tool` (github-mcp-server) with `search_pull_requests` (state: open) to ensure there are no open PRs currently modifying or relying on this file. Do NOT delete files with open PRs.
- Use `call_mcp_tool` (github-mcp-server) with `search_code` to search the repository for any import or reference to the file before recommending deletion.

### 2. 📦 Unused Dependencies

**Primary tool**: Knip (reports unused and unlisted dependencies automatically).

**Manual verification** — confirm Knip findings and check known exceptions:
*   Root `package.json` — monorepo orchestration deps (`concurrently`, `knip`, `@capacitor/assets`, `@playwright/test`, `@clerk/testing`, `@azure/msal-node`)
*   `client/package.json` — React 19 + Vite 7 + MUI 7 + Capacitor 8 + Clerk React 6
*   `admin-ui/package.json` — React 19 + Vite 7 + MUI 7 + Clerk React 6 + React Router DOM 7
*   `server/package.json` — Express 5 + Sequelize 6 + BullMQ 5 + Pino 10
*   `www/package.json` — Astro 6
*   `packages/types/package.json` — Shared TypeScript types

**Known Exceptions**:
*   `pg-hstore` — Runtime-only hstore serializer loaded by Sequelize (no direct import)
*   `pino-pretty` — Dev-only log transport, loaded via CLI flag (no direct import)
*   `@capacitor/status-bar` — Native runtime bridge, no direct import needed
*   `@capacitor/assets` — CLI tool for icon generation
*   `cross-env` — Used in npm scripts, not imported in code

### 3. 📦 Phantom Dependencies

**Primary tool**: Knip (reports unlisted/phantom dependencies automatically).

**Manual verification** — scan for packages **imported in code** but **NOT declared** in the corresponding `package.json`. These rely on transitive resolution via other packages, which is fragile and breaks under strict package managers (pnpm).

### 4. ✂️ Dead Code Blocks

**Primary tool**: Knip (reports unused exports, unused types, and duplicate exports).

**Manual verification** — check for items Knip does not cover:
- Unused functions/variables that are **not exported** (internal dead code)
- Commented-out code blocks (legacy cruft)
- Unreachable code paths

### 5. 📏 Vibe Coding Compliance Check

- Files by zone (use 500 as the hard limit in generated output):
  - 500+ lines → 💀 **God Component** — must decompose before next feature
  - 475–499 lines → 🔴 **RED** — provide decomposition plan now
  - 450–474 lines → 🟡 **Monitor** — note it, do not add features
  - < 450 lines → ✅ **Safe**
- `any` type usage in non-script code
- Missing JSDoc on complex functions

### 6. 📦 NPM Package Updates

Use `npm-check-updates` (ncu) to identify outdated packages across all workspaces:
*   Root `package.json`
*   `client/package.json`
*   `admin-ui/package.json`
*   `server/package.json`
*   `www/package.json`
*   `packages/types/package.json`

For each package, analyze:
- **Current Version**: Version specified in package.json
- **Latest Version**: Available on npm registry
- **Update Risk**: Categorize as "Critical update", "Safe to update", or "No need to update"

**Risk Assessment Criteria**:
- **Critical update**: Security vulnerabilities, major bug fixes, or deprecated current version
- **Safe to update**: Patch/minor updates with no breaking changes indicated
- **No need to update**: Already on latest or update provides no meaningful benefit

### 7. 📊 Logging Compliance

Verify structured logging standards are maintained:

- **ESLint `no-console` rule**: Confirm it is configured as `error` in `server/eslint.config.js`
- **`eslint-disable` bypasses**: Scan for `eslint-disable no-console` or `eslint-disable-next-line no-console` comments in server code — these circumvent enforcement and should be flagged
- **Logger import consistency**: Check that all server files import from `./logger.js` (or correct relative path) and use `createChildLogger('ServiceName')` pattern. Especially check nested directories like `services/managers/`, `services/extraction/`, `services/newsletter/`.
- **Client/Admin `console.log`**: Check for stray `console.log` in frontend code (not ESLint-enforced but still undesirable in production)

**Exceptions**:
- Files in `server/scripts/` — standalone CLI tools that correctly use `console.log` for terminal output

### 8. ⚙️ Queue & Background Processing Hygiene

- **Orphaned job processors**: Check for queue processor registrations in `server/services/QueueService.ts` that reference job types no longer enqueued. Cross-reference with workers in `server/workers/` (extractionWorker, graphApiPollingWorker, newsletterLinkWorker).
- **Redis configuration**: Verify ioredis 5 connection config is consistent across environments
- **Stale queue config**: Check for hardcoded values that should be environment variables
- **Live queue state** (if `mcp_redis` available): Run `mcp_redis_list` with pattern `bull:*` to check actual queue depth and identify stalled/failed jobs. Cross-reference with registered worker types in `server/workers/` to detect orphaned queues.

### 9. 🧹 Deep Clean Analysis

Perform an intensive scan for technical debt and garbage code:

#### Debug Noise
Find all instances of:
- `eslint-disable no-console` or `eslint-disable-next-line no-console` bypass comments in server code
- `console.log`, `console.dir`, `console.warn`, `console.error` in client/admin code (non-production logging)
- `debugger` statements
- Large blocks of commented-out code that look like old logic

**Exceptions**:
- Standalone scripts in `server/scripts/` (legitimate terminal output)

#### Obsolete Artifacts
Identify folders or files with names suggesting obsolescence:
- `temp`, `tmp`, `backup`, `bak`, `old`, `archive`, `v1`, `v2`, `test_junk`, `deprecated`
- Files with patterns like `*.bak`, `*.old`, `*_backup.*`, `*_old.*`

#### Orphaned Source Files
Analyze the file structure to find source files (`.ts`, `.tsx`, `.js`, `.jsx`) that:
- Are not imported by any other file
- Are not listed as entry points in `package.json`
- Are not config files or standalone scripts
- Are not migration files
- Are not route aggregator files (e.g., `admin.ts` importing from `admin/`)
- Are not required by any active skills listed in `.skills/admin-ui-patterns`, `.skills/capacitor-ops`, `.skills/clerk-sync-tracer`, `.skills/cyberpunk-ui-crafter`, `.skills/deployment-ops`, `.skills/feed-scraper`, `.skills/project-migrations`, `.skills/security-isolation`

**MCP enhancement** (if available): For any file flagged above, run `git blame` terminal command to determine the last meaningful edit. Add "Last touched" context to the output table.

### 10. 🗄️ Database Hygiene

**Primary tool** (if `mcp_postgres` available):

Run the following query and compare results against Sequelize model definitions in `server/models/` (14 models: Article, ArticleArchive, ArticleEmailContent, Feed, FeedArchive, NewsletterEmail, Organization, OrganizationMembership, SubscriptionPlan, SupportMessage, SystemSetting, User, UserActivity, VaultItem):
```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY table_name;
```

- **Orphaned tables**: Tables present in the database but with no corresponding Sequelize model — flag for review
- **Orphaned columns**: Columns present in the DB that no model defines (requires per-table inspection)
- **Index efficiency**: Run `SELECT indexname, tablename FROM pg_indexes WHERE schemaname='public' ORDER BY tablename;` and check for missing indexes on foreign keys and commonly filtered columns

### 11. ⚙️ Configuration Drift

Verify the project's environment variable documentation is accurate and complete:

- **`.env.example` completeness**: Compare `.env.example` files (`client/.env.example`, `server/.env.example`, `admin-ui/.env.example`) against all `process.env.X` and `import.meta.env.X` references in source code. Flag any env var used in code but missing from `.env.example`.
- **Stale `.env.example` entries**: Flag any variable defined in `.env.example` that is no longer referenced in any source file.
- **Consistency across workspaces**: Verify that variables shared across workspaces (e.g., `DATABASE_URL`, `CLERK_SECRET_KEY`) are consistently documented in all relevant `.env.example` files.

### 12. 📚 Documentation Staleness

Verify project documentation accurately reflects the current codebase:

- **Stale guides**: Cross-reference all files in `docs/guides/` (37 guides) against the current project structure. Flag any guide that references a component, file, or route that no longer exists.
- **Renamed references**: Search for old package or project names that may still appear in docs.
- **CHANGELOG vs version**: Verify the most recent `CHANGELOG.md` version entry matches the version in `package.json` (2.8.19). Flag gaps.
- **Feature status accuracy**: Compare `docs/project/feature-status.md` against `docs/guides/` — every item marked "Completed" should have a corresponding guide.

### 13. 🧪 Test Hygiene

- **Orphaned test files**: Find test files (`.test.ts`, `.spec.ts`, `.test.tsx`) whose corresponding source file no longer exists. These are safe-to-delete candidates.
- **Skipped/disabled tests**: Scan for `it.skip`, `describe.skip`, `xit`, `xdescribe`, `test.todo`. Report each with the file and line number.
- **Empty test files**: Identify test files that contain no `it()`, `test()`, or `expect()` calls — skeleton files with no assertions.
- **Test coverage absence**: The server has `"test": "echo \"Error: no test specified\" && exit 1"` — unit testing framework is not configured. E2E tests exist via Playwright 1 in `e2e/`. Flag unit test infrastructure as missing technical debt.

### 14. 🔒 Security Audit

- **Dependency vulnerabilities**: Run `npm audit` across all workspaces (`root`, `client/`, `server/`, `admin-ui/`, `www/`). Report all findings with severity level.
- **Hardcoded secrets**: Scan source files for patterns matching API keys, passwords, connection strings, or private keys hardcoded as string literals. Flag any match.
- **`.gitignore` coverage**: Verify `.gitignore` covers sensitive files: `.env`, `*.env`, `google-services.json`, `*.keystore`, `*.jks`, `*.p12`. Flag any sensitive file pattern that is missing.
- **CI/CD secrets safety**: Scan `.github/workflows/` files (`build-android.yml`, `cleanup.yml`, `deploy-and-test.yml`) for hardcoded secrets. All credentials should use `${{ secrets.X }}` syntax, never hardcoded values.

### 15. 🗃️ Git Hygiene

- **Large tracked files**: Identify any file tracked in git that exceeds 1 MB (binaries, database dumps, images). These should be in `.gitignore` or moved to Git LFS.
- **`.gitignore` gaps**: Check for build artifacts (`dist/`, `build/`), IDE configs (`.idea/`), and OS files (`Thumbs.db`, `.DS_Store`) that should be ignored.
- **Untracked generated files**: Check for generated files that appear in the working tree but are missing from `.gitignore` (e.g., `knip-output.txt`, `*.log`, coverage reports, `test-results/`).

### 16. 📱 Android & Hybrid App Hygiene

Verify Android/Capacitor integration health:

- **Direct `Capacitor.isNativePlatform()` calls**: Should use centralized `isNative()` from `client/src/utils/platform.ts` instead
- **Direct `window.open` calls**: Should use `useArticleOpener` hook for cross-platform URL opening
- **Stale `google-services.json`**: Verify Firebase config matches current project
- **AndroidManifest.xml drift**: Check for obsolete permissions, outdated intent filters, or missing declarations
- **Build version mismatch**: Verify `versionName` in `client/android/app/build.gradle` matches `version` in `package.json` (root: 2.8.19 + client: 2.8.19). Verify `versionCode` follows scheme: `major × 10000 + minor × 100 + patch`.
- **Orphaned Capacitor plugins**: Check `client/capacitor.config.ts` and `client/package.json` for plugins installed but not used
- **CI/CD secrets alignment**: Verify `.github/workflows/build-android.yml` references match configured GitHub Secrets
- **`removeAllListeners()` banned in `useShareTarget`**: Verify `@capgo/capacitor-share-target` uses `listenerHandle.remove()` only


---

## Output Format

### 🔬 Knip Analysis Results

Include the full output from `knip-output.txt` (generated in Task 0). Summarize the counts:

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

### 🔴 High Confidence — Delete

| Category | Path | Reason |
|:---------|:-----|:-------|
| Orphaned File | `path/to/file` | Never imported |
| Unused Dep | `package-name` | Not imported |

### 🟡 Low Confidence — Verify

| Category | Path | Notes |
|:---------|:-----|:------|
| Dynamic Usage? | `path/to/file` | Check for runtime loading |
| Plugin? | `package-name` | May be used at runtime |

### ✂️ Internal Dead Code

| File | Line(s) | Item | Action |
|:-----|:--------|:-----|:-------|
| `path` | 100-120 | `unusedFunction()` | Delete |

### 📏 Vibe Coding Violations

| File | Lines | Issue |
|:-----|:------|:------|
| `path` | 550 | Exceeds 500 lines |

### 📊 Logging Compliance Findings

| File | Line(s) | Issue | Action |
|:-----|:--------|:------|:-------|
| `path` | 42 | `eslint-disable no-console` bypass | Remove bypass, replace with logger |
| `path` | 100 | `console.log` in client code | Replace with proper logging or remove |

### 📦 NPM Package Updates

| Package | Current Version | Latest Version | Recommendation | Notes |
|:--------|:----------------|:---------------|:---------------|:------|
| `package-name` | 1.2.3 | 2.0.0 | 🔴 Critical update | Security vulnerability patched |
| `package-name` | 3.1.0 | 3.1.5 | 🟢 Safe to update | Patch release, no breaking changes |
| `package-name` | 5.0.0 | 5.0.0 | ⚪ No need to update | Already on latest |

**Recommendation Legend**:
- 🔴 **Critical update**: Security fixes, major bugs, or deprecated versions
- 🟢 **Safe to update**: Minor/patch updates with no breaking changes
- ⚪ **No need to update**: Already current or update provides no benefit

### 🧹 Deep Clean Findings

#### Debug Noise

| File | Line(s) | Type | Code Snippet |
|:-----|:--------|:-----|:-------------|
| `path/to/file` | 42 | eslint-disable bypass | `// eslint-disable-next-line no-console` |
| `path/to/file` | 100 | debugger | `debugger;` |
| `path/to/file` | 50-75 | Commented Code | Large legacy block |

#### Obsolete Artifacts

| Type | Path | Reason |
|:-----|:-----|:-------|
| Directory | `path/to/temp/` | Name suggests temporary |
| File | `path/to/file.bak` | Backup file extension |

#### Orphaned Source Files

| File | Last Modified | Notes |
|:-----|:--------------|:------|
| `path/to/orphan.ts` | 2025-01-01 | Never imported, not an entry point |

### ⚙️ Configuration Drift Findings

| Type | File | Variable | Issue |
|:-----|:-----|:---------|:------|
| Missing from `.env.example` | `server/.env.example` | `NEW_VAR` | Used in code but undocumented |
| Stale in `.env.example` | `client/.env.example` | `OLD_VAR` | No longer referenced in code |

### 📚 Documentation Staleness Findings

| Guide | Stale Reference | Issue |
|:------|:----------------|:------|
| `docs/guides/example.md` | `OldComponent.tsx` | File deleted |
| `CHANGELOG.md` | — | Tag `v2.x.x` has no corresponding entry |

### 🧪 Test Hygiene Findings

| Type | File | Notes |
|:-----|:-----|:------|
| Orphaned test | `path/to/file.test.ts` | Source file deleted |
| Skipped test | `path/to/other.test.ts:42` | `it.skip(...)` with no explanation |
| Empty test file | `path/to/empty.test.ts` | No assertions found |

### 🔒 Security Audit Findings

| Category | Severity | Detail |
|:---------|:---------|:-------|
| npm audit | 🔴 Critical | `package-name` — vulnerability description |
| Hardcoded secret | 🔴 Critical | `path/to/file:42` — API key literal |
| .gitignore gap | 🟡 Medium | `google-services.json` not in `.gitignore` |
| CI/CD secret | 🔴 Critical | Hardcoded value in `.github/workflows/build.yml:10` |

### 🗃️ Git Hygiene Findings

| Type | Path | Size / Detail |
|:-----|:-----|:--------------|
| Large tracked file | `path/to/dump.sql` | 12 MB — should be in `.gitignore` |
| .gitignore gap | `dist/` | Build artifact tracked in git |
| Untracked generated file | `knip-output.txt` | Should be in `.gitignore` |

### 📱 Android & Hybrid App Findings

| Check | Status | Notes |
|:------|:-------|:------|
| `isNative()` centralized usage | ✅/❌ | Direct `Capacitor.isNativePlatform()` calls found? |
| `useArticleOpener` for URL opening | ✅/❌ | Direct `window.open` calls found? |
| Version sync (package.json ↔ build.gradle) | ✅/❌ | Mismatch details |
| `google-services.json` matches Firebase project | ✅/❌ | Verification result |
| AndroidManifest.xml current | ✅/❌ | Obsolete entries? |
| Orphaned Capacitor plugins | ✅/❌ | Unused plugins? |
| CI/CD secrets alignment | ✅/❌ | Missing or unused secrets? |

### 🔌 MCP Live Verification Results

> Only include this section if MCP servers were detected in workspace analysis.

| Tool / Command | Query / Pattern | Result | Action |
|:-----|:----------------|:-------|:-------|
| `mcp_postgres_query` | Tables in `public` schema vs model files | _(result)_ | Orphaned tables? |
| `mcp_postgres_query` | Index coverage on FK columns | _(result)_ | Missing indexes? |
| `mcp_redis_list` | `bull:*` | _(result)_ | Stale/failed jobs? |
| `git log -1 --format="%cd" -- <file>` (terminal) | Files not touched in 90+ days | _(result)_ | Safe to delete? |
| `call_mcp_tool` (github-mcp-server) | `search_code` imports of flagged orphans | _(result)_ | References found? |

---

## Execution Rules

1. **Report first, then confirm** — Do not delete anything without user approval
2. **Archive over delete** — Move questionable files to a temporary holding area if uncertain
3. **Update documentation** — If removing features, update relevant docs
4. **Run type-check after** — Verify no breakage with `npx tsc --noEmit` (in server/ and client/)

---

## Recommended Actions with AI Model Selection

For each cleanup action identified, recommend an appropriate AI model and Antigravity mode:

| Priority | Action | Effort | Files Affected | Recommended Model | Mode |
|:---------|:-------|:------:|:---------------|:------------------|:----:|
| 🔴 High | *cleanup action* | Low/Med/High | *file list* | *model name* | Fast/Planning |
| 🟡 Medium | *cleanup action* | Low/Med/High | *file list* | *model name* | Fast/Planning |
| 🟢 Low | *cleanup action* | Low/Med/High | *file list* | *model name* | Fast/Planning |

### Model Selection Rationale

For the model recommendations in the Recommended Actions table, use the **AI Model Orchestration** section from the agent's global rules (GEMINI.md). This includes the full model roster, recommended Antigravity modes (Fast/Planning), and the 5-tier escalation path.

Match actions to models based on complexity:
- Bulk deletions and simple cleanups → fastest available model (Fast mode)
- Precision deletion and type safety work → mid-tier reasoning model (Planning mode)
- Complex refactoring requiring architectural decisions → deep reasoning model (Planning mode)

---

**Begin the Spring Cleaning analysis now.**