# Lecture 10 — Inference & Deployment
## Stanford CME295 | PM Reference Notes

**One-line concept:** Inference is running a trained model to generate responses. At scale, inference cost dominates training cost — and optimization techniques (quantization, KV cache, speculative decoding) determine the latency and cost of every user interaction.

**Why it matters for PMs:** Every product decision about latency, cost per query, and scale connects directly to inference architecture. "Can we afford to run this model?" and "Why is it slow?" are inference questions, not training questions.

---

## 1. The Inference Cost Reality

After training, inference becomes the ongoing, dominant cost:

| Scale | Economics |
|---|---|
| Training | One-time: millions of dollars, weeks of compute |
| Inference | Ongoing: cost per query × query volume |
| 1M queries/day at $0.01/query | $3.65M/year in inference costs |

**Why inference is expensive:**
- Every query requires loading model weights → compute attention → generate tokens sequentially
- Autoregressive generation: each token requires a full forward pass
- Large models (70B+) require multiple high-end GPUs just to hold weights in memory

**The PM implication:** Model quality and inference cost are in direct tension. A better model is usually a larger, slower, more expensive model. Product requirements must specify both the quality bar and the latency/cost budget simultaneously.

---

## 2. The Autoregressive Generation Bottleneck

**Prefill phase:** Process the entire input prompt in parallel — this is fast because transformers process all input tokens simultaneously.

**Decode phase:** Generate output tokens one at a time — each token requires a full forward pass through the model. This is the bottleneck.

```
Prefill: "What is the capital of France?" → processed in ~1 forward pass
Decode: "The" → pass 1; "capital" → pass 2; "of" → pass 3; "France" → pass 4; "is" → pass 5; "Paris" → pass 6
```

**Time-to-first-token (TTFT):** Latency from prompt submission to first generated token (prefill latency). Critical for perceived responsiveness.
**Tokens per second (TPS):** Decode speed. Determines how long the user waits for the full response.

---

## 3. KV Cache — The Most Important Inference Optimization

**The problem:** During autoregressive generation, the attention mechanism needs to attend to all previously generated tokens. Without optimization, this means recomputing key-value pairs for every prior token at every decode step.

**The solution — KV Cache:**
- During the prefill and decode phases, store (cache) the key (K) and value (V) attention matrices for all processed tokens
- On each decode step, only compute new K/V for the newest token; reuse cached K/V for prior tokens
- Eliminates redundant computation; dramatically speeds up decode

**The cost:** KV cache takes memory — for a long conversation, the cache grows with the context length:
```
KV cache size ≈ 2 × num_layers × num_heads × d_head × seq_len × bytes_per_element
```

For a 70B model with 32K context: KV cache can be 10–40GB — sometimes larger than the model weights for long contexts.

**PM implications:**
- Long context = large KV cache = more GPU memory required = higher cost
- Memory for KV cache comes from the same pool as model weights — limits batch size
- "Context is expensive" is a KV cache story

---

## 4. Quantization — Trading Precision for Speed

**What it is:** Reducing the numerical precision of model weights and/or activations to use less memory and enable faster computation.

| Precision | Bits | Memory vs. FP32 | Speed | Quality Impact |
|---|---|---|---|---|
| FP32 | 32 | 1× | 1× | None (baseline) |
| BF16/FP16 | 16 | 0.5× | 2–4× | Negligible |
| INT8 | 8 | 0.25× | 2–4× | Small degradation |
| INT4 | 4 | 0.125× | 3–5× | Moderate degradation |
| INT2 | 2 | 0.0625× | Experimental | Significant degradation |

**Why 4-bit works surprisingly well:** Most of a model's information is in the distribution of weights, not their precise values. Rounding to the nearest of 16 values (4-bit) captures ~99% of the information for most tasks.

**GPTQ, AWQ, GGUF:** Popular quantization formats for LLMs. AWQ (Activation-Aware Weight Quantization) is currently one of the best quality-preserving 4-bit methods.

**llama.cpp:** Open-source inference framework that runs quantized models on CPU + GPU. Made LLMs accessible on consumer hardware.

**PM application:**
- A 70B model at FP16 requires ~140GB GPU memory (2× A100s)
- Same model at INT4 requires ~35GB (fits on 1 A100 or 2 consumer GPUs)
- Cost savings: 3–4× fewer GPUs for same throughput
- Quality cost: slight degradation, often undetectable in product use cases

---

## 5. Batching — Serving Multiple Requests Together

**Naive batching:** Process each request one at a time. GPU utilization is poor — the GPU is mostly idle between requests.

**Static batching:** Group N requests into a batch; process together. Better GPU utilization but all requests in a batch must wait for the longest one to finish (batch latency = max request latency).

**Continuous batching (iteration-level batching):** The key innovation for LLM serving. Insert new requests into the batch as decode slots become available — don't wait for the entire batch to finish.

```
Standard: [Request A: 500 tokens] [Request B: 50 tokens] → both wait for A to finish
Continuous: B completes at step 50, immediately replaced by Request C → much better GPU utilization
```

**Impact:** 5–10× better throughput vs. naive approaches. Used in vLLM, TensorRT-LLM, and production LLM serving systems.

---

## 6. Speculative Decoding

**The problem:** Large model decode is slow — each token requires a full forward pass through the large model.

**The insight:** What if a small (fast) model generates candidate tokens, and the large model only needs to verify them?

**How it works:**
1. A small "draft" model generates K candidate tokens (fast — small model is cheap)
2. The large "verifier" model evaluates all K tokens in a single forward pass (parallelized)
3. If the large model agrees with a candidate token, accept it. If it disagrees, reject it and generate the correct token.

**The gain:** When the draft model is often right, you get K tokens for roughly the cost of 1 large-model forward pass. 2–3× speedup with identical output quality (mathematically proven to preserve the exact output distribution of the large model).

**Limitation:** Works best when the draft and verifier models are closely related (same family). Requires maintaining two models in memory.

---

## 7. Flash Attention

**The problem:** Standard attention computes the full N×N attention matrix for a sequence of length N. Memory requirement is O(N²) — for long sequences, this dominates GPU memory.

**Flash Attention (Dao et al., 2022):** Restructures the attention computation to use tiled computation on the GPU's fast SRAM rather than slow HBM (GPU main memory). Produces identical results but:
- Memory: O(N) instead of O(N²)
- Speed: 2–4× faster attention computation
- Enables much longer context windows on the same hardware

**Flash Attention 2 and 3:** Continued improvements; standard in all modern LLM training and inference frameworks.

**PM implication:** Flash Attention is not a quality choice — it's an efficiency choice that enables longer context at the same memory budget. When teams say "we can now support 128K context," Flash Attention is usually part of the story.

---

## 8. Distillation — Smaller Models from Larger Models

**What it is:** Training a small "student" model to mimic the outputs of a large "teacher" model.

**Knowledge distillation:**
- Train the student to match the teacher's output probability distribution (soft labels), not just the correct answer (hard labels)
- The soft distribution contains more information: the teacher's uncertainty and near-misses are useful signal

**The result:** A smaller model that is often significantly better than a model of the same size trained from scratch, because it learned from a more capable teacher.

**Examples:**
- DistilBERT: 60% of BERT's size, 97% of BERT's quality on most tasks
- Llama models fine-tuned on GPT-4 outputs (controversial — violates OpenAI ToS but technically effective)
- Phi-3 and Gemma: small models trained partly on synthetic data from larger models

**PM application for Splunk:**
- Fine-tune a small model (7B or 13B) to be a specialist in security log analysis by distilling from a larger general model
- Result: faster, cheaper inference for the specific task with quality close to the larger model
- The key: the task must be well-defined and the small model must have enough capacity

---

## 9. Serving Infrastructure Choices

| System | Best For | Notes |
|---|---|---|
| **vLLM** | High-throughput serving; open-source | Continuous batching, PagedAttention (KV cache management) |
| **TensorRT-LLM** | NVIDIA GPU optimization | Best raw throughput on NVIDIA hardware |
| **llama.cpp** | CPU/consumer GPU inference | Quantized models; good for on-device or low-cost hosting |
| **Ollama** | Local development and testing | Wraps llama.cpp with easy model management |
| **OpenAI API / Anthropic API** | Managed inference | No infrastructure; pay per token |
| **Azure OpenAI / AWS Bedrock** | Enterprise managed | Model hosting with enterprise SLAs |

---

## 10. The Latency Budget — A Framework for PMs

Define latency requirements before choosing infrastructure:

| Use Case | Acceptable TTFT | Acceptable TPS | Model Tier |
|---|---|---|---|
| Interactive chat | < 500ms | > 30 tokens/sec | Medium model, optimized serving |
| Real-time analysis | < 200ms | > 50 tokens/sec | Small model or distilled specialist |
| Batch processing | Minutes acceptable | N/A | Large model, offline |
| Code completion (IDE) | < 100ms | > 50 tokens/sec | Small model (<7B) |
| Security alert triage (Splunk) | < 2 seconds | > 20 tokens/sec | Medium model with domain fine-tuning |

---

## Key Terms

| Term | Definition |
|---|---|
| **Inference** | Running a trained model to generate outputs |
| **Prefill** | Processing the input prompt in a single forward pass (fast; parallel) |
| **Decode** | Generating output tokens one at a time (slow; sequential) |
| **TTFT** | Time-to-First-Token — latency from input to first generated token |
| **TPS** | Tokens per second — decode throughput |
| **KV Cache** | Cached key-value attention matrices from prior tokens; eliminates redundant computation |
| **Quantization** | Reducing numerical precision of weights to reduce memory and increase speed |
| **INT8/INT4** | 8-bit and 4-bit integer quantization formats |
| **Continuous batching** | Serving technique that inserts new requests into available decode slots without waiting for full batch completion |
| **Speculative decoding** | Using a small draft model to generate candidate tokens; verified by the large model in parallel |
| **Flash Attention** | Memory-efficient attention computation; O(N) memory vs O(N²) |
| **Distillation** | Training a small model to mimic a large model's outputs |
| **vLLM** | Open-source high-throughput LLM serving framework |
| **PagedAttention** | vLLM's technique for efficient KV cache memory management |
| **TensorRT-LLM** | NVIDIA's optimized inference framework |
| **GPTQ/AWQ** | Quantization algorithms for 4-bit model compression |

---

## Product Questions This Unlocks

1. "Why is our model too slow for real-time alert triage?" — Likely TTFT too high (prefill on long logs) or TPS too low (model too large). Solutions: quantization, smaller distilled model, or Flash Attention.
2. "How much will inference cost at 1M queries/day?" — Token count per query × model size × cost per token at your chosen serving tier.
3. "Can we run this model on-premises for our enterprise customers?" — Depends on model size and customer's hardware. With quantization, a 70B model might fit on a $30K GPU; without, you need a $200K+ cluster.
4. "Why does performance degrade on long context queries?" — KV cache memory pressure reduces batch size at long context, causing throughput degradation and latency increase.
5. "How do we get the quality of a 70B model at the cost of a 7B model?" — Speculative decoding (70B verifier, 7B draft) or distillation (train 7B specialist on 70B teacher outputs).

---

*Lecture 10 of 12 — Stanford CME295 LLM Foundations | PM Reference*
