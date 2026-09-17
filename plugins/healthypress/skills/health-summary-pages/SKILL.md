---
name: health-summary-pages
description: How the derived pages of a HealthyPress site are computed from posts - the set arithmetic for Current Medications, Allergies, Conditions, Care Team, Immunizations, Emergency Summary and Health Summary; the generated-page footer and Sources contract; drift detection before overwriting; partial vs. full regeneration; and where undated facts live. Use when refreshing, regenerating, or reading a health summary page, or after writing health posts that change current state.
---

# Health Summary Pages

Posts are the ledger. Pages are the view. **Every page is computed from posts; none is authored by
hand.** A page that disagrees with the posts is a bug in the derivation, not a correction to be
preserved.

Load `health-content-model` for the taxonomy these rules operate on.

## Quick reference

| Page | Derived from |
|---|---|
| Current Medications | `med-start` minus later `med-stop`, per `rx-` tag |
| Allergies & Intolerances | latest post in `allergies` per `alg-` tag |
| Conditions | `diagnosis` minus later `resolution`, per `dx-` tag, plus a resolved section |
| Care Team | latest `encounters` post per `dr-` tag |
| Immunization Record | all `immunizations` posts, newest first per vaccine |
| Emergency Summary | allergies + current meds + active conditions, severity-first |
| Health Summary | front page: counts, recent activity, links, undated facts |
| About This Site | static; explains the model and the boundaries. Not derived. |

## Set arithmetic, precisely

**Current Medications.** For each `rx-` tag: take all posts in `med-start`, `med-change`, `med-stop`
carrying it, ordered by post date. Walk forward. A `med-start` opens or reopens the row;
`med-change` updates dose/frequency on the open row; `med-stop` closes it. The drug is current iff
the last event is not a `med-stop`. Show: generic name, current dose, frequency, route, prescriber,
started (the date of the `med-start` that opened the current run), and the source post ID for each.
If a `med-change` has no preceding `med-start`, show the row and flag it as `started before this
record began` — do not invent a start.

**Allergies & Intolerances.** One row per `alg-` tag, from the **most recent** post in `allergies`
with that tag: substance, reaction (quoted), severity as stated, date (with precision noted if
fuzzy), confirmed by. Earlier reactions to the same substance are listed as prior events under the
row, not merged. Never drop an allergen; there is no "resolved" for allergies in this model.

**Conditions.** For each `dx-` tag: `diagnosis` posts minus any `resolution` post with a later date.
Two sections — **Active** (no later resolution) and **Resolved** (with the resolution date and the
stated outcome). A `resolution` with no matching `diagnosis` goes under Resolved flagged
`no diagnosis record`.

**Care Team.** One row per `dr-` tag, from the most recent post in any `encounters` leaf carrying it:
name, specialty as recorded, facility, most recent visit date, count of visits recorded. A `dr-` tag
that appears only on a `referral` is listed under **Referred, not yet seen**.

**Immunization Record.** All `immunizations` posts grouped by vaccine, newest first, with dose in
series and next-due if the record stated one. Never compute a next-due date yourself — only
transcribe one the record printed.

**Emergency Summary.** Ordered for someone reading it in a hurry: allergies (severe first, by the
severity the record states, not by your judgment) → current medications → active conditions → care
team contacts → nothing else. Say at the top, in plain language, that it is generated from
self-entered records, may be incomplete, and is not a medical record.

**Health Summary** (front page). Record counts by category · the most recent 10 records as titles
with dates and links · links to every other page · **undated facts** · a line stating when it was
generated. No narrative, no interpretation, no "trends".

## Undated facts live on pages

A fact with no usable year gets **no post** — a post with a fabricated date is worse than a fact
without one. It goes in an `## Undated facts` section on the Health Summary, as a bullet quoting the
user, e.g. `- "Broke my left wrist as a kid, no idea what year."` This section is the one part of a
generated page whose source is the conversation rather than a post, so it must be **carried forward
verbatim on every regeneration**. Read the existing section before regenerating and re-emit it.

## The sync contract

Every generated page ends with exactly this, and a `## Sources` list of the post IDs it was computed
from:

```markdown
## Sources

Posts: 412, 388, 377, 301

_Derived from 4 posts as of 2026-09-17 14:22 site time. Do not edit by hand — ask HealthyPress to refresh._
```

The post ID list is what makes drift detectable and the page auditable. Keep it even when it is long.

## Drift detection — before every regeneration

1. Read the page as it currently exists.
2. Recompute what the page *should* say from the posts named in its `## Sources` footer.
3. If the body disagrees with that recomputation, the page was hand-edited (or written by something
   that didn't follow this contract).

On drift: **stop. Do not overwrite.** Show the user a diff — what the page says now, what
regeneration would say — and ask which to keep. If they want their edit preserved, the honest answer
is that it will be lost on the next refresh unless it becomes a post: offer to create the post that
would make the derivation produce it.

A page with no footer at all is treated as drift, not as empty.

## Partial vs. full regeneration

Regenerate only the pages the just-written tags can affect. Full regeneration is for the end of a
history import and for an explicit refresh request.

| Tag or category just written | Regenerate |
|---|---|
| `rx-` on `med-start` / `med-change` / `med-stop` | Current Medications, Emergency Summary, Health Summary |
| `alg-` or category `allergies` | Allergies, Emergency Summary, Health Summary |
| `dx-` on `diagnosis` / `resolution` | Conditions, Emergency Summary, Health Summary |
| `dr-` on any `encounters` leaf or `referral` | Care Team, Health Summary |
| category `immunizations` | Immunization Record, Health Summary |
| anything else | Health Summary only |

Health Summary is on every list because it carries the counts and the recent-records list. If a
write turned out to be a no-op (an identical record already existed), skip regeneration entirely
rather than bumping timestamps for nothing.

## Writing the pages

- Pages are pages, not posts: no date, no category, no tags.
- Page status follows the site: on a private site, the pages are as private as the site. The
  Emergency Summary is not an exception — do not publish it publicly to make it shareable.
- Prefer a section-addressable replace over rewriting the whole page, so a failure mid-refresh
  leaves a partially-correct page rather than an empty one — but always rewrite `## Sources` and the
  footer in the same pass as the content they describe.
- Never write a number on a page that you can't point to a post for. `## Sources` is the proof.
- Tables are fine; charts and trend lines are not — there are no structured numeric fields behind
  them, and a rendered trend implies an interpretation this plugin doesn't make.

## Common gotchas

- A post left as a draft (status not set explicitly) is invisible to derivation, so a medication can
  quietly vanish from Current Medications. If a page count looks wrong, check for drafts first.
- Fuzzy dates share midpoint sentinels, so same-day ordering between two `precision-year` records is
  arbitrary. When two events for the same tag collide on one sentinel date, show both and flag the
  ordering as uncertain rather than picking a winner.
- A renamed or near-duplicate tag splits one row into two. Surface it; `/healthypress:review` is
  where it gets fixed.
- Regenerating with an incomplete read (unpaged listing) silently drops rows. Page all the way to a
  short page before computing.
- Post revisions aren't exposed, so an overwritten page is not recoverable. That's why drift stops
  rather than merges.

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
