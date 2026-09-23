---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 9
---
# 9. Wielozadaniowość i synchronizacja zadań/wątków
---
> Komplet pojęć zagadnienia w zakresie pokrytym przez prezentacje — czyli **Java** i **Ada 95**. Mechanizmy są nazwane i porównane; szczegóły API i kod — w [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione|NPR 05]], [[NPR 06 Wątki w Javie i elementarna synchronizacja|NPR 06]] i [[NPR 07 Pakiet java.util.concurrent|NPR 07]].

## Co synchronizujemy i na jakim poziomie

Synchronizacja rozpada się na dwie rzeczy. **Synchronizacja procesów/wątków** to koordynacja realizacji poszczególnych instrukcji, kroków i faz — kontrola **przepływu sterowania**. **Synchronizacja danych** to utrzymanie ich spójności i gwarancja, że czytający dostanie **najświeższą** wartość — kontrola **przepływu danych**. Większość mechanizmów obsługuje jeden z tych wymiarów; `volatile` wyłącznie drugi, bariera wyłącznie pierwszy, kolejka blokująca oba.

Mechanizmy występują na trzech poziomach. Na **poziomie architektury systemu komputerowego** są to zapis i odczyt współdzielonych rejestrów oraz **operacje złożone realizowane niepodzielnie**, jak `test&set` czy `exchange`. Na **poziomie systemu operacyjnego** — mechanizmy zintegrowane z zarządzaniem stanem wątku i przydziałem procesora: **semafory, zamki, zmienne warunkowe**. Na **poziomie języka programowania** — **strukturalne** konstrukcje wyrażające zależności i ograniczenia w dostępie do współdzielonych zasobów: **monitory i regiony krytyczne**.

Ten podział wyznacza główną różnicę między dwoma omawianymi narzędziami. **Ada dostarcza mechanizmów na poziomie języka** — zadanie, wejście i obiekt chroniony są konstrukcjami składniowymi, które sprawdza kompilator. **Java dostarcza ich na poziomie systemu** — `Thread`, `Semaphore` czy `Lock` są zwykłymi klasami biblioteki, a jedynymi konstrukcjami językowymi są `synchronized` i `volatile`.

## Atomowość

Operacja jest **atomowa**, gdy jest realizowana **w całości** — albo się zakończy, albo w ogóle nie rozpocznie (**niepodzielność**) — i **w „momencie" czasu**, to znaczy żadne jej efekty nie ujawniają się przed zakończeniem (**izolacja**).

W Javie atomowość jest gwarantowana dla zapisu i odczytu typów prostych oraz referencji, ale **nie dla `long` i `double`**, których 64-bitowy zapis może się rozpaść na dwie połowy. Nie jest też atomowa żadna z **pozornie atomowych** operacji `++`, `--`, `+=`, `-=`, które są w istocie sekwencją odczyt-modyfikacja-zapis. Pierwszy problem usuwa `volatile`, drugi — klasy z pakietu `java.util.concurrent.atomic`.

**`volatile`** gwarantuje, że czytający otrzyma ostatnią zapisaną wartość, **nie daje wzajemnego wykluczania**, a zapis zmiennej ulotnej dodatkowo **wypycha wcześniejsze zapisy nieulotne** tego wątku. Jest więc barierą pamięci, a nie zamkiem.

## Wzajemne wykluczanie

W **Javie** realizuje je `synchronized` — blok albo metoda. Blok `synchronized(obj)` zajmuje zamek związany integralnie z obiektem `obj`; metoda `synchronized` zajmuje zamek obiektu, dla którego została wywołana, i wyklucza się ze wszystkimi innymi metodami i blokami `synchronized` na tym samym obiekcie. Zamek jest **wielowejściowy** (*reentrant*): ten sam wątek może go zająć wielokrotnie bez zakleszczenia, a zwolnienie następuje po wyjściu z ostatniej sekcji. Nie chroni to jednak przed zakleszczeniem **dwóch wątków na dwóch obiektach**, gdy każdy trzyma jeden zamek i czeka na drugi — to klasyczne oczekiwanie cykliczne.

Interfejs **`Lock`** daje to samo, czego `synchronized` nie potrafi: przerywalne oczekiwanie (`lockInterruptibly`), nieblokującą próbę z limitem czasu (`tryLock`), zajęcie zamka w jednej metodzie i zwolnienie w innej oraz — co najważniejsze — **wiele zmiennych warunkowych na jeden zamek** (`newCondition`). **`ReadWriteLock`** rozdziela blokadę współdzieloną (`readLock`) od wyłącznej (`writeLock`).

W **Adzie** wzajemne wykluczanie jest własnością **obiektu chronionego** i nie wymaga żadnej instrukcji. Obiekt grupuje dane współdzielone w części prywatnej i udostępnia je wyłącznie przez publiczne **funkcje** (tylko odczyt), **procedury** i **wejścia** (modyfikacja). Z obiektem związane są **dwie blokady**: *shared read lock* przy wywołaniu funkcji i *exclusive read/write lock* przy wywołaniu procedury lub wejścia. Jest to ten sam zamek czytelników i pisarzy co `ReadWriteLock`, z tą różnicą, że **wybór blokady jest automatyczny** — wynika z rodzaju wywołanej operacji, a kompilator pilnuje, żeby funkcja nie modyfikowała stanu.

## Oczekiwanie warunkowe

W **Javie niskopoziomowej** służą do tego `wait()`, `notify()` i `notifyAll()`, wywoływane obowiązkowo wewnątrz sekcji `synchronized` na tym samym obiekcie — bo `wait()` **zwalnia zamek** na czas czekania i odzyskuje go przed powrotem. Mechanizm ma dwie pułapki. Po pierwsze, **sygnał nie jest pamiętany**: `notify()` wykonane, gdy nikt nie czeka, przepada, a wątek, który zaśnie chwilę później, będzie czekał w nieskończoność (**zgubiony sygnał**). Po drugie, każdy obiekt ma **tylko jedną kolejkę oczekujących**, więc wątki czekające na różne warunki trafiają razem — stąd konieczność `notifyAll()` i sprawdzania warunku w pętli.

Dlatego **z samych mechanizmów niskopoziomowych nie da się zbudować porządnego monitora**. Java ma hermetyzację danych, wzajemne wykluczanie i oczekiwanie warunkowe, ale brakuje jej **wielu nazwanych zmiennych warunkowych**. Dostarcza ich dopiero para `Lock`/`Condition`, gdzie jeden zamek może mieć wiele niezależnych kolejek (`await`, `signal`, `signalAll`).

W **Adzie** oczekiwanie warunkowe jest **deklaratywne**. Z każdym wejściem obiektu chronionego związana jest **bariera** — warunek logiczny, który musi być spełniony, żeby treść wejścia się wykonała. Programista zapisuje *warunek*, a nie *gdzie zasnąć i kogo obudzić*, więc nie da się zgubić sygnału ani zapomnieć o powiadomieniu. Zasadnicze jest przy tym, że zadanie czekające na barierę **nie trzyma blokady** — dzięki czemu inne zadania mogą wejść i zmienić stan tak, by bariera się otworzyła. Gdy warunek zależy od wartości parametru, której bariera nie widzi, zadanie wpuszcza się i przekierowuje instrukcją **`requeue`** do kolejki innego wejścia, zwykle prywatnego.

## Mechanizmy gotowe

**Semafor** (`Semaphore`) przechowuje pulę jednostek; `acquire` je pobiera, `release` oddaje, parametr `fair` gwarantuje FIFO. W odróżnieniu od zmiennej warunkowej semafor **pamięta** jednostki, więc `release` przed `acquire` nie przepada.

**Bariery**. `CyclicBarrier` zwalnia ustaloną liczbę wątków dopiero, gdy wszystkie wywołają `await`, po czym wraca do stanu początkowego; może wykonać zadanie `barrierAction` między przełamaniem a zwolnieniem wątków. `CountDownLatch` rozdziela zgłoszenie (`countDown`) od czekania (`await`), przez co zgłaszający nie musi czekać, ale jest **jednorazowy**. `Phaser` uogólnia obie: rozdziela `arrive` od `awaitAdvance` i pozwala **zmieniać liczbę uczestników** w czasie (`register`/`deregister`).

**Kolekcje współbieżne**. `BlockingQueue` blokuje przy pobraniu z pustej i przy wstawieniu do pełnej kolejki — gotowe rozwiązanie problemu ograniczonego buforowania. `TransferQueue` blokuje producenta aż do pobrania elementu, a `Exchanger` realizuje synchroniczną wymianę danych między dwoma wątkami; obie są bliskimi odpowiednikami **spotkania** Ady. `ConcurrentMap` daje atomowe wstawianie, zastępowanie i usuwanie.

**Obiekty niezmienne** są podejściem odwrotnym: obiekt, którego stan po skonstruowaniu się nie zmienia, **nie wymaga synchronizacji w ogóle**. Strategia to brak metod modyfikujących, pola `private final`, zabezpieczenie przed redefinicją w podklasach (klasa `final` lub prywatny konstruktor i fabryki) oraz — najczęściej pomijane — zabezpieczenie przed zmianą obiektów modyfikowalnych, do których obiekt trzyma referencje, przez **kopiowanie zamiast udostępniania referencji**.

## Zarządzanie wykonaniem

**Wątek w Javie** jest obiektem klasy `Thread`; jego programem głównym jest metoda `run()` — albo przesłonięta w klasie pochodnej od `Thread`, albo dostarczona przez obiekt implementujący `Runnable` i przekazana konstruktorowi. Oba warianty to ten sam mechanizm; wariant z `Runnable` jest praktyczniejszy, bo Java nie ma wielodziedziczenia. Wątek uruchamia `start()`, czeka na inny przez `join()`, oddaje procesor przez `yield()`, usypia przez `sleep()`. **`interrupt()`** nie zatrzymuje wątku, tylko wytrąca go z czekania wyjątkiem `InterruptedException` albo ustawia flagę — to jedyny bezpieczny sposób kończenia wątków, w odróżnieniu od wycofanych `stop()` i `suspend()` (ta druga **nie zwalnia blokad**, więc potrafi zakleszczyć program). **Demon** kończy się automatycznie po zakończeniu ostatniego wątku użytkownika. **Grupy wątków** (`ThreadGroup`) pozwalają operować na zbiorze logicznie powiązanych wątków; przypisanie następuje w chwili tworzenia i jest nieodwracalne.

**Zadanie w Adzie** nie jest obiektem biblioteki, tylko **jednostką strukturalizacji programu** — bytem językowym z własną składnią specyfikacji i treści. Zadanie zadeklarowane w bloku startuje automatycznie razem z nim, a deklaracja `array(1..5) of T` tworzy pięć równolegle działających zadań bez żadnej pętli. Zadanie kończy się samo albo wybiera gałąź **`terminate`** instrukcji `select`, gdy jednostka macierzysta się zakończyła i nikt już nie może wywołać jego wejść — to precyzyjniejszy odpowiednik demona.

**Pule wątków** w Javie realizuje interfejs `Executor` i jego implementacja `ThreadPoolExecutor`, konfigurowana przez `corePoolSize`, `maximumPoolSize`, `keepAliveTime` i kolejkę `workQueue`. Reguła przydziału jest nieoczywista: dopóki nie przekroczono `corePoolSize`, każde zadanie dostaje nowy wątek; potem zadania **najpierw trafiają do kolejki**, a dopiero po jej przepełnieniu tworzone są dodatkowe wątki, aż do `maximumPoolSize`, po czym kolejne są odrzucane wyjątkiem `RejectedExecutionException`. `ExecutorService` dokłada kontrolę (`shutdown`, `shutdownNow`, `awaitTermination`) i zwraca **`Future`** — uchwyt do wyniku zadania asynchronicznego, z operacjami `get`, `cancel`, `isDone`. Zadania zwracające wynik opisuje **`Callable`**, którego `call()` — w przeciwieństwie do `run()` — zwraca wartość i może zgłosić wyjątek.

## Komunikacja synchroniczna w Adzie

Ada ma mechanizm, którego Java nie ma w ogóle: **spotkanie asymetryczne** (*rendez-vous*). Zadanie czynne (klient) woła wejście zadania biernego (serwera) zapisem `Serwer.E1(…)`, serwer przyjmuje je instrukcją `accept … do … end`, a ta ze stron, która dotrze pierwsza, **czeka na drugą**. Komunikacja i synchronizacja są tu **tym samym zdarzeniem**; tylko ciało `accept` wykonuje się przy wstrzymanym kliencie.

Instrukcja **`select`** obudowuje ten mechanizm czterema zastosowaniami. Po stronie serwera daje **oczekiwanie selektywne**: alternatywę wielu gałęzi `accept`, gałęzie **dozorowane** warunkami logicznymi, gałąź `delay` (ograniczone czekanie), gałąź `else` (wycofanie oferty, gdy nikt nie czeka) i gałąź `terminate`. Dozory oblicza się **raz, na początku** wykonania `select`, więc zmiana stanu w trakcie czekania staje się widoczna dopiero przy następnym obrocie pętli; gdy wszystkie dozory są fałszywe, zgłaszany jest `Program_Error`. Kolejka pojedynczego wejścia jest domyślnie FIFO, ale **wybór między wejściami jest niedeterministyczny**. Po stronie klienta `select` daje **terminowe** i **warunkowe wywołanie wejścia** — te same `delay` i `else` w lustrzanym odbiciu. Czwarte zastosowanie, `select … then abort`, realizuje **asynchroniczną zmianę wątku sterowania**: przerywa trwające obliczenie po zadziałaniu instrukcji wyzwalającej.

## Java a Ada — zestawienie

| | **Java** | **Ada 95** |
|---|---|---|
| **Poziom mechanizmu** | biblioteka (poza `synchronized`, `volatile`) | **język** |
| **Jednostka wykonania** | obiekt `Thread` / `Runnable` | **zadanie** (`task`), byt składniowy |
| **Start** | jawny `start()` | **automatyczny** wraz z blokiem deklaracji |
| **Wzajemne wykluczanie** | `synchronized`, `Lock` — jawne | **obiekt chroniony** — wbudowane |
| **Czytelnicy/pisarze** | `ReadWriteLock` — wybór ręczny | **automatyczny** (funkcja vs procedura/wejście) |
| **Oczekiwanie warunkowe** | `wait`/`notify` (1 kolejka), `Condition` (wiele) | **bariery** wejść — deklaratywne |
| **Zgubiony sygnał** | **możliwy** przy `wait`/`notify` | **niemożliwy** — warunek, nie sygnał |
| **Komunikacja synchroniczna para-para** | brak (najbliżej `Exchanger`, `TransferQueue`) | **spotkanie** (`entry`/`accept`) |
| **Oczekiwanie selektywne** | brak | **`select`** z dozorami |
| **Przerwanie obliczenia** | `interrupt()` — wymaga współpracy | **`select … then abort`** — wywłaszczające |
| **Zakończenie wątku służebnego** | demon | **gałąź `terminate`** |
| **Pule wątków** | `Executor`, `Future`, `Callable` | brak w materiale |

Ogólna prawidłowość: **Ada daje mniej mechanizmów, ale trudniej się nimi pomylić**, bo warunki poprawności sprawdza kompilator i środowisko wykonawcze. **Java daje ich znacznie więcej**, ale poprawność spoczywa na programiście — stąd cała rodzina pułapek: `long`/`double`, `i++`, zgubiony sygnał, jedna kolejka na obiekt, kolejność kroków w puli wątków.

---
## Czego w prezentacjach nie ma

> [!todo] Zakres zagadnienia wykracza poza slajdy
> Lista egzaminacyjna obejmuje także **pthreads** (w tym zmienne warunkowe `pthread_cond_wait`/`signal`), **MPI** i **OpenMP** — prezentacje tego przedmiotu **nie omawiają ich wcale**. Notatka [[Narzędzia Przetwarzania Rozproszonego/Zmienne Warunkowe]] w vaultcie jest pusta. Brakuje też **modelu pamięci Javy** i relacji *happens-before*, **problemu ABA** przy `compareAndSet`, **priorytetów zadań Ady** i **protokołu pułapu priorytetu**. Klasyczne problemy synchronizacji (producent-konsument, czytelnicy-pisarze, jedzący filozofowie) pojawiają się w prezentacjach wyłącznie jako **lista zadań** do samodzielnej implementacji, bez rozwiązań.
