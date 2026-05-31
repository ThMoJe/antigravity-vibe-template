# Documentation Index

> **Start here**: `GEMINI.md` (project rules & coding standards)

---

## 🤖 AI Prompts

> [!CAUTION]
> Prompts are **generated outputs** — edit the META prompts in `prompts/_meta/` instead. See `GEMINI.md` § AI Prompt Ownership.

| Prompt | Purpose |
|:-------|:--------|
| [Code Review](./prompts/code-review-prompt.md) | Vibe Coding compliance review |
| [Spring Cleaning](./prompts/spring-cleaning-prompt.md) | Dead code and tech debt analysis |
| [README Generation](./prompts/readme-generation-prompt.md) | Auto-generate README.md |
| [Architecture Review](./prompts/architecture-review.md) | Architecture audit |
| [Known Patterns](./prompts/known-patterns.md) | Documented intentional code patterns |

### Meta-Prompts (Sources)

| Meta-Prompt | Generates |
|:------------|:----------|
| [`optimize-code-review.md`](./prompts/_meta/optimize-code-review.md) | `code-review-prompt.md` |
| [`optimize-spring-cleaning.md`](./prompts/_meta/optimize-spring-cleaning.md) | `spring-cleaning-prompt.md` |
| [`optimize-readme.md`](./prompts/_meta/optimize-readme.md) | `readme-generation-prompt.md` |
| [`optimize-architecture-review.md`](./prompts/_meta/optimize-architecture-review.md) | `architecture-review.md` |

---

## 📖 Developer Guides

| Guide | Description |
|:------|:------------|
| [MCP Server Setup](./guides/mcp-server-setup.md) | AI agent MCP server configuration |
| [Developer Tooling & Refactoring](./guides/vibe-coding-compliance.md) | Spring Cleaning, Knip, refactoring patterns |

---

## 🗺️ Project Management

| Document | Description |
|:---------|:------------|
| [Roadmap](./project/roadmap.md) | Strategic feature roadmap |
| [Feature Status](./project/feature-status.md) | Current status of all features |
| [Backlog](./project/backlog.md) | Prioritized backlog items |

---

## 📊 Reports

Generated reports from automated workflows:

| Path | Description |
|:-----|:------------|
| [`reports/code-review/`](./reports/code-review/) | Code review reports (retention: 5 latest) |
| [`reports/spring-cleaning/`](./reports/spring-cleaning/) | Spring cleaning reports (retention: 3 latest) |
| [`reports/architecture-review/`](./reports/architecture-review/) | Architecture review reports (retention: 3 latest) |
| [`reports/archive/`](./reports/archive/) | Archived older reports (deleted after 90 days) |

---

## 🔑 Quick Reference

| Need | Where to look |
|:-----|:--------------|
| **Coding rules** | `GEMINI.md` |
| **Global AI rules** | `~/.gemini/GEMINI.md` |
| **Changelog** | `CHANGELOG.md` |
| **Skills** | `.skills/` |
| **Workflows** | `.agent/workflows/` |
