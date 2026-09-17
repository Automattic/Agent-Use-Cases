# Changelog

All notable changes to HealthyPress are documented here.

## 0.1.0 — 2026-09-17

Initial release.

- `/healthypress:setup` — privacy-gated site setup and idempotent privacy audit: launch, set
  Private, verify, discourage search engines, disable comments and pingbacks, disable newsletter
  email and verify zero subscribers, neutral site title, timezone, taxonomy, pages, privacy report.
  Stops hard if the site can't be made Private.
- `/healthypress:log` — record one health event as a private, correctly dated, categorized, and
  tagged post, with a full read-back before saving and a refresh of only the affected pages.
- `/healthypress:backfill` — guided history intake, era by era and system by system, with
  checkpoints, a resumable captured list, a declined-topics list, and the free-plan 30-day MCP
  warning before any work.
- `/healthypress:share` — care team access on private sites only: Editor (full read, also edit) or
  Viewer (read-only, published pages only), with the tradeoff stated in plain language.
- `/healthypress:review` — read-only factual report plus hygiene audit and privacy check.
- Skills: `health-content-model`, `wpcom-mcp-operations`, `health-intake-interview`,
  `health-summary-pages`.
- `.mcp.json` declaring the WordPress.com MCP server (`wpcom`, HTTP transport, native OAuth).

### Known unverified behavior

Seven MCP behaviors this design depends on are documented but not yet verified against a live site;
three of them are load-bearing. See `docs/wpcom-mcp-notes.md` in the repository root. The commands
handle each by discovering schemas at runtime and reading writes back rather than trusting them.
