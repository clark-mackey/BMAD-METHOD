# Official Anthropic Skill Standards

## Sources

| Source | URL | Authority |
|--------|-----|-----------|
| Claude Code Docs | https://code.claude.com/docs/en/skills | Official implementation docs (always current) |
| Agent Skills Spec | https://agentskills.io/specification | Open standard spec (cross-platform) |
| Anthropic Skills Repo | https://github.com/anthropics/skills | Official example skills (17 published) |
| Thariq's Article | https://x.com/trq212/article/2033949937936085378 | Internal best practices from Anthropic engineer |

Best practices authoring guide: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices

## Frontmatter Spec

All frontmatter fields are optional. Only `description` is recommended so Claude knows when to use the skill.

### Core Fields (Claude Code)

| Field | Required | Constraints | Notes |
|-------|----------|-------------|-------|
| `name` | No | Lowercase letters, numbers, hyphens only. Max 64 chars. No leading/trailing/consecutive hyphens. | If omitted, defaults to the parent directory name. Becomes the `/slash-command`. |
| `description` | Recommended | Combined `description` + `when_to_use` truncated at **1,536 chars** in the skill listing. | If omitted, uses the first paragraph of the markdown content. Front-load the key use case. |
| `when_to_use` | No | Counts toward the 1,536-char cap with `description`. | Additional trigger phrases or example requests. Appended to `description` in the skill listing. |

### Claude Code Extensions (beyond the open standard)

| Field | Purpose |
|-------|---------|
| `disable-model-invocation` | `true` = only user can invoke via `/name`. Removes from Claude's context entirely. |
| `user-invocable` | `false` = hidden from `/` menu. Only Claude can invoke. Note: this only controls menu visibility — use `disable-model-invocation` to block programmatic invocation. |
| `argument-hint` | Autocomplete hint, e.g. `[issue-number]` or `[filename] [format]` |
| `model` | Override model when skill is active |
| `effort` | Override effort level: `low`, `medium`, `high`, `xhigh`, `max`. Available levels depend on the model. |
| `context` | `fork` = run in isolated subagent context |
| `agent` | Which subagent type for `context: fork` (e.g., `Explore`, `Plan`, `general-purpose`). Defaults to `general-purpose`. |
| `hooks` | Hooks scoped to this skill's lifecycle |
| `allowed-tools` | Tools pre-approved while skill is active. Space-delimited string or YAML list. Does NOT restrict tools — only grants permission. |
| `paths` | Glob patterns limiting auto-activation to matching files. Comma-separated string or YAML list. Same format as path-specific CLAUDE.md rules. Useful for monorepo scoping. |
| `shell` | `bash` (default) or `powershell` for `` !`command` `` injection. `powershell` requires `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`. |

### Open-Standard Fields (Agent Skills spec, cross-platform)

These belong to the open Agent Skills spec and may be honored by other tools, but are not documented in the current Claude Code frontmatter reference:

| Field | Constraints |
|-------|-------------|
| `license` | License name or reference to bundled LICENSE file |
| `compatibility` | Max 500 chars. Environment requirements (packages, network, etc.) |
| `metadata` | Arbitrary key-value map (string keys, string values) |

### String Substitutions

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | All arguments passed when invoking. If absent from content, appended as `ARGUMENTS: <value>`. |
| `$ARGUMENTS[N]` / `$N` | Specific argument by 0-based index. Shell-style quoting — wrap multi-word args in quotes. |
| `${CLAUDE_SESSION_ID}` | Current session ID |
| `${CLAUDE_SKILL_DIR}` | Directory containing SKILL.md (for plugin skills, the skill subdir, not plugin root) |
| `` !`command` `` | Inline shell command. Output replaces the placeholder before Claude sees content. |
| ` ```! ` fenced block | Multi-line shell block. Each line runs; combined output replaces the block. |

Shell injection (`!`) can be disabled globally via `"disableSkillShellExecution": true` in settings (most useful in managed settings).

## Description Field Rules

The description is the **most critical field** for grading. It determines when Claude activates the skill.

**Must include:**
- What the skill does (capability)
- When to use it (trigger conditions — specific phrases, keywords, situations)

**Should include:**
- Negative triggers ("For X, use skill-Y instead") to disambiguate from similar skills
- Multiple phrasings users might use

**Must not:**
- Cause combined `description` + `when_to_use` to exceed **1,536 characters** (gets truncated in the skill listing)
- Be a marketing summary ("A helpful skill for productivity")
- Be too vague ("Helps with PDFs")

**Good example:**
```
Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs.
Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
```

**Bad example:**
```
Helps with PDFs.
```

## Name Field Rules

- Optional — defaults to parent directory name if omitted
- Max 64 characters
- Lowercase letters (`a-z`), numbers, hyphens only
- No leading, trailing, or consecutive hyphens
- When set, should match the parent directory name to avoid confusion

## Directory Structure

```
skill-name/
├── SKILL.md          # Required: metadata + instructions
├── scripts/          # Optional: executable code
├── references/       # Optional: documentation loaded on demand
├── assets/           # Optional: templates, resources (spec standard)
└── templates/        # Alternative to assets/ (common in practice)
```

## Progressive Disclosure

Three-level loading:
1. **Metadata** (~100 tokens): `name` + `description` (+ `when_to_use`) loaded at startup for all skills
2. **Instructions** (<5000 tokens recommended): Full SKILL.md body loaded when skill activates
3. **Resources** (as needed): Files in scripts/, references/, assets/ loaded only when required

**Hard rules:**
- Keep SKILL.md under **500 lines**
- Keep file references **one level deep** from SKILL.md (no nested chains)
- Reference supporting files from SKILL.md so Claude knows they exist and when to read them

## Skill Content Lifecycle

When a skill is invoked, the rendered SKILL.md content enters the conversation as a **single message** and stays for the rest of the session. Claude Code does NOT re-read the skill file on later turns.

**Implication for skill authors:** Write guidance as **standing instructions** that apply throughout a task, not one-time steps that assume re-reading.

**Auto-compaction behavior:**
- Most-recent invocation of each skill is re-attached after summarization
- First **5,000 tokens** of each invoked skill preserved
- Combined budget across re-attached skills: **25,000 tokens**
- Budget fills from most-recently invoked skill backward — older skills can be dropped entirely

If a skill seems to "stop working" after the first response, the content is usually still present and Claude is simply choosing other approaches. Strengthen the description and instructions, or use hooks to enforce behavior. After heavy compaction, re-invoke the skill to restore full content.

## Invocation Control Matrix (Claude Code)

| Config | User can invoke | Claude can invoke | When loaded |
|--------|----------------|-------------------|-------------|
| (default) | Yes | Yes | Description always in context; full skill on invoke |
| `disable-model-invocation: true` | Yes | No | Description NOT in context; loads on user invoke only |
| `user-invocable: false` | No | Yes | Description always in context; loads on Claude invoke |

Note: subagents with preloaded skills work differently — full skill content is injected at startup rather than only on invoke.

## Skill Permission Rules

In `/permissions` settings, control Claude's skill access with:

```
Skill                  # Deny all skills (deny rule)
Skill(commit)          # Allow/deny specific skill by exact name
Skill(review-pr *)     # Prefix match — any arguments
```

To hide a skill from Claude entirely, use `disable-model-invocation: true` (removes from context).

## Context Budget

Claude Code allocates **1% of the context window** for skill descriptions, with an **8,000 character fallback**. Override with `SLASH_COMMAND_TOOL_CHAR_BUDGET` env var.

If you have many skills, descriptions get shortened to fit — which can strip the keywords Claude needs to match a request. All skill *names* are always included; only descriptions get trimmed.

To raise the limit, set `SLASH_COMMAND_TOOL_CHAR_BUDGET`. Or trim `description` and `when_to_use` at the source — each entry's combined text is capped at 1,536 chars regardless of total budget.

Run `/context` to check for warnings.

## Live Change Detection & Nested Discovery

- Adding/editing/removing skills under `~/.claude/skills/`, project `.claude/skills/`, or `.claude/skills/` inside an `--add-dir` directory takes effect **within the current session** without restarting.
- Creating a new top-level skills directory that didn't exist at session start requires a restart.
- When working with files in subdirectories, Claude Code automatically discovers skills from nested `.claude/skills/` directories — supports monorepo setups.

## Official Example Patterns (anthropics/skills repo)

17 published skills showing real patterns:

**Production skills** (source-available, used in Claude products):
- `docx`, `pdf`, `pptx`, `xlsx` — Document creation/editing

**Open source examples:**
- `skill-creator` — Meta-skill for creating other skills (includes best practices)
- `claude-api` — Library/API reference pattern with auto-trigger on imports
- `frontend-design` — Creative generation with quality standards
- `webapp-testing` — Verification skill with Playwright integration
- `mcp-builder` — Code scaffolding pattern

**Common patterns observed:**
- All use minimal frontmatter (just `name` + `description`)
- Production skills use `references/` heavily for progressive disclosure
- Scripts are self-contained with clear error messages
- No skill exceeds 500 lines in SKILL.md

## Validation

The Agent Skills spec provides a validation tool:

```bash
# Install the reference library
# https://github.com/agentskills/agentskills/tree/main/skills-ref

# Validate a skill
skills-ref validate ./my-skill
```

Checks: valid frontmatter, naming conventions, directory structure compliance.
