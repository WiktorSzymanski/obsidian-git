---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 37
---
# 37. Monitory w języku C# lub Java
---
> **Monitor** (Hoare 1974, Brinch Hansen 1973) to konstrukcja synchronizacyjna, która łączy **dane współdzielone**, **operacje** na nich i **niejawne wzajemne wykluczanie**: w danej chwili wewnątrz monitora aktywny jest co najwyżej jeden wątek. Do **synchronizacji warunkowej** monitor udostępnia **zmienne warunkowe** z operacjami `wait` i `signal`.

Klasyczny model (kolejki wejściowa, warunkowe, pilna): [[Klasyczny Monitor]], [[Synchronizacja]].

## Semantyki sygnalizacji
Po `signal` w monitorze byłyby dwa aktywne wątki: sygnalizujący i obudzony. Semantyki różnią się tym, kto kontynuuje.
| Semantyka | Po `signal` | Warunek po obudzeniu | Wzorzec kodu |
|---|---|---|---|
| **Hoare** (_signal-and-urgent-wait_) | obudzony wątek wchodzi **natychmiast**, sygnalizujący czeka w **kolejce pilnej** (_urgent queue_) i ma pierwszeństwo przed kolejką wejściową | **gwarantowany** | `if (!warunek) wait` |
| **Brinch Hansen** (_signal-and-exit_) | `signal` musi być ostatnią instrukcją – sygnalizujący opuszcza monitor | gwarantowany | – |
| **Mesa** (_signal-and-continue_, Lampson-Redell 1980) | sygnalizujący **kontynuuje**; obudzony trafia do kolejki **wejściowej** i konkuruje o blokadę | **niegwarantowany** (inny wątek mógł zmienić stan) | `while (!warunek) wait` |

**Java i C# implementują semantykę Mesa** → warunek **zawsze** sprawdzany w pętli `while`. Dodatkowo mogą wystąpić **fałszywe przebudzenia** (_spurious wakeups_) – obudzenie bez `notify`, dopuszczone przez specyfikację.

## Java – monitory wbudowane
### Zasada
- **Każdy obiekt** ma wbudowaną **blokadę** (_intrinsic lock_, _monitor lock_) i **jedną** kolejkę oczekujących (_wait set_) – jedna niejawna zmienna warunkowa na obiekt.
- Blokada jest **wielowejściowa** (_reentrant_): wątek posiadający blokadę może ponownie wejść do metody `synchronized` tego obiektu (licznik wejść).

### `synchronized`
```java
public synchronized void metoda() { ... }        // blokada this

public static synchronized void s() { ... }      // blokada obiektu Class

public void m() {
    synchronized (lock) { ... }                   // blok – dowolny obiekt jako monitor
}
```
- Blokada zwalniana automatycznie przy wyjściu z bloku, także przez **wyjątek**.
- Brak możliwości przerwania oczekiwania na blokadę ani limitu czasu.

### `wait`, `notify`, `notifyAll` (metody klasy `Object`)
- **`wait()`** – wywołany **w posiadaniu blokady** (inaczej `IllegalMonitorStateException`): **atomowo zwalnia** blokadę obiektu i usypia wątek w wait set; po obudzeniu **ponownie zdobywa** blokadę, zanim wróci z `wait`. Wersja `wait(ms)` z limitem czasu; `InterruptedException` przy przerwaniu.
- **`notify()`** – budzi **jeden dowolny** wątek z wait set (bez gwarancji kolejności).
- **`notifyAll()`** – budzi **wszystkie** czekające wątki.
- Obudzone wątki konkurują o blokadę dopiero, gdy sygnalizujący ją zwolni.

### Przykład: bufor ograniczony
```java
public class BoundedBuffer<T> {
    private final Object[] items;
    private int putIdx, takeIdx, count;

    public BoundedBuffer(int capacity) { items = new Object[capacity]; }

    public synchronized void put(T x) throws InterruptedException {
        while (count == items.length)      // pętla: semantyka Mesa + fałszywe przebudzenia
            wait();
        items[putIdx] = x;
        putIdx = (putIdx + 1) % items.length;
        count++;
        notifyAll();                       // jedna kolejka dla producentów i konsumentów → notifyAll
    }

    @SuppressWarnings("unchecked")
    public synchronized T take() throws InterruptedException {
        while (count == 0)
            wait();
        T x = (T) items[takeIdx];
        items[takeIdx] = null;
        takeIdx = (takeIdx + 1) % items.length;
        count--;
        notifyAll();
        return x;
    }
}
```
**Dlaczego `notifyAll`, a nie `notify`?** Producenci i konsumenci czekają w **tej samej** wait set. `notify` może obudzić wątek niewłaściwego typu (np. producent budzi producenta przy pełnym buforze), który znów zaśnie, a sygnał „ginie” → możliwe **zakleszczenie / zagłodzenie**. `notify` jest bezpieczne tylko, gdy wszyscy czekający czekają na ten sam warunek i każdy obudzony może zrobić postęp.

### `java.util.concurrent.locks` – monitory jawne
```java
import java.util.concurrent.locks.*;

public class BoundedBuffer2<T> {
    private final Lock lock = new ReentrantLock(true);       // true = uczciwa (FIFO)
    private final Condition notFull  = lock.newCondition();   // WIELE zmiennych warunkowych
    private final Condition notEmpty = lock.newCondition();
    private final Object[] items = new Object[100];
    private int putIdx, takeIdx, count;

    public void put(T x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) notFull.await();
            items[putIdx] = x; putIdx = (putIdx + 1) % items.length; count++;
            notEmpty.signal();          // wystarczy signal – budzimy tylko konsumentów
        } finally {
            lock.unlock();              // zawsze w finally!
        }
    }

    @SuppressWarnings("unchecked")
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) notEmpty.await();
            T x = (T) items[takeIdx]; takeIdx = (takeIdx + 1) % items.length; count--;
            notFull.signal();
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```
Zalety `ReentrantLock` + `Condition` względem `synchronized`:
- **wiele zmiennych warunkowych** na jedną blokadę (bardziej precyzyjna sygnalizacja),
- `tryLock()`, `tryLock(timeout)`, `lockInterruptibly()` – brak nieskończonego blokowania, unikanie zakleszczeń,
- **uczciwość** (kolejność FIFO), `await(timeout)`, `awaitUninterruptibly()`,
- `ReentrantReadWriteLock` (czytelnicy-pisarze), `StampedLock` (odczyty optymistyczne).
Wada: ręczne `unlock()` w `finally` – łatwo o błąd.

Gotowe struktury: `ArrayBlockingQueue`, `LinkedBlockingQueue`, `Semaphore`, `CountDownLatch`, `CyclicBarrier`, `Exchanger`.

## C# (.NET) – klasa `Monitor`
### `lock`
```csharp
private readonly object _sync = new object();

public void Metoda()
{
    lock (_sync)          // kompilowane do Monitor.Enter/Exit w try/finally
    {
        ...
    }
}
```
Rozwinięcie kompilatora:
```csharp
bool lockTaken = false;
try { Monitor.Enter(_sync, ref lockTaken); ... }
finally { if (lockTaken) Monitor.Exit(_sync); }
```
- blokada tylko na **typach referencyjnych** (typ wartościowy byłby opakowywany → za każdym razem inny obiekt),
- wielowejściowa, jak w Javie.

### Metody statyczne `Monitor`
| Metoda | Znaczenie |
|---|---|
| `Monitor.Enter(obj)` / `Exit(obj)` | zajęcie / zwolnienie blokady |
| `Monitor.TryEnter(obj, timeout)` | próba z limitem czasu |
| `Monitor.Wait(obj)` | zwolnienie blokady i oczekiwanie (odpowiednik `wait`), `Wait(obj, timeout)` |
| `Monitor.Pulse(obj)` | budzi jeden wątek (odpowiednik `notify`) – przenosi go z kolejki oczekujących do kolejki gotowych |
| `Monitor.PulseAll(obj)` | budzi wszystkie (odpowiednik `notifyAll`) |

Każdy obiekt ma **kolejkę gotowych** (_ready queue_ – czekający na blokadę) i **kolejkę oczekujących** (_waiting queue_ – po `Wait`). Semantyka Mesa → `while`.

### Przykład: bufor ograniczony w C#
```csharp
public class BoundedBuffer<T>
{
    private readonly Queue<T> _queue = new Queue<T>();
    private readonly int _capacity;
    private readonly object _sync = new object();

    public BoundedBuffer(int capacity) => _capacity = capacity;

    public void Put(T item)
    {
        lock (_sync)
        {
            while (_queue.Count == _capacity)
                Monitor.Wait(_sync);
            _queue.Enqueue(item);
            Monitor.PulseAll(_sync);
        }
    }

    public T Take()
    {
        lock (_sync)
        {
            while (_queue.Count == 0)
                Monitor.Wait(_sync);
            T item = _queue.Dequeue();
            Monitor.PulseAll(_sync);
            return item;
        }
    }
}
```
Inne prymitywy .NET: `SemaphoreSlim` (także `WaitAsync` dla `async`), `ReaderWriterLockSlim`, `Mutex` (międzyprocesowy), `ManualResetEventSlim`, `Barrier`, `System.Threading.Lock` (.NET 9), kolekcje `BlockingCollection<T>`, `ConcurrentQueue<T>`, atrybut `[MethodImpl(MethodImplOptions.Synchronized)]` (odpowiednik metody `synchronized` – blokada `this`, niezalecany).

## Porównanie Java – C#
| | Java | C# |
|---|---|---|
| Blokada wbudowana | każdy obiekt | każdy obiekt referencyjny |
| Sekcja krytyczna | `synchronized` (metoda/blok) | `lock` (blok), `Monitor.Enter/Exit` |
| Oczekiwanie | `wait()` | `Monitor.Wait()` |
| Budzenie | `notify()`, `notifyAll()` | `Monitor.Pulse()`, `PulseAll()` |
| Zmienne warunkowe | 1 niejawna; wiele przez `Condition` | 1 niejawna na obiekt |
| Próba z limitem | `Lock.tryLock(t)` | `Monitor.TryEnter(obj, t)` |
| Semantyka | Mesa | Mesa |
| Reentrant | tak | tak |

## Typowe błędy i dobre praktyki
- **`if` zamiast `while`** przed `wait` → działanie przy niespełnionym warunku.
- **Wywołanie `wait/notify` bez blokady** → wyjątek.
- **Utracone powiadomienie** (_lost wakeup_) – sprawdzenie warunku i `wait` poza wspólną sekcją krytyczną albo `notify` przed `wait`; monitor rozwiązuje, jeśli warunek i `wait` są pod tą samą blokadą.
- **`notify` przy różnych warunkach** w jednej kolejce → zakleszczenie (patrz wyżej).
- **Blokowanie na publicznym obiekcie** (`this`, `typeof(X)`, łańcuchy znaków internowane) → obcy kod może przejąć blokadę; używać prywatnego `final`/`readonly` obiektu.
- **Zagnieżdżone monitory** (_nested monitor lockout_): wątek czeka (`wait`) w monitorze B, trzymając blokadę monitora A (wait zwalnia tylko B) → inni nie wejdą do A, by go obudzić.
- **Zakleszczenie** przy zajmowaniu kilku blokad w różnej kolejności → stała kolejność blokad, `tryLock`.
- **Długie operacje i wywołania obcego kodu** (callbacki, I/O) wewnątrz monitora → zmniejszona współbieżność, ryzyko zakleszczeń.
- Monitor Javy/C# **nie gwarantuje uczciwości** – możliwe zagłodzenie.
- Wyjątek między `lock()` a `unlock()` → zawsze `try/finally`.

## Monitory w innych językach
- **Ada 95** – obiekty chronione z **barierami** (semantyka bliższa Hoare: po każdej operacji najpierw obsługiwane oczekujące wejścia z otwartą barierą) – [[09 Wielozadaniowość i synchronizacja zadań i wątków#Obiekty chronione (Ada 95)]],
- **Concurrent Pascal**, **Modula**, **Mesa** – historyczne,
- **C++** – `std::mutex` + `std::condition_variable` (`wait(lock, predykat)`), **pthreads** – `pthread_cond_*`.
