# Quantization Method

This document justifies the quantization approach adopted in this experiment and defines the scope of quantization levels evaluated.

---

## What is Quantization

During training, model weights are stored as high-precision floating-point numbers (typically BF16 or FP32). Quantization converts these weights to lower-precision representations, reducing model size and VRAM consumption at the cost of some information loss.

The core tradeoff:

```
BF16 weight: -3.14159265  (16 bits, high precision)
INT8 weight: -3.1          (8 bits, moderate loss)
INT4 weight: -3            (4 bits, significant loss)
INT2 weight: -             (2 bits, only 4 possible values)
```

The experiment uses quantization to simulate real-world deployment constraints, where models are compressed to reduce cost and increase throughput on production servers.

---

## Available Quantization Methods

| Method | Bits | Quality | vLLM Support | Notes |
|---|---|---|---|---|
| AWQ | 4bit, 8bit | ⭐⭐⭐ | ✅ Native | State of the art for 4bit |
| GPTQ | 4bit, 8bit | ⭐⭐ | ✅ Native | Widely available pre-quantized models |
| FP8 | 8bit | ⭐⭐⭐ | ✅ Native | L40S supports FP8 natively |
| BitsAndBytes | 4bit, 8bit | ⭐ | ⚠️ Partial | Simpler but less efficient |
| GGUF | 2bit–8bit | ⭐⭐ | ❌ Not native | Used by llama.cpp and Ollama |
| AQLM | 2bit | ⭐⭐ | ⚠️ Limited | Research-stage, limited tooling |
| QAT | any | ⭐⭐⭐⭐ | — | Requires retraining, out of scope |

---

## Selected Method: AWQ via vLLM

**AWQ (Activation-aware Weight Quantization)** was selected as the quantization method for the main experiment. The decision is based on three criteria:

**1. Production relevance** — AWQ is the standard method used in production server deployments where a GPU is available. The motivating scenario of this experiment is a team that deploys a guardrail on a server and quantizes it to reduce VRAM cost or increase throughput. AWQ represents this scenario accurately.

**2. Methodological consistency** — vLLM applies AWQ uniformly across all models using the same pipeline and parameters. This ensures that any observed performance differences across quantization levels are attributable to the compression itself, not to differences in tooling or quantization strategy between models.

**3. Infrastructure alignment** — the experiment runs on an L40S GPU via the Opera platform, which uses vLLM as its inference engine. AWQ is natively supported by vLLM without requiring pre-quantized model weights from external sources, allowing all five models to be quantized on-demand under identical conditions.

```yaml
# Opera/vLLM configuration example
quantization: awq
dtype: bfloat16
```

---

## Quantization Levels

The main experiment evaluates three quantization levels plus the full-precision baseline:

| Level | Description | Role |
|---|---|---|
| 16bit (BF16) | Full precision baseline | Ground truth / gabarito |
| 8bit (INT8) | Moderate compression | First degradation checkpoint |
| 4bit (INT4) | Aggressive compression | Expected inflection point |

The 16bit inference on each model defines the **ground truth labels** used to evaluate lower-precision versions. This means the experiment measures how much each model diverges from its own full-precision behavior — not from an external gold standard — when evaluated separately on EN and PT-BR inputs.

---

## Why 2-bit Was Excluded from the Main Experiment

2-bit quantization is not natively supported by vLLM. Achieving it would require a different toolchain (llama.cpp, Ollama, or AQLM), introducing a methodological inconsistency: all other levels are quantized via AWQ/vLLM, while 2-bit would use a different method and format (GGUF or AQLM).

Including 2-bit in the same analysis would conflate two variables — quantization level and quantization method — making it impossible to attribute observed degradation to bits alone.

For this reason, 2-bit is reserved as an **exploratory analysis** to be conducted separately if time and resources allow, with results reported under explicit methodological caveats. See the [README](../README.md) for details on the planned exploratory extension.

---

## Quantization Pipeline

All models follow the same quantization pipeline:

```
Original model weights (BF16, HuggingFace)
        ↓
vLLM loads model with quantization="awq"
        ↓
Inference on EN dataset → predictions + token probabilities
        ↓
Inference on PT-BR dataset → predictions + token probabilities
        ↓
Metrics computed per level: F1, Precision, Recall, ΔF1 (EN − PT)
```

This pipeline is applied identically across all five models and all three quantization levels, ensuring full comparability of results.