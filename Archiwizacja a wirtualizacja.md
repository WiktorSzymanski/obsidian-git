#SRC #Sem1 #ZSK 

- Klasyczna archiwizacja
	- np. instalacja agenta w systemie wirtualnym
	- spójne podejście do systemów fizycznych i wirtualnych
	- wyższe opłaty licencyjnej (dla każdej maszynie wirtualnej musi być licencja)
	- konieczność odtwarzania od podstaw
- Archiwizacja obrazów wstrzymanych systemów wirtualnych
	- jedno rozwiązanie dla wielu maszyn wirtualnych
	- uproszczone odtwarzanie
	- wykorzystywanie kopii migawkowych systemów plików (por. bazy danych)
- Kopiowanie obrazów systemów wirtualnych podczas pracy
	- ESX: kopia migawkowa działającego systemu
	- QEMU/KVM: podobnie
	- To jest na dysku zostaje, to co jest w pamięci podręcznej przepada
	- Można to zoptymalizować poprzez wstrzyknięcie agenta, który np. wyczyści brudne buffory (dirty buffers)
	- Jest mała szansa że kopia nie będzie spójna

### Archiwizacja maszyn wirtualnych
---
- Archiwizacja bloków dyskowych
	- on-line
	- wersjonowanie
	- deduplikacja
- Konieczność odtwarzania całej maszyny wirtualnej w celu przywrócenia pojedyńczego pliku