# Agent-Use-Cases — Design

> A Claude Code plugin marketplace of WordPress agent use cases

This document is the source of truth for the current architecture and behavior of Agent-Use-Cases. If something in the code contradicts this document, this document wins. If this document is wrong, update it — don't silently diverge. Keep it updated whenever adding, changing, or removing features. Use `.agents/decisions/` for durable rationale about why major choices were made.

This document is deliberately small. There is one plugin so far, and most of what could be said about "how a plugin is shaped" would be a description of that plugin. Only what holds independently of any one use case is written down as a rule; see [Promotion rule](#promotion-rule).

## Overview

A marketplace of Claude Code plugins. Each plugin is one real-world job done on a WordPress.com site through an agent, with no code deployed to the site. WordPress already ships the primitives most such jobs need — dated posts, hierarchical taxonomies, tags, media, user roles, private visibility, pages. What is usually missing is convention: a schema, a naming discipline, a procedure. A plugin here carries that convention as skills, and an agent applies it through the WordPress.com MCP server.

Two audiences: people who want a job done and install one plugin; contributors who add a job and work in this repo.

## Goals

- A use case runs end-to-end with only Claude Code and a WordPress.com account. Nothing is installed on the site.
- A plugin is self-sufficient: its skills alone let an agent set the job up, operate it, and explain its limits.
- Every plugin tells the user what it cannot do before they commit real data to it.
- Writes are safe by default: confirmed with the user, and read back to verify.

## Non-goals

- Not WordPress plugins. No PHP, no custom post types, no companion code on the site. A job that needs those is a WordPress plugin and belongs elsewhere.
- Not self-hosted WordPress. The MCP endpoint is WordPress.com's, and some features a job may rely on (private *site* visibility, for one) exist only there.
- Not a framework. Plugins share no code, because there is no code. Whether they should share knowledge is an open question below.
- Not a workaround for what the MCP cannot do. A skill says so and hands the user to the WordPress.com UI.

## Architecture

Five layers, top to bottom:

1. `.claude-plugin/marketplace.json` — declares every plugin and, inline, every skill path. The marketplace is the repo.
2. `plugins/<name>/` — one use case: `README.md`, `CHANGELOG.md`, `.mcp.json`, and `skills/`.
3. `skills/<skill>/SKILL.md` — a workflow the user invokes, or reference knowledge that loads behind one. Bulk material sits in the skill's `references/`.
4. The WordPress.com MCP server at `https://public-api.wordpress.com/wpcom/v2/mcp/v1` — a handful of facade tools, each selecting behavior with an `operation` and an `action`; `list` and `describe` expose the live schema.
5. A WordPress.com site — the substrate, addressed only through core primitives: posts, pages, categories, tags, media, roles, post status, site settings.

Nothing in the repo executes. The agent is the runtime; the skills are its instructions; the MCP is its only hand on the site.

## Invariants

Each of these holds today and is easy to break by accident. Each follows from the platform or from a non-goal, not from how any one plugin happens to be built.

1. **A plugin contains only Markdown and `.mcp.json`.** No executable code of any kind.
2. **Every site read and write goes through the WordPress.com MCP.** Bash is permitted only to open the OAuth URL in the user's browser.
3. **Facade parameters are never hardcoded.** A skill `describe`s an operation before its first use in a session and passes exactly what the live schema names. A parameter absent from `describe` does not exist.
4. **Every write is read back.** A success response is not evidence that the write landed; the MCP has returned `success: true` for writes that changed nothing.
5. **Every `allowed-tools` that names a `wpcom` tool lists both the `mcp__wpcom__*` and `mcp__plugin_<name>_wpcom__*` forms.** A single-prefix skill breaks depending on how the server was installed. Skill registration is not an invariant: every `SKILL.md` under `skills/` loads whether or not the `skills` array lists it (verified at v2.1.278, see `docs/plugin-mechanics.md`).
6. **A change to a plugin's behavior bumps that plugin's `version` in `marketplace.json` and adds an entry to that plugin's `CHANGELOG.md`.** There is no root changelog: the plugin is the unit a user installs, so it is the unit a version and a changelog describe. The marketplace manifest carries no version of its own, and the plugin entry's `version` is the sole update signal — see `AGENTS.md` § Versioning and releases.

### Promotion rule

A property becomes an invariant only when it follows directly from a non-goal or from the platform, or when it holds in at least two plugins and a decision record in `.agents/decisions/` explains why. Until then it is a pattern, documented with the plugin that exhibits it. This is a guard against the first plugin's choices becoming the repo's law by default.

## Decision index

- [2026-09-22 — Per-plugin versioning, and the marketplace manifest carries no version](.agents/decisions/2026-09-22-per-plugin-versioning-and-releases.md). Affects invariant 6 and the release model.

Other candidates, each currently living only in `plugins/healthypress` prose: knowledge-not-code; WordPress.com-only; posts-as-the-whole-record; setup-always-creates-a-new-site. The first two are repo-level and will get records when they are next questioned; the last two are healthypress decisions unless a second plugin makes the same choice.

## Open questions

- The generic MCP mechanics — facade pattern, auth handshake, write and read gotchas — live inside `plugins/healthypress/skills/health-record`. That is repo-level knowledge in one plugin. The second plugin will either copy it or force a decision on where shared knowledge lives.
- There is no way to test a skill against the live MCP. Facade drift is found by users.
- The second use case should be deliberately unlike the first — public rather than private, current-state rather than event-logged, low-stakes rather than sensitive — so that healthypress conventions get tested rather than confirmed by repetition.
