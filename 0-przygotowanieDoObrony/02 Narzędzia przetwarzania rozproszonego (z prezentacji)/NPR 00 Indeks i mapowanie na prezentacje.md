---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
source: "0-przygotowanieDoObrony/NarzędziaPrzetwarzaniaRozproszonego/"
---
# NPR 00. Indeks i mapowanie na prezentacje
---
> Ten katalog to **druga wersja** notatek z Narzędzi Przetwarzania Rozproszonego. W odróżnieniu od wersji pierwszej (`0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego/`), która powstała z **listy zagadnień egzaminacyjnych** i wiedzy ogólnej, tutaj **każda informacja pochodzi z oryginalnych prezentacji wykładowych** (`0-przygotowanieDoObrony/NarzędziaPrzetwarzaniaRozproszonego/slajdy–merged.pdf`, **301 slajdów** sklejonych z ośmiu wykładów, pokrywających zagadnienia **7, 8 i 9**). Materiał z `Opracowanie.pdf` (81 stron) i z notatek w vaultcie jest wpleciony **wyłącznie tam, gdzie slajd czegoś nie mówi**, i **zawsze oznaczony** blokiem `> [!note] Uzupełnienie z Opracowania` albo `> [!note] Uzupełnienie spoza slajdów`. Wnioski, których na slajdach nie ma, oznaczone są blokiem `> [!warning]`.

---
## Mapowanie: zagadnienie egzaminacyjne → notatka

### Zagadnienie 7 — Aspekty projektowe realizacji systemów rozproszonych

| # | Notatka | Slajdy | Pokrycie |
|---|---|---|---|
| 1 | [[NPR 01 Zdalne wywoływanie procedur (RPC)]] | 16–46 | 🟡 **częściowe** — pełne zagadnienia projektowe i realizacyjne, cztery semantyki błędu, konwersja danych, wiązanie; **brak przezroczystości innej niż dostępu** |
| 2 | [[NPR 03 NFS, idempotentność i bezstanowość]] | 107–114 | 🟢 **pełne** — stan postrzegany i integralny, idempotentność i bezstanowość w obu sensach, NFS jako studium przypadku |

Streszczenie do powtórki: [[NPR 07 Aspekty projektowe realizacji systemów rozproszonych|skompresowane/NPR 07]].

### Zagadnienie 8 — Podejścia do budowy systemów rozproszonych (charakterystyka porównawcza)

| # | Notatka | Slajdy | Pokrycie |
|---|---|---|---|
| 3 | [[NPR 01 Zdalne wywoływanie procedur (RPC)]] | 16–46 | 🟢 **pełne** — podejście proceduralne |
| 4 | [[NPR 02 Sun RPC i standard XDR]] | 47–76 | 🟢 **pełne** — `rpcgen`, plik `.x`, potoki i filtry XDR, odwzorowanie typów |
| 5 | [[NPR 04 Podejście obiektowe - Java RMI]] | 184–214 | 🟡 **częściowe** — pełne RMI z obiektami aktywowalnymi; **brak CORBA** i semantyki błędu RMI |
| 6 | [[NPR 08 MOM i systemy kolejkowania komunikatów]] | 1–9 | 🟢 **pełne** — cechy MOM, oba paradygmaty, model pojęciowy, lista realizacji |
| 7 | [[NPR 09 ZeroMQ]] | 10–15 | 🟡 **częściowe** — kontekst, gniazda, wzorce; **brak opcji gniazd**, w tym filtra subskrypcji |
| 8 | [[NPR 10 JMS]] | 224–254 | 🟢 **pełne** — komponenty, oba modele, model programistyczny, pięć mechanizmów niezawodności, transakcje |
| 9 | [[NPR 11 Przestrzeń krotek - Linda i JavaSpaces]] | 255–264 | 🟡 **częściowe** — model Lindy i JavaSpaces; **brak operacji `eval`** i wzorców zastosowań |
| 10 | [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione]] | 265–300 | 🔴 **brak części rozproszonej** — wykład omawia wyłącznie współbieżność **lokalną**; Annex E, partycje i PolyORB nie występują |

Streszczenie do powtórki wraz z **tabelą porównawczą sześciu podejść**: [[NPR 08 Podejścia do budowy systemów rozproszonych|skompresowane/NPR 08]].

### Zagadnienie 9 — Wielozadaniowość i synchronizacja zadań/wątków

| # | Notatka | Slajdy | Pokrycie |
|---|---|---|---|
| 11 | [[NPR 06 Wątki w Javie i elementarna synchronizacja]] | 115–152 | 🟢 **pełne** — tworzenie wątków, stany, demony, grupy, `synchronized`, `wait`/`notify`, zgubiony sygnał |
| 12 | [[NPR 07 Pakiet java.util.concurrent]] | 153–183 | 🟢 **pełne** — atomowość, `volatile`, `atomic`, kolekcje, semafor, bariery, `Lock`/`Condition`, pule wątków, obiekty niezmienne |
| 13 | [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione]] | 265–300 | 🟢 **pełne** — zadania, spotkania, `select` z dozorami, obiekty chronione, bariery, `requeue` |

Streszczenie do powtórki wraz z **zestawieniem Java ↔ Ada**: [[NPR 09 Wielozadaniowość i synchronizacja zadań i wątków|skompresowane/NPR 09]].

---
## Mapowanie odwrotne: slajdy → notatka

| Slajdy | Liczba | Temat wykładu | Notatka |
|---|---|---|---|
| 1–9 | 9 | Systemy kolejkowania komunikatów | [[NPR 08 MOM i systemy kolejkowania komunikatów]] |
| 10–15 | 6 | ØMQ (ZeroMQ) | [[NPR 09 ZeroMQ]] |
| 16–46 | 31 | RPC — zagadnienia projektowe i realizacyjne | [[NPR 01 Zdalne wywoływanie procedur (RPC)]] |
| 47–76 | 30 | Sun RPC / XDR | [[NPR 02 Sun RPC i standard XDR]] |
| **77–106** | **30** | **duplikat slajdów 47–76** | — |
| 107–114 | 8 | Przykład — Network File System | [[NPR 03 NFS, idempotentność i bezstanowość]] |
| 115–152 | 38 | Obsługa i elementarna synchronizacja wątków · Koordynacja wątków, spójność danych | [[NPR 06 Wątki w Javie i elementarna synchronizacja]] |
| 153–183 | 31 | Podejścia do synchronizacji · pakiet `java.util.concurrent` | [[NPR 07 Pakiet java.util.concurrent]] |
| 184–214 | 31 | Podejście obiektowe do budowy systemów rozproszonych | [[NPR 04 Podejście obiektowe - Java RMI]] |
| **215–223** | **9** | **duplikat slajdów 1–9** | — |
| 224–254 | 31 | JMS (wg slajdów Cezarego Sobańca) | [[NPR 10 JMS]] |
| 255–264 | 10 | Przestrzeń krotek | [[NPR 11 Przestrzeń krotek - Linda i JavaSpaces]] |
| 265–300 | 36 | Ada 95 — współbieżność | [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione]] |
| **301** | **1** | **lista zadań implementacyjnych** | — świadomie pominięty |
| **razem** | **301** | | **11 notatek + 3 streszczenia** |

**Objęte notatkami: 261 slajdów. Pominięte świadomie: 40.**

### Dlaczego pominięto

- **Slajdy 77–106** są **dokładnym duplikatem** slajdów 47–76 (Sun RPC / XDR) — sklejony PDF zawiera ten wykład dwukrotnie. Zgodność potwierdzona porównaniem warstwy tekstowej slajd po slajdzie.
- **Slajdy 215–223** są **dokładnym duplikatem** slajdów 1–9 (Systemy kolejkowania komunikatów) — ten sam powód.
- **Slajd 301** zawiera **listę czterech zadań implementacyjnych** (semafor binarny, ograniczone buforowanie z buforem jedno- i wieloelementowym, semafor ogólny) bez treści merytorycznej. Rozwiązania tych i podobnych zadań w Javie i Adzie znajdują się w `Opracowanie.pdf` na stronach 65–81 i **zgodnie z ustaleniem nie zostały objęte tym katalogiem** — pozostają w PDF-ie.

---
## Struktura katalogu

```
02 Narzędzia przetwarzania rozproszonego (z prezentacji)/
├── NPR 00 Indeks i mapowanie na prezentacje.md   ← ta notatka
├── NPR 01 … NPR 11                               ← 11 notatek tematycznych
├── assets/                                       ← 45 diagramów wyciętych ze slajdów
└── skompresowane/                                ← 3 streszczenia pod zagadnienia 7, 8, 9
```

- **Notatki tematyczne `NPR 01`–`NPR 11`** odwzorowują strukturę prezentacji jeden do jednego. Każda ma we frontmatterze pole `slajdy:` z zakresem źródłowym, a każda sekcja jest opatrzona cytowaniem w postaci `<sub>slajdy–merged.pdf, slajd N</sub>`.
- **`skompresowane/`** — warstwa **do powtórki**: jeden plik na zagadnienie egzaminacyjne, ciągłą prozą, same pojęcia i ich znaczenie, bez kodu, bez API i bez cytowań slajdów. Numeracja plików w tym podkatalogu to **numer zagadnienia** (7, 8, 9), a nie numer notatki tematycznej.

> [!warning] Kolizja numeracji
> `NPR 07` w katalogu głównym to **`java.util.concurrent`**, a `NPR 07` w `skompresowane/` to **zagadnienie 7 (aspekty projektowe)**. Podobnie `NPR 08`: w katalogu głównym **MOM**, w `skompresowane/` **zagadnienie 8**. Rozróżnia je katalog oraz pole `zagadnienie:` we frontmatterze.

---
## Zbiorcza lista braków

### 🔴 Czego prezentacje nie omawiają wcale

| Temat | Dotyczy zagadnienia | Gdzie odnotowane |
|---|---|---|
| **Rozproszona Ada** — Annex E, partycje, `Remote_Call_Interface`, PolyORB | 8 | [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione#Braki w prezentacjach]] |
| **CORBA** — IDL, ORB, pośrednik obiektowy | 8 | [[NPR 04 Podejście obiektowe - Java RMI#Braki w prezentacjach]] |
| **Rozproszona pamięć współdzielona (DSM)** | 8 | [[NPR 08 Podejścia do budowy systemów rozproszonych]] |
| **pthreads** — w tym `pthread_cond_wait`/`signal` | 9 | [[NPR 06 Wątki w Javie i elementarna synchronizacja#Braki w prezentacjach]] |
| **MPI i OpenMP** | 9 | [[NPR 06 Wątki w Javie i elementarna synchronizacja#Braki w prezentacjach]] |
| **Przezroczystość inna niż dostępu** — położenia, migracji, relokacji, replikacji, współbieżności, awarii, trwałości | 7 | [[NPR 01 Zdalne wywoływanie procedur (RPC)#Braki w prezentacjach]] |
| **Heterogeniczność, bezpieczeństwo, skalowalność, otwartość, middleware** jako aspekty projektowe | 7 | [[NPR 07 Aspekty projektowe realizacji systemów rozproszonych]] |
| **Uwierzytelnianie w Sun RPC** (`AUTH_SYS`, `AUTH_DES`) | 8 | [[NPR 02 Sun RPC i standard XDR#Braki w prezentacjach]] |
| **Model pamięci Javy** (JMM) i relacja _happens-before_ | 9 | [[NPR 07 Pakiet java.util.concurrent#Braki w prezentacjach]] |

### 🟡 Tematy poruszone, ale niedokończone

| Temat | Notatka |
|---|---|
| **Semantyka błędu RMI** — nie nazwana, mimo że RMI realizuje *co najwyżej raz* | [[NPR 04 Podejście obiektowe - Java RMI]] |
| **Protokół SELECT** — opisany dwoma zdaniami, bez omówienia nadawania identyfikatorów | [[NPR 01 Zdalne wywoływanie procedur (RPC)]] |
| **Koszt wariantów CHAN** — diagramy bez zestawienia liczby komunikatów | [[NPR 01 Zdalne wywoływanie procedur (RPC)]] |
| **Wyrównanie do 4 bajtów w XDR** — podstawowa reguła kodowania z RFC 1014 nie pada | [[NPR 02 Sun RPC i standard XDR]] |
| **Budowa uchwytu pliku NFS i protokół `mount`** | [[NPR 03 NFS, idempotentność i bezstanowość]] |
| **Rozproszone zbieranie nieużytków w RMI** (`Unreferenced`, dzierżawy) | [[NPR 04 Podejście obiektowe - Java RMI]] |
| **Opcje gniazd ZeroMQ** — w tym `ZMQ_SUBSCRIBE`, bez którego gniazdo SUB nie odbiera nic | [[NPR 09 ZeroMQ]] |
| **Kolejki martwych listów i transakcje XA w JMS** | [[NPR 10 JMS]] |
| **Operacja `eval` w Lindzie** i wzorce zastosowań przestrzeni krotek | [[NPR 11 Przestrzeń krotek - Linda i JavaSpaces]] |
| **Problem ABA** przy `compareAndSet`, `ForkJoinPool`, `CompletableFuture` | [[NPR 07 Pakiet java.util.concurrent]] |
| **Pragmy kolejkowania i atrybut `'Count`** w Adzie | [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione]] |

### 🔴 Pytania postawione na slajdach bez odpowiedzi

| Slajd | Pytanie | Gdzie odpowiedź |
|---|---|---|
| 111 | Jak zrealizować uniksowe `open`, `creat`, `read`/`write`, `lseek` na interfejsie NFS? Gdzie przechowywane są dane identyfikujące otwarty plik? | [[NPR 03 NFS, idempotentność i bezstanowość#Odwzorowanie interfejsu uniksowego na zdalny]] — **zrekonstruowana**, nie cytowana |
| 146 | Co się stanie, gdy metoda `synchronized` zostanie wywołana z innej metody `synchronized`? | slajd 147 — **zamek jest wielowejściowy** |
| 152 | Czy przy pomocy mechanizmów synchronizacji w Javie da się zbudować monitor? | [[NPR 06 Wątki w Javie i elementarna synchronizacja#Czy da się zbudować monitor?]] — **zrekonstruowana**, nie cytowana |

---
## Relacja do notatek w vaultcie

Katalog `Narzędzia Przetwarzania Rozproszonego/` zawiera 17 krótkich notatek z zajęć. Ich status wobec tego katalogu:

| Notatka w vaultcie | Status |
|---|---|
| [[Narzędzia Przetwarzania Rozproszonego/ZMQ]] | **stub** — zastąpiona przez [[NPR 09 ZeroMQ]]; jedna uwaga praktyczna z niej wykorzystana |
| [[Narzędzia Przetwarzania Rozproszonego/Wywoływanie metod zdalnych]] | **pusta** — zastąpiona przez [[NPR 04 Podejście obiektowe - Java RMI]] |
| [[Narzędzia Przetwarzania Rozproszonego/Istota Podejścia Obiektowego]] | **pusta** — zastąpiona przez [[NPR 04 Podejście obiektowe - Java RMI]] |
| [[Narzędzia Przetwarzania Rozproszonego/Zmienne Warunkowe]] | **pusta** — prezentacje nie omawiają pthreads, więc **luka pozostaje** |
| [[Narzędzia Przetwarzania Rozproszonego/Przezroczystość Migracji]] | **pusta** — prezentacje nie omawiają przezroczystości migracji, **luka pozostaje** |
| [[Narzędzia Przetwarzania Rozproszonego/ADA-95]] | sekcja „Współbieżność" **pusta** — zastąpiona przez [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione]]; **pozostaje przydatna** jako opis składni podstawowej języka, której slajdy nie obejmują |
| [[Narzędzia Przetwarzania Rozproszonego/RPC]] | krótka, **praktyczna** — `rpcgen -a`, `libtirpc`; wykorzystana w [[NPR 02 Sun RPC i standard XDR]] |
| [[Narzędzia Przetwarzania Rozproszonego/Gwarancja wykonania (semantyka błędu)]] | zgodna ze slajdami, niepełna — pełna treść w [[NPR 01 Zdalne wywoływanie procedur (RPC)]] |
| [[Narzędzia Przetwarzania Rozproszonego/Trwałość Komunikacji]], [[Narzędzia Przetwarzania Rozproszonego/Synchroniczność komunikacji]], [[Narzędzia Przetwarzania Rozproszonego/Paradygmat Interakcji Pomiędzy Zdalnymi Jednostkami]] | **uzupełniają** slajdy o terminologię (komunikacja przejściowa/nieustanna, synchroniczna/asynchroniczna) — linkowane z [[NPR 08 MOM i systemy kolejkowania komunikatów]] |
| [[Narzędzia Przetwarzania Rozproszonego/Przezroczystość]], [[Narzędzia Przetwarzania Rozproszonego/Podstawowe Własności Systemu Rozproszonego]] | **jedyne** źródło dla części zagadnienia 7 spoza slajdów |

**Wersja pierwsza katalogu** (`02 Narzędzia przetwarzania rozproszonego/`, notatki 07–09) pozostaje **nietknięta** — tak samo, jak `01 Rozproszone systemy operacyjne` obok `01 … (z prezentacji)`.
