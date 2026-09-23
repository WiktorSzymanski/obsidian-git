---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 4
---
# 4. Algorytmy wzajemnego wykluczania
---
> Komplet pojęć zagadnienia. Algorytmy są nazwane i opatrzone informacją, jaki problem rozwiązują, ale ich przebieg krok po kroku pozostaje w [[RSO 04 Algorytmy wzajemnego wykluczania|RSO 04]]. Prezentacje omawiają tylko trzy z pięciu algorytmów wymaganych przez listę zagadnień — pozostałe są tu w blokach `[!note]`.

## Problem

**Wzajemne wykluczanie** zapewnia procesom ochronę przy dostępie do zasobów: daje gwarancję, że w danej chwili **co najwyżej jeden proces** przebywa w **sekcji krytycznej**. W systemie rozproszonym procesy nie mają ani wspólnej pamięci, ani wspólnego zegara, więc jedynym narzędziem koordynacji jest **wymiana komunikatów** — i stąd bierze się cała różnorodność rozwiązań.

> [!note] Uzupełnienie spoza prezentacji — wymagania i miary
> Algorytm wzajemnego wykluczania musi spełniać: **bezpieczeństwo** (*safety*) — co najwyżej jeden proces w sekcji krytycznej; **żywotność** (*liveness*) — brak zakleszczeń i zagłodzeń, każde żądanie zostaje w końcu obsłużone; **uczciwość** (*fairness*) — żądania obsługiwane w kolejności zgłoszenia, zwykle według znaczników czasowych.
> Ocenia się je czterema miarami: **złożonością komunikacyjną** (liczba komunikatów na jedno wejście), **opóźnieniem synchronizacji** (czas od wyjścia jednego procesu do wejścia następnego), **czasem odpowiedzi** i **przepustowością**. Źródło: [[04 Algorytmy wzajemnego wykluczania]].

## Klasyfikacja

Prezentacje wyliczają trzy typy: **podejście scentralizowane**, **algorytmy rozproszone** (Lamport) i **algorytmy bazujące na żetonie** (Suzuki-Kasami). Ogólniej przyjęty podział jest dwuczłonowy: algorytmy **oparte na zezwoleniach** (*permission-based*), w których proces wchodzi po uzyskaniu zgody wszystkich albo kworum (Lamport, Ricart-Agrawala, Maekawa), oraz **oparte na żetonie** (*token-based*), w których do sekcji wchodzi posiadacz jedynego żetonu (Suzuki-Kasami, Raymond).

Wspólne założenia wszystkich klasycznych algorytmów: niezawodne kanały, brak awarii procesów, unikalne identyfikatory.

Pojęciem porządkującym jest **zbiór żądań** $R_i$ — zbiór procesów, od których proces $P_i$ musi uzyskać pozwolenie. U Lamporta i Ricarta-Agrawali jest to **zbiór wszystkich procesów**, u Maekawy — **kworum**, a algorytmy żetonowe zastępują go posiadaniem żetonu.

## Podejście scentralizowane

Jeden proces pełni rolę **koordynatora** i utrzymuje **kolejkę żądań**. Proces chcący wejść do sekcji krytycznej wysyła żądanie do koordynatora; jeśli sekcja jest wolna, dostaje **pozwolenie**, a jeśli nie — jego żądanie zostaje **zakolejkowane** albo dostaje **odmowę**. Po wyjściu proces wysyła zwolnienie, a koordynator przekazuje pozwolenie następnemu z kolejki.

Podejście jest najprostsze i najtańsze, ale koordynator jest **pojedynczym punktem awarii** i wąskim gardłem. Zakłada też, że koordynator już istnieje — jego wyłonienie to osobne zagadnienie: [[RSO Z5 Algorytmy elekcji|zagadnienie 5]]. Przebieg: [[RSO 04 Algorytmy wzajemnego wykluczania#Podejście scentralizowane|RSO 04]].

## Algorytm Lamporta

Rozwiązuje problem wzajemnego wykluczania **bez koordynatora**, opierając się na **zegarze skalarnym Lamporta**. Każdy proces utrzymuje **własną kolejkę żądań** uszeregowaną według znaczników $(ts, id)$, a zbiorem żądań jest zbiór wszystkich procesów.

Istotą algorytmu są **dwa warunki wejścia naraz**: własne żądanie musi być **na czele kolejki** oraz od każdego innego procesu musi nadejść odpowiedź ze znacznikiem **większym** od znacznika żądania. Pierwszy warunek daje uporządkowanie, drugi — pewność, że żaden proces nie zgłosi już starszego żądania. Wyjście z sekcji polega na rozesłaniu komunikatu **ZWOLNIJ** i usunięciu żądania z kolejki.

Algorytm wymaga **kanałów FIFO** i kosztuje $3(N-1)$ komunikatów na wejście (żądanie, odpowiedź, zwolnienie). Znaczniki $(ts, id)$ to zegar skalarny: [[RSO 08 Czas wirtualny i złożoność algorytmów#Zegar skalarny|RSO 08]]. Przebieg: [[RSO 04 Algorytmy wzajemnego wykluczania#Algorytm Lamporta|RSO 04]].

## Algorytm Suzuki-Kasami

Rozwiązuje ten sam problem za pomocą **żetonu**: proces posiadający żeton może wchodzić do sekcji krytycznej dopóty, dopóki nie poprosi o niego ktoś inny. Cała trudność sprowadza się do dwóch problemów, które trzeba umieć nazwać.

**Problem żądań przedawnionych**: żądanie ma postać $\text{ŻĄDANIE}(j, n)$, gdzie $n$ znaczy, że $P_j$ ubiega się o **$n$-te** wykonanie sekcji krytycznej. Każdy proces trzyma tablicę $RN_i$, w której $RN_i[j]$ to **największy numer żądania otrzymany od $P_j$**; żądanie jest przedawnione, gdy $RN_i[j] > n$. Aktualizacja to $RN_i[j] := \max(RN_i[j], n)$.

**Problem żądań zaległych**: rozstrzyga go zawartość żetonu, na którą składają się **kolejka $Q$** procesów oczekujących i **tablica $LN$**, gdzie $LN[j]$ to numer **ostatnio zrealizowanego** żądania procesu $P_j$. Po wyjściu z sekcji proces ustawia $LN[i] := RN_i[i]$.

Kluczem jest zestawienie obu tablic w warunku
$$RN_i[j] = LN[j] + 1$$
który czyta się: „$P_j$ **zgłosił** żądanie o numerze o jeden większym niż numer żądania, które **zrealizował**", czyli $P_j$ ma **oczekujące, nieobsłużone** żądanie. Ten sam warunek służy do przekazania nieużywanego żetonu i do uzupełniania kolejki $Q$ przy wyjściu z sekcji.

Koszt: **$0$ komunikatów**, gdy proces już ma żeton, albo $N$ ($N-1$ żądań plus żeton). Przebieg: [[RSO 04 Algorytmy wzajemnego wykluczania#Algorytm Suzuki-Kasami|RSO 04]].

> [!note] Uzupełnienie spoza prezentacji — Ricart-Agrawala
> Optymalizacja algorytmu Lamporta, która rozwiązuje problem **nadmiarowej liczby komunikatów**, łącząc zwolnienie z odpowiedzią. Proces, który otrzymał żądanie, odpowiada natychmiast, jeśli sam nie ubiega się o sekcję; **odracza** odpowiedź, jeśli jest w sekcji; a jeśli też się ubiega — rozstrzyga porównaniem znaczników $(ts, id)$. Wejście następuje po zebraniu odpowiedzi od wszystkich $N-1$ procesów, wyjście polega na wysłaniu odpowiedzi odroczonych. Koszt $2(N-1)$, **bez wymogu kanałów FIFO**. Wada: awaria dowolnego procesu blokuje wszystkich. Źródło: [[04 Algorytmy wzajemnego wykluczania]].

> [!note] Uzupełnienie spoza prezentacji — Maekawa
> Rozwiązuje problem **liniowego kosztu komunikacyjnego**, żądając zgody nie od wszystkich, lecz od **kworum** $R_i$. Kwora muszą spełniać cztery własności: każde dwa mają **niepuste przecięcie** (i to właśnie wspólny proces gwarantuje bezpieczeństwo), proces należy do własnego kworum, wszystkie kwora są **równoliczne** ($|R_i| = K$), a każdy proces należy do dokładnie $K$ kworów. Stąd $N = K(K-1)+1$, czyli $K \approx \sqrt{N}$ — praktycznie kworum to wiersz i kolumna w siatce $\sqrt{N} \times \sqrt{N}$.
> Wersja podstawowa może się **zakleszczyć**: procesy zbierają zgody w różnej kolejności i każdy czeka na zgodę trzymaną przez innego. Rozwiązanie dokłada trzy komunikaty — **FAILED** (twoje żądanie ma niższy priorytet), **INQUIRE** (czy na pewno możesz wejść?) i **RELINQUISH/YIELD** (oddaję zgodę) — dzięki którym proces bez szans na komplet zgód zwraca zgodę procesowi o wyższym priorytecie. Koszt od $3\sqrt{N}$ do $5\sqrt{N}$. Źródło: [[04 Algorytmy wzajemnego wykluczania]], przebieg z zajęć: [[Drawing 2024-06-19 16.38.35.excalidraw]].

> [!note] Uzupełnienie spoza prezentacji — Raymond
> Algorytm żetonowy, który rozwiązuje problem **rozgłaszania żądań do wszystkich**: procesy tworzą logiczne **drzewo rozpinające**, a każdy proces zna tylko **kierunek do posiadacza żetonu** (zmienna $HOLDER$) i trzyma kolejkę FIFO żądających sąsiadów. Żądania wędrują po krawędziach drzewa w stronę żetonu, żeton wraca tą samą drogą. Wiedza jest więc wyłącznie **lokalna**, a koszt spada do $O(\log N)$ komunikatów w drzewie zrównoważonym (w najgorszym razie $O(D)$ dla średnicy $D$). Źródło: [[04 Algorytmy wzajemnego wykluczania]].

## Zestawienie

| Algorytm | Typ | Komunikaty na wejście | Uwagi |
|---|---|---|---|
| scentralizowany | koordynator | $3$ | SPoF, wąskie gardło |
| **Lamport** | zezwolenia | $3(N-1)$ | wymaga kanałów FIFO |
| Ricart-Agrawala | zezwolenia | $2(N-1)$ | odroczone odpowiedzi |
| Maekawa | kworum | $3\sqrt{N}$–$5\sqrt{N}$ | możliwe zakleszczenie |
| **Suzuki-Kasami** | żeton + rozgłaszanie | $0$ lub $N$ | tablice $RN$, $LN$, kolejka $Q$ |
| Raymond | żeton w drzewie | $O(\log N)$ | wiedza lokalna ($HOLDER$) |

Pogrubione są algorytmy obecne w prezentacjach; porównanie tych trzech: [[RSO 04 Algorytmy wzajemnego wykluczania#Porównanie podejść|RSO 04]].

> [!note] Uzupełnienie spoza prezentacji — odporność na awarie
> Wszystkie powyższe algorytmy zakładają brak awarii: **utrata żetonu** albo milczący proces blokują system. Rozszerzenia polegają na wykrywaniu utraty żetonu i jego **regeneracji** (uzgodnionej zwykle przez elekcję — [[RSO Z5 Algorytmy elekcji|zagadnienie 5]]), na timeoutach oraz na kworach odpornych na awarie. Źródło: [[04 Algorytmy wzajemnego wykluczania]], [[Drawing 2024-12-11 08.09.32.excalidraw]].

---
## Czego w prezentacjach nie ma

> [!todo] Zakres zagadnienia wykracza poza slajdy
> Prezentacje nie zawierają: **wymagań** stawianych algorytmom (bezpieczeństwo, żywotność, uczciwość) ani **miar oceny**, **złożoności komunikacyjnej żadnego** z trzech omawianych algorytmów, algorytmów **Ricarta-Agrawali**, **Maekawy** i **Raymonda** (po $0$ trafień w 427 slajdach) ani formalnego podziału na algorytmy oparte na zezwoleniach i na żetonie. Algorytmy podane są opisowo, **bez pseudokodu**. Rejestr braków: [[RSO 04 Algorytmy wzajemnego wykluczania#Braki w prezentacjach]]. Materiał uzupełniający: [[04 Algorytmy wzajemnego wykluczania]], [[Synchronizacja]].
