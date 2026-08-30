---
title: How dotworld is put together
date: 2026-08-22
tags: [guide, concepts, architecture]
---

# How dotworld is put together

Read this once and the rest of the docs stop being surprising. Five ideas.

## 1. A file is the content; its dot is what dotworld knows about it

Your file is whatever you wrote, byte for byte. dotworld never edits it, and never puts metadata in
its frontmatter.

Everything dotworld knows *about* that file lives beside it in a **dot** — a `.dot.md` under
`.dotworld/dots/`, mirroring the file's path:

```
notes/retention.md                              <- your file, untouched
.dotworld/dots/notes/retention.md.dot.md        <- its dot
```

A dot holds a title, tags, properties, a generated summary, and an authorable body of your own. It
is a real file, durable, and it belongs in git — commit dots in the same commit as the files they
describe.

You never address a dot by its own path. Identity is the *file's* path, which is why every command
says `files=notes/retention.md` and never `files=.dotworld/dots/...`. The one exception is
`dot_attach`, and even there you name the file the stranded dot *describes*.

## 2. Files are the only source of truth

There is no database. Source files and dots are durable and in git; everything else — embeddings,
the link graph, git's blob SHA per path — is a **disposable index** under `.dotworld/derived/`, rebuilt from
the files. You can `rm -rf .dotworld/derived/` any time and lose nothing. A cold start reconstructs
it.

Two consequences worth internalising:

- **A dot exists once there is something to record.** `sync_source` mints none, and neither does
  adding a file by hand: an undotted file is indexed, searchable and readable all the same, and its
  dot is born on the first write that records something. See [indexing.md](indexing.md).
- **Reads never write.** `file_read`, `search_semantic`, `dot_get`, `file_tree` and `graph_get`
  write nothing to your repository, ever — not even a blank dot.

## 3. One command set, three faces

`dot <command> key=value` is the CLI. The same commands are served over HTTP and MCP from the same
registry, so nothing can drift between them: the CLI's `--help`, the MCP tool schema and
[commands.md](commands.md) are all projections of one definition in the code. **The alpha tarball
ships the CLI only** — the HTTP and MCP faces are in the codebase but deliberately not in the
install.

Underneath, every command answers the same envelope:

```json
{ "ok": true,  "result": … }
{ "ok": false, "error": "…", "code": "BAD_PARAMS" }
```

Over the CLI that envelope is what **`--json`** asks for. Without it, the commands that have a
plain-text rendering — `file_tree`, `file_read`, `search_grep`, `search_semantic`, `dot_list`,
`dot_get` — answer in lines you can read and pipe, and a failure goes to stderr as a `dot: ` line
instead of onto stdout. Every other command still prints the envelope, so `--json` is the reliable
form in a script. `--json '{"sourceId":"kb"}'` passes params as an object; bare `--json` asks for the
envelope back.

Advisory notes — which working tree answered, what got inferred — go to **stderr**, so piping stdout
into `jq` stays clean.

Parameters are a **closed set**. A key no command declares is refused by name rather than silently
dropped, because a dropped param is indistinguishable from an omitted one, and most params have
defaults — you would get `ok: true` and an answer computed from a default you did not choose.

## 4. A source is a repo; a workspace is your worktree of it

A **source** is one git repository dotworld knows about, registered under an id
(`source_register`). The registry is machine state, outside every repo, at
`$XDG_CONFIG_HOME/dotworld/config.json` — a source root must stay archivable without carrying a map
of your machine.

A **workspace** is your own git worktree on your own branch (`workspace_open`), so two people can
work on one source without editing the same checkout. `workspace_submit` merges your branch into
the canonical checkout.

**Conflicts are git's, not dotworld's.** A divergence is reported the way git reports one — status
plus the conflicted *paths*. dotworld never hands you both sides of a field to choose between; you
settle it in your own workspace with `file_stage`. Worked end to end in
[commands.md](commands.md#workspace_submit).

## 5. Layers: kernel, plugin, satellite

The **kernel** is the always-on core. It holds no LLM and no credentials — it *is* the auth waist.

A **plugin** is local, detachable, and may hold a model or secrets; the kernel degrades gracefully
without it. Summarising, obsolescence review, cross-workspace comms and context loading are plugins.
See [plugins.md](plugins.md).

A **satellite** is a remote, API-only consumer.

## Where every byte lives

| What | Where | Durable? |
|---|---|---|
| Your files | your repo | yes — git |
| Dots | `<repo>/.dotworld/dots/` | yes — git |
| Property vocabulary | `<repo>/.dotworld/properties.json` | yes — git |
| Authored connectors | `<repo>/.dotworld/connectors/` | yes — git |
| Derived index | `<tree>/.dotworld/derived/` | **no** — `rm -rf` freely |
| Source registry | `$XDG_CONFIG_HOME/dotworld/config.json` | machine state |
| Model weights | `$XDG_CACHE_HOME/dotworld/models/` | machine state |
| Managed worktrees | `$XDG_STATE_HOME/dotworld/worktrees/` | machine state |

The design rationale — including the options decided against — is in
[`../architecture.md`](../architecture.md).
