---
tags:
  - KonstrukcjaSystemówChmurowych
---
# Unikernels
---
>**Unikernels** (aka. *liblary operating system*) to implementacja konceptu połączania (fragmentów) systemu operacyjnego z kodem aplikacji w monolityczne środowisko wykonawcze por.: *static linking*. W ten sposób można by stowrzyć apklikację która jest mała i szybka. Izolacja procesów jest zbędna, ponieważ nadzorca zapewnia izolację. Istniała by pojedyńcza/płaska przestrzeń adresowa: OS + app. Całość uruchamiana jako pojedyncza palikacja w ramach pełnej wirtualizacji (lub bezpośrednio na sprzęcie). Pamięć trwała: bez deskryptorów plików, bez systemu plików. Można by stworzyć dedykowane alokatory pamięci, pozwalające zastosować konkretny aplikator do danej optymalizacji

## Zalety
- bardzo szybki start (*boot time*), rzędu 3 / 40 ms (łącznie z VMM)
- wysoka przepustowość i wydajność
- minimalne zapotrzebowanie na pamięć operacyjną
	- np. serwer Redis 1 MiB kodu + 10 MiB RAM
- brak kosztów wywołań systemowych
	- zbędzne kopiowanie danych między aplikacją a OS
- ominięcie kosztownych interfejsów programowych
	- BSD sockets -> DPDK, netmap, io_uring
	- storage #TODO
- bezpieczeństwo (minimalizacja *attack surface*) -> mniej miejsc w kodzie na potencjalne w błedy
- obsługa żądań sieciowych w caości - zbędnych *scheduler* #TODO

## Wady
- bardzo skomplikowany i czasochłonny proces opracowania
- konieczność dostosowania aplikacji (*porting*)
- dla każdej aplikacji oddzielnie
- **unikernel** potrzebuje sterowników dokładnie tego urządzenia, na którym będzie uruchomiony
- uruchomienie kliku aplikacji z izolacją staje się trudne

## Strategie implementacji
- Modularyzacja monolitycznego jądra - problem wewnętrznych zależności
	- `sendfile()` - zwornik *storage* i *networking*
- Przejście na nowe interfejsy, omijające OS
- Dodawanie do aplikacji brakujących funkcji z OS

### Implementacje
- OSv, Rump oparte na komponentach jądra BSD
- Lupine Linux - minimalistyczne jądro + Kernel Model Linux
	- prosta adaptacja aplikacji, duże monolityczne jądro
- [[Unikraft]]

### Problemy
- licencje: np. biblioteka `musl libc` na licencji MIT
- wieloprocesorowość/współbieżność
- obciązanie wielu rdzeni

Zależności wewnątrz jądra systemu Linux
#TODO obrazek z prez