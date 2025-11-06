# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based personal blog/website ("Quiet Reflections") using the Minimal Mistakes theme. The site contains blog posts, infrastructure documentation, and technical guides. It is designed to be hosted on GitHub Pages.

## Build & Development Commands

### Local Development
```bash
bundle exec jekyll serve
```
This starts a local development server at http://localhost:4000 with auto-regeneration on file changes.

### Build Site
```bash
bundle exec jekyll build
```
Generates the static site in the `_site/` directory.

### Install/Update Dependencies
```bash
bundle install
bundle update
```

## Site Architecture

### Directory Structure

- **`_posts/`** - Blog posts with date-based filenames (YYYY-MM-DD-title.markdown)
  - Posts use `layout: single` in front matter
  - Categories/tags determine URL structure via permalink config
  
- **`_infra/`** - Infrastructure documentation collection
  - Custom collection defined in _config.yml
  - Uses sidebar navigation defined in `_data/navigation.yml`
  - All files use `layout: single` and `classes: wide`
  
- **`_pages/`** - Static pages (About, Blogs listing, etc.)
  - Uses permalinks defined in front matter
  
- **`_data/navigation.yml`** - Defines site navigation
  - `main`: Top navigation bar
  - `infra`: Sidebar navigation for infra collection
  
- **`_site/`** - Generated static site (excluded from git)

- **`assets/css/main.scss`** - Custom CSS overrides
  - Imports minimal-mistakes base styles
  - Custom font size (0.75em) and body styling

### Jekyll Configuration

- **Theme**: Minimal Mistakes (via remote_theme: "mmistakes/minimal-mistakes@4.26.2")
- **Skin**: "air"
- **Markdown**: kramdown with GFM input
- **Pagination**: 5 posts per page
- **Plugins**: jekyll-paginate, jekyll-sitemap, jekyll-gist, jekyll-feed, jekyll-include-cache, jekyll-sass-converter

### Collections

Custom collection `_infra` is configured to:
- Output as pages (output: true)
- Use sidebar navigation (`nav: "infra"`)
- Use wide layout (`classes: wide`)

## Content Creation Patterns

### Creating Blog Posts
1. Create file in `_posts/` with format: `YYYY-MM-DD-title.markdown`
2. Include front matter:
   ```yaml
   ---
   layout: single
   title: "Post Title"
   date: YYYY-MM-DD HH:MM:SS -0700
   categories: category subcategory
   ---
   ```

### Creating Infrastructure Docs
1. Create numbered markdown file in `_infra/` (e.g., `01_jekyll.md`, `02_webcertificate.md`)
2. Include front matter with permalink:
   ```yaml
   ---
   permalink: /infra/shortname
   layout: single
   title: "Document Title"
   ---
   ```
3. Add corresponding entry to `_data/navigation.yml` under `infra` section

### Adding Navigation Items
Edit `_data/navigation.yml`:
- For top nav: add to `main` array
- For infra sidebar: add to `infra` array with title and children structure

## GitHub Pages Deployment

The site is configured for GitHub Pages deployment:
- Uses `github-pages` gem
- Remote theme method (not local theme installation)
- All GitHub Pages compatible plugins are whitelisted in _config.yml

Changes pushed to the main branch will trigger GitHub Pages build automatically.
