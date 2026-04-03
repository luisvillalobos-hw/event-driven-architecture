# Fullstack Developer Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: End-to-end features, full stack patterns, integration points

---
## Fullstack Developer Playbook

As a Fullstack Developer AI, your primary role in this `event-driven-architecture` project is to implement end-to-end features, bridging the gap between API consumers and the underlying event-driven microservices. You will focus on defining API contracts, ensuring consistent data models across services, orchestrating workflows, and understanding the full lifecycle of a request from its initial API call to its eventual processing by downstream event consumers.

### Quick Start

To rapidly onboard and understand the project:

1.  **Solution Overview**: Open `event-driven-architecture.sln` in your C# IDE to explore the project structure.
2.  **API Service Entry Points**: Review `AccountService/Program.cs`, `InventoryService/Program.cs`, and `OrchestratorService/Program.cs` to understand how API services are configured and started.
3.  **Event Consumers**: Examine `AccountService/Worker.cs`, `InventoryService/Worker.cs`, and `OrchestratorService/Worker.cs` to see how each service consumes and reacts to events.
4.  **Shared Contracts**: Inspect `Shared/` and `OrderMaker/Models/` to grasp the common DTOs and event models that define inter-service communication.
5.  **Local Testing**: Use `TestPublisher/Program.cs` as an example to publish events and interact with the system for development and testing.
    *   **Command**: `dotnet run --project TestPublisher` (modify `Program.cs` for desired event publishing).
6.  **Build & Run a Service**:
    *   **Command**: `dotnet build` (from solution root)
    *   **Command**: `dotnet run --project AccountService` (to run the AccountService API).

### Role-Specific Context

Your focus is on the complete user experience and business flow, from the client's perspective down to the data persistence layer, mediated by events.

*   **API Exposure**: You are responsible for designing and implementing RESTful (or similar) APIs in services like `AccountService`, `InventoryService`, and `OrchestratorService`.
*   **Event-Driven Interaction**: Understand that direct database calls between services are discouraged. Instead, services communicate via events published to and consumed from a message broker.
*   **Reliability Patterns**: Be aware of the `OutboxProcessor` for reliable event publishing (transactional outbox) and the `ClaimCheckProcessor` for handling large message payloads efficiently. Your API changes often trigger these patterns indirectly.
*   **Workflow Orchestration**: `OrchestratorService` is key for complex, multi-step business processes that span several services. You'll likely define API endpoints to kick off these workflows and handle events that advance them.
*   **Shared Models**: Consistency is paramount. Always define shared data structures in `Shared/` or `OrderMaker/Models/` and reuse them across your API definitions and event contracts.

### Key Areas

| Aspect                           | Relevant Files/Directories                                     | Fullstack Relevance                                                                                     |
| :------------------------------- | :------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------ |
| **Frontend-to-Backend API**      | `AccountService/Program.cs`, `InventoryService/Program.cs`, `OrchestratorService/Program.cs` (setup)                                                                           | Define API routes, request/response models, and business logic entry points for client interaction.   |
|                                  | `*.cs` files within API services (e.g., Controllers)           | Implement the actual API endpoints that clients will consume.                                           |
| **API Contracts & Type Sharing** | `Shared/`, `OrderMaker/Models/`                               | Crucial for ensuring consistency between client requests, API responses, and inter-service event models. |
| **Full Request Lifecycle**       | `API Service/Program.cs` (request entry)                       | Initial handling of HTTP requests, deserialization, and validation.                                     |
|                                  | `API Service/BusinessLogic.cs` (implicit)                      | Core business logic, potentially publishing events.                                                     |
|                                  | `OutboxProcessor/OrderOutboxWorker.cs` (indirect)              | Ensures reliable event publishing after database transactions.                                          |
|                                  | `Worker.cs` (in consuming services, e.g., `AccountService/Worker.cs`) | Event consumption and subsequent business logic execution.                                              |
|                                  | `ClaimCheckProcessor/Worker.cs` (for large payloads)           | Handles large event bodies, ensuring they are retrieved and processed correctly.                        |
| **Cross-Cutting Concerns**       | `*/Program.cs` (logging, error handling, auth config)          | Configure global aspects like logging frameworks, exception handling middleware, and authentication/authorization.                                                        |
| **Development & Testing**        | `TestPublisher/Program.cs`                                     | Essential tool for prototyping, integration testing, and debugging event flows.                         |

### Safe First Changes

1.  **Add a new field to an existing DTO**:
    *   **Example**: Add `string CustomerNotes` to `Shared/Models/OrderCreated.cs`.
    *   **Task**: Update the corresponding API endpoint (e.g., in `OrchestratorService`) and any consuming `Worker.cs` that uses this DTO.
2.  **Add a new API endpoint to `TestPublisher`**:
    *   **Example**: Create a new static method in `TestPublisher/Program.cs` to publish a custom event for testing.
    *   **Task**: Modify `TestPublisher/Program.cs` to simulate new event types or API calls.
3.  **Modify a log message**:
    *   **Example**: Change `_logger.LogInformation("Processing order {OrderId}");` to `_logger.LogInformation("Processing order {OrderId} with status {Status}");` in `OrchestratorService/Worker.cs`.
    *   **Task**: Ensure the new log message is clear and adds value without changing logic.
4.  **Create a new, isolated utility class**:
    *   **Example**: Add `Shared/Utilities/DateHelper.cs` with a simple date formatting method.
    *   **Task**: Implement a small, self-contained utility that doesn't impact core logic.

### Danger Zones

*   **Modifying Core Worker Logic**: Changes to `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor/Worker.cs`, or `MessageReplicator/` can severely impact system reliability, data consistency, and message flow. These are critical infrastructure components.
*   **Changing Shared Contracts without Impact Analysis**: Altering models in `Shared/` or `OrderMaker/Models/` without considering all producers and consumers can lead to deserialization errors, broken workflows, and runtime exceptions across multiple services. Ensure backward compatibility or coordinated deployments.
*   **`TestPublisher` in Production**: As noted in the project analysis, ensure `TestPublisher` is never deployed to production environments to prevent accidental data manipulation or security vulnerabilities.
*   **Altering `Program.cs` in Core Services**: Modifications to `AccountService/Program.cs`, `InventoryService/Program.cs`, or `OrchestratorService/Program.cs` can affect critical aspects like dependency injection, middleware pipelines, host configuration, and startup behavior. Review these changes carefully.
*   **Database Schema Changes**: Any changes to the underlying database schemas, especially those managed by the outbox pattern, require careful coordination with event contract updates and migration strategies to avoid data loss or inconsistency.

### Checklists

#### New Feature/API Endpoint Checklist

-   [ ] **API Definition**: Have new API routes and their HTTP methods been defined in the relevant service (`AccountService`, `InventoryService`, `OrchestratorService`)?
-   [ ] **Request/Response Models**: Are request and response DTOs defined in `Shared/` or `OrderMaker/Models/`?
-   [ ] **Business Logic**: Is the core business logic implemented, including validation and state changes?
-   [ ] **Event Publication**: If the feature requires downstream processing, is an event reliably published (implicitly via the service's interaction with the outbox pattern)?
-   [ ] **Event Contract**: Is the event model defined in `Shared/` or `OrderMaker/Models/`?
-   [ ] **Event Consumption**: Is there a corresponding `Worker.cs` in the consuming service(s) to handle the published event?
-   [ ] **End-to-End Test**: Can the full flow be tested using `TestPublisher` or an equivalent integration test?
-   [ ] **Logging & Error Handling**: Are appropriate log messages present, and is error handling implemented for edge cases?

#### Updating Existing Event Contract Checklist

-   [ ] **Backward Compatibility**: Is the change backward-compatible for existing consumers? (e.g., adding an optional field, not removing a required one).
-   [ ] **Version Strategy**: If not backward-compatible, is a versioning strategy implemented (e.g., new event type, schema evolution)?
-   [ ] **Producer Updates**: Have all services that publish this event been updated to reflect the new contract?
-   [ ] **Consumer Updates**: Have all services that consume this event been updated to handle the new contract?
-   [ ] **Test Data**: Have existing test data and integration tests been updated or created for the new contract?

#### Debugging Workflow Checklist

-   [ ] **API Service Logs**: Check `Program.cs` and controller logs for initial request failures or configuration issues.
-   [ ] **Event Publication**: If an event isn't triggering, verify the API service successfully processes the request and *intends* to publish the event (check local logs, outbox table if accessible).
-   [ ] **Outbox Processor**: If events are in the outbox but not being published, check `OutboxProcessor/OrderOutboxWorker.cs` logs for issues with event dispatch.
-   [ ] **Message Broker**: Verify messages are present in the message broker (manual inspection if possible).
-   [ ] **Consuming Worker Logs**: Check `Worker.cs` logs in the target service for errors during event consumption or processing.
-   [ ] **Claim Check Processor**: If large messages are involved, check `ClaimCheckProcessor/Worker.cs` logs for issues retrieving payloads.
-   [ ] **Test Publisher**: Use `TestPublisher` to isolate the event in question and manually send it to replicate the issue.