---
name: wpcom-mcp-operations
description: How to drive the WordPress.com MCP server - the STRAP facade pattern (one tool, many operations, with list and describe actions), runtime schema discovery, post status and date defaults, write confirmation, pagination, site visibility and launch ordering, trash vs. permanent delete, and the dual plugin tool-name prefixes. Use whenever calling WordPress.com MCP tools, or when an MCP call returns an unexpected parameter, permission, or validation error.
---

# WordPress.com MCP Operations

Mechanics of the WordPress.com MCP server at `https://public-api.wordpress.com/wpcom/v2/mcp/v1`.
No content conventions here — this skill is about the interface.

## Quick reference

| Situation | Do this |
|---|---|
| About to use an operation for the first time this session | `action: describe` it, then call it |
| Don't know which facade owns an operation | `action: list` on the likely facade |
| Creating a post | Set `status` explicitly. The default is draft. |
| Creating a dated record | Pass the event date; verify the response echoes it |
| A write is refused pending confirmation | Show the user what will be written, get a yes, retry with the confirmation flag |
| Listing more than a screenful | Page explicitly; don't assume you got everything |
| Deleting | Assume trash, not delete. Say so to the user. |

## The facade pattern

There is no `wpcom_create_post` tool. The server exposes roughly eight **facade** tools, each
covering a domain, and you select behavior with an `operation` parameter plus an `action`:

- `wpcom-mcp-site` — site settings, status, launch, visibility
- `wpcom-mcp-content-authoring` — posts, pages, categories, tags, media, sections, search
- `wpcom-mcp-user-management` — users, roles, invites, subscribers
- `wpcom-mcp-create-site` — provisioning a new site

Treat that mapping as a hint, not a contract. Facade names and their operation sets change.

Every facade supports two meta actions:

- `action: list` — enumerate the operations this facade supports
- `action: describe` — return the parameter schema for one operation

**The documentation is explicit that schemas evolve and the live `describe` response is the source
of truth.** So:

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

Command frontmatter must list **both** forms in `allowed-tools`, or the command breaks depending on
how the server was installed. Whichever prefix is present at runtime is the one to call.

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

Commands that call these tools must list `authenticate` and `complete_authentication` in
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
- Claude Code handles OAuth 2.1 + PKCE + dynamic client registration natively for `type: "http"`
  servers, so there is no token to configure. If auth is the problem, it shows up as `/mcp` not
  connecting, not as a bad parameter.

## Writes

**Posts and pages default to draft.** Always set `status` explicitly on create. A draft is invisible
to listings that filter by status, so a forgotten status silently loses the record.

**Confirmation.** Write operations may require a confirmation flag (commonly `user_confirmed`). The
honest pattern: compose the full record, show it to the user, get an explicit yes, then send the
write with the flag set. Whether one confirmation covers a batch or is required per write is
**unverified** — assume per-write. To avoid stalling a long import, batch the *asking*: present ten
composed records at once, take one approval, then send ten writes each carrying the flag.

**Backdating.** Pass the event date on create rather than creating and then editing. Whether a
backdated date is honored alongside a non-public status is load-bearing for history imports and is
**unverified** — so after the first backdated write of a session, read the post back and confirm the
stored date matches what you sent. If it was coerced to now, stop and tell the user before writing
more.

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
- **Search has unverified scope.** Whether the content-search operation covers non-public posts, and
  whether it can filter by category or tag, is unverified. Until it's confirmed on the site you're
  working with, build reporting on post listings with taxonomy filters, which definitely work.
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
- Do not write anything sensitive until a **read-back** says `private`.

If a site cannot be confirmed private, that is a stop condition, not a warning.

### What `settings.update` can and cannot write

Writable: `blogname`, `blogdescription`, `blog_public` (−1 private / 0 public-discouraged / 1
public), `timezone_string`, `date_format`, `time_format`, `start_of_week`, `default_role`,
`users_can_register`, `comment_registration`, `show_on_front`, `page_on_front`.

**Not writable — do not claim otherwise to the user:**

- **`default_category`** doesn't exist. A site's default post category cannot be changed.
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
- Timezone: site-local time is what the post date means. Confirm the site timezone once per session
  before computing any date.
- Undocumented rate limits exist. On a long import, pace the writes and checkpoint often so a 429
  costs one record, not the whole run.
- Media URLs on a private site may or may not require authentication. Until verified on the site in
  question, treat every uploaded file's URL as potentially public.

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
