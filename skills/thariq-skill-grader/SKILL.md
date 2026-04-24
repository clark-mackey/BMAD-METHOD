---
name: thariq-skill-grader-dev
description: LOCAL DEV FORK of the team's thariq-skill-grader. Use ONLY when user explicitly says "thariq dev", "grader dev", "use the dev grader", or "test the rubric changes". For normal grading requests ("grade this skill", "use Thariq", "audit my skill"), use anthropic-skills:thariq-skill-grader instead — this fork exists for testing rubric changes before pushing upstream to the team.
---

# Skill Grader

Grades and improves Claude Code skills against Anthropic's official standards and internal best practices.

**Standards sources** (last reviewed 2026-04-19): Official Claude Code docs (code.claude.com/docs/en/skills), Agent Skills open standard (agentskills.io), [anthropics/skills](https://github.com/anthropics/skills) repo, and Thariq Shihipar's "Writing Effective Agent Skills" (thariq.io). Canonical excerpts are mirrored in `references/official-standards.md` so grading doesn't depend on external links staying live.

## How to Use

**Single skill:** Read the target SKILL.md and its directory, then run the rubric.
**Bulk audit:** Glob for `**/SKILL.md` in the target directory, grade each, produce a summary report.

## Reference Materials

Load these on demand based on what's being graded:

- `references/official-standards.md` — frontmatter spec, character caps, lifecycle rules. Load when grading any skill.
- `references/categories.md` — the 9 skill categories. Load during classification (step 2).
- `references/exemplars.md` — gold-standard skills annotated by category. Load when scoring against "exemplary" criteria — pick the closest exemplar to compare against.
- `references/anti-patterns.md` — concrete failure modes with bad/why/fixed examples (AP-D1 through AP-DM2). Load when a dimension scores ≤2 to cite specific patterns and their fixes.
- `references/description-patterns.md` — templates for writing trigger specifications by skill type. Load when grading Dimension 1 or proposing rewrites.
- `references/thariq-article.md` — Thariq Shihipar's principles for skill design. Load when citing authority for design choices.
- `references/plugin-skill-conventions.md` — different rules for plugin-bundled skills (skills with `:` in their identifier). **Always load when grading a plugin skill** — many standalone-skill anti-patterns are correct in plugin context.
- `references/lifecycle-and-hooks.md` — how skills behave after invocation (auto-compaction, content lifecycle) and when to use hooks vs. instructions. Load when grading Dimensions 2, 4, or 7.
- `references/evals-and-comparators.md` — the Create→Eval→Improve→Benchmark loop, the Skill Creator 2.0 eval JSON schema, blind A/B comparator mechanics, reproducibility rules, capability-vs-preference eval strategy, and the Dim 11 grading heuristic. Load when scoring Dimension 11 or when a user asks how to set up eval infrastructure.
- `references/program.md` — human-authored policy: novelty criteria, promotion thresholds, target-reference map, and immutability directives. Load at the start of every grading session and every Review Mode session.
- `references/learnings.md` — append-to-top dated log of captured observations. Read the top 20 entries at the start of every grading session (for novelty filtering). Read all `status: captured` entries at the start of Review Mode.

**`evals/` directory** — regression suite aligned to Skill Creator 2.0's JSON format. Treated as immutable during a grading session. Test fixtures under `evals/test-fixtures/` are intentionally flawed and must not be graded as real skills or captured as learnings.

## Workflow

1. **Read** the skill directory — SKILL.md, references/, scripts/, templates/
2. **Classify** into one of the 9 skill categories (see `references/categories.md`). If the skill identifier contains `:`, also load `references/plugin-skill-conventions.md`.
3. **Score** against each rubric dimension (see Rubric below). Pull from `references/anti-patterns.md` and `references/exemplars.md` as needed for citations. For Dimension 11, also load `references/evals-and-comparators.md`. Load `references/program.md` (policy) and `references/learnings.md` (top 20 entries) once here — step 5 references them without re-reading.
4. **Report** the grade card with specific findings and fix suggestions. Cite anti-pattern codes (e.g., AP-D1) where applicable.
5. **Capture novel learnings.** Using the `program.md` policy and the `learnings.md` context already loaded in step 3, compare any observation that drove a score deduction against the references and against the full novelty check (top-20 scan plus a grep for distinctive phrasing — see `program.md` §Novelty Criteria). If genuinely novel, append a structured entry to the top of `learnings.md`. If every observation matches an existing reference or prior log entry, skip capture. Do not capture minor stylistic variants. Use `candidate_reference: eval-case` when the observation represents a failure mode the grader itself might regress on.
6. **Optionally rewrite** — if the user asks, produce a corrected version using `references/description-patterns.md` for any description rework.

## Review Mode

Triggered when the user says "review learnings", "promote learnings", "triage learnings", or equivalent. In Review Mode the skill is NOT grading a new skill — it is auditing `references/learnings.md` against `references/program.md` and promoting or archiving entries.

1. **Load** `references/program.md` and `references/learnings.md` in full. Also load any reference file named as a promotion target by current entries (e.g., `anti-patterns.md` if entries point there).
2. **Walk** entries top-to-bottom. For each entry with `status: captured`:
   - If it meets the promotion thresholds in `program.md` and the reviewer can articulate it in one sentence, add the content to the named candidate reference (or draft a new eval JSON under `evals/`). Edit the entry's `status:` line to `status: promoted`, and append `promoted_to:` and `promoted_at:` fields.
   - If confidence is low, the observation has not recurred, and it is no longer useful, edit `status:` to `status: archived` and append `archived_reason:` and `archived_at:` fields.
   - Otherwise leave the entry untouched.
3. **Never** modify the rubric in this file based on a single entry. Rubric changes are a separate, human-driven revision (see Immutability Directives in `program.md`).
4. **Never** delete entries. Status transitions are edits in place, so the log remains an auditable history.
5. **Report** a short summary to the user: entries reviewed, entries promoted (with targets), entries archived, entries left captured.

## The Rubric

Score each dimension 0-3. Total possible: 36 points across 12 dimensions.

| Score | Meaning |
|-------|---------|
| 0 | Absent or harmful |
| 1 | Present but weak |
| 2 | Solid |
| 3 | Exemplary |

### 1. Description / Trigger Quality (0-3)

The `description` field in frontmatter is **not a summary — it's a trigger specification.** It determines when Claude activates the skill. Per current Claude Code docs: technically optional (defaults to first paragraph) but strongly recommended. Combined `description` + `when_to_use` is truncated at **1,536 characters** in the skill listing.

| Score | Criteria |
|-------|----------|
| 3 | States what the skill does AND when to use it. Includes positive triggers (phrases/situations that should activate) and negative triggers (when NOT to activate, e.g., "For X, use skill-Y instead"). No ambiguity with other installed skills. Combined with `when_to_use` stays under 1,536 chars. |
| 2 | Good positive triggers but missing negative triggers or some overlap risk. |
| 1 | Vague or summary-style description. Would trigger too broadly or too narrowly. |
| 0 | Missing, generic ("A helpful skill"), or misleading. |

**Common failures:**
- Writing a marketing summary instead of trigger conditions
- Missing "also use when..." variants (users phrase things many ways)
- No disambiguation from similar skills
- Exceeding the 1,536-char combined cap (gets truncated in the skill listing — Claude won't see the tail)
- Not using `when_to_use` to add trigger phrases when description is at capacity

### 2. Context Efficiency (0-3)

Every token in skill instructions is a token that can't be used for the actual task.

| Score | Criteria |
|-------|----------|
| 3 | SKILL.md under 500 lines. Only contains what Claude doesn't already know. Progressive disclosure via references/. No redundancy with Claude's native knowledge. |
| 2 | Reasonable length but some unnecessary explanation of things Claude knows natively. |
| 1 | Bloated — explains basic concepts, includes full API docs inline, or duplicates content across SKILL.md and references. |
| 0 | Massively over-engineered. Wall of text that crowds out task context. |

**Test:** Would removing a section noticeably degrade output quality? If no, cut it.

### 3. Gotchas & Edge Cases (0-3)

The highest-signal content in any skill. This is what pushes Claude out of its normal behavior.

| Score | Criteria |
|-------|----------|
| 3 | Documents specific failure modes, common mistakes, and non-obvious traps from real usage. Updated regularly. |
| 2 | Has some gotchas but they're generic or haven't been updated. |
| 1 | No gotchas section. Skill only has happy-path instructions. |
| 0 | Contains actively wrong information that would cause failures. |

**What makes a great gotcha:** "When doing X, Claude will default to Y, but in this codebase/context you must do Z because [reason]."

### 4. Flexibility vs. Rigidity Balance (0-3)

Skills should define what matters while letting Claude adapt its approach.

| Score | Criteria |
|-------|----------|
| 3 | Specifies constraints and goals clearly. Leaves implementation approach flexible. Uses templates as defaults, not mandates (unless fragility requires it). |
| 2 | Mostly flexible but has some unnecessarily rigid output format requirements. |
| 1 | Over-specifies — rigid templates, exact wording requirements, step-by-step scripts where judgment would be better. |
| 0 | Either completely rigid (identical output every run) or completely vague (no actionable guidance). |

**Calibration question:** Does this encode organizational knowledge (good) or basic competence (bad)?

### 5. File Structure & Progressive Disclosure (0-3)

Skills are folders, not just files. `name` is optional (defaults to directory name) but when set should match the parent directory. SKILL.md under 500 lines. Body under ~5000 tokens recommended. File references one level deep.

| Score | Criteria |
|-------|----------|
| 3 | Clean directory structure. `name` matches directory (or omitted to use default). SKILL.md under 500 lines. References/ for detailed docs loaded on demand. Scripts/ for executable helpers. Templates/assets/ for output resources. Clear pointers from SKILL.md to when each reference should be read. |
| 2 | Reasonable structure but some content that should be in references is inline. |
| 1 | Everything crammed into SKILL.md. No use of subdirectories. |
| 0 | Disorganized, missing SKILL.md, or `name` is set to a value that doesn't match the directory (causes confusion). |

### 6. Category Fit (0-3)

Skills should map cleanly to one of the 9 categories. See `references/categories.md`.

| Score | Criteria |
|-------|----------|
| 3 | Clearly fits one category. Doesn't mix concerns (e.g., code style + testing + deployment in one skill). |
| 2 | Fits a category but has some scope creep into adjacent territory. |
| 1 | Tries to do too many things. Should be split into 2+ skills. |
| 0 | Unclear what this skill is even for. |

### 7. Verification & Safety (0-3)

Skills that produce output without verification are dangerous.

| Score | Criteria |
|-------|----------|
| 3 | Includes verification steps, safety guardrails for destructive actions, or explicit "check this before declaring done" instructions. |
| 2 | Some verification but incomplete. |
| 1 | No verification — happy path only. |
| 0 | Actively dangerous — could cause data loss, publish secrets, or break production with no safeguards. |

### 8. Redundancy Check (0-3)

Does this skill earn its keep?

| Score | Criteria |
|-------|----------|
| 3 | Encodes knowledge Claude genuinely lacks. Output with skill is measurably better than without. No overlap with other installed skills. |
| 2 | Useful but overlaps partially with Claude's native abilities or another skill. |
| 1 | Marginal value. Claude would produce 80%+ equivalent output without this skill. |
| 0 | Actively harmful — overrides Claude's better native judgment or conflicts with other skills. |

**Test:** Run the same task with and without the skill. If output is ~80% the same, the skill isn't earning its context cost.

### 9. Maintainability (0-3)

Skills rot. Models improve. What was necessary 3 months ago may be deadweight now.

| Score | Criteria |
|-------|----------|
| 3 | Clear ownership signal. Content is evergreen or has dated elements flagged for review. No stale references to deprecated tools/APIs. |
| 2 | Mostly current but some dated advice. |
| 1 | Contains references to outdated patterns or tools. |
| 0 | Clearly abandoned — references things that no longer exist. |

### 10. Data & State Management (0-3)

| Score | Criteria |
|-------|----------|
| 3 | Uses `${CLAUDE_PLUGIN_DATA}` for persistent data. Uses `${CLAUDE_SKILL_DIR}` to reference bundled scripts/assets. Never stores state in the skill folder itself. Handles missing data gracefully. |
| 2 | Reasonable data handling with minor issues. |
| 1 | Stores data in the skill folder (will be overwritten on upgrade). |
| 0 | No data management needed and correctly omitted, OR stores secrets/credentials in skill files. |

*Score N/A if the skill has no data/state requirements — don't penalize stateless skills.*

### 11. Evaluation & Testing (0-3)

Does the skill ship with the infrastructure to prove it changes outcomes and to catch regressions when instructions or the base model change?

| Score | Criteria |
|-------|----------|
| 3 | At least 3 reproducible eval scenarios exist in `evals/` using a recognized format (Skill Creator 2.0 JSON or equivalent) AND baseline established (evals confirmed to fail without the skill). A/B evidence (branch, diff, or notes from a blind comparator run) is a tiebreaker that lifts a borderline 2 to a 3; its absence alone does not demote an otherwise-3 skill to 2. |
| 2 | Eval scenarios exist but are informal, undocumented, missing baseline evidence, or fewer than 3. The Claude A/B development pattern (expert instance refines, worker instance executes, observations feed back), if clearly documented, is also an acceptable 2. |
| 1 | No evals. Skill has been tested only manually and subjectively. |
| 0 | Skill has never been tested; author is relying on intuition. |

**Load `references/evals-and-comparators.md`** when scoring this dimension. It contains the JSON schema, A/B comparator mechanics, reproducibility rules, and the capability-vs-preference strategy split.

*Score N/A if the skill's scope is genuinely too narrow for eval infrastructure to be proportionate (trivial one-shot utilities). If N/A, a missing `evals/` directory is not a defect.*

### 12. Self-Improvement Loop (0-3)

Does the skill capture novel observations from real use and feed them back into its own knowledge base and/or regression suite?

| Score | Criteria |
|-------|----------|
| 3 | Skill has a `program.md`-style policy doc, a dated `learnings.md`-style log, and a documented review/promotion protocol. Visible evidence of use: **either** ≥1 log entry marked `status: promoted` with a corresponding addition in the named reference or eval case, **or** documented evidence of a completed Review Mode pass (entries triaged, statuses updated in place) even if no promotions resulted. |
| 2 | Policy doc + log + protocol present, no promotion evidence yet (new skill or loop never exercised). |
| 1 | Learnings file exists but lacks a policy doc, promotion path, or structured entries. |
| 0 | No self-improvement mechanism. |

*Score N/A if the skill's scope is genuinely too narrow for accumulated learnings. A stale log (no entries across a long span) caps the score at 2, same logic as Maintainability.*

## Gotchas (for the grader itself)

- **N/A dimensions change the denominator.** Full scale is /36 across 12 dimensions. Each N/A dimension drops the denominator by 3 (/33, /30, /27, ...). Scale letter-grade thresholds proportionally and note every N/A dimension explicitly in the grade card.
- **Don't penalize `allowed-tools` frontmatter as scope creep.** Tool restrictions are a category-fit *strength*, not a sign the skill is doing too much.
- **Plugin-bundled skills follow different conventions.** Skills inside an installed plugin (`plugins/*/skills/`) may legitimately reference plugin paths and shared resources — don't ding them for "not user-authored" patterns.
- **Variable substitution can mislead you.** When a SKILL.md is loaded into your context via the Skill tool, `${CLAUDE_SKILL_DIR}` and similar may appear pre-substituted with absolute paths. Always Read the file directly before flagging "hardcoded path" issues.
- **Description + when_to_use is truncated at 1,536 chars in the skill listing.** Over-limit content is a 0 on Dimension 1, not a 1 — Claude literally cannot see the truncated tail, so trigger keywords past the cap don't exist for matching purposes.
- **Skill content is loaded once, not re-read.** When grading, remember that SKILL.md enters context as a single message and persists. Penalize one-time step language ("first do X, then...") in skills meant as standing instructions; reward standing-instruction phrasing.
- **A skill with zero gotchas is rarely a 3.** Real skills accumulate failure-mode notes from use. A pristine happy-path skill probably hasn't been battle-tested yet.
- **Don't capture observations already covered by loaded references.** Before appending to `learnings.md`, re-check `anti-patterns.md`, `description-patterns.md`, `exemplars.md`, and `categories.md`. Duplicates dilute the log and the `program.md` novelty criteria forbid them.
- **Eval fixtures under `evals/test-fixtures/` are intentionally flawed.** They exist to exercise the grader, not to be graded as real skills. Never score them and never capture their defects as learnings.
- **The rubric and the eval suite are both immutable during a grading session.** Do not modify `SKILL.md` rubric text or existing `evals/*.json` to make a grading behavior pass. Rubric changes require a stand-alone revision; regressions require *new* eval cases, not edits to old ones.

## Grade Card Format

```
## Skill Grade: [skill-name]

**Category:** [one of 9 categories]
**Overall Grade:** [A/B/C/D/F] ([score]/36)
**N/A Dimensions:** [list dimensions scored N/A, or "none"]

| Dimension | Score | Notes |
|-----------|-------|-------|
| Description/Trigger | X/3 | ... |
| Context Efficiency | X/3 | ... |
| Gotchas & Edge Cases | X/3 | ... |
| Flexibility Balance | X/3 | ... |
| File Structure | X/3 | ... |
| Category Fit | X/3 | ... |
| Verification & Safety | X/3 | ... |
| Redundancy Check | X/3 | ... |
| Maintainability | X/3 | ... |
| Data Management | X/3 | ... |
| Evaluation & Testing | X/3 | ... |
| Self-Improvement Loop | X/3 | ... |

**Letter Grade Scale (full /36):**
- A: 32-36 (exemplary)
- B: 26-31 (solid, minor improvements)
- C: 19-25 (functional but needs work)
- D: 12-18 (significant issues)
- F: 0-11 (rewrite needed)

*If any dimension is N/A, drop the denominator by 3 per N/A dimension and scale each band by the same ratio (e.g., /33: A 29-33, B 24-28, C 17-23, D 11-16, F 0-10).*

### Top Issues
1. [Most critical problem + specific fix]
2. [Second problem + specific fix]
3. [Third problem + specific fix]

### What's Working Well
- [Genuine strength worth preserving]
```

## Rewrite Mode

When the user asks to fix/improve a skill after grading:

1. Address the top 3 issues identified in the grade card
2. Preserve everything that scored 2+
3. Don't over-engineer — fix what's broken, leave what works
4. Show the diff, not just the new version
5. Re-grade after rewriting to confirm improvement
