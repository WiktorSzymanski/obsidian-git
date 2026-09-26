# Project Keywords — Inventory Item Reservation (TO vs ES)

The branches are **layered experiments**: two families (TO = Traditional/Outbox,
ES = Event Sourcing) share a common domain, and each numbered branch changes
exactly one variable.

---

## Shared foundation (every branch)

Explain these once — they underpin all 7 branches:

- **Inventory reservation domain** — reserve stock for an order
- **Order as an aggregate of multiple line items** — one order reserves several distinct items
- **Saga / process orchestration** — coordinating the multi-item reservation as one logical unit
- **Compensating transaction** — releasing already-reserved items when a later item in the order fails
- **Asynchronous command processing** — HTTP returns an order id immediately; the order is placed on a **work queue** and handled by background **worker threads** (producer–consumer)
- **Optimistic concurrency control** — version/expected-revision checks, no pessimistic locks
- **Retry with exponential backoff + jitter** — on concurrency conflicts
- **Correlation id / idempotency** — tracing a command end-to-end
- **Eventual consistency** between write side and read side
- **Domain events** as the integration mechanism
- **Read model / projection** — separate queryable representation
- **Write amplification experiment** — "artificial entity size" / payload padding to measure cost of writing larger records (a deliberate thesis variable)
- **Observability** — metrics (Prometheus), dashboards (Grafana), load testing (k6), latency metrics (persist time, publish lag)

---

## TO family — Traditional state + Transactional Outbox

Shared across TO-1…TO-4:

- **Current-state-as-source-of-truth** — a mutable row is the truth (vs. an event log)
- **ORM / object–relational mapping** — whole-object persistence
- **Optimistic locking via version column**
- **Normalized relational schema** — separate orders / reservations / inventory tables
- **Transactional Outbox pattern** — write state + event atomically in one DB transaction
- **Dual-write problem** — the consistency hazard the outbox exists to solve

### TO-1 — Polling outbox
- **Outbox polling publisher** (pull-based relay)
- **Competing consumers / row-level locking** (`FOR UPDATE SKIP LOCKED`)
- **Exactly-once delivery**
- **Load shedding / backpressure** (bounded queue — later removed for parity)

### TO-2 — Push via database pub/sub
- **Database NOTIFY/LISTEN** — push-based event delivery
- **Database trigger** firing notifications
- **Push vs. poll** delivery tradeoff

### TO-3 — In-process after-commit delivery
- **After-commit transactional event dispatch** — deliver synchronously when the transaction commits
- **Polling as crash-recovery fallback** (belt-and-suspenders durability)

### TO-4 — TO-3 + cache
- **In-memory cache of aggregate state** (read-through hot cache)
- **TTL / expire-after-access** (idle eviction)
- **Cache invalidation & consistency**

---

## ES family — Event Sourcing

Shared across ES-1…ES-3:

- **Event Sourcing** — append-only event log is the source of truth
- **Aggregate reconstruction by replaying events**
- **CQRS** — distinct write model (aggregate) vs. read model (projection)
- **Event store / per-aggregate stream**
- **Subscription-driven projection** (processing group updating the read model)
- **Optimistic concurrency via expected stream version**
- **Replayability / temporal queries / free audit history** (the architectural payoff)

### ES-1 — Pure event sourcing (baseline)
- **Full stream replay on every command** — no read optimizations; reference point

### ES-2 — ES-1 + snapshots
- **Snapshots** — periodic materialized aggregate state
- **Snapshot threshold / bounded replay cost**

### ES-3 — ES-2 + cache
- **In-memory aggregate cache (hot aggregate)** — skip replay *and* snapshot load
- **Cache** (the concept shared with TO-4)

---

## The two "axes" your reader needs

The whole project is a **2-family × N-optimization matrix**:

| Variable | TO | ES |
|---|---|---|
| Event delivery mechanism | TO-1 poll / TO-2 NOTIFY / TO-3 after-commit | (inherent to event store) |
| State read optimization | TO-4 cache | ES-2 snapshot, ES-3 cache |

Cross-cutting keyword pairs to highlight: **ORM** (all TO) vs **event replay** (all ES),
and **cache** (TO-4 & ES-3).
