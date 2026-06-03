# Phase 3 Runbook — mlaify.io Redirects & DNS Sunset

This runbook walks through the manual Cloudflare changes for Phase 3 of the matthewd.xyz × mlaify merge.

The CSV used in Step 2 is at `cloudflare-redirects-mlaify-io.csv` in this directory.

## Prerequisites

- Cloudflare dashboard access for the `mlaify.io` zone.
- Cloudflare dashboard access for the `opensift.org` zone.
- Cloudflare dashboard access for the `siftbook.org` zone.
- The CSV file `cloudflare-redirects-mlaify-io.csv` open in another tab — you'll upload it.

## Step 1: Verify Cloudflare account state

In the Cloudflare dashboard, confirm all three zones (`mlaify.io`, `opensift.org`, `siftbook.org`) are present and active. If any are on a different account, this runbook assumes single-account access.

## Step 2: Create the Bulk Redirects List

Cloudflare's Bulk Redirects has two parts: **Lists** (the redirect entries) and **Rules** (which lists apply to traffic). You'll create one List and one Rule that references it.

1. Open the Cloudflare account-level dashboard (the home dashboard, not a specific zone).
2. Navigate to **Bulk Redirects** in the left sidebar. As of late 2025 this lives under **Rules → Redirect Rules → Bulk Redirects** at the account level. The exact path may move; search "Bulk Redirects" if it's not where expected.
3. Click **Create a Bulk Redirect List**.
4. **Name:** `mlaify-io-sunset`
5. **Description:** `Phase 3 of matthewd.xyz × mlaify merge — 301 mlaify.io paths to matthewd.xyz`
6. Click **Upload CSV** (or "import file"; the wording varies) and select `cloudflare-redirects-mlaify-io.csv`.
7. Cloudflare validates each row. If any are rejected, the most common cause is column-header drift between when the CSV was written and the current Cloudflare schema. Fix as follows:
   - Open the CSV in a text editor.
   - Compare the header row against Cloudflare's expected schema (shown in the upload dialog).
   - Rename columns or normalize boolean values (`true` → `Yes`, etc.) to match.
   - Re-save and re-upload.
8. Click **Save** once all 16 rules validate.

## Step 3: Create the Bulk Redirect Rule that applies the list

1. Back in **Bulk Redirects**, click **Create a Bulk Redirect Rule** (sometimes called "Create rule").
2. **Name:** `mlaify-io-sunset-rule`
3. **List:** select `mlaify-io-sunset` from the dropdown.
4. **Action:** ensure it's set to `Redirect`.
5. Click **Deploy** (or "Save and deploy").

The rule is now active on Cloudflare's edge. Propagation is typically under 30 seconds.

## Step 4: Smoke-test the redirects

In a terminal:

```bash
for path in / /aegis/ /aegis/architecture/ /attackmap/ /attackmap/analyzers/ /docs/project-omekarapper/ /docs/project-opensift/ /docs/project-opencontractrx/ /build-principles/ /contribute/ /about/ /privacy/ /security/ /humans.txt /.well-known/security.txt /privacy.json; do
  echo "--- mlaify.io$path ---"
  curl -sI "https://mlaify.io$path" -o /tmp/h.txt
  grep -iE "HTTP|^location:" /tmp/h.txt | tr -d '\r' | head -2
done
```

Expected for every entry:
```
HTTP/2 301
location: https://matthewd.xyz/<rewritten-path>
```

If any rule misses (you see a 200 or the wrong location), open the Cloudflare dashboard → Bulk Redirects → the List, and check that entry's settings. The most common causes:
- `subpath_matching` toggle not enabled when it should be.
- `preserve_path_suffix` toggle not enabled.
- Source URL has a different trailing slash than what was uploaded.

## Step 5: Remove DNS records for opensift.org and siftbook.org

Both side domains were CNAME'd at GitHub Pages via the mlaify.github.io repo's CNAME file. After plan Task 4 trims that CNAME, those domains stop being claimed by GH Pages. The DNS records also need removing.

For each zone (`opensift.org` and then `siftbook.org`):

1. Open the zone in Cloudflare → **DNS** → **Records**.
2. Identify the CNAME or A records pointing at GitHub Pages. Typical patterns:
   - CNAME → `<username>.github.io` or `mlaify.github.io`
   - A records → `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153` (GitHub Pages IP block)
3. Click **Edit** → **Delete** for each record.

The domain will start returning DNS errors for HTTP requests within a few minutes (subject to TTL on cached lookups). If you want to keep the registration but make the site inactive, that's enough. If you want to fully release, also let the registration lapse at your registrar.

## Step 6: Validate side domains are gone

```bash
curl -sI https://opensift.org/ 2>&1 | head -3
curl -sI https://siftbook.org/ 2>&1 | head -3
```

Expected: connection failure (`Could not resolve host`) or a Cloudflare error page. Either is acceptable — the domains are no longer serving mlaify content.

If you still see a 200/301 response after several minutes, DNS removal didn't take — re-check Step 5.

## Step 7: Final mlaify.io smoke

```bash
curl -sIL https://mlaify.io/anything-not-in-the-rules | grep -iE "HTTP|^location:" | tr -d '\r' | head
```

Expected: 301 to `https://matthewd.xyz/anything-not-in-the-rules` (the catchall preserves the path suffix). If the catchall was deployed as exact-match instead, the location will be `https://matthewd.xyz/projects/` — also acceptable for a sunset.

## When you're done

Reply in chat with the output of Steps 4 and 6 (or just "redirects done, side domains gone"). I'll proceed with Task 4 (trim mlaify.github.io CNAME + archived notice), Task 6 (`gh repo archive`), and Task 7 (PR + CHANGELOG).

If anything in this runbook doesn't match Cloudflare's current UI, paste the discrepancy and I'll update the runbook before you proceed.
