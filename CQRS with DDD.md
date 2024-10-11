---
up: "[[CQRS]]"
---
#TRANSLATE 
## Synchronizing the two models

So far, you learned about the two separate models in CQRS:

- The `Command Model`, which focuses on the tasks and operations that introduce changes in the `Domain`
- The `Query Model`, which deals with how data and state are requested from the `Domain`

Needless to say, none of the enormous benefits of this approach would be of any value if the models cannot be kept in sync. However as the `Query Model` is essentially read-only, the synchronization only needs to go in one direction. It can be implemented in three conceptually different approaches:

Use the same storage infrastructure _(same database, same file storage, etc.)_

This approach introduces significant limitations. While your models are separate on the `Domain Model` level, they are essentially merged into the same `Model` on the infrastructure level. Optimizing the storage infrastructure for one model may have a negative impact on the other.

Synchronize on the infrastructure level _(stored procedures, triggers, etc.)_

This approach allows you to optimize infrastructure separately but couples your `Domain Model` to the replication/synchronization functionality of the storage solutions. It may or may not be possible to synchronize between different storage. What is even worse is that you may have to move or duplicate some of the `Domain` logic to the infrastructure layer and then maintain it in both places.

Synchronize via `Events`:

In this approach, all the synchronization logic remains in the `Domain Models` which are completely decoupled from the underlying storage infrastructure.

## Benefits of CQRS

The following is a summary of CQRS benefits:

Simpler models:

As each `Model` only consists of the objects, functionality, and invariants that are relevant to one of the two purposes _(making changes in the `Domain` or requesting information from the `Domain`)_, designing, implementing, and reasoning the respective `Domain Models` are easier and much more concise and coherent.

Data access flexibility:

For the same `Command Model`, you can have multiple `Query Models`, each optimized for different UIs, use cases, devices, audiences, and so on.

Independent evolution:

Each of the two `Domain Model` implementations can evolve and change independently without affecting the other.

Independent performance optimizations

Each of the two `Domain Model` implementations is free to choose the best storage solution for its purpose and optimize it accordingly without worrying about the impact it may have on other models.

Independent scalability

Each of the two `Domain Model` implementations can be independently scaled vertically and horizontally.