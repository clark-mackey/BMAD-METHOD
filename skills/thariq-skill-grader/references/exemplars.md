# Exemplars: Gold-Standard Skills

Annotated reference for what "3/3" looks like across the rubric. Source: skills shipped by Anthropic in `anthropics/skills` repo and bundled with Claude Code.

When grading, compare the target skill against the exemplar in the same category. If the target is structurally similar but worse on a dimension, the exemplar shows what better looks like.

---

## skill-creator (Reference: meta-skill / authoring tool)

**Why exemplary:** Self-referential proof — Anthropic's own guidance for how to write skills. If a skill couldn't pass its own bar, it would be embarrassing.

**What to study:**
- **Description trigger:** Includes both the task ("create or improve a skill") and the situation ("when user wants to add a slash command, package a workflow"). Multiple phrasings.
- **Progressive disclosure:** Heavy use of `references/` for detailed checklists; SKILL.md stays focused on the workflow.
- **Gotchas section is real:** Documents specific failure modes from observed usage, not generic advice.
- **Output format:** Doesn't dictate exact wording for the generated skill — gives constraints and lets Claude compose.

**Pattern to imitate:** Skill that *produces* other artifacts (skills, configs, docs) — separate the "what to produce" from "how to produce it" via references.

---

## claude-api (Reference: library/API knowledge with auto-trigger)

**Why exemplary:** Demonstrates auto-triggering on file content (imports `anthropic`/`@anthropic-ai/sdk`) rather than relying on user phrasing. The description doubles as a trigger specification with explicit TRIGGER/SKIP rules.

**What to study:**
- **Description includes negative triggers:** Explicit "SKIP" list (`openai` imports, generic provider-neutral code). This is the strongest disambiguation pattern in any shipped skill.
- **Trigger by code signal, not user phrase:** "code imports `anthropic`" is a deterministic signal Claude can verify by reading the file.
- **Includes migration guidance:** Bonus value beyond the core "how to use" — handles model version migrations.

**Pattern to imitate:** Reference skills (style guides, API conventions, framework knowledge) — anchor triggers to file content (imports, file extensions, framework markers) rather than user requests, and always include SKIP rules.

---

## frontend-design (Reference: creative generation with quality bar)

**Why exemplary:** Encodes taste and quality standards without becoming rigid. Specifies what "good" looks like and lets Claude compose.

**What to study:**
- **Quality criteria over recipes:** Lists what makes a design good (hierarchy, contrast, spacing) rather than templating exact components.
- **Constraints, not scripts:** "Use a 4px grid" is a constraint; "place button at line 42" would be a script. The skill picks constraints.
- **Visual examples in references:** Heavy use of supporting files for examples that would bloat SKILL.md.

**Pattern to imitate:** Generative skills (copywriting, design, content) — encode taste as constraints and exemplars in references, never lock down output structure.

---

## mcp-builder (Reference: code scaffolding)

**Why exemplary:** Generates working code with verification baked in. Doesn't trust the model to "just write an MCP server" — provides scaffolding and validates against spec.

**What to study:**
- **Verification baked in:** Includes "test the server starts" as a non-skippable step.
- **Bundled scripts do the precise work:** SKILL.md orchestrates, scripts/ does the deterministic operations (file scaffolding, JSON schema validation).
- **Templates as defaults:** Provides starter templates but documents when to deviate.

**Pattern to imitate:** Code-generation skills — push deterministic work to scripts, keep SKILL.md as the orchestration layer, always include a verification step.

---

## /simplify (Bundled: code-quality task)

**Why exemplary:** Tightly scoped task skill. Description tells Claude exactly when to fire and exactly what to do. No scope creep.

**What to study:**
- **Tight description:** No padding, no marketing language. States what it does and triggers.
- **No `disable-model-invocation`:** Claude *should* fire this when reviewing changes. Trust signaled by the lack of restriction.
- **Body is procedural, not encyclopedic:** Walks through the simplification workflow without re-explaining what good code is.

**Pattern to imitate:** Quality-review skills (lint, audit, review) — make them auto-firable, keep instructions procedural.

---

## /debug (Bundled: workflow with branching logic)

**Why exemplary:** Encodes a debugging methodology (systematic isolation, hypothesis testing) without becoming a wall of conditionals.

**What to study:**
- **Methodology over checklist:** Teaches a debugging *approach* rather than a fixed sequence — Claude adapts to the actual bug.
- **Explicit anti-patterns:** Calls out behaviors to avoid ("don't shotgun fixes hoping one works").
- **Standing instructions:** Written so the guidance applies throughout the debugging session, not just step 1.

**Pattern to imitate:** Methodology skills (debugging, investigation, planning) — encode the *approach* and explicitly forbid common failure modes; assume the skill body persists for the whole task.

---

## /loop (Bundled: harness-coordinated task)

**Why exemplary:** Skill whose execution depends on harness features (recurring fire). Documents the harness contract clearly so users understand what they're invoking.

**What to study:**
- **Documents the harness contract:** Explains what "recurring" means in concrete terms (interval, self-pacing, when it stops).
- **Negative triggers:** Explicit "Do NOT invoke for one-off tasks" prevents misfire.

**Pattern to imitate:** Harness-coordinated skills (scheduling, background tasks, notifications) — document the system behavior, not just the user-facing intent.

---

## How to use this reference when grading

1. **Match the target skill to the closest exemplar** by category (meta-skill, reference, creative, code-gen, task, methodology, harness).
2. **Compare dimension-by-dimension** — if exemplar scores 3 on Description and target's description has fewer trigger phrases, that's a 1–2.
3. **Don't expect identical structure** — exemplars show *patterns*, not templates. A skill can be exemplary by following a different but coherent pattern.
4. **When no exemplar matches, flag it:** A skill that fits no known pattern is either pioneering (rare, document it) or miscategorized (common, suggest splitting).
