# Chapter 9: Create the Coordinating Root Agent

> **Navigation:** [← Previous: Chapter 8: Expose Market Data Agent via A2A](./08-market-data-a2a-server.md) | [Table of Contents](../README.md) | [Next: Chapter 10: Add a Direct Tool to the Root Agent →](./10-root-agent-demo-tool.md)

| Metadata | Details |
| :--- | :--- |
| **Section** | Section 4 — Multi-Agent Coordination & Execution |
| **Estimated Time** | 20 minutes |
| **Tech Stack** | `google-adk` |
| **Constraint** | Must **not** import `market_data_agent` directly from `market_data_agent.py` |

---

## Summary

Create `src/root_agent.py` to orchestrate all three specialized sub-agents—two local in-process agents (`budgeting_agent`, `investment_agent`) and one remote network agent (`remote_market_data_agent` over A2A).

## Learning Goals

- Connect to a remote A2A service using `RemoteA2aAgent` and `AGENT_CARD_WELL_KNOWN_PATH`.
- Observe how a root ADK agent treats local and remote sub-agents uniformly during delegation.

---

## Acceptance Criteria

- [ ] `src/root_agent.py` imports `budgeting_agent` and `investment_agent` from `agents.py`.
- [ ] `remote_market_data_agent` is configured as a `RemoteA2aAgent` pointing to `http://localhost:8001` + `AGENT_CARD_WELL_KNOWN_PATH`.
- [ ] `root_agent` registers all three agents in `sub_agents` and defines clear delegation instructions.

---

## Step-by-Step Guide

Create `src/root_agent.py`:

```python
# src/root_agent.py
from dotenv import load_dotenv
from google.adk.agents import Agent
from google.adk.agents.remote_a2a_agent import RemoteA2aAgent, AGENT_CARD_WELL_KNOWN_PATH

from agents import budgeting_agent, investment_agent

load_dotenv()

remote_market_data_agent = RemoteA2aAgent(
    name="market_data_agent",
    description="Looks up current prices for stock tickers.",
    agent_card=f"http://localhost:8001{AGENT_CARD_WELL_KNOWN_PATH}",
)

root_agent = Agent(
    name="finance_advisor_root",
    model="gemini-2.0-flash",
    description="Coordinates budgeting, investment, and market-data agents into one financial plan.",
    instruction=(
        "You are a personal finance advisor. Delegate budgeting questions to budgeting_agent, "
        "allocation questions to investment_agent, and stock-price questions to market_data_agent. "
        "Combine their answers into one clear financial plan."
    ),
    sub_agents=[budgeting_agent, investment_agent, remote_market_data_agent],
)
```

---

## Beginner Tip & Notes

> **Tip:** Make sure the Chapter 8 A2A server is running before initializing `root_agent`, because `RemoteA2aAgent` fetches and resolves the remote agent card at startup. From `root_agent`'s perspective, delegating to `remote_market_data_agent` works identically to delegating to local agents.

- **Next Step:** Chapter 10 equips `root_agent` with a direct tool of its own alongside its sub-agents.

---

> **Navigation:** [← Previous: Chapter 8: Expose Market Data Agent via A2A](./08-market-data-a2a-server.md) | [Table of Contents](../README.md) | [Next: Chapter 10: Add a Direct Tool to the Root Agent →](./10-root-agent-demo-tool.md)
