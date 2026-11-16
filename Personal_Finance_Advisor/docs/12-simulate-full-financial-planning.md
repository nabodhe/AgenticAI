# Chapter 12: Simulate End-to-End Financial Planning

> **Navigation:** [← Previous: Chapter 11: Connect CLI with the Root Agent](./11-connect-cli-with-root-agent.md) | [Table of Contents](../README.md) | [Finish: Return to Index →](../README.md)

| Metadata | Details |
| :--- | :--- |
| **Section** | Section 4 — Multi-Agent Coordination & Execution |
| **Estimated Time** | 15–20 minutes |
| **Tech Stack** | `google-adk`, `google-genai` |

---

## Summary

Write a non-interactive smoke-test script (`src/simulate.py`) that sends a single composite prompt exercising all three sub-agents (`budgeting_agent`, `investment_agent`, `remote_market_data_agent`) and the root agent's direct tool (`calculate_savings_goal`).

## Learning Goals

- Craft an end-to-end integration query that triggers multi-agent delegation across local and remote A2A agents.
- Reuse the `InMemoryRunner` execution pattern for automated verification.

---

## Acceptance Criteria

- [ ] `src/simulate.py` imports `root_agent` from `root_agent.py`.
- [ ] Sends a single prompt specifying monthly income, age, risk tolerance, a stock ticker, and a savings goal.
- [ ] Prints one combined financial plan covering all four components.

---

## Step-by-Step Guide

### 1. Create `src/simulate.py`

With the Market Data A2A server still running in Terminal 1, create `src/simulate.py`:

```python
# src/simulate.py
import asyncio

from google.adk.runners import InMemoryRunner
from google.genai import types

from root_agent import root_agent


async def simulate_full_plan():
    runner = InMemoryRunner(agent=root_agent, app_name="finance_advisor")
    session = await runner.session_service.create_session(
        app_name="finance_advisor", user_id="demo_user"
    )

    query = (
        "I'm 30 years old, earn $6000 a month, have medium risk tolerance, "
        "want to know the current price of VOO, and want a plan to save $10000 "
        "for a house down payment putting away $500 a month."
    )

    async for event in runner.run_async(
        user_id="demo_user",
        session_id=session.id,
        new_message=types.Content(role="user", parts=[types.Part(text=query)]),
    ):
        if event.is_final_response():
            print("Full Financial Plan:\n", event.content.parts[0].text)


if __name__ == "__main__":
    asyncio.run(simulate_full_plan())
```

### 2. Run the Simulation

```bash
cd src
python simulate.py
```

---

## Sample Execution & Troubleshooting Note

During test execution, `RemoteA2aAgent` successfully resolved the remote A2A server on `http://localhost:8001` and retrieved the price of `VOO`:

```text
(.venv) PS D:\Projects\personal-finance-advisor\src> python .\simulate.py
Full Financial Plan:
The current price of VOO is $512.80.
```

> **Troubleshooting Multi-Turn Delegation:** Notice that in the sample run above, the model returned the `VOO` price immediately after the remote A2A transfer completed without finishing the remaining three sub-tasks (budget split, stock/bond allocation, and savings timeline).
>
> **How to fix this:** If one or more parts of a multi-domain query are skipped, strengthen `root_agent`'s `instruction` in `src/root_agent.py` to explicitly instruct the coordinator to address **every** part of the user's request before producing its final synthesis:
>
> ```python
> instruction=(
>     "You are a personal finance advisor. A user query may contain multiple requests. "
>     "Ensure you gather answers for EVERY request before giving a final summary: "
>     "1) Delegate budgeting questions to budgeting_agent, "
>     "2) Delegate asset allocation questions to investment_agent, "
>     "3) Delegate stock price lookups to market_data_agent, and "
>     "4) Call calculate_savings_goal directly for savings timelines. "
>     "Synthesize all results into a single comprehensive financial plan."
> )
> ```

---

## Beginner Tip & Notes

> **Tip:** Automated single-turn smoke tests like `simulate.py` let you validate end-to-end multi-agent orchestration in seconds after changing prompts, models, or A2A endpoints—without typing four manual prompts into `cli.py`.

- **Congratulations!** You have built and verified a complete multi-agent Personal Finance Advisor combining local ADK agents and a remote A2A microservice.

---

> **Navigation:** [← Previous: Chapter 11: Connect CLI with the Root Agent](./11-connect-cli-with-root-agent.md) | [Table of Contents](../README.md) | [Finish: Return to Index →](../README.md)
