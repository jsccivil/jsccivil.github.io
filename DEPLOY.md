# Deploying the site — jsccivil.github.io

This folder is the finished website, configured to be served from
**https://jsccivil.github.io** — a GitHub *user site* on Dr. Chauhan's own account.

No domain to buy, no DNS to configure, no certificate to wait for. HTTPS is automatic.

Push **this** folder, not the `site/` folder from the design tool.

---

## Step 1 — Create the repository

Signed in as **jsccivil**, go to <https://github.com/new>:

- **Repository name:** `jsccivil.github.io` — exactly that, including the `.github.io`
- **Public**
- Leave "Add a README", `.gitignore` and licence **unchecked**

The name is not cosmetic. A repo named `<username>.github.io` becomes a user site served
from the root of the domain. Any other name makes it a project site served from
`jsccivil.github.io/<repo-name>/`, and the "Home / Career" links would need rewriting.

### Let yourself push to it

You'll be maintaining this, not him. On the new repo: **Settings → Collaborators →
Add people → `mohitcek`**, then accept the invitation from your own account. Now you can
push with your own credentials instead of his.

---

## Step 2 — Push

```bash
cd "/Users/mj/Desktop/Papa Documents/US Application Kit/site-deploy"

# delete the CNAME file — it still points at the unregistered domain
git rm CNAME

# the previous attempt pointed at the old repo — repoint it
git remote remove origin
git remote add origin https://github.com/jsccivil/jsccivil.github.io.git

git add -A
git commit -m "Serve from jsccivil.github.io"
git push -u origin main
```

If it asks for a password: GitHub stopped accepting account passwords in 2021. Use a
[personal access token](https://github.com/settings/tokens) (classic, `repo` scope) as the
password, or `gh auth login` if you have the CLI.

> The `git rm CNAME` line matters. That file is what told GitHub to serve the site from
> `drjschauhan.com`, and it is what produced the "DNS check unsuccessful" error. If it stays
> in the repo, the new site will fail the same way. `git add -A` then picks up the rewritten
> canonical and social URLs across the four pages.

---

## Step 3 — Turn on Pages

Repo → **Settings → Pages**:

- **Source:** *Deploy from a branch*
- **Branch:** `main`, folder `/ (root)` → **Save**
- **Custom domain:** leave empty

Live at **https://jsccivil.github.io** in 1–2 minutes. First-ever deploy occasionally takes
closer to ten.

---

## Step 4 — Clean up the old repo

The earlier push went to `mohitcek/jschauhan-site`. Delete it, or at minimum remove its
custom domain (Settings → Pages). Two live copies of the same site split whatever search
ranking his name earns, and Google may pick the wrong one.

---

## Step 5 — Checks

- [ ] https://jsccivil.github.io loads with a padlock
- [ ] All four pages open; nav links work
- [ ] **Download CV** downloads the PDF
- [ ] Paste the URL into WhatsApp or LinkedIn — the preview card shows his name and stats
- [ ] Open on a phone: no sideways scrolling
- [ ] Submit to [Google Search Console](https://search.google.com/search-console) so the site
      surfaces when a committee searches his name

---

## Updating later

```bash
cd "/Users/mj/Desktop/Papa Documents/US Application Kit/site-deploy"
git add .
git commit -m "Update publications"
git push
```

Live in 1–2 minutes; hard-refresh (⌘⇧R) if you don't see it.

**Revised CV:** drop the new PDF in with the same filename, `Dr_JS_Chauhan_CV.pdf`, then
commit and push. Every download link keeps working.

---

## Adding a custom domain later

Nothing here blocks it. If you register one:

1. Add a file named `CNAME` containing just the domain, e.g. `drjschauhan.com`
2. Update the `<link rel="canonical">`, `og:url`, `og:image` and `twitter:image` tags in all
   four pages, plus `robots.txt` and `sitemap.xml` — or re-run `../site-tools/build.sh` with
   `SITE_DOMAIN=yourdomain.com`
3. At the registrar, four `A` records on `@` → `185.199.108.153`, `.109.153`, `.110.153`,
   `.111.153`, and a `CNAME` on `www` → `jsccivil.github.io`
4. Settings → Pages → Custom domain → enter it → Save, then tick **Enforce HTTPS**

GitHub keeps redirecting `jsccivil.github.io` to the new domain, so old links survive.

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
| `assets/site.css` | Responsive layer added on top |
| `assets/portrait.jpg` | Portrait, re-encoded from 1.3 MB to 122 KB |
| `Dr_JS_Chauhan_CV.pdf` | Served by the Download CV buttons |
| `og-image.png` | Link preview card |
| `favicon.svg` | Browser-tab icon |
| `robots.txt`, `sitemap.xml` | Search-engine indexing |
| `.nojekyll` | Stops GitHub running the files through Jekyll |

There is deliberately **no `CNAME` file** — that file is only for custom domains, and its
presence is what caused the "DNS check unsuccessful" error before.

---

## Why not push `site/` directly

The design tool's export is a prototype, not a deployable site. Three things would have broken:

1. **It rendered nothing without JavaScript.** Every page pulled React, ReactDOM and Babel
   (~2.5 MB) from `unpkg.com` at load and rendered through that runtime. Blocked or slow CDN
   meant a **blank page**. These pages are now plain HTML with no JavaScript.
2. **The stylesheet lived in `_ds/`.** GitHub Pages runs Jekyll, which ignores directories
   starting with an underscore — the CSS would have 404'd and the site would have appeared
   unstyled. It's now `assets/ds/`, with `.nojekyll` as backup.
3. **Unresolved variant blocks.** The homepage carried two headlines and two portrait
   treatments in `<sc-if>` tags; without the runtime both showed at once. The authored
   defaults are baked in.

Also fixed: horizontal overflow at every width below 1280px (up to 948px of sideways scroll on
a phone), since the export uses fixed pixel grids with no media queries.

**If you re-export the design**, don't copy it over this folder — run `../site-tools/build.sh`
and copy the result, or ask me to.
