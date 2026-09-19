---
description: Read-only factual report on what's in the health record, plus a data-hygiene and privacy audit — counts and patterns, never interpretation
allowed-tools: mcp__wpcom__wpcom-mcp-site, mcp__wpcom__wpcom-mcp-content-authoring, mcp__wpcom__wpcom-mcp-user-management, mcp__plugin_healthypress_wpcom__wpcom-mcp-site, mcp__plugin_healthypress_wpcom__wpcom-mcp-content-authoring, mcp__plugin_healthypress_wpcom__wpcom-mcp-user-management, mcp__wpcom__authenticate, mcp__wpcom__complete_authentication, mcp__plugin_healthypress_wpcom__authenticate, mcp__plugin_healthypress_wpcom__complete_authentication, Read, Skill, Bash
arguments:
  - name: scope
    description: What to report on — a tag, category, date range, or "hygiene" for just the audit. If omitted, reports on everything.
    required: false
---

# Review the Health Record

A factual report over what has been logged, and an audit of the record's own integrity. **This
command writes nothing.** It fixes nothing on its own — it tells the user what it found and offers
the command that would fix it.

Load `health-content-model` and `wpcom-mcp-operations` before step 1.

## Step 0.5: Connect to the MCP server

If the only `wpcom` tools available are `authenticate` and `complete_authentication`, the server
isn't authorized yet. Call `authenticate`, open the returned URL in the user's browser with Bash
(`open` / `xdg-open` / `start`), tell them the grant is account-wide, and wait for them to approve.
See `wpcom-mcp-operations` for the full handshake, including the fallback when the callback doesn't
land. Don't send the user off to configure anything — do it for them.

## Step 1: Read the record

Page through post listings with the taxonomy filters the `scope` argument implies — all the way to a
short page, so counts are real. Listings return dates and excerpts, so this is cheap; only fetch
full posts for the hygiene checks that need the body.

Build reporting on **post listings plus taxonomy filters**, not on content search: whether search
covers non-public posts, and whether it can filter by category or tag, is unverified. If a
search-based shortcut would be faster, verify it returns private posts on this site before trusting
it.

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
   pairs, and brand-vs-generic pairs (`rx-lipitor` next to `rx-atorvastatin`). These split one
   entity into two silently, so this is the highest-value check here.
2. **Missing required `## Details` keys**, per `references/record-types.md`, per leaf category.
3. **Posts in `needs-triage`** — list them so they can be reclassified.
4. **Tag-rule violations** — no `sys-`, no `src-`, more than one of either, more than 8 tags, an
   unprefixed tag, a `precision-` tag on an exact date or missing from a sentinel date.
5. **Drafts.** Any health post whose status isn't `private` — a draft is invisible to listings and
   silently undercounts this report. Also flag any post that is **public**: that's a privacy
   incident, and it goes at the top of the report, not in a list.
6. **Future-dated posts** (status `future`), which vanish from listings until their date arrives.
7. **Orphaned sequences** — a `med-stop` with no `med-start`, a `resolution` with no `diagnosis`, a
   `med-change` with no open run.
8. **Fuzzy-date collisions** — two records for the same tag on the same midpoint sentinel, where the
   ordering is arbitrary.
9. **Attachments** — posts whose `## Attachments` section names a file that isn't attached, and
   media with identifying filenames (a date pattern plus a generic type is the rule).

## Step 4: The privacy check

Read back and report, every run:

```
Privacy
  Visibility         Private ✓
  blog_public        -1 ✓
  Search engines     Discouraged ✓
  Comments           Off ✓
  Users              you (Administrator); dana@example.com (Editor); 1 pending invite
```

If **anything** is off — visibility not Private, or a public post — put it at the **top** of the
whole report, before the counts, and say what to run: `/healthypress:setup` to re-harden,
`/healthypress:share list` to review people.

## Step 5: Close

Summarize in a sentence: how many records, how many hygiene items, whether privacy is clean. Offer
the fixes as commands the user can run. Do not run them.

---

This command reports facts about what was logged. It does not interpret them or advise.
