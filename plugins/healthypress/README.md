# HealthyPress

Run a private WordPress.com site as your personal health record.

WordPress already ships nearly everything a health log needs: dated posts, hierarchical categories,
tags, media attachments, user roles, and private site visibility. HealthyPress is the knowledge that
turns those primitives into a coherent record — a closed taxonomy, a naming discipline, derived
summary pages, and a privacy-first setup sequence.

**The plugin carries knowledge, not code.** No PHP, no custom post types, no companion WordPress
plugin. Everything it does, it does through the WordPress.com MCP server against stock core
features.

> **Read "What this is not" below before you put real health information into it.** Some of it is
> legal, some of it is technical, and all of it is load-bearing.

## Install

```
/plugin marketplace add Automattic/Agent-Use-Cases
/plugin install healthypress@wordpress-agent-use-cases
```

Then:

1. At WordPress.com, go to **Preferences → AI and MCP** and enable MCP access. Nothing works until
   you do.
2. Run `/mcp` in Claude Code and connect `wpcom`. Claude Code handles the OAuth sign-in itself —
   there's no token to create or paste.
3. Run `/healthypress:setup`.

Available on all paid WordPress.com plans. **Free sites get 30 days of MCP access from site
creation** — fine for trying this out, not enough for a long history import.

## Commands

| Command | What it does |
|---|---|
| `/healthypress:setup` | Picks or creates the site, then runs the **privacy gate** — launch, set Private, verify, discourage search engines, disable comments and newsletter email, verify zero subscribers, neutral title, timezone — then creates the taxonomy and the pages and prints a privacy report. **Idempotent: re-run it any time as a privacy audit.** Stops hard if the site can't be made Private. |
| `/healthypress:log` | The daily driver. One event in, one private post out: interview, classify, resolve tags, compute the date, compose, read the whole record back for confirmation, save, attach files, refresh the affected pages. |
| `/healthypress:backfill` | Guided history intake, era by era and system by system. Checkpoints every ~10 records, keeps a resumable captured list, never re-asks about a declined topic, regenerates all pages once at the end. Warns about the free-plan 30-day cliff before starting. |
| `/healthypress:share` | Care team access. Only runs on a Private site. Invites as **Editor** (full read of the whole timeline — and, unavoidably, edit and trash rights) or **Viewer** (read-only, published pages only), in plain language about the tradeoff. Also lists, changes, and revokes. |
| `/healthypress:review` | Read-only factual report — counts, frequencies, co-occurrences, gaps — plus a hygiene audit (near-duplicate tags, missing required fields, untriaged posts, drafts, drifted pages) and a privacy check. Never interprets. |

## Skills

These load on their own when the topic comes up; you don't invoke them.

| Skill | Content |
|---|---|
| `health-content-model` | The schema: posts vs. pages, the category taxonomy, the tag namespaces, titles, excerpts, dates, the sectioned body, media rules. |
| `wpcom-mcp-operations` | WordPress.com MCP mechanics: the facade pattern, runtime schema discovery, status and date defaults, write confirmation, pagination, launch-then-privatize ordering, trash vs. delete. |
| `health-intake-interview` | How to ask: era and system recall scaffolding, fuzzy-date elicitation, non-leading questions, verbatim capture, pacing and exits. |
| `health-summary-pages` | How each derived page is computed, the footer and `## Sources` contract, drift detection, partial vs. full regeneration. |

## How the record is shaped

**A post is an event that happened at a point in time. A page is a current-state projection with no
date. Posts are the ledger; pages are the view.**

So a medication isn't a post — *starting* lisinopril is a post, *stopping* it is another, and
"Lisinopril 10 mg daily" is a row on the Current Medications page computed from the two. Same for
conditions (diagnosis minus resolution), allergies (latest reaction per allergen), and the care team.

- **Categories** are the kind of record: a closed, hierarchical list (`visit`, `lab`, `med-start`,
  `diagnosis`, `symptoms`, …), exactly one leaf per post. `needs-triage` is the site default, so
  nothing gets lost.
- **Tags** are the entities, namespaced with a closed set of prefixes: `dx-` condition, `rx-`
  medication (generic names only), `sx-` symptom, `dr-` clinician, `fac-` facility, `test-` test,
  `alg-` allergen, `sys-` body system, `src-` provenance, `precision-` date precision.
- **Titles** carry the one headline fact (`Lab: Lipid panel — LDL 142`) because there are no
  structured numeric fields.
- **Every post has an excerpt**, so "show me June" is one cheap query.
- **The post date is the event date.** Fuzzy dates get a midpoint sentinel plus a `precision-` tag
  plus the user's own words recorded verbatim. Ranges become two posts.
- **Derived pages** — Health Summary, Current Medications, Allergies & Intolerances, Conditions,
  Care Team, Immunization Record, Emergency Summary, About This Site — are generated, never
  hand-edited. Each ends with a footer naming the posts it came from. Edit one by hand and the next
  refresh stops and shows you the diff instead of overwriting.

## What this is not

### Not a legally protected medical record

**HIPAA does not apply to this site.** HIPAA covers healthcare providers, insurers, and their
business associates. It does not cover an individual's own website. Health information you put here
gets **no legal health-privacy protection** — it's protected like any other content on a hosted
website, which is to say much less.

Concretely:

- **Automattic staff can access site content**, as with any hosted WordPress.com site.
- **Hosted content is subject to legal process** — subpoenas, warrants, civil discovery.
- **Your conversation transcripts contain the same health information.** Every record you log passes
  through a conversation with Claude, and that transcript is a second copy of your health data,
  stored somewhere you don't control, under a different policy than the site. This is not a
  side effect you can turn off — it's inherent to using an agent to do the writing.

### Attachments may not be private

Media URLs on a private WordPress.com site have **not been verified** to require authentication.
Until that's confirmed on your own site, treat every uploaded file's URL as potentially reachable by
anyone who has it.

So: **redact before uploading.** Medical record numbers, full date of birth, insurance member IDs,
street addresses, and account numbers. HealthyPress also renames files to a non-identifying pattern
(`2026-03-04-lab-1.pdf`) because filenames appear in URLs — but that only protects the filename, not
the contents.

### You are one setting away from public

Site visibility is a single setting. A plugin, a theme, a support interaction, or a mis-click can
change it. Nothing in this design prevents that; what it does is make it **checkable**:

- `/healthypress:setup` is idempotent — re-run it whenever you want it re-hardened and re-verified.
- `/healthypress:review` prints a privacy check every time, and puts any failure at the top.
- Every health record is created with status `private`, so even a public site doesn't publish the
  individual records — the derived pages, however, would become visible.

Do both periodically. Privacy here is a thing you maintain, not a thing you set.

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
- **No FHIR, no CCD.** Export is a WXR file no clinician can read or import.
- **Deleting a post is a 30-day trash**, not a delete. Media deletion *is* permanent.

### Export

There is no export operation in the MCP. To get your data out, use the WordPress.com web UI:
**wordpress.com/export/`<your-site>`** produces a WXR (XML) file containing your posts, pages,
categories, tags, and media references. It is a complete backup and a useless clinical document —
no clinician will read it. Do it anyway, periodically, so the record isn't only in one place.

### If any of that is unacceptable

Then don't use this. The honest alternative for someone whose main requirement is privacy is a
**local encrypted notes file** — an encrypted disk image, a password manager's secure notes, or an
encrypted notes app — kept on a device you control, with no hosted copy and no agent transcript.
You lose the derived pages, the searchability, and the care team sharing. You gain actual control.

HealthyPress is the right tool when you want a queryable, shareable, organized health history and
you've read the above and accepted it.

## See also

`build-with-wordpress` (`Automattic/claude-code-wordpress.com`) in the official marketplace is for
**building** WordPress.com sites. HealthyPress is about **using** one as an application. They don't
overlap, and you can install both.

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
