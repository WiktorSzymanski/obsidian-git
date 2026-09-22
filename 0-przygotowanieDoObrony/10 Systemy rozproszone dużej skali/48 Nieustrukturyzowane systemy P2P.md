---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 48
---
# 48. Nieustrukturyzowane systemy P2P – strategie wyszukiwania zasobów, algorytmy, przykłady systemów
---
> **System peer-to-peer (P2P)** to system rozproszony, w którym **równorzędne węzły** (_peers_, _servents_ = server + client) współdzielą zasoby (pliki, pasmo, moc obliczeniową) **bezpośrednio**, bez centralnego serwera. Węzły tworzą **sieć nakładkową** (_overlay_) nad siecią fizyczną. W sieciach **nieustrukturyzowanych** połączenia między węzłami są **dowolne** (zwykle losowe), a **położenie zasobu nie jest związane z topologią** – każdy węzeł przechowuje, co chce, i trzeba go **wyszukać**.

## Cechy P2P
- **autonomia** węzłów, brak centralnej kontroli (lub minimalna),
- **symetria ról** – każdy węzeł może pobierać i udostępniać,
- **samoorganizacja** – dołączanie i odchodzenie bez administracji,
- **skalowalność** – wraz z liczbą użytkowników rośnie liczba zasobów,
- **churn** – wysoka dynamika: węzły często dołączają i odchodzą (średnia sesja: minuty–godziny),
- **heterogeniczność** – różne przepustowości, moc, dostępność,
- wiele węzłów za **NAT/firewallem** (niedostępne z zewnątrz).

### Nieustrukturyzowane vs ustrukturyzowane
| | Nieustrukturyzowane | Ustrukturyzowane (DHT) |
|---|---|---|
| Topologia | dowolna, losowa | ściśle określona (pierścień, drzewo, hiperkostka) |
| Położenie danych | dowolne | wyznaczone funkcją skrótu klucza |
| Wyszukiwanie | zalewanie, spacery losowe – **zapytania złożone** (słowa kluczowe, wzorce, zakresy) | **dokładne dopasowanie klucza** w $O(\log N)$ |
| Gwarancja znalezienia | brak (rzadkie zasoby mogą nie zostać znalezione) | tak (jeśli zasób istnieje) |
| Koszt utrzymania | niski, odporne na churn | wyższy (tablice routingu) |
| Przykłady | Gnutella, KaZaA, Freenet | Chord, Pastry, Kademlia – [[49 Ustrukturyzowane systemy P2P i DHT]] |

## Architektury systemów nieustrukturyzowanych
### 1. Scentralizowany indeks – Napster (1999)
- **Centralny serwer katalogowy** przechowuje indeks: jakie pliki (MP3) ma który użytkownik.
- Klient po zalogowaniu wysyła listę swoich plików; zapytanie trafia do serwera, który zwraca listę węzłów z plikiem.
- **Pobieranie bezpośrednio P2P** między użytkownikami.
- ✔ proste, szybkie i kompletne wyszukiwanie, $O(1)$ komunikatów na zapytanie; ✘ **SPoF**, koszt i skalowalność serwera, **odpowiedzialność prawna** – zamknięty sądownie w 2001 r.

### 2. Zdecentralizowany – Gnutella 0.4 (2000)
Wszystkie węzły równorzędne, brak serwera.

**Dołączenie**: węzeł musi znać adres co najmniej jednego aktywnego węzła (_bootstrap_ – listy na stronach WWW, GWebCache, pamięć podręczna z poprzednich sesji), łączy się z nim (TCP) i odkrywa kolejnych sąsiadów.

**Deskryptory (komunikaty)** – każdy z **identyfikatorem** (GUID), **TTL** i licznikiem **Hops**:
| Komunikat | Rola |
|---|---|
| **Ping** | odkrywanie węzłów – rozsyłany zalewaniem |
| **Pong** | odpowiedź: adres IP, port, liczba i rozmiar udostępnianych plików |
| **Query** | zapytanie wyszukiwania (słowa kluczowe, min. prędkość) – zalewanie |
| **QueryHit** | odpowiedź: adres, port, lista pasujących plików, identyfikator węzła |
| **Push** | prośba do węzła za firewallem, by sam połączył się z pytającym |

**Algorytm routingu**:
- węzeł po otrzymaniu Ping/Query: jeśli GUID już widział → **odrzuca** (duplikat); w przeciwnym razie zapamiętuje, z którego połączenia przyszedł, zmniejsza **TTL**, zwiększa **Hops** i przekazuje **wszystkim sąsiadom oprócz nadawcy** (jeśli TTL > 0),
- **Pong/QueryHit** wracają **ścieżką odwrotną** (_reverse path routing_) – każdy węzeł przekazuje odpowiedź do połączenia, z którego przyszło zapytanie o tym GUID,
- początkowe TTL zwykle **7** → zasięg ograniczony „horyzontem”,
- **pobieranie**: bezpośrednie połączenie HTTP (`GET /get/<index>/<nazwa>`).

**Problemy**: zalewanie generuje ogromny ruch (liczba komunikatów rośnie wykładniczo z TTL), węzły wolne (modemy) stają się wąskim gardłem, zasoby poza horyzontem niewidoczne, brak gwarancji znalezienia rzadkich plików, **free-riding** (większość użytkowników nic nie udostępnia), zatruwanie wyników.

### 3. Hybrydowy / hierarchiczny – superwęzły
**KaZaA / FastTrack** (2001), **Gnutella 0.6** (ultrapeers), Skype (dawniej):
- węzły dzielą się na **superwęzły** (_supernodes_, _ultrapeers_ – duża przepustowość, publiczny adres, długi czas działania) i **zwykłe węzły / liście** (_leaves_),
- liść łączy się z kilkoma superwęzłami i wysyła im **indeks swoich plików**,
- superwęzły tworzą **nieustrukturyzowaną sieć** między sobą i obsługują zapytania w imieniu liści (zalewanie tylko wśród superwęzłów),
- ✔ łączy efektywność indeksu z decentralizacją, odporność na słabe węzły; ✘ superwęzły obciążone, częściowa centralizacja.

**Gnutella 0.6 – QRP** (_Query Routing Protocol_): liść wysyła ultrapeerowi **tablicę skrótów słów kluczowych** swoich plików (podobną do filtra Blooma); ultrapeer przekazuje zapytanie liściowi tylko, jeśli słowa kluczowe mogą pasować → znacząco mniej ruchu. Dodatkowo **dynamiczne zapytania** (stopniowe zwiększanie zasięgu, aż do uzyskania wystarczającej liczby wyników).

### 4. Freenet (Clarke, 2000)
- cel: **anonimowość** i odporność na cenzurę; pliki identyfikowane **kluczami** (skróty),
- routing **w głąb z sterowaniem** (_steepest-ascent hill-climbing_): zapytanie przekazywane do sąsiada, którego klucze w pamięci są **najbliższe** szukanemu; przy porażce – wycofanie i następny sąsiad; TTL,
- odpowiedź wraca ścieżką odwrotną, a węzły po drodze **buforują plik** → popularne dane replikują się wzdłuż ścieżek, a sieć z czasem **uczy się** grupować podobne klucze (_small-world_),
- anonimowość: węzeł nie wie, czy sąsiad jest źródłem, czy pośrednikiem.

## Strategie wyszukiwania zasobów
### 1. Zalewanie (_flooding_, BFS z TTL)
- Zapytanie przekazywane **wszystkim** sąsiadom aż do wyczerpania TTL; duplikaty eliminowane po GUID.
- Liczba komunikatów przy średnim stopniu $d$ i TTL $t$: $\approx \sum_{i=1}^{t} d(d-1)^{i-1}$ – **wykładniczo**.
- ✔ duży zasięg, znajduje także rzadkie zasoby w horyzoncie, najkrótsza ścieżka; ✘ ogromny narzut, zła skalowalność, duplikaty.

### 2. Rozszerzający się pierścień (_expanding ring_, iteracyjne pogłębianie)
- Seria zalewań z **rosnącym TTL** (np. 1, 2, 4…) – kolejne tylko, jeśli poprzednie nie dało wystarczających wyników.
- ✔ popularne zasoby znajdowane tanio blisko; ✘ dla rzadkich zasobów powtórne przeszukiwanie tych samych węzłów; opóźnienie.
- Wariant: **iterative deepening** (Yang, Garcia-Molina) z zamrażaniem zapytań w węzłach na granicy poprzedniego zasięgu.

### 3. Spacery losowe (_random walks_, k-walkers)
- Zapytanie przekazywane **jednemu losowemu** sąsiadowi; **$k$ niezależnych spacerowiczów** równolegle (np. $k = 16$–$32$).
- **Zakończenie**: TTL albo **okresowe sprawdzanie** z węzłem pytającym (_check-back_), czy znaleziono już wystarczająco wyników.
- Liczba komunikatów rośnie **liniowo** z długością spacerów: $k \cdot TTL$.
- ✔ o rząd wielkości mniej komunikatów niż zalewanie przy podobnej skuteczności dla replikowanych zasobów (Lv, Cao, Cohen, Li, Shenker 2002); ✘ większe opóźnienie, gorsze dla rzadkich zasobów, wynik zależy od losowości.
- Warianty: spacery **ważone stopniem** węzła (węzły o dużym stopniu widzą więcej – sieci _power-law_), unikanie ponownych odwiedzin.

### 4. Wyszukiwanie ukierunkowane / inteligentne
- **Directed BFS** – przekazanie tylko do podzbioru sąsiadów, wybranego na podstawie historii: kto zwracał najwięcej wyników, miał najmniejsze opóźnienie, najwięcej plików, najdłużej działa.
- **Adaptive Probabilistic Search (APS)** – prawdopodobieństwo wyboru sąsiada aktualizowane na podstawie sukcesów spacerów.
- **Interest-based shortcuts** – skróty do węzłów, które wcześniej odpowiadały na podobne zapytania (grupy zainteresowań).

### 5. Indeksy lokalne i routingowe
- **Indeksy lokalne** (_local indices_) – każdy węzeł utrzymuje indeks zasobów węzłów w promieniu $r$ skoków; zapytanie przetwarzane tylko na wybranych głębokościach (policy), resztę pokrywają indeksy.
- **Indeksy routingowe** (_routing indices_, Crespo, Garcia-Molina) – dla każdego sąsiada liczba dokumentów w danej kategorii osiągalnych przez niego; zapytanie kierowane do najbardziej obiecującego sąsiada.
- **Filtry Blooma** – zwarte, probabilistyczne podsumowania zbiorów słów kluczowych (fałszywie pozytywne, brak fałszywie negatywnych) – wymiana między sąsiadami/ultrapeerami (QRP).

### 6. Wykorzystanie superwęzłów i hierarchii
Zapytania obsługiwane przez podsieć superwęzłów z pełnymi indeksami liści (KaZaA, Gnutella 0.6).

## Replikacja zasobów a skuteczność wyszukiwania
Skuteczność wyszukiwania zależy od **liczby kopii** zasobu. Strategie rozmieszczenia $r_i$ kopii obiektu $i$ o popularności (częstości zapytań) $q_i$ przy stałej łącznej pojemności (Cohen, Shenker 2002):
- **jednolita** – każdy obiekt tyle samo kopii,
- **proporcjonalna** – $r_i \propto q_i$ (naturalny efekt pobierania: kto pobrał, ten udostępnia),
- **pierwiastkowa** – $r_i \propto \sqrt{q_i}$ – **minimalizuje oczekiwaną długość wyszukiwania** przy spacerach losowych (optimum dla udanych wyszukiwań); jednolita i proporcjonalna dają tę samą, gorszą średnią.

Realizacja pierwiastkowej: **replikacja ścieżkowa** (kopie w węzłach na ścieżce spaceru) – liczba kopii zbliża się do $\sqrt{q_i}$.

## Porównanie strategii
| Strategia | Komunikaty | Opóźnienie | Rzadkie zasoby | Popularne zasoby |
|---|---|---|---|---|
| Zalewanie | bardzo dużo (wykładniczo) | niskie | dobre (w horyzoncie) | nadmiarowe |
| Expanding ring | umiarkowane | średnie | powtórzenia | bardzo tanio |
| Spacery losowe (k-walkers) | mało (liniowo) | wyższe | słabe | dobre |
| Directed / APS | mało | średnie | zależne od historii | dobre |
| Indeksy lokalne / routingowe | mało przy wyszukiwaniu, koszt aktualizacji indeksów | niskie | średnie | dobre |
| Superwęzły | mało | niskie | średnie | dobre |

## Zalety i wady P2P nieustrukturyzowanych
✔ proste i odporne na **churn** (brak ścisłej struktury do naprawy), obsługa **złożonych zapytań** (słowa kluczowe, metadane), brak SPoF (wersje zdecentralizowane), niski koszt dołączenia.
✘ brak gwarancji znalezienia zasobu, słaba skalowalność zalewania, duży ruch sygnalizacyjny, **free-riding**, zatruwanie treści i złośliwe węzły (fałszywe QueryHit, wirusy), ataki **Sybil**, problemy prawne, NAT.

## Przykłady systemów
| System | Rok | Typ | Wyszukiwanie |
|---|---|---|---|
| Napster | 1999 | scentralizowany indeks | serwer |
| Gnutella 0.4 | 2000 | czysty P2P | zalewanie TTL |
| Freenet | 2000 | P2P, anonimowy | routing w głąb po kluczach |
| KaZaA / FastTrack | 2001 | hybrydowy (superwęzły) | zalewanie wśród superwęzłów |
| Gnutella 0.6 (LimeWire) | 2002 | hierarchiczny (ultrapeers) | QRP + dynamiczne zapytania |
| eDonkey / eMule | 2000/2002 | serwery indeksów + (później) Kademlia | serwer / DHT |
| BitTorrent | 2001 | dystrybucja plików (wyszukiwanie poza protokołem) | tracker / DHT – [[50 Protokół BitTorrent]] |
| Skype (do 2012) | 2003 | superwęzły | – |
