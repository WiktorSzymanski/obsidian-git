---
up: "[[CQRS]]"
---
#TRANSLATE 
# Command
---
The term `Command` in CQRS refers to an operation that is expected to trigger a change in the `Domain`.

A `Command` should be modeled from the point of view of the operation that it represents, with all the necessary information, but nothing more, for the system to execute the operation.

When combining CQRS with DDD, `Commands` are typically dispatched to `Aggregates`, which are responsible for making the changes in the `Domain`. Thus it is often the behavior of the `Aggregates` that defines what `Commands` should exist in the system.

### Examples
---
In a flight booking system:

- You would likely need to model behaviors such as booking a flight, canceling a booking, updating booking data, etc.

- Therefore, you would likely need commands that trigger such behavior - for example, `BookFlight`, `CancelBooking`, `UpdateBooking`, etc.

In a product rating system

- You would likely need to model behaviors such as adding a rating from a customer, modifying existing rating or removing a rating, etc.

- Therefore, you would likely need commands that trigger such behavior - for example, `AddRating`, `UpdateRating`, `RemoveRating`, etc.

In each case, the exact commands and their data will depend on the relevant business operations that you need support in your system.