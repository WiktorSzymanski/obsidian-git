---
tags:
  - obrona
  - odpowiedź
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 2
źródło: "[[RSO Z2 Danocentryczne modele spójności]]"
---
# 2. Danocentryczne modele spójności
---
> Wypowiedź na obronę. Sekcja **Odpowiedź** to wersja na 2–3 minuty; **Rozwinięcia** to dalsze ciągi tej samej wypowiedzi — każde odpowiada jednemu pogrubionemu hasłu i można je wpleść w to miejsce. Całość czytana ciągiem to ok. 8 minut. Formalizm i pseudokody: [[RSO Z2 Danocentryczne modele spójności]].

## Odpowiedź

Punktem wyjścia jest **zwielokrotnianie**, czyli utrzymywanie wielu kopii tych samych danych na niezależnych serwerach — robi się to dla niezawodności i dla efektywności. Ceną, którą się za to płaci, jest właśnie **spójność**, i to ona jest treścią tego zagadnienia.

Naturalnym punktem odniesienia jest **spójność ścisła**: każdy odczyt zwraca wartość zapisaną przez ostatnią operację zapisu. W systemie rozproszonym jest ona nieosiągalna, bo bez globalnego zegara słowo „ostatnia" jest niejednoznaczne, a koszt jej wymuszenia byłby ogromny. Dlatego realnym wyjściem jest **osłabienie modelu spójności**, dobrane do charakteru aplikacji i danych — a cała reszta zagadnienia to uporządkowany katalog takich osłabień.

Najpierw trzeba rozdzielić dwa pojęcia. **Model spójności** mówi, jakie gwarancje system daje aplikacji, a **protokół koherencji** to algorytm rozproszony, który taki model realizuje — czyli model mówi „co", protokół „jak".

Same modele dzieli się najpierw według tego, czyj punkt widzenia opisują. Danocentryczne opisują to, co widzą wszystkie procesy naraz; modele zorientowane na klienta to osobne zagadnienie. Danocentryczne dzielą się dalej na modele przy dostępie ogólnym, gdzie uspójnianie zachodzi przy każdej modyfikacji, i przy dostępie synchronizowanym, gdzie tylko przy jawnych operacjach synchronizujących.

Przy dostępie ogólnym jest sześć modeli i wszystkie opisuje się jednym **formalizmem**: każdy proces ma własną replikę zbioru zmiennych, każda operacja ma osobną fazę żądania i wykonania, a model to zbiór warunków nałożonych na to, w jakim uszeregowaniu proces widzi operacje. Wspólnym wymaganiem wszystkich modeli jest **legalność** uszeregowania, a różnią się one wyłącznie tym, ile porządku musi ono ponadto zachować.

Najsilniejsza jest **spójność atomowa**, czyli liniowość, która zachowuje porządek czasu rzeczywistego i wymaga wspólnego porządku zapisów. **Sekwencyjna** rezygnuje z czasu rzeczywistego na rzecz porządków lokalnych procesów — i to jest jedyna różnica między nimi. **Przyczynowa** rezygnuje ze wspólnego porządku zapisów, **PRAM** dodatkowo z przyczynowości między procesami. Z boku stoi **spójność podręczna**, czyli koherencja, która wymusza wspólny porządek zapisów, ale osobno dla każdej zmiennej; jej złożenie z PRAM daje **spójność procesorową**.

Na koniec warto powiedzieć rzecz, która spina to z komunikacją grupową: **model spójności jest odbiciem porządku dostarczania komunikatów**, którymi uspójniamy repliki. Rozgłaszanie totalne daje spójność sekwencyjną, przyczynowe — przyczynową, FIFO — PRAM.

## Rozwinięcia

### Po co się zwielokrotnia i co to kosztuje

Zwielokrotnianie służy dwóm celom. Pierwszy to **niezawodność** — odporność na awarie i większa dostępność. Drugi to **efektywność**, która rozpada się na dwie rzeczy: na współbieżny dostęp do wielu serwerów, czyli równoważenie obciążenia i skalowalność liczbową, oraz na korzystanie z serwerów położonych bliżej klienta, czyli skalowalność geograficzną. Haczyk polega na tym, że zwielokrotnianie i skalowalność same w sobie są kompromisem: skracamy czas dostępu, ale musimy utrzymywać w stanie spójnym także kopie, z których nikt aktualnie nie korzysta. Gdybyśmy chcieli odwzorować zachowanie systemu scentralizowanego, musielibyśmy aktualizować wszystkie kopie transakcyjnie, z globalną synchronizacją — i to jest dokładnie ten koszt, którego się unika.

### Konteksty: DSM i trzy koncepcje dostępu

Klasycznym kontekstem tych rozważań jest **DSM**, czyli rozproszona pamięć współdzielona — wspólna wirtualna przestrzeń adresowa dostępna dla wszystkich węzłów. Mechanicznie działa jak zwykła pamięć wirtualna, z tą różnicą, że brakująca strona sprowadzana jest z innego węzła sieci, a nie z urządzenia wymiany. Dostęp do danych można w ogóle zorganizować na trzy sposoby: **dostęp zdalny**, zawsze przez sieć, najprostszy koncepcyjnie, ale płacący opóźnieniem; **relokacja**, czyli fizyczne przeniesienie obiektu bliżej, co opłaca się przy wielokrotnych, zgrupowanych odwołaniach; oraz **zwielokrotnianie**. Relokacja i zwielokrotnianie mają ten sam zestaw problemów rozstrzygniętych inaczej: problem lokalizacji, problem rozmiaru przemieszczanej jednostki, a dalej — przy relokacji **migotanie**, czyli efekt ping-pong przy naprzemiennych odwołaniach, którego przy zwielokrotnianiu nie ma, bo każdy dostaje własną kopię, a w zamian pojawia się problem spójności replik, którego waga zależy od stosunku liczby zapisów do odczytów. Sama jednostka zwielokrotniania też jest wyborem: strona łączy fizycznie kilka odrębnych obiektów logicznych i rodzi **fałszywe współdzielenie**, pojedyncza zmienna ma duży koszt jednostkowy, a obiekt — jako struktura dostępna tylko przez zdefiniowane metody — pozwala optymalizować strategię spójności, bo sposób dostępu jest z góry znany.

### Model a protokół koherencji

Protokoły koherencji dzielą się na dwa rodzaje. **Protokół unieważniania** rozsyła małe komunikaty, które jednokrotnie unieważniają repliki — kto potem będzie chciał czytać, sam sobie sprowadzi aktualną wartość. **Protokół aktualizacji** od razu przesyła nowe wartości do niespójnych replik, więc jego komunikaty są większe. Wybór między nimi sprowadza się znowu do stosunku liczby zapisów do odczytów: przy przewadze zapisów nad odczytami aktualizowanie replik, do których nikt nie zajrzy, jest czystą stratą.

### Formalizm: historie, obrazy i legalność

Formalnie system DSM to zbiór sekwencyjnych procesów i zbiór zmiennych współdzielonych, przy czym **każdy proces ma własną replikę całego zbioru zmiennych**. Operacje to zapisy i odczyty, a każda z nich przebiega w dwóch fazach — żądania i wykonania — i to właśnie rozdzielenie tych faz w ogóle umożliwia rozbieżność porządków. Opisuje się to trzema relacjami: porządkiem lokalnym procesu, porządkiem przyczynowym i **uszeregowaniem**, czyli tym, w jakiej kolejności dany proces postrzega operacje. Historia lokalna procesu jest uporządkowana liniowo, historia globalna tylko częściowo, a obraz historii w procesie zawiera jego własne operacje oraz **wszystkie zapisy w systemie**. **Legalność** uszeregowania oznacza, że każdy odczyt zwraca wartość pewnego zapisu i że między tym zapisem a odczytem nie ma w uszeregowaniu innej operacji na tej samej zmiennej o innej wartości. Legalność jest warunkiem wspólnym wszystkich modeli — każdy model to legalność plus dodatkowe warunki.

### Atomowa i sekwencyjna

**Spójność atomowa**, nazywana też liniowością, ma dwa warunki. Pierwszy: jeżeli jedna operacja kończy się w czasie rzeczywistym, zanim druga się zacznie, to każdy proces musi widzieć je w tej kolejności. Drugi: wszystkie procesy widzą zapisy w jednym, wspólnym porządku. **Spójność sekwencyjna** ma dokładnie ten sam warunek drugi, a w pierwszym zastępuje porządek czasu rzeczywistego porządkiem lokalnym procesów — czyli wymaga tylko, żeby uszanowana została kolejność, w jakiej operacje wykonywał każdy proces z osobna, a nie kolejność na zegarze. To jest jedyna różnica między tymi modelami i najczęstsze pytanie o nie. Praktyczna konsekwencja jest taka, że wykonanie sekwencyjnie spójne może „przestawiać" operacje, które w czasie rzeczywistym wcale się nie nakładały.

### Modele słabsze i ich sens

**Spójność przyczynowa** ma już tylko jeden warunek — zachowanie porządku przyczynowego — i nie wymaga, żeby wszystkie procesy widziały zapisy w tej samej kolejności. Zapisy współbieżne, czyli przyczynowo niepowiązane, mogą więc być widziane przez różne procesy inaczej. **PRAM** zachowuje wyłącznie porządek lokalny: każdy proces widzi zapisy każdego innego procesu w kolejności ich wykonania, ale zapisy różnych procesów może przeplatać po swojemu. **Spójność podręczna**, czyli koherencja, wymusza wspólny porządek zapisów, ale osobno dla każdej zmiennej — to jest w istocie spójność sekwencyjna „per zmienna", bez żadnych gwarancji między zmiennymi. A **spójność procesorowa** to po prostu PRAM i koherencja naraz.

### Hierarchia modeli

Modele układają się w zagnieżdżenie, w którym model wewnętrzny jest słabszy: atomowa zawiera się w sekwencyjnej, ta w przyczynowej, a ta w PRAM. Idąc od zewnątrz, każdy krok to rezygnacja z jednej rzeczy — atomowa zachowuje czas rzeczywisty, sekwencyjna z niego rezygnuje na rzecz porządków lokalnych, przyczynowa rezygnuje ze wspólnego porządku zapisów, a PRAM rezygnuje z przyczynowości między procesami. Spójność podręczna nie leży na tej osi, bo ogranicza się do pojedynczej zmiennej; dopiero jej złożenie z PRAM daje procesorową.

### Protokoły i odpowiedniość porządków

Protokoły realizujące te modele mają wspólny schemat: odczyt czyta lokalną replikę, zapis rozgłasza aktualizację, a różnica leży w tym, **który z dwóch dostępów blokuje i jakiego rozgłaszania użyto**. W wariancie **fast-read** odczyt jest natychmiastowy, a zapis blokuje do czasu rozgłoszenia; w **fast-write** odwrotnie — zapis jest natychmiastowy, a odczyt blokuje, dopóki są niepotwierdzone własne zapisy. Modele przyczynowy i PRAM nie blokują nigdzie. Najważniejszy jest jednak wzorzec doboru rozgłaszania: rozgłaszanie **totalne** daje spójność sekwencyjną, **przyczynowe** — przyczynową, **FIFO** — PRAM, a warianty indeksowane zmienną, czyli utrzymujące porządek osobno dla każdej zmiennej, dają koherencję i spójność procesorową. Model spójności danych jest więc dokładnie odbiciem porządku dostarczania komunikatów z [[RSO Z1 Komunikacja grupowa|zagadnienia 1]].
