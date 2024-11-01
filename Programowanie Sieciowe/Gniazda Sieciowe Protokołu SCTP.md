---
up:
tags: ProgramowanieSieciowe
---
#TODO

...

### Wymienność protokołów TCP i SCTP
---
...


### Struktura `sctp_sndrcvinfo`
---
``` C
...
```
> _zob._ `netinet/sctp.h`

`sinfo_timetolive` -> czas w milisekundach, w jakim trzeba wysłać dany pakiet. Jeśli w tym czasie się to nie uda, dany pakiet nie jest wysyłany. Jest to użyteczne w serwisach streamingowych i sieciowych grach video.


### Komunikacja wieloma łączami (_multi-homing_) - selekcja łączy
---
``` C
...
```

Operacja dodania lub usunięcia karty sieciowej do wykorzystywanych przez SCTP może zostać wykonane w trakcje działania programu.

### Komunikacja wieloma strumieniami (_multi-streaming_)
---
Aby móc korzystać z _multi-streaming_ wymagane jest włączenie tej opcji w SCTP. Można tego dokonać w następujący sposób:
``` C
memset((void*) &events, 0, sizeof(events));
events.sctp_data_io_event = 1;
setsockopt(sfd, SOL_SCTOP, SCTP_EVENTS, (const void*) &events, sizeof(events));
```

Po tej operacji można korzystać ze strumieni dzięki następującym funkcją:

###### Wysyłanie
``` C
int sctp_sendmsg(...)
```

###### Odbieranie
``` C
int sctp_recvmsg(...)
```

### Gniazda sieciowe wielu asocjacji
---
```
...
```