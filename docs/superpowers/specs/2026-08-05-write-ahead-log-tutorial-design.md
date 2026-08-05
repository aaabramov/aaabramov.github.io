# How databases survive a power cut — WAL tutorial

**Date:** 2026-08-05
**Slug:** `tutorials/write-ahead-log/`
**Audience:** junior-to-mid software engineers
**Status:** approved, ready to build

## Goal

Teach the write-ahead log from first principles by building one mini key-value store and
crashing it repeatedly. The reader should leave able to explain, unprompted, why
"append the intent, fsync it, *then* touch the data" is the move — and recognise the same
pattern in Postgres, Kafka and Raft.

Explicit non-goal: Postgres internals (LSN formats, segment files, PITR). This is the
pattern, not one vendor's implementation.

## Shape

One self-contained page at `tutorials/write-ahead-log/index.html`, cloned from
`tutorials/embeddings/index.html`'s dark theme, CSS custom properties, `.kb-back` nav,
numbered `<section>` scaffold and footer. Registered in `_data/tutorials.yml`.

### Dependencies

Pinned ESM from jsdelivr, matching the pinning style of `three@0.185.1` and
`@xenova/transformers@2.17.2` in the sibling tutorials:

- **Prism.js** — syntax highlighting for the code snippets.
- **anime.js v4** — choreographs the two multi-element sequences only: the crash freeze +
  recovery replay (§4) and the torn-write byte scrub (§5).

Both degrade gracefully if the CDN is unreachable: code stays readable unhighlighted,
animations snap instead of tween. Everything else — log records, byte views, the commit
timeline — is hand-rolled SVG and CSS. No chart library; no D3.

## The shared model

Every demo on the page drives one object, so the reader learns one system:

```
mem       Map<key,value>        volatile; lost on crash
wal       [{lsn, op, key, value, crc, durable}]   append-only
dataFile  {Map<key,value>, lastAppliedLsn}        survives a crash
```

Write path in WAL mode, as four observable micro-steps per operation:

1. `append` — record enters the WAL, `durable: false` (it's in the OS page cache)
2. `fsync` — every pending record flips to `durable: true`
3. `apply` — `mem` updates
4. `ack` — client is told "committed"

Crash semantics: `mem` is discarded; WAL records with `durable: false` are discarded;
`dataFile` persists. Recovery loads `dataFile` and replays durable records with
`lsn > dataFile.lastAppliedLsn`.

Three durability modes, used side by side in §4:

| Mode | Behaviour |
|---|---|
| **no WAL** | mutate `dataFile` in place, byte by byte; a crash mid-write leaves a torn value |
| **WAL, no fsync** | append only; records become durable on a lazy writeback tick, not on commit |
| **WAL + fsync** | append then fsync before ack |

## Sections

**1 · The power cut.** `SET balance = 400` as a 3-byte in-place overwrite. Animated
diagram: the crash lands mid-overwrite and the file holds neither 500 nor 400. Names the
two enemies — **lost** writes and **torn** writes.

**2 · The rule.** WAL in one sentence: append your intent and fsync it *before* you touch
the real data. Prism-highlighted pseudo-code of the write path with the ordering
constraint annotated. Plus the constraint that makes recovery legal: **replay must be
idempotent** — `SET k = v` is safe to run twice, `balance += 100` is not.

**3 · Step-through viewer** *(interactive)*. Scrubber across ~24 micro-steps (6 operations
× 4 steps, with a checkpoint partway so the data file visibly moves). Three synced panes —
WAL, memory, data file — plus a caption naming the current step. The reader watches the
log run ahead of the data file; that gap is the durability guarantee. Read-only.

**4 · Crash simulator** *(interactive, the centerpiece)*. The same operation stream runs
in three columns, one per durability mode. A CRASH button fires at the current tick; a
slider picks the tick; a "random crash" button re-rolls. Then **reboot** → each column
runs recovery → each reports a verdict badge:

- no WAL → `user:1 = 4??` — **corrupted**
- WAL, no fsync → **silently lost N acked writes**
- WAL + fsync → **consistent, 0 lost**

Re-runnable at any crash point. The teaching point is that only the third column is boring
every single time.

**5 · Why fsync, and why every record carries a checksum** *(interactive)*. Two small
toys. (a) The write path app → `write()` → page cache → `fsync` → platter, with a crash
fireable at each stage; `write()` returning success is not durability, which is exactly
what column two of §4 was demonstrating. (b) A single record laid out as
`[len][crc][key][value]` bytes; drag the crash point *inside* it and watch the CRC fail.
Recovery stops at the last valid record and discards the tail — a half-record is not a
half-truth, it is not a record.

**6 · Group commit** *(interactive)*. Slider for concurrent committers (1–32). Two SVG
timelines: one fsync per commit (sequential, ~0.8 ms each) versus one fsync per batch.
Throughput readout in ops/sec for both, and the honest note that batching trades a little
latency for a lot of throughput. Numbers are labelled illustrative, not benchmarked.

**7 · Where you've already seen this.** Cards mapping the pattern onto Postgres WAL,
SQLite WAL mode, Kafka's partition log, Raft's replicated log, ext4 journaling, and the
LSM memtable+WAL pair. Then the checkpointing aside — the log cannot grow forever, so
flush state and truncate the replayed prefix — and a two-line recap.

Checkpointing is deliberately one paragraph rather than its own interactive section; it is
needed to answer "how far back does replay start?" and no further.

## Verification

Preview with `python3 -m http.server 8753` → `http://localhost:8753/tutorials/write-ahead-log/`.
Drive every control in §3–§6 in a real browser, check the console is clean, check the
layout at 375 px and 1440 px, and confirm the page still works with the CDN blocked.

## Publishing

Branch `add-write-ahead-log-tutorial` → commit `tutorials/write-ahead-log/index.html` and
`_data/tutorials.yml` only → push → PR via `gh` → merge to `master`.
