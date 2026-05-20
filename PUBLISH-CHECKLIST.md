# Publish checklist — Vianium org bottom-up

> Last updated: 2026-05-19
> Owner: Angel Careaga
> Phase: Pre-publish — to be executed manually, single commit per repo,
> NO commits performed by AI assistants.

This document walks the manual publication sequence. The expectation
is **one commit per repository at the very end**, pushed by hand from
the local working trees at `D:\Projects\2026\VianiumProject\`.

---

## Pre-flight gate

Run the master audit before anything else:

```powershell
pwsh D:\Projects\2026\VianiumProject\tools\Validate-Org.ps1
```

Expected: exit code 0 ("Org is publish-ready"). Any FAIL must be
resolved before proceeding.

---

## Step 0 — Org-level setup on github.com

Do these in the GitHub UI (or `gh org`):

1. **Create the `vianium` organisation** under your personal account.
2. **Enable GitHub Sponsors** at the org level if you want the org-level
   Sponsor button (per-user @AngelCareaga sponsor link already works
   via `.github/FUNDING.yml` regardless).
3. **Add a profile picture + bio** matching the Vianium brand
   (`#00A0F0` accent, "Open-source ecosystem for Windows Phone 8.1").
4. **Default branch protection** — `main` requires PR for direct push
   (you can dismiss this rule for yourself if you're working solo).

---

## Step 1 — Create the meta repos

Push the meta repos FIRST so the org profile is live + the default
community files (`.github/FUNDING.yml`, ISSUE_TEMPLATE, ...) propagate.

```bash
# .github (org profile + default community files)
cd D:\Projects\2026\VianiumProject\.github
git init -b main
git add .
git commit -s -m "Initial public release as part of the Vianium org migration"
gh repo create vianium/.github --public --source . --push

# vianium-docs (architecture, ADRs, getting-started, donations.md)
cd D:\Projects\2026\VianiumProject\vianium-docs
git init -b main
git add .
git commit -s -m "Initial public release as part of the Vianium org migration"
gh repo create vianium/vianium-docs --public --source . --push
```

---

## Step 2 — Foundation repos (Apache 2.0, no Vianium deps)

Order matters because downstream repos reference these by URL in
their READMEs.

```bash
foundation_repos=(
  vianium-kernel
  vianium-managed-kernel
  vianium-crypto
  vianium-net
  vianium-tls
  vianium-http
  vianium-grpc
)

for r in "${foundation_repos[@]}"; do
  cd "D:/Projects/2026/VianiumProject/$r"
  git init -b main
  git add .
  git commit -s -m "Initial public release as part of the Vianium org migration"
  gh repo create "vianium/$r" --public --source . --push
done
```

---

## Step 3 — Domain repos (Apache 2.0, depend on foundation)

```bash
domain_repos=(
  vianium-mtproto
  vianium-mtproxy
  vianium-voip
  vianium-media
  vianium-image-palette
  vianium-innertube
  vianium-store
)

for r in "${domain_repos[@]}"; do
  cd "D:/Projects/2026/VianiumProject/$r"
  git init -b main
  git add .
  git commit -s -m "Initial public release as part of the Vianium org migration"
  gh repo create "vianium/$r" --public --source . --push
done
```

---

## Step 4 — Product repos (PolyForm-NC, depend on foundation+domain)

```bash
product_repos=(
  vianium-browser
  vianigram
  vianium-music
  vianium-localsend
)

for r in "${product_repos[@]}"; do
  cd "D:/Projects/2026/VianiumProject/$r"
  git init -b main
  git add .
  git commit -s -m "Initial public release as part of the Vianium org migration"
  gh repo create "vianium/$r" --public --source . --push
done
```

---

## Step 5 — Tag v0.1.0 on every repo

After all 20 are pushed, tag the initial release uniformly:

```bash
all_repos=(
  .github vianium-docs
  vianium-kernel vianium-managed-kernel vianium-crypto vianium-net
  vianium-tls vianium-http vianium-grpc
  vianium-mtproto vianium-mtproxy vianium-voip vianium-media
  vianium-image-palette vianium-innertube vianium-store
  vianium-browser vianigram vianium-music vianium-localsend
)

for r in "${all_repos[@]}"; do
  cd "D:/Projects/2026/VianiumProject/$r"
  git tag -a v0.1.0 -m "v0.1.0 - initial public release"
  git push origin v0.1.0
done
```

---

## Step 6 — Archive the legacy repos

Drop a `DEPRECATED.md` (already drafted at
[`vianium-docs/archive-notice.template.md`](archive-notice.template.md))
into each legacy repo, push it, then **Archive** the repo in the GitHub
UI (Settings → General → Danger Zone → Archive this repository).

| Legacy repo | New home |
|---|---|
| `D:\Projects\2026\WP\VianiumBrowser` (DEPRECATED.md already written) | [`vianium/vianium-browser`](https://github.com/vianium/vianium-browser) |
| `D:\Projects\2026\WP\Vianigram` (DEPRECATED.md already written) | [`vianium/vianigram`](https://github.com/vianium/vianigram) |
| `D:\Projects\2026\WP\VianiumMusic` (DEPRECATED.md already written) | [`vianium/vianium-music`](https://github.com/vianium/vianium-music) |
| `D:\Projects\2026\WP\PivoraLocalSend` (DEPRECATED.md already written) | [`vianium/vianium-localsend`](https://github.com/vianium/vianium-localsend) |

For each: commit + push the DEPRECATED.md, then archive on github.com.

---

## Step 7 — Publish the announcement

The announcement draft lives at
[`vianium-docs/announcement.md`](announcement.md). Choose ONE of:

1. **As a Discussion** on `vianium/.github` (most visible, low friction).
2. **As a Release note** attached to `v0.1.0` of `vianium-docs`.
3. **Posted to angelcareaga.com** + cross-linked from the org profile.

Recommended: do (1) immediately and (3) when there's time for a longer
write-up.

---

## Step 8 — Set up GitHub Sponsors (optional but recommended)

If you don't already have GitHub Sponsors enabled on the
`@AngelCareaga` profile:

1. https://github.com/sponsors/vianium/dashboard
2. Complete the application (tax form, payout details).
3. Create at least one tier (e.g. $5/month "Coffee" + $25/month
   "Supporter") so the existing FUNDING.yml link resolves.

The FUNDING.yml across all 20 repos already points at this profile,
so once Sponsors is live, every Sponsor button across the org becomes
clickable simultaneously.

Buy Me a Coffee — confirm the `angelcareaga` handle is correct at
https://www.buymeacoffee.com/soyangelcareaga and update FUNDING.yml +
re-run `tools\Apply-CommunityFiles.ps1 -Force` if it isn't.

---

## Post-publish smoke checks

After Step 5 completes, verify each public repo has:

- ✅ Sponsor button visible at the top of the repo page (after Sponsors
  is enabled).
- ✅ "Community Standards" green-checked under Insights → Community
  Standards.
- ✅ CI passes its first run (lint-only workflow; takes ~1 minute).
- ✅ License correctly detected by GitHub (top-right of the repo
  page shows "Apache-2.0 license" or "View license" for PolyForm-NC).

Any repo failing one of these gets a follow-up commit AFTER the
initial v0.1.0 push.

---

**Angel Careaga** · [angelcareaga.com](https://angelcareaga.com) · [@AngelCareaga](https://github.com/AngelCareaga)
