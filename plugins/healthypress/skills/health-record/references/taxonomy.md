# Taxonomy reference

The complete closed vocabularies. `SKILL.md` carries the rules; this file carries the lists.

## Categories

Hierarchical. A post gets **exactly one leaf**. Parents are containers — never assign a parent
directly, except for the childless categories, which are themselves leaves.

| Slug | Parent | Use it for |
|---|---|---|
| `encounters` | — | container |
| `visit` | `encounters` | routine or specialist office visit |
| `urgent-care` | `encounters` | walk-in / urgent care |
| `emergency` | `encounters` | ER visit |
| `hospitalization` | `encounters` | admitted, one post per admission |
| `procedure` | `encounters` | surgery, endoscopy, biopsy, injection |
| `therapy` | `encounters` | PT, OT, counseling, rehab session |
| `diagnostics` | — | container |
| `lab` | `diagnostics` | blood, urine, culture, pathology |
| `imaging` | `diagnostics` | X-ray, ultrasound, CT, MRI, echo, DEXA |
| `screening` | `diagnostics` | colonoscopy screening, mammogram, vision, hearing |
| `vitals` | `diagnostics` | BP, weight, temp, pulse ox readings |
| `medications` | — | container |
| `med-start` | `medications` | began a drug or supplement |
| `med-change` | `medications` | dose, frequency, route, or formulation changed |
| `med-stop` | `medications` | discontinued, completed a course, or was told to stop |
| `conditions` | — | container |
| `diagnosis` | `conditions` | a condition was named |
| `resolution` | `conditions` | a condition ended, resolved, or was ruled out |
| `care-admin` | — | container |
| `insurance` | `care-admin` | coverage changes, claims, denials, EOBs |
| `referral` | `care-admin` | referral issued or completed |
| `records` | `care-admin` | records requested, received, released |
| `symptoms` | — | something the user felt, one episode per post |
| `allergies` | — | a reaction event (the allergy list is read back out of these) |
| `immunizations` | — | a vaccine administered |
| `journal` | — | a dated subjective entry with no clinical event |
| `needs-triage` | — | **site default category.** Unclassifiable records land here on purpose. |

Do not create categories beyond this list. If something genuinely doesn't fit, file it in the
nearest leaf and explain in `## Details` under a `Note:` key — then mention it to the user so the
taxonomy gap is a known thing rather than a silent one.

## Tag prefixes

| Prefix | Means | Example | Notes |
|---|---|---|---|
| `dx-` | condition | `dx-hypertension`, `dx-plantar-fasciitis` | the condition itself, not the encounter |
| `rx-` | medication or supplement | `rx-lisinopril`, `rx-vitamin-d` | **generic name only** |
| `sx-` | symptom | `sx-migraine`, `sx-fatigue` | singular |
| `dr-` | clinician | `dr-amara-okafor` | last name plus first if needed to disambiguate |
| `fac-` | facility | `fac-mercy-general` | clinic, hospital, lab, pharmacy |
| `test-` | a named test or study | `test-lipid-panel`, `test-a1c`, `test-mri-lumbar` | the test, not the result |
| `alg-` | allergen | `alg-amoxicillin`, `alg-latex` | the substance |
| `sys-` | body system | `sys-cardiovascular` | closed list below, exactly one per post |
| `src-` | provenance | `src-portal` | closed list below, exactly one per post |
| `precision-` | date precision | `precision-month` | only when the date is fuzzy |

### `sys-` — body system (closed, exactly 10)

`sys-cardiovascular` · `sys-respiratory` · `sys-gastrointestinal` · `sys-musculoskeletal` ·
`sys-neurological` · `sys-endocrine` · `sys-dermatologic` · `sys-genitourinary` ·
`sys-mental-health` · `sys-general`

`sys-general` is the honest answer for whole-body records: annual physicals, immunizations, weight,
insurance, records requests. Use it rather than guessing a system.

### `src-` — provenance (closed, exactly 4)

| Tag | Means |
|---|---|
| `src-memory` | the user recalled it, no document |
| `src-portal` | copied from a patient portal or app |
| `src-document` | transcribed from a paper or PDF document (usually also attached) |
| `src-told-by-clinician` | reported verbally by a clinician, no document |

### `precision-` — date precision (closed, 2)

`precision-month` (day unknown) · `precision-year` (month unknown). Absent means the date is exact.

## Search-before-create, concretely

1. List/search tags for the bare term (`lisinopril`, not `rx-lisinopril`) — near-duplicates often
   differ in the prefix or a typo, so search the distinctive part.
2. Exactly one plausible match → reuse it.
3. Two or more plausible matches → **ask the user which one**, and note the duplicate for the user
   so it can be cleaned up later.
4. No match → create it, spelled per the rules above.
5. Never create two tags in one post that mean the same entity (`rx-vitamin-d` and `rx-cholecalciferol`).

## Worked tagging examples

**"I saw Dr. Okafor in cardiology on Tuesday, she looked at my echo and kept me on lisinopril."**
Category `visit`. Tags: `dr-amara-okafor`, `fac-…` if known, `test-echocardiogram`,
`dx-hypertension`, `rx-lisinopril`, `sys-cardiovascular`, `src-memory`. That's 7 — at the limit, so
drop the least useful key (the facility, if the clinician is already tagged) before adding more.

**"Lipid panel came back, LDL 142, they flagged it high."**
Category `lab`. Tags: `test-lipid-panel`, `sys-cardiovascular`, `src-portal`. The "flagged high" is
transcribed into `## Details` as `Flag (per report): LDL — High`, attributed to the report. No
`dx-` tag: a flagged value is not a diagnosis.

**"I had plantar fasciitis from 2015 until about 2018."**
Two posts. `diagnosis` at `2015-07-01 12:00` with `precision-year`, title `Diagnosis: Plantar
fasciitis (c. 2015)`; `resolution` at `2018-07-01 12:00` with `precision-year`, title `Resolution:
Plantar fasciitis (c. 2018)`. Both tagged `dx-plantar-fasciitis`, `sys-musculoskeletal`,
`src-memory`.

**"Amoxicillin gave me hives as a kid, maybe 2011."**
Category `allergies`, date `2011-07-01 12:00`, `precision-year`, title `Reaction: Amoxicillin —
hives (c. 2011)`. Tags: `alg-amoxicillin`, `rx-amoxicillin`, `sys-dermatologic`, `src-memory`.
