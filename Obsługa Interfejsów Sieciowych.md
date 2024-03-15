#SRC #Sem1 #PS #wykład 

Najniżej jak możemy zejść w programowaniu sieciowym to [[Warstwa Łącza Danych]]

### Obsługa Interfejsów Sieciowych Komendami Systemowymi
---
Podstawowe komendy obsługi interfejsów sieciowych w systemach operacyjnych _unix_ oraz _gnu/Linux_ to `ifconfig(8)` oraz `ip(8)`.

`ifconfig(8)` pochodzi z pakietu _net-tools_, a `ip(8)` z nowszego pakietu _iproute2_.

Polecenia z pakietu net-tools korzystają z plików w katalogu `/proc`, a jeśli nie ma tam to `ioctl(2)`.

Do obsługi parametrów specyficznych dla sieci _IEEE 802.3/Ethernet_ można posłużyć się komendą `ethtool(8)`, a dla _IEEE 802.11/Wi-Fi_ -> `iw(8)`

Wyniki `ifconfig(8)` różnią się pomiędzy systemami operacyjnymi.

Kiedyś eth to karta Ethernet, wlan to WIFI, teraz nazwa zależy od portu gdzie jest wpięty. 
np. em1 -> embeded1, ale już wifi embeded jest wlan0;

>`ethtool -p <nazwa_karty> <czas w sekundach>`
>	diody gniazda zaczną migać

OS identyfikuje karty przez identyfikatory, plusem komendy `ip(8)` jest to że wyświetla te identyfikatory.

_promiscuous mode_ -> nie zwraca uwagi na adres MAC, pobiera wszystkie ramki na dany punkt dostępu. W Ethernet nie robi to różnicy bo switch kieruje ramki na dany port. Widać różnice w Wi-Fi.
### Obsługa Interfejsów Sieciowych Funkcją Systemową `ioctl(2)`
---
- nie jest częścią standardu POSIX stąd implementacje mogą się różnić
- główne zastosowania to obsługa:
	- gniazd sieciowych
	- plików dyskowych
	- interfejsów sieciowych
	- tablic protokołu ARP
	- tablic routingu
	- systemu STREAMS

### Pytania i zadania
---
##### 1. W jakich celach włącza się interfejsy sieciowe bez skonfigurowanych adresów IP?
>
##### 2. Zapoznaj się z dokumentacją komendy `ethtool(8)`, `iwconfig(8)` i `iw(8)`.
>
##### 3. Sprawdź jaka jest różnica pomiędzy trybem nasłuchiwania (_ang. promiscuous mode_), a trybem monitorowania (_ang. monitor mode_) w interfejsach sieci bezprzewodowych IEEE 802.11/Wi-Fi.
>
##### 4. Zapoznaj się z pełną listą żądań funkcji systemowej `ioctl(2)` dla interfejsów sieciowych — zob. `netdevice(7)`.
>
##### 5. Zapoznaj się dokumentacją funkcji `getifaddrs(3)`.
>
##### 6. Zapoznaj się z implementacją funkcji `get_ifi_info` w podręczniku „Unix Network Programming, Volume 1: The Sockets Networking api” (rozdział nr 17, sekcja nr 6).
>

#TODO Zadano i ogarnąć
