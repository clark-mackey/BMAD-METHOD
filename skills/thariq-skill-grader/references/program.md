# Grader Policy (program.md)

Human-authored policy. The grader reads this at the start of every grading session and every Review Mode session. Edit deliberately; do not modify as a side effect of a single grading session.

## Novelty Criteria (for the capture step)

An observation is capture-worthy only if it is ALL of the following:

1. **Specific.** It names a concrete pattern, not a vague feeling.
2. **Not already in a loaded reference.** Re-check `anti-patterns.md`, `description-patterns.md`, `exemplars.md`, and `categories.md` before appending. If it matches an existing entry, do not capture.
3. **Not already in `learnings.md`.** Check the top 20 entries first. Then grep the full log for a distinctive phrase from the observation. Duplicates past the top-20 window are still duplicates. If the prior entry is `status: archived`, treat the archival as authoritative and do not re-capture.
4. **Relevant to skill authoring in general.** A one-off typo in the target skill is not a learning.

If any condition fails, skip capture.

## Capture Discipline

- Minor stylistic complaints are not novel.
- Variations of existing anti-patterns are not novel — they reinforce the existing entry, not a new one.
- If uncertain, set `confidence: low` and let the reviewer decide during Review Mode.

## Target-Reference Map

Route each learning by type:

| Observation type | Candidate reference |
|------------------|---------------------|
| Failure mode / anti-pattern | `references/anti-patterns.md` |
| Description / trigger issue | `references/description-patterns.md` |
| Classification gap | `references/categories.md` |
| New exemplary skill | `references/exemplars.md` |
| Rubric ambiguity | `program` (flagged for rubric revision, not auto-promoted) |
| Regression-worthy failure | `eval-case` (new JSON in `evals/`) |

A single learning may legitimately promote to both a reference and an eval case. Record both in `promoted_to` when that happens.

## Promotion Thresholds (for Review Mode)

These are guidance, not enforcement. Promote an entry if **any** of the following hold:

- The same observation has appeared in ≥2 grades.
- Confidence is high AND the candidate reference is clearly canonical.
- A completed Review Mode pass has exercised the loop and this entry is the reviewer's chosen seed promotion.

Additionally:

- Archive entries with `confidence: low` that have not recurred after a reasonable window.
- Never promote an observation the reviewer cannot articulate in one sentence.

## Immutability Directives

- **The rubric in `SKILL.md` is the yardstick.** Do not modify it based on a single grading session. Rubric changes require a stand-alone revision.
- **The eval cases in `evals/` are the regression yardstick.** Do not modify existing evals to make a new grader behavior pass. Add new evals; do not edit old ones.
- **Test fixtures under `evals/test-fixtures/` are intentionally flawed.** Do not grade them as real skills or capture their defects as learnings.

## Eval Strategy by Skill Type

Before applying Dimension 11, determine whether the target skill is a capability skill or a preference skill (see `references/evals-and-comparators.md`). The distinction changes what "good evals" look like:

- **Capability skill:** test the capability directly. Re-baseline periodically — base-model improvements may obsolete the skill.
- **Preference skill:** test for style conformance. Benchmarks catch drift when the base model changes defaults.

A skill that ships capability-style evals for preference content (or vice versa) scores no higher than 2 on Dim 11, even if the eval count and infrastructure look correct.
