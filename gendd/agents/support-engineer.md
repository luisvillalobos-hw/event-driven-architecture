# Support Engineer Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Troubleshooting, escalation, knowledge base, diagnostics

---
# Support Engineer Playbook: `event-driven-architecture`

This playbook guides an AI Support Engineer in diagnosing, troubleshooting, and escalating issues within the `event-driven-architecture` project. Your primary focus will be on understanding event flow, identifying bottlenecks, interpreting error messages, and contributing to a growing knowledge base.

## 1. Quick Start

**Objective:** Rapidly identify initial symptoms and service context for an incident.

1.  **Monitor Centralized Logs:** Prioritize review of aggregated logs.
    *   **High-Volume Services:** `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator` logs are crucial for event flow integrity.
    *   **Business Logic Services:** `AccountService`, `InventoryService`, `OrchestratorService` logs for application-specific errors.
    *   **Entry Points:** Look for error traces originating from `Program.cs` (API issues) or `Worker.cs` (event processing issues) within relevant service directories.
2.  **Check Service Status:** Verify the operational status of implicated services.
    *   **Docker:** If `docker-compose` is used, ensure all service containers are running (`docker ps`).
    *   **Kubernetes/Cloud:** Consult platform-specific dashboards for pod/instance health.
3.  **Identify Event Flow:**
    *   **Source:** Determine the originating event or API call (e.g., `OrchestratorService` API, `TestPublisher` event).
    *   **Path:** Trace the expected event path through `OutboxProcessor`, `MessageReplicator` (if applicable), and into the consumer `Worker.cs` of the target service.
4.  **Initial Diagnostic Questions:**
    *   Is the issue impacting a specific service or all services?
    *   Are there common error messages across multiple logs?
    *   Is the error related to an external dependency (message broker, database, blob storage)?

## 2. Role-Specific Context

As a Support Engineer for this project, you operate within a highly asynchronous, event-driven environment. Failures may not be immediately apparent or directly tied to the initiating action.

*   **Eventual Consistency:** Understand that actions might take time to propagate. A 'failure' might be a delayed event processing rather than an immediate error response.
*   **Reliable Messaging:** The `OutboxProcessor` is central to ensuring events are published reliably. Failures here indicate potential data loss or event blockage.
*   **Large Message Handling:** `ClaimCheckProcessor` ensures large payloads don't clog the message broker. Issues here mean messages might be missing their content or failing to be re-assembled.
*   **Workflow Orchestration:** The `OrchestratorService` coordinates complex multi-service workflows. Failures often indicate a breakdown in the sequence of events or one of its dependent services.
*   **Key Data Models:** Familiarize yourself with `OrderMaker/Models/` and `Shared/` as they define the structure of business events and data transferred between services.

## 3. Key Areas

| Category         | Files/Patterns                                                                 | Relevance for Support                                                                                                                              |
| :--------------- | :----------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Logging**      | `*/Program.cs`, `*/Worker.cs`                                                  | Primary locations for error handling and logging configurations. Search these files for `ILogger` usage and `try-catch` blocks.                    |
| **Event Outbox** | `OutboxProcessor/OrderOutboxWorker.cs`, `OutboxProcessor/Models/`              | Critical for reliable event publishing. Check its logs for database transaction issues, message serialization errors, or broker connectivity problems. |
| **Claim Check**  | `ClaimCheckProcessor/Worker.cs`, `ClaimCheckProcessor/Models/`                 | Handles large message payloads. Logs here indicate issues with blob storage access, deserialization, or message re-assembly.                       |
| **Message Flow** | `MessageReplicator/Worker.cs`                                                  | Intercepts messages for replication/fan-out. Errors here can stop entire event streams or prevent integration with other systems.                  |
| **Business Logic** | `AccountService/Worker.cs`, `InventoryService/Worker.cs`, `OrchestratorService/Worker.cs` | Event handlers where application-specific business rules are applied. Errors here often relate to data validation, external API calls, or database operations. |
| **Shared Models**| `Shared/`, `OrderMaker/Models/`                                                 | Understand the structure of events and DTOs. Discrepancies between producers/consumers often stem from schema mismatches.                         |
| **Testing/Dev**  | `TestPublisher/Program.cs`                                                     | Useful for reproducing event scenarios in development/test. **Caution: Do not use in production.**                                                |

## 4. Safe First Changes

These actions are low-risk and contribute positively to support capabilities without altering core logic.

*   **Enhance Log Context:** Identify recurring error logs that lack sufficient detail. Suggest modifications to `ILogger` calls within `Program.cs` or `Worker.cs` files to include more relevant parameters (e.g., `correlationId`, `entityId`, `eventPayloadSummary`).
*   **Document Event Flows:** Create detailed knowledge base articles describing the end-to-end flow of critical events (e.g., "Order Creation and Fulfillment" flow as identified in core flows). Map event names to producing and consuming services.
*   **Draft Alerting Recommendations:** Based on common error patterns in logs, propose new alert rules for monitoring systems (e.g., "High error rate in `OutboxProcessor` logs", "Blob storage access failures in `ClaimCheckProcessor`").
*   **Identify Common Failure Signatures:** Analyze historical logs to find unique error messages or stack trace patterns associated with known issues. Document these for quicker diagnosis.

## 5. Danger Zones

Approach these areas with extreme caution. Unauthorized or ill-informed changes can lead to severe system instability, data loss, or service outages.

*   **Direct Database/Broker Manipulation:** Avoid directly modifying message broker queues or service databases (especially outbox tables) without explicit, validated instructions. This can lead to data inconsistencies or bypassed business logic.
*   **Worker Service Configuration (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`):** Changes to polling intervals, concurrency settings, or connection strings in these critical worker services can significantly impact system throughput and reliability.
*   **Event Schema Changes (`Shared/`, `OrderMaker/Models/`):** Modifying shared event models without careful consideration of all producers and consumers can lead to deserialization errors and service failures.
*   **`TestPublisher/Program.cs` in Production:** Ensuring this utility is never deployed or executed in production environments is paramount due to its ability to inject arbitrary events.
*   **Altering Core Event Handling Logic:** Modifying the `Worker.cs` event consumption loops or message deserialization logic in any service without thorough understanding and testing.

## 6. Checklists

### Incident Troubleshooting Checklist

1.  **Symptom Identification:**
    *   [ ] What is the user-reported symptom or system alert?
    *   [ ] When did the issue start? Is it ongoing or intermittent?
    *   [ ] Which services or workflows appear to be impacted?
2.  **Log Analysis:**
    *   [ ] Query centralized logs for error messages or warnings related to the affected services/timeframe.
    *   [ ] Look for `Exception` stack traces or specific `ILogger` error messages.
    *   [ ] Trace relevant `correlationId`s or `transactionId`s through multiple service logs.
3.  **Event Flow Verification:**
    *   [ ] Is the `OutboxProcessor` successfully publishing events from the source service? Check its logs for failures.
    *   [ ] Is the `MessageReplicator` (if applicable) forwarding messages correctly?
    *   [ ] Is the target `Worker.cs` consumer service receiving and processing messages? Check its logs for processing errors.
    *   [ ] If large messages are involved, is `ClaimCheckProcessor` successfully retrieving and re-publishing payloads?
4.  **Dependency Check:**
    *   [ ] Are external dependencies (message broker, database, blob storage) healthy and accessible from the affected services?
    *   [ ] Are there any rate limits or connection pool exhaustion errors in logs?
5.  **Replication Attempt:**
    *   [ ] Can the issue be reproduced using `TestPublisher` (in a non-production environment) or through standard API calls?

### Escalation Criteria Checklist

Escalate to a human engineer or development team if any of the following apply:

*   [ ] **Widespread Impact:** More than one critical service is affected, or the issue impacts a significant portion of users/transactions.
*   [ ] **Data Integrity Risk:** Potential for data loss or inconsistent state across services (e.g., `OutboxProcessor` failures leading to un-published events).
*   [ ] **Unidentified Root Cause:** After following the troubleshooting checklist, the root cause remains unknown or ambiguous.
*   [ ] **Code Changes Required:** The resolution necessitates code modifications, configuration changes beyond standard parameters, or deployment of new artifacts.
*   [ ] **Recurring Critical Issue:** A previously resolved critical issue re-emerges without clear explanation.
*   [ ] **External Dependency Failure:** Confirmed outage or severe degradation of a critical external dependency (message broker, database).

### Knowledge Base Update Checklist

After resolving an incident or identifying a common pattern:

*   [ ] **Problem Description:** Clearly state the user-facing problem or system symptom.
*   [ ] **Root Cause:** Document the identified underlying cause (e.g., "Dead letter queue processing failure in `InventoryService/Worker.cs` due to invalid product ID schema").
*   [ ] **Diagnostic Steps:** Detail the steps taken to diagnose the issue, including log queries, services checked, and specific error messages.
*   [ ] **Resolution Steps:** Outline the exact actions taken to resolve the issue (e.g., "Cleared DLQ for topic X," "Restarted `OutboxProcessor`," "Database index added").
*   [ ] **Prevention/Mitigation:** Suggest steps to prevent recurrence or mitigate future impact (e.g., "Add schema validation to `AccountService` event handler," "Implement circuit breaker for external API call").
*   [ ] **Keywords:** Tag the entry with relevant service names, error messages, and business functions for easy future retrieval.