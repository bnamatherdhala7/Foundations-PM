# Lecture 05 — Training Infrastructure at Scale
## Stanford CME295 | PM Reference Notes

**One-line concept:** Training a frontier model requires distributing computation across thousands of GPUs simultaneously, with specialized techniques to keep them working in parallel without bottlenecks. Infrastructure choices determine iteration speed, cost, and ultimately roadmap velocity.

**Why it matters for PMs:** Infrastructure is not a background concern — it directly constrains how fast the team can run experiments, how quickly you can iterate on a model, and how much a training run costs. Slow infrastructure = slow roadmap. Understanding the vocabulary lets you engage in resource and timeline planning.

---

## 1. Why Distribution Is Necessary

A frontier model (70B+ parameters) cannot fit on a single GPU. Even if it could, training on trillions of tokens serially would take years.

**The constraint:**
- NVIDIA A100: 80GB VRAM. A 70B parameter model in float16 requires ~140GB just to hold the weights.
- GPT-4 is estimated at ~1T parameters — requires thousands of GPUs just to hold the model, let alone train it.
- Training data: trillions of tokens × multiple passes × gradient computation → petabytes of I/O.

Solution: **Distributed training** — split the work across thousands of GPUs.

---

## 2. Types of Parallelism

There are three ways to split a training job across GPUs. Real training runs use all three simultaneously (3D parallelism).

### 2.1 Data Parallelism

**Idea:** Each GPU holds a complete copy of the model, but processes a different subset of the training batch. After computing gradients, all GPUs average their gradients (AllReduce operation) and update their copies identically.

```
GPU 1: full model + batch shard 1 → gradients
GPU 2: full model + batch shard 2 → gradients
GPU 3: full model + batch shard 3 → gradients
              ↓
        AllReduce (average gradients)
              ↓
     All GPUs update their model copy identically
```

**Limitation:** Only works when the full model fits on a single GPU. Not viable for very large models.

**Scales:** Linearly with number of GPUs for data throughput.

### 2.2 Model Parallelism (Tensor Parallelism)

**Idea:** Split individual layers across multiple GPUs. Each GPU holds a horizontal slice of the weight matrices.

```
Attention layer weight matrix (4096 × 4096):
  GPU 1: rows 0–1023
  GPU 2: rows 1024–2047
  GPU 3: rows 2048–3071
  GPU 4: rows 3072–4095
```

Each GPU computes its slice of the output, then GPUs exchange partial results.

**Advantage:** Allows training models that don't fit on a single GPU.
**Limitation:** Requires very high inter-GPU bandwidth (GPUs must communicate results frequently). Only efficient on high-bandwidth interconnects (NVLink, InfiniBand).

### 2.3 Pipeline Parallelism

**Idea:** Split the model's layers across GPUs sequentially. GPU 1 handles layers 1–10, GPU 2 handles layers 11–20, etc. Data flows through GPUs in a pipeline.

```
GPU 1: Layers 1–10   →   GPU 2: Layers 11–20   →   GPU 3: Layers 21–30
       (Embedding)              (Middle)                    (Output)
```

**Advantage:** Allows very deep models to be distributed without the all-to-all communication of tensor parallelism.
**Limitation:** "Pipeline bubbles" — GPUs are idle while waiting for the previous stage to finish. Scheduling schemes (GPipe, 1F1B) reduce this but don't eliminate it.

### 2.4 The Practical Reality: 3D Parallelism

Frontier training runs combine all three:
- **Tensor parallelism** within a node (fast NVLink between GPUs on the same server)
- **Pipeline parallelism** across nodes (slower inter-node network)
- **Data parallelism** across pipeline replicas

Megatron-LM (NVIDIA) and DeepSpeed (Microsoft) are the dominant frameworks for orchestrating 3D parallelism.

---

## 3. Mixed Precision Training

Neural networks are typically trained with 32-bit floating point (FP32) weights. But FP32 is slow and memory-hungry.

**Mixed precision:** Use 16-bit floating point (FP16 or BF16) for most computations, keeping a FP32 "master copy" of weights for the gradient update step.

| Precision | Bits | Memory | Speed | Numerical Stability |
|---|---|---|---|---|
| FP32 | 32 | Baseline | Baseline | Most stable |
| FP16 | 16 | 0.5× | 2–8× faster | Can overflow for large values |
| BF16 | 16 | 0.5× | 2–8× faster | Better range than FP16; preferred for training |
| INT8 | 8 | 0.25× | 4× faster | Unstable for training; used for inference quantization |

**BF16 (Brain Float 16)** is the current standard for LLM training. It preserves the exponent range of FP32 (preventing overflow) while halving memory usage.

**PM implication:** Mixed precision training effectively doubles the model size you can fit on the same hardware at no quality cost. When teams upgrade GPU generations (A100 → H100), improved BF16 throughput is a key driver.

---

## 4. Gradient Checkpointing (Activation Recomputation)

During training, the forward pass stores all intermediate activations (the intermediate outputs of each layer) to compute gradients in the backward pass. For a large model, this can use more memory than the model weights themselves.

**Gradient checkpointing:** Discard some activations after the forward pass; recompute them when needed during the backward pass.

**The tradeoff:**
- Memory: significant savings (often 10× less activation memory)
- Compute: ~33% more FLOPs per training step (recomputing discarded activations)

**When it's used:** Almost universally in frontier model training — the memory savings are worth the compute cost.

**PM implication:** When engineers say "we need more memory for this run," gradient checkpointing is one lever before buying more hardware.

---

## 5. Optimizers and Gradient Updates

The optimizer is the algorithm that updates model weights using computed gradients.

### Adam / AdamW
The dominant optimizer for LLM training.
- Maintains running statistics (mean and variance) of gradients for each parameter
- Uses these statistics to adapt the learning rate per parameter
- **AdamW:** Adam with decoupled weight decay (prevents overfitting)

**Memory cost:** Adam stores 2 copies of the gradients (momentum + variance) per parameter. For a 70B model: ~420GB just for optimizer states. This is often larger than the model itself.

### ZeRO (Zero Redundancy Optimizer — Microsoft DeepSpeed)

ZeRO stages reduce optimizer memory by sharding optimizer states, gradients, and/or model weights across GPUs rather than replicating them:

| ZeRO Stage | What Is Sharded | Memory Reduction |
|---|---|---|
| Stage 1 | Optimizer states | 4× |
| Stage 2 | Optimizer states + gradients | 8× |
| Stage 3 | Optimizer states + gradients + model params | Linear with GPU count |

ZeRO Stage 3 makes training models too large for any single GPU possible on commodity clusters.

---

## 6. Learning Rate Scheduling

The learning rate (step size for each gradient update) is not constant during training.

**Common schedule:**
1. **Warmup:** Start with very small learning rate, ramp up over first ~1000 steps (prevents early instability)
2. **Cosine decay:** Gradually reduce learning rate following a cosine curve
3. **Final decay:** Often a sharp reduction near end of training

**PM relevance:** The learning rate schedule is one reason you can't just extend a training run cheaply after it ends — resuming after the learning rate has decayed requires careful handling. Training "from scratch" is usually more cost-effective than "continuing" for large jobs.

---

## 7. Training Stability Issues

At large scale, training runs frequently encounter instability:

| Issue | Symptom | Mitigation |
|---|---|---|
| **Loss spikes** | Training loss suddenly jumps up | Gradient clipping, reduce learning rate, discard bad batches |
| **NaN/Inf gradients** | Gradient becomes undefined | Gradient clipping, better numerical precision |
| **Hardware failures** | GPU dies mid-run | Checkpointing — save model state every N steps |
| **Data artifacts** | Bad data batch causes model degradation | Data quality filters, per-batch anomaly detection |

**Checkpointing:** Save the model's weights, optimizer states, and RNG state every N steps (typically every few hours). If a failure occurs, resume from the last checkpoint, losing only a few hours of compute.

**PM implication:** Large training runs are not "fire and forget." They require infrastructure monitoring and engineering on-call. A single uncaught hardware failure without checkpointing can waste millions of dollars of compute.

---

## 8. Hardware — The GPU Stack

| GPU | VRAM | FP16 Performance | Connectivity | Generation |
|---|---|---|---|---|
| NVIDIA A100 | 80GB | 312 TFLOPS | NVLink 3.0 | 2020 |
| NVIDIA H100 | 80GB | 989 TFLOPS | NVLink 4.0 | 2022 |
| NVIDIA H200 | 141GB | ~1,979 TFLOPS | NVLink 4.0 | 2024 |
| NVIDIA B200 | 192GB | ~4,500 TFLOPS | NVLink 5.0 | 2025 |
| Google TPU v4 | HBM | Purpose-built | 3D torus network | 2021 |
| Google TPU v5e | HBM | Purpose-built | High-bandwidth mesh | 2023 |

**TFLOPS** = Teraflops = 10^12 floating point operations per second. Higher = faster training.

**The bandwidth bottleneck:** GPU-to-GPU interconnect bandwidth matters as much as raw FLOPS. NVLink is fast within a node; InfiniBand connects nodes. Inter-node bandwidth is often the bottleneck for large distributed runs.

---

## 9. Compute Cost Estimation

A rough formula for estimating training compute:

```
C ≈ 6 × N × D

Where:
C = total FLOPs
N = model parameters
D = training tokens
```

This approximation (from Chinchilla) captures the forward pass + backward pass cost per token.

**Example:**
- 70B model trained on 2T tokens
- C ≈ 6 × 70 × 10^9 × 2 × 10^12 = 8.4 × 10^23 FLOPs
- At H100 theoretical: 10^15 FLOPs/sec × 8,000 GPUs × 50% utilization
- ~8.4 × 10^23 / (8 × 10^15) ≈ 105,000 GPU-hours ≈ 4,375 GPU-days

At $2–3/GPU-hour for H100: ~$200K–$300K for this run. (Frontier runs are much larger — GPT-4 estimated at $50–100M+)

---

## Key Terms

| Term | Definition |
|---|---|
| **Data parallelism** | Each GPU holds full model; processes different data; gradients averaged |
| **Tensor parallelism** | Model weight matrices split across GPUs (within-node) |
| **Pipeline parallelism** | Model layers split across GPUs sequentially (across-node) |
| **3D parallelism** | Combining all three parallelism strategies |
| **Mixed precision** | Using FP16/BF16 for compute while maintaining FP32 master weights |
| **BF16** | Brain Float 16 — preferred training precision; better numerical range than FP16 |
| **Gradient checkpointing** | Trading compute for memory by recomputing activations during backprop |
| **Adam / AdamW** | Dominant optimizer for LLM training; tracks gradient momentum per parameter |
| **ZeRO** | Zero Redundancy Optimizer — shards optimizer states across GPUs |
| **Checkpointing** | Saving model state periodically to recover from hardware failures |
| **FLOPs** | Floating Point Operations — unit of compute |
| **TFLOPS** | Teraflops (10^12 FLOPS/sec) — GPU throughput metric |
| **NVLink** | NVIDIA's high-bandwidth GPU-to-GPU interconnect within a server node |
| **InfiniBand** | High-speed network connecting GPU server nodes |
| **AllReduce** | Distributed operation to average gradients across all GPUs |
| **Megatron-LM** | NVIDIA's framework for large model training with 3D parallelism |
| **DeepSpeed** | Microsoft's framework for efficient distributed training (includes ZeRO) |

---

## Product Questions This Unlocks

1. "How long will the next training run take?" — Estimate from FLOPs formula, GPU count, and utilization.
2. "Why did the training run fail?" — Hardware failure (check checkpointing), loss spike, data artifact — three different mitigations.
3. "Can we iterate faster on experiments?" — Faster iteration requires smaller models, better infrastructure, or parallelizing experiments across clusters.
4. "What's the cost of this training run?" — GPU-hours × cost rate + infrastructure overhead.
5. "Why are we buying H100s instead of A100s?" — ~3× better FP16 throughput, more memory bandwidth, better NVLink — directly translates to faster training or larger models at same cost.

---

## Common PM Mistakes

- **"We can just restart the training run if something goes wrong"** — Without checkpointing, a failure loses everything. Checkpointing cadence is a safety requirement, not optional.
- **"More GPUs = proportionally faster training"** — Communication overhead (AllReduce, pipeline bubbles) means efficiency degrades as you add GPUs. The team must tune parallelism strategy.
- **"Training cost = GPU count × time"** — Also includes storage I/O, networking, cooling, memory bandwidth, and engineer time. Budget 20–30% overhead.
- **"Once training is done, we're done spending money"** — Inference at scale often costs more cumulatively than training.

---

*Lecture 5 of 12 — Stanford CME295 LLM Foundations | PM Reference*
