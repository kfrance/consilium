# Note Management

## Overview

Note management is a core feature of Consilium, enabling users to capture and organize their thoughts, information, and knowledge. Powered by BlockNotes editor, the note system is designed to be both flexible and powerful.

## Key Features

### Rich Text Editing

- **Block-Based Editor**: Uses BlockNotes for structured content organization
- **Rich Formatting**: Support for headings, lists, code blocks, quotes, etc.
- **Markdown Support**: Edit in markdown or rich text mode
- **Media Embedding**: Support for images, videos, and other media types

### Organization

- **Flat Structure**: Notes are organized in a flat collection rather than hierarchically
- **Tagging System**: Apply multiple tags to notes for powerful categorization and filtering
- **Linking**: Create bidirectional links between notes for a personal knowledge graph
- **Search**: Powerful search functionality across all notes
- **RAG-Powered Discovery**: AI helps surface relevant notes based on content and context

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
  content: JSON (BlockNotes format)
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

### BlockNotes Integration

- **Custom Extensions**: Extending BlockNotes with Consilium-specific blocks
- **Collaboration**: Future support for real-time collaboration
- **Version History**: Track changes and revert to previous versions

## Todo Integration

Notes can contain embedded todo items with the following properties:

- **Inline Tasks**: Create tasks directly within notes
- **Task Extraction**: Automatically identify and extract tasks from notes
- **Bi-directional Linking**: Changes to tasks in the todo system reflect in notes

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

### Editing Experience

- **Distraction-Free Mode**: Focus mode for writing without UI clutter
- **Split View**: Edit and preview simultaneously
- **Templates**: Pre-defined templates for common note types
- **Backlinks Panel**: See which notes link to the current note

### Mobile Experience

- **Responsive Design**: Optimized for all device sizes
- **Offline Editing**: Full functionality without internet connection
- **Quick Capture**: Streamlined input methods for mobile devices

## Integration with Other Systems

- **Planning**: Notes can be referenced in planning sessions
- **RAG System**: Notes content is indexed for retrieval-augmented generation
- **Export Options**: Markdown, PDF, HTML for sharing
- **Calendar Integration**: Link notes to calendar events

## Future Enhancements

- **Handwriting Recognition**: Improved OCR for handwritten notes
- **AI-Generated Visualizations**: Create diagrams and charts from text descriptions
- **Cross-Device Sync**: Seamless experience across all user devices
- **Advanced Collaboration**: Share and co-edit notes with other users
- **Graph Visualization**: Visual representation of note connections and relationships 