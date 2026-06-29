# Shopify Bulk Query Flow (Happy Path)

This document explains the complete lifecycle of a Shopify Bulk Query using the **Moqui SystemMessage** framework.

---

# Overall Flow

```text
Queue
   ↓
Send
   ↓
Poll
   ↓
Consume
```

Each phase is handled by a scheduled job and updates the `SystemMessage` as it progresses.

---

# PHASE 1 — Queue

## Scheduled Job

```text
queue_BulkQuerySystemMessage_BulkProductAndVariantsById_{SHOP}
```

## Service Called

```text
queue#BulkQuerySystemMessage
```

## What Happens?

- Creates a new **outgoing SystemMessage**.
- Stores the query parameters in `messageText`.
- Marks the message as ready to be sent.

### SystemMessage State

| Field | Value |
|--------|-------|
| `messageText` | `JSON.stringify(queryParams)` (e.g. `{"fromDate":"..."}`) |
| `statusId` | `SmsgProduced` |
| `isOutgoing` | `Y` |

### Flow

```text
Scheduled Job
        │
        ▼
queue#BulkQuerySystemMessage
        │
        ▼
Create Outgoing SystemMessage
        │
        ▼
Status = SmsgProduced
Payload = Query JSON
```

---

# PHASE 2 — Send

## Scheduled Job

```text
send_BulkProductAndVariantsByIdQueryProducedSystemMessages
```

## Services Called

```text
send#AllProducedSystemMessages
        ↓
send#BulkQuerySystemMessage
```

## What Happens?

1. Finds all `SmsgProduced` messages of type `BulkProductAndVariantsByIdQuery`.
2. Reads the JSON query stored in `messageText`.
3. Calls Shopify GraphQL Bulk API.
4. Shopify starts a bulk operation.
5. Stores Shopify's Bulk Operation ID.
6. Marks the message as sent.

### SystemMessage State

| Field | Value |
|--------|-------|
| `messageText` | Query JSON |
| `remoteMessageId` | Shopify Bulk Operation ID (`gid://...`) |
| `statusId` | `SmsgSent` |
| `isOutgoing` | `Y` |

### Flow

```text
Scheduled Job
        │
        ▼
send#AllProducedSystemMessages
        │
        ▼
send#BulkQuerySystemMessage
        │
        ▼
Run Shopify Bulk Query
        │
        ▼
Store Bulk Operation ID
        │
        ▼
Status = SmsgSent
```

---

# PHASE 3 — Poll

## Scheduled Job

```text
poll_BulkOperationResult_ShopifyBulkQuery_{CU/OP/PE}
```

## Services Called

```text
poll#BulkOperationResult
        ↓
process#BulkOperationResult
        ↓
get#BulkOperationResult
```

## What Happens?

1. Finds the previously sent (`SmsgSent`) SystemMessage.
2. Uses the stored `remoteMessageId`.
3. Calls Shopify to check whether the bulk operation has finished.
4. If completed:
   - Shopify returns a download URL.
   - Creates a **new incoming SystemMessage**.
   - Stores the download URL inside `messageText`.
   - Marks the original outgoing message as confirmed.

### Incoming SystemMessage

| Field | Value |
|--------|-------|
| `messageText` | Shopify Download URL |
| `statusId` | `SmsgReceived` |
| `isOutgoing` | `N` |

### Original Outgoing Message

| Field | Value |
|--------|-------|
| `statusId` | `SmsgConfirmed` |

### Flow

```text
Scheduled Job
        │
        ▼
poll#BulkOperationResult
        │
        ▼
Check Shopify Status
        │
        ▼
Bulk Operation Complete?
        │
        ├── No → Exit
        │
        └── Yes
               │
               ▼
Receive Download URL
               │
               ▼
Create NEW Incoming SystemMessage
               │
               ▼
Status = SmsgReceived
Payload = Shopify Download URL
```

---

# PHASE 4 — Consume

## Scheduled Job

```text
consume_AllReceivedSystemMessages_frequent
```

## Services Called

```text
consume#AllReceivedSystemMessages
        ↓
consume#BulkOperationResult
```

## What Happens?

1. Finds all `SmsgReceived` messages.
2. Reads the Shopify download URL from `messageText`.
3. Downloads the `.jsonl` file.
4. Stores it locally.
5. Queues another SystemMessage for SFTP transfer.

### Expected SystemMessage

| Field | Value |
|--------|-------|
| `messageText` | Shopify Download URL |
| `statusId` | `SmsgReceived` |

### Operations Performed

```text
Read messageText
        │
        ▼
Download JSONL File
        │
        ▼
Store File
        │
        ▼
Queue SFTP SystemMessage
```

---

# Complete Happy Path

```text
QUEUE
 │
 ▼
Create SystemMessage
Status = SmsgProduced
Payload = Query JSON
 │
 ▼
SEND
 │
 ▼
Send Query to Shopify
Store Bulk Operation ID
Status = SmsgSent
 │
 ▼
POLL
 │
 ▼
Check Shopify Status
 │
 ▼
Operation Complete
 │
 ▼
Create NEW Incoming Message
Payload = Download URL
Status = SmsgReceived
 │
 ▼
CONSUME
 │
 ▼
Read Download URL
 │
 ▼
Download JSONL File
 │
 ▼
Store File
 │
 ▼
Queue for SFTP
```

---

# SystemMessage Lifecycle

| Phase | Status | `messageText` |
|--------|---------|---------------|
| Queue | `SmsgProduced` | Query JSON |
| Send | `SmsgSent` | Query JSON |
| Poll (Incoming Message) | `SmsgReceived` | Shopify Download URL |
| Original Message | `SmsgConfirmed` | Query JSON |

---

# Summary

| Phase | Scheduled Job | Main Service | Purpose |
|------|----------------|--------------|---------|
| Queue | `queue_BulkQuerySystemMessage_BulkProductAndVariantsById_{SHOP}` | `queue#BulkQuerySystemMessage` | Creates an outgoing SystemMessage with query parameters |
| Send | `send_BulkProductAndVariantsByIdQueryProducedSystemMessages` | `send#BulkQuerySystemMessage` | Sends the GraphQL Bulk Query to Shopify |
| Poll | `poll_BulkOperationResult_ShopifyBulkQuery_{SHOP}` | `poll#BulkOperationResult` | Waits for Shopify to complete the bulk operation |
| Consume | `consume_AllReceivedSystemMessages_frequent` | `consume#BulkOperationResult` | Downloads the resulting `.jsonl` file using the Shopify download URL |
