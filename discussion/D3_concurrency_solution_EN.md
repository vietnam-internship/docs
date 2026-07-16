### Background

<img width="1892" height="328" alt="img" src="https://github.com/user-attachments/assets/24b19087-22c0-4b77-88d9-38d683d440c3" />

##### To summarize our requirement: 
- At an exchange branch, reservations open 7 days in advance, and within each 30-minute database row between opening and closing time, up to 6 people can reserve by default. (The actual headcount can be configurable by exchange shops.)
- The table structure for reserving a limited number of seats per exchange branch is as shown above. When we designed this structure, a race condition problem shows up. 
- Here's how we thought about solving it.

---

### Proposed Solutions Introductions

#### Plan A: 1 Row LOCKED
A pessimistic-lock approach: add a `remaining` field to `BranchTimeSlot` and decrement it under lock.

<img width="2210" height="703" alt="plan-a-locking" src="https://github.com/user-attachments/assets/ccda9757-8225-4875-89dc-3f894c536e50" />

- Lock contention happens on a single `BRANCH_TIME_SLOT` row (1 row). 
- Only the thread holding the lock checks `remaining > 0`, decrements it, and only then inserts its own row into `RESERVATION`. 
- Since every thread has to pass through this one locked row in turn, rows on the N side are always created one at a time, strictly in the order locks are acquired.

#### Plan B: SKIP LOCKED (pre-generated reservation slots)

<img width="1582" height="1350" alt="plan-b-locking" src="https://github.com/user-attachments/assets/3a9d495c-a43e-434e-a12d-e27915a836ec" />

- Since reservations open a week in advance, a batch job pre-creates the `Reservation` rows a week ahead of time. 
- An incoming request atomically claims an unlocked slot with `SELECT ... FOR UPDATE SKIP LOCKED LIMIT 1`, then `UPDATE`s.

- All six rows on the N side already exist before any request arrives. Using a composite index on `(time_slot_id, member_id)`, each thread finds an available (`member_id` is `null`) row in O(log N), then locks it with `SELECT ... FOR UPDATE SKIP LOCKED LIMIT 1`. 
- If R1 has already locked a row, R2 and R3 skip it and move on to the next open row.

---

### Pros and Cons by Plans

#### Plan A: 1 Row LOCKED

**Pros**
- Simple to implement. Check and decrement `remaining` with `SELECT ... FOR UPDATE`, then commit — with no batch job or pre-generation logic needed.
- A `Reservation` row is only created once a reservation is confirmed, so there's no unclaimed-row problem like the one Plan B has.

**Cons**
- Throughput is effectively capped by the slot's capacity (max 6). If requests pile up on the same slot, the rest queue up on the lock and response latency grows.
- If DAU grows and requests concentrate on popular slots, this structure may not hold up, forcing a redesign toward SKIP LOCKED or a caching layer.

#### Plan B: SKIP LOCKED

**Pros**
- Since each request locks a different row, requests are processed in parallel with no waiting even under load — throughput isn't bound by the slot's capacity.
- A reservation holder's info only ever gets `UPDATE`d into an already-existing row, so a read always sees the actual reservation state.

**Cons**
- Rows equal to capacity must be pre-created before the reservation window opens, which requires a separate batch job — it can be a new point of failure.
- It's normal for unclaimed rows (`member_id` == `null`) to exist at all times, which is an unusual model from a data-integrity standpoint.

---

### Reaching a Decision

@wlgns12370 : In terms of traffic, I think Plan A is the reasonable choice to start with. We should run on A while monitoring, and once bottlenecks start showing up regularly, that's the point to move to Plan B or another approach.

@GyunHeee : I'd like to propose adopting Plan A (1 Row LOCKED, pessimistic lock). Since the slot capacity is hard-capped at 6, the upper bound on Plan A's biggest downside — the lock queue — is effectively fixed as well. Even in the worst case, it's just six short transactions processed in sequence, and starting from the 7th request we can return a "fully booked" response immediately with no lock wait — so the latency we're worried about shouldn't actually be significant. Plan B, on the other hand, carries a higher operational cost at this stage. It introduces a new point of failure — the pre-generation batch (if the batch fails, reservations become impossible altogether). I think it's more reasonable to start with a simpler structure than to pre-assume unverified traffic, and this is consistent with our earlier decision to hold off on introducing a cache and solve this with the RDB alone for now.

@hyun7586 : Since finishing the entire business flow quickly within a limited timeframe is the priority, I'd propose introducing Plan A first, since it's simpler to implement, while leaving room to expand to Plan B later. Since there's an upper bound on the reservation capacity per slot, the lock-queue bottleneck risk we're worried about shouldn't be large either. On top of that, choosing the more expensive Plan B based on conceptual discussion alone, without an implementation or measured metrics, is a real cost to take on. So I think the most reasonable roadmap is to implement the flow quickly with Plan A first, then validate a move to Plan B later based on actual load-test performance data.


**=> Our team adopted: Plan A** 

---

### Request for Feedback

We thought this race condition could also be solved with a JVM-local cache or a global cache like Redis, but decided that introducing a cache layer would also mean managing consistency with the DB, which felt like over-engineering at this stage — so we chose to solve it with the RDB alone for now. Within that constraint we worked through the pros and cons of Plan A and Plan B, and we think Plan A is the right call.

##### Here's what we'd like to know(Review Points):
- **Whether the flow itself** — coming up with several options as a team, discussing them, and picking the design that fits our current situation — **actually resembles how this is done in practice.**
- **Whether it's reasonable to hold off on a cache and solve this with the RDB first.**
- Given the pros and cons above, **which of Plan A or Plan B you'd recommend for this situation, or whether there's something we're missing that we should be looking at more closely.**
