---
name: book-summary
description: Generates structured book summaries with deep research, anti-hallucination checks, an independent verification pass, tactics tables, and timeless logic explanations. Use when the user asks to summarize a book, create a book summary, asks "what are the main ideas from X", "key takeaways from X", "I just finished X by Y", "help me process this book", or "book notes on X". Do not use for generic topic summaries, article summaries, or non-book sources.
---

<!-- Variant: CC (parallel stateless subagents). Sibling: linear edition for Claude.ai chat/Cowork. Last updated 2026-04-21 (v2: typology gates, provenance, verifier pass, skeleton mode) -->

# Book Summary Skill

You generate structured book summaries with deep research and anti-hallucination checks. Output is intended to become a permanent second-brain entry, so verified provenance matters more than narrative polish.

Use this when the user asks to summarize a book, create a book summary, or asks "what are the main ideas from X", "key takeaways from X", "I just finished X by Y", "help me process this book", or "book notes on X". Do not use this for generic topic summaries, article summaries, or non-book sources.

**Isolation model — one drafter per summary, plus one verifier.** Every summary is drafted in a dispatched subagent with a fresh context window (eliminates cross-book contamination). After drafting, a second stateless verifier subagent independently fact-checks the draft against its own stated sources (eliminates self-validation). The orchestrator never drafts or verifies itself — it only dispatches, gates, and relays.

---

## Step 0: Dispatch (Orchestrator Only)

**Recursion guard:** If the task instructions you received begin with the literal token `[book-summary subagent]` or `[book-summary verifier]`, you are a dispatched subagent — skip this step and proceed to the step indicated in your instructions. Do not dispatch further subagents.

Otherwise, you are the orchestrator. Do the following:

### 0.1 Identify books and extract user-supplied material

1. Identify each distinct book the user wants summarized. A single book still dispatches one drafter.
2. **Scan the current conversation for material the user provided per book** — pasted text, excerpts, ToC listings, notes, linked PDFs read earlier. Attach to the correct book. Do not default to `"none"` if material is present. Material for Book A must not leak into Book B.

### 0.2 Dispatch drafters in parallel

For each book, call the `Agent` tool with `subagent_type=general-purpose`. Send ALL drafter dispatches in a single message.

Use this prompt verbatim, substituting bracketed fields:

> `[book-summary subagent]`
>
> You are running the book-summary skill as a DRAFTER for a single book. Invoke the `book-summary` skill and follow Steps 1 through 5 exactly. Do NOT re-enter Step 0.
>
> Book: `[TITLE]`
> Author: `[AUTHOR]`
> User-supplied material: `[paste the full text, notes, or excerpts the user provided for this book — or "none" if nothing was supplied]`
>
> Return ONE of three output formats as markdown (no JSON wrapping):
>
> - **Full draft**: YAML frontmatter + Sections 1–10 + the mandatory `## Sources & Provenance` section (Section 11).
> - **Hard Stop**: the verbatim Hard Stop response from Step 2.
> - **Skeleton**: the Skeleton Mode template from Step 2 (for when bibliographic data verifies but content bar fails).
>
> Do not return anything else. No prose, no meta-commentary, no framing.

### 0.3 Gate each drafter return

When drafters return, classify each:

- **Hard Stop** — first line begins with `"I cannot find enough verified information about"`. Relay verbatim with an orchestrator header `### [Title] — Hard Stop`. **Skip verification.** Move on.
- **Skeleton** — contains a heading matching `# [Title] — Partial Summary (Skeleton Mode)`. Relay verbatim with an orchestrator header noting it is a partial artifact. **Skip verification.** Move on.
- **Full draft** — everything else. Proceed to Step 0.4.

### 0.4 Dispatch verifiers in parallel

For each FULL DRAFT, call `Agent` with `subagent_type=general-purpose`. Send all verifier dispatches in a single message.

Use this prompt verbatim:

> `[book-summary verifier]`
>
> You are running the book-summary skill as a VERIFIER. Invoke the `book-summary` skill and follow the Verification Protocol exactly. You are NOT drafting — you are independently fact-checking the attached draft.
>
> Draft to verify:
>
> ```markdown
> [paste entire drafter output here, including the ## Sources & Provenance section]
> ```
>
> Return a structured verdict as markdown:
>
> ```markdown
> ## Verification Verdict: PASS | FAIL
>
> ### Checked Claims
> | Claim | Type | Result | Note |
> |-------|------|--------|------|
> | ... | quote | verified | found verbatim on [URL] |
> | ... | number | unverified | drafter's only source was a single blog |
> | ... | attribution | contradicted | author actually coined this in [earlier book] |
>
> ### Recommended Fixes
> - [specific actionable fix per failed claim]
> ```

### 0.5 Relay based on verdict

- **PASS**: relay the drafter's markdown verbatim under `### [Title]`. Do not append the verifier verdict.
- **FAIL**: relay the drafter's markdown verbatim under `### [Title]`, then append the verifier's `## Verification Verdict` block directly below it. Do NOT edit the draft yourself — the user decides whether to accept, correct, or re-run. Never silently drop failed claims.

### 0.6 Cross-summary lint (final step before returning to user)

Before returning to the user, scan all relayed summaries together for suspicious cross-book patterns:

- Identical or near-identical "coined term" names appearing in multiple books
- Tactics tables all of suspiciously identical length
- Abstracts with clone-like structure/length
- Quotes that appear in more than one book's Key Quotes section

If any matches, append a `## Cross-Summary Notes` block flagging them. If clean, say nothing — silence is the signal.

---

## Step 1: Mode Detection (Drafter)

Identify which mode applies before taking any other action:

- **Text Provided**: The user has supplied the full book text or comprehensive notes. Skip to Step 3 (Failure Modes).
- **Partial Text**: The user has supplied highlights, excerpts, or notes that don't cover the full book. Run Step 2 (research) to fill structural gaps (table of contents, missing frameworks), then use the provided text as the primary source for detail and quotes.
- **Research Only**: The user has supplied only a title and author. You MUST complete Step 2 in full. Do not substitute internal training knowledge for verified live research — training data is stale and frequently blends details across books.

---

## Step 2: Research Protocol (Research Only and Partial Text Modes)

### 2.1 Classify the book's typology

Assess typology before applying the content bar:

- **framework-coining** — the author proposes named models, coined terms, proprietary frameworks (e.g., Tufte, Godin, Kahneman). Content bar: ≥5 named frameworks/terms attributable to the author.
- **synthesis-handbook** — the author organizes existing practice into a reference work; coined neologisms are rare by design (e.g., most PR/publicity manuals, craft handbooks, field guides). Content bar: ≥5 discrete procedural sections or methods verifiable as the book's own organizing structure, not coined terms.
- **narrative-historical** — biography, business case-study, or history. Content bar: verified central thesis, verified arc/sequence of events, and ≥3 key subjects/sources verifiable.
- **reference** — dictionary, encyclopedia, cookbook, style guide used as lookup. Not a fit for this skill; Hard Stop and suggest the user request a targeted extraction instead of a summary.

State the typology you assigned in the YAML `typology` field.

### 2.2 Required search queries

Run all of the following before assessing confidence:

1. `"[Book Title]" "[Author]" table of contents`
2. `"[Book Title]" "[Author]" [candidate term or framework name]` — substitute as you discover terms
3. `"[Book Title]" "[Author]" site:amazon.com`
4. `"[Book Title]" "[Author]" site:books.google.com`
5. `"[Book Title]" "[Author]" site:worldcat.org OR site:loc.gov` — library catalogs often expose full ToC fields
6. `"[Author]" interview OR talk "[book title or framework]"` — author explaining their own work in their own words
7. `"[Book Title]" "[Author]" ISBN`

### 2.3 Tiered source quality

**Tier 1 — Primary (preferred for grounding every specific claim):**

- The author's own material: the book's own text (excerpts, Look Inside, Google Books page-image), the author's website, recorded interviews or lectures where the author names their framework
- Publisher's official book page
- Google Books preview (page-image or extracted text)
- Amazon Look Inside
- Library catalog records with full ToC (WorldCat, Library of Congress, major university libraries)

**Tier 2 — Corroborating (acceptable only when paired with a second independent Tier 2, or backing a Tier 1):**

- Academic papers citing specific chapters/concepts with page numbers
- Long-form reviews in major publications (NYT, WSJ, The Atlantic, The New Yorker, London Review of Books, Harvard Business Review, The Guardian long reads) that quote specific passages
- University syllabi or course notes reproducing the book's ToC

**Unacceptable as evidence for any specific claim:**

- AI-generated summaries from any source
- Blinkist, getAbstract, Shortform, or similar micro-summary services
- SparkNotes, CliffsNotes
- Anonymous or individual reviewer blogs without direct citations to primary text
- Your own training knowledge unless corroborated by a live Tier 1 or Tier 2 source from this session

Grounding rule: every specific factual claim in the summary must trace either to a Tier 1 source, or to two independent Tier 2 sources. A single Tier 2 source is not enough.

### 2.4 Minimum Bar — check all before drafting

You must confirm ALL of the following or you hit a Hard Stop or Skeleton Mode (see 2.5):

- [ ] Verified table of contents from a Tier 1 source (or two independent Tier 2 sources)
- [ ] Publication year, publisher, and edition confirmed
- [ ] ISBN-13 verified from a Tier 1 source (ISBN-10 acceptable only if ISBN-13 unavailable)
- [ ] Typology-specific content bar passed (see 2.1)
- [ ] At least one Tier 1 source accepted

### 2.5 Hard Stop vs. Skeleton Mode

If the Minimum Bar fails, choose:

- **Hard Stop** — the book cannot be responsibly summarized at all: ISBN unverifiable, ambiguous title-author match (different book by same author, or different author with same title), or every source is unacceptable. Use the Hard Stop response template below.
- **Skeleton Mode** — bibliographic data verifies but content bar fails (e.g., ToC partial, framework count below threshold). Return the Skeleton template so the user gets something actionable and knows exactly what to supply to unblock.

**Hard Stop response — use verbatim:**

> "I cannot find enough verified information about *[Title]* by *[Author]* to write an accurate summary without risk of hallucination. I was unable to verify: [list exactly what's missing]. To proceed, please provide: the book text, a table of contents, or the specific framework names you want covered."

**Skeleton Mode template — use this structure:**

```markdown
---
title: "[verified title]"
author: [verified author]
isbn: "[verified ISBN or null]"
canonical_edition: "[verified edition or null]"
typology: [assigned typology]
category: [inferred category]
tags: []
key_concepts: []
related_books: []
---

# [Title] — Partial Summary (Skeleton Mode)

## Verified Bibliographic Data
- Publisher, year, edition, ISBN, page count — everything confirmed

## Verified Table of Contents (Partial: X of Y chapters)
- Chapter titles confirmed from Tier 1, numbered as in source

## Why This Book Matters (only Tier 1/2-grounded context)
Brief, bounded to what you can cite.

## Requires User Input to Complete
- [ ] Full ToC (currently verified: X of Y chapters)
- [ ] Named frameworks/coined terms (need: [list what's missing])
- [ ] Verified quotes (none confirmed so far)
- [ ] Any specific content the user wants covered

## Sources & Provenance
[Same format as Section 11 of a full draft]
```

---

## Step 3: Review Failure Modes

Before writing, review these and actively avoid them:

**1. Generic Advice Trap** — Summarizing the book's topic rather than the author's specific methodology. Test: if you could swap the author's name for another in the same genre and the sentence still reads true, rewrite it.

**2. Hallucinated Chapter Titles, Quotes, ISBNs, Essay Numbers** — NEVER guess. Only include what you have verified. Do not commit to `Essay #50` if your source is a single blog.

**3. Thin Tactics Tables** — At least 8 specific author-coined (or for synthesis-handbook typology, author-organized) tactics, with real mechanisms in the Timeless Logic column.

**4. Incomplete Framework Extraction** — Omitting the author's most famous contributions (e.g., Tufte without data-ink ratio). Your research must identify the complete named taxonomy.

**5. YAML / Formatting Violations** — Follow the template exactly. All mandatory fields. No extras.

**6. Artifacts in Narrative Sections** — Narrative sections 1–10 must be clean. Provenance lives in Section 11 only.

**7. Secondary Source Trap** — Blinkist, AI summaries, SparkNotes are not authoritative. If they are your only sources, you have hit a Hard Stop.

**8. Confidence Theater** — Finding just enough to feel confident and blending real content with plausible fabrications. Apply the Minimum Bar explicitly.

**9. Cross-Book Attribution Drift** — Attributing a concept to the wrong book by the same author (e.g., "Smallest Viable Audience" was introduced by Godin in *This Is Marketing* (2018), not *The Practice*). If a concept was introduced in an earlier work by the same author, say so with a parenthetical.

**10. False Precision** — Committing to specific numbers (219 essays, 6 principles, Essay #50) when the source is a single non-Tier-1 blog. State ranges ("~220 essays") or drop the number unless Tier 1-verified.

---

## Step 4: Draft the Summary

Use this exact structure.

### 4.1 YAML frontmatter

All fields mandatory. No extra fields.

```yaml
---
title: "Full Book Title: Including Subtitle"
author: First Last
isbn: "978-X-XXXXX-XXX-X"
canonical_edition: "2nd ed., 2001, Publisher Name"
typology: framework-coining
category: Genre, Topic, Subject
tags: [tag1, tag2, tag3, hyphenated-tag]
key_concepts: [Author Coined Term 1, Proprietary Framework 2, Named Model 3]
related_books: ["slug-format-title-1", "slug-format-title-2"]
---
```

- `isbn`: ISBN-13 of the edition you verified, quoted. ISBN-10 fallback only. Never fabricate.
- `canonical_edition`: human-readable string naming the edition (e.g., `"2nd ed., 2001, Graphics Press"`).
- `typology`: one of `framework-coining` | `synthesis-handbook` | `narrative-historical`.

### 4.2 Section structure

**1. Abstract** — One paragraph: core argument, methodology, significance, thesis, context.

**2. The Big Idea** — The central concept or thesis in one tight paragraph.

**3. Why It Matters** — Historical context, impact, enduring relevance.

**4. Key Findings & Frameworks** — Structured around the author's actual chapter titles and named concepts. Cover the 5–10 most significant. For each: explanation in the author's terms, author's own examples, underlying mechanism. If a concept was introduced in an earlier work by the same author, flag parenthetically.

**5. Table of Tactics** — At least 8 rows:

| Tactic Name | How It Is Intended to Be Used | The Timeless Logic Behind It | Practical Application |
|-------------|-------------------------------|------------------------------|----------------------|

**6. Pitfalls & Warnings** — Specific failure modes the author identifies.

**7. Actionable Takeaways** — Concrete steps in the author's terminology, not generic advice.

**8. One-Line Verdict** — A single sentence capturing the book's ultimate value.

**9. Key Quotes** — **Every quote pinned to a retrievable source inline, in this exact format:**

> "Quote text." — *Source: [page or chapter reference if from the book, or URL if from author's own site / verified interview]*

If you cannot pin a quote to a specific retrievable source, **drop it**. No unpinned quotes. An unverifiable quote is worse than no quote for a permanent record.

**10. Further Reading & Resources** — Related books using `[[slug-format-title]]` wiki-links.

**11. Sources & Provenance** — Mandatory trailing section. Terse. Scannable. Not narrative:

```markdown
## Sources & Provenance

**Primary sources consulted:**
- [URL] — used for: ToC / ISBN / framework X / quote Y / etc.
- [URL] — used for: ...

**Claims → source mapping:**
- ToC: [URL]
- ISBN: [URL]
- Coined term "[X]" attribution: [URL]
- Quote "[first 5 words...]": [URL]
- [any other specific factual claim]: [URL]

**Residual uncertainty:**
- [claims made with less than Tier-1 grounding, flagged honestly — leave blank if none]
```

### 4.3 Critical requirements

- **Specificity:** Use only the author's own terms. Apply the name-swap test.
- **Timeless Logic:** For every tactic, explain the underlying psychology or market dynamic.
- **Quote discipline:** Every quote pinned to a retrievable source, or dropped.
- **Numeric discipline:** No specific counts without Tier-1 backing. Use ranges or drop.
- **Attribution discipline:** If a concept was introduced earlier by the same author, say so parenthetically in Section 4.

---

## Verification Protocol (Verifier Subagent Only)

If your task instructions begin with `[book-summary verifier]`, follow this protocol instead of Steps 1–5.

Your job: **independently** fact-check the draft against its own stated sources. Do not trust the drafter. Use your web tools.

For each specific factual claim in the draft:

1. **Quotes** — Fetch each URL in the drafter's `## Sources & Provenance` block. Confirm the quote appears verbatim on that page (or in the quoted source). Flag if not found or paraphrased.
2. **Numeric claims** — Re-verify every specific number (essay count, chapter count, page count, lie-factor threshold, publication year, number-of-principles). Flag unverified or contradicted.
3. **Coined-term attributions** — For each term in `key_concepts`, fetch the author's own material or a Tier 1 source. Confirm the term is used by the author in THIS book, not a later or earlier one.
4. **Chapter titles** — Cross-check every chapter title referenced in Section 4 against the drafter's claimed Tier 1 ToC source. Flag any title not grounded in Tier 1.
5. **ISBN** — Independently re-fetch from a Tier 1 source. Flag any mismatch.

Return the `## Verification Verdict` markdown block specified in Step 0.4.

- `verdict: PASS` if every claim verified or only trivially uncertain.
- `verdict: FAIL` if any quote, ISBN, or numeric claim is unverified/contradicted. Attribution or title issues of moderate severity also trigger FAIL.

Recommended fixes must be specific and actionable. Examples: "drop quote #3, unverified on cited URL", "soften 'Essay #50' to 'a mid-book essay titled…' — source is a single blog", "re-verify ISBN from Tier 1 — drafter used a retailer listing only".

---

## Step 5: Final Validation (Drafter Self-Check)

Before returning your draft to the orchestrator, verify:

1. YAML frontmatter matches the template — all mandatory fields present (including `canonical_edition` and `typology`), no extra fields.
2. `isbn` is verified ISBN-13 (or ISBN-10 fallback), quoted, not fabricated.
3. Every chapter reference in Section 4 matches the verified ToC.
4. Every quote in Section 9 is pinned inline with a retrievable source.
5. No specific numeric claim (essay count, principle count, etc.) appears without Tier-1 backing in Section 11.
6. Tactics table has at least 8 rows.
7. Sections 1–10 are narrative and contain no research notes, meta-commentary, or conversational filler.
8. Section 11 (`## Sources & Provenance`) is present, complete, and terse.
9. Output is raw markdown as specified in Step 0.2 — not wrapped in JSON, not prefaced with commentary.

Fix any failures before returning.
