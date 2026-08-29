# AGENTS.md

This file provides context, architectural details, workflow commands, and authoring guidelines for AI agents working in this repository.

---

## 1. Project Overview

- **Project**: Personal website and technical engineering blog for Saran Sankaran.
- **Live URL**: [https://saran.sankaran.dev](https://saran.sankaran.dev)
- **Repository**: `saran2020/saran2020.github.io`
- **Core Framework**: [Jekyll 4.4.x](https://jekyllrb.com/) (Ruby static site generator)
- **Theme**: [Minimal Mistakes Jekyll Theme](https://mmistakes.github.io/minimal-mistakes/) (`minimal-mistakes-jekyll` gem version `~> 4.28.1`, skin: `dirt`)
- **Hosting & Deployment**: GitHub Pages deployed via GitHub Actions (`.github/workflows/deploy.yml`) on pushes to `main`.
- **Domain Routing**: Custom domain managed through the root `CNAME` file pointing to `saran.sankaran.dev`.

---

## 2. Technology Stack & Environment

| Component | Tool / Version | Notes |
| :--- | :--- | :--- |
| **Language Runtime** | Ruby `3.3.12` | Specified in `mise.toml`, `.ruby-version`, and GitHub Actions |
| **Environment Manager** | `mise` (or `rbenv` / `asdf`) | Configured via `mise.toml` |
| **Package Manager** | Bundler (`2.5+`) | Dependencies declared in `Gemfile` & locked in `Gemfile.lock` |
| **Static Site Generator** | Jekyll `~> 4.4.1` | Configured via `_config.yml` |
| **Markdown Engine** | `kramdown` | GFM (GitHub Flavored Markdown) parser, Rouge syntax highlighter |
| **Local Web Server** | `webrick` (`~> 1.8`) | Required for Ruby 3.x local development |
| **Plugins** | `jekyll-include-cache`, `jekyll-feed`, `jekyll-sitemap`, `jekyll-paginate`, `jemoji`, `jekyll-gist` | Listed in `Gemfile` and `_config.yml` |
| **CI / CD** | GitHub Actions | `ci.yml` (pull request validation), `deploy.yml` (production deployment) |

---

## 3. Directory Structure

```
├── .github/
│   └── workflows/
│       ├── ci.yml            # CI validation workflow (jekyll build --strict)
│       └── deploy.yml        # Production GitHub Pages deployment workflow
├── _data/
│   ├── authors.yml           # Author profiles (bio, social links, avatars)
│   └── navigation.yml        # Masthead navigation menu configuration
├── _drafts/                  # Unpublished draft posts (ignored by git)
├── _includes/
│   └── head/
│       └── custom.html       # Injected into <head> (favicon, custom meta tags)
├── _pages/
│   ├── 404.md                # 404 error page (Markdown)
│   ├── blogs.md              # Blog archive listing (/blogs/)
│   ├── contact.md            # Contact form page (/contact/)
│   └── thank-you.md          # Form submission confirmation page (/thank-you)
├── _posts/                   # Published blog articles (YYYY-MM-DD-title.md)
├── assets/
│   ├── css/                  # Custom SCSS overrides if present
│   └── images/               # Media assets, illustrations, screenshots, favicons
├── 404.html                  # Fallback 404 page
├── _config.yml               # Main Jekyll configuration settings
├── AGENTS.md                 # Agent guide and instructions (this file)
├── CNAME                     # Custom domain definition (saran.sankaran.dev)
├── Gemfile                   # Ruby dependency declarations
├── Gemfile.lock              # Pinned Ruby dependencies
├── index.md                  # Site homepage
├── mise.toml                 # Tool version definitions (Ruby 3.3.12)
└── README.md                 # Human-facing project overview and setup guide
```

---

## 4. Key Development & Build Commands

All commands should be executed from the repository root:

```bash
# 1. Install dependencies
bundle install

# 2. Start local development server with live reload
bundle exec jekyll serve --livereload

# 3. Start local development server including drafts
bundle exec jekyll serve --drafts --livereload

# 4. Perform a strict build check (matches CI)
bundle exec jekyll build --strict

# 5. Build for production environment
JEKYLL_ENV=production bundle exec jekyll build
```

---

## 5. Content Authoring Guidelines

### 5.1. Blog Posts (`_posts/`)

- **File Naming**: Must follow `YYYY-MM-DD-<slug>.md` (e.g., `_posts/2024-12-26-Protobuf-vs-JSON-The-Compression-Test-That-Changed-My-Mind.md`).
- **Permalink Structure**: Configured as `/:categories/:title/`. The primary category directly determines the URL path (e.g. `categories: [Android]` leads to `/android/<slug>/`).
- **Timezone**: Default timezone is `Asia/Calcutta`.
- **Frontmatter Template**:

```markdown
---
title: "Article Title Here"
date: YYYY-MM-DD 00:00:00 +0000
categories:
  - Android
tags:
  - Android
  - Kotlin
  - Performance
---
```

- **Excerpts**: Use `<!--more-->` or leave a double newline after the introductory paragraph.
- **Images**: Store images in `assets/images/` and reference using root-relative paths:
  ```markdown
  ![](/assets/images/example-image.png){: .align-center}
  ```
- **Code Highlighting**: Use fenced code blocks with appropriate language tags (`kotlin`, `go`, `bash`, `yaml`, `html`, etc.).

### 5.2. Drafts (`_drafts/`)

- Stored in `_drafts/<slug>.md` without a date prefix in the filename.
- Git-ignored by default in `.gitignore`.
- Preview locally using `bundle exec jekyll serve --drafts`.

### 5.3. Pages (`_pages/`)

- Pages specify `permalink: /<slug>/` in their frontmatter.
- Default layout is `single` with `author_profile: true` (inherited from `_config.yml`).
- Standalone landing pages can configure custom layouts (`layout: posts` for the blog index in `_pages/blogs.md`).

### 5.4. Site Metadata & Author Config (`_data/`)

- Author data is configured in `_data/authors.yml` under the key `Saran`.
- Site navigation items are configured in `_data/navigation.yml`.

---

## 6. Configuration & Architecture Rules

1. **`_config.yml` Changes**:
   - `_config.yml` is only read at server start. If modified, the Jekyll server must be restarted.
   - Defaults are defined under `defaults:` for both `posts` and `pages`. Do not duplicate default keys in frontmatter unless intentionally overriding.

2. **Strict Verification**:
   - Always run `bundle exec jekyll build --strict` before submitting changes.
   - CI (`.github/workflows/ci.yml`) runs on Ubuntu with Ruby `3.3.12` and fails if any Liquid syntax error, invalid tag, or broken dependency is encountered.

3. **Ruby & Tooling Compatibility**:
   - Always ensure any Ruby version changes are kept synchronized across:
     - `mise.toml` (`ruby = "3.3.12"`)
     - `.ruby-version` (`3.3.12`)
     - `.github/workflows/ci.yml` (`ruby-version: '3.3.12'`)
     - `.github/workflows/deploy.yml` (`ruby-version: '3.3.12'`)

4. **Mathematical Expressions & Text Formatting**:
   - Do not use LaTeX (`$...$`). Use Unicode mathematical characters (e.g. `→`, `≤`, `≥`, `∈`, `×`, `√`) or standard ASCII equivalents.

---

## 7. Agent Checklist for Pull Requests / Modifications

- [ ] Dependencies resolve cleanly via `bundle install`.
- [ ] Frontmatter includes proper `title`, `categories`, and `tags`.
- [ ] Post filenames conform to `YYYY-MM-DD-title-slug.md`.
- [ ] Images are placed in `assets/images/` and use root-relative paths (`/assets/images/...`).
- [ ] Build succeeds with `bundle exec jekyll build --strict`.
- [ ] No generated output directories (`_site/`, `.jekyll-cache/`, `.sass-cache/`) are committed.
