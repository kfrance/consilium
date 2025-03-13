# Multi-Agent AI System

## Overview

Consilium employs a multi-agent AI architecture where specialized AI agents work together to provide comprehensive assistance. This approach allows each agent to focus on specific domains while cooperating to deliver unified experiences.

## Agent Architecture

```
┌─────────────────────────────────────────────┐
│              Agent Orchestrator             │
│                  (CrewAI)                   │
└───────────┬─────────────┬─────────────┬─────┘
            │             │             │
            ▼             ▼             ▼
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│  Core Agents  │ │ Feature Agents│ │ Utility Agents│
└───────┬───────┘ └───────┬───────┘ └───────┬───────┘
        │                 │                 │
        ▼                 ▼                 ▼
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│ - Planner     │ │ - Note Taker  │ │ - RAG Agent   │
│ - Coordinator │ │ - Todo Manager│ │ - Transcriber │
│ - Memory      │ │ - Reflection  │ │ - Integrator  │
└───────────────┘ └───────────────┘ └───────────────┘
```

## Agent Types

### Core Agents

These agents form the backbone of the system and manage the overall user experience:

- **Planner Agent**: Helps create and manage hierarchical plans from 5-year goals down to daily tasks
- **Coordinator Agent**: Directs user requests to appropriate specialized agents and synthesizes responses
- **Memory Agent**: Maintains user context and relevant information across sessions

### Feature Agents

These agents provide specific functionality within the application:

- **Note Taker Agent**: Specialized in managing notes, organizing information, and knowledge retrieval
- **Todo Manager**: Handles task creation, organization, and tracking across the system
- **Reflection Agent**: Guides user through reflection processes and analyzes progress

### Utility Agents

These agents provide supporting functionality to enhance the overall system:

- **RAG Agent**: Implements retrieval-augmented generation to provide context-aware responses using user documents
- **Transcriber Agent**: Converts images of handwritten text to digital text
- **Integrator Agent**: Connects with external services like ClickUp and email

## Multi-Provider Support

One of Consilium's key features is the ability to use different LLM providers based on:

1. **Agent Requirements**: Specific agents may perform better with certain models
2. **User Preferences**: Users can select preferred providers for different agents
3. **Cost Optimization**: Balance between performance and cost
4. **Redundancy**: Fallback options if a provider is unavailable

### Supported Providers

- OpenAI (GPT models)
- Anthropic (Claude models)
- xAI (Grok models)
- OpenRouter (for accessing various models)
- Local models (via optional integrations)

## Agent Communication and Orchestration

CrewAI provides the orchestration layer that enables:

1. **Dynamic Agent Selection**: Choosing the right agent(s) for each user request
2. **Sequential Workflows**: Chaining agents together for complex tasks
3. **Parallel Processing**: Multiple agents working simultaneously when appropriate
4. **Memory Sharing**: Common knowledge base accessible to all agents
5. **Conflict Resolution**: Resolving contradictory outputs between agents

## Extensibility

The agent system is designed to be extensible:

- **Plugin Architecture**: Allows new agents to be added without modifying core code
- **Customizable Behaviors**: Agents can be tailored to user preferences
- **Composable Workflows**: Agents can be composed in different configurations
- **Provider Agnostic**: New LLM providers can be integrated as they become available

## Limitations and Constraints

- **Cost Management**: Multiple agent interactions can increase token usage
- **Latency Challenges**: Complex multi-agent workflows may increase response time
- **Context Limitations**: Each agent has limited context window with current LLM technology
- **Consistency**: Ensuring consistent personality and information across agents 