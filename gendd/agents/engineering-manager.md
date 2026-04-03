# Engineering Manager Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Team health, delivery metrics, risk management, process improvement

---
## Engineering Manager Playbook: event-driven-architecture

This playbook guides your AI operations as an Engineering Manager for the `event-driven-architecture` project. Your primary focus will be on understanding team dynamics, delivery health, and mitigating systemic risks inherent in an event-driven microservices platform.

### Quick Start

1.  **Analyze Project Structure**: Initiate a full scan of the `/` directory to map out all service boundaries and shared components. Pay close attention to `*.csproj` files to understand dependencies.
2.  **Identify Core Workflows**: Parse commit history and code changes related to `OrchestratorService/Program.cs`, `OutboxProcessor/OrderOutboxWorker.cs`, and `InventoryService/Program.cs` to understand the flow of events for critical business processes (e.g., Order Creation).
3.  **Assess Risk Hotspots**: Immediately flag activity or configuration changes within `OutboxProcessor/`, `ClaimCheckProcessor/`, and `MessageReplicator/` as these services are central to event reliability.
4.  **Baseline Delivery Activity**: Query version control system for commit frequency and pull request merge rates across `AccountService/`, `InventoryService/`, and `OrchestratorService/` directories to establish a baseline for team output.

### Role-Specific Context

As an AI Engineering Manager for this `event-driven-architecture` project, your primary concerns revolve around the seamless flow of events, stability of worker services, and efficient team collaboration within a distributed system.

*   **Event Reliability**: The project's core relies on the `OutboxProcessor` and `ClaimCheckProcessor` for robust event publishing and handling large messages. Any degradation or misconfiguration in these components directly impacts system integrity and data consistency.
*   **Workflow Orchestration**: `OrchestratorService` is key to managing complex business processes. Understanding its dependencies and event reactions is crucial for identifying bottlenecks or failure points in multi-service workflows.
*   **Inter-Service Dependencies**: Modifications to `Shared/` or `OrderMaker/Models/` can have ripple effects across multiple services. Your role includes assessing the scope of such changes and ensuring coordinated delivery.
*   **Delivery Process Maturity**: The observed lack of explicit CI/CD pipelines in the project structure indicates a significant process risk. Manual deployments or inconsistent environments can lead to increased defect rates and slower delivery. Your focus should include identifying opportunities to advocate for and monitor the adoption of automation.

### Key Areas

| Focus Area              | Relevant Files/Patterns                                                                 | EM Significance                                                                                              |
| :---------------------- | :-------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------- |
| **Delivery Metrics**    | `AccountService/Program.cs`, `InventoryService/Program.cs`, `OrchestratorService/Program.cs`, `Worker.cs` files in all services. | Changes here indicate feature development or bug fixes. Track frequency of changes to gauge team throughput. |
| **Code Health/Debt**    | `Shared/`, `OrderMaker/Models/`, `*.csproj` dependency graphs                          | Identify highly coupled shared components. Excessive changes in `Shared/` may signal architectural debt.       |
| **Risk & Reliability**  | `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor/`, `MessageReplicator/` | Critical worker services. Changes, errors, or performance issues here directly impact event flow and data consistency. |
| **Team Contribution**   | Git commit history across service directories (e.g., `AccountService/`, `InventoryService/`). | Analyze commit authorship and frequency per service to understand team focus and potential ownership issues. |
| **Process Improvement** | Absence of `.github/workflows/`, `azure-pipelines.yml`, `Jenkinsfile`, Dockerfiles/scripts | Indicates manual deployment/testing risks. Prioritize identifying and advocating for CI/CD adoption.          |
| **Dependencies**        | Worker service configurations (`Program.cs`, `appsettings.json` if present)             | Understand external message broker/database configurations. Flag reliance on unmanaged external resources.    |

### Safe First Changes

1.  **Dependency Mapping**: Generate a comprehensive dependency graph based on `*.csproj` references and inferred message flows between services. Store this for architectural documentation.
2.  **Commit Pattern Analysis**: Analyze commit messages and patterns across all services to identify common commit types (e.g., feature, bugfix, refactor) and suggest standardization for better tracking.
3.  **Shared Library Impact Analysis**: For any proposed change to `Shared/` or `OrderMaker/Models/`, simulate the compilation impact on all dependent services without committing changes. Report the scope of affected services.
4.  **Worker Service Configuration Scan**: Scan `OutboxProcessor/Program.cs`, `ClaimCheckProcessor/Program.cs`, `MessageReplicator/Program.cs` for hardcoded values or potential environment-specific configurations that could be externalized.

### Danger Zones

1.  **Worker Service Logic Changes**: Any modification to `OutboxProcessor/OrderOutboxWorker.cs`, `ClaimCheckProcessor/`, or `MessageReplicator/` logic without extensive testing and impact analysis is a critical risk. These components are system-critical for event integrity.
2.  **Shared Library Breaking Changes**: Modifying interfaces or models within `Shared/` or `OrderMaker/Models/` without a coordinated, multi-service deployment plan will likely introduce breaking changes across the platform.
3.  **Deployment Without CI/CD**: Pushing changes to production environments without automated testing, build, and deployment pipelines (which are currently absent) carries a high risk of introducing defects and system instability.
4.  **`TestPublisher` in Production**: The `TestPublisher/Program.cs` utility should *never* be deployed or run in a production context. Its presence, if not managed, poses a security and data integrity risk.
5.  **Unmanaged External Dependencies**: Changes to underlying message broker or database configurations that are not tracked or validated through automated means pose a significant risk to the entire system's operation.

### Checklists

#### Pre-Release Review Checklist (AI-Assisted)

*   [ ] **Worker Service Impact**: Has the proposed change been analyzed for its impact on `OutboxProcessor`, `ClaimCheckProcessor`, and `MessageReplicator`?
*   [ ] **Shared Component Review**: If `Shared/` or `OrderMaker/Models/` are modified, are all dependent services identified and their compatibility confirmed?
*   [ ] **Dependency Changes**: Are there any changes to external message broker or database interactions? Are they validated for compatibility and performance?
*   [ ] **Rollback Plan**: Has a clear, automated rollback strategy been documented in case of deployment issues? (Crucial due to lack of explicit CI/CD).
*   [ ] **Observability Readiness**: Are new metrics or logging implemented for the changes, especially for event flow critical paths?

#### Risk Assessment Checklist (AI-Driven)

*   [ ] **Missing CI/CD**: Is the absence of explicit CI/CD pipelines (`.github/workflows/`, `azure-pipelines.yml`, `Jenkinsfile`) formally acknowledged and risk-rated for each release?
*   [ ] **Worker Service Health**: Query available monitoring tools (if integrated) for `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator` error rates, latency, and throughput. Flag deviations.
*   [ ] **`TestPublisher` Usage**: Verify that `TestPublisher/Program.cs` is strictly isolated to development/testing environments and not inadvertently deployed or invoked in production.
*   [ ] **Code Duplication**: Identify and report instances of similar logic across multiple services that could indicate missing shared libraries or technical debt.

#### Process Improvement Checklist (AI-Recommended)

*   [ ] **Advocate CI/CD**: Identify and propose specific patterns for introducing CI/CD pipelines (e.g., `build.yml` in `.github/workflows/`) for `AccountService`, `InventoryService`, and `OrchestratorService` first.
*   [ ] **Centralized Configuration**: Recommend moving environment-specific configurations from `Program.cs` directly into `appsettings.json` or similar externalized mechanisms for all services.
*   [ ] **Worker Service Observability**: Suggest implementing advanced logging and metrics for `OutboxProcessor`, `ClaimCheckProcessor`, and `MessageReplicator` to proactively detect issues.
*   [ ] **Shared Contract Versioning**: Propose strategies for versioning contracts in `Shared/` to manage breaking changes more effectively across services.