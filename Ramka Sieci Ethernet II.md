#SRC #Sem1 #PS 

Preambuła była wysyłana by odbiorca ramki mógł się synchronizować. Wielkość ramki nie mogła być mniejsza niż 64B. Zostało obliczone, że właśnie taką wielkość musi mieć ramka aby móc wykrywać kolizje.

Czasz propagowania ramki musi wynosić minimalnie 2 Tau (jeśli Tau to czas od jednego do drugiego końca łącza danych), aby móc wykryć kolizje jeśli takowa powstanie.

Dodatkowo ramka nie może być większa od 1518B, też jest to pozostałość z przeszłości.

#### Ethernet od IEEE jest inny od oryginalnego.
---
7B preambuła i 1B `czegoś type`

Pierwsze 3B w MAC to identyfikator producenta karty sieciowej.
Redundancja tego w SNAP w polu OUI. Rozszerzenie SNAP na początku danych po to aby nie było przerwy z EtherType.

Jeśli [[EtherType]] ma wartości poniżej 1500 to jest to wielkość danych. Powyżej 1500 to EtherType (wszystkie zdefiniowane wartości EtherType są powyżej 1500)

Zakodowanie przez WPA odrazu złamano bo z na początku pakietu WiFi jest zawsze to samo.


standard 802.11 -> WiFi
standard 802.3 -> Ethernet

#TODO to ładnie zredagować