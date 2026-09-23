---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 9
source: "slajdy–merged.pdf"
slajdy: "153–183"
---
# NPR 07. Pakiet `java.util.concurrent`
---
> Mechanizmy „wysokopoziomowe" zapowiedziane na slajdzie 142 — wszystko, czego nie dało się sensownie zbudować z `synchronized` i `wait`/`notify` (zob. [[NPR 06 Wątki w Javie i elementarna synchronizacja#Czy da się zbudować monitor?|NPR 06]]). Wykład idzie od najniższego poziomu w górę: **atomowość i widoczność** (`volatile`, pakiet `atomic`), **kolekcje współbieżne**, **klasyczne mechanizmy synchronizacji** (semafor, bariery, zamki ze zmiennymi warunkowymi), **pule wątków** (`Executor`, `Future`, `Callable`) i na końcu podejście alternatywne — **obiekty niezmienne**, które nie wymagają synchronizacji w ogóle.

---
## Dwa wymiary synchronizacji
<sub>slajdy–merged.pdf, slajd 154</sub>

| Wymiar | Czego dotyczy |
|---|---|
| **Przepływ sterowania** | kontrola **kolejności realizacji operacji** w ramach współbieżnych wątków |
| **Przepływ danych** | **dostępność aktualnych danych** podczas realizacji współbieżnego dostępu |

To ten sam podział, co na slajdzie 140 („synchronizacja procesów/wątków" vs „synchronizacja danych"), powtórzony przed omówieniem pakietu, bo poszczególne narzędzia `java.util.concurrent` obsługują **różne wymiary**: `CyclicBarrier` to przepływ sterowania, `volatile` to wyłącznie przepływ danych, a `BlockingQueue` — oba naraz.

### Trzy poziomy wsparcia
<sub>slajd 155</sub>

| Poziom | Wsparcie |
|---|---|
| **architektura systemu komputerowego** (procesor) | **atomowy zapis/odczyt pamięci**; **atomowa realizacja operacji złożonych** |
| **system operacyjny** | integracja mechanizmów synchronizacji z **kontrolą stanu (cyklu życia)** wątku/procesu |
| **język programowania** | konstrukcje programotwórcze określające **przepływ sterowania** i **sposób dostępu do współdzielonych danych** |

---
## Atomowość
<sub>slajdy–merged.pdf, slajd 156</sub>

Operacja atomowa jest:

| Własność | Znaczenie |
|---|---|
| **realizowana w całości** (**niepodzielność**) | **albo się zakończy, albo w ogóle się nie rozpocznie** |
| **realizowana w „momencie" czasu** (**izolacja**) | **żadne jej efekty nie ujawniają się do momentu zakończenia** |

Te same dwie własności pod tymi samymi nazwami wracają przy transakcjach — zob. [[SWN 09 Transakcje i atomowe zatwierdzanie]].

### Co jest atomowe w Javie
<sub>slajd 157</sub>

- Atomowość **gwarantowana** dla zapisu/odczytu danych **typu prostego**: `int`, `float` (i podobnych) oraz **referencji do obiektów**.
- **Brak atomowości** dla zapisu/odczytu danych typu **`long` i `double`**.
- Atomowość **gwarantowana** dla zapisu/odczytu danych typu prostego określonych jako **`volatile`**.
- **Brak atomowości** dla **pozornie atomowych** operacji typu `++`, `--`, `+=`, `-=`.

Dwa pierwsze punkty razem są niespodzianką dla większości programistów: `long` i `double` są 64-bitowe, więc na maszynie 32-bitowej ich zapis może się rozpaść na dwie połowy i inny wątek zobaczy **złożenie starej i nowej wartości**. Słowo `volatile` tę atomowość przywraca.

Ostatni punkt jest źródłem większości błędów współbieżności: `i++` to w rzeczywistości **odczyt, zwiększenie i zapis** — trzy operacje, między którymi może się wciąć inny wątek.

### `volatile`
<sub>slajd 158</sub>

- **Gwarancja**, że dowolny wątek czytający tak określoną zmienną **otrzyma ostatnią zapisaną w niej wartość**.
- **Nie oznacza wyłączności dostępu** do zmiennej (wzajemnego wykluczania).
- **Zapis danej ulotnej powoduje aktualizację wcześniej modyfikowanych przez dany wątek nieulotnych zmiennych.**

Trzeci punkt jest najmocniejszy i najczęściej pomijany: `volatile` działa jak **bariera pamięci** — zapis zmiennej ulotnej „wypycha" także wszystkie wcześniejsze zapisy tego wątku. Dzięki temu można opublikować cały, gotowy obiekt przez pojedynczy zapis jednej ulotnej referencji.

`volatile` rozwiązuje więc **wyłącznie przepływ danych**, nie przepływ sterowania — `i++` na zmiennej `volatile` nadal jest błędne.

---
## Pakiet `java.util.concurrent.atomic`
<sub>slajdy–merged.pdf, slajd 159</sub>

Umożliwia **atomową realizację złożonych operacji bez blokowania wątków**.

| Grupa | Typy |
|---|---|
| atomowe „typy proste" | `AtomicBoolean`, `AtomicInteger`, `AtomicLong`, `AtomicReference` |
| typy złożone z atomową realizacją operacji na poszczególnych elementach | `AtomicIntegerArray`, `AtomicLongArray`, `AtomicReferenceArray` |

### Operacje wspólne
<sub>slajd 160</sub>

| Operacja | Działanie |
|---|---|
| `get` / `set` | odczyt / ustawienie wartości |
| `getAndSet` | odczyt dotychczasowej wartości **i** ustawienie nowej |
| `compareAndSet` | ustawienie nowej wartości **pod warunkiem zgodności** dotychczasowej wartości z wartością parametru |
| `lazySet`, `weakCompareAndSet` | optymalizacje związane z realizacją pozostałych operacji w danym wątku — np. opóźnienie realizacji, **brak gwarancji _happens-before_** |

`compareAndSet` (CAS) jest operacją, na której opiera się cały pakiet: to bezpośrednie odwzorowanie sprzętowej instrukcji złożonej ze slajdu 155. Programowanie **bez blokowania** polega na powtarzaniu CAS w pętli aż do powodzenia.

### Operacje arytmetyczne
<sub>slajd 161</sub>

| Operacja | Działanie |
|---|---|
| `getAndAdd` / `addAndGet` | dodanie i zwrócenie wartości **sprzed** / **po** modyfikacji |
| `decrementAndGet`, `getAndDecrement`, `incrementAndGet`, `getAndIncrement` | modyfikacja o 1 i zwrócenie wartości sprzed lub po modyfikacji |

To właśnie atomowy odpowiednik `++` i `--` — naprawa problemu ze slajdu 157.

---
## Kolekcje współbieżne
<sub>slajdy–merged.pdf, slajdy 162–163</sub>

| Interfejs | Charakterystyka |
|---|---|
| `BlockingQueue<E>`, `BlockingDeque<E>` | kolejka FIFO z możliwością synchronizacji dostępu: **blokowanie przy próbie pobrania elementu z pustej kolejki** i **blokowanie przy próbie wstawienia elementu do pełnej kolejki** |
| `TransferQueue<E>` | kolejka **blokująca przy wstawianiu do momentu pobrania** elementu — blokowanie producenta do momentu pobrania elementu przez konsumenta |
| `Exchanger<V>` | **synchroniczna wymiana danych** pomiędzy **dwoma** wątkami |
| `ConcurrentMap<K,V>` | **atomowa realizacja** operacji zastępowania elementu, dodawania nowego elementu, usuwania elementu |
| `ConcurrentNavigableMap<K,V>` | `ConcurrentMap` obejmujący realizację operacji z interfejsu `NavigableMap` |

`BlockingQueue` jest gotowym rozwiązaniem **problemu ograniczonego buforowania** (producent-konsument) — tego samego, który w Adzie realizuje się typem chronionym z barierami (zob. [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione#Przykład — bufor cykliczny|NPR 05]]).

`TransferQueue` i `Exchanger` wprowadzają **komunikację synchroniczną** do świata Javy — to bliski odpowiednik **spotkania asymetrycznego** i **symetrycznego** z Ady.

---
## Mechanizmy synchronizacji — przegląd
<sub>slajdy–merged.pdf, slajd 164</sub>

- Klasa **`Semaphore`**
- Klasy **`CyclicBarrier`, `CountDownLatch` i `Phaser`**
- Pakiet **`java.util.concurrent.locks`**:

| Interfejsy | Rola |
|---|---|
| `Lock` | **zamek** do wzajemnego wykluczania |
| `Condition` | **zmienna warunkowa** do usypiania i budzenia — **tworzona przez obiekt typu `Lock`** |
| `ReadWriteLock` | blokady **współdzielone i wyłączne** |

| Klasy | Rola |
|---|---|
| `ReentrantLock`, `ReentrantReadWriteLock` (wraz z wewnętrznymi klasami `ReadLock` i `WriteLock`) | implementacje |
| `Abstract*Synchronizer`, `LockSupport` | do rozwijania **własnych** mechanizmów synchronizacji |

---
## `Semaphore`
<sub>slajdy–merged.pdf, slajd 165</sub>

| Grupa | Elementy |
|---|---|
| **parametry konstruktora** | `int permits` — liczba jednostek; `boolean fair` — **gwarancja FIFO** |
| **nabywanie jednostek** (opuszczanie) | `acquire`, `acquireUninterruptibly`, `tryAcquire` |
| **zwalnianie jednostek** (podnoszenie) | `release` |
| **likwidowanie jednostek** | `drainPermits`, `reducePermits` (`protected`) |

Terminologia „opuszczanie/podnoszenie" nawiązuje do klasycznych operacji **P** i **V** Dijkstry. Istotna różnica wobec `wait`/`notify`: semafor **pamięta** jednostki, więc `release` wykonane przed `acquire` **nie przepada** — to bezpośrednia odpowiedź na problem zgubionego sygnału (zob. [[NPR 06 Wątki w Javie i elementarna synchronizacja#Zgubiony sygnał|NPR 06]]).

---
## Bariery i zatrzaski
<sub>slajdy–merged.pdf, slajdy 166–168</sub>

### `CyclicBarrier`
<sub>slajd 166</sub>

| Grupa | Elementy |
|---|---|
| **parametry konstruktora** | `int parties` — liczba wątków do przełamania bariery; `Runnable barrierAction` — zadanie uruchamiane **po przełamaniu bariery (przed uwolnieniem wątków)** |
| **operacje synchronizujące** | `await`, `await(long timeout, TimeUnit unit)` — oczekiwanie na przełamanie bariery; `reset` (**uwaga na wyjątek `BrokenBarrierException`**) |
| **informacje o stanie** | `getNumberWaiting`, `getParties`, `isBroken` |

Bariera jest **cykliczna** — po przełamaniu wraca do stanu początkowego i może być użyta w kolejnej fazie obliczeń.

### `CountDownLatch`
<sub>slajd 167</sub>

| Grupa | Elementy |
|---|---|
| **parametr konstruktora** | `int count` — wartość początkowa, od której rozpoczyna się odliczanie |
| **operacje synchronizujące** | `await`, `await(long timeout, TimeUnit unit)` — oczekiwanie na osiągnięcie zera (**nie zmniejsza wartości licznika!**); `countDown` — zmniejszenie licznika o 1 |
| **informacje o stanie** | `getCount`, `toString` |

Różnica wobec `CyclicBarrier` jest zasadnicza i wynika wprost z uwagi w nawiasie: w barierze **czekanie i zgłoszenie się to ta sama operacja** (`await`), w zatrzasku są to **dwie różne operacje** (`countDown` i `await`), więc zgłaszający nie musi czekać, a czekający nie musi się zgłaszać. Zatrzask jest też **jednorazowy** — nie da się go zresetować.

### `Phaser`
<sub>slajd 168</sub>

- Mechanizm bariery umożliwiający **bardziej elastyczną kontrolę** przebiegu synchronizacji.
- **Rozdzielenie przejścia przez barierę od blokowania** w oczekiwaniu na przełamanie bariery: operacje typu **`arrive`** i operacje typu **`awaitAdvance`**.
- Możliwość **sterowania liczbą zadań** niezbędnych do przełamania bariery (do zakończenia fazy): operacje typu **`register`/`deregister`**.

`Phaser` uogólnia obie poprzednie klasy: rozdzielenie `arrive` od `awaitAdvance` daje zachowanie zatrzasku, a cykliczność faz — zachowanie bariery; dodatkowo liczba uczestników może się zmieniać w czasie, czego żadna z tamtych klas nie pozwala.

---
## `Lock` i `Condition`
<sub>slajdy–merged.pdf, slajdy 169–171</sub>

### `Lock`
<sub>slajd 169</sub>

| Metoda | Działanie |
|---|---|
| `lock`, `lockInterruptibly` | zajęcie zamka |
| `tryLock`, `tryLock(long time, TimeUnit unit)` | **nieblokująca** próba zajęcia zamka |
| `unlock` | zwolnienie zamka |
| `newCondition` | utworzenie **zmiennej warunkowej związanej z danym zamkiem** |

`newCondition` to brakujący element z [[NPR 06 Wątki w Javie i elementarna synchronizacja#Czy da się zbudować monitor?|NPR 06]]: jeden zamek może mieć **wiele** zmiennych warunkowych, więc czekający na różne warunki trafiają do **różnych kolejek**. Dopiero to pozwala zbudować pełnowartościowy monitor.

Pozostałe metody dają to, czego `synchronized` nie potrafi: **przerywalne** czekanie na zamek, **próbę z limitem czasu** i zajęcie zamka w jednej metodzie, a zwolnienie w innej.

### `Condition`
<sub>slajd 170</sub>

| Metoda | Działanie |
|---|---|
| `await`, `awaitUninterruptibly` | oczekiwanie na sygnał |
| `await(long time, TimeUnit unit)`, `awaitNanos`, `awaitUntil` | oczekiwanie **czasowe** na sygnał |
| `signal`, `signalAll` | wysłanie sygnału budzącego |

Odpowiedniki `wait`/`notify`/`notifyAll`, ale związane z konkretną zmienną warunkową, nie z obiektem.

### `ReadWriteLock`
<sub>slajd 171</sub>

| Metoda | Działanie |
|---|---|
| `Lock readLock()` | zwraca zamek do zakładania **blokady współdzielonej** |
| `Lock writeLock()` | zwraca zamek do zakładania **blokady wyłącznej** |

Jest to jawny odpowiednik pary blokad obiektu chronionego Ady — z tą różnicą, że w Adzie wybór blokady jest **automatyczny** (wynika z tego, czy wołana jest funkcja, czy procedura), a w Javie **programista wybiera go ręcznie**. Zob. [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione#Blokady|NPR 05]].

---
## Pule wątków
<sub>slajdy–merged.pdf, slajd 172</sub>

| Element | Rola |
|---|---|
| **Interfejs `Executor`** i pochodne | specyfikacja mechanizmu obsługi **puli wątków** |
| **Implementacja interfejsu `Executor`** | definicja klas do zarządzania pulą wątków |
| **Fabryka `Executors`** | wygodniejszy sposób tworzenia obiektów do zarządzania pulą wątków |
| **Interfejs `Callable`** | interfejs asynchronicznie realizowanego zadania, które **zwraca wynik lub wyjątek** |
| **Interfejs `Future`** | interfejs obiektu **reprezentującego wynik** zadania asynchronicznego |

### `Executor`
<sub>slajd 173</sub>

```java
void execute(Runnable command)
```

Sposób realizacji zleconych zadań **zależy od implementacji** interfejsu (np. `ThreadPoolExecutor`). **Brak możliwości jakiegokolwiek kontrolowania realizacji zleconych zadań** — stąd potrzeba `ExecutorService`.

### `ThreadPoolExecutor`
<sub>slajd 174</sub>

Obiekt zarządza pulą wątków: **utrzymuje wątki w gotowości** do realizacji zadań, **powiększa pulę** o dodatkowe wątki, gdy jest taka potrzeba, i **redukuje liczbę wątków**, gdy nie ma zadań do realizacji.

| Parametr konstruktora | Znaczenie |
|---|---|
| `int corePoolSize` | liczba wątków **utrzymywanych w puli** od momentu ich utworzenia |
| `int maximumPoolSize` | **górny limit** liczby wątków |
| `long keepAliveTime, TimeUnit unit` | czas utrzymywania **dodatkowych** wątków (powyżej `corePoolSize`) w stanie bezczynności |
| `BlockingQueue<Runnable> workQueue` | **kolejka zadań** do realizacji |

### Reguła przydziału zadań
<sub>slajd 175</sub>

![[npr-conc-s175-model-puli-watkow.png]]
<sub>slajdy–merged.pdf, slajd 175</sub>

1. Dopóki liczba współbieżnie realizowanych zadań **nie przekroczy `corePoolSize`**, każde zadanie otrzymuje **nowy wątek** i jest natychmiast realizowane.
2. Po osiągnięciu `corePoolSize` wykonywanych wątków kolejne zadania są **kolejkowane w `workQueue`**.
3. Po **przekroczeniu pojemności `workQueue`** uruchamiane są kolejne wątki.
4. Po osiągnięciu limitu `maximumPoolSize` kolejne zadania (zlecane przez `execute`) są **odrzucane** — wyjątek **`RejectedExecutionException`**.

Kolejność kroków 2 i 3 jest nieoczywista i warto ją zapamiętać: pula **najpierw kolejkuje, a dopiero potem tworzy dodatkowe wątki**. Konsekwencja praktyczna: przy nieograniczonej `workQueue` parametr `maximumPoolSize` **nigdy nie zostanie użyty**, bo kolejka nigdy się nie przepełni.

### Fabryka `Executors`
<sub>slajd 176</sub>

| Metoda | Rodzaj puli |
|---|---|
| `newSingleThreadExecutor` | pula 1-wątkowa |
| `newFixedThreadPool` | pula **ustalonej** wielkości |
| `newCachedThreadPool` | pula **zmiennej** wielkości |
| `newScheduledThreadPool` | pula zadań **uszeregowanych czasowo** |
| `newSingleThreadScheduledExecutor` | 1-wątkowa pula zadań uszeregowanych czasowo |

### `ExecutorService`
<sub>slajd 177</sub>

Pochodny od `Executor` (implementuje metodę `execute`); dostarcza mechanizmy umożliwiające **kontrolę realizacji zadań** i udostępniające **wyniki wykonania**.

Kontrola realizacji całości zbioru zadań:

| Metoda | Działanie |
|---|---|
| `shutdown` | **zakończenie zlecania** zadań |
| `shutdownNow` | **wymuszenie zakończenia** realizacji zadań |
| `isShutdown` | sprawdzenie, czy nastąpiło zlecenie operacji `shutdown` lub `shutdownNow` |
| `isTerminated` | sprawdzenie, czy **wszystkie zadania zostały wykonane** po operacji `shutdown` lub `shutdownNow` |
| `awaitTermination` | **oczekiwanie** na zakończenie zadań po operacji `shutdown` lub `shutdownNow` |

Rozróżnienie `shutdown` vs `shutdownNow`: pierwsza **przestaje przyjmować** nowe zadania i pozwala dokończyć bieżące, druga **przerywa** wykonywane.

---
## `Future` i `Callable`
<sub>slajdy–merged.pdf, slajdy 178–181</sub>

### `Future`
<sub>slajd 178</sub>

| Metoda | Działanie |
|---|---|
| `boolean cancel(boolean mayInterruptIfRunning)` | anulowanie wykonywania zadania |
| `boolean isCancelled()` | czy zadanie zostało anulowane **przed normalnym zakończeniem** |
| `boolean isDone()` | czy zadanie się zakończyło **w dowolny sposób**: normalnie, wyjątkiem, zostało anulowane |
| `V get()`, `get(long timeout, TimeUnit unit)` | **oczekiwanie na wynik** (na zakończenie) |

`Future` jest w RPC-owych kategoriach **uchwytem wywołania asynchronicznego**: `submit` odpowiada wysłaniu żądania bez czekania, a `get` — późniejszemu odebraniu wyniku. Por. [[NPR 01 Zdalne wywoływanie procedur (RPC)#Warianty użycia RPC|NPR 01]].

### `submit`
<sub>slajd 179</sub>

| Metoda | Co zwróci `get` |
|---|---|
| `Future<?> submit(Runnable task)` | **`null`** |
| `<T> Future<T> submit(Runnable task, T result)` | **przekazany parametr `result`** |
| `<T> Future<T> submit(Callable<T> task)` | **wynik zadania `task`**, zgodnie z definicją interfejsu `Callable` |

### `Callable`
<sub>slajd 180</sub>

Koncepcja zbliżona do interfejsu `Runnable`, ale w przeciwieństwie do niego operacja `call` **zwraca wynik i może sygnalizować wyjątek**:

```java
public interface Callable<V> {
   V call() throws Exception;
}
```

To dlatego `Runnable` nie wystarcza: jego `run()` ma typ `void` i nie deklaruje wyjątków, więc nie ma jak przekazać wyniku ani błędu z powrotem do zlecającego.

### Zlecanie zbiorów zadań
<sub>slajd 181</sub>

```java
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                              long timeout, TimeUnit unit)

<T> T invokeAny(Collection<? extends Callable<T>> tasks)
<T> T invokeAny(Collection<? extends Callable<T>> tasks,
                long timeout, TimeUnit unit)
```

`invokeAll` czeka na **wszystkie** i zwraca listę wyników; `invokeAny` zwraca wynik **pierwszego**, który się powiedzie.

### Zadania szeregowane czasowo
<sub>slajd 182</sub>

Zadanie **jednorazowe wykonywane z opóźnieniem**:

```java
<V> ScheduledFuture<V> schedule(Callable<V> callable,
                                long delay, TimeUnit unit);

ScheduledFuture<?> schedule(Runnable command,
                            long delay, TimeUnit unit);
```

Zadania **powtarzane okresowo**:

```java
ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay,
                                       long period, TimeUnit unit);

ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay,
                                          long delay, TimeUnit unit);
```

Różnica między obiema metodami okresowymi: `scheduleAtFixedRate` odmierza `period` **od początku poprzedniego wykonania**, `scheduleWithFixedDelay` — **od jego zakończenia**.

---
## Obiekty niezmienne
<sub>slajdy–merged.pdf, slajd 183</sub>

**Stan obiektu po jego skonstruowaniu nie zmienia się.** Strategia definiowania klasy dla obiektu niezmiennego:

1. **brak metod modyfikujących**,
2. wszystkie **zmienne instancji określone jako `final` i `private`**,
3. **zabezpieczenie przed redefinicją metod w podklasach** — deklaracja klasy jako `final`, prywatny konstruktor i tworzenie obiektów w fabrykach,
4. **zabezpieczenie przed zmianą obiektów modyfikowalnych**, do których istnieją referencje w danym obiekcie:
   - brak metod modyfikujących takie obiekty,
   - **brak powielania referencji** do takich obiektów — tworzenie **kopii** obiektów zamiast przekazywania/zwracania referencji.

Jest to jedyne w całym wykładzie podejście, które **eliminuje problem zamiast go rozwiązywać**: obiekt, którego stanu nie da się zmienić, nie wymaga żadnej synchronizacji przy współbieżnym dostępie — nie ma czego chronić. Punkt 4 jest przy tym najczęściej pomijany w praktyce: klasa z polem `final List<…>` **nie jest niezmienna**, jeśli udostępnia tę listę na zewnątrz.

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 153–183
> - **Model pamięci Javy** i relacja _happens-before_ — pada na slajdzie 160 w nawiasie (`weakCompareAndSet` — „brak gwarancji happen-before"), ale nigdzie nie jest zdefiniowana.
> - **Problem ABA** przy `compareAndSet` — podstawowa pułapka programowania bez blokowania, nieomówiona; brak też `AtomicStampedReference`.
> - **Polityki odrzucania zadań** (`RejectedExecutionHandler`: `AbortPolicy`, `CallerRunsPolicy`, …) — slajd 175 mówi tylko o wyjątku.
> - **`ForkJoinPool` i `CompletableFuture`** — nie występują, choć są dziś podstawowym sposobem realizacji zadań asynchronicznych w Javie.
> - **Przykłady kodu** — cały ten wykład jest wyliczeniem API bez ani jednego działającego fragmentu; poprzedni wykład ([[NPR 06 Wątki w Javie i elementarna synchronizacja|NPR 06]]) przykłady miał.
> - **Porównanie z mechanizmami Ady** — zestawienie znajduje się w [[NPR 09 Wielozadaniowość i synchronizacja zadań i wątków|skompresowane/NPR 09]].
