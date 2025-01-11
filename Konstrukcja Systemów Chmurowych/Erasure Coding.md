---
up: "[[Ceph]]"
tags:
  - KonstrukcjaSystemówChmurowych
---
# Erasure Coding
---
Podział bloków na *k* fragmentów z danymi i *m* fragmentów nadmiarowych. Pozwala to na odporność na awarię m nośników danych. Algorytm pozwala na odtwarzanie z pozostałych nośników nie ma znaczenia czy jest to nośnik k czy m. Przykładowe typowe konfiguracje:
4 + 2
8 + 3
8 + 4

Replikacja na 3 serwerach (m = 3) to ponosimy koszt 3. ( mamy 3 razy więcej danych do przechowania)
konfiguracja 8 + 3 ksozt 8 * 1/8 + 3 * 1/8 = 1,37
8 + 4 koszt 8 * 1/8 + 4 * 1/8 = 1,5

Minus tego rozwiązania moc obliczeniowa, knieczność wyznaczania bloków.

#TODO obrazek z przez