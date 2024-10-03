---
class: SRDS
---
# Architektura Lambda
---
> Ma na celu rozwiązać problem opóźnień i starych wyników poprzez utworzenie **dwóch ścieżek dla danych**

#TODO obrazek z prez

- *Hot path (speed layer)*  - podejmowanie decyzji z niepełnym zbiorem danych ale w czasie rzeczywistym
- *Cold path (batch layer)* - przechowuje wszystkie przychodzące dane w ich **surowej formie** .... . Pozwala na **dużą dokładność** 

#TODO hot i cold path zrobić
#TODO 

## Lambda Tech
#TODO


## Zalety
- #TODO
## Wady
- duża **złożoność**
- utrzymywanie dwóch rodzajów systemu
- zależność od *framework*-u
- **przetwarzanie** tych samych danych **dwa razy**
- kosztowność *batch layer*
- #TODO