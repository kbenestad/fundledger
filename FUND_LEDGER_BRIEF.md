# Fund Ledger — Build Brief

## Purpose
Tracks donations and disbursements for a specific fund. Single operator, offline-capable, self-contained files opened directly from disk (`file://`) — no server, no build step at runtime.

## Conventions
Follow all conventions in the appdevelopment basis template upstream (shared assets, file layout, code style, bizdocs look). This app defers to shared bizdocs assets when placed in a bizdocs folder, matching the pattern used by contactmanager and other apps in that suite.

## Confirmed technical foundation
- **File System Access API** (`showDirectoryPicker`, `getDirectoryHandle`, `getFileHandle`, `createWritable`) works over `file://` (no local server needed) on Chrome, Vivaldi, and Edge, tested on Debian and Windows. **Firefox and Safari are not supported** — this API is unavailable there. Build for Chromium only; no fallback needed.
- **Vivaldi quirk**: the browser's folder-access permission dialog has a visually reversed default — "Accept" is blue-filled but "Deny" has the focus border. Mouse users are unaffected. Keyboard users must tab to the correct button rather than pressing Enter on the highlighted one. This is a browser rendering quirk, not something the app can fix — add a one-line note in any user-facing docs, no code workaround needed.
- **Folder permission is not permanent.** The browser will not remember directory access forever across sessions. Assumption to build against (flag if wrong): store the directory handle in IndexedDB and silently re-verify permission on each launch; the browser may still require a one-click reconfirmation — this is a browser security limit, not an app bug.

## Data model — file/folder structure

```
Ledger.html                  <- metadata-only index + dashboard, rewritten on every action
Donors/                      <- files on donor parties
Beneficiaries/               <- files on beneficiaries
Donations/                   <- files on individual donations
Disbursements/               <- files on individual disbursements
Templates/                   <- templates to print on each record creation
Changelog/
  changelog.jsonl            <- append-only log
```

- **`Ledger.html`**: flat metadata index only (references to records, running totals for the dashboard). This is the only file that gets rewritten repeatedly — every other record file is written once and never edited.
- **Donation and Disbursement records**: each creates a dated folder named `YYYY-MM-DD_<beneficiary-or-donor-id>_<random>`, e.g. `2026-12-31_B-0042_A1B2C3`. Random suffix is 6 characters, letters and digits.
  - Inside: `record.html` (the paper trail) plus any generated documents (donation acknowledgement, receipt — markdown-formatted) saved as separate files in the same folder, listed by reference in `record.html`. Keeping documents as separate files (not embedded) allows wet signatures to be added to the physical/scanned copies later.
- **Beneficiary and Donor records**: one file each, named `B-NNNN_Lastname Firstname.html` (or `D-NNNN_...` for donors). ID card scans are base64-encoded and embedded directly in this file (these files are expected to be larger; that's accepted, unlike the record-index file which must stay small).
- **Donations and Disbursements are independent records.** "Donation is the reverse of Disbursement" describes their structural relationship (money in vs. money out), not a linkage between specific entries — there is no reversal/offsetting mechanism to build.
  - **Amendment:** this section originally specified immutable, append-only records with no edit/delete. That guarantee was deliberately removed — all four record categories (Beneficiaries, Donors, Donations, Disbursements) now have Edit and Delete in the app, per an explicit user request. See `docs/unreleased.md` / `docs/features.md`.
- **Template directory**: Markdown-formatted documents that are printed upon generation of records. Template fields inserted as `{}` blocks [if it's better to use code-blocks and leading text let me know.]. Supports single and double signatures.

## Write order (proposed default — confirm before building)
On any new record: write the record folder and its files first, **then** update `Ledger.html` last. Rationale: if the app crashes mid-operation, an orphaned record folder with no index entry is recoverable (the index can be rebuilt by scanning folders); a broken index entry pointing to a nonexistent record is not. Flag if a different order is intended.

## Changelog
Append-only log at `Changelog/changelog.jsonl`, one JSON line per action: timestamp, action type, record ID, one-line description. This is the full versioning mechanism — no git, no snapshotting of past `Ledger.html` states.

## Tabs
1. **Dashboard** (first tab) — running total of donations, disbursements, balance. Requires the operator to open/select a ledger folder before anything else works.
2. **Disbursement** — operator selects beneficiary, amount, category, notes, disbursement date. Generates a unique reference number and markdown-formatted output documents (acknowledgement, receipt).
3. **Beneficiary** — behavior depends on a config switch:
   - ID-extraction mode: built on the ID Card Reader app's template, operator uploads an ID, relevant fields are extracted, operator adds custom fields as defined in `config.yml`. **Dependency**: confirm current status/API of the ID Card Reader app before building this mode — not yet scoped in this brief.
   - Pure form-driven mode: fields entirely defined by `config.yml`.
4. **Donation** — same shape as Disbursement, opposite direction (adds to balance rather than subtracting).
5. **Donor** — same shape as Beneficiary (ID-extraction or form-driven, per config switch).
6. **Configuration** — loads and edits `config.yml`: add/remove blocks and keys. Each key defined as `key : type : value`. Supported types:
   - `text` — standard single-line field
   - `textarea` — long text field
   - `date` — date field
   - `drop-down` — value field holds a list of options, e.g. `country : drop-down : "NO, Norway", "DK, Denmark"` (used heavily for Beneficiary/Donor form fields)
   - `number` — for figures such as money amounts. 
   - `random generator` — generates the 6-character alphanumeric reference code described above
   - `reference` - reference field to other Fund Ledger fields

### Tab behavior and general overview of keys

**Disbursements**: Contains basic information on disbursement.

- Reference number: `random generator`, six digits and letters.
- Beneficiary: `reference` field linked to beneficiary records
- Previous grant: `reference` field linked to beneficiary's previous grants
- Date of grant: `date` funds were granted
- Purpose of grant: `text` summarising what the grant is for.
- Notes: `textarea`, additional information. Supports linebreaks.
- Amount: `number`, total amoutn granted
- Date of disbursement: `date` money was disbursed
- Disbursement channel: `drop-down` showing how money is being disbursed; cash, bank transfer, cheque, money transfer service
  - Depending on choice, additional fields like Bank, Bank account number, Bank account holder, Name on cheque, number of checque, or money transfer service and tracking number is displayed

 Prints on creation: Donation acknowledgement, receipt.

**Donations**: Contains basic information about inflows.

- Reference number: `random generator`, six digits and letters.
- Donor: `reference` field linked to Donor records
- Linked donation: `reference` field linked to donor's previous grants
- Date of pledge: `date` funds were pledged
- Purpose of donation: `text` summarising what the grant is for.
- Notes: `textarea`, additional information. Supports linebreaks.
- Amount: `number`, total amount donated
- Date of donation: `date` money was donated
- Donation channel: `drop-down` showing how money is being disbursed; cash, bank transfer, cheque, money transfer service
  - Contains reference to receipt given

Prints on creation: Donation receipt

**Beneficiaries**: Two modes: Either plain-text entry or ID Card Reader capture.

- Key `mode`, value `plaintext` or `idcardreader`. `plaintext` is fallback. `idcardreader` uses UNHCR cards as primary source. When clicking `Create`, offered a choice to use ID Card Reader or plaintext.
- `idcardreader` sets field that are saved in the templates. Those data are used to build the `plaintext` config keys.

```
key
  fieldname: [unique name]
  type: text|textarea|date|number|drop-down|reference
  text: [character lengths - if other keys are implement]
  textarea: [lines]
  date: past-only|future-only|YYYY-MM-DD---YYYY-MM-DD [range]
  number_range: N---N [range]
  symbols: none|before|after
  symbols_print: [what to print before or after a number]
  drop-down: "Value, Value shown in dropdown", "Value, Value shown in dropdown", "Value, Value shown in dropdown" OR countries|months|drop-down-listname [reference to drop-down-lists, see below]
  dropdown_search: yes|no
  reference: [fieldname]
  reference_limit: all-records|same-id-only
 
drop-down-lists
  name: [drop-down-listname]
  values: "Value, Value shown in dropdown", "Value, Value shown in dropdown", "Value, Value shown in dropdown"
```

Default things to ask: 
- Name
- Date of birth
- Sex
- Country of origin
- UNHCR case file no.
- UNHCR case status
- Bank, bank account, bank account holder if relevant
- Phone number
- Email

**Donors**: Similar to beneficiaries: `plaintext` or `idcardreader` is the mode. Uses passports or ID cards as basis for extraction. Fallback to `plaintext`.

- Name
- Date of birth
- Sex
- Country of origin
- Phone number
- Email

#### ID Card Reader

Reuse app from `idcardreader`. Use that app to extract data, complete the beneficiary record with those data. Extracts relevant data and saves ID card and beneficiary photo on file.

## Open items to confirm before/during build
1. Write order default above — confirm or correct.
2. ID Card Reader app: current status, and what its "template" actually exposes for reuse.
3. IndexedDB-persisted folder handle — confirm this is the intended re-launch behavior.
