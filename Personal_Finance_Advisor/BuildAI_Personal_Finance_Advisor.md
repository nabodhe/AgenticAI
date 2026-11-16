# BuildAI: Personal Finance Advisor

Welcome! In this project, you'll build a personal finance advisor made of several specialized AI agents — **budgeting**, **investing**, and **market data** — coordinated by a root agent using Google's **Agent Development Kit (ADK)** and the **A2A (Agent-to-Agent)** protocol.

## Build Order

* **Section 1:** Sets up the project environment and dependencies.
* **Section 2:** Builds two simple in-process agents (`budgeting` and `investment`) to learn the shape of an ADK agent.
* **Section 3:** Builds a third agent — `market_data` — and exposes it as its own standalone A2A service rather than running in the same process, complete with an agent card describing its capabilities.
* **Section 4:** Wires everything together under a `root` agent that can reach all three sub-agents (two locally, one remotely) and drives the system from a CLI.

## Prerequisites

* Python 3.10+
* A Gemini API key

## Project Structure

This is a multi-file, multi-process project rather than a single notebook: `src/market_data_agent.py` runs as its own server in one terminal, while `src/cli.py` and `src/simulate.py` run in another and communicate with it over the network.

```text
.
├── .env
└── src/
    ├── agents.py
    ├── cli.py
    ├── market_data_agent.py
    ├── root_agent.py
    └── simulate.py
```