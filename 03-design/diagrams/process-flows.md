# Process Flows (TO-BE): Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DES-PF |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-18 |
| Reviewers | Product Owner (Operations Director); Branch Manager (Station Quarter); Kitchen Lead (Market Hall); QA Lead |

**Purpose and scope.** BPMN-style swimlane flows of the future state: a customer pickup order, a walk-in order at the till, kitchen trouble, uncollected orders, and cancellation with refund. Lanes are subgraphs; decisions are diamonds; the rules that drive each decision are named on the shape. The AS-IS flows are in [current-vs-future-state.md](../../01-discovery/current-vs-future-state.md).

## 1. Customer pickup order

```mermaid
flowchart TB
  subgraph C["Customer"]
    C1(["Wants lunch<br/>at 12:30"])
    C2["Chooses branch,<br/>customizes items"]
    C3["Chooses pickup minute<br/>and how to pay"]
    C4["Pays on the provider's page"]
    C5["Gets 'Ready' push,<br/>walks to the branch"]
    C6(["Collects the order"])
  end
  subgraph A["Customer app or web panel"]
    A1["Shows menu, prices<br/>and allergens (BR-028)"]
    A2["Shows server-priced cart:<br/>net, VAT per rate, total"]
    A3["Shows available minutes<br/>with reasons"]
    A4["Confirmation: number, pickup,<br/>cancel deadline"]
  end
  subgraph S["Brasa server"]
    S1["Price cart<br/>(BR-023 to BR-027)"]
    S2["Slot engine: kitchen minutes,<br/>lead time, throttle (table 3.1)"]
    S3{"Minutes still free<br/>and price unchanged?"}
    S4["Create order, reserve<br/>kitchen minutes (ADR-001)"]
    S5["Refresh list or<br/>show new total"]
    S6["Webhook: payment<br/>Succeeded (BR-038)"]
    S7["Invoice through<br/>the outbox (BR-037)"]
  end
  subgraph K["Kitchen board"]
    K1["Order appears with<br/>Without and Extra"]
    K2["Prepares from T - P"]
    K3["Marks Ready"]
  end
  subgraph T["Counter and till"]
    T1{"Paid?"}
    T2["Deliver: Collected"]
    T3["Cash or card:<br/>Paid and Collected"]
  end
  C1 --> C2 --> A1 --> S1 --> A2 --> C3
  C3 --> S2 --> A3 --> S3
  S3 -- "No" --> S5 --> A3
  S3 -- "Yes" --> S4 --> A4
  S4 --> K1 --> K2 --> K3 --> C5
  A4 -->|"Pay online now"| C4 --> S6 --> S7
  C5 --> T1
  T1 -- "Yes" --> T2 --> C6
  T1 -- "No" --> T3 --> C6
```

Exceptions handled elsewhere: customer cancellation (flow 5), kitchen late or in trouble (flow 3), customer does not come (flow 4).

## 2. Walk-in or dine-in order at the till

```mermaid
flowchart TB
  subgraph CU["Walk-in customer"]
    W1(["Orders at the counter"])
    W2(["Receives food"])
  end
  subgraph ST["Counter staff on the till"]
    W3["Taps tiles; customizes<br/>in the dialog"]
    W4{"Eat in?"}
    W5["Dine in"]
    W6["Take away (default)"]
    W7{"Pays now?"}
    W8["Press and hold cash,<br/>or card with tip"]
    W9["Tap the time field:<br/>placed unpaid"]
  end
  subgraph SV["Brasa server"]
    W10["Earliest free minute,<br/>no lead time (BR-019)"]
    W11{"Any minute free<br/>before closing?"}
    W12["Offer next 5 minutes,<br/>flag Overbooked"]
    W13["Order placed and paid,<br/>invoice through the outbox"]
    W14["Order placed unpaid,<br/>in the live queue"]
  end
  subgraph KI["Kitchen board"]
    W15["Prepares and<br/>marks Ready"]
    W16{"Dine-in or<br/>already paid?"}
    W17["Collected automatically<br/>(BR-032)"]
    W18["Waits in queue:<br/>settle, then Collected"]
  end
  W1 --> W3 --> W10 --> W11
  W11 -- "Yes" --> W4
  W11 -- "No" --> W12 --> W4
  W4 -- "Yes" --> W5 --> W7
  W4 -- "No" --> W6 --> W7
  W7 -- "Yes" --> W8 --> W13 --> W15
  W7 -- "Later" --> W9 --> W14 --> W15
  W15 --> W16
  W16 -- "Yes" --> W17 --> W2
  W16 -- "No" --> W18 --> W2
```

Target: median 50 s or less from first tile to cash payment (NFR-USE-02, OBJ-06); UAT measured 41 s.

## 3. Kitchen trouble: busy window and delay protection

```mermaid
flowchart TB
  subgraph MG["Manager"]
    M1(["Fryer fails at 14:05"])
    M2["Sets busy window<br/>15:00-15:30, reason Equipment"]
    M3["Optionally releases<br/>the delay block early"]
  end
  subgraph AP["Admin Panel"]
    M4{"Window valid?<br/>(BR-015)"}
    M5["Shows the problem<br/>and the fix"]
  end
  subgraph EN["Slot engine"]
    M6["Blocks kitchen minutes<br/>15:00 to 15:29 for new orders"]
    M7["Every minute: any order late?"]
    M8{"Late order exists?"}
    M9["Block next D free minutes<br/>after the lead time,<br/>D = minutes late, max 20 (BR-016)"]
    M10["No delay block"]
  end
  subgraph BD["Boards and apps"]
    M11["Pickers refresh<br/>within 2 s"]
    M12["Boards show the block<br/>and the late order first"]
  end
  M1 --> M2 --> M4
  M4 -- "No" --> M5 --> M2
  M4 -- "Yes" --> M6 --> M11
  M7 --> M8
  M8 -- "Yes" --> M9 --> M12 --> M11
  M8 -- "No" --> M10 --> M11
  M12 --> M3 --> M10
```

## 4. Uncollected order: reminders, no-show and Prepay-only

```mermaid
flowchart TB
  subgraph SY["Brasa worker"]
    U1(["Order marked Ready"])
    U2{"App or web pickup,<br/>unpaid?"}
    U3["Anchor = later of pickup<br/>minute and ready time"]
    U4["Push at anchor + 10"]
    U5["Email at anchor + 15"]
    U6["Final email with payment<br/>link at anchor + 30"]
    U7{"Paid, collected or<br/>canceled meanwhile?"}
    U8["Stop the ladder"]
    U9["Closing + 30 min:<br/>No-show"]
    U10{"Was it unpaid?"}
    U11["Prepay-only for 30 days;<br/>no-show email (BR-041)"]
    U12["No account action"]
  end
  subgraph MN["Manager at closing"]
    U13["Counter board lists<br/>'Not collected'"]
    U14["Marks handed-over<br/>orders Collected"]
  end
  subgraph CS["Customer"]
    U15(["Collects or pays"])
    U16(["Next order:<br/>must pay online"])
  end
  U1 --> U2
  U2 -- "No" --> U12
  U2 -- "Yes" --> U3 --> U4 --> U7
  U7 -- "Yes" --> U8
  U7 -- "No" --> U5 --> U6 --> U13 --> U14 --> U9
  U15 -.-> U7
  U9 --> U10
  U10 -- "Yes" --> U11 --> U16
  U10 -- "No" --> U12
```

Changed in R1.1 by CR-003: the R1 flow suspended the account at U11 and had no "Not collected" list (U13, U14).

## 5. Cancellation and refund

```mermaid
flowchart TB
  X0(["Cancel requested"]) --> X1{"Who?"}
  X1 -- "Customer" --> X2{"Queued and now at or<br/>before T - C? (BR-030)"}
  X2 -- "No" --> X3["Refuse: 'Cancellation closed<br/>at 12:16. Call the branch.'"]
  X2 -- "Yes" --> X6
  X1 -- "Staff" --> X4{"Paid and actor is<br/>Counter Staff? (BR-031)"}
  X4 -- "Yes" --> X5["Refuse: 'Only a Manager can<br/>cancel a paid order.'"]
  X4 -- "No" --> X6["Order Canceled with reason;<br/>kitchen minutes released (BR-017)"]
  X6 --> X7{"Paid how?"}
  X7 -- "Unpaid" --> X10
  X7 -- "Online" --> X8["Automatic full refund<br/>including tip (FR-PAY-08)"]
  X7 -- "At the counter" --> X9["Manager records cash or<br/>card refund with reference"]
  X8 --> X11{"Invoiced?"}
  X9 --> X11
  X11 -- "Yes" --> X12["Cancellation receipt<br/>in the POS (FR-PAY-09)"]
  X11 -- "No" --> X10["Notify customer and boards"]
  X12 --> X10
```

## Related documents

- [Business rules](../../02-requirements/business-rules.md)
- [Sequence diagrams](sequence-diagrams.md)
- [State machines](state-machines.md)
- [Current vs future state](../../01-discovery/current-vs-future-state.md)
- [User stories](../../05-delivery/epics.md)
