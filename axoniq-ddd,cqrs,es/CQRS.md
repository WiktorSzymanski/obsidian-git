# CQRS Definition

CQRS is an abbreviation of **Command Query Responsibility Separation**, an architectural pattern that, as its name implies, separates a system into two different areas:

- One that deals with the operations that, when executed, produce changes in the system. These operations are called the `Commands`.
    
- One that takes care of the operations that are only requesting information. These operations are called the `Queries`. They are not making any changes in the system.
    

In short and a little more simply, CQRS implies considering the writes and the reads from our system independently.

### Examples

Product ratings in an online store:

On your command side, you may have operations that add a new rating from a customer, modify an existing rating, remove a rating, etc.

On your query side, you may have operations that return the average rating of a given product, the top-rated products in a given time interval, all the ratings a given customer has provided, etc.

Flight booking system:

On your command side, you may have operations that enable the user to book a flight, cancel a booking, update the booking’s data, etc.

On your query side, you may have operations that return the airline’s terms, baggage allowance, seat availability, etc.

# `Command`

The term `Command` in CQRS refers to an operation that is expected to trigger a change in the `Domain`.

A `Command` should be modeled from the point of view of the operation that it represents, with all the necessary information, but nothing more, for the system to execute the operation.

When combining CQRS with DDD, `Commands` are typically dispatched to `Aggregates`, which are responsible for making the changes in the `Domain`. Thus it is often the behavior of the `Aggregates` that defines what `Commands` should exist in the system.

### Examples

In a flight booking system:

You would likely need to model behaviors such as booking a flight, canceling a booking, updating booking data, etc.

Therefore, you would likely need commands that trigger such behavior - for example, `BookFlight`, `CancelBooking`, `UpdateBooking`, etc.

In a product rating system

You would likely need to model behaviors such as adding a rating from a customer, modifying existing rating or removing a rating, etc.

Therefore, you would likely need commands that trigger such behavior - for example, `AddRating`, `UpdateRating`, `RemoveRating`, etc.

In each case, the exact commands and their data will depend on the relevant business operations that you need support in your system.

# `Command Model`

The term `Command Model` refers to a `Domain Model` that is optimized for and contains everything that is needed for executing `Commands`.

While designing a `Command Model`, you only need to focus on behavior - the operations that result in changes in the `Domain`. Therefore, it typically consists of all `Commands`, the `Aggregates` that can execute them, along with all relevant `Entities` and `Value Objects`. Because it needs to inform the rest of the system, it usually also contains the `Events` that are used to notify other components _(especially those in the `Query Model`)_ about the changes made as a result of processing commands.

While the `Command Model` must ensure all stateful components can be properly persisted and loaded, it should not take into consideration how the data will be stored for display purposes later on.

### Examples

In a flight booking system the `Command Model` will likely consist of:

- `FlightBooking` and other relevant `Aggregates`
- `Flight`, `Booking`, `Leg`, and other `Entities`
- `Origin`, `Destination`, `DepartureDateTime`, `ArrivalDateTime`, and other `Value Objects`
- `BookFlight`, `CancelBooking`, `UpdateBooking`, … `Commands`
- `FlightBooked`, `BookingCanceled`, `BookingUpdated`, and other `Events`

In a product rating system, the `Command Model` may consist of:

- `ProductRating` and other relevant `Aggregates`
- `Product`, `CustomerRating`, `Category`, and other `Entities`
- `RatingDateTime`, `IpAddress`, and other `Value Objects`
- `AddRating`, `UpdateRating`, `RemoveRating`, and other `Commands`
- `RatingAdded`, `RatingUpdated`, `RatingRemoved`, and other `Events`

# `Query`

The term `Query` in CQRS refers to a request to retrieve information or state from the `Domain`.

A `Query` should be modeled from the point of view of the information it requests. It should contain all the necessary information, but nothing more, for the system to find or calculate it and provide a response. Depending on your needs, that may include things like:

- Which objects should be taken into account and which should be filtered out
- How the results must be ordered
- Which object details should be included in or excluded from individual results
- Whether it should contain all the results or just a portion (a “page”) of them
- And so on.

The essential characteristic of a `Query` is that it can only request data from the `Domain`. It can not make any changes in the `Domain`.

### Examples

In a flight booking system:

You would likely need to show booking details, information about the given flight, a list of matching flights, etc.

Therefore, you would likely need queries that request this information - for example, `BookingDetailsQuery`, `FlightInformationQuery`, `MatchingFlightsQuery`, …

In a product rating system

You would likely need to show the average rating of a product, the number of ratings, all ratings from a given customer, etc.

Therefore, you would likely need queries that request this information - for example, `AverageProductRatingQuery`, `RatingsCountQuery`, `CustomerRatingsQuery`, …

In each case, the exact queries and their data will depend on the information relevant to your users.

# `Query Model`

The term `Query Model` refers to a `Domain Model` that is optimized for and contains everything needed for processing `Queries`.

The `Query Model` typically consists of two sets of components that are directly related to each other:

- The information storage and retrieval components known as `Projections`, which are responsible for providing the means for data to be extracted or generated from relevant `Events` and stored according to how it will be queried and used.
- The `Query` interpretation and processing components, which are responsible for retrieving the respective data and preparing a response in the form requested/

The entire `Query Model` is essentially read-only. You have the flexibility to adapt the design of your data storage to scale and optimize it as needed. You are free to de-normalize data, apply the table-per-view concept, have multiple storages for the same data, or use any other optimization technique you see fit.

### Examples

In a flight booking system, the `Query Model` will likely consist of:

- `FlightProjection` and other relevant `Projections`
- Infrastructure components that interact with appropriate storage _(RDBS, noSQL DB, file system, etc.)_
- Domain components with `handleBookingDetailsQuery`, `handleFlightInformationQuery`, `handleMatchingFlightsQuery`, and other functions
- `BookingDetailsQuery`, `FlightInformationQuery`, `MatchingFlightsQuery`, and other `Queries`

In a product rating system, the `Query Model` may consist of:

- `ProductRatingProjection` and other relevant `Projections`
- Infrastructure components that interact with appropriate storage _(RDBS, noSQL DB, file system, etc.)_
- Domain components with `handleAverageProductRatingQuery`, `handleRatingsCountQuery`, `handleCustomerRatingsQuery`, and other functions
- `AverageProductRatingQuery`, `RatingsCountQuery`, `CustomerRatingsQuery`, and other `Queries`

# Synchronizing the two models

So far, you learned about the two separate models in CQRS:

- The `Command Model`, which focuses on the tasks and operations that introduce changes in the `Domain`
- The `Query Model`, which deals with how data and state are requested from the `Domain`

Needless to say, none of the enormous benefits of this approach would be of any value if the models cannot be kept in sync. However as the `Query Model` is essentially read-only, the synchronization only needs to go in one direction. It can be implemented in three conceptually different approaches:

Use the same storage infrastructure _(same database, same file storage, etc.)_

This approach introduces significant limitations. While your models are separate on the `Domain Model` level, they are essentially merged into the same `Model` on the infrastructure level. Optimizing the storage infrastructure for one model may have a negative impact on the other.

Synchronize on the infrastructure level _(stored procedures, triggers, etc.)_

This approach allows you to optimize infrastructure separately but couples your `Domain Model` to the replication/synchronization functionality of the storage solutions. It may or may not be possible to synchronize between different storage. What is even worse is that you may have to move or duplicate some of the `Domain` logic to the infrastructure layer and then maintain it in both places.

Synchronize via `Events`:

In this approach, all the synchronization logic remains in the `Domain Models` which are completely decoupled from the underlying storage infrastructure.

# Benefits of CQRS

The following is a summary of CQRS benefits:

Simpler models:

As each `Model` only consists of the objects, functionality, and invariants that are relevant to one of the two purposes _(making changes in the `Domain` or requesting information from the `Domain`)_, designing, implementing, and reasoning the respective `Domain Models` are easier and much more concise and coherent.

Data access flexibility:

For the same `Command Model`, you can have multiple `Query Models`, each optimized for different UIs, use cases, devices, audiences, and so on.

Independent evolution:

Each of the two `Domain Model` implementations can evolve and change independently without affecting the other.

Independent performance optimizations

Each of the two `Domain Model` implementations is free to choose the best storage solution for its purpose and optimize it accordingly without worrying about the impact it may have on other models.

Independent scalability

Each of the two `Domain Model` implementations can be independently scaled vertically and horizontally.