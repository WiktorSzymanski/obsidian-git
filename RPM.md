#SRC #Sem1 #ZSK #wykład 

- Red Hat Packages Manager -> RPM Packages Manager
- De facto standard dla systemów Linux
#TODO 


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

##### Architektóry
#TODO 

Użytkowanie
#TODO 

#### Procedura tworzenia pakietów RPM
#TODO

#### Kompilacja pakietu
- wykonywanie kroki muszą być w danej kolejności

#### Konwencje tworzenie pakietów RPM
- powinny być tworzone jako inny użytkownik jak `root`
#TODO 


### Komendy
> `rpm -q --requires <nazwa>` - wświetla jakich pakietów wymaga `<nazwa>`
> `rpm -q --whatrequires <nazwa>` - wyświetla jakie pakiety potrzebują `<nazwa>`
> `rpm -qc <nazwa>` - wyświetla liste plików konfiguracyjnych dla `<nazwa>`
> `rpm -ql <nazwa>` - wyświetla liste pakietów zainstalowynych dla `<nazwa>`

>pakiet `drpm` można zastosować tylko dla wersji bezpośrednio niższej, jako że zawiera on różnicę między wersią poprzednią i tą która jest w pakiecie.

>`zypper rm -u <nazwa>` - usuwa `<nazwa>` wraz z pakietami zainstalowanymi tylko dla tego pakietu.

