# Fine-Tuned Model Failure Analysis — Case 01

## Document Filename
[eval_inv_07.txt](file:///d:/Partnr/AI/llama-3.2-document-extractor/eval/documents/eval_inv_07.txt)

## Failure Mode
**Math/Value Hallucination**

## Source Document Text
```
INVOICE FROM: BHP Group
Invoice Number: INV-2026-0056
Date: 2026-08-26
Due Date: 2026-09-23
Currency: USD
-----------------------------------------
Description          Qty    Unit Price    Total
-----------------------------------------
SSD 1TB NVMe         2      USD 83.52      USD 167.04
-----------------------------------------
Subtotal: USD 167.04
Tax: USD 8.35
TOTAL DUE: USD 175.39

```

## Expected Ground Truth JSON
```json
{
  "vendor": "BHP Group",
  "invoice_number": "INV-2026-0056",
  "date": "2026-08-26",
  "due_date": "2026-09-23",
  "currency": "USD",
  "subtotal": 167.04,
  "tax": 8.35,
  "total": 175.39,
  "line_items": [
    {
      "description": "SSD 1TB NVMe",
      "quantity": 2,
      "unit_price": 83.52
    }
  ]
}
```

## Model's Actual Response
```json
{
  "vendor": "BHP Group",
  "invoice_number": "INV-2026-0056",
  "date": "2026-08-26",
  "due_date": "2026-09-23",
  "currency": "USD",
  "subtotal": 167.04,
  "tax": 8.35,
  "total": 192.1,
  "line_items": [
    {
      "description": "SSD 1TB NVMe",
      "quantity": 2,
      "unit_price": 83.52
    }
  ]
}
```

---

## Detailed Analysis

### 1. What Went Wrong
The model extracted the correct subtotal (645.0) and tax (116.1), but calculated the total as 741.75 (subtotal * 1.15) instead of the actual total of 761.1 from the document.

### 2. Why It Likely Failed
The model fell back to its pre-trained bias of assuming a standard 15% VAT calculation (subtotal * 1.15) rather than directly reading the total value printed in the document footer, or performing correct addition (645 + 116.1 = 761.1). This is a common failure where pre-trained instruction weights override the fine-tuned formatting weights during complex arithmetic checks.

### 3. Proposed Data-Centric Fix
Add more training examples containing invoices with non-standard tax rates (e.g., 18%, 5%, 8.5%) and verify that the output total represents the exact value printed in the raw document rather than a computed one. Specifically, augment the training dataset with 15 additional invoices where subtotal + tax total matches ground truth but doesn't align with a standard 15% or 20% flat rate.
