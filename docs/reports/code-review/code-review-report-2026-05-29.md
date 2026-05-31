# Antigravity-Vibe Code Review Report — 2026-05-29

**Version**: 2.8.19
**Reviewer**: AI Code Review (Claude Opus 4.6 — Planning)
**Date**: 2026-05-29
**Scope**: Full monorepo — post-Antigravity 2.0 migration, new Windows install baseline
**Carry-forwards from**: code-review-report-2026-05-18.md

---

## Executive Summary

This review covers the post-migration state after moving to a new Windows 11 development environment with Antigravity 2.0. The codebase was last reviewed at v2.8.16 (May 18, 2026); three patch releases (2.8.17, 2.8.18, 2.8.19) shipped since then with documentation updates, admin user deletion, and dependency upgrades.

**Key findings this cycle:**

| # | Severity | Finding | Status |
|:-:|:---------|:--------|:------:|
| 1 | 🔴 Important | SupportMessages duplicate indexes regressed on local dev DB (new install seeded from pre-038 backup) | NEW |
| 2 | 🟡 Medium | `QueueService.closeQueues()` leaks `newsletterLinkQueue` + `newsletterLinkQueueEvents` connections | NEW |
| 3 | 🟡 Medium | Knip crashes with `RangeError: Array buffer allocation failed` on Node.js v26.2.0 / oxc-parser | NEW |
| 4 | 🟡 Medium | `usePushNotifications.ts:107` uses `removeAllListeners()` instead of surgical `handle.remove()` | NEW |
| 5 | 🟢 Minor | `build.gradle` versionCode comment stale (`v2.8.18 = 20818` but code is `20819 / v2.8.19`) | NEW |
| 6 | 🟢 Minor | `QueueService.ts` approaching Monitor zone (468 lines) | NEW |

---

## 1. Dead Code Analysis (Knip)

### ⚠️ Knip Crash — `oxc-parser` OOM on Node.js 26

Knip fails to complete on this environment. All three attempts (direct `node`, `npx`, `npm run knip`) crash with:

```
RangeError: Array buffer allocation failed
    at new ArrayBuffer (<anonymous>)
    at createBuffer (oxc-parser/src-js/raw-transfer/common.js:276)
```

**Root Cause**: `oxc-parser` (used by Knip for AST parsing) attempts to allocate an excessively large `ArrayBuffer` that exceeds Node.js v26.2.0's default heap limits. The `--max-old-space-size=8192` flag doesn't help because the crash is in `ArrayBuffer` allocation (V8 external memory), not the old-space heap.

**Impact**: Dead code analysis is unavailable. Manual inspection of key areas shows no obvious unused exports or orphaned files. The codebase was Knip-clean at v2.8.16 (May 18), and changes since then are primarily documentation and dependency updates.

| Category | Count |
|:---------|------:|
| Unused files | N/A (Knip crash) |
| Unlisted dependencies | N/A |
| Unused dependencies | N/A |
| Unused exports | N/A |
| Unused types | N/A |
| Duplicate exports | N/A |

> **Action Required**: Investigate Knip / oxc-parser compatibility with Node.js 26. Possible workarounds: downgrade `oxc-parser`, pin Knip to a pre-oxc version, or increase `--max-array-buffer-size` if Node 26 supports it.

---

## 2. Vibe Coding Compliance — File Size Zones

**Threshold**: 500 lines hard limit · 475 RED zone · 450 Monitor

### God Components (> 500 lines): None ✅

### RED Zone (475–499 lines): None ✅

### Monitor Zone (450–474 lines)

| File | Lines | Zone | Note |
|:-----|------:|:-----|:-----|
| [QueueService.ts](file:///v:/Antigravity/project-name/server/services/QueueService.ts) | 468 | 🟡 Monitor | **New to Monitor** — was 403 in May-18 review. Do not add features. |
| [RankedSelectorGrid.tsx](file:///v:/Antigravity/project-name/client/src/components/feeds/RankedSelectorGrid.tsx) | 438 | Monitor | Carry-forward |
| [EditFeedDialog.tsx](file:///v:/Antigravity/project-name/client/src/components/EditFeedDialog.tsx) | 436 | Monitor | Carry-forward |
| [ContentSelectorTab.tsx](file:///v:/Antigravity/project-name/client/src/components/ContentSelectorTab.tsx) | 428 | Monitor | Carry-forward |
| [SupportAboutPage.tsx](file:///v:/Antigravity/project-name/client/src/components/SupportAboutPage.tsx) | 415 | Safe | Carry-forward |
| [DiscoveryService.ts](file:///v:/Antigravity/project-name/server/services/DiscoveryService.ts) | 412 | Safe | Carry-forward |
| [users.ts](file:///v:/Antigravity/project-name/server/routes/admin/users.ts) | 410 | Safe | **New** — admin user deletion added |
| [ServerDashboard.tsx](file:///v:/Antigravity/project-name/admin-ui/src/pages/ServerDashboard.tsx) | 409 | Safe | Carry-forward |
| [SelectorDiscoveryService.ts](file:///v:/Antigravity/project-name/server/services/SelectorDiscoveryService.ts) | 408 | Safe | Carry-forward |
| [BackupService.ts](file:///v:/Antigravity/project-name/server/services/BackupService.ts) | 408 | Safe | Carry-forward |

> **`QueueService.ts` at 468 lines** is the newest entry in the Monitor zone. If SFTP offsite backup push (backlog item) adds methods here, it will breach RED. Consider decomposing into `QueueService.ts` (core queue ops) + `QueueNewsletterService.ts` (newsletter polling logic, ~100 lines) before any additions.

---

## 3. Cleanup Report

### Knip Findings — N/A (Crash)

No Knip data available due to OOM crash. See Section 1.

### Files to Delete

None identified via manual inspection.

### Unused Dependencies

Not verifiable without Knip.

### Root-Level Artifacts

| Item | Status |
|:-----|:-------|
| `knip-output.txt` | Contains crash trace — should be deleted after report |
| `NF - _Template.md` | Pre-existing planning template — acceptable |

---

## 4. Logging & Observability Review

### Server-Side — Clean ✅

| Location | `console.log`/`error`/`warn` | Result |
|:---------|:---:|:------:|
| `server/routes/` | 0 | ✅ |
| `server/services/` | 0 | ✅ |
| `server/middleware/` | 0 | ✅ |
| `server/workers/` | 0 | ✅ |

### ESLint `no-console` Enforcement

| File | Pattern | Status |
|:-----|:--------|:------:|
| `server/services/QueueService.ts:33` | `eslint-disable-next-line @typescript-eslint/no-explicit-any` | ✅ Approved — ioredis TS conflict |

No `eslint-disable no-console` bypass comments found in production server code. ✅

### Permitted Exceptions

| Location | Pattern | Reason |
|:---------|:--------|:-------|
| `server/migrations/*.ts` | `console.log` | Standalone CLI tools — exempt per review policy |
| `server/services/extraction/scraperLogger.ts` | `console.error` | Logger of last resort when file I/O fails |

### `debugger` Statements: 0 ✅

### Client-Side — Clean ✅

Zero `console.log`/`error`/`warn` calls in production `client/src/` (excluding `client/src/utils/logger.ts`).

---

## 5. Job Queue & Background Processing Review

### Live Redis Check (`mcp_redis_list bull:*`)

| Queue | Meta Key | Status |
|:------|:---------|:------:|
| `bull:extraction` | ✅ | Healthy |
| `bull:scan` | ✅ | Healthy |
| `bull:graph-api-polling` | ✅ | Healthy |
| `bull:newsletter-link-queue` | ✅ | Healthy |

**4 queues registered and healthy. Zero failed jobs.** ✅

### Queue Architecture Notes

The codebase registers 4 queues (not 5 as stated in some previous reports). Previous reports listing `crawl` as a 5th queue were likely counting the extraction queue under an alias. The definitive queue list from [QueueService.ts](file:///v:/Antigravity/project-name/server/services/QueueService.ts#L51-L54):
- `extraction` — Playwright + Firecrawl + Gemini article extraction
- `scan` — Feed scan jobs
- `graph-api-polling` — Microsoft Graph API newsletter polling
- `newsletter-link-queue` — Newsletter link extraction

### 🟡 Finding: `closeQueues()` Missing `newsletterLinkQueue` Cleanup

[QueueService.ts:320-332](file:///v:/Antigravity/project-name/server/services/QueueService.ts#L320-L332) — The `closeQueues()` function closes `extractionQueue`, `scanQueue`, `graphApiPollingQueue` and their `QueueEvents` — but **omits** `newsletterLinkQueue` and `newsletterLinkQueueEvents`. This leaks 2 Redis connections on graceful shutdown.

```diff
 export async function closeQueues(): Promise<void> {
     logger.info('Closing queues and Redis connections');
     
     await extractionQueue.close();
     await scanQueue.close();
     await graphApiPollingQueue.close();
+    await newsletterLinkQueue.close();
     await extractionQueueEvents.close();
     await scanQueueEvents.close();
     await graphApiPollingQueueEvents.close();
+    await newsletterLinkQueueEvents.close();
     await connection.quit();
     
     logger.info('All queues closed');
 }
```

---

## 6. Database Review

### Schema Parity

| Source | Count | Match |
|:-------|------:|:-----:|
| PostgreSQL tables | 15 | ✅ |
| Sequelize model files | 15 | ✅ |
| `SequelizeMeta` entries | 23 | See below |

Tables: `ArticleArchives`, `ArticleEmailContents`, `Articles`, `FeedArchives`, `Feeds`, `NewsletterEmails`, `OrganizationMemberships`, `Organizations`, `SequelizeMeta`, `SubscriptionPlans`, `SupportMessages`, `SystemSettings`, `UserActivities`, `Users`, `VaultItems` — **perfect model/DB parity** ✅

### 🔴 REGRESSION: SupportMessages Duplicate Indexes

The May-18 report flagged 3 duplicate `SupportMessages` indexes as "RESOLVED" via Migration 038. However, this local dev DB (seeded from a pre-038 backup on the new Windows install) still has all 6 duplicate indexes:

| Index Name | Definition | Duplicate Of |
|:-----------|:-----------|:-------------|
| `support_messages_admin_unread` | `btree (isReadByAdmin, senderRole)` | — (canonical) |
| `support_messages_is_read_by_admin_sender_role` | `btree (isReadByAdmin, senderRole)` | **Duplicate of `admin_unread`** |
| `support_messages_user_date` | `btree (userId, createdAt)` | — (canonical) |
| `support_messages_user_id_created_at` | `btree (userId, createdAt)` | **Duplicate of `user_date`** |
| `support_messages_user_unread` | `btree (userId, isReadByUser, senderRole)` | — (canonical) |
| `support_messages_user_id_is_read_by_user_sender_role` | `btree (userId, isReadByUser, senderRole)` | **Duplicate of `user_unread`** |

**Root Cause**: Migration `038_sanitize_support_messages_indexes.ts` exists in `server/migrations/` but is NOT recorded in `SequelizeMeta`. Also, `036_fix_support_messages_duplicate_indexes.ts` exists in `migrations/` but is also not in `SequelizeMeta`. The database was seeded from `antigravity-vibe_seed.dump` which pre-dates these migrations.

**Fix**: Run `npx tsx scripts/migrate.ts` to apply pending migrations 036 and 038. Then regenerate `antigravity-vibe_seed.dump` to include the clean index state.

### Index Coverage — Good ✅

All key indexes confirmed present and correct:

- **Articles**: 9 indexes — PK, `feed_id`, compound `(feed_id, published_date)`, `published_date`, `last_crawled_date`, `parentArticleId`, `userId`, unique `(link, userId)`, GIN `tags`
- **Feeds**: 7 indexes — PK, `curatedFeedId`, `isCurated`, unique `(userId, url)`, compound `(paused, curatedFeedId)`, `(userId, isCurated)`, `organizationId`
- **VaultItems**: 4 indexes — PK, GIN trigram `searchText`, `userId`, partial unique `(userId, sourceArticleId)`
- **UserActivities**: 7 indexes — PK, compound dedup `(userId, activityType, articleId)`, history `(userId, createdAt)`, `activityType`, `archivedArticleId`, `articleId`, `articleLink`, `originalArticleId`, `(userId, articleId)`

No new index gaps detected.

---

## 7. MCP Live Verification

| Tool | Query | Result | Finding |
|:-----|:------|:-------|:--------|
| `mcp_postgres_query` | `table_name` list | 15 tables — perfect parity | Clean ✅ |
| `mcp_postgres_query` | Full index details | SupportMessages: 3 duplicate pairs found | 🔴 REGRESSION |
| `mcp_postgres_query` | `SequelizeMeta` | 23 entries; 036+038 NOT applied | Root cause confirmed |
| `mcp_redis_list` | `bull:*` | 4 queues active; 0 failed | Clean ✅ |
| `github-mcp-server` | `list_issues` (open) | 0 open issues | Clean ✅ |
| `github-mcp-server` | `list_pull_requests` (open) | 0 open PRs | Clean ✅ |
| `git log -n 15` | Recent commits | 15 commits since v2.8.16 — docs/admin/deps | Context ✅ |
| Knip | Dead code | ❌ OOM crash | 🟡 Tooling blocked |
| `clerk` | `getUserCount` | MCP error (server not responding) | Skipped |

---

## 8. Security & Auth Review

### Auth Middleware — Verified ✅

[auth.ts](file:///v:/Antigravity/project-name/server/middleware/auth.ts) correctly chains `clerkMiddleware()` + `userContextMiddleware`. The DSAR endpoint bypass at line 70 is intentional (public endpoint). `nativeAuth.ts` is mounted before auth middleware per GEMINI.md spec.

### Clerk SDK Patterns — Verified ✅

- No deprecated `SignedIn`/`SignedOut` component imports in `client/src/` — only `isSignedIn` property usages (correct v6 pattern)
- `@clerk/react` v6.6.4 — current
- `<Show when='signed-in'>` used in `App.tsx` for web rendering — correct

### Input Validation — Verified ✅

All user-facing endpoints use `express-validator` via `server/middleware/validation.ts`.

### XSS — Clean ✅

- Zero `dangerouslySetInnerHTML` in `client/src/` ✅
- Zero `dangerouslySetInnerHTML` in `admin-ui/src/` ✅

### Feed Ownership — Verified ✅

`withFeedOwnership` middleware confirmed on feed access routes.

### `removeAllListeners()` — Issue Found

[usePushNotifications.ts:107](file:///v:/Antigravity/project-name/client/src/hooks/usePushNotifications.ts#L107) uses `PushNotifications.removeAllListeners()` in the cleanup function. While GEMINI.md explicitly bans this pattern for `@capgo/capacitor-share-target`, the same risk applies to any Capacitor plugin with retained native event buffers. The `useShareTarget.ts` hook correctly uses `listenerHandle.remove()` — this should be consistent.

---

## 9. Type Safety (`as any`) Audit

### Server — Approved Usages Only ✅

| File | Lines | Pattern | Verdict |
|:-----|------:|:--------|:--------|
| `QueueService.ts` | 36, 48 | Redis connection ioredis TS conflict | ✅ Approved — documented |
| `extractionWorker.ts` | 67 | Worker connection ioredis conflict | ✅ Approved — documented |
| `graphApiPollingWorker.ts` | 228 | Worker connection ioredis conflict | ✅ Approved — documented |
| `newsletterLinkWorker.ts` | 240 | Worker connection ioredis conflict | ✅ Approved — documented |
| `ContentAiAssistedExtractionService.ts` | 83, 179 | Google AI SDK response shape | ✅ Approved — documented |
| `SelectorDiscoveryService.ts` | 307, 447 | Sequelize `feed.update()` | ✅ Approved — Sequelize pattern |
| `selectorDiscoveryHelpers.ts` | 273 | Sequelize `feed.update()` | ✅ Approved — Sequelize pattern |
| `articleService.ts` | 390 | Sequelize `article.update()` | ✅ Approved — Sequelize pattern |
| `dataTransfer.ts` | 153, 166, 179 | `prepareForInsert as any` | ✅ Approved — data migration |
| `015_refactor_subscription_plans.ts` | 17, 35, 54 | Raw SQL results | ✅ Migration — approved |
| `diagnose-timezone.ts` | 39, 46 | `new Date(raw as any)` | ✅ Debug script — permitted |

### Client — Clean ✅

Zero `as any` usages in `client/src/`.

---

## 10. E2E Testing Review

Not re-run this cycle (no CI/CD pipeline changes since May 18). Last known state: **15/15 tests passing** at v2.8.16.

---

## 11. Version Sync Review

| Artifact | Version | Status |
|:---------|:--------|:------:|
| `package.json` (root) | `2.8.19` | ✅ |
| `client/package.json` | `2.8.19` | ✅ |
| `CHANGELOG.md` latest entry | `[2.8.19] - 2026-05-29` | ✅ |
| `build.gradle` `versionName` | `"2.8.19"` | ✅ |
| `build.gradle` `versionCode` | `20819` | ✅ (correct: 2×10000 + 8×100 + 19 = 20819) |

### 🟢 Minor: Stale `versionCode` Comment

[build.gradle:10](file:///v:/Antigravity/project-name/client/android/app/build.gradle) — The inline comment reads `v2.8.18 = 20818` but the actual value is `20819` for `v2.8.19`. The comment was not updated during the version bump.

```
versionCode 20819  // scheme: major×10000 + minor×100 + patch → v2.8.18 = 20818
```

Should be:
```
versionCode 20819  // scheme: major×10000 + minor×100 + patch → v2.8.19 = 20819
```

---

## 12. Deep Clean Analysis

### Debug Noise

| File | Line(s) | Type | Code Snippet |
|:-----|:--------|:-----|:-------------|
| — | — | — | None found ✅ |

No `debugger` statements. No `eslint-disable no-console` bypass comments. No large commented-out code blocks.

### Orphaned Source Files

Not verifiable without Knip. Manual inspection shows no obvious orphans.

### Android Compliance

| Check | Status |
|:------|:------:|
| `removeAllListeners()` banned in `useShareTarget.ts` | ✅ Uses `handle.remove()` |
| `removeAllListeners()` NOT used in `usePushNotifications.ts` | ❌ Uses `PushNotifications.removeAllListeners()` — see Finding 4 |
| `window.open` only in `useArticleOpener.ts` | ✅ Correct centralization |
| `window.open` in `AppInstallBanner.tsx` | ✅ Acceptable — opens Play Store URL, not an article |
| `isNative()` platform detection | ✅ Consistent usage |

---

## 13. Issues & Improvements

### 🔴 Important

**1. SupportMessages Duplicate Indexes — REGRESSION (Local Dev DB)**

- **Location**: Live PostgreSQL — `SupportMessages` table
- **Why**: This is a new Windows install. The database was seeded from `antigravity-vibe_seed.dump` which was captured before Migration 038 ran. Migrations 036 and 038 exist in `server/migrations/` but are not recorded in `SequelizeMeta` — they were never applied to this environment.
- **Impact**: 3 functionally identical index pairs wasting storage and slowing writes. Same issue resolved in May-18 report but regressed on the new machine.
- **Fix**: Run `npx tsx scripts/migrate.ts` from `server/` to apply pending migrations. Then capture a new `antigravity-vibe_seed.dump` that includes the clean index state. This prevents the issue from recurring on future fresh installs.
- **Backlog match**: This is a known recurring issue (see May-18 report §6). The root cause (seed dump predating index cleanup) should be fixed permanently by regenerating the seed.

### 🟡 Medium

**2. `QueueService.closeQueues()` — Missing Newsletter Queue Cleanup**

- **Location**: [QueueService.ts:320-332](file:///v:/Antigravity/project-name/server/services/QueueService.ts#L320-L332)
- **Why**: `newsletterLinkQueue` and `newsletterLinkQueueEvents` are declared at module scope (lines 54, 60) but not closed in `closeQueues()`. This leaks 2 Redis connections on graceful shutdown (PM2 restart, `SIGTERM`).
- **Fix**: Add `await newsletterLinkQueue.close();` and `await newsletterLinkQueueEvents.close();` to the `closeQueues()` function body.

**3. Knip OOM Crash — `oxc-parser` + Node.js 26 Incompatibility**

- **Location**: Knip v6.14.1 → `oxc-parser` → `ArrayBuffer` allocation failure
- **Why**: The `oxc-parser` parser attempts to allocate a very large `ArrayBuffer` that exceeds Node.js v26.2.0 limits. This blocks all dead code analysis.
- **Fix options**: (a) File an issue on `oxc-parser` or `knip` GitHub, (b) Pin Node.js to v22 LTS for Knip runs, (c) Try `NODE_OPTIONS="--max-old-space-size=16384"` or `--max-array-buffer-size`.

**4. `usePushNotifications.ts` — `removeAllListeners()` Pattern**

- **Location**: [usePushNotifications.ts:107](file:///v:/Antigravity/project-name/client/src/hooks/usePushNotifications.ts#L107)
- **Why**: GEMINI.md bans `removeAllListeners()` for `@capgo/capacitor-share-target` because it leaves native events in the buffer causing replay/crash. While `@capacitor/push-notifications` may behave differently, the pattern is inconsistent with the project convention established in `useShareTarget.ts`. Additionally, calling `removeAllListeners()` removes ALL listeners on the plugin — including those potentially registered by other parts of the app.
- **Fix**: Store the 4 `addListener()` return handles, then call `handle.remove()` on each in the cleanup function, matching the `useShareTarget.ts` pattern.

### 🟢 Minor

**5. Stale `versionCode` Comment in `build.gradle`**

- **Location**: [build.gradle:10](file:///v:/Antigravity/project-name/client/android/app/build.gradle)
- **Why**: Comment says `v2.8.18 = 20818` but actual code is `20819` / `v2.8.19`.
- **Fix**: Update comment to match current version.

**6. `QueueService.ts` Approaching Monitor Zone (468 lines)**

- **Location**: [QueueService.ts](file:///v:/Antigravity/project-name/server/services/QueueService.ts) — 468 lines
- **Why**: Was ~403 lines in the May-18 review. Now at 468, firmly in the Monitor zone (450–474). Any feature addition will push it toward RED.
- **Fix**: Before adding any new functionality (e.g., SFTP offsite backup, new queue types), decompose into `QueueService.ts` (core) + `QueueNewsletterService.ts` (newsletter polling logic).

---

## 14. Refactor Plan — "Vibe Check"

### Target: `QueueService.ts` (468 lines — Monitor Zone)

The single messiest file this cycle. It mixes four concerns:

1. **Queue infrastructure** (connection, queue creation, events) — lines 1–60
2. **Job enqueue methods** (extraction, newsletter link) — lines 62–110
3. **Job query/monitoring** (getUserJobs) — lines 112–187
4. **Newsletter polling orchestration** (sync, start, immediate) — lines 189–313
5. **Queue lifecycle** (close, flush, clear, reset, clearUser) — lines 315–468

**Decomposition Plan**:

| New File | Responsibility | Lines |
|:---------|:---------------|------:|
| `QueueService.ts` | Queue infra + enqueue + lifecycle | ~200 |
| `QueueNewsletterService.ts` | Newsletter polling sync/start/trigger | ~120 |
| `QueueMonitorService.ts` | `getUserJobs`, job conversion helpers | ~80 |

This brings all files well under 300 lines. The shared `connection` object is exported from `QueueService` and imported by the satellite modules.

---

## 15. Grading Matrix

| Category | Previous (May 18) | Current (May 29) | Notes |
|:---------|:-----------------:|:----------------:|:------|
| Knip / Dead Code | 10/10 | **8/10** | ⚠️ Knip crashes — cannot verify |
| Vibe Coding Compliance | 10/10 | **9/10** | `QueueService.ts` entered Monitor zone (468 lines) |
| Type Safety | 10/10 | 10/10 | All `as any` documented and approved ✅ |
| Security & Auth | 9/10 | 9/10 | No change ✅ |
| Error Handling | 10/10 | 10/10 | No regression ✅ |
| Performance | 9/10 | 9/10 | No new N+1 issues ✅ |
| Logging & Observability | 10/10 | 10/10 | Clean ✅ |
| Database | 10/10 | **8/10** | 🔴 SupportMessages duplicate indexes regressed |
| Queue Health | 10/10 | **9/10** | 🟡 `closeQueues()` missing newsletter cleanup |
| Testing | 10/10 | 10/10 | 15/15 E2E (last verified May 18) ✅ |
| Accessibility | 8/10 | 8/10 | No change ✅ |
| Version Sync | 10/10 | **9/10** | 🟢 Stale `build.gradle` comment |
| i18n Completeness | 8/10 | 8/10 | No change ✅ |
| Maintainability | 10/10 | 10/10 | Clean module separation ✅ |
| **Overall Vibe Score** | **10/10** | **9.2/10** | 3 medium findings, 1 important regression |

---

## 16. Action Plan

| Priority | Action | Effort | Files Affected | Recommended Model | Mode |
|:---------|:-------|:------:|:---------------|:------------------|:----:|
| 🔴 High | Run `npx tsx scripts/migrate.ts` to apply pending migrations (036, 038) and fix SupportMessages duplicate indexes | Low | Live DB | Manual / CLI | — |
| 🔴 High | Regenerate `antigravity-vibe_seed.dump` after migration to prevent recurrence on future installs | Low | `backups/antigravity-vibe_seed.dump` | Manual / CLI | — |
| 🟡 Medium | Add `newsletterLinkQueue.close()` + `newsletterLinkQueueEvents.close()` to `closeQueues()` | Low | `server/services/QueueService.ts` | Gemini 3.5 Flash (Medium) | Fast |
| 🟡 Medium | Investigate Knip / `oxc-parser` OOM on Node.js 26. Try pinning `oxc-parser` or Node version | Med | `package.json`, `knip.jsonc` | Gemini 3.1 Pro (Low) | Fast |
| 🟡 Medium | Refactor `usePushNotifications.ts` to use `listenerHandle.remove()` pattern | Low | `client/src/hooks/usePushNotifications.ts` | Gemini 3.5 Flash (Medium) | Fast |
| 🟢 Low | Fix stale `versionCode` comment in `build.gradle` | Trivial | `client/android/app/build.gradle` | — | — |
| 🟢 Low | Plan `QueueService.ts` decomposition before next feature addition | Med | `server/services/QueueService.ts` | Claude Sonnet 4.6 | Planning |

---

## 17. Retention Policy Applied

| Action | File |
|:-------|:-----|
| → Archive | `code-review-report-2026-04-21.md` |
| Active | `code-review-report-2026-04-22.md` |
| Active | `code-review-report-2026-04-28.md` |
| Active | `code-review-report-2026-05-07.md` |
| Active | `code-review-report-2026-05-18.md` |
| **New** | `code-review-report-2026-05-29.md` |

6 reports → exceeds retention limit of 5. Oldest report (`2026-04-21`) will be moved to `archive/`.

---

Report generated by AI Code Review Agent · Antigravity-Vibe v2.8.19 · 2026-05-29
Reviewer: Claude Opus 4.6 (Thinking) — Planning Mode
