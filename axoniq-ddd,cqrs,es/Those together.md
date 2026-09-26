# `Message` Driven Architecture

In this lesson, you have learned how to use DDD, CQRS, and Event Sourcing together to build your system.

It is possible and relatively easy to use DDD, CQRS, and Event Sourcing to build your system when you allow independent parts of it to communicate with each other via `Messages`. That simple technical approach brings you the benefits of these architectural concepts:

- Preserving and guarding the boundaries of components
- Moving components to different physical locations or even replacing them
- Independently updating components without breaking other parts of the system
- Having a complete history of everything that happened in the system and use it for decision-making purposes

Not all `Messages` are the same. Most importantly, not everything can be an `Event` like many Event-driven solutions claim. Knowing the different types of `Messages` helps you map the above patterns to meaningful software interactions and implement proper routing patterns in your system.

## `Event messages`

`Event messages` are `Messages` that carry an `Event` - a notification that something important has happened in the `Domain`. They are not synchronous and unable to communicate back with the publisher. Every component interested in a given `Event` receives it and acts accordingly. The publisher cannot target a specific receiver and can not know how, when, and by who given `Event` is processed.

When you create the objects that represent`Events`, you should use past tense. For example `FlightScheduled`, `FlightCanceled`, etc.

## `Command Messages`

`Command Messages` are `Messages` carrying a `Command` - an expression of intent to change something in the `Domain`. They travel to a single destination. Even though there may be multiple components that could process a `Command`, only one of them will receive the `Command Message`.

`Command Messages` can be dispatched and processed both synchronously and asynchronously. The receiving component may return a response to:

- Acknowledge the command will be executed.
- Provide an identifier for future requests.
- Indicate an error that makes it impossible to process the command.

When you create the objects that represent `Commands`, you should use an imperative present tense. For example `ScheduleFlight`, `CancelFlight`, etc.

## `Query Messages`

`Query Messages` are `Messages` that carry a `Query` - a request for information or state from the `Domain`. You can send them to one or more destinations. Their main characteristic is that the value is in the response. The client will usually be waiting for the response to arrive. It is also possible to merge responses from several components before passing the information to the requester. You can even subscribe to receive updates when the requested information changes in the future.

When you create the objects representing `Queries`, you should name them after the type of information they request. For example `FlightSchedule`, `FlightStatus`, etc.


# `Location Transparency`

`Location Transparency` is the concept that mandates that a component does not need to know where another element resides to communicate with it.

This architectural concept is essential when your system grows and becomes more complex. It is a real lifesaver when you need to scale the system and convert its parts into independently deployable units. If the system is designed with `Location Transparency` in mind, the process is almost trivial. Selected parts running initially on the same JVM may be moved to another JVM on the same machine/container or on a separate machine/container without any code changes.

Combining DDD, CQRS, and Event Sourcing draws the boundaries of the components and ensures there is a single source of truth. Implementing them using Axon `Commands`, `Events`, and `Queries` messaging concepts gives you `Location Transparency`. Various subsystems and components communicate using `Messages` and rely on a `Message Bus` to deliver them to the proper component. The benefits of applying the architectural concepts remain intact regardless of what deployment practice you choose to use.

# System Evolution

Over time, that composition of systems has evolved from monolithic units to micro-services. Monoliths are long-lasting but problematic. Let’s take a closer look at the evolution.

## Monolith

![](https://axoniq.github.io/academy-content/assets/img/dep_monolith.png)

Monoliths are applications that are deployed as a single unit. There is nothing wrong with that. Such applications have existed for a very long time, and many will remain this way for years to come.

As the application grows, more connections and dependencies between components are introduced. With time, it becomes challenging to modify one part of the system without breaking another.

## Modular monolith

![](https://axoniq.github.io/academy-content/assets/img/dep_modular_monolith.png)

Modular monoliths are also applications deployed as a single unit. The significant difference is that their components are organized into modules with well-defined boundaries. There are multiple ways to do it - from naive to over-engineered ones. DDD, CQRS, and Event Sourcing architectural patterns exist to help solve that problem. They teach you how and where to boundaries, preserve invariants, separate different concerns, establish a single source of truth, etc.

The challenge here is to design the communication between the modules. As the system evolves, tight coupling between modules reveals its price. Despite the well-organized structure, it may be tough to extract individual modules as independently deployable units.

## Location transparent modular monolith

![](https://axoniq.github.io/academy-content/assets/img/dep_location_transparent_monolith.png)

`Location Transparency` is a design practice that mandates the use of names to identify components rather than their actual location. Applying it to a modular monolith prevents the tight coupling between modules. This gives you the freedom to extract and scale individual components without modifying the rest of the system.

This is where **Axon Framework** really shines. It embraces DDD, CQRS, and Event Sourcing principles and provides the essential building blocks and infrastructure. On top of that, it ensures `Location Transparency` in all communications between components via well-defined types of `Messages` and a smart enough `Message Bus`.

## Micro-services

![](https://axoniq.github.io/academy-content/assets/img/dep_microservices.png)

With DDD, CQRS, Event Sourcing, and `Location Transparency` in place converting a monolith to micro-services becomes a trivial task. Components already have well-established boundaries, the roles they play are clear, there is a single source of truth, and communication does not depend on the deployment architecture. To start extracting parts of the system as independently deployable units, you only need two things - a message router and an event store!

Here’s the good news: **Axon Server** is precisely that - a zero-configuration message router and event store.