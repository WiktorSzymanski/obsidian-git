#SRC #Sem1 #ZSK 

Pewne rzeczy wykonuje w tle na wątkach jątra systemu. Stąd działanie niektórych operacji widać po jakimś czasie.
#### Woluminy wewnętrzne
pozwalają tworzyć _sub_-partycje w partycjach, a następnie je podpinać, a co za tym idzie ukrywać coś w przestrzeni nad podpiętym dyskiem.

#### Kopie migawkowe
W przeciwieństwie do [[LVM]] który działał pod systemem plików, btrfs tworzy migawki na poziomie systemu pliku. 

#### Kompresja
Przy szybkim procesorze i wolnym dysku, kompresja może przyśpieszyć operacje wejścia wyjścia. Btrfs podczas kompresji sprawdza czy daje to jakikolwiek zysk, w przeciwnym wypadku nie dokonuje kompresji.