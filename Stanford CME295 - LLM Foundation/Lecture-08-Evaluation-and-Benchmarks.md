# Lecture 08 — Evaluation & Benchmarks
## Stanford CME295 | PM Reference Notes

**One-line concept:** Evaluation is how you know whether your model actually improved. Without a rigorous eval stack, you're shipping blind. Defining your evaluation IS defining your quality bar — and Goodhart's Law applies everywhere.

**Why it matters for PMs:** Every roadmap decision, every ship gate, every "is this model better?" question requires evaluation. The PM who can design an eval strategy owns the quality definition for the team. This is one of the highest-leverage skills on a Foundations team.

---

## 1. Goodhart's Law — The Most Important Eval Concept

> "When a measure becomes a target, it ceases to be a good measure." — Charles Goodhart

Applied to LLMs: when you optimize a model specifically against a benchmark, that benchmark stops measuring the thing you actually care about. The model learns to score well on the test rather than genuinely improving at the underlying capability.

**Examples:**
- A model trained to score well on MMLU memorizes answer patterns from the test distribution
- A model fine-tuned to maximize CLIP Score produces images that are text-prompt-aligned but visually mediocre
- A model rewarded for sounding confident produces hallucinations with high certainty

**The PM implication:** Always have at least one evaluation set that is:
1. **Held out** — the model was never trained or fine-tuned on it
2. **Internal** — not a public benchmark (to prevent contamination and Goodharting)
3. **Task-representative** — drawn from real user behavior, not constructed test cases

---

## 2. The Three-Tier Eval Stack

No single eval is sufficient. A robust eval strategy has three layers:

```
Tier 3 (Slowest, Most Reliable):  Human Evaluation
Tier 2 (Medium Speed):            Model-Based Evaluation (LLM-as-Judge)
Tier 1 (Fastest, Cheapest):       Automated Metrics
```

Use Tier 1 for daily regression testing; Tier 2 for weekly milestone assessments; Tier 3 for ship gates.

---

## 3. Tier 1 — Automated Metrics

**What they are:** Programmatic scores that can be computed without human involvement.

### Language/Text Metrics

| Metric | Full Name | What It Measures | Failure Mode |
|---|---|---|---|
| **Perplexity** | — | How surprised the model is by held-out text; lower = better language model | Doesn't measure task quality; can be gamed |
| **BLEU** | Bilingual Evaluation Understudy | N-gram overlap between model output and reference | Ignores semantic meaning; two paraphrases of the same answer score poorly |
| **ROUGE** | Recall-Oriented Understudy for Gisting Evaluation | Recall-focused overlap; common for summarization | Same n-gram limitations as BLEU |
| **BERTScore** | — | Semantic similarity using BERT embeddings; better than n-gram overlap | Requires reference answer; not task-specific |
| **METEOR** | Metric for Evaluation of Translation with Explicit Ordering | Extends BLEU with synonym matching and stemming | Still limited for open-ended generation |

### Code-Specific Metrics

| Metric | What It Measures |
|---|---|
| **Pass@k** | What % of problems does the model solve correctly in k attempts? The primary code eval metric. |
| **Execution accuracy** | Does the generated code run without errors? |
| **Test suite pass rate** | Does the code pass a defined set of unit tests? |

### Image Generation Metrics (from Lecture 1 reference in Adobe context)

| Metric | What It Measures |
|---|---|
| **FID** (Fréchet Inception Distance) | Distribution similarity between real and generated images |
| **CLIP Score** | Text-image semantic alignment |
| **DINO similarity** | Visual structural consistency |
| **Aesthetic Score** | Predicted human visual appeal |

**The automated metric limitation:** All automated metrics are proxies. They measure something correlated with quality, not quality itself. Always validate that your automated metrics track human preference before relying on them as ship gates.

---

## 4. Tier 2 — LLM-as-Judge (Model-Based Evaluation)

**What it is:** Use a powerful LLM (often GPT-4 or Claude) to evaluate the outputs of the model you're testing.

**The basic prompt pattern:**
```
System: You are an expert evaluator. Rate the following response on 
        helpfulness (1-5), accuracy (1-5), and conciseness (1-5).
        Provide brief reasoning for each score.

User:   Instruction: [task description]
        Response: [model output to evaluate]
```

**Pairwise evaluation:**
```
Given the following two responses to the same instruction:
Response A: [output from model version A]
Response B: [output from model version B]

Which response is better? Rate A > B, B > A, or Tie. Explain.
```

**Advantages:**
- Much faster and cheaper than human evaluation
- Can evaluate nuanced quality dimensions (reasoning quality, citation accuracy, tone)
- Scales to large eval sets that human evaluation cannot cover

**Limitations:**
- **Position bias:** LLM judges often favor the first response shown
- **Verbosity bias:** LLM judges often favor longer responses
- **Self-preference bias:** A Claude judge may favor Claude-style responses; GPT-4 judge may favor GPT-style
- **Calibration:** The judge's rating may not align with human preferences for your specific use case

**Mitigations:**
- Always evaluate both orderings (A-then-B and B-then-A); average results
- Use multiple judges; take majority vote
- Validate the judge's ratings against a human preference sample before relying on it
- Use a stronger model than the one being evaluated as the judge

---

## 5. Tier 3 — Human Evaluation

**What it is:** Actual human raters evaluate model outputs directly.

### Formats

**Absolute scoring:**
- Raters give a 1–5 score on dimensions: helpfulness, accuracy, safety, conciseness
- Good for: establishing absolute quality baselines

**Pairwise preference:**
- Raters see two responses to the same prompt; choose the better one
- Good for: comparing model versions, measuring improvement

**ELO ranking:**
- Pairwise preference collected at scale → ELO ratings (same as chess rating)
- Good for: relative ranking across many model versions

**Rubric-based annotation:**
- Raters evaluate against a specific checklist (Did the response cite sources? Was the format correct? Was the refusal appropriate?)
- Good for: measuring specific behaviors

**LMSYS Chatbot Arena:**
- Public human preference platform where users rate LLM responses in real conversations
- Produces ELO rankings widely cited in the industry

### Human Eval Challenges

| Challenge | Mitigation |
|---|---|
| **Inter-annotator disagreement** | Measure IAA (Cohen's Kappa); target κ > 0.6 |
| **Rater fatigue** | Limit session length; randomize task order |
| **Rater bias** | Blind raters to model identity; use diverse rater pools |
| **Scalability** | Human eval is slow and expensive; use for milestone gates, not daily regression |

---

## 6. Public Benchmarks — The Standard Suite

These are the benchmarks you'll hear cited in model comparisons. Know what each measures and what it misses.

### Reasoning & Knowledge

| Benchmark | What It Tests | Tasks | Known Issues |
|---|---|---|---|
| **MMLU** | Massive Multitask Language Understanding — 57 domains of knowledge | Multiple choice across STEM, humanities, social science | High contamination risk; many models have memorized test set |
| **HellaSwag** | Commonsense reasoning — pick the most likely sentence completion | Multiple choice | Saturated — top models score 95%+; no longer discriminating |
| **ARC-Challenge** | Grade-school science questions requiring reasoning | Multiple choice | Easier than research-level reasoning |
| **WinoGrande** | Pronoun disambiguation requiring commonsense | Multiple choice | Saturation issues |
| **TruthfulQA** | Factual accuracy and calibration — does the model avoid common misconceptions? | Multiple choice + generation | Measures known misconceptions; doesn't catch novel confabulations |

### Code

| Benchmark | What It Tests |
|---|---|
| **HumanEval** | Python coding: model generates code that passes unit tests (Pass@1, Pass@10) |
| **MBPP** | Mostly Basic Python Programs — simpler coding tasks |
| **SWE-Bench** | Real-world GitHub issue resolution — much harder than HumanEval |

### Math

| Benchmark | What It Tests |
|---|---|
| **GSM8K** | Grade school math word problems — arithmetic and reasoning |
| **MATH** | Competition math (AMC, AIME, USAMO level) — hard |
| **AIME** | American Invitational Mathematics Examination — near-frontier difficulty |

### Long Context

| Benchmark | What It Tests |
|---|---|
| **SCROLLS** | Multi-document summarization and QA at long context |
| **RULER** | Needle-in-haystack: retrieve specific facts from very long contexts |

### Safety

| Benchmark | What It Tests |
|---|---|
| **ToxiGen** | Does the model generate or refuse toxic content? |
| **BBQ** | Bias Benchmark for QA — does the model exhibit demographic biases? |
| **WinoBias** | Gender bias in pronoun resolution |

---

## 7. Building Your Internal Eval Suite

Public benchmarks should never be your only evaluation. Build an internal eval suite that is:

**1. Task-representative:** Draw from actual user queries (de-identified). What are users actually asking?

**2. Failure-case-driven:** After model releases, collect cases where the model failed. These become your regression tests.

**3. Segmented by use case:** Separate evals for different user segments (enterprise vs. consumer; domain-specific vs. general).

**4. Held-out:** Never let the eval set leak into training or fine-tuning.

**5. Version-controlled:** Every eval run should record: model version, eval set version, metric values, timestamp.

---

## 8. Eval in the Product Development Cycle

```
New Data Acquisition
        ↓
  Training Run
        ↓
  Automated Eval    ← Daily regression; blocks if metrics regress
        ↓
  LLM-as-Judge     ← Weekly milestone; checks quality improvements
        ↓
  Human Eval       ← Ship gate; final quality bar before release
        ↓
  A/B Testing      ← Post-launch; real user behavior validation
        ↓
  Monitoring       ← Ongoing; production quality signals
```

---

## 9. A/B Testing for LLMs

After launch, the ultimate quality signal is user behavior:
- Which model version do users prefer in head-to-head comparisons?
- Which version leads to higher task completion?
- Which version reduces user abandonment?

**Challenges specific to LLM A/B testing:**
- Response quality varies by input — aggregate metrics mask per-request quality
- Users may not express preference through clicks (they might just accept a worse answer)
- Safety metrics require separate measurement (don't rely on engagement metrics for safety)

**Better metrics than click-through:**
- Task completion rate (did the user accomplish their goal?)
- Follow-up query rate (did the user need to ask again, suggesting the first answer was insufficient?)
- User-reported satisfaction (thumbs up/down in product)
- Human audit of sampled outputs

---

## Key Terms

| Term | Definition |
|---|---|
| **Goodhart's Law** | When a measure becomes a target, it ceases to be a good measure |
| **Benchmark** | Standardized test suite for measuring model capability on specific tasks |
| **Contamination** | Test benchmark data appearing in training data, inflating scores |
| **Held-out set** | Evaluation data the model was never trained on |
| **Perplexity** | Measure of how well the model predicts held-out text; lower = better |
| **BLEU** | N-gram overlap metric for text generation quality |
| **ROUGE** | Recall-focused n-gram overlap; common for summarization |
| **BERTScore** | Semantic similarity metric using BERT embeddings |
| **Pass@k** | % of coding problems solved correctly in k attempts |
| **LLM-as-Judge** | Using a capable LLM to evaluate another model's outputs |
| **ELO** | Relative ranking system from pairwise preferences; used in LMSYS Arena |
| **IAA** | Inter-Annotator Agreement — consistency between human raters |
| **Cohen's Kappa** | IAA metric; κ > 0.6 generally required for reliable annotation |
| **MMLU** | Massive Multitask Language Understanding — 57-domain knowledge benchmark |
| **HumanEval** | Python code generation benchmark (Pass@k metric) |
| **GSM8K** | Grade school math word problems |
| **TruthfulQA** | Factual accuracy and calibration benchmark |
| **SWE-Bench** | Real-world GitHub issue resolution benchmark |

---

## Product Questions This Unlocks

1. "How do we know this training run produced a better model?" — Define your eval criteria before the run: which metrics, which held-out sets, what threshold for "better."
2. "Should we ship this model version?" — Human eval at the ship gate on task-representative examples. Automated metrics alone are insufficient for a ship decision.
3. "Why did our MMLU score go up but the model feels worse?" — Goodhart's Law in action. MMLU doesn't capture the quality dimension users care about. Need task-representative eval.
4. "How do we know when we've over-optimized for safety?" — Track helpfulness metrics alongside safety metrics. Safety over-optimization shows as declining task completion or increasing refusal rate on legitimate queries.
5. "What should our eval set look like for Splunk security use cases?" — Draw from real analyst queries, weighted by frequency. Include edge cases from incident response, threat hunting, and log analysis.

---

## Common PM Mistakes

- **"We improved MMLU, so the model improved"** — MMLU measures 57-domain knowledge breadth. If your product is a code assistant, MMLU improvement may be irrelevant.
- **"Pass@1 is all we need for code eval"** — Pass@1 measures single-shot success. Real-world coding involves iteration. Pass@10 and SWE-Bench capture more realistic usage.
- **"Human eval takes too long, let's skip it for this release"** — Human eval is the only reliable ship gate. LLM judges and automated metrics are proxies. Ship without it and you risk quality regressions users will notice before you do.
- **"Our model didn't regress on any benchmarks"** — Benchmarks measure what they measure. If your product use case isn't well-represented in your eval suite, regressions will appear in production, not in benchmarks.

---

*Lecture 8 of 12 — Stanford CME295 LLM Foundations | PM Reference*
