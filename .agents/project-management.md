# Project Management

This file is the tracked project-management contract for this repository. It tells humans and agents where work is tracked, which systems are authoritative, and how issue work should be handled.

## Repository visibility

This is a public/open-source repository. Every tracked artifact must be useful to an outside contributor with no private access. Do not include private company links, private tracker IDs, private Slack channels, customer data, or internal-only process details in tracked files, commits, public issues, or public PRs.

Maintainers may coordinate in private systems outside this repository. Those systems are not the public source of truth. If private context informs public work, summarize the relevant technical facts publicly without linking or exposing the private source.

Private local overlays, if needed, belong in `.agents/project-management.local.md` or `.agents/private/`; both are ignored by the scaffold gitignore.

## System of record

Primary tracker: `ROADMAP.md`

There is no external issue tracker configured for this project. `ROADMAP.md` is the source of truth for shaped work and autonomous "what next" routing.

## Source-of-truth precedence

1. Active assigned issue or linked project item, when one exists.
2. `ROADMAP.md` for shaped project direction and autonomous next work.
3. `DESIGN.md` for current behavior, architecture, and invariants.
4. `.agents/decisions/` for durable rationale.
5. `FOLLOW_UPS.md` and `IDEAS.md` for deferred or unshaped work, when present.

## Starting issue work

- Read the issue, linked project item, parent epic, and related PRs when available.
- Confirm acceptance criteria and current status.
- Search `DESIGN.md`, `.agents/decisions/`, and `.agents/reference/` for relevant constraints.
- Create or reuse a scratchpad session when the issue requires investigation, planning, or review.
- If scope is unclear, ask before implementing.

## During work

- Keep tracker status in sync when meaningful: started, blocked, ready for review, done.
- Add discoveries to scratchpad first; promote durable findings to `DESIGN.md`, `.agents/decisions/`, `.agents/reference/`, `ROADMAP.md`, or `CHANGELOG.md`.
- Link decision records and PRs back to the issue when useful and allowed by repository visibility.

## Closing issue work

- Verify acceptance criteria.
- Update docs and durable records in the same commit or PR as the code change.
- Move roadmap state if this shipped or changed project direction.
- Leave a concise issue or PR summary with what changed, verification, and follow-ups.

## Naming and linking

- Branch:
- Commit reference:
- PR title:
- Issue closure keyword:
