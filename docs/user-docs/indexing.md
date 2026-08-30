---
title: Indexing, links and index files
date: 2026-08-22
tags: [guide, index, wikilinks, graph]
---

# Indexing, links and index files

## Building the index

```console
$ dot sync_source sourceId=handbook
{"ok":true,"result":{"files":4,"connectors":1}}
```

`sync_source` walks the files git would track and rebuilds the derived index — embeddings, the
link graph, and the blob SHA git already reported for each path — under `.dotworld/derived/`. It
also heals a dot an older version wrote wrong: a missing `kind`, a title minted from a code comment.

Three rules follow from that:

- **A dot exists once there is something to record.** `sync_source` mints none. A file with no dot
  is fully indexed, searchable and readable all the same — a blank dot would assert exactly what its
  absence already says. The dot is born on the first write that records something: a tag, a
  property, a summary, a title, a body. The one exception is `file_create`, which authors a file and
  its dot in a single act and so writes a dot even when you pass it no metadata.
- **Reads mint nothing either.** `search_semantic`, `file_read`, `dot_get`, `file_tree` and
  `graph_get` never write to your repository.
- **Commit a dot with the file it describes.** `.dotworld/dots/` is durable and in git, so once you
  have written metadata, a dot left uncommitted is metadata no other clone has — and nothing
  reconstructs it, because there is nothing to reconstruct it from.
- **`.dotworld/derived/` is disposable.** `rm -rf` it whenever you like; the next sync rebuilds it.

`file_create` writes the file and its dot in the same act, so metadata you pass it needs no
follow-up sync.

### Why a file is not indexed

dotworld indexes what git would track, so the usual answer is that something ignores it. `sync_doctor`
asks git directly and reports the ignore file, the line and the pattern that excluded the path —
which turns "not indexed" from a dead end into a repair.

## Links

The default extractor reads **both** link syntaxes out of file bodies, in document order:

| Syntax | Written | Resolved by |
|---|---|---|
| Wikilink | `[[notes/retention.md]]` | basename or title |
| Markdown | `[the policy](retention.md)` | directory-relative to the linking file |

Wikilinks resolve loosely — `[[retention]]`, `[[retention.md]]` and `[[Data retention policy]]` can
all reach the same file — because they match on basename or title. Markdown links resolve like
paths, relative to the file containing them. Both become edges in the same graph, and resolution
happens fresh against the current file set on every index, so a link to a file that does not exist
yet simply has no edge until it does.

You can also author edges explicitly, which is how you record a relationship the prose does not
state:

```console
$ dot graph_connect from=notes/retention.md to=decisions/2026-02-adopt-postgres.md type=supersedes
```

Authored connectors live in `.dotworld/connectors/` — durable, in git. `supersedes` is a provenance
edge, which is what `graph_provenance` reads to answer truth-history questions.

Read the structure back with `graph_get` (connectors around some seed files) or `graph_affected`
(impact — the embedding neighbourhood unioned with the graph).

## Index files: a name for a set of files

An **index** is an ordinary markdown document whose body is a list of links. `index_create` writes
one:

```console
$ dot index_create path=notes/retention-pack.md \
    files='["notes/retention.md","notes/deletion-runbook.md"]' title="Retention pack"
```

This matters because **every command that takes `files` also takes `index`**. The index file becomes
a reusable, committed, editable name for that set:

```console
$ dot file_read index=notes/retention-pack.md
$ dot context_load index=notes/retention-pack.md mode=dot
$ dot summary_refresh index=notes/retention-pack.md
```

The order you wrote is the order you get back — an index whose ordering was arbitrary would not be
an index. Because it is just a markdown file with links in it, you can edit it by hand, and the
links it contains are graph edges like any others.

## Renames

Rename reconciliation runs on sync and asks git what happened, so an ordinary `git mv` keeps the
dot attached to the file.

When git has no rename to detect — the file was deleted in one commit and recreated under a new name
in another — the dot is left **stranded**. Nothing is lost; find them and reattach by hand:

```console
$ dot dot_list orphaned=true
$ dot dot_attach dot=notes/deletion-runbook.md file=notes/purge-runbook.md
```

`dot` there is the path the stranded dot *describes*, not the `.dot.md` path.
