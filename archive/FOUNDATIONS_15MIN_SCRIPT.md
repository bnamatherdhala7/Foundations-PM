# Foundations PM — 15-Minute Interview Script

**Role:** Principal PM, Adobe Research & AI — Firefly Image/Video Generation Quality
**Time budget:** 15 minutes · **Hard rule:** Never cut Block 3. Trim Block 4 if needed.

---

## The Story Arc

> **I already launched Firefly → I built the agentic layer on top of it → I've been measuring quality at Adobe's scale → Here's what I'd do as Foundations PM**

This is not a PM studying for an interview. This is the PM who launched the product — now ready to own the foundation it runs on.

---

## Block 1 — Opening (0:00–2:00)

> "I led the 0→1 Firefly and Adobe Stock Generative AI platform launch — $60M ARR across 50+ countries, partnering with AI research, applied science, engineering, and GTM to build and ship the creative workflow from the ground up.
>
> Since then I've gone deeper on the layer underneath that launch — defining the agentic harness architecture for Adobe's MCP orchestration platform, building the experimentation infrastructure that tests and measures AI features across Firefly and Document Cloud, and shipping AI quality measurement systems from scratch in my own work.
>
> The Foundations PM role is the right next move because it's the architectural layer that determines what every surface can and can't ship. I've operated above it, I've built pieces of it, and I want to own it."

**Signal the structure:**
> "Three things: what I know from inside Adobe about where the gaps are, where Firefly stands competitively and why, and three moves I'd make in 90 days."

---

## Block 2 — Inside View + Competitive Position (2:00–6:00)

### What I know from the inside

> "From running the Firefly launch I know three things that aren't visible from outside:
>
> First — **the eval gap is organizational, not technical.** The measurement infrastructure exists. The problem is that human preference signal from model eval sessions isn't feeding back into training as a closed loop. It's a gate, not a compound system.
>
> Second — **the experimentation platform I built measures AI feature performance.** I built the A/B, Bayesian, and multi-armed bandit infrastructure that runs across Firefly and Document Cloud. I know how we measure — and where the current signal stops short of telling us whether a professional will actually use the output.
>
> Third — **Adobe's MCP agentic harness is live.** I defined the architecture — multi-agent pipeline ingesting real-time signals with memory, tool orchestration, and telemetry/evals layers. The pattern that works for orchestration at Adobe scale is FSM with RAG-first retrieval. That same pattern applies directly to how Firefly should route prompt → retrieval → generation."

### The competitive position — real numbers

> "On the Artificial Analysis arena, Firefly's ELO is 971 — that's the Image 3 baseline the market sees. We're on Image 5 now with native 4MP and improved text rendering, but arena scoring lags. GPT Image 2 is at 1339 — a 368-point gap. Firefly isn't in the top 5 on image editing, which is the leaderboard that matters most for AI Studio.
>
> That gap is three separate problems:
> **Text rendering** — below 45% on Image 3, improved in Image 5 but not at Ideogram parity. LoRA fix, 90 days.
> **Image editing quality** — not in top 5. Highest urgency because AI Studio is the differentiator. Fine-tune + eval pipeline, 90 days.
> **Aesthetic ceiling** — 122 points behind Midjourney. Training data curation, 18 months."

### Why the measurement is part of the problem

> "FID and CLIP Score are the defaults. FID misses artistic quality — statistically average images score well. CLIP misses aesthetic nuance — dramatic lighting scores well while the image feels flat. Neither tells you whether a professional will use the output.
>
> The ship gate should be human preference rate ≥ 75%. Below 60% is a coin flip. You can't close 368 ELO points if the eval pipeline can't detect the delta."

---

## Block 3 — Skills → Role (6:00–11:00) ← **Never cut this block**

*Each proof point answers a specific JD requirement. 60 seconds each.*

### Adobe: I already built the launch and the platform layer

> "**$60M ARR on the Firefly 0→1 launch** — that means I've set up the GTM motion, aligned research and engineering, and defined what 'ready to ship' means for Firefly at scale. I know the internal dynamics of how quality decisions get made and where they stall.
>
> **MCP agentic harness architecture** — multi-agent pipeline with memory, tool orchestration, and telemetry/evals. The orchestration principle I validated there: deterministic state transitions beat LLM orchestration for well-defined workflows. Full audit trail, predictable behavior at 30-million-user scale.
>
> **Experimentation platform** — A/B, Bayesian, Multi-Armed Bandits across Firefly and Document Cloud. 40% increase in testing velocity. I know what it takes to run quality measurement at Adobe's scale — and where the current setup doesn't reach the human preference signal that closes the loop."

### Outside Adobe: I built the eval stack you need — Stil

> "Stil is a conversational AI editing assistant. The core piece is the Feed Cohesion Score — 0–100 visual consistency using deterministic pixel math: color temperature, brightness, contrast, saturation variance. No model. No API cost. Sub-second at any scale.
>
> The principle: pixel math is faster, cheaper, more interpretable than a learned metric. You add a model only where it can't reach.
>
> **Firefly application:** Layer 2 of the three-layer eval stack. Does a batch of Firefly outputs from the same prompt hold tonal range across seeds? Does a Custom Model produce brand-consistent outputs across 50 generations? Same math. Catches failures before they reach expensive human eval."

### I've built for all three Firefly segments

> "**Creative professionals** — Stil. Problem is visual drift, not tool quality. 200 pieces of content a year, six months later the feed doesn't look like them anymore.
>
> **Content creators** — StoryForge. 4-agent system: ideation → script → visual gen → publish. Agent orchestration harness with deterministic tool routing and multimodal LLM-as-judge evals across narrative, visual, and platform-fit dimensions. Built on Vertex AI. North star: Posted Video Rate ≥ 60% — not generation count, whether they actually posted it.
>
> **Business users** — ML propensity models at Adobe, RL-based pricing engine ($15M revenue lift), checkout AI recommendation engine ($10M ARR). Business users pay for AI that reduces decisions, not AI that generates more options."

### I understand the full platform picture — Content Trust + C2PA

> "I designed a provenance detection pipeline for Adobe Stock: C2PA manifest reading at intake + Google SynthID detection — bypassing contributor self-declaration. Adobe co-founded C2PA. Every Firefly output already carries a Content Credential. That infrastructure exists but isn't connected platform-wide. That's an engineering project, not a research project.
>
> Firefly's commercially safe training claim is only as credible as its provenance infrastructure at scale. That's a moat GPT Image and Midjourney can't match."

---

## Block 4 — First 90 Days (11:00–13:30)

> "Three moves, anchored to specific gaps.
>
> **First: define the eval contract per segment.** Express bar is 'good enough to post.' CC Pro bar is 'survives professional art direction.' GenStudio bar is 'brand-compliant without revision.' I've run experimentation across these surfaces — I know what a segment-specific quality threshold looks like in practice. Right now research is likely optimizing against a single bar and missing half the segments.
>
> **Second: ship the text rendering LoRA in 90 days.** This is a named, measurable gap with a known fix — LoRA adapter on typography failure pairs, not a foundation model project. I've shipped LoRA-pattern products already (Custom Models is this pattern). 90 days is achievable.
>
> **Third: close the feedback loop.** Every human preference eval session should produce a training brief to research within two weeks: 'Segment A preference rate dropped 3 points on lighting coherence — here are the 47 failing pairs.' The experimentation platform I built at Adobe already produces this kind of signal. Connecting it to training is the missing link.
>
> The one metric I'd track above all others: **repeat usage rate at 7 days.** Not ELO. Not generation count. Repeat usage means the output was good enough to trust again."

---

## Block 5 — Close (13:30–15:00)

> "That's the arc — what I know from inside Adobe, where the competitive gaps are, and three concrete moves. One question: where does the team feel most stuck — eval pipeline not giving clear signal, model quality ceiling, or cross-team alignment on what 'good' means per segment?"

**If time:**
> "The image editing arena gap is what I'd prioritize first given AI Studio is the differentiator. Is that a greenfield area or is there work underway?"

---

---

## Phase 2 — Walkthrough Key Points (share screen on FOUNDATIONS_PM_WALKTHROUGH.md)

*After your verbal pitch, pull up the walkthrough doc. These are the sections to linger on — 30 seconds each. Don't read. Point and anchor.*

| Section | What to say |
|---|---|
| **TL;DR** | "Five bullets, five minutes. This is the frame I'd use for every quality decision on the team." |
| **Customer Segments** | "Three quality bars — Express, CC Pro, GenStudio. The most important point: quality is not one number. Research optimizing to a single bar is why the ELO gap persists." |
| **Quality Measurement Problem** | "FID and CLIP Score with their limits named. I built the experimentation platform at Adobe — I know where these metrics stop short." |
| **Three-Layer Eval Stack** | "The architecture I'd build. Layer 2 is pixel math — I shipped this in Stil. Layer 3 feeds back to training. That loop is what's missing today." |
| **Competitive Landscape** | "Real ELO numbers. Image 5 context. The three-gap decomposition — different owners, different timelines, different fixes." |
| **Capability Roadmap** | "Tier 1 is infrastructure. Tier 2 is 90-day LoRA. Tier 3 is the 18-month foundation model work. Each line is anchored to a named benchmark deficit." |
| **Data Strategy** | "Six strategies for improving quality without customer data. Lead with Adobe Fonts → text rendering LoRA. This is the question that separates a PM who understands the constraint from one who doesn't." |
| **Evidence: What I've Built** | "Four proof points — each one maps to a JD requirement. Stil = eval stack. Vigil/GSentinel = agentic architecture. Content Trust = platform strategy. CI pipeline = competitive insight." |

**Transition line after walkthrough:**
> "This is the product thinking document I'd bring to a research alignment meeting. Not slides — a working document with named metrics, ranked priorities, and open questions I'd want answered on Day 1."

---

## Cheat Sheet — Numbers Cold

| Topic | Say this |
|---|---|
| **Your Adobe headline** | Led 0→1 Firefly + Adobe Stock GenAI launch — $60M ARR, 50+ countries |
| **MCP harness** | Defined Adobe's MCP orchestration architecture — multi-agent with memory, tool orchestration, telemetry/evals |
| **Experimentation** | Built A/B, Bayesian, MAB platform across Firefly + Document Cloud — 40% testing velocity increase |
| **Pricing engine** | RL-based dynamic pricing — 15% upgrade conversion lift, +$15M revenue |
| **Checkout AI** | Collaborative filtering + cross-sell — $10M ARR |
| **Firefly ELO** | 971 in arena (Image 3 baseline; Image 5 current, released Oct 2025) |
| **GPT Image 2 ELO** | 1339 — 368-point gap |
| **Midjourney ELO** | 1093 — 122-point gap |
| **Image editing** | Firefly not in top 5; GPT Image 2 leads at ELO 1253, Gemini 3.1 Flash at 1236 |
| **Text rendering** | <45% on Image 3; improved in Image 5; not at Ideogram parity |
| **Ship gate** | Human preference rate ≥ 75%; below 60% = coin flip |
| **Latency threshold** | Sub-2-second Generative Fill — above this, professionals abandon |
| **LoRA vs. retrain** | 10–100× cheaper; Custom Models is already this pattern |
| **FID limit** | Average images score well; misses artistic quality |
| **CLIP Score limit** | Semantic alignment only; misses aesthetic nuance |
| **Blank Drop test** | Remove image input — if output doesn't degrade, model wasn't reading it |
| **Vigil MTTR** | 47 min → 35 sec |
| **GSentinel auto-fix** | 67% |
| **StoryForge north star** | Posted Video Rate ≥ 60%; TTFV < 5 min at p95 |
| **Feed Cohesion Score** | Color temp · brightness · contrast · saturation variance |
| **Firefly moat** | IP indemnification + CC integration + Custom Models + commercially safe training |
| **C2PA** | Adobe co-founded it; every Firefly output carries a Content Credential |
| **SynthID** | Google DeepMind — 10B+ watermarked; survives cropping and compression |
| **CI pipeline cost** | $0.003 per Firefly competitive report |
| **Multi-model platform** | Adobe integrated FLUX 1.1, Imagen 4, Ideogram 3.0, Runway Gen-4 at MAX 2025 |
| **US Patent** | US11556836B1 — ML recommendation engine (Intuit) |
| **Data strategy — 3 buckets** | Bucket A: Adobe-owned. Bucket B: Open source + open-weight. Bucket C: Synthetic + behavioral. Run all three in parallel. |
| **Bucket A #1** | Adobe Fonts → text rendering LoRA. Complete font corpus, legally clean, no competitor has it. 60-day pipeline. |
| **Bucket A #2** | Stock contributor Tier 1 consent program with revenue share. Flywheel: better model → more AI content sold → more contributor revenue → more consent. |
| **Bucket B #1** | Pick-a-Pic v2 (851k pairwise prefs) + HPD v2 (798k prefs) → DPO fine-tune. Free datasets, published for this purpose. |
| **Bucket B #2** | MagicBrush (10k human-annotated editing pairs) + InstructPix2Pix (310k synthetic) → editing LoRA. Directly closes the not-in-top-5 editing gap. |
| **Bucket B #3** | DALL-E 3 recaptioning methodology: recaption Stock images with detailed descriptions using a VLM. OpenAI published this openly — it's the highest-impact prompt adherence lift technique known. |
| **Bucket B #4** | FLUX.1-schnell (Apache 2.0) distillation → Express-tier model. Best open-weight for fast inference; legal for commercial distillation. |
| **Bucket C #1** | Counterfactual degradation: degrade owned Stock images, create preference pairs. Automated, near-zero cost. |
| **Bucket C #2** | Behavioral signal (opt-in, content-free events): dwell + re-run + export rate. `{event: "re_run", latency_ms: 8400}` — no content, no PII. |
| **Data constraint framing** | "Three buckets. Adobe-owned data is the moat. Open source is free signal most teams don't fully exploit. Synthetic and behavioral compound over time." |

---

## Push-Back Scripts

**"We don't use FID/CLIP Score internally"**
> "Helpful — what's the primary signal today? I want to understand whether it's human preference-weighted, automated, and whether it's segment-specific or a single bar. That shapes how the feedback loop gets designed."

**"Our ELO is different from what you cited"**
> "I'd defer to internal data. I use the arena because it's what the market sees — enterprise buyers and designers compare on that signal. What does internal eval show relative to the arena?"

**"How would you improve video quality?"**
> "Temporal consistency as a regression metric on every build first — that maps directly to 'this video feels choppy.' VBench has 16 dimensions but temporal flicker drives abandonment. Instrument that, then run human preference ELO on outputs that pass the consistency floor."

**"Tell me more about the competitive picture"**
> "Midjourney — active litigation, enterprise can't use it. GPT Image 2 — leads on quality, locked inside ChatGPT, no CC integration. FLUX — open-weight, developer ecosystem, no commercial safety. And Adobe isn't just Firefly anymore — at MAX 2025, Adobe integrated FLUX 1.1, Imagen 4, Ideogram 3.0, Runway Gen-4. The Foundations PM job is building the eval layer that governs a fleet of models. Firefly stays the default for commercially safe generation. No competitor has IP indemnification + CC integration + Custom Models + commercially safe training."

**"Open-weight vs. proprietary?"**
> "Layer-specific. Foundation model stays Firefly — the training data advantage can't be replicated. Brand customization → LoRA on Firefly; retraining is millions of dollars. Domain grounding → RAG. Open-weight models like FLUX matter as benchmarks and as the LoRA research ecosystem — read their work, don't build an alternative foundation."

**"How do you improve Firefly quality if you can't train on customer data?"**
> "Three buckets, all running in parallel. Bucket A is Adobe-owned: Adobe Fonts is the most underexploited asset — a text rendering LoRA on the full font corpus closes the <45% accuracy gap in 60 days and no competitor has that dataset. Stock contributor consent program with revenue share builds the quality ceiling long-term. Bucket B is open source: Pick-a-Pic v2 and HPD v2 together are 1.6 million human preference pairs — free, published specifically for DPO fine-tuning image models. MagicBrush and InstructPix2Pix close the editing gap. DALL-E 3 published a recaptioning methodology — recaptioning Stock images with a VLM is the highest-impact prompt adherence lift OpenAI identified, and it's open. FLUX.1-schnell is Apache 2.0 — distill it into the Express-tier model. Bucket C is synthetic and behavioral: counterfactual degradation on owned Stock images generates hard negative pairs at near-zero cost. A privacy-safe events pipeline — dwell time, re-run rate, export rate — gives a reward signal without storing content. The framing: Adobe-owned data is the moat. Open source is free signal most teams don't exploit. Synthetic and behavioral compound over time."

**"You've been in PLG and monetization — why Foundations?"**
> "The 0→1 Firefly launch and the MCP harness architecture are the reasons. I've seen from the launch side what happens when the foundation layer doesn't give clear quality signal — you ship uncertainty. From the agentic architecture side I've seen what it takes to build orchestration that scales. Foundations PM is where both of those threads converge. The leverage is higher here than on any surface — every team that builds on top inherits what this role builds."

---

*Bharat Namatherdhala · Foundations PM Interview Prep · May 2026*
