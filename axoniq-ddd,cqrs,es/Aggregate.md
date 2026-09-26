# `Aggregate`

The term `Aggregate` is used to describe modeling the boundaries and rules of direct object relationships. In simpler terms, it represents a group of associated objects that are considered as one unit with regard to data changes.

An `Aggregate` is an abstract term that often does not exist in the actual `Domain` but is useful in the `Domain Model`. It has:

- A group of `Entities` and/or `Value Objects`
- The relations they form (ownership, dependency, etc.)
- The invariants that must be enforced all across those objects

An `Aggregate` essentially defines a boundary that separates the objects inside it from those outside. There must be a single object inside the `Aggregate` that can interact with the objects outside the `Aggregate`. Such an object is called `Aggregate Root`.

When you are modeling `Aggregates`, following these rules is crucial:

- Objects outside the `Aggregate` **cannot** have references to objects inside the `Aggregate`.
- Objects outside the `Aggregate` **can** have references to the `Aggregate Root`
- Objects inside the `Aggregate` **can** have references to each other.
- Objects inside the `Aggregate` **can** have references to the `Aggregate Roots` of other `Aggregates`.
- The `Aggregate Root` is responsible for maintaining the invariants.
- If the `Aggregate Root` is deleted, all objects inside the `Aggregate` are also deleted because no other object can have references to them.

### Example

Let’s assume you are managing a passenger itinerary that consists of several legs. A leg is a single flight from an origin to a destination, and every leg is operated by a certain flight. The combination of flights gets the passenger to its origin and destination. There are several possible ways to establish boundaries:

- One `Aggregate` represents the passenger. Another `Aggregate` represents the itinerary with all legs and flights.
- One `Aggregate` represents the passenger and itinerary. Another `Aggregate` represents each leg and respective flight.
- One `Aggregate` represents the passenger, itinerary, and legs. Another `Aggregate` represents each flight.

For the problem you must solve, _managing an itinerary for a customer_, the best answer would be the last option listed above for these reasons:

- The passenger, itinerary, and legs are one unit with regard to data changes.
- There are well-defined relationships: The itinerary belongs to the passenger; the itinerary consists of legs.
- There are invariants: the origin of the first leg and the destination of the last one never changes.
- The itinerary is what is important to the outside object, so it is the `Aggregate Root`.
- References to the passenger and individual legs from outside objects do not make sense and are not allowed.
- Legs have references to flights _(different `Aggregate`)_ that may change without affecting the itinerary.