## Relevance to This Codebase

Backend development is the core of the `event-driven-architecture` project. It encompasses the implementation of all business logic, API endpoints, and event processing mechanisms that drive the system. Given its microservices nature, each service (`AccountService`, `InventoryService`, `OrchestratorService`, etc.) represents a distinct backend component. Understanding backend patterns, communication protocols, and data flow is critical for maintaining service autonomy, ensuring data consistency across the event-driven landscape, and scaling individual components effectively. Changes to any backend service, especially regarding event contracts or API definitions, can have significant ripple effects across the entire system.

## Current State Assessment

| Artifact | Location/Pattern | Maturity Level | Notes |
| :------- | :--------------- | :------------- | :---- |
| **Language & Framework** | C#, .NET 6/7 (inferred), ASP.NET Core (web services), `IHostedService` (workers) | HIGH | All services are implemented in C# using the .NET ecosystem. Web-facing services use ASP.NET Core, while background tasks leverage the `IHostedService` pattern for long-running workers. |
| **Project Structure** | Each service (`AccountService`, `InventoryService`, `OrchestratorService`, `OutboxProcessor`, etc.) resides in its own top-level directory, containing a `.csproj` file. Common code is in `Shared/`. | HIGH | Clear separation of concerns per microservice. `Program.cs` is the entry point for API services, `Worker.cs` for background workers. |
| **API Design** | RESTful principles are inferred for `AccountService`, `InventoryService`, `OrchestratorService`, `OrderMaker`. Endpoints are likely defined using ASP.NET Core controllers. | MEDIUM (inferred) | Standard HTTP methods (GET, POST, PUT, DELETE) are likely used. JSON is the primary data exchange format. No explicit versioning strategy or API specification (e.g., OpenAPI) found. |
| **Service Layer** | Each service directory likely contains internal layers (e.g., `Controllers/`, `Services/`, `Repositories/`) for business logic and data access. | MEDIUM (inferred) | Based on common .NET patterns, a layered architecture within each service is expected. Responsibilities are separated, with business logic residing in service classes. Dependency Injection is heavily utilized (inferred). |
| **Data Access** | Entity Framework Core (EF Core) is highly likely for relational database interaction, given C# and .NET. Repositories (e.g., `*Repository.cs`) are inferred as the primary data access pattern. | MEDIUM (inferred) | Outbox pattern (`OutboxProcessor/`) implies direct database interaction for transactional outbox table management. EF Core Migrations are expected for schema evolution (inferred). |
| **Event Processing** | `Worker.cs` files in services like `AccountService.Worker`, `InventoryService.Worker`, `OrchestratorService.Worker` indicate event consumption and processing. `OutboxProcessor/` explicitly handles reliable event publishing. | HIGH | Core to the event-driven architecture, demonstrating both publishing (via Outbox) and consuming events. |
| **Error Handling** | Standard ASP.NET Core exception handling middleware is expected for web services. Individual services likely implement localized try/catch blocks for specific error scenarios. | LOW (inferred) | No global custom error response format or specific error handling strategy detected beyond framework defaults. Sensitive information leakage in error responses is a potential concern if not explicitly handled. |
| **Authentication/Authorization** | No explicit implementation details found. For an internal microservice system, internal API keys or token-based authentication (e.g., JWT) are common. | LOW (inferred) | While services might be secured via network policies, individual API endpoints within the services would typically have their own auth/authz mechanisms. |
| **External Integrations** | Message broker (e.g., RabbitMQ, Kafka) integration via `OutboxProcessor/` and various `Worker.cs` consumers. | HIGH | The system relies heavily on message brokers for inter-service communication. Specific client libraries for the broker are used (inferred). No explicit retry/circuit breaker patterns identified, but these are critical. |
| **Containerization** | `Dockerfile` and `.dockerignore` files present in most service directories. | HIGH | Each service is packaged as a Docker image for deployment, indicating a containerized deployment strategy. |

## What's Missing or Weak

| Gap/Weakness | Impact | Priority |
| :----------- | :----- | :------- |
| **Explicit API Specifications/Versioning** | Lack of OpenAPI/Swagger definitions can lead to inconsistent API contracts and hinder consumer development. No clear API versioning strategy can lead to breaking changes for API consumers. | HIGH |
| **Input Validation Enforcement** | Without clear, consistent input validation (e.g., using data annotations or validation libraries) across API endpoints, services are vulnerable to invalid data, security risks, and business logic errors. | HIGH |
| **Centralized Observability (Logging, Tracing, Metrics)** | Absence of structured logging, distributed tracing, and metrics collection makes debugging, performance monitoring, and root cause analysis in a microservices environment extremely challenging. | HIGH |
| **CI/CD Pipelines & Infrastructure as Code (IaC)** | (As per Project Context) Lack of automated build, test, and deployment processes, coupled with manual infrastructure setup, leads to slow, inconsistent, and error-prone deployments, and reduces reliability. | HIGH |
| **Robust Error Handling & Fault Tolerance** | Inconsistent or basic error handling can expose internal details to clients, make debugging difficult, and lead to cascading failures without retry policies, circuit breakers, or comprehensive dead-letter queue handling for events. | MEDIUM |
| **Defined Data Access Strategy & Optimizations** | While EF Core is inferred, specific patterns for query optimization, transaction management (Unit of Work), and handling N+1 issues are not explicit, potentially leading to performance bottlenecks. | MEDIUM |
| **Integration Resilience Patterns** | No explicit mention of retry policies, circuit breakers, or timeouts for external service calls (e.g., message broker interactions, HTTP calls), which can lead to system instability under transient failures. | MEDIUM |
| **Automated Testing Strategy** | No explicit mention of unit, integration, or contract testing frameworks or coverage targets. This leads to reduced confidence in changes and higher likelihood of regressions. | MEDIUM |

## Key Patterns and Conventions

| Pattern Name | Where Used | Notes |
| :----------- | :--------- | :---- |
| **Microservice Architecture** | All top-level service directories (e.g., `AccountService/`, `InventoryService/`, `OrchestratorService/`) | Each service is a self-contained, deployable unit, promoting loose coupling and independent scaling. |
| **Event-Driven Architecture** | Evident through `OutboxProcessor/`, `*Service.Worker/` components, and implied inter-service communication. | Services communicate primarily by publishing and consuming events. The Outbox pattern ensures atomic database updates and event publishing. |
| **Worker/Background Services (`IHostedService`)** | `*Service.Worker/` projects (e.g., `AccountService/Worker.cs`), `OutboxProcessor/OrderOutboxWorker.cs` | Utilizes the .NET `IHostedService` interface for long-running background tasks, typically consuming messages from a broker or performing periodic jobs. |
| **Shared Library for Contracts** | `Shared/` directory | Contains common data transfer objects (DTOs), event contracts, and utility functions used across multiple services, promoting consistency and reducing duplication. |
| **Standard .NET Project Structure** | All service projects (`.csproj`, `Program.cs`, `Properties/`) | Adheres to typical .NET conventions for project organization, configuration, and entry points. |
| **Dependency Injection (DI)** | Inferred in all ASP.NET Core and `IHostedService` based applications. | Services and their dependencies are registered in the `Program.cs` (or `Startup.cs` if older .NET), enabling loose coupling and testability. |
| **Repository Pattern** | Inferred within individual service data access layers (e.g., `*Repository.cs`) | Abstraction over data storage, providing a clean API for business logic to interact with persistence concerns. |
| **Claim Check Pattern** | `ClaimCheckProcessor/` (inferred from name) | Likely used for handling large event payloads by storing them separately and sending only a reference (claim check) through the message broker, reducing message size and broker load. |

## Risks and Hotspots

| Risk | Location | Severity | Mitigation |
| :--- | :------- | :------- | :--------- |
| **`Shared/` Library Breaking Changes** | `Shared/` directory | HIGH | Changes to `Shared/` can break all dependent services. Requires careful review, semantic versioning of the library, and coordinated deployments across the entire system. |
| **Outbox Processor Failure/Bottleneck** | `OutboxProcessor/` | HIGH | The `OutboxProcessor` is critical for reliable event publishing. Failure here leads to data inconsistencies and system-wide stalling. | Implement robust monitoring, alerting, and auto-scaling. Ensure idempotent event handlers for downstream consumers. |
| **Lack of CI/CD & IaC** | Overall System | HIGH | Manual deployments are prone to errors and inconsistencies, impacting reliability and speed. Recovery from failures is slower. | Invest in establishing CI/CD pipelines (e.g., Azure DevOps, GitHub Actions) and define infrastructure using IaC (e.g., Terraform, ARM templates). |
| **Inter-Service Event Contract Mismatches** | Event definitions in `Shared/`, event consumers in `*Service.Worker/` | HIGH | Breaking changes to event schemas can silently break downstream consumers, leading to data corruption or runtime errors in an event-driven system. | Implement robust schema evolution strategies (e.g., forward/backward compatibility), schema registries, and consumer contract testing. |
| **Missing Input Validation on API Endpoints** | API services (e.g., `AccountService`, `InventoryService`) | MEDIUM | Vulnerability to invalid or malicious input, leading to data integrity issues, security flaws (e.g., injection attacks), or application crashes. | Enforce comprehensive input validation using ASP.NET Core's model validation, custom attributes, or dedicated validation libraries (e.g., FluentValidation). |
| **N+1 Query Problems** | Data access layers within individual services (inferred, potentially `*Repository.cs`) | MEDIUM | Inefficient data loading where multiple queries are executed in a loop, leading to performance bottlenecks and increased database load. | Use eager loading (`.Include()`, `.ThenInclude()` in EF Core), projection, or explicit joins to fetch related data in a single query. |
| **Hardcoded Configuration Values** | Potentially `Program.cs`, `Worker.cs`, `appsettings.json` within services | MEDIUM | Sensitive information (e.g., connection strings, API keys) or environment-specific values hardcoded in source control poses security risks and hinders deployment flexibility. | Utilize .NET's configuration system (`IConfiguration`), environment variables, and secrets management (e.g., Azure Key Vault, HashiCorp Vault). |

## AI Agent Guidelines

### DO
*   Always verify the current business logic in the service-specific code files (e.g., `AccountService/Services/AccountManager.cs`) before making changes.
*   Always check the `Shared/` directory for existing data contracts and common utility methods before defining new ones.
*   Always consider the impact on downstream event consumers when proposing changes to event schemas defined in `Shared/`.
*   Always look for `Program.cs` in the root of API service directories to understand service startup, dependency injection, and HTTP pipeline configuration.
*   Always look for `Worker.cs` in the root of worker service directories to understand the main logic for background tasks and event consumption.
*   Always refer to a service's `.csproj` file to identify its dependencies and target framework.

### DON'T
*   Do NOT make direct modifications to the `Shared/` library without coordinating with all services that depend on it.
*   Do NOT introduce new database interaction logic outside of the established data access patterns (e.g., repositories) within a service.
*   Do NOT hardcode connection strings, API keys, or environment-specific values directly into code files.
*   Do NOT bypass existing input validation mechanisms on API endpoints.
*   Do NOT assume an API endpoint is secure; always verify authentication and authorization rules if they are present.

### ALWAYS CHECK
*   **`Shared/`**: Before introducing new types or modifying existing ones, check `Shared/` for reusability and potential impact on other services.
*   **`Program.cs`**: For web services, check `Program.cs` to understand configured middleware, dependency registrations, and hosted services.
