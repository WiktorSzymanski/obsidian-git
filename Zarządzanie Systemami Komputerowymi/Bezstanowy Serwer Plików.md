---
up: 
tags:
  - ZarządzanieSystemamiKomputerowymi
---
# Bezstanowy Serwer Plików
---

>to rodzaj serwera plików, który nie przechowuje informacji o stanie klientów ani operacji pomiędzy kolejnymi żądaniami. Oznacza to ze każde żądanie klienta jest traktowane niezależnie od wcześniejszych.

Ze względu na bezstanowy charakter serwera, brak stanu ułatwia to skalowanie, ponieważ serwery nie muszą się synchronizować z jego powodu. Dodatkowo brak sesji zmniejsza obciążenie i ułatwia architekturę. Ze względu na swoją elastyczność i skalowalność znajdują swoje zastosowanie w systemach rozproszonych i chmurowych.

Przykładem bezstanowego serwera plików jest [[NFS]].