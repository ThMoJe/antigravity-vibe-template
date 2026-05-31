# Role
You are a **Senior Developer Advocate & Prompt Engineer** specializing in Documentation-as-Code. Your goal is to analyze the currently open VS Code workspace and optimize the prompt in @readme-generator-prompt.md  to create a highly specific, context-aware GitHub README generating prompt

# Context
I have the **Current Prompt** in @readme-generator-prompt.md . However, it might contain hardcoded assumptions (like project name, specific libraries, or folder structures) that might not match the current workspace, as the project changes over time.

I need you to perform a **Deep Static Analysis** of the codebase and then rewrite the current Prompt to be dynamic and factually accurate for *this specific project*.

I also want you to use below base prompt as a template for the final prompt. The final prompt should be dynamic and factually accurate for *this specific project*, but include as much details as possible from the base prompt. 

# Analysis Steps (Execute Silently)
1.  **Project Identity:** Scan `package.json`, `setup.py`, `go.mod`, or equivalent files to determine the Project Name, Description, and **Version**.
2.  **Tech Stack & Architecture:**
    *   Identify the languages (TS, JS, Python, Rust, etc.).
    *   Identify frameworks (React, Vue, Django, Spring, etc.).
    *   Identify the architecture (Monorepo, Microservices, Serverless, Simple Client/Server).
    *   Identify logging libraries (Pino, Winston, etc.) and linting configs.
    *   Identify job queue / background processing systems (BullMQ, Redis, etc.).
    *   Identify deployment tools (Caddy, nginx, Docker, PM2, etc.).
    *   **Version Discovery**: Read **ALL `package.json` files** across all workspaces (root, and every subdirectory with its own `package.json`). For each significant framework and library, extract the **major version number** from `dependencies` or `devDependencies`. Always include the major version when referencing a technology (e.g., "React 19", "Express 5", "Capacitor 8" — never just "React", "Express", "Capacitor").
    *   **Configuration Scanning**: Scan configuration files (e.g., `capacitor.config.ts`, `eslint.config.js`, `vite.config.ts`, CI/CD workflows, Caddyfile, Dockerfiles) to detect tools and infrastructure not visible in `package.json` alone.
    *   **Skills Detection**: Scan the `.skills/` directory (if present). Parse each modular skill (`SKILL.md`). The generated prompt MUST list all active operational domains, runbooks, and example scripts (including `security-isolation`) explicitly in the README.
    *   **MCP Server Detection**: Parse the Antigravity MCP config (typically `~/.gemini/antigravity-ide/mcp_config.json`) to identify active MCP context servers (e.g., `postgres`, `redis`, `clerk`, `playwright`, `github-mcp-server`) available to AI agents.
    *   **Pattern & Methodology Detection**: Identify architectural patterns in active use (service layer, queue/worker, middleware chains, ORM, migration system, etc.) and coding methodologies from the project rules file (e.g., file size limits, naming conventions, coding philosophies).
3.  **Dependency Intelligence:** Look at `dependencies`/`requirements.txt`.
    *   *Example:* If `vitest` is present, the prompt should ask for a "Testing" section.
    *   *Example:* If `docker-compose.yml` exists, the prompt should ask for Docker installation steps.
    *   *Example:* If `bullmq` + `ioredis` are present, the prompt should include Redis as a prerequisite.
    *   *Example:* If `pino` is present, the prompt should mention structured logging in the tech stack.
4.  **Scripts & Commands:** Analyze `scripts` in `package.json` or `Makefile`. Identify the *actual* commands used to start, build, and test the app (e.g., is it `npm run dev` or `yarn start`?). Also check for lint commands (e.g., `npm run lint`).
5.  **Configuration:** Check for `.env.example` or config files to identify necessary environment variables. Include variables for all services (database, auth, AI, Redis, logging, etc.).
6.  **Deployment:** Check for deployment scripts, Caddyfile, nginx configs, or CI/CD pipelines to include deployment information in the README.
7.  **Mobile / Hybrid App:** Check for Capacitor (`capacitor.config.ts`), React Native, or other mobile frameworks. Look for:
    *   Android project directory (`client/android/app/src/main/`) and `client/android/app/build.gradle` for package name and version.
    *   The **framework version** in the relevant `package.json` (e.g., read `@capacitor/core` version from `client/package.json`).
    *   Platform detection utilities (e.g., `isNative()` in `client/src/utils/platform.ts`).
    *   Push notification hooks (FCM, APNs).
    *   Deep linking schemes (custom URL schemes like `antigravity-vibe://` in `AndroidManifest.xml`).
    *   CI/CD workflows for mobile builds (e.g., `.github/workflows/build-android.yml`).
    *   App store listing guides or distribution documentation.
8.  **License Detection**: Check for a `LICENSE` file in the root directory. If found, read the first line to determine the license type (MIT, Apache-2.0, GPL-3.0, etc.). If no `LICENSE` file exists, check the `license` field in root `package.json`. Use the detected license in the README badge — **do NOT default to MIT** if you cannot confirm it.
9.  **Contributing Infrastructure**: Check whether the following exist and note the result for use in the Contributing section:
    *   `CONTRIBUTING.md` — if present, link to it instead of writing generic guidance
    *   `.github/PULL_REQUEST_TEMPLATE.md` — mention it if present
    *   `.github/ISSUE_TEMPLATE/` directory — mention it if present
    *   `CODE_OF_CONDUCT.md` — reference it if present
10. **Project Status Discovery**: Read `CHANGELOG.md` to identify the most recent released version and its changes. Read `docs/project/feature-status.md` to identify what is currently live. **Do NOT assume a code review report exists.** Use CHANGELOG + feature-status as the primary sources for the Project Status section.
11. **Source of Truth**: 
    > [!WARNING]
    > The output prompt must reflect the **current codebase state exclusively**. Do NOT carry forward technology names, version numbers, or architectural descriptions from the existing output file. The existing prompt file provides the template structure and section layout — every factual claim (tech stack, versions, patterns, directory names) must come from your fresh workspace analysis. If a technology was previously mentioned in the output file but is no longer present in any `package.json` or config file, **omit it**. If a technology has been added since the last run, **include it**.
12. **Verify Accuracy**: Before saving the output, confirm:
    *   Every technology and tool mentioned exists in the project's `package.json` files or config files.
    *   Every version number matches the major version declared in `package.json`.
    *   No stale references from a previous version of the output prompt were carried forward without verification.
    *   Newly added technologies and dependencies are included.
    *   Removed technologies and dependencies are no longer mentioned.

# The Task: Prompt Refinement
Refine the **Current Prompt** provided at the bottom of this message.
*   **Remove Hardcoding:** Replace specific names like "agent" or "Gemini" with placeholders or the actual names found in the code analysis.
*   **Inject Specifics:** If you found a specific linter, ORM, logging library, job queue, or testing tool, update the "Tech Stack" instructions to specifically ask for those.
*   **Correct the Structure:** If the project does not have `client/` and `server/` folders, update the "Project Structure" instruction to reflect the actual file tree. Include new directories (e.g., `queues/`) found in the workspace.
*   **Enhance Installation:** Update the "Getting Started" section to explicitly request the package manager you found (npm, pnpm, yarn, pip, cargo) AND any infrastructure prerequisites (Redis, PostgreSQL, etc.).
*   **Update Environment Variables:** Ensure all required env vars are listed, including newer additions (e.g., `REDIS_URL`, `LOG_LEVEL`).

# Output Format
Return **only** the Optimized Prompt and replace the prompt in @readme-generator-prompt.md with the optimized prompt. The optimized prompt must preserve the high standards of the original but contain the specific context you found as well as respecting the structure of the base prompt.

----

**[BASE PROMPT]**

**Role:**
You are a Technical Writer and Open Source Maintainer. I need you to generate a professional, high-quality README.md file for this project that is ready for GitHub.

**Context:**
The project is "{PROJECT_NAME}" (an AI-powered news reader).
Please analyze the current state of the codebase (files, package.json, .env examples, and folder structure) to ensure the documentation is accurate.

**Required Sections & Content:**
1. **Header:**
  * Project Title & a catchy one-line description.
  * **Badges:** Add distinct shields.io badges for: 1-2 primary tech stack components found during analysis (e.g., Node version, React version, Python version), the verified License (do NOT default to MIT without checking the codebase), and Status (Active).
2. **About the Project:**
  * A concise overview of what the app does.
  * **Key Features:** Bullet points highlighting the AI integration, scraping capabilities, Feed Management, structured logging, background job processing, and multi-tenant support. Use emojis for visual hierarchy.
3. **Tech Stack:**
  * Create a visual or list-based section covering: Frontend, Backend, AI, Database, Logging, Job Queue, Deployment, and Tools.
4. **Getting Started (Crucial):**
  * **Prerequisites:** List required tools (Node.js, npm/yarn, PostgreSQL, Redis).
  * **Installation:** Provide clear, copy-pasteable bash commands to clone, install root dependencies, install client/server dependencies, and start the app.
  * **Environment Setup:** Document ALL necessary .env variables including database, auth, AI, Redis, and logging configuration.
5. **Project Structure:**
  * Generate a simplified tree view of the main directories based on the actual file structure. Include server directories like `queues/`, `services/`, `routes/`, and any deployment config.
6. **Usage:**
  * Brief instructions on how to add a feed, trigger a crawl, and view results.
7. **Project and code status:**
  * Read the most recent version entry from `CHANGELOG.md` and summarize what was released. Do NOT reference a code review report — it may not exist. If `docs/project/feature-status.md` exists, summarize what features are currently live.
  * (Comment this with great sense of humor from a nerdy engineer)
8. **Troubleshooting / FAQ:**
  * Add a Troubleshooting section with the most common setup issues for this specific project. Look at the tech stack to infer likely pain points:
    * Database connection errors (PostgreSQL not running, wrong credentials)
    * Redis connection errors (Redis not running, wrong port)
    * Auth callback issues (Clerk redirect URL misconfiguration)
    * Android build failures (missing `google-services.json`, JDK version)
    * Environment variable not found at runtime
  * Keep it short — 3-5 items max, each with a one-line fix.
9. **API Documentation (if applicable):**
  * If the project exposes a REST API (detected by Express routes in `server/routes/`), add a brief API reference section.
  * List the major endpoint groups (e.g., `/feeds`, `/articles`, `/auth`, `/admin`) with a one-line description of each.
  * State the authentication method (e.g., Clerk JWT via `Authorization: Bearer` header).
  * If an OpenAPI/Swagger spec exists, link to it. Otherwise note that detailed API docs are not yet published.
10. **Deployment:**
  * Brief overview of the deployment architecture (reverse proxy, process manager, automated scripts).
  * Reference the deployment documentation directory (`docs/deployment/`).
11. **Roadmap & Contributing:**
  * List 3-4 future planned features based on the roadmap (`docs/project/roadmap.md` if it exists) or code analysis.
  * **Contributing section**: If `CONTRIBUTING.md` exists, link to it. If `.github/PULL_REQUEST_TEMPLATE.md` exists, mention it. Otherwise write a short generic Contributing paragraph. Never omit this section.
12. **Developer Context, Runbooks & AI Environments:**
  * Create a distinct section documenting the operational domains and developer skills based on the `.skills/` directory (e.g., `clerk-sync-tracer`, `project-migrations`, `security-isolation`). Briefly describe what each runbook covers.
  * List the configured AI Context Servers (MCP) detected in the workspace (e.g., local `postgres`, `redis`, `clerk`, `playwright`, `github-mcp-server`) so contributors know what context tools are configured.
  * Document our **Execution Environments Matrix** (IDE vs. Application vs. CLI) from global rules, showing developers which environment fits their prompt workload best.
  * Add a short description of our AI coding workflow and include direct links to `docs/project/vibe-coding-101.md` and `docs/project/chore-reviews.md`.
13. **Mobile App (if applicable):**
  * Overview of the hybrid mobile architecture (e.g., Capacitor wrapping web app).
  * Platform support (Android, iOS).
  * Key mobile features: push notifications, in-app browser, deep linking, platform detection.
  * How to build: prerequisites (Android Studio, JDK), build commands, CI/CD pipeline reference.
  * Link to detailed developer guide and testing guide if they exist.
  * App store status or link to store listing guide.

**Style Guidelines:**
* Use standard Markdown.
* Keep the tone unformal and friendly.
* Use emojis for visual hierarchy.
* Use bullet points for lists.
* Use short sentences.
* Use humorous and relatable language.
* Include a placeholder for a "Demo Screenshot" like ![App Screenshot](./path/to/image.png).
