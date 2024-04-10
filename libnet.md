#SRC #Sem1 #PS

#TODO z prez 4/26

Pozwala bez znajomości protokołów wykorzystywanie ich przy pomocy tak naprawdę jednej funkcji.

Możemy zbudować poprawną ramkę z pakietami baz wyższych i wprowadzać w nich modyfikacje. Pozwala na tworzenie wielu ataków sieciowych. Pozwala szybko tworzyć oprogramowanie wykorzystujące standardowe protokoły, bez ich dogłębnego zrozumienia.

Przy kompilacji trzeba jawnie zlinkować bibliotekę `-libnet`


#### Schemat konstruowania ramek
---
#TODO prez 7/26
Ze wzgledu na sumy kontrolne musimy budować ramkę w **w kolejności od warstwy najwyższej do warstwy najniższej**.

`libnet_build_*()` - wskazujemy wszystkie/prawie wszystkie wartości nagłówków
`libnet_autobuild_*()` - wymaga wstazanie tylko tych nagłówków których biblioteka sama nie potrafi sobie pobrać

#### Inicjacja biblioteki `libnet`
---
``` c
libnet_t* libnet_init(int intinjection_type, const char *device, char *err_buf)
```
>`injection_type` - określa typ transmisji.
>`device`
>`err_buff`

##### Typy transmisji #TODO prez 10/26

#### Przykład użycia
---
``` C
libnet_t *l;
char err_buff[LIBNET_ERRBUF_SIZE];

l = libnet_init(LIBNET_RAW4, argv[1], errbuf);
if (l == NULL) {
	fprintf(stderr, )
}

#TODO 
```

#### Funkcje tworzące pakiety
---
#TODO prez 12/26


Jeśli w miejsce sumy kontrolnej podamy wartość inną jak 0 to wtedy będzie to wartość tej sumy kontrolnej.


`libnet-functions.h` - zawiera deklaracje funkcji do budowania ramek

`trace-root` - korzysta z TTL do wyznaczania routerów na trasie pakietu.

#TODO Pytania i zadania prez 25/26


`Scapy` - biblioteka do pythona, odpowiednik libnet i libpcap dla pythona