# Smoke Test — Manual Verification

Run these in a **fresh Claude Code session** (no prior context). The point is to confirm the grader, loaded cold, produces the expected-shape output on intentionally-flawed fixtures and on itself. Do not grade the fixtures as real skills — they exist to exercise the grader.

---

## Setup

From a fresh session with `thariq-skill-grader` installed and triggerable:

> Grade the skill at `thariq-skill-grader/evals/test-fixtures/vague-description/`.

Repeat for each fixture below.

---

## Case 1 — `vague-description`

**Fixture defects:**
- Description: "A helpful skill for working with files." — no trigger conditions, no specificity (AP-D1 "marketing summary").
- `when_to_use`: missing.
- Workflow steps are placeholders ("Do something with it").

**Expected grader behavior:**
- **Dim 1 (Description/Trigger):** 0 or 1. Must cite AP-D1.
- **Dim 2 (Context Efficiency):** likely 2 (skill is short, just empty).
- **Dim 3 (Gotchas):** 0 — none present.
- **Overall:** F or D.
- **N/A Dimensions:** Dim 10 (no data), Dim 11 (out of scope), Dim 12 (out of scope) — denominator should drop to /27.
- **Capture step:** every defect is already covered by `anti-patterns.md` / `description-patterns.md`. Nothing novel → `learnings.md` should NOT gain an entry.

**Pass criteria:**
- [ ] Grade card uses /27 (or explicit N/A note) not /36.
- [ ] AP-D1 cited by code.
- [ ] `learnings.md` unchanged after grading.

---

## Case 2 — `bloated-context`

**Fixture defects:**
- Description has trigger keywords — Dim 1 is actually OK.
- Body contains encyclopedia-style "What is JSON?" / "What is Git?" sections teaching the base model things it already knows.
- No progressive disclosure — everything loaded up front.

**Expected grader behavior:**
- **Dim 1:** 2 or 3 — triggers are concrete. Don't punish the trigger just because the body is bad.
- **Dim 2 (Context Efficiency):** 0 or 1. Flag the redundant-knowledge anti-pattern.
- **Dim 5 (File Structure):** 1 — no references/, everything inline.
- **Overall:** D likely.
- **Capture step:** "teaches base-model-known facts" may or may not be in `anti-patterns.md`. If not, ONE capture is appropriate with `candidate_reference: anti-patterns`, `confidence: medium`.

**Pass criteria:**
- [ ] Dim 1 not penalized for body problems.
- [ ] Dim 2 flagged, not Dim 3.
- [ ] At most one new `learnings.md` entry (or zero if already catalogued).

---

## Case 3 — `anti-pattern-ap-d1`

**Fixture defects:**
- Description is the canonical AP-D1 specimen: "helps with various tasks."
- Body is trivial.

**Expected grader behavior:**
- **Dim 1:** 0. AP-D1 cited explicitly.
- **Most other dimensions:** N/A or 0.
- **Overall:** F.
- **Capture step:** pure AP-D1 — already the top entry in `anti-patterns.md`. NO new learning.

**Pass criteria:**
- [ ] AP-D1 cited by code.
- [ ] Grade is F.
- [ ] `learnings.md` unchanged.

---

## Case 4 — Self-grade

> Grade `thariq-skill-grader` itself.

**Expected behavior:**
- All 12 dimensions scored (no N/A — this skill has data/state via learnings.md, has evals/, has a self-improvement loop).
- **Dim 11 (Evaluation & Testing):** 2 or 3 — 3 eval JSONs exist, fixtures exist, `evals-and-comparators.md` cited. Baseline evidence may be missing → 2 is defensible.
- **Dim 12 (Self-Improvement Loop):** 2 — `program.md`, `learnings.md`, and Review Mode all exist, but no `status: promoted` entries yet.
- **Overall:** should land B or higher. If it scores C or worse, something in the rubric or the skill is miscalibrated — investigate before shipping.

**Pass criteria:**
- [ ] Uses /36 denominator.
- [ ] Dim 11 and Dim 12 both scored (not N/A).
- [ ] Grade card shows all 12 rows.

---

## Review Mode check

> Review learnings.

**Expected behavior:**
- Loads `program.md` and `learnings.md`.
- Finds only the archived seed entry.
- Reports: 0 captured, 0 promoted, 0 newly archived, 1 already archived.
- Makes no file edits.

**Pass criteria:**
- [ ] Grader recognizes trigger phrase and enters Review Mode (does not grade anything).
- [ ] No file modifications to `learnings.md`.

---

## If any case fails

Record the failure as a new eval case under `evals/` — do NOT edit existing evals or the rubric to make it pass (Immutability Directive, `program.md`).
