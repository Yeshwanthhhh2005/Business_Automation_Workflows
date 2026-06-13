# What This Project Can Actually Do

A practical, honest description of what the system does today — not a wish-list.

---

## The core capability

**Take any business request and run it through a complete, governed decision pipeline —
end to end, automatically.**

Send one HTTP request to the CEO. The system will:

1. **Understand and classify it** — the CEO agent decides which department owns it, how risky it
   is, how urgent, and how confident it is.
2. **Govern it** — the Global Guardrail validates the request, scores its risk, and either
   approves or blocks it *before any work starts*.
3. **Execute it** — the owning department's team of agents reasons through the task and produces
   a structured decision.
4. **Re-govern it** — the proposed action is sent back through the guardrail before it is
   treated as final.
5. **Log and return it** — the verdict is recorded in a decision ledger and returned to the
   caller as structured JSON.

All of this happens in one request, with no human in the loop unless the governance rules
demand one.

## What each department can do

| Department | What you can ask it for |
|------------|-------------------------|
| **Strategy** | Evaluate options, weigh trade-offs, synthesize a cross-functional recommendation |
| **Finance** | Budget review, cost control, financial risk analysis, transaction sign-off |
| **Operations** | Demand forecasting, inventory replenishment, packaging, logistics & routing |
| **R&D** | Research inputs, propose & simulate formulations, run safety & compliance checks |
| **Marketing** | Market & trend intelligence, content/creative generation, SEO & campaign planning |
| **Sales** | Lead qualification, pricing strategy, deal handling, outbound lead outreach |
| **Customer Experience** | Product matching, advisory, sentiment analysis, support & QA |
| **Risk & Ethics** | Binding legal / safety / policy / bias rulings on any proposed action |
| **Technology & Infra** | System health, security scans, IT compliance, data ops, backups |
| **Human Resources** | Talent analytics, culture, policy enforcement, hiring, performance reviews |

## Two operating modes

- **`simulation`** — agents reason and produce a full decision, but **no real side effects fire**
  (no emails sent, no ledgers written, no external actions). Perfect for testing the
  organization's "thinking" safely.
- **`execute`** — the same flow, but real actions are permitted *after* passing the stricter
  execution gates (confidence threshold, founder authority, Risk & Ethics clearance).

The mode travels in the decision contract, so the *same* governance logic enforces it everywhere.

## Governance & safety features

This is the part that makes it a system rather than a demo:

- **Two-stage gating** — every decision is cleared once before routing and again before execution.
- **Strict decision contract** — typed JSON validated at the boundary; malformed requests are
  cleanly blocked, never crash the system.
- **Risk scoring & thresholds** — high-risk or low-confidence actions are blocked or escalated.
- **Risk & Ethics veto** — a dedicated department can block actions on legal, safety, policy, or
  **bias** grounds, and its verdict is binding.
- **Founder-approval path** — actions that need a human can be halted pending approval.
- **Circuit breaker** — a department that fails repeatedly can be automatically frozen.
- **Decision ledger** — every verdict (who/what/why/when) is appended to a Google Sheet.
- **Alerting** — high-risk blocks and contradictions can notify a Slack channel and email the
  founder.

## Integrations available

| Integration | Used for |
|-------------|----------|
| **HTTP / Webhooks** | Entry point and the guardrail service |
| **Google Sheets** | The decision ledger |
| **Slack** | High-risk / block alerts |
| **Gmail** | Founder notifications, lead emails |
| **OpenAI** | All 60 agents' reasoning, with structured-output schemas |

(The external integrations are credential-gated — drop in your own Slack/Sheets/Gmail
credentials to turn them on; the core reasoning runs without them.)

## Example: one request, fully governed

**Request**
```json
POST /webhook/ceo-orchestrator
{
  "request_id": "demo_42",
  "mode": "simulation",
  "risk_level": "low",
  "payload": { "task": "Plan inventory replenishment for next quarter" }
}
```

**What happens**
```
CEO Agent        → classifies: department=operations, task_type=operational, confidence=0.85
Global Guardrail → schema valid, low risk, simulation → approved (proposed)
Operations dept  → 7 agents: forecast demand → supply-chain → replenishment → packaging →
                   logistics → routing → stock monitor
Guardrail (exec) → re-checked, cleared
Ledger           → decision recorded
```

**Response**
```json
{
  "request_id": "demo_42",
  "department": "operations",
  "decision_status": "proposed",
  "execution_allowed": false,
  "reason": "Simulation mode - execution blocked. No side effects permitted.",
  "timestamp": "..."
}
```

Flip `mode` to `execute` and the same request runs the stricter gates and permits real actions.

## What's demonstrated vs. what extends it

**Working today:** the full multi-agent organization — classification, governance, routing,
department execution with real agent reasoning, cross-department consultation, the two gates,
and the decision ledger — all end to end.

**Natural extensions** (the agents currently reason from the model alone): wire the L3/L4 agents
to live data sources — CRM, inventory database, market-price APIs, a vector DB / knowledge
graph, analytics, lead scoring — so their decisions act on real signals rather than reasoning
in the abstract. The architecture is built for this: each agent is an isolated node you can give
tools and data without touching the rest of the system.

## Best results

Use a **paid OpenAI key** and a capable model. The agent-heavy departments (Marketing, CX, R&D)
make many model calls per request, and reliable structured-output parsing depends on model
quality and rate limits — free-tier credits will throttle and occasionally return unparsable
output. Agents auto-retry, but the model is the floor on reliability.
