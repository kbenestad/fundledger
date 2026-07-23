# Features

The running list of what Fund Ledger provides, kept in sync with what's shipped
on `main`.

- **Ledger folder on disk** via the File System Access API (Chromium-only,
  works over `file://`): folder picker, automatic folder skeleton + `config.yml`
  seeding on first open, IndexedDB-persisted folder handle with silent
  re-verify and a one-click Reconnect button.
- **Six tabs**: Dashboard, Disbursement, Beneficiary, Donation, Donor,
  Configuration. The Configuration tab can be hidden via `show-config-tab: no`
  in `config.yml`.
- **Dashboard**: fund name prominent, folder name in monospace; totals
  (donations, disbursements, balance); record counts; recent-activity feed;
  View Ledger button; Rebuild index; Close ledger.
- **Records, editable and deletable**: beneficiaries/donors saved as
  `<ID>.html` person files (ID is a random 6-character code, the same
  pattern used for donation/disbursement references — no B-/D- prefix or
  per-kind counter); donations/disbursements saved as dated
  `YYYY-MM-DD_<id>_<REF>/` folders containing `record.html` and any
  generated documents. All four categories can be edited or deleted from
  their tab's record list (see below) — records are no longer strictly
  immutable/append-only; the changelog still records every create/update/
  delete action.
- **`Ledger.html`**: metadata-only index (embedded JSON) + human-readable
  dashboard rewritten on every action; write order is record → changelog →
  index; Rebuild-index action rescans all record folders. Disbursements,
  Beneficiaries, Donations, and Donors are separate tabs with matching
  column widths; clicking a beneficiary or donor lists their linked
  disbursements/donations (each opens that record's own detail); fund name
  is shown above a fixed "Fund Ledger" heading; the "Generated" timestamp
  uses local machine time with a UTC offset; stat amounts are right-aligned.
- **Append-only changelog** at `Changelog/changelog.jsonl`.
- **Config-driven forms** from the ledger's `config.yml`: field types `text`,
  `textarea`, `date` (past/future/range), `number` (ranges + currency
  symbols), `drop-down` (inline values, `countries`, `months`, named lists,
  searchable), `random-generator` (6-char reference), `reference`
  (`all-records` / `same-id-only`), and `show_if` conditional visibility.
- **Document generation**: free-form markdown templates in `config.yml` with
  `{fieldname}` tokens, `{logo}`, and `{signatures}`; rendered to standalone
  printable HTML files saved in the record folder; auto print dialog on save.
- **Multi-language documents**: define languages (code, name, direction,
  line-height, font) in `config.yml`; a language selector appears on money
  forms when ≥2 languages are defined; per-language template strings and
  signature labels supported.
- **Signature blocks**: configurable count (1 or 2) and label text per block
  (token-substituted); 1 signature = 40% width centred; 2 signatures = 40%
  each at left and right.
- **Reprint**: any saved money record can be reprinted from its record list.
- **Embedded ID Card Reader mode** (per-tab `plaintext`/`idcardreader`
  switch): verbatim `capture.js` from kbenestad/idcardreader; embedded UNHCR
  Thailand card + passport-MRZ templates; capture → crop → OCR/QR/MRZ
  extraction → review/conflict resolution → `from_id` mapping onto the form;
  cardholder photo and card scans embedded base64 in the person file.
- **Existing-records list with Edit and Delete**, for all four categories
  (Beneficiary, Donor, Donation, Disbursement), collapsed in an accordion;
  display-ID column for persons. Edit loads the existing record into the
  form for update; for donations/disbursements this regenerates the
  document(s) and `record.html` in place (same reference/date/folder).
  Delete permanently removes a person's record file, or a money record's
  whole folder (including its generated documents); deleting a
  beneficiary/donor warns how many existing money records reference that ID
  (they keep showing it — money records are never rewritten by a person
  deletion) but does not block the delete.
- **Configuration tab accordion**: fund & currency (plus a `show-config-tab`
  toggle), languages, entry modes, per-block field editors, template +
  signature editors (with "Available tokens" modal), drop-down lists, a
  "Document layout" section (logo, document theme, fonts, sizes, widths,
  print top padding — see below), document footer, mdcms theme block, raw
  YAML editor. Template and signature-label editors show per-language
  textareas when multiple languages are defined, updating live as languages
  are added. Clearing a Document layout field deletes that key from
  `config.yml` rather than writing a blank value, so it correctly falls back
  to the app-level config or the built-in default instead of silently
  overriding a lower-priority value with nothing.
- **`padding-top-print`** key under `documents:` in `config.yml` sets the
  top margin (e.g. `2cm`) above the first line when printing or saving as PDF;
  editable directly in the Configuration tab's Document layout section.
- **Print reliability**: printing/saving as PDF waits for any per-language
  document web font to finish loading (with a safety timeout) before opening
  the print dialog, avoiding blank/invisible text from a font that's still
  downloading.
- **Ledger.html detail overlay**: clicking any reference, donation, disbursement,
  beneficiary, or donor ID opens an in-page panel showing all field values, the
  document list, and the record's folder/file path with a Copy button. Works
  whether Ledger.html is opened from the app or directly from disk.
- **Shared bizdocs design system** bundled in `app/assets/` (`style.css`,
  `ui.css`, `app.js`), kept byte-identical to bizdocs via `sync.sh`.
- **Theme and font-scale controls**, persisted to `localStorage`.
- **Inline boot config** near the top of `index.html` (branding + all UI
  strings) — works over `file://` with no server required.
- **Logo in generated documents**: place `logo.png`, `logo.jpg`, or `logo.svg`
  in the ledger folder root; loaded via File System Access API when the ledger
  connects and embedded base64 in every receipt. Works on `file://` and HTTP.
- **Documents always render in light mode** (`color-scheme: only light`):
  receipts show black text on white regardless of the OS dark-mode setting or
  any embedded theme's dark palette.
- **Configurable document footer**: `footer:` key under `documents:` in
  `config.yml` adds markdown-formatted text below every generated receipt,
  separated by a horizontal rule. Supports `{tokens}` and per-language maps.
  Editable in the Config tab "Document footer" section.
- **Config reference** at `docs/config.md`: full documentation for every key
  in `config.yml`.
- **Clickable files in `Ledger.html`**: document links and person-record file
  paths in the detail overlay are clickable. Uses a `postMessage` relay to the
  parent app window (which reads the file via File System Access API and
  returns a blob URL); falls back to native relative-path links when
  `Ledger.html` is opened directly from disk on `file://`.
