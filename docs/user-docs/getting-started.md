---
title: Getting started
date: 2026-08-22
tags: [guide, setup, source]
---

# Getting started

From nothing to a searchable source. Every command here has its full parameter list in
[commands.md](commands.md).

**Platforms.** `install.sh` is bash, so it covers Linux and macOS; on Windows you install the
tarball with `npm install -g` and seed the machine config by hand. The suite has a CI job for all
three, but CI is `workflow_dispatch`-only right now, so read that as "written to run there", not as
"every commit is verified on all three". The platform notes worth knowing before you file a bug are
in the [repository README](../../README.md#platforms).

## 1. Point dotworld at a repo

One source is one git repository.

```console
$ cd ~/handbook
$ dot source_register id=handbook root=.
```

`root=.` is resolved against your working directory. If the directory is not a git repository yet,
pass `initGit=true` and dotworld will make it one. Pass `writeAgentsDoc=true` to append a short
dotworld orientation block to the source's `AGENTS.md`, creating the file if it has none — opt-in,
because it writes to your repository.

## 2. Build the index

```console
$ dot sync_source sourceId=handbook
{"ok":true,"result":{"files":4,"connectors":1}}
```

This rebuilds the disposable index and heals a dot an older version wrote wrong. It **mints
nothing**: a file with no dot is still fully indexed, searchable and readable, and a dot comes into
being on the first write that has something to record about the file. Run it after adding or
renaming files, and when you do write metadata, stage the dot alongside the file it describes.
Details and the wikilink format are in [indexing.md](indexing.md).

## 3. Check the install is not half-working

```console
$ dot sync_doctor
```

One call, every condition that leaves an install silently degraded — a missing model, a stale build,
a registry pointing at a directory that moved. `repair=true` fixes what can be fixed automatically.

```console
$ dot version        # which build is answering, and which search engine it is running
$ dot plugin_list    # what attached, and what was skipped at boot, with reasons
```

## 4. Search it

```console
$ dot search_semantic query="how long do we keep customer logs" limit=3
```

Semantic search needs the embedding model, and there is no fallback: if `dot embedding_info`
reports `quality: "unavailable"`, `search_semantic` refuses rather than ranking on something
weaker — see [search.md](search.md) to fix that.

## 5. Working with other people

```console
$ dot workspace_open as=you     # your own worktree, on your own branch
$ dot workspace_list            # every working tree of this source
$ dot workspace_submit message="…"   # merge your branch into the canonical checkout
```

`workspace_submit` never commits on your behalf and never picks a side in a conflict. The full
worked sequence — including the rebase, `file_stage`, and the submit that finally lands — is in
[commands.md](commands.md#workspace_submit).

## Where to next

- [concepts.md](concepts.md) — what a dot is, and why there is no database
- [indexing.md](indexing.md) — indexing, wikilinks, indexes as reusable file sets
- [search.md](search.md) — fastembed setup and the three kinds of search
- [plugins.md](plugins.md) — configuring what is attached
- [commands.md](commands.md) — every command, every parameter, real captured output
