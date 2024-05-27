---
up: 
class: ZSK
---
#REFACTOR
# Lightweight Directory Access Protocol
---

>to protokół sieciowy wykorzystywany do zarządzania i dostępem do usług katalogowych. Oparty na [[X.500]], realizuje model klient-server i jest niezależny od architektury i środowiska. LDAP stosuje **hierarchiczną strukturę danych** ([[DIT]]). Jest zaprojektowany do obsługi dużych ilości danych i zoptymalizowany do większej ilości odczytów jak zapisów. Dzięki temu, że jest otwartym standardem różne systemy i aplikacje mogą ze sobą współgrać przy użyciu tego protokołu. Dane przechowywane w usłudze katalogowej są uporządkowane, zgodne z pewnym [[Schemat Danych|schematem danych]].

#### Dziedziny zastosowań LDAP
- scentralizowane zarządzanie informacją - np. użytkownikami, grupami, itp.
- ujednolicenie zarządzania danymi - np. integracja z wieloma aplikacjami
- możliwość [[Odwołania do innych serwerów|dystrybucji danych]] - rozłożenia ich na różne fizyczne serwery
- możliwość dystrybucji/delegacji zarządzania
- wyszukiwanie informacji przez zwykłych użytkowników

![[Pasted image 20240524163212.png]]

#### LDAP vs DB
- rozszerzalność [[Schemat Danych|schematów danych]] - w dowolnym momencie możemy wprowadzić dowolny atrybut
- dystrybucja danych dzięki strukturze drzewiastej pozwala dzielić informacje na logicznie powiązane segmenty
- replikacja poprzez jeden serwer _master_ i jeden lub wiele serwerów _slave_
- przetwarzanie transakcyjne co najwyżej na poziomie pojedynczego obiektu, w bazie danych jest to bardziej skomplikowane
- rozmiar danych - zwykle operuje na mniejszych porcjach danych.
- standaryzacja protokołu - dowolny klient LDAP powinien się porozumieć z innym, różne systemy zarządzania bazą danych nie gwarantują tej własności.

# Zalety stosowania usługi katalogowej
---
- Bardziej precyzyjna dystrybucja administracji
- Scentralizowana konfiguracja
- Duża efektywność dostępu do danych
	- replikacja
	- dużo danych ⇒ katalog szybszy niż pliki płaskie
- Kontrola składni wprowadzanych danych
- Możliwość szyfrowania komunikacji ([[SSL & TLS|SSL/TLS]])

## Nazwy zastępcze (aliasy)
---

są pseudo-węzłami wskazującymi na inne węzły. Koncepcyjnie działają podobnie do dowiązań symbolicznych w systemie Unix. Nie są obsługiwane przez wszystkie serwery LDAP, ze względu na degradacje efektywności. Aby węzeł stał się aliasem musi być obiektem klasy `alias` z atrybutem `aliasedObjectName` zawierającym [[DN]] właściwego obiektu.

## [[LDIF]]

## Przeszukiwanie bazy
---
### Kryteria przeszukiwania
- bazowa nazwa wyróżniająca (base)
- głębokość/zakres przeszukiwania (scope)
- warunki dla atrybutów (filtr)
### Zapis filtru
- notacja prefiksowa
- operatory: `&`, `|`, `=`, `~=`, `<`, `>`, `>=`, `<=`
- znaki specjalne: `*`, `?`

#### Przykłady
``` LDAP
(objectClass=posixAccount)
```
>obiekty klasy `posixAccount`

``` LDAP
(cn=Voytek*)
```
>obiekty (osoby) o nazwie (imieniu) zaczynającej się na `Voytek`
	
``` LDAP
(mail=*)
```
>obiekty posiadające ustawiony atrybut `mail`

``` LDAP
(|(uid=fred)(uid=bill))
```
>obiekty o identyfikatorach `fred` lub `bill`

``` LDAP
(&(|(uid=fred)(uid=bill))(objectClass=posixAccount))
```
>obiekty klasy `posixAccount` o identyfikatorach `fred` lub `bill`

### Zakres przeszukiwania
#### base
> Przeszukuje tylko dany węzeł. Przydatne gdy dany węzeł ma wiele wartości tego samego atrybutu i chcemy sprawdzić czy posiada pewną konkretną wartość.

#### one
> Przeszukuje bezpośrednie dzieci danego węzła, jeden poziom niżej. Zwykle tym węzłem jest jakaś kolekcja, a szukamy w niej pewnego wystąpienia.

#### sub
> Przeszukuje całe drzewo od danego węzła, mało efektywne.

#### children
> Przeszukuje wszystkie swoje dzieci i ich następców.

## LDAP URL
---
``` URL
ldap://serwer/DN?atrybuty?zakres?filtr
```

#### Przykłady
``` URL
ldap://ldap.put.poznan.pl/dc=put,dc=pl
ldap://srv/ou=Osoby,dc=put,dc=pl??sub?uid=voytek
ldap://srv/dc=put,dc=pl?cn?sub?uid=voytek
ldap://srv/ou=Osoby,dc=put,dc=pl?cn?one?(|(uid=a*)(uid=b*))
```

## LDAP v3
---

- Wsparcie dla języków narodowych (UTF‐8)
- Odwołania do serwerów zewnętrznych (referrals)
- Bezpieczeństwo
	- uwierzytelnianie v2: anonymous, simple, Kerberos v4
	- v3: SASL (Simple Authentication and Security Layer)
- Rozszerzalność (extended operation), np. StartTLS
- Obsługa schematów danych (kontrola poprawności danych + podstawowe definicje)
- Introspekcja funkcjonalności serwera
- [[Atrybuty Operacyjne]]

## Operacje protokołu LDAP
---

- **bind** - Uwierzytelnienie klienta. v3: Domyślnie połączenie jest anonimowe. Możliwe jest przełączanie się.
- **unbind** - Zamknięcie połączenia.
- **search**- Przeszukiwanie drzewa DIT. Argumenty: baza, zakres, filtr. Dodatkowo: lista atrybutów do zwrócenia, ograniczenie czasu wykonania zapytania i liczby obiektów zwracanych.
- **modify** - Modyfikacja istniejącego rekordu (nazwa atrybutu + operacja: dodanie, usunięcie, zamiana).
- **add** - Dodanie nowego rekordu (nazwa i zbiór atrybutów).
- **delete** - Usunięcie istniejącego rekordu.
- **modify RDN** - zmiana RDN. v3: Ogólna operacja modify DN umożliwiająca również przenoszenie obiektu w  inne miejsce drzewa DIT. 
- **compare** - Sprawdzenie obecności atrybutu o określonej wartości.
- **abandon** - Przerwanie wykonywanej operacji.

## Directory Specific Entry
---

### Directory Service Agent
Oprogramowanie serwera LDAP.
### DSA Specific Entry
Wirtualny obiekt z atrybutami opisującymi funkcjonalność
serwera i opisujący przechowywane drzewa.

#### Przykład
``` LDAP
dn:
namingContexts: dc=put,dc=pl
namingContexts: o=FirmaX,c=Poland
namingContexts: o=localfiles
supportedExtension: 1.3.6.1.4.1.4203.1.11.1
supportedLDAPVersion: 2
supportedLDAPVersion: 3
supportedSASLMechanisms: GSSAPI
subschemaSubentry: cn=Subschema
```

## Implementacje serwerów LDAP 
---
- IBM Tivoli Directory Server
- Sun Java System Directory Server
- Novell eDirectory
- Microsoft Active Directory (AD)
- Apple Open Directory
- [[OpenLDAP]]
- Fedora/Red Hat Directory Server (wcześniej Netscape Directory Server)
- Apache Directory Server