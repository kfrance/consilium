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
│ - Memory      │ │ - Todo Manager│ │ - Transcriber │
│               │ │ - Reflection  │ │ - Integrator  │
└───────────────┘ └───────────────┘ └───────────────┘
```

## Agent Types

### Core Agents

These agents form the backbone of the system and manage the overall user experience:

- **Planner Agent**: Helps create and manage hierarchical plans from 5-year goals down to daily tasks, coordinates between specialized agents, and synthesizes recommendations into cohesive plans
- **Memory Agent**: Maintains user context and relevant information across sessions

### Feature Agents

These agents provide specific functionality within the application:

- **Note Taker Agent**: Specialized in managing notes, organizing information, and knowledge retrieval
- **Todo Manager**: Handles task creation, organization, and tracking across the system
- **Reflection Agent**: Guides user through reflection processes and analyzes progress
- **Save Command Agent**: Processes `/save` commands to create notes from conversations (see [Chat Commands](../features/chat-commands.md))

### Utility Agents

These agents provide supporting functionality to enhance the overall system:

- **RAG Agent**: Implements retrieval-augmented generation to provide context-aware responses using user documents
- **Email Agent**: Scans emails for task-like content and creates tasks; handles email replies
- **ClickUp Agent**: Synchronizes tasks with ClickUp and other external services
- **Integration Agent**: Connects with external services through Composio

## Specialized Agents

### Email Agent

The Email Agent handles all email-related functionality within the system:

- **Email Scanning**: Periodically scans the user's inbox for task-like content
- **Task Creation**: Creates tasks from emails with appropriate metadata
- **Reply Handling**: Composes and sends replies to email tasks via the `/email-reply` command
- **Draft Assistance**: Helps users draft professional email responses
- **Email Authentication**: Securely manages OAuth connection to email services

The Email Agent uses natural language processing to:
1. Identify emails that contain action items or requests
2. Extract key information for task creation
3. Understand user intent for composing replies
4. Generate appropriate response content

**Email Task Workflow**:
```
Email Agent scans inbox ──► Identifies actionable emails
         │
         ▼
Creates task with email metadata ──► Task appears in task views
         │
         ▼
User can reply via `/email-reply @taskname` ──► Email Agent helps draft and send reply
         │
         ▼
Email is sent, task is updated with reply information
```

### ClickUp Agent

// ... add similar details for the ClickUp Agent ...

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

## Agent Capabilities

Agents execute their functions through access to specialized tools. Each agent is granted access to specific tools based on their role and responsibilities. These tools allow agents to:

- Create and manipulate data in the system
- Access external services and APIs
- Process and transform information
- Take actions on behalf of the user

For a detailed list of available tools and their implementations, see the [Agent Tools](../technical/agent-tools.md) documentation.

## Command Processing

Agents can be triggered through explicit chat commands (prefixed with `/`) that users type into the chat interface. These commands route to specific agents with the appropriate tools to fulfill the request.

For example, when a user types `/save`, the request is routed to the Save Command Agent which has access to the note creation tools. The agent processes the command parameters and creates a note based on the conversation content.

For more information about available commands, see the [Chat Commands](../features/chat-commands.md) documentation. 