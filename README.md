# Fund Ledger

Tracks **donations and disbursements for a single fund**. Single operator, no
server, no build step: the app is one `index.html` (plus bundled `assets/`)
that reads and writes a **ledger folder on your own disk** through the File
System Access API — it runs opened directly from disk (`file://`) or from any
static host. Nothing is ever uploaded anywhere.

Built from the bizdocs **basis** standalone starter
([design system](DESIGN.md), [conventions](CLAUDE.md)); ID-card data capture is
embedded from [kbenestad/idcardreader](https://github.com/kbenestad/idcardreader).
The full architecture and decision log is in
[FUND_LEDGER_PLAN.md](FUND_LEDGER_PLAN.md) (companion to
[FUND_LEDGER_BRIEF.md](FUND_LEDGER_BRIEF.md)).

## Browser support — Chromium only

Fund Ledger requires **Chrome, Vivaldi or Edge**. It is built on the File
System Access API, which does not exist in Firefox or Safari; there is no
fallback.

Two browser behaviours worth knowing (neither is an app bug):

- **Folder access is not remembered forever.** The app stores the folder
  handle and reconnects silently when it can, but the browser may require a
  one-click **Reconnect** confirmation at the start of a session. That is a
  browser security limit.
- **Vivaldi's folder-permission dialog** has a visually reversed default:
  "Accept" is blue-filled but keyboard focus sits on "Deny". Mouse users are
  unaffected; keyboard users should Tab to the correct button rather than
  pressing Enter on the highlighted one.

The app loads its libraries (js-yaml, and Tesseract.js/jsQR/PDF.js for ID-card
extraction) from CDNs, so the **first launch and ID extraction need internet**;
after that the browser cache usually covers offline use of the ledger core.

## The ledger folder

Pick (or create) a folder on first launch; the app scaffolds it:

```
Ledger.html                  index + dashboard — the only file ever rewritten;
                             openable directly in a browser, no app needed
config.yml                   fund, currency, entry modes, and every form field —
                             edited in the app's Configuration tab
Donors/                      one file per donor      <ID>_Name.html
Beneficiaries/               one file per beneficiary <ID>_Name.html
Donations/                   one folder per donation      YYYY-MM-DD_<id>_<REF>/
Disbursements/               one folder per disbursement  (record.html + documents)
Templates/                   markdown document templates with {fieldname} tokens
Changelog/changelog.jsonl    append-only action log (the versioning mechanism)
```

Every record file is **written once and never edited** — the paper trail is
immutable. Generated documents (acknowledgements, receipts) are standalone
printable HTML files saved next to each record, so wet signatures go on the
printed copies. If the index ever disagrees with the folders (e.g. after a
crash), **Dashboard → Rebuild index** regenerates `Ledger.html` by scanning
the record files.

## Configuration

The **Configuration** tab edits the ledger folder's `config.yml`: fund name
and currency, plaintext vs. ID Card Reader entry modes, all form fields
(`key : type : value` — text, textarea, date, number, drop-down,
random-generator, reference), drop-down lists, printed-document lists, an
optional inline mdcms theme block, and a raw-YAML editor.

The app's own boot config (branding + UI strings) is embedded near the top of
`index.html` — over `file://` a config file cannot be `fetch()`ed, which is
also why the root of this repo has no `config.yml`.

## Development

```bash
python3 -m http.server 8000    # or just open index.html in Chromium via file://
```

`assets/style.css`, `assets/ui.css` and `assets/app.js` are byte-identical
copies of bizdocs' shared design + runtime — never hand-edit them; refresh
with `./sync.sh`. `assets/capture.js` is a verbatim copy of idcardreader's
capture pipeline; sync it from that repo when it changes there. Day-to-day
work happens on the `development` branch (see [CLAUDE.md](CLAUDE.md)).
