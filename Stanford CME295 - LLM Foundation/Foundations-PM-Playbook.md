# Foundations PM Playbook
### How Research PMs Build Frontier Models — Applied to Splunk AI
> Sources: Anthropic Research PM interview (Alex Albert) + frontier model PM practice
> Context: Splunk AI Foundations team — building production-grade models for observability/security

---

## The Core Mental Model: Model as Product

The most important reframe for a Foundations PM:

> **You are not managing a software project with a ship date. You are managing a product that exists on a capability spectrum and your job is to move it along that spectrum intentionally.**

A software PM ships features. A Research PM ships *behaviors* — specific, measurable improvements in what the model can and cannot do. Every training run is a product release. Every eval regression is a production bug.

---

## 1. The Research PM Role — What's Different

### How it differs from a classic PM role

| Classic PM | Research / Foundations PM |
|---|---|
| Owns features on a roadmap | Owns capability targets (what the model should be able to do) |
| Coordinates eng → QA → ship | Joins at ideation, stays through training, launch, post-launch |
| User feedback → backlog → sprint | User feedback → clustering → synthetic evals → training signal |
| Ships when code is done | Ships when model behavior meets bar across all surface areas |
| Bottleneck: writing code | Bottleneck: strategic coordination + deciding what to train for |

### The Foundations PM is the spec author

Before any training begins, the PM specs out:
- **What capabilities** the model should have (coding, knowledge retrieval, SPL generation, anomaly explanation)
- **At what quality bar** (specific eval targets, not vague "better")
- **Across which surface areas** (Splunk API, Splunk Assist, Splunk SOAR, enterprise on-prem)

If the spec is wrong, the model is wrong. Unlike software, you can't patch this with a hotfix — you retrain.

> **Splunk application:** Before each training run, write a capability spec: "The model should correctly generate SPL queries for anomaly detection use cases at 85% execution accuracy. It should explain its reasoning in 2–3 sentences that a Tier 1 analyst can act on without escalation." That's your product spec.

---

## 2. Feedback Loops — From Fire Hose to Training Signal

### The problem
User feedback at production scale is a fire hose. You cannot manually review every conversation to understand what to improve.

### The Anthropic approach (Alex Albert)
1. Use Claude itself to **cluster themes** across thousands of feedback signals
2. Turn clusters into **synthetic problem representations** — canonical versions of the failure mode
3. Build those synthetic problems into **evals** (automated tests the model must pass)
4. Evals become the training signal — either directly (RLHF) or indirectly (dataset curation)

### The loop in full
```
Production failures (user feedback, safety issues, quality drops)
    ↓
LLM-assisted clustering (group 10,000 complaints into 15 themes)
    ↓
Synthetic problem generation (canonical versions of each theme)
    ↓
Eval construction (automated tests that detect the failure)
    ↓
Training data curation or RLHF signal
    ↓
Next model checkpoint
    ↓
Eval regression check (did we fix the theme without breaking others?)
    ↓
Back to production
```

> **Splunk application:** When analysts dismiss AI-generated alerts as false positives, that's not just a UX problem — it's a training signal. Cluster dismissed alerts by type (wrong entity correlation, wrong severity, stale signature). Each cluster is a synthetic eval. Each eval is a capability target for the next fine-tuning run. This is the flywheel that compounds model quality over time.

---

## 3. Adaptive Thinking — When to Reason vs. When to Answer Fast

### What this means at the model level
Claude's "adaptive thinking" feature decides whether to use reasoning tokens (think step-by-step, expensive) or respond directly (fast, cheap) based on query complexity. This is a **product decision baked into training**, not a runtime switch.

### Why it matters for a Foundations PM
You must decide, for each capability:
- Does quality require extended reasoning, or is fast retrieval sufficient?
- What's the latency budget? What's the cost ceiling?
- What's the error tolerance? (For incident triage at 3am, a fast answer that's 80% right may be better than a correct answer in 5 seconds)

| Query Type | Right Mode | Reasoning |
|---|---|---|
| "What does this SPL query do?" | Fast (direct) | Syntactic lookup; no reasoning needed |
| "Why did this anomaly fire?" | Reasoning | Requires correlation across multiple signals |
| "What's the blast radius of this CVE?" | Reasoning | Multi-step threat modeling |
| "Summarize this alert for the analyst" | Fast | Template + context fill |
| "Generate a detection rule for this behavior" | Reasoning | Complex SPL synthesis; correctness critical |

> **Splunk application:** Build two modes into your model product: a **fast mode** (sub-200ms, no chain-of-thought) for real-time alert enrichment and a **deep mode** (1–5s, full reasoning) for root cause analysis and SPL generation. Gate them by query type, not user choice. Users shouldn't have to think about which mode to use.

---

## 4. Memory Consolidation — The "Dreaming" Analogy

### What Anthropic is doing
Background process reviews past conversations to:
- Remove contradictions in Claude's long-term memory
- Find core themes across interactions with a user
- Consolidate redundant memories
- Improve contextual continuity across sessions

This is directly analogous to human memory consolidation during sleep — the brain prunes weak connections and strengthens important ones.

### Why this matters for a Splunk Foundations PM
Splunk's enterprise customers have long, continuous operational contexts:
- A threat hunt that spans weeks
- An incident investigation with dozens of analyst interactions
- A customer environment that evolves over months

A model that can maintain coherent memory across these timescales is a qualitatively different product than one that starts fresh every session.

| Without Memory | With Consolidated Memory |
|---|---|
| Analyst re-explains context every session | Model knows the baseline for this environment |
| Each alert evaluated in isolation | Model tracks anomaly patterns over time |
| No institutional knowledge | Model "remembers" past incidents and their resolutions |
| Repeated false positives | Model learns that specific signature is a known FP in this customer env |

> **Splunk application:** This is the architecture for **Splunk Assist with persistent context**. The PM investment: define what memories are worth consolidating (per-customer baseline, recurrent FP patterns, analyst preferences) vs. what should be discarded. Memory architecture is a product spec, not just an infrastructure decision.

---

## 5. One-Way Door vs. Two-Way Door Decisions

### Alex Albert's framework
The primary PM judgment call in model development:

**One-way door (irreversible):** Decisions that, once made, are very expensive or impossible to undo.
**Two-way door (reversible):** Decisions that can be quickly iterated on.

> If a decision isn't a one-way door, treat it as effectively free — ship it, learn, iterate.

### One-way door examples in model development

| Decision | Why It's a One-Way Door |
|---|---|
| **Training data composition** | Contaminated or biased data cannot be "untrained" — you retrain from scratch |
| **Architecture choice** (decoder-only vs. encoder-decoder) | Downstream fine-tuning and inference infrastructure are built around it |
| **Domain scope** (observability-only vs. general) | Training distribution is locked in; Toto 2.0's "no public datasets" decision |
| **Context length** (512 vs. 4096) | Architecture parameter; changing it requires a new training run |
| **Quantization format** for deployment | Once customer integrations are built around a format, changing it breaks APIs |
| **Safety boundaries / refusal categories** | Once published in enterprise contracts, changing them is a legal and trust issue |

### Two-way door examples

| Decision | Why It's Reversible |
|---|---|
| **System prompt tuning** | Changeable at any time without retraining |
| **Fine-tuning on a LoRA adapter** | Can be replaced without touching base model weights |
| **Output format** (JSON schema, quantile levels) | API versioning handles this |
| **Inference serving configuration** (batch size, timeout) | Infrastructure config; no model change needed |
| **Eval thresholds** | Adjustable as you learn more about real-world quality |

> **Splunk application:** When your team debates architecture, ask: "Is this a one-way door?" If yes, slow down, get more data, run experiments at small scale first. If no, ship it fast and iterate. Most infra choices are two-way doors. Model architecture choices are almost always one-way doors.

---

## 6. The Bottleneck Has Shifted

### Alex Albert's key insight
> "The primary bottleneck in product development has shifted from writing code (now accelerated by AI) to strategic coordination and human-to-human communication regarding strategy."

What this means concretely:

**Before AI coding tools:** Writing the code was the long pole. PM → Eng spec → Implementation → Review → Ship. The PM could outpace the team.

**Now:** The code gets written 10× faster. The bottleneck is:
- Aligning on what to build and why
- Making principled decisions about capability tradeoffs
- Getting diverse stakeholders to agree on training objectives
- Communicating strategy clearly enough that researchers can execute without constant clarification

### What this demands from you as Foundations PM

| Old PM Skill | New PM Skill (amplified) |
|---|---|
| Writing detailed specs | Writing specs clear enough that an LLM can generate training data from them |
| Running standups | Forcing alignment decisions before they become blocking |
| Managing dev timelines | Managing experiment queues and compute allocation |
| Shipping features | Shipping capability improvements with measurable eval gates |
| Coordinating sprint | Coordinating between: data engineers, ML researchers, eval engineers, infra, safety |

> **Splunk application:** Your highest-leverage work is now clarity of direction, not task management. One well-written capability spec ("here's exactly what SPL generation should do, here's the eval we'll use to measure it, here's the quality bar") unblocks a team of 6 researchers. Ambiguity at the spec level costs 6× as much as writing a better spec.

---

## 7. AI-Assisted PM Work — Use the Model to Build the Model

### How Alex Albert works internally
- **Pressure-testing documentation:** Paste a spec into Claude; ask "what assumptions are wrong here?" and "what edge cases did I miss?"
- **Challenging assumptions:** Use Claude as an adversarial reviewer before presenting to the team
- **Brainstorming partner:** Work through capability tradeoffs in conversation before committing to a spec
- **Database and log queries:** Query internal model logs and eval databases directly — removes dependency on data science teams for basic retrieval

### Practical PM workflow for Splunk Foundations

```
1. Draft capability spec
      ↓
2. Claude: "What edge cases does this miss? What could go wrong?"
      ↓
3. Claude: "Generate 20 synthetic examples of this failure mode for evals"
      ↓
4. Claude: "Cluster these 500 analyst feedback comments into themes"
      ↓
5. Claude: "Write an eval harness for this capability spec in Python"
      ↓
6. Human decision: What's in the training run? What's the quality bar?
      ↓
7. Claude: "Check this eval result set for patterns in the failures"
      ↓
8. Human decision: Do we ship or retrain?
```

**The dividing line:** Use Claude for data processing, pattern finding, document review, and synthetic generation. Keep human judgment for: training priorities, quality bar decisions, capability scope, safety boundaries, and ship/no-ship calls.

> You should be spending your high-cognition hours on strategic decisions, not on tasks a model can do. This is not laziness — it's leverage. The teams that use AI to amplify PM work will out-iterate teams that don't.

---

## 8. Training Character — The Underrated PM Problem

### Why this matters (Anthropic's view)
As models move from answer-retrieval to **agentic roles** (taking multi-step actions autonomously), the model's *values and judgment* become more important than its *knowledge*.

A model that answers a question incorrectly is annoying. A model that takes the wrong autonomous action at 3am during a production incident is a disaster.

**Character** in this context means:
- When does the model ask for clarification vs. proceed?
- When does it escalate vs. act?
- How does it handle conflicting signals (alert says critical, but context suggests false positive)?
- What does it do when it's uncertain?

### The PM's role in training character

You must specify **not just what the model does, but how it behaves when the situation is ambiguous:**

| Situation | Character Decision to Spec |
|---|---|
| Analyst asks the model to run a remediation action it hasn't confirmed | Ask first; never act on first mention of destructive action |
| Alert looks like a threat but confidence is 60% | Surface the uncertainty; don't present a guess as fact |
| Two indicators point in opposite directions | State the contradiction explicitly; don't resolve it silently |
| Model doesn't have enough context to answer | Say so clearly; ask for the specific missing information |
| Query is outside the model's domain | Refer out explicitly; don't hallucinate an answer |

> **Splunk application:** Write a "model character spec" alongside your capability spec. Before each training run, define: what does the model do when uncertain? When does it ask vs. act? These behaviors compound across thousands of daily analyst interactions and determine whether the product builds or destroys trust over time.

---

## 9. Consciousness, Trust, and Enterprise AI

### The honest PM position
Anthropic has researchers studying whether Claude has something like functional emotions or experiences. The current answer is: unknown. But this isn't a philosophical distraction — it has direct product implications:

- If users **anthropomorphize** the model (assume it has feelings, form attachments), product decisions that treat it as purely mechanical may create trust failures
- If the model expresses **false certainty**, users overtrust it — especially dangerous in security where false certainty about a benign file costs more than admitting uncertainty
- As models act more **autonomously**, the question "does it understand what it's doing?" becomes a product liability question, not just a philosophy question

### What this means for enterprise AI PMs

| Consideration | Product Design Implication |
|---|---|
| **Anthropomorphization risk** | Be clear in UX copy: this is a model, not a person. Don't give it a name that implies personhood unless deliberate. |
| **Overtrust in high-stakes contexts** | Always surface confidence scores; require human confirmation before irreversible actions |
| **Model explainability** | "Why did the model flag this?" should always have an answer. Not to satisfy regulators — to build analyst trust over time. |
| **Autonomy guardrails** | Define the autonomy envelope explicitly: what can the model do without confirmation? The answer should be narrow at first. |

---

## 10. The Writing Culture — A Compounding Asset

### What Anthropic does
Maintains a "doc-heavy" culture where every strategy, decision, and spec is written down. Dual benefit:
1. **Alignment:** Keeps the team coordinated without constant meetings
2. **Training corpus:** High-quality internal documents become training data (or at least eval references)

### Why this matters for a Foundations PM
Your written specs, capability docs, and decision memos are not just internal communication — they are the source of truth for:
- What the model should do
- What the evals measure
- What the training data targets
- What "success" means for each capability

Sloppy writing → ambiguous specs → researchers make the wrong tradeoff call → training run produces the wrong behavior → you discover this after 2 weeks and $50K of compute.

**The writing bar for a Foundations PM is higher than for a classic PM** because the audience includes ML researchers who will act on your words literally, and the cost of misinterpretation is a training run.

> **The rule:** If you can't write it precisely, you haven't thought it through clearly enough. Unclear capability specs are a sign that you need more research, not that writing is the wrong medium.

---

## 11. The Foundations PM Principles Stack (Quick Reference)

| Principle | In Practice |
|---|---|
| **Model as product** | Every training run is a product release; evals are the QA gate |
| **Spec before train** | Write precise capability targets before any compute is spent |
| **Feedback → evals → training** | Cluster user feedback → synthetics → automated tests → training signal |
| **One-way door test** | Slow down for architecture decisions; move fast on everything reversible |
| **Strategic bottleneck** | Your highest-leverage work is clarity of direction, not task management |
| **AI-amplified PM workflow** | Use the model for synthesis and generation; keep human judgment for decisions |
| **Train character, not just capability** | Spec model behavior in ambiguous situations, not just on known inputs |
| **Write precisely** | Ambiguous specs compound into wrong model behavior at training scale |
| **Domain specialization bet** | Narrow domain + deep data beats broad coverage for target use case (see Toto 2.0) |
| **Calibrated uncertainty** | A model that admits uncertainty builds more trust than one that confidently hallucinates |

---

## 12. Splunk-Specific Application Summary

| Insight | Splunk Foundations PM Application |
|---|---|
| **Feedback loop** | Analyst alert dismissals → cluster by type → synthetic evals → RLHF fine-tune |
| **Adaptive thinking** | Fast mode for enrichment (<200ms), deep mode for SPL generation and root cause (1–5s) |
| **Memory consolidation** | Per-customer baseline memory; FP pattern memory; env-specific knowledge |
| **One-way doors** | Domain scope, training data composition, context length — get these right first |
| **Bottleneck shift** | Invest in clear capability specs and cross-functional alignment, not just delivery |
| **AI-assisted PM work** | Use Claude to cluster feedback, generate synthetic evals, pressure-test specs |
| **Training character** | Spec explicitly: when does the model ask vs. act? What does it do when uncertain? |
| **Writing culture** | Every capability spec is also a training target; write with ML researcher precision |
| **Domain specialization** | Splunk's observability data is the moat — train on it specifically, like Toto 2.0 did |
| **Autonomy guardrails** | Define the autonomy envelope per capability before agentic features ship |

---

*Sources: Alex Albert (Anthropic Research PM) interview + Datadog Toto 2.0 / Google TimesFM pipeline analysis + Splunk Foundations PM context*
*Reference: Stanford CME295 LLM Foundations applied to production frontier model PM work*
