# Evals

Regression scenarios for the Thariq Skill Grader, aligned to Skill Creator 2.0's JSON format.

## Format

Each `*.json` file defines one scenario:
- `skills`: the skill(s) to load.
- `query`: what to ask the grader.
- `files`: fixture inputs.
- `expected_behavior`: assertions the judge checks.

## Fixtures

`test-fixtures/` contains intentionally flawed skills used as inputs. They are NOT real skills. The grader must not capture their defects as learnings (see `references/program.md`).

## Running

These evals are compatible with Skill Creator 2.0's benchmark mode. If you do not have that tooling, run each scenario manually in a fresh Claude Code session:

1. Open a fresh session with the Thariq grader available.
2. Ask the `query` verbatim.
3. Verify every `expected_behavior` assertion against the grade card.

A scenario passes only when every assertion holds.

## Adding Evals

New evals typically originate from promoted entries in `references/learnings.md` with `candidate_reference: eval-case`. See `references/program.md` for promotion guidance.
