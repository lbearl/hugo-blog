# Luke Bearl Hugo Blog

This repository contains a Hugo-based personal blog powered by the `hugo-coder` theme.

## Overview

- Site config: `hugo.toml`
- Theme: `themes/hugo-coder`
- Blog posts: `content/posts/`
- Default content archetype: `archetypes/default.md`

## Prerequisites

- Install [Hugo](https://gohugo.io/getting-started/installing/)
- Git (optional, for repository management)

## Local development

From the repository root, run:

```bash
hugo server -D
```

Then open:

```bash
http://localhost:1313
```

The `-D` flag enables drafts so you can preview new posts before publishing.

## Build for production

Generate the public site output with:

```bash
hugo
```

The generated files will appear in the `public/` directory.

## Creating a new post

New blog posts are stored in `content/posts/` as Markdown files.

### Recommended: use Hugo new

Hugo can scaffold a new post using the default archetype.

```bash
hugo new posts/my-new-post.md
```

This creates a new file like:

- `content/posts/my-new-post.md`

The file is initialized using `archetypes/default.md`, which contains the standard front matter.

### Manual post creation

If you want to create a post manually, add a Markdown file in `content/posts/` with front matter like this:

```markdown
+++
title = "My New Post"
date = 2026-04-26T12:00:00-00:00
draft = true
+++

Your post content starts here.
```

### Publish a post

When the post is ready to go live, change:

```toml
draft = false
```

Then rebuild the site without `-D` to confirm published content.

## Post metadata and taxonomies

The site uses the following taxonomies configured in `hugo.toml`:

- `categories`
- `series`
- `tags`
- `authors`

Add taxonomy metadata inside the post front matter if needed, for example:

```toml
categories = ["DevOps", "Hugo"]
tags = ["hugo", "blog"]
series = ["Site Setup"]
authors = ["Luke Bearl"]
```

## Notes

- `content/about.md` and `content/now.md` are standalone pages.
- `baseurl` is currently set to `http://www.lukebearl.com` in `hugo.toml`.
- The site uses the `hugo-coder` theme from `themes/hugo-coder`.

## Useful commands

```bash
# Start development server and show drafts
hugo server -D

# Create a new post using the default archetype
hugo new posts/my-new-post.md

# Build the site for production
hugo
```
