#SRC #Sem1 #PS #wykład 

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