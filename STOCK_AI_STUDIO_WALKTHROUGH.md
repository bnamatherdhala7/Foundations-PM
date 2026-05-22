# Adobe Stock AI Studio — GenAI Discovery & Customization

**Product Requirements Document**

---

## Executive Summary

Adobe Stock serves 300M+ assets to a two-sided marketplace — creators who supply content and buyers who license it. This PRD focuses on the **buyer side**, where the AI Studio opportunity is concentrated.

Three structural gaps drive buyer drop-off today:

1. **Discovery gap**: Search returns 664,000+ results with no aesthetic filtering — buyers spend more time searching than designing
2. **Customization gap**: AI editing tools exist but appear too late — after users have already abandoned the asset on the grid
3. **Trust gap**: Buyers generate assets they love, then hesitate at the license step because legal coverage for AI-generated content is unclear

The core insight: **Adobe Stock doesn't have a capability problem. It has a timing and trust problem.**

Adobe already has Change Color, Change Mood, Firefly generation, and AI Studio. The problem is these tools appear on detail pages — but buyers make keep-or-skip decisions in seconds on the search grid. Moving AI to the decision moment, and adding license certainty at the generation moment, is the unlock.

---

## The Buyer's Job

Buyers are not purchasing art. They are purchasing **certainty** — that the right asset is available, legally cleared, and won't require manual editing before production use.

> *"I need the right visual, legally cleared, fast — without leaving my workflow."*

**What buyers actually say:**
- *"I spend more time searching than designing."*
- *"Perfect composition, wrong color — I give up and go to Getty."*
- *"I generated exactly what I needed — but can I actually use it commercially?"*
- *"My legal team won't touch anything marked AI-generated."*
- *"Where did I save that? I'll have to start over."*

**The conversion drop-off that costs the most:**
```
GENERATE → "I love it" → LICENSE → "But can I actually use it?" → ABANDON
```
This moment — between generation and license confirmation — is where buyer trust breaks. Every product decision should reduce friction here.

---

## Key Problems Identified

**Grid Abandonment**: Users see thumbnails and decide within seconds. Current hover state shows only favorite, compare, and overflow menu — no AI actions. A user thinking "perfect composition, wrong color" has no path forward and abandons the asset.

**Hidden High-Value Actions**: "Change color" and "Change mood" sit in overflow menus most users never open, despite being the two most immediately useful tools for in-house creatives.

**License Ambiguity at the Moment of Generation**: Buyers don't know if IP indemnification applies to AI-generated content. Enterprise buyers cite legal ambiguity as the #1 reason they revert to Getty or Shutterstock.

**No Generation Memory**: Generated assets live nowhere. Buyers have no way to revisit, iterate, or share outputs — every session starts from scratch.

**Passive AI Badge**: Firefly-generated images are marked but not actionable. No path exists from AI results to licensed equivalents or variations.

**Weak Similar-Asset Suggestions**: The 25-tag keyword cloud on detail pages ("woman, business, leadership") describes *content*, not *aesthetic direction*.

---

## How Gen AI and Agentic Workflows Solve This

The solution is three agentic layers that work together across the buyer session:

### Layer 1 — Intent Agent (Find or generate the right asset)

**Problem:** Buyers waste time on bad prompts and irrelevant keyword results.

**What it does:**
- Interprets natural language brief → refines prompt automatically
- Pulls from catalog *and* Firefly generation in one unified results surface
- Replaces keyword tag clouds with aesthetic direction chips: *"Warm & editorial," "Corporate & clean," "Candid & authentic"*
- Learns buyer preferences across sessions

```
TODAY:    Buyer types keywords → 664,000 results → scrolls → abandons
AGENTIC:  Buyer describes project → agent surfaces 3 targeted options → done
```

### Layer 2 — License Certainty Agent (Clear without clicking)

**Problem:** Legal hesitation at the license step kills conversion after generation.

**What it does:**
- Reads buyer's plan + use case → renders plain-language clearance statement inline at the generation moment
- *"Cleared for: social, web, print, broadcast ✓ — Not cleared for: resale"*
- Flags edge cases proactively before the buyer downloads
- Stores a license receipt automatically in the buyer's history

```
TODAY:    Generate → read T&Cs → ask legal → maybe use it
AGENTIC:  Generate → see "Cleared for broadcast ✓" → use it
```

### Layer 3 — Creative Workflow Agent (From asset to production)

**Problem:** Generated assets are stranded — no memory, no handoff, no iteration.

**What it does:**
- Auto-saves generation history organized by project brief
- "Continue this" — reloads session context, prompt, and prior variations
- One-click push to Photoshop, Express, or Frame.io with license metadata attached
- Auto-sizes and formats for detected channel (LinkedIn post vs. billboard vs. video thumbnail)

```
TODAY:    Generate → download → lose it → start over next week
AGENTIC:  Generate → project saved → open in Photoshop → iterate next sprint
```

### How the three agents work together

```
Buyer types: "Q3 campaign — female founder, NYC, clean tech feel"
                          ↓
              [ INTENT AGENT ]
         Refines prompt → 3 aesthetic directions
         (catalog + Firefly, ranked by brief match)
                          ↓
              Buyer picks one asset/direction
                          ↓
         [ LICENSE CERTAINTY AGENT ]
         "Cleared for: web, social, print ✓"
              Stored in license history
                          ↓
         [ CREATIVE WORKFLOW AGENT ]
          Opens in Photoshop, sized for
          LinkedIn + OG image + Print
          Saved to "Q3 Campaign" project

Total time: under 5 minutes. No T&Cs. No re-downloads. No legal review.
```

---

## Solution Tiers

**Tier 1 (45–60 days, no model work)**:
- Bring Change color, Change mood, Change background to grid hover — primary position, not overflow
- Replace keyword tags with aesthetic direction chips (*"Warm & editorial," "Corporate & clean"*)
- Reframe "More from this series" to show conceptually related but visually distinct assets

**Tier 2 (90 days, cross-team)**:
- Make AI badge actionable: generate variations or find licensed equivalents from Firefly output
- Brief-to-Asset: paste campaign copy → see Firefly direction + licensed equivalents side-by-side
- Enable image-as-query for video search

**Tier 3 (6 months, model-dependent)**:
- Campaign Asset Agent: brief → 12 resized, ready-to-license assets per channel
- Variation Generator Agent: 1 asset → 15 variations across 3 aesthetic directions
- Brief-to-Board Agent: creates shareable mood boards with embedded licensing
- Content Refresh Agent: identifies stale assets and suggests fresh replacements
- Retrain semantic search on dwell + license signals
- Build aesthetic-aware Find Similar with video temporal embeddings

---

## New AI Features

- **Brief-to-Asset**: Paste campaign copy → see Firefly direction + licensed equivalents side-by-side
- **Brand Kit Filter**: Upload brand guide → automatically rerank all results by brand match
- **Context-Aware Search**: Upload design layout → get assets tailored to that placement
- **Campaign Coherence Score**: Check multi-asset purchases for visual consistency
- **Negative Semantic Filtering**: Search with aesthetic exclusions, not just keywords

---

## The Two-Sided Marketplace Dynamic

Buyers don't experience Adobe Stock as a two-sided marketplace — they experience it as a content machine that either delivers or doesn't. But the creator-side problems bleed through:

| Creator Problem | How Buyers Experience It |
|---|---|
| AI floods the catalog | Search quality drops — too many generic AI images |
| Creator supply thins | Niche categories go empty |
| Inconsistent review | Unusable assets surface in results |

**The strategic implication for AI Studio:** AI Studio partially bypasses catalog dependency — buyers stop depending on what creators supply and generate what they need. This only works if buyers trust the output quality and the license coverage. Fixing buyer trust is also the lever that stabilizes the two-sided marketplace.

---

## North Star Metric

**Asset Adoption Rate**: Assets used in campaigns without post-download Photoshop editing.

This captures whether the workflow actually *worked* — not just whether users clicked more buttons. A downloaded asset that required 45 minutes of editing to become usable is not a successful outcome.

---

## Success Targets

| Metric | Target | Why it matters |
|---|---|---|
| AI Studio grid activation | 25% of sessions | Proves surface area fix worked |
| Aesthetic chip → license rate | 2× keyword chip baseline | Proves intent signal quality |
| Asset Adoption Rate | +25% in 6 months | Proves the workflow actually worked |
| Campaign Agent license lift | 3× assets per session | Revenue impact |
| Brief-to-Licensed conversion | 4× vs. cold search | Competitive benchmark vs. Getty |
| Post-generation license conversion | +30% | Proves License Certainty Agent closes the trust gap |

---

## Critical Dependency

**Dwell + license event tracking must ship first.** Without this signal, semantic search retraining, aesthetic reranking, and all Tier 3 agentic logic cannot proceed. This is the zero-blocker — instrument it in parallel with Tier 1 surface work.

---

## The Strategic Prize

> Adobe Stock becomes the only place where a buyer can go from a campaign brief to a licensed, production-ready asset in under 5 minutes — without a lawyer, without a designer, without leaving Adobe.

That is not a better search. That is a different product.
