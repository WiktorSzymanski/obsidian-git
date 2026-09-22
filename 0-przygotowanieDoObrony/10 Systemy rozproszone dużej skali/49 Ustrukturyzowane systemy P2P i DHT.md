---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 49
---
# 49. Ustrukturyzowane systemy P2P – algorytmy routingu, DHT, przykłady systemów
---
> W **ustrukturyzowanych** sieciach P2P topologia nakładki i **rozmieszczenie danych** są ściśle określone przez **przestrzeń identyfikatorów**. Węzły i klucze danych dostają identyfikatory z tej samej przestrzeni, a każdy klucz jest przechowywany na **deterministycznie wyznaczonym** węźle. Pozwala to znaleźć dowolny klucz w **$O(\log N)$ skokach** z tablicą routingu rozmiaru **$O(\log N)$**, z gwarancją znalezienia, jeśli klucz istnieje.

Kontrast z sieciami nieustrukturyzowanymi: [[48 Nieustrukturyzowane systemy P2P]].

## DHT – rozproszona tablica haszująca
> **DHT** (_Distributed Hash Table_) udostępnia interfejs tablicy haszującej rozproszonej na wiele węzłów.

**Interfejs**:
- `put(klucz, wartość)` – zapis,
- `get(klucz) → wartość` – odczyt,
- podstawowa operacja: **`lookup(klucz) → węzeł`** odpowiedzialny za klucz (reszta to aplikacja).

**Przestrzeń identyfikatorów**: $m$-bitowe liczby (np. $m = 160$); $ID_{węzła} = hash(adres\ IP \| port)$ lub klucz publiczny, $ID_{klucza} = hash(nazwa\ zasobu)$ – funkcje kryptograficzne (SHA-1) dają równomierny rozkład.

**Wymagania**: równomierne obciążenie, decentralizacja, skalowalność (koszt $O(\log N)$), odporność na churn i awarie, efektywność (bliskość w sieci fizycznej).

### Spójne haszowanie (_consistent hashing_, Karger i in. 1997)
- Węzły i klucze na **okręgu** identyfikatorów $[0, 2^m)$.
- Klucz $k$ przypisany do **następnika** (_successor_) – pierwszego węzła o $ID \geq k$ (zgodnie z ruchem wskazówek zegara).
- **Dołączenie/odejście** węzła przenosi tylko klucze z **jednego sąsiedniego przedziału** – średnio $K/N$ kluczy, a nie wszystkie (jak przy `hash mod N`).
- **Węzły wirtualne** – każdy fizyczny węzeł ma wiele pozycji na okręgu → równomierniejszy rozkład, możliwość uwzględnienia pojemności (Dynamo, Cassandra).

---
## Chord (Stoica, Morris, Karger, Kaashoek, Balakrishnan – MIT, SIGCOMM 2001)
### Struktura
- **Pierścień** identyfikatorów $m$-bitowych (SHA-1), klucz $k$ na węźle $successor(k)$.
- Każdy węzeł $n$ zna:
  - **następnika** (_successor_) i **poprzednika** (_predecessor_),
  - **tablicę palców** (_finger table_) – $m$ wpisów: $finger[i] = successor\left(n + 2^{i-1} \bmod 2^m\right)$ dla $i = 1..m$ → wskaźniki na odległość 1, 2, 4, 8, … połowy okręgu,
  - **listę następników** (_successor list_) – $r$ kolejnych następników (odporność na awarie).
- Tylko $O(\log N)$ **różnych** wpisów w tablicy palców.

### Wyszukiwanie
```
n.find_successor(id):
    if id ∈ (n, successor]:
        return successor
    else:
        n' := closest_preceding_node(id)
        return n'.find_successor(id)          // iteracyjnie lub rekurencyjnie

n.closest_preceding_node(id):
    for i = m downto 1:
        if finger[i] ∈ (n, id):               // przedział otwarty na okręgu
            return finger[i]
    return n
```
Każdy skok **co najmniej połowi** odległość do celu → **$O(\log N)$ skoków** (średnio $\frac{1}{2}\log_2 N$).

**Przykład** ($m = 6$, węzły 1, 8, 14, 21, 32, 38, 42, 48, 51, 56): tablica palców węzła 8: $8+1 \to 14$, $8+2 \to 14$, $8+4 \to 14$, $8+8 \to 21$, $8+16 \to 32$, $8+32 \to 42$. Lookup(54) z węzła 8: najbliższy poprzedzający palec to 42 → w tablicy 42: $42+8=50 \to 51$ poprzedza 54 → z 51 następnik 56 ∋ 54 → **wynik 56**.

### Dołączanie i stabilizacja
1. Nowy węzeł $n$ zna dowolny węzeł $n'$ → `successor := n'.find_successor(n)` (poprzednik nieustalony).
2. **Okresowo** każdy węzeł wykonuje:
   - **`stabilize()`**: pyta następnika o jego poprzednika $x$; jeśli $x \in (n, successor)$ → `successor := x`; następnie `successor.notify(n)`,
   - **`notify(n')`**: jeśli `predecessor` nieustalony lub $n' \in (predecessor, n)$ → `predecessor := n'`,
   - **`fix_fingers()`**: odświeża losowy lub kolejny wpis tablicy palców,
   - **`check_predecessor()`**: jeśli poprzednik nie odpowiada → `predecessor := nil`.
3. Następnik **przekazuje** nowemu węzłowi klucze z przedziału $(predecessor, n]$.
- Poprawność: stabilizacja utrzymuje poprawne wskaźniki następników mimo współbieżnych dołączeń; do tego czasu wyszukiwania są wolniejsze, ale nadal poprawne.
- **Koszt dołączenia**: $O(\log^2 N)$ komunikatów do zbudowania tablicy palców.
- **Awaria węzła**: lista $r = O(\log N)$ następników pozwala pominąć martwe; **replikacja danych** na kolejnych $r$ następnikach.

---
## Pastry (Rowstron, Druschel – Microsoft/Rice, Middleware 2001)
- **Identyfikatory 128-bitowe**, traktowane jako ciągi cyfr o podstawie $2^b$ (typowo $b = 4$ → cyfry szesnastkowe).
- Klucz przechowywany na węźle o identyfikatorze **numerycznie najbliższym** kluczowi.
- **Stan węzła**:
  - **tablica routingu**: $\lceil \log_{2^b} N \rceil$ wierszy × $2^b$ kolumn; w wierszu $l$ kolumnie $d$ – węzeł, który ma z bieżącym **wspólny prefiks długości $l$** i $(l+1)$-szą cyfrę równą $d$; spośród kandydatów wybierany **najbliższy w sieci fizycznej** (metryka bliskości, np. RTT),
  - **zbiór liści** (_leaf set_) $L$ – $|L|/2$ węzłów o najbliższych **większych** i $|L|/2$ **mniejszych** identyfikatorach (odpowiednik listy następników),
  - **zbiór sąsiedztwa** (_neighborhood set_) $M$ – węzły najbliższe fizycznie (do utrzymania lokalności).
- **Routing** komunikatu z kluczem $k$:
  1. jeśli $k$ mieści się w zakresie zbioru liści → przekaż do numerycznie najbliższego liścia (koniec),
  2. w przeciwnym razie użyj tablicy routingu: węzeł, którego wspólny prefiks z $k$ jest **o co najmniej jedną cyfrę dłuższy** niż bieżący,
  3. jeśli taki wpis jest pusty → dowolny znany węzeł z prefiksem tak samo długim, ale numerycznie bliższy $k$.
- **$O(\log_{2^b} N)$ skoków** (dla $N = 10^6$, $b = 4$: ok. 5), stan $O(2^b \log_{2^b} N)$.
- **Świadomość lokalności** – ścieżki w nakładce krótkie także w sieci fizycznej.
- Zastosowania: PAST (magazyn plików), Scribe (pub-sub), SplitStream (multimedia), Squirrel (cache WWW).
- **Tapestry** (Berkeley, 2001) – podobny routing prefiksowy/sufiksowy z lokalnością, obiekty publikowane wskaźnikami (OceanStore).

---
## Kademlia (Maymounkov, Mazières – NYU, IPTPS 2002)
Najczęściej używana DHT w praktyce.
- **Identyfikatory 160-bitowe** dla węzłów i kluczy.
- **Metryka XOR**: $d(x, y) = x \oplus y$ interpretowane jako liczba.
  - $d(x, x) = 0$, $d(x, y) > 0$ dla $x \neq y$,
  - **symetryczna**: $d(x, y) = d(y, x)$ → węzły uczą się o sobie z **otrzymywanych zapytań** (każde zapytanie aktualizuje tablicę odbiorcy),
  - **nierówność trójkąta**: $d(x, z) \leq d(x, y) + d(y, z)$,
  - **jednokierunkowa**: dla danego $x$ i odległości $\Delta$ istnieje dokładnie jeden $y$ z $d(x,y) = \Delta$ → wyszukiwania tego samego klucza zbiegają się tą samą ścieżką (skuteczne cache'owanie).
  - Długość wspólnego prefiksu bitowego wyznacza „bliskość”.
- **k-kubełki** (_k-buckets_): dla każdego $0 \leq i < 160$ lista do **$k$** węzłów (typowo $k = 20$) w odległości $[2^i, 2^{i+1})$ – czyli różniących się pierwszym bitem na pozycji $i$ od końca prefiksu.
  - Uporządkowane wg czasu ostatniego kontaktu; nowy węzeł dodawany, jeśli kubełek niepełny; gdy pełny – **PING najdawniej widzianego**: jeśli odpowiada → pozostaje (nowy odrzucony), jeśli nie → zastąpiony.
  - **Preferencja dla długo działających węzłów** – węzeł działający godzinę ma większe szanse działać dalej (rozkład sesji) + odporność na ataki zalewające tablicę nowymi węzłami.
- **RPC**: `PING`, `STORE(klucz, wartość)`, `FIND_NODE(id)` (zwraca $k$ najbliższych znanych węzłów), `FIND_VALUE(klucz)` (wartość lub $k$ najbliższych węzłów).
- **Wyszukiwanie iteracyjne i równoległe**:
  1. wybierz $\alpha$ (typowo **3**) najbliższych węzłów z własnych kubełków,
  2. wyślij im **równolegle** `FIND_NODE`,
  3. z odpowiedzi wybierz $\alpha$ najbliższych jeszcze nie odpytanych i powtarzaj,
  4. zakończ, gdy $k$ najbliższych znanych węzłów zostało odpytanych i odpowiedziało.
  Równoległość omija wolne i martwe węzły bez czekania na timeouty.
- **Zapis**: `STORE` na $k$ węzłach najbliższych kluczowi; **ponowna publikacja** co godzinę (wartości wygasają po 24 h), a węzeł wysyła do nowo poznanych bliskich węzłów pary, za które są one teraz odpowiedzialne.
- $O(\log N)$ skoków, stan $O(k \log N)$.
- **Zastosowania**: **BitTorrent Mainline DHT** (miliony węzłów, wyszukiwanie peerów po infohash – [[50 Protokół BitTorrent]]), **eMule KAD**, **IPFS / libp2p**, **Ethereum** (discovery v4/v5 – odkrywanie węzłów), Storj, I2P.

---
## CAN – Content-Addressable Network (Ratnasamy i in. – Berkeley, SIGCOMM 2001)
- Przestrzeń kluczy to **$d$-wymiarowy torus** kartezjański $[0,1)^d$; klucz odwzorowany na **punkt** (d funkcji skrótu).
- Przestrzeń podzielona na **strefy** (_zones_) – hiperprostokąty, każda należy do jednego węzła, który przechowuje klucze z punktami w swojej strefie.
- **Sąsiedzi** – węzły, których strefy stykają się w $d-1$ wymiarach ($O(d)$ sąsiadów, **niezależnie od $N$**).
- **Routing zachłanny**: przekaż do sąsiada, którego strefa jest najbliżej punktu docelowego (odległość kartezjańska) → **$O(d \cdot N^{1/d})$** skoków (np. $d = \log N$ daje $O(\log N)$).
- **Dołączenie**: nowy węzeł wybiera losowy punkt, trasuje do właściciela strefy, który **dzieli strefę na pół** i przekazuje część kluczy; aktualizacja sąsiadów.
- **Odejście**: przejęcie strefy przez sąsiada (scalanie lub tymczasowa obsługa kilku stref).
- Ulepszenia: wiele przestrzeni (**realities**) dla odporności, uwzględnianie opóźnień, nakładanie stref (wielu właścicieli).

---
## Inne DHT i warianty
- **Viceroy**, **Koorde** (grafy de Bruijna – $O(\log N)$ skoków przy **stałym** stopniu $O(1)$), **Symphony** (small-world, losowe dalekie skróty),
- **Skip Graph** – obsługa **zapytań zakresowych** (DHT niszczy porządek kluczy),
- **Dynamo-style** (_zero-hop DHT_) – każdy węzeł zna **wszystkich** (członkostwo przez plotkowanie) → wyszukiwanie w 1 skoku; możliwe w centrum danych (setki–tysiące węzłów): Amazon **Dynamo**, **Cassandra**, Riak, Voldemort – [[51 Big Data, NoSQL, CAP i PACELC - uzupełnienie#Amazon Dynamo (2007)]].

## Porównanie
| System | Geometria | Metryka | Stan węzła | Skoki | Lokalność fizyczna | Zastosowania |
|---|---|---|---|---|---|---|
| **Chord** | pierścień | odległość zgodna z ruchem wskazówek | $O(\log N)$ | $O(\log N)$ | nie (w podstawowej wersji) | CFS, badania |
| **Pastry / Tapestry** | drzewo prefiksów + pierścień | prefiks + numeryczna | $O(2^b \log_{2^b} N)$ | $O(\log_{2^b} N)$ | **tak** | PAST, Scribe, OceanStore |
| **Kademlia** | drzewo binarne (XOR) | XOR | $O(k \log N)$ | $O(\log N)$ | pośrednio (RTT) | BitTorrent DHT, IPFS, Ethereum |
| **CAN** | torus $d$-wymiarowy | kartezjańska | $O(d)$ | $O(d N^{1/d})$ | opcjonalnie | badania |
| **Koorde** | graf de Bruijna | – | $O(1)$ | $O(\log N)$ | nie | teoria |
| **Dynamo (zero-hop)** | pierścień | spójne haszowanie | $O(N)$ | $O(1)$ | w centrum danych | Dynamo, Cassandra |

## Zagadnienia praktyczne
- **Churn** – ciągła naprawa tablic (stabilizacja, ponowna publikacja), replikacja danych na $r$ sąsiadach, okresowe odświeżanie; przy wysokim churnie DHT tracą efektywność.
- **Równoważenie obciążenia** – nierównomierne przedziały i popularne klucze (_hot spots_): węzły wirtualne, replikacja popularnych danych, przenoszenie węzłów.
- **Bezpieczeństwo**:
  - **atak Sybil** – jeden podmiot tworzy wiele tożsamości i przejmuje część przestrzeni (obrona: kosztowne ID – kryptograficzne zagadki, certyfikaty, ID z adresu IP),
  - **eclipse attack** – otoczenie ofiary złośliwymi węzłami w tablicy routingu (obrona: preferencja długo działających węzłów, różnorodność prefiksów/podsieci),
  - fałszywe odpowiedzi routingu i danych (obrona: niezależne ścieżki, weryfikacja skrótów treści – klucz = skrót wartości).
- **Brak złożonych zapytań** – tylko dokładny klucz; wyszukiwanie słów kluczowych przez odwrócone indeksy w DHT (kosztowne), zapytania zakresowe przez struktury z zachowaniem porządku.
- **NAT** – węzły za NAT nie mogą być w pełni uczestnikami (relay, hole punching).

## Zastosowania DHT
- **odkrywanie uczestników** – BitTorrent (peers dla infohash), Ethereum, IPFS/libp2p,
- **rozproszone magazyny danych** – Dynamo, Cassandra, Riak (w wersji zero-hop), CFS, PAST, OceanStore,
- **systemy nazw i adresowania treści** – IPFS (CID → dostawcy), I2P,
- **CDN i cache P2P** – Coral CDN, Squirrel,
- **pub-sub i multimedia** – Scribe, SplitStream,
- **komunikatory i sieci anonimowe** – Tox, Freenet (Opennet – topologia small-world).
