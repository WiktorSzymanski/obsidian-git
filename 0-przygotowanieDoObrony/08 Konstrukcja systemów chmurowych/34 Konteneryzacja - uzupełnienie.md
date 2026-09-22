---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 34
---
# 34. Konteneryzacja – uzupełnienie
---
> Uzupełnienie [[Konstrukcja Systemów Chmurowych/Konteneryzacja]], [[Docker]], [[Docker Images]]: szczegóły **izolacji procesów** (typy namespaces, capabilities, seccomp, rootless), **zarządzania zasobami** (cgroups v1/v2, OpenVZ), **systemów składowania danych** dla kontenerów oraz zalet kopii migawkowych.

## Izolacja procesów
### Przestrzenie nazw (namespaces) Linuksa
Każdy typ izoluje **widok** jednego rodzaju zasobu. Tworzone wywołaniami `clone(CLONE_NEW*)`, `unshare()`, dołączanie `setns()`; widoczne w `/proc/<pid>/ns/`.
| Namespace | Flaga | Izoluje | Od jądra |
|---|---|---|---|
| **mount** | `CLONE_NEWNS` | tablicę montowań (własny `/`, pivot_root) | 2.4.19 |
| **UTS** | `CLONE_NEWUTS` | nazwę hosta i domeny NIS | 2.6.19 |
| **IPC** | `CLONE_NEWIPC` | kolejki komunikatów, semafory, pamięć współdzieloną System V i POSIX | 2.6.19 |
| **PID** | `CLONE_NEWPID` | numerację procesów – pierwszy proces ma **PID 1** (odpowiada za sieroty i sygnały) | 2.6.24 |
| **network** | `CLONE_NEWNET` | interfejsy, adresy, tablice routingu, reguły netfilter, porty, gniazda | 2.6.29 |
| **user** | `CLONE_NEWUSER` | UID/GID i uprawnienia – **root w kontenerze ↔ zwykły użytkownik na hoście** (mapowanie `/proc/<pid>/uid_map`) | 3.8 |
| **cgroup** | `CLONE_NEWCGROUP` | widok hierarchii cgroup (kontener widzi swoją jako korzeń) | 4.6 |
| **time** | `CLONE_NEWTIME` | przesunięcia zegarów `CLOCK_MONOTONIC`/`BOOTTIME` | 5.6 |

```bash
unshare --pid --fork --mount-proc --uts --net --user --map-root-user bash
hostname kontener; ps aux     # tylko bash i ps, własna nazwa hosta
```
**Sieć kontenerów**: para interfejsów **veth** (jeden koniec w namespace kontenera, drugi podłączony do mostu `docker0`) + NAT (iptables/nftables) dla ruchu wychodzącego, przekierowanie portów (`-p 8080:80`); alternatywy: `host`, `macvlan`, overlay (VXLAN) między hostami, CNI w Kubernetes.

### Dodatkowe mechanizmy izolacji (hardening)
- **Capabilities** – kontener domyślnie z ograniczonym zbiorem (Docker usuwa m.in. `CAP_SYS_ADMIN`, `CAP_NET_ADMIN`, `CAP_SYS_MODULE`); `--cap-drop ALL --cap-add NET_BIND_SERVICE` ([[27 Mechanizmy egzekwowania polityki kontroli dostępu#DAC i uprzywilejowanie]]).
- **seccomp-bpf** – domyślny profil blokuje ~50 niebezpiecznych wywołań systemowych (`mount`, `reboot`, `kexec_load`, `ptrace` w starszych jądrach…).
- **LSM**: profile AppArmor (`docker-default`) / SELinux (etykiety `container_t` + kategorie MCS – kontenery nie widzą plików innych kontenerów).
- **Rootless containers** – demon i kontenery uruchamiane bez roota dzięki **user namespaces** (Podman domyślnie, Docker rootless) – ucieczka z kontenera daje tylko prawa zwykłego użytkownika ([[Zarządzanie Systemami Rozproszonymi/Podman]]).
- **no_new_privs**, system plików tylko do odczytu (`--read-only`), maskowanie `/proc`, `/sys`.
- **Silniejsza izolacja**: sandboxy jądra w przestrzeni użytkownika (**gVisor** – przechwytuje wywołania systemowe), **lekkie VM** na kontener (**Kata Containers**, **Firecracker** microVM w AWS Lambda) – bezpieczeństwo VM + ergonomia kontenerów.

## Zarządzanie zasobami
### Control groups (cgroups)
Hierarchiczne grupy procesów z **kontrolerami** (_subsystems_) zasobów, konfigurowane przez system plików `/sys/fs/cgroup`.

**Funkcje**: ograniczanie (limity), priorytetyzacja (wagi), rozliczanie (statystyki), sterowanie (zamrażanie, `freezer`).

| Kontroler | Parametry (v2) | Znaczenie |
|---|---|---|
| **cpu** | `cpu.max` („quota period”, np. `50000 100000` = 0,5 CPU), `cpu.weight` (1–10000, domyślnie 100) | twardy limit (CFS bandwidth) i udział przy rywalizacji |
| **cpuset** | `cpuset.cpus`, `cpuset.mems` | przypięcie do rdzeni i węzłów NUMA |
| **memory** | `memory.max` (twardy – po przekroczeniu reclaim, potem **OOM killer** w grupie), `memory.high` (dławienie), `memory.min/low` (gwarancje), `memory.swap.max` | pamięć RAM i cache stron |
| **io** | `io.max` (`rbps`, `wbps`, `riops`, `wiops` per urządzenie), `io.weight` | przepustowość dysku |
| **pids** | `pids.max` | liczba procesów (ochrona przed fork bomb) |
| **hugetlb**, **rdma**, **misc** | – | strony huge, zasoby RDMA |
| **devices** (v1) / eBPF (v2) | – | dostęp do urządzeń |

- **cgroups v1** – osobna hierarchia dla każdego kontrolera (niespójności, proces w różnych miejscach różnych drzew).
- **cgroups v2** (_unified hierarchy_, jądro 4.5+) – **jedna** hierarchia, procesy tylko w liściach, lepsze rozliczanie I/O zapisów buforowanych (powiązanie cache z cgroup), PSI (metryki presji zasobów).
- Docker: `docker run --cpus=1.5 --memory=512m --pids-limit=100 --device-write-bps /dev/sda:10mb`; Kubernetes: `requests` (planowanie, wagi) i `limits` (cgroups), klasy QoS Guaranteed/Burstable/BestEffort.
- Kontener **widzi** zasoby całego hosta w `/proc/meminfo`, `nproc` (bez LXCFS) – aplikacje (JVM) muszą czytać limity cgroup.

### Zarządzanie zasobami w OpenVZ
Uzupełnienie [[Konstrukcja Systemów Chmurowych/OpenVZ]] (monolityczna łata na jądro, przed cgroups):
- **User Beancounters (UBC)** – zestaw ~20 liczników i limitów per kontener (VE): pamięć jądra (`kmemsize`), strony prywatne (`privvmpages`), bufory gniazd (`tcpsndbuf`), liczba procesów (`numproc`), plików, gniazd; każdy z **barierą** (_barrier_) i **limitem**; licznik błędów `failcnt`,
- **Fair CPU scheduler** – dwupoziomowy: najpierw kontener (wg udziałów `cpuunits`, limit `cpulimit`), potem proces w kontenerze,
- **Dwupoziomowe kwoty dyskowe** – limit na kontener + zwykłe kwoty użytkowników wewnątrz,
- **Priorytety I/O** (CFQ),
- **VSwap** (nowsze wersje) – limit RAM + wirtualny swap,
- **checkpoint/restore i migracja na żywo** kontenerów (poprzednik CRIU).

## Systemy składowania danych dla kontenerów
### Obraz i warstwy
- Obraz = **uporządkowany stos warstw tylko do odczytu** (każda instrukcja `RUN/COPY/ADD` w Dockerfile tworzy warstwę – diff systemu plików jako archiwum tar) + manifest i konfiguracja (OCI Image Spec).
- Warstwy **adresowane treścią** (skrót SHA-256) → deduplikacja między obrazami, współdzielenie na hoście i w rejestrze, pobieranie tylko brakujących warstw.
- Kontener = warstwy obrazu + **cienka warstwa zapisywalna**; usunięcie kontenera usuwa warstwę zapisywalną.

### Sterowniki składowania (_storage drivers_ / snapshottery containerd)
| Sterownik | Mechanizm | Uwagi |
|---|---|---|
| **overlay2** | OverlayFS: `lowerdir` = warstwy, `upperdir` = warstwa kontenera; zmiana pliku → **copy-up** całego pliku | domyślny, wydajny; copy-up dużych plików kosztowny |
| **aufs** | union FS (poza jądrem mainline) | historyczny (Ubuntu) |
| **devicemapper** | **migawki thin provisioning** na urządzeniu blokowym – każda warstwa to migawka thin; zapis CoW na poziomie **bloków** | dawniej RHEL; tryb loop-lvm niezalecany |
| **btrfs** | każda warstwa to **subwolumin**, kontener = migawka subwoluminu (CoW na blokach) | wymaga Btrfs |
| **zfs** | warstwy jako **migawki i klony** ZFS | wymaga ZFS, dużo RAM |
| **vfs** | pełna kopia każdej warstwy | brak CoW, tylko testy |

Mechanizmy: [[21 Lokalne systemy plików - CoW, ZFS, migawki]], [[Zarządzanie Systemami Komputerowymi/OverlayFS]], [[Zarządzanie Systemami Komputerowymi/LVM]], [[Zarządzanie Systemami Komputerowymi/Btrfs]].

### Zalety stosowania kopii migawkowych i CoW dla kontenerów
- **szybkie tworzenie** kontenera (O(1) – migawka zamiast kopiowania gigabajtów),
- **oszczędność miejsca** – setki kontenerów z jednego obrazu współdzielą bloki; zapisywane tylko różnice,
- **współdzielenie pamięci podręcznej stron** (overlay2 – te same pliki bazowe w cache raz),
- **szybkie commit/rollback** stanu kontenera (`docker commit`, powrót do migawki),
- **efektywna migracja i replikacja** – przesyłanie tylko różnic (`btrfs send`, `zfs send`),
- łatwe tworzenie wielu środowisk (testy, CI).
Wady: warstwa zapisywalna nie nadaje się do danych intensywnie modyfikowanych (bazy) – fragmentacja, copy-up, utrata przy usunięciu → **woluminy**.

### Dane trwałe
- **Woluminy** (_volumes_) – katalog zarządzany przez Docker (`/var/lib/docker/volumes`), omija union FS (wydajność natywna), przetrwa usunięcie kontenera, może być współdzielony – [[Docker#Docker Volume]],
- **bind mounts** – montowanie katalogu hosta (konfiguracja, kod w trakcie rozwoju),
- **tmpfs** – dane ulotne w RAM (sekrety, cache),
- **sterowniki woluminów / wtyczki** – NFS, iSCSI, Ceph RBD, chmurowe dyski (EBS, Azure Disk),
- **Kubernetes**: `PersistentVolume` (zasób pamięci), `PersistentVolumeClaim` (żądanie), **StorageClass** (dynamiczne tworzenie przez provisioner), tryby dostępu RWO/ROX/RWX, **CSI** (_Container Storage Interface_ – standardowy interfejs sterowników: Ceph-CSI, Longhorn, OpenEBS), `StatefulSet` z szablonami PVC (stała tożsamość i wolumin dla każdej repliki bazy danych), migawki woluminów (`VolumeSnapshot`),
- **Rejestry obrazów** (Docker Hub, Harbor, registry) – przechowywanie warstw jako blobów (często na S3), OCI Distribution Spec.

## Porównanie izolacji
| | Proces (chroot) | Kontener (namespaces+cgroups) | Kontener z gVisor/Kata | Maszyna wirtualna |
|---|---|---|---|---|
| Jądro | wspólne | wspólne | sandbox / własne w microVM | własne |
| Izolacja | słaba | średnia (powierzchnia ataku: syscalle jądra) | silna | silna (nadzorca) |
| Start | ms | ms–s | ~100 ms–s | s–min |
| Narzut pamięci | brak | minimalny | średni | duży |
| Inny SO | ✘ | ✘ | ✘ / ✔ (Kata – jądro Linux) | ✔ |
