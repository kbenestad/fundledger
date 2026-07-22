# Unreleased

Features that have landed on `development` but haven't shipped to `main` yet.

- **Clickable files in Ledger.html**: document links and person-record file paths
  in the detail overlay are now clickable. Uses a `postMessage` relay to the
  parent app window (which reads the file via File System Access API and
  returns a blob URL); falls back to native relative-path links when Ledger.html
  is opened directly from disk on `file://`.
