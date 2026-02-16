# Runtime & Adapters

## Table of Contents
- [CopilotRuntime](#copilotruntime)
- [Framework Endpoints](#framework-endpoints)
- [LLM Adapters](#llm-adapters)
- [Backend Actions](#backend-actions)
- [Deployment Modes](#deployment-modes)
- [Processing Flow](#processing-flow)

## CopilotRuntime

Backend orchestrator managing LLM requests, tool execution, agent routing, and event streaming.

**Responsibilities:**
1. Request orchestration — processes GraphQL requests from frontend
2. LLM adapter management — interfaces with provider adapters
3. Agent execution — manages agent lifecycle and state sync
4. Action discovery — exposes frontend `useCopilotAction` definitions as tools
5. Event streaming — streams events via GraphQL subscriptions
6. Context management — aggregates `useCopilotReadable` context

## Framework Endpoints

| Framework | Function | Typical route file |
|-----------|----------|-------------------|
| Next.js App Router | `copilotRuntimeNextJSAppRouterEndpoint` | `app/api/copilotkit/route.ts` |
| Next.js Pages Router | `copilotRuntimeNextJSPagesRouterEndpoint` | `pages/api/copilotkit.ts` |
| Express/Node.js | `copilotRuntimeNodeHttpEndpoint` | Any Express route |
| NestJS | `copilotRuntimeNestEndpoint` | NestJS controller |

### Next.js App Router (most common)

```typescript
import {
  CopilotRuntime,
  OpenAIAdapter,
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

### Next.js Pages Router

```typescript
import {
  CopilotRuntime,
  OpenAIAdapter,
  copilotRuntimeNextJSPagesRouterEndpoint,
} from "@copilotkit/runtime";
import { NextApiRequest, NextApiResponse } from "next";

const serviceAdapter = new OpenAIAdapter({ model: "gpt-4o" });
const runtime = new CopilotRuntime();

export default copilotRuntimeNextJSPagesRouterEndpoint({
  runtime,
  serviceAdapter,
  endpoint: "/api/copilotkit",
});
```

### Express / Node.js

```typescript
import {
  CopilotRuntime,
  OpenAIAdapter,
  copilotRuntimeNodeHttpEndpoint,
} from "@copilotkit/runtime";
import express from "express";

const app = express();
const serviceAdapter = new OpenAIAdapter({ model: "gpt-4o" });
const runtime = new CopilotRuntime();

app.use("/api/copilotkit", copilotRuntimeNodeHttpEndpoint({
  runtime,
  serviceAdapter,
}));

app.listen(3001);
```

### With Remote Agent (Python backend)

```typescript
import {
  CopilotRuntime,
  OpenAIAdapter,
  CustomHttpAgent,
  copilotRuntimeNextJSAppRouterEndpoint,
} from "@copilotkit/runtime";

const serviceAdapter = new OpenAIAdapter({ model: "gpt-4o" });
const runtime = new CopilotRuntime({
  agents: [
    new CustomHttpAgent({
      name: "research_agent",
      url: "http://localhost:8000/copilotkit",
    }),
    new CustomHttpAgent({
      name: "writer_agent",
      url: "http://localhost:8001/copilotkit",
    }),
  ],
});

export const POST = async (req: NextRequest) => {
  const { handleRequest } = copilotRuntimeNextJSAppRouterEndpoint({
    runtime,
    serviceAdapter,
    endpoint: "/api/copilotkit",
  });
  return handleRequest(req);
};
```

## LLM Adapters

### OpenAIAdapter
```typescript
import { OpenAIAdapter } from "@copilotkit/runtime";
const adapter = new OpenAIAdapter({ model: "gpt-4o" });
// Also supports: gpt-4o-mini, gpt-4-turbo, o1, o3, etc.
```

### AnthropicAdapter
```typescript
import { AnthropicAdapter } from "@copilotkit/runtime";
const adapter = new AnthropicAdapter({ model: "claude-sonnet-4-5-20250929" });
// Also supports: claude-opus-4-6, claude-haiku-4-5-20251001
```

### GoogleGenerativeAIAdapter
```typescript
import { GoogleGenerativeAIAdapter } from "@copilotkit/runtime";
const adapter = new GoogleGenerativeAIAdapter({ model: "gemini-2.0-flash" });
// Also supports: gemini-2.5-pro, gemini-2.5-flash
```

### GroqAdapter
```typescript
import { GroqAdapter } from "@copilotkit/runtime";
const adapter = new GroqAdapter({ model: "llama-3.3-70b-versatile" });
```

### LangChainAdapter (custom chains)
```typescript
import { LangChainAdapter } from "@copilotkit/runtime";
const adapter = new LangChainAdapter({
  chainFn: async ({ messages, tools }) => {
    // Build any LangChain/LangGraph workflow
    const chain = prompt.pipe(model).pipe(parser);
    return chain.stream({ messages });
  },
});
```

### OpenAIAssistantAdapter
```typescript
import { OpenAIAssistantAdapter } from "@copilotkit/runtime";
const adapter = new OpenAIAssistantAdapter({
  assistantId: "asst_abc123",
});
```

## Backend Actions

Define server-side actions accessible to agents:

```typescript
const runtime = new CopilotRuntime({
  actions: [
    {
      name: "queryDatabase",
      description: "Execute a database query and return results",
      parameters: [
        { name: "query", type: "string", description: "SQL query", required: true },
        { name: "limit", type: "number", description: "Max rows", required: false },
      ],
      handler: async ({ query, limit = 100 }) => {
        const results = await db.query(query, { limit });
        return JSON.stringify(results);
      },
    },
    {
      name: "sendEmail",
      description: "Send an email notification",
      parameters: [
        { name: "to", type: "string", required: true },
        { name: "subject", type: "string", required: true },
        { name: "body", type: "string", required: true },
      ],
      handler: async ({ to, subject, body }) => {
        await emailService.send({ to, subject, body });
        return "Email sent successfully";
      },
    },
  ],
});
```

## Deployment Modes

| Mode | Frontend Config | Use case |
|------|----------------|----------|
| **Self-hosted** | `runtimeUrl="/api/copilotkit"` | Full control, own infrastructure |
| **Copilot Cloud** | `publicApiKey="ck_..."` | Managed runtime, no backend needed |
| **Hybrid** | Both props | Cloud features + custom runtime |

### Self-hosted
```typescript
<CopilotKit runtimeUrl="/api/copilotkit">
```

### Copilot Cloud
```typescript
<CopilotKit publicApiKey="ck_your_key_here">
```

### Hybrid (Cloud + custom agents)
```typescript
<CopilotKit publicApiKey="ck_..." runtimeUrl="/api/copilotkit">
```

### Authentication

```typescript
// Custom headers
<CopilotKit
  runtimeUrl="/api/copilotkit"
  headers={{ Authorization: `Bearer ${token}` }}
/>

// Cookies (cross-origin)
<CopilotKit
  runtimeUrl="https://api.example.com/copilotkit"
  credentials="include"
/>

// LangGraph Platform auth
<CopilotKit
  runtimeUrl="/api/copilotkit"
  properties={{ authorization: { token: langGraphToken } }}
/>
```

## Processing Flow

```
1. Framework endpoint receives GraphQL request from frontend
2. Runtime aggregates context from all useCopilotReadable() calls
3. Frontend actions from useCopilotAction() converted to tool definitions
4. If agent name specified → route to agent (CustomHttpAgent)
   Otherwise → direct LLM call via service adapter
5. LLM processes messages + context + tools
6. Tool calls executed (frontend or backend), results returned
7. All events streamed to frontend via GraphQL subscriptions:
   - TEXT_MESSAGE_START/CONTENT/END
   - TOOL_CALL_START/ARGS/END
   - STATE_SNAPSHOT
8. Frontend hooks re-render UI with new data
```
