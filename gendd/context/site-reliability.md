# Site Reliability Context Document

## Relevance to This Codebase

Site Reliability Engineering is paramount for this event-driven microservices architecture due to its inherent complexity and distributed nature. The system relies heavily on asynchronous communication via events, processed by dedicated worker services, making robust observability, error handling, and resilience strategies critical for maintaining service availability and data consistency. The `OutboxProcessor` is a core component ensuring reliable event publishing, making its operational health directly tied to the overall system's integrity. Without strong SRE practices, issues like message loss, delayed processing, service outages, or data inconsistencies could be difficult to detect, diagnose, and resolve, directly impacting business operations.

## Current State Assessment

| Artifact/Area            | Location/Pattern                                 | Maturity Level | Notes                                                                                                                                                                                                                                |
| :----------------------- | :----------------------------------------------- | :------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Observability - Logging** | `*Service/Program.cs` (for APIs), `*Service/Worker.cs` (for workers) | MEDIUM         | Standard C#/.NET structured logging is likely used through `Microsoft.Extensions.Logging` (inferred). Specific configuration for log sinks (e.g., Serilog, NLog) or correlation IDs (OpenTelemetry, custom) is not explicitly defined in the provided context, suggesting potential gaps in advanced correlation or centralized log management.                                                                                                        |
| **Observability - Metrics** | (No explicit configuration detected)             | LOW            | No direct evidence of Prometheus, Datadog, or custom metrics libraries/instrumentation. This is a significant gap, indicating a lack of system-level performance, utilization, or error rate visibility, beyond basic application logs. |
| **Observability - Tracing** | (No explicit configuration detected)             | LOW            | No direct evidence of distributed tracing instrumentation (e.g., OpenTelemetry, Jaeger). Without this, correlating operations across multiple services in an event-driven flow will be highly challenging, impacting incident MTTR. |
| **Health Checks**        | `*Service/Program.cs` (for API services)         | MEDIUM (inferred)      | ASP.NET Core applications typically include health check endpoints (e.g., `/health`, `/ready`) configured via `UseHealthChecks` or similar extensions. These would indicate service liveness and readiness, but specific dependencies checked are unknown. |
| **Alerting**             | (No explicit configuration detected)             | LOW            | No evidence of alert rule definitions (e.g., Prometheus Alertmanager, cloud provider alerts) or integration with notification channels. Likely relies on basic error logging or manual observation.                                  |
| **SLO/SLI Definitions**  | (None detected)                                  | LOW            | No explicit Service Level Indicators or Objectives are defined, meaning there is no clear quantitative measure of service health, reliability targets, or error budget tracking.                                                |
| **Reliability Patterns** | `OutboxProcessor/OrderOutboxWorker.cs`           | MEDIUM         | **Outbox Pattern:** Explicitly implemented to ensure transactional reliability for event publishing. <br/> **Retry/Circuit Breakers/Timeouts:** No explicit evidence of these patterns (e.g., Polly, Resilience4j) for external calls or inter-service communication. (inferred missing) |
| **Incident Response**    | (None detected)                                  | LOW            | No runbook documentation, incident response processes, or post-mortem templates are indicated. Incident resolution likely ad-hoc.                                                                                                   |
| **Capacity & Performance** | (None detected)                                  | LOW            | No specific configuration for auto-scaling (e.g., HPA manifests) or performance benchmarks/load tests are evident. Headroom and scaling readiness are unknown.                                                                    |

## What's Missing or Weak

| Gap                              | Impact                                                                                                 | Priority |
| :------------------------------- | :----------------------------------------------------------------------------------------------------- | :------- |
| **Comprehensive Metrics**        | Lack of real-time visibility into service performance, resource utilization, and error rates. Makes proactive detection and rapid diagnosis difficult. | HIGH     |
| **Distributed Tracing**          | Inability to follow a single request/event's flow across multiple services, leading to prolonged debugging of distributed issues. | HIGH     |
| **SLO/SLI Definitions**          | No clear definition of acceptable service performance or reliability, hindering objective assessment of service health and business impact. | HIGH     |
| **Alerting Configuration**       | Reliance on reactive observation or manual log checks. Delays detection of critical issues, increasing MTTR. | HIGH     |
| **Runbooks & Incident Processes** | Chaotic incident response, inconsistent troubleshooting, and longer recovery times due to lack of documented procedures. | HIGH     |
| **Resilience Patterns (Retry, CB, Timeout)** | Services are vulnerable to transient failures or slow/unresponsive dependencies, potentially leading to cascading failures. | MEDIUM   |
| **Log Correlation IDs**          | Without a consistent mechanism to inject and propagate correlation IDs, tracing related log messages across services is difficult. | MEDIUM   |
| **PII Handling in Logs**         | No explicit mention of PII masking or filtering, posing a compliance and security risk.                   | MEDIUM   |

## Key Patterns and Conventions

| Pattern Name                    | Where Used                                                       | Notes                                                                                                                                       |
| :------------------------------ | :--------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------ |
| **Event-Driven Architecture**   | All services (`AccountService`, `InventoryService`, `OrchestratorService`, `OutboxProcessor`, etc.) | Services communicate asynchronously via events. This pattern is central to the system's design and scalability.                             |
| **Transactional Outbox Pattern**| `OutboxProcessor/OrderOutboxWorker.cs`                           | Ensures atomic commit of database changes and outgoing event messages, critical for data consistency and reliable event publishing.           |
| **.NET Worker Services**        | `*Service/Worker.cs`, `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor`, `MessageReplicator` | Standard pattern for long-running background tasks, consuming messages/events, and performing asynchronous operations.                      |
| **ASP.NET Core Web APIs**       | `*Service/Program.cs` (e.g., `AccountService`, `InventoryService`, `OrchestratorService`, `OrderMaker`) | Entry points for web-facing services, likely exposing HTTP endpoints for business logic.                                                   |
| **Shared Data Contracts**       | `Shared/` directory                                              | Common event contracts, DTOs, and utility classes are centralized to ensure consistency across services. Changes here have system-wide impact. |

## Risks and Hotspots

| Risk                                   | Location                                     | Severity | Mitigation                                                                                                                                 |
| :------------------------------------- | :------------------------------------------- | :------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| **`OutboxProcessor` Failure**          | `OutboxProcessor/OrderOutboxWorker.cs`       | HIGH     | As a single point of failure for reliable event publishing, failure here can halt all asynchronous operations. Requires dedicated, comprehensive monitoring and alerting. Consider redundancy. |
| **Lack of Observability**              | Entire system                                | HIGH     | Without metrics, tracing, and improved logging, diagnosing system-wide issues (e.g., deadlocks, processing backlogs, cascading failures) is extremely difficult. |
| **Event Contract Breaking Changes**    | `Shared/` directory, Inter-service communication | MEDIUM   | Changes to event schemas in `Shared/` can silently break downstream consumers, leading to data corruption or runtime errors in an event-driven system. Requires strict versioning and compatibility testing. |
| **Resource Exhaustion in Workers**     | `*Service/Worker.cs`                         | MEDIUM   | Workers processing events continuously might experience memory leaks or CPU spikes, leading to service degradation or crashes. Requires aggressive resource monitoring and limits. |
| **Cascading Failures (Lack of Resilience)** | All services, especially inter-service calls | MEDIUM   | Without retry, circuit breaker, or timeout patterns, a single slow or failing dependency can propagate issues across the entire system. Implement resilience libraries (e.g., Polly). |
| **PII Exposure in Logs**               | All services (logging)                       | HIGH (if PII present) | Unmasked sensitive data in logs can lead to data breaches and compliance violations. Implement explicit PII scrubbing/masking at logging points. |
| **Manual Deployments/No IaC/CI/CD**    | Overall system (inferred from lack of mention) | HIGH     | Inconsistent environments, human error, and slow recovery times. Directly impacts reliability and agility. Implement CI/CD and Infrastructure as Code. |

## AI Agent Guidelines

### DO
*   **DO** always check `OutboxProcessor/OrderOutboxWorker.cs` and its related dependencies when investigating issues related to event publishing reliability or message backlogs.
*   **DO** verify `*Service/Program.cs` for health check configurations before assessing service liveness or readiness for deployment.
*   **DO** investigate `*Service/Worker.cs` implementations for error handling, idempotency patterns, and potential external dependencies when diagnosing processing failures.
*   **DO** review the `Shared/` directory for event contract definitions when understanding inter-service communication or proposing changes to event payloads.
*   **DO** look for `Microsoft.Extensions.Logging` usage patterns in `Program.cs` or `Worker.cs` to understand current logging levels and configured providers.

### DON'T
*   **DON'T** assume comprehensive observability (metrics, tracing) is in place; explicitly look for configuration files for these tools.
*   **DON'T** modify event contracts in `Shared/` without a thorough impact analysis on all consuming services and a clear versioning strategy.
*   **DON'T** introduce blocking operations or long-running synchronous calls within `Worker.cs` services without considering their impact on throughput and message processing backlogs.
*   **DON'T** rely solely on application logs for performance monitoring; metrics are essential for system-wide performance insights.

### ALWAYS CHECK
*   **ALWAYS CHECK** the error handling and retry logic within `*Service/Worker.cs` for robustness, especially for operations involving external systems or database writes.
*   **ALWAYS CHECK** for idempotency in worker processing logic, as messages can be redelivered in event-driven systems.
*   **ALWAYS CHECK** the configuration of any message broker (if accessible) when debugging event delivery or consumption issues related to `OutboxProcessor` or any worker service.
*   **ALWAYS CHECK** for potential resource contention or deadlocks when multiple workers are processing messages concurrently against shared resources.

## Related Context Areas

*   [devops-infrastructure](./devops-infrastructure.md): Deployment, CI/CD, container orchestration (e.g., Kubernetes HPA), which directly impacts capacity and auto-scaling.
*   [architecture](./architecture.md): Overall system design, choice of message broker, and microservices decomposition directly influence reliability patterns and observability needs.
*   [security](./security.md): Secure logging practices, PII handling, and authorization for monitoring endpoints.
*   [backend-development](./backend-development.md): Error handling strategies, external API call patterns, and resilience implementation details within individual services.

## Key Files

| Path or Glob                                       | Purpose                                                                               | Notes                                                                                             |
| :------------------------------------------------- | :------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------ |
| `*Service/Program.cs`                              | Service entry point, ASP.NET Core setup, DI container, middleware configuration, potential health checks. | Critical for understanding how API services are initialized and their high-level reliability hooks. |
| `*Service/Worker.cs` (e.g., `AccountService/Worker.cs`, `InventoryService/Worker.cs`) | Background worker service implementation for event consumption and processing.        | Key area for error handling, message processing logic, and potential bottlenecks.   |
| `OutboxProcessor/OrderOutboxWorker.cs`             | Core component for the transactional outbox pattern, ensuring reliable event publishing. | **HIGH RELIABILITY CRITICALITY:** Any issues here impact entire event flow.       |
| `Shared/**/*.cs`                                   | Contains shared data contracts (events, DTOs) and utilities.                          | Changes to these files can have system-wide reliability and compatibility impacts.  |
| `**/Dockerfile`                                    | Defines container images for services.                                                | Indicates how services are packaged and deployed, relevant for resource management. |
| `**/appsettings.json`, `**/appsettings.*.json` | Configuration settings, including logging levels, connection strings, and potentially external service endpoints. | Important for runtime behavior and tuning reliability aspects.                     |