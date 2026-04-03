# Tech Stack

This section outlines the complete planned technology stack for the `event-driven-architecture` system, categorizing tools and platforms by their primary function.

## Languages

| Technology | Purpose                                                    | Status           |
| :--------- | :--------------------------------------------------------- | :--------------- |
| C#         | Primary programming language for all services and libraries. | FACT (Confirmed) |

## Frameworks

| Technology           | Purpose                                                                                                     | Status           |
| :------------------- | :---------------------------------------------------------------------------------------------------------- | :--------------- |
| .NET                 | Core framework for building all applications and services.                                                  | FACT (Confirmed) |
| ASP.NET Core         | Framework for building RESTful APIs for services like `AccountService`, `InventoryService`, and `OrchestratorService`. | HYPOTHESIS (High) |
| .NET Worker Services | Framework for building long-running background services, such as `OutboxProcessor` and `ClaimCheckProcessor`. | FACT (Confirmed) |

## Databases & Storage

| Technology          | Purpose                                                                                | Status           |
| :------------------ | :------------------------------------------------------------------------------------- | :--------------- |
| Relational Database | Persistent storage for service-specific data (e.g., accounts, inventory, order states) and transactional outbox entries. | HYPOTHESIS (High) |
| Blob/Object Storage | Stores large message payloads as part of the Claim Check Pattern.                      | HYPOTHESIS (High) |

## Messaging & External Services

| Technology    | Purpose                                                                                    | Status           |
| :------------ | :----------------------------------------------------------------------------------------- | :--------------- |
| Message Broker| Facilitates asynchronous, event-driven communication between microservices, handles event publishing and consumption. | FACT (Confirmed) |

## Infrastructure & Operations

| Technology              | Purpose                                                                                                 | Status                       |
| :---------------------- | :------------------------------------------------------------------------------------------------------ | :--------------------------- |
| Docker                  | Containerization platform for packaging services, ensuring consistent environments across development and deployment. | FACT (Confirmed)             |
| Container Orchestrator  | **Planned:** Manages the deployment, scaling, and operational lifecycle of containerized services in production. | HYPOTHESIS (Low - *Planned*) |
| CI/CD Platform          | **Planned:** Automates the build, test, and deployment processes for all services and infrastructure changes. | HYPOTHESIS (Low - *Planned*) |
| Infrastructure as Code (IaC) | **Planned:** Manages and provisions infrastructure through code, ensuring consistency and repeatability. | HYPOTHESIS (Low - *Planned*) |

## Build Tools & Package Management

| Technology    | Purpose                                                           | Status           |
| :------------ | :---------------------------------------------------------------- | :--------------- |
| .NET SDK      | Command-line tools for building, running, and managing .NET projects. | FACT (Confirmed) |
| NuGet         | Standard package manager for .NET, handling third-party library dependencies. | FACT (Confirmed) |