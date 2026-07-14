# Unreleased

Features that have landed on `development` but haven't shipped to `main` yet.

Once a PR is pushed, update this file to reflect reality: move shipped items
out (they belong in [`features.md`](features.md)) and keep only what's still
unreleased.

## The Fund Ledger app (initial build)

The basis starter has been turned into the Fund Ledger app described in
`FUND_LEDGER_BRIEF.md` / planned in `FUND_LEDGER_PLAN.md`:

- **Ledger folder on disk** via the File System Access API (works over
  `file://`; Chromium-only): folder picker, structure + `config.yml` +
  default-template seeding on first open, IndexedDB-persisted handle with
  silent re-verify and a one-click Reconnect button.
- **Six tabs**: Dashboard, Disbursement, Beneficiary, Donation, Donor,
  Configuration.
- **Immutable records**: beneficiaries/donors as single `B-NNNN`/`D-NNNN`
  files; donations/disbursements as dated `YYYY-MM-DD_<id>_<REF>` folders with
  `record.html` + generated documents. Written once, never edited.
- **`Ledger.html`** — metadata-only index (embedded JSON) + human dashboard,
  rewritten on every action; write order record → changelog → index, plus a
  Rebuild-index recovery action that rescans the record folders.
- **Append-only changelog** at `Changelog/changelog.jsonl`.
- **Config-driven forms** from the ledger's `config.yml`: text, textarea,
  date (past/future/range), number (ranges + currency symbols), drop-down
  (inline values, `countries`, `months`, named lists, searchable), random
  6-char reference generator, reference fields (`all-records` /
  `same-id-only`), and `show_if` channel-conditional fields.
- **Document generation**: markdown templates in `Templates/` with
  `{fieldname}` tokens and `{signatures:1|2}` blocks, rendered to standalone
  printable HTML files saved in the record folder; auto print dialog on
  creation.
- **Embedded ID Card Reader mode** (per-tab `plaintext`/`idcardreader`
  switch): verbatim `capture.js` from kbenestad/idcardreader, embedded UNHCR
  Thailand card + passport-MRZ templates, capture → crop → OCR/QR/MRZ
  extraction → review/conflict resolution → `from_id` mapping onto the form;
  cardholder photo and card scans embedded base64 in the person file.
- **Configuration tab**: accordion sections (fund & currency, entry modes,
  per-block field editors, drop-down lists, inline mdcms theme block, raw
  YAML editor) writing back to the folder's `config.yml`.
- **Single fund currency** from config, used on forms, documents, dashboard
  and `Ledger.html` totals.
- **Inline boot config** near the top of `index.html` (branding + all UI
  strings) — `fetch()` is unavailable over `file://`, so the repo-root
  `config.yml` is gone; theme support accepts an inline mdcms theme object
  (applied fetch-free via the shared `kbApplyThemeToDoc()`).
- New app favicon; README rewritten for the app (Chromium-only note, Vivaldi
  permission-dialog quirk, reconnect behaviour).
