# Yinwang Style Blog Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Hugo-based Markdown blog with a Yinwang-inspired reading-focused theme, category/archive navigation, and GitHub Pages deployment.

**Architecture:** The site will be a custom Hugo project stored directly in the repository. Content will live in `content/`, templates in `layouts/`, site configuration in `hugo.toml`, static assets in `assets/` and `static/`, and deployment automation in `.github/workflows/`. The implementation will keep the first release Chinese-only while leaving configuration and content layout compatible with a later `content/en/` expansion.

**Tech Stack:** Hugo extended, Markdown, HTML templates, CSS, GitHub Actions, GitHub Pages

---

## File Structure

- Create: `hugo.toml` for Hugo site configuration, taxonomies, menu, output behavior, and future multilingual-ready defaults
- Create: `assets/css/main.css` for the site-wide reading-oriented theme
- Create: `layouts/_default/baseof.html` for the shared document shell and header/footer
- Create: `layouts/partials/head.html` for metadata, title handling, stylesheet pipeline, and viewport setup
- Create: `layouts/partials/header.html` for top navigation
- Create: `layouts/partials/post-list.html` for reusable post listing markup
- Create: `layouts/index.html` for the home page article list
- Create: `layouts/_default/single.html` for article rendering
- Create: `layouts/_default/list.html` for taxonomy and section list rendering
- Create: `layouts/archive/list.html` for year-grouped archives
- Create: `layouts/categories/terms.html` for the category index
- Create: `content/posts/hello-yinwang-style.md` for a sample post
- Create: `content/about/index.md` for the About page
- Create: `content/archive/_index.md` for the Archive page
- Create: `archetypes/default.md` for the Markdown post template
- Create: `static/.nojekyll` to avoid Jekyll interference if needed in generated output pipelines
- Create: `.github/workflows/hugo.yml` for build and GitHub Pages deployment
- Create: `README.md` for local writing, preview, and deployment instructions

### Task 1: Bootstrap Hugo Project

**Files:**
- Create: `hugo.toml`
- Create: `archetypes/default.md`
- Modify: `README.md`

- [ ] **Step 1: Write the failing configuration check**

```toml
# hugo.toml is intentionally missing before this step.
#
# The first verification expects `hugo` to fail because no site config exists yet.
```

- [ ] **Step 2: Run build to verify it fails**

Run: `hugo --gc --minify`
Expected: FAIL with an error indicating that no Hugo site configuration or layout/content structure is available yet.

- [ ] **Step 3: Write minimal Hugo configuration and archetype**

```toml
# hugo.toml
baseURL = "https://<username>.github.io/"
title = "Your Blog"
languageCode = "zh-cn"
defaultContentLanguage = "zh-cn"
hasCJKLanguage = true
enableRobotsTXT = true

[params]
  author = "Your Name"
  description = "A minimalist blog."

[taxonomies]
  category = "categories"
  tag = "tags"

[permalinks]
  posts = "/posts/:slugorfilename/"

[[menu.main]]
  name = "首页"
  pageRef = "/"
  weight = 1

[[menu.main]]
  name = "归档"
  pageRef = "/archive"
  weight = 2

[[menu.main]]
  name = "分类"
  pageRef = "/categories"
  weight = 3

[[menu.main]]
  name = "关于"
  pageRef = "/about"
  weight = 4
```

```md
---
title: "{{ replace .File.ContentBaseName `-` ` ` | title }}"
date: {{ .Date }}
draft: true
categories: []
tags: []
description: ""
---
```

```md
# README.md

## Development

Run `hugo server` for local preview.

## Build

Run `hugo --gc --minify` to generate the static site.
```

- [ ] **Step 4: Run build to verify the configuration is accepted**

Run: `hugo --gc --minify`
Expected: PASS with a generated `public/` directory, even though the site is still visually incomplete.

- [ ] **Step 5: Commit**

```bash
git add hugo.toml archetypes/default.md README.md
git commit -m "feat: bootstrap hugo site configuration"
```

### Task 2: Add Base Layout and Site Theme

**Files:**
- Create: `layouts/_default/baseof.html`
- Create: `layouts/partials/head.html`
- Create: `layouts/partials/header.html`
- Create: `assets/css/main.css`
- Test: local `hugo` build output

- [ ] **Step 1: Write the failing layout expectation**

```html
<!-- Expected layout contract after this task:
1. Every page gets a shared <head>.
2. Every page gets a header nav.
3. A main content block exists.
4. A stylesheet is loaded from assets/css/main.css.
-->
```

- [ ] **Step 2: Run build to verify the theme is still missing**

Run: `hugo --gc --minify`
Expected: FAIL or render with missing template warnings/empty pages because shared templates do not exist yet.

- [ ] **Step 3: Write the base templates and CSS**

```html
<!-- layouts/_default/baseof.html -->
<!DOCTYPE html>
<html lang="{{ site.Language.Lang }}">
  <head>
    {{ partial "head.html" . }}
  </head>
  <body>
    {{ partial "header.html" . }}
    <main class="page-shell">
      {{ block "main" . }}{{ end }}
    </main>
  </body>
</html>
```

```html
<!-- layouts/partials/head.html -->
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{{ if .IsHome }}{{ site.Title }}{{ else }}{{ .Title }} | {{ site.Title }}{{ end }}</title>
<meta name="description" content="{{ with .Description }}{{ . }}{{ else }}{{ site.Params.description }}{{ end }}">
{{ $css := resources.Get "css/main.css" | minify | fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}">
```

```html
<!-- layouts/partials/header.html -->
<header class="site-header">
  <div class="site-shell">
    <a class="site-title" href="{{ "/" | relURL }}">{{ site.Title }}</a>
    <nav class="site-nav" aria-label="Main navigation">
      {{ range site.Menus.main }}
        <a href="{{ .URL }}">{{ .Name }}</a>
      {{ end }}
    </nav>
  </div>
</header>
```

```css
/* assets/css/main.css */
:root {
  --bg: #f8f6f1;
  --text: #222222;
  --muted: #6f6a61;
  --link: #1f4b7a;
  --border: #d8d0c4;
  --max-width: 760px;
}

html {
  font-size: 18px;
}

body {
  margin: 0;
  background: var(--bg);
  color: var(--text);
  font-family: "Iowan Old Style", "Songti SC", "Noto Serif CJK SC", serif;
  line-height: 1.85;
}

a {
  color: var(--link);
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

.site-shell,
.page-shell {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: 0 1.25rem;
}

.site-header {
  padding: 2rem 0 1rem;
}

.site-title {
  color: var(--text);
  font-size: 1.5rem;
  font-weight: 600;
}

.site-nav {
  display: flex;
  gap: 1rem;
  margin-top: 0.75rem;
  font-size: 0.95rem;
}

main {
  padding-bottom: 4rem;
}
```

- [ ] **Step 4: Run build to verify the shared shell works**

Run: `hugo --gc --minify`
Expected: PASS with generated HTML that includes the shared `<header>` and stylesheet link.

- [ ] **Step 5: Commit**

```bash
git add layouts/_default/baseof.html layouts/partials/head.html layouts/partials/header.html assets/css/main.css
git commit -m "feat: add shared hugo layout and base styling"
```

### Task 3: Implement Home Page and Reusable Post List

**Files:**
- Create: `layouts/partials/post-list.html`
- Create: `layouts/index.html`
- Create: `content/posts/hello-yinwang-style.md`
- Test: home page ordering and metadata rendering

- [ ] **Step 1: Write the failing home page expectation**

```html
<!-- Expected behavior:
1. Home page lists regular pages from the posts section.
2. Posts are ordered by date descending.
3. Each row renders title, date, and categories.
-->
```

- [ ] **Step 2: Run local preview check before implementation**

Run: `hugo --gc --minify`
Expected: FAIL the behavior expectation because the home page template and sample content do not exist yet.

- [ ] **Step 3: Write the post list partial, index template, and sample post**

```html
<!-- layouts/partials/post-list.html -->
{{ $pages := . }}
<section class="post-list">
  {{ range $pages.ByDate.Reverse }}
    <article class="post-row">
      <h2 class="post-row-title"><a href="{{ .RelPermalink }}">{{ .Title }}</a></h2>
      <p class="post-row-meta">
        <time datetime="{{ .Date.Format `2006-01-02` }}">{{ .Date.Format "2006-01-02" }}</time>
        {{ with .Params.categories }}
          <span> / {{ delimit . " / " }}</span>
        {{ end }}
      </p>
      {{ with .Params.description }}
        <p class="post-row-summary">{{ . }}</p>
      {{ end }}
    </article>
  {{ end }}
</section>
```

```html
<!-- layouts/index.html -->
{{ define "main" }}
  {{ $posts := where site.RegularPages "Section" "posts" }}
  {{ partial "post-list.html" $posts }}
{{ end }}
```

```md
---
title: "像阴王风格一样开始写作"
date: 2026-06-09T12:00:00+08:00
draft: false
categories: ["随笔"]
tags: ["hugo", "blog"]
description: "这是一篇示例文章，用来验证首页列表、分类和归档。"
---

这是站点的第一篇示例文章。

后续你只需要继续往 `content/posts/` 添加 Markdown 文件。
```

- [ ] **Step 4: Run build to verify the home list renders**

Run: `hugo --gc --minify`
Expected: PASS and the generated home page contains the sample post title, date, and category.

- [ ] **Step 5: Commit**

```bash
git add layouts/partials/post-list.html layouts/index.html content/posts/hello-yinwang-style.md
git commit -m "feat: add home page post listing"
```

### Task 4: Implement Single Post Page and Reading Styles

**Files:**
- Create: `layouts/_default/single.html`
- Modify: `assets/css/main.css`
- Test: sample post detail rendering

- [ ] **Step 1: Write the failing article page expectation**

```html
<!-- Expected behavior:
1. The article page renders the title and publish date.
2. Categories render near the metadata.
3. Markdown body content is shown in a narrow reading column.
-->
```

- [ ] **Step 2: Run build to verify article rendering is still generic**

Run: `hugo --gc --minify`
Expected: FAIL the behavior expectation because article pages are still using Hugo defaults or incomplete templates.

- [ ] **Step 3: Write the single template and article styles**

```html
<!-- layouts/_default/single.html -->
{{ define "main" }}
  <article class="post-single">
    <header class="post-single-header">
      <h1>{{ .Title }}</h1>
      <p class="post-single-meta">
        <time datetime="{{ .Date.Format `2006-01-02` }}">{{ .Date.Format "2006-01-02" }}</time>
        {{ with .Params.categories }}
          <span> / {{ delimit . " / " }}</span>
        {{ end }}
      </p>
    </header>
    <div class="post-single-content">
      {{ .Content }}
    </div>
  </article>
{{ end }}
```

```css
/* append to assets/css/main.css */
.post-list {
  display: grid;
  gap: 2rem;
}

.post-row-title {
  margin: 0 0 0.4rem;
  font-size: 1.45rem;
}

.post-row-meta,
.post-row-summary,
.post-single-meta {
  color: var(--muted);
  font-size: 0.95rem;
}

.post-row-summary {
  margin: 0.6rem 0 0;
}

.post-single-header {
  margin: 1rem 0 2rem;
}

.post-single h1 {
  margin-bottom: 0.5rem;
  line-height: 1.3;
}

.post-single-content p,
.post-single-content ul,
.post-single-content ol,
.post-single-content blockquote,
.post-single-content pre {
  margin: 1.15rem 0;
}

.post-single-content code,
.post-single-content pre {
  font-family: "SFMono-Regular", "Menlo", monospace;
}

.post-single-content pre {
  overflow-x: auto;
  padding: 1rem;
  background: #efebe3;
  border: 1px solid var(--border);
}
```

- [ ] **Step 4: Run build to verify the single post page works**

Run: `hugo --gc --minify`
Expected: PASS and the generated sample post page contains the title, metadata, and article body.

- [ ] **Step 5: Commit**

```bash
git add layouts/_default/single.html assets/css/main.css
git commit -m "feat: add post detail template"
```

### Task 5: Implement Categories and Generic List Pages

**Files:**
- Create: `layouts/_default/list.html`
- Create: `layouts/categories/terms.html`
- Modify: `assets/css/main.css`
- Test: category index and per-category listing

- [ ] **Step 1: Write the failing taxonomy expectation**

```html
<!-- Expected behavior:
1. /categories/ lists all categories.
2. /categories/<name>/ lists posts in that category.
3. Category pages reuse the same reading-oriented post list style.
-->
```

- [ ] **Step 2: Run build to verify taxonomy navigation is incomplete**

Run: `hugo --gc --minify`
Expected: FAIL the behavior expectation because category templates do not exist yet.

- [ ] **Step 3: Write list templates and category styles**

```html
<!-- layouts/_default/list.html -->
{{ define "main" }}
  <section class="taxonomy-list">
    <h1>{{ .Title }}</h1>
    {{ partial "post-list.html" .Pages }}
  </section>
{{ end }}
```

```html
<!-- layouts/categories/terms.html -->
{{ define "main" }}
  <section class="category-terms">
    <h1>分类</h1>
    <ul class="category-terms-list">
      {{ range .Data.Terms.Alphabetical }}
        <li>
          <a href="{{ .Page.RelPermalink }}">{{ .Page.Title }}</a>
          <span>({{ .Count }})</span>
        </li>
      {{ end }}
    </ul>
  </section>
{{ end }}
```

```css
/* append to assets/css/main.css */
.taxonomy-list h1,
.category-terms h1 {
  margin-bottom: 1.5rem;
}

.category-terms-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.category-terms-list li {
  margin: 0.75rem 0;
}
```

- [ ] **Step 4: Run build to verify categories render correctly**

Run: `hugo --gc --minify`
Expected: PASS and generated pages include `/categories/` plus `/categories/随笔/`.

- [ ] **Step 5: Commit**

```bash
git add layouts/_default/list.html layouts/categories/terms.html assets/css/main.css
git commit -m "feat: add category pages"
```

### Task 6: Implement Yearly Archive Page

**Files:**
- Create: `layouts/archive/list.html`
- Create: `content/archive/_index.md`
- Modify: `assets/css/main.css`
- Test: yearly archive grouping

- [ ] **Step 1: Write the failing archive expectation**

```html
<!-- Expected behavior:
1. /archive/ exists.
2. Posts are grouped by year.
3. Each row links to the post and shows the date.
-->
```

- [ ] **Step 2: Run build to verify the archive page is missing**

Run: `hugo --gc --minify`
Expected: FAIL the behavior expectation because no archive content page or custom archive layout exists yet.

- [ ] **Step 3: Write the archive page and template**

```md
---
title: "归档"
layout: "archive"
---
```

```html
<!-- layouts/archive/list.html -->
{{ define "main" }}
  {{ $posts := where site.RegularPages "Section" "posts" }}
  <section class="archive-page">
    <h1>{{ .Title }}</h1>
    {{ range ($posts.GroupByDate "2006") }}
      <section class="archive-year">
        <h2>{{ .Key }}</h2>
        <ul class="archive-year-list">
          {{ range .Pages.ByDate.Reverse }}
            <li>
              <time datetime="{{ .Date.Format `2006-01-02` }}">{{ .Date.Format "01-02" }}</time>
              <a href="{{ .RelPermalink }}">{{ .Title }}</a>
            </li>
          {{ end }}
        </ul>
      </section>
    {{ end }}
  </section>
{{ end }}
```

```css
/* append to assets/css/main.css */
.archive-year {
  margin: 2rem 0;
}

.archive-year-list {
  list-style: none;
  padding: 0;
}

.archive-year-list li {
  display: flex;
  gap: 1rem;
  margin: 0.7rem 0;
}

.archive-year-list time {
  color: var(--muted);
  min-width: 3.5rem;
}
```

- [ ] **Step 4: Run build to verify archive grouping works**

Run: `hugo --gc --minify`
Expected: PASS and generated `/archive/index.html` contains a `2026` section with the sample post.

- [ ] **Step 5: Commit**

```bash
git add layouts/archive/list.html content/archive/_index.md assets/css/main.css
git commit -m "feat: add yearly archive page"
```

### Task 7: Add About Page and Writing Documentation

**Files:**
- Create: `content/about/index.md`
- Modify: `README.md`
- Test: About page generation and authoring instructions

- [ ] **Step 1: Write the failing documentation expectation**

```md
Expected deliverables after this task:
- An About page exists at /about/.
- README explains how to create a post, preview locally, and build for production.
```

- [ ] **Step 2: Run build to verify the About page is missing**

Run: `hugo --gc --minify`
Expected: FAIL the behavior expectation because `/about/` content does not exist yet.

- [ ] **Step 3: Write the About page and complete README**

```md
---
title: "关于"
draft: false
---

这里是关于页面。

你可以在这里介绍自己、写作主题和联系方式。
```

```md
# README.md

## Overview

This repository contains a Hugo blog with a minimalist reading-first theme.

## Requirements

- Hugo extended

## Local Preview

```bash
hugo server
```

Open `http://localhost:1313/`.

## Create a New Post

```bash
hugo new content/posts/my-new-post.md
```

Update the front matter:

- `title`
- `date`
- `categories`
- `tags`
- `description`
- `draft`

## Production Build

```bash
hugo --gc --minify
```

The generated files are written to `public/`.

## Deployment

Push to `main` and let GitHub Actions publish to GitHub Pages.
```

- [ ] **Step 4: Run build to verify About and docs are in place**

Run: `hugo --gc --minify`
Expected: PASS and generated `/about/index.html` exists.

- [ ] **Step 5: Commit**

```bash
git add content/about/index.md README.md
git commit -m "feat: add about page and writing docs"
```

### Task 8: Add GitHub Pages Deployment Workflow

**Files:**
- Create: `.github/workflows/hugo.yml`
- Test: workflow syntax and Hugo production build

- [ ] **Step 1: Write the failing deployment expectation**

```yaml
# Expected behavior:
# 1. A push to main triggers the workflow.
# 2. The workflow builds the Hugo site.
# 3. The workflow deploys to GitHub Pages using official actions.
```

- [ ] **Step 2: Verify deployment automation is currently missing**

Run: `rg --files .github/workflows`
Expected: FAIL with no `hugo.yml` workflow present.

- [ ] **Step 3: Write the GitHub Pages workflow**

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.127.0
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: ${{ env.HUGO_VERSION }}
          extended: true

      - name: Build
        run: hugo --gc --minify

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 4: Run build to verify the production command still passes**

Run: `hugo --gc --minify`
Expected: PASS with no production build errors.

- [ ] **Step 5: Commit**

```bash
git add .github/workflows/hugo.yml
git commit -m "ci: add github pages deployment workflow"
```

### Task 9: Final Verification and Polish

**Files:**
- Modify: `hugo.toml` if needed for final title/description cleanup
- Modify: `assets/css/main.css` if needed for final spacing fixes
- Test: complete local build and manual inspection

- [ ] **Step 1: Write the final verification checklist**

```md
Expected behavior:
- Home page lists posts in reverse chronological order.
- Single post page uses the reading layout.
- Categories index and category detail pages render.
- Archive page groups posts by year.
- About page renders.
- Draft posts stay out of production output.
- GitHub Pages workflow exists.
```

- [ ] **Step 2: Run the full build and inspect generated routes**

Run: `hugo --gc --minify`
Expected: PASS with generated routes for `/`, `/posts/hello-yinwang-style/`, `/categories/`, `/categories/随笔/`, `/archive/`, and `/about/`.

- [ ] **Step 3: Verify draft exclusion with a temporary draft file**

```md
---
title: "Draft Check"
date: 2026-06-09T13:00:00+08:00
draft: true
categories: ["检查"]
tags: []
description: "This file exists only to confirm draft exclusion."
---

This content must not appear in the production build.
```

Create the file temporarily at `content/posts/draft-check.md`, run the build, verify it does not appear in `public/`, then remove it before commit.

- [ ] **Step 4: Commit final cleanup**

```bash
git add hugo.toml assets/css/main.css README.md .github/workflows/hugo.yml layouts content archetypes
git commit -m "chore: finalize yinwang-style blog scaffold"
```
