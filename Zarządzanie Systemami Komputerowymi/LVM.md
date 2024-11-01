---
up: "[[Systemy Plików]]"
tags: ZarządzanieSystemamiKomputerowymi
---
# Logical Volume Manager
---

>to narzędzie umożliwiające dynamiczne zarządzanie przestrzenią dyskową, poprzez tworzenie logicznych woluminów i ich grup. Działa pod systemami plików i powyżej fizycznych partycji.

![[Pasted image 20240523225608.png]]

#### Podstawowa funkcjonalność:
- łączenie przestrzeni dyskowej wielu dysków i partycji
- dynamiczna zmiana rozmiaru woluminów
- tworzenie [[LVM#Kopie migawkowe w LVM|kopi migawkowych]] (w dowolnym systemie plików)

#### Funkcje zaawansowane:
- _hybrid volumes_ - łączenie różnych nośników: np. HDD + SDD
- _thin provisioning_
- RAID: 0, 1, 5, 6

#### Zalety:
- **elastyczność** pozwalająca dowolnie kształtować sumę pamięci dysków
- **wydajność** poprzez _striping_ i [[LVM#LVM cache|cache]]
- **wygoda** wynikająca z wielu wbudowanych funkcjonalności

\*_striping_ - rozkładanie danych po wszystkich dyskach fizycznych w celu zwiększenia prędkości odczytu.

#### Wady:
 - **złożoność** spowodowana dodatkową warstwą do zarządzania. Poprawne korzystanie z LVM-a może wymagać pewnej wprawy i jego znajomości.
 - Ponieważ LVM zarządza całą pamięcią komputera może spowodować **_single point of failure_**. W sytuacji gdy metadane LVM-a zostały by uszkodzone, logiczne partycje mogły by stać się niedostępne dla użytkownika.
## LVM cache
>polega na wykorzystywaniu szybkich dysków (np. SSD) jako pamięć podręczna dla wolniejszych dysków (np. HDD). Pozwala to znacząco zwiększyć wydajność wejścia/wyjścia na woluminach logicznych przechowywanych na wolniejszych dyskach, nie rezygnując z dużej pojemności oferowanej przez te wolniejsze dyski.

### Budowa
- **Cache** - wolumin logiczny do buforowania, czyli szybki dysk lub jego część przeznaczona na cache.
- **Origin** - wolumin logiczny docelowy,  czyli wolniejszy dysk lub jego część, która jest buforowana.
- **Metadata** - wolumin logiczny przechowujący metadane, które zarządzają mapowaniem danych między **Cache**, a **Origin**

### Działanie
Dane są przechowywane w **Cache** i **Origin**, gdzie te często używane przechowywane są w **Cache** w celu szybszego dostępu do nich. Na podstawie _cache policy_ **LVM** zarządza, które dane powinny trafić do **Cache**, a które do **Origin** na podstawie tego jak często są one czytane.
#### LVM cache posiada dwa tryby działania:
- **write back** - dane najpierw są zapisywane do **Cache**, a następnie w dogodnym dla systemu momencie, do docelowego woluminu (**Origin**). Poprawia to wydajność zapisu kosztem potencjalnej utraty danych jeśli doszło by do awarii przez przeniesieniem danych na dysk docelowy.
- **write through** - dane są jednocześnie zapisywane do **Cache** i **Origin**. Jest to bezpieczniejsze podejście lecz mniej wydajne.

## Kopie migawkowe w LVM
są tworzone na poziomie bloku (nie systemie plików, jak ma to miejsce w [[Btrfs]]) i wykorzystują [[CoW]]. Kiedy tworzy się [[Kopia Migawkowa|kopię migawkową]], LVM alokuje nowy obszar dyskowy, który będzie przechowywał zmienione bloki od momentu utworzenia migawki. Pierwotne dane pozostają nienaruszone, a zmiany zapisywane są na nowym obszarze. Ponieważ operuje na poziomie bloków pamięci, przy najmniejszej zmianie w pliku zapisujemy i zajmujemy nim cały blok pamięci. Pierwszy zapis pliku po utworzeniu kopi będzie mniej wydajny od kolejnych.