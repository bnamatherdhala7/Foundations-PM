# Splunk AI Foundations — The AI Engine Room

**Product Requirements Document**

---

## Executive Summary

Splunk ingests machine data from every layer of the enterprise — networks, endpoints, applications, cloud infrastructure. The data problem is solved. The intelligence problem is not.

Three structural gaps prevent Cisco and Splunk from delivering on the AI-native digital resilience vision:

1. **Model gap**: General-purpose LLMs fail on machine data. SPL queries, time series anomalies, and network telemetry require domain-specific models — not prompt-engineered GPT-4.
2. **Connectivity gap**: AI skills are siloed per product. A security agent, an observability agent, and a network agent each live in separate surfaces with no shared protocol for cross-domain reasoning.
3. **Trust gap**: Operators won't hand control to autonomous agents they can't explain, audit, or override. "Self-driving" ops fails without human-in-the-loop governance baked into the architecture.

The core insight: **The AI Engine Room already has the fuel. The gap is orchestration, domain intelligence, and trust.**

Splunk has the data. Cisco has the network visibility. The platform has the tools. The gap is a unified agentic layer that connects them — powered by models trained on machine data, not web text — with a governance framework operators will actually trust.

---

## The Customer's Job

Enterprise operators are not trying to use AI. They are trying to **stop bad things from happening before they become incidents** — and resolve incidents before they become outages.

> *"I need to know what's wrong, why it's wrong, and what to fix it — before my pager goes off."*

**Three customer segments with distinct jobs:**

| Segment | Core Job | Current Reality |
|---|---|---|
| **SOC Analyst** | Detect and contain threats before blast radius expands | 500+ alerts/day, 97% false positives, 4-hour MTTR |
| **SRE / Platform Engineer** | Maintain service reliability without manual triage | Alert storms during incidents, no causal chain, toil-heavy runbooks |
| **Developer on the Platform** | Build AI-powered operational apps without ML expertise | No standard SDK, fragmented APIs, no agent framework |

**What they actually say:**
- *"I spend 80% of my time triaging noise, not investigating real threats."*
- *"The AI surfaces an anomaly but doesn't tell me if I should care."*
- *"I need to connect my Splunk data to my ticketing system, my runbooks, and my network tools — and right now those are three separate integrations."*
- *"I don't trust autonomous remediation. One bad action at 3am and I'm explaining an outage to my CISO."*
- *"I want agents that show their work — not a black box that tells me it fixed something."*

**The moment that matters most:**
```
ALERT FIRES → Analyst opens Splunk → Sees 500 correlated events → Manual investigation begins
     ↓
  2 hours later
     ↓
Root cause found → Runbook consulted → Manual fix applied → Incident closed
     ↓
MTTR: 4 hours. Blast radius: already expanded.
```

Every product decision should reduce the gap between alert and autonomous, auditable resolution.

---

## Key Problems Identified

**General-purpose models fail on machine data**: GPT-4 and Claude were trained on web text. Splunk data is time series telemetry, SPL syntax, OCSF event schemas, network flow records. Zero-shot prompting produces hallucinated SPL queries, missed anomaly patterns, and incorrect threat classifications. Domain-specific models trained on machine data are not a differentiator — they are the baseline requirement.

**Fragmented AI skills with no shared protocol**: A security detection agent in ES, an observability agent in ITSI, and a network agent in Cisco XDR each have proprietary interfaces. There is no standard protocol for cross-domain agent collaboration. An analyst investigating a security incident that spans network, endpoint, and application layers must context-switch between three surfaces manually.

**Agentic workflows without governance fail in production**: Enterprise operators have watched AI automation cause incidents — a misconfigured auto-remediation script, a hallucinated change recommendation applied at scale. They will not adopt autonomous agents without explainability (show the reasoning chain), controllability (human approval gates), and auditability (full action log). Trust is the product, not a feature.

**No developer platform for operational AI**: Customers and partners want to build AI-powered operational apps on top of Splunk. Today they stitch together REST APIs, manage their own vector stores, and build their own agent orchestration. There is no SDK, no agent framework, no standard deployment target. The ecosystem opportunity is unrealized.

---

## How Gen AI and Agentic Workflows Solve This

Three platform layers that together deliver self-driving operations with human trust:

---

### Layer 1 — Domain Intelligence (The Right Models for Machine Data)

**Problem it solves:** General-purpose LLMs fail on Splunk's core data types.

**What it delivers:**

**Cisco Time Series Foundation Model**
- Pre-trained on network telemetry, log time series, and operational metrics — not web text
- Native anomaly detection, forecasting, and root cause correlation on machine data
- Outperforms GPT-4 on SPL generation, threat classification, and capacity forecasting benchmarks
- Served as an API for internal teams and external customers

**Domain-Specific Capabilities:**
- SPL Copilot: natural language → validated SPL, grounded in the customer's actual schema
- Threat Intelligence Embeddings: vector representations of OCSF events, enabling semantic search across security telemetry
- Runbook Intelligence: encode institutional remediation knowledge into retrievable, executable artifacts

```
TODAY:    Analyst types: "show me lateral movement from last 24 hours"
          → Generic LLM generates plausible SPL
          → SPL fails on customer's field names
          → Analyst debugs for 20 minutes

DOMAIN MODEL: Analyst types same query
          → Model knows OCSF schema + customer's index structure
          → Returns validated, executable SPL in 3 seconds
          → Analyst investigates, not debugs
```

**Voice it addresses:** *"The AI surfaces something but it's always wrong for our environment."*

---

### Layer 2 — Universal Connectivity (MCP as the Standard Protocol)

**Problem it solves:** AI skills are siloed; cross-domain agent collaboration requires manual context-switching.

**What it delivers:**

**MCP-Standardized AI Skills**
- Every Splunk and Cisco product capability exposed as an MCP server: ES threat detection, ITSI service health, XDR network visibility, Splunk Observability
- Universal connector layer: any agent, internal or customer-built, can call any skill through a single protocol
- Skill registry: discoverable, versioned, documented — the npm for operational AI capabilities

**Cross-Domain Agent Collaboration:**
```
Security incident fires
          ↓
  [ SECURITY AGENT ]          [ NETWORK AGENT ]         [ OBSERVABILITY AGENT ]
  ES: 3 compromised hosts  →  XDR: C2 beacon on port  →  ITSI: App degraded
  OCSF threat events            443 to known bad IP          since same timestamp
          ↓
  [ ORCHESTRATOR AGENT ]
  Correlates across all three domains
  Builds unified incident timeline
  Proposes: isolate hosts + block IP + page app team
```

Without MCP: three separate dashboards, manual correlation, 90 minutes.
With MCP: unified cross-domain context in one agent session, 4 minutes.

**Voice it addresses:** *"I need to connect my Splunk data to my ticketing system, my runbooks, and my network tools — right now those are three separate integrations."*

---

### Layer 3 — Governed Agentic Workflows (Self-Driving Ops with Human Trust)

**Problem it solves:** Operators won't adopt autonomous agents they can't explain, audit, or override.

**What it delivers:**

**Agentic Workflow Framework**
- FSM-based orchestration: deterministic state transitions with full audit trail (not LLM-orchestrated chaos)
- Human-in-the-loop gates: configurable approval checkpoints by action risk tier
  - Tier 1 (read-only): auto-approved — query data, correlate events, generate timeline
  - Tier 2 (reversible): one-click approval — create ticket, add to watchlist, run diagnostic
  - Tier 3 (destructive): dual approval — isolate host, block IP, rollback deployment
- Explainability trace: every agent action shows the reasoning chain, data sources, and confidence score
- Rollback capability: any agent action at Tier 2+ is reversible with one click

**Four Production Agents:**

| Agent | Trigger | What it does | Human gate |
|---|---|---|---|
| **Incident Investigation Agent** | Alert fires | Correlates events across ES + ITSI + XDR, builds causal chain, proposes RCA | Review RCA before sharing |
| **Threat Hunt Agent** | Scheduled / ad hoc | Executes multi-step hunt across telemetry, surfaces TTPs, maps to MITRE ATT&CK | Approve before escalating |
| **Auto-Remediation Agent** | Confirmed incident | Executes playbook steps with rollback capability | Approve Tier 3 actions |
| **Capacity Forecast Agent** | Weekly | Runs time series model on resource trends, generates capacity plan | Review before purchasing |

**The governance model operators will trust:**
```
Agent proposes action
        ↓
Shows: "Based on [3 data sources], I recommend [action] because [reasoning]"
Shows: "Confidence: 87% | Risk tier: 2 | Reversible: Yes"
        ↓
Operator approves with one click
        ↓
Agent executes + logs action with full context
        ↓
Post-action: "Action completed. Impact: [measured]. Rollback available for 24h."
```

**Voice it addresses:** *"I don't trust autonomous remediation. I need agents that show their work."*

---

## How the Three Layers Work Together

```
INCIDENT: Spike in failed auth + network anomaly + app latency

                    [ DOMAIN MODEL ]
         Recognizes pattern as credential stuffing + lateral movement
         Generates: OCSF-grounded event timeline, confidence 91%
                         ↓
                  [ MCP CONNECTIVITY ]
         Calls: ES (threat events) + XDR (network flows) + ITSI (app health)
         Unifies cross-domain context in single session
                         ↓
              [ GOVERNED AGENTIC FRAMEWORK ]
         Proposes: isolate 3 hosts (Tier 3) + block 2 IPs (Tier 3) + page app team (Tier 2)
         Shows reasoning chain + data sources + confidence
         Analyst approves with context, not blind trust
                         ↓
         Agent executes + logs every action
         MTTR: 4 hours → 18 minutes
```

---

## Solution Tiers

**Tier 1 (45–60 days, no model training required):**
- SPL Copilot GA: natural language → validated SPL using few-shot prompting on customer schema
- MCP server for ES and ITSI: expose core AI skills via standard protocol
- Human-in-the-loop approval framework: configurable gates by action risk tier
- Explainability trace UI: show agent reasoning chain on every action

**Tier 2 (90 days, cross-team):**
- Cisco Time Series Model v1: anomaly detection + forecasting on Splunk telemetry, customer-deployable
- MCP skill registry: discoverable, versioned catalog of all Splunk + Cisco AI capabilities
- Incident Investigation Agent GA: cross-domain correlation across ES + ITSI + XDR
- Developer SDK v1: agent framework + skill connectors for external builders

**Tier 3 (6 months, model-dependent):**
- Time Series Model v2: retrained on production telemetry with LoRA adapters per customer environment
- Auto-Remediation Agent with full rollback: Tier 2 and Tier 3 action execution
- Threat Hunt Agent: autonomous multi-step investigation with MITRE ATT&CK mapping
- Ecosystem program: certified MCP skill partners, hyperscaler integrations (AWS Bedrock, Azure AI Foundry)
- Responsible AI governance dashboard: explainability scores, bias monitoring, data lineage per model

---

## Platform Capabilities for Developers

**What we ship to the developer ecosystem:**
- **AI Foundations SDK**: agent framework, skill connectors, vector store integration — build operational AI apps without ML expertise
- **MCP Skill Marketplace**: browse, deploy, and extend certified AI skills across Cisco + Splunk + partner ecosystem
- **Model API**: domain-specific models (Time Series, SPL, threat intelligence) accessible via REST — no Splunk deployment required
- **Evaluation Framework**: benchmark kit for customers to measure model performance against their own telemetry

**The developer flywheel:**
```
Great platform → Developers build on it → Partners extend it → More skills in registry
→ More cross-domain agent capability → More customer value → More platform adoption ↺
```

---

## Responsible AI & Governance

Every capability ships with a governance layer — this is not a compliance checkbox, it is a product requirement for enterprise adoption.

| Requirement | Implementation |
|---|---|
| **Explainability** | Reasoning trace on every agent action: data sources, confidence, uncertainty |
| **Auditability** | Immutable action log: who approved, what ran, what changed, rollback status |
| **Data residency** | On-prem model deployment option for air-gapped and regulated customers |
| **Bias monitoring** | Automated detection of performance drift across customer environments |
| **Model transparency** | Training data provenance, evaluation benchmarks published to customers |
| **Human override** | Any agent action terminable mid-execution; full rollback for reversible actions |

---

## North Star Metric

**Autonomous Resolution Rate (ARR)**: The percentage of incidents fully resolved by the agentic workflow — investigation, root cause, remediation — without human action beyond initial approval gates.

This captures whether the platform actually *works* — not whether agents ran, but whether operations teams trusted them enough to let them finish.

> ARR at 30% means 30% of incidents resolved autonomously. At 60%, operators have fundamentally changed how they work. That is the product-market fit signal.

---

## Success Targets

| Metric | Baseline | Target | Why it matters |
|---|---|---|---|
| MTTR (mean time to resolve) | 4 hours | 18 minutes | The headline metric for ops teams and CISOs |
| Alert-to-investigation time | 45 minutes | 3 minutes | Incident Investigation Agent speed |
| SPL generation accuracy | ~40% valid first-try | 90% valid first-try | Domain model vs. general LLM |
| Agent action approval rate | — | >80% (operators approve agent proposals) | Trust signal — low approval = low confidence in agent |
| Developer SDK adoption | 0 | 500 external apps in 12 months | Ecosystem health |
| Autonomous Resolution Rate | 0% | 30% at 6 months | North star |
| Cross-domain agent sessions | 0 | 40% of investigations | MCP connectivity adoption |

---

## Critical Dependencies

**Telemetry instrumentation first.** Agent reasoning quality depends on rich event context. Without structured OCSF event logging, agent dwell signals, and action outcome tracking, the Time Series Model cannot be retrained and the Auto-Remediation Agent has no feedback loop. This is the zero-blocker — instrument it in parallel with Tier 1 surface work.

**MCP standardization alignment.** The connectivity layer only delivers value if all Cisco and Splunk product teams adopt MCP as the standard interface for AI skills. This is an organizational alignment dependency, not a technical one. Needs executive sponsorship.

---

## The Strategic Prize

Cisco and Splunk together have what no competitor can assemble from scratch: network visibility, security telemetry, operational data, and a 20,000-customer enterprise footprint.

The risk: without the AI Engine Room, that data advantage stays locked in dashboards. The opportunity: with domain models, MCP connectivity, and governed agentic workflows, Cisco and Splunk become the operating system for AI-native digital resilience.

> **Every other platform gives operators data and dashboards. The AI Engine Room gives operators an autonomous partner that investigates, reasons, and acts — with the operator always in control.**

That is not a better SIEM. That is a different category.
