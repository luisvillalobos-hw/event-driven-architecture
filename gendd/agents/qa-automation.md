# QA Automation Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Test frameworks, CI integration, coverage, reliability

---
## QA Automation Playbook for event-driven-architecture

This playbook outlines the responsibilities and focus areas for an AI assistant acting as a QA Automation specialist within the `event-driven-architecture` project. Your primary goal is to ensure the reliability, correctness, and performance of event-driven flows and microservices, with a strong emphasis on automation.

### 1. Quick Start

To effectively contribute, begin by understanding the core event flows and the existing test infrastructure.

*   **1.1 Understand the Core Flow:**
    *   Review `OrchestratorService/Program.cs` and `OrchestratorService/Worker.cs` to understand order orchestration.
    *   Examine `OutboxProcessor/OrderOutboxWorker.cs` to grasp reliable event publishing.
    *   Analyze `TestPublisher/Program.cs` to see how test events are manually initiated – this is a good starting point for understanding event structures.
    *   Map the "Order Creation and Fulfillment" core flow: Trace how events flow from initiation (e.g., `OrchestratorService` API) through the outbox, to other services, and finally through workers like `ClaimCheckProcessor` if large messages are involved.
*   **1.2 Locate Test Projects:**
    *   Search for directories named `*.Tests/` (e.g., `AccountService.Tests/`, `OrchestratorService.Tests/`) or a common `Tests/` folder at the root.
    *   If no explicit test projects are found, assume the standard .NET pattern and prepare to create them, typically named `[ServiceOrLibraryName].Tests.csproj`.
*   **1.3 Execute Existing Tests (If Any):**
    *   Navigate to the solution root.
    *   Run `dotnet test` to discover and execute all tests.
    *   Identify any failing or consistently slow tests.
*   **1.4 Set up Local Environment:**
    *   Ensure Docker is running to support dependent services (e.g., message broker, database).
    *   Investigate if a `docker-compose.yml` file exists to spin up the entire system. If not, note this as a critical gap for local testing.

### 2. Role-Specific Context

Your role is critical in validating the complex, asynchronous interactions inherent in an event-driven system.

*   **Event Contract Validation:** Event schemas (likely in `Shared/` or `OrderMaker/Models/`) are your most important contracts. Any change requires validation across all producers and consumers.
*   **Asynchronous Assertions:** Traditional synchronous testing patterns are insufficient. You will frequently need to assert eventual consistency, waiting for events to propagate and effects to materialize.
*   **Worker Service Reliability:** The `OutboxProcessor`, `ClaimCheckProcessor`, and `MessageReplicator` are foundational. Tests must cover their resilience to transient failures, message re-processing, and correct state transitions.
*   **Integration Testing Focus:** Given the microservice architecture, integration tests that span multiple services and event flows are paramount. Unit tests are valuable, but the system's correctness is proven at the integration level.
*   **Idempotency:** Messages can be redelivered. Your tests must confirm that processing the same event multiple times does not lead to incorrect state changes.

### 3. Key Areas

| Category                | Files/Patterns                                                | Focus for QA Automation                                                                                                      |
| :---------------------- | :------------------------------------------------------------ | :--------------------------------------------------------------------------------------------------------------------------- |
| **Test Frameworks**     | `*.Tests.csproj` (e.g., `AccountService.Tests/`), `Using NUnit/XUnit/MSTest;` | Identify and standardize the chosen framework. Ensure consistent reporting. Explore integration with assertion libraries (e.g., FluentAssertions) and mocking frameworks (e.g., Moq). |
| **Test Organization**   | `[Service]/Tests/Unit/`, `[Service]/Tests/Integration/`, `Tests/EndToEnd/` | Enforce clear separation of unit, integration, and end-to-end tests. Integration tests should mirror event flow paths.                                |
| **CI/CD Integration**   | (Likely `azure-pipelines.yml`, `.github/workflows/`, Jenkinsfile) | Ensure `dotnet test` runs automatically on every commit/PR. Configure test reporting and artifact publishing.                                 |
| **Coverage Gaps**       | `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/` | Critical worker services often lack comprehensive integration tests. Error handling paths, message deserialization failures, and retry logic are common gaps. |
| **Flaky Test Patterns** | Asynchronous assertions, timing-dependent tests, shared resources. | Identify tests that fail intermittently. Implement robust retry mechanisms in tests, use message IDs for idempotency checks, and avoid relying on strict timing. |
| **Event Contracts**     | `Shared/`, `OrderMaker/Models/`                              | Tests must validate event serialization/deserialization and schema evolution. Consider consumer-driven contract testing.               |

### 4. Safe First Changes

*   **4.1 Document Existing Tests:** Create a markdown file (`docs/Tests.md`) outlining where tests are, what they cover, and how to run them.
*   **4.2 Enhance `TestPublisher` Usage:**
    *   Create a dedicated integration test project (e.g., `IntegrationTests/`).
    *   Add a test that uses `TestPublisher` to send a known event, then asserts the eventual state of a downstream service (e.g., check `InventoryService` after an order event). This establishes a baseline for async testing.
*   **4.3 Introduce Basic CI Reporting:**
    *   If a CI pipeline exists, ensure `dotnet test --logger "trx;LogFileName=test-results.trx"` is used to generate test reports.
    *   Configure the CI system to publish these `.trx` files as test artifacts.
*   **4.4 Add a Health Check Integration Test:**
    *   For `AccountService`, `InventoryService`, and `OrchestratorService`, add a simple integration test that calls a `/health` endpoint or a basic API endpoint to ensure the service starts successfully and can connect to its dependencies.

### 5. Danger Zones

*   **5.1 Modifying Worker Service Tests (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`):** These services are critical for reliability. Changes to their tests (or adding new ones) require deep understanding of their asynchronous nature and potential impact on event flow. Verify side effects thoroughly.
*   **5.2 Introducing Timing-Dependent Assertions:** Avoid `Thread.Sleep()` or hardcoded delays in tests. Use robust polling mechanisms with timeouts (e.g., `Polly` for retries, `FluentAssertions.Eventually()` for async waits) to check for eventual consistency.
*   **5.3 External Dependencies in Integration Tests:** Direct interaction with actual message brokers or databases in tests can introduce flakiness and slow execution. Prefer test doubles where appropriate, but ensure critical integration paths use real dependencies in dedicated integration tests. Avoid shared state between integration tests.
*   **5.4 `TestPublisher` in Production:** While low risk for QA, be aware of the "Risk Hotspot" regarding `TestPublisher`. Do not deploy or encourage its use in production environments.
*   **5.5 Coverage of Error Scenarios:** Testing how the system reacts to message deserialization failures, database connection issues, or upstream service timeouts is crucial but complex. Incorrectly simulating these can lead to false positives or system instability.

### 6. Checklists

#### 6.1 New Feature/Service QA Automation Review

*   [ ] Does the new feature/service have a dedicated `*.Tests.csproj`?
*   [ ] Are unit tests covering complex business logic and edge cases?
*   [ ] Are integration tests covering event publishing and consumption paths?
*   [ ] Does the integration test suite validate the end-to-end flow of key events?
*   [ ] Are asynchronous assertions used appropriately for event-driven interactions?
*   [ ] Is idempotency of event handlers verified?
*   [ ] Does the test suite cover error handling and resilience scenarios (e.g., invalid events, transient dependency failures)?
*   [ ] Are new event contracts defined in `Shared/` or `OrderMaker/Models/` properly validated (serialization/deserialization)?
*   [ ] Are tests clearly organized into `Unit/`, `Integration/`, `EndToEnd/` categories?

#### 6.2 Pull Request (PR) Test Review

*   [ ] Do new/modified tests adhere to existing naming conventions and organization?
*   [ ] Do new tests fail before the change and pass after (if applicable)?
*   [ ] Are any existing tests becoming flaky due to the change?
*   [ ] Are the tests readable, maintainable, and self-documenting?
*   [ ] Are test dependencies (e.g., mocks, test data) isolated and managed effectively?
*   [ ] For event-related changes, are *all* impacted producers and consumers considered in the test scope?
*   [ ] Is the PR introducing or fixing any known flaky tests?
*   [ ] Is the test coverage for modified code adequate, especially for critical paths and error handling?