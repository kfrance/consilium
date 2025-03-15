# Chat Commands

## Overview

Consilium provides a command-based interface within the chat window, allowing users to trigger specific actions by typing commands that start with a forward slash (`/`). These commands provide quick access to system functionality without leaving the conversational interface.

Chat commands are interpreted by the AI system and routed to the appropriate agent with the relevant tools to complete the requested action. This creates a seamless experience where complex actions can be triggered through natural, conversational syntax.

## Available Commands

### `/save [instructions]`

The `/save` command allows you to save content from the current chat as a structured note.

**Usage:**
```
/save Summarize the conclusions we made in this chat
```

**Behavior:**
1. The system recognizes the `/save` command and routes it to the appropriate agent
2. The agent uses the provided instructions to process the chat content (in this example, summarizing the conclusions)
3. The agent creates a new note with the processed content
4. The system confirms the note creation and provides a link to the saved note

**Parameters:**
- `[instructions]`: Optional instructions for the AI on how to process the chat content before saving. If not provided, the agent will use reasonable defaults to save relevant information.

**Examples:**
- `/save Create a structured summary of our discussion about project architecture`
- `/save Extract the key decisions we made and format as bullet points`
- `/save Include only the code examples we discussed`
- `/save` (without instructions, the agent will just save the note without summarizing)

**Implementation Details:**
The command is handled by an agent with access to the note creation tool, which allows it to create and save notes in the user's note collection. See [Agent Tools](../technical/agent-tools.md) for more information about the underlying implementation.

### `/remind <instructions>`

The `/remind` command allows you to create reminders for future tasks or events, including one-time and recurring reminders.

**Usage:**
```
/remind Send the project proposal to the client tomorrow at 9am
```

**Behavior:**
1. The system recognizes the `/remind` command and routes it to the appropriate agent
2. The agent analyzes the instructions to extract the reminder content and timing information
3. If the timing information is unclear or missing, the agent asks for clarification
4. Once all required information is provided, the agent creates the reminder
5. The system confirms the reminder has been set and displays the details

**Parameters:**
- `<instructions>`: Required instructions describing what to be reminded about and when. Must include both the reminder content and timing information.

**Examples:**
- `/remind Call mom every Sunday at 6pm`
- `/remind Take medication daily at 8am and 8pm`
- `/remind Review quarterly goals on the last Friday of each quarter`
- `/remind Check on project status tomorrow morning`

**Time Specification:**
The agent can understand various time formats:
- Specific times: "9am", "14:30", "tomorrow at noon"
- Relative times: "in 3 hours", "next Tuesday", "every weekday"
- Recurring patterns: "daily", "every Monday", "first of each month"

If the timing information is ambiguous, the agent will ask clarifying questions:
```
User: /remind Follow up with the team
Agent: When would you like to be reminded to follow up with the team?
User: Tomorrow afternoon
Agent: I've set a reminder for tomorrow at 2:00 PM to follow up with the team. Would you like to adjust the time?
```

**Implementation Details:**
The command is handled by a Reminder Agent with access to the reminder creation and management tools. See [Agent Tools](../technical/agent-tools.md) for more information about the underlying implementation.

### `/email-reply @<task-reference>`

The `/email-reply` command allows you to reply to an email that was previously converted into a task.

**Usage:**
```
/email-reply @ProjectProposal
```

**Behavior:**
1. The system recognizes the `/email-reply` command and routes it to the appropriate agent
2. When you type `@`, the system provides a fuzzy search interface to find the email task you want to reply to
3. Once you select the task, the agent accesses the original email metadata
4. The agent helps you compose and send a reply to the original sender
5. After sending, the system updates the task with the reply information

**Parameters:**
- `@<task-reference>`: Required reference to the email task you want to reply to. Using `@` triggers a fuzzy search to help you find the right task.

**Example Flow:**
```
User: /email-reply @client
(System shows matching tasks: @ClientFeedback, @ClientMeeting, @ClientProposal)
User selects: @ClientProposal
Agent: I'll help you reply to the email from Alice Johnson about "Project Proposal Feedback". What would you like to say?
User: Let her know we've incorporated her feedback and the updated proposal will be ready by Friday.
Agent: I've drafted a reply. Here's a preview:

To: alice.johnson@example.com
Subject: Re: Project Proposal Feedback
---------
Hi Alice,

Thank you for your feedback on the project proposal. I wanted to let you know that we've incorporated your suggestions and will have the updated proposal ready by Friday.

Best regards,
[Your Name]
---------

Would you like me to send this, edit it, or discard it?
User: Send it
Agent: Email sent successfully to Alice Johnson. The task has been updated to reflect your reply.
```

**Implementation Details:**
The command is handled by an Email Agent with access to the email reply tool, which connects to your email service to send replies. The agent can help draft professional responses based on your instructions. See [Agent Tools](../technical/agent-tools.md) for more information about the underlying implementation.

### `/scrape <url> [options]`

The `/scrape` command allows you to extract content from websites using Crawl4AI and add it to the current chat context.

**Usage:**
```
/scrape https://example.com
```

**Behavior:**
1. The system recognizes the `/scrape` command and routes it to the appropriate agent
2. The agent sends the URL to the Crawl4AI service for processing
3. Crawl4AI extracts content from the specified URL based on the options provided
4. By default, the content is summarized to preserve context space
5. The extracted content is added to the current chat context
6. The conversation continues with the AI now having access to the scraped content

**Parameters:**
- `<url>`: Required URL to scrape. Must be a valid URL including the protocol (http:// or https://).
- `[options]`: Optional parameters to customize the scraping behavior:
  - `--depth=<number>`: How many levels of links to follow (default: 0)
  - `--full`: Include full content instead of summary (may hit token limits)
  - `--no-summary`: Same as --full, include full content instead of summary

**Examples:**
- `/scrape https://example.com` - Basic scrape of a single page with automatic summarization
- `/scrape https://example.com/blog --depth=2` - Scrape the blog and follow links up to 2 levels deep
- `/scrape https://example.com --full` - Scrape without summarizing (full content)

**Token Limits:**
The command will only add up to 100,000 tokens to the context (configurable in [Settings](../technical/settings.md)). If the content exceeds this limit, it will be truncated and a warning will be displayed to the user.

**Implementation Details:**
The command is handled by a Research Agent with access to the web scraping tool, which connects to the Crawl4AI service via API. Unlike the agent-facing tool, the command simply adds the content to the chat context without analyzing it or automatically saving it as a note. Agents can use the more powerful `webScrape` tool to perform additional operations on the scraped content.

## Command Discovery

Users can discover available commands through:

1. **Autocomplete**: Typing `/` in the chat window displays a menu of available commands
2. **Command Help**: Typing `/help` displays documentation about available commands

## Future Commands

Additional commands will be added to enhance the chat interface capabilities, including commands for:

- Task management
- Planning and goal setting
- Calendar interactions
- External system integrations
- Data visualization
- And more

For technical details about how commands are implemented, please see the [Agent Tools](../technical/agent-tools.md) documentation. 