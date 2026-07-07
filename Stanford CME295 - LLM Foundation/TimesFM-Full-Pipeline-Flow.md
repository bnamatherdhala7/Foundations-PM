# Foundation Model — Full Development Pipeline
### Concrete Example: Google TimesFM (Time Series Foundation Model, 2024)
> Decoder-only Transformer · 200M Parameters · 100B Training Points · Google TPU v4 · Zero-Shot Forecasting · Open-Weight

---

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FOUNDATION MODEL DEVELOPMENT PIPELINE                │
│                    ─────────────────────────────────                    │
│                     Example: Google TimesFM (2024)                      │
└─────────────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────┐
  │  STAGE 1 │ PROBLEM DEFINITION                        │
  │  ──────────────────────────────────────────────────  │
  │  Task: Zero-shot time series forecasting             │
  │  Input: 512 historical time steps (any domain)       │
  │  Output: Point forecast + quantiles (p10, p50, p90)  │
  │  Key bet: Foundation model beats specialized models  │
  └──────────────────────────┬───────────────────────────┘
                             │
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │  STAGE 2 │ DATA COLLECTION & PIPELINE                │
  │  ──────────────────────────────────────────────────  │
  │  Google Internal (~70B pts): Trends, YouTube, Ads    │
  │  Public (~30B pts): M4, ETT, Weather, Traffic        │
  │  Pipeline: Normalize → Clip → Window → Dedupe        │
  │  Key principle: Diversity > Volume                   │
  └──────────────────────────┬───────────────────────────┘
                             │
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │  STAGE 3 │ TOKENIZATION — PATCHING                   │
  │  ──────────────────────────────────────────────────  │
  │  512 raw values → 16 patch tokens (32 steps/patch)   │
  │  Each patch → embedding via linear projection        │
  │  Per-patch normalization (scale-agnostic learning)   │
  │  Same concept as ViT image patching, but 1D          │
  └──────────────────────────┬───────────────────────────┘
                             │
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │  STAGE 4 │ ARCHITECTURE DESIGN                       │
  │  ──────────────────────────────────────────────────  │
  │  Decoder-only Transformer (GPT-family, not BERT)     │
  │  16 layers · 32 attention heads · d_model=1024       │
  │  Raw → Patcher → Linear → Positional Enc             │
  │       → N× Decoder Blocks → Output Head (quantiles) │
  │  200M parameters total                               │
  └──────────────────────────┬───────────────────────────┘
                             │
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │  STAGE 5 │ PRETRAINING                               │
  │  ──────────────────────────────────────────────────  │
  │  Objective: Quantile Loss (not cross-entropy)        │
  │  Trains probabilistic forecasts, not point estimates │
  │  Hardware: Google TPU v4 · Optimizer: Adafactor      │
  │  Precision: BF16 · Scale: 5B training windows        │
  │  Note: 25× over Chinchilla-optimal → better serving  │
  └──────────────────────────┬───────────────────────────┘
                             │
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │  STAGE 6 │ EVALUATION                                │
  │  ──────────────────────────────────────────────────  │
  │  Tier 1 — Automated: MAE, MSE, MASE, WQL, CRPS      │
  │  Tier 2 — Benchmarks: M4, ETT, Weather, Traffic, ILI│
  │  All zero-shot (model never trained on these)        │
  │  Result: Beats supervised baselines on 8/11 datasets │
  └──────────────────────────┬───────────────────────────┘
                             │
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │  STAGE 7 │ FINE-TUNING & ADAPTATION                  │
  │  ──────────────────────────────────────────────────  │
  │  Zero-shot: Use pretrained model directly            │
  │  LoRA: Domain-specific adapters (<10MB each)         │
  │  Full fine-tune: 100K+ series, hours on single GPU   │
  │  No RLHF needed: ground truth exists (actual values) │
  └──────────────────────────┬───────────────────────────┘
                             │
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │  STAGE 8 │ INFERENCE & DEPLOYMENT                    │
  │  ──────────────────────────────────────────────────  │
  │  Managed: Google Vertex AI API                       │
  │  Open-source: HuggingFace google/timesfm-1.0-200m   │
  │  Self-hosted: 400MB at BF16 — fits on 1 A100        │
  │  Latency: <50ms per forecast · 1000s series/sec/GPU  │
  └──────────────────────────┬───────────────────────────┘
                             │
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │  STAGE 9 │ MONITORING & FEEDBACK LOOP                │
  │  ──────────────────────────────────────────────────  │
  │  Track MAE/calibration on live data as actuals land  │
  │  Retrain triggers: accuracy drop, new domain, drift  │
  │  Flywheel: actuals → labeled pairs → retrain         │
  └──────────────────────────┬───────────────────────────┘
                             │
                             └──────────────────────────────►  LOOP BACK TO STAGE 2
```

---

## Stage-by-Stage Detail

---

### Stage 1 — Problem Definition
> **Duration:** Weeks | **Key output:** Task contract + architecture bet

| Dimension | TimesFM Decision |
|---|---|
| **Task** | Zero-shot time series forecasting across all domains |
| **Input contract** | Any time series, any domain, any sampling frequency; 512 steps context |
| **Output contract** | Point forecast + calibrated quantiles (p10, p50, p90) for horizon H |
| **Model type** | Foundation model (train once, generalize) vs. specialized per-dataset |
| **Why this bet** | A large diverse-data model beats small specialized models at zero-shot — the same bet GPT-3 made for text |
| **Alternative rejected** | N-BEATS, N-HiTS, ARIMA per dataset — requires domain-specific training for every new time series |

> **PM Decision Point:** Choosing "foundation model" over specialized models trades 10–100× more training compute for zero-shot generalization. The return: no per-customer labeling, no per-domain retraining, instant deployment for new time series types.

---

### Stage 2 — Data Collection & Pipeline
> **Duration:** Months | **Key output:** 100B clean, diverse time series points

| Source | Volume | Type | Why Included |
|---|---|---|---|
| Google Trends | ~20B pts | Web query volumes | Diverse domains; seasonal patterns |
| YouTube metrics | ~15B pts | View counts, engagement | High-volume behavioral data |
| Google Ads | ~15B pts | Impression/click series | Business metrics; spiky patterns |
| Internal infra | ~20B pts | Service metrics | Technical monitoring patterns |
| M4 competition | ~5B pts | Mixed public domains | Benchmark diversity |
| ETT / Weather / Traffic | ~10B pts | Physical + infrastructure | Domain coverage |
| Finance OHLCV | ~5B pts | Stock/commodity prices | Volatile, trend-heavy patterns |
| Other public datasets | ~10B pts | Various | Long-tail coverage |

**Pipeline steps:**
```
Raw Series
  → 1. Normalize (zero-mean per series)
  → 2. Clip outliers (±3σ)
  → 3. Window into (context, forecast) pairs
  → 4. Deduplicate
  → 5. Train / Val / Test split
  → Training-Ready Dataset
```

> **PM Decision Point:** Data mix drives zero-shot generalization. Google's moat is internal behavioral data (Trends, YouTube) — diverse, high-volume, unavailable to competitors. This is the data flywheel: more diverse domains → better generalization → stronger product.

---

### Stage 3 — Tokenization (Patching)
> **Key insight:** Time series values are not discrete tokens — they need a different approach

```
Raw: [v1, v2, v3, ..., v512]          ← 512 raw float values
         │
         ▼  Patch (window of 32)
Patches: [v1–v32], [v33–v64], ..., [v481–v512]   ← 16 patches
         │
         ▼  Linear projection + per-patch normalization
Tokens:  [e1],    [e2],    ...,    [e16]           ← 16 embedding vectors
         │
         ▼  Enter Transformer
```

| Design Choice | Value | Reasoning |
|---|---|---|
| Patch size | 32 steps | Reduces 512 tokens → 16 tokens; 32× cheaper attention |
| Projection | Linear layer | Lightweight; learned end-to-end |
| Normalization | Per-patch (reversible) | Model learns patterns regardless of scale (CPU % vs. stock price) |
| Analogy | ViT image patching | Same concept applied to 1D signal instead of 2D image |

> **PM Decision Point:** Patching is the tokenization equivalent of BPE for text — reduces sequence length (cheaper attention) while preserving local temporal structure. Patch size 32 is a hyperparameter: smaller = more detail, more tokens; larger = coarser but faster.

---

### Stage 4 — Architecture Design
> **200M parameters · Decoder-only · Quantile output head**

```
Input Time Series
       │
       ▼
  ┌──────────┐
  │ Patcher  │  32-step windows → 16 patch tokens
  └────┬─────┘
       │
       ▼
  ┌──────────────┐
  │Linear Proj.  │  patch vector → d_model=1024
  └────┬─────────┘
       │
       ▼
  ┌────────────────────┐
  │ Positional Encoding│  RoPE (relative position)
  └────┬───────────────┘
       │
       ▼
  ┌─────────────────────────────┐
  │   Transformer Decoder ×16   │
  │  ┌─────────────────────┐    │
  │  │ Masked Self-Attention│   │   (can only attend to past patches)
  │  └──────────┬──────────┘    │
  │             ▼               │
  │  ┌─────────────────────┐    │
  │  │  Feed-Forward Net   │    │
  │  └──────────┬──────────┘    │
  │             ▼               │
  └─────────────┬───────────────┘
                │
                ▼
  ┌─────────────────────────────┐
  │    Output Head              │
  │  Point forecast + p10/p50/p90 quantiles per horizon step │
  └─────────────┬───────────────┘
                │
                ▼
  ┌─────────────────────────────┐
  │   Denormalize               │  restore original scale
  └─────────────────────────────┘
```

| Hyperparameter | Value |
|---|---|
| Parameters | 200M |
| Layers | 16 |
| Attention heads | 32 |
| d_model | 1024 |
| Patch size | 32 |
| Context length | 512 steps (= 16 tokens) |
| Output | Point + quantiles (p10, p50, p90) |

> **PM Decision Point:** Why decoder-only? Forecasting is generative — you generate future values conditioned on past. This maps to GPT's autoregressive paradigm. Encoder-only gives representations but can't generate. Decoder-only is the right choice.

---

### Stage 5 — Pretraining
> **Key difference from LLMs: Quantile Loss, not Cross-Entropy**

```
For each training window:
  Context: [v1 ... v512]
  Target:  [v513 ... v512+H]

Loss = Σ over quantile levels q:
  Pinball(q, predicted_q, actual)

Pinball(q, ŷ, y):
  if y ≥ ŷ: q × (y - ŷ)
  else:      (1-q) × (ŷ - y)
```

**Why quantile loss?**
- Trains probabilistic outputs — the model learns uncertainty, not just point estimates
- Confidence intervals are directly actionable in monitoring (is this spike within the 90th percentile of normal?)
- No softmax over a vocabulary — output is a continuous distribution over future values

| Setup | Value |
|---|---|
| Hardware | Google TPU v4 pods |
| Training data | 100B points (~5B context-forecast windows) |
| Optimizer | Adafactor (memory-efficient Adam variant) |
| Precision | BF16 |
| Chinchilla-optimal | ~4B tokens for 200M params |
| Actual training tokens | ~100B — 25× over-trained |
| **Why over-train?** | Inference is cheap; over-training yields better small model quality at serving time |

> **PM Decision Point:** The quantile output is a product strategy decision. "Is this CPU spike normal?" is far more actionable with "90% confident it's within normal range" vs. just a point forecast. Uncertainty = confidence interval = operator can set thresholds.

---

### Stage 6 — Evaluation
> **Three tiers — never use a single metric**

#### Tier 1 — Automated Metrics

| Metric | Full Name | What It Measures |
|---|---|---|
| **MAE** | Mean Absolute Error | Average magnitude of forecast error |
| **MSE** | Mean Squared Error | MAE but penalizes large errors more |
| **MASE** | Mean Absolute Scaled Error | Error normalized by a naïve baseline — makes cross-series comparison fair |
| **WQL** | Weighted Quantile Loss | Quality of the full probabilistic forecast distribution |
| **CRPS** | Continuous Ranked Probability Score | How well the predicted distribution matches actual outcomes |

#### Tier 2 — Zero-Shot Benchmark Suite

| Benchmark | Domain | Series Count |
|---|---|---|
| M4 | Mixed (finance, macro, micro, industry, demo, other) | 100,000 series |
| ETT | Electricity transformer temperature | 7 series, long horizon |
| Weather | 21 meteorological indicators | 21 series |
| Traffic | Highway occupancy rate sensors | 862 sensors |
| ILI | Influenza-like illness rates | 7 series |
| Exchange | Currency exchange rates | 8 series |

**Result: TimesFM wins 8 of 11 benchmarks zero-shot — no domain-specific training.**

#### Tier 3 — Human Validation
For observability use cases: can a domain expert confirm the forecast makes operational sense? This is the equivalent of human ELO for image generation.

> **PM Decision Point:** Goodhart's Law applies — after publishing MASE scores, teams optimize for MASE specifically. TimesFM uses multiple benchmarks across diverse domains to prevent metric gaming. The held-out benchmarks were never in training data — equivalent of a contamination-free eval set.

---

### Stage 7 — Fine-Tuning & Adaptation

```
Pretrained TimesFM
        │
        ├──────────────────────────────────────────────────────┐
        │                                                      │
        ▼                                                      ▼
Zero-Shot Use                                         Domain Adaptation
(No changes)                                                   │
                                                    ┌──────────┴──────────┐
                                                    │                     │
                                                    ▼                     ▼
                                               LoRA Adapter          Full Fine-Tune
                                             (< 10MB per domain)   (Update all 200M params)
                                               │                         │
                                               │ Requires: 1K+ series    │ Requires: 100K+ series
                                               │ Cost: Minutes–hours     │ Cost: Hours on 1 GPU
                                               │ Use: Industry-specific  │ Use: Proprietary patterns
                                               │ baselines               │ with unique structure
```

| Method | Data Needed | Cost | Quality |
|---|---|---|---|
| Zero-shot | None | Zero | Good for standard patterns |
| LoRA | 1K–10K series | Hours | Better for domain-specific patterns |
| Full fine-tune | 100K+ series | Days | Best for highly specific domains |

**No RLHF required:** Ground truth (actual future values) exists. The quantile loss IS the objective alignment. No human preference labeling needed — this is unique to regression/forecasting tasks vs. language generation.

> **PM Decision Point:** LoRA per customer cluster type (not per customer) is the right granularity at scale. Thousands of enterprise customers → cluster by infrastructure type (K8s, VMs, bare-metal) → one adapter per cluster. Storage: ~10MB each.

---

### Stage 8 — Inference & Deployment

```
Request: time series context [512 values]
        │
        ▼
   Feature Extraction
   (patch, normalize)
        │
        ▼
   Model Forward Pass
   (~50ms on single GPU)
        │
        ▼
   Output: point forecast + quantiles
   (p10, p50, p90 for each horizon step)
        │
        ▼
   Denormalize → Return to caller
```

| Serving Option | Latency | Cost | Use Case |
|---|---|---|---|
| Google Vertex AI | < 100ms | ~$0.001–0.005/call | Managed, no infra |
| HuggingFace Inference API | ~200ms | Pay-per-use | Prototyping |
| Self-hosted (A100) | < 50ms | ~$2–3/hr → millions of forecasts/$1 | Production at scale |
| CPU (quantized INT8) | ~500ms | Near zero | Batch / on-prem |

**Memory footprint:**

| Precision | Size | Hardware Needed |
|---|---|---|
| BF16 | ~400MB | Any single GPU |
| INT8 | ~200MB | CPU-capable |
| INT4 | ~100MB | Edge / on-prem |

> **PM Decision Point:** 200M parameters was a deliberate product decision. At GPT-4 scale, per-forecast cost would be prohibitive for high-frequency monitoring. Smaller purpose-built model = economically viable at observability scale (billions of metrics/day).

---

### Stage 9 — Monitoring & Feedback Loop

```
Production Forecasts
        │
        ▼
Actuals Arrive (future becomes present)
        │
        ▼
Accuracy Measured per series / domain
        │
        ├─── Within SLA? ──► Continue monitoring
        │
        └─── Degraded? ──► Trigger Analysis
                                  │
                          ┌───────┴────────────┐
                          │                    │
                          ▼                    ▼
                   New domain type?     Concept drift?
                   Acquire data          Retrain on
                   for that domain       recent windows
                          │                    │
                          └──────────┬─────────┘
                                     │
                                     ▼
                             Fine-tune / Retrain
                                     │
                                     ▼
                              Evaluate on held-out set
                                     │
                                     ▼
                              Ship → Back to Production
```

**Retraining triggers:**

| Trigger | Threshold | Action |
|---|---|---|
| MAE degradation | > 10% from baseline on rolling 7-day window | Fine-tune on recent data |
| Calibration drift | Quantile coverage off by > 5% | Recalibrate output head |
| New domain added | Any new infrastructure/data type | LoRA adapter for new domain |
| Scheduled refresh | Quarterly | Full fine-tune on updated data mix |

> **PM Decision Point:** The feedback loop is the most important infrastructure investment after the model itself. Production actuals → labeled training pairs → periodic retraining → better model. Define retraining triggers before launch; they belong in the product spec.

---

## The Closed Feedback Loop

```
Production         Actuals         Failure Cases      Data Gap         Retrain
Forecasts    →     Arrive     →    Collected     →    Identified   →   / Fine-tune
    ↑                                                                       │
    └───────────────────────────────────────────────────────────────────────┘
```

---

## Splunk / Observability Applicability

| Stage | TimesFM | Splunk Application |
|---|---|---|
| **Stage 2 — Data** | Google Trends, YouTube behavioral signals | Customer log + metric time series. The flywheel is already there — it needs the pipeline to convert forecast errors into training examples. |
| **Stage 3 — Tokenization** | 32-step patch windows | Log events are already time-indexed. Patch-based tokenization maps to Splunk's bucket/window aggregation model. |
| **Stage 5 — Objective** | Quantile loss → calibrated confidence intervals | For alert thresholds, "90% confident this won't exceed X" is directly actionable for operators. |
| **Stage 6 — Eval** | MASE, WQL on held-out benchmarks | For security metrics: TTD (time-to-detect), FPR (false positive rate), calibration score — operational metrics that matter more than raw MAE. |
| **Stage 7 — Fine-tuning** | LoRA per domain | LoRA adapter per customer cluster type (K8s, on-prem, cloud). Start from TimesFM open weights, fine-tune on Splunk-specific patterns. 10MB per adapter = viable at enterprise scale. |
| **Stage 8 — Inference** | 400MB self-hostable | Enterprise customers in regulated industries (financial services, healthcare) often cannot send telemetry to cloud APIs. Self-hosting is a hard requirement. |

---

*Based on: Das et al. (2024) "A decoder-only foundation model for time-series forecasting" — Google Research*
*PM Reference | Stanford CME295 LLM Foundations applied to Splunk Observability*
