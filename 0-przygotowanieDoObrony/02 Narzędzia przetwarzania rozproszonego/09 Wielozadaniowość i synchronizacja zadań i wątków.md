---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 9
---
# 9. Wielozadaniowość i synchronizacja zadań/wątków
---
> **Wielozadaniowość** to współbieżne wykonywanie wielu jednostek przetwarzania: procesów, wątków lub zadań. **Synchronizacja** to mechanizmy koordynacji ich dostępu do współdzielonych zasobów oraz uzależniania postępu jednych od stanu innych.

## Pojęcia
- **Proces** – izolowana przestrzeń adresowa; komunikacja przez IPC.
- **Wątek** – lekka jednostka wykonania w procesie: własny stos i rejestry, wspólna sterta, kod i deskryptory ([[Wątek]]).
- **Współbieżność** (_concurrency_) – przeplatanie wykonania, możliwe także na jednym procesorze; **równoległość** (_parallelism_) – jednoczesne wykonanie na wielu procesorach.
- **Sekcja krytyczna**, **wyścig** (_race condition_), **zakleszczenie**, **zagłodzenie**, **livelock**.
- **Bezpieczeństwo** (nic złego się nie stanie) i **żywotność** (coś dobrego w końcu nastąpi).
- Rodzaje synchronizacji: **wykluczająca** (dostęp do zasobu) i **warunkowa** (czekanie na spełnienie warunku) – [[Synchronizacja]].

### Co synchronizujemy
- **Synchronizacja procesów/wątków** – kontrola przepływu sterowania (kiedy i w jakiej kolejności wykonują się kroki).
- **Synchronizacja danych** – kontrola widoczności i spójności danych współdzielonych.

### Poziomy mechanizmów synchronizacji
- **Architektura**: operacje atomowe (`test&set`, `exchange`, `CAS`).
- **System operacyjny**: semafory, zamki, zmienne warunkowe.
- **Język/biblioteki**: monitory, obiekty chronione, kolejki i bariery wysokiego poziomu.

### Atomowość (Java)
- Atomowy jest pojedynczy odczyt/zapis typów prostych i referencji.
- Wyjątki praktyczne: `long` i `double` (historycznie możliwość nieatomowego dostępu 64-bitowego), oraz operacje typu `++`/`+=` (to sekwencja: odczyt–modyfikacja–zapis).
- `volatile` poprawia **widoczność** zmian między wątkami, ale **nie zastępuje** wzajemnego wykluczania.

## Mechanizmy synchronizacji
| Mechanizm | Opis |
|---|---|
| **mutex** | blokada binarna z właścicielem; lock/unlock |
| **semafor** (Dijkstra) | licznik; `P` (down) zmniejsza lub blokuje, `V` (up) zwiększa i budzi; ogólny lub binarny |
| **zmienna warunkowa** | czekanie na warunek zawsze razem z mutexem; `wait` atomowo zwalnia mutex i usypia |
| **monitor** | moduł/obiekt z wbudowanym wykluczaniem i zmiennymi warunkowymi – [[37 Monitory w C Sharp i Java]] |
| **blokada czytelników-pisarzy** | wielu czytelników lub jeden pisarz |
| **spinlock** | aktywne czekanie (TAS/CAS) – krótkie sekcje na wielu CPU |
| **bariera** | wątki czekają, aż zbierze się N wątków |
| **spotkanie** (_rendezvous_) | synchroniczna wymiana między zadaniami (Ada) |
| **obiekty chronione** | monitor z barierami (Ada 95) |
| **komunikaty** | synchronizacja przez send/receive (MPI, Go channels) |
| **operacje atomowe / lock-free** | CAS, `Atomic*`; pamięć transakcyjna – [[41 Pamięć transakcyjna]] |

---
## POSIX Threads (C)
Podstawy (tworzenie, join, detach, mutexy): [[Narzędzia Przetwarzania Rozproszonego/Lab 1-2 - Wprowadzenie do Aplikacji Wielowątkowych 1]].

### Zmienne warunkowe
```c
pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t  cnd = PTHREAD_COND_INITIALIZER;
int ready = 0;

/* wątek czekający */
pthread_mutex_lock(&mtx);
while (!ready)                      /* pętla! fałszywe przebudzenia */
    pthread_cond_wait(&cnd, &mtx);  /* atomowo: unlock + sleep; po obudzeniu: lock */
/* ... warunek spełniony, mutex zajęty ... */
pthread_mutex_unlock(&mtx);

/* wątek sygnalizujący */
pthread_mutex_lock(&mtx);
ready = 1;
pthread_cond_signal(&cnd);          /* budzi jeden; pthread_cond_broadcast - wszystkie */
pthread_mutex_unlock(&mtx);
```
- Warunek sprawdzany **w pętli `while`**: fałszywe przebudzenia (_spurious wakeups_), a po obudzeniu inny wątek mógł już zmienić stan (semantyka Mesa).
- `pthread_cond_timedwait` – czekanie z limitem czasu.
- Inne: `pthread_rwlock_*`, `pthread_spin_*`, `pthread_barrier_*`, `pthread_once`, semafory POSIX `sem_wait`/`sem_post`, TLS `pthread_key_create`.

### Przykład: bufor ograniczony
```c
void put(int x) {
    pthread_mutex_lock(&m);
    while (count == N) pthread_cond_wait(&notFull, &m);
    buf[in] = x; in = (in + 1) % N; count++;
    pthread_cond_signal(&notEmpty);
    pthread_mutex_unlock(&m);
}
```

---
## Ada – zadania i spotkania
### Zadania (_tasks_)
```ada
task type Worker is
   entry Start (Id : in Integer);   -- wejście = punkt spotkania
   entry Stop;
end Worker;

task body Worker is
   My_Id : Integer;
begin
   accept Start (Id : in Integer) do
      My_Id := Id;                  -- wykonywane podczas spotkania
   end Start;
   loop
      select
         accept Stop;  exit;
      or
         delay 1.0;                  -- alternatywa czasowa
         Put_Line ("pracuję");
      end select;
   end loop;
end Worker;
```
- Zadanie startuje automatycznie po deklaracji (_elaboracja_); jednostka nadrzędna czeka na zakończenie zadań podrzędnych.
- **Spotkanie** (_rendezvous_): wywołujący `W.Start(1)` czeka, aż zadanie wykona `accept Start`; zadanie przy `accept` czeka na wywołującego. Parametry `in`/`out` przekazywane w obie strony – komunikacja **synchroniczna i asymetryczna** (wywołujący zna nazwę zadania, zadanie nie zna wywołującego).
- Każde wejście ma **kolejkę FIFO** wywołań.

### Instrukcja `select`
- **selektywne oczekiwanie** (po stronie serwera): wiele gałęzi `accept` ze **strażnikami** `when warunek =>`; wybierana gałąź z oczekującym wywołaniem i otwartym strażnikiem; dodatkowe gałęzie: `delay` (timeout), `else` (brak czekania), `terminate` (zakończ, gdy nikt już nie może wywołać),
- **wywołanie warunkowe / czasowe** (po stronie klienta): `select W.Start(1); else ... end select;` lub `or delay 2.0;`,
- **asynchroniczna zmiana wątku sterowania** (ATC): `select delay 5.0; then abort Obliczenia; end select;`.

### Obiekty chronione (Ada 95)
```ada
protected type Buffer is
   entry Put (X : in Integer);      -- blokująco, z barierą
   entry Get (X : out Integer);
   function Count return Natural;   -- tylko odczyt, wiele współbieżnie
private
   Data : Int_Array; N : Natural := 0;
end Buffer;

protected body Buffer is
   entry Put (X : in Integer) when N < Size is   -- bariera
   begin ... N := N + 1; end Put;
   entry Get (X : out Integer) when N > 0 is
   begin ... N := N - 1; end Get;
   function Count return Natural is begin return N; end;
end Buffer;
```
- **Funkcje** – współbieżny odczyt (blokada czytelnika); **procedury** – wyłączny zapis; **wejścia** – wyłączny dostęp z **barierą** (warunkiem logicznym).
- Po każdej procedurze/wejściu bariery są przeliczane, a zakolejkowane wywołania z otwartą barierą obsługiwane są **przed** nowymi (**model „eggshell”**) – brak fałszywych przebudzeń i konieczności pętli.
- `requeue` – przekazanie wywołania do innej kolejki wejścia.

### Bariera ważona (Ada i Java – z opracowania)
W PDF-ie pojawia się wariant bariery, gdzie każde zgłoszenie może mieć „wagę” (np. `weight`), a odblokowanie następuje po przekroczeniu progu.

```ada
protected Barrier is
  entry Await (Weight : in Integer);
private
  entry Inner_Await;
  Count : Integer := 10;
  Barrier_Strength : Integer := 10;
end Barrier;
```

Wersja ideowa w Javie:
```java
public synchronized void breakThrough(int breakCount) throws InterruptedException {
    currentBreak += breakCount;
    if (currentBreak < strength) wait();
    else { currentBreak = 0; notifyAll(); }
}
```

- W notatkach do PDF zaznaczono też kompromis: **fairness** (sprawiedliwość/FIFO) vs **throughput** (przepustowość).

---
## Java
- `Thread` / `Runnable`, `ExecutorService` (pule wątków), `Future`, `CompletableFuture`,
- monitor wbudowany: `synchronized`, `wait`/`notify` – [[37 Monitory w C Sharp i Java]],
- `java.util.concurrent`: `ReentrantLock`, `Condition`, `Semaphore`, `CountDownLatch`, `CyclicBarrier`, `BlockingQueue`, `ConcurrentHashMap`, `Atomic*`,
- model pamięci i widoczność zmian: [[43 Model pamięci w języku Java]].

### Uzupełnienia z PDF
- `interrupt()` to bezpieczny mechanizm przerywania czekania (`InterruptedException`); historyczne `stop()`/`suspend()` są niezalecane.
- `Lock`/`Condition` daje wiele kolejek warunkowych na jeden zamek (w odróżnieniu od pojedynczej kolejki `wait` na monitorze obiektu).
- Mechanizmy barierowe:
  - `CyclicBarrier` – cykliczna, dla stałej liczby uczestników,
  - `CountDownLatch` – jednorazowa, licznik malejący,
  - `Phaser` – dynamiczna liczba uczestników/faz.
- Kolekcje i narzędzia synchronizacyjne: `BlockingQueue`, `TransferQueue`, `Exchanger`, `ConcurrentMap`, `Semaphore(fair=true/false)`.

## MPI – synchronizacja przez komunikaty
- punkt-punkt blokujące (`MPI_Send`, `MPI_Recv`) i nieblokujące (`MPI_Isend`, `MPI_Irecv` + `MPI_Wait`/`MPI_Test`),
- `MPI_Ssend` – synchroniczny (czeka na rozpoczęcie odbioru) – rendezvous,
- `MPI_Barrier` – bariera, operacje kolektywne jako punkty synchronizacji,
- hybryda MPI + OpenMP/pthreads: `MPI_Init_thread` z poziomami `MPI_THREAD_SINGLE/FUNNELED/SERIALIZED/MULTIPLE`.

## OpenMP (pamięć współdzielona)
`#pragma omp parallel`, `for`, `sections`, `task`; synchronizacja: `critical`, `atomic`, `barrier`, `ordered`, `omp_lock_t`; klauzule `private`/`shared`/`reduction`.

## ZeroMQ – wielowątkowość bez blokad
Zasada: **nie współdzielić gniazd między wątkami**. Wątki komunikują się przez gniazda `inproc://` (np. PUSH/PULL), co zastępuje blokady wymianą komunikatów.

---
## Porównanie narzędzi
| | pthreads | Ada | Java | MPI |
|---|---|---|---|---|
| Jednostka | wątek | zadanie (_task_) | wątek | proces (rank) |
| Pamięć | współdzielona | współdzielona (+ rozproszone Annex E) | współdzielona | rozproszona |
| Wykluczanie | mutex | obiekt chroniony | `synchronized`, `Lock` | brak wspólnych danych |
| Warunki | zmienne warunkowe (pętla) | bariery wejść | `wait`/`Condition` (pętla) | odbiór komunikatu |
| Komunikacja | wspólne zmienne | rendezvous (synchron.) | wspólne zmienne / kolejki | send/recv |
| Wsparcie języka | biblioteka | wbudowane w język | język + biblioteka | biblioteka |
| Bezpieczeństwo | łatwe błędy (brak unlock) | kontrola kompilatora, brak fałszywych przebudzeń | średnie | brak wyścigów pamięci |

## TODO (braki względem zakresu egzaminacyjnego i PDF-ów)
> [!todo] Zagadnienia, których PDF-y nie domykają lub tylko sygnalizują
> - Formalna relacja **happens-before** w Java Memory Model (pełne reguły).
> - Problem **ABA** przy operacjach `compareAndSet` i techniki obejścia.
> - Priorytety zadań w Adzie i **priority ceiling protocol** (szczegóły).
> - Pełne, formalne rozwiązania klasyków synchronizacji (czytelnicy-pisarze, filozofowie) – w PDF-ach głównie sygnalizacja/zadania.
> - Zaawansowane wzorce lock-free/wait-free i ich gwarancje postępu.

## Z sieci

### 1) Java Memory Model (JMM) i relacja happens-before
- **Cel JMM**: formalnie opisać, jakie wartości może odczytać wątek i kiedy zapis jednego wątku jest gwarantowanie widoczny dla drugiego. Bez tego kompilator/JIT/CPU mógłby legalnie wykonywać optymalizacje, które „łamą” intuicję sekwencyjnego działania programu wielowątkowego.
- **`happens-before` (HB)** to relacja porządkująca. Jeśli A HB B, to:
  1. efekty A muszą być widoczne w B,  
  2. B nie może „przesunąć się logicznie” przed A.
- Kluczowe reguły HB:
  - **program order**: w obrębie jednego wątku wcześniejsze instrukcje HB późniejsze,
  - **monitor lock rule**: `unlock(m)` HB każde późniejsze `lock(m)` tego samego monitora,
  - **volatile rule**: zapis do zmiennej `volatile` HB każdy późniejszy odczyt tej samej zmiennej,
  - **thread start rule**: wywołanie `start()` HB pierwsze akcje uruchamianego wątku,
  - **thread termination rule**: wszystkie akcje wątku HB skuteczny `join()` na tym wątku,
  - **transitivity**: jeśli A HB B i B HB C, to A HB C.
- **Dlaczego to ważne w systemach rozproszonych?**  
  Węzeł aplikacji ma zwykle wiele wątków: obsługa RPC, I/O, kolejki, timeouty, retry. Błędna publikacja stanu lokalnego (np. cache, mapa sesji, metadane kolejki) potrafi dać subtelne błędy semantyczne widoczne „jakby” były błędem sieci.
- **Praktyczny wzorzec**:
  - flaga zatrzymania/zdrowia komponentu: `volatile boolean running`,
  - złożony stan współdzielony: ochrona przez `synchronized`/`Lock`,
  - nigdy nie traktuj `volatile` jako zamiennika sekcji krytycznej.

### 2) CAS i problem ABA
- **CAS (Compare-And-Set)**: atomowa operacja „ustaw nową wartość tylko wtedy, gdy obecna jest równa oczekiwanej”. To podstawa większości struktur lock-free.
- **A-B-A**:
  - wątek T1 odczytuje `A`,
  - wątek T2 zmienia `A -> B -> A`,
  - T1 robi CAS i dostaje sukces, choć stan semantyczny mógł się zmienić (np. inny element stosu został w międzyczasie zdjęty i ponownie użyty).
- **Gdzie boli najmocniej?**
  - lock-free stack/queue/list,
  - struktury z recyklingiem węzłów,
  - kod „niskopoziomowy” pod bardzo dużą konkurencją.
- **Techniki obrony**:
  - **stemple/wersje** (`value + version`), np. `AtomicStampedReference`,
  - **markowanie** stanu referencji (`AtomicMarkableReference`),
  - **memory reclamation discipline**: hazard pointers, epoch-based reclamation, RCU-like podejścia,
  - unikanie agresywnego recyklingu obiektów.
- **Wniosek praktyczny**: lock-free zwiększa skalowalność, ale kosztuje dużo więcej w dowodzeniu poprawności niż klasyczny lock.

### 3) Ada: priorytety i Priority Ceiling Protocol (PCP)
- **Problem bazowy**: odwrócenie priorytetów – niski priorytet trzyma zasób, wysoki czeka, a średni „wypycha” niskiego z CPU i wydłuża blokadę wysokiego.
- **Priority Ceiling Protocol**:
  - każdemu obiektowi chronionemu przypisuje się pułap równy najwyższemu priorytetowi zadań, które mogą go używać,
  - wejście do sekcji krytycznej podnosi efektywny priorytet zadania tak, by ograniczyć preempcję,
  - dzięki temu maleje ryzyko długotrwałej i trudnej do oszacowania inwersji priorytetów.
- **Znaczenie inżynierskie**:
  - silniejsza przewidywalność opóźnień (latency bounds),
  - łatwiejsza analiza schedulability,
  - realna poprawa stabilności komponentów RT (np. gatewaye IoT, sterowniki, węzły edge z krytycznymi deadline’ami).

### 4) Klasyczne problemy synchronizacji (ujęcie egzaminacyjne)
- **Producent–Konsument (bufor ograniczony)**:
  - inwariant: `0 <= count <= N`,
  - producent czeka gdy bufor pełny, konsument gdy pusty,
  - poprawna implementacja wymaga `while`, nie `if` (fałszywe przebudzenia i wyścigi po przebudzeniu),
  - poprawność obejmuje i **safety** (brak przepełnienia/podpełnienia), i **liveness** (brak zakleszczenia/zagłodzenia).
- **Czytelnicy–Pisarze**:
  - polityki: preferencja czytelników, preferencja pisarzy, fair,
  - trade-off: przepustowość vs ryzyko zagłodzenia jednej grupy,
  - w systemach usługowych fair policy bywa lepsza niż maksymalizacja throughput, bo stabilizuje tail latency.
- **Jedzący filozofowie**:
  - pokazuje deadlock i starvation,
  - typowe naprawy: globalne porządkowanie zasobów, lokaj/semafor N-1, asymetria pobierania,
  - to model wielu problemów „alokuj wiele zasobów albo żaden” (połączenia, locki, sloty I/O).
- **Wersja rozproszona**:
  - lokalne sekcje krytyczne zastępuje się kolejkami komunikatów, tokenem lub consensus/lock service,
  - dochodzą awarie, timeouty i częściowa synchronia – sama poprawność algorytmu lokalnego nie wystarcza.

### 5) Lock-free / wait-free: gwarancje postępu
- **Blocking**: wątek może zatrzymać innych (np. trzyma zamek i padnie).
- **Lock-free**: system jako całość robi postęp (któryś wątek skończy w skończonej liczbie kroków).
- **Wait-free**: każdy wątek kończy operację w skończonej liczbie własnych kroków.
- **Obstruction-free**: postęp przy braku konkurencji.
- **Praktyka inżynierska**:
  - lock-free często zwiększa throughput pod dużą konkurencją,
  - koszt: trudniejsze dowodzenie poprawności, ABA i zarządzanie pamięcią,
  - lock-free nie gwarantuje fairness (pojedynczy wątek może głodować),
  - do prostych sekcji krytycznych z małą konkurencją często lepszy jest dobry `Lock` (czytelność, debugowalność, niższy koszt poznawczy).

### 6) Synchronizacja w podejściach rozproszonych (narzędzia)
- **MPI**:
  - synchronizacja przez komunikaty i operacje kolektywne (`Barrier`, `Bcast`, `Reduce`),
  - `Isend/Irecv` + `Wait/Test` umożliwiają overlap komunikacji i obliczeń,
  - model wymusza jawne myślenie o punktach synchronizacji między procesami/rankami.
- **ZeroMQ**:
  - model „share-nothing” między wątkami (brak współdzielonych gniazd),
  - wzorce `PUSH/PULL`, `REQ/REP`, `PUB/SUB` realizują koordynację bez klasycznych locków,
  - dobra praktyka: traktować kanał komunikacji jako granicę izolacji stanu.
- **Kafka / log-based systems**:
  - synchronizacja przez uporządkowany log i offsety konsumentów,
  - porządek gwarantowany per-partition, więc klucze partycjonowania wpływają na semantykę współbieżności,
  - przetwarzanie idempotentne + retry to praktyczny odpowiednik „bezpiecznej współbieżności” przy awariach.

### 7) Szybka ściąga: co dobrać do problemu
- **Wspólna pamięć + krótka sekcja krytyczna**: mutex/lock.
- **Czekanie na warunek stanu**: condition variable + `while`.
- **Limit zasobu (N miejsc/slotów)**: semaphore.
- **Faza „wszyscy muszą dojść”**: barrier/latch/phaser.
- **Skalowanie między procesami/węzłami**: komunikaty (MPI/ZeroMQ/Kafka), nie współdzielona pamięć.
- **Wysoka konkurencja i niski latency**: struktury lock-free (z kontrolą ABA i memory reclamation).
- **Deadline’y i priorytety (RT)**: PCP / protokoły priorytetowe + krótki czas trzymania zasobu.

## Zobacz też
- [[Narzędzia Przetwarzania Rozproszonego/ADA-95]]
- [[Klasyczny Monitor]]
