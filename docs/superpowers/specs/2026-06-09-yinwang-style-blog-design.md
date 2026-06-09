# Yinwang Style Hugo Blog Design

## Context

This repository starts effectively empty and will be turned into a static blog that matches the overall reading experience and information architecture of `yinwang.org`, while remaining simple to write in Markdown and easy to deploy to GitHub Pages.

The user confirmed:

- Static site generator: `Hugo`
- GitHub Pages deployment target: personal site at `https://<username>.github.io/`
- Home page shape: pure article list
- Organization model: follow `yinwang.org` style classification, archive, and ordering
- Language strategy: Chinese first, but keep the structure ready for future English content

## Goals

- Build a minimalist Chinese-first blog with a reading-oriented visual style similar to `yinwang.org`
- Allow the user to author posts directly in Markdown
- Generate a fully static website
- Deploy automatically to GitHub Pages from `main`
- Preserve a clean path for future bilingual expansion without restructuring the site

## Non-Goals

- No CMS, database, or dynamic backend
- No card-based magazine layout
- No client-side search or heavy JavaScript filtering in the first version
- No bilingual UI in the first release

## Architecture

The site will use Hugo with a custom in-repo theme implementation rather than an external theme. Content will live under `content/`, templates under `layouts/`, and styling under `assets/` or `static/` depending on whether Hugo Pipes are needed. The generated site will be deployed using GitHub Actions to GitHub Pages.

Content metadata in front matter will drive all list views, classification, and ordering. The site structure will stay simple:

- Home page for reverse-chronological article listing
- Article detail pages for reading
- Category index and per-category list pages
- Archive page grouped by year
- About page

Future English support will be enabled by keeping content and configuration compatible with later introduction of `content/en/` and multilingual Hugo settings.

## Information Architecture

### Home Page

The home page will closely follow the spirit of `yinwang.org`:

- Display a pure list of articles
- Order by publish date descending
- Show article title, publish date, and category
- Optionally show a one-line description or summary when available
- Prioritize scanability and reading over decorative presentation

There will be no hero banner, no cards, and no large promotional section above the list.

### Categories

Categories are a primary navigation and organization tool. Posts will declare categories in front matter. The site will provide:

- A category index page listing available categories
- A per-category page listing posts in that category in reverse chronological order

This mirrors the practical use of categories on `yinwang.org` without introducing an overly interactive client-side filter UI.

### Archive

The archive page will group posts by year. Within each year, posts will be listed in reverse chronological order. This page exists for long-term browsing of older writing and gives users a simple history view separate from the home page.

### About

The About page will be a standalone content page and will not compete with the home page for attention.

## Content Model

Each post will be stored as a Markdown file. The expected front matter fields are:

- `title`
- `date`
- `categories`
- `tags`
- `description`
- `draft`

`tags` will be supported in content metadata so the model stays extensible, but the first version will focus on categories, archive, and chronological ordering in the visible navigation.

An example post should be included so the writing workflow is obvious from the start.

## Layout and Visual Direction

The visual direction should evoke the calm, restrained reading experience of `yinwang.org` without trying to clone it mechanically.

### Principles

- Light background
- Dark body text
- Low-saturation link color
- Generous whitespace
- Narrow reading column for article pages
- Clear typographic hierarchy
- Minimal or no animation

### Typography

The body should favor a serif-oriented reading feel suitable for Chinese prose. Headings, metadata, lists, blockquotes, and code blocks should be styled conservatively and legibly. Code blocks should be readable but not visually dominant.

### Navigation

The top navigation should remain extremely simple:

- Home
- Archive
- Categories
- About

No icon-heavy UI, dropdown menus, or dense header controls should be introduced in the first version.

## Markdown Authoring Workflow

The user should be able to create a new post by adding a Markdown file under the content directory, filling in front matter, and writing the article body. Hugo should handle:

- Article page generation
- Home page listing
- Category page generation
- Archive view population

The workflow should not require manual editing of list pages or navigation entries when new posts are added.

## Deployment

Deployment will use the official GitHub Pages workflow with GitHub Actions.

### Deployment Model

- Source branch: `main`
- Build in GitHub Actions
- Publish the generated static output using GitHub Pages Actions

This is preferred over manually committing the built site to a separate branch because it keeps the repository cleaner and aligns with the modern GitHub Pages deployment flow.

## Extensibility

The first release will be Chinese-only, but the implementation should avoid hard-coding assumptions that would block future multilingual support. The design should leave room for:

- Adding `content/en/`
- Adding multilingual configuration
- Surfacing a language switch later when English content actually exists

The first release should not expose inactive language-switch UI.

## Error Handling and Operational Expectations

- Posts missing required front matter should still fail in an obvious, debuggable way during local preview or build review
- Draft posts should not appear in production output
- Empty category or archive states should render gracefully

Because this is a static site, operational complexity is intentionally low. Most failures will occur at authoring or build time rather than runtime.

## Testing and Verification Strategy

Verification should focus on:

- Hugo build succeeds cleanly
- Local preview renders the intended pages
- Home ordering is correct
- Category pages list the correct posts
- Archive grouping is correct
- Draft handling behaves correctly
- GitHub Actions configuration builds and deploys the site as expected

## Deliverables

The initial implementation should produce:

- Hugo site scaffold
- Custom layouts and styling
- Home page
- Single post template
- Category index and per-category pages
- Archive page
- About page
- At least one example post
- GitHub Actions deployment workflow
- Documentation for writing new posts and deploying
