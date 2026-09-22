---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 44
---
# 44. Programowanie równoległe w Cilk – model obliczeń (DAG) i algorytm szeregowania
---
> **Cilk** (MIT, C. Leiserson i in., od 1994) to rozszerzenie języka C/C++ o kilka słów kluczowych do programowania **równoległego z wątkami**, o modelu **fork-join** (dziel i zwyciężaj). Programista wskazuje, **co może** być wykonane równolegle. **System uruchomieniowy** z **algorytmem szeregowania z kradzieżą pracy** (_work stealing_) decyduje, **jak** rozdzielić zadania na procesory, z gwarancjami wydajności.

**Historia**: Cilk-1 (1994), **Cilk-5** (1998, PLDI – „work-first principle”), Cilk++ (Cilk Arts, 2008), **Intel Cilk Plus** (2010; w GCC 4.9–7, usunięty w GCC 8 i wycofany przez Intela w 2017), **OpenCilk** (MIT, 2019 – na Tapir/LLVM). Idea wpłynęła na Intel TBB, Java Fork/Join, .NET TPL, OpenMP tasks, Go.

## Słowa kluczowe
| Cilk-5 | Cilk Plus / OpenCilk | Znaczenie |
|---|---|---|
| `spawn f(x)` | `cilk_spawn f(x)` | wywołanie funkcji, które **może** wykonać się **równolegle** z kodem wywołującego (kontynuacją) |
| `sync` | `cilk_sync` | czekaj na zakończenie **wszystkich** funkcji uruchomionych przez `spawn` w bieżącej funkcji |
| `cilk` (przed definicją funkcji) | – | oznaczenie funkcji Cilk |
| – | `cilk_for` | pętla z równoległymi iteracjami (rekurencyjny podział zakresu) |
| `SYNCHED`, `abort`, `inlet` | – | zaawansowane (Cilk-5) |

- Na końcu każdej funkcji jest **niejawny `sync`** – funkcja nie wraca, dopóki nie zakończą się jej potomne.
- **Elizja szeregowa** (_serial elision_): usunięcie słów kluczowych (`cilk_spawn` → nic, `cilk_sync` → nic, `cilk_for` → `for`) daje **poprawny program sekwencyjny** o tej samej semantyce (jeśli program jest wolny od wyścigów determinacji). Ułatwia to testowanie i wnioskowanie.
- Uruchomienie `spawn` **nie gwarantuje** równoległości – to pozwolenie dla planisty.

### Przykład: Fibonacci
```c
int fib(int n) {
    if (n < 2) return n;
    int x = cilk_spawn fib(n - 1);   // potomek – może działać równolegle
    int y = fib(n - 2);              // kontynuacja wykonuje się w bieżącym wątku
    cilk_sync;                        // czekaj na x
    return x + y;
}
```
### Przykład: sortowanie przez scalanie i pętla
```c
void mergesort(int *A, int n) {
    if (n < THRESHOLD) { serial_sort(A, n); return; }   // próg – granulacja
    cilk_spawn mergesort(A, n / 2);
    mergesort(A + n / 2, n - n / 2);
    cilk_sync;
    merge(A, n);                    // scalanie może też być równoległe (p-merge)
}

cilk_for (int i = 0; i < n; ++i)
    C[i] = A[i] + B[i];
```

## Model obliczeń – acykliczny graf skierowany (DAG)
Wykonanie programu Cilk modeluje się jako **DAG obliczeń** $G = (V, E)$:
- **Wierzchołki** – **pasma** (_strands_): maksymalne sekwencje instrukcji **bez** słów kluczowych równoległości (`spawn`, `sync`, powrót). Każdy wierzchołek to jednostka pracy (koszt = czas wykonania, w najprostszym modelu 1).
- **Krawędzie** – zależności wykonania:
  - **krawędź spawn** – od pasma wywołującego do pierwszego pasma potomka,
  - **krawędź kontynuacji** (_continuation_) – do następnego pasma tej samej funkcji po `spawn`,
  - **krawędź wywołania** (_call_) – zwykłe wywołanie funkcji,
  - **krawędź powrotu** (_return_) – od ostatniego pasma funkcji do pasma po `sync` w rodzicu.
- Pasma $u$ i $v$ są **szeregowe** (_in series_), gdy istnieje ścieżka między nimi, i **równoległe** (_in parallel_), gdy nie istnieje – wtedy mogą wykonać się jednocześnie.
- DAG jest **seryjno-równoległy** (_series-parallel_) – powstaje przez szeregowe i równoległe składanie podgrafów; jego struktura zależy od danych wejściowych, ale **nie od planisty**.

```
fib(4):                   ● fib(4) pasmo przed spawn
                        ↙spawn  ↘kontynuacja
            ● fib(3)              ● fib(2) (wywołanie)
          ↙       ↘               ↙      ↘
      fib(2)     fib(1)       fib(1)    fib(0)
        ...        ...          ...       ...
                        ↘  powroty  ↙
                          ● po sync: return x+y
```

## Miary wydajności
- $T_P$ – czas wykonania na $P$ procesorach.
- **Praca** (_work_) $W = T_1$ – łączny koszt wszystkich wierzchołków (czas na 1 procesorze).
- **Rozpiętość** (_span_, głębokość, _critical-path length_) $S = T_\infty$ – koszt **najdłuższej ścieżki** w DAG (czas na nieskończonej liczbie procesorów).

**Prawa**:
- **Prawo pracy** (_work law_): $T_P \geq \dfrac{T_1}{P}$ – $P$ procesorów wykonuje co najwyżej $P$ jednostek pracy w jednostce czasu,
- **Prawo rozpiętości** (_span law_): $T_P \geq T_\infty$ – skończona liczba procesorów nie może być szybsza niż nieskończona.

**Pochodne**:
- **Przyspieszenie** (_speedup_) na $P$ procesorach: $T_1 / T_P \leq P$; **liniowe**, gdy $= \Theta(P)$; **idealne**, gdy $= P$; **superliniowe** ($> P$) niemożliwe w tym modelu (w praktyce przez efekty cache).
- **Równoległość** (_parallelism_): $T_1 / T_\infty$ – **maksymalne możliwe przyspieszenie**; dla $P$ większego niż równoległość dodatkowe procesory nie pomagają.

**Kompozycja**:
| | Praca | Rozpiętość |
|---|---|---|
| szeregowo $A$ potem $B$ | $W_A + W_B$ | $S_A + S_B$ |
| równolegle $A \parallel B$ | $W_A + W_B$ | $\max(S_A, S_B)$ |

### Analiza przykładów
- **fib(n)**: $W(n) = W(n-1) + W(n-2) + \Theta(1) = \Theta(\varphi^n)$, $S(n) = \max(S(n-1), S(n-2)) + \Theta(1) = \Theta(n)$ → **równoległość** $\Theta(\varphi^n / n)$ – ogromna.
- **cilk_for** dla $n$ iteracji po $\Theta(1)$ (rekurencyjny podział na pół): $W = \Theta(n)$, $S = \Theta(\log n)$ → równoległość $\Theta(n / \log n)$.
- **mergesort** ze scalaniem szeregowym: $W = \Theta(n \log n)$, $S(n) = S(n/2) + \Theta(n) = \Theta(n)$ → równoległość tylko $\Theta(\log n)$ (**wąskim gardłem** jest szeregowe scalanie – prawo Amdahla); z **równoległym scalaniem** $S = \Theta(\log^3 n)$ → równoległość $\Theta(n / \log^2 n)$.
- **Prawo Amdahla** jako szczególny przypadek: jeśli ułamek $f$ pracy jest szeregowy, to $T_\infty \geq f \cdot T_1$ i przyspieszenie $\leq 1/f$.

## Algorytm szeregowania – kradzież pracy (_randomized work stealing_)
### Struktury
- $P$ **wątków roboczych** (_workers_) – po jednym na procesor.
- Każdy worker ma **dwustronną kolejkę** (_deque_) **ramek** (zadań/kontynuacji gotowych do wykonania).

### Działanie (Cilk-5, „najpierw praca”)
1. **`spawn f()`**: worker **odkłada kontynuację** (resztę bieżącej funkcji) na **spód** (_tail_) **swojej** kolejki i **natychmiast wykonuje potomka** `f` (_child-first / work-first_). To jak zwykłe wywołanie funkcji – jeśli nikt nie ukradnie, wykonanie jest identyczne z sekwencyjnym.
2. **Powrót z potomka**: worker **zdejmuje ze spodu** swojej kolejki (LIFO) kontynuację i ją wykonuje (jeśli wciąż tam jest).
3. **Pusta kolejka**: worker staje się **złodziejem** – wybiera **losowo** ofiarę spośród innych workerów i **kradnie z wierzchu** (_head_) jej kolejki (FIFO) **najstarszą** kontynuację. Jeśli kolejka ofiary jest pusta – losuje ponownie.
4. **`sync`**: jeśli żaden potomek nie został ukradziony – nic nie robi (szybka ścieżka). W przeciwnym razie worker czeka na zakończenie ukradzionych potomków, a w tym czasie **sam kradnie** pracę (nie blokuje się bezczynnie – _leapfrogging_ / zawieszenie ramki).

```
worker 1 (zajęty)          worker 2 (bezczynny, kradnie)
┌──── wierzch (najstarsze) ◀────────── kradzież (FIFO)
│ kont. fib(4)  ← duży kawałek pracy
│ kont. fib(3)
│ kont. fib(2)
└──── spód (najnowsze) ◀── push/pop właściciela (LIFO)
```

### Dlaczego z wierzchu?
Najstarsze kontynuacje leżą **najwyżej w drzewie rekurencji** → reprezentują **największe** porcje pracy. Jedna kradzież daje złodziejowi dużo zajęcia, więc **kradzieże są rzadkie**, a narzut komunikacji i synchronizacji mały. Właściciel operuje na spodzie – dobra **lokalność cache** (jak stos wywołań).

### Zasada „najpierw praca” (_work-first principle_, Cilk-5)
> Minimalizuj narzut ponoszony przez **pracę** ($T_1$), nawet kosztem zwiększenia narzutu **kradzieży**.

Uzasadnienie: liczba kradzieży jest ograniczona przez $O(P \cdot T_\infty)$, a praca jest proporcjonalna do $T_1$ ≫. Implementacja:
- kompilator generuje dwie wersje każdej funkcji: **szybki klon** (_fast clone_ – wykonywany zwykle, prawie bez narzutu, `sync` jako no-op) i **wolny klon** (_slow clone_ – po kradzieży, obsługuje wznawianie ramki na innym procesorze),
- ramki alokowane na stercie, ale tanio; kolejka przez leniwe operacje bez blokad (blokady tylko przy kradzieży),
- narzut `spawn` w Cilk-5 ≈ 2–6× koszt zwykłego wywołania.

### Gwarancje teoretyczne (Blumofe, Leiserson 1994/1999)
Dla programu w pełni ścisłym (_fully strict_ – `sync` tylko na potomkach własnej funkcji) na $P$ procesorach:
- **Czas**: oczekiwany $\;E[T_P] \leq \dfrac{T_1}{P} + O(T_\infty)$ → gdy $P \ll T_1 / T_\infty$ (równoległość), przyspieszenie jest **prawie liniowe**; z dużym prawdopodobieństwem $T_P \leq T_1/P + O(T_\infty + \log P + \log(1/\varepsilon))$.
- **Pamięć**: $S_P \leq P \cdot S_1$, gdzie $S_1$ to pamięć stosu wykonania sekwencyjnego – każdy worker potrzebuje co najwyżej tyle co program szeregowy.
- **Komunikacja**: oczekiwana $O(P \cdot T_\infty \cdot (1 + n_d) \cdot S_{max})$ ($n_d$ – maks. liczba odwołań do danych, $S_{max}$ – maks. rozmiar ramki) – ograniczona przez rozpiętość, nie przez pracę.
- Liczba kradzieży: oczekiwana $O(P \cdot T_\infty)$.

Losowość wyboru ofiary zapewnia równomierne rozprowadzenie pracy i odporność na złośliwe wzorce.

### Porównanie z _work sharing_
W **dzieleniu pracy** (_work sharing_) zajęte wątki **aktywnie oddają** zadania innym przy każdym tworzeniu, co daje stały narzut. W **kradzieży pracy** bezczynne wątki same **zabierają** zadania – przy pełnym obciążeniu brak migracji i niski narzut.

## Wyścigi determinacji i detekcja
- **Wyścig determinacji** (_determinacy race_): dwa **równoległe** pasma odwołują się do tej samej lokacji pamięci i co najmniej jedno zapisuje → wynik zależy od planisty. Szczególny przypadek to wyścig danych (bez blokad).
```c
int x = 0;
cilk_spawn x++;   // wyścig: oba pasma równoległe modyfikują x
x++;
cilk_sync;        // x ∈ {1, 2}
```
- **Detektory**:
  - **Nondeterminator** (algorytm **SP-bags**, Feng-Leiserson 1997) – w jednym wykonaniu **szeregowym** zapisuje dla każdej procedury zbiory S (szeregowe) i P (równoległe) w strukturze union-find; dla każdego dostępu sprawdza, czy poprzedni dostęp był w zbiorze P → wyścig. Działa w czasie prawie liniowym i wykrywa **wszystkie** wyścigi możliwe dla danego wejścia (niezależnie od przeplotu),
  - **Cilkscreen** (Intel Cilk Plus), **Cilksan** (OpenCilk).
- **Unikanie wyścigów**:
  - **reduktory** (_reducers_, hiperobiekty) – zmienne z asocjacyjną operacją łączenia (np. `reducer_opadd<long> sum`), każdy worker ma lokalny widok, łączone w kolejności szeregowej → wynik jak w elizji szeregowej, bez blokad,
  - blokady (mutexy) – kosztem skalowalności,
  - **notacja tablicowa** Cilk Plus (`A[:] = B[:] + C[:]`) i `#pragma simd` – wektoryzacja.

## Podsumowanie
| Element | Cilk |
|---|---|
| Model | fork-join, zadania w DAG seryjno-równoległym |
| Konstrukcje | `cilk_spawn`, `cilk_sync`, `cilk_for`, reduktory |
| Semantyka | elizja szeregowa = program sekwencyjny |
| Miary | praca $T_1$, rozpiętość $T_\infty$, równoległość $T_1/T_\infty$ |
| Planista | losowa kradzież pracy, kolejki deque, work-first |
| Gwarancje | $T_P \leq T_1/P + O(T_\infty)$, $S_P \leq P S_1$ |
| Narzędzia | detektory wyścigów (SP-bags), profilery skalowalności (Cilkview) |

## Zobacz też
- [[09 Wielozadaniowość i synchronizacja zadań i wątków]] – OpenMP, pthreads
- [[45 Modele obliczeń współbieżnych - model aktorów i rachunek pi]]
