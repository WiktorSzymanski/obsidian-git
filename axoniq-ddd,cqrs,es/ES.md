# Not the same as Event Streaming

## What is Event Streaming

**Event Streaming** is gets `Events` from one component to another. Usually, it consists of temporary storage for the events and a “read forward” procedure. The combination of the two allows the consuming component to consume the messages at its own pace, and it can even replay everything if it wants. Event Streaming is unbounded; it has no real beginning nor end. Each `Event` is processed as it occurs.

Example: The stock market

Every time a stock price changes, a new message is created with information related to the stock item _(the time of day, its code, and its new price, etc.)_. Since an exchange can have thousands of individual stocks and handle many trades per day, the result is a constant stream of events produced by one system and consumed by others.

## How is Event Sourcing different

**Event Sourcing** is not about getting events from one place to another. **It is about using the past `Events` to make new decisions for future `Events`.**

Of course, as a side effect, you can also use that same event stream for Event Streaming.

# Event Sourcing

Event Sourcing is a pattern for data storage. Instead of persisting the current state of any `Entity`, it stores all past changes to the state in the order they occurred. When the `Entity` is needed, it is reconstructed by reapplying all the changes made to it in the past.

## Why add Event Sourcing to the mix

Event Sourcing is typically used with CQRS and DDD because it fits naturally into the `Command Model` where all state changes are made. When the three patterns are applied together, `Events` becomes first-class citizens of the application. They:

- Represent all changes ever made in the `Domain`.
- Are stored in an `Event Store` and never removed.
- Are immutable.
- Are the only source causing `Projections` to update in the `Query Model`.
- Are the only source from which `Aggregates` and `Entities` build their state in the `Command Model`

**That makes the `Events` the single source of truth.**

## Additional benefits of Event Sourcing

- It simplifies the modeling process _(the domain events you identify dictate the rest)_.
- It reduces development work _(no need to implement state storages)_.
- It provides greater fault-tolerance because it is possible to recover the state of the system at any point.
- It collects extensive data that you can use for analysis, predictions, etc.

## The decision to use Event Sourcing

Event Sourcing comes at price _(more on that in the next lesson)_. The decision to use it or not should be a result of evaluating your business and technical reasons. Typically, these reasons are:

- Auditing - it provides reliable audit trace. The application is as transparent as possible.
- Analytics - the detailed information captured by storing all the changes can reveal valuable insights and patterns.
- Comprehensiveness - a series of small changes is a model that is closer to the story-telling nature of humans.
- Technical reasons - it is easier to evolve and adapt the system having the complete history.
# `Event Store`

The term `Event Store` refers to an infrastructure component that stores `Events`. However, not every mechanism capable of storing events is an event store.

By design, an `Event Store` should efficiently read all events only by filtering the unsuitable events and without executing a full scan.

An `Event Store` allows clients do the following operations:

- **Append new `Events`**
    
    Append is the only changing operation that `Event Store` must implement. While appending events, the `Event Store` must be capable of recognizing conflicting changes. It is also recommended that it be capable of validating the sequence per `Aggregate` or `Entity`.
    
- **Perform full sequential read**
    
    Starting from one point, it reads every `Event` from that point onwards. The reader has the opportunity to handle the `Events` and as it handles them, it receives additional `Events`. It can decide how fast it goes through the `Event Store`.
    
- **Read aggregates’ events**
    
    A client must be able to read `Events` affecting only specific `Aggregate` or `Entity` in order to reconstruct its current state.


# Performance challenges

## Ever-growing data set

You have seen that when using Event Sourcing, the history of everything that happened in the application is recorded as `Events`. Because they are now the only source of truth, you cannot afford to lose any part of history. That introduces a non-trivial technical challenge: how do you efficiently store an ever-growing data set?

In the case of many general-purpose data storage solutions, performance degrades as the amount of data they keep increases or crosses certain thresholds. As the operations of appending events must be atomic and ordered from the client’s perspective, that would also slowly degrade the performance of your application.

The solution is to use a built-for-purpose `Event Store`. Axon Server is one such store, but there are others. While it may be possible to configure and scale some of the general-purpose data storage to meet your requirements, it may not be worth the effort.

## Efficient data retrieval

Since you must constantly reconstruct the state of an `Event-Sourced Aggregates` from past events, the obvious concern is how large that data is and how fast you can retrieve it.

- If you are not using a built-for-purpose `Event Store`, you must have very carefully designed indexes in place.
    
- If you are designing a distributed application, you should seriously consider using a consistent hashing algorithm. The goal is to ensure that all `Commands` you send to the same `Aggregate` are handled by the same application instance or cluster node. This approach will allow you to safely cache the `Aggregate` or introduce other valuable local optimizations.
    
- In all cases, when you expect to deal with `Aggregates` with a long history, you should consider using `Snapshots` - stored states of an `Aggregate` at particular points in time. You can then load the most recent `Snapshots` and apply only those `Events` that occurred later. `Snapshots` are a technical optimization that is not part of the `Domain Model`. You can delete them and recreate them without changing the system’s behavior.

# Fixing immutable data

So you stored the whole history in the form of `Events`, and those are immutable. What if the data you have already persisted is wrong? How do you fix a mistake in immutable data?

With traditional systems, you could go into the database and fix it. Thus you may think you can do the same in event-sourced applications. That is particularly tempting if you use a general-purpose storage system for your events that allows you to modify data. However, you must not do this, even if it’s technically possible. Altering `Event` data at rest may have disastrous consequences. You do not know what actions the application has already performed based on the old value, and you can not undo them by modifying the data at rest.

The proper way to deal with mistakes and incorrect data is by adding compensating `Events`. Acknowledge the error, execute operations that correct it, and append the resulting `Events` in the history. This is the only way to ensure that your history contains the truth, the whole truth, and nothing but the truth.

# The impact areas of `Events`

In the context of event-sourcing, you think of `Events` as history storage mechanisms. In the context of CQRS, you think of `Events` as mechanisms for transferring data from the `Command Model` to the `Query Model`. In event-driven systems, the `Events` are general-purpose notification mechanisms. Naturally, you may start to wonder if those can be the same events or you should have different ones in each context?

## Inside vs outside `Events`

While there is no widely accepted policy and rules about how to scope your `Events`, it helps to separate them into two groups

- **Inside events**
    
    These are the events that only make sense in context. It may be the `Aggregate` alone or the components within given `Bounded Context`. It is generally safe to share detailed information in a `Bounded Context`, and thus such events typically contain many details and occur often. These are the `Events` from which `Aggregates` reconstruct their states.
    
- **Outside events**
    
    These are the events that carry information across contexts. It is usually unnecessary to share detailed information across different `Bounded Contexts`, and thus such events typically contain only the relevant shared information and occur rarely.
    

It is acceptable practice to use the same event for different purposes as long as you keep the above distinctions in mind.

# Dealing with sensitive data

Another challenge that stems from the fact that `Events` are immutable is removing sensitive data. Unlike the error-fixing scenario, compensating `Events` cannot help here. On the other hand, privacy regulations could not care less whether your `Events` are immutable or not. Does that mean that you cannot use event-sourcing when such regulations apply? Of course not.

There are two ways to handle sensitive data:

- **Out-of-band storage**
    
    You store any sensitive information in an external system and not inside the `Events`. Instead of the actual sensitive information, your `Events` contain a reference to the external record. At any time, you can delete that information, leaving in the `Event` a reference to a non-existing record.
    
- **Crypto erasure**
    
    You can encrypt the sensitive data using one or more keys stored in a safe place outside the `Events`. At any time, you can delete any given key, which will make it impossible to retrieve the encrypted information.
    

In both cases, you should design your system to remain fully operational when sensitive data is no longer available. It is vital to keep that in mind!

# Changing the structure of the `Events`

As systems evolve, the structure of some `Events` may need to change. You may need to add fields, remove fields, or change a field type. While you can update the objects representing the `Events`, you cannot change the structure of the already stored `Events`.

Keep in mind that the `Event Store` usually contains some serialized representation of the `Events` and not Java classes. Thus it may or may not be possible to map the actual data to the new object structure. Therefore, your `Aggregates` and `Event Handlers` will likely have to work not only with the new structure but also with all previous structures.

There are two ways to deal with this challenge:

- **Backward and forward compatibility**
    
    You can establish a policy never to change specific fields with high importance. Then you make sure to treat everything else as optional. Design your `Event Handlers` not to depend on the optional fields by providing conditional logic or default values. Once this is in place, you have backward and forward compatibility, which provides a significant degree of freedom.
    
- **Upcasters**
    
    `Upcasters` are infrastructure components that operate in the space between the `Event Store` and the `Event Handler`. They are responsible for transforming a specific revision of a stored `Event` _(one read from the store or already transformed by another upcaster)_ to a more up-to-date revision of the same `Event` before passing it to the `Event Handler`.

# Dealing with infrastructural complexity

Building applications following the DDD, CQRS, and Event Sourcing architectural patterns comes with many benefits. However, as you’ve seen, there are plenty of challenges that require rather complex infrastructure components. There is no easy workaround without those. So, to be successful with those patterns, you have two options

## Use built-for-purpose platform

Axon is one such solution that provides a solid messaging platform, abstractions of the terms defined in the above patterns, multiple infrastructure components, and many more. It has out-of-the-box solutions for all the challenges mentioned in this course. Moreover, it makes it easy to build solutions with `Location Transparency` in mind so that you can start with a well-structured monolith and effortlessly evolve into micro-services.

Of course, there are other solutions out there as well. Please do your research taking into account the features and requirements your particular project has.

## DIY infrastructure

While using a built-for-purpose platform is by far the most reliable and cost-effective way, you may have good reasons and the resources to build your custom infrastructure to support DDD, CQRS, and Event Sourcing. If that is the case, the following capability map will help you keep track of all the functionalities that your solution should provide:

![](https://axoniq.github.io/academy-content/assets/img/capability_map.png)