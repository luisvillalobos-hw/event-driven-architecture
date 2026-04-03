# Architect Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: C4 diagrams, risk hotspots, tech debt, ADRs, integration and security impact assessment

---
# Architect Playbook: Event-Driven Architecture

This playbook guides an AI Architect in understanding and evolving the `event-driven-architecture` project. Your primary focus is on high-level system design, cross-cutting concerns, and ensuring architectural integrity.

## 1. Quick Start

Your initial steps should focus on establishing a foundational understanding of the system's structure and core flows.

*   **System Overview:**
    *   **Scan Solution Structure:** Analyze `event-driven-architecture.sln` to identify all projects and their relationships. Note the distinction between API services (`Program.cs`) and worker services (`Worker.cs`).
    *   **Service Entry Points:** For each service (e.g., `AccountService`, `InventoryService`, `OrchestratorService`), inspect `Program.cs` to understand startup configuration, dependency injection, and hosted services. For worker services (e.g., `OutboxProcessor`, `ClaimCheckProcessor`), examine `Worker.cs` for their main processing loops and message consumption patterns.
*   **Core Event Flow (Outbox Pattern):**
    *   **Outbox Implementation:** Review `OutboxProcessor/OrderOutboxWorker.cs` and `OutboxProcessor/Models/` to understand how events are reliably published from service outbox tables. Identify which services are likely using this pattern (e.g., `AccountService`, `InventoryService`).
    *   **Claim Check Pattern:** Examine `ClaimCheckProcessor/Models/` and infer the processing logic for large messages, typically involving external storage and a subsequent re-publishing or processing.
*   **Shared Contracts:** Review `Shared/` and `OrderMaker/Models/` to identify common DTOs, interfaces, and event definitions that form the communication contracts between services.

## 2. Role-Specific Context

As an Architect AI, your understanding of this project must center on its event-driven nature and the implications of its microservices design.

*   **Event-Driven Core:** The system's backbone is asynchronous event communication. Every design decision must consider event producers, consumers, schemas (`Shared/`, `OrderMaker/Models/`), reliability (`OutboxProcessor`, message broker), and idempotency.
*   **Reliability Patterns:** The presence of `OutboxProcessor` and `ClaimCheckProcessor` indicates a strong emphasis on message delivery guarantees and handling large payloads, which are critical architectural concerns for robust event-driven systems.
*   **Integration Hotspots:** `Shared/` and `OrderMaker/Models/` are central to integration. Changes here have cascading effects across multiple services. `MessageReplicator/` signifies cross-system or cross-topic integration capabilities.
*   **Architectural Decisions (ADRs):** While no explicit ADRs were provided in the analysis, your role includes identifying past implicit decisions and proposing formal ADRs for future significant changes. Look for comments, design patterns, or specific implementations that suggest a deliberate architectural choice.

## 3. Key Areas

| Focus Area               | Key Files/Directories                                     | Patterns & Considerations                                                                                                     |
| :----------------------- | :-------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------- |
| **System Diagramming**   | `event-driven-architecture.sln`, `Program.cs`, `Worker.cs`, `<Service>/Dockerfile` | Map services to containers, identify external dependencies (DB, Message Broker, Blob Storage), define communication channels.  |
| **Risk & Tech Debt**     | `TestPublisher/`, `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/` | Identify single points of failure, potential security flaws, scalability bottlenecks, and reliability gaps.                  |
| **ADR & Decisions**      | (No explicit location found - *Recommend establishing a `/docs/adr/` directory*) | Look for major design choices in code, e.g., Outbox pattern implementation, Claim Check logic. Document trade-offs.            |
| **Integration Assessment** | `Shared/`, `OrderMaker/Models/`, `MessageReplicator/`, all `Program.cs`/`Worker.cs` | Analyze event schemas, message contracts, consumer/producer relationships, and external system interaction points.             |
| **Security Impact**      | `TestPublisher/`, all `Program.cs`, `<Service>/appsettings.json` (inferred) | Identify authentication/authorization points, data at rest/in transit protection needs, and potential external attack vectors. |

## 4. Safe First Changes

These tasks are low-risk and excellent for familiarizing yourself with the project's architecture.

1.  **Generate Initial C4 Diagrams:** Based on `event-driven-architecture.sln` and the identified services, create:
    *   **Context Diagram:** Showing the system's external users and major external systems (e.g., Message Broker, Database, Blob Storage).
    *   **Container Diagram:** Detailing each microservice (`AccountService`, `InventoryService`, etc.) as a container, showing their primary responsibilities and communication channels.
2.  **Document Event Flow for Order Creation:** Trace the `Order Creation and Fulfillment` flow. Identify the originating service, the events published, and the consuming services. Document the role of `OutboxProcessor` and potentially `ClaimCheckProcessor` in this flow.
3.  **Identify Implicit Architectural Decisions:** Review `OutboxProcessor/` and `ClaimCheckProcessor/` code to document the *why* behind their implementation. This helps uncover existing architectural decisions.
4.  **Audit `Shared/` and `OrderMaker/Models/`:** Review these libraries for consistency in naming conventions, data types, and potential opportunities for further modularization or simplification.

## 5. Danger Zones

Proceed with extreme caution in these areas, as changes can have severe and widespread consequences.

*   **Core Reliability Workers (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`):**
    *   **Risk:** Modifying their core logic (e.g., message publishing, payload retrieval, replication strategy) can lead to data loss, inconsistencies, or complete system paralysis.
    *   **Mitigation:** Any changes *must* be accompanied by extensive testing (unit, integration, load) and a detailed ADR outlining the rationale, impact, and rollback strategy.
*   **Shared Contracts (`Shared/`, `OrderMaker/Models/`):**
    *   **Risk:** Altering message schemas or shared interfaces without careful versioning and migration strategies will break communication between services, leading to runtime errors.
    *   **Mitigation:** Always perform a full dependency analysis across all consuming services before proposing changes. Implement robust schema evolution strategies.
*   **`TestPublisher/` Deployment:**
    *   **Risk:** Deploying `TestPublisher` to production environments introduces a significant security vulnerability and a risk of accidental data manipulation or denial of service.
    *   **Mitigation:** Ensure `TestPublisher` is excluded from production builds and deployments. Its presence in the main repository requires explicit vigilance.
*   **External Dependencies Configuration:**
    *   **Risk:** Changes to message broker topics, database schemas, or blob storage configurations without coordinating across all services will cause integration failures.
    *   **Mitigation:** Treat these as critical infrastructure changes requiring explicit architectural review and careful rollout plans.
*   **Absence of CI/CD and IaC (Project-wide Risk):**
    *   **Risk:** Manual deployments or infrastructure setup lead to inconsistencies, errors, and difficulties in scaling or disaster recovery.
    *   **Mitigation:** Prioritize the introduction of CI/CD pipelines and Infrastructure as Code for all services and underlying infrastructure.

## 6. Checklists

### Architectural Decision Record (ADR) Review Checklist

*   [ ] Does the ADR clearly state the decision and its context?
*   [ ] Are the alternatives considered and their trade-offs (pros/cons) explicitly listed?
*   [ ] Is the impact on non-functional requirements (performance, scalability, security, maintainability) analyzed?
*   [ ] Are rollback strategies considered for the decision?
*   [ ] Does it specify the services/components affected by the decision?
*   [ ] Is it stored in an accessible location (e.g., `/docs/adr/` in the repository)?

### Integration Impact Assessment Checklist

*   [ ] Are all services producing/consuming the event identified?
*   [ ] Are changes to event schemas backward compatible? If not, is a versioning strategy in place?
*   [ ] Are data transformation requirements for new or modified contracts understood?
*   [ ] Are new message broker topics or queues required? If so, are they configured?
*   [ ] Are all affected services updated and deployed in the correct order to ensure compatibility?
*   [ ] Are integration tests updated/created for the changes?

### Security Impact Assessment Checklist

*   [ ] Does the change introduce new data ingress/egress points?
*   [ ] Are authentication and authorization mechanisms adequate for new/changed endpoints or event types?
*   [ ] Is data encryption (in transit and at rest) maintained or improved?
*   [ ] Are sensitive configurations protected (e.g., API keys, connection strings)?
*   [ ] Are potential attack surfaces (e.g., `TestPublisher` in dev environments, public APIs) adequately secured?
*   [ ] Has an OWASP Top 10 review been considered for any new API endpoints?

### C4 Diagram Update Checklist

*   [ ] Does the diagram reflect all new or removed services/components?
*   [ ] Are new communication paths and protocols correctly depicted?
*   [ ] Are existing relationships still accurate?
*   [ ] Are external system dependencies (DB, Message Broker, Blob Storage) correctly shown and labeled?
*   [ ] Is the diagram versioned and accessible in the project documentation?