PaiSmart is an enterprise-level AI knowledge base management system that adopts Retrieval-Augmented Generation (RAG) technology to provide intelligent document processing and retrieval capabilities.

The core technology stack includes ElasticSearch, Kafka, WebSocket, Spring Security, Docker, MySQL, and Redis.

Its goal is to help enterprises and individuals manage and utilize information in their knowledge bases more efficiently. It supports a multi-tenant architecture, allowing users to query the knowledge base through natural language and receive AI-generated responses based on their own documents.

![PaiSmart Multi-module Architecture](https://cdn.tobebetterjavaer.com/stutymore/README-20250730102133.png)

The system allows users to:

- Upload and manage various types of documents
- Automatically process and index document content
- Query the knowledge base using natural language
- Receive AI-generated responses based on their own documents

The technology stack used includes, starting with the backend:

+ Framework: Spring Boot 3.4.2 (Java 17)
+ Database: MySQL 8.0
+ ORM: Spring Data JPA
+ Cache: Redis
+ Search Engine: Elasticsearch 8.10.0
+ Message Queue: Apache Kafka
+ File Storage: MinIO
+ Document Parsing: Apache Tika
+ Security Authentication: Spring Security + JWT
+ AI Integration: DeepSeek API / Local Ollama + Doubao Embedding
+ Real-time Communication: WebSocket
+ Dependency Management: Maven
+ Reactive Programming: WebFlux

The overall project structure of the backend:

```bash
src/main/java/com/yizhaoqi/smartpai/
├── SmartPaiApplication.java      # Main application entry point
├── client/                       # External API clients
├── config/                       # Configuration classes
├── consumer/                     # Kafka consumers
├── controller/                   # REST API endpoints
├── entity/                       # Data entities
├── exception/                    # Custom exceptions
├── handler/                      # WebSocket handlers
├── model/                        # Domain models
├── repository/                   # Data access layer
├── service/                      # Business logic
└── utils/                        # Utility classes
```

Moving on to the frontend, it includes:

+ Framework: Vue 3 + TypeScript
+ Build Tool: Vite
+ UI Components: Naive UI
+ State Management: Pinia
+ Router: Vue Router
+ Styling: UnoCSS + SCSS
+ Icons: Iconify
+ Package Management: pnpm

The overall project structure of the frontend:

```bash
frontend/
├── packages/           # Reusable modules
├── public/             # Static assets
├── src/                # Main application code
│   ├── assets/         # SVG icons, images
│   ├── components/     # Vue components
│   ├── layouts/        # Page layouts
│   ├── router/         # Routing configuration
│   ├── service/        # API integration
│   ├── store/          # State management
│   ├── views/          # Page components
│   └── ...             # Other utilities and configurations
└── ...                 # Build configuration files
```

## Core Features

First, let me introduce what PaiSmart is, why I created this enterprise-level RAG knowledge base, what you can learn from this AI project, and how to unlock the PaiSmart source code repository and tutorials.

![PaiSmart's Chat Assistant: Q&A based on the knowledge base](https://cdn.tobebetterjavaer.com/paicoding/2550c873a349d8bee29d46400f12ce76.png)

![PaiSmart Architecture Overview](https://cdn.tobebetterjavaer.com/stutymore/README-20250730101618.png)

### Knowledge Base Management

PaiSmart provides comprehensive document upload and parsing features, supporting chunked uploads and resumable uploads, as well as organization and management using tags. Documents can be public or private, and can be associated with specific organization tags for better permission categorization.

![PaiSmart Document Processing](https://cdn.tobebetterjavaer.com/stutymore/README-20250730102808.png)

### AI-Driven RAG Implementation

The core of PaiSmart is its RAG implementation:

![PaiSmart Chat Interaction](https://cdn.tobebetterjavaer.com/stutymore/README-20250730102837.png)

- Performs semantic chunking on uploaded documents.
- Calls the Doubao Embedding model to generate high-dimensional vectors for each text chunk.
- Stores the vectors into ElasticSearch to support semantic search and keyword search.
- Retrieves relevant documents based on user queries.
- Provides complete context to the LLM to generate more accurate, document-based responses.

### Enterprise-Level Multi-Tenancy

PaiSmart supports a multi-tenant architecture through organization tags. Each user can create or join one or more organizations, and each organization can have independent knowledge bases and document management. This allows enterprises to manage knowledge bases for multiple teams or departments within the same system without worrying about data confusion or permission issues.

![PaiSmart Security Architecture](https://cdn.tobebetterjavaer.com/stutymore/README-20250730103118.png)

### Real-Time Communication

The system uses WebSocket technology to provide real-time interaction between users and the AI system, supporting a responsive chat interface that facilitates knowledge retrieval and AI interaction.

## Prerequisites

Before you begin, please ensure the following software is installed:

- Java 17
- Maven 3.8.6 or higher
- Node.js 18.20.0 or higher
- pnpm 8.7.0 or higher
- MySQL 8.0
- Elasticsearch 8.10.0
- MinIO 8.5.12
- Kafka 3.2.1
- Redis 7.0.11
- Docker (Optional, for running services like Redis, MinIO, Elasticsearch, and Kafka)

## Architecture Design

PaiSmart's architecture features the characteristics of a modern, cloud-native application, with clear separation of concerns, scalable components, and integration with AI technologies. The modular design allows individual components to be scaled and replaced in the future as technology evolves, especially in the rapidly changing field of AI integration.

![PaiSmart System Overview](https://cdn.tobebetterjavaer.com/stutymore/README-20250730102655.png)

The controller layer is responsible for handling HTTP requests, validating inputs, managing request/response formatting, and delegating business logic to the service layer. Controllers are organized by domain functionality. They follow RESTful design principles and integrate performance monitoring and logging for tracking API usage and troubleshooting.

```java
@RestController
@RequestMapping("/api/v1/documents")
public class DocumentController {
    @Autowired
    private DocumentService documentService;
    
    @DeleteMapping("/{fileMd5}")
    public ResponseEntity<?> deleteDocument(
            @PathVariable String fileMd5,
            @RequestAttribute("userId") String userId,
            @RequestAttribute("role") String role) {
        // Parameter validation and delegation to the service
        documentService.deleteDocument(fileMd5);
        // Response handling
    }
}
```

The service layer primarily handles the application's business logic, is transaction-aware, and can manage operations that span multiple data sources.

```java
@Service
public class DocumentService {
    @Autowired
    private FileUploadRepository fileUploadRepository;
    
    @Autowired
    private MinioClient minioClient;
    
    @Autowired
    private ElasticsearchService elasticsearchService;
    
    @Transactional
    public void deleteDocument(String fileMd5) {
        // Business logic for document deletion
        // Coordinating multiple repositories and systems
    }
}
```

The data access layer uses Spring Data JPA for database operations, providing CRUD operations for MySQL.

```java
@Repository
public interface FileUploadRepository extends JpaRepository<FileUpload, Long> {
    Optional<FileUpload> findByFileMd5(String fileMd5);
    
    @Query("SELECT f FROM FileUpload f WHERE f.userId = :userId OR f.isPublic = true OR (f.orgTag IN :orgTagList AND f.isPublic = false)")
    List<FileUpload> findAccessibleFilesWithTags(@Param("userId") String userId, @Param("orgTagList") List<String> orgTagList);
}
```

The entity layer consists of JPA entities mapped to database tables and DTOs (Data Transfer Objects) used for API requests and responses.

```java
@Entity
public class FileUpload {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String fileMd5;
    private String fileName;
    private String userId;
    private boolean isPublic;
    private String orgTag;
    // Other fields and methods
}
```

## Frontend Startup

```bash
# Navigate to the frontend project directory
cd frontend

# Install dependencies
pnpm install

# Start the project
pnpm run dev
```
