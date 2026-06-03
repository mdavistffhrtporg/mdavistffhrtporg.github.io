# Phase 2 — Content Migration: Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Bring all mlaify content onto matthewd.xyz under its new URL structure, with a personal-voice edit pass, and surface it through an updated nav (Writing · Projects · About), a `/projects/` hub, and a projects section on the homepage.

**Architecture:** Each piece of mlaify content lands as a Hugo content file under matthewd.xyz with appropriate front-matter and the same body (lightly edited for voice). Section renames use Hugo `aliases` for 301 backward-compatibility (`/posts/` → `/writing/`, `/me/` → `/about/`). Aegis and AttackMap keep their existing internal URL structure (e.g., `/aegis/architecture/`); smaller projects (OmekaRapper, OpenSift, OpenContractRx) move from mlaify's `/docs/project-<x>` to top-level matthewd.xyz `/<x>/`. The `/projects/` hub iterates `site.Params.products` to list all five with status badges.

**Tech Stack:** Same as Phase 1 — Hugo, Tailwind, Pagefind, the ported mlaify theme. No new dependencies.

**Scope boundary:** Phase 2 ships content. It does NOT touch mlaify.io domain / DNS — that's Phase 3. mlaify.io continues serving its current content until Phase 3 redirects fire. So during Phase 2, the same content exists at both matthewd.xyz/aegis and mlaify.io/aegis temporarily; that's intentional.

**Source repo paths (read-only):**
- mlaify: `/Volumes/Dev/repos/GitHub/mlaify/mlaify.github.io`

**Working repo:**
- matthewd: `/Volumes/Dev/repos/GitHub/mdavistffhrtporg/mdavistffhrtporg.github.io`

When this plan says "copy from mlaify", it means copy from the mlaify path.

**Voice convention used throughout:** First person singular ("I"), present tense. Replace "we" → "I". Replace "mlaify" with appropriate alternative (often "I" or the specific project name, e.g., "mlaify ships X" → "I ship X" or "Aegis is X"). The plan gives specific guidance per content file.

---

## File map

### Created

```
content/writing/                                # Renamed from /posts/ — files move below
content/writing/post-{1,2,3,4,5,6}/...          # 6 existing posts (moved, not duplicated)

content/aegis/_index.md                         # 8 files imported from mlaify, voice-edited
content/aegis/{apps,architecture,faq,getting-started,protocol,release-notes,security}.md

content/attackmap/_index.md                     # 7 files imported from mlaify, voice-edited
content/attackmap/{analyzers,architecture,faq,getting-started,sdk,use-cases}.md

content/omekarapper/_index.md                   # From mlaify docs/project-omekarapper.md
content/opensift/_index.md                      # From mlaify docs/project-opensift.md
content/opencontractrx/_index.md                # From mlaify docs/project-opencontractrx.md

content/principles/_index.md                    # From mlaify build-principles, heavy voice rewrite
content/contribute/_index.md                    # From mlaify contribute, voice rewrite
content/about/_index.md                         # From content/me.md + mlaify about, heavy rewrite

content/projects/_index.md                      # New hub page listing all 5 projects

layouts/projects/list.html                      # New layout for the /projects/ hub
```

### Modified

```
content/posts/                                  # DELETED (moved to /writing/)
content/me.md                                   # DELETED (folded into content/about/_index.md)

config/_default/menus/menus.en.toml             # Nav updated: Writing · Projects · About
config/_default/params.toml                     # products table populated with all 5

layouts/index.html                              # Projects section added (was deferred in Phase 1)
layouts/_partials/header.html                   # No code changes — nav is menu-driven

CHANGELOG.md                                    # Phase 2 entry under [Unreleased]
```

### Untouched

```
layouts/_partials/{head,footer,search-dialog,svg-icon,...}   # Same as Phase 1
layouts/posts/{single,list}.html                              # Renamed to writing/ in Task 2
assets/, tailwind.config.js, postcss.config.js, etc.          # No theme changes
static/                                                       # Static files unchanged
.github/workflows/hugo.yml                                    # CI already builds correctly
```

---

## Voice-edit conventions

For every content file imported from mlaify, apply these substitutions (in order). Most are mechanical; the heavy rewrites (about, principles, contribute) get explicit per-file guidance below.

| Original | Replacement | Notes |
|----------|-------------|-------|
| `we ` (lowercase, with trailing space) | `I ` | Sentence-internal "we" |
| `We ` | `I ` | Sentence-starting "We" |
| `we'll`, `we've`, `we're`, `we'd` | `I'll`, `I've`, `I'm`, `I'd` | Contractions |
| `our ` | `my ` | Possessive |
| `Our ` | `My ` | Sentence-starting |
| `us ` | `me ` | Object pronoun |
| `mlaify ships` | `I ship` | "mlaify ships X" → "I ship X" |
| `mlaify is the home of` | `Home of` | Strip the org-as-subject framing |
| `mlaify projects` | `my projects` | Possessive replacement |
| `at mlaify` | `here` or just remove | Drop locational reference |
| `[mlaify]` (link text in `[mlaify](url)`) | `me` or replace with project name | Context-dependent — review each |

**Don't blindly find-replace** "mlaify" everywhere — some references are intentional (e.g., the GitHub org name `mlaify/aegis` in code blocks and URLs). The substitution rules apply to prose only. The implementer must read each file and apply judgment, NOT run `sed`.

After substitution, re-read each file and check:
- Does the voice flow as if a single person is writing?
- Are claims about identity ("zero-trust") still accurate when scoped to "I" instead of "we"?
- Did "mlaify" appear in code blocks (e.g., `github.com/mlaify/aegis`)? Those stay — they're URL strings.

---

## Task 1: Create phase-2 branch from main

**Files:** none (branch creation only)

- [ ] **Step 1: Sync with origin**

```bash
git checkout main
git pull origin main
git log -1 --oneline
```

Expected: HEAD is at the latest Phase 1 commit (post-hot-fix), `a483aa1` or newer.

- [ ] **Step 2: Create branch**

```bash
git checkout -b phase-2-content-migration
git status
```

Expected: clean tree on `phase-2-content-migration`.

- [ ] **Step 3: No commit yet — Task 2 makes the first commit**

---

## Task 2: Rename `/posts/` → `/writing/` with backward-compatible aliases

**Files:**
- Move: `content/posts/*` → `content/writing/*`
- Move: `layouts/posts/{single,list}.html` → `layouts/writing/{single,list}.html`
- Modify: each `content/writing/post-N/index.md` — add `aliases:` to the old URL

The blog section is the most-trafficked URL space. Aliases preserve every existing bookmark.

- [ ] **Step 1: Move the content directory**

```bash
git mv content/posts content/writing
```

Hugo's section name follows the directory, so `content/writing/post-6/` will resolve to `/writing/post-6/` automatically.

- [ ] **Step 2: Add `aliases` front-matter to each post**

For each of `post-1`, `post-2`, `post-3`, `post-4`, `post-5`, `post-6`:

Read `content/writing/<slug>/index.md`. The existing front-matter looks like:

```yaml
---
cover:
  image: "images/jakob-owens-unsplash.jpg"
  alt: "..."
date: 2024-04-01T00:00:00-00:00
description: "Fedora Silverblue"
---
```

Add an `aliases:` key immediately after `description`:

```yaml
aliases:
  - /posts/<slug>/
```

Where `<slug>` is `post-1`, `post-2`, etc. So post-1's front-matter ends with:

```yaml
description: "Fedora Silverblue"
aliases:
  - /posts/post-1/
---
```

Also add an alias on the section index (which doesn't currently exist as a separate file — Hugo will auto-create the list page). To make sure `/posts/` itself redirects:

Create `content/writing/_index.md`:

```yaml
---
title: "Writing"
description: "Articles I have written."
aliases:
  - /posts/
---

Here are some articles that I have written.
```

This replaces the (now-deleted) `content/posts/_index.md` that had its alias of `/archives/` from Phase 1's hot-fix. **Important:** the `/archives/` alias must be preserved here:

```yaml
---
title: "Writing"
description: "Articles I have written."
aliases:
  - /posts/
  - /archives/
---

Here are some articles that I have written.
```

- [ ] **Step 3: Move the posts layouts to writing/**

```bash
git mv layouts/posts layouts/writing
```

This shifts the layouts so Hugo associates them with the new section. The blog post layout still uses `_partials/post-meta.html` and `_partials/share.html` — those don't need changes.

- [ ] **Step 4: Update the homepage's `where ... "Section" "posts"` to `"writing"`**

Open `layouts/index.html`. Find this line (around line 28):

```html
  {{- $posts := first 5 (where site.RegularPages "Section" "posts") }}
```

Change to:

```html
  {{- $posts := first 5 (where site.RegularPages "Section" "writing") }}
```

- [ ] **Step 5: Update the blog list layout's section filter**

Open `layouts/writing/list.html`. Find this line (around line 13):

```html
  {{- $paginator := .Paginate (where .Pages "Section" "posts") }}
```

Change to:

```html
  {{- $paginator := .Paginate (where .Pages "Section" "writing") }}
```

- [ ] **Step 6: Smoke build**

```bash
npm run build 2>&1 | tail -15
```

Verify:
- Build succeeds (Pagefind indexes pages).
- `public/writing/post-6/index.html` exists.
- `public/posts/post-6/index.html` is a meta-refresh redirect to `/writing/post-6/`.
- `public/archives/index.html` redirects to `/writing/`.

```bash
ls public/writing/post-1/index.html public/writing/post-6/index.html
grep "http-equiv=refresh" public/posts/post-1/index.html public/posts/post-6/index.html
grep "http-equiv=refresh" public/archives/index.html
```

Expected: file listings show all 6 posts under `/writing/`; the redirect grep returns matches for each.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "Rename /posts/ → /writing/ with backward-compat aliases

Section directory moved (content/posts/ → content/writing/), layouts
moved (layouts/posts/ → layouts/writing/), homepage and list-page
section filters updated.

Each old post URL (/posts/<slug>/) redirects to its new URL via
Hugo's aliases front-matter. /archives/ continues to redirect to
the writing index (was /posts/, now /writing/).

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 3: Rename `/me/` → `/about/` with alias

**Files:**
- Move: `content/me.md` → `content/about/_index.md`
- Modify: that file's front-matter (add alias)

This sets up the location for the rewritten About page in Task 9. For now, the move-and-alias keeps the existing content reachable.

- [ ] **Step 1: Create the directory and move the file**

```bash
mkdir -p content/about
git mv content/me.md content/about/_index.md
```

- [ ] **Step 2: Add `aliases:` to the moved file's front-matter**

Open `content/about/_index.md`. The current front-matter is:

```yaml
---
title: "Matthew A. Davis"

description: "My personal website"
cascade:
comments: false
---
```

Modify to:

```yaml
---
title: "About"

description: "About Matthew A. Davis."
aliases:
  - /me/
cascade:
  comments: false
---
```

Two changes: title becomes "About", and `aliases: [/me/]` so `/me/` redirects to `/about/`. The `cascade.comments: false` is preserved (no comments on the about page or its children).

**Important:** do NOT touch the body content yet — Task 9 rewrites the full About page. Right now we're just moving the file so URLs work.

- [ ] **Step 3: Smoke build**

```bash
npm run build 2>&1 | tail -10
ls public/about/index.html public/me/index.html
grep "http-equiv=refresh" public/me/index.html
```

Expected: both exist; `/me/` is a redirect to `/about/`.

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "Move /me/ to /about/ section with alias

Content stays the same for now — this just sets the URL. /me/
redirects to /about/. Task 9 will rewrite the body to merge in
mlaify's umbrella copy and remove the 'professional photographer'
lead (medical retirement, per the merge design spec).

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 4: Populate `[products]` in params.toml

**Files:**
- Modify: `config/_default/params.toml`

The Phase 1 params.toml left `[products]` empty. The `/projects/` hub (Task 11) and homepage projects section (Task 12) iterate this table, so it needs entries for all 5 projects.

- [ ] **Step 1: Replace the empty `[products]` line at the bottom of `config/_default/params.toml`**

Open the file. Find:

```toml
# Empty products table — populated in Phase 2
[products]
```

Replace with:

```toml
# Per-project params used by the /projects/ hub and homepage projects section.
[products]
  [products.aegis]
    name = "Aegis"
    tagline = "Post-quantum encrypted messaging, end to end."
    status = "alpha"
    accent = "aegis"
    url = "/aegis/"
    repoUrl = "https://github.com/mlaify/aegis"
    weight = 10

  [products.attackmap]
    name = "AttackMap"
    tagline = "Local-first defensive security analysis for real codebases."
    status = "alpha"
    accent = "attackmap"
    url = "/attackmap/"
    repoUrl = "https://github.com/mlaify/attackmap"
    weight = 20

  [products.omekarapper]
    name = "OmekaRapper"
    tagline = "AI-assisted cataloging built into the Omeka admin flow."
    status = "alpha"
    accent = "brand"
    url = "/omekarapper/"
    repoUrl = "https://github.com/mlaify/omekarapper"
    weight = 30

  [products.opensift]
    name = "OpenSift"
    tagline = "Quiz-mode study assistant for self-directed learners."
    status = "alpha"
    accent = "brand"
    url = "/opensift/"
    repoUrl = "https://github.com/mlaify/opensift"
    weight = 40

  [products.opencontractrx]
    name = "OpenContractRx"
    tagline = "Hospital pharmacy contract review, structured."
    status = "alpha"
    accent = "brand"
    url = "/opencontractrx/"
    repoUrl = "https://github.com/mlaify/opencontractrx"
    weight = 50
```

Note: Aegis and AttackMap use their own accents (`aegis`, `attackmap` — defined as CSS custom-property scopes in `assets/css/app.css`). The smaller projects use `brand` (the copper site accent) since they don't have dedicated palettes.

**Open question for the user during this task:** Are the small-project repo URLs (`mlaify/omekarapper`, `mlaify/opensift`, `mlaify/opencontractrx`) correct? Check that the repos exist at those paths before committing. If they live under a different org or name, update the `repoUrl` entries. Don't block the commit — flag in your report and continue.

- [ ] **Step 2: Verify the TOML parses**

```bash
python3 -c "
import tomllib
p = tomllib.load(open('config/_default/params.toml','rb'))
assert set(p['products'].keys()) == {'aegis','attackmap','omekarapper','opensift','opencontractrx'}, list(p['products'].keys())
print('OK: 5 products configured')
for k, v in sorted(p['products'].items(), key=lambda kv: kv[1]['weight']):
    print(f'  {v[\"weight\"]:>3} {k}: {v[\"name\"]} ({v[\"status\"]})')
"
```

Expected: `OK: 5 products configured` then 5 lines listing the products in weight order.

- [ ] **Step 3: Build (no rendered output change yet — we haven't built UI that consumes this)**

```bash
npm run build 2>&1 | tail -5
```

Expected: success.

- [ ] **Step 4: Commit**

```bash
git add config/_default/params.toml
git commit -m "Populate site.Params.products with all 5 projects

Aegis and AttackMap with their own accents; OmekaRapper, OpenSift,
OpenContractRx use the site copper accent. /projects/ hub (Task 11)
and homepage projects section (Task 12) iterate this table.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 5: Import Aegis content and voice-edit

**Files:**
- Create: `content/aegis/{_index,apps,architecture,faq,getting-started,protocol,release-notes,security}.md`

Aegis is a product. Its existing copy is product-objective (describing what Aegis is), so voice edits are minimal — mostly the `_index.md` and `apps.md` reference "we" in a few places.

- [ ] **Step 1: Copy all 8 Aegis files**

```bash
mkdir -p content/aegis
for f in _index apps architecture faq getting-started protocol release-notes security; do
  cp "/Volumes/Dev/repos/GitHub/mlaify/mlaify.github.io/content/aegis/$f.md" "content/aegis/$f.md"
done
ls content/aegis/
```

Expected: 8 files listed.

- [ ] **Step 2: Voice-edit `_index.md`**

Read `content/aegis/_index.md`. Apply the voice substitutions from the "Voice-edit conventions" section above. Specifically expect to find:

- `we hedge` → `I hedge` (or `the design hedges` if more natural)
- Any "mlaify ships Aegis" framing → "Aegis is" (already product-subject in most places)
- The `accent: "aegis"` front-matter stays

Then re-read the file and check it flows. For Aegis specifically, the product-subject voice is already natural ("Aegis is a zero-trust...") — most paragraphs need no edits.

- [ ] **Step 3: Voice-edit `apps.md`**

Look for the 2 "we"/"mlaify" occurrences. Likely "we publish" or "we maintain" referring to the app store presence. Replace with "I publish" / "I maintain" if first-person, or drop the framing if it doesn't add anything.

- [ ] **Step 4: Voice-edit `faq.md` and `security.md`**

3 and 2 occurrences respectively. Same pattern — first-person rewrite.

- [ ] **Step 5: Voice-edit `getting-started.md`**

4 occurrences. Likely instructional ("we recommend X" → "I recommend X").

- [ ] **Step 6: `architecture.md`, `protocol.md`, `release-notes.md`**

Search for "we" / "mlaify" — if any exist they're product-internal references. Edit if first-person, leave if not.

- [ ] **Step 7: Verify no stray "mlaify" prose references remain (URLs stay)**

```bash
grep -n "\\bmlaify\\b" content/aegis/*.md | grep -v "github.com/mlaify\\|gitlab.com/mlaify\\|mlaify/aegis\\|mlaify/attackmap"
```

Expected: empty output (only URL references survive). If any prose "mlaify" remains, fix it.

- [ ] **Step 8: Verify no stray first-person-plural remains**

```bash
grep -nE "\\b[Ww]e\\b|\\b[Oo]ur\\b|\\b[Uu]s\\b" content/aegis/*.md
```

Read each match in context — some uses of "we" may be intentional (e.g., quoting a spec that says "the protocol assumes we have..."). For incidental author-voice usage, change to "I". For technical/spec usage, leave.

- [ ] **Step 9: Build to confirm pages render**

```bash
npm run build 2>&1 | tail -5
ls public/aegis/index.html public/aegis/architecture/index.html public/aegis/protocol/index.html
```

Expected: 3 OK file listings.

- [ ] **Step 10: Commit**

```bash
git add content/aegis/
git commit -m "Import Aegis content with first-person voice edits

8 files (index + 7 sub-pages) imported from mlaify. Product-objective
copy is mostly unchanged; first-person references (\"we recommend\",
\"we maintain\") rewritten to \"I\". URLs to github.com/mlaify/aegis
preserved as-is — that's the repo URL, not a self-reference.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 6: Import AttackMap content and voice-edit

**Files:**
- Create: `content/attackmap/{_index,analyzers,architecture,faq,getting-started,sdk,use-cases}.md`

AttackMap has higher first-person density than Aegis — 28 "we" in `analyzers.md` alone. More voice-editing work.

- [ ] **Step 1: Copy all 7 files**

```bash
mkdir -p content/attackmap
for f in _index analyzers architecture faq getting-started sdk use-cases; do
  cp "/Volumes/Dev/repos/GitHub/mlaify/mlaify.github.io/content/attackmap/$f.md" "content/attackmap/$f.md"
done
ls content/attackmap/
```

- [ ] **Step 2: Voice-edit `_index.md`** (8 occurrences)

Read the file. Apply substitutions. Watch for "we" in:
- Marketing-y intro paragraphs
- "What we built" type framing — rewrite to "What AttackMap does" or "What I built"

- [ ] **Step 3: Voice-edit `analyzers.md`** (28 occurrences — the heaviest)

Read the file. The high count likely means a lot of "we detect X by Y" instructional prose. Each gets the same treatment:
- "we detect" → "AttackMap detects" (preferred — product-subject is clearer for analyzer docs)
- "our X" → "AttackMap's X" or "the X" depending on context

For instructional voice ("we recommend you do X"), use either "I recommend" (personal) or "The recommended approach is X" (objective). Match the surrounding tone — if the rest of the document is product-objective, lean objective.

- [ ] **Step 4: Voice-edit remaining files**

- `architecture.md` — search and edit any first-person
- `faq.md` (4 occurrences) — first-person
- `getting-started.md` (3 occurrences) — first-person
- `sdk.md` (2 occurrences) — first-person
- `use-cases.md` — search and edit any

- [ ] **Step 5: Verify clean prose**

```bash
grep -nE "\\b[Ww]e\\b|\\b[Oo]ur\\b" content/attackmap/*.md
grep -n "\\bmlaify\\b" content/attackmap/*.md | grep -v "github.com/mlaify"
```

Expected: empty (or only technical/spec usages that the implementer judged should stay).

- [ ] **Step 6: Build**

```bash
npm run build 2>&1 | tail -5
ls public/attackmap/index.html public/attackmap/analyzers/index.html
```

- [ ] **Step 7: Commit**

```bash
git add content/attackmap/
git commit -m "Import AttackMap content with first-person voice edits

7 files (index + 6 sub-pages) imported. Analyzers doc had the most
\"we\" usage; rewrote most as product-subject (\"AttackMap detects\")
since that reads more naturally for analyzer documentation than
first-person. Other files lighter touch.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 7: Import smaller projects with voice edits

**Files:**
- Create: `content/omekarapper/_index.md`, `content/opensift/_index.md`, `content/opencontractrx/_index.md`

Three short files. Each becomes a section landing at top-level.

- [ ] **Step 1: Copy and rename**

```bash
mkdir -p content/omekarapper content/opensift content/opencontractrx
cp /Volumes/Dev/repos/GitHub/mlaify/mlaify.github.io/content/docs/project-omekarapper.md content/omekarapper/_index.md
cp /Volumes/Dev/repos/GitHub/mlaify/mlaify.github.io/content/docs/project-opensift.md content/opensift/_index.md
cp /Volumes/Dev/repos/GitHub/mlaify/mlaify.github.io/content/docs/project-opencontractrx.md content/opencontractrx/_index.md
```

- [ ] **Step 2: Voice-edit `content/omekarapper/_index.md`**

5 we/mlaify occurrences. Read and apply substitutions. Front-matter: ensure `title:` is set (probably "OmekaRapper"), and consider whether the file should reference `accent: "brand"` (uses copper) or just inherit. Keep front-matter minimal.

- [ ] **Step 3: Voice-edit `content/opensift/_index.md`**

3 occurrences. Same approach.

- [ ] **Step 4: Voice-edit `content/opencontractrx/_index.md`**

3 occurrences. Same approach.

- [ ] **Step 5: Verify**

```bash
grep -nE "\\b[Ww]e\\b|\\b[Oo]ur\\b|\\bmlaify\\b" content/omekarapper/_index.md content/opensift/_index.md content/opencontractrx/_index.md | grep -v "github.com/mlaify"
```

Expected: empty or only intentional retained usages.

- [ ] **Step 6: Build and verify routes**

```bash
npm run build 2>&1 | tail -5
ls public/omekarapper/index.html public/opensift/index.html public/opencontractrx/index.html
```

- [ ] **Step 7: Commit**

```bash
git add content/omekarapper/ content/opensift/ content/opencontractrx/
git commit -m "Import smaller projects (OmekaRapper, OpenSift, OpenContractRx)

Each becomes a top-level section at /omekarapper/, /opensift/,
/opencontractrx/. Front-matter normalized, voice edited from we/our to
first-person singular where applicable.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 8: Port `build-principles` → `/principles` with substantial voice rewrite

**Files:**
- Create: `content/principles/_index.md`

This is the biggest voice rewrite in Phase 2. The mlaify version frames principles as "what every mlaify project follows." Recasting as "what every project of mine follows" requires more than substitution — sentences need rebalancing.

- [ ] **Step 1: Copy the file**

```bash
mkdir -p content/principles
cp /Volumes/Dev/repos/GitHub/mlaify/mlaify.github.io/content/build-principles/_index.md content/principles/_index.md
```

- [ ] **Step 2: Rewrite the opening section**

Open `content/principles/_index.md`. Replace the opening paragraph (the part before the first `## ` heading) with:

```markdown
These are not aspirational. They are the conventions every project of mine actually follows. If a project on this site appears to violate one of them, it's a bug — file an issue.
```

Drop the "lead:" front-matter value if present, or rewrite to: `lead: "The conventions every project of mine follows."`

- [ ] **Step 3: Rewrite each numbered principle's opening line**

Read each `## 1.`, `## 2.`, `## 3.`, `## 4.`, `## 5.` section. For each, the heading itself can largely stay. The first paragraph of the body that says "We build…" or "mlaify projects…" needs rewriting. Approach:

- "We build for the people…" → "I build for the people…"
- "Every mlaify project follows…" → "Every project I build follows…" or "Every project here follows…"
- "Our products" → "These projects"

Read each section's body in full. Rewrite any sentence that grates when read in first-person singular. Keep examples specific to the projects (Aegis, AttackMap, OmekaRapper).

- [ ] **Step 4: Front-matter**

Ensure:

```yaml
---
title: "Build principles"
description: "The five conventions every project of mine follows: real workflows, explicit status, security as design, composable boundaries, and documentation close to implementation."
date: 2026-05-08T00:00:00-05:00
lastmod: 2026-06-03T00:00:00-05:00
draft: false
weight: 20
toc: true
sidebar: false
lead: "The five conventions every project of mine follows."
---
```

- [ ] **Step 5: Final read-through**

Read the entire file as if you've never seen it. It should feel like one person wrote it, not "an org" wrote it. Watch for:
- Plural-sounding constructions ("our way of doing things")
- "We" pronouns lingering
- "mlaify" anywhere outside URLs

- [ ] **Step 6: Verify no stragglers**

```bash
grep -nE "\\b[Ww]e\\b|\\b[Oo]ur\\b|\\bmlaify\\b" content/principles/_index.md | grep -v "github.com/mlaify"
```

Expected: empty.

- [ ] **Step 7: Build and visually verify**

```bash
npm run build 2>&1 | tail -5
ls public/principles/index.html
```

- [ ] **Step 8: Commit**

```bash
git add content/principles/
git commit -m "Port build-principles → /principles with first-person rewrite

Substantial rewrite — not just substitution. The mlaify framing
(\"every mlaify project follows\") doesn't translate to a personal site
by find/replace. Each numbered principle's opening paragraph reworked
so the page reads as one person writing about their own projects.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 9: Rewrite `/about/` page (merge mlaify umbrella + new bio, no photography)

**Files:**
- Modify: `content/about/_index.md`

The Phase 1 hot-fix already moved `content/me.md` → `content/about/_index.md` (Task 3 above). The body is still the old `me.md` text starting with "Matt is a professional photographer…" — which is wrong (Matt has retired from photography for medical reasons; see the user memory file). This task replaces the body entirely.

- [ ] **Step 1: Replace `content/about/_index.md` body**

Read the current file. The front-matter (from Task 3) should be:

```yaml
---
title: "About"
description: "About Matthew A. Davis."
aliases:
  - /me/
cascade:
  comments: false
---
```

Replace everything after the closing `---` with this body. Aim for honest, calm tone — no marketing voice.

```markdown
![Matthew A. Davis](/images/pic_mdavis.png)

I'm Matt. I write at [Fedora Magazine](https://fedoramagazine.org/) as a contributor and editor for the [Fedora Project](https://fedoraproject.org/), and I build small, security-first open source tools — primarily [Aegis](/aegis/) (post-quantum encrypted messaging) and [AttackMap](/attackmap/) (defensive security analysis). A few smaller projects sit alongside those: [OmekaRapper](/omekarapper/), [OpenSift](/opensift/), and [OpenContractRx](/opencontractrx/).

I've been working in open source since 2003 — long enough to remember Fedora Core 1. My background is psychology and interdisciplinary studies: BA from the [University of Texas at Arlington](https://www.uta.edu/) in 2010, MS from the [University of North Texas](https://www.unt.edu/) in 2019. While at UTA I did neuroscience research in electrophysiology and pain and contributed to several publications. Day job has been in IT and cybersecurity.

I used to do professional photography. I no longer do — that part of my life ended for medical reasons. Some older photo essays are still on the blog and they'll stay there; I just don't make new ones.

## What I'm working on

The way the projects on this site look — small, protocol-first, status-honest, documentation-close-to-code — isn't an accident. It comes from a few convictions:

**Security architecture is design work, not paperwork.** Threat models and protocol RFCs are first-class artifacts in my repos, written before the code stabilizes, not retroactively to satisfy a checklist. When a project lacks a threat model, I say so on its page.

**Status you can trust beats marketing.** Every project here carries an explicit status. "Alpha" means alpha. "Hardening pending" means I haven't yet hardened it. I'd rather lose a lead than oversell a `v0.1`.

**Composability over monoliths.** I favor module boundaries that let other people swap pieces — analyzers in AttackMap, identity providers and relays in Aegis, AI providers in OmekaRapper and OpenSift. Costs more upfront and pays off everywhere downstream.

See [build principles](/principles/) for the longer version.

## Elsewhere

- [GitHub](https://github.com/mdavistffhrtporg) — code
- [Bluesky](https://bsky.app/profile/matthewd.xyz) — short-form
- [GitLab](https://gitlab.com/matthewdxyz) — mirrors
- [Fedora Magazine](https://fedoramagazine.org/) — writing

![](/images/mdavis_sig.png)
```

This pulls in:
- The "What I'm working on" paragraphs from mlaify's `/about/` (rewritten as first-person)
- The educational/IT/cybersec background from the original `me.md`
- Explicit, honest one-paragraph note about photography retirement
- The signature image from `me.md` at the end

**The first-line tagline** ("OSS contributor · Writer at Fedora Magazine · Builds security-first tooling" from `params.toml`) stays untouched — that lives in the homepage hero, not here. The About page is the long form.

- [ ] **Step 2: Verify the signature image path**

The original `me.md` references `/images/mdavis_sig.png`. Verify this file exists:

```bash
ls static/images/mdavis_sig.png
```

If not present, remove the trailing `![](/images/mdavis_sig.png)` line from the body.

- [ ] **Step 3: Build and read**

```bash
npm run build 2>&1 | tail -5
```

Then read `public/about/index.html` (or open in browser via `npx serve public -l 4173`). The page should:
- Render the avatar
- Read naturally in first-person
- Link to each of the 5 projects
- Have no "mlaify" or "we" prose
- Have the signature at the end (if the image file exists)

- [ ] **Step 4: Commit**

```bash
git add content/about/_index.md
git commit -m "Rewrite /about/ page: merge umbrella copy + remove photography lead

Replaces the old me.md body (\"Matt is a professional photographer…\")
with a first-person about page that honors:

- The medical retirement from professional photography (no longer a
  current identity; older posts stay as posts).
- Substantive bio: Fedora Magazine writer/editor, OSS since 2003,
  psychology/interdisciplinary education, IT/cybersec background.
- Umbrella framing for the 5 projects, recast from mlaify's plural
  voice to first-person.
- Links to /principles/ for the convictions in full.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 10: Port `/contribute/` with voice rewrite

**Files:**
- Create: `content/contribute/_index.md`

Mlaify's contribute page is 53 lines. Mostly addressed to potential contributors. Full first-person rewrite.

- [ ] **Step 1: Copy then rewrite**

```bash
mkdir -p content/contribute
cp /Volumes/Dev/repos/GitHub/mlaify/mlaify.github.io/content/contribute/_index.md content/contribute/_index.md
```

Read the file. Rewrite as a first-person invitation. The structure (sections like "What I'm looking for", "How to file a useful issue", "PR conventions") can stay; the voice changes from "We accept" / "we ask" to "I welcome" / "I ask".

Specific edits to expect:
- "We welcome contributions" → "I welcome contributions"
- "mlaify projects accept" → "These projects accept" or "I accept"
- "Our PR conventions" → "PR conventions" (drop possessive) or "My PR conventions"
- "Reach out to us at" → "Reach out to me at" — with the contact email being matthewd@matthewd.xyz per the user's existing setup

Update any contact email/method to matthewd's. The mlaify version may have a generic org email; replace.

- [ ] **Step 2: Verify no stragglers**

```bash
grep -nE "\\b[Ww]e\\b|\\b[Oo]ur\\b|\\bmlaify\\b" content/contribute/_index.md | grep -v "github.com/mlaify"
```

Expected: empty.

- [ ] **Step 3: Build and read**

```bash
npm run build 2>&1 | tail -5
```

- [ ] **Step 4: Commit**

```bash
git add content/contribute/
git commit -m "Port /contribute/ with first-person voice rewrite

Recasts the contribution guide from mlaify's plural voice (\"We
welcome\", \"our PR conventions\") to first-person. Contact email
updated to matthewd@matthewd.xyz.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 11: Create `/projects/` hub page

**Files:**
- Create: `content/projects/_index.md`, `layouts/projects/list.html`

The hub iterates `site.Params.products` (populated in Task 4) and renders each as a status-badged card.

- [ ] **Step 1: Create `content/projects/_index.md`**

```yaml
---
title: "Projects"
description: "Open source software I build — security tooling, AI-assisted workflow tools, and a couple of smaller experiments."
date: 2026-06-03T00:00:00-05:00
draft: false
comments: false
---

Two flagship projects (Aegis and AttackMap) sit alongside a few smaller ones. Each carries an honest status — alpha means alpha. See [build principles](/principles/) for the longer story on how I work.
```

- [ ] **Step 2: Create `layouts/projects/list.html`**

```html
{{ define "main" }}
<div class="container-page py-12 lg:py-16">
  <header class="mb-12 max-w-2xl">
    <h1 class="text-balance text-4xl font-semibold tracking-tight text-ink-900 sm:text-5xl dark:text-white">
      {{ .Title }}
    </h1>
    {{- with .Description }}
      <p class="mt-4 text-lg text-ink-600 dark:text-ink-300">{{ . }}</p>
    {{- end }}
    {{- with .Content }}
      <div class="prose prose-stone dark:prose-invert mt-6 max-w-none prose-a:text-copper-700 dark:prose-a:text-copper-400">
        {{ . }}
      </div>
    {{- end }}
  </header>

  {{- $products := site.Params.products }}
  <ul class="grid gap-6 sm:grid-cols-2">
    {{- range $k, $p := $products }}
      {{- $sorted := dict "key" $k "p" $p "weight" $p.weight }}
    {{- end }}
    {{- $entries := slice }}
    {{- range $k, $p := $products }}
      {{- $entries = $entries | append (dict "key" $k "name" $p.name "tagline" $p.tagline "status" $p.status "accent" $p.accent "url" $p.url "weight" $p.weight) }}
    {{- end }}
    {{- range sort $entries "weight" "asc" }}
      <li>
        <a href="{{ .url }}" data-accent="{{ .accent }}" class="group card card-hover flex h-full flex-col">
          <div class="flex items-start justify-between gap-4">
            <h2 class="text-2xl font-semibold tracking-tight text-ink-900 group-hover:accent-text dark:text-white">
              {{ .name }}
            </h2>
            <span class="status-badge status-{{ .status }}">{{ .status }}</span>
          </div>
          <p class="mt-3 text-ink-600 dark:text-ink-300">{{ .tagline }}</p>
        </a>
      </li>
    {{- end }}
  </ul>
</div>
{{ end }}
```

The `card`, `card-hover`, `status-badge`, `status-alpha`, `accent-text`, `container-page` classes are all defined in `assets/css/app.css` (ported from mlaify in Phase 1). The `data-accent="aegis"` etc. attribute scopes the CSS custom-property accent system so the Aegis card uses cyan accents on hover, AttackMap uses amber, and the rest fall back to copper.

- [ ] **Step 3: Build and check**

```bash
npm run build 2>&1 | tail -5
ls public/projects/index.html
grep -c "status-alpha" public/projects/index.html
```

Expected: file exists; "status-alpha" count is 5 (one per project).

- [ ] **Step 4: Visual smoke**

```bash
npx serve public -l 4173 &
SERVE_PID=$!
sleep 2
curl -s http://localhost:4173/projects/ | grep -oE "Aegis|AttackMap|OmekaRapper|OpenSift|OpenContractRx" | sort -u
kill $SERVE_PID
```

Expected: all 5 names present.

- [ ] **Step 5: Commit**

```bash
git add content/projects/ layouts/projects/list.html
git commit -m "Add /projects/ hub page listing all 5 projects

Iterates site.Params.products (sorted by weight) and renders each as
a status-badged card. data-accent attribute scopes per-product accent
colors (cyan for Aegis, amber for AttackMap, copper for the rest).

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 12: Add projects section to the homepage

**Files:**
- Modify: `layouts/index.html`

Phase 1 deferred the projects section because there was no content. Phase 2 adds it between the hero and the latest-writing section.

- [ ] **Step 1: Open `layouts/index.html` and add a new `<!-- Projects -->` section between the hero `</section>` and the `<!-- Latest writing -->` section**

The current file structure is:

```html
{{ define "main" }}
<!-- Hero -->
<section class="...">...</section>

<!-- Latest writing -->
<section class="...">...</section>
{{ end }}
```

After the hero `</section>` and before the `<!-- Latest writing -->` comment, insert:

```html
<!-- Projects -->
<section class="border-b border-ink-200/60 dark:border-ink-800/60">
  <div class="container-page py-16">
    <div class="mb-10 flex items-end justify-between">
      <h2 class="text-3xl font-semibold tracking-tight text-ink-900 dark:text-white">Projects</h2>
      <a href="/projects/" class="text-sm font-medium text-copper-700 hover:text-copper-800 dark:text-copper-400 dark:hover:text-copper-300">
        All projects →
      </a>
    </div>

    {{- $products := site.Params.products }}
    {{- $entries := slice }}
    {{- range $k, $p := $products }}
      {{- $entries = $entries | append (dict "key" $k "name" $p.name "tagline" $p.tagline "status" $p.status "accent" $p.accent "url" $p.url "weight" $p.weight) }}
    {{- end }}
    {{- $featured := first 2 (sort $entries "weight" "asc") }}
    <div class="grid gap-6 lg:grid-cols-2">
      {{- range $featured }}
        <a href="{{ .url }}" data-accent="{{ .accent }}" class="group card card-hover flex flex-col">
          <div class="flex items-start justify-between gap-4">
            <h3 class="text-2xl font-semibold tracking-tight text-ink-900 group-hover:accent-text dark:text-white">
              {{ .name }}
            </h3>
            <span class="status-badge status-{{ .status }}">{{ .status }}</span>
          </div>
          <p class="mt-3 text-ink-600 dark:text-ink-300">{{ .tagline }}</p>
        </a>
      {{- end }}
    </div>

    {{- $small := after 2 (sort $entries "weight" "asc") }}
    {{- with $small }}
      <ul class="mt-6 flex flex-wrap gap-3 text-sm">
        {{- range . }}
          <li>
            <a href="{{ .url }}" class="inline-flex items-center gap-2 rounded-full border border-ink-200 px-4 py-2 hover:border-copper-300 hover:text-copper-700 dark:border-ink-800 dark:hover:border-copper-600 dark:hover:text-copper-400">
              {{ .name }}
              <span class="text-xs text-ink-500 dark:text-ink-400">{{ .status }}</span>
            </a>
          </li>
        {{- end }}
      </ul>
    {{- end }}
  </div>
</section>
```

This renders the top 2 (Aegis, AttackMap) as full cards and the remaining 3 (OmekaRapper, OpenSift, OpenContractRx) as compact pill links beneath, with an "All projects →" link.

- [ ] **Step 2: Build and verify the homepage has the new section**

```bash
npm run build 2>&1 | tail -5
grep -oE "Aegis|AttackMap|OmekaRapper|OpenSift|OpenContractRx" public/index.html | sort -u
```

Expected: all 5 names appear in the homepage HTML.

- [ ] **Step 3: Commit**

```bash
git add layouts/index.html
git commit -m "Add Projects section to homepage (B layout, Phase 2 completion)

Editorial hero stack now matches the design from brainstorming: hero,
then projects (top 2 as cards + the smaller 3 as pills), then latest
writing. Pulls all data from site.Params.products.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 13: Update top nav to Writing · Projects · About

**Files:**
- Modify: `config/_default/menus/menus.en.toml`

Phase 1 nav was `Posts · About`. With Phase 2 content in place, the canonical nav becomes the planned three items.

- [ ] **Step 1: Replace the `[[main]]` entries**

Open `config/_default/menus/menus.en.toml`. The current `[[main]]` section is:

```toml
[[main]]
  name = "Posts"
  url = "/posts/"
  weight = 10

[[main]]
  name = "About"
  url = "/me/"
  weight = 20
```

Replace with:

```toml
[[main]]
  name = "Writing"
  url = "/writing/"
  weight = 10

[[main]]
  name = "Projects"
  url = "/projects/"
  weight = 20

[[main]]
  name = "About"
  url = "/about/"
  weight = 30
```

Also update the `[[footer]]` entries:
- "About" URL from `/me/` to `/about/`
- "Archives" stays pointing at `/writing/` (was `/posts/` after Phase 1 hot-fix; now needs to be `/writing/` since posts moved)
- Add a `Projects` link to the footer:

```toml
[[footer]]
  name = "About"
  url = "/about/"
  weight = 10

[[footer]]
  name = "Projects"
  url = "/projects/"
  weight = 15

[[footer]]
  name = "Archives"
  url = "/writing/"
  weight = 20

[[footer]]
  name = "Principles"
  url = "/principles/"
  weight = 25

[[footer]]
  name = "Contribute"
  url = "/contribute/"
  weight = 28

[[footer]]
  name = "Privacy"
  url = "/privacy-policy/"
  weight = 30

[[footer]]
  name = "Security"
  url = "/security/"
  weight = 40

[[footer]]
  name = "GitHub"
  url = "https://github.com/mdavistffhrtporg"
  weight = 50
```

`[[social]]` entries stay unchanged from Phase 1.

- [ ] **Step 2: Build and verify the nav renders**

```bash
npm run build 2>&1 | tail -5
grep -oE ">Writing<|>Projects<|>About<|>Principles<|>Contribute<" public/index.html | sort | uniq -c
```

Expected: counts > 0 for each (Writing/Projects/About appear in both header and possibly footer; Principles/Contribute appear in footer).

- [ ] **Step 3: Commit**

```bash
git add config/_default/menus/menus.en.toml
git commit -m "Top nav: Writing · Projects · About; footer expanded

Top nav goes from the Phase 1 \"Posts · About\" to the canonical three
items. Footer also gets Projects, Principles, and Contribute links.
Archives now points at /writing/ (the new location of the blog).

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 14: CHANGELOG entry

**Files:**
- Modify: `CHANGELOG.md`

- [ ] **Step 1: Read current state and add Phase 2 entry**

Read `CHANGELOG.md`. The `## [Unreleased]` section currently contains Phase 1's changes. Either:

- If Phase 1 has been tagged/released, add a new `## [Unreleased]` for Phase 2 above it.
- If Phase 1 is still under `[Unreleased]`, extend that section with Phase 2's bullets.

Likely the latter — Phase 1 just merged to main and hasn't been tagged with a version. So extend `[Unreleased]`. Append after Phase 1's "### Removed" subsection:

```markdown
### Added
- All mlaify content imported with first-person voice edits: Aegis (8 pages), AttackMap (7 pages), OmekaRapper, OpenSift, OpenContractRx.
- `/principles/` page (formerly mlaify's `/build-principles/`), recast in first-person.
- `/contribute/` page recast in first-person with matthewd@matthewd.xyz as the contact.
- `/projects/` hub page listing all 5 projects with status badges, sorted by weight.
- Projects section on the homepage (top 2 as cards, smaller 3 as pill links).
- `[products]` table in `params.toml` configuring all 5 projects (name, tagline, status, accent, URL, repo URL).

### Changed
- `/posts/` renamed to `/writing/`. Old URLs (`/posts/<slug>/`) 301-redirect via Hugo aliases. `/archives/` → `/writing/`.
- `/me/` renamed to `/about/`. Old URL redirects via alias. Body rewritten to remove the "professional photographer" lead (medical retirement), merge mlaify's umbrella copy in first-person, and link to `/principles/` for the longer story.
- Top nav: Writing · Projects · About (was Posts · About). Footer gains Projects, Principles, and Contribute entries.

### Removed
- `content/posts/` directory (moved to `content/writing/`).
- `content/me.md` (folded into `content/about/_index.md`).
```

- [ ] **Step 2: Commit**

```bash
git add CHANGELOG.md
git commit -m "Add CHANGELOG entry for Phase 2 content migration

Per the global rule: user-facing changes get a Keep a Changelog
entry. Phase 2 brings all mlaify content onto matthewd.xyz with
first-person voice, renames /posts/ to /writing/ and /me/ to
/about/ (with aliases), and adds the Projects nav and hub.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 15: Full local verification (ship gate)

**Files:** none

- [ ] **Step 1: Clean build**

```bash
rm -rf public/ resources/ .hugo_build.lock
npm ci
npm run build
```

Expected: success.

- [ ] **Step 2: New URLs exist**

```bash
for path in \
  aegis aegis/architecture aegis/protocol aegis/faq \
  attackmap attackmap/analyzers attackmap/architecture attackmap/faq \
  omekarapper opensift opencontractrx \
  principles contribute about projects writing 404.html; do
  if [ -f "public/$path/index.html" ] || [ -f "public/$path" ]; then
    echo "OK: /$path/"
  else
    echo "MISSING: /$path/"
  fi
done
```

Expected: 16 `OK:` lines.

- [ ] **Step 3: Old URLs redirect**

```bash
for path in posts/post-1 posts/post-6 me archives; do
  if grep -q "http-equiv=refresh" "public/$path/index.html" 2>/dev/null; then
    echo "OK: /$path/ redirects"
    grep -oE "url=[^\"]*" "public/$path/index.html" | head -1
  else
    echo "MISSING: /$path/ redirect"
  fi
done
```

Expected: 4 `OK:` + corresponding `url=...` lines pointing to the new URLs (`/writing/post-1/`, `/writing/post-6/`, `/about/`, `/writing/`).

- [ ] **Step 4: Voice-edit verification**

```bash
echo "=== Stray 'we'/'mlaify' in imported content ==="
grep -rnE "\\b[Ww]e\\b|\\b[Oo]ur\\b|\\bmlaify\\b" \
  content/aegis content/attackmap content/omekarapper content/opensift content/opencontractrx \
  content/principles content/contribute content/about \
  2>/dev/null \
  | grep -v "github.com/mlaify\\|gitlab.com/mlaify\\|mlaify/aegis\\|mlaify/attackmap\\|mlaify/omekarapper\\|mlaify/opensift\\|mlaify/opencontractrx" \
  | head -10
```

Expected: empty, or only intentional retentions (technical spec usages, NOT author voice).

- [ ] **Step 5: Projects hub renders all 5**

```bash
grep -oE "Aegis|AttackMap|OmekaRapper|OpenSift|OpenContractRx" public/projects/index.html | sort -u
```

Expected: all 5.

- [ ] **Step 6: Homepage renders the new Projects section**

```bash
grep -c "Projects" public/index.html
grep -oE "Aegis|AttackMap|OmekaRapper|OpenSift|OpenContractRx" public/index.html | sort -u
```

Expected: "Projects" count ≥ 2 (header + section heading); all 5 names present.

- [ ] **Step 7: Status badges**

```bash
grep -c "status-alpha" public/projects/index.html public/index.html
```

Expected: ≥ 5 across the two files (5 in projects hub + 2 on homepage).

- [ ] **Step 8: Pagefind index covers the new content**

```bash
npm run build 2>&1 | grep "Indexed"
```

Expected: Pagefind indexes substantially more pages than Phase 1 (was 29; should jump to 50+).

- [ ] **Step 9: Serve and click smoke**

```bash
npx serve public -l 4173 &
SERVE_PID=$!
sleep 2
for url in / /writing/ /aegis/ /attackmap/ /projects/ /about/ /principles/ /contribute/ /omekarapper/ /opensift/ /opencontractrx/ /404.html; do
  if curl -sf "http://localhost:4173$url" -o /dev/null; then
    echo "OK: 200 $url"
  else
    echo "FAIL: $url"
  fi
done
kill $SERVE_PID
```

Expected: 12 `OK:` lines.

- [ ] **Step 10: Tag the milestone (local, don't push)**

```bash
git tag -a phase-2-content-migration -m "Phase 2 of the matthewd.xyz × mlaify merge: content migration complete"
```

---

## Task 16: Push and open PR

- [ ] **Step 1: Push branch and tag**

```bash
git push -u origin phase-2-content-migration
git push origin phase-2-content-migration
```

- [ ] **Step 2: Open the PR**

```bash
gh pr create --title "Phase 2: Content migration (mlaify content → matthewd.xyz)" --body "$(cat <<'EOF'
## Summary

Imports all mlaify content under matthewd.xyz with first-person voice edits, renames `/posts/` → `/writing/` and `/me/` → `/about/` (both with backward-compat redirects), and adds the `Writing · Projects · About` nav, the `/projects/` hub, and a Projects section on the homepage.

### What lands

- Aegis (8 pages) at `/aegis/*` — light voice edits, mostly product-objective copy preserved.
- AttackMap (7 pages) at `/attackmap/*` — analyzers doc had the heaviest voice work.
- OmekaRapper, OpenSift, OpenContractRx at top-level `/<name>/`.
- `/principles/` (formerly `/build-principles/` on mlaify) — substantial first-person rewrite.
- `/contribute/` — full voice rewrite, matthewd@matthewd.xyz as contact.
- `/about/` — new body merging the educational/Fedora bio with mlaify umbrella copy, photography retirement noted honestly.
- `/projects/` hub iterating `site.Params.products`.
- Homepage projects section: Aegis + AttackMap as cards, smaller 3 as pills.

### Redirects

| Old | New |
|-----|-----|
| `/posts/<slug>/` | `/writing/<slug>/` |
| `/me/` | `/about/` |
| `/archives/` | `/writing/` |

This is Phase 2 of the merge described in [docs/superpowers/specs/2026-06-03-matthewd-mlaify-merge-design.md](docs/superpowers/specs/2026-06-03-matthewd-mlaify-merge-design.md). Phase 3 (mlaify.io 301 redirects + sunset) follows as a separate PR.

## Test plan

- [ ] CI build passes.
- [ ] All 5 project landing pages render with status badges.
- [ ] All Aegis & AttackMap sub-pages render.
- [ ] Old URLs (`/posts/post-N/`, `/me/`, `/archives/`) redirect to new ones.
- [ ] Top nav shows Writing · Projects · About.
- [ ] Homepage shows hero, projects section, latest writing.
- [ ] `/about/` reads naturally — no "we"/"mlaify" prose, photography retirement framed honestly.
- [ ] Pagefind search returns results for "aegis", "attackmap", "fedora", "luks".

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

- [ ] **Step 3: Wait for review** — leave the PR open. Do not merge without the user's go.

---

## Open questions for the user

These should be addressed during execution or at PR-review time:

1. **Tagline** — Phase 1 shipped with "OSS contributor · Writer at Fedora Magazine · Builds security-first tooling." Still good, or wordsmith? Lives in `config/_default/params.toml` under `tagline`.
2. **OG image** — `params.toml` references `images/og-default.png` which doesn't exist. Create one or change the path? Not blocking but worth fixing in this phase since social shares will start mattering as content grows.
3. **Smaller project repo URLs** — Verified during Task 4 that `mlaify/omekarapper`, `mlaify/opensift`, `mlaify/opencontractrx` exist? If they're under a different org or named differently, update `params.toml`.
4. **GitHub Discussions for Giscus comments** — still need to be confirmed enabled with the `Announcements` category. Test on any blog post after the deploy.
5. **`/about/` body draft** — Task 9 includes a full draft. PR review is your chance to wordsmith.

## Risks

| # | Risk | Mitigation |
|---|------|------------|
| 1 | Voice edits leave residual "we" usages that read awkwardly | Task-level verification greps; PR review |
| 2 | Hugo `aliases` redirects break for some edge case (e.g., trailing slash variant) | Smoke-checked in Task 2/3 and Task 15 |
| 3 | Smaller project repo URLs wrong → 404 from `/projects/` cards | Task 4 open question; verified before commit |
| 4 | `/about/` body wordsmithing differs from user's preference | Draft is shippable; PR review polishes |
| 5 | Pagefind search misses some new pages due to indexing oddities | Task 15 step 8 checks index count |
| 6 | Section rename causes accidental dead links inside post bodies (e.g., "see /posts/post-3 for X") | Grep step in verification; manual fix if found |

## Out of scope (Phase 3 or later)

- mlaify.io DNS / redirect setup.
- Archiving the mlaify.github.io repo.
- Per-product subdomains (aegis.matthewd.xyz, etc.).
- Visual polish on shortcode rendering (e.g., the `callout` shortcode styling).
- Comments enabled on project pages.

## Self-review notes (run after writing this plan)

1. **Spec coverage:**
   - All mlaify content imported ✓ (Tasks 5–10)
   - Voice-edit pass ✓ (each import task includes voice edits)
   - Section renames ✓ (Tasks 2–3)
   - Nav updated ✓ (Task 13)
   - /projects/ hub ✓ (Task 11)
   - Homepage updated ✓ (Task 12)
   - CHANGELOG ✓ (Task 14)
   - Ship gate ✓ (Task 15)
   - Phase 3 explicitly out of scope ✓
2. **Placeholder scan:** Voice edits are described as "apply substitutions, judgment per file" rather than line-by-line instructions for every file — this is intentional because the file count is high. The implementer is given the substitution table at the top, sample edits in each task, and a verification grep to catch misses. Not a placeholder; a deliberate delegation.
3. **Type/name consistency:** `/writing/`, `/about/`, `/projects/`, `/principles/`, `/contribute/` used consistently throughout. Section names match directory names. The `site.Params.products` keys (aegis, attackmap, omekarapper, opensift, opencontractrx) match the URL paths.
4. **Open items called out** in their own section.

Plan complete.
