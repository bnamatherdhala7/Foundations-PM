# Google AI PM Interview Prep Plan (Tulsee Doshi's Team)

A prep plan built from the six attributes the team looks for. Each attribute has a goal, daily/weekly practice, a concrete deliverable, and a way to talk about it in the interview.

Timestamps (e.g. 21:56-22:04) refer to the source interview video.

## Overview

| # | Attribute | Core idea | Deliverable |
|---|-----------|-----------|-------------|
| 1 | Model intuition ("vibes") | Power user with a personal opinion on how models feel | Vibe benchmark + model opinion notes |
| 2 | Technical scrappiness | Roll up sleeves: data, manual evals, limits | One hands-on data/eval teardown |
| 3 | Eval proficiency | Small, well-chosen evals unlock capabilities | A 5-20 prompt eval set with rubric |
| 4 | Adaptability | Landscape changes every ~2 weeks | A roadmap re-plan story + "model swap" design |
| 5 | Product taste & insight | When building is trivial, taste is the value | 3 "worth building / not worth building" calls |
| 6 | Build for the future | 60-20-20 framework | A 60-20-20 plan for a product you pick |

## Suggested timeline (4 weeks)

| Week | Focus | Attributes |
|------|-------|------------|
| 1 | Build the habit and the baseline | 1, 3 |
| 2 | Go deep technically | 2, 3 |
| 3 | Strategy and judgment | 4, 5, 6 |
| 4 | Synthesis and mock interviews | All |

Adjust the pace if the interview date is closer. Weeks 1 and 3 are the highest leverage.

---

## 1. Develop model intuition ("vibes")

**Why it matters:** You must be a power user who plays with AI models daily (21:56-22:04). Benchmarks are not enough; the team wants a nuanced, personal opinion on how a model feels, its personality, and where its "magic" lies (0:09-0:14, 4:58-5:18).

**Daily practice (30-45 min)**
- Use Gemini, Claude, and GPT on real tasks, not toy prompts.
- Run the same prompt on 2-3 models and note differences in tone, reasoning, formatting, refusals, and failure style.
- Keep a running log: date, model, prompt, what felt magical, what felt off.

**Weekly practice**
- Write a half-page "model personality" note per model: strengths, quirks, when you would pick it.
- When a new model or version ships, run your vibe benchmark within 24 hours and write down what changed.

**Deliverable**
- A personal vibe benchmark of 5-10 multi-step prompts, from a simple creative task to an agentic workflow.
- A notes file with your honest opinion on each model.

**How to talk about it**
- Lead with a specific observation ("On long-context summarization, X hedges more than Y, which hurts in Z use case"), not "I use AI a lot."
- Have one strong opinion about where Gemini's magic is and one thing you would fix.

**Self-check:** Can you describe a model's personality in three sentences, with an example prompt for each claim?

---

## 2. Technical scrappiness

**Why it matters:** Be willing to roll up your sleeves: dive into data, run manual evaluations, and understand a model's technical limits (22:09-22:35).

**Practice**
- Pick a public dataset or your own prompt/response logs. Read at least 100 examples by hand and tag failure modes.
- Learn the limits that shape product decisions: context window, latency, cost per token, hallucination, tool-call reliability, non-determinism.
- Build one small prototype (a script, notebook, or app) that calls a model API and measures something.
- Be able to explain, at a PM level, how training, fine-tuning, RAG, and prompting differ and when each is the right lever.

**Deliverable**
- A one-page teardown: what you looked at, the failure-mode taxonomy you found, counts per category, and what you would do about the top one.

**How to talk about it**
- Tell a story with numbers: "I read 120 outputs, 35% of failures were X, so I changed Y."

**Self-check:** Could you do the analysis yourself without waiting for an analyst or an engineer?

---

## 3. Eval proficiency

**Why it matters:** Start small. Five to 20 well-chosen prompts can be an "unlock" for stress-testing a specific capability (23:04-24:05).

**Practice**
- Choose one capability (for example: multi-step instruction following, long-document Q&A, code editing, tool use).
- Write 5-20 prompts that cover easy, typical, and edge cases. Include some the current models fail.
- Write a rubric for what "good" and "bad" look like before you run anything. Score by hand.
- Run it across at least two models. Record wins, losses, and why.
- Iterate: add prompts for each new failure mode you discover.

**Deliverable**
- An eval set (prompts + rubric + scored results) in a doc or sheet, with a short readout: what it showed, what you would ship or block on.

**Concepts to be fluent in**
- Golden sets vs. live traffic samples
- Human vs. auto-rater (LLM-as-judge) tradeoffs and their biases
- Offline eval vs. online metrics (A/B, side-by-side preference)
- Eval contamination and overfitting to your own eval
- "Evals as the new PRD": how a rubric replaces a spec for non-deterministic behavior

**Self-check:** Given a vague goal like "make the model better at planning," can you turn it into an eval in under an hour?

---

## 4. Adaptability

**Why it matters:** The landscape moves roughly every two weeks. Show that you can re-evaluate a roadmap and avoid rigid systems that break when a better model arrives (22:43-23:05, 35:49-36:42).

**Practice**
- Take a product you know and write down which parts of its design assume today's model limits (prompt chains, heavy scaffolding, hardcoded routing).
- For each, write what you would delete or simplify if the model got 2x better.
- Follow release notes and research digests weekly; write a 3-bullet "what this changes for product" note.

**Deliverable**
- A "model swap" design note: how you would structure a product so a better model drops in with minimal rework (clean eval harness, thin scaffolding, model-agnostic interfaces).
- One real or realistic story of changing course quickly when new information arrived.

**How to talk about it**
- Describe a time you killed or reshaped work because the ground moved, and what signal triggered it.
- Contrast with the failure mode: scaffolding that was clever for last year's model and now holds the product back.

**Self-check:** Which parts of your last roadmap would be wasted if the model improved next month?

---

## 5. Product taste & insight

**Why it matters:** When building becomes trivial, your value shifts to taste: separating signal from noise and deciding exactly what is worth building to create a magical experience (20:13-20:56).

**Practice**
- Each week, use 2-3 AI products critically. For each, write: what is the magic moment, what is friction, what would you cut, what would you add.
- Practice "abstract the request": take a feature ask and restate it as the underlying model capability and user need.
- Make explicit "do not build" calls and defend them.

**Deliverable**
- Three short product critiques (one paragraph each) with a clear recommendation and the evidence behind it.
- Three examples of "worth building / not worth building" with reasoning.

**How to talk about it**
- State a point of view first, then the evidence. Avoid listing features; name the one experience that matters and why.

**Self-check:** Can you name the single moment in a product that makes people say "wow," and what would break it?

---

## 6. Build for the future (60-20-20)

**Why it matters:** Ensure the core product works today (60%), while dedicating energy to features that are almost working or show glimmers of light (20%), so the product evolves as model capabilities mature (33:13-34:43). The remaining 20% is the further-out bet.

**Practice**
- Pick a product (a Google product or your own project) and fill in a 60-20-20 plan:
  - **60%:** what must work reliably today, with success metrics.
  - **20%:** what almost works now; the eval that tells you when it crosses the line.
  - **20%:** what is ambitious and not yet feasible; the capability signal you are waiting for.
- For the middle bucket, write the trigger: "when the model reaches X on eval Y, we ship Z."

**Deliverable**
- A one-page 60-20-20 plan with named evals attached to the 20% items.

**How to talk about it**
- If everything you ship works 100% today, you aimed too low. If nothing works, it is science fiction. Show where you sit between those and how you know.

**Self-check:** For each "almost working" feature, what exact eval result flips it from prototype to launch?

---

## Weekly schedule template

| Day | Activity (about 60 min) |
|-----|-------------------------|
| Mon | Model play + log entries (attribute 1) |
| Tue | Eval work: write or extend prompts and rubric (attribute 3) |
| Wed | Technical deep-dive: data reading or API prototype (attribute 2) |
| Thu | Product critique + abstract-the-request drill (attribute 5) |
| Fri | Roadmap/60-20-20 thinking + weekly "what changed" note (attributes 4, 6) |
| Weekend | Mock interview or story practice (see below) |

## Story bank to prepare

Prepare 6-8 stories, each tagged to the attributes above. Use Situation, Action, Result, and a lesson.

- A time you used hands-on data work to change a decision
- A time you built or ran an eval, and what it changed
- A time you changed course quickly
- A time your taste overruled a request or a metric
- A time you shipped something ambitious that partially worked
- A time you worked across research, engineering, and design to land a result

## Likely interview questions to rehearse

1. What is your favorite model right now and why? Where does it fall short?
2. How would you evaluate whether a new model is better for [use case]?
3. Design an eval for [capability] with 20 prompts. What is your rubric?
4. A partner team wants feature X. How do you decide whether to build it?
5. The next model drops in two weeks and changes what is possible. What do you do with your roadmap?
6. How do you balance quality, latency, and cost for a given surface?
7. Walk me through a 60-20-20 plan for [product].
8. Tell me about something you built or analyzed yourself to learn what a model could do.

## Readiness checklist

- [ ] Daily model-use log going for at least 2 weeks
- [ ] Vibe benchmark of 5-10 prompts, rerun on at least two models
- [ ] Opinion notes on Gemini and at least two other models
- [ ] One hands-on data/failure-mode teardown
- [ ] One 5-20 prompt eval with rubric and scored results
- [ ] "Model swap" design note
- [ ] Three product critiques with clear recommendations
- [ ] One 60-20-20 plan with evals attached
- [ ] 6-8 stories drafted and rehearsed out loud
- [ ] Two mock interviews completed

## Related docs in this repo

- [PM Learnings](PM%20Learnings): Evals as the new PRD, 60-20-20, and platform strategy themes
- [CI_SP_AI_FOUNDATIONS_WALKTHROUGH.md](CI_SP_AI_FOUNDATIONS_WALKTHROUGH.md)
- [AI_MODEL_LIFECYCLE_VISUAL.md](AI_MODEL_LIFECYCLE_VISUAL.md)
