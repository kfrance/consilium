# AI Integration

## Overview

Consilium's AI integration is built around a multi-agent system that leverages multiple LLM providers and specialized AI capabilities. The system uses CrewAI for orchestration, allowing different agents to collaborate on tasks while maintaining a cohesive user experience.

## AI Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        User Interface                               │
│                                                                     │
│  ┌───────────────────┐   ┌───────────────────┐   ┌───────────────┐  │
│  │    Chat Interface │   │  Planning UI      │   │ Note Taking UI│  │
│  └─────────┬─────────┘   └──────────┬────────┘   └───────┬───────┘  │
└────────────┼──────────────────────────────────────────────┼─────────┘
             │                        │                     │
             ▼                        ▼                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     API Layer (FastAPI)                             │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        CrewAI Orchestration                         │
│                                                                     │
│  ┌───────────────────┐   ┌───────────────────┐   ┌───────────────┐  │
│  │  Agent Selection  │   │  Task Planning    │   │ Context Mgmt  │  │
│  └───────────────────┘   └───────────────────┘   └───────────────┘  │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        Agent System                                 │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │  Core Agents │  │Feature Agents│  │Utility Agents│  │RAG System│ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └────┬─────┘ │
└─────────┼─────────────────┼─────────────────┼──────────────┬────────┘
          │                 │                 │              │
          ▼                 ▼                 ▼              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     LLM Provider Interface                          │
│                                                                     │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────────┐ │
│  │   OpenAI   │  │  Anthropic │  │    xAI     │  │  OpenRouter    │ │
│  └────────────┘  └────────────┘  └────────────┘  └────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

## Multi-Provider Strategy

Consilium is designed to work with multiple LLM providers to enable flexibility, optimize costs, and ensure reliability.

### Supported Providers

- **OpenAI**: GPT-3.5, GPT-4, and future models
- **Anthropic**: Claude models for longer context and nuanced reasoning
- **xAI**: Grok models for creative problem-solving
- **OpenRouter**: Access to various models through a single API
- **Local Models**: Support for running local models for privacy-sensitive users (future)

### Provider Selection Logic

The system selects the appropriate provider based on:

1. **Task Requirements**: Matching task complexity to model capabilities
2. **User Preferences**: User-specified default providers
3. **Cost Optimization**: Balancing performance needs with cost
4. **Availability**: Fallback options if a provider is unavailable
5. **Context Size**: Selecting models with appropriate context windows

### Provider Integration

```python
# Example provider interface
class LLMProvider(Protocol):
    """Interface for LLM providers."""
    
    def get_completion(self, prompt: str, **kwargs) -> str:
        """Get a completion from the LLM."""
        ...
    
    def get_chat_completion(self, messages: list[dict], **kwargs) -> str:
        """Get a chat completion from the LLM."""
        ...
    
    def get_embedding(self, text: str) -> list[float]:
        """Get an embedding for the given text."""
        ...

# Provider implementation example
class AnthropicProvider(LLMProvider):
    """Anthropic Claude provider implementation."""
    
    def __init__(self, api_key: str, model: str = "claude-2"):
        self.client = anthropic.Client(api_key=api_key)
        self.model = model
    
    def get_chat_completion(self, messages: list[dict], **kwargs) -> str:
        # Convert messages to Anthropic format
        anthropic_messages = self._convert_messages(messages)
        
        response = self.client.messages.create(
            model=self.model,
            messages=anthropic_messages,
            **kwargs
        )
        
        return response.content[0].text
    
    # ... other methods
```

## CrewAI Integration

CrewAI is used for orchestrating multiple AI agents, allowing them to collaborate on complex tasks.

### Core Components

- **Crew**: A collection of agents that work together on tasks
- **Agent**: An individual AI agent with specific capabilities
- **Task**: A unit of work assigned to an agent or crew
- **Process**: The workflow that coordinates agent activities

### Agent Configuration

```python
# Example agent configuration
from crewai import Agent, Task, Crew

# Define agents
planner_agent = Agent(
    name="Planner",
    goal="Create effective plans based on user goals and preferences",
    backstory="You are an expert planner with deep knowledge of productivity and goal-setting methods.",
    verbose=True,
    llm=anthropic_provider.get_llm(),  # Use Anthropic for this agent
)

note_agent = Agent(
    name="NoteTaker",
    goal="Help users capture and organize information effectively",
    backstory="You are an expert at knowledge management and note organization.",
    verbose=True,
    llm=openai_provider.get_llm(),  # Use OpenAI for this agent
)

# Create a crew with these agents
planning_crew = Crew(
    agents=[planner_agent, note_agent],
    tasks=[
        Task(
            description="Create a weekly plan based on monthly goals",
            expected_output="Detailed weekly plan with tasks and time blocks",
            agent=planner_agent,
        ),
        Task(
            description="Take meeting notes and extract action items",
            expected_output="Structured notes with tagged action items",
            agent=note_agent,
        )
    ],
    verbose=True
)

# Execute the crew's tasks
result = planning_crew.kickoff()
```

## Agent System

The agent system is composed of multiple specialized agents that cover different aspects of the application.

### Core Agents

- **Planner Agent**: Creates and manages plans at different time horizons
- **Coordinator Agent**: Directs user requests to appropriate agents
- **Memory Agent**: Maintains user context and memory across sessions

### Feature Agents

- **Note Taker Agent**: Specializes in note organization and knowledge management
- **Todo Manager Agent**: Handles task creation, organization, and tracking
- **Reflection Agent**: Guides the user through reflection processes

### Utility Agents

- **RAG Agent**: Retrieves relevant information from user documents
- **Transcriber Agent**: Converts images of text to digital text
- **Integrator Agent**: Connects with external services

### Agent Selection Logic

1. **User Intent Analysis**: Analyze user message to determine intent
2. **Agent Matching**: Match intent to most suitable agent(s)
3. **Context Assembly**: Gather relevant context for selected agent(s)
4. **Response Generation**: Generate response using selected agent(s)
5. **Response Integration**: Combine responses if multiple agents involved

## RAG (Retrieval-Augmented Generation) System

The RAG system enhances AI capabilities by providing relevant context from the user's documents.

### Implementation Components

- **Document Processor**: Extracts text from various document formats
- **Chunker**: Divides documents into manageable chunks
- **Embedder**: Generates vector embeddings for text chunks
- **Vector Store**: Stores and retrieves embeddings
- **Retriever**: Finds relevant chunks based on user queries
- **Context Assembler**: Combines retrieved chunks into coherent context
- **Response Generator**: Generates responses based on query and context

### RAG Process Flow

```python
# Example RAG process
class RAGSystem:
    def __init__(
        self, 
        embedder: Embedder, 
        vector_store: VectorStore,
        llm_provider: LLMProvider
    ):
        self.embedder = embedder
        self.vector_store = vector_store
        self.llm_provider = llm_provider
    
    def process_document(self, document: Document) -> None:
        """Process a document and store its embeddings."""
        # Extract text from document
        text = document.get_text()
        
        # Split text into chunks
        chunks = self.chunker.split(text)
        
        # Generate embeddings for chunks
        for chunk in chunks:
            embedding = self.embedder.embed(chunk.text)
            
            # Store embedding in vector store
            self.vector_store.add(
                embedding=embedding,
                metadata={
                    "document_id": document.id,
                    "chunk_id": chunk.id,
                    "text": chunk.text,
                    "source": document.source,
                    "created_at": document.created_at
                }
            )
    
    def query(self, query: str, filters: dict = None) -> str:
        """Query the RAG system."""
        # Generate embedding for query
        query_embedding = self.embedder.embed(query)
        
        # Retrieve relevant chunks
        results = self.vector_store.search(
            embedding=query_embedding,
            filters=filters,
            limit=5
        )
        
        # Assemble context from retrieved chunks
        context = self.context_assembler.assemble(results)
        
        # Generate response using LLM with context
        response = self.llm_provider.get_completion(
            prompt=f"Context: {context}\n\nQuestion: {query}\n\nAnswer:",
            max_tokens=500
        )
        
        return response
```

## LLM Request Management

### Prompt Engineering

- **Templating System**: Reusable prompt templates for consistency
- **Dynamic Insertion**: Inject context and parameters into prompts
- **Few-Shot Examples**: Include examples for complex tasks
- **Chained Prompting**: Break complex tasks into steps

### Token Management

- **Token Estimation**: Predict token usage before API calls
- **Context Window Management**: Handle large contexts efficiently
- **Token Budget Allocation**: Distribute token budget across tasks
- **Cost Tracking**: Monitor and report token usage

### Error Handling

- **Retry Logic**: Retry failed requests with exponential backoff
- **Fallback Strategy**: Switch to alternative providers when necessary
- **Graceful Degradation**: Provide limited functionality when AI services unavailable
- **User Feedback**: Clear error messages for API issues

## Specialized AI Features

### Image-to-Text Transcription

For converting handwritten notes:

- **Image Preprocessing**: Enhance image quality for better OCR
- **OCR Processing**: Extract text from images
- **Post-processing**: Clean and format extracted text
- **Integration**: Insert transcribed text into notes

### Intelligent Task Extraction

For identifying tasks in text:

- **NLP Analysis**: Identify action items in text
- **Intent Classification**: Determine if text represents a task
- **Entity Extraction**: Extract due dates, priorities, assignees
- **Automation**: Generate task entries from extracted information

## Security and Privacy

### Data Minimization

- **Selective Context**: Only send necessary context to LLM providers
- **Local Processing**: Process sensitive data locally when possible
- **Anonymization**: Remove identifying information when not needed

### User Controls

- **Provider Selection**: Allow users to choose preferred providers
- **Data Sharing Controls**: Granular control over what data is shared with AI
- **Opt-out Options**: Allow users to opt out of specific AI features

### Compliance Measures

- **Audit Logging**: Track AI interactions for accountability
- **Data Retention**: Clear policies on data sent to AI providers
- **Transparency**: Clear documentation of AI capabilities and limitations

## Future AI Enhancements

- **Agent Memory**: Improved long-term memory across sessions
- **Multi-modal Models**: Support for image and audio input/output
- **Agent Personalization**: Adapt agent behavior to user preferences
- **Fine-tuned Models**: Custom-tuned models for specific domains
- **On-device Models**: More capabilities running locally for privacy
- **Collaborative AI**: Agents that can collaborate with multiple users 