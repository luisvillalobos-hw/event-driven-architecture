# System Summary

## 1. System Overview & Purpose

This system is designed as an event-driven microservices platform for managing core business operations, primarily focusing on order, account, and inventory management. Its primary purpose is to process and orchestrate complex business workflows reliably and scalably. The architecture emphasizes loose coupling and high availability through asynchronous communication using events and an outbox pattern for transactional consistency.

Key business processes include:
*   **New Order Processing**: Orchestrating the steps from order creation (`OrderMaker`) through account validation (`AccountService`) and inventory reservation (`InventoryService`) to final order fulfillment. This flow heavily relies on the `OrchestratorService` to manage the workflow state.

## 2. Architecture & Communication Patterns

**FACT (Confirmed):** The system employs an event-driven microservices architecture. Each business domain (Account, Inventory, Orchestrator, Order) is encapsulated within its own service.

**HYPOTHESIS (High Confidence):** Communication between services is primarily asynchronous via a message broker. Services publish events (e.g., `OrderCreated`, `InventoryReserved`) and consume events relevant to their domain.

**Communication Pattern:**
*   **Outbox Pattern:** Critical for reliable event publishing. Services write events to an outbox table within their own database transaction. The `OutboxProcessor` then reads these events and publishes them to a message broker, ensuring atomicity between database state changes and event emission.

**Data Approach:**
*   **Decentralized Data Ownership:** Each service (e.g., `AccountService`, `InventoryService`, `OrchestratorService`) owns and manages its dedicated data store, promoting autonomy and reducing inter-service dependencies.

**Architectural Diagram (Simplified Flow):**

```
+--------------------+      +-----------------------+      +------------------+
|  Service A (API)   |----->|  DB (Outbox Table)    |<-----|  OutboxProcessor |
|  (e.g., OrderMaker)|      | (Transactional Write) |      |    (Worker)      |
+--------------------+      +-----------------------+      +------------------+
                                        |                             |
                                        V                             V Publishes
                               +-------------------+
                               |   Message Broker  |
                               +-------------------+
                                   |         |
                          Consumes |         | Publishes
                                   V         V
+--------------------+         +--------------------+         +--------------------+
|  Service B (API)   |<--------|  Service B (Worker)|<--------|  Service C (Worker)|
| (e.g., AccountSvc) |         | (Consumes Events)  |         | (Consumes Events)  |
+--------------------+         +--------------------+         +--------------------+
         |                                |                            |
         V owns                           V owns                       V owns
   +-----------+                    +-----------+                +-----------+
   | Service B |                    | Service C |                | Service D |
   |    DB     |                    |    DB     |                |    DB     |
   +-----------+                    +-----------+                +-----------+
```
*Note: Service B/C/D represent other domain services like AccountService, InventoryService, OrchestratorService. Each API service might also call its own Worker for async tasks.*

## 3. Core Components & Tech Stack

**FACT (Confirmed):**
*   **Programming Language:** C#
*   **Framework:** .NET (specifically, ASP.NET Core for API services and .NET Core for Worker services)
*   **Containerization:** Docker (indicated by `Dockerfile` and `.dockerignore` mentions)

**HYPOTHESIS (High Confidence):**
*   **Message Broker:** Essential for an event-driven architecture, likely Kafka or RabbitMQ, but not explicitly defined. The `OutboxProcessor` design strongly implies its presence.
*   **Database:** Relational databases (e.g., PostgreSQL, SQL Server) are highly probable for each service's data store, and for the outbox tables.

**Key Services:**
*   **AccountService (API & Worker):** Manages user accounts and related data.
*   **InventoryService (API & Worker):** Manages product inventory.
*   **OrchestratorService (API & Worker):** Manages long-running business processes and coordinates interactions between other services.
*   **OrderMaker (API):** Initiates new orders into the system.
*   **OutboxProcessor (Worker):** Ensures reliable event publishing from service outbox tables to the message broker.
*   **Shared (Library):** Centralized library for common code, data contracts, and utilities, reducing duplication.
*   **ClaimCheckProcessor (Worker, HYPOTHESIS):** Potentially handles large message payloads via the claim-check pattern, offloading the broker.
*   **MessageReplicator (Worker, HYPOTHESIS):** Might be used for cross-broker replication, auditing, or disaster recovery.

## 4. Facts, Hypotheses & Uncertainties

**FACTS (Confirmed):**
*   The system is an event-driven microservices architecture.
*   Implemented using C# and .NET.
*   Utilizes the outbox pattern for reliable messaging.
*   Docker is planned for containerization.
*   Services include dedicated APIs (`Program.cs`) and background workers (`Worker.cs`).

**HYPOTHESES (High Confidence):**
*   Message broker (e.g., Kafka, RabbitMQ) is a core component for inter-service communication.
*   Each service uses a dedicated database (likely relational).
*   `Shared/` library contains common DTOs/events and infrastructure code.
*   `TestPublisher` is a dev/testing utility.
*   `ClaimCheckProcessor` and `MessageReplicator` are specialized workers for advanced messaging patterns.

**UNCERTAINTIES (Needs Clarification):**
*   **Specific Message Broker:** The exact message broker technology (e.g., Kafka, RabbitMQ, Azure Service Bus) is not specified.
*   **Database Technologies:** Specific database systems for each service (e.g., PostgreSQL, SQL Server, NoSQL) are not defined.
*   **CI/CD Pipelines:** Absence of explicit CI/CD definitions implies a manual or unestablished deployment process.
*   **Infrastructure as Code (IaC):** No mention of IaC tools (e.g., Terraform, ARM templates) for provisioning infrastructure.
*   **Testing Strategy:** The lack of explicit test projects indicates an unestablished automated testing strategy.

## 5. Critical Constraints & Risks

**RISK (HIGH) - OutboxProcessor Reliability:** The `OutboxProcessor` is a single point of failure for reliable event publishing for all services. Any issues (bottlenecks, errors, downtime) can lead to data inconsistency across the system, delayed operations, or transactional failures.
    *   **Action:** Implement robust monitoring, alerting, and error handling. Ensure the `OutboxProcessor` is highly available and scalable.

**RISK (HIGH) - Lack of Automated Testing:** The absence of explicit test projects (`*Tests.csproj`) is a significant risk. Without automated unit, integration, and end-to-end tests, changes can easily introduce regressions, leading to unstable software and increased manual testing effort.
    *   **Action:** Establish a clear testing strategy. Implement unit tests for all business logic, integration tests for service interactions, and consider end-to-end tests for critical business flows.

**RISK (HIGH) - Unestablished CI/CD and IaC:** Without automated CI/CD pipelines and Infrastructure as Code, deployments will be manual, error-prone, and inconsistent across environments. This reduces deployment frequency, increases time-to-market, and hinders disaster recovery.
    *   **Action:** Prioritize setting up CI/CD pipelines for automated builds, tests, and deployments. Adopt an IaC tool to define and manage infrastructure.

**RISK (MEDIUM) - Shared Library Coupling:** Changes in the `Shared/` library can have a ripple effect across all dependent services, potentially requiring coordinated deployments.
    *   **Action:** Keep `Shared/` as lean as possible, containing only truly universal types (e.g., event contracts, minimal utilities). Avoid putting business logic or infrastructure-specific code here. Semantic Versioning is crucial.

**CONSTRAINT - Event Contract Management:** Breaking changes to event contracts (payload schema, topic names) can silently break downstream consumers, leading to data inconsistencies or runtime errors.
    *   **Action:** Implement strict versioning for events. Use schema registries (e.g., Confluent Schema Registry) if using Kafka, or document contracts thoroughly.

## 6. Developer Quick Start

To set up and run the system locally, you'll need the .NET SDK (v6.0 or later recommended) and Docker installed. This guide assumes a `docker-compose.yml` file will be created to orchestrate all services and their dependencies (e.g., message broker, databases).

**Planned Files/Services:**
*   `docker-compose.yml` (to be created)
*   Individual service projects: `AccountService/`, `InventoryService/`, `OrchestratorService/`, `OrderMaker/`, `OutboxProcessor/`, etc.

**Steps:**

1.  **Clone the Repository:**
    ```bash
    git clone <your-repository-url>
    cd event-driven-architecture
    ```

2.  **Build All Projects:**
    ```bash
    dotnet build
    ```
    