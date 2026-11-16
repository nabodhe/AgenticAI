# Production & Operations FAQ

> **Navigation:** [← Back to Index](../README.md) | [System Architecture](../Architecture_FinancialAdvisorMultiAgent.md)

This FAQ covers production readiness topics for the **Personal Finance Advisor** multi-agent system, including knowledge base management, horizontal scaling, pre-deployment testing, and security hardening.

---

## 1. How do I manage the knowledge base in this project?

In the starter implementation, knowledge is divided between static LLM weights and deterministic Python dictionaries (`mock_prices` in `src/market_data_agent.py` and fixed rules in `src/agents.py`). For production workloads, manage your knowledge base across three distinct tiers:

* **Structured Financial & User Data (Transactional Database):** Store user profiles (age, risk tolerance, income, savings goals) and portfolio holdings in a relational database such as **PostgreSQL** or **Cloud SQL**. Inject user profile context into ADK session state (`session.state`) rather than requiring users to re-enter baseline metrics on every turn.
* **Unstructured Financial Guidelines & Policy Documents (RAG / Vector Store):** For tax rules, retirement contribution limits (401(k), IRA), investment prospectuses, and compliance policies, build a Retrieval-Augmented Generation (RAG) pipeline using **Vertex AI Search**, **pgvector**, or **Qdrant**. Expose retrieval as a dedicated ADK tool (e.g., `search_financial_policies(query: str)`) on `investment_agent` or a dedicated compliance agent.
* **Live Market & Economic Data (External APIs + Cache):** Replace the `mock_prices` dictionary inside `get_stock_price()` with an adapter calling a live market data provider (such as Polygon.io, Alpha Vantage, or Yahoo Finance) backed by a short-TTL **Redis** cache (15–60 seconds) to minimize latency and avoid provider rate limits.

---

## 2. How do I scale this application in production?

Because `market_data_agent` runs as an isolated **A2A (Agent-to-Agent)** HTTP service while `budgeting_agent` and `investment_agent` run in-process under `root_agent`, you can scale the orchestration tier and remote specialist services independently:

| Layer | Prototype Setup | Production Scaling Strategy |
| :--- | :--- | :--- |
| **Root Orchestrator (`root_agent`)** | Single-process CLI / script (`InMemoryRunner`) | Wrap `root_agent` in a stateless FastAPI/ASGI service and deploy on **Google Cloud Run** or **GKE** with horizontal autoscaling based on concurrent requests and latency. |
| **Session & State Store** | `InMemoryRunner` (ephemeral, single-process only) | Replace with a persistent session service (`DatabaseSessionService` backed by Cloud SQL/PostgreSQL or Redis) so any container replica can serve any conversation turn. |
| **Remote A2A Services (`market_data_agent`)** | Single `uvicorn` process on `localhost:8001` | Deploy as an independent Cloud Run service or Kubernetes Deployment behind an internal load balancer, scaling separately during market-open traffic spikes. |
| **Token & Latency Optimization** | Uncached prompt swaps on every sub-agent transfer | Enable ADK's **`context_cache_config`** on the application so shared system instructions and tool schemas remain cached across agent handoffs, reducing Time-To-First-Token (TTFT) and API cost. |
| **Parallel Sub-Task Execution** | Sequential multi-turn tool/agent calls | Use ADK's `ParallelAgent` workflow pattern or explicit multi-tool routing when sub-tasks (budgeting, allocation, market price) have no data dependencies on each other. |

---

## 3. What evaluations or tests need to be performed before deploying to production cloud?

Before promoting the system to a production cloud environment, run a four-stage evaluation and testing pipeline:

### A. Deterministic Unit & Contract Tests (`pytest`)
* **Tool Logic & Boundary Testing:** Verify edge cases on pure Python tools—for example, confirming `suggest_allocation(age=120, risk_tolerance="high")` clamps cleanly within `[0, 100]`, `calculate_savings_goal(10000, 0)` returns a descriptive error dictionary instead of raising `ZeroDivisionError`, and `get_stock_price("INVALID")` handles unknown tickers gracefully.
* **A2A Agent Card Schema Validation:** Assert that `GET /.well-known/agent-card.json` conforms to the `a2a-sdk` specification and publishes the expected `AgentSkill` IDs and capabilities.

### B. Trajectory & Delegation Evaluations (ADK Eval / Golden Dataset)
* **Routing Accuracy:** Execute a golden dataset of 50+ single-domain, multi-domain, and out-of-scope prompts to verify `root_agent` delegates to the exact expected sub-agents (`budgeting_agent`, `investment_agent`, `market_data_agent`) and invokes `calculate_savings_goal` without skipping any part of a compound query.
* **Argument Extraction Fidelity:** Confirm the LLM extracts exact numeric parameters (e.g., `$6,000` income, `30` years old, `"VOO"`) without hallucinating values.

### C. Integration & Resilience Testing
* **A2A Network Fault Injection:** Simulate timeouts, HTTP `500` errors, and high latency from `market_data_agent` to ensure `root_agent` degrades gracefully (returning the budget and investment sections while noting market data is temporarily unavailable) rather than failing the entire request.

### D. Load & Financial Guardrail Audits
* **Concurrency Load Testing:** Run **Locust** or **k6** against the FastAPI/Cloud Run endpoint to measure p95 latency, Gemini API quota limits (`429 Resource Exhausted`), and connection pooling under load.
* **Hallucination & Compliance Audits:** Use an LLM-as-a-Judge evaluator to confirm final responses accurately reflect tool outputs (`suggest_budget`, `get_stock_price`) and include non-fiduciary financial disclaimers.

---

## 4. How do I secure this application from bad users or attackers?

Securing a multi-agent AI application requires defense-in-depth across the public API perimeter, inter-agent A2A network traffic, and the LLM reasoning layer:

### 1. Perimeter & Identity Security
* **Authentication & Authorization:** Never expose the ADK API endpoint unauthenticated. Require **OAuth 2.0 / OIDC JWTs** (via Google Identity Platform, Firebase Auth, or Auth0) at an API Gateway, and bind `user_id` and `session_id` strictly to the authenticated token subject (`sub`) to prevent Insecure Direct Object Reference (IDOR) session hijacking.
* **Rate Limiting & Quotas:** Enforce per-user and per-IP request/token rate limits at the API Gateway (e.g., Cloud Armor + Apigee/Envoy) to mitigate denial-of-wallet attacks where malicious users trigger expensive multi-agent loops.

### 2. Securing Agent-to-Agent (A2A) Communication
* **Zero-Trust Service-to-Service Auth:** Do not expose `market_data_agent` on a public interface (`0.0.0.0:8001`). Place it on a private VPC or internal-only ingress and require **mTLS** or **IAM OIDC service-account bearer tokens** on every A2A HTTP request from `root_agent`.
* **Agent Card Allowlisting:** Restrict `RemoteA2aAgent` to a strict allowlist of trusted internal A2A URLs so attackers cannot induce the coordinator to fetch a malicious external `agent-card.json` (SSRF / rogue agent injection).

### 3. Prompt Injection & Input/Output Guardrails
* **Input Screening & Jailbreak Filtering:** Inspect incoming user prompts with **Google Cloud Model Armor** (or a classifier guardrail) before passing text to `finance_advisor_root` to block prompt injections attempting to override system instructions.
* **Strict Tool Input Validation:** Validate all LLM-generated tool arguments with **Pydantic** schemas (e.g., enforcing regex `^[A-Z]{1,5}$` on `ticker` in `get_stock_price` and positive finite bounds on `monthly_income`) so crafted payloads cannot reach downstream APIs or databases.
* **Output Sanitization & Disclaimers:** Filter final responses to ensure internal stack traces, debug events, environment variables, or sensitive user identifiers are never exposed, and append a standard financial disclaimer to every generated plan.

### 4. Secrets & Supply-Chain Hygiene
* **Secret Manager & Workload Identity:** Eliminate static `.env` files in production. Load secrets from **Google Secret Manager** at runtime, or authenticate to Vertex AI using **Application Default Credentials (ADC) / Workload Identity** so no static API keys exist on disk.
* **Dependency Pinning:** Pin exact package hashes in `requirements.txt` and run automated vulnerability scans (`pip-audit` or Trivy) in CI/CD before deploying container images.