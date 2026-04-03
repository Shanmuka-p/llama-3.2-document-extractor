# Invoice JSON Schema

This document defines the strict target structure for invoice data extraction. Every model response must be a single JSON object following this format.

| Key              | Type   | Description                                                                                 | If Absent |
| :--------------- | :----- | :------------------------------------------------------------------------------------------ | :-------- |
| `vendor`         | string | Full legal name of the company issuing the invoice.                                         | `null`    |
| `invoice_number` | string | Unique identifier for the invoice.                                                          | `null`    |
| `date`           | string | Date of issue in `YYYY-MM-DD` format.                                                       | `null`    |
| `due_date`       | string | Payment deadline in `YYYY-MM-DD` format.                                                    | `null`    |
| `currency`       | string | 3-letter ISO currency code (e.g., USD, INR, EUR).                                           | `null`    |
| `subtotal`       | float  | Total amount before taxes.                                                                  | `0.0`     |
| `tax`            | float  | Total tax amount applied.                                                                   | `null`    |
| `total`          | float  | Final amount due (Subtotal + Tax).                                                          | `0.0`     |
| `line_items`     | array  | List of objects containing `description` (str), `quantity` (int), and `unit_price` (float). | `[]`      |

### Example Target Output:

{
"vendor": "Aditya Tech Solutions",
"invoice_number": "INV-2026-001",
"date": "2026-04-03",
"due_date": "2026-05-03",
"currency": "INR",
"subtotal": 1500.00,
"tax": 270.00,
"total": 1770.00,
"line_items": [
{ "description": "Llama 3.2 Training Credits", "quantity": 1, "unit_price": 1500.00 }
]
}
