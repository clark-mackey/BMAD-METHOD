# Evals & Comparators

Load when scoring **Dimension 11 — Evaluation & Testing**, or when a user asks how to set up eval infrastructure for a skill.

## The Evaluation-Driven Loop

Skill Creator 2.0 (March 2026) formalizes a four-step loop:

1. **Create** — scaffold SKILL.md, scripts, references, templates.
2. **Eval** — define structured test cases (input + expected behavior) and run them.
3. **Improve** — analyze failure logs; patch SKILL.md instructions.
4. **Benchmark** — run the full eval suite after every change; track pass rate, duration, token cost.

Evals come **before** extensive instructions. Establish a no-skill baseline first, then write the minimal SKILL.md needed to pass.

## Structured Eval Scenarios

```json
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF and save to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Reads the PDF using an appropriate library",
    "Extracts text from all pages without skipping any",
    "Saves to output.txt in a readable format"
  ]
}
```

Each scenario is reproducible: same inputs, same fixtures, same assertions. Ship at least 3 per skill.

## Reproducibility & State Isolation

The most common failure mode is flaky evals caused by ambient state — live repos, real API responses, local filesystem drift. Use fixtures and mocked data. Flag any eval with >20% variance across runs as a systemic problem, not a fluke.

## A/B Split Testing (Comparator Agents)

For each test case, spawn two independent agents: one with Version A loaded, one with Version B. An LLM judge evaluates both outputs **without knowing which is which**. Ship Version B only if it improves aggregate pass rate without regressing other scenarios.

Never overwrite a skill's instructions without A/B evidence.

## Multi-Agent Benchmarking

Benchmark mode runs many cases in parallel. Track three metrics per run:
- Pass rate (tests passed / total)
- Wall-clock duration per case
- Token cost per case

## Capability vs. Preference Skills

The distinction changes eval strategy:

| Type | Definition | Eval strategy |
|------|------------|---------------|
| **Capability** | Extends what the base model can do (parse complex PDFs, call a niche API) | Test the capability directly. Watch for model improvements making the skill redundant — re-baseline periodically. |
| **Preference** | Encodes how *you* want work done (company style, naming conventions) | Test for style conformance. Benchmarks catch drift when the base model changes defaults. |

Ask which type the skill is before designing its evals.

## Claude A / Claude B (Lightweight Baseline)

If formal evals are out of reach, the two-instance pattern is an acceptable step down:
- **Claude A** (expert) refines SKILL.md.
- **Claude B** (worker) runs the skill on real tasks in a fresh session.
- Observe Claude B's behavior, bring specific failures back to Claude A.

Documented Claude A/B practice supports a Dim 11 score of 2, not 3.

## Grading Heuristic

**No `evals/` directory → Dim 11 score capped at 1**, unless Dim 11 is explicitly marked N/A for this skill (trivial one-shot utilities where eval infrastructure is disproportionate to value).

Stale or broken scenarios (missing fixtures, assertions that no longer parse) also cap at 1.
