# ADR-001: Server-authoritative slot reservation with one document per kitchen minute

## Document control

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-02-24 (amended 2026-04-24 and 2026-09-10) |
| Deciders | Tech Lead, Business Analyst, Node.js developers |
| Consulted | Product Owner (Operations Director), Kitchen lead (Market Hall), QA Lead |
| Related | BR-010 to BR-019, FR-BRN-06 to FR-BRN-08, FR-BRN-12, NFR-PERF-01, NFR-PERF-02, DEC-02 |

## Context and problem statement

Brasa promises customers an exact pickup minute that the kitchen can meet (DEC-02). The kitchen is one production line, and an order holds the P kitchen minutes immediately before its pickup minute (BR-011, BR-012). At the lunch rush, several customers look at the same free minutes at the same time; at Market Hall, 3 to 5 placements a minute is normal between 11:45 and 12:30.

The previous system computed availability in each client and re-checked the minute in application code before saving. Under concurrency, two orders could both pass the check. Brasa needs a design where two orders can never hold the same kitchen minute, the slot list stays fast (800 ms p95, NFR-PERF-01) and the rules stay in one place.

## Decision drivers

- Correctness under concurrency: no double-booked kitchen minute, ever.
- One source of truth for availability: apps and the till must not disagree.
- Performance at the rush profile (NFR-PERF-01, NFR-PERF-02).
- Auditability: which order holds which minute, and since when.
- MongoDB as the only data store.

## Considered options

1. **Check, then insert, in application code,** guarded by an in-process lock.
2. **One document per branch and day** holding an array of occupied minutes, updated with optimistic concurrency (version field).
3. **One reservation document per kitchen minute** with a unique index on branch, date and minute, inserted in the same transaction as the order.
4. **An external lock service** (for example a cache with distributed locks) around placement.

## Decision outcome

Chosen option: **3, one reservation document per kitchen minute with a unique index**, because the database itself refuses a second holder of a minute. No lock, no retry loop, no timing assumption is needed for correctness.

- Placement runs in a multi-document transaction: re-price the cart (BR-027), insert the order, insert P reservation documents `{branchId, date, minute, orderId}`. A duplicate-key error on any reservation aborts the transaction and returns 409 SLOT_TAKEN with a fresh list (BR-013).
- The slot engine is the only code that decides availability. Apps and the till call `GET /branches/{branchId}/pickup-slots` and render the minutes and reasons the server returns.
- Busy windows and delay blocks are not reservations. They are stored on the branch and applied by the slot engine (rows A7 and A8 of business rules table 3.1), so releasing them never touches order data.
- Overbooked till orders (BR-019) hold no reservations.
- Canceled, collected and no-show orders delete their future reservations in the same transaction as the status change (BR-017).

### Amendments

- **2026-04-24 (I-02).** The S4 load test measured the slot list at 1.4 s p95. The engine now keeps a per-branch, per-day occupancy bitmap (one bit per minute) that is rebuilt from the reservations on start and updated by the same transaction that inserts or deletes reservations. The slot list reads the bitmap: 310 ms p95. The unique index stays the guarantee.
- **2026-09-10 (CR-006).** The rush throttle keeps a counter of online kitchen minutes per rush block (RUSH_BLOCK_COUNTER), updated in the placement transaction by a conditional increment that fails when the cap would be exceeded. The counter is both the read model for row A9 and the concurrency guard. Counting reservations inside the transaction was tried first and allowed write skew in the concurrency test: two placements that insert different minutes do not conflict, so both read 8 online minutes and both committed. The reservation documents remain the record ([spec 001 plan](../../../08-ai-assisted-ba/specs/001-rush-hour-slot-throttling/plan.md)).

## Consequences

**Good**
- Correct by construction: TC-BRN-006 fires 50 parallel placements for the same minute and gets exactly one 201.
- Every reservation is traceable to an order and a time, which helped in INC-2026-004 to show that no order was lost.
- Business rules for availability live in one module with decision-table tests.

**Bad, and how we live with it**
- An order writes up to 30 documents (the online limit); a typical order writes 1 to 4. Writes are small and inside one transaction.
- Reservation documents grow by about 1,500 per branch per busy day. They are archived after 30 days; the order keeps its kitchen minutes as fields.
- The bitmap must never drift from the reservations. A nightly job compares them and alerts on any difference (none so far).

## Validation

- TC-BRN-003 (WE-2), TC-BRN-006 (concurrency), TC-NFR-001 (load).
- Production: zero duplicate-key errors outside the expected SLOT_TAKEN responses; 0.4% of placements at the rush end in SLOT_TAKEN.

## Related documents

- [System architecture](../system-architecture.md)
- [Business rules table 3.1](../../../02-requirements/business-rules.md#31-pickup-minute-availability)
- [Sequence diagrams](../../diagrams/sequence-diagrams.md)
- [ERD](../../data/erd.md)
