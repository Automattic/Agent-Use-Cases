# HealthyPress

Run a private WordPress.com site as your personal health record.

WordPress already ships nearly everything a health log needs: dated posts, hierarchical categories,
tags, media attachments, user roles, and private site visibility. HealthyPress is the knowledge that
turns those primitives into a coherent record — a closed taxonomy, a naming discipline, and a
privacy-first setup sequence.

**The plugin carries knowledge, not code.** No PHP, no custom post types, no companion WordPress
plugin. Everything it does, it does through the WordPress.com MCP server against stock core
features.

> **Read "What this is not" below before you put real health information into it.** Some of it is
> legal, some of it is technical.

## Install

```
/plugin marketplace add Automattic/Agent-Use-Cases
/plugin install healthypress@wordpress-agent-use-cases
```

Then:

1. At WordPress.com, go to **Preferences → AI and MCP** and enable MCP access. Nothing works until
   you do.
2. Run `/healthypress:setup`. It handles the WordPress.com authorization itself — it opens your
   browser, you approve, and it carries on. There's no token to create or paste, and nothing to set
   up in `/mcp` first.

`/healthypress:setup` **creates a new site** every time you run it. It won't offer to use an
existing site, because a dedicated site is what keeps health content out of places it shouldn't be.

## Skills

Three skills. You invoke two of them; the third loads on its own.

| Skill | What it does |
|---|---|
| `/healthypress:setup` | Creates a new site, then runs the **privacy gate** — set Private, verify by reading the status back, require login to comment, neutral title, timezone — then creates the taxonomy and the Health Summary page and prints a privacy report. Stops hard if the site can't be confirmed Private. |
| `/healthypress:log` | The daily driver. One event in, one private post out: interview, classify, resolve tags, compute the date, compose, read the whole record back for confirmation, save, attach files. Also triggers from plain language — "log my headache", "I started a new medication". |
| `health-record` | Reference knowledge, not a workflow. The schema (what becomes a post, the category taxonomy, the tag namespaces, titles, excerpts, dates, the sectioned body, media rules) and the WordPress.com MCP mechanics (the facade pattern, runtime schema discovery, status and date defaults, write confirmation, visibility, trash vs. delete). Loads behind the other two; you don't invoke it. |

To backfill your history, run `/healthypress:log` repeatedly — there's no separate import workflow.

## How the record is shaped

**A post is an event that happened at a point in time. Posts are the whole record.** Current state
isn't stored anywhere — it's read back out of the posts when you ask for it.

So a medication isn't a post — *starting* lisinopril is a post, *stopping* it is another, and "am I
still on lisinopril" is answered from the two. Same for conditions (diagnosis minus resolution),
allergies (latest reaction per allergen), and the care team.

- **Categories** are the kind of record: a closed, hierarchical list (`visit`, `lab`, `med-start`,
  `diagnosis`, `symptoms`, …), exactly one leaf per post, created by `/healthypress:setup`.
  `needs-triage` is where unclassifiable records go — though WordPress.com does not let the MCP
  change a site's *default* category, so that safety net depends on `/log` always assigning one
  explicitly.
- **Tags** are the entities, namespaced with a closed set of prefixes: `dx-` condition, `rx-`
  medication (generic names only), `sx-` symptom, `dr-` clinician, `fac-` facility, `test-` test,
  `alg-` allergen, `sys-` body system, `src-` provenance, `precision-` date precision.
- **Titles** carry the one headline fact (`Lab: Lipid panel — LDL 142`) because there are no
  structured numeric fields.
- **Every post has an excerpt**, so "show me June" is one cheap query.
- **The post date is the event date.** Fuzzy dates get a midpoint sentinel plus a `precision-` tag
  plus the user's own words recorded verbatim. Ranges become two posts.
- **One page, Health Summary.** It's yours. Write it however you like, or ask the agent to
  summarize your record onto it. Nothing regenerates or overwrites it behind your back. Facts with
  no usable year — "broke my wrist as a kid, no idea when" — go in its `## Undated facts` section,
  since they can't become dated posts. The front page stays the blog listing of your records.

## What this is not

### Not a legally protected medical record

**HIPAA does not apply to this site.** HIPAA covers healthcare providers, insurers, and their
business associates. It does not cover an individual's own website. Health information you put here
gets **no legal health-privacy protection**.

### Where the data model breaks down

Honest limits of doing this with core WordPress:

- **No structured numeric fields.** Post meta isn't in the MCP facade, so real charting of A1c over
  time is out of reach. Numbers live in titles and body text.
- **No relations.** A lab doesn't belong to a visit. You can tag both, but nothing links them.
- **No referential integrity on tags.** `rx-lisinoprill` is a valid tag forever.
- **No unit normalization.** mg/dL and mmol/L sit side by side.
- **One kind per post is lossy.** A visit where three things happened becomes several posts.
- **Post revisions aren't exposed**, so an overwritten section isn't recoverable through the MCP.
- **Fuzzy dates collide** on shared midpoint sentinels, making same-day ordering arbitrary.
- **Scale ceiling** in the low thousands of posts before listing and derivation get slow.
- **One site = one person.** No family records on one site.
- **Sharing is all-or-nothing, and read implies write.** Adding someone is a WordPress.com web UI
  operation (People → Invite). An Editor can read every private record — the whole timeline, not a
  subset — but WordPress has no read-only-private role, so they can also edit and trash records. A
  Viewer is read-only but sees only *published* content, which on this site is the Health Summary
  page and nothing else.
- **No FHIR, no CCD.** Export is a WXR file no clinician can read or import.
- **Deleting a post is a 30-day trash**, not a delete. Media deletion *is* permanent.

### Export

There is no export operation in the MCP. To get your data out, use the WordPress.com web UI:
**wordpress.com/export/`<your-site>`** produces a WXR (XML) file containing your posts, pages,
categories, tags, and media references. It is a complete backup and a useless clinical document —
no clinician will read it. Do it anyway, periodically, so the record isn't only in one place.
