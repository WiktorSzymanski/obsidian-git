---
up: "[[Gniazda Sieciowe]]"
tags: ProgramowanieSieciowe
---
# Gniazda Sieciowe PF_NETLINK
---
[[Gniazda Sieciowe]] `PF_NETLINK` umożliwiają wymianę informacji (komunikatów) pomiędzy jądrem systemu operacyjnego a procesami przestrzeni użytkownika.

>Nie są to takie same gniazda sieciowe jak pozostałe, te nie służą do komunikacji poza pojedynczy komputer.

Mechanizm 

>Można je tak skonfigurować aby na pewne _wydarzenie_ w systemie operacyjnym, wykonywała się pewna operacja. Można je wykorzystać do szukania problemów w systemie operacyjnym.

##### Przykład utworzenia gniazda sieciowego `PF_NETLINK`
---
``` C
int sfd = socket(PF_NETLINK, SOCK_RAW, netlink_family);
# lub
int sfd = socket(PF_NETLINK, SOCK_DGRAM, netlink_family);
```
Stałe, które mogą być trzecim argumentem wywołania funkcji systemowej [[socket(2)]] dla gniazd sieciowych _PF_NETLINK_ zdefiniowane są w pliku `linux/netlink.h`.

>`netlink_family` -> moduł jądra z którego chcemy skorzystać


```
nlmsghdr | 
```
### Struktura adresowa `aockaddr_nl`
---
Gniazda sieciowe `PF_NETLINK` mogą zostać powiązane z adresem procesu (funkcją systemową `bind(2)`), który określany jest w strukturze `sockaddr_nl`.

``` C
struct sockaddr_nl {
	sa_family_t nl_family; /* AF_NETLINK */
	unsigned short nl_pad; /* Zero */
	__u32 nl_pid; /* Process PID */
	__u32 nl_groups; /* Multicast groups maska */
};
```
>_zob._ `netlink.h`

Nagłówki komunikatów przesyłanych gniazdami sieciowymi _PF_NETLINK_ opisane są strukturą 
``` C
struct nlmsghdr {
	__u32 nlmsq_len; /* Length of message including header */
	__u16 nlmsq_type; /* Type of message content */
	__u16 nlmsq_flags; /* Additional flags */
	__u32 nlmsq_seq; /* Sequence number */
	__u32 nlmsq_pid; /* Sender port ID */
}
```
#TODO prez 17/32

>Jądro może nam coś wysłać "pocięte", stąd "sequence number".


### Gniazda obsługi tablic routingu `NETLINK_ROUTE`
---
``` C
int sfd = socket(PF_NETLINK, SOCK_RAW, NETLINK_ROUTE);
```

Gniazda obsługi tablic routingu `NETLINK_ROUTE` (nazywane _rtnetlink_) umożliwiają obsługę nie tylko tablic routingu, ale także m.in. adresacji `IP`, ustawień interfejsó sieciowych, pamięci podręcznej protokołu `ARP` oraz mechanizmów kolejkowania.

#TODO prez 19/32

#TODO prez 20/32

>`rtm_dst_len` i `rtm_src_len` to są maski w postaci dziesiętnej.

```
nlmsghdr | rtmsg | 
```

#TODO prez 21/32

```
nlmsghdr | rtmsg | rtattr DANE 
```

>struktury `rtattr DANE` mogą się kolejnować w nieskończoność

#TODO prez 22/32
#TODO prez 23/32
#TODO prez 24/32

#### Przykład 
---
``` C
#TODO prez 25-27/32
```

>Kiedyś robili `OS` tak że dopisywało się do pliku, nowe podejście - wywyłanie wiadomości do jądra systemu. Nowe rozwiązanie giga skomplikowane i wgl nie wygodne.

#TODO na pytania odpowiedzieć 31/32