# Task Management

## Overview

The Task Management system in Consilium provides a centralized way to create, organize, and track tasks across the entire application. Tasks in Consilium are implemented as specialized blocks within notes, which are then dynamically aggregated into task-specific views. This approach maintains the context of tasks within their source notes while providing dedicated task management capabilities.

## Key Features

### Task Sources

- **Direct Creation**: Tasks created explicitly in the task management interface
- **Within Notes**: Tasks created as specialized blocks directly within notes
- **Planning Sessions**: Tasks generated during planning workflows
- **Chat Generation**: Tasks created through conversation with AI agents
- **Email Integration**: Tasks created from emails by an AI agent
- **External Applications**: Tasks imported from external applications like ClickUp

### Task Implementation

- **Tasks as Note Blocks**: Tasks exist as specialized block types within the BlockNote editor
- **Metadata-Rich**: Each task block contains completion status, due dates, priority, and source reference
- **Bidirectional Syncing**: Changes to tasks are reflected both in source notes and aggregated views
- **Contextual Awareness**: Tasks maintain references to their source notes for context preservation
- **Source Indicators**: Visual indicators showing where a task originated (note, email, ClickUp, etc.)

### Task Properties

- **Basic Properties**: Title, description, due date, priority
- **Status Tracking**: Not started, in progress, completed, deferred
- **Categories**: User-defined categories for organization
- **Tags**: Flexible tagging system for cross-cutting organization
- **Subtasks**: Hierarchical task breakdown
- **Assignments**: Assign tasks to people (self or others)
- **Recurrence**: Set up repeating tasks with flexible schedules
  - Support for daily, weekly, monthly, and custom patterns
  - Natural language input (e.g., "every Monday", "first of each month")
  - Configurable end conditions (specific date, number of occurrences, or indefinite)
  - Options for handling completion status of recurring instances
- **Time Estimates**: Optional time estimates for tasks
- **Size**: Numeric value for effort estimation (for planning and prioritization)
- **Attachments**: Files, links, or references attached to tasks
- **Notes**: Additional notes or context for tasks
- **Reminders**: Notification triggers that can be attached to tasks

## User Interface

### Task Views

- **Aggregated Task List**: Dynamic view that collects all task blocks across notes

### Interaction Models

- **List View**: Traditional list-based task management
- **Source Context View**: See tasks within their original notes

## Technical Implementation

### Data Model

```
Task {
  id: UUID
  title: String
  description: Text
  created_at: Timestamp
  updated_at: Timestamp
  user_id: UUID
  note_id: UUID (reference to source note)
  block_id: String (reference to specific block in the note)
  due_date: Date (optional)
  due_time: Time (optional)
  status: Enum (not_started, in_progress, completed, deferred)
  priority: Enum (high, medium, low)
  category_id: UUID (optional)
  parent_task_id: UUID (optional, for subtasks)
  source: Enum (direct, note, planning, chat, email, clickup, external)
  source_reference: String (reference ID in the source system if external)
  time_estimate: Integer (in minutes, optional)
  size: Float (optional, numeric value for effort estimation)
  tags: String[] (array of tag IDs)
  recurrence: JSON {
    pattern: String (e.g., "daily", "weekly", "monthly", "custom")
    frequency: Integer (e.g., every 2 weeks)
    days_of_week: Integer[] (array of days, where 0=Sunday, 1=Monday, etc.)
    day_of_month: Integer (1-31, for monthly patterns)
    month_of_year: Integer (1-12, for yearly patterns)
    end_type: String ("never", "after", "on_date")
    end_after_occurrences: Integer (if end_type is "after")
    end_date: Date (if end_type is "on_date")
    skip_completed: Boolean (whether to skip creating the next instance when current one is completed)
    create_next_on_completion: Boolean (whether to immediately create the next instance when current one is completed)
  }
  metadata: JSON
}

Category {
  id: UUID
  name: String
  color: String
  user_id: UUID
  parent_category_id: UUID (optional)
}

Reminder {
  id: UUID
  title: String
  description: Text
  trigger_time: Timestamp
  task_id: UUID (optional, reference to associated task)
  recurring_pattern: String (optional, cron-like pattern)
  last_triggered: Timestamp (optional)
  created_at: Timestamp
  updated_at: Timestamp
  user_id: UUID
  is_active: Boolean
  is_acknowledged: Boolean
  notification_sent: Boolean
  metadata: JSON
}
```

### BlockNote Integration

- **Custom Task Block**: Specialized BlockNote block type for tasks
- **Rich Interaction**: Interactive checkbox, due date picker, priority selector
- **Metadata Storage**: Task metadata stored within the block data structure
- **Rendering Options**: Different display modes for tasks in notes vs. task views

### External Integrations

#### Email Integration

Tasks can be automatically created from emails through an AI agent workflow:

- **Email Scanning Agent**: An AI agent that scans the user's inbox for task-like content
- **Natural Language Processing**: Identifies potential tasks in email content
- **Context Preservation**: Creates tasks with references to the original emails
- **Visual Indicators**: Tasks show they originated from email with appropriate icons/styling
- **Email Reply Updates**: Updates to tasks can be made by replying to the original email

#### ClickUp Integration

Tasks from ClickUp represent the first of many potential external application integrations:

- **Task Synchronization**: Import tasks from ClickUp into the Consilium notes system
- **Bidirectional Status Updates**: Completed tasks are marked as done in ClickUp
- **Automatic Removal**: External tasks are removed from notes when completed
- **Visual Distinction**: ClickUp tasks have distinct styling to indicate their source

#### External Application Framework

External application integration is built with an extensible framework:

- **Agent-Based Connectivity**: AI agents with specific tools for external systems
- **Composio Integration**: Using Composio (https://composio.dev/) via CrewAI for connecting to external APIs
- **Flexible Connection Model**: Framework designed to easily add new external services
- **Unified Task Experience**: External tasks appear alongside native tasks in all views

### Synchronized Task Instances

Tasks in Consilium can appear in multiple notes while maintaining synchronization:

- **Multiple Instances**: The same logical task can exist in multiple notes
- **Shared Identity**: All instances of the same task share a common identifier
- **Synchronized Updates**: Changes to one instance update all instances
- **Consistent Visualization**: Task source and metadata remain consistent across all instances

#### How Synchronization Works

1. **Shared Task ID**: When a task is added to multiple notes, all instances share a common `shared_task_id`
2. **Task Index Tracking**: The central task index tracks all instances and their relationships
3. **Bidirectional Updates**: When a task is updated in any location, the task index updates all instances
4. **Source Preservation**: The original source (note, email, ClickUp, etc.) is maintained across all instances

#### Cross-Note Task Management

The system supports various scenarios for tasks across multiple notes:

- **Adding Existing Tasks**: When adding an existing task to another note, a new synchronized instance is created
- **Completion Handling**: Completing a task in one location completes it in all locations
- **Source-Based Behavior**: External tasks (email, ClickUp) maintain their special behaviors in all locations

#### Implementation Details

```
Task {
  id: UUID                         // Unique ID for this specific task instance
  shared_task_id: UUID (optional)  // Common ID shared by all instances of the same task
  note_id: UUID                    // Reference to containing note
  block_id: String                 // Reference to specific block in the note
  // ... other task fields
}
```

### External Task Behavior

#### Email Tasks

Email tasks have special characteristics:

- **Email Source**: Marked as originating from email
- **Link to Original**: Maintain reference to the original email
- **Synchronized Instances**: Can appear in multiple notes with synchronized status
- **Reply Interface**: Can be replied to via the `/email-reply` command from any instance

#### ClickUp Tasks

ClickUp tasks represent the model for external application integration:

- **External Source**: Visually indicated as coming from ClickUp
- **Bidirectional Status**: Status changes sync between Consilium and ClickUp
- **Completion Behavior**: When completed, they are marked as done in ClickUp and removed from all notes
- **Synchronization**: All instances of a ClickUp task stay synchronized

### Recurring Task Flow

1. **Creating Recurring Tasks**:
   - User creates a task and adds recurrence by clicking the recurrence icon
   - User enters a natural language pattern (e.g., "every Monday at 9am", "first of the month")
   - System displays interpreted recurrence pattern for confirmation
   - User can customize advanced options like end conditions and instance creation behavior

2. **Managing Recurring Task Instances**:
   - When a task instance is completed, the next instance is generated based on the recurrence pattern
   - Each instance has its own due date based on the pattern
   - User can view all upcoming instances from the task detail view
   - User can edit individual instances without affecting the pattern
   - User can edit the recurrence pattern, affecting all future instances

3. **Completion Behavior Options**:
   - Standard: When a task is completed, it's marked complete and the next instance is generated
   - Skip when completed: If a task is completed early, the next instance is skipped (useful for "as-needed" tasks)
   - Create next on completion: Next instance is immediately created when current one is completed (useful for sequential tasks)

4. **Recurrence Pattern Changes**:
   - Changes to the pattern affect only future instances
   - Users can choose to regenerate all future instances when changing a pattern
   - Individual instances can be exempt from the recurrence pattern as needed
   - Deleting a recurring task provides options to delete just that instance or all future instances

5. **Visual Indicators**:
   - Recurring tasks have distinct visual indicators showing their recurrence pattern
   - Upcoming recurring task instances are visually grouped in task views
   - Missed recurring tasks are highlighted for attention

### Cross-Note Task Flow

1. **Adding to Multiple Notes**: 
   - User can add an existing task by using the @ symbol in the note editor which brings up a fuzzy search tool to find the task the user is looking for

2. **Updating Synchronized Tasks**:
   - User updates any instance of a task (status, due date, etc.)
   - System automatically updates all instances of that task
   - User receives visual confirmation that all instances were updated

3. **Deleting Task Instances**:
   - When deleting a task that exists in multiple notes only the one task instance is removed

4. **Working with External Tasks**:
   - External tasks maintain their special behaviors in all locations
   - Email tasks can be replied to from any instance using `/email-reply`
   - ClickUp tasks are removed from all notes when completed

### Task Reminders

Users can attach reminders to tasks through the task interface:

1. **Adding Reminders to Tasks**:
   - From the task detail view, click the "Add Reminder" button
   - Enter a natural language phrase describing when to be reminded (e.g., "tomorrow afternoon", "next Friday at 3pm", "in 2 hours")
   - The system parses and confirms the interpreted time
   - User can adjust by entering a new phrase if needed

2. **Managing Task Reminders**:
   - View all reminders associated with a task in the task detail view
   - Edit reminder times using natural language phrases
   - Delete reminders that are no longer needed

3. **Notification Behavior**:
   - Reminders trigger push notifications at the specified time
   - Notifications include the task title and a direct link to the task
   - For recurring reminders, each occurrence generates a separate notification
   - Notifications are marked as acknowledged when the user views the task

4. **Reminder Centralization**:
   - All reminders (both task-attached and standalone) are managed in a central system
   - Users can view all upcoming reminders in the reminders dashboard
   - Filtering options allow viewing only task reminders or only standalone reminders
   - The system presents reminders in natural language format (e.g., "Tomorrow at 9 AM")

## Future Enhancements

- **Team Collaboration**: Shared task lists and assignments
- **Dependency Tracking**: Set and track dependencies between tasks
- **Recurrence Enhancement**:
  - Custom recurrence rules using RRULE format
- **Time Tracking**: Built-in time tracking for tasks
- **Achievement System**: Gamification elements for task completion
- **Expanded External Integrations**: Add support for Trello, Asana, Jira, and other task systems
- **Email Task Extraction Rules**: User-defined rules for which emails create tasks
- **Global Task Search**: Search across all task sources with unified results
- **Size-Based Planning**: Enhanced planning tools that use task size for workload balancing
- **Velocity Tracking**: Track completion rates of tasks by size to improve estimation

## Integration with Other Features

### Notes Integration

- **Native Integration**: Tasks are a native part of notes as specialized blocks
- **Bi-directional Syncing**: Changes in tasks sync between notes and task views
- **Context Preservation**: Tasks always maintain link to original note context
- **Inline Task Creation**: Create tasks directly within notes while writing

### Planning Integration

- **Goal Alignment**: Associate tasks with goals at various levels
- **Progress Tracking**: Track goal progress through task completion
- **Planning Context**: Tasks maintain link to planning session context
- **Effort Estimation**: Size field for tasks provides numeric effort estimation for planning and prioritization
- **AI-Assisted Planning**: AI uses size values for tasks to suggest realistic task scheduling and workload management

### Chat Integration

- **Task Creation**: Create tasks directly in chat interface with note context
- **Task Updates**: Update task status through chat
- **Task Queries**: Ask about tasks and get summaries through chat
- **External Task Management**: Create, update, and query external tasks through chat commands

## User Experience

### Task Creation Flow

1. User creates a task in one of several ways:
   - Adding a task block directly within a note
   - Using the task interface in the navigation panel
   - Using a chat command to create a task
   - Tasks are automatically created from emails by the email scanning agent
   - Tasks are imported from external applications like ClickUp

2. When creating a task, the system:
   - Creates the task block in the appropriate note
   - Indexes the task for appearance in aggregated views
   - Maintains bidirectional references
   - For external tasks, stores reference to the original source

3. Tasks appear in:
   - Their source notes as interactive blocks
   - Aggregated task views based on metadata (due date, priority, etc.)
   - With visual indicators of their source (note, email, ClickUp, etc.)

### Task Management Flow

1. **Planning**: Organize and prioritize tasks
2. **Execution**: Mark tasks as in progress and completed
3. **Review**: Review completed and outstanding tasks
4. **Analysis**: Gain insights from task completion patterns
5. **External Synchronization**: Updates to external tasks propagate to source systems

### External Task Handling

1. **Email Tasks**:
   - Created by an AI agent scanning emails for task-like content
   - Include a link to the original email
   - Visually indicate they originated from email
   - Can be responded to by replying to the original email

2. **ClickUp Tasks**:
   - Imported from ClickUp and displayed in notes/task views
   - When marked complete in Consilium, they're updated in ClickUp and removed from notes
   - First of many planned external integrations via agent workflows

### Recurring Task Flow

1. **Creating Recurring Tasks**:
   - User creates a task and adds recurrence by clicking the recurrence icon
   - User enters a natural language pattern (e.g., "every Monday at 9am", "first of the month")
   - System displays interpreted recurrence pattern for confirmation
   - User can customize advanced options like end conditions and instance creation behavior

2. **Managing Recurring Task Instances**:
   - When a task instance is completed, the next instance is generated based on the recurrence pattern
   - Each instance has its own due date based on the pattern
   - User can view all upcoming instances from the task detail view
   - User can edit individual instances without affecting the pattern
   - User can edit the recurrence pattern, affecting all future instances

3. **Completion Behavior Options**:
   - Standard: When a task is completed, it's marked complete and the next instance is generated
   - Skip when completed: If a task is completed early, the next instance is skipped (useful for "as-needed" tasks)
   - Create next on completion: Next instance is immediately created when current one is completed (useful for sequential tasks)

4. **Recurrence Pattern Changes**:
   - Changes to the pattern affect only future instances
   - Users can choose to regenerate all future instances when changing a pattern
   - Individual instances can be exempt from the recurrence pattern as needed
   - Deleting a recurring task provides options to delete just that instance or all future instances

5. **Visual Indicators**:
   - Recurring tasks have distinct visual indicators showing their recurrence pattern
   - Upcoming recurring task instances are visually grouped in task views
   - Missed recurring tasks are highlighted for attention