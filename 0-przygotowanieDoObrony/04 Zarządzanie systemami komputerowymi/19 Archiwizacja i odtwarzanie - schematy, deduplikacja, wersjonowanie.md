---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 19
---
# 19. Archiwizacja i odtwarzanie: schematy archiwizacji, macierze dyskowe, wersjonowanie, deduplikacja, rsync
---
> **Archiwizacja** (kopia zapasowa, _backup_) to wykonywanie kopii danych na odrębnym nośniku lub w odrębnej lokalizacji, aby można je było **odtworzyć** po awarii sprzętu, błędzie użytkownika, ataku (ransomware) lub katastrofie. Macierz RAID **nie jest** kopią zapasową – chroni tylko przed awarią dysku, a nie przed usunięciem czy uszkodzeniem danych.

## Cele i miary
- **RPO** (_Recovery Point Objective_) – maksymalna akceptowalna **utrata danych** mierzona czasem (np. 1 h → kopie co najmniej co godzinę).
- **RTO** (_Recovery Time Objective_) – maksymalny akceptowalny **czas przywrócenia** działania.
- **Okno archiwizacji** – czas, w którym można wykonać kopię bez zakłócania pracy.
- **Retencja** – jak długo przechowuje się kopie.

## Rodzaje kopii
| Rodzaj | Co zawiera | Archiwizacja | Odtwarzanie |
|---|---|---|---|
| **pełna** (_full_) | wszystkie dane | najdłuższa, największa | najprostsze: 1 kopia |
| **przyrostowa** (_incremental_) | zmiany od **ostatniej kopii dowolnego typu** | najszybsza, najmniejsza | pełna + **wszystkie** kolejne przyrostowe |
| **różnicowa** (_differential_) | zmiany od **ostatniej pełnej** | rośnie z czasem | pełna + **ostatnia** różnicowa |
| **syntetyczna pełna** | pełna złożona na serwerze z poprzedniej pełnej i przyrostów | bez obciążania klienta | jak pełna |
| **odwrotna przyrostowa** (_reverse incremental_) | aktualna pełna + różnice „wstecz” | – | najnowsza wersja od razu dostępna (rdiff-backup) |

Wykrywanie zmian: data modyfikacji (mtime/ctime), bit archiwizacji (Windows), sumy kontrolne, dziennik zmian systemu plików, migawki (Btrfs/ZFS send – bez skanowania).

### Poziomy `dump` (0–9)
- poziom **0** – kopia pełna,
- poziom **n** – kopia wszystkich plików zmienionych od ostatniej kopii o **niższym** poziomie.
- Przykład: 0 w niedzielę, 1 w pozostałe dni → różnicowe; sekwencja 0,1,2,3,4,5,6 → przyrostowe; sekwencje mieszane (np. 0, 3, 2, 5, 4, 7, 6) – kompromis między liczbą kopii potrzebnych do odtworzenia a ich rozmiarem.

## Schematy rotacji nośników
### Dziadek-Ojciec-Syn (GFS)
- **Syn** – kopie dzienne (np. przyrostowe, pon–pt), nośniki używane cyklicznie co tydzień,
- **Ojciec** – kopie tygodniowe (pełne), przechowywane np. 4–5 tygodni,
- **Dziadek** – kopie miesięczne (pełne), przechowywane np. rok (lub roczne – dłużej).
- Daje dużą liczbę punktów przywracania przy ograniczonej liczbie nośników.

### Wieża Hanoi
- Nośnik A używany co 2 dni, B co 4, C co 8, D co 16 dni… (jak ruchy krążków).
- Punkty przywracania **wykładniczo rozrzedzone** w czasie: dużo nowych, mało starych.
- Podobnie działa **wykładnicze przedawnianie** w narzędziach (Borg `prune --keep-daily=7 --keep-weekly=4 --keep-monthly=12`) – [[Zarządzanie Systemami Komputerowymi/Borg Backup]].

### Zasada 3-2-1
- **3** kopie danych (produkcyjna + 2 zapasowe),
- na **2** różnych rodzajach nośników,
- **1** kopia poza siedzibą (_offsite_).

Wersja 3-2-1-1-0: + 1 kopia **offline/niezmienialna** (_immutable_, WORM, _air gap_ – ochrona przed ransomware) + 0 błędów weryfikacji odtwarzania.

## Nośniki i hierarchia składowania
- **taśmy** (LTO – tanie, trwałe, offline, dostęp sekwencyjny), **dyski** (szybkie, dostęp swobodny), **chmura/obiektowe** (S3 z _object lock_), **biblioteki taśmowe** z robotami,
- **HSM** – migracja rzadko używanych plików na wolniejsze nośniki z pozostawieniem odnośnika ([[Zarządzanie Systemami Komputerowymi/Funkcjonalność systemów CAD]]),
- **backup-to-disk-to-tape** – szybka kopia na dysk (holding disk w [[Zarządzanie Systemami Komputerowymi/Amanda|Amandzie]]), potem przelanie na taśmę.

## Macierze dyskowe
Macierz dyskowa łączy wiele dysków w jedno logiczne urządzenie dla **wydajności i dostępności**:
- **RAID** 0/1/5/6/10, hot spare, odbudowa – szczegóły: [[Konstrukcja Systemów Chmurowych/RAID]],
- programowy (mdadm, LVM, Btrfs, ZFS RAID-Z) vs sprzętowy (kontroler z pamięcią podtrzymywaną bateryjnie),
- **DAS** (bezpośrednio podłączona), **NAS** (współdzielenie plików przez sieć – NFS/SMB), **SAN** (dostęp blokowy – FC/iSCSI) – [[Zarządzanie Systemami Komputerowymi/Network Attached Storage]], [[Zarządzanie Systemami Komputerowymi/Storage Area Network]], [[36 Systemy składowania danych - iSCSI, multipath, OCFS, DRBD]],
- macierze z funkcjami archiwizacyjnymi: migawki, replikacja do drugiej macierzy (synchroniczna/asynchroniczna), klony.
- **RAID ≠ backup**: przypadkowe `rm`, błąd aplikacji czy szyfrowanie przez ransomware natychmiast dotykają wszystkich dysków macierzy.

## Wersjonowanie
Przechowywanie **wielu wersji** tych samych danych w czasie:
- **migawki** systemów plików/wolumenów (LVM, Btrfs, ZFS) – szybkie, oszczędne (CoW), ale na tym samym nośniku ([[Zarządzanie Systemami Komputerowymi/Kopia Migawkowa]]),
- **rsync + dowiązania twarde** (`--link-dest`) – każda kopia wygląda jak pełna, niezmienione pliki współdzielą i-węzeł ([[Zarządzanie Systemami Komputerowymi/Rsync#Rsync + snapshots]]; Time Machine – [[Zarządzanie Systemami Komputerowymi/Mac OS X - Time Machine]]),
- **różnice odwrotne** – bieżąca wersja + łatki wstecz ([[Zarządzanie Systemami Komputerowymi/Pakiet rdiff-backup]]),
- **repozytoria z deduplikacją** – każda „archiwizacja” (_archive_) to lista odwołań do fragmentów (Borg, restic),
- **ciągła ochrona danych (CDP)** – zapis każdej zmiany, odtworzenie do dowolnego momentu ([[Zarządzanie Systemami Komputerowymi/Ciągła ochrona danych]]),
- **systemy kontroli wersji** (Git) – dla kodu i konfiguracji.

## Deduplikacja
> Eliminacja powtarzających się danych: identyczne porcje są przechowywane **raz**, a pozostałe wystąpienia zastępowane **odwołaniami**. Identyfikacja porcji przez **skrót kryptograficzny** (SHA-1/SHA-256).

### Poziom
| Poziom | Opis | Przykład |
|---|---|---|
| **plikowa** (_single-instance storage_) | identyczne całe pliki przechowywane raz | BackupPC (dowiązania twarde, pula plików) – [[Zarządzanie Systemami Komputerowymi/BackupPC]] |
| **blokowa – stały rozmiar bloku** | plik dzielony na bloki np. 4 KiB | ZFS dedup, SquashFS; wstawienie bajtu na początku przesuwa wszystkie bloki → brak dopasowań |
| **blokowa – zmienny rozmiar** (_content-defined chunking_) | granice fragmentów wyznaczane **treścią**: skrót kroczący (Rabin fingerprint / buzhash) na oknie; granica, gdy skrót spełnia warunek (np. $n$ najmłodszych bitów = 0) | Borg, restic, Data Domain; wstawienie danych zmienia tylko 1–2 fragmenty |

### Miejsce i czas
- **po stronie źródła** (klient liczy skróty, wysyła tylko nowe fragmenty – oszczędza sieć) vs **po stronie celu** (serwer/urządzenie),
- **inline** (w trakcie zapisu, wymaga CPU/RAM na indeks skrótów) vs **post-process** (po zapisie, potrzebne dodatkowe miejsce tymczasowe).

### Zalety i ryzyka
- ✔ ogromna oszczędność przy wielu podobnych kopiach (maszyny wirtualne, codzienne kopie pełne): współczynnik 10–50×,
- ✔ szybsze kopie przez sieć,
- ✘ indeks skrótów w pamięci (ZFS dedup: ok. 5 GiB RAM / 1 TiB), narzut CPU,
- ✘ uszkodzenie jednego fragmentu dotyka **wszystkich** kopii, które go używają → potrzebna redundancja i weryfikacja,
- ✘ kolizje skrótów (teoretyczne przy SHA-256),
- ✘ szyfrowanie po stronie klienta z różnymi kluczami uniemożliwia deduplikację między klientami.

## Protokół rsync
Algorytm synchronizacji delta (sumy słabe kroczące + MD5, wyszukiwanie bloków w dowolnym przesunięciu) – szczegółowo w [[Zarządzanie Systemami Komputerowymi/Rsync#Algorytm Rsync]]. Najważniejsze:
- przesyłana jest tylko różnica: $B$ wysyła sumy bloków swojej wersji, $A$ wyszukuje dopasowania i odsyła instrukcje (odwołanie do bloku lub nowe dane),
- **suma krocząca** $a(k,l) = \left(\sum_{i=k}^{l} X_i\right) \bmod M$, $b(k,l) = \left(\sum_{i=k}^{l}(l-i+1)X_i\right) \bmod M$, $s = a + 2^{16} b$ – daje się przesunąć o 1 bajt w $O(1)$,
- tryby: przez SSH lub demon `rsyncd` (moduły, `rsync://`), opcje `-a` (archiwum), `--delete`, `--link-dest`, `-z`, `--checksum`,
- **zsync** – odwrócenie ról dla wielu klientów:
  1. serwer **jednorazowo** tworzy plik kontrolny `.zsync` (sumy kroczące i skróty MD4 bloków + SHA-1 całości),
  2. klient pobiera `.zsync` przez HTTP, skanuje swoją starą kopię sumą kroczącą i znajduje posiadane bloki,
  3. brakujące bloki pobiera żądaniami **HTTP Range** – serwer nie wykonuje żadnych obliczeń per klient (zwykły serwer WWW),
  ([[Zarządzanie Systemami Komputerowymi/Zsync]]),
- **Unison** – synchronizacja dwukierunkowa z wykrywaniem konfliktów ([[Zarządzanie Systemami Komputerowymi/Unison]]).

## Odtwarzanie
- **testowanie odtwarzania** (kopia niesprawdzona = brak kopii), procedury DR (_disaster recovery_), dokumentacja,
- poziomy: pojedynczy plik, cały system (_bare-metal restore_), maszyna wirtualna ([[Zarządzanie Systemami Komputerowymi/Archiwizacja a wirtualizacja]]), baza danych (spójność: migawka + log transakcji, _point-in-time recovery_),
- **spójność kopii** otwartych plików: migawka (LVM/VSS), zamrożenie aplikacji (`fsfreeze`, agent), zrzut bazy.

## Zobacz też
- [[Zarządzanie Systemami Komputerowymi/Systemy Plików]], [[Systemy plików a archiwizacja]]
- [[21 Lokalne systemy plików - CoW, ZFS, migawki]]
