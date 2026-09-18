# Conventions for agents working in this repo

This repo is a **Claude Code plugin marketplace of WordPress use cases**. Each plugin teaches an
agent to use stock WordPress as the substrate for a real-world job. The plugins carry knowledge —
prose, procedures, and schemas — not code.

Read `CONTRIBUTING.md` before adding or restructuring a plugin. This file is the short version of
what to watch for while editing.

## What lives where

```
.claude-plugin/marketplace.json   every plugin, skill, and command declared inline
plugins/<name>/                   one plugin = one use case
  README.md  CHANGELOG.md  .mcp.json
  commands/*.md                   user-invoked procedures
  skills/<skill>/SKILL.md         knowledge, ~100 lines, bulk in references/
```

## Rules that bite

**Register every path.** A skill directory or command file missing from `marketplace.json` is
silently disabled — no error, it just never loads. Adding a file is a two-file change: the file and
the manifest. Bump the plugin `version` and `metadata.version` too, and add a CHANGELOG entry.

**Core WordPress only.** No PHP, no custom post types, no custom taxonomies, no post meta, no
companion WordPress plugin. When core can't express something, document the limit rather than
working around it.

**Never hardcode an MCP facade schema.** The WordPress.com MCP uses facade tools taking an
`operation` plus `action: list` / `action: describe`, and the docs say schemas evolve and the live
`describe` response is the source of truth. Skills teach discovery. Don't copy parameter names out
of documentation and present them as a contract.

**Both tool-name prefixes in `allowed-tools`.** Plugin-scoped MCP servers get renamed, so every
command must list `mcp__wpcom__<facade>` **and** `mcp__plugin_<plugin>_wpcom__<facade>`.

**The `## Boundaries` block is duplicated on purpose.** It appears verbatim in every HealthyPress
`SKILL.md` and at the end of every HealthyPress command. Skill loading is probabilistic, so a
single-source guardrail is one that sometimes doesn't load. **Do not DRY it away, do not replace a
copy with a cross-reference, do not paraphrase a copy to fit its context.** If it changes, change
every copy in the same commit and verify they're byte-identical.

**`wpcom-mcp-operations` is copied between plugins, not shared.** Plugin sources can't share
directories. Copy it verbatim; don't fork or trim it.

## Editing prose here

The prose *is* the product; it's read by an agent at runtime, so it's specification, not
documentation.

- Rules as imperatives. Examples over explanation.
- Every failure branch in a command gets an explicit "…then STOP" and the text to say to the user.
- Gates go before writes, and are verified by reading state back, not by trusting a write's return.
- State limits plainly. If the honest answer for some users is "use something else", write that.
- Don't add hedging, interpretation, or advice to a plugin whose whole stance is that it doesn't
  interpret or advise.

## Verifying a change

1. `jq . .claude-plugin/marketplace.json`
2. Confirm every declared path exists (`.github/workflows/validate-marketplace.yml` does this).
3. Install from the local path and check `/help`, the skill list, and `/mcp`.
4. Exercise anything that writes **against a scratch site only** — never a real one.
