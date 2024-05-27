---
up: 
class: ZSK
---
# Układy partycji
---
## DOS
- 4 partycje podstawowe
- partycja rozszerzona
- Nie duży rozmiar sektora pamięci, max. 2$^3$$^2$ sektorów po 512 B => 2 TiB

## GUID Partition Table (GTP)
- część standardu _Unified Extensible Firmware Interface_
- 128 partycji
- Znacząco większy sektor pamięci, max. 2$^6$$^4$ sektorów po 4 KiB => 64 ZiB

# Rozszerzona funkcjonalność systemów plików
---

> [!Box]- księgowanie
> ![[Księgowanie Systemów Plików]]

> [!Box]- indeksowanie katalogów
> (B-tree)

> [!Box]- kompresja
> (transparentna)

> [!Box]- szyfrowanie

> [!Box]- kopiowanie przy zapisie
> (ang. _copy-on-write_)

> [!Box]- kopie migawkowe

> [!Box]- obsługa atrybutów plików
> _extended attributes_ (`xattr`) to medatane plików zawierające pewnego rodzaju informacje, wspierane w narzędziach `cp` i `tar`.
> 
> Nie ma standardu co do ich maksymalnego rozmiaru:
> - ext2/3/4, Btrfs => pojedyńczy blok dyskowy
> - XFS, ReiserFS => brak limitów
> - JFS => 255 B na nazwę i 64 KiB na wartość
>   
> W Windows istnieją dodatkowe, nazwane strumienie danych.
> 
> Inne atrybuty (`chattr`):
> - no acces time
> - append only
> - compression
> - no copy-on-write
> - immutable
> - secure delete
> - synchronous writes
> - undeletable

# Systemy plików dla urządzeń SSD/NVMe
---

- ogólnie jest 6 razy więcej odczytów jak zapisów, stąd trzeba je optymalizować. 
- w flash ssd nie można łatwo nadpisać jednego bloku, stąd jeśli chcemy zapiać małą ilość danych to musimy zaczytać cały blok i całego go zapisać
- sekwencyjny zapis metadanych i danych, dane które chcemy zapisać pakujemy w "bloki" i zapisujemy całe go pamięci
- równolegle z zapisem działa *garbage collector*, jeśli w jakimś bloku jest mało plików, to dopełniamy nimi blok w celu zwolnienia całego blaoku na którym są
- jeśli jakiś plik jest rozłożony po kilku blokach jest to nieoptymalny proces