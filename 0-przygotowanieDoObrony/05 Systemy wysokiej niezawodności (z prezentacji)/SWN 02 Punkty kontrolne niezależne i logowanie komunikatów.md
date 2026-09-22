---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
source: "Slajdy-FT-03_Recovery2.pdf"
slajdy: "1–45"
zagadnienie: 22
---
# SWN 02. Odtwarzanie stanu cz. II — checkpointing niezależny, logowanie komunikatów, garbage collection
---
> Wykład jest odpowiedzią na cztery wady checkpointingu skoordynowanego ([[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Wady skoordynowanego wyznaczania punktów kontrolnych]]). Procesy wyznaczają punkty kontrolne **niezależnie** — koszt w czasie bezawaryjnym znika, ale linii odtwarzania trzeba teraz **szukać**, a szukanie grozi **efektem domino**. Wykład podaje dwa algorytmy wyszukiwania RL — **Juanga-Venkatesana** (liczniki wiadomości) i **Wanga-Fuchsa** (graf z-zależności) — pomiędzy nimi omawia **logowanie komunikatów** (pesymistyczne / optymistyczne, u nadawcy / u odbiorcy), a kończy **odśmiecaniem** zbędnych punktów kontrolnych.

---
## Niezależne (asynchroniczne) tworzenie punktów kontrolnych
<sub>Slajdy-FT-03_Recovery2.pdf, slajd 1</sub>

**Sposób wyznaczania CP:**
- każdy $C_i$ wyznacza $cp_i$ **niezależnie od innych**…
- … co **nie gwarantuje ustalenia** $CP^\bullet$,
- trzeba wyznaczyć $RL = \langle cp_1, cp_2, \ldots, cp_n \rangle \in GC^\bullet$ (a najlepiej $RL^{*}$).

![[swn-ft03-s01-niezalezne-punkty-kontrolne.png]]
<sub>Niezależnie wyznaczone punkty kontrolne — slajd 1</sub>

### Sposób wyznaczania RL
<sub>slajd 2</sub>

Dwie drogi:
1. trzeba **odszukać** $CP^\bullet$ w zbiorze niezależnie wyznaczonych $cp_i$ — np. **metodą kolejnych wycofań**; **możliwy efekt domino!**
2. albo **wycofać się do pewnego CP niekoniecznie spójnego** i wyznaczyć RL **z precyzją do pojedynczych zdarzeń**, odtwarzając zdarzenia z rejestru utrzymywanego wraz z punktami kontrolnymi.

Druga droga prowadzi wprost do **logowania komunikatów** i do obu algorytmów tego wykładu.

### Punkty odtwarzania i linia odtwarzania
<sub>slajd 3</sub>

**Punkty odtwarzania** to:
- **punkty kontrolne** (*checkpoints*) — *w najprostszym przypadku punkt odtwarzania = punkt kontrolny*,
- **rejestrowanie zdarzeń i odtwarzanie z rejestrów** (*logging and replaying*) — *punkt odtwarzania można osiągnąć wycofując proces do poprzedzającego punktu odtwarzania i przesuwając wykonanie przetwarzania poprzez odtworzenie zdarzeń (niedeterministycznych) zapamiętanych w rejestrze, aż do osiągnięcia odpowiedniego stanu (który stanowić będzie punkt odtwarzania)*.

> **Globalny punkt odtwarzania** = wektor lokalnych punktów odtwarzania.
> **Linia odtwarzania** = **spójny** globalny punkt odtwarzania: $RL = \langle rp_1, rp_2, \ldots, rp_n \rangle$.

> [!important] Uogólnienie względem wykładu I
> W [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Linia odtwarzania]] RL była wektorem **punktów kontrolnych**. Tutaj jest wektorem **punktów odtwarzania** — a te mogą leżeć **pomiędzy** punktami kontrolnymi, osiągane przez odtworzenie zdarzeń z logu. To właśnie daje „precyzję do pojedynczych zdarzeń".

---
## Algorytm Juanga-Venkatesana
<sub>slajdy 4–9</sub>

### Założenia, idea i struktury
<sub>slajd 4</sub>

**Założenia:**
- kanały niezawodne **z uporządkowaniem** komunikatów,
- **nieskończona pojemność** kanałów.

**Idea:**
- wyszukujemy $rp_i$ **z dokładnością do pojedynczych zdarzeń komunikacyjnych**,
- detektując osierocone wiadomości przez **porównywanie ilości odebranych wiadomości z ilością wiadomości wysyłanych**.

**Wykorzystywane struktury:**
- $\text{SENT}_{i \rightarrow j}(rp_i)$ = ilość wiadomości **wysłanych** przez $P_i$ do $P_j$ do punktu $rp_i$,
- $\text{RCVD}_{i \leftarrow j}(rp_i)$ = ilość wiadomości **odebranych** przez $P_i$ od $P_j$ do punktu $rp_i$,
- przy każdym zdarzeniu wysłania lub odbierania wiadomości proces **zapamiętuje tę wiadomość w logu**,
- wartości tych struktur są zapisywane w pamięci trwałej **razem z** $cp_i$.

### Przebieg algorytmu odtwarzania
<sub>slajd 5</sub>

- Kontroler $C_f$ restartowanego po awarii procesu $P_f$ informuje o swym restarcie pozostałe procesy rozsyłając **broadcast** — **złożoność komunikacyjna $O(|E|)$**, gdzie $|E|$ jest całkowitą liczbą łączy komunikacyjnych.
- Wykonanie algorytmu przez $C_i$ rozpoczyna się, gdy $P_i$ jest restartowany **lub** gdy $C_i$ dowiaduje się o restarcie innego $P_f$. Zatem algorytm jest **inicjowany jednocześnie przez wszystkie kontrolery**. Inicjacja polega na wyznaczeniu początkowej wartości $rp_i^1$.
- Algorytm wykonuje się w **$N$ iteracjach („rundach")**, gdzie $N$ jest liczbą procesów. W $k$-tej iteracji ($k>1$) kontroler $C_i$, wykorzystując $rp_i^{k-1}$ wyznaczony w iteracji $k-1$, oblicza wartości $\text{SENT}_{i \rightarrow j}(rp_i^{k-1})$ dla każdego sąsiedniego procesu $P_j$, wysyłając ją następnie w komunikacie $ROLLBACK$. Przetwarzając komunikaty $ROLLBACK$ odebrane od sąsiadów, odnajduje taki $rp_i^k$, aby dla każdego sąsiedniego $P_j$:
$$\text{RCVD}_{i \leftarrow j}(rp_i^{k}) \leqslant \text{SENT}_{j \rightarrow i}(rp_i^{k-1})$$
- Po zakończeniu każdej rundy **co najmniej jeden proces wycofa się** do swojego ostatecznego lokalnego punktu odtwarzania — za wyjątkiem przypadków, gdy aktualnie wyznaczone punkty odtwarzania już stanowią spójną linię odtwarzania.

> [!note] Przypis ze slajdu 5
> *precyzyjniej:* $\text{RCVD}_{i \leftarrow j}(rp_i^{k}) = \text{SENT}_{j \rightarrow i}(rp_i^{k-1})$, *ew. za wyjątkiem pierwszej iteracji w procesie restartowanym* — tam obowiązuje nierówność.

### Przykład przebiegu
<sub>slajdy 6–9</sub>

Trzy procesy, awaria $P_2$; $cp_1^1, cp_1^2, cp_1^3$, $cp_2^1, cp_2^2$, $cp_3^1, cp_3^2$; zdarzenia $e_i^{k}$.

1. Proces $P_2$ po awarii jest restartowany z punktu $cp_2^2$, który odpowiada stanowi po zdarzeniu $e_2^4$. Wysłany broadcast inicjalizuje algorytm również na $P_1$ i $P_3$. Mamy $rp_1 = e_1^6$, $rp_2 = e_2^4$ oraz $rp_3 = e_3^3$ ($e_1^6$ i $e_3^3$ są **wirtualnymi punktami odtwarzania**).
2. $C_2$ wysyła $ROLLBACK(C_2, C_1, 2)$ do $C_1$ i $ROLLBACK(C_2, C_3, 1)$ do $C_3$.
3. $C_1$ wysyła $ROLLBACK(C_1, C_2, 2)$ do $C_2$ i $ROLLBACK(C_1, C_3, 0)$ do $C_3$.
4. $C_3$ wysyła $ROLLBACK(C_3, C_1, 0)$ do $C_1$ i $ROLLBACK(C_3, C_2, 1)$ do $C_2$.
5. Ponieważ $\text{RCVD}_{1 \leftarrow 2}(e_1^6) = 4 > 2$, $C_1$ wyznacza nowy $rp_1 = e_1^4$. Teraz $\text{RCVD}_{1 \leftarrow 2}(e_1^4) = 2$.
6. Ponieważ $\text{RCVD}_{3 \leftarrow 2}(e_3^3) = 2 > 1$, $C_3$ wyznacza nowy $rp_3 = e_3^2$. Teraz $\text{RCVD}_{3 \leftarrow 2}(e_3^2) = 1$.
7. Ponieważ $\text{RCVD}_{2 \leftarrow 3}(e_2^4) = 1 = \text{SENT}_{3 \rightarrow 2}(rp_3)$ oraz $\text{RCVD}_{2 \leftarrow 1}(e_2^4) = 1 < 2$, $P_2$ **nie potrzebuje wycofywać się głębiej**. W drugiej iteracji $C_2$ wysyła ponownie te same komunikaty.
8. $C_1$ wysyła $ROLLBACK(C_1, C_2, 2)$ do $C_2$ i $ROLLBACK(C_1, C_3, 0)$ do $C_3$.
9. $C_3$ wysyła $ROLLBACK(C_3, C_1, 0)$ do $C_1$ i $ROLLBACK(C_3, C_2, 1)$ do $C_2$.
10. **Trzecia runda przebiega identycznie.** Zatem ostateczna linia odtwarzania $(e_1^4, e_2^4, e_3^2)$ została wyznaczona **w pierwszej rundzie** i po niej nic się już nie zmienia.
11. Wiadomość $m$ zostanie **odtworzona z logu** procesu $P_1$.

> [!note] Uzupełnienie spoza slajdów
> [[Systemy Wysokiej Niezawodności/Algorytm Juanga-Venkatesana]] dodaje dwa założenia, których na slajdzie 4 nie ma: przetwarzanie jest **fragmentarycznie deterministyczne** (*piecewise deterministic*), a logowanie jest **optymistyczne, po stronie nadawcy i odbiorcy** — przy czym sama notatka opatruje tę drugą tezę zastrzeżeniem „prawdopodobnie". Slajdy mówią tylko, że „przy każdym zdarzeniu wysłania lub odbierania proces zapamiętuje wiadomość w logu", nie rozstrzygając trybu.

---
## Logowanie komunikatów (*message logging*)
<sub>slajdy 10–15</sub>

### Po co rejestrować wiadomości
<sub>slajd 10</sub>

Rejestrowanie wiadomości:
- pozwala **ograniczyć rollback**,
- i **uniknąć efektu domino**,
- po wycofaniu do $cp_i$ odtwarzając zarejestrowane wcześniej wiadomości,
- aż do osiągnięcia stanu $rp_i$, **w którym żadna osierocona wiadomość nie została odebrana**.

### Rejestrowanie po stronie odbiorcy
<sub>slajd 11</sub>

![[swn-ft03-s11-logowanie-u-odbiorcy.png]]
<sub>Rejestrowanie po stronie odbiorcy — slajd 11</sub>

- *receive* jest operacją **niedeterministyczną**,
- przy odtwarzaniu czekamy na wiadomość właśnie po stronie odbiorcy — odtworzenie wiadomości z logu (*replay*) **przyspiesza ponowne wykonanie przetwarzania** (*re-execution*),
- retransmisja zagubionej $m_2$ **może nie być konieczna**.

### Rejestrowanie po stronie nadawcy
<sub>slajd 12</sub>

![[swn-ft03-s12-logowanie-u-nadawcy.png]]
<sub>Rejestrowanie po stronie nadawcy, w tym przypadek rozgłaszania — slajd 12</sub>

- usprawnia **retransmisję** (skądinąd potrzebną),
- umożliwia przetwarzanie **niedeterministyczne**,
- **broadcast** — jeden zapis do logu obsługuje wszystkie kopie rozgłaszanej wiadomości.

### Rejestrowanie po obu stronach
<sub>slajd 13</sub>

Wiadomość trafia do logu i nadawcy, i odbiorcy. Slajd stawia jedno pytanie: **nadmiar?**

### Rejestrowanie pesymistyczne
<sub>slajd 14</sub>

![[swn-ft03-s14-logowanie-pesymistyczne.png]]
<sub>Rejestrowanie pesymistyczne — slajd 14</sub>

- zapis logu do pamięci trwałej **atomowo z operacją** *send* / *receive*,
- atomowo w praktyce $\rightarrow$ $C_i$: *log* **po** *receive*, ale **przed** *deliver*,
- **duży narzut czasowy** (blokowanie przetwarzania aplikacyjnego).

### Rejestrowanie optymistyczne
<sub>slajd 15</sub>

![[swn-ft03-s15-logowanie-optymistyczne.png]]
<sub>Rejestrowanie optymistyczne — slajd 15</sub>

- wraz z operacją *send* / *receive* zapis do rejestru **w pamięci ulotnej**,
- rejestr przepisany do pamięci trwałej **przy późniejszej okazji**,
- w przypadku awarii, wraz z ulotnym rejestrem, **utracone zostają niektóre wiadomości**.

> [!important] Kompromis
> Pesymistyczne = brak osieroconych wiadomości kosztem narzutu w czasie bezawaryjnym. Optymistyczne = brak narzutu kosztem możliwych **krawędzi rollback** w grafie punktów kontrolnych (slajd 27) — to dokładnie ten mechanizm, który obsługuje algorytm Wanga-Fuchsa.

---
## Zależność punktów kontrolnych — relacja z-dependency

### Interwał punktu kontrolnego
<sub>slajd 16</sub>

![[swn-ft03-s16-interwal-punktu-kontrolnego.png]]
<sub>Interwał punktu kontrolnego $I_i^{k}$ — slajd 16</sub>

$I_i^{k}$ to **interwał punktu kontrolnego**: odcinek wykonania procesu $P_i$ **zakończony** punktem $cp_i^{k}$.

### Relacja z-dependency (zigzag-dependency)
<sub>slajd 17</sub>

> $cp_j^{\,l}$ jest **bezpośrednio z-zależny** od $cp_i^{\,k}$ $\iff$
> $$i = j \;\wedge\; l = k+1$$
> **lub**
> $$i \neq j \;\wedge\; \exists_m : \big( send(P_i, \{P_j\}, m) \in I_i^{k} \;\wedge\; recv(P_j, P_i, m) \in I_j^{\,l-1} \big)$$

![[swn-ft03-s17-relacja-z-dependency.png]]
<sub>Bezpośrednia z-zależność $cp_i^{k} \rightarrow cp_j^{l}$ — slajd 17</sub>

> **Z-dependency** punktów kontrolnych to **domknięcie przechodnie** relacji **bezpośredniej** z-zależności.

### Twierdzenie 1
<sub>slajd 18</sub>

> **Theorem 1.** Z-zależność pomiędzy dwoma punktami kontrolnymi czyni z nich **parę niespójną** (*inconsistent checkpoint pair*).

### Graf punktów kontrolnych
<sub>slajdy 19–20</sub>

> **Checkpoint (z-dependency) graph**: **węzły** reprezentują punkty kontrolne (interwały punktów kontrolnych), **krawędzie** reprezentują relację **bezpośredniej** z-zależności.

![[swn-ft03-s19-graf-punktow-kontrolnych.png]]
<sub>Diagram przestrzenno-czasowy czterech procesów z wirtualnymi punktami kontrolnymi — slajd 19</sub>

![[swn-ft03-s20-rozszerzony-graf-punktow.png]]
<sub>**Rozszerzony** graf punktów kontrolnych (*extended checkpoint graph*) — zawiera wirtualne punkty kontrolne, slajd 20</sub>

---
## Algorytm Wanga-Fuchsa
<sub>slajdy 21–37</sub>

### Model systemu i idea
<sub>slajd 21</sub>

**Model systemu:**
- model awarii **fail-recovery**,
- kanały niezawodne **bez zachowania kolejności** (*reliable reordering channels*),
- **brak partycjonowania sieci**,
- przetwarzanie **deterministyczne** (*can be relaxed* — dopisek ze slajdu),
- wszystkie działania mogą zostać wycofane (*output commit*, jeśli konieczne).

**Ogólna idea:**
- wykorzystanie **grafu z-zależności punktów kontrolnych**,
- **z-dependency tracking** — wszystkie wiadomości tagowane dodatkową informacją kontrolną,
- **optymistyczne logowanie wiadomości** (odbiorca loguje treść wiadomości),
- część danych kontrolnych zapisywana w pamięci trwałej razem z punktami kontrolnymi i logami wiadomości.

### Struktury
<sub>slajd 22</sub>

- $ckpt\_num_i$ — numer interwału punktu kontrolnego procesu $P_i$,
- $send\_num_i$ — numer sekwencyjny wiadomości aplikacyjnych wysłanych przez $P_i$,
- $send\_log_i$ — zawiera elementy $(send\_num_i, j)$, gdzie $P_j$ jest odbiorcą wiadomości $m$ wysłanej przez $P_i$ z numerem $send\_num_i$ (również z $ckpt\_num_i$),
- $rec\_log_i$ — zawiera elementy $(m, send\_num_j, ckpt\_num_j, j)$ opisujące **odebraną** wiadomość $m$.

### Tworzenie punktów kontrolnych i odtwarzanie
<sub>slajd 23</sub>

**Checkpointing:** gdy kontroler $C_i$ decyduje **spontanicznie i niezależnie** od innych kontrolerów o utworzeniu kolejnego punktu kontrolnego $P_i$, zapisuje aktualny stan procesu **wraz z** $send\_log_i$, $rec\_log_i$ i $ckpt\_num_i$, a następnie **czyści ulotne logi** $send\_log_i$ i $rec\_log_i$.

**Recovery:** przy odtwarzaniu procesu $P_f$ musi zostać zbudowany **globalny graf punktów kontrolnych** całego przetwarzania. $C_f$ żąda od pozostałych kontrolerów odesłania informacji o z-zależności. Kontroler $C_i$, który otrzymał żądanie, **tworzy wirtualny punkt kontrolny** i odpowiada aktualnymi wartościami $send\_log_i$, $rec\_log_i$, $ckpt\_num_i$. Gdy $C_f$ otrzyma wszystkie odpowiedzi, konstruuje **rozszerzony graf punktów kontrolnych**.

### Przykład — pary niespójne
<sub>slajdy 24–26</sub>

![[swn-ft03-s24-przyklad-pary-niespojne.png]]
<sub>Diagram przestrzenno-czasowy i odpowiadający mu rozszerzony graf punktów kontrolnych — slajd 24</sub>

- para $(cp_3^1, cp_2^2)$ **nie jest spójna** z powodu wiadomości osieroconej $m_3$ — istnienie wiadomości osieroconej jest przedstawione w grafie punktów kontrolnych przez **krawędzie łączące punkty kontrolne różnych procesów**,
- para $(cp_2^1, cp_1^3)$ jest **również niespójna**,
- pary $(cp_2^1, cp_1^2)$ i $(cp_3^1, cp_2^1)$ są **spójne**.

Slajd 25 pyta: *a co z parą* $(cp_3^1, cp_1^2)$? Slajd 26 odpowiada:
- w wyniku awarii $P_3$ **log odbioru** $m_1$ w $P_3$ **został utracony** (logowanie optymistyczne!),
- restart $P_1$ z $cp_1^2$ **nie odtworzy** $m_1$ (ani żadnej zastępczej $m'_1$).

### Krawędzie rollback
<sub>slajdy 27–29</sub>

![[swn-ft03-s27-krawedzie-rollback.png]]
<sub>Krawędź rollback (czerwona linia przerywana) $cp_3^1 \rightarrow cp_1^2$ — slajd 27</sub>

> **Rollback edges** reprezentują wiadomości **jeszcze nie zapisane do logu** (*not-yet-logged messages*).

Wobec tego $P_1$ musi zostać wycofany do $cp_1^1$.

Slajd 28 pyta: *a co z* $m_0$ *i krawędzią rollback* $cp_1^1 \rightarrow cp_2^1$? Slajd 29 odpowiada: ponieważ $m_0$ **jest dostępna** w logu wiadomości $rec\_log_1$ przy $cp_1^2$ i może zostać odtworzona, **krawędzi rollback** $cp_1^1 \rightarrow cp_2^1$ **nie ma**.

### Zredukowany graf punktów kontrolnych
<sub>slajd 30</sub>

- węzeł z **wchodzącą** krawędzią rollback reprezentuje **potencjalną niespójność** punktów kontrolnych (dopóki wiadomość nie zostanie zalogowana),
- dlatego **wykluczamy** każdy węzeł z wchodzącą krawędzią rollback **oraz wszystkie węzły osiągalne** z takich węzłów.

![[swn-ft03-s30-zredukowany-graf.png]]
<sub>Redukcja grafu: wykluczenie $cp_1^2$, $cp_1^3$, $vcp_1^4$ — slajd 30</sub>

> **Uwaga ze slajdu:** *remember: the rollback edge is specific to the Wang-Fuchs algorithm!*

### Algorytm wyszukiwania linii odtwarzania
<sub>slajd 31</sub>

1. Umieść **ostatni punkt kontrolny każdego procesu** w zbiorze **root set**.
2. **Oznacz** wszystkie punkty kontrolne osiągalne z dowolnego punktu kontrolnego w root set.
3. **Dopóki** co najmniej jeden punkt kontrolny w root set jest oznaczony:
   - zastąp każdy oznaczony punkt kontrolny w root set **ostatnim nieoznaczonym** punktem kontrolnym tego samego procesu,
   - oznacz wszystkie punkty kontrolne osiągalne z dowolnego punktu kontrolnego w root set.
4. Końcowy root set **jest linią odtwarzania RL**.

![[swn-ft03-s31-wyszukiwanie-linii-odtwarzania.png]]
<sub>Początkowy i końcowy root set — slajd 31</sub>

### Odtwarzanie
<sub>slajd 32</sub>

- podczas ponownego wykonania (*re-execution*) przychodzące wiadomości mogą być odtwarzane z $rec\_log_i$, **dopóki log nie zostanie wyczerpany**,
- wychodzące wiadomości są **wysyłane ponownie** (*yet beware of duplicates!*).

### Redukcja narzutu logowania — wiadomości *non-state*
<sub>slajdy 33–34</sub>

![[swn-ft03-s34-wiadomosc-non-state.png]]
<sub>Wiadomość $m_1$ typu *non-state* — slajd 34</sub>

**Stan kanału** (slajd 33): względem pary $(cp_1^1, cp_2^1)$ wiadomość $m_1$ jest *in-channel* (*in-transit*, *in-state message*) — należy do stanu kanału odpowiadającego tej parze punktów kontrolnych i **zostanie utracona, jeśli nie zostanie zalogowana**.

> **Obserwacja** (slajd 34): wiadomość, która **nigdy nie stanie się częścią żadnego stanu kanału** (*non-state message*), **nie musi być logowana**.

Uzasadnienie na przykładzie: względem $(cp_1^1, cp_2^2)$ wiadomość $m_2$ jest osierocona, więc para ta jest niespójna; $m_1$ jest *non-state* — jej obecność w logach nie wpływa na spójność; skoro niezależnie od zalogowania $m_1$ para punktów kontrolnych **i tak nie jest spójna**, nie ma powodu logować $m_1$ w ogóle.

### Twierdzenie 2 i jego dowód
<sub>slajdy 35–36</sub>

> **Theorem 2.** Jeżeli w grafie punktów kontrolnych istnieje **ścieżka** z $cp_i^{k}$ do $cp_j^{\,l}$, to wszystkie wiadomości wysłane z $I_j^{\,l-1}$ i odebrane w $I_i^{k}$ są wiadomościami **non-state**.

**Dowód, część 1** (slajd 35): weź dowolne $m$ należące do takiego stanu kanału — ten stan kanału nie należy do żadnej RL. Przypuśćmy, że stan kanału odpowiadający linii odtwarzania RL zawiera taką wiadomość $m$. Z definicji RL musi zawierać $cp_i^{x \leqslant k}$ oraz $cp_j^{\,y \geqslant l}$, a zatem istnieje ścieżka z $cp_i^{x}$ do $cp_i^{k}$ oraz z $cp_j^{\,l}$ do $cp_j^{\,y}$. Jeśli istnieje również ścieżka z $cp_i^{k}$ do $cp_j^{\,l}$, to mamy ścieżkę z $cp_i^{x}$ do $cp_j^{\,y}$, co oznacza, że **te dwa punkty kontrolne są niespójne** — sprzeczność z faktem, że leżą na tej samej linii odtwarzania RL.

**Dowód, część 2** (slajd 36): weź dowolną RL — żadne takie $m$ nie może należeć do stanu kanału w tej RL. Ścieżka z $cp_i^{k}$ do $cp_j^{\,l}$ może zostać przerwana **tylko wtedy**, gdy dla pewnego $cp_q^{w}$ na tej ścieżce proces $P_q$ wycofa się do $cp_q^{z}$, $z < w$. Wówczas $P_j$ musi wycofać się do $cp_j^{\,h<l}$ z powodu ścieżki z-zależności. Wszystkie wiadomości wysłane z $I_j^{\,l-1}$ są wtedy **unieważnione** i nigdy nie staną się częścią żadnego stanu kanału. $\square$

### Przykład zastosowania
<sub>slajd 37</sub>

- $m_1$ jest *non-state* — patrz para $(cp_1^2, cp_2^1)$,
- $m_2$ jest *non-state* — patrz para $(cp_1^1, cp_2^3)$,
- a co z $m_3$? — para $(cp_2^2, cp_3^1)$ faktycznie **nie utworzy RL** (ani z $cp_1^1$, ani z $cp_1^2$), **ale $P_3$ nie wie o tym w momencie odbioru $m_3$**; z definicji $m_3$ **nie jest** *non-state*, bo para $(cp_2^2, cp_3^1)$ jest **spójna**.

---
## Garbage collection
<sub>slajdy 38–44</sub>

### Cel i podstawowe pojęcia
<sub>slajd 38</sub>

**Cel:** odzyskać punkty kontrolne, które **nie są już użyteczne** (*discardable*).

> **Def.** *Discardable* punkt kontrolny to punkt kontrolny, który **nigdy nie będzie należał do żadnej przyszłej linii odtwarzania**.

**Obsolete checkpoints:**
- poprzedzające **najgorszą możliwą** linię odtwarzania (*worst-case recovery line*),
- punkty *obsolete* są oczywiście *discardable*,
- ale **niektóre punkty non-obsolete również mogą być** *discardable*.

### Punkty przeterminowane
<sub>slajdy 39–40</sub>

Slajd 39 pyta: *gdzie znajduje się linia odtwarzania wyznaczona przez Wanga-Fuchsa w najgorszym przypadku?*

![[swn-ft03-s40-punkty-przeterminowane.png]]
<sub>Punkty *obsolete* i najgorsza możliwa linia odtwarzania — slajd 40</sub>

Slajd 40 dodaje dwa pytania: używamy tu grafu punktów kontrolnych, **ale nie rozszerzonego** — dlaczego nie?

### Punkty non-obsolete, ale usuwalne
<sub>slajdy 41–42</sub>

Slajd 41 pyta: *czy wszystkie punkty non-obsolete są potencjalnie użyteczne dla przyszłego odtwarzania?*

![[swn-ft03-s42-calkowity-efekt-domino.png]]
<sub>Całkowity efekt domino — tylko początkowe punkty kontrolne tworzą RL, slajd 42</sub>

- **całkowity efekt domino** — tylko **początkowe** punkty kontrolne tworzą RL,
- z-zależność z $cp_1^2$ do $cp_2^2$ oraz ta z $cp_2^1$ do $cp_1^2$ implikują, że $cp_1^2$ jest **niespójny z każdym** punktem kontrolnym $P_2$, więc nigdy nie będzie należał do żadnej linii odtwarzania i tym samym **jest** *discardable*,
- podobnie $cp_1^1$ i $cp_2^1$ są *discardable*,
- w sytuacji efektu domino **duża liczba punktów non-obsolete** jest utrzymywana w pamięci trwałej, co daje **duży narzut pamięciowy**.

### Twierdzenie 3
<sub>slajdy 43–44</sub>

> **Theorem 3.** Niech $N$ będzie liczbą procesów, a $\hat{G}$ **nadgrafem** grafu punktów kontrolnych $G$. Punkt kontrolny w $G$ jest **non-discardable** wtedy i tylko wtedy, gdy należy do **sumy linii odtwarzania** wszystkich $\hat{G}\text{-}vcp_i$ ($1 \leqslant i \leqslant N$).

> **Nadgraf $\hat{G}$** = graf $G$ sztucznie rozszerzony o **wszystkie** wirtualne punkty kontrolne (definicja z dymka na slajdzie 43).

![[swn-ft03-s44-punkty-usuwalne.png]]
<sub>Punkty usuwalne: $cp_1^0, cp_2^0, cp_3^0, cp_4^0$ (obsolete) oraz $cp_2^3, cp_3^3, cp_4^2, cp_4^3$ (non-obsolete) — slajd 44</sub>

> [!note] Uzupełnienie spoza slajdów
> [[Systemy Wysokiej Niezawodności/Algorytm Wanga-Fuchsa]] podaje intuicję twierdzenia 3, której slajd nie formułuje: punkt kontrolny jest *non-discardable*, jeśli przynależy **do którejkolwiek z $N$ linii odtwarzania** powstałych na skutek awarii **któregokolwiek** z procesów wchodzących w skład grafu $G$.

---
## Bibliografia
<sub>slajd 45</sub>

1. T. Juang, S. Venkatesan, *Crash Recovery with Little Overhead*, 11th International Conference on Distributed Computing Systems, 1991, pp. 454–461.
2. Y.-M. Wang, W. K. Fuchs, *Optimistic Message Logging for Independent Checkpointing in Message-Passing Systems*, Symposium on Reliable Distributed Systems — SRDS'92, pp. 147–154, 1992.
3. M. Singhal, N.G. Shivaratri, *Advanced Concepts in Operating Systems*, McGraw-Hill, 1994 (2001), ch. 12.
4. R. Chow, T. Johnson, *Distributed Operating Systems & Algorithms*, Addison Wesley Longman, 1997, ch. 13.

---
## Braki i uwagi

> [!todo] Czego nie ma na slajdach tego wykładu
> - **Złożoności** algorytmu Wanga-Fuchsa (dla Juanga-Venkatesana podana jest tylko złożoność komunikacyjna broadcastu $O(|E|)$ i liczba rund $N$).
> - **Odpowiedzi** na pytania retoryczne: „nadmiar?" (slajd 13), „gdzie jest RL w najgorszym przypadku?" (slajd 39), „dlaczego nie rozszerzony graf?" (slajd 40), „czy wszystkie non-obsolete są użyteczne?" (slajd 41).
> - **Porównania tabelarycznego** checkpointingu skoordynowanego i niezależnego — wady skoordynowanego są na slajdzie 35 wykładu I, zalety/wady niezależnego trzeba złożyć samodzielnie.
> - **Logowania przyczynowego** (*causal message logging*) — wykład zna tylko pesymistyczne i optymistyczne; trzeci tryb z systematyki Elnozahy'ego się nie pojawia.
