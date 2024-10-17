---
tags:
  - SystemyRozproszoneDużejSkali
---

# Cassandra
---
>Apache **Cassandra** to *open-source*-owa, rozproszona baza danych NoSQL. Implementuje model przechowywania danych wide-column (aka. column-oriented #questionMark), które spełnia semantyke finalnej spójności (*ang. eventually consistent*).

## Właściwości
- Dane są automatycznie rozpraszane, z (pozytywnym) wpływem na wydajność. Osiąga to za pomocą partycji. Każdy węzeł posiada określony zestaw tokenów, a system zarządzania bazą danych dystrybuuje dane w oparciu o zakresy tych tokenów w całym klastrze. Klucz partycji jest odpowiedzialny za położenie danych w odpowiednich węzłach i określenia położenia danych. Po wprowadzeniu danych do klastra pierwszym krokiem jest zastosowanie funkcji skrótu do klucza partycji. Wynik funkcji decyduje do jakiego węzła trafią dane, zgodnie z przedziałami węzłów.
- #TODO
- Wspiera kopie migawkowe (*ang. snapshot*) przedstawiające dane w czasie (PIT) co pozwala na łatwą integracje z narzędziami do towrzenia kopi zapasowej. Obsługuje również przyrostowe kopie zapasowe, w których dane mogą być archiwizowane w miarę pisania.

## Użytkowanie
Ustawienia konfiguracyjne można skonfigurować w pliku `cassandra.yaml`. Niektórymi ustawieniami można manipulować podczas działania za pomocą komandy `nodetool` lub interfejsu online, są jednak zmiany które wymagają ponownego odpalenia usługi. Do wyświetlania logów aplikacji wykorzystuje się komendę `auditlogviewer`, a do wyświetlania, odtwarzania i porównywania pełnych zapytań można wykorzystać `fqltool`.