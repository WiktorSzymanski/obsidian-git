---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 7
source: "slajdy–merged.pdf"
slajdy: "107–114"
---
# NPR 03. NFS, idempotentność i bezstanowość
---
> Krótki, ale pojęciowo najgęstszy fragment wykładu o RPC. Na przykładzie **NFS** pokazuje, co się dzieje, gdy interfejs zaprojektowany dla systemu lokalnego (uniksowe `open`/`read`/`lseek`, z **kursorem plikowym** jako stanem) trzeba odwzorować na interfejs zdalny, który ma być **bezstanowy**. Kończy się dwiema parami definicji — **idempotentność** i **bezstanowość**, każda w wariancie **postrzeganym** i **integralnym**.

---
## Architektura NFS
<sub>slajdy–merged.pdf, slajdy 107–108</sub>

NFS wkłada się pomiędzy proces a lokalny system plików, nie zmieniając interfejsu, który proces widzi. Kluczową warstwą jest **wirtualny system plików** (VFS): przechwytuje uniksowe wywołania i kieruje je albo do lokalnego systemu plików, albo do **klienta NFS**, który realizuje je przez **protokół RPC** na zdalnym **serwerze NFS**.

![[npr-nfs-s108-architektura-nfs.png]]
<sub>Wirtualny system plików rozdziela dostęp lokalny od zdalnego; między klientem a serwerem NFS działa protokół RPC. slajdy–merged.pdf, slajd 108</sub>

Symetria diagramu jest istotna: po stronie serwera także stoi VFS i lokalny system plików — serwer NFS jest zwykłym klientem swojego własnego systemu plików.

---
## Dwa interfejsy
<sub>slajdy–merged.pdf, slajdy 109–110</sub>

### Uniksowy interfejs systemu plików
<sub>slajd 109</sub>

```
open   (pathname, flags)     → fd
creat  (pathname, mode)      → fd
read   (fd, buf, count)
write  (fd, buf, count)
lseek  (fd, offset, whence)
close  (fd)
unlink (pathname)
```

### Interfejs zdalnego dostępu do pliku — protokół oparty na RPC
<sub>slajd 110</sub>

```
lookup (dirfh, name)              → fh, attr
create (dirfh, name, attr)        → newfh, attr
read   (fh, offset, count)        → data, attr
write  (fh, data, offset, count)  → attr
remove (dirfh, name)              → status
```

Różnice są systematyczne i wszystkie wynikają z jednej decyzji projektowej — **serwer nie ma pamiętać nic między wywołaniami**:

| Interfejs uniksowy | Interfejs NFS | Skąd różnica |
|---|---|---|
| `fd` — deskryptor, **indeks w tablicy serwera** | `fh` — uchwyt pliku (_file handle_), **samoopisujący się** | serwer nie trzyma tablicy otwartych plików |
| `read(fd, …)` czyta **od bieżącej pozycji** | `read(fh, offset, …)` czyta **od podanej pozycji** | kursor plikowy to stan — musi zniknąć z serwera |
| brak `lseek` w interfejsie NFS | — | kursor jest po stronie klienta, więc `lseek` nie wymaga komunikacji |
| `open(pathname)` — ścieżka **w całości** | `lookup(dirfh, name)` — **jeden składnik** naraz | serwer nie rozwija ścieżek; klient iteruje |
| brak `open`/`close` w interfejsie NFS | — | otwarcie pliku jest stanem, a stanu ma nie być |
| operacje **nie** zwracają atrybutów | prawie każda zwraca `attr` | klient musi móc odświeżyć swoją pamięć podręczną bez dodatkowego wywołania |

### Odwzorowanie interfejsu uniksowego na zdalny
<sub>slajd 111</sub>

Slajd 111 stawia pięć pytań, nie podając odpowiedzi — to zadanie do przemyślenia. Odpowiedzi wynikają wprost z tabeli powyżej:

| Pytanie ze slajdu | Odpowiedź |
|---|---|
| Jak wygląda realizacja uniksowej funkcji `open`? | ciąg wywołań `lookup`, po jednym na składnik ścieżki, począwszy od uchwytu katalogu montowania; wynikiem jest `fh`, które klient zapisuje w swojej tablicy otwartych plików |
| Jak wygląda realizacja uniksowej funkcji `creat`? | `lookup` po katalog nadrzędny, a następnie `create(dirfh, name, attr)` |
| Gdzie przechowywane są dane identyfikujące otwarty plik? | **po stronie klienta** — uchwyt `fh` i bieżąca pozycja w pliku; serwer nie wie, że plik jest otwarty |
| Jak wygląda realizacja `read`/`write`? | `read(fh, offset, count)` / `write(fh, data, offset, count)` z pozycją **doliczoną przez klienta**, a po powrocie klient przesuwa swój kursor |
| Jak wygląda realizacja `lseek`? | **bez żadnej komunikacji** — zmiana lokalnej zmiennej u klienta |

---
## Problem opisu stanu systemu
<sub>slajdy–merged.pdf, slajd 112</sub>

| Pojęcie | Definicja |
|---|---|
| **Stan postrzegany** (ang. _observable state_) | stan systemu/obiektu **z perspektywy klienta**, postrzegany poprzez **odpowiedzi serwera** |
| **Stan integralny** (ang. _inherent state_) | stan systemu **po stronie serwera**, mogący być **współtworzony przez niezależne zasoby** |

![[npr-nfs-s112-stan-postrzegany.png]]
<sub>Klient widzi wyłącznie odpowiedzi; na stan po stronie serwera składają się także zasoby, o których klient nic nie wie. slajdy–merged.pdf, slajd 112</sub>

Rozróżnienie jest po to, żeby móc powiedzieć, że coś jest bezstanowe **dla klienta**, mimo że serwer w środku oczywiście ma stan (zawartość dysku). Bez tego rozróżnienia żaden serwer plików nie byłby bezstanowy.

---
## Idempotentność procedur
<sub>slajdy–merged.pdf, slajd 113</sub>

Procedura/metoda/operacja zdalna jest **idempotentna**:

| Wariant | Warunek |
|---|---|
| **w sensie postrzegania** (ang. _observably idempotent_) | jej wywołanie z określonymi wartościami parametrów **zawsze daje taki sam efekt** (wynik, następstwa) — tzn. efekt ten jest w sposób jednoznaczny zdeterminowany **wartościami parametrów** |
| **w sensie integralnym** (ang. _inherently idempotent_) | jej wywołanie z określonymi wartościami parametrów **i przy określonym stanie zasobów** zawsze daje taki sam wynik — wynik jest jednoznacznie zdeterminowany **wartościami parametrów oraz stanem niezależnych zasobów** |

Wariant postrzegany jest **mocniejszy**: żąda, żeby sam zestaw parametrów wystarczał do przewidzenia wyniku. Wariant integralny dopuszcza, żeby wynik zależał dodatkowo od stanu zasobów — wymaga jedynie, żeby **przy niezmienionym stanie** powtórzenie dało to samo.

Na tym przykładzie widać różnicę między interfejsami: uniksowe `read(fd, buf, count)` **nie jest** idempotentne w sensie postrzegania, bo drugie wywołanie z tymi samymi parametrami zwróci **inne dane** (kursor się przesunął). NFS-owe `read(fh, offset, count)` **jest** — wynik zależy wyłącznie od parametrów i zawartości pliku.

To właśnie idempotentność uzasadnia retransmisję żądania przy **zaginionej odpowiedzi** z semantyką *co najmniej raz* — zob. [[NPR 01 Zdalne wywoływanie procedur (RPC)#Realizacja semantyki błędu|NPR 01]].

---
## Bezstanowość
<sub>slajdy–merged.pdf, slajd 114</sub>

Serwer/obiekt jest **bezstanowy** (ang. _stateless_):

| Wariant | Warunek |
|---|---|
| **w sensie postrzegania** | jeśli **wszystkie** jego procedury/metody/operacje są idempotentne **w sensie postrzegania** |
| **w sensie integralnym** | jeśli **wszystkie** jego procedury/metody/operacje są idempotentne **w sensie integralnym** |

Bezstanowość jest więc zdefiniowana **przez idempotentność**, a nie odwrotnie — to definicja o jeden poziom wyżej: własność pojedynczej operacji podniesiona do własności całego serwera przez kwantyfikator „wszystkie".

Konsekwencja praktyczna: **jedna operacja nieidempotentna wystarczy, by serwer przestał być bezstanowy**. Dlatego z interfejsu NFS usunięto `open`, `close` i `lseek` — nie dlatego, że były niewygodne, tylko dlatego, że każda z nich sama z siebie psuła własność całości.

Ten sam sposób rozumowania wraca później przy usługach REST — zob. [[13 Usługi sieciowe REST]] i [[14 Architektura zorientowana na zasoby - ROA, HATEOAS, niezawodność HTTP]].

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 107–114
> - **Budowa uchwytu pliku** (`fh`) — slajdy używają go jako pojęcia pierwotnego, nie mówiąc, z czego się składa ani dlaczego musi być samoopisujący się i trudny do podrobienia.
> - **Montowanie** (protokół `mount`) — skąd klient bierze pierwszy uchwyt katalogu, od którego zaczyna `lookup`.
> - **Pamięć podręczna NFS i jej spójność** — mimo że prawie każda operacja zwraca `attr` właśnie na potrzeby pamięci podręcznej, samego mechanizmu (ani jego modelu spójności) slajdy nie omawiają; modele spójności są w [[RSO 02 Danocentryczne modele spójności]].
> - **NFS v4 i stanowość** — wykład opisuje wyłącznie model bezstanowy, charakterystyczny dla NFS v2/v3.
> - **Odpowiedzi do slajdu 111** — slajd stawia pięć pytań i nie odpowiada na żadne; odpowiedzi w tabeli powyżej **zrekonstruowano** z zestawienia interfejsów ze slajdów 109–110, nie są cytatem z prezentacji.
