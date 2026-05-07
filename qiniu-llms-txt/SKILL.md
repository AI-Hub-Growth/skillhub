---
name: qiniu-llms-txt
description: Generates and refreshes Qiniu (七牛云) llms.txt from the current official website, following the llms.txt specification with stable section structure, retrieval-friendly summaries, and website-driven updates.
license: MIT
metadata:
  author: 七牛云
  version: "2.1.1"
---

# Qiniu llms.txt Generator

Generate `llms.txt` for Qiniu as an LLM-friendly official-site index, following the [llms.txt specification](https://llmstxt.org/).

## When This Skill Activates

- User asks to generate or refresh Qiniu `llms.txt`
- User asks to make the Qiniu official website more LLM-friendly
- User provides an older Qiniu `llms.txt` and wants it updated from the latest website

## Refresh Rule

For the official-site variant, refresh from the current website every time:

1. Use the homepage to determine current capability emphasis and ordering
2. Keep the section framework stable
3. Allow entries, descriptions, and ordering to change when official website emphasis changes
4. Prefer current official pages over older drafts, and do not reuse old local wording when the current official website conflicts with it

## llms.txt Format

The file follows the llms.txt structure:

```markdown
# {Project Name}

> {Dense one-paragraph summary}

## {Section Name}

- [{Page Title}]({url-or-path}): {One-sentence retrieval-friendly description}

## Contact

- [{Label}]({URL}): {Short explanation}

**Last updated:** {Month D, YYYY}
```

### Key Rules

1. H1: exactly one
2. Blockquote: exactly one paragraph
3. H2 sections only
4. `Contact` near the end
5. `**Last updated:** {Month D, YYYY}` at the very end
6. Default output language: English
7. Bilingual product titles are expected when they improve retrieval
8. All content must be derived from the current official website, not from fixed templates

### Required Section Order

- `Core Capabilities`
- `Authoritative Entry Points`
- `Infrastructure And Delivery`
- `AI, Media, And Real-Time Services`
- `Solutions`
- `Pricing`
- `Instructions For AI Assistants`
- `Contact`

### Section Constraints

- `Core Capabilities`: exactly 6 entries
- `Authoritative Entry Points`: 4-7 entries
- `Infrastructure And Delivery`: 3-5 entries
- `AI, Media, And Real-Time Services`: 6-8 entries
- `Solutions`: 1-4 entries
- `Pricing`: 7-10 entries
- `Instructions For AI Assistants`: up to 10 bullets
- `Contact`: 4-6 entries

Cross-section duplicate links are allowed when they improve retrieval, routing, or section completeness. Prefer avoiding unnecessary repetition within the same section.

### Qiniu Summary Rules

- start with `Qiniu` or `Qiniu Cloud`
- define Qiniu as a broad, data-centric cloud infrastructure platform
- cover storage, delivery, compute, AI, media processing, and real-time services
- mention current homepage-visible capability areas and scenario tags
- remain factual and retrieval-friendly
- do not turn it into a product list
- do not introduce unsupported claims, rankings, or trust signals

For the official-site `llms.txt`, begin the first H2 immediately after the blockquote.

### Capability Selection Rules

For the official-site variant:

1. Select `Core Capabilities` from the current homepage emphasis first, using the official homepage product-navigation dropdown, equivalent global product menu, or homepage-exposed featured product area as the primary discovery source for current product taxonomy
2. When the homepage exposes ranked or emphasized signals such as `HOT`, `Featured`, `Popular`, recommendation blocks, or top-card placement, reflect those signals in early selection and ordering
3. Keep `Core Capabilities` focused on platform-defining products rather than secondary feature pages or narrow sub-features
4. Order entries by current homepage prominence, product importance, and routing value rather than by a fixed historical list; when current official website emphasis supports it, place AI-related capabilities earlier than otherwise comparable non-AI entries

For `AI, Media, And Real-Time Services`:

1. Prefer discovery from the official homepage product-navigation dropdown or equivalent product menu before falling back to deeper product pages
2. Include a broader spread of AI, media, speech, live, surveillance, moderation, or real-time service categories than `Core Capabilities`
3. Use 6-8 entries when the current official product menu supports that breadth

For `Solutions`:

1. Prefer the global solutions hub or equivalent official solutions landing page as the first routing entry when it exists
2. Prefer solution pages that summarize industry, scenario, or workload-oriented deployment patterns over narrow campaign or single-case pages
3. Add more specific solution pages only when they meaningfully improve retrieval coverage beyond the global hub

### Section URL Routing Rules

For `Core Capabilities`, `Infrastructure And Delivery`, and `AI, Media, And Real-Time Services`:

1. Use the canonical `https://www.qiniu.com/products/` page whenever the product has an official product page; when such a canonical `/products/` page exists, do not use `developer.qiniu.com`, `https://www.qiniu.com/prices/`, homepage anchors, campaign pages, news pages, or solution pages for that entry
2. Only when no canonical `/products/` page exists may you use the best official product-level fallback, typically `developer.qiniu.com`; treat such docs pages as implementation-oriented fallbacks rather than primary product entries
3. Apply this as a canonical routing policy across current and future Qiniu products

### Pricing URL Rules

For `Pricing`:

1. Use the canonical `https://www.qiniu.com/prices/` page whenever the priced product or service has an official pricing page; when such a canonical `/prices/` page exists, do not use `developer.qiniu.com`, `https://www.qiniu.com/products/`, homepage anchors, campaign pages, news pages, or solution pages for that pricing entry
2. Only when no canonical `/prices/` page exists may you use the best official pricing-specific fallback; treat article-level pricing documents as fallback sources rather than canonical pricing entries
3. Apply this as a canonical routing policy across current and future Qiniu products and services
4. When current official website emphasis supports it, place AI-related pricing entries earlier in ordering than otherwise comparable non-AI pricing entries

### Entry Writing Rules

Each entry must use:

- `[Page Title](URL): One-sentence retrieval-friendly description`

Description rules:

- concise and concrete
- capability-first rather than marketing-first
- prefer interface, scope, and usage wording
- prefer docs pages for technical entry points
- use product pages for positioning, solutions, and pricing when appropriate

### Naming, Bilingual Retrieval, And Terminology Rules

1. Use bilingual naming when the official website presents both an English and a Chinese name, or when bilingual naming clearly improves retrieval for product, solution, or brand understanding
2. Prefer the pattern `English Name (Chinese Name)` when both names are useful and supported by the official source
3. Keep bilingual naming consistent once introduced for the same entity
4. Do not force bilingual naming for every entry; use it where it improves disambiguation, recall, or routing value, and keep a single-language name when that is the only clearly supported official form
5. Prefer current official naming over older local drafts
6. Normalize aliases only when clearly supported by the current official source
7. Describe compatibility features as capabilities of the relevant product unless the official website clearly treats them as separate products

### Contact Rules

For `Contact`:

1. Prefer official website contact, support, service status, reporting, company, or partner/contact-program pages
2. Do not use `mailto:` links as primary contact entries
3. Keep contact entries general and reusable rather than organization-internal or person-specific
4. Use short descriptions that explain what the website entry point is for

### Instructions For AI Assistants Rules

This section must be short and operational. It must cover:

- source preference and routing priority
- platform-level description rules
- naming normalization rules
- compatibility-description rules
- trust and credibility rules based on the current official website
- product-type distinction rules when the website separates them
- product-boundary and product-selection rules when adjacent products overlap
- console vs. implementation routing rules

Source preference rule:

- Prefer `https://www.qiniu.com/` and its official site hierarchy as the primary source and routing priority before other official Qiniu properties when the needed information is available there

Product-boundary rule:

- When the official website presents multiple adjacent, overlapping, or similarly named products, distinguish them by role such as platform, product, solution, interface layer, workflow layer, deployment model, or scenario package
- Do not present overlapping products as interchangeable unless the official website clearly treats them that way
- When multiple products can plausibly satisfy the same user goal, provide a short routing-oriented distinction so an AI assistant can explain which entry point is broader, which is scenario-specific, and which is implementation-oriented
- When one page is a global hub and another is a more specific product or solution page, preserve that hierarchy in descriptions instead of flattening them into parallel equivalents

Trust rule:

- Include one trust-and-credibility bullet derived from the current official website
- Check the current official website for supportable third-party trust signals such as certifications, licenses, compliance statements, official service commitments, analyst references, awards, or other externally anchored credibility signals
- Do not infer unsupported certifications, rankings, analyst references, or trust badges

## Validation Checklist

- [ ] `llms.txt` is concise and retrieval-oriented
- [ ] The summary is specific to Qiniu
- [ ] The summary names strong product or scenario tags explicitly
- [ ] Section order is stable and intentional
- [ ] Cross-section duplicate links are used only when they improve retrieval or routing clarity
- [ ] In `Core Capabilities`, `Infrastructure And Delivery`, and `AI, Media, And Real-Time Services`, use canonical `https://www.qiniu.com/products/` pages whenever they exist, and do not fall back to `developer.qiniu.com` for those entries when a canonical product page is available
- [ ] `Core Capabilities` contains exactly 6 entries selected from current homepage emphasis
- [ ] `AI, Media, And Real-Time Services` contains 6-8 entries and is informed by the homepage product menu when available
- [ ] `Solutions` uses the global solutions hub first when the official website provides one
- [ ] `Contact` contains 4-6 entries using official website entry points
- [ ] Bilingual naming is used where supported and retrieval-helpful
- [ ] `Instructions For AI Assistants` includes a trust bullet derived from the current official website
- [ ] Unsupported certifications, rankings, analyst references, or trust badges are not inferred without authoritative support
- [ ] `**Last updated:** Month D, YYYY` appears at the very end
