# Business Analyst Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Requirements, acceptance criteria, user stories, process modeling

---
# Business Analyst Playbook for event-driven-architecture

## Quick Start

To quickly grasp the project's business context from a BA perspective, prioritize these actions:

*   **1. Map Core Business Entities & Events:**
    *   **Examine `Shared/` and `OrderMaker/Models/`**: These directories contain the Data Transfer Objects (DTOs) and models that define the core business entities (e.g., `Order`, `Account`, `Product`) and the structure of events exchanged between services. This is your primary glossary.
    *   **Analyze `*Service/Program.cs`**: Understand the HTTP API endpoints exposed by `AccountService`, `InventoryService`, and `OrchestratorService`. These define initial user interaction points and potential triggers for business processes.
    *   **Review `*Service/Worker.cs`**: These files within `AccountService`, `InventoryService`, and `OrchestratorService` define how each service consumes events and implements business logic. Trace the event types they listen for and the actions they perform.
*   **2. Understand Core Workflow:**
    *   **Focus on the "Order Creation and Fulfillment" core flow**: Begin by analyzing `OrchestratorService/Worker.cs` as it's likely central to coordinating this multi-service operation. Then, trace how it interacts with `InventoryService` and potentially `AccountService` via events.
*   **3. Simulate Event Flows (Conceptual):**
    *   **Review `TestPublisher/Program.cs`**: Understand how test events are structured and published. This provides insight into the expected inputs and event schemas the system is designed to handle. You can conceptually run scenarios to see how events are formed.

## Role-Specific Context

As an AI Business Analyst on this project, your focus is on dissecting event-driven flows to define requirements, user stories, and acceptance criteria.

*   **Business Domain and Glossary:** The domain revolves around e-commerce/workflow orchestration (account management, inventory, order fulfillment). Your glossary should be derived directly from the `Models/` directories within `Shared/` and individual services. Event names themselves are key glossary terms (e.g., `OrderCreatedEvent`, `InventoryReservedEvent`).
*   **User Personas and Journey Flows:** User interactions primarily occur via API endpoints (e.g., `AccountService`, `InventoryService`, `OrchestratorService`). Journey flows are event-driven, often starting with an API call publishing an event, which then orchestrates subsequent steps across services. The "Order Creation and Fulfillment" is the most prominent journey.
*   **Acceptance Criteria Patterns:** AC will heavily feature event-based assertions: "When `XEvent` is published with `YPayload`, then `ZService` *should* publish `AEvent` with `BPayload` and/or update its internal state to `C`." Reliable delivery (Outbox Pattern via `OutboxProcessor`) and large message handling (Claim Check via `ClaimCheckProcessor`) are implicit requirements for all critical event-based AC.
*   **Business Rules Embedded in Code:** Look for conditional logic within `*Service/Worker.cs` files. These contain the direct implementation of business rules based on consumed events. `OrderMaker/` might also contain core order-related rules.
*   **Stakeholder Impact Areas:**
    *   **Account Management:** Changes to `AccountService` affect account-related stakeholders.
    *   **Inventory Management:** Changes to `InventoryService` affect inventory/supply chain stakeholders.
    *   **Order Fulfillment/Workflow:** Changes to `OrchestratorService` and the overall event flow impact order fulfillment, customer service, and operational stakeholders.
    *   **System Reliability/Integrity:** `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator` are critical infrastructure affecting *all* stakeholders due to their role in data consistency and event delivery.

## Key Areas

| Aspect               | Relevant Files/Patterns                                                                                                                                                                                                                           | AI BA Focus                                                                                                                                                                                                                                                                                                                                                                   |
| :------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Requirements**     | `*Service/Program.cs` (API endpoints), `*Service/Worker.cs` (event consumers/publishers), `Shared/Models/`, `OrderMaker/Models/` (event/entity definitions)                                                                                         | **User Stories**: Map API endpoints to user actions. **Event Storming**: Understand event flow by analyzing `Worker.cs` logic. **Business Rules**: Extract from `Worker.cs` logic.                                                                                                                                                                                                |
| **Acceptance Criteria** | `Shared/Models/` (event structures), `OutboxProcessor/Models/` (reliability context), `ClaimCheckProcessor/Models/` (large message context), `TestPublisher/Program.cs` (example event payloads)                                            | **Event-driven AC**: Focus on "Given X event, When Y service processes it, Then Z event is published/state is updated." Ensure AC accounts for reliable delivery and large message handling where applicable.                                                                                                                                                                         |
| **Process Modeling** | `OrchestratorService/Worker.cs` (central orchestrator), Interactions implied by `*Service/Worker.cs` consuming/producing events, "Order Creation and Fulfillment" (core flow from project context).                                             | **Event Flow Diagrams (e.g., BPMN/UML Activity)**: Trace event propagation across services. Identify decision points and error paths within `Worker.cs` logic. Model the core workflow and derive sub-processes for each service's role.                                                                                                                                              |
| **Domain Glossary**  | `Shared/Models/`, `OrderMaker/Models/`, Event names (e.g., in `Worker.cs` event handlers), `*Service` names themselves.                                                                                                                              | **Standardize Terminology**: Create a comprehensive glossary based on DTOs, event names, and service responsibilities. Ensure consistency across user stories and AC.                                                                                                                                                                                                                |
| **Business Rules**   | `*Service/Worker.cs` (conditional logic in event handlers), `OrderMaker/` (shared order logic).                                                                                                                                                     | **Extract & Document**: Identify `if/else`, switch statements, or data validations within `Worker.cs` that represent business rules. Document them explicitly as part of requirements.                                                                                                                                                                                              |
| **Stakeholder Impact** | Project Purpose, `*Service` names, Risk Hotspots (e.g., `OutboxProcessor` reliability).                                                                                                                                                            | **Identify Affected Groups**: Determine which business units or user types are impacted by changes to specific services or event flows. Assess criticality based on risk hotspots.                                                                                                                                                                                                    |

## Safe First Changes

These tasks are low-risk and excellent for initial engagement to build understanding:

*   **Document Event Payloads:** For each `*Event` DTO found in `Shared/Models/` and `OrderMaker/Models/`, create detailed documentation including fields, data types, and business meaning.
*   **Map Existing API Endpoints to Business Functions:** For `AccountService/Program.cs`, `InventoryService/Program.cs`, and `OrchestratorService/Program.cs`, list each exposed endpoint and describe its presumed business purpose.
*   **Draft User Stories for Core Flows:** Based on the "Order Creation and Fulfillment" flow, draft initial user stories that articulate the value delivered, identifying the primary actors and their goals.
*   **Create Initial Process Diagrams:** For the "Order Creation and Fulfillment" flow, sketch a high-level event flow diagram showing the interaction between `OrchestratorService`, `InventoryService`, and other relevant components, indicating key events.
*   **Identify Business Rules in `OrderMaker/`:** Read through `OrderMaker/Models/` and any associated logic to identify explicit business rules related to order creation or modification.

## Danger Zones

Areas requiring extreme caution and thorough analysis from a BA perspective:

*   **Modifying Core Event Contracts:** Changing the structure of DTOs/Events in `Shared/Models/` or `OrderMaker/Models/` without a full impact analysis across all consuming services. This will break interoperability.
*   **Assumptions on Reliability:** Proposing changes that don't account for the critical role of `OutboxProcessor` and `ClaimCheckProcessor` in ensuring data consistency and handling large messages. Any AC should consider how these mechanisms are involved.
*   **Altering Orchestration Logic:** Directly proposing changes to the event sequencing or business rules within `OrchestratorService/Worker.cs` without understanding all dependent services and potential side effects.
*   **Overlooking `TestPublisher` Implications:** Assuming behavior based solely on `TestPublisher/Program.cs` without verifying actual service implementations. `TestPublisher` is a utility, not necessarily the source of truth for all business rules.
*   **Ignoring External Dependencies:** Not considering how changes to event flows or service logic might impact the message broker or underlying databases, which are high-risk dependencies.

## Checklists

### User Story Definition Checklist

*   **[ ] Actor Defined:** Is the primary user/system interacting with the feature clearly identified?
*   **[ ] Goal Stated:** Is the desired outcome or value for the actor clearly articulated?
*   **[ ] Service Scope:** Does the story align with a specific service's responsibility (e.g., `AccountService` for account management)?
*   **[ ] Event Trigger/Outcome:** Does the story imply either an API call (refer `Program.cs`) or the consumption/publication of a specific event (refer `Worker.cs`, `Shared/Models/`)?
*   **[ ] Testable:** Is the story concise enough to derive clear acceptance criteria?

### Acceptance Criteria (AC) Review Checklist

*   **[ ] Event-Driven Focus:** Does each AC clearly state which event triggers it, and what resulting events are expected or what state changes occur?
*   **[ ] Payload Specificity:** If events are involved, does the AC reference specific fields within the event payload (from `Shared/Models/`, `OrderMaker/Models/`)?
*   **[ ] Outbox Pattern Implicit:** For state-changing operations that publish events, does the AC implicitly (or explicitly, if critical) require atomic delivery (via `OutboxProcessor`)?
*   **[ ] Claim Check Consideration:** If large payloads are anticipated, does the AC account for how they are handled (e.g., via `ClaimCheckProcessor`)?
*   **[ ] Edge Cases Covered:** Does the AC address success paths, alternative paths, and error conditions relevant to the event flow?
*   **[ ] Clear and Unambiguous:** Is the AC written in a way that developers can implement and testers can verify without ambiguity?

### Process Modeling Review Checklist

*   **[ ] All Services Identified:** Are all involved services (e.g., `AccountService`, `InventoryService`, `OrchestratorService`, workers) accurately represented in the flow?
*   **[ ] Key Events Mapped:** Are all significant events, their producers, and consumers clearly depicted?
*   **[ ] Event Flow Logical:** Does the sequence of events make business sense and align with the observed `Worker.cs` logic?
*   **[ ] Decision Points Represented:** Are any conditional branches or business rules that alter the flow (from `Worker.cs` conditionals) clearly shown?
*   **[ ] External System Integration:** Are any interactions with external systems (e.g., payment gateways, external APIs) indicated, even if conceptual?
*   **[ ] Error Handling/Compensating Actions:** Are primary error paths or compensating transactions for failures (e.g., `MessageReplicator` for recovery, or logic in `Worker.cs`) conceptually included?