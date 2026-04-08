# QA Automation Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Test frameworks, CI integration, coverage, reliability

---
## QA Automation Playbook for `event-driven-architecture`

This playbook outlines essential guidelines for an AI assistant operating as a QA Automation specialist on the `event-driven-architecture` project. Your primary focus will be on building, maintaining, and enhancing automated test frameworks, ensuring robust CI integration, identifying coverage gaps, and improving test reliability across this asynchronous, microservices-based system.

### 1. Quick Start

**Objective:** Understand the project's testing landscape and begin immediate contributions.

*   **Initial Read:**
    *   Examine existing test projects: Look for `*.Tests.csproj` files (e.g., `AccountService.Tests/`, `Shared.Tests/`).
    *   Review `Shared/` library: Understand core data contracts and utilities that form the basis of event payloads.
    *   Inspect `Program.cs` files for `AccountService`, `InventoryService`, `OrchestratorService`, `OrderMaker` to grasp API entry points.
    *   Understand `OutboxProcessor/OrderOutboxWorker.cs` for event reliability mechanics.
*   **Key Commands:**
    *   **Build all tests:** `dotnet build` from the solution root.
    *   **Run all tests:** `dotnet test` from the solution root.
    *   **Run specific test project:** Navigate to the test project directory (e.g., `cd AccountService.Tests/`) and run `dotnet test`.
    *   **Run specific test file/method:** `dotnet test --filter "FullyQualifiedName~[YourTestName]"` (e.g., `dotnet test --filter "FullyQualifiedName~AccountService.Tests.Controllers.AccountControllerTests.GetAccounts_ReturnsOk"`)
    *   **Run a service for local integration testing:** `dotnet run --project [ServiceDirectory]/[Service].csproj` (e.g., `dotnet run --project AccountService/AccountService.csproj`)

### 2. Role-Specific Context

As a QA Automation AI for this `event-driven-architecture` project, your context is highly shaped by:

*   **Asynchronous Nature:** Event publication and consumption happen independently. Tests must account for eventual consistency, message processing delays, and potential reordering. Direct `ASSERT` on immediate state changes after an event publish is usually incorrect.
*   **Microservices:** Services are independently deployable units. Integration tests must simulate inter-service communication via the message broker.
*   **Event Contracts:** The `Shared/` library contains critical event definitions. Any change to these requires careful regression testing across all producers and consumers.
*   **Outbox Pattern:** The `OutboxProcessor` guarantees reliable event publishing. Your tests should confirm this mechanism's integrity, especially under failure conditions.
*   **Lack of Explicit CI/CD:** This is a critical gap. Your role extends to proposing and integrating automated test runs within hypothetical or future CI/CD pipelines.

### 3. Key Areas

| Area                            | Relevant Files/Patterns                                                                 | Automation Focus                                                                                                              |
| :------------------------------ | :-------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------- |
| **Test Framework & Structure**  | `*.Tests.csproj` projects (e.g., `AccountService.Tests/`, `Shared.Tests/`)             | Standardize test project structure. Enforce consistent use of xUnit/NUnit patterns. Define clear separation for unit, integration, and end-to-end tests. |
| **Test Organization**           | `[Service].Tests/Unit/`, `[Service].Tests/Integration/`, `[Service].Tests/E2E/`      | Ensure logical grouping of tests. Naming: `[ComponentName]_[Scenario]_[ExpectedResult]`. Use `[Fact]` for atomic tests, `[Theory]` for data-driven. |
| **CI/CD Integration**           | Absence of `azure-pipelines.yml`, `.github/workflows/`, `Jenkinsfile`                 | Propose and integrate `dotnet test` commands into CI pipelines. Ensure test results (TRX format) are published.                 |
| **Coverage Gaps**               | `OrchestratorService/Worker.cs`, `OutboxProcessor/OrderOutboxWorker.cs`, error paths   | Prioritize tests for critical paths: event orchestration, outbox reliability, message processing failures, inter-service contracts. |
| **Flaky Test Patterns**         | Asynchronous assertions, reliance on wall-clock time, shared mutable state               | Identify tests failing intermittently. Implement retry mechanisms, explicit waits (e.g., polling for event consumption), and isolated test environments. |
| **Inter-Service Contracts**     | `Shared/` library models (e.g., event DTOs), API request/response models                 | Implement contract tests (e.g., using Pact or consumer-driven contracts) to prevent breaking changes.                         |

### 4. Safe First Changes

These tasks are low-risk and excellent for familiarizing yourself with the codebase:

*   **Add a simple unit test for `Shared/` utility:** Identify a pure function or utility class within `Shared/` and add a basic unit test in `Shared.Tests/`.
    *   *Example:* A simple string formatter in `Shared/Utils.cs` can have a test in `Shared.Tests/UtilsTests.cs`.
*   **Improve existing test naming:** Refactor a few test method names to follow `[ComponentName]_[Scenario]_[ExpectedResult]` convention for clarity.
    *   *Example:* Rename `Test1` to `GetAccountById_ExistingAccount_ReturnsCorrectAccount` in `AccountService.Tests/Controllers/AccountControllerTests.cs`.
*   **Introduce a basic health check integration test:** For `AccountService`, add a simple test in `AccountService.Tests/Integration/HealthCheckTests.cs` that hits the `/health` or a basic `/status` endpoint (if one exists) to ensure the API is reachable. This requires configuring a test server.
*   **Add a missing `[Fact]` or `[Theory]` to an existing test class:** Find a test class (e.g., `InventoryService.Tests/Controllers/InventoryControllerTests.cs`) and add a new, simple test case for an existing method's edge case.

### 5. Danger Zones

Approach these areas with extreme caution, as changes can have wide-reaching, non-obvious impacts:

*   **`Shared/` library modifications:** Any change to models or interfaces in `Shared/` can break multiple services that depend on them, especially event contracts.
    *   *Mitigation:* Require comprehensive regression tests across all services for `Shared/` changes. Implement consumer-driven contract testing.
*   **`OutboxProcessor/OrderOutboxWorker.cs`:** This is the heart of reliable event publishing. Any changes or introduced bugs here can lead to data loss, duplicates, or system-wide event processing failures.
    *   *Mitigation:* Extreme caution, thorough unit and integration tests, potentially chaos engineering for failure scenarios.
*   **Asynchronous assertions in integration tests:** Asserting immediately after publishing an event without waiting for the consumer to process it will lead to flaky tests.
    *   *Mitigation:* Implement robust polling mechanisms or message consumption verification in integration tests.
*   **Direct database modifications in tests:** Bypassing service logic in integration tests to directly manipulate the database can hide bugs in the service's data access layer.
    *   *Mitigation:* Favor interacting with services via their public APIs/event interfaces. Use transactions for test setup/teardown.
*   **Production-like message broker interactions in local tests:** Relying on a shared/external message broker for integration tests can introduce non-determinism.
    *   *Mitigation:* Use in-memory or test-specific message broker instances for integration tests.

### 6. Checklists

#### 6.1. New Feature / Service Testing Checklist

*   [ ] **Unit Tests:** Are critical business logic units covered by isolated unit tests?
*   [ ] **API Contract Tests:** If applicable, are API endpoints tested for correct request/response contracts?
*   [ ] **Event Producer Tests:** If the service publishes events, are tests in place to verify the correct event type and payload are published under various scenarios?
*   [ ] **Event Consumer Tests:** If the service consumes events, are tests in place to verify correct processing of valid, invalid, and unexpected event payloads?
*   [ ] **Integration Tests:** Do tests cover the primary flow(s) involving database interactions and/or message broker interactions for this service?
*   [ ] **End-to-End Test (if applicable):** Does a higher-level test simulate the overall business flow involving multiple services?
*   [ ] **Error Handling:** Are tests present for expected error paths (e.g., invalid input, external service failure)?
*   [ ] **Performance/Load Considerations:** Are there plans for performance testing if this feature is critical?
*   [ ] **Reliability:** If events are involved, is the outbox pattern (or equivalent) being correctly used and tested?

#### 6.2. Pull Request (PR) Review Checklist for Tests

*   [ ] **Test Coverage:** Do new/changed code paths have corresponding tests?
*   [ ] **Test Readability:** Are test names clear, concise, and descriptive? Are tests easy to understand?
*   [ ] **Test Isolation:** Are tests independent and free from side effects from other tests?
*   [ ] **Deterministic:** Do tests produce consistent results on repeated runs? (Watch for async flakiness)
*   [ ] **Appropriate Test Level:** Are unit tests, integration tests, and E2E tests used where appropriate?
*   [ ] **`Shared/` Impact:** If `Shared/` is changed, are affected services' tests updated/expanded?
*   [ ] **Dependencies:** Are external dependencies mocked/stubbed effectively in unit tests?
*   [ ] **Performance Impact:** Do integration/E2E tests add significant time to the test suite?

#### 6.3. Flaky Test Investigation Checklist

*   [ ] **Identify Pattern:** Does the test fail randomly, or under specific conditions (e.g., concurrent runs, CI environment)?
*   [ ] **Review Test Code:**
    *   Are there any implicit timing assumptions (e.g., `Thread.Sleep`)?
    *   Is there shared mutable state between tests?
    *   Are assertions waiting for eventual consistency (e.g., polling, Awaitility)?
*   [ ] **External Dependencies:** Is the flakiness related to an external service (database, message broker, API call)?
*   [ ] **Test Environment:** Are there differences between local and CI environments that could cause flakiness?
*   [ ] **Logging/Tracing:** Add more detailed logging or tracing to pinpoint the failure point during a flaky run.
*   [ ] **Isolation:** Try running the flaky test in isolation, and then within its class/project, to see if order or concurrency matters.

#### 6.4. Event Contract Change Review Checklist

*   [ ] **`Shared/` Impact:** Is the change in `Shared/` (e.g., event DTO)?
*   [ ] **Producers Identified:** Which services produce this event? Are their tests updated to reflect the new contract?
*   [ ] **Consumers Identified:** Which services consume this event? Are their tests updated to handle the new contract?
*   [ ] **Backward Compatibility:** Is the change backward-compatible? If not, is there a migration strategy and robust versioning?
*   [ ] **Integration Tests:** Do integration tests verify the end-to-end flow with the new event contract?
*   [ ] **Schema Validation:** If schema validation is in place, is the schema updated and tested?