## Relevance to This Codebase

In an event-driven microservices architecture like this one, strong technical leadership is paramount for maintaining system health, consistency, and scalability. This codebase features multiple interconnected services (`AccountService`, `InventoryService`, `OrchestratorService`, etc.), worker processes (`OutboxProcessor`, `ClaimCheckProcessor`), and a `Shared` library. Without clear technical guidance, standards, and oversight, such a distributed system can quickly suffer from inconsistent patterns, duplicated efforts, diverging data contracts, and unmanageable complexity.

Technical leadership is critical here to:
*   Ensure consistent application of event-driven patterns (e.g., Outbox, Claim Check).
*   Govern changes to shared contracts and the `Shared/` library, which have wide-reaching impacts.
*   Define and enforce quality standards, especially in the absence of explicit CI/CD or IaC.
*   Guide architectural evolution and ensure services adhere to defined boundaries and communication protocols.
*   Mitigate risks associated with distributed transactions and eventual consistency.
*   Facilitate efficient onboarding and collaboration across service teams, even if informal today.

## Current State Assessment

| Artifact | Location/Pattern | Maturity Level | Notes |
| :------------------------------ | :---------------------------------------------------------------------------------------------------------------------- | :--------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Architectural Structure** | Root directory structure (`AccountService/`, `InventoryService/`, `Shared/`, etc.) | HIGH             | Clearly defined services, separated by bounded context, with dedicated folders. Strong indicator of microservices and event-driven architecture. |
| **Shared Library Usage** | `Shared/` directory, referenced by multiple `*.csproj` files | HIGH             | Centralized location for common code, data contracts, and utilities, promoting consistency. |
| **Event-Driven Patterns** | `OutboxProcessor/`, `ClaimCheckProcessor/`, `*.Worker/` services | HIGH             | Explicit implementation of outbox and claim check patterns, and clear worker roles for asynchronous processing. |
| **Service Entry Points** | `Program.cs` for API services, `Worker.cs` for background services | HIGH             | Consistent and idiomatic .NET patterns for defining service entry points and background workers. |
| **Containerization** | `Dockerfile`, `.dockerignore` in service roots | MEDIUM           | Docker support is present, indicating readiness for containerized deployment, but no orchestration config is evident. |
| **C# Language Usage** | All project files (`*.csproj`, `*.cs`) | HIGH             | Consistent use of C# across the entire codebase. |
| **Testing Patterns** | (Inferred from project type) `*.Tests` projects are conventional in .NET. (Not explicitly listed in context, but standard practice) | LOW (Inferred)   | While common, specific testing frameworks or coverage configurations are not detailed. |
| **Development Workflow** | (Implicit) Git for version control | LOW              | Basic version control is assumed, but no explicit branching strategy, commit standards, or pre-commit hooks are configured or documented. |
| **Code Formatting** | (Implicit) Reliance on IDE defaults (`.editorconfig` not present) | LOW (Inferred)   | No explicit `.editorconfig` or `dotnet format` configuration file is present, implying reliance on developer/IDE settings for C# formatting. |
| **Linting/Static Analysis** | (Implicit) Default Roslyn Analyzers within .NET | LOW (Inferred)   | No custom Roslyn analyzer rulesets or explicit static analysis configuration (e.g., SonarQube config) is provided. |

## What's Missing or Weak

| Gap/Weakness | Impact | Priority | Mitigation/Recommendation |
| :------------------------------------------ | :-------------------------------------------------------------------- | :------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **No Formal CI/CD Pipelines or IaC** | Manual, inconsistent, and error-prone deployments; slow feedback loop; increased time to market; reduced reliability. | HIGH     | Implement CI/CD (e.g., GitHub Actions, Azure DevOps Pipelines) and IaC (e.g., Terraform, ARM templates) to automate builds, tests, and deployments. |
| **Lack of Explicit Linting/Formatting Config** | Inconsistent code style across developers; reduced readability; increased merge conflicts. | MEDIUM   | Introduce a `.editorconfig` file for consistent C# formatting and potentially a `Directory.Build.props` for common Roslyn Analyzer rules. |
| **No `.github/pull_request_template.md`** | Inconsistent PR descriptions; missed information; slower reviews. | MEDIUM   | Create a standard PR template to guide developers on required information, context, and checklist items. |
| **No `CODEOWNERS` file** | Ambiguous ownership; delays in getting reviews; potential for unreviewed code changes. | MEDIUM   | Define `CODEOWNERS` for critical services or the `Shared/` library to ensure appropriate team members are involved in reviews. |
| **Lack of Architecture Documentation (ADRs)** | Architectural decisions are tribal knowledge; difficult for new team members to understand system rationale; risk of design drift. | HIGH     | Start documenting key architectural decisions using Architecture Decision Records (ADRs) within a `docs/architecture/` folder. |
| **No `CONTRIBUTING.md` or Coding Standards** | Inconsistent coding patterns; difficulty in establishing best practices; slower code reviews due to style discussions. | HIGH     | Create a `CONTRIBUTING.md` that outlines coding standards, event contract guidelines, and general development practices. |
| **Missing Onboarding Documentation** | High friction for new developers; significant time investment from existing team for setup and knowledge transfer. | MEDIUM   | Develop a `docs/onboarding/GETTING_STARTED.md` with setup instructions, architecture overview, and common development tasks. |
| **Undefined Branching/Commit Strategy** | Potential for merge conflicts; difficulty in tracing changes; inconsistent commit history. | MEDIUM   | Document a branching strategy (e.g., GitFlow, Trunk-Based Development) and possibly adopt conventional commits for better changelog generation. |
| **No Explicit Quality Gates in CI** | Low-quality code can merge without checks (e.g., low test coverage, failing linters); accumulating technical debt. | HIGH     | Integrate code quality checks (linting, test coverage thresholds, static analysis) into CI pipelines. |
| **Inter-Service Contract Evolution** | Breaking changes to events can silently break consumers; difficult to manage compatibility in a distributed system. | HIGH     | Implement schema registry for events, versioning strategies for event contracts, and clear communication protocols for contract changes. |

## Key Patterns and Conventions

| Pattern/Convention           | Where Used                                                       | Notes                                                                                                                                                             |
| :--------------------------- | :--------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Microservice Architecture** | `AccountService/`, `InventoryService/`, `OrchestratorService/`   | Distinct service boundaries align with bounded contexts, providing clear separation of concerns and independent deployability. Each service has its own `Program.cs` or `Worker.cs`. |
| **Event-Driven Messaging**   | All `*.Worker/` services, `OutboxProcessor/`                      | Core architectural style, indicated by `Worker` roles and explicit use of an `OutboxProcessor` for reliable event publishing. Services communicate primarily via events. |
| **Outbox Pattern**           | `OutboxProcessor/OrderOutboxWorker.cs`                           | Ensures atomic event publishing by writing outgoing messages to a database transaction log before publishing them to a message broker. Critical for data consistency. |
| **Claim Check Pattern**      | `ClaimCheckProcessor/` (inferred)                                | Likely used to store large message payloads externally, sending only a reference (claim check) over the message broker, reducing broker load and message size. |
| **Worker Services**          | `AccountService.Worker/`, `InventoryService.Worker/`, `OrchestratorService.Worker/`, `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/` | Dedicated background services leveraging .NET's `IHostedService` pattern (inferred from `Worker.cs` files) for asynchronous event consumption and processing. |
| **Shared Library**           | `Shared/` project, referenced by other `*.csproj` files          | Centralizes common data contracts, utility functions, and possibly domain models to ensure consistency and reduce duplication across services.                     |
| **Containerization with Docker** | `Dockerfile`, `.dockerignore` in service roots                   | Each service is set up for containerization, enabling consistent build and deployment environments.                                                                  |
| **C# Idiomatic Structure**   | `*.csproj` files, `Properties/` folders, `Program.cs`/`Worker.cs` | Adherence to standard .NET project structure and coding conventions for C# applications.                                                                          |

## Risks and Hotspots

| Risk                                    | Location/Manifestation                       | Severity | Mitigation Strategy                                                                                                                                                                                                                              |
| :-------------------------------------- | :------------------------------------------- | :------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`Shared/` Library Breaking Changes**  | `Shared/` directory, all consuming services  | HIGH     | Implement strict code review for `Shared/` changes. Define and enforce clear versioning policies for the `Shared` library. Require all dependent services to be retested and potentially redeployed for major `Shared` updates. |
| **`OutboxProcessor` Failure**           | `OutboxProcessor/`                           | HIGH     | Implement robust monitoring and alerting for `OutboxProcessor` health and backlog. Ensure high availability (e.g., multiple instances). Thoroughly test its resilience to database or message broker failures.                             |
| **Inter-Service Contract Incompatibility** | Event payloads within `Shared/` or service-specific events | HIGH     | Establish an event schema registry. Enforce backward compatibility for event changes. Implement explicit event versioning and consumer-driven contract testing where feasible. Document event contracts thoroughly.                         |
| **Lack of CI/CD & IaC**                 | Project-wide (absence of `.github/workflows/`, Terraform files, etc.) | HIGH     | Prioritize implementation of automated CI/CD pipelines for all services. Adopt Infrastructure as Code (IaC) for environment provisioning to ensure consistency and reliability.                                                      |
| **Inconsistent Code Quality/Standards** | Project-wide (absence of `.editorconfig`, `CONTRIBUTING.md`, etc.) | MEDIUM   | Introduce `.editorconfig` for formatting. Document and enforce coding standards via `CONTRIBUTING.md`. Integrate static analysis tools (Roslyn Analyzers with custom rules) into the build process.                                       |
| **Slow Onboarding/Knowledge Silos**     | Project-wide (absence of `docs/onboarding/`, ADRs) | MEDIUM   | Create comprehensive onboarding documentation. Document key architectural decisions. Encourage cross-team knowledge sharing sessions.                                                                                                    |
| **Manual Deployment Errors**            | Project-wide (absence of automated deployment) | HIGH     | Directly mitigated by implementing CI/CD and IaC. Manual steps are prone to human error, leading to outages or inconsistencies.                                                                                                               |

## AI Agent Guidelines

### DO
*   **DO** always check the `Shared/` directory (`Shared/`) when analyzing data contracts, common utilities, or potential impact areas, as changes here affect multiple services.
*   **DO** examine `Program.cs` for API services (e.g., `AccountService/Program.cs`) and `Worker.cs` for background services (e.g., `OutboxProcessor/OrderOutboxWorker.cs`) to understand service entry points and configurations.
*   **DO** refer to the `Dockerfile` in a service's root directory (e.g., `AccountService/Dockerfile`) to understand its containerization and build process.
*   **DO** identify event communication patterns by examining how `*.Worker` services consume messages and how the `OutboxProcessor/` publishes them.
*   **DO** review the `csproj` files (e.g., `AccountService/AccountService.csproj`) to understand project dependencies and included files.

### DON'T
*   **DON'T** make modifications to the `Shared/` library (`Shared/`) without first considering and evaluating the potential impact on *all* services that depend on it.
*   **DON'T** assume changes to one service's event contract will not affect others; always investigate potential downstream consumers, especially without a formal schema registry.
*   **DON'T** bypass the `OutboxProcessor/` for event publishing if intending to ensure transactional integrity and reliable messaging.
*   **DON'T** assume explicit code quality tooling (linters, static analyzers) are configured; their absence is a current gap