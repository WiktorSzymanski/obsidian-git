---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 41
---
# 41. Pamięć transakcyjna – implementacja, zastosowanie, wady i zalety
---
> **Pamięć transakcyjna** (TM – _Transactional Memory_) to mechanizm synchronizacji programów współbieżnych. Programista oznacza fragment kodu jako **transakcję** (blok atomowy), a system uruchomieniowy lub sprzęt gwarantuje, że wykona się on **atomowo i w izolacji** względem innych transakcji. Współbieżność jest **optymistyczna** – transakcje wykonują się równolegle bez blokad, a przy **konflikcie** jedna z nich jest **wycofywana i ponawiana**.

Historia: **Herlihy i Moss (1993)** – hardware TM; **Shavit i Touitou (1995)** – software TM (STM).

## Idea programistyczna
```java
// z blokadami – ręczny wybór blokad, kolejność (zakleszczenia!), granulacja
synchronized (a) { synchronized (b) { a.withdraw(x); b.deposit(x); } }

// z pamięcią transakcyjną – deklaratywnie
atomic {
    a.withdraw(x);
    b.deposit(x);
}
```
- Programista mówi **co** ma być atomowe, a nie **jak** to zsynchronizować.
- Transakcja: **begin** → odczyty/zapisy pamięci → **commit** (zatwierdzenie, jeśli brak konfliktu) lub **abort** (odrzucenie zmian, ponowienie).
- Własności transakcji pamięciowych: **A**tomowość, spójność (**C**) i **I**zolacja – **bez D**urability (pamięć ulotna).
- Kryterium poprawności: **serializowalność** (efekt jak przy sekwencyjnym wykonaniu transakcji); silniejsze – **nieprzezroczystość** (_opacity_, Guerraoui-Kapałka 2008): nawet transakcje, które zostaną wycofane, **nigdy nie widzą niespójnego stanu** (inaczej mogłyby dzielić przez zero, wejść w nieskończoną pętlę, odwołać się poza tablicę). Zob. [[Linearizability#7. Transakcje i serializowalność]].

## Konflikt
Dwie współbieżne transakcje są w **konflikcie**, gdy odwołują się do tej samej lokacji pamięci i **co najmniej jedna zapisuje** (write-write lub read-write).
- **zbiór odczytów** (_read set_) i **zbiór zapisów** (_write set_) transakcji – podstawa wykrywania konfliktów.

## Implementacja – wymiary projektowe
### 1. Wersjonowanie danych (zarządzanie zmianami)
| | **Gorliwe** (_eager_, in-place update) | **Leniwe** (_lazy_, buforowanie) |
|---|---|---|
| Zapis | **bezpośrednio do pamięci**, stara wartość zapisywana w **logu wycofania** (_undo log_) | do **bufora zapisów** (_redo log / write buffer_) prywatnego dla transakcji |
| Odczyt | bezpośrednio | najpierw bufor (własne zapisy), potem pamięć |
| Commit | **szybki** (usunięcie logu) | **wolniejszy** – kopiowanie bufora do pamięci |
| Abort | **wolny** – odtworzenie z undo logu | **szybki** – odrzucenie bufora |
| Wymaga | blokowania zapisywanych lokacji (izolacja) | – |

### 2. Wykrywanie konfliktów
| | **Gorliwe / pesymistyczne** (_eager_) | **Leniwe / optymistyczne** (_lazy_) |
|---|---|---|
| Kiedy | **przy każdym dostępie** (rekordy własności, blokady na obiektach) | **przy zatwierdzaniu** (walidacja zbioru odczytów) |
| Zalety | wczesne przerwanie – mniej zmarnowanej pracy | brak narzutu przy dostępie, więcej współbieżności |
| Wady | narzut na każdy dostęp, możliwe zbędne przerwania | cała praca stracona przy konflikcie; ryzyko zagłodzenia długich transakcji |

Walidacja: sprawdzenie, czy wartości/wersje odczytanych obiektów się nie zmieniły (inkrementalnie przy każdym odczycie – dla nieprzezroczystości – albo przy commit).

### 3. Rozstrzyganie konfliktów (_contention management_)
Która transakcja przegrywa? Polityki: agresor przegrywa / wygrywa, starszy wygrywa (znaczniki czasowe – zapobiega zagłodzeniu), transakcja z mniejszą ilością pracy przegrywa (_Karma_), **wykładnicze odczekiwanie** (_exponential backoff_) przed ponowieniem, priorytety, a po wielu nieudanych próbach przejście w tryb nieoptymistyczny (globalna blokada).

### 4. Granulacja
na poziomie **obiektu** (DSTM, Java/C#), **słowa pamięci** (C/C++), **linii pamięci podręcznej** (HTM).

### 5. Izolacja względem kodu nietransakcyjnego
- **słaba atomowość** (_weak atomicity_) – gwarancje tylko między transakcjami; kod poza transakcją może widzieć/psuć stan pośredni,
- **silna atomowość** (_strong_) – także dostęp spoza transakcji traktowany jak mini-transakcja (większy narzut).

## Implementacje programowe (STM)
### TL2 – Transactional Locking II (Dice, Shalev, Shavit 2006)
Leniwe wersjonowanie, leniwe wykrywanie konfliktów zapisu, **globalny zegar wersji**.
- **Globalny zegar** $GV$ (licznik), każda lokacja (obiekt) ma **zamek z wersją** (bit blokady + numer wersji ostatniego zapisu).
- **Start**: $RV := GV$ (wersja odczytu – migawka).
- **Odczyt** $x$: jeśli jest w buforze zapisów – z bufora; w przeciwnym razie odczytaj wartość i wersję; jeśli $x$ **zablokowany** lub $wersja(x) > RV$ → **abort** (ktoś zmienił po starcie – gwarantuje nieprzezroczystość); dodaj do read set.
- **Zapis**: do bufora (write set).
- **Commit**:
  1. **zablokuj** wszystkie lokacje z write set (w ustalonej kolejności / z timeoutem; niepowodzenie → abort),
  2. $WV := ++GV$ (atomowo, CAS),
  3. **waliduj read set**: każda lokacja nie jest zablokowana przez inną transakcję i $wersja \leq RV$ (jeśli $WV = RV+1$ – walidację można pominąć),
  4. zapisz wartości z bufora, ustaw ich wersje na $WV$ i **odblokuj**.
- Transakcje tylko do odczytu: brak bufora i blokad – bardzo tanie.

### Inne algorytmy
- **DSTM** (Herlihy i in. 2003) – obiektowa, nieblokująca (obstruction-free), lokatory obiektów z wersjami,
- **McRT-STM**, **TinySTM** (gorliwe wersjonowanie + blokady zapisu przy dostępie), **SwissTM**,
- **NOrec** (Dalessandro 2010) – jeden globalny zamek sekwencyjny (seqlock) i walidacja przez **wartości**, brak metadanych per obiekt,
- **MVCC w STM** – wiele wersji obiektu, odczyty nigdy nie przerywane (Clojure, LSA).

### Wsparcie w językach i bibliotekach
- **Haskell STM** (Harris, Marlow, Peyton Jones, Herlihy 2005) – `TVar` (zmienne transakcyjne), `atomically :: STM a -> IO a`, **system typów** uniemożliwia wykonanie I/O w transakcji, **`retry`** (zablokuj do zmiany odczytanych TVar – synchronizacja warunkowa!), **`orElse`** (alternatywa – kompozycja wyborów):
  ```haskell
  transfer :: TVar Int -> TVar Int -> Int -> STM ()
  transfer from to n = do
      bal <- readTVar from
      when (bal < n) retry            -- czekaj, aż saldo wystarczy
      writeTVar from (bal - n)
      modifyTVar' to (+ n)

  main = atomically (transfer a b 100)
  ```
- **Clojure** – `ref`, `dosync`, `alter`, `commute` (operacje przemienne – mniej konfliktów), MVCC ze snapshot isolation,
- **Scala-STM** (`atomic { implicit txn => ... }`), **ScalaSTM**, **Multiverse** (Java), **Akka STM** (dawniej),
- **C/C++**: GCC `-fgnu-tm` z `__transaction_atomic { }` / `__transaction_relaxed { }` (specyfikacja TM TS dla C++ – niewdrożona do standardu),
- .NET – eksperymentalny STM.NET (porzucony),
- **Rust**, **Kotlin** – biblioteki.

## Implementacje sprzętowe (HTM)
- Procesor śledzi zbiory odczytów/zapisów w **pamięci podręcznej** (bity w liniach cache), a bufor zapisów trzyma w sprzęcie. Konflikt wykrywany przez mechanizmy spójności cache między rdzeniami.
- **Intel TSX** (Haswell 2013):
  - **RTM** (_Restricted TM_): `_xbegin()` (zwraca `_XBEGIN_STARTED` lub kod przyczyny przerwania), `_xend()`, `_xabort(kod)`, `_xtest()`,
  - **HLE** (_Hardware Lock Elision_) – prefiksy przy blokadach: sekcja krytyczna wykonywana spekulatywnie bez zajęcia blokady, przy konflikcie ponownie z blokadą,
  - wyłączany mikrokodem w wielu procesorach (błędy – Haswell, luki TAA/ZombieLoad 2019); HLE usunięte.
- **IBM**: Blue Gene/Q (pierwsze komercyjne HTM, 2012), POWER8 (`tbegin/tend`, suspend/resume), z/Architecture (zEC12); **ARM TME** (specyfikacja), **Sun Rock** (anulowany).
- **Best-effort**: sprzęt **nie gwarantuje** zatwierdzenia – transakcja przerywana przy: przepełnieniu pojemności (rozmiar cache), przerwaniu, wywołaniu systemowym, chybieniu strony, instrukcjach niedozwolonych (np. `CPUID`), przełączeniu kontekstu → **obowiązkowa ścieżka awaryjna** (_fallback_) z blokadą:
  ```c
  int retries = 3;
  while (retries--) {
      unsigned s = _xbegin();
      if (s == _XBEGIN_STARTED) {
          if (fallback_lock_held) _xabort(0xff);   // odczyt blokady → konflikt z ścieżką awaryjną
          /* sekcja krytyczna */
          _xend();
          return;
      }
      if (!(s & _XABORT_RETRY)) break;
  }
  pthread_mutex_lock(&fallback_lock); /* sekcja krytyczna */ pthread_mutex_unlock(&fallback_lock);
  ```
- **Hybrydowa TM** (HyTM, Hybrid NOrec) – szybka ścieżka HTM, ścieżka awaryjna STM zamiast globalnej blokady.

## Zastosowania
- **uproszczenie programowania współbieżnego** – struktury danych z wieloma obiektami (drzewa zrównoważone, grafy, słowniki) bez złożonych schematów blokad,
- **kompozycja operacji** atomowych (przelew między dwiema kolejkami współbieżnymi – niemożliwy przez złożenie operacji na blokadach wewnętrznych),
- **lock elision** – przyspieszenie istniejących programów z grubą granulacją blokad (glibc elision w `pthread_mutex`, bazy danych w pamięci, Java `synchronized` w badaniach),
- **bazy danych w pamięci**, systemy gier (stan świata), języki funkcyjne (Haskell, Clojure – naturalnie dzięki niemutowalności),
- **zrównoleglanie spekulatywne** (_thread-level speculation_) – pętle, których niezależność nie jest pewna.

## Zalety
- **Kompozycyjność** – transakcje można zagnieżdżać i łączyć (w przeciwieństwie do blokad, gdzie złożenie operacji wymaga ujawnienia blokad),
- **brak zakleszczeń** i odwrócenia priorytetów związanych z blokadami (brak jawnych blokad programisty),
- **deklaratywność** – mniej błędów (zapomniane blokady, zła kolejność), prostszy kod,
- **optymistyczna współbieżność** – dobra skalowalność, gdy konflikty są rzadkie (dużo odczytów, rozłączne dane): transakcje na różnych danych nie blokują się, bez zgadywania granulacji blokad,
- **automatyczna odporność na wyjątki** – abort przywraca stan,
- `retry`/`orElse` – elegancka synchronizacja warunkowa.

## Wady
- **Narzut STM** – instrumentacja każdego dostępu, bufory, walidacja: 2–10× wolniej niż sekwencyjnie przy jednym wątku; HTM – mały narzut, ale ograniczona pojemność,
- **operacje nieodwracalne** (I/O, wywołania systemowe, wysłanie pakietu) – nie można ich wycofać: zakaz (Haskell), buforowanie, tryb nieodwołalny (_irrevocable_ – transakcja wykonywana samodzielnie),
- **zmarnowana praca** i słaba wydajność przy **częstych konfliktach** (długie transakcje, „gorące” liczniki), **livelock** i **zagłodzenie** (długie transakcje ciągle przerywane przez krótkie),
- **słaba izolacja** w wielu implementacjach – interakcja z kodem nietransakcyjnym, **nieprzezroczystość** trudna i kosztowna,
- **HTM best-effort** – brak gwarancji postępu, konieczność ścieżki awaryjnej, różnice między procesorami, wyłączanie z powodu błędów i luk bezpieczeństwa,
- **debugowanie i profilowanie** trudne (niewidoczne przerwania), przewidywalność wydajności niska,
- **semantyka** zagnieżdżania (płaskie, zamknięte, otwarte), wyjątków i interakcji z `wait/notify` niejednoznaczna,
- nie rozwiązuje wszystkich problemów współbieżności – błędy atomowości przy źle dobranych granicach transakcji ([[39 Wykrywanie wyścigów wysokiego poziomu]]),
- ograniczone wsparcie w językach mainstreamowych (C++/Java/C# nie mają TM w standardzie).

## TM a blokady – podsumowanie
| | Blokady | Pamięć transakcyjna |
|---|---|---|
| Model | pesymistyczny | optymistyczny |
| Kompozycja | trudna | naturalna |
| Zakleszczenia | możliwe | brak (możliwy livelock) |
| Operacje I/O w sekcji | tak | problematyczne |
| Wydajność przy rzadkich konfliktach | ograniczona granulacją | wysoka |
| Wydajność przy częstych konfliktach | przewidywalna | spada (ponowienia) |
| Wsparcie | powszechne | Haskell, Clojure, GCC, TSX (częściowo) |

## Zobacz też
- [[Synchronizacja]] (CAS, LL/SC, operacje spekulatywne), [[43 Model pamięci w języku Java]]
- [[24 Niezawodne zatwierdzanie transakcji rozproszonych]] – transakcje w systemach rozproszonych
