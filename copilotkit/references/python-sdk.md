# Python SDK

The `copilotkit` Python package enables building backend agents (LangGraph, CrewAI) that connect to the CopilotKit frontend.

## Installation

```bash
pip install copilotkit
```

**Dependencies:** langgraph>=0.3.25,<1.1.0, langchain>=0.3.0, fastapi>=0.115.0, ag-ui-langgraph>=0.0.22, ag-ui-core>=0.0.1

## Core Classes

### CopilotKitRemoteEndpoint

Primary orchestrator for agent execution and action handling.

```python
from copilotkit import CopilotKitRemoteEndpoint, LangGraphAgent, Action

endpoint = CopilotKitRemoteEndpoint(
    agents=[agent1, agent2],  # List of agent instances
    actions=[action1],        # List of backend actions
)
```

### LangGraphAgent

Wraps a compiled LangGraph `StateGraph` for CopilotKit integration.

```python
from copilotkit import LangGraphAgent
from langgraph.graph import StateGraph

# Define your graph
builder = StateGraph(MyState)
builder.add_node("search", search_node)
builder.add_node("analyze", analyze_node)
builder.add_edge("search", "analyze")
builder.set_entry_point("search")
graph = builder.compile()

# Wrap for CopilotKit
agent = LangGraphAgent(
    name="research_agent",
    graph=graph,
)
```

**Responsibilities:**
- Bidirectional message conversion (CopilotKit JSON ↔ LangChain `BaseMessage` types)
- State merging via `langgraph_default_merge_state()` (deep merge, last-write-wins)
- Checkpoint persistence for conversation continuity
- AG-UI protocol event emission

### LangGraphAGUIAgent

Enhanced version with CopilotKit-specific custom events:

```python
# Custom event names available in agent nodes:
"copilotkit_manually_emit_message"          # Emit complete text messages
"copilotkit_manually_emit_tool_call"        # Emit tool invocations
"copilotkit_manually_emit_intermediate_state"  # Emit state snapshots
"copilotkit_exit"                           # Signal agent completion
```

### Action

Define backend actions callable by agents:

```python
from copilotkit import Action

async def lookup_user(name: str) -> dict:
    return {"name": name, "email": f"{name}@example.com"}

action = Action(
    name="lookup_user",
    description="Look up a user by name",
    handler=lookup_user,
)
```

### CrewAIAgent

Dedicated wrapper for CrewAI-based agents:

```python
from copilotkit import CopilotKitRemoteEndpoint, CrewAIAgent

agent = CrewAIAgent(
    name="crew_agent",
    crew=my_crew,  # A CrewAI Crew instance
)

endpoint = CopilotKitRemoteEndpoint(agents=[agent])
```

Use `CrewAIAgent` when integrating CrewAI multi-agent crews with CopilotKit.

## Utility Functions

Helper functions for emitting events and controlling agent execution from within LangGraph nodes:

### `copilotkit_emit_state` — Emit intermediate state

```python
from copilotkit import copilotkit_emit_state

async def search_node(state, config):
    results = await perform_search(state["query"])
    # Emit partial state so frontend can show progress
    await copilotkit_emit_state(config, {"results": results, "status": "searching"})
    return {"results": results}
```

### `copilotkit_emit_message` — Emit messages during execution

```python
from copilotkit import copilotkit_emit_message

async def analyze_node(state, config):
    await copilotkit_emit_message(config, "Starting analysis of search results...")
    analysis = await run_analysis(state["results"])
    await copilotkit_emit_message(config, f"Analyzed {len(state['results'])} results.")
    return {"analysis": analysis}
```

### `copilotkit_emit_tool_call` — Manually emit tool calls

```python
from copilotkit import copilotkit_emit_tool_call

async def process_node(state, config):
    # Manually emit a tool call event for frontend rendering
    await copilotkit_emit_tool_call(config, name="show_chart", args={"data": state["data"]})
    return state
```

### `copilotkit_interrupt` — Trigger HITL from Python

```python
from copilotkit import copilotkit_interrupt

async def review_node(state, config):
    # Pause execution and ask user for approval
    response = await copilotkit_interrupt(
        config,
        message="Please review the draft before publishing.",
    )
    if response.get("approved"):
        return {"status": "approved"}
    return {"status": "rejected"}
```

### `copilotkit_exit` — Signal agent completion

```python
from copilotkit import copilotkit_exit

async def final_node(state, config):
    await copilotkit_exit(config)
    return state
```

## FastAPI Integration

```python
from copilotkit import CopilotKitRemoteEndpoint, LangGraphAgent
from copilotkit.fastapi import add_fastapi_endpoint
from fastapi import FastAPI

agent = LangGraphAgent(name="my_agent", graph=compiled_graph)
endpoint = CopilotKitRemoteEndpoint(agents=[agent])

app = FastAPI()
add_fastapi_endpoint(app, endpoint, "/copilotkit")

# Run: uvicorn main:app --port 8000
```

**Endpoint behavior:**
- Receives POST requests from CopilotRuntime
- Routes to appropriate agent based on request
- Body contains: messages, state, threadId, properties
- Returns JSON with execution results, updated state, events
- Supports streaming for long-running operations

## Flask Integration

```python
from copilotkit import CopilotKitRemoteEndpoint, LangGraphAgent
from copilotkit.integrations.flask import add_flask_endpoint
from flask import Flask

agent = LangGraphAgent(name="my_agent", graph=compiled_graph)
endpoint = CopilotKitRemoteEndpoint(agents=[agent])

app = Flask(__name__)
add_flask_endpoint(app, endpoint, "/copilotkit")

# Run: flask run --port 8000
```

## Connect Python Agent to JS Runtime

```typescript
// In your Next.js API route
import { CopilotRuntime, OpenAIAdapter, CustomHttpAgent } from "@copilotkit/runtime";

const runtime = new CopilotRuntime({
  agents: [
    new CustomHttpAgent({
      name: "my_agent",
      url: "http://localhost:8000/copilotkit",
    }),
  ],
});
```

## State Management

### CopilotKitState

Extends LangGraph's `MessagesState`:

```python
from langgraph.graph import MessagesState

class MyState(MessagesState):
    # Your custom state fields
    query: str = ""
    results: list = []

    # CopilotKit adds automatically:
    # copilotkit.actions: List of available tools
    # copilotkit.context: List of CopilotContextItem (from useCopilotReadable)
```

### CopilotContextItem

Data exposed from frontend via `useCopilotReadable()`:

```python
# Accessible in agent nodes as:
# state["copilotkit"]["context"] -> List[CopilotContextItem]
# Each item has: description (str), value (Any)
```

### Intermediate State Streaming

Configure state snapshots during execution:

```python
from copilotkit import copilotkit_customize_config

config = copilotkit_customize_config(
    emit_messages=True,                    # Emit message events
    emit_tool_calls=True,                  # Emit all tool calls (or list specific names)
    emit_intermediate_state=[              # Emit state during specific nodes
        {
            "state_key": "results",
            "tool": "search",
            "tool_argument": "query",
        }
    ],
)
```

## Message Types

| Type | Fields | Purpose |
|------|--------|---------|
| `TextMessage` | role, content, id | User/assistant/system text |
| `ActionExecutionMessage` | name, arguments, id, parentMessageId | Tool invocations |
| `ResultMessage` | actionExecutionId, actionName, result, id | Tool results |

## AG-UI Protocol Events

Events emitted during agent execution:

| Event | Purpose |
|-------|---------|
| `TEXT_MESSAGE_START` | Begin text message (messageId, role) |
| `TEXT_MESSAGE_CONTENT` | Stream content chunk (messageId, delta) |
| `TEXT_MESSAGE_END` | Complete message |
| `TOOL_CALL_START` | Begin tool invocation (toolCallId, toolName) |
| `TOOL_CALL_ARGS` | Stream tool arguments (delta) |
| `TOOL_CALL_END` | Complete tool call |
| `STATE_SNAPSHOT` | Full state update |

## Complete Integration Flow

```
1. Frontend sends HTTP request → Node.js CopilotRuntime
2. Runtime routes to Python FastAPI endpoint (HTTP POST)
3. FastAPI invokes CopilotKitRemoteEndpoint
4. Endpoint selects appropriate LangGraphAgent
5. Agent converts messages to LangChain format
6. State merging enriches with CopilotKit properties (context, actions)
7. CompiledStateGraph executes workflow nodes
8. Events emitted in AG-UI protocol format
9. Messages converted back to CopilotKit format
10. Results streamed back through runtime → AG-UI event stream → Frontend
```

## Full Example: Research Agent

```python
from copilotkit import CopilotKitRemoteEndpoint, LangGraphAgent
from copilotkit.fastapi import add_fastapi_endpoint
from langgraph.graph import StateGraph, MessagesState
from fastapi import FastAPI

class ResearchState(MessagesState):
    query: str = ""
    sources: list = []
    summary: str = ""

async def search_node(state: ResearchState):
    query = state["query"]
    # Access frontend context from useCopilotReadable
    context = state.get("copilotkit", {}).get("context", [])
    results = await perform_search(query, context)
    return {"sources": results}

async def summarize_node(state: ResearchState):
    summary = await generate_summary(state["sources"])
    return {"summary": summary}

builder = StateGraph(ResearchState)
builder.add_node("search", search_node)
builder.add_node("summarize", summarize_node)
builder.add_edge("search", "summarize")
builder.set_entry_point("search")
graph = builder.compile()

agent = LangGraphAgent(name="research_agent", graph=graph)
endpoint = CopilotKitRemoteEndpoint(agents=[agent])

app = FastAPI()
add_fastapi_endpoint(app, endpoint, "/copilotkit")
```
