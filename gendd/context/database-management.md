# Database Management Context Document

## Relevance to This Codebase

In this event-driven microservices architecture, database management is a critical area. Each microservice is expected to own its data, implying the presence of multiple, distinct databases. The transactional outbox pattern, heavily utilized by the `OutboxProcessor`, relies on robust database transactions and careful management of an `OutboxMessages` table within each service's persistence layer to ensure reliable event publishing. Effective database management is essential for data consistency (achieved eventually across services), performance of individual services, and ensuring the integrity of business operations.

## Current State Assessment

| Artifact/Pattern            | Location/Example                                         | Maturity Level (inferred) | Notes                                                                                                                                                                                                                                                                                                 |
| :-------------------------- | :------------------------------------------------------- | :------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Database Engine**         | (Not explicitly specified)                               | Functional                | Likely SQL Server, given .NET ecosystem prevalence, but could vary per service or environment.                                                                                                                                                                                                        |
| **ORM Framework**           | `*.csproj` (e.g., `AccountService/AccountService.csproj`) | High                      | Highly likely Entity Framework Core, as it's the standard for .NET applications. Each service would likely have its own `DbContext` and entity definitions.                                                                                                                                                 |
| **Entity Definitions**      | `*/Models/*.cs` or `*/Entities/*.cs`                     | High                      | Each service (e.g., `AccountService`, `InventoryService`, `OrchestratorService`) will contain its domain-specific entities (e.g., `Account`, `Product`, `Order`). The `OutboxProcessor` implies an `OutboxMessage` entity.                                                                                  |
| **Database Configuration**  | `*/appsettings.json`, `*/Program.cs`                     | High                      | Connection strings are expected in `appsettings.json` files for each service. `Program.cs` in each service will likely register its `DbContext` with dependency injection.                                                                                                                               |
| **Migration Strategy**      | (Not explicitly specified)                               | Functional                | Inferred to be Entity Framework Core Migrations, managed within each service's project structure (e.g., `AccountService/Migrations/`). These migrations would define schema changes for each service's dedicated database.                                                                             |
| **Transactional Outbox**    | `OutboxProcessor/`, `*/Data/OutboxMessage.cs`            | High                      | The `OutboxProcessor` strongly indicates the use of a transactional outbox pattern. This means each service likely has an `OutboxMessages` table in its database, and entities for these messages are defined within service-specific or shared data layers.                                             |
| **Repository Pattern**      | `*/Repositories/*.cs`                                    | Medium                    | While EF Core can be used directly, a repository pattern (e.g., `IAccountRepository`, `AccountRepository`) is often employed to abstract data access logic and facilitate testing. This would be defined per service.                                                                                    |
| **Data Integrity**          | Defined within entity configurations                     | Medium                    | Constraints (e.g., `[Required]`, `[StringLength]`, `[Unique]`, `[ForeignKey]`) are likely defined via EF Core fluent API or data annotations within entity configurations, translating to database-level constraints during migrations.                                                                   |

## What's Missing or Weak

| Gap/Weakness                     | Impact                                                            | Priority |
| :------------------------------- | :---------------------------------------------------------------- | :------- |
| **Detailed Indexing Strategy**   | Performance degradation for frequently queried fields; N+1 queries | Medium   |
| **Explicit Backup/Recovery**     | Data loss, extended downtime in case of disaster                  | High     |
| **Performance Considerations**   | Slow queries, connection pooling issues, lack of caching          | Medium   |
| **Cross-service Schema Evolution** | Potential for breaking changes between services due to schema updates without careful coordination | Medium |
| **Centralized Database Observability** | Difficulty in monitoring performance, health, and issues across multiple databases | Medium |

## Key Patterns and Conventions

| Pattern                     | Where Used                                                       | Notes                                                                                                                                              |
| :-------------------------- | :--------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Database-per-Service**    | `AccountService`, `InventoryService`, `OrchestratorService`      | Each microservice is expected to manage its own independent database schema and data, ensuring strong encapsulation.                               |
| **Transactional Outbox**    | `OutboxProcessor`, within each service's persistence layer       | Ensures atomicity between database transactions and publishing events to a message broker. Involves an `OutboxMessages` table in each service DB. |
| **ORM-centric Data Access** | All services (`*.csproj` references to EF Core packages)         | Data interaction is primarily handled through an Object-Relational Mapper (likely Entity Framework Core) for type safety and reduced boilerplate. |
| **EF Core Migrations**      | Inferred within each service's project structure (`Migrations/`) | Database schema evolution is managed through versioned code-based migrations, allowing for controlled changes.                                     |

## Risks and Hotspots

| Risk                               | Location/Detection                                                | Severity | Mitigation                                                                                                                                                                                          |
| :--------------------------------- | :---------------------------------------------------------------- | :------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Eventual Consistency Issues**    | Cross-service data dependencies; `OutboxProcessor` reliability    | High     | Robust event-driven design, idempotent consumers, clear understanding of data ownership, and thorough testing of event flows. Monitor `OutboxProcessor` health closely.                               |
| **Database Performance Bottlenecks** | Within individual service implementations (`*/Data/`, `*/Models/`) | Medium   | Regular query profiling, index analysis, proper use of async/await for I/O operations, and connection pooling configuration.                                                                        |
| **Schema Drift between Services**  | `Migrations/` folders across different services                   | Medium   | Clear communication and versioning strategies for shared data contracts (in `Shared/`) and API/event payloads. Coordinated deployments for breaking changes.                                         |
| **Connection String Mismanagement** | `appsettings.json`, environment variables                         | Medium   | Ensure sensitive connection details are stored securely (e.g., Azure Key Vault, environment variables) and not hardcoded.                                                                               |

## AI Agent Guidelines

| Category       | Guideline                                                                                                                                                                                                                                                                                               |
| :------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **DO**         | When working with a service's database, **always** assume it's an isolated data store. Do not assume shared tables or direct access to other service databases.                                                                                                                                        |
| **DO**         | When modifying entity definitions or database schema, **always** create or update Entity Framework Core Migrations within the relevant service's project (`*/Migrations/`).                                                                                                                               |
| **DO**         | When investigating data access, **always** refer to the service's `*/Models/` (or `*/Entities/`) directory for entity definitions and its `*/Data/` directory for `DbContext` and repository implementations.                                                                                          |
| **DO**         | When checking database configuration, **always** look in `*/appsettings.json` for connection strings and `*/Program.cs` for `DbContext` service registration.                                                                                                                                        |
| **DO NOT**     | **Never** introduce direct database calls or dependencies from one service to another service's database. Inter-service communication should solely occur via events or defined APIs.                                                                                                                   |
| **ALWAYS CHECK** | The `OutboxMessages` table definition and related logic within a service's `DbContext` if issues related to reliable event publishing or eventual consistency arise.                                                                                                                                |
| **ALWAYS CHECK** | For existing index definitions within EF Core migration files or entity configurations when troubleshooting query performance.                                                                                                                                                                        |

## Related Context Areas

*   **[backend-development](./backend-development.md)**: Data access patterns, ORM usage, repository implementations, and business logic interacting with databases.
*   **[architecture](./architecture.md)**: Overall data architecture, microservice data ownership principles, and consistency models.
*   **[security](./security.md)**: Database access control, connection string security, and data encryption at rest/in transit.
*   **[site-reliability](./site-reliability.md)**: Database monitoring, health checks, connection pooling, and performance tuning.
*   **[devops-infrastructure](./devops-infrastructure.md)**: Database provisioning, infrastructure as code for databases, and backup/recovery infrastructure.

## Key Files

| Path/Pattern                        | Purpose                                                               | Notes                                                                   |
| :---------------------------------- | :-------------------------------------------------------------------- | :---------------------------------------------------------------------- |
| `*/appsettings.json`                | Database connection string configuration                              | Environment-specific settings, often containing `ConnectionStrings` section. |
| `*/Program.cs`                      | Entity Framework Core `DbContext` registration                        | Configures services, including database context injection.              |
| `*/AccountService.csproj`           | Project file, indicates EF Core package references                    | `PackageReference` for `Microsoft.EntityFrameworkCore.*`.                 |
| `*/Models/*.cs` or `*/Entities/*.cs` | Database entity definitions                                         | C# classes mapping to database tables, potentially with data annotations. |
| `*/Data/*.cs`                       | `DbContext` implementations and potentially repository interfaces/classes | Central point for database interaction within a service.                |
| `*/Migrations/*.cs`                 | Entity Framework Core migration files                                 | Versioned database schema changes for each service's database.          |
| `OutboxProcessor/`                  | Contains logic for processing outbox messages                         | Critical for reliable event publishing from service databases.          |