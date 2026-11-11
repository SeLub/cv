# Domain & Hosting Setup Guide — lubimoff.com

## Current state (before this guide)

| Item | Status |
|---|---|
| CV site | GitHub Pages, served from branch `lubimov` of `SeLub/cv` → https://selub.github.io/cv/ |
| Domain `lubimoff.com` | Bought on Cloudflare, **not attached, not configured** |
| DNS zone | Cloudflare (registrar) |

Goal: make the CV available on the domain. Both options below are fully free.

---

## Option 1 — Cloudflare Pages (RECOMMENDED)

**Why:** the DNS zone is already on Cloudflare, so the custom domain and SSL certificate are
configured in one click. You also get automatic deploys on every `git push`, preview URLs for
other branches, a global CDN with unlimited free bandwidth — and you avoid the classic GitHub
Pages + Cloudflare proxy (orange cloud) SSL/redirect-loop pitfalls. The `_redirects` file already
committed to this repo makes the site answer on both `/` and `/cv/`.

### Steps

1. Push this branch to GitHub:
   ```bash
   git push origin lubimov
   ```
2. Cloudflare Dashboard → **Workers & Pages → Create application → Pages → Connect to Git**.
   Authorize GitHub, select the repository `SeLub/cv`, then **Begin setup** with:
   - Production branch: `lubimov`
   - Framework preset: `None`
   - Build command: *(empty)*
   - Build output directory: `/`
3. After the first deploy the project is live at `https://<project-name>.pages.dev`.
4. Project → **Custom domains → Set up a custom domain** → add `lubimoff.com`, then repeat for
   `www.lubimoff.com`. Because the DNS zone is in the same Cloudflare account, it will offer to
   add the required CNAME records automatically — accept. The certificate is issued in minutes.
5. Path handling (already done via `_redirects` in the repo root):
   - `lubimoff.com/` → CV site
   - `lubimoff.com/cv/` → same CV site (rewrite, so the profile link `lubimoff.com/cv/` works)
6. After the domain is verified, you can disable GitHub Pages:
   `SeLub/cv → Settings → Pages → Remove site` (or keep it as a fallback — but then remember to
   update **both** copies on future changes to avoid content drift).
7. Update the references:
   - Profile Summary row `CV` → `https://lubimoff.com/cv/`
   - LinkedIn "Contact info → Website" → `https://lubimoff.com/cv/`
   - GitHub profile README / pinned repos, if they link `selub.github.io/cv`

### Rollback

If anything goes wrong, GitHub Pages keeps serving `selub.github.io/cv/` until you explicitly
disable it — the migration is reversible at any moment.

---

## Option 2 — Stay on GitHub Pages and attach the domain

1. Create a file named `CNAME` in the repo root containing one line:
   ```
   lubimoff.com
   ```
   (or set it via `SeLub/cv → Settings → Pages → Custom domain → lubimoff.com`, which creates
   the file for you).
2. Add DNS records in Cloudflare (set **Proxy status = DNS only / grey cloud** until the GitHub
   certificate is issued):

   | Type | Name | Value |
   |---|---|---|
   | A | `@` | `185.199.108.153` |
   | A | `@` | `185.199.109.153` |
   | A | `@` | `185.199.110.153` |
   | A | `@` | `185.199.111.153` |
   | AAAA | `@` | `2606:50c0:8000::153` |
   | AAAA | `@` | `2606:50c0:8001::153` |
   | AAAA | `@` | `2606:50c0:8002::153` |
   | AAAA | `@` | `2606:50c0:8003::153` |
   | CNAME | `www` | `selub.github.io` |

3. In Cloudflare: **SSL/TLS → Overview → Full**, and enable **Always Use HTTPS**.
4. Back in `SeLub/cv → Settings → Pages`: re-check the custom domain, wait for the
   DNS check, then **Enforce HTTPS** (certificate can take up to 24 hours).
5. Optionally enable the orange cloud (proxy) afterwards. **Test immediately**; if you get a
   redirect loop, set the records back to DNS only (grey cloud).

### Important path caveat

A custom domain attached to a **project** site replaces the path: the CV will be served at
`lubimoff.com/` and **`lubimoff.com/cv/` will 404** (GitHub does not rewrite project-site paths
under custom domains). If you need the exact `lubimoff.com/cv/` URL on GitHub Pages, you must
create a **user site** repo `SeLub/SeLub.github.io`, attach `lubimoff.com` to *that* repo (root
`index.html` = simple redirect to `/cv/`), and then the `SeLub/cv` project site automatically
becomes `lubimoff.com/cv/`.

---

## Comparison

| | Cloudflare Pages | GitHub Pages + domain |
|---|---|---|
| Custom domain setup | 1 click (same CF account) | 4 A + 4 AAAA + CNAME records |
| SSL certificate | automatic, minutes | manual, up to 24 h |
| Cloudflare proxy | native, no conflicts | known redirect-loop/SSL pitfalls |
| `lubimoff.com/cv/` path | works via `_redirects` (already committed) | only via extra user-site repo |
| `lubimoff.com/` root | serves the CV directly | serves the CV directly (project site) |
| Deploys | push → auto build | push → auto build |
| Bandwidth | unlimited (free tier) | 100 GB/month soft limit |
| Preview URLs per branch | yes | no |

## Recommendation

**Option 1 (Cloudflare Pages).** Same free price, fewer moving parts, the domain zone is already
in Cloudflare, and the `/cv/` path is preserved by the committed `_redirects` file.
