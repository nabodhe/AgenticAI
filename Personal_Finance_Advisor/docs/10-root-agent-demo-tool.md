# Chapter 10: Add a Direct Tool to the Root Agent

> **Navigation:** [← Previous: Chapter 9: Create the Coordinating Root Agent](./09-create-root-agent.md) | [Table of Contents](../README.md) | [Next: Chapter 11: Connect CLI with the Root Agent →](./11-connect-cli-with-root-agent.md)

| Metadata | Details |
| :--- | :--- |
| **Section** | Section 4 — Multi-Agent Coordination & Execution |
| **Estimated Time** | 10–15 minutes |
| **Tech Stack** | `google-adk` |

---

## Summary

Attach a standalone calculation tool directly to `root_agent` (`calculate_savings_goal`), demonstrating that a coordinator agent can execute lightweight tasks directly in addition to delegating to sub-agents.

## Learning Goals

- Combine direct `tools` and `sub_agents` on a single root `Agent`.
- Handle edge cases cleanly (such as zero or negative monthly contributions) inside tool functions.

---

## Acceptance Criteria

- [ ] `calculate_savings_goal(target_amount: float, monthly_contribution: float) -> dict` computes whole months required to reach a savings goal.
- [ ] Returns a descriptive error dictionary if `monthly_contribution <= 0` instead of raising `ZeroDivisionError`.
- [ ] Appended to `root_agent.tools`.

---

## Step-by-Step Guide

Add `calculate_savings_goal` to `src/root_agent.py` and register it on `root_agent`:

```python
# src/root_agent.py — append below root_agent definition
def calculate_savings_goal(target_amount: float, monthly_contribution: float) -> dict:
    """Calculate how many months it takes to reach a savings goal."""
    if monthly_contribution <= 0:
        return {"error": "Monthly contribution must be greater than zero."}
    months = -(-target_amount // monthly_contribution)  # ceiling division
    return {"months_needed": int(months)}


root_agent.tools.append(calculate_savings_goal)
```

---

## Beginner Tip & Notes

> **Tip:** `-(-target_amount // monthly_contribution)` is a clean Python idiom for **ceiling division** using integer floor division (`//`). Negating before and after rounds up instead of down—ensuring `2.1` months rounds up to `3` whole months needed to hit the target.

- **Next Step:** Chapter 11 connects `root_agent` to an interactive command-line interface (`src/cli.py`).

---

> **Navigation:** [← Previous: Chapter 9: Create the Coordinating Root Agent](./09-create-root-agent.md) | [Table of Contents](../README.md) | [Next: Chapter 11: Connect CLI with the Root Agent →](./11-connect-cli-with-root-agent.md)
