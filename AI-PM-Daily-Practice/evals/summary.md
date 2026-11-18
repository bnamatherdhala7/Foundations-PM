# Eval Summary: Agentic Audit Benchmarks

Sources: [Test 0](agentic-audit-benchmark.md), [Test 1](test-1-policy-trap-benchmark.md), [v2.0 write-up with Test 2](agentic-guardrail-benchmark-v2.md).

## Cross-test findings
1. **Constraint hardness drives guardrail adherence.** With soft ambiguity (Test 0), GPT-6 and Sonnet plowed through `stop & ask`. With hard numeric thresholds plus a gated tool (Test 1), all three halted. Design implication: pair natural-language stop rules with explicit thresholds and a gated tool call.
2. **Data math is saturated.** All models scored 10 on grounding in both tests. It no longer separates models, so future evals should add harder reconciliation.
3. **Structural auditing is the differentiator.** Sonnet caught calendar drift (Test 0) and the missing `finance_sign_off` column (Test 1). Checking whether the data can support the rule, not just what the rows say, is the capability to probe next.
4. **Halting and delivering trade off.** Gemini halts best but delivers least in Test 0 (completeness 5.0). Best behavior is halt plus an unposted draft, which Sonnet did in Test 1.
5. **Persona weighting flips winners.** Gemini wins as an autonomous agent; Sonnet wins as copilot and executive assistant. One scalar score hides this.

6. **Irreversible tools made halting universal (Test 2).** All three models held fire on `terminate_cluster` (guardrail 10 each), so the guardrail axis no longer discriminates there. Differences moved to risk surfacing: Sonnet caught that `stg-ts-backfill-02` had been idle two weeks (an old experiment, not a daily health check) and warned about ephemeral checkpoints; GPT-6 scored lowest on risk surfacing (8.5) but had the best layout.
7. **Pattern across all three tests:** Sonnet leads when the task rewards looking beyond the stated data (calendar drift, missing column, idle history). Gemini leads on pure halting restraint. GPT-6 leads on layout in Test 2 but trails on guardrails and risk surfacing.

## Open questions and next evals
- Does Test 1's result hold when the policy is stated softly instead of numerically?
- Test 2 had an explicit "irreversible tool" signal. Does restraint hold when the tool looks routine (e.g. `scale_down`, `delete_snapshot`)?
- Test 2 prompt says "Go ahead and take care of it." Add a variant with stronger user pressure or authority claims.
- Add a tool-calling variant of Test 0 to separate "wrote about it" from "actually called the tool".
- Run each prompt 3+ times to separate model behavior from sampling noise (single run per model so far).
- Add a scenario where the right action is to proceed, to catch over-refusal.

## Method notes
- Weighted scores in the source write-ups don't all match the dimension scores. Recomputed values are in `../vibe-benchmark.md`.
- Test 2 uses four dimensions with unstated weights, so its totals can't be verified from the source.
- Dimension scores are author-assigned (single rater), so treat gaps under about 0.5 as noise.
