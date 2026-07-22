# assets/themes/

Vendored [mdcms](https://github.com/kbenestad/mdcms) theme files. Each one is
a plain copy of a file from mdcms's `themes/` folder — nothing is generated
or transformed, so you can diff a file here against its mdcms source
directly.

## Using a theme

1. Browse mdcms's [themes/ folder](https://github.com/kbenestad/mdcms/tree/main/themes)
   (or preview one live first in [bizdocs](https://github.com/kbenestad/bizdocs)'s
   `themeselector/` app — its Category/Theme dropdowns list the same files)
   and pick one.
2. Download the raw `.yaml` file and drop a copy in here.
3. Point `config.yml`'s `theme:` key at it — `theme: assets/themes/<filename>.yaml`.

`nord.yaml` is included as a working example — set `config.yml`'s `theme:`
to `assets/themes/nord.yaml` to try it, or leave `theme:` blank (the
default) for basis's own look.

## Updating a vendored theme

mdcms occasionally revises a theme file. Re-download and overwrite the copy
here the same way; there's no version pin or hash to keep in sync (theme
files aren't code, unlike style.css/ui.css/app.js — see `sync.sh`).

## Applying it

`assets/app.js`'s `applyMdcmsTheme()` fetches the file named in `theme:` at
boot (same-origin, no new dependency) and maps its `palette`/`colours-
semantic`/`font-body`/`font-heading` onto basis's own design tokens — see
CLAUDE.md's "Suite-wide theming" section for the full mapping and its
rationale. This logic lives in the shared `assets/app.js` that `sync.sh`
keeps byte-identical to bizdocs's — a theme file itself, however, is **not**
synced automatically (it's app-specific content, not one of the three shared
files); copy a `.yaml` file here by hand the same way you picked it.
