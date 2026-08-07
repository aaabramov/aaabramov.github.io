# aaabramov.github.io

Personal Jekyll knowledge base. Deploys to https://blog.iamandrii.com via GitHub Pages on push to `master`.

## Tutorials

Interactive tutorials are **self-contained static HTML** at `tutorials/<slug>/index.html` — one file, no build step, no bundler. Use the dark-theme CSS variables and numbered-`<section>` structure of `tutorials/embeddings/index.html` as the template.

**Libraries are fine — don't hand-roll to avoid a dependency.** This is a read-only blog, so the XSS risk that would normally argue against third-party JS doesn't apply. Pull libs from a CDN (pin exact versions, as `three@0.185.1` and `@xenova/transformers@2.17.2` already do) or vendor them. Guard every CDN dependency so the page still works when it's unreachable: `window.Prism && Prism.highlightAll()`, `try/catch` around dynamic `import()`, and animation libs used only for flourish on top of state the render functions already produce.

Register each tutorial in `_data/tutorials.yml` (`slug`, `title`, `subtitle`, `tags`, `published`); the homepage renders one card per entry, sorted by `published` descending. **Keep `published` dates unique** — the sort is on that field alone, so a tie leaves card order undefined.

### Palette: two accents that must never encode different things

The shared dark theme's `--accent` (`#7aa2f7`) and `--accent-2` (`#bb9af7`) are **indistinguishable as a categorical pair** — measured ΔE 0.3 under protanopia and 8.7 for normal vision. Never use them for two series in a chart, two columns in a comparison, or alternating highlight bands. They're fine as decoration (gradients, hover states) where they don't carry meaning.

- Verified-safe pair: `--accent` + `--warn` (`#e0af68`) — ΔE 22.6 protan, 24.6 normal.
- **No safe third hue exists in this palette** (`--good` collides with `--warn`, `--accent-3` with `--accent`, and `--bad`/`--good`/`--warn` are reserved status colours). For a third category, band on lightness within one hue, or use a non-colour encoding.
- Validate before shipping rather than eyeballing: the `dataviz` skill ships `scripts/validate_palette.js` — `node scripts/validate_palette.js "#hex,#hex" --mode dark --surface "#141925" --pairs all`.

## Preview locally

Tutorials are static, so preview them without Jekyll: `python3 -m http.server 8753`, then open `http://localhost:8753/tutorials/<slug>/`.

`bundle exec jekyll serve` is currently broken on this machine (system Ruby 2.6 / bundler version mismatch vs `Gemfile.lock`). Jekyll is only needed to preview the Liquid-templated homepage (`index.html`); GitHub Pages builds the full site on push regardless.

Since the homepage can't be previewed locally, at least confirm the data file parses — note `permitted_classes`, or the `published` dates raise `Psych::DisallowedClass`:

```sh
ruby -ryaml -e 'YAML.load_file("_data/tutorials.yml", permitted_classes: [Date]).each { |t| puts "#{t["published"]} #{t["slug"]}" }'
```

## Publishing

Branch → commit → push → open PR with `gh` → merge to `master`; Pages redeploys automatically. Stage only the files for the task — the working tree often carries unrelated in-progress changes (`_config.yml`, layouts, etc.), so never `git add -A`.
