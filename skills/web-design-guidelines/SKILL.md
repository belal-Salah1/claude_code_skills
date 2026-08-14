---
name: web-design-guidelines
description: Use when building or reviewing any web UI — a generic accessibility and UX checklist covering WCAG 2.2 (semantic HTML, ARIA, images, contrast), focus and keyboard navigation, forms, touch targets, navigation landmarks, dialogs, async loading/error/empty states, motion, responsive layout and zoom, typography, Core Web Vitals, dark mode, i18n, hydration safety, and interactive states.
---

# Web Interface Design Guidelines Reference

Comprehensive rules for reviewing web UI code against WCAG 2.2, Core Web Vitals, and modern UX best practices.

This is the generic, framework-agnostic checklist. For project-specific styling
gates (BEM, design tokens, z-index scale, overflow mixins), use `ui-review`
instead — the two are complementary, not duplicates.

---

## 1. Accessibility (WCAG 2.2 Compliance)

### 1.1 Semantic HTML (First Rule)

> "If you can use a native HTML element with built-in semantics, use it instead of ARIA."

- `<button>` for actions, `<a href>` for navigation — never a `<div onClick>`
- `<nav>`, `<main>`, `<header>`, `<footer>`, `<aside>`, `<section>`, `<article>` for page regions
- `<ul>`/`<ol>`/`<li>` for lists, `<table>` with `<caption>`/`<th scope>` for tabular data
- `<form>`, `<fieldset>`, `<legend>`, `<label>` for forms
- `<h1>`–`<h6>` in order — never skip a level for styling
- Native `<dialog>`, `<details>`/`<summary>`, `<select>` before hand-rolled equivalents

### 1.2 ARIA (Only When Semantics Fall Short)

- ARIA is a patch, not a default — a wrong role is worse than no role
- Custom widgets need `role` + the keyboard behavior that role implies
- Dynamic content needs live regions (`aria-live="polite"`, `aria-live="assertive"` for critical errors)
- Collapsed sections need `aria-expanded="false"`, expanded need `aria-expanded="true"`
- Hidden elements need `aria-hidden="true"` to exclude from screen readers
- Invalid form fields need `aria-invalid="true"`
- Required fields need `aria-required="true"` or HTML `required`

### 1.3 Images & Media

- All `<img>` MUST ATTENTION have `alt` attribute
- Decorative images: `alt=""`
- Informative images: descriptive `alt` text explaining content/purpose
- Complex images (charts, diagrams): provide long description or `aria-describedby`
- Background images with meaning need a text alternative nearby
- Video: provide captions and transcripts
- Audio: provide transcripts

### 1.4 Color & Contrast

- **Normal text**: minimum 4.5:1 contrast ratio
- **Large text** (18px+ or bold 14px+): minimum 3:1 contrast ratio
- **UI components** (buttons, inputs, icons): minimum 3:1 contrast ratio
- NEVER rely solely on color to convey meaning (add icons, text, or patterns)
- Test with colorblind simulators
- Error states: don't just use red -- add icons or text

---

## 2. Focus & Keyboard Navigation

### 2.1 Focus Visibility

- Interactive elements MUST have a visible focus indicator
- NEVER `outline: none` without a replacement indicator
- Focus indicator needs 3:1 contrast against the adjacent background
- Prefer `:focus-visible` so pointer users don't see rings on click
- Focus indicator must not be clipped by `overflow: hidden` on an ancestor

### 2.2 Focus Order

- Tab order follows visual/reading order
- NEVER `tabindex` > 0 — it breaks natural tab order
- `tabindex="0"` adds a custom widget to the tab order; `tabindex="-1"` makes it programmatically focusable only
- Hidden or off-screen elements must not be tabbable

### 2.3 Keyboard Operability

- Every mouse interaction has a keyboard equivalent
- Enter/Space activate buttons; Enter follows links; Escape closes overlays
- Arrow keys navigate composite widgets (menus, tabs, listboxes, grids)
- No keyboard traps — focus can always move out of a component
- Provide a skip link to `<main>` as the first focusable element

### 2.4 Focus Management

- Opening a dialog moves focus into it; closing returns focus to the trigger
- Focus is trapped inside a modal while it is open
- Route changes in an SPA move focus to the new page heading and announce it
- Removing the focused element moves focus somewhere sensible, never to `<body>`

---

## 3. Forms & Inputs

- Every input has a `<label>` with a matching `for`/`id` — placeholder is NOT a label
- Group related controls in `<fieldset>` with `<legend>` (radio groups, address blocks)
- Use correct `type` (`email`, `tel`, `url`, `number`, `date`) plus `inputmode` and `autocomplete`
- Errors: identify the field, describe the problem in text, and link it with `aria-describedby`
- Move focus to the first invalid field on submit and summarize errors at the top
- Validate on blur/submit, not on every keystroke
- NEVER block paste (`onPaste` + `preventDefault()`) — it is hostile to password managers
- Never disable the submit button as the only validation feedback
- Show requirements (password rules, formats) before submission, not after failure
- Destructive actions need confirmation; irreversible ones need typed confirmation

---

## 4. Touch Targets & Pointer Input

- Minimum touch target 24x24 CSS px (WCAG 2.2 AA); 44x44 recommended
- Adjacent targets need spacing so a thumb can't hit two at once
- Hover-only affordances need a tap/focus equivalent — hover doesn't exist on touch
- No hover-triggered content that can't be reached by keyboard or dismissed
- Drag interactions need a single-pointer alternative (WCAG 2.2 "Dragging Movements")
- Respect safe areas on notched devices (`env(safe-area-inset-*)`)

---

## 5. Navigation & Landmarks

- One `<main>` per page, one `<h1>` per page
- Landmarks: `<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>` — label duplicates with `aria-label`
- Mark the current page with `aria-current="page"`
- Breadcrumbs in a `<nav aria-label="Breadcrumb">` with an `<ol>`
- Link text describes its destination — never "click here" or a bare "read more"
- External links and file downloads state that in the accessible name

---

## 6. Modals, Dialogs & Overlays

- Use `<dialog>` or `role="dialog"` + `aria-modal="true"`
- Label it with `aria-labelledby` pointing at the dialog title
- Trap focus while open, restore focus to the trigger on close
- Escape closes; a backdrop click closes only when nothing is unsaved
- Lock body scroll while open, restore scroll position on close
- Content behind the dialog is inert (`inert` attribute or `aria-hidden`)
- Toasts announce via a live region and stay long enough to read; never put the only action inside a toast that auto-dismisses

---

## 7. Async States: Loading, Error, Empty

Every async surface answers three questions: what do I see while it's working,
when it fails, and when there's nothing?

- **Loading**: skeleton or spinner, never a frozen blank screen. Buttons show an in-button spinner and disable
- Announce loading state to screen readers (`aria-busy`, or a polite live region)
- **Error**: a human-readable message with a retry path — never a silent failure or a raw stack trace
- **Empty**: a meaningful empty state (message + optional CTA), not a blank area
- **Disabled/in-flight**: disable the trigger while the request is in flight to prevent double submission
- **Success**: confirm completed mutations (toast, inline message, or visibly updated data)
- Reserve layout space for async content so it doesn't shift when it arrives
- Optimistic updates must roll back visibly on failure

---

## 8. Motion & Animation

- Honor `prefers-reduced-motion: reduce` — disable or replace transforms with fades
- NEVER `transition: all` — list properties explicitly (performance)
- Animate only `transform` and `opacity` where possible; avoid animating layout properties
- Keep UI transitions under ~300ms; longer feels sluggish
- No content that flashes more than 3 times per second (seizure risk)
- Auto-playing carousels/videos need a pause control and must not autoplay with sound
- Parallax and scroll-jacking break scrolling expectations — avoid or gate behind reduced-motion

---

## 9. Responsive Layout & Zoom

- NEVER `user-scalable=no` or `maximum-scale=1` — it disables zoom
- Content reflows at 320px width with no horizontal scroll (WCAG 1.4.10)
- Usable at 400% zoom
- Prefer flex/grid with `flex-wrap` and `row → column` reflow over fixed pixel layouts
- Prefer `min-width`/`max-width` over fixed `width`; `min-width: 0` on truncating flex children
- Wide content (tables, code blocks, diagrams) scrolls inside its own container, not the page body
- Test at 320 / 768 / 1024 / 1440
- Use `dvh`/`svh` rather than `vh` for full-height mobile layouts

---

## 10. Typography & Readability

- Body text minimum 16px; never below 12px
- Line height at least 1.5 for body copy
- Line length 45–75 characters (`max-width: 65ch`)
- Text spacing must survive user overrides (WCAG 1.4.12) — no fixed heights on text containers
- Prefer `rem` for font sizes so browser font settings apply
- No text baked into images
- Sufficient heading hierarchy and spacing; don't fake headings with bold text

---

## 11. Performance & Core Web Vitals

- **LCP** < 2.5s: preload the hero image/font, no lazy-loading above the fold
- **INP** < 200ms: break up long tasks, debounce expensive handlers, avoid layout thrash
- **CLS** < 0.1: `width`/`height` (or `aspect-ratio`) on every `<img>` and embed; reserve space for ads/banners
- Use `loading="lazy"` below the fold, `fetchpriority="high"` on the LCP image
- Serve modern formats (AVIF/WebP) with `srcset`/`sizes`
- `font-display: swap` plus a matched fallback metric to avoid layout shift
- Code-split routes; defer non-critical JS
- Never inject content above existing content after load

---

## 12. Theming & Dark Mode

- Define colors as tokens; don't hardcode hex values per component
- Re-check contrast in dark mode — light-mode ratios do not carry over
- Dim, don't invert: use desaturated surfaces and softened whites for dark themes
- Match `<meta name="theme-color">` to the active theme
- Honor `prefers-color-scheme` media query
- Provide manual toggle that persists preference

---

## 13. Internationalization (i18n)

- Dates/times: use `Intl.DateTimeFormat`
- Numbers/currency: use `Intl.NumberFormat`
- Use logical CSS properties (`margin-inline-start`) not `margin-left`
- Set `dir="rtl"` on `<html>` for RTL languages
- Set an accurate `lang` attribute; mark inline language changes with `lang` too
- Never concatenate translated strings — use interpolated messages with plural rules
- Allow for text expansion (German/Finnish run ~30% longer than English)

---

## 14. Hydration Safety (SSR)

- Inputs with `value` prop MUST ATTENTION have `onChange` handler
- Guard against hydration mismatches from `Date`/timezone/locale-dependent rendering
- Dynamic content based on `window`/`localStorage`: render client-side only
- Never generate random IDs during render — use a stable ID hook
- Server and client must agree on the initial markup; suppress or defer anything that can't

---

## 15. Interactive States

- Buttons and links MUST ATTENTION have hover state
- Every interactive element covers: default, hover, focus, active, disabled, loading
- Disabled: `disabled` attribute + reduced opacity + `cursor: not-allowed`
- Consider `aria-disabled="true"` instead — it keeps the element focusable and discoverable
- Selected/checked/expanded states are exposed to assistive tech, not just styled
- State changes are never conveyed by color alone

---

## Anti-Patterns Quick Reference

| Pattern                                 | Issue                                     |
| --------------------------------------- | ----------------------------------------- |
| `user-scalable=no` or `maximum-scale=1` | Disables zoom -- accessibility violation  |
| `onPaste` + `preventDefault()`          | Blocks paste -- UX hostile                |
| `transition: all`                       | Performance -- list properties explicitly |
| `outline: none` without replacement     | No visible focus indicator                |
| `<div onClick>` or `<span onClick>`     | Should be `<button>` or `<a>`             |
| `<img>` without `width`/`height`        | Causes layout shift (CLS)                 |
| `<img>` without `alt`                   | Missing alternative text                  |
| Form inputs without `<label>`           | Missing label                             |
| Icon buttons without `aria-label`       | Screen reader cannot identify action      |
| `tabindex` > 0                          | Breaks natural tab order                  |
| Color-only indicators                   | Fails colorblind users                    |
| Placeholder as label                    | Label disappears on input                 |

---

## External References

- [WCAG 2.2 Guidelines](https://www.w3.org/WAI/WCAG22/quickref/)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [Core Web Vitals](https://web.dev/vitals/)
- [prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)
