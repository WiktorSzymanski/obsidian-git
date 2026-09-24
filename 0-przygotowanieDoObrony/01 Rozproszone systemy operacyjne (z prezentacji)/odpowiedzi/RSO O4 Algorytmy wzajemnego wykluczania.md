---
tags:
  - obrona
  - odpowiedź
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 4
źródło: "[[RSO Z4 Algorytmy wzajemnego wykluczania]]"
---
# 4. Algorytmy wzajemnego wykluczania
---
> Wypowiedź na obronę. Sekcja **Odpowiedź** to wersja na 2–3 minuty; **Rozwinięcia** to dalsze ciągi tej samej wypowiedzi — każde odpowiada jednemu pogrubionemu hasłu i można je wpleść w to miejsce. Całość czytana ciągiem to ok. 8 minut. Przebieg algorytmów krok po kroku: [[RSO Z4 Algorytmy wzajemnego wykluczania]].

## Odpowiedź

Wzajemne wykluczanie to zapewnienie procesom ochrony przy dostępie do zasobu: gwarancja, że w danej chwili **co najwyżej jeden proces przebywa w sekcji krytycznej**. Problem sam w sobie jest stary i znany z systemów scentralizowanych, natomiast w systemie rozproszonym zmienia się to, czym dysponujemy: nie ma ani wspólnej pamięci, ani wspólnego zegara, więc jedynym narzędziem koordynacji jest **wymiana komunikatów**. I stąd bierze się cała różnorodność rozwiązań — każde z nich to inna odpowiedź na pytanie, kogo trzeba zapytać o zgodę i ile to kosztuje komunikatów.

Od algorytmu wymagamy trzech rzeczy: **bezpieczeństwa**, czyli żeby naprawdę nie wpuścił dwóch procesów naraz, **żywotności**, czyli braku zakleszczeń i zagłodzeń, oraz **uczciwości**, czyli obsługi żądań w kolejności zgłoszenia. Ocenia się je przede wszystkim **złożonością komunikacyjną** i **opóźnieniem synchronizacji**.

Algorytmy dzielą się na dwie rodziny. **Oparte na zezwoleniach**, gdzie proces wchodzi do sekcji po uzyskaniu zgody wszystkich albo kworum — tu należą Lamport, Ricart-Agrawala i Maekawa. I **oparte na żetonie**, gdzie do sekcji wchodzi po prostu posiadacz jedynego w systemie żetonu — Suzuki-Kasami i Raymond. Pojęciem porządkującym pierwszą rodzinę jest **zbiór żądań**, czyli zbiór procesów, od których trzeba uzyskać pozwolenie: u Lamporta i Ricarta-Agrawali to wszystkie procesy, u Maekawy — kworum, a w algorytmach żetonowych zastępuje go posiadanie żetonu.

Osobno stoi **podejście scentralizowane** z koordynatorem i kolejką żądań — najprostsze i najtańsze, bo trzy komunikaty na wejście, ale koordynator jest pojedynczym punktem awarii i wąskim gardłem.

Z rozwiązań rozproszonych punktem wyjścia jest **algorytm Lamporta**, oparty na zegarze skalarnym i kolejkach żądań u każdego procesu, kosztujący $3(N-1)$ komunikatów. **Ricart-Agrawala** zbija ten koszt do $2(N-1)$, łącząc zwolnienie z odpowiedzią. **Maekawa** schodzi do rzędu pierwiastka z $N$, pytając tylko kworum. **Suzuki-Kasami** to sztandarowy algorytm żetonowy — zero komunikatów, jeśli proces już ma żeton, i $N$ w przeciwnym razie. A **Raymond** organizuje procesy w drzewo, dzięki czemu każdy zna tylko kierunek do żetonu i koszt spada do rzędu logarytmu.

Wszystkie te algorytmy zakładają niezawodne kanały i **brak awarii procesów** — i to jest ich wspólna, najpoważniejsza słabość.

## Rozwinięcia

### Wymagania i miary oceny

Warto rozwinąć te trzy wymagania. **Bezpieczeństwo** to własność typu „nic złego się nie zdarzy" — w każdej chwili w sekcji krytycznej jest co najwyżej jeden proces. **Żywotność** to własność typu „coś dobrego w końcu nastąpi" — każde zgłoszone żądanie zostanie w końcu obsłużone, a więc system ani się nie zakleszcza, ani nikogo nie zagładza. **Uczciwość** idzie dalej i wymaga obsługi w kolejności zgłoszeń, zwykle ustalanej według znaczników czasowych. Miar oceny jest cztery: **złożoność komunikacyjna**, czyli liczba komunikatów przypadających na jedno wejście do sekcji, **opóźnienie synchronizacji**, czyli czas od wyjścia jednego procesu do wejścia następnego, a ponadto czas odpowiedzi i przepustowość. Pierwsze dwie są w porównaniach najważniejsze, bo pokazują dwa różne koszty: obciążenie sieci i bezczynność zasobu.

### Podejście scentralizowane

Tutaj jeden wyróżniony proces pełni rolę **koordynatora** i utrzymuje kolejkę żądań. Proces chcący wejść do sekcji wysyła żądanie; jeżeli sekcja jest wolna, dostaje pozwolenie, a jeżeli zajęta — jego żądanie zostaje zakolejkowane albo dostaje odmowę. Po wyjściu proces wysyła zwolnienie, a koordynator przekazuje pozwolenie następnemu z kolejki. To kosztuje trzy komunikaty na wejście niezależnie od liczby procesów, więc pod względem złożoności jest nie do pobicia. Ma jednak dwie wady: koordynator jest pojedynczym punktem awarii i wąskim gardłem, a poza tym samo istnienie koordynatora jest tu założone — jego wyłonienie to osobne zagadnienie, czyli [[RSO Z5 Algorytmy elekcji|elekcja]].

### Lamport i Ricart-Agrawala

**Algorytm Lamporta** realizuje wzajemne wykluczanie bez koordynatora, opierając się na **zegarze skalarnym**. Każdy proces utrzymuje własną kolejkę żądań uszeregowaną według pary: znacznik czasowy i identyfikator. Istotą algorytmu są dwa warunki wejścia sprawdzane naraz: własne żądanie musi być na czele kolejki, co daje uporządkowanie, oraz od każdego innego procesu musi nadejść odpowiedź ze znacznikiem większym niż znacznik żądania — to drugie daje pewność, że nikt nie zgłosi już żądania starszego. Wyjście polega na rozesłaniu komunikatu zwalniającego. Algorytm wymaga **kanałów FIFO** i kosztuje $3(N-1)$ komunikatów: żądanie, odpowiedź i zwolnienie do każdego. **Ricart-Agrawala** to jego optymalizacja, która atakuje właśnie tę nadmiarowość, łącząc zwolnienie z odpowiedzią: proces, który dostaje żądanie, odpowiada natychmiast, jeśli sam nie ubiega się o sekcję; odracza odpowiedź, jeśli w sekcji siedzi; a jeśli też się ubiega — rozstrzyga porównaniem znaczników. Wejście następuje po zebraniu wszystkich odpowiedzi, a wyjście polega na wysłaniu tych odroczonych. Koszt spada do $2(N-1)$ i znika wymóg kanałów FIFO, ale awaria dowolnego procesu blokuje wszystkich.

### Maekawa i kwora

Maekawa atakuje problem **liniowego kosztu komunikacyjnego**: zamiast zgody wszystkich żąda zgody **kworum**. Kwora muszą spełniać cztery własności: każde dwa mają niepuste przecięcie — i to właśnie ten wspólny proces gwarantuje bezpieczeństwo, bo nie wyda zgody dwóm naraz; proces należy do własnego kworum; wszystkie kwora są równoliczne; i każdy proces należy do tylu samo kworów. Z tych warunków wychodzi rozmiar kworum rzędu **pierwiastka z $N$** — praktycznie buduje się je jako wiersz i kolumnę w kwadratowej siatce procesów. Wersja podstawowa ma jednak wadę: może się **zakleszczyć**, bo procesy zbierają zgody w różnej kolejności i każdy czeka na zgodę trzymaną przez innego. Rozwiązanie dokłada trzy komunikaty — FAILED, czyli „twoje żądanie ma niższy priorytet", INQUIRE, czyli „czy na pewno możesz wejść", oraz RELINQUISH, czyli „oddaję zgodę" — dzięki czemu proces bez szans na komplet zgód oddaje zgodę procesowi o wyższym priorytecie. Koszt wynosi wtedy od trzech do pięciu pierwiastków z $N$.

### Suzuki-Kasami

To algorytm żetonowy: proces posiadający żeton może wchodzić do sekcji dopóty, dopóki nie poprosi o niego ktoś inny. Cała trudność sprowadza się do dwóch problemów, które warto nazwać wprost. Pierwszy to **żądania przedawnione**: żądanie niesie numer mówiący, że proces ubiega się o swoje $n$-te wykonanie sekcji, a każdy proces trzyma tablicę największych numerów żądań otrzymanych od każdego z pozostałych — jeżeli przychodzi numer mniejszy niż zapamiętany, żądanie jest przedawnione i się je ignoruje. Drugi to **żądania zaległe**, i ten rozstrzyga zawartość samego żetonu: żeton niesie kolejkę procesów oczekujących oraz tablicę numerów żądań **ostatnio zrealizowanych**. Kluczem jest zestawienie obu tablic: jeżeli numer żądania zgłoszonego przez dany proces jest o jeden większy od numeru żądania, które ten proces zrealizował, to znaczy, że ma on **oczekujące, nieobsłużone żądanie**. Ten sam warunek służy do uzupełniania kolejki w żetonie i do przekazania żetonu nieużywanego. Koszt to zero komunikatów, gdy proces już ma żeton, albo $N$ — czyli $N-1$ żądań plus przesłanie żetonu.

### Raymond

Raymond rozwiązuje problem, który w Suzuki-Kasami pozostaje: **rozgłaszanie żądań do wszystkich**. Procesy organizuje się w logiczne **drzewo rozpinające**, w którym każdy proces zna wyłącznie kierunek do posiadacza żetonu — jedną zmienną wskazującą sąsiada — i trzyma kolejkę FIFO żądających sąsiadów. Żądania wędrują po krawędziach drzewa w stronę żetonu, a żeton wraca tą samą drogą, obsługując po kolei zakolejkowanych. Wiedza każdego procesu jest więc wyłącznie **lokalna**, a koszt spada do rzędu logarytmu z $N$ w drzewie zrównoważonym, w najgorszym razie do średnicy drzewa.

### Zestawienie i problem awarii

Zestawiając: scentralizowany kosztuje stale trzy komunikaty, Lamport $3(N-1)$, Ricart-Agrawala $2(N-1)$, Maekawa rząd pierwiastka z $N$, Suzuki-Kasami zero albo $N$, Raymond rząd logarytmu. Widać wyraźną oś: **im mniej procesów trzeba zapytać, tym tańszy algorytm, ale tym bardziej złożona struktura, którą trzeba utrzymywać** — od braku struktury u Lamporta, przez kwora, po drzewo u Raymonda. Trzeba jednak na koniec powiedzieć, że wszystkie te algorytmy zakładają niezawodne kanały i brak awarii: utrata żetonu albo milczący proces blokują cały system. Rozszerzenia polegają na wykrywaniu utraty żetonu i jego **regeneracji**, zwykle uzgadnianej przez elekcję, na timeoutach oraz na kworach odpornych na awarie.
