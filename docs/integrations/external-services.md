# External Service Integrations

## Overview

Consilium integrates with various external services to enhance functionality and provide seamless connections with tools users already rely on. This document outlines the integration points, configuration details, and implementation strategies for each external service.

## Integration Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Consilium System                            │
│                                                                     │
│  ┌───────────────────┐   ┌───────────────────┐   ┌───────────────┐  │
│  │  Integration      │   │  Auth Service     │   │  Core Services│  │
│  │  Manager          │   │                   │   │               │  │
│  └─────────┬─────────┘   └──────────┬────────┘   └───────────────┘  │
└────────────┼──────────────────────────────────────────────────────────────────┘
             │                        │                                          
             ▼                        ▼                                          
┌─────────────────────────────────────────────────────────────────────┐
│                      Integration Service Layer                       │
│                                                                      │
│  ┌───────────────┐  ┌────────────────┐  ┌──────────────┐  ┌────────┐ │
│  │ OAuth Service │  │ Webhook Handler│  │ API Clients  │  │Adapters│ │
│  └───────┬───────┘  └───────┬────────┘  └──────┬───────┘  └───┬────┘ │
└──────────┼────────────────────────────────────┼───────────────┼──────┘
           │                │                   │               │
           ▼                ▼                   ▼               ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
│  ClickUp     │  │   Email      │  │  Calendar    │  │  Other Services  │
│  Integration │  │  Integration │  │  Integration │  │  (Future)        │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────────┘
```

## ClickUp Integration

ClickUp is integrated to synchronize tasks between Consilium and ClickUp workspaces, enabling users to manage church tasks and other organizational activities alongside personal tasks.

### Authentication

- **Method**: OAuth 2.0
- **Scopes Required**: `task:read`, `task:write`, `team:read`
- **Token Storage**: Encrypted in user profile in database

### Features

- **Task Synchronization**: Bi-directional sync of tasks between systems
- **Workspace Mapping**: Map ClickUp workspaces to specific categories in Consilium
- **Status Mapping**: Map ClickUp statuses to Consilium task statuses
- **Assignee Handling**: Preserve assignee information across systems
- **Due Date Synchronization**: Keep due dates consistent across systems

### Implementation Details

```python
# Example ClickUp client
class ClickUpClient:
    def __init__(self, access_token: str):
        self.access_token = access_token
        self.base_url = "https://api.clickup.com/api/v2"
        
    def get_teams(self):
        """Get all teams accessible to the authenticated user."""
        response = requests.get(
            f"{self.base_url}/team",
            headers={"Authorization": self.access_token}
        )
        response.raise_for_status()
        return response.json()["teams"]
    
    def get_tasks(self, list_id: str, params: dict = None):
        """Get tasks from a specific list."""
        response = requests.get(
            f"{self.base_url}/list/{list_id}/task",
            headers={"Authorization": self.access_token},
            params=params or {}
        )
        response.raise_for_status()
        return response.json()["tasks"]
    
    def create_task(self, list_id: str, task_data: dict):
        """Create a new task in the specified list."""
        response = requests.post(
            f"{self.base_url}/list/{list_id}/task",
            headers={
                "Authorization": self.access_token,
                "Content-Type": "application/json"
            },
            json=task_data
        )
        response.raise_for_status()
        return response.json()
    
    # Additional methods for updating, deleting tasks...
```

### Synchronization Strategy

1. **Initial Sync**:
   - Fetch all tasks from ClickUp workspaces
   - Create corresponding tasks in Consilium
   - Store mapping between ClickUp task IDs and Consilium task IDs

2. **Ongoing Sync**:
   - Poll for changes at regular intervals (e.g., every 30 minutes)
   - Use timestamps to identify changes since last sync
   - Apply changes to the appropriate system

3. **Conflict Resolution**:
   - If a task is updated in both systems, prefer the most recent change
   - Provide notification of conflicts that require manual resolution
   - Allow user to select which version to keep

### User Configuration

Users can configure the ClickUp integration through the settings panel:

- Connect/disconnect their ClickUp account
- Select which workspaces to sync
- Configure category mapping
- Set sync frequency preferences
- Enable/disable notifications for sync events

## Email Integration

Email integration allows users to create tasks directly from emails and receive task notifications via email.

### Authentication

- **Method**: OAuth 2.0 for Gmail, Microsoft Graph API for Outlook
- **IMAP/SMTP**: For other email providers
- **Scopes Required**: Read and send email permissions

### Features

- **Email Forwarding**: Dedicated email address for forwarding tasks
- **Email Parsing**: Extract task details from email content
- **Email Commands**: Perform actions via specially formatted emails
- **Task Notifications**: Receive email notifications for task events
- **Reply Processing**: Update tasks by replying to notification emails

### Implementation Details

```python
# Example email processor
class EmailProcessor:
    def process_incoming_email(self, email_data: dict) -> Optional[Task]:
        """Process an incoming email and convert to a task if appropriate."""
        # Extract email details
        sender = email_data["from"]
        subject = email_data["subject"]
        body = email_data["body"]
        
        # Verify sender is authorized
        user = self.get_user_from_email(sender)
        if not user:
            return None
        
        # Check if this is a task creation email
        if self.is_task_creation_email(subject, body):
            # Extract task details
            task_details = self.extract_task_details(subject, body)
            
            # Create the task
            task = Task(
                title=task_details["title"],
                description=task_details["description"],
                due_date=task_details.get("due_date"),
                priority=task_details.get("priority", "medium"),
                user_id=user.id,
                source="email",
                source_reference=email_data["message_id"]
            )
            
            return task
        
        return None
        
    def extract_task_details(self, subject: str, body: str) -> dict:
        """Extract task details from email subject and body."""
        # Basic implementation: use subject as title, body as description
        task_details = {
            "title": subject,
            "description": body
        }
        
        # Look for due date patterns like "due:YYYY-MM-DD" in body
        due_date_match = re.search(r"due:(\d{4}-\d{2}-\d{2})", body)
        if due_date_match:
            task_details["due_date"] = due_date_match.group(1)
        
        # Look for priority patterns like "priority:high" in body
        priority_match = re.search(r"priority:(high|medium|low)", body)
        if priority_match:
            task_details["priority"] = priority_match.group(1)
        
        return task_details
```

### Email Processing Flow

1. **Receive Email**:
   - User forwards email to dedicated address or sends specially formatted email
   - Email is received by email processing service

2. **Authentication**:
   - Verify sender is registered user or authorized email address
   - Check for any required authentication tokens or codes

3. **Parsing**:
   - Extract task information from email subject and body
   - Look for special formatting or commands
   - Process any attachments

4. **Task Creation**:
   - Create task with extracted information
   - Associate with appropriate categories based on rules
   - Store reference to original email

5. **Confirmation**:
   - Send confirmation email to user
   - Include link to the created task

### User Configuration

Users can configure the email integration through settings:

- Connect/disconnect email accounts
- Set up email forwarding
- Configure parsing rules and defaults
- Set notification preferences
- Create email filters and rules

## Calendar Integration

Calendar integration synchronizes tasks and planning sessions with external calendar services.

### Supported Providers

- Google Calendar
- Microsoft Outlook Calendar
- Apple Calendar (via CalDAV)
- Other CalDAV-compatible calendars

### Authentication

- **Google Calendar**: OAuth 2.0 with calendar read/write scopes
- **Microsoft Calendar**: OAuth 2.0 with Microsoft Graph API
- **CalDAV**: Username/password or token-based authentication

### Features

- **Task Synchronization**: Convert tasks with due dates to calendar events
- **Planning Sessions**: Add planning sessions to calendar
- **Event Import**: Create tasks from calendar events
- **Time Blocking**: Create calendar blocks for focused work
- **Two-way Sync**: Update events in either system

### Implementation Details

```python
# Example Google Calendar client
class GoogleCalendarClient:
    def __init__(self, credentials: dict):
        self.service = build("calendar", "v3", credentials=credentials)
        
    def create_event(self, calendar_id: str, event_data: dict) -> dict:
        """Create a new event in the specified calendar."""
        event = self.service.events().insert(
            calendarId=calendar_id,
            body=event_data
        ).execute()
        return event
    
    def update_event(self, calendar_id: str, event_id: str, event_data: dict) -> dict:
        """Update an existing event."""
        event = self.service.events().update(
            calendarId=calendar_id,
            eventId=event_id,
            body=event_data
        ).execute()
        return event
    
    def delete_event(self, calendar_id: str, event_id: str) -> None:
        """Delete an event."""
        self.service.events().delete(
            calendarId=calendar_id,
            eventId=event_id
        ).execute()
    
    def list_events(self, calendar_id: str, time_min: str, time_max: str) -> list:
        """List events in a specified time range."""
        events_result = self.service.events().list(
            calendarId=calendar_id,
            timeMin=time_min,
            timeMax=time_max,
            singleEvents=True,
            orderBy="startTime"
        ).execute()
        return events_result.get("items", [])
```

### Calendar Mapping

- **Tasks → Events**: Tasks with due dates and time estimates become calendar events
- **Planning Sessions → Events**: Planning sessions reserved on calendar
- **Events → Tasks**: External events can be imported as tasks

### Sync Behavior

- **Task Updates**: When a task changes, update corresponding calendar event
- **Event Updates**: When a calendar event changes, update corresponding task
- **Deletion Handling**: Respect deletions in either system
- **Conflict Resolution**: User preference for which system takes precedence

### User Configuration

Users configure calendar integration through settings:

- Connect/disconnect calendar accounts
- Select which calendars to sync
- Configure default visibility and notifications
- Set task-to-event mapping preferences
- Configure sync frequency

## Future Integrations

### Planned Integrations

- **Trello**: For project and task management
- **Slack**: For team communication and notifications
- **GitHub**: For development task tracking
- **Jira**: For enterprise project management
- **Notion**: For knowledge management

### Integration Framework

The integration system is designed to be extensible, allowing new integrations to be added without modifying core code:

1. **Integration Registry**: Central registry of available integrations
2. **Adapter Pattern**: Common interface for different service types
3. **Authentication Service**: Unified OAuth handling across integrations
4. **Webhook Manager**: Process incoming webhooks from external services
5. **Plugin Architecture**: Allow third-party integrations to be developed

### User-Configurable Integrations

Future versions will include:

- **Integration Marketplace**: Discover and enable integrations
- **Automation Rules**: Create custom workflows between integrations
- **Custom API Connections**: Connect to services without built-in integrations
- **Zapier/IFTTT Integration**: Connect to hundreds of services through these platforms 