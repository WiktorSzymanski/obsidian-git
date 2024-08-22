---
up: "[[Domain Model]]"
---
# Repository
---
> Mechanizm do **enkapsulacji przechowywania**, wyodrębniania i przeszukiwania emulujący **kolekcję** obiektów. Definiuje w jaki sposób w [[Domain|domenie]] chcemy odnaleźć właściwy obiekt ([[Aggregate]], [[Entity]], [[Value Object]]).

**Repozytorium** ma charakter koncepcyjny. Nie zawiera żadnych twierdzeń na temat tego, jak dokładnie dane powinny być przechowywane, wyszukiwane czy załadowywane. Definiuje jedynie opcje, które dany [[Domain Model|model domeny]] zapewnia systemowi, aby mógł przechowywać i znajdować odpowiednie obiekty w późniejszym czasie.

Wymagania i zasady w modelowanej [[Domain|domenie]] dyktują, które **repozytoria** należy dodać do [[Domain Model|modelu domeny]] oraz jakie operacje i możliwości powinny one posiadać. Powinno się powstrzymać od modelowania repozytoriów, operacji, możliwości itp. nie mających znaczenia w problemie który chcemy rozwiązać.

#### Przykład
---
Jeśli system wymaga możliwości przechowywania adresów osób, może być konieczne zaprojektowanie **repozytorium**, które ma możliwość między innymi:
- Przechowywania adresów w pamięci masowej (baza danych, [[LDAP]], plik itp.)
- Znajdowania adresu na podstawie identyfikatora osoby
- Znajdowania wszystkich adresów pasujących do nazwiska osoby i miasta