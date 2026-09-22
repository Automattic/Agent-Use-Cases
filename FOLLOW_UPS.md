# Follow-ups

Deferred items — v2 plans, intentional scope cuts, and discoveries made during development.

Never annotate `DESIGN.md`, code, or commit messages with "planned for v2" or "add later". Put it here instead. `DESIGN.md` describes what IS; `ROADMAP.md` tracks shaped features in flight; this file tracks deferred items waiting for a future re-shape.

## v2 / Deferred

- **Codex compatibility.** `claude-code-plugins` is dual-host: Codex packaging (`.agents/plugins/marketplace.json`, per-plugin `.codex-plugin/plugin.json`, `codex-skills/`) is generated from the Claude Code entries by a script and checked in CI. The same shape could apply here, but the skills are MCP-heavy (`allowed-tools` with both `mcp__wpcom__*` prefixes, the `authenticate` handshake), so Codex support needs an MCP story first, not just adapter generation. Note that Codex treats `.agents/` as its namespace; `.agents/plugins/` would sit beside the scaffold's `.agents/decisions/` and friends with no path overlap.
- **Registry parity check.** Nothing verifies that every `skills/<name>/SKILL.md` on disk is declared in its plugin's `skills` array in `marketplace.json`, and an undeclared skill is disabled silently. Neither this repo nor `claude-code-plugins` tests it. A ~15-line script would, and could be the first CI job here. Deferred: not worth the machinery with one plugin.
