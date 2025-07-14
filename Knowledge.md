# Knowledge Base System - Diagrams & Architecture

## 1. Use Case Diagram

```mermaid
graph TB
    subgraph "Actors"
        Admin[👤 Admin User]
        Client[👤 Client User]
        AIAssist[🤖 AI Assistant]
        RagService[🔧 RAG Service]
    end

    subgraph "Knowledge Base System"
        subgraph "General Knowledge Management"
            UC1[Upload File Document]
            UC2[Add URL Content]
            UC3[Create Text Document]
            UC4[Search Knowledge Base]
            UC5[Delete Knowledge Item]
            UC6[View Knowledge Gaps]
        end
        
        subgraph "Release Notes Management"
            UC7[Create Release Note]
            UC8[Upload Release Video]
            UC9[Add Video Transcript]
            UC10[Version Management]
            UC11[Set Release Date]
            UC12[Manage Labels/Tags]
        end
        
        subgraph "RAG Operations"
            UC13[Query Context]
            UC14[Generate Embeddings]
            UC15[Retrieve Similar Content]
            UC16[Process Documents]
        end
    end

    %% Admin Use Cases
    Admin --> UC1
    Admin --> UC2
    Admin --> UC3
    Admin --> UC5
    Admin --> UC7
    Admin --> UC8
    Admin --> UC9
    Admin --> UC10
    Admin --> UC11
    Admin --> UC12

    %% Client Use Cases
    Client --> UC1
    Client --> UC2
    Client --> UC3
    Client --> UC4
    Client --> UC6

    %% AI Assistant Use Cases
    AIAssist --> UC4
    AIAssist --> UC13
    AIAssist --> UC15

    %% RAG Service Use Cases
    RagService --> UC13
    RagService --> UC14
    RagService --> UC15
    RagService --> UC16

    %% Relationships
    UC1 -.-> UC14
    UC2 -.-> UC14
    UC3 -.-> UC14
    UC7 -.-> UC14
    UC8 -.-> UC16
    UC4 -.-> UC15
    UC13 -.-> UC15
```

## 2. System Architecture Diagram

```mermaid
graph TB
    subgraph "Frontend (Vue 3 + TypeScript)"
        KBView[KnowledgeBaseView.vue]
        KBTable[KnowledgeBaseTable.vue]
        KBPopup[KnowledgeBasePopup.vue]
        RNPopup[ReleaseNotePopup.vue]
        KBGapTable[KnowledgeBaseGapTable.vue]
        KBService[KnowledgeBaseService.ts]
    end

    subgraph "Backend (NestJS + TypeScript)"
        KBController[KnowledgeBaseController]
        KBServiceBE[KnowledgeBaseService]
        RagServiceBE[RagService]
        MinioService[MinioService]
        KBEntity[KnowledgeBaseEntity]
    end

    subgraph "External Services"
        RagAPI[RAG Service<br/>ki-rag.fly.dev]
        MinioStorage[MinIO Storage<br/>File Backup]
        PostgresDB[PostgreSQL<br/>Metadata Store]
        RedisCache[Upstash Redis<br/>Caching Layer]
    end

    %% Frontend connections
    KBView --> KBTable
    KBView --> KBGapTable
    KBTable --> KBPopup
    KBTable --> RNPopup
    KBTable --> KBService
    KBPopup --> KBService
    RNPopup --> KBService

    %% Frontend to Backend
    KBService --> KBController

    %% Backend connections
    KBController --> KBServiceBE
    KBController --> RagServiceBE
    KBController --> MinioService
    KBServiceBE --> KBEntity

    %% Backend to External
    RagServiceBE --> RagAPI
    MinioService --> MinioStorage
    KBEntity --> PostgresDB
    RagAPI --> RedisCache

    %% Data Flow
    KBController -.->|Store Files| MinioStorage
    KBController -.->|Process Content| RagAPI
    KBController -.->|Save Metadata| PostgresDB
```

## 3. Data Flow Diagram - File Upload Process

```mermaid
sequenceDiagram
    participant User
    participant Frontend as Vue Frontend
    participant Backend as NestJS Backend
    participant RAG as RAG Service
    participant MinIO as MinIO Storage
    participant DB as PostgreSQL

    User->>Frontend: Upload file via drag & drop
    Frontend->>Frontend: Validate file type/size
    Frontend->>Backend: POST /knowledge-base/file
    
    Backend->>Backend: Generate UUID for storage
    Backend->>RAG: POST /upsert/{subAccountId}/file
    RAG->>RAG: Extract text, create embeddings
    RAG-->>Backend: Return source_id & metadata
    
    Backend->>MinIO: Store original file
    MinIO-->>Backend: Confirm storage
    
    Backend->>DB: Save KB metadata with source_id
    DB-->>Backend: Return saved entity
    
    Backend-->>Frontend: Return knowledge base entry
    Frontend->>Frontend: Refresh table data
    Frontend-->>User: Show success message
```

## 4. Component Relationship Diagram

```mermaid
graph TD
    subgraph "KnowledgeBaseView.vue"
        direction TB
        Header[Header with Add Button]
        Tabs[Tab Navigation]
        
        subgraph "Tab: Knowledge Bases"
            KBTable[KnowledgeBaseTable<br/>isReleaseNote: false]
        end
        
        subgraph "Tab: Release Notes"
            RNTable[KnowledgeBaseTable<br/>isReleaseNote: true]
        end
        
        subgraph "Tab: Knowledge Gaps"
            KBGaps[KnowledgeBaseGapTable]
        end
    end

    subgraph "Popup Components"
        KBPopup[KnowledgeBasePopup<br/>FILE, LINK, TEXT]
        RNPopup[ReleaseNotePopup<br/>VIDEO, TRANSCRIPT]
    end

    Header --> KBPopup
    Header --> RNPopup
    KBTable --> KBPopup
    RNTable --> RNPopup
    
    KBTable -.->|Shared Component| RNTable
```

## 5. Database Entity Relationship Diagram

```mermaid
erDiagram
    KNOWLEDGE_BASE_ENTITY {
        uuid id PK
        enum type
        string description
        string storeId
        string sourceId
        string fileName
        string linkUrl
        boolean isReleaseNote
        date releaseDate
        string releaseVersion
        string_array labels
        timestamp created
    }
    
    KB_GAP_ENTITY {
        uuid id PK
        string question
        string context
        timestamp created
    }
    
    KNOWLEDGE_BASE_ENTITY ||--o{ KB_GAP_ENTITY : "helps resolve"
```

## 6. State Management Flow

```mermaid
stateDiagram-v2
    [*] --> Loading
    Loading --> KnowledgeBaseList
    
    KnowledgeBaseList --> UploadDialog : Add Document
    UploadDialog --> FileUpload : Choose File
    UploadDialog --> URLUpload : Choose URL
    UploadDialog --> TextUpload : Choose Text
    
    FileUpload --> Processing : Submit
    URLUpload --> Processing : Submit
    TextUpload --> Processing : Submit
    
    Processing --> Success : Upload Complete
    Processing --> Error : Upload Failed
    
    Success --> KnowledgeBaseList : Refresh
    Error --> UploadDialog : Retry
    
    KnowledgeBaseList --> ViewDocument : Click Item
    ViewDocument --> KnowledgeBaseList : Close
    
    KnowledgeBaseList --> DeleteConfirm : Delete Action
    DeleteConfirm --> Deleting : Confirm
    DeleteConfirm --> KnowledgeBaseList : Cancel
    
    Deleting --> KnowledgeBaseList : Complete
```

## 7. Release Notes Workflow

```mermaid
flowchart TD
    Start([New Release]) --> CreateNote[Create Release Note]
    CreateNote --> ChooseType{Choose Content Type}
    
    ChooseType -->|Video| UploadVideo[Upload Video File]
    ChooseType -->|Transcript| UploadTranscript[Upload Transcript]
    ChooseType -->|Document| UploadDoc[Upload Document]
    
    UploadVideo --> SetMetadata[Set Release Metadata]
    UploadTranscript --> SetMetadata
    UploadDoc --> SetMetadata
    
    SetMetadata --> AddVersion[Add Version Number]
    AddVersion --> SetDate[Set Release Date]
    SetDate --> AddLabels[Add Labels/Tags]
    
    AddLabels --> ProcessRAG[Process with RAG Service]
    ProcessRAG --> StoreMinIO[Store in MinIO]
    StoreMinIO --> SaveDB[Save to Database]
    
    SaveDB --> Published[Release Note Published]
    Published --> End([Complete])
```

## 8. RAG Query Process

```mermaid
sequenceDiagram
    participant AI as AI Assistant
    participant Backend as NestJS Backend
    participant RAG as RAG Service
    participant Cache as Redis Cache
    participant Vector as Vector DB

    AI->>Backend: Query with context request
    Backend->>Backend: Extract subAccountId & labels
    Backend->>Cache: Check cached results
    
    alt Cache Hit
        Cache-->>Backend: Return cached data
    else Cache Miss
        Backend->>RAG: POST /query/{subAccountId}
        RAG->>Vector: Search similar embeddings
        Vector-->>RAG: Return matching chunks
        RAG->>RAG: Rank by relevance score
        RAG-->>Backend: Return context + sources
        Backend->>Cache: Store results
    end
    
    Backend-->>AI: Return contextual information
    AI->>AI: Generate informed response
```

## 9. Multi-Tenant Data Isolation

```mermaid
graph TB
    subgraph "Tenant A"
        UserA[User A]
        KBA[Knowledge Base A]
        RAGA[RAG Namespace A]
    end
    
    subgraph "Tenant B"
        UserB[User B]
        KBB[Knowledge Base B]
        RAGB[RAG Namespace B]
    end
    
    subgraph "Shared Infrastructure"
        Frontend[Vue Frontend]
        Backend[NestJS Backend]
        RagService[RAG Service]
        MinIO[MinIO Storage]
        DB[PostgreSQL with RLS]
    end
    
    UserA --> Frontend
    UserB --> Frontend
    Frontend --> Backend
    Backend --> RagService
    Backend --> MinIO
    Backend --> DB
    
    Backend -.->|Tenant A Context| RAGA
    Backend -.->|Tenant B Context| RAGB
    
    RAGA --> KBA
    RAGB --> KBB
```

## 10. File Type Processing Matrix

```mermaid
graph LR
    subgraph "Input Types"
        PDF[📄 PDF Files]
        DOC[📝 Word Docs]
        TXT[📋 Text Files]
        URL[🌐 Web URLs]
        VIDEO[🎥 Video Files]
        AUDIO[🎵 Audio Files]
    end
    
    subgraph "Processing Pipeline"
        Extract[Text Extraction]
        Chunk[Text Chunking]
        Embed[Create Embeddings]
        Store[Vector Storage]
    end
    
    subgraph "Storage Layers"
        MinIOStore[MinIO<br/>Original Files]
        VectorDB[Vector DB<br/>Embeddings]
        PostgresDB[PostgreSQL<br/>Metadata]
    end
    
    PDF --> Extract
    DOC --> Extract
    TXT --> Extract
    URL --> Extract
    VIDEO --> Extract
    AUDIO --> Extract
    
    Extract --> Chunk
    Chunk --> Embed
    Embed --> Store
    
    Store --> VectorDB
    Extract --> MinIOStore
    Embed --> PostgresDB
```

## 11. Knowledge Gap Detection Flow

```mermaid
graph TD
    Start[User Asks Question] --> CheckKB{Knowledge Base<br/>Has Answer?}
    
    CheckKB -->|Yes| ProvideAnswer[Provide Answer from KB]
    CheckKB -->|No| LogGap[Log Knowledge Gap]
    
    LogGap --> AnalyzeGap[Analyze Question Context]
    AnalyzeGap --> SuggestContent[Suggest Content Creation]
    SuggestContent --> AdminNotify[Notify Admin]
    
    AdminNotify --> CreateContent{Admin Creates<br/>Content?}
    CreateContent -->|Yes| AddToKB[Add to Knowledge Base]
    CreateContent -->|No| MarkResolved[Mark Gap as Reviewed]
    
    AddToKB --> UpdateRAG[Update RAG Index]
    UpdateRAG --> ProvideAnswer
    
    ProvideAnswer --> End[Complete]
    MarkResolved --> End
```

This comprehensive set of diagrams illustrates the Knowledge Base system's architecture, data flows, user interactions, and integration points with the RAG service and Release Notes functionality. The system demonstrates a sophisticated multi-tenant architecture that supports both general knowledge management and specialized release note handling through a unified interface.
