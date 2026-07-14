# Fund Ledger — Build Plan

Companion to [FUND_LEDGER_BRIEF.md](FUND_LEDGER_BRIEF.md). This document records
the dependency analysis, the decisions confirmed with the user on 14 July 2026,
the resulting architecture, and the build phases.

---

## 1. Resolution of the brief's open items

### 1.1 Write order — **confirmed**
On any new record: write the record folder and its files first, then append to
`Changelog/changelog.jsonl`, then rewrite `Ledger.html` last. The dashboard gets
a **"Rebuild index"** recovery action that regenerates `Ledger.html` by scanning
the record folders, so an orphaned record folder (crash between steps) is always
recoverable.

### 1.2 ID Card Reader — current status and what is reusable
Analysed at `kbenestad/idcardreader` (2026-07-14). Findings:

- It is a **working, shipped app** built on the same basis starter: an extractor
  (`app/index.html`) and a template builder (`app/template.html`), sharing the
  bizdocs chrome.
- **There is no callable API** — its "template" for reuse means two things:
  1. **`app/assets/capture.js`** (644 lines, app-owned, deliberately UI-free):
     globals `captureToImage()` (image/PDF → canvas, multi-page picker),
     `attachCropRotate()` (aspect-locked crop + fine rotation), `cropRegion()`,
     `ocrField()` (Tesseract), `decodeQr()` (jsQR), `parseQrPayload()`,
     `normaliseDate()` (incl. Buddhist calendar + Thai digits), `ocrMrz()` /
     `parseMrzTd3()` (ICAO 9303 passport MRZ with check digits), and
     `canvasToDataURL()`. **This file is copied verbatim into fundledger** —
     it has no dependency on the extractor's UI, only on the CDN libraries
     (Tesseract.js, jsQR, PDF.js).
  2. **Document templates** (`app/templates/*.yaml`) — regions in % of the
     image, resolution-independent. Two are directly relevant:
     - `unhcr-thailand-id-card.yaml`: cardholder photo, card QR, case number,
       individual ID number, date of birth, sex, country of origin, issue and
       expiry dates — this covers most of the brief's beneficiary defaults.
     - `passport-mrz.yaml`: TD3 MRZ extraction for donors.
- The capture → crop → extract → review-each-field flow (conflict resolution
  between OCR and QR values, per-field "Reviewed ✓" gate) lives inline in the
  extractor's script; the **pattern is copied and adapted**, not imported.
- Its extraction stack is CDN-loaded (Tesseract.js 7, jsQR 1.4, PDF.js 6);
  OCR language data downloads on first use and is then browser-cached.

**Decision (user, confirmed): embed the ID Card Reader in-app.** fundledger
vendors `capture.js` unchanged, embeds the two card templates, and rebuilds the
review UI inside the Beneficiary/Donor tabs. No link-out to the separate app.

### 1.3 IndexedDB-persisted folder handle — **confirmed**
Store the `FileSystemDirectoryHandle` in IndexedDB. On each launch:
`queryPermission({ mode: 'readwrite' })`; if `granted`, reconnect silently; if
`prompt`, show a one-click **Reconnect** button (browsers require a user gesture
for `requestPermission()`). This is a browser security limit, documented in the
README, not an app bug. The README also carries the one-line Vivaldi note about
the visually reversed focus in the folder-permission dialog.

---

## 2. Decisions confirmed with the user

| # | Topic | Decision |
|---|-------|----------|
| 1 | Libraries / offline | **CDN for what actually works online** (js-yaml, Tesseract.js, jsQR, PDF.js — same tags as idcardreader). Since `fetch('config.yml')` fails over `file://`, the app's boot config is **embedded near the top of `index.html`**, and the **Configuration tab surfaces everything**, with each section wrapped in an accordion so it isn't overwhelming. |
| 2 | Generated documents | **Standalone HTML + print dialog.** Fill the markdown template, render to a self-contained `.html` saved in the record folder, auto-open the print dialog on creation (wet signatures go on the printed copy). |
| 3 | Currency | **Single fund currency** set in config; used on all amount fields, documents and dashboard totals. The per-field `symbols`/`symbols_print` spec still applies for display. |
| 4 | Write order | Record first, index last, plus rebuild-index recovery (§1.1). |
| 5 | Folder handle | IndexedDB + silent re-verify + Reconnect button (§1.3). |
| 6 | Template placeholders | **`{fieldname}` named tokens** (e.g. `{beneficiary-name}`, `{amount}`, `{reference}`) — same style as the family's i18n `{placeholder}` convention. |
| 7 | ID capture | Embedded in-app (§1.2). |

---

## 3. Architecture

### 3.1 Two configs, one editor

`file://` makes `fetch()` unusable, so configuration splits by lifecycle:

- **Boot config** — branding, accent, and the full `localisation:` block —
  lives in an inline `<script type="text/yaml" id="boot-config">` **near the
  top of `index.html`**, parsed with js-yaml at boot. This is what lets the
  chrome (toolbar, language picker, About) render before any folder is open.
  It also carries the **seed defaults**: the default ledger `config.yml`
  content, the default markdown document templates, and the embedded ID-card
  extraction templates (UNHCR card, passport MRZ) — everything the app needs
  to initialise a brand-new ledger folder.
- **Ledger config** — `config.yml` **inside the ledger folder**, read/written
  through the File System Access API (which always works, no fetch involved).
  Created on first open by copying the seed defaults. This is the file the
  brief's **Configuration tab** loads and edits: every section shown, one
  accordion per block (fund/currency, disbursement fields, donation fields,
  beneficiary fields, donor fields, drop-down lists, templates, modes).

Boot order becomes: pre-paint theme script → CDN libs → shared `assets/`
(`style.css`, `ui.css`, `app.js`) → `assets/capture.js` (verbatim idcardreader
copy) → inline boot: parse `#boot-config` → `assertValidConfig` →
`normaliseConfig` → `applyAccent` → apply theme (§3.1.1) → `initFontScale` →
`applyLang` → `render`.

#### 3.1.1 Suite-wide theming — inline mdcms theme block

The config's `theme:` key accepts, in addition to the family's usual vendored
file path, a **full inline mdcms theme block** (the `mdcms v0.4` format:
`palette.light` / `palette.dark` with `primary`, `surface`, `page`, `ink`,
`ink-muted`, `on-surface`, `on-surface-active/-title/-heading/-note/-icon`;
`colours-semantic` and `colours-semantic-dark`; `callouts`; `font-body` /
`font-heading` as `provider:Name:weight`; `font-size`; `line-height`).

No new mapping code is written: the shared `assets/app.js` already turns
exactly these keys into CSS via `kbBuildThemeCss()` and applies them with
`kbApplyThemeToDoc(theme, document)` — the same function bizdocs' own apps
use. The app-side logic is only a dispatch:

- `theme:` is an **object** → pass it straight to `kbApplyThemeToDoc()`.
  Fetch-free, so this is the path that works over `file://` — and the reason
  the inline block exists.
- `theme:` is a **string** path → fall back to the family's
  `applyMdcmsTheme()` fetch (works when the app is HTTP-hosted; degrades
  silently over `file://`, since a theme is cosmetic and must never break
  function).

Keys the shared mapper doesn't consume (`callouts`, `font-size`,
`line-height` — mdcms-page concepts) are tolerated and ignored, per the
shared layer's behaviour. Fonts stay **name-only** (no webfont is fetched),
matching the family rule. The theme block lives in the boot config and is
surfaced as its own accordion in the Configuration tab; a `theme:` block in
the ledger folder's `config.yml` overrides the boot one once the folder is
open (re-applied via the same `kbApplyThemeToDoc()`).

### 3.2 File-system layer

All ledger I/O goes through one small app-owned module (inline in
`index.html`):

- `openLedger()` — `showDirectoryPicker({ mode: 'readwrite' })`, ensure the
  folder skeleton exists (`Donors/`, `Beneficiaries/`, `Donations/`,
  `Disbursements/`, `Templates/`, `Changelog/`), seed `config.yml` and default
  templates if missing, persist the handle to IndexedDB.
- `reconnect()` — IndexedDB handle → `queryPermission` → silent resume or
  Reconnect button (§1.3).
- `writeRecord()` — enforces the write order (§1.1).
- `appendChangelog(entry)` — one JSON line: `{ ts, action, id, description }`.
- `rebuildIndex()` — scan record folders, regenerate `Ledger.html`.

### 3.3 `Ledger.html` — index + dashboard in one file

A generated, self-contained HTML file, rewritten on every action:

- **Machine part**: `<script type="application/json" id="ledger-index">` —
  arrays of `{ id, name/reference, date, amount, path }` for donors,
  beneficiaries, donations, disbursements; the ID counters (next `B-NNNN` /
  `D-NNNN`); running totals. This is what the app reads on open — it never
  re-parses record files in normal operation.
- **Human part**: a static dashboard — totals (donations in, disbursements
  out, balance, in the fund currency), recent records, and **relative links**
  to every record file, so `Ledger.html` is useful opened directly in a
  browser with the app closed.

It stays metadata-only and small: no images, no document content.

### 3.4 Records

- **Beneficiary / Donor**: one file, `B-NNNN_Lastname Firstname.html` /
  `D-NNNN_…`. Self-contained: all field values with one-click Copy buttons
  (idcardreader record style), base64-embedded cardholder photo and card
  scans (these files are allowed to be large), and an embedded JSON block so
  `rebuildIndex()` can re-read it.
- **Donation / Disbursement**: a folder
  `YYYY-MM-DD_<B/D-id>_<RANDOM6>` (6 chars, A–Z 0–9) containing `record.html`
  (same self-contained style, embedded JSON, links to its documents) plus each
  generated document as a separate printable `.html`. Written once, never
  edited — immutable, append-only, and independent of each other (no
  reversal/offset mechanism).

### 3.5 Config-driven forms

One generic form renderer builds every record form from the ledger config's
field definitions, using only `kb-*` components. Supported types, per the
brief's key spec:

- `text` (optional character length), `textarea` (lines), `date`
  (`past-only` | `future-only` | `YYYY-MM-DD---YYYY-MM-DD`), `number`
  (`number_range`, `symbols: none|before|after` + `symbols_print`),
  `drop-down` (inline `"value, label"` pairs, or `countries` | `months` |
  named list from `drop-down-lists`; `dropdown_search: yes|no`),
  `random generator` (the 6-char reference), `reference` (`reference:
  fieldname`, `reference_limit: all-records|same-id-only` — e.g. "previous
  grant" limited to the selected beneficiary's own records).
- **Conditional fields** for the channel behaviour ("bank transfer" reveals
  bank/account/holder; "cheque" reveals name/number; "money transfer service"
  reveals service/tracking number): a `show_if: fieldname=value` key on any
  field. This is an addition to the brief's spec, needed to express the
  channel-dependent fields declaratively.

Default field sets (disbursement, donation, beneficiary, donor) ship in the
seed config exactly as listed in the brief.

### 3.6 ID-extraction mode

Per-tab `mode: plaintext | idcardreader` in the ledger config (`plaintext` is
the fallback). In `idcardreader` mode, **Create** offers a choice: ID Card
Reader or plaintext.

ID path: capture (upload/scan/PDF) → crop/rotate → extract via `capture.js`
(OCR/QR/MRZ/photo) → per-field review with conflict resolution (adapted from
the extractor) → extracted values **mapped onto the configured plaintext
fields** (a `from_id: <template-field-id>` key on a config field links it to
the card template), operator fills the remaining custom fields → save. The
cardholder photo and the card scans are base64-embedded in the person's file.
Beneficiaries use the UNHCR Thailand card template; donors use the passport
MRZ template — both embedded inline (fetch-free), synced by hand from
idcardreader when they change there.

Offline caveat (accepted): extraction needs the CDN stack and, once, the OCR
language data; the plaintext path has no such dependency.

### 3.7 Documents and printing

`Templates/` in the ledger folder holds markdown templates with `{fieldname}`
tokens, seeded on first open (disbursement acknowledgement + receipt, donation
receipt) and editable by the operator as plain files. A `{signatures:1}` /
`{signatures:2}` token renders single/double signature blocks (name + line +
date). On record creation the app reads the template through the FS API, fills
the tokens, renders markdown → a small self-contained printable `.html` saved
in the record folder, and opens the print dialog. Which templates print per
record type is set in the config.

### 3.8 Tabs

`makeTabs()` with: **Dashboard · Disbursement · Beneficiary · Donation ·
Donor · Configuration**. Until a ledger folder is open, Dashboard shows only
the open/reconnect card and the other tabs are disabled.

---

## 4. Build phases

1. **Scaffold + FS layer.** Rename basis → Fund Ledger (`fundledger-lang`
   key, title, header), inline boot config, tab chrome, `openLedger()` /
   `reconnect()` / IndexedDB, folder skeleton + seeding, changelog append,
   `Ledger.html` write/read/rebuild.
2. **Configuration engine.** Ledger-config schema + validation, the generic
   form renderer (all field types incl. `show_if`), Configuration tab with
   accordion sections and add/remove keys and drop-down lists.
3. **Beneficiary + Donor, plaintext mode.** Person forms from config,
   `B-NNNN`/`D-NNNN` counters, person record files, index + changelog wiring.
4. **Donation + Disbursement.** Record forms (reference fields with
   `same-id-only`, channel conditionals, random reference), record folders,
   document generation from `Templates/` + print dialog, totals.
5. **ID-extraction mode.** Vendor `capture.js`, embed card templates,
   capture/review UI in Beneficiary/Donor tabs, field mapping, photo + scan
   embedding.
6. **Polish + release.** Dashboard proper (totals, recent activity, rebuild
   button), README (Vivaldi note, Chromium-only, permission behaviour),
   `docs/` updates, dark-mode/i18n screenshot verification, PR to `main`.

Each phase lands on `development` and is independently verifiable with the
headless-Chromium screenshot flow from CLAUDE.md (with the `file://`
behaviours exercised manually in a real Chromium, since the FS Access picker
needs a user gesture).

---

## 5. Known constraints (accepted)

- **Chromium-only** (Chrome, Vivaldi, Edge): the File System Access API does
  not exist in Firefox/Safari. No fallback is built.
- **CDN at runtime**: first launch and ID extraction need the network; a
  browser that has cached the scripts works offline for the ledger core.
- **One-click reconfirmation** of folder access may be required per session —
  browser security, surfaced as the Reconnect button.
- The Vivaldi permission-dialog focus quirk is documented, not worked around.
