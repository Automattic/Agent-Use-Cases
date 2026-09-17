---
description: Read-only factual report on what's in the health record, plus a data-hygiene and privacy audit — counts and patterns, never interpretation
allowed-tools: mcp__wpcom__wpcom-mcp-site, mcp__wpcom__wpcom-mcp-content-authoring, mcp__wpcom__wpcom-mcp-user-management, mcp__plugin_healthypress_wpcom__wpcom-mcp-site, mcp__plugin_healthypress_wpcom__wpcom-mcp-content-authoring, mcp__plugin_healthypress_wpcom__wpcom-mcp-user-management, Read, Skill
arguments:
  - name: scope
    description: What to report on — a tag, category, date range, or "hygiene" for just the audit. If omitted, reports on everything.
    required: false
---

# Review the Health Record

A factual report over what has been logged, and an audit of the record's own integrity. **This
command writes nothing.** It fixes nothing on its own — it tells the user what it found and offers
the command that would fix it.

Load `health-content-model` and `wpcom-mcp-operations` before step 1; `health-summary-pages` before
step 3.

## Step 1: Read the record

Page through post listings with the taxonomy filters the `scope` argument implies — all the way to a
short page, so counts are real. Listings return dates and excerpts, so this is cheap; only fetch
full posts for the hygiene checks that need the body.

Build reporting on **post listings plus taxonomy filters**, not on content search: whether search
covers non-public posts, and whether it can filter by category or tag, is unverified (see
`docs/wpcom-mcp-notes.md`). If a search-based shortcut would be faster, verify it returns private
posts on this site before trusting it.

## Step 2: The factual report

**Descriptive only. Counts, dates, frequencies, co-occurrences, gaps. Then stop.**

```
Health record — <site> — 247 records, 2011-03 to 2026-09

By category
  visit              41        lab                 38
  symptoms           52        med-start           19
  ...

Most tagged
  sys-neurological   61        dx-migraine         34
  dr-amara-okafor    22        rx-lisinopril        7

Recent activity
  Last 30 days       6 records (4 symptoms, 1 lab, 1 visit)
  Last 12 months     58 records

Patterns in what was logged
  sx-migraine        12 records in the last 12 months; 9 fell on a weekday
  test-a1c           4 records, roughly every 6 months, most recent 2026-07-14
  Longest gap        2013-08 to 2015-02 (18 months with no records)
```

Rules for this section, without exception:

- A count, a date, a frequency, a co-occurrence, or a gap. Nothing else.
- "9 of 12 migraines logged fell on a weekday" is allowed. "Your migraines may be work-related" is
  not — that's a hypothesis, and this command doesn't make them.
- Never call a value or a trend high, low, normal, improving, worsening, or concerning.
- Never rank causes, never suggest a next test, never suggest a conversation topic beyond offering to
  assemble records.
- A gap is a gap in the *record*, not a gap in care. Say it that way.
- If the user asks what a pattern means, decline per the Boundaries below and offer to assemble the
  relevant records for their clinician.

## Step 3: The hygiene audit

Report each, with the specific post IDs and a suggested fix. Fix nothing automatically.

1. **Near-duplicate tags.** Compare tags within each prefix for small edit distances, singular/plural
   pairs, and brand-vs-generic pairs (`rx-lipitor` next to `rx-atorvastatin`). These split derived
   page rows silently, so this is the highest-value check here.
2. **Missing required `## Details` keys**, per `references/record-types.md`, per leaf category.
3. **Posts in `needs-triage`** — list them so they can be reclassified.
4. **Tag-rule violations** — no `sys-`, no `src-`, more than one of either, more than 8 tags, an
   unprefixed tag, a `precision-` tag on an exact date or missing from a sentinel date.
5. **Drafts.** Any health post whose status isn't `private` — a draft is invisible to the derived
   pages and silently undercounts them. Also flag any post that is **public**: that's a privacy
   incident, and it goes at the top of the report, not in a list.
6. **Future-dated posts** (status `future`), which vanish from listings until their date arrives.
7. **Stale or drifted pages.** For each derived page, compare its footer's source post IDs and
   timestamp against the posts that exist now. Report `stale` (new posts since generation) separately
   from `drifted` (body disagrees with its own sources — hand-edited).
8. **Orphaned derivations** — a `med-stop` with no `med-start`, a `resolution` with no `diagnosis`, a
   `med-change` with no open run.
9. **Fuzzy-date collisions** — two records for the same tag on the same midpoint sentinel, where the
   ordering is arbitrary.
10. **Attachments** — posts whose `## Attachments` section names a file that isn't attached, and
    media with identifying filenames (a date pattern plus a generic type is the rule).

## Step 4: The privacy check

Read back and report, every run:

```
Privacy
  Visibility         Private ✓
  blog_public        -1 ✓
  Search engines     Discouraged ✓
  Comments           Off ✓
  Newsletter email   Off ✓
  Subscribers        0 ✓
  Users              you (Administrator); dana@example.com (Editor); 1 pending invite
```

If **anything** is off — visibility not Private, subscribers present, a public post, newsletter
enabled — put it at the **top** of the whole report, before the counts, and say what to run:
`/healthypress:setup` to re-harden, `/healthypress:share list` to review people.

## Step 5: Close

Summarize in a sentence: how many records, how many hygiene items, whether privacy is clean. Offer
the fixes as commands the user can run. Do not run them.

---

This command reports facts about what was logged. It does not interpret them or advise.

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
