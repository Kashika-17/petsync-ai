# DATABASE

**Project:** PetSync AI

**Version:** 1.0

**Document Version:** 1.0

**Status:** Approved

**Owner:** Engineering

---

# Table of Contents

1. Document Information
2. Database Philosophy
3. Database Architecture
4. Storage Strategy
5. Database Principles
6. Database Technologies
7. Core Business Domains
8. Aggregate Design
9. Core Entity Relationships
10. Workspace Aggregate
11. Authentication Aggregate
12. Pet Aggregate
13. Health Aggregate
14. Vaccination Aggregate
15. Medication Aggregate
16. Document Aggregate
17. Timeline Aggregate
18. Reminder Aggregate
19. Notification Aggregate
20. AI Aggregate
21. Search Aggregate
22. Billing Aggregate
23. Audit Logging
24. Event Store
25. AI Memory
26. Embeddings
27. Index Strategy
28. Naming Conventions
29. Soft Deletes
30. Transactions
31. Performance
32. Backup Strategy
33. Scaling Strategy
34. Future Database Expansion
35. Database Summary

---

# 1. Document Information

## Purpose

This document defines the complete persistence architecture for **PetSync AI Version 1.0**.

It specifies how business information is stored, organized, secured, indexed, queried, versioned, and managed throughout the lifecycle of the platform.

The database architecture supports an AI-first, event-driven, domain-driven application while remaining independent of implementation details.

This document establishes the persistence strategy used across the entire platform and serves as the authoritative reference for database design.

Detailed implementation guidance for repositories, ORM models, migrations, and SQL scripts is documented separately.

---

## Scope

This document covers:

- Database architecture
- Persistence strategy
- Storage technologies
- Business aggregates
- Entity ownership
- Relationships
- Indexing strategy
- Transactions
- AI persistence
- Event persistence
- Backup strategy
- Scalability

This document does **not** define:

- API endpoints
- Business workflows
- SQL migrations
- ORM implementation
- Repository code

Those topics are covered in their respective engineering documents.

---

## Intended Audience

This document is intended for:

- Backend Engineers
- Database Engineers
- AI Engineers
- Solution Architects
- DevOps Engineers
- Technical Leads

---

## Objectives

The persistence architecture should provide:

- Data integrity
- High availability
- Scalability
- Security
- Performance
- Auditability
- AI compatibility
- Long-term maintainability

---

# 2. Database Philosophy

## Overview

PetSync AI treats the database as the authoritative system of record for all business information.

Rather than organizing persistence around pages, screens, or API endpoints, the platform organizes data around business domains and aggregates.

The database is designed to support:

- Transactional consistency
- Domain ownership
- Event-driven processing
- AI-assisted workflows
- Semantic search
- Future platform expansion

Correctness always takes priority over optimization.

Performance optimizations should never compromise data integrity.

---

## Design Principles

### Business-First Modeling

Database entities represent business concepts rather than UI components.

Examples include:

- Workspace
- User
- Pet
- Health Record
- Vaccination
- Medication
- Reminder
- Timeline Event
- Medical Document

---

### Aggregate Ownership

Each aggregate owns its own persistence.

No aggregate should directly modify another aggregate's data.

Communication occurs through:

- Domain services
- Domain events
- Application services

---

### Single Source of Truth

Every business entity has exactly one authoritative owner.

Examples:

| Business Data | Aggregate Owner |
|--------------|-----------------|
| Pet Profile | Pet Aggregate |
| Medical History | Health Aggregate |
| Billing | Billing Aggregate |
| Notifications | Notification Aggregate |
| AI Conversations | AI Aggregate |

---

### Immutable History

Historical information should be preserved whenever possible.

Examples:

- Timeline Events
- Audit Logs
- AI Conversations
- Notifications
- Domain Events

Historical records should not be overwritten.

---

### AI-Native Persistence

Artificial Intelligence is treated as a first-class platform capability.

The persistence layer stores:

- AI Sessions
- AI Conversations
- Prompt History
- AI Memory
- Embeddings
- Recommendations
- Usage Metrics
- Provider Metadata

AI is part of the platform architecture rather than an external integration.

---

# 3. Database Architecture

## Overview

PetSync AI follows a **polyglot persistence architecture**, using different storage technologies for different workloads.

Each storage system has a clearly defined responsibility.

```text
                    Application

                          │

                  Repository Layer

                          │

        ┌─────────────────┼─────────────────┐

        ▼                 ▼                 ▼

   PostgreSQL         Redis Cache      Object Storage

        │

        ▼

      pgvector
```

The application communicates only through repositories.

Repositories abstract the underlying storage technology.

---

## PostgreSQL

Primary transactional database.

Responsibilities include:

- Users
- Workspaces
- Pets
- Health Records
- Vaccinations
- Medications
- Documents Metadata
- Timeline Events
- Notifications
- Billing
- AI Metadata
- Audit Logs

PostgreSQL is the single source of truth for business data.

---

## Redis

Acts as the in-memory data layer.

Responsibilities include:

- Session storage
- API rate limiting
- Temporary caching
- Queue coordination
- Distributed locking
- Temporary AI context

Redis is never used as permanent storage.

---

## Object Storage

Stores binary assets.

Examples include:

- Images
- PDFs
- Medical Reports
- Vaccination Certificates
- Insurance Documents
- Attachments

Only file metadata is stored in PostgreSQL.

---

## Vector Store (pgvector)

Semantic search uses **pgvector** during Version 1.0.

Responsibilities include:

- Document embeddings
- Conversation embeddings
- Timeline embeddings
- AI knowledge retrieval

Future versions may migrate to dedicated vector databases such as:

- Pinecone
- Weaviate
- Qdrant

The application architecture remains provider-independent.

---

# 4. Storage Strategy

Different categories of information have different persistence requirements.

| Data Type | Storage |
|-----------|---------|
| Users | PostgreSQL |
| Workspaces | PostgreSQL |
| Pets | PostgreSQL |
| Health Records | PostgreSQL |
| Vaccinations | PostgreSQL |
| Medications | PostgreSQL |
| Timeline Events | PostgreSQL |
| Notifications | PostgreSQL |
| Billing | PostgreSQL |
| AI Metadata | PostgreSQL |
| Uploaded Files | Object Storage |
| Sessions | Redis |
| Cache | Redis |
| Rate Limits | Redis |
| Embeddings | pgvector |

---

## Storage Responsibilities

Each storage layer has a single responsibility.

| Storage Layer | Responsibility |
|--------------|----------------|
| PostgreSQL | Business Truth |
| Redis | Performance |
| Object Storage | Binary Files |
| pgvector | Semantic Search |

No storage technology should duplicate another's responsibility unnecessarily.

---

# 5. Database Principles

The persistence layer follows several architectural principles.

## Principle 1 — Strong Consistency

Critical business operations must be ACID compliant.

Examples include:

- User Registration
- Workspace Creation
- Pet Registration
- Vaccination Records
- Medication Tracking
- Billing Transactions

---

## Principle 2 — Separation of Persistence

Business logic must never interact directly with storage technologies.

Application services communicate only with repositories.

Repositories abstract:

- SQL
- Redis
- Object Storage
- Vector Storage

---

## Principle 3 — Normalized Core Data

Transactional entities should remain normalized.

Normalization reduces:

- Data duplication
- Update anomalies
- Inconsistent records

---

## Principle 4 — Controlled Denormalization

Read-heavy workloads may use denormalized projections.

Examples:

- Dashboard summaries
- Analytics
- Search indexes
- AI retrieval context

Denormalized data must never become the primary source of truth.

---

## Principle 5 — Auditability

Every critical operation should be traceable.

Examples include:

- Record creation
- Record updates
- Soft deletion
- AI interactions
- Permission changes

Audit information must be retained according to platform policies.

---

## Principle 6 — Security

Sensitive information must support:

- Encryption at rest
- Encryption in transit
- Access control
- Audit logging
- Secure backups

---

## Principle 7 — Scalability

The persistence layer must support horizontal application scaling without architectural redesign.

Future growth should require infrastructure scaling rather than schema redesign.

---

# 6. Database Technologies

## Relational Database

**Technology:** PostgreSQL

Responsibilities:

- Transactions
- Relational integrity
- Constraints
- Business records
- ACID compliance

---

## Cache Layer

**Technology:** Redis

Responsibilities:

- Session management
- Frequently accessed data
- Temporary AI context
- Distributed coordination
- Queue support

---

## Object Storage

Supported providers include:

- Amazon S3
- Cloudflare R2

Responsibilities:

- Secure binary storage
- Versioning
- Signed URLs
- Lifecycle management

---

## Vector Store

Current implementation:

- pgvector

Future supported technologies:

- Pinecone
- Weaviate
- Qdrant

Starting with pgvector simplifies deployment while preserving the ability to migrate later.

---

# 7. Core Business Domains

The persistence model is organized around business domains.

```text
Workspace

├── Authentication

├── Pets

├── Health

├── Vaccinations

├── Medications

├── Documents

├── Timeline

├── Reminders

├── Notifications

├── Search

├── Billing

└── AI
```

Each domain owns:

- Entities
- Relationships
- Constraints
- Repositories
- Events

Domains communicate through services and domain events rather than direct database manipulation.

---

# 8. Aggregate Design

PetSync AI models persistence around **aggregates**, following Domain-Driven Design principles.

Each aggregate defines a consistency boundary.

```text
Workspace

│

├── Pet

│     ├── Health

│     ├── Vaccinations

│     ├── Medications

│     ├── Documents

│     ├── Timeline

│     ├── Notes

│     └── Reminders

│

├── Notifications

├── Billing

└── AI
```

Each aggregate has:

- One aggregate root
- Clearly defined ownership
- Transaction boundaries
- Independent lifecycle

Aggregates communicate through domain services and events.

---

# 9. Core Entity Relationships

The highest-level ownership hierarchy is shown below.

```text
Workspace
│
├── Users
│
├── Pets
│     ├── Health Records
│     ├── Vaccinations
│     ├── Medications
│     ├── Documents
│     ├── Timeline Events
│     ├── Notes
│     └── Reminders
│
├── Notifications
│
├── AI Sessions
│
├── Billing
│
└── Audit Logs
```

## Relationship Principles

The platform follows these ownership rules:

- A **Workspace** owns all business data associated with an organization or household.
- A **User** belongs to one or more workspaces through role-based access.
- A **Pet** is the aggregate root for all pet-specific information.
- Health records, medications, vaccinations, documents, reminders, notes, and timeline events belong exclusively to a specific pet.
- AI sessions may reference multiple business entities but never own them.
- Audit logs record system activity without affecting business entities.
- Cross-domain interactions occur through application services and domain events rather than direct aggregate modification.

---

**End of Part 1**

**Next:** Part 2 — Aggregate Definitions (Workspace, Authentication, Pet, Health, Vaccination, Medication, Document, Timeline, Reminder, Notification, Billing)

# 10. Workspace Aggregate

## Purpose

The Workspace Aggregate represents the highest-level ownership boundary within PetSync AI.

Every business entity in the platform belongs to a workspace, making it the primary security, authorization, collaboration, and data isolation boundary.

A workspace may represent:

- An individual pet owner
- A family
- A veterinary clinic (future)
- An organization (future)

---

## Aggregate Root

**Workspace**

The Workspace entity owns all resources created within its boundary.

---

## Owned Entities

- Workspace
- Workspace Member
- Workspace Role
- Workspace Settings
- Workspace Preferences

---

## Relationships

```text
Workspace
│
├── Members
├── Pets
├── Notifications
├── Billing
├── AI Sessions
└── Audit Logs
```

---

## Business Rules

- Every user belongs to at least one workspace.
- Every pet belongs to exactly one workspace.
- Users may belong to multiple workspaces.
- Workspace owners may invite additional members.
- Workspace deletion is soft-delete only.

---

## Transaction Boundary

Operations executed atomically include:

- Workspace creation
- Member invitation
- Member removal
- Ownership transfer

---

## Lifecycle

```text
Created
    │
Active
    │
Suspended
    │
Archived
    │
Deleted
```

---

## Future Expansion

Future versions may support:

- Organizations
- Veterinary Clinics
- Team Permissions
- Shared AI Memory
- Shared Knowledge Base

---

# 11. Authentication Aggregate

## Purpose

The Authentication Aggregate manages identity, authentication, authorization, and session management.

Business entities never directly authenticate users.

Authentication remains centralized.

---

## Aggregate Root

**User**

---

## Owned Entities

- User
- Session
- OAuth Account
- Refresh Token
- MFA Configuration
- Login History

---

## Relationships

```text
User
│
├── Sessions
├── OAuth Accounts
├── MFA
├── Workspace Memberships
└── Login History
```

---

## Business Rules

- Email addresses are unique.
- Passwords are never stored in plaintext.
- Sessions expire automatically.
- Refresh tokens are revocable.
- Users may authenticate using multiple providers.

---

## Transaction Boundary

Atomic operations include:

- Registration
- Login
- Password reset
- Email verification
- MFA activation

---

## Lifecycle

```text
Registered
    │
Verified
    │
Active
    │
Suspended
    │
Deleted
```

---

## Future Expansion

- Enterprise SSO
- Passkeys
- Hardware Security Keys
- Device Management

---

# 12. Pet Aggregate

## Purpose

The Pet Aggregate represents the central business entity of the platform.

Nearly every operational domain is associated with a pet.

---

## Aggregate Root

**Pet**

---

## Owned Entities

- Pet
- Breed
- Species
- Weight History
- Photo
- Tags

---

## Relationships

```text
Pet
│
├── Health Records
├── Vaccinations
├── Medications
├── Documents
├── Timeline
├── Notes
├── Reminders
└── AI Insights
```

---

## Business Rules

- Every pet belongs to one workspace.
- Pet names are not globally unique.
- A pet may have multiple owners.
- A pet cannot exist without a workspace.

---

## Transaction Boundary

Atomic operations include:

- Pet registration
- Pet profile updates
- Pet archival

---

## Lifecycle

```text
Created
    │
Active
    │
Archived
    │
Deleted
```

---

## Future Expansion

- Microchip Integration
- Wearable Devices
- Insurance
- Veterinary Profiles

---

# 13. Health Aggregate

## Purpose

Stores all medical information associated with a pet.

---

## Aggregate Root

**Health Record**

---

## Owned Entities

- Health Record
- Diagnosis
- Allergy
- Surgery
- Chronic Condition
- Observation

---

## Relationships

```text
Pet
│
└── Health Records
      ├── Diagnosis
      ├── Allergies
      ├── Conditions
      └── Surgeries
```

---

## Business Rules

- Every health record belongs to one pet.
- Historical records remain immutable.
- Corrections create revisions rather than overwriting history.

---

## Transaction Boundary

- Create health record
- Update diagnosis
- Record surgery

---

## Future Expansion

- Veterinary integration
- Laboratory results
- Imaging
- Prescription syncing

---

# 14. Vaccination Aggregate

## Purpose

Tracks vaccination schedules and historical immunization records.

---

## Aggregate Root

**Vaccination Record**

---

## Owned Entities

- Vaccination
- Vaccine Type
- Booster Schedule

---

## Relationships

```text
Pet
│
└── Vaccinations
      ├── Vaccine
      └── Booster
```

---

## Business Rules

- Vaccinations cannot be permanently deleted.
- Booster reminders are automatically generated.
- Historical vaccination records remain immutable.

---

## Transaction Boundary

- Record vaccination
- Schedule booster
- Complete booster

---

## Future Expansion

- Regional vaccination schedules
- Veterinary verification
- Digital certificates

---

# 15. Medication Aggregate

## Purpose

Tracks medications currently prescribed and previously administered.

---

## Aggregate Root

**Medication**

---

## Owned Entities

- Medication
- Dosage
- Schedule
- Administration Log

---

## Relationships

```text
Pet
│
└── Medications
      ├── Dosage
      ├── Schedule
      └── Logs
```

---

## Business Rules

- Medication history is preserved.
- Missed doses create timeline events.
- AI may generate adherence insights.

---

## Transaction Boundary

- Add medication
- Update dosage
- Mark dose complete

---

## Future Expansion

- Prescription imports
- Pharmacy integration
- Smart reminders

---

# 16. Document Aggregate

## Purpose

Stores metadata for uploaded documents while binary files remain in object storage.

---

## Aggregate Root

**Document**

---

## Owned Entities

- Document
- OCR Result
- AI Extraction
- File Version

---

## Relationships

```text
Pet
│
└── Documents
      ├── OCR
      ├── AI Extraction
      └── Versions
```

---

## Business Rules

- Files are immutable.
- New uploads create new versions.
- OCR processing is asynchronous.
- AI extraction never blocks uploads.

---

## Transaction Boundary

- Upload document
- Create metadata
- Publish processing events

---

## Future Expansion

- Medical classification
- Automatic tagging
- Cross-document search

---

# 17. Timeline Aggregate

## Purpose

Provides a chronological history of all significant events related to a pet.

---

## Aggregate Root

**Timeline Event**

---

## Owned Entities

- Timeline Event
- Event Category
- AI Summary

---

## Relationships

```text
Pet
│
└── Timeline
      ├── Health
      ├── Vaccinations
      ├── Medications
      ├── Documents
      └── AI Events
```

---

## Business Rules

- Timeline events are append-only.
- Existing events are never modified.
- AI summaries may reference multiple events.

---

## Transaction Boundary

- Publish event
- Generate summary

---

## Future Expansion

- Smart timelines
- AI-generated health narratives

---

# 18. Reminder Aggregate

## Purpose

Schedules and manages reminders for recurring pet care activities.

---

## Aggregate Root

**Reminder**

---

## Owned Entities

- Reminder
- Reminder Schedule
- Reminder Completion

---

## Relationships

```text
Pet
│
└── Reminders
      ├── Schedule
      └── Completion
```

---

## Business Rules

- Recurring reminders generate future occurrences.
- Completed reminders remain in history.
- AI may recommend additional reminders.

---

## Transaction Boundary

- Create reminder
- Complete reminder
- Reschedule reminder

---

## Future Expansion

- Smart scheduling
- Calendar integrations
- Voice assistants

---

# 19. Notification Aggregate

## Purpose

Manages communication delivered to users.

---

## Aggregate Root

**Notification**

---

## Owned Entities

- Notification
- Delivery Status
- Delivery Channel

---

## Relationships

```text
Workspace
│
└── Notifications
      ├── Email
      ├── Push
      └── In-App
```

---

## Business Rules

- Notifications are immutable after delivery.
- Delivery failures are retried.
- Duplicate notifications are suppressed.

---

## Transaction Boundary

- Queue notification
- Deliver notification
- Update delivery status

---

## Future Expansion

- SMS
- WhatsApp
- Slack
- Webhooks

---

# 20. Billing Aggregate

## Purpose

Tracks subscriptions, invoices, payments, and billing history.

---

## Aggregate Root

**Subscription**

---

## Owned Entities

- Subscription
- Invoice
- Payment
- Plan
- Usage Record

---

## Relationships

```text
Workspace
│
└── Subscription
      ├── Plan
      ├── Invoice
      ├── Payment
      └── Usage
```

---

## Business Rules

- Billing records are immutable.
- Subscription upgrades are versioned.
- Failed payments generate retry workflows.

---

## Transaction Boundary

- Create subscription
- Process payment
- Renew subscription
- Cancel subscription

---

## Future Expansion

- Multi-currency support
- Enterprise billing
- Tax management
- Usage-based pricing

---

**End of Part 2**

**Next:** Part 3 covers the platform-level persistence architecture, including:

- AI Aggregate
- Search Aggregate
- Audit Logging
- Event Store
- AI Memory
- Embeddings
- Index Strategy
- Naming Conventions
- Soft Deletes
- Transactions
- Performance
- Backup Strategy
- Scaling Strategy
- Future Database Expansion
- Database Summary

# 21. AI Aggregate

## Purpose

The AI Aggregate provides persistent storage for all Artificial Intelligence interactions within PetSync AI.

Unlike traditional applications where AI is stateless, PetSync AI treats AI as a long-lived platform capability with memory, history, cost tracking, provider abstraction, and contextual awareness.

This aggregate enables:

- Personalized AI experiences
- Conversation continuity
- Prompt versioning
- Usage analytics
- Cost monitoring
- Provider switching
- AI observability

---

## Aggregate Root

**AI Session**

An AI Session represents a logical interaction between a user and the AI platform.

Every conversation, recommendation, or generated insight belongs to a session.

---

## Owned Entities

- AI Session
- AI Conversation
- AI Message
- AI Memory
- AI Recommendation
- AI Prompt
- AI Provider
- AI Model
- AI Usage
- AI Cost
- AI Feedback
- AI Context

---

## Relationships

```text
Workspace
│
└── AI Session
      │
      ├── Conversations
      ├── Messages
      ├── Memory
      ├── Recommendations
      ├── Prompt History
      ├── Provider
      ├── Model
      ├── Usage
      ├── Cost
      └── Feedback
```

---

## Business Rules

- Every AI session belongs to one workspace.
- Conversations are immutable.
- AI messages are append-only.
- Prompt history is versioned.
- Every provider response is auditable.
- AI costs are permanently recorded.
- AI memory never replaces business data.

---

## Transaction Boundary

Atomic operations include:

- Start AI session
- Save conversation
- Store AI response
- Record provider usage
- Save AI feedback

---

## Lifecycle

```text
Session Created
        │
Conversation Started
        │
Messages Added
        │
Memory Updated
        │
Archived
```

---

## Future Expansion

- Multi-agent collaboration
- Autonomous workflows
- Long-term memory graphs
- Personalized reasoning
- AI marketplace

---

# 22. Search Aggregate

## Purpose

Provides centralized search capabilities across structured and unstructured platform data.

Search combines relational queries with semantic retrieval.

---

## Aggregate Root

**Search Index**

---

## Owned Entities

- Search Index
- Search Document
- Search Metadata
- Search Ranking
- Search Analytics

---

## Relationships

```text
Workspace
│
├── Documents
├── Timeline
├── Notes
├── AI Memory
└── Search Index
```

---

## Business Rules

- Search indexes are eventually consistent.
- Search does not own business data.
- Search indexes are regenerated when source data changes.
- Failed indexing jobs are retried automatically.

---

## Transaction Boundary

- Index document
- Update embeddings
- Refresh search index

---

## Future Expansion

- Multilingual search
- Voice search
- Image search
- Medical terminology search

---

# 23. Audit Logging

## Purpose

Audit logs provide a permanent, immutable history of significant system activity.

Audit records support:

- Compliance
- Security investigations
- Troubleshooting
- Operational monitoring
- User accountability

---

## Logged Events

Examples include:

- Login
- Logout
- Permission changes
- Workspace updates
- Pet creation
- Medical updates
- Billing events
- AI requests
- AI responses
- File uploads

---

## Audit Record Structure

Every audit record contains:

- Event ID
- Timestamp
- User ID
- Workspace ID
- Entity Type
- Entity ID
- Action
- Previous Value
- New Value
- Request ID
- Correlation ID
- IP Address
- User Agent

---

## Audit Principles

- Immutable
- Append-only
- Tamper-resistant
- Searchable
- Retained according to policy

---

# 24. Event Store

## Purpose

The Event Store records every significant domain event published by the application.

Unlike audit logs, events drive asynchronous workflows.

---

## Event Categories

### Domain Events

Examples:

- PetCreated
- DocumentUploaded
- ReminderCompleted
- VaccinationRecorded

---

### AI Events

Examples:

- SummaryGenerated
- InsightCreated
- EmbeddingGenerated

---

### Infrastructure Events

Examples:

- CacheInvalidated
- FileUploaded
- QueueCompleted

---

### Integration Events

Examples:

- EmailSent
- PaymentSucceeded
- WebhookDelivered

---

## Event Flow

```text
Business Action
        │
Publish Event
        │
Event Store
        │
Event Bus
        │
Workers
        │
Consumers
```

---

## Event Principles

- Immutable
- Ordered
- Replayable
- Traceable
- Independently processed

---

# 25. AI Memory

## Purpose

AI Memory stores contextual information that improves future AI interactions.

It is not intended to replace business entities.

Instead, it enriches AI reasoning.

---

## Memory Sources

- Previous conversations
- Pet profile
- Medical history
- User preferences
- Timeline
- AI summaries

---

## Memory Types

### Short-Term Memory

Valid only during an active AI session.

---

### Long-Term Memory

Persists across sessions.

Examples:

- Preferred communication style
- Frequently referenced pets
- Historical context

---

### Semantic Memory

Generated from embeddings and vector retrieval.

---

## Memory Principles

- Privacy-first
- User-controlled
- Context-aware
- Versioned
- Expirable

---

# 26. Embeddings

## Purpose

Embeddings enable semantic understanding of platform content.

Embeddings transform textual information into vector representations.

---

## Embedded Content

- Medical Reports
- Timeline Events
- Notes
- AI Conversations
- Recommendations
- Documents

---

## Embedding Pipeline

```text
Content
    │
Chunking
    │
Embedding Model
    │
Vector Generation
    │
Vector Storage
    │
Similarity Search
```

---

## Business Rules

- Embeddings never replace source data.
- Embeddings are regenerated after meaningful content changes.
- Failed embeddings are automatically retried.

---

# 27. Index Strategy

Indexes improve read performance while maintaining acceptable write performance.

---

## Primary Indexes

Every entity includes:

- Primary Key
- Foreign Keys
- Unique Constraints

---

## Secondary Indexes

Frequently queried fields should be indexed.

Examples:

- Email
- Workspace ID
- Pet ID
- Reminder Date
- Created Date
- Updated Date
- Status

---

## Full-Text Indexes

Used for:

- Notes
- Timeline
- Documents
- AI Conversations

---

## Vector Indexes

Used for semantic retrieval.

Examples:

- Document similarity
- AI context retrieval
- Recommendation engine

---

# 28. Naming Conventions

Consistency improves maintainability.

---

## Tables

Use singular nouns.

Examples:

- user
- workspace
- pet
- document
- vaccination

---

## Primary Keys

Use:

```text
id
```

---

## Foreign Keys

Examples:

```text
workspace_id

user_id

pet_id

document_id
```

---

## Timestamps

Every primary entity includes:

```text
created_at

updated_at

deleted_at
```

---

## Status Fields

Use enumerations instead of free-text values.

Examples:

- ACTIVE
- ARCHIVED
- PENDING
- COMPLETED

---

# 29. Soft Deletes

Business entities should rarely be permanently deleted.

Instead, they are marked using:

```text
deleted_at
```

Benefits include:

- Recovery
- Auditability
- Historical reporting
- AI context preservation

Hard deletes should be limited to administrative maintenance.

---

# 30. Transactions

Critical business operations execute inside database transactions.

Examples:

- User registration
- Workspace creation
- Pet creation
- Billing
- Subscription renewal

Long-running AI operations should execute outside transactions.

---

## Transaction Principles

- Short-lived
- Atomic
- Consistent
- Isolated
- Durable

---

# 31. Performance

Performance considerations include:

- Proper indexing
- Query optimization
- Pagination
- Caching
- Lazy loading
- Connection pooling
- Background processing

Heavy analytical queries should never impact transactional workloads.

---

# 32. Backup Strategy

## Objectives

Protect against:

- Data corruption
- Accidental deletion
- Infrastructure failure
- Disaster recovery scenarios

---

## Backup Components

- PostgreSQL backups
- Object Storage versioning
- Redis persistence
- Vector index recreation
- Configuration backups

---

## Backup Principles

- Automated
- Encrypted
- Verified
- Versioned
- Regularly tested

---

# 33. Scaling Strategy

The persistence architecture supports incremental scaling.

---

## Vertical Scaling

Suitable for early platform growth.

Examples:

- Additional CPU
- More memory
- Faster storage

---

## Horizontal Scaling

Future strategies include:

- Read replicas
- Partitioning
- Worker scaling
- Dedicated AI databases
- Separate search infrastructure

---

## Future Database Evolution

Potential future improvements include:

- Database sharding
- Dedicated vector databases
- Data warehouses
- Analytics databases
- Regional deployments

---

# 34. Future Database Expansion

The database has been intentionally designed to accommodate future platform capabilities.

Potential additions include:

- Veterinary Portal
- Insurance Platform
- Marketplace
- Wearable Devices
- IoT Sensors
- AI Agents Marketplace
- Family Collaboration
- Multi-Tenant Organizations
- Research Platform

These features can be introduced without redesigning existing aggregates.

---

# 35. Database Summary

PetSync AI employs a domain-driven, AI-first persistence architecture built around business aggregates rather than isolated database tables.

The architecture combines:

- PostgreSQL for transactional consistency
- Redis for caching and coordination
- Object Storage for binary assets
- pgvector for semantic retrieval

This polyglot persistence strategy enables scalability, maintainability, AI integration, and future extensibility while preserving strong data integrity.

The database is designed to support long-term platform evolution through clear aggregate boundaries, immutable history, event-driven processing, and provider-independent AI capabilities.

---

# Related Documents

- SYSTEM_DESIGN.md
- API.md
- AI_ARCHITECTURE.md
- SECURITY.md
- STORAGE.md
- PERFORMANCE.md

---

**Document Status:** Approved

**End of Document**