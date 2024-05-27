---
up: 
class: ZSK
---
#TODO
# Red Hat Packages Manager
---

 > potocznie znany jako RPM Packages Manager, jest _de facto_ standardem dla systemów linux.
 


#### Zawartość pakietu
- Sygnatura - podpisana suma kontrolna
- Nagłówek
	- nazwa oprogramowania
	- wersja oprogramowania
	- wydanie pakietu (uwzględnia m.in. laty)
	- dodatkowe informacje: licencja, opis
- Archiwum plików w skompresowanym formacie cpio (gzip, lzma, w przeszłości [[XAR]])
	- skompilowane oprogramowanie lub
	- pliki źródłowe do samodzielnej kompilacji



\* Wersja oprogramowania to kod, a wersja pakietu to wersja jego skompilowania.

#### Konwencja nazewnictwa

`<nazwa>-<wersja>-<wydanie>.<architektura>.rpm`

np. `samba-client-3.5.4-1.2.i586.rpm`

Pakiety z `-devel` w nazwie są przeznaczone dla programistów

##### Architektury
#TODO 

Użytkowanie
#TODO 

#### Procedura tworzenia pakietów RPM
1. projekt pakietu
	- aplikacja, biblioteka, dokumentacja
2. zebranie oprogramowania
	- oryginalne kody źródłowe + łatki
3. wprowadzenie poprawek
4. opracowanie procedury kompilacji całości pakietu
	- rozpakowanie źródeł, konfiguracja, kompilacja
5. opracowanie procedury aktualizacji
6. opisanie zależności między pakietami
7. utworzenie pakietu
8. testowanie pakietu

#### Kompilacja pakietu
- wykonywanie kroki muszą być w danej kolejności

#### Konwencje tworzenie pakietów RPM
- powinny być tworzone jako inny użytkownik jak `root`
#TODO 


### Komendy
> `rpm -q --requires <nazwa>` - wyświetla jakich pakietów wymaga `<nazwa>`
> `rpm -q --whatrequires <nazwa>` - wyświetla jakie pakiety potrzebują `<nazwa>`
> `rpm -qc <nazwa>` - wyświetla listę plików konfiguracyjnych dla `<nazwa>`
> `rpm -ql <nazwa>` - wyświetla listę pakietów zainstalowanych dla `<nazwa>`

>pakiet `drpm` można zastosować tylko dla wersji bezpośrednio niższej, jako że zawiera on różnicę między wersją poprzednią i tą która jest w pakiecie.

>`zypper rm -u <nazwa>` - usuwa `<nazwa>` wraz z pakietami zainstalowanymi tylko dla tego pakietu.

