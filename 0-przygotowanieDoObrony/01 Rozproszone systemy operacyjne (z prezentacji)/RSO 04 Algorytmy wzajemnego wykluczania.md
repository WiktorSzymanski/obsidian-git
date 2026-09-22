---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 4
source: "rso_sum_07.pdf"
slajdy: "1–13"
---
# RSO 04. Algorytmy wzajemnego wykluczania
---
> Pierwsza część wykładu 7 omawia **wzajemne wykluczanie** w systemie rozproszonym: podejście **scentralizowane** (koordynator z kolejką żądań), algorytm **Lamporta** (rozproszony, oparty na zezwoleniach i znacznikach czasowych) oraz algorytm **Suzuki-Kasami** (oparty na żetonie).

> [!warning] Zakres slajdów vs zakres zagadnienia
> Prezentacja omawia **tylko trzy** podejścia. **Nie ma** algorytmów Ricarta-Agrawali, Maekawy ani Raymonda, ani formalnej analizy miar złożoności wzajemnego wykluczania. Zob. [[#Braki w prezentacjach]].

---
## Wzajemne wykluczanie — definicja i typy algorytmów
<sub>rso_sum_07.pdf, slajd 2</sub>

> **Wzajemne wykluczanie** zapewnia procesom ochronę przy dostępie do zasobów, daje im np. gwarancję, że jako jedyne będą mogły z nich korzystać.

**Typy algorytmów**:
- **Podejście scentralizowane**
- **Algorytmy rozproszone** — algorytm Lamporta
- **Algorytmy bazujące na żetonie** — algorytm Suzuki-Kasami

---
## Podejście scentralizowane
<sub>rso_sum_07.pdf, slajdy 3–4</sub>

1. Jeden proces jest wybierany jako **koordynator**.
2. Proces $P$, który chce wejść do sekcji krytycznej, wysyła wiadomość do koordynatora.
3. Jeżeli inny proces nie przebywa aktualnie w danej sekcji krytycznej, koordynator odsyła **pozwolenie** do $P$.
4. Po otrzymaniu pozwolenia $P$ wchodzi do sekcji krytycznej.
5. Jeżeli w tym samym czasie do tej samej sekcji krytycznej chce się dostać inny proces, jego prośba jest **rozpatrywana później** lub otrzymuje **wiadomość odmowną**.

![[rso-w7-s04-scentralizowane-przyklad.png]]
<sub>Podejście scentralizowane — przykład. Koordynator $P_C$ utrzymuje **kolejkę żądań** (tu: $P_3$, $P_2$). Sekwencja: (1) Żądanie od $P_1$ → (2) Odpowiedź → (3) Żądanie od $P_2$ (trafia do kolejki) → (4) Żądanie od $P_3$ (do kolejki) → (5) Zwolnij od $P_1$ → (6) Odpowiedź do $P_2$ → (7) Zwolnij od $P_2$ → (8) Odpowiedź do $P_3$ → (9) Zwolnij od $P_3$. rso_sum_07.pdf, slajd 4</sub>

---
## Algorytm Lamporta
<sub>rso_sum_07.pdf, slajdy 5–7</sub>

### Wprowadzenie
<sub>slajd 5</sub>
- Wykorzystuje mechanizm **synchronizacji zegarów Lamporta**.
- **Zbiór żądań** — zbiór procesów, od których wymagane są pozwolenia na wejście do sekcji krytycznej.
- Zbiór żądań w algorytmie Lamporta jest zbiorem **wszystkich procesów**.
- Każdy proces przechowuje **kolejkę żądań** sekcji krytycznej uszeregowanych według znaczników czasowych.

### Algorytm
<sub>slajd 6</sub>

**1) Żądanie sekcji krytycznej w procesie $P_i$**
- wysłanie żądania ze znacznikiem czasu $(ts(i), i)$ do wszystkich procesów ze zbioru $R_i$,
- dodanie żądania do kolejki (również lokalnie),
- odesłanie odpowiedzi ze znacznikami czasu.

**2) Wejście do sekcji krytycznej, gdy:**
- odpowiedzi od wszystkich procesów $P_j$ mają znaczniki czasowe $(ts(j), i)$ **większe** od znacznika żądania,
- żądanie to jest **na początku kolejki** procesu żądającego.

**3) Zwalnianie sekcji krytycznej**
- wysłanie wiadomości **ZWOLNIJ** do innych procesów,
- usunięcie żądania z początku kolejki.

![[rso-w7-s07-lamport-przyklad.png]]
<sub>Algorytm Lamporta — przykład. Etykiety $\{(ts, id)\}$ pokazują zawartość kolejki żądań w każdym procesie. rso_sum_07.pdf, slajd 7</sub>

> [!tip] Powiązanie
> Mechanizm znaczników czasowych $(ts(i), i)$ to zegar skalarny Lamporta — zob. [[RSO 08 Czas wirtualny i złożoność algorytmów#Algorytm Lamporta]].

---
## Algorytm Suzuki-Kasami
<sub>rso_sum_07.pdf, slajdy 8–13</sub>

### Wprowadzenie
<sub>slajd 8</sub>
- Wykorzystywany jest **żeton**, o który ubiegają się procesy chcące wejść do sekcji krytycznej.
- Proces, który posiada żeton, może wchodzić do sekcji krytycznej **do czasu, gdy nie poprosi o niego inny proces**.
- Pojawiają się problemy, co zrobić ze: **starymi (przedawnionymi) żądaniami** i **zaległymi żądaniami**.

### Problem przedawnionych żądań
<sub>slajd 9</sub>
- Żądania mają postać $\text{ŻĄDANIE}(j, n)$, gdzie $n$ oznacza, że proces $P_j$ żąda **$n$-tego** wykonania sekcji krytycznej.
- $RN_i[1..N]$ jest tablicą przechowywaną przez proces $P_i$; $RN_i[j]$ oznacza **największą liczbę porządkową** otrzymaną w żądaniu od procesu $P_j$.
- $\text{ŻĄDANIE}(j, n)$ otrzymane przez $P_i$ jest **przedawnione**, jeżeli $RN_i[j] > n$.
- Po otrzymaniu przez $P_i$ wiadomości $\text{ŻĄDANIE}(j, n)$: $RN_i[j] = \max(RN_i[j], n)$.

### Problem zaległych żądań
<sub>slajd 10</sub>
- Zaległe żądania określane są przy użyciu zawartości **żetonu**, który składa się z **kolejki żetonu $Q$** i **tablicy żetonu $LN$**.
- $Q$ — kolejka procesów żądających.
- $LN$ jest tablicą o rozmiarze $N$, a $LN[j]$ oznacza liczbę porządkową **ostatnio wykonanego żądania** przez proces $P_j$.
- Po wykonaniu sekcji krytycznej proces $P_i$ aktualizuje tablicę $LN$: $LN[i] := RN_i[i]$.
- Na podstawie tablicy $LN$ można stwierdzić, czy jakiś proces ma zaległe żądanie.

### Algorytm — zarys
<sub>slajd 11</sub>

1. **Żądanie sekcji krytycznej**: wysłanie przez proces żądania o żeton, jeżeli go nie ma; odesłanie wolnego żetonu.
2. **Wejście do sekcji** po otrzymaniu żetonu.
3. **Zwalnianie sekcji krytycznej**: aktualizacja tablicy oraz kolejki żetonu; wysłanie żetonu do następnego procesu w kolejce.

### Algorytm — szczegóły
<sub>slajdy 12–13</sub>

**1) Żądanie sekcji krytycznej**
- Jeśli proces $P_i$ nie ma żetonu, to inkrementuje wartość $RN_i[i]$ i wysyła $\text{ŻĄDANIE}(i, sn)$, gdzie $sn = RN_i[i]$.
- Kiedy proces $P_j$ otrzymuje ŻĄDANIE, to uaktualnia $RN_j[i] = \max(RN_j[i], sn)$.
- Jeżeli $P_j$ ma **nieużywany** token, to wysyła go do procesu $P_i$, jeśli $RN_j[i] = LN[i] + 1$.

**2) Wejście do sekcji po otrzymaniu żetonu**
- Proces $P_i$ wchodzi do sekcji krytycznej po otrzymaniu żetonu.

**3) Wyjście procesu $P_i$ z sekcji krytycznej**
- $LN[i] = RN_i[i]$.
- Dla każdego procesu $P_j$, którego identyfikatora nie ma w kolejce żetonu, jego identyfikator dodawany jest do kolejki $Q$, jeśli spełniony jest warunek $RN_i[j] = LN[j] + 1$.
- Jeżeli kolejka $Q$ nie jest pusta, $P_i$ usuwa identyfikator z początku kolejki i wysyła żeton do procesu oznaczonego tym identyfikatorem.

> [!important] Klucz do zrozumienia
> Warunek $RN_i[j] = LN[j] + 1$ oznacza: „proces $P_j$ **zgłosił** żądanie o numerze o jeden większym niż numer ostatniego żądania, które $P_j$ **zrealizował**" — czyli $P_j$ ma **oczekujące, nieobsłużone** żądanie.

---
## Porównanie podejść

| | Scentralizowane | Lamporta | Suzuki-Kasami |
|---|---|---|---|
| **Typ** | koordynator | rozproszone, oparte na **zezwoleniach** | oparte na **żetonie** |
| **Struktura danych** | kolejka żądań u koordynatora | kolejka żądań **u każdego procesu**, uszeregowana wg $(ts, id)$ | żeton = kolejka $Q$ + tablica $LN$; lokalnie tablica $RN_i$ |
| **Wejście do SK** | po otrzymaniu pozwolenia od koordynatora | odpowiedzi od wszystkich o większych znacznikach **i** własne żądanie na czele kolejki | posiadanie żetonu |
| **Wada** | koordynator = pojedynczy punkt awarii | wysoka złożoność komunikacyjna | problemy z przedawnionymi i zaległymi żądaniami |

---
## Braki w prezentacjach

> [!todo] Wymagania i miary
> Prezentacja **nie formułuje** wymagań stawianych algorytmom wzajemnego wykluczania (bezpieczeństwo, żywotność, uczciwość) ani miar oceny (liczba komunikatów na wejście do SK, opóźnienie synchronizacji, czas odpowiedzi, przepustowość). Ogólne pojęcia bezpieczeństwa i żywotności są w [[RSO 08 Czas wirtualny i złożoność algorytmów#Warunki poprawności]], ale **nie są** zastosowane do wzajemnego wykluczania.

> [!todo] Złożoność algorytmów
> Nie podano złożoności komunikacyjnej **żadnego** z trzech algorytmów (np. $3(N-1)$ dla Lamporta). Uzupełnić.

> [!todo] Algorytm Ricarta-Agrawali
> Nie występuje w żadnej z 7 prezentacji (0 trafień frazy „Ricart"). Uzupełnić — REPLY z odroczeniem, $2(N-1)$ komunikatów.

> [!todo] Algorytm Maekawy
> Nie występuje w żadnej z 7 prezentacji (0 trafień). Uzupełnić — kworum $\sqrt{N}$, komunikaty INQUIRE/FAILED/RELINQUISH, możliwość zakleszczenia.

> [!todo] Algorytm Raymonda
> Nie występuje w żadnej z 7 prezentacji (0 trafień). Uzupełnić — żeton na drzewie, $O(\log N)$.

> [!todo] Formalna klasyfikacja
> Prezentacja nie wprowadza podziału na algorytmy **oparte na zezwoleniach** vs **oparte na żetonie** jako formalnej klasyfikacji — jedynie wylicza trzy „typy algorytmów" na slajdzie 2.

> [!todo] Pseudokod
> W odróżnieniu od wykładów 2–5, algorytmy wzajemnego wykluczania podane są **opisowo**, bez pseudokodu w konwencji `when e_... do`.

---
## Powiązania
- Zegar skalarny Lamporta jako podstawa znaczników $(ts, id)$ → [[RSO 08 Czas wirtualny i złożoność algorytmów#Zegar skalarny]]
- Druga część wykładu 7 — zakleszczenia → [[RSO 06 Zakleszczenie w systemach rozproszonych]]
- Stan globalny jako narzędzie oceny (np. zaginięcie/zwielokrotnienie żetonu) → [[RSO 09 Stan globalny i migawki#Ocena stanów globalnych i porównanie reprezentacji]]
- Elekcja koordynatora dla podejścia scentralizowanego → [[RSO 05 Algorytmy elekcji]]
