# aaabramov.github.io

Personal Jekyll knowledge base. Deploys to https://blog.iamandrii.com via GitHub Pages on push to `master`.

## Tutorials

Interactive tutorials are **self-contained static HTML** at `tutorials/<slug>/index.html` — inline CSS + vanilla JS, no build step, no bundler, no external deps (except optional CDN libs, e.g. transformers.js in `embeddings`). Use the dark-theme CSS variables and numbered-`<section>` structure of `tutorials/embeddings/index.html` as the template.

Register each tutorial in `_data/tutorials.yml` (`slug`, `title`, `subtitle`, `tags`, `published`); the homepage renders one card per entry, sorted by `published` descending.

## Preview locally

Tutorials are static, so preview them without Jekyll: `python3 -m http.server 8753`, then open `http://localhost:8753/tutorials/<slug>/`.

`bundle exec jekyll serve` is currently broken on this machine (system Ruby 2.6 / bundler version mismatch vs `Gemfile.lock`). Jekyll is only needed to preview the Liquid-templated homepage (`index.html`); GitHub Pages builds the full site on push regardless.

## Publishing

Branch → commit → push → open PR with `gh` → merge to `master`; Pages redeploys automatically. Stage only the files for the task — the working tree often carries unrelated in-progress changes (`_config.yml`, layouts, etc.), so never `git add -A`.
