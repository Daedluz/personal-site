# daedluz.com

Personal site built with [Hugo](https://gohugo.io/) and the
[PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, deployed to
GitHub Pages at <https://www.daedluz.com/>.

## Setup

The theme is a git submodule, so clone with it:

```bash
git clone --recurse-submodules <repo-url>
# or, in an existing clone:
git submodule update --init --recursive
```

You need the **extended** build of Hugo (`brew install hugo` on macOS gives
this). CI builds with Hugo 0.148.2 — see `.github/workflows/hugo.yaml`.

## Local preview

```bash
hugo server -D
```

Then open <http://localhost:1313/>. `-D` includes drafts; drop it to see the
site exactly as it will be published. The server live-reloads on save.

## Changing the personal intro

Everything on the landing page lives in `hugo.toml`:

- `[params.homeInfoParams]` — `Title` and `Content` are the greeting block.
  `Content` is rendered as Markdown, so `[text](/url/)` links and `*emphasis*`
  work. Raw HTML is stripped (Goldmark runs with `unsafe` off), so use Markdown
  rather than `<a>` tags.
- `[[params.socialIcons]]` — one block per icon under the intro. `name` picks
  the icon (`github`, `email`, `linkedin`, `other`, …), `url` is the link.
- `title` at the top of the file — the site title in the header and browser tab.

## Adding a new article

```bash
hugo new content posts/my-new-post.md
```

This creates the file from `archetypes/default.md`, pre-filled with TOML front
matter. Edit the front matter and write the body in Markdown:

```toml
+++
date = '2026-08-25T10:00:00-04:00'
draft = false
title = 'My New Post'
math = true   # optional, see below
+++
```

Set `draft = false` when it is ready — drafts are not built for production.
Posts appear automatically in the post list; no index needs updating.

### Math

KaTeX is loaded by `layouts/partials/extend_head.html` whenever a page sets
`math = true` in front matter (or via the site-wide `math` param in
`hugo.toml`). Use `$…$` for inline math and `$$…$$` for display math.

## Updating the CV

The PDF lives at `static/cv.pdf` and is served at `/cv.pdf`. To publish a new
version, overwrite that file and commit it — nothing else needs to change.

The page around it is `content/cv.md`, which links the PDF and embeds it with
the `{{< pdf src="/cv.pdf" >}}` shortcode (`layouts/shortcodes/pdf.html`). The
embed is hidden on screens under 768px, where mobile browsers refuse to render
inline PDFs — the download link covers that case.

Its entry points are both in `hugo.toml`: the `CV` item in `[[menu.main]]` and
the `cv` entry in `[[params.socialIcons]]`. Menu `identifier` values must match
a real page path (`cv`, `posts`) — PaperMod's header does `site.GetPage .KeyName`
and the build fails if that resolves to nothing.

## Publishing

Pushing to `main` triggers `.github/workflows/hugo.yaml`, which builds the site
and deploys it to GitHub Pages. Nothing needs to be built by hand — the
committed `public/` directory is generated output and can be ignored.
