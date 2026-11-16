# Chapter 5: Build the Budgeting Agent

> **Navigation:** [← Previous: Chapter 4: Create Your First ADK Agent](./04-create-first-agent.md) | [Table of Contents](../README.md) | [Next: Chapter 6: Build the Investment Agent →](./06-investment-agent.md)

| Metadata | Details |
| :--- | :--- |
| **Section** | Section 2 — In-Process ADK Agents |
| **Estimated Time** | 15 minutes |
| **Tech Stack** | `google-adk` |

---

## Summary

Build the **Budgeting Agent**: given a user's monthly income, it calculates and explains a monthly budget breakdown using the classic **50/30/20 rule** (50% needs, 30% wants, 20% savings).

## Learning Goals

- Convert a standard Python function into an ADK callable tool and understand why its docstring is critical.
- Scope an agent to a single, well-defined responsibility rather than building one monolithic agent.

---

## Acceptance Criteria

- [ ] `suggest_budget(monthly_income: float) -> dict` implements the 50/30/20 rule (`needs`, `wants`, `savings`).
- [ ] The function includes a clear, descriptive docstring.
- [ ] `budgeting_agent` registers `suggest_budget` in its `tools` list.
- [ ] The agent's instruction tells it to invoke the tool and explain the breakdown in plain language.

---

## Step-by-Step Guide

Add the `suggest_budget` tool function and `budgeting_agent` definition to `src/agents.py`:

```python
# src/agents.py
def suggest_budget(monthly_income: float) -> dict:
    """Suggest a monthly budget split using the 50/30/20 rule."""
    return {
        "needs": round(monthly_income * 0.50, 2),
        "wants": round(monthly_income * 0.30, 2),
        "savings": round(monthly_income * 0.20, 2),
    }


budgeting_agent = Agent(
    name="budgeting_agent",
    model="gemini-2.0-flash",
    description="Suggests a monthly budget breakdown given income.",
    instruction="Given a user's monthly income, call suggest_budget and explain the split in plain language.",
    tools=[suggest_budget],
)
```

---

## Beginner Tip & Notes

> **Tip:** ADK converts plain Python functions into callable tools automatically. The function's name, type annotations, and docstring are passed directly to the model so it knows when and how to invoke the tool. A vague docstring is the most common reason an LLM passes invalid arguments or skips calling a tool.

- **Next Step:** Chapter 6 builds the Investment Agent using the same single-responsibility pattern.

---

> **Navigation:** [← Previous: Chapter 4: Create Your First ADK Agent](./04-create-first-agent.md) | [Table of Contents](../README.md) | [Next: Chapter 6: Build the Investment Agent →](./06-investment-agent.md)
