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
| `strict: true` (the default) means the plugin's `plugin.json` is authoritative and the marketplace entry supplements it. `strict: false` means the marketplace entry is the complete definition. | This marketplace declares every component inline and its plugins have no `plugin.json`, which is the `strict: false` mode. Entries currently say `true`; validation passes either way. Decide whether to flip the flag or add `plugin.json` files before adding a second plugin. | Official docs (plugin-marketplaces.md), 2026-09-22. |
| Users receive a plugin update only when that plugin's `version` (in the marketplace entry or its `plugin.json`) changes. The top-level `metadata.version` has no documented effect. | Bump the plugin's `version` for every behavior change, with a changelog entry. Nothing depends on `metadata.version`. | Official docs for the per-plugin rule; the `metadata.version` half is inferred from its absence in the schema docs. 2026-09-22. |
| `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_SKILL_DIR}`, `${CLAUDE_PLUGIN_DATA}`, `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_SESSION_ID}`, and `${CLAUDE_EFFORT}` are substituted in skill bodies, command bodies, hooks, and `.mcp.json`. | No skill here uses them yet. When one skill needs a file from another skill's `references/`, write `${CLAUDE_PLUGIN_ROOT}/skills/<skill>/references/<file>` rather than describing the path in prose and hoping the agent resolves it. | Official docs (plugins-reference.md, skills.md), 2026-09-22. |
| On a fresh install the `wpcom` HTTP MCP server exposes only `authenticate` and `complete_authentication`; the facade tools do not exist until the handshake completes, and Claude Code's native OAuth for `type: "http"` does not complete it on its own. | An unauthenticated server looks like a server with no capabilities, not like one returning auth errors. Skills perform the handshake themselves; see `plugins/healthypress/skills/health-record/SKILL.md`. | Live server, by the repo owner, 2026-09-17. The docs are silent on OAuth for plugin-bundled servers. |
| `claude plugin validate` passes for `.claude-plugin/marketplace.json` and for `plugins/healthypress`. | Run it before every commit that touches either. | Locally, v2.1.278, 2026-09-22. |
| **Unverified:** a `skills/<name>/SKILL.md` directory that is not listed in its plugin's `skills` array is silently not loaded. | `README.md` states this from experience; the official docs do not say either way, and `validate` checks declared components, not undeclared directories. Until it is tested, declare every skill. | Not yet tested. A two-minute check: add an undeclared skill directory, load with `--plugin-dir`, see whether it appears. |

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

Two more tools exist and are unexplored here: `claude plugin eval` (v2.1.269+) runs a scored plugin test suite with a JSON or HTML report, and `/skill-doctor` (v2.1.261+) reports per-skill cost and usage.

## Refreshing this file

When the installed Claude Code is more than a few minor versions past the one at the top, have an agent that can read the web re-check every row of *Verified facts* against the pages in *Read when*, list plugin-related entries in the Claude Code CHANGELOG since that version, and update the stamp. Change a row only with a source; delete a row rather than leave it unstamped.
