# Consilium System Architecture

## Overview

Consilium is built as a modern web application with a clear separation between frontend and backend components, optimized for reliable performance, offline capabilities, and seamless AI integration.

## System Components

```
┌─────────────────────────────────────────────────────────────┐
│                       Client (PWA)                          │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │    React    │  │  IndexedDB  │  │  Service Workers    │  │
│  │  Components │  │  (DexieJS)  │  │  (Offline Support)  │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│                                                             │
└────────────────────────────┬────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                      API Layer (FastAPI)                    │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Resource   │  │   Service   │  │    Authentication   │  │
│  │  Endpoints  │  │    Layer    │  │    (Supabase)       │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│                                                             │
└──────────┬─────────────────────────────────────────┬────────┘
           │                                         │
           ▼                                         ▼
┌─────────────────────────┐              ┌─────────────────────┐
│   Database (Supabase)   │              │   AI Orchestration  │
│                         │              │      (CrewAI)       │
│  ┌─────────────────┐    │              │  ┌───────────────┐  │
│  │  User Data      │    │              │  │ Agent System  │  │
│  └─────────────────┘    │              │  └───────────────┘  │
│  ┌─────────────────┐    │              │  ┌───────────────┐  │
│  │  Notes          │◄───┼────────────► │  │ LLM Providers │  │
│  │  (with Tasks)   │    │              │  └───────────────┘  │
│  └─────────────────┘    │              │  ┌───────────────┐  │
│  ┌─────────────────┐    │              │  │External       │  │
│  │  Task Index     │    │              │  │Integrations   │  │
│  └─────────────────┘    │              │  └───────────────┘  │
│                         │              │                     │
└─────────────────────────┘              └─────────────────────┘
```

## Key Technical Decisions

### Frontend
- **React Framework**: Provides component-based architecture for the UI
- **Progressive Web App (PWA)**: Enables installation on devices and some offline functionality
- **IndexedDB (DexieJS)**: Client-side storage for offline data access
- **reachat**: UI components for chat functionality
- **BlockNote**: Extensible block-based editor with custom task blocks

### Backend
- **FastAPI**: High-performance Python web framework for API endpoints
- **PDM**: Python dependency management
- **Supabase**: Authentication and database services
- **CrewAI**: Orchestration for multiple AI agents
- **Task Indexing Service**: Indexes task blocks across notes for aggregated views
- **Composio**: Integration with external services via CrewAI for task synchronization

### AI Integration
- **Multi-provider Support**: OpenAI, Anthropic, xAI, OpenRouter, etc.
- **RAG System**: Vector database for document retrieval and contextual responses
- **Agent Specialization**: Dedicated agents for specific functions (email scanning, external app integration)
- **Tool-enabled Agents**: Agents with access to external tools and APIs via Composio

### Data Storage
- **Supabase PostgreSQL**: Primary database for structured data
- **Local Storage**: For offline capabilities and caching
- **Task-Note Relationships**: Tasks are stored as specialized blocks within notes with an index for efficient querying
- **Task Synchronization**: Task instances across multiple notes are synchronized through shared identifiers
- **External Reference Storage**: References to external systems (email, ClickUp, etc.) for synchronized tasks

## Communication Flow

1. **User Interaction**: Users primarily interact through the chat interface
2. **Request Processing**: Frontend sends structured requests to FastAPI backend
3. **Agent Orchestration**: CrewAI determines appropriate agent(s) for the request
4. **LLM Processing**: Selected AI providers process the request with relevant context
5. **Response Delivery**: Results are returned through the chat interface
6. **Data Persistence**: Relevant information is stored in Supabase and locally as needed
7. **External Synchronization**: For external tasks, changes are synchronized with source systems

## External Integration Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Consilium Core System                  │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                    CrewAI Orchestrator                  │
└───┬───────────────────────┬──────────────────────────┬──┘
    │                       │                          │
    ▼                       ▼                          ▼
┌─────────────┐     ┌──────────────┐          ┌──────────────┐
│ Email Agent │     │ ClickUp Agent│    ...   │ Custom Agents│
└──────┬──────┘     └───────┬──────┘          └───────┬──────┘
       │                    │                         │
       ▼                    ▼                         ▼
┌──────────────┐     ┌─────────────┐           ┌────────────┐
│ Email API    │     │ Composio    │    ...    │ Direct API │
│ Integration  │     │ Integration │           │ Integration│
└──────────────┘     └─────────────┘           └────────────┘
```

### Email Integration

The email integration enables two key capabilities:

1. **Email Task Creation**:
   - Email Agent scans user's inbox for actionable emails
   - Creates tasks with references to the original emails
   - Tasks maintain metadata about the email source

2. **Email Reply Functionality**:
   - User can reply to email tasks via the `/email-reply` command
   - Agent helps draft appropriate responses
   - Replies are sent through connected email service
   - Tasks are updated to reflect reply status

### ClickUp Integration

// ... existing content about ClickUp ...

## Synchronized Task Model

Consilium implements a synchronized task model that enables the same task to appear in multiple notes:

1. **Task Instances**: Tasks can exist as blocks in multiple notes
2. **Shared Identity**: All instances of the same task share a common identifier
3. **Central Index**: A task index service tracks and synchronizes all task instances
4. **Consistent Updates**: Changes to any instance propagate to all instances
5. **Source Awareness**: Each task maintains its origin information (note, email, ClickUp) across all instances

This approach provides flexibility in organizing tasks across different contexts while maintaining data consistency.

## Scalability Considerations

- **Agent Extensibility**: Architecture allows for new agents to be added without restructuring
- **Offline-First Approach**: Core functionality available without internet connection
- **Cross-Provider AI**: Not locked to a single AI provider
- **Modular Components**: Separate concerns for easier maintenance and scaling
- **Unified Data Model**: Tasks as part of notes provides a simpler data model while maintaining all functionality
- **External Integration Framework**: Consistent pattern for adding new external data sources 