---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 39
---
# 39. Algorytm heurystyczny do wykrywania sytuacji wyścigu wysokiego poziomu (_high-level data race_)
---
> **Wyścig danych wysokiego poziomu** (Artho, Havelund, Biddle – „High-Level Data Races”, 2003) to błąd współbieżności, w którym **każdy pojedynczy dostęp** do zmiennych współdzielonych jest **poprawnie chroniony** blokadą, a mimo to **grupa logicznie powiązanych zmiennych** jest odczytywana lub modyfikowana w **oddzielnych sekcjach krytycznych**. Inny wątek może zaobserwować lub utworzyć **niespójny stan** tej grupy. Klasyczne detektory (Eraser – [[38 Algorytm Eraser]]) takiego błędu nie zgłoszą.

## Motywujący przykład – współrzędne
```java
class Coord {
    private int x, y;

    // wątek T1: zapis obu współrzędnych atomowo
    public void swap() {
        synchronized (this) {
            int tmp = x; x = y; y = tmp;     // pole x i y w JEDNYM bloku
        }
    }

    // wątek T2: odczyt współrzędnych w DWÓCH blokach
    public int[] read() {
        int rx, ry;
        synchronized (this) { rx = x; }      // blok 1: {x}
        // <-- tu T1 może wykonać swap()
        synchronized (this) { ry = y; }      // blok 2: {y}
        return new int[] { rx, ry };         // para (rx, ry) może nie istnieć w żadnym stanie!
    }
}
```
- Każdy dostęp do `x` i `y` jest pod blokadą `this` → **brak klasycznego wyścigu danych** (Eraser: $C(x) = C(y) = \{this\}$).
- T1 traktuje `x` i `y` jako **jedną całość** (aktualizuje je atomowo), a T2 czyta je **osobno** → może otrzymać parę złożoną z połowy starego i połowy nowego stanu.
- To naruszenie **atomowości** operacji logicznej, a nie wyścig pojedynczej komórki.

## Pojęcia
- **Blok chroniony** – fragment kodu wykonany przy trzymaniu blokady $l$ (sekcja `synchronized`).
- **Widok** (_view_) bloku chronionego – **zbiór pól współdzielonych** (zmiennych), do których odwołuje się (czyta lub zapisuje) wątek w tym bloku:
  $$v = \{ x \mid x \text{ jest używane w bloku chronionym przez } l \}$$
  Widoki są zbierane **dynamicznie** z wykonania (lub statycznie z kodu).
- $V(t)$ – **zbiór widoków** wątku $t$ (wszystkie widoki bloków wykonanych przez $t$).
- **Widok maksymalny** – widok wątku $t$, który **nie jest podzbiorem** innego widoku tego wątku:
  $$M(t) = \{ m \in V(t) \mid \neg\exists\, v \in V(t): m \subsetneq v \}$$
  Intuicja: widok maksymalny opisuje **zbiór zmiennych, które wątek traktuje jako powiązane** (używa ich razem w jednej sekcji krytycznej).
- **Łańcuch** – zbiór zbiorów **całkowicie uporządkowany** relacją zawierania: dla dowolnych $a, b$ w łańcuchu $a \subseteq b$ lub $b \subseteq a$.

## Zgodność widoków (_view consistency_)
**Definicja (zgodność wątków)**: wątek $t_1$ jest **zgodny** z wątkiem $t_2$ ⇔ dla **każdego** widoku maksymalnego $m \in M(t_1)$ zbiór przecięć
$$\{\, m \cap v \mid v \in V(t_2) \,\}$$
tworzy **łańcuch**.

**Definicja (zgodność programu)**: program (wykonanie) jest **zgodny co do widoków**, gdy **każda para wątków** jest wzajemnie zgodna. **Brak zgodności** = **potencjalny wyścig wysokiego poziomu**.

**Intuicja**: $t_1$ uważa zmienne z $m$ za powiązane. Jeśli $t_2$ dostęp do nich **dzieli** na części, które się nie zawierają (np. osobno $\{x\}$ i osobno $\{y\}$), to $t_2$ może widzieć niespójne połączenie stanu, który $t_1$ aktualizuje atomowo. Jeśli natomiast przecięcia są „zagnieżdżone” (łańcuch: np. $\{x\} \subseteq \{x, y\}$), wątek $t_2$ zawsze obejmuje większą część grupy w jednym bloku i nie „rozcina” jej niezgodnie.

### Zastosowanie do przykładu
- $V(T_1) = \{\{x, y\}\}$, więc $M(T_1) = \{\{x, y\}\}$.
- $V(T_2) = \{\{x\}, \{y\}\}$.
- Przecięcia $m = \{x,y\}$ z widokami $T_2$: $\{x\}$ i $\{y\}$ – **żaden nie zawiera drugiego** → **nie jest łańcuchem** → $T_1$ **niezgodny** z $T_2$ → **ostrzeżenie**.

### Przykłady zgodne
- $T_2$ z widokami $\{x\}$ i $\{x, y\}$: przecięcia $\{x\} \subseteq \{x,y\}$ → łańcuch → zgodne (T2 raz czyta tylko $x$, a raz całość – nie rozcina grupy na rozłączne części).
- $T_2$ z jednym widokiem $\{x, y\}$ → zgodne.
- Wątki operujące na rozłącznych zbiorach (przecięcia puste) → $\emptyset$ jest podzbiorem wszystkiego → zgodne.

## Algorytm (heurystyka)
1. **Instrumentacja / monitorowanie** wykonania programu (np. w maszynie wirtualnej Java PathFinder lub JNuke): przy wejściu do bloku `synchronized` tworzony jest nowy, pusty widok; każde odwołanie do pola współdzielonego dodaje je do widoków **wszystkich** aktualnie otwartych bloków wątku; przy wyjściu widok dopisywany do $V(t)$.
2. Dla każdego wątku obliczenie **widoków maksymalnych** $M(t)$.
3. Dla każdej pary wątków $(t_1, t_2)$ i każdego $m \in M(t_1)$: obliczenie przecięć $m \cap v$ dla $v \in V(t_2)$ i **sprawdzenie, czy tworzą łańcuch** (posortowanie wg rozmiaru i sprawdzenie zawierania kolejnych).
4. Zgłoszenie **naruszeń zgodności widoków** wraz z polami i miejscami w kodzie.

Własności:
- analiza **niezależna od konkretnego przeplotu** (jak lockset) – wystarczy, że każdy wątek wykona swoje bloki, nawet sekwencyjnie,
- złożoność wielomianowa względem liczby widoków,
- zwykle rozważa się blokady niezależnie (widoki per blokada) lub łączy wszystkie blokady.

## Dlaczego heurystyka – fałszywe alarmy i przeoczenia
- **Fałszywe alarmy**: zmienne w jednym bloku z przypadku (niepowiązane logicznie) → „sztuczna” grupa; wątek, który celowo czyta część stanu (np. tylko rozmiar kolekcji do statystyk).
- **Przeoczenia**:
  - błędy atomowości niewyrażalne widokami: **odczyt, a potem zapis** w dwóch blokach tej samej zmiennej (_stale value_: `synchronized {v = x;} ... synchronized {x = v + 1;}`) – widoki $\{x\}, \{x\}$ są zgodne, a jest utracona aktualizacja,
  - błędy zależne od semantyki (niezmienniki) niewynikające z grupowania pól,
  - nieprzećwiczone ścieżki (analiza dynamiczna).
- Zgodność widoków **nie gwarantuje poprawności**, a jej brak **nie zawsze oznacza błąd** – to heurystyka oparta na założeniu, że programista konsekwentnie grupuje powiązane zmienne.

## Powiązane problemy i narzędzia
- **Wyścig danych (niskiego poziomu)** – [[38 Algorytm Eraser]]; wyścig wysokiego poziomu go uzupełnia.
- **Naruszenia atomowości** – bardziej ogólne podejścia:
  - **Atomizer** (Flanagan-Freund) – sprawdzanie, czy bloki oznaczone `atomic` są **serializowalne** (typy przemienności ruchów Liptona: operacje prawo-/lewostronnie przemienne),
  - **AVIO** – wzorce nieserializowalnych przeplotów dostępów (np. R–W–R),
  - **stale-value errors** (Artho, Havelund, Biddle) – użycie wartości wyniesionej z sekcji krytycznej po jej opuszczeniu,
  - **linearyzowalność** jako kryterium poprawności obiektów współbieżnych – [[Linearizability]].
- Implementacje algorytmu zgodności widoków: rozszerzenie **Java PathFinder** ([[40 Sprawdzanie modelu - Java Pathfinder]]), **JNuke**; wbudowane w detektory w badaniach nad Javą.
- **Remedium**: obejmowanie operacji na powiązanych zmiennych **jedną** sekcją krytyczną (lub zwracanie **niezmiennej migawki** – obiektu wartości), obiekty niezmienne, pamięć transakcyjna ([[41 Pamięć transakcyjna]]).

```java
public synchronized int[] read() { return new int[] { x, y }; }   // jeden blok = spójny widok
```
