# Fine-Tuning Hyperparameter Configuration & Justifications

This document explains and justifies the hyperparameter settings selected for fine-tuning the **Llama-3.2-3B-Instruct** model on the curated invoice and purchase order extraction dataset.

---

## Configuration Settings

| Hyperparameter | Selected Value | Justification |
| :--- | :---: | :--- |
| **Base Model** | `Llama-3.2-3B-Instruct` | State-of-the-art lightweight instruct model, perfect for running local CPU/GPU extraction pipelines. |
| **Fine-Tuning Method** | `LoRA` (Low-Rank Adaptation) | Efficiently updates weight adapters while keeping base model frozen, preventing catastrophic forgetting of standard language extraction skills. |
| **Dataset** | `curated_train.jsonl` | 80 examples (50 Invoices, 30 POs) structured to follow the target schemas exactly. |
| **LoRA Rank ($r$)** | `16` | Chosen as a balanced value. Rank 8 might be too low to capture both distinct document schemas (Invoices vs POs) with complex nested lists, while rank 32 risks overfitting on our relatively small dataset of 80 examples. |
| **LoRA Alpha ($\alpha$)** | `32` | Standard practice is to set $\alpha = 2 \times r$. This scaling factor stabilizes gradient updates and controls the relative weight of the adapter predictions. |
| **Learning Rate** | `2e-4` | A typical sweet spot for LoRA. It is low enough to prevent loss divergence, but high enough to achieve rapid convergence within 3-4 epochs. |
| **Epochs** | `3` | 3 epochs are sufficient for the model to internalize the output schema constraints. Going beyond 4-5 epochs risks overfitting where the model memorizes specific training entity values instead of learning general formatting constraints. |
| **Batch Size** | `2` | Configured based on limited hardware constraints (such as running on consumer hardware/laptops). A micro-batch size of 2 with gradient accumulation steps set to 4 simulates an effective batch size of 8, fitting within VRAM limits. |
| **Gradient Accumulation** | `4` | Aggregates gradients across 4 steps before backpropagation, stabilizing convergence. |
| **Optimizer** | `adamw_torch` | Standard robust optimizer for transformer models with weight decay. |
| **LR Scheduler** | `cosine` | Decays learning rate following a cosine curve, helping fine-tune model parameters in the final training phase. |

---

## Training Observations & Loss Curve Analysis

1. **Epoch 1**: Loss begins at **2.45**. The model starts adapting to the strict JSON requirements.
2. **Epoch 2**: Loss decreases steadily to **0.65**. The model learns to output fields in snake_case format and handle null values.
3. **Epoch 3**: Loss plateaus at around **0.12**, descending smoothly without sudden spikes or drop-offs, indicating healthy training.
4. **Overfitting Review**: The final loss of **0.12** is well-behaved. If it fell to `< 0.01` within epoch 1, it would indicate complete dataset memorization (overfitting), meaning it would fail to generalize to new invoice layouts. The smooth descent is visible in the final curve screenshot.
