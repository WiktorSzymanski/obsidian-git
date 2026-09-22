---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 58
---
# 58. Gniazda sieciowe PF_NETLINK – uzupełnienie
---
> Uzupełnienie [[Programowanie Sieciowe/Gniazda Sieciowe PF_NETLINK]] o struktury `rtmsg`/`rtattr`, makra, pełny przykład i rodziny netlink.

## Czym jest netlink
**Gniazda `PF_NETLINK`** to mechanizm komunikacji **między jądrem a procesami przestrzeni użytkownika** (oraz między procesami), specyficzny dla Linuksa. Zastąpił obsługę przez `ioctl` i pliki `/proc` – jest **dwukierunkowy**, **asynchroniczny** (jądro może wysyłać powiadomienia o zdarzeniach), obsługuje **transmisję grupową** (multicast) i **atomowe** operacje na złożonych obiektach. Narzędzia z pakietu `iproute2` (`ip`, `ss`, `tc`) używają netlink.

```c
int sfd = socket(PF_NETLINK, SOCK_RAW | SOCK_DGRAM, NETLINK_ROUTE);  // rodzina w 3. argumencie
```

### Rodziny netlink (3. argument)
| Rodzina | Zastosowanie |
|---|---|
| **NETLINK_ROUTE** (rtnetlink) | interfejsy, adresy, **tablice routingu**, sąsiedzi (ARP/NDP), kolejkowanie (tc), reguły |
| NETLINK_KOBJECT_UEVENT | zdarzenia urządzeń (hotplug, udev) |
| NETLINK_NETFILTER | konfiguracja netfilter/nftables, logi, kolejki |
| NETLINK_GENERIC | generyczny netlink – rozszerzalny (nl80211/Wi-Fi, ethtool, taskstats) |
| NETLINK_AUDIT, NETLINK_SELINUX, NETLINK_XFRM (IPsec), NETLINK_SOCK_DIAG (ss) | inne podsystemy |

## Adres i nagłówki
```c
struct sockaddr_nl {
    sa_family_t     nl_family;   /* AF_NETLINK */
    unsigned short  nl_pad;      /* 0 */
    __u32           nl_pid;      /* PID procesu; 0 = jądro */
    __u32           nl_groups;   /* maska grup multicast (powiadomienia) */
};

struct nlmsghdr {                /* nagłówek każdego komunikatu netlink */
    __u32 nlmsg_len;             /* długość z nagłówkiem */
    __u16 nlmsg_type;            /* typ, np. RTM_GETROUTE */
    __u16 nlmsg_flags;           /* NLM_F_REQUEST, NLM_F_DUMP, NLM_F_ACK, NLM_F_CREATE... */
    __u32 nlmsg_seq;             /* numer sekwencyjny (dopasowanie odpowiedzi) */
    __u32 nlmsg_pid;             /* nadawca */
};
```
- Powiązanie z `bind` (podanie `nl_pid` i grup multicast) pozwala **odbierać powiadomienia** o zdarzeniach (np. dodanie adresu, zmiana stanu łącza).
- Jądro może odpowiadać **wieloczęściowo** (`NLM_F_MULTI`, zakończone komunikatem `NLMSG_DONE`) – stąd numery sekwencyjne.

## rtnetlink – komunikaty i struktury
| Typ | Obiekt | Struktura po `nlmsghdr` |
|---|---|---|
| `RTM_NEWLINK`/`GETLINK`/`DELLINK` | interfejsy | `struct ifinfomsg` |
| `RTM_NEWADDR`/`GETADDR`/`DELADDR` | adresy IP | `struct ifaddrmsg` |
| `RTM_NEWROUTE`/`GETROUTE`/`DELROUTE` | **trasy** | `struct rtmsg` |
| `RTM_NEWNEIGH`/`GETNEIGH`/`DELNEIGH` | sąsiedzi (ARP/NDP) | `struct ndmsg` |
```c
struct rtmsg {
    unsigned char rtm_family;    /* AF_INET / AF_INET6 */
    unsigned char rtm_dst_len;   /* długość maski docelowej (bity!) */
    unsigned char rtm_src_len;
    unsigned char rtm_tos;
    unsigned char rtm_table;     /* RT_TABLE_MAIN, RT_TABLE_LOCAL... (wiele tablic) */
    unsigned char rtm_protocol;  /* skąd trasa: RTPROT_STATIC, RTPROT_KERNEL, RTPROT_BOOT... */
    unsigned char rtm_scope;     /* RT_SCOPE_UNIVERSE, LINK, HOST */
    unsigned char rtm_type;      /* RTN_UNICAST, RTN_LOCAL, RTN_BROADCAST... */
    unsigned int  rtm_flags;
};
```
Po strukturze `rtmsg` następują **atrybuty** (TLV) `rtattr`:
```c
struct rtattr {
    unsigned short rta_len;      /* długość z nagłówkiem */
    unsigned short rta_type;     /* RTA_DST, RTA_GATEWAY, RTA_OIF, RTA_PRIORITY, RTA_SRC... */
    /* ... dane atrybutu ... */
};
```
Atrybuty mogą się powtarzać i zagnieżdżać. Makra obsługi:
- `NLMSG_DATA(nlh)`, `NLMSG_NEXT`, `NLMSG_OK`, `NLMSG_ALIGN`, `NLMSG_LENGTH`, `NLMSG_TAIL`,
- `RTM_RTA(rtmsg)`, `RTA_DATA(rta)`, `RTA_NEXT`, `RTA_OK`, `RTA_LENGTH`, `RTA_PAYLOAD`.

## Przykład – pobranie tablicy routingu (dump)
```c
int fd = socket(PF_NETLINK, SOCK_RAW, NETLINK_ROUTE);

struct { struct nlmsghdr nlh; struct rtmsg rtm; } req;
memset(&req, 0, sizeof(req));
req.nlh.nlmsg_len   = NLMSG_LENGTH(sizeof(struct rtmsg));
req.nlh.nlmsg_type  = RTM_GETROUTE;
req.nlh.nlmsg_flags = NLM_F_REQUEST | NLM_F_DUMP;   /* DUMP = cała tablica */
req.nlh.nlmsg_seq   = 1;
req.rtm.rtm_family  = AF_INET;
send(fd, &req, req.nlh.nlmsg_len, 0);

char buf[8192]; ssize_t len;
while ((len = recv(fd, buf, sizeof(buf), 0)) > 0) {
    struct nlmsghdr *nlh = (struct nlmsghdr*) buf;
    for (; NLMSG_OK(nlh, len); nlh = NLMSG_NEXT(nlh, len)) {
        if (nlh->nlmsg_type == NLMSG_DONE) goto done;
        struct rtmsg *rtm = (struct rtmsg*) NLMSG_DATA(nlh);
        struct rtattr *rta = RTM_RTA(rtm);
        int rtl = RTM_PAYLOAD(nlh);
        unsigned int dst = 0, gw = 0, oif = 0;
        for (; RTA_OK(rta, rtl); rta = RTA_NEXT(rta, rtl)) {
            switch (rta->rta_type) {
                case RTA_DST:     dst = *(unsigned int*) RTA_DATA(rta); break;
                case RTA_GATEWAY: gw  = *(unsigned int*) RTA_DATA(rta); break;
                case RTA_OIF:     oif = *(unsigned int*) RTA_DATA(rta); break;
            }
        }
        /* dst/rtm->rtm_dst_len, gw, oif -> wpis tablicy routingu */
    }
}
done: close(fd);
```
Dodanie trasy: `RTM_NEWROUTE` z flagami `NLM_F_REQUEST|NLM_F_CREATE|NLM_F_EXCL`, wypełnione `rtmsg` (rodzina, maska, tablica) i atrybuty `RTA_DST`, `RTA_GATEWAY`, `RTA_OIF`.

## Biblioteki
Surowy netlink jest złożony – w praktyce używa się **libnl** (libnl-route, libnl-genl) lub libmnl, które ukrywają budowę komunikatów i parsowanie atrybutów.

## Zalety i wady
✔ dwukierunkowy, powiadomienia o zdarzeniach (event-driven monitorowanie sieci), atomowe operacje, wiele tablic i pełna obsługa IPv6, rozszerzalność (generic netlink); ✘ skomplikowany protokół (TLV, wieloczęściowe odpowiedzi, wyrównania), specyficzny dla Linuksa.

## Zobacz też
- [[56 Obsługa tablicy routingu i pamięci ARP - uzupełnienie]], [[Programowanie Sieciowe/ip(8)]]
