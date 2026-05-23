# Cisco / Splunk AI Foundations — Full Interview Prep

**Role:** Senior Staff PM, Splunk AI Foundations
**Rounds:** Engineering Leader (Vedant Dharnidharka) · Engineering Peer (Liang Gou / Shang Cai / Rehan Mulla) · PM Peer (Nick Ma / Mohit Verma / Jon Elli)

---

## Before You Read Anything Else — Know These Cold

| Fact | Value |
|---|---|
| **Vigil** | Network incident agent on Splunk's MCP server. MTTR 47 min → 35 sec. RAG-first retrieval eliminated hallucinated SPL queries. |
| **Stil** | Feed Cohesion Score: 0–100 visual consistency via pixel math. Cost < $0.05/user/day. Two-layer memory: choices_log (deterministic) + style_signature (AI-extracted). |
| **StoryForge** | 4-agent hook variation engine. North star: Posted Video Rate ≥ 60%. TTFV < 5 min p95. Hook variance ≥ 3× between best and worst. |
| **GSentinel** | Automated fix rate: 67%. |
| **Cisco TSM** | Zero-shot univariate forecasting. Multiresolution input architecture. 300B+ data points, >50% observability. Built on TimesFM (decoder-only). Liang Gou is first author. Vedant Dharnidharka is co-author. |
| **FSM vs LLM orchestration** | FSM (finite state machine) beats LLM orchestration for well-defined operational workflows: deterministic, auditable, predictable failure modes. |
| **Your eval philosophy** | Automated scores tell you direction. Human signal tells you arrival. Define measurement system before proposing solutions. |
| **The governance framing** | Trust is the product for enterprise agentic workflows — not a feature. Explainability, auditability, and human override are table stakes, not differentiators. |

---

---

# ROUND 1 — Engineering Leader
## Vedant Dharnidharka · 45 Minutes

**What Vedant is testing:** Can this PM lead a complex AI platform roadmap without becoming a bottleneck to engineering? Do they understand model development deeply enough to make real trade-offs — not just translate requirements? Will they fight for the right scope or just accommodate whoever asked last?

**Critical context:** Vedant is a co-author of the Cisco Time Series Model paper. He built ML infrastructure. He will probe technical depth hard and has no patience for PM-speak that papers over ignorance. He respects people who know what they don't know and are precise about it.

**Your opening (first 90 seconds):**
> "I want to use our time on substance — specifically how I think about building foundational AI platforms. I've shipped five multi-agent systems and built an evaluation framework for a production image quality system. The through-line in everything I've built is: define the measurement system before proposing a solution, because the measurement system is often more valuable than the feature. I've read the Time Series Model paper — I'd love to get into the productization questions with you."

---

### Q1: "Walk me through a significant technical trade-off you made as a PM. What was it, how did you decide, and what happened?"

**What he's testing:** Do you understand the trade-off space well enough to have had an opinion, or did you just rubber-stamp engineering's call?

**Structure:** Situation → The two options and their real costs → How you decided → The outcome → What you'd change.

**The story — Vigil:**
> "In Vigil, the incident response agent I built on Splunk's MCP server, the core trade-off was orchestration architecture: LLM-based orchestration versus FSM. LLM orchestration gives you flexibility — the agent can route dynamically across tools based on context. FSM is deterministic — you define states and transitions explicitly, and the agent always behaves predictably.
>
> The case for LLM orchestration was real: incident patterns are diverse, and a rigid FSM might miss novel failure modes. The case for FSM was also real: this was a production system with 47-minute average MTTR. An orchestration decision that goes wrong at 3am isn't just a product bug — it's a missed SLA.
>
> I decided on FSM for three reasons. First, auditability: every state transition is logged deterministically, which means post-incident review can reconstruct exactly what the agent did and why — you can't get that from an LLM decision trace. Second, failure modes: FSM fails predictably at known transitions, LLM fails unpredictably at novel inputs. For a system touching production infrastructure, predictable failure is safer than flexible failure. Third, the actual workflow: incident investigation in Splunk follows a well-defined pattern — ingest alert, retrieve context, correlate events, propose remediation, await approval. That's not a dynamic routing problem, it's a state machine.
>
> The result: MTTR dropped from 47 minutes to 35 seconds, and operators trusted it because they could read the trace. The thing I'd change: I'd build the FSM with a fallback LLM router for genuinely novel patterns — the 5% of incidents that don't fit the state machine — rather than treating FSM as the only answer."

**Follow-up 1:** *"How did you get engineering alignment on the FSM decision when LLM orchestration is more 'exciting'?"*
> "I framed it as a risk-tiered decision, not a technical preference. For consumer products, LLM orchestration's flexibility is the right call — the cost of a wrong route is a bad user experience. For production operations tooling, the cost of a wrong route is an incident. Once I put it that way, the engineering team agreed — they didn't want to own a system that could fail unpredictably in production either."

**Follow-up 2:** *"What if the engineering team had pushed back and said LLM orchestration was better?"*
> "I'd have asked for a specific failure scenario where FSM couldn't handle it, and then designed a test against that scenario. If they could show me a real incident pattern that an FSM would miss, that changes the decision. But the argument 'LLM is more flexible' without a concrete failure mode isn't a technical argument — it's a preference."

**What NOT to say:** Don't say "I deferred to engineering on the architecture." That signals you didn't have a real opinion.

---

### Q2: "Tell me about a time you disagreed with an engineering team's technical approach. What happened?"

**What he's testing:** Can you hold a position under pressure without becoming adversarial? Do you know when to change your mind versus when to push harder?

**The story — Stil eval framework:**
> "In Stil, I wanted to build a learned quality model for the Feed Cohesion Score — a neural approach to measuring visual consistency. The engineering lead pushed back: he wanted deterministic pixel math only. His argument was that a learned model would drift, be harder to debug, and add inference cost.
>
> My initial instinct was that pixel math couldn't capture aesthetic quality — that you needed semantic understanding to know if two images 'feel' coherent. But instead of going to standoff, I ran a test. We took 200 creator feeds, ran pixel math on them, and had 15 humans rate visual consistency. The correlation between pixel math and human ratings was 0.84 — strong enough that I conceded the point for the consistency sub-score.
>
> But I held my position on one dimension: semantic intent. Color temperature variance tells you if photos are tonally consistent, but it doesn't tell you if they're thematically coherent — a feed of portraits and a feed of landscapes can have identical pixel stats and feel completely different. I proposed a hybrid: pixel math for the four measurable dimensions, a single Haiku call to extract semantic theme, combined into the final score. Engineering accepted that because I was specific about which dimension required the model and why pixel math couldn't reach it.
>
> The outcome: Feed Cohesion Score runs at sub-cent cost with zero inference cost for 80% of the computation. If I'd pushed for a fully learned approach, we'd have a more expensive system with worse interpretability and no meaningful quality improvement."

**Follow-up 1:** *"How did you know when to concede and when to hold?"*
> "Data. When I had a testable hypothesis — 'pixel math doesn't capture X' — I ran the test. When the test showed pixel math worked, I conceded. When the test showed a genuine gap, I held. The principle is: never hold a position based on intuition when you can run a test that resolves it."

**Follow-up 2:** *"What if you couldn't run a test — what if it was a purely architectural decision with no quick empirical path?"*
> "Then I'd ask for the concrete failure scenario the engineering approach can't handle. If they can't name one, the architecture decision is a preference. If they can name one, that becomes the design constraint. Abstract disagreements about architecture almost always resolve when you get specific about failure modes."

---

### Q3: "How do you decide what NOT to build? Tell me about a time you cut scope. What was the reasoning?"

**What he's testing:** Do you protect engineering capacity, or do you just accumulate roadmap commitments?

**The story — Vigil scope cut:**
> "In Vigil, the original scope included auto-remediation — the agent would not just investigate and propose, but execute fixes autonomously. That was in the design document. Three weeks before the first milestone, I cut it.
>
> The reasoning was a single conversation with two SOC analysts who were the target users. I asked: 'Would you use an agent that auto-executes remediation without approval?' Both said no, and both gave the same reason: one bad auto-remediation action in production and they lose their job. The value proposition of 'faster' wasn't worth the risk of 'wrong.'
>
> The cut wasn't about engineering capacity — the team could have shipped auto-remediation on time. It was about what would actually get adopted. An agent that auto-executes with 95% accuracy and gets blocked by every security team in the enterprise is a worse product than an agent that proposes with 95% accuracy and gets adopted everywhere.
>
> I reframed the scope as 'investigation and proposal' rather than 'investigation and execution.' That's a significant scope cut, but it's the scope that ships into production and gets used. Auto-remediation becomes Tier 2 — after you've built trust through proposal accuracy."

**Follow-up 1:** *"How did engineering react to cutting scope they'd already planned for?"*
> "I brought the user research to the team directly — not just the conclusion, but the quotes. Engineers don't like building things that don't get used. When they heard 'auto-remediation would block adoption in every enterprise security team,' the scope cut made sense to them. What engineers resist is arbitrary scope reduction. What they accept is scope reduction with a clear user reason."

**Follow-up 2:** *"How do you handle a situation where a stakeholder pushes back on the scope cut?"*
> "I'd ask them to define what success looks like for the auto-remediation feature. If success is 'number of autonomous actions taken,' that's an output metric — it can look good while adoption collapses. If success is 'incident resolution rate for customers who have it enabled,' that's an outcome metric — and then the question becomes 'what adoption rate do we need for this to be net positive?' Usually that conversation resolves the disagreement."

---

### Q4: "How do you approach roadmap prioritization when engineering capacity is limited?"

**What he's testing:** Do you have a real framework, or do you just say "impact vs. effort"?

**Answer:**
> "I use a three-layer prioritization model.
>
> First layer: unblock or unlock? Some work is a blocker — if it doesn't ship, nothing downstream can ship. That goes first regardless of feature value. For the AI Foundations platform, dwell + license event tracking is a blocker — without it, semantic search retraining has no training signal. You ship the measurement infrastructure first, even though it has zero user-visible value.
>
> Second layer: what's the critical path to the north star metric? For the Cisco TSM, the north star I'd target is Autonomous Resolution Rate — what percentage of incidents resolve without human action beyond approval gates. Every roadmap item either accelerates that metric or it doesn't. If it doesn't, it needs a very clear argument for why it's worth capacity.
>
> Third layer: what's reversible? When I'm uncertain between two roadmap items of similar impact, I prefer the one that's easier to undo. Shipping a bad MCP connector is a one-sprint rollback. Shipping a bad model architecture decision is a six-month problem. I weight reversibility heavily when impact is similar.
>
> The thing I explicitly don't do: score everything on a 2x2 and treat the output as a decision. That's documentation for decisions that were already made, not a real prioritization tool."

**Follow-up:** *"Tell me about a time the prioritization didn't work out the way you expected."*
> "In StoryForge, I prioritized variation generation over hook analytics, because I thought 'more output' was the bottleneck. It wasn't — the bottleneck was that creators didn't know which variation to post. I'd optimized for generation throughput when the real constraint was decision confidence. I added hook performance analytics to the next sprint, but I lost a cycle. The lesson: always validate the assumed bottleneck with a user before optimizing for it."

---

### Q5: "Tell me about a time you had to drive execution alignment between PM and engineering when things were going sideways."

**The story — GSentinel:**
> "In GSentinel, three weeks before the target launch, the auto-fix rate was at 41% — well below the 65% threshold we'd agreed was the minimum for production. Engineering's position was that the remaining gap was edge cases that would take another two sprints.
>
> The problem was twofold: two customers were waiting, and the engineering estimate had already slipped once. I called a realignment meeting — not a status update, a decision meeting — with three items on the agenda: the actual gap, the options, and a binding decision before we left the room.
>
> Option A: slip the launch two sprints, reach 65%. Option B: launch with 41% auto-fix, but with a human-in-the-loop approval gate on all automated actions — essentially turning the 41% into a 'fast path for human review' rather than autonomous execution. Option C: scope down to the highest-confidence fix categories only and hit 65% on a narrower scope.
>
> We chose Option C. By narrowing to the four fix categories where accuracy was above 80%, we hit a 67% overall auto-fix rate, launched on schedule, and had a clear roadmap for expanding to the remaining categories. The two customers who'd been waiting got a scoped but reliable system rather than a delayed one.
>
> The alignment technique: I came with options and costs, not a problem statement. Engineers don't want to sit in a meeting about what went wrong — they want to make a decision and get back to building."

**Follow-up:** *"What would you have done if engineering hadn't agreed on any of the options?"*
> "I'd have escalated — but carefully. Escalation without context reads as 'PM is reporting engineering is behind.' Escalation with framing reads as 'we have a decision that needs a tiebreaker above our level.' I'd have gone to the engineering manager with the options document, noted that we'd reached an impasse, and asked them to arbitrate — not to override engineering, but to break the tie."

---

### Q6: "What do you know about the Cisco Time Series Model, and how would you approach productizing it?"

**What he's testing:** Did you do your homework? Can you translate a research paper into a product roadmap?

**Answer:**
> "I read the paper. The core innovation is multiresolution input — the model processes time series at multiple resolutions simultaneously, which specifically helps with extended context inputs. That's critical for observability use cases because production telemetry often has meaningful patterns at multiple time scales: a 5-minute CPU spike, a 6-hour capacity trend, and a 30-day seasonality all matter to an SRE, and you can't capture all three with single-resolution input.
>
> It's built on TimesFM, decoder-only, zero-shot, trained on 300 billion data points with over half from observability domains. Zero-shot means customers don't need to fine-tune per environment — that's a major deployment friction reduction compared to supervised alternatives.
>
> For productization, I'd approach it in three phases.
>
> Phase 1: Evaluation framework. Before we market this to customers, we need a customer-side benchmark kit — not just GIFT-Eval scores, but a structured way for a customer to run the model against their own telemetry and compare it to their current alerting baseline. Without that, the sales conversation is 'trust us.' With it, the conversation is 'here's your data, here's what we see, here's what your current system sees.'
>
> Phase 2: API design. The model needs a REST API with sensible defaults. The questions I'd drive with engineering: what's the input contract — how much history is needed for meaningful inference? What's the output contract — point forecast, confidence intervals, anomaly flag, or all three? What's the latency SLA? The deployment model matters too: customer-hosted for air-gapped enterprises, cloud-hosted for SMB, with clear data residency documentation for each.
>
> Phase 3: Integration into existing workflows. The model's value compounds when it feeds the Incident Investigation Agent — anomaly detection from TSM triggers investigation, investigation uses MCP to pull context from ES and XDR, agent proposes remediation. A standalone time series API is useful. A time series API that's the detection layer for a full agentic investigation workflow is the product story that sells."

**Follow-up:** *"What would you set as the success metric for the TSM as a product?"*
> "Two metrics. First, detection accuracy vs. existing alerting: what percentage of incidents does TSM catch that the customer's current threshold-based alerting misses? And what's the false positive rate comparison? That's the model quality metric. Second, downstream adoption: what percentage of TSM anomaly detections result in an investigation session? If the model flags 1,000 anomalies and operators only investigate 50, the problem isn't the model — it's the signal-to-noise presentation. Both metrics matter."

---

### Q7: "How do you think about build vs. buy for AI capabilities?"

**Answer:**
> "Three variables: differentiation, data advantage, and time horizon.
>
> Build when the capability is core to differentiation and you have proprietary data to make the built version better than anything you can buy. For the Cisco TSM, that's the correct call — observability time series data at Splunk scale is not available to any third-party model provider, and anomaly detection on that data is the core product motion. You can't buy your way to a better model than your own data enables.
>
> Buy — or use open-weight — when the capability is table stakes and the vendor has already solved the hard problem. RAG retrieval infrastructure, vector stores, embedding models for general text — these are not where Splunk's differentiation lives. Use open-weight or established vendors, don't rebuild them.
>
> The third case is the hardest: capability that's not yet differentiated but will be. For MCP standardization, the protocol itself is open — but the skill registry, the connector library, and the certified partner ecosystem are where the moat builds. You adopt the open protocol and build the value layer on top.
>
> The mistake I've seen is building what should be bought because it feels more strategic, and buying what should be built because it feels faster. The question isn't 'build or buy' — it's 'where does our proprietary data or domain advantage make a built version categorically better than anything available?'"

---

### Q8: "How do you handle technical debt versus new feature development?"

**Answer:**
> "I treat technical debt as a risk — specifically, the probability that it causes a customer-facing incident multiplied by the expected severity. That framing turns a vague 'we should clean this up' conversation into a concrete 'this is a tier-2 risk to SLA and should be scheduled in Q3.'
>
> In Vigil, there was a retrieval layer that worked but had no fallback when the vector store was unavailable. For a network incident agent, that's not just technical debt — it's a single point of failure in a system that's supposed to reduce MTTR. I escalated that to P1 and got it onto the sprint. Not because it was visible to users, but because its failure mode was catastrophic and the fix was bounded.
>
> The framework: technical debt that has a bounded, low-risk failure mode gets scheduled quarterly as a hygiene item. Technical debt that has an unbounded or high-risk failure mode gets treated as a bug and prioritized accordingly. The mistake is treating all technical debt as hygiene and none of it as risk."

---

### Q9: "How do you ensure that a PM doesn't become the bottleneck for engineering?"

**Answer:**
> "Three practices.
>
> First, decision documentation. I write a short document for every non-obvious decision: what we decided, why, what alternatives we rejected, and what would cause us to revisit. Engineering can move without me once they have the decision context. The bottleneck is usually not the PM's presence — it's the PM's reasoning not being accessible when the PM isn't in the room.
>
> Second, threshold delegation. For any decision below a defined complexity threshold — scope adjustments under half a sprint, technical approach changes that don't affect the user contract, implementation details — engineering doesn't need PM sign-off. I define the threshold explicitly at the start of each cycle.
>
> Third, async-first. If a question can be answered in writing, I don't schedule a meeting. Most 'quick syncs' are actually decision documents with a meeting attached. I remove the meeting and keep the document."

---

### Q10: "What questions do you have for me?"

**These show you read the paper and understand the technical landscape:**

1. *"The multiresolution architecture solves extended context inputs — I'm curious how you're thinking about the customer-side evaluation story. GIFT-Eval is a good benchmark for general forecasting, but an observability practitioner will want to see performance against their specific telemetry. Is a customer benchmark kit part of the v1 productization plan, or is that a v2 problem?"*

2. *"For the zero-shot capability — what's the hardest customer telemetry pattern you've seen the model struggle with? Understanding the known failure modes would shape how I'd design the evaluation framework and the documentation."*

3. *"How are you thinking about the on-prem versus cloud deployment split for the TSM API? Enterprise security teams with air-gapped environments are a large segment of the Splunk customer base, and the deployment model will significantly affect the GTM motion."*

---

---

# ROUND 2 — Engineering Peer
## Liang Gou / Shang Cai / Rehan Mulla · 45 Minutes

**What the engineering peer is testing:** Will this PM make my job harder or easier? Do they understand what's technically hard versus what's just unfamiliar to them? Will they protect the team from bad scope decisions, or will they bring in every stakeholder request and call it the roadmap?

**If it's Liang Gou specifically:** He built the Time Series Model. He wants to know if you understand what it does, what its limits are, and whether you can translate customer needs into model requirements without requiring a model to be something it isn't.

**Your opening:**
> "I want to be direct about how I think PM-engineering partnership works. My job is to give you the clearest possible picture of what the user needs and what success looks like, so you can make good technical decisions. It's not to approve your architecture or veto your implementation. Where I add value is at the boundary: translating user constraints into requirements, and translating technical constraints into product trade-offs. Tell me where that's worked well and where it's broken down for you before."

---

### Q1: "How do you work with engineers to define success metrics for a model?"

**What they're testing:** Do you understand that model quality metrics and product metrics are different things? Do you know the limits of automated metrics?

**Answer:**
> "I separate model quality metrics from product outcome metrics and make both explicit before any engineering work starts.
>
> Model quality metrics answer 'is the model technically performing?' For a time series forecasting model, that's things like MAPE (mean absolute percentage error) on held-out data, anomaly detection precision and recall, and benchmark performance — for TSM, GIFT-Eval plus an internal observability benchmark. These are necessary but not sufficient.
>
> Product outcome metrics answer 'is the model actually useful to operators?' For TSM, my proposed north star would be: what percentage of incidents does TSM detect that existing threshold alerting misses, with a false positive rate below X? A model that improves MAPE by 15% but doesn't change detection rate has no product value.
>
> The third layer is what I call the human signal gate. Automated metrics tell you direction — are we improving? Human signal tells you arrival — is it good enough to use? For Vigil, the engineering team was proud of 89% retrieval accuracy on SPL queries. But when I ran five operators through the system, three of them said the retrieved context was 'mostly irrelevant.' There was no metric capturing semantic relevance — only retrieval precision. We added a relevance rating step to the evaluation loop, which found a systematic gap the automated metric was missing.
>
> I'd run the same pattern for TSM: define automated benchmarks with engineering, define outcome metrics with the product team, then run a structured pilot with five operators to find what the automated metrics miss."

**Follow-up:** *"What if engineering and product can't agree on which metrics matter?"*
> "That's usually a proxy for a more fundamental disagreement about who the primary customer is. I'd make that explicit: 'We seem to have different assumptions about who we're optimizing for.' Then I'd drive alignment on the customer segment first — SRE? SOC analyst? Developer? — and the metric disagreement usually resolves once the customer is agreed."

---

### Q2: "Walk me through how you'd productize the Cisco Time Series Model."

**If it's Liang Gou, he built this model — so your framing matters a lot here.**

**Answer:**
> "Before I do, I want to understand the current deployment state — is the model running in production anywhere? Are there internal teams using it? That shapes the productization path significantly.
>
> Assuming it's research-to-production: I'd run three parallel workstreams.
>
> **Workstream 1 — Customer validation (weeks 1–4):** Identify three to five Splunk customers who are experiencing the specific pain TSM solves — high false positive rate from threshold alerting, missed capacity events, irregular telemetry that breaks existing anomaly rules. Run the model against their historical data. Don't ask them 'is this useful' — show them incidents from the past six months and ask 'would TSM have caught this earlier?' That's the product-market fit test.
>
> **Workstream 2 — API contract (weeks 2–6 with engineering):** Define the input/output contract. Questions I'd bring: minimum history window for meaningful inference? Output format — point forecast, anomaly flag, confidence interval? Latency SLA per query? What's the failure mode when input data is sparse or has gaps — does the model degrade gracefully or fail hard? The API contract is the customer-facing commitment; I'd want it specified before we write documentation.
>
> **Workstream 3 — Evaluation framework (weeks 3–8):** Build the benchmark kit customers can run against their own data. This is the sales motion enabler — instead of 'trust our GIFT-Eval numbers,' we ship a tool customers can use to evaluate TSM against their telemetry and their existing alerting. That benchmark kit is also the feedback loop for model improvement.
>
> North star for launch: three design partner customers running TSM in production with documented detection improvement over their existing alerting baseline."

**Follow-up (Liang specifically):** *"The model is zero-shot — do you think customers will need fine-tuning options?"*
> "Depends on the use case. For general anomaly detection on common observability patterns — CPU, memory, network throughput — zero-shot should generalize well given the training data distribution. The cases where I'd expect fine-tuning pressure are highly domain-specific telemetry: specialized hardware metrics, proprietary application logs, or anything with irregular seasonality that doesn't appear in general observability data. I'd watch the customer validation closely for 'it works on standard metrics but fails on our custom stuff' — that's the signal that we need a LoRA adapter path, even if it's not in v1."

---

### Q3: "Tell me about a time you pushed back on an engineering recommendation."

**The story — RAG-first vs. generative-first in Vigil:**
> "In Vigil, the initial engineering proposal was to use an LLM to generate SPL queries from natural language, then execute them. I pushed back on that architecture.
>
> My concern was hallucination on schema. Splunk environments have customer-specific field names, custom source types, and proprietary lookup tables. A generative approach would produce syntactically valid SPL that fails on execution because it references fields that don't exist in that customer's environment. I'd seen this failure mode in an earlier prototype — the model was confident and wrong.
>
> My proposal: RAG-first. Build a retrieval layer over the customer's actual schema — field names, source types, existing searches — and use that as context before generation. The LLM generates against retrieved ground truth rather than generating from scratch.
>
> Engineering's concern was latency — retrieval adds a round trip. My response: measure the latency cost and compare it to the latency of a failed query plus the analyst having to debug it. A 400ms retrieval overhead that produces a working query is faster end-to-end than a 100ms generation that produces a broken query.
>
> We shipped the RAG-first approach. The result was measurable: hallucinated query rate dropped significantly, and MTTR went from 47 minutes to 35 seconds. The retrieval overhead was 380ms — well within the operator's tolerance for a query that works."

**Follow-up:** *"What would you have done if engineering had strong reasons to prefer the generative approach?"*
> "I'd have asked to run both approaches on a sample of real customer queries and measure the execution success rate. If generative-first had a higher execution success rate on real data, I'd have conceded. My concern was a failure mode — if the data doesn't show the failure mode, the concern is theoretical. I'm happy to be wrong when there's data."

---

### Q4: "How do you handle a situation where an engineer says 'that's technically impossible'?"

**Answer:**
> "I distinguish between 'impossible' and 'not possible in the way you've framed it.'
>
> 'Technically impossible' is often shorthand for 'this would require architectural changes we haven't budgeted for,' or 'this would require a capability the model doesn't have,' or 'this conflicts with a constraint you don't know about.' Each of those is a different problem.
>
> My response: 'Walk me through what specifically breaks.' Not challenging — genuinely trying to understand the constraint. Usually one of three things happens. First, the engineer articulates a real constraint I wasn't aware of, and I redesign the requirement around it. Second, the engineer articulates a constraint that's actually a design choice, and we explore whether it's the right choice. Third, the engineer was using 'impossible' loosely and means 'hard within the current sprint' — and then we have a scope conversation, not a capability conversation.
>
> In Stil, the engineering team told me that sub-second Feed Cohesion Scores were impossible because the semantic theme extraction required an LLM call. I asked what specifically was slow. The answer: Sonnet, which they'd defaulted to for all LLM calls. I proposed Haiku for the theme extraction specifically — it's 10x cheaper and 3x faster with acceptable quality for this task. Sub-second became achievable. 'Impossible' was actually 'impossible with the model we defaulted to.'"

---

### Q5: "Tell me about a time you and an engineer disagreed on scope. How did you resolve it?"

**The story — StoryForge variation count:**
> "In StoryForge, the engineering team wanted to cap variation generation at five per session — five hook variations per brief. I wanted fifteen.
>
> Their argument was compute cost and latency. Fifteen variations at the model size we were using would take 4+ minutes, which violated the TTFV target I'd set of under 5 minutes at p95.
>
> My argument was that five variations wasn't enough to guarantee meaningful differentiation across hook psychologies. The north star metric was Hook Performance Variance — I needed at least a 3× spread between the best and worst hook to prove the variation strategy had real value. With five variations, the probability of hitting all six hook archetypes (curiosity, fear, social proof, utility, contrarian, aspirational) was low.
>
> We resolved it by separating generation from computation. Instead of generating fifteen variations at once, we generate three immediately and queue the remaining twelve asynchronously. The operator gets their first three variations under 90 seconds, which exceeds the TTFV target. The full fifteen are ready within 4 minutes, which they see when they return to review. No latency violation, full variation coverage. Engineering proposed the async architecture — I provided the constraint that drove it."

**Follow-up:** *"What if there had been no creative solution and you'd had to choose — five variations or the latency target?"*
> "I'd have asked which constraint was harder to change in the next quarter. If latency target was fixed by a customer SLA, I'd take five variations and work on variation quality. If latency target was internal and somewhat flexible, I'd push for fifteen and set an explicit optimization target for the next sprint. I don't like false choices — I'd run both through the decision framework before accepting that it was binary."

---

### Q6: "How do you think about the evaluation framework for the Time Series Model specifically?"

**This is the most technically specific question Liang Gou could ask.**

**Answer:**
> "Three layers.
>
> **Layer 1 — Standard benchmarks.** GIFT-Eval for general forecasting comparisons. This gives a defensible published benchmark against TimesFM, Chronos, and other foundation models. It's the 'we're credible in the research community' signal.
>
> **Layer 2 — Observability-specific benchmarks.** GIFT-Eval doesn't capture the specific patterns in Splunk telemetry: irregular cadence, metric gaps during incidents, correlated multi-metric failures. I'd build an internal benchmark using synthetic and real anonymized customer data that covers these patterns specifically. This is the 'we outperform general-purpose models on our actual use case' signal.
>
> **Layer 3 — Customer-side evaluation kit.** A structured tool customers can run against their own historical telemetry. Input: 6 months of metric data + incident log. Output: 'TSM would have flagged X% of your incidents Y minutes earlier with Z false positive rate.' This is the sales motion and the feedback loop for model improvement. Customers who run the eval kit and see good results are the design partner candidates. Customers who run it and see gaps are the model improvement signal.
>
> The metrics I'd track across all three layers: MAPE for point forecast accuracy, precision/recall for anomaly detection, lead time improvement (how many minutes earlier does TSM detect vs. existing alerting), and false positive rate. False positive rate is the most important for operator trust — a model that catches everything but fires 200 alerts a day will be disabled within a week.
>
> One thing I'd add: the Blank Drop test. Remove a metric from the input and see if detection degrades appropriately. If the model's anomaly detection doesn't change when you remove the metric it should be tracking, it's pattern-matching on something else. That's a calibration problem that GIFT-Eval won't catch."

---

### Q7: "How do you make engineers feel ownership of product decisions?"

**Answer:**
> "I separate the problem from the solution in every brief I write.
>
> Most PMs write requirements that specify the solution: 'build a button that does X.' That's efficient but it eliminates engineering ownership — the engineer is implementing a spec, not solving a problem.
>
> I write requirements that specify the user outcome and the constraint: 'an operator needs to know within 3 minutes of alert fire whether the incident is noise or signal, with enough context to decide whether to escalate. The solution needs to run within the existing ES UI without a context switch.' Everything else — architecture, implementation, data model — is engineering's domain.
>
> When engineers propose solutions within that frame, they own them. When they propose solutions that violate the constraint, I explain why the constraint exists — not to reject their idea, but so they can redesign with the right information.
>
> The other practice: I name the engineer who led the solution in every customer and stakeholder communication. Not as a formality — as a genuine signal that the product works because of their decision, not just the PM's requirements."

---

### Q8: "What questions do you have for me?"

**For Liang Gou specifically:**

1. *"The multiresolution architecture — when you were designing it, was the primary driver the observability data distribution, or did you find it generalized across other time series domains too? I'm asking because the productization positioning depends on whether we market TSM as observability-specific or as a general enterprise time series model."*

2. *"What's the hardest customer telemetry pattern you've encountered that the model handled worse than expected? Understanding the known failure modes upfront would shape how I structure the customer evaluation framework."*

3. *"For the zero-shot capability — how does the model behave on sparse inputs? Observability data during incidents often has gaps from missing metric collection. I want to understand the graceful degradation story before we make deployment promises to customers."*

**For Shang Cai / Rehan Mulla (if not Liang):**

1. *"Where in the current AI Foundations stack does the PM-engineering boundary create the most friction? I want to understand where the handoffs break down before I start optimizing for the wrong things."*

2. *"What's the technical decision you've made recently that you wish a PM had been more involved in — and one where PM involvement would have slowed you down? That tells me where the partnership adds value and where it gets in the way."*

---

---

# ROUND 3 — PM Peer
## Nick Ma / Mohit Verma / Jon Elli · 45 Minutes

**What the PM peer is testing:** Is this person a collaborator or a competitor? Will they fight for shared engineering resources in a way that's fair? Do they understand the dependencies between foundational platform work and product-facing work? Will they be someone I want to work with for the next three years?

**Your opening:**
> "I want to be honest about how I think PM peer relationships work at the staff level. We're going to share engineering capacity, have overlapping roadmaps, and occasionally want the same resources at the same time. I think the best PM partnerships I've had are the ones where we named those tensions explicitly early, rather than discovering them three months in when a sprint is in conflict. So I'm curious about where the boundaries are between AI Foundations and your team's roadmap — that's probably the most useful thing we can talk about."

---

### Q1: "Tell me about a time you influenced a product decision you didn't own."

**The story — MailIntel feedback loop:**
> "In MailIntel, I was working on the AI email optimization layer — my scope was the generation and send-time optimization. The analytics team owned the performance reporting dashboard. I noticed that the model's recommendations — subject line variants, CTA placement, send time — were being generated without any feedback loop from the performance data. The model was learning in a vacuum.
>
> I didn't own analytics. But I spent two weeks building a case: the generation quality after adding a feedback loop from open rates and CTRs should improve recommendations by a measurable amount within three months. I got the data from a small manual experiment where I manually incorporated analytics into prompts and measured recommendation quality. The improvement was significant enough that the analytics team PM agreed it was worth the joint investment to build an automated feedback pipeline.
>
> The mechanism: I came with a pre-run experiment and a specific ask — 'I need three specific events from your analytics system piped to my feature's context.' Not 'we should integrate our systems' (too vague) and not 'build me this dashboard' (too demanding). A specific, bounded ask with a demonstrated outcome attached to it."

**Follow-up:** *"What if the analytics PM had said no?"*
> "I'd have asked what would need to be true for the answer to be yes. Usually 'no' means 'not now, not in this form, or I don't see the value.' Each of those is a different problem. If it's capacity, I'd offer to scope down the ask. If it's value, I'd strengthen the experiment. If it's priority, I'd find out what's blocking their roadmap and see if there's a way the feedback loop serves both teams."

---

### Q2: "How do you manage competing priorities across teams when resources are shared?"

**Answer:**
> "Explicitly and early — not in the sprint where the conflict surfaces.
>
> At the start of each quarter, I map the critical path for my roadmap against the known dependencies on shared engineering resources. If I need the data infrastructure team for event tracking, and I know they're at capacity in Q2, I either move my work to Q3 or I make the case for why mine should displace something else — with the data to support that argument.
>
> The mistake I've seen PMs make is treating shared resources as a first-come-first-served system — whoever books the sprint earliest wins. That's not a prioritization system, it's a calendar game. It also means that urgent, high-value work can get blocked by lower-value work that was scheduled earlier.
>
> My approach: whenever shared resources are involved, I try to get a joint prioritization conversation on the calendar before the sprint conflict happens. Not 'can I have your team for two weeks' but 'here's what I need, here's the value, here's when — where does it fit in your team's view of priorities?'"

**Follow-up:** *"What do you do when both items are genuinely high priority and there's no obvious answer?"*
> "Escalate with a recommendation. I'd write a one-page decision document: the two competing priorities, the cost of delaying each, my recommendation, and the request — 'I need a tiebreaker from you.' That document serves two purposes: it forces me to think through whether my recommendation is actually right, and it gives the person arbitrating a clear input. 'Help us decide' without a recommendation wastes their time."

---

### Q3: "Tell me about a time a dependency slipped and affected your roadmap. What did you do?"

**The story — Vigil + instrumentation dependency:**
> "In Vigil, the agent's retrieval quality depended on structured event logs that the platform team owned. The platform team had promised to deliver structured OCSF-formatted logs by the start of sprint 3. Sprint 3 started and the logs weren't ready — they'd slipped to sprint 5.
>
> I had two options: wait two sprints (unacceptable — I had two customers waiting) or find a workaround. The workaround was a translation layer: take the existing unstructured logs and run a lightweight extraction step before passing them to the retrieval layer. It added latency and wasn't as clean as native OCSF, but it was shippable.
>
> I did two things in parallel. First, I shipped the translation layer so customer delivery wasn't delayed. Second, I had a direct conversation with the platform PM — not to complain, but to understand why the slip happened and whether the sprint 5 date was real. The answer was resource conflict with another priority. I made the case that Vigil's customer commitment was time-sensitive and negotiated for a scoped version of the OCSF work — just the three log types I needed — that they could deliver in sprint 4 without derailing their other work.
>
> The lesson: when a dependency slips, solve the immediate problem and negotiate the structural problem separately. If you only negotiate, you lose a sprint. If you only workaround, the technical debt accumulates."

---

### Q4: "How do you build influence with teams you don't have authority over?"

**Answer:**
> "Three practices.
>
> First: solve their problem first. Before I ask for anything from a team I don't own, I find something useful I can do for them. Not as a transaction — but because understanding their roadmap well enough to add value means I also understand their constraints, which makes my eventual asks more reasonable.
>
> Second: decision visibility. I share my decision documents with adjacent PM teams before I finalize them. Not for approval — for input. When a PM sees their team's constraints reflected in my decisions, they trust that their perspective is being accounted for.
>
> Third: don't optimize for credit. At the staff level, the PM who insists on being named in every outcome loses influence faster than they gain it. I've found that the PMs who are most effective at influencing shared roadmaps are the ones who can say 'that was Nick's idea' or 'the infrastructure team made that possible' — because it means people bring them problems, which is where real influence lives."

---

### Q5: "Tell me about a time you had to say no to a partner PM's request."

**Answer:**
> "In an earlier role, a partner PM asked for a feature that would have required us to rebuild a core data pipeline — the scope was significantly larger than they realized. The feature itself wasn't unreasonable, but the engineering cost was a full sprint that I couldn't absorb without slipping a committed customer delivery.
>
> My 'no' was structured: here's what it would cost, here's what it would delay, here's the alternative I can offer. The alternative was a lighter version of the feature that used the existing pipeline with a configuration change — it got them 70% of the value at 10% of the cost.
>
> They weren't fully satisfied, but they understood the reasoning. The more important outcome: because I gave them a clear cost breakdown and a real alternative, they brought the next request earlier — before it was urgent — which meant we had time to scope it properly.
>
> The way to say no without damaging the relationship: make it about the cost and the constraint, never about the priority of their work versus yours. 'I can't afford this sprint because of X customer commitment' is a real constraint. 'This isn't important enough' is a judgment that creates resentment."

---

### Q6: "How do you ensure foundational platform work gets proper credit from product teams?"

**This is a uniquely relevant question for AI Foundations — platform work often goes unrecognized.**

**Answer:**
> "This is one of the hardest problems in platform PM work, and I think about it in three ways.
>
> First: connect platform metrics to product outcomes. If the TSM anomaly detection enables a 40% reduction in MTTR for the ES team, that number should appear in both the AI Foundations OKRs and the ES OKRs — with explicit attribution. The platform team's value is visible when it shows up in the product team's results.
>
> Second: make the platform's absence felt. The best way to get credit for the platform is to clearly articulate what would have to be rebuilt without it. If every product team had to build their own time series anomaly detection, what would that cost? That's the platform's value. I'd maintain a running 'what this enables' document updated quarterly.
>
> Third: customer stories. When a customer references 'Splunk's AI' in a success story, identify whether that outcome depended on platform capabilities and make sure that's captured. Executive visibility to customer outcomes is the most durable form of platform credit.
>
> The thing I'd avoid: internal metrics that only platform teams care about — model accuracy scores, API latency percentiles. Those matter for engineering quality but they don't translate to business credit. The translation layer — from technical metrics to customer outcome metrics — is the platform PM's core job."

---

### Q7: "How do you handle a situation where two PM teams want the same engineering resources?"

**Answer:**
> "I try to get the two PMs in a room before it becomes a conflict.
>
> When I can see a resource conflict coming — usually 6–8 weeks out — I'll reach out to the other PM proactively: 'I think we're both going to want the data infrastructure team in Q3. Can we get ahead of that?' Most resource conflicts are worse because they're resolved under sprint pressure rather than with planning runway.
>
> If we can't resolve it between us, I bring both priority cases to the shared engineering manager with a recommendation. The recommendation structure: here's what each initiative is trying to achieve, here's the cost of delaying each by one quarter, here's my view on which should go first and why. I never walk into that conversation without a recommendation — 'you decide' without a recommendation is not fair to the manager and signals I haven't done the analysis.
>
> The thing I try to avoid: arguing based on seniority, relationship, or who asked first. Those are political arguments. The only arguments I'll make are: customer commitment, strategic priority, and dependency sequencing."

---

### Q8: "What do you see as the biggest risk in the AI Foundations roadmap from a cross-team perspective?"

**This shows strategic thinking and cross-team awareness.**

**Answer:**
> "The biggest risk is adoption without instrumentation. The AI Foundations platform will ship capabilities — MCP skill connectors, TSM API, the agent framework — and product teams will integrate them. But if we don't build the feedback loop from product usage back to platform improvement, we're building in the dark.
>
> The specific risk: a product team integrates the TSM API, deploys it to a segment of customers, and gets mixed results. Without instrumentation, we don't know if the mixed results are a model quality problem, an integration problem, or a use case fit problem. The platform team gets blamed for 'the AI didn't work' without the data to diagnose why.
>
> The mitigation: I'd propose a shared instrumentation standard across every product team that integrates AI Foundations capabilities. Not complex — a consistent event schema for 'AI capability invoked, outcome, operator action taken.' That data flows back to the platform team and closes the feedback loop. It also creates shared accountability — the product team has visibility into whether their integration is using the capability well, not just whether the capability worked."

---

### Q9: "Tell me about a time you had to navigate organizational dynamics to get something done."

**Answer:**
> "In StoryForge, I needed a compute budget increase that required sign-off from two teams who didn't share a reporting line. One team owned the GPU infrastructure, the other owned the AI budget allocation. Neither had a direct incentive to prioritize my ask.
>
> Instead of going through official channels — which would have taken weeks — I identified what each team was trying to accomplish. The infrastructure team was trying to demonstrate GPU utilization efficiency. The AI budget team was trying to show that AI investments had measurable ROI.
>
> I reframed my ask for each audience. To infrastructure: 'This workload has a highly predictable usage pattern — daily generation batches — which makes it ideal for demonstrating efficient utilization through scheduled compute.' To AI budget: 'Posted Video Rate is a clean, attributable outcome metric — if I can show that the GPU investment directly drives a 60%+ rate, that's the ROI story your team needs.'
>
> Both approved within a week. The lesson: organizational dynamics are usually not about politics — they're about people trying to hit their own metrics. If you can show your ask helps them hit their metrics, the dynamics change."

---

### Q10: "What questions do you have for me?"

1. *"Where does the boundary between AI Foundations and your team's roadmap create the most friction today? I want to understand the dependency structure before I start building my own view of it."*

2. *"What's the one thing you'd want an AI Foundations PM to do differently from how it's been done before — the thing that would make your team's roadmap easier to execute?"*

3. *"How does Sonal's team think about the balance between platform investment and direct product feature work? That tension is real at every platform layer, and I want to understand how it's being navigated here before I form an opinion."*

---

---

# Cross-Round Principles

## Stories to Deploy by Question Type

| Question Type | Primary Story | Secondary Story |
|---|---|---|
| Technical trade-off | Vigil (FSM vs LLM orchestration) | Stil (pixel math vs learned model) |
| Disagreement with engineering | Vigil (RAG-first) | Stil (eval framework) |
| Scope cut | Vigil (auto-remediation cut) | GSentinel (scope narrow to 67%) |
| Metrics and measurement | Stil (Feed Cohesion Score design) | StoryForge (PVR as north star) |
| Cross-functional influence | MailIntel (analytics feedback loop) | StoryForge (compute budget) |
| Execution alignment | GSentinel (Option C decision) | Vigil (instrumentation dependency) |
| Build vs. buy | INTERVIEW_FRAMEWORK responses | Vigil architecture decisions |
| Developer ecosystem | Vigil (MCP + Splunk) | Stil SDK design |

---

## Traps to Avoid — All Three Rounds

| Trap | What It Sounds Like | Why It Fails | Better Version |
|---|---|---|---|
| Deferring to engineering | "I let the team decide the architecture" | Signals you had no opinion | "I came with two options and their costs; we decided together" |
| Vague impact claims | "It improved significantly" | Unverifiable; suggests you don't track outcomes | Always attach a number: MTTR 47min → 35sec, 67% auto-fix |
| Process answers | "I'd set up a weekly sync and align stakeholders" | Describes process, not judgment | Name the specific tension you resolved and how |
| False modesty | "I was lucky the team was great" | Undersells your judgment calls | "The team executed well; my contribution was [specific decision]" |
| Over-claiming autonomy | "I single-handedly built..." | Credibility risk in a room with engineers | Be precise about what you owned vs. what was collaborative |
| Ignoring the TSM paper | Not referencing Liang/Vedant's work | Signals you didn't prepare | Reference specific technical decisions in the paper; ask about them |

---

## Numbers to Know Cold for All Three Rounds

| Number | Context |
|---|---|
| 47 min → 35 sec | Vigil MTTR improvement |
| 67% | GSentinel auto-fix rate |
| < $0.05 / user / day | Stil cost target |
| < 5 min at p95 | StoryForge TTFV |
| ≥ 60% | StoryForge Posted Video Rate target |
| ≥ 3× | StoryForge hook variance target |
| 300B+ | Cisco TSM training data points |
| >50% observability | TSM data source composition |
| 0.84 | Correlation between pixel math and human visual consistency ratings (Stil) |
| 380ms | Vigil RAG retrieval overhead (within operator tolerance) |
| $0.003 | CI pipeline cost per competitive intelligence report |

---

## Opening and Closing Templates

### Universal Opening (adapt per round):
> "Before we get into questions — I want to be direct about how I think about [this specific topic]. I've built [relevant project] and the thing I learned is [specific insight]. That's the frame I'd bring here."

### Universal Closing:
> "One thing I'd add before we close: the reason this role specifically is interesting to me is [specific thing about TSM / agentic workflows / MCP]. I've built Vigil on Splunk's MCP server — I have a working view of both what's hard about it and where the real opportunity is. I'd rather work on the problem than talk about it."

---

## The Single Most Important Sentence for Each Round

**Vedant (Engineering Leader):**
> *"I built a network incident agent on Splunk's MCP server. MTTR went from 47 minutes to 35 seconds. The architecture decision that mattered most was FSM over LLM orchestration — for the same reason that matters here: in production operations, predictable failure is safer than flexible failure."*

**Liang Gou (Engineering Peer):**
> *"I read the paper. The multiresolution architecture is solving a real problem — single-resolution models miss the layered seasonality patterns in observability telemetry. My first question as the PM productizing it would be: what's the known failure mode on sparse inputs during incidents, and how do we communicate that gracefully to operators?"*

**PM Peer:**
> *"I think platform PM work fails when the platform measures itself by technical metrics instead of the outcomes it enables in product teams. My job as AI Foundations PM is to make your team's AI roadmap faster and better — and if I can't show that in your OKRs, I'm not doing it right."*
