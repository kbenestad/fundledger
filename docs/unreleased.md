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
