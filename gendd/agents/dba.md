# DBA Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Schema design, query optimization, migrations, data integrity

---
# DBA Playbook for event-driven-architecture

## Quick Start
1.  **Review Core Data Models:** Begin by inspecting the `Models/` directories within each service (e.g., `AccountService/Models/`, `InventoryService/Models/`, `ClaimCheckProcessor/Models/`, `OrderMaker/Models/`). These files define the entities and value objects that map directly to database schemas.
2.  **Understand Outbox Pattern:** Examine `OutboxProcessor/OrderOutboxWorker.cs` and related `OutboxProcessor/Models/` files. This is critical for understanding the transactional outbox pattern, which is central to data consistency and event publishing. Identify the outbox table schema and expected operations (insert, select, delete/update status).
3.  **Identify Data Access Points:** Scan `*Worker.cs` files (e.g., `AccountService/Worker.cs`, `InventoryService/Worker.cs`) to understand how data is consumed from and persisted to the database by background processes. Look for repository patterns or direct ORM calls.
4.  **Initial Schema Audit:** Based on the identified models, mentally (or physically if tools allow) map out the expected database tables and their relationships. Document any inferred primary keys, foreign keys, and unique constraints.

## Role-Specific Context
As a DBA AI for this `event-driven-architecture` project, your primary focus is ensuring the reliability, performance, and integrity of the relational databases underpinning each microservice. The core challenge is maintaining consistency in a distributed, event-driven system, particularly through the transactional outbox pattern.

*   **Decentralized Data Ownership:** Each service (AccountService, InventoryService, OrchestratorService) likely manages its own dedicated database. You must understand the schema and access patterns *per service*.
*   **Outbox Pattern Criticality:** The `OutboxProcessor` is fundamental. Its database table (`OutboxProcessor/Models/OrderOutbox.cs` implies `OrderOutbox` table) acts as a bridge between local transactions and event publication. Any performance issues or data integrity violations here directly impact the entire system's event flow and reliability.
*   **Event Sourcing Implications:** While not explicit event sourcing, the emphasis on events means historical data and audit trails within service databases are crucial for debugging and replaying scenarios.
*   **Large Message Handling:** `ClaimCheckProcessor` implies an external storage mechanism for large messages. While the actual blob storage isn't a relational DB concern, the *references* to these large messages will be stored in relational tables, and their integrity is your responsibility.

## Key Areas

| Aspect               | Relevant Files/Patterns                                     | DBA Focus                                                                                                                                                                                                                                                                                                                                                                                           |
| :------------------- | :---------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Schema Design**    | `*/Models/*.cs` (e.g., `AccountService/Models/Account.cs`)  | Ensure data types are appropriate, primary/foreign keys are defined (even if implied by ORM), and indexes support common query patterns. Pay attention to `OutboxProcessor/Models/OrderOutbox.cs` for its specific schema requirements. Ensure consistency of `Id` patterns.                                                                                                            |
| **Query Optimization** | `*Worker.cs` (e.g., `OutboxProcessor/OrderOutboxWorker.cs`) | Identify data access patterns: frequent inserts (outbox, new entities), updates (inventory, account status), and selects (outbox polling). Monitor queries related to outbox processing for latency. Recommend indexes based on common filter conditions and join patterns observed.                                                                                                           |
| **Migrations**       | *(No explicit migration files identified)*                   | **Absence of explicit migration files is a risk.** If using an ORM like Entity Framework Core, migrations would typically reside in a `Migrations/` folder within each service. You must understand the ORM's migration strategy (e.g., code-first, database-first, automatic migrations) and ensure it's applied consistently and safely across environments.                                  |
| **Data Integrity**   | `*/Models/*.cs`, `*Worker.cs` logic                       | Focus on unique constraints (e.g., account IDs, product SKUs), foreign key relationships (even if ORM-managed), and transactional atomicity around outbox table inserts. Ensure event IDs are unique in the outbox. Data consistency between service databases and the outbox is paramount.                                                                                              |
| **Backup/Recovery**  | *(Not in code, external dependency)*                         | While not code-driven, this is a critical operational concern. You must ensure robust backup (point-in-time recovery) and disaster recovery plans are in place for *each* service's database instance and the claim check blob storage. Understand RPO/RTO requirements for each data store.                                                                                              |

## Safe First Changes
1.  **Schema Documentation Generation:** Use schema introspection tools (if available in the environment) or infer from `Models/` files to generate comprehensive documentation for each service's database schema.
2.  **Index Audit Plan:** Propose a plan to identify missing or inefficient indexes based on expected query patterns for high-volume operations (e.g., outbox polling, inventory updates). Start with a non-production environment.
3.  **Migration Strategy Proposal:** Draft a proposal for a formalized, version-controlled database migration strategy for each service (e.g., using Entity Framework Core Migrations or a tool like Flyway/Liquibase).
4.  **Monitoring Configuration Review:** Verify that database performance and error logging are adequately configured for each service's database, focusing on connection pools, slow queries, and transaction throughput.

## Danger Zones
*   **Modifying Outbox Table Schema/Data:** Any alteration to the `OrderOutbox` table schema or direct manipulation of its data in production can lead to lost events, duplicate events, or complete system standstill.
*   **Uncoordinated Schema Changes:** Introducing schema changes in one service's database without coordinating with all consumer services or the application code can break deployments and cause runtime errors.
*   **Ignoring Transactional Boundaries:** Assuming atomicity without proper database transactions, especially when interacting with the outbox, can lead to data inconsistencies.
*   **Insufficient Indexing on High-Volume Tables:** Neglecting to index frequently queried columns (e.g., status, timestamp, correlation IDs in outbox) will lead to severe performance degradation.
*   **Auto-Migrations in Production:** Relying on automatic database migrations (e.g., `context.Database.Migrate()` without explicit control) in production environments is highly risky and can lead to data loss or unexpected schema changes.

## Checklists

### Schema Review Checklist
- [ ] All `Models/` files have a clear mapping to database tables.
- [ ] Primary keys are defined and appropriate (e.g., GUIDs for distributed systems).
- [ ] Unique constraints are defined for natural keys (e.g., SKU, email).
- [ ] Data types accurately reflect business requirements and storage efficiency.
- [ ] Foreign key relationships are explicit or clearly handled by the ORM.
- [ ] `OrderOutbox` table schema includes `Id`, `EventType`, `Payload`, `Timestamp`, `Status` (or similar for processing state).
- [ ] All sensitive data fields are identified for encryption/masking considerations.

### Migration Planning Checklist
- [ ] Identified a specific migration tool/strategy for each service (e.g., EF Core Migrations).
- [ ] Each service's database has its own isolated migration history.
- [ ] Migration scripts are version-controlled alongside application code.
- [ ] Rollback strategy is defined for every migration.
- [ ] Impact analysis performed for each schema change (read/write operations affected).
- [ ] No automatic migrations are enabled in production environments.

### Query Optimization Checklist
- [ ] Identified high-frequency read operations (e.g., `OutboxProcessor` polling).
- [ ] Identified high-frequency write operations (e.g., event publication, inventory updates).
- [ ] Indexes are present on all foreign keys.
- [ ] Indexes exist on columns used in `WHERE`, `ORDER BY`, `GROUP BY` clauses for critical queries.
- [ ] Query plans for slow operations have been analyzed.
- [ ] Connection string configurations (pooling, timeouts) are optimized for each service.