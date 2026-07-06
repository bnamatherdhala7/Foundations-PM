# Lecture 04 — Scaling Laws
## Stanford CME295 | PM Reference Notes

**One-line concept:** Scaling laws are empirical equations that predict model quality as a function of model size, training data size, and compute budget — giving teams a principled way to allocate resources before spending millions on a training run.

**Why it matters for PMs:** Scaling laws govern the most expensive decisions on a Foundations team — how big to make the model, how much data to collect, how much compute to buy. The PM who understands scaling laws can translate a business quality target into a resource requirement and vice versa.

---

## 1. The Core Insight — Loss Follows Power Laws

Model performance (measured as loss — roughly, how surprised the model is by the next token) follows **power laws** with respect to:
- **N** — number of model parameters
- **D** — number of training tokens
- **C** — total compute (measured in FLOPs = floating point operations)

A power law means: doubling model size reduces loss by a fixed multiplicative amount (not a fixed additive amount). Each doubling gives you diminishing but predictable returns.

**The implication:** You can predict how good your next model will be before training it, based on smaller experiments. This is how frontier labs plan multi-year model roadmaps.

---

## 2. The Landmark Papers

### 2.1 Kaplan et al. (2020) — OpenAI Scaling Laws

The first systematic study. Key findings:
- Model performance scales smoothly and predictably with N, D, and C
- For a fixed compute budget C, the **optimal strategy** is to train a large model on fewer tokens
- Data size is less important than model size (at the time of this paper)

**The dominant recommendation from this paper:** Scale up model size; don't worry as much about data volume.

This drove GPT-3 (175B parameters) — a massive model trained on relatively fewer tokens than later optimal analyses would recommend.

### 2.2 Hoffmann et al. (2022) — Chinchilla Scaling Laws (DeepMind)

Challenged and corrected the Kaplan findings. Chinchilla trained 400+ models of varying sizes and data amounts with the same compute budget.

**The Chinchilla finding:**
For a compute-optimal model, model size (N) and training tokens (D) should scale **equally**. The Kaplan recommendation had been over-investing in model size and under-investing in training data.

**The Chinchilla optimal formula (approximate):**
```
N_optimal ≈ C^0.5
D_optimal ≈ C^0.5

Or more practically:
D_optimal ≈ 20 × N
```

For every 1B parameters, a compute-optimal model needs ~20B training tokens.

**Examples:**
- A 7B parameter model is compute-optimal with ~140B tokens
- A 70B parameter model needs ~1.4 trillion tokens
- GPT-3 (175B) was trained on ~300B tokens — about 3× less data than Chinchilla-optimal

**The Chinchilla model itself:** 70B parameters, 1.4T tokens — outperformed GPT-3 (175B) on most benchmarks despite being smaller. Smaller model, more data = better.

---

## 3. The Compute Budget Triangle

```
         Compute Budget (C)
        /                  \
       /                    \
Model Size (N)    Training Tokens (D)
```

For a fixed compute budget, you are allocating between model size and training data. There is an optimal ratio (Chinchilla). Deviating in either direction reduces quality for the same cost.

**The practical decisions this governs:**

| Question | Scaling Law Answer |
|---|---|
| "Should we train a bigger model or collect more data?" | At Chinchilla-optimal: scale both equally. If you're data-constrained, a smaller model is better. |
| "Is our model undertrained or undertrained?" | Compare actual training tokens to 20× the parameter count. Most pre-2022 models were undertrained. |
| "What quality will our next model achieve?" | Fit a power law on smaller runs, extrapolate to target compute. |
| "How much compute is needed to match Model X's quality?" | Estimate from published quality metrics and known parameter/data counts. |

---

## 4. Inference Cost vs. Training Cost — The Hidden Tradeoff

Chinchilla optimal is the right choice if you train once and run inference once. But **in practice, models are run millions of times after training.**

**The inference reality:**
- Inference cost per query scales with model size (roughly linearly for simple operations)
- A 70B model costs ~10× as much to serve per query as a 7B model
- Training cost is a one-time expense; inference cost is ongoing

**This creates a practical deviation from Chinchilla optimal:**

If you expect 100M+ queries after training, it's often worth **training a smaller model on more data** (beyond Chinchilla optimal) to get a higher-quality small model that is cheaper to serve.

**Llama 2 and Llama 3 example:** Meta trained 7B and 13B models on far more tokens than Chinchilla-optimal (~2T tokens for the 7B). Result: a 7B model that performs comparably to larger models on many tasks, at lower inference cost.

This is the **"train longer, serve cheaper"** strategy — highly relevant for any product serving at scale.

**PM framing:** For Foundations PM, the question isn't just "what model is highest quality?" but "what is the cost per query at production scale for each quality point we gain?" Scaling laws let you answer this before you build.

---

## 5. Emergent Capabilities — The Scaling Law Surprise

Some capabilities appear suddenly at certain scale thresholds, not gradually:

| Capability | Approximate Emergence Scale |
|---|---|
| Few-shot arithmetic | ~10B parameters |
| Chain-of-thought reasoning | ~50–100B parameters |
| Instruction following (zero-shot) | ~50B+ parameters |
| Code generation (useful quality) | ~10B+ parameters |
| Multilingual transfer | ~10B+ parameters |

**The "emergence" phenomenon:** Below a threshold, the model performs near-random on a task. At the threshold, performance suddenly jumps. This is controversial — some researchers argue emergence is an artifact of evaluation metrics, not a real discontinuity.

**PM implication:** You cannot extrapolate all capabilities linearly from small experiments. Some capabilities only appear at scale you can't cheaply test. This creates planning uncertainty: you don't know which new capabilities you'll get from a larger training run.

---

## 6. Scaling Laws Beyond Language

Scaling laws appear to generalize across modalities:

| Domain | Observation |
|---|---|
| **Image generation** | FID (quality) follows power law with model size and training data |
| **Code generation** | HumanEval scores scale with model size and code training tokens |
| **Multimodal** | Vision-language performance scales with both vision data and language model scale |
| **Protein structure** | AlphaFold-style models show similar scaling behavior |

**PM takeaway:** These laws are empirical, not theoretical. They can break (and do, at extreme scales or for specific capabilities). But they're the best planning tool available.

---

## 7. The Scaling Plateau Debate

Are we approaching the limits of scaling returns?

**Evidence that scaling still works:**
- GPT-4 → GPT-4o → o1 → o3 shows continued quality improvement
- Gemini Ultra → Gemini 1.5 → Gemini 2 continues improving
- Frontier labs are actively building 100T parameter clusters

**Evidence of diminishing returns:**
- Some benchmark scores are saturating (models hit near-100% on older benchmarks)
- Data availability is a constraint — high-quality text on the internet may be approaching exhaustion
- Cost per quality point is increasing

**The emerging consensus:** Scaling continues to work, but the composition is shifting:
- Raw pretraining scaling: diminishing returns on some capabilities
- **Test-time compute scaling** (reasoning models like o1, DeepSeek-R1): significant quality gains by giving the model more compute at inference, not just training
- **Synthetic data and RL**: new frontiers beyond web-text pretraining

**PM relevance:** The roadmap for Foundations teams is shifting from "bigger pretrained model" to "smarter use of compute at training and inference." Understanding this shift matters for headcount allocation, infrastructure investment, and roadmap framing.

---

## Key Terms

| Term | Definition |
|---|---|
| **Scaling law** | Empirical relationship between model quality and model size / data / compute |
| **Power law** | Mathematical relationship where doubling one variable multiplies the other by a fixed factor |
| **Chinchilla optimal** | The compute-optimal ratio: ~20 training tokens per model parameter |
| **FLOPs** | Floating Point Operations — the standard unit for measuring compute |
| **Loss** | The model's training objective; lower loss = better model (roughly: lower surprise at next token) |
| **Emergent capability** | Capability that appears suddenly at scale, not gradually |
| **Test-time compute** | Compute used during inference (e.g., chain-of-thought reasoning, search); distinct from training compute |
| **Inference cost** | Compute cost of running the model on a single query |
| **Undertrained** | A model that was trained on fewer tokens than Chinchilla-optimal given its parameter count |
| **Parameter count** | Total number of learnable weights in the model (e.g., 7B, 70B, 700B) |
| **Train-longer strategy** | Training a smaller model on more tokens than Chinchilla-optimal to get better inference economics |

---

## Product Questions This Unlocks

1. "What quality improvement will we get from the next training run?" — Fit scaling curves from current data points and extrapolate.
2. "Should we train a 7B or 70B model for this use case?" — Depends on inference volume and quality requirements; scaling laws give you the quality prediction, inference economics give you the cost projection.
3. "Why are we collecting more data rather than increasing model size?" — We may be underweight on data relative to Chinchilla optimal; or we're optimizing for inference cost, not training quality.
4. "What's our roadmap for the next 12 months of model improvement?" — Model the expected quality gains as a function of planned compute spend using scaling law curves.
5. "When will we run out of training data?" — High-quality web text is finite; estimate when your scaling laws will hit data constraints and model your synthetic data strategy accordingly.

---

## Common PM Mistakes

- **"Train the biggest model we can"** — Without proportional data, a large model is undertrained and underperforms a smaller, well-trained model.
- **"Scaling will solve it"** — Not all capabilities scale smoothly. Some require architectural changes, not just more compute.
- **"We can measure quality improvement with one eval"** — Loss and a single benchmark don't capture the full quality surface. Use multiple evaluations.
- **"Once we hit X parameters, we'll have the capability we need"** — Emergent capabilities appear at unpredictable thresholds; don't promise capabilities you haven't measured.

---

*Lecture 4 of 12 — Stanford CME295 LLM Foundations | PM Reference*
