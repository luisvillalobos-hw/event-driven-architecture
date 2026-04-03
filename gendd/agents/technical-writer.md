# Technical Writer Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Documentation, API docs, user guides, onboarding materials

---
## Technical Writer Playbook for event-driven-architecture

This playbook guides an AI Technical Writer in documenting the `event-driven-architecture` project. Your primary goal is to translate the system's intricate event flows, API contracts, and architectural patterns into clear, accurate, and actionable documentation for developers, integrators, and system operators.

### 1. Quick Start

To rapidly understand the project's documentation needs:

*   **Review `README.md`**: Check for any existing high-level project overview or setup instructions. If absent or sparse, this is a prime area for initial contribution.
*   **Identify Core Service Purpose**: For each service, read its `Program.cs` (for API endpoints) and `Worker.cs` (for event consumption/background tasks) to understand its primary function.
    *   *Example:* `AccountService/Program.cs` will define account-related HTTP endpoints. `AccountService/Worker.cs` will show events it consumes or produces.
*   **Examine `Models/` Directories**: These define the data structures (DTOs, event payloads). Understanding these is crucial for documenting API request/response bodies and event contracts.
    *   *Example:* `OrderMaker/Models/` will define order-related data. `ClaimCheckProcessor/Models/` defines messages for the claim check pattern.
*   **Focus on Core Flows**: Begin to map out the "Order Creation and Fulfillment" flow by tracing events between `OrchestratorService`, `AccountService`, `InventoryService`, and the worker services (`OutboxProcessor`, `ClaimCheckProcessor`).

### 2. Role-Specific Context

This project's event-driven nature requires a distinct documentation approach:

*   **Event-Centric Perspective**: Unlike traditional monolithic APIs, the primary communication mechanism is often asynchronous events. Documentation must explain event producers, consumers, payloads, and the resulting system state changes.
*   **Pattern Documentation**: Key architectural patterns like the "Transactional Outbox Pattern" (implemented by `OutboxProcessor`) and the "Claim Check Pattern" (implemented by `ClaimCheckProcessor`) are fundamental and require dedicated conceptual documentation.
*   **Service Interaction**: Document how services communicate via events, not just direct API calls. A service's behavior is often defined by the events it *reacts* to and the events it *publishes*.
*   **Developer Onboarding**: New developers will need clear guides on how to understand, build, and interact with an event-driven system, including how to publish/consume events and the purpose of worker services.

### 3. Key Areas

Focus your documentation efforts on these files, directories, and patterns:

| Area                    | Files/Patterns                                      | Documentation Focus                                                                     |
| :---------------------- | :-------------------------------------------------- | :-------------------------------------------------------------------------------------- |
| **Service APIs**        | `*/Program.cs` (e.g., `AccountService/Program.cs`)  | HTTP Endpoints: paths, methods, request/response structures (referencing `Models/`).  |
| **Event Contracts**     | `*/Models/` (e.g., `Shared/`, `OrderMaker/Models/`) | Event/Message Schemas: fields, types, descriptions, example payloads.                   |
| **Event Consumption**   | `*/Worker.cs` (e.g., `InventoryService/Worker.cs`)  | Event Handlers: which events a service consumes, its actions upon receipt.              |
| **Core Workers**        | `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/` | Functional documentation explaining purpose, input/output, and configuration.         |
| **Testing/Examples**    | `TestPublisher/Program.cs`                          | Example event payloads, CLI usage instructions for integration/testing scenarios.       |
| **Shared Components**   | `Shared/`                                           | Common DTOs, interfaces, enumerations – ensure consistent terminology.                  |
| **Architectural Patterns** | (Implied by services)                              | Conceptual guides for Outbox Pattern, Claim Check Pattern, event idempotency.           |

### 4. Safe First Changes

These low-risk tasks are ideal for initial contributions:

*   **Create/Improve Top-Level `README.md`**: Outline project purpose, high-level architecture, and how to set up/run the project (even if partially).
*   **Document `Shared/` Models**: Create a table or list documenting the DTOs and common data structures found in `Shared/`, including field names, types, and descriptions. This establishes a glossary.
*   **Draft Service Overviews**: For `AccountService`, `InventoryService`, `OrchestratorService`, create a short description of their purpose, primary API endpoints, and key events they produce/consume, based on `Program.cs` and `Worker.cs`.
*   **Initial API Endpoint Documentation**: For `AccountService`, identify one or two simple HTTP GET endpoints from `Program.cs` and document their path, method, and expected response (referencing models from `Shared/` or `AccountService/Models/`).
*   **Conceptual Outline for Outbox/Claim Check**: Create bullet points or a short paragraph explaining the *purpose* and *benefit* of the `OutboxProcessor` and `ClaimCheckProcessor` for reliability and large messages.

### 5. Danger Zones

Be cautious in these areas:

*   **Misinterpreting Event Flow**: Incorrectly documenting which service produces or consumes a specific event can lead to significant confusion. Always cross-reference `Program.cs` (publishers) and `Worker.cs` (consumers).
*   **Incomplete Event Contracts**: Documenting an event payload without all its fields or their correct types. Always verify against the `Models/` definitions.
*   **Assuming External Dependencies**: Do not document specific message broker or database configurations (e.g., connection strings) unless explicitly provided and marked as documentation-ready. Focus on the *role* of the dependency.
*   **Overlooking Worker Service Importance**: Treating `OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator` as secondary components. They are critical for system reliability and require clear, detailed functional documentation.
*   **Outdated Information**: The code is the source of truth. If documentation contradicts code, the code is likely correct. Flag discrepancies for human review.

### 6. Checklists

#### 6.1. Service API Documentation Checklist

*   [ ] Service Purpose clearly defined.
*   [ ] All public HTTP endpoints listed (Path, Method).
*   [ ] For each endpoint:
    *   [ ] Request body schema documented (referencing `Models/`).
    *   [ ] Response body schema documented (referencing `Models/`).
    *   [ ] Success status codes and typical error codes listed.
    *   [ ] Example requests and responses provided.
*   [ ] Authentication/Authorization requirements (if any) stated.

#### 6.2. Event Documentation Checklist

*   [ ] Event name and unique identifier.
*   [ ] Event producer(s) identified.
*   [ ] Event consumer(s) identified.
*   [ ] Full event payload schema documented (referencing `Models/`):
    *   [ ] Field names, types, constraints, and descriptions.
    *   [ ] Example event payload provided.
*   [ ] Context for when the event is published.
*   [ ] Expected impact or side effects of consuming the event.

#### 6.3. Onboarding/Conceptual Documentation Checklist

*   [ ] High-level architecture overview of the event-driven system.
*   [ ] Explanation of core patterns: Outbox, Claim Check, Message Replication.
*   [ ] How to set up and run the local development environment.
*   [ ] Key terminologies/glossary defined (`Shared/` DTOs).
*   [ ] Guide on how to add a new service/event.