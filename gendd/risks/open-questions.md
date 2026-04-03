# Open Questions

This section outlines pending validations, identified gaps, assumptions requiring verification, and recommended improvements for the `event-driven-architecture` codebase. It aims to provide clear action items for new engineers joining the project.

---

### 1. Pending Validation Items

Items that need explicit confirmation or detailed definition.

| Item                            | Impact       | Priority | Reference                                       |
| :------------------------------ | :----------- | :------- | :---------------------------------------------- |
| **Message Broker Choice**       | High (Perf, Cost, Ops) | High     | Tech Stack                                      |
| **Relational Database Choice**  | High (Perf, Cost, Ops, Features) | High     | Tech Stack, Data Map                            |
| **Blob/Object Storage Choice**  | Medium (Cost, Ops, Avail) | Medium   | Tech Stack                                      |
| **Outbox Implementation Details** | High (Data consistency, Perf) | High     | Services (OutboxProcessor), Data Map           |
| **Claim Check Lifecycle**       | Medium (Cost, Data retention) | Medium   | Services (ClaimCheckProcessor)                 |
| **MessageReplicator Exact Purpose** | Medium (System complexity, Data flow) | Medium   | Services (MessageReplicator), Core Flows        |
| **OrchestratorState Schema**    | High (Workflow correctness) | High     | Services (OrchestratorService), Data Map        |
| **Idempotency Strategy**        | High (Data consistency) | High     | Core Flows, all `Worker.cs` services            |

**Actionable Insight:** The core infrastructure choices (Message Broker, Databases, Storage) are currently inferred. Explicit selection and documentation are critical for understanding capabilities, operational overhead, and cost implications.

---

### 2. Known Gaps with Impact and Priority

Identified areas where crucial components or practices are currently missing.

| Gap                                 | Impact (Severity) | Priority (Urgency) | Reference                                     |
| :---------------------------------- | :---------------- | :----------------- | :-------------------------------------------- |
| **CI/CD Pipeline**                  | High (Deployment consistency, Speed) | High               | Risk Hotspots                                 |
| **Infrastructure as Code (IaC)**    | High (Environment consistency, Ops) | High               | Risk Hotspots                                 |
| **Automated Testing Strategy**      | High (Quality, Regression) | High               | Testing Standards                             |
| **Monitoring, Alerting, Logging**   | High (Ops, Debugging, Reliability) | High               | (Inferred - not mentioned)                    |
| **Distributed Tracing**             | High (Debugging, Performance) | High               | (Inferred - not mentioned)                    |
| **Error Handling / DLQ Strategy**   | High (Data integrity, Reliability) | High               | (Inferred - not mentioned)                    |
| **Authentication & Authorization**  | High (Security)   | High               | (Inferred - not mentioned for API/Service-to-Service) |
| **Configuration Management**        | Medium (Ops, Consistency) | Medium             | (Inferred - not mentioned)                    |
| **API Gateway**                     | Medium (Security, Routing) | Medium             | (Inferred - external access)                  |

**Actionable Insight:** These gaps represent fundamental requirements for any production-ready microservices platform. Prioritizing their implementation is crucial to ensure reliability, security, and maintainability.

---

### 3. Assumptions Needing Verification

Key assumptions made during the analysis that must be confirmed.

*   **Business Domain:**
    *   **HYPOTHESIS (Confidence: High):** The system's purpose is e-commerce or a similar domain involving account, inventory, and order management.
    *   **Verification:** Confirm the exact business domain and primary use cases with stakeholders.
*   **.NET Versioning:**
    *   **HYPOTHESIS (Confidence: High):** The project uses a modern .NET version (e.g., .NET 6+).
    *   **Verification:** Determine the specific target .NET version(s) for all projects.
*   **Database per Service Pattern:**
    *   **HYPOTHESIS (Confidence: Medium):** Each primary service (AccountService, InventoryService, OrchestratorService) will have its own dedicated database instance or schema.
    *   **Verification:** Confirm the data ownership and storage strategy (shared vs. dedicated database instances/schemas) with the architecture team. Refer to `Data Map`.
*   **CQRS/Saga Implementation:**
    *   **HYPOTHESIS (Confidence: High):** `AccountService`, `InventoryService` exhibit CQRS-like characteristics, and `OrchestratorService` implements a Saga pattern.
    *   **Verification:** Confirm if these patterns are explicit architectural choices or emergent designs, and if specific libraries or frameworks are used to implement them.
*   **TestPublisher Usage:**
    *   **HYPOTHESIS (Confidence: High):** `TestPublisher` is strictly for local development and testing, and will **not** be deployed to production environments.
    *   **Verification:** Ensure CI/CD pipelines enforce exclusion or removal of `TestPublisher` from production artifacts, or implement robust security controls if it must exist. Refer to `Risk Hotspots`.
*   **Message Delivery Semantics:**
    *   **HYPOTHESIS (Confidence: High):** The chosen message broker provides "at-least-once" message delivery.
    *   **Verification:** Confirm the message broker's delivery guarantees, as this directly influences the necessity for idempotent consumers.

**Actionable Insight:** These assumptions form the basis of many design decisions. Verifying them early prevents misaligned expectations and potential architectural debt.

---

### 4. Recommended Improvements by Priority

Prioritized list of actionable recommendations for system enhancement.

#### **High Priority**

1.  **Implement CI/CD & IaC:**
    *   **Action:** Design and implement robust CI/CD pipelines (e.g., Azure DevOps, GitHub Actions, GitLab CI) for all services, coupled with Infrastructure as Code (e.g., Terraform, Bicep) for all cloud resources.
    *   **Goal:** Ensure consistent, repeatable, and automated deployments from dev to production. (Addresses `Risk Hotspots`)
2.  **Establish Comprehensive Observability:**
    *   **Action:** Integrate a logging framework (e.g., Serilog), structured logging, centralized log aggregation (e.g., ELK, Splunk, Azure Monitor), metrics collection (e.g., Prometheus, Application Insights), alerting rules, and distributed tracing (e.g., OpenTelemetry, Zipkin) across all services.
    *   **Goal:** Provide visibility into system health, performance, and aid in debugging distributed issues.
3.  **Formalize Automated Testing:**
    *   **Action:** Introduce unit, integration, and end-to-end testing frameworks (e.g., XUnit, NSubstitute, Playwright/Selenium). Integrate these tests into the CI pipeline.
    *   **Goal:** Improve code quality, prevent regressions, and provide confidence in changes. (Addresses `Testing Standards`)
4.  **Define and Implement Error Handling & DLQ Strategy:**
    *   **Action:** Establish a standardized approach for handling message processing failures, including retry mechanisms, dead-letter queues (DLQ) for poison messages, and clear procedures for manual intervention.
    *   **Goal:** Improve system resilience and prevent data loss or inconsistencies.
5.  **Implement Authentication & Authorization:**
    *   **Action:** Secure API endpoints with appropriate authentication (e.g., JWT, OAuth2) and authorization mechanisms. Define and implement secure service-to-service communication.
    *   **Goal:** Protect sensitive data and functionality from unauthorized access. (Addresses inferred gap)

#### **Medium Priority**

1.  **Choose and Document Core Technologies:**
    *   **Action:** Make definitive choices for the Message Broker, Relational Databases, and Object Storage. Document these decisions in an Architectural Decision Record (ADR), outlining the rationale and trade-offs.
    *   **Goal:** Solidify the tech stack, enable consistent development, and facilitate operational planning. (Addresses `Pending Validation Items`)
2.  **Implement Centralized Configuration Management:**
    *   **Action:** Introduce a robust solution for managing application configuration (e.g., environment variables, Azure App Configuration, HashiCorp Vault) to externalize and secure environment-specific settings.
    *   **Goal:** Enhance deployability, security, and maintainability across different environments.
3.  **Evaluate and Implement an API Gateway:**
    *   **Action:** Assess the need for an API Gateway (e.g., Ocelot, Azure API Management, AWS API Gateway) to provide a single entry point for external consumers, handle routing, rate limiting, and additional security.
    *   **Goal:** Improve security, management, and scalability of external API access.

#### **Low Priority**

1.  **Document Business Domain & Use Cases:**
    *   **Action:** Create detailed documentation outlining the system's precise business purpose, key user personas, and primary use cases.
    *   **Goal:** Provide foundational context for all team members. (Addresses `Assumptions Needing Verification`)
2.  **Formalize Architectural Decision Records (ADRs):**
    *   **Action:** Institute a practice of documenting significant architectural decisions (e.g., choice of patterns, technologies, trade-offs) using an ADR process.
    *   **Goal:** Maintain a historical record of architectural evolution and rationale.