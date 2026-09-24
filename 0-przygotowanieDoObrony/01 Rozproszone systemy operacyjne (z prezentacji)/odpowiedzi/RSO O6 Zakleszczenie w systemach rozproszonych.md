---
tags:
  - obrona
  - odpowiedź
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 6
źródło: "[[RSO Z6 Zakleszczenie w systemach rozproszonych]]"
---
# 6. Zakleszczenie w systemach rozproszonych
---
> Wypowiedź na obronę. Sekcja **Odpowiedź** to wersja na 2–3 minuty; **Rozwinięcia** to dalsze ciągi tej samej wypowiedzi — każde odpowiada jednemu pogrubionemu hasłu i można je wpleść w to miejsce. Całość czytana ciągiem to ok. 8 minut. Predykaty i przebieg algorytmów: [[RSO Z6 Zakleszczenie w systemach rozproszonych]].

## Odpowiedź

Procesy przetwarzania rozproszonego wymieniają komunikaty — jedne wysyłają żądania przydziału zasobów, inne odsyłają potwierdzenia. Zakleszczenie rozproszone to sytuacja, w której **wszystkie procesy pewnego niepustego zbioru oczekują na wiadomości od innych procesów tego samego zbioru**. Warunki konieczne są te same cztery co w systemie scentralizowanym: wzajemne wykluczanie, przetrzymywanie i oczekiwanie, brak wywłaszczania oraz czekanie cykliczne.

To, co w wersji rozproszonej jest naprawdę trudne, to nie definicja, tylko **wykrycie**. Narzędziem opisu jest **graf oczekiwania**, w skrócie WFG, w którym wierzchołkami są procesy, a łuk oznacza, że jeden czeka na wiadomość od drugiego. Sedno trudności polega na tym, że **globalny graf oczekiwania nigdzie nie istnieje w całości** — każdy proces zna wyłącznie swoje krawędzie wychodzące, więc wykrycie zakleszczenia samo wymaga algorytmu rozproszonego.

Wobec zakleszczeń można postąpić trojako: **nie dopuszczać** do nich, czyli zapobiegać albo unikać; **dopuszczać i usuwać**; albo je **ignorować**.

Druga kluczowa rzecz to **model żądań**, bo od niego zależy, czym w ogóle jest zakleszczenie. W **modelu AND**, zwanym zasobowym, proces pasywny uaktywnia się dopiero wtedy, gdy dostanie wiadomości **od wszystkich** procesów swojego zbioru warunkującego. W **modelu OR**, komunikacyjnym, wystarczy wiadomość **od któregokolwiek**. Konsekwencja jest istotna: w modelu AND zakleszczeniu odpowiada **cykl** w grafie oczekiwania, a w modelu OR **węzeł**, czyli zbiór wierzchołków, z których osiągalne są wyłącznie wierzchołki tego samego zbioru. W modelu OR sam cykl **nie oznacza zakleszczenia**, bo proces może zostać odblokowany krawędzią prowadzącą poza cykl.

Dla każdego z tych modeli jest osobny algorytm detekcji, oba autorstwa **Chandy'ego, Misry i Haasa**. Dla modelu AND jest to **pogoń za krawędziami**: zablokowany proces rozsyła sondę ze swoim identyfikatorem, a jej powrót dowodzi istnienia cyklu. Dla modelu OR — **przetwarzanie dyfuzyjne** zapytaniami i odpowiedziami, w którym proces aktywny unieważnia wszystko, co do niego dotrze.

Warto jeszcze rozróżnić, o co właściwie pytamy, bo „wykryć zakleszczenie" znaczy co najmniej cztery różne rzeczy: czy zakleszczenie w ogóle wystąpiło, czy zakleszczony jest konkretny proces, jaki jest zbiór zakleszczonych, i jaki jest zbiór **maksymalny**.

## Rozwinięcia

### Graf oczekiwania i jego odczyt

Klasyczną reprezentacją jest **graf przydziału zasobów**, wiążący procesy z zasobami i rozpięty zwykle na wielu stanowiskach. Do wykrywania zakleszczeń używa się jednak jego uproszczenia — **grafu oczekiwania**, w którym zasoby znikają, a zostają same procesy i relacja czekania. Obowiązują dwie zasady odczytu: proces z łukiem wychodzącym jest **pasywny**, a proces bez łuków wychodzących — **aktywny**. Graf obejmujący cały system to graf globalny, ograniczony do jednego stanowiska — lokalny. I właśnie dlatego, że ten globalny nie istnieje w żadnym jednym miejscu, algorytmy detekcji polegają na tym, żeby efekt obiegu po krawędziach zastąpił nam oglądanie całego grafu naraz.

### Strategie: zapobieganie, unikanie, usuwanie

**Zapobieganie** polega na takim zorganizowaniu dostępu, żeby któryś z czterech warunków koniecznych nie mógł zajść — w systemie rozproszonym realizuje się to przez **priorytety** procesów przy dostępie do zasobów. Żeby przy tym nie zagłodzić procesów o niskich priorytetach, priorytet wywodzi się ze **znacznika czasowego**, i stąd dwie klasyczne strategie: **czekanie albo śmierć** oraz **zranienie albo czekanie**. Obie mają tę samą wadę: powodują **niepotrzebne wywłaszczenia**, czyli wycofują procesy, które wcale nie były zakleszczone. **Usuwanie** z kolei polega na wyborze ofiary — zwykle procesu najmłodszego albo najtańszego w wycofaniu — jej wycofaniu i zwolnieniu zasobów, a potem na **usunięciu jej krawędzi z grafu u wszystkich zainteresowanych**; bez tego pozostałości sond powodują wykrycia fantomowe. Przy wielu inicjatorach naraz porządkuje się ich priorytetem po identyfikatorze, żeby z jednego cyklu nie wycofać kilku procesów.

### Stany procesu i warunek uaktywnienia

Formalizm detekcji opiera się na tym samym aparacie co model środowiska przetwarzania. Proces jest w każdej chwili **aktywny** albo **pasywny**: aktywny może realizować zdarzenia wewnętrzne i komunikacyjne, pasywny dopuszcza co najwyżej zdarzenia odbioru. Przejście z pasywnego w aktywny wymaga spełnienia **warunku uaktywnienia**, powiązanego ze **zbiorem warunkującym** procesu, czyli sumą zbiorów procesów wszystkich dopuszczalnych w danej chwili zdarzeń odbioru. Wiadomości, które doprowadziły do spełnienia tego warunku, pobiera się z buforów **atomowo**. I tu jest sedno: to właśnie **postać predykatu uaktywnienia definiuje model żądań**, a model żądań definiuje, czym jest zakleszczenie. Modeli jest zresztą więcej niż dwa — poza AND i OR mówi się o modelu $k$ spośród $r$, o OR-AND i o predykatowym.

### Model AND a model OR — różnica formalna

Jeżeli rozpisać predykat zakleszczenia dla obu modeli, różnica siedzi dokładnie w dwóch miejscach. Pierwsze to **kwantyfikator po procesach zbioru warunkującego**: w modelu AND jest egzystencjalny, bo wystarczy jedna brakująca wiadomość, żeby proces pozostał pasywny; w modelu OR jest ogólny, bo brakować muszą wszystkie. Drugie to **zasięg zbioru warunkującego**: model OR żąda, żeby cały zbiór warunkujący zawierał się w zbiorze zakleszczonych, podczas gdy w AND wystarczy, że należy do niego jakiś jeden proces. Wniosek: **zakleszczenie w modelu OR jest trudniejsze do wystąpienia** — pojedynczy aktywny proces w zbiorze warunkującym wystarczy, by proces nie był zakleszczony. To jest dokładnie ta sama treść, co różnica między cyklem a węzłem w grafie.

### Cztery problemy detekcji

Warto rozróżnić cztery pytania, które potocznie nazywa się tak samo. **Detekcja wystąpienia** pyta tylko, czy istnieje jakiś zbiór zakleszczony. **Detekcja zakleszczenia procesu** pyta, czy dany, konkretny proces należy do takiego zbioru — i to jest pytanie, na które odpowiadają oba algorytmy Chandy'ego-Misry-Haasa, bo uruchamia je proces zainteresowany własnym losem. **Detekcja zbioru** wymaga wskazania zbioru zakleszczonych albo stwierdzenia, że takiego nie ma. A **detekcja zbioru maksymalnego** żąda dodatkowo, żeby wskazany zbiór zawierał wszystkie inne zbiory zakleszczone — co jest najtrudniejsze i najdroższe. W modelu aplikacyjnym proces wysyła do procesów swojego zbioru warunkującego żądania i dostaje przydziały albo odmowy, a detekcja dokłada do tego **własną warstwę komunikatów sterujących**.

### Chandy-Misra-Haas dla modelu AND

Algorytm dla modelu AND używa techniki **pogoni za krawędziami**. Zablokowany proces rozsyła wzdłuż krawędzi grafu oczekiwania **sondę** ze swoim identyfikatorem, a powrót tej sondy do inicjatora dowodzi istnienia cyklu, czyli zakleszczenia. Sondę propaguje się tylko wtedy, gdy spełnione są naraz trzy warunki: proces jest pasywny; sonda tego inicjatora jeszcze przez niego nie przechodziła, co zapobiega zapętleniu; oraz zasób od nadawcy nie został przyznany. Przy uaktywnieniu procesu znaczniki przejścia sond są czyszczone, żeby stare ślady nie powodowały fałszywych wykryć. Koszt nie przekracza liczby krawędzi grafu.

### Chandy-Misra-Haas dla modelu OR

Dla modelu OR nie wystarczy sonda, bo trzeba stwierdzić nie cykl, a węzeł — czyli upewnić się, że **żadna** ścieżka nie wyprowadza poza zbiór. Robi się to **przetwarzaniem dyfuzyjnym**: zablokowany proces rozsyła zapytania do swojego zbioru warunkującego, a proces **aktywny unieważnia** każde zapytanie i każdą odpowiedź, które do niego dotrą — i to właśnie milczenie procesu aktywnego ratuje inicjatora przed fałszywym wykryciem. Proces zablokowany propaguje dalej **pierwsze** zapytanie danego inicjatora, zapamiętując w liczniku, ile zapytań wysłał; na zapytania kolejne odpowiada od razu, o ile jest zablokowany nieprzerwanie od tego pierwszego. Odpowiedź odsyła dopiero po zebraniu odpowiedzi na wszystkie własne zapytania, a inicjator deklaruje zakleszczenie, gdy jego licznik zejdzie do zera. Poprawność ujmuje twierdzenie w dwóch częściach: **zakończenie** — inicjator naprawdę zakleszczony stwierdzi to w skończonym czasie, oraz **poprawność** — jeżeli deklaruje zakleszczenie, to faktycznie należy do pewnego zbioru zakleszczonego. Warto zauważyć, że **struktura tego algorytmu jest identyczna z algorytmem Dijkstry-Scholtena** detekcji zakończenia: pierwsze zapytanie odpowiada rodzicowi w drzewie, licznik — liczbie wysłanych wiadomości, a odpowiedź — sygnałowi. Kosztuje to dwukrotność liczby krawędzi.

### Pozostałe klasy algorytmów i problem fantomów

Poza pogonią za krawędziami wyróżnia się jeszcze trzy klasy: **scentralizowaną**, w której koordynator buduje globalny graf oczekiwania — proste, ale obarczone ryzykiem fantomów i pojedynczym punktem awarii; **przepychanie ścieżek**, gdzie węzły przesyłają sobie fragmenty ścieżek grafu; oraz podejście oparte na **stanie globalnym**, czyli migawce grafu i jej analizie. Uogólnieniem obu modeli jest algorytm **Brachy-Touega** dla modelu $p$ z $q$, obejmującego AND i OR jako przypadki skrajne: robi migawkę spójnego stanu globalnego, a potem **symuluje** rozdawanie przydziałów — proces, który w symulacji zbierze wymagane przydziały, sam staje się wolny i rozdaje je dalej wstecz, a kto się nie odblokował, jest zakleszczony. I rzecz, którą trzeba nazwać przy każdym z tych algorytmów: poza **postępem**, czyli wykryciem każdego zakleszczenia w skończonym czasie, wymagamy **bezpieczeństwa**, czyli braku **zakleszczeń fantomowych** — fałszywych wykryć wynikających z nieaktualnego, niespójnego obrazu grafu, na przykład wtedy, gdy informacja o zwolnieniu zasobu jeszcze nie dotarła do detektora.
