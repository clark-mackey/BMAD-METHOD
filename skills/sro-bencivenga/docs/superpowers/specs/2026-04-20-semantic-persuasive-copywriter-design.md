# Semantic Persuasive Copywriter — Design Spec

**Date:** 2026-04-20
**Status:** Approved for implementation planning
**Owner:** Clark (CAKE Websites & More)
**Target distribution:** Claude Teams (chat, cowork, Claude Code)

---

## 1. Purpose

Build a Claude skill that produces persuasive, AI-retrievable copy by fusing Gary Bencivenga's direct-response methods with Semantic Retrieval Optimization (SRO). The skill replaces ad-hoc copywriting prompts and guarantees a consistent quality floor for CAKE agency work across medical, legal, and other YMYL verticals.

**Name:** `semantic-persuasive-copywriter`

**Description (SKILL.md frontmatter):** Writes persuasive, AI-retrievable copy using Gary Bencivenga's direct-response methods fused with Semantic Retrieval Optimization (SRO). Use whenever you need to rewrite an existing page, draft a new blog post, or write a client newsletter — especially for medical, legal, or other YMYL businesses. Auto-detects the job family from input and routes to the right workflow. Always runs a pre-delivery grading pass (skippable on explicit user override).

**`when_to_use` (SKILL.md frontmatter):** Trigger phrases: "improve this copy," "rewrite this page," "make this landing page better," "polish this draft," "write a blog post about [X]," "draft an article about [X]," "need a newsletter for [client]," "write the [month] newsletter," "update this service page," "SEO-optimize this," "make this more persuasive," "apply Bencivenga/SRO to this," "produce copy for [client]." Verticals: medical, dental, chiropractic, optometry, veterinary, plastic surgery, med spa, mental health, law firm, immigration law, family law, financial services, insurance — any YMYL or professional-services business. Do NOT use for: technical documentation, API reference material, internal runbooks, developer-facing content, code comments, release notes, internal team communications, or anything where Bencivenga's direct-response voice would be inappropriate. For those tasks, prefer a general-purpose writing approach or a dedicated technical-writing skill.

---

## 2. Users and contexts

**Primary users:** CAKE agency team members (marketers, account managers). They are not trained copywriters and do not know Bencivenga's methodology by name. The skill must guide them, not assume fluency.

**Environments (all three must work):**
- Claude Code — full tool access (Bash, scripts, MCP).
- Claude.ai chat (Teams plan) — text-first, may have GitHub MCP.
- Claude.ai cowork — text-first, may have Project files attached.

Scripts only execute in Claude Code. Chat/cowork fall back to asking the user to paste content.

---

## 3. Job families (intake modes)

Three routed modes:

1. **Rewrite** — URL or body-content dump of an existing page that needs improvement. Most common input.
2. **Blog post** — topic brief or keyword idea for a new post.
3. **Newsletter** — monthly digest for a specific client, 3–5 item multi-section format.

Mode detection happens in `SKILL.md` Step 2; if ambiguous, the skill asks one question.

Explicitly **out of scope** for v1: sales letters from scratch, ad copy, email sequences, service page mode (handled under Rewrite).

---

## 4. Key decisions (from brainstorming)

| Decision | Choice | Rationale |
|---|---|---|
| Reference curation strategy | Hybrid: distill the 3 most-loaded references; copy-rename others with teaching headers | Ships fast; avoids blind curation of archives before observing actual load patterns |
| Primary users | CAKE team (non-copywriters) | Skill must guide and teach, not assume fluency |
| Input forms | Rewrite (URL/body) + Blog brief + Newsletter topic | Three distinct intake flows |
| Vertical handling | Medical/legal-aware auto-detect, not locked | Handles 95% use case (CAKE clients) while staying portable |
| Intake behavior | Mode-dependent | Rewrite = just go; blog = smart intake; newsletter = guided |
| Grading | Auto-run + auto-revise, grade report always delivered | Protects deliverable quality with non-copywriter users; teaches team |
| Output | Copy + H1/H2/H3 + meta title + meta description + image alt | Schema/SEO strategy out of v1 scope |
| Client brand context | Reference client files from `cake-websites/cake-brand-guidelines`, graceful degradation | Matches existing infra; skill never fails on missing brand data |
| Geo/local SEO | Deferred to v2 | Existing guardrail file is Cora-entangled; needs rewrite |
| Scripts | Minimal: URL fetcher + brand-guide fetcher | Defer other scripts until real friction shows |
| Skill name | `semantic-persuasive-copywriter` | Methodology-first, portable, brand-neutral |
| Source location | In the workbench (`skills-workbench/sro-bencivenga/`) | Simple; promotion path to Teams TBD |
| Example files | Upgraded with teaching headers, mapped by purpose | Archives become first-class teaching artifacts |

---

## 5. Directory structure

```
sro-bencivenga/                                           # workbench folder
├── SKILL.md                                              # entry point
├── references/
│   ├── client-index.md                                   # NEW — client name → slug lookup, always loaded
│   ├── sro-principles.md                                 # NEW — always loaded
│   ├── bencivenga-methods.md                             # NEW — loaded on methodology-deep tasks
│   ├── compliance-medical-legal.md                       # NEW — auto-loaded on YMYL trigger
│   ├── mode-rewrite.md                                   # NEW — loaded in rewrite mode
│   ├── mode-blog-post.md                                 # NEW — loaded in blog mode
│   ├── mode-newsletter.md                                # NEW — loaded in newsletter mode
│   ├── grading-rubric.md                                 # NEW — always loaded at grading step
│   │
│   ├── example-complete-package.md                       # RENAMED from 01_Little_Black_Book.md
│   ├── example-subscription-continuity.md                # RENAMED from 02_Newsletter.md
│   ├── example-financial-headlines.md                    # RENAMED from 03_Get_Rich_Slowly.md
│   ├── example-expose-structure.md                       # RENAMED from 04_Lies_Lies_Lies.md
│   ├── example-expert-authority.md                       # RENAMED from 05_Charles_Givens.md
│   ├── example-institutional-trust.md                    # RENAMED from 06_Merrill_Lynch.md
│   ├── example-health-magalog.md                         # RENAMED from 07_Look_Younger_Now_MAGALOG.md
│   ├── example-outrage-urgency.md                        # RENAMED from 08_Hands_Off_Washington.md
│   ├── example-olive-oil-breakdown.md                    # RENAMED from olive_oil_breakdown
│   └── interview-bencivenga.md                           # RENAMED from complete_interview.md
│
├── scripts/
│   ├── fetch-url.py                                      # URL → cleaned markdown body (with SSRF guard)
│   ├── fetch-brand-guide.py                              # client slug → concatenated guideline files
│   ├── sync-client-index.py                              # regenerates references/client-index.md from brand repo README
│   └── requirements.txt                                  # httpx, trafilatura
│
├── tests/
│   ├── fixtures/                                         # realistic input cases for behavior tests
│   │   ├── rewrite-medical-thin-source.md
│   │   ├── rewrite-legal-rich-source.md
│   │   ├── blog-medical-brief.md
│   │   ├── blog-legal-topic-only.md
│   │   └── newsletter-plastic-surgery.md
│   └── scripts/
│       ├── test_fetch_url.py
│       └── test_fetch_brand_guide.py
│
├── docs/
│   └── superpowers/
│       └── specs/
│           └── 2026-04-20-semantic-persuasive-copywriter-design.md   # this file
│
└── context/                                              # unchanged — archival source material
    └── semantic-copywriting-project/                     # existing raw source files stay here
```

**Notes:**
- Source files in `context/` are the provenance; copies (with teaching headers) live in `references/` as `example-*.md`.
- `SKILL.md` stays under ~150 lines. Detailed methodology lives in `references/`.

---

## 6. SKILL.md shape

```markdown
---
name: semantic-persuasive-copywriter
description: [see section 1 above]
when_to_use: [see section 1 above]
---

# Semantic Persuasive Copywriter

## User-override protocol (standing)
The default flow always runs intake, mode routing, and pre-delivery grading. The user can
override in one request with any of these explicit instructions:
  - "skip grading" / "no grade report" / "just deliver the draft" → skip Step 6. Include
    `_Grading skipped by user request._` at the bottom of the output.
  - "use my rubric" / "here's my style guide" → prefer the user-supplied criteria over
    `grading-rubric.md` for that request.
  - "skip intake" / "just write it" → bypass mode-specific intake questions and produce
    on best-effort reading of the input.
These overrides apply to the current request only. Return to defaults on the next invocation.

## Step 1 — Load baseline
Always load references/sro-principles.md before any copy task.

## Step 2 — Classify intent, then route to a mode
Classify the task by INTENT, not by keyword matching. Answer three questions internally:
  1. Is the primary deliverable an improved version of an existing page? → mode-rewrite
  2. Is the primary deliverable a new long-form article or post? → mode-blog-post
  3. Is the primary deliverable a recurring digest of multiple items? → mode-newsletter
A URL in the input is NOT a reliable signal — rewrites, blog posts, and newsletters can all
reference URLs. Focus on what the team member is asking for as output.
If two classifications tie or none fit, ask ONE question: "Is this a page rewrite, a new blog
post, or a newsletter?" Load the matching references/mode-*.md.

## Step 3 — Check client index, then vertical compliance
First, consult references/client-index.md to identify the client. Match on practice name,
slug, city+specialty, or doctor/attorney name — do not guess.

If a client is identified:
  - Load their brand guide (Step 4).
  - Auto-load references/compliance-medical-legal.md — every client in cake-brand-guidelines
    is YMYL. No further check needed.

If no client is identified:
  - Check whether the subject entity falls under health, medical, legal, financial, mental
    health, or life-impacting services (entity class, not keyword list). If yes, load
    references/compliance-medical-legal.md.
  - The keyword list is a safety-net fallback, not the primary check.

## Step 4 — Brand context
For the client identified in Step 3:
  - Claude Code: run `python scripts/fetch-brand-guide.py <slug>`
  - Chat/cowork: check for GitHub MCP tool; if available, fetch the four guideline files
    from clients/<slug>/guidelines/ in cake-websites/cake-brand-guidelines. Otherwise,
    ask the user to paste personality-and-tone + target-audiences.
If the brand guide is missing, thin, or cannot be fetched → produce good-faith copy,
apply generic professional medical/legal tone, flag the gap in the grade report.

## Step 5 — Execute the mode
Follow the loaded mode file's intake → produce → grade → deliver workflow.

## Step 6 — Grade before delivery (always)
Load references/grading-rubric.md. Score the output using the adversarial enumeration
protocol defined in that file (mechanical listing, not Yes/No vibes-check). Auto-revise
any Required item scoring No.

**Iteration cap: revise once. If the second grade still has Required No items, deliver
with explicit gap notes. Never loop a third time.** Token budget and user wait time matter
more than a third polish pass that usually doesn't converge.

Deliver copy + grade report.

## The Persuasion Equation (always applies)
URGENT PROBLEM + UNIQUE PROMISE + UNQUESTIONABLE PROOF + USER-FRIENDLY PROPOSITION
= IRRESISTIBLE OFFER

## Writing Rules
DO: named entities, semantic triplets, query-aligned headings, one idea per section,
expert attribution, descriptive anchor text.
DON'T: orphan pronouns, generic headings, keyword stuffing without entity context,
multiple topics per section, anonymous content, amplify promise without proportional proof.

## Progressive disclosure — when to load example files
| Task / need | Load this | Why |
|---|---|---|
| Writing anti-aging / aesthetic / health copy | example-health-magalog.md | Richest fascination bullets + benefit-specificity patterns |
| Writing financial copy (rare at CAKE) | example-financial-headlines.md | Financial headline + proof patterns |
| Writing long-form sales letter arc | example-expose-structure.md | Expose/reveal structure, credibility arc |
| Writing institutional/trust-first brand copy | example-institutional-trust.md | Soft CTA, credential-heavy, trust-first |
| Writing expert/authority positioning | example-expert-authority.md | Seminar-style proof + authority framing |
| Writing outrage/advocacy/urgency (rare) | example-outrage-urgency.md | Emotion triggers, urgency framing |
| Writing subscription/continuity copy | example-subscription-continuity.md | Subscriber psychology, value-stacking |
| Writing complete direct-mail package | example-complete-package.md | OE + letter + BRC anatomy |
| Need annotated persuasion structure | example-olive-oil-breakdown.md | Line-by-line persuasion analysis |
| Need Bencivenga's direct voice on a technique | interview-bencivenga.md | Search for the specific technique |
| Need deep methodology grounding | bencivenga-methods.md | Full technique library (fascinations, headlines, proof, triggers) |

**Protocol:** Load only what the current task demands. Never load all files.

## Output format (v1)
Every deliverable includes:
  - H1 (headline)
  - H2/H3 structure + body copy
  - Meta title (50–60 chars)
  - Meta description (140–160 chars)
  - Image alt suggestions (one per recommended image slot)
  - Grade report (from grading-rubric.md)
```

---

## 7. Reference files — scope

### New files (8)

**`client-index.md`** — always loaded
- Generated from `cake-websites/cake-brand-guidelines` README.md by `scripts/sync-client-index.py`.
- Each row: slug | client name | tagline | city | specialty | aka-names (doctor name, practice nicknames).
- Goal: deterministic client identification from input phrases like "Dr. Sobel's site," "the Asheville plastic surgeon," "Berks."
- Refreshed quarterly or when a client is added. Last-synced date stamped at top.

**`sro-principles.md`** — always loaded

**Target length:** ~600–900 words (so it stays cheap to always-load). Keep examples terse.

**Required sections (in this order):**
1. *The Persuasion Equation anchored to semantic search* — one paragraph explaining why Bencivenga's equation and SRO share the same skeleton (named entities, specific outcomes, proof, clear next step).
2. *Entity clarity* — no orphan pronouns, name the subject every time the topic shifts, use consistent naming across the whole piece. One medical example, one legal example.
3. *Semantic triplets* — Subject → Predicate → Object. Rules (1) name the subject, (2) specific predicates, (3) anchor objects to named entities or measurable outcomes, (4) one triplet per core claim. Table of 4 rewrites: vague → retrievable, all medical/legal.
4. *Query-aligned headings* — headings lead with direct answers, match search-intent phrasing (PAA style). Do-list + Don't-list.
5. *Atomic chunks* — one idea per section, answer-ready first paragraph, passages extractable without surrounding context.
6. *E-E-A-T essentials* — named author with credentials, citations to named sources, update date, bio link. Non-negotiable on YMYL.
7. *Descriptive anchor text* — no "click here"; anchor names the destination entity + action.

**Sourcing:** distill from `context/semantic-copywriting-project/semantic-copywriting-guidelines.md`. Remove all Cora references (Cora isn't used by this skill). Re-cast every example to medical/legal. If the source file's guidance depends on Cora output, rewrite the guidance to stand on its own or drop it.

**`bencivenga-methods.md`** — loaded on methodology-deep tasks
- Persuasion Equation expanded (each element + execution guidance)
- Fascinations — 7 types with formulas + examples
- Headlines — 4 U's, 25-headline rule, proof-enhancement patterns
- Proof hierarchy — 7 layers, order of credibility
- Emotional triggers matrix — 7 triggers with lead-with guidance
- Bencivenga voice rules (admit a flaw, long copy when proof exists)
- Sourced from `bencivenga_copywriting_methods_report.md` + `bencivenga_copywriting_methods_analysis.md`, merged and deduped.

**`compliance-medical-legal.md`** — auto-loaded on YMYL trigger
- FTC testimonial rules (typical-results disclosures, endorsement guide)
- Medical: no unsupported claims, before/after photo context, treatment vs. cure language, FDA-regulated terms
- Legal: state bar advertising rules, no guaranteed outcomes, past-results disclaimers
- E-E-A-T hardening: named reviewer with credentials, license numbers, update date
- Author bio block requirements
- Red-flag phrases to avoid
- Loads client's `guidelines/compliance.md` from brand guide if present (client rules supplement baseline)

**Sourcing + review requirements (non-optional):**
- Every rule MUST have an inline citation: FTC Endorsement Guide §, FDA regulation, state bar code section, or equivalent. Rules without citations are not to be written.
- Date-stamp each rule with its source-document date. Mark any rule whose source is older than 3 years for review.
- Header callout at the top of the file: *"This file encodes regulatory guardrails for copywriting, not legal advice. Consult counsel for jurisdiction-specific questions."*
- Before v1 release, file is reviewed by an attorney with healthcare/legal-advertising experience. Review date and reviewer noted in the file. No ship without this.

**`mode-rewrite.md`** — rewrite mode workflow
- Intake: URL or pasted content → fetch/clean → analyze
- Diagnostic pass: proof gaps, weak headline, missing CTA, entity ambiguity
- Pre-write mini-checklist: audience, primary CTA, proof assets, unique promise
- Produce: headline → lead → body → proof blocks → CTA → P.S.
- Output template: H1 / H2 / H3 / meta title / meta description / image alt / grade report
- Keep-vs-cut guidance (respect existing proof, strip weak copy)

**`mode-blog-post.md`** — blog mode workflow
- Smart intake: scan brief for audience, angle, keyword, CTA goal; ask ≤3 targeted gaps
- Outline: search-intent-matched H2/H3 structure, answer-first paragraphs
- Produce: semantic triplets, named-entity proof, CTA integration
- Output template: H1 / H2 / H3 / meta / image alt / suggested slug / grade report

**`mode-newsletter.md`** — newsletter mode workflow
- Guided intake (5 prompts batched): client slug, audience, month/theme, topic ideas (skill can propose), offers/events
- Always fetches brand guide for voice consistency
- Per-item structure: curiosity headline → 100–250 word body → single-action CTA
- Plus: subject line, preview text, header, footer
- Output template: subject + preview + per-item blocks + footer + grade report
- **Grading split:**
  - Per-item reduced rubric (7 checks per item, from grading-rubric.md §"per-item subset"): urgent problem, specific promise, proof element, clear CTA, named entity, triplet structure, query-aligned headline.
  - Whole-newsletter rubric (remaining 11 checks applied once): subject-line strength, preview-text complement, author block, citations present, update date, anchor-text quality, voice consistency with brand guide, compliance pass, cross-item topic coherence, schema-ready structure, final grade.
  - Auto-revise only items that fail. Do not re-grade the whole newsletter after a per-item fix.

**`grading-rubric.md`** — always loaded at grading step
- 18-point checklist (Bencivenga 7 + SRO 7 + E-E-A-T 4) with one-sentence prompts per item
- Scoring formula + grade scale (A/B/C/D/F)
- Auto-revise protocol: for each No on a Required item, prescribed fix actions
- Iteration cap: revise once, then deliver with gap notes if still failing
- Per-item subset (7 checks) for newsletter-mode per-item grading (see mode-newsletter.md)
- Worked example: weak draft → graded → revised → re-graded
- Grade report output template

**Adversarial enumeration protocol (required structure):**

Each check is a mechanical listing instruction, not a Yes/No self-assessment. The grader must enumerate concrete items from the draft, then count. Examples:

- *Entity clarity:* "List every pronoun and its antecedent. Count pronouns with ambiguous antecedents. Pass if zero."
- *Semantic triplets:* "List every core claim in the draft. For each, identify subject, predicate, object. Count claims missing a named subject. Pass if zero."
- *Proof visibility:* "List every proof element in the draft. Note its location (headline, lead, body, footer). Pass if ≥1 in headline or lead."
- *Query-aligned headings:* "Copy every heading. For each, state the likely search query it answers. Count headings that answer no query directly. Pass if zero."

Enumeration is harder to inflate than Yes/No vibes. The grade report includes the enumerations — both as evidence and as teaching artifacts for the team.

### Example files (10) — renamed with teaching headers

Each file gets a ~100-word teaching header:
- **What this exemplifies** (3–5 techniques shown)
- **When to load this** (task triggers)
- **Key moments** (section pointers into the raw text below)

Raw content beneath the header is unchanged from source. Mapping:

| New filename | Source file | Core value |
|---|---|---|
| `example-complete-package.md` | `01_Little_Black_Book.md` | Multi-element package anatomy |
| `example-subscription-continuity.md` | `02_Newsletter.md` | Continuity, value-stacking, subscriber psychology |
| `example-financial-headlines.md` | `03_Get_Rich_Slowly.md` | Financial headline + proof patterns |
| `example-expose-structure.md` | `04_Lies_Lies_Lies.md` | Long-form expose/reveal arc, credibility-building |
| `example-expert-authority.md` | `05_Charles_Givens.md` | Expert positioning, seminar-style proof |
| `example-institutional-trust.md` | `06_Merrill_Lynch.md` | Trust-first, credential-heavy, soft CTA |
| `example-health-magalog.md` | `07_Look_Younger_Now_MAGALOG.md` | Richest fascinations, benefit-specificity patterns |
| `example-outrage-urgency.md` | `08_Hands_Off_Washington.md` | Outrage triggers, urgency framing |
| `example-olive-oil-breakdown.md` | `Gary_Bencivenga___Famous__Olive_Oil__Sales_Letter_Breakdown__11_100_.md` | Line-by-line persuasion analysis |
| `interview-bencivenga.md` | `complete_interview.md` | Direct voice on techniques; search for the specific principle |

---

## 8. Scripts

Both Python, Code-only, invoked via Bash.

### `scripts/fetch-url.py`

**Purpose:** pull and clean URL content for rewrite mode.

**Invocation:** `python scripts/fetch-url.py <url>`

**Dependencies:** `httpx`, `trafilatura` (pinned in `scripts/requirements.txt`)

**Behavior:**
- Validates URL before fetching: scheme MUST be `http` or `https`; resolves hostname; rejects if resolved IP is in a private/reserved range (RFC1918 10/8, 172.16/12, 192.168/16, loopback 127/8, link-local 169.254/16, IPv6 equivalents). This is an SSRF guard.
- Fetches URL via httpx with 15s timeout, `follow_redirects=True` but redirect-target MUST pass the same IP-range check (no redirect to internal).
- Uses trafilatura to extract main content (strips nav, footer, sidebars, ads, cookie banners).
- Preserves H1/H2/H3, paragraphs, lists.
- User-agent: `CAKE-SemanticCopywriter/1.0`.
- Output: markdown to stdout.

**Output format:**
```
# <extracted H1>

**URL:** <canonical URL>
**Fetched:** <ISO timestamp>

## <H2>
<body paragraphs>
...
```

**Failure modes:** paywall / 403 / timeout → exit 1, stderr message. Skill prompts user to paste body content instead. Blocked private IP → exit 2, stderr `Refused: target resolves to a private/reserved IP range`.

### `scripts/fetch-brand-guide.py`

**Purpose:** pull per-client brand guidelines from `cake-websites/cake-brand-guidelines`.

**Invocation:** `python scripts/fetch-brand-guide.py <client-slug>`

**Dependencies:** `httpx` (no PyGithub — keep it lean).

**Token requirements (non-optional):**
- Reads `GITHUB_TOKEN` from env.
- Token MUST be a fine-grained PAT scoped to `cake-websites/cake-brand-guidelines` with `Contents: Read-only` permission. Broad-scope classic PATs are rejected.
- Script performs a pre-flight scope check: call `GET /user` + `GET /repos/cake-websites/cake-brand-guidelines` with the token. If scope check fails (missing read, or token has write/admin on other repos detectable via `/user/installations`), exit 3 with `Refused: token has more access than required. Use a fine-grained PAT scoped to cake-brand-guidelines, Contents: Read-only.`
- Setup instructions in `scripts/README.md` include the exact PAT creation steps.

**Behavior:**
- Fetches via GitHub API (not clone):
  - `clients/<slug>/README.md`
  - `clients/<slug>/guidelines/personality-and-tone.md`
  - `clients/<slug>/guidelines/target-audiences.md`
  - `clients/<slug>/guidelines/foundations.md`
  - `clients/<slug>/guidelines/compliance.md` (optional — skip if 404)
- Concatenates with section headers labeling each source file.
- Output: markdown to stdout.

**Output format:**
```
# Brand Guide: <client-name>

**Slug:** <slug>
**Fetched:** <ISO timestamp>

## Overview (README.md)
<content>

## Personality & Tone
<content>

## Target Audiences
<content>

## Foundations
<content>

## Compliance
<content — if file exists, else "No client-specific compliance file. Apply baseline medical-legal guardrails.">
```

**Failure modes:**
- Slug not found → exit 1, stderr `Client slug '<x>' not found in repo`. Skill asks user for correct slug or proceeds without brand context (flagging in grade report).
- Missing `GITHUB_TOKEN` → exit 1, stderr with setup instructions. Skill falls back to asking user to paste.
- Network failure → exit 1, stderr with error. Skill falls back to asking user to paste.

**Defensive read:** If `guidelines/` schema changes upstream, script lists the directory and loads whatever `.md` files it finds (robust to additions, safe against renames).

### `scripts/sync-client-index.py`

**Purpose:** regenerate `references/client-index.md` from the brand repo README.

**Invocation:** `python scripts/sync-client-index.py`

**Dependencies:** `httpx`.

**Behavior:**
- Reads `GITHUB_TOKEN` with same scope requirements as `fetch-brand-guide.py`.
- Fetches `cake-websites/cake-brand-guidelines/README.md`.
- Parses the client list (markdown bullet list with `[Client Name](clients/<slug>/README.md) — tagline` pattern).
- For each client, optionally fetches `clients/<slug>/guidelines/foundations.md` to extract city and specialty (cache in memory; one pass only).
- Writes `references/client-index.md` with last-synced timestamp header.
- Idempotent — safe to re-run.

**Cadence:**
- **Primary (recommended for v1.1):** GitHub Action in `cake-websites/cake-brand-guidelines` that runs on push to `main` and, when `README.md` changes, opens a PR in this skill's repo updating `references/client-index.md`. Auto-sync with human review gate.
- **Fallback for v1:** Manual quarterly run by Clark or whoever is sync'd to the brand repo. Document the last-sync date at the top of `client-index.md`.
- **Staleness protection:** `client-index.md` header includes `**Last synced:** <date>` and a note: *"If a recent client is not in this index, run sync-client-index.py (Code) or fetch cake-brand-guidelines/README.md directly."* This makes the lag visible to team members using chat/cowork, where the file is baked in at Teams distribution time and can't self-refresh until the next upload.

### Cross-environment behavior

`SKILL.md` Step 4 documents this explicitly:
- **Claude Code:** run scripts.
- **Chat (Teams):** if GitHub MCP tool is available, use it to fetch the same four files. Otherwise ask user to paste.
- **Cowork:** same as chat. Also check whether brand guide is already attached as a Project file.

---

## 9. Mode flows (end-to-end)

### Rewrite

1. User: "rewrite this page for [client]'s [service] — [URL]"
2. Load `sro-principles.md` + `mode-rewrite.md`.
3. Source fetch:
   - Claude Code: run `python scripts/fetch-url.py <url>`.
   - Chat / cowork: use available web-fetch tooling (WebFetch or equivalent) if permitted; otherwise ask the user to paste the page body.
4. Vertical check: detect medical/legal terms → load `compliance-medical-legal.md`.
5. Brand check: detect client reference → fetch brand guide. If ambiguous, ask for slug.
6. Diagnostic: inline note on source weaknesses (proof gaps, headline, CTA, entity clarity).
7. Produce: rewrite applying persuasion equation + brand voice + compliance.
8. Grade: load `grading-rubric.md`, score, auto-revise any Required No, re-grade.
9. Deliver: rewritten copy (H1/H2/H3 + meta + image alt) + grade report + brief diagnostic summary.

### Blog post

1. User: "write a blog post about [topic] for [client]"
2. Load `sro-principles.md` + `mode-blog-post.md`.
3. Brief scan: audience / angle / keyword / CTA goal. If ≥3 missing → ask up to 3 targeted questions, batched.
4. Vertical + brand checks (same as rewrite).
5. Outline: H2/H3 structure aligned to search intent.
6. Archive load (conditional): if health/financial/legal advice-heavy AND user or routing signal requests pattern reference, load a relevant `example-*.md`. Default: don't load archives.
7. Produce: answer-first paragraphs, semantic triplets, named-entity proof, CTA.
8. Grade → auto-revise → re-grade.
9. Deliver: body + H1/H2/H3 + meta + image alt + suggested slug + grade report.

### Newsletter

1. User: "write the [month] newsletter for [client]"
2. Load `sro-principles.md` + `mode-newsletter.md`.
3. Guided intake — 5 prompts batched in one question: client slug (confirm), audience, month/theme, topic ideas (or "generate 5 options"), offers/events.
4. Brand load: always fetch brand guide (voice consistency critical).
5. Vertical check: compliance almost always triggers.
6. Topic generation if requested.
7. Produce per item: curiosity headline → 100–250 word body → single-action CTA. Plus: subject, preview, header, footer.
8. Grade per item + as a whole → auto-revise → re-grade.
9. Deliver: subject + preview + per-item blocks + footer + grade report.

### Cross-mode behaviors

- **Graceful degradation:** missing brand guide → generic professional medical/legal tone; flag in grade report.
- **Compliance overrides style:** conflicts resolve to compliance; note in grade report.
- **Ambiguity handling:** undetectable mode → one question asked.

---

## 10. Testing and success criteria

### Script unit tests (`pytest`, Code-only)

- `fetch-url.py`: valid URL extracts main content; paywall exits 1; malformed URL exits 1; **private IP exits 2 (SSRF guard)**; **redirect to private IP exits 2**.
- `fetch-brand-guide.py`: valid slug returns concatenated guide; invalid slug exits 1 with helpful stderr; missing token exits 1 with setup instructions; missing optional `compliance.md` handled without error; **broad-scope token exits 3 (scope guard)**.
- `sync-client-index.py`: parses the README format correctly; produces deterministic output given fixed input; exits 1 on missing token or network failure.

### Fixture-based skill behavior tests

`tests/fixtures/` folder with realistic input cases:
- `rewrite-medical-thin-source.md`
- `rewrite-legal-rich-source.md`
- `blog-medical-brief.md`
- `blog-legal-topic-only.md`
- `newsletter-plastic-surgery.md`

Each fixture has: input + **assertable output properties** (not just a gold-standard example). Properties are concrete and checkable, e.g.:
- `grade_total >= 80`
- `h1_contains_named_entity: true`
- `proof_elements_in_headline_or_lead: >= 1`
- `cta_count: 1`
- `compliance_loaded: true` (when fixture is medical/legal)
- `brand_voice_flag: <expected>`

A lightweight harness (`tests/skill/run_fixtures.py`) posts each fixture to the Claude API with the skill loaded, parses the grade report block, and asserts properties. Failures print the fixture name, property, expected vs. actual. Run before each release; CI optional.

Manual cross-environment parity check (chat + cowork) remains a release gate — automate later.

### Grade-rubric self-consistency check

Feed the skill a known-weak draft. Verify the grade report correctly identifies weak items and the auto-revise pass addresses them.

### Cross-environment smoke test

Same input in Claude Code + chat + cowork; verify output quality parity and graceful degradation when scripts / MCP are unavailable.

### v1 ship gate

- [ ] All 8 new reference files written, reviewed, committed (including `client-index.md`).
- [ ] `compliance-medical-legal.md` reviewed by attorney with healthcare/legal-advertising experience. Reviewer and date noted in file.
- [ ] 10 example files renamed, teaching headers written.
- [ ] `SKILL.md` reads cleanly end-to-end; routing works on all three job families via intent classification (not keyword match).
- [ ] All three scripts pass unit tests (including SSRF + token-scope guards).
- [ ] All 5 fixtures pass **every** assertable property — not just grade ≥80%.
- [ ] Graceful degradation verified on each failure mode: missing brand guide, SSRF-blocked URL, missing token, 404 from GitHub API, unclassifiable input.
- [ ] Three real CAKE client deliverables (one rewrite, one blog, one newsletter) produced end-to-end in at least two environments (Claude Code + chat). Each signed off against: grade ≥85%, compliance rules applied, brand voice matched, no unresolved gap notes.
- [ ] Copyright question (see §13) resolved before Teams upload. Without a decision, ship only to Code (private use) and delay chat/cowork distribution.

---

## 11. Out of scope (v1)

- Schema markup output (JSON-LD).
- SEO strategy metadata (target keywords, semantic clusters, internal link maps).
- Local / geo SEO handling (existing guardrail file is Cora-entangled; needs rewrite for v2).
- Brand-voice companion skill / client-file auto-loading infrastructure beyond the lookup script.
- Full curation of the 10 example files into technique-first distillations.
- Additional modes: email sequences, ad copy, service pages as a distinct mode, landing pages for new offers.
- Additional scripts: grade calculator, semantic triplet linter, fascination generator.
- Telemetry, usage logging, learning loop.

---

## 12. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Reference file bloat worsens token load instead of improving it | Target SKILL.md ~150 lines; measure actual load per mode during testing; trim aggressively if needed |
| Medical/legal compliance rules go stale (FTC / FDA / state bar updates) | Date-stamp `compliance-medical-legal.md`; quarterly review cadence |
| Brand guide repo schema changes break the fetch script | Script reads defensively — lists `guidelines/` directory and loads whatever `.md` files it finds |
| Chat users lack GitHub MCP access → brand lookup always falls back to paste | Document explicitly in SKILL.md Step 4; confirm MCP availability with CAKE Teams admin before release |
| Team members bypass intake questions, produce thin-input work | Grading auto-revise pass catches thin output; grade report flags missing elements for team review |

---

## 13. Open questions

1. **Source location promotion path:** workbench → dedicated GitHub repo → Teams upload. Decide before release.
2. **Where does brand-guide script's `GITHUB_TOKEN` live for chat/cowork?** Likely not applicable (MCP or paste handles those). Code users provision their own fine-grained PAT per §8.
3. **Newsletter subject-line rules** — any CAKE-specific length / style constraints? Needs one-off check with team during `mode-newsletter.md` authoring.
4. **Bencivenga source-file copyright basis for Teams distribution.** The 10 example files are Gary Bencivenga's copyrighted work. Packaging raw promo text into a Teams-distributed skill accessible to many users may exceed fair use. Options:
   - (a) Reduce raw excerpts to short fair-use quotations with heavy commentary.
   - (b) Obtain license from the rights holder.
   - (c) Restrict access to Code-only private use (narrower audience, stronger fair-use argument).
   - (d) Replace raw text with pure pattern distillations (no source text at all).
   Blocks Teams chat/cowork distribution until decided. Code-only use can proceed as private study.
5. **Observability / usage signal (v1.1).** Ship v1 without telemetry; before v1.1, decide minimum viable observability: lightweight local JSONL log (Code), copy-paste telemetry block the team pastes into a shared doc, or nothing beyond anecdotal feedback. Decision drives whether reference-file edits can be data-informed.

---

## 14. Implementation order (preview — not the plan)

Rough sequence for the implementation plan:

1. **Rename workbench folder** `sro-bencivenga/` → `semantic-persuasive-copywriter/` to match skill name. Update this spec's path references.
2. Directory scaffolding + `SKILL.md` skeleton with frontmatter.
3. Write `sync-client-index.py`; generate initial `references/client-index.md`.
4. Write `sro-principles.md` per §7 authoring spec (baseline — unblocks all other work).
5. Write `grading-rubric.md` with adversarial enumeration protocol per §7.
6. Write `compliance-medical-legal.md` with inline citations; send to attorney for review.
7. Write the three `mode-*.md` files (newsletter includes the per-item / whole-letter grading split).
8. Write `bencivenga-methods.md` (distillation work).
9. Rename + write teaching headers on the 10 example files.
10. Write `fetch-url.py` (with SSRF guard) + `fetch-brand-guide.py` (with token-scope guard) + unit tests.
11. Write `SKILL.md` body fully with intent-classification routing per §6.
12. Create fixtures with assertable properties, build `run_fixtures.py` harness, run behavior tests, iterate.
13. Three real-client deliverables across modes and environments per v1 ship gate.
14. Resolve §13 open questions (copyright, observability, promotion path) before Teams distribution.
15. v1 ready.

The detailed plan will be produced in the next step via the `writing-plans` skill.
