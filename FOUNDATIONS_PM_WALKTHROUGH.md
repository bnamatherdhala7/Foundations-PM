# Product Thinking Document
## Adobe Firefly — GenAI Foundations: Quality, Evaluation & Roadmap

**Author:** Bharat Namatherdhala
**Role:** Adobe Research & AI — Foundations PM
**Date:** May 2026

---

## TL;DR — 60-second read

1. **Firefly's quality problem is a measurement problem, not a capability problem.** FID and CLIP Score are the industry defaults. Both fail in specific, nameable ways. The team that knows the limits of its eval stack ships better models faster — and stops shipping bad ones.

2. **The eval stack should have three layers, not one.** Automated metrics for regression detection, deterministic pixel math for style consistency across batches, and human preference ELO as the final ship gate. Automated scores tell you direction. Human signal tells you arrival.

3. **Quality/cost/latency is a routing decision.** Route Express to a distillation model; reserve the foundation model for CC Pro. Flow matching brings Generative Fill under the 2-second professional threshold. This is not a tradeoff — it is a segmentation decision.

4. **The LoRA opportunity is underexploited.** Brand customization via LoRA adapters on the Firefly foundation is 10–100× cheaper than retraining. Custom Models is already this pattern. The gap is user education on training asset quality, not model capability.

5. **Adobe's moat is the combination, not any single piece.** IP indemnification + CC workflow integration + Custom Models + commercially safe training data. No competitor has all four. The window to build distance between Firefly and challengers is open now — Midjourney is in litigation, DALL-E 3 is locked inside ChatGPT, Imagen is enterprise-only. This is the moment to set the quality and eval bar that defines the category.

---

## 1. Customer Segments — Three Distinct Quality Bars

The most important product decision in the Foundations PM role is recognizing that "quality" is not one number. It is three different thresholds that cannot be collapsed into a single eval.

### Segment A — Creative Professionals (Photoshop, Illustrator, After Effects)

**What they do:** Production-grade work — campaigns, print, broadcast, editorial.
**Their quality bar:** Would a professional art director approve this without re-editing?
**Their failure mode:** Post-generation editing time. If Generative Fill requires 15 minutes of cleanup, the workflow failed regardless of the CLIP Score.
**What they actually need from Firefly:**
- Lighting coherence between generated and existing layers
- Edge quality that survives 300dpi print
- Subject consistency across a multi-panel campaign
- Latency ≤ 2 seconds — above this, they break flow and lose trust in the tool

**Insight from building for this segment (Stil):** Creative professionals don't describe quality in metric terms. They describe it as "this looks like it belongs." The pixel math that measures feed cohesion — color temperature, tonal range, contrast variance — captures that sense of belonging between a generated element and the rest of the composition. That measurement doesn't require a model.

---

### Segment B — Content Creators (Adobe Express, Premiere, Frame.io)

**What they do:** High-volume, platform-specific content — social, video, newsletters.
**Their quality bar:** Does this look good at 1080×1080 on a phone screen in under 10 seconds of creation?
**Their failure mode:** Not using the tool. They have a creativity bottleneck, not a skill gap. If the generation takes 15 keystrokes, they default to a template.
**What they actually need:**
- Speed over ceiling quality — distillation model is the right call
- Aspect ratio and platform format awareness built in (9:16 for Reels without asking)
- Style consistency across a set of posts — the "my feed" problem

**North star for this segment:** Posted Content Rate — did they actually publish it? Not generation count. Not quality score. A generation that sits unused is a workflow failure.

---

### Segment C — Business Users (GenStudio, Creative Cloud for Teams, Enterprise)

**What they do:** Brand-compliant asset production at scale — global campaigns, regulated industries, multi-market localization.
**Their quality bar:** Is this legally cleared, brand-consistent, and producible by a non-designer?
**Their failure mode:** Legal and brand review blocking production. A generated asset that requires lawyer review before use is not faster than stock photography.
**What they actually need:**
- IP indemnification visible at the moment of generation, not buried in T&Cs
- Brand Kit compliance — generated output stays within hex codes, tone registers, composition rules
- Custom Models that lock style reliably across a campaign, not just a single asset

**Insight from building for this segment (MailIntel, CI system):** Business users pay for AI that reduces decisions, not AI that creates them. The eval metric that matters is not quality score — it is whether the output went to production without revision.

---

## 2. The Quality Measurement Problem

The foundational challenge in Foundations PM is not building better models — it is knowing when you have one.

Most eval pipelines stop at two metrics. Both fail in specific ways you must be able to name when working alongside researchers:

| Metric | What it measures | Where it fails — say this to show depth |
|---|---|---|
| **FID** | Statistical distance between real and generated distributions | A model generating statistically average images scores well while failing every professional. Measures population similarity, not individual output value. |
| **IS** (Inception Score) | Object clarity + output diversity | Classifier-dependent; fails to detect poor diversity *within* a class. High IS, still looks like stock photo. |
| **CLIP Score** | Semantic alignment between text prompt and output | "Dramatic lighting" prompt scores well while the image feels flat. Measures alignment, not aesthetic quality. |
| **VBench** *(video)* | 16 dimensions: motion smoothness, flicker, subject consistency | Resource-intensive; not a regression metric for every build. |
| **FVD** *(video)* | Distribution similarity between real and generated clips | Inherits FID's blind spots — directional, not definitive. |

> ⭐ **The measurement gap:** None of these answer the question a professional actually asks — *"Will I use this output, or will I open Photoshop?"* That question requires human signal.

**Ship gate: Human Preference Rate ≥ 75%.** Below 60% is a coin flip. Shipping a coin flip destroys professional trust faster than not shipping.

**Two tests that catch what metrics miss — run these before any human eval:**
- **Blank Drop test:** Remove the image input. If model output doesn't degrade, the model wasn't reading the image — it was pattern-matching on text. This is a visual grounding failure that FID and CLIP Score will not surface.
- **Image Sensitivity test:** Swap the image while keeping the prompt identical. If output doesn't change appropriately, visual grounding is unreliable. LMMS-Eval provides a standardized, reproducible pipeline for both.

---

## 3. The Three-Layer Eval Stack

Designed so you pay for human signal only when the model is a candidate — not on every build.

```
LAYER 1 — Automated (every build, ~minutes)
  FID · CLIP Score · IS · temporal consistency (video)
  Purpose: regression detection before humans see anything
  Cost: near zero · Signal: directional

         ↓

LAYER 2 — Pixel math (every generation batch, sub-second)
  Color temperature variance · brightness · contrast · saturation
  Purpose: style consistency across a batch or campaign
  Cost: zero — deterministic, no model inference
  Signal: catches "these images don't belong together"
  Proof: Feed Cohesion Score (Stil) — same math, in production

         ↓

LAYER 3 — Human Preference ELO (before ship gates)
  Pairwise ranking → ELO scoring → preference rate
  Purpose: final ship/no-ship gate
  Cost: highest — paid only when model is a candidate
  Signal: definitive — users vote with judgment, not clicks
```

### Feedback Loop — closing the signal chain

The eval stack only compounds value if the signal flows back into model training. The architecture:

```
Generation → User behavior signal (dwell, re-edit, abandon)
           → Human preference sessions (pairwise ELO)
           → Quality regression report (per build, per segment)
           → Research team brief: "Segment A preference rate dropped
             3 points on lighting coherence — here are the 47 failing pairs"
           → LoRA adapter fine-tune targeting the specific failure mode
           → Re-eval against the same pairs
           → Ship gate cleared or not
```

> ⭐ **The key design decision:** The feedback loop is only useful if it is segment-specific. A creative professional's failing pair and a content creator's failing pair are different failures. Aggregating them into one preference number loses the signal entirely.

---

## 4. Architecture & Routing Decisions

### Segment-based model routing

| Segment | Model | Rationale |
|---|---|---|
| **Express / consumer** | Distillation model | Lower compute, higher throughput; quality bar is "good enough to post" not "good enough for print" |
| **CC Pro / Photoshop** | Foundation model | Quality threshold is higher; 2-second latency threshold; willingness to pay supports higher inference cost |
| **GenStudio / Enterprise** | Foundation model + LoRA adapters | Brand lock required; data residency may require self-hosted open-weight for IP-sensitive campaigns |

**Flow matching:** Reduces inference steps vs. DDPM diffusion → lower latency at the same quality level. The product impact: Generative Fill under 2 seconds for CC Pro. That threshold is where professionals adopt the tool rather than abandoning it.

### Fine-tuning strategy — LoRA, not retraining

| Approach | Relative cost | Time to ship | Use when |
|---|---|---|---|
| Full retrain | $$$$ | Months | Never for product iteration |
| LoRA fine-tune | $ (10–100× cheaper) | Days | Brand customization, style gap, segment-specific quality lift |
| QLoRA | $$ | Days | Memory is the bottleneck |
| Prompt engineering | Free | Hours | Proprietary API layer — the only lever without weight access |

**Custom Models as the product expression of LoRA:** Style IDs and Custom Models are already the LoRA pattern in market. The product gap: quality of brand lock depends on diversity and quality of training assets, not model architecture. That's a user education problem — and a PM problem — not a research problem.

### Hybrid architecture — the layer recommendation

> "Foundation model → Firefly. Commercially safe training data is the moat that no open-weight model replicates. Brand customization → LoRA on Firefly. Full retraining is millions of dollars. Domain grounding → RAG. Retrieval is cheaper than training and keeps outputs current. Prompt layer → few-shot templates for proprietary APIs where weight access isn't available."

---

## 5. Competitive Landscape — Real Benchmark Data

### Where Firefly stands today (Artificial Analysis Arena ELO, May 2026)

> **Version note:** Arena benchmarks reflect Firefly Image 3 (ELO 971) — the most recent version in public eval infrastructure as of May 2026. Firefly Image 5 launched October 2025 at Adobe MAX with native 4MP output, improved human anatomy rendering, better text rendering, and layered editing. Image 5 improvements are real but not yet reflected in arena scoring. The ELO baseline below is the signal the market sees — and the gap that matters for the roadmap.

| Model | ELO (text-to-image) | ELO (image editing) | Cost/image | Speed |
|---|---|---|---|---|
| **GPT Image 2 (high)** | 1339 | 1253 | $0.04–0.17 | ~10–25s |
| **Gemini 3.1 Flash Image** | 1265 | 1236 | — | — |
| **FLUX.2 dev Turbo** | 1160 | 1161 | $0.04–0.06 | ~6–12s |
| **Midjourney v7** | 1093 | — | $10–120/mo | ~30–60s |
| **Imagen 4 (Google)** | — | — | $0.02–0.05 | ~4–10s |
| **Adobe Firefly Image 5 (current)** | **971 (Image 3 baseline)** | **Not in top 5** | $5–10/mo | ~8–15s |

*Sources: [Artificial Analysis Text-to-Image Arena](https://artificialanalysis.ai/image/leaderboard/text-to-image) · [Artificial Analysis Image Editing Arena](https://artificialanalysis.ai/image/leaderboard/editing)*

**The gap, named precisely:**
- Firefly vs. GPT Image 2: **−368 ELO points** on text-to-image
- Firefly vs. Midjourney: **−122 ELO points** on aesthetic quality
- Firefly on image editing: **not ranked in the top 5** — the arena is dominated by GPT Image and Gemini

**Image 5 improvements that are not yet in arena scoring:**
- **Native 4MP resolution** — Image 3/4 required upscaling; Image 5 outputs natively at 4 megapixels
- **Improved text rendering** — meaningful improvement on typography accuracy (specific % pending internal eval)
- **Layered editing + "Prompt to Edit"** — instruction-following edits without masking, launched at MAX 2025
- **Better human anatomy** — reduced artifacts on hands, faces, anatomical accuracy under art direction
- **Cost reduction** — 10 credits/generation (Image 4 Ultra was 20) — doubles throughput at same spend

**Adobe's multi-model platform strategy (MAX 2025):**
Adobe also integrated FLUX 1.1, Imagen 4, Ideogram 3.0, Runway Gen-4, and Veo 3.1 at MAX 2025. Firefly is no longer positioned as the only model — Adobe is becoming a model orchestration platform that routes to the best model per task. This changes the competitive framing: the Foundations PM job is building the quality and eval layer that governs a fleet of models, not defending a single model against competitors.

**On HuggingFace (open-weight community signal):**
FLUX.1-dev leads with 12.9k likes and 716k downloads — the de facto open-weight standard. Firefly is absent (not open-weight), which means the developer and fine-tuning ecosystem is building on FLUX, not Firefly. This matters for Custom Models adoption and community-generated LoRA fine-tunes.

*Source: [HuggingFace text-to-image models by likes](https://huggingface.co/models?pipeline_tag=text-to-image&sort=likes)*

### Capability gap breakdown

| Capability | Firefly Image 5 | Best-in-class | Gap owner |
|---|---|---|---|
| **Aesthetic quality / artistic style** | Below Midjourney | Midjourney v7 (ELO 1093) | −122 ELO — training data curation |
| **Text rendering in image** | Improved in Image 5; <45% on Image 3 baseline | Ideogram 3.0 | LoRA gap, not foundation model — 90-day fix |
| **Prompt adherence / compositional accuracy** | Improved in Image 5 | FLUX Pro 1.1, GPT Image 2 | Object count, spatial relationships, color fidelity |
| **Image editing quality** | Not in top 5 | GPT Image 1.5 (ELO 1264) | **Highest urgency** — AI Studio is Firefly's product differentiator |
| **Commercial safety** | Best-in-class | — | Only model with full IP indemnification and licensed training data |
| **CC workflow integration** | Best-in-class | — | Lives in Photoshop, Express, GenStudio — no competitor matches |

### Competitor analysis

| Competitor | Strength | Limitation | Firefly's actual advantage |
|---|---|---|---|
| **GPT Image 2** (ELO 1339) | Highest quality; ChatGPT distribution | Locked in OpenAI ecosystem; no design tool integration; no brand customization | Firefly is where creatives work — Photoshop, Premiere, GenStudio |
| **Midjourney v7** (ELO 1093) | Best aesthetic quality; 20M+ community | Active Getty litigation; no IP indemnification; no CC integration; closed API | Enterprise cannot use Midjourney — legal exposure is real |
| **FLUX.1-dev** (12.9k HuggingFace likes) | Open-weight; LoRA fine-tuning under $2; developer ecosystem | No commercial safety guarantee; no enterprise support; requires MLOps | Firefly is turnkey — no infrastructure burden |
| **Imagen 4 (Google)** | Strong quality; cost-efficient ($0.02/image) | Enterprise-only; no creative tool integration; no consumer surface | Firefly reaches all three segments in one platform |
| **Sora / Runway / Kling (video)** | Strong video generation quality | No CC integration; no brand lock; no IP indemnification | Firefly Video integrates into Premiere and Frame.io |

> ⭐ **The honest assessment:** Firefly is not winning on ELO scores today. It is winning on trust and workflow integration — which matter enormously to the enterprise segment but are invisible in arena rankings. The roadmap question is: how do you close the ELO gap while preserving the integration and safety advantages that no competitor can replicate?

The 368-point ELO gap to GPT Image 2 is not a single problem — it is three separate problems: text rendering (<45% accuracy), image editing quality (not in top 5), and aesthetic quality ceiling (below Midjourney). Each has a different fix. Only one of them is a model research problem. The other two are product and data problems.

---

## 6. Capability Roadmap

*Each item is anchored to a specific ELO gap, benchmark finding, or segment pain point above. This is not a wish list — it is a ranked response to named deficits.*

### The three gaps to close, in priority order

1. **Image editing quality** — Firefly not in top 5 on editing arena (GPT Image 1.5 leads at ELO 1264). AI Studio is Firefly's product differentiator. This gap makes the differentiator undefended.
2. **Text rendering** — <45% accuracy on Image 3 baseline; improved but not at parity in Image 5. Text in designs (logos, social graphics, presentations) is a core creative use case. Ideogram 3.0 specializes here. This is a LoRA problem, not a foundation model problem.
3. **Aesthetic quality ceiling** — 122-point ELO gap to Midjourney. This is a training data curation and human preference feedback problem. Longer to close, but the feedback loop architecture is the lever.

---

### Tier 1 — 45–60 days (infrastructure, no model work)

| Initiative | Addresses | Expected impact |
|---|---|---|
| **Segment-specific eval contracts** | All gaps — removes ship/no-ship ambiguity | Research and product aligned on thresholds before next model update |
| **Blank Drop + Image Sensitivity tests on every build** | Visual grounding failures | Catches grounding regressions before human eval — currently undetected |
| **Pixel math consistency layer (Layer 2 eval)** | Editing quality, batch coherence | Sub-second, zero cost; catches "doesn't belong" that FID/CLIP miss |
| **Human preference ELO pipeline** | Aesthetic gap, editing gap | Without pairwise data feeding back to training, the gap doesn't close — this is the prerequisite |
| **Arena ELO tracking on every model release** | All gaps | Treat external arena rankings as a regression metric, not a vanity number |

---

### Tier 2 — 90 days (targeted model work)

| Initiative | Addresses | Expected impact |
|---|---|---|
| **LoRA fine-tune: text rendering** | <45% text accuracy gap | Targeted adapter on typography failure pairs; 10–100× cheaper than retraining; Ideogram-style accuracy achievable at adapter layer |
| **LoRA fine-tune: image editing coherence** | Not-in-top-5 editing gap | Fine-tune on instruction-following editing pairs; targets the GPT Image 1.5 ELO gap directly |
| **Flow matching rollout for CC Pro** | Latency — professional adoption threshold | Generative Fill under 2 seconds; above 2 seconds professionals abandon the workflow |
| **Feedback loop: ELO → training brief → re-eval ≤ 2 weeks** | Aesthetic quality, editing quality | Closes the compound signal chain — human preference sessions currently don't feed training |
| **Brand Kit compliance eval** | Enterprise segment — Custom Models reliability | Measurable brand lock threshold for adapter approval; drives enterprise adoption |

---

### Tier 3 — 6 months (foundation model + cross-team)

| Initiative | Addresses | Expected impact |
|---|---|---|
| **Foundation model quality lift: aesthetic ceiling** | 122-point Midjourney ELO gap | Requires training data curation investment — highest cost, highest ceiling; target: close gap to ≤ 60 ELO points |
| **Image editing foundation work** | Editing arena entry | Move Firefly into top 3 editing arena rankings; this is the AI Studio quality story |
| **Video temporal consistency as regression metric** | Video quality — flicker drives abandonment | Measurable per build; currently not tracked as a regression gate |
| **C2PA provenance platform-wide** | Trust, enterprise, competitive moat | Adobe co-founded C2PA; every Firefly output carries a Content Credential. Connect to Stock intake, CC Libraries, GenStudio. Engineering project, not research. |
| **Multimodal eval (image + text + layout)** | Segment A, C professional quality | As Firefly moves to multi-element generation, single-asset eval is insufficient |

---

### The 18-month ELO target

| Metric | Baseline (Image 3 arena) | Image 5 (est.) | 90-day target | 18-month target |
|---|---|---|---|---|
| Text-to-image ELO | 971 | ~1,020–1,040 | 1,060 (text rendering LoRA) | 1,150 (aesthetic lift + editing LoRA) |
| Image editing ELO | Not in top 5 | Not yet tracked | Top 5 entry | Top 3 |
| Text rendering accuracy | <45% (Image 3) | Improved; target >60% | >70% (LoRA adapter) | >85% |
| Human preference rate (CC Pro) | Baseline TBD | Baseline TBD | ≥ 65% | ≥ 75% (ship gate) |

> ⭐ **The roadmap thesis:** The 368-point ELO gap to GPT Image 2 is not one problem — it is three separate problems with three different owners. Text rendering is a data and LoRA problem (90 days). Image editing quality is a fine-tuning and eval pipeline problem (90 days). Aesthetic ceiling is a foundation model and training data problem (18 months). Framing the roadmap this way lets research and product work in parallel instead of waiting for a single "quality improvement" cycle.

---

## 7. Evidence: What I've Built

### Stil — Feed Cohesion Score
**JD alignment:** Scalable evaluation frameworks, quality measurement, experience with generative imaging

Built a 0–100 image quality consistency system for content creators using deterministic pixel math — color temperature, brightness, contrast, saturation variance. No model, no API cost, sub-second at scale. The design principle: pixel math is faster, cheaper, and more interpretable than a learned quality metric. You add a model only where pixel math can't reach.

**Direct Firefly application:** Measures style consistency across a generation batch. Does a set of Firefly outputs from the same prompt hold tonal range across seeds? Does a Custom Model produce brand-consistent outputs across 50 generations? Same math, different input set.

---

### Vigil + GSentinel — Agentic architecture in production
**JD alignment:** Technical depth, agentic workflow design, operating in research environments

Five multi-agent systems shipped. Consistent finding: FSM orchestration beats LLM orchestration for well-defined workflows — full audit trail, predictable behavior, no hallucinated state transitions. In Vigil, RAG-first retrieval cut hallucinated queries by routing known patterns through retrieval before generation. MTTR: 47 min → 35 sec. GSentinel auto-resolved 67% of incidents.

**Firefly application:** Heuristics and retrieval carry the load until they can't. Then pay for inference. Keep orchestration costs predictable at production scale.

---

### Content Trust Agent — C2PA + SynthID
**JD alignment:** Competitive landscape, platform integration, evidence-based decisions

Designed a provenance detection pipeline for Adobe Stock using C2PA manifest reading + Google SynthID detection at submission intake — bypassing contributor self-declaration. Meta does the same via IPTC Digital Source Type. Adobe co-founded C2PA; every Firefly output already carries a Content Credential. The gap is the engineering connection between Firefly's generation layer and the platform's intake and library systems.

---

### CI Competitive Intelligence Pipeline
**JD alignment:** Competitive landscape analysis, cost-aware product decisions

Automated competitive analysis system for Firefly positioning. Runs at $0.003 per report. Key finding: "commercially safe AI image generator" keyword cluster is factually Firefly's — and Firefly doesn't appear in it. That's a 90-day organic acquisition win requiring a CMS change and three landing pages, not an engineering project.

---

## 8. Data Strategy — Improving Firefly Without Customer Data

Adobe's trust position is explicit: Firefly is trained only on licensed, owned, and public domain content — not customer creative work. This is the enterprise moat. It is also the hardest constraint in the data strategy.

Three buckets. Every strategy for improving Firefly quality sits in one of them.

---

### Bucket A — Adobe-Owned Assets

Adobe owns some of the best-quality creative data on the planet. The gap is using it systematically as a training pipeline, not just as a legal safe harbor.

| Asset | Training signal | Gap today |
|---|---|---|
| **Adobe Stock** (contributor opt-in consent) | Aesthetic quality ceiling, style diversity, commercial composition | Consent program exists; systematic quality-tier training pipeline does not |
| **Adobe Fonts** (full library, 100% owned) | Typography training pairs — exact text rendering ground truth at every weight and size | <45% text accuracy while Adobe owns a complete, legally clean font corpus |
| **Adobe Color** (millions of validated palettes) | Color harmony signal — which combinations humans actually rate as coherent | Underused as color training signal for generative output |
| **Behance** (public portfolios, platform T&Cs cover training use) | Professional aesthetic intent — how designers compose at the quality ceiling | Not connected to Firefly training pipeline |
| **Adobe Express templates** (Adobe-designed, owned) | Layout, composition, brand-safe patterns | Adobe-designed reference set for instruction-following fine-tuning |
| **Stock contributor program (Tier 1)** | Top 1% contributors by quality score — offered revenue share for explicit AI training consent | Becomes the quality ceiling dataset; flywheel: better model → more Firefly content sold → more contributor revenue → more consent |

> **Highest-leverage move:** Adobe Fonts → text rendering LoRA. A targeted fine-tune on typography failure pairs using the full font corpus closes the text accuracy gap faster than any other approach — and no competitor has access to this dataset. Not a research project. A data pipeline project.

---

### Bucket B — Open Source Datasets, Open-Weight Models, and Research-Published Data

This is the most underused bucket. The open research community publishes benchmark datasets, preference corpora, and model weights that are freely usable — and several are specifically designed for fine-tuning image generation models.

#### Human Preference Datasets (for DPO / RLHF fine-tuning)

These are the most directly useful: real pairwise preference rankings on generated images, published by researchers.

| Dataset | What it contains | Firefly application |
|---|---|---|
| **Pick-a-Pic v2** (HuggingFace: `yuvalkirstain/pickapic_v2`) | 851k human pairwise preferences on text-to-image generations | Direct DPO training signal — fine-tune Firefly to prefer what humans prefer |
| **HPD v2 — Human Preference Dataset** | 798k human preference annotations on image generations | Aesthetic preference signal; combined with Pick-a-Pic for scale |
| **ImageReward** (Xu et al., 2023) | 137k text-image pairs with human preference + text-image alignment scores | Both preference and alignment signal in one dataset |
| **HPS v2 — Human Preference Score** | Trained on 798k preferences; released as model + dataset | Preference reward model — use as automated quality signal in Layer 1 eval |
| **ELO-ranked arena data** (Artificial Analysis, LMSYS methodology) | Pairwise model comparison results — published leaderboard data | Benchmark anchor for Firefly's position; track movement per training run |

> **Why this matters for Firefly specifically:** Pick-a-Pic v2 + HPD v2 together = 1.6M preference pairs. Running DPO on Firefly with this signal closes the aesthetic preference gap without generating a single new piece of data. These datasets were built explicitly for this purpose and are free to use.

#### Image Quality + Aesthetic Datasets

| Dataset | Size | Firefly application |
|---|---|---|
| **LAION-Aesthetics v2** (filtered LAION-5B, aesthetic score ≥ 6.25) | 600M pairs | General aesthetic quality baseline; curated subset used by SDXL |
| **DataComp-1B** | 1.4B curated image-text pairs | Designed for training; better filtering than raw LAION |
| **AVA — Aesthetic Visual Analysis** | 250k images with human aesthetic ratings 1–10 | Aesthetic score training signal |
| **JourneyDB** | 4M Midjourney v5 outputs + structured metadata | Midjourney-quality aesthetic patterns; prompts + outputs published by researchers |
| **OpenImages V7** (Google) | 9M images with dense annotations | Object detection, segmentation, spatial relationships — compositional accuracy |
| **COYO-700M** (Kakao Brain) | 700M filtered image-text pairs | Scale; alternative to LAION with better caption quality |

#### Text Rendering Datasets (closes the #2 priority gap)

| Dataset | What it contains | Firefly application |
|---|---|---|
| **TextCaps** | 145k captions describing text in images | Caption-level text rendering training pairs |
| **TextVQA** | 45k VQA pairs about text in images | Text recognition accuracy evaluation |
| **AnyText-benchmark** | Evaluation benchmark for text generation in images | Standard regression metric for text rendering accuracy |
| **LAION-OCR** (LAION subset) | Images containing readable text, filtered from LAION | Pre-training signal for text in scene understanding |
| **Emu Edit research data** (Adobe Research published) | Multi-task image editing pairs | Adobe's own research — Emu Edit paper released evaluation data |

#### Image Editing Datasets (closes the #1 priority gap — not in top 5 on editing arena)

| Dataset | Size | Firefly application |
|---|---|---|
| **InstructPix2Pix** (Brooks et al., UC Berkeley) | 310k instruction-based editing pairs | Synthetic but directly useful for instruction-following fine-tuning |
| **MagicBrush** | 10k real human-annotated editing instruction pairs | Highest quality editing pairs available — human-verified |
| **HIVE** (Human Instruction-driven Image Editing) | Multi-modal editing benchmark | Evaluation standard for editing quality |
| **EditBench** | 240 structured editing scenarios | Targeted regression for specific edit types |

> **Emu Edit is Adobe's own research published openly.** The evaluation methodology and dataset from Adobe Research's Emu Edit paper is available. If it is not already part of Firefly's editing fine-tune pipeline, that is a gap.

#### Open-Weight Models (for distillation and architecture signal)

| Model | License | What to extract |
|---|---|---|
| **FLUX.1-schnell** (Black Forest Labs) | Apache 2.0 — commercial use allowed | Architecture patterns, fast inference approach; strongest open-weight model |
| **FLUX.1-dev** | Non-commercial research | Use for output distillation (generate synthetic pairs with Adobe prompts); read community LoRA training research |
| **Stable Diffusion 3.5 Large** (Stability AI) | Open weights | MMDiT architecture research; SD3.5 papers publish training methodology details |
| **PixArt-Sigma** | Open weights | 4K resolution architecture; low-resource high-quality training approach |
| **AuraFlow v0.3** | Open weights | Competitive quality; training methodology published |
| **Kolors** (Kwai/Kuaishou) | Open weights | Strong aesthetic quality on non-Western styles — diversity gap signal |

#### Research Papers That Publish Training Methodology (read, don't copy)

High-ranked papers on HuggingFace and arXiv publish the training configurations, dataset compositions, and fine-tuning approaches that produced their quality gains. This is competitive intelligence, not IP exposure.

| Paper / Model | What the methodology reveals |
|---|---|
| **SDXL** (Rombach et al.) | Multi-aspect ratio training, refiner model architecture, dataset filtering approach |
| **DALL-E 3** (OpenAI technical report) | Caption recaptioning to improve prompt adherence — one of the highest-impact training improvements published |
| **Emu / Emu Edit / Emu Video** (Adobe Research) | Adobe's own published findings on quality-focused fine-tuning |
| **InstructPix2Pix** methodology | How to generate high-quality synthetic editing pairs from a base model |
| **IP-Adapter** | Image prompting architecture — directly relevant to Custom Models / Style ID mechanism |
| **DPO for image generation** (Wallace et al.) | DPO applied directly to text-to-image models — the paper that proves Pick-a-Pic works for this |

> **The DALL-E 3 insight is high-leverage:** OpenAI published that recaptioning training images with detailed, accurate captions (using GPT-4V) significantly improved prompt adherence. Adobe can apply this methodology to its own Stock images using its own models. The technique is open. The data is Adobe's.

---

### Bucket C — Synthetic Data and Behavioral Signal

When owned data runs out and open data has been used, the remaining signal comes from what the model generates and how users respond to it.

#### Synthetic preference pairs from the foundation model itself

- **Counterfactual degradation:** Take a high-quality Stock image. Degrade it (add noise, flatten contrast, corrupt text). That's a preference pair: clean = preferred. Near-zero cost, automated pipeline.
- **Hard negative mining:** Run the eval stack on Firefly outputs. Find the failure cases. Re-run with different seeds until one good output appears. Good/bad pair = training signal targeting the exact failure mode.
- **Self-critique DPO:** Use a capable VLM (GPT-4V or Claude) to rate Firefly outputs on a structured rubric. Low-rated outputs become the rejected side of DPO pairs. Human raters calibrate the VLM judge quarterly.

#### Behavioral reward signal (opt-in, aggregate, content-free)

| Signal | What it means | Data architecture |
|---|---|---|
| Dwell time ≥ 8s before re-run | Negative quality signal | `{event: "re_run", latency_ms: 8400}` — no content |
| Immediate export after generation | Positive quality signal | `{event: "export", latency_ms: 2100}` — no content |
| Prompt re-runs with same seed | Negative signal — output didn't satisfy | Count only, no prompt text |
| Explicit thumbs up/down (opt-in) | Direct preference signal | Aggregate per generation parameter set |

> **The architectural requirement:** A separate events pipeline strips content before the signal reaches the training team. Events carry timestamps, action types, and hashed session IDs. No images. No prompt text tied to identifiable users. Enterprise customers are excluded from this pipeline entirely under separate T&Cs.

---

### Consolidated Priority Table — All Three Buckets

| Priority | Bucket | Action | Timeline | Cost | Expected impact |
|---|---|---|---|---|---|
| **1** | A — Adobe-owned | Adobe Fonts → text rendering LoRA | 60 days | Low | +20–30 pts text accuracy |
| **2** | B — Open source | Pick-a-Pic v2 + HPD v2 → DPO fine-tune | 60 days | Very low — data is free | Aesthetic preference lift |
| **3** | B — Open source | MagicBrush + InstructPix2Pix → editing LoRA | 90 days | Very low | Editing arena entry |
| **4** | C — Synthetic | Counterfactual degradation pipeline | 45 days | Very low — automated | Hard negative coverage |
| **5** | A — Adobe-owned | DPO on Stock (contracted rater sessions) | 90 days | Medium | Quality ceiling lift |
| **6** | B — Open source | DALL-E 3 recaptioning methodology on Stock | 90 days | Medium — compute | Prompt adherence lift |
| **7** | C — Behavioral | Opt-in behavioral signal pipeline | 90 days | Medium — eng infra | Ongoing reward signal |
| **8** | B — Open-weight | FLUX.1-schnell distillation → Express model | 120 days | Medium | Latency + Express quality |
| **9** | A — Adobe-owned | Stock contributor Tier 1 consent program | 6 months | High | Long-term quality ceiling |

> ⭐ **The Foundations PM framing:** Three buckets. Adobe-owned data is the moat — it differentiates. Open source data is free signal that most teams don't fully exploit. Synthetic and behavioral data compounds over time. The right answer to "how do you improve quality without customer data" is not one strategy — it is a layered program across all three buckets running in parallel.

---

## 9. Success Metrics

| Metric | Target | JD alignment |
|---|---|---|
| **Human Preference Rate** *(ship gate)* | ≥ 75% per segment | Expert feedback on model quality |
| **Repeat Usage Rate at 7 days** | Baseline → +20% in 6 months | Feedback loops, customer value realized |
| **Post-generation editing time** | −30% for CC Pro segment | Customer insight → practical approach |
| **Task Completion Rate** | Baseline → +15% | Tangible customer results |
| **Inference cost per generation** | Track vs. segment routing model | Quality/cost/latency evidence-based view |
| **Preference rate feedback cycle time** | Human eval → training brief → re-eval ≤ 2 weeks | Scalable eval framework |
| **Brand lock reliability (Custom Models)** | ≥ 90% brand-compliant outputs per adapter | Enterprise segment quality bar |

---

## 10. Open Questions for the Interview



1. **Eval contract:** Is there a documented quality threshold per surface (Express vs. Photoshop vs. GenStudio)? Who currently makes the ship/no-ship call when thresholds are not defined?

2. **Feedback loop gap:** Where does human preference data go after a model eval session today — is it feeding back into training, or is it a one-time gate?

3. **Video regression coverage:** Is temporal consistency measured as a regression metric on every video model build, or only at major version checkpoints?

4. **C2PA integration scope:** Firefly outputs carry Content Credentials. Is there a roadmap to use that signal within the platform — Stock intake, Creative Cloud Libraries, GenStudio asset management?

5. **LoRA in production:** What does quality variance look like across Custom Model adapters trained on different asset sets? Is there a quality floor for adapter approval?

---

*Bharat Namatherdhala · May 2026*
