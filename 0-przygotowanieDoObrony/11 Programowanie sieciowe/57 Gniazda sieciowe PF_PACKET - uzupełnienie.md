---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 57
---
# 57. Gniazda sieciowe PF_PACKET – uzupełnienie
---
> Uzupełnienie [[Programowanie Sieciowe/Gniazda Sieciowe PF_PACKET]] (tworzenie, SOCK_RAW vs SOCK_DGRAM, przykłady nadawania/odbioru, `sockaddr_ll`) o odpowiedzi na pytania z zajęć i mechanizmy odbioru wysokiej wydajności.

## Przypomnienie
`PF_PACKET` daje dostęp do **warstwy łącza danych** (warstwa 2) – odbiór i wysyłanie **całych ramek** (Ethernet). Wymaga uprawnień **`CAP_NET_RAW`** (root). Tworzenie:
```c
int sfd = socket(PF_PACKET, SOCK_RAW,   htons(ETH_P_ALL));  // z nagłówkiem łącza
int sfd = socket(PF_PACKET, SOCK_DGRAM, htons(ETH_P_ALL));  // bez nagłówka łącza ("cooked")
```
- **SOCK_RAW** – dane zawierają **nagłówek warstwy łącza** (Ethernet); nadawca musi go zbudować,
- **SOCK_DGRAM** – jądro **usuwa/dodaje** nagłówek łącza, adresowanie ze struktury `sockaddr_ll`,
- trzeci argument = **EtherType** (`ETH_P_ALL` – wszystkie protokoły, `ETH_P_IP`, `ETH_P_ARP`…) – filtruje, co trafia do gniazda.
Struktura adresowa `sockaddr_ll`, typy ramek (`PACKET_HOST`, `PACKET_OUTGOING`…): [[Programowanie Sieciowe/sockaddr_ll]].

## Odpowiedzi na pytania z zajęć
1. **Czy istnieją wartości EtherType < 1536 (0x0600)?** – Nie jako EtherType. W ramce Ethernet II pole o wartości **> 1500** to EtherType, a **≤ 1500** oznacza **długość** danych (ramka IEEE 802.3 z nagłówkiem LLC/SNAP). Zakres 1501–1535 jest niezdefiniowany. Zob. [[Programowanie Sieciowe/Ramka Sieci Ethernet II]].
2. **Ramki 802.11 (Wi-Fi) nie mają pola EtherType – jak rozpoznać protokół warstwy sieciowej?** – po nagłówku **LLC/SNAP** (RFC 1042) doklejanym w ładunku ramki 802.11: pole **OUI** i **typ** w SNAP pełnią rolę EtherType. Sterownik Wi-Fi zwykle konwertuje ramki do postaci Ethernet II przed przekazaniem wyżej.
3. **Czy gniazdo PF_PACKET do wysyłania można powiązać `bind(2)`? W jakim celu?** – Tak. `bind` ze strukturą `sockaddr_ll` (pola `sll_family`, `sll_protocol`, `sll_ifindex`) **przypina gniazdo do konkretnego interfejsu** i EtherType – wtedy odbierane są ramki tylko z tego interfejsu, a przy wysyłaniu `sendto` można pominąć część adresu. Bez `bind` gniazdo odbiera ze wszystkich interfejsów.

## Filtrowanie – BPF / LSF
Dołączenie **filtra pakietów** (Berkeley Packet Filter) do gniazda przenosi filtrowanie do **jądra** (mniej kopiowania do przestrzeni użytkownika):
```c
setsockopt(sfd, SOL_SOCKET, SO_ATTACH_FILTER, &bpf, sizeof(bpf));
```
Kod filtra można wygenerować z wyrażenia `tcpdump` (`tcpdump -dd "ip and tcp"`) – zob. [[Programowanie Sieciowe/tcpdump(1)]]. Biblioteka **libpcap** opakowuje PF_PACKET i BPF – [[Programowanie Sieciowe/libpcap]]. **eBPF** (`SO_ATTACH_BPF`) – rozszerzone filtry i XDP (przetwarzanie ramek w sterowniku, przed stosem sieciowym).

## Tryb nasłuchiwania i odbiór wydajny
- **Tryb promiscuous** dla `ETH_P_ALL` (przechwytywanie całego ruchu segmentu): włączenie przez `SIOCSIFFLAGS`/`IFF_PROMISC` lub czyściej opcją `PACKET_ADD_MEMBERSHIP` (`PACKET_MR_PROMISC`) – automatyczne wyłączenie przy zamknięciu gniazda.
- **PACKET_MMAP** (`PACKET_RX_RING` / `PACKET_TX_RING`) – **współdzielony bufor pierścieniowy** (mmap) między jądrem a aplikacją: ramki odbierane/wysyłane bez `recvfrom`/`sendto` (bez kopiowania i przełączeń kontekstu) → wysoka wydajność (analizatory, generatory ruchu).
- **AF_PACKET v3 (TPACKET_V3)**, **PACKET_FANOUT** – rozkładanie ruchu na wiele gniazd/wątków (hash, round-robin) dla przetwarzania wielordzeniowego.
- Wysyłanie: `sendto`/`send`; z TX_RING – wypełnianie ramek w buforze i `send` wyzwalający transmisję.

## Zastosowania
analizatory (Wireshark/tcpdump przez libpcap), własne implementacje protokołów poza jądrem, generatory i wstrzykiwanie ramek (testy, ataki), narzędzia bezpieczeństwa/IDS, DHCP/PXE, mostkowanie w przestrzeni użytkownika. Alternatywa niskopoziomowa: [[59 Nieprzetworzone gniazda sieciowe - uzupełnienie]] (warstwa 3, IP).

## Zobacz też
- [[Programowanie Sieciowe/Gniazda Sieciowe]], [[Programowanie Sieciowe/EtherType]]
