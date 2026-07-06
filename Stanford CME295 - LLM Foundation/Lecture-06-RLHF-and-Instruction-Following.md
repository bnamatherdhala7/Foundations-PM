# Lecture 06 — RLHF & Instruction Following
## Stanford CME295 | PM Reference Notes

**One-line concept:** After pretraining, a model knows a lot but doesn't know how to be helpful. Instruction tuning and RLHF (Reinforcement Learning from Human Feedback) are the steps that teach the model to follow instructions, be helpful, be honest, and be safe. This is where product quality is built on top of raw capability.

**Why it matters for PMs:** A pretrained model is not a product. The gap between "has capability" and "actually useful for users" is closed by this stage. Understanding it tells you what's fixable through post-training vs. what requires retraining, and why different model versions behave differently.

---

## 1. The Problem: Pretrained Models Are Not Assistant Models

A pretrained language model has one skill: predict the next token. Given "The capital of France is", it generates "Paris" — great. But given "Can you help me write an email to my manager?", a raw pretrained model might:
- Continue the sentence as if it's an internet document
- Generate a Wikipedia-style article about email etiquette
- Produce training data artifacts

The model has the underlying knowledge to write the email. It doesn't know it's supposed to be a helpful assistant.

**The alignment gap:** The gap between raw model capability and useful product behavior. Closing this gap is the job of instruction tuning and RLHF.

---

## 2. Stage 1: Supervised Fine-Tuning (SFT)

**What it is:** Fine-tune the pretrained model on a dataset of (instruction, response) pairs demonstrating desired behavior.

**Example training pair:**
```
Instruction: "Write a professional email declining a meeting invitation."
Response: "Dear [Name], Thank you for the invitation to meet on [date]. 
Unfortunately, I have a prior commitment at that time and will not be 
able to attend. I would welcome the opportunity to connect at a later 
date. Best regards, [Your Name]"
```

**How SFT data is collected:**
- Human contractors or employees write high-quality responses to diverse instructions
- Teams use data vendors (Scale AI, Surge AI) for volume
- Quality is critical — SFT data quality directly determines post-training model behavior
- Typical dataset size: 10K–1M examples (much smaller than pretraining)

**What SFT teaches the model:**
- Response format (how to structure an answer)
- Tone (helpful, professional, concise)
- Following multi-step instructions
- Task coverage (the range of tasks the model should handle)

**Limitation of SFT alone:** Human-written examples are expensive to produce at scale. The model learns from demonstrations but can't yet generalize well to instruction types not in the SFT dataset. Also: it's hard for humans to write examples of highly-nuanced ideal behavior (e.g., the perfectly calibrated refusal of a borderline request).

---

## 3. Stage 2: Reward Model Training

**What it is:** Train a separate model that predicts human preference — given two responses to the same instruction, which is better?

**Data collection:**
- Show human raters 2–4 model completions for the same prompt
- Raters rank them: "Response A is better than Response B"
- These pairwise preferences are used to train a reward model (RM)

**The reward model:**
- Takes (instruction, response) as input
- Outputs a scalar score (higher = better response)
- Trained to predict which response a human would prefer
- Usually built on the same pretrained model backbone as the main model

**Why pairwise ranking instead of absolute scores?**
- Humans are much better at comparative judgments ("A is better than B") than absolute quality ratings ("rate this response 1–10")
- Pairwise rankings are more consistent and less noisy across raters
- Much more efficient — one human judgment captures relative quality of multiple responses

---

## 4. Stage 3: Reinforcement Learning (PPO)

**What it is:** Use the reward model to give the LLM feedback on its own responses, and update the LLM to maximize reward.

**The RL loop:**
```
1. Sample a prompt from a dataset
2. LLM generates a response
3. Reward model scores the response
4. PPO algorithm updates LLM weights to increase expected reward
5. Repeat millions of times
```

**PPO (Proximal Policy Optimization):** The RL algorithm used. It updates the model but constrains how much it changes per step (the "proximal" part) — preventing the model from changing so dramatically that it "forgets" its pretraining knowledge.

**The KL constraint:** During PPO, a penalty term keeps the fine-tuned model from drifting too far from the original SFT model. Without this, the model might learn to game the reward model in nonsensical ways — producing high reward scores but terrible actual quality (reward hacking).

**What RLHF teaches:**
- Nuanced helpfulness that's hard to write down in SFT data
- Better calibration (knowing when to express uncertainty)
- Safer refusals (declining harmful requests while remaining helpful for legitimate ones)
- Stylistic preferences (conciseness, tone, format)

---

## 5. The Full Pipeline: SFT → RM → PPO

```
Pretrained LLM
      ↓
  Stage 1: SFT
  (Instruction-response demos)
      ↓
  SFT Model
      ↓
  Stage 2: Reward Model Training
  (Pairwise human preferences)
      ↓
  Reward Model (RM)
      ↓
  Stage 3: PPO Fine-tuning
  (LLM generates → RM scores → PPO updates LLM)
      ↓
  RLHF-Aligned Model
  (ChatGPT, Claude, Gemini, etc.)
```

---

## 6. RLAIF — Reinforcement Learning from AI Feedback

A variant of RLHF where an AI model (instead of humans) generates the preference judgments.

**Why RLAIF:**
- Human feedback is expensive and slow
- At scale (billions of comparisons), human labeling is infeasible
- AI raters, when well-calibrated, can be faster and cheaper

**How it works:**
- A "critic" model (often a larger, more capable model) evaluates responses
- The critic's preferences are used to train the reward model in place of human raters

**Limitations:**
- The critic model's biases are inherited by the trained model
- AI feedback misses some nuances humans catch (cultural context, common sense)
- Not appropriate for safety-critical judgments (AI should not rate whether a response is dangerous)

**Anthropic's Constitutional AI (CAI):** An RLAIF approach where the AI is given a set of principles ("the constitution") and generates its own critique and revision of responses, eliminating the need for human-labeled preference data for many helpfulness and safety behaviors.

---

## 7. DPO — Direct Preference Optimization

**What it is:** A newer, simpler alternative to RLHF PPO that achieves similar results without the separate reward model and RL loop.

**How it works:**
- Take the same human preference data (pairwise comparisons)
- Directly optimize the LLM to assign higher probability to preferred responses and lower probability to dispreferred ones
- No separate reward model; no RL; just supervised learning on the preference data

**Advantages over PPO:**
- Simpler to implement and tune (no reward model training, no RL infrastructure)
- More stable training (no reward hacking, no KL constraint needed)
- Competitive quality on most benchmarks

**Disadvantage:**
- Less flexible — can't incorporate real-time feedback or online learning as easily
- Some evidence that PPO may achieve higher quality ceilings for complex reasoning tasks

**PM relevance:** DPO has democratized alignment — smaller teams without RL infrastructure can now align models to human preferences. This is why many open-source models (Mistral, Llama fine-tunes) are well-aligned without requiring massive RLHF pipelines.

---

## 8. Instruction Tuning vs. RLHF — What Each Fixes

| Approach | What It Teaches | What It Doesn't Fix |
|---|---|---|
| **SFT (Instruction Tuning)** | Format, tone, task coverage | Nuanced quality, edge case handling, calibration |
| **RLHF (RM + PPO)** | Nuanced preferences, safety, calibration | Factual accuracy, capabilities not in pretraining |
| **DPO** | Similar to RLHF, simpler process | Same limitations as RLHF |
| **RLAIF / CAI** | Scalable alignment, safety rules | Human-specific cultural nuances |

**Key insight:** None of these add new knowledge. They shape how the model uses what it already knows. If the pretrained model doesn't know a fact, RLHF cannot teach it that fact — it can only shape how the model responds when it doesn't know.

---

## 9. The Helpfulness-Safety-Honesty Tradeoff

The three desired properties often pull in different directions:

- **Helpfulness:** Give the user what they need
- **Safety:** Don't produce harmful outputs
- **Honesty:** Don't confabulate; express appropriate uncertainty

**The tradeoff examples:**
- A model optimized too heavily for safety becomes unhelpful (refuses legitimate requests)
- A model optimized too heavily for helpfulness may produce harmful outputs
- A model penalized for expressing uncertainty may confabulate confidently

**Anthropic's HHH framework:** Helpful, Harmless, Honest. These three are in tension, and the reward modeling process must carefully balance them.

**PM relevance:** When users complain about the model being "too cautious" or "over-refusing," that's the safety-helpfulness tradeoff set too conservatively. When users get harmful outputs, the balance is set too permissively. Adjusting this balance is an ongoing product decision — not a one-time training choice.

---

## 10. Hallucinations — Why RLHF Doesn't Fully Solve It

**What is hallucination?** The model confidently generates factually incorrect information — a product of the next-token prediction objective. The model is trained to produce fluent, plausible continuations, not factually verified ones.

**Why RLHF helps partially:**
- Human raters penalize confident incorrect statements
- The model learns to express uncertainty on topics it's uncertain about

**Why RLHF doesn't fully fix it:**
- Human raters can't verify every factual claim
- Reward models learn rater preferences, not factual accuracy
- Models may learn to sound more uncertain without becoming more accurate

**Current best mitigation:** Retrieval-Augmented Generation (RAG) — ground model responses in retrieved, verified documents. Not an architecture fix; a product architecture fix.

---

## Key Terms

| Term | Definition |
|---|---|
| **SFT** | Supervised Fine-Tuning — fine-tuning on (instruction, response) demonstration pairs |
| **RLHF** | Reinforcement Learning from Human Feedback — the full SFT → RM → PPO pipeline |
| **Reward Model (RM)** | Trained model that predicts human preference scores for responses |
| **PPO** | Proximal Policy Optimization — the RL algorithm used to update the LLM using RM scores |
| **KL divergence** | Measure of how much the fine-tuned model has drifted from the SFT model; used as a penalty |
| **RLAIF** | Reinforcement Learning from AI Feedback — AI model generates preference labels |
| **Constitutional AI (CAI)** | Anthropic's technique: AI critiques and revises its own responses using a rule set |
| **DPO** | Direct Preference Optimization — simplified alignment without RM or RL |
| **Pairwise comparison** | Rater selects preferred response from two options; used for RM training |
| **Reward hacking** | Model learns to maximize reward score in ways that don't correspond to actual quality |
| **Hallucination** | Model generating factually incorrect information with apparent confidence |
| **HHH** | Helpful, Harmless, Honest — Anthropic's alignment framework |
| **Alignment** | The degree to which a model's behavior matches intended human values and instructions |
| **Alignment gap** | The gap between raw model capability and useful, safe, aligned behavior |
| **RAG** | Retrieval-Augmented Generation — grounding responses in retrieved documents to reduce hallucination |

---

## Product Questions This Unlocks

1. "Why does the model refuse legitimate requests?" — Safety-helpfulness balance set too conservatively in RM training; can be adjusted via system prompt tuning or model fine-tuning.
2. "Why does the model hallucinate on [specific domain]?" — Insufficient coverage in pretraining data + RM raters unable to verify domain-specific facts. Mitigation: RAG.
3. "How do we improve the model's tone for enterprise users?" — SFT on enterprise-appropriate examples + RM trained on enterprise rater preferences.
4. "Can we fine-tune an open-source model to be better aligned with our use case?" — Yes, via SFT + DPO on domain-specific preference data. DPO makes this accessible without RL infrastructure.
5. "Why did the model's behavior change between versions?" — RLHF data, RM calibration, or PPO hyperparameters were changed. Different fine-tuning mix = different behavioral tendencies.

---

## Common PM Mistakes

- **"The model knows the answer but won't give it"** — May be safety over-refusal, or the model genuinely doesn't know and is confabulating rather than declining. These require different fixes.
- **"RLHF makes the model smarter"** — RLHF doesn't add new capabilities; it shapes how existing capabilities are expressed. A model can't learn new facts through RLHF.
- **"We can tune away all hallucinations"** — Hallucination is a fundamental property of next-token prediction. RLHF reduces it; architecture + retrieval is needed to get close to eliminating it.
- **"Instruction tuning is fine-tuning"** — SFT is a type of fine-tuning, but "fine-tuning" is broader (includes domain-specific fine-tuning, parameter-efficient fine-tuning like LoRA, etc.).

---

*Lecture 6 of 12 — Stanford CME295 LLM Foundations | PM Reference*
