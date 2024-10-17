---
class: RBD
---
# Rozproszona baza danych
---
#TODO 

#### DDBS nie jest:
- systemem komputerowym z podziałem czasu
- Luźno lub ściśle powiązany system procesorowy
- #TODO

#TODO obraz z prez

## Założenia
- Dane przechowywane w wielu lokalizacjach -> każda lokalizacja składa się z jednego logicznego procesora
- Logiczne procesory w różnych lokalizacjach są połączone siecią komputerową -> nie system wieloprocesorowy
	- Równoległe systemy baz danych
- Rozproszona baza danych jest bazą danych, a nie zbiorem plików -> dane logicznie powiązane, jak pokazano we wzorcach dostępu użytkowników
	- Relacyjny model danych
- Rozproszony system DBMS to pełnoprawny system DBMS
	- Nie zdalny system plików, nie monitor transakcyjny