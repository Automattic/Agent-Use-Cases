---
name: <skill-name>
description: <What knowledge this holds, then "Use when…" with the concrete phrases and situations that should trigger it. This description is the entire basis for whether the skill loads, so write the user's words, not yours.>
---

# <Skill Title>

<Two or three sentences: what this skill is authoritative about, and what it deliberately leaves to
other skills. Name the other skills.>

## Quick reference

| Situation | Do this |
|---|---|
| <the common case> | <the answer, in a few words> |
| <the case people get wrong> | <the answer> |

<A table people can act on from the first screen. If a reader needs the whole skill to use it, the
table is wrong.>

## <Topic>

<One H2 per topic. Rules as imperatives. Examples over explanation — a worked example of the right
output is worth a paragraph of description.>

## <Topic>

<Keep SKILL.md to roughly 100 lines. Bulk — full vocabularies, per-type field tables, long worked
examples — goes to `references/<topic>.md` and gets linked from here by relative path, like
`references/taxonomy.md`.>

## Common gotchas

- <The failure that's silent. These are the highest-value lines in the file.>
- <The default that's wrong for this use case.>
- <The thing that isn't recoverable.>

<If the use case has a safety or scope block, paste it here verbatim — identical text in every
SKILL.md and at the end of every command. Skill loading is probabilistic, so a guardrail that lives
in one file is a guardrail that sometimes doesn't load.>
