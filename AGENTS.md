# AGENTS.md

Guidance for OpenCode sessions working with this Jekyll-based portfolio site.

## Setup

- Ruby version is pinned to **3.4.4** (see `.ruby-version`)
- Always use `bundle exec` prefix for Jekyll commands
- Dependencies managed via `Gemfile` + `bundle install`

## Essential Commands

```bash
# Development server (auto-reloads on file changes)
bundle exec jekyll serve          # http://localhost:4000

# Production build (outputs to `_site/`)
bundle exec jekyll build

# Health check
bundle exec jekyll doctor
```

## Key Architecture

- **Site type**: Static portfolio/resume (not a blog engine)
- **Collections**: `works/` contains portfolio items (not `_posts/`)
- **Templating**: Liquid + Jekyll, SCSS in `_sass/`
- **Data files**: YAML in `_data/` (content.yml, home.yml, etc.)
- **Plugins**: jekyll-archives, jekyll-paginate-v2, jekyll-seo-tag

## Build Behavior

- `_config.yml` changes require server restart to take effect
- Generated dirs: `.jekyll-cache/`, `_site/` (both in `.gitignore`)
- SASS output is compressed (see `sass.style: compressed` in `_config.yml`)

## Deployment

- Platform: Netlify
- Build command: `bundle install && bundle exec jekyll build --verbose --trace`
- Publish dir: `_site/`
- Production env var: `JEKYLL_ENV=production`

## Gotchas

- Portfolio entries go in `_works/`, not `_posts/`
- Posts in `_posts/` are for blog content only
- Contact form uses `formsubmit.io` (configured in `_config.yml`)
- No formal test suite—validate via `jekyll doctor` and manual browser check
