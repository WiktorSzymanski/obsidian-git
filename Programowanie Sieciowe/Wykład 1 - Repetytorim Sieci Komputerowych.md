
### Wstęp
Na koniec wykładów zadanka "do domu".
#### Zaliczenie:
- kolokwium pisemne, pytania otwarte
- na ostatnim wykładzie
- 4-5 pytań jedno z zadań "domowych"

Materiały na stronie prowadzącego.
Prowadzący poleca : https://www.unixnetworkprogramming.com

Będzie kod jądra linuxa na wykładach.

"libnet pozwala nam gotować (przygotować gotowe)" ~ MK


### Wykład
##### ISO/OSI
- Warstwa łącza danych
	- komputery połączone kablami do tego samego switcha mogą się komunikować przez ethernet, umożliwia to warstwa łącza danych
	- dwa sposoby radzenia sobie z kolizjami, dla ethernet (wykrywanie) i wifi (unikanie)
	- adres MAC identyfikuje interfejs sieciowy

- Warstwa sieciowa
	- występuje tu komunikacja pomiędzy komputerami nie bezpośrednio/fizycznie połączonymi
	- dodaje nagłówek z adresem IP
	- adres IP jest identyfikatorem systemu operacyjnego
	- jeśli adres IP odbiorcy i nadawcy jest w tej samej sieci system uważa że jest podłączony fizycznie z odbiorcą
	- jeśli jest odwrotnie wstawia to do ramki i wysyła do rutera (cały przepływ trza gdzieś indziej znaleźć bo nie nadąrzam)
	
- Warstwa transportowa
	- występuje tu komunikacja pomiędzy procesami
	- TCP i UDP
		- w TCP jest retransmisja danych gdy odbiorca nie potwierdzi jej otrzymania, UDP nie
	- wykorzystuje numery portu, które muszą być unikalne w skali systemu operacyjnego, identyfikuje on proces w systemie operacyjnym
	
- Warstwa prezentacji
	- TLS
	- szyfrowanie i deszyfrowanie
	
- Warstwa aplikacji
	- tu działają "nasze" aplikacje

##### Zestaw protokołów internetowych
zob. _nazwa_usługi_ -> zobacz, w konsoli można wpisać _nazwa_usługi_

##### Ethertype

##### Podstawowe gniazda sieciowe

\***Obrazek z prezki slajd 18/34**

- dostęp do gniazd surowych i PF_PACKET ma tylko root.

Google zastąpił TCP własnym protokolem QUICK. Jeśli by mieli zaimplementować go jako surowe gniazdo to odpalenie googla by wymagało roota, lub poprosić wszystkich by dodali ich protokół. Zaimplementowali QUICK tak aby wykorzystywał UDP by to obejść.

- socket zwraca deskryptor.
- deskryptor (jakiś int) to liczba po której jest dowiązanie symboliczne do i-węzła opisujący to połącznie np. socket:[numer i-węzła]
- 0 - wejście, 1 - wyjście, 2 - wyjście diagnostyczne
- netstat wyświetla i-węzły w bardziej przyjazny sposób
#TODO Zadanie domowe z prezki