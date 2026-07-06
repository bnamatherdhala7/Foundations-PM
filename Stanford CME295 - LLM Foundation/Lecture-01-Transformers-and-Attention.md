# Lecture 01 — Transformers & Attention
## Stanford CME295 | PM Reference Notes

**One-line concept:** Transformers are the architecture that powers every major LLM. They replaced older sequence models by learning to pay "attention" to all parts of an input simultaneously rather than processing it word-by-word.

**Why it matters for PMs:** The architecture is not a detail — it is the capability ceiling. Understanding what transformers can and can't do tells you where model failures are architectural (hard to fix without retraining) vs. data-related (fixable with better training data) vs. post-training (fixable with fine-tuning or prompting).

---

## 1. What Is NLP — The Task Taxonomy

Before transformers, NLP tasks were categorized into three buckets. This taxonomy still structures how teams scope model work:

| Task Type | What It Does | Examples |
|---|---|---|
| **Classification** | Predicts a single label for the whole input | Sentiment analysis ("positive/negative"), intent detection ("is this a question or a command?") |
| **Multi-label / Span Labeling** | Labels specific components inside the input | Named Entity Recognition (NER) — identifying "Adobe" as a company, "San Jose" as a location |
| **Generation** | Produces new text from an input | Translation, summarization, question answering, code generation |

**PM takeaway:** When a team says "we're doing NER" or "this is a classification task," they're telling you the output structure — which tells you the labeling cost (classification labels are cheap, span labels are more expensive), the evaluation approach, and the likely failure modes.

---

## 2. Tokenization & Embeddings

### What Is Tokenization?

Models cannot read raw text. Text must be broken into **tokens** — discrete units the model processes. A token is roughly a word fragment (not a full word, not a single character — somewhere in between for most modern models).

**The three approaches and their tradeoffs:**

| Method | How It Works | Vocabulary Size | Strength | Weakness |
|---|---|---|---|---|
| **Word-level** | Split on spaces/punctuation | Very large (~50K+) | Simple | Fails on unseen words; no shared representation for "run" and "running" |
| **Subword-level** | Split into word roots and suffixes (e.g., "running" → "run" + "##ing") | Medium (~30K–50K) | Handles most languages; shares roots | Some languages tokenize inefficiently |
| **Character-level** | One token per character | Small (~256) | Handles any misspelling | Very long sequences; no semantic meaning per token |

**The standard today:** Byte Pair Encoding (BPE) and WordPiece — both subword methods. GPT models use BPE. BERT uses WordPiece. They're similar in practice.

### What Is a Token in Practice?

- English: ~1 token ≈ 0.75 words. "The quick brown fox" ≈ 5 tokens.
- Code: more tokens per line than English prose.
- Non-Latin scripts (Chinese, Arabic, Hindi): often 2–5× more tokens per word than English — directly impacts cost and quality for multilingual applications.

### What Are Embeddings?

After tokenization, each token is mapped to a **vector** — a list of ~768 to 4096 floating point numbers. This vector is the model's internal representation of that token's meaning. Related concepts have vectors that are geometrically close: "king" and "queen" are closer to each other than to "database."

**Word2Vec (historical context):** The 2013 model that proved vectors could encode semantics. Trained using a proxy task: predict the word that appears in context around a given word. The vectors it learned encoded relationships like king − man + woman ≈ queen.

Modern LLMs use **contextual embeddings** — the vector for a word changes depending on the surrounding sentence. "Bank" in "river bank" has a different embedding than "bank" in "bank account."

**PM takeaway on tokens:**
- Context window = measured in tokens, not words. A "32K context window" model can process roughly 24,000 words.
- API pricing = per token. Longer prompts cost more and add latency.
- Multilingual deployments: budget 2–3× more token usage for non-English languages, which means higher cost per query.

---

## 3. RNNs — Why We Needed Something Better

Before transformers (pre-2017), the dominant sequence model was the **Recurrent Neural Network (RNN)**.

### How RNNs Work
An RNN reads a sequence one token at a time, maintaining a "hidden state" — a compressed memory of everything seen so far. At each step: new token + current hidden state → new hidden state.

**The problem — Vanishing/Exploding Gradients:**
When training RNNs on long sequences, the gradient signal (the error used to update weights) either shrinks to near-zero (vanishing) or grows uncontrollably (exploding) as it flows back through many time steps. The result: the model can't learn long-range dependencies — it effectively forgets context from early in a sequence by the time it reaches the end.

**Practical consequence:** An RNN summarizing a 2,000-word document struggles to connect information in paragraph 1 to paragraph 20. The "memory" from paragraph 1 has been overwritten.

**LSTMs** (Long Short-Term Memory networks) added a more sophisticated gating mechanism to address this, but still processed sequentially — slow to train, limited in the length of dependencies they could model.

**PM takeaway:** RNNs were a sequential bottleneck — couldn't be parallelized across a sequence, and couldn't reliably use long context. The transformer solved both.

---

## 4. The Transformer Architecture

The 2017 paper "Attention Is All You Need" (Vaswani et al., Google Brain) introduced the transformer and replaced RNNs as the dominant architecture for sequence tasks.

### The Core Idea — Self-Attention

Instead of processing tokens one at a time, a transformer processes all tokens in a sequence **simultaneously** and lets each token attend to every other token.

**Analogy:** Imagine reading a sentence and, for each word, instantly knowing how relevant every other word in the sentence is to understanding that word's meaning. Self-attention does this mathematically — for each token, it computes a weighted sum over all other tokens, where the weights reflect relevance.

**The attention equation:**
```
Attention(Q, K, V) = softmax(QK^T / √d_k) × V
```
- **Q (Query):** "What am I looking for?"
- **K (Key):** "What do I contain?"
- **V (Value):** "What information do I provide?"

For every token, the model asks: "Given what I'm looking for (Q), how much should I attend to each other token (K), and what should I take from each (V)?"

You don't need to memorize the math — but you need to understand the intuition: **attention is a learned relevance score between every pair of tokens.**

### The Full Architecture

**Encoder:**
- Takes the input sequence
- Applies Multi-Head Self-Attention (see below)
- Applies a Feed-Forward Network (FFN) — a simple neural network applied to each token independently
- Produces context-aware representations of each token
- Used in: BERT, RoBERTa, models designed for understanding

**Decoder:**
- Uses **Masked Self-Attention** — each token can only attend to previous tokens (prevents "cheating" by looking at future tokens during generation)
- Uses **Cross-Attention** — attends to the encoder's output, allowing the decoder to reference the input
- Generates output tokens one at a time
- Used in: GPT models (decoder-only), T5 (encoder-decoder)

**Modern LLMs (GPT-4, Claude, Llama, Gemini):** Use decoder-only transformers. No encoder needed — the model both understands and generates using the same decoder stack.

### Multi-Head Attention

Instead of one attention mechanism, the transformer runs **multiple attention heads in parallel**, each with different learned weights. Each head learns different kinds of relationships:
- Head 1 might learn syntactic structure (subject-verb agreement)
- Head 2 might learn coreference (connecting "it" to the noun it refers to)
- Head 3 might learn semantic similarity

The outputs are concatenated and projected. The model learns what to look for; the engineer doesn't specify it.

**PM takeaway:** Multi-head attention is why transformers generalize — they simultaneously learn multiple types of linguistic and semantic relationships rather than forcing the model into one type of representation.

### Label Smoothing (Training Detail)

During training, instead of teaching the model that the correct answer has probability 1.0 and everything else has probability 0.0, label smoothing assigns probability 1 − ε to the correct answer and distributes ε across other tokens. This prevents the model from becoming overconfident and improves generalization.

**PM relevance:** Overconfident models produce worse-calibrated outputs — they assign high confidence to wrong answers. Label smoothing is one technique that improves calibration, which matters for applications where uncertainty should influence downstream decisions (e.g., medical, legal, financial applications).

---

## 5. Encoder-Only vs. Decoder-Only vs. Encoder-Decoder

| Type | Architecture | Best For | Examples |
|---|---|---|---|
| **Encoder-only** | Encoder stack | Understanding, classification, embeddings | BERT, RoBERTa |
| **Decoder-only** | Decoder stack | Generation, completion, chat, agents | GPT-4, Claude, Llama, Gemini |
| **Encoder-Decoder** | Both | Sequence-to-sequence (translation, summarization) | T5, BART |

Most frontier models (GPT-4, Claude, Gemini, Llama) are decoder-only. When someone says "LLM," they typically mean a large decoder-only transformer.

---

## Key Terms

| Term | Definition |
|---|---|
| **Transformer** | Neural network architecture based on self-attention; the foundation of all modern LLMs |
| **Self-Attention** | Mechanism allowing each token to attend to every other token in the sequence |
| **Multi-Head Attention** | Multiple parallel attention mechanisms learning different relationship types |
| **Tokenization** | Breaking raw text into tokens (sub-word units) that the model processes |
| **BPE** | Byte Pair Encoding — the most common subword tokenization method (used in GPT models) |
| **Embedding** | A vector representation of a token in high-dimensional space |
| **Contextual Embedding** | Embedding that changes based on surrounding context (vs. static Word2Vec-style) |
| **RNN** | Recurrent Neural Network — sequential predecessor to transformers; struggles with long-range dependencies |
| **LSTM** | Long Short-Term Memory — improved RNN with better gradient flow; still sequential |
| **Vanishing Gradient** | Training problem where gradient signal shrinks to near-zero over long sequences |
| **Encoder** | Transformer component that creates context-aware representations of input |
| **Decoder** | Transformer component that generates output tokens one at a time |
| **Masked Attention** | Attention where each token only sees previous tokens — used in decoders to prevent future-token leakage |
| **Cross-Attention** | Attention where the decoder attends to encoder outputs |
| **Label Smoothing** | Training technique that prevents overconfidence by softening target probability distributions |
| **NER** | Named Entity Recognition — identifying entities (people, places, organizations) in text |
| **Word2Vec** | Early embedding model (2013) proving vectors can encode semantic relationships |

---

## Product Questions This Unlocks

1. "When the model makes an error, is the failure architectural or is it a data/training issue?" — Architecture failures require retraining with different model choices; data failures can be fixed by improving training data.
2. "How does context window size affect our cost and latency at scale?" — Token counting lets you project API costs and latency budgets.
3. "Why is our model performing worse on Japanese than English?" — Tokenization efficiency: Japanese may use 3–4× the tokens per word.
4. "Why can't the model reliably reference something mentioned at the beginning of a very long document?" — Attention degrades at long context; this is an architectural and training constraint, not a bug.
5. "Should we use an encoder-only or decoder-only model for our use case?" — Classification/understanding → encoder; generation → decoder.

---

## Common PM Mistakes

- **"Context window = word limit"** — It's token limit. Assume 0.75 words per token for English as a rough conversion.
- **"Let's just make the model bigger"** — Bigger models are slower, more expensive to serve, and don't necessarily fix the failure mode you care about.
- **"Why can't we fix this in prompting?"** — Some failures are architectural (model literally cannot attend to context beyond a certain distance). Prompting can't fix architecture limits.
- **Treating all languages as equivalent** — Multilingual quality and cost varies significantly based on tokenization efficiency. Non-Latin scripts cost more and often perform worse on less-represented languages.

---

*Lecture 1 of 12 — Stanford CME295 LLM Foundations | PM Reference*
