#SRC #Sem1 #PS 

#TODO prez 4/26

Do podstawowych dniazd sieciowych w OS mają wszyscy ale do niższych (surowych i PF_PACKET) mają dostęp tylko konta _root_.

#TODO prez 5/26

PF_INET -> protokoły internetowe

Gniazdo PF_PACKET przy tworzeniu definuje jakiego typu [[EtherType]] będzie przyjmował.
Trzeci argument funkcji `socket` z PF_PACKET oznacza te EtherType-y.
> `ETH_P_ALL` -> dowolny protokół sieciowy warstwy trzeciej

Jeśli chcemy nasłuchiwać wszysto to poza przejmowaniem każdego protokołu to dodatkowo powinniśmy włączyć tryb _promiscuos_.

Wszystkie kopie ramek wysyłane przez stos protokołów i kartę sieciową przekazywne są do gniazda PF_PACKET. Ponieważ gniazdo PF_PACKET jest wstanie przejmować całą komunikację urządzenia, tylko _root_ ma do niego dostęp.

#TODO prez 13/26

802.11 -> WiFi
802.3 -> Ethernet

### Ramka Sieci Ethernet II
Preambuła -> wysyłana by odbiorca ramki mógł się synchronizować.

Nie mgół być mnieszy niż 64B aby móc wykrywać kolizje. Nie może być większy od 1518B, też pozostałość przeszłości.

Casz propagowania min 2 Tau (jeśli Tau to czas od jednego do drugiego końca), aby wykryć kolizje jeśli taka powstanie.

#### Ethernet od IEEE jest inny od oryginalnego.
7B preambuła i 1B `czegoś type`

Pierwsze 3B w MAC to identyfikator producenta karty sieciowej.
Redundancja tego w SNAP w polu OUI. Rozszerzenie SNAP na początku danych po to aby nie było przerwy z EtherType.
Jeśli [[EtherType]] ma wartości poniżej 1500 to jest to wielkość danych. Powyżej 1500 to EtherType (wszystkie zdefiniowane wartości EtherType są powyżej 1500)

Zakodowanie przez WPA odrazu złamano bo z na początku pakietu WiFi jest zawsze to samo.


#### Przykład
#TODO prez 18/26

`fhead` -> początek naszego buffora, mapowany na `coś`.

To na warstwie łącza danych jest więc nie ma IP.

argv[1] -> index karty sieciowej
argv[2] -> adres MAC odbiorcy
argv[3] -> dane przesylane

Na ramkach WiFi jest enkapsulacja Ethernet, a sterownik sieci WiFi sobie to konwertuje.
#TODO opisać przykład

`recvfrom()` - operacja blokująca

### Pytania:
1. Nie
2. #TODO odpowiedzieć na pytania z wykładu