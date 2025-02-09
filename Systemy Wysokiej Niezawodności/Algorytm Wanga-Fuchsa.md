---
tags:
  - SystemyWysokiejNiezawodności
---
# Algorytm Wanga-Fuchsa
---
>  Jest to algorytm **odtwarzania** stanu przy użyciu **asynchronicznych punktów kontrolnych** (_ang. Independent Checkpointing_).
>  
>  Jego założeniem jest wykorzystanie _checkpoint z-dependency graph_ do wykrywania osieroconych wiadomości. Wszystkie wiadomości są tagowane dodatkowymi informacjami kontrolnymi (_z-dependency tracking_). Algorytm wykorzystuje **optymistyczne logowanie wiadomości** (_ang. optimistic message logging_) po stronie **odbierającego**. Niektóre dane kontrolne są zapisywane w pamięci stałej wraz z punktami kontrolnymi i logowanymi wiadomościami.
## Założenia
- model awarii _fail-recovery_
- kanały są niezawodne bez uporządkowania komunikatów (_ang. reordering messages_)
- brak partycjonowania sieci
- przetwarzanie **deterministyczne**
- wszystkie akcje mogą zostać wycofane, wykorzystuje _output commit_ jeśli to konieczne

## Wykorzystywane struktury
- **$ckpt\_num_i$** - numer interwału punktu kontrolnego procesu $P_i$
- **$send\_num_i$** - numer sekwencyjny wiadomości wysłanej przez proces $P_i$
- **$send\_log_i$** - zawiera elementy ($send\_num_i$, $j$) gdzie $P_j$ jest procesem docelowym wiadomości $m$ wysłanej przez $P_i$ z liczbą sekwencyjną $send\_num_i$. Zawiera więc pary "jaka wiadomość została wysłana do jakiego procesu"
- **$rec\_log_i$** - zawiera elementy ($m$, $send\_num_j$, $ckpt\_num_j$, $j$) opisujące odebraną wiadomość $m$, a więc zawiera informacje, że odebrano wiadomość $m$ o numerze sekwencyjnym $send\_num_j$, wysłaną przez proces $P_j$ podczas jego $ckpt\_num_j$ interwału.

## Tworzenie punków kontrolnych
Kiedy kontroler $C_i$ spontanicznie i niezależnie od innych zadecyduje aby utworzyć punkt kontrolny procesu $P_i$,  przechowuje aktualny stan procesów wraz z $send\_log_i$, $rec\_log_i$ i $ckpt\_num_i$. Następnie czyści ulotne dzienniki $send\_log_i$ i $rec\_log_i$.

## Algorytm odtwarzania
Podczas odtwarzania poprzez upadły proces $P_f$ globalny graf punktów kontrolnych (_ang. global checkpoint graph_) całego przetwarzania musi zostać utworzony. $C_f$ żąda od pozostałych kontrolerów by wysłały do niego informacje _z-dependency_. Kontroler $C_i$ który otrzymał takie żądanie, tworzy **wirtualny punkt kontrolny** (_ang. virtual checkpoint_) i odpowiada na żądanie aktualnymi wartościami $send\_log_i$, $rec\_log_i$ i $ckpt\_num_i$. Kiedy $C_f$ otrzyma wszystkie odpowiedzi, tworzy **rozszerzony graf punktów kontrolnych** (_ang. extended checkpoint graph_).

![[Pasted image 20250206104601.png]]

### Rollback edge
Ponieważ **algorytm Wanga-Fuchsa** wykorzystuje **optymistyczne logowanie wiadomości**, może zaistnieć taka sytuacja, że proces który doznał awarii nie zdążył zapisać wiadomości do logu. (Na powyższym przykładzie: wiadomość $m1$ nie została zapisana) Jeśli zaistniała taka sytuacja tworzony jest **rollback edge** reprezentujący **wiadomość** jeszcze **nie zapisaną do logu** (na grafie reprezentowana czerwoną kreskowaną linią).

![[Pasted image 20250206105251.png]]
Węzeł do którego przychodzi **rollback edge** jest potencjalną niespójnością punktów kontrolnych, stąd każdy węzeł do którego przychodzi **rollback edge** jak i wszystkie inne węzły do których można się dostać z tego węzła są wykluczane z grafu.
![[Pasted image 20250206105817.png]]

## Algorytm wyznaczania linii odtwarzania
1. Każdy ostatni punkt kontrolny danego procesu znajduje się w **root set**.
2. Wszystkie punkty kontrolne do których można dotrzeć z punktu kontrolnego znajdującego się w **root set** są oznaczane.
3. Jeśli jakiś punkt kontrolny z **root set**-u jest oznaczony:
	- wymień każdy oznaczony _checkpoint_ z **root set**-u na ostatni, nie oznaczony _checkpoint_ tego procesu
	- Powróć do kroku _2_.
4. Gdy żaden punkt kontrolny z **root set**-u nie jest oznaczony, **root set** tworzy $RL$ (Linię odtwarzania z _ang. Recovery Line_)
![[Pasted image 20250206110551.png]]

## Garbage collection
Jego celem jest pozbycie się zbędnych _checkpoint_-ów.

- **discardable checkpoints** - to takie które **nigdy** nie będą częścią, żadnej przyszłej **linii odtwarzania** .
- **obsolete checkpoints** - poprzedzające **linię odtwarzania**, która została by użyta w najgorszym wypadku.
Punkty kontrolne typu **obsolete** są też **discardable** ale niektóre **non-obsolete** punkty również mogą być **discardable**.

![[Pasted image 20250206120951.png]]

> [!info] Twierdzenie
>Niech $N$ będzie liczbą procesów, a $Ĝ$ nadgrafem _checkpoint graph_-u $G$. Punkt kontrolny w $G$ jest _non-discardable_ jeśli należy do połączonych (_ang. Union_) linii odtwarzania  wszystkich $Ĝ-vcp_i$ gdzie  $(1\leq i\leq N)$.

>[!cite] Definicja
**Nadgraf $Ĝ$** - to graf $G$ sztucznie rozszerzony o wszystkie wirtualne punkty kontrolne.

Prościej mówiąc, twierdzenie to oznacza, że _checkpoint_ jest _non-discardable_ jeśli przynależy do którejkolwiek z $N$ linii odtwarzania, powstałych na skutek awarii któregokolwiek z procesów, wchodzących w skład grafu $G$.

![[Pasted image 20250206122822.png]]![[Pasted image 20250206122842.png]]