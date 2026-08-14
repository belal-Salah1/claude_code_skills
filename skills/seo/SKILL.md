---
name: seo
description: Use when creating or editing pages, routes, layouts, templates, sitemaps, robots.txt, or structured data in a website - covers titles and meta descriptions, heading order, semantic HTML, canonical and Open Graph URLs, JSON-LD schema, images and alt text, internal links, URL slugs, indexing, Core Web Vitals, author and trust signals, and being cited by AI answer engines
---

# SEO & AEO Rules

Applies to any page that should be found — by a search engine, or by an answer
engine like Google AI Overviews, ChatGPT, Perplexity, or Copilot. The two mostly
want the same things: a clear answer, structured markup, and a source worth
trusting.

Prefer the project's existing SEO components, utilities, and patterns over
introducing new ones.

## Metadata

- Every indexable page needs a unique title and meta description. Duplicates
  across pages are a bug, not a shortcut.
- Titles 50–60 characters, descriptions 150–160 — longer gets truncated.
- Put the distinguishing words first: `Loan Calculator | Site` beats
  `Site | Tools | Loan Calculator`.
- Render all metadata into the HTML the server sends — at build time or on the
  server. Never inject meta tags client-side.
- Canonical and Open Graph URLs must be absolute, derived from a single
  configured site URL rather than hardcoded or built from the current location.
- Every indexable page needs a self-referencing canonical. Syndicated or
  duplicated content points its canonical at the original.
- Give social previews an explicit image, title, and description. Open Graph
  images should be ~1200x630.
- Follow the project's existing canonical, Open Graph, and structured-data
  patterns.

## Structure and semantics

- Use one clear H1, then H2/H3 in order — never skip a level for styling.
- Use semantic elements: `<main>`, `<section>`, `<table>`, `<ol>`/`<ul>`,
  `<form>`, `<label>`, `<button>`.
- Data belongs in a real `<table>` with `<caption>` and scoped `<th>`, not a
  grid of divs.
- Set `lang` on `<html>`. For multi-language sites, add reciprocal `hreflang`
  links including a self-reference and an `x-default`.
- Include the responsive viewport meta tag.

## Answering the question

Answer engines extract answers; they don't reward buildup. Write so a machine
can lift the answer out cleanly.

- **Lead with the answer, then explain.** Not 500 words of history before the
  definition.
- **Headings should be the questions users ask.** "How does X work?" beats
  "Details". "What is X?" beats "Overview".
- **Prefer lists, tables, and Q&A pairs** over long prose for anything
  enumerable — they extract far more reliably than paragraphs.
- **Cover the follow-up questions** on the same page, or link to a page that
  does. Being the complete answer is what gets cited.
- **Define terms and address common misconceptions** rather than assuming.
- **Show freshness:** display publish and update dates, and emit `dateModified`
  in structured data. Update content substantively — touching the date without
  changing the content is worse than leaving it.
- **Cite primary sources** and link to them: studies, documentation, official
  references.

## Trust signals

Both ranking and AI answer selection weigh who is behind the content.

- Attribute content to a named author, with a bio, credentials, and links to
  their profiles (`sameAs` in `Person` schema).
- Show publication and last-updated dates.
- Provide real contact information, a privacy policy, and terms.
- Serve everything over HTTPS.
- Show first-hand evidence where it exists: real examples, screenshots, test
  results, case studies — not theory alone.
- **YMYL topics** (health, finance, legal, safety) need extra rigor: review by a
  qualified expert, a visible reviewer credit, and disclaimers where
  appropriate.

## Links and URLs

- Slugs: lowercase, hyphenated, descriptive, no dates or IDs unless meaningful.
- Treat published URLs as permanent. Changing one requires a 301 from the old
  URL, plus updating internal links that point at it.
- Internal link text should describe the destination — never "click here" or
  "read more" alone.
- Be consistent about trailing slashes; one form should redirect to the other,
  not serve both.
- Every indexable page should be reachable by a crawlable `<a href>` from
  somewhere in the site. Links built only by JavaScript click handlers are not.

## Images and media

- Every meaningful image needs alt text describing it. Decorative images take
  empty `alt=""`.
- Set explicit width and height (or aspect ratio) so the layout does not shift
  while loading.
- Lazy-load below-the-fold images; never lazy-load the main above-the-fold one.
- Give descriptive filenames and serve appropriately sized, modern formats
  (WebP/AVIF).

## Structured data

- Emit structured data as `<script type="application/ld+json">` rendered with
  the page, not added by client-side script.
- Keep visible markup consistent with the structured data describing it. Only
  mark up what the page genuinely contains — invented ratings, prices, or FAQs
  risk a manual penalty.
- Values must be plain text. Strip rich text and HTML before serializing, and
  escape anything user-generated.
- Validate before shipping: [Rich Results Test](https://search.google.com/test/rich-results),
  [Schema.org Validator](https://validator.schema.org/).

| Page type | Type | Key fields |
| --- | --- | --- |
| Article, blog post | `Article` / `BlogPosting` | `headline`, `description`, `image`, `datePublished`, `dateModified`, `author`, `publisher` |
| FAQ | `FAQPage` | `mainEntity[]` of `Question` → `acceptedAnswer` |
| Product | `Product` | `name`, `image`, `offers` (`price`, `priceCurrency`, `availability`), `aggregateRating` |
| Site-wide | `Organization` | `name`, `url`, `logo`, `sameAs[]`, `contactPoint` |
| Author | `Person` | `name`, `url`, `jobTitle`, `sameAs[]` |
| Any nested page | `BreadcrumbList` | `itemListElement[]` of `ListItem`, `position` starting at 1 |

A page usually needs several. Combine them under `@graph` with `@context`
declared once:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    { "@type": "Article", "headline": "..." },
    { "@type": "BreadcrumbList", "itemListElement": [] },
    { "@type": "Organization", "name": "..." }
  ]
}
```

## Crawlability and indexing

- New routes must appear in the generated sitemap; excluded and noindexed pages
  must not. Include `lastModified` — `changefreq` and `priority` are ignored by
  Google.
- `robots.txt` should point at the sitemap, disallow API and admin routes, and
  must not block CSS or JS the page needs to render.
- Decide deliberately whether AI crawlers may read the site — `GPTBot`,
  `ClaudeBot`, `PerplexityBot`, `Google-Extended`. Allowing them makes citation
  possible; blocking them keeps content out of training. `Google-Extended` is
  separate from Search indexing, so blocking it costs no rankings. Revisit this
  periodically; it changes fast.
- Noindex thin, duplicate, or utility pages: internal search results, filter
  permutations, staging routes, thank-you pages.
- Missing pages must return a real 404 (or 410), not a 200 with an error
  message.
- Never let a staging or preview deployment serve indexable pages.
- Keep important explanatory content in the server-rendered HTML, before
  JavaScript runs.

## Performance

Core Web Vitals affect rankings. Targets:

- **LCP** (largest contentful paint) < 2.5s
- **INP** (interaction to next paint) < 200ms
- **CLS** (cumulative layout shift) < 0.1

- The largest above-the-fold element should load early — preload or prioritize
  it, and don't hide it behind a client-side fetch.
- Reserve space for anything that loads late: images, embeds, ads, banners.
- Load fonts with `font-display: swap` and a matched fallback metric.
- Don't block rendering on non-critical scripts.

## Content

- Avoid generic filler SEO content.
- One page per topic. Two pages targeting the same intent compete with each
  other; merge them and redirect.

## New page checklist

```text
[ ] Unique title and meta description, correct length
[ ] Self-referencing absolute canonical
[ ] Open Graph / social tags with an image
[ ] Exactly one H1, headings in order, phrased as real questions
[ ] Answer appears before the explanation
[ ] Semantic markup; tables are real tables
[ ] All images have alt text and explicit dimensions
[ ] Author, publish date, and update date shown where relevant
[ ] Structured data matches visible content, and validates
[ ] Route is in the sitemap and linked from somewhere crawlable
[ ] Main content present in server-rendered HTML
[ ] Any changed URL has a 301 from the old one
```

## Common mistakes

| Mistake | Why it hurts |
| --- | --- |
| Metadata set client-side in a `useEffect` / `onMounted` | Crawlers may index the pre-hydration HTML — the default title |
| Canonical built from `window.location` | Yields wrong or relative URLs, and can't run at build time |
| Copying the same description across a page family | Search engines drop duplicates rather than ranking both |
| Headings chosen for font size | Breaks the document outline; use CSS for size |
| Headings like "Overview" / "Details" / "More" | Match nothing anyone searches for, and give AI no extraction anchor |
| Burying the answer under context and history | Answer engines lift the first clear statement; there isn't one |
| Content revealed only after a client-side fetch | Not reliably crawled or indexed |
| Renaming a URL without a redirect | Discards existing rankings and backlinks |
| Marking up FAQs the page doesn't display | Structured-data spam; risks a manual penalty |
| Bumping `dateModified` without editing the content | A known spam signal, not a freshness win |
| Rich text or HTML passed into JSON-LD values | Invalid structured data; may be rejected wholesale |

---

Parts of the AEO, trust-signal, and structured-data guidance are adapted from
[sanity-io/agent-toolkit](https://github.com/sanity-io/agent-toolkit/tree/main/skills/seo-aeo-best-practices)
(MIT), generalized to be framework-agnostic.
