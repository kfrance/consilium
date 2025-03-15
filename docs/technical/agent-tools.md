# Agent Tools

## Overview

The Consilium agent system is built around a tool-based architecture where specialized AI agents have access to various tools that enable them to perform specific actions. These tools provide the bridge between AI agents and system functionality, allowing agents to manipulate data, interact with external services, and take actions on behalf of the user.

Tools are implemented as function-calling interfaces that agents can use, with appropriate permissions and validation. Each tool is designed for a specific purpose and is made available to agents based on their role and the context of the user's request.

## Tool Categories

Tools in Consilium are organized into the following categories:

### Note Management Tools

These tools allow agents to work with notes in the system.

| Tool Name | Description | Used By |
|-----------|-------------|---------|
| `noteCreate` | Creates a new note with specified content | Note Agent, Save Command Agent |
| `noteUpdate` | Updates an existing note | Note Agent |
| `noteDelete` | Deletes a note | Note Agent |
| `noteSearch` | Searches across notes with semantic or keyword search | Knowledge Agent, Note Agent |
| `noteLink` | Creates links between notes | Knowledge Agent |

### Task Management Tools

These tools allow agents to work with tasks.

| Tool Name | Description | Used By |
|-----------|-------------|---------|
| `taskCreate` | Creates a new task | Task Agent |
| `taskUpdate` | Updates an existing task | Task Agent |
| `taskComplete` | Marks a task as completed | Task Agent |
| `taskSearch` | Searches for tasks based on various criteria | Task Agent, Planning Agent |

### External Integration Tools

These tools enable integration with external systems.

| Tool Name | Description | Used By |
|-----------|-------------|---------|
| `emailScan` | Scans emails for task-like content | Email Agent |
| `emailReply` | Composes and sends replies to emails | Email Agent |
| `clickUpSync` | Synchronizes tasks with ClickUp | ClickUp Agent |
| `webScrape` | Extracts and processes website content using Crawl4AI | Research Agent |
| `composioTool` | Generic interface to Composio integrations | Integration Agent |

### Planning Tools

These tools support planning and goal management.

| Tool Name | Description | Used By |
|-----------|-------------|---------|
| `goalCreate` | Creates a new goal | Planning Agent |
| `goalUpdate` | Updates an existing goal | Planning Agent |
| `planningSession` | Initiates a guided planning session | Planning Agent |
| `reflectionCreate` | Creates a reflection entry | Reflection Agent |

### Utility Tools

These tools provide various utility functions.

| Tool Name | Description | Used By |
|-----------|-------------|---------|
| `webSearch` | Searches the web for information | Research Agent |
| `systemCommand` | Executes a system command (restricted) | System Agent |
| `fileReader` | Reads files from the user's storage | File Agent |
| `imageGeneration` | Generates images based on descriptions | Creative Agent |

## Command-Tool Mapping

Chat commands in Consilium map to specific tools that agents use to fulfill the command. The following table shows the relationship between commands and tools:

| Command | Primary Tool | Supporting Tools | Agent |
|---------|--------------|------------------|-------|
| `/save` | `noteCreate` | `noteSearch` | Save Command Agent |
| `/task` | `taskCreate` | `taskSearch` | Task Agent |
| `/email` | `emailScan` | `taskCreate` | Email Agent |
| `/email-reply` | `emailReply` | `taskSearch` | Email Agent |
| `/remind` | `reminderCreate` | `reminderSearch` | Reminder Agent |
| `/plan` | `planningSession` | `goalCreate`, `taskCreate` | Planning Agent |
| `/scrape` | `webScrape` | `noteCreate`, `webSearch` | Research Agent |

**Note about `/scrape` command vs `webScrape` tool**: While the `/scrape` command uses the `webScrape` tool, the command implementation is simplified to only add content to the chat context. The full `webScrape` tool available to agents has additional capabilities including content analysis, metadata extraction, and automatic note creation that aren't directly exposed through the command interface.

## Tool Implementation

### Tool Interface

Each tool follows a standard interface structure:

```typescript
interface Tool {
  name: string;
  description: string;
  parameters: ToolParameter[];
  execute: (params: Record<string, any>, context: ToolContext) => Promise<ToolResult>;
  requiredPermissions: Permission[];
}

interface ToolParameter {
  name: string;
  description: string;
  type: 'string' | 'number' | 'boolean' | 'object' | 'array';
  required: boolean;
  default?: any;
  enum?: string[];
}

interface ToolContext {
  user: User;
  conversation: Conversation;
  agent: Agent;
}

interface ToolResult {
  success: boolean;
  data?: any;
  error?: string;
}
```

### Example Tool Implementation

Here's an example implementation of the `noteCreate` tool:

```typescript
const noteCreateTool: Tool = {
  name: 'noteCreate',
  description: 'Creates a new note with the specified content',
  parameters: [
    {
      name: 'title',
      description: 'The title of the note',
      type: 'string',
      required: true
    },
    {
      name: 'content',
      description: 'The content of the note in markdown format',
      type: 'string',
      required: true
    },
    {
      name: 'tags',
      description: 'Tags to associate with the note',
      type: 'array',
      required: false
    }
  ],
  execute: async (params, context) => {
    try {
      const { title, content, tags = [] } = params;
      
      // Create note in database
      const note = await noteService.createNote({
        title,
        content,
        userId: context.user.id,
        tags
      });
      
      return {
        success: true,
        data: {
          noteId: note.id,
          title: note.title,
          url: `/notes/${note.id}`
        }
      };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  },
  requiredPermissions: ['notes:create']
};
```

Here's an example implementation of the `emailReply` tool:

```typescript
const emailReplyTool: Tool = {
  name: 'emailReply',
  description: 'Composes and sends a reply to an email task',
  parameters: [
    {
      name: 'taskId',
      description: 'The ID of the email task to reply to',
      type: 'string',
      required: true
    },
    {
      name: 'content',
      description: 'The content of the reply',
      type: 'string',
      required: true
    },
    {
      name: 'subject',
      description: 'Optional custom subject line. If not provided, will use Re: original subject',
      type: 'string',
      required: false
    }
  ],
  execute: async (params, context) => {
    try {
      const { taskId, content, subject } = params;
      
      // Get task details
      const task = await taskService.getTask(taskId);
      
      // Verify it's an email task
      if (task.source !== 'email' || !task.metadata.email) {
        return {
          success: false,
          error: 'The specified task is not an email task'
        };
      }
      
      // Prepare email reply
      const emailData = {
        to: task.metadata.email.sender,
        subject: subject || `Re: ${task.metadata.email.subject}`,
        content,
        inReplyTo: task.metadata.email.messageId,
        userId: context.user.id
      };
      
      // Send the email
      const result = await emailService.sendEmail(emailData);
      
      // Update task with reply info
      await taskService.updateTask(taskId, {
        metadata: {
          ...task.metadata,
          email: {
            ...task.metadata.email,
            lastReplied: new Date(),
            replyCount: (task.metadata.email.replyCount || 0) + 1
          }
        }
      });
      
      return {
        success: true,
        data: {
          messageId: result.messageId,
          sentAt: result.sentAt
        }
      };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  },
  requiredPermissions: ['email:send']
};

const webScrapeTool: Tool = {
  name: 'webScrape',
  description: 'Extracts content from websites using Crawl4AI',
  parameters: [
    {
      name: 'url',
      description: 'The URL to scrape',
      type: 'string',
      required: true
    },
    {
      name: 'depth',
      description: 'How many levels of links to follow (default: 0)',
      type: 'number',
      required: false
    },
    {
      name: 'summarize',
      description: 'Whether to summarize the content (default: true)',
      type: 'boolean',
      required: false
    },
    {
      name: 'summaryLength',
      description: 'Target length for summary (brief, medium, detailed)',
      type: 'string',
      required: false,
      enum: ['brief', 'medium', 'detailed']
    },
    {
      name: 'maxTokens',
      description: 'Maximum number of tokens to include (default: system setting)',
      type: 'number',
      required: false
    },
    {
      name: 'saveAsNote',
      description: 'Whether to save the scraped content as a note',
      type: 'boolean',
      required: false
    },
    {
      name: 'noteTitle',
      description: 'Title for the note if saving as note',
      type: 'string',
      required: false
    },
    {
      name: 'filters',
      description: 'Filters to apply when scraping (e.g., content types, date ranges)',
      type: 'object',
      required: false
    },
    {
      name: 'analyze',
      description: 'Whether to analyze the content and extract key information',
      type: 'boolean',
      required: false
    },
    {
      name: 'analysisInstructions',
      description: 'Specific instructions for content analysis',
      type: 'string',
      required: false
    }
  ],
  execute: async (params, context) => {
    try {
      const { 
        url, 
        depth = 0, 
        summarize = true,
        summaryLength = 'medium',
        maxTokens = context.user.settings.scraping.maxTokens || 100000,
        saveAsNote = false, 
        noteTitle, 
        filters = {},
        analyze = false,
        analysisInstructions = ''
      } = params;
      
      // Validate URL
      if (!url.match(/^https?:\/\/.+/)) {
        return {
          success: false,
          error: 'Invalid URL. URL must start with http:// or https://'
        };
      }
      
      // Call Crawl4AI service
      const scrapeResult = await crawl4AIService.scrape({
        url,
        depth,
        filters,
        userId: context.user.id
      });
      
      // Process the result
      let processedContent = {
        title: scrapeResult.title || url,
        content: scrapeResult.content,
        metadata: {
          source: url,
          scrapedAt: new Date(),
          pageCount: scrapeResult.pageCount,
          wordCount: scrapeResult.wordCount,
          wasTruncated: false
        }
      };
      
      // Check if content exceeds token limit
      const tokenCount = await tokenCountService.countTokens(processedContent.content);
      if (tokenCount > maxTokens) {
        processedContent.metadata.wasTruncated = true;
        processedContent.metadata.originalTokenCount = tokenCount;
        
        if (summarize) {
          // Summarize content instead of truncating
          const summary = await contentSummaryService.summarize({
            content: processedContent.content,
            length: summaryLength,
            maxTokens: maxTokens,
            userId: context.user.id
          });
          
          processedContent.content = summary;
          processedContent.metadata.isSummary = true;
          processedContent.metadata.summaryLength = summaryLength;
        } else {
          // Truncate content to fit token limit
          processedContent.content = await tokenCountService.truncateToTokenLimit(
            processedContent.content, 
            maxTokens
          );
        }
      } else if (summarize) {
        // Summarize even if under token limit (if requested)
        const summary = await contentSummaryService.summarize({
          content: processedContent.content,
          length: summaryLength,
          userId: context.user.id
        });
        
        processedContent.content = summary;
        processedContent.metadata.isSummary = true;
        processedContent.metadata.summaryLength = summaryLength;
      }
      
      // Analyze content if requested
      let analysis = null;
      if (analyze) {
        analysis = await contentAnalysisService.analyze({
          content: processedContent.content,
          instructions: analysisInstructions,
          userId: context.user.id
        });
        
        processedContent.metadata.analysis = analysis;
      }
      
      // Save as note if requested
      let noteId = null;
      if (saveAsNote) {
        const note = await noteService.createNote({
          title: noteTitle || processedContent.title,
          content: {
            type: 'doc',
            content: [
              {
                type: 'paragraph',
                content: [
                  {
                    type: 'text',
                    text: `Source: ${url}`
                  }
                ]
              },
              ...formatContentForBlockNote(processedContent.content)
            ]
          },
          metadata: {
            source: 'web_scrape',
            originalUrl: url,
            ...processedContent.metadata
          },
          userId: context.user.id
        });
        noteId = note.id;
      }
      
      return {
        success: true,
        data: {
          ...processedContent,
          analysis,
          noteId
        }
      };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  },
  requiredPermissions: ['web:scrape', 'notes:create']
};

## Tool Access Control

Access to tools is controlled through a permission system:

1. Each tool declares the permissions it requires
2. Agents are granted specific permissions based on their role
3. The system checks permissions before allowing an agent to use a tool
4. User settings can further restrict tool access

## Future Tool Development

The tool system is designed to be extensible, allowing new tools to be added as Consilium's capabilities expand. Planned tools include:

- Calendar integration tools
- Advanced research tools
- Collaborative editing tools
- Enhanced external service integrations

For information about how users interact with these tools through commands, see the [Chat Commands](../features/chat-commands.md) documentation. 