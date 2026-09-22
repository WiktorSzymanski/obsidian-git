---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 38
---
# 38. Algorytm Eraser do wykrywania sytuacji wyścigu
---
> **Eraser** (Savage, Burrows, Nelson, Sobalvarro, Anderson – SOSP 1997, „Eraser: A Dynamic Data Race Detector for Multithreaded Programs”) to **dynamiczny** detektor wyścigów danych w programach wielowątkowych opartych na blokadach. Wykorzystuje **algorytm zbioru blokad** (_lockset_). Sprawdza, czy **każda zmienna współdzielona jest konsekwentnie chroniona przez co najmniej jedną tę samą blokadę** (_locking discipline_).

## Wyścig danych (_data race_)
**Wyścig danych** występuje, gdy:
1. **dwa wątki** jednocześnie (bez synchronizacji) odwołują się do **tej samej komórki pamięci**,
2. **co najmniej jedno** odwołanie jest **zapisem**,
3. odwołania nie są chronione wspólną blokadą (nie ma między nimi uporządkowania).

Skutki: niedeterministyczne, trudne do odtworzenia błędy (utracone aktualizacje, niespójne struktury), zależne od przeplotu i sprzętu. Przykład `licznik++` = odczyt, dodanie, zapis – dwa wątki mogą oba odczytać 5 i zapisać 6.

**Race condition** (sytuacja wyścigu) to szersze pojęcie: poprawność zależy od względnej kolejności zdarzeń. Wyścig danych jest jej szczególnym, dobrze zdefiniowanym przypadkiem.

## Podejścia do wykrywania
| Podejście | Idea | Wady |
|---|---|---|
| **statyczne** (analiza kodu, systemy typów) | analiza wszystkich ścieżek bez uruchomienia | fałszywe alarmy, adnotacje, trudne dla aliasów i wskaźników |
| **dynamiczne happened-before** (zegary Lamporta/wektorowe, np. RecPlay, FastTrack) | wyścig = dwa współbieżne (nieuporządkowane przyczynowo) dostępy | wykrywa tylko wyścigi **ujawnione w danym przeplocie**; wymaga śledzenia wszystkich synchronizacji; kosztowne |
| **dynamiczne lockset (Eraser)** | sprawdzenie dyscypliny blokowania | fałszywe alarmy dla innych mechanizmów synchronizacji; tylko wykonane ścieżki kodu |

**Zaleta Erasera** względem happened-before: **nie zależy od przeplotu**. Wykrywa naruszenie dyscypliny blokowania, nawet jeśli w danym wykonaniu wątki akurat nie przeplotły się niebezpiecznie.

## Podstawowy algorytm lockset
**Założenie (dyscyplina blokowania)**: każda zmienna współdzielona $v$ jest chroniona pewną blokadą, która jest zajęta przy **każdym** dostępie do $v$.

**Struktury**:
- $locks\_held(t)$ – zbiór blokad trzymanych obecnie przez wątek $t$ (aktualizowany przy `lock`/`unlock`),
- $C(v)$ – **zbiór kandydujących blokad** dla zmiennej $v$ (blokady, które chroniły **wszystkie** dotychczasowe dostępy).

**Algorytm**:
```
dla każdej zmiennej v:  C(v) := zbiór wszystkich możliwych blokad

przy każdym dostępie (odczycie lub zapisie) wątku t do v:
    C(v) := C(v) ∩ locks_held(t)          // udoskonalenie zbioru (lockset refinement)
    jeśli C(v) = ∅ to
        ZGŁOŚ OSTRZEŻENIE: v nie jest chroniona żadną spójną blokadą
```

### Przykład
```c
lock(mu1);  v = v + 1;  unlock(mu1);      // locks_held = {mu1}      C(v) = {mu1, mu2}∩{mu1} = {mu1}
lock(mu2);  v = v + 1;  unlock(mu2);      // locks_held = {mu2}      C(v) = {mu1}∩{mu2} = ∅  → OSTRZEŻENIE
```
Każdy dostęp był chroniony, ale **różnymi** blokadami – brak wspólnej ochrony = potencjalny wyścig.

Twierdzenie: jeśli $C(v)$ nie jest pusty po wszystkich dostępach, to istnieje blokada chroniąca wszystkie dostępy w tym wykonaniu, a dostępy przez tę blokadę nie mogą wystąpić jednocześnie.

## Udoskonalenia – maszyna stanów zmiennej
Podstawowy algorytm zgłasza zbyt wiele fałszywych alarmów w typowych, poprawnych wzorcach:
1. **inicjalizacja** – zmienna tworzona i wypełniana przez jeden wątek **bez blokad**, zanim zostanie udostępniona innym,
2. **dane współdzielone tylko do odczytu** – po inicjalizacji wiele wątków tylko czyta (bez blokad – poprawne),
3. **blokady czytelników-pisarzy**.

Eraser dla każdej komórki pamięci utrzymuje **stan**:
```
                 zapis/odczyt pierwszego wątku
   ┌─────────┐ ─────────────────────────────▶ ┌───────────┐
   │ Virgin  │                                │ Exclusive │ ◀─┐ zapis/odczyt tego samego
   └─────────┘                                └───────────┘ ──┘ (pierwszego) wątku
                                                 │       │
                          odczyt przez nowy wątek│       │zapis przez nowy wątek
                                                 ▼       ▼
                                          ┌────────┐  zapis  ┌──────────────────┐
                                          │ Shared │ ──────▶ │ Shared-Modified  │
                                          └────────┘         └──────────────────┘
                                          odczyt: C(v) ∩=     odczyt/zapis: C(v) ∩=
                                          BEZ ostrzeżeń       OSTRZEŻENIE gdy C(v)=∅
```
| Stan | Znaczenie | Aktualizacja $C(v)$ | Ostrzeżenia |
|---|---|---|---|
| **Virgin** | nowa, nieużywana pamięć (świeżo zaalokowana) | nie | nie |
| **Exclusive** | dostęp tylko jednego wątku (inicjalizacja) | **nie** | nie |
| **Shared** | wiele wątków, ale po inicjalizacji tylko **odczyty** | **tak** | **nie** (dane tylko do odczytu są bezpieczne) |
| **Shared-Modified** | wiele wątków i co najmniej jeden zapis po udostępnieniu | tak | **tak**, gdy $C(v) = \emptyset$ |

- Przejście z Exclusive następuje dopiero, gdy **inny wątek** odwoła się do zmiennej – zakłada się, że inicjalizacja się zakończyła.
- **Ryzyko**: wyścig w chwili „przekazania” zmiennej innemu wątkowi nie jest wykrywany (kompromis na rzecz mniejszej liczby fałszywych alarmów).

## Blokady czytelników-pisarzy
Zmienna może być chroniona blokadą **rw**: odczyt pod blokadą w trybie odczytu lub zapisu, zapis tylko w trybie **zapisu**.
```
przy odczycie v przez t:   C(v) := C(v) ∩ locks_held(t)          // blokady w dowolnym trybie
przy zapisie v przez t:    C(v) := C(v) ∩ write_locks_held(t)    // tylko blokady w trybie zapisu
                            jeśli C(v) = ∅ → ostrzeżenie
```
Dzięki temu zapis pod blokadą **tylko do odczytu** jest zgłaszany jako błąd.

## Implementacja
- Pierwotnie dla **Digital Unix / Alpha**, przez **binarną instrumentację** kodu narzędziem **ATOM** – bez rekompilacji programu:
  - każde **ładowanie i zapis** do pamięci (sterta, zmienne globalne) przechwytywane,
  - wywołania **blokowania i odblokowania** (`pthread_mutex_lock` itp.) aktualizują $locks\_held(t)$,
  - alokacje (`malloc`) inicjalizują stan pamięci jako Virgin.
- **Słowo cienia** (_shadow word_) dla każdego 32-bitowego słowa pamięci: 2 bity stanu + 30 bitów – identyfikator wątku (w stanie Exclusive) lub **indeks zbioru blokad** (_lockset index_).
- **Tablica zbiorów blokad** – każdy unikalny zbiór blokad przechowywany raz (posortowany wektor), wyniki przecięć zapamiętywane w pamięci podręcznej → niski koszt aktualizacji.
- Zmienne na **stosie** nie są sprawdzane (zakłada się, że są prywatne dla wątku).
- **Narzut**: spowolnienie **10–30×**, podwojenie zużycia pamięci.
- Raport zawiera nazwę zmiennej/adres, wątek, **stos wywołań** przy dostępie naruszającym dyscyplinę (oraz ostatni dostęp zmieniający $C(v)$).

## Fałszywe alarmy i adnotacje
Źródła fałszywych alarmów:
- **ponowne użycie pamięci** – prywatne alokatory (pule) zwalniają i ponownie przydzielają pamięć bez `free/malloc`, więc stan nie wraca do Virgin,
- **prywatne blokady** – synchronizacja własnymi mechanizmami, których Eraser nie rozpoznaje (spinlocki, operacje atomowe CAS, semafory, zmienne warunkowe bez blokad),
- **łagodne wyścigi** (_benign races_) – celowe, niekrytyczne dostępy bez blokady (np. statystyki, liczniki przybliżone, double-checked flag).

Adnotacje w kodzie (makra):
- `EraserReuse(adres, rozmiar)` – przywrócenie pamięci do stanu Virgin,
- `EraserReadLock(lock)`, `EraserReadUnlock`, `EraserWriteLock(lock)`, `EraserWriteUnlock` – informowanie o prywatnych blokadach,
- `EraserIgnoreOn()` / `EraserIgnoreOff()` – wyłączenie sprawdzania fragmentu (łagodne wyścigi).

## Ograniczenia
- **Dynamiczność** – sprawdza tylko **wykonane** ścieżki kodu (potrzebne dobre testy pokrycia).
- Wymaga, by program stosował **dyscyplinę blokowania** – programy synchronizujące się inaczej (kanały, lock-free, `fork-join`) dają fałszywe alarmy.
- **Nie wykrywa**:
  - **wyścigów wysokiego poziomu** – każdy dostęp chroniony, ale grupa powiązanych zmiennych modyfikowana w osobnych sekcjach krytycznych ([[39 Wykrywanie wyścigów wysokiego poziomu]]),
  - **naruszeń atomowości** (sprawdź-potem-działaj pod dwiema oddzielnymi blokadami),
  - zakleszczeń,
  - wyścigów przy przekazywaniu własności pomiędzy wątkami (stan Exclusive).
- Wysoki narzut – nieużywany produkcyjnie.

## Następcy i narzędzia
- **Hybrydowe** (lockset + happened-before) – mniej fałszywych alarmów: MultiRace, RaceTrack (.NET), O'Callahan-Choi,
- **FastTrack** (Flanagan-Freund 2009) – precyzyjny happened-before z epokami zamiast pełnych zegarów wektorowych,
- **ThreadSanitizer (TSan)** – Google, w GCC/Clang (`-fsanitize=thread`) i Go (`-race`): początkowo hybryda lockset + happened-before, obecnie happened-before z pamięcią cienia,
- **Helgrind** (Valgrind) – początkowo algorytm Erasera, później hybrydowy; **DRD**,
- **Java**: PreciseRaceDetector w [[40 Sprawdzanie modelu - Java Pathfinder|Java PathFinder]], RoadRunner, IBM ConTest, Intel Inspector.
