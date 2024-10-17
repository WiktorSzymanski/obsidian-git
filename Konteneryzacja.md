---
class: KSCh
---
# Konteneryzacja
---
## Historia
- Unix `chroot` - lata '80
	- pierwszy krok do stworzenia kontenera, zamknięcie się w jakimś katalogu. Jeśli aplikacja została by złamana narażony jest tylko dany folder.
- FreeBSD jails/Solaris Zones - 2000
- OpenVZ - 2005
- Cgroups - 2006
	- próba wstawienia konteneryzacji do systemu linux.
- AIX WPARs (Workload partitions) IBM - 2007
- LXC (Linux Containers) - 2008
	- konteneryzacja na poziomie systemu operacyjnego, alternatywne rozwiązanie do OpenVZ
- Docker - 2010

LXC i OpenVZ to dwa pierwsze systemu konteneryzacji, które umożliwiały izolację bez potrzeby pełnej wirtualizacji sprzętu. Kontenery działają na poziomie systemu operacyjnego, co oznacza, że nie mają własnego jądra, ale współdzielą jądro z gospodarzem, co zmiejsza zużycie zasobów.

## Kontener aplikacyjny
- Lekka jednostka uruchomieniowa, która zapewnia aplikację oraz jej zależności
- Główny proces (PID 1) to proces aplikacji (np. interpreter Python)
- Zawiera minimalną warstwę systemu operacyjnego
- Jeśli działający na nim program umiera to kontener wraz z nim
## LXC
- Kontener systemowy, uruchamia pełne środowisko systemu operacyjnego
- Główny proces to `systemd` lub `init`
- **Cel**: Izolacja i wirtualizacja na poziomie systemu operacyjnego
- **Przykład**: Wielozadaniowość na poziomie systemu
## Wirtualizacja a konteneryzacja
**Wirtualizacja**:
- każda VM napędza własny OS z jądrem systemu operacyjnego
- możliwość uruchamiania różnych OS
- Większe zurzycie zasobów
- Narzut na wydajność (*hipervizor*)

**Konteneryzacja**:
- Współdzielą jądro systemu z gospodarzem
- mniejsze koszty (tańsze przełączanie kontekstu)
- lżejsze i szybsze uruchamianie aplikacji
- mniejsze zużycie zasobów
- krótki czas ...
- #TODO 

#TODO obrazek z przez

## Narzędzia implementacji kontenerów w systemie Linux
Stare podejście
- OpenVZ: monolityczna, duża łata na jądro systemu
- problematyczna adaptacja do nowszych wersji jądra

Nowe podejście
- Namespaces
- Control groups (Cgroups)

## Namespaces
Przestrzenie nazw pozwalające nas izolować od innych procesów.
- identyfikacja procesów: własna lista procesów, łącznie z PID=1
- nazwa systemu (hostname) i domeny (domainname)
- lista użytkowników i grup (włączając `root`)
- obsługa sieci: oddzielne interfejsty, filtracja pakietów, adresy IP, tablica routingu
- tablica montowania systemów plków wirtualne systemy plików (`/proc`, `/sys`)
- mechanizmy komunikacje międzyprocesorowej

## Control groups (Cgroups)
Mechanizm zarządzania grupami procesów:
- agregacja grupy procesów
- priorytety
- przydział zasobów (CPU, RAM, I/O, network)
- rozliczanie (accounting)

## Wady kontenerów
- ograniczanie do tego samego systemu operacyjnego
	- wyjątki: ABI dla kodu z innych OS (np. Linux pod FreeBSD)
- słabsza izolacja niż w przypadku VM -> bezpieczeństwo
	- funkcje syst. sa wykonywane przez to samo jądro systemu
- Powielanie dużych fragmentów systemów plików dla kontenerów
- Ograniczona przenaszalność

## Ograniczanie do tego samemgo systemu operacyjnego
- Kontenery uruchamiane wewnątrz maszyny wirtualnej (pełna wirtualizacja)
	- cf. Docker dla Linux vx dla Windows i Mac OS X
- Redukcja wydajności
- Problematyczny dostęp do GPU
- #TODO

## Powielanie systemów plików dla kontenerów
- Zastosowanie systemów plików typu *union*
	- (mała) nakładka na wzorcowy system plików z różnicami
- Kopie migawkowe LVM (*physical extents*)
- Kopie migawkowe systemu plików Btrfs (ZFS, *disk bloks*)

### Zalety stosowania kopii migawkowych
- #TODO 

## Migracja kontenerów
### Synchronizacja systemu plików
- współdzielona składnica
- wykorzystanie operacji *send* systemu plików Btrfs -> wysłanie tylko diff

### Synchronizacja pamięci
- wstępna synchronizacja bez blokowania przetwarzania
- zamrożenie + końcowa synchronizacja zmodyfikowanych stron