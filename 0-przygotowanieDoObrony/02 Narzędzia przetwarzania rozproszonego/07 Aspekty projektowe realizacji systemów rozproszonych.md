---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 7
---
# 7. Aspekty projektowe realizacji systemów rozproszonych
---
> **System rozproszony** to zbiór niezależnych komputerów, które użytkownikom wydają się jednym spójnym systemem (Tanenbaum). Projektując go, trzeba rozwiązać problemy nieobecne w systemie scentralizowanym: brak wspólnej pamięci i zegara, częściowe awarie, opóźnienia i heterogeniczność.

Definicja i podstawowe własności: [[Narzędzia Przetwarzania Rozproszonego/Wykład 1 - Sylabus 1#System rozproszony]], [[Narzędzia Przetwarzania Rozproszonego/Podstawowe Własności Systemu Rozproszonego]].

## Główne aspekty (wyzwania) projektowe
### 1. Współdzielenie zasobów
Dostęp do zasobów przez **zarządcę zasobu** z dobrze zdefiniowanym interfejsem ([[Zarządzanie Systemami Rozproszonymi/System Rozproszony#Koncepcja dostępu do współdzielonych zasobów]]). Wymaga nazewnictwa, lokalizowania zasobów i kontroli dostępu.

### 2. Heterogeniczność
Różne sieci, sprzęt (kolejność bajtów, rozmiary typów), systemy operacyjne, języki programowania i implementacje.
- **Rozwiązania**: warstwa pośrednia (_middleware_: RPC, CORBA, gRPC, MOM), wspólna reprezentacja danych (XDR, CDR, Protocol Buffers, JSON/XML), maszyny wirtualne (JVM).

### 3. Otwartość
Możliwość rozszerzania i reimplementacji systemu dzięki **publicznym, ustandaryzowanym interfejsom**. Pojęcia: **interoperacyjność** (współpraca implementacji różnych producentów), **przenośność** (uruchomienie na innej platformie), **rozszerzalność**. Zasada **oddzielenia polityki od mechanizmu**.

### 4. Przezroczystość (ISO RM-ODP)
| Rodzaj | Ukrywa przed użytkownikiem |
|---|---|
| dostępu | różnice w reprezentacji danych i sposobie dostępu (lokalny/zdalny identyczny) |
| położenia | fizyczną lokalizację zasobu (nazwa nie zawiera adresu) |
| migracji | przeniesienie zasobu w inne miejsce (bez wpływu na dostęp) |
| relokacji | przeniesienie zasobu **w trakcie używania** |
| replikacji | istnienie wielu kopii zasobu |
| współbieżności | współdzielenie zasobu przez wielu użytkowników |
| awarii | awarie i odtwarzanie komponentów |
| trwałości | to, czy zasób jest w pamięci ulotnej, czy na dysku |

Pełna przezroczystość jest nieosiągalna i nie zawsze pożądana: opóźnień sieci nie da się ukryć, a ukrywanie awarii kosztuje wydajność. Zob. [[Narzędzia Przetwarzania Rozproszonego/Przezroczystość]].

### 5. Skalowalność
System działa efektywnie przy wzroście:
- **rozmiaru** (liczby użytkowników i zasobów),
- **geograficznym** (duże opóźnienia, zawodne łącza WAN),
- **administracyjnym** (wiele niezależnych domen administracyjnych).

**Techniki** (zob. [[Algorytmy Rozproszone/Wykład 2]]):
- **komunikacja asynchroniczna** – nie blokować klienta w oczekiwaniu na odpowiedź,
- **podział i rozproszenie** (_partitioning_) – np. DNS, sharding,
- **replikacja i buforowanie** (_caching_) – kosztem spójności,
- unikanie algorytmów scentralizowanych: brak globalnego stanu, decyzje na podstawie lokalnej informacji, odporność na awarię pojedynczego węzła, brak założenia o globalnym zegarze.

### 6. Obsługa awarii (niezawodność)
**Awarie częściowe** – część komponentów działa, część nie; trudno odróżnić awarię od opóźnienia.
- **Wykrywanie**: sumy kontrolne, timeouty, heartbeaty, detektory awarii.
- **Maskowanie**: retransmisje, redundancja (replikacja).
- **Tolerowanie**: degradacja jakości usługi.
- **Odtwarzanie**: punkty kontrolne, logi – [[22 Wsteczne odtwarzanie stanu - uzupełnienie]].
- **Modele awarii**: crash, omission, timing, bizantyjskie – [[23 Rozproszone uzgadnianie w środowisku zawodnym]].
- Miary: dostępność, niezawodność, MTTF, MTTR.

### 7. Współbieżność
Wiele procesów jednocześnie korzysta ze współdzielonych zasobów. Wymaga synchronizacji (wzajemne wykluczanie, transakcje, kontrola współbieżności) i spójnego uporządkowania zdarzeń (zegary logiczne).

### 8. Komunikacja i model interakcji
Wybór paradygmatu (komunikaty, RPC, obiekty zdalne, pamięć wspólna, publish/subscribe) oraz semantyki komunikacji: synchroniczna/asynchroniczna, przejściowa/trwała, gwarancje dostarczenia. Zob. [[08 Podejścia do budowy systemów rozproszonych]].

### 9. Synchronizacja i czas
Brak globalnego zegara → synchronizacja zegarów fizycznych (Cristian, Berkeley, NTP) albo zegary logiczne (Lamport, wektorowe). Koordynacja: wzajemne wykluczanie, elekcja, konsensus.

### 10. Spójność i replikacja
Kompromis spójność – dostępność – wydajność (CAP/PACELC). Wybór modelu spójności: [[02 Danocentryczne modele spójności]], [[03 Modele spójności zorientowane na klienta]].

### 11. Nazewnictwo
Nazwy (czytelne dla ludzi), identyfikatory (unikalne) i adresy (lokalizacja). Usługi nazewnicze (DNS, LDAP), rozwiązywanie nazw, nazwy płaskie vs hierarchiczne.

### 12. Bezpieczeństwo
Poufność, integralność, dostępność. Uwierzytelnianie w otwartej sieci, autoryzacja, bezpieczne kanały (TLS), ochrona przed DoS, bezpieczeństwo kodu mobilnego.

### 13. Jakość usług (QoS)
Wydajność (czas odpowiedzi, przepustowość), gwarancje dla multimediów (opóźnienie, jitter), adaptacyjność.

## Organizacja oprogramowania
| Typ systemu | Opis | Cel |
|---|---|---|
| **DOS** – rozproszony SO | ścisłe powiązanie, jeden obraz systemu na multikomputerze | ukrycie i zarządzanie zasobami sprzętowymi |
| **NOS** – sieciowy SO | niezależne SO z usługami sieciowymi (np. NFS, rlogin) | udostępnianie zasobów lokalnych klientom zdalnym |
| **Middleware** | warstwa nad NOS dająca wspólny model programistyczny | przezroczystość rozproszenia i heterogeniczności |

## Architektury
- **klient-serwer** (w tym wielowarstwowe: prezentacja – logika – dane),
- **peer-to-peer** – [[48 Nieustrukturyzowane systemy P2P]], [[49 Ustrukturyzowane systemy P2P i DHT]],
- **oparte na zdarzeniach / publish-subscribe** (luźne powiązanie w czasie i przestrzeni),
- **współdzielona przestrzeń danych** (Linda, tuple spaces),
- **SOA / mikroserwisy** – [[Technologie internetowe w przetwarzaniu rozproszonym/SOA]].

## Błędne założenia (8 _fallacies_, Deutsch)
Projektanci często niesłusznie zakładają, że: 1) sieć jest niezawodna, 2) opóźnienie jest zerowe, 3) przepustowość jest nieskończona, 4) sieć jest bezpieczna, 5) topologia się nie zmienia, 6) jest jeden administrator, 7) koszt transportu jest zerowy, 8) sieć jest jednorodna.
