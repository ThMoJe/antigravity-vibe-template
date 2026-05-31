# Role
You are an **Expert Code Quality Engineer and Prompt Architect**. Your goal is to analyze the current VS Code workspace and generate a highly specific, context-aware **Spring Cleaning Prompt** for an AI agent.

# Context
You are currently running inside a VS Code workspace. The user wants a spring cleaning prompt that is tailored *specifically* to this project's architecture, entry points, and file organization.

# Instructions
1.  **Analyze the Workspace**: Silently scan the workspace to identify:
    *   **Project Name & Description**: Check `package.json`, `README.md`, or the root folder name. Use the version from `package.json`.
    *   **Architecture**: Is it a Monorepo? Client/Server? Identify the main directories and applications (e.g., `client`, `server`, `admin-ui`).
    *   **Entry Points**: Locate the main entry files (e.g., `main.tsx`, `index.ts`, `App.tsx`).
    *   **Tech Stack**: Identify frameworks (React, Vue, Express), languages (TS, Python), module system (ESM, CommonJS).
        *   **Version Discovery**: Read **ALL `package.json` files** across all workspaces (root, and every subdirectory with its own `package.json`). For each significant framework and library, extract the **major version number** from `dependencies` or `devDependencies`. Always include the major version when referencing a technology (e.g., "React 19", "Express 5", "Capacitor 8" — never just "React", "Express", "Capacitor").
        *   **Configuration Scanning**: Scan configuration files (e.g., `capacitor.config.ts`, `eslint.config.js`, `vite.config.ts`, CI/CD workflows, Caddyfile, Dockerfiles) to detect tools and infrastructure not visible in `package.json` alone.
        *   **Pattern & Methodology Detection**: Identify architectural patterns in active use (service layer, queue/worker, middleware chains, ORM, migration system, etc.) and coding methodologies from the project rules file (e.g., file size limits, naming conventions, coding philosophies).
    *   **Key Directories**: Identify important source directories (e.g., `components/`, `services/`, `routes/`, `models/`, `queues/`).
    *   **Dead Code Tooling**: Check for a `knip.jsonc` (or `knip.json` / `.knip.jsonc`) config file and a `knip` npm script in `package.json`. If found, Knip is the project's **primary dead code analyzer** — use `{DEAD_CODE_TOOL}` placeholder for the tool name and `{KNIP_CONFIG}` for the config file path. Check the npm script definition for the exact command and flags (e.g., `--reporter markdown`).
    *   **Logging**: Check for structured logging libraries (e.g., Pino, Winston) and ESLint rules enforcing `no-console`.
    *   **Job Queue**: Check for BullMQ, Redis, or other background processing systems. Identify queue config files and worker patterns.
    *   **Linting**: Check for ESLint configs and identify enforced rules that affect spring cleaning scope.
    *   **Scripts**: List standalone scripts in `scripts/` directories — these use `console.log` legitimately and should not be flagged.
    *   **Skills Detection**: Scan the `.skills/` directory (if present). Parse each modular skill (`SKILL.md`). The generated prompt MUST protect all files inside modular skill directories (runbooks, helper scripts, resource files, templates, and implementation examples) from accidental deletion, and NEVER flag them as orphaned files or dead code.
    *   **MCP Servers**: Parse the Antigravity MCP config (typically `~/.gemini/antigravity-ide/mcp_config.json`). If `postgres`, `redis`, `clerk`, `playwright`, or `github-mcp-server` are configured, the generated prompt MUST include live verification steps (using direct terminal commands for Git log/blame checks, `call_mcp_tool` for lazy tools like `github-mcp-server` and `clerk`, and eager wrappers for direct DB/cache checks) to ensure high-confidence cleanup recommendations.

2.  **Historical Report Awareness**: The generated spring cleaning prompt MUST instruct the cleaning agent to read all previous spring cleaning reports in `docs/reports/spring-cleaning/` before starting its analysis. Issues that appear in previous reports as **resolved (strikethrough)**, **deferred to backlog**, **marked as false positive**, or **disregarded/not-an-issue** MUST NOT be re-flagged in the new report. The generated prompt should include a dedicated instruction section for this, placed before the analysis tasks begin.

3.  **Analyze the prompt template** to identify:
    *   **Project-Specific Exceptions**: Dynamic file loading patterns (e.g., `fs.readFile` for prompts), runtime-only dependencies.
    *   **Known Safe Zones**: Translation files, static assets, standalone scripts that should not be flagged.

4.  **Source of Truth**: 
    > [!WARNING]
    > The output prompt must reflect the **current codebase state exclusively**. Do NOT carry forward technology names, version numbers, or architectural descriptions from the existing output file. The existing prompt file provides the template structure and section layout — every factual claim (tech stack, versions, patterns, directory names, file counts) must come from your fresh workspace analysis. If a technology was previously mentioned in the output file but is no longer present in any `package.json` or config file, **omit it**. If a technology has been added since the last run, **include it**.

5.  **Generate the Prompt**: Fill in the **Template** below by replacing all `{PLACEHOLDERS}` with the actual details you found in the workspace.
    *   *Example*: Replace `{PROJECT_NAME}` with "Antigravity-Vibe".
    *   *Example*: Replace `{ENTRY_POINTS}` with "`client/src/main.tsx`, `admin-ui/src/main.tsx`, `server/index.ts`".
    *   *Example*: Replace `{RUNTIME_DEPS}` with "pg, pg-hstore, tsx, typescript".

6.  **Verify Accuracy**: Before saving the output, confirm:
    *   Every technology and tool mentioned exists in the project's `package.json` files or config files.
    *   Every version number matches the major version declared in `package.json`.
    *   No stale references from a previous version of the output prompt were carried forward without verification.
    *   Newly added technologies and dependencies are included.
    *   Removed technologies and dependencies are no longer mentioned.

7.  **Strict Output**: Output **ONLY** the final, filled-in Markdown prompt in `docs/prompts/spring-cleaning-prompt.md`. If file exists, replace it.

---

# [TEMPLATE START]

# Spring Cleaning Prompt — {PROJECT_NAME}

> **Workflow**: Optimized for {CODING_PHILOSOPHY} practices
> **Last Optimized**: {TODAY_DATE}

You are a **{CODING_PHILOSOPHY}** expert and **Code Quality Engineer**. Perform a comprehensive "Spring Cleaning" of the {PROJECT_NAME} workspace ({VERSION}) to eliminate technical debt and ensure AI-readiness.

---

## Project Context

**Architecture**:
{ARCHITECTURE_DETAILS}

**Entry Points**:
{ENTRY_POINTS}

**Key Directories**:
{KEY_DIRECTORIES}

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

### 0. 🔬 {DEAD_CODE_TOOL} — Automated Dead Code Analysis

Run the {DEAD_CODE_TOOL} dead code analyzer **first** and save the full output to a file. Terminal output is often truncated for large projects, so always capture to a file. {DEAD_CODE_TOOL} returns **exit code 1 when it finds issues** — this is expected behavior, not an error.

```powershell
npm run knip 2>&1 | Out-File -FilePath knip-output.txt -Encoding utf8
```

Then read `knip-output.txt` to get the full, untruncated results.

**Configuration**: `{KNIP_CONFIG}` (workspace-aware, all known exceptions pre-configured).

Include the full {DEAD_CODE_TOOL} output (from `knip-output.txt`) in the report. Use the results as the starting point for Tasks 1–3 below — {DEAD_CODE_TOOL} findings are high-confidence and should be reported directly. The manual checks below serve as a **verification layer** for edge cases {DEAD_CODE_TOOL} cannot detect (dynamic imports, runtime-loaded files, etc.).

> **Maintaining `{KNIP_CONFIG}`**: If you discover new legitimate exceptions during analysis (e.g., a new runtime-only dependency or dynamically loaded file), add them to `{KNIP_CONFIG}` rather than documenting them only in this prompt. The goal is zero false positives from `npm run knip`.

### 1. 🗂️ Orphaned Files

**Primary tool**: {DEAD_CODE_TOOL} (reports unused files automatically).

**Manual verification** — check for edge cases {DEAD_CODE_TOOL} may miss:
{EXCEPTION_PATTERNS}

**MCP & Terminal-enhanced confidence** (if available): 
- Run standard read-only local Git terminal commands (`git log -1 --format="%cd" -- <file>` or `git blame <file>`) to check when the suspected orphan was last modified. Files not touched in 90+ days are high-confidence deletion candidates.
- **CRITICAL**: Use `call_mcp_tool` (github-mcp-server) with `search_pull_requests` (state: open) to ensure there are no open PRs currently modifying or relying on this file. Do NOT delete files with open PRs.
- Use `call_mcp_tool` (github-mcp-server) with `search_code` to search the repository for any import or reference to the file before recommending deletion.

### 2. 📦 Unused Dependencies

**Primary tool**: {DEAD_CODE_TOOL} (reports unused and unlisted dependencies automatically).

**Manual verification** — confirm {DEAD_CODE_TOOL} findings and check known exceptions:
{PACKAGE_JSON_LOCATIONS}

**Known Exceptions**:
{RUNTIME_DEPS}

### 3. 📦 Phantom Dependencies

**Primary tool**: {DEAD_CODE_TOOL} (reports unlisted/phantom dependencies automatically).

**Manual verification** — scan for packages **imported in code** but **NOT declared** in the corresponding `package.json`. These rely on transitive resolution via other packages, which is fragile and breaks under strict package managers (pnpm).

### 4. ✂️ Dead Code Blocks

**Primary tool**: {DEAD_CODE_TOOL} (reports unused exports, unused types, and duplicate exports).

**Manual verification** — check for items {DEAD_CODE_TOOL} does not cover:
- Unused functions/variables that are **not exported** (internal dead code)
- Commented-out code blocks (legacy cruft)
- Unreachable code paths

### 5. 📏 {CODING_PHILOSOPHY} Compliance Check

- Files by zone (use 500 as the hard limit in generated output):
  - 500+ lines → 💀 **God Component** — must decompose before next feature
  - 475–499 lines → 🔴 **RED** — provide decomposition plan now
  - 450–474 lines → 🟡 **Monitor** — note it, do not add features
  - < 450 lines → ✅ **Safe**
- `any` type usage in non-script code
- Missing JSDoc on complex functions

### 6. 📦 NPM Package Updates

Use `npm-check-updates` (ncu) to identify outdated packages across all workspaces:
{PACKAGE_JSON_LOCATIONS}

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

- **ESLint `no-console` rule**: Confirm it is configured as `error` in the server ESLint config
- **`eslint-disable` bypasses**: Scan for `eslint-disable no-console` or `eslint-disable-next-line no-console` comments in server code — these circumvent enforcement and should be flagged
- **Logger import consistency**: Check that all server files import from the correct logger module with correct relative paths (especially in nested directories like `managers/`, `extractors/`)
- **Client/Admin `console.log`**: Check for stray `console.log` in frontend code (not ESLint-enforced but still undesirable in production)

**Exceptions**:
- Files in `{SCRIPTS_DIR}` — standalone CLI tools that correctly use `console.log` for terminal output

### 8. ⚙️ Queue & Background Processing Hygiene

- **Orphaned job processors**: Check for queue processor registrations that reference job types no longer enqueued
- **Redis configuration**: Verify `{QUEUE_BACKEND}` connection config is consistent across environments
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
- Standalone scripts in `{SCRIPTS_DIR}` (legitimate terminal output)

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
- Are not required by any active skills listed in `{ACTIVE_SKILLS_LIST}`

**MCP enhancement** (if `mcp_git` available): For any file flagged above, run `mcp_git_git_blame` to determine the last meaningful edit. Add "Last touched" context to the output table.

### 10. 🗄️ Database Hygiene

**Primary tool** (if `mcp_postgres` available):

Run the following query and compare results against Sequelize model definitions in `server/models/`:
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

- **`.env.example` completeness**: Compare `.env.example` files (root, `client/`, `server/`, `admin-ui/`) against all `process.env.X` and `import.meta.env.X` references in source code. Flag any env var used in code but missing from `.env.example`.
- **Stale `.env.example` entries**: Flag any variable defined in `.env.example` that is no longer referenced in any source file.
- **Consistency across workspaces**: Verify that variables shared across workspaces (e.g., `DATABASE_URL`, `CLERK_SECRET_KEY`) are consistently documented in all relevant `.env.example` files.

### 12. 📚 Documentation Staleness

Verify project documentation accurately reflects the current codebase:

- **Stale guides**: Cross-reference all files in `docs/guides/` against the current project structure. Flag any guide that references a component, file, or route that no longer exists.
- **Renamed references**: Search for old package or project names (e.g., previous project name if renamed) that may still appear in docs.
- **CHANGELOG vs git tags**: Use `mcp_git_git_tag` (if available) to list all release tags. Check that each tag has a corresponding entry in `CHANGELOG.md`. Flag gaps.
- **Feature status accuracy**: Compare `docs/project/feature-status.md` against `docs/guides/` — every item marked "Completed" should have a corresponding guide.

### 13. 🧪 Test Hygiene

- **Orphaned test files**: Find test files (`.test.ts`, `.spec.ts`, `.test.tsx`) whose corresponding source file no longer exists. These are safe-to-delete candidates.
- **Skipped/disabled tests**: Scan for `it.skip`, `describe.skip`, `xit`, `xdescribe`, `test.todo`. Report each with the file and line number.
- **Empty test files**: Identify test files that contain no `it()`, `test()`, or `expect()` calls — skeleton files with no assertions.
- **Test coverage absence**: If no testing framework is detected in any `package.json`, flag the entire test infrastructure as missing technical debt.

### 14. 🔒 Security Audit

- **Dependency vulnerabilities**: Run `npm audit` across all workspaces (`root`, `client/`, `server/`, `admin-ui/`). Report all findings with severity level.
- **Hardcoded secrets**: Scan source files for patterns matching API keys, passwords, connection strings, or private keys hardcoded as string literals. Flag any match.
- **`.gitignore` coverage**: Verify `.gitignore` covers sensitive files: `.env`, `*.env`, `google-services.json`, `*.keystore`, `*.jks`, `*.p12`. Flag any sensitive file pattern that is missing.
- **CI/CD secrets safety**: Scan `.github/workflows/` files for hardcoded secrets. All credentials should use `${{ secrets.X }}` syntax, never hardcoded values.

### 15. 🗃️ Git Hygiene

- **Large tracked files**: Identify any file tracked in git that exceeds 1 MB (binaries, database dumps, images). These should be in `.gitignore` or moved to Git LFS.
- **`.gitignore` gaps**: Check for build artifacts (`dist/`, `build/`, `.next/`), IDE configs (`.idea/`, `.vscode/` if not intentionally shared), and OS files (`Thumbs.db`, `.DS_Store`) that should be ignored.
- **Untracked generated files**: Check for generated files that appear in the working tree but are missing from `.gitignore` (e.g., `knip-output.txt`, `*.log`, coverage reports).

### 16. 📱 Android & Hybrid App Hygiene

Verify Android/Capacitor integration health:

- **Direct `Capacitor.isNativePlatform()` calls**: Should use centralized `isNative()` from `utils/platform.ts` instead
- **Direct `window.open` calls**: Should use `useArticleOpener` hook for cross-platform URL opening
- **Stale `google-services.json`**: Verify Firebase config matches current project
- **AndroidManifest.xml drift**: Check for obsolete permissions, outdated intent filters, or missing declarations
- **Build version mismatch**: Verify `versionName` in `build.gradle` matches `version` in `package.json` (root + client)
- **Orphaned Capacitor plugins**: Check `capacitor.config.ts` and `package.json` for plugins installed but not used
- **CI/CD secrets alignment**: Verify `.github/workflows/build-android.yml` references match configured GitHub Secrets


---

## Output Format

### 🔬 {DEAD_CODE_TOOL} Analysis Results

Include the full output from `knip-output.txt` (generated in Task 0). Summarize the counts:

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

### 📏 {CODING_PHILOSOPHY} Violations

| File | Lines | Issue |
|:-----|:------|:------|
| `path` | 550 | Exceeds {MAX_LINES} lines |

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
4. **Run type-check after** — Verify no breakage with `{TYPE_CHECK_COMMAND}`

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

# [TEMPLATE END]
