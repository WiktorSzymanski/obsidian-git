---
tags:
  - SystemyRozproszoneDużejSkali
---
# NoSQL
---
## Charakterystyka
- nie relacyjne
- zwykle open-source
- ukierunkowane na rozproszenie
## Unikają
- narzucania własności [[ACID]]
- składni języka SQL
## Zapewniają
- skalowalność
- częste i łatwe zmiany schematu
- pozwala na przechowywanie dużych ilości danych

## Rodzaje modeli danych noSQL
### Key / value
- kolekcja par klucz / wartość
- natura przechowywanych wartości jest transparentna dla bazydanych - [[Schemaless]]
- #TODO

### Column-Oriented
- podobne do key/value, ale wartość może mieć wiele atrybutów (kolumn)
- **kolumna** - grupa danych wartości pewnego typu
- przechowuje i procesuje dane jako kolumny nie rzędy
- #TODO 

### Document
- podobne do *column-oriented*
- #TODO 

### Graph
#TODO

## Modele dystrybucji
- Są dwie techniki dystrybucji danych:
	- [[Replikacja]] - bierze te same dane i kopiuje je po wielu węzłach (ang. *node*)
		- **Master-slave**
		- **Peer-to-peer**
	- [[Sharding]] - dane dzielimy na porcje i różne porcje danych są rozkładane po węzłach
- Single server
	- brak dystybucji
	- #TODO