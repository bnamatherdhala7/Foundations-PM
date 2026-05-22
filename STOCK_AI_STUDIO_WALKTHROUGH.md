# Adobe Stock AI Studio — PM Walkthrough
### For: Hiring Manager Conversation
### Role: Principal PM, GenAI on Adobe Stock

---

## The Opening Frame (say this first)

> "I want to walk you through how I think about Adobe Stock AI Studio — not as a feature list, but as a customer workflow problem. Adobe Stock has 300 million licensed assets. The challenge isn't inventory. It's the gap between what a customer types and what they actually need — and the gap between finding an asset and using it in a campaign. AI Studio is the product that closes both of those gaps. Let me walk through where the pain is today, what I'd build to solve it, and how I'd measure whether it's working."

---

## Section 1: The Customer Reality Today

### Who is using Adobe Stock

Three distinct customers — each with a different version of the same frustration:

**1. The enterprise brand team** (highest value segment)
A marketing team at a Fortune 500 company. They license hundreds of assets per month for global campaigns. They have a brand guide — specific colors, specific visual registers, specific rules about what "looks like us." Their problem: Stock gives them technically accurate results, but nothing that *feels like the brief*. They spend more time searching than creating.

**2. The in-house creative** (volume user)
A designer at an agency or mid-size company. They need the right asset fast — they're not evaluating tools, they're on a deadline. Their problem: they find an asset that's compositionally perfect, but the color is wrong, the background doesn't work, or the proportions don't match their layout. They leave Stock, open Photoshop, fix it manually, come back. Every edit is a 30-minute detour.

**3. The content creator / SMB marketer** (growth segment)
Someone running social for a brand or business. They don't have a designer. They need something that looks professional and is on-brand in 10 minutes. Their problem: they don't know the right keywords. They describe what they want in natural language — "warm, authentic, woman entrepreneur, coffee shop vibe" — and Stock returns results that look like a catalog from 2015.

---

## Section 2: The Five Pain Points (with evidence)

### Pain Point 1 — Keyword search doesn't understand creative intent

**The behavior:** User types a natural language creative brief. Stock's search engine parses nouns and adjectives. Returns technically correct results that are creatively wrong.

**Example:** Search "confident professional woman technology warm editorial" → gets 3,000 results of women at laptops in office lighting. None of them feel like a Fast Company cover, which is what the brief actually means.

**Why it's a GenAI problem specifically:** CLIP-based semantic search can match concepts, but it doesn't encode *aesthetic register* — editorial vs. commercial, candid vs. staged, warm vs. clinical. The model has to be trained on creative brief language patterns, not just image-caption pairs. The signal to train on is **dwell + license** — users who looked at an asset for a long time before buying. That dwell signal encodes "this felt right" in a way that download-only data misses entirely.

**Size of the problem:** Every session where a user searches, browses for 5+ minutes, and leaves without licensing is a lost transaction. On a catalog of 300M assets, the search experience is the product.

---

### Pain Point 2 — "Right asset, wrong color" kills the workflow

**The behavior:** A customer finds a hero image they love. The jacket is red. Their brand is blue. Today: download → Photoshop → mask → recolor → re-export → re-upload to campaign manager. 45 minutes of production work on a creative decision that took 30 seconds.

**What exists:** AI Studio has a Change Color tool at `stock.adobe.com/ai-studio/images/change-color`. The model works. **The problem is where it lives.** It's a separate destination the user has to navigate to intentionally. It's not surfaced at the moment of purchase friction.

**The fix:** Surface the Change Color tool inline, at the moment of behavioral abandonment. When a user dwells on an asset for 8+ seconds and doesn't add to cart, show a one-line prompt: *"Wrong color for your brand? Change it with AI — free preview."* That converts a lost session into a purchase with zero new model work. It's a placement decision, not an engineering project.

**The metric:** Track the "almost bought" cohort — dwell ≥ 8s + no add-to-cart. Measure what % convert when the inline prompt is shown. Hypothesis: 30–40% conversion, which is direct revenue recovery from sessions that were already lost.

---

### Pain Point 3 — No path from Firefly generation to licensed Stock

**The behavior:** A designer uses Firefly to explore visual directions. They generate an image that captures exactly the right look for the campaign. But for the actual paid media placement — which requires IP indemnification — they need a licensed Stock asset. Today: they start a completely new Stock search from scratch. The Firefly output that encoded their creative intent is disconnected from Stock.

**Why this is the most important unsolved problem:** This is the only cross-Adobe workflow that no competitor can replicate. You need to own both the generation (Firefly) and the licensed library (Stock) to build a "find similar licensed assets" button. Canva can't build this. Midjourney can't build this. Getty doesn't have Firefly. This is a structural moat that currently exists in pieces and isn't connected.

**The fix:** "Find similar in Stock" as a native output action on every Firefly-generated image. One click → Stock returns 12 licensed assets that match the color temperature, composition register, and style of the generated image. The user gets the creative direction they already decided on, with full licensing. The conversion rate from this flow should be 3–5× higher than cold Stock search because the creative decision is already made.

**The cross-team ask:** Firefly (the generation model and visual embedding), Stock engineering (the similarity search index), Licensing (entitlement handoff). This is the "influence cross-Adobe teams to build what customers need when it doesn't yet exist" problem verbatim.

---

### Pain Point 4 — Variation breadth is broken for campaign teams

**The behavior:** A digital campaign team needs 5 visual directions for A/B testing a hero image. Each direction needs 3–5 representative assets for evaluation. Today: 5 separate searches, separate filtering sessions, separate evaluation time. An hour of search work before a single creative decision is made.

**The size:** Enterprise campaign teams run 10–20 A/B tests per quarter. Each test requires multiple visual directions. The current search experience makes each variation a full production effort — exactly the "hook testing deficit" problem.

**The fix:** One prompt → clustered variation set. User enters the brief → AI Studio returns 5 conceptually distinct visual directions, each with 3–5 representative assets. Not 50 variations of the same composition — 5 genuinely different takes, each internally coherent as a direction.

**The StoryForge parallel (use this in the conversation):**
> "This is the problem I solved with StoryForge for video creators. They couldn't test which hook resonated because producing even one variation was a full effort. The solution was a variation engine: one input, 10 strategically differentiated outputs. AI Studio needs the same architecture for discovery: one brief, 5 curated visual directions, immediately comparable."

---

### Pain Point 5 — Video b-roll is keyword-trapped

**The behavior:** A video editor needs b-roll of "a woman working confidently in a coffee shop, warm natural light, shallow depth of field, editorial pacing, 10–15 seconds." Types those exact words. Gets generic results. Spends 40 minutes browsing. Either settles or gives up.

**Why video is harder than image:** Images can be evaluated in a thumbnail. Video requires temporal understanding — the quality of motion, the pacing, the depth of field change over time. These properties don't survive keyword search or even standard visual embedding. A video-native embedding model has to understand *what the clip feels like to use in an edit*, not just what objects appear in it.

**The size:** Video subscription plans are the highest-margin segment in Stock. Time-to-license for video is estimated 3–4× longer than for images. AI Studio closing that gap is a direct margin improvement, not just a UX improvement.

**The fix:** Video semantic search with temporal embeddings — motion quality, pacing, depth of field transitions, subject consistency — built on the same dimensional framework as VBench. The research work for Firefly video evaluation and the Stock video search index are the same problem from two directions. A PM who understands both can coordinate them.

---

## Section 3: The AI Studio Features Today (and their gaps)

| Feature | What it does | Where it works well | The gap |
|---|---|---|---|
| **Text to Image** | Generate new images using Firefly, commercially licensed | Creative exploration, concept mockups | No path back to licensed Stock assets matching the result |
| **Generate Similar** | Find visually similar assets in the catalog | Compositional matches | Misses stylistic/aesthetic register — returns objects, not vibes |
| **Remove Background** | AI-powered background removal | Product photography, cut-outs | Works but lives in a separate workflow from search |
| **Change Color** | Change object colors in stock images | Brand color matching | Lives at a separate URL — not surfaced at purchase friction moment |
| **Generative Fill / Expand** | Extend image canvas, fill objects | Layout adaptation, format resizing | Not connected to brand style — every fill starts from scratch |

**The pattern across all five:** The models work. The gap is workflow integration — each tool is a destination instead of a capability that appears at the moment the user needs it.

---

## Section 4: The 6–12 Month Roadmap I'd Build

### Tier 1 — Ship in 60 days (placement fixes, no new model work)

**1A. Inline Change Color at abandonment point**
When: user dwells ≥ 8s on an asset without adding to cart.
Trigger: "Wrong color? Change it with AI."
Effort: frontend placement + event tracking. No model work.
Metric: abandoned session → license conversion rate.

**1B. Search results → AI Studio direct links**
Add an "Edit with AI Studio" button on every image result card.
Surfaces Generate Similar, Remove Background, Change Color directly from search results.
Effort: frontend addition. No backend changes.
Metric: AI Studio feature activation rate per search session.

---

### Tier 2 — Ship in 90 days (product experiments)

**2A. Firefly → "Find Similar in Stock" bridge**
After any Firefly generation, surface a Stock similarity panel.
Returns 8–12 licensed assets matching the generated image's visual embedding.
Cross-team: Firefly (embedding export), Stock search (similarity query).
Metric: Firefly session → Stock license conversion rate (target: 3× cold search baseline).

**2B. Campaign variation set**
Prompt → 5 visual directions, each with 3–5 representative assets.
Built on clustering of semantic search results + editorial curation layer.
Metric: assets-viewed-per-session, A/B test launch rate for enterprise users.

---

### Tier 3 — Ship in 6 months (model-dependent)

**3A. Semantic search with creative brief language**
Fine-tune the Stock search embedding on dwell + license signal.
Captures aesthetic register, not just object labels.
Metric: search-to-license conversion rate for natural language queries (target: 2× keyword baseline).

**3B. Video temporal semantic search**
Video-native embeddings encoding motion quality, pacing, depth of field.
Metric: video time-to-license (target: reduce from estimated 4× image → 1.5× image).

**3C. Batch brand transformation**
Select 50 licensed assets → apply brand color profile + export formats + brand overlay.
Batch pipeline with Custom Models integration from Firefly.
Metric: production time per asset batch (target: 2 days → 20 minutes for enterprise teams).

---

## Section 5: How I'd Measure Success

### North star metric: Asset Adoption Rate
*Did the customer actually use the asset in a campaign — not just download it?*

This is the metric equivalent of StoryForge's Posted Video Rate. Download measures what was available; adoption measures whether the workflow actually worked. If AI Studio is improving discovery and customization, adoption rate rises because customers find assets that fit without heavy manual editing.

### Supporting metrics by phase

| Metric | What it measures | Target |
|---|---|---|
| Abandoned session conversion rate | Inline color prompt → license | ≥ 30% of ≥8s-dwell non-purchasers |
| Semantic query conversion rate | Natural language search → license | 2× keyword query baseline |
| Firefly → Stock conversion | Firefly generation → Stock license | 3× cold search baseline |
| Time-to-license (image) | Search → first license | < 3 minutes at 75th percentile |
| Time-to-license (video) | Search → first license | < 10 minutes at 75th percentile |
| Variation set adoption | 5-direction cluster → ≥1 license | ≥ 60% of enterprise sessions |
| AI Studio activation per session | Users who interact with ≥1 AI tool | 25% → 50% within 6 months |

---

## Section 6: Demo Script — What to Show and Say

*This is the 15-minute walkthrough for the hiring manager. Each beat has what to show and what to say.*

---

### Beat 1 — Open with the problem, not the product (2 min)

**What to say:**
> "Before I show you anything in the product, I want to establish the customer problem. Adobe Stock has 300 million licensed assets — the inventory problem is solved. The remaining problem is in two parts: finding the right asset, and using it without an hour of Photoshop work afterward. AI Studio is the product that closes both of those gaps. Everything I'm going to show you today maps back to one of those two problems."

**Why this works:** Grounds the conversation in customer value before you touch the product. Sets up every demo beat as a problem → solution pair.

---

### Beat 2 — Show the search gap (3 min)

**What to do:** Open stock.adobe.com. Type a creative brief in natural language: *"confident woman entrepreneur warm editorial coffee shop natural light"*

**What to say:**
> "Watch what happens. The results are technically accurate — there are women, there are coffee shops. But none of these feel like the brief. A creative director looking at this would immediately say 'none of these are right.' The search engine parsed the nouns. It didn't parse the intent. The visual register this brief is asking for — editorial warmth, candid movement, natural depth of field — those aren't nouns. They're aesthetic properties that the current search model doesn't encode.

> The AI Studio opportunity is a semantic search layer trained on creative brief language and the dwell + license signal. When a user looks at an asset for 8 seconds before buying, that's the training signal that says 'this felt right.' We're not capturing that today. When we do, the search-to-license conversion rate for natural language queries should double."

---

### Beat 3 — Show the Change Color workflow problem (3 min)

**What to do:** Navigate to `stock.adobe.com/ai-studio/images/change-color`.

**What to say:**
> "This tool works. The model is solid — you can see it adapts the color realistically while preserving shadows, textures, and lighting. The problem isn't the model. The problem is that I had to navigate here intentionally. A customer who finds an asset in search results that's compositionally perfect but the wrong color for their brand doesn't know this exists. They download the asset, open Photoshop, spend 45 minutes manually masking and recoloring, and come back to Stock only to repeat the same problem on the next asset.

> The fix I'd ship in 60 days requires no model work at all — it's a placement decision. When a user dwells on an asset for 8 or more seconds and doesn't add it to cart, we show one line: 'Wrong color for your brand? Change it with AI.' That single trigger converts an abandoned session into a purchase. We're not building anything new — we're surfacing what we already built at the moment it's needed."

**The prompt to leave with them:**
> "What I find interesting about this problem is that we have the full data to identify the 'almost bought' cohort — dwell time, scroll depth, add-to-cart events. We're sitting on a behavioral signal that tells us exactly which users were a nudge away from purchasing. AI Studio is the nudge."

---

### Beat 4 — The Firefly → Stock gap (3 min)

**What to do:** Open Firefly at firefly.adobe.com. Generate an image: *"confident woman entrepreneur warm tones coffee shop editorial lighting."* Show the generated result.

**What to say:**
> "This is the creative decision moment. The designer just generated the exact look they want for the campaign. Now watch what happens. They need a licensed asset for the paid media placement — Firefly-generated images carry indemnification, but they want to see if there's a Stock equivalent with a human photographer's quality. They have to leave Firefly, go back to Stock, and start a completely new search from scratch. The visual intent they just captured — that specific color temperature, that composition register, that depth of field quality — none of it transfers to the Stock search experience.

> The feature I'd build: a 'Find Similar in Stock' button as a native output action on every Firefly generation. One click returns 12 licensed Stock assets that match the generated image's visual embedding. The customer gets the creative direction they already decided on, with full licensing.

> No competitor can build this. You need to own both the generation model and the licensed library. Midjourney has generation but no library. Getty has a library but no generation. Only Adobe has both — and right now, they don't talk to each other."

---

### Beat 5 — Close with the roadmap and metrics (2 min)

**What to say:**
> "So here's how I'd prioritize. Tier 1 — the 60-day wins — are placement fixes. Change Color inline at the abandonment moment. AI Studio shortcuts from search result cards. These require no model work and have directly measurable revenue impact from day one.

> Tier 2 — 90 days — the Firefly-to-Stock bridge. This is the biggest cross-team project because it requires Firefly and Stock engineering to share a visual embedding pipeline. But the conversion rate hypothesis — 3× cold search baseline — justifies the coordination cost.

> Tier 3 — 6 months — the model-dependent work. Semantic search retrained on creative brief language. Video temporal embeddings. Batch brand transformation for enterprise teams.

> The north star metric throughout: Asset Adoption Rate. Not downloads — actual use in a campaign. If a customer downloads an asset and immediately opens Photoshop for 45 minutes, the discovery workflow failed. If they download it and it goes straight into their campaign manager, the workflow worked. That's the metric that tells me whether AI Studio is solving the real problem."

---

### Closing line

> "The throughline across all of this is that AI Studio already has the right capabilities — the models are good. The gap is workflow integration. Each tool lives at a separate destination instead of appearing at the moment the customer needs it. My job as PM is to close that gap — to make AI Studio the layer that sits between every Stock interaction and every customer friction point. That's what I'd build."

---

## Quick Reference: Facts and Numbers for This Role

| Fact | Value |
|---|---|
| Adobe Stock catalog | 300M+ licensed assets |
| Change Color tool URL | stock.adobe.com/ai-studio/images/change-color |
| "Almost bought" signal | Dwell ≥ 8s + no add-to-cart = purchase intent without conversion |
| Inline color prompt hypothesis | 30–40% conversion from abandoned sessions |
| Firefly → Stock conversion hypothesis | 3× cold search baseline (intent already established) |
| Video time-to-license vs. image | Estimated 3–4× longer due to keyword-trapped search |
| Batch transformation ROI | 2 days of production work → 20 minutes for enterprise teams |
| Semantic search lift hypothesis | 2× search-to-license conversion vs. keyword queries |
| North star metric | Asset Adoption Rate (used in campaign, not just downloaded) |
| Firefly moat | Only platform with both generation (Firefly) + licensed library (Stock) |
| No competitor can build Beat 4 | Midjourney has no library. Getty has no generation. Only Adobe has both. |
