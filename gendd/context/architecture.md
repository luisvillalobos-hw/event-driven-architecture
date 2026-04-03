# Architecture Context Document

## Relevance to This Codebase

This codebase implements an event-driven microservices architecture, making a deep understanding of its architectural design paramount. The system's purpose is to orchestrate complex business workflows using asynchronous event communication. This necessitates careful attention to service boundaries, communication patterns, data consistency, and the resilience of event processing mechanisms. Modifications or extensions to any service or event flow require a clear grasp of how components interact via messages and APIs, and how critical patterns like the transactional outbox and claim check ensure data integrity and handle large message payloads. Understanding the architecture is crucial for maintaining reliability, scalability, and the overall correctness of business processes.

## Current State Assessment

| Artifact / Aspect | Location / Evidence | Maturity / Description |
| :---------------- | :------------------ | :------------------- |
| **Architectural Style** | Project structure (`AccountService/`, `InventoryService/`, etc.) | **Event-Driven Microservices:** Each service (`AccountService`, `InventoryService`, `OrchestratorService`) is a distinct deployable unit, exposing APIs and/or consuming/producing events. Background worker services (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`) handle specific asynchronous tasks, indicative of a distributed system relying heavily on message-passing. |
| **Communication Patterns** | `Program.cs` (for API endpoints), `Worker.cs` (for event consumers), `OutboxProcessor/` (event publishing) | **RESTful APIs** for synchronous interactions (`AccountService`, `InventoryService`, `OrchestratorService`). **Asynchronous Event Messaging** via an inferred message broker for inter-service communication and workflow orchestration. The `OutboxProcessor` ensures reliable event publishing. |
| **Data Architecture** | `OutboxProcessor/` (database outbox table inferred), `ClaimCheckProcessor/` (external storage inferred) | **Database-per-Service (inferred):** Each service likely manages its own data store. **Transactional Outbox:** Implemented via `OutboxProcessor` for atomic event publishing from local service databases. **External Blob/Object Storage (inferred):** Used by `ClaimCheckProcessor` for handling large message payloads, referencing them via a claim check mechanism. |
| **System Context (C4-L1)** | `README.md` (project purpose), overall project structure | The system is a core platform for processing business workflows (e.g., e-commerce order fulfillment) interacting with external users (via APIs) and potentially other internal/external systems (via published events). |
| **Container View (C4-L2)** | Directories: `AccountService/`, `InventoryService/`, `OrchestratorService/`, `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/`, `TestPublisher/`, `OrderMaker/`, `Shared/` | **API Services:** `AccountService`, `InventoryService`, `OrchestratorService` (C# .NET applications exposing REST APIs). **Worker Services:** `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator` (C# .NET background services). **Utility/Library Components:** `TestPublisher` (CLI tool), `OrderMaker` (shared library), `Shared` (common library). **External Containers (inferred):** Message Broker, Relational Databases, Blob Storage. |
| **Architectural Decisions** | Existence of `OutboxProcessor/`, `ClaimCheckProcessor/` | **Event-Driven Paradigm:** Central to the system's design. **Transactional Outbox Pattern:** Chosen for reliable event publishing. **Claim Check Pattern:** Adopted for handling large event payloads. |
| **Key Libraries/Utilities** | `OrderMaker/`, `Shared/` | Provide shared models (`OrderMaker/Models/`, `Shared/Models/`) and common utilities to promote consistency across services. |

## What's Missing or Weak

| Gap / Weakness | Impact | Priority |
| :------------- | :----- | :------- |
| **No explicit CI/CD or IaC** | Potential for inconsistent deployments, manual errors, slow deployments, and challenges in scaling or disaster recovery. | HIGH |
| **No C4 diagrams or ADRs** | Lacks formal documentation of architectural decisions and system overview. This makes onboarding new team members difficult and increases the risk of architectural drift over time. | MEDIUM |
| **Undefined Performance/Scalability Targets** | Without explicit targets, it's difficult to assess if the architecture meets non-functional requirements or to plan for future growth. | LOW |
| **Monitoring and Observability Strategy (inferred missing)** | Lack of explicit setup for distributed tracing, centralized logging, and metrics makes diagnosing issues in an event-driven microservices environment significantly harder. | MEDIUM |
| **External Dependency Configurations** | While a message broker and databases are inferred, their specific configurations, connection details, and management are not explicitly documented or centralized, leading to operational risks. | MEDIUM |

## Key Patterns and Conventions

| Pattern / Convention | Where Used | Notes |
| :------------------- | :--------- | :---- |
| **Event-Driven Architecture** | Across all services, particularly worker services (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`) | Services communicate primarily by producing and consuming events via an inferred message broker, enabling loose coupling and asynchronous workflows. |
| **Transactional Outbox Pattern** | `OutboxProcessor/` | Ensures atomicity between a service's local database transaction and the publication of domain events to the message broker. Events are first written to a local outbox table, then processed and dispatched by `OutboxProcessor`. |
| **Claim Check Pattern** | `ClaimCheckProcessor/` | Used to handle large event payloads. Instead of sending the full payload directly in the message, a reference (claim check) is sent, and the `ClaimCheckProcessor` retrieves the actual data from external blob storage when needed. |
| **C# .NET Solution Structure** | Root directory (`.sln` file), individual service/library directories (e.g., `AccountService/`, `Shared/`) | Standard Visual Studio solution with separate projects for each logical component, facilitating modularity and independent development. |
| **API Service Entry Point** | `*/Program.cs` for `AccountService`, `InventoryService`, `OrchestratorService` | Standard .NET Host builder pattern for configuring and running API services, including Kestrel server setup and dependency injection. |
| **Worker Service Entry Point** | `*/Worker.cs` for `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator` | Standard .NET `BackgroundService` pattern for long-running background tasks, typically consuming messages from a broker or polling a database. |

## Risks and Hotspots

| Risk | Location / Evidence | Severity | Mitigation |
| :--- | :------------------ | :------- | :--------- |
| **Lack of CI/CD & IaC** | Overall System (absence of `azure-pipelines.yml`, `terraform/`, `k8s/` etc.) | HIGH | Implement automated CI/CD pipelines for builds, tests, and deployments. Introduce Infrastructure as Code (IaC) for environment provisioning and configuration. |
| **Critical Worker Service Failure** | `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/` | HIGH | Implement robust monitoring and alerting for these services. Ensure sufficient redundancy (e.g., multiple instances) and auto-scaling. Implement dead-letter queueing and retry mechanisms for message processing. |
| **Message Broker / Database Dependency Failure** | Inferred external dependencies | HIGH | Define clear disaster recovery and backup strategies for these critical components. Implement circuit breakers and retry policies in services interacting with them. Monitor health and performance of these dependencies. |
| **`TestPublisher` in Production** | `TestPublisher/Program.cs` | MEDIUM | Ensure `TestPublisher` is excluded from production builds and deployments. Implement strong authentication and authorization if similar utilities are needed in production, or use a dedicated administration UI. |
| **Eventual Consistency Challenges** | Overall Event-Driven Architecture | MEDIUM | Ensure all services are designed to handle eventual consistency. Implement idempotent consumers. Provide clear mechanisms for tracing and debugging event flows to diagnose discrepancies. |
| **Undocumented External System Integrations** | No explicit `docs/integrations/` | MEDIUM | Document all external systems and APIs interacted with, including their purpose, data contracts, authentication mechanisms, and error handling strategies. |

## AI Agent Guidelines

| Category       | Guidance                                                                                                       |
| :------------- | :------------------------------------------------------------------------------------------------------------- |
| **DO**         | When investigating inter-service communication, **always check** the `OutboxProcessor/` for how events are published and `Worker.cs` files in other services for how they are consumed. |
| **DO**         | When assessing data flow for large messages, **always review** `ClaimCheckProcessor/` logic to understand how external storage is utilized. |
| **DO**         | Before modifying shared models or interfaces, **always inspect** the `Shared/` and `OrderMaker/` directories to understand their usage across services. |
| **DO**         | To understand a service's entry point and configuration, **always refer** to its `Program.cs` file (for APIs) or `Worker.cs` file (for background tasks). |
| **DON'T**      | **Do NOT** assume synchronous communication between services unless explicit REST API calls are visible in the code. Favor understanding event-driven communication. |
| **DON'T**      | **Do NOT** introduce direct database access between services; respect the database-per-service paradigm. |
| **ALWAYS CHECK** | **Always check** the project structure for new services or libraries that might introduce new architectural concerns or dependencies. |
| **ALWAYS CHECK** | **Always check** the `Models/` subdirectories within services and shared libraries for current data structures and contracts. |
| **ALWAYS CHECK** | **Always check** the `README.md` for high-level project goals and any documented architectural principles. |

## Related Context Areas

*   **[backend-development](./backend-development.md)**: For understanding the specific API designs, service logic implementations, and data access patterns within each C# .NET service.
*   **[devops-infrastructure](./devops-infrastructure.md)**: For deployment strategies, containerization details (Docker is implied), and the management of the message broker and databases.
*   **[security](./security.md)**: For assessing the security posture, authentication/authorization boundaries across services, and secure handling of data, especially for `TestPublisher` implications.
*   **[database-management](./database-management.md)**: For details on the specific database technologies used by each service and the outbox pattern's interaction with them.
*   **[site-reliability](./site-reliability.md)**: For understanding monitoring, alerting, resilience patterns, and disaster recovery strategies relevant to the worker services and external dependencies.

## Key Files

| Path / Pattern                 | Purpose                                                                 | Notes                                                               |
| :----------------------------- | :---------------------------------------------------------------------- | :------------------------------------------------------------------ |
| `*.sln`                        | Solution file for the entire project.                                   | Defines the project structure and dependencies.                     |
| `AccountService/`              | Directory for the Account API service.                                  | Contains `Program.cs` (entry point) and business logic.             |
| `InventoryService/`            | Directory for the Inventory API service.                                | Contains `Program.cs` (entry point) and business logic.             |
| `OrchestratorService/`         | Directory for the Orchestrator API service.                             | Contains `Program.cs` (entry point) and workflow orchestration logic. |
| `OutboxProcessor/`             | Directory for the transactional outbox worker service.                  | Critical for reliable event publishing.                             |
| `ClaimCheckProcessor/`         | Directory for the claim check worker service.                           | Handles large message payloads by externalizing data.               |
| `MessageReplicator/`           | Directory for the message replication worker service.                   | Manages event flow between topics/brokers.                          |
| `TestPublisher/Program.cs`     | Entry point for a utility to publish test events.                       | Primarily for development/testing, potential risk if deployed.      |
| `OrderMaker/`                  | Directory for a shared library containing order-related models/logic.   | Consumed by other services for common order definitions.            |
| `Shared/`                      | Directory for a general shared library.                                 | Contains common DTOs, interfaces, and utilities.                    |
| `*/Program.cs`                 | Entry point for API services.                                           | Defines service configurations and API routes.                      |
| `*/Worker.cs`                  | Entry point for background worker services.                             | Defines event consumers or background tasks.                        |
| `*/Models/`                    | Directories containing data models/DTOs.                                | Defines data structures used across services or within a service.   |
| `.dockerignore`                | Defines files/patterns to ignore when building Docker images.           | Implies the use of Docker for containerization.