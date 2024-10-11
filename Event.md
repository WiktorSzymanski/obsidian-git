---
up: "[[Domain Model]]"
---
# Event
---
>**Event** to informacja, że coś ważnego wydażyło się w domenie.

**Event**-ów można używać aby wyraźnie zaimplementować *efekty* i *skutki uboczne* zmian w danej [[Domain|domenie]]. W ten sposób można komunikować się z innymi komponentami w tej samej [[Domain|domenie]] mając pewność, że są świadome i zareagują odpowiednio na ważne zmiany.

**Event**-y są nazywane w czasie przeszłym. Nie da się odwrócić lub wykonać inaczej **eventu**, który miał już miejsce. Jest jednak możliwe aby wykonać operacje (często nazywaną *compensating action*), która przywraca stan [[Domain|domeny]] do tego przed wykonaniem się danego **eventu** i wtedy opublikowanie nowego **eventu** (często nazywanego *compensating event*) by poinformować pozostałe komponenty. 

**Event**-y mogą mieć różny zasięg. Niektóre mogą mieć tylko znaczenie w danym [[Aggregate|agregacie]], a inne mogą mieć ważne informacje dla wszystkich komponentów w danym [[Bounded Context]]. Jeszce inny może istnieć, którego celem będzie poinformowanie innego [[Bounded Context]].

#### Przykłady
---
- Przedmiot został dodany do kosza z zakupami
- Otrzymano zamówienie
- Upłynął ostateczny termin uniszczenia wpłaty