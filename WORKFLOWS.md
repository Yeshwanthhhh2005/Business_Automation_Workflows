# About the Workflows

The system is 12 n8n workflows: one orchestrator, one governance authority, and ten
departments. Each file is a self-contained, importable workflow; they are wired together by
workflow ID (cross-workflow calls) and by HTTP (the governance webhook).

---

## The 12 at a glance

| # | Workflow | Role | Agents | Entry | Calls |
|---|----------|------|:------:|-------|-------|
| 0 | **CEO Orchestrator** | Front door — classify & route every request | 1 | Webhook / Schedule / Manual | WF-10, then one department |
| 1 | **Strategy (CBS)** | Strategic evaluation & multi-dept synthesis | 2 | Execute Workflow | WF-2,3,4,5,6,8 |
| 2 | **Finance (CFO)** | Financial decisions, cost & risk control | 6 | Execute Workflow | WF-2 internal |
| 3 | **Operations (COO)** | Demand, inventory, fulfilment | 7 | Execute Workflow | WF-2 |
| 4 | **R&D (AFA)** | Product research, formulation & compliance | 8 | Execute Workflow | WF-8, WF-3, WF-5 |
| 5 | **Marketing (CMO)** | Market intelligence, content & campaigns | 1 + 8 tools | Execute Workflow | WF-8 |
| 6 | **Sales (CSO)** | Lead qualification, pricing, conversion | 7 | Execute Workflow | WF-2 |
| 7 | **Customer Experience (CEA)** | Product matching, advisory & support | 7 | Execute Workflow | WF-4 |
| 8 | **Risk & Ethics (CREA)** | Legal, safety, policy, bias authority | 5 | Execute Workflow | — |
| 9 | **Technology & Infrastructure** | Health, security, data & governance ops | 2 | Execute Workflow | WF-10 |
| 10 | **Global Guardrail** | Approve / block / log every decision | 0 (rules) | **HTTP webhook** | — |
| HR | **Human Resources (CHRO)** | Talent, culture, policy, hiring | 6 | Execute Workflow | — |

---

## The Universal Decision Contract

Every workflow speaks the same language: a JSON object that travels with the request and is
enriched at each step. Core fields:

| Field | Meaning |
|-------|---------|
| `request_id` | Unique id for the request (echoed end-to-end) |
| `source_workflow` | Who produced this contract |
| `department` / `task_type` | Routing target and kind of work |
| `mode` | `simulation` (no side effects) or `execute` (real actions) |
| `risk_level` | `low` / `medium` / `high` |
| `confidence` | Model confidence, 0–1 |
| `founder_available` | Whether human approval is reachable |
| `crea_status` | Risk & Ethics verdict: pending / approved / blocked / needs_human |
| `decision_status` | proposed / approved / blocked / halted |
| `failure_count` | For the circuit breaker |
| `audit` | Logs, approvals, overrides, integrity hash |

The Global Guardrail validates this contract on the way in; departments add their analysis;
the final response carries the resulting decision.

---

## Workflow details

### WF-0 · CEO Orchestrator — *the front door*
The only public entry point (webhook `ceo-orchestrator`, plus schedule and manual triggers for
testing). The **CEO Agent** reads the incoming task, classifies it (department, task type,
urgency, risk, confidence), and assembles the Universal Decision Contract. It then:
1. calls **WF-10** for an **approval gate**,
2. routes the approved contract to the right department,
3. after the department runs, calls **WF-10** again for an **execution gate**,
4. returns the final decision.

### WF-10 · Global Guardrail Authority & Decision Ledger — *the law*
A deterministic, rules-only workflow (no LLM). It is the authority every decision passes
through — twice. It performs strict **schema validation**, **locked-field integrity** checks,
**risk normalization**, simulation/execution gating, confidence thresholds, founder-authority
and Risk & Ethics routing, a **circuit breaker** (freezes a department after repeated
failures), and writes each verdict to a **decision ledger** (Google Sheets). It can also fire
**Slack** and **Gmail** alerts for high-risk blocks and contradictions. If a contract is
invalid it returns a clean `blocked` decision rather than crashing.

### WF-1 · Strategy (CBS)
Two agents evaluate strategic options and produce a final recommendation. Strategy is the most
*consultative* department — it can call Finance, Operations, R&D, Marketing, Sales, and Risk &
Ethics to gather cross-functional input before deciding.

### WF-2 · Finance (CFO)
A six-agent finance team: the **CFO Orchestrator** owns the decision, supported by Finance
Intelligence, Cost Audit, and Risk Analysis (analysis) plus Trading Ops and Risk Ops
(execution). Other departments call it for cost/approval sign-off.

### WF-3 · Operations (COO)
Seven agents covering demand forecasting, supply-chain intelligence, replenishment, packaging,
logistics, routing, and stock monitoring. Escalates to Finance (WF-2) for cost approval.

### WF-4 · R&D (AFA)
The deepest pipeline — eight agents from ingredient discovery and formulation through lab and
stability simulation, safety, compliance, and cost optimization, ending in **AFA Final
Approval**. Consults Risk & Ethics, Operations, and Marketing.

### WF-5 · Marketing (CMO)
A different topology: one **CMO Orchestrator** agent that calls eight specialist agents as
**tools** (market intelligence, trend analysis, content creation, design, writing, SEO,
campaign ops, content strategy). Validates claims with Risk & Ethics (WF-8).

### WF-6 · Sales (CSO)
Seven agents: lead qualification, pricing strategy, the CSO orchestrator, standard and senior
sales reps, partnerships, and automated lead emails. Coordinates pricing with Finance.

### WF-7 · Customer Experience (CEA)
Seven agents handling product matching, a fallback questionnaire for low-confidence cases,
advisory, sentiment, support, QA, and journey mapping — with bias/safety enforcement built in
and escalation to R&D (WF-4) when needed.

### WF-8 · Risk & Ethics (CREA) — *the conscience*
Five agents (legal compliance, safety override, policy validation, bias detection, audit ops)
that other departments escalate to for a binding compliance/safety/bias ruling. Its verdict
(`crea_status`) flows back into the contract and is honoured by the guardrail.

### WF-9 · Technology & Infrastructure (CTO + CIO)
Split into two orchestrators: the **CTO** side (system integrity, cybersecurity, IT compliance,
access control) and the **CIO** side (data engineering, knowledge-graph validation, analytics,
backups). Their decisions are merged and a governance signal is sent to WF-10.

### WF-HR · Human Resources (CHRO)
Six agents: the CHRO orchestrator plus talent insights, culture & engagement, policy
enforcement, hiring ops, and performance review — producing a full HR decision contract.

---

## How they're wired

- **Cross-workflow calls** use n8n **Execute Workflow** nodes referencing the target by its
  workflow ID (resourceLocator). All department workflows start from an *Execute Workflow
  Trigger*, so they can be invoked as sub-workflows.
- **Governance calls** use **HTTP** to WF-10's webhook (`wf10-decision-contract`), because the
  guardrail is webhook-triggered and reused as a shared service by both WF-0 and WF-9.
- **The two gates** (approval before routing, execution before action) mean a department's
  *proposed* output must clear governance again before it is treated as final — enforcing the
  rule that *all actions pass through the guardrail before execution*.
