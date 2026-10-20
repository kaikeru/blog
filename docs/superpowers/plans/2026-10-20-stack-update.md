# Blog Stack Update Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the Hugo blog build clean on current Hugo and deploy from this repo to GitHub Pages via GitHub Actions, with analytics and Twitter removed and Chroma replacing highlight.js.

**Architecture:** Keep Hugo and the in-repo `themes/kaikeru` theme. Upgrade Hugo first and fix whatever the build flags, then strip analytics/Twitter/highlight.js, then replace the Azure pipeline with a Pages workflow. There is no test suite; each task is verified by running `hugo` and inspecting `public/`.

**Tech Stack:** Hugo extended (current release), Bulma 0.7.4 and Font Awesome 5.7.2 (unchanged, loaded from CDN), Chroma (built into Hugo), GitHub Actions + GitHub Pages.

**Spec:** `docs/superpowers/specs/2026-10-20-stack-update-design.md`

## Global Constraints

- Keep Hugo, the `kaikeru` theme, existing content, and the existing look; no redesign.
- `baseURL = "https://www.kaikeru.com/"` and `static/CNAME` must be preserved.
- Analytics: removed entirely. Twitter social link: removed. highlight.js: removed, replaced by Hugo's built-in Chroma.
- Deploy: GitHub Actions to GitHub Pages from this repo; `azure-pipelines.yml` deleted.
- Do not edit post content except where a build error forces it.
- Work on branch `stack-update`, not `master`.

## Review Focus

- Fenced code blocks with unusual or missing languages (` ```.gitignore `, ` ```txt `, ` ```Dockerfile `, ` ```toml `) must render as code blocks, not error or vanish (Task 2 checks every one).
- `hugo server` is served at `localhost`; templates must not depend on the production `baseURL` (Task 1 checks the served pages).
- Posts without `tags` must not break the `single.html` / `list-item.html` tag check (Task 1 checks every post renders).
- The mobile nav burger currently depends on jQuery, which is never loaded, so it does nothing; it must toggle the menu with no external script (Task 1).
- The empty `404.html` must still produce a `404.html` in `public/` (Task 3 checks).
- The deploy workflow must succeed with the `CNAME` file present and `public/` ignored by git (Task 3 checks a branch run).

---

## File Structure

| File | Responsibility |
|---|---|
| `config.toml` | Site config: drop analytics, Twitter, highlight params; add `[markup.highlight]` |
| `themes/kaikeru/layouts/partials/head.html` | Remove analytics/hljs/twitter-cards includes; link syntax CSS |
| `themes/kaikeru/layouts/partials/footer.html` | Remove highlight.js script |
| `themes/kaikeru/layouts/partials/header.html` | Replace jQuery burger toggle with vanilla JS |
| `themes/kaikeru/layouts/_default/single.html`, `partials/list-item.html` | Fix tag check and any other deprecations the build reports |
| `themes/kaikeru/static/css/syntax.css` | Generated Chroma stylesheet (new) |
| `.github/workflows/deploy.yml` | Build and deploy to Pages (new) |
| `azure-pipelines.yml` | Deleted |
| `README.md` | Describe the new pipeline |

---

### Task 1: Upgrade Hugo and fix template breakage

**Files:**
- Modify: `config.toml`, `themes/kaikeru/layouts/**` (only what the build flags)

**Interfaces:**
- Consumes: nothing.
- Produces: a branch where `hugo` builds without errors; the installed Hugo version string, recorded for Task 3.

- [ ] **Step 1: Create the branch and install Hugo**

```bash
cd /Users/kyle/repositories/blog
git checkout -b stack-update
brew install hugo
hugo version
```
Expected: a line like `hugo v0.1xx.x+extended ... darwin/arm64`. Write the version number (e.g. `0.1xx.x`) down; Task 3 pins it.

- [ ] **Step 2: Run the baseline build and capture all errors and warnings**

```bash
hugo --gc --minify 2>&1 | tee /tmp/hugo-baseline.txt
```
Expected: likely FAIL or WARN. Read the full output; each error names a file and line.

- [ ] **Step 3: Fix the known suspects**

In `themes/kaikeru/layouts/partials/head.html` the `twitter_cards.html` and `google_analytics_async.html` internal templates were removed from recent Hugo. Delete those two includes now (Task 2 finishes the cleanup):

```html
    {{ template "_internal/opengraph.html" . }}
</head>
```
(i.e. replace the lines from `{{ template "_internal/twitter_cards.html" . }}` through the closing `{{ end }}` of the analytics `if` with nothing.)

In `themes/kaikeru/layouts/_default/single.html` and `partials/list-item.html`, replace the slice-vs-int comparison with a truthiness check:

```html
{{ if .Params.tags }}
```
in place of `{{ if gt .Params.tags 0}}` (both files).

In `single.html`, fix the category link, which has stray spaces inside the string, and the double slash in tag links:

```html
href="{{ "categories/" | relURL }}{{ $category | urlize }}"
```
```html
<a class="tag" href="{{ "tags/" | relURL }}{{ . | urlize }}">#{{ . }}</a>
```
(use the same tag link line in `list-item.html`).

Replace `.Site.LanguageCode` in `baseof.html` with `.Site.Language.LanguageCode` if the build reports it deprecated. Apply the same minimal rule to any other deprecation in `/tmp/hugo-baseline.txt` (`.Site.Author`, `.Site.BaseURL`, `.Data.Pages`, `.URL`, `.Scratch`): use the replacement the warning names.

In `config.toml`, remove the `[author]` table if the warning says it is unsupported (the theme reads `.Site.Params.Author`, which is set under `[params]`).

- [ ] **Step 4: Replace the jQuery burger toggle with vanilla JS**

In `themes/kaikeru/layouts/partials/header.html`, replace the whole `<script>...</script>` block with:

```html
    <script>
        document.querySelector(".navbar-burger").addEventListener("click", function () {
            this.classList.toggle("is-active");
            document.querySelector(".navbar-menu").classList.toggle("is-active");
        });
    </script>
```

- [ ] **Step 5: Rebuild until clean**

```bash
hugo --gc --minify
```
Expected: exits 0 and prints no `WARN` or `ERROR` lines. Repeat Step 3's rule for anything remaining.

- [ ] **Step 6: Check pages render**

```bash
ls public/index.html public/post/*/index.html public/resume/index.html public/reading-list/index.html
```
Expected: all four posts plus resume and reading-list exist.

Then run `hugo server -D` and open `http://localhost:1313/`, one post, and `/resume/`. Resize to mobile width and confirm the burger opens the menu.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "Upgrade Hugo and fix theme template deprecations"
```

---

### Task 2: Remove analytics and Twitter; switch to Chroma highlighting

**Files:**
- Modify: `config.toml`, `themes/kaikeru/layouts/partials/head.html`, `themes/kaikeru/layouts/partials/footer.html`
- Create: `themes/kaikeru/static/css/syntax.css`

**Interfaces:**
- Consumes: the clean-building branch from Task 1.
- Produces: a site with no third-party analytics/highlighting scripts and Chroma CSS classes on code blocks.

- [ ] **Step 1: Edit `config.toml`**

Remove these lines:

```toml
googleAnalytics = "UA-81904592-1"
```
```toml
  highlight = true
  highlightStyle = "atom-one-dark"
  highlightLanguages = ["java", "python", "yaml", "bash", "dockerfile"]
```
and the whole Twitter block:

```toml
[[menu.social]]
    url = "https://twitter.com/aKaikeru/"
    name = "Twitter"
    weight = 2
```
Change the copyright line to `copyright = "©️2018 - 2026 Kyle Wagner"` only if the existing line's start year is wrong; otherwise just bump the end year to `2026`. Fix the header comment to say "Hugo config for the kaikeru theme." Add at the end of the file:

```toml
[markup.highlight]
  noClasses = false
  style = "onedark"
```

- [ ] **Step 2: Remove highlight.js includes**

In `partials/head.html` delete:

```html
    <link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/10.1.1/styles/default.min.css">
```
and add, directly after the `style.css` link:

```html
    <link rel="stylesheet" href="{{ "css/syntax.css" | relURL }}" />
```
In `partials/footer.html` delete the line:

```html
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/10.1.1/highlight.min.js"></script>
```

- [ ] **Step 3: Generate the Chroma stylesheet**

```bash
hugo gen chromastyles --style=onedark > themes/kaikeru/static/css/syntax.css
head -5 themes/kaikeru/static/css/syntax.css
```
Expected: CSS rules such as `.chroma { ... }`.

- [ ] **Step 4: Rebuild and verify code blocks**

```bash
hugo --gc --minify
grep -c 'class="chroma"' public/post/*/index.html
grep -rn "UA-81904592\|google-analytics\|highlight.js\|twitter.com" public/ | head
```
Expected: the counts are non-zero for `20181228-...` and `20190313-...` posts (posts with fenced code), and the second grep prints nothing. (If the `20181228` post text still mentions Twitter in prose, that is content and is fine; the nav link must be gone.)

Then run `hugo server -D` and open the `freeStaticSite` and `useDockerBuild` posts. Visually confirm the `bash`, `yaml`, `toml`, `txt`, `Dockerfile`, and ` ```.gitignore ` blocks all render as styled code blocks with a dark background and readable text.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "Remove analytics and Twitter; use Hugo built-in highlighting"
```

---

### Task 3: Replace Azure Pipelines with a GitHub Pages workflow

**Files:**
- Create: `.github/workflows/deploy.yml`
- Delete: `azure-pipelines.yml`
- Modify: `README.md`

**Interfaces:**
- Consumes: the Hugo version recorded in Task 1 Step 1 (call it `X.Y.Z`).
- Produces: a workflow that deploys `public/` to Pages on push to `master` and on manual dispatch.

- [ ] **Step 1: Write the workflow**

Create `.github/workflows/deploy.yml`, replacing `X.Y.Z` with the version from Task 1:

```yaml
name: Deploy blog

on:
  push:
    branches: [master]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

env:
  HUGO_VERSION: X.Y.Z

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Hugo
        run: |
          wget -q -O "${{ runner.temp }}/hugo.deb" \
            "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb"
          sudo dpkg -i "${{ runner.temp }}/hugo.deb"
      - name: Build
        run: hugo --gc --minify
      - uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 2: Delete the Azure pipeline**

```bash
git rm azure-pipelines.yml
```

- [ ] **Step 3: Update `README.md`**

Replace the `## Build Pipeline` section with:

```markdown
## Build Pipeline
Pushes to `master` run `.github/workflows/deploy.yml`, which builds the site with Hugo and publishes it to GitHub Pages. The custom domain comes from `static/CNAME`. To preview locally, install Hugo and run `hugo server -D`.
```
Also change "Azure Pipelines add on ... published at the kaikeru/kaikeru.github.io repo" wording out of the Purpose section if it appears there.

- [ ] **Step 4: Verify the build output the workflow will publish**

```bash
hugo --gc --minify
ls public/CNAME public/robots.txt public/404.html public/index.html
```
Expected: all four files exist. (If `404.html` is missing, add content to `themes/kaikeru/layouts/404.html`, e.g. `{{ define "main" }}<section class="section"><h1 class="title">Page not found</h1><p><a href="/">Back to the blog</a></p></section>{{ end }}`, and rebuild.)

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "Replace Azure Pipelines with GitHub Pages workflow"
```

- [ ] **Step 6: Hand off the manual steps to the owner**

Do not push or change repo settings without the owner's go-ahead. Tell the owner to: set Pages source to "GitHub Actions" in repo settings; push the branch and run the workflow via `workflow_dispatch` to confirm it builds; after merging to `master` and a successful deploy, move the custom domain to this repo's Pages site, retire or archive `kaikeru/kaikeru.github.io`, and delete the old Azure pipeline and its GitHub token.

---

## Self-Review

- Spec coverage: Hugo upgrade (T1), config cleanup, analytics, Twitter, Chroma (T2), CI/deploy, README, CNAME preserved (T3), manual steps (T3 Step 6), verification (each task). External asset pinning: Bulma/Font Awesome are already versioned and https; `//cdnjs` protocol-relative URLs were removed with highlight.js, so nothing remains to change.
- Placeholders: `X.Y.Z` is the one deliberate fill-in, taken from `hugo version` output in Task 1 Step 1.
- Consistency: file names (`syntax.css`, `deploy.yml`) and the `stack-update` branch are used identically throughout.
