---
tags:
  - SystemyWysokiejNiezawodności
---
# Algorytm Phase-King
---
>Algorytm konsensusu probabilistycznego.
>
>Ogólnie działanie algorytmu polega na $f+1$ **fazach** (_ang. phase_), gdzie w każdej inny process jest koordynatorem, tzw. **królem fazy** (_ang. phase king_). Każda **faza** składa się z dwóch kroków, gdzie w tym drugim **król** odgrywa znaczącą rolę.
## Założenia
- $S^{Sync}\{ \varnothing\}$
- procesy z błędami bizantyjskimi (_ang. Byzantine failures_), gdzie takich procesów $f < \frac{N}{4}$
- niezawodne kanały komunikacyjne
## Algorytm
1. Krok
	- Na początku każdej fazy, każdy proces **rozgłasza** (_ang. broadcast_) wartość którą uznaje za rozwiązanie konsensusu do wszystkich innych procesów i **oczekuje** na wartości rozgłaszane przez inne.
	- Jeśli jakaś wartość występuję częściej jak $\frac{N}{2}$, proces $P_i$ ustawia wartość $majority$ na tę wartość
	- Jeśli żadna wartość nie otrzyma więcej jak $\frac{N}{2}$, wtedy domyślna wartość 0 jest używana jako $majority$
2. Krok
	- **Król fazy**, czyli proces $P_k$ w $k$-tej fazie, **rozgłasza** swoją wartość $majority$ która odgrywa rolę _tie-breaker_-a
	- Kiedy $P_i$ odbiera _tie-breaker_-a, ustawia swoją wartość $v_i$ na wartość **rozgłoszoną** przez **króla fazy**, jeśli ta wartość posiada wagę $w_i \leq \frac{N}{2} + f$
	- Jeśli $w_i$ procesu $P_i$ jest większe od $\frac{N}{2} + f$, ustawia swoją wartość $v_i$ na tą którą ma w zmiennej $majority$

>[!hint]
>Ponieważ w głosach odebranych przez $P_i$ mających wpływ na wynik jego lokalnego $majority$, $f$ głosów może być **fałszywych**, $P_i$ może nie posiadać wyraźnej większości jeśli ma tylko więcej od $\frac{N}{2}$ głosów. W takiej sytuacji przyjmuje _tie-breaker_ **króla fazy**. Aby $majority$ procesu $P_i$ było wyraźną większością potrzebuje ich $\frac{N}{2} + f$.