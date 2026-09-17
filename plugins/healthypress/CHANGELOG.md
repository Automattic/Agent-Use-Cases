# Changelog

All notable changes to HealthyPress are documented here.

## 0.1.2 — 2026-09-17

First live run of `/healthypress:setup` against a real WordPress.com site, which falsified the
plan's central privacy assumption.

### Fixed

- **The privacy gate no longer launches the site.** The plan asserted that a fresh site sits in
  Coming Soon and that Private only becomes available after launching, so setup ran
  launch → privatize → verify. Verified against a live site: a freshly provisioned site already
  reports `visibility: private`, `launch_status: unlaunched`, `is_private: true`. Launching is
  unnecessary and moves the site *toward* live — the command as shipped would have caused the exact
  exposure the gate exists to prevent. Setup now reads status, changes visibility only if it isn't
  already private, and never launches.
- Corrected the same launch-first ordering in `wpcom-mcp-operations`.

### Changed

- **Taxonomy creation in `/healthypress:setup` is now written against the verified API.** `parent`
  takes a numeric category ID, not a slug, so the five container categories must be created first
  and their IDs captured before any leaf. The step now reads existing terms by slug for idempotency,
  requests `include_fields` to keep responses small, takes one user approval for the whole batch
  (`user_confirmed` is per write and does not carry), and paces the writes in batches.
- The hardening settings are now applied as a single batched `settings.update` with one user
  approval, since `user_confirmed` is required per write operation.
- Setup now closes open registration, sets `Y-m-d` / `H:i` formats, and detects the timezone from
  the local system instead of asking.

### Added

- Honest gap reporting in the privacy report. Three things the plan assumed were settable are not:
  the **default post category** (no `default_category` field), **comments and pingbacks off** (no
  `default_comment_status` / `default_ping_status` — only "require login to comment"), and
  search-engine discouragement as its own setting (it's the `blog_public: 0` *public* state).
  The report now names these as gaps and points at wp-admin.
- A recorded false-success case: **`users_can_register: false` returns `success: true` with a
  `before/after` transition and does not persist.** Reproduced twice. The report shows the
  read-back value, never the write response.

### Notes

- `docs/wpcom-mcp-notes.md`: open questions 2 (privacy ordering) and 5 (`user_confirmed` semantics)
  are answered and moved to the verified table, along with nine other findings — 28 facade tools
  rather than ~8, subdomain derived from the title, front page must be published first, and the
  full `settings.update` field list. Questions 1, 3, 4, 6, and 7 remain open.

## 0.1.1 — 2026-09-17

First run against a live install, which corrected two assumptions.

### Changed

- **`/healthypress:setup` now performs the WordPress.com authorization itself.** An unauthenticated
  MCP server exposes only `authenticate` and `complete_authentication` — the facade tools are absent
  entirely, and Claude Code's native OAuth for HTTP servers does not complete the handshake. Setup
  now calls `authenticate`, opens the URL in the user's browser, and continues, instead of telling
  them to go configure `/mcp`. The same handshake was added to the other four commands and
  documented in `wpcom-mcp-operations`.
- **`/healthypress:setup` always creates a new site** when run without arguments. It no longer lists
  existing sites or asks the user to choose one — a dedicated site is what keeps health content out
  of places it shouldn't be, and the site picker was both friction and a footgun. Passing a `site`
  argument now means **audit mode**: re-harden and re-verify a site HealthyPress already set up,
  refusing to touch a site that has no HealthyPress structure on it. The site title is set to a
  neutral default rather than asked about.

### Added

- Disclosure, in the README and before the authorization prompt, that **the OAuth grant is
  account-wide** — sites, posts, media, taxonomy, users, stats, and notifications across the whole
  WordPress.com account — and is issued to a shared Claude Code OAuth client rather than to
  HealthyPress by name.

### Fixed

- `docs/wpcom-mcp-notes.md` listed native OAuth handling as a settled fact. It was wrong; both auth
  findings are now recorded as verified with the date and method.

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
