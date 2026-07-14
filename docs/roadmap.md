# Roadmap

Where this app (Fund Ledger) is headed. Updated manually or by Claude Code —
not tied to any particular release.

The full build plan — dependency analysis of `kbenestad/idcardreader`,
confirmed decisions, architecture, and phases — lives in
[`../FUND_LEDGER_PLAN.md`](../FUND_LEDGER_PLAN.md) (companion to
[`../FUND_LEDGER_BRIEF.md`](../FUND_LEDGER_BRIEF.md)).

Build phases, in order:

1. Scaffold + File System Access layer (folder open/reconnect, IndexedDB
   handle, folder skeleton + seeding, changelog, `Ledger.html` write/read/
   rebuild).
2. Configuration engine (ledger-config schema, generic form renderer,
   Configuration tab with accordion sections).
3. Beneficiary + Donor tabs, plaintext mode.
4. Donation + Disbursement tabs (channel conditionals, reference fields,
   document generation from `Templates/` + print).
5. ID-extraction mode (embedded ID Card Reader pipeline).
6. Polish + first release to `main`.
