---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 9
source: "slajdy–merged.pdf"
slajdy: "115–152"
---
# NPR 06. Wątki w Javie i elementarna synchronizacja
---
> Dwa wykłady: **„Obsługa i elementarna synchronizacja wątków"** (slajdy 115–138) i **„Koordynacja wątków, spójność danych"** (139–152). Pierwszy omawia wątek jako byt — dwa sposoby tworzenia, diagram stanów, API klasy `Thread`, demony i grupy wątków. Drugi wprowadza **trzy poziomy mechanizmów synchronizacji**, a następnie mechanizmy „niskopoziomowe" Javy: `synchronized` oraz `wait`/`notify`/`notifyAll`, kończąc pytaniem, czy da się z nich zbudować **monitor**. Mechanizmy „wysokopoziomowe" — zob. [[NPR 07 Pakiet java.util.concurrent]].

---
## Zakres wykładu
<sub>slajdy–merged.pdf, slajd 116</sub>

Tworzenie wątków · stany wątków i ich zmiana · demony · grupy wątków · synchronizacja wątków (wzajemne wykluczanie, oczekiwanie na zmiennych warunkowych, pakiet `java.util.concurrent`).

---
## Tworzenie wątków
<sub>slajdy–merged.pdf, slajdy 117–123</sub>

- Wątek reprezentowany jest w procesie na JVM przez **obiekt klasy `Thread`** (w szczególności jej pochodnej).
- **Programem głównym wątku** jest metoda `run()` klasy wywiedzionej z `Thread` **lub** dowolnej klasy implementującej interfejs `Runnable`.

Stąd dwa sposoby:

| Sposób | Kroki |
|---|---|
| **Dziedziczenie z klasy `Thread`** | definicja klasy pochodnej od `Thread`; utworzenie obiektu zdefiniowanej klasy |
| **Implementacja interfejsu `Runnable`** | definicja klasy implementującej `Runnable`; utworzenie obiektu tej klasy; utworzenie obiektu klasy `Thread` **z przekazaniem referencji** do utworzonego obiektu |

### Wariant 1 — dziedziczenie
<sub>slajd 119</sub>

```java
class MThread extends Thread {
    void run() {
        ...
    }
}

MThread th;
th = new MThread();
```

### Wariant 2 — `Runnable`
<sub>slajd 120</sub>

```java
class MClass implements Runnable {
    void run() {
        ...
    }
}

MClass obj = new MClass();
Thread  th = new Thread(obj);
```

### Dlaczego to działa
<sub>slajdy 121–122</sub>

Interfejs `Runnable` deklaruje jedną metodę:

```java
public interface Runnable {
   public abstract void run();
}
```

a klasa `Thread` ma domyślną implementację `run()`, która **deleguje do przekazanego obiektu**:

```java
private Runnable target;

public void run() {
    if (target != null)
        target.run();
}
```

Referencja `target` ustawiana jest w konstruktorze, jeśli zostanie przekazany parametr klasy implementującej `Runnable`.

Oba warianty są więc **tym samym mechanizmem**: `Thread` zawsze woła `run()`, a różnica polega na tym, czy przesłonięto tę metodę, czy podstawiono `target`. Wariant z `Runnable` jest praktyczniejszy, bo Java nie ma wielodziedziczenia — klasa już dziedzicząca po czymś innym nie mogłaby rozszerzyć `Thread`.

### Konstruktory `Thread`
<sub>slajd 123</sub>

```java
Thread();
Thread(Runnable target);
Thread(String name);
Thread(Runnable target, String name);
```

---
## Stany wątku
<sub>slajdy–merged.pdf, slajd 124</sub>

![[npr-thr-s124-stany-watku.png]]
<sub>Diagram stanów wątku: `initial` → (`start()`) → `runnable`; uśpienie jawne lub w wyniku synchronizacji prowadzi do `blocked`, obudzenie lub przerwanie uśpienia wraca do `runnable`; `suspend()` przenosi do stanów `suspended`, `resume()` z nich wyprowadza; `stop()` lub zakończenie `run()` prowadzi do `exiting`. slajdy–merged.pdf, slajd 124</sub>

Diagram ma **dwa niezależne wymiary**: *zablokowany czy nie* oraz *zawieszony czy nie* — stąd cztery stany pośrednie, a nie dwa. Metody `suspend()`/`resume()` są pozostałością wczesnych wersji Javy.

### Metody zmieniające stan
<sub>slajdy 125–126</sub>

| Metoda | Działanie |
|---|---|
| `void start()` | uruchomienie wątku |
| `void stop()` | zakończenie działania wątku |
| `void run()` | metoda wykonywana przez wątek (główny program wątku) |
| `void suspend()` | zawieszenie wątku — **wątek nie zwalnia blokad** |
| `void resume()` | wznowienie wykonywania zawieszonego wątku |
| `void interrupt()` | przerwanie oczekiwania wątku w stanie zablokowania |
| `static void sleep(long milsec [, int nanosec])` | uśpienie wątku na podany okres czasu |
| `static void yield()` | „oddanie procesora" innemu wątkowi **o tym samym priorytecie** |

Uwaga ze slajdu 125 — **`suspend()` nie zwalnia blokad** — jest powodem, dla którego ta metoda jest niebezpieczna: wątek zawieszony w sekcji krytycznej blokuje wszystkich pozostałych i nikt nie może go wznowić, jeśli sam potrzebuje tego samego zamka.

Dwie metody statyczne (`sleep`, `yield`) różnią się od reszty tym, że **zmiana stanu następuje w wątku wywołującym** — wątek woła je, żeby zmienić **własny** stan.

### Przerwania
<sub>slajd 127</sub>

- **Przerwanie** — wywołanie metody `interrupt()` na obiekcie wątku — przerywa oczekiwanie wątku (np. w `sleep`, `join`, `wait`) poprzez zgłoszenie wyjątku **`InterruptedException`**.
- Jeśli wątek **nie jest w stanie `blocked`**, fakt przerwania jest odnotowywany poprzez ustawienie odpowiedniej flagi (_interrupted status_).
- `static Thread.interrupted()` — zwraca `true`, jeśli flaga jest ustawiona (dla **bieżącego** wątku), **i ją kasuje**.
- `isInterrupted()` (na obiekcie wątku) — zwraca tę informację dla danego wątku, **ale nie kasuje flagi**.

Przerwanie jest **prośbą, nie rozkazem**: nie zatrzymuje wątku, tylko zostawia mu informację albo wytrąca go z czekania. To jedyny bezpieczny sposób kończenia wątków — w przeciwieństwie do `stop()`.

### Pozostałe metody
<sub>slajdy 128–131</sub>

| Grupa | Metody |
|---|---|
| **nazwa** | `void setName(String name)`, `String getName()` — z punktu widzenia systemu nazewnictwo wątków **nie ma żadnego znaczenia** |
| **priorytet** | `void setPriority(int priority)`, `int getPriority()`; stałe `final`: `Thread.MIN_PRIORITY`, `Thread.MAX_PRIORITY`, `Thread.NORM_PRIORITY` — **większa wartość oznacza wyższy priorytet** |
| **oczekiwanie na zakończenie** | `void join([long milsec [, int nanosec]])`; `boolean isAlive()` — `true`, jeśli wątek został uruchomiony przez `start()`, ale nie zakończył jeszcze działania |
| **statyczne, o wątkach procesu** | `static Thread currentThread()`; `static int enumerate(Thread threadArray[])`; `static int activeCount()` |

---
## Demony
<sub>slajdy–merged.pdf, slajd 132</sub>

**Demon** jest takim wątkiem, który **kończy swoje działanie po zakończeniu ostatniego wątku użytkownika**.

| Metoda | Działanie |
|---|---|
| `void setDaemon(boolean on)` | zmienia wątek użytkownika na wątek-demon lub odwrotnie |
| `boolean isDaemon()` | sprawdza, czy wątek jest demonem |

Demon rozwiązuje ten sam problem, co gałąź `terminate` w Adzie — zob. [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione#Gałąź `terminate`|NPR 05]] — ale w sposób znacznie prostszy i mniej precyzyjny: demon jest **zabijany** w dowolnym miejscu, gdy zniknie ostatni wątek użytkownika, podczas gdy zadanie Ady wybiera `terminate` tylko w bezpiecznym punkcie.

---
## Grupy wątków
<sub>slajdy–merged.pdf, slajdy 133–138</sub>

- Łączenie wątków w grupy ma na celu **ułatwienie zarządzania zbiorami logicznie powiązanych ze sobą wątków** (np. grupa wątków w serwerze do obsługi określonego klienta na połączeniu sieciowym).
- Wątek musi zostać **przypisany do grupy w momencie tworzenia** i **pozostaje w niej do końca swego istnienia**.
- Grupy tworzą **hierarchię** wynikającą z zawierania się jednych grup w innych — każda nowo tworzona grupa jest częścią innej grupy.

Przypisanie następuje przez konstruktor `Thread`:

```java
Thread(ThreadGroup group, Runnable target);
Thread(ThreadGroup group, String name);
Thread(ThreadGroup group, Runnable target, String name);
```

Grupa reprezentowana jest przez obiekt klasy `ThreadGroup`:

```java
ThreadGroup(String name)                       // podgrupa grupy wątku bieżącego
ThreadGroup(ThreadGroup parent, String name)   // podgrupa grupy wskazanej
```

| Grupa metod | Metody |
|---|---|
| **operacje zbiorcze** | `void stop()`, `void suspend()`, `void resume()` — na **wszystkich** wątkach w grupie |
| **przeglądanie** | `int enumerate(Thread list[])`, `int enumerate(Thread list[], boolean recurse)`, `int activeCount()`, `int enumerate(ThreadGroup list[])`, `int enumerate(ThreadGroup list[], boolean recurse)` |
| **usuwanie** | `destroy()` |

`destroy()` jest skuteczna **tylko jeśli wszystkie wątki w grupie i podgrupach zostały zakończone**, i **rekurencyjnie usuwa również wszystkie podgrupy** grupy usuwanej.

---
## Poziomy mechanizmów synchronizacji
<sub>slajdy–merged.pdf, slajdy 140–141</sub>

Najpierw rozróżnienie **co** synchronizujemy:

| Rodzaj | Na czym polega |
|---|---|
| **Synchronizacja procesów/wątków** | **koordynacja realizacji** poszczególnych instrukcji (kroków, faz); **kontrola przepływu sterowania** |
| **Synchronizacja danych** | **utrzymanie spójności danych**; gwarancja **dostępu do najświeższych wartości** zmiennych/stanów obiektów (uwzględnienie wyników ostatnich modyfikacji) |

Potem **na jakim poziomie** mechanizm działa:

| Poziom | Mechanizmy |
|---|---|
| **architektury systemu komputerowego** | zapis/odczyt współdzielonych zmiennych (tzw. **współdzielone rejestry**); **złożone operacje realizowane niepodzielnie**, np. `test&set`, `exchange` |
| **systemu operacyjnego** | zarządzanie procesami/wątkami (ich stanem), **integracja z mechanizmami przydziału procesora** (szeregowania), np. **semafory, zamki, zmienne warunkowe** |
| **języka programowania** | **strukturalne** mechanizmy synchronizacji udostępniające konstrukcje do wyrażania zależności i ograniczeń w dostępie do współdzielonych zasobów — **monitory, regiony krytyczne** |

Ta trójpodziałka porządkuje cały materiał przedmiotu: `test&set` to poziom sprzętu, `Semaphore` z `java.util.concurrent` to poziom systemu, a **obiekt chroniony Ady** to poziom języka — zob. [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione#Obiekty chronione|NPR 05]]. Java, jak się okaże, zatrzymuje się **pomiędzy** poziomem systemu a poziomem języka.

### Podział mechanizmów Javy
<sub>slajd 142</sub>

| Grupa | Mechanizmy |
|---|---|
| **„niskopoziomowe"** | wzajemne wykluczanie — **blok/metoda `synchronized`**; oczekiwanie na spełnienie warunku — **`wait()`, `notify()`, `notifyAll()`**. Blok `synchronized` oraz te metody mogą być realizowane na **dowolnym obiekcie** (obiekcie klasy `Object`) |
| **„wysokopoziomowe"** — pakiet `java.util.concurrent` (od wersji 1.5) | atomowe operacje na obiektach; **zamki (`Lock`) i zmienne warunkowe (`Condition`)**; **semafory (`Semaphore`), bariery (`CyclicBarrier`)** itp.; współbieżnie dostępne kolekcje (`ConcurrentHashMap`, `ConcurrentLinkedQueue`, `CopyOnWriteArrayList`, `CopyOnWriteArraySet`) |

Fakt, że `synchronized` i `wait`/`notify` działają na **dowolnym obiekcie**, jest podstawową decyzją projektową Javy: każdy obiekt ma zamek i jedną kolejkę oczekujących, niezależnie od tego, czy go potrzebuje.

---
## Wzajemne wykluczanie — `synchronized`
<sub>slajdy–merged.pdf, slajd 143</sub>

- **Blok `synchronized`** na danym obiekcie **zajmuje zamek związany (integralnie) z tym obiektem**:

```java
synchronized (obj) {
   ...
}
```

- **Metoda typu `synchronized`** zajmuje zamek związany z **obiektem, dla którego jest wywoływana** — będzie zatem wykluczać wykonanie **innych metod typu `synchronized` lub bloków `synchronized` na tym obiekcie**.

### Przykład
<sub>slajd 144</sub>

```java
public class Konto {
   private float kwota;

   public synchronized boolean wyplata(float k) {
      if (k <= kwota) {
         kwota -= k;
         return true;
      }
      return false;
   }

   public synchronized void wplata(float k) {
      kwota += k;
   }
}
```

Bez `synchronized` sprawdzenie `k <= kwota` i odjęcie mogłyby zostać przedzielone wykonaniem innego wątku — konto zeszłoby poniżej zera.

### Synchronizacja fragmentu metody
<sub>slajd 145</sub>

Jeśli **tylko fragment** kodu metody ma się wykluczać z innymi metodami typu `synchronized`, można to osiągnąć przez utworzenie bloku `synchronized` **na referencji `this`**:

```java
{
    ...
    synchronized (this) {
        ...
    }
}
```

Jest to dokładny odpowiednik metody `synchronized` zawężony do fragmentu — bo metoda `synchronized` zajmuje zamek **tego samego** obiektu `this`.

### Zamek wielowejściowy
<sub>slajdy 146–147</sub>

Slajd 146 stawia pytanie: **co się stanie, jeśli metoda typu `synchronized` zostanie wywołana z innej metody typu `synchronized` (czyli przez ten sam wątek)? Czy nastąpi zakleszczenie, a jeśli nie, to czy nie nastąpi przedwczesne zwolnienie blokady obiektu?**

![[npr-thr-s147-zamek-wielowejsciowy.png]]
<sub>slajdy–merged.pdf, slajd 147</sub>

Odpowiedź, wprost ze slajdu: **zamek zintegrowany z obiektem jest wielowejściowy** (ang. _reentrant_). Ten sam wątek może go zająć wielokrotnie — nie zakleszcza się. Zwolnienie następuje dopiero po wyjściu z **ostatniej** sekcji, więc nie ma też przedwczesnego zwolnienia; zamek zlicza zagnieżdżenia.

### Zakleszczenie
<sub>slajd 148</sub>

Wielowejściowość chroni przed zakleszczeniem wątku **z samym sobą na jednym obiekcie**, ale nie przed zakleszczeniem **dwóch wątków na dwóch obiektach**:

![[npr-thr-s148-zakleszczenie.png]]
<sub>Wątek A trzyma zamek obiektu `a` i woła `b.method()`; wątek B trzyma zamek obiektu `b` i woła `a.method()`. ZAKLESZCZENIE. slajdy–merged.pdf, slajd 148</sub>

Jest to klasyczne **oczekiwanie cykliczne** — zob. [[RSO 06 Zakleszczenie w systemach rozproszonych]].

---
## Oczekiwanie na spełnienie warunku
<sub>slajdy–merged.pdf, slajd 149</sub>

| Metoda | Działanie |
|---|---|
| `void wait([long milsec [, int nanosec]])` | czeka na spełnienie warunku — na **sygnał** wysyłany przez `notify()` lub `notifyAll()` |
| `void notify()` | wysyła sygnał do **wątku** oczekującego po wywołaniu `wait()` danego obiektu |
| `void notifyAll()` | wysyła sygnał do **wszystkich** wątków oczekujących po wywołaniu `wait()` |

**Metody `wait()`, `notify()` i `notifyAll()` muszą być wywoływane w bloku (metodzie) `synchronized` na tym samym obiekcie**, w przeciwnym przypadku zgłaszany jest wyjątek `NotOwnerException`.

Wymóg ten jest konieczny, bo `wait()` **zwalnia zamek** na czas czekania i **odzyskuje go** przed powrotem — bez posiadania zamka nie byłoby czego zwalniać.

### Zgubiony sygnał
<sub>slajd 150</sub>

![[npr-thr-s150-zgubiony-sygnal.png]]
<sub>Pierwsze `notify()` wątku B następuje, zanim wątek A wejdzie w `wait()` — sygnał jest **ignorowany**. Dopiero drugie `notify()` jest sygnałem budzącym. slajdy–merged.pdf, slajd 150</sub>

To podstawowa różnica między **zmienną warunkową a semaforem**: sygnał **nie jest pamiętany**. Jeśli nikt nie czeka w chwili `notify()`, sygnał przepada bezpowrotnie, a wątek, który wejdzie w `wait()` chwilę później, będzie czekał w nieskończoność.

### Poprawny wzorzec użycia
<sub>slajd 151</sub>

![[npr-thr-s151-schemat-wait-notify.png]]
<sub>Wątek A w sekcji `synchronized` sprawdza, czy warunek jest spełniony, i jeśli nie — woła `wait()`. Wątek B w sekcji `synchronized` zmienia stan (modyfikuje zmienne/obiekty), po czym woła `notify()`. slajdy–merged.pdf, slajd 151</sub>

Z diagramu wynikają dwie reguły, których slajd nie wypowiada wprost, ale które z niego wynikają: **sprawdzenie warunku i `wait()` muszą być w tej samej sekcji krytycznej** (inaczej stan mógłby się zmienić między sprawdzeniem a zaśnięciem — to właśnie zgubiony sygnał), a **zmiana stanu musi poprzedzać `notify()`** i być w tej samej sekcji.

---
## Czy da się zbudować monitor?
<sub>slajdy–merged.pdf, slajd 152</sub>

Wykład kończy się pytaniem: **czy przy pomocy mechanizmów synchronizacji w Javie da się zbudować monitor?** Slajd nie podaje odpowiedzi.

> [!warning] Odpowiedzi nie ma na slajdach
> Poniższe rozważenie **zrekonstruowano** z materiału slajdów 141–151, nie jest cytatem z prezentacji.
>
> Java daje trzy z czterech składników monitora: **hermetyzację danych** (pola prywatne), **wzajemne wykluczanie** (`synchronized` na `this`) i **oczekiwanie warunkowe** (`wait`/`notify`). Brakuje czwartego — **wielu nazwanych zmiennych warunkowych**: każdy obiekt ma **dokładnie jedną** kolejkę oczekujących, więc wątki czekające na różne warunki (np. „bufor niepusty" i „bufor niepełny") trafiają do tej samej kolejki. Stąd konieczność `notifyAll()` zamiast `notify()` i sprawdzania warunku **w pętli**, a nie w `if`.
>
> Odpowiedź brzmi więc: **monitor da się zbudować, ale nie da się go zbudować wydajnie ani bezpiecznie samymi mechanizmami niskopoziomowymi**. Brakujący składnik dostarcza dopiero para `Lock`/`Condition` z [[NPR 07 Pakiet java.util.concurrent]], gdzie jeden zamek może mieć **wiele** zmiennych warunkowych. Ada rozwiązuje ten sam problem inaczej — **barierami** deklarowanymi przy wejściach obiektu chronionego (zob. [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione#Wejścia obiektu chronionego i bariery|NPR 05]]).

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 115–152
> - **Odpowiedź na pytanie ze slajdu 152** — zob. blok `> [!warning]` powyżej.
> - **Model pamięci Javy** (JMM), relacja _happens-before_, `volatile` — slajd 140 mówi o „gwarancji dostępu do najświeższych wartości", ale mechanizm tej gwarancji omówiony jest dopiero przy `volatile` w [[NPR 07 Pakiet java.util.concurrent]].
> - **Konieczność sprawdzania warunku w pętli `while`, a nie `if`** (problem *spurious wakeup* i wyścigu po obudzeniu) — slajd 151 pokazuje schemat, z którego to wynika, ale nie formułuje reguły.
> - **`Thread.State` i nowoczesny diagram stanów** — slajd 124 pokazuje diagram z `suspend`/`resume`, czyli metodami wycofanymi (_deprecated_) od Javy 1.2, podobnie jak `stop()`.
> - **`ThreadLocal`** — nie występuje.
> - **Zmienne warunkowe w pthreads** (`pthread_cond_wait`/`signal`) — zagadnienie 9 obejmuje także pthreads, których prezentacje tego przedmiotu **w ogóle nie omawiają**; notatka [[Narzędzia Przetwarzania Rozproszonego/Zmienne Warunkowe]] jest pusta.
> - **MPI i OpenMP** — wymienione w liście zagadnienia 9, nieobecne w prezentacjach.
