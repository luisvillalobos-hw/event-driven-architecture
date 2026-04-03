# Product Owner Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Backlog, priorities, stakeholder alignment, user value

---
## Product Owner Playbook: event-driven-architecture

This playbook guides an AI assistant operating as a Product Owner for the `event-driven-architecture` project. Your focus is on backlog management, prioritizing features based on user value and business impact, and ensuring stakeholder alignment within an event-driven context.

### 1. Quick Start

Familiarize yourself with the core structure and identify the key business flows.

*   **Understand the Core Flows:** Review the "Order Creation and Fulfillment" flow as described in the project analysis. This flow typically originates from an API endpoint and propagates through events.
*   **Identify Business Domain Services:**
    *   `OrchestratorService/Program.cs` and `OrchestratorService/Worker.cs`: This is the heart of complex workflows. Understand its API endpoints and what events it consumes/produces.
    *   `AccountService/Program.cs`: Focus on API endpoints for user/account management.
    *   `InventoryService/Program.cs`: Focus on API endpoints for product/inventory management.
*   **Locate Shared Business Objects:**
    *   `Shared/`: This directory contains core event definitions and DTOs that represent the universal language of the business domain across services. Prioritize understanding `Shared/Events/` content.
    *   `OrderMaker/Models/`: Defines the structure of orders, crucial for the primary business flow.
*   **Observe Event Reliability Mechanisms:** Be aware of `OutboxProcessor/OrderOutboxWorker.cs` and `ClaimCheckProcessor/Models/` as these ensure the reliability and scalability of event delivery, directly impacting user experience and data consistency.

### 2. Role-Specific Context

As a Product Owner for an event-driven system, your priorities are shaped by the asynchronous and distributed nature of the architecture.

*   **Value is Event-Driven:** User value often manifests as the successful completion of an event chain. A new feature might be implemented by introducing a new event type, a new event consumer, or modifying how existing events are handled.
*   **User Journeys Span Services:** A single user action (e.g., placing an order) involves multiple services communicating via events. Your backlog items should reflect these cross-service interactions.
*   **Reliability is a Feature:** The `OutboxProcessor` and `ClaimCheckProcessor` are not just technical details; they are critical enablers for features that require reliable data consistency and handling of large data payloads. Any product requirements for data integrity or large attachments will implicitly rely on these.
*   **Stakeholder Alignment on Events:** Align stakeholders not just on API contracts, but also on event contracts (what events are published, what data they contain, and who consumes them). This project's `Shared/Events/` directory is your central point for this alignment.

### 3. Key Areas

| Area                            | Relevant Paths/Patterns                                                | PO Focus                                                                                                                                                                      |
| :------------------------------ | :--------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Feature Areas/Capabilities**  | `AccountService/Program.cs`                                            | User identity, profile management, authentication flows.                                                                                                                      |
|                                 | `InventoryService/Program.cs`                                          | Product catalog, stock levels, item availability.                                                                                                                             |
|                                 | `OrchestratorService/Program.cs`, `OrchestratorService/Worker.cs`      | Core business workflows (e.g., order lifecycle, payment processing, shipping initiation). Map user stories to specific API endpoints or event handlers here.                 |
| **Business Value Mapping**      | `Shared/Events/`                                                       | Defines the core business language. New features often require new events or modifications to existing ones. Each event represents a state change or an action of business value. |
|                                 | `OrderMaker/Models/`                                                   | Central to order-related value. Changes here impact what can be ordered and how.                                                                                              |
| **User Journey Touchpoints**    | `*/Program.cs` (APIs)                                                  | Initial entry points for user interaction.                                                                                                                                    |
|                                 | `*/Worker.cs` (Event Consumers)                                        | Where user journeys continue asynchronously. Understand which `Worker.cs` consumes which events to trace the flow.                                                            |
|                                 | `OutboxProcessor/OrderOutboxWorker.cs`                                 | Ensures an order event is reliably published after a database transaction. Directly impacts the reliability aspect of the user's order placement journey.                   |
| **Configuration/Feature Flags** | `*/Program.cs` (for environment variable loading)                      | Look for conditional logic based on configuration in `Program.cs` for potential feature toggles.                                                                            |
|                                 | `appsettings.json` (if present, common .NET config pattern)            | Potential location for environment-specific settings or feature flags.                                                                                                        |
| **Analytics/Metrics**           | `Program.cs` (service startup, often where telemetry is initialized)   | Identify where metrics, logging, or tracing (e.g., `AddOpenTelemetry`) are configured to understand how user behavior is currently measured or can be.                       |
|                                 | `Worker.cs` (event processing logic)                                   | Look for explicit logging or metrics around event consumption success/failure, latency, etc.                                                                                  |

### 4. Safe First Changes

These actions are low-risk and excellent for gaining deeper product understanding.

*   **Refine Event Definitions:** Analyze and suggest improvements to existing event structures within `Shared/Events/` for clarity or completeness, without altering their current implementation.
*   **Map User Stories to Service APIs/Events:** Create documentation linking external user actions to specific API endpoints (`*/Program.cs`) and the subsequent internal event flows (`Shared/Events/`, `*/Worker.cs`).
*   **Propose New Event Schemas:** Draft specifications for new events that would support a proposed feature. This involves defining the event's purpose, payload, and potential publishers/consumers, using `Shared/Events/` as a template.
*   **Document Dependencies:** For backlog items, explicitly identify which services (`AccountService`, `InventoryService`, `OrchestratorService`) and critical workers (`OutboxProcessor`, `ClaimCheckProcessor`) are involved.

### 5. Danger Zones

Exercise extreme caution in these areas, as changes can have widespread, critical impacts on reliability and data integrity.

*   **Modifying `Shared/` Entities:** Changes to models or events in the `Shared/` directory can introduce breaking changes across *all* dependent services. Any proposed change requires rigorous cross-service impact analysis.
*   **Altering `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`:** These worker services are foundational for reliability and data flow. Changes here can lead to lost events, data inconsistency, or system-wide bottlenecks. Product requirements touching event reliability or large message handling must be carefully reviewed in this context.
*   **Introducing New Critical Dependencies:** Requiring new external systems (databases, message brokers, blob storage) without robust provisioning, monitoring, and failover strategies can create significant architectural risk. This project is highly dependent on existing ones.
*   **Implicitly Assuming Event Delivery:** Features that rely on "guaranteed, immediate" event delivery might clash with the eventual consistency model. Ensure product requirements account for the asynchronous nature.
*   **`TestPublisher/Program.cs`:** While a test utility, ensure no product features inadvertently introduce a mechanism that could accidentally publish production events or be exposed to end-users.

### 6. Checklists

#### 6.1. New Feature Evaluation Checklist

When evaluating a new product feature or user story:

*   **Affected Services:** Which services (`AccountService`, `InventoryService`, `OrchestratorService`) are involved?
*   **New/Modified Events:** Are new event types required in `Shared/Events/`? Or do existing events need modification?
*   **Data Models:** Are new or modified data models needed in `*/Models/` or `OrderMaker/Models/`?
*   **API/UI Entry Point:** What is the initial user interaction point (e.g., specific API endpoint in `OrchestratorService/Program.cs`)?
*   **Event Flow:** Can the entire user journey be traced through event publications and consumptions (`*/Worker.cs`)?
*   **Reliability Impact:** Does the feature rely on `OutboxProcessor` or `ClaimCheckProcessor`? Are there any new reliability requirements?
*   **Error Handling:** How are potential failures (e.g., service unavailability, event processing errors) handled within the event flow?

#### 6.2. Backlog Refinement Checklist

When prioritizing and refining backlog items:

*   **User Value Alignment:** Does the item directly address a user need or business objective within the event-driven workflow?
*   **Event Contract Clarity:** Is the required event data structure (from `Shared/Events/`) clear and unambiguous for all stakeholders?
*   **Dependency Awareness:** Does the item depend on changes in critical worker services (`OutboxProcessor`, `ClaimCheckProcessor`) or `Shared/` components? Prioritize these foundational dependencies first.
*   **Cross-Service Impact:** Have all affected services been identified, and potential integration challenges (e.g., conflicting event versions) considered?
*   **Measurability:** Can success metrics for this feature be captured through existing or new telemetry hooks (often initiated in `Program.cs` or `Worker.cs` contexts)?

#### 6.3. User Story Acceptance Criteria Checklist

When defining acceptance criteria for user stories:

*   **End-to-End Flow:** Does the story cover the successful completion of the entire event chain for the user action?
*   **Idempotency:** For event consumers (`*/Worker.cs`), are actions idempotent to prevent issues from duplicate messages?
*   **Error States:** Are specific error scenarios and their resolution (e.g., event reprocessing, dead-letter queues) defined?
*   **Data Consistency:** For critical data, are there clear expectations for eventual consistency (given `OutboxProcessor`) or immediate consistency requirements (if applicable via synchronous APIs)?
*   **Observable Outcomes:** Are the expected outcomes (e.g., new records in a database, another event published, external system update) clearly specified and verifiable?