---
name: book-summary
description: Generates structured book summaries with deep research, anti-hallucination checks, tactics tables, and timeless logic explanations. Use when the user asks to summarize a book, create a book summary, asks "what are the main ideas from X", "key takeaways from X", "I just finished X by Y", "help me process this book", or "book notes on X". Do not use for generic topic summaries, article summaries, or non-book sources.
---

<!-- Variant: CC (parallel stateless subagents). Sibling: linear edition for Claude.ai chat/Cowork. Last updated 2026-04-21 -->

# Book Summary Skill

You generate structured book summaries with deep research and anti-hallucination checks. When the user asks to summarize a book, create a book summary, or asks "what are the main ideas from X", "key takeaways from X", "I just finished X by Y", "help me process this book", or "book notes on X" — follow this process exactly.

Do not use this for generic topic summaries, article summaries, or non-book sources.

**Isolation model — one subagent per summary.** Every summary is produced in a dispatched subagent with a fresh context window. This eliminates cross-book contamination (misattributed frameworks, hybrid concepts) when the user requests multiple books. The orchestrator never drafts a summary itself — it only dispatches and relays.

---

## Step 0: Dispatch (Orchestrator Only)

**Recursion guard:** If the task instructions you received begin with the literal token `[book-summary subagent]`, you ARE the dispatched subagent — skip this step entirely and proceed to Step 1. Do not dispatch further subagents.

Otherwise, you are the orchestrator. Do the following:

1. Identify each distinct book the user wants summarized. A single book still dispatches one subagent.
2. For each book, call the `Agent` tool with `subagent_type=general-purpose`. Send ALL dispatches in a single message so they run in parallel.
3. Use this dispatch prompt verbatim, substituting the bracketed fields. Do NOT classify mode yourself — the subagent handles that in Step 1.

   > `[book-summary subagent]`
   >
   > You are running the book-summary skill for a single book. Invoke the `book-summary` skill and follow Steps 1 through 5 exactly. Do NOT re-enter Step 0.
   >
   > Book: `[TITLE]`
   > Author: `[AUTHOR]`
   > User-supplied material: `[paste the full text, notes, or excerpts the user provided for this book — or "none" if the user gave only a title/author]`
   >
   > Return the complete summary as markdown (YAML frontmatter + all 10 sections). If you hit a Hard Stop, return the exact Hard Stop response verbatim — do not fabricate.

4. When subagents return, relay each summary to the user as-is. Do not edit, merge, re-order, or add commentary to the summaries themselves. A brief orchestrator header like `### [Title]` between summaries is acceptable.

---

## Step 1: Mode Detection

Identify which mode applies before taking any other action:

- **Text Provided**: The user has supplied the full book text or comprehensive notes. Skip to Step 3 (Failure Modes).
- **Partial Text**: The user has supplied highlights, excerpts, or notes that don't cover the full book. Run Step 2 (research) to fill structural gaps (table of contents, missing frameworks), then use the provided text as the primary source for detail and quotes.
- **Research Only**: The user has supplied only a title and author. You MUST complete Step 2 in full. Do not substitute internal training knowledge for verified live research — training data is stale and frequently blends details across books.

---

## Step 2: Research Protocol (Research Only and Partial Text Modes)

### Required Search Queries

Run all of the following before assessing confidence:

1. `"[Book Title]" "[Author]" table of contents` — find actual chapter names, not paraphrases
2. `"[Book Title]" "[Author]" [coined term or framework name]` — substitute terms you discover as you go
3. `"[Book Title]" "[Author]" site:amazon.com` — Amazon Look Inside frequently shows real ToC listings
4. `"[Book Title]" "[Author]" site:books.google.com` — Google Books previews often show opening pages and structure
5. `"[Author]" interview OR talk methodology "[genre]"` — find the author explaining their own framework in their own words
6. `"[Book Title]" "[Author]" ISBN` — verify the ISBN of the edition you are summarizing

### Acceptable Sources

- Publisher's official book page
- Amazon product page (Look Inside, table of contents section)
- Google Books preview
- Author's own website, blog, or recorded interviews where they explain their framework
- Academic papers that cite specific chapters or concepts from the book
- Long-form reviews in major publications (NYT, WSJ, The Atlantic, etc.) that quote specific passages

### Unacceptable Sources (as sole evidence)

Do not use these to fill gaps — they frequently compress, reorder, or invent framework names:

- AI-generated summaries from any source
- Blinkist, getAbstract, or similar micro-summary services
- SparkNotes or CliffsNotes
- Anonymous blog posts without direct citations to the text
- Your own training knowledge, if not corroborated by a live source from this session

### Minimum Bar — Check All Before Proceeding

You must confirm ALL of the following or you have hit a Hard Stop:

- [ ] Verified table of contents — at least partial, with real chapter names (not inferred)
- [ ] At least 5 author-coined terms, named frameworks, or proprietary models identified
- [ ] Publication year and publisher confirmed
- [ ] ISBN verified from an acceptable source (ISBN-13 preferred; ISBN-10 acceptable only if ISBN-13 is unavailable)
- [ ] At least one acceptable source found (not a micro-summary service)

### Hard Stop

Stop and report to the user if ANY of the following are true:

- You cannot find the actual table of contents — only secondary descriptions of it
- Fewer than 5 distinct author-coined terms are verifiable from live sources
- Every source returned is a secondary summary (Blinkist, AI blog, anonymous post) with no primary source
- You cannot verify an ISBN from an acceptable source — never fabricate one
- Search results return a different book by the same author, or a different author with the same title — ask the user to confirm the exact edition (publisher, year, ISBN) before proceeding

**Hard Stop response — use this exactly:**

> "I cannot find enough verified information about *[Title]* by *[Author]* to write an accurate summary without risk of hallucination. I was unable to verify: [list exactly what's missing]. To proceed, please provide: the book text, a table of contents, or the specific framework names you want covered."

Do not write a partial summary. Do not hedge and write anyway.

---

## Step 3: Review Failure Modes

Before writing, review these common AI book summary failures and actively avoid them:

**1. The "Generic Advice" Trap** — Summarizing the book's topic rather than the author's specific methodology. Test: if you could swap the author's name for another in the same genre and the sentence still reads true, it is too generic. Rewrite it.

**2. Hallucinated Chapter Titles, Quotes, and ISBNs** — Inventing plausible-sounding chapter names, attributing quotes to the wrong author, or guessing at an ISBN. NEVER guess. Only include what you have verified.

**3. Thin Tactics Tables** — Creating a table with 4-5 generic rows like "Use Active Voice." The table MUST have at least 8 specific, author-coined tactics with real psychological/behavioral mechanisms in the Timeless Logic column.

**4. Incomplete Framework Extraction** — Listing 3-5 concepts and stopping, omitting the author's most famous contributions (e.g., summarizing Tufte without the Data-Ink Ratio). Your research must identify the complete taxonomy of named concepts.

**5. YAML and Formatting Violations** — Omitting a mandatory field (e.g., `isbn`) or adding unapproved YAML fields (language, rating, etc.). Follow the template exactly. No missing fields, no extra fields.

**6. Artifacts and Meta-Commentary** — Leaving research notes, conversational filler, or instructions in the output. The final document must be clean.

**7. The Secondary Source Trap** — Treating Blinkist, AI summaries, or SparkNotes as authoritative. They frequently compress, reorder, or invent framework names. If these are your only sources, you have hit a Hard Stop.

**8. Confidence Theater** — Finding just enough surface info to feel like you can proceed, then blending real content with plausible fabrications. Apply the Minimum Bar checklist explicitly. Partial confidence is not confidence.

---

## Step 4: Draft the Summary

Use this exact structure:

### YAML Frontmatter

Every summary MUST begin with this exact YAML block. All fields below are mandatory. Do NOT add any other fields.

```yaml
---
title: "Full Book Title: Including Subtitle"
author: First Last
isbn: "978-X-XXXXX-XXX-X"
category: Genre, Topic, Subject
tags: [tag1, tag2, tag3, hyphenated-tag]
key_concepts: [Author Coined Term 1, Proprietary Framework 2, Named Model 3]
related_books: ["slug-format-title-1", "slug-format-title-2"]
---
```

The `isbn` field is mandatory. Use the ISBN-13 of the edition you verified during research (prefer the print edition's ISBN-13; fall back to ISBN-10 only if ISBN-13 is unavailable). Always quote the value so hyphenated ISBNs aren't misparsed. If you cannot verify an ISBN from an acceptable source, that is a Hard Stop — do not fabricate one.

### Section Structure

**1. Abstract** — One-paragraph overview of the book's core argument, methodology, and significance. Must include the author's primary thesis and the context of the work.

**2. The Big Idea** — The central concept or thesis. What is the one thing the author wants the reader to take away?

**3. Why It Matters** — Historical context, impact, significance. How did it change the field? Why is it still relevant?

**4. Key Findings & Frameworks** — Structured around the author's actual chapter titles, named concepts, and proprietary frameworks. Cover the 5-10 most significant concepts or chapters. For books with more, group related chapters under the author's framework headings. For each:
- Detailed explanation using the author's specific terminology
- Specific examples provided by the author
- The underlying mechanism or principle

**5. Table of Tactics** — At least 8 specific, author-coined tactics:

| Tactic Name | How It Is Intended to Be Used | The Timeless Logic Behind It | Practical Application |
|-------------|-------------------------------|------------------------------|----------------------|
| [Author's Term] | [Specific application] | [Psychological/behavioral mechanism] | [Concrete, modern example] |

**6. Pitfalls & Warnings** — Specific failure modes identified by the author, using the author's terminology.

**7. Actionable Takeaways** — Concrete steps using author-specific instructions, not generic advice.

**8. One-Line Verdict** — A single powerful sentence summarizing the book's ultimate value.

**9. Key Quotes** — Only verified quotes. If you cannot verify a quote, do not include it.

**10. Further Reading & Resources** — Related books using `[[slug-format-title]]` wiki-links.

### Critical Requirements

- **Specificity:** Use only the author's coined terms and named frameworks. Apply the name-swap test.
- **Timeless Logic:** For every tactic, explain the underlying psychology or market dynamic that makes it work across eras and platforms.
- **Quotes:** Only include quotes you can verify against a specific source. Never fabricate.

---

## Step 5: Final Validation

Before delivering, verify:
1. YAML frontmatter exactly matches the template — all mandatory fields present (including `isbn`), no extra fields.
2. The `isbn` value is a verified ISBN-13 (or ISBN-10 fallback), quoted, not fabricated.
3. Filename (if saving) is the full title including subtitle in lowercase slug format.
4. Every chapter reference matches the verified table of contents.
5. Output contains no research notes, artifacts, or meta-commentary.
6. Tactics table has at least 8 rows.

Fix any failures before delivering.
