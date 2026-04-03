## Data Ownership

Effective data ownership is crucial in a microservices architecture to maintain service autonomy, ensure data integrity, and manage complexity. Each service is responsible for its own persistent data, defining its schema, access patterns, and lifecycle.

### Data Entities & Ownership Boundaries

The following table outlines the planned data entities, their primary owner, storage type, and location. This establishes clear boundaries for data responsibility.

| Data Entity          | Owning Service                                   | Primary Storage Type      | Storage Location                  | Remarks                                                                                                                                                                                                                                                                                                                                                          |
| :------------------- | :----------------------------------------------- | :------------------------ | :-------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Account Data**     | `AccountService`                                 | Relational Database       | `AccountService` Database         | FACT (Confirmed ownership). HYPOTHESIS (Relational Database, common for account management).                                                                                                                                                                                                                                                         |
| **Inventory Data**   | `InventoryService`                               | Relational Database       | `InventoryService` Database       | FACT (Confirmed ownership). HYPOTHESIS (Relational Database, common for inventory management).                                                                                                                                                                                                                                                         |
| **Order/Workflow State** | `OrchestratorService`                            | Relational Database       | `OrchestratorService` Database    | FACT (Confirmed ownership). Stores current state and history of multi-service workflows (e.g., order fulfillment). HYPOTHESIS (Relational Database, common for workflow state and aggregates).                                                                                                                                                    |
| **Outbox Messages**  | Any service utilizing the Outbox Pattern (e.g., `AccountService`, `InventoryService`, `OrchestratorService`) | Relational Database       | Service's local database (outbox table) | FACT (Confirmed ownership and storage type). Transient state for reliable event publishing. `OutboxProcessor` reads from these tables but does not own the data.                                                                                                                                                                                      |
| **Claim Check Payloads** | Publisher of large message (e.g., `OrchestratorService` for large order details) | Object Storage            | Dedicated Object Store (e.g., S3, Azure Blob Storage) | FACT (Confirmed ownership and storage type). The actual large payload; `ClaimCheckProcessor` manages retrieval but not ownership.                                                                                                                                                                                                                                    |
| **Message Broker Queues/Topics** | Message Broker (external system)                 | Message Queue/Topic       | Message Broker instance           | HYPOTHESIS (Message Queue/Topic storage). Transient storage for events in transit. Not owned by a specific application service but managed by the Message Broker system.                                                                                                                                                                                 |

### Cross-Service Data Access Patterns

Direct access to another service's private database is **strictly prohibited**. All data access across service boundaries must occur through well-defined interfaces:

1.  **Event-Driven Updates (FACT)**:
    *   Services publish events (e.g., `OrderRequested`, `InventoryReserved`, `AccountCharged`) to the Message Broker (`OutboxProcessor`).
    *   Other interested services consume these events (`AccountService/Worker.cs`, `InventoryService/Worker.cs`, `OrchestratorService/Worker.cs`) to update their own local data models or trigger business logic.
    *   This promotes eventual consistency and loose coupling.
    *   Example: `OrchestratorService` learns about inventory reservation status by consuming the `InventoryReserved` event published by `InventoryService`.

2.  **API Calls (HYPOTHESIS)**:
    *   While not explicitly detailed in the core flow, services may expose RESTful or gRPC APIs to provide read-only views or perform specific actions on data they own.
    *   Example: `OrchestratorService` might make an API call to `AccountService` to retrieve high-level customer information for display purposes, rather than duplicating account data. *Action*: Ensure API contracts are clear and strictly versioned.

3.  **Claim Check Pattern (FACT)**:
    *   For large message payloads, a service publishes a small event containing a reference (claim check ID) to the Message Broker.
    *   The actual large payload is stored in Object Storage.
    *   `ClaimCheckProcessor` retrieves the full payload using the reference and re-publishes or processes the complete message.
    *   This prevents message broker overload and ensures large data can be processed.

### Data Flow ASCII Diagram (Order Creation & Fulfillment)

```text
+---------------------+    (1) API Call     +---------------------+
| External Client     +---------------------> OrchestratorService |
+---------------------+                     +---------+-----------+
                                                      |
                                                      | (2) Save Order State
                                                      |     Publish 'OrderRequested'
                                                      |     to Outbox
                                                      V
                                            +---------+-----------+   (3) Read Outbox
                                            | OrchestratorService |<------------------+
                                            |      Database       |                   |
                                            | (Order/Workflow State)  (Transactional) |
                                            +----------+----------+                   |
                                                       |                              |
                                            +----------V----------+                   |
                                            | OutboxProcessor     |                   |
                                            | (OrderOutboxWorker) |                   |
                                            +----------+----------+                   |
                                                       |                              |
                                                       | (4) Publish 'OrderRequested' |
                                                       |     to Message Broker        |
                                            +----------V----------+                   |
+---------------------------------------------------------------------------------------------------------------------+
|                                          MESSAGE BROKER (e.g., Kafka, RabbitMQ)                                     |
+---------------------------------------------------------------------------------------------------------------------+
     |                     |                      |
     | (5) Consume         | (5) Consume          |
     | 'OrderRequested'    | 'OrderRequested'     |
     V                     V                      V
+----------+----------+ +----------+----------+ +----------+----------+
| InventoryService    | | AccountService    | | OrchestratorService |
| (Worker)            | | (Worker)          | | (Worker)          |
+----------+----------+ +----------+----------+ +----------+----------+
     |                     |                      |
     | (5a) Check/Reserve  | (5b) Prepare Charge  | (11) Update Order Status
     |      Inventory      |      Account         |
     |     Publish 'InventoryReserved'        | Publish 'AccountCharged' |
     |     to Outbox       | to Outbox          |
     V                     V                      V
+----------+----------+ +----------+----------+ +----------+----------+
| InventoryService    | | AccountService    | | OrchestratorService |
|      Database       | |      Database       | |      Database       |
| (Inventory Data)    | | (Account Data)      | | (Order/Workflow State) |
+---------------------+ +---------------------+ +---------------------+

Legend:
----->: API/Direct interaction
----->: Event/Message flow
+-----+ : Service/Component
[-----] : Database/Storage
```
*Note: The diagram illustrates the primary event flow for order processing, showing how services consume events to update their owned data. `OutboxProcessor` mediates between service databases and the Message Broker.*

### Data Ownership Risks

1.  **Data Consistency Challenges (High)**:
    *   **Risk**: Eventual consistency is inherent in event-driven systems. If events are delayed, lost, or processed out of order, data in consuming services might temporarily become inconsistent with the source of truth.
    *   **Action**:
        *   Implement idempotent consumers (`Worker.cs` logic).
        *   Utilize message ordering guarantees (if available and necessary) from the Message Broker.
        *   Design reconciliation mechanisms or sagas (`OrchestratorService`) to detect and resolve inconsistencies.
        *   Consider adding monitoring for message lag and processing errors.

2.  **Schema Evolution (Medium)**:
    *   **Risk**: Changes to the schema of data owned by one service (e.g., `AccountService`'s account data) can break consuming services if events or API contracts are not evolved carefully.
    *   **Action**:
        *   Implement robust versioning strategies for events and APIs (e.g., adding new fields as optional, not removing existing ones without a deprecation period).
        *   Use schema registries for event contracts if the Message Broker supports it.
        *   Establish clear communication protocols between teams for schema changes.

3.  **Data Duplication and Denormalization (Medium)**:
    *   **Risk**: Services may store copies of data owned by other services to reduce cross-service calls and improve performance (e.g., `OrchestratorService` might cache customer IDs). If not managed, this can lead to stale data and increased complexity.
    *   **Action**:
        *   Clearly document denormalized data and its source.
        *   Implement event-driven mechanisms to update denormalized data promptly (e.g., `AccountService` publishes `AccountUpdated` events).
        *   Define acceptable staleness levels for duplicated data.

4.  **Data Lifecycle Management (Medium)**:
    *   **Risk**: Managing data retention, archival, and deletion consistently across services, especially with event streams, can be complex.
    *   **Action**:
        *   Define clear data retention policies for each data entity and service.
        *   Implement mechanisms for data archival or deletion within each owning service.
        *   Consider the implications for event replay and historical data when purging data.

5.  **Access Control and Security (High)**:
    *   **Risk**: Without strict access controls, services or external entities might gain unauthorized access to data they don't own.
    *   **Action**:
        *   Implement database-level access controls, ensuring only the owning service can directly access its database.
        *   Apply API authentication and authorization mechanisms (`AccountService API`, `InventoryService API`).
        *   Secure the Message Broker and Object Storage with appropriate credentials and network segmentation.