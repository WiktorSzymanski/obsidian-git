---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 21
---
# 21. Lokalne systemy plików – uzupełnienie: CoW, ZFS, rodzaje migawek
---
> Uzupełnienie istniejących notatek: [[Zarządzanie Systemami Komputerowymi/UnionFS]], [[Zarządzanie Systemami Komputerowymi/OverlayFS]], [[Zarządzanie Systemami Komputerowymi/Btrfs]], [[Zarządzanie Systemami Komputerowymi/LVM]], [[Zarządzanie Systemami Komputerowymi/Kopia Migawkowa]]. Zawiera zebraną definicję **Copy-on-Write**, opis **ZFS**, rodzaje migawek i szczegóły systemów union.

## Copy-on-Write (CoW)
> **Kopiowanie przy zapisie** – dane współdzielone przez kilka „kopii” nie są fizycznie kopiowane w chwili tworzenia kopii. Fizyczna kopia bloku (strony, pliku) powstaje dopiero przy **pierwszej modyfikacji**. Modyfikacja nigdy nie nadpisuje oryginalnego bloku – nowa wersja trafia w **nowe miejsce**, a wskaźniki są aktualizowane.

### CoW w systemie plików (Btrfs, ZFS)
1. Zapis zmienionego bloku danych w **wolne miejsce**.
2. Zapis nowej wersji bloku metadanych (węzeł drzewa B), który wskazuje na nowy blok – także w nowe miejsce.
3. Propagacja zmian w górę drzewa aż do **korzenia**.
4. **Atomowa** podmiana wskaźnika korzenia (superblok / uberblock).

- **Spójność po awarii** bez fsck i bez klasycznego dziennika – przed podmianą korzenia widoczny jest stary, spójny stan, po niej nowy.
- **Migawka** = zachowanie starego korzenia (O(1), bez kopiowania danych); bloki zwalniane dopiero, gdy nie odwołuje się do nich żadna wersja (liczniki odwołań / czas narodzin bloku).
- **Klony / reflinki**: `cp --reflink=always a b` – nowy plik współdzieli bloki z oryginalnym (Btrfs, XFS, OCFS2).
- **Wady**: **fragmentacja** plików często modyfikowanych losowo (bazy danych, obrazy VM) – stąd `chattr +C` (no-CoW) w Btrfs; narzut na każdy zapis; potrzeba wolnego miejsca nawet do usunięcia danych.

### CoW w innych kontekstach
- `fork()` – strony pamięci współdzielone przez rodzica i potomka do pierwszego zapisu,
- migawki LVM (blokowe CoW z osobnym obszarem na stare bloki – [[Zarządzanie Systemami Komputerowymi/LVM#Kopie migawkowe w LVM]]),
- obrazy maszyn wirtualnych qcow2 (warstwa nad obrazem bazowym), warstwy obrazów kontenerów (OverlayFS kopiuje plik w górę – _copy-up_),
- deduplikacja stron pamięci VM (KSM) – [[Konstrukcja Systemów Chmurowych/Wirtualizacja Pamięci]].

Rysunek z wykładu (dowiązanie w Btrfs vs dowiązanie trwałe): [[ZSK_w_1.excalidraw]].

## Rodzaje migawek
| Rodzaj | Działanie | Koszt odczytu migawki | Koszt zapisu | Przykłady |
|---|---|---|---|---|
| **CoW (copy-on-first-write)** | przy pierwszej modyfikacji bloku **stara zawartość kopiowana do obszaru migawki**, nowa zapisywana w miejscu | migawka czyta z obszaru migawki lub oryginału | podwójny zapis przy pierwszej zmianie | LVM (klasyczne), VSS Windows |
| **ROW (redirect-on-write)** | nowa zawartość zapisywana **w nowe miejsce**, oryginał zostaje w miejscu | brak narzutu | brak kopiowania, fragmentacja | Btrfs, ZFS, LVM thin, NetApp WAFL |
| **klon / split-mirror** | pełna kopia (lustro odłączone od RAID-1) | pełna niezależność | zajmuje 100% miejsca | macierze, `lvconvert --splitmirrors` |
| **ciągła (CDP)** | log wszystkich zapisów, dowolny punkt w czasie | odtworzenie z logu | zapis do logu | [[Zarządzanie Systemami Komputerowymi/Ciągła ochrona danych]] |

**Spójność migawki**:
- **crash-consistent** – jak po nagłym odłączeniu zasilania,
- **application-consistent** – aplikacja/baza zamrożona lub opróżniona przed migawką (`fsfreeze`, VSS writers, `FLUSH TABLES WITH READ LOCK`).

## ZFS (Sun Microsystems, 2005; OpenZFS)
> System plików i menedżer woluminów w jednym, projektowany pod kątem **integralności danych** i prostoty administracji. Łączy RAID, woluminy, system plików, migawki i replikację.

### Architektura
- **Pula** (`zpool`) – cała przestrzeń z wielu urządzeń; systemy plików pobierają miejsce z puli dynamicznie (brak partycjonowania na sztywno).
- **vdev** – wirtualne urządzenie w puli: pojedynczy dysk, **mirror**, **RAID-Z1/Z2/Z3** (1/2/3 dyski parzystości); pula = stripe z vdevów.
- **Zbiory danych** (_datasets_): systemy plików (hierarchiczne, z dziedziczonymi właściwościami: `compression`, `quota`, `reservation`, `recordsize`, `atime`), **zvol** (urządzenie blokowe), migawki, klony.
- **DMU/ZPL** – warstwy obiektowe; całość to **drzewo Merkle** bloków.

### Integralność danych
- **Transakcyjny CoW** – grupy transakcji (TXG) zatwierdzane co kilka sekund, atomowa podmiana **uberbloku** → brak fsck.
- **Sumy kontrolne end-to-end** (fletcher4 / SHA-256) przechowywane **w bloku-rodzicu**, a nie przy danych. Wykrywa to ciche uszkodzenia (_bit rot_), błędne zapisy, zapisy w złe miejsce (_misdirected writes_).
- **Samonaprawianie** – przy błędzie sumy odczyt z innej kopii (mirror/RAID-Z/`copies=2`) i nadpisanie uszkodzonej.
- **scrub** – okresowa weryfikacja wszystkich bloków.
- **RAID-Z** – zmienna szerokość pasa, każdy zapis to pełny pas → brak „write hole” RAID-5 (niespójności parzystości po awarii w trakcie zapisu).

### Funkcje
- **migawki** (`zfs snapshot pool/home@2024-01-01`), **klony** (zapisywalne kopie migawek), rollback,
- **send/receive** – przyrostowa replikacja migawek (`zfs send -i @a pool/fs@b | ssh host zfs receive`) – tylko zmienione bloki, bez skanowania,
- **kompresja** (lz4, zstd, gzip) transparentna,
- **deduplikacja** blokowa (kosztowna – tabela DDT w RAM),
- **szyfrowanie** natywne na poziomie datasetu,
- **ARC** (_Adaptive Replacement Cache_) w RAM, **L2ARC** (cache odczytu na SSD), **ZIL/SLOG** – dziennik intencji dla zapisów synchronicznych (na szybkim SSD),
- rozmiary: 128-bitowy (praktycznie nieograniczony).

### Btrfs vs ZFS vs LVM
| | Btrfs | ZFS | LVM + ext4/XFS |
|---|---|---|---|
| Model | SP z woluminami (subvolumes) | pula + datasety | warstwa blokowa + osobny SP |
| CoW / sumy kontrolne | ✔ / ✔ | ✔ / ✔ (end-to-end) | ✘ (migawki CoW blokowe) / ✘ |
| RAID | 0,1,10 stabilne; 5/6 niestabilne | mirror, RAID-Z1/2/3 | mdadm / LVM RAID |
| Migawki | subwolumeny, tanie | tanie, send/receive | kosztowne dla wydajności (klasyczne), thin – lepsze |
| Licencja | GPL, w jądrze Linux | CDDL – moduł poza jądrem | GPL |
| Wymagania | umiarkowane | dużo RAM (ARC, dedup) | niskie |

## Systemy plików typu union – szczegóły
> Nakładają kilka katalogów (**gałęzi**/**warstw**) i pokazują ich **sumę** jako jedno drzewo. Zwykle niższe warstwy są tylko do odczytu, a zmiany trafiają do najwyższej warstwy zapisywalnej.

- **Wyszukiwanie**: plik szukany od najwyższej warstwy w dół; plik wyższej warstwy **przesłania** plik o tej samej nazwie niżej.
- **Katalogi**: zawartość **scalana** z wszystkich warstw.
- **Modyfikacja pliku z warstwy RO**: **copy-up** – plik kopiowany w całości do warstwy zapisywalnej, potem modyfikowany (kosztowne dla dużych plików).
- **Usunięcie pliku z warstwy RO**: nie da się fizycznie usunąć → w warstwie górnej tworzony **whiteout** (OverlayFS: urządzenie znakowe 0/0; unionfs-fuse: plik `.unionfs/nazwa_HIDDEN~` – [[Zarządzanie Systemami Komputerowymi/Lab 2 -#Grupowanie systemow plików]]; aufs: `.wh.nazwa`).
- **Usunięcie katalogu i utworzenie od nowa**: katalog **nieprzezroczysty** (_opaque_) – ukrywa zawartość niższych warstw.
- **Implementacje**: `mount -t union` (BSD, 4.4BSD-Lite), **UnionFS** (Linux, moduł/FUSE, wiele gałęzi RW), **aufs** (dawny Docker, Knoppix), **OverlayFS** (w jądrze od 3.18, `lowerdir`/`upperdir`/`workdir`, wiele warstw dolnych):
  ```bash
  mount -t overlay overlay -o lowerdir=/lower1:/lower2,upperdir=/upper,workdir=/work /merged
  ```
- **Zastosowania**: **kontenery** (warstwy obrazów + cienka warstwa zapisywalna kontenera – [[Docker Images]]), systemy live CD/USB (RO squashfs + tmpfs), Android, stacje bezdyskowe, piaskownice.

## Zarządcy woluminów – uzupełnienie
- **LVM**: PV (_physical volume_) → VG (_volume group_) → LV (_logical volume_); **PE/LE** (_physical/logical extents_) – odwzorowanie przez **device-mapper** w jądrze; `pvcreate`, `vgcreate`, `lvcreate -L 10G -n data vg0`, `lvextend -r`, `pvmove` (migracja danych na żywo między dyskami).
- **LVM thin provisioning** – pula cienka (`lvcreate --thin`), woluminy większe niż fizyczna pojemność; migawki thin (ROW, wydajne) – [[Zarządzanie Systemami Komputerowymi/Thin Provisioning]].
- **mdadm** – programowy RAID, **device-mapper** (dm-crypt, dm-cache, dm-thin, dm-multipath), **Stratis** (Red Hat, pule na XFS), **ZFS/Btrfs** – zarządzanie woluminami wbudowane w SP.
