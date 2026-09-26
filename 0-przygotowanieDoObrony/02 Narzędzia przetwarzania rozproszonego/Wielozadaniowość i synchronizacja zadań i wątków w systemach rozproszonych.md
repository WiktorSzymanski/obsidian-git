---
tags:
  - obrona
  - synchronizacja
  - wielozadaniowość
up: "[[Mapa zagadnień]]"
zagadnienie: 9
---
# Wielozadaniowość i synchronizacja zadań/wątków w narzędziach i podejściach stosowanych w budowie systemów rozproszonych

## 1. Znaczenie zagadnienia

System rozproszony składa się z wielu współbieżnie działających jednostek: procesów, wątków, zadań, usług lub odbiorców komunikatów. Jednostki te mogą wykonywać się na jednym komputerze albo na wielu węzłach połączonych siecią. W obu przypadkach trzeba odpowiedzieć na dwa pytania:

1. **Kiedy** poszczególne jednostki mogą wykonać swoje operacje?
2. **Jak zapewnić**, że korzystając ze wspólnych danych albo wymieniając komunikaty, nie doprowadzą do niespójności?

Temu służą **wielozadaniowość** i **synchronizacja**.

**Wielozadaniowość** oznacza organizację wykonywania wielu jednostek przetwarzania. Jednostką może być proces, wątek albo zadanie. Jednostki mogą wykonywać się:

- **współbieżnie** — ich wykonanie przeplata się w czasie; jest to możliwe nawet na jednym procesorze,
- **równolegle** — kilka jednostek wykonuje instrukcje jednocześnie na różnych rdzeniach lub procesorach.

**Synchronizacja** oznacza koordynację ich działania. Nie ogranicza się tylko do wzajemnego wykluczania. Obejmuje również:

- ustalenie kolejności operacji,
- oczekiwanie na spełnienie warunku,
- przekazywanie danych,
- zapewnienie widoczności zmian,
- uzgadnianie faz obliczeń,
- obsługę zakończenia, anulowania i przekroczenia czasu oczekiwania.

W systemach rozproszonych synchronizacja lokalna jest tylko jednym poziomem problemu. Gdy jednostki znajdują się na różnych węzłach, wspólnej pamięci nie ma, a synchronizacja odbywa się przez komunikaty, kolejki, zdalne wywołania lub wspólną usługę koordynującą. Dochodzą wtedy opóźnienia, utrata komunikatów, awarie procesów i podział sieci.

## 2. Co właściwie synchronizujemy?

### 2.1. Synchronizacja przepływu sterowania

Synchronizacja przepływu sterowania określa, **w jakiej kolejności** mogą wykonać się operacje poszczególnych jednostek.

Przykłady:

- wątek może rozpocząć obliczenia dopiero po zakończeniu inicjalizacji przez inny wątek,
- konsument może pobrać element dopiero wtedy, gdy producent umieści go w buforze,
- wszystkie procesy muszą zakończyć jedną fazę, zanim rozpoczną następną,
- klient oczekuje na odpowiedź zdalną albo na potwierdzenie przyjęcia żądania.

Do tej grupy należą między innymi:

- `join`,
- zmienne warunkowe,
- semafory,
- bariery,
- zatrzaski,
- spotkania (*rendez-vous*),
- operacje `send`/`receive`,
- blokujące pobieranie z kolejki.

### 2.2. Synchronizacja danych

Synchronizacja danych określa, **kiedy i w jaki sposób** jednostki mogą czytać oraz modyfikować wspólny stan. Obejmuje:

- atomowość operacji,
- wzajemne wykluczanie,
- spójność danych,
- widoczność zapisów między wątkami,
- zachowanie relacji między zapisami i odczytami.

Do tej grupy należą:

- zamki i monitory,
- obiekty chronione,
- operacje atomowe,
- `volatile`,
- kolekcje współbieżne,
- niezmienność obiektów,
- komunikaty, które zastępują bezpośredni dostęp do wspólnej pamięci.

Pojedynczy mechanizm może obsługiwać oba wymiary. Na przykład `volatile` zapewnia widoczność danych, ale nie koordynuje złożonej operacji `odczyt–modyfikacja–zapis`. Bariera koordynuje przejście do kolejnej fazy, ale sama nie chroni dowolnych danych. Kolejka blokująca jednocześnie przechowuje dane i określa, kiedy producent lub konsument może kontynuować.

## 3. Podstawowe jednostki wykonania

### 3.1. Proces

**Proces** jest wykonywanym programem posiadającym własną przestrzeń adresową. Procesy są od siebie izolowane, dlatego bezpośrednie współdzielenie pamięci nie jest ich domyślnym sposobem komunikacji. Wymieniają dane za pomocą mechanizmów komunikacji międzyprocesowej (IPC, *Inter-Process Communication*), takich jak:

- potoki,
- kolejki komunikatów,
- pamięć współdzielona,
- gniazda,
- zdalne wywołania,
- pliki lub usługi pośredniczące.

W modelu MPI (Message Passing Interface, interfejs przekazywania komunikatów) jednostką wykonania jest zwykle proces identyfikowany przez **rangę** (*rank*). Procesy nie korzystają ze wspólnej pamięci, więc zależności muszą być wyrażone przez komunikaty.

### 3.2. Wątek

**Wątek** jest lekką jednostką wykonania wewnątrz procesu. Wątki tego samego procesu współdzielą między innymi:

- kod programu,
- stertę,
- zmienne globalne,
- otwarte pliki i inne zasoby procesu.

Każdy wątek ma jednak własny:

- stos,
- licznik rozkazów,
- zestaw rejestrów,
- stan wykonania.

Wspólna sterta ułatwia komunikację, ale powoduje ryzyko wyścigów. Wątek może modyfikować dane, które w tym samym czasie czyta lub modyfikuje inny wątek.

### 3.3. Zadanie

**Zadanie** jest jednostką pracy lub jednostką strukturalizacji programu współbieżnego. W zależności od narzędzia może oznaczać:

- pracę przekazaną do puli wątków,
- aktywny byt języka programowania,
- element grafu obliczeń,
- żądanie oczekujące w kolejce.

W Adzie zadanie (*task*) jest konstrukcją języka. W Javie zadanie jest zwykle reprezentowane przez `Runnable` albo `Callable`, a wykonanie zapewnia wątek bezpośredni lub pula wątków.

## 4. Poprawność współbieżnego programu

### 4.1. Atomowość

Operacja **atomowa** jest:

- **niepodzielna** — wykonuje się w całości albo nie wykonuje się wcale,
- **izolowana** — jej częściowe efekty nie są obserwowalne przez inne jednostki.

Przykładowo pojedynczy atomowy zapis nie gwarantuje, że bezpieczna jest sekwencja:

```text
odczytaj wartość
sprawdź warunek
zmień wartość
zapisz wartość
```

Jeżeli dwa wątki wykonują taką sekwencję jednocześnie, mogą odczytać tę samą wartość i utracić jeden z zapisów. Dlatego operacja `i++` nie jest równoważna jednej operacji atomowej: składa się z odczytu, zwiększenia i zapisu.

### 4.2. Widoczność i kolejność

Sam fakt zapisania wartości przez jeden wątek nie oznacza jeszcze, że drugi wątek natychmiast ją zobaczy. Kompilator, maszyna wirtualna i procesor mogą zmieniać kolejność operacji, o ile nie narusza to zasad modelu pamięci.

Relacja **happens-before** opisuje gwarantowany porządek między operacjami. Jeżeli operacja A zachodzi przed operacją B w sensie tej relacji, efekty A muszą być widoczne w B. Najważniejsze przykłady:

- wcześniejsze instrukcje tego samego wątku zachodzą przed późniejszymi instrukcjami tego wątku,
- zwolnienie zamka zachodzi przed późniejszym przejęciem tego samego zamka,
- zapis do `volatile` zachodzi przed późniejszym odczytem tej samej zmiennej,
- wywołanie `start()` zachodzi przed rozpoczęciem pracy uruchamianego wątku,
- zakończenie wątku zachodzi przed skutecznym `join()` na tym wątku.

> [!note] Z sieci
> Formalne wyjaśnienie relacji *happens-before* i modelu pamięci Javy nie jest omówione w dostarczonych prezentacjach. W praktyce jest istotne dla lokalnych elementów usług rozproszonych, takich jak cache, rejestr sesji, kolejka zadań lub stan połączenia.

### 4.3. Safety i liveness

Własności **safety** oznaczają, że nic złego nie nastąpi. Przykłady:

- dwóch wątków nie znajdzie się jednocześnie w sekcji krytycznej,
- bufor nie zostanie przepełniony ani opróżniony poniżej zera,
- komunikat nie zostanie przekazany do niewłaściwego odbiorcy.

Własności **liveness** oznaczają, że coś dobrego ostatecznie nastąpi. Przykłady:

- oczekujący wątek w końcu otrzyma zasób,
- klient w końcu otrzyma odpowiedź albo błąd,
- zadanie nie będzie pomijane bez końca,
- system nie zatrzyma się w oczekiwaniu cyklicznym.

Mechanizm może zapewniać safety, ale nie zapewniać liveness. Przykładowo zamek może chronić dane przed wyścigiem, a jednocześnie program może zakleszczyć się przez niewłaściwą kolejność przejmowania dwóch zamków.

## 5. Typowe problemy synchronizacji

### 5.1. Wyścig

**Wyścig** (*race condition*) występuje wtedy, gdy wynik zależy od przypadkowej kolejności przeplotu operacji współbieżnych. Szczególnym przypadkiem jest sytuacja, w której dwa wątki równocześnie aktualizują tę samą zmienną i jeden zapisuje wynik na podstawie nieaktualnego odczytu.

Wyścig może dotyczyć:

- wartości w pamięci,
- kolejności komunikatów,
- aktualizacji stanu po odebraniu odpowiedzi,
- wyboru lidera,
- obsługi tego samego żądania po retransmisji.

### 5.2. Zakleszczenie

**Zakleszczenie** (*deadlock*) to stan, w którym każda z jednostek czeka na zasób lub zdarzenie, które może zostać dostarczone tylko przez inną oczekującą jednostkę.

Przykład:

1. Wątek A zajmuje zamek `a` i czeka na zamek `b`.
2. Wątek B zajmuje zamek `b` i czeka na zamek `a`.
3. Żaden wątek nie może kontynuować ani zwolnić pierwszego zamka.

W systemie rozproszonym analogiem jest wzajemne oczekiwanie na komunikat lub odpowiedź. Dodatkowe ryzyko wynika z tego, że nie wiadomo, czy druga strona oblicza wynik, czy uległa awarii.

Sposoby ograniczania ryzyka:

- ustalenie globalnej kolejności przejmowania zasobów,
- unikanie oczekiwania na zdalną odpowiedź podczas trzymania lokalnego zamka,
- używanie limitów czasu i operacji `tryLock`,
- skracanie sekcji krytycznych,
- wykrywanie cykli oczekiwania,
- stosowanie komunikatów zamiast wspólnej pamięci.

### 5.3. Zagłodzenie i brak sprawiedliwości

**Zagłodzenie** (*starvation*) występuje, gdy jednostka jest gotowa do pracy, ale przez nieograniczony czas nie otrzymuje procesora, zasobu albo komunikatu. Mechanizm może zapewniać postęp całego systemu, a mimo to pojedynczy wątek może być stale pomijany.

**Sprawiedliwość** (*fairness*) oznacza, że oczekujące jednostki otrzymują możliwość postępu zgodnie z określoną polityką, często FIFO (*First In, First Out*, pierwszy wszedł, pierwszy wyszedł). Sprawiedliwość może zmniejszać przepustowość, ponieważ system nie zawsze obsługuje zadanie, które jest w danej chwili najtańsze.

### 5.4. Livelock

W **livelocku** jednostki nie są zablokowane formalnie, ale wykonują działania, które wzajemnie uniemożliwiają postęp. Przykładowo dwa wątki mogą naprzemiennie zwalniać i ponownie zajmować zasób, stale ustępując sobie nawzajem.

### 5.5. Odwrócenie priorytetów

**Odwrócenie priorytetów** występuje, gdy zadanie o wysokim priorytecie czeka na zasób trzymany przez zadanie o niskim priorytecie, a wykonywanie zadania o niskim priorytecie jest dodatkowo opóźniane przez zadanie o priorytecie średnim.

W systemach czasu rzeczywistego stosuje się protokoły dziedziczenia priorytetu oraz **Priority Ceiling Protocol (PCP, protokół pułapu priorytetu)**. Zasobowi przypisuje się pułap odpowiadający najwyższemu priorytetowi zadania, które może go używać. Ogranicza to czas blokowania zadania krytycznego i ułatwia analizę terminowości.

> [!note] Z sieci
> Priorytety i protokół pułapu nie są rozwinięte w dostarczonych prezentacjach dotyczących Ady, dlatego ten fragment jest uzupełnieniem spoza materiału podstawowego.

## 6. Poziomy mechanizmów synchronizacji

Mechanizmy można uporządkować według poziomu, na którym są realizowane.

### 6.1. Poziom sprzętowy

Procesor udostępnia atomowe operacje, na przykład:

- `test-and-set`,
- `exchange`,
- **CAS (Compare-And-Set, porównaj i ustaw)**.

CAS ustawia nową wartość tylko wtedy, gdy bieżąca wartość jest równa wartości oczekiwanej. Operacja jest podstawą wielu struktur bez blokad.

### 6.2. Poziom systemu operacyjnego

System operacyjny może usypiać i budzić wątki oraz integrować synchronizację z planowaniem procesora. Typowe mechanizmy to:

- mutexy,
- semafory,
- zmienne warunkowe,
- blokady czytelników i pisarzy,
- bariery,
- wątki lokalne i mechanizmy oczekiwania.

### 6.3. Poziom języka i biblioteki

Język lub biblioteka mogą opisać zależności wyżej niż pojedynczy zamek:

- monitory,
- obiekty chronione,
- zadania i wejścia,
- kolejki blokujące,
- pule wątków,
- futures,
- kolekcje współbieżne.

Ada umieszcza część tych mechanizmów bezpośrednio w języku. Java oferuje `synchronized` i `volatile` jako konstrukcje języka, a większość pozostałych mechanizmów w pakiecie `java.util.concurrent`.

## 7. Klasyczne mechanizmy

### 7.1. Mutex

**Mutex** (*mutual exclusion*, wzajemne wykluczanie) jest blokadą binarną. W danej chwili sekcję krytyczną może wykonywać tylko jeden właściciel.

Prawidłowy schemat:

```text
lock(mutex)
    sekcja krytyczna
unlock(mutex)
```

Zamek powinien być zwalniany także przy błędzie lub wyjątku. Nie wolno wykonywać długich operacji wejścia-wyjścia ani czekać na zdalną odpowiedź, trzymając lokalny mutex, chyba że jest to świadoma decyzja projektowa.

### 7.2. Semafor

**Semafor** jest licznikiem dostępnych jednostek zasobu. Operacja `P`/`down`/`acquire` zmniejsza licznik albo usypia wywołującego, gdy nie ma jednostek. Operacja `V`/`up`/`release` zwiększa licznik i może obudzić oczekującego.

Semafor może reprezentować:

- jeden zasób — semafor binarny,
- pulę `N` zasobów,
- liczbę wolnych miejsc w buforze,
- ograniczenie liczby równoczesnych żądań.

Ważna różnica wobec zmiennej warunkowej: semafor **pamięta** zwolnione jednostki. `release` wykonane przed `acquire` nie przepada.

### 7.3. Zmienna warunkowa

Zmienna warunkowa służy do oczekiwania na stan, który może zmienić inny wątek. Zawsze jest związana z mutexem.

Schemat:

```text
lock(mutex)
while warunek nie jest spełniony:
    wait(condition, mutex)
    # wait atomowo zwalnia mutex i usypia;
    # po obudzeniu ponownie zajmuje mutex
wykonaj operację
unlock(mutex)
```

Warunek musi być sprawdzany w pętli `while`, a nie w `if`, ponieważ:

- może wystąpić fałszywe przebudzenie (*spurious wakeup*),
- po obudzeniu inny wątek mógł już wykorzystać zasób,
- `notifyAll` budzi także wątki oczekujące na inne warunki.

### 7.4. Monitor

**Monitor** łączy:

- hermetyzację danych,
- automatyczne wzajemne wykluczanie,
- zmienne warunkowe,
- operacje udostępniane innym wątkom.

Dobrze zaprojektowany monitor ukrywa dane i pozwala czekać na warunek bez ujawniania sposobu blokowania.

### 7.5. Bariera

**Bariera** dzieli obliczenia na fazy. Wątek, który do niej dotrze, zatrzymuje się, dopóki nie dotrze wymagana liczba uczestników. Dopiero wtedy wszyscy mogą przejść do następnej fazy.

Bariera pasuje do algorytmów iteracyjnych, w których każda iteracja korzysta z wyników poprzedniej. Nie jest właściwym narzędziem do ochrony pojedynczej zmiennej.

### 7.6. Czytelnicy i pisarze

Blokada czytelników i pisarzy pozwala wielu czytelnikom korzystać ze stanu jednocześnie, ale wyklucza pisarza z każdym innym dostępem. Można stosować:

- preferencję czytelników — dobra przepustowość, możliwe zagłodzenie pisarza,
- preferencję pisarzy — ogranicza zagłodzenie pisarzy, może opóźniać czytelników,
- politykę sprawiedliwą — kompromis między przepustowością i przewidywalnością.

### 7.7. Lock-free i wait-free

Struktura:

- **blocking** może zatrzymać inne wątki przez zamek,
- **lock-free** gwarantuje, że system jako całość robi postęp,
- **wait-free** gwarantuje, że każdy wątek kończy operację w skończonej liczbie własnych kroków,
- **obstruction-free** gwarantuje postęp przy braku konkurencji.

Lock-free nie oznacza automatycznie sprawiedliwości. Jeden wątek może stale przegrywać operacje CAS. Takie rozwiązania są trudniejsze do zweryfikowania i mogą wymagać obsługi problemu ABA.

**Problem ABA (A-B-A)** występuje, gdy wątek T1 odczyta wartość A, wątek T2 zmieni ją na B, a następnie ponownie na A. T1 wykonuje CAS i widzi oczekiwane A, chociaż stan pośredni miał znaczenie. Stosuje się między innymi wersjonowanie wartości, `AtomicStampedReference`, znaczniki oraz bezpieczne zarządzanie pamięcią.

## 8. Java: wątki i synchronizacja

### 8.1. Tworzenie i cykl życia wątku

Wątek jest reprezentowany przez obiekt `Thread`. Program wątku znajduje się w `run()`. Można:

1. odziedziczyć po `Thread` i przesłonić `run()`,
2. zaimplementować `Runnable` i przekazać obiekt do `Thread`.

Drugi wariant jest zwykle elastyczniejszy, ponieważ klasa może już dziedziczyć po innej klasie.

Podstawowe operacje:

- `start()` — rozpoczęcie wykonywania nowego wątku,
- `run()` — kod wykonywany przez wątek; bezpośrednie wywołanie `run()` nie tworzy nowego wątku,
- `join()` — oczekiwanie na zakończenie wątku,
- `sleep()` — uśpienie bieżącego wątku,
- `yield()` — dobrowolne oddanie procesora,
- `interrupt()` — zgłoszenie prośby o przerwanie.

`interrupt()` nie zabija wątku. Może przerwać `sleep`, `join` lub `wait` przez `InterruptedException`, a poza stanem oczekiwania ustawia flagę przerwania. Wątek powinien współpracować, okresowo sprawdzając tę flagę lub poprawnie obsługując wyjątek.

`stop()` i `suspend()` są niebezpieczne. Zawieszenie wątku w sekcji krytycznej może pozostawić zamek zajęty na czas nieokreślony.

**Demon** jest wątkiem, którego działanie nie utrzymuje procesu przy życiu po zakończeniu wszystkich wątków użytkownika. Nie jest to bezpieczny mechanizm finalizacji zasobów, ponieważ demon może zostać zakończony w dowolnym momencie.

### 8.2. `synchronized`

Blok:

```java
synchronized (obiekt) {
    // sekcja krytyczna
}
```

zajmuje zamek związany z `obiekt`. Metoda `synchronized` zajmuje zamek obiektu, na którym została wywołana.

Zamek wbudowany w obiekt jest **reentrant**, czyli wielowejściowy. Ten sam wątek może wejść ponownie do sekcji chronionej tym samym zamkiem. Zamek zostanie zwolniony dopiero po wyjściu z odpowiadającej liczby zagnieżdżonych sekcji.

Reentrant nie oznacza odporności na zakleszczenie dwóch wątków. Nadal możliwy jest cykl:

```text
wątek A: lock(a), czeka na b
wątek B: lock(b), czeka na a
```

### 8.3. `wait`, `notify`, `notifyAll`

Metody muszą być wywoływane podczas posiadania zamka tego samego obiektu:

- `wait()` zwalnia zamek i usypia wątek,
- `notify()` budzi jeden oczekujący wątek,
- `notifyAll()` budzi wszystkie oczekujące wątki.

`notify` nie jest pamiętanym zdarzeniem. Jeżeli w chwili wywołania nikt nie czeka, sygnał przepada. Dlatego poprawny kod opiera się na stanie i pętli:

```java
synchronized (monitor) {
    while (!warunek) {
        monitor.wait();
    }
    // warunek jest spełniony, zamek jest zajęty
}
```

Niskopoziomowe `wait`/`notify` dają jedną kolejkę oczekujących na obiekt. Jeżeli różne grupy czekają na różne warunki, zwykle trzeba użyć `notifyAll()` i ponownego sprawdzenia warunku. Para `Lock`/`Condition` pozwala utworzyć wiele nazwanych kolejek warunkowych.

### 8.4. Widoczność i atomowość w Javie

`volatile` zapewnia widoczność zapisu dla innych wątków i ustanawia odpowiednią relację pamięci. Nie zapewnia jednak wzajemnego wykluczania. Nadal błędne jest:

```java
volatile int counter;
counter++;
```

`counter++` pozostaje sekwencją odczytu, modyfikacji i zapisu.

Operacje odczytu i zapisu większości typów prostych oraz referencji są atomowe. Historycznie problem dotyczył nieulotnych wartości `long` i `double`; deklaracja `volatile` zapewnia ich atomowy dostęp. Niezależnie od tego złożone operacje wymagają zamka albo klasy atomowej.

Pakiet `java.util.concurrent.atomic` udostępnia między innymi:

- `AtomicBoolean`,
- `AtomicInteger`,
- `AtomicLong`,
- `AtomicReference`,
- tablice atomowe.

Najważniejsza operacja `compareAndSet` aktualizuje wartość tylko wtedy, gdy nadal jest równa wartości oczekiwanej.

### 8.5. `Lock`, `Condition` i `ReadWriteLock`

`Lock` daje większą kontrolę niż `synchronized`:

- `lock()` i `unlock()`,
- `lockInterruptibly()` — oczekiwanie przerywalne,
- `tryLock()` — próba bez blokowania,
- `tryLock` z limitem czasu,
- `newCondition()` — utworzenie zmiennej warunkowej.

`Condition` ma odpowiedniki `wait`/`notify`:

- `await()`,
- `signal()`,
- `signalAll()`.

Ważne: `Condition` jest związana z konkretnym zamkiem i także wymaga posiadania tego zamka przed oczekiwaniem lub sygnalizowaniem.

`ReadWriteLock` udostępnia osobno:

- współdzielony `readLock`,
- wyłączny `writeLock`.

Jest odpowiednikiem rozróżnienia funkcji, procedur i wejść w obiekcie chronionym Ady, ale w Javie programista wybiera blokadę jawnie.

### 8.6. Semafory, bariery i kolejki

`Semaphore` przechowuje liczbę pozwoleń. Parametr `fair` może włączyć politykę FIFO.

`CyclicBarrier`:

- czeka na ustaloną liczbę uczestników,
- po przełamaniu może wykonać `barrierAction`,
- może być używana ponownie w kolejnych fazach.

`CountDownLatch`:

- posiada licznik malejący przez `countDown()`,
- inne wątki czekają przez `await()`,
- jest jednorazowy,
- wątek wywołujący `countDown()` nie musi czekać.

`Phaser`:

- obsługuje wiele faz,
- rozdziela zgłoszenie przybycia od oczekiwania,
- pozwala dynamicznie rejestrować i wyrejestrowywać uczestników.

`BlockingQueue` jest gotową implementacją ograniczonego bufora producent–konsument:

- pobranie z pustej kolejki może blokować,
- wstawienie do pełnej kolejki może blokować.

`TransferQueue` może blokować producenta aż do odebrania elementu przez konsumenta. `Exchanger` realizuje synchroniczną wymianę danych między dwiema stronami.

### 8.7. Pule wątków i asynchroniczne zadania

Pula wątków oddziela opis zadania od decyzji, który wątek je wykona. `Executor` przyjmuje `Runnable`, a `ExecutorService` dodatkowo pozwala zarządzać cyklem życia puli.

`ThreadPoolExecutor` używa między innymi:

- `corePoolSize`,
- `maximumPoolSize`,
- `keepAliveTime`,
- `workQueue`.

Reguła przydziału zadań jest ważna:

1. do osiągnięcia `corePoolSize` tworzone są nowe wątki,
2. potem zadania trafiają najpierw do `workQueue`,
3. dopiero po przepełnieniu kolejki tworzone są wątki ponad rozmiar podstawowy,
4. po osiągnięciu `maximumPoolSize` kolejne zadania mogą zostać odrzucone.

`ExecutorService`:

- `shutdown()` przestaje przyjmować nowe zadania i pozwala zakończyć bieżące,
- `shutdownNow()` próbuje przerwać wykonywanie,
- `awaitTermination()` oczekuje na zakończenie.

`Callable<V>` zwraca wynik i może zgłosić wyjątek. `Future<V>` jest uchwytem do wyniku:

- `get()` czeka na wynik,
- `get` z limitem czasu ogranicza oczekiwanie,
- `cancel()` próbuje anulować zadanie,
- `isDone()` i `isCancelled()` informują o stanie.

W systemie rozproszonym `Future` przypomina lokalny uchwyt do asynchronicznego wywołania. Nie rozwiązuje jednak sam problemów zdalnej awarii, retransmisji ani duplikatów.

### 8.8. Niezmienność

**Niezmienny obiekt** nie zmienia stanu po utworzeniu. Jeżeli nie udostępnia modyfikowalnego stanu, może być bezpiecznie współdzielony bez blokad.

Typowe zasady:

- prywatne pola `final`,
- brak metod modyfikujących,
- klasa `final` lub inne zabezpieczenie przed podklasowaniem,
- kopie defensywne dla modyfikowalnych obiektów,
- brak zwracania wewnętrznych referencji do modyfikowalnych danych.

Niezmienność jest często prostsza niż synchronizacja. Zamiast chronić zmieniany obiekt, tworzy się nowy stan i publikuje go atomowo.

## 9. Ada 95: zadania, spotkania i obiekty chronione

### 9.1. Zadania

W Adzie zadanie (*task*) jest konstrukcją języka, a nie zwykłą klasą biblioteczną. Zadeklarowane zadanie startuje automatycznie. Można utworzyć wiele zadań tego samego typu, na przykład tablicę zadań.

Specyfikacja zadania określa jego wejścia:

```ada
task type Worker is
   entry Start (Id : in Integer);
   entry Stop;
end Worker;
```

Treść zadania implementuje obsługę wejść przez `accept`.

### 9.2. Spotkanie asymetryczne

**Spotkanie** (*rendez-vous*) jest synchroniczną komunikacją między dwoma zadaniami:

- zadanie-klient wywołuje wejście serwera,
- zadanie-serwer przyjmuje je przez `accept`,
- strona, która dotrze pierwsza, czeka na drugą,
- ciało `accept` wykonuje się, gdy obie strony są gotowe.

Komunikacja i synchronizacja są jednym zdarzeniem. Klient musi znać nazwę zadania i wejścia, natomiast serwer nie musi znać tożsamości klienta.

Spotkanie jest odpowiednikiem synchronicznego wywołania, ale z jawną konstrukcją oczekiwania po stronie serwera.

### 9.3. `select`

Instrukcja `select` ma cztery zastosowania:

1. **oczekiwanie selektywne** po stronie serwera,
2. **terminowe wywołanie wejścia** po stronie klienta,
3. **warunkowe wywołanie wejścia** po stronie klienta,
4. **asynchroniczna zmiana wątku sterowania** przez `then abort`.

Oczekiwanie selektywne pozwala wybierać między wieloma wejściami:

```ada
select
   accept E1 (...) do
      ...
   end E1;
or
   accept E2 (...) do
      ...
   end E2;
end select;
```

Gałąź może mieć **dozór** (*guard*), czyli warunek logiczny. Dozory są obliczane na początku wykonania `select`; jeżeli stan zmieni się podczas oczekiwania, nowa wartość będzie uwzględniona dopiero przy następnym wykonaniu instrukcji.

Pozostałe gałęzie:

- `delay` ogranicza czas oczekiwania,
- `else` nie czeka i wykonuje się tylko wtedy, gdy nie ma natychmiastowego spotkania,
- `terminate` pozwala zakończyć zadanie, gdy nie ma już możliwych klientów.

`select ... then abort` pozwala przerwać trwające obliczenie po wystąpieniu zdarzenia wyzwalającego, na przykład po upływie czasu. Jest to silniejszy mechanizm niż `interrupt()` w Javie, ponieważ nie wymaga współpracy przerywanego obliczenia. Przerwanie obliczenia w dowolnym miejscu wymaga jednak ostrożności, zwłaszcza gdy obliczenie posiada zasoby lub wykonuje operacje, które muszą zostać dokończone.

### 9.4. Obiekty chronione

**Obiekt chroniony** jest adowym odpowiednikiem monitora. Ukrywa współdzielone dane i udostępnia operacje:

- **funkcje** — tylko odczyt,
- **procedury** — modyfikacja bez warunku,
- **wejścia** — modyfikacja po spełnieniu bariery.

Komunikacja przez obiekt chroniony jest asynchroniczna: zadanie może zgłosić żądanie, a obiekt wykona je, gdy warunki będą spełnione.

Obiekt ma automatyczne blokady:

- funkcje mogą być wykonywane współbieżnie jako odczyty,
- procedura lub wejście wymaga wyłącznego dostępu,
- wywołanie wejścia, którego bariera jest fałszywa, trafia do kolejki i nie blokuje dostępu innym zadaniom.

### 9.5. Bariery i bufor ograniczony

Bufor ograniczony ma dwa warunki:

- producent może wstawić element, gdy `count < capacity`,
- konsument może pobrać element, gdy `count > 0`.

W Adzie zapisuje się te warunki bezpośrednio przy wejściach:

```ada
entry Put (X : in Item) when Count < Capacity;
entry Get (X : out Item) when Count > 0;
```

Zadanie czekające na barierę nie trzyma blokady obiektu. Dzięki temu inne zadanie może wejść, zmienić licznik i otworzyć barierę. Nie ma potrzeby ręcznego `wait`, `notify` ani pętli chroniącej przed zgubionym sygnałem.

### 9.6. `requeue`

`requeue` przekazuje wywołanie do kolejki innego wejścia. Jest przydatne, gdy pełny warunek dopuszczenia zależy od parametrów wywołania, których nie można uwzględnić w prostej barierze. Z perspektywy klienta nadal trwa jedno wywołanie wejścia.

### 9.7. Java i Ada — porównanie

| Właściwość | Java | Ada 95 |
|---|---|---|
| Jednostka wykonania | `Thread`, `Runnable`, `Callable` | zadanie `task` |
| Uruchomienie | jawne `start()` | automatyczne przy deklaracji |
| Wzajemne wykluczanie | `synchronized`, `Lock` | obiekt chroniony |
| Oczekiwanie warunkowe | `wait`/`notify` albo `Condition` | bariery wejść |
| Odczyt i zapis | programista wybiera blokadę | funkcja/procedura/entry określają tryb |
| Komunikacja synchroniczna | brak wbudowanego rendez-vous; `Exchanger` jest przybliżeniem | `entry`/`accept` |
| Oczekiwanie selektywne | brak bezpośredniego odpowiednika | `select` |
| Pula wątków | `ExecutorService` i `Future` | brak analogicznego mechanizmu w materiale |
| Główne ryzyko | zgubiony sygnał, zła widoczność, zła blokada | błędny dobór wejść, dozorów i zakończenia |

Ada przenosi więcej warunków poprawności do konstrukcji języka. Java jest bardziej ogólna, ale wymaga jawnego połączenia atomowości, widoczności, blokad i oczekiwania.

## 10. Synchronizacja przez komunikację w systemach rozproszonych

Gdy jednostki znajdują się w różnych procesach lub węzłach, nie powinny synchronizować się przez bezpośredni dostęp do pamięci. Synchronizacja staje się częścią protokołu komunikacyjnego.

### 10.1. RPC

**RPC (Remote Procedure Call, zdalne wywoływanie procedur)** ukrywa komunikację sieciową pod składnią wywołania procedury. W podstawowym wariancie:

1. klient wysyła żądanie,
2. serwer wykonuje procedurę,
3. serwer odsyła wynik,
4. klient czeka na odpowiedź.

Jest to synchronizacja przepływu sterowania: klient pozostaje zablokowany do czasu odpowiedzi albo błędu.

RPC może jednak mieć wariant:

- wywołania asynchronicznego, w którym klient nie czeka na wynik,
- wywołania asynchronicznego z potwierdzeniem odbioru,
- wywołania zwrotnego (*callback*), w którym serwer później wywołuje procedurę udostępnioną przez klienta.

W przypadku utraty żądania lub odpowiedzi pojawia się problem retransmisji. Zaginiona odpowiedź może wyglądać tak samo jak zaginione żądanie, więc powtórzenie może uruchomić operację drugi raz. Dlatego:

- operacje powinny być idempotentne, gdy jest to możliwe,
- żądania można numerować,
- serwer może pamiętać wynik obsłużonego żądania,
- należy ustalić semantykę wykonania: co najmniej raz, co najwyżej raz albo brak gwarancji.

**Dokładnie raz** (*exactly-once*) nie jest bezwarunkowo osiągalne w obecności awarii serwera lub sieci. Synchronizacja zdalna musi więc obejmować również timeouty, retry i obsługę duplikatów.

### 10.2. Warstwa CHAN w RPC

W opisie RPC warstwa **CHAN (Channel, kanał)** synchronizuje żądania z odpowiedziami i może wykrywać, czy druga strona nadal działa. Warianty obejmują:

- jawne potwierdzenia,
- potwierdzenia domniemane,
- próbkowanie serwera przez `PING`/`PONG`,
- bicie serca wysyłane przez serwer.

W długich obliczeniach samo czekanie na odpowiedź nie rozróżnia:

- wolnego serwera,
- awarii serwera,
- awarii klienta,
- podziału sieci.

Dlatego timeout i heartbeat są elementami synchronizacji, a nie tylko optymalizacji.

### 10.3. MOM i kolejki komunikatów

**MOM (Message-Oriented Middleware, oprogramowanie pośredniczące zorientowane na komunikaty)** wprowadza pośrednika, zwykle brokera i kolejkę. Nadawca i odbiorca są rozdzieleni:

- **w przestrzeni** — nie muszą znać swoich adresów,
- **w czasie** — nie muszą działać jednocześnie.

W modelu punkt-punkt:

- producent umieszcza komunikat w kolejce,
- jeden konsument go pobiera,
- komunikat może czekać, gdy konsument jest nieaktywny.

W modelu publish/subscribe:

- producent publikuje komunikat na temat,
- wielu subskrybentów może otrzymać własną kopię,
- zwykła subskrypcja obejmuje komunikaty publikowane po zapisaniu się.

Trwała komunikacja pozwala synchronizować procesy mimo ich czasowej niedostępności. Cena to konieczność określenia:

- trwałości komunikatów,
- potwierdzeń,
- semantyki redelivery,
- kolejności,
- czasu życia,
- obsługi duplikatów.

### 10.4. JMS

**JMS (Java Message Service, usługa komunikatów Javy)** jest specyfikacją interfejsu dla systemów komunikatów, a nie pojedynczym brokerem.

Najważniejsze elementy:

- `ConnectionFactory` tworzy połączenie,
- `Connection` reprezentuje logiczne połączenie z dostawcą,
- `Session` jest jednowątkowym kontekstem tworzenia i odbioru wiadomości,
- `Destination` oznacza kolejkę albo temat,
- producent wysyła wiadomości,
- konsument odbiera je synchronicznie przez `receive()` albo asynchronicznie przez `MessageListener`.

JMS wspiera:

- komunikację punkt-punkt i publish/subscribe,
- potwierdzenia,
- komunikaty trwałe,
- priorytet,
- czas życia,
- trwałe subskrypcje,
- transakcje sesji.

W sesji transakcyjnej `commit()` zatwierdza wysłanie wyprodukowanych wiadomości i potwierdza odebrane wiadomości. `rollback()` usuwa wysyłane wiadomości i powoduje ponowne dostarczenie odebranych. Nie należy w jednej transakcji wysyłać komunikatu i czekać na odpowiedź, która zostanie wysłana dopiero po `commit()` — prowadziłoby to do zakleszczenia protokołu.

### 10.5. ZeroMQ

**ZeroMQ** jest biblioteką komunikacyjną z wbudowanym kolejkowaniem w gniazdach. W przeciwieństwie do klasycznego MOM nie wymaga centralnego brokera.

Podstawowe wzorce:

- `REQ-REP` — naprzemienne żądanie i odpowiedź,
- `PUB-SUB` — publikowanie do subskrybentów,
- `PUSH-PULL` — potokowe rozdzielanie pracy.

Brak brokera oznacza mniejszy narzut i prostszą topologię, ale kolejka jest związana z procesem. Zakończenie procesu może oznaczać utratę niezapisanych komunikatów. ZeroMQ nie zapewnia więc automatycznie takiej trwałości jak broker z trwałą kolejką.

Ważna zasada z materiałów uzupełniających: gniazd ZeroMQ nie należy współdzielić między wątkami. Wątki mogą komunikować się przez `inproc://`, czyli lokalny kanał komunikatów. Zamiast współdzielenia struktur i ręcznego blokowania każdy wątek posiada własny kontekst pracy, a synchronizacja odbywa się przez przesyłane komunikaty.

### 10.6. Przestrzeń krotek — Linda i JavaSpaces

**Linda** organizuje współdzieloną przestrzeń krotek. Komunikat nie jest pobierany po nazwie kolejki, lecz przez dopasowanie treści do wzorca.

Operacje:

- `Output` — umieszczenie krotki,
- `Input` — pobranie i usunięcie dopasowanej krotki,
- `Read` — odczyt bez usuwania,
- `Try_Input` i `Try_Read` — warianty nieblokujące.

Jeżeli pasującej krotki jeszcze nie ma, `Input` lub `Read` mogą blokować. W ten sposób przestrzeń krotek staje się jednocześnie:

- mechanizmem komunikacji,
- mechanizmem synchronizacji,
- miejscem koordynowania zadań.

Nie ma gwarancji FIFO. Jeżeli kilka krotek pasuje do wzorca, może zostać wybrana dowolna. Kolejność trzeba zakodować w atrybutach, jeśli jest wymagana.

W JavaSpaces krotka jest obiektem `Entry`, a `write`, `take`, `read`, `takeIfExists` i `readIfExists` odpowiadają operacjom Lindy. Dzierżawa (*lease*) ogranicza czas życia krotki, która mogłaby pozostać w przestrzeni po awarii producenta.

## 11. Uzupełnienie: MPI i OpenMP

> [!note] Z sieci
> Dostarczone prezentacje wymieniają MPI i OpenMP jako zakres zagadnienia, ale nie omawiają ich szczegółowo. Poniższy fragment uzupełnia ten brak na podstawie oficjalnej dokumentacji Open MPI i specyfikacji OpenMP.

### 11.1. MPI

**MPI (Message Passing Interface, interfejs przekazywania komunikatów)** jest standardem komunikacji między procesami. Każdy proces ma własną pamięć, a synchronizacja odbywa się przez jawne operacje komunikacyjne.

Przykładowe operacje:

- `MPI_Send` i `MPI_Recv` — blokujące wysłanie i odbiór,
- `MPI_Isend` i `MPI_Irecv` — rozpoczęcie operacji nieblokującej,
- `MPI_Wait` lub `MPI_Test` — sprawdzenie albo oczekiwanie na zakończenie,
- `MPI_Barrier` — bariera dla wszystkich procesów w komunikatorze.

Operacje nieblokujące pozwalają nakładać komunikację na obliczenia, ale bufor używany przez `MPI_Isend` lub `MPI_Irecv` nie może być zmieniany przed zakończeniem operacji. `MPI_Barrier` synchronizuje wejście do kolejnej fazy, lecz nie powinien być dodawany bez potrzeby, ponieważ wymusza oczekiwanie także na procesy gotowe wcześniej.

### 11.2. OpenMP

**OpenMP (Open Multi-Processing)** rozszerza programy wykorzystujące pamięć współdzieloną o dyrektywy równoległości.

Najważniejsze konstrukcje:

- `parallel` — utworzenie zespołu wątków,
- `for` — podział iteracji pętli,
- `sections` — podział niezależnych sekcji pracy,
- `task` — utworzenie zadania wykonywanego przez środowisko,
- `critical` — wzajemne wykluczanie nazwanej sekcji,
- `atomic` — atomowa aktualizacja pojedynczej operacji,
- `barrier` — oczekiwanie wszystkich wątków zespołu,
- `taskwait` — oczekiwanie na zadania potomne,
- `reduction` — bezpieczne połączenie wyników lokalnych.

MPI i OpenMP mogą być łączone w programie hybrydowym: MPI rozdziela pracę między procesy lub węzły, a OpenMP tworzy wątki wewnątrz każdego procesu.

Źródła uzupełnienia:

- [Open MPI — `MPI_Barrier`](https://docs.open-mpi.org/en/v5.0.x/man-openmpi/man3/MPI_Barrier.3.html)
- [Open MPI — `MPI_Isend`](https://docs.open-mpi.org/en/main/man-openmpi/man3/MPI_Isend.3.html)
- [OpenMP Specifications](https://www.openmp.org/specifications/)

## 12. Wzorzec producent–konsument

Problem producenta–konsumenta jest wspólnym przykładem dla wątków, zadań i komunikatów.

Mamy ograniczony bufor o pojemności `N`:

- producent dodaje elementy,
- konsument pobiera elementy,
- producent nie może dodać elementu do pełnego bufora,
- konsument nie może pobrać elementu z pustego bufora.

Warunki bezpieczeństwa:

```text
0 <= liczba_elementów <= N
```

Rozwiązania:

- mutex + zmienne warunkowe `notFull` i `notEmpty`,
- semafory `empty` i `full`,
- `BlockingQueue` w Javie,
- wejścia z barierami w Adzie,
- kolejka MOM,
- kanał `PUSH-PULL` w ZeroMQ,
- przestrzeń krotek z blokującym `Input`.

Najważniejsza różnica polega na zakresie:

- mutex chroni lokalne dane,
- `BlockingQueue` łączy ochronę i przekazywanie danych w jednym procesie,
- kolejka MOM lub ZeroMQ przenosi synchronizację między procesami,
- przestrzeń krotek wybiera komunikat na podstawie wzorca, a nie kolejności.

## 13. Jak dobierać mechanizm?

| Problem | Właściwy mechanizm |
|---|---|
| Jedna krótka sekcja krytyczna w pamięci współdzielonej | mutex, `synchronized`, `Lock` |
| Ograniczona liczba zasobów | semafor |
| Czekanie na stan danych | zmienna warunkowa + `while`, `Condition`, bariera wejścia |
| Wielu czytelników i pojedynczy pisarz | `ReadWriteLock`, obiekt chroniony |
| Oczekiwanie na zakończenie jednej fazy | `CountDownLatch` lub `join` |
| Powtarzające się fazy ze stałą liczbą uczestników | `CyclicBarrier` |
| Fazy z dynamiczną liczbą uczestników | `Phaser` |
| Przekazywanie pracy w jednym procesie | `BlockingQueue` i pula wątków |
| Wymiana danych między dwoma wątkami | `Exchanger` lub rendez-vous |
| Komunikacja między procesami bez wspólnej pamięci | komunikaty, MPI, ZeroMQ |
| Trwałe rozdzielenie producenta i konsumenta w czasie | MOM/JMS |
| Wybór komunikatu po treści | Linda/JavaSpaces |
| Szybka komunikacja RPC | synchroniczne RPC, a dla braku blokowania — RPC asynchroniczne |
| Ograniczenie wpływu awarii i powtórzeń | timeout, retry, idempotentność, numerowanie żądań |

## 14. Najważniejsze zasady projektowe

1. Najpierw rozdziel synchronizację danych od synchronizacji przepływu sterowania. Jeden mechanizm rzadko rozwiązuje oba problemy poprawnie.
2. Zdefiniuj inwariant, który ma pozostać prawdziwy, na przykład `0 <= count <= N`.
3. Chroń cały złożony warunek i jego modyfikację jednym mechanizmem. Sam `volatile` nie chroni sekwencji `odczyt–sprawdzenie–zapis`.
4. Przy zmiennych warunkowych sprawdzaj warunek w `while`, nie w `if`.
5. Nie trzymaj lokalnego zamka podczas oczekiwania na zdalną odpowiedź, jeżeli może to utworzyć cykl zależności.
6. Ustal kolejność przejmowania wielu zasobów.
7. Ograniczaj czas trzymania zamków i nie wykonuj w sekcji krytycznej nieprzewidywalnego wejścia-wyjścia.
8. Dla komunikacji zdalnej określ semantykę powtórzeń, timeoutu, potwierdzeń i duplikatów.
9. Jeżeli dane mogą być niezmienne, preferuj publikowanie nowego stanu zamiast współdzielenia mutowalnej struktury.
10. Wybieraj komunikację przez komunikaty, gdy izolacja stanu jest ważniejsza niż bezpośredni dostęp do pamięci.
11. Nie utożsamiaj braku blokady z brakiem synchronizacji. Kolejka, `Future`, `receive`, `Input` i rendez-vous także narzucają kolejność i punkty oczekiwania.
12. Rozróżniaj gwarancję bezpieczeństwa od gwarancji postępu. Program może nie mieć wyścigu, ale nadal może się zakleszczyć lub zagłodzić.

## 15. Podsumowanie

Wielozadaniowość tworzy wiele równocześnie działających jednostek, natomiast synchronizacja określa ich bezpieczną współpracę. Współbieżność lokalna opiera się na pamięci współdzielonej, zamkach, semaforach, zmiennych warunkowych, barierach, monitorach i pulach wątków. Java udostępnia te mechanizmy głównie jako biblioteki, a Ada wbudowuje zadania, spotkania i obiekty chronione w język.

W systemie rozproszonym wspólna pamięć przestaje być dostępna. Synchronizacja jest wtedy realizowana przez RPC, komunikaty, kolejki, pub/sub, ZeroMQ, JMS, MPI albo przestrzeń krotek. Każde z tych podejść inaczej rozwiązuje problem czasu, kolejności, trwałości, potwierdzeń i awarii.

Najważniejsze pytanie projektowe brzmi nie „który mechanizm jest najlepszy?”, lecz:

> **Jaki stan ma być współdzielony, kto może go zmieniać, na jaki warunek trzeba czekać, jaki postęp musi być zagwarantowany i co się stanie po awarii lub powtórzeniu komunikatu?**

Dopiero odpowiedzi na te pytania pozwalają dobrać zamek, barierę, kolejkę, zadanie, komunikat lub zdalne wywołanie.

## Źródła w vaultcie

- [[0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego/09 Wielozadaniowość i synchronizacja zadań i wątków]]
- [[0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego (z prezentacji)/NPR 05 Ada 95 - zadania, spotkania i obiekty chronione]]
- [[0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego (z prezentacji)/NPR 06 Wątki w Javie i elementarna synchronizacja]]
- [[0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego (z prezentacji)/NPR 07 Pakiet java.util.concurrent]]
- [[0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego (z prezentacji)/NPR 08 MOM i systemy kolejkowania komunikatów]]
- [[0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego (z prezentacji)/NPR 09 ZeroMQ]]
- [[0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego (z prezentacji)/NPR 10 JMS]]
- [[0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego (z prezentacji)/NPR 11 Przestrzeń krotek - Linda i JavaSpaces]]
- [[0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego (z prezentacji)/NPR 01 Zdalne wywoływanie procedur (RPC)]]
- `0-przygotowanieDoObrony/NarzędziaPrzetwarzaniaRozproszonego/Opracowanie.pdf`
- `0-przygotowanieDoObrony/NarzędziaPrzetwarzaniaRozproszonego/slajdy–merged.pdf`
