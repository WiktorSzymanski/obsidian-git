---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 40
---
# 40. Sprawdzanie modelu (_model checking_) na przykładzie Java PathFinder
---
> **Sprawdzanie modelu** to automatyczna technika weryfikacji formalnej. Systematycznie przegląda się **wszystkie osiągalne stany** skończonego modelu systemu i sprawdza, czy spełnia on zadaną **własność** (zwykle wyrażoną w logice temporalnej). Gdy własność jest naruszona, narzędzie zwraca **kontrprzykład** – ścieżkę wykonania prowadzącą do błędu.

(E. Clarke, E. A. Emerson, J. Sifakis – Nagroda Turinga 2007.)

## Podstawy
### Model – struktura Kripkego
$M = (S, S_0, R, L)$:
- $S$ – skończony zbiór **stanów** (wartości zmiennych, liczniki programu, stosy wątków),
- $S_0 \subseteq S$ – stany początkowe,
- $R \subseteq S \times S$ – **relacja przejść** (kroki programu, w tym przeploty wątków),
- $L: S \to 2^{AP}$ – etykietowanie stanów zdaniami atomowymi prawdziwymi w stanie.

Pytanie: $M \models \varphi$ ?

### Własności
- **Bezpieczeństwo** (_safety_) – „nic złego się nie stanie”: brak zakleszczenia, asercja nigdy nie jest fałszywa, brak wyjątku, wzajemne wykluczanie. Naruszenie ma **skończony** kontrprzykład (ścieżkę do złego stanu). Sprawdzane przez przeszukanie osiągalnych stanów.
- **Żywotność** (_liveness_) – „coś dobrego w końcu nastąpi”: każde żądanie zostanie obsłużone, proces wejdzie do sekcji krytycznej. Kontrprzykład to **nieskończona** ścieżka (cykl – _lasso_). Wymaga wyszukiwania cykli akceptujących (automaty Büchiego, nested DFS), często z założeniami uczciwości (_fairness_).

### Logiki temporalne
- **LTL** (liniowa) – formuły o pojedynczych ścieżkach: $\mathbf{G}\,p$ (zawsze), $\mathbf{F}\,p$ (kiedyś), $\mathbf{X}\,p$ (w następnym stanie), $p\,\mathbf{U}\,q$ ($p$ aż do $q$). Przykłady: $\mathbf{G}\,\neg(cs_1 \wedge cs_2)$ (wzajemne wykluczanie), $\mathbf{G}(req \rightarrow \mathbf{F}\,grant)$ (każde żądanie obsłużone).
- **CTL** (rozgałęziona) – kwantyfikatory ścieżek $\mathbf{A}$ (wszystkie), $\mathbf{E}$ (istnieje): $\mathbf{AG}\,\mathbf{EF}\,reset$ (z każdego stanu da się wrócić do stanu początkowego).

### Rodzaje sprawdzania modelu
| Rodzaj | Idea | Przykłady |
|---|---|---|
| **jawnych stanów** (_explicit-state_) | przeszukiwanie grafu stanów (DFS/BFS), stany odwiedzone w tablicy haszującej | **SPIN** (Promela), **Java PathFinder**, Murφ, TLC (TLA+) |
| **symboliczne** | zbiory stanów reprezentowane **BDD** (binarne diagramy decyzyjne), obliczanie punktów stałych | SMV, NuSMV |
| **ograniczone** (_bounded_, BMC) | rozwinięcie przejść na $k$ kroków i sprowadzenie do **SAT/SMT** | CBMC, ESBMC |
| **abstrakcja z uszczegółowianiem** (CEGAR) | sprawdzanie modelu abstrakcyjnego, uszczegółowianie po fałszywym kontrprzykładzie | SLAM, BLAST, CPAchecker |

### Problem eksplozji przestrzeni stanów
Liczba stanów rośnie **wykładniczo** z liczbą zmiennych i wątków; liczba przeplotów $n$ wątków wykonujących po $k$ kroków to $\dfrac{(nk)!}{(k!)^n}$.

**Techniki redukcji**:
- **redukcja częściowego porządku** (_partial order reduction_, POR) – przeploty różniące się tylko kolejnością **niezależnych** operacji (np. dostępów do zmiennych lokalnych) prowadzą do tego samego stanu → wystarczy jeden reprezentant,
- **dopasowanie stanów** (_state matching_) z haszowaniem, **kompresja** (collapse), **bitstate hashing** (supertrace – przybliżone, może pominąć stany),
- **abstrakcja** danych (zakresy zamiast konkretnych wartości), **redukcja symetrii** (identyczne wątki),
- **przeszukiwanie z heurystykami** (najpierw ścieżki prawdopodobnie błędne), ograniczenie głębokości,
- **wykonanie symboliczne** zamiast wyliczania wartości danych.

### Model checking a testowanie
| | Testowanie | Sprawdzanie modelu |
|---|---|---|
| Przeploty | jeden (losowy, zależny od planisty) | **wszystkie** (modulo redukcje) |
| Powtarzalność błędu | trudna | kontrprzykład = deterministyczna ścieżka |
| Pokrycie | niepełne | pełne dla skończonego modelu |
| Skalowalność | duże systemy | ograniczona (eksplozja stanów) |

---
## Java PathFinder (JPF)
> **Java PathFinder** – sprawdzarka modeli **jawnych stanów** dla programów w **kodzie bajtowym Javy**, rozwijana w **NASA Ames Research Center** (K. Havelund, W. Visser, od 1999; open source od 2005). Zamiast budować model ręcznie, JPF **wykonuje rzeczywisty program** we **własnej maszynie wirtualnej** i systematycznie eksploruje wszystkie ścieżki wynikające z niedeterminizmu.

Historia: JPF1 tłumaczył Javę na Promelę (SPIN); **JPF2** – własna JVM napisana w Javie.

### Architektura
```
                ┌──────────────────────────────── JPF ────────────────────────────────┐
program.class ─▶│  VM (JVM w Javie): interpretacja kodu bajtowego, stan: sterta,     │
                │     wątki (stosy ramek), pola statyczne                             │
 *.jpf config ─▶│  Search: DFS / BFS / heurystyki  ◀─▶ tworzenie, zapisywanie,       │─▶ raport:
                │                                       porównywanie, przywracanie   │   ścieżka błędu
                │                                       stanów (backtracking)        │   (trace)
                │  ChoiceGenerators: planista wątków, Verify.getInt(), ...            │
                │  Properties: NoDeadlock, NoUncaughtExceptions, asercje             │
                │  Listeners: rozszerzenia (wykrywanie wyścigów, pomiary)             │
                │  Model classes / MJI native peers: biblioteki, kod natywny          │
                └─────────────────────────────────────────────────────────────────────┘
```
- **VM** – interpretuje instrukcje kodu bajtowego i zarządza **stanem programu** (sterta, stosy, zmienne statyczne, stany wątków i monitorów).
- **Search** – steruje eksploracją: po każdym **przejściu** (_transition_ – sekwencja instrukcji między punktami wyboru) sprawdza, czy nowy stan był już **odwiedzony** (**dopasowanie stanów** przez serializację/haszowanie stanu); jeśli tak – **wycofanie** (_backtrack_), jeśli nie – kontynuacja. Po zakończeniu gałęzi wraca do ostatniego punktu wyboru z niewyczerpanymi alternatywami (**przywracanie zapisanego stanu**).
- **Choice Generators** (generatory wyborów) – źródła niedeterminizmu:
  - **planowanie wątków** – w **punktach planowania** (_scheduling points_) wybierany jest każdy z możliwych do uruchomienia wątków,
  - **niedeterminizm danych** – jawne wybory w kodzie testowym:
    ```java
    import gov.nasa.jpf.vm.Verify;
    int n = Verify.getInt(0, 3);          // JPF sprawdzi n = 0, 1, 2, 3
    boolean b = Verify.getBoolean();      // oba przypadki
    ```
  - własne generatory (np. zdarzenia GUI, zakresy liczb zmiennoprzecinkowych).
- **Redukcja częściowego porządku** w JPF: przełączanie wątków tylko przed instrukcjami o **widocznym efekcie** dla innych wątków – dostęp do pól **współdzielonych** (osiągalnych z wielu wątków), operacje monitorów (`monitorenter/exit`, `wait/notify`), `Thread.start/join/yield`. Instrukcje lokalne wykonywane są w jednym przejściu. Znacząco zmniejsza liczbę stanów.
- **Właściwości** (_properties_) sprawdzane po każdym przejściu:
  - `gov.nasa.jpf.vm.NoDeadlockedProperty` – brak **zakleszczenia** (wszystkie wątki zablokowane, a nie wszystkie zakończone),
  - `gov.nasa.jpf.vm.NoUncaughtExceptionsProperty` – brak nieobsłużonych wyjątków (w tym `AssertionError`, `NullPointerException`, `ArrayIndexOutOfBounds`),
  - `NotDeadlockedProperty`, `NoOutOfMemoryErrorProperty`, własne właściwości.
- **Listeners** (słuchacze) – mechanizm rozszerzeń na zdarzenia VM i wyszukiwania (wykonanie instrukcji, nowy stan, wycofanie, znaleziony błąd):
  - **PreciseRaceDetector** – precyzyjne wykrywanie **wyścigów danych**: czy w danym stanie dwa wątki mogą wykonać dostęp do tego samego pola (zapis+odczyt/zapis) w kolejnych krokach,
  - wykrywanie **wyścigów wysokiego poziomu** (zgodność widoków – [[39 Wykrywanie wyścigów wysokiego poziomu]]),
  - `DeadlockAnalyzer`, `ExecTracker` (śledzenie), `CoverageAnalyzer`, `StateSpaceDot` (graf stanów), `BudgetChecker` (limity czasu/pamięci).
- **Klasy modelowe i MJI** (_Model Java Interface_) – kod natywny i duże biblioteki (I/O, sieć, refleksja) nie mogą być wykonywane w JPF VM. **Klasy modelowe** (uproszczone implementacje) i **native peers** (metody wykonywane w **hostowej** JVM, poza eksploracją stanów) zapewniają ich obsługę i redukują stan.

### Konfiguracja i uruchomienie
```properties
# Philosophers.jpf
target = Philosophers
classpath = build/classes
search.class = gov.nasa.jpf.search.DFSearch
listener = gov.nasa.jpf.listener.PreciseRaceDetector
search.properties = gov.nasa.jpf.vm.NotDeadlockedProperty,gov.nasa.jpf.vm.NoUncaughtExceptionsProperty
search.multiple_errors = false
vm.por = true
report.console.property_violation = error,trace,snapshot
```
```bash
bin/jpf Philosophers.jpf
```
Wynik przy błędzie: **nazwa naruszonej właściwości**, **ścieżka** (_trace_) – sekwencja przejść z informacją, który wątek wykonał jakie instrukcje i jakie wybory zostały podjęte; **migawka** stanu wątków; statystyki (liczba stanów nowych/odwiedzonych, wycofań, maksymalna głębokość, zużycie pamięci).

### Przykład – wyścig i zakleszczenie
```java
public class Racer implements Runnable {
    int d = 42;
    public void run() { doSomething(1001); d = 0; }           // zapis w wątku t

    public static void main(String[] args) {
        Racer racer = new Racer();
        Thread t = new Thread(racer);
        t.start();
        doSomething(1000);
        int c = 420 / racer.d;                                // odczyt w main → ArithmeticException gdy d==0
        System.out.println(c);
    }
    static void doSomething(int n) { try { Thread.sleep(n); } catch (InterruptedException e) {} }
}
```
Testy prawie zawsze przejdą (opóźnienia sugerują kolejność), a JPF znajdzie przeplot, w którym `d = 0` wykona się przed dzieleniem → `NoUncaughtExceptionsProperty` naruszona; `PreciseRaceDetector` zgłosi wyścig na `Racer.d`.

Klasyczny przykład zakleszczenia – **filozofowie** biorący widelce w tej samej kolejności: JPF znajduje ścieżkę, w której każdy wziął lewy widelec → `NotDeadlockedProperty`.

### Rozszerzenia (projekty jpf-*)
- **jpf-symbc – Symbolic PathFinder (SPF)** – **wykonanie symboliczne**: wejścia jako zmienne symboliczne, zbieranie **warunków ścieżki** (_path conditions_), rozwiązywanie solverem ograniczeń (Choco, Z3) → **generacja przypadków testowych** osiągających pokrycie gałęzi, wykrywanie błędów zależnych od danych,
- **jpf-nhandler** – automatyczne delegowanie metod natywnych do hostowej JVM,
- **jpf-awt** (GUI), **jpf-net-iocache** (sieć), **jpf-concurrent** (modele `java.util.concurrent`), **net-iocache**, **jpf-probabilistic**,
- sprawdzanie LTL (jpf-ltl), analiza programów Android (jpf-android).

### Zastosowania
Weryfikacja oprogramowania NASA (moduły systemu wykonawczego Remote Agent – znaleziono realne zakleszczenia; komponenty misji marsjańskich), testowanie algorytmów współbieżnych, dydaktyka, generowanie testów.

### Zalety i ograniczenia JPF
✔ działa na **rzeczywistym kodzie** (bez ręcznego modelu), sprawdza **wszystkie przeploty** i wartości z generatorów wyborów, **deterministyczny kontrprzykład**, rozszerzalny (listeners), wykrywa zakleszczenia, wyścigi, wyjątki, naruszenia asercji.

✘ **eksplozja stanów** – praktyczny limit rzędu **~10 tys. linii** kodu aplikacji (plus biblioteki), programy muszą być **zamknięte** (środowisko modelowane: sterownik testowy, modele I/O), wolniejszy od JVM o rzędy wielkości, ograniczona obsługa bibliotek natywnych, sieci i GUI, pamięć na przechowywanie stanów, wynik zależy od zakresu niedeterminizmu wprowadzonego przez użytkownika (sprawdzone jest to, co wymodelowano).

## Zobacz też
- [[38 Algorytm Eraser]] – dynamiczne wykrywanie wyścigów bez eksploracji przeplotów
- [[Linearizability]] – kryterium poprawności sprawdzane np. przez sprawdzarki linearyzowalności
