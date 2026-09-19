---
name: health-content-model
description: The canonical content model for a WordPress site used as a personal health record - what becomes a post vs. a page, the closed category taxonomy, namespaced entity tags, title/excerpt/date rules, and the sectioned post body. Use whenever the user wants to log, record, file, or organize anything health related ("log my headache", "add my lab results", "I started a new medication", "record yesterday's visit"), with or without a slash command, and whenever reading health records back out of the site.
---

# Health Content Model

HealthyPress stores a personal health record in stock WordPress: posts, pages, categories, tags,
media. No custom post types, no post meta, no plugins. This skill is the schema. Follow it exactly —
consistency is the only thing that makes the record queryable later.

For MCP mechanics (facades, `describe`, status defaults, backdating) load `wpcom-mcp-operations`.
For page derivation load `health-summary-pages`. For intake technique load `health-intake-interview`.

## The one rule everything follows

> **A post is an event that happened at a point in time. A page is a current-state projection with
> no date. Posts are the ledger; pages are the view. Pages are always derived from posts, never
> authored by hand.**

A medication is not a post. *Starting* lisinopril is a post; *stopping* it is another post.
"Lisinopril 10 mg daily" is a row on the Current Medications page, computed from `med-start` posts
tagged `rx-lisinopril` minus later `med-stop` posts with the same tag. Conditions work the same way
(diagnosis − resolution), as do allergies (latest reaction per allergen) and the care team.

## Quick reference

| Question | Answer |
|---|---|
| Post or page? | Something happened on a date → post. Current state → page (derived). |
| Category | Exactly one leaf from the closed list. Never invent one. Unsure → `needs-triage`. |
| Tags | Namespaced entities only. Search before create. Max 8. Exactly one `sys-` and one `src-`. |
| Title | `<Type>: <Subject> — <most specific fact>`, ≤70 chars, no date (except journal/fuzzy). |
| Excerpt | Required on every post. One factual line, ≤160 chars. |
| Date | The clinical event date, local site time, never in the future. |
| Status | `private` on every health record. Always set it explicitly. |
| Body | H2 sections in order: Summary, Details, Attachments, Follow-up, Provenance. |

## Categories — kind of record

Hierarchical, closed, exactly one leaf per post. Created once by `/healthypress:setup`.

| Parent | Leaves |
|---|---|
| `encounters` | `visit`, `urgent-care`, `emergency`, `hospitalization`, `procedure`, `therapy` |
| `diagnostics` | `lab`, `imaging`, `screening`, `vitals` |
| `medications` | `med-start`, `med-change`, `med-stop` |
| `conditions` | `diagnosis`, `resolution` |
| `care-admin` | `insurance`, `referral`, `records` |
| `symptoms`, `allergies`, `immunizations`, `journal` | (no children) |
| `needs-triage` | the site default category, so uncategorized records stay findable |

Agents may not invent categories. A record that doesn't fit goes in the nearest leaf with a note in
`## Details`. If a record wants two kinds, it is two records.

## Tags — entities

Closed set of prefixes: `dx-` condition · `rx-` medication (generic name only) · `sx-` symptom ·
`dr-` clinician · `fac-` facility · `test-` lab/imaging test · `alg-` allergen · `sys-` body system
(closed list of 10) · `src-` provenance · `precision-` date precision.

1. Lowercase, ASCII, hyphenated, **singular**, always namespaced. Use a hyphen prefix, not a colon —
   WordPress slugifies `rx:lisinopril` to `rxlisinopril` and the namespace is lost.
2. Generic drug names only; brand goes in the body. This is what keeps `rx-*` a usable key.
3. **Search before create.** Run a tag list/search for the term and reuse what exists. Two plausible
   matches → ask the user. This is the only defense against `rx-lisinopril` / `rx-lisinoprill`.
4. Max 8 tags per post. Every post gets exactly one `sys-` and one `src-`.
5. Never tag severities, dosages, dates, or numbers. The prefix list is closed.

Full vocabularies (all 10 `sys-` values, the `src-` and `precision-` values, worked examples):
`references/taxonomy.md`.

## Titles, excerpts, dates

```
Visit: Dr. Amara Okafor (Cardiology) — echo follow-up
Lab: Lipid panel — LDL 142
Med start: Lisinopril 10 mg daily
Symptom: Migraine — 6h, right-side, aura
Reaction: Amoxicillin — hives (c. 2011)
Journal: 2026-09-14 — rough sleep, 3/10 energy
```

Put the one headline number in the title — with no structured numeric fields, the title is the index.
No date in the title (post listings return it, and dates in titles poison content search), except
journal entries and fuzzy-dated records, which get a human parenthetical.

**Set `excerpt` on every post** — one factual line, ≤160 chars. Post listings return excerpts, so
this turns "show me June" into one cheap call instead of N per-post fetches. Highest-leverage
convention in the model.

**The post date IS the clinical event date** — when it happened, not when it was recorded. Recording
time goes in `## Provenance`. Always local site time. **Never a future date** — WordPress sets status
`future` and the record disappears until that day. Exact day but no time → `12:00:00`.

Partial dates use midpoint sentinels, carrying the fuzziness in three places: a `precision-` tag, a
title parenthetical, and a verbatim `Date reported as:` line in `## Details`.

| User says | Date | Tag | Title |
|---|---|---|---|
| "sometime in March 2019" | `2019-03-15 12:00` | `precision-month` | `(Mar 2019)` |
| "spring of 2019" | `2019-04-15 12:00` | `precision-month` | `(spring 2019)` |
| "sometime in 2015" | `2015-07-01 12:00` | `precision-year` | `(c. 2015)` |
| no usable year at all | **don't create a post** | — | undated fact on the Health Summary page |

**Ranges become two posts.** "Plantar fasciitis 2015 to 2018" is a `diagnosis` at 2015 and a
`resolution` at 2018, both tagged `dx-plantar-fasciitis`. A single midpoint post breaks the
active/resolved derivation.

## Post body — stable H2 sections

Section-addressable edits exist in the MCP, so the section headings are part of the contract. Use
this order and omit empty sections: `## Summary` (1–3 sentences, subjective content quoted verbatim)
· `## Details` (per-kind key list) · `## Attachments` · `## Follow-up` · `## Provenance`.

Required `## Details` keys per leaf category, with worked examples: `references/record-types.md`.

Two hard rules from the recorder stance:

- **"Flag (per report)" is transcription, never judgment.** Copy what the lab printed and attribute
  it to the report. Never write "this is high" on your own authority.
- **Subjective content is quoted, never paraphrased into clinical register.** "My chest felt tight
  and weird" does not become "reports chest tightness."

## Media

Upload, attach to exactly one event post, reference from `## Attachments`. **Filenames must be
non-identifying** — they appear in URLs. Use `YYYY-MM-DD-<generic-type>-<n>.<ext>`
(`2026-03-04-lab-1.pdf`); descriptive text goes in the media title/caption/alt, which aren't in the
URL and are searchable. Never transcribe a whole document — extract the required Details keys, quote
the Impression verbatim, and note "full document attached (4 pages)". Image generation has no role
here. Media deletion is permanent and unrecoverable.

## Common gotchas

- Posts default to draft. Always set status `private` explicitly, or the record is invisible and the
  derived pages silently undercount.
- `private` status also avoids triggering subscription email. Publishing a health record publicly is
  the one unrecoverable mistake in this plugin.
- A future date turns a record into a scheduled post that vanishes from listings.
- Creating a near-duplicate tag is silent and permanent-ish. Search first, every time.
- Post revisions aren't exposed, so a section overwrite is not recoverable from the site.
- Don't put PHI-ish identifiers (MRN, full DOB, insurance ID, street address) in titles, slugs, or
  filenames. Those travel further than the body.
