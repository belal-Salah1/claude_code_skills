---
name: seo
description: Use when creating or editing pages, routes, layouts, templates, sitemaps, or redirects in a website - covers titles and meta descriptions, heading order, semantic HTML, canonical and Open Graph URLs, structured data, images and alt text, internal links, URL slugs, indexing, and crawlable content
---

# SEO Rules

Apply to any page a search engine should index. Prefer the project's existing
SEO components, utilities, and patterns over introducing new ones.

## Metadata

- Every indexable page needs a unique title and meta description. Duplicates
  across pages are a bug, not a shortcut.
- Keep titles ~60 characters and descriptions ~155 characters — longer gets
  truncated in results.
- Put the distinguishing words first: `Loan Calculator | Site` beats
  `Site | Tools | Loan Calculator`.
- Render all metadata into the HTML the server sends — at build time or on the
  server. Never inject meta tags client-side.
- Canonical and Open Graph URLs must be absolute, derived from a single
  configured site URL rather than hardcoded or built from the current location.
- Every indexable page needs a self-referencing canonical.
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
  links including a self-reference.
- Include the responsive viewport meta tag.

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
- Give descriptive filenames and serve appropriately sized, modern formats.

## Structured data

- Emit structured data as `<script type="application/ld+json">` rendered with
  the page, not added by client-side script.
- Keep visible markup consistent with the structured data describing it — FAQ,
  breadcrumb, product, and review markup must match what the page actually shows.
- Only mark up what the page genuinely contains. Invented ratings, prices, or
  FAQs risk manual penalties.
- Validate new structured data before shipping it.

## Crawlability and indexing

- New routes must appear in the generated sitemap; excluded and noindexed pages
  must not.
- `robots.txt` should point at the sitemap and must not block CSS or JS the page
  needs to render.
- Noindex thin, duplicate, or utility pages: internal search results, filter
  permutations, staging routes, thank-you pages.
- Missing pages must return a real 404 (or 410), not a 200 with an error
  message.
- Never let a staging or preview deployment serve indexable pages.
- Keep important explanatory content in the server-rendered HTML, before
  JavaScript runs.

## Performance

- The largest above-the-fold element should load early — preload or prioritize
  it, and don't hide it behind a client-side fetch.
- Reserve space for anything that loads late: images, embeds, ads, banners.
- Don't block rendering on non-critical scripts.

## Content

- Avoid generic filler SEO content.
- One page per topic. Two pages targeting the same intent compete with each
  other; merge them and redirect.
- Front-load the answer — the useful content goes above the explanation of why
  the page exists.

## New page checklist

```text
[ ] Unique title and meta description, correct length
[ ] Self-referencing absolute canonical
[ ] Open Graph / social tags with an image
[ ] Exactly one H1, headings in order
[ ] Semantic markup; tables are real tables
[ ] All images have alt text and explicit dimensions
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
| Content revealed only after a client-side fetch | Not reliably crawled or indexed |
| Renaming a URL without a redirect | Discards existing rankings and backlinks |
| Marking up FAQs the page doesn't display | Structured-data spam; risks a manual penalty |
