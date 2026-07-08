# TravelX — Sequence Diagrams (Mermaid source, EN)

> These are the exact Mermaid sources rendered to PNG and embedded into
> `TravelX_PRD_v1.1_EN.docx` §18 (Sequence Diagrams). Abstracted to three lanes only —
> User (Frontend) → Server (Backend) → Database. Notification writes are folded into the
> Server→Database step rather than shown as a separate service.
>
> A Korean-language version of these same flows (with additional batch/API context) lives in
> `PRD_Review_and_Changes.md` §3.

## 18.1 Exchange Rate Comparison

```mermaid
sequenceDiagram
    participant User as User (Frontend)
    participant Server as Server (Backend API)
    participant DB as Database

    User->>Server: Search currency / request rate comparison (GET /currencies)
    Server->>DB: Query per-branch rates + latest recommendation signal
    DB-->>Server: Rate rows + recommendation (Now/Wait/Neutral)
    Server-->>User: Rate comparison list + recommendation badge (with disclaimer)

    Note over Server,DB: Rates are refreshed once daily by an offline batch job (not shown)
```

## 18.2 Currency Exchange Reservation

```mermaid
sequenceDiagram
    participant User as User (Frontend)
    participant Server as Server (Backend API)
    participant DB as Database

    User->>Server: Select currency/amount/branch/pickup date-time and confirm
    Server->>DB: BEGIN TRANSACTION
    Server->>DB: SELECT stock FOR UPDATE (branch + currency)
    DB-->>Server: Current stock

    alt Sufficient stock (stock >= amount)
        Server->>DB: Deduct stock, insert Reservation (RESERVED), insert notification, COMMIT
        DB-->>Server: Success
        Server-->>User: 201 Created + confirmation (reservation No./QR)
    else Insufficient stock
        Server->>DB: ROLLBACK
        DB-->>Server: Rolled back
        Server-->>User: 409 Conflict (stock insufficient)
    end

    Note over User,Server: On cancellation
    User->>Server: Request cancellation (DELETE /reservations/{id})
    Server->>DB: Set status = CANCELLED, restore stock (+amount), insert notification, COMMIT
    DB-->>Server: Success
    Server-->>User: 200 OK
```
