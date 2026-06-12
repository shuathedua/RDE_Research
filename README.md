# Detonation Notes — Hugo blog starter

A ready-to-deploy Hugo blog using the PaperMod theme, with KaTeX math
rendering and a GitHub Actions workflow that publishes to GitHub Pages.

## What's included

```
rde-blog/
├── hugo.yaml                  # site config — edit the CHANGE THIS lines
├── content/
│   ├── about.md               # your about page — edit this
│   └── posts/
│       └── paper-review-template.md   # sample post + review structure
├── archetypes/posts.md        # template used by `hugo new`
├── layouts/partials/extend_head.html  # KaTeX loader
├── themes/PaperMod/           # theme, vendored (no submodule fuss)
└── .github/workflows/hugo.yml # auto-deploy on push to main
```

## Setup (one time, ~15 minutes)

1. **Install Hugo** (extended version) — on Windows: `winget install Hugo.Hugo.Extended`,
   on Mac: `brew install hugo`. Verify with `hugo version`.

2. **Edit `hugo.yaml`** — every line marked `CHANGE THIS`: your GitHub
   username, blog title, name, and social links. Also edit
   `content/about.md`.

3. **Create the GitHub repo.** On github.com, create a new **public** repo
   (e.g. `rde-blog`). Then from inside this folder:

   ```bash
   git init
   git add .
   git commit -m "Initial site"
   git branch -M main
   git remote add origin https://github.com/YOURUSERNAME/rde-blog.git
   git push -u origin main
   ```

4. **Enable GitHub Pages.** In the repo: Settings → Pages → under
   "Build and deployment", set **Source** to **GitHub Actions**.
   The included workflow handles everything else. Your site goes live at
   `https://YOURUSERNAME.github.io/rde-blog/` within a couple of minutes
   of each push.

## Daily writing workflow

```bash
hugo new posts/my-next-review.md   # creates a pre-structured draft
hugo server -D                     # live preview at localhost:1313
# write in any editor; the preview hot-reloads
# when ready: set draft: false in the front matter, then
git add . && git commit -m "Post: my next review" && git push
```

## Math

`math: true` is on site-wide. Write `$inline$` or `$$display$$` LaTeX
directly in Markdown:

```
$$ D_{CJ} \approx \sqrt{2(\gamma^2-1)\,q} $$
```

One gotcha: Markdown can eat backslashes/underscores in rare cases. If an
expression renders wrong, wrap it in a code-free line by itself or use
`\\(` `\\)` delimiters.

## Custom domain (optional, later)

Buy the domain, then in repo Settings → Pages add it as the custom domain,
create the DNS records your registrar/GitHub docs specify, and update
`baseURL` in `hugo.yaml`. Don't block on this — publish first.

## Updating the theme (occasional)

The theme is vendored in `themes/PaperMod` at **v8.0**, and the deploy
workflow pins **Hugo 0.145.0** to match. If you update the theme to a
newer release (https://github.com/adityatelange/hugo-PaperMod), also bump
`HUGO_VERSION` in `.github/workflows/hugo.yml` — PaperMod v9+ requires
Hugo 0.146 or newer. (Vendoring was chosen over git submodules to keep
cloning simple.)
