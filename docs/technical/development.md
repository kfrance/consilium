# Development Setup

## Overview

This document outlines the development environment setup for the Consilium project. The development stack is designed to provide a consistent and efficient workflow for both frontend and backend development.

## Prerequisites

Before starting, ensure you have the following installed:

- **Git**: Version control
- **Node.js**: v18+ for frontend development
- **Python**: 3.10+ for backend development
- **PDM**: Python dependency management
- **Docker**: For containerized development environment (optional)
- **VSCode**: Recommended editor (with suggested extensions)

## Repository Structure

```
consilium/
├── frontend/           # React frontend application
├── backend/            # Python FastAPI backend
├── docs/               # Project documentation
├── .github/            # GitHub Actions workflows
├── docker/             # Docker configuration files
├── scripts/            # Development and utility scripts
└── README.md           # Project README
```

## Frontend Setup

### Installation

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install
```

### Configuration

Create a `.env.local` file in the `frontend` directory:

```
VITE_API_URL=http://localhost:8000
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

### Development Server

```bash
# Start development server
npm run dev

# Run tests
npm test

# Build for production
npm run build
```

### Key Frontend Dependencies

- **React**: UI library
- **TypeScript**: Type safety
- **Vite**: Build tool
- **Tailwind CSS**: Utility-first CSS
- **React Router**: Routing
- **Redux Toolkit**: State management
- **React Query**: Data fetching
- **DexieJS**: IndexedDB wrapper
- **reachat**: Chat UI components
- **BlockNotes**: Rich text editor

## Backend Setup

### Installation

```bash
# Navigate to backend directory
cd backend

# Install dependencies using PDM
pdm install
```

### Configuration

Create a `.env` file in the `backend` directory:

```
# Database
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/consilium

# Authentication
SECRET_KEY=your_secret_key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Services
OPENAI_API_KEY=your_openai_api_key
ANTHROPIC_API_KEY=your_anthropic_api_key
XAI_API_KEY=your_xai_api_key
OPENROUTER_API_KEY=your_openrouter_api_key

# Supabase
SUPABASE_URL=your_supabase_url
SUPABASE_SERVICE_KEY=your_supabase_service_key

# Vector DB
VECTOR_DB_URL=your_vector_db_url
VECTOR_DB_API_KEY=your_vector_db_api_key
```

### Development Server

```bash
# Start development server
pdm run dev

# Run tests
pdm run test

# Run linting
pdm run lint
```

### Key Backend Dependencies

- **FastAPI**: Web framework
- **Pydantic**: Data validation
- **SQLAlchemy**: ORM
- **Alembic**: Database migrations
- **CrewAI**: AI orchestration
- **LangChain**: LLM integration framework
- **Supabase**: Authentication and database
- **Pytest**: Testing framework

## Database Setup

### Supabase Local Development

For local development, you can use Supabase's local development environment:

```bash
# Install Supabase CLI
npm install -g supabase

# Start local Supabase
supabase start

# Get local connection URL
supabase status
```

### Migrations

```bash
# Generate migration
pdm run alembic revision --autogenerate -m "description"

# Run migrations
pdm run alembic upgrade head
```

## AI Development Tools

### Setting Up LLM Providers

1. **OpenAI**:
   - Register at platform.openai.com
   - Create an API key
   - Add to your `.env` file

2. **Anthropic**:
   - Register at console.anthropic.com
   - Create an API key
   - Add to your `.env` file

3. **Local Models** (optional):
   - Install compatible local model server (e.g., llama.cpp)
   - Configure API endpoint in settings

### CrewAI Setup

CrewAI is used for agent orchestration. Configure your agent definitions in:

```
backend/app/ai/agents/
```

## Docker Development Environment

For a containerized development experience:

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down
```

## VSCode Configuration

Recommended extensions and settings for VSCode:

### Recommended Extensions

- ESLint
- Prettier
- Python
- Tailwind CSS IntelliSense
- Error Lens
- Git Lens
- Docker

### Workspace Settings

Create a `.vscode/settings.json` file:

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": false,
  "python.linting.ruffEnabled": true,
  "python.formatting.provider": "black",
  "python.formatting.blackArgs": ["--line-length", "88"],
  "[python]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "ms-python.black-formatter"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "typescript.tsdk": "frontend/node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true
}
```

## Testing

### Frontend Testing

```bash
# Run unit tests
cd frontend
npm test

# Run E2E tests
npm run test:e2e
```

### Backend Testing

```bash
# Run all tests
cd backend
pdm run test

# Run specific test file
pdm run test tests/api/test_notes.py

# Run with coverage
pdm run test-cov
```

## Common Development Tasks

### Adding a New Frontend Feature

1. Create feature components in `frontend/src/components/`
2. Add new routes in `frontend/src/App.tsx` if needed
3. Create or update Redux slices in `frontend/src/store/`
4. Add API calls in `frontend/src/services/api/`
5. Add tests in `frontend/src/tests/`

### Adding a New Backend Endpoint

1. Create or update schemas in `backend/app/schemas/`
2. Add endpoint to appropriate router in `backend/app/api/endpoints/`
3. Implement service logic in `backend/app/services/`
4. Add repository methods in `backend/app/db/repositories/`
5. Add tests in `backend/tests/`

### Adding a New Agent

1. Define new agent class in `backend/app/ai/agents/`
2. Register agent in the agent registry
3. Update agent selection logic if needed
4. Add agent-specific prompt templates
5. Add tests for agent behavior

## Troubleshooting

### Common Issues

- **Supabase Connection Issues**: Check if local Supabase is running and connection strings are correct
- **API Key Issues**: Verify all API keys are correctly set in environment variables
- **Dependency Conflicts**: Run `pdm update` or `npm update` to resolve conflicts
- **Database Migration Errors**: Check migration history and fix conflicts

### Getting Help

- Check the #developers channel in team communication
- Refer to documentation in the `docs` directory
- Open an issue on GitHub

## Continuous Integration

GitHub Actions workflows are set up for:

- Code linting and formatting
- Unit and integration tests
- Build verification
- Dependency scanning

## Release Process

1. Create a new feature branch
2. Implement changes and tests
3. Submit a pull request to the `develop` branch
4. After review and approval, merge to `develop`
5. Periodically, `develop` is merged to `main` for releases

## Environment Management

- **Development**: Local environment for individual development
- **Staging**: Shared environment for integration testing
- **Production**: Live environment for end users

Each environment has its own configuration and database. 