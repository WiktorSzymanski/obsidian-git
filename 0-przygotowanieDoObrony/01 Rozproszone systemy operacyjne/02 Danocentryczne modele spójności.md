---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 2
---
# 2. Danocentryczne modele spójności
---
> **Model spójności** to kontrakt między systemem (replikowaną pamięcią/bazą) a klientem. Określa, jakie wartości mogą zwracać odczyty przy współbieżnych zapisach na wielu replikach. Modele **danocentryczne** (_data-centric_) nakładają ograniczenia na porządek operacji widziany **przez wszystkie procesy**, niezależnie od tego, z którą repliką komunikuje się klient.

Wprowadzenie i klasyfikacja: [[Algorytmy Rozproszone/Model Spójności]].

## Formalizm
- **Operacje**: $w_i(x)v$ – proces $P_i$ zapisuje wartość $v$ do obiektu $x$; $r_i(x)v$ – $P_i$ odczytuje z $x$ wartość $v$.
- **Historia** $H$ – zbiór wszystkich operacji wykonanych w systemie wraz z porządkiem ich wykonania.
- **Historia lokalna** $H|P_i$ – podciąg operacji procesu $P_i$ w kolejności wykonania.
- **Uszeregowanie** $S$ – liniowy (totalny) porządek pewnego zbioru operacji.
- **Uszeregowanie legalne** – każdy odczyt $r(x)v$ w $S$ zwraca wartość **ostatniego poprzedzającego go zapisu** $w(x)v$ w $S$ (lub wartość początkową, gdy zapisu nie ma).
- **Relacja przyczynowości operacji** $o_1 \rightarrow o_2$: $o_1$ poprzedza $o_2$ w historii lokalnej tego samego procesu, **albo** $o_1 = w(x)v$ i $o_2 = r(x)v$ (odczyt zwrócił wartość tego zapisu), plus domknięcie tranzytywne. Zapisy nieporównywalne są **współbieżne**.

## Modele (od najsilniejszego)
### Spójność atomowa (_atomic_, _strict_, ≈ linearyzowalność)
Istnieje **jedno legalne uszeregowanie** $S$ wszystkich operacji $H$, zgodne z **porządkiem czasu rzeczywistego** (jeśli $o_1$ zakończyła się przed rozpoczęciem $o_2$, to $o_1$ przed $o_2$ w $S$).
- Odczyt zawsze zwraca wynik najnowszego zapisu – system zachowuje się jak pojedyncza kopia.
- Wymaga globalnej synchronizacji (blokady / konsensus) – kosztowna. Zob. [[Linearizability]].

### Spójność sekwencyjna (Lamport, 1979)
Istnieje **jedno legalne uszeregowanie** $S$ wszystkich operacji, które zachowuje **porządek lokalny każdego procesu**: $\forall_i\ H|P_i$ jest podciągiem $S$.
- Wszystkie procesy „widzą” **ten sam** przeplot, ale nie musi on odpowiadać czasowi rzeczywistemu.
- Implementacja: rozgłaszanie totalne zapisów (zob. [[01 Komunikacja grupowa]]).

### Spójność przyczynowa (Ahamad i in., 1995)
Dla **każdego procesu** $P_i$ istnieje legalne uszeregowanie $S_i$ zbioru: operacje $P_i$ oraz **wszystkie zapisy** (wszystkich procesów), zachowujące **porządek przyczynowy** $\rightarrow$.
- Zapisy **powiązane przyczynowo** widziane w tej samej kolejności wszędzie.
- Zapisy **współbieżne** mogą być widziane przez różne procesy w różnej kolejności.
- Implementacja: rozgłaszanie przyczynowe zapisów (zegary wektorowe).

### Spójność procesorowa – PRAM (_Pipelined RAM_, Lipton-Sandberg, 1988), zwana też FIFO
Dla **każdego procesu** $P_i$ istnieje legalne uszeregowanie $S_i$ zbioru: operacje $P_i$ oraz wszystkie zapisy, zachowujące **porządek lokalny każdego procesu** (zapisy jednego procesu widziane w kolejności, w której zostały wykonane).
- Zapisy **różnych** procesów mogą być widziane w dowolnej kolejności, nawet jeśli są powiązane przyczynowo.
- Implementacja: rozgłaszanie FIFO zapisów – każdy proces jakby miał „potok” (_pipeline_) zapisów od każdego innego.

### Koherencja / spójność słaba
> [!warning] Zweryfikować z wykładem
> W vaultcie termin „koherencja” pojawia się tylko jako „protokół spójności lub koherencji” ([[Algorytmy Rozproszone/Model Spójności]]). Poniżej standardowe znaczenia z literatury (Tanenbaum/Mosberger); sprawdzić, który model prowadząca nazywała koherencją.

- **Koherencja** (_coherence_) – w literaturze DSM pojęcie równoważne spójności: **zbieżność stanu replik**, tzn. wszystkie repliki po zakończeniu aktualizacji mają ten sam stan. Protokół koherencji utrzymuje repliki w zgodzie z przyjętym modelem.
- **Spójność słaba** (_weak consistency_) – zmiany propagowane do replik dopiero w jawnych punktach synchronizacji (np. zakończenie sekcji krytycznej). Między nimi repliki mogą się różnić.
- **Spójność ostateczna** (_eventual_) – przy braku nowych zapisów wszystkie repliki w końcu zbiegną się do tego samego stanu. Zob. [[46 Ostateczna spójność - CRDT i typy chmurowe]].
- **Spójność wejścia** (_entry consistency_, Munin/Orca) – odczyt spójny, jeśli proces trzyma blokadę związaną z danym obiektem.

## Hierarchia
$$\text{atomowa} \Rightarrow \text{sekwencyjna} \Rightarrow \text{przyczynowa} \Rightarrow \text{PRAM (procesorowa)} \Rightarrow \text{ostateczna}$$
Każdy silniejszy model spełnia słabszy; im słabszy, tym mniejszy koszt synchronizacji i większa dostępność.

## Przykłady historii ($x$ początkowo 0)
### Sekwencyjna, ale nie atomowa
```
P1: w1(x)1                  (kończy się w t=1)
P2:           r2(x)0        (wykonany w t=2)
```
$S = r_2(x)0,\ w_1(x)1$ jest legalne i zachowuje porządki lokalne ⇒ **sekwencyjna**. W czasie rzeczywistym zapis zakończył się przed odczytem ⇒ **nie atomowa**.

### Przyczynowa, ale nie sekwencyjna
```
P1: w1(x)1
P2: w2(x)2                   (współbieżnie z w1)
P3:        r3(x)1   r3(x)2
P4:        r4(x)2   r4(x)1
```
- $S_3 = w_1(x)1,\ r_3(x)1,\ w_2(x)2,\ r_3(x)2$; $S_4 = w_2(x)2,\ r_4(x)2,\ w_1(x)1,\ r_4(x)1$ – legalne, zapisy współbieżne ⇒ **przyczynowa**.
- Jedno wspólne uszeregowanie wymagałoby $w_1 < w_2$ (dla $P_3$) i $w_2 < w_1$ (dla $P_4$) ⇒ **nie sekwencyjna**.

### PRAM, ale nie przyczynowa
```
P1: w1(x)1
P2:        r2(x)1  w2(x)2
P3:                         r3(x)2  r3(x)1
```
- $w_1(x)1 \rightarrow r_2(x)1 \rightarrow w_2(x)2$, więc $w_1 \rightarrow w_2$. $P_3$ widzi 2, a potem 1 ⇒ **naruszona przyczynowa**.
- PRAM dla $P_3$: $S_3 = w_2(x)2,\ r_3(x)2,\ w_1(x)1,\ r_3(x)1$ – zapisy różnych procesów w dowolnej kolejności, porządki lokalne zachowane ⇒ **PRAM spełniony**.

Ćwiczenia z zajęć: [[modele_spojnosci_zad_dom.excalidraw]], [[AlgorytmyRozproszoneCwiczenia.excalidraw]].

## Porównanie
| Model | Liczba uszeregowań | Co musi być zachowane | Implementacja |
|---|---|---|---|
| Atomowa | 1 wspólne | czas rzeczywisty | blokady globalne, konsensus |
| Sekwencyjna | 1 wspólne | porządki lokalne | rozgłaszanie totalne |
| Przyczynowa | po 1 na proces | porządek przyczynowy | rozgłaszanie przyczynowe |
| PRAM | po 1 na proces | porządki lokalne (FIFO) | rozgłaszanie FIFO |

## Zobacz też
- [[03 Modele spójności zorientowane na klienta]]
- [[Algorytmy Rozproszone/Replikacja]]
- [[Linearizability#8. Porównanie właściwości porządkowania]]
