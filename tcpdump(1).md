>	Analizator komunikacji sieciowej `tcpdump(1)` wypisuje na standardowe wyjście informacji o wszystkich nasłuchiwanych ramkach sieciowych, które spełniają podane, jako argument wywołania, wyrażenie filtracji. Wyrażenie filtracji może zostac pominięte jeśli nie chcemy filtrować pakietów. 
##### Przełączniki 
#TODO 

### Przykładowe wyrażenie funkcji filtracji ramek sieciowych
---
`host sundown`
`host helios and \(hot or ace \)` -> komunikacja między komputerami helios z hot lub ace
`src` -> wszystkie ramki z komputera helios do jakiegoś http (80) lub _ (443)
-> ramki transmitujące ip pomięczy ace i nie helios
->
-> pomijanie adresów mac

#### Użycie
`tcpdump -ni eth0 icmp` 

### Filtry BPF i LSF
#### Problem Filtrowania Pakietów
>Jak szybko przejżeć jakie są protokoły w ramce?

Jezyk filtrowania podobny do assemblera. Słowo ma 4 bajty. 

`tcpdump` potrafi to obsługiwać.

#### Przykład 
`tcpdump -d "ip and udp"` -> towrzy kod w psełdo assemlberze do filtracji BPF
`tcpdump -dd "ip and udp` -> fragment kodu w języku C do flitracji

``` C
struct sock_filter code[] = {
// można tu wstawić to co zwróci tcpdump -dd
};

struct sock_fprog bpf = {
	.len = ARRAY_SIZE(code),
	.filter = code
};

sfd = socket(PF_PACKET, SOCK_RAW, htons(ETH_P_ALL));
setsockopt(sfd, SOL_SOCKET, SO_ATTACH_FILTER, &bpf, sizeof(bpf));
```

korzysta z [[libpcap]] (?)