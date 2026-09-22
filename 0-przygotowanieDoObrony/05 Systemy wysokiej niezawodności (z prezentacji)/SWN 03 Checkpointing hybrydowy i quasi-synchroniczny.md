---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
source: "Slajdy-FT-04_Recovery3.pdf"
slajdy: "1–42"
zagadnienie: 22
---
# SWN 03. Odtwarzanie stanu cz. III — checkpointing hybrydowy i quasi-synchroniczny
---
> Trzeci wykład o odtwarzaniu łączy zalety dwóch poprzednich. **Checkpointing hybrydowy** = asynchroniczne punkty kontrolne (brak narzutu w czasie bezawaryjnym) + okazjonalna synchronizacja ustalająca RL (koniec z efektem domina); jego sztandarowym algorytmem jest **Manetho** z **logowaniem przyczynowym** i **grafem poprzedzania przyczynowego**. **Checkpointing quasi-synchroniczny** idzie dalej: algorytm **Manivannana-Singhala** wymusza punkty kontrolne komunikacją tak, by **każdy** punkt kontrolny był $CP^\bullet$ — proces po awarii wycofuje się tylko do swojego ostatniego punktu kontrolnego i nie czeka na innych.

---
## Checkpointing hybrydowy
<sub>Slajdy-FT-04_Recovery3.pdf, slajdy 1–2</sub>

> **Hybrid checkpointing** = *asynchronous checkpointing* (*failure free performance*) **+** *occasional synchronization gets an RL* (*to finally avoid domino danger*).

![[swn-ft04-s01-checkpointing-hybrydowy.png]]
<sub>Checkpointing hybrydowy — cztery procesy, slajd 1</sub>

**Typowe rozwiązanie** (slajd 2):
- niezależny checkpointing **+ logowanie komunikatów (optymistyczne)**,
- **okazjonalna globalna koordynacja** punktów kontrolnych,
- np. synchronizacja i dostęp do pamięci trwałej **tylko przy *output commit***,
- ☹ **ale protokoły odtwarzania stają się dość złożone**.

## Logowanie przyczynowe
<sub>slajd 3</sub>

**Causal logging:**
- **izoluje proces od skutków awarii innych procesów**,
- ogranicza rollback dowolnego procesu, który uległ awarii, **do najświeższego punktu kontrolnego w pamięci trwałej**.

> **Podstawowy niezmiennik logowania przyczynowego:** informacja o każdym zdarzeniu, które **przyczynowo poprzedza** stan procesu (wg relacji *happened-before* $\rightarrow$), jest albo **w pełni zalogowana**, albo **dostępna lokalnie** dla tego procesu.

### Przykład — maximum recoverable state line
<sub>slajd 4</sub>

![[swn-ft04-s04-maximum-recoverable-state-line.png]]
<sub>Linia maksymalnego stanu odtwarzalnego przy awarii $P_2$ i $P_3$ — slajd 4</sub>

Podczas gdy wiadomości $m_6$ i $m_7$ mogą zostać utracone przy awarii, proces $P_1$ w stanie $s_1^2$ ma informację o **wszystkich zdarzeniach niedeterministycznych poprzedzających jego stan w porządku przyczynowym**. Są to odbiory wiadomości $m_1$, $m_2$, $m_3$, $m_4$ i $m_5$. Dzięki temu kontroler $C_1$ będzie w stanie **pokierować odtwarzaniem** $P_2$ i $P_3$, ponieważ zna kolejność, w jakiej $P_2$ powinien odtworzyć wiadomości $m_2$ i $m_4$, by osiągnąć stan $s_2^2$, oraz kolejność, w jakiej $P_3$ powinien odtworzyć $m_3$, by osiągnąć stan $s_3^1$. Takie wiadomości mogą być odtworzone **z logu nadawcy** $P_1$ albo zostaną **wygenerowane ponownie** podczas odtwarzania $P_2$ i $P_3$.

---
## Algorytm Manetho
<sub>slajdy 5–9</sub>

### Model systemu i idea
<sub>slajd 5</sub>

**Model systemu:**
- awarie procesów typu **fail-recovery**,
- kanały **zawodne, bez zachowania kolejności** (*unreliable reordering channels*),
- obliczenie jest **fragmentarycznie deterministyczne** (*piecewise deterministic*) — sekwencja deterministycznych interwałów stanu, każdy rozpoczęty zdarzeniem niedeterministycznym,
- **wszystkie zdarzenia niedeterministyczne** (*recv*, wewnętrzne) są logowane **optymistycznie**,
- ***output commit***.

**Ogólna idea:** logowanie przyczynowe z użyciem **grafu poprzedzania przyczynowego** (*antecedence graph*).

### Graf poprzedzania przyczynowego
<sub>slajd 6</sub>

- dostarcza każdemu procesowi **kompletną** historię zdarzeń niedeterministycznych, które przyczynowo wpłynęły na jego stan.

> **Def.** *Antecedence graph* $AG(s_i^{k})$ stanu $s_i^{k}$ składa się z węzła reprezentującego **interwał stanu** $s_i^{k}$ (*w istocie: zdarzenia niedeterministycznego rozpoczynającego fragmentarycznie deterministyczny interwał* — przypis ze slajdu), węzłów reprezentujących **zdarzenia niedeterministyczne poprzedzające** $s_i^{k}$ oraz krawędzi odpowiadających relacji ***happened-before***.

![[swn-ft04-s06-antecedence-graph.png]]
<sub>Graf poprzedzania przyczynowego na $P_1$ w stanie $s_1^2$ (odpowiada diagramowi ze slajdu 4) — slajd 6</sub>

### Narzut przenoszenia grafu
<sub>slajd 7</sub>

- AG jest **doklejany** (*piggybacked*) do każdej wysyłanej wiadomości aplikacyjnej,
- przenoszenie **całego** AG w każdej wiadomości może dać **nieakceptowalny narzut**,
- na szczęście jasne jest, że nowa wiadomość przenosi graf będący **nadgrafem** tego dołączonego do poprzedniej wiadomości wysłanej **tym samym kanałem**,
- praktycznie zatem Manetho stosuje **przyrostowe doklejanie** (*incremental piggybacking*).

### Przyrostowe doklejanie
<sub>slajdy 8–9</sub>

![[swn-ft04-s08-incremental-piggybacking.png]]
<sub>Przyrostowe doklejanie grafu — slajd 8</sub>

1. Dla dowolnych kolejnych $m_1$, $m_2$ wysłanych z $P_i$ do $P_j$, wiadomość $m_2$ przenosi **tylko różnicę** między $m_1.AG_i$ a bieżącym $AG_i$:
$$m_2.AG_i = AG_i - m_1.AG_i$$
2. Jeśli $P_i$ **wcześniej odebrał** $m_1$ od $P_j$, to $C_i$ może **wykluczyć** $m_1.AG_j$ z $m_2.AG_i$ (nie ma potrzeby odsyłać $m_1.AG_j$ z powrotem do $C_j$):
$$m_2.AG_i = AG_i - m_1.AG_j$$

> [!warning] Przypis ze slajdu 8
> *uwaga na własności kanałów (zawodne nonFIFO)!* — przy kanałach gubiących i zmieniających kolejność samo „przenoszenie różnicy" wymaga ostrożności, bo odbiorca może nie mieć bazy, względem której różnica była liczona.

> [!note] Uzupełnienie spoza slajdów
> [[Systemy Wysokiej Niezawodności/Algorytm Manetho]] opisuje przebieg odtwarzania, którego slajdy nie podają wprost: gdy proces $P_f$ ulega awarii i odtwarza się na podstawie swojego ostatniego punktu kontrolnego, **poprawne procesy wysyłają do niego swoje grafy poprzedzania przyczynowego**; na ich podstawie $P_f$ odtwarza zagubione wiadomości, które miały miejsce po punkcie kontrolnym, od którego wznowił stan. Notatka podkreśla też, że przesyłanie AG **nie służy** tworzeniu punktów odtwarzania — te powstają spontanicznie i niezależnie, jak w checkpointingu asynchronicznym.

---
## Checkpointing quasi-synchroniczny — algorytm Manivannana-Singhala

### Charakterystyka i model systemu
<sub>slajd 10</sub>

**Manivannan-Singhal [2]:**
- **niezależny checkpointing wymuszany komunikacją** (*independent communication-induced checkpointing*),
- **wybiórcze pesymistyczne** logowanie wiadomości (po stronie **odbiorcy**),
- proces, który uległ awarii, musi wycofać się **tylko do swojego ostatniego punktu kontrolnego** (asynchronicznie); zapewnia się, że CP jest **spójny z najświeższym lokalnym punktem kontrolnym każdego procesu** $\rightarrow$ RL,
- okazjonalna globalna koordynacja punktów kontrolnych,
- **brak efektu domina**.

**Model systemu:** kanały **zawodne, bez zachowania kolejności**; model awarii **fail-recovery**.

### Idea i struktury
<sub>slajdy 11–12</sub>

- niezależnie wyznaczane punkty kontrolne nazywamy **podstawowymi** (*basic checkpoints*),
- wymuszane komunikacją, wyzwalane odbiorem wiadomości — **wymuszonymi** (*forced checkpoints*),
- **wymuszone punkty kontrolne pomagają przesuwać RL** do przodu.

**Struktury:**
- każda wiadomość jest **doklejana** numerem interwału punktu kontrolnego $ckpt\_num_i$ (numer sekwencyjny najświeższego lokalnego punktu kontrolnego) **nadawcy** $P_i$,
- wartość $next_i$ oznacza numer sekwencyjny, który zostanie przypisany kolejnemu **podstawowemu** punktowi kontrolnemu tworzonemu przez $C_i$.

**Idea (cd.)** (slajd 12):
- wartość $next_i$ **tyka** co jednostkę czasu $\Delta t_i$,
- idealnie (choć niekoniecznie) wszystkie $\Delta t_i$ ($i = 1..N$) są sobie bliskie,
- **podstawowe** punkty kontrolne powstają w interwałach $x \cdot \Delta t_i$,
- głównym celem $next_i$ jest utrzymanie numerów interwałów (najświeższego punktu kontrolnego) **wszystkich procesów możliwie zbliżonych**.

### Algorytm checkpointingu QS
<sub>slajd 13</sub>

```
ckpt_num_i := 0
next_i     := 1

on tick:
    next_i := next_i + 1

on taking a basic ckpt:
    if next_i > ckpt_num_i then
        ckpt_num_i := next_i
        save new cp_i
        cp_i.ckpt_num := ckpt_num_i
    else
        skip taking the new cp_i          // ckpt synchro negative

when sending message m:
    m.ckpt_num := ckpt_num_i
    send m

on reception of message m:
    if m.ckpt_num > ckpt_num_i then       // ckpt synchro positive
        ckpt_num_i := m.ckpt_num
        save new cp_i                     // forced ckpt
        cp_i.ckpt_num := ckpt_num_i
    deliver the message
```

> **Dymek ze slajdu** przy gałęzi `else`: $C_i$ **już wykonał wymuszony punkt kontrolny** z $ckpt\_num_i \geqslant next_i$ — dlatego podstawowy punkt kontrolny jest pomijany.

> [!important] Kluczowa kolejność
> Wymuszony punkt kontrolny powstaje **przed dostarczeniem** wiadomości $m$ do procesu. Gdyby powstał po dostarczeniu, $m$ znalazłaby się wewnątrz punktu kontrolnego jako odbiór bez odpowiadającego wysłania — czyli wiadomość osierocona.

### Przykład checkpointingu
<sub>slajdy 14–15</sub>

$P_1$ tworzy podstawowy punkt kontrolny co $3\Delta t_1$, $P_2$ co $2\Delta t_2$, $P_3$ co każde $\Delta t_3$.

![[swn-ft04-s14-qs-przyklad-basic-checkpoints.png]]
<sub>Podstawowe punkty kontrolne trzech procesów — slajd 14</sub>

![[swn-ft04-s15-qs-przyklad-forced-checkpoint.png]]
<sub>Ten sam przebieg z wymuszonym punktem kontrolnym $cp_2^3$ — slajd 15</sub>

- $m_1$ **wymusza** na $P_2$ wykonanie punktu kontrolnego $cp_2^3$ **przed** przetworzeniem $m_1$, ponieważ $m_1.ckpt\_num = 3$, a $ckpt\_num_2 = 2$ (czyli $m_1.ckpt\_num > ckpt\_num_2$),
- $m_2$ **nie wymusza** na $P_3$ punktu kontrolnego ($m_2.ckpt\_num \not> ckpt\_num_3$).

---
## Podstawowy algorytm odtwarzania QS
<sub>slajdy 16–17</sub>

**Założenie:** jeśli proces ulega awarii, **żaden inny proces nie ulega awarii**, dopóki wszystkie procesy nie zostaną wycofane do RL.

```
on fail_i:
    roll back to the latest cp_i
    send Rollback(cp_i.ckpt_num) to all other C_j

on receiving Rollback(rp_num):
    if ckpt_num_i >= rp_num then
 (1)    find the earliest cp_i :: cp_i.ckpt_num >= rp_num
 (2)    roll back to that cp_i
    else                                  // P_i does not roll back at all
 (3)    save new cp_i                     // similar to vcp_i
        cp_i.ckpt_num := rp_num
    ckpt_num_i := cp_i.ckpt_num
```

**Obserwacja i poprawność** (slajd 17):
- wszystkie procesy restartują od **najwcześniejszego** punktu kontrolnego spełniającego $cp_i.ckpt\_num \geqslant rp\_num$ **(1)**, ponieważ albo zostały wycofane do **(2)**, albo kontynuują od **(3)** tego $rp\_num$,
- punkty kontrolne, od których procesy restartują, tworzą $CP^\bullet$, czyli **poprawną RL**.

### Przykłady odtwarzania
<sub>slajdy 18–22</sub>

**Przykład 1** (slajd 18):
- $C_3$ wycofa $P_3$ do $cp_3^5$ i wyśle $Rollback(5)$ do $C_1$ i $C_2$,
- $C_2$ wycofa $P_2$ do $cp_2^5$ (najwcześniejszy $cp_2.ckpt\_num \geqslant 5$),
- $C_1$ wycofa $P_1$ do $cp_1^6$ (najwcześniejszy $cp_1.ckpt\_num \geqslant 5$).

![[swn-ft04-s18-qs-odtwarzanie-przyklad1.png]]
<sub>Przykład 1: awaria $P_3$ i rozgłoszenie $Rollback(5)$ — slajd 18</sub>

Slajd 19 pyta: *czy $C_2$ naprawdę musi wycofać $P_2$ do $cp_2^5$? czy $C_1$ musi wycofać $P_1$ do $cp_1^6$?* Slajd 20 pokazuje ten sam przebieg z **dodatkowymi wiadomościami $m_5$ i $m_6$** — które czynią te wycofania koniecznymi.

![[swn-ft04-s20-qs-odtwarzanie-przyklad1-m5m6.png]]
<sub>Ten sam przebieg z wiadomościami $m_5$, $m_6$ — slajd 20</sub>

**Przykład 2** (slajdy 21–22): $C_1$ **utworzy nowy** $cp_1^5$ (ponieważ **nie ma** żadnego $cp_1.ckpt\_num \geqslant 5$) — to jest gałąź **(3)**, odpowiednik wirtualnego punktu kontrolnego; $P_1$ **nie wycofuje się wcale**.

![[swn-ft04-s22-qs-odtwarzanie-przyklad2.png]]
<sub>Przykład 2: powstanie nowego $cp_1^5$ zamiast wycofania — slajd 22</sub>

### Lematy pomocnicze
<sub>slajd 23</sub>

> **Claim C1.** Jeśli $P_i$ zostaje wycofany do $cp_i^{x_i}$ po otrzymaniu $Rollback(rp\_num)$ od $C_f$, to
> **a)** $x_i \geqslant rp\_num$, **ale**
> **b)** wszystkie punkty kontrolne wykonane przez $C_i$ **przed** $cp_i^{x_i}$ mają numery interwałów **mniejsze** niż $rp\_num$.

> **Claim C2.** Dla dowolnej wiadomości $m$ **wysłanej** przez $P_i$: $send_i(\overrightarrow{P_i,P_j}, m) \in cp_i^{x_i} \iff m.ckpt\_num < x_i$.

> **Claim C3.** Dla dowolnej wiadomości $m$ **odebranej** przez $P_i$: $recv_i(\overleftarrow{P_i,P_j}, m) \in cp_i^{x_i} \Rightarrow m.ckpt\_num < x_i$.

> **Claim C4.** Dla dowolnego $i$: $C_i$ dostarcza odebraną wiadomość $m$ do $P_i$ **dopiero po** wykonaniu punktu kontrolnego $cp_i^{x_i}$ takiego, że $x_i \geqslant m.ckpt\_num$.

### Twierdzenie 1 i dowód
<sub>slajdy 24–25</sub>

> **Theorem 1** (*safety property*). Załóżmy, że po otrzymaniu $Rollback(r)$ od $C_f$ proces $P_i$ zostaje wycofany do $cp_i^{x_i}$. Wówczas zbiór
> $$CP = \{cp_1^{x_1}, \ldots, cp_f^{\,r}, \ldots, cp_i^{x_i}, \ldots, cp_N^{x_N}\}$$
> jest **poprawną RL**.

**Dowód** (nie wprost):
- **(P0)** z C1.a mamy $\bigwedge_{i=1..N} x_i \geqslant r$,
- **(P1)** przypuśćmy, że CP **nie** jest spójne, tzn. $\exists_m : recv_i(P_i,P_j,m) \in cp_i^{x_i} \wedge send_j(P_j,P_i,m) \notin cp_j^{x_j}$,
- **(P2)** zatem $m.ckpt\_num \geqslant x_j$ — z C2,
- **(P3)** oraz $m.ckpt\_num < x_i$ — z C3,
- **(P4)** z (P0), (P2) i (P3) otrzymujemy: $r \leqslant x_j \leqslant m.ckpt\_num < x_i$,
- **(P5)** z C4: $P_i$ musiał przetworzyć $m$ dopiero po wykonaniu pewnego $cp_i^{x}$ takiego, że $x \geqslant m.ckpt\_num$,
- **(P6)** skoro $r \leqslant m.ckpt\_num$ (P4) oraz $x \geqslant m.ckpt\_num$ (P5) $\Rightarrow r \leqslant m.ckpt\_num \leqslant x \Rightarrow x \geqslant r$,
- **(P7)** skoro $recv_i(P_i,P_j,m) \in cp_i^{x_i}$, to $m$ musiała zostać przetworzona przez $P_i$ **przed** $cp_i^{x_i}$,
- **(P8)** zatem ten $cp_i^{x}$, dla którego $x \geqslant r$ (P6), został wykonany **przed** $cp_i^{x_i}$,
- **(P9)** ale z C1.b: wszystkie $cp_i^{x}$ **poprzedzające** $cp_i^{x_i}$ mają numery interwałów $x$ **mniejsze** niż $r$ — sprzeczność.

Zatem założenie (P1), że CP nie jest spójne, było błędne; CP jest $CP^\bullet$ i stanowi RL. $\blacksquare$

### Garbage collection
<sub>slajd 26</sub>

Rozszerzenie **podstawowego** algorytmu odtwarzania QS o jedną linię:

```
on receiving Rollback(rp_num):
    if ckpt_num_i >= rp_num then
        find the earliest cp_i :: cp_i.ckpt_num >= rp_num
        roll back to that cp_i
        discard all the checkpoints beyond cp_i      // <-- garbage collection
    else
        save new cp_i
        cp_i.ckpt_num := rp_num
    ckpt_num_i := cp_i.ckpt_num
```

> [!warning] Rozbieżność z notatką w vaultcie
> [[Systemy Wysokiej Niezawodności/Algorytm Manivannana-Singhala]] opisuje ten krok jako usunięcie punktów kontrolnych **przed** tym, do którego proces został przywrócony. Slajd 26 mówi **`discard all the checkpoints beyond cp_i`** — a więc te **za** (po) punktem wycofania, unieważnione przez rollback. Kasowanie punktów **poprzedzających RL** też występuje w wykładzie, ale jako osobna uwaga na slajdzie 41 („after a process has established a RL, all local checkpoints preceding the line can be deleted"). To dwa różne mechanizmy — notatka w vaultcie skleiła je w jeden.

---
## Pełny algorytm odtwarzania QS
<sub>slajdy 27–40</sub>

### Motywacja i dodatkowe struktury
<sub>slajdy 27–28</sub>

- z **Twierdzenia 1** wszystkie procesy wycofują się do $CP^\bullet$,
- jednak przywrócenie spójnego stanu wymaga także **właściwej obsługi wiadomości** odebranych/wysłanych przed i po wycofaniu,
- pełne odtwarzanie QS używa **wybiórczego pesymistycznego** logowania po stronie **odbiorcy**,
- **tylko dla tych wiadomości**, które prawdopodobnie trzeba będzie odtworzyć, jeśli proces wycofa się do punktu kontrolnego poprzedzającego odbiór tej wiadomości (**zredukowany narzut logowania**),
- wiadomość $m$ odebrana po $cp_i$ jest logowana do $cp_i.msg\_log$.

**Założenie:** wiadomości odebrane, gdy proces jest niesprawny, są **buforowane** (zostaną dostarczone po odtworzeniu).

**Struktury** (slajd 28): jeśli $P_i$ ulega awarii, $C_i$ inicjuje odtwarzanie z nowym
- **numerem inkarnacji** $inc\_num_i$ (początkowo 0),
- **numerem linii odtwarzania** $RL\_num_i$ (początkowo 0),

które **razem jednoznacznie identyfikują** to odtwarzanie. Każda wiadomość $m$ jest doklejana bieżącymi $inc\_num_i$ ($m.inc\_num$) i $RL\_num_i$ ($m.RL\_num$).

**Gdy $P_i$ ulega awarii:** $C_i$ wycofuje $P_i$ do $cp_i^{x_i}$, **inkrementuje** $inc\_num_i$, ustawia $RL\_num_i := cp_i^{x_i}.ckpt\_num$ ($= x_i$) i wysyła $Rollback(inc\_num_i, RL\_num_i)$ do wszystkich pozostałych $C_j$.

### Obsługa komunikatu Rollback
<sub>slajd 29</sub>

Po odebraniu $Rollback(inc\_num, RL\_num)$ odbiorca $C_i$:
- $inc\_num_i := inc\_num$,
- $RL\_num_i := RL\_num$,
- wycofuje się do swojego **najwcześniejszego** punktu kontrolnego $cp_i^{x_i}$ takiego, że $cp_i^{x_i}.ckpt\_num \geqslant RL\_num$,
- jeśli $C_i$ **nie ma** takiego $cp_i$ (wszystkie punkty kontrolne $P_i$ mają numery mniejsze niż $RL\_num$), **nie wycofuje się**, tylko tworzy nowy $cp_i^{x_i}$ z $cp_i^{x_i}.ckpt\_num := RL\_num$,
- (w przeciwnym razie) $C_i$ **scala logi wiadomości** z punktów kontrolnych **następujących po** $cp_i^{x_i}$,
- i przypisuje je jako $cp_i^{x_i}.msg\_log$ (są to więc wszystkie wiadomości zalogowane po $cp_i^{x_i}$).

### Reguła odtwarzania wiadomości
<sub>slajd 30</sub>

- gdy $C_i$ przywraca $cp_i^{x_i}$, odtwarza z $cp_i^{x_i}.msg\_log$ **tylko te** wiadomości, których **wysłanie nie zostanie cofnięte**,
- tzn. wiadomości **powstałe na lewo od bieżącej RL i dostarczone na prawo od niej**,
- **formalnie:** po wycofaniu do $cp_i^{x_i}$ wiadomość $m$ jest odtwarzana z logu **wtedy i tylko wtedy**, gdy $m$ została wcześniej odebrana **po** $cp_i^{x_i}$ **oraz** $m.ckpt\_num < RL\_num_i$,
- to załatwia **wiadomości utracone** (unikając duplikacji).

### Przykład: wiadomości utracone i zduplikowane
<sub>slajdy 31–33</sub>

Przebieg bazowy z $inc\_num = 0$ (slajd 31). Po awarii $P_3$ i wycofaniu ($inc\_num_3 := 1$, $RL\_num_3 := 5$) — slajd 32:
- $m_4$ i $m_6$ **zostaną odtworzone** (ponieważ ich wysłanie nie zostało cofnięte) $\leftarrow$ **wiadomości utracone**,
- $m_5$ i $m_7$ **nie zostaną odtworzone** (ich zdarzenia wysłania zostały cofnięte) $\leftarrow$ **brak duplikacji**.

Przy ponownym wykonaniu (slajd 33) $m_5$ i $m_7$ są **wysyłane ponownie w nowej inkarnacji** (z $inc\_num = 1$, $RL\_num = 5$).

### Wiadomości opóźnione i zduplikowane
<sub>slajdy 34–35</sub>

![[swn-ft04-s34-wiadomosci-opoznione.png]]
<sub>Wiadomość **opóźniona** $m_8$ — slajd 34</sub>

> $m$ odebrana przez $P_i$ jest **opóźniona** (*delayed*) $\iff m.inc\_num < inc\_num_i$ **i** $m.ckpt\_num < RL\_num_i$.

![[swn-ft04-s35-wiadomosci-zduplikowane.png]]
<sub>Wiadomość **zduplikowana** $m_8$ — slajd 35</sub>

> $m$ odebrana przez $P_i$ jest **zduplikowana** (*duplicated*) $\iff m.inc\_num < inc\_num_i$ **i** $m.ckpt\_num \geqslant RL\_num_i$.

Różnica: opóźniona **zostanie odebrana po odtworzeniu** (i trzeba ją przetworzyć); zduplikowana **zostanie odebrana po odtworzeniu, ale także wysłana ponownie w następnej inkarnacji** (i trzeba ją odrzucić).

### Obsługa wiadomości — trzy przypadki
<sub>slajdy 36–37, 39</sub>

**Case 1** — $m$ wysłana w **poprzedniej** inkarnacji ($m.inc\_num < inc\_num_i$), slajd 36:
- nadawca $P_j$ **nie wiedział** o odtwarzaniu w chwili wysyłania $m$,
- nadawca $P_j$ **wycofa się przed** tym wysłaniem $\iff m.ckpt\_num \geqslant RL\_num_i$,
- zatem odbiorca $P_i$ powinien **przetworzyć** $m$ $\iff m.ckpt\_num < RL\_num_i$ $\leftarrow$ **delayed**,
- w przeciwnym razie $C_i$ powinien **odrzucić** $m$, bo jej wysłanie zostanie cofnięte $\leftarrow$ **duplicated**,
- ponadto: jeśli $m.ckpt\_num < RL\_num_i$, to $m$ jest **logowana do bieżącego $cp_i.msg\_log$ przed przetworzeniem**, aby umożliwić odtworzenie w razie przyszłego wycofania do $RL\_num \geqslant m.ckpt\_num$.

**Case 2** — $m$ wysłana w **bieżącej** inkarnacji ($m.inc\_num = inc\_num_i$), slajd 37:
- jeśli $m.ckpt\_num < ckpt\_num_i$, to $m$ jest **logowana przed przetworzeniem**; jeśli $C_i$ będzie dalej wycofywał $P_i$ do bieżącego punktu kontrolnego, $m$ będzie musiała być odtworzona $\iff$ nowe $RL\_num > m.ckpt\_num$,
- jeśli $m.ckpt\_num > ckpt\_num_i$, to $m$ jest przetwarzana **po** wykonaniu przez $C_i$ nowego $cp_i$ z $cp_i.ckpt\_num := m.ckpt\_num$, a $m$ **nie jest logowana**,
- jeśli $m.ckpt\_num = ckpt\_num_i$, to $m$ jest przetwarzana **bez logowania i bez** nowego punktu kontrolnego.

> **Reguła logowania wiadomości** (wynikająca z Case 1 i Case 2, slajd 37):
> $m$ odebrana przez $C_i$ jest logowana przed dostarczeniem do $P_i$ $\iff$
> $$(m.inc\_num < inc\_num_i \wedge m.ckpt\_num < RL\_num_i) \;\text{ lub }\; (m.inc\_num = inc\_num_i \wedge m.ckpt\_num < ckpt\_num_i)$$

![[swn-ft04-s38-log-m7-przed-przetworzeniem.png]]
<sub>Przykład do Case 2 (slajd 38): $m_7.inc\_num = inc\_num_1 = 1$ i $m_7.ckpt\_num = 5 < ckpt\_num_1 = 6$, więc $m_7$ musi być zalogowana przed przetworzeniem — bo jeśli $P_1$ wycofa się do $cp_1^6$, $C_1$ będzie musiał odtworzyć odbiór $m_7$, o ile nowe $RL\_num > 5$.</sub>

**Case 3** — $m$ wysłana w **przyszłej** inkarnacji ($m.inc\_num > inc\_num_i$), slajd 39:
- $C_i$ **dowiaduje się** w ten sposób o odtwarzaniu,
- ustawia $RL\_num_i := m.RL\_num$ oraz $inc\_num_i := m.inc\_num$,
- i wycofuje się do najwcześniejszego punktu kontrolnego $cp_i$ takiego, że $cp_i.ckpt\_num \geqslant m.RL\_num$,
- po wycofaniu $m$ jest obsługiwana **jak w Case 2** (bo teraz $m.inc\_num = inc\_num_i$).

![[swn-ft04-s40-m7-przed-rollback.png]]
<sub>Przykład do Case 3 (slajd 40): $m_7$ została odebrana **przed** komunikatem $Rollback(1,5)$ od $C_3$, więc w chwili odbioru $m_7.inc\_num = 1$, a $inc\_num_1 = 0$.</sub>

### Podsumowanie algorytmu
<sub>slajd 41</sub>

- proces, który uległ awarii, wycofuje się **do swojego ostatniego punktu kontrolnego** i informuje o tym pozostałych,
- **zawsze istnieje $CP^\bullet$ zawierający ten punkt kontrolny** = **brak efektu domina**,
- proces może wznowić obliczenia **bez czekania**, aż pozostałe procesy się wycofają = checkpointing i odtwarzanie są **w pełni asynchroniczne**,
- checkpointing wymuszany komunikacją **inteligentnie kieruje** tworzeniem punktów kontrolnych, eliminując „bezużyteczne" punkty kontrolne (**każdy CP jest $CP^\bullet$**),
- z każdą wiadomością doklejane są **tylko 3 liczby całkowite** ($ckpt\_num$, $inc\_num$, $RL\_num$),
- wiadomości są logowane **pesymistycznie, ale wybiórczo, po stronie odbiorcy**,
- **garbage collection:** po ustaleniu przez proces RL, wszystkie lokalne punkty kontrolne **poprzedzające** tę linię mogą zostać usunięte.

---
## Bibliografia
<sub>slajd 42</sub>

1. E.N. Elnozahy, W. Zwaenepoel, *Manetho, Transparent Rollback-Recovery with Low Overhead, Limited Rollback and Fast Output Commit*, IEEE Transactions on Computers, vol. 41, no. 5, pp. 526–531, 1992.
2. D. Manivannan, M. Singhal, *Quasi-synchronous checkpointing: Models, characterization, and classification*, IEEE Transactions on Parallel and Distributed Systems, pp. 703–713, 1996.
3. A. D. Kshemkalyani, M. Singhal, *Distributed Computing — Principles, Algorithms, and Systems*, Cambridge University Press, 2008, ch. 13.

---
## Braki i uwagi

> [!todo] Czego nie ma na slajdach tego wykładu
> - **Algorytmu odtwarzania Manetho** — slajdy 5–9 opisują wyłącznie model, graf poprzedzania i doklejanie; sam przebieg odtwarzania trzeba wziąć z [[Systemy Wysokiej Niezawodności/Algorytm Manetho]].
> - **Złożoności** obu algorytmów (liczba wiadomości, rozmiar doklejanych danych poza uwagą „tylko 3 liczby całkowite" dla QS).
> - **Tabeli porównawczej** trzech technik checkpointingu (skoordynowany / niezależny / quasi-synchroniczny) — trzeba ją złożyć z [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane]], [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów]] i tej notatki.
> - **Dokończenia dowodu Twierdzenia 1** w wersji ze slajdu 24 — pełny wywód jest dopiero na slajdzie 25 (tu przytoczony w całości).
