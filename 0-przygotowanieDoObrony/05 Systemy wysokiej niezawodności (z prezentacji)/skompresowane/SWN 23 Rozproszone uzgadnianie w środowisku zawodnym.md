---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 23
---
# 23. Problemy rozproszonego uzgadniania w środowisku zawodnym — pojęcia
---
> Skrót pojęciowy. Algorytmy są tylko nazwane i zlinkowane — opis w [[SWN 05 Problemy uzgadniania i wyniki niemożliwości|SWN 05]]–[[SWN 08 Detektory awarii i replikacja procesu|SWN 08]].

## Model systemu
**$\mathbb{S}^{Sync}$** — kanał przechowuje co najwyżej 1 wiadomość; funkcja tranzycji stanu i funkcja generacji wiadomości deterministyczne; wykonanie startuje z pustymi kanałami
**runda** — generacja wiadomości + tranzycja stanu; procesy **krok w krok** (*lock-step*)
**$\mathbb{S}\{M\}$** — system $\mathbb{S}$ wzbogacony o mechanizm $M$ (np. $\mathbb{S}^{Sync}\{C\}$ — podpis cyfrowy, $\mathbb{S}^{Async}\{FD\}$ — detektor awarii, $\mathbb{S}^{Async}\{\text{stable storage}\}$)

**Awarie:** *stopping* — proces zatrzymuje się, także **w środku kroku 1**, umieszczając tylko podzbiór wiadomości · *bizantyjska* — generuje stan i wiadomości **dowolnie** · *awaria łącza* — gubienie wiadomości
**przemilczenie** — brak oczekiwanego komunikatu; odbiorca przyjmuje **wartość domyślną**, w $\mathbb{S}^{Sync}$ jest wykrywalne do końca rundy

## Problemy uzgadniania
**Coordinated attack** — konsensus binarny przy **zawodnych kanałach**
 *Termination* — wszystkie procesy w końcu decydują
 *Agreement* — żadne dwa nie decydują różnie
 *Weak Validity* — (1) wszystkie startują z 0 → decyzja 0; (2) wszystkie startują z 1 **i wszystkie komunikaty dostarczone** → decyzja 1

**Konsensus (C)** — *Termination*: każdy $P_c$ decyduje dokładnie 1 wartość · *Agreement*: to samo $v_i$ · *Validity*: $v_i$ zaproponowane przez pewien $P_i$
**Byzantine Agreement (BA)** — wyróżnione źródło $P_s$ rozgłasza wartość · *Validity*: **jeśli $P_s$ poprawne**, decyzją jest jego wartość
**Interactive Consistency (IC)** — uzgadniany **wektor** $\langle v_1,\ldots,v_N \rangle$ · *Validity*: jeśli $P_i$ poprawny, $i$-ta składowa jest jego propozycją
**Uniform agreement** — żadne dwa procesy (także te, które później padną) nie decydują różnie

**Redukcje:** BA to szczególny przypadek IC · $N$ kopii BA rozwiązuje IC · IC rozwiązuje C (większość lub pierwsza składowa wektora) · C rozwiązuje BA (źródło rozsyła wartość, potem konsensus)
→ **brak porządku liniowego** — sprowadzalność nie znaczy, że jeden problem jest słabszy

**Warianty:** *konsensus binarny* ($v \in \{0,1\}$) · *$k$-set consensus* — co najwyżej $k$ różnych decyzji · *approximate agreement* — wartości **bliskie sobie** · *renaming* — wartości **koniecznie różne**, z warunkiem *Anonymity*: kod nie może zależeć od początkowego ID · *elekcja*

**zadanie rozproszone** — $\mathcal{T}: \mathbb{In}^{N} \rightarrow \mathbb{Out}^{N}$, częściowa funkcja wektor wejść → zbiór legalnych wektorów decyzji; uogólnia konsensus i elekcję
**$f$-crash resilient solution** — *Termination* w każdym $f$-crash *fair execution* + *Consistency*: $O \in \mathcal{T}(I)$
**$f$-initially-dead fair execution** — $\geqslant N-f$ procesów aktywnych, każdy aktywny poprawny, każda wiadomość do poprawnego dostarczona

## Wyniki niemożliwości
**awarie łączy** — nie istnieje algorytm rozwiązujący Coordinated Attack na grafie 2 węzłów z zawodną krawędzią, **nawet w $\mathbb{S}^{Sync}$**; dowód przez ciąg wykonań nieodróżnialnych kończący się naruszeniem *Validity*
**FLP'85** — brak **deterministycznego** rozwiązania konsensusu w systemie **asynchronicznym**, gdy choćby jeden proces może ulec **awarii typu crash**
→ istotne są **wszystkie trzy** warunki; przyczyna: nie odróżnimy procesu martwego od wolnego
**BA** — nie istnieje $f$-odporny algorytm w $\mathbb{S}^{Sync}$ dla $\mathbf{f \geqslant \frac{1}{3}N}$; baza: 3 generałów i 1 zdrajca, uogólnienie przez symulację $f$ generałów jednym
**approximate agreement** — ten sam próg $f \geqslant \frac{1}{3}N$; dowód przez transformację AA → BA → **osłabienie dokładności nie pomaga**
**terminacja** — nie istnieje 1-odporne fail-stop rozwiązanie konsensusu, które **zawsze** się kończy
**Monte Carlo** — nie istnieje 1-odporne fail-stop rozwiązanie Monte Carlo konsensusu
**Las Vegas** — nie istnieje $f$-odporne dla $f \geqslant \frac{N}{2}$; dla $f < \frac{N}{2}$ **istnieje**

**Monte Carlo** — zawsze się kończy, prawdopodobieństwo poprawnej konfiguracji końcowej $> 0$
**Las Vegas** — kończy się z prawdopodobieństwem $> 0$, **wszystkie** konfiguracje końcowe poprawne

## Obejścia FLP
1. **silniejsza synchronia** — model synchroniczny lub quasi-synchroniczny
2. **słabszy model awarii** — *initially dead-processes*: żaden proces nie pada **po wykonaniu zdarzenia**; konsensus i elekcja osiągalne deterministycznie dla $f < \frac{N}{2}$
3. **poświęcenie liveness na rzecz safety** — osłabienie terminacji do *„każdy $P_c$ w końcu decyduje z prawdopodobieństwem 1"* (randomizacja) albo rezygnacja z niej (Paxos)
4. **rozszerzenie modelu o detektor awarii** — $\mathbb{S}^{Async}\{FD\}$

→ **[[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Algorytm Fischera-Lyncha-Patersona|Fischer-Lynch-Paterson]]** — konsensus w modelu procesów początkowo martwych; buduje graf, wyznacza jednoznaczny *knot* procesów poprawnych, $O(N^2)$ wiadomości

## Komunikaty ustne i podpisane
**ustne** (*oral*) — wadliwy proces może **sfałszować** komunikat lub zmienić treść przed przekazaniem; brak weryfikacji autentyczności
**podpisane** (*signed*) — podpis nie do podrobienia, każdy może zweryfikować; **wadliwe procesy szkodzą mniej**

**Założenia sieci:** **A1** każdy komunikat dostarczony poprawnie · **A2** odbiorca zna nadawcę · **A3** brak komunikatu jest wykrywalny · **A4** (tylko podpisane) podpis nie do sfałszowania, zmiana wykrywalna, autentyczność weryfikowalna

$majority(v_1,\ldots,v_n) = v$ gdy $v$ występuje $> \frac{n}{2}$ razy, inaczej $0$
$\texttt{choice}(V_i) = v$ gdy $V_i = \{v\}$, inaczej $0$ (RETREAT)
→ przy komunikatach ustnych **brak komunikatu liczy się jako 0**; przy podpisanych **brakujący komunikat jest ignorowany** — liczą się tylko podpisane rozkazy

| | ustne | podpisane |
|---|---|---|
| warunek na $f$ | $f < \frac{1}{3}N$ | formalnie $f<N$; rozsądnie $N \geqslant f+2$ |
| rundy | $f+1$ | $f+1$ |
| komunikaty | $\Omega(N^{f+1})$ | $O(N^2)$; po optymalizacji $\leqslant (N-1)+2(N-1)^2$ |

→ **[[SWN 06 Awarie bizantyjskie#BA z komunikatami ustnymi — algorytm OM|OM (Lamport-Shostak-Pease)]]** — BA na komunikatach ustnych przy $N > 3f$
→ **[[SWN 06 Awarie bizantyjskie#BA z komunikatami podpisanymi — algorytm SM|SM]]** — BA na komunikatach podpisanych; znosi ograniczenie $N > 3f$ i redukuje złożoność do $O(N^2)$

## Algorytmy konsensusu
→ **[[SWN 07 Konsensus#Paxos|Paxos]]** — konsensus w $\mathbb{S}^{Async}$, $f < \frac{N}{2}$, kosztem rezygnacji z terminacji (livelock bez elekcji) · [[Systemy Wysokiej Niezawodności/Algorytm Paxos|vault]]
→ **[[SWN 07 Konsensus#Algorytm Bracha-Touega — model i idea|Bracha-Toueg]]** — konsensus binarny typu Las Vegas, $f < \frac{N}{2}$, terminacja z prawdopodobieństwem 1 · [[Systemy Wysokiej Niezawodności/Algorytm Bracha-Touega|vault]]
→ **[[SWN 07 Konsensus#Konsensus procesów bizantyjskich — algorytm Phase-King|Phase-King (Berman-Garay)]]** — konsensus procesów bizantyjskich w $\mathbb{S}^{Sync}$, $f < \frac{N}{4}$, wielomianowa liczba komunikatów · [[Systemy Wysokiej Niezawodności/Algorytm Phase-King|vault]]
→ **[[SWN 07 Konsensus#Rozwiązanie konsensusu z użyciem dowolnego $\mathcal{D} \in \mathcal{S}$|Chandra-Toueg dla $\mathcal{S}$]]** — konsensus detektorem silnym; $N$ rund, decyduje „autorytet", $f < N$
→ **[[SWN 07 Konsensus#Rozwiązanie konsensusu z użyciem dowolnego $\mathcal{D} \in \Diamond\mathcal{S}$|Chandra-Toueg dla $\Diamond\mathcal{S}$]]** — konsensus detektorem ostatecznie silnym; rotujący koordynator, kworum większościowe, $f < \lceil \frac{N}{2} \rceil$

> ⚠ Bracha-Toueg w tym kursie to algorytm **konsensusu**, nie detekcji zakleszczenia — to inny algorytm tych samych autorów, w prezentacjach nieobecny.

## Replikacja procesu
**aktywna** (*state-machine*) — żądanie do wszystkich replik, każda odpowiada; strategia zbierania: *first come*, *all existing*, $R<n$
 ☺ klient nigdy nie ponawia żądania · dopuszczalne awarie bizantyjskie (zależnie od $R$, np. $R>3f$)
 ☹ **tylko operacje deterministyczne** · większe zużycie zasobów · **brak skalowalności**
 → niedeterminizm **nieodróżnialny od awarii bizantyjskiej**; głosowanie go nie ratuje — stany replik już się rozeszły
**pasywna** (*primary-backup*) — replika główna narzuca całkowity porządek uaktualnień
 ☺ niskie zużycie zasobów · **dopuszczalny niedeterminizm**
 ☹ brak zastosowań czasu rzeczywistego · **brak awarii bizantyjskich**
 → wymaga **członkostwa**, atomowego uaktualnienia, **elekcji** nowej repliki głównej, identyfikacji żądań
**nested invocations** — replika główna wywołuje inny serwer na podstawie decyzji niedeterministycznej i pada; nowa replika wywołuje **inny** serwer → **niespójny stan globalny**

## Komunikacja grupowa
**RBcast** — *Termination*: $P_c$ rozgłasza $m$ ⇒ w końcu je dostarcza · *Agreement*: **jakiś** $P_c$ dostarcza ⇒ **wszystkie** $P_c$ dostarczają · *Validity*: $m$ rozgłoszone przez pewien $P_i$, dostarczane co najwyżej raz
**UBcast** — *Agreement*: **jakikolwiek $P$** (poprawny **lub nie**) dostarcza $m$ ⇒ wszystkie $P_c$ w końcu dostarczają
→ różnica: RBcast wiąże tylko procesy poprawne; UBcast traktuje dostarczenie przez **kogokolwiek** jako zobowiązanie
**TOcast** = RBcast + całkowity porządek dostarczania (*atomic broadcast*, ABcast) — wymagany przez **replikację aktywną**
**VScast** — *Agreement*: wszystkie $P_c \in (v_i \cap v_{i+1})$ w końcu dostarczają $m$ · *Validity*: $m$ wysłane w $v_i$ dostarczane **przed** jakimkolwiek komunikatem z $v_{i+1}$ — wymagany przez **replikację pasywną**
**widok** (*view*) — każda zmiana członkostwa w grupie tworzy nowy widok

**Hierarchia:** RBcast → (Total Order) → TOcast; w dół (FIFO Order, Causal Order) → FIFO/Causal RBcast i TOcast; trzeci wymiar — **Uniform**

**$\text{TOcast} \cong C \cong \text{VScast}$** → z FLP wynika, że w $\mathbb{S}^{Async}\{\varnothing\}$ **nie da się zaimplementować ani replikacji aktywnej, ani pasywnej**

→ **[[SWN 08 Detektory awarii i replikacja procesu#Reliable Broadcast przez dyfuzję komunikatów|RBcast przez dyfuzję]]** — realizacja niezawodnego rozgłaszania: odbiorca przy pierwszym odbiorze retransmituje do wszystkich
→ **[[SWN 08 Detektory awarii i replikacja procesu#Rozwiązanie TOcast przy użyciu konsensusu|TOcast z konsensusu]]** — dowód $C \succeq \text{TOcast}$: okresowy konsensus nad zbiorami niedostarczonych wiadomości + deterministyczny porządek w podzbiorze

## Detektory awarii
**FD** — zbiór modułów „wyroczni", po jednej na proces, dostarczających **listę procesów podejrzewanych**; wyrocznie **mogą być omylne**, ale błędne podejrzenia nie mogą powstrzymywać poprawnych procesów

**Completeness** — **SC**: w końcu każdy $P_f$ podejrzewany przez **każdy** $P_c$ · **WC**: przez **pewien** $P_c$ (każdy $P_f$ może być podejrzewany przez **inny** $P_c$)
**Accuracy** — **SA**: **żaden** $P_c$ **nigdy** nie podejrzewany · **WA**: **pewien** $P_c$ **nigdy** · **EA**: żaden $P_c$ **w końcu** · **EWA**: pewien $P_c$ w końcu (**ten sam** $P_c$ musi w końcu nie być podejrzewany przez wszystkich)

| | SA | WA | EA | EWA |
|---|---|---|---|---|
| **SC** | $\mathcal{P}$ | $\mathcal{S}$ | $\Diamond\mathcal{P}$ | $\Diamond\mathcal{S}$ |
| **WC** | $\mathcal{Q}$ | $\mathcal{W}$ | $\Diamond\mathcal{Q}$ | $\Diamond\mathcal{W}$ |

**$\Gamma$-dokładność** — dokładność **nieosiągalna przy podziale sieci**, więc ogranicza się ją do podzbioru $\Gamma$; $\Gamma_1 \subset \Gamma_2$ ⇒ $\Gamma$-dokładność w $\Gamma_2$ implikuje ją w $\Gamma_1$
**redukcja** $\mathcal{D} \succeq \mathcal{D}'$ — $\mathcal{D}$ „emuluje" $\mathcal{D}'$ algorytmem transformacji $T_{\mathcal{D} \rightarrow \mathcal{D}'}$
**równoważność** $(\mathcal{D} \succeq \mathcal{D}' \wedge \mathcal{D}' \succeq \mathcal{D}) \Rightarrow \mathcal{D} \cong \mathcal{D}'$
Zachodzi: $\mathcal{P} \cong \mathcal{Q}$, $\mathcal{S} \cong \mathcal{W}$, $\Diamond\mathcal{P} \cong \Diamond\mathcal{Q}$, $\Diamond\mathcal{S} \cong \Diamond\mathcal{W}$; oraz $\mathcal{P} \succ \mathcal{S}$, $\mathcal{P} \succ \Diamond\mathcal{P}$, $\mathcal{S} \succ \Diamond\mathcal{S}$
**klasy po przekątnej są nieporównywalne**: $\Diamond\mathcal{P}$ i $\mathcal{S}$, $\Diamond\mathcal{Q}$ i $\mathcal{W}$, $\Diamond\mathcal{P}$ i $\mathcal{W}$, $\Diamond\mathcal{Q}$ i $\mathcal{S}$
Ponadto $\Diamond\mathcal{S}(\Gamma) \cong \Diamond\mathcal{S}$, ale $\Diamond\mathcal{W} \succ \Diamond\mathcal{W}(\Gamma)$ i $\mathcal{P} \succ \mathcal{P}(\Gamma)$

→ **[[SWN 08 Detektory awarii i replikacja procesu#Redukcja i równoważność|transformacja WC → SC]]** — dowodzi $SC \cong WC$: każdy kontroler rozsyła swoją listę podejrzeń i sumuje cudze

**Uniform Consensus** — każdy protokół rozwiązujący konsensus przy użyciu $\Diamond\mathcal{S}$ rozwiązuje też Uniform Consensus
**TRBcast** — jak RBcast, ale **każdy $P_c$ zawsze dostarcza dokładnie jeden komunikat**; gdy $P_s$ pada przed rozgłoszeniem, dostarczany jest $m_F$ (*null*). **Równoważny BA.** Rozwiązywalny przy dowolnej liczbie awarii **używając $\mathcal{P}$**; **nierozwiązywalny** przy $\Diamond\mathcal{P}$, $\mathcal{S}$ ani $\Diamond\mathcal{S}$, nawet przy jednej awarii
**algorytm *indulgent*** — kończy się i daje poprawny wynik, gdy FD spełnia specyfikację; gdy nie spełnia, może się nie zakończyć, **ale jeśli się zakończy, wynik jest poprawny**
**okresy stabilności** — FD nieimplementowalny w $\mathbb{S}^{Async}$ (własności muszą zachodzić **na zawsze**); w praktyce wystarczy, by zachodziły **dostatecznie długo**

**Rozwiązywalność wg siły modelu:** RBcast — $\mathbb{S}^{Async}\{\varnothing\}$ · C / TOcast / VScast — $\mathbb{S}^{Async}\{\Diamond\mathcal{W}\}$ · BA = TRBcast, nieblokujące zatwierdzanie atomowe — $\mathbb{S}^{Async}\{\mathcal{P}\}$ · synchronizacja zegarów — $\mathbb{S}^{Sync}\{\varnothing\}$
