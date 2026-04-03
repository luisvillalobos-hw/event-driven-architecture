# Backend Developer Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: APIs, data access, integrations, async patterns

---
## Backend Developer Playbook: event-driven-architecture

This playbook guides an AI assistant operating as a Backend Developer within the `event-driven-architecture` project. Your core focus will be on API development, data interaction, reliable asynchronous communication, and integrating services.

---

### 1. Quick Start

Your primary responsibility involves understanding and extending service logic.

*   **API Service Entry Points**:
    *   Identify API endpoints in `*/Program.cs` files for services like `AccountService`, `InventoryService`, `OrchestratorService`.
    *   Look for `MapControllers()`, `AddEndpointsApiExplorer()`, or explicit endpoint definitions within these files.
*   **Event Consumer Logic**:
    *   Locate background event processing logic in `*/Worker.cs` files for each service. These `IHostedService` implementations are where events are consumed and processed.
    *   Specifically examine `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor/Worker.cs`, and `MessageReplicator/Worker.cs` for critical message infrastructure.
*   **Shared Contracts**:
    *   Review `Shared/` and `OrderMaker/Models/` for common DTOs, interfaces, and event definitions.
*   **Testing Tool**:
    *   Use `TestPublisher/Program.cs` to understand how events are published manually for development and testing.

**To Run a Service (e.g., AccountService):**
Navigate to the service directory (`AccountService/`) and execute: `dotnet run`

---

### 2. Role-Specific Context

This project is deeply event-driven. Direct synchronous communication between services is rare; most inter-service interactions happen via message brokers.

*   **Event as Primary Communication**:
    *   Assume services communicate by publishing and subscribing to events via a message broker. Direct API calls between your microservices are an anti-pattern unless explicitly justified (e.g., lookup of static reference data).
    *   Events are the contract. Changes to events in `Shared/` or `OrderMaker/Models/` must be treated as API contract changes.
*   **Reliable Event Publishing (Outbox Pattern)**:
    *   All domain events published from an API service or worker **must** go through the outbox pattern. The `OutboxProcessor` ensures atomicity between local database transactions and event publication. Do not bypass this.
    *   Reference `OutboxProcessor/OrderOutboxWorker.cs` and related models in `OutboxProcessor/Models/` to understand its operation.
*   **Large Message Handling (Claim Check Pattern)**:
    *   When an event payload exceeds message broker size limits, the claim check pattern is used. The actual data is stored in object storage (e.g., Azure Blob Storage, S3), and the event message contains only a reference (claim check).
    *   The `ClaimCheckProcessor` is responsible for retrieving the original payload and re-publishing/processing the full event. If your service produces large messages, consider integrating with this pattern.
*   **API Responsibility**:
    *   API services (`AccountService`, `InventoryService`, `OrchestratorService`) are responsible for translating external requests (HTTP) into domain commands and events, then publishing these via the outbox. They may also query local read models or current state.
*   **Worker Responsibility**:
    *   `Worker.cs` files in API services, and standalone workers like `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`, consume events, perform business logic, update local state, and potentially publish new events.

---

### 3. Key Areas

| Area                       | Relevant Files/Directories                                                                                                    | Patterns/Conventions                                                                                                                                                                                                                                                                          |
| :------------------------- | :---------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **API Design**             | `*/Program.cs` (e.g., `AccountService/Program.cs`), `Controllers/` directories (inferred)                                     | Uses ASP.NET Core Minimal APIs or standard Controllers. RESTful principles, clear resource naming. Focus on accepting commands and queries, not direct service-to-service calls.                                                                                                                   |
| **Data Access**            | `OutboxProcessor/Models/`, service-specific `Models/` (inferred), `*DbContext.cs` (inferred)                                   | Likely uses Entity Framework Core for relational database access. Data consistency with event publishing via the outbox. Focus on bounded contexts; services own their data.                                                                                                                       |
| **Service Communication**  | `*/Worker.cs` (all services), `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/`, `Shared/`, `OrderMaker/Models/` | **Message Broker centric.** Publish/Subscribe model. All state-changing events should use the transactional outbox (`OutboxProcessor`). Claim check for large messages (`ClaimCheckProcessor`). Shared event contracts in `Shared/` and `OrderMaker/Models/`.                                     |
| **Async Processing**       | `*/Worker.cs` (all services), `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/`                               | `IHostedService` implementations for continuous background processing (event consumption, outbox processing, message replication). Asynchronous operations (`async/await`) are standard within event handlers.                                                                                  |
| **Error Handling/Logging** | `Program.cs` (for host config), `Worker.cs` (for consumer-specific handling)                                                 | Centralized logging configuration (e.g., Serilog, .NET's built-in logging). Structured logging is preferred. Implement robust `try-catch` blocks in event consumers to handle transient errors and prevent message reprocessing loops or dead-letter queue saturation. Circuit breaker patterns. |
| **Shared Contracts**       | `Shared/`, `OrderMaker/Models/`                                                                                              | Defines common DTOs, event messages, enums, interfaces. **Changes here impact all consumers/publishers.** Versioning is critical for these contracts.                                                                                                                                         |

---

### 4. Safe First Changes

These tasks are good for familiarization with minimal risk to core functionality:

1.  **Add a new read-only API endpoint**: To an existing service (e.g., `AccountService/`) that returns a hardcoded value or static data. This helps understand API setup without data interaction.
2.  **Add a new field to an existing DTO**: In `Shared/` or `OrderMaker/Models/` (e.g., add `DisplayName` to an `OrderCreatedEvent`) that is *not yet consumed* by any service. Ensure it's marked as optional if added to an existing event.
3.  **Improve logging detail**: In a non-critical event handler within any `Worker.cs` file. Add more `_logger.LogInformation()` statements without changing logic.
4.  **Create a new test event**: In `TestPublisher/Program.cs` and observe its flow through the system (if any service is configured to consume it).

---

### 5. Danger Zones

Proceed with extreme caution in these areas, as mistakes can have cascading, system-wide impacts:

*   **Modifying `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`**: These are critical infrastructure components. Any misconfiguration or bug can halt event flow, lead to data loss, or inconsistent state.
*   **Bypassing the Outbox Pattern**: Directly publishing events to the message broker from an API service without using the outbox will lead to atomicity violations and potential data inconsistencies if the database transaction fails but the message is sent.
*   **Changing Shared Contracts (`Shared/`, `OrderMaker/Models/`)**: Modifying existing event schemas (e.g., renaming a field, changing a type) without a proper versioning strategy and graceful degradation for older consumers will break integrations.
*   **Introducing Synchronous Inter-Service API Calls**: Resist the urge to call another service's API directly from your service logic. This violates the event-driven philosophy and introduces tight coupling, latency, and single points of failure.
*   **Uncontrolled Database Schema Changes**: Modifying a database schema without considering migrations, data integrity, and the service's bounded context.
*   **Unhandled Exceptions in `Worker.cs`**: An unhandled exception in an event consumer can lead to message reprocessing loops, dead-letter queue saturation, or missed events. Implement robust error handling.

---

### 6. Checklists

#### 6.1. New API Endpoint Development

*   [ ] **Endpoint Design**: Does it adhere to RESTful principles?
*   [ ] **Input Validation**: Are all incoming requests validated?
*   [ ] **Command/Query Mapping**: Is the API request correctly translated into a domain command or query?
*   [ ] **Outbox Usage (for commands)**: If the API initiates a state-changing operation, is an event published via the outbox pattern?
*   [ ] **Logging**: Are requests, key actions, and errors logged appropriately?
*   [ ] **Security**: Is the endpoint appropriately authorized and authenticated (if applicable)?
*   [ ] **Error Responses**: Does it return meaningful HTTP status codes and error messages?

#### 6.2. New Event Handler Implementation (`Worker.cs`)

*   [ ] **Event Contract**: Are you using the correct, current version of the event schema from `Shared/` or `OrderMaker/Models/`?
*   [ ] **Idempotency**: Is the event handler idempotent? Can it safely process the same message multiple times without side effects?
*   [ ] **Error Handling**: Are transient errors handled with retries? Are critical errors logged and moved to a dead-letter queue (if configured)?
*   [ ] **Transactionality**: If the handler involves database writes and publishing new events, is the outbox pattern used for the new events?
*   [ ] **Logging**: Is the consumption of the message and key processing steps logged?
*   [ ] **Performance**: Is the handler efficient? Are there any blocking operations or N+1 query issues?
*   [ ] **Claim Check (if applicable)**: If processing a large message, does it correctly interact with the claim check pattern (e.g., retrieving the full payload)?

#### 6.3. Data Model Change

*   [ ] **Bounded Context**: Does this change belong to this service's bounded context?
*   [ ] **Schema Evolution**: How will existing data be migrated? Is this a non-breaking change (e.g., adding a nullable column) or breaking (e.g., renaming a column)?
*   [ ] **Shared Contracts Impact**: If this change impacts a DTO in `Shared/` or `OrderMaker/Models/`, have all consumers been identified and addressed (via versioning or coordinated deployment)?
*   [ ] **Outbox/Worker Impact**: Do any existing workers or outbox mechanisms need to be updated to handle the new data structure?
*   [ ] **Database Migrations**: Are appropriate EF Core migrations (or similar) created and tested?