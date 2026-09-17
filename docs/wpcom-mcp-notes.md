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
| STRAP facade pattern: facade tools each taking an `operation` plus `action: list` / `action: describe` / `action: execute`. **28 tools, not ~8** | **verified 2026-09-17** — inspected the tool list after authorizing |
| The four facades HealthyPress uses all exist: `wpcom-mcp-site`, `wpcom-mcp-content-authoring`, `wpcom-mcp-user-management`, `wpcom-mcp-create-site`. Also present: `wpcom-user-sites`, `wpcom-mcp-account`, `wpcom-plans-list`, site-builder and domain tools | **verified 2026-09-17** |
| Always treat the live `describe` response as the source of truth | stated explicitly in the docs |
| MCP access is opt-in per user at WordPress.com → Preferences → AI and MCP | WordPress.com docs |
| Available on all paid plans; free sites get 30 days from site creation | WordPress.com docs |
| Tool availability respects the connected user's WordPress role | WordPress.com docs |
| ~~The **Private** option only appears after a site is launched~~ / ~~a fresh site is in Coming Soon~~ — **both false.** A freshly provisioned site reports `visibility: private`, `launch_status: unlaunched`, `is_private: true`. Private is not gated behind launching, and a new site is not in Coming Soon. **Do not launch a HealthyPress site** — launching moves it toward live for no benefit | **verified 2026-09-17** — `manage-site.status` `get_status` + `settings.get` on a site seconds after `site.provision` |
| `blog_public` and the visibility operation **agree** on a fresh site — setting `blog_public: -1` on an already-private site reports it as `unchanged`. No conflict observed | **verified 2026-09-17** |
| `user_confirmed` is required **per write operation**, on every facade. It does not carry across calls; the create-site runbook says its gate "does not auto-clear from earlier confirmations" | **verified 2026-09-17** — `safety_policy` in `wpcom-mcp-site` `list`, and each write schema |
| `settings.update` writes: `blogname`, `blogdescription`, `blog_public` (−1/0/1), `timezone_string`, `date_format`, `time_format`, `start_of_week`, `default_role`, `users_can_register`, `comment_registration`, `show_on_front`, `page_on_front`. **No `default_category`. No `default_comment_status` / `default_ping_status`** | **verified 2026-09-17** — `describe settings.update` |
| **`users_can_register: false` silently fails.** The write returns `success: true` with `before: true, after: 0`, and read-back still reports `true`. Reproduced twice. Probably inert on Simple sites. A false success report, not a dropped parameter | **verified 2026-09-17** — write, read back, repeat |
| A static front page requires the page to be **published first** — `settings.update` rejects a draft as `page_on_front` | **verified 2026-09-17** — create-site runbook, `next_actions` |
| The new site's **subdomain is derived from the title** (lowercase, strip diacritics, drop non-alphanumerics) and is not an input. If the slug is taken, WordPress.com appends a numeric suffix **at provision time**, so the final URL isn't knowable in advance. `subdomain.check` reports availability first | **verified 2026-09-17** — `site.instructions` runbook + `subdomain.check` |
| `create-site` **requires the user to confirm the title and derived URL** before `site.provision`. A fully non-interactive site creation is not possible | **verified 2026-09-17** — `site.instructions` rules |
| Section-addressable edits exist as `post-sections.*` / `page-sections.*`, and the facade recommends them over full updates for partial edits | **verified 2026-09-17** — `wpcom-mcp-content-authoring` tool description |
| WordPress.com emails new posts to subscribers; non-public statuses don't trigger it | WordPress core / WordPress.com behavior |
| Post deletion is a 30-day trash; media deletion is permanent | WordPress core behavior |
| **Authorization is a tool-driven handshake, not native.** An unauthenticated server exposes only `authenticate` and `complete_authentication`; the facade tools are absent entirely. Claude Code's native OAuth for `type: "http"` does not complete it. | **verified 2026-09-17** — installed the plugin locally and inspected the available `wpcom` tools |
| **The OAuth grant is account-wide.** Requested scopes include `global`, `sites`, `posts`, `media`, `taxonomy`, `users`, `stats`, `notifications`, `read` — every site on the account, not the one being managed. Uses a shared Claude Code `client_id`, so the grant isn't identifiable as HealthyPress in the user's connected-apps list. | **verified 2026-09-17** — read the authorization URL returned by `authenticate` |
| Plugin-scoped MCP tools are renamed, so both `mcp__<server>__*` and `mcp__plugin_<plugin>_<server>__*` must appear in `allowed-tools` | observed in `automattic-claude-code-plugins` |

## Open questions

Resolve each against a real site using `action: describe` and a scratch site, then move the answer
into the table above with the date and the method. Questions 2 (privacy ordering) and 5
(`user_confirmed` semantics) were answered on 2026-09-17 and have moved up. Two of the remaining
are load-bearing — marked **★**.

| # | Question | Why it matters | How to check |
|---|---|---|---|
| 1 ★ | Does post-create accept a backdated `date` (or `date_gmt`) **together with** status `private`, honored rather than coerced? | The entire history-import use case depends on it. | Create a post dated 2015 with status `private`, then read it back and compare the stored date and status to what was sent. |
| 3 ★ | Are media URLs on a Private site protected? | Attachments are the most sensitive content in a health record. | Upload a file to a private site, copy its URL, then open it logged out (private window or a different browser/device). |
| 4 | Does the content-search operation return non-public posts, and can it filter by category or tag? | If not, reporting must be built on post listings plus taxonomy filters. | Search for a string that exists only in a private post. Then try a search scoped to a category and to a tag. |
| 6 | Does the free-site 30-day MCP window run from **site creation** or from **first MCP use**? | Changes the warning `/healthypress:backfill` gives, and whether a fresh free site is usable at all. | Create a free site, wait, then first use MCP against it and observe when access ends. Or ask the WordPress.com team — cheaper than waiting 30 days. |
| 7 | Undocumented rate limits on a long run (say 200 sequential post creates). | Determines write pacing and checkpoint frequency during a history import. | Write 50 posts in sequence against a scratch site, logging response times and any 429s, then extrapolate. |

### How the current design handles each while they're open

| # | Mitigation in the shipped commands and skills |
|---|---|
| 1 | `/log` reads the created post back and compares the stored date to the intended one; `/backfill` verifies the first backdated write of the session and **stops** on mismatch before writing more. |
| 3 | The plugin README states attachments are potentially public; `/log` enforces non-identifying filenames and reminds the user to redact identifiers before uploading. |
| 4 | `/review` is built on post listings plus taxonomy filters, and says so. |
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
- **An unauthenticated server is indistinguishable from an empty one.** Check for the presence of
  `authenticate` before concluding a capability is missing or a facade was renamed.
- **Post revisions aren't exposed**, so a section replace is not undoable from the site.
