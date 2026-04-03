# Architecture Overview

This system is engineered as an **event-driven microservices platform** designed for processing and orchestrating complex business workflows, such as order fulfillment, within an e-commerce or similar domain. It emphasizes loose coupling, scalability, and resilience through asynchronous communication patterns.

## 1. Architecture Pattern Description

The core architectural pattern is **Event-Driven Microservices Architecture**. This approach decomposes the system into small, independently deployable services that communicate primarily through asynchronous events.

*   **Characteristics**:
    *   **Domain-Driven Design**: Services (`AccountService`, `InventoryService`, `OrchestratorService`) align with distinct business capabilities.
    *   **Asynchronous Communication**: Services publish and subscribe to events via a Message Broker. This promotes loose coupling and allows services to react to changes without direct dependencies.
    *   **Transactional Outbox Pattern**: Each service publishing events ensures atomicity between its local database transaction and event publication by writing events to an outbox table. The `OutboxProcessor` then reliably publishes these events to the Message Broker.
    *   **Claim Check Pattern**: For handling large event payloads that exceed Message Broker limits or are costly to transmit repeatedly, a claim check pattern is employed. Services store large data in blob storage and publish a message containing only a reference. The `ClaimCheckProcessor` is responsible for retrieving the actual payload.
    *   **Polyglot Persistence**: Each service is responsible for its own data store, allowing choice of the most suitable database technology for its specific needs.

*   **Evidence**:
    *   FACT (Confirmed): Repository name `event-driven-architecture` directly states the pattern.
    *   FACT (Confirmed): Multiple `Worker.cs` files in distinct service directories (`AccountService/Worker.cs`, `InventoryService/Worker.cs`, `OrchestratorService/Worker.cs`) indicate event consumption.
    *   FACT (Confirmed): Presence of `OutboxProcessor` and `OrderOutboxWorker.cs` confirms the transactional outbox pattern for reliable event publishing.
    *   FACT (Confirmed): Dedicated `ClaimCheckProcessor` directory confirms the claim check pattern for large messages.
    *   FACT (Confirmed): Distinct service directories (`AccountService`, `InventoryService`, `OrchestratorService`) with their own entry points (`Program.cs`, `Worker.cs`) confirm independent deployable units (microservices).

## 2. High-Level Architecture Diagram

```ascii
+-----------------+     +-----------------+     +-----------------+
|   External      |     |   API Gateway   |     |  API Clients    |
|   Client/User   | <---|   (Planned)     |<--->|   (e.g., Mobile)|
+-----------------+     +-------+---------+     +-----------------+
                                | HTTP/REST
                                V
                 +-------------------------------------------------+
                 |        Load Balancing & Routing (Planned)       |
                 +-------------------------------------------------+
                                | HTTP/REST
          +----------------------------------------------------------------------------------+
          | Service Layer                                                                    |
          |                                                                                  |
          |    +-------------------+    +-------------------+    +-----------------------+   |
          |    | AccountService    |    | InventoryService  |    | OrchestratorService   |<--| (Optional Sync Calls)
          |    | (API & Worker)    |    | (API & Worker)    |    | (API & Worker)        |   |
          |    +---------+---------+    +---------+---------+    +-----------+-----------+   |
          |              |                      |                        |                    |
          |              V                      V                        V                    |
          |      +---------------+      +---------------+      +-------------------+        |
          |      |  AccountDB    |      |  InventoryDB  |      | OrchestratorDB    |        |
          |      | (Relational)  |      | (Relational)  |      | (Relational)      |        |
          |      +---------------+      +---------------+      +-------------------+        |
          |                                 |                                                 |
          +---------------------------------|-------------------------------------------------+
                                            | (Transactional Outbox)
                                            V
                          +---------------------------------+
                          |        Outbox Processor         |
                          |   (OrderOutboxWorker.cs)        |
                          +---------------------------------+
                                            | (Publishes Events)
                                            V
               +-------------------------------------------------------------+
               |                       Message Broker                        |
               |       (e.g., Kafka, RabbitMQ, Azure Service Bus)            |
               +-------------------------------------------------------------+
                 ^   ^   ^   ^   ^   ^   ^   ^   ^   ^   ^   ^   ^   ^   ^
                 |   |   |   |   |   |   |   |   |   |   |   |   |   |   | (Consumes Events)
                 |   |   |   |   |   |   |   |   |   |   |   |   |   |   +-------------------+
                 |   |   |   |   |   |   |   |   |   |   |   |   +--------------------------+
                 |   |   |   |   |   |   |   |   +------------------------------------------+
                 |   |   +------------------------------------------------------------------+
                 V   V
    +-------------------+           +-------------------+         +-------------------+
    | ClaimCheckProcessor |         | MessageReplicator |         | Other Consumers   |
    | (Worker)          |<-------->| (Worker)          |<------->| (e.g., Analytics) |
    +---------+---------+           +-------------------+         +-------------------+
              | (Blob Storage I/O)
              V
    +-------------------+
    | Blob/Object Storage|
    | (Large Payloads)  |
    +-------------------+
```

## 3. Service Communication Patterns

*   **Asynchronous Eventing (Primary)**:
    *   **Publish/Subscribe**: Services publish domain events to a shared Message Broker, and other interested services subscribe to these events.
    *   **Reliable Event Publishing**: Utilizes the **Transactional Outbox Pattern** via the `OutboxProcessor`. This guarantees that events are published only if the local database transaction commits successfully.
        *   **Actionable**: Ensure robust monitoring and alerting for the `OutboxProcessor` to detect processing delays or failures.
    *   **Large Message Handling**: Implements the **Claim Check Pattern** via the `ClaimCheckProcessor`. Large event payloads are stored in Blob/Object Storage, and the message broker only carries a reference (claim check). The `ClaimCheckProcessor` retrieves the full payload when needed.
        *   **Actionable**: Configure appropriate retention policies and security for the Blob/Object Storage.
    *   **Event Replication**: The `MessageReplicator` service suggests the ability to copy events between different topics or brokers, potentially for fan-out to external systems, auditing, or analytics.
*   **Synchronous API Calls (Secondary/Inferred)**:
    *   HYPOTHESIS (Medium Confidence): The `OrchestratorService` lists `AccountService` and `InventoryService` as dependencies, suggesting it might make direct HTTP/REST calls to these services for certain command-driven interactions, especially within a saga orchestration.

## 4. Routing and Gateway Layers

*   **API Gateway**:
    *   HYPOTHESIS (High Confidence): An API Gateway is planned for external ingress. This component will serve as the single entry point for client applications, handling concerns such as authentication, authorization, rate limiting, and request routing to the appropriate microservices.
        *   **Actionable**: Define the API Gateway's responsibilities, technology choice (e.g., Ocelot, Azure API Management, NGINX), and deployment strategy early in the project.
*   **Load Balancing & Internal Routing**:
    *   HYPOTHESIS (High Confidence): Behind the API Gateway, a load balancer and service discovery mechanism will distribute incoming requests across multiple instances of API-exposed services (`AccountService`, `InventoryService`, `OrchestratorService`). Internal service-to-service communication (if synchronous) would rely on service discovery or a service mesh.
        *   **Actionable**: Research and select a service discovery solution (e.g., Kubernetes DNS, Consul) and a load balancing strategy.

## 5. Database Architecture

*   **Polyglot Persistence**:
    *   FACT (Confirmed): Each core service (`AccountService`, `InventoryService`, `OrchestratorService`) owns and manages its dedicated data store. This prevents direct coupling between services via a shared database.
*   **Relational Databases**:
    *   FACT (Confirmed): The use of C#/.NET and the transactional outbox pattern strongly implies the primary use of **Relational Databases** (e.g., SQL Server, PostgreSQL, MySQL) for most service data and outbox tables.
        *   **Actionable**: Standardize on a specific relational database technology and define schema migration strategies.
*   **Blob/Object Storage**:
    *   FACT (Confirmed): Leveraged by the `ClaimCheckProcessor` to store large event payloads. This provides a cost-effective and scalable solution for unstructured data.
        *   **Actionable**: Implement secure access controls and data lifecycle management for the chosen Blob/Object Storage.
*   **Outbox Tables**:
    *   FACT (Confirmed): Each service that publishes events will maintain an 'outbox' table within its own database to ensure transactional integrity. The `OutboxProcessor` will monitor and publish entries from these tables.

## 6. Security Boundaries

*   **Service Isolation**: Each microservice runs as an independent process, enforcing a natural security boundary at the process level.
*   **API Gateway Security (Planned)**:
    *   HYPOTHESIS (High Confidence): The planned API Gateway will be the primary enforcement point for external security, handling authentication (Who is this user?) and authorization (What can this user do?).
    *   **Actionable**: Implement robust authentication (e.g., OAuth2/OpenID Connect) and fine-grained authorization policies at the API Gateway.
*   **Internal Communication Security**:
    *   HYPOTHESIS (Medium Confidence): Internal synchronous API calls between services should be secured, potentially using mTLS (mutual TLS) or token-based authentication (e.g., JWTs) to ensure only authorized services can communicate.
    *   **Actionable**: Define and implement an internal service-to-service authentication and authorization strategy.
*   **Data Ownership and Access**: Strict data ownership by services limits the attack surface and helps enforce data access boundaries.
*   **Risk: TestPublisher**:
    *   **RISK (High)**: The `TestPublisher` (TestPublisher/Program.cs), if deployed to production or accessible in unsecured environments, poses a significant risk. It can bypass normal application logic and directly inject events into the system, potentially leading to data corruption or service disruption.
    *   **