#SRC #Sem1 #PS 

W systemach operacyjnych gnu/Linux istnieją dwa mechanizmy systemowe umożliwiające dostęp do warstwy łącza danych: 
- _gniazda sieciowe domeny komunikacyjnej `PF_PACKET`_ (nowsze rozwiązanie)
- _gniazda sieciowe typu `SOCK_PACKET`_ (rozwiązanie przestarzałe).

Rozwiązanie nowsze oferuje więcej funkcji filtracji danych oraz działa efektywniej w odniesieniu do rozwiązania starszego. Do podstawowych gniazd sieciowych w systemie operacyjnym mają wszyscy ale do niższych (surowych, `PF_PACKET` i `SOCK_PACKET`) mają dostęp tylko konta z uprawnieniami _root_-a.

![[Pasted image 20240315191441.png]]

`PF_PACKET` określa domenę komunikacyjną i wymaga typu `SOCK_RAW` lub `SOCK_DGRAM`. `SOCK_PACKET` określa typ komunikacji w domenie `PF_INET` (protokoły internetowe).

Wybranie typu `SOCK_DGRAM` dla gniazd sieciowych w domenie `PF_PACKET` spowoduje, że dane przekazywane z takiego gniazda **nie będą zawierały** nagłówka warstwy łącza danych (_ang. „cooked” packets_). Dane wysyłane takim gniazdem sieciowym **nie powinny zawierać** nagłówka warstwy łącza danych — dane adresowe pobrane zostaną ze struktury adresowej.

Wybranie typu `SOCK_RAW` dla gniazda sieciowego w domenie `PF_PACKET` spowoduje, że dane przekazywane z takiego gniazda **będą zawierały** nagłówek warstwy łącza danych. Dane wysyłane takim gniazdem **muszą zawierać** nagłówek warstwy łącza danych — niezmodyfikowane dane zostaną w całości przekazane do sterownika karty sieciowej i wysłane.

Dla obu rodzajów gniazda `PF_PACKET` przy tworzeniu trzeba określić jakiego typu [[EtherType]] będzie przyjmował.

#### Przykład tworzenia gniazda
---
##### Gniazdo sieciowe domeny komunikacyjnej `PF_PACKET`
``` c 
int sfd = socket(PF_PACKET, SOCK_RAW, htons(ETH_P_ALL));
```

##### Gniazdo sieciowe typu `SOCK_PACKET`
``` c
int sfd = socket(PF_INET, SOCK_PACKET, htons(ETH_P_ALL));
```

>Trzeci argument funkcji `socket` oznacza EtherType. W systemach operacyjnych _unix_ oraz _gnu/Linux_ lista wartości EtherType zapisana jest w pliku `if_ether.h` (stałe `ETH_P_*).
> `ETH_P_ALL` -> dowolny protokół sieciowy warstwy trzeciej

To jakie pakiety będzie przechwytywać gniazdo zależy od EtherType jaki zostanie mu nadany. Jeśli chcemy nasłuchiwać cały ruch sieciowy, przejmowaniem każdego protokołu sieciowego warstwy trzeciej (`ETH_P_ALL`), dodatkowo powinniśmy włączyć tryb _promiscuos_ na interfejsie sieciowym z którym powiązane będzie gniazdo.

![[Pasted image 20240315194435.png]]
Kopie wszystkich ramek o podanym EtherType wysyłane przez stos protokołów i interfejs sieciową przekazywane są do gniazda `PF_PACKET`.

Podczas transmitowania i odbierania ramek, oraz wiązania gniazda sieciowego z adresem przy użyciu funkcji systemowej `bind(2)`, wykorzystywana jest struktura [[sockaddr_ll]].

#### Ramki sieciowe kopiowane do gniazda `PF_PACKET`
---
Domyślnie do gniazda sieciowego domeny komunikacyjnej `PF_PACKET` kopiowane są wszystkie transmitowane i odbierane ramki w systemie.

Może to jednak ulec zmianie w zależności od następujących czynników:
- włączenia lub wyłączenia trybu nasłuchiwania
- wartości [[EtherType]] podanej jako trzeci parametr funkcji systemowej [[socket(2)]]
- powiązania gniazda sieciowego z adresem warstwy łącza danych (i konkretnym interfejsem sieciowym) funkcją systemową `bind(2)`.

Podczas wiązania gniazda sieciowego domeny komunikacyjnej `PF_PACKET` z adresem warstwy łącza danych, wartości następujących pól struktury [[sockaddr_ll]] brane są pod uwagę:
- `sll_family` (w tym przypadku z wartością `AF_PACKET`),
- `sll_protocol` 
- `sll_ifindex`
Pozostałe pola mogą zawierać wartość 0

Na ramkach WiFi jest enkapsulacja ramek Ethernet, sterownik sieci WiFi sobie to konwertuje przed wysłaniem i po odebraniu.

#### Przykład
---
##### Transmisja ramki własnej sieci Ethernet
``` c
#define ETH_P_CUSTOM 0x8888
int sfd, ifindex;
char *frame, *fdata;

struct ethhdr* fhead;
struct ifreq ifr;
struct sockaddr_ll sall;

sfd = socket(PF_PACKET, SOCK_RAW, htons(ETH_P_CUSTOM));
frame = malloc(ETH_FRAME_LEN);
memset(frame, 0, ETH_FRAME_LEN);

fhead = (struct ethhdr*) frame;
fdata = frame + ETH_HLEN;
strncpy(ifr.ifr_name, argv[1], IFNAMSIZ);
ioctl(sfd, SIOCGIFINDEX, &ifr);
ifindex = ifr.ifr_ifindex;
ioctl(sfd, SIOCGIFHWADDR, &ifr);

memset(&sall, 0, sizeof(struct sockaddr_ll));
sall.sll_family = AF_PACKET;
sall.sll_protocol = htons(ETH_P_CUSTOM);
sall.sll_ifindex = ifindex;
sall.sll_hatype = ARPHRD_ETHER;
sall.sll_pkttype = PACKET_OUTGOING;
sall.sll_halen = ETH_ALEN;
sscanf(argv[2], "%hhx:%hhx:%hhx:%hhx:%hhx:%hhx",
	&sall.sll_addr[0], &sall.sll_addr[1], &sall.sll_addr[2],
	&sall.sll_addr[3], &sall.sll_addr[4], &sall.sll_addr[5]);
memcpy(fhead->h_dest, &sall.sll_addr, ETH_ALEN);
memcpy(fhead->h_source, &ifr.ifr_hwaddr.sa_data, ETH_ALEN);
fhead->h_proto = htons(ETH_P_CUSTOM);

memcpy(fdata, argv[3], strlen(argv[3]) + 1);
sendto(sfd, frame, ETH_HLEN + strlen(argv[3]) + 1, 0,
		(struct sockaddr*) &sall, sizeof(struct sockaddr_ll));
free(frame); close(sfd);
```

>**Wybrane zmienne***
>`fhead`
>	początek buffora, mapowany na `coś`
>`argv[1]`
>	index interfejsu sieciowego
>`argv[2]`
>	adres MAC odbiorcy
>`argv[3]`
>	przesyłane dane

Jest to operacja na warstwie łącza danych stąd nie ma potrzeby adresu IP.

##### Odbiór ramki własnej sieci Ethernet
``` c
#define ETH_P_CUSTOM 0x8888
int sfd, i; ssize_t len;
char *frame, *fdata;

struct ethhdr* fhead;
struct ifreq ifr;
struct sockaddr_ll sall;

sfd = socket(PF_PACKET, SOCK_RAW, htons(ETH_P_CUSTOM));
strncpy(ifr.ifr_name, argv[1], IFNAMSIZ);
ioctl(sfd, SIOCGIFINDEX, &ifr);

memset(&sall, 0, sizeof(struct sockaddr_ll));
sall.sll_family = AF_PACKET;
sall.sll_protocol = htons(ETH_P_CUSTOM);
sall.sll_ifindex = ifr.ifr_ifindex;
sall.sll_hatype = ARPHRD_ETHER;
sall.sll_pkttype = PACKET_HOST;
sall.sll_halen = ETH_ALEN;

bind(sfd, (struct sockaddr*) &sall,
  sizeof(struct sockaddr_ll));

while(1) {
  frame = malloc(ETH_FRAME_LEN);
  memset(frame, 0, ETH_FRAME_LEN);
  fhead = (struct ethhdr*) frame;
  fdata = frame + ETH_HLEN;
  len = recvfrom(sfd, frame, ETH_FRAME_LEN, 0, NULL, NULL);
  /* ... */
  free(frame);
}
close(sfd);
```
>**Wybrane zmienne**
>`recvfrom()`
>	operacja blokująca 
> #TODO opisać przykład

### Pytania i zadania
---
##### 1. Czy istnieją wartości EtherType mniejsze niż 1536 (0x0600)?
> 	  Nie
> 	  
##### 2. Ramki protokołów sieci bezprzewodowych IEEE 802.11 nie posiadają pola typ, w którym zawarta byłaby wartość EtherType. Na jakiej podstawie rozpoznawany jest zatem w tym przypadku protokół warstwy sieciowej?
>
##### 3. Czy gniazdo sieciowe domeny komunikacyjnej PF_PACKET służące do wysyłania danych można powiązać z adresem przy użyciu funkcji bind(2)? Jeżeli tak, to w jakim celu?
>

#TODO odpowiedzieć na pytania z wykładu