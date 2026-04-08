# Quality Assurance Context Document

## Relevance to This Codebase

For an event-driven microservices architecture like this project, robust Quality Assurance (QA) and automated testing are paramount. The distributed nature, reliance on asynchronous communication, eventual consistency patterns (like the Outbox), and numerous inter-service dependencies (`Shared/` library, event contracts) introduce significant complexity and potential for hard-to-diagnose issues. Without comprehensive automated tests, changes can easily introduce regressions, break event contracts, or cause data inconsistencies across services. The current apparent lack of automated testing presents a critical risk to the reliability, maintainability, and safe evolution of this system.

## Current State Assessment

| Artifact | Location/Pattern | Maturity Level | Notes |
| :------- | :--------------- | :------------- | :---- |
| **Automated Tests (Unit, Integration, E2E, Contract)** | *No identifiable files/directories* | **Absent** | No dedicated test project files (`*.Tests.csproj`), test directories (`tests/`, `__tests__/`), or common test framework configurations found. This indicates a complete absence of automated testing. |
| **Test Frameworks** | *Not applicable* | **Absent** | No evidence of xUnit, NUnit, MSTest, or any other .NET testing framework configuration or usage. |
| **Test Data Management** | *Not applicable* | **Absent** | No patterns for generating or managing test data could be identified. |
| **Test Coverage Reporting** | *Not applicable* | **Absent** | No configuration for coverage tools (e.g., Coverlet) found. |
| **CI/CD Test Integration** | *Not applicable* | **Absent** | No CI/CD workflow files (`.github/workflows/`) are present that would indicate test stages. |
| **Test Utility** | `TestPublisher/` | **Low** | A simple console application likely used for manually sending events for development or ad-hoc testing, not for automated QA. |

## What's Missing or Weak

| Gap Type                 | Impact                                                                                                 | Priority |
| :----------------------- | :----------------------------------------------------------------------------------------------------- | :------- |
| **Unit Tests**           | Undetected bugs in core business logic within each service, making refactoring unsafe.                 | HIGH     |
| **Integration Tests**    | Failure to validate inter-service communication, event handling, and database interactions (e.g., Outbox pattern). Leads to silent contract breaks. | HIGH     |
| **End-to-End (E2E) Tests** | Critical user journeys across multiple services are not validated, risking major production outages.    | HIGH     |
| **Contract Tests**       | Event schema or API contract changes can break downstream consumers without detection, leading to data inconsistencies. | HIGH     |
| **CI/CD Test Integration** | No automated feedback loop for regressions; developers are unaware of broken code until deployment or manual testing. | HIGH     |
| **Test Data Management** | Inability to consistently set up test environments, leading to brittle and unreproducible manual tests.  | HIGH     |
| **Performance Tests**    | No validation of service throughput or latency under load, risking performance bottlenecks in production. | MEDIUM   |
| **Security Tests**       | No automated checks for common vulnerabilities (e.g., authentication bypass, injection).                 | MEDIUM   |

## Key Patterns and Conventions

| Pattern                  | Where Used           | Notes                                                                                                                                                                                                                                 |
| :----------------------- | :------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Manual Event Publishing** | `TestPublisher/`     | This project acts as a utility to manually send events. It is a tool for ad-hoc testing or development validation, not an automated test suite. It implies manual verification is currently the primary method for testing event flows. |
| **Outbox Pattern**       | `OutboxProcessor/`   | The Outbox pattern ensures transactional publishing of events. While critical for reliability, its correct functioning is not verified by automated tests, creating a significant risk hotspot.                                       |
| **Shared Library Contracts** | `Shared/`            | The `Shared/` library defines common data contracts and utilities. Any changes here impact all dependent services. The lack of contract tests means breaking changes can easily propagate undetected.                               |

## Risks and Hotspots

| Risk                                     | Location/Pattern                       | Severity | Mitigation Strategy (AI Agent Guidance)                                                                                     |
| :--------------------------------------- | :------------------------------------- | :------- | :-------------------------------------------------------------------------------------------------------------------------- |
| **Undetected Business Logic Bugs**       | All services (e.g., `AccountService/`, `InventoryService/`, `OrchestratorService/`) | HIGH     | Prioritize adding unit tests for critical business logic paths. Any change to core logic should be flagged for manual review and testing. |
| **Inter-Service Communication Breakage** | Between all services (event contracts) | HIGH     | Implement contract testing for event schemas and API endpoints. Any changes to message contracts (`Shared/`) should be treated as extremely high risk. |
| **Outbox Pattern Failure**               | `OutboxProcessor/`                     | HIGH     | Crucial for transactional reliability; lack of tests risks data inconsistency. Flag `OutboxProcessor/` changes for immediate, in-depth manual testing. |
| **Regression Bugs from Changes**         | Entire codebase                        | HIGH     | Without automated tests, every code change risks introducing regressions. Recommend comprehensive manual regression testing for all deployments. |
| **Delayed Bug Detection**                | Entire codebase                        | HIGH     | Bugs are likely to be found late in the development cycle or, worse, in production, increasing cost and impact. Urge immediate attention to test automation. |
| **Difficult & Costly Refactoring**       | Entire codebase                        | HIGH     | Fear of breaking existing functionality will hinder necessary refactoring and architectural improvements.                   |
| **Scaling Issues from Untested Code**    | All services                           | MEDIUM   | Performance issues may not be detected until production load. Consider performance testing early.                           |

## AI Agent Guidelines

### Do
- **Do** always assume there are no automated tests protecting any part of the codebase.
- **Do** always recommend discussing a testing strategy (manual or automated) for any proposed change.
- **Do** emphasize the importance of manual regression testing before any deployment, especially for changes to critical paths (e.g., `OutboxProcessor/`, `Shared/`).
- **Do** highlight the high risk of making changes to `Shared/` library without automated test verification.
- **Do** advise creating unit tests for new business logic components if implementing new features.

### Don't
- **Don't** assume any test coverage exists for existing functionality.
- **Don't** suggest refactoring complex logic without first establishing some form of test safety net, even if it's manual.
- **Don't** proceed with significant architectural changes (e.g., event contract modifications) without explicitly flagging the lack of contract tests.

### Always Check
- **Always check** for any proposed changes to event schemas defined in `Shared/` or services' request/response models, and flag them as high risk due to the absence of contract tests.
- **Always check** if changes are being made to `OutboxProcessor/`, `AccountService/`, `InventoryService/`, or `OrchestratorService/` and flag them as critical areas that require dedicated manual testing efforts.
- **Always check** if new features are being added without corresponding tests, and prompt for how their correctness will be validated.

## Related Context Areas

-   **backend-development**: Details about service implementation and internal logic are relevant for understanding where unit and integration tests are needed.
-   **devops-infrastructure**: The lack of CI/CD pipeline integration is a critical gap that prevents automated tests from running.
-   **delivery-management**: The absence of a strong QA process directly impacts release confidence, delivery speed, and overall project risk.

## Key Files

| Path or Glob                                      | Purpose                                                    | Notes                                                                                                                                                                   |
| :------------------------------------------------ | :--------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AccountService/`                                 | Core business logic for accounts.                          | Critical area requiring unit, integration, and API tests.                                                                                                               |
| `AccountService.Worker/`                          | Background processing for accounts.                        | Needs integration tests for event consumption and side effects.                                                                                                         |
| `InventoryService/`                               | Core business logic for inventory.                         | Critical area requiring unit, integration, and API tests.                                                                                                               |
| `InventoryService.Worker/`                        | Background processing for inventory.                       | Needs integration tests for event consumption and side effects.                                                                                                         |
| `OrchestratorService/`                            | Business process orchestration.                            | Highly complex, requiring extensive integration and E2E tests to validate workflows.                                                                                    |
| `OrchestratorService.Worker/`                     | Background processing for orchestration.                   | Critical for workflow execution; needs integration tests for event handling and process state changes.                                                                  |
| `OutboxProcessor/OrderOutboxWorker.cs`            | Reliably publishes messages from outbox.                   | **CRITICAL HOTSPOT.** Requires dedicated integration tests to ensure transactional integrity and reliable publishing.                                                   |
| `Shared/`                                         | Common contracts, models, and utilities.                   | **CRITICAL HOTSPOT.** Changes here have system-wide impact. Requires contract tests and careful regression testing across all services.                                 |
| `TestPublisher/Program.cs`                        | Utility for manual event publishing.                       | The only "test-related" artifact, indicating current reliance on manual event triggering for validation. Not an automated test suite.                                    |
| All `*.csproj` files (e.g., `AccountService/AccountService.csproj`) | Project definitions for services and libraries.            | Absence of corresponding `*.Tests.csproj` files explicitly shows no dedicated test projects are set up, which is common in .NET for automated tests.                   |
| All `Dockerfile` files                            | Containerization definitions.                              | While not directly QA, these imply containerized deployments, which would benefit greatly from automated tests verifying container health and application readiness. |