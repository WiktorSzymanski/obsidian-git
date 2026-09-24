---
tags:
  - obrona
  - odpowiedź
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 9
źródło: "[[NPR 09 Wielozadaniowość i synchronizacja zadań i wątków]]"
---
# 9. Wielozadaniowość i synchronizacja zadań/wątków
---
> Wypowiedź na obronę. Sekcja **Odpowiedź** to wersja na 2–3 minuty; **Rozwinięcia** to dalsze ciągi tej samej wypowiedzi — każde odpowiada jednemu pogrubionemu hasłu i można je wpleść w to miejsce. Całość czytana ciągiem to ok. 10 minut — to najobszerniejsze z zagadnień tego przedmiotu. Pełne zestawienie Java ↔ Ada: [[NPR 09 Wielozadaniowość i synchronizacja zadań i wątków]].

## Odpowiedź

Zacznę od tego, że **synchronizacja to w istocie dwie różne rzeczy**. Pierwsza to synchronizacja wątków, czyli koordynacja realizacji kolejnych instrukcji, kroków i faz — kontrola **przepływu sterowania**. Druga to synchronizacja danych, czyli utrzymanie ich spójności i gwarancja, że czytający dostanie najświeższą wartość — kontrola **przepływu danych**. Większość mechanizmów obsługuje tylko jeden z tych wymiarów: `volatile` wyłącznie drugi, bariera wyłącznie pierwszy, a kolejka blokująca oba.

Druga rzecz porządkująca to **poziom, na którym mechanizm żyje**. Na poziomie **architektury** mamy zapis i odczyt współdzielonych rejestrów oraz operacje złożone realizowane niepodzielnie, jak `test&set`. Na poziomie **systemu operacyjnego** — semafory, zamki i zmienne warunkowe, zintegrowane z zarządzaniem stanem wątku i przydziałem procesora. Na poziomie **języka** — konstrukcje strukturalne, czyli monitory i regiony krytyczne.

I ten właśnie podział wyznacza główną różnicę między dwoma narzędziami omawianymi w przedmiocie. **Ada dostarcza mechanizmów na poziomie języka**: zadanie, wejście i obiekt chroniony to konstrukcje składniowe, które sprawdza kompilator. **Java dostarcza ich na poziomie systemu**: `Thread`, `Semaphore` czy `Lock` to zwykłe klasy biblioteki, a jedynymi konstrukcjami językowymi są `synchronized` i `volatile`.

Z tego wynika cała reszta. W Javie trzeba osobno zadbać o **atomowość**, osobno o **wzajemne wykluczanie** i osobno o **oczekiwanie warunkowe** — a każdy z tych obszarów ma swoje pułapki, z których najważniejsza to **zgubiony sygnał** przy `wait` i `notify`. W Adzie wzajemne wykluczanie jest własnością obiektu chronionego i nie wymaga żadnej instrukcji, a oczekiwanie warunkowe jest **deklaratywne** — programista zapisuje warunek, a nie to, gdzie zasnąć i kogo obudzić.

Ada ma też mechanizm, którego Java nie ma w ogóle: **spotkanie asymetryczne** wraz z instrukcją `select`, czyli oczekiwaniem selektywnym. Java odpowiada na to szerokim zestawem **gotowych mechanizmów** z pakietu współbieżności oraz **pulami wątków**.

Ogólna prawidłowość jest taka: **Ada daje mniej mechanizmów, ale trudniej się nimi pomylić**, bo warunki poprawności sprawdza kompilator; **Java daje ich znacznie więcej**, ale poprawność spoczywa na programiście.

## Rozwinięcia

### Atomowość i pułapki Javy

Operacja jest **atomowa**, gdy spełnia dwa warunki: **niepodzielność**, czyli albo wykona się w całości, albo w ogóle się nie rozpocznie, oraz **izolację**, czyli żadne jej efekty nie ujawniają się przed zakończeniem. W Javie atomowość jest gwarantowana dla zapisu i odczytu typów prostych oraz referencji, ale z dwoma wyjątkami, o które zwykle się pyta. Pierwszy: **`long` i `double`** — ich 64-bitowy zapis może się rozpaść na dwie połowy, i to naprawia `volatile`. Drugi: **operacje pozornie atomowe** jak inkrementacja czy `+=`, które w rzeczywistości są sekwencją odczyt-modyfikacja-zapis; tu pomagają klasy atomowe z pakietu współbieżności. Warto od razu powiedzieć, czym `volatile` **nie** jest: gwarantuje, że czytający dostanie ostatnią zapisaną wartość, i wypycha wcześniejsze zapisy nieulotne tego wątku, ale **nie daje wzajemnego wykluczania** — jest barierą pamięci, a nie zamkiem.

### Wzajemne wykluczanie w Javie

Podstawą jest `synchronized`, jako blok albo jako metoda. Blok zajmuje zamek związany integralnie ze wskazanym obiektem, a metoda — zamek obiektu, dla którego ją wywołano, i wyklucza się ze wszystkimi innymi metodami i blokami synchronizowanymi na tym samym obiekcie. Zamek jest **wielowejściowy**, więc ten sam wątek może go zająć wielokrotnie bez zakleszczenia siebie samego. Nie chroni to jednak przed zakleszczeniem dwóch wątków na dwóch obiektach, gdy każdy trzyma jeden zamek i czeka na drugi — to zwykłe oczekiwanie cykliczne. Interfejs **`Lock`** daje cztery rzeczy, których `synchronized` nie potrafi: **przerywalne oczekiwanie**, **nieblokującą próbę z limitem czasu**, zajęcie zamka w jednej metodzie i zwolnienie w innej, a przede wszystkim **wiele zmiennych warunkowych na jeden zamek**. Osobno jest **`ReadWriteLock`**, który rozdziela blokadę współdzieloną do odczytu od wyłącznej do zapisu.

### Obiekty chronione Ady

W Adzie wzajemne wykluczanie nie jest czynnością, tylko **własnością obiektu**. Obiekt chroniony grupuje dane współdzielone w części prywatnej i udostępnia je wyłącznie przez trzy rodzaje operacji publicznych: **funkcje**, które tylko czytają, oraz **procedury** i **wejścia**, które modyfikują. Z obiektem związane są dwie blokady — współdzielona przy wywołaniu funkcji i wyłączna przy wywołaniu procedury lub wejścia. To jest dokładnie ten sam zamek czytelników i pisarzy co `ReadWriteLock` w Javie, z jedną zasadniczą różnicą: **wybór blokady jest automatyczny**, bo wynika z rodzaju wywołanej operacji, a kompilator pilnuje, żeby funkcja niczego nie modyfikowała. Programista nie może więc pomylić blokady, bo jej nie wybiera.

### Oczekiwanie warunkowe: sygnał kontra warunek

W Javie niskopoziomowej służą do tego `wait`, `notify` i `notifyAll`, wywoływane obowiązkowo wewnątrz sekcji synchronizowanej na tym samym obiekcie — bo `wait` **zwalnia zamek** na czas czekania i odzyskuje go przed powrotem. Mechanizm ma dwie pułapki. Pierwsza to **zgubiony sygnał**: powiadomienie wykonane, gdy nikt nie czeka, po prostu przepada, a wątek, który zaśnie chwilę później, będzie czekał w nieskończoność. Druga to **jedna kolejka oczekujących na obiekt**: wątki czekające na różne warunki trafiają razem, stąd konieczność budzenia wszystkich i sprawdzania warunku w pętli. Dlatego trzeba powiedzieć wprost, że **z samych mechanizmów niskopoziomowych nie da się zbudować porządnego monitora** — Java ma hermetyzację, wzajemne wykluczanie i oczekiwanie warunkowe, ale brakuje jej wielu nazwanych zmiennych warunkowych; dostarcza ich dopiero para `Lock` i `Condition`. W Adzie problem nie istnieje, bo z każdym wejściem związana jest **bariera**, czyli warunek logiczny, który musi być spełniony, żeby treść wejścia się wykonała. Programista zapisuje *warunek*, a nie *gdzie zasnąć i kogo obudzić*, więc nie da się zgubić sygnału ani zapomnieć o powiadomieniu. Kluczowe jest przy tym, że zadanie czekające na barierę **nie trzyma blokady**, dzięki czemu inne zadania mogą wejść i zmienić stan tak, by bariera się otworzyła. A gdy warunek zależy od wartości parametru, której bariera nie widzi, zadanie wpuszcza się i przekierowuje instrukcją **`requeue`** do kolejki innego wejścia.

### Gotowe mechanizmy Javy

**Semafor** przechowuje pulę jednostek, które się pobiera i oddaje; w odróżnieniu od zmiennej warunkowej semafor **pamięta** jednostki, więc oddanie przed pobraniem nie przepada. **Bariery** są trzy: cykliczna zwalnia ustaloną liczbę wątków dopiero wtedy, gdy wszystkie się zgłoszą, i wraca potem do stanu początkowego; **zatrzask** rozdziela zgłoszenie od czekania, przez co zgłaszający nie musi czekać, ale jest jednorazowy; a `Phaser` uogólnia obie i pozwala **zmieniać liczbę uczestników w czasie**. Z kolekcji współbieżnych najważniejsza jest **kolejka blokująca**, czyli gotowe rozwiązanie problemu ograniczonego buforowania; obok niej `TransferQueue` i `Exchanger`, które są najbliższymi odpowiednikami spotkania Ady. Osobnym, odwrotnym podejściem są **obiekty niezmienne**: obiekt, którego stan po skonstruowaniu się nie zmienia, nie wymaga synchronizacji w ogóle — wymaga za to braku metod modyfikujących, pól prywatnych i finalnych, zabezpieczenia przed redefinicją w podklasach oraz, co najczęściej się pomija, **kopiowania zamiast udostępniania referencji** do obiektów modyfikowalnych.

### Zarządzanie wykonaniem: wątek Javy, zadanie Ady

**Wątek w Javie** jest obiektem klasy `Thread`, a jego programem głównym metoda `run` — albo przesłonięta w klasie pochodnej, albo dostarczona przez obiekt implementujący `Runnable`; ten drugi wariant jest praktyczniejszy, bo Java nie ma wielodziedziczenia. Podstawowe operacje to uruchomienie, oczekiwanie na zakończenie innego wątku, oddanie procesora i uśpienie. Osobno trzeba powiedzieć o **przerwaniu**: nie zatrzymuje ono wątku, tylko wytrąca go z czekania wyjątkiem albo ustawia flagę — i jest to **jedyny bezpieczny sposób kończenia wątków**, w odróżnieniu od wycofanych metod zatrzymania i zawieszenia, z których ta druga nie zwalnia blokad i potrafi zakleszczyć program. Wątek **demon** kończy się automatycznie po zakończeniu ostatniego wątku użytkownika. **Zadanie w Adzie** jest natomiast nie obiektem biblioteki, tylko **jednostką strukturalizacji programu** — bytem językowym z własną składnią specyfikacji i treści. Zadanie zadeklarowane w bloku startuje automatycznie razem z nim, a deklaracja tablicy zadań tworzy od razu kilka równolegle działających zadań, bez żadnej pętli. Kończy się samo albo wybiera gałąź `terminate`, gdy jednostka macierzysta się zakończyła i nikt już nie może wywołać jego wejść — co jest precyzyjniejszym odpowiednikiem demona.

### Pule wątków

Po stronie Javy dochodzi jeszcze warstwa, której w materiale o Adzie nie ma: **pule wątków**, czyli interfejs `Executor` i jego implementacja konfigurowana rozmiarem podstawowym, rozmiarem maksymalnym, czasem życia nadmiarowych wątków i kolejką zadań. Reguła przydziału jest nieoczywista i warto ją umieć powiedzieć: dopóki nie przekroczono rozmiaru podstawowego, **każde zadanie dostaje nowy wątek**; potem zadania **najpierw trafiają do kolejki**, a dopiero po jej przepełnieniu tworzone są dodatkowe wątki aż do rozmiaru maksymalnego; kolejne są odrzucane wyjątkiem. Nadbudowa `ExecutorService` dokłada kontrolę nad zamykaniem puli i zwraca **`Future`**, czyli uchwyt do wyniku zadania asynchronicznego, z operacjami pobrania wyniku, anulowania i sprawdzenia gotowości. Zadania zwracające wynik opisuje `Callable`, którego metoda — w przeciwieństwie do `run` — zwraca wartość i może zgłosić wyjątek.

### Spotkanie i oczekiwanie selektywne

**Spotkanie asymetryczne** to komunikacja synchroniczna między parą zadań: klient woła wejście serwera, serwer przyjmuje wywołanie instrukcją `accept`, a ta ze stron, która dotrze pierwsza, czeka na drugą. Komunikacja i synchronizacja są tym samym zdarzeniem, a tylko ciało `accept` wykonuje się przy wstrzymanym kliencie. Obudowuje to instrukcja **`select`**, która ma cztery zastosowania. Po stronie serwera daje **oczekiwanie selektywne**: alternatywę wielu gałęzi `accept`, gałęzie **dozorowane** warunkami logicznymi, gałąź opóźnienia dającą ograniczone czekanie, gałąź `else` wycofującą ofertę, gdy nikt nie czeka, oraz gałąź `terminate`. Dwie rzeczy warto tu zaznaczyć: **dozory oblicza się raz, na początku** wykonania `select`, więc zmiana stanu w trakcie czekania staje się widoczna dopiero przy następnym obrocie pętli, a gdy wszystkie dozory są fałszywe, zgłaszany jest błąd; oraz że kolejka pojedynczego wejścia jest FIFO, ale **wybór między wejściami jest niedeterministyczny**. Po stronie klienta ten sam `select` daje wywołanie terminowe i warunkowe, czyli te same gałęzie w lustrzanym odbiciu. Czwarte zastosowanie, `select … then abort`, realizuje **asynchroniczną zmianę wątku sterowania**, przerywając trwające obliczenie — i to jest mechanizm wywłaszczający, podczas gdy przerwanie w Javie wymaga współpracy przerywanego wątku.

### Pointa zestawienia

Zestawiając oba narzędzia: w Javie start wątku jest jawny, w Adzie automatyczny; wzajemne wykluczanie w Javie trzeba napisać, w Adzie jest własnością obiektu; wybór blokady czytelników i pisarzy w Javie jest ręczny, w Adzie automatyczny; oczekiwanie warunkowe w Javie opiera się na sygnale, który można zgubić, w Adzie na warunku, którego zgubić się nie da. Za tym stoi jedna decyzja projektowa: **Ada przenosi warunki poprawności do kompilatora i środowiska wykonawczego, Java zostawia je programiście** — i stąd bierze się cała rodzina jej pułapek, od 64-bitowego zapisu, przez inkrementację, po zgubiony sygnał i kolejność kroków przy przydziale zadań w puli.
