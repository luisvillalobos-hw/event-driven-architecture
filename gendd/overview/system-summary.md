# System Summary

This document provides an executive overview of the `event-driven-architecture` system, outlining its purpose, core characteristics, technology stack, and critical considerations for engineers joining the project.

## System Overview

The system is an **event-driven microservices platform** engineered to process and orchestrate complex business workflows, specifically in domains like e-commerce, involving account management, inventory tracking, and order fulfillment. It achieves high reliability and scalability through asynchronous communication, leveraging an **Outbox Pattern** for reliable event publishing and a **Claim Check Pattern** for handling large message payloads efficiently.

## Key Characteristics

*   **Architecture Classification**: FACT (Confirmed) - Event-driven Microservices.
    *   Individual services (`AccountService`, `InventoryService`, `OrchestratorService`) operate as independent deployable units, communicating primarily through events.
*   **Communication Patterns**:
    *   **Asynchronous Messaging**: Services publish and subscribe to events via a central Message Broker.
    *   **Transactional Outbox**: FACT (Confirmed) - The `OutboxProcessor` ensures atomic event publishing by integrating message publication with local database transactions.
    *   **Claim Check**: FACT (Confirmed) - The `ClaimCheckProcessor` manages large message payloads by storing them externally (e.g., in object storage) and transmitting only a reference in the message.
*   **Data Approach**:
    *   **Decentralized Data Ownership**: Each microservice owns its domain data (e.g., `AccountService` owns account data).
    *   **Eventual Consistency**: Data across services achieves consistency over time through event propagation.
    *   **Polyglot Persistence (Hypothesis)**: While primarily relational databases are inferred, specific database types and potential use of other storage are undetermined.

## Tech Stack

*   **Core Language & Platform**: C#, .NET
*   **Containerization**: Docker
*   **Messaging**: Message Broker (HYPOTHESIS: e.g., Kafka, RabbitMQ)
*   **Data Storage**:
    *   Relational Database (HYPOTHESIS: e.g., PostgreSQL, SQL Server) for transactional data and outbox tables.
    *   Blob/Object Storage (HYPOTHESIS: e.g., Azure Blob Storage, AWS S3) for large claim check payloads.
*   **Shared Libraries**: `Shared` for common DTOs/utilities, `OrderMaker` for order-related models/logic.

## Fact vs. Uncertainty

*   **FACT (Confirmed)**:
    *   System is an event-driven microservices architecture.
    *   Uses C#/.NET and Docker for development and deployment.
    *   Transactional Outbox Pattern (`OutboxProcessor`) and Claim Check Pattern (`ClaimCheckProcessor`) are core to message reliability and large payload handling.
    *   Specific services: `AccountService`, `InventoryService`, `OrchestratorService`, `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`, `TestPublisher`, `OrderMaker`, `Shared`.
*   **HYPOTHESIS (Confidence: High)**:
    *   Specific Message Broker technology (e.g., Kafka, RabbitMQ).
    *   Specific Relational Database technology (e.g., PostgreSQL, SQL Server).
    *   Specific Blob/Object Storage provider.
    *   Need for explicit CI/CD and Infrastructure as Code (IaC) is critical but not yet implemented.
    *   Lack of robust automated testing frameworks.

## Critical Constraints & Risks

*   **HIGH RISK: External Dependencies**: The system critically depends on robust Message Broker and Database infrastructure. Undefined configurations or mismanagement of these external components pose significant risks to resilience, performance, and data integrity.
    *   **Action**: Prioritize defining, configuring, and securing these dependencies, ideally with Infrastructure as Code (IaC).
*   **MEDIUM RISK: Operational Bottlenecks**: Critical worker services (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`) are central to event flow. Failures or performance issues in these can halt event processing, leading to data inconsistencies or system performance degradation.
    *   **Action**: Implement comprehensive monitoring, alerting, and auto-scaling for these worker services.
*   **MEDIUM RISK: Lack of CI/CD & IaC**: Without explicit CI/CD pipelines and IaC, deployments are prone to manual errors, inconsistency, and hinder rapid iteration or recovery.
    *   **Action**: Establish CI/CD pipelines and begin implementing IaC for infrastructure provisioning and configuration.
*   **LOW RISK: `TestPublisher` in Production**: The `TestPublisher` is useful for development but could pose a security risk or lead to accidental data corruption if deployed to production environments without proper securing or removal.
    *   **Action**: Ensure `TestPublisher` is excluded from production builds or secured with stringent access controls.

## Quick Start (Development)

To get a local development environment running:

1.  **Prerequisites**: Ensure Docker Desktop (or equivalent Docker engine) is installed and running.
2.  **Clone Repository**:
    ```bash
    git clone <repository_url> event-driven-architecture
    cd event-driven-architecture
    ```
3.  **Start Infrastructure (HYPOTHESIS)**:
    Assuming a `docker-compose.yml` will be provided for local Message Broker and Database:
    ```bash
    docker compose up -d
    ```
4.  **Build & Run Services**:
    To run all services concurrently (example, using a shell script or IDE):
    ```bash
    # Example for AccountService
    cd AccountService
    dotnet run
    # Repeat for InventoryService, OrchestratorService, OutboxProcessor, ClaimCheckProcessor, MessageReplicator
    ```
    Alternatively, open the `.sln` file in an IDE (e.g., Visual Studio Code, Visual Studio) and run desired projects.
5.  **Publish Test Events**:
    Use the `TestPublisher` to simulate event flows:
    ```bash
    cd TestPublisher
    dotnet run # Follow prompts or provide arguments as designed
    ```
    *(Note: Specific `dotnet run` arguments for `TestPublisher` will need to be documented once defined.)*