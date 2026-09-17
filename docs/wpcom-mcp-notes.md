# WordPress.com MCP — shared field notes

Notes on the WordPress.com MCP server (`https://public-api.wordpress.com/wpcom/v2/mcp/v1`) that
every use case in this repo shares. Behavior verified here belongs in a skill; behavior still open
belongs in the table below.

**Rule:** a claim in this file is either marked **verified** with a date and how it was checked, or
it's **open**. There is no third state. Do not write a schema here and treat it as a contract — the
docs are explicit that schemas evolve and the live `describe` response is the source of truth.

## What's settled

| Fact | Basis |
|---|---|
| STRAP facade pattern: ~8 facade tools, each taking an `operation` plus `action: list` / `action: describe` | WordPress.com MCP docs |
| Facades observed: `wpcom-mcp-site`, `wpcom-mcp-content-authoring`, `wpcom-mcp-user-management`, `wpcom-mcp-create-site` | WordPress.com MCP docs |
| Always treat the live `describe` response as the source of truth | stated explicitly in the docs |
| MCP access is opt-in per user at WordPress.com → Preferences → AI and MCP | WordPress.com docs |
| Available on all paid plans; free sites get 30 days from site creation | WordPress.com docs |
| Tool availability respects the connected user's WordPress role | WordPress.com docs |
| The **Private** visibility option only appears after a site is launched | WordPress.com docs |
| A freshly provisioned site is in Coming Soon, which exposes a shareable preview link | WordPress.com docs |
| WordPress.com emails new posts to subscribers; non-public statuses don't trigger it | WordPress core / WordPress.com behavior |
| Post deletion is a 30-day trash; media deletion is permanent | WordPress core behavior |
| Claude Code handles OAuth 2.1 + PKCE + dynamic client registration natively for `type: "http"` | Claude Code docs |
| Plugin-scoped MCP tools are renamed, so both `mcp__<server>__*` and `mcp__plugin_<plugin>_<server>__*` must appear in `allowed-tools` | observed in `automattic-claude-code-plugins` |

## Open questions

Resolve each against a real site using `action: describe` and a scratch site, then move the answer
into the table above with the date and the method. Three of these are load-bearing — marked **★**.

| # | Question | Why it matters | How to check |
|---|---|---|---|
| 1 ★ | Does post-create accept a backdated `date` (or `date_gmt`) **together with** status `private`, honored rather than coerced? | The entire history-import use case depends on it. | Create a post dated 2015 with status `private`, then read it back and compare the stored date and status to what was sent. |
| 2 ★ | Exact privacy ordering: can provisioning produce a private site directly, or must it be launched first? Does launching a not-yet-private site expose it, and for how long? Does `settings.update`'s `blog_public` disagree with the visibility operation, and which wins? | Determines whether `/healthypress:setup` has an exposure window it can't close. | On a scratch site: read status → launch → immediately read status → set visibility private → read back. Separately set `blog_public` and the visibility operation to conflicting values and read both. |
| 3 ★ | Are media URLs on a Private site protected? | Attachments are the most sensitive content in a health record. | Upload a file to a private site, copy its URL, then open it logged out (private window or a different browser/device). |
| 4 | Does the content-search operation return non-public posts, and can it filter by category or tag? | If not, reporting must be built on post listings plus taxonomy filters. | Search for a string that exists only in a private post. Then try a search scoped to a category and to a tag. |
| 5 | `user_confirmed` semantics — can one confirmation cover a batch of writes, or is it required per write? | Shapes the backfill checkpoint design; a per-write requirement makes a 200-record import a 200-prompt conversation. | Send two writes after a single confirmation and see whether the second is refused. |
| 6 | Does the free-site 30-day MCP window run from **site creation** or from **first MCP use**? | Changes the warning `/healthypress:backfill` gives, and whether a fresh free site is usable at all. | Create a free site, wait, then first use MCP against it and observe when access ends. Or ask the WordPress.com team — cheaper than waiting 30 days. |
| 7 | Undocumented rate limits on a long run (say 200 sequential post creates). | Determines write pacing and checkpoint frequency during a history import. | Write 50 posts in sequence against a scratch site, logging response times and any 429s, then extrapolate. |

### How the current design handles each while they're open

| # | Mitigation in the shipped commands and skills |
|---|---|
| 1 | `/log` reads the created post back and compares the stored date to the intended one; `/backfill` verifies the first backdated write of the session and **stops** on mismatch before writing more. |
| 2 | `/setup` sets both `blog_public` and the visibility operation, then treats a **read-back** as the only evidence, and stops hard if it doesn't report Private. No content is written before that read succeeds. |
| 3 | The plugin README states attachments are potentially public; `/log` enforces non-identifying filenames and reminds the user to redact identifiers before uploading. |
| 4 | `/review` is built on post listings plus taxonomy filters, and says so. |
| 5 | `/backfill` assumes **per-write** confirmation and batches the *asking* (10 composed records, one approval, 10 flagged writes). |
| 6 | `/setup` warns on a free plan and continues; `/backfill` stops and asks before starting. |
| 7 | `/backfill` paces writes and checkpoints every ~10 records so a rate limit costs one record, not the run. |

## Practical gotchas worth keeping

- **Posts and pages default to draft.** Always set status explicitly on create. A draft is invisible
  to status-filtered listings, so a forgotten status silently loses the record.
- **Never send a future date.** WordPress converts it to a scheduled post (status `future`) that
  disappears from listings until that day. A future date is almost always a timezone bug.
- **Page listings explicitly.** Keep requesting until a page comes back short; a first page that
  looks complete usually isn't.
- **Read back, don't trust the write.** A parameter `describe` doesn't list can be silently dropped
  rather than rejected.
- **Term creation is not idempotent in a predictable way.** Search for a slug first, and read back
  the term ID you actually got.
- **Confirm the site timezone once per session** before computing any date.
- **Post revisions aren't exposed**, so a section replace is not undoable from the site.
