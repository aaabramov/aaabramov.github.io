# Semantic search from scratch — tutorial design

**Date:** 2026-08-05
**Path:** `tutorials/semantic-search-from-scratch/index.html`
**Audience:** junior-to-mid software engineers who have already read the `embeddings` tutorial
**Status:** approved by the repo owner, ready to build

## Goal

Teach everything between "I can turn text into a vector" and "I have a search system that
works on real data." This is an explicit sequel to `tutorials/embeddings/`, which ends by
brute-force ranking ten hard-coded sentences and name-drops RAG without teaching it.

**Do not re-teach:** what an embedding is, what cosine similarity is, or that similar
meanings cluster. Those are the previous tutorial's job. Link back to
`/tutorials/embeddings/` in section 1 and move on.

## Title & framing

- Title: **Semantic search from scratch**
- Tagline: everything between an embedding and a search system that actually works.
- Slug stays `semantic-search-from-scratch`.

## Non-negotiable decisions (already settled — do not revisit)

1. **Scope** is the full post-embedding pipeline: chunking → indexing → brute force vs ANN
   → hybrid search → reranking → RAG.
2. **The centerpiece is hybrid search** (section 5). ANN/HNSW is deliberately kept
   compact — it is not the star.
3. **Vectors are precomputed and baked in, with an optional live model toggle.**
4. **Code snippets are Scala 3.**

## Existing draft

`tutorials/semantic-search-from-scratch/` currently holds an abandoned, broken draft from
2026-06-04. Its markup is corrupted (CSS declarations comma-separated, `--warn:$e0af68`,
`class=""slb"`, truncated mid-section, zero JavaScript behind three empty demo containers).

- **Delete** `_css.txt` and `01_header.html` — scratch fragments, nothing salvageable.
- **Rewrite** `index.html` from scratch. Keep only its framing: the keyword-search-fails
  opening, the "my bike has a puncture" vs "how to fix a flat tire" example, and the
  pipeline spine.

## House conventions (from CLAUDE.md — follow exactly)

- Self-contained static HTML at `tutorials/<slug>/index.html`. No build step, no bundler.
- Clone the dark theme, CSS custom properties, `.kb-back` nav, numbered `<section>`
  scaffold and footer from `tutorials/embeddings/index.html`. Match it closely.
- CDN libraries are allowed and are the house pattern; pin exact versions, as
  `three@0.185.1` and `@xenova/transformers@2.17.2` already do.
- Register the tutorial in `_data/tutorials.yml` (`slug`, `title`, `subtitle`, `tags`,
  `published`). Homepage sorts by `published` descending.
- Preview with `python3 -m http.server 8753`. `bundle exec jekyll serve` is broken on this
  machine; do not try to fix it.
- **Stage only the files for this task.** The working tree carries unrelated changes.
  Never `git add -A`.

## Dependencies

Pinned ESM/scripts from jsdelivr, each guarded so the page still works if the CDN is
unreachable:

- **Prism** `1.29.0` — core, then `components/prism-java.min.js`, then
  `components/prism-scala.min.js`. Order matters: the Scala grammar is
  `Prism.languages.extend("java", …)` and silently fails to register without Java first.
  Its keyword list covers Scala 3 (`enum`, `given`, `extension`, `derives`).
  Theme Prism's tokens to the existing palette in CSS; do not load a Prism theme stylesheet.
- **`@xenova/transformers@2.17.2`** — only behind the optional "load the real model"
  toggle, lazily imported on click. It must never load on page load.

Guard everything: `window.Prism && Prism.highlightAll()`, and a `try/catch` around any
dynamic import.

## The corpus and its vectors

**Do not invent embedding vectors, and do not fake similarity scores.** Every number on
the page must come from the real model.

Procedure:

1. Write a corpus of ~24 short documents modelling a **bike-workshop support knowledge
   base**. The topic is chosen because it naturally mixes:
   - exact tokens that only keyword search can catch — model numbers like
     `Shimano XT M8100`, error codes, part numbers;
   - paraphrasable how-tos that only vector search can catch — "how to fix a flat tire"
     retrieved by "my bike has a puncture".
2. Compute real `all-MiniLM-L6-v2` embeddings for every document and for each preset
   query by driving transformers.js in a Playwright page (load a local page that imports
   `@xenova/transformers`, embed the strings, dump the vectors as JSON). Mean-pool and
   L2-normalise exactly as the `embeddings` tutorial does.
3. Bake the vectors into `index.html` rounded to 3 decimals. ~24 docs × 384 dims is
   roughly 70 KB; a final page near 200 KB is acceptable.
4. The optional live-model toggle then handles arbitrary free-text queries.

If the Playwright route fails, stop and report it — do not substitute hand-written
vectors.

## Sections

**1 · Where the last tutorial left off.** Ten sentences, brute-force ranked. Real corpora
are 100k documents of varying length, queries contain product codes and typos, and `O(n)`
starts to bite. Name the five gaps this page closes. Link to `/tutorials/embeddings/`.

**2 · Chunking** *(interactive)*. Sliders for chunk size and overlap over a real document,
with a fixed question below. Demonstrate all three failure modes: chunks too large dilute
the answer with noise; too small split the answer across a boundary; overlap rescues the
boundary case. Least flashy section, highest-value lesson.

**3 · What a vector store actually holds.** Short. Vectors + metadata + an index. Scala 3
snippet of the corpus type and the ingest loop.

**4 · Brute force, and when it stops working.** Linear scan cost against corpus size, then
the ANN idea and the recall-vs-latency trade. Keep compact. State honestly that under
roughly 100k documents you do not need ANN.

**5 · Hybrid search** *(interactive — THE CENTERPIECE)*. Three result columns side by
side: BM25 keyword, vector, and Reciprocal Rank Fusion. Preset queries chosen so each
strategy visibly wins one and loses another:

| Query | Expected behaviour |
|---|---|
| `M8100` | keyword nails it; vector returns vaguely-bike-shaped noise |
| `my bike has a puncture` | vector finds "how to fix a flat tire"; keyword returns nothing |
| `brakes squeal when wet` | both mediocre; fusion surfaces the right document |

Show per-document RRF scores so the fusion is not a black box. Implement BM25 properly
(k1 ≈ 1.2, b ≈ 0.75) — do not substitute naive term counting. **Verify the three queries
actually behave as tabled once real vectors are in; if one does not, change the query or
the corpus until it does, and say so in the report.**

**6 · Reranking.** Bi-encoder retrieves 50 cheaply, cross-encoder reorders precisely. Show
the ordering change and the cost argument. Static or lightly interactive.

**7 · RAG, demystified.** Retrieve → assemble → generate, showing the actual prompt string
being built from retrieved chunks. The point: RAG is retrieval plus string concatenation,
and all the difficulty lives in sections 2–6.

**8 · What to actually build.** Decision guide by corpus size.

## Accessibility & style constraints

- Colour must never be the only encoding; label everything.
- **Do not use `--accent` (#7aa2f7) and `--accent-2` (#bb9af7) as a categorical pair** —
  measured ΔE 0.3 under protanopia and 8.7 for normal vision, i.e. indistinguishable.
  For the three hybrid-search columns pick hues that are pairwise distinct; blue #7aa2f7 +
  amber #e0af68 is a verified-safe pair, and a third needs checking against both.
- No horizontal page scroll at 375 px. Wide content scrolls inside its own container.
- Honour `prefers-reduced-motion`.

## Verification (all required before reporting done)

1. Serve with `python3 -m http.server 8753` and load
   `http://localhost:8753/tutorials/semantic-search-from-scratch/`.
2. Zero console errors.
3. Drive **every** control in sections 2, 5 and 6 and confirm output changes sensibly.
4. Confirm the three preset queries in section 5 produce the tabled outcomes, using the
   real baked vectors. Report the actual top-3 for each.
5. No horizontal overflow at 375 px and 1440 px.
6. Re-test with `cdn.jsdelivr.net` blocked: no page errors, all interactives still work.
7. Confirm `_data/tutorials.yml` still parses
   (`ruby -ryaml -e 'YAML.load_file("_data/tutorials.yml", permitted_classes: [Date])'`).

Report what you actually ran and what it output. Do not claim a check passed without
having run it.

## Publishing

Do **not** commit, push, open a PR, or merge. Leave the changes in the working tree for
review.
