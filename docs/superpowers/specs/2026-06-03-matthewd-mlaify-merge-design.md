# matthewd.xyz × mlaify merge — design spec

**Date:** 2026-06-03
**Author:** Matthew A. Davis
**Status:** Draft, awaiting approval

## Summary

I'm merging `mlaify.github.io` into `mdavistffhrtporg.github.io` (matthewd.xyz). The result is a single, re-vamped personal site that hosts my writing, my projects (Aegis, AttackMap, OmekaRapper, OpenSift, OpenContractRx), and my "about" surface as peers — not a blog with projects bolted on, not a product hub with a bio bolted on. mlaify.io 301-redirects to matthewd.xyz and the mlaify repo is archived.

## Goals

1. One canonical site at matthewd.xyz. No content duplication, no domain split-brain.
2. Personal site and projects are visual peers. Neither is subordinate.
3. Re-vamped visual identity — copper as a distinctive personal accent, project accents preserved.
4. Preserve everything that was on mlaify: protocol/architecture pages, status badges, threat-model framing, build principles, contribute.
5. Modernize the theme — replace PaperMod with mlaify's Tailwind + Pagefind theme as the foundation.
6. Personal voice throughout ("I", not "we"/"mlaify").

## Non-goals

- Photography surface. I've medically retired from professional photography. Existing photo posts stay as posts; no `/photography` gallery.
- New visual identity for Aegis or AttackMap. Their existing accents (cyan, amber) stay.
- Framework change. Staying on Hugo.
- Sunset of mlaify.io domain. The domain stays registered and 301s everything to matthewd.xyz.

## Decisions (locked)

| # | Question | Decision |
|---|----------|----------|
| 1 | Site identity | Personal + product hub, peers |
| 2 | mlaify.io fate | 301 redirect everything to matthewd.xyz |
| 3 | Theme foundation | Port mlaify's custom Tailwind theme; extend with blog layouts |
| 4 | Project URL structure | Top-level (`/aegis`, `/attackmap`, …) — 1:1 redirect from mlaify.io |
| 5 | Homepage layout | Editorial hero stack (single narrative path: who → what I build → what I've written) |
| 6 | Hero tagline shape | Multi-hat — three short tags, e.g., "OSS contributor · Writer at Fedora Magazine · Builds security-first tooling" |
| 7 | Photography surface | None — stays inline as posts |
| 8 | Visual identity | Bold copper personal accent (`#b45309` family); warm-stone neutrals; product accents preserved |
| 9 | Top nav | Writing · Projects · About (3 items + search/theme toggle) |
| 10 | Other mlaify content | All of it carries: smaller projects, build-principles, contribute, about umbrella copy |
| 11 | Voice | First person ("I"), personal |
| 12 | Execution shape | Phased on a `redesign` branch, three independently shippable phases |

## Information architecture

```
matthewd.xyz/
├─ /                            Homepage — editorial hero stack
├─ /writing/                    Blog index (renamed from /posts; old URL 301s)
│  └─ /writing/<slug>/          Posts (existing 6 + future)
├─ /aegis/                      ← mlaify.io/aegis 1:1
│  └─ /aegis/{architecture,faq,getting-started,protocol,security,apps,release-notes}/
├─ /attackmap/                  ← mlaify.io/attackmap 1:1
│  └─ /attackmap/{architecture,faq,getting-started,sdk,analyzers,use-cases}/
├─ /projects/                   Hub: all 5 projects with status badges
├─ /omekarapper/                ← mlaify.io/docs/project-omekarapper (rewrite)
├─ /opensift/                   ← mlaify.io/docs/project-opensift (rewrite)
├─ /opencontractrx/             ← mlaify.io/docs/project-opencontractrx (rewrite)
├─ /about/                      Merged: mlaify umbrella copy + my /me content (personal voice)
├─ /principles/                 ← mlaify /build-principles
├─ /contribute/                 ← mlaify /contribute
├─ /search/                     Pagefind
├─ /tags/, /categories/         Hugo taxonomies (kept)
├─ /privacy-policy/, /security/ Kept
├─ /humans.txt
├─ /.well-known/security.txt
└─ /privacy.json
```

Top nav: `Writing · Projects · About`. Right-aligned: search icon + dark-mode toggle.

## Theme & visual system

### Foundation

The Hugo theme is the mlaify theme, ported into this repo (not consumed as a Hugo module). Replaces PaperMod entirely. `themes/PaperMod` and the `.gitmodules` reference are removed.

Stack:
- **Hugo** (extended) — already in CI
- **Tailwind CSS 3** — content sources: `layouts/**/*.html`, `content/**/*.md`, `assets/js/**/*.js`
- **Pagefind 1.x** — static search index built after Hugo build
- **PostCSS + Autoprefixer**
- **Prettier** for formatting
- **Inter** (UI/headings), **JetBrains Mono** (code)

### Palette

Tailwind theme extension (in `tailwind.config.js`):

```js
colors: {
  // Warm neutrals (replaces ink slate from mlaify)
  stone: {
    50:  '#fafaf9',
    100: '#f5f5f4',
    200: '#e7e5e4',
    300: '#d6d3d1',
    400: '#a8a29e',
    500: '#78716c',
    600: '#57534e',
    700: '#44403c',
    800: '#292524',
    900: '#1c1917',
    950: '#0c0a09',
  },
  // Personal accent — copper
  copper: {
    50:  '#fff7ed',
    100: '#ffedd5',
    200: '#fed7aa',
    300: '#fdba74',
    400: '#fb923c',
    500: '#f97316',
    600: '#ea580c',
    700: '#b45309', // primary
    800: '#92400e',
    900: '#7c2d12',
  },
  // Product accents — preserved from mlaify
  aegis:     { /* cyan scale, kept */ },
  attackmap: { /* amber scale, kept */ },
  brand: { /* alias of copper for theme compatibility */ },
}
```

Copper is used for: nav-active state, link color, blockquote left rule, button primary fill, focus rings, inline `<code>` background tint.

### Typography

- **Inter** — UI + headings + body. (Considered serif for long-form posts; rejected because it conflicts visually with product pages and complicates the type system.)
- **JetBrains Mono** — code blocks, inline code, status badges.
- **Display** — Inter at heavier weight for h1/h2.

### Dark mode

`darkMode: 'class'` (Tailwind class strategy). System default with manual toggle, persisted in `localStorage`. Existing Giscus comments partial already responds to theme changes via `postMessage` — no change needed.

### Shortcodes (ported from mlaify, kept)

`status`, `callout`, `cta`, `feature`, `feature-grid`, `app-badges`, `repo-link`. Used heavily on project pages. No new shortcodes added in this spec.

### New layouts (extending mlaify's theme)

| Layout | Purpose | Source |
|--------|---------|--------|
| `layouts/index.html` | Homepage — editorial hero stack | Rebuilt from mlaify's index |
| `layouts/_default/single.html` | Generic single page | Extended from mlaify's |
| `layouts/posts/single.html` (or `writing/single.html`) | Blog post — cover image, reading time, tags, share, prev/next, Giscus | New |
| `layouts/posts/list.html` | `/writing/` index | New |
| `layouts/projects/list.html` | `/projects/` hub with status badges | New |
| `layouts/partials/comments.html` | Existing Giscus partial | Carries over verbatim |
| `layouts/partials/share.html` | Share buttons | New |
| `layouts/partials/post-meta.html` | Reading time, tags, author | New |

## Homepage anatomy

```
┌────────────────────────────────────────────────────────────────┐
│  matthewd.xyz       Writing  Projects  About    🔍   ☀/🌙       │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  (avatar)   Matthew A. Davis                                   │
│             OSS contributor · Writer at Fedora Magazine        │
│             · Builds security-first tooling                    │
│             [ Read writing ]   About →                         │
│                                                                │
├────────────────────────── PROJECTS ────────────────────────────┤
│  ┌─ Aegis ─────────────┐   ┌─ AttackMap ──────────┐            │
│  │ alpha · cyan        │   │ alpha · amber        │            │
│  │ Post-quantum E2E    │   │ Local-first          │            │
│  │ messaging.          │   │ defensive analysis.  │            │
│  └─────────────────────┘   └──────────────────────┘            │
│  [OmekaRapper]  [OpenSift]  [OpenContractRx]                   │
│                                                  All projects →│
├────────────────────── LATEST WRITING ──────────────────────────┤
│  • D850 thoughts            Dec 27, 2025                       │
│  • Happy Holidays           Dec 24, 2025                       │
│  • LUKS2 & TPM2             Apr 4,  2024                       │
│                                                  All writing → │
├────────────────────────────────────────────────────────────────┤
│  footer: © 2026 Matthew A. Davis · CC BY-NC-SA 4.0 (content)   │
│  · MIT (code) · Privacy · Security · Humans · GitHub Pages     │
└────────────────────────────────────────────────────────────────┘
```

Tagline starting point: **"OSS contributor · Writer at Fedora Magazine · Builds security-first tooling."** — to be wordsmithed before ship.

## Content migration

### Files updated

1. **`content/me.md` → `content/about/_index.md`**
   - Rewrite to remove "professional photographer" lead.
   - Merge selected text from mlaify's `content/about/_index.md` (umbrella framing), recast in first person.
   - Preserve education (UTA Psychology BA 2010, UNT MS Interdisciplinary Studies 2019), Fedora Magazine role, IT/cybersecurity background, OSS-since-2003 longevity.
   - New lead: a paragraph about what I work on now (OSS, security tooling, writing) and what I believe (the build-principles values in personal voice).

2. **`content/posts/` → `content/writing/`** — section rename.
   - Existing posts move file-by-file.
   - Front-matter `aliases` added to each post for old `/posts/<slug>/` URLs (Hugo native alias = lightweight redirect).
   - Existing `content/archives.md` updated if it hardcodes the section path.

3. **mlaify content imported verbatim**, then voice-edited:
   - `content/aegis/` (8 files including `_index.md`)
   - `content/attackmap/` (7 files including `_index.md`)
   - `content/docs/project-omekarapper.md` → `content/omekarapper/_index.md`
   - `content/docs/project-opensift.md` → `content/opensift/_index.md`
   - `content/docs/project-opencontractrx.md` → `content/opencontractrx/_index.md`
   - `content/build-principles/_index.md` → `content/principles/_index.md`
   - `content/contribute/_index.md` → `content/contribute/_index.md`
   - mlaify's `content/about/_index.md` — text salvaged, merged into `content/about/_index.md` (above)

4. **New: `content/projects/_index.md`** — hub page listing all 5 projects.

### Voice edits (personal voice pass)

Every imported page needs a "we"/"mlaify" → "I" pass:

- `aegis/_index.md` — "Aegis is …" stays (product description), but "we hedge against …" → "I hedge against …" / "the design hedges against …"
- `attackmap/_index.md` — same treatment.
- `principles/_index.md` (was build-principles) — significant rewrite. Currently "every mlaify project follows…" → "every project of mine follows…". Pronouns throughout: "we build" → "I build".
- `contribute/_index.md` — full rewrite for personal voice.
- Project status framing stays honest — "alpha means alpha" is mine to keep.

### Standards files reconciliation

Each repo has its own `SECURITY.md`, `humans.txt`-equivalent content, etc. matthewd.xyz's versions are canonical. If mlaify's diverged (different vuln-disclosure language, different email addresses), reconcile to the matthewd.xyz versions. Verify:

- `SECURITY.md` — already in matthewd repo, uses matthewd email.
- `static/humans.txt` — verify exists and matches CLAUDE.md template; update domain + last-updated.
- `static/.well-known/security.txt` — verify exists; update policy URL to `https://matthewd.xyz/security`.
- `static/privacy.json` — verify exists.

## Redirect strategy

mlaify.io serves a static `_redirects` (or Cloudflare Rules) implementing:

| From | To | Type |
|------|-----|------|
| `mlaify.io/` | `matthewd.xyz/projects/` | 301 |
| `mlaify.io/aegis(/.*)?` | `matthewd.xyz/aegis$1` | 301 |
| `mlaify.io/attackmap(/.*)?` | `matthewd.xyz/attackmap$1` | 301 |
| `mlaify.io/docs/project-omekarapper(/.*)?` | `matthewd.xyz/omekarapper$1` | 301 |
| `mlaify.io/docs/project-opensift(/.*)?` | `matthewd.xyz/opensift$1` | 301 |
| `mlaify.io/docs/project-opencontractrx(/.*)?` | `matthewd.xyz/opencontractrx$1` | 301 |
| `mlaify.io/build-principles(/.*)?` | `matthewd.xyz/principles$1` | 301 |
| `mlaify.io/contribute(/.*)?` | `matthewd.xyz/contribute$1` | 301 |
| `mlaify.io/about(/.*)?` | `matthewd.xyz/about$1` | 301 |
| `mlaify.io/security(/.*)?` | `matthewd.xyz/security` | 301 |
| `mlaify.io/privacy(/.*)?` | `matthewd.xyz/privacy-policy` | 301 |
| `mlaify.io/.well-known/security.txt` | `matthewd.xyz/.well-known/security.txt` | 301 |
| `mlaify.io/humans.txt` | `matthewd.xyz/humans.txt` | 301 |
| `mlaify.io/privacy.json` | `matthewd.xyz/privacy.json` | 301 |
| `mlaify.io/(.*)` | `matthewd.xyz/$1` | 301 catch-all |

Internal redirect (matthewd.xyz):

| From | To | Mechanism |
|------|-----|-----------|
| `/posts/<slug>/` | `/writing/<slug>/` | Hugo `aliases` front-matter on each post |
| `/me/` | `/about/` | Hugo alias on `content/about/_index.md` |

## Comments — Giscus

Giscus is already wired in `layouts/partials/comments.html`:
- Repo: `mdavistffhrtporg/mdavistffhrtporg.github.io`
- Repo ID: `R_kgDOMNpTyg`
- Category: `Announcements` (`DIC_kwDOMNpTys4C0Q3O`)
- Mapping: `pathname`
- Theme: dynamic (responds to dark-mode toggle via `postMessage`)

Work to do in the new theme:

1. In `layouts/posts/single.html` (or `writing/single.html`), include the partial **only on posts** (`{{ partial "comments.html" . }}`), gated by `{{ if .Params.comments | default true }}`.
2. Do **not** render comments on:
   - Homepage
   - `/about/`, `/principles/`, `/contribute/`, `/privacy-policy/`, `/security/`
   - Project landing pages (`/aegis/`, `/attackmap/`, …) — sub-pages off; landing pages off
   - Taxonomy / list pages
3. Each page using comments needs the `<div class="giscus"></div>` mount point inside the partial-include block.
4. Verify GitHub Discussions is still enabled on the repo with the `Announcements` category present. **Action item before ship:** confirm in repo settings.
5. Dark-mode integration: the new theme's toggle must `postMessage({type: 'theme-changed'})` to the document on toggle (or the existing partial's listener won't fire). The partial currently reads `document.body.classList.contains('dark')` — the new toggle must apply `.dark` to `<body>`.

**Open question to validate during implementation:** does the existing repo-id / category-id pair still match? If GitHub re-issued them (unlikely but possible), regenerate at giscus.app/configure.

## Search — Pagefind

Replaces PaperMod's Fuse.js search.

- `npm install pagefind` already in mlaify's `package.json` — port to root `package.json`.
- Build step: `pagefind --site public` runs after `hugo --minify --gc`.
- Search UI: port `layouts/_partials/search-dialog.html` from mlaify.
- Header search icon opens the dialog; `Cmd/Ctrl+K` keyboard shortcut.
- Index excludes: footer, header, comments, code-block line numbers (via `data-pagefind-ignore` on those wrapper elements in the layouts).
- `content/search.md` (currently a PaperMod Fuse page) is removed.

## CI / GitHub Actions

The new workflow at `.github/workflows/hugo.yml` merges both source workflows. Key changes from current matthewd workflow:

1. Add Node dependency install (`npm ci`) — mlaify uses, matthewd's `package-lock.json` doesn't exist yet (will be created).
2. Add Tailwind build step — invoked via Hugo's Tailwind pipe or a pre-step `npm run build:css`.
3. Add Pagefind index step after Hugo build: `npx pagefind --site public`.
4. Keep Dart Sass install (Hugo Pipes asset processing).
5. Keep Hugo cache restore/save.
6. Keep Go setup (Hugo modules, if any).
7. Pin: Hugo 0.153.2 (current), Node 24.12.0 (current), Dart Sass 1.97.1 (current), Pagefind 1.4.0 (from mlaify).

mlaify's repo workflow will be replaced with a static redirect deploy or the repo archived (mlaify.io's hosting moves to a Cloudflare-Pages-or-Workers redirect setup — see Redirect strategy).

### Local dev

```bash
npm install
npm run dev      # hugo server --disableFastRender --noHTTPCache
npm run build    # hugo --minify --gc && pagefind --site public
npm run format   # prettier -w
```

Ported from mlaify's `package.json`.

## Phased rollout

Work happens on a `redesign` branch. `main` keeps serving the current PaperMod site until cutover.

### Phase 1 — Theme port (own PR)

Goal: new theme rendering the existing matthewd.xyz content at parity. No content additions yet.

- Remove PaperMod + `.gitmodules` submodule reference.
- Add `package.json`, `tailwind.config.js`, `postcss.config.js`, `assets/`, `layouts/` from mlaify (minus mlaify-specific content).
- Add `layouts/posts/single.html`, `layouts/posts/list.html` for blog rendering.
- Update `hugo.yaml` for new theme (no theme dir; layouts in repo root).
- Update `.github/workflows/hugo.yml` per CI section above.
- Recreate homepage (editorial hero stack) with existing posts only.
- Wire Giscus partial into posts.
- Apply copper palette + warm-stone neutrals.
- Verify Pagefind search works.
- Verify dark mode + comments theme sync.

**Ship gate:** all current matthewd.xyz pages render in the new theme on the branch's preview; Giscus works; Pagefind returns results.

### Phase 2 — Content migration (own PR)

Goal: mlaify content lives on matthewd.xyz.

- Import all mlaify `content/` (aegis, attackmap, smaller projects, build-principles, contribute, about).
- Voice-edit pass: "we" → "I", "mlaify" → my name / personal framing.
- Move `content/posts/` → `content/writing/`. Add `aliases:` front-matter.
- Move `content/me.md` content into `content/about/_index.md`.
- Create `content/projects/_index.md` hub.
- Wire top nav: Writing · Projects · About.
- Update CHANGELOG.md (Keep a Changelog).

**Ship gate:** all mlaify URLs reachable on matthewd.xyz under new paths; nav works; project status badges render; CHANGELOG entry added.

### Phase 3 — Redirects + sunset (own PR + ops work)

Goal: mlaify.io stops being a live destination; matthewd.xyz is canonical.

- Configure Cloudflare (or wherever mlaify.io DNS terminates) with the redirect map above. **Cannot land via PR alone — requires DNS/Cloudflare access.**
- Archive `mlaify/mlaify.github.io` repo (read-only on GitHub).
- Update external references where I control them:
  - GitHub repo READMEs in `mlaify/*` repos referencing mlaify.io → matthewd.xyz.
  - Package metadata (PyPI `attackmap`, etc.) if site URL was set to mlaify.io.
  - Bluesky / GitLab profile links if they point to mlaify.io.
- Smoke-test top redirects: aegis, attackmap, principles, root.

**Ship gate:** mlaify.io paths 301 to correct matthewd.xyz paths; archive complete; no broken inbound links from my own properties.

## Risks & open items

| # | Item | Mitigation |
|---|------|------------|
| 1 | Giscus repo/category IDs stale | Verify during Phase 1; regenerate at giscus.app/configure if needed |
| 2 | mlaify.io DNS lives where? | Confirm before Phase 3 — Cloudflare vs. GitHub Pages CNAME. Affects redirect mechanism |
| 3 | External inbound links to specific mlaify.io subpaths I don't know about | Catch-all redirect (`mlaify.io/(.*)` → `matthewd.xyz/$1`) handles unknowns gracefully |
| 4 | Pagefind index size + GH Pages artifact limit | Should be fine (<1MB), but monitor first build |
| 5 | Tagline copy ("multi-hat") needs wordsmithing | Block on it before Phase 1 ship — placeholder is fine for branch |
| 6 | `principles/_index.md` voice rewrite is the biggest content edit | Schedule before Phase 2 ship gate; review once before merging |
| 7 | CONTENT_LICENSE.md / LICENSE handling for mlaify content | Both repos use the same licenses (MIT code + CC BY-NC-SA content); no conflict |

## Out of scope (deferred)

- Photography gallery / portfolio surface.
- RSS feed redesign beyond what Hugo defaults provide.
- Newsletter signup.
- Comments on project pages.
- Analytics (currently disabled in `hugo.yaml`; staying off).
- Per-product subdomains (e.g., aegis.matthewd.xyz). Not warranted.
- Migration of GitHub Discussions data into any other system.

## Acceptance

This spec is approved when:
- All 12 locked decisions are reflected.
- Phasing is acceptable to me.
- Open items list is acknowledged.

After approval, hand off to `writing-plans` to produce the implementation plan (per-phase tasks, dependencies, verification steps).
