---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 4
---
# 4. Algorytmy wzajemnego wykluczania
---
> **Rozproszone wzajemne wykluczanie** zapewnia, że w danej chwili **co najwyżej jeden proces** przebywa w **sekcji krytycznej** (SK). Procesy nie mają wspólnej pamięci ani zegara, więc koordynują się wyłącznie przez wymianę komunikatów.

## Wymagania
- **Bezpieczeństwo** (_safety_) – co najwyżej jeden proces w SK.
- **Żywotność** (_liveness_) – brak zakleszczeń i zagłodzeń; każde żądanie w końcu obsłużone.
- **Uczciwość** (_fairness_) – żądania obsługiwane w kolejności ich zgłoszenia (np. według znaczników czasowych).

## Miary
- **złożoność komunikacyjna** – liczba komunikatów na jedno wejście do SK,
- **opóźnienie synchronizacji** – czas od wyjścia jednego procesu z SK do wejścia następnego,
- **czas odpowiedzi** – czas od zgłoszenia żądania do wyjścia z SK,
- **przepustowość** – liczba wejść do SK na jednostkę czasu.

## Klasyfikacja
- **Oparte na zezwoleniach** (_permission-based_): proces wchodzi, gdy uzyska zgodę od wszystkich / kworum (Lamport, Ricart-Agrawala, Maekawa).
- **Oparte na żetonie** (_token-based_): do SK wchodzi posiadacz jedynego żetonu (Suzuki-Kasami, Raymond).

Wspólne założenia: niezawodne kanały, brak awarii procesów, unikalne identyfikatory; znaczniki czasowe Lamporta $(ts, id)$ – zob. [[01 Komunikacja grupowa#Zegary logiczne (podstawa uporządkowania)]].

---
## Algorytm Lamporta (1978)
**Założenia**: kanały **FIFO**. Każdy proces ma kolejkę żądań uporządkowaną wg $(ts, id)$.

1. **Żądanie**: $P_i$ wysyła $REQUEST(ts_i, i)$ do wszystkich i wstawia żądanie do własnej kolejki.
2. **Odbiór REQUEST** przez $P_j$: wstawia żądanie do kolejki, odsyła $REPLY$ (ze znacznikiem).
3. **Wejście do SK**, gdy spełnione oba warunki:
   - żądanie $P_i$ jest na **czele** jego kolejki,
   - $P_i$ otrzymał od **każdego** innego procesu komunikat ze znacznikiem **większym** niż $ts_i$.
4. **Wyjście**: $P_i$ usuwa swoje żądanie, wysyła $RELEASE$ do wszystkich; odbiorcy usuwają żądanie $P_i$ z kolejek.

- Komunikaty: $3(N-1)$ (REQUEST, REPLY, RELEASE).
- Kanały FIFO gwarantują, że po otrzymaniu komunikatu z większym znacznikiem od $P_j$ nie nadejdzie już starsze żądanie $P_j$.

## Algorytm Ricarta-Agrawali (1981)
Optymalizacja Lamporta: połączenie RELEASE i REPLY (odroczone odpowiedzi).

1. **Żądanie**: $P_i$ wysyła $REQUEST(ts_i, i)$ do wszystkich.
2. **Odbiór REQUEST** od $P_j$ przez $P_i$:
   - $P_i$ nie ubiega się i nie jest w SK → natychmiast $REPLY$,
   - $P_i$ jest w SK → **odracza** odpowiedź,
   - $P_i$ też się ubiega → porównuje $(ts_j, j)$ z $(ts_i, i)$: żądanie $P_j$ wcześniejsze → $REPLY$; w przeciwnym razie odracza.
3. **Wejście** po otrzymaniu $REPLY$ od wszystkich $N-1$ procesów.
4. **Wyjście**: wysłanie odroczonych $REPLY$.

- Komunikaty: $2(N-1)$; kanały FIFO niepotrzebne.
- Wada: awaria dowolnego procesu blokuje wszystkich (brak odpowiedzi).

## Algorytm Maekawy (1985)
Zgoda wymagana tylko od **kworum** $R_i$ (_request set_), nie od wszystkich.

**Własności kworów**:
1. $R_i \cap R_j \neq \emptyset$ dla każdych $i, j$ – każde dwa kworum mają wspólny proces (to on gwarantuje bezpieczeństwo),
2. $P_i \in R_i$,
3. $|R_i| = K$ dla każdego $i$,
4. każdy proces należy do dokładnie $K$ kworów.

Wtedy $N = K(K-1) + 1$, czyli $K \approx \sqrt{N}$ (praktycznie: kworum = wiersz + kolumna w siatce $\sqrt{N} \times \sqrt{N}$).

### Wersja podstawowa
- $P_i$ wysyła $REQUEST(ts, i)$ do członków $R_i$.
- Członek $P_j$: jeśli nie udzielił zgody nikomu → wysyła $LOCKED$ (_grant_) i zapamiętuje; w przeciwnym razie kolejkuje żądanie.
- $P_i$ wchodzi po zebraniu zgód od całego $R_i$; po wyjściu wysyła $RELEASE$, a członek udziela zgody następnemu z kolejki.

**Problem**: możliwe **zakleszczenie** – procesy zbierają zgody w różnej kolejności i każdy czeka na zgodę trzymaną przez inny (cykl).

### Wersja z rozwiązaniem zakleszczeń
Dodatkowe komunikaty: **FAILED**, **INQUIRE**, **RELINQUISH (YIELD)**.
- Członek $P_j$ udzielił zgody $P_k$ i dostaje żądanie od $P_i$:
  - żądanie $P_i$ **późniejsze** (niższy priorytet) niż $P_k$ lub niż któreś w kolejce → wysyła $P_i$ **FAILED** (żądanie zakolejkowane),
  - żądanie $P_i$ **wcześniejsze** niż $P_k$ → wysyła $P_k$ **INQUIRE** („czy na pewno możesz wejść?”).
- $P_k$ po otrzymaniu INQUIRE:
  - jeśli otrzymał jakikolwiek **FAILED** (nie ma szans zebrać wszystkich zgód) → odsyła **RELINQUISH** i oddaje zgodę; $P_j$ udziela jej $P_i$, a żądanie $P_k$ wraca do kolejki,
  - jeśli już zebrał wszystkie zgody → ignoruje, po wyjściu wyśle RELEASE.

- Komunikaty: $3\sqrt{N}$ (bez konfliktów) do $5\sqrt{N}$ (z INQUIRE/RELINQUISH/FAILED).
- Przebieg z zajęć (graf, zakleszczenie procesów 1, 2, 5): [[Drawing 2024-06-19 16.38.35.excalidraw]].

---
## Algorytm Suzuki-Kasami (1985) – żeton, rozgłaszanie
**Struktury**:
- $RN_i[j]$ – u każdego procesu: największy numer żądania od $P_j$, o którym wie $P_i$,
- żeton zawiera: $LN[j]$ – numer ostatniego **obsłużonego** żądania $P_j$, oraz kolejkę $Q$ procesów czekających.

**Algorytm**:
1. **Żądanie**: jeśli $P_i$ nie ma żetonu: $RN_i[i]{+}{+}$, rozgłasza $REQUEST(i, sn = RN_i[i])$.
2. **Odbiór REQUEST** $(i, sn)$ przez $P_j$: $RN_j[i] := \max(RN_j[i], sn)$ (stare/zdublowane żądania ignorowane). Jeśli $P_j$ ma **bezczynny** żeton i $RN_j[i] = LN[i] + 1$ → wysyła żeton do $P_i$.
3. **Wejście**: po otrzymaniu żetonu.
4. **Wyjście**: $LN[i] := RN_i[i]$; dla każdego $j$ z $RN_i[j] = LN[j] + 1$ (niezaspokojone żądanie) i $j \notin Q$ → dopisz $j$ do $Q$. Jeśli $Q$ niepusta → zdejmij pierwszy proces i wyślij mu żeton.

- Komunikaty: $0$ (proces ma już żeton) lub $N$ ($N-1$ REQUEST + 1 żeton).
- Warunek $RN[j] = LN[j]+1$ pozwala odróżnić aktualne żądania od przestarzałych.

## Algorytm Raymonda (1989) – żeton w drzewie
Procesy tworzą **drzewo rozpinające** (logiczne); żeton krąży po krawędziach drzewa.

**Zmienne procesu**:
- $HOLDER$ – sąsiad w kierunku posiadacza żetonu (lub sam proces, gdy ma żeton),
- $request\_q$ – kolejka FIFO sąsiadów (i siebie) żądających żetonu,
- $USING$ – czy proces jest w SK, $ASKED$ – czy wysłał już żądanie do $HOLDER$.

**Procedury**:
- **ASSIGN_PRIVILEGE**: jeśli $HOLDER = self$, $\neg USING$ i $request\_q$ niepusta → zdejmij $head$; $ASKED := false$; jeśli $head = self$ → $USING := true$ (wejście do SK); w przeciwnym razie $HOLDER := head$ i wyślij $PRIVILEGE$ do $head$.
- **MAKE_REQUEST**: jeśli $HOLDER \neq self$, $request\_q$ niepusta i $\neg ASKED$ → wyślij $REQUEST$ do $HOLDER$, $ASKED := true$.

**Zdarzenia**:
- chcę wejść: dopisz $self$ do $request\_q$; ASSIGN_PRIVILEGE; MAKE_REQUEST,
- odebrano $REQUEST$ od sąsiada $X$: dopisz $X$; ASSIGN_PRIVILEGE; MAKE_REQUEST,
- odebrano $PRIVILEGE$: $HOLDER := self$; ASSIGN_PRIVILEGE; MAKE_REQUEST,
- wyjście z SK: $USING := false$; ASSIGN_PRIVILEGE; MAKE_REQUEST.

Po przekazaniu żetonu, jeśli w kolejce są kolejni, proces od razu wysyła REQUEST do nowego $HOLDER$ (żeton „wróci”).

- Komunikaty: $O(\log N)$ średnio (drzewo zrównoważone), $O(D)$ w najgorszym przypadku ($D$ – średnica drzewa).

---
## Porównanie
| Algorytm | Typ | Komunikaty / wejście | Opóźnienie synchronizacji | Uwagi |
|---|---|---|---|---|
| Lamport | zezwolenia | $3(N-1)$ | $T$ | wymaga FIFO |
| Ricart-Agrawala | zezwolenia | $2(N-1)$ | $T$ | odroczone odpowiedzi |
| Maekawa | kworum | $3\sqrt{N}$ – $5\sqrt{N}$ | $2T$ | możliwe zakleszczenie → INQUIRE/RELINQUISH |
| Suzuki-Kasami | żeton + rozgłaszanie | $0$ lub $N$ | $0$ lub $T$ | tablice RN, LN |
| Raymond | żeton w drzewie | $O(\log N)$ | $T \log(N)/2$ | lokalna wiedza (HOLDER) |

($T$ – czas przesłania komunikatu.)

## Odporność na awarie
Klasyczne algorytmy zakładają brak awarii: utrata żetonu lub milczący proces blokują system. Rozszerzenia to wykrywanie utraty żetonu i jego regeneracja (np. uzgodnienie przez elekcję – [[05 Algorytmy elekcji]]), timeouty oraz kworum odporne na awarie. Pytania z zajęć o odporny algorytm żetonowy: [[Drawing 2024-12-11 08.09.32.excalidraw]].
