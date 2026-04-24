# Thariq Shihipar — "Lessons from Building Claude Code: How We Use Skills"

**Source:** https://github.com/shanraisshan/claude-code-best-practice/blob/main/tips/claude-thariq-tips-17-mar-26.md
**Original X thread:** https://x.com/trq212/status/2033949937936085378
**Author:** Thariq Shihipar (Anthropic, Claude Code team)
**Date:** 2026-03-17
**Mirrored:** 2026-04-19 — verbatim excerpts below.

---

## Context (verbatim)

> Skills have become one of the most used extension points in Claude Code. They're flexible, easy to make, and simple to distribute. But this flexibility also makes it hard to know what works best. Thariq shares the lessons learned from using skills extensively at Anthropic with hundreds of them in active use.

> A common misconception is that skills are "just markdown files", but the most interesting part is that they're **folders** that can include scripts, assets, data, etc. — things the agent can discover, explore, and manipulate. Skills also have a wide variety of configuration options including registering dynamic hooks.

---

## The 9 Skill Categories (canonical taxonomy)

> After cataloging all of their skills, the team noticed they cluster into 9 recurring categories. The best skills fit cleanly into one; the more confusing ones straddle several.

| # | Category | Description | Example skills |
|---|----------|-------------|----------------|
| 1 | **Library & API Reference** | Skills that explain how to correctly use a library, CLI, or SDKs. Often include reference code snippets and gotchas. | billing-lib, internal-platform-cli, frontend-design |
| 2 | **Product Verification** | Skills that describe how to test or verify code is working. Often paired with Playwright, tmux, etc. "It can be worth having an engineer spend a week just making your verification skills excellent." | signup-flow-driver, checkout-verifier, tmux-cli-driver |
| 3 | **Data Fetching & Analysis** | Connect to data and monitoring stacks. Include credentials, dashboard IDs, common workflow instructions. | funnel-query, cohort-compare, grafana |
| 4 | **Business Process & Team Automation** | Automate repetitive workflows into one command. Saving previous results in log files helps the model stay consistent. | standup-post, create-`<ticket-system>`-ticket, weekly-recap |
| 5 | **Code Scaffolding & Templates** | Generate framework boilerplate. Combine with composable scripts. Useful when scaffolding has natural language requirements. | new-`<framework>`-workflow, new-migration, create-app |
| 6 | **Code Quality & Review** | Enforce code quality and help review code. Can include deterministic scripts. May run automatically as hooks or in GitHub Actions. | adversarial-review, code-style, testing-practices |
| 7 | **CI/CD & Deployment** | Help fetch, push, and deploy code. May reference other skills to collect data. | babysit-pr, deploy-`<service>`, cherry-pick-prod |
| 8 | **Runbooks** | Take a symptom (Slack thread, alert, error signature), walk through multi-tool investigation, produce a structured report. | `<service>`-debugging, oncall-runner, log-correlator |
| 9 | **Infrastructure Operations** | Routine maintenance and operational procedures — some destructive, benefit from guardrails. | `<resource>`-orphans, dependency-management, cost-investigation |

**Grading note:** This is the canonical 9-category taxonomy. Cross-reference `references/categories.md` against this list — if there's drift, this article is authoritative.

---

## The 9 Tips for Making Skills (verbatim)

### Tip 1: Don't State the Obvious

> Claude Code knows a lot about your codebase, and Claude knows a lot about coding, including many default opinions. If you're publishing a skill that is primarily about knowledge, try to focus on information that pushes Claude out of its normal way of thinking. The frontend design skill is a great example — it was built by iterating with customers on improving Claude's design taste, avoiding classic patterns like the Inter font and purple gradients.

**Grading mapping:** Dimension 2 (Context Efficiency), Dimension 8 (Redundancy Check).

### Tip 2: Build a Gotchas Section

> The highest-signal content in any skill is the Gotchas section. These sections should be built up from common failure points that Claude runs into when using your skill. Ideally, you will update your skill over time to capture these gotchas.

**Grading mapping:** Dimension 3 (Gotchas & Edge Cases). Direct authority for the rubric.

### Tip 3: Use the File System & Progressive Disclosure

> A skill is a folder, not just a markdown file. You should think of the entire file system as a form of context engineering and progressive disclosure. Tell Claude what files are in your skill, and it will read them at appropriate times. The simplest form is to point to other markdown files — e.g., split detailed function signatures and usage examples into `references/api.md`. You can have folders of references, scripts, examples, etc.

**Grading mapping:** Dimension 5 (File Structure & Progressive Disclosure).

### Tip 4: Avoid Railroading Claude

> Claude will generally try to stick to your instructions, and because skills are so reusable you'll want to be careful of being too specific. Give Claude the information it needs, but give it the flexibility to adapt to the situation. Instead of prescriptive step-by-step instructions, give the goal and constraints.

**Grading mapping:** Dimension 4 (Flexibility vs. Rigidity Balance). Direct authority — "give the goal and constraints" is the precise principle.

### Tip 5: Think through the Setup

> Some skills may need to be set up with context from the user. A good pattern is to store this setup information in a `config.json` file in the skill directory. If the config is not set up, the agent can then ask the user for information. You can instruct Claude to use the AskUserQuestion tool for structured, multiple choice questions.

**Grading mapping:** Dimension 10 (Data & State Management). Note the canonical pattern: `config.json` in skill directory + `AskUserQuestion` for missing config.

### Tip 6: The Description Field Is For the Model

> When Claude Code starts a session, it builds a listing of every available skill with its description. This listing is what Claude scans to decide "is there a skill for this request?" Which means the description field is not a summary — it's a description of **when to trigger** this skill. Write it for the model.

**Grading mapping:** Dimension 1 (Description / Trigger Quality). This is the canonical statement of "description = trigger spec, not summary." Cite directly when grading Dimension 1.

### Tip 7: Memory & Storing Data

> Some skills can include a form of memory by storing data within them. You could store data in anything as simple as an append-only text log file or JSON files, or as complicated as a SQLite database. Data stored in the skill directory may be deleted when you upgrade the skill, so use `${CLAUDE_PLUGIN_DATA}` as a stable folder per plugin to store data in.

**Grading mapping:** Dimension 10 (Data & State Management). Direct authority for the `${CLAUDE_PLUGIN_DATA}` rule.

### Tip 8: Store Scripts & Generate Code

> One of the most powerful tools you can give Claude is code. Giving Claude scripts and libraries lets Claude spend its turns on composition, deciding what to do next rather than reconstructing boilerplate. Claude can then generate scripts on the fly to compose this functionality for more advanced analysis.

**Grading mapping:** Dimension 5 (File Structure) — bundled scripts/ folder is a quality signal. Dimension 4 (Flexibility) — composition over reconstruction.

### Tip 9: On Demand Hooks

> Skills can include hooks that are only activated when the skill is called, and last for the duration of the session. Use this for more opinionated hooks that you don't want to run all the time but are extremely useful sometimes.

**Examples cited:**
- `/careful` — blocks `rm -rf`, `DROP TABLE`, force-push, `kubectl delete` via PreToolUse matcher on Bash
- `/freeze` — blocks any Edit/Write that's not in a specific directory

**Grading mapping:** Dimension 7 (Verification & Safety). Cross-reference `references/lifecycle-and-hooks.md` for hooks-vs-instructions decisions.

---

## Distribution & Marketplace Guidance (verbatim)

### Distributing Skills

> Two ways to share skills with your team:
> - **Check into your repo** (under `.claude/skills`) — best for smaller teams working across relatively few repos
> - **Make a plugin** and have a Claude Code Plugin marketplace where users can upload and install plugins
>
> Every skill that is checked in also adds a little bit to the context of the model. As you scale, an internal plugin marketplace allows you to distribute skills and let your team decide which ones to install.

### Managing a Marketplace

> There isn't a centralized team that decides which skills go into a marketplace. Instead, try and find the most useful skills organically. Upload to a sandbox folder in GitHub and point people to it in Slack or other forums. Once a skill has gotten traction (which is up to the skill owner to decide), they can put in a PR to move it into the marketplace. Curation before release is important to avoid redundant skills.

### Composing Skills

> You may want to have skills that depend on each other. For example, a file upload skill that uploads a file, and a CSV generation skill that makes a CSV and uploads it. This sort of dependency management is not natively built into marketplaces or skills yet, but you can just reference other skills by name, and the model will invoke them if they are installed.

### Measuring Skills

> To understand how a skill is doing, use a PreToolUse hook that lets you log skill usage within the company. This means you can find skills that are popular or are undertriggering compared to expectations.

---

## Conclusion (verbatim)

> Skills are incredibly powerful, flexible tools for agents, but it's still early and we're all figuring out how to use them best. Think of this more as a grab bag of useful tips that we've seen work than a definitive guide. The best way to understand skills is to get started, experiment, and see what works for you. Most of ours began as a few lines and a single gotcha, and got better because people kept adding to them as Claude hit new edge cases.

---

## How to use this reference when grading

1. **Cite Thariq directly** by tip number (e.g., "Tip 6: description is a trigger spec, not a summary") — gives team members a recognizable authority.
2. **Use the 9 categories** as the canonical taxonomy. If `references/categories.md` drifts from this list, this article wins.
3. **The verbatim quotes are stable claims** — they won't be re-paraphrased on each grading run. Anchor controversial calls in direct quotes.
4. **Note the humility in the conclusion** — the article frames itself as "a grab bag of useful tips," not a strict standard. When a skill *intentionally* breaks a guideline for a documented reason, that's not automatically a deduction.
