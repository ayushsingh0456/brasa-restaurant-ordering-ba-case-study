# Realtime Events: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-API-RT |
| Version | 1.1 |
| Status | Approved |
| Owner | Tech Lead (co-authored with the Business Analyst) |
| Last updated | 2026-08-07 |
| Reviewers | Flutter and React developers; QA Lead |

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-03-27 | R1 events; clients joined rooms with a `join` message |
| 1.1 | 2026-08-07 | CR-004 after INC-2026-004: server-side room membership, branch sequence on every event, heartbeat, snapshot on connect and gap; `join` removed |

**Purpose and scope.** The contract for live updates between the Brasa server and the boards, tills and customer apps: transport, rooms, ordering guarantees, the client algorithm and the event catalog. The design rationale is [ADR-002](../03-design/architecture/adr/ADR-002-live-updates-snapshot-and-sequence.md); the rules are BR-020 and BR-045.

## 1. Transport and rooms

| Topic | Rule |
|---|---|
| Transport | Socket.IO over WebSocket, with long-polling fallback; path `/v1/live` |
| Authentication | The access token in the handshake `auth.token`; an expired token closes the connection with `auth_expired`, and the client refreshes and reconnects |
| Rooms | Joined by the server on every connection, from the token. Staff: `branch:{branchId}` for each assigned branch in use (the till and boards send `branchId` in the handshake query). Customers: `customer:{customerId}`, plus `slots:{branchId}` while a pickup sheet is open: the app sends `picker.open` or `picker.close` with the branch ID and the server adds or removes the membership. The slots room carries only `slots.changed`. Clients cannot join rooms themselves. |
| Ordering | Events in a branch room carry `seq`, which increases by exactly 1 per event at that branch. Customer-room events carry the order's `version`. |
| Heartbeat | The server emits `heartbeat` every 10 s with `serverTime` and the branch `seq` |
| Payload size | Events carry the changed order in the board view (no contact data for the kitchen view); at most about 4 KB |

## 2. Client algorithm (boards and tills)

```text
on connect (first or reconnect):
    status = RECONNECTING
    snapshot = GET /v1/branches/{branchId}/snapshot?view={KITCHEN|COUNTER|TILL}
    state = snapshot.orders; last = snapshot.seq
    apply buffered events with seq > last, in order
    status = LIVE

on event e:
    if e.seq <= last: ignore                      # duplicate or already in the snapshot
    elif e.seq == last + 1: apply(e); last = e.seq
    else: buffer(e); reload snapshot as on connect   # gap

every second:
    if now - lastHeartbeat >= 10 s: status = RECONNECTING, gray the list
    if now - lastHeartbeat >= 60 s: status = NOT_LIVE, show the full-width banner
```

The decision table is [business rules table 3.7](../02-requirements/business-rules.md#37-live-update-handling-on-boards-and-tills). Customer apps follow the same pattern with the order list as their snapshot and `version` instead of `seq`.

## 3. Event catalog

Common envelope: `{ "event": "...", "seq": 1031, "branchId": "LOC-01", "serverTime": "2026-09-18T12:05:02+02:00", "data": { ... } }`.

| Event | Room | Emitted when | Data | Boards: alert | Consumers |
|---|---|---|---|---|---|
| `order.created` | branch | An order is placed in any channel (not while Pending payment) | Order (board view) | Sound and "New order {number}" | Kitchen and counter boards, tills |
| `order.updated` | branch | A called-up order is saved, or a customer's Pending order becomes Queued | Order | Sound and "Order updated {number}" | Boards, tills |
| `order.status_changed` | branch, customer | Queued to In preparation (timer), Ready, Collected, No-show | Order number, from, to, at | Silent | Boards, tills, customer app |
| `order.canceled` | branch, customer | Any cancellation | Order number, actor role, reason | Sound and "Order canceled {number}" if by the customer; silent if by staff | Boards, tills, customer app |
| `order.paid` | branch, customer | Attempt Succeeded | Order number, method, tip | Silent | Counter board, tills, customer app |
| `order.invoiced` | customer | Invoice issued | Order number, invoice number | — | Customer app |
| `order.locked` / `order.unlocked` | branch | Call-up opened, saved, released or expired | Order number, till, initials | Silent | Boards, tills |
| `refund.updated` | branch, customer | Refund Pending, Succeeded or Needs attention | Order number, amount, status | Silent; Needs attention adds a counter-board badge | Counter board, customer app |
| `slots.changed` | branch and `slots:{branchId}` | Reservations, busy window, delay block, closure or throttle release change availability | Date, first changed minute | — | Customer pickers, tills |
| `delay_block.changed` | branch | Block computed, changed or released | Minutes, late order, minutes late | Silent banner | Boards |
| `busy_window.changed` | branch | Set, replaced or removed | Start, end, reason | Silent banner | Boards, tills |
| `product.availability_changed` | branch | Sold out or available again | Product ID, soldOutToday | — | Tills, customer menus |
| `branch.updated` | branch | Hours, closures or settings change, or deactivation | Branch | — | All clients at the branch |
| `heartbeat` | branch, customer | Every 10 s | serverTime, seq | — | All clients |

## 4. Guarantees and limits

- **At least once, in order, per branch.** A client can receive an event twice or miss some during a disconnect; `seq` and the snapshot make both harmless.
- **The snapshot wins.** If an event disagrees with the snapshot, the snapshot is right.
- **No business decision on the client.** Clients never compute availability, prices or lateness; the badges use `serverTime` (BR-020).
- **Verification.** TC-TIL-011, TC-TIL-012 and the chaos test TC-NFR-004 (100 forced disconnects, zero missed changes).

## Related documents

- [ADR-002 Live updates: snapshot and sequence](../03-design/architecture/adr/ADR-002-live-updates-snapshot-and-sequence.md)
- [OpenAPI contract](openapi.yaml)
- [API guidelines](api-guidelines.md)
- [Sequence diagrams](../03-design/diagrams/sequence-diagrams.md)
- [INC-2026-004 Till and board desync after reconnect](../07-operations/incidents/INC-2026-004-till-board-desync-after-reconnect.md)
