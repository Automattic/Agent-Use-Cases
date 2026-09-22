# .agents/reference/

Agent-only long-tail reference documentation. Tracked in git — survives across clones — but written
specifically for AI agents working in this repository, not for human contributors.

## Audience and placement

| Surface | Audience | Use it for |
|---------|----------|------------|
| `AGENTS.md` (and nested) | Agent | High-density entry-point content. Auto-loaded at session start. Keep it lean. |
| `.agents/project-management.md` | Human + Agent | Tracker source of truth, issue workflow, public/private coordination rules. |
| `.agents/decisions/` | Human + Agent | Durable decision records: rationale, alternatives, consequences, supersession. |
| `.agents/reference/` (this directory) | Agent | Long-tail detail too big or too domain-specific to inline in AGENTS.md. |
| `./docs/` | Human + Agent | Project reference (architecture deep-dives, runbooks, glossary, API). |

## What goes here

- **Code-shape pattern catalogs** under `patterns/<area>.md` — "to add a new payment handler, copy this
  shape; canonical mocking pattern for tests; how we structure a hook handler". Cold-reading agents copy
  the first pattern they see, so making the canonical shape explicit prevents drift.
- **Agent operational protocols beyond AGENTS.md** — long-tail rituals, edge-case handling, project-specific
  agent constraints that don't fit inline.
- **Multi-agent coordination protocols** — only when multiple agents (Claude + Codex + Gemini) operate
  with non-trivial coordination needs.

## What does NOT go here

- **MCP / harness inventory.** Harness concerns (MCP servers, plugins, skills) live at the harness level
  (`.claude/settings.json`, MCP server configs), not in project documentation.
- **Architecture and current behavior.** Those go in `DESIGN.md` or `./docs/`.
- **Project-management systems, issue workflow, and public/private coordination policy.** Those go in `.agents/project-management.md`.
- **Decision rationale, alternatives, and supersession history.** Those go in `.agents/decisions/`.
- **Glossary and runbooks.** Those go in `./docs/` — they're useful to humans too.
- **Operational protocols that fit inline in AGENTS.md.** Keep them inline so they're auto-loaded.

## Adding a pattern

Pick a kebab-case slug per area. Suggested file structure:

```markdown
# <Area> patterns

## Pattern: <name>

**When:** Brief description of when to apply this pattern.

**Shape:**

```<lang>
// canonical code
```

**Notes:** Caveats, gotchas, or rationale.
```
