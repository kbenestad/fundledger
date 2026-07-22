# Configuration reference

All configuration lives in `config.yml` inside the ledger folder. The app
seeds a default copy on first open; subsequent edits go through the
Configuration tab or by editing the file directly.

---

## Top-level keys

| Key | Values | Default | Description |
|-----|--------|---------|-------------|
| `show-config-tab` | `yes` \| `no` | `yes` | Show or hide the Configuration tab. Set to `no` to prevent end-users from changing settings. |

---

## `fund:`

```yaml
fund:
  name: "My Fund"
  currency-code: "USD"
  currency-symbol: "$"
  currency-position: before   # before | after
```

| Key | Description |
|-----|-------------|
| `name` | Fund name, shown on the dashboard and substituted as `{fund-name}` in document templates. |
| `currency-code` | Three-letter ISO currency code (informational; shown in UI labels). |
| `currency-symbol` | Symbol printed before or after monetary amounts. |
| `currency-position` | `before` (e.g. $100) or `after` (e.g. 100 ฿). |

---

## `documents:`

Controls generated receipts and acknowledgements.

```yaml
documents:
  logo: yes
  logo-max-width-web: 250px
  logo-max-width-pdf: 7.5cm
  padding-top-print:
  theme:
  font-body:
  font-heading:
  font-size: 15
  line-height: 1.6
  max-width-web: 720px
  max-width-pdf: 17cm
```

### Logo

| Key | Values | Default | Description |
|-----|--------|---------|-------------|
| `logo` | `yes` \| `no` | `yes` (if unset) | Whether to embed the logo in generated documents. Use the `{logo}` token in a template to place it. |
| `logo-max-width-web` | CSS length | `200px` | Maximum logo width in the browser/screen view. |
| `logo-max-width-pdf` | CSS length | `5cm` | Maximum logo width when printing or saving as PDF. |

**Logo file location**: place `logo.png`, `logo.jpg`, or `logo.svg` in the
**ledger folder root** (the folder you opened or created when you first launched
the app). The app loads the file via the File System Access API when the ledger
is connected, so it works on both `file://` and `http://`.

### Typography

| Key | Values | Default | Description |
|-----|--------|---------|-------------|
| `font-body` | `"provider:Font Name:weights"` | system-ui | Body font. Provider: `bunny` or `google`. Example: `"bunny:IBM Plex Sans:300,400"`. |
| `font-heading` | same format | same as body | Heading font. |
| `font-size` | integer (px) | `15` | Base font size in pixels. |
| `line-height` | number | `1.6` | Line height multiplier. |

### Layout

| Key | Values | Default | Description |
|-----|--------|---------|-------------|
| `max-width-web` | CSS length | `720px` | Sheet max-width in the browser. |
| `max-width-pdf` | CSS length | `17cm` | Sheet max-width in print / save-as-PDF. |
| `padding-top-print` | CSS length | none | Top margin above the first line of content when printing. Example: `2cm`. |

### Theme

| Key | Values | Default | Description |
|-----|--------|---------|-------------|
| `theme` | relative path | none | Path to an mdcms theme YAML file (e.g. `assets/themes/nord.yaml`). Leave blank for plain black-on-white. Theme only affects screen view; print always uses black on white. |

---

## `beneficiary:` and `donor:`

```yaml
beneficiary:
  mode: plaintext          # plaintext | idcardreader
  id-template: unhcr-thailand-id-card
  display-id: unhcr-case-no
  fields:
    - { fieldname: name, label: "Name", type: text, required: yes }
    ...
```

| Key | Values | Default | Description |
|-----|--------|---------|-------------|
| `mode` | `plaintext` \| `idcardreader` | `plaintext` | `idcardreader` enables the embedded camera/scan capture flow. |
| `id-template` | `unhcr-thailand-id-card` \| `passport-mrz` | — | OCR/MRZ template used in `idcardreader` mode. |
| `display-id` | field name | — | Field whose value is shown as the ID prefix in drop-downs and lists. |
| `fields` | list of field definitions | — | See [Field keys](#field-keys) below. |

---

## `donation:` and `disbursement:`

```yaml
donation:
  folder-date-field: date-of-donation
  signatures: 1
  signature-labels:
    - "{donor-name}"
  template: |
    {logo}
    # {fund-name}
    ...
  fields:
    ...
```

| Key | Values | Default | Description |
|-----|--------|---------|-------------|
| `folder-date-field` | field name | — | Field whose value is used to date-prefix the record folder name. |
| `signatures` | `1` \| `2` | `1` | Number of signature blocks inserted by `{signatures}`. |
| `signature-labels` | list of strings | — | Label for each signature block (token-substituted). Single list for all languages, or a map of `code: [...]` for per-language labels. |
| `template` | markdown string | — | Document template in Markdown. See [Template tokens](#template-tokens). Can be a plain string or a `{en: "...", th: "..."}` map for per-language templates. |
| `fields` | list of field definitions | — | See [Field keys](#field-keys) below. |

---

## Field keys

Every entry in a `fields:` list accepts:

| Key | Values | Default | Description |
|-----|--------|---------|-------------|
| `fieldname` | slug | required | Unique identifier; also the `{fieldname}` token in templates. |
| `label` | string | fieldname | Label shown on the form. |
| `type` | see below | required | Field type. |
| `required` | `yes` \| `no` | `no` | Whether the field must be filled before saving. |
| `show_if` | `fieldname=value` | — | Field is only shown when another field has a specific value. |
| `from_id` | ID-template field ID (or list) | — | Pre-fills this field from the ID card reader result. |

### Field types

#### `text`
Plain single-line text.

| Extra key | Description |
|-----------|-------------|
| `text` | Maximum character length (integer). |

#### `textarea`
Multi-line text.

| Extra key | Description |
|-----------|-------------|
| `textarea` | Number of visible rows (integer). |

#### `date`
Date picker.

| Extra key | Values | Description |
|-----------|--------|-------------|
| `date` | `past-only` \| `future-only` \| `YYYY-MM-DD---YYYY-MM-DD` | Restricts selectable dates. |

#### `number`
Numeric input.

| Extra key | Values | Description |
|-----------|--------|-------------|
| `number_range` | `N---N` | Minimum and maximum value (e.g. `0---9999`). |
| `symbols` | `none` \| `before` \| `after` \| `currency` | Where to show the currency or custom symbol. `currency` uses the fund's own symbol and position. |
| `symbols_print` | string | Literal symbol to use in the saved record (overrides the derived symbol). |

#### `drop-down`
Select from a fixed list.

| Extra key | Values | Description |
|-----------|--------|-------------|
| `drop-down` | `["value, Label", …]` \| `countries` \| `months` \| list name | Values inline, built-in lists, or a named list from `drop-down-lists:`. |
| `dropdown_search` | `yes` \| `no` | Replace the `<select>` with a searchable text input. |

#### `random-generator`
Auto-generates a random 6-character alphanumeric reference on load; user can regenerate.

#### `reference`
Links to a person (beneficiary or donor) or a prior record.

| Extra key | Values | Description |
|-----------|--------|-------------|
| `reference` | `beneficiary` \| `donor` \| `donation` \| `disbursement` | Which type to look up. |
| `reference_limit` | `all-records` \| `same-id-only` | `same-id-only` restricts the pick list to records for the same person as another `reference` field on the same form. |

---

## Template tokens

In a `template:` string, `{fieldname}` is replaced with the saved value of
that field. Additional built-in tokens:

| Token | Value |
|-------|-------|
| `{logo}` | Embedded logo image (or blank if no logo). Place on its own line. |
| `{signatures}` | Signature block(s) (count from `signatures:`). |
| `{fund-name}` | Fund name from `fund.name`. |
| `{reference}` | The record's reference number. |
| `{amount}` | Formatted monetary amount with currency symbol. |
| `{donor-name}` / `{donor-id}` | Linked donor's name and ID (donation forms). |
| `{beneficiary-name}` / `{beneficiary-id}` | Linked beneficiary's name and ID (disbursement forms). |

---

## `languages:`

```yaml
languages:
  - code: en
    name: English
  - code: th
    name: ภาษาไทย
    direction: rtl        # ltr (default) | rtl
    line-height: 1.9
    font: "bunny:Sarabun:300,400"
```

When two or more languages are defined, a language selector appears on each
money form and the generated document uses the selected language's template and
signature labels (if per-language variants are defined).

| Key | Description |
|-----|-------------|
| `code` | Language code used as the key in per-language template maps. |
| `name` | Display name shown in the language selector. |
| `direction` | `ltr` (default) or `rtl` for right-to-left scripts. |
| `line-height` | Override `documents.line-height` for this language. |
| `font` | Override `documents.font-body` for this language. |

---

## `drop-down-lists:`

Named lists reusable across multiple drop-down fields.

```yaml
drop-down-lists:
  - name: channels
    values: ["cash, Cash", "bank-transfer, Bank transfer"]
```

Each entry in `values` is `"stored-value, Display Label"`.
