---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 6
source: "rso_sum_07.pdf"
slajdy: "14–53"
---
# RSO 06. Zakleszczenie w systemach rozproszonych
---
> Druga część wykładu 7 definiuje **zakleszczenie rozproszone**, warunki konieczne, graf przydziału zasobów i **graf oczekiwania (WFG)**, strategie postępowania, a następnie formalizuje zakleszczenie w **modelu AND** i **modelu OR** wraz z klasyfikacją problemów detekcji i algorytmami **Chandy-Misra-Haas** dla obu modeli.

---
## Wprowadzenie i definicja
<sub>rso_sum_07.pdf, slajdy 14–15</sub>

Procesy tworzące przetwarzanie rozproszone komunikują się ze sobą za pomocą mechanizmu wymiany wiadomości, realizując wspólny cel przetwarzania. Jedne procesy wysyłają komunikaty zawierające **żądania przydziału pewnych zasobów**, inne — w odpowiedzi — przesyłają ewentualnie komunikaty **potwierdzające przydział** żądanych zasobów.

> **Definicja nieformalna**: w przetwarzaniu rozproszonym może w ogólności wystąpić sytuacja, w której **wszystkie procesy pewnego niepustego zbioru procesów oczekują na wiadomości** (potwierdzające na przykład przydział zasobów) **od innych procesów tego właśnie zbioru**. Stan taki nazywany jest **zakleszczeniem rozproszonym** (ang. _distributed deadlock_).

## Warunki konieczne zakleszczenia
<sub>rso_sum_07.pdf, slajd 16</sub>

1. **Wzajemne wykluczanie**
2. **Istnienie procesu, który blokuje zasób, a jednocześnie sam czeka na zasób blokowany przez inny proces**
3. **Brak wywłaszczania zasobów**
4. **Czekanie cykliczne**

## Graf przydziału zasobów
<sub>rso_sum_07.pdf, slajd 17</sub>

![[rso-w7-s17-graf-przydzialu-zasobow.png]]
<sub>Graf przydziału zasobów rozpięty na dwóch stanowiskach — procesy $P_1$–$P_4$ oraz zasoby na stanowiskach 1 i 2. rso_sum_07.pdf, slajd 17</sub>

## Strategie postępowania z zakleszczeniami
<sub>rso_sum_07.pdf, slajd 18</sub>

**Sposoby postępowania**:
- **niedopuszczanie** do zakleszczeń,
- **dopuszczanie** do zakleszczeń i **późniejsze ich usuwanie**,
- **ignorowanie** zakleszczeń.

**Metody niedopuszczania do zakleszczeń**:
- **zapobieganie** zakleszczeniom,
- **unikanie** zakleszczeń.

### Zapobieganie zakleszczeniom w systemach rozproszonych
<sub>rso_sum_07.pdf, slajd 19</sub>
- Przydział **priorytetów** dla procesów przy dostępie do zasobów.
- Zastosowanie **znaczników czasowych** — możliwość pozbycia się problemu zagłodzenia procesów o niskich priorytetach. Metody:
  1. **Czekanie albo śmierć** (ang. _wait-die_),
  2. **Zranienie albo czekanie** (ang. _wound-wait_).
- **Wadą** powyższych algorytmów jest występowanie **niepotrzebnych wywłaszczeń**.

### Wykrywanie zakleszczeń — graf oczekiwania (WFG)
<sub>rso_sum_07.pdf, slajd 20</sub>

- Do wykrywania zakleszczeń używa się **grafu oczekiwania (WFG)**, który reprezentuje stan przydziału zasobów.
- Jeżeli stan przedstawiany przez graf dotyczy **całego systemu rozproszonego**, mówimy o **globalnym grafie oczekiwania**.
- Jeżeli stan reprezentowany przez graf dotyczy **tylko danego stanowiska**, jest to **lokalny graf oczekiwania**.

![[rso-w7-s20-graf-oczekiwania-wfg.png]]
<sub>Graf oczekiwania — cykl $P_1 \leftarrow P_2 \leftarrow P_4 \leftarrow P_3 \leftarrow P_2$. rso_sum_07.pdf, slajd 20</sub>

---
## Formalizm — procesy aktywne i pasywne
<sub>rso_sum_07.pdf, slajdy 21–25</sub>

W każdej chwili proces może być w jednym z dwóch stanów: **aktywnym** albo **pasywnym**.

- **Proces aktywny** może realizować przetwarzanie wykonując operacje odpowiadające zajściu **zdarzeń wewnętrznych i komunikacyjnych**.
- W stanie **pasywnym** procesu $P_i$ ($passive_i = True$) dopuszczalne są natomiast **co najwyżej zdarzenia odbioru**.
- Zmiana stanu procesu z pasywnego na aktywny uwarunkowana jest osiągnięciem gotowości przez choćby jedno z dopuszczalnych zdarzeń odbioru, czyli spełnieniem tak zwanego **warunku uaktywnienia**.

> **Warunek uaktywnienia** (ang. _activation condition_) procesu $P_i$ związany jest ze **zbiorem warunkującym** $\mathcal{D}_i$, zbiorem $\mathcal{P}_i^{A}$, oraz predykatem $activate_i(\mathcal{X})$.

> **Zbiór warunkujący** (ang. _dependent set_) jest sumą mnogościową zbiorów $\mathcal{P}_i^{S}$ wszystkich zdarzeń odbioru dopuszczalnych w danej chwili.

Warunek uaktywnienia wyrażony przez predykat:
$$ready_i(\mathcal{X}) \equiv (\mathcal{P}_i^{A} \supseteq \mathcal{X}) \wedge activate_i(\mathcal{X})$$

Gdy proces jest uaktywniany, wiadomości, których dostarczanie doprowadziło do spełnienia warunku uaktywnienia, są **atomowo** pobierane z buforów wejściowych i dalej przetwarzane.

> [!tip] To samo w wykładzie 1
> Slajdy 21–25 są powtórzeniem materiału z wykładu 1 — zob. [[RSO 07 Model środowiska przetwarzania#Warunek uaktywnienia]], gdzie podana jest także pełna definicja predykatu $activate_i(\mathcal{X})$ (wzór 2.15).

## Definicja problemu
<sub>rso_sum_07.pdf, slajd 26</sub>

> Przez $deadlock(\mathcal{B})$ oznaczamy predykat stwierdzający, że w danej chwili $\tau$, **niepusty zbiór procesów $\mathcal{B}$ jest zbiorem procesów zakleszczonych**.

---
## Model AND
<sub>rso_sum_07.pdf, slajdy 27–28</sub>

> W modelu **AND** proces pasywny staje się aktywnym, jeżeli dotarły wiadomości **od wszystkich procesów** tworzących zbiór warunkujący. Model ten nazywany jest również **modelem zasobowym**.

![[rso-w7-s27-model-and.png]]
<sub>Model AND — proces czeka na wiadomości od **wszystkich** procesów zbioru $\mathcal{D}_i$. rso_sum_07.pdf, slajd 27</sub>

### Zakleszczenie w modelu AND
<sub>slajd 28</sub>

$$
\begin{aligned}
deadlock(\mathcal{B}) \equiv\ & (\mathcal{B} \subseteq \mathcal{P}) \wedge (\mathcal{B} \neq \varnothing)\ \wedge \\
& (\forall P_i :: P_i \in \mathcal{B} :: (\, passive_i\ \wedge \\
& \quad (\exists P_j :: P_j \in \mathcal{D}_i \cap \mathcal{B} :: (\neg\, \textit{in-transit}_i[j] \wedge \neg\, available_i[j]))\,))
\end{aligned}
$$

Czytanie: **każdy** proces zbioru $\mathcal{B}$ jest pasywny i **istnieje** proces $P_j$ w jego zbiorze warunkującym należący do $\mathcal{B}$, od którego **żadna wiadomość nie jest ani w tranzycie, ani dostępna**. W modelu AND wystarczy **jedna** brakująca wiadomość, żeby proces pozostał pasywny.

## Model OR
<sub>rso_sum_07.pdf, slajdy 29–30</sub>

> W modelu **OR** do uaktywnienia procesu wystarczy **jedna wiadomość od któregokolwiek** z procesów ze zbioru warunkującego. Model ten nazywany jest również **modelem komunikacyjnym**.

![[rso-w7-s29-model-or.png]]
<sub>Model OR — wystarczy jedna wiadomość z $\mathcal{D}_i$. rso_sum_07.pdf, slajd 29</sub>

### Zakleszczenie w modelu OR
<sub>slajd 30</sub>

$$
\begin{aligned}
deadlock(\mathcal{B}) \equiv\ & (\mathcal{B} \subseteq \mathcal{P}) \wedge (\mathcal{B} \neq \varnothing)\ \wedge \\
& (\forall P_i :: P_i \in \mathcal{B} :: (\, passive_i\ \wedge\ \mathcal{D}_i \subseteq \mathcal{B}\ \wedge \\
& \quad (\forall P_j :: P_j \in \mathcal{D}_i :: (\neg\, \textit{in-transit}_i[j] \wedge \neg\, available_i[j]))\,))
\end{aligned}
$$

> [!important] Różnica AND vs OR
> | | AND | OR |
> |---|---|---|
> | Kwantyfikator po $P_j$ | $\exists$ — wystarczy **jedna** brakująca wiadomość | $\forall$ — muszą brakować **wszystkie** |
> | Dodatkowy warunek | $P_j \in \mathcal{D}_i \cap \mathcal{B}$ | $\mathcal{D}_i \subseteq \mathcal{B}$ — **cały** zbiór warunkujący musi być w $\mathcal{B}$ |
>
> Zakleszczenie w modelu OR jest więc **trudniejsze do wystąpienia**: cały zbiór warunkujący musi być zakleszczony.

---
## Przykłady zakleszczeń
<sub>rso_sum_07.pdf, slajdy 31–33</sub>

![[rso-w7-s31-przyklady-zakleszczen-wfg.png]]
<sub>**Wait-For Graph (WFG)**: $N_1 \ldots N_5$ — węzły środowiska przetwarzania; $P_1 \ldots P_5$ — procesy wykonywane w odpowiednich węzłach; łuk $P_i \rightarrow P_j$ reprezentuje fakt, że proces $P_i$ **oczekuje na wiadomość** od procesu $P_j$. Zasady: każdy proces w grafie z łukiem wychodzącym jest **pasywny**; procesy bez łuków wychodzących są **aktywne**; zakładamy, że wszystkie kanały są puste. rso_sum_07.pdf, slajd 31</sub>

![[rso-w7-s32-przyklad-model-and.png]]
<sub>**Przykład — model AND.** Zbiory warunkujące: $\mathcal{D}_1 = \{P_4, P_5\}$, $\mathcal{D}_2 = \{P_1, P_4\}$, $\mathcal{D}_3 = \{P_2\}$, $\mathcal{D}_4 = \{P_3\}$. $deadlock(\mathcal{B})$ zachodzi dla $\mathcal{B} = \{P_1, P_2, P_3, P_4\}$. rso_sum_07.pdf, slajd 32</sub>

![[rso-w7-s33-przyklad-model-or.png]]
<sub>**Przykład — model OR.** Zbiory warunkujące: $\mathcal{D}_1 = \{P_4, P_5\}$, $\mathcal{D}_2 = \{P_4\}$, $\mathcal{D}_3 = \{P_2\}$, $\mathcal{D}_4 = \{P_2, P_3\}$. $deadlock(\mathcal{B})$ zachodzi dla $\mathcal{B} = \{P_2, P_3, P_4\}$. rso_sum_07.pdf, slajd 33</sub>

> [!note] Dlaczego $P_1$ nie jest zakleszczony w modelu OR
> $\mathcal{D}_1 = \{P_4, P_5\}$, a $P_5$ jest **aktywny** (brak łuków wychodzących), więc $\mathcal{D}_1 \not\subseteq \mathcal{B}$ — warunek modelu OR nie jest spełniony. W modelu AND wystarczyło, że $P_4 \in \mathcal{D}_1 \cap \mathcal{B}$.

---
## Klasyfikacja problemów detekcji zakleszczenia
<sub>rso_sum_07.pdf, slajdy 34–38</sub>

| Problem | Predykat | Nr |
|---|---|---|
| **Detekcja wystąpienia zakleszczenia** — czy istnieje w pewnej chwili zbiór $\mathcal{B}$, dla którego predykat $deadlock(\mathcal{B})$ jest prawdziwy? | $dE \equiv (\exists \mathcal{B} :: deadlock(\mathcal{B}))$ | (5.9) |
| **Detekcja zakleszczenia procesu $P_i$** | $dP_i \equiv ((\exists \mathcal{B} :: deadlock(\mathcal{B})) \wedge P_i \in \mathcal{B})$ | (5.10) |
| **Detekcja zakleszczenia zbioru procesów** — znalezienie zbioru $\mathcal{B}^{*}$ | $deadlock(\mathcal{B}^{*}) \vee ((\mathcal{B}^{*} = \varnothing) \wedge (\nexists \mathcal{B} :: deadlock(\mathcal{B})))$ | (5.11) |
| **Detekcja maksymalnego zbioru zakleszczonego** | $(deadlock(\mathcal{B}^{*}) \vee \mathcal{B}^{*} = \varnothing) \wedge maxdead(\mathcal{B}^{*})$, gdzie $maxdead(\mathcal{B}^{*}) \equiv (\forall \mathcal{B} :: deadlock(\mathcal{B}) \Rightarrow (\mathcal{B} \subseteq \mathcal{B}^{*}))$ | (5.12), (5.13) |

## Model aplikacyjnego przetwarzania rozproszonego
<sub>rso_sum_07.pdf, slajd 39</sub>

![[rso-w7-s39-model-aplikacyjny.png]]
<sub>Proces $P_i$ wysyła do procesów swojego zbioru warunkującego $\mathcal{D}_i$ wiadomości **REQUEST**, a otrzymuje **GRANT** albo **CANCEL**. rso_sum_07.pdf, slajd 39</sub>

---
## Algorytm Chandy-Misra-Haas dla modelu AND
<sub>rso_sum_07.pdf, slajdy 40–43</sub>

Algorytm **sondujący** (_probe_): zablokowany proces rozsyła sondę ze swoim identyfikatorem; jeżeli sonda **wróci do inicjatora**, wykryte jest zakleszczenie.

```
type PROBE extends FRAME is record of
    initIndex : INTEGER
end record

probeOut          : PROBE
granted_i         : array [1..n] of BOOLEAN := False
D_i               : set of PROCESS_ID
recvProbe_i       : array [1..n] of BOOLEAN := False
α_i               : INTEGER
k                 : INTEGER
deadlockDetected_i: BOOLEAN := False
```

```
 1. when e_start(Q_α, DeadlockDetection) do
 2.    if passive_α
 3.    then
 4.       for all Q_k :: P_k ∈ D_α do
 5.          probeOut.initIndex := α
 6.          send(Q_α, Q_k, probeOut)
 7.       end for
 8.    end if
 9. end when

10. when e_activate(P_i) do
11.    for all k ∈ {1, 2, ..., n} do
12.       recvProbe_i[k] := False
13.    end for
14. end when

15. when e_receive(Q_j, Q_i, probeIn: PROBE) do
16.    α_i := probeIn.initIndex
17.    if passive_i ∧ (¬recvProbe_i[α_i]) ∧ (¬granted_i[j])
18.    then
19.       recvProbe_i[α_i] := True
20.       if α_i = i
21.       then
22.          deadlockDetected_i := True
23.          decide(deadlockDetected_i)
24.       else
25.          probeOut.initIndex := α_i
26.          for all Q_k :: P_k ∈ D_i do
27.             send(Q_i, Q_k, probeOut)
28.          end for
29.       end if
30.    end if
31. end when
```

**Kluczowe warunki propagacji sondy** (linia 17):
- $passive_i$ — proces musi być pasywny (zablokowany),
- $\neg recvProbe_i[\alpha_i]$ — sonda od tego inicjatora **jeszcze nie przeszła** przez ten proces (unikanie zapętlenia),
- $\neg granted_i[j]$ — zasób od $P_j$ **nie został przyznany**, więc proces rzeczywiście na niego czeka.

**Wykrycie**: $\alpha_i = i$ — sonda **wróciła do swojego inicjatora**, czyli w WFG istnieje cykl.

**Reset**: przy uaktywnieniu procesu (`e_activate`) czyszczone są wszystkie znaczniki `recvProbe_i`.

---
## Detekcja zakleszczenia dla modelu OR
<sub>rso_sum_07.pdf, slajdy 44–53</sub>

> Algorytm detekcji zakleszczenia dla modelu **OR** opiera się na **przetwarzaniu dyfuzyjnym** (ang. _query computation_).

> **Twierdzenie 5.2** (slajd 45). Jeżeli inicjator $Q_\alpha$ rozpoczyna detekcję w chwili, gdy jego proces aplikacyjny $P_\alpha$ jest zakleszczony, to $Q_\alpha$ stwierdzi zakleszczenie procesu $P_\alpha$ w skończonym czasie (algorytm detekcji zakończy się).

> **Twierdzenie 5.2** (slajd 46, kontynuacja). Jeżeli inicjator $Q_\alpha$ deklaruje, że jego proces aplikacyjny $P_\alpha$ jest zakleszczony, to $P_\alpha$ należy do pewnego zbioru procesów zakleszczonych w chwili zakończenia algorytmu.

> [!note] Numeracja twierdzeń
> Oba slajdy (45 i 46) noszą w oryginale etykietę „Twierdzenie 5.2" — są to dwie części tego samego twierdzenia (zakończenie i poprawność).

### Typy wiadomości
<sub>slajd 47</sub>

```
type CONTROL extends FRAME is record of
   initIndex : INTEGER
   queryNo   : INTEGER
end record

type QUERY extends CONTROL
type REPLY extends CONTROL
```

### Zasada działania
<sub>slajdy 48–50</sub>

- **Zablokowany proces inicjuje** proces wykrywania zakleszczenia wysyłając wiadomość QUERY do wszystkich procesów ze swojego zbioru warunkującego.
- Kiedy **aktywny** proces otrzymuje wiadomość QUERY lub REPLY — **unieważnia ją** (ignoruje).
- Kiedy **zablokowany** proces $P_k$ otrzymuje wiadomość $\text{QUERY}(i, j, k)$, podejmuje działania:
  1. Jeśli to **pierwsza** wiadomość QUERY otrzymana przez $P_k$ i zainicjowana przez $P_i$ (_engaging query_), $P_k$ **propaguje** QUERY do wszystkich procesów ze swojego zbioru warunkującego i nadaje lokalnej wartości $num_k(i)$ wartość równą **liczbie wysłanych wiadomości**.
  2. Jeśli otrzymana wiadomość **nie jest pierwsza** i $P_k$ był zablokowany od otrzymania pierwszej wiadomości QUERY od $P_i$, to $P_k$ **zwraca REPLY**. W przeciwnym razie ignoruje otrzymaną wiadomość.
- Proces $P_k$ utrzymuje zmienną logiczną $wait_k(i)$ oznaczającą fakt, że jest zablokowany od otrzymania ostatniej wiadomości QUERY.
- Kiedy zablokowany proces otrzymuje wiadomość $\text{REPLY}(i, j, k)$, **zmniejsza** wartość $num_k(i)$, jeśli zachodzi $wait_k(i)$.
- Proces **odsyła odpowiedź** na wiadomość QUERY dopiero po uzyskaniu wiadomości REPLY na każdą wiadomość QUERY, którą on sam wysłał.
- **Inicjator wykrywa zakleszczenie** po otrzymaniu wiadomości REPLY na wszystkie wiadomości QUERY, które wysłał.

### Zapis algorytmu
<sub>slajdy 51–53</sub>

**Inicjacja przetwarzania dyfuzyjnego dla zablokowanego procesu $P_i$:**
```
send query(i, i, j) to all processes Pj in the dependent set DSi of Pi;

num_i(i) = |DSi|;
wait_i(i) = true;
```

**Gdy zablokowany proces $P_k$ otrzymuje $query(i, j, k)$:**
```
if this is the engaging query for process Pi then
   send query(i, k, m) to all Pm in its dependent set DSk;
   num_k(i) = |DSk|;
   wait_k(i) = true
else
   if wait_k(i) then send a reply(i, k, j) to Pj
```

**Gdy proces $P_k$ otrzymuje $reply(i, j, k)$:**
```
if wait_k(i) then
   num_k(i) = num_k(i) − 1;
   if num_k(i) = 0 then
      if i = k then declare a deadlock
      else
         send reply(i, k, m) to the process Pm
         which sent the engaging query.
```

> [!important] Analogia
> Struktura algorytmu jest identyczna z algorytmem **Dijkstry-Scholtena** detekcji zakończenia: _engaging query_ ≙ rodzic ($engager$), $num_k(i)$ ≙ licznik $sentNo$, REPLY ≙ sygnał. Zob. [[RSO 10 Detekcja zakończenia#Model przetwarzania dyfuzyjnego i algorytm Dijkstry-Scholtena]].

---
## Porównanie algorytmów CMH

| | Model AND | Model OR |
|---|---|---|
| **Mechanizm** | sonda (PROBE) rozsyłana po WFG | przetwarzanie dyfuzyjne (QUERY / REPLY) |
| **Wykrycie** | sonda **wraca do inicjatora** ($\alpha_i = i$) | inicjator otrzymał REPLY na **wszystkie** wysłane QUERY |
| **Stan lokalny** | `recvProbe_i[]`, `granted_i[]` | $num_k(i)$, $wait_k(i)$ |
| **Odpowiedzi zwrotne** | brak — sonda tylko się propaguje | tak — REPLY wraca do rodzica |
| **Reset** | przy `e_activate` czyszczone `recvProbe_i` | aktywny proces unieważnia QUERY/REPLY |

---
## Braki w prezentacjach

> [!todo] Algorytm Brachy-Touega
> Nie występuje w żadnej z 7 prezentacji (0 trafień frazy „Bracha"). Uzupełnić.

> [!todo] Złożoność algorytmów CMH
> Prezentacja **nie podaje** złożoności czasowej ani komunikacyjnej algorytmów Chandy-Misra-Haas dla modelu AND i OR. Uzupełnić (dla AND: $O(m)$ komunikatów, gdzie $m$ = liczba krawędzi WFG).

> [!todo] Dowody twierdzeń
> Twierdzenia 5.2 (slajdy 45–46) podane są **bez dowodu**; dla algorytmu CMH AND **nie ma** twierdzenia o poprawności w ogóle.

> [!todo] Zakleszczenia fałszywe (phantom deadlocks)
> Problem wykrywania nieistniejących zakleszczeń wskutek nieaktualnych migawek WFG **nie jest omawiany**.

> [!todo] Metody usuwania zakleszczeń
> Slajd 18 wymienia „dopuszczanie do zakleszczeń i późniejsze ich usuwanie", ale **metody usuwania** (wybór ofiary, wywłaszczenie, rollback) **nie są omówione**.

> [!todo] Modele żądań inne niż AND / OR
> Modele _k spośród r_, OR-AND i predykatowy wprowadzone są w wykładzie 1 ([[RSO 07 Model środowiska przetwarzania#Modele żądań]]), ale **nie ma** dla nich definicji zakleszczenia ani algorytmów detekcji.

---
## Powiązania
- Procesy aktywne/pasywne, warunek uaktywnienia, modele żądań AND/OR → [[RSO 07 Model środowiska przetwarzania#Modele żądań]]
- Predykaty $\textit{in-transit}$ i $available$ na stanie kanału → [[RSO 07 Model środowiska przetwarzania#Stan kanału i predykaty]]
- Przetwarzanie dyfuzyjne i algorytm Dijkstry-Scholtena (ta sama struktura) → [[RSO 10 Detekcja zakończenia]]
- Detekcja zakleszczenia jako przypadek oceny stanu globalnego → [[RSO 09 Stan globalny i migawki#Po co wyznaczać stan globalny]]
- Pierwsza część wykładu 7 — wzajemne wykluczanie → [[RSO 04 Algorytmy wzajemnego wykluczania]]
