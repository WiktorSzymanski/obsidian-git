---
tags:
  - SystemyWysokiejNiezawodności
---
# Algorytm Juanga-Venkatesana
---
> Jest to algorytm **odtwarzania** stanu przy użyciu **asynchronicznych punktów kontrolnych** (_ang. Independent Checkpointing_).
> 
> Generalny koncept algorytmu to wyszukać taki $rp_i$ z dokładnością do pojedynczych zdarzeń komunikacyjnych, dla którego nie ma osieroconych wiadomości, wykrywając je poprzez porównanie ilości odebranych wiadomości z ilością wysłanych.
> 
> Stosuje (PRAWDOPODOBNIE) **optymistyczne** rejestrowanie logów po stronie **nadawcy** i **odbiorcy**

## Założenia
- kanały są niezawodne z uporządkowaniem komunikatów
- kanały mają nieskończoną pojemność
- przetwarzanie jest fragmentarycznie deterministyczne (_ang. piecewise deterministic_)

## Wykorzystywane struktury
- $rp_i$ - punkt przywracania (_ang. recovery point_) $i$-tego procesu
- **$SENT_{i \rightarrow j}(rp_i)$** - ilość wiadomości wysłanych do momentu $rp_i$ przez $P_i$ do $P_j$ 
- **$RCVD_{i \leftarrow j}(rp_i)$** - ilość wiadomości odebranych do momentu $rp_i$ przez $P_i$ od $P_j$
- Przy każdym zdarzeniu **wysłania** lub **odbierania wiadomości** proces zapamiętuje tę wiadomość w **logu**

Wartości tych struktur są **zapisywane w pamięci trwałej** razem z $cp_i$

## Algorytm odtwarzania
- Wykonanie algorytmu przez $C_i$ rozpoczyna się gdy $P_i$ jest restartowany lub gdy $C_i$ dowiaduje się o restarcie innego $P_f$.
- Inicjacja polega na wyznaczeniu początkowej wartości $rp_i^1$.
- W $k$-tej iteracji ($k>1$) kontroler $C_i$ wykorzystując $rp_i^{k-1}$ wyznaczony w iteracji $k-1$ (w poprzedniej iteracji), oblicza wartości $SENT_{i \rightarrow j}(rp_i^{k-1})$ dla każdego sąsiedniego procesu $P_j$, wysyłając ją następnie w komunikacie $ROLLBACK$.
- Przetwarzając komunikaty $ROLLBACK$ odebrane od sąsiadów, odnajduje taki $rp_i^k$ aby dla każdego sąsiedniego procesu $P_j$: $$RCVD_{i \leftarrow j}(rp_i^k) = SENT_{j \rightarrow i}(rp_i^{k-1})$$Wyjątkiem może być pierwsza iteracja w **procesie restartowanym** gdzie warunek między zmiennymi może być $\leq$ zamiast $=$.

>[!tip]
>- Kontroler $C_i$ może dowiedzieć się o restarcie innego $P_f$ ponieważ w momencie jego **restartu po awarii**, kontroler $C_f$ informuje o tym pozostałe procesy **rozsyłając broadcast**.
>
>- Algorytm jest inicjowany jednocześnie przez wszystkie kontrolery procesów.
>- Algorytm wykonuje się w $N$ iteracjach („rundach”), gdzie $N$ jest liczbą procesów.
>- **Złożoność komunikacyjna** algorytmu to  $O(|E|)$, gdzie $|E|$ jest całkowitą liczbą łączy komunikacyjnych.
>- Po zakończeniu każdej rundy, co najmniej jeden proces wycofa się do swojego ostatecznego lokalnego punktu odtwarzania, za wyjątkiem przypadków, gdy aktualnie wyznaczone punkty odtwarzania już stanowią spójną linię odtwarzania.

