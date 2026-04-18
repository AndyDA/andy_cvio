# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Setup

This is a Jekyll-based static site. To set up the development environment:

1. Install Ruby (version 3.4.4 as specified in .ruby-version)
2. Install Bundler: `gem install bundler`
3. Install dependencies: `bundle install`

## Common Commands

- **Start development server**: `bundle exec jekyll serve`
  - Site will be available at http://localhost:4000
  - Automatically regenerates when files change

- **Build for production**: `bundle exec jekyll build`
  - Outputs static files to the `_site/` directory

- **Check for issues**: `bundle exec jekyll doctor`
  - scans for known issues and potential problems

- **Netlify deployment**: The site is configured for Netlify via `netlify.toml`
  - Build command: `bundle install && bundle exec jekyll build --verbose --trace`
  - Publish directory: `_site/`

## Project Structure

- `_config.yml`: Main Jekyll configuration (site settings, plugins, collections)
- `_data/`: YAML files containing reusable content (navigation, footer, UI text, etc.)
- `_includes/`: Reusable UI components (header, footer, widgets)
- `_layouts/`: Page layouts (default, page, post, etc.)
- `_posts/`: Blog posts in Markdown format
- `_works/`: Collection for portfolio works (configured in _config.yml)
- `assets/`: Static assets (CSS, JavaScript, images)
- `*.html`: Main pages (index, about, portfolio, contacts, etc.)
- `Gemfile`: Ruby dependencies (Jekyll and plugins)
- `netlify.toml`: Netlify build configuration

## Architecture Notes

- The site uses Jekyll's templating system with Liquid syntax
- Content is structured via:
  - Data files in `_data/` for global settings and reusable content
  - Collections (`works`) for portfolio items
  - Standard pages and posts
- Styling is handled by `syntax.css` and potentially custom SCSS in `_sass/` (if present)
- JavaScript is minimal; the theme relies on CSS for animations and interactions
- SEO is managed via the `jekyll-seo-tag` plugin

## Notes

- The site does not have a formal test suite. Validation can be done via `jekyll doctor` and manual browser testing.
- When updating dependencies, run `bundle update` followed by `bundle install`.
- The `.jekyll-cache/` and `_site/` directories are generated and should be ignored by version control (they are in .gitignore).