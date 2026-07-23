# Unreleased

Features that have landed on `development` but haven't shipped to `main` yet.

- **Edit and Delete for all four record categories** (Beneficiaries, Donors,
  Donations, Disbursements): Donations/Disbursements previously had no list of
  existing records in the app at all — just a create-new form. All four
  categories now show an "Existing records" accordion with Edit and Delete
  per row. Editing a money record regenerates its document(s) and
  `record.html` in place (same reference/folder); deleting a money record
  permanently removes its whole folder, including generated documents.
  **This removes the immutable/append-only guarantee** documented for
  Donations/Disbursements in `FUND_LEDGER_BRIEF.md` — a deliberate,
  explicitly-requested change, not a bug fix. Deleting a Beneficiary/Donor
  that has linked money records warns how many records will be left pointing
  at a now-missing ID, but does not block the deletion.
- **Fix: Signature labels field in the Configuration tab never saved.** The
  textarea's `input` handler referenced an out-of-scope variable, throwing a
  `ReferenceError` on every keystroke, so the `signature-labels` value was
  silently left unchanged no matter what was typed — "Save configuration"
  always wrote back the old value for that field. This is why the donation
  template's default signature label stayed stuck on `{donor-name}` even
  after editing it. Also changed the seed default for new ledgers: the
  donor/beneficiary signature label now defaults to the generic
  "Name (PRINTED):" instead of auto-filling the linked person's name.
