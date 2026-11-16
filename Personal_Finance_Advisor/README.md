# Personal Finance Advisor — Multi-Agent AI System

**Built with Google Agent Development Kit (ADK) & Agent-to-Agent (A2A) Protocol**

---

## Project Overview

Welcome! In this project, you will build a **Personal Finance Advisor** powered by specialized AI agents—**Budgeting**, **Investment**, and **Market Data**—orchestrated by a **Root Agent** using **Google's Agent Development Kit (ADK)** and the **A2A (Agent-to-Agent)** protocol.

Rather than packing every prompt and tool into one monolithic agent, this architecture demonstrates how to combine:

1. **In-Process Sub-Agents** (`budgeting_agent`, `investment_agent`) running locally inside the main application process.
2. **Remote A2A Microservice Agent** (`market_data_agent`) hosted independently over HTTP on port `8001` and discovered dynamically via its `/.well-known/agent-card.json` endpoint.
3. **Coordinator Direct Tools** (`calculate_savings_goal`) attached directly to `root_agent`.

---

## System Architecture

```text
+-------------------------------------------------------------------+
|                        User Interface                             |
|              src/cli.py (Interactive) | src/simulate.py           |
+---------------------------------+---------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
|                  Root Agent (finance_advisor_root)                |
|                       [ src/root_agent.py ]                       |
|                                                                   |
|  Direct Tool: calculate_savings_goal()                            |
+----------+----------------------+-----------------------+---------+
           |                      |                       |
   (In-Process)           (In-Process)            (HTTP / A2A Card)
           |                      |                       |
           v                      v                       v
+--------------------+  +--------------------+  +-------------------+
|  Budgeting Agent   |  |  Investment Agent  |  | Market Data Agent |
| [ src/agents.py ]  |  | [ src/agents.py ]  |  | [ Port 8001 ]     |
|                    |  |                    |  |                   |
| suggest_budget()   |  | suggest_allocation |  | get_stock_price() |
+--------------------+  +--------------------+  +-------------------+
```

> **Deep Dive:** For full component breakdowns, Mermaid diagrams, deployment topologies, and runtime diagnostics, see the [System Architecture Document](./docs/Architecture_FinancialAdvisorMultiAgent.md). For production scaling, testing, and security guidance, see the [Production & Operations FAQ](./docs/FAQ.md).

---

## Prerequisites

- **Python 3.10+**
- **Google Gemini API Key** (created in Google Cloud Console)
- Two terminal windows (one for the standalone A2A server, one for the Root Agent CLI/simulation)

---

## Repository Structure

```text
personal-finance-advisor/
├── README.md                                          # Title page, architecture & tutorial index
├── Architecture_FinancialAdvisorMultiAgent.md         # Detailed system architecture & deployment guide
└── docs/                                              # Sequential step-by-step tutorials, logs & FAQ
    ├── 01-create-api-key.md                           # Chapter 1: Google Cloud & Gemini API key
    ├── 02-virtual-environment.md                      # Chapter 2: Python virtual environment setup
    ├── 03-project-setup-and-dependencies.md           # Chapter 3: ADK/A2A packages & project layout
    ├── 04-create-first-agent.md                       # Chapter 4: Minimal tool-less ADK agent
    ├── 05-budgeting-agent.md                          # Chapter 5: Budgeting agent (50/30/20 rule)
    ├── 06-investment-agent.md                         # Chapter 6: Investment allocation agent
    ├── 07-market-data-agent.md                        # Chapter 7: Standalone market data agent
    ├── 08-market-data-a2a-server.md                   # Chapter 8: AgentCard & Uvicorn A2A server
    ├── 09-create-root-agent.md                        # Chapter 9: Root agent & RemoteA2aAgent wiring
    ├── 10-root-agent-demo-tool.md                     # Chapter 10: Adding a direct tool to Root Agent
    ├── 11-connect-cli-with-root-agent.md              # Chapter 11: Interactive CLI & sample output
    ├── 12-simulate-full-financial-planning.md         # Chapter 12: End-to-end simulation & tuning
    ├── ErrorDueToPackage_starlette.applications.Starlette.md  # Troubleshooting: Missing sse_starlette in A2A server
    ├── Output-Cli.md                                  # Sample end-to-end interactive CLI execution log
    ├── FAQ.md                                         # Production knowledge base, scaling, QA & security FAQ
    └── assets/                                        # Setup reference screenshots
        ├── image1.png
        ├── image2.png
        └── image3.png
```

---

## Reference, Architecture & Troubleshooting Guides

| Document | Description |
| :--- | :--- |
| [System Architecture & Deployment Guide](./docs/Architecture_FinancialAdvisorMultiAgent.md) | Component specifications, end-to-end workflow, multi-process deployment topology, and ADK runtime diagnostics. |
| [Production & Operations FAQ](./docs/FAQ.md) | Guidance on local setup, managing knowledge bases (DB/RAG/Cache), cloud autoscaling, pre-deployment evaluations, and security hardening. |
| [A2A Startup Troubleshooting (`sse_starlette`)](./docs/ErrorDueToPackage_starlette.applications.Starlette.md) | Root cause, resolution steps, and verification logs for `ModuleNotFoundError: No module named 'sse_starlette'` when starting the A2A server. |
| [CLI Execution & Delegation Output Log](./docs/Output-Cli.md) | Full interactive CLI transcript showing multi-agent handoffs (`budgeting_agent`, `investment_agent`, remote `market_data_agent`, and `calculate_savings_goal`). |

---

## Tutorial Index (Sequential Learning Path)

### Section 1: Environment & Project Setup
| Chapter | Title | Key Concepts & Deliverables |
| :---: | :--- | :--- |
| **01** | [Create a Google API Key](./docs/01-create-api-key.md) | Enable Gemini API in Google Cloud Console and configure API key restrictions. |
| **02** | [Create a Python Virtual Environment](./docs/02-virtual-environment.md) | Isolate dependencies with `venv` across Windows, macOS, and Linux. |
| **03** | [Project Setup & ADK Dependencies](./docs/03-project-setup-and-dependencies.md) | Install `google-adk`, `a2a-sdk`, `uvicorn`, `python-dotenv`, and initialize `src/agents.py`. |

### Section 2: In-Process ADK Agents
| Chapter | Title | Key Concepts & Deliverables |
| :---: | :--- | :--- |
| **04** | [Create Your First ADK Agent](./docs/04-create-first-agent.md) | Understand `Agent` anatomy (`name`, `model`, `description`, `instruction`). |
| **05** | [Build the Budgeting Agent](./docs/05-budgeting-agent.md) | Convert `suggest_budget()` into an ADK tool implementing the 50/30/20 rule. |
| **06** | [Build the Investment Agent](./docs/06-investment-agent.md) | Build `suggest_allocation()` using the "110 minus age" rule with risk clamping. |

### Section 3: Standalone A2A Microservice
| Chapter | Title | Key Concepts & Deliverables |
| :---: | :--- | :--- |
| **07** | [Build the Market Data Agent](./docs/07-market-data-agent.md) | Create decoupled `src/market_data_agent.py` with `get_stock_price()`. |
| **08** | [Expose Market Data Agent via A2A](./docs/08-market-data-a2a-server.md) | Define `AgentCard`, wrap with `to_a2a()`, and serve on port `8001` via `uvicorn`. |

### Section 4: Multi-Agent Coordination & Execution
| Chapter | Title | Key Concepts & Deliverables |
| :---: | :--- | :--- |
| **09** | [Create the Coordinating Root Agent](./docs/09-create-root-agent.md) | Wire local agents and `RemoteA2aAgent` under `finance_advisor_root`. |
| **10** | [Add a Direct Tool to the Root Agent](./docs/10-root-agent-demo-tool.md) | Attach `calculate_savings_goal()` directly onto `root_agent.tools`. |
| **11** | [Connect CLI with the Root Agent](./docs/11-connect-cli-with-root-agent.md) | Build interactive `src/cli.py` using `InMemoryRunner` and inspect sample output. |
| **12** | [Simulate End-to-End Financial Planning](./docs/12-simulate-full-financial-planning.md) | Run full multi-agent smoke test (`src/simulate.py`) and tune delegation instructions. |

---

## Quick Start (Running the Complete System)

1. **Terminal 1 — Start the Market Data A2A Server:**
   ```bash
   cd src
   python -m uvicorn market_data_agent:a2a_app --host 0.0.0.0 --port 8001
   ```
   *(If you encounter `No module named 'sse_starlette'`, see the [A2A Startup Troubleshooting Guide](./docs/ErrorDueToPackage_starlette.applications.Starlette.md).)*

2. **Terminal 2 — Run the Interactive CLI or Automated Simulation:**
   ```bash
   cd src
   python cli.py        # Interactive chat loop (see docs/Output-Cli.md for sample run)
   # OR
   python simulate.py   # Single-query end-to-end smoke test
   ```