---
tags:
  - Magisterka
---
Aplikacja biura podróży wymaga synchronizacji z usługami:
- transportu:
	- lotniska
	- pociągi
	- autokary
- ubezpieczenia
- noclegi
- wycieczki


1. Można wydzielić mikro-serwisy (które będą odpowiednikami pojedynczej lambdy przy podejściu FaaS) - w ES mikro-serwis mógłby zapisywać eventy związane ze swoją działalnością
2. Eventy kompensujące rzucane w przypadku gdy jakiś np. środek transportu będzie opóźniony


## Use case-y użytkownika
1. Rezerwacja usługi:
	- z możliwością wyboru wariantu
2. Anulowanie usługi:
	- lub jakiegoś jej wariantu
3. Zmiana usługi:
	- lub jakiegoś jej wariantu
	  
## Akcie sytemu
1. Tworzenie usług
	- w zależności od dostępnych terminów noclegowych, wycieczkowych i transportu (raczej zawsze można wykupić ubezpieczenie)
2. Kompensacja w przypadku opóźnień transportów
3. Kompensacje spowodowane synchronizacją systemu


## Mock-owane microserwisy
1. Ich dane na podstawie pliku konfiguracyjnego zawierającego np. informacje o dostępnych lotach.
2. Możliwość losowego opóźniania transportów lub "zawsze takiego samego" (przy testach różnych rozwiązań warto aby takie zdarzenia występowały, ale powinny jednolicie dla każdego z przypadków) (na dobrą sprawę dwa tryby działania: normal i tests)



## Przy pisaniu pracy:
- napomnieć możliwe podejścia do non blocking API



## NIEPEWNE
- W "klasycznym rozwiązaniu" czy może wystąpić komunikacja wewnątrz aplikacji przez Event-Driven Architecture? Wprowadza to eventual consistency co zdaje się nie być takie klasycznie? Jeśli z kolei w klasycznym rozwiązaniu dokonamy transakcyjne operacji bookowania oferty, to czy w innych też nie powinno być jako że chcemy aby aplikacje poza architekturą były takie same. Czy uznajemy, że ponieważ inne architektury wprowadzają już eventual consistency to możemy sobie pozwolić na EDA ponieważ ma to sens aby to wykorzystać kiedy już generujemy eventy? A dziwnym z kolei by było transakcyjne zapisywanie 4 eventów, jeśli i tak jest to eventual consistent



## Spis treści
0. Wstęp
	- motywacja (dlaczego takie porównanie)
	- cel
	- struktura pracy
1. Podstawy teoretyczne
	- czym co jest
	- jak to jest implementowane na ogół 
2. Istniejące rozwiązania
	- można sprawdzić w literaturze czy są prace które zajmowały się takim porównaniem, lub analizą jednego z tych podejść
3. Zaproponowana architektura testowanej aplikacji
	- co w jakiej implementacji jest wyróżnione co jak ze sobą działa w wysokiego poziomu abstrakcji
4. Opis implementacji
	- ogólnie co mają wszystkie tak samo
	- tutaj konkretnie o implementacji jak została napisana
5. Testy
	- jak to było testowane, jakie środowisko, podejście metodologiczne, jak wykonywane ile razy powtarzane
	- otrzymane wyniki i ich analiza, tabelki wykresy, mini dyskusja na temat wyników
6. Podsumowanie


	(najtrudniejsze Wstęp i Podsumowanie)