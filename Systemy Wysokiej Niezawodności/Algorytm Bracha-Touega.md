---
tags:
  - SystemyWysokiejNiezawodności
---
# Algorytm Bracha-Touega
---
>**Probabilistyczny algorytm binarnego konsensusu** (_ang. Probabilistic binary consensus algorithm_) będący implementacją podejścia **Las Vegas**.
>
>Ogólne działanie algorytmu polega na początkowym wyborze przez każdy proces losowo wartości 0 lub 1 jako ich $v_i$ z **wagą** $w_i = 1$. Każdy proces który jeszcze **nie podjął decyzji** określa nową wartość $v_i$ i wagę $w_i$ na podstawie pierwszych $N-f$ wiadomości które otrzymał tej rundy. **Waga** szacuje liczbę procesów które zagłosowały $v=v_i$ w poprzedniej rundzie.

## Założenia
- $S^{Async}\{ \varnothing\}$
- model awarii procesów _fail-stop_, gdzie $f < \frac{N}{2}$
- niezawodne kanały komunikacyjne

## Algorytm
- proces $P_i$ **rozsyła** (_ang. broadcast_) komunikat $<k_i, v_i, w_i>$ do wszystkich, łącznie ze sobą
- $P_i$ oczekuje komunikatu $<k_i, v_i, w_i>$ od $N-f$ procesów
	- jeśli przychodząca wiadomość $<k_j, v_j, w_j>$ posiada wagę $w_j>\frac{N}{2}$ to proces $P_i$ ustawia wartość $v_i = v_j$, a $w_i$ ustawiane jest na ilość komunikatów głosujących na $v=majority(v_j)$
	- jeśli więcej jak $f$ (powyżej lub równo $\frac{N}{2}$) przychodzących wiadomości posiada wagę $w_j>\frac{N}{2}$, proces $P_i$ **podejmuje decyzję** $v_i$ i **rozsyła** $<k+1, v_i, N-f>$ i $<k+1, v_i, N-f>$ po czym **kończy działanie**  
	- jeśli waga przychodzących wiadomości $w_j \leq \frac{N}{2}$ wtedy wartość $v_i = majority(v_j)$ (jest wyliczana przez funkcję wyznaczającą większość), a $w_i$ ustawiane jest na ilość komunikatów głosujących na $v=majority(v_j)$
- Kiedy jakiś proces **zadecyduje**, wszystkie inne **poprawne** procesy gwarantują zagłosowanie w ciągu **dwóch rund**

## Poprawność
- Jeśli $P_i$ czekało by na więcej jak $N-f$ komunikatów w $k$-tej rundzie,  mogło by to doprowadzić do **zagłodzenia** jeśli $f$ procesów by upadło.
- Jeśli $P_i$ otrzymuje komunikaty $<k, v_x, w_x>$ i $<k_i, v_y \neq v_x, w_y>$, wtedy $w_x + w_y \leq N$, więc $w_x$ i $w_y$ nie mogą być na raz większe od $\frac{N}{2}$.
- Finalnie skoro $P_i$ czeka na $N-f$ komunikatów $<k, v_j, w_j>$ i może zadecydować tylko jeśli więcej jak $f$ komunikatów będzie miało **wagę** większą od $\frac{N}{2}$, jest to istotne aby $N-f > f$, więc $f < \frac{N}{2}$ jest wymagane. 

> [!info] Twierdzenie
>Dla $f < \frac{N}{2}$, algorytm konsensusu Bracha-Touega jest algorytmem Las Vegas który **kończy działanie** (_ang. terminates_) z prawdopodobieństwem 1.

>[!warning]- Dowód _Validity_
>Procesy nie mogą **podjąć innych decyzji**:
>- Załóżmy, że $P_i$ zadecydował $v$ w $k$-tej rundzie
>- Wtedy w $k$-tej rundzie proces $P_i$ otrzymał $v_j=v$ i $w_j>\frac{N}{2}$ od więcej jak $f$ otrzymanych wiadomości
>- Stąd w rundzie $k$ każdy poprawny proces otrzymał przynajmniej jedną wiadomość $<k, v, w>$ z $w>\frac{N}{2}$
>- Stąd w rundzie $k+1$ wszystkie poprawne procesy zagłosują $v$
>- Stąd w rundzie $k+2$ wszystkie poprawne procesy zagłosują $v$ z $w=N-f$
>- Co doprowadza do tego, że na końcu rundy $k+2$ wszystkie poprawne procesy **podejmą** tę samą **decyzję** o wartości $v$.

>[!warning]- Dowód _Termination_
> Algorytm zakończy działanie:
> - Ponieważ kanały komunikacyjne są niezawodne, istnieje pewne prawdopodobieństwo $p > 0$ na to, że w każdej rundzie $k$ wszystkie procesy odbiorą jako pierwsze $N-f$ wiadomości od tych samych procesów 
> - Po tej rundzie $k$ wszystkie poprawne procesy będą posiadały te samą wartość $v$
> - Po rundzie $k+1$ wszystkie poprawne procesy będą posiadały te samą wartość $v$ z wagą $w=N-f$
> - Po rundzie $k+2$ wszystkie poprawne procesy **zadecydują** na wartość $v$
> - Stąd algorytm **zakończy się** z prawdopodobieństwem 1 