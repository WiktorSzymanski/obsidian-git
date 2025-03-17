---
tags:
  - SystemyWysokiejNiezawodności
---
# Replikacja Procesu
---
## Replikacja aktywna
- każda replika odsyła odpowiedź do procesu
- klient może wybrać strategię odbierania odpowiedzi:
	- first
	- all existing (n)
	- R < n
- dynamicne grupy, bez znaczenia kto się w niej znajduje, stąd znacząca jest komunikacja grupowa
### Spójność
Wielu klientów może się komunikować w tym samym czasie z replikami. Zapytania muszą być **w pełi uporządkowane** (_totally ordered_). Dokładna kolejność nie ma znaczenia ważne aby wszystkie repliki miały zapytania w tej samej kolejności. 

### Zalety
- jeśli istnieje jakaś działająca replika, klient otrzyma odpowiedź
- błędy bizantyjskie mogą być dozwolone
### Wady
- wszystkie repliki cały czas działają
- źle skaluje się, dodawanie kolejnych replik nie poprawia wydajności, a może ją pogorszyć
## Replikacja pasywna
- _primary_ przesyła odpowiedź do klienta po tym jak wszystkie _backup_-y odpowiedzą _primary_
- dynamiczne grupy wymagają serwisu zarządzającego członkami
- niezawodność zależy od repliki _primary_, gdy ta upadnie:
	- operacja _update_ musi być atomowa ([[Reliable Broadcast]])
	- nowya replika _primary_ musi być wybrana
	- klient może zaobserwować opóźnienia
	- może być konieczność ponowienia zapytania
### Spójność
Replika _primary_ wymusza **pełne uporządkowanie** zapytań

Przetwarzanie niedeterministyczne jest dozwolone ponieważ tulko jedna replika obsługuje zapytania i wymusza globalną spójność pomiędzy członkami grupy. Problematyczne może się okazać _nested invocations_.