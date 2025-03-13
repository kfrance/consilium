# Data Storage Design

## Overview

Consilium employs a hybrid data storage strategy that combines cloud-based PostgreSQL (via Supabase), client-side storage (via DexieJS/IndexedDB), and specialized vector storage for AI features. This approach balances reliability, performance, offline capabilities, and specialized AI storage needs.

## Storage Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Client Application                           │
│                                                                     │
│  ┌───────────────────┐   ┌───────────────────┐   ┌───────────────┐  │
│  │     DexieJS       │   │  Local Storage    │   │ Cache Storage │  │
│  │    (IndexedDB)    │   │                   │   │               │  │
│  └────────┬──────────┘   └─────────┬─────────┘   └───────┬───────┘  │
│           │                        │                     │          │
└───────────┼────────────────────────┼─────────────────────┼──────────┘
            │                        │                     │
            ▼                        ▼                     ▼
┌───────────────────────────────────────────────────────────────────────┐
│                        Sync Layer                                     │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │         Conflict Resolution & Synchronization Logic             │  │
│  └─────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────┬─────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                           API Layer                                 │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     Server-Side Storage                             │
│                                                                     │
│  ┌───────────────────┐   ┌───────────────────┐   ┌───────────────┐  │
│  │     Supabase      │   │  Vector Database  │   │ File Storage  │  │
│  │    PostgreSQL     │   │                   │   │               │  │
│  └───────────────────┘   └───────────────────┘   └───────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

## Primary Data Storage (Supabase PostgreSQL)

Supabase provides a PostgreSQL database as the primary storage solution for structured data.

### Key Characteristics

- **Relational Data Model**: Structured storage with enforced relationships
- **ACID Compliance**: Ensures data integrity and consistency
- **Managed Service**: Reduces operational overhead
- **Real-time Capabilities**: Supports real-time data subscriptions
- **Row-Level Security**: Fine-grained access control directly in the database

### Data Organization

- **Core Tables**: Users, notes, tasks, goals, etc. (see database schema in backend documentation)
- **Junction Tables**: For many-to-many relationships
- **History Tables**: For tracking changes to important entities
- **Metadata Tables**: For system configuration and preferences

### Performance Considerations

- **Indices**: Strategic indices on frequently queried columns
- **Materialized Views**: For complex, frequently-accessed data
- **Connection Pooling**: Efficient database connection management
- **Query Optimization**: Carefully crafted queries for performance

## Client-Side Storage (DexieJS/IndexedDB)

DexieJS provides a wrapper around the browser's IndexedDB API for client-side storage.

### Key Characteristics

- **Offline-First**: Application works without internet connection
- **Large Data Storage**: Can store significant amounts of data client-side
- **Structured Data**: Supports complex data structures and indices
- **Transaction Support**: ACID transactions for reliable operations
- **Observable Queries**: React to data changes in real-time

### Data Organization

```javascript
// Example Dexie database schema
const db = new Dexie('ConsiliumDB');
db.version(1).stores({
  notes: '++id, title, content, user_id, parent_id, created_at, updated_at, is_synced',
  tasks: '++id, title, description, status, priority, due_date, user_id, is_synced',
  goals: '++id, title, timeframe, parent_goal_id, progress, user_id, is_synced',
  offlineChanges: '++id, entity_type, entity_id, operation, data, timestamp'
});
```

### Sync Strategy

1. **Optimistic Updates**: UI updates immediately while changes are queued
2. **Background Sync**: Changes are synced when online
3. **Conflict Resolution**: Strategy for handling conflicting changes
4. **Retry Mechanism**: Failed sync attempts are retried with exponential backoff
5. **Sync Status Indicators**: UI feedback on sync status

## Vector Storage (Qdrant/Pinecone)

A specialized vector database for storing and retrieving embeddings for the RAG system.

### Key Characteristics

- **Vector-Based**: Optimized for high-dimensional vector storage
- **Similarity Search**: Efficient nearest-neighbor search algorithms
- **Metadata Filtering**: Filter vectors based on associated metadata
- **Scalability**: Handles large volumes of vectors efficiently
- **Cloud-Hosted**: Managed service reduces operational overhead

### Data Organization

- **Collections**: Organized by document type or domain
- **Points**: Vector embeddings with associated metadata
- **Payload**: Additional data associated with each vector
- **Indices**: Optimized for fast similarity search

### Integration with RAG System

1. **Embedding Generation**: Text chunks are converted to vector embeddings
2. **Vector Storage**: Embeddings are stored with metadata in the vector database
3. **Similarity Search**: User queries are converted to embeddings and matched
4. **Result Filtering**: Results are filtered based on metadata criteria
5. **Context Assembly**: Retrieved chunks are assembled into a coherent context

## Cache Storage

Browser's Cache Storage API is used for caching static assets and API responses.

### Key Characteristics

- **Fast Access**: Optimized for quick retrieval
- **Controlled Expiration**: TTL-based cache invalidation
- **Service Worker Integration**: Works with PWA offline capabilities
- **Size Limitations**: Limited by browser quotas

### Caching Strategy

- **Static Assets**: Long-lived cache for static resources (JS, CSS, images)
- **API Responses**: Short-lived cache for frequently-accessed data
- **Stale-While-Revalidate**: Show cached data while fetching fresh data

## Additional Storage Mechanisms

### Local Storage

Used for small, frequently-accessed data:

- **User Preferences**: Theme, language, UI settings
- **Auth Tokens**: Short-lived authentication tokens
- **Feature Flags**: User-specific feature enablement

### Session Storage

Used for temporary, session-scoped data:

- **Session State**: Current UI state, open panels, etc.
- **Temporary Drafts**: In-progress content that doesn't need to persist
- **Wizard Progress**: Multi-step process state

### File Storage (Supabase Storage)

Used for storing user-uploaded files:

- **Document Attachments**: Files attached to notes or tasks
- **Profile Images**: User avatars and profile photos
- **Exported Data**: Generated reports or exports

## Data Lifecycle Management

### Creation

1. Data is created in the client application
2. Stored immediately in client-side storage
3. Synced to server-side storage when online
4. Indexed for search and retrieval as needed

### Retrieval

1. Check client-side cache/storage first
2. Fall back to server request if not available locally
3. Update local cache with retrieved data
4. Notify UI of data availability

### Update

1. Apply optimistic update to local data
2. Record change in sync queue
3. Synchronize with server when online
4. Resolve conflicts if any

### Deletion

1. Mark as deleted locally
2. Queue deletion for server synchronization
3. Apply server-side deletion when online
4. Periodically clean up deleted items

## Security Considerations

### Client-Side Data Protection

- **Sensitive Data Encryption**: Encrypt sensitive data before storing
- **Storage Quotas**: Monitor and respect browser storage limits
- **Clear on Logout**: Option to clear sensitive data on logout
- **Data Validation**: Validate data before storing locally

### Server-Side Data Protection

- **Row-Level Security**: Supabase RLS for access control
- **Data Encryption**: Encryption at rest for sensitive data
- **Audit Logging**: Track access and changes to sensitive data
- **Backup Strategy**: Regular backups with point-in-time recovery

## Performance Optimization

### Data Fetching Strategies

- **Pagination**: Fetch data in manageable chunks
- **Lazy Loading**: Load data only when needed
- **Prefetching**: Anticipate user needs and fetch ahead
- **Caching**: Intelligent caching of frequently accessed data

### Storage Efficiency

- **Data Compression**: Compress large text fields when appropriate
- **Binary Storage**: Use efficient formats for binary data
- **Normalization**: Proper database normalization to reduce redundancy
- **JSON Storage**: Use JSONB for flexible schema needs

## Future Enhancements

- **End-to-End Encryption**: Optional E2EE for sensitive user data
- **Improved Conflict Resolution**: More sophisticated merge strategies
- **Distributed Cache**: Shared cache for multi-user scenarios
- **Advanced Query Capabilities**: More powerful querying options
- **Enhanced Offline Experience**: More capabilities while offline
- **Cross-Device Sync**: Direct device-to-device synchronization 