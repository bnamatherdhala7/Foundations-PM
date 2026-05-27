# AI/ML Pipeline Lifecycle

## Phase 1 — Data Pipeline
*How raw sources become training-ready embeddings.*

## Phase 2 — Training Pipeline
*Pretraining through alignment and safety tuning.*

## Phase 3 — Evaluation, Deployment, and Observability
*From golden datasets through production monitoring and governance.*

---

## Notes

All three phases are fully expanded. Every node is clickable — tap any box to drill deeper into that topic.

**The iterative feedback loop** (shown as the dashed return arrow in Phase 3) is critical. Production signals from observability feed back into both the data pipeline (flagging new failure cases) and the training pipeline (triggering fine-tune cycles). This is where the lifecycle becomes continuous rather than a one-time build.

**Two phases worth highlighting** — observability (Phase 3, step 11) and governance (step 12) — are where most real-world production issues surface, especially at Adobe given IP sensitivity around training data and the EU AI Act's documentation requirements.
