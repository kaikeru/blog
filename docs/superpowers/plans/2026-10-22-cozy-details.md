# Cozy Details Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add cozy touches to the live notebook theme: paper grain and vignette, patterned washi tape, pen-drawn month underlines, footer compass, 404 stamp, a Goodreads-fed "currently" sticky note, margin notes, and optional taped photos.

**Architecture:** All changes live in `themes/kaikeru` plus a small data file, two config keys and a workflow schedule. Decorative effects are CSS-only (inline SVG data URIs, gradients). The "currently" note reads the public Goodreads shelf RSS at build time with `resources.GetRemote` inside `try`, falling back to a data file. Margin notes are a Hugo shortcode styled as a float into the empty right-hand slot. Photos are optional front-matter fields.

**Tech Stack:** Hugo 0.168 (extended), plain CSS, Hugo shortcodes and data files, GitHub Actions schedule.

**Spec:** `docs/superpowers/specs/2026-10-22-cozy-details-design.md`

## Global Constraints

- No JavaScript and no browser requests to third parties; Goodreads is fetched by the build only. No cover images.
- Layout, fonts, content width (`--col: 680px`), pagination and the 1100px breakpoint are unchanged. A post with no `image` and no margin notes must render exactly as before.
- Reading time stays `N min` (notes) and `N min read` (posts). No cups of tea.
- Evening/dark mode is out of scope.
- Contrast must stay at least 4.5:1 on the textured paper (worst-case grain), including the yellow sticky notes. The accent is darkened from `#4a7a68` to `#43705f` to leave room for the grain.
- A failed Goodreads fetch must only warn; the build never fails because of it.
- Do not change post content, `baseURL`, `static/CNAME`, or the existing deploy trigger (push to `master` and `workflow_dispatch` keep working).
- Work on branch `cozy-details`, not `master`. Temporary sample content (posts, images) is never committed.

## Review Focus

- Goodreads outcomes: live book, fetch error (unresolvable host), 404/empty response, and a shelf with exactly one book (the feed returns a single map, not a list) must all be handled (Task 2 tests each).
- Both "currently" instances must never show at the same time, and the note must be absent when there is nothing to show (Task 2).
- Margin note must stay inside the viewport at exactly 1280px and wider with no horizontal scroll, and must be inline (not floated) at 1279px (Task 3 measures).
- A margin note longer than the paragraph beside it must not spill out of the bottom of the page (Task 3).
- A post with `image` but empty or missing `image_alt` renders a decorative image with no caption; `image` as a full URL or a `/images/...` path both resolve (Task 4).
- A note with a photo keeps its 4:5 shape and a long title does not push the excerpt out (Task 4).
- Grain, vignette, doodles and tape never reduce text contrast below 4.5:1 and never appear in print (Task 1).
- Reduced-motion users get no tilt on the new elements (Tasks 2-4).

---

## File Structure

| File | Responsibility |
|---|---|
| `themes/kaikeru/static/css/style.css` | All new styles (appended/edited per task) |
| `themes/kaikeru/layouts/partials/footer.html` | Compass doodle + sign-off + copyright |
| `themes/kaikeru/layouts/404.html` | LOST? stamp |
| `themes/kaikeru/layouts/partials/currently-data.html` | Fetch and normalise the "currently" data (returns a dict) |
| `themes/kaikeru/layouts/partials/currently.html` | Render the sticky note (takes a class string) |
| `themes/kaikeru/layouts/partials/header.html`, `_default/baseof.html` | Place the two `currently` instances |
| `themes/kaikeru/layouts/shortcodes/note.html` | Margin note shortcode |
| `themes/kaikeru/layouts/_default/single.html`, `partials/list-item.html` | Optional photo on post and on note |
| `data/currently.toml` | `working_on`, `updated`, fallback `reading` |
| `config.toml` | `params.goodreadsShelf`, `[caches.getresource]` |
| `.github/workflows/deploy.yml` | Daily schedule |

---

### Task 1: Look: grain, vignette, tape, underline, footer, 404 stamp

**Files:**
- Modify: `themes/kaikeru/static/css/style.css`, `themes/kaikeru/layouts/partials/footer.html`, `themes/kaikeru/layouts/404.html`

**Interfaces:**
- Consumes: existing tokens and classes.
- Produces: darker `--accent` (`#43705f`) used by every later task; `.stamp`, `.doodle`, `.sign`, `.copy` classes.

- [ ] **Step 1: Branch and commit the spec and plan**

```bash
cd /Users/kyle/repositories/blog
git checkout -b cozy-details
git add docs/superpowers
git commit -q -m "Add spec/plan for cozy details"
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"; grep -c 'class=stamp' public/404.html
```
Expected: no WARN/ERROR; the count prints `0` (stamp not there yet).

- [ ] **Step 2: Darken the accent and add the grain and vignette**

In `style.css`, change `--accent: #4a7a68;` to `--accent: #43705f;` (in `:root`).

Replace the line `    background: var(--paper);` (in the `body` rule) with:

```css
    background-color: var(--paper);
    background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='180' height='180'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='.85' numOctaves='2' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 .45  0 0 0 0 .36  0 0 0 0 .25  0 0 0 .09 0'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
```

Directly after the `body { ... }` rule add:

```css
/* Soft warm vignette behind everything */
body::before {
    content: ""; position: fixed; inset: 0; z-index: -1; pointer-events: none;
    background: radial-gradient(ellipse at center, transparent 55%, rgba(120, 90, 50, .10) 100%);
}
```

- [ ] **Step 3: Patterned tape**

Replace the three `.note:nth-child(3n+k)::before` lines with:

```css
.note:nth-child(3n+1)::before { left: 2rem; background: repeating-linear-gradient(45deg, rgba(122, 168, 150, .6) 0 6px, rgba(255, 255, 255, .45) 6px 12px); transform: rotate(-3deg); }
.note:nth-child(3n+2)::before { left: 3.4rem; background: radial-gradient(rgba(255, 255, 255, .75) 1.8px, transparent 2.2px) 0 0 / 9px 9px, var(--tape-sand); transform: rotate(3deg); }
.note:nth-child(3n+3)::before { left: 1.3rem; background: conic-gradient(rgba(196, 120, 120, .5) 25%, rgba(255, 255, 255, .5) 0 50%, rgba(196, 120, 120, .5) 0 75%, rgba(255, 255, 255, .5) 0) 0 0 / 10px 10px; transform: rotate(-2deg); }
```

In `.page::before`, change `background: var(--tape-sage);` to `background: repeating-linear-gradient(45deg, rgba(122, 168, 150, .6) 0 6px, rgba(255, 255, 255, .45) 6px 12px);`.

- [ ] **Step 4: Pen underline on month headings**

Replace the `.month { ... }` rule's two body lines

```css
    font-family: var(--hand); font-weight: 500; font-size: 2.1rem; line-height: 1.2; color: var(--accent);
    border-bottom: 1px solid var(--rule); margin: 0 0 1.6rem; padding-bottom: .1rem;
```
with

```css
    font-family: var(--hand); font-weight: 500; font-size: 2.1rem; line-height: 1.2; color: var(--accent);
    margin: 0 0 1.6rem; padding-bottom: .55rem;
    background: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='300' height='8' viewBox='0 0 300 8' preserveAspectRatio='none'><path d='M0 4 C20 1 40 7 60 4 S100 1 120 4 S160 7 180 4 S220 1 240 4 S280 7 300 4' fill='none' stroke='%2343705f' stroke-opacity='.55' stroke-width='1.6' stroke-linecap='round'/></svg>") left bottom / 100% 8px no-repeat;
```

- [ ] **Step 5: Footer sign-off and 404 stamp**

`themes/kaikeru/layouts/partials/footer.html`:

```html
<footer class="foot">
    <svg class="doodle" width="46" height="46" viewBox="0 0 46 46" fill="none" stroke="#43705f" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><circle cx="23" cy="23" r="17"/><circle cx="23" cy="23" r="1.6" fill="#43705f"/><path d="M23 9 L26 23 L23 37 L20 23 Z"/><path d="M23 4 v3 M23 39 v3 M4 23 h3 M39 23 h3"/></svg>
    <p class="sign">thanks for stopping by</p>
    <p class="copy">{{ .Site.Copyright }} &middot; Made with 💖</p>
</footer>
```

`themes/kaikeru/layouts/404.html` (add the stamp as the first child of the article):

```html
{{ define "main" }}
<article class="page">
    <div class="stamp" aria-hidden="true">LOST?</div>
    <h1>Page not found</h1>
    <div class="content">
        <p>This page isn't in the notebook.</p>
    </div>
    <p class="meta"><a href="{{ "/" | relURL }}">&larr; back to the entries</a></p>
</article>
{{ end }}
```

Append to `style.css`:

```css
/* Footer sign-off */
.foot .doodle { display: block; margin: 0 auto .2rem; }
.foot .sign { font-size: 1.7rem; line-height: 1.2; }
.foot .copy { font-size: 1.1rem; color: #6b625a; }

/* 404 stamp */
.stamp {
    display: inline-block; border: 4px double #b0493f; border-radius: 6px; color: #b0493f; opacity: .85;
    font-family: var(--hand); font-weight: 700; font-size: 2.4rem; letter-spacing: 3px; line-height: 1.2;
    padding: .1rem 1.1rem; transform: rotate(-8deg); margin-bottom: .6rem;
}
@media (prefers-reduced-motion: reduce) { .stamp { transform: none; } }
```

In the existing `@media print` block change `.side, .foot, .pager { display: none; }` to `.side, .foot, .pager, .stamp { display: none; }` and add `body::before { display: none; } body { background-image: none; }` inside it.

- [ ] **Step 6: Contrast check on the textured paper**

```bash
python3 - <<'EOF'
def lum(h):
    h=h.lstrip('#'); r,g,b=[int(h[i:i+2],16)/255 for i in (0,2,4)]
    f=lambda c: c/12.92 if c<=0.03928 else ((c+0.055)/1.055)**2.4
    return 0.2126*f(r)+0.7152*f(g)+0.0722*f(b)
def ratio(a,b):
    la,lb=sorted([lum(a),lum(b)],reverse=True); return (la+0.05)/(lb+0.05)
# worst-case grain: 9% of (115,92,64) over paper #faf8f3 -> about #eeeae1; vignette adds a little more
for name,fg,bg in [
  ('accent/paper','#43705f','#faf8f3'),('accent/worst grain','#43705f','#eae6dc'),
  ('accent/surface','#43705f','#fffdf8'),('ink/worst grain','#2f2a25','#eae6dc'),
  ('copy/worst grain','#6b625a','#eae6dc'),('accent/yellow','#43705f','#fbf1cf'),
  ('updated/yellow','#6b5d38','#fbf1cf'),('margin/yellow','#5d4528','#fbf1cf')]:
    print(f"{name:20s} {ratio(fg,bg):.2f}")
EOF
```
Expected: every ratio at least `4.5`. If any fails, darken that foreground (or lower the grain alpha `.09` to `.06`) until it passes, and record the change in the commit message.

- [ ] **Step 7: Build and verify**

```bash
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"
grep -c 'class=stamp' public/404.html; grep -c 'thanks for stopping by' public/index.html
grep -c 'feTurbulence' public/css/style.css
```
Expected: no WARN/ERROR; `1`, `1`, `1`.

Run `hugo server -D` and, with the Chrome tools, load `/`, a post and `/nope/`. Because the window cannot be resized, test narrow widths with fixed-width iframes as in the earlier redesign work. Confirm: grain is visible but faint, the vignette darkens the edges slightly, three tape patterns cycle on notes, month headings have the wobbly underline, the footer shows the compass and sign-off, the 404 shows the tilted red stamp, nothing overlaps and there is no horizontal scroll at 390, 1100 and 1500px.

- [ ] **Step 8: Commit**

```bash
git add -A
git commit -m "Cozy look: grain, vignette, patterned tape, pen underline, footer compass, 404 stamp"
```

---

### Task 2: "Currently" sticky note

**Files:**
- Create: `data/currently.toml`, `themes/kaikeru/layouts/partials/currently-data.html`, `themes/kaikeru/layouts/partials/currently.html`
- Modify: `config.toml`, `themes/kaikeru/layouts/partials/header.html`, `themes/kaikeru/layouts/_default/baseof.html`, `themes/kaikeru/static/css/style.css`, `.github/workflows/deploy.yml`

**Interfaces:**
- Consumes: Task 1 tokens (`--accent`, `--tape-rose`).
- Produces: `partials/currently-data.html` (no arguments needed; returns `dict "reading" (dict "title" "author" "link" or empty) "working_on" string "updated" string`); `partials/currently.html` taking a class string (`"currently--side"` or `"currently--foot"`); CSS classes `.currently`, `.currently--side`, `.currently--foot`.

- [ ] **Step 1: Confirm there is no note yet (expected to fail)**

```bash
hugo --gc --minify >/dev/null 2>&1; grep -c 'aria-label=Currently' public/index.html
```
Expected: `0`.

- [ ] **Step 2: Data file and config**

Create `data/currently.toml`:

```toml
# Shown in the "currently" sticky note.
# `reading` is only used if the Goodreads fetch fails; leave it empty to hide the line then.
reading = ""
working_on = "this website"
updated = "Oct 2026"
```

In `config.toml`, under `[params]` add:

```toml
  goodreadsShelf = "https://www.goodreads.com/review/list_rss/38312672?shelf=currently-reading"
```
and add a Goodreads link to the social menu, between the Github and Linkedin entries (the header lowercases names, so it shows as `goodreads ↗`):

```toml
[[menu.social]]
  url = "https://www.goodreads.com/user/show/38312672-kyle"
  name = "Goodreads"
  weight = 2
```
Then append at the end of the file:

```toml

# Re-fetch remote resources (the Goodreads feed) at most hourly in local rebuilds.
[caches.getresource]
  maxAge = "1h"
```

- [ ] **Step 3: The data partial**

Create `themes/kaikeru/layouts/partials/currently-data.html`:

```
{{- $data := site.Data.currently | default dict -}}
{{- $reading := dict -}}
{{- with site.Params.goodreadsShelf -}}
  {{- with try (resources.GetRemote .) -}}
    {{- with .Err -}}
      {{- warnf "currently: Goodreads fetch failed: %s" . -}}
    {{- else with .Value -}}
      {{- $feed := .Content | transform.Unmarshal -}}
      {{- $items := $feed.channel.item -}}
      {{- if reflect.IsMap $items -}}{{- $items = slice $items -}}{{- end -}}
      {{- with index $items 0 -}}
        {{- $title := .title | replaceRE `\s*\([^)]*\)\s*$` "" -}}
        {{- $author := .author_name | replaceRE `\s+` " " -}}
        {{- $reading = dict "title" $title "author" $author "link" (printf "https://www.goodreads.com/book/show/%v" .book_id) -}}
      {{- end -}}
    {{- else -}}
      {{- warnf "currently: Goodreads returned no data" -}}
    {{- end -}}
  {{- end -}}
{{- end -}}
{{- if not $reading -}}
  {{- with $data.reading -}}{{- $reading = dict "title" . -}}{{- end -}}
{{- end -}}
{{- return (dict "reading" $reading "working_on" ($data.working_on | default "") "updated" ($data.updated | default "")) -}}
```

- [ ] **Step 4: The note partial and its placement**

Create `themes/kaikeru/layouts/partials/currently.html` (the context `.` is the class string):

```html
{{ $class := . }}
{{ $c := partialCached "currently-data.html" site }}
{{ if or $c.reading $c.working_on }}
<aside class="currently {{ $class }}" aria-label="Currently">
    <b class="currently-title">currently</b>
    {{ with $c.reading }}
    <span class="currently-line">reading: {{ if .link }}<a href="{{ .link }}" rel="noopener">{{ .title }}</a>{{ else }}{{ .title }}{{ end }}{{ with .author }} by {{ . }}{{ end }}</span>
    {{ end }}
    {{ with $c.working_on }}<span class="currently-line">working on: {{ . }}</span>{{ end }}
    {{ with $c.updated }}<span class="currently-updated">updated {{ . }}</span>{{ end }}
</aside>
{{ end }}
```

In `partials/header.html`, add before the closing `</header>`:

```html
    {{ partial "currently.html" "currently--side" }}
```

In `_default/baseof.html`, add between the `</main>` line and the footer partial:

```html
            {{- partial "currently.html" "currently--foot" -}}
```

- [ ] **Step 5: Styles**

In `style.css`, in the `@media (max-width: 1100px)` block change `.foot { grid-column: 1; grid-row: 3; }` to `.foot { grid-column: 1; grid-row: 4; }`. Then append:

```css
/* "Currently" sticky note */
.currently {
    position: relative; margin-top: 1.6rem; padding: .8rem .8rem .9rem; background: #fbf1cf; border: 1px solid #ecdfa8;
    box-shadow: 1px 2px 4px rgba(0, 0, 0, .12); transform: rotate(-2.2deg);
    font-family: var(--hand); font-size: 1.2rem; line-height: 1.3; color: var(--ink);
}
.currently::before { content: ""; position: absolute; top: -8px; left: 2.6rem; width: 46px; height: 15px; background: var(--tape-rose); transform: rotate(3deg); }
.currently-title { display: block; color: var(--accent); font-size: 1.45rem; margin-bottom: .25rem; }
.currently-line { display: block; margin-top: .2rem; }
.currently a { color: var(--accent); text-decoration: none; }
.currently a:hover { text-decoration: underline; }
.currently-updated { display: block; margin-top: .45rem; font-size: 1rem; color: #6b5d38; }
.currently--foot { display: none; }
@media (max-width: 1100px) {
    .currently--side { display: none; }
    .currently--foot { display: block; grid-column: 1; grid-row: 3; justify-self: center; width: min(100%, 320px); margin: 2rem 0 0; }
}
@media (prefers-reduced-motion: reduce) { .currently { transform: none; } }
```
In the `@media print` block, change the first selector line to also hide `.currently`: `.side, .foot, .pager, .stamp, .currently { display: none; }`.

- [ ] **Step 6: Daily rebuild**

In `.github/workflows/deploy.yml`, change the `on:` block to:

```yaml
on:
  push:
    branches: [master]
  schedule:
    - cron: "17 11 * * *"
  workflow_dispatch:
```
(Scheduled runs use the default branch. GitHub pauses schedules after about 60 days with no repository activity; a push or manual run resumes it.)

- [ ] **Step 7: Verify every Goodreads outcome**

```bash
S=/private/tmp/claude-501/-Users-kyle-repositories-blog/a7786d7e-8679-47bf-97d3-b291836f3891/scratchpad
cd /Users/kyle/repositories/blog
# (a) live
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"
grep -o 'currently-line>reading: <a href=[^>]*>[^<]*</a> by [^<]*' public/index.html | head -1
grep -o 'href=https://www.goodreads.com/book/show/[0-9]*' public/index.html | head -1
grep -o 'href=https://www.goodreads.com/user/show/38312672-kyle[^>]*>goodreads' public/index.html | head -1
# (b) unresolvable host -> warning, no reading line
printf '[params]\n  goodreadsShelf = "https://goodreads.invalid/x"\n' > $S/bad-host.toml
rm -rf resources; hugo --gc --minify --config config.toml,$S/bad-host.toml 2>&1 | grep -E "WARN|ERROR"
grep -c 'reading:' public/index.html; grep -c 'working on: this website' public/index.html
# (c) 404 from Goodreads + data-file fallback
printf '[params]\n  goodreadsShelf = "https://www.goodreads.com/review/list_rss/0?shelf=currently-reading"\n' > $S/bad-id.toml
cp data/currently.toml $S/currently.bak
sed -i '' 's/^reading = ""/reading = "Fallback Book"/' data/currently.toml
rm -rf resources; hugo --gc --minify --config config.toml,$S/bad-id.toml 2>&1 | grep -E "WARN|ERROR"
grep -o 'reading: Fallback Book' public/index.html | head -1
# (d) nothing to show -> no aside at all
printf 'reading = ""\nworking_on = ""\nupdated = ""\n' > data/currently.toml
rm -rf resources; hugo --gc --minify --config config.toml,$S/bad-id.toml 2>&1 | grep -E "WARN|ERROR"
grep -c 'aria-label=Currently' public/index.html
cp $S/currently.bak data/currently.toml; rm -rf resources; git status --short data/
```
Expected: (a) prints the "reading: ... by ..." markup, then a `href=https://www.goodreads.com/book/show/<id>` book link, then the `goodreads` social link to your profile (`/user/show/38312672-kyle`), and no WARN/ERROR. (b) one `WARN ... Goodreads fetch failed` line and no ERROR; `0` reading lines; `1` for the working-on line. (c) a `WARN ... returned no data` line, no ERROR, and `reading: Fallback Book`. (d) warnings only, and `0` asides. The last command shows `data/currently.toml` restored (it is untracked, so `git status` lists it as `??` with the original content; confirm with `cat data/currently.toml`).

Then run `hugo server -D` and check, via iframes at 1100 and 1101px: at 1101px exactly one note is visible in the left column under the social links, tilted, with a pink tape strip and the book as a link; at 1100px exactly one note is visible above the footer and none in the left column. Count visible notes with `[...document.querySelectorAll('.currently')].filter(e=>e.offsetParent).length` and expect `1` at both widths.

- [ ] **Step 8: Commit**

```bash
git add -A
git commit -m "Add Goodreads-fed 'currently' note with data-file fallback and daily rebuild"
```

---

### Task 3: Margin notes

**Files:**
- Create: `themes/kaikeru/layouts/shortcodes/note.html`
- Modify: `themes/kaikeru/static/css/style.css`

**Interfaces:**
- Consumes: Task 1 tokens and tape colors.
- Produces: shortcode `{{< note >}}text{{< /note >}}` and class `.margin-note`.

- [ ] **Step 1: Write the shortcode**

Create `themes/kaikeru/layouts/shortcodes/note.html`:

```html
<aside class="margin-note">{{ .Page.RenderString (dict "display" "inline") .Inner }}</aside>
```

- [ ] **Step 2: Styles**

Append to `style.css`:

```css
/* Margin notes: inline on narrow screens, hanging in the right-hand slot on wide ones */
.margin-note {
    position: relative; display: block; margin: 0 0 1.1rem; padding: .7rem .9rem;
    background: #fbf1cf; border: 1px solid #ecdfa8; box-shadow: 1px 2px 4px rgba(0, 0, 0, .12);
    font-family: var(--hand); font-size: 1.3rem; line-height: 1.3; color: #5d4528; transform: rotate(1.2deg);
}
.margin-note::before { content: ""; position: absolute; top: -8px; left: 1.4rem; width: 46px; height: 15px; background: var(--tape-sage); transform: rotate(-3deg); }
.content::after { content: ""; display: block; clear: right; }
@media (min-width: 1280px) {
    /* page padding-right is 3.2rem; hang the note 1.6rem past the page edge */
    .margin-note { float: right; clear: right; width: 190px; margin: 0 calc(-190px - 3.2rem - 1.6rem) 1rem 1rem; transform: rotate(2deg); }
    .margin-note::before { display: none; }
    .margin-note::after { content: ""; position: absolute; right: 100%; top: 50%; width: 1.6rem; border-top: 2px solid rgba(93, 69, 40, .45); }
}
@media (prefers-reduced-motion: reduce) { .margin-note { transform: none; } }
@media print { .margin-note { float: none; width: auto; margin: 0 0 1rem; transform: none; } .margin-note::after { display: none; } }
```
(The floated note's right edge sits at 190px + 1.6rem past the page edge, so at 1280px it ends well inside the window.)

- [ ] **Step 3: Temporary sample post**

```bash
cat > content/post/zz-margin-sample.md <<'EOF'
---
title: "Margin note sample"
date: 2026-10-01
tags: [sample]
draft: false
---
First paragraph of ordinary text so there is something to sit beside.

{{< note >}}this took me **three** tries!{{< /note >}}

Second paragraph with a short note beside the one above it. Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.

{{< note >}}A much longer margin note that is taller than the paragraph next to it, to check that the page grows to contain it instead of letting it spill out of the bottom of the sheet. It goes on for a while so that it wraps over many lines in the narrow column.{{< /note >}}

Short last paragraph.
EOF
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"; grep -o '<aside class=margin-note>[^<]*' public/post/zz-margin-sample/index.html | head -2; grep -c '<aside class=margin-note><p>' public/post/zz-margin-sample/index.html
```
Expected: no WARN/ERROR; two `margin-note` asides whose text is plain (with `<strong>` for the bold word); the last count is `0` (no nested paragraph).

- [ ] **Step 4: Measure at the breakpoint**

With `hugo server -D` and the Chrome tools, run in the page this script on `/post/zz-margin-sample/`, which opens fixed-width iframes and reports geometry:

```js
const out=[]; for (const w of [1279,1280,1500]){ const f=document.createElement('iframe'); f.style.cssText='position:fixed;left:0;top:0;width:'+w+'px;height:900px;border:0;visibility:hidden'; f.src='/post/zz-margin-sample/'; document.body.appendChild(f); await new Promise(r=>f.onload=r); await new Promise(r=>setTimeout(r,200)); const d=f.contentDocument, win=f.contentWindow; const n=[...d.querySelectorAll('.margin-note')]; const p=d.querySelector('.page').getBoundingClientRect(); out.push({w, floats:n.map(e=>getComputedStyle(e).float), maxRight:Math.round(Math.max(...n.map(e=>e.getBoundingClientRect().right))), pageRight:Math.round(p.right), pageBottom:Math.round(p.bottom), noteBottoms:n.map(e=>Math.round(e.getBoundingClientRect().bottom)), hscroll:d.documentElement.scrollWidth>w}); f.remove(); } JSON.stringify(out)
```
Expected: at 1279 both `float` are `none`, notes are inside the page (`maxRight` at most `pageRight`); at 1280 and 1500 both `float` are `right`, `maxRight` is greater than `pageRight` and at most `w - 16`, every note bottom is at most `pageBottom` (the long note does not spill out), and `hscroll` is `false` everywhere. Take a screenshot at 1500 to confirm the notes sit level with their paragraphs and the connector line shows.

- [ ] **Step 5: Remove the sample and commit**

```bash
rm content/post/zz-margin-sample.md
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"; git status --short
git add -A
git commit -m "Add margin note shortcode"
```
Expected: no WARN/ERROR; `git status` before `add` shows only the theme files (no `zz-margin-sample`).

---

### Task 4: Optional photos on posts and notes

**Files:**
- Modify: `themes/kaikeru/layouts/_default/single.html`, `themes/kaikeru/layouts/partials/list-item.html`, `themes/kaikeru/static/css/style.css`

**Interfaces:**
- Consumes: Task 1 tokens.
- Produces: front matter `image` (a `/path` under `static/` or a full URL) and `image_alt`; classes `.photo`, `.note-photo`, `.has-photo`.

- [ ] **Step 1: Post markup**

In `single.html`, insert after the `<h1>{{ .Title }}</h1>` line:

```html
    {{ with .Params.image }}
    {{ $src := . }}{{ if hasPrefix $src "/" }}{{ $src = $src | relURL }}{{ end }}
    <figure class="photo">
        <img src="{{ $src }}" alt="{{ $.Params.image_alt }}" loading="lazy">
        {{ with $.Params.image_alt }}<figcaption>{{ . }}</figcaption>{{ end }}
    </figure>
    {{ end }}
```

- [ ] **Step 2: Note markup**

In `partials/list-item.html` change the first line to

```html
<li class="note{{ if .Params.image }} has-photo{{ end }}">
```
and insert inside the `<a>`, before the `note-meta` span:

```html
        {{ with .Params.image }}{{ $src := . }}{{ if hasPrefix $src "/" }}{{ $src = $src | relURL }}{{ end }}<span class="note-photo"><img src="{{ $src }}" alt="" loading="lazy"></span>{{ end }}
```

- [ ] **Step 3: Styles**

Append to `style.css`:

```css
/* Taped photos */
.photo {
    position: relative; margin: 0 0 1.8rem; padding: 8px 8px 10px; background: #fff; border: 1px solid var(--line);
    box-shadow: 0 1px 5px rgba(0, 0, 0, .25); transform: rotate(1.4deg);
}
.photo::before { content: ""; position: absolute; top: -10px; left: 42%; width: 70px; height: 20px; background: radial-gradient(rgba(255, 255, 255, .75) 1.8px, transparent 2.2px) 0 0 / 9px 9px, var(--tape-sand); transform: rotate(-4deg); }
.photo img { display: block; width: 100%; height: auto; }
.photo figcaption { font-family: var(--hand); font-size: 1.3rem; line-height: 1.3; color: #5d554d; padding: .4rem .2rem 0; }
.note-photo { display: block; flex: none; margin: -.1rem .2rem .6rem; padding: 5px; background: #fff; box-shadow: 0 1px 3px rgba(0, 0, 0, .25); transform: rotate(1.4deg); }
.note-photo img { display: block; width: 100%; aspect-ratio: 3 / 2; object-fit: cover; }
.has-photo .note-title { -webkit-line-clamp: 2; line-clamp: 2; }
@media (prefers-reduced-motion: reduce) { .photo, .note-photo { transform: none; } }
@media print { .photo, .note-photo { transform: none; box-shadow: none; } .photo::before { display: none; } }
```

- [ ] **Step 4: Temporary samples**

```bash
python3 - <<'EOF'
import zlib, struct
w,h=1200,800
raw=b''.join(b'\x00'+bytes(v for x in range(w) for v in (int(120+90*x/w), int(150+60*y/h), int(130+80*(1-x/w)))) for y in range(h))
def chunk(t,d): c=struct.pack('>I',len(d))+t+d; return c+struct.pack('>I',zlib.crc32(t+d)&0xffffffff)
png=b'\x89PNG\r\n\x1a\n'+chunk(b'IHDR',struct.pack('>IIBBBBB',w,h,8,2,0,0,0))+chunk(b'IDAT',zlib.compress(raw,9))+chunk(b'IEND',b'')
import os; os.makedirs('static/images',exist_ok=True); open('static/images/zz-test.png','wb').write(png)
EOF
mk(){ printf -- "---\ntitle: \"$1\"\ndate: $2\ntags: [sample]\n$3draft: false\n---\nSample body text for the photo test. Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.\n" > content/post/$4.md; }
mk "Photo with caption and a deliberately long title that should clamp in the note" 2026-10-03 'image: "/images/zz-test.png"\nimage_alt: "Sample caption"\n' zz-photo-caption
mk "Photo decorative" 2026-10-02 'image: "/images/zz-test.png"\nimage_alt: ""\n' zz-photo-decor
mk "Photo as full URL" 2026-10-01 'image: "https://example.com/pic.jpg"\nimage_alt: "Remote"\n' zz-photo-url
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"
grep -o '<figure class=photo>.*</figure>' public/post/zz-photo-caption/index.html | head -c 300; echo
grep -o '<figure class=photo>.*</figure>' public/post/zz-photo-decor/index.html | head -c 300; echo
grep -o '<img src=[^ ]* alt=[^>]*>' public/post/zz-photo-url/index.html | head -1
grep -c 'class="note has-photo"\|class=note\ has-photo' public/index.html
grep -c 'note-photo' public/post/20200214-howilearneddevops/index.html
```
Expected: no WARN/ERROR; the caption post shows `<img src=/images/zz-test.png alt="Sample caption" ...>` and a `<figcaption>Sample caption</figcaption>`; the decorative post shows `alt=""` (or no alt value) and no figcaption; the URL post keeps `https://example.com/pic.jpg`; the home page has `3` photo notes; the existing "How I learned DevOps" post has no `note-photo` markup (count `0`), proving unchanged posts are untouched.

- [ ] **Step 5: Look**

With `hugo server -D` and the Chrome tools, view `/post/zz-photo-caption/` and the home page. Confirm: the post photo is a taped, tilted white-bordered print with its caption in handwriting; photo notes on the home page keep the same 4:5 size as other notes (compare `getBoundingClientRect` widths and heights of a photo note and a plain note: heights equal within 1px at the same width), the 2-line title clamp holds with the long title, and the excerpt fades instead of overflowing. Check via iframe at 390px: no horizontal scroll.

- [ ] **Step 6: Remove samples and commit**

```bash
rm content/post/zz-photo-*.md static/images/zz-test.png; rmdir static/images 2>/dev/null
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"; git status --short
git add -A
git commit -m "Add optional taped photos on posts and notes"
```
Expected: no WARN/ERROR; `git status` before `add` shows only the three theme files (no `zz-` files, no `static/images`).

---

## Self-Review

- **Spec coverage:** grain, vignette, tape patterns, pen underline, footer compass, 404 stamp (T1); currently note with Goodreads, fallbacks, placement, cache, daily schedule (T2); margin notes with wide/narrow/print behavior (T3); photos on posts and notes (T4). Reading time stays minutes (Global Constraints). Contrast on worst-case grain is checked in T1 Step 6, and the darker accent is the planned remedy. Verification matrix from the spec is covered across the four tasks (fallback paths in T2, 1279/1280/1500 in T3, photo variants in T4).
- **Placeholders:** none; the only conditional is T1 Step 6's explicit "darken or lower alpha if a ratio fails".
- **Consistency:** class names (`currently`, `currently--side`, `currently--foot`, `margin-note`, `photo`, `note-photo`, `has-photo`, `stamp`, `doodle`), partial names and the `--accent: #43705f` value are used identically across tasks; the 1100px footer grid row changes from 3 to 4 in T2 when the footer-position note is added at row 3.
