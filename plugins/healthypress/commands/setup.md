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

`describe` then call the provisioning operation on `wpcom-mcp-create-site`. For the name: generate a
neutral, non-identifying one (`personal-log-<4 random chars>` is fine) rather than asking. Do not
pick anything clever or descriptive — site names appear in URLs, search results, and link previews.
The display title gets set to something neutral during the privacy gate and can be changed
later; the URL is forever, so keep it dull.

Tell the user the site was created and that you are now making it private — then do that
immediately. **A new site starts in Coming Soon, which is not private**, so the privacy gate in
step 4 must run in the same turn. Do not hand the user a site URL and finish; provisioning without
the immediate privacy gate is the most dangerous thing this command could do.

## Step 3: Check the plan and the MCP clock

Read the site's plan. If it is a **free** site, tell the user:

> `<site>` is on the free plan. MCP access on free sites is limited to 30 days from site creation.

Continue — setup does not need confirmation for this. (`/healthypress:backfill` stops and asks.)

## Step 4: The privacy gate

**Verified 2026-09-17 against a live site: a freshly provisioned WordPress.com site already reports
`visibility: private` and `launch_status: unlaunched`.** It is *not* in Coming Soon, and Private is
*not* gated behind launching. So:

> **Do NOT launch the site.** Launching transitions it toward live. There is no reason to launch a
> HealthyPress site, ever — a private unlaunched site is exactly the posture we want. If a future
> version needs `manage-site.launch`, that's a deliberate decision, not a setup step.

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

Then harden the rest. Batch these into **one** `settings.update` call with `user_confirmed: true`,
after showing the user the field list and getting a single approval — the facade requires
confirmation per write operation, so one call means one approval instead of six:

5. **`blog_public: -1`** — private. Include it even when already set; it's the belt to the
   visibility braces.
6. **`users_can_register: false`** — a private WordPress.com site is visible to site members, so
   open registration is worth closing. **Known issue: this write does not stick.** It returns
   `success` with a `before/after` transition and the value stays `true` on read-back. Most likely
   the option is inert on Simple WordPress.com sites, where accounts are account-level rather than
   site-level. Attempt it, then **report whatever the read-back actually says** — never report it as
   closed on the strength of the write response.
7. **`comment_registration: true`** — requires a logged-in account to comment. **This is not the
   same as disabling comments**, and `settings.update` has no `default_comment_status` or
   `default_ping_status` field, so comments and pingbacks **cannot be turned off through the MCP.**
   Say that plainly in the report and point at **Settings → Discussion** in wp-admin.
8. **`timezone_string`** — the user's local timezone. Every record's date depends on it. Detect it
   from the local system rather than asking.
9. **`date_format: "Y-m-d"` and `time_format: "H:i"`** — unambiguous dates in a health log.
10. **Site title and tagline.** A new site already carries the neutral title chosen at step 2 and an
    empty tagline, so normally there is nothing to do. Only write `blogname` if the existing title
    names a person or a condition, and say so in the report.

Then, separately from the settings write:

11. **Newsletter.** Read the newsletter status and **verify the subscriber count is zero**. A fresh
    site reports `total: 0` and `has_active_subscribers: false`, which is the expected state and
    needs no write. If there *are* subscribers, list them and stop to ask before any content exists.
    Only call the newsletter settings update if subscription email is actually enabled.

**Not settable through the MCP — state these as gaps in the report, don't pretend otherwise:**

- **The default post category.** `settings.update` has no `default_category` field, so
  `needs-triage` cannot be made the site default even though step 5 creates it as a term. Records
  therefore rely on `/healthypress:log` always assigning a category explicitly, and on
  `/healthypress:review` auditing for strays.
- **Comments and pingbacks off.** See item 7.
- **Search-engine discouragement as its own setting.** It's the `blog_public: 0` state, which is
  *public*. `private` (`-1`) is strictly stronger, so this is moot on a HealthyPress site.

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
flag does not carry across calls. So show the user the list **once**, take **one** approval, then
issue the writes with the flag set on each. Send them in batches rather than all at once, so an
undocumented rate limit costs one batch instead of the run.

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

## Step 7: Print the privacy report

Show the user a report they can actually verify, with the **read-back** value for each line — not
the value you asked for. Mark anything that couldn't be verified or couldn't be set:

```
HealthyPress setup — personallogq4t8.wordpress.com (blog 257423784)

Privacy
  Visibility            private ✓        (read back)
  Launch status         unlaunched ✓     (never launched, by design)
  blog_public           -1 ✓
  Registration open     yes ✗            write does not persist — see below
  Comments              login required ⚠  cannot be disabled via MCP
  Newsletter email      no sends, 0 subscribers ✓
  Site title            "Personal Log q4t8" (neutral) ✓
  Tagline               empty ✓
  Timezone              America/Chicago ✓

Structure
  Categories            28 present (23 created, 5 already existed)
  Default category      not settable via MCP ⚠
  Front page            Health Summary ✓
  Pages                 8 present

Fix in wp-admin (not reachable through the MCP):
  Settings → Discussion  turn off comments and pingbacks
  Settings → General     confirm membership/registration is closed

Anything marked ✗ needs attention before you log health information.
```

Then say plainly, every time this command runs:

> Two things this setup cannot do for you:
>
> - **This is not a legally protected medical record.** HIPAA covers providers and insurers, not
>   your own website. Automattic staff can access site content as with any hosted site, and hosted
>   content is subject to legal process.
> - **Our conversation transcripts contain the same health information**, and that's a second copy
>   you don't control.
>
> Full detail is in the plugin README, and on your new **About This Site** page.

Finally, point at what's next: `/healthypress:log` for a record, `/healthypress:backfill` for
history, `/healthypress:share` for care team access.

---

This command configures a site. It does not record, interpret, or advise on health information.

## Boundaries

HealthyPress is a recorder and organizer. It is not a clinician.

- Do NOT diagnose, suggest a diagnosis, or rank possible causes.
- Do NOT recommend, adjust, start, or stop any treatment, medication, dose, or supplement.
- Do NOT interpret a lab value, vital, or imaging result as good, bad, normal, concerning,
  or improving. Transcribe the reference range and flag the report itself printed,
  attributed to the report. Nothing more.
- Do NOT tell the user whether something is urgent, or estimate risk or prognosis.
- DO surface factual patterns over what was logged: counts, dates, frequencies,
  co-occurrences, gaps. "3 migraines logged in June, all on weekdays." Then stop.
- DO quote the user's own words for anything subjective. Never translate them into
  clinical language.
- If asked "what does this mean?" or "should I be worried?", say plainly that you record
  and organize but cannot interpret health information, and that their clinician can.
  Offer to assemble the relevant records for that conversation instead.

**One exception — safety.** If the user describes an acute emergency (chest pain, stroke
signs, trouble breathing, anaphylaxis, severe bleeding, overdose, or thoughts of self-harm),
stop the logging workflow immediately, tell them to contact emergency services or a crisis
line now, and do not resume until they say the situation is resolved.
