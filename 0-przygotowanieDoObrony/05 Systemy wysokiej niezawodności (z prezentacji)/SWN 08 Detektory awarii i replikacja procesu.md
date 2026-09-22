---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
source: "Slajdy-FT-11_ProcessReplication.pdf"
slajdy: "1–44"
zagadnienie: "23 (uzupełnienie)"
---
# SWN 08. Replikacja procesu, komunikacja grupowa i detektory awarii
---
> Wykład spina **replikację** z **konsensusem**. Zaczyna od dwóch modeli replikacji procesu — **aktywnej** (*state-machine*) i **pasywnej** (*primary-backup*) — i pokazuje, że każdy z nich wymaga innego rodzaju rozgłaszania: **TOcast** i **VScast**. Następnie dowodzi, że **TOcast $\cong$ C $\cong$ VScast**, więc FLP zabrania obu w $\mathbb{S}^{Async}\{\varnothing\}$. Rozwiązaniem jest rozszerzenie modelu o **detektory awarii**: wykład podaje ich własności (*completeness*, *accuracy*), **osiem klas** ($\mathcal{P}$, $\mathcal{Q}$, $\mathcal{S}$, $\mathcal{W}$ i ich „ostateczne" warianty $\Diamond$), redukcje między nimi oraz algorytmy konsensusu dla $\mathcal{S}$ i $\Diamond\mathcal{S}$.

> [!info] Zakres
> To jest **uzupełnienie zagadnienia 23** — formalnie osobny wykład, ale zawiera pojęcia, bez których nie da się odczytać `FT_ConsensusFD.pdf` ([[SWN 07 Konsensus#Konsensus z detektorami awarii]]) ani odpowiedzieć na punkt „detektory awarii (◇S, P), konsensus z detektorem (Chandra-Toueg)" z [[Braki w notatkach]].

---
## Replikacja procesu
<sub>Slajdy-FT-11_ProcessReplication.pdf, slajdy 1–2</sub>

Slajd 1 umiejscawia **replikację** wśród trzech filarów niezawodności (obok *robust algorithms* i *stabilization*) i wskazuje trzy jej poziomy: **hardware**, **data**, **services (processes)**.

![[swn-ft11-s02-aktywna-vs-pasywna.png]]
<sub>Dwa modele: **aktywna replikacja = state-machine replication** (żądanie trafia do wszystkich replik, każda odpowiada) oraz **pasywna replikacja = primary-backup** (żądanie obsługuje replika główna, która rozsyła uaktualnienia do kopii zapasowych) — slajd 2</sub>

---
## Replikacja aktywna

### Charakterystyka
<sub>slajd 3</sub>

- klient może wybrać **strategię zbierania odpowiedzi**: *first come*, *all existing* ($n$), $R < n$,
- może radzić sobie z **awariami bizantyjskimi**, zależnie od $R$ (np. $R > 3f$),
- **grupy dynamiczne** (choć członkostwo w grupie jest nieistotne),
- ▸ większość frameworków chmurowych: **Google Spanner**, **Apache Zookeeper**, **MySQL Group Replication**.

### Spójność
<sub>slajd 4</sub>

![[swn-ft11-s04-aktywna-wielu-klientow.png]]
<sub>Dwóch klientów $P_i$ i $P_j$ wysyła żądania jednocześnie — slajd 4</sub>

- **spójność:** wielu klientów może zgłaszać żądania jednocześnie,
- ▸ żądania muszą być **całkowicie uporządkowane** (*totally ordered*),
- ▸ **dokładna relacja porządkująca jest nieistotna** — wszystkie repliki muszą po prostu przetwarzać żądania **w tej samej kolejności**.

### Niedeterminizm
<sub>slajd 5</sub>

- przetwarzanie niedeterministyczne **nie jest dozwolone** — mogłoby prowadzić do różnych odpowiedzi na to samo żądanie klienta,
- **nie da się go odróżnić od awarii bizantyjskich**,
- **głosowanie ani porozumienie bizantyjskie nie rozwiązują problemu** — każda odpowiedź może być inna,
- nawet jeśli klient wybierze jedną odpowiedź, **stany poszczególnych replik rozeszły się do niespójności**.

### Podsumowanie
<sub>slajd 6</sub>

**Zalety:**
- klient **nigdy nie ponawia żądania** (o ile tylko $f < N$),
- **awarie bizantyjskie mogą być dozwolone**.

**Wady:**
- dozwolone są **tylko operacje deterministyczne** (wymóg spójności),
- **większe zużycie zasobów**: $n \leqslant N$ replik przetwarza każde żądanie,
- **brak skalowalności**: zwiększanie $N$ nie zwiększa wydajności (przepustowości).

---
## Replikacja pasywna

### Charakterystyka
<sub>slajd 7</sub>

- **spójność** replik: **replika główna narzuca całkowity porządek uaktualnień**,
- **grupy dynamiczne** (wymagana usługa **członkostwa** — *membership*),
- **niezawodność** (gdy replika główna ulega awarii):
  - ▸ operacja uaktualnienia musi być **atomowa** ($\rightarrow$ *Reliable Broadcast*),
  - ▸ **nowa replika główna musi zostać wybrana**,
  - ▸ klient może zaobserwować **opóźnienie**,
  - ▸ klient może być zmuszony **ponowić żądanie**,
  - ▸ każde żądanie musi być **jednoznacznie identyfikowane**.

### Niedeterminizm i wywołania zagnieżdżone
<sub>slajdy 8–9 i 11</sub>

- przetwarzanie niedeterministyczne **jest dozwolone**, ponieważ tylko jedna replika przetwarza żądania,
- a uaktualnienia zapewniają globalną spójność wśród członków grupy,
- **…chyba że** rozważymy **wywołania zagnieżdżone** (*nested invocations*).

![[swn-ft11-s11-nested-invocations.png]]
<sub>Łańcuch: klient $P_i$ → usługa front-end → usługa back-end — slajdy 9 i 11</sub>

**Scenariusz problemu** (slajd 11):
- replika główna $x^1$ wywołuje inny serwer $y^{(k)}$,
- na podstawie **decyzji niedeterministycznej**,
- następnie $x^1$ **ulega awarii po otrzymaniu odpowiedzi** od $y^{(k)}$,
- nowa replika główna $x^2$ wywołuje **inny serwer** $z^{(m)}$,
- na podstawie **własnej decyzji niedeterministycznej**,
- **oba $y$ i $z$ zostały uaktualnione** — niespójny stan globalny.

### Spójność przy wielu klientach
<sub>slajd 10</sub>

![[swn-ft11-s10-pasywna-zmiana-primary.png]]
<sub>Awaria repliki głównej $x^1$ w trakcie obsługi i wybór nowej repliki głównej ($x^3$) — slajd 10</sub>

▸ **żądania muszą być zsynchronizowane ze zmianami członkostwa** (*membership changes*).

### Podsumowanie
<sub>slajd 12</sub>

**Zalety:**
- **niskie zużycie zasobów** (tylko jedna replika przetwarza żądania),
- **możliwe przetwarzanie niedeterministyczne** (dopóki nie ma wywołań zagnieżdżonych).

**Wady:**
- **brak zastosowań czasu rzeczywistego** (mogą wystąpić duże opóźnienia),
- **brak awarii bizantyjskich** (klienci nie mogą otrzymywać niepoprawnych odpowiedzi).

---
## Komunikacja grupowa

### Porządek i widoki
<sub>slajd 13</sub>

**Porządek komunikatów:**
- **replikacja aktywna wymaga rozgłaszania całkowicie uporządkowanego** — **TOcast** (*totally ordered multicast*); bywa nazywane **rozgłaszaniem atomowym** — **ABcast**.

**Widoki:**
- każda zmiana członkostwa w grupie (na skutek awarii lub odtworzenia procesu) stanowi **nowy widok** (*view*) tej grupy,
- **replikacja pasywna wymaga rozgłaszania synchronicznego względem widoków** — **VScast** (*view synchronous multicast*).

### TOcast i RBcast
<sub>slajd 14</sub>

> **TOcast = RBcast + TO** (całkowity porządek dostarczania)

> **RBcast (Reliable Broadcast):**
> **Termination:** jeśli $P_c$ rozgłasza $m$ $\Rightarrow$ w końcu je dostarcza.
> **Agreement:** jeśli **jakiś** $P_c$ dostarcza $m$ $\Rightarrow$ **wszystkie** $P_c$ dostarczają $m$.
> **Validity:** $m$ zostało rozgłoszone przez pewien $P_i$; $m$ jest dostarczane **co najwyżej raz**.

*(Dymek na slajdzie podkreśla, że $P_c$ w warunku Termination oznacza proces **poprawny**.)*

### Reliable Broadcast przez dyfuzję komunikatów
<sub>slajd 15</sub>

```
C_i:
    RBcast(m):
        {
          send m to all P  (including P_i)
        }

    Rdeliver(m) occurs as follows:
        {
          when receive(m) for the first time (for that m)
              if sender(m) != P_i then send m to all
        }
```

Schemat warstwowy pod pseudokodem: **podsystem komunikacyjny** odbiera `receive(m)` i wykonuje `deliver(m)` do procesu $P_i$.

### Uniform Broadcast
<sub>slajd 16</sub>

W RBcast: jeśli **nadawca $m$ uległ awarii** $\Rightarrow$ $m$ jest dostarczane **przez wszystkie $P_c$ albo przez żaden**.

> **UBcast (Uniform Broadcast):**
> **Agreement:** jeśli **jakikolwiek $P$** (poprawny **lub nie**) dostarcza $m$ $\Rightarrow$ **wszystkie $P_c$** w końcu dostarczają $m$ **(!!!)**

*Uwaga ze slajdu:* $P_f$ (proces, który uległ awarii) **może dostarczyć cokolwiek chce**.

> [!important] Różnica RBcast vs UBcast
> RBcast wiąże tylko procesy **poprawne**: jeśli proces dostarczył $m$ i zaraz potem padł, reszta nie musi nic robić. UBcast obejmuje **także procesy wadliwe** jako przesłankę — dostarczenie przez kogokolwiek zobowiązuje wszystkich poprawnych.

### View Synchronous Multicast
<sub>slajd 17</sub>

- **spójny zbiór komunikatów dostarczanych mimo zmian członkostwa w grupie**,
- $v_i$, $v_{i+1}$ — dwa **kolejne widoki** tej samej grupy.

> **VScast:**
> **Agreement:** wszystkie $P_c \in (v_i \cap v_{i+1})$ w końcu dostarczają $m$.
> **Validity:** jeśli $m$ zostało wysłane w $v_i$, to musi zostać dostarczone w każdym $P_c \in (v_i \cap v_{i+1})$ **przed jakimkolwiek komunikatem z $v_{i+1}$**.

### Hierarchia rozgłaszania uporządkowanego
<sub>slajdy 18–19</sub>

![[swn-ft11-s17-ordered-broadcasts.png]]
<sub>Siatka: **RBcast** → (Total Order) → **TOcast**; w dół (FIFO Order) → **FIFO RBcast** / **FIFO TOcast**; dalej (Causal Order) → **Causal RBcast** / **Causal TOcast** — slajd 18</sub>

![[swn-ft11-s18-ordered-broadcasts-3d.png]]
<sub>**3D cube** — trzeci wymiar to **Uniform**: RBcast → (Uniform) → **UBcast**, i analogicznie dla pozostałych — slajd 19</sub>

### Konsensus a TOcast
<sub>slajdy 20–21</sub>

Slajd 20 zestawia obok siebie definicje **C (Consensus)** i **TOcast = RBcast + TO** (por. [[SWN 07 Konsensus#Definicja konsensusu i powtórka]]).

> **TOcast $\cong$ C $\cong$ VScast**

**Co to znaczy?** (slajd 21)
- jeśli mamy rozwiązanie dla **TOcast** $\Rightarrow$ umiemy rozwiązać **C**,
- jeśli mamy rozwiązanie dla **C** $\Rightarrow$ umiemy rozwiązać **TOcast**,
- *pierwsza implikacja jest trywialna, drugą zobaczymy później*.

---
## Niemożliwość konsensusu i konsekwencje dla replikacji
<sub>slajdy 22–23</sub>

![[swn-ft11-s41-solvability-of-problems.png]]
<sub>Rozwiązywalność problemów wobec siły modelu — **RBcast** wystarcza $\mathbb{S}^{Async}\{\varnothing\}$, **Consensus / TOcast / VScast** wymagają $\mathbb{S}^{Async}\{\Diamond\mathcal{W}\}$, **BA = TRBcast** i **nieblokujące zatwierdzanie atomowe** wymagają $\mathbb{S}^{Async}\{\mathcal{P}\}$, a **synchronizacja zegarów** — $\mathbb{S}^{Sync}\{\varnothing\}$; slajd 41</sub>

**FLP'85 Impossibility Result** (slajd 22): nie istnieje deterministyczne 1-odporne fail-stop rozwiązanie konsensusu w systemie asynchronicznym.

**Dlaczego?**
- w systemach asynchronicznych **nie możemy odróżnić procesu, który uległ awarii, od wolnego lub bardzo odległego**,
- w systemach synchronicznych **będziemy wiedzieć, że jakiś proces padł** = **doskonała detekcja awarii**,
- **doskonała detekcja awarii = konsensus rozwiązywalny**,
- w systemach asynchronicznych = **brak detekcji awarii**,
- **brak detekcji awarii = konsensus nierozwiązywalny**.

**Co to oznacza?** (slajd 23)
- ☹ **nie możemy zaimplementować ani TOcast, ani VScast w $\mathbb{S}^{Async}\{\varnothing\}$** — czyli **ani aktywnej, ani pasywnej replikacji**,
- **co można zrobić?** ☺ znaleźć rozwiązania randomizowane · ☺ **rozszerzyć model $\mathbb{S}^{Async}$**,
- **czego potrzebujemy?** — **po prostu detekcji awarii: $\mathbb{S}^{Async}\{FD\}$**; *czy doskonała detekcja awarii jest konieczna?*

---
## Detektory awarii

### Definicja
<sub>slajd 24</sub>

> **FD = Failure Detector**
> - **rozproszony detektor uszkodzeń** = zbiór modułów „wyroczni",
> - każda z wyroczni jest **przyłączona do jednego procesu**,
> - jej zadaniem jest **dostarczanie listy procesów podejrzewanych o uszkodzenie**,
> - **wyrocznie mogą być omylne**,
> - **błędne podejrzenia nie mogą powstrzymywać poprawnych procesów** przed zachowaniem się zgodnie z ich specyfikacją.

### Własności — completeness i accuracy
<sub>slajdy 25–26</sub>

**Podstawowe** (slajd 25):
- **C** (*Completeness*) = w końcu **każdy $P_f$** jest podejrzewany przez **każdy** $P_c$,
- **A** (*Accuracy*) = **żaden $P_c$** **nigdy** nie jest podejrzewany przez żaden $P_c$.

**Pełna klasyfikacja** (slajd 26):

| Completeness | Accuracy |
|---|---|
| **SC** (*Strong Completeness*) = w końcu każdy $P_f$ podejrzewany przez **każdy** $P_c$ | **SA** (*Strong Accuracy*) = **żaden $P_c$ nigdy** nie jest podejrzewany przez żaden $P_c$ |
| **WC** (*Weak Completeness*) = w końcu każdy $P_f$ podejrzewany przez **pewien** $P_c$ | **WA** (*Weak Accuracy*) = **pewien $P_c$ nigdy** nie jest podejrzewany przez żaden $P_c$ |
| | **EA** (*Eventual Accuracy*) = **żaden $P_c$ w końcu** nie jest podejrzewany przez żaden $P_c$ |
| | **EWA** (*Eventual Weak Accuracy*) = **pewien $P_c$ w końcu** nie jest podejrzewany przez żaden $P_c$ |

> **Dymki ze slajdu:** przy **WC** — *każdy $P_f$ może być podejrzewany przez **inny** $P_c$*; przy **EWA** — ***ten sam*** *$P_c$ musi w końcu nie być podejrzewany przez wszystkie $P_c$*.

### Osiem klas detektorów
<sub>slajdy 27–28</sub>

![[swn-ft11-s26-klasy-fd-relacje.png]]
<sub>Relacja między klasami — wyjście z $\mathcal{P}$ (SC+SA): osłabienie completeness (**WC**) daje $\mathcal{Q}$ (WC+SA), osłabienie accuracy do EA daje $\Diamond\mathcal{P}$ (SC+EA), a do WA — $\mathcal{S}$ (SC+WA); slajd 27</sub>

![[swn-ft11-s27-klasy-fd-szescian.png]]
<sub>Pełny **sześcian ośmiu klas**: $\mathcal{P}$ (SC+SA), $\mathcal{Q}$ (WC+SA), $\Diamond\mathcal{P}$ (SC+EA), $\Diamond\mathcal{Q}$ (WC+EA), $\mathcal{S}$ (SC+WA), $\mathcal{W}$ (WC+WA), $\Diamond\mathcal{S}$ (SC+EWA), $\Diamond\mathcal{W}$ (WC+EWA); slajd 28</sub>

| Klasa | Skład | Nazwa |
|---|---|---|
| $\mathcal{P}$ | SC + SA | *perfect* |
| $\mathcal{Q}$ | WC + SA | — |
| $\mathcal{S}$ | SC + WA | *strong* |
| $\mathcal{W}$ | WC + WA | *weak* |
| $\Diamond\mathcal{P}$ | SC + EA | *eventually perfect* |
| $\Diamond\mathcal{Q}$ | WC + EA | — |
| $\Diamond\mathcal{S}$ | SC + EWA | *eventually strong* |
| $\Diamond\mathcal{W}$ | WC + EWA | *eventually weak* |

### $\Gamma$-dokładne detektory awarii
<sub>slajd 29</sub>

**Rozdział sieci:**
- własność dokładności **nie jest możliwa do osiągnięcia w przypadku podziału sieci** na rozłączne fragmenty (*partitioning*),
- własność **$\Gamma$-dokładności** (*$\Gamma$-accuracy*) dotyczy **jedynie procesów należących do pewnego podzbioru $\Gamma$**.

**Nowe klasy detektorów uszkodzeń:**
- **silna $\Gamma$-dokładność** — żaden poprawny proces w zbiorze $\Gamma$ nie jest podejrzewany przez inny proces z tego zbioru,
- **słaba $\Gamma$-dokładność** — pewien poprawny proces (niekoniecznie ze zbioru $\Gamma$) nie jest podejrzewany przez żaden proces ze zbioru $\Gamma$,
- $\Gamma_1 \subset \Gamma_2$: jeśli zachodzi własność $\Gamma$-dokładności w zbiorze $\Gamma_2$, to zachodzi też $\Gamma$-dokładność w zbiorze $\Gamma_1$,
- klasy: $\mathcal{P}(\Gamma)$, $\Diamond\mathcal{P}(\Gamma)$, $\mathcal{Q}(\Gamma)$, $\Diamond\mathcal{Q}(\Gamma)$, $\mathcal{S}(\Gamma)$, $\Diamond\mathcal{S}(\Gamma)$, $\mathcal{W}(\Gamma)$, $\Diamond\mathcal{W}(\Gamma)$.

### Redukcja i równoważność
<sub>slajdy 30–32</sub>

**Redukcja** (slajd 30):
- **algorytm transformacji**: $T_{\mathcal{D} \rightarrow \mathcal{D}'}$,
- **redukcja**: $\mathcal{D} \succeq \mathcal{D}'$ ($\mathcal{D}$ „emuluje" $\mathcal{D}'$; $\mathcal{D}'$ „jest słabszy niż" $\mathcal{D}$),
- przykład: $SC \succeq WC$ (*oczywiste: pusta transformacja*).

**Równoważność:**
$$(\mathcal{D} \succeq \mathcal{D}' \wedge \mathcal{D}' \succeq \mathcal{D}) \Rightarrow \mathcal{D} \cong \mathcal{D}'$$
przykład: $WC \succeq SC \;\Rightarrow\; SC \cong WC$.

**Algorytm transformacji z WC do SC** (slajd 31):

```
C_i:
    output_i <- {}                      // output_i will emulate D'_i

    || Task1: repeat forever
        {
          suspected_i <- D_i            // C_i queries its local FD module
          send(i, suspected_i) to all C
        }

    || Task2: when receive(j, suspected_j) from some C_j
        {
          output_i <- (output_i UNION suspected_j) - P_j
        }
```

**Wnioski z sześcianu** (slajd 32):
- zachodzi $\mathcal{P} \cong \mathcal{Q}$, $\mathcal{S} \cong \mathcal{W}$, $\Diamond\mathcal{P} \cong \Diamond\mathcal{Q}$, $\Diamond\mathcal{S} \cong \Diamond\mathcal{W}$; a także $\mathcal{P} \succ \mathcal{S}$, $\mathcal{P} \succ \Diamond\mathcal{P}$, $\mathcal{S} \succ \Diamond\mathcal{S}$ itd.,
- **klasy po przekątnej są nieporównywalne**: $\Diamond\mathcal{P}$ i $\mathcal{S}$, $\Diamond\mathcal{Q}$ i $\mathcal{W}$, $\Diamond\mathcal{P}$ i $\mathcal{W}$, $\Diamond\mathcal{Q}$ i $\mathcal{S}$,
- ponadto $\Diamond\mathcal{S}(\Gamma) \cong \Diamond\mathcal{S}$, ale $\Diamond\mathcal{W} \succ \Diamond\mathcal{W}(\Gamma)$ oraz $\mathcal{P} \succ \mathcal{P}(\Gamma)$.

---
## Równoważność C $\cong$ TOcast w praktyce

### Redukcje
<sub>slajd 33</sub>

| Redukcja | Jak |
|---|---|
| **TOcast $\succeq$ C** | każdy $P_c$ decyduje o **pierwszej wartości dostarczonej w porządku TO** |
| **C $\succeq$ TOcast** | **okresowy konsensus** nad podzbiorami komunikatów jeszcze niedostarczonych + **a priori deterministyczny** porządek dostarczania w obrębie podzbioru |
| **C $\succeq$ VScast** | **okresowy konsensus** nad podzbiorami dostarczonych komunikatów **i składem następnego widoku $v_{i+1}$** + a priori deterministyczny porządek dostarczania komunikatów wysłanych w widoku $v_i$ |

### Rozwiązanie TOcast przy użyciu konsensusu
<sub>slajdy 34–35</sub>

```
C_i:
    init:
        R_delivered  = {}
        TO_delivered = {}
        k = 0

    TOcast(m):
        RBcast(m)

    TOdeliver() occurs as follows:
     || when Rdeliver(m)
            R_delivered = R_delivered UNION {m}

     || when R_delivered - TO_delivered != {}
            k++
            TO_undelivered = R_delivered - TO_delivered
            propose(k, TO_undelivered)
            wait until decide(k, msgSet^k)
            TO_deliver^k = msgSet^k - TO_delivered
            deliver all msg from TO_deliver^k in deterministic order
            TO_delivered = TO_delivered UNION TO_deliver^k
```

---
## Rozwiązywanie konsensusu detektorem awarii

### FD $\in \mathcal{S}$
<sub>slajd 36</sub>

- $\mathcal{S} = $ SC + WA $\Rightarrow$ **co najmniej jeden $P_c$ nigdy nie podejrzewany** („autorytet"),
- **$N$ rund**:
  - $N-1$ propozycji,
  - rozgłoszenie ostatecznego zbioru propozycji,
  - „wyliczana" **część wspólna** (*de facto* decyduje autorytet),
  - **decyzja = pierwsza pozostała propozycja**,
- $f < N$,
- **eventual reliable channels**.

> *Remember:* $\mathcal{S} \cong \mathcal{W}$.

> Pełny pseudokod tego algorytmu (Figure 5 z pracy Chandry-Touega) jest w [[SWN 07 Konsensus#Rozwiązanie konsensusu z użyciem dowolnego $\mathcal{D} \in \mathcal{S}$]].

### FD $\in \Diamond\mathcal{S}$
<sub>slajd 37</sub>

- $\Diamond\mathcal{S} = $ SC + EWA $\Rightarrow$ **na początku każdy proces może być podejrzewany** (nie ma autorytetu),
- **liczba rund nie jest ograniczona** (do skutku):
  - każdy z procesów kolejno staje się **koordynatorem rundy** (*rotating coordinator paradigm*),
  - koordynator czeka na zebranie **kworum $Q_c$**,
  - **decyzja rozgłaszana przez RBcast**,
- $f < \left\lceil \frac{N}{2} \right\rceil$, gdyż $Q_c = \left\lfloor \frac{N}{2} \right\rfloor + 1$ (**większość**),
- **eventual reliable channels**.

> *Remember:* $\Diamond\mathcal{S} \cong \Diamond\mathcal{W}$.

> Pełny pseudokod (Figure 6 z pracy Chandry-Touega) — [[SWN 07 Konsensus#Rozwiązanie konsensusu z użyciem dowolnego $\mathcal{D} \in \Diamond\mathcal{S}$]].

### Uniform Consensus
<sub>slajd 38</sub>

> **Każdy protokół rozwiązujący konsensus przy użyciu $\Diamond\mathcal{S}$ rozwiązuje również Uniform Consensus.**

---
## Problemy pokrewne

### TRBcast
<sub>slajdy 39–40</sub>

*Czy istnieją inne znane problemy, których rozwiązania wymagają FD silniejszego niż $\Diamond\mathcal{S}$?*

> **TRBcast (Terminating Reliable Broadcast):**
> **Termination:** każdy $P_c$ w końcu dostarcza **dokładnie jeden** komunikat.
> **Validity:** jeśli $P_s$ rozgłasza $m$ **i pozostaje poprawny** $\Rightarrow$ w końcu dostarcza $m$.
> **Agreement:** jeśli jakiś $P_c$ dostarcza $m$ $\Rightarrow$ wszystkie $P_c$ dostarczają $m$.
> **Integrity:** $m$ zostało rozgłoszone przez $P_s$ i jest dostarczane co najwyżej raz.

Istnieje **wyróżniony proces $P_s$**, który ma rozgłosić pojedynczy komunikat. Problem TRBcast jest podobny do RBcast, **z tym że wymaga, by każdy $P_c$ zawsze dostarczył jakiś komunikat** — nawet jeśli $P_s$ jest wadliwy i np. pada przed rozgłoszeniem. W takim przypadku pozwalamy procesom dostarczyć **specjalny komunikat $m_F$** (tj. *null*), który w istocie nie został rozgłoszony (i dowodzi, że $P_s$ uległ awarii). Umownie przyjmujemy $sender(m_F) = P_s$.

> *Ten problem jest w istocie **równoważny porozumieniu bizantyjskiemu**.*

**Twierdzenia** (slajd 40):
1. **TRB da się rozwiązać** w systemach asynchronicznych przy **dowolnej liczbie awarii** typu crash, **używając $\mathcal{P}$**.
2. **TRB nie da się rozwiązać** w systemach asynchronicznych przy użyciu $\Diamond\mathcal{P}$, $\mathcal{S}$ ani $\Diamond\mathcal{S}$, **nawet przy założeniu co najwyżej jednej awarii**.

*W istocie $\mathcal{P}$ jest **najsłabszym FD** wymaganym do rozwiązania powtarzanych instancji TRB (wiele instancji z każdym procesem jako $P_s$).*

**Komentarz** (slajd 41): kluczowa różnica między TRB a RBcast polega na tym, że **TRB wymaga ostatecznego dostarczenia jakiegoś komunikatu** — nawet jeśli nadawca padnie tuż przed rozgłoszeniem (wtedy wysyłany jest komunikat *null*). Zatem **TRB wymaga rozpoznania awarii** (czyli **detekcji awarii**), w przeciwieństwie do sytuacji, gdy po prostu żaden komunikat nie zostaje wysłany. Jest to więc **równoważne zdolności odróżnienia procesu wolnego od procesu, który uległ awarii**.

---
## Podsumowanie detektorów awarii

### Detektory awarii jako narzędzie
<sub>slajd 42</sub>

**Zalety:**
- **naturalne** — autentyczne rozszerzenie modelu systemu asynchronicznego $\mathbb{S}^{Async}$,
- **minimalne** — minimalna dodatkowa informacja wystarczająca do rozwiązania konsensusu: $\mathbb{S}^{Async}\{FD\}$,
- **proste** — łatwe do zrozumienia i użycia,
- **przenośne** — dają się użyć do rozwiązania ogromnego zakresu problemów (członkostwo w grupie, elekcja, zatwierdzanie atomowe, …),
- **efektywne** — dla małego $f$.

**Wada:**
- **skomplikowane algorytmy** — muszą radzić sobie z **błędnymi podejrzeniami** detektora.

### Detektory awarii jako narzędzie teoretyczne
<sub>slajd 43</sub>

**Nieimplementowalne w $\mathbb{S}^{Async}$:** własności FD muszą zachodzić **na zawsze** (ewentualnie od pewnego nieznanego, ale skończonego momentu).

**W praktyce — implementacja zbliżona do własności FD:** muszą zachodzić **wystarczająco długo** (tj. aby rozwiązać problem).

> **Algorytm *indulgent*** oparty na FD **kończy się i produkuje poprawny wynik, jeśli FD zachowuje się zgodnie ze swoją specyfikacją. Jeśli FD nie spełnia swojej specyfikacji, algorytm może się nie zakończyć, ale jeśli się zakończy — zawsze produkuje poprawny wynik.**

- zakładamy istnienie **okresów stabilności** (*stable periods*), podczas których własności FD zachodzą,
- **podczas okresów stabilności konsensus i replikacja (aktywna i pasywna) są możliwe**.

---
## Bibliografia
<sub>slajd 44</sub>

1. M. J. Fischer, N. A. Lynch, M. S. Paterson, *Impossibility of distributed consensus with one faulty process*, Journal of the ACM no. 32, 1985, pp. 374–382.
2. T. D. Chandra, S. Toueg, *Unreliable Failure Detectors for Reliable Distributed Systems*, Proc. 10th ACM Symposium on Principles of Distributed Computing, ACM Press, 1991.
3. J. Kobusiński, A. Szajkowska, M. Szychowiak, *Detektory uszkodzeń a niezawodna komunikacja grupowa*, VI Konferencja POLMAN '99, 1999 ([link](https://www.cs.put.poznan.pl/mszychowiak/publications/Polman99-FD.ps.gz)).
4. J. Brzeziński, J. Kobusiński, *A Survey of Software Failure Detector Protocols*, Foundations of Computing and Decision Sciences, vol. 28, no. 2, 2003.
5. G. Tel, *Introduction to Distributed Algorithms*, Cambridge University Press, 2000, ch. 16.
6. A. D. Kshemkalyani, M. Singhal, *Distributed Computing — Principles, Algorithms, and Systems*, Cambridge University Press, 2008, ch. 15.
7. M. Raynal, *Communication and Agreement Abstractions for Fault-Tolerant Asynchronous Distributed Systems*, Morgan & Claypool Pub., 2010.

---
## Powiązania i braki

> [!note] Uzupełnienie spoza slajdów
> [[Systemy Wysokiej Niezawodności/Replikacja Procesu]] pokrywa się ze slajdami 3–12 niemal zdanie w zdanie. Dwa dopowiedzenia, których slajdy nie mają wprost: w replikacji aktywnej **„jeśli istnieje jakaś działająca replika, klient otrzyma odpowiedź"** oraz w pasywnej **„primary przesyła odpowiedź do klienta po tym, jak wszystkie backupy odpowiedzą primary"**.

> [!warning] Rozbieżność z notatką w vaultcie
> [[Systemy Wysokiej Niezawodności/Replikacja Procesu]] linkuje do `[[Reliable Broadcast]]` przy operacji atomowego uaktualnienia. W katalogu głównym vaultu **istnieje plik `Reliable Broadcast.md`** (nieśledzony w gicie), więc link nie wisi — ale definicja RBcast z tego wykładu (slajd 14) jest **kompletniejsza**: podaje wszystkie trzy warunki oraz rozróżnienie RBcast / UBcast, którego w vaultcie nie ma.

> [!todo] Czego nie ma na slajdach tego wykładu
> - **Dowodu redukcji C $\succeq$ VScast** — slajd 33 opisuje ją słownie, pseudokodu nie ma (dla TOcast jest, slajd 35).
> - **Algorytmów implementacji detektorów awarii** (heartbeat, ping-ack, adaptacyjne timeouty) — są tylko własności i klasy; por. [4] w bibliografii.
> - **Dowodów twierdzeń o TRBcast** (slajd 40 podaje je bez uzasadnienia).
> - **Dowodu, że $\Diamond\mathcal{W}$ jest najsłabszym detektorem wystarczającym do konsensusu** — pojawia się tylko na diagramie rozwiązywalności (slajd 41).
