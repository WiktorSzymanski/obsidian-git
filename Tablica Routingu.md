#SRC #Sem1 #PS

Tablica routingu zawiera adresy `IP` sąsiednich (tj. działających w tej samej sieci `IP`) routerów, przez które wiedzie trasa do oddalonych (tj. niedostępnych bezpośrednio) sieci `IP`.

Każdy wpis w tablicy routingu zawiera m.in. następujące informacje:
- adres `IP` sieci docelowej
- adres `IP` routera sąsiedniego na trasie do sieci docelowej
- maskę adresu `IP` sieci docelowej
- flagi
- metrykę trasy ("odległość" od sieci docelowej)
- liczbę referencji do danej trasy (wartość ta nie jest wykorzystywana przez jądro systemu operacyjnego `GNU/Linux`)
- liczbę trafień/chybień zapytań do pamięci podręcznej tablicy routingu.

> zob. `route(8)`

>Matryka odregłości sieci `RIP` to ilość przeskoków (przejścia przez routery) do sieci docelowej. Istnieją inne metryki jak np. `OSPF`, które biorą pod uwagę przepustowość łączy.

Zawartość tablicy routingu dostępna jest #TODO prez 7/25 `/proc/net/route`
#### Flagi dla wpisów w tablicy routingu
---
- U - "route up"
- H - "target is a host"
- G - "use gateway" : kiedy nie jest bezpośrednio dostępny ale mamy dla niego bramkę
- R - "reinstate route for dynamic routing" : wpis jest dodany ale musi być odświeżany
- D - "dynamically installed by deamon or redirect" : wpis dodany przez protokół a nie administratora
- M - "modified from routing deamon or redirect" : zmodyfikowana przez protokół routingu dynamicznego
- A - "" : #TODO prez 8/25
- C - "" :
- ! - "" : Do danej sieci nie chcemy używać danej bramy

>Redirect host jest kiedy są dwa komputery podłączone do jednego routera ale są w dwóch sieciach wirtualnych.

>`Destination` o wartości `0.0.0.0` wypisane przez `route -n` oznacza bramę domyślną.

Komenda `route -Cn` wyświetla adresy zapisane w pamięci `cache`. Są one brane z pliku `/proc/net/rt_cache` i wyświetlane w czytelny sposób.

---

W systemach operacyjnych `GNU/Linux`, do obsługi tablicy routingu dostępne są następujące żądania:
- `SIOCADDRT` - dodaje nowy wpis do tablicy routingu
- `SIOCDELRT` - usuwa wpis z tablicy routingu
Do obu żądań trzecim argumentem wywołania funkcji `ioctl(2)` jest ... #TODO prez 16/25

`rtentry` - wiele argumentów jest dla kompatybilności wstecznej. 

#TODO prez 17/25

##### Flagi dla wpisów do tablicy routingu
---
``` C
#TODO prez 18/25
```

##### Przykład
	dodanie bramy domyślnej/dodanie trasy do tablicy routingu
---
``` C
#TODO prez 19-20/25
```

