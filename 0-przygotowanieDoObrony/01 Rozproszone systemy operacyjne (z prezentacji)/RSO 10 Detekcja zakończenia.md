---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
source: "rso_sum_04.pdf"
slajdy: "1–43"
---
# RSO 10. Problem detekcji zakończenia
---
> Wykład 4 stawia problem: **jak stwierdzić, że przetwarzanie rozproszone się zakończyło**, skoro proces pasywny może zostać uaktywniony przez wiadomość będącą jeszcze w kanale. Rozróżnia zakończenie **dynamiczne** i **statyczne**, a następnie podaje trzy algorytmy: **Dijkstra-Feijen-van Gasteren** (model synchroniczny, pierścień, kolory), **Dijkstry-Scholtena** (przetwarzanie dyfuzyjne, graf) i **Misra '83** (systemy asynchroniczne, cykl kanałów).

---
## Przykład motywujący — sortowanie rozproszone
<sub>rso_sum_04.pdf, slajdy 2–5</sub>

Rozważmy problem sortowania rozproszonego zbioru $\mathcal{X}$ składającego się z $v$ różnych liczb naturalnych, w środowisku rozproszonym o $n$ węzłach (procesorach), $n < v$.

- Zadaniem każdego procesu jest uporządkowanie przypisanej mu części zbioru liczb naturalnych i wyznaczenie **elementu minimalnego**.
- Elementy minimalne są wysyłane do **lewych** sąsiadów.
- Po otrzymaniu wiadomości z wartością minimalną, proces wyznacza **element maksymalny** i wysyła go do **prawego** sąsiada.

**Definicje**: zbiór wstępnie podzielony na podzbiory $\mathcal{X}_i$; $v_i$ — liczba elementów zbioru $\mathcal{X}_i$; $min_i$ / $max_i$ — minimalny / maksymalny element $\mathcal{X}_i$; $P_i$ — procesy tworzące przetwarzanie rozproszone topologii **łańcucha**, skojarzone ze zbiorami $\mathcal{X}_i$. Pary procesów składowych $P_i$, $P_{i+1}$ połączone są kanałami dwukierunkowymi.

![[rso-w4-s04-sortowanie-rozproszone-przyklad.png]]
<sub>Sortowanie rozproszone — przykład, rso_sum_04.pdf, slajd 4</sub>

---
## Definicje zakończenia
<sub>rso_sum_04.pdf, slajdy 6–11</sub>

> **Definicja nieformalna**: problem detekcji zakończenia przetwarzania rozproszonego polega na sprawdzeniu, czy wszystkie procesy przetwarzania są w **stanie pasywnym** oraz czy **żadna wiadomość będąca w kanale** (transmitowana lub dostępna) **nie uaktywni** któregokolwiek z tych procesów.

| Rodzaj | Definicja |
|---|---|
| **zakończenie dynamiczne** | przetwarzanie rozproszone jest w stanie zakończenia dynamicznego, jeżeli **żaden proces składowy nie będzie już nigdy uaktywniony**. Stan ten będzie utrzymywany **pomimo, że pewne wiadomości są wciąż transmitowane, a pewne wiadomości są już dostępne** |
| **zakończenie statyczne** | przetwarzanie jest w stanie zakończenia statycznego, jeżeli: wszystkie procesy są pasywne · wszystkie wiadomości znajdujące się w kanałach są dostępne · dla żadnego procesu nie jest spełniony warunek uaktywnienia |
| **klasyczna definicja** | przetwarzanie rozproszone jest w stanie zakończenia, jeżeli w danej chwili **wszystkie procesy są pasywne i wszystkie kanały są puste** |

> **Problem detekcji zakończenia** przetwarzania rozproszonego obejmującego zbiór procesów sprowadza się do sprawdzenia, czy przetwarzanie osiągnęło określony stan zakończenia.

> [!tip] Powiązanie
> Pojęcia procesu aktywnego/pasywnego oraz warunku uaktywnienia pochodzą z wykładu 1 — zob. [[RSO 07 Model środowiska przetwarzania#Warunek uaktywnienia]].

---
## Model przetwarzania synchronicznego
<sub>rso_sum_04.pdf, slajd 12</sub>

W modelu **przetwarzania synchronicznego** przyjmuje się, że transmisje są **natychmiastowe**. Stąd kanały mogą być uznane za puste przez cały czas i problem zakończenia sprowadza się do sprawdzenia, czy **wszystkie procesy są jednocześnie pasywne**.

---
## Algorytm Dijkstra, Feijen, van Gasteren
<sub>rso_sum_04.pdf, slajdy 13–22</sub>

### Koncepcja
<sub>slajd 13</sub>
- Monitorom przypisany jest kolor: $White$ lub $Black$ (początkowo $White$).
- W **pierścieniu** przesyłany jest znacznik (z przypisanym kolorem).
- Początkowo monitory mają kolor $White$, a zmieniają kolor na $Black$, gdy odpowiadający im proces aplikacyjny wyśle wiadomość do procesu **o indeksie większym**.
- Znacznik jest przesyłany dalej, gdy obserwowany proces staje się **pasywny**.
- Po wysłaniu znacznika monitorowi przypisywany jest kolor $White$.
- Algorytm kończy się, gdy znacznik koloru $White$ dotrze do inicjatora.

Dla uproszczenia prezentacji wykorzystano funkcje:
$$succ(i) = (i) \bmod_n + 1 \qquad pred(i) = (i+n-2) \bmod_n + 1$$

![[rso-w4-s14-runda-niepowodzenie.png]]
<sub>Runda zakończona niepowodzeniem — znacznik wraca **czarny**, konieczna kolejna iteracja. rso_sum_04.pdf, slajd 14</sub>

![[rso-w4-s15-runda-sukces.png]]
<sub>Runda zakończona sukcesem — znacznik wraca **biały**, wykryto zakończenie. rso_sum_04.pdf, slajd 15</sub>

### Zapis algorytmu
<sub>slajdy 16–22</sub>

```
type PACKET extends FRAME is record of
     data : MESSAGE
end record

type TOKEN extends FRAME is record of
     colour : enum {White, Black}
end record

msgIn                 : MESSAGE
pcktOut               : PACKET
tokenOut              : TOKEN
tokenPresent_i        : BOOLEAN := False
procColour_i          : enum {White, Black} := White
terminationDetected_i : BOOLEAN := False
```

```
 1. procedure InitProc()
 2.    tokenOut.colour := White
 3.    send(Q_α, Q_pred(α), tokenOut)
 4.    tokenPresent_α := False
 5.    procColour_α := White
 6. end procedure

 7. when e_start(Q_α, TerminationDetection) do
 8.    wait until passive_α
 9.    InitProc()
10. end when

11. when e_send(P_i, P_j, msgOut: MESSAGE) do
12.    if i < j then
13.       procColour_i := Black
14.    end if
15.    pcktOut.data := msgOut
16.    send(Q_i, Q_j, pcktOut)
17. end when

18. when e_receive(Q_j, Q_i, pcktIn: PACKET) do
19.    msgIn := pcktIn.data
20.    deliver(P_j, P_i, msgIn)
21. end when

22. when e_receive(Q_succ(i), Q_i, tokenIn: TOKEN) do
23.    tokenPresent_i := True
24.    wait until passive_i
25.    if i = α then
26.       if procColour_i = White ∧ tokenIn.colour = White then
27.          terminationDetected_i := True
28.          decide(terminationDetected_i)
29.       else
30.          InitProc()
31.       end if
32.    else
33.       tokenOut.colour := tokenIn.colour
34.       if procColour_i = Black then
35.          tokenOut.colour := Black
36.       end if
37.       send(Q_i, Q_pred(i), tokenOut)
38.       tokenPresent_i := False
39.       procColour_i := White
40.    end if
41. end when
```

> [!note] Kierunek obiegu
> Znacznik krąży **w kierunku poprzednika** ($Q_{pred(i)}$), a odbierany jest **od następnika** ($Q_{succ(i)}$) — czyli przeciwnie do numeracji. Dlatego „czarnym” czyni proces wysłanie wiadomości „w górę”, do procesu o **większym** indeksie: taka wiadomość mogłaby uaktywnić proces, który znacznik już odwiedził.

---
## Model przetwarzania dyfuzyjnego i algorytm Dijkstry-Scholtena
<sub>rso_sum_04.pdf, slajdy 23–35</sub>

### Model
<sub>slajdy 23–24</sub>

> **Przetwarzanie dyfuzyjne** (ang. _diffusing computation_) jest specyficznym przetwarzaniem rozproszonym, w którym wyróżnia się:
> - **inicjatora** — może w dowolnej chwili rozpocząć przetwarzanie dyfuzyjne wysyłając wiadomość aplikacyjną do jednego lub wielu procesów kooperujących,
> - **hierarchię** kooperujących procesów.

**Założenia dodatkowe**:
- proces aktywny staje się procesem pasywnym tylko w wyniku pewnego **zdarzenia wewnętrznego**,
- proces **zawsze** staje się aktywny po otrzymaniu wiadomości,
- proces pasywny może stać się aktywny **tylko** w wyniku otrzymania wiadomości.

### Koncepcja
<sub>slajd 25</sub>
- Monitory procesów aplikacyjnych przesyłają wiadomości kontrolne (**sygnały**) jako pewnego rodzaju **odpowiedzi** na wiadomości aplikacyjne.
- Monitor pasywnego inicjatora może stwierdzić zakończenie przetwarzania po odebraniu wiadomości kontrolnych od **wszystkich** monitorów związanych z procesami uaktywnionymi przez inicjatora.

![[rso-w4-s27-graf-dyfuzyjny-sygnaly.png]]
<sub>Graf przetwarzania dyfuzyjnego — sygnały wracają w stronę inicjatora. rso_sum_04.pdf, slajd 27 (slajd 26 pokazuje ten sam graf ze strzałkami w kierunku wiadomości aplikacyjnych).</sub>

### Zapis algorytmu
<sub>slajdy 28–34</sub>

```
type PACKET extends FRAME is record of
     data : MESSAGE
end record

type SIGNAL extends FRAME

msgIn                 : MESSAGE
pcktOut               : PACKET
signalIn              : SIGNAL
engager_i             : PROCESS_ID
notEngager_i          : set of PROCESS_ID
recvNo_i              : INTEGER := 0
sentNo_i              : INTEGER := 0
terminationDetected_i : BOOLEAN := False
```

```
 1. when e_start(P_α, P_α^R, msgOut: MESSAGE, DiffusingComputation) do
 2.    Q_α^R := {Q_j : P_j ∈ P_α^R}
 3.    pcktOut.data := msgOut
 4.    sentNo_α := |Q_α^R|
 5.    send(Q_α, Q_α^R, pcktOut)
 6. end when

 7. when e_send(Q_i, Q_j, signalOut: SIGNAL) do
 8.    if recvNo_i = 1 ∧ sentNo_i = 0 ∧ passive_i
 9.    then
10.       Q_j := engager_i
11.       send(Q_i, Q_j, signalOut)
12.    else
13.       for Q_j ∈ notEngager_i do
14.          notEngager_i := notEngager_i \ {Q_j}
15.          send(Q_i, Q_j, signalOut)
16.       end for
17.    end if
18.    recvNo_i := recvNo_i - 1
19. end when

20. when e_receive(Q_j, Q_i, signalIn: SIGNAL) do
21.    sentNo_i := sentNo_i - 1
22.    if Q_i = Q_α ∧ sentNo_i = 0 then
23.       wait until passive_i
24.       terminationDetected_i := True
25.       decide(terminationDetected_i)
26.    end if
27. end when

28. when e_send(P_i, P_j, msgOut: MESSAGE) do
29.    pcktOut.data := msgOut
30.    sentNo_i := sentNo_i + 1
31.    send(Q_i, Q_j, pcktOut)
32. end when

33. when e_receive(Q_j, Q_i, pcktIn: PACKET) do
34.    if recvNo_i = 0 then
35.       engager_i := Q_j
36.    else
37.       notEngager_i := notEngager_i ∪ {Q_j}
38.    end if
39.    recvNo_i := recvNo_i + 1
40.    msgIn := pcktIn.data
41.    deliver(P_j, P_i, msgIn)
42. end when
```

**Kluczowa idea**: pierwszy nadawca, który uaktywnił proces $P_i$, zostaje jego **rodzicem** ($engager_i$) w drzewie dyfuzyjnym. Sygnał do rodzica wysyłany jest **dopiero**, gdy proces jest pasywny, odebrał tylko jedno „uaktywniające” zobowiązanie ($recvNo_i = 1$) i spłacił wszystkie własne ($sentNo_i = 0$). Sygnały do pozostałych nadawców ($notEngager_i$) odsyłane są od razu.

> **Twierdzenie.** Jeżeli przetwarzanie dyfuzyjne uległo zakończeniu, to fakt ten stwierdzi algorytm Dijkstry-Scholtena.

---
## Algorytm Misra '83 (systemy asynchroniczne)
<sub>rso_sum_04.pdf, slajdy 36–43</sub>

### Założenia
<sub>slajd 36</sub>
- **brak założeń o topologii** przetwarzania,
- **brak założeń o czasie przesyłania** wiadomości,
- **niezawodna komunikacja**,
- **kanały FIFO**,
- używa znacznika (ang. _token_).

### Zapis algorytmu
<sub>slajdy 37–42</sub>

```
type PACKET extends FRAME is record of
     data : MESSAGE
end record

type TOKEN extends FRAME is record of
     nb : INTEGER
end record

msgIn                 : MESSAGE
pcktOut               : PACKET
tokenOut              : TOKEN
tokenPresent_i        : BOOLEAN := False
succ_i                : PROCESS_ID
C                     : set of CHANNEL_ID
colour_i              : enum {White, Black} := Black
terminationDetected_i : BOOLEAN := False
```

```
 1. when e_start(P_α, TerminationDetection) do
 2.    tokenOut.nb := 0
 3.    send(Q_α, succ_α, tokenOut)
 4. end when

12. when e_receive(Q_j, Q_i, pcktIn: PACKET) do
13.    msgIn := pcktIn.data
14.    colour_i := Black
15.    passive_i := False
16.    deliver(P_j, P_i, msgIn)
17. end when

18. when e_receive(Q_j, Q_{i≠α}, tokenIn: TOKEN) do
19.    tokenPresent_i := True
20.    wait until passive_i = True
21.    if colour_i = Black then
22.       tokenOut.nb := 0
23.    else
24.       tokenOut.nb := tokenOut.nb + 1
25.    end if
26.    send(Q_i, succ_i, tokenOut)
27.    colour_i := White
28.    tokenPresent_i := False
29. end when

30. when e_receive(Q_j, Q_α, tokenIn: TOKEN) do
31.    tokenPresent_i := True
32.    if colour_i = White ∧ tokenOut.nb = |C| then
33.       terminationDetected_i := True
34.       decide(terminationDetected_i)
35.    else
36.       if colour_i = Black then
37.          tokenOut.nb := 0
38.       else
39.          tokenOut.nb := tokenOut.nb + 1
40.       end if
41.       send(Q_i, succ_i, tokenOut)
42.       colour_i := White
43.       tokenPresent_i := False
44.    end if
45. end when
```

> [!note] Numeracja linii
> W oryginale prezentacji numeracja przeskakuje z 4 na 12 (slajdy 39–40) — brakujące linie 5–11 nie występują w materiale.

**Idea licznika $nb$**: znacznik obiega **cykl obejmujący wszystkie kanały komunikacyjne** i zlicza kolejne „białe” przejścia. Zakończenie stwierdzane jest, gdy licznik osiągnie $\lvert\mathcal{C}\rvert$ — czyli znacznik przeszedł wszystkie kanały bez napotkania procesu, który w międzyczasie odebrał wiadomość (i stał się $Black$).

### Cechy algorytmu Misra '83
<sub>slajd 43</sub>
- możliwość rozszerzenia do **dowolnej topologii**,
- **wiele monitorów naraz** może wykrywać zakończenia,
- **cykl obejmujący wszystkie kanały komunikacyjne musi być znany z góry**.

---
## Porównanie algorytmów

| | Dijkstra-Feijen-van Gasteren | Dijkstry-Scholtena | Misra '83 |
|---|---|---|---|
| **Model** | synchroniczny (transmisje natychmiastowe) | przetwarzanie dyfuzyjne | asynchroniczny |
| **Topologia** | **pierścień** | graf (hierarchia / drzewo dyfuzyjne) | dowolna, ale cykl po wszystkich kanałach znany z góry |
| **Mechanizm** | znacznik + kolory White/Black monitorów | sygnały jako „odpowiedzi”, licznik $recvNo$/$sentNo$, rodzic $engager$ | znacznik z licznikiem $nb$ + kolory |
| **Kanały** | — | — | **FIFO**, niezawodne |
| **Kto wykrywa** | inicjator (znacznik biały wraca) | inicjator ($sentNo_\alpha = 0$) | wiele monitorów naraz |

> [!todo] Braki w prezentacjach
> - **Złożoność** (czasowa i komunikacyjna) nie została podana dla **żadnego** z trzech algorytmów detekcji zakończenia. Uzupełnić z podręcznika.
> - Nie omówiono **dowodów poprawności** — twierdzenia (slajd 35) podane są bez dowodu, a dla alg. DFvG i Misra '83 twierdzeń brak w ogóle.
> - Nie omówiono detekcji zakończenia dla modeli żądań innych niż jednostkowy / AND (np. modelu OR) — mimo że modele te wprowadzono w wykładzie 1.

---
## Powiązania
- Procesy aktywne/pasywne, warunek uaktywnienia, modele żądań → [[RSO 07 Model środowiska przetwarzania]]
- Kanały FIFO, konwencja zapisu algorytmów, złożoność → [[RSO 08 Czas wirtualny i złożoność algorytmów]]
- Detekcja zakończenia jako przypadek detekcji stanu globalnego → [[RSO 09 Stan globalny i migawki]]
- Podobna technika kolorowania (White/Red) w migawkach → [[RSO 09 Stan globalny i migawki#Algorytm Lai-Yang (kanały nonFIFO)]]
