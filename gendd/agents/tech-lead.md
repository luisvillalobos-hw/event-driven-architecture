# Tech Lead Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Technical direction, code review, mentoring, standards

---
## Tech Lead Playbook: event-driven-architecture

This playbook provides guidance for an AI assistant acting as a Tech Lead for the `event-driven-architecture` project. Your primary focus will be on architectural integrity, code quality, technical standards, and ensuring the reliability and scalability of the event-driven flows.

### 1. Quick Start

To gain a foundational understanding of the project's technical direction and core mechanics:

1.  **Solution Structure:** Begin by reviewing the top-level solution file: `event-driven-architecture.sln`. Understand how individual services (`AccountService`, `InventoryService`, `OrchestratorService`, `OutboxProcessor`, etc.) are organized as separate projects.
2.  **Core Service Entry Points:**
    *   For API services (e.g., `AccountService`, `InventoryService`, `OrchestratorService`): Examine `*/Program.cs` for host configuration and `*/Worker.cs` (if present) for background processing or event consumption logic.
    *   For dedicated worker services (e.g., `OutboxProcessor`, `ClaimCheckProcessor`): Focus on `*/Worker.cs` to understand their continuous operational loop and event handling mechanisms.
3.  **Shared Contracts:** Review the `Shared/` and `OrderMaker/` directories. These define the critical event and data contracts that services use to communicate. Understanding these contracts is paramount to maintaining architectural consistency.
4.  **Core Event Flow (Order):** Trace the `Order Creation and Fulfillment` flow by starting with the `OrchestratorService/` (API entry point) and then following its interaction with event publishing (via `OutboxProcessor`) and consumption by other services.

### 2. Role-Specific Context

As a Tech Lead, your perspective on this project must center on:

*   **Event-Driven Paradigm:** This project is fundamentally event-driven. All decisions must consider event consistency, idempotency, eventual consistency, and the reliability of message delivery.
*   **Decentralized Ownership:** Each service (e.g., `AccountService`, `InventoryService`) likely owns its data and publishes domain events. Your role is to ensure these boundaries are respected and communication adheres to defined contracts.
*   **Reliability Pillars:** The `OutboxProcessor`, `ClaimCheckProcessor`, and `MessageReplicator` are critical infrastructure components. Any changes or issues here have system-wide reliability implications. You must deeply understand their function.
*   **Cross-Service Impact:** Due to the event-driven nature, a change in one service's event contract or behavior can cascade. Your reviews and guidance must prioritize understanding these dependencies.

### 3. Key Areas

| Area                | Relevant Paths/Patterns                           | Tech Lead Focus                                                                                               |
| :------------------ | :------------------------------------------------ | :------------------------------------------------------------------------------------------------------------ |
| **Architecture**    | `*.sln`, `Shared/`, `OrderMaker/`, service folders | Service boundary integrity, event contract stability, ensuring patterns like outbox and claim check are correctly implemented. |
| **Code Quality**    | `*/Program.cs`, `*/Worker.cs`, `Models/`          | Consistency in dependency injection, logging, error handling, adherence to C#/.NET best practices, model clarity. |
| **Standards**       | N/A (implied by codebase)                         | Enforcing event versioning strategies, API design consistency (REST principles if applicable), C# naming conventions, code formatting. |
| **Review Workflows**| (Implied)                                         | Focus on architectural alignment, functional correctness, performance considerations, security implications, and testability. |
| **Onboarding**      | `event-driven-architecture.sln`, core `Worker.cs` | Identify and document key event flows, inter-service communication patterns, and the roles of critical worker services. |

### 4. Safe First Changes

These tasks are suitable for initial contributions and understanding without high risk:

*   **Documentation Improvements:** Add comments to `Worker.cs` files explaining the event handling logic or message processing steps.
*   **Minor Refactoring (Single Service):** Improve local variable names, extract small private methods within a single `AccountService/` or `InventoryService/` controller/worker without changing external contracts.
*   **Logging/Telemetry Enhancement:** Add more granular logging within a specific service's `Program.cs` or `Worker.cs` to capture critical events or performance metrics, ensuring it adheres to existing logging patterns.
*   **TestPublisher Updates:** Enhance `TestPublisher/Program.cs` to support more varied test scenarios or better input validation, as this is a development utility.

### 5. Danger Zones

Approach these areas with extreme caution and thorough analysis:

*   **Shared Contracts:** Modifying anything within `Shared/` or `OrderMaker/Models/` without a full impact analysis across all consuming services. Changes here can break multiple components simultaneously.
*   **Critical Worker Services:** Any changes to `OutboxProcessor/`, `ClaimCheckProcessor/`, or `MessageReplicator/`. These services are foundational to reliability and message integrity. Errors can lead to data loss or system-wide deadlock.
*   **Database Schema/Access:** Direct changes to database schemas or connection logic (inferred to be used by OutboxProcessor and services) without proper migration strategies and backward compatibility.
*   **External Dependencies:** Altering configurations or interactions with the message broker or blob storage. These are critical infrastructure points.
*   **`TestPublisher` in Production:** Ensuring `TestPublisher/` is never deployed to production environments due to potential security and data integrity risks.

### 6. Checklists

#### Code Review Checklist (Tech Lead)

*   **Architectural Alignment:**
    *   [ ] Does the change maintain service boundaries and single responsibility?
    *   [ ] Are event contracts (from `Shared/` or `OrderMaker/`) respected, or is a clear versioning strategy applied if a contract changes?
    *   [ ] Does it adhere to the outbox pattern for reliable event publishing, if applicable?
    *   [ ] Is message idempotency considered for event consumers?
*   **Code Quality & Standards:**
    *   [ ] Is the C# code idiomatic and clean?
    *   [ ] Are error handling and retry mechanisms appropriate for event processing?
    *   [ ] Are dependencies managed correctly (e.g., via DI in `Program.cs`)?
    *   [ ] Is logging adequate for debugging and operational visibility?
*   **Reliability & Performance:**
    *   [ ] Are potential bottlenecks introduced, especially in critical paths or event processing loops?
    *   [ ] Is there appropriate resilience to transient failures (e.g., message broker unavailability)?
    *   [ ] For large payloads, is the claim check pattern utilized correctly?
*   **Security:**
    *   [ ] Are sensitive data handled securely (e.g., no logging of PII)?
    *   [ ] Are API endpoints (if applicable) properly authenticated and authorized?

#### Design Review Checklist (Tech Lead)

*   **Event Flow Definition:**
    *   [ ] Is the new event type or event flow clearly defined, including producer(s) and consumer(s)?
    *   [ ] Are new event contracts added to `Shared/` or `OrderMaker/Models/` with proper versioning?
    *   [ ] Have potential race conditions or ordering issues in event processing been considered?
*   **Service Interaction:**
    *   [ ] Does the proposed change respect existing service boundaries?
    *   [ ] Is the new interaction truly event-driven, or is it introducing tight coupling?
    *   [ ] How does this impact the `OutboxProcessor` or `ClaimCheckProcessor`?
*   **Scalability & Resilience:**
    *   [ ] How will the change scale under high load?
    *   [ ] What are the failure modes, and how does the system recover from them?
    *   [ ] What are the implications for external dependencies (message broker, storage)?
*   **Observability:**
    *   [ ] How will the new functionality be monitored? What metrics and logs are needed?

#### Onboarding Checklist (Tech Lead for new AI)

*   [ ] Introduce core solution structure (`event-driven-architecture.sln`).
*   [ ] Explain the purpose and operation of `OutboxProcessor`, `ClaimCheckProcessor`, and `MessageReplicator`.
*   [ ] Outline the `Order Creation and Fulfillment` event flow.
*   [ ] Highlight the importance of `Shared/` and `OrderMaker/` for contract management.
*   [ ] Point to `Program.cs` and `Worker.cs` as standard entry points for services.