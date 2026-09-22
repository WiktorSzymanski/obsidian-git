---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
source: "rso_sum_02.pdf"
slajdy: "1–54"
---
# RSO 08. Czas wirtualny i złożoność algorytmów
---
> Wykład 2 wprowadza **monitor** i konwencję zapisu algorytmów rozproszonych, definiuje **czas wirtualny (logiczny)** oraz jego realizacje — **zegar skalarny** (alg. Lamporta) i **zegar wektorowy** (alg. Matterna), omawia **kanały FIFO** (alg. Müllendera) i **kanały FC**, a na końcu podaje aparat **oceny złożoności** algorytmów rozproszonych (czasowa, komunikacyjna: pakietowa i bitowa).

---
## Monitor
<sub>rso_sum_02.pdf, slajdy 2–3</sub>

Z każdym procesem aplikacyjnym $P_i$ skojarzony jest **proces monitora** $Q_i$. Procesy aplikacyjne i monitory komunikują się przez wspólne środowisko komunikacyjne.

**Cechy monitora**:
- monitor **może odczytywać (obserwować)** zmienne lokalne procesu,
- monitor **może obserwować i kontrolować** zdarzenia komunikacyjne,
- monitor **nie ma** natomiast możliwości zmiany stanu procesu przez przypisanie jego zmiennym lokalnym nowych wartości.

## Konwencja zapisu algorytmów
<sub>rso_sum_02.pdf, slajdy 4–9</sub>

**Typy komunikatów**: aplikacyjne, kontrolne, sygnały, pakiety.
**Wspólne atrybuty**: identyfikator typu komunikatu, identyfikator komunikatu, identyfikator nadawcy, identyfikator odbiorcy.

```
type FRAME is record of
   tag: ...   /* pole identyfikatora typu */
   mId: ...   /* pole identyfikatora wiadomości */
   sId: ...   /* pole identyfikatora nadawcy */
   rId: ...   /* pole identyfikatora odbiorcy */
end record

type MESSAGE extends FRAME is record of  ...  end record
type CONTROL extends FRAME is record of  ...  end record
type SIGNAL  extends FRAME is record of       end record

type PACKET  extends FRAME is record of
   ...
   data: MESSAGE
end record
```

---
## Czas wirtualny
<sub>rso_sum_02.pdf, slajdy 10–13</sub>

Zegary realizowane w systemach asynchronicznych mają stanowić **aproksymację czasu rzeczywistego**. Aproksymacja taka uwzględnia jedynie zachodzące w systemie zdarzenia i dlatego czas ten nazywany jest **czasem wirtualnym (logicznym)**.

W odróżnieniu od czasu rzeczywistego upływ czasu wirtualnego **nie jest autonomiczny** — zależy od występujących w systemie zdarzeń i stąd określone wartości czasu wirtualnego mogą **nigdy nie wystąpić**. Czas wirtualny wyznacza się za pomocą **zegarów logicznych** (ang. _logical clocks_).

> **Zegar logiczny systemu rozproszonego** jest funkcją $\mathcal{T}: \Lambda \rightarrow \mathcal{Y}$, odwzorowującą zbiór zdarzeń $\Lambda$ w zbiór uporządkowany $\mathcal{Y}$, taką że
> $$(E \mapsto E') \Rightarrow (\mathcal{T}(E) < \mathcal{T}(E')) \tag{4.1}$$
> gdzie $<$ jest relacją porządku na zbiorze $\mathcal{Y}$.

> [!important] Implikacja działa tylko w jedną stronę
> W ogólności relacja odwrotna **nie musi** być spełniona, tzn. $\mathcal{T}(E) < \mathcal{T}(E') \nRightarrow E \mapsto E'$.

**Właściwości zegarów logicznych** (slajd 13):
- jeżeli zdarzenie $E$ zachodzi przed $E'$ w tym samym procesie, to wartość zegara logicznego odpowiadającego zdarzeniu $E$ jest mniejsza od wartości zegara odpowiadającego zdarzeniu $E'$,
- w przypadku przesyłania wiadomości $M$, czas logiczny przyporządkowany zdarzeniu nadania wiadomości $M$ jest zawsze mniejszy niż czas logiczny przyporządkowany zdarzeniu odbioru tej wiadomości.

---
## Zegar skalarny
<sub>rso_sum_02.pdf, slajdy 14–20</sub>

> Jeżeli przeciwdziedzina $\mathcal{Y}$ funkcji zegara logicznego jest zbiorem liczb naturalnych $\mathbb{N}$ lub rzeczywistych $\mathbb{R}$, to zegar nazywany jest **zegarem skalarnym**.

**Realizacja**: funkcja $\mathcal{T}(E)$ implementowana jest przez zmienne naturalne $clock_i$, $1 \leqslant i \leqslant n$, skojarzone z procesami $P_i$ (monitorami $Q_i$). Wartość zmiennej $clock_i$ reprezentuje w każdej chwili wartość funkcji $\mathcal{T}(E_i^{k})$ odnoszącą się do ostatniego zdarzenia $E_i^{k}$ jakie zaszło w procesie $P_i$.

### Algorytm Lamporta
<sub>rso_sum_02.pdf, slajdy 16–18</sub>

```
type PACKET extends FRAME is record of
        clock : INTEGER
        data  : MESSAGE
end record

msgIn   : MESSAGE
pcktOut : PACKET
clock_i : INTEGER
d       : INTEGER
```

```
 1.  when e_send(P_i, P_j, msgOut: MESSAGE) do
 2.     clock_i := clock_i + d
 3.     pcktOut.clock := clock_i
 4.     pcktOut.data  := msgOut
 5.     send(Q_i, Q_j, pcktOut)
 6.  end when

 7.  when e_receive(Q_j, Q_i, pcktIn: PACKET) do
 8.     clock_i := max(clock_i, pcktIn.clock) + d
 9.     msgIn := pcktIn.data
10.     deliver(P_j, P_i, msgIn)
11.  end when

12.  when e_internal(P_i, *) do
13.     clock_i := clock_i + d
14.  end when
```

![[rso-w2-s19-przyklad-zegary-skalarne.png]]
<sub>Przykład synchronizacji zegarów logicznych — rso_sum_02.pdf, slajd 19. Zaznaczono: $(E \mapsto E') \Rightarrow (\mathcal{T}(E) < \mathcal{T}(E'))$ zachodzi, ale $(\mathcal{T}(E) < \mathcal{T}(E')) \Rightarrow (E \mapsto E')$ **nie** zachodzi.</sub>

![[rso-w2-s20-relacja-zdarzenia-zegar.png]]
<sub>Relacja między zbiorem zdarzeń a zbiorem wartości zegara skalarnego — rso_sum_02.pdf, slajd 20. Zdarzenia współbieżne ($\parallel$) mogą odwzorować się na $<$, $=$ lub $>$.</sub>

---
## Zegar wektorowy
<sub>rso_sum_02.pdf, slajdy 21–28</sub>

> **Zegarem wektorowym** jest zegar logiczny, dla którego przeciwdziedzina funkcji $\mathcal{T}$, oznaczana dalej dla odróżnienia przez $\mathcal{T}^{V}$, jest **zbiorem $n$-elementowych wektorów liczb naturalnych lub rzeczywistych**.

**Realizacja**: funkcja $\mathcal{T}^{V}$ implementowana jest przez zmienne tablicowe $vClock_i$, $1 \leqslant i \leqslant n$, skojarzone z poszczególnymi procesami. Zmienna $vClock_i$ jest tablicą $[1..n]$ liczb naturalnych, odpowiadającą pewnej aproksymacji czasu globalnego z perspektywy procesu $P_i$.

### Algorytm Matterna
<sub>rso_sum_02.pdf, slajdy 23–25</sub>

```
type PACKET extends FRAME is record of
        vClock : array [1..n] of INTEGER
        data   : MESSAGE
end record

msgIn   : MESSAGE
pcktOut : PACKET
vClock_i: array [1..n] of INTEGER
d       : INTEGER
k       : INTEGER
```

```
 1.  when e_send(P_i, P_j, msgOut: MESSAGE) do
 2.     vClock_i[i] := vClock_i[i] + d
 3.     pcktOut.vClock := vClock_i
 4.     pcktOut.data   := msgOut
 5.     send(Q_i, Q_j, pcktOut)
 6.  end when

 7.  when e_receive(Q_j, Q_i, pcktIn: PACKET) do
 8.     vClock_i[i] := vClock_i[i] + d
 9.     for all k ∈ {1, 2, ..., n} do
10.        vClock_i[k] := max(vClock_i[k], pcktIn.vClock[k])
11.     end for
12.     msgIn := pcktIn.data
13.     deliver(P_j, P_i, msgIn)
14.  end when

15.  when e_internal(P_i, *) do
16.     vClock_i[i] := vClock_i[i] + d
17.  end when
```

### Twierdzenia
<sub>rso_sum_02.pdf, slajdy 26, 28</sub>

> **Twierdzenie 4.1.** W każdej chwili czasu rzeczywistego
> $$\forall i,j :: vClock_i[i] \geqslant vClock_j[i] \tag{4.2}$$
> gdzie zmienna $vClock_i[i]$ reprezentuje skalarny czas lokalny procesu $P_i$, a zmienna $vClock_j[i]$, $j \neq i$, aktualne wyobrażenie procesu $P_j$ o bieżącym skalarnym czasie lokalnym procesu $P_i$.

> **Twierdzenie 4.2.** Niech $\mathcal{T}^{V}(E)$ oraz $\mathcal{T}^{V}(E')$ będą wartościami zegarów wektorowych zdarzeń $E$ i $E'$. Wówczas:
> $$(E \mapsto E') \iff (\mathcal{T}^{V}(E) < \mathcal{T}^{V}(E')) \tag{4.3}$$

> [!tip] Kluczowa różnica wobec zegara skalarnego
> Dla zegara wektorowego zachodzi **równoważność** (⟺), a nie tylko implikacja — dlatego zegar wektorowy **wykrywa współbieżność** zdarzeń.

### Relacje na etykietach wektorowych
<sub>rso_sum_02.pdf, slajd 27</sub>

$$vClock_i = vClock_j \iff \forall_k\ vClock_i[k] = vClock_j[k]$$
$$vClock_i \neq vClock_j \iff \exists_k\ vClock_i[k] \neq vClock_j[k]$$
$$vClock_i \leqslant vClock_j \iff \forall_k\ vClock_i[k] \leqslant vClock_j[k]$$
$$vClock_i \nleqslant vClock_j \iff \exists_k\ vClock_i[k] > vClock_j[k]$$
$$vClock_i < vClock_j \iff vClock_i \leqslant vClock_j \wedge vClock_i \neq vClock_j$$
$$vClock_i \nless vClock_j \iff \neg(vClock_i \leqslant vClock_j \wedge vClock_i \neq vClock_j)$$
$$vClock_i \parallel vClock_j \iff vClock_i \nless vClock_j \wedge vClock_j \nless vClock_i$$

---
## Kanały FIFO
<sub>rso_sum_02.pdf, slajdy 29–34</sub>

> Kanały gwarantujące porządek odbioru wiadomości zgodny z kolejnością wysyłania będziemy nazywać **kanałami FIFO** (ang. _First-In-First-Out_).

### Algorytm Müllendera
Realizuje kanał FIFO nad kanałem nonFIFO z użyciem numerów sekwencyjnych i bufora opóźniającego.

```
type PACKET extends FRAME is record of
        seqNo : INTEGER
        data  : MESSAGE
end record

msgIn      : MESSAGE
pcktOut    : PACKET
delayBuf_i : array [1..n] of set of PACKET := ∅
seqNo_i    : array [1..n] of INTEGER := 0
delivNo_i  : array [1..n] of INTEGER := 0
delivered_i: BOOLEAN
```

```
 1.  when e_send(P_i, P_j, msgOut: MESSAGE) do
 2.     pcktOut.data := msgOut
 3.     seqNo_i[j] := seqNo_i[j] + 1
 4.     pcktOut.seqNo := seqNo_i[j]
 5.     send(Q_i, Q_j, pcktOut)
 6.  end when

 7.  when e_receive(Q_j, Q_i, pcktIn: PACKET) do
 8.     if pcktIn.seqNo = delivNo_i[j] + 1
 9.     then
10.        msgIn := pcktIn.data
11.        deliver(P_j, P_i, msgIn)
12.        delivNo_i[j] := delivNo_i[j] + 1
13.        delivered_i := True
14.     else
15.        delayBuf_i[j] := delayBuf_i[j] ∪ {pcktIn}
16.        delivered_i := False
17.     end if
18.     while delivered_i do
19.        delivered_i := False
20.        for all pckt ∈ delayBuf_i[j] do
21.           if pckt.seqNo = delivNo_i[j] + 1 then
22.              msgIn := pckt.data
23.              deliver(P_j, P_i, msgIn)
24.              delivNo_i[j] := delivNo_i[j] + 1
25.              delivered_i := True
26.              delayBuf_i[j] := delayBuf_i[j] \ {pckt}
27.           end if
28.        end for
29.     end while
30.  end when
```

**Cechy kanałów FIFO** (slajd 34):
- są pewnym mechanizmem synchronizacji wymaganym przez wiele aplikacji,
- ułatwiają znalezienie rozwiązania i konstrukcję algorytmów rozproszonych dla wielu problemów,
- **ograniczają**, w porównaniu z kanałami nonFIFO, współbieżność komunikacji, a tym samym efektywność przetwarzania.

---
## Kanały typu FC (Flush Channels)
<sub>rso_sum_02.pdf, slajdy 35–38</sub>

> **Kanały typu FC** (ang. _Flush Channels_) łączą zalety kanałów FIFO i nonFIFO (pewien stopień synchronizacji i współbieżnej komunikacji).

| Mechanizm (operacja) komunikacji | Zdarzenia | Wiadomości |
|---|---|---|
| $send^{t}$ (ang. _two-way-flush send_) | $e\_send^{t}$ | $M^{t}$ |
| $send^{f}$ (ang. _forward-flush send_) | $e\_send^{f}$ | $M^{f}$ |
| $send^{b}$ (ang. _backward-flush-send_) | $e\_send^{b}$ | $M^{b}$ |
| $send^{o}$ (ang. _ordinary send_) | $e\_send^{o}$ | $M^{o}$ |

> **Wyprzedzanie wiadomości** (slajd 36): powiemy, że wiadomość $M'$ **wyprzedza** wiadomość $M$ w kanale $C_{i,j}$, jeżeli wiadomość $M$ została wysłana przez $P_i$ **wcześniej** niż $M'$, lecz proces $P_j$ **najpierw odebrał** wiadomość $M'$.

![[rso-w2-s37-typy-wiadomosci-fc.png]]
<sub>Typy wiadomości w kanałach FC — rso_sum_02.pdf, slajd 37. $M^{t}$ (TF) zachowuje się jak FIFO w obie strony, $M^{o}$ (OF) jak nonFIFO.</sub>

**Implementacja kanałów FC** (slajd 38) może wykorzystywać: selektywne rozgłaszanie, liczniki, potwierdzenia, …

---
## Środowisko zachowujące uporządkowanie przyczynowe
<sub>rso_sum_02.pdf, slajdy 39–40</sub>

$$(e\_send(P_i, P_j, M) \mapsto e\_send(P_k, P_j, M')) \Rightarrow (e\_receive(P_i, P_j, M) \mapsto_j e\_receive(P_k, P_j, M')) \tag{4.10}$$

![[rso-w2-s40-uporzadkowanie-przyczynowe.png]]
<sub>rso_sum_02.pdf, slajdy 39–40. Kanały FIFO **nie wystarczają** — wymagane jest uporządkowanie przyczynowe obejmujące ścieżki przez procesy pośrednie.</sub>

---
## Złożoność algorytmów rozproszonych

### Funkcje kosztu — oznaczenia i definicja
<sub>rso_sum_02.pdf, slajdy 41–43</sub>

- $\Delta_A^{\delta}$ — zbiór wszystkich poprawnych danych wejściowych $\delta$ algorytmu $A$,
- $\mathcal{Z}_A^{*}(\delta)$ — **koszt wykonywania** algorytmu $A$ dla danych $\delta$, gdzie $\delta \in \Delta_A^{\delta}$ i $\mathcal{Z}_A^{*}: \Delta_A^{\delta} \rightarrow \mathbb{R}$,
- $\mu$ — rozmiar danych wejściowych $\delta$ (rozmiar zadania), taki że $\mu = \mathcal{W}(\delta)$, gdzie $\mathcal{W}: \Delta_A^{\delta} \rightarrow \mathbb{N}$ jest zadaną funkcją.

W praktyce, zamiast kosztu $\mathcal{Z}_A^{*}(\delta)$ stosuje się zwykle jego oszacowanie w funkcji rozmiaru zadania $\mu = \mathcal{W}(\delta)$.

> **Funkcją kosztu wykonania algorytmu** nazywać będziemy odwzorowanie
> $$\mathcal{Z}_A : \Delta_A^{\mu} \rightarrow \mathbb{R} \tag{4.11}$$
> gdzie $\Delta_A^{\mu}$ jest zbiorem wszystkich poprawnych danych wejściowych o rozmiarze $\mu$ algorytmu $A$.

Najczęściej stosowane jest odwzorowanie **pesymistyczne** (najgorszego przypadku):
$$\mathcal{Z}_A(\mu) = \sup\{\mathcal{Z}_A^{*}(\delta) : \delta \in \Delta_A^{\delta} \wedge \mathcal{W}(\delta) = \mu\} \tag{4.12}$$

### Rząd funkcji
<sub>rso_sum_02.pdf, slajdy 44–46</sub>

Niech $f$ i $g$ będą dowolnymi funkcjami odwzorowującymi $\mathbb{N}$ w $\mathbb{R}$.

| Zapis | Nazwa | Warunek |
|---|---|---|
| $f = O(g)$ (4.13) | $f$ jest **co najwyżej** rzędu funkcji $g$ | istnieje stała rzeczywista $c > 0$ oraz $n_0 \in \mathbb{N}$ takie, że dla każdej wartości $n > n_0$, $n \in \mathbb{N}$ zachodzi $\lvert f(n)\rvert < c \cdot \lvert g(n)\rvert$ (4.14) |
| $f = \Theta(g)$ (4.15) | $f$ jest **dokładnie** rzędu funkcji $g$ | $f = O(g) \wedge g = O(f)$ |
| $f = \Omega(g)$ (4.16) | $f$ jest **co najmniej** rzędu funkcji $g$ | $g = O(f)$ |

### Złożoność czasowa
<sub>rso_sum_02.pdf, slajdy 47–48</sub>

W wypadku algorytmów rozproszonych **złożoność czasowa** jest funkcją kosztu wykonania, wyrażoną przez **liczbę kroków algorytmu do jego zakończenia**, przy założeniu, że:
- czas wykonywania każdego kroku (operacji) jest stały,
- kroki wykonywane są synchronicznie,
- czas transmisji wiadomości jest stały.

W analizie złożoności czasowej algorytmów rozproszonych przyjmuje się też na ogół, że:
- czas przetwarzania lokalnego (wykonania każdego kroku) jest **pomijalny (zerowy)**,
- czas transmisji jest **jednostkowy**.

### Złożoność komunikacyjna
<sub>rso_sum_02.pdf, slajd 49</sub>

**Złożoność komunikacyjna** jest funkcją kosztu wykonania algorytmu wyrażaną przez:
- **liczbę pakietów (wiadomości)** przesyłanych w trakcie wykonywania algorytmu do jego zakończenia,
- **sumaryczną długość (w bitach)** wszystkich wiadomości przesłanych w trakcie wykonywania algorytmu.

W konsekwencji wyróżniamy złożoność **pakietową** i **bitową**.

### Przykład — bariera
<sub>rso_sum_02.pdf, slajdy 50–53</sub>

**Przykład (1) — topologia: graf w pełni połączony**
- koordynator rozgłasza komunikat początku bariery,
- wszyscy uczestnicy odbierają komunikat i odsyłają potwierdzenia,
- po otrzymaniu potwierdzeń od wszystkich procesów koordynator rozsyła komunikat końca bariery.

Złożoność: BARRIER $n-1$, ACK $n-1$, END $n-1$ $\Rightarrow$ **$3 \cdot (n-1)$** komunikatów.

**Przykład (2) — topologia: pierścień logiczny**
- koordynator wysyła komunikat początku bariery,
- wszyscy uczestnicy odbierają komunikat i przesyłają go dalej,
- po otrzymaniu komunikatu rozpoczynającego operację bariery koordynator przesyła komunikat końca bariery.

Złożoność: BARRIER $n$, END $n$ $\Rightarrow$ **$2 \cdot n$** komunikatów.

---
## Warunki poprawności
<sub>rso_sum_02.pdf, slajd 54</sub>

Analizę poprawności algorytmu rozproszonego (procesu rozproszonego) dekomponuje się zwykle na analizę jego bezpieczeństwa i żywotności:
- właściwość **bezpieczeństwa** (ang. _safety_, _consistency_),
- właściwość **żywotności (postępu)** (ang. _liveness_, _progress_).

---
## Powiązania
- Model zdarzeń, relacja $\mapsto$ i diagramy przestrzenno-czasowe → [[RSO 07 Model środowiska przetwarzania]]
- Zegary skalarne i wektorowe są podstawą porządków dostarczania komunikatów → [[RSO 01 Komunikacja grupowa]]
- Uporządkowanie przyczynowe wraca jako Causal Order rozgłaszania → [[RSO 01 Komunikacja grupowa]]
- Zegary wektorowe wykorzystuje algorytm Lai-Yang i algorytm kolorujący → [[RSO 09 Stan globalny i migawki]]
