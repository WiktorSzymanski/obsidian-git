---
up: "[[Domain Model]]"
---
#TRANSLATE 
# Bounded Context
---
The term `Bounded Context` explicitly defines a specific part of the `Domain Model` where specific objects have consistent meaning and relevant characteristics.

In English, _(like almost any other language)_, the same word can mean different things. A “bank” is a financial institution, a set or series of similar things, or the land alongside a river or lake. In this case, the context defines the meaning. It’s the same with objects in the `Domain Model`. They can have different meanings and relevance for different actors, in different use cases, and more. They are defined by their context.

`Bounded Contexts` draw the boundaries of the contexts that exist in the `Domain Model`. Within those boundaries, you describe the characteristics of the objects that are important in the context. While defining a context, you must not only consider the problems you need to solve but also those you do not want to solve. You must ensure the model’s conceptual consistency by using clear and concise meanings.

### Examples
---
Consider the “flight” object:

The passenger view:
	A passenger most likely cares about the flight number, the check-in time, the gate, and how much luggage is allowed. The passenger context of your model should take those specific concerns into account and ignore all the other information that is irrelevant from the passenger’s perspective.

The airport’s ground crew view:
	The ground crew most likely cares about the flight number, the gate, the aircraft specs, the time of arrival, and the turn-around time. The ground crew context of your model should take those specific concerns into account and ignore all the other information that is irrelevant from the ground crew’s perspective.

The cabin crew view:
	The cabin crew most likely cares about the flight number, the crew members, the service schedule, and the number of passengers. The cabin crew context of your model should take those specific concerns into account and ignore all the other information that is irrelevant from the cabin crew’s perspective.
>>>>>>> Stashed changes
