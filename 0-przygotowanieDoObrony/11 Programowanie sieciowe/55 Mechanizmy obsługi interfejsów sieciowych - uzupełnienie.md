---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 55
---
# 55. Mechanizmy programistyczne obsługi interfejsów sieciowych – uzupełnienie
---
> Uzupełnienie [[Programowanie Sieciowe/Obsługa Interfejsów Sieciowych]] o pełną listę żądań `ioctl(2)`, strukturę `ifreq`, funkcję `getifaddrs(3)` oraz odpowiedzi na pytania z zajęć.

## Sposoby obsługi interfejsów z poziomu programu
1. **`ioctl(2)`** na deskryptorze zwykłego gniazda (`AF_INET`, `SOCK_DGRAM`) – klasyczny, przenośny w rodzinie UNIX, oparty na strukturze `ifreq`.
2. **`getifaddrs(3)`** – wygodne pobranie listy wszystkich interfejsów i ich adresów.
3. **gniazda `PF_NETLINK` / rtnetlink** (`NETLINK_ROUTE`) – natywny mechanizm Linuksa, zdarzenia o zmianach – [[58 Gniazda sieciowe PF_NETLINK - uzupełnienie]].
4. pliki w **`/proc/net`** i **`/sys/class/net/<if>/`** (odczyt stanu, statystyk, flag).
5. komendy zewnętrzne: `ip(8)` ([[Programowanie Sieciowe/ip(8)]]), `ifconfig(8)`, `ethtool(8)`, `iw(8)`.

## Struktura `ifreq` i żądania `ioctl`
```c
struct ifreq {
    char ifr_name[IFNAMSIZ];        /* nazwa interfejsu, np. "eth0" */
    union {
        struct sockaddr ifr_addr;   /* adres */
        struct sockaddr ifr_netmask;
        struct sockaddr ifr_hwaddr; /* adres MAC */
        short  ifr_flags;           /* flagi */
        int    ifr_ifindex;         /* indeks interfejsu */
        int    ifr_mtu;
        char   ifr_slave[IFNAMSIZ];
        char*  ifr_data;
        /* ... */
    };
};
```
| Żądanie | Działanie |
|---|---|
| `SIOCGIFINDEX` / `SIOCGIFNAME` | indeks ↔ nazwa interfejsu |
| `SIOCGIFADDR` / `SIOCSIFADDR` | pobierz / ustaw adres IP |
| `SIOCGIFNETMASK` / `SIOCSIFNETMASK` | maska podsieci |
| `SIOCGIFBRDADDR` / `SIOCSIFBRDADDR` | adres rozgłoszeniowy |
| `SIOCGIFDSTADDR` | adres drugiego końca (łącza punkt-punkt) |
| `SIOCGIFFLAGS` / `SIOCSIFFLAGS` | flagi: `IFF_UP`, `IFF_BROADCAST`, `IFF_LOOPBACK`, `IFF_POINTOPOINT`, **`IFF_PROMISC`**, `IFF_MULTICAST`, `IFF_RUNNING` |
| `SIOCGIFHWADDR` / `SIOCSIFHWADDR` | adres sprzętowy (MAC) |
| `SIOCGIFMTU` / `SIOCSIFMTU` | MTU |
| `SIOCGIFCONF` | lista wszystkich interfejsów i adresów (tablica `ifreq`) |
| `SIOCADDMULTI` / `SIOCDELMULTI` | adresy multicast warstwy łącza |
>zob. `netdevice(7)`, `ioctl(2)`. Zmiana konfiguracji (`SIOCS*`, `IFF_PROMISC`) wymaga uprawnień (`CAP_NET_ADMIN`).

```c
/* pobranie MAC i indeksu interfejsu */
int fd = socket(AF_INET, SOCK_DGRAM, 0);
struct ifreq ifr; memset(&ifr, 0, sizeof(ifr));
strncpy(ifr.ifr_name, "eth0", IFNAMSIZ - 1);
ioctl(fd, SIOCGIFINDEX,  &ifr);   int idx = ifr.ifr_ifindex;
ioctl(fd, SIOCGIFHWADDR, &ifr);   unsigned char *mac = (unsigned char*) ifr.ifr_hwaddr.sa_data;
ioctl(fd, SIOCGIFFLAGS,  &ifr);
ifr.ifr_flags |= IFF_PROMISC;     ioctl(fd, SIOCSIFFLAGS, &ifr);   /* tryb nasłuchiwania */
```

## `getifaddrs(3)`
```c
struct ifaddrs *ifap, *ifa;
getifaddrs(&ifap);
for (ifa = ifap; ifa; ifa = ifa->ifa_next) {
    if (!ifa->ifa_addr) continue;
    int family = ifa->ifa_addr->sa_family;   /* AF_INET, AF_INET6, AF_PACKET */
    /* ifa->ifa_name, ifa->ifa_flags, ifa->ifa_addr, ifa->ifa_netmask */
}
freeifaddrs(ifap);
```
Zwraca listę **wszystkich** interfejsów z adresami (IPv4, IPv6, warstwy łącza) – wygodniejsze niż pętla `SIOCGIFCONF`.

## Odpowiedzi na pytania z zajęć
1. **Po co włączać interfejs bez adresu IP?** – analizatory ruchu (tryb nasłuchiwania/monitorowania, `tcpdump`, IDS), mostkowanie (bridge), agregacja (bonding), praca tylko na warstwie łącza (`PF_PACKET` – [[57 Gniazda sieciowe PF_PACKET - uzupełnienie]]), interfejsy pomocnicze (VLAN trunk), analiza bezpieczeństwa – host niewidoczny w warstwie 3.
2. **`ethtool` / `iwconfig` / `iw`** – parametry sprzętowe: prędkość, dupleks, offloading, statystyki, sterownik (`ethtool`); Wi-Fi: SSID, kanał, moc, tryb (`iw` – nowe, `iwconfig` – przestarzałe z pakietu wireless-tools).
3. **Tryb nasłuchiwania (_promiscuous_) vs monitorowania (_monitor_)**:
   - **promiscuous** – karta przekazuje **wszystkie ramki** widziane na medium, ale nadal działa w ramach **skojarzonej sieci** (Ethernet: switch i tak kieruje ramki portami; Wi-Fi: tylko po uwierzytelnieniu i deszyfrowaniu),
   - **monitor** (tylko Wi-Fi) – karta przechwytuje **surowe ramki 802.11** (także zarządzające i kontrolne) ze **wszystkich** sieci na kanale, **bez asocjacji**, z nagłówkami radiotap; podstawa audytu bezpieczeństwa sieci bezprzewodowych.
4. `netdevice(7)` – pełna lista żądań; `getifaddrs(3)`, `get_ifi_info` (UNP) – przenośne pobieranie informacji o interfejsach.

## Zobacz też
- [[Programowanie Sieciowe/ioctl(2)]], [[Programowanie Sieciowe/Warstwa Łącza Danych]]
- [[56 Obsługa tablicy routingu i pamięci ARP - uzupełnienie]]
