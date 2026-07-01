# Baseline Evaluation Summary

Established the performance metrics of the pre-trained **Llama 3.2 3B Instruct** model using our best engineered baseline extraction prompt.

## Baseline Metrics (Held-Out Test Set of 20 Documents)

- **Baseline Parse Success Rate**: 30.0%
- **Average Key Accuracy**: 54.8%
- **Average Value Accuracy**: 51.4%

The base model struggle with formatting constraints, frequently wrapping outputs in markdown code fences or including conversational preambles, which completely breaks direct JSON parsing and results in a low baseline parse success rate.
