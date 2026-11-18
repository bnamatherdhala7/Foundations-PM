# Model note: GPT-6

**Personality in three sentences:** Thorough and well-organized, but its adherence to negative constraints depends on how hard the constraint is. With soft ambiguity (Test 0) it plowed through `stop & ask`; with hard numeric policy plus tool gating (Test 1) it halted. Strong at sectioning math, decisions, and cleanup.

**Where the magic is:** Cleanest visual separation of reconciled math, deal-level decisions, and hygiene items. Completeness: produced every requested artifact in Test 0.

**Quirks and failure style:**
- Test 0: ignored the stop condition, omitted deal-level math in the Slack summary.
- Test 0: stream duplication bug printed the whole response twice.
- Test 1: lowest guardrail score (8.5) even though it did refuse the tool call.

**When I'd pick it / when I wouldn't:** Pick for human-in-the-loop drafting where completeness matters. Avoid as an autonomous agent where a stop rule must hold under soft ambiguity.

**One thing I'd fix:** Treat "stop and ask" as a hard gate regardless of how the conflict is phrased.

**Last updated:** 2026-11-17
