# Installation Guide

**Docs:** [Overview](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/README.md) · **Installation & Upgrading** · [Changelog](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/CHANGELOG.md) · [Report an issue](https://github.com/rcb0727/powerplatform-mcp-server/issues)

This MCP server uses the **stdio** transport and works with any MCP-compatible AI client.

**On this page:** [Prerequisites](#prerequisites) · [Install](#install-from-npm) · [Updating](#updating) · [First-Time Setup](#first-time-setup) · [Configure Your AI Client](#configure-your-ai-client) · [Enterprise Tenants](#enterprise-tenants-with-strict-consent-policies)

## Prerequisites

- Node.js 22.19+
- Microsoft 365 work account with Power Automate access
- Azure CLI, signed in with `az login` (setup checks this for you)
- **Linux only**: libsecret for secure token storage
  ```bash
  # Ubuntu/Debian runtime
  sudo apt-get install libsecret-1-0 gnome-keyring

  # Fedora/RHEL runtime
  sudo dnf install libsecret gnome-keyring
  ```

  If you build native modules from source, also install the development package:

  ```bash
  # Ubuntu/Debian
  sudo apt-get install libsecret-1-dev

  # Fedora/RHEL
  sudo dnf install libsecret-devel
  ```

  If setup fails with `libsecret-1.so.0: cannot open shared object file`, the
  runtime package above is missing.

## Install from npm

```bash
npm install -g powerplatform-mcp-server
```

## Updating

> **Stop running MCP instances before you upgrade.** Quit your AI clients (Claude Desktop, Cursor, VS Code, etc.) and stop any `powerplatform-mcp-server --http` servers first. A running process keeps the old version loaded in memory, and on Windows it can hold file locks in npm's global directory — causing `EBUSY`/`EPERM` errors and a half-upgraded install that's confusing to debug.

1. Quit your AI clients and stop any `--http` server instances
2. Upgrade:
   ```bash
   npm install -g powerplatform-mcp-server@latest
   ```
3. Verify the new version:
   ```bash
   powerplatform-mcp-server --version
   ```
4. Relaunch your AI client — it starts the new version automatically. Claude Code picks it up on the next session.

`powerplatform-mcp-server --update` does the npm upgrade for you — the same rule applies: close clients first.

See the [Changelog](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/CHANGELOG.md) for what's new and any version-specific upgrade notes.

## Rolling back

Every published version stays on npm permanently, so if an upgrade misbehaves you can return to the version you were on with one command. Same rule as updating: quit your AI clients first.

1. Pick the version: the [Changelog](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/CHANGELOG.md) lists every release and what changed, or run `npm view powerplatform-mcp-server versions`.
2. Install it by number:
   ```bash
   npm install -g powerplatform-mcp-server@1.0.2
   ```
3. Verify, then relaunch your AI client:
   ```bash
   powerplatform-mcp-server --version
   ```

A rollback changes only the installed package — your configuration and Azure CLI sign-in are untouched, and there's nothing else to undo (no service, no database, no migrations). When the issue is resolved, `npm install -g powerplatform-mcp-server@latest` moves you forward again.

**Organizations:** deployment scripts can pin an exact version (`@1.0.2` instead of `@latest`) so every machine runs the version IT approved — rolling back then means pushing the deployment again with the prior pin. If a release forced you to roll back, please [report it](https://github.com/rcb0727/powerplatform-mcp-server/issues) so it gets fixed for everyone.

## First-Time Setup

```bash
powerplatform-mcp-server --setup
```

This interactive wizard will:
1. Check the Azure CLI is installed and signed in (running `az login` if needed)
2. Let you choose a tool set: all tools, Power Automate only, connectors/Power Apps, Dataverse, Power Pages, or custom
3. Discover your Power Automate environments
4. Create the configuration file

## Configure Your AI Client

- [Claude Desktop](#claude-desktop)
- [Claude Code (CLI)](#claude-code-cli)
- [VS Code (GitHub Copilot)](#vs-code-github-copilot)
- [Cursor](#cursor)
- [Google Gemini CLI](#google-gemini-cli)
- [ChatGPT (OpenAI)](#chatgpt-openai)
- [Other MCP Clients](#other-mcp-clients)

---

### Claude Desktop

Add to your Claude Desktop config file:

| OS | Config Path |
|----|-------------|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| Linux | `~/.config/Claude/claude_desktop_config.json` |

```json
{
  "mcpServers": {
    "powerautomate": {
      "command": "powerplatform-mcp-server"
    }
  }
}
```

Restart Claude Desktop. The Power Automate tools will appear automatically.

---

### Claude Code (CLI)

Add the server from your terminal:

```bash
claude mcp add powerautomate -- powerplatform-mcp-server
```

Or add it to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "powerautomate": {
      "command": "powerplatform-mcp-server"
    }
  }
}
```

---

### VS Code (GitHub Copilot)

Add to your workspace `.vscode/mcp.json` (or user-level `mcp.json`):

```json
{
  "servers": {
    "powerautomate": {
      "type": "stdio",
      "command": "powerplatform-mcp-server"
    }
  }
}
```

Or open the Command Palette (`Ctrl+Shift+P`) and run **MCP: Add Server**.

---

### Cursor

Add to `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project):

```json
{
  "mcpServers": {
    "powerautomate": {
      "command": "powerplatform-mcp-server"
    }
  }
}
```

Restart Cursor to pick up the new server.

---

### Google Gemini CLI

Add to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "powerautomate": {
      "command": "powerplatform-mcp-server"
    }
  }
}
```

---

### ChatGPT (OpenAI)

ChatGPT requires a remote HTTPS MCP endpoint. This server supports that via the `--http` flag.

**1. Start the server in HTTP mode:**

```bash
powerplatform-mcp-server --http --port 3000
```

This starts the MCP server with Streamable HTTP transport at `http://localhost:3000/mcp`.

**2. Expose via tunnel (pick one):**

```bash
# ngrok
ngrok http 3000

# Cloudflare Tunnel
cloudflared tunnel --url http://localhost:3000
```

Copy the HTTPS URL (e.g. `https://abc123.ngrok-free.app`).

**3. Add to ChatGPT:**

1. Open [ChatGPT](https://chat.openai.com) → Settings → MCP Servers
2. Click **Add Server**
3. Enter URL: `https://your-tunnel-url.ngrok-free.app/mcp`
4. Save

The Power Automate tools will appear in ChatGPT's tool picker.

> **Security**: The tunnel exposes your local MCP server to the internet. Only run it while actively using ChatGPT. Stop the server and tunnel when done.

---

### Other MCP Clients

**Stdio transport** (default — for local clients):

```bash
powerplatform-mcp-server
```

**HTTP transport** (for remote/web clients):

```bash
powerplatform-mcp-server --http --port 3000
```

This starts a Streamable HTTP server on `POST /mcp` compatible with any MCP client that supports the HTTP transport.


## Enterprise Tenants with Strict Consent Policies

If your tenant requires admin consent for all applications:

1. **Add only the API permissions for the feature set you selected** in Microsoft Entra:
   - Flow Service (`7df0a125-d3be-4c96-aa54-591f83ff541c`): `Flows.Read.All`, `Flows.Manage.All`, `Activity.Read.All`, `Approvals.Manage.All`
   - Optional SharePoint/Excel helpers: Microsoft Graph `User.Read`, `Sites.ReadWrite.All`, `Files.ReadWrite.All`
   - Optional connections/connectors/Power Apps: PowerApps Service (`475226c6-020e-4fb2-8a90-7a972cbfc1d4`) `User`
   - Optional Dataverse/admin/Power Pages config: BAP Admin API (`0e0bf3cc-3078-4fd4-9ef3-cb6dc0245b10`) `user_impersonation`
   - Optional Dataverse/Power Pages config: Dynamics CRM (`00000007-0000-0000-c000-000000000000`) `user_impersonation`

2. **Grant admin consent** for the selected permissions via:
   ```
   https://login.microsoftonline.com/{tenant-id}/adminconsent?client_id={your-client-id}
   ```

3. Re-run `powerplatform-mcp-server --setup` to authenticate.

Skipped feature scopes are recorded in `features.enabled`; their tools are hidden and their auth checks are skipped. Without the PowerApps Service permission, connector and Power Apps tools are unavailable. Without the BAP Admin API permission, admin tools and Dataverse URL auto-discovery are unavailable.

## CLI reference



## Reducing approval prompts

216 tools means a lot of permission prompts if you approve each one. The
annotations this server ships let you allow the safe ones and keep the gate
where it matters — **100 tools are read-only, 39 are destructive, 77 are
ordinary writes.**

Allow the reads, keep prompts for everything that changes state:

```jsonc
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "mcp__powerplatform__list_flows",
      "mcp__powerplatform__get_flow",
      "mcp__powerplatform__get_runs",
      "mcp__powerplatform__diagnose_flow",
      "mcp__powerplatform__query_dataverse_rows",
      "mcp__powerplatform__list_connections",
      "mcp__powerplatform__test_connection"
    ]
  }
}
```

Add whichever other `list_*` / `get_*` / `search_*` / `diagnose_*` tools you use
often — every one of them is annotated `readOnlyHint: true` and cannot change
anything.

Allowing the whole server (`"mcp__powerplatform"`) also works, but it
auto-approves `delete_flow`, `reset_environment`, and 37 other irreversible
operations. Don't, unless you are working in a throwaway environment.
