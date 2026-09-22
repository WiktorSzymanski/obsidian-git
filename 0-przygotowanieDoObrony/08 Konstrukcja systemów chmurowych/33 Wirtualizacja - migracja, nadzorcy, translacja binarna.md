---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 33
---
# 33. Wirtualizacja systemów operacyjnych – uzupełnienie
---
> Uzupełnienie istniejących notatek ([[Zarządzanie Systemami Rozproszonymi/Wirtualizacja]], [[Konstrukcja Systemów Chmurowych/Wirtualizacja w procesorach z rodziny x86]], [[Konstrukcja Systemów Chmurowych/Wirtualizacja Pamięci]], [[Konstrukcja Systemów Chmurowych/Obsługa Urządzań]]) o: **typy nadzorców**, **translację binarną**, **migrację maszyn wirtualnych** (brak w notatkach) oraz **wirtualizację GPU** i **agregację**.

## Przegląd metod obsługi instrukcji wrażliwych
| Metoda | Jak obsługiwane są instrukcje wrażliwe | Modyfikacja gościa | Wydajność | Przykłady |
|---|---|---|---|---|
| **trap-and-emulate** (klasyczna) | instrukcje uprzywilejowane w trybie użytkownika powodują pułapkę → VMM emuluje | nie | dobra, ale tylko gdy wszystkie wrażliwe są uprzywilejowane (Popek-Goldberg) | IBM System/370 |
| **pełna wirtualizacja z translacją binarną** | kod jądra gościa skanowany i tłumaczony w locie, instrukcje wrażliwe zastępowane wywołaniami VMM | nie | dobra (narzut przy przełączeniach) | VMware Workstation/ESX (do ~2010), VirtualBox (bez VT-x), QEMU TCG |
| **parawirtualizacja** | gość świadomie wywołuje nadzorcę (**hiperwywołania**, _hypercalls_) | **tak** (jądro) | bardzo dobra | Xen PV, sterowniki virtio |
| **wsparcie sprzętowe** | procesor w trybie gościa przechwytuje instrukcje wrażliwe (VM exit) | nie | bardzo dobra (II generacja – EPT/NPT) | Intel VT-x, AMD-V; KVM, Hyper-V, ESXi |
| **emulacja** | interpretacja/translacja **całego** zbioru instrukcji innej architektury | nie | niska | QEMU (ARM na x86), Bochs – [[Konstrukcja Systemów Chmurowych/Emulacja]] |

## Translacja binarna (VMware)
- Gość działa w **ring 1** (jądro) i ring 3 (aplikacje); VMM w ring 0.
- **Kod użytkownika** gościa (ring 3) wykonuje się **bezpośrednio** na procesorze – nie zawiera instrukcji wrażliwych w trybie jądra.
- **Kod jądra** gościa jest przed wykonaniem **tłumaczony blokami** (do najbliższego skoku – _translation unit_):
  - instrukcje nieszkodliwe kopiowane bez zmian (_ident translation_),
  - instrukcje wrażliwe (np. `POPF`, `SGDT`, `SIDT`, `SMSW`, dostęp do rejestrów sterujących, `CLI/STI`) zastępowane sekwencjami emulującymi ich efekt na **wirtualnym** stanie procesora lub wywołaniem VMM,
  - skoki poprawiane na adresy w **pamięci podręcznej translacji** (_translation cache_), bloki łączone (_chaining_) → kolejne wykonanie bez ponownego tłumaczenia.
- Śledzenie zapisów do tablic stron gościa (ochrona stron) i utrzymywanie **shadow page tables**.
- ✔ działa na x86 bez wsparcia sprzętowego i bez modyfikacji gościa; ✘ złożoność VMM, narzut na operacje jądra (wywołania systemowe, przełączanie kontekstu, obsługa stron).

## Typy nadzorców (VMM / hypervisor)
| | **Typ 1** – natywny (_bare-metal_) | **Typ 2** – gospodarza (_hosted_) |
|---|---|---|
| Umiejscowienie | bezpośrednio na sprzęcie | jako aplikacja/moduł w systemie gospodarza |
| Sterowniki urządzeń | własne lub w uprzywilejowanej domenie (Xen dom0, Hyper-V partycja nadrzędna) | systemu gospodarza |
| Wydajność, izolacja | wyższa, mniejsza TCB | niższa, zależność od gospodarza |
| Zastosowanie | serwery, chmura | stacje robocze, testy, programiści |
| Przykłady | VMware ESXi, Xen, Microsoft Hyper-V, KVM* | VMware Workstation, VirtualBox, Parallels, QEMU |

\* **KVM** – moduł jądra Linux zamieniający je w nadzorcę typu 1 (Linux sam jest „gospodarzem” i nadzorcą); QEMU w przestrzeni użytkownika emuluje urządzenia. Podział typów jest umowny.

Architektura **mikrojądrowa** (Xen, Hyper-V: mały nadzorca + uprzywilejowana domena ze sterownikami) vs **monolityczna** (ESXi: sterowniki w nadzorcy) – [[Konstrukcja Systemów Chmurowych/Xen]].

## Wirtualizacja pamięci i urządzeń – skrót
- Pamięć: trzy poziomy adresów (wirtualne gościa → „fizyczne” gościa → fizyczne maszyny), **shadow page tables** (VMM utrzymuje odwzorowanie złożone, śledzi zmiany tablic gościa) vs **EPT/NPT** (sprzętowe dwupoziomowe przekształcanie w MMU, TLB z tagami VPID/ASID); nadmierne przydzielanie (**balloon**, **deduplikacja stron** KSM/TPS, kompresja, swap nadzorcy) – [[Konstrukcja Systemów Chmurowych/Wirtualizacja Pamięci]], [[VM-baloon.excalidraw]].
- Urządzenia: **emulacja** (np. karta e1000, pułapki na porty I/O/MMIO), **parawirtualizacja** (virtio: kolejki pierścieniowe `virtqueue` we wspólnej pamięci, `vhost` w jądrze), **przekazywanie** (_passthrough_, VT-d/IOMMU – DMA remapping, przerwania remapping), **SR-IOV** (funkcje wirtualne VF karty) – [[Konstrukcja Systemów Chmurowych/Device Passthrough]].

## Migracja maszyn wirtualnych
> Przeniesienie działającej (lub zatrzymanej) maszyny wirtualnej między fizycznymi serwerami. Stan VM obejmuje: **pamięć**, **stan procesorów** (rejestry), **stan urządzeń wirtualnych**, **połączenia sieciowe** i **dyski**.

### Cele
- konserwacja sprzętu bez przestoju usług,
- **równoważenie obciążenia** (VMware DRS), konsolidacja w celu oszczędności energii (wyłączanie hostów),
- ucieczka przed awarią (prognozowana awaria sprzętu),
- przeniesienie bliżej danych/użytkowników.

### Migracja „na zimno” (_offline, cold_)
VM zatrzymana (lub wstrzymana – _suspend_ z zapisem stanu), kopiowanie obrazu/stanu, uruchomienie na docelowym hoście. Prosta, ale z przestojem.

### Migracja „na żywo” (_live migration_)
**Pre-copy** (Clark i in., Xen 2005; VMware vMotion, KVM):
1. **Rezerwacja** zasobów na hoście docelowym.
2. **Iteracyjne kopiowanie pamięci** przy działającej VM:
   - runda 1: kopiowanie **wszystkich** stron,
   - runda $k$: kopiowanie stron **zmodyfikowanych** (_dirty_) w trakcie rundy $k-1$ – śledzenie przez bitmapę brudnych stron (zabezpieczenie stron przed zapisem / bit dirty w EPT, PML).
3. **Stop-and-copy**: gdy zbiór brudnych stron jest mały (lub osiągnięto limit rund), VM zostaje **wstrzymana**; kopiowane są pozostałe strony + stan CPU i urządzeń.
4. **Zatwierdzenie** – host docelowy potwierdza kompletny obraz (źródło nadal ma kopię – bezpieczne wycofanie).
5. **Aktywacja** na hoście docelowym; **gratuitous ARP / RARP** ogłasza nowe położenie adresu MAC w przełącznikach.

- **Czas przestoju** (_downtime_) – typowo dziesiątki–setki ms (połączenia TCP przetrwają).
- **Całkowity czas migracji** – zależy od rozmiaru pamięci, szybkości zmian i przepustowości sieci.
- **Problem**: VM zapisująca pamięć szybciej, niż sieć ją przesyła, **nie zbiega się** → spowalnianie VM (_auto-converge_, dławienie CPU), kompresja (XBZRLE), przejście na post-copy.

**Post-copy**:
1. VM wstrzymywana od razu, przesyłany jest tylko stan CPU/urządzeń (i minimalny zbiór stron),
2. VM **wznawiana na hoście docelowym**,
3. brakujące strony pobierane **na żądanie** przy chybieniach (_demand paging_ przez sieć, userfaultfd) oraz aktywnie w tle (_pre-paging_).
- ✔ każda strona przesyłana **dokładnie raz**, krótki i przewidywalny czas migracji; ✘ spadek wydajności po wznowieniu, **awaria źródła lub sieci w trakcie = utrata VM** (stan rozdzielony między hosty).

**Hybrydowa**: jedna runda pre-copy, potem post-copy.

### Wymagania i zagadnienia
- **Dyski**: **współdzielona pamięć masowa** (SAN, NFS, Ceph RBD – [[36 Systemy składowania danych - iSCSI, multipath, OCFS, DRBD]]) – migruje tylko pamięć; bez niej **migracja pamięci masowej** (VMware Storage vMotion, KVM block migration/drive-mirror: kopiowanie bloków + śledzenie zmienionych bloków w trakcie, analogicznie do pre-copy).
- **Sieć**: ta sama domena warstwy 2 (VLAN) lub sieć nakładkowa (VXLAN) – VM zachowuje adresy IP/MAC.
- **Zgodność procesorów**: gość mógł wykryć instrukcje (AVX-512…) niedostępne na docelowym CPU → **maskowanie CPUID** do wspólnego zbioru (VMware EVC, model CPU w libvirt).
- **Urządzenia przekazane** (passthrough, SR-IOV) utrudniają migrację – stan w sprzęcie; rozwiązania: odłączenie przed migracją, bonding z virtio.
- **Bezpieczeństwo**: szyfrowanie kanału migracji (zawartość pamięci może zawierać klucze).
- **Punkty kontrolne VM** (_snapshot/checkpoint_) – zapis stanu pamięci i dysku (qcow2, delta disk) do przywrócenia; podstawa **tolerowania awarii przez replikację stanu**: VMware FT (_lockstep_), Remus (Xen – ciągłe asynchroniczne checkpointy co ~25 ms, bufor wyjścia sieci do zatwierdzenia – _output commit_, por. [[22 Wsteczne odtwarzanie stanu - uzupełnienie]]), KVM COLO.
- Migracja **kontenerów**: CRIU – [[Konstrukcja Systemów Chmurowych/Konteneryzacja#Migracja kontenerów]].

## Agregacja
**Odwrotność partycjonowania**: wiele fizycznych maszyn tworzy jedną **większą maszynę wirtualną** (system SMP z sumą procesorów i pamięci).
- **vSMP** (ScaleMP) – nadzorca na każdym węźle + szybka sieć InfiniBand, spójny obraz jednego komputera dla nieprzerobionego SO; zarządzanie spójnością pamięci między węzłami (migracja/replikacja stron) – [[Konstrukcja Systemów Chmurowych/Versitale SMP]].
- Problemy: opóźnienia dostępu do „zdalnej” pamięci (NUMA skrajnie nierównomierne), koszt; konkurencja ze strony klastrów z MPI.
- Pokrewne: **klastry pojedynczego obrazu systemu** (SSI – OpenMosix, Kerrighed), **rozproszona pamięć współdzielona** ([[08 Podejścia do budowy systemów rozproszonych#5. Rozproszona pamięć współdzielona (DSM)]]).
- Paradygmaty: partycjonowanie (serwer → wiele VM), agregacja (wiele serwerów → jedna VM), remoting (zdalny pulpit) – [[Konstrukcja Systemów Chmurowych/Open Virtualization Format#Paradygmaty wirtualizacji]].

## Wirtualizacja GPU
| Metoda | Opis | Współdzielenie | Wydajność |
|---|---|---|---|
| **emulacja / API forwarding** | wirtualna karta, polecenia OpenGL/DirectX przekazywane do gospodarza (VirGL, VMware SVGA 3D) | wiele VM | niska–średnia |
| **passthrough** | cała fizyczna karta przydzielona jednej VM (VFIO + IOMMU), sterownik producenta w gościu | ✘ (1 VM) | natywna |
| **mediated devices (mdev)** – vGPU | sterownik gospodarza dzieli GPU na wirtualne instancje z przydziałem pamięci i czasu; ścieżka danych bezpośrednia, sterowanie przez gospodarza (NVIDIA vGPU/GRID, Intel GVT-g) | ✔ | bliska natywnej |
| **SR-IOV** | GPU udostępnia funkcje wirtualne VF (AMD MxGPU, Intel SR-IOV na nowych kartach) | ✔ | bliska natywnej |
| **MIG** (Multi-Instance GPU, NVIDIA A100+) | sprzętowy podział GPU na izolowane instancje (rdzenie, pamięć) | ✔ | natywna, izolacja sprzętowa |
| **time-slicing** (np. w Kubernetes) | współdzielenie w czasie bez izolacji pamięci | ✔ | zależna od obciążenia |

Problemy: migracja VM z GPU, licencjonowanie vGPU, izolacja pamięci GPU, planowanie zadań.
