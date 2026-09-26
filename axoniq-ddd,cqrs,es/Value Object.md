# `Value Object`

An `Value Object` is one of two ways of including `Domain` objects in the `Domain Model`. It is used to represent an object that has no conceptual identity and is fundamentally **defined by its attributes**.

You will model a `Domain` object as `Value Object` when:

- The values of the attributes of that object uniquely identify it. If any of them changes, you no longer consider that to be the same object.
- All objects that have the attribute values are essentially the same object.

As `Value Objects` are identified by their attributes, they must be **immutable**. Instead of changing a value of an attribute, you must create a different `Value Object`.

An object may have a natural identity that is not relevant to the `Domain` you operate in, and thus it should not be in the `Domain Model` you are building. In this case, you should ignore the identity and represent it as a `Value Object` in the `Domain Model`. The system should always use the complete set of attribute values to create, find and store such objects.

### Examples

A $10 bill _(when modeling a cash register)_:

A $10 bill has a nominal value of $10. If you change the nominal value, it is no longer a $10 bill; it’s a different bill.

All other $10 bills have the same nominal value. For any cash register operation, they are indistinguishable from one another and thus essentially are the same object.

Therefore, you should probably model a $10 bill as a `Value Object`. Each bill has an explicit identifier _(Serial Number)_ that is not relevant in the cash register `Domain`. You should ignore it and not include it in your `Domain Model`

An address _(when modeling an address book)_:

An address has a city, street, and number. If you change any of the attribute values, it is no longer the same address.

Any number of addresses with the same values of their fields are indistinguishable from one another and thus essentially are the same object.

Therefore, you should probably model an address as a `Value Object`. An address does not have an explicit identifier. You don’t need one, and you should not try to invent one.