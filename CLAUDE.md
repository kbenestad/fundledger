# CLAUDE.md

Guidance for building a **standalone kbenestad business-document app** from this
template. For the design system and the rules that keep an app pixel-identical to
the bizdocs family, see [DESIGN.md](DESIGN.md).

## What this is

This repository is **`basis`** — a self-contained starter for a single small web
app that produces a business document (invoice, quote, packing slip, contract,
…) as a PDF, looking and behaving **exactly** like the apps in the
[bizdocs](https://github.com/kbenestad/bizdocs) series, but shipping on its own
(its own repo, its own hosting), with **no build step and no backend**.

The whole app is **one `index.html`** (all HTML/CSS/JS inline) plus a
`config.yml` and a bundled `assets/` folder. You build an app by copying this
template and replacing the marked app-specific parts — never by hand-assembling
the chrome, which is what guarantees the pixel-perfect match.

## Repository layout

```
index.html          the app: chrome + boot + your form + your PDF (all inline)
config.yml          branding + all UI strings (localisation), read at boot
assets/
  style.css         design tokens / colour scheme + reset + page shell   ┐ shared
  ui.css            the kb-* UI component library                        │ design,
  app.js            shared runtime (DOM, theme, modals, i18n, config, …) ┘ DO NOT EDIT
  favicon.svg       placeholder icon — replace with a real set per app
  site.webmanifest
sync.sh             refresh the three shared files from bizdocs (see below)
CLAUDE.md · DESIGN.md · README.md
```

### The shared files are bundled, and must stay byte-identical

Unlike an in-series bizdocs app (which links one shared `../assets/`), this app
**bundles its own copy** of the design + runtime in `assets/` and references it
locally (`assets/style.css`, not `../assets/style.css`). That is what makes the
folder standalone.

The price of self-containment is duplication, so the discipline is strict:
**`assets/style.css`, `assets/ui.css` and `assets/app.js` are byte-identical
copies of bizdocs' shared files. Never hand-edit them.** A bizdocs UI change is
pulled in wholesale, not patched here:

```bash
BIZDOCS_REF=main ./sync.sh --from-github   # pull the three shared files from GitHub
# or, if a bizdocs checkout sits alongside this repo:
./sync.sh ../bizdocs/assets
git diff -- assets                         # review, then commit
```

If you ever need app-specific CSS, it goes in `index.html`'s inline `<style>`;
if you need app-specific copy, it goes through `config.yml` + `S()`. The moment
you edit the shared files directly, drop-in sync stops being clean and the app
drifts from the family. Don't.

### A bug *in* the shared layer is not fixed here either

If something in `style.css`, `ui.css`, or `app.js` is actually broken — not
"this template needs different behaviour" but "this component/rule is wrong
for everyone who uses it" — resist hand-patching it directly in this repo's
`assets/`. **This repo's copy is not the canonical one.** It is downstream of
**[kbenestad/bizdocs](https://github.com/kbenestad/bizdocs)** `assets/`
(bizdocs has its own `development` branch and PR flow, same shape as this
repo's), exactly the same way every app built from this template is
downstream of *this* repo. A hand-patch here fixes nothing at the root, and
the next `sync.sh` run silently overwrites it.

Fix it in bizdocs `main` first. Then, back here on `development`:

```bash
BIZDOCS_REF=main ./sync.sh --from-github   # or ./sync.sh ../bizdocs/assets
git diff -- assets                         # confirm it's the expected shared-file diff, only
```

If the bug was in a component this template's own `index.html` doesn't
exercise (nothing here called it, so the bug never had a live reference to
even reproduce against), add the minimal demo code needed to cover it —
future apps copied from this template should inherit a working, exercised
reference, not just a silently-fixed CSS rule.

## How the app boots

1. A tiny inline pre-paint script in `<head>` reads `localStorage['kb-theme']`
   and sets `data-theme` before first paint (avoids a flash). Keep it.
2. CDN libraries load: `js-yaml` (config) and `pdf-lib` (PDF output).
3. `assets/app.js` loads and defines the shared globals.
4. The inline `<script>` runs: `loadYamlConfig()` fetches + parses `config.yml`,
   **`assertValidConfig()` validates it** (see below), `normaliseConfig()`
   flattens the `localisation:` block, `applyAccent()` / `applyMdcmsTheme()` /
   `initFontScale()` / `applyLang()` apply settings (the theme, if `config.yml`
   sets one, overrides accent-colour — see "Suite-wide theming" below), and
   `render()` builds the UI. State persists to `localStorage` under app-specific
   keys.

Boot order to preserve: `loadYamlConfig → assertValidConfig → normaliseConfig →
applyAccent → applyMdcmsTheme → initFontScale → applyLang → render`.

### Config validation (keep this guard)

`config.yml` is fetched at runtime. On a static host a **missing** `config.yml`
— or an SPA fallback, or a stale cached page — is served as an **HTML page with
a 200**. `jsyaml.load()` then returns a plain *string*, `CFG.localisation` is
undefined, `normaliseConfig()` silently early-returns, and the app paints its
full chrome with **every label showing as a raw key, an empty language dropdown,
and no error at all**. It looks broken with no clue why.

`assertValidConfig()` in the inline boot turns that into a clear, actionable
error:

```js
function assertValidConfig(cfg) {
  if (!cfg || typeof cfg !== 'object' || !cfg.localisation)
    throw new Error('config.yml loaded but did not parse to a valid configuration. '
      + 'The server most likely returned an HTML page (a 404 / SPA fallback, or a '
      + 'stale cached page) instead of the YAML file. …');
}
```

Keep this in the inline boot — **not** in `assets/app.js`, which must stay
byte-identical to bizdocs. (`loadYamlConfig()` deliberately doesn't validate
shape; that's the app's job.)

## Suite-wide theming

Beyond the single `accent-colour`, the app can be themed from
[mdcms](https://github.com/kbenestad/mdcms)'s theme library:

- **`config.yml`'s `theme:` key** names a **vendored** theme file, e.g.
  `theme: assets/themes/nord.yaml` — a plain copy of a file from mdcms's
  `themes/` folder, dropped into `assets/themes/` by hand (see
  `assets/themes/README.md`). Leave blank (the default) for basis's own look.
- **Same-origin, so no new dependency.** `applyMdcmsTheme()` (`app.js`)
  fetches the file via a plain relative `fetch()` — no CDN, no CSP change
  needed (this app doesn't even ship a CSP `<meta>`). Missing/broken theme
  file degrades silently to basis's own default look; a theme is cosmetic
  and must never break the app's actual function.
- **Applying it** is `kbApplyThemeToDoc()` in `app.js` — the same shared
  function bizdocs's own apps use for a real theme application, and that
  bizdocs's `themeselector/` app uses (targeting an iframe instead) to
  preview a theme before committing to one. See bizdocs's own CLAUDE.md
  ("Suite-wide theming" section) for the full palette-mapping rationale;
  it lives in `app.js`, so it's identical here via `sync.sh`.
- **A theme file itself is not synced automatically** — unlike
  `style.css`/`ui.css`/`app.js`, a `.yaml` file in `assets/themes/` is
  app-specific content, not one of the three shared files `sync.sh` pulls.
  Copy one by hand the same way you picked it.
- **Fonts are name-only** — `font-body`/`font-heading` only show if that
  font happens to be installed locally already; no webfont is fetched
  (kept out of `app.js` deliberately, to keep this feature at zero new
  dependency for every app that shares it, including this one).

## Building your app from this template

1. **Rename / rebrand.** Set the `<title>`, the doc-title `<h1>`, the brand
   fallback text, and the `basis-lang` localStorage key (and any other per-app
   keys) to your app's name.
2. **Build the form.** Replace `buildExampleCard()` with your real form, built
   **only from `kb-*` components** (see DESIGN.md). Put any bespoke layout
   (column grids, repeating rows, signature pad) in the inline `<style>`.
3. **Build the PDF.** Replace `onDownload()` with your real document using
   **pdf-lib** (`PDFLib`, already loaded).
4. **Fill in `config.yml`.** Keep the header keys; add every user-facing string
   to the `ui:` block of **every** language; route all copy through `S('key')`.
5. **Drop in real icons.** Replace `assets/favicon.svg` / `site.webmanifest`
   with a full favicon / PWA icon set.

## Running / previewing locally

The app `fetch`es `config.yml`, so it must be served over HTTP — opening
`index.html` via `file://` will fail.

```bash
python3 -m http.server 8000
# then open http://localhost:8000/
```

## Verifying a change

There are no automated tests — verify visually by rendering the app with a
headless Chromium:

```bash
CHROME=/path/to/chromium      # e.g. /opt/pw-browsers/chromium-*/chrome-linux/chrome
"$CHROME" --headless=new --no-sandbox --disable-gpu \
  --virtual-time-budget=8000 --window-size=1200,1600 \
  --screenshot=out.png "http://localhost:8000/index.html"
```

`--virtual-time-budget` lets the JS-built UI settle before the screenshot.

**Caveat:** in sandboxed environments the browser often cannot reach the CDNs,
so `js-yaml` fails to load and you'll see the config error. To verify a full
render, vendor `js-yaml` locally **for the test only** — download it, drop a copy
into `assets/`, point a throwaway copy of `index.html` at the local file, and
screenshot that. Delete the throwaway files afterwards; never commit them.
(`pdf-lib` is only needed to generate a PDF, not for the initial render.) Worth
screenshotting after a change: the full form, dark mode, the About modal, and a
non-English language.

## Development workflow

Day-to-day development happens on the **`development`** branch, not `main`.
Before a push to `main`, open a pull request — **the user decides when a PR is
opened**, don't push to `main` unopenedly on your own initiative.

**There are exactly two branches for this repo: `development` and `main`.
Never use, invent, or leave work stranded on any other branch** — no
`claude/whatever-slug`, no per-task branch of any kind, ever, for any reason,
including when a session's own harness/runtime hands you a differently-named
branch as that session's default. That default is a mechanism of the
*session*, not a decision about where this repo's history lives, and it is
never license to commit there and call the work done. When it happens: do
the work, then before finishing create `development` (from `main`, or from
the harness branch's tip if that has the newer work) if it doesn't already
exist, and push your commits there too, so `development` on the remote ends
up with every commit. Verify with `git ls-remote --heads origin` — don't
assume from having pushed *somewhere*.

While on `development`, keep these docs current:

- **`docs/unreleased.md`** — features that have landed on `development` but
  haven't shipped to `main` yet. Once a PR is pushed, update this doc so it
  reflects reality (move shipped items out, keep only what's still unreleased).
- **`docs/features.md`** — the running list of features the app has, kept in
  sync with what's actually shipped on `main`.
- **`docs/known-bugs.md`** — known bugs. Remove an entry the moment its bug is
  fixed; this file should only ever list what's currently broken.
- **`docs/roadmap.md`** (or `docs/roadmap.html`) — where the app is headed.
  Updated manually or by Claude Code, not tied to any particular release.

## Versioning (`VERSION.md`)

`VERSION.md` in the repo root tracks the app's release version, in this
format:

```
**kbenestad/{reponame} • {App name}**
**Version:** X.Y.Z - latest commit {commit}
**Last updated:** {Date - format d Mmmm YYYY}
```

Update it whenever a push to `main` happens:

- **X (major)** — a defined feature set is complete, or a breaking change
  makes the app incompatible with previous versions.
- **Y (minor)** — new features ship, increasing what the app can do.
- **Z (point)** — a new iteration ships, typically one PR. Several PRs can
  land before a point release if they're part of the same iteration — ask the
  user if it's unclear whether a batch of PRs warrants its own point release.

**Recording the merge commit is a direct-to-main push, not a PR.** Once a PR
merges, `VERSION.md` and the footer's `APP_VERSION.commit` need to point at
the resulting merge commit hash. That bookkeeping commit is pushed straight to
`main` rather than going through its own PR — a PR for it would itself need a
follow-up commit recording *its* merge hash, forever one commit behind. Keep
this exception narrow: it covers only updating the commit hash (and, when
relevant, the version number/date) to match a merge that already happened —
never new feature work.

## Footer

Every app built from this template must have a footer. Right-aligned, two
lines:

```
vX.Y.Z • {commit}
{d Mmmm YYYY}
```

The version/commit line matches `VERSION.md`; the second line is its
"Last updated" date. The commit links to the latest PR (the PR that shipped
it), not to a raw commit view.

Each PR description must:

- Mention any commits that landed directly on `main` without their own PR
  since the previous PR.
- Link back to the previous PR.

## Conventions (the pixel-perfect rules)

- **Reuse the shared layer; never fork it.** Styling and cross-cutting logic
  live in `assets/`. Don't reintroduce design tokens, `kb-*` components, DOM
  helpers, or theme/modal/i18n/format code in the app — use what `app.js` and
  the CSS already provide.
- **Same classes/IDs for the same element** as the bizdocs apps, so a synced
  `ui.css` change lands correctly. Only genuinely app-specific layout belongs in
  the inline `<style>`.
- **Scope.** The main inline script is wrapped in an IIFE
  (`(async function(){ … })()`), so its functions are not globals; the globals
  from `app.js` ($, el, makeSizeControl, kbAbout, lookupString, …) are visible
  inside it.
- **localStorage keys.** Theme = `kb-theme`, font scale = `kb-font-scale`
  (shared, don't rename). Per-app data uses your own keys (e.g. `basis-lang`).
- **No hardcoded user-facing text.** Add a key to every language block in
  `config.yml` and look it up via `S()`. Use `{placeholder}` tokens for values.
- **PDF output uses pdf-lib.** It can draw from scratch and embed/append
  existing PDF/image bytes (receipts, signatures), so it covers every app.
- **The container is ephemeral / hosting is static.** Commit your work; deploy
  `index.html` + `config.yml` + `assets/` together as plain static files.
