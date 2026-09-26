# `Domain`

The term `Domain` refers to a specific sphere of knowledge influence or activity. It is the **subject area** or the **environment** (or both) to which a program is applied.

A `Domain` is part of the natural world and typically consists of the following elements:

- Terms that have a specific definition in the sphere of knowledge influence or activity
- Objects that have specific states at specific points in time
- Operations that result in object creation, change of state, or termination
- Invariants that must be enforced across state changes

For example, in the “banking” `Domain` there are

- Terms like `balance`, `transaction`, `asset`, which have precise definitions
- Objects like `customer`, `account`, `expenses`, which can be in different states at different times
- Operations like `account creation`, `money transfer`, `money withdrawal`, which result in state changes
- Invariants like `users can't go over their credit limits`, which must be enforced no matter what the operation is

When building software, you should avoid re-inventing the `Domain`. If you feel like your software needs new or different terms, objects, or operations, you should consult with domain experts first. The chances are that what you want to use already exists in the `Domain`, but you haven’t discovered it yet.

When a `Domain` is too broad, abstract, or both, it is helpful to identify any subdomains and respective objects, operations, and invariant. that may exist. Start by asking the domain experts. Many domains have known natural or well-defined subdomains that have developed over time.