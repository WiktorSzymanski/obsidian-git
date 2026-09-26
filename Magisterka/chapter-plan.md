# Chapter Plan — "System Overview"

This document proposes the structure of the **System Overview** chapter. It does **not**
contain the chapter text; each entry states *what* will be written in that section, plus
suggested figures/tables and the source material it should be derived from. The depth and
register already established in `system-overview.md` (the detailed TO-1 prose) is the
intended template for the per-system subsections.

The chapter has one overriding job: establish that the two architectures differ **only** in
the dimension under study and are held equal in every other respect, so that the performance
results in later chapters are attributable to the architectural choice rather than to
incidental implementation differences.

---

## 1. Chapter Introduction and Roadmap

**Contains:** A short framing paragraph stating the purpose of the chapter — to describe the
two implemented systems (Transactional Outbox and Event Sourcing) and their variants — and a
roadmap of the sections. State explicitly that both systems implement the *same* functional
scenario on the *same* infrastructure, and that each system is realised as a family of
incremental variants representing successive optimisations.

**Note:** Define the naming convention used throughout (TO-1…TO-4, ES-1…ES-3) here, once.

---

## 2. Reference Scenario and Functional Requirements *(shared)*

**Contains:** The business domain common to both systems — an inventory item reservation
service: items with finite stock, orders that reserve quantities across one or more items,
the lifecycle of an order (pending → confirmed/rejected), and the overselling-avoidance
requirement under concurrency. Frame the domain as a *deliberately minimal but
contention-heavy* workload chosen so measured differences arise from the event-handling
mechanism rather than business complexity.

**Figures/Tables:** Table of functional requirements; a domain-concept diagram (item,
reservation, order).

---

## 3. Common Reference Architecture *(shared)*

**Contains:** Everything the two systems share, described once so it is not repeated per
system. This is the backbone that makes them comparable.

### 3.1 Technology baseline
**Contains:** Kotlin/JVM, Spring Boot, PostgreSQL, Flyway, Jackson, single-service
deployment. State that persistence is synchronous (blocking JDBC) in both.
**Figures/Tables:** Technology/version table.

### 3.2 Interaction model and API
**Contains:** The shared REST surface and — crucially — the **asynchronous order pipeline**:
synchronous item creation vs. command-and-poll order placement (202 Accepted + status poll).
Explain why this shape is required to drive both systems at high concurrency.

### 3.3 Request admission and execution model
**Contains:** The shared threading model — bounded web tier, a worker/command stage, an
unbounded work queue (absorb-not-shed), and the database connection pool as the true
concurrency limit.

### 3.4 The mocked message broker
**Contains:** The decision to mock the downstream broker (log + record publish latency) in
both systems, and why: it removes broker/network variance so the measurement isolates each
architecture's own event-handling cost.

### 3.5 Deployment topology
**Contains:** The containerised runtime (application + PostgreSQL + load generator) and how a
single variant is brought up for a run.
**Figures/Tables:** Deployment/topology diagram (one per family, since the datastore differs:
PostgreSQL-as-state for TO, PostgreSQL-as-event-store for ES).

---

## 4. Design for Comparability *(methodological core)*

**Contains:** The explicit statement of **controlled variables vs. independent variables**.
List the parity decisions enforced across both families: single shared clock as the latency
origin, payload-size inflation to equalise serialized event sizes, unbounded queue for equal
load absorption, identical mocked broker, and the same workload/harness. Then name the
*independent* variable of each family (delivery mechanism for TO; state-reconstruction
strategy for ES). This section is what licenses the causal claims in the results chapter.

**Figures/Tables:** A two-column table: "Held constant across all variants" vs. "Varied
within each family".

---

## 5. System A — Transactional Outbox

> Use `system-overview.md` as the drafted basis for 5.1–5.5; trim its standalone framing so it
> reads as a section of this chapter rather than a self-contained document.

### 5.1 Conceptual basis
**Contains:** The dual-write problem and how the outbox pattern resolves it (atomic state +
event in one transaction; deferred relay). One or two paragraphs of theory.

### 5.2 Domain model and persistence
**Contains:** The three persistent entities (item with optimistic version, reservation,
order), the relational schema, and the `event_publication` outbox table (Spring Modulith
Event Publication Registry). Stress the co-location of business tables and outbox in one
database as the enabler of atomicity.
**Figures/Tables:** ER/schema listing; note the inline serialized-event payload column.

### 5.3 Command and data flow
**Contains:** The two-phase order processing — admission (persist pending order, enqueue) and
reservation (single transaction: load → apply domain rule → persist state + reservation +
order status → record events to outbox → commit) — and rollback-on-failure semantics.
**Figures/Tables:** Sequence diagram of the order write path.

### 5.4 Concurrency control
**Contains:** Optimistic locking on the item version, the bounded exponential-backoff retry
on lock conflicts, and how this maps onto the contended workload.

### 5.5 Event delivery — the variation point
**Contains:** Introduce that all TO variants share §5.2–5.4 and differ **only** in how
recorded outbox events are delivered. Then one subsection per variant:

- **5.5.1 TO-1 — Polling relay.** Scheduled drain claiming pending rows with
  `FOR UPDATE SKIP LOCKED`, once-per-event delivery, poll interval as the latency bound.
- **5.5.2 TO-2 — PostgreSQL NOTIFY/LISTEN.** Trigger emits `pg_notify` on outbox insert; a
  dedicated listening connection wakes a direct processor that delivers from the notification
  payload (no extra read); polling retained as crash-recovery backup. Push vs. poll trade-off.
- **5.5.3 TO-3 — Synchronous after-commit dispatch.** Spring Modulith's default multicaster
  delivers in the after-commit phase on the processing thread; polling retained as backup.
  Lowest latency, delivery coupled to the request path.
- **5.5.4 TO-4 — After-commit + read-path cache.** TO-3 plus a Caffeine cache of inventory
  state on the reservation load path (bounded size, idle TTL, version-guarded). Tests the
  effect of a read-side optimisation.

**Figures/Tables:** Comparative table of the four delivery mechanisms (push/pull, latency
characteristic, delivery guarantee, failure recovery).

---

## 6. System B — Event Sourcing

> Mirror the structure of §5 so the two systems read symmetrically.

### 6.1 Conceptual basis
**Contains:** Events as the source of truth; state derived by replay; CQRS separation of
write model and read model. Contrast with the outbox's "state-plus-outbox" model.

### 6.2 Write model — aggregates and the event store
**Contains:** The Axon 5 setup with a PostgreSQL-backed JDBC event store (event, snapshot,
token, saga tables), Axon Server disabled, dedicated connection pool. The two event-sourced
aggregates (inventory item, order) and how a command rehydrates an aggregate from its stream,
validates, and appends new events.
**Figures/Tables:** Event-store schema table; command → aggregate → event diagram.

### 6.3 Process orchestration — the saga
**Contains:** The order-reservation saga as a process manager: it reacts to order creation,
reserves items sequentially across aggregates via commands, and compensates (releases) on
failure before failing the order. Explain why orchestration replaces the single-transaction
reservation used by TO, and the consequences (eventual consistency, compensation).
**Figures/Tables:** Saga state/sequence diagram including the compensation path.

### 6.4 Read model — projections (CQRS)
**Contains:** Tracking event processors that build the query-side projections (inventory and
order tables) idempotently via a per-aggregate revision guard, and the resulting
read-after-write/projection-lag implications. Tie back to the same query API as TO.

### 6.5 Event delivery
**Contains:** The mock-Kafka tracking processor as a separate subscriber on the event stream,
recording publish latency — the ES counterpart to the TO relay, on the same mocked broker.

### 6.6 Concurrency control
**Contains:** Aggregate-level optimistic concurrency in the event store (concurrency
exception on conflicting append) and the retry strategy; contrast with TO's row-version
optimistic locking.

### 6.7 State-reconstruction strategy — the variation point
**Contains:** Introduce that all ES variants share §6.2–6.6 and differ **only** in how
aggregate state is reconstructed before handling a command. Then one subsection per variant:

- **6.7.1 ES-1 — Pure event sourcing.** Full-stream replay on every command; no snapshot, no
  cache. The baseline whose cost grows with stream length.
- **6.7.2 ES-2 — Snapshotting.** Event-count-triggered snapshots; rehydration loads the latest
  snapshot plus the tail. Trade-off of snapshot write cost vs. replay cost.
- **6.7.3 ES-3 — Snapshots + aggregate cache.** Adds an in-memory aggregate cache so hot
  aggregates skip rehydration entirely; relate to the TO-4 read-cache idea as the analogous
  optimisation.

**Figures/Tables:** Comparative table of the three reconstruction strategies (what is read on
each command, expected cost driver, memory cost).

---

## 7. Cross-System Comparison of Mechanisms

**Contains:** A consolidating section that places the two architectures side by side along the
axes that matter for the thesis: where atomicity is achieved, how events reach consumers, how
contention is resolved, where state lives and how it is read, and the consistency model
(transactional vs. eventual). This section sets up the hypotheses tested in the results
chapter without yet presenting data.

**Figures/Tables:** Master comparison table (rows = TO-1…TO-4, ES-1…ES-3; columns = atomicity
mechanism, delivery mechanism, consistency model, primary cost driver, variation axis).

---

## 8. Variant-to-Experiment Mapping

**Contains:** A brief bridge to the evaluation chapter: which variants are compared against
which, and what each comparison is meant to isolate (e.g. within-TO delivery-mechanism
sweep; within-ES reconstruction-strategy sweep; cross-family TO vs. ES at matched
optimisation levels). Keep it to the mapping; the results themselves belong to a later
chapter.

**Figures/Tables:** Matrix of planned comparisons.

---

## 9. Scope, Simplifications, and Cross-References

**Contains:** Honest statement of what the implementations abstract away and why it is
acceptable for a controlled comparison — notably the mocked broker, the synthetic payload
inflation, and the narrow domain. **Observability/instrumentation and the load-testing
methodology are intentionally not described here**; cross-reference the dedicated
Methodology/Measurement chapter where the metric catalogue, Prometheus/Grafana stack, and k6
load profiles are defined.

---

### Drafting notes
- §5 can be assembled largely from the existing `system-overview.md`; §6 must be written new
  but should follow the same section order so the chapter is symmetric.
- Keep all per-variant descriptions parallel in structure (same subheadings, same order) so a
  reader can diff TO-1↔TO-4 and ES-1↔ES-3 at a glance.
- Every diagram should exist in both a TO and an ES form where the concept applies, to
  reinforce the controlled-comparison framing.
