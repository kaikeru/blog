# Travel-Notebook Redesign - Design

Date: 2026-10-20

## Goal

Redesign kaikeru.com as a minimal, cozy travel-journal / traveler's-notebook site. Keep the same page types (home list, single post, resume, reading list, 404), the same content, Hugo, and free GitHub Pages hosting. No analytics, no framework, no JavaScript beyond what the menu needs.

## Decisions (from brainstorming)

- Depth: new look, same structure. No new content types.
- Paper: plain warm paper. No dot grid, no ruled lines (they never align with text and hurt readability).
- Type: Lora for headings and body; Caveat for small handwritten accents only (never body text).
- Layout: left column with site name and menu; content centered in the window.
- Home: posts as taped, page-shaped notes, grouped by month, slightly askew, with handwritten tags.
- Post page: one plain straight page with a single strip of tape (no stacked sheets, no stitch line).
- Centering: content is centered in the viewport, not merely to the right of the menu, so readers never look left on wide screens.

## Layout

- Every page uses a three-column grid: `1fr | content | 1fr`. The left slot holds `kaikeru` and the menu (sticky, stays in view while scrolling); the right slot is empty. The content column is exactly centered in the window.
- Content column max width is 680px on all pages (home, post, resume, reading list, 404).
- Below about 1100px, where the side slots would crowd the content, the menu becomes a row above the content (also the phone layout).
- Left column: `kaikeru` in Caveat (large), a short sage rule, then menu items `entries`, `resume`, `reading list` (current page marked with a green triangle), then `github` and `linkedin` as off-site links. Menu items come from the existing `menu` config; Twitter stays removed.

## Visual system

- Fonts: Lora (400, 600, italic 400) and Caveat (500, 700), self-hosted as woff2 in `themes/kaikeru/static/fonts` so pages make no third-party requests. System serif / cursive fallbacks, `font-display: swap`.
- Colors (CSS custom properties): paper background `#faf8f3`, note/page surface `#fffdf8`, border `#e6dfd0`, ink `#2f2a25`, accent sage darkened as needed so handwritten text meets WCAG AA contrast on the surface color (mockup sage `#5f8f7d` is slightly light for small text), tape colors sage / sand / rose at about 50% opacity.
- Bulma and Font Awesome are removed; one small hand-written stylesheet replaces them. Chroma `syntax.css` (onedark) is kept; code blocks sit in the page as dark printouts.

## Home page

- Posts are grouped by month under handwritten (Caveat, about 30px) month headings with a thin rule, newest first.
- Notes are page-shaped (4:5) in the 680px column: 2 per row (about 326px each), wrapping onto new rows, 1 per row on a phone. The last row is left aligned. A month with one post leaves the rest of its row empty.
- Each note: handwritten date and read time, Lora title, excerpt faded out at the bottom, up to 3 handwritten tags at the bottom (prefixed `~` on the first). The whole note is the link target. Excerpt is plain text (`plainify`), not block HTML.
- Askew effect in pure CSS: a repeating five-step pattern via `:nth-child` sets rotation (between about -1.5 and +1.5 degrees) and tape color/position, so it looks random but is identical on every visit and rebuild. Notes straighten on hover and focus; rotation is disabled under `prefers-reduced-motion`. Phone tilt is gentler.
- Pagination is kept at 24 notes per page, with simple handwritten previous/next links. Month headings continue across pages.

## Post page

- Left column and a centered plain page (680px) with one strip of tape, soft shadow, no tilt.
- Handwritten date and read time, Lora title and body, handwritten tags at the bottom, then a thin rule with handwritten previous / next entry links.
- Resume, reading list and other single pages use the same page without the date, read time or tags.

## 404

A taped page reading "Page not found" with a handwritten link back to the entries.

## Files affected (all under `themes/kaikeru` unless noted)

- Layouts: `_default/baseof.html`, `_default/single.html`, `_default/list.html`, `index.html`, `404.html`, `partials/header.html`, `partials/footer.html`, `partials/head.html`, `partials/list-item.html`, `partials/pagination.html`.
- Static: new `css/style.css` (replaces the Bulma-based one), `fonts/` (Lora, Caveat woff2), existing `css/syntax.css` kept.
- Content, `config.toml` and the deploy workflow are unchanged. Config may gain a `paginate` setting if needed.

## Accessibility and performance

- Text contrast meets WCAG AA; visible keyboard focus styles on menu items and notes; the tilt respects `prefers-reduced-motion`.
- No JavaScript is required; the narrow-screen menu is a plain row, not a toggle.
- Two font families, self-hosted and subset where practical; no external requests.

## Verification

- `hugo --gc --minify` builds with no warnings.
- Visual check in a browser at phone, laptop and wide widths for the home page, a post, the resume, the reading list and the 404: content centered, no horizontal scroll, notes wrap correctly, tags and month headings render.
- Check contrast of handwritten accents, keyboard focus, and that code blocks still render.
- Test with sample extra posts (not committed) to confirm wrapping, month grouping with several posts, and pagination.

## Risks

- Handwriting fonts read poorly if overused; kept to short accents only.
- Askew notes can look messy if heights differ; mitigated with fixed page-shaped size and clipped, faded excerpts.
- Removing Bulma may expose unstyled markup in older posts (tables, images, blockquotes); the new stylesheet needs basic styles for these, checked against the existing four posts.

## Out of scope

Dark mode, new content types (trips, places, photo pages), search, comments, analytics, and any change to post content.
