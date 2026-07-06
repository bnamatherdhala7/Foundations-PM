# Lecture 03 — Pretraining & Data Pipeline
## Stanford CME295 | PM Reference Notes

**One-line concept:** Pretraining is where the model learns everything it knows — by predicting the next token across hundreds of billions of words of text. The data pipeline that feeds pretraining determines the model's capability ceiling.

**Why it matters for PMs:** Training data decisions made here determine what the model can and cannot do. A model cannot reliably do things that are absent or underrepresented in its training data — no amount of prompting or fine-tuning will fully compensate. Data strategy IS model strategy.

---

## 1. What Is Pretraining?

Pretraining is the initial, large-scale training run where the model learns from a massive corpus of text using a **self-supervised objective** — no human labels required.

### The Pretraining Objective: Next-Token Prediction

The model is trained to predict the next token given all previous tokens. Given the input "The capital of France is", the model should predict "Paris."

Formally: maximize P(token_t | token_1, token_2, ..., token_{t-1}) across all positions in all documents.

**Why this is powerful:**
- To predict text well, the model must learn grammar, facts, reasoning, style, and language structure
- All of this emerges from a single objective without any task-specific labeling
- Scale: train on enough text, with a large enough model, and the model generalizes to tasks it was never explicitly trained for

**The emergent capabilities surprise:** Capabilities like arithmetic, code generation, few-shot learning, and logical reasoning were not explicitly trained — they emerged from scale. This was unexpected and remains partially unexplained (the subject of active research).

---

## 2. Pretraining Data Sources

Modern LLM training corpora are assembled from multiple sources, each with different quality and coverage characteristics:

| Source | Examples | Quality | Coverage |
|---|---|---|---|
| **Web crawl** | Common Crawl, C4, RefinedWeb | Variable — noisy, contains spam/errors | Broadest coverage of internet text |
| **Books** | BookCorpus, Project Gutenberg, Books3 | High — edited, coherent, long-form | Literature, nonfiction, fiction |
| **Academic papers** | arXiv, PubMed, Semantic Scholar | High — rigorous, domain-specific | Science, medicine, math |
| **Code** | GitHub, StackOverflow, CodeSearchNet | High — structured, logic-heavy | Code generation, reasoning |
| **Wikipedia** | English + multilingual Wikipedia | High — encyclopedic, factual | Facts, structured knowledge |
| **Curated datasets** | OpenWebText, Pile, RedPajama | Medium-high — human-filtered | Varies |
| **Synthetic data** | Model-generated text | Variable — depends on generator | Used to fill gaps in rare domains |

**Typical mix for a frontier model (approximate):**
- Web crawl: 40–60% of tokens
- Books: 10–20%
- Code: 10–15%
- Wikipedia: 5–10%
- Other (academic, curated): 10–20%

The exact mix is typically proprietary. The mix determines what the model knows.

---

## 3. The Data Pipeline — From Raw Web to Training-Ready Tokens

Raw web data is unusable for training. It must go through multiple quality filtering stages:

```
Raw Web Crawl (petabytes)
        ↓
  1. URL/Domain Filtering    → Remove known spam, adult, and hate sites
        ↓
  2. Language Identification → Keep target languages; discard or separate others
        ↓
  3. Deduplication           → Remove exact and near-duplicate documents
        ↓
  4. Quality Filtering       → Remove low-quality text (boilerplate, SEO spam, incoherent text)
        ↓
  5. Content Filtering       → Remove CSAM, illegal content, dangerous instructions
        ↓
  6. PII Scrubbing           → Remove or redact personally identifiable information
        ↓
  7. Tokenization            → Convert filtered text to token sequences
        ↓
  8. Shuffling & Batching    → Randomize sequence order for training
        ↓
Training-Ready Dataset
```

### 3.1 Deduplication — Why It Matters

A significant fraction of web crawl data is duplicated (same article on 50 different sites, forum posts quoted verbatim, etc.). Training on duplicated data:
- Wastes compute (model sees the same examples repeatedly)
- Causes the model to memorize specific text (privacy risk and overfitting)
- Skews quality — viral but low-quality content that propagates widely gets overrepresented

**Deduplication methods:**
- **Exact deduplication:** Hash-based matching (fast, misses paraphrases)
- **Near-deduplication (MinHash/LSH):** Approximates similarity using hashing; catches paraphrased duplicates
- **Semantic deduplication:** Embedding-based; most thorough but computationally expensive

### 3.2 Quality Filtering Methods

| Method | How It Works | Tradeoff |
|---|---|---|
| **Heuristic filters** | Remove documents with too many special characters, wrong language, too short/long | Fast; may remove valid edge cases |
| **Perplexity filtering** | Use a small LM to score text; remove text that is hard to predict (garbled) | Biases toward standard language patterns |
| **Classifier-based** | Train a classifier on high-quality vs. low-quality text; filter by score | More accurate; requires labeled training data for the classifier |
| **URL/domain allowlist** | Manually curated list of high-quality domains | High precision; low recall (misses many good sources) |

**The quality-quantity tradeoff:**
More aggressive filtering → higher quality but less data → potentially worse coverage of long-tail domains.
Less aggressive filtering → more data but noisier → model may learn bad patterns.

**Chinchilla (2022) showed:** For a given compute budget, training on more, higher-quality tokens outperforms training on fewer tokens with a larger model. Data quality and quantity matter as much as model size.

### 3.3 Data Mixture and Resampling

Not all data sources are equally represented in the raw crawl. To get the model to perform well on code, for example, code data may need to be **upsampled** (shown to the model more frequently than its natural frequency in the corpus).

**Data mixture is a hyperparameter.** The ratio of web:books:code:Wikipedia affects:
- Model performance on different task types
- The model's "personality" (more web-like = more conversational; more books = more formal)
- Multilingual ability

**Epochs:** Most frontier models train on each token approximately once (1 epoch). Some datasets are repeated if the model hasn't seen enough of them.

---

## 4. Training Data and Model Capabilities — The Direct Link

**A model can only reliably do what was in the training data.**

| Capability | Required Training Data |
|---|---|
| Code generation | Large code corpus (GitHub-scale) |
| Medical question answering | Medical literature, clinical notes |
| Multilingual performance | Representative text in each language |
| Formal reasoning / math | Math textbooks, proofs, competition problems |
| Current events knowledge | Depends on training cutoff date |
| Legal reasoning | Legal documents, case law |

**The training data cutoff:**
Models have a knowledge cutoff — they don't know about events after their training data was collected. This is often a product limitation: users ask about recent events, and the model either doesn't know or confabulates.

**PM takeaway:** If your product requires up-to-date knowledge, the model must be paired with retrieval (RAG) or fine-tuned on recent data. The base model cannot be updated in real-time.

---

## 5. Data Contamination — A Critical Quality Issue

**Definition:** Test benchmark questions or answers appearing in the training data. If the model was trained on data that includes answers to the benchmarks it's evaluated on, the evaluation is invalid.

**Why it happens:** Web crawls contain everything published online — including benchmark datasets, academic papers that describe them, and forums discussing the answers.

**Why it matters for PMs:**
- Inflated benchmark scores may not reflect real capability
- A model may score well on MMLU but fail on a clean internal benchmark
- When evaluating vendor models, ask: "What is your contamination detection process?"

---

## 6. Synthetic Data — The New Frontier

As high-quality human-generated text reaches scale limits, teams increasingly use **model-generated synthetic data** to supplement training:

**Use cases:**
- Generating diverse examples of rare tasks
- Creating adversarial examples for robustness
- Generating multilingual data for underrepresented languages
- Generating code examples in less common programming languages

**The risk — Model Collapse:**
If a model is trained on data generated by another model (or itself in a loop), and that pattern is repeated:
- The model's output distribution narrows over iterations
- Rare or diverse outputs are progressively lost
- Quality degrades toward a "generic average"

**The solution:** Always mix synthetic data with real human-generated data; maintain diversity metrics; monitor output distribution over training iterations.

**PM takeaway on synthetic data:** Synthetic data is a valid tool for filling gaps, but it must be used carefully. A training corpus that is predominantly synthetic will produce a worse model than one rooted in high-quality human-generated text. Track the real-to-synthetic ratio as a quality metric.

---

## 7. IP, Licensing, and Data Ethics

The training data question is increasingly a legal and reputational question, not just a quality question.

**The core tension:**
- Scraping web data is technically legal in many jurisdictions but has been contested in court (NY Times v. OpenAI, Getty Images v. Stability AI)
- Many creators and publishers argue that training on their work without consent or compensation is a form of copying
- Enterprise customers increasingly ask: "Can we be sued for using your model's outputs?"

**Three sourcing postures:**

| Posture | Description | Example |
|---|---|---|
| **License-based** | Only use data from licensed sources | Adobe Firefly (Stock), some medical AI |
| **Policy-based (opt-out)** | Train on everything, remove on request | Google (post-2023), OpenAI |
| **Rights-reserved** | Default web crawl; contest claims in court | Many open-source models |

**IP indemnification:**
Some vendors (Adobe, Google Vertex AI) offer legal indemnification — if you're sued for copyright infringement due to a model's output, they'll cover legal costs. This is a significant enterprise differentiator.

**PM vocabulary:**
- **Provenance-first:** Training data lineage documented from source → the sourcing decision was made before data was collected
- **Policy-first:** Training data sourcing is defended by legal policy → the legal defense is retroactive

---

## Key Terms

| Term | Definition |
|---|---|
| **Pretraining** | Initial large-scale training on unlabeled text using next-token prediction |
| **Self-supervised learning** | Training using labels derived from the data itself (next token), no human annotation |
| **Common Crawl** | Large-scale web crawl dataset; the base of most LLM training corpora |
| **Deduplication** | Removing duplicate or near-duplicate documents from training data |
| **MinHash/LSH** | Efficient algorithms for near-duplicate detection |
| **Perplexity filtering** | Filtering low-quality text by how unpredictable it is to a reference model |
| **Data mixture** | The ratio of different data sources in the training corpus |
| **Upsampling** | Training on a data source more frequently than its natural representation |
| **Knowledge cutoff** | Date after which the model has no knowledge from training data |
| **Data contamination** | Benchmark test data appearing in training data, inflating eval scores |
| **Model collapse** | Quality degradation from training on predominantly model-generated data |
| **Synthetic data** | Model-generated text used to supplement training |
| **RAG** | Retrieval-Augmented Generation — supplying retrieved documents to extend model knowledge beyond cutoff |
| **PII** | Personally Identifiable Information — must be removed from training data |
| **CSAM** | Child Sexual Abuse Material — zero-tolerance filter at data ingestion |
| **IP indemnification** | Vendor's legal guarantee to defend customers against copyright claims |

---

## Product Questions This Unlocks

1. "Why doesn't the model know about [recent event]?" — Training data cutoff. Solution: RAG or periodic fine-tuning on recent data.
2. "Can we trust this benchmark score?" — Ask about contamination detection methodology.
3. "How do we improve the model's performance on [specific domain]?" — Domain-specific data acquisition and upsampling in the training mix.
4. "What's our legal exposure for outputs?" — Depends on training data provenance and whether vendor offers IP indemnification.
5. "How long will it take to improve multilingual quality?" — Requires acquiring and processing more diverse multilingual data, tokenizer evaluation, and a training run — likely months, not weeks.
6. "Can we add synthetic data to improve edge case coverage?" — Yes, but track real-to-synthetic ratio; maintain diversity gates.

---

## Common PM Mistakes

- **"We can just prompt the model to know about this domain"** — Prompting cannot add knowledge not present in training data. It can only surface and recombine existing knowledge.
- **"Bigger training dataset = better model"** — Quality filters that remove duplicates and noise matter as much as raw size.
- **"Our benchmark scores prove model quality"** — Contaminated benchmarks inflate scores. Always validate on an internal, held-out evaluation set.
- **"We'll update the model with new data monthly"** — Full pretraining reruns are extremely expensive. Updates require either periodic fine-tuning (limited) or RAG (for factual recency).

---

*Lecture 3 of 12 — Stanford CME295 LLM Foundations | PM Reference*
