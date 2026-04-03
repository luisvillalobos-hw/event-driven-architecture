## Component Catalog

This section provides a catalog of all planned services and modules within the `event-driven-architecture` system, detailing their type, purpose, key components, responsibilities, dependencies, and data ownership.

### AccountService
- **Type**: API Service (FACT (Confirmed))
- **Purpose**: Manages user accounts, provides API endpoints, and processes account-related events. (FACT (Confirmed))
- **Key Components**: `AccountService/Program.cs` (API Entry), `AccountService/Worker.cs` (Event Consumer), Internal business logic & models (HYPOTHESIS (Confidence: High))
- **Responsibilities**: User account CRUD operations, handling account-specific events (e.g., charging), publishing account-domain events. (HY