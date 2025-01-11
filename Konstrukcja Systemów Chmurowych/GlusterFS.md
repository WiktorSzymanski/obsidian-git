---
tags:
  - KonstrukcjaSystemówChmurowych
up:
---
# Gluster FS
---
>**Gluster FS** to rozproszony system plików działający w przestrzeni użytkownika. Skalowany liniowo do petabajtów i tysięcy klientów. Metadane są w pełni rozproszone ... #TODO 

### Zalety
- prosty
- samo-korygujący
### Wady
- brak optymalizacji dla dysków SSD
- #TODO

## Architektura
#TODO obrazek z prez

>Sieć jest warunkiem koniecznym by osiągnąć wysoką skalowalność.

## Podstawowe pojęcia
**Trusted Storage Pool** - dynamiczna kolekcja serwerów składujących dane
#TODO 

## Typy woluminów
**Distributed** - pliki są rozproszone pomiędzy *node*-ami w zależności od wartości *hash*-a pliku.
**Stripped** - części plików mogą być rozporszone pomiędzy *node*-ami, przechowywane jako *sparse files*, może obsuługiwać bardzo duże pliki - przekraczające wielkość dysku fizycznego.
**Replicated** - pliki są przechowywane na wielu *node*-ach. (*transactions* + *changelogs*) → HA
**Distributed replication** #TODO
**Stripped replication**
**Distributed Stripped Replication**

## Zarządzanie wolumenami
Dynamiczne operacjie:
- dodanie "cegiełek" do wolumenów
- usuwanie "cegiełek" do wolumenów
- *rebalancing* rozłożenia danych na woluminie
- podmiana "cegiełki" #TODO 

## Medatane
**Metadane** są rozproszone, a każdy węzeł może zlokalizować dane bez odpytywania innych węzłów.

## Geo-replication
| Regular replication           | Geo-replication                                         |
| ----------------------------- | ------------------------------------------------------- |
| Mirrors data accross clusters | Mirrors data accros geographically distributed clusters |
| Provides high-avilability     | Ensures backing up of data for disaster recovery        |
| Synchronous replication       | #TODO                                                   |
|                               |                                                         |
#TODO wywalić ten wiersz