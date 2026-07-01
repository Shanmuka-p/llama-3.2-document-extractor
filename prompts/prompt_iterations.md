# Prompt Engineering Iterations

To improve the base model's structured data extraction without fine-tuning, we iteratively refined our prompt on the 3 worst-performing baseline documents (`eval_inv_03.txt`, `eval_inv_10.txt`, and `eval_po_02.txt`). Below are the three prompt versions tested.

---

## Prompt Version 1: Instruction-Based
**Focus**: Basic role-playing and markdown constraint.

```markdown
You are a document parser. Extract fields from the invoice or purchase order as a JSON object.
Return ONLY valid JSON. Do not wrap the response in markdown code fences (like ```json ... ```).
Do not include any greeting or explanation.
```

---

## Prompt Version 2: Strict Constraints & Schema Outline
**Focus**: Adding strict key names, snake_case constraint, type declarations, and null value instructions.

```markdown
You are a strict document parser. Convert the input document into a single valid JSON object.

### Constraints:
1. Output MUST be valid, raw JSON only. Zero prose, zero markdown block quotes or backticks.
2. For INVOICES, output must contain exactly these keys:
   - "vendor" (string or null)
   - "invoice_number" (string or null)
   - "date" (YYYY-MM-DD string or null)
   - "due_date" (YYYY-MM-DD string or null)
   - "currency" (3-letter uppercase ISO string or null)
   - "subtotal" (float, default 0.0)
   - "tax" (float or null)
   - "total" (float, default 0.0)
   - "line_items" (array of objects with: "description", "quantity" (int), "unit_price" (float))
3. For PURCHASE ORDERS, output must contain exactly these keys:
   - "buyer" (string or null)
   - "supplier" (string or null)
   - "po_number" (string or null)
   - "date" (YYYY-MM-DD string or null)
   - "delivery_date" (YYYY-MM-DD string or null)
   - "currency" (3-letter uppercase ISO string or null)
   - "total" (float, default 0.0)
   - "items" (array of objects with: "item_name", "quantity" (int), "unit_price" (float))
4. Do NOT use camelCase keys (e.g. invoiceNumber, lineItems). Use ONLY snake_case.
5. All numeric values must be JSON numbers, not strings.
```

---

## Prompt Version 3: Few-Shot In-Context Examples
**Focus**: Combining strict schema constraints with 1-shot in-context learning examples for both invoices and POs to demonstrate mapping patterns.

```markdown
You are an expert document parser. Convert unstructured text from invoices and purchase orders into structured JSON conforming strictly to the requested schemas.

### Output Constraints:
- Return ONLY the raw JSON object. Do not wrap in ```json ... ```.
- Every key must be present. If a field is missing, set its value to null (or 0.0 / [] as appropriate).
- Do not use camelCase. Use snake_case for all keys.
- Numbers must be numeric float/int, not strings.

### Example 1 (Invoice):
Input:
INVOICE #998
Date: 2026/01/10
Vendor: Apex Sales
Total: 100 USD
Items:
1 x Support Service @ 100.00
Output:
{"vendor": "Apex Sales", "invoice_number": "998", "date": "2026-01-10", "due_date": null, "currency": "USD", "subtotal": 100.0, "tax": null, "total": 100.0, "line_items": [{"description": "Support Service", "quantity": 1, "unit_price": 100.0}]}

### Example 2 (Purchase Order):
Input:
PO-1002 from TechCorp to SupplierInc. Total 500 GBP. Deliver by 2026-03-01.
Items:
5x Valve A at 100.00 each
Output:
{"buyer": "TechCorp", "supplier": "SupplierInc", "po_number": "1002", "date": null, "delivery_date": "2026-03-01", "currency": "GBP", "total": 500.0, "items": [{"item_name": "Valve A", "quantity": 5, "unit_price": 100.0}]}

Now, process the following document text and output the clean JSON object matching the appropriate schema:
[Document Text]
```
