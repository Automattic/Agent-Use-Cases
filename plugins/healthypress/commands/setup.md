---
description: Set up or audit a private WordPress.com site as a personal health record — privacy gate first, then taxonomy and pages
allowed-tools: mcp__wpcom__wpcom-mcp-site, mcp__wpcom__wpcom-mcp-create-site, mcp__wpcom__wpcom-mcp-content-authoring, mcp__wpcom__wpcom-mcp-user-management, mcp__plugin_healthypress_wpcom__wpcom-mcp-site, mcp__plugin_healthypress_wpcom__wpcom-mcp-create-site, mcp__plugin_healthypress_wpcom__wpcom-mcp-content-authoring, mcp__plugin_healthypress_wpcom__wpcom-mcp-user-management, AskUserQuestion, Skill
arguments:
  - name: site
    description: The WordPress.com site to use (domain or site ID). If omitted, you will be asked, with the option to create a new site.
    required: false
---

# Set Up HealthyPress

Creates — or audits — a private WordPress.com site configured as a personal health record. This
command is **idempotent**: re-running it is the privacy audit. Run it any time you want to confirm
the site is still private.

Load the `wpcom-mcp-operations` skill before step 1 and the `health-content-model` skill before
step 5.

**The order of these steps is the whole point. Do not write any health content before step 4
verifies that the site is Private.**

## Step 1: Verify the MCP connection

Call `action: list` on the site facade (`wpcom-mcp-site`, under whichever tool-name prefix is
available). If it fails with an authorization or connection error, tell the user:

> I can't reach the WordPress.com MCP server. Two things to check:
>
> 1. At WordPress.com, go to **Preferences → AI and MCP** and enable MCP access.
> 2. Run `/mcp` here and confirm `wpcom` shows as connected (it will walk you through sign-in).
>
> Then re-run `/healthypress:setup`.

Then STOP.

## Step 2: Choose or create the site

If the `site` argument was given, use it. Otherwise list the user's sites and ask via
`AskUserQuestion` which to use, offering **Create a new site** as an option.

Before accepting an **existing** site with content on it, check whether it has posts or pages. If it
does, warn:

> `<site>` already has `<n>` posts and `<m>` pages. HealthyPress will add categories, tags, and
> pages to this site, and setting it Private will hide the existing content from the public. It will
> not delete anything. Use a site with nothing else on it if you have one.

Ask for explicit confirmation before continuing. If they decline, STOP.

If creating a new site: `describe` then call the provisioning operation on `wpcom-mcp-create-site`.
Do not pick a clever name — see step 3 on the site title. A new site starts in Coming Soon, which is
**not** private, so the privacy gate in step 4 must run immediately. Do not hand the user a site URL and
finish; provisioning without the privacy gate is the most dangerous thing this command could do.

## Step 3: Check the plan and the MCP clock

Read the site's plan. If it is a **free** site, tell the user:

> `<site>` is on the free plan. MCP access on free sites is limited to 30 days from site creation.
> Setup and day-to-day logging will work, but a long history import may hit that cliff partway
> through. A paid plan removes the limit.

Continue — setup does not need confirmation for this. (`/healthypress:backfill` stops and asks.)

## Step 4: The privacy gate

Run in this exact order, and treat the **read-back** as the only evidence:

1. **Status.** Call the site status operation. Record the current visibility.
2. **Launch.** If the site is in Coming Soon or unlaunched, launch it. The **Private** option does
   not exist until a site is launched, so this step is mandatory even though launching sounds like
   the opposite of what we want. The window between launch and the next step is why nothing has
   been written yet.
3. **Set visibility to Private.**
4. **Set `blog_public` in site settings** to the private value as well. These two can disagree; set
   both.
5. **Read the status back.** If it does not report Private, STOP and tell the user:

   > I could not make `<site>` Private, so I'm stopping before writing anything. Right now it
   > reports `<actual status>`, which means the content would be reachable. Please set
   > **Settings → General → Privacy → Private** in the WordPress.com dashboard, then re-run
   > `/healthypress:setup`.

   Do not proceed to step 5. Do not create categories, pages, or posts.

Then harden the rest, reading each back after setting it:

6. **Discourage search engines.**
7. **Disable comments and pingbacks** site-wide.
8. **Disable newsletter / subscription email.** Then **verify the subscriber count is zero** via the
   user-management facade. If there are subscribers, list them and ask the user whether to remove
   them before any content exists. Do not continue to step 5 with subscribers present unless the
   user explicitly says to.
9. **Set a neutral site title** — no name, no condition, no "health record". Site titles show up in
   search results, link previews, and email. Suggest something like `Personal Log` and let the user
   pick. Clear the tagline.
10. **Set the site timezone** to the user's local timezone. Every record's date depends on this.
11. **Set the default post category to `needs-triage`** (after step 5 creates it, so re-read this
    setting at the end of step 5 and set it then).

## Step 5: Create the taxonomy

Create the categories from `health-content-model` — parents first, then leaves with their parent
assigned. The full list is in that skill's `references/taxonomy.md`.

This is idempotent: check for an existing category by slug before creating. Never create a duplicate
and never rename an existing one. Report created vs. already-present counts.

Then set `needs-triage` as the site's default post category and read it back.

Do not pre-create tags. Tags are created on demand by `/healthypress:log`, which searches first.

## Step 6: Create the pages

Create the eight pages, each with the generated-page footer from the `health-summary-pages` skill and
an empty `## Sources` list. They will be empty until there are posts, and that's correct:

Health Summary · Current Medications · Allergies & Intolerances · Conditions · Care Team ·
Immunization Record · Emergency Summary · About This Site

Set **Health Summary** as the site's front page (a static front page, not the blog index — the blog
index would list health records in reverse-chronological order as the first thing anyone sees).

**About This Site** is the one non-derived page. Write it with: what this site is, that it is
self-entered and not a medical record, that HIPAA does not apply to it, the model in two sentences
(posts are events, pages are derived), and the Boundaries block below.

Existing pages are left alone, except that a page missing the footer is reported in step 7 rather
than silently overwritten.

## Step 7: Print the privacy report

Show the user a report they can actually verify, with the read-back value for each line:

```
HealthyPress setup — <site>

Privacy
  Visibility            Private ✓
  blog_public           -1 ✓
  Search engines        Discouraged ✓
  Comments              Off ✓
  Pingbacks             Off ✓
  Newsletter email      Off ✓
  Subscribers           0 ✓
  Site title            "Personal Log" (neutral) ✓
  Timezone              America/Chicago ✓

Structure
  Categories            29 present (24 created, 5 already existed)
  Default category      needs-triage ✓
  Front page            Health Summary ✓
  Pages                 8 present

Anything marked ✗ needs fixing before you log health information.
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
