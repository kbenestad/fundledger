# Unreleased

Features that have landed on `development` but haven't shipped to `main` yet.

- **Logo loading via File System Access API**: `logo.png` / `.jpg` / `.svg` placed
  in the ledger folder root is now loaded when the ledger connects, so logos appear
  in generated documents on `file://` (not just HTTP). The existing fetch-based path
  is kept as a fallback for HTTP serving.
- **Generated documents force light mode** (`color-scheme: only light`): prevents
  the browser from applying dark-mode colors to receipts and acknowledgements,
  regardless of OS setting or embedded theme.
- **Config reference** at `docs/config.md`: documents every key in `config.yml`.
