---
title: Getting started
date: 2026-08-22
tags: [guide, setup, source, publish-policy]
---

# Getting started

From nothing to a searchable source. Every command here has its full parameter list in
[commands.md](commands.md).

## 1. Point dotworld at a repo

One source is one git repository.

```console
$ cd ~/handbook
$ dot source_register id=handbook root=.
```

`root=.` is resolved against your working directory. If the directory is not a git repository yet,
pass `initGit=true` and dotworld will make it one.

**A repo dotworld has just met is pushable by default.** With `neverPushDotworld` omitted, nothing
changes about how `git push` behaves — `.dotworld/` pushes like any other tracked file. Opt in
explicitly if you want it kept off a remote:

```console
$ dot source_register id=handbook root=. neverPushDotworld=true
```

Check what you got — the result tells you whether the hook actually landed:

```json
{ "ok": true, "result": { "id": "handbook", "root": "/home/you/handbook",
  "publishPolicy": { "neverPushDotworld": true, "prePushHook": { "installed": true } } } }
```

Four things worth knowing about the policy:

- **A plain `git push` keeps working after the first push, too.** The hook's own filtered push
  rewrites the remote to a history your next local commit can't fast-forward from, so an unforced
  `git push` would otherwise be rejected before the hook even runs, every time after the first.
  dotworld heads this off itself: it scopes a forced refspec to the branch it just pushed and sets
  upstream tracking, since git's own `--set-upstream` never lands here (the raw push it would ride
  on is always the one this hook aborts). If you see the rejection anyway, just `git push` again —
  do **not** `git pull` to clear it, that merges the filtered history (the copy with `.dotworld/`
  stripped out) back into your own branch.
- **The claim is never written unless it is enforced.** The hook is installed *first*;
  `publish-policy.json` is only written if it landed. That file is committed and travels with the
  repo, while the hook is per-clone and can be refused — so writing the promise without the
  enforcement would leave repos carrying a guarantee that was false in the tree you cloned into.
- **An explicit `true` that cannot be installed is refused**, with the installer's reason, rather
  than recorded as a claim nothing enforces.
- **The policy does not survive a protected push.** It lives under `.dotworld/`, so the filter
  strips it too. A fresh clone re-installs the hook from the committed file the next time you
  register the source.

Turning the policy off (`source_update id=handbook neverPushDotworld=false`) needs no hook and
always succeeds.

## 2. Build the index

```console
$ dot sync_source sourceId=handbook
{ "ok": true, "result": { "files": 4, "connectors": 1 } }
```

This mints a dot for every file that lacks one and rebuilds the disposable index. **Do this before
you commit**, then stage the new dots alongside the files that produced them — a file added by hand
is invisible to every other clone until its dot is committed. Details and the wikilink format are in
[indexing.md](indexing.md).

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

Semantic search needs the embedding model. If `dot embedding_info` reports the `hashing` engine
rather than `fastembed`, results are much worse than the config promises — see
[search.md](search.md) to fix that.

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
