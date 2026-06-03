# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- New theme foundation: Tailwind CSS 3, Pagefind 1.4 search, JetBrains Mono + Inter fonts.
- Personal copper accent palette (`#b45309` family) site-wide; Aegis (cyan) and AttackMap (amber) accents preserved for Phase 2.
- Editorial hero-stack homepage with avatar, tagline, latest-writing list.
- Blog post layout with cover image, post-meta (reading time, tags, date), share row (Bluesky + copy link), prev/next, and Giscus comments.
- Blog list layout with paginated two-column post grid.
- 404 page.
- Shortcodes ported from mlaify in preparation for Phase 2: `callout`, `status`, `cta`, `feature`, `feature-grid`, `app-badges`, `repo-link`.
- Hugo config split into `config/_default/*.toml` files (hugo, module, markup, params, languages, menus).
- All mlaify content imported with first-person voice edits: Aegis (8 pages), AttackMap (7 pages), OmekaRapper, OpenSift, OpenContractRx.
- `/principles/` page (formerly mlaify's `/build-principles/`), recast in first-person.
- `/contribute/` page recast in first-person with matthewd@matthewd.xyz as the contact.
- `/projects/` hub page listing all 5 projects with status badges, sorted by weight.
- Projects section on the homepage (top 2 as cards, smaller 3 as pill links).
- `[products]` table in `params.toml` configuring all 5 projects (name, tagline, status, accent, URL, repo URL).

### Changed
- Replaced PaperMod theme with a port of mlaify's custom Hugo theme.
- Moved Giscus partial from `layouts/partials/` to `layouts/_partials/`.
- CI: added Node, `npm ci`, and Pagefind index steps to GitHub Pages workflow.
- Footer "Archives" link now points to `/posts/` (the working blog index); `/archives/` 301-redirects there via Hugo alias for backward compatibility.
- `/posts/` renamed to `/writing/`. Old URLs (`/posts/<slug>/`) 301-redirect via Hugo aliases. `/archives/` → `/writing/`.
- `/me/` renamed to `/about/`. Old URL redirects via alias. Body rewritten to remove the "professional photographer" lead (medical retirement), merge mlaify's umbrella copy in first-person, and link to `/principles/` for the longer story.
- Top nav: Writing · Projects · About (was Posts · About). Footer gains Projects, Principles, and Contribute entries.

### Removed
- PaperMod theme submodule.
- `hugo.yaml` and `hugo_bak.yaml` (replaced by `config/_default/`).
- `content/search.md` (PaperMod Fuse-based search page; replaced by Pagefind).
- `content/posts/` directory (moved to `content/writing/`).
- `content/me.md` (folded into `content/about/_index.md`).
