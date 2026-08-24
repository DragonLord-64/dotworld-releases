---
title: presentations
date: 2026-08-21
tags: [index, presentations]
---

# presentations

Slide decks about dotworld, for showing rather than reading. Open the `.html` in a browser —
no build step, no assets, no network.

- [`dotworld-presentation.html`](dotworld-presentation.html) — six foundation sections (where
  dotworld lives in a repo, the architecture in brief, the dot and its properties, the kernel's
  features, the four plugins, and the three faces), then seven use cases of four slides each:
  the problem, the initial state, the commands run, and what changed. An appendix closes with the
  full architecture diagram. Navigate with `→`/`←`, `1`–`9` to jump, `0` or `Esc` for the agenda
  (every section is clickable there).

**Generated, not authored.** `dotworld-presentation.html` is assembled from the per-section
fragments in [`parts/`](parts/) by `npm run deck`. Edit a fragment, not the deck — an edit to the
assembled file is lost on the next rebuild. `parts/SPEC.md` is the slide-writing brief the use-case
fragments follow (`parts/architecture-reference.html` is adapted straight from
[`../architecture-diagram.html`](../architecture-diagram.html) instead, scoped so its SVG classes
don't collide with the shared shell styles). `node scripts/deck-autofit.mjs` reports (or, with
`--apply`, fixes) monospace panels too wide for a 1440px window.

**Viewing over SSH.** If you're SSH'd in with no local browser to point at a `file://` path,
`scripts/serve-presentations.sh` serves this directory over HTTP so you can open it from VS Code's
Simple Browser or a forwarded port in your local browser instead.

**Where the evidence comes from.** The use cases are `emberfall`, a sample C++ game repo with
seven demo branches, plus `emberfall-forge` for the cross-repo one. Neither is checked in here.
The foundation sections show this repo instead — its own dots, its own `dotworld.config.json`,
and real `workspace_list`/`comms_read` output from the agents working on it. Either way the deck is
a record of runs, so its command output is a real transcript from the date it was built, not a live
invocation of the current build — a command surface that changes afterwards will drift from what
the slides show.
