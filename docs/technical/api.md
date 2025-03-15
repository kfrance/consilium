# API Documentation

## Overview

The Consilium API provides programmatic access to notes, tasks, planning, and AI functionality. Built with FastAPI, it follows RESTful principles and uses JSON for data exchange.

## Base URL

```
https://api.consilium.app/v1
```

For local development:

```
http://localhost:8000/v1
```

## Authentication

### Authentication Methods

The API uses JWT (JSON Web Token) authentication.

#### Obtaining Tokens

```http
POST /auth/token
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "secure_password"
}
```

Response:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

#### Refreshing Tokens

```http
POST /auth/refresh
Content-Type: application/json

{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### Using Tokens

Include the access token in the Authorization header for all authenticated requests:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Rate Limiting

API requests are subject to rate limiting:

- 60 requests per minute for authenticated users
- 10 requests per minute for unauthenticated requests

Rate limit headers included in responses:
- `X-RateLimit-Limit`: Maximum requests per window
- `X-RateLimit-Remaining`: Remaining requests in current window
- `X-RateLimit-Reset`: Time when the current window resets (Unix timestamp)

## Error Handling

### Error Response Format

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": {}
  }
}
```

### Common HTTP Status Codes

- `200 OK`: Request succeeded
- `201 Created`: Resource created
- `400 Bad Request`: Invalid request parameters
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Permission denied
- `404 Not Found`: Resource not found
- `409 Conflict`: Request conflicts with current state
- `422 Unprocessable Entity`: Validation error
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error

## API Endpoints

### User Management

#### Get Current User

```http
GET /users/me
Authorization: Bearer {access_token}
```

Response:

```json
{
  "id": "user_123",
  "email": "user@example.com",
  "name": "John Doe",
  "created_at": "2023-01-15T00:00:00Z",
  "preferences": {
    "theme": "dark",
    "timezone": "America/New_York"
  }
}
```

#### Update User Preferences

```http
PATCH /users/me/preferences
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "theme": "light",
  "timezone": "Europe/London"
}
```

### Notes

#### List Notes

```http
GET /notes
Authorization: Bearer {access_token}
```

Query Parameters:
- `folder_id` (optional): Filter by folder
- `tag_id` (optional): Filter by tag
- `search` (optional): Search term
- `sort` (optional): Sort field (default: updated_at)
- `order` (optional): Sort order (asc/desc, default: desc)
- `page` (optional): Page number (default: 1)
- `page_size` (optional): Items per page (default: 20)

Response:

```json
{
  "items": [
    {
      "id": "note_123",
      "title": "Meeting Notes",
      "content": {
        "type": "doc",
        "content": [
          {
            "type": "paragraph",
            "content": [
              {
                "type": "text",
                "text": "Important meeting notes"
              }
            ]
          }
        ]
      },
      "created_at": "2023-01-15T00:00:00Z",
      "updated_at": "2023-01-16T00:00:00Z",
      "user_id": "user_123",
      "parent_id": "folder_456",
      "tags": ["tag_789"]
    }
  ],
  "pagination": {
    "total": 45,
    "page": 1,
    "page_size": 20,
    "pages": 3
  }
}
```

#### Get Note

```http
GET /notes/{note_id}
Authorization: Bearer {access_token}
```

Response: Same as note object in list response

#### Create Note

```http
POST /notes
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "title": "New Note",
  "content": {
    "type": "doc",
    "content": [
      {
        "type": "paragraph",
        "content": [
          {
            "type": "text",
            "text": "This is a new note"
          }
        ]
      }
    ]
  },
  "parent_id": "folder_456",
  "tags": ["tag_789"]
}
```

Response: Created note object

#### Update Note

```http
PUT /notes/{note_id}
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "title": "Updated Note",
  "content": {
    "type": "doc",
    "content": [
      {
        "type": "paragraph",
        "content": [
          {
            "type": "text",
            "text": "Updated content"
          }
        ]
      }
    ]
  },
  "parent_id": "folder_456",
  "tags": ["tag_789", "tag_101"]
}
```

Response: Updated note object

#### Delete Note

```http
DELETE /notes/{note_id}
Authorization: Bearer {access_token}
```

Response:

```json
{
  "success": true,
  "message": "Note deleted"
}
```

### Tasks

#### List Tasks

```http
GET /tasks
Authorization: Bearer {access_token}
```

Query Parameters:
- `category_id` (optional): Filter by category
- `status` (optional): Filter by status
- `due_date` (optional): Filter by due date
- `priority` (optional): Filter by priority
- `search` (optional): Search term
- `page` (optional): Page number
- `page_size` (optional): Items per page

Response:

```json
{
  "items": [
    {
      "id": "task_123",
      "title": "Complete project proposal",
      "description": "Draft the proposal for the new client",
      "status": "in_progress",
      "priority": "high",
      "due_date": "2023-02-15",
      "created_at": "2023-01-15T00:00:00Z",
      "updated_at": "2023-01-16T00:00:00Z",
      "user_id": "user_123",
      "category_id": "category_456",
      "size": 5,
      "tags": ["tag_789"]
    }
  ],
  "pagination": {
    "total": 28,
    "page": 1,
    "page_size": 20,
    "pages": 2
  }
}
```

#### Get Task

```http
GET /tasks/{task_id}
Authorization: Bearer {access_token}
```

Response: Same as task object in list response

#### Create Task

```http
POST /tasks
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "title": "New Task",
  "description": "Task description",
  "status": "not_started",
  "priority": "medium",
  "due_date": "2023-03-01",
  "category_id": "category_456",
  "size": 3,
  "tags": ["tag_789"]
}
```

Response: Created task object

#### Update Task

```http
PUT /tasks/{task_id}
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "title": "Updated Task",
  "description": "Updated description",
  "status": "in_progress",
  "priority": "high",
  "due_date": "2023-03-10",
  "category_id": "category_456",
  "size": 8,
  "tags": ["tag_789", "tag_101"]
}
```

Response: Updated task object

#### Delete Task

```http
DELETE /tasks/{task_id}
Authorization: Bearer {access_token}
```

Response:

```json
{
  "success": true,
  "message": "Task deleted"
}
```

### Goals and Planning

#### List Goals

```http
GET /goals
Authorization: Bearer {access_token}
```

Query Parameters:
- `timeframe` (optional): Filter by timeframe (5-year, 1-year, monthly, weekly, daily)
- `status` (optional): Filter by status
- `parent_id` (optional): Filter by parent goal

Response:

```json
{
  "items": [
    {
      "id": "goal_123",
      "title": "Learn new programming language",
      "description": "Become proficient in Rust programming",
      "timeframe": "1-year",
      "status": "in_progress",
      "progress": 35,
      "created_at": "2023-01-15T00:00:00Z",
      "updated_at": "2023-01-16T00:00:00Z",
      "user_id": "user_123",
      "parent_goal_id": "goal_456"
    }
  ],
  "pagination": {
    "total": 15,
    "page": 1,
    "page_size": 20,
    "pages": 1
  }
}
```

#### Create Planning Session

```http
POST /planning/sessions
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "session_type": "weekly",
  "notes": "Planning for week of March 10",
  "goals": ["goal_123", "goal_456"]
}
```

Response:

```json
{
  "id": "session_123",
  "session_type": "weekly",
  "notes": "Planning for week of March 10",
  "started_at": "2023-03-10T09:00:00Z",
  "ended_at": null,
  "user_id": "user_123",
  "goals": ["goal_123", "goal_456"],
  "status": "in_progress"
}
```

### AI Interaction

#### Chat Completion

```http
POST /ai/chat
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "messages": [
    {
      "role": "user",
      "content": "Help me plan my week"
    }
  ],
  "agent": "planner",
  "context": {
    "include_goals": true,
    "include_tasks": true,
    "include_calendar": true
  }
}
```

Response:

```json
{
  "id": "chat_123",
  "created_at": "2023-03-10T09:05:00Z",
  "messages": [
    {
      "role": "user",
      "content": "Help me plan my week"
    },
    {
      "role": "assistant",
      "content": "I can help you plan your week. Looking at your current goals and tasks, here's a suggested plan...",
      "metadata": {
        "agent": "planner",
        "sources": [
          {
            "type": "goal",
            "id": "goal_123"
          },
          {
            "type": "task",
            "id": "task_456"
          }
        ]
      }
    }
  ],
  "agent": "planner"
}
```

#### Document Query (RAG)

```http
POST /ai/query
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "query": "What did I write about the new project requirements?",
  "filters": {
    "document_types": ["note"],
    "date_range": {
      "start": "2023-01-01",
      "end": "2023-03-10"
    },
    "tags": ["project", "requirements"]
  }
}
```

Response:

```json
{
  "answer": "Based on your notes, the new project requirements include implementing a user authentication system, designing a responsive dashboard, and integrating with the payment gateway...",
  "sources": [
    {
      "id": "note_123",
      "title": "Project Requirements Meeting",
      "snippet": "...discussed the authentication system requirements including OAuth support...",
      "relevance_score": 0.92
    },
    {
      "id": "note_456",
      "title": "Dashboard Design Notes",
      "snippet": "...responsive dashboard should include user metrics and activity charts...",
      "relevance_score": 0.85
    }
  ]
}
```

### External Integrations

#### List Connected Services

```http
GET /integrations/services
Authorization: Bearer {access_token}
```

Response:

```json
{
  "items": [
    {
      "id": "integration_123",
      "service": "clickup",
      "connected_at": "2023-02-15T00:00:00Z",
      "status": "active",
      "metadata": {
        "workspace_ids": ["workspace_123"]
      }
    },
    {
      "id": "integration_456",
      "service": "gmail",
      "connected_at": "2023-01-10T00:00:00Z",
      "status": "active",
      "metadata": {
        "email": "user@example.com"
      }
    }
  ]
}
```

#### Connect Service

```http
POST /integrations/services/{service_name}/connect
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "auth_code": "oauth_code_from_service",
  "redirect_uri": "https://consilium.app/oauth/callback"
}
```

Response:

```json
{
  "id": "integration_789",
  "service": "clickup",
  "connected_at": "2023-03-10T10:00:00Z",
  "status": "active",
  "metadata": {
    "workspace_ids": ["workspace_123"]
  }
}
```

#### Disconnect Service

```http
DELETE /integrations/services/{integration_id}
Authorization: Bearer {access_token}
```

Response:

```json
{
  "success": true,
  "message": "Service disconnected"
}
```

## Webhooks

### Webhook Format

Consilium can send webhook notifications when events occur in your account.

```json
{
  "event": "note.created",
  "id": "event_123",
  "created_at": "2023-03-10T10:15:00Z",
  "data": {
    "note_id": "note_123",
    "title": "New Note",
    "user_id": "user_123"
  }
}
```

### Available Events

- `note.created`
- `note.updated`
- `note.deleted`
- `task.created`
- `task.updated`
- `task.completed`
- `task.deleted`
- `goal.created`
- `goal.updated`
- `goal.completed`
- `integration.connected`
- `integration.disconnected`

### Managing Webhooks

#### List Webhooks

```http
GET /webhooks
Authorization: Bearer {access_token}
```

#### Create Webhook

```http
POST /webhooks
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "url": "https://example.com/webhooks/consilium",
  "events": ["note.created", "task.completed"],
  "description": "Notification webhook for my integration"
}
```

#### Delete Webhook

```http
DELETE /webhooks/{webhook_id}
Authorization: Bearer {access_token}
```

## API Versioning

The API uses versioning to ensure backwards compatibility:

- Current stable version: `v1`
- Beta version (if available): `v2-beta`

Include the version in the URL path:
- `https://api.consilium.app/v1/notes`
- `https://api.consilium.app/v2-beta/notes`

## SDKs and Libraries

Official client libraries:

- JavaScript/TypeScript: [consilium-js](https://github.com/consilium/consilium-js)
- Python: [consilium-python](https://github.com/consilium/consilium-python)

## Testing and Sandbox

For development and testing, use the sandbox environment:

```
https://api-sandbox.consilium.app/v1
```

The sandbox environment has the following characteristics:
- Isolated from production data
- Test accounts available
- Rate limits are more generous
- May include experimental features

## API Changes and Deprecation

The API follows these versioning principles:

- Backwards-compatible changes may be added to existing versions
- Breaking changes require a new API version
- Deprecated features will be announced at least 6 months in advance
- Deprecated versions will be supported for at least 12 months

Subscribe to the [API Changes mailing list](https://consilium.app/developers/api-updates) for notifications.

## Support and Feedback

For API support:
- Developer Documentation: [https://consilium.app/developers](https://consilium.app/developers)
- API Status: [https://status.consilium.app](https://status.consilium.app)
- Developer Forum: [https://forum.consilium.app](https://forum.consilium.app)
- Support Email: api-support@consilium.app 