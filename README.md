# WordPress Agent Use Cases

A Claude Code plugin marketplace of **WordPress use cases**. Each plugin teaches an agent to use
stock WordPress as the substrate for a specific real-world job.

The premise: WordPress already ships the primitives a surprising number of applications need —
dated posts, hierarchical taxonomies, tags, media attachments, user roles, private visibility,
pages. What's usually missing isn't code, it's *convention*: a schema, a naming discipline, and a
procedure. That's what these plugins carry.

**Knowledge, not code.** No PHP, no custom post types, no companion WordPress plugins. Everything
runs against stock core features through the WordPress.com MCP server. If a use case needs a custom
post type, it isn't an agent use case — it's a WordPress plugin, and it belongs somewhere else.

## Install

```
/plugin marketplace add Automattic/Agent-Use-Cases
```

Then install the use case you want:

```
/plugin install healthypress@wordpress-agent-use-cases
```

Every plugin here talks to WordPress.com over MCP, so once per account:

1. At WordPress.com, go to **Preferences → AI and MCP** and enable MCP access.
2. Run `/mcp` in Claude Code and connect `wpcom`. Claude Code handles OAuth itself — no token to
   create or paste.

MCP is available on all paid WordPress.com plans. **Free sites get 30 days from site creation.**

## Plugins

| Plugin | Use case |
|---|---|
| [`healthypress`](plugins/healthypress) | A private WordPress.com site as a **personal health record**: privacy-hardened setup with a closed health taxonomy, then ongoing journaling into private, dated, tagged posts. Records and organizes; never diagnoses or advises. |

## Repository layout

```
.claude-plugin/marketplace.json   every plugin and skill declared inline
plugins/<name>/.claude-plugin/plugin.json   the plugin's manifest: version + components
plugins/<name>/                   one plugin = one use case
plugins/<name>/skills/            SKILL.md per skill, bulk in references/
plugins/<name>/.mcp.json          MCP servers the plugin needs
```

## Adding a use case

Create `plugins/<your-use-case>/` with a `.claude-plugin/plugin.json`, a `README.md`, a
`CHANGELOG.md`, an `.mcp.json`, and one `skills/<skill>/SKILL.md` per skill. The manifest is the
plugin: it carries the version and declares the components. Then add a catalogue entry to
`.claude-plugin/marketplace.json` with just a `name`, a `source`, and a `description` — never a
`version` and never a component, because Claude Code resolves those from the manifest and a second
copy on the entry only goes stale. Every `SKILL.md` under `skills/` loads on its own; the
manifest's `skills` array is there so the inventory is reviewable. Run
`claude plugin validate .claude-plugin/marketplace.json` and
`claude plugin validate plugins/<name> --strict` before you commit. Every change to a plugin's
behavior bumps its `version` and adds an entry to its `CHANGELOG.md`, because users receive an
update only when that version changes; a merge to `trunk` is the release. The full procedure is in
[`AGENTS.md`](AGENTS.md) under "Versioning and releases", and the dev loop and the verified plugin
mechanics are in [`docs/plugin-mechanics.md`](docs/plugin-mechanics.md).

Two rules carry most of the weight. **Core WordPress only**: if a use case needs a custom post type,
it isn't an agent use case. And **never hardcode an MCP facade's parameters** — `describe` the
operation at runtime, then read every write back, because an unrecognized parameter can be silently
dropped and a success response isn't evidence.

## License

GPL-3.0-or-later. See [`LICENSE`](LICENSE).
