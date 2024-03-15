#SRC #Sem1 #PS 

Struktura `sockaddr_ll` jest odpowiednikiem `sockaddr_in` w gniazdach podstawowych dla [[Gniazda Sieciowe PF_PACKET|gniazd PF_PACKET]]. Opisuje adresy warstwy łącza danych i jest wykorzystywana podczas transmitowania i odbierania ramek oraz podczas wiązania gniazda sieciowego z adresem przy użyciu funkcji systemowej` bind(2)`.

``` c
struct sockaddr_ll {
	unsigned short sll_family;      /* Always PF_PACKET. */
	__be16 sll_protocol;            /* Network layer protocol. */
	int sll_ifindex;                /* Interface number. */
	unsigned short sll_hatype;      /* ARP hardware type. */
	unsigned char sll_pkttype;      /* Packet type. */
	unsigned char sll_halen;        /* Length of address. */
	unsigned char sll_addr[8];      /* Data link layer address. */
	};
```
##### Atrybuty
---
>`sll_protocol`
>	oznacza rodzaj protokołu warstwy sieciowej przez [[EtherType]]

>`sll_ifindex
>	numer interfejsu sieciowego z którym wiążemy gniazdo. Można go uzyskać przy pomocy funkcji systemowej `ioctl(2)` z żądaniem `SIOCGIFINDEX`

>`sll_hatype`
>	typy sprzętowe protokołu `ARP`. Zapisane są w pliku `if_arp.h`. Są najmniej istotne.

>`sll_packettype`
>	informacja o rodzaju ramki. Typy ramek, tj. wartość pola `sll_pkttype`, zapisane są w pliku `if_packet.h`

##### Możliwości typów ramek `sll_packettype`
---
``` c
#define PACKET_HOST 0       /* To us. */
#define PACKET_BROADCAST 1  /* To all. */
#define PACKET_MULTICAST 2  /* To group. */
#define PACKET_OTHERHOST 3  /* To someone else. Packet sent to not ours MAC addres */
#define PACKET_OUTGOING 4   /* Originated by us. */
```