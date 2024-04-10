#SRC #Sem1 #PS #wykład 

Biblioteka `libpcap` umożliwa nasłuchiwanie oraz filtrowanie ramek sieciowych. Jest ona dostępna dla systemów operacyjnych `unix` i `gnu/Linux` oraz Windows (jako `WinPcap`).

Wymaga jawnego linkowania tej biblioteki (`-lpcap` w przypadku kompilatora gcc)

`*errbuff` -> wstaźnik na buffor w który będą wpisywane ewentualne błędy.

Łatwo można włączyć tryb `promisc` przez pcap_set_promisc:
- wartość powyżej 0 włącza
- wartość poniżej 0 wyłącza

`pcap_set_snaplen` -> rozmiar buffora utrzymywana na każdą odbieraną ramkę. Może być mniej i wtedy biblioteka ucina ten pakiet.
`pcap_set_timeout` -> trzeba ustawić bo bez tego nie będzie działać. Jeśli w tym czasie się nie uda pobrać ramki to wtedy przerywa. Może to mieć miejsce w momencie gdy system mówi że jest coś do odebrania, ale gdy próbujemy coś pobrać to nic tam nie ma.

`pcap_lookupnet` -> pozwala łatwo pobrać adres IP i maskę interfejsu sieciowego.
`pcap_compile` -> gotowa funkcja do skompilowania wyrażenia filtracji (takiego jak w `tcpdump` i wstawia wynik do struktury `bpf_program`)
`pcap_setfilter` -> dodanie filtru do handlera

`pcap_loop` -> podajemy handler z funkcji create, `callback` wskaźnik na funkcje wykonywaną za każdym razem jak złapiemy jakąś ramkę, `cnt` oznacza kiedy przerwać pętle (10 -> po 10 odebrnych ramkach, -1 -> w nieskończoność), `*user` parametr który możemy sobie podać.

`pcap_next_e` -> chce złapać następną ramkę

`calback` powinien mieć taką strukturę:
``` C
typedef void (*pcap_handler)(u_char *user, const struct pcap_pkthdr *h, const u_char *bytes);
```

Jest to opakowanie gniazd `PF_PACKET` o dodatkowe funkcje filtracji.

Można wysyłać ramki poprzez `libpcap` przez `pcap_inject` ale do tego jest inna biblioteka. 

### Przyklad
---
##### Nasłuchiwanie ramek sieciowych
``` C
char* errbuff;
pcap_t* handle;

errbuff = malloc(PACP_ERRBUF_SIZE);
handle = pcap_create(argv[1], errbuff);

```
#TODO 

`signal(SIGINT, stop)` -> jeśli program odbierze `SIGINT` to ma wykonać stop. Kultularna obsługa `Ctrl + C`

`atexit(cleanup)` -> 

prez 18/23 - tak mniejwięcej wygląda implementacja `tcpdump`

