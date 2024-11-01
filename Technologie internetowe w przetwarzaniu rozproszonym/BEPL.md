---
up: 
tags:
  - TechnologieInternetoweWPrzetwarzaniuRozproszonym
---
### Business Process Execution Language
---
- Na tyle wysoko poziomowe, że mogą się tym zajmować osoby "nie techniczne"
- Definicja procesu biznesowego abstrahująca od podstawowej aplikacji/usług
- Silnik BEPL wykonuje kod
- Procesy są opisane przy użyciu [[WSDL]]
- Wiadomości wysyłane przy użyciu [[SOAP]]
### Struktury
---
- **partners** - aktorzy w transakcji biznesowej
- **containers** - definicje wiadomości
- **operations** - wymagane typy usług sieciowych ([[Web Services]])
- **port types** - wymagane połączenia usług sieciowych
### Orkiestracja vs Choreografia
---
- Choreografia jest skupiona na widoku jednego uczestnika (np. peer to peer)
- Model orkiestracji obejmuje wszystkie strony i powiązane z nimi interakcje, dając globalny obraz systemu
### BEPL Programming Language
---
- Mechanizm korelacji komunikatów oparty na właściwościach
- Zmienne typowane przez [[XML]] i [[WSDL]]
- Rozszerzalny model wtyczek językowych umożliwiający pisanie wyrażeń i zapytań w wielu językach.
	- BPEL domyślnie obsługuje XPath 1.0
- Konstrukcje programowania strukturalnego, w tym if-then-elseif-else, while, sequence (uporządkowane) i flow (wykonanie równoległe)
- _scope_ umożliwiający hermetyzację logiki za pomocą zmiennych lokalnych, obsługi błędów, obsługi kompensacji i obsługi zdarzeń.
- Serializowane _scope_-y kontrolujące współbieżny dostęp do zmiennych