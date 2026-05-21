# Talking Script — Adobe Principal PM, Research & AI (Foundations)

*Conversational, not a recitation. Know the structure, improvise the words.*

---

## Opening — Position yourself in 30 seconds

> "My background is in building AI products that require actual technical depth to get right — not workflow tools on top of existing models, but systems where the hard part is defining what good looks like. That's the thread across everything I've built: evaluation frameworks, competitive positioning for generative AI, and agentic architectures where the model tradeoff decisions are part of the product design. I've been focused specifically on Firefly's foundational challenges because I think there's a short window where the enterprise positioning gap is exploitable, and I want to be the person closing it."

---

## Thread 1: Evaluation Frameworks → your clearest proof point

*JD map: "build scalable evaluation frameworks" and "set quality benchmarks."*

**The bridge from what you built:**
> "The project most directly relevant to this role is Stil — a creative editing assistant I built for content creators. One of the core infrastructure pieces is something I call the Feed Cohesion Score. It gives a 0–100 number to visual consistency across a creator's feed using deterministic pixel math — color temperature variance, brightness, contrast, saturation. No model inference. Sub-second at any scale. Zero API cost.

> The insight it demonstrates is that you don't need a learned quality model to measure consistency. Pillow math is faster, cheaper, and more interpretable. You add a model only where pixel math genuinely can't reach.

> For Firefly, that same architecture extends directly into three evaluation dimensions:

> First — **prompt adherence**: does the output contain what the prompt asked for? One CLIP call per image, semantic similarity between prompt embedding and image. This is the one dimension that requires a model because pixel math can't reach semantic intent.

> Second — **style consistency across variations**: given N outputs from the same prompt and style reference, what's the variance in color temperature, brightness, contrast, saturation? Same math as Feed Cohesion Score, different input. Measures whether a batch of Firefly outputs is coherent with each other and with the reference.

> Third — **human preference signal**: pairwise ranking sessions fed into an ELO system, then correlated with generation parameters — sampler settings, CFG values, style-reference weights. That's the signal that closes the gap between what the pixel metric measures and what users actually want.

> The system becomes a quality gate: every batch that goes through Firefly web runs the consistency check automatically. If variance is above threshold, the batch is flagged before it reaches the user."

**North star metric answer:**
> "Human Preference Rate — what percentage of generations does a user prefer over the alternative they were shown? Below 60% is coin flip. Target is 75%+. Feed Cohesion Score is the leading indicator; HPR is the lagging validator."

**Quality gap vs. Midjourney:**
> "The evaluation framework above is how you close it — or at least measure where the gap is precisely enough to direct research. The prompt adherence score tells you whether the gap is semantic (model misses intent) or aesthetic (model captures intent but style falls short). Those are different research problems with different solutions."

---

## Thread 2: The Triangle of Constraints — Quality, Cost, Latency

*JD map: "build an evidence-based view on quality, cost, and latency, making strategic tradeoff decisions."*

**The framing:**
> "Adobe serves everyone from hobbyists in Express to enterprise marketing teams in GenStudio. When I think about the roadmap for something like Generative Fill, I don't look for 'better' images — I balance three dimensions simultaneously.

> Latency: the professional user expects sub-2-second generation times. Quality: the output has to handle complex compositing — maintaining lighting when replacing a subject's jacket, not just swapping a color. Cost: inference on a foundation model runs real money at scale.

> The strategic tradeoff I'd make: route Express users to a lighter, faster distillation model — lower latency, lower inference cost, sufficient quality for social content. Reserve the high-compute foundation models for Creative Cloud Pro tier, where the willingness-to-pay covers the higher inference cost and the workflow demands deeper architectural control.

> This isn't just a cost optimization. It's a product architecture decision: the distillation model for Express needs to be good enough that users don't feel they're getting a lesser product, but the quality ceiling for a hobbyist making an Instagram post is genuinely different from a creative director compositing a campaign hero image. Matching model capability to segment need is how you make both of those users happy without serving either of them wrong."

**If they push on how you'd decide where to draw the line:**
> "The test I'd use: can the user tell the difference in their workflow context? A hobbyist viewing output on mobile at 1× zoom will not see the difference between the distillation model and the foundation model on most prompts. A retoucher working at 200% zoom in Photoshop absolutely will on complex edge cases. That observation determines where the routing threshold is — not a general quality score, but a segment-specific quality threshold."

---

## Thread 3: Translating Research to Requirements

*JD map: "work alongside model researchers to guide research direction" and "transform early-stage capabilities into market-ready requirements."*

**The core framing — research wants zero-shot perfection, you need commercial-ready consistency:**
> "The gap I've seen between research and product is this: research optimizes for the impressive demo. Product needs the predictable baseline. Those are different objectives, and bridging them is most of the job.

> The way I do it: I translate early-stage research into requirements by defining 'control surfaces' — the knobs the creator needs to steer the AI's output — rather than waiting for the model to guess better. For Firefly video, that's the difference between a researcher showing you beautiful motion generation in a controlled demo and a product that a marketer can actually use. The marketer doesn't need perfect motion. They need motion they can predict and adjust.

> Concretely: if a researcher shows me a promising video motion model, my next question isn't 'what's the FID score?' It's 'what parameters can a user control to get from this interesting output to a predictable one?' If the answer is 'nothing yet,' the research isn't ready to translate to requirements. If the answer is 'speed and amplitude of motion, anchored to reference frames,' now I can write a requirement: users can lock a reference frame and define motion intensity on a 1–10 scale. That's a control surface. That's what moves research from 'interesting' to 'shippable.'"

**If they ask how you handle researchers who resist productization:**
> "I don't position it as productization vs. research. I position it as: 'here's the benchmark that would prove your work is ready for the next step.' Researchers want their work to matter. If the benchmark is well-defined and achievable, most researchers will orient toward it — because it gives their exploration direction without constraining it. The benchmark I define is the bridge, not the constraint."

---

## Thread 4: Technical Depth — the language of the engineer

*JD map: "strong technical knowledge to engage with model researchers about architecture, quality metrics, and tradeoffs."*

**Lead with the architecture evolution you understand:**
> "I'm comfortable discussing model tradeoffs at the architecture level. For image generation specifically: the evolution from basic latent diffusion to flow matching is meaningful for Firefly. Flow matching gives you better sample efficiency — you get to high-quality outputs in fewer inference steps — which directly improves the latency side of the quality/cost/latency triangle. That's not just a research improvement; it's a product decision that unlocks sub-2-second generation on the Pro tier without sacrificing quality.

> On Style IDs and Custom Models: when I think about brand consistency features, I understand we're not just fine-tuning a model. We're building a modular system where user-provided reference assets act as low-rank adapters — LoRA or similar grounding mechanisms — that constrain the generation space to a brand's specific aesthetic without full retraining. The product implication is that the quality of the brand lock depends on the quality and diversity of the training assets the user provides. That's a user education problem as much as it's a model problem."

**From my own projects:**
> "The pattern I keep applying: use the cheapest model that can close the gap, and be explicit about where it can't. In Stil, every task runs on Haiku — agent loop, style extraction, insights grading. Under $0.05 per user per day. Feed consistency scoring runs on Pillow — no model at all. I add Sonnet only where Haiku's interpretation fails on complex aesthetic intent.

> In Vigil — a network incident investigation agent I built on Splunk's MCP server — the tradeoff was model inference versus RAG-retrieved SPL patterns. RAG-first: Pinecone retrieves vetted query patterns at each investigation step before the model generates anything. You add model generation only where retrieval fails. Same principle as the Firefly evaluation framework — heuristics and retrieval carry you until they can't, then you bring in the expensive inference."

**Agentic architectures:**
> "I've built five multi-agent systems across different domains. The consistent finding: FSM orchestration beats LLM orchestration for well-defined workflows — deterministic state transitions, full audit trail, no LLM deciding what step comes next when the decision tree is already known. You use LLM orchestration only where the workflow genuinely can't be specified in advance. Most production creative workflows are not that open-ended."

---

## Thread 5: Competitive Differentiation — the Content Supply Chain

*JD map: "analyze the competitive landscape and find opportunities to differentiate."*

**The moat framing:**
> "Firefly's moat isn't the model — it's the Content Supply Chain. Midjourney generates an image in a silo. Firefly generates an image that is already part of a PSD file, has embedded Content Credentials metadata, and is ready for export to Adobe Experience Manager. The goal isn't better generation; it's maximizing value-per-generation by ensuring the output is immediately useful in the context of the user's workflow.

> Concrete example: when Firefly generates vectors in Illustrator, those aren't just vector paths — they're layer-organized, editable paths that slot into an existing design file. That's a different product than 'AI generates a vector and you export it.' The workflow integration is the product.

> On the competitive landscape: Canva wins on distribution (260M MAU flywheel), Midjourney wins on raw creative quality for professionals, OpenAI wins on API and developer reach. Firefly's white space is enterprise-to-creative pipeline — the only AI image generator your legal team will approve that also works inside your existing creative stack. IP indemnification plus CC integration plus Custom Models trained on your brand library. No competitor can say that sentence."

**The specific gap I'd fix first:**
> "Midjourney's active lawsuit exposure has left the 'commercially safe AI image generator' keyword cluster completely unclaimed. Firefly is the factually correct answer to every query in that cluster and doesn't appear in the rankings. That's a 90-day organic acquisition opportunity that costs nothing — a CMS change and three landing pages. The enterprise buying signal is already there; Firefly just isn't visible when procurement teams are searching."

---

## Thread 6: Roadmapping in a Research Environment

*JD map: "comfort operating in research environments with evolving success definitions."*

**The Dual-Track framing:**
> "I manage roadmap evolution in research environments using Dual-Track Discovery.

> Track 1 is Features — shipping the Generative Expand tools and Generative Fill improvements that users expect. These have defined success criteria and near-term delivery. Track 2 is Capabilities — long-term bets on agentic workflows and next-generation models, where success is a series of benchmarks, not a ship date. For example: how many steps can Firefly AI Assistant automate within an Adobe app without human intervention? The roadmap for Track 2 isn't 'ship agentic workflows by Q3.' It's 'demonstrate 5-step autonomous task completion at 90% accuracy by Q3, which unlocks the decision to productize.'

> The anchor that keeps both tracks coherent: define the measurement system and the quality threshold before you define the feature. That sequence holds the roadmap stable as research direction evolves, because the benchmark is fixed even when the technical path to it changes.

> For Firefly video specifically: the roadmap question isn't 'when can we ship text-to-video.' It's 'what does good look like at each quality dimension — temporal consistency, motion naturalness, prompt fidelity — and what's the minimum viable threshold for each segment?' A professional creative and an SMB marketer have different thresholds. Those are different release gates."

---

## Thread 7: Customer Insights → three segments, three specific insights

*JD map: "develop deep customer insights" and "interpret needs from creative professionals, creators, and businesses."*

> "The Adobe customer base is not one segment, and the Firefly requirement is different for each.

> **Creative professionals (Photoshop/Illustrator users)** — I built Stil for this segment. The insight: their problem isn't finding an AI tool. It's visual drift. 200+ pieces of content a year, and their feed stops looking like them six months in because 200 micro-decisions under time pressure slowly diverge from their original intent. They don't notice it happening; they notice it six months later. The Firefly implication: consistency-at-volume is the value proposition for this segment, not generation quality per se. Every Firefly output needs to feel like it came from the same brand intent across a campaign, not just like it's technically good in isolation.

> **Creators and content producers (Express users)** — StoryForge was built for this segment. The insight: they don't have a creativity problem. They have a production bottleneck. One good 15-second video takes 3 hours. The 'hook testing deficit' means they can't test which angle resonates because producing even one variation is a full production effort. The Firefly video requirement for this segment isn't 'is the quality good enough' — it's 'can I produce 10 variations of the same concept in one session at no marginal cost?' That's a different product than the professional use case.

> **Business users (GenStudio/Campaign Manager)** — my competitive intelligence and MailIntel work informed this segment. The insight: business users pay for AI that tells them what to do next, not just AI that generates content. The Firefly enterprise value isn't 'generate images.' It's 'every brief that goes into Campaign Manager auto-generates on-brand creative that's already cleared for commercial use and tagged with Content Credentials for compliance.' The feedback loop that needs to exist: output → performance signal → model improvement → better output. That loop has to be visible and causal for the enterprise buyer to renew."

---

## The three principles to anchor every answer

**1. Commercially safe first.** Always mention Adobe's commercially trained data and IP indemnification. This is the enterprise procurement differentiator. No competitor offers it. Lead with it when legal, compliance, or enterprise buyers come up.

**2. Workflow integration over replacement.** Always talk about how AI enters the Adobe app, not how it replaces the creative. Generative Fill doesn't replace the retoucher — it eliminates the hours they spent on tedious compositing so they can spend more time on the decisions only they can make. "Co-creation" is the frame.

**3. Human retains artistic control.** The AI does the heavy lifting; the human makes the final call. This is the answer to the "democratization trap" and to any concern about AI displacing creative professionals.

---

## The democratization trap — counter it directly

*If asked: "Doesn't AI make professional tools too easy, reducing the value of expertise?"*

> "Adobe is moving the goalposts. The value was never in rendering — rendering is becoming a commodity. The value is in the integration: how the generated asset fits into a global campaign, is tagged with Content Credentials metadata for compliance and attribution, adheres to brand guidelines via Custom Models, and routes through Adobe Experience Manager for approval and localization. That entire chain requires expertise to design and operate. Firefly commoditizes the pixel-pushing. It doesn't commoditize the judgment about which pixel-push was right for this brief, this audience, this channel, and this regulatory context. That judgment is what you, as a Foundations PM, are building infrastructure to support."

---

## Close — why this role specifically

> "The reason I want this specific role is that foundational capabilities compound. Firefly features that ship in Photoshop, Express, and Campaign Manager all run on the same generation pipeline. If I improve the style consistency evaluation framework at the foundation layer, every surface inherits it. A PM working on a single surface builds a great feature. A PM working on foundations changes what's possible for every team that builds on top of it. That's the leverage I'm looking for — and I've spent the last year deliberately building the technical and strategic context to be useful here on day one."

---

## Quick-fire answers

**"What's your biggest limitation?"**
> "I haven't worked inside a model research organization, so I'll have a ramp on the internal research workflow — how prioritization happens between exploration and production alignment, how product requirements reach research teams in a form they can act on. I'm aware of that gap. What I bring is the ability to define quality metrics that researchers can actually optimize against, which I think is the harder half of that interface to find in a PM hire."

**"Walk me through a quality/cost/latency tradeoff."**
> Use the Express distillation model vs. CC Pro foundation model example from Thread 2. Specific, segment-aware, maps directly to Firefly's actual architecture decisions.

**"How do you work with researchers?"**
> Use the control surfaces framing from Thread 3. Define what the creator needs to steer output, not what the model needs to get better.

---

## Numbers to know cold

| Fact | Number |
|---|---|
| Firefly competitive edge | Only enterprise IP indemnification + CC integration + Custom Models combo |
| Canva MAU | 260M |
| Professional latency expectation | Sub-2-second for Generative Fill |
| Midjourney price | $10/month (no IP indemnification) |
| Stil cost target | < $0.05 / user / day |
| GSentinel auto-fix rate | 67% |
| Vigil MTTR | 47 min → 35 seconds |
| StoryForge PVR target | ≥ 60% |
| StoryForge TTFV | < 5 min at 95th percentile |
| Hook Performance Variance | ≥ 3× top vs. bottom hook |
| CI system cost | $0.003 per Firefly report (Haiku pipeline) |
| Feed Cohesion Score dimensions | Color temp, brightness, contrast, saturation |
| Human Preference Rate target | ≥ 75% |
| Flow matching benefit | Better sample efficiency → fewer inference steps → lower latency |
| LoRA / Style ID mechanism | Low-rank adapters constrain generation space to brand aesthetic without full retraining |
