# Changelog

**Docs:** [Overview](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/README.md) · [Installation & Upgrading](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/INSTALL.md) · **Changelog** · [Report an issue](https://github.com/rcb0727/powerplatform-mcp-server/issues)

> **Upgrading?** Quit your AI clients first so no `powerplatform-mcp-server` process is running, then `npm install -g powerplatform-mcp-server@latest`. Details: [Updating safely](https://github.com/rcb0727/powerplatform-mcp-server/blob/main/INSTALL.md#updating).

## [1.0.3] - 2026-07-29

- **License changed from MIT to the Free for the People License 1.0.** Free to use, study, modify, and share — it just stays free: no selling or monetizing the software or forks of it, and every shared copy carries the same terms. Versions 1.0.2 and earlier remain MIT.

## [1.0.2] - 2026-07-28

- Readable npm landing page; the full 216-tool reference generates into TOOLS.md.

## [1.0.1] - 2026-07-28

- MCP server key renamed to `powerplatform` — side-by-side installs with powerautomate-mcp no longer collide.

## [1.0.0] - 2026-07-28

First release of **powerplatform-mcp-server** — all 216 Power Platform tools, authenticated entirely through the Azure CLI. Derived from `powerautomate-mcp` 0.16.2; version history below this entry is inherited from that lineage.

- **Authentication is the Azure CLI, full stop.** There is no Microsoft Entra app registration, no admin-consent URL, and no token cache owned by this server (~3,700 lines of MSAL-era machinery removed). Tokens come from `az account get-access-token` per resource, riding the CLI's signed-in session. `--setup` is five steps (az check/login → tool selection → environment → config → AI-client wiring); `--login` wraps `az login`; the in-chat `sign_in` tool relays az's own device-code flow so no terminal is ever required.
- **Own identity end to end.** Package, binary, and config directory are all `powerplatform-mcp-server`, so it installs and runs side-by-side with `powerautomate-mcp` without touching its configuration.

---

*This project is derived from the `powerautomate-mcp` lineage at version 0.16.2. For the history of the 216 tools inherited by 1.0.0, see the [upstream changelog](https://github.com/rcb0727/powerplatform-mcp-docs/blob/main/CHANGELOG.md).*
