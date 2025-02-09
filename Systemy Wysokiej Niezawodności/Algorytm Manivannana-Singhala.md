---
tags:
  - SystemyWysokiejNiezawodności
---
# Algorytm Manivannana-Singhala
---
>Jest to algorytm **odtwarzania** stanu przy użyciu **quasi-synchronicznych punktów kontrolnych** (_ang. Quasi-synchronous Checkpointing_).
>
>Zakłada tworzenie punktów kontrolnych:
>- **niezależnych** (_ang. independent_) - zwanych **podstawowymi punktami kontrolnymi** (_ang. basic checkpoints_)
>- **implikowanych komunikacją** (_ang. communication-induced_) - zwanych **wymuszonymi punktami kontrolnymi** (_ang. forced checkpoints_), pomagających przesuwać $RL$ (linię odtwarzania z _ang. recovery line_)
>
>Wybiórczo **pesymistycznie** loguje wiadomości **po stronie odbierającego** (_ang. receiver side_). Upadły proces musi cofnąć się (**asynchronicznie**) tylko do swojego ostatniego punktu kontrolnego, a algorytm zapewnia, że $CP$ jest **spójny** z ostatnim punktem kontrolnym każdego innego procesu. 


## Założenia
- kanały komunikacyjne są zawodne bez uporządkowania komunikatów (_ang. reordering messages_)
- model awarii _fail-recovery_

## Wykorzystywane struktury
- każda wiadomość jest przesyłana wraz (_ang. piggybacked_) z numerem interwału punktu kontrolnego (_ang. checkpoint interval number_) $ckpt\_num_i$. Jest to numer sekwencyjny ostatniego lokalnego _checkpoint_-u $P_i$ - nadawcy wiadomości.
- $next_i$ to numer sekwencyjny, który będzie przypisany do kolejnego **podstawowego** punktu kontrolnego tworzonego przez $C_i$

## Tworzenie punktów kontrolnych
- wartość $next_i$ jest zmieniana co kwant czasu $\Delta t_i$
- **podstawowe punkty kontrolne** są robione co interwał $x \times \Delta t_i$, jeśli nie powstał jeszcze punkt kontrolny dla zadanego $cpkt\_num_i$. W momencie tworzenia punktu kontrolnego wartość $cpkt\_num_i$ jest ustawiana na $next_i$.
- w momencie otrzymania przez $P_i$ wiadomości $m$ o większym $cpkt\_num_j$ niż $cpkt\_num_i$, tworzony jest **wymuszony punkt kontrolny**. Ważne jest to, że jest on **tworzony zanim** wiadomość $m$ zostaje dostarczona do $P_i$. Wraz z utworzeniem **wymuszonego punktu kontrolnego** wartość $cpkt\_num_i$ jest ustawiana na $cpkt\_num_j$ (większą wartość interwału zawartą w otrzymanej wiadomości).


>[!tip]
>Idealnie (ale nie koniecznie) jakby wszystkie kwanty czasu $\Delta t_i$ dla $(i=1..N)$ były jak najbliżej siebie.
>
>Zadaniem $next_i$ jest utrzymanie w miarę równych wartości numerów interwałów wszystkich procesów.

## Algorytm odtwarzanie
- proces $P_f$ który uległ awarii cofa się do swojego ostatniego $cp_f$ który staje się jego **punktem odtwarzania**, od niego dalej może kontynuować przetwarzanie.  Kontroler $C_f$ wysyła do wszystkich pozostałych $C_j$ wiadomość $Rollback(cp_f.ckpt\_num)$ z numerem interwału swojego $rp$.
- proces $P_j$ po otrzymaniu wiadomości $Rollback(rp\_num)$ sprawdza czy jego $ckpt\_num_j$ jest **większy** lub **równy** $rp\_num$:
	- jeśli tak:
		>_(1)_ Szuka najwcześniejszego (najmniejszego) $cp_j$ takiego który jest jednocześnie **większy** lub **równy** $rp\_num$
		_(2)_ Cofa się do tego punktu kontrolnego

	- jeśli nie:
		>_(3)_ Tworzy punkt kontrolny $cp_j$ z $ckpt\_num$ równym $rp\_num$, ustawia swój $ckpt\_num_j$ na ten otrzymany we wiadomości $Rollback$ i kontynuuje przetwarzanie. Jest to podobne do $vcp_i$ (**wirtualnego punktu kontrolnego**).

>[!note] Obserwacja
>wszystkie procesy są odtwarzane od najwcześniejszego _checkpoint_-u który spełnia $cp_i.ckpt\_num \geq rp\_num$ _(1)_ ponieważ wszystkie albo cofnęły się _(2)_ lub kontynuują przetwarzanie _(3)_ od tego $rp\_num$.
>
>Algorytm jest poprawny ponieważ punkty kontrolne od których procesy wznowiły przetwarzanie tworzą $CP^\bullet$, a co za tymi idzie $RL$.


> [!check] Claim C1:
Jeśli $P_i$ zostanie cofnięty do $cp_i^{x_i}$ po otrzymaniu $Rollback(rp\_num)$ od $C_f$, to:
> 1. $x_i \geq rp\_num$, ale
> 2. wszystkie punkty kontrolne wykonane przez $C_i$ przed $cp_i^{x_i}$ mają numery interwałów mniejsze niż $rp\_num$.

> [!check] Claim C2:
Dla każdej wiadomości $m$ wysłanej przez $P_i$:
$$send_i(\overrightarrow{P_i, P_j}, m) \in cp_i^{x_i} \iff m.ckpt\_num < x_i$$

> [!check] Claim C3:
Dla każdej wiadomości $m$ otrzymanej przez $P_i$:
$$recv_i(\overleftarrow{P_i, P_j}, m) \in cp_i^{x_i} \implies m.ckpt\_num < x_i$$

> [!check] Claim C4:
Dla każdego $i$: $C_i$ dostarcza otrzymaną wiadomość $m$ do $P_i$ tylko po tym, jak wykonał punkt kontrolny $cp_i^{x_i}::x_i \geq m.ckpt\_num$

>[!info] Twierdzenie
>Zakładając, że po otrzymaniu $Rollbak(r)$ od $C_f$ proces $P_i$ jest przywracany do $cp^{x_i}_i$, wtedy zbiór $CP = {cp^{x_1}_1,...,cp^r_f,...,cp^{x_i}_i,...,cp^{x_N}_N}$ tworzy poprawny $RL$ 

## Garbage collection
Po otrzymaniu wiadomości $Rollback(rp\_num)$ i wykonaniu należytych instrukcji przywrócenia stanu do odpowiedniego punktu kontrolnego, usuwane są wszystkie punkty kontrolne które miały miejsce przed tym do którego przywrócony został proces.
## Pełen algorytm odtwarzania
#TODO 