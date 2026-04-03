# QA Engineer Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Test planning, AC validation, exploratory testing

---
## QA Engineer Playbook: event-driven-architecture

This playbook guides an AI assistant in performing QA engineering tasks for the `event-driven-architecture` project, focusing on test planning, Acceptance Criteria (AC) validation, and exploratory testing within an event-driven context.

### 1. Quick Start

Your primary goal is to understand the flow of events and data through the system, identify discrepancies, and validate business logic against ACs.

1.  **Initial Scan:** Analyze `event-driven-architecture.sln` to identify all projects. Prioritize `AccountService`, `InventoryService`, `OrchestratorService` (API entry points) and `OutboxProcessor`, `ClaimCheckProcessor` (critical event workers).
2.  **Understand Core Flow:** Review the `Order Creation and Fulfillment` core flow. This involves `OrchestratorService` and interaction with other services via events. Trace potential event paths using files in `Shared/` and `OrderMaker/Models/`.
3.  **Simulate Inputs:**
    *   To initiate events for testing, inspect `TestPublisher/Program.cs`. Understand its parameters and capabilities for sending various event types.
    *   For direct API interaction, analyze `Program.cs` files in `AccountService/`, `InventoryService/`, `OrchestratorService/` to identify exposed endpoints and expected request/response schemas.
4.  **Observe System State:** Develop strategies to monitor message broker queues (simulated or actual), database states (especially outbox tables for `OutboxProcessor`), and service logs to track event processing.

### 2. Role-Specific Context

As a QA AI, your focus is on validating the *eventual consistency* and *correctness* of data propagation.

*   **Asynchronous Nature:** Expect delays between an action and its observed effect. AC validation must account for eventual state, not immediate.
*   **Event Contracts:** The `Shared/` and `OrderMaker/Models/` directories define the structure of events. Any deviation in event publication or consumption from these contracts is a critical defect.
*   **Outbox Pattern:** Understand that services publish events transactionally via an outbox (processed by `OutboxProcessor`). Test scenarios must consider the atomic nature of database writes and event publishing.
*   **Claim Check Pattern:** For large messages, `ClaimCheckProcessor` is vital. Test cases should cover scenarios with varying message sizes to ensure proper claim check processing and payload retrieval.
*   **State Management:** Services maintain their own state. Validate that event processing correctly updates local states and that these states remain consistent across the distributed system over time.

### 3. Key Areas

| Area                        | Description                                                                                                                                                                                                            | Relevant Files/Patterns                                                                                                                                    |
| :-------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Planning & ACs**     | Deconstruct ACs into sequences of API calls, event publications, and expected downstream effects. Identify which services are involved in each step.                                                                    | `AccountService/Program.cs` (API), `AccountService/Worker.cs` (Event Consumption), `Shared/` (Event Contracts), `OrderMaker/Models/` (Order-specific events) |
| **Exploratory Testing**     | Generate diverse inputs via `TestPublisher/Program.cs` or direct API calls. Observe the entire event chain: message broker, worker service processing, database updates, and subsequent API query results.                  | `TestPublisher/Program.cs`, `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor/`, `*/Worker.cs` files                                                |
| **Edge Cases & Boundaries** | Test invalid event schemas, extremely large event payloads (ClaimCheckProcessor), rapid-fire event publications (concurrency/throttling), and sequences of events that might lead to race conditions or incorrect state. | `Shared/`, `OrderMaker/Models/` (schema variations), `ClaimCheckProcessor/` (payload size), `OutboxProcessor/` (concurrency)                                   |
| **Risk-Based Priorities**   | Prioritize testing of core business flows (e.g., `Order Creation and Fulfillment`) and critical infrastructure components (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`).                               | `OrchestratorService/`, `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/`                                                                     |
| **Data Validation**         | Verify that data transforms correctly between services via events. Check database integrity after event processing. Test for data loss or corruption during transit.                                                   | All `*/Worker.cs` (processing logic), `Shared/`, `OrderMaker/Models/` (data contracts), any database interaction points.                                      |
| **Error Scenarios**         | Test how services handle malformed events, missing required data, duplicate events, or failures in downstream systems. Validate error logging and potential dead-letter queueing.                                        | All `*/Worker.cs` (error handling logic), `Shared/` (contract violations)                                                                                   |

### 4. Safe First Changes

These actions are low-risk and excellent for familiarizing yourself with the system's behavior.

*   **Basic Event Publishing:** Using `TestPublisher/Program.cs`, publish a simple, valid event (e.g., an Account Created event or a small order event) and observe the logs of relevant services (`AccountService/Worker.cs`, `OrchestratorService/Worker.cs`).
*   **API Sanity Check:** Make basic GET requests to endpoints exposed by `AccountService/Program.cs`, `InventoryService/Program.cs`, and `OrchestratorService/Program.cs` to confirm they are reachable and return expected default responses.
*   **Schema Exploration:** Analyze the models in `Shared/` and `OrderMaker/Models/` to build an internal knowledge graph of all event and DTO schemas without altering any code.
*   **Configuration Review:** Inspect `appsettings.json` or `Program.cs` files in each service to understand how message broker connections, database connections, and other external dependencies are configured.

### 5. Danger Zones

Exercise extreme caution when interacting with these areas. Changes or incorrect testing here can lead to data loss, system instability, or cascading failures.

*   **Worker Services (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`):**
    *   **Risk:** Any misconfiguration or bug here can halt event flow, lead to data inconsistencies, message loss, or performance degradation across the entire system.
    *   **Caution:** Avoid directly modifying their code unless absolutely necessary for specific test harnesses. Focus on observing their behavior through their interactions with the message broker and databases.
*   **Message Broker and Databases:**
    *   **Risk:** These are external dependencies. Incorrect test data injection or deletion can corrupt system state.
    *   **Caution:** Always use dedicated test environments. Never directly manipulate production databases or message queues during testing.
*   **Event Schema Changes (`Shared/`, `OrderMaker/Models/`):**
    *   **Risk:** Modifying an event contract can break backward compatibility for all services consuming that event, leading to widespread deserialization errors or incorrect business logic.
    *   **Caution:** Any proposed schema change requires extensive regression testing across all dependent services.
*   **High-Volume/Concurrency Testing:**
    *   **Risk:** Overwhelming the system with too many events or API calls in a non-controlled environment can lead to resource exhaustion, deadlocks, or unintended state corruption.
    *   **Caution:** Implement rate limiting or carefully scale up test loads to prevent destabilizing the test environment.

### 6. Checklists

#### 6.1. AC Validation Checklist

*   **Identify Originator:** Which service/API initiates the action described in the AC? (e.g., `OrchestratorService/Program.cs` for Order Creation).
*   **Trace Event Flow:**
    *   Is an event published? If so, which one (from `Shared/` or `OrderMaker/Models/`)?
    *   Is `OutboxProcessor` successfully publishing the event from the initiating service's outbox?
    *   Which worker services (`*/Worker.cs`) consume this event?
    *   Are all intermediate event transformations correctly applied?
*   **Verify Downstream Effects:**
    *   Are expected database changes observed in consuming services?
    *   Are subsequent API queries reflecting the new state?
    *   Are other services reacting as expected via their own events or logic?
*   **Check Error Paths:** What happens if an input is invalid? Is an error event published, or is the original event re-tried/dead-lettered?

#### 6.2. Event Flow Testing Checklist

*   **Schema Compliance:** Does the published event strictly adhere to its defined schema in `Shared/` or `OrderMaker/Models/`?
*   **Payload Integrity:** Is the event payload preserved correctly across transit, especially for large messages processed by `ClaimCheckProcessor`?
*   **Event Ordering:** For sequential events, is the order maintained and processed correctly by consumers?
*   **Idempotency:** Can an event be processed multiple times without causing incorrect state changes? (e.g., if a message broker re-delivers)
*   **Error Handling:**
    *   If a consuming service fails, is the event retried?
    *   Does it eventually go to a dead-letter queue (DLQ)?
    *   Is appropriate logging generated for failures?

#### 6.3. Regression Risk Assessment Checklist

*   **Impacted Services:** Identify all services that consume or produce events related to a proposed change.
*   **Event Contract Changes:** If any event schema (`Shared/`, `OrderMaker/Models/`) is altered, are all consumers updated and tested for compatibility?
*   **Worker Service Logic:** Has any logic in `OutboxProcessor`, `ClaimCheckProcessor`, or `MessageReplicator` been changed? If so, prioritize extensive integration testing.
*   **Concurrency Implications:** Does the change introduce new race conditions or affect existing concurrency handling, especially around `OutboxProcessor`?
*   **External Dependencies:** Does the change affect how services interact with the message broker or databases?