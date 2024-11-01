---
tags:
  - SystemyWysokiejNiezawodności
---
# Odtwarzanie
---

## Rodzaje
**[[Odtwarzanie Postępowe|Postępowe]]** (*forward recovery*)
> Jeśli natura błędu powodującego awarię pozwala na usunięcie błędów ze stanu systemu i system (proces) jest wyposarzony w mechanizmy obsługujące dany błąd, to błąd ten można efektywnie wyeliminować (naprawić) i umożliwić postęp przetwarzania. (np. całkowita utrata precyzji w wyniku operacji zmiennoprzecinkowej może zostać usunięta przez podanie wyniku "zero" w obsłudze błędu (*arythmetic underflow*))

**[[Odtwarzanie Wsteczne|Wsteczne]]** (*backward recovery*)
> Jeśli błąd jest nieprzewidywalny lub awaria nieodwracalna (np. *arythmetic overflow*), można jedynie wymienić cały stan systemu na wcześniej zarejestrowany, wolny od błędów. 
