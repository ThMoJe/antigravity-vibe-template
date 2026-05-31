# Known Code Patterns (Not Bugs)

> **Purpose**: Reference document for code review and spring cleaning workflows to prevent misidentification of intentional design patterns as bugs.

This document catalogs code patterns in the codebase that may appear suspicious during automated analysis but are **intentional design choices**. Always consult this document before flagging potential issues.

---

## 1. {Pattern Name}

**Pattern**: Brief description of what the code looks like.

**Why This Is Correct**:
- Explanation of why this pattern exists.
- Technical justification for the approach.

**Where to Find**: `path/to/file.ts` (line X)

**Resolution**: Not a bug. {Brief explanation of when this can be revisited.}

---

## Usage Guidelines

### For Code Reviews
1. Before flagging `as any` casts or unusual patterns, check this document
2. If a pattern matches, note it as "intentional design pattern" rather than a bug
3. If simplification is possible, suggest it as a low-priority improvement, not a critical issue

### For Spring Cleaning
1. This document focuses on **logic patterns**, not file/dependency cleanup
2. Spring cleaning should still flag orphaned files, unused dependencies, and dead code
3. Consult this document only if the analysis identifies suspicious structural patterns

### For Meta-Prompt Optimization
1. When regenerating `code-review-prompt.md` or `spring-cleaning-prompt.md`, always include a reference to this document
2. Ensure the regenerated prompt instructs the reviewer to consult this document before flagging suspicious patterns

---

## Maintenance

**When to Update**:
- A code review flags an intentional pattern as a bug
- A new `as any` exception is approved
- A new security middleware pattern is introduced
- An existing pattern is refactored or removed

**How to Update**:
1. Add new patterns to the numbered list above
2. Update the "Where to Find" field if files are refactored
3. Remove patterns when the underlying SDK/library no longer requires the workaround
