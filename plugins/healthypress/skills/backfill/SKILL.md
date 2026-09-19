---
name: backfill
description: Guided intake of past health history — era by era, system by system, with checkpoints and a resumable captured list. Use when the user wants to backfill, import, or walk through their past health history rather than log a single new event.
allowed-tools: mcp__wpcom__wpcom-mcp-site, mcp__wpcom__wpcom-mcp-content-authoring, mcp__plugin_healthypress_wpcom__wpcom-mcp-site, mcp__plugin_healthypress_wpcom__wpcom-mcp-content-authoring, mcp__wpcom__authenticate, mcp__wpcom__complete_authentication, mcp__plugin_healthypress_wpcom__authenticate, mcp__plugin_healthypress_wpcom__complete_authentication, AskUserQuestion, Read, Skill, Bash
---

# Backfill Health History

Walks the user's past into the record. This is a long, tiring conversation, so it is built around
checkpoints, a visible captured list, and an easy exit at every turn.

Load `health-content-model` and `wpcom-mcp-operations` before step 2.

## Step 0: Safety check

If the user describes an **acute emergency in progress**, stop the workflow immediately, tell them to
contact emergency services or a crisis line now, and do not resume until they say it's resolved.
Then STOP. Backfill is about the past, but people bring up the present while talking about the past.

## Step 0.5: Connect to the MCP server

If the only `wpcom` tools available are `authenticate` and `complete_authentication`, the server
isn't authorized yet. Call `authenticate`, open the returned URL in the user's browser with Bash
(`open` / `xdg-open` / `start`), tell them the grant is account-wide, and wait for them to approve.
See `wpcom-mcp-operations` for the full handshake, including the fallback when the callback doesn't
land. Don't send the user off to configure anything — do it for them.

## Step 1: Check the site and the plan — before any work

1. Read the site status. If visibility is not **Private**, STOP and point at `/healthypress:setup`.
2. Read the site plan. **If it is a free site**, stop and ask before doing anything:

   > `<site>` is on the free plan, and MCP access on free sites is limited to **30 days from site
   > creation**. A history import is the one thing here that takes multiple sessions, so it's the
   > thing most likely to get cut off partway through — leaving a half-entered record and no way to
   > finish it through me.
   >
   > Options: upgrade the plan first, do the import in one sitting knowing it may be incomplete, or
   > wait.

   Ask with `AskUserQuestion` and **do not start until they choose to proceed.** If they decline,
   STOP.

3. Confirm the site timezone once. Every historical date depends on it.

## Step 2: Set the scope and the shape of the session

Read existing records first (a post listing with dates and excerpts is one cheap call) so you never
ask about something already captured. Show the user what's already there.

Then agree on a plan, in the user's words. Default shape:

**Era pass** — now/active → last year → last five years → each earlier decade → childhood.
**System pass** — heart · lungs · digestion · bones and joints · head and nerves · hormones · skin ·
kidneys and urinary · mental health · everything else.
**Forgettables pass** — surgeries · ER visits · hospitalizations · drug reactions · vaccines ·
stopped medications · pregnancies · screenings.

Tell them up front: one question at a time, "I don't know" is a fine answer, and they can stop at
any point without losing what's been captured.

If `args` is `resume`, find the captured list from the previous session (step 4) and pick up where
it left off. Otherwise use `args` (e.g. "medications", "the 2010s"), if given, to set where to
start — or ask if it wasn't given.

## Step 3: Work one record at a time

For each record, follow `/healthypress:log`'s composition rules exactly — one leaf category, tags
searched before created, midpoint sentinels and `precision-` tags for fuzzy dates, required
`## Details` keys, required excerpt, status `private`, event date in site-local time.

Two differences from `/log`, because 60 individual confirmations would make this unusable:

- **Batch the asking, not the writing.** Compose up to 10 records, show them all as a compact list
  (title · date · category · tags), take one approval for the batch, then write them one at a time
  each carrying the write-confirmation flag. Whether one confirmation can cover a batch server-side
  is unverified — assume per-write and set the flag on each.
- **Full records on request.** The compact list is the default; if the user wants to see a full
  record before it's written, show it.

Verify the **first** backdated write of the session by reading it back. If the stored date doesn't
match what you sent, STOP and tell the user before writing 200 records with wrong dates.

Pace the writes. Undocumented rate limits exist; a 429 should cost one record, not the batch.

## Step 4: Checkpoint every ~10 records

After each batch, show the running captured list and offer the exit first:

```
Captured so far — 23 records

  2015    diagnosis        Plantar fasciitis (c. 2015)          #412
  2016    procedure        Right knee arthroscopy                #413
  ...

Still open from our plan: childhood, vaccines, mental health
Declined: hospitalizations

Stop here · take a break (I'll note where we are) · keep going
```

Make stopping the easy option, not the awkward one. If they stop, print the captured list plus
what's still open, tell them `/healthypress:backfill resume` continues from here, and go to step 5.

**Never ask about anything the user declined.** Keep a declined list and honor it for the rest of the
session and any resumed one. If they said "I don't want to get into the hospital stuff," that topic
is closed until they reopen it.

If they seem tired or distressed, offer the break before they have to ask for it.

## Step 5: Undated facts, once, at the end

If any facts in this session had no usable year, they got no post. Offer to append them, as bullets
quoting the user verbatim, to the `## Undated facts` section of the Health Summary page. **Append
only** — read the section, add to it, and leave the rest of the page exactly as the user left it.

HealthyPress does not generate or regenerate the Health Summary page. If the user wants it brought
up to date with everything just captured, they can ask — that's an ordinary request, not part of
this skill.

## Step 6: Report

- Records created, by category.
- New tags created (so the user can spot a misspelling while it's still fresh).
- Records that landed in `needs-triage`.
- What's still open from the plan, and what was declined.
- Any undated facts added to the Health Summary page.
- One line: `/healthypress:review` will audit the result; `/healthypress:backfill resume` continues.

---

This skill interviews and records. It does not interpret history or advise on it.
