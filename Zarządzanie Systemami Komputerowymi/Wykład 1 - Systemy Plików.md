Materiały : www.cs.put.poznan.pl/csobaniec/edu/zsk

DO WYPEŁNiENIA Z PREZKĄ

### Wprowadznie

- DOS - nie duży rozmiar sektora pamięci (max. 2TiB)
- GUID Partition Table (GPT) - znacząco więszy sektor pamięci

Księgowanie:
- operacje zapisujemy w logu aby przy awarii wiedzieć czy plik poprawie się zapisał (trochę jak z DBMS)
- w ten sposób z logu wiemy czy utraciliśmy pliki i nie musimy długotrwale skanować
- odnosi się do metadanych, np. które bloki zajmuje plik
- stało się standardem

Pamięć SSD/NVMe:
- ogólnie jest 6 razy więcej odczytów jak zapisów, stąd trzeba je optymalizować. 
- w flash ssd nie można łatwo nadpisać jednego bloku, stąd jeśli chcemy zapiać małą ilość danych to musimy zaczytać cały blok i całego go zapisać
- sekwencyjny zapis metadanych i danych, dane które chcemy zapisać pakujemy w "bloki" i zapisujemy całe go pamięci
- równolegle z zapisem działa *garbage collector*, jeśli w jakimś bloku jest mało plików, to dopełniamy nimi blok w celu zwolnienia całego blaoku na którym są
- jeśli jakiś plik jest rozłożony po kilku blokach jest to nieoptymalny proces

System plików w pliku:
- za pomocą systemu loop
- więcej na Lab 1

System plików tylko do odczytu:
- Kompresja - ze względu na wolny odczyt dekompresja nie była odczuwalna (CPU robił to szybciej jak odczytywane były nowe dane)
- Deduplikacja - usuwanie kopii, wszystkie identyczne pliki odwołują się do jednej kopii tego pliku.

NTFS w Linux:
- pierwsze implementacje były ryzykowne i tworzone przez *reverse engienering* (źle napisane?)
- implementowany przy pomocy FUSE

FUSE:
- system plików jako proces, poza jądrem systemu
- wygoda implementacji, jeśli się wywali to process, a nie jądro
- mniej wydajny

\***IMG Z PRESKI DODAĆ**

Systemy plików w pamięci:
- wydajny, bo w pamięci podręcznej
- ulotne

Grupowanie systemów plików:
- FreeBSD pozwala zrobić union komendy mount, tz. nie nadpisujemy danych już zamątowanych (domyślnie mount przykrywa tak że nie widać warstwy niżej).

Rozszerzone atrybuty systemów plików:
- eXtended attributes - struktura klucz wartość
- wszystkie systemy plików i narzęcia do archiwizacji muszą wspierać metadane aby nie tracić informacji o pliku przy jego przenoszeniu/kopiowaniu
- windows ma dodatkowe, nazwane strumienie danych, ale to już wiemy że:
	- "jest nie wykorzystywane i conajwyżej zagrożenie stanowi" ~ BSI
	- "dobre środowisko dla wirusów" ~ ZSK
- inne atrybuty (chattr np. zapobiegające niechcianym operacją usuwaina)

##### "Jak coś w systemie nie działa to pierwsze co sprawdzić to czy jest wolne miejsce na dysku"

Local Volume Manager ([[LVM]]):
- podzielone prtycje są sztywne, więc chcemy dynamicznie powiększyć partycje która pilnie wymaga więcej miejsca (np. varlog)
- pozwala tworzyć kopie migawkowe - kopia utrwalająca stan (spójny), w danym momencie, jak kopiujemy bardzo duży plik (kopujemy go przez 2h) początek tego pliku (kopiowany prawie 2h temu) nie jest od "tego samego" pliku co koniec tego pliku, kopia migawkowa pozwala na utrwalenie stanu pliku który dalej można sobie zgrywać, a oryginalny plik dalej żyje swoim życiem
- hybrid volumines - łącznie różnych nośników np. HDD i SDD

\***IMG Z PREZKI LVM slajd 13/23**

LVM cache:
- bez sensu duże pliki z HDD do cache bo raczej żadko będziemy je potrzebowali "pod ręką"

Thin provisioning: 
- zużywa przestrzeń dyskową dopiero jak coś na niej zapisze
- nie trzyma zaalokowanej i nie używanej przestrzeni
- jeśli mieli byśmy być "niewypłacalni" możemy dołożyć dodatkowej przestrzeni dyskowej, stąd będzie wrażenie że wszyscy użytkownicy mają tyle pamieci ile powinni mieć (nie pamiętam czemu ale było coś że wtedy jest jak z bankami, nie ma całej tej pamięci którą mieć powinny)

\***IMG Z PREZKI slajd 16/23**


Mechanizm Copy-on-Write:
- jest użyteczny i ciekawy
- funkcjonuje na poziomie systemu plików

Btrfs:
- warto wiedzieć i poeksperymentować
- korzysta z CoW
	- Przydatne w przypadku archiwizacji wielowersyjnej
	- Minusem jest to, że jeśli plik ulegnie awarii wszystkie nasze "kopie" też będą uszkodzone
- pozwala na tworzenie wersji systemów plików
- zapisywanie skompresowanych danych jako tych "normalnych" (?)
- mechanizmy RAID0/1/5 od ręki, RAID5 niestabilny na ten moment na Btrfs
- LVM vs Btrfs z RAID0:
	- w LVM zapis na jednej partycji, jak nie ma miejsca to na drugiej
	- RAID0 dzieli na bloki i równolegle je zapisuje

LVM z RAID1 vs Btrfs z RAID1:
- RAID sprzętowy i softwerowy czyta na raz z jednego dysku
- Btrfs czyta równolegle, stąd daje nam x2 prędkości odczytu

Podwoluminy:
- pozwalają tworzyć kopie migawkowe
- limity objętości, bardziej elastyczna od LVM

Obsługa mechanizmu CoW:
![[dowiązanie_trwałe.png]]
![[dowiązanie_w_btrfs.png]]
Btrfs send-recieve:
- do przetransportowania tylko zmiany, bloki dyskowe które się zmieniły, nie musimy szkuać które się zmieniły bo Btrfs wie które to są, stąd złożoność O(1)
- bardzo przyda się przy backup-ie
- wysyłanie snapschot-a będzie błyskawiczne

Btrfs Seeding Device:
- możliwość inicjowania systemu plików z zawartością innego systemu plików

Wydajność:
- różne miary wydajności:
	- throughput
	- latency
	- IOPS - InOutPerSecond (?)
	- obsługa kopii migawkowych