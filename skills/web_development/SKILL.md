# Web Development Craft

Not yet wired into a Creator generation pipeline (Creator has no dedicated website/SaaS project type yet) — authored ahead of that pipeline existing, both as a reference for Flutter Web output (which Creator CAN already build via `flutter build web`) and as a starting point once a dedicated web pipeline is built.

## Responsive layout is not optional

- Every layout needs to work across at least three rough breakpoints (mobile ~375-480px, tablet ~768px, desktop ~1200px+) — a fixed-width design that only works at one viewport size is the most common "clearly not production-ready" tell in generated web output.
- Prefer relative units (`%`, `rem`, `vw`/`vh`, CSS Grid/Flexbox's own sizing) over fixed pixel widths for anything that should adapt — fixed-pixel layouts are what break first when the viewport changes.
- Use CSS Grid for two-dimensional layout (a page's overall structure) and Flexbox for one-dimensional layout (a row of nav items, a column of cards) — reaching for one when the other fits better tends to produce more overridden/hacky CSS than necessary.

## Semantic HTML and accessibility

- Use semantic elements (`<nav>`, `<main>`, `<header>`, `<footer>`, `<article>`, `<button>` for anything clickable) instead of `<div>`/`<span>` with click handlers for everything — this is both an accessibility requirement (screen readers rely on semantic structure) and a real SEO factor, not just a style preference.
- Every image needs meaningful `alt` text (or explicitly empty `alt=""` for a purely decorative image) — missing `alt` text is one of the most common, easiest-to-avoid accessibility failures.
- Form inputs need an associated `<label>` (via `for`/`id` or wrapping) — a form that only relies on placeholder text as a label is inaccessible and loses the label entirely once the user starts typing.

## Performance basics

- Images should be sized/compressed appropriately for their actual display size, not a full-resolution source image scaled down purely in CSS — this is one of the largest, easiest-to-fix contributors to slow page loads.
- Load non-critical JavaScript (analytics, below-the-fold interactive widgets) asynchronously/deferred rather than blocking initial render.

## SaaS/product-site specific conventions

- A marketing/landing page needs a clear, singular call-to-action above the fold — a page with multiple competing CTAs or no clear primary action converts worse regardless of how polished the visual design is.
- Pricing pages should make the recommended/most-common tier visually distinct (not just listed identically alongside the others) — an undifferentiated pricing table forces the user to do comparison work the design should be doing for them.
- Every form (signup, contact, checkout) needs inline validation feedback (not just a rejection after full submission) and a clear success state after submission — silent submission with no confirmation reads as broken even when it technically worked.
