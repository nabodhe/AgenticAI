# Troubleshooting: `ModuleNotFoundError: No module named 'sse_starlette'`

> **Navigation:** [← Back to Index](../README.md) | [Chapter 8: Expose Market Data Agent via A2A](./08-market-data-a2a-server.md) | [Production & Operations FAQ](./FAQ.md)

| Metadata | Details |
| :--- | :--- |
| **Component** | Standalone Market Data A2A Server (`src/market_data_agent.py`) |
| **Error Type** | `ModuleNotFoundError` during ASGI application lifespan startup |
| **Affected Command** | `uvicorn market_data_agent:a2a_app --host 0.0.0.0 --port 8001` |

---

## 1. Error Description & Stack Trace

When starting the standalone A2A server (`market_data_agent:a2a_app`) with Uvicorn, the Starlette lifespan initialization fails because `sse_starlette` is missing from the virtual environment:

```text
self._config = config or A2aAgentExecutorConfig()
...\src\.venv\Lib\site-packages\google\adk\a2a\utils\agent_to_a2a.py:251: UserWarning: [EXPERIMENTAL] AgentCardBuilder: ADK Implementation for A2A support (A2aAgentExecutor, RemoteA2aAgent and corresponding supporting components etc.) is in experimental mode and is subject to breaking changes. A2A protocol and SDK are themselves not experimental. Once it's stable enough the experimental mode will be removed. Your feedback is welcome.
  card_builder = AgentCardBuilder(
Market Data Agent is running as an A2A server on http://localhost:8001/<starlette.applications.Starlette object at 0x00000173A11A6F90>
INFO:     Started server process [16168]
INFO:     Waiting for application startup.
ERROR:    Traceback (most recent call last):
  File "...\src\.venv\Lib\site-packages\starlette\routing.py", line 656, in lifespan
    async with self.lifespan_context(app) as maybe_state:
               ~~~~~~~~~~~~~~~~~~~~~^^^^^
  File "...\Python313\Lib\contextlib.py", line 214, in __aenter__
    return await anext(self.gen)
           ^^^^^^^^^^^^^^^^^^^^^
  File "...\src\.venv\Lib\site-packages\google\adk\a2a\utils\agent_to_a2a.py", line 280, in _combined_lifespan
    await setup_a2a(app)
  File "...\src\.venv\Lib\site-packages\google\adk\a2a\utils\agent_to_a2a.py", line 266, in setup_a2a
    _compat.attach_a2a_routes_to_app(
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^
        app,
        ^^^^
    ...<4 lines>...
        prefix=prefix,
        ^^^^^^^^^^^^^^
    )
    ^
  File "...\src\.venv\Lib\site-packages\google\adk\a2a\_compat.py", line 946, in attach_a2a_routes_to_app
    from a2a.server.routes import create_agent_card_routes
  File "...\src\.venv\Lib\site-packages\a2a\server\routes\__init__.py", line 9, in <module>
    from a2a.server.routes.jsonrpc_routes import create_jsonrpc_routes
  File "...\src\.venv\Lib\site-packages\a2a\server\routes\jsonrpc_routes.py", line 22, in <module>
    from a2a.server.routes.jsonrpc_dispatcher import JsonRpcDispatcher
  File "...\src\.venv\Lib\site-packages\a2a\server\routes\jsonrpc_dispatcher.py", line 12, in <module>
    from a2a.compat.v0_3.jsonrpc_adapter import JSONRPC03Adapter
  File "...\src\.venv\Lib\site-packages\a2a\compat\v0_3\jsonrpc_adapter.py", line 6, in <module>
    from sse_starlette.sse import EventSourceResponse
ModuleNotFoundError: No module named 'sse_starlette'

ERROR:    Application startup failed. Exiting.
```

---

## 2. Root Cause

`a2a-sdk`'s HTTP/JSON-RPC server routing relies on `sse-starlette` for Server-Sent Events (SSE) streaming support. In newer `a2a-sdk` releases, HTTP server dependencies are packaged under the `a2a-sdk[http-server]` extra rather than the base `a2a-sdk` package.

---

## 3. Resolution Steps

### Step 1: Install `sse-starlette` and `a2a-sdk[http-server]`
With your virtual environment activated, install the missing HTTP server dependencies:

```bash
python -m pip install sse-starlette "a2a-sdk[http-server]"
```

**Verification:**
```bash
python -c "import sse_starlette; print('OK')"
```
Confirm the command outputs `OK` without errors.

### Step 2: Update `requirements.txt`
Freeze the updated environment dependencies so future installations include `sse-starlette`:

```bash
python -m pip freeze > requirements.txt
```

### Step 3: Restart the Market Data A2A Server
```bash
python -m uvicorn market_data_agent:a2a_app --host 0.0.0.0 --port 8001
```

**Verification:**
- The server logs `Application startup complete.` and `Uvicorn running on http://0.0.0.0:8001`.
- Opening `http://localhost:8001/.well-known/agent-card.json` in a browser returns the Market Data Agent JSON card.

> **Cleanup Tip:** In the startup log (`Market Data Agent is running as an A2A server on http://localhost:8001/<starlette.applications.Starlette object...>`), `a2a_app` was interpolated directly inside an f-string. Remove `{a2a_app}` from that `print()` statement in `src/market_data_agent.py` to keep startup logs clean.

---

## 4. Verification Installation Log

```text
(.venv) D:\Projects\src>python -m pip install sse-starlette "a2a-sdk[http-server]"
Collecting sse-starlette
  Downloading sse_starlette-3.7.0-py3-none-any.whl.metadata (15 kB)
Requirement already satisfied: a2a-sdk[http-server] in d:\projects\src\.venv\lib\site-packages (1.2.3)
Requirement already satisfied: starlette>=0.49.1 in d:\projects\src\.venv\lib\site-packages (from sse-starlette) (1.9.0)
Requirement already satisfied: anyio>=4.7.0 in d:\projects\src\.venv\lib\site-packages (from sse-starlette) (4.15.1)
Requirement already satisfied: google-api-core>=1.26.0 in d:\projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (2.34.0)
Requirement already satisfied: googleapis-common-protos>=1.70.0 in d:\projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (1.75.5)
Requirement already satisfied: httpx>=0.28.1 in d:\projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (0.28.1)
Requirement already satisfied: json-rpc>=1.15.0 in d:\projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (1.15.0)
Requirement already satisfied: packaging>=24.0 in d:\projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (26.3)
Requirement already satisfied: protobuf>=5.29.5 in d:\projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (7.36.2)
Requirement already satisfied: pydantic>=2.11.3 in d:\projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (2.14.1)
Downloading sse_starlette-3.7.0-py3-none-any.whl (18 kB)
Installing collected packages: sse-starlette
Successfully installed sse-starlette-3.7.0
```