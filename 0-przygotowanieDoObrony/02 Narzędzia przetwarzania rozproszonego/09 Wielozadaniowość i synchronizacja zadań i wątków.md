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

---
## Java
- `Thread` / `Runnable`, `ExecutorService` (pule wątków), `Future`, `CompletableFuture`,
- monitor wbudowany: `synchronized`, `wait`/`notify` – [[37 Monitory w C Sharp i Java]],
- `java.util.concurrent`: `ReentrantLock`, `Condition`, `Semaphore`, `CountDownLatch`, `CyclicBarrier`, `BlockingQueue`, `ConcurrentHashMap`, `Atomic*`,
- model pamięci i widoczność zmian: [[43 Model pamięci w języku Java]].

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

## Zobacz też
- [[Narzędzia Przetwarzania Rozproszonego/ADA-95]]
- [[Klasyczny Monitor]]
