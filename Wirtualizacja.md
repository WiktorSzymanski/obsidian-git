---
class: KSCh
aliases:
  - KSCh
  - ZSR
---
#TODO rozbić na mniejsze np. dodatkowe Wirtualizacja Procesora, Pamięci itp.
# Wirtualizacja
---
>Powstała na potrzeby **optymalizacji zasobów**, by uruchamiać wielu systemów operacyjnych na jednym serwerze, zapewnienia **izolacji**, stabilności i bezpieczeństwa systemu, **elastyczności** pozwalająca uruchamiać różne OS na jednym urządzeniu oraz łatwego tworzenia nowych virtualnych środowisk co pozwala na **skalowalność**.

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
- **Bezpieczeństwo** - umieszczenie poszczególnych usług sieciowych w oddzielnych serwerach wirtualnych, czyli izolacjia
- **Partycjonowanie/Hosting** - sprzedaż wirtualnych serwerów w całości administrowanych przez klienta.
- **Tworzenie oprogramowania  i testowanie** nowego oprogramowania, zwłaszcza systemowego, szybszy restart.
- **Edukacja** - bezpieczne ćwiczenia z administracji, każdy student ma konto administratora. Uszkodzony system łatwo odtworzyć.
- **Oszczędność energii** - redukcja liczby serwerów wpływa na mniejsze zużycie energii.

## Własności wirtualizacji
**Nadzorca wirtualizacji** *Virtual Maschine Monitor* (VMM), *hypervisor* (*supervisor of supervisors*)
### Własność środowiska tworzonego przez VMM:
- **Równoważność** maszyny wirtualnej i sprzętu fizycznego 
- **Kontrola zasobów** wirtualnych maszyn (VM)
- **Wydajność** - większość kodu maszyny wirtualnej wykonuje się bez udziału nadzorcy VM

## Wirtualizacja w procesorach z rodziny x86
Normalnie działający system operacyjny.
- Ring 0 -> bezpośredni dostęp do pamięci i urządzeń
- Ring 3 -> applikacje
- Virtual Maschine Monitor (VMM) musi byc poniżej systemu operacyjnego (OS)
#TODO Obrazek z prez

**Instrukcje uprzywilejowane** wymagają trybu uprzywilejowanego (Ring 0, Ring 123 -> trap)
**Instrukcje krytyczne** wpływają na konfigurację zasobów w systemie lub ich zachowanie zależy od konfiguracji zasobów
**Instrukcje wrażliwe** krytyczne ale nie uprzywilejowane

#### Warunki wirtualizacji
Warunkji Popka i Goldberga (1974)
- wszystkie wrażliwe instrukcje wykonwane w trybie użytkownika są przechwytywane i kierowane do VMM
- procesory x86 Intela nie 
#TODO 

### Pełna wirtualizacja
- Translacja (on-line) instrukcji wrażliwych jądra wirtualizowanego OS
- Normalne wykonywanie pozostałych istrukcji
- VMM wirtualizuje BIOS, urządzenia i pamięć
- VMM pracuje w ramach bazowego OS
- vOS jest odseparowany od sprzętu
- vOS nie jest świadomy wirtualizacji
- vOS nie wymaga zmian
#TODO obrazek z przez
#### Przykład
- VMware Workstation
- VirtualBox
- VirtualPC
- Parallels

### Parawirtualizajca
- Zmodyfikowany vOS, świadomy wirtualizacji
- Instukcje krytyczne -> jawne wywołania nadzorcy
- Nadzorca: zarządzanie pamięcią, obsługa przerwań
- Lepsza wydajność przez brak konieczności wykonywania wielu operacji zamiast jedenej w przypadku instrukcji wrażliwych

#TODO obraz z prez
#### Przykład
- Xen
- MS Hyper-V
- VMware Tools
- VirtualBox Guest Additions

### Wirtualizacja wspierana sprzętowo
- Automatyczne przechwytywanie instrukcji wrażliwych i przekazywanie sterowania do VMM
- Ograniczone możliwości kontrolowania -> mniejsza efektywność niż wcześniejszych modeli
#TODO obrazek z prez
#### Przykład
- Intel Virtualization Technology (VT-x),
- AMD-V

## Wirtualizacja Pamięci
 - Podobna do klasycznej obsługi pamięci wirtualnej
 - Dodatowy poziom wirtualizacji, poniżej wirtualizacji vOS
	 - pamięć aplikacji vOS
		 - wirtualna pamięć fizyczna vOS
		 - faktyczna pamięć wizyczna
 - Optymalizacja: tablice stron kopiowane z vOS w celu sktócenia odwzorowań (*ang. shawod pages*)

#### Klasyczne odwzorowanie adresów pamięci
ostatnie odwzorowania **LPN** (*Logical Page Number*) -> **PPN** (*Physical Page Number*) przechowywane są w rejestrach **TLB** (*Translation Lookaside Buffer*)

#TODO obrazek z prez

- VMM: PPN -> MPN
- shadow page tables: LPN -> MPN (widoczne przez sprzęt)
- rejestry TLB zawierają ostatnie odwołania LPN -> MPN (przeładowanie podczas przełączanie na inny vOS). 
#TODO obrazek z prez

> Problem: Rejestry TLB były czyszcone co kwant czasu procesora więc tylko na chwilę mieliśmy te optymalizacje

##### Wsparcie sprzętowe drugiej generacji:
- AMD
	- Tagged TLB
	- Rapid Virtualization Indexing (RVI) (poprzednio: Nested Page Tables)
- Intel
	- Extended Page Tables
- wykorzystanie *huge pages*: 2MiB, 1GiB
- VMware tests: 42%, RedHat: 200% for OLTP, up to 600%

> Rozwiązuje to problem z czyszczonymi rejestrami.

MMU (Memory Menagement Unit) korzysta zarówno z odwzorowań LPN -> PPN jak i PPN -> MPN. Co za tym idzie znika konieczność korzystania z *shadow pages*.

#TODO obrazek z prez

Zaawansowane zarządzanie pamięcią:
- opóźnianie alokacji pamięci
- [[CoW|copy on write]]
- zwalnianie nieużywanej pamięci
- deduplikacja stron pamięci - jeśli mamy odpalone wiele takich samych systemów to można wykorzystać tylko jednej załadowanej, nie trzeba ich ładować dla każdej VM.
	- VMware: *Transparent Page Shareing*
	- KVM: *Kernel SamePage Merging*
	- VirtualBox: *Page Fusion*
- kompresja stron pamięci
- sterownik pamięci typu *baloon* -> *overcommit* - vOS myśli że ma pewną ilość pamięci, a tym czasem jest ona dostępna dla hosta
![[VM-baloon.excalidraw.png]]


- programowa implementacja urządzeń
	- np. wirualny interwejs sieciowy, wirtualna podsieć
	- podmiana fizycznego sprzętu bez przerywania pracy
- Emulacja znanych urządzeń -> większa przenośność
- Przekazywanie urządzeń do wybranych vOS
- Obsługa DMA (translacja adresów)
- Zarządzanie przerwaniami 

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

