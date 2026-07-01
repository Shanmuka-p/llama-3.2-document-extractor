# Invoice JSON Schema — Binding Specification

> **Status: BINDING** — Every training example in `data/curated_train.jsonl` and every model evaluation must conform to this schema exactly. Any deviation is a curation error.

---

## Overview

This schema defines the canonical machine-readable structure for **invoice documents**. The model must return a single, top-level JSON object with exactly these keys — no additional keys, no prose, no markdown code fences.

---

## Field-Level Specification

| Key | Type | Format | Description | If Field is Absent in Document |
|:---|:---|:---|:---|:---|
| `vendor` | `string` | Free text | Full legal name of the issuing company (e.g., `"Tata Steel Ltd."`) | `null` |
| `invoice_number` | `string` | Free text | Unique identifier assigned by the vendor (e.g., `"INV-2024-00421"`) | `null` |
| `date` | `string` | `YYYY-MM-DD` | Invoice issue date. Must be ISO 8601. Never use `DD/MM/YYYY` or `MM-DD-YYYY`. | `null` |
| `due_date` | `string` | `YYYY-MM-DD` | Payment deadline. Same format as `date`. | `null` |
| `currency` | `string` | ISO 4217, 3-letter uppercase | Currency code (e.g., `"USD"`, `"EUR"`, `"INR"`, `"GBP"`, `"JPY"`) | `null` |
| `subtotal` | `float` | Numeric, 2 decimal places | Pre-tax sum of all line items. Must be a JSON number, **not** a string. | `0.0` |
| `tax` | `float` | Numeric, 2 decimal places | Total tax amount applied (GST, VAT, HST, etc.). Must be a JSON number. | `null` |
| `total` | `float` | Numeric, 2 decimal places | Final amount due (`subtotal + tax`). Must be a JSON number. | `0.0` |
| `line_items` | `array` | Array of objects | One entry per billable line. Empty array `[]` if no line items listed. | `[]` |

### `line_items` Object Sub-Schema

Each element of the `line_items` array must be a JSON object with exactly these three keys:

| Sub-Key | Type | Description | If Absent |
|:---|:---|:---|:---|
| `description` | `string` | Human-readable item name or service description | `""` (empty string) |
| `quantity` | `integer` | Unit count. Must be a JSON integer (not a float like `1.0`, not a string). | `1` |
| `unit_price` | `float` | Price per single unit. Must be a JSON number. | `0.0` |

---

## Canonical Example Output

```json
{
  "vendor": "Aditya Tech Solutions Pvt. Ltd.",
  "invoice_number": "INV-2026-00841",
  "date": "2026-04-03",
  "due_date": "2026-05-03",
  "currency": "INR",
  "subtotal": 15000.00,
  "tax": 2700.00,
  "total": 17700.00,
  "line_items": [
    {"description": "Llama 3.2 Fine-Tuning Credits — 100hr GPU", "quantity": 2, "unit_price": 5000.00},
    {"description": "Dataset Curation Service", "quantity": 1, "unit_price": 5000.00}
  ]
}
```

---

## Absent Field Examples

- **No tax on the invoice**: `"tax": null` — Do NOT omit the key.
- **No due date on the invoice**: `"due_date": null` — Do NOT omit the key.
- **No vendor name legible**: `"vendor": null` — Do NOT hallucinate a name.
- **No line items listed**: `"line_items": []` — Return an empty array, not `null`.
- **No subtotal listed**: `"subtotal": 0.0` — Use `0.0`, not `null`.

---

## Enforcement Rules

1. **All 9 top-level keys must always be present** — even if their value is `null` or `0.0`. A response missing any key has `has_all_required_keys = false` in evaluation.
2. **Dates must always be `YYYY-MM-DD` strings.** A date of `"03/04/2026"` is a schema violation.
3. **`subtotal`, `tax`, `total`, `quantity`, `unit_price` must always be JSON numbers**, never strings. `"total": "17700.00"` is a schema violation.
4. **`quantity` must be a JSON integer.** `1.0` is wrong; `1` is correct.
5. **`currency` must be exactly 3 uppercase letters.** `"USD"` is correct; `"US Dollars"` is wrong.
6. **No prose before or after the JSON.** No markdown code fences (` ``` `). The model output, from character 1 to the last character, must be parseable by `json.loads()`.

---

## Rationale

These rules are designed to maximise downstream parseability in a production pipeline. A single extra character (a backtick, a trailing comma) causes `json.loads()` to raise an exception, requiring human intervention. Consistency in types (float vs string for totals, int vs float for quantity) enables schema validation without type-coercion logic in the consumer.
