---
title: dotworld user documentation
date: 2026-08-22
tags: [index, docs, guide]
---

# dotworld user documentation

For people and agents *using* dotworld. (Design rationale for contributors is
[`../architecture.md`](../architecture.md); writing a plugin is [`../plugins.md`](../plugins.md).)

**Registering a source that already lives in a shared or public remote?** dotworld is local-by-default
on the first registration: a pre-push hook keeps `.dotworld/` off `git push` without you asking for it,
and a plain `git push` keeps working on every push after the first. See
[getting-started.md §1](getting-started.md#1-point-dotworld-at-a-repo) before you register.

Start here:

1. **[getting-started.md](getting-started.md)** — register a source, set the push policy, build the
   index, first search.
2. **[concepts.md](concepts.md)** — files vs dots, why there is no database, sources and workspaces,
   and where every byte lives.
3. **[indexing.md](indexing.md)** — `sync_source`, the two link syntaxes, index files as reusable
   names for a set of files, and stranded dots.
4. **[search.md](search.md)** — the three kinds of search, setting up fastembed, and the
   `maxTokens` limit that surprises people.
5. **[plugins.md](plugins.md)** — where `dotworld.config.json` is found, what goes in it, and how
   secrets are referenced.
6. **[commands.md](commands.md)** — **generated.** Every command, every parameter with its
   description, and a real captured session.

## About commands.md

It is not written by hand and must not be edited. The descriptions come from the same `meta.params`
that generates `dot --help` and the MCP tool schema, and every example on the page was **executed**
at build time against a throwaway four-file repository — what you see under `$ dot …` is what the
CLI actually printed, including the refusals.

```console
$ npm run docs          # rebuild it
$ npm run docs:check    # fail if the committed file is out of date
```

`test/e2e/user-docs-cannot-drift.test.ts` fails if a command or parameter is added, renamed or
removed without the page following, in either direction, and it checks that every example invocation
passes only params the command declares — a stale example is a line that fails when you paste it.

**What that test does not cover, honestly:** it reads the committed page, so it catches drift in the
command set but not a change in the *shape of a result*. If you change what a command returns, the
captured outputs are stale until someone runs `npm run docs`. `npm run docs:check` is the guard, and
it is deliberately not in CI — the build needs the real fastembed weights, and CI declines the model
pull by policy.

Fields shown as `"<varies>"` are redacted, not invented: build stamps, timestamps and the
time-derived staleness scores cannot be captured reproducibly, and printing a plausible-looking
number would be a value that is true for nobody.

## Not covered here

MCP. dotworld serves the same command set over MCP and HTTP, but these docs are CLI-only for now.
