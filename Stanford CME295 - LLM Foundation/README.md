# Stanford CME295 — Large Language Models
## PM Reference Guide: Frontier Model Foundations

**Purpose:** Quick-reference notes for a Product Manager on an AI Foundations / Frontier Models team. Each lecture doc translates technical concepts into product decisions, tradeoffs, and vocabulary you need to work fluently with research and engineering.

**How to use this:**
- Read the **One-line concept** and **Why it matters for PMs** first — those are the 60-second version.
- Use **Key Terms** as a glossary during technical discussions.
- Use **Product Questions This Unlocks** before planning meetings or roadmap reviews.
- Use **Common PM Mistakes** to self-check before presenting a data or model strategy.

---

## Lecture Index

| # | Lecture | Core PM Takeaway |
|---|---|---|
| [01](Lecture-01-Transformers-and-Attention.md) | Transformers & Attention | Architecture choice = capability ceiling. Know why transformers replaced everything before them. |
| [02](Lecture-02-Tokenization-and-Embeddings.md) | Tokenization & Embeddings | "Context window" = tokens, not words. Tokenization affects cost, latency, and multilingual quality. |
| [03](Lecture-03-Pretraining-and-Data-Pipeline.md) | Pretraining & Data Pipeline | Training data is the capability ceiling. Quality filtering decisions made here determine everything downstream. |
| [04](Lecture-04-Scaling-Laws.md) | Scaling Laws | Compute budget must be split optimally between model size and data volume. Bigger model ≠ better model. |
| [05](Lecture-05-Training-Infrastructure.md) | Training Infrastructure at Scale | Infrastructure choices = cost structure and iteration velocity. Slow iteration = slower roadmap. |
| [06](Lecture-06-RLHF-and-Instruction-Following.md) | RLHF & Instruction Following | This is where the model learns to be useful. Alignment = product quality layer on top of raw capability. |
| [07](Lecture-07-Fine-Tuning-and-Adaptation.md) | Fine-Tuning & Adaptation | LoRA and parameter-efficient fine-tuning are the PM's tool for customization without full retraining costs. |
| [08](Lecture-08-Evaluation-and-Benchmarks.md) | Evaluation & Benchmarks | Defining your eval is defining your quality bar. Goodhart's Law applies. |
| [09](Lecture-09-Multimodal-Models.md) | Multimodal Models | Each modality adds capability but also evaluation complexity and data pipeline cost. |
| [10](Lecture-10-Inference-and-Deployment.md) | Inference & Deployment | Inference optimization = cost structure and UX quality. Latency is a product requirement. |
| [11](Lecture-11-Safety-and-Alignment.md) | Safety & Alignment | Safety is a product design constraint, not a post-hoc filter. Red teaming = QA for failure modes. |
| [12](Lecture-12-Frontier-Models-and-Future.md) | Frontier Models & What's Next | Reasoning models, agents, and tool use — the product decisions that matter in 2025–2027. |

---

## The Mental Model That Ties Everything Together

```
Training Data Quality
        ↓
  Architecture Choice
        ↓
  Pretraining at Scale    ← Scaling Laws govern this
        ↓
  Instruction Tuning / RLHF   ← Makes it useful
        ↓
    Evaluation          ← Defines your quality bar
        ↓
  Fine-Tuning / Adaptation    ← Customization layer
        ↓
  Inference & Deployment  ← Where cost meets UX
        ↓
  Safety & Monitoring    ← Ongoing product quality
```

Every PM decision on a Foundations team maps to at least one layer of this stack.

---

*Based on Stanford CME295 — Large Language Models. Notes adapted for PM use.*
