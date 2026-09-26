---
tags:
  - obrona
  - synchronizacja
  - wielozadaniowość
  - skrót
up: "[[Wielozadaniowość i synchronizacja zadań i wątków w systemach rozproszonych]]"
zagadnienie: 9
---
# Wielozadaniowość i synchronizacja zadań/wątków w systemach rozproszonych — skrót

## 1. Istota zagadnienia

W systemach rozproszonych wiele procesów, wątków, zadań i usług wykonuje się współbieżnie. Synchronizacja odpowiada za:

- **przepływ sterowania** — kolejność operacji, oczekiwanie na warunek i przechodzenie między fazami,
- **przepływ danych** — atomowość, wzajemne wykluczanie, spójność i widoczność zmian.

Lokalnie jednostki mogą korzystać ze wspólnej pamięci. Między węzłami wspólnej pamięci nie ma, więc synchronizacja odbywa się przez komunikaty, kolejki, zdalne wywołania albo usługi koordynujące. Wtedy do problemów lokalnych dochodzą opóźnienia, utrata komunikatów, timeouty, awarie i podział sieci.

## 2. Warunki poprawności

### Atomowość i widoczność

Operacja atomowa jest niepodzielna i izolowana. Pojedynczy atomowy odczyt lub zapis nie czyni atomową całej sekwencji:

```text
odczyt -> sprawdzenie -> modyfikacja -> zapis
```

Dlatego `i++` nie jest operacją atomową.

W Javie `volatile` zapewnia widoczność zapisu i odpowiednią kolejność pamięci, ale nie zapewnia wzajemnego wykluczania. `volatile int counter; counter++` nadal jest błędne.

Relacja **happens-before** opisuje gwarantowany porządek między operacjami: zapis przed późniejszym odczytem `volatile`, `unlock` przed kolejnym `lock` tego samego zamka, `start()` przed pracą uruchamianego wątku i zakończenie wątku przed skutecznym `join()`.

> [!note] Z sieci
> Model pamięci Javy i relacja *happens-before* nie są rozwinięte w prezentacjach, ale są istotne przy synchronizacji lokalnych elementów usług rozproszonych, na przykład cache, rejestru sesji i kolejki zadań.

### Safety i liveness

- **Safety** — nic niepożądanego nie nastąpi: brak wyścigu, przepełnienia bufora, jednoczesnego wejścia do sekcji krytycznej.
- **Liveness** — coś pożądanego ostatecznie nastąpi: wątek otrzyma zasób, klient otrzyma odpowiedź albo system przejdzie do następnej fazy.

Mechanizm może zapewniać safety, ale nie liveness. Zamek chroni dane, lecz niewłaściwa kolejność zamków może doprowadzić do zakleszczenia.

### Najważniejsze problemy

- **Wyścig** — wynik zależy od kolejności przeplotu operacji.
- **Zakleszczenie** (*deadlock*) — jednostki czekają cyklicznie na zasoby lub komunikaty.
- **Zagłodzenie** (*starvation*) — jednostka jest bez końca pomijana.
- **Livelock** — jednostki wykonują działania, lecz żadna nie robi postępu.
- **Odwrócenie priorytetów** — zadanie wysokiego priorytetu czeka na zasób trzymany przez zadanie niskiego priorytetu.

Ograniczanie zakleszczeń wymaga między innymi globalnej kolejności przejmowania zasobów, krótkich sekcji krytycznych, `tryLock`, timeoutów i unikania oczekiwania na zdalną odpowiedź podczas trzymania lokalnego zamka.

## 3. Klasyczne mechanizmy synchronizacji

| Mechanizm | Zastosowanie |
|---|---|
| **Mutex** (*mutual exclusion*, wzajemne wykluczanie) | jedna sekcja krytyczna może być wykonywana przez jednego właściciela |
| **Semafor** | licznik dostępnych zasobów lub pozwoleń; `acquire`/`release` |
| **Zmienna warunkowa** | oczekiwanie na zmianę stanu; zawsze z mutexem i sprawdzaniem warunku w `while` |
| **Monitor** | hermetyzacja danych, automatyczne wykluczanie i oczekiwanie warunkowe |
| **Bariera** | wszyscy uczestnicy muszą dojść do punktu przed rozpoczęciem następnej fazy |
| **Read-write lock** | wielu czytelników albo jeden pisarz |
| **Kolejka blokująca** | bezpieczne przekazanie pracy i rozwiązanie producenta–konsumenta |

Semafor pamięta zwolnione pozwolenia. Zmienna warunkowa nie pamięta `notify`, dlatego sygnał może zostać zgubiony. Poprawny schemat zmiennej warunkowej:

```text
lock(mutex)
while warunek jest fałszywy:
    wait(condition, mutex)
wykonaj operację
unlock(mutex)
```

Pętla `while` jest konieczna z powodu fałszywych przebudzeń i możliwości, że po obudzeniu inny wątek wykorzystał zasób.

## 4. Java

### Synchronizacja niskopoziomowa

`synchronized` zajmuje zamek związany z obiektem. Zamek jest reentrant, więc ten sam wątek może wejść ponownie do sekcji chronionej tym samym zamkiem. Nie chroni to jednak przed zakleszczeniem dwóch wątków posiadających różne zamki.

`wait()`, `notify()` i `notifyAll()`:

- muszą być wywoływane przy posiadaniu zamka tego samego obiektu,
- `wait()` zwalnia zamek na czas oczekiwania i odzyskuje go przed powrotem,
- `notify()` budzi jeden wątek, `notifyAll()` wszystkie,
- sygnał wykonany, gdy nikt nie czeka, przepada.

Jeden obiekt ma jedną kolejkę oczekujących. `Lock` i `Condition` pozwalają utworzyć wiele niezależnych kolejek warunkowych oraz używać `tryLock`, oczekiwania przerywalnego i limitów czasu.

### `java.util.concurrent`

- `Semaphore` — pula pozwoleń; opcjonalna polityka FIFO.
- `CyclicBarrier` — cykliczna bariera dla ustalonej liczby uczestników.
- `CountDownLatch` — jednorazowy licznik; zgłoszenie (`countDown`) jest oddzielone od oczekiwania (`await`).
- `Phaser` — wielofazowa bariera z dynamiczną liczbą uczestników.
- `BlockingQueue` — blokuje przy pobieraniu z pustej i wstawianiu do pełnej kolejki.
- `TransferQueue` — może czekać, aż konsument odbierze element.
- `Exchanger` — synchroniczna wymiana danych między dwoma wątkami.
- `ReadWriteLock` — osobna blokada odczytu i zapisu.
- `AtomicInteger`, `AtomicReference` i inne klasy atomowe — operacje bez klasycznego zamka.

**CAS (Compare-And-Set, porównaj i ustaw)** zmienia wartość tylko wtedy, gdy nadal jest równa wartości oczekiwanej. Jest podstawą struktur lock-free. Należy uważać na problem **ABA (A-B-A)**: wartość może wrócić do A, mimo że w międzyczasie stan semantycznie się zmienił. Stosuje się wersjonowanie i znaczniki.

### Pule wątków

`ExecutorService` oddziela zadanie od wątku wykonującego. `Callable` zwraca wynik, a `Future` jest uchwytem do wyniku i pozwala czekać, anulować oraz sprawdzać stan zadania.

W `ThreadPoolExecutor`:

1. do `corePoolSize` tworzone są nowe wątki,
2. następne zadania trafiają najpierw do `workQueue`,
3. dopiero po przepełnieniu kolejki tworzone są wątki do `maximumPoolSize`,
4. dalsze zadania mogą zostać odrzucone.

`shutdown()` pozwala zakończyć bieżące zadania, a `shutdownNow()` próbuje je przerwać.

### Niezmienność

Niezmienny obiekt nie wymaga blokad, jeśli po utworzeniu nie zmienia stanu i nie udostępnia modyfikowalnych referencji. W praktyce wymaga to pól `private final`, braku metod modyfikujących, ochrony przed podklasowaniem i kopii defensywnych.

## 5. Ada 95

### Zadania i spotkania

W Adzie zadanie (*task*) jest konstrukcją języka i startuje automatycznie po deklaracji. Jego `entry` jest punktem komunikacji z innym zadaniem.

**Spotkanie** (*rendez-vous*) jest synchroniczne:

- klient wywołuje wejście,
- serwer przyjmuje je przez `accept`,
- strona, która dotrze pierwsza, czeka na drugą,
- ciało `accept` wykonuje się przy gotowych obu stronach.

`select` umożliwia:

- oczekiwanie selektywne na wiele wejść,
- wywołanie terminowe (`delay`),
- wywołanie warunkowe (`else`),
- zakończenie zadania (`terminate`),
- asynchroniczną zmianę sterowania przez `then abort`.

Gałęzie wejść mogą mieć **dozory**. Są one obliczane na początku wykonania `select`; zmiana stanu staje się widoczna dopiero przy następnym wykonaniu instrukcji.

### Obiekty chronione

Obiekt chroniony jest monitorem wbudowanym w język:

- **funkcja** — odczyt, możliwy współbieżnie,
- **procedura** — wyłączna modyfikacja,
- **wejście** — modyfikacja po spełnieniu bariery.

Bariera wejścia jest deklaratywnym warunkiem. Zadanie, które czeka na fałszywą barierę, nie trzyma blokady, więc inne zadanie może zmienić stan i otworzyć wejście. Eliminuje to ręczne `wait`/`notify` oraz problem zgubionego sygnału.

`requeue` przekazuje wywołanie do kolejki innego wejścia, gdy pełny warunek dopuszczenia można sprawdzić dopiero po wejściu do obiektu.

| Java | Ada 95 |
|---|---|
| `synchronized`, `Lock` | obiekt chroniony |
| `wait`/`notify`, `Condition` | bariery wejść |
| `ReadWriteLock` | automatyczny podział funkcja/procedura |
| `Exchanger` | przybliżenie spotkania |
| brak bezpośredniego `select` | `select` z dozorami |

Ada przenosi więcej warunków poprawności do konstrukcji języka, Java daje więcej mechanizmów bibliotecznych, ale wymaga ich ręcznego połączenia.

## 6. Synchronizacja przez komunikację rozproszoną

### RPC

**RPC (Remote Procedure Call, zdalne wywoływanie procedur)** w podstawowym wariancie blokuje klienta do czasu odpowiedzi. Wariant asynchroniczny pozwala klientowi kontynuować pracę, a callback umożliwia serwerowi późniejsze wywołanie procedury klienta.

Problemy synchronizacji RPC:

- utrata żądania lub odpowiedzi,
- retransmisja i duplikaty,
- timeout bez wiedzy, czy serwer nadal pracuje,
- awaria klienta i osierocone obliczenie,
- niepewność, czy operacja wykonała się przed awarią.

Dlatego stosuje się idempotentność, numery żądań, zapamiętywanie wyników, timeouty, próbkowanie `PING`/`PONG` i bicie serca. Semantyka **exactly-once** (dokładnie raz) nie jest bezwarunkowo osiągalna przy awariach; praktycznie rozważa się co najmniej raz, co najwyżej raz albo brak gwarancji.

### MOM i JMS

**MOM (Message-Oriented Middleware, oprogramowanie pośredniczące zorientowane na komunikaty)** rozdziela nadawcę i odbiorcę w czasie oraz przestrzeni. Broker przechowuje wiadomości w kolejce.

- **Punkt-punkt** — wiadomość trafia do jednego konsumenta i może czekać na jego uruchomienie.
- **Publish/subscribe** — wiadomość trafia do wielu aktualnych subskrybentów.

**JMS (Java Message Service, usługa komunikatów Javy)** jest specyfikacją interfejsu. Udostępnia kolejki i tematy, synchroniczne `receive()` oraz asynchroniczny `MessageListener`.

Najważniejsze mechanizmy synchronizacji i niezawodności JMS:

- potwierdzenia po odbiorze lub przetworzeniu,
- trwałe wiadomości,
- trwałe subskrypcje,
- priorytet i czas życia,
- transakcje `commit()`/`rollback()`,
- ponowne dostarczanie po wycofaniu transakcji.

Przy przetwarzaniu wiadomości trzeba uwzględnić możliwość duplikatów i zaprojektować odbiorcę idempotentnie.

### ZeroMQ

**ZeroMQ** jest biblioteką z kolejkowaniem w gniazdach, bez centralnego brokera. Podstawowe wzorce:

- `REQ-REP` — żądanie i odpowiedź,
- `PUB-SUB` — publikowanie i subskrypcja,
- `PUSH-PULL` — potokowe rozdzielanie pracy.

Brak brokera zmniejsza narzut, ale kolejki są związane z procesem i nie zapewniają automatycznie trwałości po awarii. Gniazd nie powinno się współdzielić między wątkami; lokalna synchronizacja może być zastąpiona komunikacją przez `inproc://`.

### Linda i JavaSpaces

W przestrzeni krotek komunikat jest wybierany przez dopasowanie treści do wzorca:

- `Output`/`write` — umieszczenie,
- `Input`/`take` — pobranie i usunięcie,
- `Read`/`read` — odczyt bez usuwania,
- `Try_*`/`*IfExists` — wariant nieblokujący.

Blokujące `Input` i `Read` mogą pełnić funkcję synchronizacyjną. Przestrzeń nie gwarantuje FIFO, więc kolejność należy zakodować w atrybutach. JavaSpaces dodaje dzierżawę (*lease*), która ogranicza czas życia krotki.

## 7. MPI i OpenMP

> [!note] Z sieci
> Prezentacje wymieniają MPI i OpenMP, ale nie omawiają ich szczegółowo. Poniższe punkty są krótkim uzupełnieniem.

**MPI (Message Passing Interface, interfejs przekazywania komunikatów)** synchronizuje procesy przez jawne komunikaty:

- `MPI_Send`/`MPI_Recv` — operacje blokujące,
- `MPI_Isend`/`MPI_Irecv` — rozpoczęcie operacji nieblokującej,
- `MPI_Wait`/`MPI_Test` — oczekiwanie lub sprawdzenie zakończenia,
- `MPI_Barrier` — przejście procesów do kolejnej fazy.

Po rozpoczęciu operacji nieblokującej bufor nie może być zmieniany przed jej zakończeniem.

**OpenMP (Open Multi-Processing)** synchronizuje wątki w pamięci współdzielonej:

- `critical` — wzajemne wykluczanie,
- `atomic` — atomowa aktualizacja,
- `barrier` — oczekiwanie wszystkich wątków,
- `task` — jednostka pracy,
- `taskwait` — oczekiwanie na zadania potomne,
- `reduction` — bezpieczne łączenie wyników.

MPI może rozdzielać pracę między węzły, a OpenMP tworzyć wątki wewnątrz każdego procesu.

## 8. Producent–konsument

Dla bufora o pojemności `N` musi być zachowany inwariant:

```text
0 <= liczba_elementów <= N
```

Producent czeka, gdy bufor jest pełny, a konsument, gdy jest pusty. Problem można rozwiązać przez:

- mutex i zmienne warunkowe,
- semafory,
- `BlockingQueue`,
- wejścia z barierami w Adzie,
- kolejkę MOM,
- wzorzec `PUSH-PULL` w ZeroMQ,
- blokujące `Input` w przestrzeni krotek.

Różnica polega na skali: mutex chroni lokalny stan, kolejka współbieżna przekazuje pracę w jednym procesie, a MOM/ZeroMQ przenoszą koordynację między procesami.

## 9. Dobór mechanizmu

| Potrzeba | Mechanizm |
|---|---|
| Krótka sekcja krytyczna | mutex, `synchronized`, `Lock` |
| Ograniczona liczba zasobów | semafor |
| Oczekiwanie na stan danych | condition + `while`, `Condition`, bariera wejścia |
| Wielu czytelników i jeden pisarz | `ReadWriteLock`, obiekt chroniony |
| Jednorazowe oczekiwanie na zakończenie | `CountDownLatch`, `join` |
| Powtarzalne fazy | `CyclicBarrier` |
| Dynamiczne fazy i uczestnicy | `Phaser` |
| Przekazywanie pracy w procesie | `BlockingQueue` + pula wątków |
| Komunikacja dwóch wątków | `Exchanger`, rendez-vous |
| Komunikacja między procesami | MPI, ZeroMQ, MOM/JMS |
| Trwałe rozdzielenie producenta i konsumenta | MOM/JMS |
| Wybór wiadomości po treści | Linda/JavaSpaces |
| Zdalne żądanie z odpowiedzią | RPC |

## 10. Najważniejsze wnioski

1. Rozdziel synchronizację przepływu sterowania od synchronizacji danych.
2. Chroń wspólnie cały warunek i jego modyfikację; `volatile` nie zastępuje zamka.
3. Przy zmiennych warunkowych sprawdzaj warunek w `while`.
4. Nie trzymaj lokalnego zamka podczas oczekiwania na zdalną odpowiedź.
5. Ustal kolejność przejmowania wielu zasobów i ograniczaj czas sekcji krytycznej.
6. Dla komunikacji rozproszonej określ timeouty, retry, potwierdzenia, duplikaty i semantykę wykonania.
7. Rozważ niezmienność i komunikaty jako alternatywę dla współdzielenia mutowalnego stanu.
8. Pamiętaj, że brak wyścigu nie oznacza braku zakleszczeń, zagłodzenia ani awarii protokołu.

## Zobacz też

- [[Wielozadaniowość i synchronizacja zadań i wątków w systemach rozproszonych]]
- [[0-przygotowanieDoObrony/02 Narzędzia przetwarzania rozproszonego (z prezentacji)/skompresowane/NPR 09 Wielozadaniowość i synchronizacja zadań i wątków]]
