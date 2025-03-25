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
