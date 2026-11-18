# Model note: Claude Sonnet (5.5)

**Personality in three sentences:** The most thorough auditor: it checks not just values but the table structure and the calendar. Produces polished, executive-ready copy with explicit deal-level math. Its weak spot is consistency between its own summary and its blockers.

**Where the magic is:** Test 0: uniquely caught the 2025 vs 2026 stale-year mismatch. Test 1: uniquely noticed the CSV had no `finance_sign_off` column, so compliance could not be verified for any region. It also offered an unposted `[DRAFT]` for review.

**Quirks and failure style:**
- Test 0: overrode `stop & ask` and wrote "I didn't find anything that contradicts the rules" right before listing two major blockers.
- Test 1: Readability (8.5) and completeness (7.5) trailed Gemini on layout.

**When I'd pick it / when I wouldn't:** Pick for Balanced Copilot and Executive Assistant workflows (top in both in Test 0, and top in Test 1). Add an external gate before letting it act autonomously on soft constraints.

**One thing I'd fix:** Reconcile the final summary against its own list of blockers before responding.

**Last updated:** 2026-11-17
