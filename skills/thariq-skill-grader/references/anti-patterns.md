# Anti-Patterns: Concrete Failure Modes

Catalog of bad patterns observed in real skills. Each entry: **Bad** snippet → **Why it fails** → **Fixed** version.

When grading, match the target skill against this catalog. Citing a specific anti-pattern in the grade card is more actionable than "description is weak."

---

## Description anti-patterns (Dimension 1)

### AP-D1: The marketing summary

**Bad:**
```yaml
description: A powerful and flexible tool for working with PDFs and document workflows.
```

**Why it fails:** Marketing voice gives Claude no trigger conditions. "Powerful and flexible" is for landing pages, not for matching user intent. Claude can't decide *when* to fire.

**Fixed:**
```yaml
description: Extracts text and tables from PDFs, fills PDF forms, and merges PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or extraction.
```

---

### AP-D2: The single-trigger description

**Bad:**
```yaml
description: Use when user asks to deploy.
```

**Why it fails:** Real users phrase requests many ways ("ship it", "push to prod", "release this"). One trigger phrase causes false negatives.

**Fixed:**
```yaml
description: Deploy the application to production. Use when user says "deploy", "ship", "push to prod", "release", or asks to make changes live.
```

---

### AP-D3: Ambiguous overlap with other skills

**Bad:** Two skills both have description "Reviews code for issues" — `/review` and `/lint`.

**Why it fails:** Claude has no basis to pick one over the other. Will fire arbitrarily.

**Fixed:** Each description includes negative triggers:
```yaml
# /review
description: Comprehensive code review against project conventions and architecture. For mechanical style fixes, use /lint instead.

# /lint
description: Auto-fix style violations (formatting, unused imports, naming). For architectural review, use /review instead.
```

---

### AP-D4: Description that exceeds the budget

**Bad:** A 2,000-character description packed with examples.

**Why it fails:** Truncated at 1,536 chars in the skill listing. Trigger keywords past the cap don't exist for matching purposes — Claude literally can't see them.

**Fixed:** Front-load trigger keywords; push examples and detail to `when_to_use` (still subject to the same combined cap) or to the SKILL.md body (only loaded when invoked).

---

## Context efficiency anti-patterns (Dimension 2)

### AP-C1: Re-teaching what Claude knows

**Bad:**
```markdown
## What is Git?

Git is a distributed version control system created by Linus Torvalds in 2005...
```

**Why it fails:** Burns tokens on something Claude already knows perfectly. The skill should encode *organizational* knowledge (your repo's commit conventions, your branching model), not basics.

**Fixed:** Cut entirely. Start with the project-specific guidance:
```markdown
## Commit conventions for this repo

- Conventional Commits format (feat:, fix:, chore:)
- Trailer required: `Refs: PROJ-1234`
```

---

### AP-C2: Inline reference docs

**Bad:** A 600-line SKILL.md that includes the full OpenAPI spec inline.

**Why it fails:** The full spec loads into context every time the skill fires, even when only a subset is relevant. SKILL.md hard limit is 500 lines for a reason.

**Fixed:** Move the spec to `references/openapi-spec.md`. SKILL.md says "for full spec, see references/openapi-spec.md" so Claude loads it on demand.

---

### AP-C3: Duplicated content across files

**Bad:** The same "verification checklist" appears in SKILL.md and in `references/verification.md`.

**Why it fails:** When both load, Claude sees duplicate instructions and may treat them as a sequence ("verify, then verify again"). Wastes tokens and creates ambiguity.

**Fixed:** Single source of truth. SKILL.md either contains the checklist OR points at it — never both.

---

## Gotchas anti-patterns (Dimension 3)

### AP-G1: Generic gotchas that aren't gotchas

**Bad:**
```markdown
## Gotchas
- Always test your code
- Be careful with production data
- Make sure to commit your changes
```

**Why it fails:** Universal advice, not real failure modes. Provides zero signal.

**Fixed:** Specific, observed traps:
```markdown
## Gotchas
- The `/users/{id}` endpoint returns 200 with empty body for soft-deleted users — check `deleted_at` field.
- `httpx` defaults to no timeout; always pass `timeout=10` or it will hang on slow upstream.
```

---

### AP-G2: Pristine happy-path skill

**Bad:** No gotchas section at all. Skill only documents the success path.

**Why it fails:** Real skills accumulate failure-mode notes from use. Absence of gotchas means the skill hasn't been battle-tested OR the author hasn't recorded what they learned. Either way, future users will hit the same walls.

**Fixed:** Even a new skill should document at least one anticipated failure mode and how to recognize it.

---

## Flexibility anti-patterns (Dimension 4)

### AP-F1: Over-templated output

**Bad:**
```markdown
Always respond in this exact format:

## Summary
[exactly 2 sentences]

## Findings
1. [finding 1]
2. [finding 2]
3. [finding 3]
```

**Why it fails:** Real findings don't come in groups of three. Forced structure produces padding or truncation. Rigid format is rarely justified outside of machine-parsed output.

**Fixed:** Specify what must be present, not exact shape:
```markdown
Include a summary, the findings (as many as exist), and at least one recommended next action.
```

---

### AP-F2: Step-by-step where judgment is needed

**Bad:**
```markdown
1. Read the file
2. Find the function
3. Edit line 42 to be `return True`
4. Save the file
```

**Why it fails:** Either the bug is exactly as described (then a script would do it) or it isn't (then Claude needs judgment, not a script). Encoding specific lines makes the skill rot the moment the file changes.

**Fixed:** Encode the *approach*, not the exact steps. Or move the deterministic work to a `scripts/` file and have SKILL.md invoke it.

---

## File structure anti-patterns (Dimension 5)

### AP-S1: Everything in SKILL.md

**Bad:** A 1,200-line SKILL.md with embedded code, examples, references, and templates.

**Why it fails:** Loads in full on every invocation. Most content is irrelevant to any single use.

**Fixed:** Split: orchestration in SKILL.md (under 500 lines), details in `references/`, executable code in `scripts/`, output templates in `assets/` or `templates/`.

---

### AP-S2: Nested reference chains

**Bad:** SKILL.md references `references/overview.md` which references `references/api/v2/auth.md` which references `references/api/v2/auth/oauth.md`.

**Why it fails:** Claude has to load multiple files to find the actual content. Defeats the point of progressive disclosure.

**Fixed:** Keep references one level deep from SKILL.md. If a reference doc needs sub-references, it's probably too big.

---

### AP-S3: `name` mismatch with directory

**Bad:** `~/.claude/skills/deploy-prod/SKILL.md` with `name: deploy_to_production`.

**Why it fails:** Slash command becomes `/deploy_to_production` but the directory says `deploy-prod`. Confusing in error messages, breaks intuition. Also: underscores aren't valid in skill names per spec.

**Fixed:** Either omit `name` (defaults to directory) or set `name: deploy-prod` to match.

---

## Category fit anti-patterns (Dimension 6)

### AP-CF1: The Swiss Army knife

**Bad:** A skill called `/dev-utils` that handles deployment, testing, code review, and changelog generation.

**Why it fails:** Description can't articulate triggers cleanly. Claude won't know when to fire it. Should be 4 separate skills.

**Fixed:** Split into `/deploy`, `/test`, `/review`, `/changelog`. Each gets a sharp description and clear scope.

---

## Verification anti-patterns (Dimension 7)

### AP-V1: Destructive operations without guardrails

**Bad:**
```markdown
1. Delete all files matching the pattern
2. Drop the database table
3. Force-push to main
```

No confirmation step, no dry-run option, no scope check.

**Why it fails:** A skill that can lose data without confirmation will eventually lose data. Especially dangerous for `disable-model-invocation: false` skills (auto-firable destructive skills are a footgun).

**Fixed:** Add `disable-model-invocation: true`, list the affected scope before acting, require explicit user confirmation for destructive operations, or use hooks to gate the destructive tool calls.

---

### AP-V2: Claims success without checking

**Bad:**
```markdown
1. Run the deploy
2. Tell user "Deploy complete"
```

**Why it fails:** "Deploy complete" because the script returned 0, not because the deploy actually worked. Real verification means checking the deploy artifact / health endpoint / log output.

**Fixed:** After action, fetch evidence (curl health endpoint, query DB, check logs) and report based on the evidence, not the exit code.

---

## Maintainability anti-patterns (Dimension 9)

### AP-M1: Dated tool references with no review marker

**Bad:** Skill references `prettier-eslint` (deprecated) or `pip` workflows in a `uv`-using project.

**Why it fails:** Skills rot. A tool reference without a "last reviewed" date or owner becomes a liability — new users follow advice that doesn't apply anymore.

**Fixed:** Either remove the dated reference or annotate it: `<!-- reviewed: 2026-01-15 — still current as of Prettier 3.x -->`.

---

## Data management anti-patterns (Dimension 10)

### AP-DM1: State stored in skill folder

**Bad:** Skill writes `cache.json` to its own directory.

**Why it fails:** Plugin upgrades and skill reinstalls overwrite the folder. State is lost. Also: skill folders can be shared/version-controlled, so cache pollutes git.

**Fixed:** Use `${CLAUDE_PLUGIN_DATA}` (for plugin skills) or a project-local `.claude/` directory for state. Reference bundled assets via `${CLAUDE_SKILL_DIR}` (read-only).

---

### AP-DM2: Secrets in skill files

**Bad:** API key hardcoded in SKILL.md or in a bundled script.

**Why it fails:** Skills get shared, committed, distributed. Secrets in skills are secrets in public.

**Fixed:** Read from environment variables. Document the required env vars in SKILL.md, never the values.

---

## How to use this catalog when grading

1. **Cite the AP code in the grade card** — "Description scores 1: AP-D1 (marketing summary) and AP-D3 (overlaps with /review)."
2. **Suggest the fix from the entry** — graders shouldn't have to reinvent solutions for known problems.
3. **If a skill exhibits no anti-pattern but still feels off**, that's a candidate to add a new entry to this catalog.
