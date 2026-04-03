## Order Creation and Fulfillment Flow Documentation

### ASCII Sequence Diagram

```
External Client  OrchestratorSvc  OutboxProcessor  Message Broker  InventorySvc  AccountSvc
       |               |                |               |             |             |
       |  1. OrderReq  |                |               |             |             |
       |-------------->|                |               |             |             |
       |               | 2. Save state, |               |             |             |
       |               |    Publish OE  |               |             |             |
       |               |<---------------|               |             |             |
       |               |                |               |             |             |
       |               |                | 3. Detect OE  |             |             |
       |               |                |<------------- |             |             |
       |               |                | 4. Publish OE |             |             |
       |               |                |-------------->|             |             |
       |               |                |               |             |             |
       |               |                |               | 5. Consume OE|             |
       |               |                |               |<------------|             |
       |               |                |               |  Check/Reserve |             |
       |               |                |               |  Publish IE/IF |             |
       |               |                |               |<--------------------------|
       |               |                |               |             |             |
       |               |                | 6. Detect IE  |             |             |
       |               |                |<------------- |             |             |
       |               |                | 7. Publish IE |             |             |
       |               |                |-------------->|             |             |
       |               |                |               |             | 8. Consume IE|
       |               |                |               |             |<------------|
       |               |                |               |             | Charge/AF    |
       |               |                |               |             | Publish ACE/AFE|
       |               |                |               |             |<--------------------------|
       |               |                |               |             |             |
       |               |                | 9. Detect ACE |             |             |
       |               |                |<------------- |             |             |
       |               |                | 10. Publish ACE|             |             |
       |               |                |-------------->|             |             |
       |               | 11. Consume IE/ACE             |             |             |
       |               |<-------------------------------|             |             |
       |               |     Update Order Status        |             |             |
       |               |--------------------------------|             |             |
       |               |                |               |             |             |
       | (Optional)    | 12. Claim Check|               |             |             |
       |               |<---------------|               |             |             |
       | (Optional)    |                |               |             |             |
       |               |                | 13. Replicate |             |             |
       |               |                |-------------->| Analytics / Etc.
```

### Step-by-Step Flow Documentation

1.  **External client makes API call**
    *   **Planned:** External client or internal process calls `OrchestratorService/Program.cs` to request a new order.
    *   **Error Handling:** (Assumed) Client retries on connection issues. `OrchestratorService` handles malformed requests (400 Bad Request).

2.  **`OrchestratorService` processes request**
    *   **Planned:** Validates request, saves initial order state to its database, and publishes an 'OrderRequested' event to its outbox.
    *   **File:** `OrchestratorService/Program.cs`
    *   **Error Handling:** (Assumed) Validation failures block saving. Database transaction ensures atomicity of order save and outbox write. Retries on transient DB errors.

3.  **`OutboxProcessor` detects 'OrderRequested' event**
    *   **Planned:** `OutboxProcessor/OrderOutboxWorker.cs` polls/detects 'OrderRequested' from `OrchestratorService`'s outbox.
    *   **File:** `OutboxProcessor/OrderOutboxWorker.cs`
    *   **Error Handling:** (Assumed) Resilient to temporary DB unavailability; continues polling.

4.  **`OutboxProcessor` publishes 'OrderRequested' event**
    *   **Planned:** Publishes the detected 'OrderRequested' event to the Message Broker (MB).
    *   **File:** `OutboxProcessor/OrderOutboxWorker.cs`
    *   **Error Handling:** (Assumed) Retries on MB connection failures. Marks event as published in outbox *after* successful MB publish.

5.  **`InventoryService` consumes and processes 'OrderRequested'**
    *   **Planned:** `InventoryService/Worker.cs` consumes 'OrderRequested', checks/reserves inventory, updates local state, and publishes 'InventoryReserved' (or 'InventoryFailed') to its outbox.
    *   **File:** `InventoryService/Worker.cs`
    *   **Error Handling:** (Assumed) Idempotent consumption. If inventory is insufficient, publishes 'InventoryFailed'. DB transaction for reserve and outbox.

6.  **`OutboxProcessor` detects 'InventoryReserved' event**
    *   **Planned:** `OutboxProcessor/OrderOutboxWorker.cs` detects 'InventoryReserved' (or 'InventoryFailed') from `InventoryService`'s outbox.
    *   **File:** `OutboxProcessor/OrderOutboxWorker.cs`
    *   **Error Handling:** (Assumed) Same as step 3.

7.  **`OutboxProcessor` publishes 'InventoryReserved' event**
    *   **Planned:** Publishes 'InventoryReserved' (or 'InventoryFailed') to the MB.
    *   **File:** `OutboxProcessor/OrderOutboxWorker.cs`
    *   **Error Handling:** (Assumed) Same as step 4.

8.  **`AccountService` consumes and processes 'InventoryReserved'**
    *   **Planned:** `AccountService/Worker.cs` consumes 'InventoryReserved', attempts to charge customer, updates local state, and publishes 'AccountCharged' (or 'AccountFailed') to its outbox.
    *   **File:** `AccountService/Worker.cs`
    *   **Error Handling:** (Assumed) Idempotent consumption. If charge fails, publishes 'AccountFailed'. DB transaction for charge and outbox.

9.  **`OutboxProcessor` detects 'AccountCharged' event**
    *   **Planned:** `OutboxProcessor/OrderOutboxWorker.cs` detects 'AccountCharged' (or 'AccountFailed') from `AccountService`'s outbox.
    *   **File:** `OutboxProcessor/OrderOutboxWorker.cs`
    *   **Error Handling:** (Assumed) Same as step 3.

10. **`OutboxProcessor` publishes 'AccountCharged' event**
    *   **Planned:** Publishes 'AccountCharged' (or 'AccountFailed') to the MB.
    *   **File:** `OutboxProcessor/OrderOutboxWorker.cs`
    *   **Error Handling:** (Assumed) Same as step 4.

11. **`OrchestratorService` consumes events and updates order status**
    *   **Planned:** `OrchestratorService/Worker.cs` consumes 'AccountCharged' and 'InventoryReserved' events, updates order status to 'Fulfilled' (or 'Failed') in its database.
    *   **File:** `OrchestratorService/Worker.cs`
    *   **Error Handling:** (Assumed) Idempotent consumption. Handles various combinations of success/failure events to determine final order status (e.g., if InventoryFailed but AccountCharged, may need compensation logic).

12. **(Optional) `ClaimCheckProcessor` for large payloads**
    *   **Planned:** If initial request was large, `OrchestratorService` publishes 'ClaimCheckCreated' (Assumed). `ClaimCheckProcessor` (Assumed new service) retrieves actual payload for subsequent steps.
    *   **File:** (Assumed) `ClaimCheckProcessor/Worker.cs`
    *   **Error Handling:** (Assumed) Retrieval failures lead to retries or event failure.

13. **(Optional) `MessageReplicator` for analytics**
    *   **Planned:** `MessageReplicator` (Assumed new service) replicates all/specific events to an analytics system or another environment.
    *   **File:** (Assumed) `MessageReplicator/Worker.cs`
    *   **Error Handling:** (Assumed) Event replication failures are logged and retried; non-critical path, so core flow not impacted.

### Step/File Reference Table

| Step | Component(s) Involved | File Reference(s)                       |
| :--- | :-------------------- | :-------------------------------------- |
| 1    | External Client, OrchestratorService | `OrchestratorService/Program.cs` |
| 2    | OrchestratorService   | `OrchestratorService/Program.cs` |
| 3    | OutboxProcessor, OrchestratorService DB | `OutboxProcessor/OrderOutboxWorker.cs` |
| 4    | OutboxProcessor, Message Broker | `OutboxProcessor/OrderOutboxWorker.cs` |
| 5    | InventoryService, Message Broker | `InventoryService/Worker.cs`       |
| 6    | OutboxProcessor, InventoryService DB | `OutboxProcessor/OrderOutboxWorker.cs` |
| 7    | OutboxProcessor, Message Broker | `OutboxProcessor/OrderOutboxWorker.cs` |
| 8    | AccountService, Message Broker | `AccountService/Worker.cs`         |
| 9    | OutboxProcessor, AccountService DB | `OutboxProcessor/OrderOutboxWorker.cs` |
| 10   | OutboxProcessor, Message Broker | `OutboxProcessor/OrderOutboxWorker.cs` |
| 11   | OrchestratorService, Message Broker | `OrchestratorService/Worker.cs`  |
| 12   | OrchestratorService, ClaimCheckProcessor | (Assumed) `ClaimCheckProcessor/Worker.cs` |
| 13   | MessageReplicator, Message Broker | (Assumed) `MessageReplicator/Worker.cs` |