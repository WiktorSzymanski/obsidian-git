---
up: "[[Domain Model]]"
---
#TODO
# Event
---
> An `Event` is a notification that something relevant has happened inside the domain.

You use `Events` for explicitly implementing the effects and side-effects of the changes in your domain. This is how you communicate with other components in the same domain to make sure they are aware of and react somehow to important changes.

Note that `Events` are expressed in past tense and are immutable. What has happened has happened, and it cannot un-happen or happen differently. It may be possible to perform an action _(often called a compensating action)_ that returns the state of the domain to the one before the `Event` occurred and then publish a new `Event` _(often called a compensating event)_ to notify other components. You cannot, however, retroactively change an event.

`Events` may have different impact ranges. Some are only relevant in a given `Aggregate`. Others may have important information for all components in a given `Bounded Context`. Yet another can exist for the purpose of notifying different `Bounded Contexts`.

### Examples

Item added to the shopping cart:

This event is likely to be only relevant in the `Aggregate` responsible for the given shopping cart. Note the past tense. You could, in response, execute a “remove an item from shopping cart” operation, which will result in reverting the state and emitting an “Item removed from the shopping cart” compensating event. However, you cannot change the fact that an item was added.

An order has been received:

This event is likely to be only relevant in the `Bounded Context` responsible for managing orders. There may be multiple components that need to react somehow to that fact _(check product availability, calculate discount prices, etc.)_. You cannot change the fact the order was received; however, you can cancel it and issue “An order has been canceled” event.

The payment deadline has expired:

This event is likely to be only relevant in multiple `Bounded Contexts` that need to react somehow to that fact _(issue customer notification, calculate contractual fines, restrict access to services, etc.)_. You cannot change, nor can you compensate an expiration notification. You can, however, choose to ignore it and optionally update the expiration date.