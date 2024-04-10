#SRC #Sem1 #PS #wykład 

- Warstwa 2 [[ISO & OSI]]
- Dostęp do tej warstwy ma tylko root, tak samo programy korzystające z niej potrzebują uprawnień `root`
- Korzyści
	- Daje nam możliwość implementacji analizatorów komunikacji sieciowej (w szczególności w tzw _trybie nasłuchiwania_ ang. Promiscous mode) - Wireshark korzysta z bibliotek (?) działających na tej warstwie (`tcpdump` to taki konsolowy wireshark))
	- Możliwość implementacji protokołów warstwy sieciowej poza jądrem systemu operacyjnego
	- Możliwość realizacji komunikacji sieciowej bez potrzeby użytkowania lub konfigurowania stosu protokołów internetowych
- Komunikacja możliwa między urządzeniami połączonymi ze sobą fizycznie (oba podłączone do jednego switcha lub do tego samego Access point-a)
#### Podwarstwy warstwy łącza danych
![[Pasted image 20240309162008.png]]

MAC -> Medium Access Control, realizowane przez wykrywanie kolizji w Ethernet i zapobieganie kolizji w sieciach WiFi

#TODO Zapytać o to?
Multipleksacja Protokołu (?) - przekazywanie do odpowiedniego {czegoś} w warstwie sieciowej, na podstawie EtherType

LLC -> Logical Link Control wymysł IEEE dodany co sieci bezprzewodowej

Poza nagłówkiem Ethernet może też znależć się nagłówek LLC
