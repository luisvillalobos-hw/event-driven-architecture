# System Summary

This document provides a high-level overview of the `event-driven-architecture` system, intended for new engineers. It covers the system's purpose, key architectural characteristics, technical stack, knowns and unknowns, critical constraints, and steps to get started with local development.

## 1. System Overview

### 1.1 Purpose
This system implements an event-driven microservices architecture designed to handle core domain operations including account management, inventory management, and order processing. Its primary goal is to demonstrate principles of distributed systems, reliable event publishing, and advanced messaging patterns in a transactional context. The `OrderMaker` library and the outlined `Order Placement` flow indicate a central business function around fulfilling customer orders.

### 1.2 Key Characteristics
*   **Architecture:** FACT (Confirmed) Event-driven microservices. The system is composed of several independent services (`AccountService`, `InventoryService`, `OrchestratorService`, etc.) that communicate predominantly asynchronously through events.
*   **Communication Patterns:** Events are the primary communication mechanism. Key patterns include:
    *   **Transactional Outbox:** FACT (Confirmed) Implemented by `OutboxProcessor` to ensure reliable event publishing from service databases.
    *   **Claim Check:** FACT (Confirmed) Supported by `ClaimCheckProcessor` for handling large message payloads efficiently.
    *   **Message Replication:** FACT (Confirmed) Handled by `MessageReplicator`, potentially for auditing, logging, or inter-broker communication.
*   **Data Approach:** Each core service (e.g., `AccountService`, `InventoryService`) owns its data, implying a service-per-entity data ownership model. Data consistency across services is expected to be eventually consistent, a fundamental aspect of event-driven architectures.
*   **Tech Stack:** FACT (Confirmed) C# and .NET for service implementation, Docker for containerization and local orchestration.

### 1.3 Tech Stack Summary
*   **Languages:** C#
*   **Frameworks:** .NET
*   **Containerization:** Docker
*   **Messaging:** HYPOTHESIS (High Confidence) Requires an external Message Broker (e.g., RabbitMQ, Kafka) for inter-service communication.
*   **Databases:** HYPOTHESIS (High Confidence) Relational databases (e.g., PostgreSQL, SQL Server) for service-specific data persistence and outbox tables. HYPOTHESIS (Medium Confidence) Blob storage for Claim Check payloads.

### 1.4 Confirmed Findings vs. Uncertainties

| Aspect                | Status      | Details                                                                                                                                                                                                                                                             |
| :-------------------- | :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Event-Driven Architecture** | FACT (Confirmed) | Explicitly defined by the repository name and service structure.                                                                                                                                                                                                      |
| **C# / .NET**         | FACT (Confirmed) | Evident from project files (`.csproj`, `Program.cs`, `Worker.cs`).                                                                                                                                                                                                    |
| **Docker Use**        | FACT (Confirmed) | Presence of Dockerfiles suggests containerization.                                                                                                                                                                                                                    |
| **Core Services**     | FACT (Confirmed) | `AccountService`, `InventoryService`, `OrchestratorService`, `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator` are defined.                                                                                                                                 |
| **Reliable Messaging**| FACT (Confirmed) | `OutboxProcessor` confirms transactional outbox pattern. `ClaimCheckProcessor` confirms claim check pattern.                                                                                                                                                          |
| **Specific Message Broker** | HYPOTHESIS (High Confidence) | A message broker is essential for an event-driven system but its specific type (e.g., RabbitMQ, Kafka, Azure Service Bus) is not defined.                                                                                                                    |
| **Specific Database Types** | HYPOTHESIS (High Confidence) | Relational databases are strongly implied for persistence in .NET applications but specific types are not defined. Blob storage is inferred for claim checks.                                                                                                 |
| **CI/CD & IaC**       | HYPOTHESIS (Low Confidence) | No explicit files for Continuous Integration/Deployment or Infrastructure as Code were found. This will need to be planned.                                                                                                                                   |
| **Testing Strategy**  | HYPOTHESIS (Low Confidence) | No explicit test projects or frameworks were identified. This is a critical gap that needs addressing.                                                                                                                                                       |

### 1.5 Critical Constraints & Risks

*   **Eventual Consistency Complexity:** RISK (High) Debugging, tracing, and understanding data flow in an asynchronous, eventually consistent event-driven system is inherently complex. This can lead to challenges in meeting strict consistency requirements or debugging production issues.
    *   **Action:** Implement robust distributed tracing, logging, and monitoring from the outset. Design idempotent consumers.
*   **Infrastructure & Operations:** RISK (High) The current lack of explicit CI/CD pipelines, Kubernetes deployments, or Infrastructure as Code (IaC) poses significant challenges for reliable deployment, scaling, and operational management in production environments.
    *   **Action:** Prioritize definition and implementation of CI/CD pipelines and IaC for all services and infrastructure components.
*   **Transactional Boundaries:** RISK (Medium) While the Outbox pattern handles reliable event publishing, managing complex business transactions that span multiple services (Distributed Transactions) requires careful design to ensure correctness without traditional 2PC.
    *   **Action:** Favor saga patterns or compensative transactions for multi-service workflows, focusing on idempotency and error handling.

## 2. Quick-Start Development Setup

To get the system running locally for development, follow these steps. This setup assumes Docker Desktop is installed and running.

1.  **Clone the Repository:**
    ```bash
    git clone <repository-url>
    cd event-driven-architecture
    ```

2.  **Build the Solution:**
    ```bash
    dotnet build
    ```
    *This will compile all services and libraries.*

3.  **Start Services with Docker Compose:**
    HYPOTHESIS (High Confidence) A `docker-compose.yml` file will be created to orchestrate all services and their dependencies (e.g., message broker, databases).
    ```bash
    docker compose up --build -d
    ```
    *This command will build Docker images (if not already built), create and start containers for all services and their external dependencies (like a message broker and databases) in detached mode.*

4.  **Verify Services:**
    Check the status of your running containers:
    ```bash
    docker ps
    ```
    You should see containers for `AccountService`, `InventoryService`, `OrchestratorService`, `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`, and any inferred infrastructure components (e.g., `messagebroker`, `database`).

5.  **Publish Test Events:**
    Use the `TestPublisher` to send initial events and observe the system's behavior.
    ```bash
    # Navigate to the TestPublisher project directory
    cd TestPublisher
    # Run the publisher (specific commands depend on TestPublisher's implementation)
    dotnet run -- <arguments_for_test_event>
    ```
    *Refer to the `TestPublisher` documentation for specific event publishing commands.*

6.  **Access APIs (if applicable):**
    Once services are running, you can access their HTTP endpoints (e.g., `AccountService`, `InventoryService`, `OrchestratorService`) at their configured ports.
    *   HYPOTHESIS (High Confidence) Default ports will be configured in `Properties/launchSettings.json` or `docker-compose.yml`.

### 2.1 Stopping Services
To stop and remove all containers defined in `docker-compose.yml`:
```bash
docker compose down
```