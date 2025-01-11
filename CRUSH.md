---
up: "[[Ceph]]"
---
# CRUSH
---
**Controlled Replication Under Scalable Hashing**
-  lokalizacja obiektów jest wyliczana przez klienta nie odczytywana z centralnego serwera
- algorytm uwzględnia struktórę fizyczną systemu rozproszonego (dyski, węzły, racks, switches, zasilanie)
- możliwość definiowania failure zones
- samo-zarządzalność
- autokorekta
	- wykrywanie awarii
	- odtawrzanie danych

## Działanie
Pobiera z serwera MON (ceph-mon) *cluster-map*
object -> hash -> placement groups -> primary OSD


## Hierarchia
#TODO obrazek z przez
