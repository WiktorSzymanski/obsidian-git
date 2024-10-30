---
tags:
  - SystemyWysokiejNiezawodności
---
# Odtwarzanie
---

## Rodzaje
**Postępowe** (*forward recovery*)
> Minusem tego działania jest konieczność przewidzenia danego błędu i napisania dla niego procedury

**[[Odtwarzanie Wsteczne|Wsteczne]]** (*backward recovery*)
> Polega na przywróceniu stanu sprzed awarii. Jest to bardzo skomplikowane dla systemu rozproszonego w implementacji. Konieczne jest znalezienie takiego staniu który nie posiada już błędu. W przeciwnym razie **odtwarzanie wsteczne** jest nieefektywne ponieważ wystąpi ten sam błąd przy operacji, która doprowadziła do [[Awarie|awarii]].

