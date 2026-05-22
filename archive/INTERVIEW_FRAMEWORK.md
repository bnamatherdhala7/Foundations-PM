# Adobe GenAI PM — Interview Framework & Deep-Dive Notes

*Structured for delivery. Every section uses First / Second / Third framing.
Each technical decision ties back to user impact and a business metric.*

---

## The role in one sentence

Build the foundational generative AI capabilities — image and video — that every Adobe creative surface depends on, by balancing quality, latency, cost, privacy, and scalability while aligning PM, engineering, research, design, and infrastructure teams.

---

## Part 1: Core PM Framework for Any Problem Question

When asked to evaluate, improve, or build a Firefly capability, always structure the answer across five phases. Use this as your backbone — fill in the specific details per question.

### Phase 1 — Discovery & Stakeholder Alignment

**Goal:** Understand user pain, workflow bottlenecks, quality expectations, and business constraints before any technical decision.

**Stakeholders to name:**
- PM teams (surface owners: Photoshop, Express, GenStudio)
- International teams (localization, regional quality variance)
- Design (creative quality bar)
- GTM and Marketing (positioning, messaging, launch readiness)
- Engineering (infrastructure constraints, latency budgets)
- Data science / Research (model capabilities and timelines)

**Discovery questions I'd ask first:**
- Where does workflow friction occur — which steps require the most manual editing after generation?
- Which generation types fail most often, and what does the failure look like?
- What quality gaps matter most to professional users vs. business users vs. hobbyists?
- What is the current infrastructure cost per generation, and where is it growing fastest?
- What latency threshold causes users to abandon the workflow?

**⚡ Improvement note:** Don't just list stakeholders — name the *tension* between them. Engineering will push for latency; research will push for quality; GTM will push for shipping dates. Your job is to define the threshold at which each stakeholder's bar is met, not to find consensus.

### Phase 2 — Define Success Metrics First

Define the measurement system before proposing a solution. This is what separates a strategic PM from a feature PM.

**Image quality metrics (and their limits):**

| Metric | What it measures | Its limit — say this to show depth |
|---|---|---|
| FID | Statistical distance between real and generated image distributions | Misses artistic quality entirely; a model generating statistically average images scores well while failing professionals |
| IS (Inception Score) | Object clarity + output diversity | Classifier-dependent; fails to detect poor diversity *within* a class |
| CLIP Score | Semantic alignment between text prompt and image | Misses aesthetic nuance — a "dramatic lighting" prompt can score well while the output feels flat |
| Human Preference Rate | % of generations a user prefers over an alternative | The only metric that can't be gamed; target ≥ 75% |

**Video quality metrics:**

| Metric | What it measures | Its limit |
|---|---|---|
| VBench | 16-dimension suite: motion smoothness, temporal flickering, subject consistency, spatial relationships | Resource-intensive; not a regression metric for every build |
| FVD | Distribution similarity between real and generated video clips | Inherits FID's blind spots |
| Temporal consistency / flicker | Pixel displacement between frames | Maps directly to the user complaint "this video feels choppy" — most actionable metric |

**Multimodal reliability tests (run these in production before any eval):**
- **Blank Drop test:** Remove the image input. If model performance doesn't degrade, it wasn't reading the image — it was pattern-matching on text. Catches visual grounding failures before they reach users.
- **Image Sensitivity test:** Swap the image while keeping the prompt identical. If the answer doesn't change appropriately, visual grounding is unreliable. LMMS-Eval provides a standardized, reproducible pipeline for both.

**Product metrics — always add these, they're what leadership cares about:**

| Metric | Definition | Why it matters |
|---|---|---|
| Task Completion Rate | % of users who successfully finish the generative workflow | Measures whether the capability is actually usable, not just technically impressive |
| Time Saved | Reduction in editing time post-generation | Direct proxy for creative throughput |
| Repeat Usage Rate | % of users who return to the workflow within 7 days | The quality signal that no benchmark captures — users vote with behavior |
| Adoption Rate | % of eligible users who actively switch to the new workflow | GTM and retention signal |
| Infrastructure Cost per Generation | Total inference cost per completed generation | Determines pricing model and margin at scale |

**Business metrics — elevate from technical PM to strategic PM:**
- Retention lift for users who adopt the AI workflow vs. those who don't
- Conversion from free tier to paid tier when AI capability is the trigger
- Infrastructure savings from model efficiency improvements (e.g., distillation reduces inference cost X%)
- Creative throughput: assets produced per user per week, tracked over time

**⚡ Improvement note:** Most candidates stop at quality metrics. Always add the business metrics table. It signals you're thinking about the P&L, not just the model.

### Phase 3 — Benchmarking Strategy (A/B testing structure)

**The right framing:** Run controlled comparisons. Never make architecture decisions on opinion.

**A/B test design for a Firefly capability evaluation:**
- **Control:** Existing workflow (current Firefly model or proprietary API — Gemini/OpenAI)
- **Treatment:** Proposed architecture (Firefly fine-tuned, open-weight, or hybrid)
- **Dimensions to measure in parallel:** quality (CLIP Score, FID, human preference), cost (inference $/generation), latency (p50 and p95 generation time), adoption (task completion rate, repeat usage)

**Why this approach wins in the room:**
- Grounds the decision in data, not advocacy
- Reduces organizational risk — you're not betting on theory
- Gives research teams a clear benchmark to optimize toward
- Gives leadership a defensible ship/no-ship gate

### Phase 4 — Model Architecture Decision

**The three options and when to use each:**

**Option A — Proprietary APIs (Gemini, OpenAI, Claude)**
- Use when: fast iteration is the priority, infrastructure expertise is limited, baseline performance is sufficient
- Advantage: minimal setup, strong reasoning, fast API access
- Disadvantage: no weight access, limited customization, vendor dependency, external data routing (privacy risk), long-term inference cost

**Option B — Open-weight models (Stable Diffusion, LLaMA, Mistral)**
- Use when: fine-tuning flexibility is required, data privacy is a hard constraint, long-term inference cost matters, on-prem deployment is needed
- Advantage: full weight access, self-hosting, fine-tuning, privacy, lower long-term cost
- Disadvantage: infrastructure investment required, quality may initially lag proprietary, need evaluation pipelines

**Option C — Hybrid architecture (recommended for Firefly)**
- Combine: Firefly foundation model + RAG for domain grounding + LoRA adapters for brand customization + lightweight prompt engineering for workflow alignment
- Why: preserves Adobe's commercially safe training advantage while adding customization, privacy, and cost efficiency at the edges

**⚡ My recommendation framing:**
> "I wouldn't frame this as Firefly vs. open-weight vs. proprietary. I'd frame it as: which layer of the stack benefits from each approach? Foundation model — Firefly, because the training data advantage is the enterprise moat. Brand customization — LoRA adapters on top of Firefly, because full retraining is millions of dollars and months of time. Domain grounding — RAG, because retrieval is cheaper than training and keeps outputs current. Prompt layer — few-shot templates, because for proprietary APIs this is the only customization lever you have."

### Phase 5 — Fine-Tuning Strategy (not retraining from scratch)

**Why not retrain from scratch:**
- Costs millions in GPU compute
- Requires months of training time
- Needs massive, curated datasets
- Loses the pretrained intelligence you paid for

**The efficient path:**

**LoRA (Low-Rank Adaptation):**
- Freeze the original model weights
- Add lightweight trainable adapter layers (small rank matrices)
- Train only the adapters — not the foundation model
- Merge at inference time
- Result: preserved pretrained knowledge, low compute cost, fast experimentation, easy rollback

**QLoRA (Quantized LoRA):**
- Adds quantization on top of LoRA — reduced precision memory usage
- Lower GPU memory requirement, cheaper training, better scaling
- Tradeoff: slightly more engineering complexity
- Use when: memory is the bottleneck or training budget is limited

**In Firefly's context:** Style IDs and Custom Models are exactly this pattern — user-provided reference assets act as low-rank adapters that constrain the generation space to a brand's aesthetic without retraining the foundation model. The product implication: quality of the brand lock depends on quality and diversity of training assets. That's a user education problem as much as a model problem.

**Dataset strategy:**
- Bootstrap with public datasets: Open Images, ImageNet, LAION (image); Kinetics, YouTube-8M (video)
- Differentiate with Adobe's proprietary asset: Creative workflow behaviors, professional editing patterns, Stock image quality bar
- The PM insight: public datasets give you general capability. Adobe's proprietary data is where the differentiation comes from — it encodes professional creative intent, not just pixel patterns.

### Phase 6 — Production Rollout Plan

| Phase | Goal | Actions |
|---|---|---|
| **Phase 1 — Discovery** | Define the problem space | Gather workflow pain points, define success metrics, align stakeholders on the measurement framework |
| **Phase 2 — Pilot** | Validate with real users | Small A/B experiment, compare quality + latency + cost, run Blank Drop and Image Sensitivity tests |
| **Phase 3 — Optimization** | Close the gap | Fine-tune with LoRA, improve prompting, add RAG for domain grounding, reduce inference cost with distillation |
| **Phase 4 — Scale** | Expand to production | Route by segment (Express → distillation model, CC Pro → foundation model), monitor adoption and infrastructure cost |
| **Phase 5 — Continuous iteration** | Prevent regression | Human preference feedback loops, VBench regression testing on each model update, quality regression monitoring before every release |

### Phase 7 — Cost Optimization

Every architecture decision has a cost dimension. Know these variables:

| Cost Driver | What to optimize |
|---|---|
| GPU inference cost | Route lighter segments to distillation models; reserve foundation model for Pro tier |
| Latency | Flow matching reduces inference steps → lower latency at same quality |
| Memory | QLoRA reduces GPU memory usage for fine-tuning |
| API pricing | Proprietary API cost grows linearly with scale; open-weight amortizes over infrastructure investment |
| Fine-tuning cost | LoRA/QLoRA vs. full fine-tune is 10–100× cheaper depending on model size |
| Hosting | On-prem open-weight for privacy-sensitive enterprise; cloud API for consumer tier |

**The PM framing for cost:**
> "I think about cost in three time horizons. Today: what does the current inference cost per generation allow us to offer at what price point? In 6 months: does distillation or quantization change that math enough to expand to a new segment? In 18 months: does the open-weight trajectory close the quality gap with proprietary models at a cost advantage that changes the build/buy decision?"

---

## Part 2: My Projects Mapped to the JD

*Use these when asked "tell me about a project" — each one answers a specific JD requirement.*

### Evaluation frameworks → Stil (Feed Cohesion Score)

> "I built a working image quality measurement system called the Feed Cohesion Score — 0–100 visual consistency across a creator's feed using deterministic pixel math. The design principle: you don't need a learned quality model to measure consistency. Pixel math is faster, cheaper, more interpretable. You add a model only where it can't reach. For Firefly, this maps to a three-layer eval stack: CLIP Score for prompt adherence (knows its limit — misses aesthetic nuance), pixel math for style consistency across a batch, and ELO-based human preference as the final gate. Automated scores tell you direction. Human signal tells you arrival."

### Quality / cost / latency tradeoffs → Stil architecture

> "In Stil every task runs on Haiku — under $0.05 per user per day. Feed consistency scoring runs on Pillow: no model, no inference cost. I add Sonnet only where Haiku genuinely fails. The pattern: use the cheapest model that closes the gap, be explicit about where it can't. For Firefly: route Express users to a distillation model, reserve foundation model for Creative Cloud Pro tier where the quality threshold and willingness-to-pay are both higher."

### Agentic architecture in production → GSentinel, Vigil, MailIntel

> "I've built five multi-agent systems. The consistent finding: FSM orchestration beats LLM orchestration for well-defined creative workflows. Deterministic state transitions give you full audit trail and predictable behavior. In Vigil — a network incident agent on Splunk's MCP server — RAG-first retrieval cut hallucinated queries by eliminating the generation-from-scratch path for known patterns. Same principle for Firefly: heuristics and retrieval carry the load until they genuinely can't, then you pay for inference."

### Competitive differentiation → CI system (Firefly Acquisition Playbook)

> "I built an automated competitive intelligence pipeline that generates acquisition playbooks for Adobe products at $0.003 per report. The Firefly finding: Midjourney's active lawsuit exposure has left the 'commercially safe AI image generator' keyword cluster completely unclaimed. Firefly is the factually correct answer to every query in that cluster and doesn't appear. That's a 90-day organic acquisition win — a CMS change and three landing pages. The enterprise moat: IP indemnification + CC integration + Custom Models. No competitor has all three."

### Customer insights across three segments → Stil + StoryForge + MailIntel

> "I've built for all three Adobe customer segments. Creative professionals (Stil): their problem is visual drift, not tool quality — 200 micro-decisions under time pressure across three tools slowly diverge from original creative intent. Express creators (StoryForge): production bottleneck, not creativity — one 15-second video takes 3 hours; the metric that matters is Posted Video Rate (did they actually post it?), not generation quality. Business users (MailIntel): they pay for AI that tells them what to do next, not just AI that generates — the feedback loop from output to performance signal to model improvement has to be visible and causal."

---

## Part 3: Polished Closing Statement

*Use this when asked "walk me through your overall approach" or at the close of any major question.*

> "My approach starts with deeply understanding workflow pain points and defining measurable success metrics before any technical decision — quality, task completion, latency, repeat usage, and infrastructure cost. Then I'd run controlled A/B evaluations comparing existing proprietary workflows against Firefly or open-weight alternatives across those dimensions simultaneously.

> Rather than retraining models from scratch — which costs millions and takes months — I'd prioritize LoRA or QLoRA adapters to reduce infrastructure cost and accelerate experimentation while preserving the pretrained intelligence. I'd pair that with a hybrid architecture: Firefly foundation for commercially safe generation, LoRA for brand customization, RAG for domain grounding, and prompt engineering at the proprietary API layer where weight access isn't available.

> On the evaluation side, I'd build a three-layer quality stack: automated metrics like CLIP Score and FID for tracking model progress over time, pixel math for style consistency and consistency regression testing, and human preference ELO sessions as the final gate before any capability moves to production. Automated scores tell you direction; human signal tells you arrival.

> Throughout, I'd partner closely with research, engineering, design, and GTM teams — using the benchmark I define as the shared contract between research's exploration and product's production requirements. And I'd measure success not just in model quality scores but in the business outcomes: retention lift, creative throughput, infrastructure savings, and conversion from free to paid driven by the AI workflow."

---

## Part 4: Areas for Improvement (and how to fix them)

### 1. Tie every technical decision to user impact

**Weak:** "I'd use flow matching because it reduces inference steps."
**Strong:** "Flow matching reduces inference steps, which brings Generative Fill latency under 2 seconds — the threshold at which professionals stop waiting and trust the tool. Below that threshold, workflow adoption increases measurably."

**Fix:** After every technical point, add: "Which means for the creator…" or "The product impact is…"

### 2. Add the business metric layer

**Weak:** "We'd measure quality with FID and CLIP Score."
**Strong:** "We'd use FID and CLIP Score to track model progress over time, but the business metric that matters to leadership is retention lift for users who adopt the AI workflow — historically, workflow-integrated AI features drive 15–20% better retention than standalone generators. That's the metric I'd track in the pilot."

**Fix:** Always add one business metric alongside every quality metric. Retention, conversion, throughput, infrastructure savings.

### 3. Use numbered structure — always

**Weak:** "There are several things I'd look at here — the quality, the cost, and also the latency matters, and we'd want to think about privacy too..."
**Strong:** "I'd evaluate four dimensions. First, quality — measured with CLIP Score for prompt adherence and human preference for the creative bar. Second, cost — inference cost per generation and projected cost at production scale. Third, latency — p95 generation time against the 2-second professional threshold. Fourth, privacy — whether the architecture can meet enterprise data residency requirements."

**Fix:** Any time you list more than two things, number them explicitly.

### 4. Name the tension, not just the stakeholders

**Weak:** "I'd align with PM, engineering, research, and GTM teams."
**Strong:** "The tension I'd need to resolve upfront: research teams will push for quality benchmarks that require more compute, which conflicts with engineering's latency budget. I'd resolve it by defining a segment-specific quality threshold — the quality bar for Express is different from the bar for Photoshop Pro — which lets both teams have a target that's achievable within their constraints."

**Fix:** When you name stakeholders, always name what they want and where the conflict is.

### 5. Anchor open-weight vs. proprietary to a specific decision

**Weak:** "Open-weight models offer more flexibility."
**Strong:** "The specific decision where I'd choose open-weight over proprietary API: any enterprise customer with data residency requirements. Proprietary APIs route data through external cloud infrastructure. For enterprise creative teams working on unreleased campaigns or IP-sensitive assets, that's a blocker. Self-hosted open-weight with LoRA adapters on their own infrastructure solves the residency problem while maintaining the customization Adobe's Custom Models promise."

---

## Quick Reference: Numbers to Know Cold

| Fact | Value |
|---|---|
| Firefly enterprise moat | IP indemnification + CC integration + Custom Models — no competitor has all three |
| Professional latency threshold | Sub-2-second (Generative Fill) — users abandon above this |
| Human Preference Rate target | ≥ 75% (below 60% = coin flip) |
| VBench dimensions | 16 (motion smoothness, temporal flickering, subject consistency, spatial relationships, +12) |
| Flow matching benefit | Fewer inference steps → lower latency at same quality level |
| LoRA mechanism | Freeze base model, train lightweight adapters; merge at inference |
| QLoRA benefit over LoRA | Quantization reduces GPU memory usage; useful when training budget is limited |
| Blank Drop test | Remove image input — if answer doesn't degrade, model wasn't reading the image |
| Image Sensitivity test | Swap image keeping prompt same — answer must change appropriately |
| CLIP Score limit | Misses aesthetic nuance; statistical alignment, not creative intent |
| FID limit | Misses artistic quality; measures distribution, not individual image value |
| IS limit | Classifier-dependent; fails to detect within-class diversity problems |
| Canva MAU | 260M |
| Stil cost target | < $0.05 / user / day |
| StoryForge north star | Posted Video Rate ≥ 60% — did they actually post it? |
| StoryForge TTFV | < 5 min at 95th percentile |
| GSentinel auto-fix rate | 67% |
| Vigil MTTR | 47 min → 35 sec |
| CI pipeline cost | $0.003 / Firefly competitive report |
| Feed Cohesion Score inputs | Color temp, brightness, contrast, saturation variance |
