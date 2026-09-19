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
| [`healthypress`](plugins/healthypress) | A private WordPress.com site as a **personal health record**: privacy-hardened setup, guided history backfill, ongoing journaling, a Health Summary page, and care team sharing. Records and organizes; never diagnoses or advises. |

## Repository layout

```
.claude-plugin/marketplace.json   every plugin, skill, and command declared inline
plugins/<name>/                   one plugin = one use case
CONTRIBUTING.md                   how to add use case #2
CLAUDE.md                         conventions for agents working in this repo
```

## Adding a use case

Create `plugins/<your-use-case>/` and register every component path in
`.claude-plugin/marketplace.json`. Read [`CONTRIBUTING.md`](CONTRIBUTING.md) first — it covers
the core-only rule, the runtime-schema-discovery rule, and why some text in this repo is duplicated
on purpose.

## License

GPL-3.0-or-later. See [`LICENSE`](LICENSE).
