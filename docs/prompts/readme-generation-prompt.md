# README Generation Prompt — Antigravity-Vibe

> **Last Optimized**: 2026-05-29

**Role:**
You are a Technical Writer and Open Source Maintainer. I need you to generate a professional, high-quality README.md file for this project that is ready for GitHub.

**Context:**
The project is **Antigravity-Vibe** (branded as **Antigravity-Vibe** in the UI) — an AI-powered news reader and feed management platform (v2.8.19). It is built as an npm workspaces monorepo with five applications: `client/` (React 19 + Vite 7 web app with Capacitor 8 Android hybrid), `admin-ui/` (React 19 + Vite 7 admin dashboard), `server/` (Node.js 22+ Express 5 API), `www/` (Astro 6 marketing site), and `e2e/` (Playwright 1 end-to-end tests). Shared TypeScript types live in `packages/types/` (`@project-name/types`).

Please analyze the current state of the codebase (files, package.json, .env examples, and folder structure) to ensure the documentation is accurate.

**Required Sections & Content:**
1. **Header:**
  * Project Title: **Antigravity-Vibe** with a catchy one-line description.
  * **Badges:** Add distinct shields.io badges for: Node.js 22+, React 19, Express 5, TypeScript 5, the verified License (ISC — confirmed in root `package.json`), and Status (Active Development).

2. **About the Project:**
  * A concise overview of what the app does — bridges the gap between RSS and the open web using AI-powered content discovery.
  * **Key Features:** Bullet points highlighting:
    - 🤖 AI integration (Google Gemini SDK for headline ratings, content summarization, selector discovery)
    - 🕷️ Multi-strategy scraping (Playwright stealth browser, Cheerio, Mozilla Readability, rss-parser)
    - 📰 Feed Management (personal, curated, organization feeds with promotion/demotion)
    - 🏢 Multi-tenant support (Clerk organizations with local database mirror)
    - 📊 Pino 10 structured logging with pino-http 11
    - ⚙️ BullMQ 5 background job processing (content extraction, newsletter link processing, Graph API polling)
    - 🏛️ Vault (save & annotate articles with TipTap rich text editor)
    - 📱 Android hybrid app (Capacitor 8 with push notifications, deep linking, native Google OAuth, share intents)
    - 🌐 Internationalization (i18next 26 with multiple language support)
    - 🎨 Cyberpunk-Executive design system with Antigravity-Vibe and aiVOLUTION theme modes (MUI 7)
    - 📧 Newsletter email feeds (Microsoft Graph API integration)
    - 🔐 Clerk identity with webhook sync and local database mirror
    Use emojis for visual hierarchy.

3. **Tech Stack:**
  * Create a visual or list-based section covering:
    - **Frontend**: React 19, Vite 7, MUI 7, Framer Motion 12, TanStack React Query 5, Clerk React 6, i18next 26, Lucide Icons 1, TipTap 3, react-markdown 10, babel-plugin-react-compiler 1
    - **Admin UI**: React 19, Vite 7, MUI 7 (+ MUI X Date Pickers 8), Clerk React 6, React Router DOM 7, react-colorful 5
    - **Backend**: Node.js 22+, Express 5, Sequelize 6 (PostgreSQL), Clerk Express 2 / Clerk Backend 3
    - **AI**: Google Generative AI SDK 0.x (`@google/generative-ai`)
    - **Scraping**: Playwright 1 (+ playwright-extra 4 + stealth plugin), Cheerio 1, Mozilla Readability 0.6, Turndown 7, rss-parser 3
    - **Database**: PostgreSQL (pg 8, Sequelize 6)
    - **Logging**: Pino 10, pino-http 11
    - **Job Queue**: BullMQ 5 + Redis (ioredis 5), Bull Board 7
    - **Push Notifications**: Firebase Admin 13 (FCM)
    - **Mobile**: Capacitor 8, @capgo/capacitor-social-login 8, @capgo/capacitor-share-target 8
    - **Marketing Site**: Astro 6
    - **Testing**: Playwright 1 (E2E), @clerk/testing 2
    - **Security**: Helmet 8, express-rate-limit 8, express-validator 7, Svix 1 (webhook verification)
    - **Deployment**: Caddy (reverse proxy), PM2 (process manager), GitHub Actions CI/CD
    - **Tools**: ESLint 9, Prettier 3, TypeScript 5, Knip 6 (dead code), tsx 4 (runtime)

4. **Getting Started (Crucial):**
  * **Prerequisites:** Node.js 22+, npm 10+, PostgreSQL 14+, Redis 7+. Optional: Android Studio + JDK 17 (for mobile development).
  * **Installation:** Provide clear, copy-pasteable commands:
    ```bash
    git clone https://github.com/your-org/project-name.git
    cd project-name
    npm install
    ```
  * **Environment Setup:** Document ALL necessary .env variables grouped by service:
    - **Database**: `DATABASE_URL`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`
    - **Auth**: `CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, `CLERK_WEBHOOK_SECRET`
    - **AI**: `GEMINI_API_KEY`
    - **Redis**: `REDIS_HOST`, `REDIS_PORT`
    - **Logging**: `LOG_LEVEL`
    - **Firebase**: `FIREBASE_SERVICE_ACCOUNT` (JSON key)
    - **Microsoft Graph**: `MS_GRAPH_CLIENT_ID`, `MS_GRAPH_CLIENT_SECRET`, `MS_GRAPH_TENANT_ID`
    - **Google OAuth**: `GOOGLE_WEB_CLIENT_ID` (server), `VITE_GOOGLE_WEB_CLIENT_ID` (client)
    - **Client**: `VITE_CLERK_PUBLISHABLE_KEY`, `VITE_API_URL` (mode-specific)
    Reference `.env.example` files in `client/`, `server/`, and `admin-ui/`.
  * **Running locally:**
    ```bash
    npm run start          # All apps (server + client + admin + www)
    cd server && npm run dev   # Server only
    cd client && npm run dev   # Client only
    cd admin-ui && npm run dev # Admin only
    ```

5. **Project Structure:**
  * Generate a simplified tree view based on the actual file structure:
    ```
    project-name/
    ├── client/          # React 19 web app + Capacitor Android hybrid
    │   ├── src/         # Components, hooks, context, theme, utils, i18n
    │   ├── android/     # Native Android project
    │   └── capacitor.config.ts
    ├── admin-ui/        # Admin dashboard (React 19)
    ├── server/          # Express 5 API server
    │   ├── models/      # Sequelize model definitions (14 models)
    │   ├── routes/      # Express route handlers
    │   ├── services/    # Business logic services
    │   ├── middleware/   # Auth, validation, feed ownership
    │   ├── workers/     # BullMQ job processors
    │   ├── migrations/  # Database migration scripts
    │   ├── scripts/     # CLI debug & utility scripts
    │   ├── prompts/     # AI prompt templates
    │   └── templates/   # Email/HTML templates
    ├── www/             # Astro 6 marketing site
    ├── e2e/             # Playwright E2E tests
    ├── packages/types/  # Shared TypeScript interfaces (@project-name/types)
    ├── .skills/         # AI operational skills (8 domains)
    ├── docs/            # Architecture, guides, project docs, prompts
    ├── scripts/         # Dev & deployment scripts
    └── GEMINI.md        # AI agent project rules
    ```

6. **Usage:**
  * Brief instructions on how to add a feed, trigger a crawl, and view results.
  * Mention the admin dashboard at `http://localhost:5174` for feed curation and user management.

7. **Project and code status:**
  * Read the most recent version entry from `CHANGELOG.md` and summarize what was released.
  * If `docs/project/feature-status.md` exists, summarize what features are currently live.
  * (Comment this with great sense of humor from a nerdy engineer)

8. **Troubleshooting / FAQ:**
  * Add a Troubleshooting section with the most common setup issues:
    * PostgreSQL not running → `pg_isready` check, verify DATABASE_URL
    * Redis not running → `redis-cli ping`, check REDIS_HOST/PORT
    * Clerk webhook issues → Need `caddy run` for local tunnel
    * Android emulator blank screen → Run `adb reverse tcp:5173 tcp:5173` and `adb reverse tcp:3001 tcp:3001`
    * `VITE_GOOGLE_WEB_CLIENT_ID` missing → Required on both client AND server for native OAuth
  * Keep it short — 5 items max, each with a one-line fix.

9. **API Documentation:**
  * List the major endpoint groups:
    - `/feeds` — Feed CRUD and management
    - `/articles` — Article listing and state management
    - `/auth` — Authentication sync and user profile
    - `/organizations` — Organization and membership management
    - `/vault` — Vault item CRUD
    - `/support` — Support chat messaging
    - `/admin` — Admin-only operations (user management, system settings)
    - `/webhooks` — Clerk webhook handler
    - `/push-notifications` — FCM token registration
    - `/translations` — i18n translation management
  * Authentication: Clerk JWT via `Authorization: Bearer` header (Express middleware).
  * Note that detailed API docs are not yet published.

10. **Deployment:**
  * Brief overview: Caddy reverse proxy → PM2 process manager → Node.js/Express.
  * Environment matrix: development, test (`test.antigravity.vibe`), production (`antigravity.vibe`).
  * CI/CD via GitHub Actions (`deploy-and-test.yml`, `build-android.yml`, `cleanup.yml`).
  * Reference `docs/deployment/` for detailed deployment guides and `scripts/deploy/` for automated scripts.

11. **Roadmap & Contributing:**
  * Reference `docs/project/roadmap.md` for the full roadmap. List 3-4 future planned features from analysis.
  * **Contributing**: No `CONTRIBUTING.md` exists. Write a short contributing paragraph encouraging PRs, referencing `GEMINI.md` for coding standards and the Vibe Coding philosophy.

12. **Developer Context, Runbooks & AI Environments:**
  * **Operational Skills** (`.skills/`):
    - `admin-ui-patterns` — Admin UI development patterns and conventions
    - `capacitor-ops` — Android build, debug, and deployment procedures
    - `clerk-sync-tracer` — Clerk auth flow, webhook debugging, sync verification
    - `cyberpunk-ui-crafter` — UI design system implementation and theming
    - `deployment-ops` — Server deployment and infrastructure operations
    - `feed-scraper` — Feed scraping pipeline and extraction strategies
    - `project-migrations` — Database migration creation, execution, and deployment protocol
    - `security-isolation` — Security boundaries and tenant isolation patterns
  * **AI Context Servers (MCP)**:
    - `postgres` — Direct read-only SQL against the local database
    - `redis` — Inspect BullMQ queue state and cache keys
    - `clerk` — Up-to-date Clerk SDK patterns
    - `github-mcp-server` — GitHub search, issue tracking, PR management
    - `playwright` — Browser automation and testing
    - `docker` — Container management
    - `firebase-mcp-server` — Firebase project management
    - `fetch` — URL fetching and YouTube transcript extraction
  * **Execution Environments Matrix**:
    - **Antigravity IDE** — Multi-file development, feature building, complex logic
    - **Antigravity Application** — UI prototyping, visual design, asset generation
    - **Antigravity CLI** — Batch automations, headless test suites, scheduled scripts
  * Link to `docs/project/vibe-coding-101.md` and `docs/project/chore-reviews.md`.

13. **Mobile App:**
  * **Architecture**: Capacitor 8 wrapping the React 19 web app as a hybrid Android app.
  * **Platform**: Android (package: `com.antigravity.vibe`). iOS not yet supported.
  * **Key features**: FCM push notifications (`@capacitor/push-notifications`), Chrome Custom Tabs (`@capacitor/browser`), deep linking (`antigravity-vibe://auth/callback`), native Google OAuth (`@capgo/capacitor-social-login`), share intents (`@capgo/capacitor-share-target`), platform detection (`isNative()` from `client/src/utils/platform.ts`).
  * **Building**: Requires Android Studio + JDK 17. Use `npm run android:dev` for dev builds or `android-dev.ps1` PowerShell launcher.
  * **CI/CD**: GitHub Actions workflow (`.github/workflows/build-android.yml`) for automated APK/AAB builds.
  * Reference `docs/guides/android-app-development.md`, `docs/guides/android-cicd-pipeline.md`, and `docs/guides/google-play-listing.md`.

**Style Guidelines:**
* Use standard Markdown.
* Keep the tone unformal and friendly.
* Use emojis for visual hierarchy.
* Use bullet points for lists.
* Use short sentences.
* Use humorous and relatable language.
* Include a placeholder for a "Demo Screenshot" like `![App Screenshot](./docs/screenshot.png)`.