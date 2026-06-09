# babydragon

A Hugo blog scaffold with a Yinwang-inspired reading-focused theme.

## Requirements

- Hugo extended (`hugo version`)

## Local Preview

```bash
hugo server
```

Then open `http://localhost:1313/`.

## Create a New Post

```bash
hugo new content/posts/my-new-post.md
```

Update the front matter before publishing:

- `title`
- `date`
- `categories`
- `tags`
- `description`
- `draft`

Write the article body in Markdown below the front matter.

## Production Build

```bash
hugo --gc --minify
```

The generated static site is written to `public/`.

## Deployment

Before publishing, update `baseURL` in `hugo.toml` to your GitHub Pages domain.

Push to `main` and let GitHub Actions deploy the site to GitHub Pages.
