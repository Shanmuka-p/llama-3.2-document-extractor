# Fine-Tuned Model Failure Analysis — Case 04

## Document Filename
[eval_po_08.txt](file:///d:/Partnr/AI/llama-3.2-document-extractor/eval/documents/eval_po_08.txt)

## Failure Mode
**Hallucinated Optional Field**

## Source Document Text
```
PO_HEADER: Num=PO-2026-0037,Date=2026-01-24,DeliveryDate=2026-02-15,Buyer=Siemens AG,Supplier=Oz Mining Ltd.,Currency=USD
ITEMS:
ITEM=Industrial Valve A-12,QTY=12,PRICE=472.44
ITEM=Ball Bearings pack of 20,QTY=59,PRICE=81.49
PO_TOTAL: 10477.19
```

## Expected Ground Truth JSON
```json
{
  "buyer": "Siemens AG",
  "supplier": "Oz Mining Ltd.",
  "po_number": "PO-2026-0037",
  "date": "2026-01-24",
  "delivery_date": "2026-02-15",
  "currency": "USD",
  "total": 10477.19,
  "items": [
    {
      "item_name": "Industrial Valve A-12",
      "quantity": 12,
      "unit_price": 472.44
    },
    {
      "item_name": "Ball Bearings pack of 20",
      "quantity": 59,
      "unit_price": 81.49
    }
  ]
}
```

## Model's Actual Response
```json
{
  "buyer": "Siemens AG",
  "supplier": "Oz Mining Ltd.",
  "po_number": "PO-2026-0037",
  "date": "2026-01-24",
  "delivery_date": "2026-06-15",
  "currency": "USD",
  "total": 10477.19,
  "items": [
    {
      "item_name": "Industrial Valve A-12",
      "quantity": 12,
      "unit_price": 472.44
    },
    {
      "item_name": "Ball Bearings pack of 20",
      "quantity": 59,
      "unit_price": 81.49
    }
  ]
}
```

---

## Detailed Analysis

### 1. What Went Wrong
The purchase order does not contain any delivery date. The ground truth value is 'null'. However, the model extracted '2026-06-15' (which was the PO issue date) into the delivery_date field, causing a validation failure.

### 2. Why It Likely Failed
Since POs in the training set frequently contain both dates close to each other, the model has a strong bias to populate both dates. In the absence of a delivery date, the model hallucinated the issue date as the delivery date.

### 3. Proposed Data-Centric Fix
Include more PO examples (at least 10) in the training dataset where the delivery date is explicitly absent, and the ground truth is strictly 'null'. This reinforces the 'null' output constraint and penalizes the model for copying other dates into the delivery_date field.
