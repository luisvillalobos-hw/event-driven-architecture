# Backend Developer Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: APIs, data access, integrations, async patterns

---
# Backend Developer Playbook: Event-Driven Architecture

This playbook outlines essential knowledge and actions for an AI assistant performing the Backend Developer role within the `event-driven-architecture` project. Your primary focus will be on API development, data persistence, inter-service communication (especially event-driven patterns), and robust async processing.

## 1. Quick Start

To effectively contribute, begin by understanding the core service interactions and deployment model.

*   **Understand Service Roles:**
    *   Review `Program.cs` in API services (`AccountService`, `InventoryService`, `OrchestratorService`, `OrderMaker`) to identify endpoint configurations and service registrations.
    *   Examine `Worker.cs` in background services (`AccountService.Worker`, `InventoryService.Worker`, `OrchestratorService.Worker`, `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`) to understand message consumption and background task logic.
    *   Focus on `OutboxProcessor/OrderOutboxWorker.cs` for critical event publishing mechanics.
*   **Identify Message Contracts:**
    *   Explore the `Shared/` library. This is where all shared DTOs, events, and common interfaces reside. Understanding these contracts is paramount for inter-service communication.
*   **Local Development Setup:**
    *   **Build/Run individual services (via .NET CLI):** Navigate to a service directory (e.g., `AccountService/`) and execute `dotnet run`. This is useful for focused development.
    *   **Build/Run via Docker:** Locate `Dockerfile` in service roots.
        *   `docker build -t <service-name> .`
        *   `docker run -p <host-port>:<container-port> <service-name>`
        *   For multi-service orchestration, look for a `docker-compose.yml` (not explicitly identified, but a common pattern). If not present, be prepared to manage individual containers.
*   **Test Event Publishing:**
    *   Utilize `TestPublisher/Program.cs` to manually send events and observe their processing by worker services. This is invaluable for tracing event flows.

## 2. Role-Specific Context

Your primary responsibility as a Backend Developer here is to build and maintain the engine of the event-driven system.

*   **API Design & Endpoint Conventions:**
    *   APIs are primarily HTTP/REST-based, likely using ASP.NET Core MVC controllers.
    *   Look for common endpoint patterns (e.g., `/api/{resource}`). Consistency in routing, request/response DTOs (often defined in `Shared/`), and HTTP verbs is expected.
    *   Services like `AccountService`, `InventoryService`, `OrchestratorService`, and `OrderMaker` expose these public interfaces.
*   **Data Access Patterns & ORM Usage:**
    *   While not explicitly stated, .NET projects typically use an ORM like Entity Framework Core.
    *   Expect to find `DbContext` implementations, repository patterns, or direct database interactions within service layers.
    *   The outbox pattern (driven by `OutboxProcessor`) implies a dedicated outbox table in the database for reliable event publishing. Look for `SaveChanges` being coupled with event creation within a transaction.
*   **Service-to-Service Communication:**
    *   **Asynchronous & Event-Driven:** This is the project's core. Communication primarily occurs through a message broker via events.
    *   **Outbox Pattern:** The `OutboxProcessor` ensures atomic database transactions and message publishing, making it a critical component for data consistency. Events are first written to a local database outbox and then reliably forwarded to the message broker.
    *   **Event Contracts:** All shared event definitions are in `Shared/`. Any new event or modification must be defined here.
    *   **Consumers:** `*.Worker` services are event consumers, subscribing to relevant topics/queues.
*   **Async Processing & Queue Patterns:**
    *   Background processing is handled by `*.Worker` services, likely implemented using .NET's `IHostedService`.
    *   Message consumption should be designed for idempotency – ensuring that processing an event multiple times yields the same result, as message brokers can deliver messages more than once.
    *   Look for retry mechanisms, error queues (dead-letter queues), and monitoring for worker health.
*   **Error Handling & Logging Standards:**
    *   Expect `ILogger<T>` for structured logging across all services.
    *   API endpoints should return appropriate HTTP status codes with informative error bodies.
    *   Worker services must handle exceptions gracefully to prevent message loss and ensure retries or dead-lettering.
    *   Look for global exception handlers in API services (e.g., middleware).

## 3. Key Areas

| Area                        | Relevant Files/Directories                                                                                                    | Patterns to Observe                                                                                                                                                                                                                                                                                                                                   |
| :-------------------------- | :---------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **API Endpoints**           | `AccountService/Program.cs`, `InventoryService/Program.cs`, `OrchestratorService/Program.cs`, `OrderMaker/Models/` (for DTOs) | ASP.NET Core `AddControllers`, routing attributes (`[Route]`, `[HttpGet]`, etc.), input validation, DTOs for request/response, common HTTP status codes.                                                                                                                                                                                              |
| **Data Access**             | Service-specific code (e.g., `AccountService/` for account data, `InventoryService/` for inventory data)                      | `DbContext` (likely EF Core), repository/unit of work patterns, direct SQL interaction (less likely but possible), transactional boundaries, the outbox table used by `OutboxProcessor`.                                                                                                                                                                |
| **Event Definitions**       | `Shared/`                                                                                                                     | C# classes/records representing events, often with an `Id` and `Timestamp`, potentially versioned. Used by both publishers and consumers to serialize/deserialize messages.                                                                                                                                                                               |
| **Event Publishing**        | `OutboxProcessor/OrderOutboxWorker.cs`, service-specific logic before calling `SaveChanges()`                                 | Transactions encompassing business logic updates and outbox table inserts. `OutboxProcessor` then reads from this table and publishes to the message broker. Look for `IOutboxMessage` or similar interfaces.                                                                                                                                           |
| **Event Consumption**       | `AccountService.Worker/Worker.cs`, `InventoryService.Worker/Worker.cs`, `OrchestratorService.Worker/Worker.cs`              | `IHostedService` implementations, message deserialization, business logic execution, idempotency checks, acknowledgment/negative acknowledgment (ACK/NACK) to the message broker.                                                                                                                                                                    |
| **Claim Check Pattern**     | `ClaimCheckProcessor/`                                                                                                        | Storage and retrieval of large message payloads, potentially using a blob storage or database, with the message broker only carrying a reference (the "claim check").                                                                                                                                                                                    |
| **Message Replication**     | `MessageReplicator/`                                                                                                          | Logic for consuming messages from one source and republishing to another, possibly for data consistency, auditing, or disaster recovery.                                                                                                                                                                                                           |
| **Logging & Monitoring**    | `Program.cs` in all services, service-specific code using `ILogger`                                                           | `AddLogging` configuration, structured logging with contextual information, appropriate log levels (Debug, Info, Warning, Error, Critical), dependency injection of `ILogger`.                                                                                                                                                                         |
| **Cross-Cutting Concerns**  | `Shared/`                                                                                                                     | Common utilities, extension methods, base classes, interfaces, and shared configuration patterns.                                                                                                                                                                                                                                                           |

## 4. Safe First Changes

These tasks are suitable for initial contributions and understanding the codebase with minimal risk.

*   **Enhance Existing Logging:** Add more detailed `ILogger` calls (e.g., `LogInformation`, `LogDebug`) within a specific API endpoint's or worker's business logic, without changing the logic itself.
    *   _Example:_ Add `_logger.LogInformation("Processing order {OrderId}", order.Id);` in `OrchestratorService.Worker/Worker.cs`.
*   **Add a Non-Critical Read-Only API Endpoint:** Create a new GET endpoint in `AccountService/` that fetches existing data without modifying the database or publishing events.
    *   _Example:_ A new endpoint `/api/accounts/{accountId}/details` that returns more account details.
*   **Refactor Internal Utility:** Identify a small, self-contained helper method within a single service (e.g., `InventoryService/`) and refactor it for clarity or performance, ensuring all existing call sites are updated.
*   **Add an Optional Field to an Internal Event:** If a new, non-breaking data point is needed *within* an event that is *only* consumed by services you control, add it to the event definition in `Shared/` and ensure consumers safely ignore it or handle it if present. This is safer than modifying existing critical fields.
    *   _Example:_ Adding `string? OptionalNote` to an `OrderCreatedEvent` in `Shared/` if all existing consumers are robust to new fields.

## 5. Danger Zones

Proceed with extreme caution in these areas. Changes here can have wide-ranging, potentially catastrophic impacts.

*   **`Shared/` Library Modifications:**
    *   **Risk:** Changes to event contracts, DTOs, or critical utilities within `Shared/` will affect ALL dependent services. This can lead to runtime errors, deserialization failures, or logical inconsistencies across the system.
    *   **Mitigation:** Any changes *must* be backward-compatible (e.g., adding optional fields, new events), rigorously tested across all services, and ideally coordinated with a phased deployment strategy. Avoid modifying existing fields or removing types unless absolutely necessary and with a strong migration plan.
*   **`OutboxProcessor/`:**
    *   **Risk:** This service is vital for reliable event publishing. Any bug, performance bottleneck, or misconfiguration will directly impact data consistency, lead to lost events, or halt event flow entirely.
    *   **Mitigation:** Treat this service as mission-critical. Understand its mechanics thoroughly before proposing changes. Implement extensive testing, monitoring, and alerts.
*   **Database Schema Changes:**
    *   **Risk:** Modifying database schemas (especially for tables shared across services, or the outbox table) can cause data loss, application errors, or deployment failures.
    *   **Mitigation:** Always use migrations (if EF Core is used) and plan changes carefully. Coordinate with all services interacting with the modified tables.
*   **Breaking Changes to Existing Event Contracts:**
    *   **Risk:** Modifying an existing field's type, removing a field, or changing the semantic meaning of an event published to the message broker can silently break downstream consumers, leading to data corruption or unexpected behavior.
    *   **Mitigation:** Favor adding new, optional fields. For significant changes, consider versioning events (e.g., `OrderCreatedV2`) or implementing a robust schema evolution strategy. Never assume all consumers will be updated simultaneously.
*   **Non-Idempotent Worker Logic:**
    *   **Risk:** If a worker processes a message and its logic isn't idempotent, reprocessing the same message (due to transient errors or message broker re-delivery) can lead to duplicate operations (e.g., creating the same order twice, debiting an account multiple times).
    *   **Mitigation:** Always design worker logic to be idempotent. Implement checks to prevent re-processing (e.g., checking if an order ID has already been processed) or ensure operations inherently produce the same result if executed multiple times.

## 6. Checklists

### Checklist: Implementing a New API Endpoint

*   [ ] **Define Request/Response DTOs:** Create appropriate DTOs in `Shared/` or within the service, ensuring they only expose necessary data.
*   [ ] **Create Controller/Action:** Implement the new endpoint in the relevant service's controller.
*   [ ] **Routing & HTTP Method:** Use appropriate `[Route]` and HTTP verb attributes (`[HttpGet]`, `[HttpPost]`, etc.).
*   [ ] **Input Validation:** Apply data annotations (`[Required]`, `[Range]`) or use a library like FluentValidation.
*   [ ] **Business Logic:** Implement or delegate to appropriate service/domain logic.
*   [ ] **Error Handling:** Return meaningful HTTP status codes (e.g., 200 OK, 201 Created, 400 Bad Request, 404 Not Found, 500 Internal Server Error) and structured error responses.
*   [ ] **Logging:** Add `ILogger` calls for important events (request received, processing started/finished, errors).
*   [ ] **Authentication/Authorization (if applicable):**