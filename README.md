# guardrail-quantization-ptbr-eval

Evaluating the impact of quantization on LLM guardrail performance across English and Brazilian Portuguese.

---

## Overview

This repository contains the code, configurations, data, and LaTeX source for an undergraduate thesis (TCC) investigating whether quantization amplifies multilingual safety gaps in LLM guardrail models.

The central research question is:

> Does quantization disproportionately degrade guardrail performance on Brazilian Portuguese (PT-BR) compared to English (EN), and if so, at which quantization level does this divergence become significant?

---

## Experiment Design

Each guardrail model is evaluated against itself — never compared directly to other models — across two languages and three quantization levels. This isolates language and quantization as independent variables while avoiding confounds from differing safety taxonomies across models.

| Variable | Values |
|---|---|
| Models | 5 guardrail models (see below) |
| Languages | English (EN), Brazilian Portuguese (PT-BR) |
| Quantization levels | 16bit (baseline), 8bit, 4bit |
| Quantization method | AWQ via vLLM |
| Metrics | F1, Precision, Recall, ΔF1 (EN − PT) |

The **16bit inference on each model defines the ground truth labels** used to evaluate lower-precision versions.

---

## Selected Models

| Model | Group |
|---|---|
| `meta-llama/Llama-Guard-3-8B` | Fully multilingual |
| `meta-llama/Llama-Guard-3-1B` | Fully multilingual |
| `nvidia/Llama-3.1-Nemotron-Safety-Guard-8B-v3` | Fully multilingual |
| `google/shieldgemma-9b` | Hybrid (multilingual base, EN-only safety fine-tuning) |
| `ibm-granite/granite-guardian-4.1-8b` | Hybrid (multilingual base, EN-only safety fine-tuning) |

For full justification of model inclusion and exclusion, see [`docs/model-selection.md`](docs/model-selection.md).

---

## Repository Structure

```
guardrail-quantization-ptbr-eval/
├── latex/                  # Full thesis (TCC)
│   ├── main.tex
│   ├── chapters/
│   └── references.bib
├── data/
│   ├── raw/                # Original EN dataset
│   ├── translated/         # PT-BR translated dataset
│   └── processed/          # Ready for inference
├── scripts/
│   ├── translate.py        # EN → PT-BR translation
│   ├── inference.py        # Runs guardrail inference via vLLM
│   ├── evaluate.py         # Computes F1, precision, recall, ΔF1
│   └── plots.py            # Generates figures for the thesis
├── configs/
│   └── models.yaml         # Opera/vLLM model configurations
├── results/                # CSV outputs from inference runs
├── notebooks/              # Exploratory analysis
└── docs/
    ├── model-selection.md  # Model inclusion/exclusion justification
    └── quantization-method.md  # Quantization method justification
```

---

## Exploratory Extension: 2-bit Quantization

2-bit quantization is **not part of the main experiment** due to the lack of native vLLM support. Including it in the main pipeline would require a different toolchain (llama.cpp or AQLM), introducing a methodological inconsistency that would confound quantization level with quantization method.

However, 2-bit is planned as a **separate exploratory analysis** using llama.cpp/Ollama with GGUF format. Results will be reported with explicit caveats and in a separate section of the thesis, not alongside the main AWQ results.

This extension is scoped for TCC 2 (final thesis) and is not part of the TCC 1 proposal.

---

## Infrastructure

- **GPU:** NVIDIA L40S (48GB VRAM)
- **Inference engine:** vLLM via Opera platform
- **Quantization:** AWQ (Activation-aware Weight Quantization)
- **Backup compute:** RunPod (~$1/h for L40S instances)

---

## Commit Convention

This repository follows [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope): short description in English

# Types
feat      — new script, config, or document
fix       — bug fix or correction
docs      — changes to markdown documentation
results   — new experiment outputs
chore     — configs, dependencies, tooling
refactor  — code restructuring without behavior change

# Scopes
models, data, scripts, configs, results, docs, latex
```

Examples:
```
feat(scripts): add vLLM inference pipeline
docs(models): update model selection justification
results(llama-guard-3): add 8bit EN baseline results
chore(configs): add Opera yaml for ShieldGemma 9B
```

---

## Citation

> [Author]. *Evaluating the Impact of Quantization on LLM Guardrail Performance across English and Brazilian Portuguese*. Undergraduate Thesis (TCC), [Institution], [Year].

---

## License

[To be defined]