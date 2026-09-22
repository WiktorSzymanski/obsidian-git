---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 22
---
# 22. Wsteczne odtwarzanie stanu przetwarzania rozproszonego
---
> Komplet pojęć zagadnienia. Algorytmy są nazwane i opatrzone informacją, jaki problem rozwiązują, ale ich przebieg pozostaje w [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane|SWN 01]]–[[SWN 04 Algorytmy odtwarzania - opis szczegółowy|SWN 04]].

## Awarie i reakcje systemu

### Rodzaje awarii
Wyróżnia się cztery rodzaje awarii, różniące się przyczyną i możliwą reakcją. **Awaria procesu** bierze się z zakleszczenia, przekroczenia czasu oczekiwania, błędu ochrony albo niespójności danych; reaguje się na nią wycofaniem procesu (*abort*) lub restartem przetwarzania. **Awaria węzła** ma źródło w błędach programowych lub sprzętowych albo w zaniku zasilania, a jedyną sensowną reakcją jest zatrzymanie węzła i restart ze zdefiniowanego wcześniej stanu. **Awaria pamięci masowej** wymaga rekonstrukcji danych z archiwum. **Awaria komunikacyjna**, wywołana uszkodzeniem medium transmisyjnego albo urządzeń sieciowych, prowadzi do naprawy łącza i retransmisji utraconych komunikatów.

### Modele restartu w modelu fail-recovery
To, w jakim stanie system znajdzie się po restarcie, opisują trzy modele. Przy **przerwie** system wznawia pracę w **dokładnie tym samym stanie**, który poprzedzał awarię — nic nie zostaje utracone. Przy **amnezji** system startuje w **stanie predefiniowanym**, całkowicie niezależnym od tego, co działo się w chwili awarii. Przypadkiem pośrednim jest **częściowa amnezja**, w której część stanu pokrywa się ze stanem sprzed awarii, a pozostała część jest predefiniowana; tak zachowuje się na przykład serwer plików po restarcie.

## Dwie filozofie odtwarzania

### Odtwarzanie postępowe
**Odtwarzanie postępowe** (*forward recovery*) polega na tym, że jeśli natura błędu powodującego awarię pozwala usunąć błędy ze stanu systemu, a system lub proces jest wyposażony w mechanizmy obsługujące dany błąd, to błąd można efektywnie wyeliminować i **umożliwić dalszy postęp przetwarzania**. System nie cofa się więc do żadnego wcześniejszego stanu — naprawia stan bieżący i idzie naprzód. Przykładem jest całkowita utrata precyzji w wyniku operacji zmiennopozycyjnej (*arithmetic underflow*), którą obsługa błędu usuwa, podstawiając w miejsce wyniku wartość zero.

Podejście to ma cztery istotne ograniczenia, które razem przesądzają o jego wąskiej stosowalności. Po pierwsze, jego skuteczność **zależy od trafności przewidywania błędów** i poprawnego przewidzenia ich konsekwencji — trzeba z góry wiedzieć, co może pójść źle i jaki będzie tego skutek. Po drugie, mechanizm **projektuje się pod konkretny system i pod konkretne błędy**, więc nie przenosi się między zastosowaniami. Po trzecie, wobec **błędów nieprzewidywalnych jest bezużyteczny**, bo nie ma dla nich przygotowanej procedury naprawczej. Po czwarte i najważniejsze, z powyższych powodów odtwarzania postępowego **nie da się zaimplementować jako ogólnego mechanizmu systemowego** — zawsze pozostaje rozwiązaniem aplikacyjnym.

### Odtwarzanie wsteczne
**Odtwarzanie wsteczne** (*backward recovery*) stosuje się wtedy, gdy błąd jest nieprzewidywalny albo awaria nieodwracalna — na przykład przy nadmiarze zmiennopozycyjnym (*arithmetic overflow*). Nie próbuje się wówczas naprawiać stanu bieżącego, lecz **wymienia się cały stan systemu na wcześniej zarejestrowany**, o którym zakłada się, że był wolny od błędów. To właśnie ta technika jest przedmiotem całego zagadnienia.

Jej zaletą jest **uniwersalność**: ponieważ nie analizuje się natury błędu, a jedynie przywraca wcześniejszy stan, metoda **nie zależy od rodzaju błędu**, daje się zastosować w każdym systemie i — w przeciwieństwie do odtwarzania postępowego — **może zostać zaimplementowana jako ogólny mechanizm systemowy**.

Poprawne odtworzenie wsteczne wymaga spełnienia dwóch warunków jednocześnie: wcześniejszy stan musi zostać **poprawnie przywrócony**, a ponadto musi on **poprzedzać wystąpienie uszkodzenia**, które doprowadziło do błędu. Gdyby przywrócony stan zawierał już uszkodzenie, odtwarzanie byłoby jałowe — przetwarzanie ponownie dobiegłoby do tej samej awarii.

Za uniwersalność płaci się trzema kosztami. **Narzut** bywa duży, bo trzeba systematycznie zapisywać stan lub historię przetwarzania. **Nawrót** oznacza, że na ogół nie ma gwarancji, iż awaria nie wystąpi ponownie po odtworzeniu stanu. **Niepowtarzalność**, czyli niemożliwość wycofania, dotyczy tych komponentów, których po prostu nie da się cofnąć — przede wszystkim **interakcji ze światem zewnętrznym**, jak wydanie pieniędzy przez bankomat.

## Odtwarzanie pojedynczego węzła

Węzeł systemu składa się z pamięci operacyjnej, w której rezydują proces $P_i$ i jego **kontroler** $C_i$, procesora oraz dwóch rodzajów pamięci zewnętrznej: **pomocniczej** i **trwałej**. Rozróżnienie procesu i kontrolera jest kluczowe dla całego zagadnienia: wszystkie algorytmy odtwarzania formułuje się jako działania **kontrolerów**, nie procesów aplikacyjnych. Odtworzenie stanu węzła można oprzeć na wycofywaniu operacji, na przywracaniu stanu albo na połączeniu obu podejść.

### Wycofywanie operacji
**Wycofywanie operacji** (*operation-based recovery*) polega na przechowywaniu w rejestrze (*log*, *audit trail*) informacji o wykonanych operacjach, tak aby dało się je odwrócić. Podstawową strategią jest **updating-in-place**, w której każdy zapis rejestruje jednocześnie krotkę $\langle OBJ, UNDO, REDO \rangle$, gdzie $OBJ$ to identyfikator modyfikowanego obiektu, $UNDO$ — jego stan sprzed modyfikacji, a $REDO$ — stan nowy. Operacja odtwarzalna jest wówczas implementowana jako kolekcja trzech operacji: **do** wykonuje działanie i je rejestruje, **undo** wycofuje je zgodnie z polem $UNDO$, a **redo** odtwarza je zgodnie z polem $REDO$.

Rozwiązanie to ma jednak **problem braku atomowości**: jeśli awaria nastąpi pomiędzy dokonaniem zmiany a zapisaniem rejestru, stan obiektu i zawartość logu rozjadą się. Usuwa to **write-ahead-log**, który narzuca kolejność zapisów — modyfikacja wykonywana jest zawsze **po** zapisaniu informacji $UNDO$, a informacja $REDO$ trafia do rejestru **przed** zatwierdzeniem zmian. Dzięki temu log zawsze wyprzedza stan, a nie odwrotnie.

### Przywracanie stanu
**Przywracanie stanu** (*state-based recovery*) polega na przechowywaniu **pełnego stanu** procesu, pamięci lub obiektu, zamiast historii operacji. Pierwszą techniką są **shadow pages**: żądanie zapisu trafia do **kopii roboczej** strony, a oryginał zachowywany jest jako kopia zapasowa, czyli *shadow page*. Po pomyślnym zapisie kopii roboczej kopia zapasowa jest usuwana, a jeśli zapis się nie powiedzie — służy do przywrócenia stanu. Technika bywa realizowana sprzętowo, a jej kosztem jest zajmowanie dodatkowego miejsca w czasie modyfikacji. Szczególnym wariantem jest **twin-page**, w którym stale utrzymuje się dwie kopie danych, a operacje wykonuje się na nich **naprzemiennie**, dzięki czemu kopia bliźniacza zawsze przechowuje stan poprzedni.

Drugą techniką przywracania stanu jest **checkpointing**, któremu poświęcona jest reszta zagadnienia.

## Punkt kontrolny

### Definicja i zasięg
**Punkt kontrolny** (*checkpoint*) to stan należący do wykonania, zachowany w **pamięci trwałej** w celu umożliwienia wznowienia przetwarzania od tego stanu podczas odtwarzania wstecznego. **Lokalnym punktem kontrolnym** $cp_i$ nazywa się punkt kontrolny pojedynczego procesu $P_i$, utworzony przez jego kontroler $C_i$. **Globalny punkt kontrolny** $CP = \langle cp_i : \forall P_i \rangle$ to wektor lokalnych punktów kontrolnych wszystkich procesów.

W niektórych protokołach rozróżnia się dodatkowo dwa rodzaje punktów lokalnych. Punkt **ostateczny** (*permanent*) należy do spójnego globalnego punktu kontrolnego i **tylko on może być punktem odtwarzania**; punkt **wstępny** (*tentative*) to punkt, który jeszcze się ostatecznym nie stał. Oba zapisywane są w pamięci trwałej, ale dla każdego procesu istnieje w danej chwili co najwyżej jeden punkt ostateczny.

### Pary punktów kontrolnych
**Parą punktów kontrolnych** nazywa się fragment globalnego punktu kontrolnego złożony z **dwóch** punktów lokalnych. Zbiór wszystkich takich par oznacza się **CPP**, a zbiór par **spójnych** — **CCPP**, przy czym $CCPP \subseteq CPP$. Globalny punkt kontrolny jest spójny dokładnie wtedy, gdy spójne są wszystkie tworzone przez niego pary:
$$CP = \langle cp_i : 1 \leqslant i \leqslant n \rangle \text{ jest spójny} \iff \bigwedge_{1 \leqslant i,j \leqslant n} (cp_i, cp_j) \in CCPP$$

Spójność pojedynczej pary zależy przy tym od kilku czynników naraz: od **determinizmu** zdarzeń i przetwarzania, od **semantyki wiadomości** oraz od dostępności dodatkowych mechanizmów obsługujących duplikację, utratę wiadomości czy niespójność stanu kanałów.

### Interwał punktu kontrolnego
**Interwałem punktu kontrolnego** $I_i^{k}$ nazywa się odcinek wykonania procesu $P_i$ **zakończony** punktem $cp_i^{k}$. Pojęcie to jest podstawą relacji z-zależności, opisanej dalej.

## Zagrożenia przy odtwarzaniu rozproszonym

Odtwarzanie pojedynczego węzła jest problemem lokalnym. W systemie rozproszonym dochodzi jednak komunikacja, przez co wycofanie jednego procesu może unieważnić stan innych. Poniższe zjawiska opisują, co dokładnie może pójść źle.

### Wiadomość osierocona
Para $(cp_i, cp_j)$ zawiera **wiadomość osieroconą** $m$, gdy odbiór wiadomości znalazł się w punkcie kontrolnym, a odpowiadające mu wysłanie — nie:
$$recv(P_j, P_i, m) \in cp_j \;\wedge\; send(P_i, \{P_j\}, m) \notin cp_i$$
Po wznowieniu przetwarzania odbiorca „pamięta" zatem wiadomość, której nadawca nigdy nie wysłał. W ogólnym przypadku taki obraz stanu jest **nie do przyjęcia**, ponieważ może naruszyć własność bezpieczeństwa przetwarzania: jeśli osierocona wiadomość przenosiła unikalny przywilej, na przykład żeton, to po odtworzeniu **oba procesy będą go posiadać**. O tym, czy konkretna wiadomość osierocona faktycznie szkodzi, rozstrzyga więc **znajomość jej semantyki**.

### Wiadomość utracona
Para $(cp_i, cp_j)$ zawiera **wiadomość utraconą** $m$, gdy sytuacja jest odwrotna — wysłanie znalazło się w punkcie kontrolnym, a odbiór nie:
$$send(P_i, \{P_j\}, m) \in cp_i \;\wedge\; recv(P_j, P_i, m) \notin cp_j$$
W przeciwieństwie do osierocenia **utrata wiadomości nie psuje spójności** obrazu stanu — taka para odpowiada poprawnemu obrazowi przetwarzania. Potrzebny jest jednak dodatkowy mechanizm odtwarzający wiadomości **znaczące**, na przykład przenoszące żeton. Ta asymetria między osieroceniem a utratą wraca później w definicjach punktu spójnego i silnie spójnego.

### Duplikacja i niespójność stanu kanałów
Jeśli przetwarzanie jest **deterministyczne**, to po wznowieniu od punktu kontrolnego proces powtórzy te same zdarzenia, a więc **wyśle ponownie wiadomość, którą odbiorca już otrzymał** — powstaje **duplikacja**, wymagająca mechanizmu tolerującego powtórzenia. Jeśli natomiast przetwarzanie jest **niedeterministyczne**, wynik ponownego wykonania jest nieprzewidywalny: w miejsce pierwotnej wiadomości $m_1$ może zostać wysłana zupełnie inna wiadomość $m_2$. Wówczas $m_1$, wciąż znajdująca się w kanale, **stanie się osierocona w chwili odebrania** — to zjawisko nazywa się **niespójnością stanu kanałów**.

### Efekt domino
Naturalną reakcją na napotkanie kontrowersyjnej pary punktów kontrolnych jest **metoda kolejnych wycofań**: zamiast wznawiać przetwarzanie od tej pary, szuka się **wcześniejszego** globalnego punktu kontrolnego, który spełnia wymogi poprawnego odtworzenia. Kłopot w tym, że wycofanie jednego procesu osieraca wiadomości u jego sąsiadów, co wymusza wycofanie także ich, to zaś unieważnia kolejne wiadomości. Ta kaskada nosi nazwę **efektu domino** i w najgorszym razie cofa całe przetwarzanie **aż do jego początku**, unieważniając cały dotychczasowy postęp.

### Ciągły restart
Skrajnym następstwem niedeterminizmu i niespójności kanałów jest **ciągły restart** (*recovery livelock*): po każdym restarcie powstaje nowa wiadomość osierocona, która wymusza kolejny restart, i tak w nieskończoność. System pozostaje formalnie sprawny, ale nigdy nie osiąga postępu.

## Spójność obrazu stanu

### Spójny i silnie spójny punkt kontrolny
Aby uporządkować powyższe zjawiska, wprowadza się dwie definicje odpowiadające spójnemu obrazowi stanu przetwarzania rozproszonego.

**Spójny punkt kontrolny** $CP^\bullet$ to taki, który jest domknięty względem relacji poprzedzania — jeśli zdarzenie należy do punktu kontrolnego, to należą do niego także wszystkie zdarzenia je poprzedzające:
$$(e \in CP \wedge e' \rightarrow e) \Rightarrow e' \in CP$$
W szczególności oznacza to, że $recv(m) \in CP^\bullet \Rightarrow send(m) \in CP$, czyli że **nie ma w nim wiadomości osieroconych**.

**Silnie spójny punkt kontrolny** $CP^{!}$ to spójny punkt kontrolny spełniający dodatkowo warunek odwrotny:
$$CP^{!} = CP^\bullet \;+\; \big(send(m) \in CP \Rightarrow recv(m) \in CP\big)$$
Różnica między nimi jest dokładnie tą asymetrią, o której była mowa wyżej: **$CP^\bullet$ dopuszcza wiadomości utracone**, czyli takie, które są jeszcze w kanale, natomiast **$CP^{!}$ wymaga, by kanały były puste**.

### Linia odtwarzania
Zbiór wszystkich globalnych punktów kontrolnych oznacza się **GC**, a jego podzbiór złożony z punktów spójnych — **$GC^\bullet$**. **Linią odtwarzania** (*recovery line*, RL) nazywa się spójny globalny punkt odtwarzania, czyli element $GC^\bullet$. **Punktem odtwarzania** $rp_i$ jest każdy punkt lokalny należący do jakiejś linii odtwarzania.

Istotne jest, że punkt odtwarzania **nie musi pokrywać się z punktem kontrolnym**. Można go osiągnąć, wycofując proces do wcześniejszego punktu odtwarzania i przesuwając wykonanie do przodu przez **odtworzenie zdarzeń zapamiętanych w rejestrze**, aż do żądanego stanu. To właśnie daje możliwość wyznaczania linii odtwarzania „z precyzją do pojedynczych zdarzeń", a nie tylko do całych punktów kontrolnych.

Spośród wielu możliwych linii odtwarzania szczególne znaczenie ma **najświeższa linia odtwarzania** $RL^{*}$, zdefiniowana jako ta, która zawiera wszystkie pozostałe:
$$RL^{*} :: \bigwedge_{RL \in GC^\bullet} RL \subseteq RL^{*}$$
Dwa pojęcia uzupełniające opisują sytuacje szczególne. **Zawężenie linii odtwarzania** zachodzi, gdy wyznaczona linia obejmuje tylko **część** procesów. **Wirtualny punkt odtwarzania** $vcp_i$ powstaje, gdy proces uznaje swój **bieżący stan** za punkt kontrolny, nie zapisując go — dzięki czemu może wejść w skład linii odtwarzania bez ponoszenia kosztu zapisu.

### Output commit
Żaden protokół odtwarzania nie cofnie operacji wykonanych na elementach spoza systemu. Problem ten rozwiązuje **output commit** — protokół, którego zadaniem jest zagwarantowanie, że stan, w którym wykonano interakcję zewnętrzną, **nigdy nie będzie musiał zostać wycofany**. Ceną jest dodatkowy narzut, zwykle blokowanie przetwarzania przed każdą taką interakcją; ponadto dane pobrane z zewnątrz muszą być składowane w pamięci trwałej, aby dało się je odtworzyć. Output commit musi wchodzić w skład **każdego** protokołu odtwarzania w systemie mającym kontakt ze światem zewnętrznym, dlatego w dalszych rozważaniach jego istnienie przyjmuje się milcząco.

## Techniki tworzenia punktów kontrolnych

Cztery techniki różnią się tym, ile koordynacji wymagają w czasie bezawaryjnym i ile pracy zostawiają na moment odtwarzania.

### Checkpointing skoordynowany
W **checkpointingu skoordynowanym** (synchronicznym) procesy uzgadniają moment zapisu tak, aby powstający globalny punkt kontrolny **zawsze był $CP^\bullet$** — a w efekcie od razu stanowił najświeższą linię odtwarzania. Odtwarzanie jest dzięki temu trywialne, ale cena jest wysoka i sprowadza się do czterech wad: technika wymaga **dodatkowych wiadomości kontrolnych**, synchronizacja powoduje **duży narzut i opóźnienie**, przy małej liczbie awarii prowadzi do **zbędnego ponoszenia tego kosztu**, a co najgorsze — koszt ten ponosi się **nawet wtedy, gdy w danym wykonaniu awarie w ogóle nie wystąpią**.

Technikę tę realizuje **[[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Algorytm Koo-Touega — idea i rodzaje punktów kontrolnych|algorytm Koo-Touega]]**, który rozwiązuje problem uzgodnienia momentu zapisu między procesami, opierając się na przetwarzaniu dyfuzyjnym i dwufazowym protokole zatwierdzania, oraz na rozróżnieniu punktów wstępnych i ostatecznych. Pełny pseudokod znajduje się w [[SWN 04 Algorytmy odtwarzania - opis szczegółowy#Algorytm Koo-Toueg|SWN 04]], a skrótowy opis w notatce [[Systemy Wysokiej Niezawodności/Algorytm Koo-Touega|Algorytm Koo-Touega]].

### Checkpointing niezależny
W **checkpointingu niezależnym** (asynchronicznym) każdy kontroler wyznacza punkty kontrolne **samodzielnie**, nie uzgadniając niczego z pozostałymi. Znika przez to cały narzut czasu bezawaryjnego, ale znika też gwarancja spójności: powstały globalny punkt kontrolny **na ogół nie jest $CP^\bullet$**, więc linię odtwarzania trzeba dopiero **odszukać**, a poszukiwanie metodą kolejnych wycofań grozi **efektem domino**.

Możliwe są dwie drogi wyjścia. Pierwsza to odszukanie spójnego punktu kontrolnego wprost w zbiorze niezależnie wyznaczonych punktów. Druga to wycofanie się do punktu niekoniecznie spójnego i wyznaczenie linii odtwarzania **z precyzją do pojedynczych zdarzeń**, odtwarzając je z rejestru utrzymywanego razem z punktami kontrolnymi — co prowadzi wprost do logowania komunikatów.

Problem wyznaczenia linii odtwarzania dla niezależnie tworzonych punktów rozwiązują dwa algorytmy. **[[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Algorytm Juanga-Venkatesana|Algorytm Juanga-Venkatesana]]** wykrywa wiadomości osierocone, porównując **liczniki** wiadomości wysłanych i odebranych, i dzięki temu wyznacza punkty odtwarzania z dokładnością do pojedynczych zdarzeń komunikacyjnych; opis również w notatce [[Systemy Wysokiej Niezawodności/Algorytm Juanga-Venkatesana|Algorytm Juanga-Venkatesana]]. **[[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Algorytm Wanga-Fuchsa|Algorytm Wanga-Fuchsa]]** rozwiązuje ten sam problem, budując **graf z-zależności** punktów kontrolnych i stosując logowanie optymistyczne, przez co musi dodatkowo radzić sobie z wiadomościami niezapisanymi jeszcze w logu; pseudokod w [[SWN 04 Algorytmy odtwarzania - opis szczegółowy#Algorytm Wanga-Fuchsa|SWN 04]], opis w [[Systemy Wysokiej Niezawodności/Algorytm Wanga-Fuchsa|Algorytm Wanga-Fuchsa]].

### Checkpointing hybrydowy
**Checkpointing hybrydowy** łączy zalety obu poprzednich: stosuje asynchroniczne wyznaczanie punktów, dzięki czemu nie obciąża wykonania bezawaryjnego, ale uzupełnia je **okazjonalną synchronizacją**, która ustala linię odtwarzania i tym samym **ostatecznie usuwa groźbę efektu domino**. Typowe rozwiązanie łączy checkpointing niezależny z logowaniem optymistycznym i przeprowadza globalną koordynację tylko przy *output commit*. Ceną jest **znaczna złożoność protokołów odtwarzania**.

Technikę tę realizuje **[[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny#Algorytm Manetho|algorytm Manetho]]**, który rozwiązuje problem izolacji procesu od skutków awarii innych procesów: dzięki logowaniu przyczynowemu ogranicza wycofanie dowolnego procesu **do jego najświeższego punktu kontrolnego w pamięci trwałej**. Pseudokod odtwarzania w [[SWN 04 Algorytmy odtwarzania - opis szczegółowy#Algorytm Manetho|SWN 04]], opis w [[Systemy Wysokiej Niezawodności/Algorytm Manetho|Algorytm Manetho]].

### Checkpointing quasi-synchroniczny
**Checkpointing quasi-synchroniczny** dąży do synchronizacji, ale niekoniecznie ją osiąga. Operuje dwoma rodzajami punktów kontrolnych. **Podstawowe punkty kontrolne** (*basic*) każdy proces wyznacza niezależnie, co ustalony interwał czasu. **Wymuszone punkty kontrolne** (*forced*) są implikowane komunikacją — powstają, gdy odebrana wiadomość niesie wyższy numer interwału niż bieżący numer odbiorcy. Kluczowe jest, że wymuszony punkt kontrolny tworzony jest **przed dostarczeniem wiadomości** do procesu; gdyby powstał po dostarczeniu, odbiór znalazłby się wewnątrz punktu kontrolnego bez odpowiadającego mu wysłania, czyli powstałaby wiadomość osierocona. Wymuszone punkty kontrolne **przesuwają linię odtwarzania do przodu**.

Technikę tę realizuje **[[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny#Checkpointing quasi-synchroniczny — algorytm Manivannana-Singhala|algorytm Manivannana-Singhala]]**, który rozwiązuje problem takiego kierowania tworzeniem punktów kontrolnych, by **każdy** z nich był $CP^\bullet$. Dzięki temu proces po awarii wycofuje się wyłącznie do swojego ostatniego punktu kontrolnego i **wznawia pracę bez czekania**, aż pozostałe procesy się wycofają, a efekt domino nie występuje w ogóle. Opis również w notatce [[Systemy Wysokiej Niezawodności/Algorytm Manivannana-Singhala|Algorytm Manivannana-Singhala]].

## Logowanie komunikatów

### Cel logowania
**Rejestrowanie wiadomości** pozwala **ograniczyć zasięg wycofania** i **uniknąć efektu domino**. Mechanizm jest prosty: po wycofaniu procesu do punktu kontrolnego odtwarza się z rejestru wiadomości zarejestrowane wcześniej, przesuwając wykonanie do przodu aż do stanu, **w którym żadna wiadomość osierocona nie została odebrana**. Dzięki temu proces nie musi cofać się dalej, niż to konieczne.

### Miejsce logowania
Wiadomość można rejestrować po stronie odbiorcy, nadawcy albo obu naraz. **Rejestrowanie u odbiorcy** wynika z tego, że operacja odbioru jest **niedeterministyczna**; odtworzenie wiadomości z rejestru **przyspiesza ponowne wykonanie** przetwarzania, a retransmisja zagubionej wiadomości może okazać się niepotrzebna. **Rejestrowanie u nadawcy** z kolei usprawnia **retransmisję**, która i tak bywa potrzebna, umożliwia przetwarzanie **niedeterministyczne**, a przy rozgłaszaniu pozwala jednym zapisem obsłużyć wszystkie kopie wiadomości.

### Tryby logowania
**Rejestrowanie pesymistyczne** zapisuje log do pamięci trwałej **atomowo z operacją** wysłania lub odbioru; w praktyce kontroler zapisuje log **po** odebraniu, ale **przed** dostarczeniem wiadomości do procesu. Gwarantuje to brak wiadomości osieroconych, ale okupione jest **dużym narzutem czasowym**, ponieważ blokuje przetwarzanie aplikacyjne.

**Rejestrowanie optymistyczne** zapisuje wiadomość do rejestru w **pamięci ulotnej**, a do pamięci trwałej przepisuje go dopiero przy sprzyjającej okazji. Narzut w czasie bezawaryjnym znika, ale w razie awarii **razem z ulotnym rejestrem tracone są niektóre wiadomości** — i to właśnie te utracone zapisy rodzą później krawędzie rollback w grafie punktów kontrolnych.

**Rejestrowanie przyczynowe** (*causal logging*) izoluje proces od skutków awarii innych procesów, opierając się na niezmienniku: informacja o każdym zdarzeniu, które **przyczynowo poprzedza** stan procesu, jest albo **w pełni zalogowana**, albo **dostępna lokalnie** dla tego procesu. Dzięki temu rollback dowolnego procesu ogranicza się do jego najświeższego punktu kontrolnego.

**Wybiórcze rejestrowanie pesymistyczne** jest kompromisem stosowanym w checkpointingu quasi-synchronicznym: loguje się pesymistycznie, ale **tylko te wiadomości**, które prawdopodobnie trzeba będzie odtworzyć, jeśli proces wycofa się do punktu kontrolnego poprzedzającego ich odbiór.

### Struktury wspierające logowanie przyczynowe
**Graf poprzedzania przyczynowego** (*antecedence graph*) danego stanu to skierowany graf acykliczny zawierający węzeł tego stanu oraz węzły **wszystkich zdarzeń niedeterministycznych, które go poprzedzają**, połączone krawędziami odpowiadającymi relacji *happened-before*. Graf ten dostarcza procesowi kompletnej historii zdarzeń, które wpłynęły na jego stan, i jest **doklejany do każdej wysyłanej wiadomości**. Ponieważ przesyłanie całego grafu dawałoby nieakceptowalny narzut, stosuje się **doklejanie przyrostowe**, przesyłając wyłącznie różnicę względem tego, co odbiorca już zna.

Odrębnym problemem jest odróżnienie komunikatów sprzed awarii od tych wysłanych po cofnięciu procesu. Służy do tego **numer inkarnacji**, przesyłany z każdą wiadomością i **zwiększany po każdym odtworzeniu**; komunikaty oznaczone nieaktualnym numerem są odrzucane. Na tej podstawie definiuje się dwa rodzaje wiadomości wymagających odmiennego traktowania. Wiadomość jest **opóźniona**, gdy $m.inc\_num < inc\_num_i$ **i** $m.ckpt\_num < RL\_num_i$ — jej wysłanie nie zostanie cofnięte, więc należy ją **przetworzyć**. Wiadomość jest **zduplikowana**, gdy $m.inc\_num < inc\_num_i$ **i** $m.ckpt\_num \geqslant RL\_num_i$ — jej wysłanie zostanie cofnięte i powtórzone w nowej inkarnacji, więc należy ją **odrzucić**.

## Zależność punktów kontrolnych

### Relacja z-zależności
Punkt kontrolny $cp_j^{\,l}$ jest **bezpośrednio z-zależny** (*zigzag-dependent*) od $cp_i^{k}$ w dwóch przypadkach: albo gdy chodzi o kolejne punkty tego samego procesu, czyli $i = j \wedge l = k+1$, albo gdy podczas interwału $I_i^{k}$ została wysłana wiadomość odebrana w interwale $I_j^{\,l-1}$:
$$i \neq j \;\wedge\; \exists_m \big( send(P_i, \{P_j\}, m) \in I_i^{k} \;\wedge\; recv(P_j, P_i, m) \in I_j^{\,l-1} \big)$$
Pełna relacja **z-dependency** to **domknięcie przechodnie** relacji bezpośredniej. Jej znaczenie wyraża twierdzenie: **z-zależność pomiędzy dwoma punktami kontrolnymi czyni z nich parę niespójną**. Innymi słowy, znalezienie ścieżki w grafie z-zależności jest równoważne stwierdzeniu, że dwa punkty nie mogą jednocześnie należeć do jednej linii odtwarzania.

### Graf punktów kontrolnych i krawędzie rollback
**Graf punktów kontrolnych** (*checkpoint graph*) to graf, w którym węzły reprezentują punkty kontrolne, czyli w istocie ich interwały, a krawędzie — relację **bezpośredniej** z-zależności. **Rozszerzony graf punktów kontrolnych** to ten sam graf uzupełniony o **wirtualne punkty kontrolne**, dzięki czemu obejmuje także bieżące stany procesów, które nie uległy awarii.

Przy logowaniu optymistycznym pojawia się dodatkowy rodzaj krawędzi. **Krawędź rollback** reprezentuje wiadomość, która **nie została jeszcze zapisana w logu** i w związku z tym przepadła wraz z pamięcią ulotną. Węzeł z wchodzącą krawędzią rollback oznacza **potencjalną niespójność**, dlatego przy wyznaczaniu linii odtwarzania **wyklucza się zarówno ten węzeł, jak i wszystkie węzły z niego osiągalne**.

Samo wyznaczanie linii odtwarzania przebiega iteracyjnie na zbiorze zwanym **root set**. Początkowo zawiera on **ostatnie punkty kontrolne wszystkich procesów**; następnie oznacza się wszystkie punkty osiągalne z elementów tego zbioru, a każdy oznaczony element zastępuje się **ostatnim nieoznaczonym** punktem tego samego procesu. Gdy żaden element nie jest już oznaczony, **root set stanowi linię odtwarzania**.

### Wiadomości non-state
Narzut logowania można ograniczyć, zauważając, że nie każda wiadomość jest warta zapisania. Względem danej pary punktów kontrolnych wiadomość znajdująca się w kanale nazywana jest *in-transit* i zostanie utracona, jeśli nie zostanie zalogowana. Natomiast wiadomość, która **nigdy nie stanie się częścią żadnego stanu kanału**, nazywana jest **wiadomością non-state** i **nie musi być logowana w ogóle** — jej obecność w rejestrze nie wpływa na spójność żadnej pary. Formalnym uzasadnieniem jest twierdzenie: jeśli w grafie punktów kontrolnych istnieje **ścieżka** z $cp_i^{k}$ do $cp_j^{\,l}$, to wszystkie wiadomości wysłane z interwału $I_j^{\,l-1}$ i odebrane w $I_i^{k}$ są wiadomościami *non-state*.

## Odśmiecanie punktów kontrolnych

### Punkty usuwalne i przeterminowane
Celem odśmiecania jest odzyskanie przestrzeni zajmowanej przez punkty kontrolne, które nie są już do niczego potrzebne. Punkt kontrolny nazywa się **usuwalnym** (*discardable*), jeżeli **nigdy nie będzie należał do żadnej przyszłej linii odtwarzania**. Szczególnym przypadkiem są punkty **przeterminowane** (*obsolete*) — poprzedzające **najgorszą możliwą** linię odtwarzania. Każdy punkt przeterminowany jest oczywiście usuwalny, ale zależność nie działa w drugą stronę: **niektóre punkty nieprzeterminowane również są usuwalne**.

Widać to wyraźnie przy **całkowitym efekcie domino**, gdy linię odtwarzania tworzą wyłącznie **początkowe** punkty kontrolne. Splot z-zależności może wtedy sprawić, że pewien punkt jest niespójny z **każdym** punktem innego procesu, a więc nigdy nie wejdzie do żadnej linii odtwarzania — mimo że formalnie nie jest przeterminowany. W takiej sytuacji duża liczba punktów nieprzeterminowanych zalega w pamięci trwałej, dając **znaczny narzut pamięciowy**.

### Kryterium usuwalności
Precyzyjne kryterium podaje twierdzenie oparte na pojęciu **nadgrafu**. **Nadgraf $\hat{G}$** to graf punktów kontrolnych $G$ sztucznie rozszerzony o **wszystkie** wirtualne punkty kontrolne. Twierdzenie mówi, że punkt kontrolny w $G$ jest **nieusuwalny wtedy i tylko wtedy**, gdy należy do **sumy linii odtwarzania** wszystkich $\hat{G}\text{-}vcp_i$ dla $1 \leqslant i \leqslant N$, gdzie $N$ jest liczbą procesów. Intuicyjnie: punkt jest nieusuwalny, jeśli przynależy do **którejkolwiek z $N$ linii odtwarzania** powstałych na skutek awarii **któregokolwiek** z procesów.
