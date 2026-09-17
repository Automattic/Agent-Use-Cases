---
name: health-intake-interview
description: Interview technique for eliciting health history and daily health records from a person - era-based and system-based recall scaffolding, fuzzy-date elicitation, non-leading questions, one question at a time, capturing verbatim before structuring, and always offering an exit. Use when gathering health information from the user, backfilling history, or when an answer is vague, partial, or emotionally heavy.
---

# Health Intake Interview

How to ask. The content model says what to store; this says how to get it without distorting it or
exhausting the person.

The person you are interviewing knows their own history and is probably tired of explaining it.
Your job is to make remembering easy and to write down what they actually said.

## Quick reference

| Rule | Why |
|---|---|
| One question at a time | Stacked questions get one answer and lose the rest |
| Never lead | A suggested symptom becomes a remembered symptom |
| Capture verbatim first, structure second | Paraphrase destroys the only subjective data there is |
| Offer an exit every few records | This is tiring and sometimes upsetting |
| "I don't know" is a complete answer | Record `unknown` and move on |
| Never ask twice about something declined | Re-asking reads as pressure |

## Non-leading questions

Ask open, then narrow only into what they already said.

| Don't | Do |
|---|---|
| "Did the pain radiate down your arm?" | "What happened next?" |
| "Was it a migraine?" | "What did it feel like?" |
| "Were you fasting for the lab?" | "Anything the lab told you about how to prepare?" |
| "That sounds like a bad reaction — was it anaphylaxis?" | "What did the reaction look like?" |
| "Are you still taking it?" | "Where does that medication stand now?" |

If you need a specific field the content model requires, ask for the field, not for a value:
"Who ordered it?" not "Was that Dr. Okafor?" — even when Dr. Okafor is the obvious guess. A guessed
name that gets a polite yes becomes a permanent wrong tag.

When you do need to offer options (a category, a tag disambiguation), offer them as a genuine list
including "none of these" — not as a leading pair.

## Capture verbatim, then structure

1. Let them tell it in their own words. Don't interrupt to classify.
2. Write their words down as they said them — that text goes in `## Summary` in quotes.
3. Then ask only for the missing required keys, one at a time.
4. Read the structured record back before saving, and ask "did I get that right?"

Never translate into clinical register. "My chest felt tight and weird" stays exactly that. If a
clinician used a clinical term, attribute it: `Basis (per clinician): "likely costochondritis"`.

## Era-based recall scaffolding

For history intake, walk time in chunks the person can actually picture. Anchors beat dates:

> "Let's do this in chunks. Start with right now — what's active for you currently? Then we'll work
> backwards."

Useful anchors, in rough order of reliability: where they lived · what job they had · which
insurance or clinic they used · big life events (a move, a birth, a graduation) · other people's
ages. "Was that before or after you moved to Portland?" places a memory far better than "what year
was that?"

Suggested passes: **now / active** → **last year** → **the last five years** → **each earlier
decade** → **childhood**. Then stop and offer a system pass.

## System-based recall scaffolding

Time misses whole categories, so sweep by body system once the eras are done. Ask one system at a
time, in plain language, and accept "nothing" quickly:

heart and blood pressure · lungs and breathing · stomach and digestion · bones, joints, and muscles ·
head, nerves, and migraines · hormones, thyroid, diabetes · skin · kidneys and urinary ·
mental health · everything else (vaccines, surgeries, injuries, allergies, screenings)

Also sweep these, which people reliably forget: surgeries and procedures · ER visits ·
hospitalizations · allergies and drug reactions · vaccines · medications they stopped · pregnancies ·
family history they've been told matters · anything a clinician told them to "keep an eye on".

## Fuzzy-date elicitation

Most history has no exact date, and pushing for one produces a confident fake. Ask in this order and
take the first answer that lands:

1. "Do you remember roughly when?"
2. "What season, or what part of the year?"
3. "Which year, even approximately?"
4. "Was it before or after <a known anchor>?"
5. "Closer to when you lived in <place>, or later?"

Then reflect it back with the imprecision intact: "So sometime in 2015, not sure which month — I'll
record it as approximately 2015." Never round a fuzzy date into a fake exact one, and never widen a
date the person gave precisely.

If there is no usable year at all, say so and offer the alternative: "I can't file this as a dated
record without a year, but I can put it on your Health Summary as an undated fact. Want that?"

## Pacing and exits

- Checkpoint every few records: "That's five recorded. Want to keep going, take a break, or stop
  here?" Make stopping the easy option, not the awkward one.
- Keep a visible running list of what's been captured so a break is resumable.
- Never ask about anything they declined. If they said "I don't want to get into the hospital stuff,"
  that topic is closed until they reopen it.
- If they seem distressed, stop asking questions and say what's true: this can be heavy, and we can
  stop or come back to it.
- Never ask why they stopped a medication, left a clinician, or declined a treatment more than once.
  One ask, then record whatever they gave, including nothing.

## Handling uncertainty in the answer

| They say | Record |
|---|---|
| "I think it was 10 milligrams?" | `Dose: 10 mg (user unsure)` |
| "Some kind of blood pressure pill" | `Drug (generic): unknown — "some kind of blood pressure pill"` and ask if a pharmacy could confirm |
| "The doctor said something about my thyroid" | quote it verbatim; do not name a condition they didn't name |
| "It was bad" (severity) | quote `"it was bad"`; do not assign a number |
| Two conflicting dates | record both in `## Details` as `Date reported as:` and pick the later-stated one for the post date |

Uncertainty marked as uncertain is good data. Uncertainty smoothed into false precision is a
corrupted record, and nothing downstream can tell.

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
