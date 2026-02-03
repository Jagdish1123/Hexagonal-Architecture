# Hexagonal Architecture: Complete Guide

> *A comprehensive guide to understanding and implementing Hexagonal Architecture (Ports & Adapters Pattern) with a practical Note Management application example in Java.*

---

## 📑 Table of Contents

### Part I: Hexagonal Architecture Theory
1. [Introduction](#1-introduction)
   - [What is Hexagonal Architecture?](#what-is-hexagonal-architecture)
   - [Why Hexagonal Architecture?](#why-hexagonal-architecture)
   - [Core Principles](#core-principles)

2. [Architecture Components](#2-architecture-components)
   - [The Domain (Core)](#the-domain-core)
   - [Ports](#ports)
   - [Adapters](#adapters)
   - [Dependency Flow](#dependency-flow)

3. [Key Concepts](#3-key-concepts)
   - [Inbound vs Outbound](#inbound-vs-outbound)
   - [Ports vs Adapters](#ports-vs-adapters)
   - [Dependency Inversion](#dependency-inversion)

### Part II: Practical Implementation (Note Registry App)
4. [Application Overview](#4-application-overview)
   - [Project Structure](#project-structure)
   - [Technology Stack](#technology-stack)

5. [Domain Layer Implementation](#5-domain-layer-implementation)
   - [Domain Services](#domain-services)
   - [Inbound Ports](#inbound-ports-use-cases)
   - [Outbound Ports](#outbound-ports-spi)
   - [Domain DTOs](#domain-dtos)

6. [Adapter Layer Implementation](#6-adapter-layer-implementation)
   - [Inbound Adapters](#inbound-adapters)
   - [Outbound Adapters](#outbound-adapters)
   - [Mappers & Entities](#mappers--entities)

7. [Request Flow Visualization](#7-request-flow-visualization)
   - [REST API Flow](#rest-api-flow)
   - [Event-Driven Flow](#event-driven-flow)
   - [Complete Architecture View](#complete-architecture-view)

8. [Testing Strategy](#8-testing-strategy)
   - [Unit Testing](#unit-testing-domain)
   - [Integration Testing](#integration-testing-adapters)
   - [End-to-End Testing](#end-to-end-testing)

9. [Advanced Scenarios](#9-advanced-scenarios)
   - [Multiple Database Support](#multiple-database-support)
   - [Adding New Interfaces](#adding-new-interfaces)
   - [Technology Migration](#technology-migration)

10. [Best Practices & Guidelines](#10-best-practices--guidelines)
    - [Do's and Don'ts](#dos-and-donts)
    - [Common Pitfalls](#common-pitfalls)
    - [Performance Considerations](#performance-considerations)

11. [Getting Started](#11-getting-started)
    - [Prerequisites](#prerequisites)
    - [Setup Instructions](#setup-instructions)
    - [Running the Application](#running-the-application)

12. [Resources & References](#12-resources--references)

---

## Part I: Hexagonal Architecture Theory

## 1. Introduction

### What is Hexagonal Architecture?

**Hexagonal Architecture**, also known as **Ports and Adapters**, is an architectural pattern created by Alistair Cockburn in 2005. It aims to create loosely coupled application components that can be easily connected to their software environment through ports and adapters.

**The Core Idea:**
- Place your **business logic** at the center
- Isolate it from **external concerns** (databases, frameworks, UI)
- Define clear **boundaries** using interfaces (ports)
- Implement technical details through **adapters**

```
              External World
                    │
        ┌───────────┼───────────┐
        │           │           │
    Adapters    🏛️ CORE     Adapters
        │      (Domain)        │
        └───────────┼───────────┘
                    │
              External World
```

### Why Hexagonal Architecture?

| Problem | Traditional Layered Architecture | Hexagonal Architecture |
|---------|----------------------------------|------------------------|
| **Database Changes** | Ripples through all layers | Only adapter changes |
| **Testing Business Logic** | Requires full stack setup | Pure unit tests possible |
| **Technology Migration** | Major refactoring needed | Swap adapters independently |
| **Multiple Interfaces** | Code duplication | Reuse domain logic |
| **Framework Updates** | Affects entire codebase | Isolated to adapters |

**Business Benefits:**
- ✅ **Faster Development**: Parallel work on domain and adapters
- ✅ **Lower Risk**: Changes isolated to specific components
- ✅ **Better Testing**: Test business logic without infrastructure
- ✅ **Technology Freedom**: Not locked into specific frameworks
- ✅ **Easier Onboarding**: Clear separation makes code easier to understand

### Core Principles

1. **Domain Independence**: Business logic has no dependencies on external frameworks
2. **Dependency Inversion**: Domain defines interfaces; adapters implement them
3. **Explicit Boundaries**: Clear separation through ports (interfaces)
4. **Testability**: Each component can be tested in isolation
5. **Flexibility**: Easy to swap implementations without affecting the core

---

## 2. Architecture Components

### The Domain (Core)

The **Domain** is the heart of your application containing all business logic.

**Characteristics:**
- ✅ Pure Java/business logic
- ✅ No framework dependencies (no Spring, JPA, Kafka annotations)
- ✅ No knowledge of databases, HTTP, or external systems
- ✅ Contains business rules and workflows
- ✅ Defines what the application can do (use cases)

**Structure:**
```
domain/
├── service/          # Business logic implementations
├── port/
│   ├── in/          # What app CAN DO (driving ports)
│   └── out/         # What app NEEDS (driven ports)
└── dto/             # Domain data models
```

### Ports

**Ports** are interfaces that define contracts between the domain and the outside world.

#### Inbound Ports (Driving Ports)
- Define **what the application can do**
- Represent **use cases**
- Called BY adapters
- Entry points to the domain

**Example:**
```java
public interface CreateNoteUseCase {
    NoteDto createNote(NoteDto noteDto);
}
```

#### Outbound Ports (Driven Ports)
- Define **what the application needs**
- Represent **external dependencies**
- Called BY domain
- Implemented by adapters

**Example:**
```java
public interface NoteRepositoryPort {
    NoteDto save(NoteDto noteDto);
    Optional<NoteDto> findById(UUID id);
}
```

### Adapters

**Adapters** are concrete implementations that connect the domain to external systems.

#### Inbound Adapters (Driving Adapters)
- **Receive** requests from outside
- **Translate** to domain language
- **Invoke** use cases (inbound ports)

**Examples:**
- REST Controllers (HTTP → Domain)
- CLI Commands (Console → Domain)
- Event Listeners (Kafka → Domain)
- GraphQL Resolvers (GraphQL → Domain)

#### Outbound Adapters (Driven Adapters)
- **Implement** outbound ports
- **Execute** technical operations
- **Translate** domain models to external formats

**Examples:**
- JPA Repositories (Domain → Database)
- REST Clients (Domain → External API)
- Event Publishers (Domain → Kafka)
- File System (Domain → Files)

### Dependency Flow

**The Golden Rule:** Dependencies point INWARD toward the domain.

```
┌─────────────────────────────────────────────┐
│         INBOUND ADAPTER                     │
│         (REST Controller)                   │
│              │                              │
│              │ depends on                   │
│              ▼                              │
│         INBOUND PORT ────────────────┐      │
│         (Interface)                  │      │
└──────────────────────────────────────┼──────┘
                                       │
┌──────────────────────────────────────┼──────┐
│              │                       │      │
│              │ implemented by        │      │
│              ▼                       │      │
│         DOMAIN SERVICE               │      │
│         (Business Logic)             │      │
│              │                       │      │
│              │ depends on            │      │
│              ▼                       │      │
│         OUTBOUND PORT ◄──────────────┘      │
│         (Interface)                         │
│              ▲                              │
│              │ implemented by               │
│              │                              │
└──────────────┼──────────────────────────────┘
               │
┌──────────────┼──────────────────────────────┐
│              │                              │
│         OUTBOUND ADAPTER                    │
│         (JPA Repository)                    │
└─────────────────────────────────────────────┘

KEY: Domain defines interfaces (ports)
     Adapters implement those interfaces
     Domain NEVER depends on adapters
```

---

## 3. Key Concepts

### Inbound vs Outbound

| Aspect | Inbound | Outbound |
|--------|---------|----------|
| **Direction** | Outside → Domain | Domain → Outside |
| **Purpose** | Trigger domain operations | Fulfill domain needs |
| **Port Type** | Use Case interfaces | Dependency interfaces |
| **Adapter Type** | Controllers, Listeners | Repositories, Clients |
| **Who Calls** | External systems call adapters | Domain calls ports |
| **Examples** | REST, CLI, Events | Database, API, Files |

### Ports vs Adapters

```
PORT (Interface)
├─ Defined in: domain/port/
├─ Contains: Method signatures only
├─ Purpose: Define contract
└─ Example: CreateNoteUseCase

ADAPTER (Implementation)
├─ Defined in: adapter/in/ or adapter/out/
├─ Contains: Actual implementation
├─ Purpose: Connect to real systems
└─ Example: NoteController, NoteJpaAdapter
```

### Dependency Inversion

**Traditional Approach (BAD):**
```
Controller → Service → JpaRepository
(High-level depends on low-level)
```

**Hexagonal Approach (GOOD):**
```
Controller → UseCase ← Service → RepositoryPort ← JpaAdapter
           (interface)         (interface)
           
Domain defines interfaces
Adapters implement them
```

**Benefits:**
- Domain doesn't know about Spring, JPA, HTTP
- Easy to mock for testing
- Can swap implementations without changing domain
- Clear contracts between layers

---

## Part II: Practical Implementation (Note Registry App)

## 4. Application Overview

### Project Description

**Note Registry** is a production-grade Note Management application demonstrating Hexagonal Architecture principles. It supports:
- Creating, reading, updating, and deleting notes
- Multiple interfaces (REST, CLI, Kafka)
- Multiple persistence options (JPA, MongoDB, File System)
- Event publishing for integration

### Project Structure

```
src/main/java/com/example/note_registry/
│
├── domain/                          # 🏛️ CORE BUSINESS LOGIC
│   ├── service/                     # Business rule implementations
│   │   └── NoteService.java
│   │
│   ├── port/
│   │   ├── in/                      # 📥 INBOUND PORTS (Use Cases)
│   │   │   ├── CreateNoteUseCase.java
│   │   │   ├── UpdateNoteUseCase.java
│   │   │   ├── DeleteNoteUseCase.java
│   │   │   └── GetNoteUseCase.java
│   │   │
│   │   └── out/                     # 📤 OUTBOUND PORTS (Dependencies)
│   │       ├── NoteRepositoryPort.java
│   │       └── NoteEventPublisherPort.java
│   │
│   └── dto/                         # Domain Data Models
│       └── NoteDto.java
│
├── adapter/                         # 🔌 INFRASTRUCTURE LAYER
│   │
│   ├── in/                          # Inbound Adapters (Driving)
│   │   ├── rest/
│   │   │   └── NoteController.java
│   │   ├── cli/
│   │   │   └── NoteCliAdapter.java
│   │   └── kafka/
│   │       └── NoteEventConsumer.java
│   │
│   ├── out/                         # Outbound Adapters (Driven)
│   │   ├── jpa/
│   │   │   ├── NoteJpaAdapter.java
│   │   │   └── NoteJpaRepository.java
│   │   ├── mongo/
│   │   │   ├── NoteMongoAdapter.java
│   │   │   └── NoteMongoRepository.java
│   │   ├── filesystem/
│   │   │   └── NoteFileSystemAdapter.java
│   │   └── kafka/
│   │       └── NoteEventPublisher.java
│   │
│   ├── entity/                      # Persistence Models
│   │   ├── NoteEntity.java          # JPA entity
│   │   └── NoteDocument.java        # MongoDB document
│   │
│   └── mapper/                      # DTO ↔ Entity Converters
│       └── NoteMapper.java
│
└── config/                          # ⚙️ CONFIGURATION
    ├── BeanConfiguration.java
    └── ApplicationConfig.java
```

### Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Language** | Java 17+ | Core programming language |
| **Framework** | Spring Boot 3.x | Dependency injection & configuration |
| **Build Tool** | Maven 3.8+ | Dependency management |
| **Web** | Spring Web | REST API endpoints |
| **Persistence** | Spring Data JPA | Relational database access |
| | MongoDB | NoSQL document storage |
| | Hibernate | ORM implementation |
| **Messaging** | Apache Kafka | Event streaming |
| **Testing** | JUnit 5 | Unit testing framework |
| | Mockito | Mocking framework |
| | Spring Boot Test | Integration testing |
| **Database** | PostgreSQL/MySQL | Production database |
| | H2 | In-memory testing database |

---

## 5. Domain Layer Implementation

### Domain Services

**Location:** `domain/service/NoteService.java`

**Responsibilities:**
- Implement business logic
- Orchestrate use cases
- Validate business rules
- Coordinate between ports

**Example:**
```java
@Service
public class NoteService implements CreateNoteUseCase, 
                                   UpdateNoteUseCase,
                                   GetNoteUseCase,
                                   DeleteNoteUseCase {
    
    private final NoteRepositoryPort repositoryPort;
    private final NoteEventPublisherPort eventPublisher;
    
    // Constructor injection (framework-agnostic)
    public NoteService(NoteRepositoryPort repositoryPort,
                      NoteEventPublisherPort eventPublisher) {
        this.repositoryPort = repositoryPort;
        this.eventPublisher = eventPublisher;
    }
    
    @Override
    public NoteDto createNote(NoteDto noteDto) {
        // Business validation
        validateNote(noteDto);
        
        // Generate ID and timestamp
        noteDto.setId(UUID.randomUUID());
        noteDto.setCreatedAt(LocalDateTime.now());
        
        // Save via port
        NoteDto savedNote = repositoryPort.save(noteDto);
        
        // Publish event via port
        eventPublisher.publishNoteCreated(savedNote);
        
        return savedNote;
    }
    
    private void validateNote(NoteDto note) {
        if (note.getTitle() == null || note.getTitle().isBlank()) {
            throw new InvalidNoteException("Title cannot be empty");
        }
        if (note.getContent() != null && note.getContent().length() > 5000) {
            throw new InvalidNoteException("Content too long");
        }
    }
}
```

**Key Points:**
- ✅ No Spring annotations on methods (only `@Service` for wiring)
- ✅ No JPA, HTTP, or Kafka code
- ✅ Depends only on port interfaces
- ✅ Pure business logic

### Inbound Ports (Use Cases)

**Location:** `domain/port/in/`

These interfaces define what the application can do.

**CreateNoteUseCase.java:**
```java
public interface CreateNoteUseCase {
    NoteDto createNote(NoteDto noteDto);
}
```

**UpdateNoteUseCase.java:**
```java
public interface UpdateNoteUseCase {
    NoteDto updateNote(UUID id, NoteDto noteDto);
}
```

**GetNoteUseCase.java:**
```java
public interface GetNoteUseCase {
    Optional<NoteDto> getNoteById(UUID id);
    List<NoteDto> getAllNotes();
}
```

**DeleteNoteUseCase.java:**
```java
public interface DeleteNoteUseCase {
    void deleteNote(UUID id);
}
```

### Outbound Ports (SPI)

**Location:** `domain/port/out/`

These interfaces define what the application needs from infrastructure.

**NoteRepositoryPort.java:**
```java
public interface NoteRepositoryPort {
    NoteDto save(NoteDto noteDto);
    Optional<NoteDto> findById(UUID id);
    List<NoteDto> findAll();
    void deleteById(UUID id);
    boolean existsById(UUID id);
}
```

**NoteEventPublisherPort.java:**
```java
public interface NoteEventPublisherPort {
    void publishNoteCreated(NoteDto noteDto);
    void publishNoteUpdated(NoteDto noteDto);
    void publishNoteDeleted(UUID noteId);
}
```

### Domain DTOs

**Location:** `domain/dto/NoteDto.java`

**Purpose:**
- Transfer data across boundaries
- Decouple domain from persistence models
- Ensure type safety

**Example:**
```java
public class NoteDto {
    private UUID id;
    private String title;
    private String content;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // Constructors
    public NoteDto() {}
    
    public NoteDto(UUID id, String title, String content) {
        this.id = id;
        this.title = title;
        this.content = content;
    }
    
    // Getters and Setters
    // ... (standard Java bean methods)
    
    // Business methods (if needed)
    public boolean isNew() {
        return id == null;
    }
}
```

**Key Points:**
- ✅ Plain Java class (POJO)
- ✅ No annotations (except validation if needed)
- ✅ No JPA/Hibernate references
- ✅ Can contain business validation logic

---

## 6. Adapter Layer Implementation

### Inbound Adapters

#### REST Controller

**Location:** `adapter/in/rest/NoteController.java`

**Responsibilities:**
- Handle HTTP requests
- Validate input
- Map HTTP data to domain DTOs
- Invoke use cases
- Return HTTP responses

**Example:**
```java
@RestController
@RequestMapping("/api/notes")
public class NoteController {
    
    private final CreateNoteUseCase createNoteUseCase;
    private final GetNoteUseCase getNoteUseCase;
    private final UpdateNoteUseCase updateNoteUseCase;
    private final DeleteNoteUseCase deleteNoteUseCase;
    
    public NoteController(CreateNoteUseCase createNoteUseCase,
                         GetNoteUseCase getNoteUseCase,
                         UpdateNoteUseCase updateNoteUseCase,
                         DeleteNoteUseCase deleteNoteUseCase) {
        this.createNoteUseCase = createNoteUseCase;
        this.getNoteUseCase = getNoteUseCase;
        this.updateNoteUseCase = updateNoteUseCase;
        this.deleteNoteUseCase = deleteNoteUseCase;
    }
    
    @PostMapping
    public ResponseEntity<NoteDto> createNote(@RequestBody @Valid CreateNoteRequest request) {
        NoteDto noteDto = mapToDto(request);
        NoteDto created = createNoteUseCase.createNote(noteDto);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<NoteDto> getNote(@PathVariable UUID id) {
        return getNoteUseCase.getNoteById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @GetMapping
    public ResponseEntity<List<NoteDto>> getAllNotes() {
        List<NoteDto> notes = getNoteUseCase.getAllNotes();
        return ResponseEntity.ok(notes);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<NoteDto> updateNote(@PathVariable UUID id,
                                              @RequestBody @Valid UpdateNoteRequest request) {
        NoteDto noteDto = mapToDto(request);
        NoteDto updated = updateNoteUseCase.updateNote(id, noteDto);
        return ResponseEntity.ok(updated);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteNote(@PathVariable UUID id) {
        deleteNoteUseCase.deleteNote(id);
        return ResponseEntity.noContent().build();
    }
    
    private NoteDto mapToDto(CreateNoteRequest request) {
        return new NoteDto(null, request.getTitle(), request.getContent());
    }
}
```

#### CLI Adapter

**Location:** `adapter/in/cli/NoteCliAdapter.java`

```java
@Component
public class NoteCliAdapter implements CommandLineRunner {
    
    private final CreateNoteUseCase createNoteUseCase;
    private final GetNoteUseCase getNoteUseCase;
    
    @Override
    public void run(String... args) {
        if (args.length == 0) return;
        
        String command = args[0];
        
        switch (command) {
            case "create":
                handleCreate(args);
                break;
            case "list":
                handleList();
                break;
            // ... other commands
        }
    }
    
    private void handleCreate(String[] args) {
        String title = args[1];
        String content = args[2];
        
        NoteDto noteDto = new NoteDto(null, title, content);
        NoteDto created = createNoteUseCase.createNote(noteDto);
        
        System.out.println("Note created: " + created.getId());
    }
}
```

#### Kafka Consumer

**Location:** `adapter/in/kafka/NoteEventConsumer.java`

```java
@Component
public class NoteEventConsumer {
    
    private final CreateNoteUseCase createNoteUseCase;
    
    @KafkaListener(topics = "note-commands", groupId = "note-service")
    public void consumeNoteCommand(String message) {
        NoteCommand command = parseCommand(message);
        
        if ("CREATE".equals(command.getAction())) {
            NoteDto noteDto = command.toDto();
            createNoteUseCase.createNote(noteDto);
        }
    }
}
```

### Outbound Adapters

#### JPA Adapter

**Location:** `adapter/out/jpa/NoteJpaAdapter.java`

```java
@Component
public class NoteJpaAdapter implements NoteRepositoryPort {
    
    private final NoteJpaRepository jpaRepository;
    private final NoteMapper mapper;
    
    public NoteJpaAdapter(NoteJpaRepository jpaRepository, NoteMapper mapper) {
        this.jpaRepository = jpaRepository;
        this.mapper = mapper;
    }
    
    @Override
    public NoteDto save(NoteDto noteDto) {
        NoteEntity entity = mapper.toEntity(noteDto);
        NoteEntity saved = jpaRepository.save(entity);
        return mapper.toDto(saved);
    }
    
    @Override
    public Optional<NoteDto> findById(UUID id) {
        return jpaRepository.findById(id)
            .map(mapper::toDto);
    }
    
    @Override
    public List<NoteDto> findAll() {
        return jpaRepository.findAll().stream()
            .map(mapper::toDto)
            .collect(Collectors.toList());
    }
    
    @Override
    public void deleteById(UUID id) {
        jpaRepository.deleteById(id);
    }
    
    @Override
    public boolean existsById(UUID id) {
        return jpaRepository.existsById(id);
    }
}
```

**JPA Repository Interface:**
```java
public interface NoteJpaRepository extends JpaRepository<NoteEntity, UUID> {
    // Spring Data JPA provides implementation
}
```

#### MongoDB Adapter

**Location:** `adapter/out/mongo/NoteMongoAdapter.java`

```java
@Component
@Profile("mongodb")
public class NoteMongoAdapter implements NoteRepositoryPort {
    
    private final NoteMongoRepository mongoRepository;
    private final NoteMapper mapper;
    
    @Override
    public NoteDto save(NoteDto noteDto) {
        NoteDocument document = mapper.toDocument(noteDto);
        NoteDocument saved = mongoRepository.save(document);
        return mapper.toDto(saved);
    }
    
    // ... similar implementation
}
```

#### Kafka Publisher

**Location:** `adapter/out/kafka/NoteEventPublisher.java`

```java
@Component
public class NoteEventPublisher implements NoteEventPublisherPort {
    
    private final KafkaTemplate<String, String> kafkaTemplate;
    
    @Override
    public void publishNoteCreated(NoteDto noteDto) {
        String event = createEventJson(noteDto, "NOTE_CREATED");
        kafkaTemplate.send("note-events", event);
    }
    
    @Override
    public void publishNoteUpdated(NoteDto noteDto) {
        String event = createEventJson(noteDto, "NOTE_UPDATED");
        kafkaTemplate.send("note-events", event);
    }
    
    @Override
    public void publishNoteDeleted(UUID noteId) {
        String event = createDeleteEventJson(noteId);
        kafkaTemplate.send("note-events", event);
    }
}
```

### Mappers & Entities

#### Mapper

**Location:** `adapter/mapper/NoteMapper.java`

```java
@Component
public class NoteMapper {
    
    // DTO → Entity
    public NoteEntity toEntity(NoteDto dto) {
        if (dto == null) return null;
        
        NoteEntity entity = new NoteEntity();
        entity.setId(dto.getId());
        entity.setTitle(dto.getTitle());
        entity.setContent(dto.getContent());
        entity.setCreatedAt(dto.getCreatedAt());
        entity.setUpdatedAt(dto.getUpdatedAt());
        return entity;
    }
    
    // Entity → DTO
    public NoteDto toDto(NoteEntity entity) {
        if (entity == null) return null;
        
        NoteDto dto = new NoteDto();
        dto.setId(entity.getId());
        dto.setTitle(entity.getTitle());
        dto.setContent(entity.getContent());
        dto.setCreatedAt(entity.getCreatedAt());
        dto.setUpdatedAt(entity.getUpdatedAt());
        return dto;
    }
    
    // DTO → Document (for MongoDB)
    public NoteDocument toDocument(NoteDto dto) {
        // Similar implementation
    }
}
```

#### JPA Entity

**Location:** `adapter/entity/NoteEntity.java`

```java
@Entity
@Table(name = "notes")
public class NoteEntity {
    
    @Id
    private UUID id;
    
    @Column(nullable = false, length = 255)
    private String title;
    
    @Column(length = 5000)
    private String content;
    
    @Column(name = "created_at", nullable = false)
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Getters and Setters
}
```

#### MongoDB Document

**Location:** `adapter/entity/NoteDocument.java`

```java
@Document(collection = "notes")
public class NoteDocument {
    
    @Id
    private String id;
    
    @Field("title")
    private String title;
    
    @Field("content")
    private String content;
    
    @Field("created_at")
    private LocalDateTime createdAt;
    
    // Getters and Setters
}
```

---

## 7. Request Flow Visualization

### REST API Flow

**Complete Journey: Client → Database → Client**

```
═══════════════════════════════════════════════════════════════════
Step 1: HTTP Request
═══════════════════════════════════════════════════════════════════
                    📱 HTTP Client
                         │
                         │ POST /api/notes
                         │ Content-Type: application/json
                         │ {
                         │   "title": "Meeting Notes",
                         │   "content": "Discuss Q1 goals"
                         │ }
                         ▼
───────────────────────────────────────────────────────────────────

═══════════════════════════════════════════════════════════════════
Step 2: Inbound Adapter (REST Controller)
═══════════════════════════════════════════════════════════════════
            ┌─────────────────────────────────┐
            │   🌐 NoteController             │
            │   (Inbound Adapter)             │
            │                                 │
            │  @PostMapping("/notes")         │
            │  createNote(@RequestBody...)    │
            │                                 │
            │  • Validates HTTP request       │
            │  • Maps JSON → NoteDto          │
            │  • No business logic here!      │
            └──────────────┬──────────────────┘
                           │
                           │ Invokes
                           ▼
───────────────────────────────────────────────────────────────────

═══════════════════════════════════════════════════════════════════
Step 3: Inbound Port (Use Case Interface)
═══════════════════════════════════════════════════════════════════
            ┌─────────────────────────────────┐
            │   📋 CreateNoteUseCase          │
            │   (Inbound Port - Interface)    │
            │                                 │
            │  interface {                    │
            │    NoteDto createNote(NoteDto)  │
            │  }                              │
            │                                 │
            │  • Defines contract             │
            │  • No implementation            │
            └──────────────┬──────────────────┘
                           │
                           │ Implemented by
                           ▼
───────────────────────────────────────────────────────────────────

═══════════════════════════════════════════════════════════════════
Step 4: Domain Service (Business Logic Core)
═══════════════════════════════════════════════════════════════════
            ┌─────────────────────────────────┐
            │   ⚙️  NoteService                │
            │   (Domain Core)                 │
            │                                 │
            │  Business Operations:           │
            │  ✓ Validate title != null       │
            │  ✓ Check content length < 5000  │
            │  ✓ Generate UUID                │
            │  ✓ Set createdAt timestamp      │
            │  ✓ Apply business rules         │
            │                                 │
            │  • Pure Java logic              │
            │  • No framework code            │
            │  • Framework-agnostic           │
            └──────────────┬──────────────────┘
                           │
                           │ Depends on
                           ▼
───────────────────────────────────────────────────────────────────

═══════════════════════════════════════════════════════════════════
Step 5: Outbound Port (Repository Interface)
═══════════════════════════════════════════════════════════════════
            ┌─────────────────────────────────┐
            │   🔌 NoteRepositoryPort         │
            │   (Outbound Port - Interface)   │
            │                                 │
            │  interface {                    │
            │    NoteDto save(NoteDto)        │
            │    Optional<NoteDto> findById() │
            │  }                              │
            │                                 │
            │  • Defines what domain needs    │
            │  • Domain doesn't know DB type  │
            └──────────────┬──────────────────┘
                           │
                           │ Implemented by
                           ▼
───────────────────────────────────────────────────────────────────

═══════════════════════════════════════════════════════════════════
Step 6: Outbound Adapter (JPA Implementation)
═══════════════════════════════════════════════════════════════════
            ┌─────────────────────────────────┐
            │   💾 NoteJpaAdapter             │
            │   (Outbound Adapter)            │
            │                                 │
            │  Operations:                    │
            │  ① Map NoteDto → NoteEntity     │
            │  ② jpaRepository.save(entity)   │
            │  ③ Map NoteEntity → NoteDto     │
            │  ④ Return to domain             │
            │                                 │
            │  • Contains JPA code            │
            │  • Handles persistence details  │
            └──────────────┬──────────────────┘
                           │
                           │ Persists to
                           ▼
            ┌─────────────────────────────────┐
            │   🗄️  PostgreSQL Database       │
            │                                 │
            │  SQL Execution:                 │
            │  INSERT INTO notes              │
            │    (id, title, content,         │
            │     created_at)                 │
            │  VALUES                         │
            │    ('uuid', 'Meeting Notes',    │
            │     'Discuss...', '2024-01-15') │
            └─────────────────────────────────┘
───────────────────────────────────────────────────────────────────

═══════════════════════════════════════════════════════════════════
Step 7: Response Flow (Reverse Journey)
═══════════════════════════════════════════════════════════════════

    🗄️  Database
         │ Returns saved entity
         ▼
    💾 JPA Adapter
         │ Maps Entity → DTO
         │ Returns: NoteDto
         ▼
    ⚙️  NoteService
         │ Returns: NoteDto
         ▼
    🌐 NoteController
         │ Maps DTO → JSON
         │ Status: 201 Created
         ▼
    📱 HTTP Client
    
    HTTP Response:
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "title": "Meeting Notes",
      "content": "Discuss Q1 goals",
      "createdAt": "2024-01-15T10:30:00"
    }
    
═══════════════════════════════════════════════════════════════════
```

### Event-Driven Flow

**Asynchronous Processing: Kafka → Domain → Database → Kafka**

```
┌───────────────────────────────────────────────────────────────┐
│              EVENT-DRIVEN PROCESSING FLOW                     │
└───────────────────────────────────────────────────────────────┘

Step 1: Event Arrives
═══════════════════════════════════════════════════════════════
    📨 Kafka Topic: "note-commands"
         │
         │ Event Message:
         │ {
         │   "action": "CREATE",
         │   "data": {
         │     "title": "Event Note",
         │     "content": "..."
         │   }
         │ }
         ▼
───────────────────────────────────────────────────────────────

Step 2: Event Consumed
═══════════════════════════════════════════════════════════════
    ┌──────────────────────────┐
    │ 🎧 NoteEventConsumer     │ ◄── Inbound Adapter
    │ (Kafka Listener)         │
    │                          │
    │ @KafkaListener           │
    │ consumeNoteCommand()     │
    │                          │
    │ • Deserializes message   │
    │ • Parses command         │
    └───────────┬──────────────┘
                │
                │ Triggers
                ▼
───────────────────────────────────────────────────────────────

Step 3: Use Case Invoked
═══════════════════════════════════════════════════════════════
    ┌──────────────────────────┐
    │ 📋 CreateNoteUseCase     │ ◄── Inbound Port
    │                          │
    │ createNote(noteDto)      │
    └───────────┬──────────────┘
                │
                ▼
───────────────────────────────────────────────────────────────

Step 4: Business Logic
═══════════════════════════════════════════════════════════════
    ┌──────────────────────────┐
    │ ⚙️  NoteService           │ ◄── Domain Service
    │                          │
    │ Process:                 │
    │ • Validate               │
    │ • Generate ID            │
    │ • Business rules         │
    └─────┬──────────────┬─────┘
          │              │
          │              └────────────────────┐
          ▼                                   ▼
───────────────────────────────────────────────────────────────

Step 5: Parallel Operations
═══════════════════════════════════════════════════════════════
    ┌──────────────────┐              ┌──────────────────┐
    │ 💾 Save to DB    │              │ 📤 Publish Event │
    │                  │              │                  │
    │ Repository Port  │              │ Event Port       │
    │      ▼           │              │      ▼           │
    │ JPA Adapter      │              │ Kafka Publisher  │
    └──────────────────┘              └─────────┬────────┘
                                                │
                                                ▼
───────────────────────────────────────────────────────────────

Step 6: Event Published
═══════════════════════════════════════════════════════════════
                        ┌──────────────────────┐
                        │ 🚀 KafkaPublisher    │ ◄── Outbound Adapter
                        │                      │
                        │ Publishes:           │
                        │ {                    │
                        │   "type":            │
                        │     "NOTE_CREATED",  │
                        │   "noteId": "...",   │
                        │   "timestamp": "..." │
                        │ }                    │
                        └──────────┬───────────┘
                                   │
                                   ▼
                        📨 Kafka Topic: "note-events"
                        
                        ✅ Other services can consume this event

═══════════════════════════════════════════════════════════════
```

### Complete Architecture View

**The Full Hexagonal Picture**

```
                    ╔════════════════════════════════════╗
                    ║     EXTERNAL ACTORS (Primary)      ║
                    ╚════════════════════════════════════╝
                                   │
    ┌──────────────────────────────┼──────────────────────────────┐
    │                              │                              │
    ▼                              ▼                              ▼
    
🌐 REST API              🎧 Event Streams            💻 CLI Interface
(Web Clients)            (Kafka Topics)              (Terminal)

    │                              │                              │
    └──────────────────────────────┼──────────────────────────────┘
                                   │
    ╔══════════════════════════════▼══════════════════════════════╗
    ║                    INBOUND ADAPTERS                         ║
    ║            (Translate External → Domain)                    ║
    ║  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      ║
    ║  │ REST         │  │ Kafka        │  │ CLI          │      ║
    ║  │ Controller   │  │ Consumer     │  │ Adapter      │      ║
    ║  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      ║
    ╚═════════╪══════════════════╪══════════════════╪═════════════╝
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 │
    ╔══════════════════════════════▼══════════════════════════════╗
    ║                      INBOUND PORTS                          ║
    ║           (USE CASE INTERFACES)                             ║
    ║         "What the application CAN DO"                       ║
    ║                                                             ║
    ║  📋 CreateNoteUseCase    📝 UpdateNoteUseCase               ║
    ║  🔍 GetNoteUseCase       🗑️ DeleteNoteUseCase               ║
    ║                                                             ║
    ║  • Defined by domain                                        ║
    ║  • Implemented by services                                  ║
    ║  • Called by inbound adapters                               ║
    ╚═════════════════════════════╪═══════════════════════════════╝
                                  │
                                  │ Implemented by
                                  ▼
    ╔══════════════════════════════════════════════════════════════╗
    ║                    ⭐ DOMAIN CORE ⭐                          ║
    ║                  (BUSINESS LOGIC LAYER)                      ║
    ║              "The Heart of the Application"                  ║
    ║                                                              ║
    ║    ┌──────────────────────────────────────────────┐          ║
    ║    │         ⚙️  NoteService                       │          ║
    ║    │                                              │          ║
    ║    │  Pure Business Logic:                        │          ║
    ║    │  • Validate business rules                   │          ║
    ║    │  • Orchestrate operations                    │          ║
    ║    │  • Coordinate between ports                  │          ║
    ║    │  • NO framework dependencies                 │          ║
    ║    │  • NO knowledge of HTTP, DB, Kafka           │          ║
    ║    │  • Pure Java + Domain DTOs                   │          ║
    ║    │                                              │          ║
    ║    │  Dependencies: ONLY Port Interfaces          │          ║
    ║    └──────────────────────────────────────────────┘          ║
    ║                                                              ║
    ╚═════════════════════════════╪════════════════════════════════╝
                                  │
                                  │ Depends on
                                  ▼
    ╔══════════════════════════════════════════════════════════════╗
    ║                     OUTBOUND PORTS                           ║
    ║          (DEPENDENCY INTERFACES)                             ║
    ║         "What the application NEEDS"                         ║
    ║                                                              ║
    ║  🔌 NoteRepositoryPort      📤 NoteEventPublisherPort        ║
    ║  📁 FileStoragePort         🔔 NotificationPort              ║
    ║                                                              ║
    ║  • Defined by domain                                         ║
    ║  • Implemented by outbound adapters                          ║
    ║  • Called by domain service                                  ║
    ╚═════════════════════════════╪════════════════════════════════╝
                                  │
                                  │ Implemented by
                                  ▼
    ┌─────────────────────────────┼─────────────────────────────┐
    │                             │                             │
    ▼                             ▼                             ▼
    
💾 JPA Adapter          🚀 Kafka Publisher      📁 FileSystem Adapter
(PostgreSQL)            (Events)                (Local Storage)

    │                             │                             │
    └─────────────────────────────┼─────────────────────────────┘
                                  │
    ╔══════════════════════════════▼══════════════════════════════╗
    ║                   OUTBOUND ADAPTERS                         ║
    ║            (Translate Domain → External)                    ║
    ║  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      ║
    ║  │ JPA/Mongo    │  │ Kafka        │  │ FileSystem   │      ║
    ║  │ Repository   │  │ Producer     │  │ Handler      │      ║
    ║  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      ║
    ╚═════════╪══════════════════╪══════════════════╪═════════════╝
              │                  │                  │
    ┌─────────┴──────────────────┴──────────────────┴─────────┐
    │                                                          │
    ▼                          ▼                          ▼
    
🗄️  Database            📨 Message Broker       💿 File System
(PostgreSQL/MongoDB)    (Apache Kafka)          (Local Disk)

                    ╔════════════════════════════════════╗
                    ║  EXTERNAL SYSTEMS (Secondary)      ║
                    ╚════════════════════════════════════╝


════════════════════════════════════════════════════════════════
                        KEY PRINCIPLES
════════════════════════════════════════════════════════════════

    → Dependencies point INWARD (toward domain)
    → Domain defines ALL interfaces (ports)
    → Adapters implement those interfaces
    → Domain NEVER depends on adapters
    → Domain is completely isolated and testable
    
════════════════════════════════════════════════════════════════
```

---

## 8. Testing Strategy

### Unit Testing (Domain)

**Test business logic in complete isolation**

**Example: NoteServiceTest.java**

```java
class NoteServiceTest {
    
    private NoteRepositoryPort mockRepository;
    private NoteEventPublisherPort mockPublisher;
    private NoteService noteService;
    
    @BeforeEach
    void setUp() {
        mockRepository = mock(NoteRepositoryPort.class);
        mockPublisher = mock(NoteEventPublisherPort.class);
        noteService = new NoteService(mockRepository, mockPublisher);
    }
    
    @Test
    void shouldCreateNoteWithValidData() {
        // GIVEN
        NoteDto inputDto = new NoteDto(null, "Test Note", "Content");
        NoteDto savedDto = new NoteDto(UUID.randomUUID(), "Test Note", "Content");
        
        when(mockRepository.save(any(NoteDto.class)))
            .thenReturn(savedDto);
        
        // WHEN
        NoteDto result = noteService.createNote(inputDto);
        
        // THEN
        assertNotNull(result.getId());
        assertEquals("Test Note", result.getTitle());
        verify(mockRepository).save(any(NoteDto.class));
        verify(mockPublisher).publishNoteCreated(any(NoteDto.class));
    }
    
    @Test
    void shouldThrowExceptionWhenTitleIsNull() {
        // GIVEN
        NoteDto invalidDto = new NoteDto(null, null, "Content");
        
        // WHEN & THEN
        assertThrows(InvalidNoteException.class, 
            () -> noteService.createNote(invalidDto));
        
        verify(mockRepository, never()).save(any());
    }
    
    @Test
    void shouldThrowExceptionWhenContentTooLong() {
        // GIVEN
        String longContent = "x".repeat(5001);
        NoteDto invalidDto = new NoteDto(null, "Title", longContent);
        
        // WHEN & THEN
        assertThrows(InvalidNoteException.class,
            () -> noteService.createNote(invalidDto));
    }
}
```

**Benefits:**
- ✅ No database required
- ✅ No Spring context
- ✅ Fast execution (milliseconds)
- ✅ Tests pure business logic
- ✅ Easy to maintain

### Integration Testing (Adapters)

**Test adapter implementations with real infrastructure**

**Example: NoteJpaAdapterTest.java**

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class NoteJpaAdapterTest {
    
    @Autowired
    private NoteJpaRepository jpaRepository;
    
    private NoteMapper mapper;
    private NoteJpaAdapter adapter;
    
    @BeforeEach
    void setUp() {
        mapper = new NoteMapper();
        adapter = new NoteJpaAdapter(jpaRepository, mapper);
    }
    
    @Test
    void shouldSaveAndRetrieveNote() {
        // GIVEN
        NoteDto noteDto = new NoteDto(
            UUID.randomUUID(),
            "Integration Test",
            "Testing JPA adapter"
        );
        
        // WHEN
        NoteDto saved = adapter.save(noteDto);
        Optional<NoteDto> retrieved = adapter.findById(saved.getId());
        
        // THEN
        assertTrue(retrieved.isPresent());
        assertEquals("Integration Test", retrieved.get().getTitle());
    }
    
    @Test
    void shouldReturnEmptyWhenNoteNotFound() {
        // WHEN
        Optional<NoteDto> result = adapter.findById(UUID.randomUUID());
        
        // THEN
        assertTrue(result.isEmpty());
    }
}
```

**Benefits:**
- ✅ Tests real database interactions
- ✅ Uses H2 in-memory database
- ✅ Validates mapping logic
- ✅ Catches SQL/ORM issues

### End-to-End Testing

**Test complete flow from API to database**

**Example: NoteControllerE2ETest.java**

```java
@SpringBootTest
@AutoConfigureMockMvc
class NoteControllerE2ETest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    void shouldCreateNoteViaRestApi() throws Exception {
        // GIVEN
        CreateNoteRequest request = new CreateNoteRequest(
            "E2E Test Note",
            "Full integration test"
        );
        
        String requestJson = objectMapper.writeValueAsString(request);
        
        // WHEN & THEN
        mockMvc.perform(post("/api/notes")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").exists())
            .andExpect(jsonPath("$.title").value("E2E Test Note"))
            .andExpect(jsonPath("$.content").value("Full integration test"))
            .andExpect(jsonPath("$.createdAt").exists());
    }
    
    @Test
    void shouldReturnNotFoundForNonExistentNote() throws Exception {
        // WHEN & THEN
        mockMvc.perform(get("/api/notes/" + UUID.randomUUID()))
            .andExpect(status().isNotFound());
    }
}
```

**Test Pyramid:**
```
         /\
        /  \  E2E Tests (Few)
       /____\  - Slow but comprehensive
      /      \ - Test user journeys
     /________\
    /          \ Integration Tests (Some)
   /____________\ - Test adapter implementations
  /              \ - Real infrastructure
 /________________\
/                  \ Unit Tests (Many)
/____________________\ - Fast and focused
                        - Test business logic
```

---

## 9. Advanced Scenarios

### Multiple Database Support

**Run the same application with different databases**

**Configuration:**

```java
// application-jpa.yml
spring:
  profiles: jpa
  datasource:
    url: jdbc:postgresql://localhost:5432/notes
  jpa:
    hibernate:
      ddl-auto: update

// application-mongodb.yml
spring:
  profiles: mongodb
  data:
    mongodb:
      uri: mongodb://localhost:27017/notes
```

**Conditional Beans:**

```java
@Configuration
public class PersistenceConfiguration {
    
    @Bean
    @Profile("jpa")
    @Primary
    public NoteRepositoryPort jpaRepositoryAdapter(
            NoteJpaRepository jpaRepo, 
            NoteMapper mapper) {
        return new NoteJpaAdapter(jpaRepo, mapper);
    }
    
    @Bean
    @Profile("mongodb")
    @Primary
    public NoteRepositoryPort mongoRepositoryAdapter(
            NoteMongoRepository mongoRepo,
            NoteMapper mapper) {
        return new NoteMongoAdapter(mongoRepo, mapper);
    }
}
```

**Result:**
```bash
# Run with PostgreSQL
java -jar app.jar --spring.profiles.active=jpa

# Run with MongoDB
java -jar app.jar --spring.profiles.active=mongodb
```

**What Changes:**
- ❌ Controller: NO CHANGE
- ❌ Domain Service: NO CHANGE
- ❌ Use Cases: NO CHANGE
- ✅ Only adapter implementation swapped
- ✅ Only configuration changed

### Adding New Interfaces

**Add GraphQL without touching existing code**

**New Inbound Adapter:**

```java
@Controller
public class NoteGraphQLAdapter {
    
    private final CreateNoteUseCase createUseCase;
    private final GetNoteUseCase getUseCase;
    
    @QueryMapping
    public List<NoteDto> allNotes() {
        return getUseCase.getAllNotes();
    }
    
    @MutationMapping
    public NoteDto createNote(@Argument String title, 
                             @Argument String content) {
        NoteDto dto = new NoteDto(null, title, content);
        return createUseCase.createNote(dto);
    }
}
```

**Result:**
- ✅ GraphQL, REST, and CLI all work simultaneously
- ✅ All use the same domain logic
- ✅ No code duplication
- ✅ Each adapter tailored to its protocol

### Technology Migration

**Scenario: Migrate from JPA to MongoDB**

**Step 1: Create MongoDB adapter**
```java
@Component
public class NoteMongoAdapter implements NoteRepositoryPort {
    // Implementation
}
```

**Step 2: Run both in parallel (Strangler Pattern)**
```java
@Component
public class DualWriteAdapter implements NoteRepositoryPort {
    
    private final NoteJpaAdapter jpaAdapter;
    private final NoteMongoAdapter mongoAdapter;
    
    @Override
    public NoteDto save(NoteDto dto) {
        // Write to both
        NoteDto jpaResult = jpaAdapter.save(dto);
        mongoAdapter.save(dto);
        return jpaResult;
    }
}
```

**Step 3: Verify data consistency**

**Step 4: Switch to MongoDB**
```java
@Configuration
public class MigrationConfig {
    
    @Bean
    @Primary
    public NoteRepositoryPort repositoryPort(NoteMongoAdapter adapter) {
        return adapter; // MongoDB now primary
    }
}
```

**Step 5: Remove JPA adapter**

**Benefits:**
- ✅ Zero downtime migration
- ✅ Gradual rollout possible
- ✅ Easy rollback if issues
- ✅ Domain logic unchanged

---

## 10. Best Practices & Guidelines

### Do's and Don'ts

#### ✅ DO

**Domain Layer:**
- ✅ Keep domain pure and framework-agnostic
- ✅ Use constructor injection
- ✅ Validate business rules in services
- ✅ Use meaningful domain exceptions
- ✅ Keep DTOs simple and focused

**Ports:**
- ✅ Define clear, cohesive interfaces
- ✅ Use domain language in method names
- ✅ Keep interfaces small and focused (ISP)
- ✅ Document expected behavior

**Adapters:**
- ✅ Handle all framework-specific code here
- ✅ Map between domain and external models
- ✅ Log adapter-level operations
- ✅ Handle infrastructure exceptions
- ✅ Use dependency injection

**Testing:**
- ✅ Write unit tests for all domain logic
- ✅ Mock port dependencies
- ✅ Test adapters with real infrastructure
- ✅ Write E2E tests for critical paths

#### ❌ DON'T

**Domain Layer:**
- ❌ Add Spring annotations to domain classes
- ❌ Use JPA entities in domain
- ❌ Import framework classes
- ❌ Handle HTTP concerns
- ❌ Catch infrastructure exceptions

**Ports:**
- ❌ Include implementation details
- ❌ Use framework-specific types
- ❌ Create "god interfaces"
- ❌ Leak adapter concerns

**Adapters:**
- ❌ Put business logic in adapters
- ❌ Call other adapters directly
- ❌ Expose infrastructure models to domain
- ❌ Skip input validation

**General:**
- ❌ Create circular dependencies
- ❌ Skip testing
- ❌ Over-engineer simple features
- ❌ Mix adapter concerns

### Common Pitfalls

#### 1. **Anemic Domain Model**

**Problem:**
```java
// Services with no logic, just pass-through
public class NoteService {
    public NoteDto createNote(NoteDto dto) {
        return repository.save(dto); // No business logic!
    }
}
```

**Solution:**
```java
public class NoteService {
    public NoteDto createNote(NoteDto dto) {
        validateTitle(dto.getTitle());
        validateContent(dto.getContent());
        enrichWithMetadata(dto);
        NoteDto saved = repository.save(dto);
        publishCreatedEvent(saved);
        return saved;
    }
}
```

#### 2. **Port Pollution**

**Problem:**
```java
// Port exposing adapter details
public interface NoteRepositoryPort {
    EntityManager getEntityManager(); // ❌ JPA leakage
    Session getSession(); // ❌ Hibernate leakage
}
```

**Solution:**
```java
// Clean, domain-focused port
public interface NoteRepositoryPort {
    NoteDto save(NoteDto note);
    Optional<NoteDto> findById(UUID id);
}
```

#### 3. **Bypassing Ports**

**Problem:**
```java
// Adapter calling another adapter directly
public class NoteController {
    private NoteJpaAdapter jpaAdapter; // ❌ Direct dependency
}
```

**Solution:**
```java
// Use ports/use cases
public class NoteController {
    private CreateNoteUseCase createUseCase; // ✅ Through port
}
```

### Performance Considerations

#### 1. **Mapping Overhead**

**Issue:** DTO ↔ Entity conversions on every request

**Solutions:**
- Use MapStruct for compile-time mapping
- Cache frequently accessed mappings
- Consider projection queries for read-only data

```java
@Mapper(componentModel = "spring")
public interface NoteMapper {
    NoteDto toDto(NoteEntity entity);
    NoteEntity toEntity(NoteDto dto);
}
```

#### 2. **N+1 Query Problem**

**Issue:** Lazy loading causing multiple queries

**Solutions:**
- Use fetch joins in adapters
- Implement repository methods with proper fetching
- Keep performance concerns in adapters, not domain

```java
@Query("SELECT n FROM NoteEntity n LEFT JOIN FETCH n.tags WHERE n.id = :id")
Optional<NoteEntity> findByIdWithTags(@Param("id") UUID id);
```

#### 3. **Excessive Abstraction**

**Issue:** Too many layers hurting performance

**Balance:**
- Don't create ports for every tiny operation
- Group related operations in single ports
- Measure actual performance impact
- Optimize hot paths if needed

---

## 11. Getting Started

### Prerequisites

**Required:**
- Java 17 or higher
- Maven 3.8+
- IDE (IntelliJ IDEA, Eclipse, or VS Code)

**Optional (for full features):**
- PostgreSQL 14+ (or MySQL 8+)
- MongoDB 5.0+
- Apache Kafka 3.0+
- Docker & Docker Compose

### Setup Instructions

#### 1. **Clone and Build**

```bash
# Clone repository
git clone https://github.com/your-org/note-registry.git
cd note-registry

# Build project
mvn clean install

# Run tests
mvn test
```

#### 2. **Database Setup**

**Option A: Using Docker Compose**

```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: notes
      POSTGRES_USER: noteuser
      POSTGRES_PASSWORD: notepass
    ports:
      - "5432:5432"
  
  mongodb:
    image: mongo:5.0
    ports:
      - "27017:27017"
  
  kafka:
    image: confluentinc/cp-kafka:7.0.0
    ports:
      - "9092:9092"
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
```

```bash
docker-compose up -d
```

**Option B: Local Installation**

Install PostgreSQL, MongoDB, and Kafka locally following official documentation.

#### 3. **Configuration**

**application.yml:**

```yaml
spring:
  application:
    name: note-registry
  
  datasource:
    url: jdbc:postgresql://localhost:5432/notes
    username: noteuser
    password: notepass
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
  
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: note-service
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer

server:
  port: 8080

logging:
  level:
    com.example.note_registry: DEBUG
```

### Running the Application

#### 1. **Start with JPA (PostgreSQL)**

```bash
mvn spring-boot:run -Dspring-boot.run.profiles=jpa
```

#### 2. **Start with MongoDB**

```bash
mvn spring-boot:run -Dspring-boot.run.profiles=mongodb
```

#### 3. **Access the Application**

**REST API:**
```bash
# Create a note
curl -X POST http://localhost:8080/api/notes \
  -H "Content-Type: application/json" \
  -d '{"title":"My First Note","content":"Hello Hexagonal!"}'

# Get all notes
curl http://localhost:8080/api/notes

# Get specific note
curl http://localhost:8080/api/notes/{id}
```

**CLI:**
```bash
java -jar target/note-registry.jar create "CLI Note" "Created from command line"
java -jar target/note-registry.jar list
```

**Kafka:**
```bash
# Send event
kafka-console-producer --broker-list localhost:9092 --topic note-commands
{"action":"CREATE","data":{"title":"Kafka Note","content":"From event"}}
```

---

## 12. Resources & References

### Official Documentation

- **Hexagonal Architecture**
  - [Original Article by Alistair Cockburn](https://alistair.cockburn.us/hexagonal-architecture/)
  - [Ports and Adapters Pattern](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/)

- **Clean Architecture**
  - [Clean Architecture by Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
  - [Book: Clean Architecture](https://www.oreilly.com/library/view/clean-architecture-a/9780134494166/)

- **Domain-Driven Design**
  - [Domain-Driven Design by Eric Evans](https://www.domainlanguage.com/ddd/)
  - [Implementing Domain-Driven Design by Vaughn Vernon](https://vaughnvernon.com/)

### Tools & Frameworks

- [Spring Boot](https://spring.io/projects/spring-boot)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [Apache Kafka](https://kafka.apache.org/)
- [MapStruct](https://mapstruct.org/)

### Community & Learning

- [Hexagonal Architecture GitHub Topic](https://github.com/topics/hexagonal-architecture)
- [DDD Community](https://github.com/ddd-crew)
- [Software Architecture Subreddit](https://www.reddit.com/r/softwarearchitecture/)

---

## 📊 Quick Reference

### Architecture Layers

| Layer | Location | Purpose | Dependencies |
|-------|----------|---------|--------------|
| **Domain** | `domain/` | Business logic | None (pure Java) |
| **Inbound Ports** | `domain/port/in/` | Use case contracts | None |
| **Outbound Ports** | `domain/port/out/` | Dependency contracts | None |
| **Inbound Adapters** | `adapter/in/` | External triggers | Domain ports |
| **Outbound Adapters** | `adapter/out/` | Infrastructure | Domain ports |

### Data Flow

```
External Request
      ↓
Inbound Adapter (translate)
      ↓
Inbound Port (contract)
      ↓
Domain Service (business logic)
      ↓
Outbound Port (contract)
      ↓
Outbound Adapter (translate)
      ↓
External System
```

### Key Principles

1. **Dependency Rule**: Dependencies point inward toward domain
2. **Domain Independence**: Core has zero framework dependencies
3. **Interface Segregation**: Small, focused ports
4. **Testability**: Each layer tested independently
5. **Flexibility**: Easy to swap adapters

---

## 🎯 Summary

**Hexagonal Architecture provides:**

✅ **Clean Separation**: Business logic isolated from technical concerns
✅ **Testability**: Easy to test each component independently
✅ **Flexibility**: Swap databases, frameworks, or protocols easily
✅ **Maintainability**: Changes isolated to specific layers
✅ **Scalability**: Independent deployment and evolution
✅ **Domain Focus**: Business rules are first-class citizens

**Best suited for:**
- Enterprise applications
- Long-lived projects
- Projects with evolving requirements
- Systems requiring high testability
- Applications with multiple interfaces
- Microservices architectures

**Trade-offs:**
- More initial setup than simple layered architecture
- Requires discipline to maintain boundaries
- Can be over-engineering for simple CRUD apps
- Learning curve for team members

---

## 📝 License

This guide and the example Note Registry application are licensed under the MIT License.

---

## 👥 Contributing

Contributions are welcome! Please ensure:
- Code follows hexagonal architecture principles
- All layers properly separated
- Comprehensive tests included
- Documentation updated

---

**Made with ❤️ for clean, maintainable software architecture**

*Last Updated: January 2025*
