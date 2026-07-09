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

    User->>Server: Request rate comparison
    Server->>DB: Query rate data
    DB-->>Server: Rate data response
    Server-->>User: Return rate comparison result + recommendation badge

    Note over Server,DB: Rates are refreshed once daily by an offline batch job (not shown)
```

## 18.2 Currency Exchange Reservation

```mermaid
sequenceDiagram
    participant User as User (Frontend)
    participant Server as Server (Backend API)
    participant DB as Database

    User->>Server: Request currency exchange reservation
    Server->>DB: Check stock
    DB-->>Server: Stock status response

    alt Sufficient stock
        Server->>DB: Process reservation (deduct stock + register reservation)
        DB-->>Server: Processing complete
        Server-->>User: Reservation complete + confirmation returned
    else Insufficient stock
        Server-->>User: Reservation failed response
    end

    Note over User,Server: On cancellation
    User->>Server: Request cancellation
    Server->>DB: Process cancellation (restore stock + update status)
    DB-->>Server: Processing complete
    Server-->>User: Cancellation complete response
```
