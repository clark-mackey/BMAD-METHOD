# Skill Lifecycle & Hooks Integration

How skills behave once invoked, how they survive (or don't) auto-compaction, and when to use hooks instead of skill instructions for guaranteed behavior.

---

## Skill content lifecycle (the basics)

When you or Claude invoke a skill:

1. The **rendered SKILL.md content** (frontmatter stripped, `${VAR}` substitutions applied, `` !`cmd` `` blocks executed) enters the conversation as a **single message**.
2. That message **stays in context for the rest of the session**. Claude Code does NOT re-read SKILL.md on later turns.
3. Subsequent turns see the skill content as historical context, not as a fresh instruction.

**Implication for skill authors:** Write the body as **standing instructions** that should apply throughout a task, not as one-time procedural steps that assume re-execution.

### Bad (assumes re-reading):
```markdown
First, read this file. Then, do X. Now, having done X, proceed to Y.
```

### Good (standing instructions):
```markdown
When working on this task, always:
- Do X before Y
- Verify Z after each change
- Stop if W is detected
```

---

## Auto-compaction behavior

When the conversation hits its token limit, Claude Code summarizes older turns to free space. This affects skills:

| Behavior | Detail |
|----------|--------|
| Most-recent invocation preserved | Each skill's *most recent* invocation is re-attached after the summary |
| Per-skill budget | First **5,000 tokens** of each invoked skill are kept |
| Combined budget | All re-attached skills share **25,000 tokens** total |
| Drop order | Budget fills from most-recent skill backward — older skills drop first |
| Re-invocation | Calling the skill again restores its full content |

**Implication for skill authors:**
- **Front-load critical instructions** in the first 5,000 tokens. Anything past that gets dropped on first compaction.
- **If a skill seems to "stop working" mid-session**, the content may have been dropped. Re-invoke to restore.
- **Long reference material belongs in `references/`**, not inline — references load on-demand and get fresh budget each time.

---

## Hooks vs. skill instructions

A skill says "you should do X." A hook *makes* X happen.

| Mechanism | Strength | Use when |
|-----------|----------|----------|
| Skill instruction | Suggestion — Claude can override | Best-practice guidance, methodology, taste |
| Hook | Hard enforcement — Claude can't bypass | Safety guardrails, mandatory checks, audit trails |

**Rule of thumb:** If "Claude ignored the instruction" would be a problem, use a hook. If "Claude weighed the instruction against context" is the right behavior, use a skill instruction.

---

## When skills should use hooks

The `hooks` frontmatter field scopes hooks to the skill's lifecycle. Use it when:

### 1. Mandatory pre/post actions

```yaml
---
name: deploy
hooks:
  PreToolUse:
    - matcher: { tool_name: Bash }
      hooks:
        - { type: command, command: "${CLAUDE_SKILL_DIR}/scripts/audit-cmd.sh" }
  PostToolUse:
    - matcher: { tool_name: Bash }
      hooks:
        - { type: command, command: "${CLAUDE_SKILL_DIR}/scripts/log-result.sh" }
---
```

Every Bash command run while the skill is active goes through audit + log. The skill body can't bypass this.

### 2. Verification gates

A `PostToolUse` hook can refuse to let the conversation proceed if a check fails — stronger than a skill saying "verify your work."

### 3. Destructive-action guards

A `PreToolUse` hook on `Bash(rm *)` can require confirmation, even if the skill body forgets to. Belt + suspenders for high-stakes skills.

---

## When NOT to use hooks

Hooks add complexity and run on every matching event. Don't use them for:

- **Stylistic preferences** ("commit messages should be lowercase") — skill instruction is enough
- **Anything Claude does well already** — hooks aren't a substitute for trusting Claude
- **Personalization** — global settings or CLAUDE.md cover this better
- **Long-running operations** — hooks should be fast; offload work to scripts the skill invokes explicitly

---

## Subagent execution (`context: fork`)

A skill with `context: fork` runs in an isolated subagent context. The skill body becomes the subagent's prompt; the agent type (`Explore`, `Plan`, `general-purpose`, or any custom agent) determines tools and model.

**Use when:**
- The task needs heavy exploration that would bloat main context (use `Explore`)
- The task needs planning isolation (use `Plan`)
- The skill should produce a single result without polluting conversation state

**Don't use when:**
- The skill is a reference/style guide — subagent has no main-context awareness, so guidance "for the current conversation" is meaningless
- The skill needs to interact iteratively with the user — subagents don't take follow-ups

### The pairing with hooks

`context: fork` + `hooks` is a powerful pattern: the subagent runs in isolation but every tool call still goes through the skill's hooks. This gives you both isolation and enforcement.

---

## Grading implications

When grading Dimension 7 (Verification & Safety):

| Skill characteristic | Score |
|---------------------|-------|
| Says "verify your work" but no enforcement | 1 |
| Documents a verification step Claude *could* skip | 2 |
| Uses a `PostToolUse` hook to enforce verification | 3 |
| Destructive action with no `disable-model-invocation`, no hooks, no confirmation step | 0 |

When grading Dimension 2 (Context Efficiency):

- A skill whose first 5,000 tokens contain only setup/preamble scores 1 — critical instructions get dropped on compaction
- A skill that uses `references/` for detail and keeps SKILL.md under 5,000 tokens scores 2–3

When grading Dimension 4 (Flexibility):

- A skill that uses hooks for the "must enforce" behavior and instructions for the "best practice" behavior scores 3 — correct tool for each job
- A skill that tries to enforce everything via instructions ("always", "never", "MUST") with no hook backing scores 1–2 — over-rigid and bypassable
