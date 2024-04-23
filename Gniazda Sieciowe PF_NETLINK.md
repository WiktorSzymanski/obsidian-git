#SRC #Sem1 #PS 

[[Gniazda Sieciowe]] `PF_NETLINK` umożliwiają wymianę informacji (komunikatów) pomiędzy jądrem systemu operacyjnego a procesami przestrzeni użytkownika.

>Nie są to takie same gniazda sieciowe jak pozostałe, te nie służą do komunikacji poza pojedyńczy komputer.

Mechanizm 

>Można je tak skonfigurować aby na pewne _wydarzenie_ w systemie operacyjnym, wykonywała się pewna operacja. Można je wykorzystać do szukania problemów w systemie operacyjnym.

##### Przykład utworzenia gniazda sieciowego `PF_NETLINK`
---
``` C
int sfd = socket(PF_NETLINK, SOCK_RAW, netlink_family);
# lub
int sfd = socket(PF_NETLINK, SOCK_DGRAM, netlink_family);
```

#TODO prez 15/32

>`netlink_family` -> moduł jądra z którego chcemy skorzystać


```
nlmsghdr | 
```
### Struktura adresowa `aockaddr_nl`
---
Gniazda sieciowe `PF_NETLINK` mogą zostać powiązane z adresem procesu (funkcją systemową `bind(2)`), który określany jest w strukturze `sockaddr_nl`.

#TODO prez 16/32

>_zob._ `netlink.h`

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