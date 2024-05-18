#SRC #Sem1 #TIWPR 
### Service Oriented Architecture
---
to architektura tworzenia aplikacji biznesowych przez dynamiczne luźne powiązania interfejsów usług, implementacją ich funkcjonalności oraz realizacją wywołań ich operacji. Ma to na celu pokonanie barier języków programowania i tworzenie dynamicznej sieci [[Usługa|usług]].

- Podejście i metodologia biznesowa i technologiczna
- Decyzje biznesowe wspierane przez technologię

Zajmuje się długofalową wizją budową/rozwojem systemu informatycznego. Stare aplikacje napisane w przestarzałych językach mogą być używane przy nowych rozwiązaniach w nowych technologiach. Zastępuje (kosztowny) rozwój **integracji** procesem **kompozycji** (_zero-integration enterprise_).  

SOA to kolejny etap rozwoju metodologii tworzenia systemów informatycznych
- Programowanie strukturalne (funkcyjne)
- Programowanie obiektowe
- Programowanie komponentowe (również rozproszone)
- Service Oriented Architecture
### Zalety
---
- porządkuje relacje pomiędzy oferentami usułg a ich konsumentami
- pozwala na ponowne użycie elementów oprogramowania/usług
- zapewnia enkapsulację funkcjonalności
- precyzyjnie definiuje interfejsy
- zapewnia elastyczność aplikacji tworzonych na drodze kompozycji

### Czym nie jest
---
- nie eliminuje instniejacych architektur budowy oprogramowania
- nie jest technologią, lecz paradygmatem
- nie należy utożsamiać SOA z technologią Web Services

### Założenia SOA
---
- Ustandaryzowany interfejs - możliwość podmiany implementacji usług
- Luźne powiązania - niezależność usługi od siebie, możliwość ewolucji implementacji
- Abstrakcja usług - ograniczenie interfejsu (kontraktu)
- Wielokrotne wykorzystanie - w różnych kontekstach i współbieżnie (wydajność, wersjonowanie interfejsów)
- Bezstanowość usług - skalowalność, uniwersalność, pozwala odpalić wiele tych samych usług i nie ma znaczenia do której zapytanie pójdzie bo wszystkie działają tak samo
- Katalog usług - formalny opis usługi, odkrywanie i dynamiczne wywoływanie usług, pozwala wybrać odpowiednią usługę do danego problemu
- Kompozycja usług - budowanie złożonych usług z podstawowych usług
### Service-Oriented Solution Stack
---
![[Pasted image 20240518112231.png]]

> Do komunikacji pomiędzy usługami wykorzystuje się [[ESB|Enterprise Service Bus]].

### Istniejące rozwiązania:
---
- Oracle SOA Suite
- IBM SOA Foundation
- Microsoft .Net
- Sun Java Composite Application Platform Suite (Java CAPS)
- JBoss Enterprise SOA Platform

### Implementacja SOA - usługi sieciowe
---
- Modele usług sieciowych (Web services):
	- [[Web Services]] (WS-\*)
	- Representational State Transfer ([[REST]])
	- Modele Hybrydowe
- Ewentualnie:
	- rozproszone obiekty (np. CORBA)