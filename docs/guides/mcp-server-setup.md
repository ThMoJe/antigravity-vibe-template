# MCP (Model Context Protocol) Server Setup

## What is MCP?
*   **Model Context Protocol (MCP)**: Standard connecting AI systems (e.g., Google Antigravity) with external tools and data sources.
*   **Purpose**: Extends AI capabilities by providing access to specialized functions, external information, and services.
*   **Use Cases**: Enables AI assistant to:
    *   Read database (Postgres).
    *   Inspect Redis queues.
    *   Scrape and search web content (Firecrawl).
    *   Access Clerk SDK patterns.
    *   Manage GitHub issues/PRs.

## Configured Servers
*   Defined in `mcp_config.json` (typically `~/.gemini/antigravity/mcp_config.json`).
*   **Primary Servers**:

| Server | Execution Environment | Primary Use Case |
|:-------|:----------------------|:-----------------|
| **Postgres** | Node / `npx` | Direct read-only SQL against local database (`mcp_postgres_query`) |
| **Redis** | Node / `npx` | Inspect BullMQ queue state and cache keys |
| **Firecrawl** | Node / `npx` | Web scraping, crawling, search, and content extraction |
| **Clerk** | Node / `npx` | Up-to-date Clerk SDK patterns for Express/React |
| **GitHub** | **Docker Container** | GitHub search, issue tracking, PR management operations |

*   **MCP Server Architecture Overview**:
    ```mermaid
    graph TD
        A[Google Antigravity] --> B(MCP Server Interface)
        B --> C1(Postgres MCP - Node/npx)
        B --> C2(Redis MCP - Node/npx)
        B --> C3(Firecrawl MCP - Node/npx)
        B --> C4(Clerk MCP - Node/npx)
        B --> C5(GitHub MCP - Docker)

        C1 --> D1[Local PostgreSQL DB]
        C2 --> D2[Local Redis Instance]
        C3 --> D3[Web Content - Scrape/Search]
        C4 --> D4[Clerk SDK Patterns]
        C5 --> D5[GitHub API]

        style C5 fill:#f9f,stroke:#333,stroke-width:2px
    ```

## The GitHub MCP Server (Docker)
*   **Observation**: Sporadic `ghcr.io/github/github-mcp-server` Docker containers may appear. **This is expected behavior.**
*   **Reason**: The official GitHub MCP server is designed by GitHub to run natively as an isolated Docker container, not directly via Node.js.
*   **Configuration Example**:
    ```json
    "github-mcp-server": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "GITHUB_PERSONAL_ACCESS_TOKEN",
        "ghcr.io/github/github-mcp-server"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_pat_here"
      }
    }
    ```
*   **Initiation**: Google Antigravity automatically starts these containers in the background whenever the AI executes a workflow requiring GitHub API access (e.g., `/code-review`, reading pull requests, searching issues).
*   **Lifecycle & Cleanup**:
    *   Ephemeral: Each instance is new and isolated for the specific request.
    *   Automatic Deletion: The `--rm` flag ensures Docker automatically deletes the container the moment the AI session ends or the connection closes. No manual management or cleanup is required.

*   **GitHub MCP Server Workflow**:
    ```mermaid
    sequenceDiagram
        participant A as Google Antigravity
        participant D as Docker Engine
        participant C as GitHub MCP Container
        participant G as GitHub API

        A->>A: AI workflow requires GitHub access
        A->>D: Invoke `docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server`
        D->>C: Start new container instance (with PAT)
        C->>G: Make GitHub API request
        G-->>C: Return API response
        C-->>A: Forward response to Antigravity
        A->>A: Process GitHub data
        C--x D: Container exits (due to --rm)
        D--x D: Docker automatically removes container
    ```

## Setting This Up
*   **New Developer Checklist (Google Antigravity)**:
    1.  Locate your `mcp_config.json` file in your Antigravity installation folder.
    2.  Ensure Node.js is installed (required for `npx`-based MCP servers).
    3.  Install and run **Docker Desktop** (or equivalent Docker engine) for the `github-mcp-server`.
    4.  Create a Classic Personal Access Token (PAT) on GitHub with `repo` scopes.
    5.  Update the `GITHUB_PERSONAL_ACCESS_TOKEN` environment variable in your `mcp_config.json` file with your PAT.
    6.  Verify local Postgres and Redis instances are running before instructing the AI to query them.
*   **AI Prompting**: Refer to the "MCP Server Integration" section in the `GEMINI.md` Global Rules context for specific usage instructions.