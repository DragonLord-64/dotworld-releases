# dotworld (alpha releases)

Install artifacts (`install.sh`, release tarballs) published as
[GitHub Releases](https://github.com/DragonLord-64/dotworld-releases/releases), plus the
[user docs](docs/user-docs) and [presentation](docs/presentation) kept in sync with each release.
The source lives in a private repo — this one exists so the install command works without a login,
so the docs are readable without one either, and so alpha testers have somewhere public to file bugs.

## Docs

- [`docs/user-docs`](docs/user-docs) — getting started, concepts, indexing, search, plugins, and the
  full generated command reference.
- [`docs/presentation`](docs/presentation) — the slide deck (`dotworld-presentation.html`); open it
  in a browser, no build step.

Both are synced from the source repo on every release and also ship as `dotworld-docs-*.tar.gz` on
the [release page](https://github.com/DragonLord-64/dotworld-releases/releases).

## Install

```bash
curl -fsSL https://github.com/DragonLord-64/dotworld-releases/releases/latest/download/install.sh | bash
```

Needs `node >= 22` and `git`. The script explains itself — run it with no args to see what it does,
or read [`install.sh`](https://github.com/DragonLord-64/dotworld-releases/releases/latest/download/install.sh)
before piping it into a shell, which is always a reasonable thing to want to do.

To install a specific version instead of latest, or from a file you already downloaded:

```bash
./install.sh https://github.com/DragonLord-64/dotworld-releases/releases/download/v0.1.0-alpha.1/dotworld-0.1.0-alpha.1.tgz
./install.sh path/to/dotworld-0.1.0-alpha.1.tgz
```

## Bugs

Please [open an issue](https://github.com/DragonLord-64/dotworld-releases/issues) here — include
`dot version` output and what you ran. This repo's issue tracker is the bug-report channel; the
source repo is private.
