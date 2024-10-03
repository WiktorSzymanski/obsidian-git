---
up:
---
#TRANSLATE 
# CQRS
---
CQRS is an abbreviation of **Command Query Responsibility Separation**, an architectural pattern that, as its name implies, separates a system into two different areas:

- One that deals with the operations that, when executed, produce changes in the system. These operations are called the `Commands`.

- One that takes care of the operations that are only requesting information. These operations are called the `Queries`. They are not making any changes in the system.


In short and a little more simply, CQRS implies considering the writes and the reads from our system independently.

### Examples
---
Product ratings in an online store:
- On your command side, you may have operations that add a new rating from a customer, modify an existing rating, remove a rating, etc.

- On your query side, you may have operations that return the average rating of a given product, the top-rated products in a given time interval, all the ratings a given customer has provided, etc.

Flight booking system:
- On your command side, you may have operations that enable the user to book a flight, cancel a booking, update the booking’s data, etc.

- On your query side, you may have operations that return the airline’s terms, baggage allowance, seat availability, etc.