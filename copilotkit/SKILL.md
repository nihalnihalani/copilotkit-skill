---
name: copilotkit
description: Build AI copilots and agentic applications using CopilotKit, the open-source framework for in-app AI agents. Use this skill when building React/Next.js/Angular apps with CopilotKit, implementing chat UIs, frontend/backend actions, shared state, human-in-the-loop workflows, generative UI, CoAgents with LangGraph, or connecting agents via AG-UI/MCP/A2A protocols. Triggers on CopilotKit, copilotkit, useCopilotAction, useCopilotReadable, useCoAgent, useAgent, CopilotRuntime, CopilotChat, CopilotSidebar, CopilotPopup, CopilotTextarea, AG-UI, agentic frontend, in-app AI copilot, useFrontendTool, useRenderToolCall, useDefaultTool, useCoAgentStateRender, useLangGraphInterrupt, useCopilotChat, useCopilotAdditionalInstructions, useCopilotChatSuggestions, useHumanInTheLoop, CopilotTask, copilot runtime, LangGraphAgent, BasicAgent, BuiltInAgent, CopilotKitRemoteEndpoint, A2UI, MCP Apps.
---

# CopilotKit

Full-stack open-source framework (MIT, v1.50) for building agentic applications with AI copilots embedded directly in React and Angular UIs. Angular support via `@copilotkitnext/angular` (Angular 18+19). 28k+ GitHub stars.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  Frontend Layer                                      │
│  @copilotkit/react-core  - Provider, hooks           │
│  @copilotkit/react-ui    - Chat components, styles   │
└──────────────┬──────────────────────────────────────┘
               │ AG-UI Protocol (HTTP event streaming)
┌──────────────▼──────────────────────────────────────┐
│  Backend Layer                                       │
│  @copilotkit/runtime     - CopilotRuntime            │
│  LLM Adapters: OpenAI, Anthropic, Google, Groq, etc  │
└──────────────┬──────────────────────────────────────┘
               │ HTTP (AG-UI events)
┌──────────────▼──────────────────────────────────────┐
│  Agent Layer (optional)                              │
│  copilotkit (Python SDK) - LangGraph, CrewAI, etc    │
│  Or: Google ADK, AWS Strands, Microsoft Agent FW     │
└─────────────────────────────────────────────────────┘
```

## Quick Start

### New project
```bash
npx copilotkit@latest create -f next
```

### Existing project
```bash
npm install @copilotkit/react-core @copilotkit/react-ui @copilotkit/runtime
```

### Environment
```bash
# .env.local
OPENAI_API_KEY="sk-..."
# or ANTHROPIC_API_KEY, GOOGLE_API_KEY, GROQ_API_KEY
```

### Backend API route (`app/api/copilotkit/route.ts`)
```typescript
import {
  CopilotRuntime,
  OpenAIAdapter, // or AnthropicAdapter, GoogleGenerativeAIAdapter, GroqAdapter, LangChainAdapter
  copilotRuntimeNextJSAppRouterEndpoint,
} from "@copilotkit/runtime";
import { NextRequest } from "next/server";

const serviceAdapter = new OpenAIAdapter({ model: "gpt-4o" });
const runtime = new CopilotRuntime();

export const POST = async (req: NextRequest) => {
  const { handleRequest } = copilotRuntimeNextJSAppRouterEndpoint({
    runtime,
    serviceAdapter,
    endpoint: "/api/copilotkit",
  });
  return handleRequest(req);
};
```

### Frontend provider (`layout.tsx`)
```typescript
import { CopilotKit } from "@copilotkit/react-core";
import "@copilotkit/react-ui/styles.css";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <CopilotKit runtimeUrl="/api/copilotkit">
          {children}
        </CopilotKit>
      </body>
    </html>
  );
}
```

### Chat UI (`page.tsx`)
```typescript
import { CopilotPopup } from "@copilotkit/react-ui"; // or CopilotSidebar, CopilotChat

export default function Home() {
  return (
    <>
      <YourApp />
      <CopilotPopup
        instructions="You are an AI assistant for this app."
        labels={{ title: "Assistant", initial: "How can I help?" }}
      />
    </>
  );
}
```

## Core Hooks

### `useCopilotReadable` — Expose app state to LLM
```typescript
useCopilotReadable({
  description: "Current user profile and preferences",
  value: { name: user.name, role: user.role, preferences },
});
```

### `useCopilotAction` — Define executable actions
```typescript
useCopilotAction({
  name: "addItem",
  description: "Add a new item to the list",
  parameters: [
    { name: "title", type: "string", required: true },
    { name: "priority", type: "string", description: "low, medium, or high" },
  ],
  handler: async ({ title, priority = "medium" }) => {
    setItems(prev => [...prev, { id: Date.now().toString(), title, priority }]);
    return `Added "${title}" with ${priority} priority`;
  },
});
```

### `useCopilotChat` — Programmatic chat control
```typescript
const { appendMessage, stopGeneration, reset, reloadMessages } = useCopilotChat();
```

### `useCopilotAdditionalInstructions` — Dynamic context-aware prompts
```typescript
useCopilotAdditionalInstructions({
  instructions: "User is on the settings page. Help them configure preferences.",
});
```

### `useCopilotChatSuggestions` — Auto-generate suggestions from app state
```typescript
useCopilotChatSuggestions({
  instructions: "Suggest actions based on the current app state.",
});
```

### `CopilotTask` — Run one-off programmatic tasks
```typescript
import { CopilotTask } from "@copilotkit/react-core";

const task = new CopilotTask({ instructions: "Summarize the data" });
await task.run(context);
```

## Advanced Patterns

Detailed guides organized by topic — load only what's needed:

- **Generative UI** (static AG-UI, declarative A2UI, open-ended MCP Apps): See [references/generative-ui.md](references/generative-ui.md)
- **Shared State & CoAgents** (useCoAgent, useAgent, bidirectional sync, LangGraph): See [references/coagents-shared-state.md](references/coagents-shared-state.md)
- **Human-in-the-Loop** (buildtime/runtime HITL, approval flows, agent steering): See [references/human-in-the-loop.md](references/human-in-the-loop.md)
- **Runtime & Adapters** (all LLM adapters, framework endpoints, backend actions): See [references/runtime-adapters.md](references/runtime-adapters.md)
- **Python SDK** (LangGraphAgent, FastAPI, actions, state, events): See [references/python-sdk.md](references/python-sdk.md)
- **Styling & Customization** (CSS, custom components, headless mode): See [references/styling-customization.md](references/styling-customization.md)

## UI Components

| Component | Import | Use case |
|-----------|--------|----------|
| `CopilotChat` | `@copilotkit/react-ui` | Embedded inline chat panel |
| `CopilotSidebar` | `@copilotkit/react-ui` | Collapsible sidebar chat |
| `CopilotPopup` | `@copilotkit/react-ui` | Floating popup chat |
| `CopilotTextarea` | `@copilotkit/react-ui` | AI-powered textarea drop-in |

All accept `instructions`, `labels`, `suggestions`, custom message/input components, and `observabilityHooks`.

## CopilotKit Provider Props

| Prop | Type | Purpose |
|------|------|---------|
| `runtimeUrl` | `string` | Self-hosted runtime endpoint |
| `publicApiKey` | `string` | Copilot Cloud API key |
| `headers` | `object` | Custom auth headers |
| `credentials` | `string` | Cookie handling (`"include"` for cross-origin) |
| `agent` | `string` | Default agent name |
| `properties` | `object` | Thread metadata, authorization |
| `onError` | `function` | Error handler callback |
| `showDevConsole` | `boolean` | Dev error banners |
| `enableInspector` | `boolean` | Debugging inspector tool |
| `renderActivityMessages` | `array` | Custom renderers (A2UI, etc.) |

## Agent Protocols

| Protocol | Purpose | Package |
|----------|---------|---------|
| **AG-UI** | Agent ↔ User interaction, event streaming | `@ag-ui/core`, `@ag-ui/client` |
| **MCP** | Agent ↔ External tools | MCP server integration |
| **A2A** | Agent ↔ Agent communication | A2A protocol support |

## Supported Agent Frameworks

LangGraph, Google ADK, Microsoft Agent Framework, AWS Strands, CrewAI, Mastra, PydanticAI, AG2, VoltAgent, Blaxel.

## LLM Adapters

`OpenAIAdapter`, `AnthropicAdapter`, `GoogleGenerativeAIAdapter`, `GroqAdapter`, `LangChainAdapter`, `OpenAIAssistantAdapter`.

## Key Packages

| Package | Purpose |
|---------|---------|
| `@copilotkit/react-core` | Provider + all hooks |
| `@copilotkit/react-ui` | Chat UI components + styles |
| `@copilotkit/runtime` | Backend runtime + LLM adapters + framework endpoints |
| `copilotkit` (Python) | Python SDK for LangGraph/CrewAI agents |
| `@copilotkitnext/angular` | Angular SDK (Angular 18+19) |
| `@ag-ui/core` | AG-UI protocol types/events |
| `@ag-ui/client` | AG-UI client implementation |

## Error Handling

### Frontend `onError` callback

```typescript
<CopilotKit
  runtimeUrl="/api/copilotkit"
  onError={(error) => {
    console.error("CopilotKit error:", error);
    toast.error("AI assistant encountered an error");
  }}
>
```

### Tool render `"failed"` status

All render functions receive `status === "failed"` when a tool errors. Always handle this:

```typescript
render: ({ status, args, result }) => {
  if (status === "failed") return <ErrorCard message="Tool execution failed" />;
  // ...
}
```

### Python SDK exception types

| Exception | Meaning |
|-----------|---------|
| `ActionNotFoundException` | Requested action not registered |
| `AgentNotFoundException` | Requested agent not found in endpoint |

## Versioning

Current version: **v1.50**. The v2 runtime interface is available at the `/v2` path. Next-generation packages use the `@copilotkitnext/*` namespace (e.g., `@copilotkitnext/angular`).

## Common Patterns Cheat Sheet

| Want to... | Use |
|-----------|-----|
| Show a chat bubble | `<CopilotPopup>` |
| Give LLM app context | `useCopilotReadable()` |
| Let LLM call functions | `useCopilotAction()` |
| Render UI from tool calls | `useFrontendTool()` or `useRenderToolCall()` |
| Default fallback tool renderer | `useDefaultTool()` |
| Sync state bidirectionally | `useCoAgent()` |
| Show agent progress | `useCoAgentStateRender()` |
| Ask user for approval | `useHumanInTheLoop()` |
| Handle LangGraph interrupts | `useLangGraphInterrupt()` |
| Control chat programmatically | `useCopilotChat()` |
| Run one-off tasks | `CopilotTask` |
| Connect Python agent | `CopilotKitRemoteEndpoint` + `LangGraphAgent` |
