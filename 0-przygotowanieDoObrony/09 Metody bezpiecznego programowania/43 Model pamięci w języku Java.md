---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 43
---
# 43. Model pamięci w języku Java
---
> **Model pamięci Javy** (JMM – _Java Memory Model_, JLS rozdz. 17, poprawiony w **JSR-133** dla Javy 5, W. Pugh, J. Manson i in.) określa **semantykę dostępu wątków do pamięci współdzielonej**. Mówi, kiedy zapis wykonany przez jeden wątek jest **gwarantowanie widoczny** dla innego wątku i które przeploty i optymalizacje są dozwolone. Jest to **kontrakt** między programistą a JVM/kompilatorem JIT/procesorem: poprawnie zsynchronizowany program zachowuje się przewidywalnie niezależnie od sprzętu.

> [!note] Dwa znaczenia
> „Model pamięci Javy” bywa też rozumiany jako **podział pamięci JVM na obszary** (sterta, stosy wątków, metaspace). Opisano to na końcu notatki. W kontekście MBP chodzi przede wszystkim o **JMM – semantykę współbieżności**.

## Motywacja – dlaczego potrzebny jest model
Nowoczesny sprzęt i kompilatory **nie wykonują programu dosłownie**:
- **pamięć podręczna procesora** – każdy rdzeń ma własne cache L1/L2; zapis może jeszcze nie być widoczny w cache innego rdzenia,
- **bufory zapisu** (_store buffers_) – procesor x86 odkłada zapisy i kontynuuje wykonanie,
- **zmiana kolejności instrukcji** przez procesor (_out-of-order execution_) i **kompilator JIT** (rejestry zamiast pamięci, eliminacja zbędnych odczytów/zapisów, przenoszenie odczytów przed pętlę, inlining),
- **słabe modele pamięci** (_relaxed memory models_) procesorów – [[Synchronizacja]].

Bez synchronizacji wątek może więc widzieć **nieaktualne** wartości (w nieskończoność), wartości **pośrednie** albo w **innej kolejności**, niż wskazuje kod. Sekwencyjna spójność (ang. _sequential consistency_) **nie jest** gwarantowana przez sprzęt.

### Przykład – brak widoczności
```java
public class NoVisibility {
    private static boolean ready;
    private static int number;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (!ready) Thread.yield();   // może nigdy nie zobaczyć ready == true
            System.out.println(number);       // może wypisać 0!
        }).start();
        number = 42;
        ready = true;
    }
}
```
JIT może zamienić pętlę na `if (!ready) while (true) yield();` (odczyt `ready` raz), a wątek może zobaczyć `ready = true` przed `number = 42`.

### Przykład – TSO (x86)
```
Wątek 1:  x = 1;  r1 = y;
Wątek 2:  y = 1;  r2 = x;
```
Przy spójności sekwencyjnej niemożliwe jest $r1 = r2 = 0$. Na x86 (**TSO – _Total Store Order_**) zapisy trafiają najpierw do **bufora zapisu** rdzenia, więc oba odczyty mogą przeczytać stare 0 → wynik $r1 = r2 = 0$ jest możliwy. Z tego powodu klasyczne algorytmy wzajemnego wykluczania (Dekker, Peterson) **nie działają** bez barier pamięci. Na ARM/POWER modele są jeszcze słabsze.

## Wyścig danych w JMM
**Wyścig danych**: dwa wątki odwołują się do tej samej zmiennej (pole, element tablicy – **nie** zmienne lokalne, które są prywatne dla wątku), co najmniej jeden zapisuje, i dostępy nie są uporządkowane relacją happens-before (brak synchronizacji).
- JMM dopuszcza w takim przypadku **wiele wyników**, ale – w odróżnieniu od C/C++ – **nie niezdefiniowane zachowanie**: Java gwarantuje **bezpieczeństwo typów i pamięci** (odczytana wartość to jakaś wartość zapisana wcześniej lub domyślna; brak odczytu „śmieci” poza typem – z wyjątkiem niekoniecznie atomowych zapisów `long`/`double`).
- Zapisy/odczyty `long` i `double` (64-bit) **nie muszą być atomowe** na 32-bitowych JVM (możliwy odczyt połowy starej, połowy nowej wartości) – JLS 17.7.

## Relacja happens-before
> Jeśli zdarzenie A **happens-before** B, to efekty A są **gwarantowanie widoczne** w B. Jeśli między dwoma dostępami nie ma relacji happens-before, JVM może je przeplatać i optymalizować dowolnie.

**Reguły** (JLS 17, JSR-133):
1. **Kolejność w wątku** – instrukcja wcześniejsza w tym samym wątku happens-before późniejszej.
2. **Blokada monitora** – `unlock` monitora happens-before każdego **kolejnego** `lock` tego monitora (wątek wchodzący do `synchronized` widzi wszystkie zapisy wykonane pod tą samą blokadą).
3. **Pole `volatile`** – zapis do pola `volatile` happens-before każdego kolejnego odczytu tego pola.
4. **`Thread.start()`** – wywołanie `start()` happens-before pierwszej akcji uruchamianego wątku (nowy wątek widzi stan sprzed `start`).
5. **`Thread.join()`** – zakończenie wątku happens-before powrotu z `join()` w wątku oczekującym.
6. **Przerwanie** – `interrupt()` happens-before wykrycia przerwania (`isInterrupted`, `InterruptedException`).
7. **Konstrukcja obiektu** – zakończenie konstruktora happens-before finalizacji obiektu (`finalize`).
8. **Inicjalizacja klasy** – statyczny inicjalizator wykonuje się raz, pod niejawną blokadą, a zakończenie inicjalizacji jest widoczne dla wszystkich wątków.
9. **Pola `final`** – po zakończeniu konstruktora (jeśli `this` nie „uciekło”) wartości pól `final` są widoczne dla wszystkich wątków, także bez synchronizacji (reguła JSR-133).
10. **Klasy `java.util.concurrent`** – dokumentują własne gwarancje (np. `CountDownLatch.countDown` happens-before powrotu z `await`, obiekt włożony do `BlockingQueue` jest widoczny dla pobierającego, `Future.get` widzi wynik zadania).

Relacja jest **tranzytywna**.

## Mechanizmy
### `synchronized`
Daje **wzajemne wykluczanie** (atomowość sekcji) **i widoczność** (reguła 2). Wszystkie odczyty i zapisy zmiennej współdzielonej muszą być pod **tą samą** blokadą (także odczyty!) – [[37 Monitory w C Sharp i Java]].

### `volatile`
- **Widoczność** – odczyt zawsze widzi ostatni zapis (JIT nie może buforować wartości w rejestrze; zapis tworzy barierę pamięci).
- **Brak atomowości operacji złożonych**: `count++` na polu `volatile` to odczyt-modyfikacja-zapis → wyścig (utracone aktualizacje).
- **Kiedy wystarcza**: zapis nie zależy od aktualnej wartości (flagi `running = false`), jeden wątek zapisuje, wielu czyta, zmienna nie uczestniczy w niezmienniku z innymi zmiennymi.
- Tańsze od blokad (brak blokowania), ale każdy zapis to kosztowna bariera (nieważność cache).

### `final` i niemutowalność
- Pola `final` przypisane w konstruktorze: po poprawnej konstrukcji są widoczne dla wszystkich wątków bez synchronizacji.
- Obiekty **niemutowalne** (wszystkie pola `final`, stan niezmienny, referencje do obiektów niemutowalnych lub niedostępne na zewnątrz) są **zawsze bezpieczne wątkowo** (`String`, `Integer`, rekordy z niemutowalnymi polami).
- **Ucieczka `this` z konstruktora** (rejestracja listenera, uruchomienie wątku w konstruktorze) → inny wątek może zobaczyć obiekt częściowo skonstruowany, nawet pola `final` z wartością domyślną.

### Klasy atomowe i `VarHandle`
`AtomicInteger`, `AtomicLong`, `AtomicReference`, `LongAdder` (liczniki o wysokiej współbieżności) – operacje **CAS** (_compare-and-set_), np. `incrementAndGet()`; `VarHandle` (Java 9) – niskopoziomowe CAS, `getOpaque`/`setOpaque`, `getAcquire`/`setRelease` z jawnie określoną semantyką widoczności.

### `ThreadLocal`
Zmienne prywatne dla wątku – brak współdzielenia, więc nie ma wyścigu.

## Bezpieczna publikacja obiektu
Obiekt jest **bezpiecznie opublikowany**, gdy referencja do niego i jego stan stają się widoczne dla innych wątków **jednocześnie**. Sposoby:
- inicjalizacja w **statycznym inicjalizatorze**,
- zapis referencji do pola **`volatile`** lub `AtomicReference`,
- zapis do pola **`final`** poprawnie skonstruowanego obiektu,
- zapis pod **blokadą** / przez **współbieżną kolekcję** (`ConcurrentHashMap`, `BlockingQueue`).

Niebezpieczna publikacja: `public Holder holder; ... holder = new Holder(42);` – inny wątek może zobaczyć referencję do obiektu, zanim konstruktor ustawi pola (kolejność „przydziel pamięć → przypisz referencję → wykonaj konstruktor” jest dozwolona dla JIT).

## Double-Checked Locking – klasyczny błąd
```java
// BŁĘDNE przed Javą 5 i bez volatile
class Singleton {
    private static Singleton instance;
    public static Singleton getInstance() {
        if (instance == null) {                    // 1. sprawdzenie bez blokady
            synchronized (Singleton.class) {
                if (instance == null)              // 2. sprawdzenie pod blokadą
                    instance = new Singleton();    // 3. przydział + przypisanie + konstruktor
            }
        }
        return instance;                           // może zwrócić NIEZAINICJALIZOWANY obiekt
    }
}
```
**Problem**: przypisanie `instance` może nastąpić **przed** zakończeniem konstruktora (dozwolona zmiana kolejności). Inny wątek w kroku 1 widzi `instance != null` bez synchronizacji → brak happens-before → używa częściowo skonstruowanego obiektu.

**Poprawki**:
```java
// 1. volatile (poprawne od JSR-133 / Java 5)
private static volatile Singleton instance;

// 2. Initialization-on-demand holder idiom – leniwie i bez synchronizacji
class Singleton {
    private Singleton() {}
    private static class Holder { static final Singleton INSTANCE = new Singleton(); }
    public static Singleton getInstance() { return Holder.INSTANCE; }  // inicjalizacja klasy jest bezpieczna wątkowo
}

// 3. enum
enum Singleton { INSTANCE; }
```

## Zmiany wprowadzone przez JSR-133 (Java 5)
- **precyzyjna definicja `volatile`** (wcześniej niejasna – DCL z `volatile` nie działał),
- **gwarancje pól `final`** (wcześniej inny wątek mógł zobaczyć zmianę wartości pola `final`, np. `String` z niewłaściwym `offset`),
- sformalizowanie happens-before i dopuszczalnych optymalizacji,
- semantyka `wait`/`notify` z przerwaniami.

## Obszary pamięci JVM (drugie znaczenie)
| Obszar | Współdzielony | Zawartość | Błąd przy braku miejsca |
|---|---|---|---|
| **Sterta** (_heap_) | ✔ przez wszystkie wątki | **obiekty i tablice**, pola instancyjne | `OutOfMemoryError: Java heap space` |
| **Metaspace** (Java 8+; wcześniej PermGen) | ✔ | metadane klas, stałe, kod metod (poza stertą) | `OutOfMemoryError: Metaspace` |
| **Stos wątku** (_JVM stack_) | ✘ prywatny | **ramki** metod: zmienne **lokalne** (prymitywy, referencje), stos operandów | `StackOverflowError` |
| **Rejestr PC** | ✘ | adres bieżącej instrukcji | – |
| **Stos metod natywnych** | ✘ | wywołania JNI | `StackOverflowError` |
| **Code cache** | ✔ | skompilowany kod JIT | – |
| pamięć bezpośrednia | ✔ | `ByteBuffer.allocateDirect` | `OutOfMemoryError: Direct buffer memory` |

- **Zmienne lokalne i parametry** są prywatne dla wątku (bezpieczne), a **obiekty**, na które wskazują, leżą na **współdzielonej stercie** (analiza ucieczki JIT – _escape analysis_ – może alokować nieuciekające obiekty na stosie lub rozbić je na skalary).
- **Sterta pokoleniowa** (GC): **młode pokolenie** (Eden + 2 obszary Survivor – kolekcje częste, kopiujące; większość obiektów „umiera młodo”), **stare pokolenie** (obiekty długowieczne, kolekcje rzadsze), odśmiecacze: Serial, Parallel, **G1** (domyślny, regiony), **ZGC**, **Shenandoah** (niskie pauzy, współbieżne kompaktowanie z barierami odczytu/zapisu).
- **TLAB** (_Thread-Local Allocation Buffer_) – każdy wątek przydziela obiekty z własnego kawałka Edenu bez synchronizacji.
- Parametry: `-Xms`, `-Xmx` (sterta), `-Xss` (rozmiar stosu wątku), `-XX:MaxMetaspaceSize`.
