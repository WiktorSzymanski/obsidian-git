# Prezentacja obrony – tekst do wygłoszenia

Łącznie ok. 10 minut. Na slajdach działowych (2, 4, 6, 10, 15) nie trzeba nic mówić, wystarczy je przeklikać przy przejściu do nowej części.

---

## Slajd 1 – Tytuł (~20 s)

Dzień dobry. Nazywam się Wiktor Szymański i chciałbym przedstawić moją pracę magisterską „Review and Analysis of Design Patterns in Distributed Systems”, czyli przegląd i analizę wzorców projektowych w systemach rozproszonych. Promotorką pracy jest dr hab. inż. Anna Kobusińska, prof. PP.

## Slajd 2 – Rozdział 1: Motywacja

Zacznę od motywacji.

## Slajd 3 – Pytania badawcze (~1 min)

Systemy rozproszone oparte na zdarzeniach potrzebują gwarancji, że opublikowane zdarzenie opisuje zmianę stanu, która rzeczywiście nastąpiła. Tę gwarancję dają dwa znane wzorce: Transactional Outbox i Event Sourcing. W literaturze opisuje się je zwykle osobno, rzadko jako alternatywy dla tego samego problemu, a wybór jednego zazwyczaj wyklucza drugi.

Dlatego zbudowałem ten sam system dwa razy, raz na każdym z tych wzorców, i zmierzyłem obie implementacje w jednakowych warunkach. Praca odpowiada na cztery pytania:

- **Q1:** jak mechanizm dostarczania zdarzeń z outboxa wpływa na wydajność i jakie wprowadza ograniczenia,
- **Q2:** jak snapshoty wpływają na wydajność systemu opartego na Event Sourcingu,
- **Q3:** czy cache po stronie zapisu poprawia jego wydajność,
- **Q4:** jak przestrzeganie granic agregatów wpływa na rywalizację o zasoby bazy danych.

## Slajd 4 – Rozdział 2: Omówione wzorce

## Slajd 5 – Omówione wzorce (~1 min)

W części teoretycznej omawiam cztery wzorce: ich model koncepcyjny, motywację i przykłady zastosowań w przemyśle.

- **Transactional Outbox**: źródłem prawdy jest bieżący stan. Zdarzenie trafia do tabeli outbox w tej samej transakcji co zmiana stanu, a osobny proces je publikuje.
- **Event Sourcing** odwraca tę zależność. Źródłem prawdy są zdarzenia, a bieżący stan odtwarza się przez ich ponowne odtworzenie (replay).
- **CQRS** rozdziela model zapisu od modelu odczytu. W moim systemie strona komend dopisuje zdarzenia, a zapytania są obsługiwane z projekcji.
- **Saga** koordynuje proces biznesowy obejmujący kilka agregatów. Zamiast jednej dużej transakcji wykonuje sekwencję lokalnych kroków z akcjami kompensującymi.

## Slajd 6 – Rozdział 3: System referencyjny

## Slajd 7 – Domena (~40 s)

System referencyjny to usługa rezerwacji produktów magazynowych. Ma dwa agregaty, Inventory Item i Order, oraz jeden niezmiennik: stanu magazynowego nie można przekroczyć. Zamówienie jest potwierdzane tylko wtedy, gdy uda się zarezerwować wszystkie jego pozycje. W przeciwnym razie jest odrzucane w całości. Obie implementacje mają ten sam interfejs REST, ten sam stos technologiczny i tę samą bazę PostgreSQL. Różnią się tylko wzorcem i granicami transakcji.

## Slajd 8 – Warianty Transactional Outbox (~30 s)

System z outboxem ma trzy warianty i różnią się one wyłącznie sposobem wyzwalania dostarczania zdarzeń:

- **TO-1** cyklicznie odpytuje tabelę outbox co 100 ms,
- **TO-2** reaguje na mechanizm LISTEN/NOTIFY PostgreSQL wywoływany triggerem,
- **TO-3** publikuje zdarzenie w fazie after-commit tej samej transakcji.

## Slajd 9 – Warianty Event Sourcing (~30 s)

System oparty na Event Sourcingu jest zbudowany na Axon Framework. Jego warianty różnią się sposobem odtwarzania stanu agregatu:

- **ES-1** za każdym razem odtwarza pełny strumień zdarzeń,
- **ES-2** dodaje snapshot co 100 zdarzeń,
- **ES-3** dodatkowo wprowadza cache agregatów na ścieżce zapisu.

## Slajd 10 – Rozdział 4: Testy

## Slajd 11 – Środowisko (~20 s)

Wszystkie testy uruchamiałem na tej samej maszynie, z tymi samymi limitami kontenerów: 10 CPU i 12 GiB pamięci dla usługi oraz 10 CPU i 16 GiB dla bazy danych.

## Slajd 12 – Scenariusze obciążenia (~30 s)

Użyłem trzech scenariuszy obciążenia:

- **Breakpoint**: obciążenie rośnie o 10 zamówień na sekundę co minutę, żeby znaleźć punkt, w którym system przestaje nadążać.
- **Average-load**: stałe 60 zamówień na sekundę przez godzinę, żeby sprawdzić stabilność w czasie.
- **Stress**: stałe obciążenie w okolicy punktu kolana wariantu z cache, służące do porównania ES-2 z ES-3.

Wyniki breakpoint to średnia z czterech powtórzeń, a pozostałych scenariuszy z trzech.

## Slajd 13 – Warianty obciążenia (~30 s)

Do tego doszły trzy warianty obciążenia:

- **W-base**: 100 produktów, 4 pozycje w zamówieniu.
- **W-hot**: tylko 25 produktów, więc zamówienia znacznie częściej na siebie nachodzą, co obciąża optimistic locking.
- **W-fan**: 400 produktów, ale 16 pozycji w zamówieniu, więc każde zamówienie generuje dużo więcej pracy.

## Slajd 14 – Definicja punktu kolana (~30 s)

Główną miarą przepustowości jest **punkt kolana** (knee). Dzielę przebieg testu na 30-sekundowe okna. Okno jest zrównoważone, jeśli zaległości rosną w nim o nie więcej niż 1% napływającego obciążenia. Punkt kolana to ostatnie zrównoważone okno. Biorę ostatnie zrównoważone, a nie pierwsze niezrównoważone, bo pojedyncze zaburzenie, na przykład pauza garbage collectora, nie oznacza jeszcze wyczerpania przepustowości. Punkt kolana przesuwa się dopiero wtedy, gdy deficyt staje się trwały.

## Slajd 15 – Rozdział 5: Wyniki

## Slajd 16 – Q1 (~1 min 10 s)

Zarówno NOTIFY, jak i publikacja after-commit skróciły medianę opóźnienia publikacji z 94 ms przy odpytywaniu do około 8–10 ms. Mechanizm dostarczania wpłynął jednak nie tylko na opóźnienie. Odpytywanie wypuszcza zdarzenia dużymi paczkami, a te nagłe skoki równoległego przetwarzania powodowały wiele konfliktów optimistic locking: 18,5 ponowień na sekundę w TO-1 wobec mniej niż 0,01 w TO-3.

Największym zaskoczeniem był TO-2. NOTIFY wysłany wewnątrz transakcji zakłada podczas commitu globalną blokadę wyłączną, więc wszystkie commity w systemie są serializowane. Punkt kolana spadł do 168 zamówień na sekundę, czyli do jednej czwartej wyniku odpytywania, a na tę blokadę czekało naraz do 107 transakcji. Moje obejście ustawia flagę po commicie, a osobny wątek wysyła powiadomienie co 10 ms. Przywraca to punkt kolana do 658, ale ten interwał dolicza się do opóźnienia.

TO-3 jest najszybszy, ale szybko opublikować zdarzenie może tylko ta instancja, która je zapisała. Jeśli padnie przed publikacją, i tak potrzebny jest mechanizm zapasowy.

## Slajd 17 – Q2 (~50 s)

Agregat Inventory Item żyje tak długo jak pozycja w katalogu, więc jego strumień zdarzeń stale rośnie. Bez snapshotów ładowanie agregatu robiło się coraz wolniejsze, co widać na lewym wykresie w skali logarytmicznej. ES-1 nie utrzymał nawet 60 zamówień na sekundę przez godzinę: przepustowość załamała się po 18–21 minutach.

Przy snapshocie co 100 zdarzeń czas ładowania pozostaje stały. W 60. minucie mediana wynosiła 713 mikrosekund zamiast 907 milisekund. Dla agregatów długożyjących snapshoty są więc koniecznością, a nie opcjonalną optymalizacją. Agregaty krótkożyjące albo odczytywane tylko w ograniczonym oknie czasowym mogą ich nie potrzebować.

## Slajd 18 – Q3 (~40 s)

Cache po stronie zapisu podniósł punkt kolana o około 8%, z 465 do 500 zamówień na sekundę. Zmniejszył też ilość danych przesyłanych z bazy, bo odtwarzane są tylko zdarzenia, których brakuje instancji w cache. Zysk był ograniczony, bo cache przyspiesza obsługę komend, a w obu wariantach ograniczeniem była saga zamówień, która nie nadążała. Odpowiedź brzmi więc: tak, cache pomaga, ale to, jak bardzo, zależy od tego, czy ładowanie agregatu jest rzeczywiście wąskim gardłem.

## Slajd 19 – Q4 (~1 min 10 s)

TO-3 celowo narusza granicę agregatu i rezerwuje wszystkie pozycje zamówienia w jednej transakcji. ES-2 ją respektuje: każdą pozycję rezerwuje osobnym zapisem, a całość koordynuje saga.

Przy W-hot punkt kolana TO-3 spadł o 52%, a ES-2 tylko o 9%. Pod koniec testu TO-3 odrzucał 43% zakończonych zamówień, a ES-2 7,9%. Przy W-fan i 140 zamówieniach na sekundę współczynnik konfliktów TO-3 wynosił 0,55 wobec 0,026, czyli był około 21 razy wyższy. Do tego każde ponowienie w TO-3 wymaga zapisania od nowa całego 16-pozycyjnego zamówienia, a nie jednej pozycji.

Przestrzeganie granic ma jednak swoją cenę. Reguła biznesowa jest egzekwowana tylko ostatecznie (eventual consistency), a saga dokłada pracy. Przy W-fan punkt kolana ES-2 też spadł, o 67%. Przyczyną nie była rywalizacja, bo system nie odrzucił ani jednego zamówienia, tylko baza danych, która wykorzystała cały limit 10 rdzeni. Przestrzeganie granic agregatów zmniejsza więc rywalizację, ale kosztem większej ilości pracy na jedno zamówienie.

## Slajd 20 – Dziękuję (~10 s)

Podsumowując: żaden z wzorców nie jest po prostu lepszy. Wydajność każdego wariantu zależy od tego, który mechanizm staje się wąskim gardłem. Dziękuję za uwagę i chętnie odpowiem na pytania.

---

## Do sprawdzenia przed obroną

- **Slajd 16:** w kolumnie z obejściem jest „every 10 m”. W pracy jest 10 ms, więc to pewnie literówka.
- **Slajd 19:** kolumna „Rejected orders” pokazuje wartości z 60. minuty. W tekście pracy dla W-hot są też średnie z całego przebiegu (28,4% wobec 2,58%). Jeśli komisja zapyta, trzeba powiedzieć, o którą wartość chodzi.
- **Czas:** jeśli wychodzi za długo, najpierw skrócić slajdy 12–14.
