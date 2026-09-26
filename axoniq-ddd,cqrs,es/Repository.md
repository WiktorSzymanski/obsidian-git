# `Repository`

The term `Repository` describes an infrastructure component that can store, find, and provide instances of `Aggregates`, `Entities` and `Value Objects`

A `Repository` is conceptual. It makes no claims about how exactly the data should be stored, searched, or loaded. It merely defines the options a given `Domain Model` provides for the system to be able to store and find the correct objects later.

The requirements and rules in the `Domain` you are modeling dictate which `Repositories` you add to your `Domain Model` and which operations and capabilities they should have. As with everything else you have seen so far, you should refrain from modeling `Repositories`, operations, capabilities, etc., that are not relevant to the problem space.

### Example

If your system requires you to be able to maintain people’s addresses, you may need to model a `Repository` that has the ability to:

- Store addresses in some storage _(database, LDAP, file, remote service, etc.)_.
- Find an address by a person’s identifier.
- Find all addresses matching person’s last name and a city
- And more.