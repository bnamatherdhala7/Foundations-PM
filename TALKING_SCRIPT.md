# Talking Script — Adobe Principal PM, Research & AI (Foundations)

*Conversational, not a recitation. Know the structure; improvise the words.*

---

## Opening — 30 seconds

> "My background is in building AI products where the hard part is defining what good looks like — not workflow tools on top of existing models, but systems where the evaluation framework is itself a product decision. That thread runs through everything I've built: image quality measurement, competitive positioning for generative AI, and agentic architectures where the model tradeoff is baked into the design. I've been focused on Firefly's foundational challenges specifically because I think there's a short window where the enterprise positioning gap is exploitable, and I want to be the person closing it."

---

## Thread 1: Evaluation Frameworks → your clearest proof point

*JD map: "build scalable evaluation frameworks" and "set quality benchmarks."*

**Lead with what you built — then extend it to Firefly:**
> "The most directly relevant project is Stil — a creative editing assistant I built for content creators. The core infrastructure piece is the Feed Cohesion Score: a 0–100 consistency number across a creator's feed, computed using deterministic pixel math — color temperature variance, brightness, contrast, saturation. No model inference. Sub-second at any scale. Zero API cost.

> The design principle it demonstrates: you don't need a learned quality metric to measure consistency. Pixel math is faster, cheaper, and more interpretable. You add a model only where pixel math can't reach.

> For Firefly, that same architecture maps to three evaluation dimensions. The key framing first: automated scores often fail to capture human-centric 'feel.' So I build a stack where heuristics carry the bulk of the load and human signal validates where they can't."

**The three-layer Firefly eval stack:**
> "**Layer 1 — Prompt adherence (CLIP Score):** Semantic alignment between the user's text prompt and the generated image, using CLIP embeddings. This is the only layer that requires model inference, because pixel math can't reach semantic intent. Important caveat: CLIP Score measures statistical alignment, not artistic interpretation. A prompt for 'dramatic lighting' can score well on CLIP while the actual lighting feels flat to a human eye.

> **Layer 2 — Style consistency (pixel math):** Given N outputs from the same prompt and style reference, what's the variance in color temperature, brightness, contrast, saturation? Same math as Feed Cohesion Score. Measures whether a batch is visually coherent with itself and with the reference — without touching inference budget.

> **Layer 3 — Human preference signal (ELO):** Pairwise ranking sessions — 'which of these two do you prefer?' — fed into an ELO system, then correlated with generation parameters: sampler settings, CFG values, style-reference weights. This closes the gap between what the automated metric measures and what users actually want. This is the layer you can't replace with a score."

**North star metric:**
> "Human Preference Rate — what percentage of generations does a user prefer over the alternative they were shown? Below 60% is coin flip. Target is 75%+. CLIP Score and Feed Cohesion Score are leading indicators; HPR is the lagging validator that tells you whether the automated metrics are actually tracking user value."

---

## Thread 2: Evaluation Benchmarks — the metric map

*JD map: "provide expert feedback on model quality" — you need to know these and their limits.*

**The framing to open with:**
> "One thing I'm deliberate about as a PM: I know the standard benchmarks, but I also know what each one can't measure. Automated scores consistently fail to capture human-centric 'feel.' Using FID alone to make a ship decision is like using load time as your only UX metric."

**Image generation — three metrics, three limits:**
> "**FID (Fréchet Inception Distance):** The industry standard. Measures the statistical distance between the feature distributions of real images and generated images. Lower is better. The limit: FID captures whether your output distribution looks statistically similar to real images — it says nothing about whether an individual image is artistically good or prompt-faithful. A model that generates statistically average images will score well on FID while failing creative professionals.

> **IS (Inception Score):** Measures both clarity of objects and diversity of output. The limit: IS is heavily dependent on the pre-trained classifier it uses, and it often fails to detect poor diversity *within* a class. A model that generates 1,000 slightly different photos of the same dog breed will score high on IS while being practically useless for a creative who needs variety.

> **CLIP Score:** Semantic alignment between text prompt and generated image. This is the essential metric for instruction following — it tells you whether the model understood the prompt. The limit: CLIP measures statistical co-occurrence in its training data. It can miss cultural, aesthetic, or contextual nuance that a human reader of the prompt would catch immediately."

**Video generation — the time dimension makes this harder:**
> "**VBench:** Currently the gold standard evaluation suite for video. It breaks quality into 16 fine-grained dimensions — motion smoothness, temporal flickering, subject consistency, spatial relationships, background stability. The value is specificity: instead of 'this video is bad,' VBench tells you exactly which dimension is failing, which tells you where to direct the research.

> **FVD (Fréchet Video Distance):** The video equivalent of FID. Measures distribution similarity between real and generated video clips. Inherits FID's blind spots — it won't catch an artistically compelling but statistically unusual output as 'good.'

> **Temporal Consistency / Flicker Metrics:** Specialized algorithms measuring pixel displacement between frames to identify jitter or stutter. This is the one metric that directly maps to a user's subjective complaint — 'the video feels choppy' — which makes it the most actionable for a product decision."

**Multimodal LLM evaluation — understanding, not just generation:**
> "**LLM-as-a-Judge (MT-bench):** Using a high-capability model — GPT-4o or Claude — to grade the multimodal model's response against a rubric. The value is scalability; the risk is the judge's biases becoming your quality floor. If your judge has a blind spot, your eval has a blind spot.

> **Visual Sensitivity / Reliability Tests — the two I'd run first in production:**
> — *Blank Drop test:* Remove the image from the prompt. If the model's answer doesn't degrade significantly, it wasn't actually reading the image — it was pattern-matching on text. This is a production safety check, not a quality check.
> — *Image Sensitivity test:* Swap the image for a different one while keeping the prompt identical. If the answer doesn't change appropriately, the model's visual grounding is unreliable.

> **LMMS-Eval:** A comprehensive framework for evaluating across text, image, and video tasks in a standardized pipeline. The key value for a Foundations PM: reproducibility and statistical significance. You can't make research-to-product decisions on eval results that shift 3 points run-to-run due to sampling variance."

**The line that lands:**
> "The way I use these together: CLIP Score and FID for tracking model progress over time, VBench dimensions for video-specific regression testing, and human preference sessions as the final gate before a capability moves from Track 2 to Track 1. Automated scores tell you *direction*. Human signal tells you *arrival*."

---

## Thread 3: Quality / Cost / Latency — the triangle

*JD map: "build an evidence-based view on quality, cost, and latency, making strategic tradeoff decisions."*

> "Adobe serves everyone from hobbyists in Express to enterprise teams in GenStudio. For something like Generative Fill, I don't optimize for 'better' — I balance three dimensions simultaneously.

> Latency: professionals expect sub-2-second generation. Quality: the output has to handle complex compositing — maintaining lighting when replacing a subject's jacket, not just swapping color. Cost: foundation model inference runs real money at Firefly's scale.

> The tradeoff I'd make: route Express users to a lighter, faster distillation model — lower latency, lower cost, sufficient quality for social content. Reserve high-compute foundation models for Creative Cloud Pro tier, where willingness-to-pay covers the inference cost and the workflow demands deeper architectural control.

> This is a product architecture decision, not just a cost optimization. The distillation model needs to be good enough that Express users don't feel they have a lesser product. The quality ceiling for a hobbyist making an Instagram post is genuinely different from a creative director compositing a campaign hero image at 200% zoom. Matching model capability to segment threshold is how you serve both users correctly."

**Where you draw the routing line:**
> "The test: can the user tell the difference *in their workflow context*? A hobbyist on mobile at 1× zoom will not see the gap between distillation and foundation on most prompts. A retoucher in Photoshop at 200% zoom on a complex edge case absolutely will. That observation — not a general quality score — determines the routing threshold."

---

## Thread 4: Research → Requirements

*JD map: "work alongside model researchers to guide research direction" and "transform early-stage capabilities into market-ready requirements."*

> "The gap between research and product: research optimizes for the impressive demo. Product needs the predictable baseline. Those are different objectives, and bridging them is most of the job.

> I translate early research into requirements by defining 'control surfaces' — the knobs the creator needs to steer output — rather than waiting for the model to guess better. For Firefly video, that's the difference between a researcher showing you beautiful motion generation in a controlled demo and a product a marketer can actually use. The marketer doesn't need perfect motion. They need motion they can predict and adjust.

> Concretely: if a researcher shows me a promising motion model, my next question isn't 'what's your FVD score?' It's 'what parameters can a user control to get from this interesting output to a predictable one?' If the answer is 'nothing yet,' the research isn't ready to translate to requirements. If the answer is 'motion speed and amplitude, anchored to reference frames,' I can write the requirement: lock a reference frame, set motion intensity 1–10. That's a control surface. That's what moves research from 'interesting' to 'shippable.'"

**On researchers who resist productization:**
> "I don't frame it as productization vs. research. I frame it as: 'here's the benchmark that proves your work is ready for the next step.' Researchers want their work to matter. A well-defined, achievable benchmark gives exploration direction without constraining it. The benchmark is the bridge, not the constraint."

---

## Thread 5: Technical Depth — speaking the engineer's language

*JD map: "strong technical knowledge to engage with model researchers about architecture, quality metrics, and tradeoffs."*

> "I'm comfortable discussing model tradeoffs at the architecture level.

> On the generation pipeline: the evolution from basic latent diffusion to flow matching is directly relevant for Firefly. Flow matching gives better sample efficiency — high-quality outputs in fewer inference steps — which improves latency without sacrificing quality. That's not just a research result; it's what unlocks sub-2-second generation on the Pro tier.

> On Style IDs and Custom Models: we're not fine-tuning a model. We're building a modular system where user-provided reference assets act as low-rank adapters — LoRA or similar grounding mechanisms — that constrain the generation space to a brand's specific aesthetic without full retraining. The product implication: quality of the brand lock depends on quality and diversity of the user's training assets. That's a user education problem as much as a model problem."

**From my own projects:**
> "The pattern I keep applying: use the cheapest model that can close the gap, and be explicit about where it can't. In Stil, every task runs on Haiku — agent loop, style extraction, insights grading — under $0.05 per user per day. Feed consistency scoring runs on Pillow: no model, no cost. I add Sonnet only where Haiku fails on complex aesthetic intent.

> In Vigil — a network incident investigation agent I built on Splunk's MCP server — the tradeoff was model inference vs. RAG-retrieved SPL patterns. RAG-first: Pinecone retrieves vetted query patterns at each step before the model generates anything. You add generation only where retrieval fails. Same principle as the Firefly eval stack: heuristics and retrieval carry you until they genuinely can't."

---

## Thread 6: Competitive Differentiation — the Content Supply Chain

*JD map: "analyze the competitive landscape and find opportunities to differentiate."*

> "Firefly's moat isn't the model. It's the Content Supply Chain. Midjourney generates an image in a silo. Firefly generates an image that's already in a PSD file, has embedded Content Credentials metadata, and is ready for export to Adobe Experience Manager. The goal isn't better generation — it's maximizing value-per-generation by making the output immediately useful in the user's workflow.

> When Firefly generates vectors in Illustrator, those aren't just paths — they're layer-organized, editable paths that slot into an existing design file. The workflow integration *is* the product.

> Competitive map: Canva wins on distribution (260M MAU), Midjourney wins on creative quality for professionals, OpenAI wins on API and developer reach. Firefly's white space: enterprise-to-creative pipeline. The only AI image generator your legal team will approve that also works inside your existing creative stack. IP indemnification plus CC integration plus Custom Models trained on your brand library. No competitor can say that sentence."

**The gap I'd close first:**
> "Midjourney's active lawsuit exposure has left the 'commercially safe AI image generator' keyword cluster completely unclaimed. Firefly is the factually correct answer to every query in that cluster and doesn't appear. That's a 90-day organic acquisition win — a CMS change and three landing pages. Enterprise procurement teams are already searching for this. Firefly just isn't visible when they do."

---

## Thread 7: Roadmapping in a Research Environment

*JD map: "comfort operating in research environments with evolving success definitions."*

> "I use Dual-Track Discovery.

> Track 1 is Features — Generative Expand, Generative Fill improvements — defined success criteria, near-term delivery. Track 2 is Capabilities — long-term bets on agentic workflows and next-gen models — where success is a series of benchmarks, not a ship date. For Track 2, the roadmap isn't 'ship agentic workflows by Q3.' It's 'demonstrate 5-step autonomous task completion at 90% accuracy by Q3, which unlocks the productization decision.'

> The anchor that keeps both tracks coherent: define the measurement system and quality threshold before you define the feature. The benchmark is stable even when the technical path to it changes.

> For Firefly video: the roadmap question isn't 'when can we ship text-to-video.' It's 'what does good look like at each VBench dimension — temporal consistency, motion smoothness, subject consistency — and what's the minimum viable threshold per segment?' A professional creative and an SMB marketer have different thresholds. Those are different release gates."

---

## Thread 8: Customer Insights — three segments, three insights

*JD map: "develop deep customer insights" and "interpret needs from creative professionals, creators, and businesses."*

> "**Creative professionals (Photoshop/Illustrator)** — Stil was built for this segment. The insight: their problem isn't finding an AI tool, it's visual drift. 200+ pieces of content a year, and their feed stops looking like them six months in because 200 micro-decisions under time pressure slowly diverge from original intent. Firefly implication: consistency-at-volume is the value proposition, not generation quality per se. Every output needs to feel like it came from the same brand intent across a campaign.

> **Creators and content producers (Express)** — StoryForge was built here. The insight: they don't have a creativity problem. They have a production bottleneck. One 15-second video takes 3 hours. They can't test which hook resonates because producing one variation is already a full effort. Firefly video requirement for this segment: produce 10 variations of the same concept in one session at no marginal cost. That's a different product than the professional use case.

> **Business users (GenStudio/Campaign Manager)** — competitive intelligence and MailIntel work informed this. The insight: they pay for AI that tells them what to do next, not just AI that generates. Firefly enterprise value: every brief that goes into Campaign Manager auto-generates on-brand creative already cleared for commercial use and tagged with Content Credentials for compliance. The feedback loop — output → performance signal → model improvement — has to be visible and causal for the enterprise buyer to renew."

---

## Three principles that anchor every answer

**1. Commercially safe first.** Adobe's commercially licensed training data and IP indemnification is the enterprise procurement differentiator. No competitor offers it. Lead with it whenever legal, compliance, or enterprise buyers come up.

**2. Workflow integration, not replacement.** The AI enters the Adobe app; it doesn't replace the creative. Generative Fill eliminates tedious compositing so the retoucher spends more time on decisions only they can make. Frame is always "co-creation."

**3. Human retains artistic control.** AI does the heavy lifting; the human makes the final call. This answers the democratization trap and any concern about displacement.

---

## The democratization trap — counter it directly

*If asked: "Doesn't AI make professional tools too easy, eroding the value of expertise?"*

> "Adobe is moving the goalposts. The value was never in rendering — rendering is becoming a commodity. The value is in the integration: how a generated asset fits into a global campaign, is tagged with Content Credentials for compliance and attribution, adheres to brand guidelines via Custom Models, and routes through AEM for approval and localization. That chain requires expertise to design and operate. Firefly commoditizes the pixel-pushing. It doesn't commoditize the judgment about which pixel-push was right for this brief, this audience, this channel, and this regulatory context. That judgment is what a Foundations PM is building infrastructure to support."

---

## Close

> "The reason I want Foundations specifically is that it compounds. Every Firefly feature that ships in Photoshop, Express, and Campaign Manager runs on the same generation pipeline. If I improve the evaluation framework at the foundation layer, every surface inherits it. A PM on a single surface builds a great feature. A PM on foundations changes what's possible for every team that builds on top of it. That's the leverage I'm looking for — and I've spent the last year building the technical and strategic context to be useful here on day one."

---

## Quick-fire answers

**"What's your biggest limitation?"**
> "I haven't worked inside a model research organization, so I'll ramp on how prioritization flows between exploration and production alignment. What I bring is the ability to define quality metrics that researchers can actually optimize against — which I think is the harder half of that interface to find in a PM hire."

**"Walk me through a quality/cost/latency tradeoff."**
> Express distillation model vs. CC Pro foundation model. Segment-specific quality threshold, not a general score. (Thread 3.)

**"How do you work with researchers?"**
> Control surfaces framing. Define what the creator needs to steer output. Ask 'what can the user control?' not 'what's the FID score?' (Thread 4.)

**"What metrics would you use to evaluate Firefly image quality?"**
> Layer 1: CLIP Score for prompt adherence (knows its limit — misses aesthetic nuance). Layer 2: pixel math for style consistency (cheap, fast, interpretable). Layer 3: ELO-based human preference as the final gate. FID for tracking model progress over time. (Thread 2.)

**"How would you evaluate Firefly video quality?"**
> VBench as the primary suite — 16 dimensions, specific enough to direct research. FVD for distribution-level tracking. Temporal consistency metrics for the user-facing 'choppiness' complaint. Human preference sessions before any capability moves to Track 1. (Thread 2.)

---

## Numbers to know cold

| Metric / Fact | Value |
|---|---|
| Firefly moat | IP indemnification + CC integration + Custom Models — no competitor has all three |
| Canva MAU | 260M |
| Pro latency expectation | Sub-2-second (Generative Fill) |
| VBench dimensions | 16 (motion smoothness, temporal flickering, subject consistency, spatial relationships, +12) |
| Human Preference Rate target | ≥ 75% (below 60% = coin flip) |
| FID limit | Statistical distribution only — misses artistic quality |
| IS limit | Classifier-dependent — misses within-class diversity |
| CLIP Score limit | Statistical alignment — misses aesthetic nuance |
| Blank Drop test | Performance should degrade when image removed; if not, model isn't reading the image |
| Image Sensitivity test | Answer should change when image swapped; if not, visual grounding is unreliable |
| Flow matching benefit | Fewer inference steps → lower latency at same quality |
| LoRA / Style ID | Low-rank adapters constrain generation to brand aesthetic without full retraining |
| Stil cost target | < $0.05 / user / day |
| GSentinel auto-fix rate | 67% |
| Vigil MTTR | 47 min → 35 sec |
| StoryForge PVR target | ≥ 60% (Posted Video Rate — did they actually post it?) |
| StoryForge TTFV | < 5 min at 95th percentile |
| Hook Performance Variance | ≥ 3× top vs. bottom hook |
| CI system cost | $0.003 / Firefly report (Haiku pipeline) |
| Feed Cohesion Score inputs | Color temp, brightness, contrast, saturation variance |
