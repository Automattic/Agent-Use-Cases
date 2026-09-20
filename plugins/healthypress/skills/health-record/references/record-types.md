# Record types reference

Required `## Details` keys per leaf category, plus title patterns and worked examples.

Keys are written as `Key: value` lines in a `## Details` section, in the order listed. If a required
key is genuinely unknown, write `unknown` rather than omitting the line — an omitted key is
indistinguishable from an unasked question.

Every record also carries, in `## Provenance`: `Recorded: <YYYY-MM-DD>` (today, as distinct from the
event date), `Source: <how the user knows this>`, and — when the date is fuzzy — the
`Date reported as:` line goes in `## Details`.

## Body section order

`## Summary` → `## Details` → `## Attachments` → `## Follow-up` → `## Provenance`. Omit empty
sections. Never rename or reorder them; section-addressable edits key off the heading text.

---

## encounters

### `visit` / `urgent-care` / `emergency`
Title: `Visit: <Clinician> (<Specialty>) — <reason>` · `Urgent care: <Facility> — <reason>` ·
`ER: <Facility> — <reason>`
Keys: Clinician · Specialty · Facility · Reason for visit · Findings discussed · Plan (per
clinician) · Orders placed · Next appointment

### `hospitalization`
Title: `Hospitalization: <Facility> — <reason>`
Keys: Facility · Admitted · Discharged · Reason for admission · Discharge diagnosis (per discharge
summary) · Procedures during stay · Discharge medications · Discharge instructions (per summary)

### `procedure`
Title: `Procedure: <Name> — <site or detail>`
Keys: Procedure · Performed by · Facility · Indication · Anesthesia · Findings (per report) ·
Complications · Aftercare instructions

### `therapy`
Title: `Therapy: <Type> — session <n>`
Keys: Type · Provider · Facility · Focus of session · Exercises or homework assigned · Sessions
remaining

## diagnostics

### `lab`
Title: `Lab: <Panel or test> — <headline result>`
Keys: Test · Ordered by · Performed at · Collected · Result (per analyte) · Reference range (per
report) · Flag (per report) · Notes (per report)

`Result`, `Reference range`, and `Flag` repeat once per analyte, aligned:

```
Result: LDL 142 mg/dL; HDL 51 mg/dL; Total cholesterol 210 mg/dL; Triglycerides 85 mg/dL
Reference range (per report): LDL <100 mg/dL; HDL >40 mg/dL; Total <200 mg/dL; Trig <150 mg/dL
Flag (per report): LDL — High; Total cholesterol — High; HDL — (none); Triglycerides — (none)
```

**`Flag (per report)` is transcription.** Copy only what the report printed. If the report printed
no flag, write `(none)`. Never derive a flag by comparing the result to the range yourself.

### `imaging`
Title: `Imaging: <Modality> <body part> — <headline finding>`
Keys: Study · Body part · Modality · Ordered by · Performed at · Contrast · Impression (verbatim
from report) · Findings (verbatim excerpt or "full report attached") · Comparison (per report)

Quote the Impression verbatim in quotation marks. Do not summarize a radiologist.

### `screening`
Title: `Screening: <Name> — <result>`
Keys: Screening · Ordered by · Performed at · Result (per report) · Next due (per report)

### `vitals`
Title: `Vitals: <context> — <headline value>`
Keys: Measured by · Context (home / clinic / device) · Blood pressure · Heart rate · Weight ·
Height · Temperature · Oxygen saturation · Device

Omit vital keys that weren't measured. One post per measurement session, not per value.

## medications

### `med-start`
Title: `Med start: <Generic> <dose> <frequency>`
Keys: Drug (generic) · Brand · Dose · Route · Frequency · Prescriber · Indication · Pharmacy ·
Duration or course length · Instructions (per prescriber)

### `med-change`
Title: `Med change: <Generic> — <old> → <new>`
Keys: Drug (generic) · Brand · Previous dose · New dose · Previous frequency · New frequency ·
Changed by · Reason (per prescriber) · Effective date

### `med-stop`
Title: `Med stop: <Generic> — <reason>`
Keys: Drug (generic) · Brand · Last dose taken · Stopped by (prescriber / self) · Reason (per
prescriber, or the user's own words quoted) · Taper instructions

All three carry the same `rx-` tag. Working out what someone is currently taking depends on it.

## conditions

### `diagnosis`
Title: `Diagnosis: <Condition> — <by whom or how>`
Keys: Condition (as named) · Diagnosed by · Facility · Basis (per clinician) · Status at diagnosis ·
Related tests

### `resolution`
Title: `Resolution: <Condition> — <resolved / ruled out>`
Keys: Condition (as named) · Outcome (resolved / ruled out / in remission, per clinician) ·
Determined by · Basis (per clinician)

## symptoms

Title: `Symptom: <Name> — <duration, location, quality>`
Keys: Symptom · Onset · Duration · Location · Quality (user's words, quoted) · Severity (user's own
0–10 if they gave one) · Triggers noted by user · What helped (user's words) · Related to

Severity is only ever the user's own number, quoted as theirs. Never assign one.

## allergies

Title: `Reaction: <Substance> — <reaction>`
Keys: Substance · Reaction · Onset after exposure · Severity (user's or clinician's words, quoted) ·
Treatment given · Confirmed by · Avoid since

One post per reaction event. The allergy list is read back out of these records — the latest
reaction per `alg-` tag.

## immunizations

Title: `Immunization: <Vaccine> — <dose n or season>`
Keys: Vaccine · Dose in series · Administered by · Facility · Lot number (if on the record) · Site ·
Next dose due (per record)

## care-admin

### `insurance`
Title: `Insurance: <Carrier> — <what changed>`
Keys: Carrier · Plan · Member ID (last 4 only) · Effective · What changed · Claim or EOB reference ·
Amount

**Never write a full insurance member ID, MRN, or account number.** Last four digits only.

### `referral`
Title: `Referral: <From> → <To specialty>`
Keys: Referred by · Referred to · Specialty · Reason (per referring clinician) · Authorization
number (last 4 only) · Expires · Status

### `records`
Title: `Records: <action> — <facility>`
Keys: Action (requested / received / released) · Facility · Records covering · Requested · Received ·
Format · Where stored

## journal

Title: `Journal: <YYYY-MM-DD> — <short phrase in the user's words>`
Keys: (none required)
`## Summary` carries the entry, quoting the user verbatim. Journal entries are the one record type
that keeps the date in the title, because there is no other subject to name them by.

---

## Worked example — a lab record

```
Title:    Lab: Lipid panel — LDL 142
Excerpt:  Fasting lipid panel at Mercy General; LDL 142 mg/dL, flagged high on the report.
Date:     2026-03-04 08:15:00
Status:   private
Category: lab
Tags:     test-lipid-panel, dr-amara-okafor, sys-cardiovascular, src-portal

## Summary

Fasting lipid panel drawn at Mercy General, ordered by Dr. Okafor at the February visit. Results
came back through the portal the same afternoon.

## Details

Test: Lipid panel (fasting)
Ordered by: Dr. Amara Okafor
Performed at: Mercy General Lab
Collected: 2026-03-04 08:15
Result (per analyte): LDL 142 mg/dL; HDL 51 mg/dL; Total cholesterol 210 mg/dL; Triglycerides 85 mg/dL
Reference range (per report): LDL <100 mg/dL; HDL >40 mg/dL; Total <200 mg/dL; Triglycerides <150 mg/dL
Flag (per report): LDL — High; Total cholesterol — High; HDL — (none); Triglycerides — (none)
Notes (per report): "Fasting status confirmed by patient."

## Attachments

- 2026-03-04-lab-1.pdf — full lipid panel report (2 pages)

## Follow-up

Recheck in 3 months per the report's standing order note.

## Provenance

Recorded: 2026-03-05
Source: WordPress.com patient portal export, transcribed
```

## Worked example — a fuzzy-dated diagnosis

```
Title:    Diagnosis: Plantar fasciitis (c. 2015)
Excerpt:  Diagnosed with plantar fasciitis in the right heel, recalled as sometime in 2015.
Date:     2015-07-01 12:00:00
Status:   private
Category: diagnosis
Tags:     dx-plantar-fasciitis, sys-musculoskeletal, src-memory, precision-year

## Summary

User recalls being diagnosed with plantar fasciitis in the right heel: "it hurt every morning for
the first ten steps." No records on hand.

## Details

Condition (as named): Plantar fasciitis
Diagnosed by: unknown
Facility: unknown
Basis (per clinician): unknown
Status at diagnosis: active
Date reported as: "sometime in 2015"

## Provenance

Recorded: 2026-09-17
Source: user recollection, no document
```
