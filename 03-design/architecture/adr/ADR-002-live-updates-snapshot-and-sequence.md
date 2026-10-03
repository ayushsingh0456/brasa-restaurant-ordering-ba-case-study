# ADR-002: Live updates as sequenced notifications with a snapshot on every connect

## Document control

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-07-24 (supersedes the R1 live-update design of 2026-02-24) |
| Deciders | Tech Lead, Business Analyst, Flutter and React developers |
| Consulted | QA Lead, Product Owner (Operations Director), Branch Manager (Station Quarter) |
| Related | BR-020, BR-045, FR-TIL-10, FR-TIL-11, NFR-PERF-03, NFR-REL-02, NFR-OBS-02, CR-004, INC-2026-004 |

## Context and problem statement

The kitchen board, the counter board, every till and the customer's app must show the same state of an order within 2 seconds (NFR-PERF-03). R1 used Socket.IO as follows:
- A client joined its branch room once, by emitting `join` after the first connect.
- Events carried the changed order, and clients applied them directly.
- There was no replay, no ordering check and no indication of a stale connection.

On 2026-07-17 (INC-2026-004), Station Quarter's router restarted during the lunch rush. Both tills reconnected automatically but never re-emitted `join`, so they received no events for 40 minutes. Their queues looked normal and showed no warning. Staff re-keyed three app orders as walk-ins, and the kitchen cooked them twice.

The question: how do clients stay correct across disconnects, server restarts and missed events, and how do people know when a screen is not live?

## Decision drivers

- A screen must never look live when it is not.
- After any disruption, the client must converge on the server's state without human action.
- Keep latency under 2 seconds at p95 and load low (24 staff clients, about 600 customer sessions at peak).
- Work across several API instances.

## Considered options

1. **Keep events as the source of truth** and add Socket.IO connection state recovery, which replays missed packets for a short period.
2. **Polling only:** clients fetch the board every 5 seconds.
3. **Events as notifications with a branch sequence number, plus a versioned snapshot** loaded on every connect, reconnect or sequence gap; rooms joined by the server.
4. **Server-sent events** instead of Socket.IO.

## Decision outcome

Chosen option: **3, sequenced notifications with a snapshot.**

- **Server-side room membership.** The server joins the socket to its rooms in the connection handler, from the token's role and branches (FR-IAM-06). A reconnect is a new connection, so it always re-joins. The client no longer emits `join`.
- **Branch sequence.** Every change at a branch increments a counter stored in MongoDB and stamps the event with it. Events for one branch are therefore totally ordered.
- **Snapshot.** `GET /branches/{branchId}/snapshot` returns all open orders, the busy window, the delay block and the current sequence. Clients load it after every connect and whenever they see a gap (sequence > last + 1), then apply only events above the snapshot's sequence (business rules table 3.7).
- **Heartbeat and indicator.** The server sends a heartbeat every 10 seconds. Clients show "Reconnecting" after 10 seconds without one and a full-width "Not live" banner after 60 seconds (BR-045, FR-TIL-11).
- **Customers** use the same pattern in a `customer:{id}` room; their snapshot is the order list.

Option 1 was rejected because recovery is time-limited and in-memory, does not survive an app restart or a deploy, and still gives no signal to the user. Option 2 met correctness but not the 2-second target without heavy load. Option 4 offered nothing over Socket.IO for this case.

## Consequences

**Good**
- Correctness no longer depends on every event arriving. In the chaos test (TC-NFR-004), 100 forced disconnects gave zero missed changes.
- Staleness is visible within 10 seconds.
- Clients became simpler: one code path (load snapshot, then apply events), the same after start, reconnect or gap.

**Bad, and how we live with it**
- A snapshot is about 40 KB for a busy branch. Reconnect storms (for example after a deploy) are spread with jittered reconnect delays of 0.5 to 3 seconds.
- The sequence counter is a write per change. At peak this is under 10 writes a second per branch, well within MongoDB's capacity.
- Events still carry the changed order for speed; clients must treat them as hints and accept that the snapshot wins.

## Validation

- TC-TIL-011, TC-TIL-012, TC-NFR-002, TC-NFR-004.
- Production since patch 1.0.5 (2026-08-10): median recovery after a disconnect 1.8 seconds; no stale-board report from staff.
- NFR-OBS-02 (b) pages if a branch has no connected board or till for 2 minutes during opening hours.

## Related documents

- [Realtime events](../../../04-api/realtime-events.md)
- [INC-2026-004 Till and board desync after reconnect](../../../07-operations/incidents/INC-2026-004-till-board-desync-after-reconnect.md)
- [Business rules table 3.7](../../../02-requirements/business-rules.md#37-live-update-handling-on-boards-and-tills)
- [Sequence diagrams](../../diagrams/sequence-diagrams.md)
