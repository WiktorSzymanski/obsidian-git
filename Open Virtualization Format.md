---
class: KSCh
---
# Open Virtualization Format
---
> Otwarty, bezpieczny, wydajny i rozszerzalny format dla reprezentacji maszyn wirtualnych. Jego podstawowe coechy to:
> - Optymalizacja...
> - #TODO 
> - Przenośność (*portability from origin*)
> 	- Level 1: związanie z konkretnym systemem
> 	- Level 2: wsparcie dla określonej pratformy sprzętowej (np. Xen HVM)
> 	- Level 3: wsparcie dla wielu platform sprzętowych (automatyczne wykrywanie i konfigurowanie sprzętu)
> - Obsługa wielu języków
> - OVF zawiera: #TODO 

Tworzenie OVF:
- instalacja wiwrtualnego systemu(ów) na wirtualnym dychku(ach)
- kodowanie wietualnych dysków (np. kompresja)
- przygotowanie #TODO 

- Przejście od wirtualnych maszyn do wirtualnych aplikacji
- Okrojony system operacyjny (JeOS) + aplikacja 
	- SUSE Linux Enterprise JeOS - ok. 100MiB
	- openSUSE MicroOS (Tumbleweed based - rolling distribution)
- Wygoda instalacji, przenoścność, łatwiejsze zarządzanie
- Standaryzacja, konsolidacja
- Problem: aktualizacja VA z zachowaniem konfiguracji

## Wirtualizacja zagnieżdżona 
- Maszyna wirtualna pracująca w ramach maszyny wirutalnej. Konieczne w momencie wykupienia maszyny wirtualnej i chęci dalszej wirtualizacji.
- Wydajność
- Warianty:
	- kontener -> kontener
	- VM -> kontener
	- VM -> VM
- Emulacja sprzętowego wsparcia dla wirtualizacji. Początkowo wspierane tylko przez profesjonane produkty takie jak ESXi, teraz przeszło do mniejszych takich jak VirtualBox

## Inne podejście do wirtualizacji
Proces virtual machine
- Java Virtualm Machine
- .Net Framework
- Kompilacja JIT

Klastry - wirtualizacja komunikacji:
- PVM
- MPI
- => middleware

## VMware NSX
- Wirtualizacja sieci
	- logiczne porty, przełączniki, rutery
	- logiczne ściany ogniowe (Firewall)
	- logiczne sieci VLAN, VPN
- Zalety
	- szybkie wdrażanie
	- automatyzacja
	- elastyczność
	- NSX RESTful API

## Paradygmaty wirtualizacji
- Server virtualization:
	- Server z maszynami wirtualnymi -> partitioning
- Desktop virtualization
	- zdalny desktop -> remoting
- Aggregation -> odwrotność partycjonowania
	- łączenie wielu serwerów w jeden wirtualny