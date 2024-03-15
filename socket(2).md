#SRC #Sem1 #PS #wykład 


``` c
int socket(int domain, int type, int protocol);
```

##### Parametry
---
> `domain` 
> 	określa tzw. domenę komunikacyjną. Domeny takie zadeklarowane są w pliku nagłówkowym` sys/socket.h`, a dla [[Podstawowe Gniazda Sieciowe|gniazd podstawowych]] protokołu ipv4 domena komunikacyjna określona jest stałą `PF_INET`.

>`type`
>	określa typ (semantykę) komunikacji we wskazanej domenie komunikacyjnej. Dla gniazd podstawowych dostępne są dwa typy:
>		_strumieniowy_ (_TCP/IP_) określony stałą `SOCK_STREAM`
>		_datagramowy_ (_UDP/IP_) określony stałą `SOCK_DGRAM`

>`protocol`
>	określa numer protokołu dla wybranej domeny komunikacyjnej oraz wybranego typu komunikacji. Dla gniazd podstawowych przyjmuje wartość `0`.

##### Wartość zwracana
>	numer [[Deskryptor|deskryptora]] gniazda
>	w przypadku błędu `-1`