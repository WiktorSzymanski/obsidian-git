---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
source: "~/Documents/RozproszoneSystemyOperacyjne/"
---
# RSO 00. Indeks i mapowanie na prezentacje
---
> Ten katalog to **druga wersja** notatek z Rozproszonych Systemów Operacyjnych. W odróżnieniu od wersji pierwszej (`0-przygotowanieDoObrony/01 Rozproszone systemy operacyjne/`), która powstała z **listy zagadnień egzaminacyjnych** i wiedzy ogólnej, tutaj **każda informacja pochodzi z oryginalnych prezentacji wykładowych** dr Anny Kobusińskiej (`~/Documents/RozproszoneSystemyOperacyjne/`, 7 plików PDF, **427 slajdów**). Wszystko, czego w slajdach nie ma, jest jawnie oznaczone blokiem `> [!todo]` albo `> [!warning]`.

---
## Mapowanie: zagadnienie egzaminacyjne → notatka

| #   | Zagadnienie                              | Notatka                                             | Źródło                    | Pokrycie                                                                                                                                                  |
| --- | ---------------------------------------- | --------------------------------------------------- | ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Komunikacja grupowa                      | [[RSO 01 Komunikacja grupowa]]                      | `rso_sum_05.pdf` s. 1–56  | 🟡 **częściowe** — pełne BEB/RB/URB i specyfikacje porządków; brak klasyfikacji grup, usługi członkostwa, synchronizacji widoków i algorytmów FIFO/CO/TO  |
| 2   | Danocentryczne modele spójności          | [[RSO 02 Danocentryczne modele spójności]]          | `rso_sum_06.pdf` s. 1–55  | 🟢 **pełne** — wszystkie 6 modeli z warunkami, przykładami i protokołami; brak def. porządku przyczynowego (slajd nieczytelny) i modeli synchronizowanych |
| 3   | Modele spójności zorientowane na klienta | [[RSO 03 Modele spójności zorientowane na klienta]] | —                         | 🔴 **brak** — jedno zdanie na slajdzie 26 wykł. 6                                                                                                         |
| 4   | Algorytmy wzajemnego wykluczania         | [[RSO 04 Algorytmy wzajemnego wykluczania]]         | `rso_sum_07.pdf` s. 1–13  | 🟡 **częściowe** — scentralizowany, Lamport, Suzuki-Kasami; brak Ricart-Agrawali, Maekawy, Raymonda, wymagań i złożoności                                 |
| 5   | Algorytmy elekcji                        | [[RSO 05 Algorytmy elekcji]]                        | —                         | 🔴 **brak całkowity** — 0 trafień w 427 slajdach                                                                                                          |
| 6   | Zakleszczenie w systemach rozproszonych  | [[RSO 06 Zakleszczenie w systemach rozproszonych]]  | `rso_sum_07.pdf` s. 14–53 | 🟢 **pełne** — modele AND/OR z formalnymi predykatami, klasyfikacja detekcji, CMH AND i OR; brak Brachy-Touega i złożoności                               |

## Notatki dodatkowe (materiał ze slajdów spoza listy zagadnień)

| Notatka | Źródło | Zawartość |
|---|---|---|
| [[RSO 07 Model środowiska przetwarzania]] | `rso_sum_01.pdf` s. 1–101 | cechy/cele SR, przezroczystość, otwartość, skalowalność, GRID, chmura; węzeł, łącze, kanał, stan kanału i predykaty, operacje komunikacyjne, stan procesu, zdarzenia, funkcja tranzycji, procesy aktywne/pasywne, warunek uaktywnienia, **modele żądań**, relacja poprzedzania, diagramy przestrzenno-czasowe, graf stanów osiągalnych |
| [[RSO 08 Czas wirtualny i złożoność algorytmów]] | `rso_sum_02.pdf` s. 1–54 | monitor, konwencja zapisu (FRAME/MESSAGE/CONTROL/PACKET), czas wirtualny, **zegar skalarny (Lamport)**, **zegar wektorowy (Mattern)**, kanały FIFO (Müllender) i FC, uporządkowanie przyczynowe środowiska, funkcje kosztu, rząd funkcji, złożoność czasowa i komunikacyjna, przykład bariery |
| [[RSO 09 Stan globalny i migawki]] | `rso_sum_03.pdf` s. 1–65 | konfiguracja spójna, linia odcięcia, odcięcie spójne, modele stanów globalnych, **Chandy-Lamport**, **Lai-Yang**, **algorytm kolorujący** |
| [[RSO 10 Detekcja zakończenia]] | `rso_sum_04.pdf` s. 1–43 | zakończenie dynamiczne/statyczne, **Dijkstra-Feijen-van Gasteren**, przetwarzanie dyfuzyjne, **Dijkstry-Scholtena**, **Misra '83** |

---
## Mapowanie odwrotne: prezentacja → notatka

| Plik | Slajdy | Tytuł wykładu | Notatka |
|---|---|---|---|
| `rso_sum_01.pdf` | 101 | Rozproszone systemy operacyjne — wprowadzenie i model | [[RSO 07 Model środowiska przetwarzania]] |
| `rso_sum_02.pdf` | 54 | Czas wirtualny, złożoność algorytmów | [[RSO 08 Czas wirtualny i złożoność algorytmów]] |
| `rso_sum_03.pdf` | 65 | Konstrukcja spójnego obrazu stanu globalnego | [[RSO 09 Stan globalny i migawki]] |
| `rso_sum_04.pdf` | 43 | Problem detekcji zakończenia | [[RSO 10 Detekcja zakończenia]] |
| `rso_sum_05.pdf` | 56 | Mechanizmy rozgłaszania niezawodnego | [[RSO 01 Komunikacja grupowa]] |
| `rso_sum_06.pdf` | 55 | Zwielokrotnianie i spójność | [[RSO 02 Danocentryczne modele spójności]] |
| `rso_sum_07.pdf` | 53 | Wzajemne wykluczanie i zakleszczenie | s. 1–13 → [[RSO 04 Algorytmy wzajemnego wykluczania]]<br>s. 14–53 → [[RSO 06 Zakleszczenie w systemach rozproszonych]] |
| **razem** | **427** | | **10 notatek** |

Każdy z 427 slajdów jest objęty którąś z notatek — żaden zakres nie został pominięty.

---
## Zbiorcza lista TODO

### 🔴 Zagadnienia bez materiału w prezentacjach
- **Zag. 3** — modele zorientowane na klienta (RYW, MR, MW, WFR) → [[RSO 03 Modele spójności zorientowane na klienta]]
- **Zag. 5** — algorytmy elekcji (Bully, pierścieniowe, echo, Raft) → [[RSO 05 Algorytmy elekcji]]

### 🟡 Algorytmy nieobecne w prezentacjach
| Algorytm | Dotyczy | Notatka |
|---|---|---|
| Ricart-Agrawala | wzajemne wykluczanie | [[RSO 04 Algorytmy wzajemnego wykluczania#Braki w prezentacjach]] |
| Maekawa (kworum $\sqrt{N}$) | wzajemne wykluczanie | [[RSO 04 Algorytmy wzajemnego wykluczania#Braki w prezentacjach]] |
| Raymond (żeton na drzewie) | wzajemne wykluczanie | [[RSO 04 Algorytmy wzajemnego wykluczania#Braki w prezentacjach]] |
| Bracha-Toueg | detekcja zakleszczenia | [[RSO 06 Zakleszczenie w systemach rozproszonych#Braki w prezentacjach]] |
| ISIS / sekwencer | rozgłaszanie totalne | [[RSO 01 Komunikacja grupowa#Braki w prezentacjach]] |
| algorytmy FIFO / CO (numery sekwencyjne, zegary wektorowe) | porządki dostarczania | [[RSO 01 Komunikacja grupowa#Braki w prezentacjach]] |

### 🟡 Brakujące analizy złożoności
- Chandy-Lamport i algorytm kolorujący → [[RSO 09 Stan globalny i migawki#Porównanie trzech algorytmów]]
- wszystkie trzy algorytmy detekcji zakończenia → [[RSO 10 Detekcja zakończenia#Porównanie algorytmów]]
- wszystkie trzy algorytmy wzajemnego wykluczania → [[RSO 04 Algorytmy wzajemnego wykluczania#Braki w prezentacjach]]
- Chandy-Misra-Haas AND i OR → [[RSO 06 Zakleszczenie w systemach rozproszonych#Braki w prezentacjach]]

### 🟡 Pojęcia wspomniane, ale niezdefiniowane
- **detektor awarii** (doskonały) — używany w RB pasywnym i URB, nigdzie nie zdefiniowany → [[RSO 01 Komunikacja grupowa#Braki w prezentacjach]]
- **usługa członkostwa**, **zmiana widoku**, **synchronizacja widoków** → [[RSO 01 Komunikacja grupowa#Braki w prezentacjach]]
- **modele spójności przy dostępie synchronizowanym** (słaba, zwalniania, wejścia, zakresu) → [[RSO 02 Danocentryczne modele spójności#Klasyfikacja modeli spójności replik]]
- **spójność ostateczna** (_eventual consistency_) — nie występuje w ogóle

### 🔴 Slajdy technicznie nieczytelne
| Slajd | Problem | Notatka |
|---|---|---|
| `rso_sum_06.pdf` s. 31 „Definicja porządku przyczynowego" | formuły osadzone jako **pusta grafika** (stencil 1024×257, 37 B); brak warstwy tekstowej — treści **nie da się odzyskać** | [[RSO 02 Danocentryczne modele spójności#Definicja uszeregowania legalnego]] |
| `rso_sum_06.pdf` s. 54 „Relacje pomiędzy modelami" | etykiety i wzory renderują się niemal niewidocznie — odtworzone z **warstwy tekstowej** PDF | [[RSO 02 Danocentryczne modele spójności#Hierarchia modeli spójności]] |

### 🟡 Zadania bez rozwiązań
- `rso_sum_06.pdf` s. 55 — „W jakich modelach spójności operacja $r_3(x)v$ zwróci 1 / 2 / 3?" → [[RSO 02 Danocentryczne modele spójności#Zadanie z prezentacji]]

---
## Konwencje przyjęte w tym katalogu

- **Prefiks `RSO`** w nazwach plików — żeby nie kolidować z wikilinkami do wersji pierwszej, która jest linkowana z [[Mapa zagadnień]] i notatek innych przedmiotów.
- **Odsyłacz do slajdu** pod każdym nagłówkiem: `<sub>rso_sum_03.pdf, slajdy 19–23</sub>` — numeracja **slajdów**, nie stron PDF (każda strona PDF zawiera 2 slajdy w układzie handout).
- **Rysunki** w podkatalogu `assets/`, nazwane `rso-w<nr wykładu>-s<nr slajdu>-<opis>.png`, osadzane przez `![[...]]` z podpisem `<sub>`.
- **Braki** oznaczone blokiem `> [!todo] Brak w prezentacjach` z opisem, co dokładnie sprawdzono.
- **Nic spoza slajdów** nie jest opisane jako treść wykładu — wiedza zewnętrzna pojawia się wyłącznie wewnątrz bloków TODO jako wskazówka, czego szukać.

## Wersja pierwsza

Katalog `0-przygotowanieDoObrony/01 Rozproszone systemy operacyjne/` — 6 notatek napisanych z listy zagadnień egzaminacyjnych. **Zawiera materiał, którego nie ma w prezentacjach** (Ricart-Agrawala, Maekawa, Raymond, Bracha-Toueg, Bully, algorytmy pierścieniowe, RYW/MR/MW/WFR, ISIS, synchronizacja widoków), ale nie odzwierciedla notacji i akcentów wykładu. **Obie wersje warto czytać razem**: tę dla zgodności z wykładem, tamtą dla uzupełnienia luk.
