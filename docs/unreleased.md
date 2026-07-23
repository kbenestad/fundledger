# Unreleased

Features that have landed on `development` but haven't shipped to `main` yet.

- **Money-record lists sorted by their own date field.** The Existing-records
  list (app Configuration/money tabs) and `Ledger.html`'s Disbursements/
  Donations tabs previously ordered rows by creation/insertion order
  (newest-saved first). They now sort by each record's own
  `date-of-disbursement` / `date-of-donation` value (newest date first,
  reference as a tiebreaker), so backdated or out-of-order entries land in
  the right place.
