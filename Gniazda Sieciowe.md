#SRC #Sem1 #PS #wykład 

Gniazda Sieciowe są traktowane przez system operacyjny jako pliki specjalne. Aby korzystać z mechanizmu komunikacji internetowej wymagane jest utworzenie gniazda sieciowego.

[[Podstawowe Gniazda Sieciowe]] pozwalają na komunikację strumieniową (_TCP/IP_) i datagramową (_UDP/IP_). Dostęp do gniazd surowych i [[Gniazda Sieciowe PF_PACKET|PF_PACKET]] ma tylko użytkownik z uprawnieniami _root_-a.

Google zastąpił TCP własnym protokołem [[QUIC]].

### Pytania i zadania
---
##### 1. Czy kapsułkowaniu zawsze musi towarzyszyć fragmentacja i na jakich warstwach modelu warstwowego [[ISO & OSI|ISO/OSI]] może ona wystąpić?
>	Kapsułkowanie nie zawsze musi towarzyszyć fragmentacji. Fragmentacja jest procesem dzielenia dużych pakietów danych na mniejsze części, aby mogły być przesyłane przez sieć, który może wystąpić na różnych warstwach modelu OSI, ale jest szczególnie istotny w warstwie sieciowej, gdzie dane są segmentowane do pakietów, które są następnie _encapsulated_ (opakowywane) na niższe warstwy. Kapsułkowanie, z drugiej strony, to proces dodawania nagłówków do danych na każdej warstwie modelu OSI, co pozwala na ich odpowiednie przetwarzanie i przesyłanie przez sieć.

>	Kapsułkowanie może wystąpić na następujących warstwach modelu OSI:
>		- **Warstwa transportowa**: Dane są segmentowane i dodawany jest nagłówek zawierający numer portu aplikacji.
>		
>		- **Warstwa sieciowa**: Tworzone jest pakiet poprzez dodawanie nagłówka zawierającego adresy IP nadawcy i odbiorcy.
>		
>		- **Warstwa łącza danych**: Tworzona jest ramka poprzez dodanie nagłówka zawierającego adresy fizyczne (MAC) nadawcy i odbiorcy.
>		
>		- **Warstwa fizyczna**: Ramka jest kodowana i przesyłana do odbiorcy.
>	
	Fragmentacja natomiast może wystąpić na warstwie sieciowej, gdzie dane są dzielone na pakiety, które są następnie _encapsulated_ na niższe warstwy modelu OSI. Fragmentacja jest niezbędna do przesyłania danych przez sieć, gdzie rozmiar pakietów może być ograniczony przez ograniczenia sieciowe lub sprzętowe.
	
##### 2. Jakie są różnice pomiędzy komunikacją strumieniową a komunikacją datagramową?
>	- **Komunikacja strumieniowa** (ang. stream-oriented communication) polega na przesyłaniu danych jako ciągłego strumienia bajtów. W tym modelu dane są przesyłane w sposób sekwencyjny, a otrzymanie danych jest gwarantowane w tej samej kolejności, w jakiej zostały wysłane. Komunikacja strumieniowa jest często używana w aplikacjach, które wymagają stabilności i niezawodności, takich jak telefonia VoIP, transmisja wideo, czy przesyłanie plików.
>	
>	- **Komunikacja datagramowa** (ang. datagram-oriented communication) polega na przesyłaniu danych jako niezależnych jednostek, zwanych datagramami. Każdy datagram może być przesyłany niezależnie od innych i może dotrzeć do odbiorcy w innej kolejności, niż w jakiej został wysłany. Komunikacja datagramowa jest używana w aplikacjach, które wymagają elastyczności i szybkości, takich jak gry online, gdzie ważne jest, aby dane dotarły do odbiorcy jak najszybciej, a niekoniecznie w określonej kolejności.
	
##### 3. Sprawdź zawartość plików `/etc/protocols`, `/etc/services` oraz `if_ether.h1` w systemie operacyjnym unix lub gnu/Linux.
>	`/etc/protocols`
>		zawiera listę protokołów i przypisane do nich liczby
>		
>	`/etc/services`
>		zawiera listę serwisów internetowych i przypisanych do nich portów
>		
>	`if_ether.h1`
>		zawiera definicję stałych i struktur związanych z protokołami Ethernet
		 
##### 4. Sprawdź co oznacza wartość [[EtherType]] równa `0x8870`.
>	Proponowana jako specjalna wartość dla Jumbo Frames (ramki Ethernet o rozmiarze większym niż standardowy 1518 bajtów) w celu rozwiązania konfliktu, który miał miejsce, gdy rozmiar ładunku ramki Ethernet był bliski zakresu używanego przez EtherType. Jednakże, propozycja ta nie została zaakceptowana i jest obecnie uważana za nieaktywnną. Wartość 0x8870 była zaproponowana w kontekście większych pakietów dla protokołu IS-IS, ale nie została oficjalnie przyjęta i nie jest już używana w standardowej implementacji protokołu Ethernet. Propozycja ta była implementowana w niektórych routerach Cisco w ich implementacji protokołu IS-IS, gdzie była używana do paddingu pakietów Hello (IIH) w ramkach IS-IS. Wartość EtherType 0x8870 nie jest więc oficjalnie przypisana do żadnego protokołu i nie powinna być używana w nowych implementacjach.
		
##### 5. Czy implementując komunikację _datagramową_ (bezpołączeniową) z zastosowaniem podstawowych gniazd sieciowych możliwe jest użycie funkcji `connect(2)`? Jeżeli tak, to w jakim celu?
> 	Funkcja `connect(2)` jest używana do połączenia gniazda z adresem zdalnym, ale w kontekście komunikacji datagramowej (UDP), jej działanie jest nieco inne niż w przypadku komunikacji strumieniowej (TCP).
> 
> 	W komunikacji datagramowej, wywołanie `connect` na gnieździe nie ustanawia pełnego połączenia w sensie TCP, ale wiąże gniazdo z określonym adresem i portem zdalnym. Dzięki temu, wszystkie kolejne operacje wysyłania i odbierania danych za pomocą tej funkcji (np. `send`, `recv`) będą automatycznie kierowane do lub od tego konkretnego adresu i portu. Jest to szczególnie przydatne w aplikacjach, które komunikują się z jednym, stałym adresem zdalnym, ponieważ eliminuje konieczność określania adresu i portu zdalnego za każdym razem, co może przyspieszyć komunikację i uprościć kod.