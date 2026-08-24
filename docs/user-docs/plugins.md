---
title: Configuring plugins
date: 2026-08-22
tags: [guide, plugins, config]
---

# Configuring plugins

This is how to *configure* what is attached. Writing a new plugin is
[`../plugins.md`](../plugins.md).

## Where the config comes from

`dotworld.config.json` declares the whole composition — embedding engine, plugins, file kinds and
declared sources. dotworld looks for it in this order, and **stops at the first hit**:

1. `$DOTWORLD_PLUGIN_CONFIG`, used exactly as given. A path that does not exist is a real, surfaced
   error, not a silent fallthrough.
2. `dotworld.config.json` in the machine config dir (`$XDG_CONFIG_HOME/dotworld/`).
3. A repository's own `.dotworld/dotworld.config.json`, found by walking **up** from your working
   directory the way git finds its repository — but never past the git toplevel.

That last bound is deliberate: a stray config in `$HOME` must never govern an unrelated project
underneath it. Outside a git repository there is no toplevel to bound the walk, so only your exact
working directory is checked.

**Config discovery is sensitive to where you are standing.** If plugins or the embedding engine seem
to have vanished, ask what dotworld actually found:

```console
$ dot embedding_info     # `configFile` names the file in effect, `note` says so in prose
$ dot plugin_list        # what attached, and what was skipped at boot — with reasons
```

## The shape

```json
{
  "version": "1",
  "embedding": {
    "engine": "fastembed",
    "settings": { "model": "all-MiniLM-L6-v2" }
  },
  "kinds": [
    { "pattern": "\\.(md|markdown)$", "kind": "prose" },
    { "pattern": "\\.(ts|js|py|go|rs)$", "kind": "code" }
  ],
  "sources": [
    { "id": "handbook", "root": "." }
  ],
  "plugins": [
    { "id": "obsolescence", "engine": "rule-based", "settings": { "rules": [] } },
    { "id": "summarizer", "engine": "claude-code",
      "settings": { "model": "haiku", "maxBodyChars": 12000, "timeoutMs": 60000 } },
    { "id": "comms", "engine": "fs-log" },
    { "id": "context-loader", "engine": "compose" }
  ]
}
```

- **`plugins[].engine`** picks the implementation behind the plugin's seam. Omit it for the
  manifest's default. An unknown name is skipped with a reason rather than crashing the face.
- **`kinds`** maps filename patterns to a `kind` property on the dot.
- **`sources`** declares sources that are registered at boot, so a repo named here is live the moment
  a face connects. An already-registered source keeps its root — only an explicit
  `source_register`/`source_update` moves it.

## Secrets are named, never inlined

A plugin that needs credentials takes them by **reference** to an environment variable:

```json
{ "id": "summarizer", "engine": "some-api", "credsEnv": { "apiKey": "SUMMARIZER_API_KEY" } }
```

`credsEnv` maps the name the plugin asks for to the env var it is read from. Never put a secret in
this file — it is committed.

## Attachment is resilient by design

An unknown plugin id, an incompatible host, a bad engine name or a missing credential **skips that
plugin** and boots without it. The kernel is model-free and always works; plugins are the optional
layer. `plugin_list` reports both what attached and what was skipped, with the reason for each — so
a missing capability is diagnosable rather than mysterious.

## What the shipped plugins give you

| Plugin | Adds |
|---|---|
| `summarizer` | `summary_refresh` — rewrites summaries that no longer describe their file |
| `obsolescence` | `obsolescence_review` — which notes does this decision put in doubt |
| `comms` | `comms_send` / `comms_read` — cross-workspace messaging on a source-wide log |
| `context-loader` | `context_load` — a `files`/`index` list as one prompt-ready blob |

Detach any of them and the kernel keeps working; the command simply is not there.

## A/B a composition

Point `DOTWORLD_PLUGIN_CONFIG` at a second file to compare compositions without editing the first:

```console
$ DOTWORLD_PLUGIN_CONFIG=.dotworld/dotworld.offline.config.json dot plugin_list
```
