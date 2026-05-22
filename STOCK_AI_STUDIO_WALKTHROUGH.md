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

1. **Discovery gap** — customers cannot translate a creative brief into the right asset. Keyword search returns technically accurate results that are creatively wrong. Natural language intent is lost at the query layer.

2. **Customization gap** — customers find an asset they want but cannot use it without 30–45 minutes of manual editing in Photoshop. The tools to fix this (Change Color, Remove Background, Generative Fill) exist inside AI Studio but are invisible at the moment of purchase friction.

> **The opportunity:** AI Studio is not missing capabilities. It is missing workflow integration — each tool lives at a separate destination instead of appearing at the moment the customer needs it.

---

## 2. Goals

| Goal | Metric | Target |
|---|---|---|
| Increase discovery quality | Search-to-license conversion rate (semantic queries) | 2× keyword baseline |
| Recover abandoned purchase intent | Abandoned session → license conversion | ≥ 30% of dwell ≥ 8s cohort |
| Connect Firefly generation to Stock licensing | Firefly session → Stock license rate | 3× cold-search baseline |
| Reduce post-download editing time | Asset Adoption Rate (used in campaign without Photoshop edit) | +25% in 6 months |

### North Star Metric: Asset Adoption Rate

> ⭐ Downloads measure what was **available**. Adoption measures whether the workflow actually **worked**.

A customer who downloads an asset and immediately opens Photoshop for 45 minutes of manual editing is a failed discovery. A customer who downloads and routes straight to their campaign manager is a successful one. Adoption Rate is the only metric that captures the difference.

---

## 3. Customer Segments

### Segment A — Enterprise Brand Team *(Highest value)*
Fortune 500 in-house agencies and brand managers. License hundreds of assets per month for global campaigns. Have a brand guide — specific colors, visual register, composition rules. Their problem: Stock returns results that are technically correct and creatively wrong. They spend more time searching than creating.

### Segment B — In-House Creative *(Volume user)*
Designers at agencies and mid-size companies. On a deadline. They find an asset that is compositionally perfect but the wrong color, or the background doesn't work. They leave Stock, edit in Photoshop for 30–45 minutes, and return. Every edit is a detour that reduces likelihood of repeat purchase.

### Segment C — SMB Marketer / Content Creator *(Growth segment)*
No dedicated designer. Needs something professional and on-brand in 10 minutes. Does not know the right keywords — describes what they want in natural language. Gets results that look like a generic catalog from 2015.

---

## 4. Pain Points

### Pain Point 1 — Keyword search does not understand creative intent

**Behavior:** User types *"confident woman entrepreneur warm editorial coffee shop natural light."* Stock parses nouns. Returns 3,000 technically accurate, creatively wrong results. Creative director rejects the batch.

**Root cause:** The current search embedding is trained on object-label pairs, not on aesthetic register — editorial vs. commercial, candid vs. staged, warm vs. clinical. These properties determine creative fit but do not survive keyword search.

> **⭐ Key insight:** The training signal that closes this gap already exists in our data. **Dwell + license pairs** — users who viewed an asset for ≥ 8 seconds before purchasing — encode "this felt right" in a way that download-only data cannot. We are not capturing this behavioral signal today.

**Impact:** Every session where a user searches, browses 5+ minutes, and exits without licensing is a lost transaction. On a 300M-asset catalog, search quality is the product.

---

### Pain Point 2 — Change Color tool is invisible at the moment it matters

**Behavior:** Customer finds a hero image — perfect composition, wrong color for their brand. Does not know the Change Color tool exists at `stock.adobe.com/ai-studio/images/change-color`. Downloads the asset, opens Photoshop, masks and recolors manually. 45 minutes for a decision that should take 30 seconds.

**Root cause:** AI Studio tools are destinations, not capabilities. They require intentional navigation rather than appearing at the moment of purchase friction.

> **⭐ Key insight:** The fix is a **placement decision, not an engineering project.** When a user dwells on an asset for ≥ 8 seconds without adding to cart, surface one line inline: *"Wrong color for your brand? Change it with AI."* No new model work. No backend changes. Frontend event trigger + one UI component. This is a 60-day revenue recovery on intent that already exists.

**The "almost bought" cohort:** Users who dwell ≥ 8s + do not add to cart are the highest-intent non-purchasers in the funnel. This cohort is identifiable today from existing session data and represents direct recoverable revenue.

---

### Pain Point 3 — Firefly generation and Stock licensing are disconnected

**Behavior:** Designer generates an image in Firefly that captures exactly the right look for a campaign. For paid media placement — which requires IP indemnification — they need a licensed Stock asset. Today: start a completely new Stock search from scratch. The creative intent encoded in the Firefly output is lost.

> **⭐ Key insight:** No competitor can build this connection. Midjourney has generation but no licensed library. Getty has a library but no generation. **Only Adobe has both — and right now they do not talk to each other.** "Find Similar in Stock" is a structural moat, not a feature.

**The conversion case:** A user arriving at Stock from a Firefly generation has already made the creative decision — they just need the licensed version. This is the highest-intent Stock customer Adobe has. Hypothesis: conversion from this flow is **3× cold-search baseline** because the creative decision is already made.

**Cross-team dependency:** Firefly (visual embedding export) + Stock engineering (similarity search). PM role: define the embedding contract, let each team own implementation.

---

### Pain Point 4 — Campaign teams cannot find variation sets

**Behavior:** A digital campaign team needs 5 visual directions for A/B testing a hero image. Each direction requires a separate search, separate filtering, separate evaluation session. One hour of search work before a single creative decision is made.

> **⭐ Key insight:** This is the same architecture problem solved in StoryForge — a multi-agent variation engine: one input → 10 strategically differentiated outputs. Variation breadth requires a fundamentally different architecture than ranked search results: **one brief → clustered directions**, not 50 variations of the same composition.

---

### Pain Point 5 — Video b-roll search is keyword-trapped

**Behavior:** Video editor needs *"confident woman working, warm natural light, shallow depth of field, editorial pacing, 10–15 seconds."* Types exactly that. Gets generic results. Spends 40 minutes browsing.

**Root cause:** Video requires temporal understanding — motion quality, pacing, depth-of-field change over time. These properties do not survive keyword search or standard visual embeddings.

> **⭐ Key insight:** Video time-to-license is estimated 3–4× longer than image today. Closing that gap is a direct margin improvement on the highest-value subscription tier. The dimensional framework for this — motion smoothness, subject consistency, temporal coherence — is the same framework used in Firefly video evaluation (VBench). A PM who understands both can coordinate the research across teams without duplicating effort.

---

## 5. Current AI Studio Feature Map

| Feature | Works well | The gap |
|---|---|---|
| **Text to Image** | Creative exploration, concept mockups | No path to licensed Stock assets matching the result |
| **Generate Similar** | Compositional matches | Misses aesthetic register — returns objects, not feel |
| **Remove Background** | Product photography cut-outs | Separate destination, not surfaced in search flow |
| **Change Color** | Brand color matching, lighting-aware recolor | Separate URL — invisible at purchase abandonment point |
| **Generative Fill / Expand** | Layout adaptation, format resizing | Not connected to brand style — every fill starts from scratch |

> **Pattern across all five:** The models work. The gap is always the same — tools are destinations instead of capabilities that appear when the customer needs them.

---

## 6. Proposed Solutions

### Solution 1 — Inline Change Color at abandonment *(Tier 1 · 60 days)*

**Trigger:** User dwells ≥ 8s on asset + does not add to cart.
**Action:** Surface inline: *"Wrong color for your brand? Change it with AI — free preview."*
**Cost:** Frontend event tracking + one UI component. Zero model work. Zero backend changes.
**Metric:** Abandoned session → license conversion. Target: ≥ 30% of the ≥ 8s non-purchaser cohort.

---

### Solution 2 — AI Studio shortcuts on search result cards *(Tier 1 · 60 days)*

**What:** "Edit with AI Studio" action on every search result card — surfaces Remove Background, Change Color, Generate Similar directly from results.
**Cost:** Frontend addition only.
**Metric:** AI Studio activation rate per search session. Target: +25%.

---

### Solution 3 — Firefly → "Find Similar in Stock" bridge *(Tier 2 · 90 days)*

**What:** After any Firefly generation, a Stock similarity panel returns 8–12 licensed assets matching the generated image's visual embedding.
**Cross-team:** Firefly exports a visual embedding per generation. Stock search queries against it.
**Metric:** Firefly session → Stock license rate. Target: 3× cold-search baseline.

> ⭐ **This is the only feature in the AI image generation market that no competitor can replicate.** It requires owning both the generation model and the licensed library simultaneously.

---

### Solution 4 — Campaign variation set *(Tier 2 · 90 days)*

**What:** One prompt → 5 conceptually distinct visual directions, each with 3–5 representative assets.
**Metric:** Assets viewed per session, downstream A/B test launch rate for enterprise users.

---

### Solution 5 — Semantic search on dwell + license signal *(Tier 3 · 6 months)*

**What:** Fine-tune the Stock search embedding on dwell + license behavioral pairs. Captures aesthetic register, not just object labels.
**Metric:** Search-to-license conversion, natural language queries. Target: 2× keyword baseline.

---

### Solution 6 — Video temporal semantic search *(Tier 3 · 6 months)*

**What:** Video-native embeddings encoding motion quality, pacing, depth of field, subject consistency.
**Metric:** Video time-to-license. Target: reduce from 4× image baseline to 1.5× image baseline.

---

## 7. Roadmap

```
60 DAYS                    90 DAYS                    6 MONTHS
──────────────────         ─────────────────          ──────────────────────
Tier 1 · No model work     Tier 2 · Cross-team        Tier 3 · Model-dependent
──────────────────         ─────────────────          ──────────────────────
Inline Color Change    →   Firefly → Stock        →   Semantic search retrain
at abandonment moment      bridge (Find Similar)       (dwell + license signal)

AI Studio shortcuts    →   Campaign variation     →   Video temporal embeddings
on search result cards     set (1 brief → 5 dir)
                                                       Batch brand transformation
                                                       (enterprise)
```

**Why Tier 1 first:** Maximum revenue impact, minimum engineering cost. The "almost bought" cohort is identifiable today. Intervention requires no new AI infrastructure. Results measurable within 2 weeks of shipping.

---

## 8. Success Metrics

| Metric | Definition | Target | Signals |
|---|---|---|---|
| **Asset Adoption Rate** *(north star)* | Assets used in campaign without post-download editing | +25% in 6 months | Workflow success |
| Abandoned session conversion | Dwell ≥ 8s + no cart → license after prompt | ≥ 30% | Revenue recovery |
| Semantic query conversion | Natural language search → license | 2× keyword baseline | Search quality |
| Firefly → Stock conversion | Firefly session → Stock license | 3× cold-search | Cross-Adobe bridge value |
| Video time-to-license | Search → first video license | ≤ 1.5× image baseline | Video parity |
| AI Studio activation per session | Users interacting with ≥ 1 AI tool | 25% → 50% | Discoverability |

---

## 9. Dependencies & Risks

| Dependency | Owner | Risk if unresolved |
|---|---|---|
| Visual embedding export from Firefly | Firefly engineering | Solution 3 blocked |
| Dwell + license behavioral signal | Data / Analytics | Solutions 1 and 5 blocked |
| Batch processing pipeline | Stock engineering | Enterprise batch transformation blocked |
| Custom Models API integration | Firefly platform | Brand style in batch workflow blocked |

**Primary risk:** Optimizing for search engagement instead of campaign adoption. Engagement metrics can rise while the customer still edits in Photoshop for 45 minutes after every download. Mitigation: track Asset Adoption Rate as a downstream check on every engagement metric.

---

## 10. Open Questions for Discovery

1. What does current dwell data look like for the "almost bought" cohort — do we have event tracking on dwell ≥ 8s + no add-to-cart today?
2. How is the Firefly / Stock engineering relationship structured — shared embedding infrastructure, or greenfield dependency?
3. Does the team have a north star metric for AI Studio today, or is metric definition an open question for the incoming PM?

---

*Bharat Namatherdhala · May 2026*
