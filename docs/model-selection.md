# Model Selection

This document justifies the inclusion and exclusion of LLM guardrail models for the experiment evaluating the impact of quantization on multilingual safety performance (English vs. Brazilian Portuguese).

## Selection Criteria

Models were evaluated against four criteria:

1. **Multilingual support** — does the model declare or demonstrate support for Portuguese (PT-BR)?
2. **Task scope** — is the model designed for toxicity/content safety classification in production settings?
3. **Taxonomy compatibility** — is the model's safety taxonomy compatible enough to allow intra-model EN vs. PT comparison?
4. **Computational viability** — can the model be loaded and quantized on a single L40S GPU (48GB VRAM)?

The core methodological decision is to **never compare models against each other**, but to compare **each model against itself** across languages (EN vs. PT-BR) and quantization levels (16bit → 8bit → 4bit → 2bit). This isolates language and quantization as variables, avoiding confounds from differing taxonomies across models.

---

## Model Groups

Based on the selection process, models fall into two groups:

- **Fully multilingual** — base model and safety fine-tuning both include PT-BR
- **Hybrid** — base model is multilingual, but safety fine-tuning was done in English only

Both groups are intentional. Hybrid models serve as a natural reference point: if fully multilingual models converge toward hybrid model performance under aggressive quantization, this would indicate that quantization erases the investment in multilingual safety training.

---

## Included Models

### `meta-llama/Llama-Guard-3-8B`
- **Group:** Fully multilingual
- **Justification:** Explicitly supports PT among 8 languages (EN, FR, DE, HI, IT, PT, ES, TH) in both base model and safety fine-tuning. Uses the MLCommons hazard taxonomy. Most cited guardrail in recent multilingual safety literature. Already deployed in the lab infrastructure.

### `meta-llama/Llama-Guard-3-1B`
- **Group:** Fully multilingual
- **Justification:** Same architecture and training pipeline as Llama-Guard-3-8B, but significantly smaller. Included to test whether model size amplifies the quantization × language interaction effect. The 1B/8B pair within the same family acts as a controlled size contrast, allowing isolation of parameter count as a variable independent of training differences.

### `nvidia/Llama-3.1-Nemotron-Safety-Guard-8B-v3`
- **Group:** Fully multilingual
- **Justification:** Explicitly supports PT-BR among 9+ languages, with strong zero-shot generalization to over 20 languages. Most downloaded NVIDIA guardrail model. Uses the Aegis 2.0 taxonomy, which — while different from MLCommons — does not affect intra-model EN vs. PT comparison. LoRA-tuned on Llama-3.1-8B-Instruct, making it architecturally distinct from Llama Guard 3 despite sharing the same base model family.

### `google/shieldgemma-9b`
- **Group:** Hybrid
- **Justification:** Built on Gemma 2 (multilingual base), but safety fine-tuning was conducted exclusively in English as documented in the ShieldGemma paper (Zeng et al., 2024). Selected over the 2B variant for better generalization on external benchmarks and higher community adoption (5,425 vs. 6,427 downloads/month, comparable; 27B excluded due to 136 downloads/month and marginal 1.7% AU-PRC gain). Represents a model where multilingual capability comes from the base, not from safety-specific training.

### `ibm-granite/granite-guardian-4.1-8b`
- **Group:** Hybrid
- **Justification:** Most recent IBM Granite Guardian release. Base model (Granite 4.1) explicitly supports PT among 12 languages. However, the Guardian safety fine-tuning paper only describes English training data, placing it in the hybrid group alongside ShieldGemma. Chosen over granite-guardian-3.2 variants as the most current release. Apache 2.0 license — no access approval required.

---

## Excluded Models

### `meta-llama/Prompt-Guard-86M`
- **Reason:** Different task. Designed for prompt injection and jailbreak detection, not toxicity/content safety classification. Out of scope.

### `meta-llama/LlamaGuard-7b`
- **Reason:** Original 2023 release based on Llama 2. English-only, no multilingual support. Superseded in all dimensions by Llama Guard 3.

### `meta-llama/Meta-Llama-Guard-2-8B`
- **Reason:** Based on Llama 3 but without declared multilingual support. Superseded by Llama Guard 3, which adds PT support and improved taxonomy alignment. Including both Guard 2 and Guard 3 would introduce redundancy without adding a new variable.

### `meta-llama/Llama-Guard-4-12B`
- **Reason:** Multimodal model (text + image). Per official Meta documentation, it is optimized for English-language text, and for other languages developers are expected to test and validate independently. Meta itself recommends Llama Guard 3 8B and 1B for multilingual text-only classification — which is this experiment's exact use case. Including Guard 4 would also create an imbalance with three Meta models against one from each other vendor.

### `google/shieldgemma-2b`
- **Reason:** Replaced by ShieldGemma 9B. Performance difference is small but meaningful (1.2% AU-PRC), and the 9B is more representative of real-world deployment patterns.

### `google/shieldgemma-27b`
- **Reason:** Only 136 downloads/month, indicating minimal community adoption. Marginal 1.7% AU-PRC gain over 9B does not justify the additional VRAM cost. Would likely be inviable alongside other models on a single L40S.

### `ibm-granite/granite-guardian-3.2-2b` and `granite-guardian-3.2-8b`
- **Reason:** Superseded by granite-guardian-4.1-8b, which is based on a newer and more capable multilingual base model.

### `allenai/wildguard`
- **Reason:** English-only (`language: en` declared in model card). Based on Mistral-7B-v0.3, which has limited multilingual coverage compared to other base models in this selection. Additionally, WildGuard's primary differentiator is refusal detection — a task outside this experiment's scope. Despite high adoption (79,054 downloads/month), the model is popular for reasons unrelated to the research question here.

### `OpenSafetyLab/MD-Judge-v0.1`
- **Reason:** Older version, superseded by v0.2.

### `OpenSafetyLab/MD-Judge-v0_2-internlm2_7b`
- **Reason:** MD-Judge is a benchmark evaluator, not a production guardrail. It was designed to assess other models using the SALAD-Bench taxonomy (6 domains, 16 tasks, 65 categories), which is incompatible with standard content safety datasets. Additionally, it was trained on English and Chinese only — PT is not in scope.

### `nvidia/llama-3.1-nemoguard-8b-content-safety`
- **Reason:** Replaced by the v3 multilingual variant (Llama-3.1-Nemotron-Safety-Guard-8B-v3), which explicitly supports PT-BR and achieves stronger multilingual performance.

---

## Summary Table

| Model | Group | Decision |
|---|---|---|
| `meta-llama/Llama-Guard-3-8B` | Fully multilingual | ✅ Included |
| `meta-llama/Llama-Guard-3-1B` | Fully multilingual | ✅ Included |
| `nvidia/Llama-3.1-Nemotron-Safety-Guard-8B-v3` | Fully multilingual | ✅ Included |
| `google/shieldgemma-9b` | Hybrid | ✅ Included |
| `ibm-granite/granite-guardian-4.1-8b` | Hybrid | ✅ Included |
| `meta-llama/Prompt-Guard-86M` | — | ❌ Different task |
| `meta-llama/LlamaGuard-7b` | — | ❌ Obsolete, EN-only |
| `meta-llama/Meta-Llama-Guard-2-8B` | — | ❌ Superseded by Guard 3 |
| `meta-llama/Llama-Guard-4-12B` | — | ❌ Multimodal, EN-optimized |
| `google/shieldgemma-2b` | — | ❌ Replaced by 9B |
| `google/shieldgemma-27b` | — | ❌ Negligible adoption, marginal gain |
| `ibm-granite/granite-guardian-3.2-2b` | — | ❌ Superseded by 4.1 |
| `ibm-granite/granite-guardian-3.2-8b` | — | ❌ Superseded by 4.1 |
| `allenai/wildguard` | — | ❌ EN-only, refusal-focused |
| `OpenSafetyLab/MD-Judge-v0.1` | — | ❌ Obsolete |
| `OpenSafetyLab/MD-Judge-v0_2-internlm2_7b` | — | ❌ Benchmark evaluator, EN/ZH only |
| `nvidia/llama-3.1-nemoguard-8b-content-safety` | — | ❌ Replaced by v3 |