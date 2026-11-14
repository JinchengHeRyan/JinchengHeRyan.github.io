# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal academic website (https://JinchengHeRyan.github.io) built with Jekyll on the upstream [al-folio](https://github.com/alshedivat/al-folio) theme. Most files are unmodified template code, and the template's demo content (sample posts, projects, news, `about_einstein.md`, the CV and people pages) is still in the repo but hidden. The site owner's own content is mostly in a handful of files:

- `_pages/about.md`: homepage bio (layout `about`, permalink `/`)
- `_bibliography/papers.bib`: publications, rendered by jekyll-scholar
- `assets/img/publication_preview/`: thumbnails referenced by the bib `preview` field
- `_config.yml`: identity, social IDs, analytics, feature flags
- `_data/repositories.yml`: GitHub users and repos for the (hidden) repositories page

## Rules

- Never run `git commit` unless the user explicitly asks for it. When asked, keep the commit message short and clear, and don't add a `Co-Authored-By` or any other Anthropic/Claude attribution lines.
- Don't add publication entries to `_bibliography/papers.bib` yourself. If a change needs a new bib entry, tell the user and let them add it.

## Commands

There is no test suite. The checks CI runs are a Prettier format check and a link check.

```bash
# Local dev server, recommended (prebuilt image; serves on http://localhost:8080 with livereload)
docker compose pull && docker compose up
docker compose -f docker-compose-slim.yml up        # slimmer image, same behavior

# Without Docker (needs Ruby/Bundler, ImageMagick, and `pip install jupyter`); serves on :4000
bundle install
bundle exec jekyll serve --lsi

# Production build, same as CI (bin/cibuild)
JEKYLL_ENV=production bundle exec jekyll build --lsi
purgecss -c purgecss.config.js                      # CI strips unused CSS from _site after the build

# Formatting: CI fails on any Prettier diff
npm install
npx prettier . --check
npx prettier . --write
```

Changes to `_config.yml` are only picked up after restarting `jekyll serve`. The Docker entry point (`bin/entry_point.sh`) watches the file and restarts Jekyll on its own. Other files hot-reload.

## Deployment

Pushing to `master` runs `.github/workflows/deploy.yml`, which builds the site and force-pushes `_site` to the `gh-pages` branch. GitHub Pages serves `gh-pages`. Never edit `gh-pages` directly: every deploy overwrites it. Edits to the docs files (README/INSTALL/CUSTOMIZE/FAQ/CONTRIBUTING) don't trigger a deploy.

## Architecture

- **Pages and nav**: Each `_pages/*.md` file is a page. The navbar (`_includes/header.liquid`) lists only pages with `nav: true`, sorted by `nav_order`. To hide a page, set `nav: false` instead of deleting the file. That's how blog, projects, CV, repositories, teaching, and people are hidden right now. Only Publications is in the nav.
- **Homepage sections** (`_layouts/about.liquid`) are switched on by front matter in `about.md` (`news`, `selected_papers`, `social`), and some also need a site-wide flag. For example, news shows only if `announcements.enabled: true` in `_config.yml` (it's currently `false`).
- **Publications pipeline**: `_pages/publications.md` calls `{% bibliography %}`, and each entry renders through `_layouts/bib.liquid`. `selected = {true}` in a bib entry puts it in the homepage "Selected Publications" list (`_includes/selected_papers.liquid`). Custom bib fields like `pdf`, `website`, `code`, `preview`, `abstract`, `bibtex_show`, `award`, and `google_scholar_id` drive the buttons and badges. A relative path in `pdf`/`poster`/`slides`/`supp` resolves under `assets/pdf/`, and a relative `preview` resolves under `assets/img/publication_preview/`. The fields listed in `filtered_bibtex_keywords` in `_config.yml` are stripped from the "Bib" popup. Matching `scholar.first_name`/`last_name` highlights the site owner in author lists. Co-author links come from `_data/coauthors.yml`, and venue `abbr` links from `_data/venues.yml`.
- **Custom plugins** (`_plugins/`) add Liquid tags and filters used by the layouts: `google_scholar_citations` (scrapes Scholar at build time with random sleeps, so builds get slow when entries have `google_scholar_id`), `details`, `file_exists`, `hideCustomBibtex`, `remove_accents`, and the `bust_file_cache`/`bust_css_cache` filters. `external-posts.rb` pulls RSS feeds from `external_sources` in `_config.yml` into the posts collection at build time. It still points at the template's Medium feed. `download-3rd-party.rb` fills in the `{{version}}` placeholders in `third_party_libraries` CDN URLs and can vendor them into `assets/libs/` when `download: true`.
- **Third-party JS/CSS** is loaded from CDN URLs with SRI hashes, all set in `third_party_libraries` in `_config.yml`. If you bump a version, update its `integrity` hash too.
- **Images**: jekyll-imagemagick generates responsive WebP variants (480/800/1400px) of everything in `assets/img/`. `_includes/figure.liquid` emits `srcset`s that point to them, so local builds need ImageMagick installed.
- **Styling**: The theme color and the light/dark palettes are in `_sass/_themes.scss` (`--global-theme-color`), and the named color values are in `_sass/_variables.scss`. `assets/css/main.scss` is the entry point.
- **CV page**: built from `assets/json/resume.json` (JSON Resume, loaded with jekyll-get-json) when that file exists, and from `_data/cv.yml` otherwise.

## Conventions

- Prettier settings: `printWidth: 150` and `@shopify/prettier-plugin-liquid` (`.prettierrc`). Files in `.prettierignore` are skipped.
- `Gemfile.lock` is gitignored, so don't commit it.
- Since this is a template fork, prefer changing config and content over editing layouts and includes. That makes future al-folio upgrades easier to merge (see "Upgrading from a previous version" in `INSTALL.md`).
