---
up:
---
#TRANSLATE 
# Command Model
---
The term `Command Model` refers to a `Domain Model` that is optimized for and contains everything that is needed for executing `Commands`.

While designing a `Command Model`, you only need to focus on behavior - the operations that result in changes in the `Domain`. Therefore, it typically consists of all `Commands`, the `Aggregates` that can execute them, along with all relevant `Entities` and `Value Objects`. Because it needs to inform the rest of the system, it usually also contains the `Events` that are used to notify other components _(especially those in the `Query Model`)_ about the changes made as a result of processing commands.

While the `Command Model` must ensure all stateful components can be properly persisted and loaded, it should not take into consideration how the data will be stored for display purposes later on.

### Examples
---
In a flight booking system the `Command Model` will likely consist of:

- `FlightBooking` and other relevant `Aggregates`
- `Flight`, `Booking`, `Leg`, and other `Entities`
- `Origin`, `Destination`, `DepartureDateTime`, `ArrivalDateTime`, and other `Value Objects`
- `BookFlight`, `CancelBooking`, `UpdateBooking`, … `Commands`
- `FlightBooked`, `BookingCanceled`, `BookingUpdated`, and other `Events`

In a product rating system, the `Command Model` may consist of:

- `ProductRating` and other relevant `Aggregates`
- `Product`, `CustomerRating`, `Category`, and other `Entities`
- `RatingDateTime`, `IpAddress`, and other `Value Objects`
- `AddRating`, `UpdateRating`, `RemoveRating`, and other `Commands`
- `RatingAdded`, `RatingUpdated`, `RatingRemoved`, and other `Events`