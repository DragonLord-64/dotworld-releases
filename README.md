# dotworld (alpha releases)

This repo holds nothing but install artifacts: `install.sh` and release tarballs, published as
[GitHub Releases](https://github.com/DragonLord-64/dotworld-releases/releases). The source lives in
a private repo — this one exists so the install command works without a login and so alpha testers
have somewhere public to file bugs.

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
