# Foundation Model — Full Development Pipeline
### Concrete Example: Datadog Toto 2.0 (May 2026)
> **TOTO = Time Series Optimized Transformer for Observability**
> Decoder Transformer · 5 sizes (4M → 2.5B) · 5.04T Training Points · Zero-Shot Observability Forecasting · Apache 2.0

---

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FOUNDATION MODEL DEVELOPMENT PIPELINE                │
│                    ─────────────────────────────────                    │
│                  Example: Datadog Toto 2.0 (May 2026)                   │
└─────────────────────────────────────────────────────────────────────────┘

  ┌───────────────────────────────────────────────────────────────────┐
  │  STAGE 1 │ PROBLEM DEFINITION                                     │
  │  ───────────────────────────────────────────────────────────────  │
  │  Task: Zero-shot forecasting on IT/observability time series      │
  │  Domain-specific bet: train only on ops metrics, outperform       │
  │  general-purpose models on observability data                     │
  │  Output: Probabilistic forecasts (9-quantile distribution)        │
  └──────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │  STAGE 2 │ DATA COLLECTION & PIPELINE                            │
  │  ───────────────────────────────────────────────────────────────  │
  │  Datadog internal (42.5%): ~2.1T pts — anonymized customer        │
  │  observability metrics (CPU, mem, error rates, latency, etc.)     │
  │  Synthetic (57.5%): ~2.9T pts via TempoPFN generation            │
  │  Zero public datasets used in pretraining (key design choice)     │
  │  Total: 5.04 trillion data points                                 │
  └──────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │  STAGE 3 │ TOKENIZATION & INPUT NORMALIZATION                    │
  │  ───────────────────────────────────────────────────────────────  │
  │  Patch size: 32 timesteps per token (same as Toto v1)            │
  │  Context: 4,096 steps → 128 tokens (8× longer than v1)          │
  │  Normalization: arcsinh transform (handles multi-OOM metrics)    │
  │  Key: arcsinh(x) ≈ log(2x) for large x — compresses spikes      │
  └──────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │  STAGE 4 │ ARCHITECTURE DESIGN                                   │
  │  ───────────────────────────────────────────────────────────────  │
  │  Factorized Space-Time Attention (1 space block : 2 time blocks) │
  │  Captures cross-series correlations + temporal patterns together  │
  │  5 model sizes: 4M, 22M, 313M, 1B, 2.5B (same architecture)    │
  │  Scaled via u-μP (unit-scaled maximal update parameterization)   │
  └──────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │  STAGE 5 │ PRETRAINING                                           │
  │  ───────────────────────────────────────────────────────────────  │
  │  Objective: Pinball loss on 9 quantile levels (not Student-T)   │
  │  Decoding: Contiguous Patch Masking — one forward pass           │
  │  (not autoregressive) — all forecast steps predicted in parallel │
  │  Optimizer: NorMuon (replaces AdamW from v1)                    │
  └──────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │  STAGE 6 │ EVALUATION                                            │
  │  ───────────────────────────────────────────────────────────────  │
  │  BOOM benchmark (Datadog internal): 350M obs, 2,807 prod series  │
  │  GIFT-Eval: top 3 among all foundation models on CRPS rank       │
  │  TIME benchmark: top 3 on every metric                           │
  │  vs. TimesFM (observability sMAPE): Toto 0.672 vs. 1.246        │
  └──────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │  STAGE 7 │ FINE-TUNING & ADAPTATION                              │
  │  ───────────────────────────────────────────────────────────────  │
  │  Zero-shot: directly usable on ops metrics (domain-matched)      │
  │  LoRA on smaller sizes (4M, 22M) for customer-specific baselines  │
  │  No RLHF: ground truth exists (actual future metric values)      │
  └──────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │  STAGE 8 │ INFERENCE & DEPLOYMENT                                │
  │  ───────────────────────────────────────────────────────────────  │
  │  Open-weight (Apache 2.0): HuggingFace Datadog/Toto-2.0-*       │
  │  TSFM.ai: hosted API for all 5 sizes                             │
  │  Not in Datadog production (research artifact as of May 2026)   │
  │  Size range: ~8MB (4M) → ~5GB (2.5B) — right-size for use case  │
  └──────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │  STAGE 9 │ MONITORING & FEEDBACK LOOP                            │
  │  ───────────────────────────────────────────────────────────────  │
  │  Calibration: do the 9 quantiles cover actual coverage rates?    │
  │  Concept drift: infrastructure behavior changes with deployments  │
  │  Flywheel: production actuals → new training pairs → fine-tune   │
  └──────────────────────────────┬────────────────────────────────────┘
                                 │
                                 └──────────────────────────────►  LOOP BACK TO STAGE 2
```

---

## Toto 1.0 → Toto 2.0: What Changed and Why

| Dimension | Toto 1.0 (Jul 2024) | Toto 2.0 (May 2026) | Why It Changed |
|---|---|---|---|
| **Parameters** | 103M (one size) | 4M, 22M, 313M, 1B, 2.5B | Scaling study — prove quality scales predictably |
| **Context length** | 512 steps | 4,096 steps | Longer infrastructure patterns (weekly cycles, slow drifts) |
| **Training data** | 1T pts (75% obs, 20% public, 5% synthetic) | 5.04T pts (42.5% obs, 0% public, 57.5% synthetic) | Removed public data; scaled synthetic via TempoPFN |
| **Output head** | Student-T Mixture Model | 9-quantile pinball loss | Numerical stability at scale; consistent with industry standard |
| **Decoding** | Autoregressive (step-by-step) | Contiguous Patch Masking (one forward pass) | 10–20× faster inference; enables parallel forecast |
| **Optimizer** | AdamW | NorMuon | Better gradient conditioning at large scale |
| **Normalization** | Per-patch standard norm | arcsinh transform | Handles multi-order-of-magnitude IT metrics (e.g. 1 req/s → 1M req/s) |
| **Scaling method** | Manual hyperparameter tuning | u-μP (unit-scaled maximal update parameterization) | Transfer hyperparameters from small to large model — cuts HPO cost |

---

## Stage-by-Stage Detail

---

### Stage 1 — Problem Definition
> **Duration:** Weeks | **Key output:** Domain-specific vs. general-purpose architecture bet

| Dimension | Toto 2.0 Decision |
|---|---|
| **Task** | Zero-shot probabilistic forecasting on IT/observability metrics |
| **Domain bet** | Train *only* on observability data — no general time series. Domain specificity beats breadth for the target use case. |
| **Input contract** | Any IT metric series, 4,096 steps context, any sampling rate |
| **Output contract** | 9-quantile distribution (p10, p20, ..., p90) for horizon H |
| **Five sizes** | Right-size the model to the deployment constraint: 4M for edge/on-prem CPU inference, 2.5B for max-quality cloud |
| **vs. TimesFM** | TimesFM bets on breadth (any domain). Toto bets on depth (ops metrics only). Toto wins by ~46% sMAPE on observability data. |

> **PM Decision Point:** The domain-specialization bet is a market segmentation decision, not just a technical one. By training exclusively on observability metrics, Datadog accepts that Toto will be worse than TimesFM on stock prices and weather. The return: dramatically better quality on the exact data Datadog's customers have. This is the product strategy: own one vertical deeply rather than covering all domains at average quality.

---

### Stage 2 — Data Collection & Pipeline
> **Duration:** Months | **Key output:** 5.04 trillion training points, no public datasets

```
Toto 2.0 Training Mix
─────────────────────
42.5% Datadog Internal Observability Metrics (~2.14T points)
  ├── CPU utilization per host/container
  ├── Memory usage (RSS, virtual, swap)
  ├── Network bytes in/out per interface
  ├── Request rates (p50, p95, p99 latency)
  ├── Error rates by service
  ├── Disk I/O and throughput
  └── Custom business metrics (APM spans, RUM sessions)

57.5% TempoPFN Synthetic Data (~2.90T points)
  ├── Trend + seasonality combinations
  ├── ARMA processes with varying orders
  ├── Spike and step-change injections
  ├── Composite multi-pattern series
  └── Rare edge cases (long flat periods, sudden regime changes)
```

**Why no public time series datasets (a key Toto 2.0 decision):**

| Reason | Explanation |
|---|---|
| **Distribution mismatch** | Public datasets (M4, ETT, weather) follow statistical distributions different from operational IT metrics. Including them may hurt more than help on the target domain. |
| **Contamination risk** | Public benchmarks used for evaluation (GIFT-Eval, TIME) partly overlap with public training datasets — removing them makes evaluation cleaner. |
| **Synthetic fills the gap** | TempoPFN generates high-quality synthetic time series that match the statistical properties of IT metrics without benchmark contamination. |

**Pipeline:**
```
Raw Datadog metrics (anonymized, customer-consented)
  → 1. Anonymize / PII strip (no customer IDs, IP addresses in training)
  → 2. arcsinh normalization (compress multi-OOM range)
  → 3. Patch into 32-step windows
  → 4. Quality filter (minimum length, remove flat/zero series)
  → 5. Mix with TempoPFN synthetic at 42.5/57.5 ratio
  → 6. Shuffle and batch
  → Training-Ready Dataset (5.04T points)
```

> **PM Decision Point:** Removing all public data is a bold call that most teams wouldn't make. It requires confidence that (a) your internal data is large and diverse enough and (b) synthetic data can fill gaps. Datadog's 5+ year dataset of billions of customer metrics series gave them that confidence. The payoff: no benchmark contamination + cleaner eval.

---

### Stage 3 — Tokenization & Input Normalization
> **Key innovation: arcsinh normalization for multi-order-of-magnitude IT data**

**The IT metrics challenge:**
```
CPU usage:          0% → 100%           (range: 100×)
Memory:             100MB → 500GB       (range: 5,000×)
Request rate:       1 req/s → 2M req/s  (range: 2,000,000×)
Error rate:         0.001% → 100%       (range: 100,000×)
```

Standard z-score normalization collapses these into a shared scale — but a spike from 1 req/s to 10,000 req/s and from 100k req/s to 200k req/s both matter operationally, even though their absolute change is vastly different.

**arcsinh normalization:**
```
arcsinh(x) = ln(x + √(x² + 1))

Properties:
  arcsinh(0)     = 0
  arcsinh(1)     ≈ 0.88
  arcsinh(10)    ≈ 3.00
  arcsinh(100)   ≈ 5.30
  arcsinh(1000)  ≈ 7.60
  arcsinh(1M)    ≈ 14.5

Effect: compresses large values logarithmically, preserves small-value
        precision, handles zeros cleanly (unlike log(x))
```

**Why not just log(x)?** Log is undefined at zero. Operational metrics frequently hit exactly zero (a service that's down, a network interface with no traffic). arcsinh handles zero as a first-class value.

**Patching (same as Toto v1 and TimesFM):**
```
4,096 raw values → 128 patch tokens (32 steps/patch)
Each patch → linear projection → d_model embedding
Context: 128 tokens (vs. 16 tokens in TimesFM at 512 steps)
```

> **PM Decision Point:** arcsinh normalization is a product quality decision that looks like an engineering detail. Without it, a model fine-tuned on low-traffic services predicts poorly on high-traffic services (orders-of-magnitude scale mismatch). With it, the same model weight handles a startup with 100 req/s and an enterprise with 10M req/s. This is what makes zero-shot generalization work across Datadog's customer base.

---

### Stage 4 — Architecture Design
> **Five sizes, one architecture, scaled via u-μP**

#### The Model Family

| Size | Parameters | Use Case |
|---|---|---|
| **Toto 2.0 Mini** | 4M | On-prem / edge / CPU inference; latency-critical |
| **Toto 2.0 Small** | 22M | Lightweight cloud; high-throughput batch |
| **Toto 2.0 Medium** | 313M | Production balance: quality vs. cost |
| **Toto 2.0 Large** | 1B | High-quality forecasting; premium tier |
| **Toto 2.0 XL** | 2.5B | Best quality; SLA-critical forecasting |

#### Factorized Space-Time Attention (Key Toto Innovation)

Standard transformers attend only across the time dimension. Toto's architecture alternates between two attention types:

```
Input: M metrics × T time patches

Time Attention Block (×2):
  Each metric attends to its own time history
  Captures: trends, seasonality, temporal autocorrelation

Space Attention Block (×1):
  Each time step attends across all M metrics simultaneously
  Captures: cross-metric correlations (CPU and memory spike together)

Pattern: [Time, Time, Space, Time, Time, Space, ...]
Ratio: 2 time blocks per 1 space block
```

**Why this matters for observability:**
A CPU spike and a corresponding memory spike in the same service 2 minutes later is a cross-metric temporal pattern. Pure time-only attention misses this. Pure space-only attention misses temporal structure. Factorized space-time captures both efficiently.

#### u-μP — Scaling Without Hyperparameter Re-Tuning

**The scaling problem:** For most transformer models, learning rate, initialization, and other hyperparameters must be re-tuned for each model size. This is expensive — you can't just make the model bigger.

**u-μP (unit-scaled maximal update parameterization):**
- Specific parameterization of weights and learning rates such that the optimal hyperparameters transfer across model sizes
- Tune hyperparameters on the 4M model (cheap); transfer to 2.5B (expensive to tune)
- Enables principled scaling without running costly HPO at each size

> **PM Decision Point:** u-μP is why Toto 2.0 can offer 5 model sizes as a coherent product line, not 5 separate R&D efforts. The PM benefit: you can set a quality-cost tradeoff per customer tier (Mini for edge, XL for premium) without paying 5× the tuning cost.

#### Full Forward Pass

```
M series × 4,096 steps each
       │
       ▼
  arcsinh normalization
       │
       ▼
  Patch (32 steps → 1 token) → 128 tokens per series
       │
       ▼
  Linear projection → d_model
       │
       ▼
  Positional encoding
       │
       ▼
  ┌─────────────────────────────────────────┐
  │  Factorized Space-Time Decoder Blocks   │
  │  [Time Attn] → [Time Attn] → [Space Attn] → ...  │
  └─────────────────────────┬───────────────┘
                            │
                            ▼
  ┌─────────────────────────────────────────┐
  │  Quantile Output Head                  │
  │  Predicts p10, p20, p30, ..., p90      │
  │  for each forecast horizon step        │
  └─────────────────────────┬───────────────┘
                            │
                            ▼
  Denormalize (inverse arcsinh)
       │
       ▼
  9-quantile forecast distribution per step
```

---

### Stage 5 — Pretraining
> **Key innovations: Contiguous Patch Masking + NorMuon optimizer**

#### Contiguous Patch Masking (CPM) — The Decoding Revolution

**Toto v1 (autoregressive):**
```
To forecast H steps:
  Step 1: Predict patch 1 → append → Step 2: Predict patch 2 → ... H steps
  Cost: H separate forward passes
  Latency: O(H) — slow for long horizons
```

**Toto 2.0 (CPM — Contiguous Patch Masking):**
```
Mask all forecast patches simultaneously:
  Input:  [context patches] [MASK] [MASK] [MASK] ... [MASK]
  Output: [---skip---     ] [p̂1  ] [p̂2  ] [p̂3  ] ... [p̂H  ]
  Cost: 1 forward pass regardless of horizon length
  Latency: O(1) — same cost for 1 step or 1,000 steps
```

**Why this works:** The model sees the full masked forecast window during training and learns to fill in all masked patches simultaneously — analogous to BERT's masked language modeling but applied to contiguous future patches.

**The speedup:** 10–20× faster inference for long-horizon forecasts. Critical for observability use cases where you might forecast 1,000+ steps (e.g., 7-day capacity planning at 10-minute granularity).

#### NorMuon Optimizer

NorMuon is a second-order optimization method that:
- Applies Nesterov momentum in the gradient direction
- Normalizes gradients using a matrix orthogonalization step (Neon algorithm)
- Better conditioning than AdamW for large transformer training
- Particularly effective at large scale where AdamW's learning rate sensitivity increases

**PM relevance:** Better optimizer → faster convergence to the same quality → lower training compute cost → more training runs per budget → faster iteration on the roadmap.

#### Quantile Loss (Replacing Student-T from v1)

```
Pinball Loss for quantile q:
  L_q(y, ŷ_q) = max(q × (y - ŷ_q), (q-1) × (y - ŷ_q))

Total loss = mean over 9 quantile levels (p10, p20, ..., p90)
             × all forecast horizon steps
             × all series in batch
```

**Why drop Student-T?**
- Student-T mixture is expressive but numerically unstable at very large scale
- Pinball loss is simpler, stable, and directly optimizes the quantile levels the product uses
- No difference in user-facing quality; significant difference in training stability at 2.5B parameters

| Setup | Value |
|---|---|
| Quantile levels | 9 (p10, p20, ..., p90) |
| Optimizer | NorMuon |
| Decoding | Contiguous Patch Masking (1 forward pass) |
| Training data | 5.04T points (large models) / 3.40T (small models) |
| Precision | BF16 |
| Scaling framework | u-μP across all 5 sizes |

---

### Stage 6 — Evaluation
> **BOOM is Datadog's internal benchmark — this is their held-out "real production" test**

#### BOOM Benchmark (Datadog's Core Eval)

**BOOM = Benchmark for Observability and Operational Metrics**

| Property | Value |
|---|---|
| Source | Real Datadog customer production data (anonymized) |
| Size | 350M observations |
| Series | 2,807 real production metric series |
| Coverage | CPU, memory, network, request rates, error rates, latency |
| Purpose | The held-out benchmark that prevents Goodharting on public datasets |

**Why BOOM matters:** All competing models (TimesFM, Moirai, Chronos) were evaluated on the same BOOM series. BOOM data was never in any model's training set. This is the cleanest apples-to-apples comparison available.

#### Public Benchmarks

| Benchmark | Toto 2.0 Result | Key Metric |
|---|---|---|
| **GIFT-Eval** | Top 3 among all foundation models | CRPS rank |
| **TIME benchmark** | Top 3 on every metric | Multiple |
| **BOOM (observability sMAPE)** | 0.672 (vs. TimesFM 1.246) | sMAPE |
| **LSF zero-shot** | MAE 0.312 / MSE 0.265 | Outperforms Moirai + TimesFM |

#### The Three-Tier Eval Stack

```
Tier 1 (Daily regression):
  MASE, CRPS, WQL on BOOM held-out window
  Alert if any metric degrades >5% vs. best checkpoint

Tier 2 (Per-model-size milestone):
  Full GIFT-Eval + TIME benchmark suite
  Compare all 5 sizes on same benchmark before release

Tier 3 (Ship gate):
  Domain expert review: do forecasts make operational sense?
  Calibration check: do p10/p90 quantiles cover 10%/90% of actuals?
```

> **PM Decision Point:** BOOM is Datadog's proprietary competitive moat in evaluation. Publishing a benchmark from real customer data (anonymized) is a strategic move — it makes the comparison on Datadog's terms. Competitors cannot optimize specifically for BOOM because they can't access the data. This is the evaluation equivalent of the IP moat.

---

### Stage 7 — Fine-Tuning & Adaptation

```
Pretrained Toto 2.0 (any size)
         │
         ├────────────────────────────────────────────────────────┐
         │                                                        │
         ▼                                                        ▼
  Zero-Shot Use                                       Domain Fine-Tuning
  (No changes)                                                    │
  Best for: Standard IT metrics                      ┌────────────┴────────────┐
  (CPU, memory, request rates)                       │                         │
  Already in training distribution                   ▼                         ▼
                                               LoRA on Mini/Small        Full Fine-Tune
                                               (customer-specific        (specialized infra:
                                               baselines, seasonality)   custom hardware,
                                               < 10MB per adapter        proprietary protocols)
```

**Model size selection guide for Splunk/observability:**

| Use Case | Recommended Size | Reasoning |
|---|---|---|
| Real-time anomaly detection (< 100ms) | Mini (4M) | Fits in RAM; CPU-inferrable |
| Per-customer baseline learning | Small (22M) | LoRA-fine-tunable; low storage |
| General-purpose ops forecasting | Medium (313M) | Best quality/cost balance |
| SLO breach prediction (SLA-critical) | Large (1B) | Better calibration at tails |
| Capacity planning, quarterly forecasts | XL (2.5B) | Best long-horizon accuracy |

---

### Stage 8 — Inference & Deployment

#### The CPM Inference Advantage

```
Toto v1 (autoregressive) — forecasting 100 steps:
  100 forward passes → ~1,000ms on GPU

Toto 2.0 (CPM) — forecasting 100 steps:
  1 forward pass → ~10–50ms on GPU
  Same for 1 step as 1,000 steps
```

**This changes what's economically viable at observability scale.** Datadog monitors hundreds of billions of metric time series. Autoregressive inference at that scale is prohibitive. CPM makes large-scale forecasting economically feasible.

#### Deployment Options

| Option | Model | Latency | Cost | Requirement |
|---|---|---|---|---|
| TSFM.ai hosted API | All 5 sizes | < 100ms | Pay-per-call | Internet access |
| Self-hosted GPU | Medium–XL | < 50ms | ~$2–3/GPU-hr | GPU server |
| Self-hosted CPU (quantized) | Mini, Small | < 500ms | Near zero | Standard CPU |
| On-prem air-gapped | Mini (INT8) | < 200ms | Hardware only | No cloud |

**Memory footprint (BF16):**

| Size | BF16 Memory | INT4 Memory | Min Hardware |
|---|---|---|---|
| 4M Mini | ~8MB | ~2MB | Any CPU |
| 22M Small | ~44MB | ~11MB | Any CPU |
| 313M Medium | ~626MB | ~157MB | 1 GPU or high-RAM CPU |
| 1B Large | ~2GB | ~500MB | 1× A10/3090 |
| 2.5B XL | ~5GB | ~1.25GB | 1× A100 |

> **PM Decision Point:** The 4M Mini model running at 8MB is a deliberate product decision for enterprise observability. Air-gapped environments (banking, government, healthcare) cannot call cloud APIs. An 8MB model that runs on a standard CPU means Toto 2.0 can be embedded in on-prem Splunk/Datadog agents without any GPU infrastructure. No competitor's general-purpose model fits in this envelope.

---

### Stage 9 — Monitoring & Feedback Loop

```
Production Forecasts
        │
        ▼
Actuals Arrive (next metric window)
        │
        ├─── Quantile Calibration Check ───────────────────────────────┐
        │    Does p10 cover 10% of actuals? p90 cover 90%?            │
        │    Drift > 3%? → Recalibrate output head                    │
        │                                                              │
        ├─── Accuracy Monitoring ─────────────────────────────────────┤
        │    MASE / CRPS on rolling 7-day window                      │
        │    Degrade > 10%? → Trigger fine-tuning                     │
        │                                                              │
        └─── Concept Drift Detection ─────────────────────────────────┘
             New service deployed? Traffic pattern shifted?
             → Retrain customer-specific LoRA adapter

Analyst Feedback (for alert-driven workflows):
  Alert confirmed → positive label
  Alert dismissed → negative label (false positive)
  Weekly batch → fine-tuning trigger
  This loop is the highest-leverage product investment
```

**Retraining triggers:**

| Signal | Threshold | Response |
|---|---|---|
| MASE degradation | > 10% on rolling 7-day window | Fine-tune on recent data |
| Quantile calibration drift | > 3% off target coverage | Recalibrate output scaling |
| New service type onboarded | Any new infra pattern | LoRA adapter for new domain |
| Major infrastructure change | Deploy, migration, scaling event | Force retrain signal for affected series |
| Scheduled refresh | Quarterly | Full fine-tune on updated data mix |

---

## Toto 2.0 vs. TimesFM — Head-to-Head PM Comparison

| Dimension | Datadog Toto 2.0 | Google TimesFM |
|---|---|---|
| **Domain** | IT/observability only | General-purpose (any domain) |
| **Parameter range** | 4M → 2.5B (5 sizes) | 200M (one size) |
| **Context length** | 4,096 steps | 512 steps (original) |
| **Training data** | 5.04T pts (42.5% real ops + 57.5% synthetic) | 100B pts (diverse public + Google internal) |
| **Public datasets in training** | None | Yes (M4, ETT, etc.) |
| **Decoding** | CPM (1 forward pass) | Autoregressive (H forward passes) |
| **Output** | 9-quantile pinball | Point + quantiles (p10, p50, p90) |
| **Normalization** | arcsinh (handles multi-OOM) | Per-patch z-score |
| **Scaling method** | u-μP (hyperparameters transfer across sizes) | Single size — no scaling family |
| **Observability sMAPE** | 0.672 | 1.246 (46% worse) |
| **On GIFT-Eval (general)** | Top 3 | Competitive but not specialist |
| **Open-source** | Apache 2.0 (HuggingFace) | Apache 2.0 (HuggingFace) |
| **Smallest model** | 4M params / 8MB BF16 | 200M params / 400MB BF16 |
| **In Datadog product** | Not yet (research as of May 2026) | Vertex AI (production) |
| **Best for** | Any IT/ops metric forecasting | General time series, non-IT domains |

**When to use Toto 2.0 over TimesFM:**
- Your data is IT/operational metrics (CPU, memory, error rates, latency)
- You need on-prem / air-gapped deployment (Mini at 8MB is CPU-runnable)
- You need long-horizon forecasts efficiently (CPM makes 1,000-step forecasts as fast as 1-step)
- You need multiple quality tiers for different cost points (5 sizes)

**When to use TimesFM over Toto 2.0:**
- Your time series are non-IT (weather, finance, retail demand)
- You're already on Google Cloud (Vertex AI integration)
- You need a stable, production-deployed API (TimesFM is in Vertex AI production; Toto 2.0 is not yet)

---

## Splunk / Observability Applicability

| Stage | Toto 2.0 | Splunk PM Application |
|---|---|---|
| **Stage 2 — Data** | 42.5% real ops metrics — the domain Splunk data lives in | Splunk customer log/metric data is in the same distribution as Toto's training data. Zero-shot performance is higher than TimesFM without any fine-tuning. |
| **Stage 3 — Normalization** | arcsinh handles multi-OOM IT metrics | Splunk metrics span orders of magnitude (1 event/hr to 1M events/sec). arcsinh normalization is directly applicable — consider this for any Splunk-native model. |
| **Stage 4 — Space-Time Attention** | Cross-metric correlation modeling | Service dependency correlation (CPU spike → memory spike → error rate spike) is exactly what Splunk analysts need. Toto's space attention captures this natively. |
| **Stage 5 — CPM decoding** | 1 forward pass for any horizon | For capacity planning queries in Splunk ("will this host run out of disk in 7 days?"), CPM makes long-horizon forecasting economically viable at query time. |
| **Stage 8 — 4M Mini model** | 8MB, CPU-inferrable | Enterprise Splunk customers in air-gapped environments (finance, DoD, healthcare) can run the Mini model inside their Splunk forwarder without any GPU infrastructure or cloud API call. |
| **Stage 9 — Analyst feedback** | Confirmed/dismissed alerts → training signal | Splunk analysts confirming or dismissing anomaly alerts is a free labeling pipeline. Wire this into Toto LoRA fine-tuning for continuous improvement per customer. |

---

## BOOM Benchmark — Why Datadog Published Their Own

Most AI benchmarks use public datasets. Datadog created BOOM specifically to:

1. **Measure what matters for their product** — public benchmarks (M4, ETT) don't contain IT infrastructure metrics
2. **Prevent benchmark contamination** — BOOM data has never been published, so no model can train on it
3. **Force honest comparison** — TimesFM, Moirai, Chronos, and Toto were all evaluated on the same BOOM series under the same conditions
4. **Own the evaluation narrative** — publishing BOOM means the community evaluates observability models on Datadog's terms

> **PM Takeaway:** If you're building models for a specialized domain, create a benchmark for that domain. A proprietary held-out benchmark is both a quality control tool and a competitive narrative asset. "Best on BOOM" is a defensible claim because BOOM is the benchmark that measures what your customers actually care about.

---

*Based on: Toto 1.0 — Cohen et al. (2024), arXiv 2407.07874 | Toto 2.0 — Khwaja et al. (2026), arXiv 2605.20119*
*PM Reference | Stanford CME295 LLM Foundations applied to Datadog Toto 2.0*
