# Backend Technical Design

## Overview

The Consilium backend is built using FastAPI, a modern, high-performance Python web framework optimized for API development. The architecture is designed around RESTful principles with WebSocket support for real-time features, emphasizing scalability, maintainability, and security.

## Technology Stack

### Core Framework

- **FastAPI**: Python web framework for building APIs
- **Pydantic**: Data validation and settings management
- **PDM**: Python dependency and virtual environment management
- **Uvicorn/Gunicorn**: ASGI servers for production deployment

### Database

- **Supabase**: PostgreSQL database with supporting services
- **SQLAlchemy**: SQL toolkit and ORM for database interactions
- **Alembic**: Database migration tool
- **Pydantic**: Model validation and serialization

### AI and ML

- **CrewAI**: Multi-agent AI orchestration framework
- **LangChain**: Framework for developing applications with LLMs
- **Various LLM Clients**: OpenAI, Anthropic, xAI, OpenRouter clients
- **Embedding Models**: Text embedding generation for RAG
- **Vector Database**: For storing and retrieving embeddings (Qdrant, etc.)

### Authentication and Security

- **Supabase Auth**: Authentication services
- **Python-Jose**: JWT token handling
- **Passlib**: Password hashing
- **CORS Middleware**: Cross-origin resource sharing controls

### Testing and Quality

- **Pytest**: Testing framework
- **Pytest-cov**: Test coverage reporting
- **Black**: Code formatting
- **Ruff**: Linting
- **Mypy**: Static type checking

## System Architecture

### Layered Architecture

```
┌────────────────────────────────────────────────────┐
│                  API Layer (Routers)               │
│                                                    │
│   ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │
│   │   User API  │  │  Notes API  │  │ Tasks API │  │
│   └─────────────┘  └─────────────┘  └───────────┘  │
│   ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │
│   │ Planning API│  │   Chat API  │  │  Auth API │  │
│   └─────────────┘  └─────────────┘  └───────────┘  │
└────────────────────────┬───────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────┐
│               Service Layer (Business Logic)       │
│                                                    │
│   ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │
│   │ User Service│  │Notes Service│  │Task Service│  │
│   └─────────────┘  └─────────────┘  └───────────┘  │
│   ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │
│   │ Plan Service│  │ Chat Service│  │Auth Service│  │
│   └─────────────┘  └─────────────┘  └───────────┘  │
│                 ┌───────────────┐                  │
│                 │Task Indexing  │                  │
│                 │   Service     │                  │
│                 └───────────────┘                  │
└────────────────────────┬───────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────┐
│                  Repository Layer                  │
│                                                    │
│   ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │
│   │ User Repo   │  │ Notes Repo  │  │ Task Repo │  │
│   └─────────────┘  └─────────────┘  └───────────┘  │
│   ┌─────────────┐  ┌─────────────┐                 │
│   │ Plan Repo   │  │  Chat Repo  │                 │
│   └─────────────┘  └─────────────┘                 │
└────────────────────────┬───────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────┐
│                    Data Layer                      │
│                                                    │
│   ┌─────────────────────┐  ┌────────────────────┐  │
│   │ Supabase PostgreSQL │  │   Vector Database  │  │
│   └─────────────────────┘  └────────────────────┘  │
└────────────────────────────────────────────────────┘
```

### AI System Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Agent Manager                      │
│                                                     │
│   ┌──────────────────────────────────────────────┐  │
│   │              CrewAI Orchestrator             │  │
│   └──────────────────────────────────────────────┘  │
└───────────┬───────────────────────────┬─────────────┘
            │                           │
            ▼                           ▼
┌───────────────────────┐   ┌───────────────────────────┐
│    Agent Registry     │   │    Context Provider       │
│                       │   │                           │
│  ┌─────────────────┐  │   │  ┌─────────────────────┐  │
│  │  Agent Catalog  │  │   │  │ User Profile Context│  │
│  └─────────────────┘  │   │  └─────────────────────┘  │
│  ┌─────────────────┐  │   │  ┌─────────────────────┐  │
│  │ Agent Config    │  │   │  │ Document Context    │  │
│  └─────────────────┘  │   │  └─────────────────────┘  │
└───────────┬───────────┘   │  ┌─────────────────────┐  │
            │               │  │ Task Context        │  │
            │               │  └─────────────────────┘  │
            │               └───────────────────────────┘
            │                            ▲
            │                            │
            ▼                            │
┌───────────────────────┐                │
│   LLM Providers       │                │
│                       │                │
│  ┌─────────────────┐  │                │
│  │  OpenAI Client  │  │                │
│  └─────────────────┘  │                │
│  ┌─────────────────┐  │                │
│  │ Anthropic Client│  │                │
│  └─────────────────┘  │                │
│  ┌─────────────────┐  │   ┌────────────┴──────────┐
│  │   xAI Client    │──┼───│    RAG System         │
│  └─────────────────┘  │   │                       │
│  ┌─────────────────┐  │   └───────────────────────┘
│  │ OpenRouter Client│  │
│  └─────────────────┘  │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ External Integrations │
│                       │
│  ┌─────────────────┐  │
│  │ Email Service   │  │
│  └─────────────────┘  │
│  ┌─────────────────┐  │
│  │    Composio     │  │
│  └─────────────────┘  │
│  ┌─────────────────┐  │
│  │ Direct API      │  │
│  │ Integrations    │  │
│  └─────────────────┘  │
└───────────────────────┘
```

## Directory Structure

```
backend/
├── app/
│   ├── api/
│   │   ├── endpoints/         # API route handlers
│   │   │   ├── auth.py
│   │   │   ├── users.py
│   │   │   ├── notes.py
│   │   │   ├── tasks.py
│   │   │   ├── planning.py
│   │   │   └── chat.py
│   │   ├── dependencies.py     # Dependency injection
│   │   └── router.py           # API router configuration
│   ├── core/
│   │   ├── config.py           # Application configuration
│   │   ├── security.py         # Security utilities
│   │   ├── exceptions.py       # Custom exceptions
│   │   └── logger.py           # Logging configuration
│   ├── db/
│   │   ├── base.py             # Base database models
│   │   ├── session.py          # Database session handling
│   │   └── repositories/       # Database access layers
│   ├── models/
│   │   ├── user.py             # User models
│   │   ├── note.py             # Note models
│   │   ├── task.py             # Task models
│   │   ├── planning.py         # Planning models
│   │   └── chat.py             # Chat models
│   ├── schemas/                # Pydantic schemas for validation
│   ├── services/               # Business logic services
│   │   ├── user_service.py
│   │   ├── note_service.py
│   │   ├── task_service.py
│   │   ├── planning_service.py
│   │   └── chat_service.py
│   ├── ai/
│   │   ├── agents/             # AI agent definitions
│   │   ├── crews/              # CrewAI configurations
│   │   ├── providers/          # LLM provider integrations
│   │   ├── rag/                # RAG system components
│   │   └── utils/              # AI utilities
│   └── utils/                  # General utilities
├── alembic/                    # Database migrations
├── tests/                      # Test suite
│   ├── conftest.py             # Test configuration
│   ├── api/                    # API tests
│   ├── services/               # Service tests
│   └── ai/                     # AI system tests
├── pyproject.toml              # Project dependencies (PDM)
├── .env.example                # Example environment variables
├── Dockerfile                  # Docker configuration
└── main.py                     # Application entry point
```

## Database Schema

### Core Tables

```
Users
- id: UUID (PK)
- email: String (unique)
- hashed_password: String
- full_name: String
- created_at: Timestamp
- updated_at: Timestamp
- is_active: Boolean
- is_superuser: Boolean
- preferences: JSONB

Notes
- id: UUID (PK)
- title: String
- content: JSONB (BlockNotes format)
- user_id: UUID (FK to Users)
- parent_id: UUID (self-reference, optional)
- created_at: Timestamp
- updated_at: Timestamp
- is_archived: Boolean
- is_pinned: Boolean
- metadata: JSONB

Tasks
- id: UUID (PK)
- title: String
- description: Text
- user_id: UUID (FK to Users)
- status: Enum
- priority: Enum
- due_date: Date
- due_time: Time
- created_at: Timestamp
- updated_at: Timestamp
- parent_id: UUID (self-reference, optional)
- source: Enum
- source_reference: String
- metadata: JSONB

Goals
- id: UUID (PK)
- title: String
- description: Text
- user_id: UUID (FK to Users)
- timeframe: Enum
- parent_goal_id: UUID (FK to Goals, optional)
- status: Enum
- progress: Integer
- priority: Enum
- due_date: Date
- created_at: Timestamp
- updated_at: Timestamp
- metadata: JSONB

Reflections
- id: UUID (PK)
- content: Text
- user_id: UUID (FK to Users)
- reflection_type: Enum
- created_at: Timestamp
- metadata: JSONB

Conversations
- id: UUID (PK)
- title: String
- user_id: UUID (FK to Users)
- created_at: Timestamp
- updated_at: Timestamp
- metadata: JSONB

Messages
- id: UUID (PK)
- conversation_id: UUID (FK to Conversations)
- content: Text
- role: Enum
- agent_id: String (optional)
- created_at: Timestamp
- metadata: JSONB
```

## API Endpoints

### Authentication

- `POST /api/auth/login`: User login
- `POST /api/auth/register`: User registration
- `POST /api/auth/refresh`: Refresh authentication token
- `POST /api/auth/logout`: User logout

### Users

- `GET /api/users/me`: Get current user profile
- `PUT /api/users/me`: Update user profile
- `GET /api/users/preferences`: Get user preferences
- `PUT /api/users/preferences`: Update user preferences

### Notes

- `GET /api/notes`: List user notes
- `POST /api/notes`: Create a new note
- `GET /api/notes/{note_id}`: Get a specific note
- `PUT /api/notes/{note_id}`: Update a note
- `DELETE /api/notes/{note_id}`: Delete a note
- `GET /api/notes/{note_id}/tasks`: Get tasks from a note

### Tasks

- `GET /api/tasks`: List user tasks
- `POST /api/tasks`: Create a new task
- `GET /api/tasks/{task_id}`: Get a specific task
- `PUT /api/tasks/{task_id}`: Update a task
- `DELETE /api/tasks/{task_id}`: Delete a task
- `PUT /api/tasks/{task_id}/status`: Update task status
- `GET /api/tasks/external`: List tasks from external sources
- `POST /api/tasks/sync`: Manually trigger sync with external systems

### Planning

- `GET /api/goals`: List user goals
- `POST /api/goals`: Create a new goal
- `GET /api/goals/{goal_id}`: Get a specific goal
- `PUT /api/goals/{goal_id}`: Update a goal
- `DELETE /api/goals/{goal_id}`: Delete a goal
- `GET /api/reflections`: List user reflections
- `POST /api/reflections`: Create a new reflection

### Chat

- `GET /api/conversations`: List user conversations
- `POST /api/conversations`: Create a new conversation
- `GET /api/conversations/{conversation_id}`: Get a specific conversation
- `DELETE /api/conversations/{conversation_id}`: Delete a conversation
- `GET /api/conversations/{conversation_id}/messages`: Get messages in a conversation
- `POST /api/conversations/{conversation_id}/messages`: Send a new message
- `WS /api/chat/{conversation_id}`: WebSocket for real-time chat

### AI Services

- `POST /api/ai/agents`: List available AI agents
- `POST /api/ai/transcribe`: Transcribe image to text
- `POST /api/ai/rag/query`: Query the RAG system
- `POST /api/ai/documents/process`: Process a document for RAG

### External Integrations

- `GET /api/external/providers`: List available external providers
- `POST /api/external/connect/{provider}`: Connect to external provider
- `DELETE /api/external/disconnect/{provider}`: Disconnect from provider
- `GET /api/external/status/{provider}`: Get connection status
- `POST /api/external/sync/{provider}`: Force sync with provider

## Authentication and Security

### Authentication Flow

1. User registers or logs in through Supabase Auth
2. Backend receives and validates the JWT token
3. User identity is extracted from the validated token
4. Permissions are checked based on user identity and requested resource

### Security Measures

- **HTTPS**: All API communications are encrypted
- **JWT Validation**: Tokens are cryptographically verified
- **CORS Protection**: Configured to allow only permitted origins
- **Rate Limiting**: Prevents abuse and DoS attacks
- **Input Validation**: All inputs validated through Pydantic schemas
- **SQL Injection Protection**: Parameterized queries via SQLAlchemy
- **Dependency Scanning**: Regular scanning for vulnerable dependencies

## Deployment Architecture

### Development

- Local development environment with Docker
- PDM for dependency management
- Supabase local development

### Staging

- Containerized deployment on cloud platform
- CI/CD pipeline for automated testing and deployment
- Staging database with anonymized data

### Production

- Horizontally scalable API servers
- Load balancing for distributed traffic
- Database connection pooling
- Caching layer for frequently accessed data
- Managed Supabase instance
- Monitoring and alerting

## Testing Strategy

- **Unit Tests**: Testing individual functions and methods
- **Integration Tests**: Testing interactions between components
- **API Tests**: Testing API endpoints
- **Load Tests**: Testing system performance under load
- **Security Tests**: Testing for common vulnerabilities

## Future Considerations

- **GraphQL API**: Alternative to REST for more flexible data fetching
- **Microservices**: Breaking down into domain-specific services as the application grows
- **Kubernetes**: For more advanced container orchestration
- **Event-Driven Architecture**: Using message queues for async processing
- **Custom Vector Database**: For improved RAG performance and cost efficiency

## Service Implementation

### Notes Service

The Notes Service manages the creation, retrieval, updating, and deletion of notes:

- **CRUD Operations**: Basic note management functions
- **Content Parsing**: Processing and validating BlockNote JSON content
- **Tag Management**: Handling note tagging and categorization
- **Block Analysis**: Processing different block types in notes
- **Task Block Processing**: Identifying and extracting task blocks for indexing
- **Version Management**: Tracking note history and versions

### Task Service

The Task Service manages task blocks within notes and provides aggregated task views:

- **Task Indexing**: Maintains an index of all task blocks across notes
- **Task Aggregation**: Retrieves and aggregates tasks from multiple notes
- **State Synchronization**: Ensures bidirectional syncing between task views and notes
- **Filtering Capability**: Provides various filters for task views (due date, status, etc.)
- **Task Block CRUD**: Manages task block operations within note content
- **Parent Note Reference**: Maintains references to parent notes for each task
- **External Task Handling**: Special handling for tasks from external sources

### Task Indexing Service

A specialized service that handles the indexing and retrieval of tasks from notes:

- **Content Scanning**: Scans note content for task blocks
- **Index Maintenance**: Maintains a separate index of tasks for efficient querying
- **Real-time Updates**: Updates the index when notes or tasks change
- **Context Preservation**: Maintains references to source notes for each task
- **Query Optimization**: Provides optimized query patterns for task retrieval
- **Bi-directional Updates**: Propagates changes from task views back to source notes

### External Integration Services

#### Email Integration Service

Handles the scanning of emails and creation of tasks from email content:

- **Email API Integration**: Connects to email providers via secure OAuth
- **Email Scanning Agent**: AI agent that analyzes emails for task-like content
- **Periodic Scanning**: Scheduled scanning of inbox for new task-relevant emails
- **Email Reference Tracking**: Maintains references to source emails for tasks
- **Reply Processing**: Handles email replies that relate to existing tasks

#### External Application Service

Manages integration with external task management systems:

- **Composio Integration**: Uses Composio (https://composio.dev/) via CrewAI for API access
- **ClickUp Sync**: Bidirectional synchronization with ClickUp tasks
- **Task Transformation**: Converts external task formats to Consilium task blocks
- **Status Propagation**: Ensures task status changes propagate to external systems
- **Cleanup Service**: Removes completed external tasks from notes
- **Extensible Framework**: Designed to easily add new integrations

## Data Models

### Note Model

```python
class Note(BaseModel):
    id: UUID
    title: str
    content: Dict[str, Any]  # BlockNote JSON structure
    created_at: datetime
    updated_at: datetime
    user_id: UUID
    tags: List[UUID] = []
    linked_notes: List[UUID] = []
    is_pinned: bool = False
    is_archived: bool = False
    metadata: Dict[str, Any] = {}

    class Config:
        orm_mode = True
```

### Task Index Model

```python
class TaskIndex(BaseModel):
    id: UUID
    note_id: UUID  # Reference to parent note
    block_id: str  # Reference to specific block in note
    title: str
    description: str = ""
    status: TaskStatus
    due_date: Optional[date] = None
    due_time: Optional[time] = None
    priority: TaskPriority
    created_at: datetime
    updated_at: datetime
    user_id: UUID
    parent_task_id: Optional[UUID] = None
    source: TaskSource = TaskSource.DIRECT
    source_reference: Optional[str] = None  # Reference ID in source system
    size: Optional[float] = None  # Numeric value for effort estimation
    metadata: Dict[str, Any] = {}

    class Config:
        orm_mode = True

class TaskStatus(str, Enum):
    NOT_STARTED = "not_started"
    IN_PROGRESS = "in_progress"
    COMPLETED = "completed"
    DEFERRED = "deferred"

class TaskPriority(str, Enum):
    HIGH = "high"
    MEDIUM = "medium"
    LOW = "low"

class TaskSource(str, Enum):
    DIRECT = "direct"
    NOTE = "note"
    PLANNING = "planning"
    CHAT = "chat"
    EMAIL = "email"
    CLICKUP = "clickup"
    EXTERNAL = "external"
```

### External Reference Model

```python
class ExternalTaskReference(BaseModel):
    id: UUID
    task_id: UUID  # Reference to TaskIndex
    system: str  # "email", "clickup", etc.
    external_id: str  # ID in external system
    system_url: Optional[str] = None  # URL to external system if applicable
    last_synced: datetime
    sync_status: SyncStatus
    metadata: Dict[str, Any] = {}

    class Config:
        orm_mode = True

class SyncStatus(str, Enum):
    SYNCED = "synced"
    PENDING = "pending"
    ERROR = "error"
``` 