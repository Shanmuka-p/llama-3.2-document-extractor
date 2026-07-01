# Purchase Order (PO) JSON Schema — Binding Specification

> **Status: BINDING** — Every PO training example in `data/curated_train.jsonl` and every PO evaluation must conform to this schema exactly. Any deviation is a curation error.

---

## Overview

This schema defines the canonical machine-readable structure for **Purchase Order (PO) documents**. A purchase order is issued by a **buyer** to a **supplier** to authorise the purchase of goods or services. The model must return a single, top-level JSON object with exactly these keys — no additional keys, no prose, no markdown code fences.

---

## Field-Level Specification

| Key | Type | Format | Description | If Field is Absent in Document |
|:---|:---|:---|:---|:---|
| `buyer` | `string` | Free text | Full legal name of the purchasing organisation (e.g., `"Mahindra & Mahindra Ltd."`) | `null` |
| `supplier` | `string` | Free text | Full legal name of the supplying organisation (e.g., `"Bosch India Pvt. Ltd."`) | `null` |
| `po_number` | `string` | Free text | Unique Purchase Order reference number (e.g., `"PO-2024-00159"`) | `null` |
| `date` | `string` | `YYYY-MM-DD` | Date the PO was issued. Must be ISO 8601. | `null` |
| `delivery_date` | `string` | `YYYY-MM-DD` | Expected or requested delivery date. Same format as `date`. | `null` |
| `currency` | `string` | ISO 4217, 3-letter uppercase | Currency code (e.g., `"USD"`, `"EUR"`, `"GBP"`, `"INR"`, `"JPY"`) | `null` |
| `total` | `float` | Numeric, 2 decimal places | Total authorised purchase amount. Must be a JSON number. | `0.0` |
| `items` | `array` | Array of objects | One entry per ordered item or service. Empty array `[]` if none listed. | `[]` |

### `items` Object Sub-Schema

Each element of the `items` array must be a JSON object with exactly these three keys:

| Sub-Key | Type | Description | If Absent |
|:---|:---|:---|:---|
| `item_name` | `string` | Name or description of the item/service being purchased | `""` (empty string) |
| `quantity` | `integer` | Number of units ordered. Must be a JSON integer. | `1` |
| `unit_price` | `float` | Agreed price per single unit. Must be a JSON number. | `0.0` |

---

## Canonical Example Output

```json
{
  "buyer": "Mahindra & Mahindra Ltd.",
  "supplier": "Bosch India Pvt. Ltd.",
  "po_number": "PO-2024-00159",
  "date": "2024-03-10",
  "delivery_date": "2024-04-15",
  "currency": "INR",
  "total": 485000.00,
  "items": [
    {"item_name": "ABS Sensor Module — Model BS6", "quantity": 200, "unit_price": 1500.00},
    {"item_name": "Electronic Control Unit (ECU) Rev. 3", "quantity": 50, "unit_price": 5500.00},
    {"item_name": "Wiring Harness Assembly — Type D", "quantity": 100, "unit_price": 850.00}
  ]
}
```

---

## Absent Field Examples

- **No delivery date on PO**: `"delivery_date": null` — Do NOT omit the key.
- **Supplier name not listed**: `"supplier": null` — Do NOT hallucinate a name.
- **No items breakdown**: `"items": []` — Return an empty array, not `null`.
- **PO number absent**: `"po_number": null` — Do NOT invent a number.

---

## Key Distinction from Invoice Schema

| Dimension | Invoice | Purchase Order |
|:---|:---|:---|
| Issuer | `vendor` | `supplier` |
| Recipient | *(implicit — the buyer)* | `buyer` |
| Item array key | `line_items` | `items` |
| Item description key | `description` | `item_name` |
| Tax field | `tax` (present) | *(absent — POs are pre-tax authorisations)* |
| Subtotal field | `subtotal` (present) | *(absent — only `total` is authorised)* |

> **Critical**: Do not confuse `line_items` (invoice) with `items` (PO). Using the wrong array key name is a schema violation.

---

## Enforcement Rules

1. **All 8 top-level keys must always be present** — even when their value is `null` or `0.0`. Missing any key = `has_all_required_keys = false` in evaluation.
2. **Dates must always be `YYYY-MM-DD` strings.** `"10/03/2024"` is a schema violation.
3. **`total` and `unit_price` must be JSON numbers**, not strings. `"total": "485000.00"` is wrong.
4. **`quantity` must be a JSON integer.** `200.0` is wrong; `200` is correct.
5. **`currency` must be exactly 3 uppercase letters.**
6. **No prose before or after the JSON.** No markdown code fences. Raw JSON only.

---

## Rationale

Purchase orders function as legally binding procurement documents in enterprise systems. The schema deliberately excludes `tax` and `subtotal` because POs authorise spend — they do not calculate tax (invoices do). Including these fields in PO outputs would cause false negatives during schema validation in production ERP integrations (e.g., SAP Ariba, Oracle Procurement).
