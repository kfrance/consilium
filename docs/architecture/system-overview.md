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
│  │  Notes          │    │              │  │ LLM Providers │  │
│  └─────────────────┘    │              │  └───────────────┘  │
│  ┌─────────────────┐    │              │                     │
│  │  Tasks & Goals  │    │              │                     │
│  └─────────────────┘    │              │                     │
│                         │              │                     │
└─────────────────────────┘              └─────────────────────┘
```

## Key Technical Decisions

### Frontend
- **React Framework**: Provides component-based architecture for the UI
- **Progressive Web App (PWA)**: Enables installation on devices and some offline functionality
- **IndexedDB (DexieJS)**: Client-side storage for offline data access
- **reachat**: UI components for chat functionality

### Backend
- **FastAPI**: High-performance Python web framework for API endpoints
- **PDM**: Python dependency management
- **Supabase**: Authentication and database services
- **CrewAI**: Orchestration for multiple AI agents

### AI Integration
- **Multi-provider Support**: OpenAI, Anthropic, xAI, OpenRouter, etc.
- **RAG System**: Vector database for document retrieval and contextual responses

### Data Storage
- **Supabase PostgreSQL**: Primary database for structured data
- **Local Storage**: For offline capabilities and caching

## Communication Flow

1. **User Interaction**: Users primarily interact through the chat interface
2. **Request Processing**: Frontend sends structured requests to FastAPI backend
3. **Agent Orchestration**: CrewAI determines appropriate agent(s) for the request
4. **LLM Processing**: Selected AI providers process the request with relevant context
5. **Response Delivery**: Results are returned through the chat interface
6. **Data Persistence**: Relevant information is stored in Supabase and locally as needed

## Scalability Considerations

- **Agent Extensibility**: Architecture allows for new agents to be added without restructuring
- **Offline-First Approach**: Core functionality available without internet connection
- **Cross-Provider AI**: Not locked to a single AI provider
- **Modular Components**: Separate concerns for easier maintenance and scaling 