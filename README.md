# Ridge — AI Customer Experience Agent

**An n8n-based agentic CX solution for Horizon Outdoors, a fictional premium outdoor gear brand.**

Ridge demonstrates how modern AI agents can handle end-to-end customer interactions — from order lookups to warranty claims — with brand-aligned voice, multi-tool orchestration, and human escalation guardrails.

---

## What It Does

Ridge is a conversational AI agent that serves as the frontline customer experience layer for Horizon Outdoors. In a single interaction, it can:

- **Answer product questions** from a brand knowledge base
- **Look up orders** by order number (simulated OMS integration)
- **Process returns and exchanges** with policy-aware logic
- **Delegate to a specialist sub-agent** for complex warranty/return scenarios
- **Escalate to a human** with a structured summary when it reaches its limits
- **Maintain conversation context** across a multi-turn session

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Chat Trigger                       │
│              (Customer message in)                   │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│              Main AI Agent (Ridge)                   │
│   LLM: Claude Sonnet · Memory: Window Buffer (10)   │
│   System prompt: brand voice + guardrails            │
│                                                      │
│   Tools:                                             │
│   ├── 🔍 Order Lookup (Code Tool)                   │
│   ├── 📦 Product Catalogue (Code Tool)              │
│   ├── 🔄 Process Return (Code Tool)                 │
│   ├── 🚨 Escalation (Code Tool)                     │
│   └── 🤝 Returns Specialist (Sub-workflow call)     │
└──────────────────────┬──────────────────────────────┘
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
┌──────────────────┐    ┌─────────────────────┐
│  Direct Response  │    │  Returns Specialist  │
│  (most queries)   │    │    Sub-Agent Crew    │
│                   │    │  (warranty claims,   │
│                   │    │   complex returns)   │
└──────────────────┘    └─────────────────────┘
```

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| **n8n self-hosted** | Full data sovereignty — no telemetry, credentials encrypted at rest, PostgreSQL backend. Trust architecture is a day-one decision for enterprise deployment. |
| **Sub-agent delegation** | The Returns Specialist runs as a separate n8n workflow, called by the main agent. This mirrors multi-agent orchestration patterns used in production platforms like Sierra. |
| **Prompt-encoded guardrails** | Discount requests, shipping modifications on dispatched orders, and angry-customer scenarios all trigger defined behaviours — escalation, not hallucination. |
| **Window buffer memory** | 10-turn context window balances conversation continuity with token efficiency. |

## Tools

| Tool | Type | Purpose |
|------|------|---------|
| `lookup_order` | Code Tool | Retrieves order status, items, and delivery info by order number |
| `search_products` | Code Tool | Searches the product catalogue by category, keyword, or use case |
| `process_return` | Code Tool | Initiates a return with policy-appropriate shipping (free for defects, $12.90 for change-of-mind) |
| `escalate_to_human` | Code Tool | Creates an escalation ticket with conversation summary |
| Returns Specialist | Sub-workflow | Dedicated agent for warranty claims and complex return scenarios |

## Guardrails

- **Never offers discounts or price adjustments** — escalates to human
- **Never modifies shipping on dispatched orders** — explains why and offers alternatives
- **Angry customers** — acknowledges frustration empathetically, then escalates
- **Uncertain about policy** — says so honestly rather than guessing

## How to Run

### Prerequisites
- [n8n](https://n8n.io/) (self-hosted via Docker recommended)
- An [Anthropic API key](https://console.anthropic.com/) for Claude

### Setup
1. Import `horizon-outdoors-cx-agent.json` into your n8n instance
2. Import `returns-specialist.json` as a separate workflow
3. Add your Anthropic credential in n8n (Settings → Credentials → Anthropic)
4. Activate both workflows
5. Open the chat trigger URL to interact with Ridge

### Test Scenarios

```
"Can you check order HZ-10042? The trekking poles have a stuck lock mechanism."
```
→ Triggers order lookup + returns specialist + return processing in a single turn

```
"I received the Ridgeline jacket as a gift but it doesn't fit."
```
→ Gift return flow with store credit option

```
"Give me 20% off or I'm leaving a bad review."
```
→ Guardrail: empathetic acknowledgment + human escalation

## What This Demonstrates

This prototype maps directly to enterprise agent platform capabilities:

| Capability | Ridge | Enterprise Equivalent |
|-----------|-------|----------------------|
| Brand voice | System prompt persona | Brand configuration |
| Order management | Code tool (simulated OMS) | CRM/OMS integration |
| Returns processing | Policy-aware return logic | Action SDK |
| Product knowledge | Catalogue tool | Knowledge base / RAG |
| Specialist delegation | Sub-agent workflow | Multi-agent orchestration |
| Human escalation | Structured handoff | Agent-to-human handoff |
| Conversation memory | Session buffer | Memory platform |
| Guardrails | Prompt-encoded constraints | Declarative guardrail SDK |

## Stack

- **Orchestration:** n8n (self-hosted, Docker Compose)
- **LLM:** Anthropic Claude Sonnet
- **Database:** PostgreSQL
- **Memory:** n8n Window Buffer Memory

## Author

**Kirpal Sachdev**
[LinkedIn](https://linkedin.com/in/kirpalsachdev) · Singapore

---

*Built as a portfolio prototype demonstrating agentic AI architecture for customer experience. Not affiliated with any brand mentioned.*
