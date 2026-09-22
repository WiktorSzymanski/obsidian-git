---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 2
source: "rso_sum_06.pdf"
slajdy: "1–55"
---
# RSO 02. Zwielokrotnianie i danocentryczne modele spójności
---
> Wykład 6 zaczyna od **motywacji zwielokrotniania** (niezawodność, efektywność, DSM), przechodzi przez **koncepcje dostępu do danych** (zdalny, relokacja, zwielokrotnianie) i **protokoły koherencji**, a następnie formalnie definiuje **danocentryczne modele spójności**: atomową, sekwencyjną, przyczynową, PRAM, podręczną i procesorową — każdą z warunkiem formalnym, przykładem historii i protokołem realizującym.

---
## Zwielokrotnianie — motywacja
<sub>rso_sum_06.pdf, slajdy 2–4</sub>

> **Zwielokrotnianie** polega na utrzymywaniu wielu kopii danych (obiektów) na **niezależnych serwerach**.

**Cele zwielokrotniania**:
1. zwiększenie **efektywności**,
2. zwiększenie **niezawodności** (i dostępności).

**Główny problem — spójność**: modele spójności · protokoły spójności · mobilność.

**Po co zwielokrotniać?** <sub>slajd 3</sub>

| Niezawodność | Efektywność |
|---|---|
| odporność na awarie | współbieżny dostęp do wielu serwerów: równoważenie obciążenia, skalowalność w sensie liczbowym |
| | wykorzystanie bliższych serwerów: skalowalność w sensie geograficznym, mniejsze opóźnienia |

**Zwielokrotnianie a skalowalność — kompromis** <sub>slajd 5</sub>:
- skracanie czasu dostępu do serwerów,
- utrzymywanie kopii w stanie spójnym — koszt utrzymywania **nieużywanych** kopii,
- spójność odwzorowująca system scentralizowany — transakcyjna aktualizacja wszystkich kopii, globalna synchronizacja,
- **osłabienie modelu spójności** — zależnie od charakteru aplikacji i zwielokrotnianych danych.

---
## DSM — rozproszona pamięć dzielona
<sub>rso_sum_06.pdf, slajdy 6–9</sub>

> **DSM** (ang. _Distributed Shared Memory_) — wspólna wirtualna przestrzeń adresowa, dostępna dla wszystkich węzłów systemu rozproszonego.

**Zalety**:
- wygodny paradygmat programowania równoległego,
- skalowalność i łatwość rozbudowy,
- dostęp do fizycznej pamięci wszystkich węzłów,
- środowisko uruchomieniowe dla programów równoległych pisanych dla maszyn wieloprocesorowych.

**Obsługa błędu strony w systemie DSM** (slajdy 8–9): analogicznie do pamięci wirtualnej, ale zamiast urządzenia wymiany (dysku) brakująca strona sprowadzana jest **z innego węzła sieci**.

---
## Koncepcje dostępu do danych
<sub>rso_sum_06.pdf, slajdy 10–20</sub>

| Koncepcja | Opis | Zalety / wady |
|---|---|---|
| **Dostęp zdalny** | zawsze poprzez sieć | prostota koncepcji i implementacji; opóźnienia komunikacyjne |
| **Relokacja** | fizyczna zmiana lokalizacji obiektu | zmniejszenie czasu dostępu; koszt przenoszenia obiektu między węzłami; opłacalne przy wielokrotnych, zgrupowanych odwołaniach |
| **Zwielokrotnianie** | kopie w lokalnych węzłach | zmniejszenie czasu dostępu; **problem spójności** |

**Relokacja — charakterystyka** <sub>slajd 16</sub>:
1. **Problem lokalizacji** — adres obiektu zmienia się w czasie.
2. **Problem rozmiaru i struktury przemieszczanej jednostki** — małe obiekty → duży poziom współdzielenia; duże obiekty → mały narzut administracyjny.
3. **Problem migotania** (ang. _trashing_, _ping-pong effect_) — naprzemienne odwołania kilku procesów.

**Zwielokrotnianie — charakterystyka** <sub>slajd 20</sub>:
- **Problem lokalizacji** — tworzenie nowych replik, usuwanie starych replik.
- **Problem rozmiaru i struktury** obiektów zwielokrotnianych.
- **Problem migotania nie występuje** — kopia dla każdego ubiegającego się węzła.
- **Problem spójności kopii (replik)** — stosunek liczby zapisów do odczytów.

---
## Struktura zwielokrotnianej jednostki
<sub>rso_sum_06.pdf, slajdy 21–22</sub>

| Jednostka | Charakterystyka |
|---|---|
| **Strona** | fizyczne połączenie kilku odrębnych obiektów logicznych w jedną jednostkę udostępnianą jako całość przez DSM (**problem fałszywego współdzielenia**) |
| **Pojedyncza zmienna** | duży jednostkowy koszt relokacji i utrzymywania spójności |
| **Obiekt** | hermetyczna struktura danych udostępniana tylko przez zdefiniowane metody — możliwość optymalizacji w strategii utrzymywania spójności w związku ze ściśle określonym sposobem dostępu (poprzez metody) |

![[rso-w6-s22-falszywe-wspoldzielenie.png]]
<sub>Fałszywe współdzielenie — dwa procesy odwołują się do **różnych** obiektów leżących na **tej samej stronie**, co wymusza jej przesyłanie tam i z powrotem. rso_sum_06.pdf, slajd 22</sub>

---
## Protokół koherencji
<sub>rso_sum_06.pdf, slajd 23</sub>

> **Protokół koherencji (spójności)** jest algorytmem rozproszonym realizującym określony model spójności.

1. **Protokół unieważniania** danych (ang. _invalidation protocol_) — małe komunikaty; jednokrotnie unieważnienie.
2. **Protokół aktualizacji** danych (ang. _update protocol_) — niespójne repliki są aktualizowane; większe komunikaty.

---
## Model spójności
<sub>rso_sum_06.pdf, slajdy 24–28</sub>

> **Model spójności** określa gwarancje dotyczące spójności replik, dawane aplikacji (równoległej) przez system.

Pytania: W jaki sposób definiować model spójności? W jaki sposób określić gwarancje dla aplikacji? Kiedy i w jaki sposób egzekwować te gwarancje?

### Spójność ścisła
<sub>slajd 25</sub>

**Spójność ścisła** (ang. _strict consistency_) — każdy odczyt zmiennej $x$ zwraca wartość odpowiadającą wynikowi **ostatnio** wykonanej operacji zapisu.

W systemach rozproszonych:
- **niejednoznaczność określenia _ostatni_** (brak globalnego zegara),
- duży koszt realizacji spójności ścisłej.

### Klasyfikacja modeli spójności replik
<sub>slajdy 26–28</sub>

**Modele spójności nastawione na dane** (danocentryczne):
- modele spójności przy **dostępie ogólnym** — uspójnianie danych przy każdej modyfikacji,
- modele spójności przy **dostępie synchronizowanym** — uspójnianie danych tylko podczas wykonywania jawnych operacji synchronizujących.

**Modele spójności nastawione na klienta** — uwzględnienie mobilności klienta → [[RSO 03 Modele spójności zorientowane na klienta]].

| Dostęp ogólny <sub>(slajd 27)</sub> | Dostęp synchronizowany <sub>(slajd 28)</sub> |
|---|---|
| **Spójność atomowa** (_atomic consistency_) / **liniowość** (_linearizability_) | **Spójność słaba** (_weak consistency_) |
| **Spójność sekwencyjna** (_sequential consistency_) | **Spójność zwalniania** (_release consistency_) |
| **Spójność przyczynowa** (_causal consistency_) | **Spójność wejścia** (_entry consistency_) |
| **Spójność PRAM** (_pipelined RAM consistency_) | **Spójność zakresu** (_scope consistency_) |
| **Spójność podręczna** (_cache consistency_) / **koherencja** (_coherence_) | |
| **Spójność procesorowa** (_processor consistency_) | |

> [!todo] Brak w prezentacjach
> Modele przy **dostępie synchronizowanym** (słaba, zwalniania, wejścia, zakresu) są tylko **wymienione** na slajdzie 28 — **brak definicji i przykładów**. Uzupełnić.

---
## Formalizm

### Podstawowe założenia
<sub>rso_sum_06.pdf, slajd 29</sub>

W skład systemu DSM wchodzą:
- zbiór **sekwencyjnych** procesów $P = \{p_1, p_2, \ldots, p_n\}$,
- zbiór **współdzielonych zmiennych** $X = \{x_1, x_2, \ldots\}$.

Każdy proces ma **własną replikę całego zbioru** $X$. Proces $p_i$ może realizować na zmiennej $x \in X$ operacje:
- **zapisu** wartości $v$, oznaczane $w_i(x)v$,
- **odczytu** wartości $v$, oznaczane $r_i(x)v$.

Realizacja operacji przebiega w **dwóch fazach**: żądanie operacji (ang. _operation issue_) i wykonanie operacji (ang. _operation execution_).

### Oznaczenia
<sub>rso_sum_06.pdf, slajd 30</sub>

| Symbol | Znaczenie |
|---|---|
| $w_i(x)v$ | zapis wartości $v$ do zmiennej $x$ wykonany przez proces $p_i$ |
| $r_i(x)v$ | odczyt wartości $v$ ze zmiennej $x$ wykonany przez proces $p_i$ |
| $O$ | zbiór wszystkich operacji w systemie |
| $O_i$ | zbiór operacji procesu $p_i$ (żądanych przez $p_i$) |
| $OW$ | zbiór wszystkich operacji **zapisu** w systemie |
| $O\vert x$ | zbiór wszystkich operacji **na zmiennej $x$** |
| $\rightarrow_i$ | **lokalny porządek** operacji procesu $p_i$ |
| $\rightarrow$ | **porządek przyczynowy** |
| $\mapsto_i$ | **uszeregowanie** operacje postrzegane są przez proces $p_i$ |

### Definicja historii
<sub>rso_sum_06.pdf, slajd 33</sub>

| Pojęcie | Definicja |
|---|---|
| **Historia lokalna** (procesu $p_i$) | zbiór **liniowo** uporządkowany $h_i = (O_i, \rightarrow_i)$ |
| **Historia globalna** | zbiór **częściowo** uporządkowany $h = (O, \rightarrow)$ |
| **Obraz historii $h$ w procesie $p_i$** | zbiór **liniowo** uporządkowany $hv_i = (O_i \cup OW, \mapsto_i)$ |
| **Obraz historii $h$** | kolekcja obrazów procesów: $hv = \langle hv_1, hv_2, \ldots, hv_n \rangle$ |

### Definicja uszeregowania legalnego
<sub>rso_sum_06.pdf, slajd 32</sub>

> Uszeregowanie $\mapsto_i$ jest **legalne** $\iff$
> $$\forall_{\substack{w(x)v \in OW \\ r(x)v \in O_i}} \left( w(x)v \mapsto_i r(x)v \wedge \nexists_{o(x)u \in O_i \cup OW} [\, u \neq v \wedge w(x)v \mapsto_i o(x)u \mapsto_i r(x)v \,] \right)$$

> [!note] Uwaga z prezentacji
> W celu uproszczenia definicji zakłada się, że **każda operacja zapisu danej zmiennej zapisuje unikalną wartość**, co umożliwia identyfikowanie operacji zapisu poprzez tą wartość.

Intuicyjnie: odczyt zwraca wartość zapisaną przez pewien zapis, a **między** tym zapisem a odczytem nie ma w uszeregowaniu innej operacji na tej samej zmiennej o innej wartości.

> [!todo] Brak w prezentacji — definicja porządku przyczynowego
> Slajd **31** nosi tytuł „Definicja porządku przyczynowego" i zawiera trzy ponumerowane warunki (1), (2), (3), ale są one osadzone jako **pusta grafika** — w pliku PDF nie ma ani czytelnego obrazu, ani warstwy tekstowej dla tych formuł (sprawdzone: `pdfimages` pokazuje stencil 1024×257 o rozmiarze 37 B, `pdftotext` nie zwraca żadnej treści). **Definicji relacji $\rightarrow$ (porządek przyczynowy operacji) nie da się odczytać z prezentacji** — uzupełnić z podręcznika.

---
## Danocentryczne modele spójności przy dostępie ogólnym

W każdym z poniższych modeli obraz $hv$ historii $h$ musi spełniać podane warunki, przy czym **każde uszeregowanie $\mapsto_i$ musi być legalne**.

### Spójność atomowa (liniowość)
<sub>rso_sum_06.pdf, slajdy 38–39</sub>

$$\forall_{o_1, o_2 \in O_i \cup OW} \left( \left( \exists_{j=1..n}\ o_1 \rightarrow_{RT} o_2 \right) \Rightarrow o_1 \mapsto_i o_2 \right)$$
$$\forall_{w_1, w_2 \in OW} \left( \forall_{i=1..n} w_1 \mapsto_i w_2 \ \vee\ \forall_{i=1..n} w_2 \mapsto_i w_1 \right)$$

gdzie $o_1 \rightarrow_{RT} o_2$ oznacza, że **$o_1$ kończy się w czasie rzeczywistym, zanim zaczyna się $o_2$**.

![[rso-w6-s39-atomowa-przyklad.png]]
<sub>Spójność atomowa — przykład. rso_sum_06.pdf, slajd 39</sub>

### Spójność sekwencyjna
<sub>rso_sum_06.pdf, slajdy 34–35</sub>

$$\forall_{o_1, o_2 \in O_i \cup OW} \left( \left( \exists_{j=1..n}\ o_1 \rightarrow_j o_2 \right) \Rightarrow o_1 \mapsto_i o_2 \right)$$
$$\forall_{w_1, w_2 \in OW} \left( \forall_{i=1..n} w_1 \mapsto_i w_2 \ \vee\ \forall_{i=1..n} w_2 \mapsto_i w_1 \right)$$

> [!important] Atomowa vs sekwencyjna
> Jedyna różnica: atomowa wymaga zachowania porządku **czasu rzeczywistego** ($\rightarrow_{RT}$), sekwencyjna tylko porządku **lokalnego procesów** ($\rightarrow_j$). Drugi warunek (wszyscy widzą zapisy w tym samym porządku) jest identyczny.

![[rso-w6-s35-sekwencyjna-przyklad.png]]
<sub>Spójność sekwencyjna — przykład. $hv_1$: $w_2(x)1 \mapsto_1 r_1(x)1 \mapsto_1 w_2(x)2$, $hv_2$: $w_2(x)1 \mapsto_2 w_2(x)2$. rso_sum_06.pdf, slajd 35</sub>

### Spójność przyczynowa
<sub>rso_sum_06.pdf, slajdy 40–41</sub>

$$\forall_{o_1, o_2 \in O_i \cup OW} \left( o_1 \rightarrow o_2 \Rightarrow o_1 \mapsto_i o_2 \right)$$

**Jeden warunek** — zachowanie porządku przyczynowego. Brak wymogu, by wszystkie procesy widziały zapisy w tym samym porządku.

![[rso-w6-s41-przyczynowa-przyklad.png]]
<sub>Spójność przyczynowa — przykład. $hv_1$: $w_1(x)2 \mapsto_1 w_1(y)1 \mapsto_1 w_2(x)1 \mapsto_1 r_1(x)1$, $hv_2$: $w_2(x)1 \mapsto_2 w_1(x)2 \mapsto_2 w_1(y)1 \mapsto_2 r_2(y)1 \mapsto_2 r_2(x)2$. rso_sum_06.pdf, slajd 41</sub>

### Spójność PRAM (pipelined RAM)
<sub>rso_sum_06.pdf, slajdy 43–44</sub>

$$\forall_{o_1, o_2 \in O_i \cup OW} \left( \left( \exists_{j=1..n}\ o_1 \rightarrow_j o_2 \right) \Rightarrow o_1 \mapsto_i o_2 \right)$$

**Jeden warunek** — zachowanie porządku lokalnego procesów (to sam warunek co pierwszy warunek spójności sekwencyjnej, **bez** wymogu wspólnego porządku zapisów).

![[rso-w6-s44-pram-przyklad.png]]
<sub>Spójność PRAM — przykład. $hv_3$: $w_2(x)2 \mapsto_3 r_3(x)2 \mapsto_3 w_1(x)1 \mapsto_3 r_3(x)1 \mapsto_3 w_1(y)1$. rso_sum_06.pdf, slajd 44</sub>

### Spójność podręczna (koherencja)
<sub>rso_sum_06.pdf, slajdy 46–47</sub>

$$\forall_{x \in X} \ \forall_{w_1, w_2 \in OW \cap O\vert x} \left( \forall_{i=1..n} w_1 \mapsto_i w_2 \ \vee\ \forall_{i=1..n} w_2 \mapsto_i w_1 \right)$$

**Jeden warunek** — wszystkie procesy widzą w tym samym porządku zapisy **dotyczące tej samej zmiennej** (spójność sekwencyjna „per zmienna").

![[rso-w6-s47-podreczna-przyklad.png]]
<sub>Spójność podręczna — przykład. rso_sum_06.pdf, slajd 47</sub>

### Spójność procesorowa
<sub>rso_sum_06.pdf, slajdy 50–51</sub>

**PRAM + spójność podręczna** — muszą być spełnione oba warunki:

$$\forall_{x \in X} \ \forall_{w_1, w_2 \in OW \cap O\vert x} \left( \forall_{i=1..n} w_1 \mapsto_i w_2 \ \vee\ \forall_{i=1..n} w_2 \mapsto_i w_1 \right)$$
$$\forall_{o_1, o_2 \in O_i \cup OW} \left( \left( \exists_{j=1..n}\ o_1 \rightarrow_j o_2 \right) \Rightarrow o_1 \mapsto_i o_2 \right)$$

![[rso-w6-s51-procesorowa-przyklad.png]]
<sub>Spójność procesorowa — przykład. $hv_1$: $w_1(x)2 \mapsto_1 w_1(y)1 \mapsto_1 r_1(x)2 \mapsto_1 w_2(x)1 \mapsto_1 r_1(x)1$, $hv_2$: $w_1(x)2 \mapsto_2 w_2(x)1 \mapsto_2 r_2(y)0 \mapsto_2 w_1(y)1 \mapsto_2 r_2(y)1 \mapsto_2 r_2(x)1$. rso_sum_06.pdf, slajd 51</sub>

---
## Protokoły realizujące modele
<sub>rso_sum_06.pdf, slajdy 36–37, 42, 45, 48–49, 52</sub>

### Model sekwencyjny — algorytm `fast-read`
<sub>slajd 36</sub>

```
• upon read(x)                • upon receipt of U(x, v) from p_k
  return M_i[x]                 M_i[x] := v
                                if k = i
• upon write(x, v)                 signal
  atomic_broadcast U(x, v)      end if
  wait
```
Odczyt jest **natychmiastowy** (lokalny), zapis **blokuje** do czasu rozgłoszenia atomowego.

### Model sekwencyjny — algorytm `fast-write`
<sub>slajd 37</sub>

```
• upon read(x)                • upon receipt of U(x, v) from p_k
  if num_i ≠ 0                  M_i[x] := v
     wait                       if k = i
  end if                           num_i := num_i − 1
  return M_i[x]                    if num_i = 0
                                      signal
• upon write(x, v)                 end if
  num_i := num_i + 1            end if
  FIFO_atomic_broadcast U(x, v)
```
Zapis jest **natychmiastowy**, odczyt **blokuje**, dopóki są niepotwierdzone własne zapisy ($num_i \neq 0$).

### Model przyczynowy
<sub>slajd 42</sub>

```
• upon read(x)                • upon receipt of U(x, v) from p_k
  return M_i[x]                 if k ≠ i
                                   M_i[x] := v
• write(x, v)                   end if
  M_i[x] := v
  causal_broadcast U(x, v)
```

### Model PRAM
<sub>slajd 45</sub>

```
• upon read(x)                • upon receipt of U(x, v) from p_k
  return M_i[x]                 if k ≠ i
                                   M_i[x] := v
• upon write(x, v)              end if
  M_i[x] := v
  FIFO_broadcast U(x, v)
```

### Spójność podręczna — `fast-read`
<sub>slajd 48</sub>

```
• upon read(x)                • upon receipt of U(x, v) from p_k
  return M_i[x]                 M_i[x] := v
                                if k = i
• upon write(x, v)                 signal
  atomicx_broadcast U(x, v)     end if
  wait
```

### Spójność podręczna — `fast-write`
<sub>slajd 49</sub>

```
• upon read(x)                • upon on receipt of U(x,v) from p_k
  if num_i[x] ≠ 0               M_i[x] := v
     wait                       if k = i
  end if                           num_i[x] := num_i[x] − 1
  return M_i[x]                    if num_i[x] = 0
                                      signal
• upon write(x, v)                 end if
  num_i[x] := num_i[x] + 1      end if
  FIFOx_atomicx_broadcast U(x,v)
```

### Spójność procesorowa — `fast-write`
<sub>slajd 52</sub>

```
• upon read(x)                • upon receipt of U(x, v) from p_k
  if num_i[x] ≠ 0               M_i[x] := v
     wait                       if k = i
  end if                           num_i[x] := num_i[x] − 1
  return M_i[x]                    if num_i[x] = 0
                                      signal
• upon write(x, v)                 end if
  num_i[x] := num_i[x] + 1      end if
  FIFO_atomicx_broadcast U(x, v)
```

> [!tip] Wzorzec
> Nazwa mechanizmu rozgłaszania w protokole **wprost odpowiada modelowi**: `atomic_broadcast` → sekwencyjna, `causal_broadcast` → przyczynowa, `FIFO_broadcast` → PRAM, `atomicx`/`FIFOx` (indeks x = **per zmienna**) → podręczna i procesorowa. Zob. [[RSO 01 Komunikacja grupowa#Porządki dostarczania]].

---
## Naruszenie porządku przyczynowego
<sub>rso_sum_06.pdf, slajd 53</sub>

![[rso-w6-s53-naruszenie-porzadku.png]]
<sub>Zachodzi $w_1(x)1 \rightarrow w_1(x)2 \rightarrow w_2(y)1$, a $hv_3$: $w_2(y)1 \rightarrow r_3(y)1 \mapsto w_1(x)1 \rightarrow r_3(x)1 \mapsto w_1(x)2 \rightarrow r_3(x)2$ — proces $p_3$ widzi $w_2(y)1$ **przed** $w_1(x)1$, mimo że $w_1(x)1$ przyczynowo poprzedza $w_2(y)1$. rso_sum_06.pdf, slajd 53</sub>

---
## Hierarchia modeli spójności
<sub>rso_sum_06.pdf, slajd 54</sub>

Slajd 54 („Relacje pomiędzy modelami spójności") przedstawia **zagnieżdżone prostokąty** — model wewnętrzny jest **słabszy**:

```
┌─ atomowa ──────────────────────────────────────────┐
│  ∀(o1 →RT o2 ⇒ o1 ↦i o2)  +  wspólny porządek zapisów│
│  ┌─ sekwencyjna ──────────────────────────────────┐ │
│  │  ∀(o1 →j o2 ⇒ o1 ↦i o2) + wspólny porządek zap.│ │
│  │  ┌─ przyczynowa ──────────────────────────────┐│ │
│  │  │  ∀(o1 → o2 ⇒ o1 ↦i o2)                     ││ │
│  │  │  ┌─ PRAM ───────────┐ ┌─ podręczna ───────┐││ │
│  │  │  │ ∀(o1 →j o2 ⇒ …)  │ │ per zmienna x     │││ │
│  │  │  └──────────────────┘ └───────────────────┘││ │
│  │  │            ╰── procesorowa = PRAM + podr. ─╯││ │
│  │  └────────────────────────────────────────────┘│ │
│  └────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────┘
```

$$\text{atomowa} \subset \text{sekwencyjna} \subset \text{przyczynowa} \subset \text{PRAM}$$
$$\text{spójność procesorowa} = \text{PRAM} \cap \text{podręczna}$$

> [!note] Czytelność slajdu 54
> Na slajdzie 54 etykiety „przyczynowa", „PRAM", „podręczna" i „procesorowa" oraz wzory renderują się w prezentacji **bardzo jasno** (niemal niewidocznie). Powyższy schemat i wzory odtworzono z **warstwy tekstowej** pliku PDF.

---
## Zadanie z prezentacji
<sub>rso_sum_06.pdf, slajd 55</sub>

> W jakich modelach spójności wskazana operacja odczytu $r_3(x)v$:
> 1. zwróci wartość $v = 1$,
> 2. zwróci wartość $v = 2$,
> 3. zwróci wartość $v = 3$?

![[rso-w6-s55-zadanie.png]]
<sub>rso_sum_06.pdf, slajd 55</sub>

> [!todo] Brak rozwiązania
> Prezentacja **nie podaje odpowiedzi** do tego zadania — wykład kończy się na slajdzie 55. Rozwiązać samodzielnie / sprawdzić z ćwiczeń.

---
## Braki w prezentacjach

> [!todo] Definicja porządku przyczynowego (slajd 31)
> Formuły są nieczytelne (pusta grafika, brak warstwy tekstowej). Uzupełnić definicję relacji $\rightarrow$ na operacjach.

> [!todo] Modele przy dostępie synchronizowanym
> Spójność słaba, zwalniania, wejścia i zakresu — tylko wymienione (slajd 28), bez definicji.

> [!todo] Spójność ostateczna (eventual consistency)
> Nie występuje w prezentacji w ogóle.

> [!todo] Dowody relacji między modelami
> Slajd 54 pokazuje hierarchię graficznie, ale **bez uzasadnienia** inkluzji.

---
## Powiązania
- Mechanizmy rozgłaszania używane przez protokoły koherencji → [[RSO 01 Komunikacja grupowa]]
- Modele zorientowane na klienta (drugi rodzaj modeli spójności replik) → [[RSO 03 Modele spójności zorientowane na klienta]]
- Porządek przyczynowy zdarzeń (poziom systemowy, nie operacji) → [[RSO 07 Model środowiska przetwarzania#Relacja poprzedzania zdarzeń]]
- Rozgłaszanie przyczynowe i FIFO → [[RSO 01 Komunikacja grupowa#Porządki dostarczania]]
