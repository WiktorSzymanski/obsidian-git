---
tags:
  - SystemyWysokiejNiezawodności
---
# Algorytm Manetho
---
>Jest to algorytm **odtwarzania** stanu przy użyciu **hybrydowych punktów kontrolnych** (_ang. Hybrid Checkpointing_).
>
>Koncept jego działania to wykorzystanie przyczynowego logowania (_ang. causal logging_) i grafu poprzedzania przyczynowego (_ang. antecedence graph_).

## Założenia
- model awarii _fail-recovery_
- kanały są zawodne bez uporządkowania komunikatów (_ang. reordering messages_)
- przetwarzanie jest fragmentarycznie deterministyczne (_ang. piecewise deterministic_), oznacza to, że składa się z sekwencji deterministycznych interwałów stanu, każdy rozpoczęty niedeterministycznym zdarzeniem
- wszystkie niedeterministyczne zdarzenia (_ang. nondeterministic events_) są zapisywane do logu **optymistycznie**
- w razie kontaktu ze "światem zewnętrznym" stosowany jest _output commit_ 

## Tworzenie punktów odtwarzania
Podobnie jak w przypadku **asynchronicznych punktów kontrolnych**, punkty kontrolne są tworzone spontanicznie i niezależnie od pozostałych procesów.

>[!hint] Przesyłanie grafu poprzedzania przyczynowego
>Nie służy do tworzenia punktów odtwarzania, ale jest konieczne w trakcie działania przetwarzania rozproszonego, aby wraz z wiadomościami pomiędzy procesami był wysyłany _antecedence graph_. 

## Algorytm odtwarzania
Gdy dany proces $P_f$ ulega awarii i odtwarza się na postawie swojego ostatniego punktu kontrolnego, poprawne procesy wysyłają do niego swoje **grafy poprzedzania przyczynowego**. Na ich podstawie proces $P_f$ może odtworzyć w swoim stanie zagubione wiadomości, które miały miejsce po _checkpoint_-cie od którego odtworzył swój stan.
###### space-time diagram![[Pasted image 20250206135404.png]]
###### antecedence graph![[Pasted_image_20250206134728.png]]