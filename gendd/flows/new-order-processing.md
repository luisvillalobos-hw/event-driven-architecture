This document details the planned "New Order Processing" flow within the event-driven microservices architecture. This high-criticality flow orchestrates order creation, financial checks, and inventory reservations, ensuring reliable processing via an outbox pattern and message broker.

```
Client         OrderMaker   OutboxProc   MessageBroker   Orchestrator   AccountSvc   InventorySvc   Downstream
  |               |            |                |             |             |              |             |
  |-- New Order --> OrderMaker : 1. API Request
  |               |  2a. Create Order & Outbox
  |               |<-- Saved -->
  |<-- Ack -->    |
  |               |            |                |             |             |              |             |
  |               |<------- Outbox Poll -------->|             |             |              |             |
  |               |            |-- 2b. Publish OrderCreated --> MessageBroker
  |               |            |                |             |             |              |             |
  |               |            |                |<-- 3