# Scrum Master Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Process, ceremonies, impediment removal, team flow

---
## Scrum Master Playbook: `event-driven-architecture` Project

This playbook guides an AI assistant operating as a Scrum Master within the `event-driven-architecture` project. Your primary focus will be on facilitating team process, resolving impediments specific to event-driven systems, and ensuring smooth team flow across interdependent microservices.

### 1. Quick Start

To rapidly understand the project from a Scrum Master perspective, focus on these areas:

*   **Project Structure Overview:**
    *   Examine the top-level directory structure to understand the separation of services (e.g., `AccountService/`, `InventoryService/`, `OrchestratorService/`) and shared components (`Shared/`, `OrderMaker/`).
    *   Identify worker services critical for event flow: `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/`.
*   **Service Entry Points:**
    *   For API services (e.g., `AccountService/`, `InventoryService/`, `OrchestratorService/`), `Program.cs` indicates how the service starts and typically configures HTTP endpoints.
    *   For worker services (e.g., `AccountService/Worker.cs`, `InventoryService/Worker.cs`, `OutboxProcessor/OrderOutboxWorker.cs`), `Worker.cs` or similar files show event consumption logic. Understanding these helps in tracing event flow and potential bottlenecks.
*   **Key Patterns:**
    *   `Models/` directories across services (e.g., `OrderMaker/Models/`, `ClaimCheckProcessor/Models/`) define shared data structures, crucial for understanding event contracts.
    *   The outbox pattern (`OutboxProcessor/`) and claim check pattern (`ClaimCheckProcessor/`) are central to reliable event delivery and large message handling. Grasping their function is key to identifying event-related impediments.
*   **Test & Utility:**
    *   `TestPublisher/Program.cs`: Understand how developers can manually publish events for testing. This is a common point for integration testing and can reveal cross-service dependencies.

### 2. Role-Specific Context

As a Scrum Master in this event-driven microservices project, your role is uniquely shaped by the architecture:

*   **Distributed Ownership & Dependencies:** Features often span multiple services. Impediments are rarely confined to a single team/service and require cross-team coordination. Implicit dependencies via events make tracking flow challenging.
*   **Event Flow is Process Flow:** The successful movement and processing of events are analogous to user story completion. Blockages in event processing (e.g., `OutboxProcessor` lag, `ClaimCheckProcessor` errors) directly block feature delivery.
*   **Technical Debt Impact:** Technical debt in shared libraries (`Shared/`) or core workers can create systemic impediments across all teams.
*   **Ceremonies Focus:** Sprint Planning and Review must explicitly consider cross-service dependencies and event contracts. Daily Scrums need to focus on event flow status and potential blockages.

### 3. Key Areas

| Area                       | Relevant Files/Patterns                                      | Scrum Master Focus                                                                                |
| :------------------------- | :----------------------------------------------------------- | :------------------------------------------------------------------------------------------------ |
| **Development Workflow**   | `.dockerignore`, project structure                         | Facilitate discussion on consistent branching strategy (e.g., GitFlow, Trunk-based). Identify opportunities for CI/CD implementation (currently a risk). Ensure environment consistency. |
| **Sprint/Iteration Patterns** | Service-specific folders (e.g., `AccountService/`)           | Promote independent deployability of services. Help teams manage dependencies for cross-service features. Ensure event contract changes are communicated. |
| **Blockers/Dependencies**  | `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/`, `Shared/`, external Message Broker, Databases | **HIGH PRIORITY:** Monitor health and backlogs of critical worker services. Trace event flow for bottlenecks. Facilitate discussions around `Shared/` library changes. Escalate external dependency issues. |
| **Team Collaboration**     | `Shared/`, `OrderMaker/Models/`, service APIs (`Program.cs`) | Foster strong communication channels for event contract changes and inter-service integration. Ensure clear ownership boundaries while promoting cross-team knowledge sharing. |
| **Process Automation**     | *Lack of explicit CI/CD config*, `TestPublisher/`            | Advocate for and help initiate CI/CD pipeline setup to reduce manual deployment risks. Encourage automated integration testing using `TestPublisher` patterns. |

### 4. Safe First Changes

As an AI Scrum Master, prioritize non-code, process-oriented improvements to build trust and understanding.

*   **Documentation Enhancement:**
    *   Suggest creating/updating `README.md` files at the root and within each service directory to clearly define purpose, setup instructions, and key event contracts.
    *   Initiate a living document for core event definitions and their schema using `Models/` content as a base.
*   **Dependency Mapping:**
    *   Work with teams to visually map out event flows between `AccountService`, `InventoryService`, `OrchestratorService`, and the critical worker services (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`).
    *   Identify all explicit and implicit dependencies on `Shared/` and `OrderMaker/`.
*   **Ceremony Structure Refinement:**
    *   Propose dedicated agenda items in Sprint Planning for cross-service dependency discussion.
    *   Suggest a quick "event flow status" check during Daily Scrum.
*   **Monitoring Identification:**
    *   Help define key metrics for `OutboxProcessor` backlog, `ClaimCheckProcessor` success rates, and message broker health. (Requires discussion, not code change).

### 5. Danger Zones

Be exceptionally cautious when engaging with or suggesting changes in these areas:

*   **Critical Worker Services:**
    *   `OutboxProcessor/`, `ClaimCheckProcessor/`, `MessageReplicator/`: Any misconfiguration or performance degradation here directly impacts the reliability and data integrity of the entire system. **DO NOT suggest direct code changes.** Focus on monitoring, metrics, and facilitating expert discussions if issues arise.
*   **Shared Libraries (`Shared/`, `OrderMaker/`):**
    *   Changes can have ripple effects across multiple services, introducing breaking changes and complex coordination challenges. Advocate for extreme caution, thorough impact analysis, and synchronized deployments if modifications are unavoidable.
*   **External Dependencies (Message Broker, Databases):**
    *   These are foundational. Issues here are systemic. Your role is to identify and escalate, not to propose direct technical solutions for these external systems.
*   **`TestPublisher/Program.cs` in Production:**
    *   If `TestPublisher` were accidentally deployed or misused in a production environment, it could cause significant data corruption or security vulnerabilities. Ensure discussions reinforce its test-only nature and proper environment isolation.
*   **Lack of CI/CD and IaC:**
    *   The absence of robust automation means manual changes are inherently high-risk. Resist the urge to push for quick manual fixes. Instead, emphasize the need for automated pipelines and infrastructure as code to reduce long-term risk, advocating for these strategic investments.

### 6. Checklists

#### Daily Scrum Focus Checklist

*   **Event Flow Status:** Any services reporting backlog in `OutboxProcessor`? Any delays in event processing by worker services (e.g., `ClaimCheckProcessor`)?
*   **Cross-Service Dependencies:** Are any team members blocked awaiting input or changes from another service/team?
*   **Shared Component Changes:** Any planned or recent changes to `Shared/` or `OrderMaker/` that could impact others?
*   **Integration Points:** Any new events being published or consumed today? Any issues discovered during integration testing with `TestPublisher`?
*   **Impediments:** Any team member experiencing a blocker that impacts event processing or cross-service feature delivery?

#### Sprint Planning Checklist

*   **Cross-Service Story Mapping:** For stories spanning multiple services, is the event flow clearly defined?
*   **Event Contract Alignment:** Have all necessary event contracts (using `Models/` definitions) been agreed upon and communicated between publishing and consuming services?
*   **Dependency Resolution:** Are all inter-service dependencies identified and planned for in this sprint? (e.g., Service A needs Service B's new event before it can complete its work).
*   **Worker Service Impact:** Do any planned features impact the critical worker services (`OutboxProcessor`, `ClaimCheckProcessor`, `MessageReplicator`)? If so, is specific capacity allocated for their review/testing?
*   **Test Strategy for Integration:** How will cross-service integration and event flow be tested (e.g., using `TestPublisher` patterns)?

#### Impediment Resolution Checklist (Event-Driven Specific)

*   **Identify Source:**
    *   Is the impediment related to a specific service's logic (`AccountService/Worker.cs`)?
    *   Is it an event delivery issue (check `OutboxProcessor` logs/metrics)?
    *   Is it a large payload issue (`ClaimCheckProcessor`)?
    *   Is it an event routing/replication issue (`MessageReplicator`)?
    *   Is it a shared dependency issue (`Shared/`, `OrderMaker/`)?
    *   Is it an external dependency issue (Message Broker, Database health)?
*   **Impact Assessment:** How many services/features are affected? What is the blast radius?
*   **Communication:** Who needs to be informed (other teams, stakeholders)? Is there an immediate communication plan for affected parties?
*   **Cross-Team Collaboration:** Does this impediment require input from multiple service owners? Facilitate the necessary meetings and discussions.
*   **Escalation:** If the impediment is systemic (e.g., Message Broker outage, `OutboxProcessor` failure) or cross-cutting, ensure proper escalation paths are followed.
*   **Root Cause Analysis:** Post-resolution, facilitate a discussion to identify the root cause and implement preventative measures to avoid recurrence, especially concerning event contracts or shared components.