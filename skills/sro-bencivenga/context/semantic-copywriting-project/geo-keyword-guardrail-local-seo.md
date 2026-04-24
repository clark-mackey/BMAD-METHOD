# Geo Keyword Guardrail: Procedure/Service Pages

## Core Principle

**A procedure page is about the procedure, not the location.**

Geographic relevance is established through site architecture, schema, internal links, and off-page signalsâ€”not keyword stuffing. Competitors spam location keywords because they don't know better. Cora reflects what competitors do, not what works best.

---

## Why Cora Overstates Geo Keywords

Cora showed CN Plastic Surgery has 13 "Philadelphia" mentions vs. 76 max on Page 1. This does NOT mean you need 63 more.

**What's actually happening:**
- Competitors use location spam as a lazy geo signal
- Cora measures correlation, not causation
- Google uses off-page data (GBP, citations, links, entity associations) to determine geo relevance
- Over-optimization with geo keywords can harm topical authority

---

## How We Establish Geo Relevance (Without Keyword Spam)

| Signal Type | Implementation |
|-------------|----------------|
| **Site Architecture** | Procedure pages link to Area Served pages; Area Served pages link back to procedures with geo-modified anchor text |
| **Schema** | `areaServed` property on procedure pages lists all service areas; Area Served pages use LocalBusiness schema |
| **Internal Links** | "Areas Served" sidebar/section with links to Bryn Mawr, Mainline, Philadelphia pages |
| **Images** | Geo data embedded in image EXIF metadata |
| **Off-Page** | GBP, citations, local links all reinforce geo association |
| **Google Business Profile** | Campaign-tagged link to homepage; GBP category matches services |

---

## Geo Keyword Limits for Procedure Pages

### Allowed (Natural Placement)
- **Title tag**: 1 mention (e.g., "Tummy Tuck Philadelphia" or "Tummy Tuck Bryn Mawr")
- **H1**: 0-1 mention (optionalâ€”procedure name can stand alone)
- **Body copy**: 2-4 mentions total, only where contextually natural
- **Areas Served section**: Unlimited (this is the designated geo zone)
- **Schema**: Full list in `areaServed` property

### Avoid
- Geo keywords in H2 subheadings (unless the section is specifically about location)
- Repeating "Philadelphia tummy tuck" or "[city] + [procedure]" pattern throughout body copy
- Stuffing neighborhoods/suburbs into body paragraphs

### FAQ Exception: PAA-Detected Geo Questions
If Google People Also Ask or Cora's Questions sheet shows location-specific questions (e.g., "What is the cost of a tummy tuck in Philadelphia?"), include those questions verbatim in the FAQ section. This ensures computed analysis shows nâ‰¥1 for geo-referenced FAQ questions rather than n=0.

**Example detected questions to include:**
- "What is the cost of a tummy tuck in Philadelphia and Bucks County?"
- "About tummy tuck surgery in Philadelphia?"
- "Considering abdominoplasty in Philadelphia?"

**Do not invent geo FAQ questions.** Only use those detected in PAA or competitor analysis.

---

## Correct Structure for Geo on Procedure Pages

```
[H1] Procedure Name (geo optional)

[Body content about procedure - minimal geo mentions]

[H2] Areas Served
We serve patients from [Area 1], [Area 2], and [Area 3]. 
[Link to Area Served: Bryn Mawr]
[Link to Area Served: Mainline Philadelphia]  
[Link to Area Served: Philadelphia]

[Schema: areaServed includes all locations]
```

**The geo section is a contained module.** It handles geographic association so the rest of the page can focus on topical authority.

---

## When to Override Cora's Geo Recommendations

**Ignore Cora's geo keyword gap if:**
- Site has proper Area Served page structure
- Schema includes `areaServed` property
- GBP is properly configured and linked
- Page already has an "Areas Served" internal link section

**Act on Cora's geo recommendation only if:**
- No Area Served pages exist
- Schema is missing `areaServed`
- Title tag lacks any geo modifier
- Page has zero internal links to location pages

---

## For CN Plastic Surgery Specifically

**Current state:** 13 "Philadelphia" mentions
**Cora recommendation:** Add 63 more
**Actual recommendation:** Keep at ~15-20 total

**Where geo should appear:**
1. Title tag: "Drain-Free Tummy Tuck Philadelphia | Dr. Claytor"
2. Schema `areaServed`: Philadelphia, Bryn Mawr, Mainline suburbs
3. Areas Served sidebar: Links to each geo page
4. 1-2 natural body mentions: "patients throughout the Philadelphia region"
5. Footer NAP: Bryn Mawr address

**Where geo should NOT appear:**
- Every H2 heading
- FAQ questions (unless asking about location)
- Repeated "[city] tummy tuck" phrases in body copy
- Case history descriptions (unless patient location is relevant)

---

## Summary

Let site architecture do the geo work. Procedure pages build **topical authority** (tummy tuck expertise). Area Served pages build **geographic authority** (Philadelphia service area). Internal links connect them. Schema tells machines the relationship.

**Cora measures what competitors do. We do what actually works.**
