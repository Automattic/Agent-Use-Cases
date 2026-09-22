# 2026-09-22 - The plugin manifest owns the plugin's definition

Status: Accepted
Date: 2026-09-22
Deciders: Vlad Olaru, with Claude

## Context

Until now no plugin here shipped a `plugin.json`. Every component and the version were declared inline on the marketplace entry, and `DESIGN.md` described that as the architecture: "declares every plugin and, inline, every skill path."

Two things over one day showed the cost.

healthypress 0.2.0 failed to load for every installed user. The entry had been set to `strict: false` on the reasoning that, with no `plugin.json` in the repo, the entry must be the complete definition. It is not: Claude Code synthesizes a `plugin.json` into the installed copy from the entry, components and all. Under `strict: false` that synthesized manifest is a rival declaration, and the plugin fails with "conflicting manifests". 0.2.1 restored the default `strict: true` and the plugin loaded again, but the shape that made the mistake reachable was still there.

Separately, the release procedure written in the same session ended in a `claude plugin tag` step that could not run, because that command requires `plugins/<name>/.claude-plugin/plugin.json`. Chasing that led here. The tagging step has since been dropped on its own merits — a `{name}--v{version}` tag exists only to resolve another plugin's declared dependency, and nothing here declares one — so it is not a reason for this decision, only how the missing manifest was noticed.

## Decision

Every plugin ships `plugins/<name>/.claude-plugin/plugin.json`, and that manifest is the authority on what the plugin is: its `version`, its metadata, and every component it declares.

The marketplace entry becomes a catalogue row: `name`, `source`, `description`. It declares no components and carries no version. `strict` stays unset, at its default `true`.

Verified before adopting, at v2.1.278: `claude plugin validate` passes the marketplace, and `claude plugin validate plugins/healthypress --strict` passes the manifest.

## Alternatives considered

**Keep declaring everything on the entry, and work around the tag command.** This was the first response, committed and then reset before it was pushed. It treated the symptom: it leaves the synthesized-manifest collision one careless `strict: false` away. The workaround itself turned out to be unnecessary, since the repo does not tag at all.

**Add a manifest that carries only metadata, letting `skills/` and `.mcp.json` be discovered by convention.** Both are auto-detected — the docs place MCP servers at "`.mcp.json` in plugin root" and skills under `skills/`, and a verified finding in `docs/plugin-mechanics.md` confirms every `SKILL.md` under `skills/` loads whether or not anything lists it. Rejected because an explicit inventory is easier to review than a convention, and because a stray `skills/` directory shipping is the live risk that inventory guards.

**Declare components in both the manifest and the entry.** Under `strict: true` the two merge rather than collide, so this would work. Rejected because two declarations of one thing guarantee drift, and because the entry is exactly where the 0.2.0 failure came from.

**Duplicate `version` on the entry as well, for pre-install display.** Rejected outright. The docs warn that Claude Code takes the `plugin.json` value without warning, so a stale entry version masks the real one invisibly. This is the version equivalent of the conflict just fixed.

## Consequences

The 0.2.0 failure mode is designed out rather than avoided. With the manifest owning components and the entry declaring none, there is nothing to collide whatever `strict` says.

The version now lives next to the `CHANGELOG.md` that describes it, which is what invariant 6 always said the unit should be.

`description` is deliberately left in both the manifest and the entry. The entry's value wins for display, and the duplication is cosmetic rather than functional, so it does not carry the risk that a duplicated `version` does. Worth revisiting if it drifts.

Two invariants changed and one was added: invariant 1 now admits `plugin.json`, invariant 6 points at the manifest, and invariant 7 forbids declaring components on an entry. The Architecture section gained a layer.

Verified after merging, at v2.1.278: `claude plugin update` took the installed copy from 0.2.1 to 0.2.2, `claude plugin list` reports `✔ enabled` with no error, and `claude plugin details` lists three skills and one MCP server. The cached `.claude-plugin/plugin.json` is byte-identical to the committed one, which settles the assumption this decision rested on — Claude Code copies a real manifest rather than synthesizing over it, and only synthesizes when none exists.

## Links

- Current-state doc: [DESIGN.md](../../DESIGN.md) — Architecture, invariants 1, 6, 7
- Procedure: [AGENTS.md](../../AGENTS.md) § Versioning and releases
- Platform facts: [docs/plugin-mechanics.md](../../docs/plugin-mechanics.md) § Verified facts, § Releasing
- Release: `plugins/healthypress/CHANGELOG.md` 0.2.2
- Official sources: `https://code.claude.com/docs/en/plugin-marketplaces.md` § Strict mode, § Optional plugin fields; `https://code.claude.com/docs/en/plugins-reference.md` § Version management, § Metadata fields; `https://code.claude.com/docs/en/plugin-dependencies.md` § Tag plugin releases for version resolution
- Supersedes: partly, [2026-09-22-per-plugin-versioning-and-releases.md](2026-09-22-per-plugin-versioning-and-releases.md)
- Superseded by:
