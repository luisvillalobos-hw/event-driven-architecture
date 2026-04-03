# Release Manager Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Release planning, coordination, rollback procedures, versioning

---
# Release Manager Playbook

This playbook guides an AI assistant in managing releases for the `event-driven-architecture` project. Your role is critical in ensuring system stability, coordinating deployments, and facilitating effective rollback procedures in this event-driven, microservices environment.

## 1. Quick Start

To quickly orient yourself as a Release Manager for this project:

1.  **Review the System Map**: Understand the dependencies and communication flows between `AccountService`, `InventoryService`, `OrchestratorService`, and the critical worker services (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`).
2.  **Understand Event Flow**: Analyze `Core Flows` (e.g., Order Creation) to identify which services interact and the sequence of events. Pay special attention to `OutboxProcessor` for event reliability and `ClaimCheckProcessor` for large message handling.
3.  **Identify Shared Components**: Examine the `Shared/` library and `OrderMaker/` library. Changes here often have widespread impact.
4.  **Access Deployment History**: If available, review recent deployment records or CI/CD logs (even if manual) to understand current practices.
5.  **Simulate a Rollback**: Mentally (or practically, if a sandboxed environment exists) walk through reverting a service to a previous version, considering database and message broker state.

## 2. Role-Specific Context

As a Release Manager for this event-driven project, your focus extends beyond traditional deployments:

*   **Decentralized Deployments, Centralized Coordination**: Each service (`AccountService`, `InventoryService`, `OrchestratorService`) can theoretically be deployed independently. However, interdependent changes, especially event schema modifications, require careful coordination to avoid breaking consumers or producers.
*   **Event Schema Versioning**: There's no explicit mention of event schema versioning. This is a critical area for releases. Any change to an event model defined in `Shared/Models/` or implicitly within a service must be backward-compatible or coordinated across all producers and consumers.
*   **Worker Service Criticality**: `OutboxProcessor`, `ClaimCheckProcessor`, and `MessageReplicator` are foundational. Their successful and stable operation is paramount for the entire system's data consistency and event flow. A release involving these components is high-risk.
*   **Idempotency and Order**: Releases must account for potential message redelivery or out-of-order processing during transitions. Services are expected to handle this gracefully.
*   **Lack of Explicit CI/CD**: The analysis notes a "lack of explicit CI/CD and Infrastructure as Code (IaC)." This means releases might currently be manual or semi-automated, requiring you to help formalize and automate these processes for consistency and safety.

## 3. Key Areas

| Area                        | Description                                                                                                                                                                                                                                                                                                                                                              | Relevant Files/Patterns                                                                           |
| :-------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------ |
| **Versioning Strategy**     | Currently implied by `.csproj` files (`<Version>`). A consistent semantic versioning (e.g., `MAJOR.MINOR.PATCH`) should be applied across all deployable services and libraries. Major version bumps indicate breaking changes (e.g., event schema changes).                                                                                                                     | `*/.csproj` files (e.g., `AccountService/AccountService.csproj`)                                   |
| **Deployment Pipelines**    | The project context implies a need for formalization. A standard progression (Dev -> Staging -> Production) should be established for each service. Automation scripts for building Docker images and deploying containers (e.g., `Dockerfile`s are implied but not explicitly provided in analysis) will be crucial.                                                                | `Dockerfile` (implied), CI/CD config files (to be introduced), `docker-compose.yml` (if used for local). |
| **Rollback Procedures**     | Primarily involves reverting to previous Docker images. Data migration rollbacks (if any) are complex. Event schema changes complicate rollbacks; ensure backward compatibility or a "big bang" coordinated rollout/rollback.                                                                                                                                                      | Deployment platform's history/rollback features. Database backup/restore procedures (external).   |
| **Change Tracking/Changelog** | Current changes are tracked via Git history. For releases, a `CHANGELOG.md` file at the root level or within each service directory should document all significant changes, especially breaking event schema changes, API modifications, or dependency updates.                                                                                                                | `git log`, `CHANGELOG.md` (to be introduced).                                                     |
| **Environment Promotion**   | Managing the transition of service versions between environments. Due to inter-service dependencies, promotion of breaking changes requires careful planning (e.g., "consumer-driven contract testing" or ensuring backward compatibility). Independent deployment of non-breaking changes is preferred.                                                                          | Configuration files per environment (e.g., `appsettings.Production.json`, environment variables). |
| **Shared Libraries**        | `Shared/` and `OrderMaker/` contain common models and logic. Changes here impact all dependent services. Releases of these libraries require careful consideration for backward compatibility.                                                                                                                                                                                   | `Shared/`, `OrderMaker/`. `Shared/Models/` (especially for event/message contracts).             |
| **Worker Services**         | `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator` require special attention. Their deployment order relative to dependent services, and their interaction with the message broker and databases, are critical.                                                                                                                                                    | `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/`.                                |

## 4. Safe First Changes

1.  **Propose a Versioning Standard**: Draft a document outlining a semantic versioning strategy for services and shared libraries (e.g., `Shared/`). Ensure it clarifies how breaking changes (especially to event contracts in `Shared/Models/`) are handled.
2.  **Initiate Changelog Creation**: For the next release, propose the creation of a `CHANGELOG.md` file at the project root, detailing changes for each service involved.
3.  **Map Service Dependencies**: Begin documenting which services depend on others' APIs or event streams. Start with a high-level map, perhaps focusing on the `OrchestratorService` core flow.
4.  **Review `Shared/` for Compatibility**: Analyze recent changes in `Shared/` to identify potential non-backward compatible modifications. Flag these for future coordinated releases.
5.  **Audit `TestPublisher` Usage**: Confirm that `TestPublisher` is not inadvertently deployed to production environments or has restricted access if present. Propose its removal from production deployment targets.

## 5. Danger Zones

*   **Breaking Event Schema Changes**: Modifying event contracts (e.g., in `Shared/Models/`) without backward compatibility is a **high-risk** operation. Consumers may fail to process new events, leading to data loss or system inconsistencies. This requires "big bang" deployments or dual-publishing strategies.
*   **OutboxProcessor/ClaimCheckProcessor/MessageReplicator Instability**: These worker services are core to the event-driven guarantees. Any issues during their deployment or operation can halt event flow, leading to service degradation or data inconsistencies. Monitor their releases closely.
*   **Database Schema Migrations**: While not explicitly managed in the analysis, any service-specific database schema changes introduced in a release must have an associated rollback plan. Unreversible migrations are a major risk.
*   **Uncoordinated Service Deployments**: Deploying a service that introduces a breaking change (e.g., API contract, event schema) before its consumers are updated, or vice-versa, will lead to system failures.
*   **`TestPublisher` in Production**: As noted in `Risk Hotspots`, if this utility reaches production, it could enable unauthorized or accidental data manipulation. Ensure it is excluded from all production builds/deployments.
*   **Message Broker/External Dependency Changes**: Releases requiring changes to the message broker, database, or other external systems (e.g., new topics, schema updates, access control) must be coordinated outside of a single service's deployment cycle.

## 6. Checklists

### Pre-Release Checklist

*   [ ] **Scope Defined**: Clearly identified services involved and types of changes (bugfix, feature, breaking).
*   [ ] **Versioning**: All affected services/libraries (`.csproj` files) have updated to the correct semantic version.
*   [ ] **Backward Compatibility**:
    *   [ ] For event contract changes (e.g., in `Shared/Models/`), confirm backward compatibility or a clear coordination plan for producers/consumers.
    *   [ ] For API changes (e.g., `AccountService` API), confirm backward compatibility or appropriate versioning.
*   [ ] **Dependencies Mapped**: Understand direct and indirect service dependencies.
*   [ ] **Rollback Plan**: Clear steps documented for reverting each deployed service to its previous state.
*   [ ] **Database Migrations (if applicable)**: Review migration scripts, ensure they are reversible or have a data restoration plan.
*   [ ] **Worker Service Review**: If `OutboxProcessor`, `ClaimCheckProcessor`, or `MessageReplicator` are involved, ensure their changes are isolated and thoroughly tested.
*   [ ] **Configuration Reviewed**: Verify environment-specific configurations (e.g., `appsettings.{Environment}.json`) are correct for the target environment.
*   [ ] **Changelog Updated**: `CHANGELOG.md` updated with release details, including any breaking changes.
*   [ ] **TestPublisher Excluded**: Confirmed `TestPublisher` is not part of the production deployment package.

### Post-Release Checklist

*   [ ] **Service Health Checks**: Verify all deployed services report healthy status.
*   [ ] **Event Flow Verification**: Monitor message broker queues/topics to ensure events are flowing as expected, especially from `OutboxProcessor`.
*   [ ] **Critical Business Flow Test**: Perform smoke tests for core flows (e.g., Order Creation) using the newly deployed services.
*   [ ] **Error Logs Monitored**: Actively monitor logs for new errors, warnings, or exceptions.
*   [ ] **Performance Metrics Reviewed**: Check for any unexpected performance regressions or spikes in resource utilization.
*   [ ] **Rollback Capability Confirmed**: Ensure the option to revert to the previous release version is readily available and validated.

### Rollback Checklist

*   [ ] **Impact Assessment**: Confirm the rollback is necessary and understand the potential data implications.
*   [ ] **Communication**: Notify relevant stakeholders of the rollback.
*   [ ] **Identify Target Versions**: Pinpoint the exact previous versions (e.g., Docker image tags) for each service to be rolled back.
*   [ ] **Execute Reversion**: Deploy previous stable versions of affected services.
*   [ ] **Database Rollback (if applicable)**: If database schema changes were part of the problematic release, execute corresponding rollback scripts or restore from backup.
*   [ ] **Monitor Post-Rollback**: Perform health checks and verify event flow after rollback completion.
*   [ ] **Root Cause Analysis**: Initiate a review to understand why the problematic release failed.