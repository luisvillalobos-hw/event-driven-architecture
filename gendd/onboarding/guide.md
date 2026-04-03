# Developer Onboarding Guide: Event-Driven Architecture

Welcome to the `event-driven-architecture` project! This guide provides a rapid introduction to our system, outlining its core purpose, architecture, development setup, and key areas to focus on.

## 1. System Overview

This system is an event-driven microservices platform designed for processing and orchestrating business workflows, primarily focusing on e-commerce or similar domains. It emphasizes reliable event publishing and handles large message payloads efficiently.

### 1.1 Architecture & Core Concepts

FACT (Confirmed): The system employs an **event-driven microservices architecture**. This is evidenced by the repository name, multiple independent `Worker.cs` files across distinct service directories, and specialized components for event handling.

**Key Architectural Patterns:**
*   **Transactional Outbox Pattern**: Ensures atomicity between local database transactions and message publication. Implemented by the `OutboxProcessor`.
*   **Claim Check Pattern**: Handles large message payloads by storing them externally (e.g., blob storage) and passing a reference in the event. Managed by the `ClaimCheckProcessor`.
*   **Microservices**: Services are independently deployable units, each owning specific business capabilities (e.g., `AccountService`, `InventoryService`, `OrchestratorService`).

### 1.2 Tech Stack

FACT (Confirmed):
*   **Languages & Frameworks**: C#, .NET
*   **Containerization**: Docker (implied by `.dockerignore`)

HYPOTHESIS (Confidence: High):
*   **Message Broker**: Essential for event-driven communication (e.g., Kafka, RabbitMQ, Azure Service Bus).
*   **Relational Database**: For persistent storage in services and the Outbox (e.g., PostgreSQL, SQL Server).
*   **Blob/Object Storage**: For storing large payloads in the Claim Check pattern (e.g., Azure Blob Storage, AWS S3).

## 2. Reading Order for Documentation

To effectively onboard, we recommend the following reading order for *this* guide and related resources:

1.  **System Overview (Section 1 of this document)**: Grasp the high-level purpose and architectural patterns.
2.  **Services & Data Map (Section 3 & 4 of this document)**: Understand the responsibilities of each microservice and its data ownership.
3.  **Core Flows (Section 5 of this document)**: See how services collaborate to achieve business outcomes (e.g., Order Creation).
4.  **Planned Development Setup (Section 6 of this document)**: Get your local environment up and running.
5.  **Recommended First Tasks (Section 7 of this document)**: Begin contributing with guided tasks.
6.  **Conventions & Testing (Section 9 & 10 of this document)**: Familiarize yourself with our coding standards and testing approach.
7.  **Dangerous Zones (Section 8 of this document)**: Understand high-risk areas requiring extra caution.

## 3. Services

The system comprises several distinct services and libraries:

| Service/Library        | Type     | Description                                                                                                                                                                                                                                                                                           | Key Files                                                | Dependencies                                                                                                                                                                            |
| :--------------------- | :------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AccountService`       | API      | Manages user accounts and related business logic. Consumes and publishes account-related events.                                                                                                                                                                                                       | `AccountService/Program.cs` `AccountService/Worker.cs`   | Message Broker, Database, `Shared`                                                                                                                                                      |
| `InventoryService`     | API      | Manages product inventory and stock levels. Consumes and publishes inventory-related events.                                                                                                                                                                                                           | `InventoryService/Program.cs` `InventoryService/Worker.cs` | Message Broker, Database, `Shared`                                                                                                                                                      |
| `OrchestratorService`  | API      | Coordinates complex business workflows (e.g., order fulfillment) by reacting to and publishing events from other services. Acts as the central orchestrator for multi-service operations. | `OrchestratorService/Program.cs` `OrchestratorService/Worker.cs` | Message Broker, Database, `AccountService`, `InventoryService`, `Shared`, `OrderMaker`                                                                                                  |
| `OutboxProcessor`      | Worker   | **Critical reliability component.** Monitors service outbox tables to reliably publish domain events to the Message Broker, ensuring transactional consistency.                                                                                                                                    | `OutboxProcessor/OrderOutboxWorker.cs`                   | Database (for outbox entries), Message Broker, `Shared`                                                                                                                                 |
| `ClaimCheckProcessor`  | Worker   | Processes 'claim check' messages by retrieving large payloads from external storage, then re-publishing the full message or processing it directly.                                                                                                                                                 | `ClaimCheckProcessor/Models/` (Worker logic planned)     | Message Broker, Blob/Object Storage, `Shared`                                                                                                                                           |
| `MessageReplicator`    | Worker   | Replicates messages between message broker topics/instances, potentially for fan-out, integration, or disaster recovery.                                                                                                                                                                              | `MessageReplicator/` (Worker logic planned)              | Source Message Broker, Target Message Broker, `Shared`                                                                                                                                  |
| `TestPublisher`        | CLI      | A utility for manually publishing test events, primarily for development, debugging, and integration testing.                                                                                                                                                                                        | `TestPublisher/Program.cs`                               | Message Broker, `Shared`, `OrderMaker`                                                                                                                                                  |
| `OrderMaker`           | Library  | Provides shared models and business logic related to creating and managing orders, consumed by other services.                                                                                                                                                                                         | `OrderMaker/Models/`                                     | `Shared`                                                                                                                                                                                |
| `Shared`               | Library  | Contains common definitions, DTOs, interfaces, and utilities shared across multiple services to promote consistency.                                                                                                                                                                                   | `Shared/`                                                | None                                                                                                                                                                                    |

## 4. Data Map

Data ownership and storage within the system:

| Data Type             | Owned By          | Storage                                | Notes                                                                                    |
| :-------------------- | :---------------- | :------------------------------------- | :--------------------------------------------------------------------------------------- |
| Account Data          | `AccountService`  | HYPOTHESIS (High): Relational Database | API for commands/queries, Worker for event-driven updates.                               |
| Inventory Data        | `InventoryService`| HYPOTHESIS (High): Relational Database | API for commands/queries, Worker for event-driven updates.                               |
| Order/Workflow State  | `OrchestratorService`| HYPOTHESIS (High): Relational Database | Implements Saga pattern; Worker for event-driven state updates.                          |
| Outbox Messages       | All services using outbox| HYPOTHESIS (High): Relational Database | Transient records; `OutboxProcessor` reads, Message Broker consumes.                     |
| Claim Check Payloads  | `ClaimCheckProcessor` & publishing service | HYPOTHESIS (High): Blob/Object Storage | Event carries reference; `ClaimCheckProcessor` retrieves payload.                        |

## 5. Core Flows: Order Creation and Fulfillment

This end-to-end flow illustrates the event-driven interactions for processing an order:

```ascii
+-------------+      +-------------------+      +-----------------+      +-----------------+
|   Client    | API  | OrchestratorService | DB   |  Orchestrator   | Outbox | OutboxProcessor |
| (e.g., API) |----->| (1. Order Request)|----->|   Service DB    |-------->|   (3. Publish   |
+-------------+      +-------------------+      +-----------------+        |  OrderRequested)|
                                                                             +-----------------+
                                                                                      |
                                                                                      v
                                                                             +-----------------+
                                                                             | Message Broker  |
                                                                             +-----------------+
                                                                                      |
         +-----------------+                                                          |
         |    Inventory    |<---------------------------------------------------------+
         |     Service     |
         | (5. Reserve Inv.)|------------------------------------+
         +-----------------+                                    |
                 |DB                                             |
                 v                                               |
         +-----------------+                                    |
         | Inventory DB    |                                    |
         +-----------------+                                    |
                 |Outbox                                         |
                 v                                               |
         +-----------------+                                    |
         | OutboxProcessor |------------------------------------|
         | (6. Publish Inv.)|                                    |
         +-----------------+                                    |
                 |                                                |
                 v                                                |
         +-----------------+                                    |
         | Message Broker  |<------------------------------------+
         +-----------------+
                 |
                 v
         +-----------------+
         |    Account      |<---------------------------------------------------------+
         |     Service     |
         | (8. Charge Acct.)|------------------------------------+
         +-----------------+                                    |
                 |DB                                             |
                 v                                               |
         +-----------------+                                    |
         | Account DB      |                                    |
         +-----------------+                                    |
                 |Outbox                                         |
                 v                                               |
         +-----------------+                                    |
         | OutboxProcessor |------------------------------------|
         | (9. Publish Acct.)|                                   |
         +-----------------+                                    |
                 |                                                |
                 v                                                |
         +-----------------+                                    |
         | Message Broker  |<------------------------------------+
         +-----------------+
                 |
                 v
         +-------------------+
         | OrchestratorService |
         | (11. Update Order  |<----------------------------------+ (Consumed: InventoryReserved, AccountCharged)
         |     Status)       |
         +-------------------+
                 |DB
                 v
         +-----------------+
         |  Orchestrator   |
         |   Service DB    |
         +-----------------+
```

**Steps:**
1.  **Client Request**: External client (or `TestPublisher`) calls `OrchestratorService` API to request an order.
2.  **Orchestrator Initiates**: `OrchestratorService` validates, saves initial order state to its DB, and publishes an 'OrderRequested' event to its outbox.
3.  **Outbox Publishing (Orchestrator)**: `OutboxProcessor` detects 'OrderRequested' in `OrchestratorService`'s outbox and publishes it to the Message Broker.
4.  **Inventory Service Consumes**: `InventoryService/Worker.cs` consumes 'OrderRequested', reserves inventory, updates its local state, and publishes 'InventoryReserved' (or 'InventoryFailed') to its outbox.
5.  **Outbox Publishing (Inventory)**: `OutboxProcessor` detects 'InventoryReserved' and publishes it to the Message Broker.
6.  **Account Service Consumes**: `AccountService/Worker.cs` consumes 'InventoryReserved', attempts to charge the account, updates its local state, and publishes 'AccountCharged' (or 'AccountFailed') to its outbox.
7.  **Outbox Publishing (Account)**: `OutboxProcessor` detects 'AccountCharged' and publishes it to the Message Broker.
8.  **Orchestrator Finalizes**: `OrchestratorService/Worker.cs` consumes 'AccountCharged' and 'InventoryReserved' events, updating the order status to 'Fulfilled' (or 'Failed') in its database.
9.  **Optional Claim Check**: If the initial request was large, `OrchestratorService` could publish a 'ClaimCheckCreated' event, and `ClaimCheckProcessor` would retrieve the payload for processing.
10. **Optional Replication**: `MessageReplicator` could replicate these events for analytics or other systems.

## 6. Planned Development Setup

This project uses .NET and Docker for local development.

**Prerequisites:**
*   .NET SDK (latest stable version)
*   Docker Desktop (or equivalent Docker engine for your OS)
*   Git
*   An IDE (Visual Studio, VS Code with C# Dev Kit, Rider)

**Setup Steps:**

1.  **Clone the Repository:**
    ```bash
    git clone [repository-url]
    cd event-driven-architecture
    ```

2.  **Build All Projects:**
    Open the solution file (`event-driven-architecture.sln`) in your IDE and build all projects. This will ensure all dependencies are resolved. Alternatively, from the root:
    ```bash
    dotnet build
    ```

3.  **Start Local Dependencies (Message Broker, Database, Blob Storage):**
    HYPOTHESIS (Confidence: High): We will use `docker-compose` to manage local infrastructure.
    ```bash
    # (Planned) Ensure a docker