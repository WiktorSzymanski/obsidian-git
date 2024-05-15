#SRC #Sem1 #PS #TODO

##### Gniazda strumieniowe TCP
``` C
int socket(PF_INET6, SOCK_STREAM, 0); 
```

##### Gniazda datagramowe UDP
``` C
int socket(PF_INET6, SOCK_DGRAM, 0); 
```

...

### Struktura adresowa `sockaddr_in6`
---
``` C
struct sockaddr_in6 {
	unsigned short int sin6_family; /* AF_INET6 */
	...
	... flowinfo /* (jest ignorowane) */
	...
	... scope_id /* przy kożystaniu z link local tutaj wpisujemy ID interfejsu sieciowego (%eth)*/
}
```

```C
struct in6_addr {
	__u8 s6_addr[16]; /* IPv6 address */
}
```