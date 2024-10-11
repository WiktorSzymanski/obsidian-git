---
up:
---
#TRANSLATE 
# Query Model

The term `Query Model` refers to a `Domain Model` that is optimized for and contains everything needed for processing `Queries`.

The `Query Model` typically consists of two sets of components that are directly related to each other:

- The information storage and retrieval components known as `Projections`, which are responsible for providing the means for data to be extracted or generated from relevant `Events` and stored according to how it will be queried and used.
- The `Query` interpretation and processing components, which are responsible for retrieving the respective data and preparing a response in the form requested/

The entire `Query Model` is essentially read-only. You have the flexibility to adapt the design of your data storage to scale and optimize it as needed. You are free to de-normalize data, apply the table-per-view concept, have multiple storages for the same data, or use any other optimization technique you see fit.

### Examples

In a flight booking system, the `Query Model` will likely consist of:

- `FlightProjection` and other relevant `Projections`
- Infrastructure components that interact with appropriate storage _(RDBS, noSQL DB, file system, etc.)_
- Domain components with `handleBookingDetailsQuery`, `handleFlightInformationQuery`, `handleMatchingFlightsQuery`, and other functions
- `BookingDetailsQuery`, `FlightInformationQuery`, `MatchingFlightsQuery`, and other `Queries`

In a product rating system, the `Query Model` may consist of:

- `ProductRatingProjection` and other relevant `Projections`
- Infrastructure components that interact with appropriate storage _(RDBS, noSQL DB, file system, etc.)_
- Domain components with `handleAverageProductRatingQuery`, `handleRatingsCountQuery`, `handleCustomerRatingsQuery`, and other functions
- `AverageProductRatingQuery`, `RatingsCountQuery`, `CustomerRatingsQuery`, and other `Queries`