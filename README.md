# Autonomous AI Company — A Multi-Agent Business Operating System

> A network of **60 AI agents**, organized into **12 coordinated workflows** that mirror the
> departments of a real company — a CEO that routes work, specialist departments that do it,
> and a global governance authority that approves, blocks, and logs **every** decision.
> Built end-to-end in [n8n](https://n8n.io), powered by OpenAI, runs anywhere via Docker.

---

## What is this project?

Most "AI agent" demos are a single chatbot with some tools. This is something different: an
attempt to model an **entire organization** as cooperating AI agents, with the same checks and
balances a real company has.

A request comes in (a task, a decision to make, a job to run). It is:

1. **Classified and routed** by a CEO orchestrator agent.
2. **Cleared by a governance authority** before anything happens — schema-validated, risk-scored,
   and gated.
3. **Executed by the relevant department**, where a team of specialist agents (a department
   head + analysts + operational agents) reason through it and produce a decision.
4. **Re-cleared by governance** before any real-world action is taken.
5. **Logged to a decision ledger** and returned to the caller.

Nothing in the system acts without passing through the governance layer first. That single
idea — *every decision is governed, contracted, and auditable* — is what makes this more than
a pile of prompts.

## Architecture at a glance

```
            incoming request (webhook / schedule / manual)
                              │
                              ▼
                 ┌───────────────────────────┐
                 │  WF-0 · CEO Orchestrator   │   classifies intent, builds the
                 │  (the "front door")        │   Universal Decision Contract
                 └─────────────┬──────────────┘
                               │  ① approval gate
                               ▼
                 ┌───────────────────────────┐
                 │ WF-10 · Global Guardrail   │   validate → risk-score → approve/block
                 │ Authority & Decision Ledger│   → log.  Re-checked before execution.
                 └─────────────┬──────────────┘
                approved       │
        ┌──────────┬──────────┬┴─────────┬──────────┬──────────┬──────────┐
        ▼          ▼          ▼          ▼          ▼          ▼          ▼
      WF-1       WF-2       WF-3       WF-4       WF-5       WF-6 …      WF-HR
    Strategy   Finance   Operations    R&D      Marketing   Sales        HR
        │          ▲          │          │          │          │
        └─ consult ┴──────────┴── cross-department escalation ─┴── WF-8 Risk & Ethics
                               │
                               ▼  ② execution gate (WF-10 again)
                          final response
```

- **WF-0 — CEO Orchestrator:** the single entry point. Classifies the request and routes it.
- **WF-10 — Global Guardrail:** the governance authority every decision passes through (twice).
- **WF-1…WF-9 + WF-HR — Departments:** Strategy, Finance, Operations, R&D, Marketing, Sales,
  Customer Experience, Risk & Ethics, Technology/Infrastructure, and Human Resources.

## The big ideas

| Principle | How it shows up |
|-----------|-----------------|
| **Governed by default** | A guardrail workflow gates routing *and* execution; nothing runs unchecked. |
| **One contract everywhere** | A single *Universal Decision Contract* (request_id, department, risk, confidence, mode, audit…) threads through all 12 workflows. |
| **Departments are agent teams** | Each department is a hierarchy: orchestrator → specialist analysts → operational agents. |
| **Simulation vs. execution** | Run in `simulation` to reason without side effects, or `execute` for real actions — the same governance applies. |
| **Auditable** | Every decision is written to a decision ledger with who/what/why. |
| **Modular** | Departments are independent workflows wired by ID; add or swap one without touching the rest. |

## Tech stack

- **Orchestration:** n8n (12 workflow exports, importable as-is)
- **Intelligence:** OpenAI chat models with **structured-output contracts** (typed JSON, not free text)
- **Agents:** 52 reasoning agents + 8 specialist agent-tools = **60 AI agents**
- **Integrations:** HTTP webhooks, Slack alerts, Gmail notifications, Google Sheets decision ledger
- **Runtime:** self-hosted via Docker (zero infra cost; you pay only for model tokens)

## Documentation

| Doc | What's inside |
|-----|---------------|
| **[AGENTS.md](AGENTS.md)** | Every agent, what it does, and how they're wired together |
| **[WORKFLOWS.md](WORKFLOWS.md)** | Each of the 12 workflows explained in detail |
| **[CAPABILITIES.md](CAPABILITIES.md)** | What the system can actually do, end to end |

## Run it locally (Docker)

```powershell
# 1. Start n8n with this folder mounted as the workflow source
docker run -d --name n8n --restart unless-stopped -p 5678:5678 `
  -e NODE_FUNCTION_ALLOW_BUILTIN=crypto -e NODE_FUNCTION_ALLOW_EXTERNAL=uuid `
  -v n8n_data:/home/node/.n8n `
  -v "${PWD}:/import" `
  docker.n8n.io/n8nio/n8n

# 2. Add your OpenAI API key as an "OpenAI" credential in the UI (http://localhost:5678)

# 3. Import the 12 workflows (IDs are preserved so cross-workflow calls resolve)
docker exec n8n n8n import:workflow --separate --input=/import
# (import deactivates workflows — re-activate each with `n8n publish:workflow --id=<id>`, then restart)
```

Then fire a request at the CEO:

```powershell
curl -Method POST http://localhost:5678/webhook/ceo-orchestrator `
  -ContentType "application/json" `
  -Body '{"request_id":"demo_1","mode":"simulation","risk_level":"low","payload":{"task":"Plan inventory replenishment for next quarter"}}'
```

Watch it run under **Executions** at http://localhost:5678.

## Status

The full orchestration runs end-to-end: classify → govern → route → execute (department agents) →
re-govern → respond, with a clean, governed decision returned for every department. Use a paid
OpenAI key for reliable structured-output parsing across the agent-heavy departments.
