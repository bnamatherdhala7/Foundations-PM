# Product Thinking Document
## Adobe Firefly — GenAI Quality, Evaluation & Foundations

**Author:** Bharat Namatherdhala
** Adobe Research & AI — Foundations
**Date:** May 2026

---

## TL;DR — 60-second read

1. **Firefly's quality problem is not capability — it's measurement.** FID and CLIP Score are the industry defaults. Both are wrong in specific, nameable ways. The team that knows their limits builds better models faster.

2. **The three-layer eval stack:** automated metrics for direction, deterministic pixel math for style consistency, human preference ELO as the final ship gate. Automated scores tell you direction. Human signal tells you arrival.

3. **Quality/cost/latency is a routing decision, not a tradeoff.** Route Express users to a distillation model. Reserve the foundation model for CC Pro. Flow matching brings Generative Fill under the 2-second professional threshold where adoption actually happens.

4. **The LoRA opportunity is larger than the fine-tuning budget.** Brand customization via LoRA adapters on the Firefly foundation model is 10–100× cheaper than retraining. Custom Models is already this pattern — the product insight is that quality of brand lock depends on quality and diversity of training assets, which is a user education problem as much as a model problem.

5. **Adobe's structural moat is being underused.** IP indemnification + CC integration + Custom Models. No competitor has all three. Midjourney leads on aesthetic quality but carries active litigation. The "commercially safe AI image generator" keyword cluster is effectively unclaimed — Firefly is the factual answer to every query in it.

---

## 1. The Quality Measurement Problem

The foundational challenge is not generating good images — it is knowing when you have.

Most evaluation pipelines reach for two metrics and stop:

| Metric | What it measures | Why it's not enough |
|---|---|---|
| **FID** (Fréchet Inception Distance) | Statistical distance between real and generated image distributions | A model generating statistically average images scores well while failing every professional user. Measures population similarity, not individual image value. |
| **IS** (Inception Score) | Object clarity + output diversity | Classifier-dependent; fails to detect poor diversity *within* a class. |
| **CLIP Score** | Semantic alignment between text prompt and image | A "dramatic lighting" prompt scores well while the image feels flat. Measures alignment, not aesthetic quality. |

> ⭐ **The measurement gap:** None of these metrics answer the question a professional actually asks — *"Will I use this, or will I open Photoshop?"* That question requires human signal.

**The only metric that can't be gamed: Human Preference Rate.**

Target: ≥ 75%. Below 60% is a coin flip. Shipping a coin flip to professional users destroys trust faster than not shipping at all.

---

## 2. The Three-Layer Eval Stack

The right architecture balances cost against fidelity. Human eval is the most accurate signal — and the most expensive. The stack is designed so you only pay for human signal at the right moment.

```
LAYER 1 — Automated (run on every build)
  FID · CLIP Score · IS
  Purpose: catch regressions before humans see them
  Cost: near zero
  Signal: directional — tells you if you're moving the right way
         ↓
LAYER 2 — Pixel math (run on every batch)
  Color temperature variance · brightness · contrast · saturation
  Purpose: measure style consistency across a generation batch
  Cost: zero — deterministic, no model inference, sub-second
  Signal: catches the "these images don't belong together" feel
  Proof: Feed Cohesion Score (Stil) — same math, running in production
         ↓
LAYER 3 — Human Preference ELO (run before ship gates)
  Pairwise ranking sessions → ELO scoring → preference rate
  Purpose: final ship/no-ship gate
  Cost: highest — but only paid when the model is a candidate
  Signal: definitive — users vote with judgment, not clicks
```

> ⭐ **The design principle:** Use the cheapest signal that can answer the current question. Automated metrics are for tracking. Pixel math is for consistency. Human preference is for arrival. You add cost only when the cheaper layer genuinely can't reach.

**Video-specific additions:**

| Metric | What it captures | Why it matters |
|---|---|---|
| **Temporal consistency / flicker** | Pixel displacement between frames | Maps directly to "this video feels choppy" — the most actionable video quality complaint |
| **VBench** | 16-dimension suite: motion smoothness, subject consistency, spatial relationships | Run on major versions, not every build — resource-intensive |
| **FVD** | Distribution similarity between real and generated clips | Inherits FID's blind spots — directional, not definitive |

**Multimodal reliability tests (run before any eval goes to humans):**

- **Blank Drop test:** Remove the image input. If model performance doesn't degrade, the model wasn't reading the image — it was pattern-matching on text. This is a visual grounding failure that automated metrics will not catch.
- **Image Sensitivity test:** Swap the image while keeping the prompt identical. If the output doesn't change appropriately, visual grounding is unreliable. LMMS-Eval provides a standardized pipeline for both.

---

## 3. Architecture Decisions

### The quality/cost/latency routing decision

The right framing is not "how do we optimize quality, cost, and latency simultaneously" — that is not possible at Firefly's scale. The right framing is: **which segment gets which model, and why.**

| Segment | Model tier | Rationale |
|---|---|---|
| **Adobe Express (SMB, consumer)** | Distillation model | Lower compute, higher throughput, quality bar is "good enough to post" not "good enough for print" |
| **Creative Cloud Pro (professionals)** | Foundation model | Quality threshold is higher; willingness to pay is higher; latency tolerance is slightly higher |
| **GenStudio / Enterprise** | Foundation model + LoRA adapters | Brand customization required; data residency may require self-hosted open-weight |

**Flow matching:** Reduces inference steps compared to DDPM-based diffusion → lower latency at the same quality level. The product impact: Generative Fill stays under the 2-second professional threshold where users trust the tool and adopt the workflow.

### Fine-tuning strategy — LoRA, not retraining

Full retraining costs millions in GPU compute and months of calendar time. LoRA (Low-Rank Adaptation) freezes the original model weights and adds lightweight trainable adapter layers — trained only, not the foundation.

| Approach | Cost | Time | Quality | Use when |
|---|---|---|---|---|
| Full retrain | $$$$ | Months | Ceiling | Never, for product iteration |
| LoRA fine-tune | $ | Days | Near-ceiling | Brand customization, style adaptation |
| QLoRA | $$ | Days | Near-ceiling | Memory is the bottleneck |
| Prompt engineering | Free | Hours | Bounded | Proprietary API layer only |

> ⭐ **The Firefly product insight:** Custom Models and Style IDs are exactly the LoRA pattern — user-provided reference assets constrain the generation space without retraining. The quality of brand lock depends on quality and diversity of training assets. That's a user education problem as much as a model problem.

### Hybrid architecture recommendation

> "I wouldn't frame this as Firefly vs. open-weight vs. proprietary API. I'd frame it as: which layer of the stack benefits from which approach?
>
> Foundation model → Firefly, because the commercially safe training data is the enterprise moat and no open-weight model replicates it.
> Brand customization → LoRA adapters on Firefly, because full retraining is millions of dollars and months.
> Domain grounding → RAG, because retrieval is cheaper than training and keeps outputs current.
> Prompt layer → few-shot templates, because for proprietary APIs this is the only customization lever."

---

## 4. Evidence: What I've Built

*Three projects mapped to specific Foundations PM requirements.*

### Stil — Feed Cohesion Score

**Maps to:** Style consistency evaluation, batch quality measurement

Built a working image quality measurement system for content creators. The Feed Cohesion Score gives a 0–100 consistency rating across a creator's feed using deterministic pixel math — color temperature, brightness, contrast, and saturation variance. No API calls. No model inference. Sub-second at any scale.

**The design principle that matters for Firefly:** You do not need a learned quality metric to measure style consistency. Pixel math is faster, cheaper, and more interpretable. You add a model only where pixel math genuinely can't reach — which is semantic prompt adherence.

**Direct application to Firefly:** The same math measures consistency *across variations from a single prompt*. Does a batch of Firefly-generated images hold tonal range and stylistic coherence across different seeds? Does a style-reference input produce outputs that actually reflect the reference? This eval layer runs at zero marginal cost.

---

### Vigil + GSentinel — Agentic architecture in production

**Maps to:** Agentic workflow design, RAG-first architecture, multi-agent reliability

Shipped five multi-agent systems. The consistent finding: **FSM orchestration beats LLM orchestration for well-defined creative workflows.** Deterministic state transitions give a full audit trail and predictable behavior — which matters when you're shipping to 30M Creative Cloud users.

In Vigil — a network incident response agent on Splunk's MCP server — RAG-first retrieval eliminated hallucinated queries by routing known incident patterns through retrieval instead of generating answers from scratch. MTTR: 47 minutes → 35 seconds.

**The principle for Firefly:** Retrieve known prompt structures, style seeds, and brand constraints first. Generate only where retrieval fails. Inference cost stays predictable. Quality stays consistent.

GSentinel auto-resolved 67% of security incidents without human intervention using the same FSM-first pattern.

---

### Content Trust Agent — C2PA + SynthID

**Maps to:** Provenance infrastructure, commercially safe generation, content authenticity

Designed a Content Trust Agent for Adobe Stock that fixes the AI content mislabeling problem buyers were experiencing. Root cause: the "Exclude AI" filter relied on contributor self-declaration — mislabeled AI content passed through.

The fix: read C2PA manifest at submission intake and run Google's SynthID detection as a secondary signal. Meta does the same via IPTC Digital Source Type. Both bypass self-declaration entirely.

| | Google | Meta | Adobe (current) | Adobe (should be) |
|---|---|---|---|---|
| Detection method | SynthID watermark + C2PA manifest | C2PA/IPTC at upload | Self-declaration | C2PA manifest reading + SynthID |
| Bypasses mislabeling? | Yes | Yes | No | Yes |

**Why this matters for Foundations:** Adobe co-founded C2PA. Every Firefly output already carries a Content Credential. This infrastructure exists in Firefly, Photoshop, and Express. The Foundations PM role is where the decision to connect it to the rest of the platform gets made.

---

## 5. How I'd Approach This Role — First 90 Days

### Days 1–30: Define the eval contract

"Quality" means different things for an Express SMB and a Photoshop Pro. Before any architecture decision, I'd map the quality threshold per segment and make it the shared contract between research and product teams. This ends the recurring conversation about whether a model is ready to ship — both teams work against the same number.

Deliverable: a one-page quality brief per segment, reviewed and signed off by research, engineering, and surface PM leads.

### Days 31–60: Audit the current eval pipeline

- What is automated and runs on every build?
- What runs only on major versions?
- Where is human preference data being collected — and is it feeding back into training?
- Where is the gap between what is measured and what users experience?

Most teams have a divergence point somewhere in this chain. Finding it is the highest-leverage first move.

### Days 61–90: Ship one LoRA improvement

Identify the workflow with the most post-generation editing time. Fine-tune a LoRA adapter that closes that specific gap. Measure the result: editing time before and after, adoption rate, repeat usage at 7 days.

This establishes the cadence: define quality → measure it → close it → repeat.

---

## 6. Success Metrics

| Metric | Definition | Target | Why it matters |
|---|---|---|---|
| **Human Preference Rate** *(ship gate)* | % of generations preferred over alternative in pairwise ELO | ≥ 75% | The only metric that can't be gamed |
| **Repeat Usage Rate** | % of users who return to the workflow within 7 days | Baseline → +20% in 6 months | The quality signal no benchmark captures |
| **Post-generation editing time** | Time spent in Photoshop after a Firefly generation | −30% | Whether the output was actually usable |
| **Task Completion Rate** | % of users who complete the generative workflow | Baseline → +15% | Measures usability, not just quality |
| **Infrastructure cost per generation** | Total inference cost per completed generation | Track against segment routing model | Determines pricing and margin at scale |
| **AI filter precision** *(Content Trust)* | % of "Exclude AI" results confirmed human photography | ≥ 95% | Whether the provenance layer is trustworthy |

> ⭐ **The north star question:** Are users coming back? Repeat usage at 7 days is the signal that compresses quality, usability, latency, and trust into one number. A user who returns already answered the question "was it good enough?"

---

## 7. Open Questions

1. **Eval pipeline coverage:** What percentage of model updates currently trigger a human preference eval before ship? Is there a defined threshold, or is it judgment-based?

2. **Segment quality thresholds:** Is there a documented quality bar per surface (Express vs. Photoshop vs. GenStudio)? If not, who currently makes the ship/no-ship call?

3. **LoRA in production:** How many active LoRA adapters are running in production today across Custom Models? What does the quality variance look like across adapters trained on different asset sets?

4. **Temporal consistency for video:** Is flicker/temporal consistency currently measured as a regression metric on every video model build, or only at major version checkpoints?

5. **C2PA integration scope:** Firefly outputs carry Content Credentials. Is there a roadmap to use that provenance data within the Adobe platform — Stock intake, Creative Cloud libraries, GenStudio workflows?

---

*Bharat Namatherdhala · May 2026*
