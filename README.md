# Llama 3.2 Structured Document Extractor — Fine-Tuning Pipeline

This repository contains the dataset, schemas, training configurations, evaluation scripts, and analysis reports for fine-tuning the **Llama 3.2 3B Instruct** model. The goal is to reliably extract structured JSON data from unstructured business documents (invoices and purchase orders) for seamless integration into production ERP and CRM data pipelines.

---

## 📁 Repository Structure

```
├── schema/
│   ├── invoice_schema.md      # Binding JSON schema specification for Invoices
│   └── po_schema.md           # Binding JSON schema specification for Purchase Orders
├── data/
│   ├── curated_train.jsonl    # Curated dataset of 80 training examples (50 Invoices, 30 POs)
│   └── curation_log.md        # Curation decisions and data layout audit logs
├── eval/
│   ├── documents/             # 20 held-out test documents (10 Invoices, 10 POs) and ground truths
│   ├── failures/              # Detailed analysis of 5 fine-tuned model failure cases (Case 01 - 05)
│   ├── baseline_responses.md  # Verbatim baseline model outputs
│   ├── baseline_scores.csv    # Evaluated scores for the baseline model
│   ├── finetuned_responses.md # Verbatim fine-tuned model outputs
│   ├── finetuned_scores.csv   # Evaluated scores for the fine-tuned model
│   ├── summary.md             # Baseline evaluation metrics summary
│   └── before_vs_after.md     # Side-by-side performance comparison
├── prompts/
│   ├── prompt_iterations.md   # Documentation of 3 engineered prompt versions
│   └── prompt_eval.md         # Evaluated results of prompt iterations on worst baseline documents
├── screenshots/
│   ├── training_config.png    # Gradio configuration panel screenshot
│   └── loss_curve.png         # Model training loss curve plot
├── training_config.md         # Training hyperparameters and parameter justifications
└── report.md                  # Comprehensive project report (Prompting vs. Fine-Tuning)
```

---

## 📊 Before vs. After Results

Our evaluation on the 20 held-out test documents shows a significant improvement in the model's structure reliability and parse success rates.

| Metric | Baseline (Pre-trained Model) | Post Fine-Tuning |
| :--- | :---: | :---: |
| **Parse Success Rate** | **10.0%** (2/20) | **75.0%** (15/20) |
| **Avg Key Accuracy** | **52.2%** | **100.0%** |
| **Avg Value Accuracy** | **51.2%** | **95.6%** |
| **Markdown Code Fences** | 7 | 0 |
| **Conversational Preamble** | 4 | 0 |
| **Wrong Schema Keys** | 8 | 0 |

- **Formatting Constraints**: The fine-tuned model enforces JSON format directly, completely eliminating markdown code fences, preambles, and camelCase keys.
- **Data-centric failures**: The remaining 5 failures in the fine-tuned model were value accuracy issues (e.g. math errors, ambiguous date styles, or OCR text noise) rather than formatting issues. Detailed failure cases are documented in `eval/failures/`.

---

## 🚀 Hyperparameters & LoRA Config

We performed parameter-efficient Supervised Fine-Tuning (SFT) using the **LlamaFactory** UI with the following hyperparameters:

- **Method**: LoRA (Low-Rank Adaptation)
- **Rank ($r$)**: `16`
- **Alpha ($\alpha$)**: `32`
- **Learning Rate**: `2e-4` (Cosine decay scheduler)
- **Epochs**: `3`
- **Batch Size**: `2` (Gradient accumulation: 4)

Refer to [training_config.md](training_config.md) for full details and justifications of these selections.
