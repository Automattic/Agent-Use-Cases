# Follow-ups

Deferred items — v2 plans, intentional scope cuts, and discoveries made during development.

Never annotate `DESIGN.md`, code, or commit messages with "planned for v2" or "add later". Put it here instead. `DESIGN.md` describes what IS; `ROADMAP.md` tracks shaped features in flight; this file tracks deferred items waiting for a future re-shape.

## v2 / Deferred

- **Codex compatibility.** `claude-code-plugins` is dual-host: Codex packaging (`.agents/plugins/marketplace.json`, per-plugin `.codex-plugin/plugin.json`, `codex-skills/`) is generated from the Claude Code entries by a script and checked in CI. The same shape could apply here, but the skills are MCP-heavy (`allowed-tools` with both `mcp__wpcom__*` prefixes, the `authenticate` handshake), so Codex support needs an MCP story first, not just adapter generation. Note that Codex treats `.agents/` as its namespace; `.agents/plugins/` would sit beside the scaffold's `.agents/decisions/` and friends with no path overlap.
- **Registry parity check.** Dropped 2026-09-22. The premise was that an undeclared skill directory is silently disabled; it is not (every `SKILL.md` under `skills/` loads, see `docs/plugin-mechanics.md`). The remaining risk is the reverse, a stray directory shipping, and `claude plugin details` shows the loaded inventory, which is enough for now.
