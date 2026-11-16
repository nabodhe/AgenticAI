 ERROR occurred while running application. starlette.applications.Starlette
 
 self._config = config or A2aAgentExecutorConfig()
D:\***Projects\src\.venv\Lib\site-packages\google\adk\a2a\utils\agent_to_a2a.py:251: UserWarning: [EXPERIMENTAL] AgentCardBuilder: ADK Implementation for A2A support (A2aAgentExecutor, RemoteA2aAgent and corresponding supporting components etc.) is in experimental mode and is subject to breaking changes. A2A protocol and SDK are themselves not experimental. Once it's stable enough the experimental mode will be removed. Your feedback is welcome.
  card_builder = AgentCardBuilder(
Market Data Agent is running as an A2A server on http://localhost:8001/<starlette.applications.Starlette object at 0x00000173A11A6F90>
INFO:     Started server process [16168]
INFO:     Waiting for application startup.
ERROR:    Traceback (most recent call last):
  File "D:\***Projects\src\.venv\Lib\site-packages\starlette\routing.py", line 656, in lifespan
    async with self.lifespan_context(app) as maybe_state:
               ~~~~~~~~~~~~~~~~~~~~~^^^^^
  File "C:\Python313\Lib\contextlib.py", line 214, in __aenter__
    return await anext(self.gen)
           ^^^^^^^^^^^^^^^^^^^^^
  File "D:\***Projects\src\.venv\Lib\site-packages\google\adk\a2a\utils\agent_to_a2a.py", line 280, in _combined_lifespan
    await setup_a2a(app)
  File "D:\***Projects\src\.venv\Lib\site-packages\google\adk\a2a\utils\agent_to_a2a.py", line 266, in setup_a2a
    _compat.attach_a2a_routes_to_app(
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^
        app,
        ^^^^
    ...<4 lines>...
        prefix=prefix,
        ^^^^^^^^^^^^^^
    )
    ^
  File "D:\***Projects\src\.venv\Lib\site-packages\google\adk\a2a\_compat.py", line 946, in attach_a2a_routes_to_app
    from a2a.server.routes import create_agent_card_routes
  File "D:\***Projects\src\.venv\Lib\site-packages\a2a\server\routes\__init__.py", line 9, in <module>
    from a2a.server.routes.jsonrpc_routes import create_jsonrpc_routes
  File "D:\***Projects\src\.venv\Lib\site-packages\a2a\server\routes\jsonrpc_routes.py", line 22, in <module>
    from a2a.server.routes.jsonrpc_dispatcher import JsonRpcDispatcher
  File "D:\***Projects\src\.venv\Lib\site-packages\a2a\server\routes\jsonrpc_dispatcher.py", line 12, in <module>
    from a2a.compat.v0_3.jsonrpc_adapter import JSONRPC03Adapter
  File "D:\***Projects\src\.venv\Lib\site-packages\a2a\compat\v0_3\jsonrpc_adapter.py", line 6, in <module>
    from sse_starlette.sse import EventSourceResponse
ModuleNotFoundError: No module named 'sse_starlette'

ERROR:    Application startup failed. Exiting.

Resolution : 

This error occurs because a2a-sdk's HTTP/JSON-RPC server routing relies on sse-starlette (for Server-Sent Events streaming support), which isn't installed in your .venv yet. (In newer a2a-sdk versions, HTTP server extras are packaged under a2a-sdk[http-server] rather than the bare a2a-sdk package.)

Install sse-starlette (or the full HTTP server extra for a2a-sdk) into your active virtual environment and restart the server:

Install sse-starlette and the a2a-sdk HTTP server dependencies into .venv:
Run below command.
python -m pip install sse-starlette "a2a-sdk[http-server]"

Verification: python -c "import sse_starlette; print('OK')" prints OK without errors.

Update your requirements.txt so this dependency is saved:
Run below command.
python -m pip freeze > requirements.txt

Restart the Market Data A2A server:
Run below command.
python -m uvicorn market_data_agent:a2a_app --host 0.0.0.0 --port 8001

Verification: The server completes startup with Application startup complete. and Uvicorn running on [http://0.0.0.0:8001](http://0.0.0.0:8001) (and http://localhost:8001/.well-known/agent-card.json returns your Agent Card JSON).

(Minor note: In your startup log Market Data Agent is running as an A2A server on http://localhost:8001/<starlette.applications.Starlette object...>, you likely printed a2a_app directly in an f-string—removing {a2a_app} from that print() statement will clean up the console message.)

LOG After installation of starlette
(.venv) D:\***Projects\src>python -m pip install sse-starlette "a2a-sdk[http-server]"
Collecting sse-starlette
  Downloading sse_starlette-3.7.0-py3-none-any.whl.metadata (15 kB)
Requirement already satisfied: a2a-sdk[http-server] in d:\***projects\src\.venv\lib\site-packages (1.2.3)
Requirement already satisfied: starlette>=0.49.1 in d:\***projects\src\.venv\lib\site-packages (from sse-starlette) (1.9.0)
Requirement already satisfied: anyio>=4.7.0 in d:\***projects\src\.venv\lib\site-packages (from sse-starlette) (4.15.1)
Requirement already satisfied: google-api-core>=1.26.0 in d:\***projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (2.34.0)
Requirement already satisfied: googleapis-common-protos>=1.70.0 in d:\***projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (1.75.5)
Requirement already satisfied: httpx>=0.28.1 in d:\***projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (0.28.1)
Requirement already satisfied: json-rpc>=1.15.0 in d:\***projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (1.15.0)
Requirement already satisfied: packaging>=24.0 in d:\***projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (26.3)
Requirement already satisfied: protobuf>=5.29.5 in d:\***projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (7.36.2)
Requirement already satisfied: pydantic>=2.11.3 in d:\***projects\src\.venv\lib\site-packages (from a2a-sdk[http-server]) (2.14.1)
Requirement already satisfied: idna>=2.8 in d:\***projects\src\.venv\lib\site-packages (from anyio>=4.7.0->sse-starlette) (3.20)
Requirement already satisfied: typing_extensions>=4.16.0 in d:\***projects\src\.venv\lib\site-packages (from anyio>=4.7.0->sse-starlette) (4.16.0)
Requirement already satisfied: proto-plus<2.0.0,>=1.26.1 in d:\***projects\src\.venv\lib\site-packages (from google-api-core>=1.26.0->a2a-sdk[http-server]) (1.29.0)
Requirement already satisfied: google-auth<3.0.0,>=2.14.1 in d:\***projects\src\.venv\lib\site-packages (from google-api-core>=1.26.0->a2a-sdk[http-server]) (2.62.0)
Requirement already satisfied: requests<3.0.0,>=2.33.0 in d:\***projects\src\.venv\lib\site-packages (from google-api-core>=1.26.0->a2a-sdk[http-server]) (2.34.2)
Requirement already satisfied: certifi in d:\***projects\src\.venv\lib\site-packages (from httpx>=0.28.1->a2a-sdk[http-server]) (2026.10.19)
Requirement already satisfied: httpcore==1.* in d:\***projects\src\.venv\lib\site-packages (from httpx>=0.28.1->a2a-sdk[http-server]) (1.0.9)
Requirement already satisfied: h11>=0.16 in d:\***projects\src\.venv\lib\site-packages (from httpcore==1.*->httpx>=0.28.1->a2a-sdk[http-server]) (0.16.0)
Requirement already satisfied: annotated-types>=0.6.0 in d:\***projects\src\.venv\lib\site-packages (from pydantic>=2.11.3->a2a-sdk[http-server]) (0.8.0)
Requirement already satisfied: pydantic-core==2.50.1 in d:\***projects\src\.venv\lib\site-packages (from pydantic>=2.11.3->a2a-sdk[http-server]) (2.50.1)
Requirement already satisfied: typing-inspection>=0.4.4 in d:\***projects\src\.venv\lib\site-packages (from pydantic>=2.11.3->a2a-sdk[http-server]) (0.4.4)
Requirement already satisfied: pyasn1-modules>=0.2.1 in d:\***projects\src\.venv\lib\site-packages (from google-auth<3.0.0,>=2.14.1->google-api-core>=1.26.0->a2a-sdk[http-server]) (0.4.2)
Requirement already satisfied: cryptography>=38.0.3 in d:\***projects\src\.venv\lib\site-packages (from google-auth<3.0.0,>=2.14.1->google-api-core>=1.26.0->a2a-sdk[http-server]) (50.0.2)
Requirement already satisfied: charset_normalizer<4,>=2 in d:\***projects\src\.venv\lib\site-packages (from requests<3.0.0,>=2.33.0->google-api-core>=1.26.0->a2a-sdk[http-server]) (3.5.2)
Requirement already satisfied: urllib3<3,>=1.26 in d:\***projects\src\.venv\lib\site-packages (from requests<3.0.0,>=2.33.0->google-api-core>=1.26.0->a2a-sdk[http-server]) (2.8.0)
Requirement already satisfied: cffi>=2.0.0 in d:\***projects\src\.venv\lib\site-packages (from cryptography>=38.0.3->google-auth<3.0.0,>=2.14.1->google-api-core>=1.26.0->a2a-sdk[http-server]) (2.1.1)
Requirement already satisfied: pyasn1<0.7.0,>=0.6.1 in d:\***projects\src\.venv\lib\site-packages (from pyasn1-modules>=0.2.1->google-auth<3.0.0,>=2.14.1->google-api-core>=1.26.0->a2a-sdk[http-server]) (0.6.4)
Requirement already satisfied: pycparser in d:\***projects\src\.venv\lib\site-packages (from cffi>=2.0.0->cryptography>=38.0.3->google-auth<3.0.0,>=2.14.1->google-api-core>=1.26.0->a2a-sdk[http-server]) (3.12)
Downloading sse_starlette-3.7.0-py3-none-any.whl (18 kB)
Installing collected packages: sse-starlette
Successfully installed sse-starlette-3.7.0

[notice] A new release of pip is available: 24.3.1 -> 26.3
[notice] To update, run: python.exe -m pip install --upgrade pip

