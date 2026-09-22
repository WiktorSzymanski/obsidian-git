---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 6
---
# 6. Zakleszczenie w systemach rozproszonych
---
> **Zakleszczenie** (_deadlock_) to stan, w którym zbiór procesów jest **trwale zablokowany**: każdy proces ze zbioru czeka na zdarzenie (zwolnienie zasobu, komunikat), które może spowodować tylko inny proces z tego zbioru. W systemie rozproszonym nie ma globalnego stanu ani wspólnego zegara, więc wykrycie zakleszczenia wymaga specjalnych algorytmów.

## Warunki konieczne (Coffman)
1. **wzajemne wykluczanie** – zasób używany przez co najwyżej jeden proces,
2. **przetrzymywanie i oczekiwanie** – proces trzyma zasoby i czeka na kolejne,
3. **brak wywłaszczania** – zasobów nie można odebrać,
4. **cykliczne oczekiwanie**.

## Graf zależności (WFG – _Wait-For Graph_)
- Wierzchołki – procesy; krawędź $P_i \rightarrow P_j$ oznacza, że $P_i$ czeka na $P_j$ (na zasób lub komunikat od $P_j$).
- **Zbiór zależności** $DS_i$ – procesy, na które czeka $P_i$.
- Globalny WFG jest rozproszony – każdy proces zna tylko swoje krawędzie wychodzące.

## Modele zakleszczeń (modele żądań)
| Model | Kiedy proces zostaje odblokowany | Warunek zakleszczenia w WFG |
|---|---|---|
| **jednozasobowy** | otrzyma jeden zasób (stopień wyjściowy ≤ 1) | **cykl** |
| **AND** | otrzyma **wszystkie** żądane zasoby | **cykl** |
| **OR** | otrzyma **dowolny jeden** z żądanych | **węzeł** (_knot_): zbiór wierzchołków, z których osiągalne są tylko wierzchołki tego zbioru, bez krawędzi wychodzących poza zbiór |
| **AND-OR** | spełni dowolną formułę logiczną (np. $a \wedge (b \vee c)$) | brak prostej charakteryzacji grafowej – analiza stanu globalnego |
| **p-z-q** ($p$-_out-of_-$q$, k-z-n) | otrzyma dowolne $p$ z $q$ żądanych | uogólnia AND ($p=q$) i OR ($p=1$); wykrywanie przez symulację przydziałów |
| **nieograniczony** | dowolny warunek | analiza stabilnej własności stanu globalnego |

W modelu OR cykl **nie oznacza** zakleszczenia – proces może zostać odblokowany przez krawędź prowadzącą poza cykl.

## Strategie obsługi
- **Zapobieganie** (_prevention_) – eliminacja jednego z warunków: przydział wszystkich zasobów na starcie, wywłaszczanie, **globalne uporządkowanie zasobów** (brak cykli), znaczniki czasowe **wait-die** (starszy czeka, młodszy się wycofuje) / **wound-wait** (starszy wywłaszcza młodszego).
- **Unikanie** (_avoidance_) – np. algorytm bankiera. W systemie rozproszonym niepraktyczne: wymaga globalnego stanu przy każdym przydziale.
- **Wykrywanie i usuwanie** (_detection & resolution_) – najczęstsze. Wymaga:
  - **postępu**: każde zakleszczenie wykryte w skończonym czasie,
  - **bezpieczeństwa**: brak **fantomowych zakleszczeń** – fałszywych wykryć spowodowanych nieaktualnym, niespójnym obrazem WFG (np. zwolnienie zasobu jeszcze nie dotarło do detektora).
  - **usunięcia**: wycofanie ofiary i **usunięcie informacji o jej krawędziach** z pozostałych węzłów.

## Klasy algorytmów wykrywania
- **scentralizowane** – koordynator buduje globalny WFG (np. Ho-Ramamoorthy); ryzyko fantomów, SPoF,
- **przepychanie ścieżek** (_path-pushing_, Obermarck) – węzły przesyłają sobie fragmenty ścieżek WFG,
- **pogoń za krawędziami** (_edge-chasing_) – sondy wzdłuż krawędzi WFG (Chandy-Misra-Haas AND),
- **obliczenia dyfuzyjne** (_diffusing computation_) – echa w WFG (Chandy-Misra-Haas OR),
- **stan globalny** – migawka WFG i jej analiza (Bracha-Toueg, Kshemkalyani-Singhal).

---
## Algorytm Chandy-Misra-Haas – model AND (1983, _edge-chasing_)
**Sonda** $probe(i, j, k)$: inicjowana przez $P_i$, wysłana przez $P_j$ do $P_k$.

1. **Inicjacja**: zablokowany proces $P_i$ (podejrzewa zakleszczenie – np. długo czeka) wysyła $probe(i, i, k)$ do każdego $P_k \in DS_i$.
2. **Odbiór** $probe(i, j, k)$ przez $P_k$:
   - jeśli $P_k$ jest **zablokowany**, $P_j$ faktycznie na niego czeka i $P_k$ nie przesyłał jeszcze sond od $P_i$ → wysyła $probe(i, k, m)$ do każdego $P_m \in DS_k$,
   - jeśli $P_k$ jest aktywny → sonda jest odrzucana.
3. **Wykrycie**: jeśli $P_i$ otrzyma $probe(i, j, i)$ → sonda wróciła po cyklu → $P_i$ jest zakleszczony.

- Każda krawędź WFG przenosi sondę inicjatora co najwyżej raz → co najwyżej $|E|$ komunikatów (w wersji zasobowej: $m(n-1)/2$ dla $m$ procesów na $n$ węzłach).
- Komunikaty małe (3 identyfikatory), brak budowania globalnego grafu, brak fantomów (przy modelu AND i poprawnej implementacji).
- Wersja oryginalna rozróżnia procesy lokalne (zależności badane lokalnie) i sondy międzywęzłowe.

## Algorytm Chandy-Misra-Haas – model OR (obliczenia dyfuzyjne)
Wykrywa **węzeł** w WFG. Komunikaty: $query(i, j, k)$ i $reply(i, j, k)$.

1. **Inicjacja**: zablokowany $P_i$ wysyła $query(i, i, j)$ do wszystkich $P_j \in DS_i$ i zapamiętuje liczbę oczekiwanych odpowiedzi $num_i(i) := |DS_i|$.
2. **Odbiór** $query(i, j, k)$ przez $P_k$:
   - $P_k$ **aktywny** → ignoruje (nie odpowiada; to „ratuje” inicjatora),
   - $P_k$ zablokowany i to **pierwsze** zapytanie od inicjatora $i$ (_engaging query_) → zapamiętuje nadawcę $engager_k(i) := j$, wysyła $query(i, k, m)$ do wszystkich $P_m \in DS_k$, ustawia $num_k(i) := |DS_k|$, $wait_k(i) := true$,
   - $P_k$ zablokowany, kolejne zapytanie od $i$ (i nadal $wait_k(i)$) → od razu odpowiada $reply(i, k, j)$.
3. **Odbiór** $reply(i, j, k)$ przez $P_k$ z $wait_k(i)$: $num_k(i){-}{-}$; gdy $num_k(i) = 0$:
   - $k = i$ → **zakleszczenie** $P_i$,
   - w przeciwnym razie wysyła $reply(i, k, engager_k(i))$.
4. Jeśli $P_k$ zostanie w międzyczasie odblokowany → $wait_k(i) := false$, przestaje odpowiadać.

Inicjator dostaje odpowiedzi na wszystkie zapytania tylko wtedy, gdy **wszystkie** osiągalne procesy są zablokowane – czyli należy do węzła.
- Komunikaty: $2|E|$ na jedno uruchomienie (jedno query i jedno reply na krawędź).

---
## Algorytm Bracha-Toueg (1987) – model p-z-q (N-out-of-M)
> [!warning] Nie mylić
> [[Systemy Wysokiej Niezawodności/Algorytm Bracha-Touega]] opisuje probabilistyczny **algorytm konsensusu** tych samych autorów. Tu chodzi o **algorytm wykrywania zakleszczeń**.

**Idea**: najpierw wykonujemy **migawkę** (spójny stan globalny) WFG. Następnie **symulujemy** wysyłanie przydziałów przez procesy, które mogłyby zostać odblokowane. Proces, który w symulacji nie zostanie odblokowany, jest zakleszczony.

**Stan w migawce** dla procesu $u$:
- $Out_u$ – procesy, do których $u$ wysłał niezrealizowane żądania,
- $In_u$ – procesy, które czekają na $u$ (wysłały do niego żądania),
- $requests_u$ – liczba przydziałów, których $u$ jeszcze potrzebuje (w modelu $p$-z-$q$: $p$ minus liczba już otrzymanych); $requests_u = 0$ ⇔ $u$ nie jest zablokowany,
- flagi $notified_u := false$, $free_u := false$.

**Komunikaty**: **NOTIFY**, **GRANT**, **DONE**, **ACK**.

**Faza 1 – migawka**: obraz $Out$/$In$ z uwzględnieniem komunikatów w kanałach, np. algorytmem Chandy-Lamport.

**Faza 2 – symulacja** (inicjator wykonuje $Notify$):
```
Notify(u):
    notified_u := true
    wyślij NOTIFY do wszystkich w ∈ Out_u
    if requests_u = 0 then Grant(u)
    czekaj na DONE od wszystkich w ∈ Out_u

Grant(u):
    free_u := true
    wyślij GRANT do wszystkich w ∈ In_u
    czekaj na ACK od wszystkich w ∈ In_u

po odebraniu NOTIFY od v:
    if not notified_u then Notify(u)
    wyślij DONE do v

po odebraniu GRANT od v:
    if requests_u > 0 then
        requests_u := requests_u - 1
        if requests_u = 0 then Grant(u)
    wyślij ACK do v
```
- NOTIFY rozchodzi się falą wzdłuż krawędzi $Out$ (w stronę procesów, na które się czeka); procesy niezablokowane zaczynają „rozdawać” GRANT wzdłuż $In$.
- GRANT symuluje odblokowanie – proces, który zbierze wymagane $p$ przydziałów, sam staje się wolny i rozdaje GRANT dalej.
- DONE/ACK zapewniają, że inicjator wie o zakończeniu symulacji.

**Wynik**: po zakończeniu **inicjator jest zakleszczony ⇔** $free_{inicjator} = false$.

- Działa dla ogólnego modelu $p$-z-$q$ (w szczególności AND i OR).
- Koszt: $O(|E|)$ komunikatów na każdą fazę (4 typy komunikatów na krawędź + migawka).

## Porównanie
| Algorytm | Model | Technika | Komunikaty |
|---|---|---|---|
| CMH AND | AND | pogoń za krawędziami (probe) | $\le \lvert E\rvert$ |
| CMH OR | OR | obliczenia dyfuzyjne (query/reply) | $2\lvert E\rvert$ |
| Bracha-Toueg | p-z-q | migawka + symulacja przydziałów | $O(\lvert E\rvert)$ |

## Usuwanie zakleszczenia
- wybór ofiary (najmłodszy proces, najmniejszy koszt wycofania), wycofanie (_abort_) i zwolnienie zasobów,
- **usunięcie z WFG** krawędzi ofiary u wszystkich zainteresowanych – inaczej pozostałości sond spowodują wykrycie fantomowe,
- przy wielu jednoczesnych inicjatorach – priorytet po id, by nie wycofywać kilku procesów z tego samego cyklu.

## Zobacz też
- [[04 Algorytmy wzajemnego wykluczania#Algorytm Maekawy (1985)]] – przykład zakleszczenia w algorytmie
- [[Synchronizacja]] – warunki zakleszczenia w programach współbieżnych
