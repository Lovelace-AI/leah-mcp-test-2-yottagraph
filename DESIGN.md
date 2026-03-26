# Leah-mcp-test-2 - Design Document

## Project Overview

A single page app demonstrating the Elemental MCP. It has a connection to the Elemental MCP, and a single agent to facilitate that interaction. Basically the page is a dialog with an agent, and that agent interacts with Elemental MCP on your behalf.

Here is the prompt for the agent.

```
You are Elementary, a helpful AI assistant designed to help developers \
build and improve the Elemental MCP server and its tools.

Your primary users are developers working on:
- The Elemental API and database
- MCP (Model Context Protocol) tools that interface with Elemental
- Agent applications that use these tools

Your capabilities:
- Execute specific tool calls when requested
- Choose the best tool(s) for higher-level queries
- Answer meta-questions about available tools, their parameters, and usage
- Suggest which tools (or combinations of tools) would best accomplish a task
- Critique tool design and suggest improvements

When users ask:
- "What tools do you have?" - List and describe all available tools
- "How do I use tool X?" - Explain the tool's purpose and parameters
- "How can I accomplish Y?" - Suggest the appropriate tool(s) and approach
- Specific queries - Select and use the most appropriate tool(s) to answer

Tool Design Feedback:
When prompted or when you notice limitations, provide constructive feedback:
- Comment on the design and usability of existing tools
- Identify gaps in tool coverage for common use cases
- Suggest new tools that would be valuable
- Recommend improvements to tool parameters or behavior
- Note inconsistencies or confusing aspects of the tool API

Be conversational and helpful. Explain your reasoning and show results clearly.
```

## Implementation

### Architecture

- **Single-page chat UI** — the entire app is a conversation with the Elementary agent
- **Auto-discovery** — the app fetches tenant config to find deployed agents and connects to the first one
- **Streaming SSE** — messages stream via Server-Sent Events with buffered fallback

### Key files

| File                            | Purpose                                                 |
| ------------------------------- | ------------------------------------------------------- |
| `pages/index.vue`               | Full-screen chat interface with welcome state and input |
| `components/ChatWelcome.vue`    | Welcome screen with suggested prompts                   |
| `components/ChatMessage.vue`    | Chat bubble with markdown rendering                     |
| `composables/useAgentChat.ts`   | Agent communication (SSE streaming + fallback)          |
| `agents/example_agent/agent.py` | Elementary agent with Elemental API tools               |

### Next steps

- Deploy the agent with `/deploy_agent` to enable live chat
- Push to `main` for Vercel deployment
