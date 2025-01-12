---
tags:
  - KonstrukcjaSystemówChmurowych
up:
---
# Gluster FS
---
>**Gluster FS** to rozproszony system plików działający w przestrzeni użytkownika. Skalowany liniowo do petabajtów i tysięcy klientów. **Metadane** są w pełni rozproszone, każdy węzeł może zlokalizować dane bez odpytywania innych. Oznacza to, że nie mamy jednego węzła z informacjami o tym gdzie znajdują się jakie dane, co powoduje brak **SPoF** (*Single Point Of Failure*). Dostęp do niego jest poprzez FUSE czy jak jest to zalecane aplikację korzystającą z `libgfapi`. Alternatywnie można połączyć się poprzez NFS, SMB, REST czy HDFS lecz to wymaga podłączenia się do jakiegoś węzła, co sprawia, że ów węzeł stał by się **SPoF**. Na jego bazie powstał [[Ceph]].

### Zalety
- prosty
- samo-korygujący
- serwery przechowujące dane mogą korzystać z dowolnych technologii, nie ma znaczenia jak składowane są dane
### Wady
- brak optymalizacji dla dysków SSD
- brak mechanizmów kontroli dostępu (tylko IP)

## Architektura
![[Pasted image 20250111181405.png]]

>Sieć jest warunkiem koniecznym by osiągnąć wysoką skalowalność.

## Podstawowe pojęcia
**Trusted Storage Pool** - dynamiczna kolekcja serwerów składujących dane
**Brick** - pojedynczy katalog udostępniany przez węzeł
**Volume** - logiczna kolekcja **bricks**, która może być zamontowana

## Typy woluminów
**Distributed** - pliki są rozproszone pomiędzy *node*-ami w zależności od wartości *hash*-a pliku.
**Stripped** - części plików mogą być rozproszone pomiędzy *node*-ami, przechowywane jako *sparse files*, może obsługiwać bardzo duże pliki - przekraczające wielkość dysku fizycznego.
**Replicated** - pliki są przechowywane na wielu *node*-ach. (*transactions* + *changelogs*) → HA
**Distributed replication** - pliki są przechowywane podobnie jak RAID 10, rozpraszane pomiędzy dwie pary dysków, które następnie je replikują
**Stripped replication** - podobnie jak **distributed replication** ale z fragmentami plików, (RAID 01)
**Distributed Stripped Replication** - pliki są dzielone na fragmenty, fragmenty są replikowane na serwerach, ale na których zależy od algorytmu dystrybucji (angażuje 8 serwerów)

## Zarządzanie wolumenami
Dynamiczne operacje:
- dodanie "cegiełek" do wolumenów
- usuwanie "cegiełek" do wolumenów
- *rebalancing* rozłożenia danych na woluminie
- podmiana "cegiełki" w woluminie

## Geo-replication
| Regular replication                                                         | Geo-replication                                                                                         |
| --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Mirrors data accross clusters                                               | Mirrors data accros geographically distributed clusters                                                 |
| Provides high-avilability                                                   | Ensures backing up of data for disaster recovery                                                        |
| Synchronous replication (każdy plik jest wysyłany do wszystkich **bricks**) | Asynchronous replication (sprawdza zmiany w plikach co jakiś czas i synchronizuje je po wykryciu zmian) |
