# Site Reliability Context Document

## Relevance to This Codebase

Site Reliability Engineering (SRE) is paramount for this event-driven microservices platform due to its heavy reliance on asynchronous event processing and the criticality of its worker services. The system's core functionality, such as reliable event delivery via the outbox pattern, handling large messages through claim checks, and inter-broker message replication, directly depends on the continuous availability, performance, and correct functioning of services like `OutboxProcessor`, `ClaimCheckProcessor`, and `MessageReplicator`. Any failure or degraded performance in these components can lead to data inconsistencies, processing backlogs, and ultimately, system-wide failures. Robust observability, defined Service Level Objectives (SLOs), effective alerting, and implemented reliability patterns are essential to maintain the integrity and responsiveness of the event flow and overall system health. The identified risk hotspot regarding these critical worker services underscores the need for a strong SRE focus.

## Current State Assessment

### Observability -- Logging
*   **Format**: Likely structured JSON logging through `Microsoft.Extensions.Logging` (inferred). Standard .NET applications typically configure this in `Program.cs` for API services and within `Worker.cs` for background services.
*   **Level Usage**: Consistent use of `ILogger` for informational and error logging is expected in `Program.cs` and `Worker.cs` files across all services (e.g., `AccountService/Program.cs`, `OutboxProcessor/OrderOutboxWorker.cs`).
*   **Correlation ID**: No explicit correlation ID implementation (e.g., via HTTP headers or message properties) or PII handling configurations identified. (inferred)
*   **Files**: `**/Program.cs`, `**/Worker.cs`

### Observability -- Metrics
*   **Coverage**: No explicit monitoring configuration files (e.g., Prometheus, Datadog agents) or dedicated `metrics/` directories were identified. It is inferred that dedicated, custom metrics instrumentation for RED/USE or business metrics is not extensively implemented. (inferred)
*   **Golden Signals**: Basic system-level metrics might be collected by infrastructure, but application-level metrics for request rate, error rate, and duration are likely limited or absent. (inferred)

### Observability -- Tracing
*   **Instrumentation**: No explicit distributed tracing instrumentation (e.g., OpenTelemetry, Jaeger) or related SDK imports were identified. (inferred)
*   **Coverage**: It is inferred that cross-service trace correlation is not implemented, making end-to-end request tracing challenging. (inferred)

### SLO Analysis
*   **Definitions**: No explicit Service Level Indicators (SLIs) or Service Level Objectives (SLOs) are defined or tracked within the codebase or project documentation. (inferred)
*   **Error Budget**: Consequently, no error budget tracking mechanism is present. (inferred)

### Alerting Assessment
*   **Rules**: No explicit alerting rule definitions (e.g., Prometheus alert rules, Grafana alerts) or notification channel integrations were identified within the codebase. (inferred)
*   **Actionability**: It is inferred that alerts are not formally structured or tracked, potentially leading to reactive troubleshooting. (inferred)

### Incident Response
*   **Documentation**: No dedicated `runbooks/` or incident response process documentation was identified. (inferred)
*   **PIRs**: No post-incident review (PIR) templates or incident communication channels were identified. (inferred)

### Reliability Patterns
*   **Health Checks**: ASP.NET Core health checks are likely implemented in `Program.cs` for API services (e.g., `AccountService/Program.cs`, `InventoryService/Program.cs`, `OrchestratorService/Program.cs`) to signal service readiness and liveness.
*   **Retries/Timeouts**: Basic retry mechanisms and timeouts for external calls (e.g., to databases, message brokers, external APIs) are likely implemented within critical worker services (e.g., `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor/Models/`, `MessageReplicator/`) via standard .NET resilience patterns or libraries, though no explicit resilience library (like Polly) was identified. (inferred)
*   **Circuit Breakers**: No explicit circuit breaker implementations were identified. (inferred)
*   **Outbox Pattern**: Reliably implemented in `OutboxProcessor` (specifically `OutboxProcessor/OrderOutboxWorker.cs`) to ensure transactional event publishing.
*   **Claim Check Pattern**: Reliably implemented in `ClaimCheckProcessor` (models in `ClaimCheckProcessor/Models/`) for handling large message payloads.

### Capacity and Performance
*   **Resource Utilization**: No application-specific configuration for monitoring CPU, memory, or connection pool sizing was identified. (inferred)
*   **Auto-scaling**: No explicit auto-scaling configuration (e.g., HPA manifests) or performance benchmarks were identified. (inferred)

## What's Missing or Weak

| Gap | Impact | Priority |
|---|---|---|
| **Comprehensive Observability (Metrics & Tracing)** | Slow incident detection, inability to debug complex distributed issues, lack of performance insights. | HIGH |
| **Defined SLIs/SLOs and Error Budget** | No clear targets for reliability, difficulty in prioritizing reliability work, inability to measure service health objectively. | HIGH |
| **Formal Alerting Strategy** | Reactive incident response, potential for missed critical issues or alert fatigue from poorly configured alerts. | HIGH |
| **Incident Response Documentation (Runbooks, PIRs)** | Increased MTTR, inconsistent incident handling, loss of institutional knowledge. | MEDIUM |
| **Distributed Correlation ID Implementation** | Inability to trace a single request/event flow across multiple services and log files. | MEDIUM |
| **Explicit Resilience Libraries (e.g., Polly for Circuit Breakers)** | Increased risk of cascading failures, especially for external dependencies of critical worker services. | MEDIUM |
| **Capacity Planning & Auto-scaling Configuration** | Risk of performance degradation and outages under load, inefficient resource utilization. | MEDIUM |
| **PII Handling in Logs** | Security and compliance risks if sensitive data is logged without masking. | LOW |

## Key Patterns and Conventions

| Pattern | Where Used | Notes |
|---|---|---|
| **ASP.NET Core Health Checks** | `AccountService/Program.cs`, `InventoryService/Program.cs`, `OrchestratorService/Program.cs` | Standard for signaling service readiness and liveness; likely configured in `Program.cs` for API services. |
| **Standard .NET Logging** | All `Program.cs` and `Worker.cs` files (e.g., `AccountService/Worker.cs`, `OutboxProcessor/OrderOutboxWorker.cs`) | Utilizes `Microsoft.Extensions.Logging` for structured logging (inferred), configured during host builder setup. |
| **Outbox Pattern** | `OutboxProcessor/OrderOutboxWorker.cs`, `OutboxProcessor/Models/` | Ensures atomic transaction between local database changes and event publishing to the message broker. |
| **Claim Check Pattern** | `ClaimCheckProcessor/Models/` | Handles large message payloads by storing them externally and passing references, configured for efficient message processing. |
| **Background Worker Processing** | `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor/Models/`, `MessageReplicator/` | Services inherit from `BackgroundService` or implement `IHostedService` for long-running, event-driven tasks. |
| **Basic Retries/Timeouts** | (Inferred in worker services) `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor/Models/` | Likely used when interacting with external dependencies like databases or message brokers to handle transient failures. |

## Risks and Hotspots

| Risk | Location | Severity | Mitigation |
|---|---|---|---|
| **Critical Worker Service Failure/Degradation** | `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/` | HIGH | Implement comprehensive metrics (backlogs, processing rates, errors) and proactive alerts. Introduce circuit breakers for external calls. Enhance logging with correlation IDs. Define clear SLOs for event processing latency and throughput. |
| **Lack of Observability** | Across all services (no dedicated metrics/tracing config) | HIGH | Introduce OpenTelemetry for metrics and tracing. Instrument key operations (event publishing/consumption, external calls). Configure a centralized logging solution. |
| **Absence of Defined SLOs/SLIs** | Project-wide | HIGH | Define availability, latency, and throughput SLOs for critical business flows and individual services, especially worker services. Track these through dashboards and integrate with alerting. |
| **Undeclared Resilience for Dependencies** | All services interacting with message broker, databases | MEDIUM | Explicitly implement resilience patterns (retry with exponential backoff, timeouts, circuit breakers via libraries like Polly) for all external calls. |
| **No Formal Incident Response** | Project-wide | MEDIUM | Develop runbooks for common failures. Establish an on-call rotation and define clear escalation paths. Implement a post-incident review (PIR) process. |
| **Uncontrolled `TestPublisher` usage** | `TestPublisher/Program.cs` | LOW | Ensure `TestPublisher` is not deployed to production environments or is strictly access-controlled and audited if present in lower environments. |

## AI Agent Guidelines

### Do
*   **Prioritize reliability improvements for worker services**: Always check `OutboxProcessor/`, `ClaimCheckProcessor/`, and `MessageReplicator/` for potential SRE enhancements (monitoring, error handling, resilience).
*   **Look for existing health checks**: Review `Program.cs` files in API services for `/health` or `/ready` endpoints and their underlying logic.
*   **Examine logging configurations**: Check `Program.cs` and `Worker.cs` files for `Microsoft.Extensions.Logging` setup and ensure consistent log levels and structured output.
*   **Identify external calls**: When modifying or adding new functionality, especially in worker services, identify external service dependencies (databases, message brokers, APIs) and suggest basic resilience patterns (retry, timeout).
*   **Consult `*.csproj` files**: Look for references to resilience libraries (e.g., Polly) or monitoring agents, although none were explicitly identified.

### Don't
*   **Assume comprehensive observability**: Do not assume that metrics, tracing, or robust alerting are already in place for detailed performance analysis or debugging.
*   **Modify worker service logic without considering event flow impacts**: Changes to `OutboxProcessor/`, `ClaimCheckProcessor/`, or `MessageReplicator/` can severely disrupt the entire event-driven system.
*   **Assume SLOs or error budgets exist**: Do not make decisions based on predefined service level targets without explicit documentation.
*   **Bypass logging**: Ensure all new functionality logs relevant information for debugging and operational insights, using appropriate log levels.

### Always Check
*   **Worker service logs**: Before any significant changes or troubleshooting, always review the logs of `OutboxProcessor`, `ClaimCheckProcessor`, and `MessageReplicator` for errors, backlogs, or performance warnings.
*   **External dependency health**: When investigating issues, always consider the health and performance of the message broker and databases as critical dependencies.
*   **Resilience configurations**: Before modifying code that makes external calls, verify any existing retry or timeout mechanisms and their thresholds.
*   **Impact on event atomicity**: For changes affecting `OutboxProcessor`, always ensure that the transactional outbox pattern remains intact and atomic.

## Related Context Areas
*   **[devops-infrastructure](./devops-infrastructure.md)**: Deployment strategies, CI/CD pipelines, container orchestration, and infrastructure monitoring directly impact the system's reliability and operational capabilities.
*   **[architecture](./architecture.md)**: The overall event-driven architecture, service boundaries, and communication patterns dictate many of the reliability challenges and solutions.
*   **[backend-development](./backend-development.md)**: Error handling, exception management, thread safety in worker processes, and correct implementation of external calls contribute directly to service reliability.

## Key Files

| Path or Glob | Purpose | Notes |
|---|---|---|
| `**/Program.cs` | Entry point for API services, hosting configuration, health checks, logging setup. | Primary location for service-level configuration for API services. |
| `**/Worker.cs` | Background service implementation, event consumption, business logic for workers, logging. | Crucial for worker services like `AccountService/Worker.cs`, `InventoryService/Worker.cs`. |
| `OutboxProcessor/OrderOutboxWorker.cs` | Core implementation of the transactional outbox pattern. | **CRITICAL**: Ensures reliable event publishing; directly impacts data consistency. |
| `ClaimCheckProcessor/Models/` | Defines models and logic for handling large message payloads via the claim check pattern. | **CRITICAL**: Essential for processing large events efficiently and reliably. |
| `MessageReplicator/`