# CopilotKit Skill for AI Coding Agents

[![Install with skills.sh](https://img.shields.io/badge/skills.sh-install-blue?style=flat-square)](https://skills.sh)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-green.svg?style=flat-square)](LICENSE)
[![CopilotKit](https://img.shields.io/badge/CopilotKit-28k%2B%20stars-orange?style=flat-square)](https://github.com/CopilotKit/CopilotKit)

A comprehensive skill that teaches AI coding agents how to build agentic applications with [CopilotKit](https://copilotkit.ai) -- the open-source framework for embedding AI copilots directly into React and Next.js apps.

## Installation

```bash
npx skills add nihalnihalani/copilotkit-skill@copilotkit
```

Works with any agent that supports the [skills.sh](https://skills.sh) standard.

## What's Covered

This skill provides deep knowledge across the entire CopilotKit ecosystem:

- **Quick Start & Setup** -- New project scaffolding, existing project integration, environment configuration
- **Core Hooks** -- `useCopilotReadable`, `useCopilotAction`, `useCopilotChat`, `useCopilotAdditionalInstructions`, `useCopilotChatSuggestions`
- **UI Components** -- `CopilotChat`, `CopilotSidebar`, `CopilotPopup`, `CopilotTextarea`
- **Generative UI** -- Static AG-UI tool calls, declarative A2UI rendering, open-ended MCP Apps
- **CoAgents & Shared State** -- `useCoAgent`, `useAgent`, bidirectional state sync with LangGraph
- **Human-in-the-Loop** -- Buildtime/runtime HITL patterns, approval flows, agent steering, `useLangGraphInterrupt`
- **Runtime & Adapters** -- `CopilotRuntime`, OpenAI/Anthropic/Google/Groq adapters, Next.js/Express/Hono endpoints
- **Python SDK** -- `LangGraphAgent`, FastAPI integration, actions, state management, event streaming
- **Styling & Customization** -- CSS variables, custom components, headless mode
- **Agent Protocols** -- AG-UI, MCP, and A2A protocol integration

## File Structure

```
copilotkit/
  SKILL.md                              # Main skill file (architecture, hooks, components, cheat sheet)
  references/
    generative-ui.md                    # Generative UI patterns (AG-UI, A2UI, MCP Apps)
    coagents-shared-state.md            # CoAgents and bidirectional state sync
    human-in-the-loop.md                # HITL workflows and approval patterns
    runtime-adapters.md                 # Runtime setup, LLM adapters, framework endpoints
    python-sdk.md                       # Python SDK for LangGraph/CrewAI agents
    styling-customization.md            # CSS theming, custom components, headless mode
```

## Compatible Agents

This skill works with any AI coding agent that supports the skills.sh format:

| Agent | Supported |
|-------|-----------|
| **Claude Code** (Anthropic) | Yes |
| **Cursor** | Yes |
| **Cline** | Yes |
| **Windsurf** | Yes |
| **Amp** | Yes |
| **Any skills.sh compatible agent** | Yes |

## How It Works

When installed, the skill gives your AI coding agent expert-level knowledge of CopilotKit. The agent will automatically reference this skill when you:

- Ask it to add a copilot to your React/Next.js app
- Work with CopilotKit hooks like `useCopilotAction` or `useCopilotReadable`
- Build generative UI with tool call rendering
- Set up CoAgents with LangGraph for bidirectional state sync
- Implement human-in-the-loop approval workflows
- Configure the CopilotKit runtime and LLM adapters
- Connect Python-based agents via the CopilotKit Python SDK

## Resources

- [CopilotKit Documentation](https://docs.copilotkit.ai)
- [CopilotKit GitHub](https://github.com/CopilotKit/CopilotKit)
- [AG-UI Protocol](https://github.com/ag-ui-protocol/ag-ui)
- [skills.sh](https://skills.sh)

## Contributing

Contributions are welcome! If you find outdated patterns, missing features, or want to improve the skill:

1. Fork the repository
2. Create a feature branch (`git checkout -b improve-skill`)
3. Edit the relevant files under `copilotkit/`
4. Submit a pull request

Please ensure your changes reflect the latest CopilotKit documentation and APIs.

## License

This project is licensed under the Apache License 2.0 -- see the [LICENSE](LICENSE) file for details.
