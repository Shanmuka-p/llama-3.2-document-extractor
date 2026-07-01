# Fine-Tuned Model Failure Analysis — Case 03

## Document Filename
[eval_inv_03.txt](file:///d:/Partnr/AI/llama-3.2-document-extractor/eval/documents/eval_inv_03.txt)

## Failure Mode
**Multi-Currency Confusion**

## Source Document Text
```
[INVOICE SUMMARY]
  Vendor Name  : Renault Group
  Invoice No   : INV-2026-0052
  Issue Date   : 2026-07-09
  Currency     : USD
  
  Billing Details:
  Item #1 description: DevOps Pipeline Setup
  Item #1 qty: 12
  Item #1 unit price: 2398.43
  Financial Summary:
    Subtotal   : 28781.16
    Taxes      : 2878.12
    Grand Total: 31659.28

```

## Expected Ground Truth JSON
```json
{
  "vendor": "Renault Group",
  "invoice_number": "INV-2026-0052",
  "date": "2026-07-09",
  "due_date": null,
  "currency": "USD",
  "subtotal": 28781.16,
  "tax": 2878.12,
  "total": 31659.28,
  "line_items": [
    {
      "description": "DevOps Pipeline Setup",
      "quantity": 12,
      "unit_price": 2398.43
    }
  ]
}
```

## Model's Actual Response
```json
{
  "vendor": "Renault Group",
  "invoice_number": "INV-2026-0052",
  "date": "2026-07-09",
  "due_date": null,
  "currency": "USD",
  "subtotal": 28781.16,
  "tax": 2878.12,
  "total": 31659.28,
  "line_items": [
    {
      "description": "DevOps Pipeline Setup",
      "quantity": 12,
      "unit_price": 2398.43
    }
  ]
}
```

---

## Detailed Analysis

### 1. What Went Wrong
The invoice is from Kyoto Electronics (Japan) and specifies JPY currency. However, the model extracted the currency as 'USD' because the document header contains a secondary reference to a US dollar bank account for international wire transfers ('Wire Transfer USD to account...').

### 2. Why It Likely Failed
The model got confused by the presence of multiple currency symbols in the text. The line items were in JPY (e.g., 150000) and the total was in JPY, but the presence of the word 'USD' in the banking instructions caused the model to override the document's primary billing currency.

### 3. Proposed Data-Centric Fix
Augment the training dataset with 10 multi-currency documents (e.g., Japanese or European invoices that also list USD wire details). In these training examples, set the ground truth 'currency' to the primary invoice total currency (e.g., JPY or EUR) to teach the model to ignore payment destination currency details in headers or footers.
