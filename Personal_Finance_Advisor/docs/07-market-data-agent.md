# Chapter 7: Build the Market Data Agent

> **Navigation:** [← Previous: Chapter 6: Build the Investment Agent](./06-investment-agent.md) | [Table of Contents](../README.md) | [Next: Chapter 8: Expose Market Data Agent via A2A →](./08-market-data-a2a-server.md)

| Metadata | Details |
| :--- | :--- |
| **Section** | Section 3 — Standalone A2A Service |
| **Estimated Time** | 15 minutes |
| **Tech Stack** | `google-adk` |
| **Constraint** | Must **not** import anything from `src/agents.py` |

---

## Summary

Build the **Market Data Agent** in its own standalone module (`src/market_data_agent.py`). Unlike the budgeting and investment agents, this agent will be exposed as a separate **Agent-to-Agent (A2A)** microservice running in its own OS process.

## Learning Goals

- Identify when an agent belongs in an isolated module and process rather than co-located with in-process agents.
- Replicate the tool-and-agent pattern in a self-contained file with independent imports.

---

## Acceptance Criteria

- [ ] `src/market_data_agent.py` imports `Agent` directly from `google.adk.agents` (zero imports from `agents.py`).
- [ ] `get_stock_price(ticker: str) -> dict` looks up tickers in a mock price dictionary and returns a structured error dictionary for unknown tickers.
- [ ] `market_data_agent` registers `get_stock_price` in its `tools` list.

---

## Step-by-Step Guide

Create `src/market_data_agent.py` with its own standalone imports:

```python
# src/market_data_agent.py
from dotenv import load_dotenv
from google.adk.agents import Agent

load_dotenv()


def get_stock_price(ticker: str) -> dict:
    """Fetch the latest price for a stock ticker (mocked for this project)."""
    mock_prices = {
        "AAPL": 227.50,
        "GOOGL": 172.30,
        "MSFT": 425.10,
        "VOO": 512.80,
    }
    price = mock_prices.get(ticker.upper())
    if price is None:
        return {"ticker": ticker.upper(), "error": "Unknown ticker"}
    return {"ticker": ticker.upper(), "price_usd": price}


market_data_agent = Agent(
    name="market_data_agent",
    model="gemini-2.0-flash",
    description="Looks up current prices for stock tickers.",
    instruction="Given a stock ticker, call get_stock_price and report the price.",
    tools=[get_stock_price],
)
```

---

## Beginner Tip & Notes

> **Tip:** Using a mock price dictionary keeps this project runnable without paid financial data APIs. Because `market_data_agent.py` will execute in its own server process in Chapter 8, it must remain completely decoupled from `agents.py`.

- **Next Step:** Chapter 8 wraps `market_data_agent` in an A2A server and publishes its Agent Card.

---

> **Navigation:** [← Previous: Chapter 6: Build the Investment Agent](./06-investment-agent.md) | [Table of Contents](../README.md) | [Next: Chapter 8: Expose Market Data Agent via A2A →](./08-market-data-a2a-server.md)
