# Foundations PM — 15-Minute Interview Script

**Role:** Principal PM, Adobe Research & AI — Firefly Image/Video Generation Quality
**Time budget:** 15 minutes
**Goal:** Show technical depth in AI eval + competitive knowledge + proof of shipping

*Hard rule: If you run long on Block 2, trim Block 4. Never cut Block 3 — it's the proof.*

---

## Block 1 — Opening (0:00–2:00)

*One sentence who. One sentence what. One sentence why this role.*

> "I'm a product manager who builds AI systems and measures them — not at the whiteboard level, but in production. I've shipped a working image quality measurement system, five multi-agent pipelines, and a competitive intelligence system that runs at $0.003 per report. I'm here because the Foundations PM role is exactly where I want to apply that — building the eval and architecture layer that every Adobe creative surface depends on."

**Signal the structure:**
> "I want to cover three things: how I think about Firefly's quality gap and how to close it, the systems I've built that map to these challenges, and how I'd approach the first 90 days."

---

## Block 2 — Technical Depth: Where Firefly Stands + Eval Framework (2:00–6:00)

*Lead with real data — this signals you've done the research, not just the frameworks.*

### Start with the competitive position

> "I want to anchor this in real data, not theory. On the Artificial Analysis arena today, Firefly 3 has an ELO of 971 on text-to-image. GPT Image 2 is at 1339 — that's a 368-point gap. Midjourney is at 1093. On the image editing leaderboard, which is the leaderboard that matters most for AI Studio — Firefly doesn't appear in the top five. GPT Image 1.5 leads at 1264.

> The 368-point gap is not one problem. It's three separate problems, each with a different owner and a different fix:
>
> First — text rendering accuracy: Firefly 3 is below 45% on complex text in images. This is a LoRA problem, not a foundation model problem. Targeted fine-tuning on typography failure pairs can close most of this gap in 90 days.
>
> Second — image editing quality: not in the top 5 on the editing arena. GPT Image and Gemini dominate. This is the most urgent gap because AI Studio is Firefly's product differentiator — it's the thing no competitor can replicate inside a professional workflow.
>
> Third — aesthetic quality ceiling: 122 ELO points behind Midjourney. This is a training data curation and human preference feedback loop problem. Longer to close, but the lever is the eval pipeline."

### The eval framework — why current measurement is part of the problem

> "If those gaps persist, part of the reason is how quality is currently measured. Most teams reach for FID and CLIP Score and stop there. Both are wrong in specific ways.

> FID measures statistical distance between distributions. A model generating statistically average images scores well on FID while failing every professional. CLIP Score measures semantic alignment — a 'dramatic lighting' prompt scores well while the image feels flat. Neither tells you whether a professional will use the output.

> The ship gate should be human preference rate ≥ 75%. Below 60% is a coin flip. You can't close a 368-point ELO gap if the eval pipeline can't detect the delta.

> The architecture I'd build: three layers. Automated metrics on every build for regression detection. Deterministic pixel math for style consistency — no API cost, sub-second, catches batch incoherence that FID misses. Human preference ELO as the final gate. Automated scores tell you direction. Human signal tells you arrival.

> And critically — that human signal has to feed back into training. If it's a one-time gate and not a compound loop, the gap doesn't close."

### Routing decision

> "On cost and latency: route Express to a distillation model, reserve the foundation model for CC Pro. Flow matching reduces inference steps and brings Generative Fill under 2 seconds — the professional adoption threshold."

---

## Block 3 — Projects as Proof Points (6:00–11:00)

*Three projects, 90 seconds each. What I built → the principle → how it applies to Firefly.*

### Stil — Feed Cohesion Score (6:00–7:30)

> "I built an image quality measurement system called the Feed Cohesion Score — a 0–100 consistency rating across a creator's social feed using deterministic pixel math: color temperature, brightness, contrast, saturation variance. No model, no API cost, sub-second at any scale.

> The design principle: you don't need a learned quality metric to measure consistency. Pixel math is faster, cheaper, more interpretable. You add a model only where pixel math can't reach.

> Direct application to Firefly: this is Layer 2 of the eval stack. Does a batch of generations from the same prompt hold tonal range across different seeds? Does a Custom Model produce brand-consistent outputs across 50 generations? Same math. That measurement costs nothing to run and catches failures before they reach human eval."

### Vigil + GSentinel — Agentic architecture (7:30–9:00)

> "Five multi-agent systems shipped. The consistent finding: FSM orchestration beats LLM orchestration for well-defined workflows. Deterministic state transitions give you a full audit trail — critical when you're shipping to 30 million Creative Cloud users and need to know exactly why an output failed.

> In Vigil, RAG-first retrieval cut hallucinated queries by routing known patterns through retrieval before generation. MTTR dropped from 47 minutes to 35 seconds. The principle: heuristics and retrieval carry the load until they genuinely can't. Then you pay for inference.

> For Firefly: retrieve known prompt structures, style seeds, brand constraints first. Generate only where retrieval fails. Inference cost stays predictable. Quality stays consistent."

### Content Trust Agent — C2PA + SynthID (9:00–10:30)

> "Most recently, I designed a Content Trust Agent for Adobe Stock. Buyers were experiencing AI content getting through the 'Exclude AI' filter because the filter relied on contributor self-declaration. The fix: read C2PA manifest at intake and run Google's SynthID detection — bypassing self-declaration entirely. Meta does the same with IPTC Digital Source Type.

> The Foundations insight: Adobe co-founded C2PA. Every Firefly output already carries a Content Credential. That infrastructure exists in Firefly, Photoshop, and Express — it's not connected to the rest of the platform. Connecting it is an engineering project, not a research project.

> Why this matters for this role: Firefly's commercially safe training claim is only as credible as its provenance infrastructure at scale. C2PA is the proof layer. As the 47.85% AI content problem compounds on Stock, Firefly's verified provenance becomes a competitive asset that GPT Image and Midjourney can't match."

---

## Block 4 — First 90 Days: Three Concrete Moves (11:00–13:30)

*Not "I'd learn the product." Actual decisions anchored to the data above.*

> "Three things in the first 90 days, tied to the specific gaps I just named.

> **First — define the eval contract per segment.** Right now there's likely a single quality bar for all surfaces. That's why the ELO gap exists but isn't being closed systematically — research is optimizing for the wrong target for half the segments. Express quality bar is 'good enough to post.' CC Pro quality bar is 'survives professional art direction.' GenStudio quality bar is 'brand-compliant without revision.' Three different numbers. One contract per segment ends the ship/no-ship debate.

> **Second — ship the text rendering LoRA in 90 days.** Text accuracy below 45% is a named, specific, measurable gap. Ideogram built a specialty here. The fix is not retraining the foundation model — it's a targeted LoRA adapter on typography failure pairs. This is the fastest ELO-moving lever available without a model research investment. 90 days is achievable.

> **Third — close the feedback loop.** Human preference sessions are currently a gate, not a loop. The output of every eval session should become a training brief to research within two weeks: 'Segment A preference rate dropped 3 points on lighting coherence — here are the 47 failing pairs.' That brief drives the next LoRA fine-tune. That's how you systematically close a 368-point gap instead of hoping the next training run improves things.

> The metric I'd track above all others: **repeat usage rate at 7 days**. Not ELO, not generation count. Repeat usage means the output was good enough to trust again. That's what we're actually trying to build."

---

## Block 5 — Close + Questions (13:30–15:00)

> "That's the framework — competitive position, eval stack, three 90-day moves. One question before I stop: where does the team currently feel most stuck — is it the eval pipeline not giving clear signal, the model quality ceiling, or the cross-team alignment on what 'good' means?"

**Question 2 if time:**
> "The image editing arena gap is what I'd prioritize first given AI Studio is the differentiator. Is there existing work on the editing quality side, or is that a greenfield opportunity?"

---

## Cheat Sheet — Deliver Without Hesitation

| Topic | Say this exactly |
|---|---|
| **Firefly ELO today** | 971 text-to-image (Artificial Analysis, May 2026) |
| **GPT Image 2 ELO** | 1339 — gap of 368 points |
| **Midjourney ELO** | 1093 — gap of 122 points |
| **Image editing** | Firefly not in top 5; GPT Image 1.5 leads at 1264 |
| **Firefly text rendering** | <45% accuracy on complex text (Firefly 3) |
| **FLUX.1-dev HuggingFace** | 12.9k likes, 716k downloads — the open-weight community benchmark |
| **The three ELO gaps** | Text rendering (LoRA, 90 days) · editing quality (fine-tune + eval, 90 days) · aesthetic ceiling (training data, 18 months) |
| **Ship gate** | Human preference rate ≥ 75% — below 60% is a coin flip |
| **Professional latency threshold** | Sub-2-second Generative Fill — above this, professionals abandon |
| **Flow matching** | Fewer inference steps → lower latency at same quality |
| **LoRA vs. full retrain** | 10–100× cheaper; Custom Models is already this pattern |
| **FID limit** | Statistically average images score well; misses artistic quality |
| **CLIP Score limit** | Measures semantic alignment, not aesthetic nuance |
| **Blank Drop test** | Remove image input — if output doesn't degrade, model wasn't reading the image |
| **Image Sensitivity test** | Swap image, keep prompt — output must change appropriately |
| **Vigil MTTR** | 47 min → 35 sec |
| **GSentinel auto-fix rate** | 67% |
| **Feed Cohesion Score inputs** | Color temp · brightness · contrast · saturation variance |
| **Firefly moat** | IP indemnification + CC integration + Custom Models + commercially safe training — no competitor has all four |
| **C2PA** | Adobe co-founded it; every Firefly output carries a Content Credential |
| **SynthID** | Google DeepMind — 10B+ pieces watermarked; survives cropping and compression |
| **AI content on Stock** | 47.85% of all Adobe Stock images are AI-generated (PetaPixel, May 2025) |
| **CI pipeline cost** | $0.003 per Firefly competitive report |

---

## Push-Back Scripts

**"We don't use FID/CLIP Score internally"**
> "That's genuinely useful — what's the primary signal your team uses today? I'd be curious whether it's more human preference-weighted or automated, and whether it's segment-specific or a single bar across surfaces."

**"Our ELO position is different from what you cited"**
> "Good to know — the Artificial Analysis arena is one data point, and I'd defer to internal data. The reason I cited it is that it's what the market sees, so even if internal eval says something different, the arena perception shapes how enterprise buyers and designers compare tools. What does your internal eval show?"

**"How would you improve video quality specifically?"**
> "Start with temporal consistency as a regression metric on every build — that's the dimension that maps to 'this video feels choppy,' which is the user complaint you can act on. VBench gives 16 dimensions but temporal flicker is the one that drives abandonment. Once that's instrumented, run human preference ELO on outputs that score ≥ 80 on consistency, to check whether the overall feel actually lands."

**"Tell me more about the competitive picture"**
> "Firefly's ELO gap is real and addressable, but the market framing is different. Midjourney is in active litigation — enterprise can't use it. GPT Image 2 leads on quality but lives inside ChatGPT with no CC integration. FLUX leads the open-weight ecosystem with 12.9k HuggingFace likes but has no commercial safety guarantee. Firefly is the only option that is commercially safe, workflow-integrated, and brand-customizable. The gap to close is quality — and that's the Foundations PM job."

**"What's your view on open-weight vs. proprietary for Firefly?"**
> "Layer-specific: foundation model stays Firefly — the commercially safe training data is the moat that no open-weight model replicates. Brand customization goes to LoRA adapters on Firefly — full retraining is millions of dollars. Domain grounding goes to RAG. The open-weight models like FLUX matter as a benchmark and as the community fine-tuning ecosystem that generates signal on what techniques work. We should be reading their LoRA research, not building an alternative foundation."

---

*Bharat Namatherdhala · Foundations PM Interview Prep · May 2026*
