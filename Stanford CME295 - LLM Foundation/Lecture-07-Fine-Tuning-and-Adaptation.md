# Lecture 07 — Fine-Tuning & Adaptation
## Stanford CME295 | PM Reference Notes

**One-line concept:** Fine-tuning adapts a pretrained model to a specific domain or behavior without retraining from scratch. Parameter-efficient methods like LoRA do this at a fraction of the cost, making customization accessible to product teams.

**Why it matters for PMs:** Fine-tuning is the customization layer between a general-purpose foundation model and a product-specific model. Understanding the options and tradeoffs lets you scope when to fine-tune, what kind, and what it will cost vs. what you'll gain.

---

## 1. Why Fine-Tune at All?

A pretrained + RLHF model (like GPT-4 or Claude) is a general-purpose assistant. For many products, that's sufficient. But fine-tuning makes sense when:

1. **Domain-specific behavior** — the model needs to respond in a highly specific style, use proprietary terminology, or follow domain-specific formats (legal, medical, code review)
2. **Consistent task format** — you want the model to always return structured JSON, or always follow a specific response template
3. **Reducing prompt length** — behaviors that currently require long system prompts can be "baked in" via fine-tuning, reducing per-query cost
4. **Performance on narrow tasks** — a fine-tuned small model often outperforms a larger general model on specific tasks at lower cost
5. **Proprietary knowledge** — you want the model to "know" your internal documentation without runtime retrieval

---

## 2. Full Fine-Tuning

**What it is:** Continue training the model on new task-specific data, updating all weights.

**Process:**
- Start with a pretrained (or instruction-tuned) model
- Train on your dataset with a lower learning rate than pretraining
- All model parameters are updated

**Advantages:**
- Maximum quality for the target task
- No architectural constraints

**Disadvantages:**
- **Catastrophic forgetting:** Training on new data can degrade performance on tasks not in the fine-tuning dataset — the model "forgets" some of its general capabilities
- **Cost:** Requires the full model in GPU memory + gradient storage + optimizer states — same infrastructure as pretraining but on smaller data
- **Serving:** You now maintain a full separate model copy per fine-tuned variant

**When to use:** Only when you have sufficient data (100K+ examples) and need maximum domain-specific quality, and have the infrastructure to serve separate model weights.

---

## 3. Parameter-Efficient Fine-Tuning (PEFT)

The key insight: most of the model's general intelligence doesn't need to change. Only a small subset of weights need updating for task adaptation.

PEFT methods update a small number of new parameters while keeping the original model frozen. This dramatically reduces cost while achieving quality close to full fine-tuning.

### 3.1 LoRA — Low-Rank Adaptation

The dominant PEFT method. Used in Firefly Custom Models, most open-source fine-tuning pipelines, and many enterprise AI products.

**The idea:**
The weight update matrix (ΔW) that would result from full fine-tuning can be approximated by two smaller matrices:
```
ΔW ≈ A × B
Where:
  A is (d × r)    — r << d
  B is (r × k)    — r << k
  r = rank (typically 4, 8, 16, or 64)
```

Instead of updating the full weight matrix W (d × k parameters), you train only A and B ((d+k)×r parameters).

**Compression ratio:** For rank r=16 on a 4096×4096 weight matrix:
- Full update: 16.7M parameters
- LoRA: 2 × 4096 × 16 = 131K parameters — **127× fewer parameters**

During inference, you can merge the LoRA weights back into the original: W_new = W + A×B. No inference overhead.

**QLoRA:** LoRA applied to a quantized base model (4-bit or 8-bit quantization). Allows fine-tuning large models on consumer hardware.
- A 70B model at 4-bit quantization fits in ~35GB — accessible on 2–4 consumer GPUs
- Quality is slightly lower than full-precision LoRA but dramatically more accessible

**PM implication for Firefly/Adobe:**
Firefly Custom Models is essentially a LoRA product: users provide their brand assets, and the product trains LoRA adapters on the Firefly foundation model that capture the brand's visual style. The adapter is small, cheap to store, and can be loaded on top of the shared foundation model per-request.

**Scaling:** A company can serve thousands of LoRA adapters on a single base model — one adapter per customer, loaded dynamically. This is the enterprise customization architecture.

### 3.2 Prompt Tuning

**What it is:** Instead of updating model weights, train a small set of "soft prompt" tokens prepended to the input. These tokens are learned vectors that steer the model's behavior.

**How it works:**
- The original model is completely frozen
- A small set of trainable vectors (the "prefix") are prepended to every input
- Only these prefix vectors are updated during training

**Advantages:**
- No weight updates — extremely cheap to store (just the prefix vectors)
- Easy to swap different behaviors by switching prefixes

**Disadvantages:**
- Quality typically lower than LoRA for complex tasks
- Less interpretable (the soft tokens have no human-readable meaning)
- Requires the model to support prefix injection at inference

### 3.3 Adapter Layers

**What it is:** Insert small trainable neural network modules between existing layers of the frozen model.

**Architecture:**
```
Original Layer → Adapter (small FFN: down-project → nonlinearity → up-project) → Next Layer
```

The adapter is a bottleneck: it projects down to a small dimension, applies a nonlinearity, then projects back up. Only the adapter weights are trained.

**Compared to LoRA:** Adds inference latency (the adapters are separate modules, not merged into weights). LoRA is preferred for production deployment because it merges back to zero overhead.

---

## 4. In-Context Learning (ICL) — The No-Gradient Alternative

**What it is:** Instead of updating model weights, provide examples of desired behavior directly in the prompt. The model uses these examples to infer the task pattern.

**Few-shot learning example:**
```
Input: "The movie was fantastic." → Output: positive
Input: "I hated every minute of it." → Output: negative
Input: "The acting was mediocre." → Output: ???
```

The model infers the pattern from the examples and applies it to the new input — no training required.

**Zero-shot:** No examples; just an instruction. "Classify the sentiment of the following text as positive or negative."

**Performance vs. fine-tuning:**
- ICL is convenient and requires no training infrastructure
- Fine-tuning typically outperforms ICL for narrow tasks when you have sufficient data
- ICL degrades as task complexity increases

**PM relevance:** ICL is the fastest path to evaluating whether a task is doable before investing in fine-tuning. Use ICL for prototyping; fine-tune for production if quality isn't sufficient.

---

## 5. Retrieval-Augmented Generation (RAG) vs. Fine-Tuning

Two common approaches to extending model knowledge:

| Dimension | RAG | Fine-Tuning |
|---|---|---|
| **What it does** | Retrieves relevant documents at inference time and passes them in context | Updates model weights to internalize new knowledge or behavior |
| **Best for** | Dynamic, frequently updated knowledge; factual grounding | Stable domain knowledge; style/format adaptation; task-specific behavior |
| **Knowledge updates** | Near-real-time (update the retrieval index) | Requires a new fine-tuning run |
| **Cost** | Retrieval infrastructure + longer context | Training compute (one-time) |
| **Hallucination risk** | Lower (grounded in retrieved documents) | Higher (model may confabulate internalized facts) |
| **When to use** | Product knowledge bases, support docs, company policies | Domain-specific behavior, consistent formatting, specialized terminology |

**The hybrid pattern (RAG + Fine-Tuning):**
- Fine-tune for style, behavior, and domain vocabulary
- Use RAG for dynamic factual grounding
- Both together: consistent behavior + accurate, current knowledge

---

## 6. When Fine-Tuning Doesn't Help (and What Does)

Fine-tuning is not a universal fix:

| Problem | Why Fine-Tuning Doesn't Help | Better Solution |
|---|---|---|
| Model doesn't know a fact | Fine-tuning shapes behavior; can't add new factual knowledge reliably | RAG |
| Model is too slow | Fine-tuning doesn't change inference cost | Distillation, quantization, smaller model |
| Model gives wrong answers on new domain | Knowledge not in pretraining → fine-tuning on wrong knowledge may worsen performance | Add domain data to pretraining (expensive) or use RAG |
| Model refuses reasonable requests | RLHF safety settings; fine-tuning can help, but so can system prompt engineering | System prompt or LoRA on behavior |

---

## 7. Data Requirements for Fine-Tuning

A common question: how much data do I need?

| Method | Data Requirement | Quality Requirement |
|---|---|---|
| **Full fine-tuning** | 100K–1M+ examples | High |
| **LoRA** | 1K–100K examples | High |
| **Prompt tuning** | 1K–10K examples | Medium |
| **Few-shot ICL** | 5–50 examples | Must be high quality |

**Data quality matters more than quantity for PEFT:** 1,000 high-quality task-specific examples can outperform 100,000 mediocre ones for LoRA fine-tuning. This is the same principle as pretraining data quality.

---

## Key Terms

| Term | Definition |
|---|---|
| **Fine-tuning** | Continuing model training on task-specific data |
| **Full fine-tuning** | Updating all model parameters on new data |
| **PEFT** | Parameter-Efficient Fine-Tuning — updating only a small fraction of parameters |
| **LoRA** | Low-Rank Adaptation — approximates weight updates with low-rank matrices |
| **QLoRA** | LoRA applied to a quantized (4-bit) base model |
| **Rank (r)** | The bottleneck dimension in LoRA; controls the number of trainable parameters |
| **Adapter** | Small trainable modules inserted between frozen model layers |
| **Prompt tuning** | Training soft (learnable) token embeddings prepended to every input |
| **ICL** | In-Context Learning — task pattern inferred from examples in the prompt |
| **Few-shot learning** | ICL with a small number of examples in the prompt |
| **Zero-shot** | No examples; model must infer task from instruction alone |
| **Catastrophic forgetting** | Degradation of general capabilities when fine-tuning on narrow data |
| **RAG** | Retrieval-Augmented Generation — augmenting prompts with retrieved documents |
| **Distillation** | Training a smaller model to mimic a larger model's outputs |
| **Quantization** | Reducing numerical precision of model weights to reduce size and inference cost |

---

## Product Questions This Unlocks

1. "Should we fine-tune or use RAG for our knowledge base?" — RAG for dynamic/updated knowledge; fine-tuning for stable behavioral adaptation.
2. "How many examples do we need to fine-tune this?" — 1K–10K high-quality examples for LoRA; evaluate quality at each scale.
3. "Can we support custom models per enterprise customer?" — Yes, via LoRA: one base model, many small adapters. Load the customer's adapter per request.
4. "Why doesn't fine-tuning improve factual accuracy?" — Fine-tuning shapes behavior, not knowledge. Facts require pretraining or RAG.
5. "What's the cost of fine-tuning vs. building a new model?" — Fine-tuning: hours to days, thousands of dollars. New model: months, millions. Fine-tuning first is almost always right.

---

## Common PM Mistakes

- **"Fine-tuning will make the model smarter"** — It makes the model better at specific tasks; may not improve general intelligence at all.
- **"We need 1M examples to fine-tune"** — For LoRA, 1K–10K high-quality examples is often sufficient and outperforms large noisy datasets.
- **"RAG and fine-tuning are alternatives"** — They solve different problems and work well together.
- **"We can fine-tune away hallucinations"** — Hallucination reduction requires architectural and RAG approaches, not just fine-tuning.

---

*Lecture 7 of 12 — Stanford CME295 LLM Foundations | PM Reference*
