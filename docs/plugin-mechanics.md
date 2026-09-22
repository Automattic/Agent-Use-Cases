# Claude Code plugin mechanics

What this repo relies on from Claude Code's plugin system, and where to read the rest. This file records only what the official docs do not say, what was verified against a live Claude Code, and the dev loop for this repo. Schema tables, frontmatter fields, and changelogs are one URL away and are not copied here, because a copy goes stale and a stale copy is worse than none.

Last reviewed against Claude Code **v2.1.278** on **2026-09-22**.

## Read when

| Before you… | Read |
|---|---|
| Add or change a plugin entry in `.claude-plugin/marketplace.json` | https://code.claude.com/docs/en/plugin-marketplaces.md |
| Add a component to a plugin (skill, command, agent, hook, MCP server) or touch `plugin.json` | https://code.claude.com/docs/en/plugins-reference.md |
| Write or edit a `SKILL.md`, especially its frontmatter or description | https://code.claude.com/docs/en/skills.md |
| Change `.mcp.json` or an `allowed-tools` list that names MCP tools | https://code.claude.com/docs/en/mcp.md |
| Look for anything else | https://code.claude.com/docs/llms.txt (index; append `.md` to any docs URL for raw Markdown) |

## Verified facts

Each row says how it was verified and when. A row marked **unverified** is a working assumption, not a fact.

| Fact | Why it matters here | Verified |
|---|---|---|
| A plugin-bundled MCP server's tools are named `mcp__plugin_<plugin>_<server>__<tool>`; the same server added by the user is `mcp__<server>__<tool>`. | Every `allowed-tools` list that names a `wpcom` tool must carry both forms, or the skill breaks depending on how the server was installed. | Official docs (mcp.md), 2026-09-22. |
| The model-visible limit on a skill's `description` (plus legacy `when_to_use`) is 1,536 characters combined. The `/skills` menu may truncate what it displays; the model receives the full text. | Long, trigger-rich descriptions are fine. A 250-character cap circulated in older notes and is wrong. | Official docs (skills.md), 2026-09-22. |
| **`strict: false` breaks an installed plugin whose components are declared inline in the marketplace entry.** At install, Claude Code synthesizes `.claude-plugin/plugin.json` into the cached copy from the entry, components and all. `strict: false` means the entry is the complete definition, so that synthesized manifest is a second declaration and the plugin fails to load with "conflicting manifests: both plugin.json and marketplace entry specify components". | This is why every plugin here ships a real `plugin.json` that owns its components while its marketplace entry declares none: with the two disjoint and `strict` left at its default `true`, there is nothing to collide. Never set `strict: false`, and never move a component declaration onto an entry. This shipped broken in healthypress 0.2.0, was fixed in 0.2.1, and was designed out in 0.2.2. | Official docs (plugin-marketplaces.md § Strict mode) for the rule; the synthesized `plugin.json` and the load failure observed locally in the install cache, v2.1.278, 2026-09-22. |
| Users receive a plugin update only when that plugin's `version` changes. Claude Code resolves it from the first of: the plugin's `plugin.json`, the plugin's marketplace entry, the source's git commit SHA, an `archive` source's sha256, then `unknown`. The marketplace's own `version` is not in that chain. | Bump the plugin manifest's `version` for every behavior change, with a changelog entry. It is the sole update signal here. Setting `version` on the marketplace entry as well is a trap the docs call out: Claude Code takes the `plugin.json` value without warning, so the second copy goes stale invisibly. | Official docs (plugins-reference.md § Version management), 2026-09-22. |
| The marketplace-level `version` is "the marketplace manifest version", and `metadata.version` is only its legacy nesting: "`description` and `version` are also accepted under `metadata` for backward compatibility." Nothing reads either one. | This marketplace declares neither. Do not reintroduce one, and do not bump anything per release except the plugin entry. | Official docs (plugin-marketplaces.md § Marketplace schema) and the published JSON Schema, which types the marketplace version as a bare string while the plugin manifest's is "Semantic version… following semver.org". Confirmed locally at v2.1.278, 2026-09-22: `claude plugin validate` passes a non-semver `metadata.version` and warns only about a missing description when the whole block is removed, and `claude plugin marketplace list --json` carries no version key for any registered marketplace. |
| Claude Code synthesizes `.claude-plugin/plugin.json` into a plugin's cached copy **only when the plugin ships none**, building it from the marketplace entry. When the plugin ships one, the cached file is that file, byte for byte. | This is what makes the manifest-owns-the-definition shape safe: the manifest a user runs is the manifest in the repo, so `plugin.json` is worth treating as the shipped artifact. It is also why an entry that declares components is dangerous without a manifest — the synthesized copy becomes a rival declaration. | Locally, v2.1.278, 2026-09-22: compared `~/.claude/plugins/cache/wordpress-agent-use-cases/healthypress/0.2.2/.claude-plugin/plugin.json` against the committed file after a real `claude plugin update`; identical. The 0.2.0 cache, from before the plugin had a manifest, held a synthesized one. |
| `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_SKILL_DIR}`, `${CLAUDE_PLUGIN_DATA}`, `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_SESSION_ID}`, and `${CLAUDE_EFFORT}` are substituted in skill bodies, command bodies, hooks, and `.mcp.json`. | No skill here uses them yet. When one skill needs a file from another skill's `references/`, write `${CLAUDE_PLUGIN_ROOT}/skills/<skill>/references/<file>` rather than describing the path in prose and hoping the agent resolves it. | Official docs (plugins-reference.md, skills.md), 2026-09-22. |
| On a fresh install the `wpcom` HTTP MCP server exposes only `authenticate` and `complete_authentication`; the facade tools do not exist until the handshake completes, and Claude Code's native OAuth for `type: "http"` does not complete it on its own. | An unauthenticated server looks like a server with no capabilities, not like one returning auth errors. Skills perform the handshake themselves; see `plugins/healthypress/skills/health-record/SKILL.md`. | Live server, by the repo owner, 2026-09-17. The docs are silent on OAuth for plugin-bundled servers. |
| `claude plugin validate` passes for `.claude-plugin/marketplace.json` and for `plugins/healthypress`. | Run it before every commit that touches either. | Locally, v2.1.278, 2026-09-22. |
| Every `skills/<name>/SKILL.md` under a plugin's default `skills/` directory loads whether or not the plugin's `skills` array lists it, for both `strict` values. The array adds skill locations outside `skills/`; an entry may be a single skill directory or a directory containing several. | Listing `./skills/<name>` in the manifest is harmless and does nothing, but this repo lists them anyway, because an explicit inventory is easier to review than a directory convention. An undeclared skill is not disabled; the live risk is the reverse, a stray directory with a `SKILL.md` under `skills/` ships. `claude plugin details <plugin>@<marketplace>` prints the inventory that will load. | Locally, v2.1.278, 2026-09-22: throwaway marketplace with one declared and one undeclared skill; the `--debug` log reads "Loaded 2 skills from plugin probe default directory" and `plugin details` lists both. Older versions untested. |

## Dev loop

Nothing in this repo needs a build. To exercise a change against a real Claude Code session:

```bash
claude --plugin-dir ./plugins          # loads every plugin in the folder from the working tree (v2.1.265+)
/reload-plugins                        # inside the session: pick up edits without restarting
claude plugin validate .claude-plugin/marketplace.json
claude plugin validate plugins/<name>
```

If the released plugin is also installed, `--plugin-dir` loads both and they register the same skills. Disable the release for that session only:

```bash
claude --plugin-dir ./plugins --settings '{"enabledPlugins":{"healthypress@wordpress-agent-use-cases":false}}'
```

`--plugin-dir` loads the plugin from the working tree and never consults the marketplace entry, so it cannot catch a bad entry. The strict-mode break above passed the whole dev loop and only appeared on an installed copy. Check `claude plugin list` for the plugin's line after a release lands: it prints the resolved version, and a broken entry shows as `✘ failed to load` with the reason.

Two more tools exist and are unexplored here: `claude plugin eval` (v2.1.269+) runs a scored plugin test suite with a JSON or HTML report, and `/skill-doctor` (v2.1.261+) reports per-skill cost and usage.

## Releasing

The procedure is `AGENTS.md` § Versioning and releases. Two commands it leans on:

- `claude plugin details <name>@<marketplace>` prints the component inventory and projected token cost of an **installed** plugin. It takes no `--plugin-dir`, despite what its not-found error suggests, so it reports what users have rather than what the working tree holds.
- `claude plugin tag [path]` (v2.1.278) creates a `{name}--v{version}` git tag. **Unused here, deliberately.** Such a tag does one thing: Claude Code lists tags with that prefix to resolve another plugin's declared dependency on this one, and nothing here declares dependencies. An untagged relative-path plugin degrades gracefully anyway — Claude Code installs the marketplace's current copy and checks the constraint at load. Tags play no part in whether a user receives an update; that is the manifest's `version`. The command reads the version from `plugin.json` and exits `✘ No plugin manifest found` without one, refuses a dirty tree, and takes `--dry-run`, `-m <msg>`, `--push`, `--remote`, `-f`. Verified locally, v2.1.278, 2026-09-22; purpose per `plugin-dependencies.md` § Tag plugin releases for version resolution.

## Refreshing this file

When the installed Claude Code is more than a few minor versions past the one at the top, have an agent that can read the web re-check every row of *Verified facts* against the pages in *Read when*, list plugin-related entries in the Claude Code CHANGELOG since that version, and update the stamp. Change a row only with a source; delete a row rather than leave it unstamped.
