---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
source: "rso_sum_03.pdf"
slajdy: "1–65"
---
# RSO 09. Konstrukcja spójnego obrazu stanu globalnego
---
> Wykład 3 odpowiada na pytanie: **jak wyznaczyć stan globalny systemu rozproszonego, w którym nie ma zegara globalnego ani pamięci wspólnej?** Definiuje konfigurację spójną i linię odcięcia, porównuje dwa modele reprezentacji stanu globalnego, a następnie podaje trzy algorytmy migawek: **Chandy-Lamporta** (kanały FIFO), **Lai-Yang** (kanały nonFIFO, historia komunikacji) i **algorytm kolorujący procesy i wiadomości**.

---
## Proces rozproszony — formalizm
<sub>rso_sum_03.pdf, slajdy 2–4</sub>

> **Proces rozproszony** $\Pi$, będący współbieżnym wykonaniem zbioru $\mathcal{P} = \{P_1, P_2, \ldots, P_n\}$ procesów sekwencyjnych $P_i$, opisuje uporządkowana czwórka
> $$\Pi = \langle \Sigma, \Sigma^{0}, \Lambda, \Phi \rangle \tag{7.1}$$

gdzie:
- $\Sigma$ — zbiór **stanów globalnych** procesu rozproszonego, $\Sigma \subseteq \mathcal{S}_1 \times \mathcal{S}_2 \times \ldots \times \mathcal{S}_n$,
- $\Sigma^{0}$ — zbiór **stanów początkowych**, $\Sigma^{0} \subseteq \mathcal{S}_1^{0} \times \mathcal{S}_2^{0} \times \ldots \times \mathcal{S}_n^{0}$,
- $\Lambda$ — zbiór **zdarzeń**, $\Lambda = \mathcal{E}_1 \cup \mathcal{E}_2 \cup \ldots \cup \mathcal{E}_n$,
- $\Phi$ — **funkcja tranzycji**, taka że $\Phi \subseteq \Sigma \times \Lambda \times \Sigma$.

**Częściowe wykonanie** procesu rozproszonego $\Pi$ utożsamia się z ciągiem $\Sigma^{0}, E^{1}, \Sigma^{1}, E^{2}, \ldots, \Sigma^{s}, E^{s+1}, \Sigma^{s+1}$, składającym się naprzemiennie ze stanów i zdarzeń, takim że dla każdego $u$, $0 \leqslant u \leqslant s$:
$$\langle \Sigma^{u}, E^{u+1}, \Sigma^{u+1} \rangle \in \Phi \tag{7.2}$$

Przez **wykonanie (realizację)** $\Upsilon$ procesu $\Pi$ rozumieć będziemy częściowe wykonanie **rozpoczynające się stanem początkowym** $\Sigma^{0} \in \Sigma^{0}$.

## Stan osiągalny i konfiguracja spójna
<sub>rso_sum_03.pdf, slajdy 5–10, 14</sub>

> Stan $\Sigma'$ procesu jest **osiągalny** ze stanu $\Sigma$, co oznaczamy $\Sigma \rightsquigarrow \Sigma'$ (7.3), jeżeli istnieje częściowe wykonanie $\Sigma^{0}, E^{1}, \Sigma^{1}, E^{2}, \ldots, \Sigma^{s}, E^{s+1}, \Sigma^{s+1}$ procesu $\Pi$, takie że $\Sigma = \Sigma^{0}$ a $\Sigma' = \Sigma^{s+1}$.

> Jeżeli istnieje wykonanie procesu $\Pi$ takie, że $\Sigma$ jest stanem końcowym, to stan ten nazwiemy **globalnym stanem osiągalnym (spójnym)** procesu rozproszonego $\Pi$ (ang. _reachable, consistent_).

**Historia wykonania**: każdemu wykonaniu $\Upsilon$ odpowiada pewien ciąg stanów $\Sigma^{0}, \Sigma^{1}, \ldots, \Sigma^{s+1}$, nazywany **śladem wykonania (realizacji) procesu**, oraz ciąg zdarzeń $E^{0}, E^{1}, \ldots, E^{s+1}$, nazywany **historią wykonania (realizacji) procesu**. Historię oznaczamy przez $\Xi^{s}$, a zbiór historii przez $\Xi$.

**Konfiguracja**:
- **Zbiór konfiguracji** (obrazów stanu globalnego) $\Gamma$ jest iloczynem kartezjańskim stanów procesów składowych: $\Gamma = \mathcal{S}_1 \times \mathcal{S}_2 \times \ldots \times \mathcal{S}_n$,
- **Konfiguracja** — wektor stanów lokalnych wszystkich składowych procesów sekwencyjnych: $\Gamma = \langle S_1, S_2, \ldots, S_n \rangle$.

> Konfigurację $\Gamma$ nazwiemy **konfiguracją spójną** lub **obrazem spójnym**, jeżeli $\forall E, \forall E'$ zachodzi:
> $$(E' \in \Gamma \wedge E \mapsto E') \Rightarrow (E \in \Gamma) \tag{7.4}$$

> **Twierdzenie 7.1.** Konfiguracja $\Gamma = \langle S_1^{k_1}, S_2^{k_2}, \ldots, S_n^{k_n} \rangle$, reprezentująca stan osiągalny przetwarzania rozproszonego $\Pi$, jest konfiguracją spójną.

## Linia odcięcia, odcięcie spójne
<sub>rso_sum_03.pdf, slajdy 11–14</sub>

**Linia odcięcia** dzieli zbiór zdarzeń na **przeszłość** oraz **przyszłość**.

> **Odcięciem** $\Psi$ zbioru zdarzeń $\Lambda$ nazwiemy skończony zbiór $\Psi \subseteq \Lambda$, taki że:
> $$(E' \in \Psi \wedge E \mapsto_i E') \Rightarrow (E \in \Psi) \tag{7.5}$$

> Odcięcie $\Psi$ zbioru zdarzeń $\Lambda$ nazwiemy **odcięciem spójnym**, gdy:
> $$(E' \in \Psi \wedge E \mapsto E') \Rightarrow (E \in \Psi) \tag{7.6}$$

Odcięcie $\Psi_2$ jest **późniejsze** od $\Psi_1$, jeżeli $\Psi_1 \subseteq \Psi_2$.

> [!important] Różnica
> Odcięcie zwykłe zamyka się względem poprzedzania **lokalnego** ($\mapsto_i$), odcięcie **spójne** — względem pełnej relacji poprzedzania ($\mapsto$), a więc także wzdłuż komunikatów.

![[rso-w3-s12-odciecie-spojne.png]]
<sub>Odcięcie spójne — rso_sum_03.pdf, slajd 12</sub>

![[rso-w3-s13-odciecie-niespojne.png]]
<sub>Odcięcie niespójne — rso_sum_03.pdf, slajd 13. Przykłady naruszeń: $E_2^{1} \mapsto E_1^{1}$, $E_1^{1} \in \Psi_3$, a $E_2^{1} \notin \Psi_3$; $E_2^{3} \mapsto E_3^{2}$, $E_3^{2} \in \Psi_4$, a $E_2^{3} \notin \Psi_4$.</sub>

Każdemu odcięciu $\Psi$ opisanemu przez linię odcięcia $\sigma_1^{k_1}, \sigma_2^{k_2}, \ldots, \sigma_n^{k_n}$ odpowiada konfiguracja
$$\Gamma = \langle S_1^{k_1}, S_2^{k_2}, \ldots, S_n^{k_n} \rangle \tag{7.7}$$

> **Twierdzenie 7.5.** Niech $\Gamma$ będzie konfiguracją, a $\Psi$ odpowiadającym jej odcięciem. Konfiguracja $\Gamma$ jest konfiguracją spójną **wtedy i tylko wtedy**, gdy $\Psi$ jest odcięciem spójnym.

---
## Po co wyznaczać stan globalny
<sub>rso_sum_03.pdf, slajd 15</sub>

Wiele problemów istniejących w systemach rozproszonych można sprowadzić do problemu oceny stanu globalnego:
- śledzenie i sterowanie wykonaniem programu rozproszonego (monitoring, debugging),
- **detekcja stanów awaryjnych** (utraty wiadomości, zakleszczenia) lub **oczekiwanych** (zakończenia obliczeń rozproszonych),
- dostosowywanie konfiguracji i funkcji systemu do zmieniającego się obciążenia.

## Modele stanów globalnych
<sub>rso_sum_03.pdf, slajdy 16–27</sub>

Ilustracja na problemie: **wzajemne wykluczanie trzech procesów** współdzielących pewien zasób; warunkiem uzyskania dostępu jest posiadanie **znacznika** (ang. _token_), który krąży między procesami połączonymi w logiczny pierścień.

### Reprezentacja I — stan globalny jako złożenie stanów procesów
<sub>slajdy 17–21</sub>

Stan $S_i(\tau)$ procesu $P_i$ w każdej chwili $\tau$ czasu globalnego (rzeczywistego) zdefiniowany jest przez trzy zmienne:
$$S_i(\tau) = \langle present_i(\tau), outLog_i(\tau), inLog_i(\tau) \rangle \tag{7.9}$$
- $present_i(\tau)$ — przyjmuje wartość $True$ tylko wówczas, gdy znacznik typu TOKEN znajduje się w chwili $\tau$ w procesie $P_i$,
- $outLog_i(\tau)$ — kolejka znaczników **wysłanych** do chwili $\tau$ przez proces $P_i$,
- $inLog_i(\tau)$ — kolejka znaczników **odebranych** przez proces $P_i$ do chwili $\tau$.

| Zalety | Wady |
|---|---|
| ogranicza liczbę składowych stanu globalnego do **liczby procesów** | podejście jest adekwatne tylko dla systemów z **kanałami niezawodnymi** |
| | **rosnący koszt** związany z zapamiętywaniem wysłanych i odebranych wiadomości |

### Reprezentacja II — stan globalny jako złożenie stanów procesów i kanałów
<sub>slajdy 22–25</sub>

- **$n+m$ składowych** odpowiadających stanom lokalnym wszystkich $n$ procesów i wszystkich $m$ kanałów,
- prostsza reprezentacja stanu procesu kosztem większej liczby składowych.

Stan procesu $P_i$ określony jest przez zmienną logiczną $present_i$ oraz przez liczniki $sentNo_i$ (ang. _sent number_) i $recvNo_i$ (ang. _received number_). Stan kanału $L_{i,j}$ określany przez zbiór znaczników znajdujących się aktualnie w kanale $C_{i,j}$.

**Prostsza reprezentacja**: stan procesu określony przez zmienną $present_i$ (posiadanie znacznika), stan kanału określony przez zmienną $present_{i,j}$ (znacznik w kanale).

![[rso-w3-s25-model-stanow-przyklad2.png]]
<sub>Modele stanów globalnych — przykład 2, rso_sum_03.pdf, slajd 25. $\Sigma(\tau) = \langle S_1(\tau), S_2(\tau), S_3(\tau), L_{1,2}(\tau), L_{2,3}(\tau), L_{3,1}(\tau) \rangle$</sub>

### Ocena stanów globalnych i porównanie reprezentacji
<sub>slajdy 26–27</sub>

Przykłady oceny stanów globalnych:
- **zaginięcie znacznika** — może prowadzić do blokady całego systemu,
- **zwielokrotnienie znaczników** — może prowadzić do obecności kilku procesów w sekcji krytycznej.

| Reprezentacja | Charakterystyka |
|---|---|
| **pierwsza** | operuje na informacji **najpełniejszej**: unikalnych znacznikach i całej historii komunikacji |
| **druga** | **utożsamia** wszystkie znaczniki i wyróżnia tylko stany ich obecności oraz nieobecności w poszczególnych procesach i kanałach |

---
## Problem konstrukcji stanu globalnego
<sub>rso_sum_03.pdf, slajdy 28–31</sub>

**Metoda wyznaczania**: proces chcący wyznaczyć stan globalny wysyła żądania wyznaczania stanów lokalnych, a następnie konstruuje na ich podstawie stan globalny.

**Problemy** — otrzymane składowe stany lokalne procesów mogą być:
- **przestarzałe**,
- **niekompletne**,
- odpowiadać **konfiguracjom niespójnym** (reprezentującym stany nieosiągalne).

Dodatkowe **założenia upraszczające** (slajd 31, w podejściu naiwnym):
- dostępny jest dla wszystkich procesów globalny zegar czasu rzeczywistego,
- znane jest maksymalne opóźnienie komunikacji,
- względne prędkości przetwarzania w poszczególnych węzłach są ograniczone.

### Koncepcja konstrukcji obrazu spójnego (przy powyższych założeniach)
<sub>rso_sum_03.pdf, slajdy 32–34</sub>

1. Proces inicjatora $Q_\alpha$ wysyła do monitorów wszystkich procesów przetwarzania rozproszonego wiadomość: „zapamiętaj stan lokalny w chwili $\tau_s$”.
2. Monitory $Q_i$ zapamiętują stan lokalny $S_i$ w chwili $\tau_s$ i natychmiast wysyłają wiadomość kontrolną do wszystkich monitorów. Zapamiętują też stan $L_{k,i}$ kanałów wejściowych jako zbiorów wszystkich wiadomości aplikacyjnych, które zostały wysłane przed $\tau_s$ a dotarły do $Q_i$ po chwili $\tau_s$.
3. Gdy monitor $Q_i$ otrzyma przez kanał $C_{j,i}$ pierwszą wiadomość o etykiecie czasowej większej od $\tau_s$, aktualną wartość $L_{j,i}$ uznaje jako stan tego kanału w chwili $\tau_s$.
4. Monitor $Q_i$, po otrzymaniu wiadomości o etykietach czasowych większej od $\tau_s$ ze **wszystkich** kanałów wejściowych, przesyła stan lokalny i wyznaczone stany kanałów wejściowych do inicjatora $Q_\alpha$.
5. Po odebraniu od wszystkich monitorów wiadomości zawierających stany lokalne oraz stany ich kanałów wejściowych w chwili $\tau_s$, inicjator $Q_\alpha$ konstruuje obraz stanu globalnego $\Gamma$.

---
## Algorytm Chandy-Lamporta
<sub>rso_sum_03.pdf, slajdy 35–47</sub>

### Założenia dotyczące środowiska przetwarzania
<sub>slajd 35</sub>
- **niezawodne kanały zachowujące uporządkowanie wiadomości** (niezawodne kanały **FIFO**),
- stan reprezentowany jest w postaci złożenia lokalnych stanów procesów i stanów kanałów,
- pełen **asynchronizm** komunikacji i przetwarzania,
- **brak zegara globalnego**.

### Koncepcja
<sub>slajdy 36–38</sub>
- Pewien monitor $Q_\alpha$ inicjuje **proces konstrukcji (detekcji)** spójnego obrazu stanu globalnego. $Q_\alpha$ zapamiętuje stan lokalny skojarzonego z nim procesu aplikacyjnego $P_\alpha$ i wysyła wiadomość kontrolną (**znacznik**, _marker_) do wszystkich incydentnych monitorów.
- Każdy monitor $Q_i$ po odebraniu znacznika z kanału $C_{j,i}$ sprawdza, czy jest to **pierwszy znacznik** odebrany w danym procesie detekcji. Jeżeli tak, monitor $Q_i$ zapamiętuje stan $S_i$ procesu $P_i$, uznaje stan kanału $C_{j,i}$ **za pusty** i propaguje znacznik przez wszystkie swoje kanały wyjściowe.
- Jeżeli stan procesu $P_i$ został już wcześniej zapamiętany, to $Q_i$ uznaje za stan kanału $C_{j,i}$ zbiór tych wszystkich wiadomości aplikacyjnych, które dotarły tym kanałem **po zapamiętaniu stanu a przed otrzymaniem znacznika**.
- Po odebraniu znaczników ze **wszystkich** kanałów wejściowych, monitor $Q_i$ przesyła zapamiętany stan lokalny $procState_i$ oraz stan $chanState_i$ do monitora $Q_\beta$.

![[rso-w3-s39-ilustracja-chandy-lamporta.png]]
<sub>Ilustracja działania algorytmu Chandy-Lamporta — rso_sum_03.pdf, slajd 39. Dwa różne przebiegi dają dwie różne (ale spójne) konfiguracje $\Gamma^{1}$ i $\Gamma^{2}$.</sub>

### Zapis algorytmu
<sub>slajdy 40–46</sub>

```
type PACKET extends FRAME is record of
  data : MESSAGE
end record

type MARKER extends FRAME

type STATE extends FRAME is record of
  procState : PROCESS_STATE
  chanState : array [1..n] of set of MESSAGE
end record

msgIn      : MESSAGE
pcktOut    : PACKET
markerOut  : MARKER
stateOut   : STATE
recvMark_i : array [1..n] of BOOLEAN := False
chanState_i: array [1..n] of set of MESSAGE := ∅
procState_i: PROCESS_STATE
C_i^IN     : set of CHANNEL_ID
C_i^OUT    : set of CHANNEL_ID
involved_i : BOOLEAN := False
```

```
 1. procedure RECORDSTATE() do
 2.    procState_i := S_i
 3.    for all C_{j,i} ∈ C_i^IN do
 4.       chanState_i[j] := ∅
 5.    end for
 6.    involved_i := True
 7.    send(Q_i, Q_i^OUT, markerOut)
 8. end procedure

 9. procedure SENDSTATE(i) do
10.    stateOut.procState := procState_i
11.    stateOut.chanState := chanState_i
12.    send(Q_i, Q_β, stateOut)
13. end procedure

14. when e_start(Q_α, TakeSnapshot) do
15.    RECORDSTATE()
16.    recvMark_α[α] := True
17.    if C_α^IN = ∅ then
18.       SENDSTATE(α)
19.    end if
20. end when

21. when e_send(P_i, P_j, msgOut: MESSAGE) do
22.    pcktOut.data := msgOut
23.    send(Q_i, Q_j, pcktOut)
24. end when

25. when e_receive(Q_j, Q_i, markerIn: MARKER) do
26.    if ¬ involved_i then
27.       RECORDSTATE()
28.    end if
29.    recvMark_i[j] := True
30.    if ∀ C_{j,i} ∈ C_i^IN :: recvMark_i[j] then
31.       SENDSTATE(i)
32.    end if
33. end when

34. when e_receive(Q_j, Q_i, pcktIn: PACKET) do
35.    msgIn := pcktIn.data
36.    if involved_i ∧ ¬ recvMark_i[j] then
37.       chanState_i[j] := chanState_i[j] ∪ {msgIn}
38.    end if
39.    deliver(P_j, P_i, msgIn)
40. end when
```

> **Twierdzenie.** Algorytm Chandy-Lamporta wyznacza w **skończonym czasie konfigurację spójną**.

---
## Algorytm Lai-Yang (kanały nonFIFO)
<sub>rso_sum_03.pdf, slajdy 48–57</sub>

Lai i Yang przedstawili algorytm, który **nie wymaga**, by kanały były typu FIFO. W algorytmie zakłada się, że reprezentacje stanów lokalnych obejmują **historię komunikacji**, a więc odpowiednie zbiory wiadomości dotychczas wysłanych i odebranych. Stan kanału wyznacza się jako różnicę:
$$outLog_{i,j}(\tau) = inLog_{i,j}(\tau')$$
gdzie $\tau$ — wysłanie znacznika, $\tau'$ — odebranie znacznika.

**Koncepcja**:
- Jeśli po zapamiętaniu stanu proces $P_i$ już nigdy nie wyśle wiadomości do $P_j$, to nie ma potrzeby przesłania do $Q_j$ znacznika.
- Jeśli wiadomości będą w dalszym ciągu wysyłane, to dołączany jest do nich **znacznik w formie etykiety**.
- Wyróżnione pakiety ze znacznikiem przechwytywane są przez monitor $Q_j$ odbiorcy i powodują zapamiętanie stanu procesu $P_j$, a przed przekazaniem wiadomości aplikacyjnej procesowi $P_j$.
- Algorytm przydziela procesom (kontrolerom) oraz wiadomościom jeden z **dwóch kolorów**: $White$ albo $Red$.

```
type PACKET extends FRAME is record of
   colour : enum {White, Red}
   data   : MESSAGE
end record

type STATE extends FRAME is record of
   procState : PROCESS_STATE
   sentLog   : set of MESSAGE
   recvLog   : set of MESSAGE
end record

procColour_i : enum {White, Red} := White
sentLog_i, outLog_i, recvLog_i, inLog_i : set of MESSAGE := ∅
```

```
 1. procedure RECORDSTATE() do
 2.    procState_i := S_i
 3.    sentLog_i := outLog_i
 4.    recvLog_i := inLog_i
 5. end procedure

 6. procedure SENDSTATE(i) do
 7.    stateOut.procState := procState_i
 8.    stateOut.sentLog := sentLog_i
 9.    stateOut.recvLog := recvLog_i
10.    send(Q_i, Q_β, stateOut)
11. end procedure

12. when e_start(Q_α, TakeSnapshot) do
13.    RECORDSTATE()
14.    procColour_α := Red
15.    dummyOut.pcktColour := Red
16.    dummyOut.data := ∅
17.    send(Q_α, Q \ {Q_α}, dummyOut)
18.    SENDSTATE(α)
19. end when

20. when e_send(P_i, P_j, msgOut: MESSAGE) do
21.    outLog_i := outLog_i ∪ {msgOut}
22.    pcktOut.colour := procColour_i
23.    pcktOut.data := msgOut
24.    send(Q_i, Q_j, pcktOut)
25. end when

26. when e_receive(Q_j, Q_i, pcktIn: PACKET) do
27.    if pcktIn.colour = Red ∧ procColour_i = White then
28.       procColour_i := Red
29.       RECORDSTATE(i)
30.       SENDSTATE(i)
31.    end if
32.    msgIn := pcktIn.data
33.    if msgIn ≠ ∅ then
34.       inLog_i := inLog_i ∪ {msgIn}
35.       deliver(P_j, P_i, msgIn)
36.    end if
37. end when
```

> **Złożoność** (slajd 57): dla grafu pełnego reprezentującego topologię przetwarzania rozproszonego złożoność **czasowa** algorytmu Lai-Yanga wynosi **1**, a złożoność **komunikacyjna**, w sensie liczby przesyłanych znaczników, wynosi **$n-1$**, pomijając fazę przesyłania wiadomości o stanach procesów i kanałów.

---
## Algorytm kolorujący procesy i wiadomości
<sub>rso_sum_03.pdf, slajdy 58–65</sub>

**Koncepcja**:
- Każdy proces pierwotnie ma kolor $White$, a staje się $Red$ po zapamiętaniu jego stanu lokalnego a przed przekazaniem mu pierwszej wiadomości z pakietem koloru $Red$.
- Inicjator detekcji stanu globalnego zapamiętuje stan lokalny procesu, staje się $Red$ i wysyła **pusty pakiet koloru $Red$** do wszystkich monitorów.
- Wiadomości będące w wyznaczonym obrazie stanu globalnego w kanałach, to wiadomości w **pakietach koloru $White$ odebrane przez monitor koloru $Red$**.
- Za każdym razem, gdy monitor otrzymuje tego typu pakiet, przesyła zawartą w nim wiadomość do inicjatora.

```
type PACKET extends FRAME is record of
   colour : enum {White, Red}
   data   : MESSAGE
end record

type PROC_STATE extends FRAME is record of
   procState : PROCESS_STATE
end record

type CHAN_STATE extends FRAME is record of
   chanState : MESSAGE
end record

procColour_i : enum {White, Red} := White
```

```
 1. when e_start(Q_α, TakeSnapshot) do
 2.    procState_α := S_α
 3.    procColour_i := Red;
 4.    dummyOut.colour := Red;
 5.    dummyOut.data := ∅
 6.    send(Q_α, Q \ {Q_α}, dummyOut)
 7. end when

 8. when e_send(P_i, P_j, msgOut: MESSAGE) do
 9.    pcktOut.colour := procColour_i
10.    pcktOut.data := msgOut
11.    send(Q_i, Q_j, pcktOut)
12. end when

13. when e_receive(Q_j, Q_i, pcktIn: PACKET) do
14.    msgIn := pcktIn.data
15.    if pcktIn.colour = White ∧ procColour_i = Red
16.    then
17.       chanStateOut.chanState := msgIn
18.       send(Q_i, Q_α, chanStateOut)
19.    end if
20.    if pcktIn.colour = Red ∧ procColour_i = White then
21.       procColour_i := Red
22.       procState_i := S_i
23.       procStateOut.procState := procState_i
24.       send(Q_i, Q_α, procStateOut)
25.    end if
27.    if msgIn ≠ ∅ then
28.       deliver(P_j, P_i, msgIn)
29.    end if
30. end when
```

> [!note] Numeracja linii
> Numeracja w slajdzie 65 przeskakuje z 25 na 27 — tak jest w oryginale prezentacji.

---
## Porównanie trzech algorytmów

| | Chandy-Lamport | Lai-Yang | Kolorujący procesy i wiadomości |
|---|---|---|---|
| **Kanały** | niezawodne **FIFO** | **nonFIFO** | nonFIFO |
| **Nośnik znacznika** | osobna wiadomość MARKER | **kolor** pakietu (White/Red) | kolor pakietu (White/Red) |
| **Stan kanału** | wiadomości odebrane między zapamiętaniem stanu a znacznikiem | różnica $outLog$ i $inLog$ (historia komunikacji) | wiadomości $White$ odebrane przez monitor $Red$, przesyłane do inicjatora |
| **Koszt stanu lokalnego** | stan procesu + tablica stanów kanałów | stan procesu + **pełne logi** wysłanych/odebranych | stan procesu (bez logów) |
| **Złożoność komunikacyjna** | ~~TODO~~ patrz niżej | $n-1$ znaczników (graf pełny) | ~~TODO~~ patrz niżej |

> [!todo] Braki w prezentacjach
> Prezentacja podaje złożoność **wyłącznie dla algorytmu Lai-Yanga** (slajd 57). Dla algorytmu Chandy-Lamporta i algorytmu kolorującego **nie podano** złożoności czasowej ani komunikacyjnej — uzupełnić z podręcznika lub notatek z ćwiczeń.

> [!todo] Brak w prezentacjach
> W wykładzie **nie omówiono** twierdzenia o poprawności algorytmu Lai-Yanga ani algorytmu kolorującego (dla Chandy-Lamporta jest twierdzenie na slajdzie 47, ale **bez dowodu**).

---
## Powiązania
- Stan procesu, stan kanału, relacja $\mapsto$, stany współbieżne, graf stanów osiągalnych → [[RSO 07 Model środowiska przetwarzania]]
- Kanały FIFO/nonFIFO, zegary logiczne, złożoność → [[RSO 08 Czas wirtualny i złożoność algorytmów]]
- Detekcja stanu oczekiwanego (zakończenia obliczeń) → [[RSO 10 Detekcja zakończenia]]
- Detekcja stanu awaryjnego (zakleszczenia) → [[RSO 06 Zakleszczenie w systemach rozproszonych]]
