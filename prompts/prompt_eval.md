# Prompt Engineering Evaluation Results

We evaluated the performance of the three prompt iterations on the 3 worst-performing baseline documents from Part C:
1. `eval_inv_03.txt` (Invoice)
2. `eval_inv_10.txt` (Invoice)
3. `eval_po_02.txt` (Purchase Order)

Below are the detailed evaluations of each prompt version compared against the Fine-Tuned (FT) model.

---

## 1. Prompt Version 1: Instruction-Based

| Document | Is Valid JSON? | Has Required Keys? | Key Accuracy | Value Accuracy | Formatting Issues / Failures |
| :--- | :---: | :---: | :---: | :---: | :--- |
| `eval_inv_03.txt` | True | False | 0.00 (camelCase) | 0.00 | Output used camelCase keys (`invoiceNumber`, `dueDate`, `lineItems`). |
| `eval_inv_10.txt` | False | False | 0.00 | 0.00 | Wrapped JSON in markdown fences (` ```json `), failing direct parsing. |
| `eval_po_02.txt` | True | False | 0.00 (camelCase) | 0.00 | Output used camelCase keys (`poNumber`, `deliveryDate`, `items`). |

- **Parse Success Rate**: **0.0%** (0 / 3)
- **Key Takeaway**: Basic instructions are ignored by the base model; it reverts to standard code fences and camelCase defaults.

---

## 2. Prompt Version 2: Strict Constraints & Schema Outline

| Document | Is Valid JSON? | Has Required Keys? | Key Accuracy | Value Accuracy | Formatting Issues / Failures |
| :--- | :---: | :---: | :---: | :---: | :--- |
| `eval_inv_03.txt` | True | True | 1.00 (snake_case) | 1.00 | Extracted successfully. Snake_case keys enforced. |
| `eval_inv_10.txt` | False | False | 1.00 | 0.00 | Enforced snake_case keys, but wrapped output in markdown fences. |
| `eval_po_02.txt` | True | True | 1.00 (snake_case) | 1.00 | Extracted successfully. Snake_case keys enforced. |

- **Parse Success Rate**: **66.7%** (2 / 3)
- **Key Takeaway**: Specifying exact key names and types forces correct snake_case, but the model still fails to suppress markdown fences for `eval_inv_10.txt`.

---

## 3. Prompt Version 3: Few-Shot In-Context Examples

| Document | Is Valid JSON? | Has Required Keys? | Key Accuracy | Value Accuracy | Formatting Issues / Failures |
| :--- | :---: | :---: | :---: | :---: | :--- |
| `eval_inv_03.txt` | True | True | 1.00 | 1.00 | Parse success. Direct raw JSON. |
| `eval_inv_10.txt` | True | True | 1.00 | 1.00 | Parse success. Few-shot example successfully suppressed code fences. |
| `eval_po_02.txt` | True | True | 1.00 | 1.00 | Parse success. Direct raw JSON. |

- **Parse Success Rate**: **100.0%** (3 / 3)
- **Key Takeaway**: Few-shot templates provide structural examples that override conversational defaults and markdown fences, achieving a perfect format match on these 3 documents.

---

## 4. Prompt Version 3 (Base Model) vs. Fine-Tuned Model

On these specific 3 documents:
- **Base Model (Prompt Version 3)**:
  - Parse Success Rate: **100.0%** (3 / 3)
  - Key Accuracy: **100.0%**
  - Value Accuracy: **100.0%**
- **Fine-Tuned Model**:
  - Parse Success Rate: **66.7%** (2 / 3) - `eval_inv_03.txt` failed due to multi-currency confusion (extracted currency JPY as USD).
  - Key Accuracy: **100.0%**
  - Value Accuracy: **66.7%** (due to the value accuracy issue on currency in `eval_inv_03.txt`).

*Note: While few-shot prompting achieves a perfect score here, it consumes high input context lengths (~1,000 tokens per call), increasing cost and latency compared to the fine-tuned model (which achieves strict formatting with a minimal 50-token instruction prompt).*
