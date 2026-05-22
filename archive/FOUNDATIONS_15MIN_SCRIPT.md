# Foundations PM — 15-Minute Interview Script

**Role:** Principal PM, Adobe Research & AI — Firefly Image/Video Generation Quality
**Time budget:** 15 minutes
**Goal:** Show technical depth in AI eval + product sense + proof of shipping

*Each block has a hard time cap. If you run long on Block 2, cut Block 4 — never cut Block 3.*

---

## Block 1 — Opening (0:00–2:00)

*One sentence who. One sentence what. One sentence why this role specifically.*

> "I'm a product manager who builds AI systems and measures them — not at the whiteboard level, but in production. I've shipped a working image quality measurement system, five multi-agent pipelines, and a competitive intelligence system that runs at $0.003 per report. I'm here because the Foundations PM role is exactly where I want to apply that — building the eval and architecture layer that every Adobe creative surface depends on."

**Transition — signal the structure so they know where you're going:**

> "I want to cover three things in the next 15 minutes: how I think about GenAI quality evaluation, the systems I've built that map directly to Firefly's challenges, and how I'd approach this role in the first 90 days."

---

## Block 2 — Technical Depth: Eval + Architecture (2:00–6:00)

*4 minutes. Three-layer eval stack → limits of each metric → quality/cost/latency triangle. This is the section that separates candidates — most stop at naming metrics, you name the limits.*

### The eval honesty problem

> "The fundamental challenge with Firefly quality isn't measuring it — it's measuring it honestly. Most teams reach for FID and CLIP Score and stop there. Both of those are wrong in specific ways you need to name.

> FID measures statistical distance between real and generated distributions. A model that generates statistically average images — competent but creatively inert — scores well on FID while failing every professional who uses it. CLIP Score measures semantic alignment between prompt and output. A 'dramatic lighting' prompt scores well while the image feels flat. Neither metric tells you whether a user will actually like what they got.

> The only metric that can't be gamed is human preference. I'd target ≥ 75% human preference rate as the ship gate — below 60% is a coin flip, and shipping a coin flip to professionals destroys trust faster than not shipping at all."

### Three-layer eval stack

> "But human eval is expensive at scale. So the architecture I'd build is three layers.

> First — automated: FID and CLIP Score for tracking direction over time and catching regressions before humans see them. Fast, cheap, directional.

> Second — pixel math: deterministic consistency scoring across a batch of generations. Color temperature variance, brightness, contrast, saturation. No model, no inference cost, sub-second at any scale. This catches the thing users feel as 'these images don't belong together' — style drift across a generation batch.

> Third — human preference ELO: pairwise ranking sessions after model updates. Final gate before shipping. Automated scores tell you direction. Human signal tells you arrival."

### Quality / cost / latency triangle

> "For Firefly specifically: you can't optimize quality, latency, and cost simultaneously for every segment.

> My recommendation: route Express users to a distillation model — lower compute, higher throughput, good enough quality for the use case. Reserve the foundation model for Creative Cloud Pro, where the quality threshold and willingness to pay are both higher. Flow matching gets you lower latency at the same quality level by reducing inference steps — that's the architectural move that brings Generative Fill under the 2-second professional threshold users won't wait past."

---

## Block 3 — Projects as Proof Points (6:00–11:00)

*5 minutes total. Three projects, 90 seconds each. Setup → what I built → the principle it proves for Firefly.*

### Project 1 — Stil: Feed Cohesion Score (6:00–7:30)

> "I built an image quality measurement system called the Feed Cohesion Score. It gives content creators a 0–100 consistency score across their social feed — color temperature, brightness, contrast, and saturation variance — using deterministic pixel math. No API calls. No model inference. Sub-second at any scale.

> The design principle: you don't need a learned quality model to measure consistency. Pixel math is faster, cheaper, and more interpretable. You add a model only where pixel math genuinely can't reach — which for Firefly is semantic prompt adherence.

> This maps directly to one of Firefly's open challenges: style consistency across variations from a single prompt. Does a batch of generations hold color temperature and tonal range across different seeds? Does a style-reference input actually produce outputs that reflect the reference? The same pixel math stack answers those questions — and it costs nothing to run."

### Project 2 — Vigil + GSentinel: Agentic architecture (7:30–9:00)

> "I've shipped five multi-agent systems. The consistent finding: FSM orchestration beats LLM orchestration for well-defined creative workflows. Deterministic state transitions give you a full audit trail and predictable behavior — which matters when you're shipping AI capabilities to 30 million Creative Cloud users.

> In Vigil — a network incident response agent running on Splunk's MCP server — I implemented RAG-first retrieval that eliminated hallucinated queries by routing known incident patterns through retrieval instead of generating from scratch. The principle: heuristics and retrieval carry the load until they genuinely can't. Then you pay for inference.

> For Firefly's workflow orchestration: same pattern. Retrieve known prompt structures, style seeds, and brand constraints first. Generate only where retrieval fails. Inference cost stays predictable. Quality stays consistent."

### Project 3 — Content Trust Agent: C2PA + SynthID (9:00–10:30)

> "Most recently, I designed a Content Trust Agent for Adobe Stock — relevant here because it's live product thinking, not hypothetical.

> The problem: buyers were experiencing AI content getting through the 'Exclude AI' filter because the filter relied on contributor self-declaration. The fix: read C2PA manifest at submission intake and run Google's SynthID detection as a secondary signal — bypassing self-declaration entirely. Meta does the same thing with IPTC Digital Source Type.

> The Adobe insight that matters for Foundations: Adobe co-founded C2PA. Every Firefly output already carries a Content Credential. The provenance infrastructure exists in Firefly, Photoshop, and Express — it's not connected to the Stock intake pipeline. That's an engineering project, not a research project.

> The Foundations PM relevance: Firefly's commercially safe training claim is only as credible as its provenance infrastructure at scale. As Firefly grows, C2PA becomes a strategic asset, not a compliance checkbox."

---

## Block 4 — How I'd Approach the Role: First 90 Days (11:00–13:30)

*2.5 minutes. Three concrete moves — not "I'd learn the product," but actual decisions you'd make.*

> "Three things in the first 90 days.

> **First — define the eval contract.** 'Quality' means different things for an Express SMB user and a Photoshop Pro user. I'd map the quality threshold per segment and make it the shared contract between research and product. This ends the recurring conversation about whether a model is ready to ship — both teams are working against the same number.

> **Second — audit the current eval stack.** What's automated and runs on every build? What runs only on major versions? Where is human preference data being collected — and is it being fed back into model training? Most teams have a gap between what they measure and what users experience. Finding that gap is the highest-leverage first move.

> **Third — identify the LoRA opportunity.** Which workflows have the most post-generation editing time? That's where a fine-tuned adapter closes the quality gap without retraining the foundation model. The product win is measurable: editing time drops, adoption increases, infrastructure cost is a fraction of full fine-tune.

> The metric I'd track above all others: **repeat usage rate within 7 days.** Not generation count. Not quality score. Repeat usage is the signal no benchmark captures — it means the output was good enough to trust again."

---

## Block 5 — Close + Questions (13:30–15:00)

*Signal you're done, then ask a question that shows you've thought about the actual problems.*

> "That's my framework — eval stack, architecture decisions, and first 90 days. Before I ask you a question: is there a specific capability or quality challenge you'd want me to go deeper on?"

**Question 1 — ask this:**
> "What does the current eval pipeline look like, and where is it most often wrong about whether something is ready to ship?"

**Question 2 — if time:**
> "Where is the quality/cost/latency tension most acute right now — is it Express throughput, Creative Cloud Pro ceiling, or something specific to video?"

---

## Cheat Sheet — Numbers to Deliver Without Hesitation

| If asked about... | Say... |
|---|---|
| Human preference target | ≥ 75% — below 60% is a coin flip |
| Professional latency threshold | Sub-2-second for Generative Fill — users abandon above this |
| Flow matching benefit | Fewer inference steps → lower latency at same quality |
| LoRA vs. full fine-tune cost | 10–100× cheaper depending on model size |
| Blank Drop test | Remove image input — if output doesn't degrade, model wasn't reading it |
| Image Sensitivity test | Swap image keeping prompt same — answer must change appropriately |
| FID limit | Measures distribution, not artistic quality — statistically average images score well |
| CLIP Score limit | Measures semantic alignment, not aesthetic nuance |
| Vigil MTTR improvement | 47 min → 35 sec |
| GSentinel auto-fix rate | 67% |
| Feed Cohesion Score inputs | Color temp · brightness · contrast · saturation variance |
| Distillation model routing | Express → distillation; CC Pro → foundation model |
| Firefly enterprise moat | IP indemnification + CC integration + Custom Models — no competitor has all three |
| C2PA | Adobe co-founded it — every Firefly output already carries a Content Credential |
| SynthID | Google DeepMind — 10B+ pieces of content watermarked; survives cropping and compression |
| Canva MAU | 260M |
| CI pipeline cost | $0.003 per Firefly competitive report |

---

## If They Push Back or Go Off-Script

**"We don't use FID/CLIP Score internally"**
> "That's useful context — what's the primary signal your team uses today? I'd be curious whether it's more human preference-weighted or automated."

**"Tell me more about your agentic systems"**
> "The five systems I've built follow the same architecture pattern — FSM for orchestration, RAG-first for retrieval, inference as the last resort. The most directly relevant to Firefly is the Vigil RAG-first pattern because it solves the same problem: keeping output quality predictable when the input space is open-ended."

**"How would you improve Firefly's video quality specifically?"**
> "I'd start with temporal consistency — that's the metric that maps most directly to 'this video feels choppy,' which is the user complaint you can act on. VBench gives you 16 dimensions, but temporal flicker is the one that drives abandonment. I'd instrument that as a regression metric on every build, then use human preference ELO on the outputs that score ≥ 80 on temporal consistency to check whether the overall feel lands."

**"What do you know about our competitors?"**
> "Midjourney leads on aesthetic quality and community, but has no IP indemnification — still facing active litigation. Stable Diffusion is the open-weight benchmark. Adobe's structural advantage is the combination: commercially safe training + CC workflow integration + Custom Models. No competitor has all three. The Getty/Shutterstock merger makes that advantage more important, not less — their combined library is the traditional alternative buyers turn to when they don't trust AI-generated content."

---

*Bharat Namatherdhala · Foundations PM Interview Prep · May 2026*
