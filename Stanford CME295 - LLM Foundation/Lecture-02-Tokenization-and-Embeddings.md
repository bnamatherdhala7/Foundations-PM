# Lecture 02 — Tokenization & Embeddings (Deep Dive)
## Stanford CME295 | PM Reference Notes

**One-line concept:** Tokenization is how raw text becomes numbers a model can process. The choice of tokenizer affects cost, quality, latency, and multilingual performance — all of which are PM-level concerns.

**Why it matters for PMs:** Every API call is billed in tokens. Every latency budget is measured in tokens. Every multilingual quality gap often traces back to tokenization. Understanding tokenization is understanding the cost and quality knobs for any LLM-based product.

---

## 1. Why Text Must Be Tokenized

Neural networks operate on numbers — specifically floating-point vectors. Raw text (Unicode characters, words, sentences) must be converted to a fixed numeric representation before the model can process it.

The tokenizer sits between raw text and the model:
```
Raw Text → Tokenizer → Token IDs → Embedding Layer → Vectors → Transformer
```

The tokenizer has two jobs:
1. **Encode:** Convert text to a sequence of integer IDs
2. **Decode:** Convert a sequence of integer IDs back to text (during generation)

---

## 2. Tokenization Methods

### Word-Level Tokenization
Split on spaces and punctuation. "Hello world!" → ["Hello", "world", "!"]

**Problems:**
- Vocabulary explodes: every inflection of a word ("run," "running," "ran," "runner") is a separate token
- Out-of-vocabulary (OOV) problem: any word not seen during training is unknown — common for rare words, names, technical jargon
- Doesn't generalize: the model learns nothing about the relationship between "run" and "runner"

### Character-Level Tokenization
One token per character. "Hello" → ["H", "e", "l", "l", "o"]

**Advantage:** Handles any word, any misspelling, any language — vocabulary is tiny (~256 UTF-8 characters)
**Problems:** Sequences are very long (every word = many tokens), making training slow and expensive. The model must learn to assemble meaning from individual characters.

### Subword-Level Tokenization (The Standard)
Break words into frequent subword units. "running" → ["run", "##ning"]. Common substrings across many words share tokens.

**Byte Pair Encoding (BPE)** — used by GPT models:
- Start with individual characters
- Iteratively merge the most frequent adjacent pair
- Repeat until vocabulary reaches target size (typically 30K–100K tokens)
- Result: common words are single tokens; rare words are split into subwords

**WordPiece** — used by BERT:
- Similar to BPE but uses a likelihood criterion instead of raw frequency for merges

**SentencePiece** — used by Llama, T5:
- Language-agnostic; operates on raw bytes rather than Unicode characters
- Handles languages without spaces between words (Japanese, Chinese) more gracefully

**PM comparison:**

| Tokenizer | Model | Vocab Size | Notes |
|---|---|---|---|
| BPE | GPT-2, GPT-3, GPT-4 | 50K (GPT-2), expanded in GPT-4 | Fast, widely adopted |
| WordPiece | BERT, RoBERTa | 30K | Slightly different merging criterion |
| SentencePiece (BPE) | Llama, T5, PaLM | 32K–64K | Language-agnostic; handles non-Latin scripts better |
| Tiktoken | OpenAI GPT-3.5+, GPT-4 | 100K | Faster, handles code and multilingual better than GPT-2 tokenizer |

---

## 3. The Tokenization Efficiency Problem — Why It's a PM Issue

### English vs. Non-English Token Efficiency

Tokenizers are trained on data that is overwhelmingly English (and code). This means English text tokenizes efficiently — common words become single tokens — but non-English text often tokenizes into many more tokens per word.

**Approximate tokens per word by language:**

| Language | Approx. Tokens per Word | Impact |
|---|---|---|
| English | ~1.3 | Baseline |
| French/Spanish/German | ~1.5–2.0 | Moderate overhead |
| Arabic | ~2.0–3.0 | Significant overhead |
| Hindi | ~2.5–4.0 | High overhead |
| Chinese | ~1.5–2.0 | Moderate (characters are often single tokens) |
| Japanese | ~2.0–3.5 | High overhead; mixed scripts |

**Product implication:**
- A 32K context window in English might only hold ~10K words of Hindi text
- API cost for multilingual deployments is 2–3× higher per user message in many languages
- Models trained with English-heavy tokenizers have worse quality on low-resource languages — not just because of training data, but because they represent those languages less efficiently

**Say in planning meetings:** *"If we're deploying this in Japanese, budget 2–3× the API cost per query and expect a quality gap vs. English until we evaluate on real Japanese examples."*

---

## 4. Context Window = Token Window

The context window of an LLM is measured in **tokens**, not words or characters.

| Model Generation | Context Window | Approx. English Words |
|---|---|---|
| GPT-3 | 4K tokens | ~3,000 words |
| GPT-3.5 | 16K tokens | ~12,000 words |
| GPT-4 | 8K–128K tokens | ~6,000–96,000 words |
| Claude 3 | 200K tokens | ~150,000 words |
| Gemini 1.5 | 1M tokens | ~750,000 words |

**Why context window matters for product:**
- Retrieval-Augmented Generation (RAG) design depends on context window size — larger windows allow more retrieved context per query
- Long document summarization, meeting transcripts, codebases: feasibility depends on whether the content fits in context
- Cost: models charge per input token — a large context window used fully is expensive per call

---

## 5. Embeddings — From Tokens to Meaning

After tokenization, each token ID is mapped to a dense vector — its **embedding**.

### Embedding Dimensions
- GPT-2 (small): 768 dimensions
- GPT-3: 12,288 dimensions
- Modern large models: 4,096–8,192+ dimensions

Each dimension is a learned feature. The embedding matrix is a table: row = token ID, column = feature. This matrix is learned during pretraining alongside the rest of the model.

### Static vs. Contextual Embeddings

**Static embeddings (Word2Vec, GloVe, FastText):**
- Each word has one fixed vector regardless of context
- "Bank" in "river bank" and "bank account" have the same vector
- Limitation: polysemy (words with multiple meanings) cannot be represented

**Contextual embeddings (Transformers):**
- Each token's vector depends on the full surrounding context
- "Bank" gets a different vector depending on whether the sentence is about water or finance
- This is what makes transformers powerful for complex language understanding

### The Embedding Space Has Structure

Famous example from Word2Vec: `king − man + woman ≈ queen`

This arithmetic works because the embedding space encodes semantic relationships as geometric directions. Transformers learn richer versions of these relationships.

**PM application — Embedding-based search:**
Semantic search products (RAG, document retrieval, recommendation) often work by comparing embeddings:
1. Embed query → vector
2. Embed all documents → vectors
3. Find documents with vectors closest to query vector (cosine similarity)
4. Return most similar documents

This is why "semantic search" understands synonyms and paraphrases — it's comparing meaning-vectors, not keyword strings.

---

## 6. Special Tokens

Every tokenizer includes special tokens that carry structural meaning:

| Token | Meaning | Example Use |
|---|---|---|
| `[BOS]` / `<s>` | Beginning of sequence | Marks the start of a document |
| `[EOS]` / `</s>` | End of sequence | Tells the model when to stop generating |
| `[PAD]` | Padding | Fills shorter sequences to match batch length |
| `[UNK]` | Unknown | Represents tokens not in vocabulary (rare in modern BPE) |
| `[SEP]` | Separator | Separates two segments (BERT-style) |
| `[CLS]` | Classification | BERT's "summary token" used for classification tasks |

---

## 7. Positional Encoding — How the Model Knows Order

Self-attention processes all tokens in parallel — it doesn't inherently know the order of tokens. To give the model position information, a **positional encoding** is added to each token's embedding.

**Original (sinusoidal):** Fixed mathematical function of position — position 1 gets a different encoding than position 500
**Learned positional embeddings:** The model learns position representations during training
**RoPE (Rotary Position Embedding):** Used by Llama, Falcon, and many modern models. Encodes relative position rather than absolute position — helps models generalize to longer sequences than they were trained on.

**PM implication:** RoPE-based models can often be extended to longer context windows more easily after training, which is why many open-source models have been "context-extended" after release.

---

## Key Terms

| Term | Definition |
|---|---|
| **Token** | The basic unit of text the model processes; roughly a word fragment |
| **Tokenizer** | Converts raw text ↔ token IDs |
| **BPE** | Byte Pair Encoding — iterative subword tokenization used in GPT models |
| **WordPiece** | Subword tokenization used in BERT; similar to BPE with different merging criterion |
| **SentencePiece** | Language-agnostic subword tokenizer used in Llama, T5 |
| **Tiktoken** | OpenAI's fast tokenizer for GPT-3.5+ and GPT-4 |
| **Embedding** | Dense vector representation of a token |
| **Contextual Embedding** | Token embedding that changes based on surrounding context |
| **Context Window** | Maximum number of tokens a model can process at once |
| **Word2Vec** | 2013 model demonstrating semantic vector arithmetic |
| **Positional Encoding** | Added to token embeddings to give the model sense of token order |
| **RoPE** | Rotary Position Embedding — relative position encoding; supports context extension |
| **OOV** | Out-of-Vocabulary — words not in the tokenizer's vocabulary (rare in BPE) |
| **RAG** | Retrieval-Augmented Generation — augmenting prompts with retrieved document embeddings |
| **Cosine Similarity** | Similarity metric between two vectors; used in semantic search |

---

## Product Questions This Unlocks

1. "What's the per-query cost for our multilingual feature in Japanese?" — Calculate avg. token count in Japanese × cost per token × expected query volume.
2. "Why does our context window feel smaller than advertised?" — Users may be sending non-English text, code, or formatted text that tokenizes less efficiently.
3. "Can we use semantic search here instead of keyword search?" — Yes if you have embeddings; semantic search is better for paraphrase matching, worse for exact-match lookup.
4. "Why does the model confuse two different meanings of the same word?" — Likely a static embedding artifact; contextual embeddings should handle this, but may fail if context is insufficient.
5. "Should we fine-tune embeddings for our domain?" — Domain-specific fine-tuning of embeddings often significantly improves retrieval quality in specialized fields (legal, medical, code).

---

## Common PM Mistakes

- **"Our 32K context window gives us 32K words"** — ~24K words in English; less in other languages. Always convert tokens to words for user-facing estimates.
- **"Semantic search is always better than keyword search"** — Not true. Semantic search excels at paraphrase matching; keyword search excels at exact recall of specific terms. Hybrid is usually best.
- **"All models have the same tokenizer"** — No. You cannot transfer token counts across models. GPT-4 and Claude use different tokenizers; the same text may produce different token counts.
- **"We can expand the context window after training with no quality loss"** — Context extension (via RoPE scaling, etc.) works up to a point; models often degrade in quality at the very edges of extended context windows.

---

*Lecture 2 of 12 — Stanford CME295 LLM Foundations | PM Reference*
