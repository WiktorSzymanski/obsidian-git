---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
source: "Slajdy-FT-07_Voting.pdf"
slajdy: "1–29"
zagadnienie: 24
---
# SWN 10. Algorytmy głosowania — statyczne (Gifford) i dynamiczne (Jajodia-Mutchler)
---
> **Głosowanie jest mechanizmem pokonującym ograniczenia atomowego zatwierdzania** ([[SWN 09 Transakcje i atomowe zatwierdzanie]]): zamiast wymagać zgody **wszystkich** replik, wystarczy zebrać **kworum**. Wykład podaje **algorytm Gifforda** (głosowanie statyczne: $V_i$, $R$, $W$ stałe, warunki $W \geqslant M$ i $R + W > V$), pokazuje jego słabość wobec **partycjonowania sieci**, a następnie **głosowanie dynamiczne** — protokół **Jajodii-Mutchlera**, który adaptuje zbiór głosujących do stanu systemu po awarii, oraz techniki **dynamicznej realokacji głosów**.

---
## Głosowanie — definicja problemu
<sub>Slajdy-FT-07_Voting.pdf, slajd 1</sub>

> **Głosowanie** — mechanizm pokonujący ograniczenia atomowego zatwierdzania.

**Definicja problemu:** należy zagwarantować **atomową spójność** operacji `read` i `write` na replikach obiektu $x$ (repliki w procesach $P_1, \ldots, P_i, \ldots, P_N$).

> **Reguła podstawowa:** zanim proces $P_i$ uzyska dostęp do obiektu $x$, musi zdobyć odpowiednią liczbę głosów od pozostałych procesów (**quorum**).

**Model systemu:** każdy proces $P_i$ (tj. każda replika) posiada pewną liczbę głosów $V_i$.

---
## Statyczne głosowanie

### Model systemu
<sub>slajd 2</sub>

- każda operacja na replice wymaga uzyskania **blokady**: **wyłącznej** (*exclusive-lock*) albo **współdzielonej** (*shared-lock*),
- blokady zakłada **lokalny zarządca** (*lock manager*),
- każda replika posiada **monotoniczny numer wersji** $VN_i$ równy ilości dokonanych modyfikacji,
- model awarii **fail-recovery** (procesy **i** kanały).

### Ogólna koncepcja
<sub>slajd 3</sub>

- przed wykonaniem operacji zbieramy głosy od innych procesów, aż uzbieramy odpowiednie **quorum**,
- operacja `read` wymaga **read-quorum $R$**,
- operacja `write` wymaga **write-quorum $W$**.

**Problemy:** kiedy zakończyć zbieranie głosów? którą kopię odczytać? które kopie uaktualniać?

### Algorytm Gifforda
<sub>slajdy 4–5</sub>

1. $P_i$ składa $Lock\_Request$ do lokalnego zarządcy.
2. Gdy blokada została założona, $P_i$ wysyła komunikat $Vote\_Request$ do wszystkich procesów.
3. Gdy $P_j$ odbiera $Vote\_Request$, składa $Lock\_Request$ do lokalnego zarządcy. Jeśli blokada została założona, **odsyła do $P_i$ wersję swojej repliki $VN_j$ oraz liczbę głosów $V_j$**.
4. $P_i$ decyduje o wykonaniu operacji w zależności od liczby uzbieranych głosów (*timeout*):

$$V_{read} = \sum_{k \in \mathbb{O}} V_k, \quad \text{gdzie } \mathbb{O} \text{ oznacza zbiór numerów procesów, które przysłały głosy}$$
$$V_{read} \geqslant R \;\Rightarrow\; P_i \text{ zebrał quorum dla odczytu}$$

$$V_{write} = \sum_{k \in \mathbb{Q}} V_k, \quad \text{gdzie } \mathbb{Q} = \{k \in \mathbb{O} : VN_k = VN_{max}\} \;\text{ i }\; VN_{max} = \max\{VN_j : j \in \mathbb{O}\}$$
$$V_{write} \geqslant W \;\Rightarrow\; P_i \text{ zebrał quorum dla zapisu}$$

5. Jeśli $P_i$ **nie zebrał** quorum, wysyła $Release\_Lock$ do lokalnego zarządcy oraz do wszystkich $P_k : k \in \mathbb{O}$.
6. Jeśli $P_i$ **zebrał** quorum, sprawdza, czy jego replika jest aktualna ($VN_i = VN_{max}$). Jeśli $VN_i < VN_{max}$, **aktualna kopia jest sprowadzana** od dowolnego $P_k : k \in \mathbb{Q}$.
7. Jeśli $P_i$ ma wykonać operację `read`, **czyta lokalną kopię**. Jeśli `write` — **modyfikuje lokalną kopię oraz $VN_i$** i wysyła uaktualnienie do wszystkich $P_k : k \in \mathbb{Q}$ (**uaktualnienie wyłącznie świeżych replik**). Wówczas $P_i$ wysyła $Release\_Lock$ do lokalnego zarządcy oraz do wszystkich $P_k : k \in \mathbb{O}$.
8. Każdy $P_j$, otrzymując uaktualnienie, modyfikuje lokalną replikę, a otrzymując $Release\_Lock$ — zwalnia blokadę.

**Problemy:** *Jak zagwarantować poprawność $R$ i $W$? Jak rozdzielić głosy pomiędzy procesy?*

> [!important] Kluczowy szczegół
> $V_{write}$ sumuje głosy **tylko tych replik, które są aktualne** ($VN_k = VN_{max}$) — dlatego zbiór $\mathbb{Q} \subseteq \mathbb{O}$. Zapis idzie tylko do świeżych replik; starsze nadrabiają zaległości później (slajd 10).

### Poprawność R i W
<sub>slajd 6</sub>

$$V = \sum_i V_i \qquad \text{majority } M = \left\lceil \frac{V+1}{2} \right\rceil$$
$$\boxed{W \geqslant M} \qquad \boxed{R + W > V}$$

**To gwarantuje, że:**
- zawsze istnieje **podzbiór aktualnych replik** dysponujących sumą $W$ głosów,
- $W$ jest **odpowiednio duże**, by wykluczyć współbieżne modyfikacje w **rozłącznych podzbiorach** replik,
- **część wspólna** *read-quorum* i *write-quorum* **nie jest pusta**, dzięki czemu w każdym *read-quorum* będzie **co najmniej jedna aktualna replika**.

### Rozdział głosów
<sub>slajdy 7–9</sub>

Przykład: 4 procesy, $V = \sum_i V_i = 5$, z atrybutami *speed* i *reliability*.

![[swn-ft07-s07-rozdzial-glosow-wariant-a.png]]
<sub>Wariant (a): $V_1=1, V_2=1, V_3=2, V_4=1$; $M=3$, **$W=4$, $R=2$** — szybkie odczyty ($R$ małe) patrz $P_3$; **awaria procesu $P_3$ uniemożliwia zebranie write-quorum**, slajd 7</sub>

**Wariant (b)** (slajd 8): te same $V_i$, ale $M=3$, **$W=3$, $R=3$** — zebranie *read-quorum* lub *write-quorum* wymaga **3 dostępnych jednocześnie procesów** lub $P_3$ i przynajmniej jednego innego procesu.

**Akcent na niezawodność** (slajd 9): głosy przesunięte na najbardziej niezawodny proces $P_4$ — $V_3 = 1$, $\mathbf{V_4 = 2}$; nadal $M=3$, $W=3$, $R=3$.

### Uwagi i dlaczego „statyczne"
<sub>slajd 10</sub>

- **nie trzeba zliczać wyłącznie głosów z $VN_{max}$**, aby uzyskać *write-quorum*,
- **starsze repliki mogą być uaktualniane przy okazji** wykonywania operacji `write`.

> **Dlaczego głosowanie nazywa się statyczne?** — $V_i$, $R$ oraz $W$ są **stałe i niezależne od bieżącego stanu systemu**.

**Problem:** co jeśli $P_3$ **i** $P_4$ jednocześnie ulegną awarii? lub oddzielą się od pozostałych (**rozłączenie sieci komunikacyjnej**)?

### Partycjonowanie
<sub>slajd 11</sub>

![[swn-ft07-s11-partycjonowanie.png]]
<sub>Partycjonowanie przy $M=3$, $W=3$, $R=3$ — slajd 11</sub>

- podział $\{P_1 P_2 P_3\} \mid \{P_4\}$ → lewa strona jest **partycją większościową** (*majority partition*) i może działać,
- podział $\{P_1 P_2\} \mid \{P_3\} \mid \{P_4\}$ → **brak quorum**,
- awaria $P_3$ i $P_4$ przy $\{P_1 P_2\}$ → **tu też brak quorum**.

### CAP
<sub>slajd 12</sub>

![[swn-ft07-s12-cap-szescian.png]]
<sub>**CAP Impossibility Result** — sześcian o osiach Consistency, Availability i Partition tolerance; powierzchnia odcinająca pokazuje, że nie da się osiągnąć 100% we wszystkich trzech wymiarach naraz, slajd 12</sub>

> Ten sam wynik w postaci diagramu Venna (z umieszczeniem 2PC, Paxosa i Gossipu) pojawia się w [[SWN 07 Konsensus#Konsensus a CAP]].

### Graf partycjonowania
<sub>slajd 13</sub>

> **Partitioning graph** reprezentuje **historię podziału sieci**: **wierzchołki = partycje**, **krawędzie = podział lub scalenie**.

![[swn-ft07-s13-partitioning-graph.png]]
<sub>Graf partycjonowania: $\{P_1P_2P_3P_4P_5\}$ dzieli się na $\{P_1P_2P_3\}$ i $\{P_4P_5\}$, potem $\{P_1P_2P_3\}$ na $\{P_1\}$ i $\{P_2P_3\}$, a następnie $\{P_2P_3\}$ scala się z $\{P_4P_5\}$ w $\{P_2P_3P_4P_5\}$ — slajd 13</sub>

---
## Dynamiczne głosowanie

### Idea i warianty
<sub>slajd 14</sub>

> **Idea:** adaptować **liczbę głosów** lub **zbiór głosujących procesów** do stanu systemu po awarii, tak aby móc zebrać quorum.

**Warianty:**
- **majority based voting** — zmienny **zbiór procesów** stanowiących większość niezbędną do modyfikacji replik,
- **dynamic vote reassignment** — zmienna **liczba głosów** przypisanych do poszczególnych replik.

### Protokół Jajodii-Mutchlera
<sub>slajd 15</sub>

**Majority-based dynamic voting protocol:**
- zbiór węzłów tworzących większość jest **dynamicznie zmieniany**, aby zwiększyć dostępność w razie awarii,
- są to **te węzły, które zostały uaktualnione podczas najświeższej modyfikacji**.

**Voting protocol (Jajodia-Mutchler [1]):**
- wybiera **jedną z partycji**, w której operacje `read` i `write` mogą być kontynuowane — tę, która **mogłaby stanowić większość w konfiguracji ostatniej modyfikacji**,
- o ile tylko można wyróżnić **partycję pierwotną** (*primary partition*).

### Założenia i struktury
<sub>slajd 16</sub>

**Założenia:** dla uproszczenia każdy proces trzyma **jedną replikę z jednym głosem**; wszystkie procesy są **liniowo ponumerowane** $P_1, P_2, \ldots$

**Dane protokołu:**
- **$VN_i$** (*Version Number*) — liczy liczbę udanych modyfikacji repliki w $P_i$,
- **$RU_i$** (*Replicas Updated*) — liczba replik uaktualnionych w **najświeższej** modyfikacji,
- **$DS_i$** (*Distinguished Sites*) — lista zależna od $RU_i$: **gdy $RU_i$ jest parzyste**, $DS_i$ wskazuje replikę **większą (w porządku liniowym) niż wszystkie pozostałe repliki** uczestniczące w najświeższej modyfikacji repliki w $P_i$. **Gdy $RU_i$ jest nieparzyste, $DS_i = \varnothing$**.

> **Przypis ze slajdu:** *w istocie z wyjątkiem $RU_i = 3$, gdzie oryginalny protokół przełącza się na głosowanie statyczne, a $DS_i$ wymienia trzy repliki uczestniczące w najświeższej modyfikacji — Jajodia i Mutchler stwierdzili, że dla 3 replik głosowanie statyczne spisuje się lepiej niż dynamiczne; tutaj to pomijamy.*

### Prześledzenie przykładu
<sub>slajdy 17–21</sub>

![[swn-ft07-s17-jajodia-mutchler-stan.png]]
<sub>Stan początkowy: wszystkie $VN = 3$, $RU = 5$, $DS = -$ (*irrelevant*, bo $RU$ nieparzyste). $P_2$ chce zapisać i stwierdza, że komunikuje się tylko z $P_1$ i $P_3$. Ponieważ $VN_{max} = 3$, $RU$ związane z $VN_{max}$ wynosi 5, a partycja $\{P_1P_2P_3\}$ ma **3 z 5 kopii**, $P_2$ wie, że należy do **wyróżnionej partycji pierwotnej** — slajd 17</sub>

| Krok | Stan | Wydarzenie |
|---|---|---|
| slajd 18 | $VN$: 4,4,4,3,3 · $RU$: 3,3,3,5,5 | $P_3$ chce zapisać, komunikuje się tylko z $P_2$. $VN_{max}=4$, $RU=3$; partycja $\{P_2P_3\}$ ma **2 z 3 kopii** → $P_3$ należy do partycji pierwotnej i wykonuje zapis |
| slajd 19 | $VN$: 4,5,5,3,3 · $RU$: 3,2,2,5,5 · $DS$: –,$P_2$,$P_2$,–,– | nowa partycja większościowa $\{P_2P_3\}$ ma 2 węzły, $RU=2$ jest **parzyste**, więc $DS$ ustawiane na $P_2$ (**$P_2$ ma najwyższy porządek** z $\{P_2,P_3\}$). Teraz $P_4$ chce zapisać i odkrywa, że komunikuje się z $P_2$, $P_3$ i $P_5$; najświeższa wersja w $\{P_2P_3P_4P_5\}$ to $VN_{max}=5$ z $RU=2$, a **oba $P_2$, $P_3$ należą do nowej partycji**, która staje się pierwotną |
| slajd 20 | $VN$: 4,6,6,6,6 · $RU$: 3,4,4,4,4 · $DS$: –,$P_2$,$P_2$,$P_2$,$P_2$ | $RU=4$ jest parzyste, więc $DS = P_2$ ($P_2$ nadal ma najwyższy porządek). Wreszcie $P_3$ chce zapisać i odkrywa, że komunikuje się tylko z $P_2$ |
| slajd 21 | $VN$: 4,7,7,6,6 · $RU$: 3,2,2,4,4 · $DS$: –,$P_2$,$P_2$,$P_2$,$P_2$ | partycja $\{P_2P_3\}$ zawiera **dokładnie połowę** węzłów poprzedniej partycji **i zawiera wyróżniony węzeł $P_2$** (**$DS$ służy do rozstrzygnięcia remisu**) → modyfikacja jest wykonywana w partycji pierwotnej $\{P_2P_3\}$ |

> [!important] Rola $DS$
> $DS$ rozstrzyga **remis**: gdy nowa partycja zawiera **dokładnie połowę** węzłów poprzedniej, bez wyróżnionego węzła nie dałoby się wskazać, która z dwóch połówek jest pierwotna. Dlatego $DS$ istnieje tylko dla **parzystego** $RU$.

> [!note] Uzupełnienie spoza slajdów
> Notatka [[Dynamiczne głosowanie]] dopowiada dwie konsekwencje, których slajdy nie formułują wprost: **$DS$ nie istnieje, gdy $RU$ jest nieparzyste** (bo wtedy remis jest niemożliwy) oraz **jeśli $DS$ zostanie rozdzielony, system nie ma prawa postępu**.

### Statyczne a dynamiczne głosowanie
<sub>slajdy 22–24</sub>

![[swn-ft07-s22-static-vs-dynamic.png]]
<sub>Porównanie dla historii podziałów z grafu partycjonowania: **głosowanie dynamiczne pozwala na postęp tam, gdzie statyczne już nie** — przy podziale do $\{P_2P_3\}$ partycja większościowa nie istnieje, a partycja pierwotna tak; slajd 22</sub>

![[swn-ft07-s24-static-vs-dynamic-anomalia.png]]
<sub>Inna historia podziałów: głosowanie dynamiczne kurczy partycję pierwotną aż do **pojedynczego węzła $P_1$** — na slajdzie zaznaczone **wykrzyknikiem**. Po scaleniu $\{P_1P_3P_5\}$ obie metody znów zgadzają się co do partycji; slajd 24</sub>

> **Pułapka głosowania dynamicznego:** partycja pierwotna może skurczyć się do **jednego węzła** — wtedy wystarczy awaria tego jednego węzła, by system stracił możliwość postępu, mimo że działa większość replik.

### Dynamiczna realokacja głosów
<sub>slajd 25</sub>

**Dynamic vote reassignment — strategie:**

**Group Consensus:**
- węzły w grupie aktywnej (pierwotnej) **uzgadniają nowy przydział głosów** algorytmem konsensusu,
- ☹ **konsensus jest dość skomplikowany**.

**Autonomous Reassignment:**
- każdy węzeł używa **własnego widoku systemu**, by zdecydować o zmianie swoich głosów i wybraniu nowego $V_i$ **bez oglądania się na pozostałe węzły**,
- ☹ może nie być optymalny, ale…
- ☺ …jest **szybki, prosty i elastyczny**.

### Polityki zwiększania głosów
<sub>slajdy 26–28</sub>

**Vote increasing policies** (slajd 26):
1. **Overthrow technique** — **jeden węzeł** w grupie aktywnej przejmuje więcej głosów; **wymagana elekcja**.
2. **Alliance technique** — **wszystkie węzły** w grupie aktywnej zwiększają swoje głosy.

![[swn-ft07-s27-overthrow-technique.png]]
<sub>**Overthrow technique** (slajd 27): $V = \sum_i V_i$, $M = \left\lceil \frac{V+1}{2} \right\rceil = W$; $P_f$ uległ awarii, $P_e$ został wybrany, by zwiększyć swoją siłę głosu o $2V_f$; wówczas $V' = V + V_f$ oraz $W' = W + V_f$</sub>

**Alliance technique** (slajd 28):
- np. wszystkie $n$ pozostałych węzłów zwiększa swoją siłę głosu o $\left\lceil \frac{2V_f}{n} \right\rceil$ każdy,
- albo wszystkie $n$ pozostałych węzłów zwiększa swoją siłę głosu o $2V_f$ każdy,
- **$W'$ zmienia się odpowiednio**.

---
## Literatura
<sub>slajd 29</sub>

1. S. Jajodia, D. Mutchler, *Dynamic Voting*, ACM SIGMOD Conference, 1987.
2. A. Schiper, *Replication. Theory and Practice*, Springer, 2010.

---
## Zestawienie

| | **Statyczne (Gifford)** | **Dynamiczne (Jajodia-Mutchler)** |
|---|---|---|
| Co jest stałe | $V_i$, $R$, $W$ — niezależne od stanu | zbiór głosujących **zmienia się** po każdej modyfikacji |
| Warunki poprawności | $W \geqslant M$, $R + W > V$ | partycja pierwotna = większość **konfiguracji ostatniej modyfikacji** |
| Struktury | $VN_i$, $V_i$, $V$, $M$ | $VN_i$, $RU_i$, $DS_i$ |
| Partycjonowanie | działa tylko **partycja większościowa** | działa **partycja pierwotna** — może być mniejsza niż większość całości |
| Słabość | brak quorum przy wielu awariach lub podziale | partycja pierwotna może skurczyć się **do 1 węzła** |

---
## Braki i uwagi

> [!todo] Czego nie ma na slajdach tego wykładu
> - **Pełnego pseudokodu** protokołu Jajodii-Mutchlera — slajdy 17–21 pokazują wyłącznie prześledzony przykład; notatka [[Systemy Wysokiej Niezawodności/Algorytm Jajodia-Mutchlera]] w vaultcie jest **pusta**.
> - **Złożoności komunikacyjnej** obu algorytmów.
> - **Dowodu poprawności** warunków $W \geqslant M$ i $R + W > V$ — slajd 6 podaje je z uzasadnieniem słownym.
> - **Wariantu $RU_i = 3$** protokołu Jajodii-Mutchlera — jawnie pominięty przypisem na slajdzie 16.
> - **Polityk zmniejszania głosów** (*vote decreasing*) — slajd 26 wymienia tylko *vote increasing policies*.
> - **Algorytmów kworum opartych na strukturze** (siatka, drzewo, kworum $\sqrt{N}$ Maekawy) — w ogóle nie występują.
