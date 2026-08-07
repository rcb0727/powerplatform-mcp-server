# Changelog

**Docs:** [Overview](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/README.md) · [Installation & Upgrading](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/INSTALL.md) · **Changelog** · [Report an issue](https://github.com/rcb0727/powerplatform-mcp-server/issues)

> **Upgrading?** Quit your AI clients first so no `powerplatform-mcp-server` process is running, then `npm install -g powerplatform-mcp-server@latest`. Details: [Updating safely](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/INSTALL.md#updating).

## [1.0.6] - 2026-08-07

- **Power Pages permission errors now tell the truth.** The management API reports a missing user role as a 401 with "User doesn't have required permissions" (D004) — the tools used to swallow that body and say "Authentication failed," pointing at sign-in when the real fix is an environment admin granting a Power Pages role. The error now names the actual problem and the fix.
- **"InvalidApiVersion" now says what it almost always means: your install is outdated.** When Microsoft rejects a call over a retired API version, the error names your installed version and gives the exact update command, instead of reading like a product bug.
- **`--setup --client skip` works as documented.** The wizard always supported "skip" as a pre-selected answer for the AI-app step, but the CLI rejected the word as an unknown client and showed the menu anyway. Scripted and unattended setups can now skip client wiring cleanly.

## [1.0.5] - 2026-08-06

- **You'll hear about new versions in the chat.** When a newer release is on npm, the first tool response of a session carries a one-line note — current version, new version, and the exact update command for how you installed it — then the session stays quiet. `PA_MCP_UPDATE_NOTICE=0` turns it off. The check is the existing non-blocking startup check; no tool call ever waits on the npm registry.
- **Switch environments without leaving the chat — `switch_environment` (227 → 228 tools).** Working across dev/test/prod no longer means re-running `--setup`: ask to switch by environment name or ID (see `list_environments`) and the session re-points every tool — including Dataverse, whose per-environment org URL is re-resolved automatically. The switch is **session-only by design**: nothing is written to your configuration, restarting the server returns to your default, and access is always limited to what your Azure CLI account already has in the target environment. While switched, every destructive tool's output names the active environment, so "delete that flow" can never quietly land in the wrong place.
- **better-sqlite3 12 → 13** (schema cache engine, SQLite 3.53). Held back until 13.0.2 fixed the packaging bug that made npm compile from source on every install (upstream #1503); 13.0.2 ships prebuilt binaries for all supported platforms and installs in under a second with no compiler needed. No schema or API changes — the cache file carries over as-is.

## [1.0.4] - 2026-08-04

- **Dataverse depth: 216 → 227 tools.** Eleven new tools close the gaps between "CRUD on rows" and "work with Dataverse like it's a real database":
  - **`execute_fetchxml`** — aggregates, grouping, and link-entity joins that OData can't express ("opportunities closed above $50k this quarter, grouped by owner").
  - **`get_dataverse_option_set` / `get_dataverse_relationships`** — the anti-hallucination pair: choice columns' label↔integer mappings and relationship schema names are the two things an AI cannot guess, and guessing them wrong writes bad data silently or 404s.
  - **`create_dataverse_column` / `create_dataverse_lookup_column`** — schema write (string, memo, integer, decimal, money, datetime, boolean, picklist; lookups create the N:1 relationship in the same call). Confirm-gated; reminds you to publish.
  - **`batch_dataverse_operations`** — up to 100 operations in one `$batch`, with optional all-or-nothing changesets; batches containing deletes are confirm-gated. The right tool for "add every approved row from this spreadsheet."
  - **`upsert_dataverse_row`** (alternate-key create-or-update, reports which happened), **`assign_dataverse_row`** (ownership to a user or team), **`associate_dataverse_rows` / `disassociate_dataverse_rows`** (N:N and 1:N links), **`search_dataverse`** (full-text relevance search across tables).
- **Auth errors now name the real cause.** When Microsoft refuses to renew the Azure CLI's session, the error says why instead of a blanket "not signed in": your organization's sign-in frequency policy expired the session on schedule (recurring by design — not a bug), the sign-in was revoked by a password change or security event, it expired after 90 days without use, or Conditional Access is blocking the device/network outright (that one points at IT, since signing in again won't fix it).
- **Dataverse errors now carry a category** — `[PERMISSIONS]`, `[SCHEMA_MISMATCH]`, or `[ENV_LIMITATION]` — so the AI knows whether to fix a name, ask an admin, or stop retrying something the environment has disabled, instead of hammering an unwinnable call.

## [1.0.3] - 2026-07-29

- **License: MIT → Community License 1.0.** Free to use, study, modify, and share — including at work. One rule: it stays free. Selling or otherwise monetizing the software, or any fork of it, is not permitted, and every shared copy carries the same terms. Versions published before this change remain MIT.

## [1.0.2] - 2026-07-28

- **The npm page is readable now.** The package README is the streamlined landing page (pitch, install, Azure CLI auth, capability grid, security, architecture) instead of the 700-line wall; the full 216-tool reference generates into `TOOLS.md` (shipped in the package and on [GitHub](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/TOOLS.md)). `docs:tools`/`docs:check` now maintain the README grid, TOOLS.md, and CLAUDE.md together, so none of them can drift.

## [1.0.1] - 2026-07-28

- **MCP server key renamed to `powerplatform`.** Client configs written by `--setup`/`--client` and tool prefixes (`mcp__powerplatform__*`) now match the package identity, and a side-by-side install with `powerautomate-mcp` no longer collides on the server key. If you installed 1.0.0, re-run `powerplatform-mcp-server --client <your app>` to update the entry.

## [1.0.0] - 2026-07-28

First release of **powerplatform-mcp-server** — all 216 Power Platform tools, authenticated entirely through the Azure CLI. Derived from `powerautomate-mcp` 0.16.2; version history below this entry is inherited from that lineage.

- **Authentication is the Azure CLI, full stop.** There is no Microsoft Entra app registration, no admin-consent URL, and no token cache owned by this server (~3,700 lines of MSAL-era machinery removed). Tokens come from `az account get-access-token` per resource, riding the CLI's signed-in session. `--setup` is five steps (az check/login → tool selection → environment → config → AI-client wiring); `--login` wraps `az login`; the in-chat `sign_in` tool relays az's own device-code flow so no terminal is ever required.
- **Own identity end to end.** Package, binary, and config directory are all `powerplatform-mcp-server`, so it installs and runs side-by-side with `powerautomate-mcp` without touching its configuration.

---

*This project is derived from the `powerautomate-mcp` lineage at version 0.16.2. For the history of the 216 tools inherited by 1.0.0, see the [upstream changelog](https://github.com/rcb0727/powerplatform-mcp-docs/blob/main/CHANGELOG.md).*
