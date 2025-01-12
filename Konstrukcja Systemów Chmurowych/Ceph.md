---
tags:
  - KonstrukcjaSystemówChmurowych
up:
---
# Ceph
---
## Podstawowe własności
- skalowalność (*scale-out*)
- niezawodność, brak SPoF (*Single Point of Failure*)
- pracuje na zwykłych komputerach
- efektywne przechowywanie zarówno dużych jak i licznych małych plików (cf. HDFS)
- *software-defined storage*

## Interfejsy dostępu
**block storage** → SAN, interfejs blokowy, klient otrzymuje urządzenie reprezentujące dysk logiczny
**file storage** → NAS, montujemy strukturę katalogową
**object storage** → płaski zbiór porcji danych jednoznacznie identyfikowanych. (worek z plikami bez żadnej hierarchii). Ten typ dostępu daje nam największą wydajność dostępu do plików

## Komponenty
![[Pasted image 20250111183835.png]]

[[RADOS]] - Reliable Autonomic Distributed Object Store
**RBD** - RADOS Block Device, protokół będący odpowiednikiem iSCSI, dający dostęp do urządzeń blokowych.
**RADOS GW** - RADOS Gateway, pozwala na dostęp do danych przez interfejs REST-owy
**CAPH FS** - możliwość montowania systemu plików standardowymi protokołami (NFS, SMB)
[[OSD]] - Object Storage Device
**MON** - Monitor
**MDS** - Metadata Server, może byś SPoF w przypadku block storage i file storage

> Dedykowana biblioteka `librados` pozwala w pełni wykorzystać możliwości **Ceph**-a.

## Skalowalność
**Pojemność**
- dodatowe dyski
- większe dyski
- dodatkowe dyski
**Wydajność**
- dodatkowe dyski
- dodatkowe serwery
- dobry projekt sieci
- dyski SSD (mogą działać jako *cache* do często pobieranych plików)
- równoważne obciązenie

## Ceph journal
- wstępny zapis do logu (journal)
- przepiswyanie do docelowej lokalizacji
- cel przyśpieszenie zapisu i spójność
- ciągły obszar zapisu dla małych plików 
- wykorzystywanie mechanicmu [[CoW]] systemu Btrfs
- #TODO 

## Placement Groups
Algorytm rozpraszania CRUSH

#TODO obrazek z prez

**PG** - **Placement Groups** grupują obiektuy, jest ich tyle ile replik. Pojedyńczy serwer OSD obsługuje pomiędzy 50-100 **PG**. Jeśli każde dane mają być replikowane na wielu serwerach to primary przesyła do secondary, secondary to tritiary itd.
#TODO

## Ceph Pool
- Zbiór placement groups
- taki wolumin
- określa rodzaj i liczebność replik
- możliwość tworzenia #TODO 

## Ceph Rebalancing
- 300 s nieaktywności OSD -> awaria, po tym czasie odpala się struktura reorganizacji danych
- odtwarzanie danych
- podobny proces po dodaniu nowych serwerów, wyrównanie obłożenia dysków
- waga OSD decyduje o ilości przesyłanych danych na dany serwer OSD

## Replikacja Ceph
- Elastycznie konfigurowalna na poziomie plików/katalogów
- Dynamiczna rekonfiguracja
- Rozproszenie replik na różnych serwerach -> eliminacja przeciążeń po awarii
- Nie ma potrzeby posiadania dedykowanych dysków zapasowych
- Możliwość stosowania dysków o różnych pojemnościach
- Możliowść [[Erasure Coding]] #TODO 

## Ceph monitor
Nadzoruje stan całego klastra.
stan:
- serwerów OSD
- placement groups (PG)
- #TODO