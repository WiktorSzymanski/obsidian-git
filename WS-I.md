#SRC #Sem1 #TIWPR

### Web Services Interoperability (Organization)

### Cele
---
#### Osiągnięcie interoperacyjności usług sieciowych
- Integracja specyfikacji
- Promowanie spójnych implementacji
- Zapewnienie widocznej reprezentacji zgodności
#### Przyspieszenie wdrażania usług sieci Web
- Oferowanie wskazówek dotyczących implementacji i najlepszych praktyk
- Dostarczanie narzędzi i przykładowych aplikacji
- Zapewnienie forum dla implementujących, na którym deweloperzy mogą współpracować
#### Zachęcanie do przyjmowania usług sieci Web
- Budowanie konsensusu branżowego w celu zmniejszenia ryzyka _Early adopters_
- Zapewnienie forum dla użytkowników końcowych w celu komunikowania wymagań
- Podnoszenie świadomości wymagań biznesowych klientów

### Rezultaty
---
#### Profile
- Zdefiniowany zestaw specyfikacji lub standardów dla konkretnych wersji
- Wytyczne i konwencje dotyczące wspólnego korzystania z tych specyfikacji w sposób zapewniający interoperacyjność
### Przykładowe aplikacje
- Przypadki użycia i scenariusze użycia oparte na wymaganiach klienta
- Przykładowy kod i aplikacje zbudowane w wielu środowiskach
- Demonstracja interoperacyjności opartej na profilach
#### Narzędzia do testowania i materiały pomocnicze
- Narzędzia testujące implementacje profili pod kątem zgodności z profilami
- Dokumentacja pomocnicza i białe księgi

### Obecne grupy robocze WS-I
---
- **Basic Profile Working Group**
	Podstawowy zestaw specyfikacji, które stanowią podstawę dla usług sieci Web.
- **Basic Security Profile Working Group**
	Bezpieczeństwo komunikatów [[SOAP]], transport i inne kwestie bezpieczeństwa
- **XML Schema Work Plan Working Group**
	Planowanie odpowiednich rozwiązań dla kwestii interoperacyjności [[XML]] Schema

### Basic Profile - 1.0 i 1.1
---
- SOAP 1.1, WSDL 1.1, UDDI 2.0, XML 1.0, XML Schema, HTTP 1.1, TLS 1.0, SSL 3.0, X.509, HTTP over TLS
- Rozwiązano ponad 200 problemów związanych z interoperacyjnością
- Konwencje dotyczące przesyłania wiadomości, opisu, wykrywania, takie jak:
	- Deprecjacja RPC-encoded (użycie schematu jako interoperacyjnego systemu typów)
	- Wsparcie i wytyczne dla RPC/literal
	- Unikalne podpisy dla komunikatów wejściowych
	- Wyjaśnienia dotyczące obsługi awarii i błędów
	
### Basic Security Profile
---
- Scenariusze bezpieczeństwa (projekt grupy roboczej)
	- Dokumentacja zagrożeń bezpieczeństwa w interoperacyjnych usługach sieciowych wraz z potencjalnymi środkami zaradczymi.
- Podstawowy profil bezpieczeństwa 1.0 (projekt grupy roboczej)
	- Zajmuje się bezpieczeństwem transportu, bezpieczeństwem komunikatów SOAP i innymi względami bezpieczeństwa dla profili WS-I.
	- Profile specyfikacji bezpieczeństwa usług sieci Web OASIS
	- zatwierdzony projekt