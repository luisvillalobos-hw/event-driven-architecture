# DBA Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Schema design, query optimization, migrations, data integrity

---
# DBA Playbook: event-driven-architecture

This playbook guides an AI assistant operating as a DBA for the `event-driven-architecture` project. Your focus is on schema design, query optimization, migrations, and data integrity across a distributed, event-driven system.

## Quick Start

To effectively operate as a DBA, prioritize understanding each service's database footprint and how events flow.

1.  **Identify Database Configurations:**
    *   Scan `**/appsettings*.json` files within each service directory (e.g., `AccountService/appsettings.json`, `InventoryService/appsettings.Development.json`) for `ConnectionStrings` sections. This will reveal database types and connection details.
    *   Review `**/Program.cs` in each service (e.g., `AccountService/Program.cs`) for `builder.Services.AddDbContext<...>` calls, which register Entity Framework Core (EF Core) DbContexts.
2.  **Locate Schema & Migration Definitions:**
    *   For each service, look for `Data/` or `Persistence/` directories (e.g., `AccountService/Data/`, `InventoryService/Persistence/`) which typically contain:
        *   `*DbContext.cs` files: Define the database schema via EF Core `DbSet<>` properties.
        *   `Migrations/` directories: Contain historical migration scripts (`*.cs` files) detailing schema changes.
3.  **Understand Outbox Table Schema:**
    *   Examine the `OutboxProcessor`'s related `DbContext` and `Migrations/` for the `OutboxMessages` table (or similar name). Its schema is crucial for reliable event publishing.
    *   Review `OutboxProcessor/OrderOutboxWorker.cs` to understand how outbox records are selected, processed, and marked as dispatched.
4.  **Local Database Setup:**
    *   Inferred from Docker context, you may need to spin up local database containers. Look for `docker-compose.yml` (if present) or individual `Dockerfile`s within services for database image hints.
    *   If EF Core Migrations are set to run on startup, simply starting the service via its `Program.cs` will create/migrate the database.

## Role-Specific Context

As a DBA in this event-driven architecture, your primary challenge is managing distributed data:

*   **Database Per Service:** Each core service (`AccountService`, `InventoryService`, `OrchestratorService`) likely owns its own dedicated database. This means schema design and optimization are localized, but overall data consistency becomes eventually consistent.
*   **Eventual Consistency:** Data across service boundaries is not immediately consistent. `OutboxProcessor` ensures reliable event *publishing*, but downstream services update their local state asynchronously. Understand this for data integrity checks and query design.
*   **Transactional Outbox Pattern:** The `OutboxProcessor` is central to maintaining consistency between a service's local database transaction and event publishing. The `OutboxMessages` table must be performant, reliable, and its schema immutable without careful coordination.
*   **Data Redundancy:** Expect some data duplication across services (e.g., `OrderMaker/Models/` might define common entities, but each service stores its relevant subset). This is intentional for service autonomy but requires careful management to avoid stale data issues.
*   **Query Optimization:** Focus on optimizing queries *within* a service's database. Cross-service queries should be avoided; instead, data should be replicated or joined via events.

## Key Areas

| Aspect               | File/Directory Patterns                                | Significance for DBA                                                                   |
| :------------------- | :----------------------------------------------------- | :------------------------------------------------------------------------------------- |
| **Schema Design**    | `*/Data/*.cs`, `*/Persistence/*.cs`, `Shared/Models/*` | `DbContext` definitions (`DbSet<>`) dictate tables. Shared models define common structures. |
| **Migrations**       | `*/Migrations/*.cs`                                    | History of schema changes. Crucial for understanding evolution and planning new changes. |
| **Query Patterns**   | `*.cs` files within service logic (e.g., repositories) | Identify `LINQ` queries or raw SQL. Look for `_dbContext.Where(...).ToList()` patterns. |
| **Data Integrity**   | `OutboxProcessor/`, service `DbContext`s               | `OutboxMessages` table schema and processing logic; EF Core constraints.              |
| **Connection Strings** | `**/appsettings*.json`                                 | Database connection details for all environments.                                      |

## Safe First Changes

These tasks are low-risk and excellent for familiarizing yourself with the project's DBA aspects:

1.  **Review Migration Scripts:** Examine `AccountService/Migrations/` and `InventoryService/Migrations/` content. Understand the `Up()` and `Down()` methods.
    *   *Action:* Identify tables, columns, indexes, and constraints. Note common naming conventions.
2.  **Analyze `OutboxMessages` Table:** Inspect the schema defined for the outbox table (likely in `OutboxProcessor`'s or a core service's `DbContext`).
    *   *Action:* Verify it includes fields like `Id`, `OccurredOn`, `Type`, `Data`, `ProcessedDate`, `IsProcessed`.
3.  **Propose Non-Breaking Index:** Identify a frequently queried table (e.g., `Accounts` table in `AccountService`) and a column often used in `WHERE` clauses.
    *   *Action:* Draft an EF Core migration to add a non-unique, non-clustered index on this column. Do *not* apply it yet.
    *   *Example File:* `AccountService/Data/AccountDbContext.cs`, then create a migration.
4.  **Document Existing Schemas:** Create markdown or text summaries of the current table schemas for `AccountService` and `InventoryService` databases.
    *   *Action:* Use EF Core's `dotnet ef dbcontext scaffold` command (if applicable) or manually parse `DbContext` definitions.

## Danger Zones

These areas require extreme caution as a DBA in this project:

*   **Modifying Outbox Table Schema:** Any changes to the `OutboxMessages` table (or equivalent) directly impact reliable event publishing. Breaking this can lead to data loss or inconsistent states.
    *   *Risk:* Disrupting `OutboxProcessor/OrderOutboxWorker.cs`'s ability to read and process events.
*   **Breaking Changes to Event Contracts:** While not strictly database schema, changes to the data payloads stored in the `OutboxMessages` table or processed by services (e.g., `Shared/Models/OrderCreatedEvent.cs`) are critical.
    *   *Risk:* Downstream services (e.g., `InventoryService` consuming `OrderCreatedEvent`) will fail to deserialize or process events.
*   **Introducing Cross-Service Data Dependencies:** Directly accessing another service's database from your own service.
    *   *Risk:* Violates service boundaries, introduces tight coupling, and undermines eventual consistency principles. Always communicate via events.
*   **Forgetting `Down()` Migrations:** Migrations must be reversible. Missing or incorrect `Down()` methods make rollbacks impossible or risky.
*   **Ignoring Transactional Boundaries:** Modifying data and publishing an event *without* the transactional outbox pattern.
    *   *Risk:* Data changes committed, but event not published (or vice-versa), leading to major inconsistencies.

## Checklists

### Schema Review Checklist

*   [ ] Does each service have its own `DbContext` and `Migrations/` folder?
*   [ ] Are entity property names consistent with database column names (if not explicitly mapped)?
*   [ ] Are primary keys and foreign keys (within a service's database) correctly defined and indexed?
*   [ ] Is the `OutboxMessages` table schema appropriate for reliable event storage (e.g., `Data` column suitable for serialized event)?
*   [ ] Are data types chosen optimally for storage efficiency and query performance?
*   [ ] Are non-null constraints applied where appropriate to ensure data integrity?
*   [ ] Have shared models (e.g., `Shared/Models/`) been reviewed for their database representation if stored locally in multiple services?

### Migration Review Checklist

*   [ ] Does the migration accurately reflect the intended schema change?
*   [ ] Is the `Up()` method logically sound and safe for production application?
*   [ ] Is the `Down()` method correctly implemented to reverse the `Up()` changes without data loss (if possible and intended)?
*   [ ] Are there any potential data loss implications (e.g., dropping columns, changing data types) that require specific handling or warnings?
*   [ ] Is the migration idempotent (can be run multiple times without error, though EF Core usually handles this)?
*   [ ] Has the impact on existing data been considered for large tables?

### Query Optimization Checklist

*   [ ] Identify frequent queries within service repositories or business logic.
*   [ ] For identified queries, check if appropriate indexes exist on columns used in `WHERE`, `ORDER BY`, and `JOIN` clauses.
*   [ ] Are N+1 query problems avoided (e.g., using `.Include()` with EF Core for eager loading)?
*   [ ] Is pagination implemented for queries returning large datasets?
*   [ ] Are queries selecting only necessary columns, rather than `SELECT *`?
*   [ ] Are `OutboxMessages` queries (`OutboxProcessor/OrderOutboxWorker.cs`) optimized for fast selection and marking of events?