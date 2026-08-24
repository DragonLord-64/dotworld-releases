---
title: Semantic search and fastembed
date: 2026-08-22
tags: [guide, search, embedding, fastembed]
---

# Semantic search and fastembed

## Three kinds of search, and when each is right

| Command | Answers | Use it when |
|---|---|---|
| `search_semantic` | ranked candidate files for a task | you know the *idea*, not the words |
| `search_grep` | exact string / identifier matches | you know the exact token |
| `search_properties` | files by dot metadata | "which files does alice own" |

`search_semantic` returns *candidates*, not answers — `score` is cosine similarity after a lexical
rerank of the top of the ranking. Read the ones that look right with `file_read`.

Two fields on every hit are worth checking before you read anything: `obsolete` and `superseded`. A
superseded file can still rank above its replacement, and this is the only reliable way to tell.

## Setting up fastembed

Semantic search only works as advertised with a real embedding model. Without it dotworld falls back
to a built-in `hashing` embedder, which is much worse — and the failure is quiet, which is the
problem.

**Check first:**

```console
$ dot embedding_info
```

`"engine": "fastembed"` with `"quality": "ok"` means you are set. `"engine": "hashing"` means you are
not, and `quality` will say which flavour of not:

| `quality` | What happened |
|---|---|
| `ok` | the configured engine is live |
| `degraded-no-config` | no `dotworld.config.json` was found from this directory |
| `degraded-…` with `reason: "construct-failed"` | the package or the weights are missing |
| `degraded-…` with `reason: "unknown-engine"` | a bad engine name, or a stale build |
| `reason: "auto-fallback"` | `engine: "auto"` looked for a real engine and found none |

**Pull the weights.** This is the only time dotworld reaches the network for embeddings; after it
runs, the model loads fully offline and query time makes no outbound request.

```console
$ npm run pull-model
```

It also runs from `postinstall` and from the startup script, in a mode that may decline and can never
fail an install. Opt out with `DOTWORLD_PULL_MODEL=0`; force it under CI with `DOTWORLD_PULL_MODEL=1`.

The weights are **machine state** — one copy at `$XDG_CACHE_HOME/dotworld/models` (else
`~/.cache/dotworld/models`) serves every source and every worktree. Override with
`DOTWORLD_MODEL_CACHE`, and pick a different fastembed model with `DOTWORLD_MODEL`.

**Configure the engine** in `dotworld.config.json`:

```json
{
  "embedding": {
    "engine": "fastembed",
    "settings": { "model": "all-MiniLM-L6-v2" }
  }
}
```

Then re-index, because embeddings are part of the derived index:

```console
$ dot sync_source sourceId=handbook
```

## `fastembed` or `auto` — the difference is what happens when it breaks

- **`engine: "fastembed"`** is a demand. If it cannot be built, `search_semantic` **refuses** with
  `BAD_PARAMS` rather than silently ranking on the weaker model. Pick this when quiet degradation
  would be worse than an error — which, for anything you are trusting the ranking of, it usually is.
- **`engine: "auto"`** is a preference. It takes a real engine if one is available and falls back to
  `hashing` otherwise, reporting `reason: "auto-fallback"`. Landing on the fallback *obeyed* the
  request, so `honored` stays `true` — check `quality`, not `honored`, to know whether search is
  healthy.
- **`engine: "hashing"`** is the built-in fallback. No download, no dependency, noticeably worse
  ranking. Fine for tests and for exact-match workflows that lean on `search_grep`.

## The limit worth knowing: `maxTokens`

`embedding_info` reports `maxTokens` — for `all-MiniLM-L6-v2`, 512. That is how much of a document
the model actually sees. **A file longer than that is ranked on its opening section only**, and
everything past it is silently unseen.

This is not a bug you can configure away; it is the model. In practice it means a long document
whose relevant part is buried in the middle may not rank, so for exact identifiers prefer
`search_grep`, and when search is degraded, open more candidates than usual.
