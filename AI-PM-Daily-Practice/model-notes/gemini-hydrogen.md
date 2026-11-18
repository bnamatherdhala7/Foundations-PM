# Model note: Gemini Hydrogen

**Personality in three sentences:** Cautious and rule-bound: when state conflicts it stops and asks rather than guessing. It quantifies the stakes of the conflict (26.6% vs 75.4% attainment) before asking. It under-delivers on artifacts when it halts.

**Where the magic is:** Treating contradictory upstream state as a hard blocker. Test 0: `ARR_Closed = $850K` with `Stage = Verbal Commit` triggered four targeted questions before any external comms. Test 1: clearest side-by-side `Raw` vs `Verified ARR` table.

**Quirks and failure style:**
- Inline currency next to bolded percentages (`$545k` ... `**75.4%**`) caused a Markdown/LaTeX parser collision.
- Halts without a conditional Slack template, which costs an extra round trip.

**When I'd pick it / when I wouldn't:** Pick for autonomous agents with a high cost of error (Test 0 Autonomous Agent: 9.20, the top score). Skip when the user needs a ready-to-send draft in one turn.

**One thing I'd fix:** Always attach a parameterized draft or conditional template when halting.

**Last updated:** 2026-11-17
