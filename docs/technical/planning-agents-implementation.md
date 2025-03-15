# Planning Agents Implementation

This document provides technical details for implementing the multi-agent planning system in Consilium.

## Architecture Overview

The planning system is built using CrewAI as the agent orchestration framework, with multiple specialized agents working together under the coordination of a central Planner Agent.

```
┌─────────────────────────────────────────────────────────┐
│                  Planning System                         │
│                                                         │
│  ┌─────────────┐           ┌─────────────────────────┐  │
│  │             │◄─────────►│   Domain-Specific       │  │
│  │             │           │   Agents                │  │
│  │             │           │                         │  │
│  │   Planner   │           │  ┌─────────────────┐   │  │
│  │   Agent     │◄─────────►│  │  Health Agent   │   │  │
│  │             │           │  └─────────────────┘   │  │
│  │             │           │  ┌─────────────────┐   │  │
│  │             │◄─────────►│  │ Spiritual Agent │   │  │
│  │             │           │  └─────────────────┘   │  │
│  │             │           │  ┌─────────────────┐   │  │
│  │             │◄─────────►│  │   Work Agent    │   │  │
│  └─────────────┘           │  └─────────────────┘   │  │
│        ▲                   └─────────────────────────┘  │
│        │                                                │
│        ▼                                                │
│  ┌─────────────┐           ┌─────────────────────────┐  │
│  │             │           │                         │  │
│  │    User     │◄─────────►│       Memory            │  │
│  │ Interaction │           │       System            │  │
│  │             │           │                         │  │
│  └─────────────┘           └─────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Agent Implementation

### Planner Agent

The Planner Agent is implemented as a CrewAI agent with specialized prompts and tools:

```python
from crewai import Agent
from crewai.tools import Tool

# Create tools for the Planner Agent
calendar_tool = Tool(
    name="calendar_access",
    description="Access the user's calendar events and availability",
    func=calendar_access_function
)

goal_access_tool = Tool(
    name="goal_access",
    description="Access the user's goals at different time horizons",
    func=goal_access_function
)

# Create the Planner Agent
planner_agent = Agent(
    name="PlannerAgent",
    goal="Create effective, balanced plans that align with the user's goals across different time horizons",
    backstory="""You are an expert planning assistant that helps the user create plans
    from daily activities to long-term goals. You coordinate with specialized domain agents
    to create comprehensive, personalized plans that align the user's daily activities with
    their aspirations.""",
    verbose=True,
    allow_delegation=True,
    tools=[calendar_tool, goal_access_tool]
)
```

### Domain-Specific Agents

Each domain agent is implemented with specialized knowledge and access permissions:

```python
# Health Agent
health_agent = Agent(
    name="HealthAgent",
    goal="Provide optimal health and fitness recommendations aligned with the user's goals and current state",
    backstory="""You are a health and fitness expert who helps the user maintain and improve
    their physical wellbeing. You have access to their fitness data, exercise history, and health-related goals.""",
    verbose=True,
    tools=[
        Tool(name="exercise_history", func=access_exercise_history),
        Tool(name="fitness_goals", func=access_fitness_goals),
        Tool(name="health_metrics", func=access_health_metrics)
    ]
)

# Spiritual Agent
spiritual_agent = Agent(
    name="SpiritualAgent",
    goal="Support the user's spiritual growth and relationships aligned with their values",
    backstory="""You are a spiritual advisor who helps the user nurture their spiritual life
    and important relationships. You have access to their spiritual practices, relationship goals,
    and reflection history.""",
    verbose=True,
    tools=[
        Tool(name="spiritual_goals", func=access_spiritual_goals),
        Tool(name="reflection_history", func=access_reflection_history),
        Tool(name="relationship_data", func=access_relationship_data)
    ]
)

# Work Agent
work_agent = Agent(
    name="WorkAgent",
    goal="Help the user optimize productivity and achieve professional goals",
    backstory="""You are a productivity and career expert who helps the user excel in their
    professional life. You have access to their work projects, career goals, and productivity patterns.""",
    verbose=True,
    tools=[
        Tool(name="career_goals", func=access_career_goals),
        Tool(name="work_projects", func=access_work_projects),
        Tool(name="productivity_data", func=access_productivity_data)
    ]
)
```

## Agent Discovery Mechanism

The discovery mechanism is implemented through a combination of CrewAI's delegation capabilities and custom message passing:

```python
# Example implementation of agent discovery
class DiscoveryService:
    def __init__(self, agents):
        self.agents = agents
        
    def ask_agent(self, asking_agent, target_agent_name, question, context=None):
        """Allow one agent to ask another agent a question"""
        # Find the target agent
        target_agent = next((a for a in self.agents if a.name == target_agent_name), None)
        if not target_agent:
            return f"Agent {target_agent_name} not found"
        
        # Format the question with context
        formatted_question = question
        if context:
            formatted_question = f"Context: {context}\n\nQuestion: {question}"
        
        # Use CrewAI's task delegation to get a response
        response = target_agent.execute_task(formatted_question)
        return response

# Usage example
discovery = DiscoveryService([planner_agent, health_agent, spiritual_agent, work_agent])

# Planner agent asking health agent a question
response = discovery.ask_agent(
    asking_agent=planner_agent,
    target_agent_name="HealthAgent",
    question="What exercise would be appropriate for Thursday given the user's fitness goals?",
    context="User has a presentation in the afternoon and needs to maintain energy"
)
```

## Agent Memory System

The memory system is implemented using CrewAI's built-in memory capabilities with custom extensions:

```python
from crewai import Crew
from crewai.memory import LongTermMemory, ShortTermMemory

# Initialize the memory systems
long_term_memory = LongTermMemory()
short_term_memory = ShortTermMemory()

# Setup the crew with memory
planning_crew = Crew(
    agents=[planner_agent, health_agent, spiritual_agent, work_agent],
    tasks=[],  # Tasks will be generated dynamically
    process="sequential",
    memory=True,
    verbose=True
)

# Implementation of the /remember command
def process_remember_command(message, crew):
    if message.startswith("/remember"):
        memory_content = message[10:].strip()  # Extract content after /remember
        
        # Store in long-term memory
        crew.memory.long_term.add(
            {"content": memory_content, "type": "user_preference"}
        )
        
        return "I'll remember this preference for future planning."
    return None

# Memory management interface functions
def list_memories(crew, memory_type="all"):
    """List all stored memories"""
    if memory_type == "long_term" or memory_type == "all":
        long_term_memories = crew.memory.long_term.get_all()
        # Format and return memories
    
    if memory_type == "short_term" or memory_type == "all":
        short_term_memories = crew.memory.short_term.get_all()
        # Format and return memories
        
    return formatted_memories

def delete_memory(crew, memory_id):
    """Delete a specific memory by ID"""
    try:
        crew.memory.long_term.delete(memory_id)
        return True
    except:
        return False
```

## Agent Configuration Interface

The system includes a user interface for configuring domain agents:

```python
# Agent configuration model
class AgentConfig:
    def __init__(self, name, description, agent_type, prompt_template, context_template):
        self.name = name
        self.description = description
        self.agent_type = agent_type
        self.prompt_template = prompt_template
        self.context_template = context_template
        self.associated_goals = []
        
    def add_associated_goal(self, goal_id):
        """Associate a goal with this agent"""
        if goal_id not in self.associated_goals:
            self.associated_goals.append(goal_id)
            
    def remove_associated_goal(self, goal_id):
        """Remove a goal association"""
        if goal_id in self.associated_goals:
            self.associated_goals.remove(goal_id)
            
    def update_prompt_template(self, new_template):
        """Update the agent's prompt template"""
        self.prompt_template = new_template
        
    def update_context_template(self, new_template):
        """Update the agent's context template"""
        self.context_template = new_template
        
# Function to create an agent from configuration
def create_agent_from_config(config):
    """Create a CrewAI agent from a configuration object"""
    # Get associated goals
    associated_goals = get_goals_by_ids(config.associated_goals)
    
    # Format the backstory using the prompt template
    backstory = config.prompt_template.format(
        associated_goals=associated_goals
    )
    
    # Create the agent
    agent = Agent(
        name=config.name,
        goal=config.description,
        backstory=backstory,
        verbose=True,
        tools=get_tools_for_agent_type(config.agent_type)
    )
    
    return agent
```

## Integration Points

The planning system integrates with other Consilium features:

### Calendar Integration

```python
def sync_plan_with_calendar(plan_items):
    """Sync plan items with the user's calendar"""
    for item in plan_items:
        # Check if item should be added to calendar
        if item.get("add_to_calendar", False):
            # Create calendar event
            calendar_id = get_user_primary_calendar_id()
            create_calendar_event(
                calendar_id=calendar_id,
                title=item["title"],
                start_time=item["start_time"],
                end_time=item["end_time"],
                description=item.get("description", "")
            )
```

### Task Integration

```python
def convert_plan_items_to_tasks(plan_items):
    """Convert relevant plan items to trackable tasks"""
    tasks = []
    for item in plan_items:
        if item.get("is_task", False):
            # Create task
            task = {
                "title": item["title"],
                "description": item.get("description", ""),
                "due_date": item.get("due_date"),
                "priority": item.get("priority", "medium"),
                "related_goal_id": item.get("related_goal_id")
            }
            tasks.append(task)
    
    # Add tasks to the task system
    add_tasks_to_system(tasks)
```

## Sample Flows

### Daily Planning Flow

```python
def daily_planning_flow(user_id):
    """Complete daily planning flow"""
    # 1. Create the planning crew with appropriate agents
    crew = setup_planning_crew(user_id)
    
    # 2. Gather context
    context = {
        "calendar_events": get_calendar_events_for_today(user_id),
        "ongoing_tasks": get_ongoing_tasks(user_id),
        "goals": get_user_goals(user_id),
        "reflection": get_latest_reflection(user_id)
    }
    
    # 3. Create the daily planning task
    daily_planning_task = Task(
        description=f"""
        Create a daily plan for the user based on their calendar, tasks, and goals.
        
        Calendar Events for Today:
        {format_calendar_events(context['calendar_events'])}
        
        Ongoing Tasks:
        {format_tasks(context['ongoing_tasks'])}
        
        Goals to Consider:
        {format_goals(context['goals'])}
        
        Recent Reflection:
        {context['reflection']}
        """,
        expected_output="A complete daily plan with time blocks and priorities",
        agent=crew.get_agent("PlannerAgent")
    )
    
    # 4. Execute the planning process
    result = crew.kickoff(tasks=[daily_planning_task])
    
    # 5. Process the result
    plan = parse_planning_result(result)
    
    # 6. Save the planning session
    save_planning_session(user_id, "daily", plan, context)
    
    return plan
```

## CrewAI Configuration

Example of CrewAI configuration for the planning system:

```python
from crewai import Crew, Process

# Configure the planning crew
planning_crew = Crew(
    agents=[planner_agent, health_agent, spiritual_agent, work_agent],
    process=Process.sequential,  # Use sequential process for coordinated planning
    verbose=True,
    memory=True,  # Enable built-in memory
    cache=True    # Cache responses for efficiency
)

# Example task creation
planning_task = Task(
    description="Create a comprehensive plan for Thursday based on the user's goals and schedule",
    expected_output="A detailed Thursday plan with time blocks and activities",
    agent=planner_agent
)

# Execute the planning process
result = planning_crew.kickoff(tasks=[planning_task])
```

This document provides a technical foundation for implementing the planning agent system in Consilium. The actual implementation may require adjustments based on specific requirements and integration points with other system components. 