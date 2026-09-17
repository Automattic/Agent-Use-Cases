# Contributing

How to add use case #2.

## One plugin = one use case

A use case is a real-world job someone does with a WordPress site. Not a feature, not a tool — a
job, with a setup procedure, a daily driver, and a content model. If you can't name the job in a
sentence ("run a private site as a personal health record"), it isn't a use case yet.

Start by copying the skeleton:

```
cp -R docs/use-case-template plugins/<your-use-case>
```

Nothing in `docs/` is registered with the marketplace, so the template is inert until you copy it.

## Register every component path

`.claude-plugin/marketplace.json` declares everything **inline** — there are no per-plugin
`plugin.json` files in this repo. Add one entry to the `plugins` array with `name`, `description`,
`source`, `version`, `license`, `strict: true`, `mcpServers`, and explicit `skills` and `commands`
arrays listing **every** skill directory and **every** command file.

> **An omitted path silently disables the component.** No error, no warning — the command just
> doesn't appear in `/help` and the skill never loads. This is the single most common way a change
> here breaks. The CI workflow checks that every declared path exists on disk; it cannot check that
> every file on disk is declared, so that part is on you.

Bump **both** versions on every change: the plugin's `version` and the marketplace's
`metadata.version`. Add a `CHANGELOG.md` entry in the plugin.

## Core-only rule

Stock WordPress features only: posts, pages, hierarchical categories, tags, media, users and roles,
site visibility, site settings. No PHP. No custom post types. No custom taxonomies. No post meta. No
companion WordPress plugin.

**If you want a custom post type, you're writing a WordPress plugin, not an agent use case.** That's
a legitimate thing to want and the wrong repo for it. The constraint is what makes these plugins
installable in one step against any WordPress.com site, and what keeps them knowledge rather than
software.

The corollary: when core can't express something, **say so in the README** rather than working
around it. HealthyPress has a "Where the data model breaks down" section listing twelve honest
limits. Write yours.

## Never hardcode a facade schema

The WordPress.com MCP uses a facade pattern — a handful of tools, each taking an `operation` plus
`action: list` / `action: describe`. The documentation states that schemas evolve and the live
`describe` response is the source of truth.

So: skills and commands **teach discovery**. They say "`describe` this operation before its first
use, then pass exactly the parameters the live schema names." They never list parameter names as if
they were a contract — including parameter names copied out of `docs/wpcom-mcp-notes.md`.

Related: **read writes back.** A parameter the schema doesn't recognize can be silently dropped
rather than rejected, so success responses aren't evidence. Verify the stored state.

## Copy `wpcom-mcp-operations` verbatim

Every use case here needs the same MCP mechanics, and plugin sources can't share directories. When
you need that knowledge, **copy `plugins/healthypress/skills/wpcom-mcp-operations/SKILL.md`
verbatim** into your plugin. Don't fork it, don't trim it to what you currently use, and don't
import it by path.

Two things to change in the copy: the plugin-scoped tool prefix
(`mcp__plugin_<your-plugin>_wpcom__*`), and — if your use case has no safety stance — drop the
trailing `## Boundaries` block, which is HealthyPress-specific.

Promoting it to its own plugin isn't worth the install friction while there's one consumer. Once
there are three copies and they've drifted, revisit that decision deliberately.

## Skill shape

One `SKILL.md` per skill directory, roughly 100 lines:

```
---
name: <skill-name>
description: <what it knows> Use when <the user's actual phrases and situations>.
---

# Title
Overview — what this skill owns, what it leaves to other skills
## Quick reference    a table you can act on from the first screen
## <Topic>            one H2 per topic, rules as imperatives, examples over prose
## Common gotchas     the silent failures and the wrong defaults
```

Bulk goes to `references/<topic>.md` in the skill directory, linked by relative path. Full
vocabularies, per-type field tables, and long worked examples belong there, not in `SKILL.md`.

The `description` is the entire basis on which the skill gets loaded. Write the phrases a user would
actually type ("log my headache"), not a summary of the file's contents.

## Command shape

Commands are **user-invoked procedures with a beginning and an end**. Skills are **knowledge the
agent loads when the topic comes up**. If it has no end state, it's a skill.

Frontmatter: `description` (one imperative line — this is what `/help` shows), `allowed-tools`, and
`arguments` with a `description` and `required` per argument. Body: numbered steps, gates before
writes, explicit "…then STOP" for every failure branch, and a closing scope line.

**`allowed-tools` must list both MCP tool-name prefixes** — `mcp__wpcom__<facade>` and
`mcp__plugin_<your-plugin>_wpcom__<facade>` — because plugin-scoped servers get renamed and the
command otherwise breaks depending on how the server was installed.

Name the skills the command depends on, and the step to load them before. Skill loading is
probabilistic; naming it in the procedure is what makes it reliable.

## Safety text is duplicated on purpose

HealthyPress's `## Boundaries` block appears **verbatim** in all four `SKILL.md` files and as the
closing block of all five commands. That is not an oversight and it is not a DRY violation waiting
to be fixed.

**Skill loading is probabilistic.** A guardrail that lives in one file is a guardrail that sometimes
doesn't load — and the times it doesn't are exactly the times someone asks a question the guardrail
was written for. Duplication is the mechanism.

So: **do not factor it out, do not replace copies with a cross-reference, and do not paraphrase a
copy to fit its surroundings.** If the text needs to change, change every copy in the same commit.
A script that checks the copies are byte-identical is a welcome contribution; a script that
deduplicates them is not.

## Before you open a pull request

1. `jq . .claude-plugin/marketplace.json` — valid JSON.
2. Every `source`, `skills`, `commands`, and `mcpServers` path in it exists on disk. (CI does this;
   run it locally first.)
3. Install from a local path and confirm the components actually appear:
   ```
   /plugin marketplace add /path/to/Agent-Use-Cases
   /plugin install <your-use-case>@wordpress-agent-use-cases
   ```
   Then check `/help` lists every command, the skills are listed, and `/mcp` connects `wpcom`.
4. Run the whole flow **against a scratch site**, never a real one. Then delete the scratch site.
5. If you discovered something about the MCP along the way, put it in `docs/wpcom-mcp-notes.md` —
   marked **verified** with the date and how you checked, or left in the open-questions table.
