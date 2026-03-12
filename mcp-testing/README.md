# MCP Testing

This workspace is configured with several MCP (Model Context Protocol) servers for use with VS Code Copilot.

## MCP Servers

### Context Mode

- **Type:** stdio
- **Command:** `context-mode`
- **Purpose:** A context-saving sandbox that optimizes how other MCP tools use the conversation context window. It intercepts heavy tool output (e.g., large Playwright snapshots, API responses) and routes it through a local SQLite index — only a small summary enters the conversation. Also provides session continuity by tracking file edits, git operations, tasks, and errors, allowing the model to resume after context compaction.
- **Setup:** Requires `npm install -g context-mode`. Hooks are configured in `.github/hooks/context-mode.json`.

### Microsoft Learn

- **Type:** HTTP
- **URL:** `https://learn.microsoft.com/api/mcp`
- **Purpose:** Provides access to official Microsoft and Azure documentation. Supports searching docs, fetching full pages, and finding code samples from Microsoft Learn.

### Playwright

- **Type:** stdio
- **Command:** `npx @playwright/mcp@latest`
- **Purpose:** Browser automation via Playwright. Enables taking page snapshots, clicking elements, filling forms, navigating pages, and other browser interactions directly from the AI agent.

### Context7

- **Type:** HTTP
- **URL:** `https://mcp.context7.com/mcp`
- **Purpose:** Retrieves up-to-date documentation and code examples for any library. Useful for getting current API references that may be newer than the model's training data.
- **Auth:** Requires a Context7 API key (prompted on first use).

## Setup

1. Open this workspace in VS Code.
2. Ensure `context-mode` is installed globally: `npm install -g context-mode`.
3. Restart VS Code to activate all MCP servers.
4. The Context7 server will prompt for an API key on first use — get one at [context7.com](https://context7.com).
