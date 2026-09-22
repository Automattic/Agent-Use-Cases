# .agents/decisions/

Tracked human + agent decision records. Use this directory for durable rationale: why the project chose one path over another, what alternatives were considered, what consequences were accepted, and what later superseded the decision.

Decision records are part of the repository's human-and-agent operating layer. They are not scratchpad notes: they survive fresh clones and should be useful months later when a cold agent or human asks "why is this shaped this way?"

For public/open-source repositories, decision records must be public-safe: no private tracker URLs, private Slack channels, customer data, internal-only project names, or private process details. If private context informed a decision, summarize the public technical facts without exposing the private source.

## When to write a decision record

Create or update a decision record when work chooses among meaningful alternatives and the outcome affects:

- Architecture or invariants
- Roadmap direction or feature scope
- Persistence, data model, or API shape
- Security, privacy, or operational posture
- A canonical implementation pattern future agents will copy
- A notable reversal, pivot, or rejected path that future work is likely to revisit

Do not write a decision record for routine implementation details, obvious local refactors, or choices fully explained by the code diff.

## Lifecycle

- **Proposed**: drafted before the human or team has accepted it.
- **Accepted**: the project is following this decision.
- **Superseded**: replaced by a newer decision. Keep the old record; add `superseded_by` and link the replacement.

Use `TEMPLATE.md` for new records. Name records `YYYY-MM-DD-<slug>.md`.

When a decision affects current architecture or invariants, link it from `DESIGN.md` under "Decision index".
