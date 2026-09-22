---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 20
---
# 20. Rozproszone systemy plików
---
> **Rozproszony system plików** (DFS) umożliwia wielu klientom dostęp do plików przechowywanych na zdalnych serwerach **tak, jakby były lokalne**. Musi przy tym zapewnić współdzielenie, przezroczystość, spójność przy współbieżnym dostępie, wydajność (buforowanie), niezawodność (replikacja) i bezpieczeństwo.

## Modele dostępu
### Model ładowania i rozładowania (_upload/download_)
- Przy otwarciu **cały plik** przesyłany do klienta, operacje wykonywane lokalnie, po zamknięciu zmieniony plik odsyłany na serwer.
- ✔ proste, szybkie operacje po pobraniu, mało komunikacji w trakcie pracy
- ✘ potrzebne miejsce u klienta, nieefektywne dla dużych plików i małych zmian, trudna współbieżna modyfikacja
- Przykład: AFS (buforowanie całych plików), FTP.

### Model dostępu zdalnego (_remote access_)
- Plik pozostaje na serwerze; klient wysyła żądania **pojedynczych operacji** (read/write na fragmentach, getattr).
- ✔ dostęp do dużych plików, spójność kontrolowana przez serwer
- ✘ każda operacja to komunikacja → zależność od sieci i obciążenie serwera
- Przykład: NFS, SMB/CIFS (z buforowaniem bloków).

## Przezroczystość w DFS
- **dostępu** – te same wywołania (`open`, `read`) dla plików lokalnych i zdalnych (VFS w jądrze),
- **położenia** – nazwa pliku nie ujawnia serwera (`/home/jan` zamiast `serwer1:/export/home/jan`),
- **niezależność od położenia** (_location independence_) – nazwa nie zmienia się, gdy plik migruje na inny serwer (AFS – wolumeny; NFS – nie),
- **replikacji**, **współbieżności**, **awarii**, **skalowalności**.

### Nazewnictwo
- **montowanie po stronie klienta** (NFS): każdy klient montuje zdalne katalogi w dowolnych miejscach → przestrzenie nazw mogą się różnić między klientami,
- **globalna, jednolita przestrzeń nazw** (AFS: `/afs/komorka/…`, DFS Windows: `\\domena\dfs\…`): wszyscy klienci widzą to samo drzewo.

## Semantyki współbieżnego dostępu
| Semantyka | Opis | Przykład |
|---|---|---|
| **UNIX (jednej kopii)** | każdy odczyt widzi wynik **ostatniego zapisu**, nawet innego procesu na innej maszynie; zmiany widoczne natychmiast | lokalny SP; w DFS wymaga natychmiastowej propagacji / blokad (klastrowe GFS2, OCFS2) |
| **sesji** | zmiany widoczne dla innych dopiero po **zamknięciu** pliku; przy współbieżnych zapisach **wygrywa ostatnie zamknięcie** (_last close wins_) | AFS |
| **plików niezmiennych** | pliki można tylko tworzyć i czytać; „modyfikacja” = nowy plik (nowa wersja) | systemy archiwalne, obiektowe (S3 – obiekty), HDFS (append-only) |
| **transakcyjna** | operacje na plikach w atomowych transakcjach (ACID) | systemy transakcyjne (Locus), bazy danych |
| **zamknięcia-do-otwarcia** (_close-to-open_) | klient otwierający plik widzi wersję po ostatnim zamknięciu (sprawdza atrybuty przy `open`); w trakcie – cache | NFS |

Mechanizmy: blokady (_advisory_/_mandatory_, byte-range), leasingi i delegacje (NFSv4), oportunistyczne blokady (SMB), wywołania zwrotne (AFS).

## Serwer stanowy vs bezstanowy
| | Bezstanowy (NFSv2/v3) | Stanowy (NFSv4, SMB, AFS) |
|---|---|---|
| Informacja o klientach | brak (każde żądanie samowystarczalne: uchwyt pliku + offset) | otwarte pliki, pozycje, blokady, cache klienta |
| Awaria serwera | restart i obsługa od nowa – klient powtarza żądania | utrata stanu → procedura odtwarzania (okres łaski, klienci odzyskują blokady) |
| Awaria klienta | nie dotyczy serwera | serwer musi wykryć i zwolnić zasoby (lease) |
| Operacje | powinny być **idempotentne** | open/close, blokady wbudowane |
| Wydajność | większe komunikaty, brak optymalizacji | krótsze żądania, read-ahead, delegacje |
| Blokady plików | poza protokołem (NLM, `lockd`/`statd`) | w protokole |

Zob. [[Zarządzanie Systemami Komputerowymi/Bezstanowy Serwer Plików]].

## Buforowanie (_caching_) po stronie klienta
- **gdzie**: pamięć RAM (szybko) lub dysk lokalny (trwały, duży – AFS),
- **granulacja**: bloki vs całe pliki,
- **walidacja**:
  - **inicjowana przez klienta** – sprawdzanie atrybutów (mtime) przy otwarciu lub okresowo (NFS: cache atrybutów 3–60 s),
  - **inicjowana przez serwer** – serwer powiadamia o zmianie (AFS _callback_, SMB oplock break, NFSv4 odwołanie delegacji).

## Replikacja w DFS
- **Cele**: dostępność (awaria serwera), wydajność (lokalny odczyt), skalowalność.
- **Przezroczystość replikacji** – klient nie wie, z której kopii korzysta.
- **Sposoby**:
  - **jawna** – klient/aplikacja sama wybiera repliki,
  - **leniwa** (_lazy_) – zapis na jednej kopii (primary), propagacja do pozostałych w tle (repliki tylko do odczytu – AFS volumes RO),
  - **grupowa** – zapis wysyłany do wszystkich replik jednocześnie (rozgłaszanie – Coda, GlusterFS replicated),
- **Protokoły spójności**: primary-backup, **ROWA** (_read-one-write-all_), głosowanie z kworum $R + W > N$ ([[Systemy Wysokiej Niezawodności/Algorytm Gifforda]]), wersje/wektory wersji do wykrywania konfliktów (Coda),
- replikacja **plików niezmiennych** nie wymaga kontroli spójności ([[Konstrukcja Systemów Chmurowych/Modele pików]]).

---
## NFS (Network File System, Sun, 1984)
- Model **dostępu zdalnego**, protokół oparty na **ONC RPC** z kodowaniem **XDR**; montowanie po stronie klienta.
- **Uchwyt pliku** (_file handle_) – nieprzezroczysty identyfikator pliku nadany przez serwer (FS id + i-węzeł + generacja); pozwala na bezstanowość.
- **NFSv2** (1989, RFC 1094): UDP, bezstanowy, pliki do 2 GiB, zapisy synchroniczne; osobne protokoły MOUNT i NLM (blokady, `lockd` + `statd` do odtwarzania po awarii).
- **NFSv3** (1995, RFC 1813): 64-bitowe rozmiary, TCP, **zapisy asynchroniczne** z `COMMIT` (serwer może buforować, klient ponawia po restarcie serwera wykrytym przez _write verifier_), `READDIRPLUS`, lepsze atrybuty.
- **NFSv4** (2000/2003, RFC 7530): **stanowy** – `OPEN`/`CLOSE`, blokady w protokole z **leasingiem**, **delegacje** (serwer oddaje klientowi prawo lokalnej obsługi pliku, odwołuje przez callback), **operacje złożone** (`COMPOUND` – wiele operacji w jednym RPC), jeden port **2049** (łatwy firewall), pseudo-system plików (jedna przestrzeń eksportów), **RPCSEC_GSS / Kerberos** (uwierzytelnianie, integralność, szyfrowanie), ACL zbliżone do NTFS, nazwy użytkowników zamiast UID.
- **NFSv4.1** – sesje (exactly-once semantics), **pNFS** (równoległy dostęp do danych bezpośrednio na serwerach danych, serwer metadanych rozdziela układ); **v4.2** – kopiowanie po stronie serwera, pliki rzadkie.
- Semantyka **close-to-open**, tryby eksportu `sync`/`async` – [[Zarządzanie Systemami Komputerowymi/NFS]].

## CIFS / SMB (Microsoft)
- **SMB** (_Server Message Block_, IBM/Microsoft, lata 80.), **CIFS** (1996) – dialekt SMB1 upubliczniony jako „internetowy” system plików; potem SMB2 (Vista, 2006) i SMB3 (Windows 8/2012).
- Protokół **stanowy, sesyjny**: `NEGOTIATE` → `SESSION_SETUP` (uwierzytelnienie NTLM/Kerberos) → `TREE_CONNECT` (udział `\\serwer\udział`) → `CREATE`/`READ`/`WRITE`/`CLOSE`; także drukarki, potoki nazwane (RPC).
- Transport: NetBIOS over TCP (137–139) lub bezpośrednio TCP **445**.
- **Oportunistyczne blokady** (_oplocks_): serwer daje klientowi prawo lokalnego buforowania – **exclusive** (odczyt+zapis w cache), **batch** (także odroczenie open/close), **level II** (tylko cache odczytu, wielu klientów); gdy inny klient otwiera plik → **oplock break** (klient opróżnia bufory). W SMB2+ zastąpione **leasingami** (także na katalogach).
- Semantyka zbliżona do lokalnej Windows: **tryby współdzielenia** (_share modes_) przy otwarciu, blokady zakresów bajtów wymuszane (_mandatory_).
- **SMB2**: mniej komend (19 zamiast >100), łączenie komend, większe bufory, _durable handles_ (przetrwanie chwilowego rozłączenia).
- **SMB3**: **multichannel** (wiele połączeń/interfejsów), **szyfrowanie** end-to-end (AES-CCM/GCM), **transparent failover** (klastry), SMB Direct (RDMA), **scale-out file server**.
- Implementacja uniksowa: **Samba** – [[Zarządzanie Systemami Komputerowymi/Samba]].

## AFS (Andrew File System, Carnegie Mellon, 1983)
Projektowany dla **skalowalności** (tysiące stacji kampusu).
- **Architektura**: serwery **Vice** i klient **Venus** (cache manager) na każdej stacji.
- **Buforowanie całych plików na dysku lokalnym** (model upload/download):
  1. przy `open` Venus sprawdza, czy ma ważną kopię, jeśli nie – pobiera cały plik (w nowszych wersjach duże fragmenty),
  2. `read`/`write` wykonywane **lokalnie** bez komunikacji,
  3. przy `close` zmieniony plik odsyłany na serwer.
- **Obietnica wywołania zwrotnego** (_callback promise_): serwer zapamiętuje, którzy klienci mają kopię, a przy zmianie pliku wysyła im **callback break** → kopia unieważniona. Klient nie musi odpytywać serwera przy każdym otwarciu – główne źródło skalowalności.
- **Semantyka sesji** (last close wins) – współbieżna edycja tego samego pliku nie jest synchronizowana.
- **Globalna przestrzeń nazw** `/afs/<komórka>/…`; **komórki** (_cells_) – niezależne domeny administracyjne.
- **Wolumeny** – jednostki administracji (np. katalog domowy), mogą **migrować** między serwerami (niezależność od położenia – baza lokalizacji wolumenów VLDB); **repliki tylko do odczytu** wolumenów, **migawki** (backup volume).
- **Bezpieczeństwo**: uwierzytelnianie **Kerberos**, **ACL na katalogach** (prawa `rlidwka`), grupy definiowane przez użytkowników.
- Implementacje: Transarc AFS → IBM, **OpenAFS**, kAFS w jądrze Linux; wpływ na DCE/DFS i NFSv4.

## Coda (CMU, 1987–)
Następca AFS nastawiony na **wysoką dostępność** i **pracę bez połączenia** (klienci mobilni).
### Replikacja serwerów
- Wolumen replikowany na grupie serwerów **VSG** (_Volume Storage Group_).
- **AVSG** (_Accessible VSG_) – serwery aktualnie osiągalne dla klienta.
- Odczyt z jednego (preferowanego) serwera AVSG, **zapis równolegle do wszystkich** serwerów AVSG (strategia optymistyczna – zapis możliwy, jeśli choć jeden serwer dostępny).
- Spójność: **wektory wersji CVV** (_Coda Version Vector_) dla każdego pliku; po połączeniu partycji porównanie wektorów → jeśli jeden dominuje, aktualizacja; jeśli **współbieżne** → **konflikt** (rozwiązanie automatyczne dla katalogów, ręczne lub przez ASR – _application-specific resolvers_ – dla plików).

### Praca bez połączenia (_disconnected operation_) – stany klienta Venus
1. **Gromadzenie** (_hoarding_) – przy połączeniu klient buforuje pliki wg **bazy gromadzenia** (HDB – lista plików z priorytetami od użytkownika + ostatnio używane), okresowy _hoard walk_.
2. **Emulacja** (_emulation_) – po utracie połączenia Venus sam obsługuje żądania z cache i zapisuje modyfikacje w **logu CML** (_Client Modification Log_, z optymalizacjami, np. usuwanie zbędnych operacji).
3. **Reintegracja** (_reintegration_) – po odzyskaniu połączenia log jest odtwarzany na serwerach; konflikty wykrywane i zgłaszane.
- Tryb **słabego połączenia** (_write disconnected_) – asynchroniczna, stopniowa reintegracja przy wolnym łączu.

## GFS / GFS2 (Global File System, Sistina → Red Hat)
> [!note] W kontekście listy „GFS, OCFS” chodzi najpewniej o klastrowy **Red Hat GFS**. Poniżej także Google File System dla porządku.

- **Klastrowy system plików ze współdzielonym dyskiem** (_shared-disk_): wszystkie węzły mają bezpośredni dostęp blokowy do tego samego urządzenia (SAN/iSCSI, DRBD dual-primary). Brak serwera plików – każdy węzeł sam montuje system plików.
- **Rozproszony menedżer blokad DLM** koordynuje dostęp do metadanych i danych (blokady na i-węzłach, zasobach), a węzły buforują dane pod ochroną blokad.
- **Dziennik na każdy węzeł** – awaria węzła → inny węzeł odtwarza jego dziennik.
- **Fencing** (STONITH) – odcięcie węzła podejrzanego o awarię od dysku, zanim zostanie odtworzony (ochrona przed uszkodzeniem danych przy podziale klastra – _split brain_).
- Semantyka **zgodna z POSIX** (jak lokalny SP) mimo wielu węzłów.
- Integracja z Pacemaker/Corosync (członkostwo, kworum).
- **GFS2** (w jądrze Linux od 2.6.19) – przebudowa GFS: lepsza wydajność, dziennik jako plik, prostsze metadane.
- Zastosowania: klastry HA, współdzielone obrazy maszyn wirtualnych, aplikacje wymagające wspólnego systemu plików na wielu węzłach.

### Google File System (2003) – dla porównania
- Jeden **master** (metadane w RAM, log operacji) + wiele **chunkserverów**; pliki dzielone na **fragmenty 64 MiB** replikowane domyślnie **3×**.
- Klient pobiera od mastera lokalizację fragmentów i czyta/zapisuje **bezpośrednio** u chunkserverów; **lease** dla repliki primary ustala kolejność mutacji.
- Optymalizacja pod duże pliki, sekwencyjne odczyty i **dopisywanie** (_record append_); **złagodzona spójność**.
- Pierwowzór HDFS – [[53 Przetwarzanie dużych danych - Hadoop i Apache Spark]].

## OCFS / OCFS2 (Oracle Cluster File System)
- **OCFS** (2002) – klastrowy SP ze współdzielonym dyskiem dla Oracle RAC; przeznaczony tylko do plików bazy danych (nie w pełni POSIX).
- **OCFS2** (2005, w jądrze Linux od 2.6.16) – ogólnego przeznaczenia, **zgodny z POSIX**:
  - **współdzielony dysk** (SAN, iSCSI, DRBD), wszystkie węzły montują jednocześnie,
  - **o2cb** – stos klastrowy: **heartbeat dyskowy** (każdy węzeł zapisuje znacznik w obszarze heartbeat na dysku) + **sieciowy**, członkostwo, **o2dlm** – rozproszony menedżer blokad,
  - **dziennik na węzeł** (JBD2), **sloty węzłów** (maks. liczba węzłów ustalana przy formatowaniu),
  - **fencing przez samorestart** – węzeł, który utracił łączność z kworum, restartuje się (_self-fencing_),
  - ekstenty, pliki rzadkie, **reflinki** (kopie CoW plików), rozszerzone atrybuty, ACL POSIX, kwoty.
- Zastosowania: Oracle RAC, współdzielone repozytoria maszyn wirtualnych (Oracle VM), klastry aplikacyjne.
- Możliwość użycia z pacemakerem zamiast o2cb.

## Porównanie systemów
| System | Architektura | Model dostępu | Stan | Semantyka | Replikacja / HA | Buforowanie i spójność |
|---|---|---|---|---|---|---|
| NFSv3 | klient-serwer | zdalny | bezstanowy | close-to-open | brak (HA przez klastrowanie serwera) | cache atrybutów z timeoutem |
| NFSv4 | klient-serwer | zdalny | stanowy | close-to-open | pNFS, repliki (fs_locations) | delegacje z callbackiem |
| SMB/CIFS | klient-serwer | zdalny | stanowy | ≈ lokalny Windows | SMB3 failover, DFS-R | oplocks / leases |
| AFS | klient-serwer, komórki | upload/download (całe pliki) | stanowy (callbacki) | sesji | repliki RO wolumenów | callback promise |
| Coda | klient-serwer | upload/download | stanowy | sesji + konflikty | VSG/AVSG, wektory wersji | hoarding, praca offline |
| GFS2 | współdzielony dysk | bezpośredni blokowy | – | POSIX | przez SAN/DRBD, fencing | DLM |
| OCFS2 | współdzielony dysk | bezpośredni blokowy | – | POSIX | przez SAN/DRBD, self-fencing | o2dlm, heartbeat dyskowy |

## Zobacz też
- Skalowalne klastrowe systemy plików: [[Konstrukcja Systemów Chmurowych/GlusterFS]], [[Konstrukcja Systemów Chmurowych/Ceph]]
- [[36 Systemy składowania danych - iSCSI, multipath, OCFS, DRBD]]
