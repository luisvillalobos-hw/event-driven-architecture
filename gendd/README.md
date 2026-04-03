# event-driven-architecture — GenDD Analysis

This system appears to be an event-driven microservices platform designed for processing and orchestrating business workflows, likely in an e-commerce or similar domain, involving account management, inventory tracking, and order fulfillment. It emphasizes reliable event publishing through an outbox pattern and handles large messages with a claim check processor.

## Contents

| Directory | Description |
|-----------|-------------|
| `overview/` | System summary and tech stack |
| `architecture/` | Architecture overview, components, data ownership |
| `flows/` | Core flow documentation |
| `risks/` | Risk hotspots and open questions |
| `onboarding/` | Developer onboarding guide |
| `pass1-scan-findings.md` | Raw scan findings |
| `pass2-infer-findings.md` | LLM inference findings |
| `pass3-validation-packet.md` | Validation questions |
| `pass3-validation-responses.md` | Human validation responses |

## Architecture

**Pattern:** event-driven (high confidence)

**Tech Stack:** C#, .NET (implied by .csproj and .sln files), Docker (from .dockerignore), Message Broker (inferred from 'event-driven-architecture', OutboxProcessor, MessageReplicator, Worker.cs), Relational Database (inferred from OutboxProcessor and general microservice data storage needs), Blob/Object Storage (inferred from ClaimCheckProcessor)
