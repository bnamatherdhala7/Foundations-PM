# Phase 1 — Data Pipeline Deep Dive
## How Raw Sources Become Training-Ready Embeddings

**Author:** Bharat Namatherdhala
**Role:** Adobe Research & AI — Foundations PM
**Date:** May 2026
**Context:** Foundation model data strategy — Adobe Firefly Image 5 + Video

---

## Why Data Pipeline Quality Is the Quality Ceiling

Every model can only be as good as what it was trained on. This is especially true for diffusion models where:
- The aesthetic ceiling is set by the highest-quality images in the training corpus
- Prompt adherence quality is directly tied to caption semantic density
- Text rendering accuracy is limited by how many text-containing training pairs exist
- Human preference alignment requires human preference *data* — which requires a dedicated pipeline to generate

The data pipeline is not a prerequisite to the work. It **is** the work. The research team can run the same architecture on better-curated data and produce measurable ELO improvements without changing a single model parameter.

---

## Step 1 — Source Inventory and Consent Governance

Before ingestion, every source needs a legal classification. This is a PM function, not an engineering function.

### The three consent tiers

| Tier | Definition | Training use | Examples |
|---|---|---|---|
| **Tier 0 — Owned** | Adobe creates or owns the asset outright | Unrestricted | Adobe Fonts, Adobe Color, Express templates |
| **Tier 1 — Consented** | Creator explicitly opted in to AI training use | Full training pipeline | Stock Tier 1 program, Behance explicit opt-in |
| **Tier 2 — Licensed / Public Domain** | License permits AI training (CC0, commercial license, platform T&Cs cover training) | Training permitted; review periodically | LAION-Aesthetics, Pick-a-Pic, CC0 datasets |

**Tier 1 Stock Contributor Program — the most important initiative in the data strategy:**

Adobe Stock has hundreds of millions of contributor-submitted images. The default contributor agreement does not grant training consent — it must be explicit. The program structure:

```
Identify: Top 1% contributors by quality score (estimated 50k–100k contributors)
Offer: Revenue share on Firefly-generated content that uses their style as training signal
  → Firefly generates → output sold on Stock → contributor receives royalty
Consent: Explicit opt-in form, revocable at any time
Result: Quality ceiling dataset — the same contributors whose work defines Stock's premium tier
```

The flywheel this creates: better model → more Firefly-assisted content accepted to Stock → contributor earns more → contributor renews and expands consent → better training data → better model.

**Revocation architecture (required by design):**
When a contributor revokes consent, their images must be removable from training without requiring a full pipeline rebuild. This requires:
- Every shard tagged with contributor IDs
- A revocation list checked against the shard manifest before each training run
- Images from revoked contributors excluded from new shards going forward
- If revocation is a legal requirement (GDPR, CCPA): retroactive removal policy must be defined

---

### Source registry schema

Every source should have a structured record:

```
source_id: adobe_stock_tier1_v2
display_name: Adobe Stock — Tier 1 Consent Program
consent_tier: 1
license_type: explicit_ai_training_consent
refresh_frequency: monthly
current_volume: 12.4M images (as of 2026-05-01)
quality_tier: ceiling (aesthetic score ≥ 7.5)
resolution_range: 2000px–8000px
recaptioned: true (Florence-2, 2026-03)
revocation_support: yes (contributor-level shard tagging)
owner: [PM name], [Legal contact]
last_reviewed: 2026-05-01
next_review: 2026-08-01
```

---

## Step 2 — Ingestion and Format Normalization

### Format normalization
Raw sources arrive in heterogeneous formats: JPEG, PNG, WebP, TIFF, RAW, PSD (Photoshop), AI (Illustrator), video frames. The pipeline must normalize all of these before downstream processing.

```
Input: any source format
Output: standardized storage format

Rules:
  - Convert RAW/TIFF to lossless WebP (preserves quality, reduces size 20–30% vs. TIFF)
  - Convert PSD/AI to flattened PNG at native resolution before further processing
  - Strip EXIF metadata at this stage (privacy; don't train on GPS coordinates)
  - Retain: image dimensions, original aspect ratio, source ID, contributor ID (hashed), consent tier
  - Store: {image.webp, image.json (metadata)}
```

### Metadata schema per image (minimal, purpose-limited)

```json
{
  "source_id": "adobe_stock_tier1",
  "contributor_hash": "sha256:...",
  "consent_tier": 1,
  "width": 4032,
  "height": 3024,
  "aspect_ratio": 1.333,
  "ingest_date": "2026-05-01",
  "quality_score": null,
  "caption": null,
  "embeddings": null
}
```

Fields populated by downstream stages. No image content stored in metadata.

---

## Step 3 — Deduplication at Scale

### Why this matters more than it looks

A dataset that is 30% near-duplicates is functionally a smaller dataset at 1.43× the cost. The model sees the same image multiple times, over-fits to it, and wastes gradient updates that could be learning from novel examples. LAION-5B was found post-publication to contain roughly 40% near-duplicates by semantic similarity.

### Two-pass deduplication

**Pass 1: Exact deduplication (cheap — run first)**
```
Algorithm: MD5 or SHA256 hash on raw bytes
Catches: exact copies, same image re-compressed at different quality settings
Cost: near zero — hash computation is fast even at billions of images
Tool: standard hashing library; no GPU needed
```

**Pass 2: Semantic deduplication (expensive — run after embedding generation)**
```
Algorithm: CLIP ViT-L/14 embedding → FAISS approximate nearest neighbor
Threshold: cosine similarity > 0.95 → near-duplicate → keep one, discard rest
Catches: cropped variants of the same image, recolored stock photo variants,
         same composition different lighting (common in Stock bracketed submissions)
Cost: requires embedding generation (Step 5) — run after embeddings exist
Tool: FAISS (Facebook AI Similarity Search) — handles billion-scale on GPU cluster
```

**Dedup within Adobe Stock contributor submissions:**
Stock contributors frequently submit bracketed shots — same scene, minor exposure or crop variation. These are the highest-value near-duplicates to catch: they inflate contributor weighting while adding no new signal.

```
Per-contributor dedup pass: run semantic dedup within each contributor's submission batch
Threshold: cosine similarity > 0.90 (slightly looser — preserve intentional variants)
Output: deduplicated per-contributor set before adding to general pool
```

---

## Step 4 — Quality Filtering

Quality filters define the floor below which no image enters training. The order matters — run cheapest filters first to avoid wasting compute on images that will be discarded anyway.

### Filter stack in application order

**Stage A — Hard gates (discard immediately, no further processing):**

```
Resolution: discard < 512×512
  Firefly Image 5 is 4MP native — training data minimum should be 1024×1024
  for Tier 1 quality ceiling dataset

Aspect ratio: discard extreme ratios (> 3.5:1 or < 1:3.5)
  These require extreme padding/cropping that destroys spatial relationships

Format validity: discard corrupted files, zero-byte files, truncated JPEG
```

**Stage B — Classifier-based filters:**

```
NSFW/safety filter: run Adobe's safety classifier
  Threshold: discard P(unsafe) > 0.1 (conservative — false negatives are worse)
  
Watermark detection: detect visible watermarks
  Do not train on watermarked images — model learns to generate watermarks
  Discard or route to "pending clearance" queue for legal review
  
Blur / compression artifacts: BRISQUE score (Blind/Referenceless Image Spatial Quality Evaluator)
  Threshold: BRISQUE > 40 → discard (severe quality degradation)
  
Aesthetic score: LAION aesthetic classifier (outputs 0–10)
  General pool threshold: ≥ 5.0
  Quality ceiling tier (Tier 1 Stock, Behance): ≥ 7.5
  Text rendering training subset: no aesthetic gate — require visible text instead
```

**Stage C — Content-specific routing (not discard — separate into specialized buckets):**

```
Text density: detect images with significant visible text
  Route to text rendering training bucket
  Adobe Fonts outputs always route here
  
Anatomy density: detect images with human figures (face + body)
  Route to anatomy-enriched training subset
  Addresses the hand/face artifact gap in Image 5
  
Typography: detect images with typographic design elements (logos, posters, type-heavy layouts)
  Route to typography training bucket
  Feeds the Photoshop / Illustrator segment quality
  
Color palette fidelity: detect images that match Adobe Color validated palettes
  Flag for color training signal pipeline
```

---

## Step 5 — Captioning and Recaptioning

**This is the highest-leverage optimization in the pipeline.**

### The captioning quality problem

Most image datasets are paired with captions that look like:
- `"woman at desk"` (stock image alt-text)
- `"IMG_4729.JPG"` (filename-as-caption — still exists in public datasets)
- `"A photo"` (generic auto-generated)

A model trained on these captions learns to respond to short, generic descriptions. When a creative professional writes: `"a woman in her early 40s wearing a blue linen blazer at a standing desk, warm afternoon window light, shallow depth of field, muted warm color palette"` — the model has never seen a training example that maps to that specificity.

The DALL-E 3 technical report identified caption quality as the single highest-impact training improvement, outperforming architecture changes and dataset scale increases.

### Recaptioning architecture

**Tier 1 (top 5M images — full structured captioning):**

```
Input: image
Model: Florence-2 (fast, open-weight, runs on A100 batch)
  or Claude Vision (spot-check quality validation on 10k sample)

Output schema:
{
  "subject": "A woman in her mid-30s wearing a cream linen shirt...",
  "count": "single subject",
  "spatial": "subject centered, background kitchen blurred at f/2.8",
  "lighting": "warm directional light from upper left, soft shadows",
  "color_palette": "cream, muted warm tones, natural wood",
  "style": "lifestyle photography, natural light, editorial quality",
  "mood": "calm, domestic, aspirational",
  "text_in_image": null,
  "quality_tags": ["sharp focus", "natural lighting", "professional composition"],
  "aesthetic_score": 8.2
}
```

The JSON schema is then flattened to a natural language caption for training:
> *"A woman in her mid-30s wearing a cream linen shirt holds a white ceramic cup with both hands. Warm directional morning light enters from the upper left, casting soft shadows on the wooden table. The background kitchen is softly blurred. Calm, domestic mood. Cream and muted warm color palette. Professional lifestyle photography quality."*

**Tier 2 (next 50M images — lightweight captioning):**
Florence-2 Florence-2 in fast mode: subject + lighting + style only. Full structured captioning too expensive at 50M scale in a single pass. Run full captioning on Tier 2 iteratively over 6 months.

**Tier 3 (general pool — existing captions + enhancement):**
Append quality tags and aesthetic score to existing captions. Do not replace — too expensive at hundreds of millions of images. Prioritize recaptioning the highest-quality images first.

### Adobe Fonts — special captioning: deterministic, zero ongoing cost

```
Input: font name + weight + style + text string
Output: rendered image at multiple sizes + exact text as caption

Pipeline:
  for each font variant in Adobe Fonts library (complete corpus):
    for each text_sample in text_sample_bank:
      render(font_variant, text_sample, sizes=[24px, 48px, 72px, 144px])
      caption = f"{text_sample} rendered in {font.name} {font.weight} {font.style}"
      save as training pair

This is fully automated. No VLM needed. The caption IS the ground truth.
Covers every typeface in Adobe's library at every weight and size.
No other model has access to this dataset.
```

**Text sample bank for font rendering:**
- Alphabet + digits + punctuation at character level
- Common words (English + 10 top languages — serves multilingual text rendering)
- Short phrases (headlines, call-to-action, UI labels)
- Paragraph text (body copy, accessibility-relevant font sizes)
- Special characters, ligatures, diacritics

---

## Step 6 — Embedding Generation

### What to embed and why

| Embedding | Model | Use case | Storage |
|---|---|---|---|
| Visual embedding | CLIP ViT-L/14 (OpenCLIP) | Semantic dedup · dataset retrieval · text-image alignment scoring | float32 · 768-dim per image |
| Text embedding | CLIP text encoder | Match captions to images for similarity search | float32 · 768-dim per caption |
| Aesthetic score | LAION aesthetic classifier | Quality tier assignment; training set weighting | float32 · scalar |
| Safety score | Adobe safety classifier | Pre-filtering confirmation | float32 · scalar |
| VAE latent | Firefly's own VAE encoder | **CACHE THIS — eliminates re-encoding at training time** | float16 · ~16KB–64KB per image at 1024px |

### VAE latent caching — the most important compute optimization

During training, every image must pass through the VAE encoder before entering the diffusion model. This is deterministic — the same image always produces the same latent. At scale, re-encoding on every epoch is pure waste.

```
One-time cost: encode all N images once
  N = 500M images
  Encoding time: ~0.5ms per image on A100
  Total: 500M × 0.5ms = ~70 GPU-hours (one-time)

Savings per training epoch:
  Re-encoding cost avoided: 70 GPU-hours per epoch
  10 epochs: 700 GPU-hours saved
  At H100 spot rate (~$2/hr): ~$1,400 saved per training run
  Monthly model updates: ~$16,800/year per model version

For video (spatial + temporal VAE):
  Video clips require encoding each frame
  Savings scale proportionally — video encoding is more expensive per clip than image
  Cache video clip latents per-scene to maximize reuse
```

**Storage requirement for cached VAE latents:**
```
Image at 1024px: ~32KB latent (float16)
500M images: ~16 TB (manageable on object storage at ~$0.02/GB/month = ~$320/month)
```

### FAISS index for semantic dedup and retrieval

After embeddings are generated, build a FAISS index for:
1. Semantic deduplication (find pairs with cosine similarity > 0.95)
2. Dataset retrieval during training (find the N most relevant training examples for a given prompt)
3. Hard negative mining (find generated outputs that are similar to high-quality training images — useful for self-critique DPO)

```
Index type: FAISS IVF-PQ (Inverted File + Product Quantization)
  IVF for fast approximate search at billion scale
  PQ for memory compression (768-dim float32 → 64-byte compressed)

Build time: ~8 GPU-hours for 500M embeddings (one-time)
Search speed: ~1ms per query at 500M scale (fast enough for training-time retrieval)
Memory: ~32 GB for 500M compressed embeddings (fits on a single A100 80GB)
```

---

## Step 7 — Dataset Construction: Mixing and Curriculum Design

### Aspect ratio bucketing

Training on a fixed square resolution (1024×1024) requires either center-cropping or padding every non-square image. Both approaches destroy content:
- **Center crop:** loses edges, destroys wide-angle compositions, cuts subject context
- **Padding:** introduces black borders that the model learns as a pattern

Aspect ratio bucketing groups images by their native aspect ratio and processes each bin at a matching resolution:

```python
# Example bucket definitions for Firefly training
BUCKETS = {
    "square":       (1024, 1024),   # 1:1
    "portrait_23":  (832,  1216),   # 2:3 — portrait photography
    "portrait_34":  (896,  1152),   # 3:4 — phone vertical
    "landscape_32": (1216, 832),    # 3:2 — landscape photography
    "landscape_43": (1152, 896),    # 4:3 — standard camera
    "widescreen":   (1344, 768),    # 16:9 — video/cinema aspect
    "ultrawide":    (1536, 640),    # 12:5 — cinematic crop
}
# Each image is assigned to nearest bucket by aspect ratio
# Batches are formed within buckets — no mixed-aspect batches
```

Why this matters for Image 5: Native 4MP output at various aspect ratios requires that the training data itself preserves native composition at multiple aspect ratios. Without bucketing, a 4MP widescreen output is trained on cropped portrait data — composition learning is degraded.

### Curriculum design — three stage mixing ratios

```
Stage 1 — Broad knowledge (first 20% of training compute)
  Purpose: general visual understanding, diverse concepts, broad style range
  Mix:
    40% DataComp-1B (filtered web data — broad concept coverage)
    30% LAION-Aesthetics v2 (aesthetic quality baseline)
    20% Adobe Stock general pool (quality signal without bias)
    10% specialty buckets (text rendering, anatomy, color)
  Total volume: 200M–400M unique images
  Resolution: 256px → 512px (progressive)

Stage 2 — Quality ceiling (next 50% of training compute)
  Purpose: aesthetic quality lift — the model learns what "great" looks like
  Mix:
    50% Adobe Stock Tier 1 (quality-ceiling dataset, aesthetic ≥ 7.5)
    20% Behance professional portfolios
    15% JourneyDB (Midjourney aesthetic quality reference)
    15% AVA aesthetic high-score subset (aesthetic 8–10)
  Total volume: 20M–50M images (smaller, higher quality)
  Resolution: 512px → 1024px

Stage 3 — Gap closure (final 30% of training compute)
  Purpose: targeted capability improvement for specific eval gaps
  Mix:
    30% DPO preference pairs (Pick-a-Pic v2 + HPD v2 + production pairs)
    25% Editing pairs (MagicBrush + InstructPix2Pix + Emu Edit)
    25% Text rendering pairs (Adobe Fonts pipeline output + TextCaps)
    10% Anatomy-dense subset (human figure quality improvement)
    10% Color fidelity (Adobe Color palettes + matching reference images)
  Total volume: 2M–5M pairs (all specialized, high signal)
  Resolution: 1024px → 4MP
```

**Mixing ratio as a hyperparameter:**
Track FID, CLIP Score, and human preference rate at each Stage boundary. Treat Stage ratios as tunable hyperparameters with their own eval loop. A 10% shift in Stage 2 Behance weighting can produce measurable ELO movement. Log every training run with its exact shard manifest — reproducibility is a quality standard.

---

## Step 8 — WebDataset Sharding and Streaming

### Why WebDataset over individual files

Training on individual image files at 500M scale means the filesystem is handling 500M small file open/read/close operations per epoch. On a distributed training cluster with 256 GPUs, each reading independently, this creates I/O bottlenecks that throttle GPU utilization.

WebDataset packages images into sequential tar archives (shards):
- **Sequential reads:** one large file read vs. millions of small ones → 10–20× I/O improvement
- **Streaming:** data loads from object storage (S3/GCS) directly into GPU memory — no "copy dataset to disk" step
- **Shuffle at shard level:** shuffle the order of shards per epoch, shuffle within each shard — statistically equivalent to full dataset shuffle
- **Portability:** any training cluster can read from any cloud storage with the same streaming code

### Shard structure

Each shard is a `.tar` file containing:
```
shard_0001.tar:
  ├── 000001.webp          (image)
  ├── 000001.txt           (structured caption)
  ├── 000001.npz           (cached VAE latent)
  ├── 000001.json          (metadata: source, consent tier, aspect ratio, quality score)
  ├── 000002.webp
  ├── ...
  └── 005000.webp          (~5000 samples per shard)
```

### Shard manifest and versioning

```yaml
# shard_manifest_v3_tier1_recaptioned.yaml
version: 3
created: 2026-05-01
total_shards: 2480
total_images: 12_400_000
sources:
  - adobe_stock_tier1: 10_200_000 images
  - behance_public: 1_800_000 images
  - adobe_fonts_text_pairs: 400_000 pairs
quality_tier: ceiling (aesthetic ≥ 7.5)
caption_status: florence2_structured_v2
vae_latents: cached (1024px)
aspect_ratio_bucketed: true
dedup_pass: semantic_v2 (cosine > 0.95)
revocation_checked: 2026-05-01
consent_tier_min: 1
legal_review: approved 2026-04-28
```

This manifest is the exact dataset used in a training run. Store it alongside the model checkpoint for reproducibility and audit.

---

## Step 9 — Feedback Reinjection Loop

The pipeline becomes a compounding flywheel when production signals generate new training pairs targeting specific failure modes.

### Channel A — Human preference DPO pairs

```
Human preference session (ELO pairwise rating, Label Studio or Scale AI)
  ↓
Session output: {prompt, winner_image_id, loser_image_id, rater_id, segment}
  ↓
Quality filter: discard sessions with rater agreement < 70% (noisy labels degrade DPO)
  ↓
Diversity check: deduplicate prompts (don't over-represent popular prompts)
  ↓
Add to DPO training set
  ↓
Next fine-tune run uses these pairs in Stage 3 mix

Volume target: 10k new high-quality pairs per month
Quality bar: rater agreement ≥ 70%, prompt diversity ≥ 80%
Cycle time: ≤ 2 weeks from session completion to training brief
```

### Channel B — Hard negative mining from production

```
Production event stream (aggregate, content-stripped, no PII)
  → identify prompts where user ran generation ≥ 3 times
     (behavioral signal: output did not satisfy — user kept retrying)
  ↓
Re-run those prompts with expanded seed range (50 seeds per prompt)
  ↓
VLM quality scoring on all 50 outputs per prompt
  → take top 1 (winner) and bottom 3 (losers)
  ↓
DPO pairs: {prompt, winner, losers} — targeting the exact failure mode users encountered
  ↓
Add to DPO training set with label "production_hard_negative"

Volume: depends on production usage — scales automatically as Firefly grows
Signal quality: high — these are real user failure cases, not synthetic
```

### Channel C — Synthetic augmentation for specific gaps

**Text rendering (ongoing — automated):**
```
Adobe Fonts → new font variants added → automatically generate new text rendering pairs
Run nightly, add to text rendering training bucket
No human review needed — ground truth is deterministic
```

**Anatomy (monthly):**
```
Review anatomy failure cases flagged by the Layer 2 eval
Pull matching high-quality anatomy-dense images from Stock Tier 1
Add to anatomy-enriched Stage 2 subset
```

**Color accuracy (quarterly):**
```
Adobe Color adds new validated palettes
Generate synthetic images constrained to exact palette
Add to color fidelity training bucket
```

---

## The Adobe Data Advantages No Competitor Has

| Asset | What it enables | Why competitors can't replicate |
|---|---|---|
| Adobe Fonts complete corpus | Text rendering LoRA — deterministic, ground-truth pairs | Competitors would need to license or generate equivalent; Adobe owns it outright |
| Adobe Stock Tier 1 consent program | Quality ceiling dataset — premium creative photography | Competitors don't have a contributor consent program tied to commercial revenue sharing |
| Behance portfolios | Aesthetic ceiling — how designers actually compose | Platform-specific; competitors would need their own creator network |
| Adobe Color | Color harmony training signal — validated human palettes | 10M+ human-validated palettes; not available as a training dataset elsewhere |
| C2PA provenance | Trust moat — every training image can be traced | Adobe co-founded C2PA; Firefly can prove training provenance at audit |
| EU AI Act documentation | Enterprise and regulatory requirement | Adobe's provenance logging architecture is already built; competitors building from scratch |

---

## Summary — What to Do First

Ordered by ROI, not by complexity:

1. **Adobe Fonts → text rendering pair generator** — automated, near-zero cost, closes the #2 capability gap directly. Not a research task. A data pipeline engineering task.

2. **Semantic deduplication pass on current corpus** — one-time run; likely removes 20–40% of near-duplicates; immediate quality lift at zero additional data cost.

3. **VAE latent caching** — one-time encoding of current training corpus; 15–25% training compute savings on every future run.

4. **Recaption top 5M Stock Tier 1 images** — Florence-2 at scale; highest-quality images get structured captions first; directly improves prompt adherence.

5. **Human preference ELO pipeline** — Label Studio deployment; 10k preference pairs/month; prerequisite for DPO and closing the aesthetic ELO gap.

6. **DPO fine-tune run on Pick-a-Pic v2 + HPD v2** — 1.6M preference pairs at zero acquisition cost; direct path to +30–50 ELO points.

7. **Tier 1 Stock contributor consent program** — PM-owned initiative; builds the flywheel; long-term compounding quality advantage.

---

*Related files:*
- *[AI_MODEL_LIFECYCLE_VISUAL.md](AI_MODEL_LIFECYCLE_VISUAL.md) — Visual 3-phase overview with Mermaid diagrams*
- *[FIREFLY_DATA_PIPELINE_STRATEGY.md](FIREFLY_DATA_PIPELINE_STRATEGY.md) — Firefly Image 5 full pipeline + video extension + compute cost*
- *[FOUNDATIONS_PM_WALKTHROUGH.md](FOUNDATIONS_PM_WALKTHROUGH.md) — Full product walkthrough*

*Bharat Namatherdhala · May 2026*
