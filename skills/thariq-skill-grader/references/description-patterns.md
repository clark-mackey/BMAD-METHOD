# Description Patterns by Skill Type

Templates for the highest-leverage field. Combined `description` + `when_to_use` is capped at 1,536 chars in the skill listing — front-load triggers, push examples to `when_to_use`.

The basic shape is always: **[Capability]. Use when [trigger conditions]. For [overlap case], use [other-skill] instead.**

---

## Pattern 1: Workflow / Task Skills

For skills that perform a discrete action: deploy, commit, ship, summarize PR.

**Template:**
```yaml
description: [Verb] [the thing] [optional: scope/destination]. Use when user says "[trigger 1]", "[trigger 2]", "[trigger 3]", or asks to [common phrasings]. For [related but different action], use [other-skill] instead.
disable-model-invocation: true  # Almost always true for action skills with side effects
```

**Example:**
```yaml
description: Stage and commit current changes following Conventional Commits format. Use when user says "commit", "save changes", "make a commit", or asks to checkpoint work. For pushing to remote, use /push instead. For PR creation, use /pr instead.
```

**Trigger writing checklist:**
- 3+ phrasings of the same intent
- 1+ negative trigger (point at adjacent skills)
- Verb-led, not noun-led ("Commit changes" beats "Commits feature")
- `disable-model-invocation: true` if it has side effects you don't want Claude initiating

---

## Pattern 2: Reference / Knowledge Skills

For skills that inject conventions, style guides, or framework knowledge. Should auto-trigger by file content.

**Template:**
```yaml
description: [What knowledge it provides] for [domain/framework]. Auto-trigger when [code signal: imports/extensions/markers] OR when user asks about [topic phrasings]. SKIP when [exclusion conditions].
when_to_use: |
  TRIGGER: [specific file patterns, imports, frameworks]
  SKIP: [provider-neutral code, other-framework files, generic ML]
```

**Example (modeled on /claude-api):**
```yaml
description: Build, debug, and optimize Claude API / Anthropic SDK apps. Includes prompt caching guidance and model migration support.
when_to_use: |
  TRIGGER: code imports `anthropic` or `@anthropic-ai/sdk`; user asks about Anthropic SDK, prompt caching, tool use, or thinking; user adds/tunes a Claude feature in a file.
  SKIP: file imports `openai` or other-provider SDK; filename like `*-openai.py`; provider-neutral code; general programming/ML.
```

**Trigger writing checklist:**
- Anchor to **deterministic file signals** (imports, extensions, framework markers) when possible
- TRIGGER list and SKIP list are equally important — without SKIP, the skill fires too broadly
- Don't include `disable-model-invocation` — reference skills should auto-fire

---

## Pattern 3: Creative / Generative Skills

For skills that produce creative output: copywriting, design, content.

**Template:**
```yaml
description: Generate [output type] following [brand/style markers]. Use when user asks to [common request phrasings] or when working with [content type]. For [related but different output], use [other-skill] instead.
```

**Example:**
```yaml
description: Generate landing page hero sections following Bencivenga sales-letter principles. Use when user asks for a hero, headline, above-the-fold copy, or landing page opener. For full-page sales letters, use /bencivenga-sro-copywriter instead. For short product descriptions, use /copywriting instead.
```

**Trigger writing checklist:**
- Identify the **output artifact** clearly ("landing page hero" not "marketing copy")
- Disambiguate from skills with similar artifacts but different scope/style
- Avoid quality adjectives ("powerful", "beautiful") — they don't help triggering

---

## Pattern 4: Methodology / Approach Skills

For skills that encode a *way of working*: debugging, brainstorming, planning.

**Template:**
```yaml
description: [Approach name] methodology for [problem domain]. Use when [problem type] requires [type of thinking]. For [adjacent methodology], use [other-skill] instead.
```

**Example:**
```yaml
description: Systematic debugging via hypothesis isolation. Use when a bug's root cause is unclear, when shotgun fixes have failed, or when user says "debug", "investigate", or "this is broken and I don't know why". For known-cause fixes, edit the code directly. For performance issues, use /performance-optimization instead.
```

**Trigger writing checklist:**
- State the **methodology**, not the topic ("hypothesis isolation" not "fixes bugs")
- Include the **failure-mode trigger** ("when shotgun fixes have failed") — this is when methodology skills earn their keep
- Forbid the lazy alternative explicitly ("don't shotgun fixes")

---

## Pattern 5: Auditing / Review Skills

For skills that evaluate something: code review, security review, accessibility audit.

**Template:**
```yaml
description: [Audit type] of [target] against [standard/criteria]. Use when [trigger conditions] or after [pre-condition action]. For [adjacent audit], use [other-skill] instead.
```

**Example:**
```yaml
description: Security review of changed code against OWASP top 10 and project security standards. Use when user says "security review", "check for vulnerabilities", or after major changes to auth/data-handling code. For accessibility audits, use /accessibility-auditor instead. For general code review, use /review instead.
```

**Trigger writing checklist:**
- Name the **standard** being audited against (OWASP, WCAG, project conventions)
- Include the **pre-condition trigger** ("after auth changes") — audit skills are often triggered by *what just happened*
- Most audit skills should auto-fire (no `disable-model-invocation`)

---

## Pattern 6: Harness-Coordinated Skills

For skills that use harness features (scheduling, hooks, subagents).

**Template:**
```yaml
description: [Capability] [via harness mechanism]. Use when [user intent matches]. Do NOT use for [common misfire case].
```

**Example (modeled on /loop):**
```yaml
description: Run a prompt or slash command on a recurring interval (e.g. /loop 5m /foo). Omit the interval to let the model self-pace. Use when the user wants to set up a recurring task, poll for status, or run something repeatedly on an interval. Do NOT invoke for one-off tasks.
```

**Trigger writing checklist:**
- Show the **invocation syntax** in the description — users need to see how to call it
- Include explicit **"Do NOT invoke for X"** to prevent obvious misfires
- Document the harness contract briefly (interval, behavior)

---

## When to use `when_to_use` vs. inline description

**Inline (in `description`)** — for the highest-priority triggers, the ones that must match.

**`when_to_use`** — for:
- Long lists of TRIGGER/SKIP code signals (Pattern 2)
- Example user requests (Patterns 1, 3, 5)
- Edge cases that don't fit a clean sentence

Both count toward the 1,536-char combined cap. If you're at the cap, cut examples first, keep triggers.

---

## Pre-flight description checklist

Before shipping a skill, verify the description:

- [ ] Names a capability (verb + object), not a quality
- [ ] Lists 2–4 trigger phrasings users would actually say
- [ ] Includes ≥1 negative trigger pointing at adjacent skills
- [ ] Combined with `when_to_use`, stays under 1,536 chars
- [ ] No marketing language ("powerful", "flexible", "easy")
- [ ] Front-loaded — most important trigger keywords in the first sentence
- [ ] If reference/auto-trigger skill: includes deterministic file signals AND a SKIP list

A description that fails 2+ of these will score 1/3 on Dimension 1 even if the rest of the skill is great.
