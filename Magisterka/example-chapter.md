# System Overview

## 1. Introduction and Roadmap

This chapter describes the two systems whose behaviour under load forms the empirical basis of
this thesis. Both implement the same functional service — an inventory reservation service that
accepts customer orders and reserves stock against a catalogue of items — and both run on the
same infrastructure, exposed through the same interface and exercised by the same workload. They
differ in one respect only: the architectural style by which a change to application state is
made durable and announced to the rest of the system. The first system, hereafter the *TO*
system, realises this through the **Transactional Outbox** pattern; the second, the *ES* system,
through **Event Sourcing**. The purpose of the chapter is to establish, in enough detail to be
reproducible, that the two systems are alike in every dimension except the one under study, so
that the performance differences reported in later chapters may be attributed to the
architectural choice rather than to incidental differences of implementation.

Each architecture is not realised as a single program but as a family of incremental variants,
each variant adding one optimisation to its predecessor while leaving everything else unchanged.
The Transactional Outbox family comprises four variants, denoted **TO-1** through **TO-4**, which
share an identical write path and differ only in the mechanism by which recorded events are
delivered to consumers. The Event Sourcing family comprises three variants, denoted **ES-1**
through **ES-3**, which share an identical command and projection machinery and differ only in
the strategy by which an aggregate's state is reconstructed before a command is handled. This
naming convention — `TO-n` and `ES-n`, with the index increasing as optimisations accumulate —
is used throughout the remainder of the thesis.

The chapter proceeds as follows. Section 2 states the reference scenario and the functional
requirements common to both systems. Section 3 describes the reference architecture they share —
the technology baseline, the interaction model, the execution model, the mocked message broker,
and the deployment topology — so that this common ground need not be repeated for each system.
Section 4 makes explicit the methodological design that holds the two families comparable,
distinguishing the variables held constant from the variable under study in each family.
Sections 5 and 6 then describe the Transactional Outbox and Event Sourcing systems in turn,
following a deliberately parallel structure so that the two may be read side by side. Section 7
consolidates the comparison along the axes that matter for the thesis, and Section 8 maps the
seven variants onto the experiments that follow. Section 9 states honestly what the
implementations abstract away, and cross-references the separate chapter in which the measurement
methodology is defined.

---

## 2. Reference Scenario and Functional Requirements

The business domain shared by both systems is an inventory item reservation service. A catalogue
holds a set of *inventory items*, each carrying a finite quantity of available stock. Clients
place *orders*, and each order requests the reservation of a quantity of stock across one or more
items. An order progresses through a small lifecycle: it is first *pending*, and is subsequently
either *confirmed*, once every requested line has been satisfied, or *rejected*, carrying in that
case a categorised reason for the failure. The governing business rule is the avoidance of
overselling: stock may never be reserved beyond the quantity available, even when many orders
contend for the same item at the same instant.

The domain is deliberately minimal. It contains exactly enough structure to exercise the central
concern of the thesis — the reliable publication of domain events as a by-product of a state
change — and no more. In particular it is deliberately *contention-heavy*: the workload directs
many concurrent orders at a small catalogue, frequently at a single item, so that reservations
routinely collide on the same unit of state. This is a methodological choice rather than a
modelling one. By keeping the business logic trivial and the contention high, the measured
differences between the two systems arise from the mechanism by which each resolves that
contention and publishes the resulting events, rather than from the cost of any business
computation, which is negligible in both.

The functional requirements common to both systems are summarised in Table 2.1.

**Table 2.1 — Functional requirements (shared by both systems).**

| # | Requirement |
|---|---|
| F1 | An inventory item may be created with a given identifier and an initial available quantity. |
| F2 | The current available quantity of any item may be queried. |
| F3 | An order may be placed, requesting the reservation of a quantity across one or more items. |
| F4 | An order is confirmed only if every requested line can be satisfied from available stock. |
| F5 | An order is rejected, with a categorised reason, if any line cannot be satisfied. |
| F6 | Stock is never reserved beyond the available quantity, under any degree of concurrency (no overselling). |
| F7 | The outcome of an order may be queried after submission. |
| F8 | Every state change publishes a corresponding domain event exactly once to a downstream consumer, with no event lost and none emitted for a change that did not durably occur. |

*Figure 2.1 — Domain-concept diagram: the relationship between an inventory item, the
reservations held against it, and the order that requests them. (To be drawn.)*

---

## 3. Common Reference Architecture

This section describes everything the two systems hold in common. It is the backbone that makes
them comparable, and it is stated once here so that Sections 5 and 6 may concentrate solely on
what distinguishes each system.

### 3.1 Technology baseline

Both systems are single Spring Boot services written in Kotlin and running on the Java Virtual
Machine, each backed by a single PostgreSQL database. Persistence is synchronous in both: every
database interaction blocks the executing thread, and no part of the request path is reactive.
Schema evolution is managed by Flyway, and event payloads are serialised with Jackson. The two
systems are built from the same dependency set and run on the same JVM and database versions; the
technology baseline is summarised in Table 3.1.

**Table 3.1 — Technology baseline (shared).**

| Concern | Technology |
|---|---|
| Language / runtime | Kotlin on the JVM |
| Application framework | Spring Boot |
| Persistence access | Synchronous, blocking JDBC |
| Database | PostgreSQL (single instance) |
| Schema migration | Flyway |
| Event serialisation | Jackson (JSON) |
| Outbox / event-store infrastructure | Spring Modulith Event Publication Registry (TO) · Axon Framework 5 JDBC event store (ES) |
| Deployment | Single containerised service + PostgreSQL + load generator |

The one deliberate divergence within this baseline is the persistence infrastructure itself,
because it *is* the dimension under study: the TO system stores business state in relational
tables alongside a Spring Modulith outbox table, whereas the ES system stores an append-only
event log in an Axon-managed JDBC event store. Every other element of the baseline is identical.

### 3.2 Interaction model and API

Both systems expose the same small REST interface. Inventory items may be created and queried,
and orders may be placed and their status subsequently retrieved. The two write operations,
however, differ fundamentally in their interaction style, and this difference is central to the
systems' behaviour under load.

Item creation is *synchronous*: the HTTP response is returned only once the new item has been
durably committed, and it reports the created resource directly. Order placement, by contrast, is
*asynchronous* and follows a **command-and-poll** shape. A submitted order is merely *admitted*:
the service assigns the order an identifier, returns a `202 Accepted` response carrying that
identifier, and defers the reservation work to a background process. The client does not learn
the outcome from the submission response; instead it polls a status endpoint until the order is
reported confirmed or rejected. The shared REST surface is given in Table 3.2.

**Table 3.2 — REST interface (shared).**

| Method & path | Style | Response |
|---|---|---|
| `POST /inventory` | Synchronous | `201 Created` with the item |
| `GET /inventory`, `GET /inventory/{id}` | Synchronous | `200 OK` with item state |
| `POST /inventory/orders` | Asynchronous (command) | `202 Accepted` with the order identifier |
| `GET /inventory/orders/{orderId}` | Synchronous (poll) | `200 OK` with order status and any failure reason |

This command-and-poll structure is not an incidental design choice but a requirement of the
experimental method. It decouples the rate at which orders are *admitted* from the rate at which
they can be *processed*, so that the reservation pipeline — the component whose architecture is
under study — can be driven at arbitrarily high concurrency without the admission path itself
becoming the bottleneck. Both systems adopt it identically.

### 3.3 Request admission and execution model

Both systems operate three logically distinct groups of threads, sized so that the reservation
pipeline, rather than the web tier, is the limiting resource under load. A bounded pool of
web-server threads handles incoming HTTP requests; for orders these threads perform only the
brief admission step before returning. A worker stage, fed by a work queue, performs the
reservation work proper. And a small number of background threads drive the delivery of events
(the relay in TO, the tracking processors in ES).

The work queue feeding the worker stage is **unbounded** in both systems. The consequence is that
order submission never fails on the grounds of backpressure: surplus load is absorbed in memory
as growing latency rather than shed through rejection. This is a deliberate parity decision,
discussed further in Section 4, taken so that the two systems exhibit the same load-absorption
behaviour and their throughput may be compared meaningfully.

Underlying all three thread groups, a bounded pool of database connections mediates access to
PostgreSQL. It is this connection pool, and not the number of application threads, that ultimately
caps the number of transactions in flight at any instant. The pool is therefore the true
concurrency limit of each system, and is configured to the same effective size across both.

### 3.4 The mocked message broker

In both systems the downstream message broker to which events are ultimately destined is
*mocked* rather than real. The component that receives an event for publication serialises it and
records the fact and latency of its delivery, but does not transmit it across a network to an
external broker. This is a considered methodological choice. By removing the network and a real
broker from the measured path, the experiment isolates the cost of each architecture's *own*
event-handling machinery — the atomic recording and poll-driven relay of the outbox in one case,
the appending and tracking-processor consumption of the event log in the other — and holds the
downstream conditions identical across the two systems. Any difference that is then observed is
attributable to the architectures under study rather than to broker behaviour, queueing, or
network variance, none of which the thesis sets out to measure.

Crucially, the mock records a single, comparable quantity in both systems: the *publication lag*,
defined as the elapsed time between the logical creation of an event and the moment the mock
broker receives it for delivery. Because event-creation time is stamped from a single shared
clock (Section 4), this lag is measured on the same basis in both systems and constitutes one of
the primary points of comparison.

### 3.5 Deployment topology

A single variant is brought up as a small set of containers: the application service, its
PostgreSQL database, and the load generator that drives the workload. Only one variant runs at a
time; a run consists of starting the chosen variant against a freshly migrated database,
applying the workload, and collecting the recorded metrics. The two families share this topology
exactly, with the sole difference that the database plays a different role in each: in the TO
family it is the system of record for business state and additionally hosts the outbox table,
whereas in the ES family it hosts the append-only event store and the derived read-model tables.

*Figure 3.1 — Deployment topology, TO form: application service, PostgreSQL (business tables +
`event_publication` outbox), load generator. (To be drawn.)*

*Figure 3.2 — Deployment topology, ES form: application service, PostgreSQL (Axon event store +
projection tables), load generator. (To be drawn.)*

---

## 4. Design for Comparability

The validity of the comparison rests on a single methodological principle: that the two systems
differ in exactly one independent variable, with everything else held constant. This section
states explicitly what is controlled and what is varied, because it is this design that licenses
the causal claims made in the results chapter.

Several aspects of the implementations exist not to optimise either system in isolation but to
keep the comparison fair. They are the *controlled variables*, held identical across both
families:

- **A single shared clock as the latency origin.** In both systems the creation time of an event
  is stamped explicitly, at the moment the event is produced, from one shared system clock. In
  the TO system this is done by hand at the point of production; in the ES system Axon stamps the
  same instant at the moment the event is applied to its aggregate. Because publication lag is
  measured from this stamped instant, both systems use the identical notion of "when the event
  came into being", and the latency measurement is on a common footing.
- **Equalised serialized event sizes.** Domain events carry a configurable filler field that
  allows their serialized payload to be inflated to a chosen size. This is used to bring the
  on-the-wire size of TO events into line with that of ES events, so the two are evaluated at
  equal payload sizes rather than at sizes that happen to differ because of representational
  accidents.
- **An unbounded work queue (absorb, not shed).** As described in Section 3.3, the worker stage
  of each system is fed by an unbounded queue. Neither system rejects load; both degrade through
  latency. This equalises load-absorption behaviour so that throughput comparisons remain
  meaningful at saturation.
- **An identical mocked broker.** Both systems publish to the same mock described in Section 3.4,
  which records publication lag and performs no real I/O. The downstream is therefore removed as
  a source of variance in both.
- **An identical contention regime and harness.** Both systems are driven by the same workload,
  against the same small catalogue, through the same command-and-poll API, and both resolve the
  resulting write contention with optimistic concurrency control retried under an identical
  exponential-backoff schedule (an initial delay of twenty-five milliseconds, doubling on each
  attempt up to a five-hundred-millisecond ceiling).

Against this constant background, each family varies exactly one thing — its *independent
variable*:

- In the **Transactional Outbox** family, the independent variable is the **event-delivery
  mechanism**: how a recorded outbox event is conveyed to the consumer. The write path that
  records the event is identical across TO-1…TO-4.
- In the **Event Sourcing** family, the independent variable is the **state-reconstruction
  strategy**: how an aggregate's current state is rebuilt before a command is handled. The
  command, projection, and delivery machinery are identical across ES-1…ES-3.

Table 4.1 summarises the division.

**Table 4.1 — Controlled versus independent variables.**

| Held constant across all variants | Varied within a family |
|---|---|
| Domain, functional requirements, and REST API | TO: the event-delivery mechanism (polling → NOTIFY → after-commit → +read cache) |
| Shared clock as the latency origin | ES: the state-reconstruction strategy (full replay → snapshots → +aggregate cache) |
| Serialized event size (filler-equalised) | |
| Unbounded work queue (absorb, not shed) | |
| Mocked broker and publication-lag metric | |
| Workload, contention regime, and retry/backoff schedule | |
| Synchronous blocking JDBC; single PostgreSQL; bounded connection pool as the true concurrency limit | |

The cross-family comparison (TO versus ES) is necessarily a comparison of whole architectures
rather than of a single variable; it is meaningful precisely because every controlled variable
above is shared, so the architectures are compared on equal terms. Within each family, by
contrast, the comparison isolates a single mechanism.

---

## 5. System A — Transactional Outbox

### 5.1 Conceptual basis

A service that modifies its database and must also announce that modification to other components
faces the *dual-write problem*. If the database commit and the message publication are performed
as two independent operations, a failure occurring between them leaves the system inconsistent:
either the state changes without a corresponding event being emitted, or an event is emitted for
a change that was never durably committed. The Transactional Outbox pattern resolves this by
recording the outgoing event in the same database, within the same transaction, as the state
change it describes, and by delegating the actual delivery of that event to a separate relay
process that reads it from the database after the fact. The atomic database transaction is thus
made the single point at which both the state change and the intent to publish become durable —
they commit together or not at all — and the inherently unreliable act of delivery is moved out
of the critical path entirely.

The implementation described here is a faithful realisation of this pattern. It is built upon the
Event Publication Registry provided by Spring Modulith, a mature implementation of an outbox,
whose default delivery behaviour is replaced so that the pattern's properties are exhibited
exactly as intended and so that delivery may be varied independently of recording across the four
variants.

### 5.2 Domain model and persistence

The domain comprises three persistent entities. The central and most heavily contended is the
*inventory item*, which records the quantity of stock currently available together with a numeric
*version* used for optimistic concurrency control. The reservation rule is encapsulated within
the item: a request is rejected if its quantity is non-positive or exceeds the available stock,
and is otherwise satisfied by deducting the quantity and producing both a reservation record and
a domain event. Because the item is modelled as an immutable value updated by copying, this logic
is free of side effects; the persistence of the new state and the emission of the event are the
responsibility of the surrounding command handler. The second entity, the *reservation*, is a
line-level record asserting that a quantity of a particular item is held on behalf of an order.
The third, the *order*, represents the unit of work requested by a client and carries its
three-state lifecycle and, on rejection, a categorised reason.

Three domain events accompany these entities: one signalling the creation of an item, one the
successful reservation of a single line, and one summarising a confirmed order. Each carries a
correlation identifier and a creation timestamp stamped from the shared clock, as discussed in
Section 4.

The relational schema is established by a single Flyway migration and divides naturally into
business state and the outbox. The business state occupies three tables. `inventory_state` holds
one row per item — the identifier as primary key, the available quantity, a version column, and a
last-updated timestamp — with a check constraint forbidding a negative quantity as a final
database-level guard against overselling. `reservations` records the quantity reserved per item
per reservation, keyed jointly by item and reservation identifier and constrained by a foreign
key back to the item. `orders` records the user, status, and optional failure reason of each
order.

The outbox is the `event_publication` table, the storage structure of the Spring Modulith Event
Publication Registry. Each row represents a single event awaiting or having completed delivery
and stores, among other fields, a unique identifier, the listener and event types, the fully
serialised event payload as JSON text, the publication time, and — once delivered — the
completion time. A row is *pending* precisely while its completion time is null. Because the
entire payload is stored inline, the relay can reconstruct and deliver an event without consulting
the business tables. It is the co-location of `event_publication` with the business tables in one
database that makes the atomicity guarantee possible: a single transaction spans both the change
to business state and the insertion of the corresponding event row.

*Figure 5.1 — Relational schema: `inventory_state`, `reservations`, `orders`, and the
`event_publication` outbox table, with the inline serialized-event payload column highlighted.
(To be drawn.)*

### 5.3 Command and data flow

The processing of an order proceeds in two distinct phases, separated by the asynchronous
boundary of the command-and-poll API.

The first phase, *admission*, runs on the HTTP request thread. A unique order identifier is
generated, a pending order row is written and committed immediately, the reservation work is
handed to the worker pool, and the identifier is returned to the client with a `202 Accepted`.
Committing the pending order before any further work matters: it guarantees that the background
processor — and any concurrent status query — reliably observes the order's existence. Because
the worker queue is unbounded, admission never rejects an order.

The second phase, *reservation*, runs asynchronously on a worker thread and is wholly contained
within a single database transaction. The processor loads every item referenced by the order in
one query; applies the reservation rule to each line, obtaining a new item state, a reservation
record, and a reserved-item event for each; constructs and timestamps the summarising order
event; and then, still within the transaction, writes the updated item states (each subject to
its optimistic version check), the reservation records, and the transition of the order to
*confirmed*. Finally, and again within the same transaction, it records each domain event by
publishing it — which, as the next subsection explains, inserts the event into the outbox table
rather than delivering it. The transaction then commits, rendering state changes and event
records durable as one atomic unit. Should any step fail, the whole transaction is rolled back —
state and events alike — and the order is separately marked *rejected* with a reason reflecting
the cause.

```
HTTP thread                     worker thread (single transaction)
-----------                     ----------------------------------
POST /orders
  assign orderId
  INSERT order PENDING ─commit
  enqueue(orderId) ──────────►  load items
  return 202 (orderId)          apply reservation rule per line
                                build + stamp order event
                                UPDATE inventory_state (version-checked)
                                INSERT reservations
                                UPDATE order = CONFIRMED
                                INSERT events into event_publication
                                ─────────────────────────────── commit
                                       (on failure: rollback all,
                                        mark order REJECTED)
```
*Figure 5.2 — Sequence of the order write path in the TO system. The reservation phase is one
atomic transaction spanning state changes and the outbox insert.*

### 5.4 Concurrency control

Because all clients contend for a small set of items, and frequently for one item, concurrent
reservations routinely collide on the optimistic *version* of the item they modify, producing
optimistic-locking failures. The reservation method is therefore made retryable: a failed attempt
is retried with exponential backoff on the schedule fixed in Section 4 (an initial delay of
twenty-five milliseconds, doubling to a five-hundred-millisecond ceiling). So that the retry
interception is actually applied, the method is invoked through the framework's proxy rather than
as a direct internal call, which would bypass it. An order whose retries are exhausted without
success is rejected. This row-version optimistic locking, and the manner in which it copes with
the contended workload, is the TO system's answer to a problem the ES system solves differently
(Section 6.6), and the contrast is one of the principal subjects of the performance comparison.

### 5.5 Event delivery — the variation point

All four TO variants share the domain model, persistence, command flow, and concurrency control
described in Sections 5.2–5.4 without alteration. They differ **only** in how a recorded outbox
event is delivered to the consumer. The baseline, TO-1, is described in full; each subsequent
variant is then described as a delta upon its predecessor.

**5.5.1 TO-1 — Polling relay.** In the baseline, recording and delivery are completely decoupled.
A customised event multicaster replaces the framework default: when a domain event is published
during a business transaction, the multicaster writes its row into the outbox within that
transaction but, in deliberate contrast to the default, does *not* invoke the event's listeners
after the transaction commits. Publishing an event therefore has no immediate delivery
consequence; it serves only to enrol the event durably and atomically alongside the state change.
(The customised multicaster also bypasses an in-memory bookkeeping map used by the default
registry, which tracks publications by object identity and could never reconcile that identity
against the distinct object later reconstructed from JSON, leading to an unbounded accumulation of
stale entries.)

Delivery is the sole responsibility of a separate, scheduled relay. At a fixed interval the relay
drains the outbox: it repeatedly claims a batch of pending rows, ordered by publication time,
using a `SELECT … FOR UPDATE SKIP LOCKED` query bounded by a configurable batch size; for each
claimed row it reconstructs the event from its stored payload, delivers it to the mock broker,
and stamps the completion time; and it continues claiming batches until one returns fewer rows
than the limit, indicating the backlog is exhausted. The claim, delivery, and completion of a
batch all occur within a single transaction. The choice of `FOR UPDATE SKIP LOCKED` is what makes
the relay correct and efficient under concurrency: each claimed row is locked for the duration of
the transaction, its completion is committed before the lock is released, and any concurrent
drain simply skips already-locked rows. A delivered row is therefore never reselected, and two
concurrent relays never deliver the same event. The **poll interval directly bounds the latency**
between an event's creation and its delivery, and is the principal tunable parameter of the
mechanism. The mechanism provides *at-least-once* delivery: under normal operation each event is
relayed exactly once, but a failure after delivery yet before the completion commit would cause
redelivery on recovery, the customary guarantee of the pattern, which presupposes idempotent
consumers.

**5.5.2 TO-2 — PostgreSQL NOTIFY/LISTEN.** TO-2 retains the atomic recording of TO-1 unchanged
and adds a *push* path on top of the *pull* path, in order to collapse the poll-interval latency
bound. A database trigger fires `pg_notify` on the channel `event_publication_notify` whenever a
row is inserted into the outbox; because `NOTIFY` is delivered only when the inserting transaction
commits, the push path inherits the outbox's atomicity, and because the payload is merely the
row's identifier it stays clear of PostgreSQL's notification-size limit. A dedicated listening
component holds its own standalone JDBC connection — deliberately not one borrowed from the
application pool, so that the long-lived `LISTEN` session never disturbs the pool's sizing or
lifecycle — and, on each notification, hands the identifier to a small pool of delivery threads.
Each delivery first *claims* the row with an `UPDATE … SET completion_date = … WHERE id = ? AND
completion_date IS NULL`; a claim that updates zero rows means another path already delivered the
event, and is skipped. This claim makes delivery idempotent across the push and pull paths and
allows them to coexist safely. The polling relay of TO-1 is retained, but demoted to a *backup*
that sweeps any publication the push path missed (for example, notifications lost while no
listener was connected, which a catch-up pass also recovers on reconnection). TO-2 thus trades the
poll interval's fixed latency for near-immediate, event-driven delivery, at the cost of a
long-lived listening connection and a more elaborate delivery path.

**5.5.3 TO-3 — Synchronous after-commit dispatch.** TO-3 removes both the custom multicaster and
the explicit claim-and-deliver processor, reverting to Spring Modulith's default behaviour: the
default multicaster records each event in the outbox within the business transaction and then, in
the after-commit phase, delivers it directly to its listener. Delivery is therefore driven by the
request-processing path itself rather than by a relay or a notification, which yields the lowest
delivery latency of the family — an event is published essentially as soon as its transaction
commits. The polling relay is again retained as a backup that resubmits any publication left
incomplete (for instance because the process died between commit and delivery). The trade-off is
that delivery is now coupled to the processing path: the work of publishing is borne by the same
stage that performs the reservation, rather than being shed onto an independent background
process.

**5.5.4 TO-4 — After-commit dispatch with a read-path cache.** TO-4 is TO-3 augmented with a
read-side optimisation, leaving the delivery mechanism of TO-3 intact. A bounded in-memory cache
(Caffeine, with a maximum size and an idle time-to-live of five minutes) holds recently used
inventory item states. On the reservation load path, the handler first serves whatever items it
can from the cache and fetches only the misses from the database. After a successful commit, and
outside the transaction, the just-saved, version-incremented items are merged back into the cache
under a version guard that never overwrites a newer state with an older one, keeping the cache
monotonic so that a slow writer cannot drag it behind the database. On an optimistic-locking
conflict the affected keys are evicted, so that the retry re-reads fresh state from the database
rather than re-serving a stale cached version into the version-checked update — which would
otherwise guarantee the retry's failure. TO-4 therefore tests whether a read-side cache yields a
measurable benefit on the TO write path, and serves as the structural analogue, within the TO
family, of the aggregate cache introduced in the ES family (Section 6.7.3).

The four delivery mechanisms are compared in Table 5.1.

**Table 5.1 — The four TO event-delivery mechanisms.**

| Variant | Mechanism | Push / pull | Latency characteristic | Delivery guarantee | Failure recovery |
|---|---|---|---|---|---|
| TO-1 | Scheduled polling relay (`FOR UPDATE SKIP LOCKED`) | Pull | Bounded by the poll interval | At-least-once | The poll itself |
| TO-2 | `NOTIFY`/`LISTEN` push + claim, polling backup | Push (+pull backup) | Near-immediate on commit | At-least-once | Backup poller + reconnect catch-up |
| TO-3 | Synchronous after-commit dispatch, polling backup | Push (in-process) | Lowest; on the commit path | At-least-once | Backup poller |
| TO-4 | TO-3 + read-path cache | Push (in-process) | As TO-3 | At-least-once | Backup poller |

---

## 6. System B — Event Sourcing

This section mirrors the structure of Section 5 so that the two systems read symmetrically.

### 6.1 Conceptual basis

Event Sourcing inverts the relationship between state and events. Rather than storing current
state and emitting an event to describe each change, the system stores the *sequence of events*
as the sole source of truth and derives current state by replaying that sequence. There is no
dual write to reconcile, because there is only one write: appending an event to the log. The
dual-write problem that motivates the Transactional Outbox therefore does not arise here in the
same form; an event's durability and its existence as the record of a change are one and the same
act.

This shifts complexity elsewhere. Because the stored form is a log of events rather than a
queryable representation of state, the system separates its *write model* from its *read model* —
the principle of Command Query Responsibility Segregation (CQRS). The write model rebuilds an
aggregate from its event stream to validate and handle a command, appending new events; the read
model is a separate, derived projection of those events into queryable tables, maintained
asynchronously. Where the TO system kept a single authoritative representation of state and an
outbox beside it, the ES system keeps an authoritative event log and derives every other
representation — including the very tables the query API reads — from it. The implementation uses
Axon Framework 5 with a PostgreSQL-backed JDBC event store; the Axon Server component is disabled,
so that the event store is plain PostgreSQL and the comparison with the TO database is on equal
infrastructural terms.

### 6.2 Write model — aggregates and the event store

The write model comprises two event-sourced aggregates. The *inventory item* aggregate holds an
identifier and an available quantity; it handles a create command by applying an
*item-created* event, and a reserve command by checking the requested quantity against the
available quantity and applying either an *item-reserved* event or an *item-reservation-failed*
event, the latter carrying a reason. It also handles a release command — applying an
*item-reservation-released* event that returns stock — which exists to support compensation
(Section 6.3). Each event has a corresponding event-sourcing handler that mutates the in-memory
state: a reservation decrements the available quantity, a release increments it, a failure
changes nothing. The *order* aggregate holds the order identifier and its lines, and handles
create, complete, and fail commands by applying the corresponding order events.

The event store is a set of Axon-managed PostgreSQL tables. The central `domain_event_entry`
table stores one row per event, holding a monotonically increasing global index, the aggregate
identifier, the per-aggregate sequence number, the event type and payload, and the timestamp;
a uniqueness constraint on the pair (aggregate identifier, sequence number) is the mechanism by
which the store enforces optimistic concurrency on appends (Section 6.6). A `snapshot_event_entry`
table holds aggregate snapshots (used from ES-2 onward), a `token_entry` table holds the
positions of the tracking processors that consume the stream, and a pair of saga tables holds the
persistent state of the process manager (Section 6.3). Axon is given a *dedicated* JDBC connection
pool, separate from the application's primary pool, so that event-store access never contends with
the projection writes that draw on the application pool; the two pools together are sized to the
same effective concurrency limit used by the TO system.

Handling a command follows the event-sourcing cycle: the framework *rehydrates* the target
aggregate by reading its event stream from the store and replaying each event through the
aggregate's event-sourcing handlers to reconstruct current state; the command handler then
validates the command against that state and *appends* any resulting events to the store. It is
this rehydration step — what is read, and how much is replayed — that the three ES variants vary,
and Section 6.7 returns to it. To make the cost of the cycle observable, the event-storage engine
is wrapped so that it records the time spent reading a snapshot, the time spent fetching event
rows, the time spent replaying them in memory, and the time spent appending new events — the last
of these being the direct counterpart of the TO system's outbox-write timing, so that the two
systems' write costs may be compared on the same basis.

*Figure 6.1 — Command → aggregate rehydration (read stream, replay) → validate → append event,
against the `domain_event_entry` store. (To be drawn.)*

### 6.3 Process orchestration — the saga

The single-transaction reservation that the TO system performs in one worker thread cannot be
expressed the same way here, because the items of an order are separate aggregates, each with its
own independently appended event stream; there is no single transaction that can atomically
reserve several of them. The ES system therefore replaces the transaction with *orchestration*: a
saga (a process manager) coordinates the reservation across aggregates by issuing commands and
reacting to the resulting events.

The order-reservation saga starts when an order is created. It reserves the order's items *one at
a time*: it sends a reserve command for the first item, and on observing the corresponding
item-reserved event it advances to the next, until every line has been reserved, at which point it
sends a complete command to the order aggregate and ends. If any reservation fails — the saga
observes an item-reservation-failed event — it *compensates*: it sends a release command for each
item already reserved, returning that stock, and then sends a fail command to the order aggregate
before ending. To keep the saga's own processing thread from blocking on per-aggregate locks or
JDBC writes, the saga dispatches its commands onto a separate executor; the lock wait then happens
on a pool thread, and the saga merely awaits the resulting event in the stream. The saga
processor itself runs with parallelism across multiple segments so that distinct orders are
progressed concurrently.

This orchestration has two consequences that distinguish the ES system from the TO system at the
behavioural level. First, an order is satisfied by a *series* of separately committed reservations
rather than by one atomic transaction, so the system is *eventually consistent* with respect to an
order's completion. Second, partial failure must be repaired explicitly by *compensation* rather
than implicitly by transactional rollback. Both are intrinsic to the event-sourced, multi-aggregate
style and are not incidental to this implementation.

*Figure 6.2 — Saga state/sequence: start on order-created; loop reserve-per-item on each
item-reserved; on completion send complete; on item-reservation-failed, release each reserved
item and fail the order. (To be drawn, including the compensation path.)*

### 6.4 Read model — projections (CQRS)

The query side is built by *tracking event processors* that consume the event stream and maintain
ordinary relational projection tables — an inventory projection and an order projection — exposed
through the same query API as the TO system. Each processor reads events in order, recording its
position in the token table so that it resumes where it left off. The inventory projection upserts
the available quantity per item; the order projection writes the order's pending, confirmed, and
rejected states. Both write through a connection drawn from the Axon pool.

Idempotency is enforced by a *per-aggregate revision guard*: each projection row records the
sequence number of the last event applied to it, and an update takes effect only if the incoming
event's sequence number is greater. A processor that re-delivers an event after a restart
therefore cannot apply it twice, which makes the projection safe under the at-least-once delivery
of the tracking processors.

Because the projections are maintained asynchronously, the read model lags the write model by the
time it takes a processor to consume and apply an event — the *projection lag*, which the system
records explicitly. This has a visible consequence for the command-and-poll API: when an order is
admitted, its order-created event is appended synchronously, but the corresponding pending row in
the order projection is written only once the order projection processor catches up. A status poll
issued immediately after admission may therefore briefly observe no row at all, and will observe
the *confirmed* or *rejected* status only after the saga has completed and the projection has
advanced. This read-after-write delay is the price of CQRS and is one of the behaviours the
experiments quantify.

### 6.5 Event delivery

The counterpart of the TO relay is a further tracking processor — the mock-broker publisher —
subscribed to the same event stream as the projections but kept as a *separate* processing group
with its own token. For each event it records the publication lag against the shared-clock
creation time and performs no real I/O, exactly as in the TO system (Section 3.4). Delivery to the
mock broker is thus, in the ES system, simply another consumer of the log, rather than a distinct
relay reading a distinct outbox; but it is measured identically, so the publication-lag figures of
the two systems are directly comparable.

### 6.6 Concurrency control

Concurrency control in the ES system is enforced at the level of the aggregate by the event store
itself. Two commands that concurrently rehydrate the same aggregate will each attempt to append an
event at the same next sequence number; the uniqueness constraint on (aggregate identifier,
sequence number) allows only one to succeed, and the other fails with a concurrency exception.
This is the event-sourced analogue of the TO system's row-version optimistic lock — both are
optimistic, detecting conflict at write time rather than preventing it with a lock — but the unit
of conflict differs: a version column on a single state row in TO, versus the append position of
an event stream in ES. Conflicts are retried on the *same* exponential-backoff schedule fixed in
Section 4 (twenty-five milliseconds, doubling to five hundred), applied by a retry scheduler that
retries only genuine concurrency exceptions and gives up after a bounded number of attempts. The
retry schedule is thus held constant across the two systems, and only the conflict unit differs.

### 6.7 State-reconstruction strategy — the variation point

All three ES variants share the write model, saga, projections, delivery, and concurrency control
of Sections 6.2–6.6 without alteration. They differ **only** in how an aggregate's state is
reconstructed before a command is handled. As in Section 5.5, the baseline is described in full
and each subsequent variant as a delta.

**6.7.1 ES-1 — Pure event sourcing.** In the baseline, every command rehydrates its aggregate by
reading and replaying its *entire* event stream from the beginning. There is no snapshot and no
cache. This is event sourcing in its purest form, and its defining cost characteristic is that the
work of handling a command grows with the length of the aggregate's history: an item reserved
many thousands of times must replay all of those events before the next reservation can be
validated. ES-1 is therefore the variant against which the cost of history accumulation is
measured, and the baseline that the two optimisations below set out to relieve.

**6.7.2 ES-2 — Snapshotting.** ES-2 leaves rehydration semantically unchanged but bounds its cost
by introducing *snapshots*. After an aggregate has accumulated a configured number of events (set
here to fifty), the framework writes a snapshot capturing its state at that point. A subsequent
rehydration then loads the latest snapshot and replays only the events appended *after* it, rather
than the whole stream. (For the snapshot to be serialisable, the aggregate's fields are made
visible to the serializer.) This trades a periodic snapshot-write cost against a replay cost that
no longer grows without bound, and is expected to flatten the history-length dependence that
dominates ES-1. The snapshot threshold is the principal tunable parameter.

**6.7.3 ES-3 — Snapshots with an aggregate cache.** ES-3 retains snapshotting and adds an
in-memory *aggregate cache*, so that a *hot* aggregate — one recently used — is served directly
from memory and skips rehydration from the store entirely. The cache holds the live, already-
reconstructed aggregate; only a cache miss falls back to the snapshot-plus-tail load of ES-2. The
cache used here keeps strong references and is not evicted under memory pressure, which suits the
deliberately tiny catalogue (a handful of items) and ensures that, under sustained load on those
few items, the framework's snapshot counter does not reset and the hot aggregates are never
replayed from scratch. ES-3 is the structural analogue, within the ES family, of the TO read-path
cache of Section 5.5.4: in both, the final variant adds an in-memory cache of the contended state
to the path that would otherwise read it from the database on every operation.

The three reconstruction strategies are compared in Table 6.1.

**Table 6.1 — The three ES state-reconstruction strategies.**

| Variant | Read on each command | Expected cost driver | Memory cost |
|---|---|---|---|
| ES-1 | Entire event stream, replayed in full | Grows with stream length (history accumulation) | Minimal (no retained state) |
| ES-2 | Latest snapshot + events appended since | Bounded by the snapshot interval; periodic snapshot writes | One snapshot row per aggregate |
| ES-3 | Nothing on a cache hit; snapshot + tail on a miss | Cache hit rate; rehydration only on misses | Hot aggregates held in memory |

---

## 7. Cross-System Comparison of Mechanisms

With both systems described, their differences can be placed side by side along the axes that
matter for the thesis. The two architectures achieve *atomicity* in fundamentally different
places: the TO system makes the state change and the intent to publish atomic by committing them
in one database transaction, whereas the ES system has no dual write to make atomic, because the
single act of appending an event is itself the state change. They convey events to consumers
differently: TO records events in an outbox and *relays* them by one of four mechanisms, whereas
ES exposes a single event log that consumers — projections and the mock broker alike — *subscribe*
to. They resolve write contention with the same optimistic, retry-on-conflict discipline but on
different units: a version column on one state row versus the append position of an event stream.
They differ in where state lives and how it is read: TO keeps an authoritative, directly queryable
state and reads it back as such, whereas ES keeps an authoritative log and reads state either by
*replaying* it (write side) or from an asynchronously *derived projection* (read side). And they
differ in consistency model: the TO order is satisfied within a single transaction and is
therefore *transactionally consistent* at the moment of commit, whereas the ES order is satisfied
by an orchestrated series of separately committed reservations and is therefore *eventually
consistent*, with explicit compensation on partial failure.

These differences, and how each variant sits within them, are summarised in Table 7.1. The table
is intended to make the controlled-comparison design legible at a glance: reading down a column
shows what is held constant across a family, and reading across a row shows where the two families
diverge.

**Table 7.1 — Master comparison of the seven variants.**

| Variant | Atomicity mechanism | Delivery mechanism | Consistency model | Primary cost driver | Variation axis |
|---|---|---|---|---|---|
| TO-1 | State + event in one DB transaction | Scheduled polling relay | Transactional (per order) | Poll-interval latency; relay cost | Delivery |
| TO-2 | State + event in one DB transaction | `NOTIFY`/`LISTEN` push + backup poll | Transactional (per order) | Notification + claim cost | Delivery |
| TO-3 | State + event in one DB transaction | Synchronous after-commit dispatch | Transactional (per order) | Delivery on the commit path | Delivery |
| TO-4 | State + event in one DB transaction | After-commit dispatch + read cache | Transactional (per order) | Cache hit rate on the read path | Delivery |
| ES-1 | Single append (no dual write) | Tracking-processor subscription | Eventual (saga-orchestrated) | Full-stream replay length | Reconstruction |
| ES-2 | Single append (no dual write) | Tracking-processor subscription | Eventual (saga-orchestrated) | Snapshot interval; snapshot writes | Reconstruction |
| ES-3 | Single append (no dual write) | Tracking-processor subscription | Eventual (saga-orchestrated) | Cache hit rate on rehydration | Reconstruction |

This consolidation sets up, without yet presenting data, the hypotheses the results chapter tests:
that within the TO family the delivery mechanism trades latency against coupling to the request
path; that within the ES family reconstruction strategy trades replay cost against memory and
snapshot-write cost; and that across families the two architectures differ measurably in latency,
throughput, and their response to contention even when every other variable is held equal.

---

## 8. Variant-to-Experiment Mapping

The seven variants are exercised in three groups of comparisons, each isolating a different
effect. The *within-TO* sweep (TO-1 → TO-2 → TO-3, with TO-4 added) isolates the effect of the
delivery mechanism on publication latency and on the throughput of the write path, holding the
write path itself constant. The *within-ES* sweep (ES-1 → ES-2 → ES-3) isolates the effect of the
state-reconstruction strategy on command-handling cost as history accumulates, holding the rest of
the ES machinery constant. The *cross-family* comparison pairs the two architectures at matched
levels of optimisation — most directly the two cache-bearing endpoints, TO-4 and ES-3, but also
the unoptimised baselines TO-1 and ES-1 — to compare the architectures as wholes on equal terms.
Table 8.1 summarises the planned comparisons and what each is meant to isolate; the results
themselves belong to the evaluation chapter.

**Table 8.1 — Planned comparisons.**

| Comparison | Variants | Isolates |
|---|---|---|
| Within-TO delivery sweep | TO-1 vs TO-2 vs TO-3 | Effect of delivery mechanism on publication latency and write-path throughput |
| TO read-cache effect | TO-3 vs TO-4 | Marginal benefit of a read-path cache on the TO write path |
| Within-ES reconstruction sweep | ES-1 vs ES-2 vs ES-3 | Effect of snapshotting and aggregate caching on command cost under history growth |
| Cross-family baselines | TO-1 vs ES-1 | Architectures compared with no optimisation applied |
| Cross-family optimised | TO-4 vs ES-3 | Architectures compared at matched (cache-bearing) optimisation levels |

---

## 9. Scope, Simplifications, and Cross-References

The implementations abstract several things away, deliberately, and it is worth stating which and
why the abstraction is acceptable for a controlled comparison. The *message broker is mocked*:
no event leaves the process for a real broker, and publication is measured as a lag rather than as
end-to-end delivery to a consuming system. This is acceptable, indeed necessary, because the
thesis measures the cost of each architecture's own event-handling machinery, not the behaviour of
a broker, and mocking removes the broker identically from both systems. The *event payloads are
synthetically inflated* by a filler field; this is acceptable because its sole purpose is to
equalise serialized sizes across the two systems, and the same mechanism is applied to both. The
*domain is narrow* — a handful of items and a single business rule — and the catalogue is
deliberately small; this is acceptable because the narrowness is what concentrates the measurement
on the event-handling mechanism rather than on business complexity, and the same domain is
implemented identically on both sides.

Two topics are intentionally *not* described in this chapter. The observability and
instrumentation of the systems — the catalogue of metrics, the Prometheus and Grafana stack that
collects and visualises them, and the exact definitions of the timers referenced in passing above
(publication lag, projection lag, state-load and state-persist timings) — belong to the
Methodology and Measurement chapter, as does the load-testing methodology, including the workload
profiles, concurrency levels, and run protocol used to drive the systems. They are deferred there
so that this chapter may remain a description of the systems themselves, and the reader is
referred to that chapter for the manner in which the systems described here are measured.
