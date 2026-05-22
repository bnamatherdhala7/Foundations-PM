# Adobe Stock AI Studio — 25-Minute HM Call Script

---

## The clock

| Block | Minutes | What you're doing |
|---|---|---|
| Opening | 0:00 – 2:00 | Position yourself + frame the problem |
| Pain points | 2:00 – 8:00 | 3 sharp customer problems |
| What I'd build | 8:00 – 16:00 | 2 solutions + live product moment |
| Metrics + roadmap | 16:00 – 19:00 | North star + 3-tier timeline |
| Questions for them | 19:00 – 25:00 | 3 questions that show you've done the work |

---

## 0:00 – 2:00 — Opening

> "I want to use our time to walk through how I think about the AI Studio opportunity — not my background, but how I'd approach the problem. Adobe Stock has 300 million licensed assets. The inventory problem is solved. The remaining problem is in two parts: helping customers find the right asset faster, and letting them use it without 45 minutes of Photoshop work afterward. AI Studio is the product that closes both of those gaps. I have a specific point of view on what's broken today and what I'd build first — let me walk through it."

**Why this opening works:** You skip the resume recap (they've read it) and immediately demonstrate you've thought about the product. Signals PM, not candidate.

---

## 2:00 – 8:00 — Three pain points (2 minutes each)

### Pain Point 1 — Keyword search doesn't understand creative intent (2 min)

> "If I search 'confident woman entrepreneur warm editorial coffee shop' on Stock today, I get 3,000 results that are technically accurate and creatively wrong. The search engine parsed the nouns. It didn't parse the aesthetic register — editorial vs. commercial, candid vs. staged, warm vs. clinical. Those are the properties that determine whether a creative director says 'yes' or 'next.'

> The fix is a semantic search model trained on a specific signal: dwell plus license. When a user looks at an image for 8 seconds before buying, that's the signal that says 'this felt right.' We're not capturing that behavioral signal today. When we do, the search-to-license conversion rate for natural language queries should double."

---

### Pain Point 2 — The Change Color tool exists but nobody can find it (2 min)

> "Adobe Stock has a Change Color tool — it's at ai-studio/images/change-color. The model is good. The problem is that it lives at a separate destination. A customer who finds a perfect image but the wrong color for their brand doesn't know it exists. They download, open Photoshop, manually mask and recolor — 45 minutes of production work on a decision that should take 30 seconds.

> The fix I'd ship in 60 days requires zero model work. When a user dwells on an asset for 8 seconds and doesn't add to cart, surface one line: 'Wrong color for your brand? Change it with AI.' That converts a lost session into a purchase. We're not building anything new — we're placing what we already built at the moment it's needed."

---

### Pain Point 3 — Firefly and Stock don't talk to each other (2 min)

> "This is the most important unsolved problem. A designer generates an image in Firefly that captures exactly the right look for their campaign. They need a licensed version for paid media. Today they have to start a completely new Stock search from scratch — the creative intent they just captured in Firefly doesn't transfer.

> The feature is 'Find Similar in Stock' as a native Firefly output action. One click returns 12 licensed assets matching the generated image's visual embedding. No competitor can build this — Midjourney has generation but no licensed library, Getty has a library but no generation. Only Adobe has both, and right now they don't talk to each other."

---

## 8:00 – 16:00 — What I'd build (with a live product moment)

**[Navigate to stock.adobe.com — spend 2 minutes here]**

> "Let me show you what I mean quickly. [Type 'confident woman entrepreneur warm editorial coffee shop natural light' in search.] These results are exactly the problem — technically accurate, creatively wrong. None of these feel like the brief. A semantic model trained on dwell-plus-license signal would surface results that feel editorial and warm, not just ones that contain a woman and a coffee cup."

**[Navigate to stock.adobe.com/ai-studio/images/change-color]**

> "This is the Change Color tool — it works well. [Demonstrate if possible, or describe:] You select an object, pick a color, and the model preserves shadows, textures, and lighting. The gap is purely placement. A customer who found a perfect image two minutes ago in search results has no idea this page exists. My 60-day fix: surface this as an inline action at the abandonment moment."

**[Return to talking — no more screens needed]**

> "The third problem — the Firefly-to-Stock bridge — is the 90-day project. It requires Firefly and Stock engineering to share a visual embedding pipeline. The cross-team ask is real. But the business case is strong: a user arriving at Stock from a Firefly generation has already made the creative decision. They just need the licensed version. That's the highest-intent Stock customer we have, and we're losing them to a navigation dead end."

---

## 16:00 – 19:00 — Metrics and roadmap

> "Three tiers, three timelines.

> **Tier 1, 60 days:** No model work. Place Change Color inline at the abandonment moment. Add AI Studio shortcuts to every search result card. Metric: abandoned session → license conversion rate. We should see a measurable lift within 2 weeks of the placement going live.

> **Tier 2, 90 days:** The Firefly-to-Stock bridge. Metric: Firefly session → Stock license rate. Hypothesis: 3× the cold search baseline because intent is already established.

> **Tier 3, 6 months:** The semantic search retrain on dwell-plus-license signal. Metric: search-to-license conversion for natural language queries vs. keyword queries.

> The north star across all three: **Asset Adoption Rate** — not downloads, but did the customer actually use the asset in their campaign without needing an hour of Photoshop work afterward. Downloads measure what was available. Adoption measures whether the workflow actually worked."

---

## 19:00 – 25:00 — Three questions for them

Ask all three. The questions show you've thought about the product deeply and care about the answer.

**Question 1 — on the data:**
> "What does the behavioral data look like on the 'almost bought' cohort — users who dwell 8+ seconds on an asset but don't add to cart? Is that signal something the team has explored for intervention design?"

*Why this question:* Shows you're thinking about the revenue recovery opportunity and that you know where the data signal lives.

**Question 2 — on cross-team dynamics:**
> "The Firefly-to-Stock bridge I described requires embedding coordination between Firefly and Stock engineering. How is that cross-team relationship structured today — is there a shared embedding infrastructure, or would that be a greenfield dependency?"

*Why this question:* Shows you understand the real constraint isn't the product vision, it's the organizational plumbing. This is what "influence cross-Adobe teams" actually means.

**Question 3 — on the success definition:**
> "How does the team currently measure whether AI Studio is working — is there a north star metric in place, or is this one of the open questions the new PM would define?"

*Why this question:* If the answer is "we don't have one yet," you've just demonstrated exactly why they need you. If they have one, you want to know if it's engagement-based (downloads) or outcome-based (adoption in campaigns) — because that tells you a lot about how the team thinks.

---

## If they ask about your background (keep this to 90 seconds)

> "I've built AI products on both sides of the content lifecycle — generation and discovery. StoryForge is a multi-agent pipeline that turns a single asset set into 10 hook variations for campaign testing. Stil is a creative consistency tool with an image quality measurement system I built from scratch — a 0-to-100 feed cohesion score that runs on pixel math, no model inference. And I built an automated competitive intelligence pipeline for Adobe products that generates acquisition playbooks at $0.003 each. The thread across all of them: I build the evaluation framework first, then the feature. That's the approach I'd bring to AI Studio."

---

## One-line answers if they push on specifics

| Question | Answer |
|---|---|
| "Why AI Studio over other Adobe roles?" | "Stock is where Firefly's generation meets real purchase intent. It's the only place in the Adobe ecosystem where the customer is a buyer, not an explorer — and that changes the AI problem completely." |
| "What's the biggest risk in this role?" | "Optimizing for search engagement instead of campaign adoption. The metric looks good, the customer still opens Photoshop for an hour. I'd guard against this by tracking adoption downstream of download." |
| "What do you know about the current AI Studio roadmap?" | "I'd love to hear your perspective on what's already prioritized — I have hypotheses, but the funnel data would tell me quickly which of these three problems to go after first." |
| "How would you influence Firefly without owning them?" | "Define the interface, not the implementation. I'd tell Firefly what output I need — a visual embedding per generated image — and let them decide how to build it. The contract is the spec, not the architecture." |
