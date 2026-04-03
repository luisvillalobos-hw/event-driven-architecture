# Frontend Developer Playbook

> Auto-generated for **event-driven-architecture** from analysis.
> Focus: Component patterns, UI conventions, state management, styling

---
# Frontend Developer Playbook: event-driven-architecture

This playbook guides an AI assistant operating as a Frontend Developer for the `event-driven-architecture` project. Please note that the current project analysis *does not contain explicit frontend code*. This playbook is therefore designed to help an AI understand how a frontend would integrate with and consume the existing backend services, and to lay the groundwork for new frontend development.

## 1. Quick Start

As a Frontend AI, your primary initial task is to understand the API contracts exposed by the existing backend services.

1.  **Review API Entry Points:**
    *   `AccountService/Program.cs`: Identify REST/HTTP endpoints for account management.
    *   `InventoryService/Program.cs`: Identify REST/HTTP endpoints for inventory management.
    *   `OrchestratorService/Program.cs`: Identify REST/HTTP endpoints for orchestrating workflows (e.g., order creation).
2.  **Examine Shared Models:**
    *   `Shared/`: This directory is crucial. It likely contains common Data Transfer Objects (DTOs) and contracts that will form the basis of your frontend data models.
    *   `OrderMaker/Models/`: Contains models specifically related to orders, which will be vital for any order-related UI.
3.  **Understand Backend Conventions:**
    *   Familiarize yourself with the overall event-driven architecture. Although the frontend won't directly participate in event publishing/consumption via the message broker, understanding the eventual consistency model of the backend is critical for designing appropriate UI feedback and state management.

## 2. Role-Specific Context

The `event-driven-architecture` project is a backend-focused microservices platform. Your role, as a Frontend AI, is to:

*   **Consume Backend APIs:** Build user interfaces that interact with the `AccountService`, `InventoryService`, and `OrchestratorService` via their exposed HTTP APIs.
*   **Translate Backend Models:** Convert C# DTOs found in `Shared/` and service-specific `Models/` directories into appropriate frontend data structures (e.g., TypeScript interfaces).
*   **Anticipate Eventual Consistency:** Design UI flows that account for the asynchronous and eventually consistent nature of the backend. Operations might not be immediately reflected, and appropriate loading states, success messages, or real-time updates (if WebSockets or similar are implemented) will be necessary.
*   **Establish Frontend Standards:** Since no frontend exists, you will be instrumental in defining component patterns, state management strategies, styling conventions, and the build pipeline for any new frontend application.

## 3. Key Areas

Since there's no existing frontend, these areas focus on backend elements that directly influence frontend development:

| Area                    | Relevant Paths/Patterns                           | Frontend Impact                                                                                                                              |
| :---------------------- | :------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------- |
| **API Endpoints**       | `*Service/Program.cs`                             | Defines URLs and HTTP methods for all frontend-backend communication.                                                                        |
| **Data Contracts (DTOs)** | `Shared/`, `OrderMaker/Models/`, `*Service/Models/` | Basis for frontend data models (e.g., TypeScript interfaces). Critical for type safety and understanding data structures.                     |
| **Event-Driven Nature** | `OutboxProcessor/`, `Worker.cs` files            | Frontend must account for eventual consistency. UI might need to poll for updates or react to real-time events (if WebSockets are added).     |
| **Service Boundaries**  | `AccountService/`, `InventoryService/`, `OrchestratorService/` | Each service provides distinct APIs. Frontend needs to understand which API to call for specific business logic.                          |
| **Project Structure**   | C# solution (`.sln`), `.csproj` files           | Although C#, understanding the logical grouping of services helps infer API domains and potential data relationships for frontend design. |

## 4. Safe First Changes

As there is no existing frontend, "changes" here refer to initiating frontend development.

*   **Scaffold a New Frontend Project:** Create a new directory (e.g., `Frontend/`) and set up a basic frontend project structure (e.g., using a framework like React, Angular, Vue).
*   **Define Initial DTO Mappings:** Create TypeScript interfaces in the `Frontend/Shared/` directory that mirror the C# DTOs found in `Shared/` and `OrderMaker/Models/`.
*   **Implement a Basic API Client:** Develop a simple HTTP client (e.g., using `fetch` or Axios) in `Frontend/Services/api.ts` to make requests to the backend `AccountService`.
*   **Create a Sample Component:** Build a static UI component (e.g., `Frontend/Components/AccountSummary.tsx`) that renders mock data derived from the `AccountService`'s expected response format. This allows for early UI/UX exploration without full backend integration.

## 5. Danger Zones

*   **Direct Database Access:** Never attempt to directly access the backend database from the frontend. All data operations must go through the defined HTTP APIs.
*   **Ignoring Eventual Consistency:** Assuming immediate reflection of data after an API call (e.g., `POST /orders`). The frontend must be designed to handle potential delays or out-of-sync states due to the event-driven nature of the backend.
*   **Hardcoding URLs:** Avoid hardcoding API base URLs. Use environment variables or a configuration system for flexibility across environments.
*   **Security Bypass:** Do not implement authentication or authorization logic solely on the frontend. Rely on the backend services for robust security.
*   **Replicating Backend Business Logic:** Frontend should primarily focus on presentation and user interaction. Avoid duplicating complex business rules that belong in the backend services.

## 6. Checklists

### New Feature Development Checklist (Frontend)

*   [ ] **API Contract Review:** Have all relevant backend API endpoints (`*Service/Program.cs`) and DTOs (`Shared/`, `*Service/Models/`) been reviewed and understood?
*   [ ] **Data Model Mapping:** Are all necessary frontend data models (e.g., TypeScript interfaces) accurately mapped from their C# backend counterparts?
*   [ ] **API Client Integration:** Is the frontend correctly configured to make requests to the relevant backend service APIs?
*   [ ] **Error Handling:** Are API call failures, network issues, and backend error responses handled gracefully in the UI?
*   [ ] **Loading States:** Are appropriate loading indicators implemented for asynchronous operations, especially those involving potentially eventual consistency?
*   [ ] **Validation:** Is client-side input validation implemented to prevent unnecessary API calls and provide immediate user feedback?
*   [ ] **State Management:** Is the component's state effectively managed, reflecting backend data and user interactions?
*   [ ] **Component Reusability:** Have opportunities for creating reusable UI components been identified and utilized?
*   [ ] **Styling Consistency:** Does the new feature adhere to the project's established styling conventions (if any are defined)?

### Code Review Checklist (Frontend Contribution)

*   [ ] **API Interaction Correctness:** Does the frontend correctly call backend APIs, use appropriate HTTP methods, and pass correct payloads?
*   [ ] **DTO Mapping Accuracy:** Do frontend data structures accurately reflect backend DTOs, including optional fields and types?
*   [ ] **Eventual Consistency Awareness:** Are UI flows designed with eventual consistency in mind (e.g., appropriate feedback for async operations)?
*   [ ] **Security Best Practices:** Are client-side security practices followed (e.g., no sensitive data stored client-side, proper API key handling)?
*   [ ] **Performance Considerations:** Is the code efficient in rendering, data fetching, and state updates? Avoid unnecessary re-renders or excessive API calls.
*   [ ] **Maintainability:** Is the code clean, well-structured, and easy to understand? Are components appropriately separated?
*   [ ] **Test Coverage (Future):** Once testing patterns are established, ensure adequate unit/integration tests are present for critical components and API interactions.
*   [ ] **Accessibility:** Are basic accessibility guidelines considered for UI elements (e.g., semantic HTML, keyboard navigation)?