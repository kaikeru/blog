# Travel-Notebook Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the Bulma-based theme with a hand-written travel-notebook theme: left menu column, content centered at 680px, month-grouped askew "notes" on the home page, a plain taped page for posts.

**Architecture:** Same Hugo theme (`themes/kaikeru`), same content and config. One hand-written stylesheet with CSS custom properties replaces Bulma and Font Awesome. Layout is a three-column CSS grid (`1fr | 680px | 1fr`) with a sticky menu in the left slot that collapses above the content under 1100px. Home and list pages share one `notes.html` partial that groups pages by month. Tilt and tape are pure CSS via `:nth-child`.

**Tech Stack:** Hugo 0.168 (extended), plain CSS, self-hosted Lora and Caveat (woff2, latin subset), Hugo's Chroma (`syntax.css`, already present).

**Spec:** `docs/superpowers/specs/2026-10-20-notebook-redesign-design.md`

## Global Constraints

- Content column max width is `680px` on every page (home, post, resume, reading list, list pages, 404); content is centered in the viewport, not merely right of the menu.
- Fonts: Lora (400, 600, italic 400) for text, Caveat (500, 700) for short handwritten accents only, never body text. Self-hosted in `themes/kaikeru/static/fonts`; no third-party requests (no Google Fonts, no CDN CSS/JS).
- Colors: paper `#faf8f3`, surface `#fffdf8`, border `#e6dfd0`, rule `#e0d9c9`, ink `#2f2a25`, accent `#4a7a68` (darkened sage; must be at least 4.5:1 on both paper and surface), tapes sage `rgba(122,168,150,.55)`, sand `rgba(214,170,120,.55)`, rose `rgba(196,120,120,.45)`. No dot grid and no ruled lines.
- Notes: page-shaped (`aspect-ratio: 4/5`), 2 per row in the 680px column, 1 per row under 600px. Rotation between -1.5deg and +1.5deg, no vertical offsets. Straighten on hover/focus; no rotation under `prefers-reduced-motion`.
- Pagination: 24 notes per page. Home lists only the `post` section (not Resume or Reading List).
- Remove Bulma and Font Awesome. Keep `static/css/syntax.css` (Chroma onedark). No JavaScript.
- Do not change post content, `baseURL`, `static/CNAME`, or `.github/workflows/deploy.yml`. Twitter stays removed.
- Work on branch `notebook-redesign`, not `master`.

## Review Focus

- A post with no `tags` (JLPT) renders no tags row on its note or its page and nothing breaks.
- Resume and Reading List must not appear as notes on the home page, but must still render as plain pages with no date, read time, tags or prev/next.
- A very long title ("Free Website Hosting with Github and DevOps") must not overflow its 4:5 note; the title is clamped and the note keeps its shape.
- A month with a single post leaves the second grid cell empty and keeps its heading; month groups continue across pagination pages.
- Wide content (code blocks, tables, images) must scroll or shrink inside the 680px page on a phone with no horizontal page scroll.
- Between 1101px and about 1300px the sticky menu must not overlap the centered content; at 1100px and below the menu sits above the content.
- Menu marks the current page (`entries` for home and any post, `resume`, `reading list`); keyboard focus is visible on menu items and whole-note links.
- Tag pages (`/tags/devops/`) and the `/post/` section list use the same notes layout and a post's tag links resolve.
- The 404 page uses the new layout.

---

## File Structure

| File | Responsibility |
|---|---|
| `themes/kaikeru/static/fonts/*.woff2`, `README.md` | Self-hosted Lora and Caveat, with source/licence note |
| `themes/kaikeru/static/css/style.css` | Whole theme stylesheet (replaced; built up over the 3 tasks) |
| `themes/kaikeru/layouts/_default/baseof.html` | Page shell: `.site` grid wrapping header, main, footer |
| `themes/kaikeru/layouts/partials/head.html` | Meta, fonts preload, stylesheets (no Bulma/FA) |
| `themes/kaikeru/layouts/partials/header.html` | Left column: name, menu, social links |
| `themes/kaikeru/layouts/partials/footer.html` | Copyright line |
| `themes/kaikeru/layouts/_default/single.html` | Taped plain page: posts and plain pages |
| `themes/kaikeru/layouts/partials/list-item.html` | One note |
| `themes/kaikeru/layouts/partials/notes.html` | Month-grouped notes + pager (home and list pages) |
| `themes/kaikeru/layouts/partials/pagination.html` | Handwritten newer/older links |
| `themes/kaikeru/layouts/index.html`, `_default/list.html` | Home and list pages, both via `notes.html` |
| `themes/kaikeru/layouts/404.html` | Not-found page |
| `config.toml` | `[[menu.main]]` entries, `[pagination]` |
| `.gitignore` | Ignore `.superpowers/` |

---

### Task 1: Shell, fonts, menu and the plain page

**Files:**
- Create: `themes/kaikeru/static/fonts/` (5 woff2 + `README.md`)
- Replace: `themes/kaikeru/static/css/style.css`
- Modify: `themes/kaikeru/layouts/_default/baseof.html`, `partials/head.html`, `partials/header.html`, `partials/footer.html`, `_default/single.html`, `config.toml`, `.gitignore`

**Interfaces:**
- Consumes: nothing.
- Produces: CSS tokens (`--paper --surface --line --rule --ink --accent --tape-sage --tape-sand --tape-rose --col --hand --serif`), the `.site` grid with `.side` / `main` / `.foot`, the `.page` component (tape via `::before`), `.meta`, `.tags`, `.prevnext` classes. Task 2 builds notes on these tokens; Task 3 appends to the stylesheet.

- [ ] **Step 1: Branch, ignore the brainstorm folder, record the baseline**

```bash
cd /Users/kyle/repositories/blog
git checkout -b notebook-redesign
printf '.superpowers/\n' >> .gitignore
git add -A && git commit -q -m "Add spec/plan for notebook redesign; ignore .superpowers"
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"; grep -c 'bulma' public/index.html
```
Expected: no WARN/ERROR; the count prints `1` or more (baseline: Bulma is still linked). After this task it must print `0`.

- [ ] **Step 2: Download the fonts and verify them**

```bash
cd themes/kaikeru/static && mkdir -p fonts && cd fonts
B=https://cdn.jsdelivr.net/fontsource/fonts
curl -fsSL -o lora-400.woff2 $B/lora@latest/latin-400-normal.woff2
curl -fsSL -o lora-600.woff2 $B/lora@latest/latin-600-normal.woff2
curl -fsSL -o lora-400-italic.woff2 $B/lora@latest/latin-400-italic.woff2
curl -fsSL -o caveat-500.woff2 $B/caveat@latest/latin-500-normal.woff2
curl -fsSL -o caveat-700.woff2 $B/caveat@latest/latin-700-normal.woff2
file *.woff2; ls -l *.woff2
```
Expected: all five say `Web Open Font Format (Version 2)` and are each a few KB to ~50KB. If any `curl` fails or the file is HTML, stop and fix the URL before continuing.

Create `themes/kaikeru/static/fonts/README.md`:

```markdown
# Fonts

Self-hosted latin subsets, SIL Open Font License 1.1.

- Lora (400, 600, 400 italic) - Cyreal. Source: fontsource (`@fontsource/lora`).
- Caveat (500, 700) - Impallari Type. Source: fontsource (`@fontsource/caveat`).
```

- [ ] **Step 3: Replace the stylesheet**

Overwrite `themes/kaikeru/static/css/style.css` with:

```css
@font-face { font-family: "Lora"; font-weight: 400; font-style: normal; font-display: swap; src: url("../fonts/lora-400.woff2") format("woff2"); }
@font-face { font-family: "Lora"; font-weight: 600; font-style: normal; font-display: swap; src: url("../fonts/lora-600.woff2") format("woff2"); }
@font-face { font-family: "Lora"; font-weight: 400; font-style: italic; font-display: swap; src: url("../fonts/lora-400-italic.woff2") format("woff2"); }
@font-face { font-family: "Caveat"; font-weight: 500; font-style: normal; font-display: swap; src: url("../fonts/caveat-500.woff2") format("woff2"); }
@font-face { font-family: "Caveat"; font-weight: 700; font-style: normal; font-display: swap; src: url("../fonts/caveat-700.woff2") format("woff2"); }

:root {
    --paper: #faf8f3;
    --surface: #fffdf8;
    --line: #e6dfd0;
    --rule: #e0d9c9;
    --ink: #2f2a25;
    --accent: #4a7a68;
    --tape-sage: rgba(122, 168, 150, .55);
    --tape-sand: rgba(214, 170, 120, .55);
    --tape-rose: rgba(196, 120, 120, .45);
    --col: 680px;
    --hand: "Caveat", "Segoe Script", "Bradley Hand", cursive;
    --serif: "Lora", Georgia, "Times New Roman", serif;
}

*, *::before, *::after { box-sizing: border-box; }

body {
    margin: 0;
    background: var(--paper);
    color: var(--ink);
    font-family: var(--serif);
    font-size: 1.0625rem;
    line-height: 1.8;
}

a { color: var(--accent); }

/* Page shell: menu | content | empty, content centered in the window */
.site {
    display: grid;
    grid-template-columns: minmax(0, 1fr) minmax(0, var(--col)) minmax(0, 1fr);
    column-gap: 2rem;
    min-height: 100vh;
    padding: 2.5rem 1rem 1.5rem;
    align-content: start;
}
.side { grid-column: 1; grid-row: 1; align-self: start; position: sticky; top: 2.5rem; font-family: var(--hand); }
.site > main { grid-column: 2; grid-row: 1; min-width: 0; }
.foot { grid-column: 2; grid-row: 2; margin-top: 3rem; text-align: center; font-family: var(--hand); font-size: 1.3rem; color: var(--accent); }
.foot p { margin: 0; }

/* Menu */
.side .name { display: block; font-size: 2.6rem; font-weight: 700; line-height: 1; color: var(--ink); text-decoration: none; }
.side .rule { width: 40px; border: 0; border-top: 2px solid var(--accent); margin: .7rem 0 .9rem; }
.side ul { list-style: none; margin: 0; padding: 0; }
.side li { position: relative; padding-left: 1.1rem; font-size: 1.5rem; line-height: 1.9; }
.side a { color: var(--ink); text-decoration: none; }
.side a:hover { text-decoration: underline; text-decoration-thickness: 1px; }
.side a[aria-current="page"] { color: var(--accent); }
.side a[aria-current="page"]::before { content: "\25B8"; position: absolute; left: 0; }
.side .social { margin-top: .6rem; }
.side .social li { font-size: 1.2rem; color: var(--accent); }
.side .social a { color: var(--accent); }

/* Narrow screens and phones: menu becomes a row above the content */
@media (max-width: 1100px) {
    .site { grid-template-columns: minmax(0, var(--col)); justify-content: center; }
    .side { grid-column: 1; grid-row: 1; position: static; margin-bottom: 2rem; }
    .site > main { grid-column: 1; grid-row: 2; }
    .foot { grid-column: 1; grid-row: 3; }
    .side ul { display: flex; flex-wrap: wrap; column-gap: 1.3rem; }
    .side li { padding-left: 0; font-size: 1.4rem; line-height: 1.7; }
    .side a[aria-current="page"]::before { position: static; margin-right: .2rem; }
    .side .social { margin-top: 0; }
    .side .rule { display: none; }
}

/* The page: one plain sheet with a strip of tape */
.page {
    position: relative;
    background: var(--surface);
    border: 1px solid var(--line);
    padding: 2.8rem 3.2rem 2.2rem;
    box-shadow: 1px 3px 8px rgba(0, 0, 0, .1);
}
.page::before {
    content: ""; position: absolute; top: -11px; left: 3.5rem; width: 84px; height: 24px;
    background: var(--tape-sage); transform: rotate(-2.5deg);
}
.page h1 { font-size: 2.1rem; line-height: 1.25; font-weight: 600; margin: .2rem 0 1.4rem; }
.meta, .tags, .prevnext { font-family: var(--hand); color: var(--accent); }
.meta { font-size: 1.4rem; line-height: 1.4; margin: 0; }
.tags { font-size: 1.45rem; margin: 1.8rem 0 0; }
.tags a { color: var(--accent); text-decoration: none; margin-right: .9rem; }
.tags a:hover { text-decoration: underline; }
.prevnext { display: flex; justify-content: space-between; gap: 1rem; font-size: 1.4rem; line-height: 1.3; margin-top: 1.6rem; padding-top: .8rem; border-top: 1px solid var(--rule); }
.prevnext a { color: var(--accent); text-decoration: none; }
.prevnext a:hover { text-decoration: underline; }
.prevnext .later { margin-left: auto; text-align: right; }

/* Prose inside pages */
.content > :first-child { margin-top: 0; }
.content p, .content ul, .content ol, .content blockquote, .content table, .content pre { margin: 0 0 1.1rem; }
.content h2 { font-size: 1.45rem; line-height: 1.3; font-weight: 600; margin: 2rem 0 .7rem; }
.content h3 { font-size: 1.2rem; line-height: 1.3; font-weight: 600; margin: 1.6rem 0 .6rem; }
.content h4 { font-size: 1.05rem; font-weight: 600; margin: 1.4rem 0 .5rem; }
.content img { max-width: 100%; height: auto; }
.content hr { border: 0; border-top: 1px solid var(--rule); margin: 2rem 0; }
.content blockquote { margin-left: 0; padding-left: 1rem; border-left: 3px solid var(--tape-sand); font-style: italic; }
.content table { display: block; max-width: 100%; overflow-x: auto; border-collapse: collapse; }
.content th, .content td { border: 1px solid var(--line); padding: .35rem .7rem; text-align: left; }
.content :not(pre) > code { background: #f1ece0; padding: .1em .35em; border-radius: 3px; font-size: .9em; }
.content pre { background: #282c34; color: #abb2bf; padding: 1rem 1.1rem; border-radius: 3px; overflow-x: auto; font-size: .85rem; line-height: 1.6; }
.content pre code { background: none; padding: 0; color: inherit; }
.content .chroma { margin: 0 0 1.1rem; }

@media (max-width: 600px) {
    .page { padding: 2.2rem 1.2rem 1.6rem; }
    .page h1 { font-size: 1.7rem; }
    .page::before { left: 1.4rem; }
}
```

- [ ] **Step 4: Rewrite the shell and partials**

`themes/kaikeru/layouts/_default/baseof.html`:

```html
<!DOCTYPE html>
<html lang="{{ .Site.Language.Locale }}">
    {{- partial "head.html" . -}}
    <body>
        <div class="site">
            {{- partial "header.html" . -}}
            <main id="content">
            {{- block "main" . }}{{- end }}
            </main>
            {{- partial "footer.html" . -}}
        </div>
    </body>
</html>
```

`themes/kaikeru/layouts/partials/head.html`:

```html
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
    <title>{{ if not .IsHome }}{{ with .Title }}{{ . }} | {{ end }}{{ end }}{{ .Site.Title }}</title>
    <meta name="description"
        content="{{ with .Description }}{{ . }}{{ else }}{{ with .Summary }}{{ . | plainify | truncate 160 }}{{ else }}{{ .Site.Params.description }}{{end }}{{ end }}">
    <link rel="canonical" href="{{ .Permalink }}" />
    <link rel="preload" href="{{ "fonts/lora-400.woff2" | relURL }}" as="font" type="font/woff2" crossorigin>
    <link rel="preload" href="{{ "fonts/caveat-500.woff2" | relURL }}" as="font" type="font/woff2" crossorigin>
    <link rel="stylesheet" href="{{ "css/style.css" | relURL }}" />
    <link rel="stylesheet" href="{{ "css/syntax.css" | relURL }}" />
    {{ template "_internal/opengraph.html" . }}
</head>
```

`themes/kaikeru/layouts/partials/header.html`:

```html
{{ $current := . }}
<header class="side">
    <a class="name" href="{{ "/" | relURL }}">{{ .Site.Title }}</a>
    <hr class="rule">
    <nav aria-label="Main">
        <ul>
            {{ range .Site.Menus.main }}
            {{ $isCurrent := eq .URL $current.RelPermalink }}
            {{ if and (eq .Identifier "entries") (or $current.IsHome (eq $current.Section "post")) }}{{ $isCurrent = true }}{{ end }}
            <li><a href="{{ .URL }}"{{ if $isCurrent }} aria-current="page"{{ end }}>{{ lower .Name }}</a></li>
            {{ end }}
        </ul>
        <ul class="social">
            {{ range .Site.Menus.social }}
            <li><a href="{{ .URL }}" rel="me noopener">{{ lower .Name }} <span aria-hidden="true">&#8599;</span></a></li>
            {{ end }}
        </ul>
    </nav>
</header>
```

`themes/kaikeru/layouts/partials/footer.html`:

```html
<footer class="foot">
    <p>{{ .Site.Copyright }} &middot; Made with 💖</p>
</footer>
```

`themes/kaikeru/layouts/_default/single.html`:

```html
{{ define "main" }}
{{ $isPost := eq .Section "post" }}
<article class="page">
    {{ if $isPost }}
    <p class="meta"><time datetime="{{ .PublishDate.Format "2006-01-02" }}">{{ .PublishDate.Format "January 2, 2006" }}</time> &middot; {{ .ReadingTime }} min read</p>
    {{ end }}
    <h1>{{ .Title }}</h1>
    <div class="content">
        {{ .Content }}
    </div>
    {{ if $isPost }}
    {{ with .Params.tags }}
    <p class="tags">{{ range $i, $tag := . }}<a href="{{ printf "tags/%s/" ($tag | urlize) | relURL }}">{{ if eq $i 0 }}~{{ end }}{{ $tag }}</a>{{ end }}</p>
    {{ end }}
    <nav class="prevnext" aria-label="Entries">
        {{ with .NextInSection }}<a class="earlier" href="{{ .RelPermalink }}" rel="prev">&larr; {{ .Title }}</a>{{ end }}
        {{ with .PrevInSection }}<a class="later" href="{{ .RelPermalink }}" rel="next">{{ .Title }} &rarr;</a>{{ end }}
    </nav>
    {{ end }}
</article>
{{ end }}
```

- [ ] **Step 5: Menu config**

In `config.toml` replace the commented block

```toml
# [[menu.main]]
#   url = "/"
#   name = "Home"
#   weight = 1
```
with

```toml
[[menu.main]]
  identifier = "entries"
  url = "/"
  name = "Entries"
  weight = 1

[[menu.main]]
  identifier = "resume"
  url = "/resume/"
  name = "Resume"
  weight = 2

[[menu.main]]
  identifier = "reading-list"
  url = "/reading-list/"
  name = "Reading List"
  weight = 3
```
and replace the comment lines about Font Awesome icon names above `[[menu.social]]` with `# Social links shown under the main menu.`

- [ ] **Step 6: Build and verify**

```bash
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"
grep -c 'bulma\|fontawesome\|googleapis' public/index.html public/post/*/index.html
grep -o 'aria-current=page[^>]*>[a-z ]*' public/resume/index.html public/post/20200214-howilearneddevops/index.html
grep -o 'class=earlier[^>]*>[^<]*\|class=later[^>]*>[^<]*' public/post/20190313-usedockerbuild/index.html
```
Expected: no WARN/ERROR; every count is `0`; the resume file shows a `resume` current marker and the DevOps post shows `entries`; the Docker post (March 2019) has an `earlier` link to "JLPT" and a `later` link to "How I learned DevOps". If earlier/later are swapped, swap `NextInSection` and `PrevInSection` in `single.html`.

Then run `hugo server -D` and, with the Chrome tools, open `http://localhost:1313/post/20200214-howilearneddevops/` and `/resume/` at 1440px, 1200px and 390px wide. Confirm: the page is exactly centered (equal space left and right of the 680px column) at 1440px; the menu is in the left slot and does not overlap the page at 1200px; the menu is a row above the page at 390px with no horizontal scroll; tape, handwritten date, tags and dark code blocks look right.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "Notebook theme: shell, fonts, menu and plain page"
```

---

### Task 2: Home page notes

**Files:**
- Modify: `themes/kaikeru/layouts/index.html`, `_default/list.html`, `partials/list-item.html`, `partials/pagination.html`, `themes/kaikeru/static/css/style.css`, `config.toml`
- Create: `themes/kaikeru/layouts/partials/notes.html`

**Interfaces:**
- Consumes: Task 1 tokens and shell.
- Produces: `partials/notes.html` taking a `*Pager` (from `.Paginate`) and rendering month groups plus the pager; `partials/list-item.html` taking a Page and rendering one `<li class="note">`; CSS classes `.month`, `.notes`, `.note`, `.note-title`, `.note-excerpt`, `.note-tags`, `.pager`. Task 3 reuses `notes.html` in `list.html`.

- [ ] **Step 1: Confirm the home page is not yet notes (expected to fail)**

```bash
hugo --gc --minify >/dev/null 2>&1; grep -c 'class=note' public/index.html
```
Expected: `0`.

- [ ] **Step 2: Add pagination config**

Append to `config.toml`:

```toml
[pagination]
  pagerSize = 24
```

- [ ] **Step 3: Write the templates**

`themes/kaikeru/layouts/partials/list-item.html`:

```html
<li class="note">
    <a href="{{ .RelPermalink }}">
        <span class="note-meta"><time datetime="{{ .PublishDate.Format "2006-01-02" }}">{{ .PublishDate.Format "Jan 2" }}</time> &middot; {{ .ReadingTime }} min</span>
        <span class="note-title">{{ .Title }}</span>
        <span class="note-excerpt">{{ .Summary | plainify }}</span>
        {{ with .Params.tags }}
        <span class="note-tags">{{ range $i, $tag := first 3 . }}<span>{{ if eq $i 0 }}~{{ end }}{{ $tag }}</span>{{ end }}</span>
        {{ end }}
    </a>
</li>
```

`themes/kaikeru/layouts/partials/notes.html`:

```html
{{ range .Pages.GroupByDate "January 2006" }}
<section class="month-group">
    <h2 class="month">{{ .Key }}</h2>
    <ul class="notes">
        {{ range .Pages }}{{ partial "list-item.html" . }}{{ end }}
    </ul>
</section>
{{ end }}
{{ partial "pagination.html" . }}
```

`themes/kaikeru/layouts/partials/pagination.html`:

```html
{{ if gt .TotalPages 1 }}
<nav class="pager" aria-label="Pages">
    {{ if .HasPrev }}<a href="{{ .Prev.URL }}" rel="prev">&larr; newer entries</a>{{ else }}<span></span>{{ end }}
    <span>page {{ .PageNumber }} of {{ .TotalPages }}</span>
    {{ if .HasNext }}<a href="{{ .Next.URL }}" rel="next">older entries &rarr;</a>{{ else }}<span></span>{{ end }}
</nav>
{{ end }}
```

`themes/kaikeru/layouts/index.html`:

```html
{{ define "main" }}
{{ $posts := where site.RegularPages "Section" "post" }}
{{ partial "notes.html" (.Paginate $posts) }}
{{ end }}
```

- [ ] **Step 4: Append the notes CSS**

Append to `themes/kaikeru/static/css/style.css`:

```css
/* Home: month headings and notes */
.month {
    font-family: var(--hand); font-weight: 500; font-size: 2.1rem; line-height: 1.2; color: var(--accent);
    border-bottom: 1px solid var(--rule); margin: 0 0 1.6rem; padding-bottom: .1rem;
}
.month-group { margin-bottom: 2.4rem; }
.notes { list-style: none; margin: 0; padding: 0; display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 30px 28px; }
.note {
    position: relative; aspect-ratio: 4 / 5; background: var(--surface); border: 1px solid var(--line);
    box-shadow: 1px 2px 5px rgba(0, 0, 0, .1);
}
.note a { display: flex; flex-direction: column; height: 100%; padding: 1.5rem 1.3rem 1.1rem; color: inherit; text-decoration: none; overflow: hidden; }
.note::before { content: ""; position: absolute; top: -1px; width: 58px; height: 18px; z-index: 1; }
.note-meta { font-family: var(--hand); font-size: 1.3rem; line-height: 1.3; color: var(--accent); margin-top: .35rem; }
.note-title {
    font-size: 1.3rem; line-height: 1.3; font-weight: 600; margin-top: .1rem;
    display: -webkit-box; -webkit-line-clamp: 3; line-clamp: 3; -webkit-box-orient: vertical; overflow: hidden;
}
.note-excerpt {
    flex: 1; min-height: 0; overflow: hidden; margin-top: .6rem; font-size: .92rem; line-height: 1.65;
    -webkit-mask-image: linear-gradient(#000 70%, transparent); mask-image: linear-gradient(#000 70%, transparent);
}
.note-tags { font-family: var(--hand); font-size: 1.35rem; line-height: 1.3; color: var(--accent); padding-top: .5rem; }
.note-tags span { margin-right: .7rem; }

/* Askew: fixed repeating pattern, so it looks random but never changes */
.note:nth-child(5n+1) { transform: rotate(-1.4deg); }
.note:nth-child(5n+2) { transform: rotate(1.1deg); }
.note:nth-child(5n+3) { transform: rotate(-0.7deg); }
.note:nth-child(5n+4) { transform: rotate(1.5deg); }
.note:nth-child(5n+5) { transform: rotate(-1.2deg); }
.note:nth-child(3n+1)::before { left: 2rem; background: var(--tape-sage); transform: rotate(-3deg); }
.note:nth-child(3n+2)::before { left: 3.4rem; background: var(--tape-sand); transform: rotate(3deg); }
.note:nth-child(3n+3)::before { left: 1.3rem; background: var(--tape-rose); transform: rotate(-2deg); }

@media (prefers-reduced-motion: no-preference) {
    .note { transition: transform .15s ease, box-shadow .15s ease; }
}
.note:hover, .note:focus-within { transform: none; box-shadow: 2px 4px 10px rgba(0, 0, 0, .15); }

.pager { display: flex; justify-content: space-between; gap: 1rem; font-family: var(--hand); font-size: 1.5rem; color: var(--accent); margin-top: 1rem; }
.pager a { color: var(--accent); text-decoration: none; }
.pager a:hover { text-decoration: underline; }

@media (max-width: 600px) {
    .notes { grid-template-columns: minmax(0, 320px); justify-content: center; }
    .note:nth-child(5n+1), .note:nth-child(5n+2), .note:nth-child(5n+3), .note:nth-child(5n+4), .note:nth-child(5n+5) { transform: rotate(-0.6deg); }
    .note:nth-child(even) { transform: rotate(0.6deg); }
}
@media (prefers-reduced-motion: reduce) {
    .note, .note:nth-child(n) { transform: none; }
}
```

- [ ] **Step 5: Build and verify**

```bash
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"
grep -o 'class=note>' public/index.html | wc -l
grep -o '<h2 class=month>[^<]*' public/index.html
grep -o 'class=note-title>[^<]*' public/index.html
```
Expected: no WARN/ERROR; `4` notes; headings `February 2020`, `March 2019`, `February 2019`, `December 2018` in that order; the four note titles are exactly the four posts (no "Resume" or "Reading List" among them).

Create temporary sample posts (not committed) to exercise wrapping, multi-note months and pagination:

```bash
for i in $(seq 1 30); do d=$(date -v-${i}d +%Y-%m-%d); printf -- "---\ntitle: \"Sample entry $i with a deliberately long title to test clamping\"\ndate: $d\ntags: [one, two, three, four]\ndraft: false\n---\nLorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation.\n\nSecond paragraph of sample text.\n" > content/post/zz-sample-$i.md; done
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"; ls public/page/; grep -o 'class=note>' public/index.html | wc -l
```
Expected: `public/page/2` exists; the home page has `24` notes.

With `hugo server -D` and the Chrome tools, check `http://localhost:1313/` at 1440px, 1200px and 390px: two notes per row (one on phone), tilt is gentle and varied, titles clamp to 3 lines, the long excerpt fades, tags show at most 3, notes straighten on hover, page 2 links work, and month headings span the column. Then remove the samples:

```bash
rm content/post/zz-sample-*.md
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"; git status --short
```
Expected: no WARN/ERROR; only the intended theme/config changes show (no `zz-sample` files).

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "Notebook theme: month-grouped askew notes on the home page"
```

---

### Task 3: List pages, 404, accessibility pass

**Files:**
- Modify: `themes/kaikeru/layouts/_default/list.html`, `themes/kaikeru/layouts/404.html`, `themes/kaikeru/static/css/style.css`

**Interfaces:**
- Consumes: `partials/notes.html` (a `*Pager`), `.page`, `.month`, `.notes` from Tasks 1-2.
- Produces: finished theme; nothing later depends on it.

- [ ] **Step 1: Confirm list and 404 pages are not yet on the new layout (expected to fail)**

```bash
hugo --gc --minify >/dev/null 2>&1; grep -c 'class=note' public/tags/devops/index.html; grep -c 'Page not found' public/404.html
```
Expected: `0` and `1` or `0` (the 404 has the old layout; the list page has no notes).

- [ ] **Step 2: Write the templates**

`themes/kaikeru/layouts/_default/list.html`:

```html
{{ define "main" }}
<h1 class="list-title">{{ .Title }}</h1>
{{ with .Content }}<div class="content list-intro">{{ . }}</div>{{ end }}
{{ partial "notes.html" (.Paginate .Pages) }}
{{ end }}
```

`themes/kaikeru/layouts/404.html`:

```html
{{ define "main" }}
<article class="page">
    <h1>Page not found</h1>
    <div class="content">
        <p>This page isn't in the notebook.</p>
    </div>
    <p class="meta"><a href="{{ "/" | relURL }}">&larr; back to the entries</a></p>
</article>
{{ end }}
```

- [ ] **Step 3: Append the finishing CSS**

Append to `themes/kaikeru/static/css/style.css`:

```css
/* List pages */
.list-title { font-family: var(--hand); font-weight: 700; font-size: 2.6rem; line-height: 1.1; color: var(--ink); margin: 0 0 1.4rem; }
.list-intro { margin-bottom: 1.6rem; }

/* Keyboard focus */
a:focus-visible { outline: 2px solid var(--accent); outline-offset: 3px; border-radius: 2px; }
.note a:focus-visible { outline-offset: -4px; }

/* 404 link is handwritten */
.page .meta a { color: var(--accent); text-decoration: none; }
.page .meta a:hover { text-decoration: underline; }

/* Print: just the page */
@media print {
    .side, .foot, .pager { display: none; }
    .site { display: block; padding: 0; }
    .page, .note { box-shadow: none; transform: none; }
    .page::before, .note::before { display: none; }
}
```

- [ ] **Step 4: Contrast check**

Run this once (Python 3, no packages):

```bash
python3 - <<'EOF'
def lum(h):
    h=h.lstrip('#'); r,g,b=[int(h[i:i+2],16)/255 for i in (0,2,4)]
    f=lambda c: c/12.92 if c<=0.03928 else ((c+0.055)/1.055)**2.4
    return 0.2126*f(r)+0.7152*f(g)+0.0722*f(b)
def ratio(a,b):
    la,lb=sorted([lum(a),lum(b)],reverse=True); return (la+0.05)/(lb+0.05)
for fg,bg in [('#4a7a68','#faf8f3'),('#4a7a68','#fffdf8'),('#2f2a25','#fffdf8'),('#abb2bf','#282c34')]:
    print(fg,bg,round(ratio(fg,bg),2))
EOF
```
Expected: every ratio is at least `4.5`. If the accent fails, darken `--accent` in `style.css` until both accent rows pass, and note the new value in the commit message.

- [ ] **Step 5: Full build and browser pass**

```bash
hugo --gc --minify 2>&1 | grep -E "WARN|ERROR"
ls public/404.html public/tags/devops/index.html public/post/index.html
grep -o 'class=note>' public/tags/devops/index.html | wc -l
grep -c 'Page not found' public/404.html
```
Expected: no WARN/ERROR; all three files exist; `3` notes on the `devops` tag page (the static-site, Docker and "How I learned DevOps" posts all carry the `devops` tag); `1` for the 404 check.

With `hugo server -D` and the Chrome tools, at 1440px, 1200px, 1101px, 1100px and 390px check: home, a post (including the Free Website Hosting post for its long code blocks and `.gitignore` block, which must now be dark), JLPT (no tags), resume, reading list, `/tags/devops/`, `/post/`, and a missing URL. Confirm: nothing overlaps at 1101px, no horizontal page scroll anywhere, Tab key shows a visible focus ring on menu items and notes, and emulating `prefers-reduced-motion: reduce` removes the tilt.

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "Notebook theme: list pages, 404, focus and print styles"
```

---

## Self-Review

- **Spec coverage:** layout and centering (T1), fonts/colors/no Bulma (T1), home month groups, notes, tags, tilt, wrap, pagination (T2), post page, plain pages (T1), 404 (T3), list/tag pages (T3), accessibility, reduced motion, contrast, verification with sample posts (T2-T3), menu config and current-page marker (T1). Spec risk "older posts need basic element styles" is covered by the Prose block in T1 Step 3.
- **Placeholders:** none; the only conditional is the explicit accent-darkening fallback in T3 Step 4.
- **Consistency:** tokens (`--accent`, `--surface`, `--tape-*`, `--hand`, `--serif`), class names (`.page`, `.meta`, `.tags`, `.prevnext`, `.month`, `.notes`, `.note`, `.pager`) and partial names (`notes.html`, `list-item.html`, `pagination.html`) are used identically across tasks.
