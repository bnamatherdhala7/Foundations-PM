# Call Prep — Firefly PM Role

---

## 1. Before the call (3 actions)

- [ ] **Pin FF-Stil first on GitHub profile** — go to github.com/bnamatherdhala7 → Edit profile → Customize pins → drag FF-Stil to position 1
- [ ] **README updated** ✓ — "How this connects to Firefly" section added to FF-Stil README
- [ ] **Know these cold (story-forge)** — see section 4 below

---

## 2. The core narrative to deliver on Firefly eval

Open with the Stil bridge, then extend to Firefly. Do not open a notebook. Talk through it.

**The bridge sentence:**
> "I built a working image quality measurement system in Stil — it's called the Feed Cohesion Score. It gives a 0–100 number to visual consistency using deterministic pixel math: color temperature variance, brightness, contrast, saturation. No model inference, sub-second at scale. The interesting thing is that same infrastructure maps directly onto what Firefly needs for output quality evaluation."

**The extension (three dimensions):**
1. **Prompt adherence** — CLIP embedding similarity between the prompt and the output. Did the image actually contain what was asked for? This is the one place you need a model because pixel math can't reach semantic intent.
2. **Style consistency across variations** — same math as Feed Cohesion Score, different input. Instead of measuring variance across a creator's feed over time, you measure variance across N outputs from the same prompt+style reference. Are the outputs actually coherent with each other?
3. **Human preference ranking** — pairwise sessions (which of these two do you prefer?), ELO scoring, then correlated with generation parameters: sampler settings, CFG values, style-reference weight. This surfaces which Firefly settings actually drive human preference, not just technical quality.

**The key insight to land:**
> "The thing Stil demonstrates is that you don't need a learned quality model to measure consistency — Pillow math is faster, cheaper, and more interpretable. You add a model only where pixel math genuinely can't reach. For Firefly eval, that means one CLIP call per image for prompt adherence, and zero model calls for everything else in the consistency stack."

---

## 3. If they push deeper on the eval framework

**"How would you run it in practice?"**
> Build it as a quality gate on generation: every batch that goes through Firefly web runs the consistency check automatically. Feed Cohesion Score for the batch surfaces immediately — if variance is above threshold, the batch is flagged before it reaches the user. Human preference sessions happen async, separate from generation — users rate pairs after the fact. The ELO model trains on accumulated ratings and becomes the signal for model tuning.

**"What's the north star metric?"**
> User-perceived quality at volume. The proxy metric is Human Preference Rate — what % of generations does a user prefer over the alternative they were shown? If you're at 60%, you're at coin flip. Target is 75%+. Feed Cohesion Score is the leading indicator; Human Preference Rate is the lagging validator.

**"What's the hardest part?"**
> Style adherence. Color temperature and brightness variance are easy to quantify. Whether an image "feels like" the style reference is harder — CLIP helps but the embeddings don't fully capture aesthetic intent. That's where you need human signal to close the gap between what the metric measures and what users actually want.

---

## 4. Story-forge north star metrics (know these cold)

| Metric | Target | Why |
|---|---|---|
| **Posted Video Rate (PVR)** | ≥ 60% | % of generated videos creators actually post — the real quality signal, not generation quality |
| **Time-to-First-Video (TTFV)** | < 5 min at 95th percentile | If it's slower than doing it manually, it's not a tool |
| **Hook Performance Variance** | ≥ 3× between top and bottom hook | Proves the variation strategy has real value — otherwise you're just generating noise |

**Why PVR is the one metric:** Task completion is not quality. A video can generate successfully and still never get posted because it doesn't match the creator's brand or audience. PVR measures whether the output was actually good enough to use, which is the only signal that matters.

---

## 5. Quick facts about Stil to quote

- **Feed Cohesion Score:** 0–100, measures color temperature + brightness + contrast + saturation variance across uploaded photos. Pure Pillow math, no API calls, zero cost.
- **Two-layer memory:** choices_log (deterministic, from actual tool calls) + style_signature (AI-extracted intent via Haiku, fills the semantic gap). Behavior beats declaration.
- **Cost target:** < $0.05 per user per day. All Haiku.
- **The core insight:** Most "memory" features store what you *said*. Stil stores what you *did*. The choices log reads actual API tool calls — no paraphrasing, no interpretation, deterministic ground truth.

---

## 6. One sentence on each project (for intro)

**FF-Stil:** "I built a conversational AI editing assistant that stores what you do rather than what you say, and ships a Feed Cohesion Score that quantifies visual consistency — the thing that drives brand drift that creators only notice six months after it started."

**StoryForge:** "I built a 4-agent hook variation engine for course creators that turns one asset set into 10 strategically differentiated promo videos, each built on a different hook psychology — the north star metric is whether creators actually post the output, not whether it generates successfully."
