# <Use Case Name>

> **Copy-me skeleton.** Copy this whole directory to `plugins/<your-use-case>/`, replace every
> `<placeholder>`, delete what you don't need, then register the plugin in
> `.claude-plugin/marketplace.json`. Nothing in `docs/` is registered, so this template is inert
> until you copy it.

One sentence: what real-world job does an agent do with stock WordPress here?

Two or three more: why WordPress is a reasonable substrate for this job — which core primitives
(posts, pages, taxonomies, media, roles, visibility) map onto which parts of the problem.

**The plugin carries knowledge, not code.** No PHP, no custom post types, no companion WordPress
plugin. If you find yourself wanting one, see the core-only rule in `CONTRIBUTING.md`.

## Install

```
/plugin marketplace add Automattic/Agent-Use-Cases
/plugin install <your-use-case>@wordpress-agent-use-cases
```

Then:

1. At WordPress.com, go to **Preferences → AI and MCP** and enable MCP access.
2. Run `/mcp` and connect `wpcom`.
3. Run `/<your-use-case>:setup`.

## Commands

| Command | What it does |
|---|---|
| `/<your-use-case>:setup` | Verifies the MCP connection, prepares the site, creates the structure, prints a report the user confirms. Idempotent. |
| `/<your-use-case>:<verb>` | The daily driver. |

Keep this to the procedures a user would actually invoke. If it has no beginning and end, it's a
skill, not a command.

## Skills

| Skill | Content |
|---|---|
| `<name>-content-model` | What becomes a post, a page, a category, a tag. The schema. |
| `wpcom-mcp-operations` | Copied verbatim from `plugins/healthypress/skills/wpcom-mcp-operations/`. Do not fork it. |

## How the content is shaped

State your one rule, the way HealthyPress states "a post is an event, a page is a projection". Then
the taxonomy, the naming discipline, and the title/excerpt/date conventions. Be specific enough that
two agents on two days produce the same structure.

## What this is not

Say plainly where the model breaks down, what data protections do and don't apply, and who shouldn't
use this. If the honest answer for some users is "use something else", write that sentence.

## See also

Related plugins, and the shared field notes in `docs/wpcom-mcp-notes.md`.
