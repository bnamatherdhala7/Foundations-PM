# Foundations PM — 15-Minute Interview Script

**Role:** Principal PM, Adobe Research & AI — Firefly Image/Video Generation Quality
**Time budget:** 15 minutes · **Hard rule:** Never cut Block 3. Trim Block 4 if needed.

---

## The Story Arc

> **Who I am → What I've proven → Why I fit → What I'd do first**

Every block connects a skill I've shipped to a gap Firefly has today. This is not a generic PM pitch. It is a direct answer to the JD.

---

## Block 1 — Opening (0:00–2:00)

> "I'm a PM who builds and measures AI systems in production — not in docs, in code.
>
> I shipped a working image quality measurement system. I shipped five multi-agent pipelines. I built a competitive intelligence system that runs at $0.003 per report. And the through-line in all of it is the same thing this role is about: you can't improve what you can't measure, and most teams stop measuring too early.
>
> The Foundations PM job is the eval and architecture layer that every Adobe creative surface depends on. That's exactly where I want to work — because I've been building that layer from scratch, and I want to build it at the scale where it actually changes the product."

**Then signal the structure in one sentence:**
> "Three things: where Firefly stands and why, the systems I've built that map to those gaps, and three moves I'd make in 90 days."

---

## Block 2 — Where Firefly Stands + The Measurement Problem (2:00–6:00)

*Lead with real numbers. This is where you show you did the research.*

### The competitive position

> "The Artificial Analysis arena shows Firefly at ELO 971 — that's the Image 3 baseline the market sees. We're on Image 5 now with native 4MP, better text rendering, and layered editing, but arena scoring lags. GPT Image 2 is at 1339. Midjourney is at 1093. On image editing — the leaderboard that matters most for AI Studio — Firefly isn't in the top 5.
>
> That 368-point gap to GPT Image 2 is not one problem. It is three:
>
> **Text rendering** — below 45% on Image 3, improved in Image 5 but not at Ideogram parity. LoRA fix, not a foundation model problem. 90 days.
>
> **Image editing quality** — not in top 5. This is the highest urgency because AI Studio is the Firefly differentiator. 90 days with fine-tuning and a proper eval pipeline.
>
> **Aesthetic ceiling** — 122 points behind Midjourney. Training data curation and human preference feedback. 18 months."

### Why the measurement is part of the problem

> "If those gaps persist, part of the reason is the eval stack. FID measures distribution similarity — a model generating statistically average images scores well while failing every professional. CLIP Score measures semantic alignment — 'dramatic lighting' scores well while the image feels flat. Neither answers whether a professional will use the output.
>
> The ship gate should be human preference rate ≥ 75%. Below 60% is a coin flip.
>
> The fix: three eval layers. Automated metrics every build for regression. Pixel math for style consistency across batches — no cost, sub-second. Human preference ELO as the final gate. And that human signal has to feed back into training within two weeks — not sit in a spreadsheet."

---

## Block 3 — Skills → Role (6:00–11:00) ← **Never cut this block**

*Each project answers a specific JD requirement. Lead with what I built, then how it applies.*

### Skill 1: I built the eval stack you need — Stil Feed Cohesion Score

> "I built a 0–100 image quality consistency system using deterministic pixel math: color temperature, brightness, contrast, saturation variance. No model. No API cost. Sub-second.
>
> **The principle:** pixel math is faster, cheaper, and more interpretable than a learned metric. You add a model only where it can't reach.
>
> **The Firefly application:** this is Layer 2 of the eval stack I'd build here. Does a batch of Firefly outputs from the same prompt hold tonal range across seeds? Does a Custom Model produce brand-consistent outputs across 50 generations? Same math. That's the eval layer that catches failures before they reach expensive human eval."

### Skill 2: I build agentic systems that work in production — Vigil + GSentinel

> "Five multi-agent systems shipped. The consistent finding: FSM orchestration beats LLM orchestration for well-defined workflows. Deterministic state — full audit trail.
>
> In Vigil, RAG-first retrieval cut hallucinated queries by routing known patterns through retrieval before generation. MTTR: 47 minutes → 35 seconds. GSentinel auto-resolved 67% of incidents.
>
> **The principle:** retrieval and heuristics carry the load until they genuinely can't. Then you pay for inference. Keep orchestration costs predictable at 30-million-user scale.
>
> **The Firefly application:** retrieve known prompt structures, brand constraints, style seeds first. Generate only where retrieval fails."

### Skill 3: I understand the full platform picture — Content Trust Agent

> "I designed a provenance detection pipeline for Adobe Stock: C2PA manifest reading at intake + Google SynthID detection — bypassing contributor self-declaration entirely. Meta does the same with IPTC Digital Source Type.
>
> **The Foundations insight:** Adobe co-founded C2PA. Every Firefly output already carries a Content Credential. That infrastructure exists but isn't connected across the platform. Connecting it is an engineering project, not a research project.
>
> **Why it matters for this role:** Firefly's commercially safe training claim is only as credible as its provenance infrastructure at scale. That's a competitive moat GPT Image and Midjourney can't match."

---

## Block 4 — First 90 Days (11:00–13:30)

> "Three concrete moves.
>
> **First: define the eval contract per segment.** Express bar is 'good enough to post.' CC Pro bar is 'survives professional art direction.' GenStudio bar is 'brand-compliant without revision.' Three thresholds. Research optimizes to the wrong target until these exist on paper.
>
> **Second: ship the text rendering LoRA.** Text accuracy is a named gap with a known fix — LoRA adapter on typography failure pairs. Not a foundation model project. 90 days is achievable.
>
> **Third: close the feedback loop.** Human preference sessions are currently a gate, not a loop. Every session should produce a training brief to research within two weeks. That's how you systematically close 368 ELO points instead of hoping the next training run helps.
>
> The one metric I'd track above all others: **repeat usage rate at 7 days**. Not ELO. Not generation count. Repeat usage means the output was good enough to trust again."

---

## Block 5 — Close (13:30–15:00)

> "That's the arc — competitive position, three-layer eval stack, three 90-day moves. One question: where does the team feel most stuck today — is it the eval pipeline not giving clear signal, the model quality ceiling, or cross-team alignment on what 'good' means?"

**If time:**
> "The image editing arena gap is what I'd prioritize first. AI Studio is the differentiator — is that a greenfield area or is there existing work underway?"

---

## Cheat Sheet — Numbers to Deliver Without Hesitation

| Topic | Say this |
|---|---|
| **Firefly ELO** | 971 in arena (Image 3 baseline; Image 5 current, released Oct 2025) |
| **GPT Image 2 ELO** | 1339 — 368-point gap |
| **Midjourney ELO** | 1093 — 122-point gap |
| **Image editing** | Firefly not in top 5; GPT Image 1.5 leads at 1264 |
| **Text rendering** | <45% on Image 3; improved in Image 5; not at Ideogram parity |
| **FLUX.1-dev** | 12.9k HuggingFace likes, 716k downloads — open-weight community benchmark |
| **Ship gate** | Human preference rate ≥ 75%; below 60% = coin flip |
| **Latency threshold** | Sub-2-second Generative Fill — above this, professionals abandon |
| **Flow matching** | Fewer inference steps → lower latency at same quality |
| **LoRA vs. retrain** | 10–100× cheaper; Custom Models is already this pattern |
| **FID limit** | Average images score well; misses artistic quality |
| **CLIP Score limit** | Semantic alignment only; misses aesthetic nuance |
| **Blank Drop test** | Remove image input — if output doesn't degrade, model wasn't reading it |
| **Image Sensitivity test** | Swap image, keep prompt — output must change |
| **Vigil MTTR** | 47 min → 35 sec |
| **GSentinel auto-fix rate** | 67% |
| **Feed Cohesion Score** | Color temp · brightness · contrast · saturation variance |
| **Firefly moat** | IP indemnification + CC integration + Custom Models + commercially safe training |
| **C2PA** | Adobe co-founded it; every Firefly output carries a Content Credential |
| **SynthID** | Google DeepMind — 10B+ pieces watermarked; survives cropping and compression |
| **CI pipeline cost** | $0.003 per competitive report |
| **Multi-model platform** | Adobe integrated FLUX 1.1, Imagen 4, Ideogram 3.0, Runway Gen-4 at MAX 2025 |

---

## Push-Back Scripts

**"We don't use FID/CLIP Score internally"**
> "Helpful to know — what's the primary signal today? I'm curious whether it's human preference-weighted, automated, and whether it's segment-specific or a single bar."

**"Our ELO is different from what you cited"**
> "I'd defer to internal data. I use the arena because it's what the market sees — enterprise buyers and designers compare on that. What does internal eval show?"

**"How would you improve video quality?"**
> "Temporal consistency as a regression metric on every build — that's the dimension that maps to 'this video feels choppy.' VBench has 16 dimensions but temporal flicker is the one that drives abandonment. Instrument that first, then run human preference ELO on outputs that pass the consistency floor."

**"Tell me more about the competitive picture"**
> "Midjourney — active litigation, enterprise can't use it. GPT Image 2 — leads on quality, locked inside ChatGPT, no CC integration. FLUX — open-weight, developer ecosystem, no commercial safety guarantee. And Adobe isn't just Firefly anymore — at MAX 2025, Adobe integrated FLUX 1.1, Imagen 4, Ideogram 3.0, Runway Gen-4. The Foundations PM job is building the eval layer that governs a fleet of models. Firefly stays the default for commercially safe generation. The moat is the combination: IP indemnification, CC integration, Custom Models, commercially safe training. No competitor has all four."

**"Open-weight vs. proprietary?"**
> "Layer-specific. Foundation model stays Firefly — the training data advantage can't be replicated. Brand customization goes to LoRA on Firefly — retraining is millions of dollars. Domain grounding goes to RAG. Open-weight models like FLUX matter as benchmarks and as the LoRA research ecosystem. Read their work, don't build an alternative foundation."

---

*Bharat Namatherdhala · Foundations PM Interview Prep · May 2026*
