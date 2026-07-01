# Before vs. After Evaluation Comparison

Comparison of the pre-trained Llama 3.2 3B Instruct model (Baseline) vs. the LoRA Fine-Tuned model on the 20 held-out test documents.

| Metric | Baseline (Base Model) | Post Fine-Tuning |
| :--- | :---: | :---: |
| **Parse Success Rate** | 30.0% (6/20) | 100.0% (20/20) |
| **Avg Key Accuracy** | 54.8% | 100.0% |
| **Avg Value Accuracy** | 51.4% | 98.2% |
| **Responses with Markdown Fences** | 6 | 0 |
| **Responses with Prose Preamble** | 3 | 0 |
| **Responses with Wrong Schema Keys** | 3 | 0 |

## Key Findings

1. **Format Enforcement**: Fine-tuning updated the weights of the model so that JSON formatting became a hard constraint. No markdown fences or prose preambles were produced by the fine-tuned model.
2. **Schema Alignment**: The fine-tuned model consistently output the exact keys defined in the schema (e.g., snake_case, required fields as `null` or empty arrays) instead of generating camelCase or omitting fields.
3. **Parse success rate** increased dramatically from **30.0%** to **100.0%**, showing that parameter-efficient fine-tuning (LoRA) is highly effective for converting an instructional model into a reliable structured-output pipeline extractor.
