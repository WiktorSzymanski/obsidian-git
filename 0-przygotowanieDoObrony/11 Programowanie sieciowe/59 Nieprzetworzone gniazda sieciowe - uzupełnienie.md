---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 59
---
# 59. Nieprzetworzone gniazda sieciowe – uzupełnienie
---
> Uzupełnienie [[Nieprzetworzone Gniazda Sieciowe]] i [[Programowanie Sieciowe/Struktura nagłówka pakietów IPv4]] o pełny przykład transmisji, opcję `IP_HDRINCL`, uprawnienia i porównanie poziomów gniazd.

## Czym są nieprzetworzone (surowe) gniazda
**Gniazda surowe** (`SOCK_RAW`) dają dostęp do **warstwy sieciowej (3 – IP)**, z pominięciem warstwy transportowej (TCP/UDP). Pozwalają:
- odbierać pakiety wybranego protokołu IP wraz z **nagłówkiem IP**,
- **budować własne pakiety** dowolnego protokołu (ICMP, własne protokoły warstwy 3, niestandardowe nagłówki),
- implementować protokoły poza jądrem (np. OSPF, narzędzia jak `ping`, `traceroute`, skanery).

Wymagają uprawnień **`CAP_NET_RAW`** (root). Poziom niższy (warstwa 2, ramki) to `PF_PACKET` – [[57 Gniazda sieciowe PF_PACKET - uzupełnienie]].

```c
int s = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);   // gniazdo surowe dla ICMP
int s = socket(AF_INET, SOCK_RAW, IPPROTO_RAW);     // tylko do wysyłania; implikuje IP_HDRINCL
```
Trzeci argument = **numer protokołu** (`/etc/protocols`, `IPPROTO_ICMP`, `IPPROTO_TCP`, własny). Wartość `0` – wszystkie protokoły, których jądro „nie obsługuje samodzielnie”, trafiają do gniazda.

## Odbiór
- Odebrane dane **zawsze zawierają kompletny nagłówek IP** (jądro go dołącza) – stąd trzeba znać jego strukturę: [[Programowanie Sieciowe/Struktura nagłówka pakietów IPv4]] (pola `ihl`, `ttl`, `protocol`, `check`, `saddr`, `daddr`; przesunięcie do danych = `ihl * 4` bajtów).
- Jądro dostarcza do gniazda surowego kopie pakietów **wybranego protokołu**; pakiety TCP/UDP obsługiwane samodzielnie przez jądro **nie** trafiają do surowego gniazda `IPPROTO_TCP/UDP` w standardowy sposób (stąd sniffery używają `PF_PACKET`/libpcap).
- Jeden pakiet może trafić do **wielu** gniazd (surowych i normalnych).
- ICMP: gniazdo `IPPROTO_ICMP` odbiera komunikaty ICMP (echo reply w `ping`).

## Wysyłanie i `IP_HDRINCL`
- **Domyślnie** jądro **samo buduje nagłówek IP** – aplikacja podaje tylko dane (ładunek protokołu warstwy 3), a adres docelowy w `sendto`. Jądro wypełnia adres źródłowy, TTL, sumę kontrolną, id.
- Opcja **`IP_HDRINCL`** (`setsockopt(s, IPPROTO_IP, IP_HDRINCL, &on, ...)`, `on = 1`) → **aplikacja dołącza własny nagłówek IP**. Umożliwia sfałszowanie adresu źródłowego (spoofing), ustawienie TTL, flag, opcji. Jądro może uzupełnić sumę kontrolną IP i id, jeśli zerowe. `IPPROTO_RAW` implikuje `IP_HDRINCL`.
- Sumę kontrolną **warstwy wyższej** (ICMP, pseudo-nagłówek TCP/UDP) liczy aplikacja.

## Przykład – wysłanie pakietu IPv4 z własnym nagłówkiem (ICMP echo)
```c
int s = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);
int on = 1;
setsockopt(s, IPPROTO_IP, IP_HDRINCL, &on, sizeof(on));   // sami budujemy nagłówek IP

char packet[4096]; memset(packet, 0, sizeof(packet));
struct iphdr   *ip  = (struct iphdr*)   packet;
struct icmphdr *icmp = (struct icmphdr*)(packet + sizeof(struct iphdr));

/* nagłówek IP */
ip->version  = 4;
ip->ihl      = 5;                                  /* 5 * 4 = 20 B */
ip->tos      = 0;
ip->tot_len  = htons(sizeof(struct iphdr) + sizeof(struct icmphdr));
ip->id       = htons(12345);
ip->frag_off = 0;
ip->ttl      = 64;
ip->protocol = IPPROTO_ICMP;
ip->saddr    = inet_addr("192.168.1.10");          /* można sfałszować */
ip->daddr    = inet_addr("192.168.1.1");
ip->check    = 0;                                  /* jądro uzupełni lub liczymy sami */

/* nagłówek ICMP echo request */
icmp->type   = ICMP_ECHO;                          /* 8 */
icmp->code   = 0;
icmp->un.echo.id       = htons(getpid());
icmp->un.echo.sequence = htons(1);
icmp->checksum = 0;
icmp->checksum = checksum(icmp, sizeof(struct icmphdr));  /* suma ICMP liczona przez nas */

struct sockaddr_in dst = { .sin_family = AF_INET, .sin_addr.s_addr = ip->daddr };
sendto(s, packet, ntohs(ip->tot_len), 0, (struct sockaddr*)&dst, sizeof(dst));
close(s);
```
(`checksum` – standardowa suma uzupełnieniowa do 1: suma 16-bitowych słów z zawinięciem przeniesień, negacja.)

## Żądania `ioctl` i opcje
- `FIOASYNC` – mimo nazwy włącza tryb **nieblokujący/sterowany sygnałem** (`SIGIO`), nie asynchroniczny w sensie POSIX AIO,
- `SIOCGSTAMP` – znacznik czasu odbioru pakietu,
- opcje `setsockopt`: `IP_HDRINCL`, `IP_TTL`, `IP_OPTIONS`, `SO_BINDTODEVICE` (przypięcie do interfejsu), `ICMP_FILTER`.

## Biblioteki wspomagające
- **libnet** – budowanie i wstrzykiwanie pakietów wszystkich warstw bez znajomości szczegółów (`libnet_build_*`, kolejność od warstwy najwyższej ze względu na sumy kontrolne) – [[Programowanie Sieciowe/libnet]],
- **libpcap** – przechwytywanie (odbiór) – [[Programowanie Sieciowe/libpcap]],
- **Scapy** (Python) – odpowiednik libnet + libpcap.

## Porównanie poziomów gniazd
| Poziom | Domena / typ | Warstwa | Dane zawierają | Uprawnienia | Zastosowanie |
|---|---|---|---|---|---|
| podstawowe | `AF_INET`, `SOCK_STREAM`/`SOCK_DGRAM` | transportowa (TCP/UDP) | ładunek aplikacji | brak (dowolny użytkownik) | zwykłe aplikacje sieciowe |
| **surowe** | `AF_INET`, **`SOCK_RAW`** | **sieciowa (IP)** | **nagłówek IP + dane** | `CAP_NET_RAW` | ping, traceroute, własne protokoły L3, skanery |
| **PF_PACKET** | `PF_PACKET`, `SOCK_RAW`/`SOCK_DGRAM` | **łącza danych (L2)** | ramka Ethernet | `CAP_NET_RAW` | sniffery, wstrzykiwanie ramek, protokoły L2 |
| netlink | `PF_NETLINK` | jądro ↔ user | komunikaty konfiguracji | zależnie | konfiguracja sieci – [[58 Gniazda sieciowe PF_NETLINK - uzupełnienie]] |

Podstawowe gniazda i `socket(2)`: [[Programowanie Sieciowe/Podstawowe Gniazda Sieciowe]], [[Programowanie Sieciowe/socket(2)]].

## Zastosowania i bezpieczeństwo
`ping`/`traceroute` (ICMP, TTL), skanery portów (nmap – własne pakiety SYN), implementacje protokołów routingu, narzędzia diagnostyczne i bezpieczeństwa; **ataki**: spoofing adresu źródłowego (`IP_HDRINCL`), SYN flood, ICMP flood – dlatego wymagane uprawnienia roota i filtrowanie na zaporach.

## Zobacz też
- [[60 Obsługa operacji wejścia-wyjścia komunikacji sieciowej]], [[Programowanie Sieciowe/Gniazda Sieciowe]]
