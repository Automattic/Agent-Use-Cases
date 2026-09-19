---
description: Record one health event — interview, classify, tag, date, compose, confirm, save as private, refresh affected pages
allowed-tools: mcp__wpcom__wpcom-mcp-site, mcp__wpcom__wpcom-mcp-content-authoring, mcp__plugin_healthypress_wpcom__wpcom-mcp-site, mcp__plugin_healthypress_wpcom__wpcom-mcp-content-authoring, mcp__wpcom__authenticate, mcp__wpcom__complete_authentication, mcp__plugin_healthypress_wpcom__authenticate, mcp__plugin_healthypress_wpcom__complete_authentication, AskUserQuestion, Read, Skill, Bash
arguments:
  - name: entry
    description: What happened, in your own words (e.g. "saw Dr. Okafor today about the echo"). If omitted, you will be asked.
    required: false
---

# Log a Health Record

The daily driver. One event in, one private post out, plus a refresh of only the pages that event
can affect.

Load `health-content-model` and `wpcom-mcp-operations` before step 2; load `health-intake-interview`
if anything needs asking, and `health-summary-pages` before step 8.

## Step 0: Safety check, before anything else

Read what the user said. If it describes an **acute emergency in progress** — chest pain, stroke
signs, trouble breathing, anaphylaxis, severe bleeding, overdose, or thoughts of self-harm — stop
the logging workflow immediately, tell them to contact emergency services or a crisis line now, and
do not resume until they say the situation is resolved. Then STOP. Do not create a post, do not ask
clarifying questions about the record.

A past event being described in the past tense ("I went to the ER in March") is a record, not an
emergency. Judge the tense and the timeframe, and if it's genuinely ambiguous, ask whether this is
happening right now.

## Step 0.5: Connect to the MCP server

If the only `wpcom` tools available are `authenticate` and `complete_authentication`, the server
isn't authorized yet. Call `authenticate`, open the returned URL in the user's browser with Bash
(`open` / `xdg-open` / `start`), tell them the grant is account-wide, and wait for them to approve.
See `wpcom-mcp-operations` for the full handshake, including the fallback when the callback doesn't
land. Don't send the user off to configure anything — do it for them.

## Step 1: Confirm the site is set up

Read the site status. If visibility is **not Private**, STOP:

> `<site>` is currently `<status>`, not Private. I'm not going to write health information to a site
> that isn't private. Run `/healthypress:setup` to fix it.

If the site has no HealthyPress categories, say so and point at `/healthypress:setup`. Then STOP.

Note the site timezone; every date below is in site-local time.

## Step 2: Get the event

Use the `entry` argument if given. Otherwise ask, once, openly: "What would you like to record?"

Capture their words verbatim before doing anything else — that text becomes `## Summary`, in quotes.
Then, following `health-intake-interview`, ask **one question at a time** for only the required
`## Details` keys the record type needs and the user hasn't already given. Never lead. `unknown` is
an acceptable value for any key.

If what they described is really two events (a visit *and* the lab that came from it), say so and
offer to record both, one at a time. Never merge two kinds into one post.

## Step 3: Classify to exactly one leaf category

Pick the leaf from the closed list in `health-content-model`. If two leaves are genuinely plausible,
ask with `AskUserQuestion`, offering the two plus "none of these". If nothing fits, use
`needs-triage` and tell the user it's filed for later triage — do not invent a category.

## Step 4: Resolve tags — search before create

1. List/search existing tags for each entity's bare term (`lisinopril`, not `rx-lisinopril`).
2. One plausible match → reuse it.
3. Two or more → ask the user which; mention the near-duplicate so `/healthypress:review` can clean
   it up.
4. None → create it per the naming rules (lowercase, ASCII, hyphenated, singular, namespaced,
   generic drug names only).
5. Enforce: max 8 tags, exactly one `sys-`, exactly one `src-`. If you're over 8, drop the least
   specific entity tag, never the `sys-` or `src-`.

Do not skip the search. A silent near-duplicate tag splits a derived page row in two and nothing
downstream can detect it.

## Step 5: Compute the date

The post date is the **clinical event date** in site-local time, not the recording time.

- Exact day, no time → `12:00:00`.
- Fuzzy → use the midpoint sentinel from `health-content-model`, add the `precision-` tag, add the
  parenthetical to the title, and put a verbatim `Date reported as: "<their words>"` line in
  `## Details`.
- No usable year at all → **do not create a post.** Offer to add it to the Health Summary's
  `## Undated facts` instead.
- A date range → **two posts** (e.g. `diagnosis` then `resolution`). Confirm that with the user and
  do them one at a time.
- **Never a future date.** If the computed date is in the future, it's a timezone or year error —
  ask rather than writing it.

## Step 6: Compose, then read the whole record back

Build the title (`<Type>: <Subject> — <most specific fact>`, ≤70 chars, headline number included),
the excerpt (**required**, one factual line, ≤160 chars), and the body in the fixed section order,
omitting empty sections.

Then show the user the **complete record as it will be saved** — title, excerpt, date, category,
tags, and full body — and ask for a yes. Not a summary of it; the actual text. This read-back is
where wrong tags and wrong dates get caught.

If they want changes, revise and read it back again. Loop until they confirm or ask to abandon it.

## Step 7: Create the post as private

`describe` the post-create operation before its first use this session, then create with **status
explicitly `private`** and the computed date. Posts default to draft, and a draft is invisible to the
derived pages; `private` also avoids triggering subscription email.

Read the created post back and verify: stored date matches what you sent (backdating can be
coerced), status is `private`, category and tags are attached. If the date came back as now instead
of the date you sent, tell the user before writing anything else:

> WordPress stored this record at `<actual>` instead of `<intended>`. Backdating isn't behaving as
> expected on this site, so I've stopped. The record exists but has the wrong date.

Then STOP.

## Step 8: Attachments, if any

For each file the user wants attached:

- Rename to `YYYY-MM-DD-<generic-type>-<n>.<ext>` before upload. **Filenames appear in URLs**, so
  they must not identify the person or the condition.
- Put the descriptive text in the media title, caption, and alt — those aren't in the URL.
- Attach to this one post and list it under `## Attachments`.
- Do not transcribe the whole document. Extract the required `## Details` keys, quote any Impression
  verbatim, and note `full document attached (<n> pages)`.
- Remind the user once per session: **media URLs on a private site are not verified to be
  protected.** Redact MRNs, full date of birth, insurance IDs, and addresses before uploading.

## Step 9: Refresh only the affected pages

Using the tag→page table in `health-summary-pages`, regenerate only the pages this record can
change. Run drift detection on each page first: if a page's body disagrees with its `## Sources`
footer, **stop and show the diff** rather than overwriting.

## Step 10: Report

One short confirmation: the title, the post ID, the date as stored, the category, the tags, and which
pages were refreshed. Then offer to log another.

---

This command records and files what the user tells it. It does not interpret findings or advise.
