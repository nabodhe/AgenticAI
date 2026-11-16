# Chapter 6: Build the Investment Agent

> **Navigation:** [← Previous: Chapter 5: Build the Budgeting Agent](./05-budgeting-agent.md) | [Table of Contents](../README.md) | [Next: Chapter 7: Build the Market Data Agent →](./07-market-data-agent.md)

| Metadata | Details |
| :--- | :--- |
| **Section** | Section 2 — In-Process ADK Agents |
| **Estimated Time** | 15 minutes |
| **Tech Stack** | `google-adk` |

---

## Summary

Build the **Investment Agent**: given a user's age and risk tolerance, it recommends a stock-to-bond portfolio allocation.

## Learning Goals

- Reinforce the single-function, single-agent design pattern with multi-parameter input handling.
- Clamp computed financial percentages safely between `0` and `100`.

---

## Acceptance Criteria

- [ ] `suggest_allocation(age: int, risk_tolerance: str) -> dict` applies the `"110 minus age"` rule of thumb for base stock allocation.
- [ ] Allocation adjusts for `risk_tolerance` (`"low"`, `"medium"`, or `"high"`) and clamps the final percentage to `[0, 100]`.
- [ ] `investment_agent` registers `suggest_allocation` in its `tools` list.

---

## Step-by-Step Guide

Add `suggest_allocation` and `investment_agent` to `src/agents.py`:

```python
# src/agents.py
def suggest_allocation(age: int, risk_tolerance: str) -> dict:
    """Suggest a simple stock/bond allocation based on age and risk tolerance."""
    base_stock_pct = max(0, min(100, 110 - age))
    adjustment = {"low": -15, "medium": 0, "high": 15}.get(risk_tolerance.lower(), 0)
    stock_pct = max(0, min(100, base_stock_pct + adjustment))
    return {"stocks_pct": stock_pct, "bonds_pct": 100 - stock_pct}


investment_agent = Agent(
    name="investment_agent",
    model="gemini-2.0-flash",
    description="Suggests a simple stock/bond allocation based on age and risk tolerance.",
    instruction="Given a user's age and risk tolerance, call suggest_allocation and explain the split.",
    tools=[suggest_allocation],
)
```

---

## Beginner Tip & Notes

> **Tip:** `max(0, min(100, ...))` is the standard Python idiom for clamping a value within `[0, 100]`. It prevents extreme ages or aggressive risk adjustments from producing negative percentages or allocations exceeding 100%.

- **Milestone:** Section 2 is complete—you now have two working in-process agents (`budgeting_agent` and `investment_agent`). Section 3 builds a third agent designed to run as an independent network service.

---

> **Navigation:** [← Previous: Chapter 5: Build the Budgeting Agent](./05-budgeting-agent.md) | [Table of Contents](../README.md) | [Next: Chapter 7: Build the Market Data Agent →](./07-market-data-agent.md)
