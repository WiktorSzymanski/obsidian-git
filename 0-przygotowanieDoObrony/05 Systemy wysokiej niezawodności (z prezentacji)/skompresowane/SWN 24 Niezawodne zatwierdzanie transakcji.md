---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 24
---
# 24. Niezawodne zatwierdzanie transakcji rozproszonych — pojęcia
---
> Skrót pojęciowy. Algorytmy są tylko nazwane i zlinkowane — opis w [[SWN 09 Transakcje i atomowe zatwierdzanie|SWN 09]] i [[SWN 10 Algorytmy głosowania|SWN 10]].

## Transakcje
**atomowa akcja** — zbiór niepodzielnych operacji
**Problemy:** *współbieżność* — koordynacja żądań dotyczących tych samych obiektów · *izolacja* — wykonanie **i odtwarzanie** atomowej akcji musi być niezależne od innych akcji
**konflikt żądań** — współbieżne żądania dotyczą **tego samego obiektu** i **przynajmniej jedno wymaga modyfikacji**

Reakcje zarządcy obiektu na konflikt:
**WAIT** — nowe żądanie kolejkowane (akcja może być wycofana później)
**REJECT** — nowe żądanie odrzucone → *abort*
**PREEMPT** — żądanie bieżąco obsługiwane anulowane → *abort*

## Blokady
**blokada wyłączna** (*exclusive-lock*) / **współdzielona** (*shared-lock*); zakłada je **lokalny zarządca** (*lock manager*)
**zakleszczenie** — gdy akcja oczekująca na kolejną blokadę może utrzymywać wcześniej uzyskane
 reakcje: (1) detekcja zakleszczenia + abort jednej lub kilku akcji · (2) timeout na **utrzymywanie** blokady albo na **oczekiwanie** na nią
 → timeout prowadzi do **zbędnego wycofywania** akcji
**etykieta czasowa** — $TS = \langle clock(i), unique\_node\_ID(i) \rangle$; daje **globalnie jednoznaczne uszeregowanie żądań a priori**, alternatywne wobec dynamicznego porządku 2PL

→ **[[SWN 09 Transakcje i atomowe zatwierdzanie#2PL i zakleszczenie|2PL]]** — uszeregowanie zbioru atomowych akcji przez dynamiczne porządkowanie operacji
→ **[[SWN 09 Transakcje i atomowe zatwierdzanie#Strategie rozstrzygania konfliktów|WAIT-DIE / WOUND-WAIT]]** — rozstrzyganie konfliktów blokad po etykietach czasowych zamiast przez oczekiwanie
 *WAIT-DIE*: $TS_i < TS_j$ → $i$ czeka; $TS_i \geqslant TS_j$ → $i$ odrzucone
 *WOUND-WAIT*: $TS_i < TS_j$ → $j$ odrzucone; $TS_i \geqslant TS_j$ → $i$ czeka
 → problem obu: zbędne wycofywanie akcji, **niezawodność zegarów**

## Atomowe zatwierdzanie
**Zgodność** — transakcję lokalnie zatwierdzają **wszystkie węzły albo żaden**
**Poprawność** — jeśli wszystkie węzły zakończyły operacje pomyślnie **i wszystkie odpowiedzi zostały poprawnie dostarczone**, transakcja powinna zostać zatwierdzona
**protokół zatwierdzania** — gwarantuje globalną **atomowość i trwałość**; każda składowa transakcja zapisuje lokalnie rejestry UNDO i REDO (kolejność wg **write-ahead-log**)

**Stany automatów** — koordynator: $q_0$ (początkowy), $w_0$ (oczekiwanie), $a_0$ (abort), $c_0$ (commit), $CT_0$ (complete) · uczestnik: $q_i$, $w_i$, $a_i$, $c_i$; w 3PC dochodzi $p_0$ / $p_i$ (*precommit*)
**przejścia F i T** — $F$ = *failure*, $T$ = *timeout*

**concurrency set** $\text{Cset}(s_i)$ — zbiór wszystkich stanów pozostałych procesów, które mogą występować **współbieżnie** ze stanem $s_i$ (bez uwzględnienia tranzycji F i T)
**warunek zablokowania** — zablokowanie może wystąpić, gdy dla pewnego stanu $s$ zbiór $\text{Cset}(s)$ zawiera **jednocześnie stany $a$ i $c$**
→ w 2PC: $\text{Cset}(w_i) = \{w_0, \mathbf{a_0}, \mathbf{c_0}, q_j, w_j, \mathbf{a_j}, \mathbf{c_j}\}$ — proces w $w_i$ **nie odróżni lokalnie** abortu od commitu
→ w 3PC stan $p$ **rozdziela** $a$ i $c$, więc żaden $\text{Cset}$ nie zawiera obu: jeśli $\text{Cset}(s_i)$ zawiera $c$, proces może samodzielnie przejść do $c_i$, w przeciwnym razie do $a_i$

**Punkty awarii 2PC** (model fail-stop):
**K1** koordynator nie zapisał [COMMIT] → wysyła *rollback*, UNDO; **procesy zablokowane** do odebrania rollback
**K2** awaria między [COMMIT] a [COMPLETE] → wysyła *commit*; **procesy zablokowane** do odebrania commit
**K3** awaria po [COMPLETE] → nie ma czego rozważać
**P1** brak odpowiedzi na *agree_req* → **timeout**, koordynator wysyła *rollback*
**P2** uczestnik zapisał UNDO/REDO, ale brak *ack* → uczestnik **pyta o ostateczną decyzję**; jeśli padł przed zmodyfikowaniem wszystkich obiektów → **REDO**

**Zablokowanie przetwarzania** — przy awarii koordynatora procesy czekają na jego odtworzenie, **przetrzymując zasoby i blokady**

→ **[[SWN 09 Transakcje i atomowe zatwierdzanie#2PC — dwufazowe zatwierdzanie|2PC (Gray)]]** — globalna atomowość transakcji rozproszonej; **blokujący**
→ **[[SWN 09 Transakcje i atomowe zatwierdzanie#3PC — trójfazowe zatwierdzanie|3PC (Skeen)]]** — zatwierdzanie **nieblokujące** przy awarii koordynatora, dzięki buforowemu stanowi *precommit*
 założenia: kanały *uniform reliable reordering* (dostarczą komunikat **nawet gdy nadawca padł**) · koordynator używa **URBcast** · **timeout nieomylnie** wskazuje awarię · **co najwyżej jeden węzeł ulega awarii**

## Własności terminacji
**non-blocking** — zakończenie osiąga **co najmniej 1 proces**, jeśli co najmniej 1 proces nie ulega awarii
**wait-freedom** — **każdy** proces, który nie ulega awarii, osiąga zakończenie **bez względu na zachowanie innych**
→ *wait-freedom* jest **silniejsza** niż *non-blocking*

**Tw. 1** — nie istnieje nieblokujący protokół atomowego zatwierdzania odporny na **arbitralne defekty 2 węzłów**
**Tw. 2** — ani odporny na **rozdzielenie sieci** (*partitioning*) przy możliwości gubienia komunikatów
**Tw. 3** — ani odporny na **wielokrotne rozdzielenie sieci**
→ dlatego założenia 3PC (1 awaria, nieomylny timeout) są tak mocne

## Głosowanie
**quorum** — liczba głosów, którą proces musi zebrać przed dostępem do obiektu
**read-quorum $R$** / **write-quorum $W$** · $V_i$ — głosy repliki · $VN_i$ — **monotoniczny numer wersji** = liczba dokonanych modyfikacji
$$V = \sum_i V_i \qquad M = \left\lceil \frac{V+1}{2} \right\rceil$$
**$W \geqslant M$** — tylko większość zapisuje; wyklucza dwa rozłączne write-quorum
**$R + W > V$** — read-quorum i write-quorum mają część wspólną → w każdym read-quorum jest **co najmniej jedna aktualna replika**

$V_{read} = \sum_{k \in \mathbb{O}} V_k$, gdzie $\mathbb{O}$ — procesy, które przysłały głosy
$V_{write} = \sum_{k \in \mathbb{Q}} V_k$, gdzie $\mathbb{Q} = \{k \in \mathbb{O} : VN_k = VN_{max}\}$
→ zapis liczy głosy **tylko aktualnych replik** i idzie tylko do nich; starsze nadrabiają przy okazji kolejnych zapisów

**statyczne** — $V_i$, $R$, $W$ **stałe i niezależne od bieżącego stanu**
**dynamiczne** — adaptacja do stanu systemu po awarii, by dało się zebrać quorum
 *majority based voting* — zmienny **zbiór** procesów stanowiących większość
 *dynamic vote reassignment* — zmienna **liczba głosów** przypisanych replikom

**partycja większościowa** (*majority partition*) — dysponuje większością głosów **całości**
**partycja pierwotna** (*primary partition*) — mogłaby stanowić większość w **konfiguracji ostatniej modyfikacji**; może być **mniejsza niż większość całości**
**partitioning graph** — historia podziału sieci: wierzchołki = partycje, krawędzie = podział lub scalenie

Struktury głosowania dynamicznego: **$VN_i$** (numer wersji) · **$RU_i$** — liczba replik uaktualnionych w najświeższej modyfikacji · **$DS_i$** — wyróżniona replika, gdy $RU_i$ **parzyste** (największa w porządku liniowym spośród uczestników ostatniej modyfikacji); gdy $RU_i$ **nieparzyste**, $DS_i = \varnothing$
→ $DS$ rozstrzyga **remis**, gdy nowa partycja ma **dokładnie połowę** węzłów poprzedniej; przy nieparzystym $RU$ remis jest niemożliwy
→ **jeśli $DS$ zostanie rozdzielony, system nie ma prawa postępu**
→ pułapka: partycja pierwotna może skurczyć się **do jednego węzła**

**Polityki zwiększania głosów** — *Group Consensus* (węzły uzgadniają nowy przydział algorytmem konsensusu; skomplikowane) vs *Autonomous Reassignment* (każdy węzeł decyduje sam; nieoptymalne, ale szybkie i elastyczne)

→ **[[SWN 10 Algorytmy głosowania#Algorytm Gifforda|Gifford]]** — atomowa spójność `read`/`write` na replikach przez kworum **statyczne** · [[Systemy Wysokiej Niezawodności/Algorytm Gifforda|vault]]
→ **[[SWN 10 Algorytmy głosowania#Protokół Jajodii-Mutchlera|Jajodia-Mutchler]]** — kworum **dynamiczne**: postęp w partycji pierwotnej mniejszej niż większość całości · [[Dynamiczne głosowanie|vault]]
→ **[[SWN 10 Algorytmy głosowania#Polityki zwiększania głosów|Overthrow / Alliance]]** — realokacja siły głosu po awarii: jeden wybrany węzeł przejmuje $2V_f$ (wymaga elekcji) albo wszystkie pozostałe zwiększają głosy

## CAP
**Consistency · Availability · Partition tolerance** — nie da się osiągnąć 100% we wszystkich trzech wymiarach naraz
**2PC** — Consistency + Availability · **Gossip** — Availability + Partition tolerance · **Paxos** — Consistency + Partition tolerance
