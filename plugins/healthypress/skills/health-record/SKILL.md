---
name: health-record
description: The content model and WordPress.com MCP mechanics for running a WordPress site as a personal health record - what becomes a post, the closed category taxonomy, namespaced entity tags, title/excerpt/date rules, the sectioned post body, plus the facade pattern, runtime schema discovery, post status and date defaults, write confirmation, site visibility, and trash vs. delete. Use whenever the user wants to log, record, file, or organize anything health related ("log my headache", "add my lab results", "I started a new medication", "record yesterday's visit"), whenever reading health records back out of the site, and whenever a WordPress.com MCP call returns an unexpected parameter, permission, or validation error.
---

# Health Record

Everything HealthyPress knows: the schema the record is stored in, and the interface it is stored
through. `/healthypress:setup` creates and hardens the site; `/healthypress:log` records events.
This skill is what both of them reach for.

Stock WordPress only — posts, pages, categories, tags, media. No custom post types, no post meta,
no plugins. Follow the schema exactly; consistency is the only thing that makes the record queryable
later.

## Quick reference

| Question | Answer |
|---|---|
| Post or page? | Everything you record is a post. The only page is Health Summary, which the user owns. |
| Category | Exactly one leaf from the closed list. Never invent one. Unsure → `needs-triage`. |
| Tags | Namespaced entities only. Search before create. Max 8. Exactly one `sys-` and one `src-`. |
| Title | `<Type>: <Subject> — <most specific fact>`, ≤70 chars, no date (except journal/fuzzy). |
| Excerpt | Required on every post. One factual line, ≤160 chars. |
| Date | The clinical event date, local site time, never in the future. |
| Status | `private` on every health record. Always set it explicitly. |
| Body | H2 sections in order: Summary, Details, Attachments, Follow-up, Provenance. |
| About to use an MCP operation for the first time this session | `action: describe` it, then call it |
| Don't know which facade owns an operation | `action: list` on the likely facade |
| A write is refused pending confirmation | Show the user what will be written, get a yes, retry with the confirmation flag |
| Listing more than a screenful | Page explicitly; don't assume you got everything |
| Deleting | Assume trash, not delete. Say so to the user. |

## The one rule everything follows

> **A post is an event that happened at a point in time. Posts are the whole record. Current state
> is not stored anywhere — it is read out of the posts when someone asks for it.**

A medication is not a post. *Starting* lisinopril is a post; *stopping* it is another post. "Am I
currently on lisinopril?" is answered by reading `med-start` posts tagged `rx-lisinopril` and
checking for a later `med-stop` with the same tag. Conditions work the same way
(diagnosis − resolution), as do allergies (latest reaction per allergen) and the care team.

The site has exactly one page, **Health Summary**. It is not derived and not maintained by this
plugin: the user writes it, or asks the agent to summarize onto it. Never overwrite it as a side
effect of logging. The one place HealthyPress touches it is the `## Undated facts` section, which it
appends to.

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
| `needs-triage` | where a record goes when no leaf fits. **Not** the site default — `default_category` is not writable through the MCP, so this only catches records `/healthypress:log` assigns to it explicitly |

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
   matches → ask the user. Creating a near-duplicate is silent and permanent-ish: it splits one
   entity into two and nothing downstream can detect it. This is the only defense against
   `rx-lisinopril` / `rx-lisinoprill`.
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
time goes in `## Provenance`. Always local site time. Exact day but no time → `12:00:00`.

Partial dates use midpoint sentinels, carrying the fuzziness in three places: a `precision-` tag, a
title parenthetical, and a verbatim `Date reported as:` line in `## Details`.

| User says | Date | Tag | Title |
|---|---|---|---|
| "sometime in March 2019" | `2019-03-15 12:00` | `precision-month` | `(Mar 2019)` |
| "spring of 2019" | `2019-04-15 12:00` | `precision-month` | `(spring 2019)` |
| "sometime in 2015" | `2015-07-01 12:00` | `precision-year` | `(c. 2015)` |
| no usable year at all | **don't create a post** | — | append to `## Undated facts` on the Health Summary page |

**Ranges become two posts.** "Plantar fasciitis 2015 to 2018" is a `diagnosis` at 2015 and a
`resolution` at 2018, both tagged `dx-plantar-fasciitis`. A single midpoint post makes it impossible
to tell later whether the condition is active or resolved.

## Post body — stable H2 sections

Section-addressable edits exist in the MCP, so the section headings are part of the contract. Use
this order and omit empty sections: `## Summary` (1–3 sentences, subjective content quoted verbatim)
· `## Details` (per-kind key list) · `## Attachments` · `## Follow-up` · `## Provenance`.

Required `## Details` keys per leaf category, with worked examples: `references/record-types.md`.

## Media

Upload, attach to exactly one event post, reference from `## Attachments`. **Filenames must be
non-identifying** — they appear in URLs. Use `YYYY-MM-DD-<generic-type>-<n>.<ext>`
(`2026-03-04-lab-1.pdf`); descriptive text goes in the media title/caption/alt, which aren't in the
URL and are searchable. Never transcribe a whole document — extract the required Details keys, quote
the Impression verbatim, and note "full document attached (4 pages)". Image generation has no role
here. **Media deletion is permanent and unrecoverable.**

**Verified 2026-09-22: media URLs on a private site are protected** — direct, resized and Photon-CDN
requests all 403 anonymously. That protects the URL, not the contents; anyone granted access can
open the file. Redact MRNs, full date of birth, insurance IDs, and addresses before uploading.

## The recorder stance

This plugin records and organizes. It does not diagnose, interpret, or advise.

- **"Flag (per report)" is transcription, never judgment.** Copy what the lab printed and attribute
  it to the report. Never write "this is high" on your own authority.
- **Subjective content is quoted, never paraphrased into clinical register.** "My chest felt tight
  and weird" does not become "reports chest tightness."
- **Never call a value or a trend high, low, normal, improving, worsening, or concerning** — not in
  a post body, and not when reading the record back. Counts, dates, frequencies, co-occurrences, and
  gaps are fine. "9 of 12 migraines logged fell on a weekday" is a count. "Your migraines may be
  work-related" is a hypothesis, and not yours to make. A gap is a gap in the *record*, not a gap in
  care — say it that way.

---

# The WordPress.com MCP interface

Mechanics of the server at `https://public-api.wordpress.com/wpcom/v2/mcp/v1`.

## The facade pattern

There is no `wpcom_create_post` tool. The server exposes **facade** tools — 28 of them when last
counted, on 2026-09-22 — each covering a domain, and you select behavior with an `operation`
parameter plus an `action`. Four matter here:

- `wpcom-mcp-site` — site settings, status, launch, visibility
- `wpcom-mcp-content-authoring` — posts, pages, categories, tags, media, sections, search
- `wpcom-mcp-user-management` — users, roles, invites, subscribers
- `wpcom-mcp-create-site` — provisioning a new site

Treat that mapping as a hint, not a contract. Facade names and their operation sets change.

Every facade supports two meta actions: `action: list` enumerates the operations it supports, and
`action: describe` returns the parameter schema for one operation.

**The documentation is explicit that schemas evolve and the live `describe` response is the source
of truth.** So:

**`describe` on `wpcom-mcp-content-authoring` is site-gated** (verified 2026-09-22): it needs a real
`wpcom_site`, and an unknown one fails with "Site not found" before any schema comes back.
`wpcom-mcp-site` describes without one. So resolve the site first, then discover content schemas.

> **Never hardcode a facade's parameter names from memory or from a document, including this one.
> `describe` the operation before its first use in a session, and pass exactly the parameters the
> live schema names.**

If a parameter you expected is absent from `describe`, it does not exist. Don't send it, and don't
work around its absence by inventing a different one — tell the user the capability isn't available
through the MCP and offer the WordPress.com web UI instead.

### Both tool-name prefixes

When these tools are reached through a plugin, Claude Code renames them. The same tool can arrive as
either:

```
mcp__wpcom__<facade>
mcp__plugin_healthypress_wpcom__<facade>
```

Skill frontmatter must list **both** forms in `allowed-tools`, or the skill breaks depending on how
the server was installed. Whichever prefix is present at runtime is the one to call.

## Authentication — the facade tools don't exist until you authorize

**Verified 2026-09-17.** On a fresh install the server exposes only two tools, `authenticate` and
`complete_authentication`. The facade tools are absent entirely — so an unauthenticated server looks
like a server with no capabilities, not like a server returning auth errors. Claude Code's native
OAuth handling for `type: "http"` does **not** complete this on its own.

The handshake, which you should just perform rather than asking the user to go configure something:

1. Call `authenticate`. It returns an authorization URL.
2. **Open the URL in the user's browser** with Bash — `open` on macOS, `xdg-open` on Linux, `start`
   on Windows. Don't make them copy a 600-character URL.
3. Wait for them to confirm, then call a facade operation. The tools appear automatically once the
   local callback lands.
4. If the facade tools still aren't there, the callback didn't land. Ask for the full
   `http://localhost:<port>/callback?...` URL from their address bar and pass it to
   `complete_authentication`.

**The grant is account-wide.** The requested scopes cover sites, posts, media, taxonomy, users,
stats, and notifications across the user's entire WordPress.com account — not the one site being
managed. Say so before they approve; it's their decision to make, and a separate account is a
reasonable response to it.

Skills that call these tools must list `authenticate` and `complete_authentication` in
`allowed-tools` under **both** prefixes, alongside the facades.

## Access and gating

- Authorizing is separate from **enabling**. The user enables MCP access at WordPress.com →
  **Preferences → AI and MCP**. If facade calls fail with an authorization error *after* a
  successful handshake, that's the setting to point at.
- Available on all paid plans. **Free sites get 30 days from site creation** (whether the clock
  starts at creation or at first MCP use is unverified). Warn before starting any long,
  multi-session project on a free site.
- Tool availability also respects the connected user's WordPress role. An operation missing from
  `list` may be a permissions artifact rather than a server change — check the role before concluding
  the feature is gone.
- **An operation can also be switched off per account.** `list` returns a `disabled_operations`
  array alongside the live ones, each with a reason such as "This operation is disabled in your MCP
  settings" (verified 2026-09-22 on `wpcom-mcp-site`, where the `connection.*` operations were
  off). This is reported explicitly rather than by silent absence, so read both arrays.
- Claude Code handles OAuth 2.1 + PKCE + dynamic client registration natively for `type: "http"`
  servers, so there is no token to configure. If auth is the problem, it shows up as `/mcp` not
  connecting, not as a bad parameter.

## Writes

**Posts and pages default to draft.** Always set `status` explicitly on create — `private` for every
health record. A draft is invisible to listings that filter by status, so a forgotten status silently
loses the record. `private` also avoids triggering subscription email; publishing a health record
publicly is the one unrecoverable mistake in this plugin.

**Confirmation.** Write operations may require a confirmation flag (commonly `user_confirmed`). The
honest pattern: compose the full record, show it to the user, get an explicit yes, then send the
write with the flag set. **Verified 2026-09-17: the flag is per write and does not carry**, so set
it on every call. Batching several writes into one call is what lets one approval cover them.

**Verified 2026-09-22: `user_confirmed` accepts the boolean `true`, or the strings `'true'`,
`'yes'`, `'on'`, `'1'`, case-insensitive. It explicitly rejects a free-form approval phrase** —
passing the user's actual words ("Yes, create it") fails the write. Extract their consent into one
of the accepted forms.

**Which writes need the flag is discoverable, not guesswork.** `action: list` on a facade returns a
`safety_policy.applies_to` array naming the operations that require it. On `wpcom-mcp-site`
(2026-09-22) that is `settings.update`, `theme.set`, `manage-site.launch`,
`manage-site.set-visibility`, the `monitor.*` and `account-protection.*` toggles,
`newsletter.update_settings`, `scan.run`, and the `connection.*` operations. Read the policy rather
than assuming.

**Timezone.** Site-local time is what the post date means. Confirm the site timezone once per
session before computing any date.

**Backdating.** Pass the event date on create rather than creating and then editing. **Verified
2026-09-22:** a past date is honored alongside `private` and is not coerced to now.

**`meta` is not storage for you.** `describe` lists it, so the unlisted-parameter rule misses it:
only platform keys exist (SEO, newsletter, social), you cannot add one, and an unknown key returns
`success` and stores nothing. Numbers go in the title and body.

**Never send a future date.** WordPress converts it to a scheduled post, status `future`, which
disappears from normal listings until that day arrives. If a date computes to the future because of
a timezone mismatch, fix the timezone, not the date.

**Section edits.** A section-replace operation exists for post bodies, keyed on heading text. It's
the right tool for amending one part of a record. Post revisions are not exposed through the MCP, so
a replace is not undoable from the site — read the section first if the old content matters.

## Reads

- **Listings return more than IDs.** Post listings include dates and excerpts, so a well-populated
  excerpt turns "what happened in June" into one call rather than N fetches. Prefer one list call
  with filters over many get calls.
- **Page explicitly.** Ask for a page size and keep requesting until a page comes back short. A
  first page that looks complete usually isn't.
- **Search cannot see the record.** `content.search` covers published content only (verified
  2026-09-22: a token inside a private post returns zero) and filters by post type, not taxonomy.
  Every record here is `private`. Build all reporting on `posts.list` with `status` and taxonomy
  filters.
- **Status filters.** When listing, name the statuses you want. Defaults skew toward public content.

## Site visibility and launch ordering

**Verified 2026-09-17, and it is the opposite of what the WordPress.com docs imply.** A freshly
provisioned site already reports:

```
launch_status: "unlaunched"
visibility:    "private"
is_private:    true
```

So: **Private is not gated behind launching, and a new site is not in Coming Soon.** The sequence
"launch, then privatize" is wrong — a site can be private whether launched or not, so treat the two
settings as independent. Whether to launch is a product decision, not a privacy requirement.

- `blog_public` in site settings and the visibility operation **agree** on a fresh site; setting
  `blog_public: -1` on an already-private site comes back as `unchanged`. Set both anyway on an
  older site, then read back and trust the read.
- `get_status` can report a legacy `discourage_search` visibility (public, search engines
  discouraged). `set_visibility` **cannot target it** — from that state the only choices are
  `public` or `private`. `private` is strictly stronger, so this never matters here.
- The visibility operation refuses to make a site private when it has active subscribers unless
  `user_confirmed: true` is passed. Read the subscriber count first rather than forcing it.
- **Read privacy from `get_status` only.** Verified 2026-09-22: `wpcom-user-sites` reported
  `is_private: false` for a site that `manage-site.status` and `settings.get` both reported private
  at the same moment. An anonymous request returns HTTP **200** with a login wall, so a status code
  is not evidence either.
- Do not write anything sensitive until a **read-back** says `private`.

If a site cannot be confirmed private, that is a stop condition, not a warning.

### What `settings.update` can and cannot write

Writable, confirmed against the live `describe` on 2026-09-22: `blogname`, `blogdescription`,
`blog_public` (−1 private / 0 public-discouraged / 1 public), `timezone_string`, `date_format`,
`time_format`, `start_of_week`, `default_role`, `users_can_register`, `comment_registration`,
`show_on_front`, `page_on_front`. `blogname` and `blogdescription` also accept the REST-convention
aliases `name` and `description`; pass both forms and the canonical key wins.

**Not writable — do not claim otherwise to the user:**

- **`default_category`** doesn't exist — absent from the live schema, re-confirmed 2026-09-22. A
  site's default post category cannot be changed, which is why `needs-triage` only ever catches what
  a skill assigns to it explicitly.
- **`default_comment_status` / `default_ping_status`** don't exist. **Comments and pingbacks cannot
  be disabled through the MCP.** `comment_registration: true` only requires a logged-in account.
  Point the user at Settings → Discussion in wp-admin.
- **`users_can_register: false` silently fails.** It returns `success: true` with a
  `before: true, after: 0` transition and the value stays `true` on read-back. Reproduced twice;
  probably inert on Simple sites. This is a **false success report** — the strongest possible
  argument for reading every write back.

A static front page needs the page **published first**; `settings.update` rejects a draft as
`page_on_front`.

## Deletes

- Post deletion is a **30-day trash**, not a delete. Content still exists on the server. Tell the
  user that plainly rather than saying "deleted".
- Media deletion is **permanent and unrecoverable**.
- There is no bulk undo. Confirm before any multi-item delete, and enumerate what will go.

## Common gotchas

- Sending a parameter that `describe` doesn't list usually fails as a validation error, but can also
  be silently dropped. Verify the effect by reading back, not by trusting a success response.
- A tag or category "create" with an existing slug may return the existing term or a near-duplicate
  depending on the operation. Search first; read back the term ID you actually got.
- A future date turns a record into a scheduled post that vanishes from listings.
- Undocumented rate limits exist. On a long import, pace the writes and checkpoint often so a 429
  costs one record, not the whole run.
- Don't put PHI-ish identifiers (MRN, full DOB, insurance ID, street address) in titles, slugs, or
  filenames. Those travel further than the body.
