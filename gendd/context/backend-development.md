# Backend Development Context Document

## Relevance to This Codebase
Backend development is the core of this event-driven microservices system. It encompasses all service logic, API endpoints, and event processing mechanisms that form the backbone of the application. The system's architecture, heavily reliant on inter-service communication via events and dedicated worker processes, means that the backend defines how business processes are orchestrated, data is managed, and external interactions occur. Understanding the backend's structure, patterns, and conventions is crucial for any modification, debugging, or extension of the system's functionality.

## Current State Assessment

### Project Structure and Services
The codebase is organized as a C# .NET solution, with distinct projects representing individual microservices and shared libraries.
- **API Services**: `AccountService/`, `InventoryService/`, `OrchestratorService/`
  - Each contains a `Program.cs` for application startup (inferred ASP.NET Core web host configuration) and a `Worker.cs` (inferred for background tasks or event consumption within the API service itself, potentially for health checks or small internal processes).
  - API controllers are likely located in a `Controllers/` directory within these service projects (inferred by standard ASP.NET Core conventions).
- **Worker Services**: `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/`
  - These are dedicated background services primarily containing `Worker.cs` files (e.g., `OutboxProcessor/OrderOutboxWorker.cs`). They are responsible for event processing, message replication, and large message payload handling.
- **Shared Libraries**: `Shared/`, `OrderMaker/`
  - `Shared/` contains common DTOs, interfaces, and utilities.
  - `OrderMaker/` specifically provides models and potentially logic related to order creation, likely consumed by `OrchestratorService` and `TestPublisher`.
- **Utility Application**: `TestPublisher/`
  - A console application (`TestPublisher/Program.cs`) for manual event publishing, used for development and testing.

### API Design and Patterns
(Inferred) API services (`AccountService`, `InventoryService`, `OrchestratorService`) likely expose RESTful HTTP APIs.
- Endpoint definition is expected to follow ASP.NET Core Controller patterns.
- Request/response formats are likely JSON, with DTOs defined in `Models/` directories or shared libraries (`Shared/`, `OrderMaker/`).
- Specifics regarding API versioning, input validation, and error response formats are not explicitly detailed in the provided signals but are critical for consistency.

### Service Layer Architecture
Business logic is distributed across services.
- Each service (`AccountService`, `InventoryService`, `OrchestratorService`) encapsulates its own domain logic.
- Services likely utilize Dependency Injection (inferred from `Program.cs` for .NET Core).
- Communication between services is predominantly event-driven, reducing direct coupling.

### Data Access Patterns
- Relational database usage is inferred from the `OutboxProcessor` (monitoring 'outbox tables').
- It is highly probable that Entity Framework Core (EF Core) is used for ORM, given the C#/.NET stack (inferred).
- A repository pattern may be implemented on top of EF Core, separating data access logic from business logic.
- Specifics on migration strategy, `DbContext` configurations, and query optimization patterns are not detailed.

### Event-Driven Patterns
- **Outbox Pattern**: Implemented by `OutboxProcessor/OrderOutboxWorker.cs`. This ensures atomic publishing of domain events by storing them in a local "outbox" table before reliably sending them to a message broker.
- **Claim Check Pattern**: Implemented by `ClaimCheckProcessor/`. This handles large message payloads by storing the payload externally (e.g., blob storage) and sending a message containing only a reference (the "claim check"). The `ClaimCheckProcessor` retrieves the full payload using this reference.

### Configuration Management
- Configuration is likely managed via `appsettings.json` files and environment variables, standard for .NET Core applications.
- Specifics on secret management are not detailed.

## What's Missing or Weak
- **Explicit Data Access Configuration**: While EF Core is inferred, explicit details on `DbContext` setup, migration management (e.g., `Add-Migration`), repository implementations, and unit-of-work patterns are not detailed. (Impact: Medium, Priority: Medium)
- **Comprehensive API Design Guidelines**: Specific patterns for API versioning, consistent input validation using attributes/middlewares, and standardized error response formats are not provided. (Impact: Medium, Priority: Medium)
- **Authentication and Authorization**: No details on how API endpoints are secured, what authentication scheme is used (e.g., JWT), or authorization mechanisms (e.g., roles/claims). (Impact: High, Priority: High)
- **Global Error Handling**: No explicit mention of global exception handling middleware or custom error types. (Impact: Medium, Priority: Medium)
- **Testing Strategy**: Lack of explicit documentation on unit, integration, and API testing patterns, frameworks used, or test data management. (Impact: Medium, Priority: Medium)
- **External Integration Resilience**: While `ClaimCheckProcessor` handles large messages, explicit details on HTTP client configurations (retries, timeouts, circuit breakers) for other external calls are not provided. (Impact: Low, Priority: Low)
- **Observability**: Details on logging patterns (structured logging, correlation IDs), metrics, and tracing are not explicitly provided. (Impact: Medium, Priority: Medium)

## Key Patterns and Conventions
| Pattern | Where Used | Notes |
|---|---|---|
| **Microservices Architecture** | `AccountService/`, `InventoryService/`, `OrchestratorService/` | Each service is a distinct, deployable unit with its own responsibilities. |
| **Event-Driven Communication** | System-wide, especially `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/` | Services communicate primarily through publishing and consuming events. |
| **Outbox Pattern** | `OutboxProcessor/` | Ensures reliable, transactional publishing of domain events from a service's local database. |
| **Claim Check Pattern** | `ClaimCheckProcessor/` | Manages large message payloads by externalizing storage and passing references. |
| **Worker Services** | `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor/Worker.cs` | Dedicated `IHostedService` implementations for background tasks and event processing. |
| **ASP.NET Core APIs** | `AccountService/Program.cs`, `InventoryService/Program.cs`, `OrchestratorService/Program.cs` | Standard framework for building RESTful web services in C#. |
| **Dependency Injection** | `Program.cs` files across all services | Standard .NET Core mechanism for managing dependencies and service lifetimes. |
| **Shared Libraries for Contracts** | `Shared/`, `OrderMaker/` | Centralized definitions for DTOs, events, and common interfaces, promoting consistency. |

## Risks and Hotspots
| Risk | Location | Severity | Mitigation |
|---|---|---|---|
| **Lack of explicit CI/CD and IaC** | Overall System | Medium | Implement automated build, test, and deployment pipelines; define infrastructure using tools like Azure Bicep or Terraform. |
| **"TestPublisher" in main repository** | `TestPublisher/Program.cs` | Low | Ensure `TestPublisher` is excluded from production builds/deployments; consider moving to a separate `test-utils` repository if it's meant for shared testing. |
| **Worker Service Criticality (Outbox, ClaimCheck, MessageReplicator)** | `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/` | High | Implement robust monitoring, alerting, and error handling; ensure worker scalability and resilience (e.g., dead-letter queues, retry policies). |
| **External Dependency Failures (Message Broker, Databases)** | System-wide | High | Implement circuit breakers, retries with backoff, and timeouts for external calls; define clear disaster recovery plans for these components. |
| **Inconsistent API Input Validation** | `AccountService/`, `InventoryService/`, `OrchestratorService/` (Controllers) | Medium | Standardize validation (e.g., using data annotations or FluentValidation); implement global validation middleware. |
| **Missing Authentication/Authorization** | `AccountService/`, `InventoryService/`, `OrchestratorService/` (Controllers) | High | Implement a robust authentication and authorization layer, ensuring all sensitive endpoints are protected and access is correctly scoped. |

## AI Agent Guidelines

### Do
- **Explore `Program.cs`**: Always check `Program.cs` files in service projects to understand startup configuration, dependency injection setup, and middleware pipeline.
- **Review `Worker.cs` for Background Logic**: For any background processing or event consumption logic, refer to `Worker.cs` files (e.g., `OutboxProcessor/OrderOutboxWorker.cs`, `AccountService/Worker.cs`).
- **Consult `Shared/` and `OrderMaker/` for Contracts**: Before defining new DTOs or interfaces, always check these shared libraries for existing types that can be reused.
- **Inspect `Models/` directories**: Look for entity definitions, request/response DTOs, and event structures within service-specific `Models/` directories (e.g., `ClaimCheckProcessor/Models/`).
- **Verify Configuration**: Always check `appsettings.json` and environment variables for service-specific settings and external service connection strings.

### Don't
- **Assume API Security**: Do not assume API endpoints are secured without verifying explicit authentication/authorization attributes or middleware configuration.
- **Modify Shared Libraries Casually**: Avoid making changes to `Shared/` or `OrderMaker/` without understanding the potential impact on all consuming services.
- **Bypass Event-Driven Flow**: Do not introduce direct service-to-service HTTP calls if an event-driven mechanism is already in place or more appropriate for the use case.

### Always Check
- **Startup Configuration**: `Program.cs` files for host configuration, service registrations, and middleware.
- **API Endpoint Definitions**: `Controllers/` directories (inferred) within API services for routing and request handling.
- **Event Consumption Logic**: `Worker.cs` files for how services react to and process events.
- **Data Schemas/DTOs**: `Models/` directories and `Shared/` for data structures used in APIs and events.
- **Outbox/ClaimCheck Implementation**: When dealing with event publishing or large messages, refer to `OutboxProcessor/` and `ClaimCheckProcessor/` respectively.

## Related Context Areas
- **[architecture](./architecture.md)**: For overall system design, service boundaries, and communication patterns.
- **[database-management](./database-management.md)**: For specific database schemas, migration strategies, and query optimization details (relevant for inferred EF Core usage).
- **[security](./security.md)**: For detailed authentication, authorization, and secret management implementations.
- **[quality-assurance](./quality-assurance.md)**: For testing strategies, coverage, and quality gates related to backend services.
- **[fullstack-development](./fullstack-development.md)**: For understanding the API contract between frontend and backend services.

## Key Files
| Path or Glob | Purpose | Notes |
|---|---|---|
| `*/Program.cs` | Entry point for API services and console applications. | Configures application host, DI, and middleware. |
| `*/Worker.cs` | Background task/event consumer entry point. | Contains `IHostedService` implementations for background processing. |
| `*/Controllers/*.cs` | Defines API endpoints and handles HTTP requests. | (Inferred) Standard ASP.NET Core MVC controllers. |
| `*/Models/*.cs` | Data Transfer Objects (DTOs) and domain models. | Service-specific data structures. |
| `Shared/**/*.cs` | Shared DTOs, interfaces, and utilities. | Common types consumed by multiple services. |
| `OrderMaker/**/*.cs` | Order-specific models and logic