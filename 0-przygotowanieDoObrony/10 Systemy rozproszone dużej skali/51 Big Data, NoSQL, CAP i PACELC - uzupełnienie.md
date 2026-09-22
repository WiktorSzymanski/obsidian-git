---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 51
---
# 51. Analiza i przetwarzanie dużej ilości danych – uzupełnienie: źródła danych, NoSQL, teoria CAP, PACELC
---
> Uzupełnienie [[Systemy Rozproszone Dużej Skali/Big Data]] (definicje, 3V/5V), [[Systemy Rozproszone Dużej Skali/noSQL]], [[Systemy Rozproszone Dużej Skali/Teoria CAP]] (stub) o **źródła dużych danych**, **szczegóły modeli NoSQL**, **formalne twierdzenie CAP** i **PACELC**.

## Definicje i charakterystyka – przypomnienie i rozszerzenie
**Big Data** – zbiory danych, których **objętość, różnorodność i szybkość napływu** przekraczają możliwości tradycyjnych systemów (relacyjnych baz na pojedynczym serwerze). Wymagają nowych architektur (skalowanie poziome, przetwarzanie równoległe) i metod analizy.

Charakterystyka „V”:
- **Volume** (objętość) – terabajty, petabajty, eksabajty,
- **Velocity** (szybkość) – napływ w czasie rzeczywistym, strumienie,
- **Variety** (różnorodność) – dane **ustrukturyzowane** (tabele), **półustrukturyzowane** (JSON, XML, logi), **nieustrukturyzowane** (tekst, obrazy, wideo, dźwięk),
- **Veracity** (wiarygodność) – szum, braki, niepewność, sprzeczności,
- **Value** (wartość) – użyteczność dla decyzji,
- dodatkowe: **Variability** (zmienność znaczenia i przepływów), **Visualization**, **Validity**, **Volatility** (jak długo dane są aktualne).

## Źródła dużych danych
| Kategoria | Przykłady | Charakter |
|---|---|---|
| **generowane przez ludzi** | media społecznościowe (posty, zdjęcia, filmy), e-maile, blogi, recenzje, wyszukiwania, dokumenty | nieustrukturyzowane, duża różnorodność |
| **generowane przez maszyny** | **logi** serwerów i aplikacji, **czujniki IoT** (przemysł, smart city, pojazdy), telemetria, GPS, RFID, kamery monitoringu, sieci telekomunikacyjne (CDR) | szybki strumień, półustrukturyzowane |
| **transakcyjne / biznesowe** | transakcje bankowe i kartowe, zakupy e-commerce, ERP, CRM, systemy rezerwacji, giełda (dane tick) | ustrukturyzowane, wysoka wiarygodność |
| **clickstream i interakcje** | kliknięcia na stronach, zachowanie w aplikacjach, reklama online (bidding) | bardzo duża szybkość |
| **naukowe** | fizyka (LHC – ~90 PB/rok), astronomia (teleskopy – SKA), genomika (sekwenatory), klimatologia, symulacje | ogromna objętość |
| **dane publiczne i otwarte** | statystyki państwowe, mapy (OpenStreetMap), rejestry, Common Crawl | różne |
| **medyczne** | elektroniczna dokumentacja, obrazowanie (MRI, TK), urządzenia noszone | wrażliwe (prywatność) |

## Bazy danych NoSQL – rozszerzenie
**NoSQL** („Not only SQL”) – nierelacyjne systemy zaprojektowane pod **skalowanie poziome**, elastyczny schemat i wysoką dostępność. Zwykle rezygnują z pełnego ACID i złączeń na rzecz wydajności.

### ACID vs BASE
| ACID (relacyjne) | BASE (wiele NoSQL) |
|---|---|
| **A**tomowość, **C**spójność, **I**zolacja, **D**trwałość | **B**asically **A**vailable – system zasadniczo dostępny (także częściowo przy awariach) |
| silna spójność, transakcje | **S**oft state – stan może się zmieniać bez nowych zapisów (propagacja replik) |
| pesymistyczne, koordynacja | **E**ventual consistency – spójność ostateczna |

### Modele danych
| Model | Struktura | Operacje | Przykłady | Zastosowania |
|---|---|---|---|---|
| **klucz–wartość** | słownik: klucz → nieprzezroczysta wartość (blob) | `get/put/delete` po kluczu | Redis, **Amazon DynamoDB**, Riak, Memcached, etcd | cache, sesje, koszyki, liczniki |
| **szerokokolumnowy** (_wide-column_, rodziny kolumn) | wiersz (klucz) → rodziny kolumn → kolumny (zmienna liczba, rzadkie) z wersjami/znacznikami czasu | zakresy kluczy, zapis bardzo szybki (LSM) | **BigTable**, **HBase**, **Cassandra**, ScyllaDB | szeregi czasowe, logi, IoT, wiadomości |
| **dokumentowy** | kolekcje dokumentów JSON/BSON z zagnieżdżeniami, schemat elastyczny | zapytania po polach, indeksy wtórne, agregacje | **MongoDB**, CouchDB, Couchbase, Elasticsearch (wyszukiwanie) | katalogi produktów, CMS, profile użytkowników |
| **grafowy** | węzły i krawędzie z właściwościami | przechodzenie grafu (_traversal_), dopasowanie wzorców (Cypher, Gremlin, SPARQL) | **Neo4j**, JanusGraph, Amazon Neptune | sieci społeczne, rekomendacje, wykrywanie oszustw, grafy wiedzy |
| inne | szeregi czasowe (InfluxDB, Prometheus), wyszukiwarki (Elasticsearch), obiektowe | – | – | – |

**Szczegóły**:
- **Kolumnowe (BigTable, 2006)** – „rzadka, rozproszona, trwała, wielowymiarowa, posortowana mapa” $(wiersz: string, kolumna: string, czas: int64) \to string$; wiersze posortowane i dzielone zakresami na **tablety** (serwery tabletów); zapis: **commit log** + **memtable** (RAM) → okresowo **SSTable** (niezmienne pliki na GFS) + **kompakcja** (struktura **LSM-tree**), filtry Blooma przy odczycie; koordynacja przez **Chubby** (usługa blokad Paxos). Uzupełnienie [[Systemy Rozproszone Dużej Skali/BigTable]].
- **Dokumentowe** – dokument jako jednostka atomowości i lokalności (denormalizacja zamiast złączeń), MongoDB: replikacja przez **replica set** (primary + secondary, elekcja Raft-podobna), **sharding** po kluczu (zakresowy/haszujący), `writeConcern`/`readConcern` do sterowania spójnością.
- **Grafowe** – **index-free adjacency** (węzeł bezpośrednio wskazuje sąsiadów) → przechodzenie grafu w czasie zależnym od odwiedzonej części, nie od rozmiaru bazy; trudniejsze skalowanie poziome.

### Amazon Dynamo (2007)
Wysoka dostępność zapisów („koszyk zawsze przyjmuje dodanie”), architektura bez mistrza. Uzupełnienie [[Systemy Rozproszone Dużej Skali/Dynamo]].
- **Spójne haszowanie** z **węzłami wirtualnymi** – partycjonowanie ([[49 Ustrukturyzowane systemy P2P i DHT#Spójne haszowanie (_consistent hashing_, Karger i in. 1997)]]),
- **replikacja** na $N$ kolejnych węzłach pierścienia (**lista preferencji**),
- **kworum konfigurowalne**: $R$ (odczyt), $W$ (zapis); $R + W > N$ → nakładanie się kworów (np. $N=3, R=2, W=2$),
- **„niechlujne kworum”** (_sloppy quorum_) i **wskazówki przekazania** (_hinted handoff_) – przy awarii zapis na następnym zdrowym węźle z adnotacją, później oddany,
- **wersjonowanie zegarami wektorowymi** – wykrywanie współbieżnych wersji, zwracanych klientowi do **scalenia** (_semantic reconciliation_),
- **read repair** – naprawa nieaktualnych replik przy odczycie,
- **anty-entropia** z **drzewami Merkle** – porównanie zakresów kluczy i synchronizacja tylko różnic,
- **członkostwo i wykrywanie awarii przez plotkowanie** ([[Algorytmy Rozproszone/Gossiping]]).
- Następcy: Cassandra (model danych BigTable + dystrybucja Dynamo), Riak, Voldemort, DynamoDB (usługa – inna architektura).

### Modele dystrybucji danych
- **Sharding** (partycjonowanie poziome) – uzupełnienie [[Sharding]]:
  - **zakresowy** (_range_) – zakresy kluczy na shardach; ✔ zapytania zakresowe, ✘ gorące zakresy (np. klucze czasowe),
  - **haszujący** (_hash_) – $shard = hash(klucz) \bmod N$ lub spójne haszowanie; ✔ równomierny rozkład, ✘ brak zakresów,
  - **katalogowy** (_directory-based_) – tablica przypisań klucz → shard (elastyczny, SPoF katalogu),
  - **geograficzny** – wg regionu użytkownika,
  - **rebalansowanie** przy dodaniu węzłów, dzielenie shardów, problem **gorących kluczy** (celebryci), zapytania rozgłaszane do wszystkich shardów (_scatter-gather_), transakcje między shardami.
- **Replikacja**: master-slave (primary-backup) / multi-master / bez mistrza z kworum – [[Algorytmy Rozproszone/Replikacja]]; synchroniczna vs asynchroniczna.
- **Kombinacja**: każdy shard replikowany.

### NewSQL
Relacyjny model i **transakcje ACID** + skalowanie poziome NoSQL: **Google Spanner** (globalne transakcje, **TrueTime** – zegary atomowe/GPS z ograniczoną niepewnością, Paxos na shard), **CockroachDB** (Raft, HLC), **TiDB**, **YugabyteDB**, **VoltDB** (w pamięci, jednowątkowe partycje) – [[Systemy Rozproszone Dużej Skali/newSQL]].

## Teoria CAP
### Hipoteza i twierdzenie
- **Eric Brewer** (PODC 2000, keynote) – hipoteza: rozproszony system współdzielonych danych może spełniać co najwyżej **dwie z trzech** własności.
- **Gilbert i Lynch** (2002) – formalne **twierdzenie**:
  > W **asynchronicznym** modelu sieci **nie da się** zaimplementować obiektu odczytu/zapisu, który gwarantuje jednocześnie **dostępność** i **spójność atomową** we **wszystkich** wykonaniach, także takich, w których komunikaty są **gubione** (partycje).

### Definicje (Gilbert–Lynch)
- **C – spójność** (_Consistency_) = **spójność atomowa / linearyzowalność**: każda operacja wygląda, jakby wykonała się natychmiastowo na jednej kopii; odczyt zwraca wynik najnowszego zapisu ([[Linearizability]]). **Nie** to samo co „C” w ACID (integralność).
- **A – dostępność** (_Availability_): **każde** żądanie otrzymane przez **niezawodzący** węzeł musi otrzymać **odpowiedź** (w skończonym czasie) – nie błąd ani oczekiwanie w nieskończoność.
- **P – tolerancja podziału** (_Partition tolerance_): system działa mimo **dowolnej utraty komunikatów** między węzłami (podział sieci na części, które się nie komunikują).

### Szkic dowodu
1. Dwa węzły $G_1$, $G_2$ z repliką $v = v_0$; **podział sieci** – żaden komunikat między nimi nie dociera.
2. Klient zapisuje $v_1$ do $G_1$ → z **dostępności** $G_1$ musi potwierdzić zapis.
3. Inny klient czyta $v$ z $G_2$ → z **dostępności** $G_2$ musi odpowiedzieć; nie otrzymał informacji o zapisie, więc zwraca $v_0$.
4. Odczyt po potwierdzonym zapisie zwraca starą wartość → **brak linearyzowalności**. Sprzeczność.
5. W systemie asynchronicznym $G_2$ nie odróżni podziału od opóźnienia → nie może bezpiecznie czekać.

### Właściwa interpretacja
- W systemie rozproszonym **podziałów sieci nie da się wykluczyć** (awarie łączy, przełączników, konfiguracji, długie pauzy GC). **P nie jest opcją** – wybór dotyczy tego, **co robić w czasie podziału**:
  - **CP** – zachować spójność: część węzłów (mniejszość) **odmawia** obsługi (błąd/timeout) → brak dostępności,
  - **AP** – zachować dostępność: wszystkie strony obsługują żądania → **rozbieżność** replik, później uzgadnianie (spójność ostateczna).
- „**CA**” oznacza system, który nie toleruje podziałów (pojedynczy węzeł, klaster w jednej szafie z założeniem niezawodnej sieci).
- Brewer (2012, „CAP Twelve Years Later”): „2 z 3” jest mylące – wybór jest **ciągły i lokalny** (per operacja, per dane), a **podziały są rzadkie**; w normalnej pracy można mieć C i A. Ważne jest **wykrywanie podziału**, tryb pracy w podziale i **odtwarzanie** po nim (np. CRDT – [[46 Ostateczna spójność - CRDT i typy chmurowe]]).
- CAP nie mówi nic o **opóźnieniach** w normalnej pracy – to uzupełnia PACELC.

### Klasyfikacja przykładowa (domyślne konfiguracje)
| Typ | Systemy |
|---|---|
| **CP** | HBase, BigTable, MongoDB (zapisy/odczyty majority), ZooKeeper, etcd, Consul, Spanner, CockroachDB, Redis Cluster (częściowo), bazy z konsensusem (Raft/Paxos) |
| **AP** | Cassandra, Riak, Dynamo, CouchDB, Voldemort, DNS, systemy z CRDT |

Wiele systemów pozwala **dostrajać** wybór (kworum, poziomy spójności per zapytanie – Cassandra `ONE`/`QUORUM`/`ALL`).

## PACELC
**Daniel Abadi** (2010 – blog, 2012 – IEEE Computer, „Consistency Tradeoffs in Modern Distributed Database System Design”):
> **Jeśli** występuje podział (**P**), system wybiera między dostępnością (**A**) a spójnością (**C**); **w przeciwnym razie** (**E**lse), przy normalnej pracy, wybiera między **opóźnieniem** (**L**atency) a **spójnością** (**C**).

$$\text{if } P \text{ then } (A \text{ or } C) \text{ else } (L \text{ or } C)$$

**Uzasadnienie „E: L vs C”**: replikacja wymaga decyzji, jak aktualizować repliki. **Synchroniczna** (czekanie na potwierdzenie większości / wszystkich replik, lub jeden mistrz obsługujący zapisy i silne odczyty) daje **spójność**, ale **zwiększa opóźnienie** (szczególnie przy replikach w odległych regionach). **Asynchroniczna** daje **niskie opóźnienie**, ale odczyty z replik mogą być nieaktualne. Ten kompromis występuje **zawsze**, a nie tylko w rzadkich chwilach podziału – dlatego w praktyce jest ważniejszy.

### Klasyfikacja (Abadi i późniejsze)
| Klasa | W podziale | Normalnie | Systemy |
|---|---|---|---|
| **PA/EL** | dostępność | niskie opóźnienie | **Dynamo**, **Cassandra**, **Riak**, Voldemort (domyślne ustawienia) |
| **PC/EC** | spójność | spójność | **BigTable/HBase**, VoltDB/H-Store, Megastore, **Spanner**, systemy ACID z replikacją synchroniczną, CockroachDB |
| **PA/EC** | dostępność | spójność | **MongoDB** (klasyfikacja Abadiego dla starszych wersji: w normalnej pracy wszystkie odczyty/zapisy przez primary; przy podziale primary w mniejszości mógł przyjmować zapisy, które są potem wycofywane), GigaSpaces |
| **PC/EL** | spójność | niskie opóźnienie | **Yahoo PNUTS** (asynchroniczna replikacja dla odczytów – „timeline consistency”; przy podziale mistrz rekordu niedostępny dla zapisu) |

Dostrajanie: Cassandra/DynamoDB z odczytem `QUORUM` / `ConsistentRead=true` przesuwają się ku EC kosztem opóźnienia; Spanner z odczytami _stale_ (bounded staleness) ku EL.

### CAP vs PACELC
| | CAP | PACELC |
|---|---|---|
| Zakres | tylko sytuacja podziału sieci | podział **i** normalna praca |
| Kompromis | C vs A | C vs A (przy P), **C vs L** (normalnie) |
| Formalne | twierdzenie (Gilbert-Lynch) | ramy klasyfikacji (nie twierdzenie) |
| Praktyczne znaczenie | rzadkie zdarzenia | codzienny wybór projektowy |

## Zobacz też
- [[52 Architektura systemów Big Data - narzędzia]], [[53 Przetwarzanie dużych danych - Hadoop i Apache Spark]]
- [[Systemy Rozproszone Dużej Skali/Cassandra]], [[Systemy Rozproszone Dużej Skali/Schemaless]]
