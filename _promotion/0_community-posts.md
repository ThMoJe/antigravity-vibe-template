# Community Post Drafts

---

## 1. Reddit — r/vibecoding

**Title:** Built a system where Antigravity review prompts update themselves when your codebase changes

**Body:**

Hey vibers 👋

I built a self improving and healing workflow for Antigravity 2.0 where your AI review prompts **update themselves** based on your actual codebase. No more stale checklists that still reference packages you removed months ago.

The core idea: **meta-prompts** — instructions that tell the AI to scan your `package.json`, `GEMINI.md`, skills, and MCP config, then *generate* hyper-specific review prompts from scratch. Every time. Your code review checklist literally reads the newspaper every morning.

What's in the template:

- 🧠 Self-optimizing prompts for code review, spring cleaning, and architecture review
- 🔄 Dual-model implementation review (Gemini drafts the plan, Claude stress-tests it)
- 🛠️ Modular `.skills/` the AI loads on-demand instead of bloating every conversation
- 📊 Timestamped review reports with retention policies — resolved issues stay resolved
- 📱 A script that consolidates all your docs into a Gemini Gem so you can query your repo from your phone

**GitHub:** [https://github.com/ThMoJe/antigravity-self-evolving-reviews](https://github.com/ThMoJe/antigravity-self-evolving-reviews)

**Help me improve it? The README has a TLDR — you can be running self-evolving reviews on your own project in about 10 minutes.** Download, unzip into your workspace, run `/run-meta-prompts`, then run `/spring-cleaning` or `/code-review`. That's it.

Feedback, suggestions, and PRs welcome!

---

## 2. Google AI Developer Forum — discuss.ai.google.dev/c/antigravity

**Title:** `[Show & Tell] Self-optimizing AI review prompts — an Antigravity 2.0 boilerplate template`

**Body:**

Hi everyone 👋

I wanted to share something I've been building with Antigravity 2.0 that I think might be useful to other vibe coders here.

**Quick backstory:** I coded professionally 20+ years ago, then stopped. I recently came back to build a full-stack app — but this time I don't write code. I command it. Antigravity is my IDE, Gemini and Claude are my engineering team, and I just set the direction.

Along the way, I developed a workflow infrastructure that I've extracted into a reusable template. The key innovation is **meta-prompting** — prompts that analyze your live codebase and generate context-aware review prompts automatically.

### What's in the template

- **Meta-prompts** (`docs/prompts/_meta/`) that scan your `package.json` files, `GEMINI.md` rules, `.skills/` directories, and MCP server config to generate hyper-specific code review, spring cleaning, and architecture review prompts
- **Modular skills** (`.skills/`) — domain-specific runbooks the AI loads lazily instead of bloating every conversation's context
- **Slash-command workflows** (`.agent/workflows/`) — `/code-review`, `/spring-cleaning`, `/architecture-review`, `/run-meta-prompts`, and more
- **Report retention** — timestamped reports with archival policies so the AI reads its own history and doesn't re-flag resolved issues
- **Dual-model implementation review** — Gemini reviews the plan as architect, Claude stress-tests it as pragmatist
- **Gemini Gems integration** — a `consolidate_docs.ps1` script that merges all `.md` files so you can upload them to a Gem and ask your repo questions from the Google Gemini app on your phone 📱

### The repo

**GitHub:** [https://github.com/ThMoJe/antigravity-self-evolving-reviews](https://github.com/ThMoJe/antigravity-self-evolving-reviews)

Fair warning: I haven't published anything to GitHub since before Git existed (I'm a CVS/SVN era survivor 🦖). If I've committed some open-source faux pas, I'd genuinely appreciate a heads-up.

This template was extracted from a production app I'm building entirely with Antigravity, and I'm close to releasing it. I'm sharing because I think my approach — as a non-developer doing vibe coding — might offer a different perspective than what experienced developers would build. Sometimes not knowing "the right way" leads to interesting solutions.

Feedback, suggestions, and PRs welcome!
