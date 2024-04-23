#SRC #Sem1 #PS 

W systemach operacyjnych `GNU/Linux`, do obsługi pamięci podręcznej protokołu `ARP` dostępne są trzy żądania:
- `SIOCSARP` - dodaje nowy wpis do pamięci podręcznej
- `SIOCDARP` - usuwa istniejący wpis z pamięci podręcznej
- `SIOCGARP` - pobiera informacje o odwzorowaniu z pamięci podręcznej

Dla wszystkich trzech żądań, trzecim argumentem wywołania funkcji systemowej `ioctl(2)` jest wskaźnik na struktórę `arpreq`.
``` C
struct arpreq {
	struct sockaddr arp_pa;       /* protocol address */
	struct sockaddr arp_ha;       /* hardware address */
	int arp_flags;                /* flags */
	struct sockaddr arp_netmask;  /* netmask (only for proxy) */
	char arp_dev[16];
}
```

##### Flagi dla wpisów w pamięci podręcznej protokołu `ARP`
---
``` C
#define ATF_COM			0x02		/* completed entry (ha valid)	*/
#define	ATF_PERM		0x04		/* permanent entry		*/
#define	ATF_PUBL		0x08		/* publish entry		*/
#define	ATF_USETRAILERS	0x10		/* has requested trailers	*/
#define ATF_NETMASK     0x20        /* want to use a netmask (only
					   for proxy entries) */
#define ATF_DONTPUB		0x40		/* don't answer this addresses	*/
```
>publish entry - informacja czy mają być udostępniane
>has requested trailers - jest przestażała i nie używana

>zob. `if_arp.h`

##### Przykład
	wypisanie odpowiedzi arp
---
``` C
int fd;
struct arpreq arq;
struct sockaddr_in* sin;
unsigned char* mac;

memset(&arq, 0, sizeof(arq));
sin = (struct sockaddr_in*) &arq.arp_pa; /* wskazuje na miejsce gdzie ma być IP*/
sin->sin_family = AF_INET;
sin->sin_addr.s_addr = inet_addr(argv[1]);
strcpy(arq.arp_dev, IFNAME);
fd = socket(PF_INET, SOCK_DGRAM, 0);
ioctl(fd, SIOCGARP, &arq);
printf("%s\t", inet_ntoa(sin->sin_addr));

if (arq.arp_flags & ATF_COM) {
	mac = (unsigned char *) &arq.arp_ha.sa_data[0];
	printf("%02x:%02x:%02x:%02x:%02x:%02x",
		mac[0], mac[1], mac[2], mac[3], mac[4], mac[5]);
	print("\tC");
	if (arq.arp_flags & ATF_PERM) printf("M");
	if (arq.arp_flags & ATF_PUBL) printf("MP"); 
	if (arq.arp_flags & ATF_USETRAILERS) printf("T"); 
	if (arq.arp_flags & ATF_NETMASK) printf("N");  
} else
	printf("(incomplete)");
printf("\n");
close(fd);
```
