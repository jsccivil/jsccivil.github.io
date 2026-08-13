# Deploying the site to GitHub Pages

This folder is the finished, ready-to-push website. Push **this** folder — not the
`site/` folder from the design tool. (See "Why not push `site/` directly" at the bottom.)

You are creating a **new repository** under your existing account (`mohitcek`) with a
**custom domain**. Your own site at `mohitcek.github.io` is untouched — a user site and a
project site coexist happily.

About 20 minutes of work, then up to 24 hours of waiting for DNS.

---

## Step 0 — Register the domain

These files assume **`drjschauhan.com`**. Check availability at
[Cloudflare Registrar](https://www.cloudflare.com/products/registrar/) (at-cost, ~$10/yr) or
[Namecheap](https://www.namecheap.com). Avoid GoDaddy — renewal prices climb.

Alternatives if taken: `jschauhan.com`, `jschauhan.org`, `chauhan-civil.com`. Avoid `.info`,
`.xyz`, `.site` — they read as low-credibility on a CV.

**If you pick a different domain**, run this once inside this folder to update every file:

```bash
cd "/Users/mj/Desktop/Papa Documents/US Application Kit/site-deploy"
grep -rl "drjschauhan.com" . | xargs sed -i '' "s/drjschauhan\.com/yourdomain.com/g"
```

That covers `CNAME`, all four pages (canonical + social tags), `robots.txt` and `sitemap.xml`.
Confirm with `grep -r drjschauhan .` — it should print nothing.

---

## Step 1 — Create the repository

At <https://github.com/new>:

- **Name:** `jschauhan-site`
- **Visibility:** **Public** (Pages on a free account only serves public repos)
- **Do not** add a README, .gitignore or licence

---

## Step 2 — Push

```bash
cd "/Users/mj/Desktop/Papa Documents/US Application Kit/site-deploy"

git init
git add .
git commit -m "Publish academic site for Dr. J. S. Chauhan"
git branch -M main
git remote add origin https://github.com/mohitcek/jschauhan-site.git
git push -u origin main
```

If it asks for a password, GitHub no longer accepts account passwords — use a
[personal access token](https://github.com/settings/tokens), or switch the remote to SSH
(`git@github.com:mohitcek/jschauhan-site.git`).

Confirm on github.com that you see `index.html`, `career.html`, `publications.html`,
`recognition.html`, `404.html`, `CNAME`, `.nojekyll` and the `assets/` folder.

---

## Step 3 — Turn on Pages

Repo → **Settings → Pages**:

- **Source:** *Deploy from a branch*
- **Branch:** `main`, folder `/ (root)` → **Save**
- **Custom domain:** should already show `drjschauhan.com` (read from the `CNAME` file)

The site is live at `https://mohitcek.github.io/jschauhan-site/` within a couple of minutes,
before the domain resolves. Open it to confirm the deploy worked.

---

## Step 4 — Point the domain at GitHub

In your registrar's DNS panel, delete any pre-filled parking records, then add:

**Four A records** (bare domain):

| Type | Name | Value |
|---|---|---|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |

**Four AAAA records** (IPv6, recommended):

| Type | Name | Value |
|---|---|---|
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |

**One CNAME record** (so `www.` works):

| Type | Name | Value |
|---|---|---|
| CNAME | `www` | `mohitcek.github.io` |

> The CNAME value is your **account** domain, *not* the repo name — this trips up most people.
> On Cloudflare set the proxy status to **DNS only** (grey cloud), or certificate issuance fails.

Check it:

```bash
dig drjschauhan.com +short        # the four 185.199.x.x addresses
dig www.drjschauhan.com +short    # mohitcek.github.io
```

---

## Step 5 — Enforce HTTPS

Once DNS resolves, return to **Settings → Pages** and tick **Enforce HTTPS** (available after
GitHub issues the free certificate). A CV link that throws a security warning is worse than no
link at all.

Stuck on "certificate not yet created" after 24 hours? Remove the custom domain, save, re-add
it, save again — that re-triggers issuance.

---

## Step 6 — Final checks

- [ ] `https://drjschauhan.com` loads with a padlock
- [ ] `https://www.drjschauhan.com` redirects to the bare domain
- [ ] All four pages open and the nav links work
- [ ] **Download CV** downloads the PDF
- [ ] Paste the URL into WhatsApp or LinkedIn — the preview card shows his name and stats
- [ ] Open on a phone: no sideways scrolling, header wraps cleanly
- [ ] Submit the domain to [Google Search Console](https://search.google.com/search-console)

---

## Updating later

Edit the HTML, then:

```bash
cd "/Users/mj/Desktop/Papa Documents/US Application Kit/site-deploy"
git add .
git commit -m "Update publications"
git push
```

Live in 1–2 minutes. Hard-refresh (⌘⇧R) if a change doesn't appear.

**When the CV is revised:** drop the new PDF in, keeping the filename `Dr_JS_Chauhan_CV.pdf`,
then commit and push. Every download link keeps working.

---

## What's in this folder

| File | Purpose |
|---|---|
| `index.html` | Home — hero, stats, research areas, timeline, contact |
| `career.html` | Appointments, CETDEC, funded projects |
| `publications.html` | Selected publications |
| `recognition.html` | Awards, patents, international standing |
| `404.html` | Styled not-found page |
| `assets/ds/styles.css` | Design-system stylesheet from the design tool |
| `assets/site.css` | Responsive layer added on top (see below) |
| `assets/portrait.jpg` | Portrait, re-encoded from 1.3 MB to 122 KB |
| `Dr_JS_Chauhan_CV.pdf` | Served by the Download CV buttons |
| `CNAME` | Tells GitHub which domain serves this repo |
| `og-image.png` | Link preview card for LinkedIn/WhatsApp/X |
| `favicon.svg` | Browser-tab icon |
| `robots.txt`, `sitemap.xml` | Search-engine indexing |
| `.nojekyll` | Stops GitHub running the files through Jekyll |

---

## Why not push `site/` directly

The design tool's export is a **prototype**, not a deployable site. Three things would have
broken in production, all fixed in this folder:

1. **It rendered nothing without JavaScript.** Every page loaded React, ReactDOM and Babel
   (~2.5 MB) from `unpkg.com` at page load and rendered through that runtime. If unpkg is slow,
   blocked (some university and corporate networks do block CDNs), or simply has a bad day, the
   visitor got a **blank page**. Those pages are now plain HTML — no JavaScript at all.
2. **The stylesheet lived in `_ds/`.** GitHub Pages runs Jekyll, which silently ignores any
   directory beginning with an underscore — the CSS would have 404'd and the site would have
   appeared completely unstyled. It now lives in `assets/ds/` and `.nojekyll` is shipped as well.
3. **Unfinished variant blocks.** The homepage carried two alternative headlines and two portrait
   treatments in `<sc-if>` tags for the design tool to choose between. Without the runtime, both
   showed at once. The authored defaults are now baked in — the "Low cost is not low quality."
   headline and the duotone portrait.

Also fixed: the layout overflowed horizontally at every width below 1280px (up to 948px of
sideways scroll on a phone), because the export uses fixed pixel grids with no media queries.
`assets/site.css` adds the breakpoints, and hover states — which were runtime-only attributes —
are now real CSS.

**If you re-export from the design tool**, don't copy the raw export over this folder. Re-run
the flattener in `../site-tools/` (`./build.sh`) and copy the result across, or ask me to.
