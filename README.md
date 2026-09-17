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
| [`healthypress`](plugins/healthypress) | A private WordPress.com site as a **personal health record**: privacy-hardened setup, guided history backfill, ongoing journaling, derived summary pages (current medications, allergies, conditions, emergency summary), and care team sharing. Records and organizes; never diagnoses or advises. |

## Before you put real data in one of these

These plugins store real information on a hosted website, and the honest limits are the same for all
of them:

- **Hosted content has no special legal protection.** A site you own is not a regulated system of
  record. For health data specifically, **HIPAA does not apply** — it covers providers and insurers,
  not your own website.
- **Automattic staff can access site content**, as with any hosted WordPress.com site, and hosted
  content is subject to legal process.
- **Your conversation transcripts are a second copy.** Everything you tell the agent to record
  passes through a conversation, stored somewhere you don't control under a different policy than
  the site. There's no version of agent-driven data entry where this isn't true.
- **Site visibility is one setting.** Each plugin ships an idempotent setup command that doubles as
  a privacy audit. Re-run it; don't assume.
- **Export is manual.** There's no export operation in the MCP. Use `wordpress.com/export/<site>`
  for a WXR backup.

Each plugin's own README states where its data model breaks down and who shouldn't use it. Read that
one too.

## Repository layout

```
.claude-plugin/marketplace.json   every plugin, skill, and command declared inline
plugins/<name>/                   one plugin = one use case
docs/use-case-template/           copy-me skeleton for a new use case
docs/wpcom-mcp-notes.md           shared field notes on the WordPress.com MCP
CONTRIBUTING.md                   how to add use case #2
CLAUDE.md                         conventions for agents working in this repo
```

## Adding a use case

Copy `docs/use-case-template/`, register every component path in
`.claude-plugin/marketplace.json`, and read [`CONTRIBUTING.md`](CONTRIBUTING.md) first — it covers
the core-only rule, the runtime-schema-discovery rule, and why some text in this repo is duplicated
on purpose.

## See also

[`build-with-wordpress`](https://github.com/Automattic/claude-code-wordpress.com) in the official
Automattic marketplace is for **building** WordPress.com sites. This marketplace is about **using**
one as an application. Install both if you want both.

## License

GPL-3.0-or-later. See [`LICENSE`](LICENSE).
