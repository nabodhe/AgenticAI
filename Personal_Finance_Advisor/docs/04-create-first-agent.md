# Chapter 4: Create Your First ADK Agent

> **Navigation:** [← Previous: Chapter 3: Project Setup & ADK Dependencies](./03-project-setup-and-dependencies.md) | [Table of Contents](../README.md) | [Next: Chapter 5: Build the Budgeting Agent →](./05-budgeting-agent.md)

| Metadata | Details |
| :--- | :--- |
| **Section** | Section 2 — In-Process ADK Agents |
| **Estimated Time** | 10 minutes |
| **Tech Stack** | `google-adk` |

---

## Summary

Create a minimal tool-less agent in `src/agents.py` to understand the structure of the ADK `Agent` class before building specialized domain agents.

## Learning Goals

- Learn the minimum required fields for an ADK `Agent`.
- Understand that a tool-less agent is still a functional agent that answers directly from the underlying LLM's knowledge.

---

## Acceptance Criteria

- [ ] `first_agent` defines `name`, `model` (`"gemini-2.0-flash"`), `description`, and `instruction`.
- [ ] `first_agent` has no `tools` configured.
- [ ] The instruction directs the agent to answer basic personal-finance questions in short, plain language.

---

## Step-by-Step Guide

Add `first_agent` to `src/agents.py`:

```python
# src/agents.py
first_agent = Agent(
    name="finance_assistant_demo",
    model="gemini-2.0-flash",
    description="A friendly assistant that answers basic personal-finance questions.",
    instruction="You are a helpful personal finance assistant. Keep answers short and practical.",
)
```

---

## Beginner Tip & Notes

> **Tip:** Every ADK agent requires at minimum a `name` (used internally and by parent agents for routing), a `model`, and an `instruction` (its system prompt). Tools are added in Chapters 5 and 6.

- **Note:** `first_agent` is strictly for learning the `Agent` class signature and is not wired into the final multi-agent graph. Chapter 5 builds the first production domain agent.

---

> **Navigation:** [← Previous: Chapter 3: Project Setup & ADK Dependencies](./03-project-setup-and-dependencies.md) | [Table of Contents](../README.md) | [Next: Chapter 5: Build the Budgeting Agent →](./05-budgeting-agent.md)
