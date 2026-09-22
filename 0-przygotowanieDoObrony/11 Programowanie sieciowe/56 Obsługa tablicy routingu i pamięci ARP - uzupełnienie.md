---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 56
---
# 56. Mechanizmy obsługi tablicy routingu i pamięci podręcznej ARP – uzupełnienie
---
> Uzupełnienie [[Programowanie Sieciowe/Tablica Routingu]], [[Programowanie Sieciowe/Pamięć Podręczna Protokołu ARP]] i [[Programowanie Sieciowe/ioctl(2)]] o wiele tablic routingu i porównanie mechanizmów programistycznych.

## Trzy sposoby programistycznej obsługi
| Mechanizm | Tablica routingu | Pamięć ARP | Uwagi |
|---|---|---|---|
| **`ioctl(2)`** ze strukturą `rtentry` / `arpreq` | `SIOCADDRT`, `SIOCDELRT` | `SIOCSARP`, `SIOCDARP`, `SIOCGARP` | prosty, tylko główna tablica, przestarzały dla routingu |
| **`PF_NETLINK` / rtnetlink** (`NETLINK_ROUTE`) | `RTM_NEWROUTE`, `RTM_DELROUTE`, `RTM_GETROUTE` | `RTM_NEWNEIGH`, `RTM_DELNEIGH`, `RTM_GETNEIGH` | pełne możliwości: wiele tablic, powiadomienia o zmianach, IPv6 – [[58 Gniazda sieciowe PF_NETLINK - uzupełnienie]] |
| pliki `/proc/net/route`, `/proc/net/arp` + komendy `ip route`, `ip neigh`, `arp` | odczyt | odczyt | diagnostyka |

## Tablica routingu – `ioctl` i `rtentry`
Struktura `rtentry`, flagi (`RTF_UP`, `RTF_GATEWAY`, `RTF_HOST`…) i przykład dodania bramy domyślnej: [[Programowanie Sieciowe/Tablica Routingu]]. Wpis zawiera: sieć docelową, maskę, bramę (następny skok), interfejs, metrykę, flagi.

## Wiele tablic routingu (policy routing)
Współczesne jądra Linux mają **wiele tablic** i **reguły** (`ip rule`) decydujące, której użyć – na podstawie adresu źródłowego, znacznika (fwmark), interfejsu wejściowego.
- Tablice domyślne (`/etc/iproute2/rt_tables`): **`local`** (254 – adresy lokalne i broadcast, zarządzana przez jądro – nic nie dopisywać), **`main`** (253 – zwykłe trasy, domyślnie pokazywana przez `ip route show`), **`default`** (255).
- `ip rule` – lista reguł z priorytetami; pierwsza pasująca wskazuje tablicę.
- Zastosowania: **routing na podstawie źródła** (różne bramy dla różnych podsieci), rozdzielenie ruchu między łącza (multihoming), VPN, VRF.
```bash
echo "100 firma" >> /etc/iproute2/rt_tables
ip route add 10.0.0.0/24 dev eth1 src 10.0.0.2 table firma
ip rule add from 10.0.0.2 table firma        # ruch z tego IP używa tablicy "firma"
ip route show table firma
```
Programowo tablice obsługuje się przez **rtnetlink** (pole `rtm_table` w `struct rtmsg`), nie przez `ioctl`. Szczegóły: [[Programowanie Sieciowe/ip(8)]].

## Pamięć podręczna ARP
Odwzorowanie IP → MAC; wpisy, flagi (C/M/P), `/proc/net/arp`: [[Programowanie Sieciowe/Pamięć Podręczna Protokołu ARP]].
- `ioctl` z `arpreq`: `SIOCGARP` (odczyt), `SIOCSARP` (dodanie – flaga `ATF_PERM` dla wpisu stałego, `ATF_PUBL` dla proxy ARP), `SIOCDARP` (usunięcie) – przykład wypisania odpowiedzi ARP: [[Programowanie Sieciowe/ioctl(2)]].
- **Tablica sąsiadów** (_neighbour table_) – uogólnienie ARP (IPv4) i **NDP** (IPv6, komunikaty ICMPv6: Neighbor Solicitation/Advertisement); obsługiwana przez rtnetlink (`RTM_*NEIGH`), stany wpisu: `INCOMPLETE`, `REACHABLE`, `STALE`, `DELAY`, `PROBE`, `FAILED`, `PERMANENT`.
- Bezpieczeństwo: **ARP spoofing** – obrona przez wpisy stałe (`arp -s`, flaga PERM), monitoring (arpwatch), DAI na przełącznikach.
```bash
ip neigh show                         # tablica sąsiadów (ARP/NDP)
ip neigh add 10.0.0.5 lladdr aa:bb:cc:dd:ee:ff dev eth0 nud permanent
ip -6 neigh show
```

## Zobacz też
- [[57 Gniazda sieciowe PF_PACKET - uzupełnienie]], [[58 Gniazda sieciowe PF_NETLINK - uzupełnienie]]
