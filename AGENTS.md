# Agents Used

This system runs **60 AI agents**: 52 reasoning agents plus 8 specialist *agent-tools*, spread
across 11 agent-driven workflows. A 12th workflow (the Global Guardrail) is a deterministic,
rule-based governance engine with no LLM agents — by design, the authority that judges agents
is not itself an agent.

---

## The agent pattern

Every reasoning agent is built from the same three-node primitive in n8n:

```
   ┌──────────────────────┐      ai_languageModel     ┌─────────────────────┐
   │  OpenAI Chat Model    │ ───────────────────────▶ │                     │
   └──────────────────────┘                            │   Agent (LLM)       │ ──▶ typed JSON
   ┌──────────────────────┐      ai_outputParser       │   + system prompt   │     output
   │ Structured Output     │ ───────────────────────▶ │                     │
   │ Parser (JSON schema)  │                            └─────────────────────┘
   └──────────────────────┘
```

- **Chat Model** — an OpenAI model, all bound to a single shared credential.
- **Agent** — carries the role's system prompt (its "job description").
- **Structured Output Parser** — forces the model to return **typed JSON matching a schema**,
  not free text. This is what lets one agent's output become the next agent's input reliably.

Agents also **auto-retry** transient model failures, so a single flaky response doesn't break a
department.

## Two ways agents are organized

**1) Sequential specialist chains** (most departments)
A department orchestrator hands off to analysts, who hand off to operational agents — each step
enriching the same decision contract. Example (Finance): `CFO Orchestrator → Finance Intelligence
→ Cost Audit → Risk Analysis → Trading Ops → Risk Ops`.

**2) Orchestrator-with-tools** (Marketing)
One **CMO Orchestrator** agent treats eight specialist agents as callable **tools**, invoking
them as needed rather than in a fixed line. This shows the system supports both pipeline and
tool-calling agent topologies.

## The hierarchy

Agents are layered like an org chart:

```
 L1  CEO Orchestrator ........................ routes every request
 L2  Department heads (CFO, COO, CMO, CSO, CHRO, CTO, CIO, AFA, CBS, CREA)
 L3  Specialist analysts ..................... domain reasoning
 L4  Operational agents ...................... execution-level tasks
 ──  Global Guardrail (rule-based) ........... approves/blocks/logs all of the above
```

## Full roster

### L1 — Orchestration
| Agent | Workflow | Role |
|-------|----------|------|
| CEO Agent | WF-0 | Classifies intent, picks the department, sets risk/urgency, builds the decision contract |

### WF-1 · Strategy (CBS)
| Agent | Layer | Role |
|-------|-------|------|
| CBS Agent – Strategy Evaluation | L2 | Evaluates strategic options and trade-offs |
| CBS Agent – Final Recommendation | L2 | Consolidates inputs into a final recommendation |

### WF-2 · Finance (CFO)
| Agent | Layer | Role |
|-------|-------|------|
| CFO – Finance Orchestrator | L2 | Owns the financial decision |
| Finance Intelligence Agent | L3 | Financial analysis & insight |
| Cost Audit Agent | L3 | Cost review and control |
| Risk Analysis Agent | L3 | Financial risk assessment |
| Trading Ops Agent | L4 | Trading/transaction operations |
| Risk Ops Agent | L4 | Risk operations |

### WF-3 · Operations (COO)
| Agent | Layer | Role |
|-------|-------|------|
| Demand Forecasting Agent | L3 | Predicts demand |
| Supply Chain Intelligence Agent | L3 | Supply-chain analysis |
| Replenishment Calculation Agent | L3 | Computes replenishment needs |
| Packaging Operations Agent | L4 | Packaging decisions |
| Logistics Agent | L4 | Logistics planning |
| Routing Operations Agent | L4 | Fulfilment routing |
| Stock Monitor Agent | L4 | Inventory monitoring & reorder triggers |

### WF-4 · R&D (AFA)
| Agent | Layer | Role |
|-------|-------|------|
| Ingredient Discovery Agent | L3 | Researches inputs/components |
| Formula Suggestions Agent | L3 | Proposes formulations |
| Lab Simulation Agent | L3 | Simulates lab results |
| Stability Simulation Agent | L3 | Simulates stability |
| Safety Assessment Agent | L3 | Safety evaluation |
| Compliance Check Agent | L3 | Regulatory compliance |
| Cost Optimization Agent | L4 | Optimizes formulation cost |
| AFA Final Approval Agent | L2 | Final R&D sign-off |

### WF-5 · Marketing (CMO) — *orchestrator + tools*
| Agent | Layer | Role |
|-------|-------|------|
| CMO Orchestrator Agent | L2 | Drives the campaign, calls the tools below |
| Market Intelligence Agent *(tool)* | L3 | Market research |
| Trend Analysis Agent *(tool)* | L3 | Trend detection |
| Content Creator Agent *(tool)* | L3 | Content generation |
| Designer Agent *(tool)* | L3 | Creative/visual direction |
| Writer Agent *(tool)* | L3 | Copywriting |
| SEO Ops Agent *(tool)* | L4 | SEO operations |
| Campaign Ops Agent *(tool)* | L4 | Campaign execution |
| Content Strategy Agent *(tool)* | L3 | Content strategy |

### WF-6 · Sales (CSO)
| Agent | Layer | Role |
|-------|-------|------|
| CSO Orchestrator Agent | L2 | Owns the sales decision |
| Lead Qualification Agent | L3 | Scores/qualifies leads |
| Pricing Strategy Agent | L3 | Pricing decisions |
| Sales Rep Agent | L4 | Standard sales handling |
| Senior Sales Rep Agent | L4 | Complex/high-value deals |
| Partnerships Agent | L3 | Partnership opportunities |
| Write Email for Leads Agent | L4 | Outbound lead emails |

### WF-7 · Customer Experience (CEA)
| Agent | Layer | Role |
|-------|-------|------|
| Shade Matching Agent | L3 | Product-matching logic |
| Fallback Questionnaire Agent | L3 | Guided fallback when confidence is low |
| Beauty Advisory Agent | L3 | Product advisory |
| Customer Sentiment Agent | L3 | Sentiment analysis |
| Support Ops Agent | L4 | Support handling |
| QA Ops Agent | L4 | Quality assurance |
| User Journey Agent | L3 | Journey/experience mapping |

### WF-8 · Risk & Ethics (CREA) — *the conscience layer*
| Agent | Layer | Role |
|-------|-------|------|
| Legal Compliance Agent | L3 | Legal review |
| Safety Override Agent | L3 | Safety veto authority |
| Policy Validation Agent | L3 | Policy conformance |
| Bias Detection Agent | L3 | Detects biased/unfair outputs |
| Audit Ops Agent | L4 | Audit trail operations |

### WF-9 · Technology & Infrastructure (CTO + CIO)
| Agent | Layer | Role |
|-------|-------|------|
| CTO Orchestrator Agent | L2 | Technology decisions, health & security checks |
| CIO Orchestrator Agent | L2 | Infrastructure, data, compliance & backups |

### WF-HR · Human Resources (CHRO)
| Agent | Layer | Role |
|-------|-------|------|
| CHRO Orchestrator Agent | L2 | Owns HR decisions |
| Talent Insights Agent | L3 | Talent analytics |
| Culture Engagement Agent | L3 | Culture & engagement |
| Policy Enforcement Agent | L3 | HR policy enforcement |
| Hiring Ops Agent | L4 | Hiring operations |
| Performance Review Agent | L4 | Performance reviews |

## How the agents connect

Agents don't just talk inside their own department — departments consult each other, and
everything answers to governance.

```
                         WF-0 CEO
                            │ routes to one department
   ┌──────────┬──────────┬──┴───────┬──────────┬──────────┬──────────┐
  WF-1       WF-2       WF-3       WF-4       WF-5       WF-6 …      WF-HR
 Strategy  Finance   Operations    R&D      Marketing   Sales        HR

   cross-department consultations (via Execute Workflow):
     WF-1 Strategy   ─▶ WF-2, WF-3, WF-4, WF-5, WF-6, WF-8
     WF-3 Operations ─▶ WF-2  (CFO approval)
     WF-4 R&D        ─▶ WF-8 (Risk), WF-3 (Ops), WF-5 (Marketing)
     WF-5 Marketing  ─▶ WF-8  (claim validation)
     WF-6 Sales      ─▶ WF-2  (Finance)
     WF-7 CX         ─▶ WF-4  (R&D escalation)
     WF-9 Tech/Infra ─▶ WF-10 (governance signal)

   governance (every decision, via HTTP webhook):
     WF-0  ─▶ WF-10   approval gate  (before routing)
     WF-0  ─▶ WF-10   execution gate (before the department's action runs)
```

- **Inside a department:** agents are connected by n8n's `ai_languageModel` / `ai_outputParser`
  links (model + parser feeding each agent) and chained by `main` connections (one agent's
  typed output flowing into the next).
- **Between departments:** an **Execute Workflow** node calls the other department as a
  sub-workflow, passing the decision contract; the callee runs its own agents and returns.
- **To governance:** WF-0 and WF-9 call WF-10 over **HTTP** (it is webhook-triggered), so the
  guardrail can also be reached as a standalone service.
- **Risk & Ethics (WF-8)** is never routed to directly — it is *escalated to* by other
  departments when an action needs a compliance/safety/bias ruling, acting as a shared conscience.
