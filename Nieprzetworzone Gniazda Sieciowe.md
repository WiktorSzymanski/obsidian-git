#TODO - wszystko z prez
#### Tworzenie nieprzetworzonych gniazd sieciowych
---
>`SOCK_RAW` na drugim argumencie funkcji `socket` oznacza że gniazdo jest gniazdem nieprzetworzonym/surowym.

>Pakiety odbierane z nieprzetworzonych gniazd sieciowych zawsze zawierają nagłówek _IP_. Stąd ważne aby wiedzieć jak wygląda [[Struktura nagłówka pakietów IPv4]].

>w `setsockopt` wartość `&on` równa 0 oznacza wyłączyć, a 1 (>=1) oznacza włączyć.

#### Pakiety transmitowane
---

#### Pakiety odbierane
---
>Jeśli jądro nie dba o numer jakiegoś protokołu to bezproblemowo przekazuje je do warstwy aplikacji.

>Jeśli pakiet spełnia wiele warunków może zostać przekazany do wielu gniazd/aplikacji.

>Trzeci argument `socket` o wartości `0` oznacza, że wszystko co może trafić do nieprzetworzonego gniazda sieciowego, do niego trafi.

### Przykład
	transmisja pakietu IPv4
---
``` C
	#TODO
```


#### Żądania funkcji systemowej [[ioctl(2)]] dla gniazd sieciowych
---
>`FIOASYNC` - w brew nazwie nie jest to asynchronicznie (pozwala on na obsługę nieblokującą), domyślnie blokujące,  


### Pytania i zadania
---
#TODO