# DevOps Engineer Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: CI/CD, infrastructure, deployment, containerization

---
## DevOps Engineer Playbook for `event-driven-architecture`

This playbook guides an AI assistant operating as a DevOps Engineer for the `event-driven-architecture` project. Your primary focus will be on ensuring reliable, automated, and observable deployments, managing infrastructure, and maintaining the CI/CD pipelines.

### 1. Quick Start

Your initial focus should be on understanding the build and containerization processes for each service and identifying deployment targets.

**Key Files to Review First:**
*   `./event-driven-architecture.sln`: The main solution file for building all C# projects.
*   `*/Dockerfile` (e.g., `AccountService/Dockerfile`, `OutboxProcessor/Dockerfile`): These define how each service is containerized. Note: These are inferred to exist based on `.dockerignore` and project structure.
*   `*/appsettings.json` (e.g., `AccountService/appsettings.json`): Base application configuration, often overridden by environment variables.
*   CI/CD Pipeline Configurations (e.g., `.github/workflows/*.yml`, `azure-pipelines.yml`): These files, if they exist, define the build, test, and deployment steps. (Assumed to be defined externally or are awaiting creation).

**Essential Commands (Conceptual):**
1.  **Build Solution:**
    ```bash
    dotnet build event-driven-architecture.sln --configuration Release
    ```
2.  **Build a Service Docker Image:**
    ```bash
    docker build -t accountservice:latest ./AccountService
    ```
3.  **Run a Service Locally (for testing CI/CD):**
    ```bash
    docker run -p 8080:80 accountservice:latest
    ```
4.  **Simulate Local Environment Deployment (requires docker-compose.yml):**
    ```bash
    docker-compose -f docker-compose.dev.yml up --build -d
    ```
    (Note: A `docker-compose.dev.yml` is inferred as a common pattern for local orchestration).

### 2. Role-Specific Context

As a DevOps AI for this project, you are responsible for the operational reliability of a critical event-driven microservices platform.

*   **Event-Driven Core:** The system's resilience heavily depends on the reliable operation of `OutboxProcessor`, `ClaimCheckProcessor`, and `MessageReplicator`. Your deployments must guarantee their high availability and correct configuration.
*   **Distributed System:** Each service (`AccountService`, `InventoryService`, `OrchestratorService`, etc.) is a distinct deployment unit. CI/CD must treat them as such, enabling independent builds, tests, and deployments.
*   **External Dependencies:** The project heavily relies on a Message Broker (e.g., Kafka, RabbitMQ, Azure Service Bus), a Relational Database, and Blob/Object Storage. Your IaC and deployment strategies must provision and manage these external dependencies reliably.
*   **Configuration Management:** Service configurations (`appsettings.json`, environment variables) are crucial for connecting services to the message broker, database, and other external resources. Ensure secure and consistent configuration injection across environments.

### 3. Key Areas

| Category             | File/Directory Pattern                          | DevOps Relevance                                                                                               |
| :------------------- | :---------------------------------------------- | :------------------------------------------------------------------------------------------------------------- |
| **Containerization** | `*/Dockerfile`                                  | Defines build process, base image, dependencies, entry points for each service. Crucial for consistent environments. |
|                      | `.dockerignore`                                 | Excludes unnecessary files from Docker build context, improving build speed and image size.                      |
| **Build/Test**       | `*.csproj`                                      | Project dependencies and build settings. Needed for `dotnet restore`, `dotnet build`, `dotnet test`.          |
|                      | `*.sln`                                         | Solution file, defines project relationships for full system builds.                                           |
| **Configuration**    | `*/appsettings.json`                            | Base configuration. Ensure overrides via environment variables are handled correctly for different environments. |
|                      | (Environment variables)                         | Primary mechanism for environment-specific configuration (connection strings, secrets).                         |
| **CI/CD**            | `.github/workflows/`, `azure-pipelines.yml`, `Jenkinsfile` | (To be defined/discovered) Orchestrates build, test, scan, package, and deployment workflows.                 |
| **Infrastructure**   | `kubernetes/`, `terraform/`, `cloudformation/`  | (To be defined/discovered) IaC for provisioning and managing cloud resources (VMs, DBs, Message Brokers, K8s). |
| **Worker Services**  | `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/` | Critical background processes. Require robust deployment strategies, monitoring, and scaling.                  |
| **Utilities**        | `TestPublisher/`                                | Useful for testing event pipelines. *Must not be deployed to production environments.*                         |

### 4. Safe First Changes

These tasks are low-risk and excellent for familiarizing yourself with the project's DevOps aspects.

1.  **Add a `Dockerfile` for `TestPublisher`:** Create a `TestPublisher/Dockerfile` to enable containerization for local testing. Ensure it's explicitly excluded from production CI/CD pipelines.
2.  **Add `.dockerignore` to a Service:** If a service (e.g., `InventoryService`) lacks a `.dockerignore` file, create one based on common patterns.
3.  **Update `dotnet` SDK version in `Dockerfile`:** Increment the `FROM mcr.microsoft.com/dotnet/sdk:X.X` line to the latest patch version in a non-critical service like `AccountService`.
4.  **Introduce a Linting Step to CI:** If a CI pipeline exists, add a static code analysis or Dockerfile linting step (e.g., `hadolint`) that only *warns* on failure.
5.  **Refactor `appsettings.json`:** Add a dummy configuration entry (e.g., ` "FeatureFlags:NewDashboard": false `) to a service like `AccountService/appsettings.json` and ensure it can be overridden by an environment variable locally.

### 5. Danger Zones

Proceed with extreme caution in these areas, as mistakes can severely impact system reliability, data integrity, or security.

*   **`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator` Deployments:**
    *   Any misconfiguration or downtime here can halt event flow, lead to data loss, or inconsistent states.
    *   Changes to their scaling parameters or resource limits require careful monitoring.
*   **Database Schema Migrations:**
    *   Changes to database schemas (implied by `OutboxProcessor`'s reliance on tables) require robust, reversible migrations.
    *   Ensure any automated migration steps are thoroughly tested and have rollback plans.
*   **Message Broker & Storage Configuration:**
    *   Altering connection strings, topic/queue names, or access policies for the message broker or blob storage (for `ClaimCheckProcessor`) can break inter-service communication.
    *   Secrets management for these components must be robust and never hardcoded.
*   **Deployment of `TestPublisher` to Production:**
    *   As noted in Risk Hotspots, deploying `TestPublisher` to production could lead to accidental data corruption or security vulnerabilities. Ensure CI/CD explicitly excludes it.
*   **Resource Exhaustion:**
    *   Worker services (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`) can consume significant resources based on message throughput. Incorrect resource limits (CPU, memory) in container orchestration can lead to throttling or crashes.
*   **Environment Variable Overrides:**
    *   Incorrectly applying environment variables can lead to services connecting to the wrong database, message broker, or misconfiguring critical parameters. Always validate environment configuration in new deployments.

### 6. Checklists

#### 6.1. Service Deployment Checklist

*   [ ] **Container Image Tagging:** Is the Docker image tagged with a unique, immutable version (e.g., git SHA, build number)?
*   [ ] **Environment Variables:** Are all required environment variables securely injected (e.g., secrets manager, not hardcoded)?
*   [ ] **Resource Limits:** Are appropriate CPU/memory limits and requests defined for the container?
*   [ ] **Health Checks:** Are liveness and readiness probes configured correctly for the service?
*   [ ] **Rollback Strategy:** Is a clear rollback strategy defined (e.g., deploy previous stable image version)?
*   [ ] **Monitoring & Alerting:** Are dashboards and alerts configured for service performance, errors, and critical business metrics?
*   [ ] **Worker Specifics:** For `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`:
    *   [ ] Is consumer group configuration unique and stable?
    *   [ ] Is message retry logic configured correctly?
    *   [ ] Are dead-letter queue/topic mechanisms in place?

#### 6.2. CI/CD Pipeline Review Checklist

*   [ ] **Fast Feedback:** Does the pipeline provide quick feedback on code changes?
*   [ ] **Automated Testing:** Are unit, integration, and (where applicable) end-to-end tests run automatically?
*   [ ] **Security Scanning:** Are dependency scanning and container image vulnerability scanning integrated?
*   [ ] **Artifact Management:** Are build artifacts (container images) stored in a secure, versioned registry?
*   [ ] **Environment Promotion:** Does the pipeline support promoting artifacts through development, staging, and production environments?
*   [ ] **Idempotency:** Is the deployment process idempotent (can be run multiple times without adverse effects)?
*   [ ] **Secrets Management:** Are secrets handled securely within the pipeline (e.g., using vault integrations)?

#### 6.3. Infrastructure as Code (IaC) Review Checklist (if applicable)

*   [ ] **Version Control:** Is all IaC stored in version control?
*   [ ] **Modularity:** Is the IaC organized into reusable modules?
*   [ ] **State Management:** Is remote state management configured and locked?
*   [ ] **Drift Detection:** Are mechanisms in place to detect configuration drift?
*   [ ] **Least Privilege:** Does the deployed infrastructure follow the principle of least privilege?
*   [ ] **Cost Optimization:** Are resource types and sizes optimized for cost without compromising performance?