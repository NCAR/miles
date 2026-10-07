# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The NCAR MILES (Machine Integration and Learning for Earth Systems) group website: a Hugo static site deployed to GitHub Pages (custom domain `miles.ucar.edu`). There is no application code or test suite; most changes are content edits to YAML data files and Markdown pages.

`CONTRIBUTING.md` was inherited from the NCAR VAST site, so its references to "VAST", `config.toml`, `data/services.yml` and the `vast` repo are stale. Its descriptions of Hugo structure and the theme are still accurate.

## Commands

```bash
hugo serve                 # local dev server with live reload at http://localhost:1313/
hugo --minify              # production build into public/ (what CI runs)
python update_metrics.py   # recompute data/metrics.yml counts (requires ruamel.yaml)
hugo new news/<slug>.md    # scaffold a page from themes/ncar/archetypes
```

CI uses Hugo extended (0.144.2 in `.github/workflows/hugo.yml`). `public/` and `resources/` are build output and are gitignored.

## Architecture

- **`config.yml`**: site params, the header nav menu, and footer sitemap links. A new top-level section has to be added to the nav and footer here by hand.
- **`data/*.yml`**: most page content is generated from these files by templates in `themes/ncar/layouts`. Each file has top-level `enable`/`title` keys, plus a list under `items` (publications, presentations, collaboration, realtimeproducts) or `members` (team, team_plus, alumni). Mapping:
  - `team.yml` (MILES Core), `team_plus.yml` (MILES+), `alumni.yml` → partials rendered on the About page
  - `publications.yml` → `/publications`; new entries are added at the **top** of `items` (newest first). `authors` may be an inline list or a block list.
  - `presentations.yml` → `/presentations`
  - `collaboration.yml` (SIParCS intern projects) → `/collaboration`
  - `realtimeproducts.yml` → `/realtimeproducts`
  - `whatwedo.yml` → home page panels
  - `metrics.yml` → home page counters
- **`content/<section>/`**: `_index.md` front matter sets the title and banner image for each section. Individual pages (`projects/*.md`, `software/*.md`, `news/*.md`, `datasets/*.md`) are rendered by `single.html` for that section and listed on the section page and home page. Software pages use front-matter fields such as `tagline`, `docs`, `repo` and `image`.
- **`themes/ncar/`**: the active theme lives in this repo (not a submodule). It has one layout directory per content section, plus `partials/` for the home page and About page blocks. A new section needs a matching layout directory here. `themes/roxo-hugo` is present but not used.
- **`static/images`, `static/pdfs`**: assets referenced by relative paths such as `images/projects/foo.png`.

## Metrics auto-update

`data/metrics.yml` counters are derived data. `counter_item[0..2]` hold the counts of `publications.yml`, `realtimeproducts.yml` and `collaboration.yml` items, in that order. On push to `main` that touches those files, `.github/workflows/update-metrics.yaml` recomputes the counts and commits "Update metrics.yml from changes to data". Run `update_metrics.py` locally, or let CI do it. Don't hand-edit the counts, and keep the order of the three counter items, because the script indexes them by position.

## Deployment

Pushes to `main` build and deploy through two workflows: `hugo.yml` (the actions/deploy-pages flow) and the older `ghpages.yml` (peaceiris, which pins Hugo 0.110.0 and sets the CNAME). PRs to `main` run the `ghpages.yml` build as a check.
