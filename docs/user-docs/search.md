---
title: Semantic search and fastembed
date: 2026-08-29
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

Semantic search needs a real embedding model, and **there is no built-in fallback.** An install that
never pulled the weights has no semantic search rather than a worse version of it: `search_semantic`
**refuses** with `BAD_PARAMS`, names `search_grep` as what still works, and gives you the one-time
pull command. An absent capability is honest; one that quietly ranks differently under the same name
is not.

**Check first:**

```console
$ dot embedding_info
```

`quality` is the one field that answers "is search healthy right now", and it has two values:

| `quality` | What it means |
|---|---|
| `ok` | a model is loaded and search ranks |
| `unavailable` | no model is loaded and `search_semantic` refuses |

When search is unavailable, `reason` says why, and `note` says it in prose with the remedy:

| `reason` | What happened |
|---|---|
| `unknown-engine` | the engine name in config is not one this build knows — check the spelling against the `available` list, or the build predates the engine |
| `construct-failed` | the name is recognised but the package or the pulled weights are missing |
| `null` with `requested: null` | no `dotworld.config.json` was found from this directory, so no engine was ever requested |

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

## `fastembed` or `auto` — the difference is only in the diagnosis

- **`engine: "fastembed"`** names the engine you want. If it cannot be built, `reason` is
  `construct-failed` and the message names *that* engine.
- **`engine: "auto"`** means "a real engine if one can be built, **nothing** otherwise". It is not a
  request for a weaker model — there isn't one. If nothing can be built it also reports
  `construct-failed`, phrased about `auto` rather than a named engine.

Either way the outcome when no model loads is identical: `quality: "unavailable"` and a refusal. The
choice only changes which sentence you get back.

## Long files are embedded whole

`embedding_info` reports `maxTokens` — for `all-MiniLM-L6-v2`, 512. That is the ceiling on a single
**pass** through the model, not on how much of a file is seen.

A file longer than the ceiling is split into overlapping chunks, each embedded separately, and a
path's vectors are folded with `MAX` — so a match anywhere in a file ranks the file, including a
passage buried in the middle. Chunk boundaries are never part of the cache key; the whole file's git
blob SHA is, so boundaries are free to move between versions.

A provider that declares no `maxTokens` is not chunked, because there is no ceiling to chunk around
and one vector is the honest representation of it.

Two practical notes survive that: for exact identifiers `search_grep` is still the right tool, and
a hit is a candidate to open, not an answer.
