# Database Management Context Document

## Relevance to This Codebase

Database Management is a critical area for this event-driven microservices platform due to its distributed nature and reliance on transactional guarantees. Each service (e.g., AccountService, InventoryService, OrchestratorService) is implicitly designed to manage its own data, typically within a dedicated database or schema to maintain autonomy and ensure data ownership.

The core of this system's reliability for event publishing hinges on the **Transactional Outbox Pattern**, primarily managed by the `OutboxProcessor`. This pattern requires a database transaction to atomically commit both business data changes and the recording of an outgoing event to an outbox table. Without robust database design, transaction management, and performance considerations, the reliability of event propagation and the consistency of the entire system would be severely compromised.

Understanding the database structures, migration strategies, and query patterns within each service is essential for ensuring data integrity, system performance, and the reliable functioning of the event-driven architecture.

## Current State Assessment

| Artifact            | Location/Pattern                                 | Maturity Level | Notes                                                                                                                                                                             |
| :------------------ | :----------------------------------------------- | :------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Database Engine** | (Inferred: Relational DB)                        | Established    | The system's design with a transactional outbox pattern strongly implies the use of a relational database for each service's primary data storage. Specific engine unknown.         |
| **ORM Framework**   | (Inferred: Entity Framework Core for C#)         | Established    | Given the C#/.NET tech stack, Entity Framework Core is the most probable ORM used within individual services for data access and mapping. No explicit configurations found.          |
| **Schema Definitions** | `*/Models/` directories (inferred)            | Low (unseen)   | Each service is expected to define its own data models (entities) within its `Models/` directory, which would map to database tables. Specific table definitions are not visible. |
| **Migrations**      | (Not explicitly identified)                      | Low (unseen)   | No explicit `Migrations/` folders or migration files (e.g., `*.sql`, `V*__*.sql`) were identified. A migration strategy is inferred to exist for schema evolution, but its implementation is unknown. |
| **Outbox Table**    | `OutboxProcessor/Models/` (inferred)             | Established    | The `OutboxProcessor` implies the existence of an `OutboxMessage` entity within each service's database, used to store events reliably before publishing.                            |
| **Connection Strings** | (Not explicitly identified)                      | Low (unseen)   | Database connection strings are expected to be configured per service, likely via environment variables or application configuration files (e.g., `appsettings.json`), but are not visible in the provided context. |

## What's Missing or Weak

| Gap                         | Impact                                                                                                 | Priority |
| :-------------------------- | :----------------------------------------------------------------------------------------------------- | :------- |
| **Explicit Schema Definitions** | Difficulty in understanding database structure, relationships, and data types without code inspection or reverse engineering.                                                                                   | High     |
| **Visible Migration Strategy**  | Lack of insight into how schema changes are managed, potential for manual errors, and difficulty in ensuring backward compatibility or reversibility.                                                                | High     |
| **Connection Pooling/Timeout Config** | Unoptimized database connections can lead to performance bottlenecks or resource exhaustion under load.                                                                                             | Medium   |
| **Index Definitions**           | Without visible index definitions, query performance optimization is unverified; potential for slow queries.                                                                                                   | Medium   |
| **Data Integrity Constraints**  | Absence of visible foreign key, unique, and check constraints means data integrity relies solely on application logic, increasing risk of inconsistent data.                                                      | High     |
| **Backup & Recovery Strategy**  | No visible plan for database backups, point-in-time recovery, or testing, posing a significant data loss risk.                                                                                                | High     |

## Key Patterns and Conventions

| Pattern                     | Where Used                                                                        | Notes                                                                                                                                                                                                                                                |
| :-------------------------- | :-------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Transactional Outbox Pattern** | `OutboxProcessor/OrderOutboxWorker.cs`, each service's internal database | Ensures atomicity between business logic updates and event publishing. Events are first written to an `OutboxMessage` table within the same database transaction as business data changes, then picked up by the `OutboxProcessor`.                 |
| **Database Per Service**    | Implied across `AccountService`, `InventoryService`, `OrchestratorService`      | Each microservice is expected to own and manage its own database (or schema). This promotes service autonomy and reduces coupling.                                                                                                                       |
| **ORM for Data Access**     | Inferred within `AccountService`, `InventoryService`, `OrchestratorService`     | Services are likely using an ORM (e.g., Entity Framework Core) to interact with their respective databases, mapping C# entities to database tables and providing an abstraction over raw SQL queries.                                                       |
| **Repository Pattern**      | `*/Models/` (inferred)                                                          | Given C# best practices for microservices, a repository pattern or similar abstraction is likely used to encapsulate data access logic within each service, making it easier to manage data operations and potentially swap underlying ORMs or databases. |

## Risks and Hotspots

| Risk                                     | Location/Detection                                                | Severity | Mitigation                                                                                                                                                                                            |
| :--------------------------------------- | :---------------------------------------------------------------- | :------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Transactional Outbox Failure**         | `OutboxProcessor/OrderOutboxWorker.cs`, Service databases         | High     | Any issue preventing `OutboxProcessor` from reading or publishing events from service outbox tables can halt event flow, leading to data inconsistencies across the system. Ensure robust monitoring and error handling. |
| **Data Inconsistency (Missing Constraints)** | All service databases (inferred)                                  | High     | Without database-level constraints (FKs, UNIQUE, NOT NULL), data integrity relies solely on application code, which is prone to errors, especially as the system evolves. Implement constraints at the DB level.      |
| **Performance Bottlenecks**              | Any service performing complex queries; lack of index awareness   | Medium   | Unoptimized queries, missing indexes, or N+1 patterns can degrade system performance. Review ORM query patterns and ensure appropriate indexing for critical operations.                                        |
| **Unknown Schema Evolution**             | Absence of visible migration files                                | Medium   | Changes to database schemas could be manual, inconsistent, or lack proper versioning, leading to deployment issues or data loss. Standardize a migration tool and process.                                      |
| **Data Loss (No Backup/Recovery Plan)**  | Overall system, specifically related to each service's database | High     | Critical data loss could occur without a defined and tested backup and recovery strategy for each service's persistent data store.                                                                                  |

## AI Agent Guidelines

### DO
- **Always confirm database interactions**: When making changes related to data persistence or event publishing, always investigate the specific service's `Worker.cs` and related `Models/` directories for ORM entity definitions and how they interact with the database.
- **Consult `OutboxProcessor` for event publishing**: For any change affecting how events are published from a service, always review `OutboxProcessor/OrderOutboxWorker.cs` and the inferred `OutboxMessage` entity to understand the transactional outbox mechanism.
- **Consider database per service principle**: Assume that each service (AccountService, InventoryService, OrchestratorService) manages its own isolated database schema and data. Changes to one service's schema should not directly impact others.

### DON'T
- **Do not assume shared database tables**: Avoid introducing shared tables or direct foreign key relationships between different service databases, as this violates the microservice principle of data ownership.
- **Do not bypass the transactional outbox for critical events**: Do not attempt to publish domain events directly to a message broker without first persisting them in the service's outbox table within a database transaction.
- **Do not hardcode database connection details**: Avoid embedding connection strings or credentials directly in source code.

### ALWAYS CHECK
- **Service-specific `Models/` directories**: Always check the `Models/` directory within each service (e.g., `AccountService/Models/`, `InventoryService/Models/`) for the ORM entities that define its database schema and relationships.
- **Database transaction boundaries**: Always check where database transactions are initiated and committed, especially in `Worker.cs` files or business logic, to ensure atomicity, particularly with the outbox pattern.
- **Query performance**: Always check for potential N+1 query patterns or inefficient data retrieval when implementing new features or modifying existing data access logic.

## Related Context Areas

*   **backend-development**: Provides context on how services implement data access logic, ORM usage patterns, and repository implementations.
*   **architecture**: Offers insight into the overall data architecture, service boundaries, and data ownership principles in the microservices landscape.
*   **security**: Relevant for understanding database access control, data encryption at rest/in transit, and PII storage considerations.
*   **site-reliability**: Crucial for database monitoring, performance tuning, connection health, and disaster recovery planning.
*   **devops-infrastructure**: Covers the deployment, provisioning, and management of database instances, including containerization (e.g., Docker) and cloud services.

## Key Files

| Path/Pattern                                     | Purpose                                                         | Notes                                                                                                     |
| :----------------------------------------------- | :-------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------- |
| `OutboxProcessor/OrderOutboxWorker.cs`           | Orchestrates the transactional outbox pattern.                  | Reads events from service outbox tables and publishes them to the message broker.                           |
| `OutboxProcessor/Models/`                        | Defines the `OutboxMessage` entity.                             | Describes the structure of events stored in service outbox tables.                                          |
| `*/Worker.cs` (e.g., `AccountService/Worker.cs`) | Service-specific background processing, often including DB writes. | Entry point for event consumers or background tasks that interact with the service's database.              |
| `*/Models/` (e.g., `InventoryService/Models/`)   | Defines ORM entities for each service.                          | Expected location for database table definitions and relationships specific to each microservice.           |
| `*.csproj` files (within each service)           | Project dependencies, including ORM libraries.                  | Implies the use of specific ORM frameworks (e.g., `Microsoft.EntityFrameworkCore` packages).                |
| `appsettings.json` (inferred)                    | Configuration files for database connection strings.            | Expected to hold environment-specific database connection details and other runtime configurations.         |