---
tags:
  - KonstrukcjaSystemówChmurowych
  - ZarządzanieSystemamiRozproszonymi
---
#TODO rozbić na mniejsze np. dodatkowe Wirtualizacja Procesora, Pamięci itp.
# Wirtualizacja
---
>Powstała na potrzeby **optymalizacji zasobów**, by uruchamiać wielu systemów operacyjnych na jednym serwerze, zapewnienia **izolacji**, stabilności i bezpieczeństwa systemu, **elastyczności** pozwalająca uruchamiać różne OS na jednym urządzeniu oraz łatwego tworzenia nowych wirtualnych środowisk co pozwala na **skalowalność**.

Wirtualizacja adresuje problem fragmentacji, nie do końca go rozwiązując. Narzut wirtualizacji powoduje zwiększone zużycie zasobów w ogólne, mimo prawdopodobnego zmniejszenia liczby maszyn fizycznych.

## Zalety
- Lepsze zarządzanie zasobami
- Elastyczność
- Bezpieczeństwo i izolacja
## Wady
- Narzut na wydajność (*hipervizor*)
- Złożoność zarządzania
- Większe zasoby sprzętowe

## Zastosowania wirtualizacji
- **Konsolidacja serwerów** - zmniejszenie liczby fizycznych serwerów poprzez przeniesienie ich na nowe, szybsze w trybie wirtualizacji
- **Bezpieczeństwo** - umieszczenie poszczególnych usług sieciowych w oddzielnych serwerach wirtualnych, czyli izolacja
- **Partycjonowanie/Hosting** - sprzedaż wirtualnych serwerów w całości administrowanych przez klienta.
- **Tworzenie oprogramowania  i testowanie** nowego oprogramowania, zwłaszcza systemowego, szybszy restart.
- **Edukacja** - bezpieczne ćwiczenia z administracji, każdy student ma konto administratora. Uszkodzony system łatwo odtworzyć.
- **Oszczędność energii** - redukcja liczby serwerów wpływa na mniejsze zużycie energii.

## Własności wirtualizacji
**Nadzorca wirtualizacji** *Virtual Machine Monitor* (VMM), *hypervisor* (*supervisor of supervisors*)
### Własność środowiska tworzonego przez VMM:
- **Równoważność** maszyny wirtualnej i sprzętu fizycznego 
- **Kontrola zasobów** wirtualnych maszyn (VM)
- **Wydajność** - większość kodu maszyny wirtualnej wykonuje się bez udziału nadzorcy VM

## Wyzwania wirtualizacji
- [[Wirtualizacja w procesorach z rodziny x86]]
- [[Wirtualizacja Pamięci]]
- [[Obsługa Urządzań]]

### Dyskusja
- Wydajność
	- parawitualizacja
	- pełna wirtualizacja z translacją instrukcji wrażliwych
	- wirtualizacja ze wsparciem sprzętowym
- Wsparcie sprzętowe II generacji -> wyższa efektywność
- Parawirtualizacja wymaga modyfikacji OS
	- dodatowe koszty
	- niemożliwość wprowadzenia zmian (systemy Windows)
	- modyfikacje OS są dla konkretnego nadzorcy

## Wirtualizacja na poziomie systemu operacyjnego
- Virtual Private Server (VPS), Virtual Environments (VE)
	- własny system plików
	- własna baza użytkowników
	- pamięć
	- procesy
	- adresy sieciowe
- System bazowy i wirtualny muszą być tym samym systemem operacynym
- Zmodyfikowane, pojedyńcze jądro
- Dodatowe obciążenie rzędu 1-3%
- Możliwość uruchomnienia setek VPS na jednym komputerze
#### Przykłady
- FreeBSD Jails
- Solaris Containers
- OpenVZ
- LXC
- Docker

### Punkty kontrolne i migracja
#TODO 

"Wszyscy mówią o, za przeproszeniem, sztucznej inteligencji" ~ C. Sobaniec

## Wirtualizacja GPU
### Passthrough
#TODO
### Mediated devices
#TODO

