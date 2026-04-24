# Plugin-Bundled Skill Conventions

Plugin skills (skills inside an installed plugin marketplace package) follow rules that differ from user-authored standalone skills in `~/.claude/skills/`. The grader must apply different criteria — what looks like an anti-pattern in a standalone skill is often correct in a plugin context.

---

## How to identify a plugin skill

Plugin skills live under `~/.claude/plugins/cache/<marketplace>/<plugin>/skills/<skill-name>/SKILL.md` (or a similar marketplace-specific path). When invoked, they appear as `plugin-name:skill-name` in the available-skills list (e.g., `superpowers:brainstorming`, `anthropic-skills:skill-creator`).

**If the skill identifier contains a colon, it's a plugin skill.** Apply this reference.

---

## Convention 1: Namespacing via plugin prefix

**Standalone:** `name: my-skill` → `/my-skill`
**Plugin:** `name: my-skill` in plugin `acme` → `/acme:my-skill`

The plugin prefix prevents naming collisions across marketplaces. A user can install `acme:deploy` and `widgets:deploy` simultaneously without conflict.

**Grading implication:** Don't penalize plugin skills for `name` values that match other plugins' skills. The namespace isolates them.

---

## Convention 2: Data persistence via `${CLAUDE_PLUGIN_DATA}`

Plugin skills must NOT write to their own directory — plugin updates wipe the install path and lose state.

**Correct:**
```bash
echo "$result" > "${CLAUDE_PLUGIN_DATA}/cache.json"
```

**Wrong:**
```bash
echo "$result" > "${CLAUDE_SKILL_DIR}/cache.json"  # Lost on plugin update
```

**Grading implication:** A plugin skill that writes to `${CLAUDE_SKILL_DIR}` scores 0 on Dimension 10 (Data Management). Standalone skills can use the project's `.claude/` dir or any persistent location they choose — the same rule doesn't apply.

---

## Convention 3: Bundled assets via `${CLAUDE_SKILL_DIR}`

Plugin skills frequently reference bundled scripts, templates, and reference docs. These are read-only and must use `${CLAUDE_SKILL_DIR}` rather than absolute paths or relative-to-cwd paths.

**Correct:**
```bash
python "${CLAUDE_SKILL_DIR}/scripts/process.py"
cat "${CLAUDE_SKILL_DIR}/templates/report.md"
```

**Wrong:**
```bash
python ./scripts/process.py            # Depends on cwd
python /home/user/.claude/.../process.py  # Hardcoded user path
```

**Grading implication:** Plugin skills get +0 (already correct) for using `${CLAUDE_SKILL_DIR}`; standalone skills score similarly when they use the same pattern, but standalone skills aren't required to — they can use simpler relative paths if SKILL.md documents the cwd assumption.

---

## Convention 4: Marketplace-managed updates

Plugin skills update via `/plugin update <name>`. Users don't edit them in place — local edits get overwritten on the next update.

**Grading implication:**
- Don't suggest "edit this file to fix" for a plugin skill — suggest "open a PR to the marketplace" instead.
- A plugin skill that hasn't updated in 6+ months may be stale (check `installed_plugins.json` for `gitCommitSha` vs marketplace HEAD).
- Don't penalize plugin skills for lacking inline customization hooks — that's not what they're for.

---

## Convention 5: Marketplace-shared resources

Plugins can bundle multiple skills that share a common `references/` or `scripts/` directory at the plugin root. A skill's SKILL.md may reference `${CLAUDE_PLUGIN_ROOT}/shared/...` (or similar marketplace-specific variable).

**Grading implication:** When auditing a plugin skill, check the parent plugin directory for shared resources before flagging "missing references." The skill may legitimately depend on plugin-level files.

---

## Convention 6: User customization via override skills

Users who want to modify a plugin skill should create a same-named local skill at `~/.claude/skills/<name>/`, not edit the plugin file. Higher-priority locations win (personal > project; plugin uses a separate namespace so doesn't conflict).

The `thariq-skill-grader-dev` skill in this folder is itself an example: it's a local fork of `anthropic-skills:thariq-skill-grader` for testing rubric changes before pushing upstream.

**Grading implication:** When you see a user's local skill that mirrors a plugin skill, it's likely an intentional override or dev fork — not a duplicate to flag.

---

## Convention 7: Different review-cadence expectations

Plugin skills are reviewed by the marketplace maintainer. Standalone skills are reviewed by the user only.

**Grading implication for Dimension 9 (Maintainability):**
- Plugin skills inherit the marketplace's quality bar — if `anthropic-skills:*` skills look polished, that's the bar.
- Standalone skills should have an owner signal (frontmatter author? CLAUDE.md reference? a "last reviewed" comment?) — plugin skills don't need this since the plugin manifest serves the same purpose.

---

## Special case: Bundled skills (built into Claude Code)

Built-in bundled skills like `/simplify`, `/debug`, `/loop`, `/claude-api` are neither plugin nor user — they ship with Claude Code itself. They appear in the skill list without a `:` prefix but cannot be edited locally.

**Grading implication:** Treat as exemplars (see [exemplars.md](exemplars.md)). They embody Anthropic's current best practices and update with Claude Code releases. Compare user-authored skills *against* these, but don't grade them — they're the standard.

---

## Quick reference: standalone vs plugin grading differences

| Concern | Standalone skill | Plugin skill |
|---------|------------------|--------------|
| Data path | Any persistent location OK | Must use `${CLAUDE_PLUGIN_DATA}` |
| Asset references | Relative paths OK if cwd documented | Must use `${CLAUDE_SKILL_DIR}` |
| Naming collisions | Real concern | Namespaced by plugin |
| Update mechanism | User edits in place | `/plugin update <name>` |
| Owner signal | Should be present (frontmatter, README) | Plugin manifest serves this |
| Fix recommendations | "Edit SKILL.md to..." | "Open PR to marketplace" |
| Local override | N/A | Create same-named local skill |
