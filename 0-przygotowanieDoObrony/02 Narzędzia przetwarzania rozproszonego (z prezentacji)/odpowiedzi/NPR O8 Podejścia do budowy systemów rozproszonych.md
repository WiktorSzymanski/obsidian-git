---
tags:
  - obrona
  - odpowiedź
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 8
źródło: "[[NPR 08 Podejścia do budowy systemów rozproszonych]]"
---
# 8. Podejścia do budowy systemów rozproszonych (charakterystyka porównawcza)
---
> Wypowiedź na obronę. Sekcja **Odpowiedź** to wersja na 2–3 minuty; **Rozwinięcia** to dalsze ciągi tej samej wypowiedzi — każde odpowiada jednemu pogrubionemu hasłu i można je wpleść w to miejsce. Całość czytana ciągiem to ok. 8 minut. Pełna tabela porównawcza: [[NPR 08 Podejścia do budowy systemów rozproszonych]].

## Odpowiedź

Wszystkie podejścia do budowy systemów rozproszonych rozwiązują ten sam problem — jak dwa procesy na różnych maszynach mają ze sobą współpracować — i różnią się odpowiedzią na jedno pytanie: **czy fakt rozproszenia ukrywać, czy go ujawnić**. To jest właściwa oś tego zagadnienia i wokół niej ustawiłbym całą charakterystykę porównawczą.

Pierwsza rodzina **ukrywa**. **Zdalne wywoływanie procedur**, czyli **RPC**, i **zdalne wywoływanie metod**, czyli **RMI**, udają, że wywołanie jest lokalne: programista pisze zwykłe wywołanie, a namiastki chowają sieć. Cena jest zawsze ta sama — udawanie załamuje się przy awariach, bo wywołanie lokalne nie może „nie dojść", a zdalne może. Stąd bierze się cały aparat semantyk błędu.

Druga rodzina **ujawnia**. Przekazywanie komunikatów, **systemy kolejkowania** — z **JMS** jako specyfikacją klasyczną i **ZeroMQ** jako przypadkiem skrajnym, bez brokera — oraz **przestrzeń krotek** wymagają jawnych operacji wysłania i odebrania. Programista widzi, że coś jest przesyłane, i musi się z tym liczyć — ale w zamian dostaje **rozłączenie stron w czasie i przestrzeni**, którego pierwsza rodzina dać nie potrafi.

Osobno stawiam **spotkania Ady**, bo one też ujawniają rozproszenie, ale nie po to, żeby strony rozłączyć — przeciwnie, po to, żeby je ściśle zsynchronizować.

Najostrzej podejścia różnią się na dwóch wymiarach. **Sprzężenie w przestrzeni** to pytanie, czy nadawca musi znać odbiorcę: w RPC i RMI musi, w systemach kolejkowania zna tylko nazwę kolejki albo tematu, a w przestrzeni krotek nie ma nawet nazwy skrzynki — jest **opis treści**, czyli identyfikacja asocjacyjna. **Sprzężenie w czasie** to pytanie, czy obie strony muszą działać jednocześnie: w RPC, RMI i spotkaniach Ady muszą, a w kolejkach i przestrzeni krotek nie — komunikat przeżywa nieobecność obu stron i to nazywamy komunikacją nieustanną.

Z zestawienia wychodzi prawidłowość, którą warto nazwać wprost: **przezroczystość dostępu i luźne sprzężenie wzajemnie się wykluczają**. Podejścia, które najlepiej ukrywają sieć, najmocniej wiążą strony — bo żeby udawać wywołanie lokalne, trzeba mieć konkretnego, działającego rozmówcę. A te, które strony rozłączają, muszą ujawnić, że komunikat jest gdzieś przekazywany. Podobnie **trwałość kosztuje wydajność**.

## Rozwinięcia

### RPC

Zdalne wywołanie procedury jest historycznie pierwszym i najbardziej wpływowym pomysłem. Jednostką jest **procedura**, wywołanie jest domyślnie synchroniczne, choć istnieją warianty asynchroniczne i wywołanie zwrotne. Interfejs opisuje się w **osobnym języku** — w Sun RPC jest to plik przetwarzany przez generator, który wytwarza namiastki. Dane konwertuje się **kanonicznie**, czyli do jednego wspólnego formatu pośredniego, klienta wiąże się z serwerem **dynamicznie przez łącznik**, a błędy interpretuje według wybranej semantyki: co najmniej raz albo co najwyżej raz. Identyfikacja odbiorcy jest tu najbardziej konkretna ze wszystkich podejść — adres serwera plus numer programu, wersji i procedury.

### RMI

RMI to ten sam pomysł przeniesiony na obiekty. Jednostką jest **obiekt zdalny**, a operacją metoda zadeklarowana w interfejsie wywiedzionym z `Remote`. Istotna różnica wobec RPC jest taka, że interfejs opisuje się **w języku implementacji**, a nie w osobnym języku opisu interfejsów — co jest wygodne, ale zamyka nas w jednym ekosystemie. Drugą ważną rzeczą jest sposób przekazywania argumentów: obiekty zwykłe idą **przez wartość** i muszą być serializowalne, a obiekty zdalne **przez referencję** — serwer dostaje wtedy namiastkę i może wołać z powrotem do klienta, co daje naturalne wywołania zwrotne. Semantyka jest ustalona na „co najwyżej raz", a błąd objawia się wyjątkiem. Trwałości nie ma: mechanizm obiektów aktywowalnych pozwala wprawdzie obiekt ożywić na żądanie, ale **nie utrwala jego stanu**.

### Systemy kolejkowania komunikatów

Tutaj wchodzi **pośrednik**. Nadawca umieszcza komunikat w **kolejce**, przy komunikacji punkt-punkt z jednym konsumentem, albo publikuje go pod **tematem**, przy publikuj-subskrybuj z wieloma konsumentami. Za trwałe przechowanie odpowiada **zarządca kolejek**, czyli broker. To właśnie on daje komunikację **nieustanną**: strony nie muszą się znać ani działać jednocześnie, a komunikat czeka. **JMS** jest specyfikacją takiego systemu dla Javy — z transakcjami, trybami potwierdzeń, trwałością komunikatów, priorytetami, czasem życia i subskrypcjami trwałymi. Odbiór może być synchroniczny, przez jawne pobranie, albo asynchroniczny, przez nasłuchiwacz.

### ZeroMQ jako przypadek skrajny

ZeroMQ warto wymienić osobno, bo łamie regułę tej rodziny: **nie ma w nim brokera**. Kolejkowanie jest wbudowane w gniazdo, interfejs jest wzorowany na gniazdach BSD, a komunikat to **nieinterpretowana sekwencja bajtów** — biblioteka nie wie, co przesyła. Schematy komunikacji wynikają z typów gniazd: żądanie-odpowiedź, publikuj-subskrybuj, rozdzielaj-zbieraj. W zamian za wydajność traci się trwałość i gwarancje: skoro nic nie jest zapisywane, to nic nie przeżywa awarii. To jest dokładnie ta sama wymiana, o której mówiłem — trwałość za wydajność, tylko postawiona na przeciwnym końcu niż w JMS.

### Przestrzeń krotek

Linda i JavaSpaces idą najdalej w rozluźnianiu sprzężenia: zastępują nazwę skrzynki **dopasowaniem treści**. Procesy umieszczają w przestrzeni **krotki**, pobierają je operacją pobrania i odczytują bez pobierania operacją odczytu, a atrybuty krotki w zapytaniu mogą pozostać bez wartości i wtedy pasują do czegokolwiek. Dwie rzeczy warto podkreślić. Po pierwsze, przestrzeń jest **zbiorem, a nie kolejką** — nie ma tu żadnego uporządkowania, jest tylko gwarancja żywotności, czyli że krotka w końcu zostanie znaleziona. Po drugie, ponieważ operacje pobrania i odczytu **blokują**, przestrzeń krotek jest zarazem mechanizmem synchronizacji, a nie tylko komunikacji. Trwałość realizuje się przez **dzierżawę**, czyli ograniczony czas życia krotki.

### Spotkania i obiekty chronione Ady

**Spotkanie asymetryczne** w Adzie to komunikacja synchroniczna między parą zadań. Zadanie czynne, czyli klient, woła **wejście** zadania biernego, a to przyjmuje wywołanie odpowiednią instrukcją; ta ze stron, która dotrze pierwsza, czeka na drugą. Nie ma tu żadnego pośrednika ani buforowania — **komunikacja i synchronizacja są tym samym zdarzeniem**, i to jest najściślejsze sprzężenie w czasie ze wszystkich omawianych podejść. **Obiekty chronione** są uzupełnieniem: dają komunikację asynchroniczną między wieloma zadaniami przez dane współdzielone, z wzajemnym wykluczaniem wbudowanym w język, a nie doklejanym biblioteką.

### Co z czego wynika w zestawieniu

Jeśli zestawić te podejścia w tabeli, widać kilka regularności. **Pośrednik** występuje tylko w MOM i w przestrzeni krotek — łącznik w RPC i rejestr w RMI pośrednikami nie są, bo służą wyłącznie do wiązania, a sama komunikacja idzie potem bezpośrednio. **Opis interfejsu** jest albo w osobnym języku, jak w RPC, albo w języku implementacji, jak w RMI i Adzie, albo nie ma go wcale, bo liczy się treść komunikatu lub kształt krotki. **Liczba odbiorców** to jeden dla RPC, RMI, kolejki i pobrania krotki, a wielu dla tematu i dla odczytu krotki. I wreszcie **semantyka błędu**: wybieralna w RPC, ustalona w RMI, zastąpiona trybami potwierdzeń i transakcjami w JMS, a w przestrzeni krotek sprowadzona do samej żywotności.

### Pointa porównania

Na koniec warto zebrać to w dwa zdania, bo o to właściwie pyta „charakterystyka porównawcza". Po pierwsze: **przezroczystość dostępu kupuje się ścisłym sprzężeniem**, a luźne sprzężenie kupuje się jawną komunikacją — nie ma podejścia, które dawałoby jedno i drugie, i to nie jest niedopatrzenie, tylko konsekwencja tego, że udawanie wywołania lokalnego wymaga konkretnego, żywego rozmówcy. Po drugie: **każda gwarancja ma swoją cenę w wydajności** — trwałe komunikaty JMS oznaczają zapis na dysk przy każdej wiadomości, a ZeroMQ jest szybkie dokładnie dlatego, że nie zapisuje niczego. Wybór podejścia jest więc wyborem miejsca na tych dwóch osiach, a nie wyborem narzędzia „lepszego" czy „gorszego".
