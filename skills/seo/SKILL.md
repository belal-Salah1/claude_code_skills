---
name: seo
description: Use when creating or editing pages, routes, layouts, or templates in a website - covers titles and meta descriptions, heading order, semantic HTML, canonical and Open Graph URLs, structured data, sitemaps, and crawlable content
---

# SEO Rules

Apply to any page a search engine should index. Prefer the project's existing
SEO components, utilities, and patterns over introducing new ones.

## Metadata

- Every indexable page needs a unique title and meta description.
- Render all metadata into the HTML the server sends — at build time or on the
  server. Never inject meta tags client-side.
- Canonical and Open Graph URLs must be absolute, derived from a single
  configured site URL rather than hardcoded or built from the current location.
- Follow the project's existing canonical, Open Graph, and structured-data
  patterns.

## Structure and semantics

- Use one clear H1, then H2/H3 in order — never skip a level for styling.
- Use semantic elements: `<main>`, `<section>`, `<table>`, `<ol>`/`<ul>`,
  `<form>`, `<label>`, `<button>`.
- Data belongs in a real `<table>` with `<caption>` and scoped `<th>`, not a
  grid of divs.

## Structured data

- Emit structured data as `<script type="application/ld+json">` rendered with
  the page, not added by client-side script.
- Keep visible markup consistent with the structured data describing it — FAQ,
  breadcrumb, product, and review markup must match what the page actually shows.

## Crawlability

- New routes must appear in the generated sitemap.
- Keep important explanatory content in the server-rendered HTML, before
  JavaScript runs.

## Content

- Avoid generic filler SEO content.
