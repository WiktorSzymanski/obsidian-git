# `Entity`

An `Entity` is one of two ways of including `Domain` objects in a `Domain Model`. It is used to represent an object that has an **identity** and a **thread of continuity**.

You will model a `Domain` object as an `Entity` when:

- The values of the attributes of that object do not identify it. They may change at any time, but you still consider that to be the same object.
- Multiple objects have the same values and attributes, but you still consider them to be different objects.

The object’s identity may already exist in the `Domain` in a form that naturally becomes part of the `Domain Model`. But it may also be an abstract concept. In this case, you need to find a non-invasive way to represent it in the `Domain Model`. Either way, it is crucial that each `Entity` you model has a **unique identifier** that the system can use to find, update, and store it.

### Examples

A person _(when modeling HR operations)_

You can change your name, title, and uniform, which will give you different attributes. Yet, you still must be recognized as you by the system.

Other people may have the same name, title, uniform, and other attributes as you, but those individuals must not be recognized as you by the system.

Therefore, you should probably model that person as an `Entity`. Since a person does not have a natural and explicit identifier _(fingerprints, DNA samples, and so on are not relevant in this `Domain`)_, you should use one that is already established in the `Domain` _(employee ID?)_. If there is none, you should introduce one that fits seamlessly in your `Domain Model`.

A bank account _(when modeling bank transfers)_

The balance, limits, and overdraft allowance might change, but it is still the same account.

Other accounts might have the same balance, limits, and overdraft allowance, but they are different accounts.

Therefore, you should probably model a bank account as an `Entity`.