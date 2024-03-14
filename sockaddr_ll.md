#SRC #Sem1 #PS 

Struktura `sockaddr_ll` zastępuje _sockaddr_in_ zwykłych gniazd sieciowych dla PF_PACKET.
#TODO prez 11/26

`sll_protocol` -> [[EtherType]]
`sll_ifindex` -> numer indexu danego interfejsu z którym chcemy powiązać gniazdo
`sll_packettype` -> informacja o rodzaju ramki #TODO prez 12/26

`PACKET_OTHERHOST` -> ramka która nie została nadana na nasz adres MAC

Typy ARP są najmniej istotne. Ich rodzaje znajdują się w pliku if_arp.h