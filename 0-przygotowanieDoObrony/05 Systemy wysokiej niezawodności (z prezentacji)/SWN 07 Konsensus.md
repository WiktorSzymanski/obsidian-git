---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
source: "Slajdy-FT-12_Consensus.pdf + FT_ConsensusFD.pdf"
slajdy: "1–45 + 2 strony"
zagadnienie: 23
---
# SWN 07. Konsensus — Paxos, rozwiązania probabilistyczne, konsensus bizantyjski
---
> Wykład zbiera **trzy drogi obejścia FLP** i pokazuje po jednym algorytmie dla każdej: **Paxos** (rezygnacja z terminacji), **Bracha-Toueg** (randomizacja typu Las Vegas) i **Phase-King** (silniejsza synchronia + ograniczenie $f < \frac{N}{4}$ dla procesów bizantyjskich). Kończy się umiejscowieniem konsensusu w **twierdzeniu CAP**. Notatkę uzupełnia `FT_ConsensusFD.pdf` — dwie figury z pracy **Chandry-Touega** o konsensusie z detektorami awarii, których na slajdach nie ma.

---
## Definicja konsensusu i powtórka
<sub>Slajdy-FT-12_Consensus.pdf, slajdy 1–3 (oznaczone REPLAY)</sub>

> **Termination:** wszystkie $P_c$ decydują dokładnie 1 wartość $v_i$.
> **Agreement:** wszystkie $P_c$ decydują to samo $v_i$.
> **Validity:** $v_i$ zostało zaproponowane przez pewien $P_i$.

**FLP'85** (slajd 2): nie istnieje deterministyczne rozwiązanie konsensusu w systemie asynchronicznym, jeśli choćby jeden proces może ulec awarii typu crash.

**Obejścia** (slajd 3):
1. **Synchronia** — model systemu synchronicznego (quasi-synchronicznego),
2. **Słabszy model awarii** — np. w modelu *initially dead-processes*, słabszym niż *fail-stop*, konsensus i elekcja są osiągalne **deterministycznie**,
3. **Poświęcenie *liveness* na rzecz *safety*** = osłabienie warunku terminacji, np.
   - **(3')** randomizacja: *Termination: every $P_c$ eventually decides with probability 1*,
   - **(3'')** albo **Paxos**.

> Szczegóły obejść (1) i (2) — [[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Obejścia]] i [[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Model procesów początkowo martwych]].

---
## Paxos
<sub>slajdy 4–8</sub>

### Założenia modelu i idea
<sub>slajd 4</sub>

**Założenia (początkowe):**
- $\mathbb{S}^{Async}\{\varnothing\}$,
- awarie procesów typu **fail-stop**, $f < \frac{N}{2}$, tj. co najmniej $M = \left\lceil \frac{N+1}{2} \right\rceil$ procesów jest poprawnych,
- **niezawodne kanały** komunikacyjne.

**Ogólna idea:**
- propozycja zostaje wybrana jako decyzja, jeśli jest **zaakceptowana przez większość** procesów,
- każda propozycja ma **unikalny numer** $pn_i$,
- **tylko propozycja o najwyższym numerze $pn_i$ wygrywa** (tj. zostaje *accepted*),
- propozycja raz **zaakceptowana pozostaje taką**,
- $\mathbb{S}^{Async}\{\text{uniqID}\}$: $pn_i = i$.

### Paxos basic
<sub>slajd 5</sub>

1. $P_i$ rozgłasza żądanie `PREPARE` z $pn_i$ do wszystkich (włącznie z sobą¹).
2. Dowolny odbiorca $P_j$ odpowiada na `PREPARE`:
   - `OK` — **obietnica, że nigdy więcej nie zaakceptuje propozycji o numerze mniejszym niż $pn_i$** — jeśli $pn_i$ jest większe od numeru dowolnego `PREPARE`, na które już odpowiedział,
   - **albo** `ACCEPTED` z propozycją $v_k$, którą $P_j$ już zaakceptował z $pn_k$ ($pn_k$ może być mniejsze niż $pn_i$).
3. Jeśli $P_i$ otrzyma odpowiedź (`OK` lub `ACCEPTED`) od **większości $M$** procesów, to:
   - ustawia $v_i = v_k$ z **najwyższym $pn_k$** spośród wszystkich odpowiedzi `ACCEPTED`, jeśli takie są,
   - i w tym przypadku ustawia również $pn_i = pn_k$,
   - następnie rozgłasza `PROPOSE` z $v_i$ i $pn_i$ do wszystkich.
4. Gdy $P_j$ odbiera `PROPOSE` z $v_i$ i $pn_i$, **akceptuje propozycję $v_i$**, chyba że już odpowiedział na żądanie `PREPARE` z wyższym $pn_k$.
5. Jeśli $P_i$ zaakceptował wartość $v_i$, rozgłasza komunikat `ACCEPTED` z $v_i$ i $pn_i$.
6. Gdy $P_i$ odbiera `ACCEPTED` z **tym samym** $v_j$ od $M$ procesów, **decyduje $v_j$**.

> ¹ *W rzeczywistości $P_i$ nie musi wysyłać komunikatów do siebie, ale może po prostu włączyć rozgłaszany komunikat bezpośrednio do zbioru komunikatów, które odbiera później.*

### Założenia modelu (cd.)
<sub>slajd 6</sub>

- **awarie przemilczenia w kanałach komunikacyjnych** $\rightarrow$ $P_i$ wysyła rozgłoszenia za pomocą **RBcast**,
- **model awarii fail-recovery** $\rightarrow$ $\mathbb{S}^{Async}\{\text{stable storage}\}$: $P_i$ musi pamiętać w pamięci trwałej **$v_k$, które zaakceptował** (krok 5) **albo najwyższe $pn_k$, które zgodził się zaakceptować** (krok 2).

### Terminacja
<sub>slajdy 7–8</sub>

- **Paxos = rezygnacja z warunku Termination**,
- jeśli proces wygrywający krok 3 i mający właśnie zgłosić propozycję **ulegnie awarii** $\rightarrow$ **livelock**,
- jeśli pozwolimy procesom restartować (ponawiać swoje niedoszłe propozycje) z **inkrementowanym $pn_i$**, może to nigdy się nie skończyć (nigdy nie ma absolutnego zwycięzcy w kroku 3) $\rightarrow$ **nadal livelock**,
- to jednak wymaga $\mathbb{S}^{Async}\{\text{Election}\}$ $\rightarrow$ **Paxos extended** [3], **Raft** [4],
- wówczas, aby zagwarantować postęp, **wyróżniony proces może zostać wybrany jako jedyny wydający `PROPOSE`** (w kroku 3),
- dla wielu (współbieżnych) instancji Paxosa $\rightarrow$ **Paxos multi** [2].

> **Impossibility Result for termination in $\mathbb{S}^{Async}\{\varnothing\}$** (slajd 8):
> **Nie istnieje 1-odporne fail-stop rozwiązanie konsensusu, które zawsze się kończy.**

---
## Rozwiązania probabilistyczne

### Monte Carlo i Las Vegas
<sub>slajd 9</sub>

> Algorytm probabilistyczny jest **Monte Carlo**, jeżeli: **zawsze się kończy** oraz **prawdopodobieństwo uzyskania poprawnej konfiguracji końcowej jest większe od zera**.

> Algorytm probabilistyczny jest **Las Vegas**, jeżeli: **kończy się z prawdopodobieństwem większym od zera** oraz **wszystkie konfiguracje końcowe są poprawne**.

### Wyniki niemożliwości
<sub>slajdy 10–11</sub>

> **Nie istnieje 1-odporne fail-stop rozwiązanie Monte Carlo konsensusu.**

> **Nie istnieje $f$-odporne fail-stop rozwiązanie Las Vegas konsensusu dla $f \geqslant \frac{N}{2}$.**
>
> *Jeśli $f < \frac{N}{2}$, to $f$-crash odporne rozwiązanie Las Vegas dla konsensusu **istnieje**.*

### Algorytm Bracha-Touega — model i idea
<sub>slajd 12</sub>

**Las Vegas binary consensus algorithm (Bracha-Toueg [5])**

**Model:** $\mathbb{S}^{Async}\{\varnothing\}$; awarie procesów **fail-stop**, $f < \frac{N}{2}$; **niezawodne kanały**.

**Ogólna idea:**
- początkowo (na starcie rundy 0) każdy proces **losowo wybiera wartość 0 lub 1** jako $v_i$,
- z **wagą** $w_i = 1$,
- proces, który jeszcze nie zdecydował, wyznacza nowe $v_i$ i $w_i$ na podstawie **pierwszych $N-f$ komunikatów**, które odbiera w danej rundzie,
- **waga przybliża liczbę procesów, które w poprzedniej rundzie głosowały $v = v_i$**.

### Algorytm
<sub>slajd 13</sub>

**Runda $k \geqslant 0$:**
- $P_i$ rozgłasza $\langle k, v_i, w_i \rangle$ do wszystkich (włącznie z sobą²),
- $P_i$ czeka na $\langle k, v_j, w_j \rangle$ od **$N-f$ procesów**,
- jeśli $w_j > \frac{N}{2}$ dla komunikatu przychodzącego $\langle k, v_j, w_j \rangle$, to $v_i = v_j$,
- jeśli $w_j > \frac{N}{2}$ dla **więcej niż $f$** komunikatów przychodzących, to $P_i$ **decyduje $v_i$**, rozgłasza $\langle k+1, v_i, N-f \rangle$ oraz $\langle k+2, v_i, N-f \rangle$, a następnie **kończy**,
- jeśli $w_j \leqslant \frac{N}{2}$, to $v_i = majority(v_j)$, a $w_i$ = liczba przychodzących głosów $v = majority(v_j)$ (domyślnie 0, jak zwykle),
- **gdy proces decyduje, wszystkie pozostałe procesy poprawne mają gwarancję zdecydowania w ciągu 2 rund**.

> ² *Ponownie, w rzeczywistości $P_i$ nie musi wysyłać komunikatów do siebie, ale może wirtualnie włączyć rozgłaszany komunikat do zbioru komunikatów, które odbiera w rundzie (odpowiednio obliczając $w_i$).*

### Przykład
<sub>slajdy 14–27</sub>

$N = 3$, $f = 1$. *Każda runda: poprawny proces wymaga **2 komunikatów przychodzących** ($N-f$) i **2 głosów $v$ o wadze 2** ($> f$ komunikatów o wadze $> \frac{N}{2}$), by zdecydować $v$.* Początkowo $P_1$ i $P_2$ losowo wybierają 1, a $P_3$ wybiera 0.

![[swn-ft12-s19-bracha-toueg-runda0.png]]
<sub>Runda $k = 0$ — $P_1$ odbiera pierwsze 2 komunikaty (od siebie i od $P_3$) i zmienia $v_1$ na $majority(v_j) = 0$; $P_2$ i $P_3$ odbierają od $P_1$ i $P_2$, uzyskując $v_2 = 1, w_2 = 2$ i $v_3 = 1, w_3 = 2$ — slajdy 19–20</sub>

![[swn-ft12-s21-bracha-toueg-runda1.png]]
<sub>Runda $k = 1$ — $P_2$ odbiera komunikaty od $P_2$ i $P_3$ (oba o wadze 2) i **decyduje 1**; $P_1$ i $P_3$ odbierają od $P_1$ i $P_3$ — slajd 21</sub>

![[swn-ft12-s25-bracha-toueg-awaria-p2.png]]
<sub>Runda $k = 3$ — $P_2$ **uległ awarii** (już zdecydował i zakończył); pozostałe procesy kontynuują z $v = 1$, $w = 2$ — slajd 25</sub>

### Poprawność
<sub>slajdy 28–31</sub>

> **Theorem 1.** Dla $f < \frac{N}{2}$ algorytm konsensusu Bracha-Touega jest algorytmem **Las Vegas kończącym się z prawdopodobieństwem 1**.

**Dowód, cz. 1** (slajdy 28–30): *najpierw dowodzimy, że procesy nie mogą zdecydować różnych wartości.*

**Dowód, cz. 2 — terminacja** (slajd 31):
- ponieważ kanały są niezawodne, istnieje szansa $\rho > 0$, że w każdej rundzie $k$ **wszystkie procesy odbiorą pierwsze $N-f$ komunikatów od tych samych procesów**,
- po tej rundzie $k$ wszystkie procesy poprawne mają **tę samą wartość $v$**,
- po rundzie $k+1$ wszystkie procesy poprawne mają tę samą wartość $v$ z $w = N-f$,
- po rundzie $k+2$ wszystkie procesy poprawne **zdecydowały wartość $v$**,
- zatem algorytm kończy się z prawdopodobieństwem 1. $\blacksquare$

> [!warning] Rozbieżność z notatkami w vaultcie
> [[Braki w notatkach]] i [[Mapa zagadnień]] odnotowują ostrzeżenie, że [[Systemy Wysokiej Niezawodności/Algorytm Bracha-Touega]] opisuje **algorytm konsensusu, a nie detekcji zakleszczenia**. Slajdy to potwierdzają: **Bracha-Toueg na tym wykładzie to algorytm konsensusu binarnego typu Las Vegas**. Algorytm Brachy-Touega dla detekcji zakleszczenia to **inny algorytm tych samych autorów**, w prezentacjach SWN **nieobecny**.

---
## Konsensus procesów bizantyjskich — algorytm Phase-King

### Model i idea
<sub>slajd 32</sub>

**Phase-King algorithm (Berman-Garay [6]) for binary consensus**

**Założenia modelu:**
- $\mathbb{S}^{Sync}\{\varnothing\}$,
- awarie **bizantyjskie**, $\mathbf{f < \frac{N}{4}}$ procesów złośliwych,
- **niezawodne kanały** komunikacyjne.

**Ogólna idea:**
- **$f+1$ faz** (każda z dwoma krokami),
- **rotujący koordynator**: w każdej fazie liderem jest inny proces (***phase king***).

### Algorytm
<sub>slajd 33</sub>

```
for phase = 1 to f+1 do
    Execute the following step 1 actions:
        broadcast v_i to all processes;
        await value v_j from each process P_j;
        majority = majority(v_1, ..., v_N);
        w_i = number of times that majority occurs;

    Execute the following step 2 actions:
        if i == phase then
            broadcast majority to all processes;
        receive tiebreaker from P_phase
            (default value if nothing is received);
        if w_i <= N/2 + f then v_i = tiebreaker;
        else v_i = majority;
        if phase == f+1 then decide value v_i.
```

![[swn-ft12-s34-phase-king-rundy.png]]
<sub>**Wielomianowa liczba komunikatów** — przebieg faz 1, 2, …, $f+1$ z rotującym koordynatorem $P_{f+1}$, slajd 34</sub>

### Krok 1 — głosowanie
<sub>slajdy 35–36</sub>

- w pierwszym kroku każdej fazy każdy proces **rozgłasza swoje oszacowanie** wartości konsensusu do wszystkich pozostałych procesów,
- i podobnie **czeka na wartości** rozgłoszone przez innych,
- jeśli jakakolwiek wartość występuje **więcej niż $\frac{N}{2}$ razy**, to $P_i$ ustawia swoją zmienną `majority` na tę wartość,
- i ustawia $w_i$ na **liczbę głosów** otrzymanych na wartość większościową,
- jeśli żaden głos nie występuje więcej niż $\frac{N}{2}$ razy (**co może się zdarzyć, gdy procesy złośliwe nie odpowiadają, a procesy poprawne są między sobą podzielone**), to **używana jest wartość domyślna** (powiedzmy 0, jak zwykle).

### Krok 2 — tie-breaker króla fazy
<sub>slajdy 37–40</sub>

- w drugim kroku każdej fazy **król fazy inicjuje przetwarzanie** (królem fazy $P_{phase}$ dla fazy $k$ jest proces $P_k$),
- rozgłasza swoją wartość większościową `majority`, która pełni rolę **głosu rozstrzygającego** (*tie-breaker*) dla tych procesów, które nie uzyskały swojego $w_i$ większego niż $\frac{N}{2} + f$,
- gdy $P_i$ odbiera tie-breaker, **aktualizuje swoje oszacowanie $v_i$ do wartości wysłanej przez króla fazy**, jeśli jego własne $w_i \leqslant \frac{N}{2} + f$,
- **ponieważ wśród głosów na jego własną wartość `majority` $f$ głosów mogło być fałszywych**,
- zatem $P_i$ **nie ma wyraźnej większości głosów** (tj. $> \frac{N}{2}$) od procesów poprawnych,
- dlatego $P_i$ **przyjmuje wartość króla fazy**.

**Natomiast** (slajd 39): jeśli $w_i > \frac{N}{2} + f$, to otrzymał **wyraźną większość głosów od procesów niezłośliwych**, więc aktualizuje swoje oszacowanie $v_i$ do **własnej wartości większościowej**, **niezależnie od tego, jaki tie-breaker wysłał król fazy**.

**Na końcu $f+1$ faz** (slajd 40): oszacowanie $v_i$ wszystkich procesów poprawnych jest **poprawną wartością konsensusu**.

### Dowód poprawności
<sub>slajdy 41–43</sub>

**1)** Spośród $f+1$ faz:
- król fazy pewnej fazy $k$ jest **niezłośliwy**,
- ponieważ jest co najwyżej $f$ procesów złośliwych.

**2)** Ponieważ król tej fazy $k$ jest niezłośliwy, **wszystkie niezłośliwe $P_i$ mają tę samą wartość oszacowania $v_i$ na końcu fazy $k$** — dowolne dwa procesy niezłośliwe $P_i$ i $P_j$ mogą ustawić swoje oszacowanie $v$ na trzy sposoby:

**(a)** oba $P_i$ i $P_j$ używają **własnych** wartości `majority`:
- załóżmy, że wartość `majority` procesu $P_i$ to $x$, co implikuje, że $w_i > \frac{N}{2} + f$,
- a spośród tych głosujących co najmniej $\frac{N}{2}$ jest niezłośliwych,
- to implikuje, że $P_j$ musiał także otrzymać co najmniej $\frac{N}{2}$ głosów na $x$,
- co implikuje, że jego wartość `majority` również musi być $x$.

**(b)** oba $P_i$ i $P_j$ używają wartości tie-breaker króla fazy $P_k$:
- ponieważ $P_k$ jest niezłośliwy, **musiał wysłać tę samą wartość tie-breaker do obu** $P_i$, $P_j$.

**(c)** $P_i$ używa swojej wartości `majority` jako nowego oszacowania, a $P_j$ używa tie-breakera króla fazy:
- załóżmy, że wartość `majority` procesu $P_i$ to $x$,
- skoro $P_i$ używa swojej wartości większościowej $x$, to $w_i > \frac{N}{2} + f$,
- a spośród tych głosujących co najmniej $\frac{N}{2}$ jest niezłośliwych,
- to implikuje, że $P_k$ musiał także otrzymać co najmniej $\frac{N}{2}$ głosów na $x$,
- co implikuje, że jego wartość `majority`, którą wysyła jako tie-breaker, również musi być $x$.

Dla wszystkich trzech możliwości dowolne dwa procesy niezłośliwe $P_i$ i $P_j$ **uzgadniają oszacowanie konsensusu na końcu fazy $k$**, w której król $P_k$ jest niezłośliwy.

**3)** Wszystkie procesy niezłośliwe mają to samo oszacowanie konsensusu $x$ na początku fazy $k+1$:
- i **nadal mają to samo oszacowanie na końcu fazy $k+1$**,
- jest to oczywiste, ponieważ mamy **$N > 4f$**,
- i każdy proces niezłośliwy otrzymuje co najmniej $N - f > \frac{N}{2} + f$ głosów na $x$ od pozostałych procesów niezłośliwych w pierwszym kroku fazy $k+1$,
- zatem wszystkie procesy niezłośliwe **zachowują swoje oszacowanie $v$ jako $x$** na końcu fazy $k+1$.

*Ta sama logika obowiązuje dla wszystkich kolejnych faz. Zatem wartość konsensusu jest poprawna.* $\blacksquare$

> [!important] Dlaczego $f < \frac{N}{4}$, a nie $f < \frac{N}{3}$
> Warunek $N > 4f$ jest **silniejszy** niż warunek $N > 3f$ z algorytmu OM ([[SWN 06 Awarie bizantyjskie]]). Phase-King płaci tym za **wielomianową liczbę komunikatów** — OM osiąga optymalne $N > 3f$, ale kosztem $\Omega(N^{f+1})$ komunikatów.

---
## Konsensus a CAP
<sub>slajd 44 (REPLAY)</sub>

![[swn-ft12-s44-cap-impossibility.png]]
<sub>CAP Impossibility: **2PC** leży na przecięciu Consistency i Availability, **Gossip** — Availability i Partition tolerance, a **Paxos** (wskazany strzałką) — **Consistency i Partition tolerance**; slajd 44</sub>

---
## Konsensus z detektorami awarii
<sub>FT_ConsensusFD.pdf, s. 1–2 (Figure 5 i Figure 6 z pracy Chandry-Touega)</sub>

Dwie figury dołączone do materiałów kursu — **nieobecne w prezentacjach** — podają algorytmy konsensusu wykorzystujące **detektory awarii**.

### Rozwiązanie konsensusu z użyciem dowolnego $\mathcal{D} \in \mathcal{S}$
<sub>FT_ConsensusFD.pdf, s. 1, Figure 5</sub>

Każdy proces $p$ wykonuje:

```
procedure propose(v_p)
    V_p  <- <bot, bot, ..., bot>        {p's estimate of the proposed values}
    V_p[p] <- v_p
    D_p  <- V_p

    Phase 1:  {asynchronous rounds r_p, 1 <= r_p <= n-1}
        for r_p <- 1 to n-1
            send (r_p, D_p, p) to all
            wait until [for all q: received (r_p, D_q, q) or q in FD_p]
                                          {query the failure detector}
            msgs_p[r_p] <- { (r_p, D_q, q) | received (r_p, D_q, q) }
            D_p <- <bot, bot, ..., bot>
            for k <- 1 to n
                if V_p[k] = bot and exists (r_p, D_q, q) in msgs_p[r_p]
                                    with D_q[k] != bot then
                    V_p[k]  <- D_q[k]
                    D_p[k]  <- D_q[k]

    Phase 2:  send V_p to all
        wait until [for all q: received V_q or q in FD_p]
                                          {query the failure detector}
        lastmsgs_p <- { V_q | received V_q }
        for k <- 1 to n
            if exists V_q in lastmsgs_p with V_q[k] = bot then V_p[k] <- bot

    Phase 3:  decide( first non-bot component of V_p )
```

- **Faza 1** to $n-1$ **rund asynchronicznych**, w których procesy wymieniają **różnice** $\Delta_p$ swoich wektorów oszacowań; czekanie kończy się, gdy komunikat dotarł **albo** detektor awarii $\mathcal{D}_p$ podejrzewa nadawcę.
- **Faza 2** usuwa z wektora te pozycje, których **którykolwiek** proces nie zdołał wypełnić — to zapewnia, że wszystkie procesy poprawne mają **identyczny** wektor.
- **Faza 3** decyduje o **pierwszej niepustej** składowej wektora.

### Rozwiązanie konsensusu z użyciem dowolnego $\mathcal{D} \in \Diamond\mathcal{S}$
<sub>FT_ConsensusFD.pdf, s. 2, Figure 6</sub>

Wariant z **rotującym koordynatorem** $c_p = (r_p \bmod n) + 1$, czterema fazami w rundzie i **kworum $\left\lceil \frac{n+1}{2} \right\rceil$**:

```
procedure propose(v_p)
    estimate_p <- v_p       {estimate_p is p's estimate of the decision value}
    state_p    <- undecided
    r_p        <- 0                        {r_p is p's current round number}
    ts_p       <- 0    {ts_p is the last round in which p updated estimate_p}

    {Rotate through coordinators until decision is reached}
    while state_p = undecided
        r_p <- r_p + 1
        c_p <- (r_p mod n) + 1                {c_p is the current coordinator}

        Phase 1: {All processes p send estimate_p to the current coordinator}
            send (p, r_p, estimate_p, ts_p) to c_p

        Phase 2: {The current coordinator gathers ceil((n+1)/2) estimates
                  and proposes a new estimate}
            if p = c_p then
                wait until [for ceil((n+1)/2) processes q:
                            received (q, r_p, estimate_q, ts_q) from q]
                msgs_p[r_p] <- { (q, r_p, estimate_q, ts_q) | received ... }
                t <- largest ts_q such that (q, r_p, estimate_q, ts_q) in msgs_p[r_p]
                estimate_p <- select one estimate_q such that
                              (q, r_p, estimate_q, t) in msgs_p[r_p]
                send (p, r_p, estimate_p) to all

        Phase 3: {All processes wait for the new estimate proposed by
                  the current coordinator}
            wait until [received (c_p, r_p, estimate_{c_p}) from c_p
                        or c_p in FD_p]          {Query the failure detector}
            if [received (c_p, r_p, estimate_{c_p}) from c_p] then
                estimate_p <- estimate_{c_p}
                ts_p <- r_p
                send (p, r_p, ack) to c_p
            else send (p, r_p, nack) to c_p    {p suspects that c_p crashed}

        Phase 4: {The current coordinator waits for ceil((n+1)/2) replies.
                  If they indicate that ceil((n+1)/2) processes adopted its
                  estimate, the coordinator R-broadcasts a decide message}
            if p = c_p then
                wait until [for ceil((n+1)/2) processes q:
                            received (q, r_p, ack) or (q, r_p, nack)]
                if [for ceil((n+1)/2) processes q: received (q, r_p, ack)] then
                    R-broadcast(p, r_p, estimate_p, decide)

{If p R-delivers a decide message, p decides accordingly}
when R-deliver(q, r_q, estimate_q, decide)
    if state_p = undecided then
        decide(estimate_q)
        state_p <- decided
```

> [!note] Uzupełnienie spoza slajdów
> Klasy detektorów awarii ($\mathcal{P}$, $\mathcal{S}$, $\Diamond\mathcal{S}$, $\Diamond\mathcal{W}$) i ich własności (**completeness**, **accuracy**) są omówione na wykładzie o replikacji procesu — [[SWN 08 Detektory awarii i replikacja procesu]]. Bez tego kontekstu powyższe figury pozostają samym pseudokodem: $\mathcal{S}$ to detektor **silny**, a $\Diamond\mathcal{S}$ — **ostatecznie silny** (*eventually strong*), najsłabszy detektor wystarczający do rozwiązania konsensusu.

---
## Bibliografia
<sub>slajd 45</sub>

1. M. J. Fischer, N. A. Lynch, M. S. Paterson, *Impossibility of distributed consensus with one faulty process*, Journal of the ACM no. 32, 1985, pp. 374–382.
2. L. Lamport, *Paxos made simple*, ACM SIGACT News no. 32, vol. 4, 2001, pp. 51–58.
3. J. Kirsch, Y. Amir, *Paxos for System Builders*, Technical Report CNDS-2008-2, Johns Hopkins University, 2008.
4. D. Ongaro, J. Ousterhout, *In Search of an Understandable Consensus Algorithm*, Proc. USENIX ATC'14, 2014, pp. 305–320.
5. W. Fokkink, *Distributed Algorithms. An Intuitive Approach*, MIT Press, 2013, ch. 12–13.
6. P. Berman, J. A. Garay, K. J. Perry, *Towards Optimal Distributed Consensus*, Proc. 30th FOCS, 1989, pp. 410–415.
7. R. Guerraoui, M. Hurfin, A. Mostefaoui, R. Oliveira, M. Raynal, A. Schiper, *Consensus in Asynchronous Distributed Systems: A Concise Guided Tour*, Advances in Distributed Systems: From Algorithms to Systems, Springer, 2000, pp. 33–47.

---
## Zestawienie algorytmów

| Algorytm | Model | Warunek na $f$ | Obejście FLP | Gwarancja |
|---|---|---|---|---|
| **Paxos** | $\mathbb{S}^{Async}\{\varnothing\}$, fail-stop | $f < \frac{N}{2}$ | rezygnacja z Termination | safety zawsze; liveness tylko z elekcją |
| **Bracha-Toueg** | $\mathbb{S}^{Async}\{\varnothing\}$, fail-stop | $f < \frac{N}{2}$ | randomizacja (Las Vegas) | terminacja z prawd. 1 |
| **Phase-King** | $\mathbb{S}^{Sync}\{\varnothing\}$, bizantyjskie | $f < \frac{N}{4}$ | silniejsza synchronia | deterministyczna, $f+1$ faz |
| **Chandra-Toueg** ($\Diamond\mathcal{S}$) | $\mathbb{S}^{Async}\{\mathcal{D}\}$, crash | $f < \frac{N}{2}$ | detektor awarii | por. [[SWN 08 Detektory awarii i replikacja procesu]] |

---
## Braki i uwagi

> [!todo] Czego nie ma na slajdach tego wykładu
> - **Pierwszej części dowodu Twierdzenia 1** dla Brachy-Touega (slajdy 28–30 zapowiadają „najpierw dowodzimy, że procesy nie mogą zdecydować różnych wartości", ale sam wywód pozostaje w [5]).
> - **Złożoności komunikacyjnej** Paxosa, Brachy-Touega i Phase-Kinga (dla Phase-Kinga jest tylko jakościowe „polynomial number of messages" na slajdzie 34).
> - **Dowodu wyników niemożliwości** dla rozwiązań Monte Carlo i Las Vegas (slajdy 10–11 podają same twierdzenia z odsyłaczem do [5]).
> - **Definicji klas detektorów awarii** — potrzebnej do odczytania `FT_ConsensusFD.pdf`; jest w [[SWN 08 Detektory awarii i replikacja procesu]].
