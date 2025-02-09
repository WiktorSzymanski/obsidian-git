---
tags:
  - SystemyWysokiejNiezawodności
---
# Algorytm Paxos
---
> Algorytm konsensusu zaproponowany przez Lamporta.
> 
> Zamysł algorytmu polega na akceptacji wartości **proponowanej** przez proces (_ang. proposal_) jeśli jest **zaakceptowana** (_ang. accepted_) przez większość procesów. Jeśli jakaś **propozycja** jest już **zaakceptowana** nie może zostać zmieniona. Każda **propozycja** ma przypisany **unikatowy** numer $pn_i$. Tylko **propozycja** z największym numerem $pn_i$ wygrywa, to znaczy jest **akceptowana**.

## Założenia
- $S^{Async}\{uniqID\}:pn_i = i$ - System asynchroniczny rozbudowany o mechanizm unikatowych identyfikatorów procesów
- model awarii _fail-stop_, gdzie $f < \frac{N}{2}$ to znaczy co najmniej $M = \left\lceil \frac{N+1}{2} \right\rceil$ procesów jest poprawnych
- niezawodne kanały komunikacyjne

>[!hint]-
>$S$ oznacza system ($S^{Sync}$ - synchroniczny, $S^{Async}$ - asynchroniczny), a poprzez $S\{M\}$ oznaczamy system $S$ rozbudowany o dodatkowy mechanizm.

>[!attention]-
>- W przypadku błędów zaniechania (_ang. omission failures_) w kanałach komunikacyjnych, należy zastosować **rozgłaszanie** przez _Reliable Broadcast_
>- Paxos może zostać zastosowany dla modelu awarii _fail-recovery_, jednak wtedy system musi zostać poszerzony o $S^{Async}\{stable storage\}$, $P_i$ musi zapisywać w **pamięci trwałej** jakie $v_k$ **zaakceptowało** lub na jaki największy $pn_k$ odesłało $OK$.
## Algorytm
1. Proces $P_i$ **rozgłasza** (_ang. broadcasts_) komunikat $PREPARE$ z wartością $v_i$ i numerem $pn_i$.
2. Każdy proces $P_j$ odpowiada na $PREPARE$:
	- $OK$ jednocześnie gwarantując, że nie **zaakceptuje** żadnej innej **propozycji** $PREPARE$ z numerem mniejszym od $pn_i$, który właśnie otrzymał.
	- $ACCEPTED$ z proponowanym $v_k$ którą proces $P_j$ już **zaakceptował**. W tym wypadku $pn_k$ może być mniejsze od $pn_i$, ale jako, już **zaakceptował** to **nie może się już wycofać**.
3. Jeśli $P_i$ odbierze odpowiedzi ($OK$ lub $ACCEPTED$) od większości $M$ procesów:
	- Jeśli odebrał jakiekolwiek $ACCEPTED$, ustawia $v_i$ na równe $v_k$ tej wiadomości $ACCEPTED$ o największym $pn_k$, oraz ustawia $pn_i = pn_k$.
	- **rozgłasza** komunikat $PROPOSE$ z wartościami $v_i$ oraz $pn_i$
4. Gdy $P_j$ odbiera $PROPOSE$ z $v_i$ i $pn_i$, **akceptuje** go jeśli wcześniej nie odpowiedział na $PREPARE$ z większym numerem $pn_k$
5. Jeśli $P_j$ **zaakceptuje** wartość $v_i$, **rozgłasza** komunikat $ACCEPTED$ z $vi$ i $pn_i$
6. Kiedy $P_i$ odbierze $ACCEPTED$ z tą samą wartością $v_j$ od $M$ procesów, **podejmuje decyzję** $v_j$

## Zakończenie (_ang. Termination_) 
W Paxos-ie z $S^{Async}\{ \varnothing\}$ i błędami zaniechania nie mamy gwarancji zakończenia. Pare sytuacji może prowadzić do tzw. _livelock_:
- proces "wygrywający" krok _3._ mający właśnie wysłać $PROPOSE$ upada
- jeśli proces powróci (by powtórzyć potencjalne $PROPOSE$) z podniesioną wartością $pn_i$, może nigdy nie dotrzeć do końca (nie ma absolutnego "zwycięscy" kroku _3._)

Aby zagwarantować progres w $S^{Async}\{ \varnothing\}$, można wybrać konkretny proces jako jedyny, który będzie mógł rozsyłać $PROPOSE$ (w kroku _3._), co wymaga $S^{Async}\{Election\}$ (takie rozwiązanie z jednym procesem podejmującym decyzje oryginalnie zaproponował Lamport). Istnieje również rozwiązanie z wieloma, równoległe działającymi instancjami Paxos-a, a działanie samego algorytmu się nie zmienia.