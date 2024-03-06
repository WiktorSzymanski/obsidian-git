#SRC #Sem1 #TIWPR #wykład #w2 

to architektura tworzenia aplikacji biznesowych przez dynamiczne luźne powiązania interfejsów usług, implementacja ich funkcjonalności oraz realizajca wywołań ich operacji. Ma to na celu pokonanie barier języków programowania i tworzenie dynamicznej sieci usług.

- Podejście i metodologia biznesowa i technologiczna
- Decyzje biznesowe wspierane przez technologię

Zajmuje się długofalową wizją budową/rozwojem systemu informatyczngo. Ze wzglą stare aplikacje napisane w przestarzałych jezykach mogą być używane przy nowych rozwiązaniach w nowych technologiach. Zastępuje (kosztowny) rozwój **integracji** procesem **kompozycji** (_zero-integration enterprise_).  

SOA to kolejny etap rozwoju metodologii tworzenia systemów informatycznych
- Programowanie strukturalne (funkcyjne)
- Programowanie obiektowe
- Programowanie komponentowe (również rozproszone)
- Service Oriented Architecture

#### Założenia:
- Ustandaryzowany interfejs - możliwość podmiany implementacji usług
- Luźne powiązania - niezależność usługi od siebie, możliwość ewolucji implementacji
- Abstrakcja usług - ograniczenie interfejsu (kontraktu)
- Wielokrotne wykorzystanie - w różnych kontekstach i współbieżnie (wydajność, wersjonowanie interfejsów)
- Bezstanowość usług - skalowalność, uniwersalność, pozwala odpalić wiele tych samych usług i nie ma znaczenia do której zapytanie pójdzie bo wysztskie działają tak samo
- Katalog usług - formalny opis usługi, odkrywanie i dynamiczne wywoływanie usług, pozwala wybrać odpowiednią usługę do danego problemu
- Kompozycja usług - budowanie złożonych usług z podstawowych usług

#### Zalety:
- porządkuje relacje pomiędzy oferentami usułg a ich konsumentami
- pozwala na ponowne użycie elementów oprogramowania/usług
- zapewnia enkapsulację funkcjonalności
- precyzyjnie definiuje interfejsy
- zapewnia elastyczność aplikacji tworzonych na drodze kompozycji

#### Czym nie jest:
- nie eliminuje instniejacych architektur budowy oprogramowania
- nie jest technologią, lecz paradygmatem
- nie należy utożsamiać SOA z technologią Web Services
### Usługa
to moduł oprogramowania odpowienio duży, aby realizować pewną kompletną funkcję biznesową

"Service is a unit of solution logic to which service orientation has been applied to a meaningful extent." ~ Thomas Erl, SOA Design Patterns 

### Service-Oriented Solition Stack

![[Pasted image 20240305085949.png]]

#### Istniejące rozwiązania:
- Oracle SOA Suite
- IMB SOA Foundation
- Microsoft .Net
- Sun Java Composite ...
#TODO 

### Implementacja SOA
#### Usługi sieciowe
- Modele usług sieciowych (Web services):
	- Web Services (WS-\*)
	- Representional State Transfer (REST)
	- Modele Hybrydowe
	- Ewentualnie rozproszone obiekty (np. CORBA)