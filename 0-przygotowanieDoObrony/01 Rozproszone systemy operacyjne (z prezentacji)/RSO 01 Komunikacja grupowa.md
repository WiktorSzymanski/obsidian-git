---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 1
source: "rso_sum_05.pdf"
slajdy: "1–56"
---
# RSO 01. Komunikacja grupowa (mechanizmy rozgłaszania niezawodnego)
---
> Wykład 5 omawia **komunikację grupową** jako mechanizm rozsyłania wiadomości do grupy procesów, a następnie buduje **hierarchię mechanizmów rozgłaszania niezawodnego**: BEB → RB → URB, oraz porządki dostarczania: **FIFO (RFB)**, **przyczynowy (RCB)** i **globalny (TO)**. Dla BEB, RB (algorytm pasywny i aktywny) i URB podane są pełne algorytmy wraz ze złożonością.

> [!warning] Zakres slajdów vs zakres zagadnienia
> Prezentacja **nie omawia** części zagadnienia 1 z listy egzaminacyjnej: grup otwartych/zamkniętych, płaskich/hierarchicznych, statycznych/dynamicznych, szczegółów usługi członkostwa, synchronizacji widoków (_virtual synchrony_) ani algorytmów realizacji porządków (sekwencer, ISIS). Zob. sekcję [[#Braki w prezentacjach]] na końcu.

---
## Podstawowe definicje
<sub>rso_sum_05.pdf, slajdy 2–6</sub>

> Nieformalnie, przez **rozgłaszanie** rozumiemy mechanizm (abstrakcję) komunikacyjny, za pomocą którego proces może wysłać wiadomość do **grupy procesów**. W mechanizmach rozgłaszania niezawodnego gwarantowane są ponadto pewne własności **pomimo występowania awarii**.

**Przykładowe zastosowania**: systemy z wieloma uczestnikami · zwielokrotnianie (replikacja).

**Komunikacja grupowa** (ang. _group communication_) to mechanizm umożliwiający rozsyłanie (ang. _multicast_) poprzez organizowanie procesów (szerzej — obiektów) w grupy. Obejmuje **dwa aspekty**:
- **zarządzanie grupami procesów** (usługa członkostwa, ang. _membership service_),
- **algorytmy (niezawodnego) rozsyłania wiadomości w grupie**.

Pozwala modelować niezawodną komunikację procesów przy założeniu, że zbiór procesów **dynamicznie się zmienia**.

| Pojęcie | Definicja |
|---|---|
| **Grupa** | rzeczywisty zbiór procesów uczestniczących we wspólnym przetwarzaniu i komunikujących się poprzez przekazywanie wiadomości |
| **Obraz grupy** (ang. _view_) | grupa **widziana w danej chwili** czasu rzeczywistego w pojedynczym procesie; obraz grupy jest generowany przez usługę członkostwa; **różne procesy mogą mieć różne obrazy tej samej grupy w tym samym momencie** |

![[rso-w5-s06-obraz-grupy.png]]
<sub>Obraz grupy — proces $q$ uległ awarii, ale $p$ ma jeszcze obraz $\{p, q, r\}$, a $r$ już $\{p, r\}$. rso_sum_05.pdf, slajd 6</sub>

### Architektura systemu
<sub>slajd 8</sub>

Aplikacja komunikuje się z **warstwą komunikacji grupowej** przez trzy operacje: **wyślij** (w dół), **odbierz** (w górę) i **zmiana_obrazu** (w górę). Do warstwy komunikacji grupowej dochodzą z zewnątrz sygnały **awaria/powrót**.

### Przykładowe systemy
<sub>slajd 7</sub>
- **ISIS** — pionierski system GCS — `http://www.cs.cornell.edu/Info/Projects/Isis/`
- **Horus (Ensemble)** — nowoczesna wersja ISIS — `http://www.cs.cornell.edu/Info/Projects/HORUS/`
- **Jgroups** — system GCS dla języka Java — `http://www.jgroups.org`
- **Transis** — `http://www.cs.huji.ac.il/labs/transis/`

---
## Klasy mechanizmów rozgłaszania niezawodnego
<sub>rso_sum_05.pdf, slajd 9</sub>

| Skrót | Nazwa polska | Nazwa angielska |
|---|---|---|
| **BEB** | Podstawowe rozgłaszanie niezawodne | _best-effort broadcast_ |
| **RB** | Zgodne rozgłaszanie niezawodne | _regular reliable broadcast_ |
| **URB** | Jednolite rozgłaszanie niezawodne | _uniform reliable broadcast_ |
| **RFB** | Zgodne rozgłaszanie niezawodne z uporządkowaniem FIFO wiadomości | _FIFO reliable broadcast_ |
| **RCB** | Zgodne rozgłaszanie niezawodne z przyczynowym uporządkowaniem wiadomości | _causal reliable broadcast_ |
| **TO** | Zgodne rozgłaszanie niezawodne z globalnym uporządkowaniem wiadomości | _total order reliable broadcast_ |

---
## BEB — podstawowe rozgłaszanie niezawodne
<sub>rso_sum_05.pdf, slajdy 10–17</sub>

### Specyfikacja
<sub>slajd 10</sub>

| Własność | Ang. | Treść |
|---|---|---|
| **Ważność** | _best-effort validity_ | Jeżeli procesy $P_i$ oraz $P_j$ są **poprawne**, to każda wiadomość rozgłaszana przez $P_i$ jest ostatecznie dostarczona do $P_j$ |
| **Brak powielania** | _no duplication_ | Jeżeli wiadomość jest dostarczona, to jest dostarczona **co najwyżej raz** |
| **Brak samogeneracji** | _no creation_ | Jeżeli jakaś wiadomość jest dostarczona do procesu $P_j$, to została wcześniej rozgłoszona przez jakiś proces $P_i$ |

### Operacje komunikacyjne
<sub>slajd 11</sub>
$$send^{BRB}(P_i, \mathcal{P}, M) \tag{12.1}$$
Zdarzenie: $e\_send^{BRB}(P_i, \mathcal{P}, M)$.
$$deliver^{BRB}(P_j, P_i, M) \tag{12.2}$$
Operacja uaktywniająca zdarzenie $e\_receive^{BRB}(P_j, P_i, M)$ i przekazująca w efekcie wiadomość $M$ do procesu aplikacyjnego $P_i$.

### Założenia
<sub>slajd 12</sub>
- dostępny jest mechanizm **kanałów niezawodnych** (ang. _perfect point-to-point links_),
- algorytm **nie wymaga detektorów awarii**, a więc przyjmuje model przetwarzania z **ukrytymi awariami** (ang. _fail-silent_).

### Algorytm
<sub>slajdy 13–14</sub>

```
type PACKET extends FRAME is record of
     data : MESSAGE
end record

pcktOut : PACKET
msgIn   : MESSAGE
```

```
 1. when e_send^BRB(P_i, P, msgOut: MESSAGE) do
 2.    pcktOut.data := msgOut
 3.    for all P_j ∈ P do
 4.       send^PL(Q_i, Q_j, pckOut);
 5.    end for
 6. end when

 7. when e_receive^PL(Q_j, Q_i, pcktIn: PACKET) do
 8.    msgIn := pckIn.data
 9.    deliver^BRB(P_j, P_i, msgIn)
10. end when
```

### Złożoność
<sub>slajd 14</sub>
Pomijając czas przetwarzania lokalnego, nadawca wysyła wiadomość do wszystkich procesów w **1 kroku**. Zatem:
- złożoność **czasowa** wynosi **1**,
- złożoność **komunikacyjna** wynosi **$n$**.

![[rso-w5-s15-ilustracja-beb.png]]
<sub>Ilustracja podstawowego rozgłaszania niezawodnego. Legenda: `broadcast M` = $send^{BRB}(P_i, \mathcal{P}, M)$, `deliver M` = $deliver^{BRB}(P_j, P_i, M)$. rso_sum_05.pdf, slajd 15</sub>

---
## RB — zgodne rozgłaszanie niezawodne
<sub>rso_sum_05.pdf, slajdy 16–32</sub>

### Specyfikacja
<sub>slajd 16</sub>

| Własność               | Ang.             | Treść                                                                                                                                              |
| ---------------------- | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Ważność**            | _validity_       | Jeżeli proces $P_i$ jest poprawny, to każda wiadomość rozgłaszana przez ten proces jest ostatecznie dostarczona do $P_i$                           |
| **Brak powielania**    | _no duplication_ | Jeżeli wiadomość jest dostarczona, to jest dostarczona co najwyżej raz                                                                             |
| **Brak samogeneracji** | _no creation_    | Jeżeli jakaś wiadomość została dostarczona do procesu $P_i$, to została wcześniej rozgłoszona przez jakiś proces $P_i$                             |
| **Zgodność**           | _agreement_      | Jeżeli jakaś wiadomość została odebrana przez pewien **poprawny** proces $P_i$, to ostatecznie **wszystkie poprawne procesy** odbiorą tę wiadomość |

Operacje: $send^{RRB}(P_i, \mathcal{P}, M)$ (12.3) i $deliver^{RRB}(P_j, P_i, M)$ (12.4).

### Algorytm pasywny (_lazy reliable broadcast_)
<sub>slajdy 18–24</sub>

**Założenia**: dostępny jest **doskonały detektor awarii** oraz **podstawowe rozgłaszanie niezawodne**.

```
type PACKET extends FRAME is record of
     origin : PROCESS_ID
     data   : MESSAGE
end record

pcktIn     : PACKET
msgOut     : MESSAGE
correct_i  : set of PROCESS_ID := P
delivered_i: set of MESSAGE := ∅
from_i     : array [1..n] of set of pair of ⟨PROCESS_ID, MESSAGE⟩ := ∅
pId        : PROCESS_ID
msg        : MESSAGE
```

```
 1. when e_send^RRB(P_i, P, msgOut: MESSAGE) do
 2.    pcktOut.origin := P_i
 3.    pcktOut.data := msgOut
 4.    send^BRB(Q_i, Q, pcktOut)
 5. end when

 6. when e_receive^BRB(Q_j, Q_i, pcktIn: PACKET) do
 7.    msgIn := pckIn.data
 8.    if msgIn ∉ delivered_i then
 9.       delivered_i := delivered_i ∪ {msgIn}
10.       deliver^RRB(pcktIn.origin, P_i, msgIn)
11.       from_i[j] := from_i[j] ∪ ⟨pcktIn.origin, msgIn⟩
12.       if P_j ∉ correct_i then
13.          send^BRB(Q_i, Q, pcktIn)
14.       end if
15.    end if
16. end when

17. when e_crash(P_j) do
18.    correct_i := correct_i \ {P_j}
19.    for all ⟨pId, msg⟩ ∈ from_i[j] do
20.       pcktOut.origin := pId
21.       pcktOut.data := msg
22.       send^BRB(Q_i, Q, pcktOut)
23.    end for
24. end when
```

**Idea**: proces **nie retransmituje** wiadomości dopóki nadawca żyje. Retransmisja („odkurzenie” zapamiętanych wiadomości z `from_i[j]`) następuje dopiero po wykryciu awarii nadawcy $P_j$ — stąd nazwa **pasywny / leniwy**.

**Złożoność** <sub>slajd 24</sub>:

| Przypadek | Czasowa | Komunikacyjna |
|---|---|---|
| optymistyczny | 1 | $n$ |
| pesymistyczny | $n$ | $n^2$ |

### Algorytm aktywny (_eager reliable broadcast_)
<sub>slajdy 26–32</sub>

**Założenia**: dostępny jest mechanizm podstawowego rozgłaszania niezawodnego (**bez** detektora awarii).

```
type PACKET extends FRAME is record of
     origin : PROCESS_ID
     data   : MESSAGE
end record

pcktIn      : PACKET
msgOut      : MESSAGE
delivered_i : set of MESSAGE := ∅
```

```
 1. when e_send^RRB(P_i, P, msgOut: MESSAGE) do
 2.    pcktOut.data := msgOut
 3.    pcktOut.origin := P_i
 4.    deliver^RRB(P_i, P_i, msgOut)
 5.    delivered_i := delivered_i ∪ {msgOut}
 6.    send^BRB(Q_i, Q, pcktOut)
 7. end when

 8. when e_receive^BRB(Q_j, Q_i, pcktIn: PACKET) do
 9.    msgIn := pckIn.data
10.    if msgIn ∉ delivered_i then
11.       delivered_i := delivered_i ∪ {msgIn}
12.       deliver^RRB(pcktOut.origin, P_i, msgIn)
13.       send^BRB(Q_i, Q, pcktOut)
14.    end if
15. end when
```

**Idea**: każdy proces, który po raz pierwszy odbiera wiadomość, **natychmiast** rozgłasza ją dalej (_flooding_) — niezależnie od tego, czy nadawca żyje.

**Złożoność** <sub>slajd 32</sub>:

| Przypadek | Czasowa | Komunikacyjna |
|---|---|---|
| optymistyczny | 1 | $n^2$ |
| pesymistyczny | $n$ | $n^2$ |

---
## URB — jednolite rozgłaszanie niezawodne
<sub>rso_sum_05.pdf, slajdy 33–42</sub>

### Specyfikacja
<sub>slajd 33</sub>

Ważność, brak powielania i brak samogeneracji — **jak w RB**. Różnica dotyczy zgodności:

| Własność | Ang. | Treść |
|---|---|---|
| **Jednolita zgodność** | _uniform agreement_ | Jeżeli jakaś wiadomość została odebrana przez pewien proces $P_i$ (**poprawny bądź niepoprawny**), to ostatecznie wszystkie poprawne procesy odbiorą tę wiadomość |

> [!important] RB vs URB
> W RB zgodność dotyczy tylko procesów **poprawnych**: jeśli wiadomość odebrał wyłącznie proces, który potem uległ awarii, pozostałe nie muszą jej odebrać. W URB zgodność „zaraża” także proces, który **potem** ulegnie awarii — jeśli on odebrał, to odbiorą **wszyscy poprawni**.

Operacje: $send^{URB}(P_i, \mathcal{P}, M)$ (12.5) i $deliver^{URB}(P_j, P_i, M)$ (12.6).

### Algorytm z potwierdzeniami od wszystkich (_all-ack uniform reliable broadcast_)
<sub>slajdy 35–41</sub>

**Założenia**: podstawowe rozgłaszanie niezawodne · niezawodne kanały komunikacyjne · **doskonały detektor awarii**.

```
type PACKET extends FRAME is record of
     origin : PROCESS_ID
     data   : MESSAGE
end record

pcktIn      : PACKET
msgOut      : MESSAGE
correct_i   : set of PROCESS_ID := P
delivered_i : set of MESSAGE := ∅
pending_i   : set of PACKET := ∅
ack_i       : set of pair of ⟨PACKET, PROCESS_ID⟩ := ∅
result      : set of PROCESS_ID
pcktAck     : PACKET
pId         : PROCESS_ID
pckt        : PACKET
```

```
 1. function SELECT(pcktIn: PACKET)
 2.    result := ∅
 3.    for all ⟨pcktAck, pId⟩ ∈ ack_i ∧ pcktAck = pcktIn do
 5.       result := result ∪ {pId}
 6.    end for
 7.    return result
 8. end function

 9. when e_send^URB(P_i, P, msgOut: MESSAGE) do
10.    pcktOut.origin := P_i
11.    pcktOut.data := msgOut
12.    pending_i := pending_i ∪ {pcktOut}
13.    send^BRB(Q_i, Q, pcktOut)
14. end when

15. when e_crash(P_j) do
16.    correct_i := correct_i \ {P_j}
17. end when

18. when e_receive^BRB(Q_j, Q_i, pcktIn: PACKET) do
19.    ack_i := ack_i ∪ ⟨pcktIn, P_j⟩
20.    if pcktIn ∉ pending_i then
21.       pending_i := pending_i ∪ {pcktIn}
22.       send^BRB(Q_i, Q, pcktIn)
23.    end if
24. end when

25. when ∃pckt :: pckt ∈ pending_i ::
       pckt.data ∉ delivered_i ∧ correct_i ⊆ SELECT(pckt) do
26.    delivered_i := delivered_i ∪ {pckt.data}
27.    deliver^URB(pckt.origin, P_i, pckt.data)
28. end when
```

**Idea**: wiadomość jest **dostarczana aplikacji dopiero wtedy**, gdy potwierdziły ją (przez retransmisję BEB) **wszystkie procesy uznawane za poprawne** — warunek `correct_i ⊆ SELECT(pckt)`. Stąd dwa kroki zamiast jednego.

![[rso-w5-s41-ilustracja-urb.png]]
<sub>Ilustracja algorytmu URB — zbiory obok zdarzeń to zbiory procesów, które potwierdziły wiadomość. rso_sum_05.pdf, slajd 41</sub>

**Złożoność** <sub>slajd 42</sub>:

| Przypadek | Czasowa | Komunikacyjna |
|---|---|---|
| optymistyczny | 2 | $n^2$ |
| pesymistyczny | $n + 1$ | $n^2$ |

---
## Porządki dostarczania

### Causal Order (CO)
<sub>rso_sum_05.pdf, slajd 49</sub>

$m_1 \rightarrow m_2$, gdy:
- a) obie wiadomości zostały rozgłoszone przez **ten sam proces**, $m_1$ przed $m_2$,
- b) $m_1$ została **odebrana** przez pewien proces $P_i$, a $m_2$ została **rozgłoszona** przez $P_i$ **po odebraniu** $m_1$,
- c) istnieje wiadomość $m_3$ taka, że dla $m_1$ i $m_3$, lub $m_3$ i $m_2$ zachodzi a) lub b).

### Specyfikacje RFB i RCB
<sub>rso_sum_05.pdf, slajd 50</sub>

| Mechanizm | Własności | Dodatkowy warunek porządku |
|---|---|---|
| **Reliable FIFO Broadcast (RFB)** | RB1, RB2, RB3, RB4 | **FIFO Order**: Jeżeli proces $P_i$ rozgłosił wiadomość $m_1$ **przed** $m_2$, to każdy proces $P_j$ **nie odbierze** $m_2$, jeśli nie odebrał wcześniej $m_1$ |
| **Reliable Causal Broadcast (RCB)** | RB1, RB2, RB3, RB4 | **Causal order**: Proces $P_i$ **nie odbierze** wiadomości $m_2$, dopóki nie odebrał wszystkich wiadomości $m_1$ takich, że $m_1 \rightarrow m_2$ |

### Total Order Broadcast (TO)
<sub>rso_sum_05.pdf, slajd 54</sub>

RB1, RB2, RB3, RB4 plus:

- **Globalne uporządkowanie wiadomości** (_total order_): wszystkie procesy odbierają wiadomości w **tym samym porządku**, tj. jeśli $P_i$ i $P_j$ są **poprawnymi** procesami, które odbierają wiadomość $m$, i jeśli $P_i$ odbiera $m'$ przed $m$, to także $P_j$ odbiera $m'$ przed $m$.
- **Jednolite globalne uporządkowanie wiadomości** (_uniform total order_): jeśli $P_i$ i $P_j$ odbierają wiadomość $m$ i jeśli $P_i$ odbiera $m'$ przed $m$, to także $P_j$ odbiera $m'$ przed $m$ (**bez** ograniczenia do procesów poprawnych).

---
## Zadania sprawdzające z prezentacji
<sub>rso_sum_05.pdf, slajdy 43–48, 51–53, 55–56</sub>

### Jaki rodzaj rozgłaszania przedstawiony jest na rysunku?

![[rso-w5-s43-zadanie.png]]
<sub>slajd 43 — **P1 i P3 nigdy nie odbiorą m2** (P2 odebrał m2, a P1 uległ awarii). Brak zgodności ⇒ **żaden** z omawianych mechanizmów zgodnego rozgłaszania.</sub>

![[rso-w5-s44-zadanie.png]]
<sub>slajd 44 — **Wszystkie poprawne procesy odbiorą m1 i m2 — BEB, RB i URB**</sub>

![[rso-w5-s45-zadanie.png]]
<sub>slajd 45 — **Żaden poprawny proces nie odbierze m — BEB i RB**</sub>

![[rso-w5-s46-zadanie.png]]
<sub>slajd 46 — **Żaden poprawny proces nie odbierze m2, ale wszystkie niepoprawne odbiorą m2 (_we don't care_) — BEB i RB** (nie URB!)</sub>

![[rso-w5-s47-zadanie.png]]
<sub>slajd 47 — **BEB, RB i URB**</sub>

### Podaj przykład przetwarzania, które spełnia własności RB, ale nie spełnia własności URB

![[rso-w5-s48-zadanie.png]]
<sub>slajd 48 — **Jest spełniona własność zgodności, ale nie jednolitej zgodności**: P3 odebrał m i uległ awarii, P2 nigdy nie odbierze.</sub>

### Zaproponuj przetwarzanie spełniające RB, ale nie spełniające FIFO

![[rso-w5-s51-zadanie.png]]
<sub>slajd 51</sub>

### Podaj przykład przetwarzania, które spełnia warunki RFB i nie spełnia warunków RCB

![[rso-w5-s52-zadanie.png]]
<sub>slajd 52 — **$m_1 \rightarrow m_2$, a $m_2$ zostało odebrane przed $m_1$**</sub>

### Zaproponuj przetwarzanie spełniające RB, RFB i RCB

![[rso-w5-s53-zadanie.png]]
<sub>slajd 53</sub>

### Czy na poniższym rysunku jest zapewniony TO?

![[rso-w5-s55-zadanie.png]]
<sub>slajd 55 — **TO: tak; FIFO: nie**</sub>

![[rso-w5-s56-zadanie.png]]
<sub>slajd 56 — **TO: tak; FIFO: tak; causal: tak**</sub>

---
## Podsumowanie hierarchii

```
BEB  (best-effort)      ważność tylko między procesami poprawnymi
 ↓   + zgodność
RB   (regular)          poprawny odebrał ⇒ wszyscy poprawni odbiorą
 ↓   + jednolitość
URB  (uniform)          ktokolwiek odebrał ⇒ wszyscy poprawni odbiorą

RB + FIFO order      = RFB
RB + causal order    = RCB        (RCB ⇒ RFB)
RB + total order     = TO
```

| Mechanizm | Czasowa (opt.) | Komunikacyjna (opt.) | Czasowa (pes.) | Komunikacyjna (pes.) |
|---|---|---|---|---|
| **BEB** | 1 | $n$ | — | — |
| **RB pasywny** | 1 | $n$ | $n$ | $n^2$ |
| **RB aktywny** | 1 | $n^2$ | $n$ | $n^2$ |
| **URB all-ack** | 2 | $n^2$ | $n+1$ | $n^2$ |

---
## Braki w prezentacjach

> [!todo] Grupy procesów — klasyfikacja
> Prezentacja definiuje tylko **grupę** i **obraz grupy**. Brak podziału na grupy **otwarte/zamknięte**, **płaskie/hierarchiczne**, **statyczne/dynamiczne** — wymaganych przez listę zagadnień egzaminacyjnych. Uzupełnić.

> [!todo] Usługa członkostwa i synchronizacja widoków
> Usługa członkostwa (_membership service_) jest tylko **wspomniana** (slajdy 4, 6, 8). Brak omówienia **zmian widoku** (_view change_) i **synchronizacji widoków** (_virtual synchrony_). Uzupełnić.

> [!todo] Algorytmy realizacji porządków FIFO / CO / TO
> Podane są **wyłącznie specyfikacje** RFB, RCB i TO (slajdy 50, 54) — **brak algorytmów** ich realizacji: numerów sekwencyjnych dla FIFO, zegarów wektorowych dla CO, sekwencera / uzgadniania priorytetów (**ISIS**) dla TO. Uzupełnić.

> [!todo] Zegary logiczne
> Zegary skalarne i wektorowe — będące podstawą porządku przyczynowego — omówione są w **innym wykładzie**: [[RSO 08 Czas wirtualny i złożoność algorytmów#Zegar skalarny]] i [[RSO 08 Czas wirtualny i złożoność algorytmów#Zegar wektorowy]]. Nie ma w wykładzie 5 powiązania ich z RCB.

> [!todo] Detektory awarii
> Algorytmy RB pasywny i URB zakładają „**doskonały detektor awarii**”, ale prezentacja **nie definiuje**, czym jest detektor awarii ani jakie ma klasy (P, ◊P, S, ◊S). Uzupełnić.

---
## Powiązania
- Operacje grupowe $send(P_i, \mathcal{P}_i^{R}, M)$ i zdarzenia $e\_send$/$e\_receive$ → [[RSO 07 Model środowiska przetwarzania#Operacje komunikacyjne]]
- Uporządkowanie przyczynowe środowiska i kanały FIFO → [[RSO 08 Czas wirtualny i złożoność algorytmów#Środowisko zachowujące uporządkowanie przyczynowe]]
- Zegary wektorowe jako narzędzie realizacji CO → [[RSO 08 Czas wirtualny i złożoność algorytmów#Zegar wektorowy]]
- Replikacja jako zastosowanie rozgłaszania → [[RSO 02 Danocentryczne modele spójności]]
