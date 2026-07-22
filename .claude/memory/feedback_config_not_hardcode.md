---
name: feedback_config_not_hardcode
description: Do not hardcode labels, settings, or values in index.html JS. Put configurable settings in config.yml (the ledger's seed config), not in boot-config YAML or JS code.
metadata:
  type: feedback
---

Don't add hardcoded UI strings, labels, or configurable values to the boot-config YAML block or the JavaScript in index.html.

**Why:** User was explicit: "NO DO NOT MAKE CHANGES TO INDEX!!!! STOP HARDCODING!!! PUT IT IN CONFIG!!!" — new app-level settings (typography, max-width, logo, etc.) belong in the ledger's `config.yml` (the seed-config YAML block in index.html, which becomes the actual configurable file in the ledger folder). The JavaScript should just read from `STATE.ledger.config`.

**How to apply:**
- New document/output settings → add to `seed-config` YAML in index.html, under a new key (e.g., `documents:`)
- JavaScript reads from `STATE.ledger.config.documents` at render/save time
- Do NOT add a new dedicated UI section in the Config tab for these settings; users edit them via the existing "Raw YAML" accordion
- Do NOT add new keys to the boot-config YAML (that's for app branding / localisation, not operational config)
- Do NOT hardcode font stacks, colors, max-widths, or other configurable values in JS strings
