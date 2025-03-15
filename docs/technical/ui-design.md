# UI/UX Design

## Overview

The Consilium user interface is designed around a chat-centric experience that provides seamless access to AI agents, notes, tasks, and planning tools. The design prioritizes simplicity, discoverability, and consistency while maintaining powerful features for power users.

## Design Principles

### 1. Chat-Centric Interaction

- Natural language is the primary interface for interacting with the system
- Chat remains accessible throughout the application for quick access to AI assistance
- Context-aware suggestions based on current view/task

### 2. Progressive Disclosure

- Core features are immediately visible and accessible
- Advanced features are revealed progressively as users need them
- Complexity is managed through thoughtful information architecture

### 3. Consistent Mental Model

- Predictable patterns across the application
- Similar actions work similarly across different sections
- Unified visual language and interaction patterns

### 4. Efficient Workflows

- Minimize clicks/taps for common tasks
- Keyboard shortcuts for power users
- Context preservation during task switching

### 5. Adaptable Experience

- Responsive design for all device sizes
- Dark/light mode support
- Customizable layouts and views

## Application Structure

The Consilium interface is structured around a three-panel layout that balances chat interaction with note management:

```
+-----------------------------------------------------------------------------------------------+
|  APP HEADER: [Logo] Consilium                                       [Search] [Settings] [User] |
+---------------+-----------------------------------+---------------------------------------+
| NAVIGATION    |           CHAT WORKSPACE          |           ACTIVE NOTE                |
|               |                                   |                                       |
| [🔍 Search]   |  Chat with Consilium              | [Note Title]                   [Edit] |
|               |                                   |                                       |
| CHATS         |  +---------------------------+    | +-----------------------------------+ |
| [+] New Chat  |  | Agent: How can I help you |    | |                                   | |
| > Project Plan|  | today?                    |    | |                                   | |
| > Task Ideas  |  |                           |    | |                                   | |
| > Meeting     |  | You: Help me organize my  |    | |         Note Content              | |
|               |  | project notes.            |    | |                                   | |
| NOTES         |  |                           |    | |     (Editable workspace)          | |
| [+] New Note  |  | Agent: I'd be happy to.   |    | |                                   | |
|               |  | Which note would you like |    | |                                   | |
| TASKS         |  | to work with?             |    | |                                   | |
| [+] New Task  |  |                           |    | +-----------------------------------+ |
| > Complete UI |  | You: Project.md           |    |                                       |
| > Update docs |  |                           |    |                                       |
|               |  | Agent: I've opened        |    |                                       |
| PINNED NOTES  |  | Project.md. What would    |    |                                       |
| > Project.md  |  | you like to do with it?   |    |                                       |
| > Goals.md    |  +---------------------------+    |                                       |
| > Song.md     |  | /                      [Send]  |                                       |
| > Ideas.md    |                                   |                                       |
| > Meeting.md  |                                   |                                       |
+---------------+-----------------------------------+---------------------------------------+
```

### Design Rationale

This three-panel layout was chosen after careful consideration of several alternatives. The key design decisions include:

1. **Combined Navigation Panel**: The left panel combines both chat history and notes access in a single column, making efficient use of vertical space while providing quick access to both types of content.

2. **Chat-Centric Workflow**: The central panel focuses on the chat interaction, which is the primary way users interact with the multi-agent system. This keeps the conversation front and center.

3. **Contextual Note Panel**: The right panel displays the active note being discussed or edited, providing immediate context for the conversation without requiring mode switching.

4. **Integrated Command Interface**: Commands are entered directly in the chat input using a slash (/) prefix, maintaining a conversational flow while providing structured interactions.

5. **Pinned Notes Section**: Frequently accessed notes can be pinned for quick access, reducing navigation time for important content.

6. **Unified Search**: The search bar at the top of the navigation panel allows searching across both chats and notes, providing a single entry point for finding information.

7. **Integrated Task Management**: Tasks are treated as specialized blocks within notes that are dynamically aggregated into a central task view, maintaining context while providing multiple ways to view and interact with tasks.

This design balances several competing needs:

- **Spatial Consistency**: Elements maintain their positions, helping users build a consistent mental model
- **Context Preservation**: Related information (chat and relevant note) is visible simultaneously
- **Efficient Space Usage**: The navigation panel serves dual purposes without wasted space
- **Minimal Mode Switching**: Users can interact with notes and chat without changing modes or views

The design is inspired by modern productivity tools while being optimized for the unique requirements of a multi-agent AI system where conversation and note-taking are deeply integrated activities.

## Key UI Components

### Navigation and Layout

#### Persistent Navigation

- **Combined Navigation Panel**: Houses both chat history and notes access
- **Command Interface**: Slash commands in chat input (e.g., "/search", "/task")
- **Search Bar**: Unified search across chats and notes
- **User Profile**: Access to user settings and profile information

#### Responsive Behavior

- **Desktop**: Three-panel layout as shown above
- **Tablet**: Collapsible navigation panel, adjustable split between chat and note
- **Mobile**: Focus on one panel at a time with easy navigation between them

### Chat Interface

#### Message Display

- **Message Types**: Text, rich content (markdown), interactive elements
- **Agent Indicators**: Visual distinction between different AI agents
- **Context Indicators**: Show what information/context is being used
- **Source References**: Links to referenced documents or notes

#### Input Mechanisms

- **Text Input**: Primary input method with autocomplete
- **Command Interface**: Type "/" to access special commands (see [Chat Commands](../features/chat-commands.md))
- **Voice Input**: Dictation option (microphone button)
- **Attachments**: Drag and drop files, images, or links

#### Agent Selection

- **Quick Switcher**: Easily switch between specialized agents
- **Context Panel**: Shows and allows editing of current context
- **Suggestions**: AI-driven suggestions for relevant agents

### Note-Taking Interface

#### Editor Components

- **Rich Text Editor**: BlockNote implementation with customizations
- **Task Block Support**: Native support for interactive task blocks
- **Toolbar**: Formatting options, insert functions, sharing
- **Focus Mode**: Distraction-free writing environment

#### Organization Views

- **List View**: Compact list of notes with previews
- **Pinned Notes**: Priority access to important notes
- **Tag View**: Notes organized by tags

### Task Management Interface

Tasks in Consilium are implemented as specialized blocks within notes, which are then dynamically aggregated into task-specific views. This approach maintains the context of tasks within their source notes while providing dedicated task management capabilities.

#### Task Implementation

- **Tasks as Note Blocks**: Tasks exist as specialized block types within the BlockNote editor
- **Metadata-Rich**: Each task block contains completion status, due dates, priority, and source reference
- **Bidirectional Syncing**: Changes to tasks are reflected both in source notes and aggregated views
- **Contextual Awareness**: Tasks maintain references to their source notes for context preservation

#### Task Views

- **Aggregated Task List**: Dynamic view that collects all task blocks across notes

#### Task Components

- **Task Block**: Custom BlockNote block with checkbox, title, metadata fields
- **Detail Expansion**: Expandable details showing source note and related information
- **Quick Add**: Interface for rapidly adding tasks to appropriate notes
- **Inline Creation**: Create tasks directly within notes or via the aggregated view

### Planning Interface

#### Goal Hierarchy

- **Visualization**: Interactive view of goal relationships
- **Timeline**: Visual representation of goals over time
- **Progress Tracking**: Visual indicators of goal progress

#### Planning Session

- **Guided Interface**: Step-by-step planning process
- **Context Panel**: Relevant goals, tasks, and notes
- **Reflection Tools**: Prompts and templates for reflection

## Visual Design

### Color System

- **Primary Color**: #3366FF - Used for primary actions and branding
- **Secondary Color**: #00CCBB - Used for secondary actions and accents
- **Neutral Colors**: Gray scale from #F8F9FA to #212529
- **Semantic Colors**:
  - Success: #28A745
  - Warning: #FFC107
  - Error: #DC3545
  - Info: #17A2B8

### Typography

- **Primary Font**: Inter - Clean, modern sans-serif for UI elements
- **Monospace Font**: JetBrains Mono - For code blocks, technical content
- **Type Scale**:
  - Heading 1: 2.25rem
  - Heading 2: 1.75rem
  - Heading 3: 1.5rem
  - Heading 4: 1.25rem
  - Body: 1rem
  - Small: 0.875rem

### Iconography

- **Style**: Outlined icons with consistent stroke width
- **Size**: 24px (standard), 20px (compact), 16px (inline)
- **Usage**: Functional icons paired with text labels for clarity

### Light and Dark Themes

- **Light Theme**: White background with dark text for bright environments
- **Dark Theme**: Dark background with light text for low-light environments
- **Auto Switching**: Based on system preferences or time of day

## Interaction Patterns

### Common Actions

- **Creation**: Consistent "+" button across sections
- **Editing**: Direct manipulation of content
- **Organization**: Drag and drop for rearrangement
- **Search**: Unified search accessible from anywhere
- **Command Execution**: Slash commands in chat for quick actions (see [Chat Commands](../features/chat-commands.md))
- **Sharing**: Consistent sharing interface across content types

### Keyboard Shortcuts

- **Global**:
  - ⌘+K / Ctrl+K: Command palette
  - ⌘+/ / Ctrl+/: Show keyboard shortcuts
  - ⌘+F / Ctrl+F: Search current view
  - Esc: Close overlay/dialog

- **Notes**:
  - ⌘+N / Ctrl+N: New note
  - ⌘+E / Ctrl+E: Edit/View toggle
  - ⌘+D / Ctrl+D: Duplicate note

- **Tasks**:
  - ⌘+T / Ctrl+T: New task
  - ⌘+Enter / Ctrl+Enter: Complete task
  - ⌘+Shift+T / Ctrl+Shift+T: Set due date

### Gesture Support

- **Swipe**: Navigate between sections
- **Pinch**: Zoom in/out of content
- **Long Press**: Reveal contextual actions
- **Double Tap**: Quick edit mode

## Accessibility

### Design for Inclusivity

- **Color Contrast**: WCAG AA compliance minimum (AAA target)
- **Text Sizing**: Scalable text without breaking layouts
- **Focus Indicators**: Clear visual focus states
- **Screen Reader Support**: Semantic HTML and ARIA attributes
- **Keyboard Navigation**: Complete functionality without mouse

## Mobile Experience

### Optimizations

- **Touch Targets**: Minimum 44×44px for interactive elements
- **Simplified Navigation**: Bottom bar with essential functions
- **Adaptive Layouts**: Reflow content for smaller screens
- **Thumb-Friendly Design**: Important actions within thumb reach

### Mobile-Specific Features

- **Offline Mode**: Full functionality without internet
- **Share Extensions**: Integration with OS sharing functionality
- **Notifications**: Native system notifications for important events
- **Widget Support**: Quick access to common functions

## UI/UX Workflow Examples

### Note Creation and Organization

1. User taps "+" button in Notes section
2. Editor opens with cursor in title field
3. As user types, AI suggests tags based on content
4. User completes note and taps "Save"
5. Note appears in appropriate sections based on content
6. AI suggests related notes and potential follow-up actions

### Task Management Flow

1. User creates a task in one of three ways:
   - Adding a task block directly within a note
   - Using the task interface
   - Using a chat command to create a task

2. When creating a task, the system:
   - Creates the task block in the appropriate note
   - Indexes the task for appearance in aggregated views
   - Maintains bidirectional references

3. Tasks appear in both:
   - Their source notes as interactive blocks
   - Aggregated task views based on metadata (due date, priority, etc.)

4. When a task is updated:
   - Changes sync bidirectionally between note and task view
   - Updates are reflected in real-time across all views

5. Completing a task:
   - Updates the task's status in both source note and aggregated views
   - Can trigger optional reflection prompts
   - Maintains the task's history within its original context

6. Task organization:
   - Tasks can be filtered and sorted in aggregated views
   - Source context is always preserved and accessible
   - Related tasks can be viewed by note source or other metadata

### Planning Session Flow

1. User initiates planning session (daily, weekly, etc.)
2. System gathers relevant goals, tasks, and notes
3. Guided interface walks through planning process
4. AI provides suggestions based on past behavior and goals
5. User finalizes plan with time blocks and priorities
6. Plan syncs with calendar and task system

### Chat Command Workflow

1. User types `/` in the chat input field
2. System displays a menu of available commands with descriptions
3. User selects or types a command (e.g., `/save`)
4. If the command requires parameters, the system provides inline guidance
5. User completes the command with necessary parameters
6. System routes the command to the appropriate agent with the required tools (see [Agent Tools](agent-tools.md))
7. Agent executes the command using the appropriate tools
8. System displays the result of the command execution to the user
9. The chat history records both the command and its outcome

## Prototype and Design System

### Design System Components

- **Component Library**: Reusable UI components with documentation
- **Style Guide**: Visual standards and usage guidelines
- **Pattern Library**: Common interaction patterns and examples
- **Accessibility Guide**: Best practices for inclusive design

### Prototyping

- **Low-Fidelity Wireframes**: Early concept exploration
- **High-Fidelity Mockups**: Detailed visual design
- **Interactive Prototypes**: Test user flows and interactions
- **User Testing**: Validate designs with real users

## Future UI Considerations

- **AR/VR Integration**: Extended reality interfaces for immersive planning
- **Voice-First Experience**: Enhanced voice interaction for hands-free use
- **AI-Generated UI**: Dynamically generated interfaces based on user behavior
- **Peripheral Integration**: Smart watch, smart displays, ambient computing
- **Biometric Adaptation**: UI that responds to emotional and physical states 