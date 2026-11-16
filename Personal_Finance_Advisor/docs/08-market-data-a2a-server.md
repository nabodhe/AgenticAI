# Chapter 8: Expose Market Data Agent via A2A

> **Navigation:** [← Previous: Chapter 7: Build the Market Data Agent](./07-market-data-agent.md) | [Table of Contents](../README.md) | [Next: Chapter 9: Create the Coordinating Root Agent →](./09-create-root-agent.md)

| Metadata | Details |
| :--- | :--- |
| **Section** | Section 3 — Standalone A2A Service |
| **Estimated Time** | 20–25 minutes |
| **Tech Stack** | `google-adk`, `a2a-sdk`, `uvicorn` |

---

## Summary

Expose `market_data_agent` as a standalone **A2A (Agent-to-Agent)** HTTP service: define an `AgentCard` describing its capabilities, wrap the agent into an ASGI application via `to_a2a`, and launch it with `uvicorn` on port `8001`.

## Learning Goals

- Understand how an `AgentCard` enables remote processes to discover an agent's identity, URL, and skills without importing its Python source code.
- Host an ADK agent as an independent network service using `to_a2a` and `uvicorn`.

---

## Acceptance Criteria

- [ ] `market_data_agent_card` defines `name`, `description`, `url`, `version`, `capabilities`, and an `AgentSkill` for stock price lookups.
- [ ] `a2a_app = to_a2a(market_data_agent, port=8001)` wraps the agent into a runnable ASGI server app.
- [ ] Running `uvicorn` starts the server listening on port `8001`.
- [ ] Navigating to `http://localhost:8001/.well-known/agent-card.json` returns the JSON agent card.

---

## Step-by-Step Guide

### 1. Add the Agent Card & A2A Server Wrapper

Append the following code to `src/market_data_agent.py`:

```python
# src/market_data_agent.py — append below market_data_agent
from google.adk.a2a.utils.agent_to_a2a import to_a2a
from a2a.types import AgentCard, AgentSkill, AgentCapabilities

market_data_agent_card = AgentCard(
    name="Market Data Agent",
    description="Looks up current prices for stock tickers.",
    url="http://localhost:8001/",
    version="1.0.0",
    capabilities=AgentCapabilities(streaming=False),
    skills=[
        AgentSkill(
            id="get_stock_price",
            name="Get Stock Price",
            description="Fetches the latest price for a given stock ticker.",
            tags=["finance", "market-data"],
        )
    ],
)

# Exposes market_data_agent as a standalone A2A server on port 8001
a2a_app = to_a2a(market_data_agent, port=8001)
```

### 2. Start the A2A Server in Terminal 1

From inside the `src/` directory, start the server:

```bash
cd src
uvicorn market_data_agent:a2a_app --host 0.0.0.0 --port 8001
```

### 3. Verify the Well-Known Agent Card Endpoint

Open [http://localhost:8001/.well-known/agent-card.json](http://localhost:8001/.well-known/agent-card.json) in your browser (or via `curl`) to verify the server is live and serving its metadata.

---

## Beginner Tip & Notes

> **Tip:** Keep this terminal running for the remainder of the tutorial! Both `cli.py` (Chapter 11) and `simulate.py` (Chapter 12) connect to `http://localhost:8001` over HTTP. This decouples the service so agents can be deployed independently—even across different machines or programming languages.

- **Milestone:** Section 3 is complete. Section 4 builds the Root Agent that coordinates both local and remote sub-agents.

---

> **Navigation:** [← Previous: Chapter 7: Build the Market Data Agent](./07-market-data-agent.md) | [Table of Contents](../README.md) | [Next: Chapter 9: Create the Coordinating Root Agent →](./09-create-root-agent.md)
