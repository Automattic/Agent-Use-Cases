# 2026-09-22 - Per-plugin versioning, and the marketplace manifest carries no version

Status: Accepted, partly superseded
Date: 2026-09-22
Deciders: Vlad Olaru, with Claude

> **Partly superseded** by [2026-09-22 — The plugin manifest owns the plugin's definition](2026-09-22-plugin-manifest-ownership.md). The release model, RULE 0, the no-`[Unreleased]` convention, and the removal of the marketplace version all still stand. What changed: the plugin's `version` moved from the marketplace entry to the plugin's own `plugin.json`, and the tagging step below was dropped entirely.

## Context

The repo carried the rule "bump the plugin's `version`, add a `CHANGELOG.md` entry" in three places — `DESIGN.md` invariant 6, `README.md`, `docs/plugin-mechanics.md` — but no release procedure anywhere. No CI, no tags, no publish step, and nothing stating that a merge to `trunk` is what ships a plugin to users. Three concrete gaps followed from that.

`AGENTS.md` told agents to append to a `CHANGELOG.md` `[Unreleased]` section. `plugins/healthypress/CHANGELOG.md` has twelve dated headings and has never had one, and no doc described the step that would turn one into the other.

Nothing said what a version bump means. Removing three user-invocable commands took `0.1.10` to `0.2.0`; removing skills and commands took `0.1.8` to `0.1.9`. One commit moved `0.1.5` to `0.1.7`, leaving `0.1.6` with a changelog entry for a state that was never released.

`DESIGN.md` listed the meaning of the marketplace `metadata.version` as an open question, citing README text that no longer existed, while practice bumped it in lockstep with the plugin on every release.

The sibling marketplace at `vladolaru/claude-code-plugins` had solved the first two with a RULE 0 section inline in its root `AGENTS.md`, and had implicitly answered the third by setting `metadata.version` once in December 2025 and never touching it again across nine months, 111 tags, and one plugin going from 1.0 to 1.120.0.

## Decision

**A merge to `trunk` is the release.** There is no build, publish step, or staging branch. Users refresh with `/plugin marketplace update`, which re-reads `.claude-plugin/marketplace.json` from the default branch.

**The plugin entry's `version` is the sole update signal.** Every commit that changes a plugin's behavior bumps it and adds a `CHANGELOG.md` entry under that version, in the same commit. Bump size follows commit type: `feat` minor, `fix`/`refactor`/`perf` patch, breaking major. Contributor-only changes bump nothing.

**No `[Unreleased]` section.** A changelog heading is written with its version and its date in the same commit as the change; merging publishes it. While a bump is still unmerged, further changes of similar impact fold into that version rather than opening a new one.

**The marketplace manifest carries no version.** `metadata` is removed from `.claude-plugin/marketplace.json` and `description` moved to its modern top-level spelling.

The full procedure lives in `AGENTS.md` § Versioning and releases. This record holds the rationale.

## Alternatives considered

**Keep `metadata.version` and bump it with the plugin.** Rejected on evidence. The official docs describe the marketplace-level `version` only as "Marketplace manifest version" and state that `description` and `version` "are also accepted under `metadata` for backward compatibility" — so `metadata.version` is a legacy nesting, not a distinct field. It is absent from the plugin version-resolution chain in `plugins-reference.md` § Version management, absent from the docs' canonical example, ignored by `claude plugin validate` (a non-semver value passes; removing the block warns only about the missing description), and absent from `claude plugin marketplace list --json`, which carries no version key for any registered marketplace. The published JSON Schema types it as a bare string while the plugin manifest's `version` is "Semantic version… following semver.org". Nothing reads it, and bumping it per release implied a meaning it does not have.

**Keep it, frozen, as a manifest-format marker.** This is what `claude-code-plugins` does by accident. Rejected because a field nobody reads and nobody bumps is a standing invitation for the next agent to bump it.

**Copy the sibling's `## [<version>] - UNRELEASED` convention**, where the version is assigned immediately and only the date stays `UNRELEASED` until release. Rejected because it does not transfer. There, "released" means tagged; here, a merge to `trunk` ships to users immediately, so a merged section marked `UNRELEASED` would be a lie. The sibling's own trigger wording has already drifted from its practice: RULE 0 says fold "if the latest bump is not pushed yet", but a fix was folded into 1.120.0 after it shipped in a merged PR.

**A separate `docs/releasing.md` runbook.** Rejected as too far from where the rule is needed. The procedure is short, and `AGENTS.md` is auto-loaded at session start while `docs/` is read on a trigger.

**Adopt the sibling's `<plugin>/v<semver>` tag format.** Rejected in favor of `claude plugin tag`, which ships with Claude Code and produces `{name}--v{version}`. *Superseded: the repo does not tag at all. A `{name}--v{version}` tag exists only so Claude Code can resolve another plugin's declared dependency, and nothing here declares one.*

## Consequences

The release rule is now one procedure in one place, with a bump-size policy an agent can apply without judgment calls, and the `[Unreleased]` contradiction is gone.

Verification moves after the merge, which is new and slightly uncomfortable: `claude plugin validate` passes manifests that still fail to load, and the dev loop's `--plugin-dir` never reads the marketplace entry, so neither can catch a bad entry. Writing this record surfaced a live instance — `strict: false`, added to the healthypress entry in 3f34958 with no version bump, made every installed copy fail with "conflicting manifests", because Claude Code synthesizes a `plugin.json` into the cached copy from the entry. Fixed in 0.2.1, and recorded in `docs/plugin-mechanics.md`. The post-merge `claude plugin list` check exists because of it.

Still unguarded: nothing enforces RULE 0. There is no CI in this repo, so the rule holds only because agents read `AGENTS.md`. A bump-vs-changelog consistency check is the obvious first CI job if that proves insufficient.

## Links

- Current-state doc: [DESIGN.md](../../DESIGN.md) — invariant 6
- Procedure: [AGENTS.md](../../AGENTS.md) § Versioning and releases
- Platform facts: [docs/plugin-mechanics.md](../../docs/plugin-mechanics.md) § Verified facts, § Releasing
- Related scratchpad session: `.agents/scratchpad/journal/2026-09-22-release-instructions-and-metadata-version.md`, extending `.agents/scratchpad/journal/2026-09-22-claude-code-plugins-comparison.md`
- Official sources: `https://code.claude.com/docs/en/plugin-marketplaces.md` § Marketplace schema, § Strict mode; `https://code.claude.com/docs/en/plugins-reference.md` § Version management; `https://json.schemastore.org/claude-code-marketplace.json`
- Supersedes:
- Superseded by: partly, [2026-09-22-plugin-manifest-ownership.md](2026-09-22-plugin-manifest-ownership.md)
