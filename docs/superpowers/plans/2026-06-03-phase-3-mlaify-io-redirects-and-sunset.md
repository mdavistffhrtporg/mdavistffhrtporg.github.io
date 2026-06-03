# Phase 3 — mlaify.io Redirects & Repo Sunset: Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make matthewd.xyz the canonical destination for all mlaify.io URLs (and sunset the side domains `opensift.org` and `siftbook.org`), then archive the `mlaify/mlaify.github.io` repo.

**Architecture:** Cloudflare Bulk Redirects handle path-preserving 301s at the edge — no origin hop, no Worker code. The mlaify.github.io repo content is replaced with a minimal archived-notice index page (defensive fallback if a Cloudflare rule ever fails to match), then the repo is archived via the GitHub API. Side domains lose their GH Pages claim (removed from CNAME) and their DNS records (removed via Cloudflare dashboard).

**Tech Stack:** Cloudflare dashboard (Bulk Redirects + DNS), `gh` CLI (repo archive), curl (smoke tests). No code changes to matthewd.xyz repo except a small CHANGELOG entry.

**Scope boundary:** Phase 3 ships redirects + archive. It does NOT touch external references in other repos (mlaify/aegis-spec, mlaify/attackmap, package metadata, etc.) — those would be a separate small follow-up. The catchall Cloudflare redirect handles unknown external links gracefully in the meantime.

**Side actor caveat:** Several tasks require the user to act in the Cloudflare dashboard. They are clearly marked **USER ACTION** and the plan provides exact values to paste.

---

## File map

### Created

```
docs/superpowers/ops/cloudflare-redirects-mlaify-io.csv     # Bulk Redirects upload file
docs/superpowers/ops/phase-3-runbook.md                     # Step-by-step manual procedure for the Cloudflare changes
```

### Modified (in matthewd.xyz repo)

```
CHANGELOG.md                                                 # Phase 3 entry under [Unreleased]
```

### Modified (in mlaify/mlaify.github.io repo, separately)

```
content/_index.md                                            # Replaced with a "moved" notice
static/CNAME                                                 # Reduced to just mlaify.io (opensift.org & siftbook.org removed)
```

### External actions (not code, but tracked)

```
Cloudflare dashboard: upload Bulk Redirects list for mlaify.io
Cloudflare dashboard: delete DNS records for opensift.org and siftbook.org
GitHub: archive mlaify/mlaify.github.io repo via `gh repo archive`
```

---

## Task 1: Generate the Cloudflare Bulk Redirects CSV

**Files:**
- Create: `docs/superpowers/ops/cloudflare-redirects-mlaify-io.csv`

Cloudflare's Bulk Redirects feature accepts a CSV upload where each row is one source URL → target URL mapping with toggles for preserve-suffix and preserve-query-string.

- [ ] **Step 1: Create the ops directory**

```bash
mkdir -p docs/superpowers/ops
```

- [ ] **Step 2: Write `docs/superpowers/ops/cloudflare-redirects-mlaify-io.csv`**

```csv
source_url,target_url,status,preserve_query_string,subpath_matching,preserve_path_suffix,include_subdomains
mlaify.io/aegis/,https://matthewd.xyz/aegis/,301,true,true,true,false
mlaify.io/attackmap/,https://matthewd.xyz/attackmap/,301,true,true,true,false
mlaify.io/docs/project-omekarapper/,https://matthewd.xyz/omekarapper/,301,true,true,true,false
mlaify.io/docs/project-opensift/,https://matthewd.xyz/opensift/,301,true,true,true,false
mlaify.io/docs/project-opencontractrx/,https://matthewd.xyz/opencontractrx/,301,true,true,true,false
mlaify.io/docs/projects/,https://matthewd.xyz/projects/,301,true,false,false,false
mlaify.io/docs/,https://matthewd.xyz/projects/,301,true,false,false,false
mlaify.io/build-principles/,https://matthewd.xyz/principles/,301,true,true,true,false
mlaify.io/contribute/,https://matthewd.xyz/contribute/,301,true,true,true,false
mlaify.io/about/,https://matthewd.xyz/about/,301,true,true,true,false
mlaify.io/privacy/,https://matthewd.xyz/privacy-policy/,301,true,false,false,false
mlaify.io/security/,https://matthewd.xyz/security/,301,true,true,true,false
mlaify.io/.well-known/security.txt,https://matthewd.xyz/.well-known/security.txt,301,false,false,false,false
mlaify.io/humans.txt,https://matthewd.xyz/humans.txt,301,false,false,false,false
mlaify.io/privacy.json,https://matthewd.xyz/privacy.json,301,false,false,false,false
mlaify.io/,https://matthewd.xyz/projects/,301,true,true,true,false
```

Notes on each toggle:
- `subpath_matching=true` + `preserve_path_suffix=true`: matches the source path *and* anything below it; appends the suffix to the target. So `mlaify.io/aegis/architecture` → `matthewd.xyz/aegis/architecture`.
- `subpath_matching=false`: exact-match only.
- Single-file rules (security.txt, humans.txt, privacy.json) use exact-match (no subpath, no suffix preservation).
- `mlaify.io/` last (it's the catchall — Cloudflare evaluates list entries top-down, first match wins).
- `include_subdomains=false`: rules only fire for the apex `mlaify.io`, not e.g. `www.mlaify.io` (which doesn't exist for this domain).

**Important:** Cloudflare's CSV importer may want column headers in a different order or different boolean representations (`Yes`/`No` instead of `true`/`false`). The format above matches the documented spec for "Bulk Redirect Lists CSV upload" as of late 2025. If the upload fails, the runbook in Task 2 has the manual UI fallback.

- [ ] **Step 3: Verify CSV row count and parseability**

```bash
wc -l docs/superpowers/ops/cloudflare-redirects-mlaify-io.csv
python3 -c "
import csv
with open('docs/superpowers/ops/cloudflare-redirects-mlaify-io.csv') as f:
    r = csv.DictReader(f)
    rows = list(r)
print(f'{len(rows)} redirect rules')
for row in rows:
    print(f'  {row[\"source_url\"]:<45} -> {row[\"target_url\"]}')
"
```

Expected: `17` (16 rules + header) from `wc -l`; Python prints 16 redirect rules in order.

- [ ] **Step 4: Commit (the matthewd.xyz repo)**

```bash
git checkout -b phase-3-mlaify-io-redirects
git add docs/superpowers/ops/cloudflare-redirects-mlaify-io.csv
git commit -m "Add Cloudflare Bulk Redirects CSV for mlaify.io sunset

16 path-preserving 301 redirects from mlaify.io paths to matthewd.xyz
equivalents. Used by Phase 3 of the merge — uploaded to Cloudflare
Bulk Redirects dashboard (see ops/phase-3-runbook.md). Last entry is
the catchall.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 2: Write the Cloudflare runbook

**Files:**
- Create: `docs/superpowers/ops/phase-3-runbook.md`

A self-contained checklist for the user to execute the Cloudflare changes. This is the "how to apply the CSV" doc.

- [ ] **Step 1: Write `docs/superpowers/ops/phase-3-runbook.md`**

```markdown
# Phase 3 Runbook — mlaify.io Redirects & DNS Sunset

This runbook walks through the manual Cloudflare changes for Phase 3.
The CSV used in Step 2 is at `cloudflare-redirects-mlaify-io.csv` in this directory.

## Prerequisites

- Cloudflare dashboard access for the `mlaify.io` zone.
- Cloudflare dashboard access for the `opensift.org` zone.
- Cloudflare dashboard access for the `siftbook.org` zone.
- The CSV file `cloudflare-redirects-mlaify-io.csv` (open in another tab — you'll upload it).

## Step 1: Verify current Cloudflare account state

In the Cloudflare dashboard, confirm all three zones (`mlaify.io`, `opensift.org`, `siftbook.org`) are present and active. If any are on a different account, this runbook assumes single-account access.

## Step 2: Create the Bulk Redirects list

Cloudflare's Bulk Redirects has two parts: **Lists** (the redirect entries) and **Rules** (which lists apply to traffic). You'll create one List and one Rule that references it.

1. Open the Cloudflare account-level dashboard (not zone-level).
2. Navigate to **Bulk Redirects** (currently under Rules → Redirect Rules → Bulk Redirects, may move).
3. Click **Create a Bulk Redirect List**.
4. Name: `mlaify-io-sunset`
5. Description: `Phase 3 of matthewd.xyz × mlaify merge — 301 mlaify.io paths to matthewd.xyz`
6. Click **Upload CSV** (or import file) and upload `cloudflare-redirects-mlaify-io.csv`.
7. Cloudflare will validate the rows. Fix any rejected rows (most likely cause: a column header mismatch — re-check Cloudflare's current expected format).
8. Click **Save**.

## Step 3: Create the Bulk Redirect Rule that applies the list

1. Back in **Bulk Redirects**, click **Create a Bulk Redirect Rule**.
2. Name: `mlaify-io-sunset-rule`
3. List: `mlaify-io-sunset` (the one you just created).
4. Click **Deploy**.

The rule is now active. Cloudflare's edge will start applying the redirects within ~30 seconds.

## Step 4: Smoke-test the redirects

In a terminal (works without bot-challenge issues because curl follows 301s without invoking JS challenge):

```bash
# Should each 301 to the matthewd.xyz equivalent
for path in / /aegis/ /aegis/architecture/ /attackmap/ /attackmap/analyzers/ /docs/project-omekarapper/ /docs/project-opensift/ /docs/project-opencontractrx/ /build-principles/ /contribute/ /about/ /privacy/ /security/ /humans.txt /.well-known/security.txt /privacy.json; do
  echo "--- mlaify.io$path ---"
  curl -sI "https://mlaify.io$path" -o /tmp/h.txt
  grep -iE "HTTP|^location:" /tmp/h.txt | head -2
done
```

Expected: every entry shows `HTTP/2 301` (or 308) and a `location:` pointing at `matthewd.xyz` with the correct rewritten path.

If any rule misses, check the Cloudflare dashboard → Bulk Redirects → List entries for typos. Then re-run the smoke.

## Step 5: Remove DNS records for opensift.org and siftbook.org

Both side domains were CNAME'd at GitHub Pages via the mlaify.github.io repo's CNAME file. After Task 4 of this plan trims the CNAME file, those domains stop being claimed by GH Pages. The corresponding DNS records need removing:

For each zone (`opensift.org` and `siftbook.org`):

1. Open the zone in Cloudflare → **DNS** → **Records**.
2. Identify the CNAME or A records pointing at GitHub Pages (`<org>.github.io` or `185.199.108.x` / `185.199.109.x` / `185.199.110.x` / `185.199.111.x` IPs).
3. **Delete** those records.

The domain will start returning DNS errors for HTTP requests. If you want to preserve the registration (in case you want to reuse it later) but make it inactive, that's enough. If you want to fully release, also let the registration lapse at the registrar.

## Step 6: Validate side domains are gone

```bash
curl -sI https://opensift.org/ 2>&1 | head -3
curl -sI https://siftbook.org/ 2>&1 | head -3
```

Expected: connection failure or NXDOMAIN-style error (`Could not resolve host`). If you see a 200/301 response, DNS records still exist — re-check Step 5.

## Step 7: Final mlaify.io smoke

```bash
# Apex catchall
curl -sIL https://mlaify.io/anything-not-in-the-rules | grep -iE "HTTP|^location:" | head
```

Expected: 301 to `matthewd.xyz/anything-not-in-the-rules` (suffix-preserved catchall) OR 301 to `matthewd.xyz/projects/` if the catchall is configured exact-match. Either is acceptable for a sunset.
```

- [ ] **Step 2: Commit**

```bash
git add docs/superpowers/ops/phase-3-runbook.md
git commit -m "Add Phase 3 runbook for Cloudflare ops

Step-by-step manual procedure: upload the redirects CSV, create the
rule, smoke-test 16 paths via curl, sunset opensift.org and
siftbook.org DNS, final validation. Self-contained so the runbook
can be re-applied if any rule drifts.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Task 3: USER ACTION — Apply Cloudflare Bulk Redirects

**Files:** none (manual ops)

This task is performed by the user in the Cloudflare dashboard, following the runbook from Task 2.

- [ ] **Step 1: User applies the Bulk Redirects list and rule**

Follow `docs/superpowers/ops/phase-3-runbook.md` Steps 1–3. Reports back when the rule is deployed.

- [ ] **Step 2: Verify redirects are live via curl**

Run the smoke test from the runbook Step 4. Capture the output. Every entry must show 301 + location pointing at matthewd.xyz.

If any redirect is wrong or missing, the user fixes the corresponding entry in the Cloudflare dashboard and re-runs the smoke. Don't proceed to Task 4 until all 16 rules return correct 301s.

- [ ] **Step 3: No commit — just a status update**

Reply with the smoke-test output. The plan moves on to Task 4.

---

## Task 4: Update `mlaify/mlaify.github.io` — trim CNAME, replace content with archived notice

**Files (in `/Volumes/Dev/repos/GitHub/mlaify/mlaify.github.io`, a separate repo):**
- Modify: `static/CNAME` (or `CNAME` at repo root, whichever GH Pages uses)
- Modify: `content/_index.md`

The Cloudflare redirects in Tasks 1–3 already handle mlaify.io. This task is defense-in-depth: if a Cloudflare rule ever fails, the mlaify.github.io origin still returns a useful "moved" page instead of stale content. And it removes the side domain claims.

- [ ] **Step 1: Switch to the mlaify repo**

```bash
cd /Volumes/Dev/repos/GitHub/mlaify/mlaify.github.io
git checkout main
git pull origin main
git status
```

Expected: clean tree on main.

- [ ] **Step 2: Trim `CNAME` to just `mlaify.io`**

The repo has two CNAME files: `./CNAME` (3 domains) and `./static/CNAME` (1 domain). The one GitHub Pages actually uses is generated by Hugo from `static/CNAME` (the static dir is what GH Pages serves). But the repo-root `CNAME` may also be honored. Both must be trimmed.

Read the current file:

```bash
cat CNAME
cat static/CNAME
```

If `CNAME` (root) contains `mlaify.io`, `opensift.org`, `siftbook.org` — overwrite to just:

```
mlaify.io
```

If `static/CNAME` already contains just `mlaify.io`, leave it. Otherwise overwrite to the same content.

Edit with:

```bash
echo "mlaify.io" > CNAME
echo "mlaify.io" > static/CNAME
```

- [ ] **Step 3: Replace `content/_index.md` with an archived-notice page**

The current `content/_index.md` is the mlaify hero. Overwrite the body. Front-matter can stay; just replace what's below the closing `---`.

The new body:

```markdown
# This site has moved.

mlaify content now lives at **[matthewd.xyz](https://matthewd.xyz/)**.

- [Aegis](https://matthewd.xyz/aegis/) — post-quantum encrypted messaging
- [AttackMap](https://matthewd.xyz/attackmap/) — defensive security analysis
- [Projects hub](https://matthewd.xyz/projects/) — all five projects
- [About](https://matthewd.xyz/about/)
- [Build principles](https://matthewd.xyz/principles/)
- [Contribute](https://matthewd.xyz/contribute/)

Most old URLs (`mlaify.io/aegis/...`, `mlaify.io/attackmap/...`) 301-redirect automatically. If you landed here from a stale link that didn't redirect, please [file an issue at the new home](https://github.com/mdavistffhrtporg/mdavistffhrtporg.github.io/issues) so the redirect rule can be fixed.

This repository is archived as of June 2026.
```

- [ ] **Step 4: Build and verify**

```bash
npm run build 2>&1 | tail -5 || hugo --gc --minify 2>&1 | tail -5
ls public/CNAME && cat public/CNAME
ls public/index.html
```

Expected: build succeeds, `public/CNAME` contains only `mlaify.io`, `public/index.html` exists and contains "This site has moved."

- [ ] **Step 5: Commit and push**

```bash
git add CNAME static/CNAME content/_index.md
git commit -m "Replace site with moved notice; trim CNAME to mlaify.io only

Phase 3 of the matthewd.xyz × mlaify merge:
- opensift.org and siftbook.org removed from CNAME so GH Pages stops
  claiming them. DNS records for those domains will be removed at
  Cloudflare separately.
- Homepage replaced with a static \"This site has moved\" notice
  linking to matthewd.xyz. Cloudflare 301-redirects handle most
  traffic at the edge; this is defense if a rule ever misses.

The repo will be archived on GitHub after this lands.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
git push origin main
```

- [ ] **Step 6: Wait for GH Pages deploy and verify**

```bash
sleep 30
curl -sIL -H "User-Agent: Mozilla/5.0" https://mlaify.io/ 2>&1 | grep -iE "HTTP|^location:" | head -3
```

Expected: 301 to matthewd.xyz/projects/ (because Cloudflare rules from Task 3 fire first). If you see the new mlaify.github.io content directly, the Cloudflare rule for `mlaify.io/` isn't matching — fix in the dashboard before proceeding.

If the Cloudflare rule is bypassed (e.g., DNS pointed directly at GH Pages without Cloudflare proxy enabled), the user is responsible for ensuring Cloudflare's orange-cloud proxy is on for the apex record.

---

## Task 5: USER ACTION — Remove DNS records for opensift.org and siftbook.org

**Files:** none (manual ops)

- [ ] **Step 1: User removes DNS records for both side domains in Cloudflare**

Follow `docs/superpowers/ops/phase-3-runbook.md` Step 5. Removes GitHub Pages A/CNAME records from both zones.

- [ ] **Step 2: Verify both side domains stop responding**

```bash
curl -sI https://opensift.org/ 2>&1 | head -3
curl -sI https://siftbook.org/ 2>&1 | head -3
```

Expected: connection failure (`Could not resolve host`) or, if the domain still resolves via Cloudflare but has no origin, a Cloudflare error page. Either is acceptable — the domains are no longer serving mlaify content.

- [ ] **Step 3: No commit — status update only**

---

## Task 6: Archive `mlaify/mlaify.github.io` repo

**Files:** none in matthewd repo (GitHub API call)

After redirects work and the side domains are gone, freeze the source repo so no future commits accidentally undo any of this.

- [ ] **Step 1: Confirm the repo is in a clean state for archiving**

```bash
gh repo view mlaify/mlaify.github.io --json isArchived,defaultBranchRef,pushedAt
```

Expected: `isArchived: false`, `defaultBranchRef: main`, `pushedAt` matches the Task 4 commit.

- [ ] **Step 2: Archive the repo**

```bash
gh repo archive mlaify/mlaify.github.io --yes
```

Expected: no error. The repo becomes read-only on GitHub. Existing GH Pages deploys continue serving (archive doesn't disable Pages — only further pushes).

- [ ] **Step 3: Verify archive**

```bash
gh repo view mlaify/mlaify.github.io --json isArchived
```

Expected: `{"isArchived": true}`.

- [ ] **Step 4: No commit — this is a GitHub-side action**

---

## Task 7: Final smoke + matthewd.xyz CHANGELOG entry

**Files (matthewd.xyz repo):**
- Modify: `CHANGELOG.md`

- [ ] **Step 1: Switch back to the matthewd repo and the Phase 3 branch**

```bash
cd /Volumes/Dev/repos/GitHub/mdavistffhrtporg/mdavistffhrtporg.github.io
git checkout phase-3-mlaify-io-redirects
```

- [ ] **Step 2: Run the full smoke gauntlet**

```bash
echo "=== mlaify.io redirects ==="
for path in / /aegis/ /aegis/architecture/ /attackmap/ /attackmap/analyzers/ /docs/project-omekarapper/ /docs/project-opensift/ /docs/project-opencontractrx/ /build-principles/ /contribute/ /about/ /privacy/ /security/ /humans.txt /.well-known/security.txt /privacy.json /catchall-test; do
  status=$(curl -so /dev/null -w "%{http_code}" "https://mlaify.io$path")
  location=$(curl -sI "https://mlaify.io$path" | grep -i "^location:" | tr -d '\r' | head -1)
  echo "$status mlaify.io$path -> $location"
done

echo "=== side domains gone ==="
for d in opensift.org siftbook.org; do
  result=$(curl -sI "https://$d/" 2>&1 | head -1)
  echo "$d -> $result"
done

echo "=== matthewd.xyz still serving ==="
for path in / /writing/ /aegis/ /projects/ /about/; do
  status=$(curl -so /dev/null -w "%{http_code}" -A "Mozilla/5.0" "https://matthewd.xyz$path")
  echo "$status matthewd.xyz$path"
done
```

Expected:
- All 17 `mlaify.io` paths show 301 with the right `location:`.
- Both side domains show connection failure or Cloudflare error.
- All 5 `matthewd.xyz` paths still 200.

If anything is wrong, fix in Cloudflare and re-run.

- [ ] **Step 3: Add CHANGELOG entry**

Open `CHANGELOG.md`. Under the existing `[Unreleased]` section, append to `### Changed`:

```markdown
- mlaify.io paths now 301-redirect to matthewd.xyz via Cloudflare Bulk Redirects (Phase 3). The redirect list is checked into `docs/superpowers/ops/cloudflare-redirects-mlaify-io.csv`; the runbook is at `docs/superpowers/ops/phase-3-runbook.md`.
- mlaify/mlaify.github.io repo archived as defense-in-depth (origin still serves a "moved" notice if a Cloudflare rule ever misses).
```

Add to `### Removed`:

```markdown
- Side domains opensift.org and siftbook.org sunset (CNAME claim removed from mlaify.github.io, DNS records deleted in Cloudflare).
```

- [ ] **Step 4: Commit and push**

```bash
git add CHANGELOG.md
git commit -m "CHANGELOG: Phase 3 — mlaify.io redirects + repo archive

Documents the Phase 3 changes: Cloudflare Bulk Redirects for mlaify.io
paths, side-domain sunset (opensift.org, siftbook.org), and the
mlaify/mlaify.github.io repo archival.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
git push -u origin phase-3-mlaify-io-redirects
```

- [ ] **Step 5: Open the PR**

```bash
gh pr create --title "Phase 3: mlaify.io redirects + repo archive (docs only)" --body "$(cat <<'EOF'
## Summary

Documents and verifies the Phase 3 changes from the matthewd.xyz × mlaify merge:

- **Cloudflare Bulk Redirects** for all mlaify.io paths → matthewd.xyz equivalents (CSV in `docs/superpowers/ops/`).
- **Side domain sunset** — opensift.org and siftbook.org no longer claimed by GH Pages or pointed at the old origin.
- **Repo archive** — mlaify/mlaify.github.io marked read-only on GitHub.
- **Runbook** — `docs/superpowers/ops/phase-3-runbook.md` documents the manual Cloudflare procedure so it's reproducible.

This PR is **docs-only** for matthewd.xyz — no theme, content, or workflow changes. The actual user-visible changes happened in the Cloudflare dashboard and in the mlaify/mlaify.github.io repo (separately committed and archived).

This is the final phase of the merge described in [docs/superpowers/specs/2026-06-03-matthewd-mlaify-merge-design.md](docs/superpowers/specs/2026-06-03-matthewd-mlaify-merge-design.md).

## Test plan

- [ ] All 17 mlaify.io paths return 301 with correct `Location:`.
- [ ] opensift.org and siftbook.org no longer respond (connection failure or Cloudflare error).
- [ ] matthewd.xyz still serves all routes (no regression).
- [ ] mlaify/mlaify.github.io shows "Archived" badge on GitHub.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

- [ ] **Step 6: Admin-merge (same pattern as Phase 1 and 2)**

```bash
gh pr merge --merge --admin --delete-branch
```

---

## Open questions for the user

1. **Cloudflare account access.** This plan assumes you have dashboard access to all three zones. If a zone is on a different account or org, the runbook needs adjustment.
2. **opensift.org / siftbook.org registration.** The plan removes DNS records but doesn't address the domain registrations themselves. If you want to fully release the domains, let them lapse at the registrar after a few months. If you want to keep them parked, the removed DNS records are enough.
3. **External README updates.** The mlaify/* org has multiple project repos whose READMEs reference `mlaify.io/...`. Phase 3 does NOT update those — the Cloudflare catchall handles them. If you want explicit updates, that's a small follow-up PR per repo.
4. **Cloudflare CSV format drift.** Cloudflare's Bulk Redirects CSV schema occasionally changes (column names, boolean representations). If the upload in Task 3 rejects rows, fall back to the dashboard's manual entry UI — same fields, same values.

## Risks

| # | Risk | Mitigation |
|---|------|------------|
| 1 | Cloudflare rule misses a path | Catchall rule + defense-in-depth "moved" notice in mlaify.github.io |
| 2 | DNS propagation lag for side-domain removal | Wait 5+ minutes after Step 5 of the runbook; some resolvers cache up to 1h |
| 3 | mlaify.github.io GH Pages keeps serving after CNAME trim | The CNAME change tells GH Pages to stop claiming the side domains; Cloudflare DNS removal makes them unreachable regardless |
| 4 | External tools (RSS readers, search indexes) may take days to follow 301s | Acceptable — 301 is the right signal, downstream catches up |
| 5 | Repo archive accidentally disables GH Pages | Archiving does NOT disable Pages; only blocks further pushes. Verified per GitHub docs |

## Out of scope (defer to a future "polish" pass)

- External README updates in mlaify/aegis-spec, mlaify/attackmap, etc.
- Package metadata updates (PyPI attackmap, etc.) where they reference mlaify.io.
- Social profile bio link updates (Bluesky, GitLab) if they mention mlaify.
- Search engine resubmission / sitemap pings.

## Self-review notes

1. **Spec coverage:**
   - mlaify.io → matthewd.xyz 301s ✓ (Tasks 1–3)
   - Side domain sunset ✓ (Tasks 4 + 5 — added in this plan; not in original spec but per the new decision)
   - Repo archive ✓ (Task 6)
   - Test plan ✓ (Task 7)
2. **Placeholder scan:** Cloudflare CSV format is the only uncertain field — the plan notes the manual UI fallback. No TBDs.
3. **Type/name consistency:** All redirect targets use `matthewd.xyz/` consistently. `/principles/` (not `/build-principles/`), `/projects/` (not `/docs/projects/`), `/about/` (not `/me/`) — all match the Phase 2 IA.

Plan complete.
