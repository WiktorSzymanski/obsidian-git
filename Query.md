---
up: "[[DDD]]"
---
#TRANSLATE 
# Query
---
The term `Query` in CQRS refers to a request to retrieve information or state from the `Domain`.

A `Query` should be modeled from the point of view of the information it requests. It should contain all the necessary information, but nothing more, for the system to find or calculate it and provide a response. Depending on your needs, that may include things like:

- Which objects should be taken into account and which should be filtered out
- How the results must be ordered
- Which object details should be included in or excluded from individual results
- Whether it should contain all the results or just a portion (a “page”) of them
- And so on.

The essential characteristic of a `Query` is that it can only request data from the `Domain`. It can not make any changes in the `Domain`.

### Examples

In a flight booking system:

- You would likely need to show booking details, information about the given flight, a list of matching flights, etc.

- Therefore, you would likely need queries that request this information - for example, `BookingDetailsQuery`, `FlightInformationQuery`, `MatchingFlightsQuery`, …

In a product rating system

- You would likely need to show the average rating of a product, the number of ratings, all ratings from a given customer, etc.

- Therefore, you would likely need queries that request this information - for example, `AverageProductRatingQuery`, `RatingsCountQuery`, `CustomerRatingsQuery`, …

In each case, the exact queries and their data will depend on the information relevant to your users.