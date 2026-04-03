# SRE Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Reliability, observability, incident response, SLOs

---
# SRE Playbook for `event-driven-architecture`

This playbook outlines core SRE responsibilities for the `event-driven-architecture` project. Your primary focus will be on reliability, observability, incident response, and defining/monitoring SLOs for this event-driven system.

---

## 1. Quick Start

To effectively operate as an SRE on this project, prioritize understanding the core event flow and observability hooks.

**Initial Code Review Focus:**
1.  **Service Entry Points:** Review `AccountService/Program.cs`, `InventoryService/Program.cs`, `OrchestratorService/Program.cs` to understand:
    *   Service startup and configuration.
    *   Dependency Injection setup (e.g., for database contexts, message broker clients).
    *   Any existing logging, metrics, or tracing configuration.
    *   Health check endpoints (e.g., `app.UseHealthChecks("/health")`).
2.  **Worker Logic:** Examine `AccountService/Worker.cs`, `InventoryService/Worker.cs`, `OrchestratorService/Worker.cs`, `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor/Worker.cs`, `MessageReplicator/Worker.cs` to understand:
    *   How messages are consumed from the broker.
    *   Core business logic and event processing.
    *   Error handling patterns (retries, dead-lettering).
3.  **Shared Observability:** Look for a `Shared/` project or common conventions that define logging contexts, metric instruments, or tracing patterns. If not explicit, assume basic .NET logging and propose enhancements.

**To Run/Observe Locally (Inferred):**
*   **Individual Service Startup:** Navigate to a service directory (e.g., `AccountService/`) and execute `dotnet run`. Repeat for critical worker services to simulate local event flow.
*   **Docker Containerization:** While `docker-compose.yml` is not provided, expect `Dockerfile`s within service directories. You may need to `docker build` and `docker run` services or entire solutions for a more integrated local environment.
*   **Test Event Injection:** Use `TestPublisher/Program.cs` by running `dotnet run` from its directory to send test events, observing their flow through the system via logs.

---

## 2. Role-Specific Context

The `event-driven-architecture` project's event-driven nature presents unique SRE challenges and opportunities.

*   **Reliability is Event Flow:** The health of the system is directly tied to the reliable publishing, consumption, and processing of events. Focus on message broker health, outbox processing, and consumer idempotency.
*   **Worker Services are Critical:** `OutboxProcessor`, `ClaimCheckProcessor`, and `MessageReplicator` are not optional; their continuous and correct operation is paramount for data consistency and event integrity.
*   **Distributed State:** Data consistency often relies on eventual consistency across services, coordinated by events. Ensure robust error handling and monitoring for potential data drift or stalled workflows.
*   **SLOs on Event Latency:** Key Service Level Objectives (SLOs) will likely revolve around end-to-end event processing latency (e.g., from `OutboxProcessor` commit to final service processing), message throughput, and error rates in event consumers.

---

## 3. Key Areas

**Observability:**
*   **Logging:**
    *   **Locations:** `*/Program.cs` (initialization), `*/Worker.cs` (event processing details), `Shared/` (common logging utilities/context enrichment).
    *   **Pattern:** Expect `ILogger<T>` injection. Monitor for structured logging (e.g., Serilog, NLog) setup that includes event IDs, correlation IDs, and relevant business context (e.g., OrderId, AccountId).
    *   **Action:** Ensure all event consumption and publication points in `Worker.cs` files log at appropriate levels (Information, Warning, Error) with sufficient context for tracing.
*   **Metrics:**
    *   **Locations:** `*/Program.cs` (instrumentation setup), `*/Worker.cs` (business logic counters/timers).
    *   **Pattern:** Look for `.NET Core Metrics` API usage, `Prometheus` client libraries, or `Application Insights` integration.
    *   **Action:** Propose metrics for queue depth (message broker), event processing duration, successful/failed event counts per type, and external dependency call latency.
*   **Tracing:**
    *   **Locations:** `*/Program.cs` (OpenTelemetry/Distributed Tracing setup), `Shared/` (context propagation).
    *   **Pattern:** Expect `ActivitySource` or similar to propagate correlation IDs across service boundaries and through the message broker.
    *   **Action:** Verify that trace IDs are consistently propagated through message headers for end-to-end visibility of event flows.

**Error Handling and Resilience Patterns:**
*   **Event Consumers (`*/Worker.cs`):**
    *   **Pattern:** Look for `try-catch` blocks around message processing logic, retry policies (e.g., Polly), and explicit dead-letter queue (DLQ) mechanisms. Ensure idempotency for messages that might be re-processed.
    *   **Risk:** Lack of idempotent processing can lead to duplicate operations (e.g., double debit) on message redelivery.
*   **Outbox Pattern (`OutboxProcessor/OrderOutboxWorker.cs`):**
    *   **Pattern:** Transactional writes to a local outbox table followed by asynchronous publishing to the message broker.
    *   **Risk:** `OutboxProcessor` failure directly halts event publishing and can lead to data inconsistencies. Monitor its health and latency closely.
*   **Claim Check Pattern (`ClaimCheckProcessor/Worker.cs`):**
    *   **Pattern:** Messages containing references to large payloads stored externally. `ClaimCheckProcessor` retrieves the actual payload.
    *   **Risk:** Failure of `ClaimCheckProcessor` or the external storage (e.g., blob storage) will stall message processing for large events.

**Health Checks and Readiness Probes:**
*   **Locations:** `*/Program.cs` (e.g., `app.UseHealthChecks()`).
*   **Pattern:** Expect basic `/health` endpoint and potentially more detailed `/ready` and `/live` endpoints checking critical dependencies (database, message broker, blob storage).
*   **Action:** Ensure all API services expose health checks. Validate worker services have internal health reporting or mechanisms for external monitoring tools to ascertain their health (e.g., metrics for last processed message timestamp).

**SLO/SLI Candidates:**

| SLO/SLI Type         | Description                                                                                                                                                                                                                                           | Target                                        |
| :------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------- |
| **Event Processing Latency** | Time taken from an event being written to a service's outbox until its final processing by all subscribed services. Specifically, `OutboxProcessor` publication time to `Worker.cs` consumption completion time.                            | P99 < 500ms                                   |
| **API Latency**      | Time taken for API services (`AccountService`, `InventoryService`, `OrchestratorService`) to respond to synchronous requests.                                                                                                                         | P99 < 200ms                                   |
| **Event Error Rate** | Percentage of events that fail processing after all retries and are sent to a Dead Letter Queue (DLQ) or lead to unhandled exceptions in `Worker.cs` instances.                                                                                    | < 0.1%                                        |
| **Outbox Processor Throughput** | Number of events successfully published by `OutboxProcessor` per second.                                                                                                                                                                            | > 100 events/sec (adjust based on load)       |
| **Claim Check Success Rate** | Percentage of `ClaimCheckProcessor` operations that successfully retrieve payloads from external storage and re-publish/process the full message.                                                                                                 | > 99.9%                                       |
| **Message Broker Queue Depth** | Monitor the number of messages awaiting processing in critical queues.                                                                                                                                                                                 | < 1000 messages (per critical queue)          |

**Incident Response Readiness:**
*   **Logging:** Ensure logs contain correlation IDs, `Activity.Current?.Id`, and business entity IDs (e.g., OrderId, AccountId) for easy filtering and tracing across services.
*   **Alerting:** Set up alerts on critical metrics (error rates, latency spikes, queue depth) and specific log patterns (e.g., `[ERROR]` messages from `OutboxProcessor`).
*   **Playbooks:** Document procedures for:
    *   Restarting/scaling `OutboxProcessor`, `ClaimCheckProcessor`, and other worker services.
    *   Investigating messages in DLQs.
    *   Debugging stalled event flows using tracing data.

---

## 4. Safe First Changes

These changes offer opportunities to contribute and learn without introducing significant risk to the system.

*   **Enhance Logging Detail:**
    *   **Task:** For a specific `Worker.cs` (e.g., `InventoryService/Worker.cs`), ensure that every incoming event's unique ID and relevant business IDs (e.g., ProductId) are logged at the start of its processing.
    *   **File Example:** `InventoryService/Worker.cs`
    *   **Code Pattern:** `_logger.LogInformation("Processing InventoryUpdateEvent {EventId} for Product {ProductId}", message.EventId, message.ProductId);`
*   **Add Basic Health Checks:**
    *   **Task:** If not present, add a basic `/health` endpoint to an API service that simply returns 200 OK.
    *   **File Example:** `AccountService/Program.cs`
    *   **Code Pattern:** `app.UseHealthChecks("/health");`
*   **Propose a New Metric:**
    *   **Task:** Identify a critical operation within a `Worker.cs` (e.g., database write in `OutboxProcessor`) and propose a simple counter for its success/failure or a timer for its duration.
    *   **File Example:** `OutboxProcessor/OrderOutboxWorker.cs`
    *   **Code Pattern (Conceptual):** Suggest adding `MeterFactory` and `Counter<long>` for "outbox_events_published_total".
*   **Review `appsettings.json`:**
    *   **Task:** Examine `appsettings.json` and `appsettings.Development.json` in each service to ensure environment-specific configurations are clear and non-sensitive data is used for local development.
    *   **Files:** `*/appsettings.json`, `*/appsettings.Development.json`

---

## 5. Danger Zones

Approach these areas with extreme caution. Changes here can have cascading and severe impacts on system reliability and data integrity.

*   **External Dependencies Configuration (High Risk):**
    *   **Files:** Connection strings and client configurations in `*/Program.cs`, `appsettings.json` (for database, message broker, blob storage).
    *   **Risk:** Incorrect configuration can lead to service unavailability, data loss, or performance bottlenecks across the entire event flow.
*   **`OutboxProcessor/OrderOutboxWorker.cs` Logic (High Risk):**
    *   **File:** `OutboxProcessor/OrderOutboxWorker.cs`
    *   **Risk:** Modifications to the outbox pattern's core logic (e.g., how messages are read, marked as sent, or published) can break transactional reliability, leading to missed events or duplicate publications.
*   **Event Schema Changes (Medium/High Risk):**
    *   **Files:** `Shared/` (DTOs), `OrderMaker/Models/`, `ClaimCheckProcessor/Models/`.
    *   **Risk:** Non-backward-compatible changes to event schemas without proper versioning and consumer adaptation will cause deserialization errors and service failures.
*   **Worker Idempotency (Medium Risk):**
    *   **Files:** All `*/Worker.cs` files, especially those performing state-changing operations.
    *   **Risk:** If a worker processes a message multiple times (due to retries or infrastructure issues) and its operations are not idempotent, it can lead to inconsistent or incorrect business state (e.g., multiple inventory decrements for one order).
*   **`TestPublisher/Program.cs` in Production (Medium Risk):**
    *   **File:** `TestPublisher/Program.cs`
    *   **Risk:** If this utility is accidentally deployed or accessible in production, it could allow unauthorized or accidental event injection, potentially corrupting data or triggering unintended workflows. Ensure it's never part of production deployments.

---

## 6. Checklists

### New Service Onboarding Review

*   [ ] **Observability Initialized:**
    *   [ ] Logging configured in `Program.cs` with relevant context enhancers.
    *   [ ] Basic metrics instruments (e.g., request count, error count) set up.
    *   [ ] Distributed tracing enabled, propagating activity IDs.
*   [ ] **Error Handling:**
    *   [ ] Message consumption logic (`Worker.cs`) wrapped in `try-catch` with appropriate logging.
    *   [ ] Retry policies (e.g., Polly) applied to transient errors.
    *   [ ] Dead Letter Queue (DLQ) mechanism configured for unprocessable messages.
    *   [ ] Idempotency strategy for event consumers documented/implemented.
*   [ ]