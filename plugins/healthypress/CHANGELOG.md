# Changelog

All notable changes to HealthyPress are documented here.

## 0.2.0 — 2026-09-19

### Removed

- **`/healthypress:backfill` is gone.** With it: the era / system / forgettables recall passes, the
  batch-of-10 pacing, the checkpoint block and running captured list, the declined-topics list
  honored across resumed sessions, the `args: resume` contract, and the free-plan gate that
  *blocked* a long import. The underlying 30-day fact survives in `health-record`; the blocking gate
  does not. Backfilling history now means running `/healthypress:log` repeatedly.
- **`/healthypress:review` is gone.** With it: the two report templates and the **nine-item hygiene
  audit** — near-duplicate tag detection, missing required `## Details` keys, `needs-triage` strays,
  tag-rule violations, drafts and public posts, `future`-dated posts, orphaned
  `med-stop`/`resolution` sequences, fuzzy-date collisions, and attachment mismatches. This was the
  plugin's only inverse-of-the-schema knowledge and there is no replacement. What survives is the
  never-interpret rule, now three lines in `health-record` under `## The recorder stance`.
- **`/healthypress:share` is gone.** With it: the Editor-vs-Viewer disclosure. Worth stating
  plainly, because it's a real gap — WordPress has no read-only-private role, so granting a
  clinician read access to the private timeline also grants edit and trash rights, and nothing in
  the plugin will now say so before it happens. Sharing is a WordPress.com web UI operation now;
  the tradeoff is documented in the README instead.
- **`health-content-model` and `wpcom-mcp-operations` are gone as separate skills**, merged into
  `health-record`.

### Changed

- **Three skills: `setup`, `log`, `health-record`.** Down from seven, and from ~1,390 lines of skill
  prose to ~925.
- **`health-record` is the content model and the MCP mechanics in one file.** The schema half comes
  from `health-content-model`, the interface half from `wpcom-mcp-operations`, with every "Verified
  2026-09-17" finding and every explicit "unverified" flag intact. `references/taxonomy.md` and
  `references/record-types.md` moved under it unchanged apart from stale cross-references.
- **The duplication is gone.** The "Connect to the MCP server" block had been copied verbatim into
  four files, the not-Private stop gate appeared three times in three wordings, and
  search-before-create appeared three times. One copy each now.
- `/healthypress:log` and `/healthypress:setup` are otherwise unchanged in behavior; they just load
  `health-record` instead of two separate reference skills.

## 0.1.10 — 2026-09-19

### Changed

- **`setup`, `log`, `backfill`, `share`, and `review` are now skills, not slash commands.** The
  files moved from `commands/*.md` to `skills/<name>/SKILL.md`, and `marketplace.json` lists all
  seven skills under a single `skills` array — the `commands` array is gone.
- **They still run via `/healthypress:setup` and friends** — a skill named `setup` responds to the
  same slash form a command did — but they can now also trigger from plain language ("log my
  headache", "set up a private health site") the way `health-content-model` and
  `wpcom-mcp-operations` already did.
- **Named arguments became a single freeform `args` string.** `log`'s `entry`, `review` and
  `backfill`'s `scope`, and `share`'s `action`/`email` are no longer declared frontmatter
  arguments; each skill now reads whatever free text `args` carries (or asks, if it's omitted) at
  the point it used to read its argument.

## 0.1.9 — 2026-09-19

### Removed

- **The `health-summary-pages` skill is gone**, along with everything built on it: the set
  arithmetic for each derived page, the `## Sources` footer contract, drift detection, and the
  tag→page partial-regeneration table.
- **`/healthypress:setup` no longer creates eight pages.** Current Medications, Allergies &
  Intolerances, Conditions, Care Team, Immunization Record, Emergency Summary, and About This Site
  are all gone.
- **`/healthypress:log` step 9 (refresh affected pages) and `/healthypress:backfill` step 5
  (regenerate all pages) are gone.** Logging a record no longer writes to any page.
- **`/healthypress:review` no longer audits pages** for staleness or drift.
- **The `health-intake-interview` skill is gone.** The non-leading-question table, era and system
  recall scaffolding, the fuzzy-date elicitation ladder, the pacing/exit rules, and the
  uncertainty-recording table went with it. The plugin ships two skills now:
  `health-content-model` and `wpcom-mcp-operations`.
  - `/healthypress:log` keeps the rules it depended on inline: one question at a time, ask open,
    ask for the field rather than a guessed value, `unknown` is acceptable.
  - `/healthypress:backfill` already spelled out the era, system, and forgettables passes inline,
    so its plan shape is unchanged; it just no longer cites the skill as the source.
  - No replacement guidance exists for *how* to elicit a fuzzy date conversationally. The mechanics
    of storing one (midpoint sentinels, `precision-` tags, verbatim `Date reported as:`) remain in
    `health-content-model`.

### Changed

- **One page: Health Summary.** `/healthypress:setup` creates it published, seeded with a short
  intro, an empty `## Undated facts` section, and the boundaries that used to live on About This
  Site. It is not derived. The user maintains it, or asks the agent to summarize their health onto
  it — either way nothing regenerates or overwrites it on a schedule.
- **The blog listing stays as the front page.** Setup no longer sets `show_on_front` /
  `page_on_front`.
- **Undated facts still go on the Health Summary page**, but by **appending** to its
  `## Undated facts` section rather than regenerating the page.
- `health-content-model`'s one rule is now "posts are the whole record; current state is read back
  out of them when asked" rather than "posts are the ledger, pages are the view."
- `/healthypress:share` describes what a Viewer sees as the Health Summary page rather than a set
  of derived pages.

## 0.1.8 — 2026-09-18

### Removed

- **`/healthypress:setup` no longer suggests disabling comments.** Commenting is a normal frontend
  feature of the site, not something the command should point users toward turning off. The
  `comment_registration: true` setting stays (it just keeps anonymous commenters out on an
  already-private site), but the instruction to point at Settings → Discussion in wp-admin, and the
  "Fix in wp-admin" block in the example privacy report, are gone.
- **Dropped the "default category cannot be set" note and report line.** The `needs-triage` safety
  net for unclassifiable records already depends on `/healthypress:log` always assigning a leaf
  explicitly and on `/healthypress:review` auditing for strays — neither depends on setup
  announcing the gap, so it's no longer called out as a warning.

## 0.1.7 — 2026-09-18

### Changed

- **`/healthypress:setup` asks for a memorable site name instead of generating a random one.** Step
  2 now asks the user for a short word (first name, nickname, etc.) and builds
  `healthypress-<word>` as both the title and the derived URL slug, rather than
  `healthypress-<4 random chars>`.
- **Added an explicit approval pause before site creation.** The command shows the proposed title
  and the `would_be_url` from `subdomain.check` and waits for a yes before calling
  `site.provision`. It is the only approval pause in the command — everything from provisioning
  through the end of the privacy gate still runs without further pauses.
- Removed the now-stale "keep the title neutral/dull" guidance and the step-4 rule that rewrote
  `blogname` if it looked identifying, since an identifying, memorable title is now the intended
  default. The agent is instructed to say the memorable-vs-identifying trade-off out loud once,
  before the user picks a word.

## 0.1.6 — 2026-09-18

### Removed

- **`/healthypress:setup` no longer attempts `users_can_register: false`.** It was a known
  false-success write on Simple sites anyway; registration is left at the WordPress.com default.
- **Approval gates dropped from the settings-hardening batch and the category-creation batch.**
  Both still pass `user_confirmed: true` on every write (the facade requires it), but the command no
  longer pauses to show the field/category list and wait for a yes first.

### Added

- **`/healthypress:setup` now trashes WordPress's own default placeholder content** — the sample
  "About" page ("This is an example of a page...") and the "Hello World!" post that WordPress.com
  creates on every new site — before printing the privacy report.

### Changed

- **The generated site-name base changed from `personal-log` to `healthypress`** (e.g.
  `healthypress-<4 random chars>`).

## 0.1.5 — 2026-09-18

### Removed

- **The repo-wide `docs/` directory is gone**, including `docs/wpcom-mcp-notes.md` (the shared
  verified/open-questions log) and `docs/use-case-template/`. References to it in
  `wpcom-mcp-operations` and `/healthypress:review` were trimmed to drop the dead pointer while
  keeping the underlying unverified-behavior caveats.

## 0.1.4 — 2026-09-17

### Removed

- **Newsletter/subscription email is no longer treated as a privacy gate.** `/healthypress:setup`
  dropped the step that verified zero subscribers and disabled subscription email;
  `/healthypress:review`'s privacy check dropped the "Newsletter email" and "Subscribers" lines and
  no longer flags them as out of posture. A private site already limits who can see or subscribe to
  it, and an invited care team member may want the notifications.

## 0.1.3 — 2026-09-17

### Removed

- **`/healthypress:setup` no longer has an audit mode.** The `site` argument and the branch that
  re-hardened and re-verified an existing HealthyPress site are gone; setup now always provisions a
  brand new site. Re-auditing an existing site is planned as its own command later.

## 0.1.2 — 2026-09-17

First live run of `/healthypress:setup` against a real WordPress.com site, which falsified the
plan's central privacy assumption.

### Fixed

- **The privacy gate no longer launches the site.** The plan asserted that a fresh site sits in
  Coming Soon and that Private only becomes available after launching, so setup ran
  launch → privatize → verify. Verified against a live site: a freshly provisioned site already
  reports `visibility: private`, `launch_status: unlaunched`, `is_private: true`. Launching is
  unnecessary and moves the site *toward* live — the command as shipped would have caused the exact
  exposure the gate exists to prevent. Setup now reads status, changes visibility only if it isn't
  already private, and never launches.
- Corrected the same launch-first ordering in `wpcom-mcp-operations`.

### Changed

- **Taxonomy creation in `/healthypress:setup` is now written against the verified API.** `parent`
  takes a numeric category ID, not a slug, so the five container categories must be created first
  and their IDs captured before any leaf. The step now reads existing terms by slug for idempotency,
  requests `include_fields` to keep responses small, takes one user approval for the whole batch
  (`user_confirmed` is per write and does not carry), and paces the writes in batches.
- The hardening settings are now applied as a single batched `settings.update` with one user
  approval, since `user_confirmed` is required per write operation.
- Setup now closes open registration, sets `Y-m-d` / `H:i` formats, and detects the timezone from
  the local system instead of asking.

### Added

- Honest gap reporting in the privacy report. Three things the plan assumed were settable are not:
  the **default post category** (no `default_category` field), **comments and pingbacks off** (no
  `default_comment_status` / `default_ping_status` — only "require login to comment"), and
  search-engine discouragement as its own setting (it's the `blog_public: 0` *public* state).
  The report now names these as gaps and points at wp-admin.
- A recorded false-success case: **`users_can_register: false` returns `success: true` with a
  `before/after` transition and does not persist.** Reproduced twice. The report shows the
  read-back value, never the write response.

### Notes

- `docs/wpcom-mcp-notes.md`: open questions 2 (privacy ordering) and 5 (`user_confirmed` semantics)
  are answered and moved to the verified table, along with nine other findings — 28 facade tools
  rather than ~8, subdomain derived from the title, front page must be published first, and the
  full `settings.update` field list. Questions 1, 3, 4, 6, and 7 remain open.

## 0.1.1 — 2026-09-17

First run against a live install, which corrected two assumptions.

### Changed

- **`/healthypress:setup` now performs the WordPress.com authorization itself.** An unauthenticated
  MCP server exposes only `authenticate` and `complete_authentication` — the facade tools are absent
  entirely, and Claude Code's native OAuth for HTTP servers does not complete the handshake. Setup
  now calls `authenticate`, opens the URL in the user's browser, and continues, instead of telling
  them to go configure `/mcp`. The same handshake was added to the other four commands and
  documented in `wpcom-mcp-operations`.
- **`/healthypress:setup` always creates a new site** when run without arguments. It no longer lists
  existing sites or asks the user to choose one — a dedicated site is what keeps health content out
  of places it shouldn't be, and the site picker was both friction and a footgun. Passing a `site`
  argument now means **audit mode**: re-harden and re-verify a site HealthyPress already set up,
  refusing to touch a site that has no HealthyPress structure on it. The site title is set to a
  neutral default rather than asked about.

### Added

- Disclosure, in the README and before the authorization prompt, that **the OAuth grant is
  account-wide** — sites, posts, media, taxonomy, users, stats, and notifications across the whole
  WordPress.com account — and is issued to a shared Claude Code OAuth client rather than to
  HealthyPress by name.

### Fixed

- `docs/wpcom-mcp-notes.md` listed native OAuth handling as a settled fact. It was wrong; both auth
  findings are now recorded as verified with the date and method.

## 0.1.0 — 2026-09-17

Initial release.

- `/healthypress:setup` — privacy-gated site setup and idempotent privacy audit: launch, set
  Private, verify, discourage search engines, disable comments and pingbacks, disable newsletter
  email and verify zero subscribers, neutral site title, timezone, taxonomy, pages, privacy report.
  Stops hard if the site can't be made Private.
- `/healthypress:log` — record one health event as a private, correctly dated, categorized, and
  tagged post, with a full read-back before saving and a refresh of only the affected pages.
- `/healthypress:backfill` — guided history intake, era by era and system by system, with
  checkpoints, a resumable captured list, a declined-topics list, and the free-plan 30-day MCP
  warning before any work.
- `/healthypress:share` — care team access on private sites only: Editor (full read, also edit) or
  Viewer (read-only, published pages only), with the tradeoff stated in plain language.
- `/healthypress:review` — read-only factual report plus hygiene audit and privacy check.
- Skills: `health-content-model`, `wpcom-mcp-operations`, `health-intake-interview`,
  `health-summary-pages`.
- `.mcp.json` declaring the WordPress.com MCP server (`wpcom`, HTTP transport, native OAuth).

### Known unverified behavior

Seven MCP behaviors this design depends on are documented but not yet verified against a live site;
three of them are load-bearing. See `docs/wpcom-mcp-notes.md` in the repository root. The commands
handle each by discovering schemas at runtime and reading writes back rather than trusting them.
