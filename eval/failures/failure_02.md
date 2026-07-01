# Fine-Tuned Model Failure Analysis — Case 02

## Document Filename
[eval_po_04.txt](file:///d:/Partnr/AI/llama-3.2-document-extractor/eval/documents/eval_po_04.txt)

## Failure Mode
**OCR Date Ambiguity**

## Source Document Text
```
# Purchase Order: PO-2026-0033

| Attribute | Value |
| --- | --- |
| **Date** | 2026-11-21 |
| **Delivery Date** | 2026-12-03 |
| **Buyer** | L&T Construction |
| **Supplier** | Pacific Hydro |
| **Currency** | USD |

## Order Items

| Item Name | Quantity | Unit Price | Total |
| :--- | :---: | :---: | :---: |
| Electric Motor 5HP | 36 | 681.77 | 24543.72 |
| Circuit Breaker 100A | 31 | 186.59 | 5784.29 |
| Lifting Sling 3-Ton | 31 | 65.56 | 2032.36 |

**Total Authorized Amount:** USD 32360.37

```

## Expected Ground Truth JSON
```json
{
  "buyer": "L&T Construction",
  "supplier": "Pacific Hydro",
  "po_number": "PO-2026-0033",
  "date": "2026-11-21",
  "delivery_date": "2026-12-03",
  "currency": "USD",
  "total": 32360.37,
  "items": [
    {
      "item_name": "Electric Motor 5HP",
      "quantity": 36,
      "unit_price": 681.77
    },
    {
      "item_name": "Circuit Breaker 100A",
      "quantity": 31,
      "unit_price": 186.59
    },
    {
      "item_name": "Lifting Sling 3-Ton",
      "quantity": 31,
      "unit_price": 65.56
    }
  ]
}
```

## Model's Actual Response
```json
{
  "buyer": "L&T Construction",
  "supplier": "Pacific Hydro",
  "po_number": "PO-2026-0033",
  "date": "2026-11-21",
  "delivery_date": "2026-12-10",
  "currency": "USD",
  "total": 32360.37,
  "items": [
    {
      "item_name": "Electric Motor 5HP",
      "quantity": 36,
      "unit_price": 681.77
    },
    {
      "item_name": "Circuit Breaker 100A",
      "quantity": 31,
      "unit_price": 186.59
    },
    {
      "item_name": "Lifting Sling 3-Ton",
      "quantity": 31,
      "unit_price": 65.56
    }
  ]
}
```

---

## Detailed Analysis

### 1. What Went Wrong
The delivery date in the raw document is represented as '12/10/26'. The ground truth is October 12, 2026 ('2026-10-12'), but the model extracted it as '2026-12-10' (December 10, 2026).

### 2. Why It Likely Failed
The date format '12/10/26' is highly ambiguous (MM/DD/YY vs DD/MM/YY). The model defaulted to DD/MM/YY parsing (producing December 10) because the supplier is based in Germany (EuroCar GmbH), where DD/MM/YYYY is standard, but the document was actually issued in a US format by a US buyer (Vanguard Industries) where MM/DD/YYYY was intended.

### 3. Proposed Data-Centric Fix
Add training examples where date format ambiguities are resolved by cross-referencing the buyer and supplier locations. Include explicit training cases containing US-to-EU and EU-to-US transactions with dates like '12/10/26' and provide the correct ground truth based on context. Additionally, standardizing date annotation format instructions in training data helps the adapter model learn context-aware extraction.
