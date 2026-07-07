# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Ilyass Taouil's personal academic website (`itaouil.github.io`), built on the
[al-folio](https://github.com/alshedivat/al-folio) Jekyll theme. Most files here are unmodified
upstream theme scaffolding. The parts that are actually "the user's site" are the content files:
`_pages/`, `_bibliography/papers.bib`, `_news/`, `_projects/`, `_data/`, `assets/img/`, and
`_config.yml`. Prefer editing content over touching theme layouts/plugins unless the task is
explicitly about changing site behavior.

## Commands

Local development (Ruby/Bundler):

```bash
bundle install                      # first-time dependency install
bundle exec jekyll serve --lsi      # serve with live reload at http://localhost:4000 (--lsi enables related-posts)
bundle exec jekyll build --lsi      # production build into _site/
```

Local development (Docker, no local Ruby needed):

```bash
docker compose up                   # serves at http://localhost:8080
```

Production build (mirrors CI):

```bash
export JEKYLL_ENV=production
bundle exec jekyll build --lsi
purgecss -c purgecss.config.js      # strips unused CSS from _site/assets/css/
```

There is no test suite. `pre-commit` runs whitespace/EOF/YAML/large-file checks (`.pre-commit-config.yaml`).

## Deployment

Deployment is automatic — do **not** deploy by hand. On push to `master` (or `main`),
`.github/workflows/deploy.yml` builds the site with `JEKYLL_ENV=production`, runs `purgecss`, and
publishes `_site/` to the `gh-pages` branch via GitHub Pages. `master` holds source; `gh-pages` holds
built output. Just commit source changes to `master`. (`bin/deploy` is the manual equivalent, rarely needed.)

## Architecture

Jekyll static site. Content sources compile to a static `_site/`. Key pieces that span multiple files:

- **Publications** are driven by `jekyll-scholar`. `_bibliography/papers.bib` is the source of truth;
  the `publications` page and the "selected papers" block on the homepage are generated from it. Custom
  BibTeX fields control rendering — `selected={true}` surfaces a paper on the homepage, `preview={file.gif}`
  sets its thumbnail (in `assets/img/publication_preview/`), and `abbr`, `arxiv`, `pdf`, `code`, `website`,
  `bibtex_show`, etc. render as links/badges. The list of fields hidden from the shown BibTeX lives in
  `filtered_bibtex_keywords` in `_config.yml`. Scholar settings (style `apa`, group-by-year) are also in `_config.yml`.

- **Homepage** is `_pages/about.md` (`layout: about`, `permalink: /`). Its front matter toggles the
  news / latest-posts / selected-papers / social sections.

- **Collections** (`_config.yml`): `_news/` (announcements shown on homepage) and `_projects/`
  (portfolio cards). Blog posts live in `_posts/`.

- **Custom Liquid plugins** in `_plugins/` (loaded because this builds outside GitHub's safe mode):
  `hideCustomBibtex.rb` (strips the filtered BibTeX fields), `external-posts.rb` (pulls in posts from RSS
  feeds listed under `external_sources` in `_config.yml`), `cache-bust.rb` (MD5-hashes asset URLs),
  `details.rb`, and `file-exists.rb`.

- **Structure directories** follow Jekyll conventions: `_layouts/` (page templates), `_includes/`
  (partials), `_sass/` + `assets/css/` (styles; `_sass/_themes.scss` / `_variables.scss` for theming),
  `_data/` (structured YAML — `cv.yml`, `coauthors.yml`, `repositories.yml`, `venues.yml`).

## Notes

- `IEEEtran.cls`, `paper_template.tex/.pdf`, and `references.bib` in the repo root are a stray LaTeX
  paper template, unrelated to the website build. Don't wire them into the site.
- Full theme documentation (adding pages, projects, blog features, image config) is in `README.md`.
