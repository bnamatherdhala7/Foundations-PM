# Product Requirements Document
## Adobe Stock AI Studio — GenAI Discovery & Customization

**Author:** Bharat Namatherdhala
**Role:** Principal PM Candidate, Adobe Stock GenAI
**Status:** Draft for Review
**Date:** May 2026

---

## 1. Problem Statement

Adobe Stock has 300M+ licensed assets. The inventory problem is solved.

The remaining problem is in two parts:

1. **Discovery gap** — customers cannot translate a creative brief into the right asset. Keyword search returns 664,000+ results with no aesthetic filtering. The tools that exist to help customers find what they want are surfaced too late — after the user has already decided to leave.

2. **Customization gap** — the AI Studio editing tools (Change color, Change mood, Change background, Animate, Expand) are on the asset detail page. But the customer's exit decision happens earlier — on the search grid, when they hover a thumbnail and decide it is not worth clicking through.

> **⭐ The core insight:** Adobe Stock already has the right capabilities. Change color, change mood, animate, find similar, similar videos — all exist. The gap is **when** they appear. The customer makes the keep-or-skip decision on the grid. That is where AI Studio needs to be.

---

## 2. Current Product Audit

*Grounded in direct product observation across two views.*

### View 1 — Search results grid

**Query tested:** "confident woman entrepreneur"
**Results:** 664,244 images

| What exists | Where | Gap |
|---|---|---|
| Find Similar | Left panel + right-click menu | Not visible on hover — buried interaction |
| AI badge | Bottom-left corner of Firefly-generated cards | Purely informational — zero click behavior |
| Suggestion chips | Below search bar | Keyword synonyms only, not aesthetic directions |
| Standard Content filter | Active by default | No aesthetic register filter (warm, editorial, candid) |

**Grid hover state — direct observation:**

A Firefly-generated image on hover shows three icons in the top-right corner:
- ❤️ Favorite
- Compare
- `...` Overflow menu

**That is the complete hover surface.** No Change color. No Change mood. No Change background. No AI Studio entry point. The AI badge sits in the bottom-left corner — static, non-clickable. Even for assets the user has already licensed ("Licensed" badge in the bottom-right), there is no inline AI action.

> **⭐ This screenshot is the proof point for Solution S1.** The customer who sees a yellow blazer and thinks "perfect composition, wrong color for my brand" has three options on hover: favorite it, compare it, or open a menu that does not contain the action they need. They will skip the asset. They will never reach the detail page where Change color exists.

**The abandonment moment is here — on the grid.** A user who does not click through to the detail page never sees a single AI Studio capability.

---

### View 2 — Asset detail page (AI-generated image #1485022142)

**What already exists and works:**

| Feature | Location | What it does |
|---|---|---|
| **Edit in AI Studio** | Action bar below image | Primary entry point to AI Studio |
| **Type to edit** | Action bar | Natural language editing instruction |
| **Change background** | Action bar | AI background replacement |
| **Expand** | Action bar | Generative canvas expansion |
| **Change mood** | `...` overflow menu | Adjusts overall tonal/lighting feel |
| **Change color** | `...` overflow menu | Object-specific color replacement |
| **Animate image** | `...` overflow menu | Generates motion from a still |
| **More from this series** | Below image | Related assets from same creator/shoot |
| **Similar in Videos** | Below image | Cross-media: video equivalents of this image |
| **Similar Keywords** | Below Similar in Videos | 25-tag keyword cloud |

**What is missing on the detail page:**

| Gap | Impact |
|---|---|
| No "Find similar licensed assets" from an AI-generated image | The highest-leverage cross-Adobe workflow does not exist |
| Similar Keywords is a generic tag cloud | 25 labels with no aesthetic signal — "woman, business, leadership, diversity" — these do not help the customer find a different creative direction |
| "More from this series" shows the same scene | Not variation — repetition. Customers need different directions, not more of the same composition |
| Change mood / Change color buried in `...` | Two of the highest-value AI actions are hidden in an overflow menu most users will never open |

---

## 3. Voice of Customer — Real Pain Points from External Sources

*Verified data from public sources to validate what the product audit shows.*

### 3a. The AI content saturation problem

**Verified fact (PetaPixel / CineD, May 2025):** 47.85% of all images on Adobe Stock are now AI-generated. Adobe has introduced upload limits in response to growing concerns about quality dilution.

> **Customer complaint (Adobe community):** *"Keep getting pushed AI images even with it toggled off."*

**What this means for the PRD:** The AI badge on grid results is not just a labeling feature — it is a trust and quality signal that customers actively try to filter out. When the AI badge has no interactive behavior (as confirmed in screenshot 3), users who want human photography cannot act on the information. The badge marks the asset but gives the customer nothing to do with that information.

**Impact on Solution S1:** The grid hover state gap is worse than it appears. Not only are AI Studio actions missing — the AI badge itself is passive. A customer who wants to find a licensed human-photographed equivalent of an AI result has no path to do so.

---

### 3b. Search result volume without quality

**Verified user complaint (Adobe community forum):**
> *"The search results are showing me about 50% catalogs, and 50% brochures, portfolios, company profiles... giving me about 26 pages of inflated and largely useless search results."*

**Second complaint:**
> *"I can't find any of those images in search with my keywords."* *(Contributor whose approved images don't surface with relevant tags)*

**What this means for the PRD:** The search quality problem is two-sided. Buyers get too many results with no aesthetic filter. Contributors cannot get their assets discovered. Both are symptoms of the same root cause: the search index ranks on keyword tag coverage, not on behavioral quality signals like dwell + license.

**Direct confirmation of Pain Point 1:** A search for "confident woman entrepreneur" returning 664,244 results is not a surprising edge case. It is the documented, consistent behavior of a system that optimizes for coverage over relevance.

---

### 3c. Platform credibility under pressure

**Verified (Trustpilot):** Adobe Stock scores 1.3/5 from 235 public reviews.

**Context:** The majority of Trustpilot reviews are from contributors (uploaders), not buyers. The dominant complaints are support responsiveness and account policy. However, the buyer-side pain is surfaced in the search complaints above — and in the product audit itself.

**What this means for the PRD:** The credibility problem compounds. 47.85% AI-generated content + poor search filtering + passive AI badges = buyers cannot trust that results match their brief. The AI Studio tools that could restore trust (Change color to make an AI image match your brand; Find Similar to find a human-photographed equivalent) are hidden from the surface where customers form that trust or distrust — the search grid.

---

### 3d. Summary: visual pain point vs. stated pain point

| Source | What customers say | What the product shows |
|---|---|---|
| Community forum | "Too many irrelevant results — can't find what I need" | 664,244 results, no aesthetic filter, generic keyword chips |
| Community forum | "AI images keep appearing even with filter off" | AI badge is passive — no click behavior, no filter action on hover |
| Product observation | Not stated — customers just leave | Change color, Change mood hidden in overflow; not on grid hover |
| PetaPixel/CineD | 47.85% AI content — quality dilution concern | No path from AI result to human-photographed equivalent |
| Product observation | Not stated — customers open Photoshop | "Edit in AI Studio" on detail page only — invisible from grid |

> **⭐ The pattern:** What customers say and what the product shows are the same problem described from different angles. Customers say "I can't find what I want." The product shows there is no mechanism to narrow 664,244 results by aesthetic register. Customers say "AI images keep appearing." The product shows the AI badge does nothing. The solutions are not speculative — they are direct responses to documented, observable gaps.

---

## 4. Goals

| Goal | Metric | Target |
|---|---|---|
| Surface AI Studio at the grid abandonment moment | AI Studio activation from grid (not detail page) | Baseline → 25% of sessions |
| Replace keyword chips with aesthetic direction clusters | Chip click → license rate | 2× keyword chip baseline |
| Connect Firefly-generated results to licensed Stock assets | AI badge → Stock license rate | Measured vs. no-action baseline |
| Reduce "Similar Keywords" to actionable aesthetic filters | Time-to-shortlist per session | −40% |
| Reduce post-download editing time | Asset Adoption Rate | +25% in 6 months |

### North Star Metric: Asset Adoption Rate

> ⭐ Downloads measure what was **available**. Adoption measures whether the workflow actually **worked**.

Asset Adoption Rate = assets used directly in a campaign without post-download Photoshop editing. This is the metric that captures whether AI Studio is genuinely reducing the customization gap — not whether customers clicked more buttons.

---

## 4. Customer Segments

### Segment A — Enterprise Brand Team *(Highest value)*
Fortune 500 in-house agencies. License hundreds of assets monthly for global campaigns. Have a brand guide — specific colors, tonal register, composition rules. **Problem:** 664,244 results for a specific query. No way to filter by the aesthetic properties that determine usability. They click into an asset, see it is wrong, go back, repeat. The AI Studio tools that could help are on the detail page — they never get there.

### Segment B — In-House Creative *(Volume user)*
Agency designers on deadline. Find a compositionally perfect asset in the wrong color. Do not know Change color and Change mood exist — both are in the `...` overflow menu on the detail page. Leave Stock, edit in Photoshop for 30–45 minutes. Return for the next asset. **Problem:** The two most valuable AI actions for this user are hidden in a menu they will never open.

### Segment C — SMB Marketer / Content Creator *(Growth segment)*
No dedicated designer. Describes what they want in natural language. Gets 25 generic keyword tags ("woman, business, leadership, diversity") as the "similar" suggestion. **Problem:** Similar Keywords is a label cloud, not a creative direction. It sends the user deeper into the same aesthetic dead end.

---

## 5. Pain Points

### Pain Point 1 — AI Studio actions appear after the customer has already decided to leave

**What the product does today:**
The full AI Studio capability — Edit in AI Studio, Type to edit, Change background, Expand, Change mood, Change color, Animate — is on the asset detail page. To reach it, a user must click through from the grid, which requires deciding the asset is worth investigating.

The customer's actual decision point is the grid thumbnail. If the thumbnail does not look right — wrong color, wrong background, wrong mood — the customer does not click. They never see the AI Studio action bar.

> **⭐ Key insight:** The user who would benefit most from Change color is the one who did not click through. They saw a thumbnail with the wrong color and skipped it. We are showing the tool to users who already decided the asset was interesting — and hiding it from users who needed it to make that decision.

**The fix:** Bring three AI actions to the grid hover state — **Change color**, **Change mood**, and **Change background**. These are the three actions that answer "this is almost right but not quite" — which is the thought that happens on the grid, not on the detail page.

---

### Pain Point 2 — Change mood and Change color are in the overflow menu

**What the product does today:**
The `...` overflow menu on the detail page contains Change mood, Change color, and Animate image. The primary action bar shows Edit in AI Studio, Type to edit, Change background, and Expand.

**The prioritization problem:** Change color and Change mood are more immediately useful to more customers than Expand. A customer who needs to match their brand palette uses Change color on almost every licensed asset. A customer who needs to shift a corporate-looking photo to editorial uses Change mood. Both are in the menu most users never open.

> **⭐ Key insight:** Move Change color and Change mood into the primary action bar. Demote Expand to the overflow. This is a 1-day product decision that increases the discoverability of the two highest-value AI actions for the in-house creative segment without any engineering work.

---

### Pain Point 3 — Similar Keywords is a missed aesthetic direction opportunity

**What the product does today:**
Below "Similar in Videos," the detail page shows a 25-tag keyword cloud: woman, business, leadership, diversity, team, black woman, conference, meeting, empowerment, professional, collaboration, success, african american, communication, equality, corporate...

These are object and subject labels. They describe what is in the image. They do not describe how it feels.

> **⭐ Key insight:** Replace the Similar Keywords section with **Aesthetic Direction chips**: "Warm & editorial · Corporate & clean · Candid & authentic · Diverse & dynamic." Each chip filters Stock results by tonal register rather than by object label. A customer who is on a detail page for one image and thinks "this is close but I want something warmer" can click "Warm & editorial" and get results that match that feel — not results that contain the same keywords.

**This is the highest-leverage, lowest-cost improvement on the detail page.** It requires reframing what the chips represent, not rebuilding the search model.

---

### Pain Point 4 — "More from this series" shows repetition, not variation

**What the product does today:**
The "More from this series" strip shows 6+ images from the same creator session. For the image shown — a conference room group shot — the series strip shows more conference room group shots with the same subjects in slightly different poses.

**The creative need:** When a customer is evaluating an asset, they are not looking for more of the same scene. They are looking for a different direction — same concept, different aesthetic approach.

> **⭐ Key insight:** Rename and reframe this section. "More from this series" → **"Different directions on this concept."** The strip should show 5 images that are *conceptually* related but *aesthetically* different: one warm and candid, one corporate and clean, one action-forward, one portrait-focused, one environmental. This is the variation breadth problem solved at the detail page level — no new model needed if we use clustering on the existing Find Similar index.

---

### Pain Point 5 — AI-generated results are not actionable from the search grid

**What the product does today:**
Firefly-generated images appear in search results with an AI badge. On the detail page, there is no action to generate a variation or find a licensed equivalent. On the grid, the badge is present but has no click behavior.

> **⭐ Key insight:** The AI badge is the right surface — it marks an asset the customer is already looking at in discovery mode. Two actions should be available from the badge: "Generate a variation" (opens Firefly with the same prompt and style seed) and "Find licensed similar" (runs a Stock similarity search against the generated image's embedding). The second action is the structural moat — no competitor can build it because no competitor owns both Firefly and a licensed asset library.

---

### Pain Point 6 — Video search is keyword-trapped

**What exists:** "Similar in Videos" on the detail page surfaces video equivalents of the current image — already built and surfaced.

**What is missing:** The reverse path — searching *for* video directly with natural language that captures temporal properties. Motion quality, pacing, lighting consistency across frames — these do not survive keyword search.

> **⭐ Key insight:** "Similar in Videos" from an image reference is a strong signal about what the customer actually wants. This interaction — image as query for video — should become a primary video discovery mode, not a secondary strip on an image detail page. If a customer finds the right image aesthetic, let them search video using that image as the query.

---

## 6. Solution Map

### Tier 1 — No model work · Ship in 45–60 days

**S1: Bring AI actions to the grid hover state**
Add Change color, Change mood, Change background as hover actions on grid thumbnails alongside the existing heart/download icons.
- **Cost:** Frontend only. The tools already exist — this is placement.
- **Metric:** AI Studio activation from grid view. Baseline → target 25% of sessions.

**S2: Promote Change color + Change mood to the primary action bar**
Swap Expand into the `...` overflow. Move Change color and Change mood into the visible action bar.
- **Cost:** 1-day product decision. No engineering.
- **Metric:** Change color + Change mood usage rate.

**S3: Replace Similar Keywords with Aesthetic Direction chips**
"Warm & editorial · Corporate & clean · Candid & authentic · Diverse & dynamic"
Each chip reranks search results by tonal register, not by swapping keywords.
- **Cost:** Frontend chip reframe + dwell-based reranking signal. No new model.
- **Metric:** Chip click → license rate vs. current keyword chip baseline. Target: 2×.

**S4: Rename "More from this series" → "Different directions on this concept"**
Reframe the strip to show aesthetically distinct takes on the same subject, not more of the same scene.
- **Cost:** Clustering logic on existing Find Similar index. No new model.
- **Metric:** Strip asset → license rate.

---

### Tier 2 — Cross-team · Ship in 90 days

**S5: Make AI badge actionable — Generate variation + Find licensed similar**
Two actions on every AI-badged result: generate a variation in Firefly, or find the licensed Stock equivalent.
- **Cost:** Firefly integration (variation) + visual embedding query on Stock index (licensed similar).
- **Metric:** AI badge → Stock license rate.

> ⭐ "Find licensed similar" from a Firefly-generated image is the only workflow in the AI image generation market that no competitor can replicate.

**S6: Image-as-query for video search**
Promote "Similar in Videos" from a detail page strip to a primary video search mode: upload or select an image → Stock returns video results that match its tonal and compositional qualities.
- **Cost:** Image embedding → video similarity query. Firefly + Stock video index integration.
- **Metric:** Image-referenced video search → video license rate.

---

### Tier 3 — Model-dependent · Ship in 6 months

**S7: Semantic search retrained on dwell + license signal**
Fine-tune the search embedding on behavioral pairs — dwell ≥ 8s then license — to capture aesthetic fit, not just object labels.
- **Metric:** Natural language search → license conversion. Target: 2× keyword baseline.

**S8: Aesthetic Find Similar**
Add tonal register, lighting quality, and shooting style as similarity dimensions — independent of subject.
- **Metric:** Find Similar → license rate. Target: 2× current.

**S9: Video temporal semantic search**
Video-native embeddings encoding motion quality, pacing, depth of field.
- **Metric:** Video time-to-license. Target: ≤ 1.5× image baseline.

---

## 7. Roadmap

```
45–60 DAYS                  90 DAYS                     6 MONTHS
─────────────────────       ─────────────────────       ──────────────────────
Tier 1 · No model work      Tier 2 · Cross-team         Tier 3 · Model work
─────────────────────       ─────────────────────       ──────────────────────
AI actions on grid      →   AI badge actionable     →   Semantic search retrain
hover state                 (Generate + Find licensed)   (dwell + license signal)

Change color + mood     →   Image-as-query          →   Aesthetic Find Similar
to primary action bar       for video search
                                                         Video temporal search
Aesthetic Direction
chips (replace keyword
tags on detail page)

"Different directions"
strip (replace "More
from this series")
```

---

## 8. Success Metrics

| Metric | Definition | Target | Signals |
|---|---|---|---|
| **Asset Adoption Rate** *(north star)* | Assets used in campaign without post-download editing | +25% in 6 months | Workflow success |
| AI Studio grid activation | AI Studio used from grid hover (not detail page) | 25% of sessions | Placement fix impact |
| Aesthetic chip → license | Clicks on direction chip → license | 2× keyword chip baseline | Direction > label |
| AI badge → license | AI badge click action → Stock license | vs. no-action baseline | Cross-Adobe bridge |
| Change color/mood usage | Actions used per session | 3× post-promotion baseline | Action bar fix impact |
| Semantic query conversion | Natural language search → license | 2× keyword baseline | Search quality |
| Video time-to-license | Search → first video license | ≤ 1.5× image baseline | Video parity |

---

## 9. Dependencies & Risks

| Dependency | Owner | Risk |
|---|---|---|
| Dwell + license event tracking | Analytics | Reranking signal and semantic retrain blocked |
| Firefly visual embedding per generation | Firefly engineering | S5 "Find licensed similar" blocked |
| Video embedding model | Research | S9 blocked |
| Find Similar clustering logic | Stock ML | S4 "Different directions" strip blocked |

**Primary risk:** Engagement metrics rise while Adoption Rate stagnates. A customer who clicks more chips and views more detail pages but still opens Photoshop for 45 minutes has not been helped. Mitigation: Adoption Rate reviewed in every sprint alongside engagement metrics. Divergence = engagement wins are not real wins.

---

## 10. New AI Features & Agentic Workflows to Improve License Rates

*These are not fixes to what exists — they are net-new capabilities that create new license-triggering moments.*

---

### 10a. AI Features

**F1 — Brief-to-Asset (Prompt-to-Licensed)**
User enters a creative brief or pastes campaign copy → AI Studio returns a split view: Firefly-generated image on the left (directional), closest licensed Stock equivalents on the right (ready to license).

Why it lifts license rate: The customer who starts with a Firefly generation is exploring direction. Showing the licensed equivalent at that exact moment — when intent is highest — collapses the gap between "I know what I want" and "I found it." No new discovery session required.

Metric: Brief-to-license completion rate. Hypothesis: 4× cold search baseline because the creative decision and the discovery happen in the same interaction.

---

**F2 — Brand Kit Filter**
User uploads their brand guide (hex colors, visual rules, tone adjectives) once. AI Studio automatically reranks every search result set by brand match — assets that fit the brand palette surface first, assets that conflict are deprioritized. For assets that are "almost right," an inline Change color preview applies the brand palette automatically.

Why it lifts license rate: The #1 reason in-house creatives leave without licensing is "this is close but not quite our brand." Brand Kit Filter converts "close" into "done." No Photoshop session required.

Metric: Sessions with Brand Kit active vs. without — license rate delta. Target: +35% license rate for enterprise segment.

Cross-team: Custom Models (Firefly) for the style matching layer; Stock search index for the reranking layer.

---

**F3 — Context-Aware Search (Page-Drop Search)**
User pastes a URL or uploads a design layout. AI Studio reads the color scheme, content type, layout structure, and visual register of the page → returns assets that would fit that specific placement in context.

Why it lifts license rate: Designers searching for assets without context often over-filter or under-filter. A designer building a landing page for a fintech product does not search "fintech landing page hero" — they search "professional woman technology" and get 664,244 results. Context-aware search understands the destination and filters for fit.

Metric: Context-aware session → license rate vs. standard search. Hypothesis: 2.5× because result relevance is anchored to a real use case.

---

**F4 — Campaign Coherence Score**
User adds multiple assets to a collection or licensing cart → AI Studio scores the set for visual coherence: color temperature variance, lighting consistency, tonal register alignment (the same dimensions as the Feed Cohesion Score in Stil, applied to a multi-asset purchase).

> ⭐ A set that scores below 60 gets a flag: "These assets may look inconsistent in a campaign. Here are 3 replacements that would improve cohesion." Each replacement is a new license opportunity.

Why it lifts license rate: Enterprise teams often license 10–20 assets per campaign. They don't discover that the set is incoherent until it is assembled in the campaign manager — after the brief is over. Surfacing this at checkout converts a "good enough" cart into a higher-quality, higher-value purchase.

Metric: Cart-level coherence warning → replacement license rate. Target: 20% of flagged carts add at least one replacement asset.

---

**F5 — Negative Semantic Filtering**
Search with exclusions in natural language: *"confident woman entrepreneur — not posed, not corporate, not stock-looking."* The model understands aesthetic exclusions, not just keyword exclusions.

Why it lifts license rate: The most common reason a customer browses 5+ pages and exits without licensing is that every result looks like a stock photo. Negative filtering removes the category feel without requiring the customer to articulate what they DO want — they just exclude what they don't.

Metric: Negative filter usage → license rate vs. standard query. Target: 1.8× because precision reduces browsing time.

---

### 10b. Agentic Workflows

**A1 — Campaign Asset Agent** *(Highest license rate impact)*

```
Input: campaign brief (product, audience, message, channels, brand guide)
         ↓
Step 1: Agent parses brief → extracts visual requirements per channel
         ↓
Step 2: Runs parallel Stock searches for each channel format
        (Instagram 1:1, LinkedIn 16:9, email header, display banner)
         ↓
Step 3: Applies Brand Kit Filter to each result set
         ↓
Step 4: Pre-applies Change color + Change mood previews
         ↓
Step 5: Returns curated shortlist: 3 assets per channel, licensed and resized
         ↓
Output: 12 campaign-ready assets, one-click license all
```

Why it lifts license rate: The agent collapses a 2-hour manual workflow (4 searches × 30 min each) into a single interaction. It licenses more assets per session because it surfaces needs the customer did not explicitly search for (the display banner asset the customer forgot to brief for).

Metric: Campaign Agent sessions → total assets licensed per session. Hypothesis: 3× single-search session.

---

**A2 — Variation Generator Agent** *(A/B testing enablement)*

```
Input: one licensed asset
         ↓
Step 1: Generates Firefly variations in 3 aesthetic directions
        (editorial warm / corporate clean / candid authentic)
         ↓
Step 2: For each direction, finds 3–5 licensed Stock equivalents
         ↓
Step 3: Returns 3×5 grid — 3 directions × 5 assets each
         ↓
Output: 15 campaign-ready variations, ready for A/B test launch
```

Why it lifts license rate: Current behavior — customer licenses 1 asset, runs it, tests nothing. With the Variation Agent, one license triggers 14 additional license opportunities. The agent converts a single-asset purchase into a campaign testing workflow.

Metric: Agent-triggered additional licenses per session. Target: 4–6 additional licenses per agent invocation.

---

**A3 — Brief-to-Board Agent** *(Shortens creative review cycle)*

```
Input: creative brief text or uploaded PDF
         ↓
Step 1: Extracts 4–5 distinct visual directions from the brief
         ↓
Step 2: Curates 3 assets per direction from Stock
         ↓
Step 3: Groups into a visual mood board
         ↓
Step 4: Annotates each direction with rationale ("this direction
        emphasizes the collaborative tone from brief section 2")
         ↓
Step 5: Calculates license cost per direction
         ↓
Output: Shareable PDF mood board with licensing options per direction
```

Why it lifts license rate: Creative review cycles kill license momentum. A customer who built a mood board, presented it, and got approval licenses within hours. A customer who is still in discovery licenses in days — or not at all. The Brief-to-Board Agent compresses the brief-to-approval cycle and attaches a licensing action directly to the approval moment.

Metric: Board-share → license completion rate (from shared board link). Target: 50% of shared boards result in at least one license within 48 hours.

---

**A4 — Content Refresh Agent** *(Repeat license enablement)*

```
Input: user's existing licensed asset library (or campaign folder)
         ↓
Step 1: Analyzes visual style of existing library
         ↓
Step 2: Identifies assets that are visually stale
        (overused composition patterns, trend-outdated visual register)
         ↓
Step 3: Finds fresh Stock assets in the same aesthetic register
         ↓
Step 4: Flags upcoming license expirations
         ↓
Output: "Refresh recommendations" — 10 fresh assets that match your
        library's style but update the visual language
```

Why it lifts license rate: Repeat licensing is the highest-margin revenue. A customer who licensed 20 assets 18 months ago and let them expire is not a lost customer — they are a customer with a known aesthetic preference and a demonstrated willingness to pay. The Refresh Agent converts license expirations into renewal opportunities.

Metric: Refresh Agent → license renewal rate. Target: 40% of customers who receive a refresh recommendation license at least one new asset within 30 days.

---

### 10c. License Rate Impact Summary

| Feature / Agent | License trigger mechanism | Estimated lift | Timeline |
|---|---|---|---|
| **Campaign Asset Agent** | Brief → multi-channel asset pack in one session | 3× assets per session | 90 days |
| **Brief-to-Licensed (F1)** | Firefly exploration → instant licensed equivalent | 4× vs. cold search | 90 days |
| **Brand Kit Filter (F2)** | Removes "almost right" barrier for enterprise | +35% enterprise license rate | 90 days |
| **Variation Generator Agent** | 1 license → 15 variation opportunities | 4–6 additional licenses per session | 6 months |
| **Campaign Coherence Score (F4)** | Cart-level gap → replacement license | 20% of flagged carts add asset | 60 days |
| **Brief-to-Board Agent** | Mood board share → approval → license | 50% board-share → license within 48h | 6 months |
| **Context-Aware Search (F3)** | Design-context precision reduces abandonment | 2.5× license rate vs. standard search | 6 months |
| **Content Refresh Agent** | License expiration → renewal opportunity | 40% refresh recommendation → new license | 6 months |
| **Negative Filtering (F5)** | Reduces browsing time → faster license decision | 1.8× license rate for aesthetic queries | 90 days |

> **⭐ The pattern across every feature and agent:** Each one eliminates a specific friction point between customer intent and license completion. The Campaign Asset Agent eliminates the multi-search session. Brand Kit Filter eliminates the Photoshop detour. Brief-to-Board eliminates the approval delay. None of these are "nice to have" engagement features — every one has a direct, measurable path to a license event.

---

## 11. Open Questions

1. **Dwell data:** Do we track dwell ≥ 8s + no add-to-cart on grid thumbnails today? This signal drives three solutions.
2. **Find Similar model:** Is the current similarity embedding aesthetic-aware at any dimension, or purely compositional?
3. **Firefly embedding access:** Is there an existing API to retrieve a visual embedding per Firefly generation, or is this greenfield?
4. **Action bar prioritization:** What drove the decision to put Change background in the primary bar and Change color/mood in overflow? Is there usage data that supports this hierarchy?
5. **North star:** Does the team track Asset Adoption Rate today, or is this open for the incoming PM to define?

---

*Bharat Namatherdhala · May 2026*
