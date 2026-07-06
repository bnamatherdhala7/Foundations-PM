# Lecture 12 — Frontier Models & What's Next
## Stanford CME295 | PM Reference Notes

**One-line concept:** The frontier is moving from "models that know things" to "models that reason, plan, and act." Reasoning models, agentic systems, and tool use define the product decisions that matter in 2025–2027.

**Why it matters for PMs:** The most important product capabilities in the next 2 years are not in bigger base models — they're in reasoning, agents, and tool integration. A Foundations PM at Splunk who understands these directions can anticipate where to invest before competitors do.

---

## 1. The Post-Pretraining Scaling Shift

For most of 2018–2023, the dominant strategy was: scale pretraining compute → better models. This is slowing.

**What's replacing it:**

| Strategy | Period | Mechanism |
|---|---|---|
| **Pretraining scaling** | 2018–2023 | Bigger model + more data = better quality |
| **RLHF / instruction tuning** | 2022–present | Better alignment from human feedback |
| **Test-time compute scaling** | 2023–present | More compute at inference = better reasoning |
| **Synthetic data + RL** | 2024–present | Model generates its own training signal |
| **Agentic RL** | 2025–present | Model learns by interacting with environments |

The current frontier is **test-time compute scaling** (reasoning models) and **agentic reinforcement learning**.

---

## 2. Reasoning Models — The Test-Time Compute Revolution

### What Is Test-Time Compute?

Instead of using more compute at training time, use more compute at inference time — let the model "think" longer before answering.

**The key insight:** For difficult problems (math, code, complex reasoning), allowing the model to generate intermediate reasoning steps dramatically improves accuracy — even without changing the model weights.

### Chain-of-Thought (CoT) Prompting

**Discovery:** Google researchers (Wei et al., 2022) found that prompting the model to "think step by step" or providing examples with explicit reasoning steps dramatically improved performance on math and reasoning tasks.

```
Standard:  "What is 83 × 47?" → Model directly outputs answer (often wrong)

CoT:       "What is 83 × 47? Let's think step by step."
           → Model generates: "83 × 47 = 83 × 40 + 83 × 7 = 3320 + 581 = 3901"
```

CoT emerges above ~50B parameters — smaller models don't benefit.

### o1 / o3 — OpenAI's Reasoning Models

**The approach:**
- Train the model with RL to search for good reasoning chains before answering
- The model learns to generate extended internal reasoning ("thinking tokens") that are optimized for correctness
- Users see only the final answer; the reasoning chain can be many thousands of tokens long

**The results:**
- o1 outperformed PhD-level experts on physics, chemistry, and biology benchmarks
- Near-perfect on AIME (competition math)
- Dramatically better on code: solved 49% of problems on SWE-Bench (previously ~14%)

**The cost:** o1 uses 5–20× more tokens than GPT-4 per query (all the "thinking tokens"). Much more expensive and slower.

### DeepSeek-R1 — Open-Source Reasoning

DeepSeek (Chinese AI lab) replicated the o1 approach and released weights:
- Used GRPO (Group Relative Policy Optimization) instead of PPO
- Trained with pure RL: reward = 1 if final answer is correct, 0 otherwise
- Discovered that reasoning emerged naturally from this objective — without supervised reasoning examples
- Performance comparable to o1 on math and code benchmarks
- Open-weight (accessible to everyone)

**PM implication:** Reasoning capability can emerge from RL with simple correctness rewards. This means domain-specific reasoning (security log analysis, incident root cause) may be achievable with domain-specific RL training even on smaller models.

---

## 3. Agentic AI — Models That Act

An **agent** is an LLM that can take actions in an environment, observe results, and plan multi-step sequences of actions toward a goal.

### The Agentic Architecture

```
User Goal
    ↓
  LLM (the "brain")
    ↓
  Tool Selection
    ↓
  Tool Execution
    ↓
  Observation (tool output)
    ↓
  LLM reasoning about observation
    ↓
  Next tool selection
    ↓
  ... (multi-step loop)
    ↓
  Final Response to User
```

### Tool Types

| Tool Category | Examples | Splunk Relevance |
|---|---|---|
| **Search / retrieval** | Web search, document search, RAG | Log search, threat intel lookup |
| **Code execution** | Python REPL, bash | SPL (Splunk Processing Language) execution |
| **External APIs** | Weather, databases, services | SIEM APIs, threat feeds (VirusTotal, MISP) |
| **File operations** | Read/write files | Log file access, report generation |
| **Browser/UI control** | Web browsing, form filling | Dashboard navigation |
| **Memory tools** | Store/retrieve information | Incident context persistence |

### Why Agents Are Hard to Get Right

| Challenge | Description |
|---|---|
| **Compounding errors** | In a 10-step plan, each step has an error rate. 90% accuracy per step = 35% success on 10-step plan |
| **Irreversibility** | Agents may take real-world actions that can't be undone (send email, delete file) |
| **Context management** | Long multi-step conversations exhaust context windows |
| **Tool selection** | Model must choose the right tool; wrong tool = wasted steps |
| **Prompt injection** | Malicious content in retrieved documents can hijack agent actions |

**The "Human in the Loop" design pattern:**
For high-stakes actions (deleting data, sending alerts, modifying configurations), require human confirmation before execution. This is the primary safety mechanism for enterprise agents.

---

## 4. Multi-Agent Systems

Multiple LLM agents collaborating, each specializing in different tasks:

```
Orchestrator Agent (plans and delegates)
    ├── Research Agent (searches and retrieves)
    ├── Analysis Agent (processes data)
    ├── Code Agent (writes and executes code)
    └── Communication Agent (drafts summaries/reports)
```

**Why multi-agent:**
- Decompose complex tasks into specialized sub-tasks
- Each agent can use a different model (small fast model for retrieval, large model for analysis)
- Parallel execution of independent sub-tasks
- Easier to test, debug, and improve individual agents

**Example for Splunk security:**
```
Orchestrator receives alert
    → Research Agent: searches for CVE details + threat intel
    → Log Analysis Agent: queries relevant logs via SPL
    → Correlation Agent: connects log patterns to threat intel
    → Report Agent: generates incident summary
    → Human approval → Remediation Agent executes fix
```

---

## 5. Retrieval-Augmented Generation (RAG) — Production Architecture

RAG is not just "give the model documents." The production architecture has many components:

```
User Query
    ↓
  Query understanding / expansion
    ↓
  Retrieval (dense + sparse)
    ↓
  Re-ranking (cross-encoder)
    ↓
  Context assembly (select top-K docs)
    ↓
  Augmented prompt → LLM
    ↓
  Response with citations
    ↓
  Fact verification (optional)
```

**Dense retrieval:** Embed query and documents; find nearest neighbors by vector similarity. Good for semantic matching.
**Sparse retrieval:** BM25 keyword matching. Good for exact term matching.
**Hybrid:** Combine both; use RRF (Reciprocal Rank Fusion) to merge results.
**Re-ranking:** Use a more expensive cross-encoder model to re-score the top-K retrieved documents for relevance.

**PM relevance for Splunk:**
Splunk's knowledge base (documentation, SPL syntax, detection rules, past incidents) is a natural RAG corpus. A security copilot grounded in Splunk's own knowledge base and customer-specific runbooks will dramatically outperform a general LLM for Splunk-specific queries.

---

## 6. Long Context vs. RAG

As context windows expand (1M+ tokens), the question arises: why not just put everything in context?

| Dimension | Long Context | RAG |
|---|---|---|
| **Relevance** | All documents present; model finds relevant | Only retrieved documents present |
| **Cost** | Very high (all tokens charged) | Lower (only retrieved + query) |
| **Latency** | High (prefill at full context length) | Lower |
| **Knowledge updates** | Must update entire context | Update retrieval index only |
| **Long-tail performance** | Model struggles at extreme context edges | Only works well on what it retrieves |

**The practical answer:** Use RAG for known, structured knowledge bases. Use long context for one-off tasks where you need to reason over a specific set of documents (e.g., analyze this specific incident's full log file).

---

## 7. Structured Output — Critical for Enterprise AI

**The need:** Enterprise products often need LLM output in structured formats (JSON, XML, SQL) rather than natural language.

**Approaches:**

| Approach | How It Works | Quality |
|---|---|---|
| **Prompt instructions** | "Return JSON in the format: {field: value}" | Unreliable; model may deviate |
| **Output parsing + retry** | Parse output; retry if invalid | Adds latency; still fails for complex schemas |
| **JSON mode / structured outputs** | Model constrained to only produce valid JSON | Reliable; standard in OpenAI, Anthropic APIs |
| **Grammar-constrained decoding** | Inference-time constraint that forces valid output per defined grammar | Perfectly reliable; slight quality tradeoff |

**PM implication:** For any product that feeds LLM output into downstream code, structured output is a reliability requirement, not a nice-to-have.

---

## 8. The Splunk-Specific Frontier Priorities

Given the role is Splunk Foundations PM, these are the most relevant frontier topics to prioritize:

**1. Domain-Specific Reasoning (o1-style, security-focused)**
- Train or fine-tune a reasoning model on security incidents, log analysis, and threat hunting workflows
- RL signal: is the model's root cause analysis correct? Is its SPL query syntactically valid and semantically correct?

**2. SPL (Splunk Processing Language) Code Generation**
- This is the "code generation" problem applied to Splunk's query language
- Eval: execution accuracy + query correctness against known-good results
- Training data: human-written SPL queries with documentation of what they accomplish

**3. Agentic Incident Response**
- Multi-step agent: receive alert → gather context → run queries → correlate → draft response
- Human-in-the-loop for any remediation actions
- Key challenge: compounding error rate over multi-step investigation

**4. RAG over Security Knowledge**
- Grounding responses in: CVE databases, MITRE ATT&CK framework, Splunk documentation, customer runbooks
- The retrieval quality here is critical — a missed CVE reference in a triage is a product failure

**5. Multimodal for Dashboard Analysis**
- Analyzing Splunk dashboards (images or rendered charts) alongside log data
- Still early but directionally important for complex incident visualization

---

## 9. Model Capability Timeline — What to Expect

| Capability | Now (2025) | 2026 | 2027 |
|---|---|---|---|
| **Text reasoning** | Strong (o3-level) | Very strong; domain-specific reasoning | Near-expert level on specialized domains |
| **Code generation** | Good (SWE-Bench 50%) | Multi-file refactoring; autonomous testing | End-to-end software tasks |
| **Agentic tasks** | Basic multi-step | Reliable 5–10 step agents | Complex 20+ step autonomous workflows |
| **Long context quality** | Good to 100K; degrades at edges | Reliable at 1M | Working solutions for full codebases |
| **Multimodal understanding** | Images well; video limited | Video understanding | Real-time multimodal reasoning |
| **On-device models** | 7B models usable | 13B on-device; good quality | 30B+ on-device; near-cloud quality |

---

## Key Terms

| Term | Definition |
|---|---|
| **Test-time compute** | Using more compute at inference to improve quality (vs. more training compute) |
| **Chain-of-thought (CoT)** | Generating explicit intermediate reasoning steps before a final answer |
| **o1 / o3** | OpenAI's reasoning models trained with RL to generate extended thinking chains |
| **DeepSeek-R1** | Open-source reasoning model trained with RL; comparable to o1 on math/code |
| **Agent** | LLM that can select and execute tools in a multi-step loop |
| **Tool use** | Model's ability to call external APIs, execute code, or perform actions |
| **Multi-agent system** | Multiple collaborating LLM agents with specialized roles |
| **RAG** | Retrieval-Augmented Generation — grounding responses in retrieved documents |
| **Dense retrieval** | Vector similarity-based document retrieval |
| **BM25** | Sparse keyword-matching retrieval algorithm |
| **Hybrid retrieval** | Combining dense and sparse retrieval |
| **Re-ranking** | Using a cross-encoder to re-score retrieved documents for relevance |
| **RRF** | Reciprocal Rank Fusion — method to merge rankings from multiple retrieval systems |
| **Structured output** | Constraining model output to valid JSON/XML/SQL format |
| **SPL** | Splunk Processing Language — Splunk's query language |
| **MITRE ATT&CK** | Adversarial tactics and techniques knowledge base; key reference for security LLMs |
| **Human-in-the-loop** | Requiring human confirmation before irreversible agent actions |
| **Compounding errors** | Error accumulation in multi-step agent workflows |
| **GRPO** | Group Relative Policy Optimization — RL algorithm used in DeepSeek-R1 |

---

## Product Questions This Unlocks

1. "Should we build a reasoning model or use o1 via API?" — If Splunk-specific accuracy is the priority and you have domain training data, a smaller domain-specific reasoning model (trained with RL on security tasks) will outperform o1 on your task at lower cost.
2. "How do we build a reliable multi-step investigation agent?" — Define the step-level error budget: if each step has 90% accuracy, a 5-step agent has 59% end-to-end success. Human checkpoints for critical decisions.
3. "Should we use RAG or just give the model a large context window?" — RAG for structured knowledge bases with known schemas (CVEs, ATT&CK, runbooks). Long context for one-off document analysis.
4. "What's our roadmap for SPL generation?" — Start with fine-tuning on SPL query + intent pairs. Eval on execution accuracy. Add RL with correctness signal. Then agent that writes, tests, and iterates SPL.
5. "How do we handle prompt injection in a security log pipeline?" — Input validation, instruction hierarchy, sandboxed tool execution, and human approval for any actions that modify data or configurations.

---

*Lecture 12 of 12 — Stanford CME295 LLM Foundations | PM Reference*
