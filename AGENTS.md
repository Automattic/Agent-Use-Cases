# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository. It is auto-loaded by Codex (and by Claude Code via the `CLAUDE.md` shim) at the start of any session in this directory.

## Project

**Agent-Use-Cases** — A Claude Code plugin marketplace of WordPress agent use cases

Each plugin here teaches an agent to run one real-world job on stock WordPress. The premise is that WordPress already ships the primitives a surprising number of applications need — dated posts, hierarchical taxonomies, tags, media attachments, user roles, private visibility, pages — and that what is usually missing is not code but convention: a schema, a naming discipline, and a procedure. A plugin in this repo carries that convention as skills, and carries nothing else.

**Knowledge, not code.** No PHP, no custom post types, no companion WordPress plugin. Every WordPress read and write goes through the WordPress.com MCP server; the only other tool a skill uses is Bash, to open the OAuth URL. The content model uses core primitives only — posts, pages, categories, tags, media, roles, post status — but the platform is WordPress.com, not stock WordPress: the MCP endpoint is WordPress.com-only, and some features a use case leans on, such as private *site* visibility, are WordPress.com additions that self-hosted core lacks. The boundary that matters is what the MCP facades expose on a WordPress.com site. That is narrower than core in places (no post meta, no revisions, no comment settings; `describe` is the source of truth), and when the MCP cannot do something the job needs, a skill says so and hands the user to the WordPress.com web UI rather than pretending. If a use case needs a custom post type, or anything the facades do not expose, it is not an agent use case — it is a WordPress plugin, and it belongs somewhere else.

The repo is the marketplace. `.claude-plugin/marketplace.json` declares every plugin and every skill path inline; `plugins/<name>/` holds one use case, with a `SKILL.md` per skill under `skills/` and bulk material in each skill's `references/`. Two silent-failure modes are worth knowing cold. A skill directory that exists on disk but is missing from its plugin's `skills` array in `marketplace.json` is disabled with no error anywhere — it simply never loads. And an MCP facade parameter the server does not recognize can be dropped without complaint, so never hardcode a facade's parameters: `describe` the operation at runtime, then read every write back, because a success response is not evidence that the write landed.

## Repository Visibility

**Agent-Use-Cases is public/open-source.** All tracked files, commits, PRs, issues, and durable agent artifacts must be public-safe and useful to contributors without private access.

- Do not include private company links, private tracker IDs, private Slack channels, customer data, or internal-only process details in tracked artifacts.
- Keep public issues and PRs self-contained. If private context informed the work, summarize the relevant technical facts without exposing the private source.
- Private local overlays belong in `.agents/project-management.local.md` or `.agents/private/`; both are ignored by the scaffold gitignore.
- Before promoting scratchpad notes into tracked docs, strip private context and verify the result stands on public information.

## Where to find what

<!-- Every row below points at a file that exists. Add a row when a new durable surface appears; remove a row when a file goes away. -->

| Surface | Audience | Contents |
|---------|----------|----------|
| `DESIGN.md` | Human + Agent | Architecture, **Invariants**, current-state source of truth. **If code contradicts DESIGN.md, DESIGN wins (or DESIGN is updated).** |
| `README.md` | Human + Agent | Project intro for visitors |
| `plugins/<name>/CHANGELOG.md` | Human + Agent | That plugin's user-visible changes ([Keep a Changelog](https://keepachangelog.com/en/1.1.0/)). There is no root changelog. |
| `ROADMAP.md` | Human + Agent | Shaped / in-flight features — "what next" for autonomous agents |
| `IDEAS.md` | Human + Agent | Pre-decision sparks; promote to `ROADMAP.md` when shaped |
| `FOLLOW_UPS.md` | Human + Agent | Deferred items |
| `./docs/` | Human + Agent | Reference: architecture deep-dives, runbooks, glossary, API refs |
| `.agents/project-management.md` | Human + Agent | Project-management system of record, issue workflow, tracker conventions |
| `.agents/decisions/` | Human + Agent | Durable decision records: rationale, alternatives, consequences, supersession |
| `.agents/reference/` | Agent-first | Long-tail: code-shape patterns, agent operational protocols |
| `.agents/scratchpad/` | Agent-first, gitignored | Working-doc area (designs, plans, reviews) — dies on fresh clone |

## Protocols

### Session start

1. Read this `AGENTS.md` (you're reading it now — auto-loaded).
2. Read `DESIGN.md` — Invariants + Architecture sections.
3. Read `.agents/project-management.md` before working on issues, tracker state, roadmap changes, or PR coordination.
4. For autonomous work, read `ROADMAP.md` to find "what next": the first unchecked step of the first feature under "In progress", else the top "Planned" feature whose dependencies are met.
5. If the task asks "why", involves architecture, or may change project direction, scan `DESIGN.md`'s Decision index and relevant records in `.agents/decisions/`.
6. If your work will produce multiple artifacts (design + plan + review + stress-test), open `.agents/scratchpad/sessions/YYYY-MM-DD-{slug}/` and copy from `.agents/scratchpad/TEMPLATE.md` for frontmatter.

### Units of work

Use these boundaries when deciding where information belongs:

| Unit | Meaning | Durable surface |
|------|---------|-----------------|
| Project | The whole repository and its operating rules | `AGENTS.md`, `DESIGN.md`, `.agents/reference/` |
| Project management | Tracker source of truth, issue workflow, public/private coordination rules | `.agents/project-management.md` |
| Roadmap feature | Human-shaped unit of autonomous work | `ROADMAP.md` |
| Work session | One agent run or multi-agent bundle | `.agents/scratchpad/sessions/YYYY-MM-DD-{slug}/` |
| Decision | A choice among meaningful alternatives | `.agents/decisions/YYYY-MM-DD-{slug}.md` |
| Commit / PR | Atomic version-control unit | Git history, PR body, updated docs |
| Release | User-visible shipped change set, per plugin | `plugins/<name>/CHANGELOG.md` |

### Working with progress discipline

Graduate keepers from working artifacts to versioned project docs *as work progresses*, not at session end. There is no signal that a session is "closing" — the LLM doesn't know when the conversation will end. Make the institutional record durable through commits and doc updates throughout the work.

When you reach any of these milestones, graduate immediately — same commit as the code change:

- **Architecture changed?** → update `DESIGN.md` (Architecture and/or Invariants).
- **Introduced an invariant** (property that must always hold)? → add to `DESIGN.md` Invariants.
- **Chose among meaningful alternatives?** → add or update `.agents/decisions/YYYY-MM-DD-<slug>.md`; link it from `DESIGN.md` if it affects current architecture or invariants.
- **Established or changed a code-shape pattern?** → add or update `.agents/reference/patterns/<area>.md`.
- **Shipped user-visible behavior?** → append to that plugin's `CHANGELOG.md` `[Unreleased]`.
- **Moved roadmap state?** → update `ROADMAP.md` so "what next" stays accurate.
- **Code changes?** → Conventional Commits + body explaining [Context] → [Problem] → [Solution]. Summarize linked resources inline (links break).

What's left in `.agents/scratchpad/sessions/{slug}/` after graduation is, by definition, transient. It dies on fresh clone — by design. The institutional record lives in commits, `DESIGN.md`, `.agents/project-management.md`, `.agents/decisions/`, `.agents/reference/`, `ROADMAP.md`, and each plugin's `CHANGELOG.md`.

### Decision records

`.agents/decisions/` is tracked human-and-agent project memory. Create or update a decision record when the project chooses among meaningful alternatives and the outcome affects architecture, invariants, roadmap direction, public behavior, persistence shape, security posture, operational model, or a canonical implementation pattern future agents will copy.

Do not write a decision record for routine implementation details. Use `.agents/decisions/TEMPLATE.md`, name records `YYYY-MM-DD-<slug>.md`, and mark superseded records instead of deleting or rewriting history.

### Multi-agent work

When multiple agents work in parallel:

- The orchestrating agent owns the final synthesis and durable doc updates.
- Subagents write scoped artifacts with `target`, `status`, and `reconciles` frontmatter when useful.
- Parallel code edits must have disjoint ownership scopes.
- Before completion, reconcile contradictory findings into one durable decision, design update, or follow-up.

### Before finishing

Before claiming work is complete, check whether this change requires updates to:

- `DESIGN.md` — architecture, behavior, or invariants changed.
- `.agents/decisions/` — a meaningful choice was made among alternatives.
- `.agents/reference/patterns/` — a repeated or canonical code shape changed.
- `.agents/project-management.md` — tracker source of truth, issue workflow, or public/private coordination policy changed.
- the plugin's `CHANGELOG.md` — that plugin's user-visible behavior changed.
- `ROADMAP.md` — work moved between Planned, In progress, and Shipped.

### Commit discipline

- One logical change per commit. If your subject contains "and", split.
- Conventional Commits: `feat:`, `fix:`, `refactor:`, `perf:`, `chore:`, `docs:`, etc.
- Body: [Context] → [Problem] → [Solution].
- Reference issues with `Refs <id>` in commits, `Closes <id>` in PRs.

## Scratchpad

`.agents/scratchpad/` is your working-doc area. Gitignored — institutional knowledge graduates via the progress discipline above; what's left here dies on fresh clone.

Two tiers, plus free-form:

| Tier | When | Path |
|------|------|------|
| `sessions/` | Multi-artifact coherent work (design + plan + review + stress-test in one bundle) | `.agents/scratchpad/sessions/YYYY-MM-DD-{slug}/` |
| `journal/` | Single-artifact loose notes (quick triage, daily notes, isolated analyses) | `.agents/scratchpad/journal/YYYY-MM-DD-{slug}.md` |
| free-form | Anything that doesn't fit either tier (raw data dumps, ad-hoc subdirs, temporary collections) | `.agents/scratchpad/<whatever-makes-sense>` |

Required frontmatter on every `.md` artifact (copy from `.agents/scratchpad/TEMPLATE.md`):

- `session: YYYY-MM-DD-{slug}` — for sessions/: matches folder name; for journal/free-form: artifact's slug
- `type: design | plan | review | stress-test | report | exploration | analysis | data | readme`
- `by: claude | codex | gemini | subagent:<name>`
- `created: YYYY-MM-DD HH:MM`

Optional fields when informative: `model`, `tool`, `target`, `reconciles`, `promoted_to`, `related_decision`, `last_updated`, `status: draft | final | superseded`, `superseded_by`.

Non-`.md` files (CSVs, images, scripts, raw output) are exempt from frontmatter.
