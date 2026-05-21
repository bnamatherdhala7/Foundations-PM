# Talking Script — Adobe Principal PM, Research & AI (Foundations)

*Conversational, not a recitation. Know the structure, improvise the words.*

---

## Opening — Position yourself in 30 seconds

> "My background is in building AI products that require actual technical depth to get right — not workflow tools on top of existing models, but systems where the hard part is defining what good looks like. That's the thread across everything I've built: evaluation frameworks, competitive positioning for generative AI, and agentic architectures where the model tradeoff decisions are part of the product design. I've been focused specifically on Firefly's foundational challenges because I think there's a short window where the enterprise positioning gap is exploitable, and I want to be the person closing it."

---

## Thread 1: Evaluation Frameworks → your clearest proof point

*This maps to: "build scalable evaluation frameworks" and "set quality benchmarks."*

**The bridge from what you built:**
> "The project that's most directly relevant is Stil — a creative editing assistant for content creators. One of the core infrastructure pieces I built is something I call the Feed Cohesion Score. It gives a 0–100 number to visual consistency across a creator's feed using deterministic pixel math — color temperature variance, brightness, contrast, saturation. No model inference. Sub-second at any scale. Zero API cost.

> The insight it demonstrates is that you don't need a learned quality model to measure consistency. Pillow math is faster, cheaper, and more interpretable than a trained metric. You add a model only where pixel math genuinely can't reach.

> For Firefly, that same architecture extends directly into three evaluation dimensions:

> First — **prompt adherence**: does the output contain what the prompt asked for? One CLIP call per image, semantic similarity between prompt embedding and image. This is the one dimension that requires a model because pixel math can't reach semantic intent.

> Second — **style consistency across variations**: given N outputs from the same prompt and style reference, what's the variance in color temperature, brightness, contrast, saturation? Same math as Feed Cohesion Score, different input. Measures whether a batch of Firefly outputs is actually coherent with each other and with the reference.

> Third — **human preference signal**: pairwise ranking sessions — 'which of these two do you prefer?' — fed into an ELO system, then correlated with generation parameters. This surfaces which sampler settings, CFG values, and style-reference weights actually drive preference, not just technical quality. That's the signal that closes the gap between what the pixel metric measures and what users want.

> The system becomes a quality gate: every batch that goes through Firefly web runs the consistency check automatically. If variance is above threshold, the batch is flagged before it reaches the user. Human preference sessions run async and train the ELO model over time."

**If they ask about north star metric:**
> "Human Preference Rate — what percentage of generations does a user prefer over the alternative they were shown? Below 60% you're at coin flip. Target is 75%+. Feed Cohesion Score is the leading indicator; HPR is the lagging validator."

---

## Thread 2: Competitive Landscape → you've done the work

*This maps to: "analyze the competitive landscape and find opportunities to differentiate."*

**Don't just say you have an opinion — quote the specific gap:**
> "I've built an automated competitive intelligence system that generates acquisition playbooks for Adobe products. I have a Firefly-specific playbook that I can walk through, but the one finding I'd put in front of any Firefly PM is this: Firefly has the strongest enterprise story in AI image generation — IP indemnification that literally no competitor offers — but it ranks for almost none of the high-intent queries users type when they're choosing an AI image tool. Those two facts should not coexist.

> The specific opportunity: Midjourney's active lawsuit exposure has left the 'commercially safe AI image generator' keyword cluster completely unclaimed. Zero competition. Firefly is the factually correct answer to every query in that cluster and doesn't appear. That's a 90-day organic ranking opportunity that costs nothing to pursue — it's a CMS change plus three landing pages.

> On the competitive landscape more broadly: Canva wins on distribution (260M MAU flywheel), Midjourney wins on raw creative quality for professionals, OpenAI wins on API and developer reach. Firefly's white space is enterprise-to-creative pipeline — you're the only product with CC integration, Custom Models that train on a brand's own image library, and indemnification. That's not a feature list, that's a category: 'the only AI image generator your legal team will approve that also works inside your existing creative stack.' No one else can say that sentence."

**If they push on quality gap vs. Midjourney:**
> "That's the right tension to sit in. The evaluation framework above is how you close it — or at least measure where the gap is precisely enough to direct research. You can't fix a quality gap you can't quantify. The prompt adherence score tells you whether the gap is semantic (model misses intent) or aesthetic (model captures intent but style falls short). Those are different research problems with different solutions."

---

## Thread 3: Customer Insights → you build for real segments

*This maps to: "develop deep customer insights" and "interpret needs from creative professionals, creators, and businesses."*

**Three segments, three projects, each with a specific insight:**
> "I've deliberately built products for different points on the creative spectrum because the Adobe customer base is not one segment.

> **Creative professionals** — Stil was built for this. The insight that drove the architecture: the problem for serious creators isn't that they can't find an AI tool. It's visual drift. They make 200+ pieces of content a year and their feed stops looking like them six months in — not because their aesthetic changed, but because 200 micro-decisions made under time pressure, across three tools, slowly diverge from their original intent. They don't notice it happening. They notice it after. The implication for Firefly: consistency-at-volume is the value proposition, not generation quality per se. A professional needs every Firefly output to feel like it came from the same brand intent, not just like it's technically good.

> **Creators/course creators** — StoryForge is the project here. The insight: these users don't have a creativity problem or a generation quality problem. They have a production bottleneck. One good 15-second video takes 3 hours to make. The hook testing deficit means they can't test which angle resonates because producing even one variation is a full production effort. For Firefly video, the question isn't 'is the quality good enough' — it's 'can I produce 10 variations of the same concept in one session?' That's a different product requirement.

> **Business users** — MailIntel and the competitive intelligence work. The insight: business users will pay for AI tools that tell them what to do next, not just for AI tools that generate content. The Firefly enterprise play isn't 'generate images' — it's 'every brief that goes into Campaign Manager can automatically generate on-brand creative that's already cleared for commercial use.' That's the 'establish feedback loops to ensure customer value is realized' problem — the value has to be visible and causal, not just 'Firefly is available in this workflow.'"

---

## Thread 4: Technical depth → model tradeoffs and architecture

*This maps to: "strong technical knowledge to engage with model researchers about architecture, quality metrics, and tradeoffs."*

**Lead with a specific tradeoff decision you made:**
> "The technical pattern I keep applying across projects is: use the cheapest model that can close the gap, and be explicit about where it can't.

> In Stil, every task runs on Haiku — agent loop, style extraction, insights grading, asset tagging. Target is under $0.05 per user per day. Color palette extraction and feed consistency scoring run on Pillow — no model at all. The tradeoff decision: Haiku can't reliably capture aesthetic intent from a conversation transcript, but it can extract the structural signal reliably. So Haiku handles the behavioral ground truth layer; the AI-extracted intent layer is a prompt task Haiku handles well enough. If a user's aesthetic was genuinely complex, you'd use Sonnet there — but for most creators, Haiku's interpretation plus the deterministic choices log gives you 80% of the value at 10% of the cost.

> In Vigil — a network incident investigation agent I built on top of Splunk's MCP server — the tradeoff was between model inference for pattern matching versus RAG-retrieved SPL patterns. The answer was RAG-first: Pinecone retrieves vetted Splunk Processing Language queries at each investigation step before the model generates anything. This cuts hallucinated SPL by eliminating the 'generation from scratch' path for well-known query patterns, while still allowing the model to synthesize novel queries for edge cases. You add model generation only where retrieval fails.

> For Firefly, the relevant version of this question is: where in the generation pipeline does a learned quality metric add value that a heuristic can't? My answer is: prompt adherence is the one place. Aesthetic quality, style consistency, technical artifacts — heuristics and pixel math get you 80% of the way there without model inference cost. Prompt adherence requires understanding semantic intent, which heuristics can't reach."

**If they ask about agentic architectures:**
> "I've built five multi-agent systems across different problem domains — GSentinel (benefits rejection resolution), Vigil (network incident investigation), MailIntel (SMB campaign generation), Brief to Campaign (marketing workflow), and StoryForge (video hook variation). The consistent finding: FSM orchestration beats LLM orchestration for well-defined workflows because you get deterministic state transitions with full audit trail. You use LLM orchestration only where the decision tree is genuinely too complex or variable to specify in advance. Most production workflows are not that complex."

---

## Thread 5: Roadmapping in a research environment

*This maps to: "comfort operating in research environments with evolving success definitions."*

**The key framing:**
> "The way I think about roadmapping for foundational capabilities is different from feature roadmapping. In features, the customer need is relatively stable and the uncertainty is execution. In foundations, the capability space is evolving faster than the product surface — which means your roadmap needs to specify the quality threshold and the evaluation method before it specifies the feature.

> The pattern I've used: define the measurement system first, then set the threshold that constitutes 'production-ready,' then work backwards to what the model or architecture needs to deliver that threshold. That sequence keeps the roadmap anchored even as the research direction evolves, because the quality benchmark is stable even when the technical path to it changes.

> For Firefly specifically: the roadmap question for video generation isn't 'when can we ship text-to-video.' It's 'what does good look like at each quality dimension — temporal consistency, motion naturalness, prompt fidelity — and what's the minimum viable threshold for each segment?' A professional creative and an SMB marketer have different thresholds. The professional needs temporal consistency at near-broadcast quality; the SMB marketer needs a 15-second social clip that doesn't have obvious artifacts. Those are different bars, and they imply different release gates."

---

## Close — why this role specifically

> "The reason I'm interested in this specific role is that foundational capabilities is where the leverage is. Firefly features that ship in Photoshop, Express, and Campaign Manager all run on the same generation pipeline. If I improve the style consistency evaluation framework at the foundation layer, every surface inherits it. That's the kind of work that compounds. A PM working on a single surface can build a great feature. A PM working on foundations can change what's possible for every team that builds on top of it — and that's the problem scope I want to be working on."

---

## Quick-fire answers (if they ask directly)

**"What's your biggest limitation in this role?"**
> "I haven't worked inside a model research organization before, so I'll have a learning curve on the internal research workflow — how prioritization happens between exploration and production alignment, how feedback loops from product requirements reach research teams. I'm aware of that gap and I'd close it fast. What I bring is a proven ability to define quality metrics that researchers can actually optimize against, which I think is the harder half of that interface."

**"What do you know about Firefly's current quality gaps?"**
> "Based on my competitive research: the perceived gap vs. Midjourney is most acute in creative style fidelity for professional use cases — outputs described as 'too clean' or 'generic.' The training data constraint (commercially licensed, no scraped web) limits exposure to the long tail of aesthetic styles that pros care about. Custom Models partially address this for enterprise customers. The evaluation gap is that there's currently no published benchmark that lets users or Adobe itself measure this gap precisely against Midjourney — which means it's hard to target the improvement."

**"Walk me through a quality tradeoff decision."**
> Use the Haiku vs. Sonnet Stil example from Thread 4. Or: "In StoryForge, the north star metric is Posted Video Rate — did the creator actually post the output, not did it generate successfully. That's a quality definition that no technical metric captures, which means you need a behavioral measurement. We targeted 60% PVR as the threshold above which the tool is genuinely useful. Below that, generation quality improvements are irrelevant."

---

## Numbers to know cold

| Fact | Number |
|---|---|
| Midjourney lawsuit exposure | Active lawsuits (Getty vs. Stability, artists class action) |
| Firefly competitive edge | Only enterprise IP indemnification + CC integration combo |
| Canva MAU | 260M |
| Stil cost target | < $0.05 / user / day |
| GSentinel auto-fix rate | 67% |
| Vigil MTTR improvement | 47 min → 35 seconds |
| StoryForge PVR target | ≥ 60% |
| TTFV target | < 5 min at 95th percentile |
| Hook Performance Variance | ≥ 3× top vs. bottom hook |
| CI system cost | $0.003 per report (Haiku pipeline) |
| Feed Cohesion Score dimensions | Color temp, brightness, contrast, saturation |
