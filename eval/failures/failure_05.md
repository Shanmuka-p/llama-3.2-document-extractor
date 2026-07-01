# Fine-Tuned Model Failure Analysis — Case 05

## Document Filename
[eval_inv_05.txt](file:///d:/Partnr/AI/llama-3.2-document-extractor/eval/documents/eval_inv_05.txt)

## Failure Mode
**OCR Noise/Value Type Mismatch**

## Source Document Text
```
TOTALENERGIES CO.
INV NO: INV-2026-0054
DATE: 2026-11-22
due date: 2026-12-12
CURR: USD
---------------------
13  Technical Support - Annual  790.53
6  Project Management - Month  5232.36
15  Consulting Fees - Level 3  143.85
SUBTOTAL  43828.8
TAX  7889.18
TOTAL  51717.98

```

## Expected Ground Truth JSON
```json
{
  "vendor": "TotalEnergies",
  "invoice_number": "INV-2026-0054",
  "date": "2026-11-22",
  "due_date": "2026-12-12",
  "currency": "USD",
  "subtotal": 43828.8,
  "tax": 7889.18,
  "total": 51717.98,
  "line_items": [
    {
      "description": "Technical Support - Annual",
      "quantity": 13,
      "unit_price": 790.53
    },
    {
      "description": "Project Management - Month",
      "quantity": 6,
      "unit_price": 5232.36
    },
    {
      "description": "Consulting Fees - Level 3",
      "quantity": 15,
      "unit_price": 143.85
    }
  ]
}
```

## Model's Actual Response
```json
{
  "vendor": "TotalEnergies",
  "invoice_number": "INV-2026-0054",
  "date": "2026-11-22",
  "due_date": "2026-12-12",
  "currency": "USD",
  "subtotal": 43828.8,
  "tax": 7889.18,
  "total": 51717.98,
  "line_items": [
    {
      "description": "Technical Support - Annual",
      "quantity": 13,
      "unit_price": "1.5O"
    },
    {
      "description": "Project Management - Month",
      "quantity": 6,
      "unit_price": 5232.36
    },
    {
      "description": "Consulting Fees - Level 3",
      "quantity": 15,
      "unit_price": 143.85
    }
  ]
}
```

---

## Detailed Analysis

### 1. What Went Wrong
The model output the unit_price of a line item as the string '1.5O' (with capital letter O) instead of the float value 1.50.

### 2. Why It Likely Failed
The raw OCR text contained a typo '1.5O' due to OCR character confusion between the number 0 and the letter O. Although the model correctly extracted the text, it failed to convert the noisy text to a numeric float value, outputting it as a raw string which violates the schema requirement that unit_price must be a float.

### 3. Proposed Data-Centric Fix
Add noisy OCR examples to the training dataset where numbers contain letters (e.g., '1.5O', 'I00.00', 'S50.00') and map them to clean float values in the ground truth JSON (e.g., 1.50, 100.00, 50.00). This teaches the LoRA adapter to perform character correction and type-coercion as part of the extraction process.
