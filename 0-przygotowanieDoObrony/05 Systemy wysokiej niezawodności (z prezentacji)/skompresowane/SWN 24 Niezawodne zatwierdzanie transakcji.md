---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 24
---
# 24. Niezawodne zatwierdzanie transakcji rozproszonych
---
> Komplet pojęć zagadnienia. Algorytmy są nazwane i opatrzone informacją, jaki problem rozwiązują, ale ich przebieg pozostaje w [[SWN 09 Transakcje i atomowe zatwierdzanie|SWN 09]] i [[SWN 10 Algorytmy głosowania|SWN 10]].

## Transakcje

### Atomowa akcja i jej problemy
**Atomowa akcja** (*atomic action*) to zbiór operacji niepodzielnych, traktowanych jako jedna całość. Jej realizacja w systemie rozproszonym rodzi dwa problemy:

- **Współbieżność** — trzeba jakoś koordynować żądania współbieżnych atomowych akcji dotyczące operacji na **tych samych obiektach**.
- **Izolacja** — wykonanie atomowej akcji musi być niezależne od innych atomowych akcji, a **tym bardziej niezależne musi być jej odtwarzanie**: wycofanie jednej akcji nie może pociągać za sobą wycofywania innych.

### Konflikty żądań
Współbieżne żądania są **w konflikcie**, jeżeli dotyczą **tego samego obiektu** i **przynajmniej jedno z nich wymaga jego modyfikacji**. Dwa odczyty konfliktu nie tworzą; tworzy go dopiero zapis. Podstawowym narzędziem rozwiązywania konfliktów jest **blokowanie dostępu** (*locking*).

Gdy zarządca obiektu otrzymuje żądanie będące w konflikcie z akcją już wykonywaną, może zareagować na trzy sposoby:

- **WAIT** — kolejkuje nowe żądanie.
- **REJECT** — odrzuca nowe żądanie.
- **PREEMPT** — anuluje żądanie bieżąco obsługiwane.

Przy REJECT i PREEMPT odpowiednie atomowe akcje są **wycofywane** natychmiast; przy WAIT akcja może zostać wycofana później.

## Blokady

### Dwufazowe blokowanie i zakleszczenie
Podstawowym protokołem porządkującym dostęp jest **[[SWN 09 Transakcje i atomowe zatwierdzanie#2PL i zakleszczenie|dwufazowe blokowanie]]** (*2 Phase Locking*, 2PL), który rozwiązuje problem **uszeregowania zbioru atomowych akcji** przez dynamiczne porządkowanie ich operacji. Blokady mogą być **wyłączne** (*exclusive-lock*) albo **współdzielone** (*shared-lock*), a zakłada je **lokalny zarządca blokad** (*lock manager*).

Sama blokada **nie zawiera żadnych informacji pomocnych w wykrywaniu ani rozwiązywaniu konfliktów** — wie jedynie, że zasób jest zajęty. Prowadzi to wprost do **zakleszczenia**: jeżeli atomowa akcja oczekująca na założenie kolejnej blokady może **utrzymywać blokady wcześniej uzyskane**, powstaje cykliczne oczekiwanie.

Możliwe reakcje są dwie:

- **Detekcja zakleszczenia** — połączona z wycofaniem jednej lub kilku atomowych akcji.
- **Timeout** — nakładany albo na **utrzymywanie** blokady, albo na **oczekiwanie** na jej założenie. Wadą jest **zbędne wycofywanie** akcji, które wcale nie były zakleszczone, lecz jedynie powolne.

### Porządkowanie a priori i etykiety czasowe
Protokół 2PL gwarantuje uszeregowanie atomowych akcji przez **dynamiczne** porządkowanie operacji — kolejność wyłania się dopiero w trakcie wykonania. Alternatywą jest wyznaczenie tej kolejności **a priori**, przez globalnie jednoznaczne uszeregowanie żądań. W systemie rozproszonym zapewnia je mechanizm **etykiet czasowych** (*timestamps*), w którym globalna etykieta ma postać pary
$$TS = \langle clock(i), unique\_node\_ID(i) \rangle$$
Pierwszy składnik porządkuje żądania w czasie, drugi rozstrzyga remisy między węzłami, dzięki czemu porządek jest globalnie jednoznaczny.

### Strategie rozstrzygania konfliktów
Mając etykiety czasowe, konflikt blokad można rozstrzygnąć bez czekania w nieskończoność. Zarządca obiektów zakłada blokady zgodnie z uszeregowaniem żądań, a przy konflikcie stosuje jedną z dwóch strategii, które razem noszą nazwę **[[SWN 09 Transakcje i atomowe zatwierdzanie#Strategie rozstrzygania konfliktów|WAIT-DIE i WOUND-WAIT]]** i rozwiązują problem **zakleszczeń przy zakładaniu blokad**.

- **WAIT-DIE** — gdy $TS_i < TS_j$, nowe żądanie $i$ **czeka** na blokadę posiadaną przez $j$; gdy $TS_i \geqslant TS_j$, żądanie $i$ jest **odrzucane**, a akcja wycofywana.
- **WOUND-WAIT** — odwrotnie: gdy $TS_i < TS_j$, odrzucane jest żądanie $j$; gdy $TS_i \geqslant TS_j$, żądanie $i$ **czeka**.

Starsza akcja jest więc w WAIT-DIE cierpliwa, a w WOUND-WAIT agresywna.

Obie strategie dzielą dwa problemy:

- Prowadzą do **zbędnego wycofywania akcji**, podobnie jak timeout.
- Opierają się na **niezawodności zegarów** — poprawność etykiet zależy od jakości synchronizacji czasu.

## Atomowe zatwierdzanie

### Wymagania
Celem atomowego zatwierdzania jest zagwarantowanie **globalnej atomowości** transakcji rozproszonej, co rozpada się na dwa wymagania:

- **Zgodność** — transakcję lokalnie zatwierdzają **wszystkie węzły albo żaden**; nie może dojść do sytuacji, w której część węzłów zatwierdziła, a część wycofała.
- **Poprawność** — jeżeli wszystkie węzły zakończyły swoje operacje pomyślnie **i wszystkie odpowiedzi węzłów zostały poprawnie dostarczone**, to transakcja **powinna zostać zatwierdzona**.

Drugi człon poprawności jest istotny: bez niego trywialny protokół, który zawsze wycofuje transakcję, spełniałby zgodność.

**Protokół zatwierdzania** (*commitment*) gwarantuje globalną atomowość i trwałość. Każda składowa transakcja dokonuje lokalnie modyfikacji, zapisując rejestry $UNDO$ i $REDO$, przy czym kolejność tych zapisów wyznacza zasada **write-ahead-log** — log musi wyprzedzać stan.

### Stany automatów zatwierdzania
Zachowanie uczestników protokołu opisuje się automatami skończonymi:

- **Koordynator** — stan początkowy $q_0$, stan oczekiwania na odpowiedzi $w_0$, następnie stan wycofania $a_0$ albo zatwierdzenia $c_0$, wreszcie $CT_0$ oznaczający zakończenie protokołu.
- **Uczestnik** — analogicznie: $q_i$, $w_i$ oraz $a_i$ lub $c_i$.
- **Stan buforowy** $p_0$ i $p_i$ (*precommit*) — dochodzi w protokole trójfazowym.

Automaty uzupełnia się o dwa rodzaje przejść nadzwyczajnych:

- **$F$** — awaria (*failure*).
- **$T$** — przekroczenie czasu oczekiwania (*timeout*).

### Concurrency set i warunek zablokowania
Aby formalnie uchwycić, kiedy protokół może się zablokować, wprowadza się pojęcie zbioru stanów współbieżnych. **Concurrency set** stanu $s_i$, oznaczany $\text{Cset}(s_i)$, to zbiór wszystkich stanów pozostałych procesów, które mogą występować **współbieżnie** ze stanem $s_i$; przy jego wyznaczaniu **nie uwzględnia się tranzycji $F$ i $T$**.

Warunek brzmi: **zablokowanie może wystąpić, gdy dla pewnego stanu $s$ zbiór $\text{Cset}(s)$ zawiera jednocześnie stany typu $a$ i typu $c$**. Znaczenie tego warunku jest bardzo konkretne — jeśli proces znajdujący się w stanie $s$ dopuszcza, że reszta systemu jest w trakcie wycofywania **albo** w trakcie zatwierdzania, to **na podstawie samego stanu lokalnego nie odróżni tych dwóch sytuacji** i nie może podjąć samodzielnej decyzji.

### Awarie i odtwarzanie w protokole dwufazowym
W modelu fail-stop wyróżnia się pięć istotnych punktów awarii — trzy po stronie koordynatora:

- **K1** — awaria, zanim koordynator zapisał decyzję o zatwierdzeniu. Przy odtwarzaniu koordynator wysyła wycofanie i wykonuje $UNDO$, a **uczestnicy pozostają zablokowani** aż do odebrania tej decyzji.
- **K2** — awaria pomiędzy zapisaniem decyzji o zatwierdzeniu a zakończeniem protokołu. Koordynator wysyła zatwierdzenie, a uczestnicy znów **pozostają zablokowani** do czasu jego odebrania.
- **K3** — awaria po zakończeniu protokołu; nie ma już czego odtwarzać.

i dwa po stronie uczestnika:

- **P1** — koordynator nie otrzymał odpowiedzi na żądanie zgody. Rozwiązuje to **timeout**, po którym koordynator rozsyła wycofanie.
- **P2** — uczestnik zapisał już rejestry $UNDO$ i $REDO$, ale koordynator nie otrzymał od niego potwierdzenia. Przy odtwarzaniu uczestnik **pyta koordynatora o ostateczną decyzję**, a jeśli padł, zanim zmodyfikował wszystkie obiekty, wykonuje $REDO$. Alternatywnie koordynator może po prostu powtarzać wysyłanie decyzji.

Z punktów K1 i K2 wynika **zablokowanie przetwarzania**: jeżeli koordynator ulegnie awarii, uczestnicy czekają na jego odtworzenie, **przetrzymując zasoby i utrzymując blokady**. To właśnie ta własność jest głównym zarzutem wobec protokołu dwufazowego.

### Protokoły zatwierdzania
**[[SWN 09 Transakcje i atomowe zatwierdzanie#2PC — dwufazowe zatwierdzanie|Dwufazowe zatwierdzanie]]** (*2 Phase Commitment*, 2PC) Graya rozwiązuje problem **globalnej atomowości transakcji rozproszonej**: w pierwszej fazie koordynator zbiera zgody wszystkich uczestników, w drugiej rozsyła decyzję. Jest prosty i powszechnie stosowany, ale **blokujący** — w stanie oczekiwania $\text{Cset}(w_i)$ zawiera zarówno stany wycofania, jak i zatwierdzenia, więc uczestnik nie ma jak rozstrzygnąć sytuacji samodzielnie.

**[[SWN 09 Transakcje i atomowe zatwierdzanie#3PC — trójfazowe zatwierdzanie|Trójfazowe zatwierdzanie]]** (*3 Phase Commitment*, 3PC) Skeena rozwiązuje problem **zablokowania przy awarii koordynatora**, wprowadzając dodatkowy **stan buforowy** *precommit*. Stan ten **rozdziela** w każdym zbiorze stanów współbieżnych stany wycofania od stanów zatwierdzenia, dzięki czemu żaden $\text{Cset}$ nie zawiera obu naraz. Uczestnik może wtedy podjąć **niezależną decyzję lokalną**: jeżeli jego $\text{Cset}$ zawiera stan zatwierdzenia, przechodzi do zatwierdzenia, a w przeciwnym razie do wycofania.

Cena za nieblokowanie jest jednak wysoka i kryje się w założeniach:

- **Kanały jednolicie niezawodne i bez zachowania kolejności** — w szczególności muszą dostarczyć wysłany komunikat **nawet wtedy, gdy jego nadawca uległ awarii**.
- **Jednolite rozgłaszanie niezawodne** po stronie koordynatora.
- **Nieomylna detekcja awarii węzła** — sieć musi ją wykrywać, a timeout wskazuje ją bezbłędnie.
- **Co najwyżej jeden węzeł może ulec awarii.**

## Własności terminacji

### Nieblokowanie i wolność od czekania
Dwie własności opisują, na ile protokół gwarantuje postęp:

- **Nieblokowanie** (*non-blocking*) — algorytm gwarantuje osiągnięcie zakończenia przez **co najmniej jeden proces**, o ile co najmniej jeden proces nie ulega awarii.
- **Wolność od czekania** (*wait-freedom*) — własność mocniejsza: **każdy** proces, który nie ulega awarii, osiągnie zakończenie **bez względu na zachowanie innych procesów**.

W tym ujęciu protokół dwufazowy nie spełnia nawet nieblokowania, a trójfazowy jest nieblokujący, ale nie wolny od czekania.

### Granice nieblokowania
Trzy twierdzenia wyznaczają granice tego, co da się osiągnąć, i pokazują zarazem, dlaczego założenia protokołu trójfazowego muszą być tak mocne. Nie istnieje nieblokujący protokół rozproszonego atomowego zatwierdzania, który byłby odporny na:

- **arbitralne defekty dwóch węzłów**,
- **rozdzielenie sieci** na rozłączne podsieci (*partitioning*) przy możliwości gubienia komunikatów,
- **wielokrotne rozdzielenie sieci**.

Widać stąd, że gdy tylko osłabi się założenia protokołu trójfazowego — dopuszczając dwie awarie zamiast jednej albo podział sieci — **nieblokujący protokół przestaje istnieć**. Łączy się to bezpośrednio z twierdzeniem CAP, omówionym na końcu.

## Głosowanie

### Definicja problemu
**Głosowanie** jest mechanizmem **pokonującym ograniczenia atomowego zatwierdzania**: zamiast wymagać zgody wszystkich replik, wymaga jedynie zebrania odpowiedniej ich liczby. Problem brzmi: należy zagwarantować **atomową spójność operacji odczytu i zapisu na replikach obiektu**. Regułą podstawową jest, że **zanim proces uzyska dostęp do obiektu, musi zdobyć odpowiednią liczbę głosów od pozostałych procesów**, czyli **kworum**.

Model systemu opisuje się następująco:

- Każda replika dysponuje pewną **liczbą głosów** $V_i$.
- Każda operacja na replice wymaga uzyskania **blokady** zakładanej przez lokalnego zarządcę.
- Każda replika przechowuje **monotoniczny numer wersji** $VN_i$, równy liczbie dokonanych na niej modyfikacji.
- Model awarii to *fail-recovery*, obejmujący zarówno procesy, jak i kanały.
- Operacja odczytu wymaga **kworum odczytu** $R$, a operacja zapisu — **kworum zapisu** $W$.

### Warunki poprawności kworów
Oznaczmy przez $V$ łączną liczbę głosów wszystkich replik, a przez $M$ większość:
$$V = \sum_i V_i \qquad M = \left\lceil \frac{V+1}{2} \right\rceil$$
Poprawność zapewniają dwa warunki nakładane jednocześnie:

- **$W \geqslant M$** — zapisywać może tylko większość, co **wyklucza powstanie dwóch rozłącznych kworów zapisu**, a więc współbieżne modyfikacje w rozłącznych podzbiorach replik.
- **$R + W > V$** — **kworum odczytu i kworum zapisu mają niepustą część wspólną**, dzięki czemu w każdym kworum odczytu znajdzie się **co najmniej jedna aktualna replika**.

Przy zbieraniu głosów rozróżnia się dwie sumy:

- **Kworum odczytu** — liczą się głosy wszystkich replik, które odpowiedziały: $V_{read} = \sum_{k \in \mathbb{O}} V_k$, gdzie $\mathbb{O}$ jest zbiorem procesów, które przysłały głosy.
- **Kworum zapisu** — liczą się **wyłącznie głosy replik aktualnych**: $V_{write} = \sum_{k \in \mathbb{Q}} V_k$, gdzie $\mathbb{Q} = \{k \in \mathbb{O} : VN_k = VN_{max}\}$, a $VN_{max}$ jest najwyższym numerem wersji wśród tych, które odpowiedziały.

Uaktualnienie rozsyłane jest również tylko do replik aktualnych; **starsze repliki nadrabiają zaległości przy okazji kolejnych zapisów**.

### Głosowanie statyczne i jego granice
Głosowanie nazywa się **statycznym**, ponieważ wartości $V_i$, $R$ oraz $W$ są **stałe i niezależne od bieżącego stanu systemu**. Rozdział głosów między repliki bywa przy tym decyzją projektową:

- Przesunięcie głosów na repliki **szybkie** — przyspiesza odczyty.
- Przesunięcie głosów na repliki **niezawodne** — zwiększa szansę zebrania kworum po awarii.

Problem atomowej spójności odczytu i zapisu przy kworum statycznym rozwiązuje **[[SWN 10 Algorytmy głosowania#Algorytm Gifforda|algorytm Gifforda]]**, który zbiera głosy od replik, sprawdza aktualność własnej kopii i rozsyła uaktualnienie do replik świeżych. Opis również w notatce [[Systemy Wysokiej Niezawodności/Algorytm Gifforda|Algorytm Gifforda]].

Słabością podejścia statycznego jest zachowanie przy **awarii wielu replik naraz lub przy podziale sieci**. Skoro progi są stałe, a większość liczona względem całości, to po rozpadzie systemu na fragmenty kworum może być **nieosiągalne w żadnym z nich**.

### Partycjonowanie
Rozróżnia się dwa rodzaje wyróżnionych partycji:

- **Partycja większościowa** (*majority partition*) — fragment sieci dysponujący większością głosów **całego systemu**; tylko on może kontynuować pracę przy głosowaniu statycznym.
- **Partycja pierwotna** (*primary partition*) — fragment, który mógłby stanowić większość **w konfiguracji ostatniej modyfikacji**; może być **mniejszy niż większość całości**, co stanowi istotę głosowania dynamicznego.

Historię podziałów opisuje **graf partycjonowania** (*partitioning graph*), w którym wierzchołki reprezentują partycje, a krawędzie — **podział lub scalenie**.

### Głosowanie dynamiczne
**Głosowanie dynamiczne** polega na **adaptowaniu liczby głosów lub zbioru głosujących procesów do stanu systemu po awarii**, tak aby kworum pozostało osiągalne. Występuje w dwóch wariantach:

- **Majority based voting** — zmienia **zbiór** procesów stanowiących większość.
- **Dynamic vote reassignment** — zmienia **liczbę głosów** przypisanych poszczególnym replikom.

W pierwszym wariancie zbiór węzłów tworzących większość jest zmieniany tak, by obejmował **te węzły, które zostały uaktualnione podczas najświeższej modyfikacji**. Wymaga to trzech struktur utrzymywanych przy każdej replice:

- **Numer wersji $VN_i$** — zlicza udane modyfikacje.
- **$RU_i$** — mówi, ile replik uaktualniono w **najświeższej** modyfikacji.
- **$DS_i$** — lista wskazująca **wyróżnioną replikę**; wypełniana tylko wtedy, gdy $RU_i$ jest **parzyste**, i wskazuje wówczas replikę największą w porządku liniowym spośród uczestniczących w ostatniej modyfikacji. Gdy $RU_i$ jest **nieparzyste**, $DS_i$ pozostaje puste.

Sens tego rozróżnienia jest następujący: **$DS$ służy do rozstrzygnięcia remisu**, gdy nowa partycja zawiera **dokładnie połowę** węzłów partycji poprzedniej i bez dodatkowego kryterium nie dałoby się wskazać, która połowa jest pierwotna. Przy nieparzystym $RU$ remis jest niemożliwy, więc wyróżniona replika nie jest potrzebna. Wynika stąd też ograniczenie: **jeśli $DS$ zostanie rozdzielony, system traci prawo postępu**.

Problem utrzymania postępu w partycji mniejszej niż większość całości rozwiązuje **[[SWN 10 Algorytmy głosowania#Protokół Jajodii-Mutchlera|protokół Jajodii-Mutchlera]]**, który wybiera jedną partycję zdolną kontynuować odczyty i zapisy — tę, która mogłaby stanowić większość w konfiguracji ostatniej modyfikacji — o ile tylko da się wyróżnić partycję pierwotną. Uzupełniające uwagi w notatce [[Dynamiczne głosowanie|Dynamiczne głosowanie]].

Głosowanie dynamiczne ma jednak własną pułapkę: skoro większość liczy się względem coraz mniejszej konfiguracji, **partycja pierwotna może kurczyć się aż do pojedynczego węzła**. Wystarczy wtedy awaria tego jednego węzła, by system stracił możliwość postępu, mimo że większość replik wciąż działa.

### Realokacja siły głosu
Drugim wariantem adaptacji jest zmiana liczby głosów. Decyzję o nowym przydziale można podjąć na dwa sposoby:

- **Group Consensus** — węzły grupy aktywnej **uzgadniają nowy przydział algorytmem konsensusu**; rozwiązanie poprawne, ale skomplikowane.
- **Autonomous Reassignment** — każdy węzeł decyduje o zmianie własnych głosów **na podstawie swojego widoku systemu, bez oglądania się na pozostałych**; bywa nieoptymalne, ale jest szybkie, proste i elastyczne.

Same polityki zwiększania głosów realizuje para technik **[[SWN 10 Algorytmy głosowania#Polityki zwiększania głosów|Overthrow i Alliance]]**, rozwiązujących problem **odtworzenia zdolności zebrania kworum po awarii**.

- **Overthrow** — siłę głosu przejmuje **jeden** wybrany węzeł grupy aktywnej, co wymaga przeprowadzenia elekcji.
- **Alliance** — swoje głosy zwiększają **wszystkie** węzły grupy aktywnej, dzięki czemu elekcja nie jest potrzebna.

W obu przypadkach wraz z sumą głosów **zmienia się odpowiednio próg kworum zapisu**.

## Twierdzenie CAP
Wszystkie powyższe ograniczenia podsumowuje **twierdzenie CAP**, zgodnie z którym nie da się jednocześnie osiągnąć pełnej **spójności** (*Consistency*), pełnej **dostępności** (*Availability*) i pełnej **odporności na podział sieci** (*Partition tolerance*). Znane rozwiązania lokują się na krawędziach tego kompromisu:

- **2PC** — spójność i dostępność, rezygnuje z odporności na podział.
- **Gossip** — dostępność i odporność na podział, rezygnuje ze spójności.
- **Paxos** — spójność i odporność na podział, rezygnuje z dostępności.
