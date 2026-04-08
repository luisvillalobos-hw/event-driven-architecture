## DevOps & Infrastructure Context Document

### Relevance to This Codebase

For an event-driven microservices architecture like `event-driven-architecture`, robust DevOps and infrastructure practices are paramount. The system relies on multiple interconnected services (AccountService, InventoryService, OrchestratorService, various Workers, OutboxProcessor) that need consistent build, deployment, and operational environments. The absence of formalized CI/CD and Infrastructure as Code (IaC), despite the use of Docker for containerization, presents significant challenges. Without these, deployments are likely manual, inconsistent, and prone to errors, leading to potential service instability, difficult debugging, and slow delivery of new features. Effective DevOps is critical for managing the complexity of inter-service communication, ensuring reliable event delivery (especially with the OutboxProcessor), and maintaining environment parity across development, testing, and production.

### Current State Assessment

| Artifact Category | Description                                          | Location                                                | Maturity Level |
| :---------------- | :--------------------------------------------------- | :------------------------------------------------------ | :------------- |
| **Containerization** | Docker definitions for each service.                | `*/Dockerfile`                                          | HIGH           |
|                    | Global Docker ignore rules.                          | `.dockerignore`                                         | HIGH           |
|                    | Local multi-service orchestration (inferred).        | `docker-compose.yml`                                    | MEDIUM         |
| **Build Automation** | Standard .NET build processes within Dockerfiles.   | `*/Dockerfile` (using `dotnet build`, `dotnet publish`) | HIGH           |
| **Environment Config** | Environment variables within Dockerfiles or `docker-compose.yml`. | `*/Dockerfile`, `docker-compose.yml` (inferred)         | LOW            |

### What's Missing or Weak

| Gap/Weakness                     | Impact                                                                         | Priority |
| :------------------------------- | :----------------------------------------------------------------------------- | :------- |
| **CI/CD Pipelines**              | Manual and inconsistent builds/deployments; slow feedback loops; high risk of human error; no automated testing or security scanning. | CRITICAL |
| **Infrastructure as Code (IaC)** | Manual infrastructure provisioning and configuration; configuration drift; lack of version control for infrastructure; difficult to reproduce environments. | CRITICAL |
| **Centralized Secret Management** | Secrets likely stored directly in configuration files, environment variables, or CI system directly without rotation/auditing. | HIGH     |
| **Automated Deployment Strategy** | No defined blue/green, canary, or rolling update mechanism; manual rollbacks; potential for downtime during deployments. | HIGH     |
| **Environment Management**       | Lack of clear definitions for development, staging, production environments; inconsistent configurations across environments. | MEDIUM   |
| **Container Image Security Scan** | No automated scanning for vulnerabilities in Docker images.                       | MEDIUM   |
| **Observability Integration**    | No explicit integration for metrics, logging, or tracing into CI/CD/deployment workflows. | MEDIUM   |

### Key Patterns and Conventions

| Pattern Name          | Where Used                                         | Notes                                                                      |
| :-------------------- | :------------------------------------------------- | :------------------------------------------------------------------------- |
| **Per-Service Dockerfile** | `AccountService/Dockerfile`, `InventoryService/Dockerfile`, `OutboxProcessor/Dockerfile`, etc. | Each runnable component has its own Dockerfile for containerization.       |
| **Multi-stage Docker Builds** | Within each `*/Dockerfile` (inferred).      | Uses a build stage for compilation and a runtime stage for the final image to reduce size. |
| **Local Orchestration with Docker Compose** | `docker-compose.yml` (inferred).                 | Used for running multiple services and their dependencies locally for development. |
| **Environment Variables for Configuration** | `Dockerfile` `ENV` instructions, `docker-compose.yml` environment sections. | Configuration values are passed into containers via environment variables. |

### Risks and Hotspots

| Risk                               | Location                                                     | Severity | Mitigation                                                       |
| :--------------------------------- | :----------------------------------------------------------- | :------- | :--------------------------------------------------------------- |
| **Manual Deployment Inconsistency** | Entire codebase (due to lack of CI/CD and IaC).              | HIGH     | Implement CI/CD for automated builds/deployments; introduce IaC. |
| **Configuration Drift**            | Production and staging environments (lack of IaC).           | HIGH     | Implement IaC to manage all infrastructure components.           |
| **Secret Exposure/Insecurity**     | `docker-compose.yml`, environment variable definitions (inferred). | HIGH     | Implement a dedicated secret management solution (e.g., Vault, AWS Secrets Manager). |
| **Outdated/Vulnerable Dependencies** | `*.csproj` files (lack of automated scanning).               | MEDIUM   | Integrate dependency vulnerability scanning into CI/CD pipeline. |
| **Service Downtime During Deploy** | All services (lack of automated deployment strategy).        | MEDIUM   | Implement a robust deployment strategy (e.g., rolling updates, blue/green). |
| **Large Docker Image Sizes**       | Individual `*/Dockerfile` (if multi-stage builds are not optimized or base images are large). | LOW      | Review Dockerfiles for optimization opportunities, ensure efficient multi-stage builds. |

### AI Agent Guidelines

#### DO

*   **Refer to `*/Dockerfile`**: Always consult the specific service's `Dockerfile` to understand its build process, base image, and runtime dependencies.
*   **Check `docker-compose.yml`**: Examine the top-level `docker-compose.yml` for local development setup, service interdependencies, and environment variable configurations when addressing local development or testing scenarios.
*   **Identify Build Steps in Dockerfiles**: When proposing changes related to building or packaging, refer to the `dotnet build` and `dotnet publish` steps within the `Dockerfile` for the correct context.
*   **Assume Manual Operations**: When asked about deployment or infrastructure management, assume these are currently manual processes and highlight the need for automation.

#### DON'T

*   **Assume CI/CD Automation**: Do not assume the existence of CI/CD pipelines (e.g., GitHub Actions, GitLab CI, Jenkins) for build, test, or deploy operations.
*   **Assume IaC Management**: Do not assume that cloud infrastructure (e.g., databases, message brokers, compute instances) is managed by Infrastructure as Code tools (e.g., Terraform, Pulumi).
*   **Suggest In-Place Updates**: Do not suggest or rely on the ability to update running containers or infrastructure components without a complete redeployment through a defined process.
*   **Embed Secrets Directly**: Never suggest adding secrets directly into Dockerfiles, `docker-compose.yml`, or version-controlled configuration files.

#### ALWAYS CHECK

*   **`*/Dockerfile` for each service**: Before proposing any container-related change (base image, dependencies, entrypoint).
*   **`docker-compose.yml`**: For how services are linked and configured in a local development environment.
*   **The absence of CI/CD configuration files**: Before discussing deployment automation or pipeline steps.
*   **The absence of IaC files**: Before discussing infrastructure provisioning or management.
*   **Environment variable usage**: How configuration is passed to services, particularly for environment-specific settings.

### Related Context Areas

*   [architecture](./architecture.md) — For understanding how services interact and how deployments might affect system design.
*   [site-reliability](./site-reliability.md) — For addressing monitoring, logging, and incident response implications for deployments.
*   [security](./security.md) — For understanding secret management, image scanning, and overall DevSecOps posture.
*   [release-management](./release-management.md) — For informing strategies around versioning, release cadence, and deployment artifacts.
*   [quality-assurance](./quality-assurance.md) — For integrating automated testing into any future CI/CD processes.

### Key Files

| File/Pattern              | Purpose                                                      | Notes                                                              |
| :------------------------ | :----------------------------------------------------------- | :----------------------------------------------------------------- |
| `*/Dockerfile`            | Defines how each individual service/worker is containerized. | Critical for understanding build process and runtime environment.  |
| `.dockerignore`           | Specifies files/directories to exclude from Docker build context. | Ensures efficient and secure image builds.                         |
| `docker-compose.yml`      | Orchestrates multiple services for local development/testing. | Crucial for understanding local service dependencies and setup.    |
| `scripts/` (if present)   | Custom build or deployment scripts.                          | Currently assumed minimal, but could exist for manual tasks.       |