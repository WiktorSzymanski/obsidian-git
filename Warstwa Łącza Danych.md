#SRC #Sem1 #PS #wykład 

- Warstwa 2
-  Dostęp do tej warstwy ma tylko root, tak samo programy korzystające z niej potrzebują uprawnień `root`
- Korzyści
	- Daje nam możliwość implementacji analizatorów komunikacji sieciowej - Wireshark korzysta z bibliotek (?) działających na tej warstwie
	- Możliwość implementacji protokołów warstwy sieciowej poza jądrem systemu operacyjnego
	- Możliwość realizacji komunikacji sieciowej bez potrzeby użytkowania lub konfigurowania stosu protokołów internetowych
- Komunikacja możliwa między urządzeniami połączonymi ze sobą fizycznie (oba podłączone do jednego switcha lub do tego samego Access point-a)
#TODO rysunek 1 z prez

#### Podwarstwy warstwy łącza danych
#TODO rysunek 2 z prez
MAC -> Medium Access Control, realizowane przez wykrywanie kolizji w Ethernet i zapobieganie kolizji w sieciach WiFi

Multipleksacja Protokołu - przekazywanie do odpowiedniego {czegoś} w warstwie sieciowej, na podstawie EtherType

LLC -> Logical Link Control wymysł IEEE dodany co sieci bezprzewodowej

Poza nagłówkiem Ethernet może też znależć się nagłówek LLC
