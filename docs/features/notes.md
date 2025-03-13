# Note Management

## Overview

Note management is a core feature of Consilium, enabling users to capture and organize their thoughts, information, and knowledge. Powered by BlockNotes editor, the note system is designed to be both flexible and powerful, with integrated task management capabilities.

## Key Features

### Rich Text Editing

- **Block-Based Editor**: Uses BlockNotes for structured content organization
- **Rich Formatting**: Support for headings, lists, code blocks, quotes, etc.
- **Markdown Support**: Edit in markdown or rich text mode
- **Media Embedding**: Support for images, videos, and other media types
- **Task Blocks**: Specialized blocks for creating and managing tasks within notes

### Organization

- **Flat Structure**: Notes are organized in a flat collection rather than hierarchically
- **Tagging System**: Apply multiple tags to notes for powerful categorization and filtering
- **Linking**: Create bidirectional links between notes for a personal knowledge graph
- **Search**: Powerful search functionality across all notes
- **RAG-Powered Discovery**: AI helps surface relevant notes based on content and context
- **Pin System**: Pin important notes for quick access

### AI-Enhanced Capabilities

- **Smart Suggestions**: AI-generated content suggestions while writing
- **Auto-Tagging**: Recommendations for tags based on note content
- **Content Summarization**: AI-generated summaries of longer notes
- **Question Answering**: Ask questions about your notes with contextual answers
- **Related Notes**: AI-powered suggestions for related notes while viewing or editing

### Capture Methods

- **Direct Input**: Traditional typing within the editor
- **Voice-to-Text**: Dictate notes using speech recognition
- **Image Capture**: Take photos of handwritten notes for automatic transcription
- **Web Clipping**: Save content from websites with source attribution

## Technical Implementation

### Data Model

```
Note {
  id: UUID
  title: String
  content: JSON (BlockNotes format, including task blocks)
  created_at: Timestamp
  updated_at: Timestamp
  user_id: UUID
  tags: String[] (array of tag IDs)
  linked_notes: UUID[] (array of linked note IDs)
  is_pinned: Boolean
  is_archived: Boolean
  metadata: JSON
}

Tag {
  id: UUID
  name: String
  color: String
  user_id: UUID
}
```

### Storage

- **Primary Storage**: Supabase PostgreSQL database
- **Local Cache**: DexieJS (IndexedDB wrapper) for offline access
- **Sync Mechanism**: Bi-directional sync with conflict resolution
- **Vector Embeddings**: Note content is embedded for semantic search and RAG
- **Task Indexing**: Task blocks are indexed separately for efficient aggregation

### BlockNotes Integration

- **Custom Extensions**: Extending BlockNotes with Consilium-specific blocks
- **Task Block Type**: Specialized block type for tasks with interactive elements
- **Collaboration**: Future support for real-time collaboration
- **Version History**: Track changes and revert to previous versions

## Integrated Task Management

Tasks in Consilium are implemented as specialized blocks within notes:

- **Task Blocks**: Create interactive task blocks directly within notes
- **Rich Metadata**: Each task block contains status, due date, priority, etc.
- **Dynamic Aggregation**: Task blocks from all notes are aggregated into task-specific views
- **Bidirectional Syncing**: Changes to task blocks sync between notes and task views
- **Context Preservation**: Tasks always maintain their context within the source note

## User Experience

### Creating a Note

1. User can create a note from:
   - The notes dashboard
   - The global command bar (⌘+N)
   - Within the chat by asking an agent
   - Converting a chat conversation to a note

### Note Discovery

- **Tag-Based Views**: Filter notes by single or multiple tags
- **Full-Text Search**: Search across all note content
- **Semantic Search**: Find notes with similar meaning even with different wording
- **Recently Accessed**: Quick access to recently viewed or edited notes
- **AI-Suggested Related Notes**: Contextually relevant notes suggested by the AI
- **Pinned Notes**: Quick access to frequently used notes

### Working with Tasks in Notes

- **Inline Creation**: Create tasks directly within notes while writing
- **Interactive Elements**: Check off tasks, set due dates, and update status directly in notes
- **Task Extraction**: View aggregated tasks across all notes in dedicated task views
- **Status Synchronization**: Task status updates in either notes or task views are bidirectionally synced

### Editing Experience
- **Backlinks Panel**: See which notes link to the current note

### Mobile Experience

- **Responsive Design**: Optimized for all device sizes
- **Offline Editing**: Full functionality without internet connection
- **Quick Capture**: Streamlined input methods for mobile devices

## Integration with Other Systems

- **Planning**: Notes can be referenced in planning sessions
- **RAG System**: Notes content is indexed for retrieval-augmented generation
- **Handwriting Recognition**: Improved OCR for handwritten notes
- **Cross-Device Sync**: Seamless experience across all user devices