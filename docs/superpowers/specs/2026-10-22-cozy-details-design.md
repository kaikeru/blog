# Cozy Details - Design

Date: 2026-10-22

## Goal

Add cozy, journal-like touches to the live notebook theme (`themes/kaikeru`) without changing its layout, type, colors, pagination, or its no-JavaScript, free-hosting nature. Everything new degrades gracefully: a post that doesn't use a feature looks exactly as it does today.

## Scope

In: paper grain and vignette, patterned washi tape, hand-drawn underline on month headings, footer sign-off doodle, 404 stamp, a "currently" sticky note (Goodreads plus a data file), margin notes (shortcode), optional taped photos on notes and posts.

Out: evening/dark mode (deferred), new content types, JavaScript, third-party requests from the browser, any change to existing posts.

## Look

### Paper grain and vignette
- Body background keeps `--paper` and gains a faint noise texture drawn as an inline SVG (`feTurbulence`) data URI tiled at about 180px, at about 9% opacity; no image file.
- A soft vignette (radial gradient, about 10% warm brown at the edges) is layered on the page background. Both are decorative only, never over text, and removed in print.
- Notes and pages (`--surface`) stay flat so text contrast is unchanged. Contrast of `--accent` and `--ink` on the textured paper must still be at least 4.5:1 (re-run the existing contrast check).

### Washi tape patterns
- Notes cycle three tapes by `:nth-child(3n+k)`: sage stripes (`repeating-linear-gradient` 45deg, sage at about 60% / white at about 45%), sand polka dots (white dots over sand), rose checks (`conic-gradient`). Positions and small rotations keep the existing pattern.
- The post/plain page tape (`.page::before`) becomes the sage stripes. All tape is CSS only.

### Pen underline
- `.month` loses its straight rule; it gets a wobbly 8px-high SVG underline (data-URI background, accent at about 55% opacity, full heading width) at the bottom.

### Footer sign-off
- `footer.html`: a small inline-SVG compass doodle in the accent color, the handwritten line "thanks for stopping by", then the existing copyright line (smaller). SVG has `aria-hidden="true"`.

### 404 stamp
- `404.html` gets a `.stamp` element reading "LOST?": Caveat 700, red `#b0493f`, 4px double border, rotated about -8deg, opacity about 85%, above the existing title.

## "Currently" note

- A yellow sticky (`#fbf1cf`, border `#ecdfa8`, rose tape strip, tilt about -2deg, Caveat) with the heading "currently" and two lines: `reading: <title> by <author>` (title links to its Goodreads book page) and `working on: <text>`, plus an optional small "updated" line.
- Placement: in the left column under the social links on desktop (inside the sticky `.side`). At 1100px and below it moves to the bottom of the page above the footer. Implemented as two instances of one partial (`currently.html`) with CSS showing exactly one per breakpoint (the hidden one is `display: none`, so it is not read twice). Marked up as `<aside aria-label="Currently">`.
- Data:
  - `data/currently.toml` with optional keys `reading` (fallback text), `working_on`, `updated`.
  - Goodreads URL in `config.toml` as `params.goodreadsShelf = "https://www.goodreads.com/review/list_rss/38312672?shelf=currently-reading"`. The public shelf feed needs no key (verified); nothing secret is stored.
- Build-time fetch (in `currently.html`, via `resources.GetRemote` inside `try`): parse the RSS with `transform.Unmarshal`, take the first `<item>`: title (strip the trailing series text in parentheses), `author_name` (collapse repeated spaces), and `book_id` for the link `https://www.goodreads.com/book/show/<book_id>`.
- Fallbacks, in order: fetched book; `reading` from the data file; otherwise the reading line is omitted. If both lines are empty the whole note is omitted. A fetch error logs a warning (`warnf`) and never fails the build.
- Caching: `[caches.getresource] maxAge = "1h"` in `config.toml` so local rebuilds do not keep a stale response.
- Refresh: `.github/workflows/deploy.yml` gains `schedule: - cron: "17 11 * * *"` (daily) so the book updates without a push. GitHub pauses scheduled workflows after about 60 days with no repository activity; a push or manual run re-enables it.
- Links: the book title links to its Goodreads book page, and a `Goodreads` entry (your profile, `https://www.goodreads.com/user/show/38312672-kyle`) is added to the social links between Github and Linkedin via `[[menu.social]]` in `config.toml`.
- Privacy: only the public shelf, fetched by the build server; the browser makes no request to Goodreads (no cover images are used).

## Margin notes

- Shortcode `layouts/shortcodes/note.html`: `{{< note >}}text{{< /note >}}` renders `<aside class="margin-note">` with the inner content (Markdown allowed, inline only).
- Wide screens (viewport at least 1280px): the note floats right with a negative right margin so it sits in the empty right-hand slot, level with the paragraph it follows: width 190px, yellow sticky style, Caveat, tilt about +2deg, a short pen-line connector on its left edge. It must stay inside the viewport (no horizontal scroll) at 1280px and wider.
- Narrower: it renders inline as a small taped yellow note, full width of the text column, no float.
- Print: inline, no tilt.

## Photos on notes and posts

- Optional front matter `image` (site-root path under `static/`, e.g. `/images/kyoto.jpg`, or a full URL) and `image_alt`. No image processing in this version; authors should supply a JPEG about 800px wide.
- Post page: a taped, white-bordered print (6px frame, bottom 20px for a caption), tilted about +1.5deg, at the top of the page content under the title. Caption is `image_alt` in Caveat.
- Note: the same print, smaller (about 28% of the note's height), above the date line; title clamps to 2 lines and the excerpt shrinks to fit. The note stays 4:5.
- `loading="lazy"`, explicit `width`/`height` not required. `image_alt` empty means the image is decorative (`alt=""`) and no caption is shown.
- No `image` means no markup change at all.

## Files affected

Under `themes/kaikeru` unless noted:
- `static/css/style.css`: grain, vignette, tape patterns, underline, currently note, margin note, photo, stamp, footer, print and responsive rules.
- `layouts/partials/`: new `currently.html`; edited `list-item.html` (photo), `header.html`, `footer.html`.
- `layouts/_default/baseof.html`: second `currently.html` instance above the footer.
- `layouts/_default/single.html`, `layouts/404.html`: photo, stamp.
- `layouts/shortcodes/note.html`: new.
- Repo root: `data/currently.toml` (new), `config.toml` (`params.goodreadsShelf`, `[caches.getresource]`), `.github/workflows/deploy.yml` (schedule).
- Posts, `baseURL`, `static/CNAME` unchanged.

## Accessibility and performance

- Decorative layers (grain, vignette, tape, doodles) are CSS or `aria-hidden` SVG; no text is carried by them. Currently note and margin notes are `<aside>`; photos need `image_alt` or are decorative.
- Existing focus styles and reduced-motion rules stay; new tilts are also disabled under `prefers-reduced-motion`.
- Grain is a single tiled background (no fixed attachment, no animation) to keep scrolling cheap.
- No JavaScript; no new client-side requests.

## Verification

- `hugo --gc --minify` builds with no warnings; also builds with the network blocked or an invalid Goodreads URL (fallback path) and with the data file empty.
- Currently note: shows the live book; falls back to the data-file `reading`; omits cleanly when empty; link points at the Goodreads book page.
- Temporary sample content (not committed): a post with `image` and `image_alt`, one with `image` and empty alt, one with margin notes.
- Browser checks (via fixed-width iframes, since the window cannot be resized) at 390, 1100, 1101, 1279, 1280 and 1500px: no horizontal scroll, no overlap, currently note in the correct place, margin note inside the viewport, photos tilted and clipped correctly, notes still 4:5, centered content unchanged.
- Re-run the contrast check on the textured paper; check the 404 stamp, the footer, and print preview basics.

## Risks

- Goodreads could change or block the feed; mitigated by the fallback and a warning-only failure. A stale note between rebuilds is expected (daily).
- Scheduled workflows can be paused by GitHub after 60 days of inactivity; documented above.
- Floats with negative margins for margin notes can clip or overflow at the 1280px threshold; verified at 1279/1280/1500px.
- SVG noise may render slightly differently across browsers; kept faint so differences are not visible.

## Out of scope

Evening mode, image processing/resizing, Goodreads covers, ratings or review text, any other Goodreads shelf, and any change to existing content.
