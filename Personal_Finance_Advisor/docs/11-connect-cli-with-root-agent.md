# Chapter 11: Connect CLI with the Root Agent

> **Navigation:** [← Previous: Chapter 10: Add a Direct Tool to the Root Agent](./10-root-agent-demo-tool.md) | [Table of Contents](../README.md) | [Next: Chapter 12: Simulate End-to-End Financial Planning →](./12-simulate-full-financial-planning.md)

| Metadata | Details |
| :--- | :--- |
| **Section** | Section 4 — Multi-Agent Coordination & Execution |
| **Estimated Time** | 20–25 minutes |
| **Tech Stack** | `google-adk`, `google-genai` |

---

## Summary

Wire `root_agent` into an interactive command-line chat loop (`src/cli.py`) so you can converse with the multi-agent system in real time.

## Learning Goals

- Initialize an `InMemoryRunner` and user session, and drive turns from a `while` input loop.
- Filter the asynchronous event stream using `event.is_final_response()` to separate user-facing answers from intermediate delegation and tool-call events.

---

## Acceptance Criteria

- [ ] `src/cli.py` imports `root_agent` from `root_agent.py`.
- [ ] Creates an `InMemoryRunner` and session for `root_agent`.
- [ ] Reads user prompts in a loop, wraps them in `types.Content`, and prints the final agent response.
- [ ] Cleanly exits when the user enters `"exit"`.

---

## Step-by-Step Guide

### 1. Create `src/cli.py`

Ensure Terminal 1 is still running the Market Data A2A server (`uvicorn market_data_agent:a2a_app --port 8001`), then create `src/cli.py`:

```python
# src/cli.py
import asyncio

from google.adk.runners import InMemoryRunner
from google.genai import types

from root_agent import root_agent


async def run_cli():
    runner = InMemoryRunner(agent=root_agent, app_name="finance_advisor")
    session = await runner.session_service.create_session(
        app_name="finance_advisor", user_id="cli_user"
    )

    print("Personal Finance Advisor — type 'exit' to quit.")
    while True:
        user_input = input("You: ")
        if user_input.strip().lower() == "exit":
            break

        async for event in runner.run_async(
            user_id="cli_user",
            session_id=session.id,
            new_message=types.Content(role="user", parts=[types.Part(text=user_input)]),
        ):
            if event.is_final_response():
                print("Advisor:", event.content.parts[0].text)


if __name__ == "__main__":
    asyncio.run(run_cli())
```

### 2. Run the CLI in Terminal 2

```bash
cd src
python cli.py
```

---

## Sample Execution Output

Below is a verified console session demonstrating `root_agent` delegating a budgeting prompt to `budgeting_agent` and invoking `suggest_budget(monthly_income=5000)`:

```text
(.venv) PS D:\Projects\personal-finance-advisor\src> python .\cli.py
Welcome to the Personal Finance Advisor CLI!
Type 'exit' to quit.

You: My income is $5000, help me budgeting?

[Event Flow]
1. finance_advisor_root -> transfer_to_agent(agent_name='budgeting_agent')
2. budgeting_agent      -> suggest_budget(monthly_income=5000)
3. tool_response        -> {'needs': 2500.0, 'wants': 1500.0, 'savings': 1000.0}

Advisor: Based on your monthly income of $5,000, here is a recommended monthly budget using the popular **50/30/20 rule**:

* **Needs (50% — $2,500):** This covers your absolute essentials to live and work, such as rent/mortgage, utilities, groceries, basic transportation, and minimum debt payments.
* **Wants (30% — $1,500):** This is for your lifestyle choices and discretionary spending, such as dining out, entertainment, subscriptions, hobbies, and shopping.
* **Savings & Debt Repayment (20% — $1,000):** This goes toward building your financial future, including emergency savings, retirement accounts, investments, and extra debt payments above the minimums.

This split helps you maintain a balanced lifestyle while steadily building savings and covering your necessities!

You: exit
Goodbye!
```

---

## Beginner Tip & Notes

> **Tip:** `runner.run_async(...)` yields a stream of internal events (agent transfers, tool calls, tool responses, and final messages). Checking `event.is_final_response()` ensures only the synthesized answer is printed to the user.

- **Next Step:** Chapter 12 builds an automated single-query script (`src/simulate.py`) to test all agents and tools simultaneously.

---

> **Navigation:** [← Previous: Chapter 10: Add a Direct Tool to the Root Agent](./10-root-agent-demo-tool.md) | [Table of Contents](../README.md) | [Next: Chapter 12: Simulate End-to-End Financial Planning →](./12-simulate-full-financial-planning.md)
