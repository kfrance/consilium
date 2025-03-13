# Consilium

![Consilium Logo](docs/assets/logo.png)

Consilium (Latin for "council") is a multi-agent AI system designed to help you achieve goals, plan your time, store notes, and track tasks. Think of it as having your own council of AI assistants, each specialized in different aspects of personal productivity and knowledge management.

## Key Features

- **AI-Powered Planning** - Set and manage goals across different time horizons, from 5-year plans to daily tasks
- **Smart Note Taking** - Capture and organize thoughts with AI-enhanced organization
- **Task Management** - Collect and organize todos from multiple sources
- **Document Intelligence** - Ask questions about your documents and notes using RAG technology
- **Multi-Agent System** - Access specialized AI agents for different needs
- **Multi-Provider Support** - Use LLMs from OpenAI, Anthropic, xAI, and more
- **Offline First** - Work with or without an internet connection
- **External Integrations** - Connect with ClickUp, email, calendar, and more

## Project Status

🚧 **Under Development** 🚧

Consilium is currently in early development. This repository contains the design documentation for the project.

## Documentation

For detailed information about the project, check out the [documentation](docs/README.md).

### Architecture

- [System Overview](docs/architecture/system-overview.md)
- [Agent System](docs/architecture/agent-system.md)

### Features

- [Notes](docs/features/notes.md)
- [Planning & Goals](docs/features/planning.md)
- [Tasks](docs/features/tasks.md)
- [RAG (Document Intelligence)](docs/features/rag.md)

### Technical Details

- [Frontend](docs/technical/frontend.md)
- [Backend](docs/technical/backend.md)
- [Storage](docs/technical/storage.md)
- [AI Integration](docs/technical/ai-integration.md)
- [Development Setup](docs/technical/development.md)
- [Deployment](docs/technical/deployment.md)

### Integrations

- [External Services](docs/integrations/external-services.md)

## Technology Stack

### Frontend
- React
- TypeScript
- Redux Toolkit
- DexieJS (IndexedDB)
- BlockNotes (Rich Text Editor)
- Reachat (Chat UI)
- Progressive Web App (PWA)

### Backend
- Python
- FastAPI
- PDM (Python Dependency Manager)
- CrewAI (AI Orchestration)
- Supabase (Auth & Database)

## Getting Started

See the [Development Setup](docs/technical/development.md) guide to get started with local development.

## Vision

Consilium aims to be more than just another productivity tool. By combining the power of modern AI with a thoughtfully designed user experience, we're creating a system that adapts to your workflow rather than forcing you to adapt to it.

Our core principles:

1. **AI as a Council, Not a Commander** - Multiple specialized agents working together to assist you
2. **Your Data, Your Control** - Privacy-first approach with local-first storage when possible
3. **Seamless Integration** - Connect with the tools you already use
4. **Adaptive Intelligence** - Learn from your patterns to provide better assistance over time
5. **Human-Centered Design** - Technology that adapts to humans, not the other way around

## License

[MIT License](LICENSE)

## Acknowledgements

- [BlockNotes](https://github.com/TypeCellOS/BlockNote) for the rich text editor
- [CrewAI](https://github.com/joaomdmoura/crewAI) for AI orchestration
- [Supabase](https://supabase.com/) for authentication and database
- All the open source libraries that make this project possible 