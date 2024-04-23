#SRC #Sem1 #PS

Tablica routingu zawiera adresy `IP` sąsiednich (tj. działających w tej samej sieci `IP`) routerów, przez które wiedzie trasa do oddalonych (tj. niedostępnych bezpośrednio) sieci `IP`.

Każdy wpis w tablicy routingu zawiera m.in. następujące informacje:
- adres `IP` sieci docelowej
- adres `IP` routera sąsiedniego na trasie do sieci docelowej
- maskę adresu `IP` sieci docelowej
- flagi
- metrykę trasy ("odległość" od sieci docelowej)
- liczbę referencji do danej trasy (wartość ta nie jest wykorzystywana przez jądro systemu operacyjnego `GNU/Linux`)
- liczbę trafień/chybień zapytań do pamięci podręcznej tablicy routingu.
>zob. `route(8)`

Zawartość tablicy routingu dostępna jest takżę w pliku `/proc/net/route`.
>zob. `proc(5)`

>Metryka odregłości sieci `RIP` to ilość przeskoków (przejścia przez routery) do sieci docelowej. Istnieją inne metryki jak np. `OSPF`, które biorą pod uwagę przepustowość łączy.

#### Flagi dla wpisów w tablicy routingu
---
- U - "route up"
- H - "target is a host"
- G - "use gateway" : kiedy nie jest bezpośrednio dostępny ale mamy dla niego bramkę
- R - "reinstate route for dynamic routing" : wpis jest dodany ale musi być odświeżany
- D - "dynamically installed by deamon or redirect" : wpis dodany przez protokół a nie administratora
- M - "modified from routing deamon or redirect" : zmodyfikowana przez protokół routingu dynamicznego
- A - "installed by addrconf"
- C - "cache entry"
- ! - "reject route" : Do danej sieci nie chcemy używać danej bramy

Akceptacja komunikatów `ICMP redirect` wymaga ustalenia wartości `1` w pliku `/proc/sys/net/ipv4/conf/*/accept_redirects`.

>Redirect host jest kiedy są dwa komputery podłączone do jednego routera ale są w dwóch sieciach wirtualnych.

>`Destination` o wartości `0.0.0.0` wypisane przez `route -n` oznacza bramę domyślną.

Komenda `route -Cn` wyświetla adresy zapisane w pamięci `cache`. Są one brane z pliku `/proc/net/rt_cache` i wyświetlane w czytelny sposób.

---

W systemach operacyjnych `GNU/Linux`, do obsługi tablicy routingu dostępne są następujące żądania:
- `SIOCADDRT` - dodaje nowy wpis do tablicy routingu
- `SIOCDELRT` - usuwa wpis z tablicy routingu
Do obu żądań trzecim argumentem wywołania funkcji `ioctl(2)` jest wskaźnik na strukturę `rtentry`.
>zob. `route.h`

>Struktura `rtentry` posiada wiele argumentów współcześnie nie używanych, ale potrzebnych dla kompatybilności wstecznej. 

``` C
struct rtentry {
	unsigned long	rt_pad1;
	struct sockaddr	rt_dst;		/* target address */
	struct sockaddr	rt_gateway;	/* gateway addr (RTF_GATEWAY) */
	struct sockaddr	rt_genmask;	/* target network mask (IP)	*/
	unsigned short	rt_flags;
	short			rt_pad2;
	unsigned long	rt_pad3;
	void			*rt_pad4;
	short			rt_metric;	/* +1 for binary compatibility! */
	char __user		*rt_dev;	/* forcing the device at add */
	unsigned long	rt_mtu;		/* per route MTU/Window */
#ifndef __KERNEL__
#define rt_mss	rt_mtu			/* Compatibility :-( */
#endif
	unsigned long	rt_window;	/* Window clamping 		*/
	unsigned short	rt_irtt;	/* Initial RTT			*/
};
```

##### Flagi dla wpisów do tablicy routingu
---
``` C
#define	RTF_UP			0x0001		/* route usable		  	*/
#define	RTF_GATEWAY		0x0002		/* destination is a gateway	*/
#define	RTF_HOST		0x0004		/* host entry (net otherwise)	*/
#define RTF_REINSTATE	0x0008		/* reinstate route after tmout	*/
#define	RTF_DYNAMIC		0x0010		/* created dyn. (by redirect)	*/
#define	RTF_MODIFIED	0x0020		/* modified dyn. (by redirect)	*/
#define RTF_MTU			0x0040		/* specific MTU for this route	*/
#define RTF_MSS			RTF_MTU		/* Compatibility :-(		*/
#define RTF_WINDOW		0x0080		/* per route window clamping	*/
#define RTF_IRTT		0x0100		/* Initial round trip time	*/
#define RTF_REJECT		0x0200		/* Reject route			*/
```

##### Przykład
	dodanie bramy domyślnej/dodanie trasy do tablicy routingu
---
``` C
int fd;
struct rtentry route;
struct sockaddr_in* addr;

fd = socket(AF_INET, SOCK_DGRAM, 0);

memset(&route, 0, sizeof(route));

addr = (struct sockaddr_in*) &route.rt_gateway;
addr->sin_family = AF_INET;
addr->sin_addr.s_addr = inet_addr("192.168.1.1");

addr = (struct sockaddr_in*) &route.rt_dst;
addr->sin_family = AF_INET;
addr->sin_addr.s_addr = INADDR_ANY;

addr = (struct sockaddr_in*) &route.rt_genmask;
addr->sin_family = AF_INET;
addr->sin_addr.s_addr = INADDR_ANY;

route.rt_flags = RTF_UP | RTF_GATEWAY;

route.rt_metric = 0;

ioctl(fd, SIOCADDRT, &route);

close(fd);
```

### Tablice routingu w systemach operacyjnych `GNU/Linux`
---
Współczesne systemy operacyjne `GNU/Linux` wspierają wiele tablic routingu.

>Nie chcemy nic dopisywać do `local`
>Domyśla tablica routingu `main`

#TODO prez 4/32

##### Tablica `local`

##### Tablica `main`

#TODO prez 5/32

