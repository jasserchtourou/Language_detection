# KI-RAG & KI-Parser-Video System - AI-Focused Architecture

## 1. Use Case Diagram - AI Services Focus

```mermaid
graph TB
    subgraph "Actors"
        Admin[👤 Admin User]
        Client[👤 Client User]
        AIAssist[🤖 AI Assistant]
   ## 6. AI Processing Flow - KI-RAG & KI-Pars## 7. KI-RAG Internal Architecturer-Videoend

    s## 8. KI-Parser-Video Internal Architecturebgraph "AI Core Services## 9. AI Query Flow - Smart Context Re## 10. Release Notes AI Workflow ## 11. AI Services Integration & Data Flow KI-Parser-Video Focusrieval
        subgraph "KI-RAG Service (Python)"
            UC1[Process Documents]
            UC2[Generate Embeddings]
            UC3[Vector Search]
            UC4[Text Extraction]
            UC5[Query Context]
        end
        
        subgraph "KI-Parser-Video Service (Python)"
            UC6[Parse Video Content]
            UC7[Extract Release Metadata]
            UC8[Version Detection]
            UC9[Generate Release Notes]
            UC10[Content Analysis]
        end
        
        subgraph "UI Operations"
            UC11[Upload Content]
            UC12[Search Knowledge Base]
            UC13[Manage Release Notes]
        end
    end

    %% User interactions with AI services
    Admin --> UC11
    Admin --> UC13
    Client --> UC11
    Client --> UC12
    AIAssist --> UC5
    AIAssist --> UC3

    %% AI service interactions
    UC11 -.-> UC1
    UC11 -.-> UC6
    UC1 -.-> UC2
    UC6 -.-> UC7
    UC7 -.-> UC8
    UC8 -.-> UC9
    UC12 -.-> UC3
    UC5 -.-> UC3
    UC9 -.-> UC2
```

## 5. System Architecture - AI Services Focus

```mermaid
graph TB
    subgraph "Frontend Layer - Vue 3 TypeScript"
        KBView[KnowledgeBaseView.vue]
        KBTable[KnowledgeBaseTable.vue]
        KBPopup[KnowledgeBasePopup.vue]
        RNPopup[ReleaseNotePopup.vue]
        KBService[KnowledgeBaseService.ts]
    end

    subgraph "Backend Layer - NestJS TypeScript"
        KBController[KnowledgeBaseController]
        KBServiceBE[KnowledgeBaseService]
        RagClient[RAG Service Client]
        MinioService[MinioService]
        KBEntity[KnowledgeBaseEntity]
    end

    subgraph "AI Core Services - Python on Fly.io"
        subgraph "KI-RAG Service"
            RagAPI[FastAPI Server]
            TextProcessor[Text Processing Engine]
            EmbeddingGen[Embedding Generator]
            VectorSearch[Vector Search Engine]
            DocumentParser[Document Parser]
        end
        
        subgraph "KI-Parser-Video Service"
            VideoAPI[Video Parser API]
            VideoProcessor[Video Content Analyzer]
            MetadataExtractor[Metadata Extractor]
            VersionDetector[Version Detector]
            ReleaseGenerator[Release Notes Generator]
        end
    end

    subgraph "Storage Infrastructure"
        MinioStorage[MinIO File Storage]
        PostgresDB[PostgreSQL Metadata]
        RedisCache[Upstash Redis Cache]
        VectorDB[Vector Database]
    end

    %% Frontend to Backend
    KBView --> KBTable
    KBTable --> KBService
    KBService --> KBController

    %% Backend to AI Services
    KBController --> RagClient
    RagClient --> RagAPI
    KBController --> VideoAPI

    %% AI Service Internal Flow
    RagAPI --> TextProcessor
    TextProcessor --> EmbeddingGen
    EmbeddingGen --> VectorSearch
    VideoAPI --> VideoProcessor
    VideoProcessor --> MetadataExtractor
    MetadataExtractor --> VersionDetector
    VersionDetector --> ReleaseGenerator
    ReleaseGenerator --> RagAPI

    %% Storage Connections
    RagAPI --> VectorDB
    RagAPI --> RedisCache
    VideoAPI --> RedisCache
    KBController --> MinioStorage
    KBServiceBE --> PostgresDB
```

## 3. Frontend Architecture - Vue 3 TypeScript Knowledge Base Interface

```mermaid
graph TD
    subgraph "Vue 3 Frontend Application"
        subgraph "Main Application Structure"
            App[App.vue<br/>Root Component]
            Router[Vue Router<br/>Navigation Management]
            I18n[Vue I18n<br/>Multi-language Support]
            Pinia[Pinia Store<br/>State Management]
        end
        
        subgraph "Knowledge Base Views"
            KBView[KnowledgeBaseView.vue<br/>Main KB Interface]
            KBTable[KnowledgeBaseTable.vue<br/>Content Listing]
            
            subgraph "Upload Components"
                KBPopup[KnowledgeBasePopup.vue<br/>File/Text/URL Upload]
                RNPopup[ReleaseNotePopup.vue<br/>Video/Transcript Upload]
            end
            
            subgraph "Content Display"
                AudioPlayer[AudioPlayer.vue<br/>Audio Playback]
                VideoPlayer[Video Player Component]
                DocumentViewer[Document Viewer]
            end
        end
        
        subgraph "Service Layer"
            KBService[KnowledgeBaseService.ts<br/>API Client]
            HTTPClient[HTTP Client<br/>Axios + Interceptors]
            AuthService[Authentication Service<br/>Zitadel Integration]
        end
        
        subgraph "UI Components Library"
            ElementPlus[Element Plus<br/>UI Components]
            TailwindCSS[Tailwind CSS<br/>Styling Framework]
            VueIcons[Vue Icons<br/>Icon Library]
        end
    end

    subgraph "Backend Integration Points"
        NestJSAPI[NestJS API Gateway]
        KIRAGService[KI-RAG Service]
        KIParserService[KI-Parser-Video Service]
    end

    %% App structure
    App --> Router
    App --> I18n
    App --> Pinia
    Router --> KBView
    
    %% Knowledge Base flow
    KBView --> KBTable
    KBTable --> KBPopup
    KBTable --> RNPopup
    KBTable --> AudioPlayer
    KBTable --> DocumentViewer
    
    %% Service connections
    KBPopup --> KBService
    RNPopup --> KBService
    KBTable --> KBService
    KBService --> HTTPClient
    HTTPClient --> AuthService
    
    %% UI styling
    KBView --> ElementPlus
    KBTable --> TailwindCSS
    KBPopup --> VueIcons
    
    %% API connections
    HTTPClient --> NestJSAPI
    NestJSAPI -.->|Process Documents| KIRAGService
    NestJSAPI -.->|Process Videos| KIParserService
```

## 4. Frontend User Journey & Interaction Flow

```mermaid
sequenceDiagram
    participant User
    participant KBView as KnowledgeBaseView.vue
    participant KBTable as KnowledgeBaseTable.vue
    participant KBPopup as Upload Popup
    participant KBService as KnowledgeBaseService.ts
    participant NestJS as NestJS Backend
    participant KIRAG as KI-RAG Service
    participant KIParser as KI-Parser-Video

    User->>KBView: Navigate to Knowledge Base
    KBView->>KBTable: Load knowledge base content
    KBTable->>KBService: fetchKnowledgeBases()
    KBService->>NestJS: GET /knowledge-base
    NestJS-->>KBService: Return KB list
    KBService-->>KBTable: Display content list
    KBTable-->>User: Show knowledge base items
    
    User->>KBView: Click "Add Document" 
    KBView->>KBPopup: Open upload dialog
    
    alt Regular Document Upload
        User->>KBPopup: Upload file/text/URL
        KBPopup->>KBService: createFileKnowledgeBase()
        KBService->>NestJS: POST /knowledge-base/file
        NestJS->>KIRAG: Process document
        KIRAG-->>NestJS: Return processed data
        NestJS-->>KBService: Return success
    else Video/Release Note Upload
        User->>KBPopup: Upload video content
        KBPopup->>KBService: createReleaseNote()
        KBService->>NestJS: POST /knowledge-base/release-note
        NestJS->>KIParser: Process video
        KIParser->>KIRAG: Store processed content
        KIRAG-->>KIParser: Confirm storage
        KIParser-->>NestJS: Return release note
        NestJS-->>KBService: Return success
    end
    
    KBService-->>KBPopup: Show success message
    KBPopup->>KBTable: Refresh content list
    KBTable-->>User: Display updated list
```

## 5. AI Processing Flow - KI-RAG & KI-Parser-Video

```mermaid
sequenceDiagram
    participant User
    participant Vue as Vue Frontend
    participant NestJS as NestJS Backend
    participant KIRAG as KI-RAG Service
    participant KIParser as KI-Parser-Video
    participant Vector as Vector DB
    participant Redis as Redis Cache
    participant MinIO as MinIO Storage

    User->>Vue: Upload content (file/video/text)
    Vue->>NestJS: POST /knowledge-base/upload
    
    alt Regular Document Processing
        NestJS->>KIRAG: Process document
        KIRAG->>KIRAG: Extract text content
        KIRAG->>KIRAG: Clean and normalize text
        KIRAG->>KIRAG: Create semantic chunks
        KIRAG->>KIRAG: Generate embeddings
        KIRAG->>Vector: Store embeddings
        KIRAG->>Redis: Cache processed results
        KIRAG-->>NestJS: Return source_id and metadata
    else Video/Release Note Processing
        NestJS->>KIParser: Process video content
        KIParser->>KIParser: Analyze video/transcript
        KIParser->>KIParser: Extract metadata
        KIParser->>KIParser: Detect version information
        KIParser->>KIParser: Generate release notes
        KIParser->>KIRAG: Send processed content to RAG
        KIRAG->>KIRAG: Create embeddings for release content
        KIRAG->>Vector: Store release embeddings
        KIParser->>Redis: Cache release data
        KIParser-->>NestJS: Return processed release note
    end
    
    NestJS->>MinIO: Store original file
    NestJS-->>Vue: Return success with metadata
    Vue-->>User: Show upload confirmation
```

## 6. KI-RAG Internal Architecture

```mermaid
graph TD
    subgraph "KI-RAG Service Core Components"
        API[FastAPI Application]
        
        subgraph "Document Processing Pipeline"
            FileHandler[File Handler]
            TextExtractor[Text Extractor]
            ContentCleaner[Content Cleaner]
            ChunkProcessor[Semantic Chunker]
        end
        
        subgraph "Embedding & Search Engine"
            EmbeddingModel[Embedding Model]
            VectorProcessor[Vector Processor] 
            SearchEngine[Similarity Search]
            RankingSystem[Relevance Ranking]
        end
        
        subgraph "Storage Interface"
            VectorStore[Vector Store Interface]
            CacheManager[Redis Cache Manager]
            MetadataStore[Metadata Handler]
        end
    end

    subgraph "External Dependencies"
        OpenAI[OpenAI API]
        HuggingFace[HuggingFace Models]
        VectorDB[Vector Database]
        Redis[Redis Cache]
    end

    API --> FileHandler
    FileHandler --> TextExtractor
    TextExtractor --> ContentCleaner
    ContentCleaner --> ChunkProcessor
    ChunkProcessor --> EmbeddingModel
    EmbeddingModel --> VectorProcessor
    VectorProcessor --> VectorStore
    
    API --> SearchEngine
    SearchEngine --> VectorStore
    SearchEngine --> RankingSystem
    
    VectorStore --> VectorDB
    CacheManager --> Redis
    EmbeddingModel --> OpenAI
    EmbeddingModel --> HuggingFace
```

## 7. KI-Parser-Video Internal Architecture

```mermaid
graph TD
    subgraph "KI-Parser-Video Service Core"
        VideoAPI[FastAPI Video Parser]
        
        subgraph "Video Processing Pipeline"
            VideoIngester[Video File Ingester]
            TranscriptExtractor[Transcript Extractor]
            ContentAnalyzer[Content Analyzer]
            StructureParser[Structure Parser]
        end
        
        subgraph "Release Notes Intelligence"
            VersionDetector[Version Detector]
            ChangeAnalyzer[Change Analyzer]
            FeatureExtractor[Feature Extractor]
            NotesGenerator[Notes Generator]
        end
        
        subgraph "Metadata Processing"
            MetadataExtractor[Metadata Extractor]
            TagGenerator[Tag Generator]
            CategoryClassifier[Category Classifier]
            TimestampProcessor[Timestamp Processor]
        end
        
        subgraph "Output Formatting"
            TemplateEngine[Template Engine]
            MarkdownGenerator[Markdown Generator]
            StructuredOutput[Structured Output]
        end
    end

    subgraph "External Services"
        VideoProcessingAPI[Video Processing API]
        NLPModels[NLP Models]
        MLModels[ML Classification Models]
        KIRAGService[KI-RAG Service]
    end

    VideoAPI --> VideoIngester
    VideoIngester --> TranscriptExtractor
    TranscriptExtractor --> ContentAnalyzer
    ContentAnalyzer --> StructureParser
    
    StructureParser --> VersionDetector
    VersionDetector --> ChangeAnalyzer
    ChangeAnalyzer --> FeatureExtractor
    FeatureExtractor --> NotesGenerator
    
    ContentAnalyzer --> MetadataExtractor
    MetadataExtractor --> TagGenerator
    TagGenerator --> CategoryClassifier
    CategoryClassifier --> TimestampProcessor
    
    NotesGenerator --> TemplateEngine
    TemplateEngine --> MarkdownGenerator
    MarkdownGenerator --> StructuredOutput
    StructuredOutput --> KIRAGService
    
    VideoIngester --> VideoProcessingAPI
    ContentAnalyzer --> NLPModels
    VersionDetector --> MLModels
```

## 8. AI Query Flow - Smart Context Retrieval

```mermaid
sequenceDiagram
    participant AI as AI Assistant
    participant NestJS as NestJS Backend
    participant KIRAG as KI-RAG Service
    participant KIParser as KI-Parser-Video
    participant Vector as Vector Database
    participant Redis as Redis Cache

    AI->>NestJS: Query for context with user question
    NestJS->>Redis: Check for cached similar queries
    
    alt Cache Hit
        Redis-->>NestJS: Return cached context
    else Cache Miss
        NestJS->>KIRAG: POST /query with question and filters
        KIRAG->>KIRAG: Process and understand query intent
        KIRAG->>Vector: Search for similar embeddings
        Vector-->>KIRAG: Return matching content chunks
        KIRAG->>KIRAG: Rank results by relevance score
        
        alt Release Notes Specific Query
            KIRAG->>KIParser: Check for release-specific context
            KIParser->>KIParser: Filter by version/date/features
            KIParser-->>KIRAG: Return enhanced release context
        end
        
        KIRAG->>KIRAG: Combine and format final context
        KIRAG-->>NestJS: Return structured context with sources
        NestJS->>Redis: Cache results for future queries
    end
    
    NestJS-->>AI: Return contextual information
    AI->>AI: Generate informed response with context
```

## 9. Release Notes AI Workflow - KI-Parser-Video Focus

```mermaid
flowchart TD
    Start([Video/Content Upload]) --> KIParser[KI-Parser-Video Service]
    
    KIParser --> VideoAnalysis[Video Content Analysis]
    VideoAnalysis --> TranscriptExtract[Extract Transcript/Text]
    TranscriptExtract --> ContentStructure[Parse Content Structure]
    
    ContentStructure --> VersionDetect[AI Version Detection]
    VersionDetect --> ChangeAnalysis[AI Change Analysis]
    ChangeAnalysis --> FeatureDetect[Feature Detection]
    FeatureDetect --> CategoryClass[Content Categorization]
    
    CategoryClass --> MetaGeneration[Generate Metadata]
    MetaGeneration --> ReleaseFormat[Format Release Notes]
    ReleaseFormat --> QualityCheck[AI Quality Check]
    
    QualityCheck --> SendToRAG[Send to KI-RAG]
    SendToRAG --> RAGProcess[RAG Processing]
    RAGProcess --> EmbedGenerate[Generate Embeddings]
    EmbedGenerate --> VectorStore[Store in Vector DB]
    
    VectorStore --> CacheUpdate[Update Redis Cache]
    CacheUpdate --> MetadataStore[Store Metadata]
    MetadataStore --> Complete([Release Note Ready])
    
    %% Error handling
    QualityCheck -->|Issues Found| VersionDetect
    VideoAnalysis -->|Processing Error| Start
```

## 10. AI Services Integration & Data Flow

```mermaid
graph TB
    subgraph "UI Layer - Vue TypeScript"
        UI[Vue 3 Knowledge Base Interface]
    end
    
    subgraph "API Gateway - NestJS"
        Gateway[NestJS Controller]
    end
    
    subgraph "KI-RAG Service - Python AI Core"
        subgraph "Text Processing Engine"
            TextParser[Multi-format Text Parser]
            Cleaner[Content Cleaner & Normalizer]
            Chunker[Semantic Text Chunker]
        end
        
        subgraph "Embedding & Vector Engine"
            EmbedModel[Embedding Model]
            VectorEngine[Vector Processing Engine]
            SimilarityEngine[Similarity Search Engine]
        end
        
        subgraph "Query Intelligence"
            QueryProcessor[Query Understanding]
            ContextBuilder[Context Builder]
            Ranker[Relevance Ranker]
        end
    end
    
    subgraph "KI-Parser-Video - Python Release AI"
        subgraph "Video Intelligence"
            VideoParser[Video Content Parser]
            TranscriptAI[AI Transcript Analyzer]
            ContentAI[Content Intelligence Engine]
        end
        
        subgraph "Release Intelligence"
            VersionAI[AI Version Detector]
            ChangeAI[Change Detection AI]
            ReleaseAI[Release Notes AI Generator]
        end
    end
    
    subgraph "Storage Infrastructure"
        VectorDB[Vector Database]
        Redis[Redis Cache]
        MinIO[File Storage]
        Postgres[Metadata DB]
    end

    %% Main flow
    UI --> Gateway
    Gateway --> TextParser
    Gateway --> VideoParser
    
    %% KI-RAG internal flow
    TextParser --> Cleaner
    Cleaner --> Chunker
    Chunker --> EmbedModel
    EmbedModel --> VectorEngine
    VectorEngine --> VectorDB
    
    QueryProcessor --> SimilarityEngine
    SimilarityEngine --> ContextBuilder
    ContextBuilder --> Ranker
    
    %% KI-Parser-Video flow
    VideoParser --> TranscriptAI
    TranscriptAI --> ContentAI
    ContentAI --> VersionAI
    VersionAI --> ChangeAI
    ChangeAI --> ReleaseAI
    ReleaseAI --> EmbedModel
    
    %% Storage connections
    VectorEngine --> Redis
    QueryProcessor --> Redis
    VideoParser --> MinIO
    Gateway --> Postgres
```

## 11. Multi-Tenant AI Architecture with Intelligent Isolation

```mermaid
graph TB
    subgraph "Tenant A - Isolated AI Context"
        UserA[👤 User A]
        KBA[📚 Knowledge Base A]
        RNA[📝 Release Notes A]
        RAGA[🤖 KI-RAG Instance A]
        PARSERA[🎥 KI-Parser-Video A]
    end
    
    subgraph "Tenant B - Isolated AI Context"
        UserB[👤 User B]
        KBB[📚 Knowledge Base B]
        RNB[📝 Release Notes B]
        RAGB[🤖 KI-RAG Instance B]
        PARSERB[🎥 KI-Parser-Video B]
    end
    
    subgraph "Shared UI & Gateway Layer"
        Vue[Vue 3 Frontend]
        NestJS[NestJS API Gateway]
        Auth[Authentication Layer]
    end
    
    subgraph "AI Services Layer - Python on Fly.io"
        subgraph "KI-RAG Service Cluster"
            RAGCore[RAG Core Engine]
            EmbeddingCluster[Embedding Service Cluster]
            SearchCluster[Search Engine Cluster]
        end
        
        subgraph "KI-Parser-Video Service Cluster"
            VideoCore[Video Parser Core]
            AIAnalysis[AI Analysis Cluster]
            ReleaseCore[Release Generator Core]
        end
    end
    
    subgraph "Intelligent Storage Isolation"
        VectorNS[Vector DB with Namespaces]
        RedisNS[Redis with Tenant Prefixes]
        MinIOBuckets[MinIO Tenant Buckets]
        PostgresRLS[Postgres with RLS]
    end

    %% User connections
    UserA --> Vue
    UserB --> Vue
    Vue --> Auth
    Auth --> NestJS
    
    %% Tenant routing with AI context
    NestJS -.->|Tenant A Context + AI Params| RAGCore
    NestJS -.->|Tenant B Context + AI Params| RAGCore
    NestJS -.->|Tenant A Video Context| VideoCore
    NestJS -.->|Tenant B Video Context| VideoCore
    
    %% AI service processing
    RAGCore --> EmbeddingCluster
    RAGCore --> SearchCluster
    VideoCore --> AIAnalysis
    VideoCore --> ReleaseCore
    
    %% Tenant-aware AI instances
    RAGCore -.->|Namespace A| RAGA
    RAGCore -.->|Namespace B| RAGB
    VideoCore -.->|Context A| PARSERA
    VideoCore -.->|Context B| PARSERB
    
    %% Isolated data connections
    RAGA --> KBA
    RAGB --> KBB
    PARSERA --> RNA
    PARSERB --> RNB
    
    %% Storage with isolation
    EmbeddingCluster --> VectorNS
    SearchCluster --> RedisNS
    NestJS --> MinIOBuckets
    NestJS --> PostgresRLS
```

## Summary: AI-Powered Knowledge & Release Notes System

This architecture centers around **three main layers**:

### **🎨 Vue 3 TypeScript Frontend**
- **Modern component architecture** with KnowledgeBaseView.vue and specialized upload popups
- **Intelligent file handling** supporting documents, videos, and release notes
- **Real-time UI updates** with Element Plus components and Tailwind CSS styling
- **Multi-language support** with Vue I18n and enterprise authentication via Zitadel

### **🤖 KI-RAG Service (Primary AI Engine)**
- **Advanced text processing** with semantic understanding
- **Neural embedding generation** for context-aware search
- **Intelligent query processing** with relevance ranking
- **Multi-format document support** (PDF, DOC, TXT, URLs)

### **🎥 KI-Parser-Video Service (Release Notes AI)**
- **AI-powered video content analysis** and transcript extraction
- **Intelligent version detection** and change analysis
- **Automated release notes generation** with smart formatting
- **Seamless integration** with KI-RAG for knowledge storage

### **🔧 Supporting Infrastructure**
- **NestJS API Gateway** for orchestration and authentication
- **Multi-tenant isolation** with intelligent context separation
- **Optimized storage** with Vector DB, Redis caching, and file storage

The system delivers enterprise-grade AI capabilities for knowledge management and automated release documentation, with the Vue.js frontend providing an intuitive interface for users to interact with the powerful Python AI services.
