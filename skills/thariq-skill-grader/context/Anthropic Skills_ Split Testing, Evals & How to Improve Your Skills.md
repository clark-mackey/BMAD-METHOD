# Anthropic Skills: Split Testing, Evals & How to Improve Your Skills

> **Research date:** April 21, 2026
> **Sources:** Anthropic official docs, Thariq Shihipar (LinkedIn, Mar 18 2026), Skill Creator 2.0 release notes (Mar 3 2026), r/ClaudeCode community, The Tool Nerd

---

## 1. Background: Why This Matters

The original `skill-creator` workflow was, in the words of practitioners, "vibes-based." You wrote a `SKILL.md`, ran the skill, and if it failed you guessed how to fix it. There was no objective way to measure whether a skill actually changed outcomes or merely added tokens to the context window.

A real-world example illustrates the risk: a team built a skill that generated weekly client reports from CRM data. It worked perfectly on Tuesday. After a silent model update on Thursday, the skill began putting revenue numbers in the wrong columns — no error, no warning, clean formatting, wrong numbers. The client called before the team noticed.

> **The March 3, 2026 update to `skill-creator` is Anthropic's answer to this problem.** It shifts the paradigm from "write instructions and hope" to "define success criteria, measure against them, and iterate."

---

## 2. What Changed in Skill Creator 2.0 (March 2026)

Anthropic introduced a four-step loop that treats skills as software requiring regression tests:

| Step | What It Does |
|------|--------------|
| **Create** | Scaffold a skill with `SKILL.md`, scripts, references, and templates as before |
| **Eval** | Define structured test cases (input + expected behavior assertions) and run them |
| **Improve** | Analyze failure logs and receive targeted suggestions for fixing `SKILL.md` instructions |
| **Benchmark** | Run the full eval suite after every change; see pass rate, duration, and token cost |

### 2.1 Evals as First-Class Citizens

You can now create formal evaluation scenarios that look like this:

```json
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF file using an appropriate PDF processing library",
    "Extracts text content from all pages without missing any pages",
    "Saves the extracted text to a file named output.txt in a clear, readable format"
  ]
}
```

Evals force you to define scenarios and assertions, run them, and iterate — which is how you discover whether your skill actually changes outcomes or just adds tokens.

### 2.2 Blind A/B Split Testing

The most significant addition is **comparator agents for A/B testing**:

- Two independent agents are spawned for each test case: one with the skill loaded, one without (or one with Version A, one with Version B).
- An AI judge evaluates both outputs **without knowing which version is which**, ensuring the comparison is unbiased.
- Results are aggregated across the full benchmark suite, giving you a statistically grounded basis for deciding which version of a skill to ship.

This is the mechanism that makes split testing actionable: you can now compare two versions of a skill's instructions and get an objective verdict.

### 2.3 Multi-Agent Parallel Testing

Benchmark mode runs many test cases simultaneously using multiple agents. This makes large eval suites practical — community reports indicate suites of 30,000–50,000 tokens are common. The benchmark output includes:

- Number of tests passed vs. failed
- Time per test case
- Token cost per run

### 2.4 Automated Improvement Suggestions

When an eval fails, the "Improve" mode reads the failure logs and suggests specific changes to your `SKILL.md`. This is not fully autonomous self-repair, but it dramatically reduces the time spent manually debugging prompt instructions.

### 2.5 Description Quality Assistance

`skill-creator` now also helps you write better `description` fields so Claude knows exactly when to trigger the skill. It analyzes your description and suggests improvements for clarity and trigger specificity.

---

## 3. Anthropic's Official Best Practices (Current)

The following principles are drawn directly from the [official Anthropic skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) page and Thariq Shihipar's March 2026 article.

### 3.1 Core Principles

**Concise is key.** The context window is a public good. Every token in `SKILL.md` competes with conversation history, the system prompt, and the actual user request. The default assumption is that Claude is already very smart — only add context Claude does not already have.

**Set appropriate degrees of freedom.** Match the level of specificity to the task's fragility:

| Freedom Level | When to Use | Example |
|---------------|-------------|---------|
| **High** (text instructions) | Multiple valid approaches; context-dependent decisions | Code review process |
| **Medium** (pseudocode/parameterised scripts) | Preferred pattern exists; some variation acceptable | Report generation template |
| **Low** (exact scripts, few parameters) | Fragile, error-prone operations; consistency critical | Database migration sequence |

**Test with all models you plan to use.** What works for Claude Opus may need more detail for Claude Haiku. Aim for instructions that work well across the model tier you intend to deploy.

### 3.2 Evaluation-Driven Development

Anthropic's official guidance now explicitly states: **create evaluations *before* writing extensive documentation.**

The recommended workflow is:

1. **Identify gaps** — Run Claude on representative tasks without a skill. Document specific failures.
2. **Create evaluations** — Build at least three scenarios that test these gaps.
3. **Establish a baseline** — Measure Claude's performance without the skill.
4. **Write minimal instructions** — Create just enough content to address the gaps and pass evaluations.
5. **Iterate** — Execute evaluations, compare against baseline, and refine.

This ensures you are solving actual problems rather than anticipating requirements that may never materialise.

### 3.3 The Claude A / Claude B Development Pattern

Anthropic recommends a hierarchical two-instance development pattern:

- **Claude A** (the expert): Helps you design and refine `SKILL.md` instructions.
- **Claude B** (the worker): Tests the skill on real tasks in a fresh session.

You observe Claude B's behavior, bring specific failure observations back to Claude A ("Claude B forgot to filter test accounts when I asked for a regional report"), and Claude A suggests targeted improvements. This cycle of observe → refine → test is more effective than assumptions-based iteration.

### 3.4 The Gotchas Section

> "The highest-signal content in any skill is the Gotchas section." — Thariq Shihipar

A great gotcha follows this pattern:

> "When doing X, Claude will default to Y, but in this codebase/context you must do Z because [reason]."

Gotchas should be built up from real failure modes encountered in production, not anticipated ones. Update this section continuously as new failure patterns emerge.

### 3.5 Progressive Disclosure

Skills are folders, not just files. The three-level loading system:

| Level | Content | When Loaded |
|-------|---------|-------------|
| **Metadata** (~100 tokens) | `name` + `description` from frontmatter | Always, at session start |
| **Instructions** (<500 lines) | Full `SKILL.md` body | When skill triggers |
| **Resources** (as needed) | `references/`, `scripts/`, `templates/` | On demand |

Keep `SKILL.md` under 500 lines. Move variant-specific details, API docs, and domain schemas into `references/` files. Tell Claude explicitly when to read each reference file.

---

## 4. How Your Local Skills Compare

### 4.1 `skill-creator`

**Strengths:**
- Excellent coverage of file structure and progressive disclosure patterns.
- Good guidance on degrees of freedom and context efficiency.
- `init_skill.py` scaffold and `quick_validate.py` provide a solid foundation.

**Gaps vs. current best practices:**
- No mention of the Create → Eval → Improve → Benchmark loop.
- `init_skill.py` does not scaffold an `evals/` directory or sample test cases.
- `quick_validate.py` only checks frontmatter syntax — it does not validate trigger quality, line count, or eval coverage.
- No guidance on the Claude A / Claude B development pattern.

### 4.2 `thariq-skill-grader`

**Strengths:**
- Comprehensive 10-dimension rubric covering trigger quality, context efficiency, gotchas, flexibility, file structure, category fit, verification, redundancy, maintainability, and data management.
- The "Redundancy Check" test (run with and without the skill; if output is ~80% the same, the skill isn't earning its context cost) is excellent.

**Gaps vs. current best practices:**
- No rubric dimension for **Evaluation & Testing** (does the skill include reproducible test cases?).
- No rubric dimension for **Benchmark Coverage** (has the skill been tested against a baseline?).
- The "Verification & Safety" dimension covers task-level verification steps but does not address skill-level regression testing.

---

## 5. Actionable Recommendations to Improve Your Skills

### 5.1 Adopt Evaluation-Driven Development Immediately

For every skill you create or update, follow this sequence before writing extensive instructions:

1. Run Claude on the target task without the skill. Note every failure.
2. Write 3–5 eval scenarios capturing those failures.
3. Confirm the evals fail without the skill (baseline).
4. Write the minimal `SKILL.md` needed to pass the evals.
5. Run the benchmark. Ship only when pass rate is satisfactory.

### 5.2 Implement Split Testing for Prompt Tuning

When iterating on an existing skill, do not overwrite it directly:

1. Create a `v2` branch of the skill with proposed changes.
2. Run both versions against your eval suite.
3. Use a blind LLM judge to compare outputs.
4. Merge the new version only if it improves the pass rate without regressing on other scenarios.

### 5.3 Update `skill-creator` to Include Evals

Modify `init_skill.py` to scaffold an `evals/` directory with a sample test case template:

```
skill-name/
├── SKILL.md
├── evals/
│   └── sample_eval.json   ← NEW: scaffold this automatically
├── scripts/
├── references/
└── templates/
```

Add a section to `skill-creator/SKILL.md` explaining the Create → Eval → Improve → Benchmark loop, and update `quick_validate.py` to warn when no `evals/` directory is present.

### 5.4 Add an Evaluation Dimension to `thariq-skill-grader`

Extend the rubric from 10 to 11 dimensions by adding:

**11. Evaluation & Testing (0-3)**

| Score | Criteria |
|-------|----------|
| 3 | At least 3 reproducible eval scenarios exist. Skill has been benchmarked against a no-skill baseline. A/B tested before any major instruction change. |
| 2 | Some eval scenarios exist but they are informal or not reproducible. |
| 1 | No evals. Skill has only been tested manually and subjectively. |
| 0 | Skill has never been tested. Author is relying entirely on intuition. |

Update the grade scale accordingly (total possible: 33 points).

### 5.5 Address Reproducibility in Evals

Community experience shows that evals depending on local environment state are fragile. To make your evals reliable:

- Use mocked or fixture data rather than live repositories.
- Run evals in a sandboxed environment so results are consistent across machines.
- Flag any eval with greater than 20% variance across runs as a systemic issue rather than a fluke.
- Consider integrating eval runs into CI/CD so regressions are caught automatically on every push.

### 5.6 Distinguish Capability Skills from Preference Skills

Anthropic now explicitly categorises skills into two types, and the evaluation strategy differs:

| Type | Definition | Eval Strategy |
|------|------------|---------------|
| **Capability skill** | Extends what the base model can do (e.g., extracting data from complex PDFs) | Test whether the capability works correctly; watch for when the base model improves enough to make the skill redundant |
| **Preference skill** | Encodes how *you* want work done (e.g., formatting standups in your company's style) | Test whether the output matches your style preferences; benchmarks catch drift when the model changes |

---

## 6. Checklist for a Best-Practice Skill (Updated)

Before shipping any skill, verify:

**Core Quality**
- [ ] Description is specific, written in third person, and includes both what the skill does and when to use it
- [ ] `SKILL.md` body is under 500 lines
- [ ] Additional details are in separate `references/` files
- [ ] No time-sensitive information (or in an "old patterns" section)
- [ ] Consistent terminology throughout
- [ ] Examples are concrete, not abstract
- [ ] File references are one level deep
- [ ] Progressive disclosure used appropriately

**Gotchas & Edge Cases**
- [ ] A dedicated Gotchas section exists
- [ ] Gotchas are drawn from real failure modes, not anticipated ones
- [ ] Gotchas are updated after each production incident

**Evaluation & Testing** *(new)*
- [ ] At least 3 eval scenarios created before writing extensive instructions
- [ ] Baseline established (evals confirmed to fail without the skill)
- [ ] Skill benchmarked and pass rate documented
- [ ] A/B tested against previous version before any major instruction change
- [ ] Evals use reproducible, mocked data where possible
- [ ] Tested with Haiku, Sonnet, and Opus

**Code and Scripts**
- [ ] Scripts solve problems rather than punt to Claude
- [ ] Error handling is explicit and helpful
- [ ] No "voodoo constants" (all values justified)
- [ ] Required packages listed and verified as available
- [ ] No Windows-style paths (all forward slashes)
- [ ] Validation/verification steps for critical operations

---

## 7. Key Takeaways

The central shift in Anthropic's approach is from **"does this feel right?"** to **"does this pass the benchmark?"** This mirrors how mature software teams treat code: no feature ships without tests, and no prompt change ships without evals.

Your existing `thariq-skill-grader` and `skill-creator` skills are well-structured and cover information architecture excellently. The primary gap is the absence of evaluation infrastructure — both in the tooling (no `evals/` scaffold, no eval-aware validation) and in the rubric (no testing dimension). Closing this gap will bring your skill development workflow into alignment with Anthropic's current best practices and make your skills significantly more resilient to model updates.

---

*Generated by Manus · April 21, 2026*
