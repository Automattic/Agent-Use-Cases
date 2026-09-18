---
description: Create a new private WordPress.com site as a personal health record — privacy gate first, then taxonomy and pages.
allowed-tools: mcp__wpcom__wpcom-mcp-site, mcp__wpcom__wpcom-mcp-create-site, mcp__wpcom__wpcom-mcp-content-authoring, mcp__wpcom__wpcom-mcp-user-management, mcp__wpcom__authenticate, mcp__wpcom__complete_authentication, mcp__plugin_healthypress_wpcom__wpcom-mcp-site, mcp__plugin_healthypress_wpcom__wpcom-mcp-create-site, mcp__plugin_healthypress_wpcom__wpcom-mcp-content-authoring, mcp__plugin_healthypress_wpcom__wpcom-mcp-user-management, mcp__plugin_healthypress_wpcom__authenticate, mcp__plugin_healthypress_wpcom__complete_authentication, AskUserQuestion, Bash, Skill
---

# Set Up HealthyPress

**This command always creates a brand new private WordPress.com site.** It never asks you to pick
from your existing sites, and it never writes health structure onto a site that has other things on
it.

Load the `wpcom-mcp-operations` skill before step 1 and the `health-content-model` skill before
step 5.

**The order of these steps is the whole point. Do not write any health content before step 4
verifies that the site is Private.**

## Step 1: Connect to the MCP server

Check which `wpcom` tools are available.

**If the facade tools (`wpcom-mcp-site` and friends) are present**, you're authenticated. Go to
step 2.

**If the only `wpcom` tools available are `authenticate` and `complete_authentication`**, the server
is reachable but not yet authorized. This is the normal state on a fresh install. Do not tell the
user to go configure anything — just do the handshake for them:

1. Call the `authenticate` tool. It returns an authorization URL.
2. **Open that URL in their browser immediately** with Bash, so they don't have to copy anything:
   `open '<url>'` on macOS, `xdg-open '<url>'` on Linux, `start '<url>'` on Windows.
3. Tell them what's happening and what they're approving:

   > I've opened WordPress.com in your browser to authorize access.

4. Wait for them to confirm they've approved it, then call `action: list` on the site facade.
5. If the facade tools still aren't available, the local callback probably didn't land. Ask:

   > The redirect didn't complete. Copy the full URL from your browser's address bar — it starts
   > with `http://localhost:` — and paste it here.

   Then call `complete_authentication` with it and retry the facade call.

If the facade call fails with an authorization error even after a successful handshake, MCP access
isn't enabled on the account. Tell the user:

> WordPress.com is rejecting MCP calls. Go to **Preferences → AI and MCP** at WordPress.com and
> enable MCP access, then re-run `/healthypress:setup`.

Then STOP.

## Step 2: Create the site

Create a new site. Do not list the user's sites and do not ask them to choose one; a dedicated site
is the whole point, and picking an existing one is how health content ends up somewhere it
shouldn't be.

Ask the user for a short word to make the name memorable — a first name, nickname, or anything
else they'd recognize (it does not need to be their real WordPress.com username). Combine it as
`healthypress-<word>` for both the display title and the derived URL slug.

`describe` then call the provisioning operation on `wpcom-mcp-create-site`. Derive the subdomain
slug from `healthypress-<word>` using the tool's own derivation rule (lowercase, strip diacritics,
remove non-alphanumeric), then call `subdomain.check`. If the slug is taken, tell the user and ask
for a different word, or offer the numeric-suffixed slug WordPress.com proposes instead.

**Get explicit approval before provisioning.** Show the user the proposed title and the exact
`would_be_url` from `subdomain.check`, and wait for a yes before calling `site.provision`. This is
the one approval pause in this command — once they say go, everything from provisioning through the
end of the privacy gate runs without further pauses.

Tell the user the site was created and that you are now making it private — then do that
immediately. The privacy gate in step 4 must run in the same turn. Do not hand the user a site URL
and finish; skipping the immediate privacy gate is the most dangerous thing this command could do.

## Step 3: Check the plan and the MCP clock

Read the site's plan. If it is a **free** site, tell the user:

> `<site>` is on the free plan. MCP access on free sites is limited to 30 days from site creation.

Continue — setup does not need confirmation for this. (`/healthypress:backfill` stops and asks.)

## Step 4: The privacy gate

Run in this order, and treat the **read-back** as the only evidence:

1. **Read the status.** Call the site status operation (`get_status`). Record `visibility`,
   `launch_status`, and `has_active_subscribers`.
2. **If visibility is already `private`** — the normal case on a new site — change nothing here and
   go to the hardening list.
3. **If visibility is not `private`**, set it to private via the visibility operation, and also set
   `blog_public: -1` in site settings. On a fresh site these agree and the settings write reports
   `blog_public` as unchanged; on an older site they may not, so set both.
4. **Read the status back.** If it does not report `private`, STOP and tell the user:

   > I could not confirm `<site>` is Private, so I'm stopping before writing anything. It reports
   > `<actual visibility>`, which means the content could be reachable. Please set
   > **Settings → General → Privacy → Private** in the WordPress.com dashboard, then tell me and
   > I'll read the status back and continue.

   Do not proceed. Do not create pages or posts.

Then harden the rest. Batch these into **one** `settings.update` call with `user_confirmed: true` —
the facade requires that flag per write operation, so one call means setting it once instead of
several times. No need to pause for approval first; write, then report what the read-back shows.

5. **`blog_public: -1`** — private. Include it even when already set; it's the belt to the
   visibility braces.
6. **`comment_registration: true`** — requires a logged-in account to comment. **This is not the
   same as disabling comments**, and `settings.update` has no `default_comment_status` or
   `default_ping_status` field, so comments and pingbacks **cannot be turned off through the MCP.**
   Say that plainly in the report and point at **Settings → Discussion** in wp-admin.
7. **`timezone_string`** — the user's local timezone. Every record's date depends on it. Detect it
   from the local system rather than asking.
8. **`date_format: "Y-m-d"` and `time_format: "H:i"`** — unambiguous dates in a health log.
9. **Site title and tagline.** A new site already carries the title chosen at step 2 and an empty
   tagline, so normally there is nothing to do here.

## Step 5: Create the taxonomy

Create the categories from `health-content-model` — the full closed list is in that skill's
`references/taxonomy.md`. Verified schema for `categories.create`: `name` (required), `slug`,
`description`, `parent` (a category **ID**, not a slug), `include_fields`, `user_confirmed`.

1. **Read what exists first.** List the site's categories and match on **slug**. Never create a
   duplicate, never rename an existing term. On a re-run this is what makes the step idempotent.
2. **Parents before leaves.** `parent` takes a numeric ID, so the five container categories
   (`encounters`, `diagnostics`, `medications`, `conditions`, `care-admin`) must be created first and
   their returned IDs captured. Creating a leaf before its parent orphans it at the top level.
3. **Then the leaves**, each with its parent's ID, and the five childless categories (`symptoms`,
   `allergies`, `immunizations`, `journal`, `needs-triage`) with no parent.
4. Pass `include_fields: ["id", "slug", "parent"]` to keep responses small — 28 full term objects is
   a lot of noise for data you only need the IDs from.
5. Give each container a `description` saying it is a container and must never be assigned directly.

**Confirmation and pacing.** Every create is a separate write requiring `user_confirmed: true`; the
flag does not carry across calls, so set it on each one — no need to pause for approval first. Send
them in batches rather than all at once, so an undocumented rate limit costs one batch instead of
the run.

Report created vs. already-present counts.

**The default category cannot be set.** `settings.update` has no `default_category` field, so
`needs-triage` cannot be made the site's default despite existing as a term. Say so in the report:
the safety net for unclassifiable records depends on `/healthypress:log` always assigning a leaf
explicitly, and on `/healthypress:review` auditing for strays.

Do not pre-create tags. Tags are created on demand by `/healthypress:log`, which searches first.

## Step 6: Create the pages

Create the eight pages, each with the generated-page footer from the `health-summary-pages` skill and
an empty `## Sources` list. They will be empty until there are posts, and that's correct:

Health Summary · Current Medications · Allergies & Intolerances · Conditions · Care Team ·
Immunization Record · Emergency Summary · About This Site

Each `pages.create` is a separate write needing `user_confirmed: true`. Show the user the list once,
take one approval, then create all eight.

Then wire the front page, in this order — **the order is enforced by the API**:

1. **Publish** Health Summary (`status: "publish"`). `settings.update` rejects a page that isn't
   published, and a draft cannot render as a front page.
2. `settings.update` with `show_on_front: "page"`, `page_on_front: <Health Summary id>`, and
   `user_confirmed: true`.
3. Read the setting back.

A static front page matters: the default blog index would list health records in
reverse-chronological order as the first thing anyone sees.

**About This Site** is the one non-derived page. Write it with: what this site is, that it is
self-entered and not a medical record, that HIPAA does not apply to it, the model in two sentences
(posts are events, pages are derived), and the Boundaries block below.

Every new site also comes with WordPress's own default placeholder content: a page titled "About"
(content starting "This is an example of a page...") and a post titled "Hello World!". These are
not HealthyPress content and have nothing to do with the About This Site page above — **trash
both** before moving on (`pages.delete` / `posts.delete` only move to trash via the MCP; that's
fine, no need for a permanent purge). Find them by listing pages and posts and matching the default
title/slug (`about` / `hello-world`), and confirm the page's content still contains "This is an
example of a page" before trashing it — don't delete on title alone.

## Step 7: Print the privacy report

Show the user a report they can actually verify, with the **read-back** value for each line — not
the value you asked for. Mark anything that couldn't be verified or couldn't be set:

```
HealthyPress setup — healthypressq4t8.wordpress.com (blog 257423784)

Privacy
  Visibility            private ✓        (read back)
  blog_public           -1 ✓
  Comments              login required ⚠  cannot be disabled via MCP
  Site title            "healthypress-jordan" ✓
  Tagline               empty ✓
  Timezone              America/Chicago ✓

Structure
  Categories            28 present (23 created, 5 already existed)
  Default category      not settable via MCP ⚠
  Front page            Health Summary ✓
  Pages                 8 present
  Default WP content    removed (sample "About" page, "Hello World!" post)

Fix in wp-admin (not reachable through the MCP):
  Settings → Discussion  turn off comments and pingbacks

Anything marked ✗ needs attention before you log health information.
```


---

This command configures a site. It does not record, interpret, or advise on health information.