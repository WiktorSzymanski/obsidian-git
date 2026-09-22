---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 36
---
# 36. Systemy składowania danych: thin provisioning, SAN, iSCSI, multipath I/O, OCFS, DRBD, GlusterFS, Ceph
---
> Warstwa składowania danych w chmurze i centrum danych musi zapewnić **pojemność, wydajność, dostępność i elastyczność przydziału**. Rozwiązania układają się w stos: **dostęp blokowy przez sieć** (SAN, iSCSI) → **nadmiarowość ścieżek** (multipath) i **replikacja** (DRBD) → **klastrowe systemy plików** na współdzielonym dysku (OCFS2) → **skalowalne rozproszone systemy składowania** (GlusterFS, Ceph).

Istniejące notatki: [[Zarządzanie Systemami Komputerowymi/Thin Provisioning]], [[Zarządzanie Systemami Komputerowymi/Storage Area Network]], [[Konstrukcja Systemów Chmurowych/GlusterFS]], [[Konstrukcja Systemów Chmurowych/Ceph]], [[CRUSH]], [[OSD]], [[Konstrukcja Systemów Chmurowych/RAID]].

## Rodzaje dostępu do danych
| | Blokowy | Plikowy | Obiektowy |
|---|---|---|---|
| Jednostka | blok (LBA) – klient widzi dysk | pliki i katalogi | obiekt (klucz + dane + metadane) w płaskiej przestrzeni |
| System plików | po stronie klienta | po stronie serwera | brak (API REST) |
| Protokoły | FC, iSCSI, NVMe-oF, RBD | NFS, SMB | S3, Swift |
| Zastosowania | bazy danych, dyski VM | współdzielone katalogi | archiwa, media, kopie, big data |
| Architektura | **SAN** | **NAS** | magazyn obiektowy |

## Thin provisioning (cienkie przydzielanie)
> Przydzielanie **logicznej pojemności** większej niż faktycznie zarezerwowana. Fizyczne bloki pobierane są z **puli** dopiero przy **pierwszym zapisie** (_allocate-on-write_).

- **Thick provisioning** (grube): cała pojemność rezerwowana od razu (_eager zeroed_ – wyzerowana, _lazy zeroed_ – zerowana przy zapisie).
- **Nadmierna subskrypcja** (_overcommit_): suma pojemności woluminów > pojemność puli (jak bank – nie wszyscy wykorzystają wszystko jednocześnie).
- ✔ lepsza utylizacja, niższe koszty początkowe, szybkie tworzenie woluminów, podstawa tanich migawek i klonów;
- ✘ ryzyko **wyczerpania puli** → błędy zapisu we wszystkich woluminach naraz: konieczne **monitorowanie progów** i alarmy, rozbudowa w porę; fragmentacja; narzut metadanych.
- **Odzyskiwanie miejsca**: po usunięciu pliku SP musi poinformować warstwę blokową – **TRIM/UNMAP/discard** (`fstrim`, `mount -o discard`), inaczej bloki pozostają zajęte.
- Implementacje: LVM thin pools (`lvcreate --type thin-pool`), dm-thin, obrazy qcow2/VMDK thin, macierze (3PAR, NetApp), Ceph RBD (domyślnie cienkie), ZFS zvol sparse.

## SAN (Storage Area Network)
> **Dedykowana, wydajna sieć** łącząca serwery z pamięcią masową (macierzami, bibliotekami taśmowymi), udostępniająca **dostęp blokowy**. Serwer widzi zasoby macierzy jak lokalne dyski (**LUN** – _Logical Unit Number_).

- **Fibre Channel (FC)** – klasyczna technologia SAN:
  - bezstratna sieć (sterowanie przepływem kredytami), prędkości 8/16/32/64 Gb/s,
  - adresacja **WWN** (_World Wide Name_) – WWPN portów, WWNN węzłów,
  - przełączniki FC (_fabric_), **zoning** (izolacja – które porty widzą które), **LUN masking** (macierz udostępnia LUN tylko wybranym hostom),
  - karty **HBA** (_Host Bus Adapter_),
  - topologie: point-to-point, pętla arbitrowana (FC-AL), switched fabric.
- **FCoE** – ramki FC w bezstratnym Ethernecie (DCB).
- **iSCSI** – SCSI przez TCP/IP (niżej).
- **NVMe over Fabrics** (RDMA, TCP, FC) – nowoczesny, niskie opóźnienia dla SSD.
- Cechy: odseparowanie ruchu danych od LAN, duża przepustowość, niskie opóźnienie, konsolidacja składowania, migawki i replikacja na macierzach, bootowanie z SAN, klastry z **współdzielonym dyskiem**.
- Wady: koszt (FC), złożoność, SAN sam w sobie nie zapewnia współdzielonego dostępu do plików – wiele hostów montujących ten sam LUN wymaga **klastrowego systemu plików** (OCFS2, GFS2, VMFS).

## iSCSI (RFC 7143, dawniej 3720)
> Protokół przenoszący **polecenia SCSI** (CDB) przez **TCP/IP** (port **3260**). Pozwala zbudować SAN na zwykłym Ethernecie.

### Elementy
- **Inicjator** (_initiator_) – klient: programowy (`open-iscsi` w Linux, Microsoft iSCSI Initiator) lub sprzętowy (**iSCSI HBA** / karta z odciążaniem TCP – TOE).
- **Cel** (_target_) – serwer udostępniający LUN-y: macierz, serwer Linux z **LIO** (`targetcli`), `tgt`, TrueNAS, Ceph iSCSI gateway.
- **LUN** – jednostka logiczna (dysk) w obrębie celu; zaplecze: plik, wolumin LVM, dysk, RBD.
- **Nazewnictwo** (unikalne globalnie):
  - **IQN**: `iqn.<rrrr-mm>.<odwrócona domena>:<nazwa>`, np. `iqn.2003-01.org.linux-iscsi.serwer1:storage.lun0`,
  - **EUI**: `eui.02004567A425678D`, **NAA**.
- **Portal** – adres IP + port celu.

### Działanie
1. **Odkrywanie** (_discovery_): statyczne (adres podany ręcznie), **SendTargets** (inicjator pyta portal o listę celów), **iSNS** (_Internet Storage Name Service_ – rejestr, jak DNS dla SAN), SLP.
2. **Logowanie** (_login phase_): zestawienie **sesji** (jedno lub wiele połączeń TCP), negocjacja parametrów (rozmiary PDU, `HeaderDigest`/`DataDigest` – CRC32C, `InitialR2T`, `ImmediateData`), uwierzytelnienie (**CHAP**).
3. **Faza pełnej pracy** (_full feature phase_): **PDU iSCSI** zawierające polecenia SCSI (`READ(10)`, `WRITE(10)`), dane (Data-In/Data-Out), **R2T** (_Ready To Transfer_ – sterowanie przesyłem danych zapisu), odpowiedzi ze statusem, numeracja (CmdSN, StatSN).
4. Wylogowanie; odtwarzanie połączeń po awarii sieci (_session recovery_).

```bash
# cel (Linux LIO)
targetcli /backstores/block create lun0 /dev/vg0/lv_data
targetcli /iscsi create iqn.2024-01.pl.firma:storage
targetcli /iscsi/iqn.2024-01.pl.firma:storage/tpg1/luns create /backstores/block/lun0
targetcli /iscsi/iqn.2024-01.pl.firma:storage/tpg1/acls create iqn.2024-01.pl.firma:web1

# inicjator (open-iscsi)
iscsiadm -m discovery -t sendtargets -p 10.0.0.10
iscsiadm -m node -T iqn.2024-01.pl.firma:storage -p 10.0.0.10 --login
lsblk     # nowy dysk /dev/sdX
```

### Bezpieczeństwo i dobre praktyki
- ACL na celu (IQN inicjatorów), uwierzytelnianie CHAP (słabe), **IPsec** dla poufności, **dedykowana sieć/VLAN** dla ruchu składowania,
- **jumbo frames** (MTU 9000), sterowanie przepływem, sieć 10/25/100 GbE, **multipath** (nie bonding).

### iSCSI vs FC
| | iSCSI | Fibre Channel |
|---|---|---|
| Sieć | Ethernet/IP (routowalny, także WAN) | dedykowana FC |
| Koszt | niski (standardowy sprzęt) | wysoki (HBA, przełączniki) |
| Wydajność | zależy od sieci, narzut TCP | bezstratny, niskie opóźnienia |
| Kompetencje | administratorzy sieci IP | specjalistyczne |
| Adresacja | IQN, IP | WWN, zoning |

## Multipath I/O
> Wykorzystanie **wielu fizycznych ścieżek** między serwerem a pamięcią masową (różne karty HBA/NIC, przełączniki, kontrolery macierzy) do tego samego LUN. Daje **odporność na awarię** elementu ścieżki i **równoważenie obciążenia**.

```
          ┌── HBA1 ── przełącznik A ── kontroler A ──┐
serwer ───┤                                           ├── LUN 5 (macierz)
          └── HBA2 ── przełącznik B ── kontroler B ──┘
```
- **Problem**: bez oprogramowania multipath system widzi ten sam LUN jako **kilka urządzeń** (`/dev/sdb`, `/dev/sdc`) → ryzyko uszkodzenia danych przy równoległym użyciu.
- **Linux dm-multipath**:
  - demon **`multipathd`** identyfikuje ścieżki do tego samego LUN po **WWID** (identyfikator SCSI VPD 0x83),
  - tworzy jedno urządzenie **`/dev/mapper/mpathX`** (device-mapper target `multipath`),
  - **grupy ścieżek** (_path groups_) z priorytetami,
  - **path checkery** (`tur` – TEST UNIT READY, `directio`) wykrywają awarie i powrót ścieżek,
  - **polityki wyboru** (_path selector_):
    - `round-robin` – rotacja I/O,
    - `queue-length` – najkrótsza kolejka,
    - `service-time` – najmniejszy szacowany czas obsługi,
  - **polityki grupowania**: `failover` (jedna aktywna ścieżka, reszta zapasowa – active/passive), `multibus` (wszystkie aktywne – active/active), `group_by_prio` (wg priorytetów, np. ALUA),
  - `no_path_retry` / `queue_if_no_path` – kolejkowanie I/O przy chwilowej utracie wszystkich ścieżek,
  - `failback` – powrót do preferowanej ścieżki po naprawie.
- **ALUA** (_Asymmetric Logical Unit Access_) – macierze active/passive sygnalizują ścieżki optymalne (do kontrolera-właściciela LUN) i nieoptymalne.
- Konfiguracja: `/etc/multipath.conf`; `multipath -ll` – podgląd topologii.
- Inne implementacje: Windows MPIO + DSM, VMware NMP/PSP, PowerPath (EMC).
- Dla iSCSI: osobne sesje przez różne interfejsy/podsieci (nie LACP bonding – multipath działa na poziomie SCSI i lepiej rozkłada ruch).

```text
mpatha (36001405a1b2c3d4e5f60000000000001) dm-2 LIO-ORG,lun0
size=100G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 3:0:0:0 sdb 8:16 active ready running
`-+- policy='service-time 0' prio=10 status=enabled
  `- 4:0:0:0 sdc 8:32 active ready running
```

## OCFS2 (Oracle Cluster File System 2)
> **Klastrowy system plików ze współdzielonym dyskiem**: wiele węzłów jednocześnie montuje ten sam system plików na wspólnym urządzeniu blokowym (SAN/iSCSI/DRBD dual-primary) i ma do niego równoległy dostęp do odczytu i zapisu z **semantyką POSIX**.

- **Stos klastrowy o2cb** (lub Pacemaker/Corosync):
  - konfiguracja węzłów (`/etc/ocfs2/cluster.conf`),
  - **heartbeat dyskowy** – każdy węzeł co 2 s zapisuje znacznik w obszarze heartbeat na współdzielonym dysku; brak zapisu przez próg → węzeł uznany za martwy,
  - **heartbeat sieciowy** (TCP, port 7777),
  - **o2dlm** – **rozproszony menedżer blokad** (poziomy blokad NL/PR/EX na i-węzłach, metadanych, danych); koordynuje buforowanie – węzeł może buforować dane pliku, gdy trzyma odpowiednią blokadę.
- **Dziennik na każdy węzeł** (JBD2) – po awarii węzła inny węzeł **odtwarza jego dziennik** i zwalnia blokady.
- **Sloty węzłów** – maksymalna liczba węzłów montujących (ustalana przy `mkfs.ocfs2 -N`).
- **Fencing**: węzeł, który utracił łączność z większością (kworum), **sam się restartuje** (_self-fencing_), by nie uszkodzić danych przy podziale klastra (_split brain_).
- **Funkcje**: ekstenty, pliki rzadkie, **reflinki** (kopie CoW plików – szybkie klony obrazów VM), rozszerzone atrybuty, ACL POSIX, kwoty, indeksowane katalogi, rozmiar do 4 PiB.
- **Zastosowania**: Oracle RAC (pliki bazy), repozytoria obrazów VM (Oracle VM, KVM z live migration), klastry aplikacyjne HA wymagające wspólnych plików.
- Porównanie z rozproszonymi SP (GlusterFS/Ceph): OCFS2 skaluje się do **dziesiątek** węzłów, wymaga współdzielonego urządzenia (SAN) – brak własnej replikacji danych; szczegóły i porównanie z GFS2 – [[20 Rozproszone systemy plików#OCFS / OCFS2 (Oracle Cluster File System)]].

## DRBD (Distributed Replicated Block Device, LINBIT)
> Moduł jądra Linux replikujący **urządzenie blokowe** w czasie rzeczywistym przez sieć TCP między węzłami – **„RAID-1 przez sieć”** (_shared-nothing_). Aplikacje i system plików widzą urządzenie `/dev/drbdX`.

### Architektura
- Każdy węzeł ma **lokalny dysk zaplecza** (_backing device_, np. LV).
- **Role**:
  - **Primary** – może być używany (montowany, zapisywany),
  - **Secondary** – tylko odbiera replikację (niedostępny dla aplikacji).
- **Single-primary** (typowo) – z systemem plików lokalnym (ext4/XFS), failover przez menedżer klastra.
- **Dual-primary** – oba węzły zapisują jednocześnie → **wymaga klastrowego SP** (OCFS2, GFS2) lub aplikacji koordynującej (live migration VM).
- **Metadane** (wewnętrzne – na końcu dysku, lub zewnętrzne): identyfikatory generacji (GI) do określenia, która kopia jest aktualna; **dziennik aktywności** (_activity log_) – obszary w trakcie zapisu; **bitmapa** niesynchronizowanych bloków → po przerwaniu połączenia synchronizowane są **tylko zmienione bloki** (szybka resynchronizacja).

### Protokoły replikacji
| Protokół | Zapis potwierdzony aplikacji, gdy… | Charakter | Utrata danych przy awarii primary |
|---|---|---|---|
| **A** – asynchroniczny | zapisany na lokalnym dysku **i** umieszczony w buforze wysyłki TCP | najszybszy, dla WAN/DR | możliwa (dane w buforze) |
| **B** – półsynchroniczny (_memory synchronous_) | lokalny zapis zakończony **i** dane dotarły do **pamięci** węzła secondary | kompromis | tylko przy jednoczesnej awarii obu węzłów |
| **C** – synchroniczny | zapis potwierdzony na **dyskach obu** węzłów | najbezpieczniejszy, najczęściej używany (LAN) | brak |

### Split brain
Oba węzły stały się primary podczas przerwy w komunikacji → rozbieżne dane. DRBD wykrywa po ponownym połączeniu (identyfikatory generacji) i rozwiązuje wg polityk (`after-sb-0pri`: discard-zero-changes, discard-younger-primary…) lub ręcznie. Zapobieganie: **fencing/STONITH**, kworum (DRBD 9 z 3 węzłami lub _diskless tiebreaker_), `resource-and-stonith`.

### Zastosowania
- **wysokodostępne usługi** bez SAN: NFS, bazy danych (MySQL/PostgreSQL), iSCSI target – z **Pacemaker + Corosync** (DRBD jako zasób promowany do primary, potem montowanie SP, adres IP, usługa),
- replikacja do zapasowego ośrodka (protokół A, DRBD Proxy – buforowanie i kompresja przez WAN),
- **DRBD 9**: replikacja do wielu węzłów (do 32), węzły bezdyskowe, **LINSTOR** – orkestracja woluminów DRBD (sterownik CSI dla Kubernetes, integracja z Proxmox/OpenStack).

### DRBD vs inne
| | DRBD | Macierz SAN z replikacją | Ceph RBD |
|---|---|---|---|
| Sprzęt | zwykłe serwery + sieć | dedykowana macierz | wiele zwykłych serwerów |
| Skala | 2–kilka węzłów (per zasób) | macierz | setki–tysiące OSD |
| Replikacja | blokowa, synchroniczna/asynchroniczna | przez macierz | obiektowa, rozproszona (CRUSH) |
| Złożoność | niska | średnia (koszt) | wysoka |

## Skalowalne klastrowe systemy plików
### GlusterFS
Szczegóły: [[Konstrukcja Systemów Chmurowych/GlusterFS]]. Najważniejsze:
- **brak serwera metadanych** – położenie pliku wyliczane przez **Elastic Hashing** (DHT: skrót nazwy pliku → zakres skrótów przypisany do „cegiełki” – _brick_ – w atrybutach rozszerzonych katalogu),
- działanie w przestrzeni użytkownika na stosie **translatorów** (moduły: dystrybucja, replikacja, dyspersja, cache, klient/serwer),
- woluminy: distributed, replicated (z kworum i arbitrem), **dispersed** (erasure coding), distributed-replicated; **self-heal** (naprawa po powrocie węzła), geo-replikacja asynchroniczna,
- dostęp: natywny klient FUSE, `libgfapi` (QEMU), NFS-Ganesha, SMB,
- ✔ prosty, skalowanie przez dodawanie cegiełek z rebalansowaniem; ✘ słabsza wydajność przy wielu małych plikach i operacjach na metadanych (listowanie katalogów rozproszonych).

### Ceph
Szczegóły: [[Konstrukcja Systemów Chmurowych/Ceph]], [[CRUSH]], [[OSD]], [[Konstrukcja Systemów Chmurowych/RADOS]], [[Konstrukcja Systemów Chmurowych/Erasure Coding]]. Uzupełnienia:
- **RADOS** – rozproszony magazyn obiektów; wszystko (bloki, pliki) sprowadzone do obiektów w **pulach**,
- **mapy klastra** utrzymywane przez **monitory MON** (kworum z algorytmem **Paxos**, nieparzysta liczba 3/5): mapa monitorów, **mapa OSD** (stan up/down, in/out), mapa PG, **mapa CRUSH** (hierarchia awarii: host → szafa → rząd → ośrodek, reguły rozmieszczania), mapa MDS; klient pobiera mapy i sam wylicza położenie → **brak centralnej tablicy lokalizacji**,
- **ścieżka obiektu**: `pg = hash(nazwa obiektu) mod liczba_PG` → CRUSH(pg, mapa, reguła) → lista OSD (primary pierwszy); klient zapisuje do primary, a ten replikuje do pozostałych i potwierdza po zapisie wszystkich (silna spójność),
- **liczba PG** na pulę: zasada kciuka $\approx \dfrac{\text{liczba OSD} \times 100}{\text{liczba replik}}$ zaokrzone do potęgi 2; obecnie **autoscaler PG**,
- **BlueStore** (domyślny od Luminous) – OSD zapisuje **bezpośrednio na urządzenie blokowe** (bez systemu plików), metadane w RocksDB (na SSD/NVMe), sumy kontrolne, kompresja; wcześniej **FileStore** (XFS/Btrfs + dziennik),
- **MGR** (manager – metryki, dashboard, moduły), **RGW** (S3/Swift), **RBD** (obrazy blokowe – cienkie, migawki, klony CoW, mirroring między klastrami; używane przez OpenStack Cinder, Kubernetes Ceph-CSI, Proxmox), **CephFS** (POSIX z serwerami metadanych **MDS** – aktywne/rezerwowe, dynamiczne dzielenie drzewa katalogów między MDS),
- **odtwarzanie**: OSD `down` → po `mon_osd_down_out_interval` (domyślnie 600 s) oznaczony `out` → rebalans i odtworzenie replik na innych OSD; **scrub** / **deep scrub** (porównanie sum),
- **pule replikowane** (`size=3`, `min_size=2`) vs **erasure coded** (np. k=4, m=2 – mniejszy narzut miejsca, większy CPU, wyższe opóźnienie).

### GlusterFS vs Ceph
| | GlusterFS | Ceph |
|---|---|---|
| Interfejsy | plikowy (+ obiekty/bloki przez bramy) | **zunifikowany**: blok (RBD), plik (CephFS), obiekt (RGW) |
| Metadane | brak serwera metadanych (hashing) | MON (mapy) + CRUSH; MDS tylko dla CephFS |
| Rozmieszczenie danych | DHT na nazwach plików, cegiełki | CRUSH na obiektach, świadomość topologii awarii |
| Spójność | replikacja synchroniczna z kworum | silna (primary-copy) |
| Złożoność | niska | wysoka |
| Skala | setki węzłów | tysiące węzłów, eksabajty |
| Status | Red Hat zakończył wsparcie komercyjne (2024) | aktywnie rozwijany, standard w OpenStack |
