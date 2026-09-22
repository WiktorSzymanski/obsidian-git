---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 47
---
# 47. Architektury systemów rozproszonych dużej skali: klasyfikacje, idea działania, zastosowania, przykłady implementacji
---
> **System rozproszony dużej skali** obejmuje od setek do milionów węzłów, często **geograficznie rozproszonych**, należących do **wielu domen administracyjnych** i działających w warunkach ciągłych zmian (**churn**, awarie). Wyzwaniami są **skalowalność** (rozmiar, geografia, administracja), **heterogeniczność**, **brak globalnej wiedzy i kontroli** oraz **awarie jako norma**, a nie wyjątek.

Aspekty projektowe: [[07 Aspekty projektowe realizacji systemów rozproszonych]].

## Klasyfikacje
### Wg celu (rodzaju współdzielonych zasobów)
| Klasa | Współdzielony zasób | Przykłady |
|---|---|---|
| **systemy obliczeniowe** (HPC) | moc obliczeniowa | klastry, superkomputery, gridy, obliczenia ochotnicze |
| **systemy informacyjne / transakcyjne** | dane i usługi biznesowe | systemy bankowe, e-commerce, SOA/mikroserwisy |
| **systemy danych** | przechowywanie i przetwarzanie danych | rozproszone bazy NoSQL, Big Data, systemy plików |
| **dystrybucja treści** | pasmo i pamięć podręczna | CDN, P2P (BitTorrent), streaming |
| **komunikacja** | usługi komunikacyjne | VoIP, komunikatory, systemy pub-sub |
| **wszechobecne (pervasive)** | czujniki, urządzenia | IoT, sieci czujnikowe, systemy mobilne, smart home |

### Wg organizacji (architektury)
| Architektura | Idea | Zalety | Wady |
|---|---|---|---|
| **scentralizowana: klient–serwer / wielowarstwowa** | serwery udostępniają usługi, klienci korzystają; skalowanie przez farmy serwerów, load balancery, replikację | prostota, kontrola, spójność | wąskie gardło, SPoF, koszt |
| **zdecentralizowana: P2P** | węzły równorzędne (_servent_) pełnią obie role, samoorganizacja w sieć nakładkową | skalowalność z liczbą uczestników, brak SPoF, niski koszt | brak kontroli, bezpieczeństwo, churn |
| **hybrydowa** | część scentralizowana (indeks, tracker, superwęzły) + P2P | kompromis | częściowy SPoF |
| **klastry** | ściśle powiązane, jednorodne węzły w jednej sieci LAN, jedna administracja | wydajność, łatwe zarządzanie | ograniczone do jednego ośrodka |
| **grid** | federacja zasobów wielu organizacji (wirtualne organizacje) | ogromne zasoby naukowe | heterogeniczność, złożony middleware |
| **chmura** | skonsolidowane centra danych dostawcy, zasoby na żądanie (wirtualizacja) | elastyczność, ekonomia skali | uzależnienie, prywatność |
| **edge / fog** | przetwarzanie **blisko źródła danych** (brzeg sieci, bramy, stacje bazowe) | niskie opóźnienia, mniej transferu, prywatność | zarządzanie wieloma małymi ośrodkami |
| **sterowana zdarzeniami / pub-sub** | producenci i konsumenci zdarzeń rozprzężeni przez brokera/log | luźne powiązanie, skalowalność | złożone śledzenie, spójność ostateczna |

### Wg powiązania i jednorodności
- **ściśle powiązane** (_tightly coupled_ – multiprocesory, klastry HPC z szybką siecią InfiniBand) vs **luźno powiązane** (gridy, P2P, Internet),
- **jednorodne** vs **heterogeniczne**,
- wg **topologii nakładkowej**: nieustrukturyzowane / ustrukturyzowane (DHT) / hierarchiczne.

---
## Klastry
**Idea**: zbiór niezależnych komputerów (węzłów) połączonych szybką siecią, pracujących jako jeden zasób.
- **HPC** (obliczenia naukowe): MPI, planista zadań (**Slurm**, PBS), równoległy system plików (Lustre, GPFS), sieć InfiniBand; lista TOP500 (Frontier, Aurora).
- **HA** (_high availability_): Pacemaker/Corosync, failover, współdzielona pamięć (DRBD, SAN – [[36 Systemy składowania danych - iSCSI, multipath, OCFS, DRBD]]).
- **Klastry równoważenia obciążenia**: farmy serwerów WWW za load balancerem (LVS, HAProxy).
- **Klastry kontenerowe**: [[Zarządzanie Systemami Rozproszonymi/Kubernetes]], Google **Borg** (poprzednik Kubernetes), Apache Mesos.
- **Hurtownie obliczeniowe** (_warehouse-scale computers_, Barroso-Hölzle) – centra danych Google/Amazon jako jeden komputer o dziesiątkach tysięcy serwerów.

## Gridy
**Idea** (Foster: „grid to system, który koordynuje zasoby **niepodlegające centralnej kontroli**, używając **standardowych, otwartych protokołów**, by dostarczać **nietrywialną jakość usług**”). Zasoby wielu instytucji łączone w **wirtualne organizacje**.
- **Warstwy architektury**: fabric (zasoby lokalne) → connectivity (komunikacja, bezpieczeństwo GSI/X.509) → resource (zarządzanie pojedynczym zasobem – GRAM, GridFTP) → collective (katalogi, brokering, planowanie) → application.
- **Middleware**: **Globus Toolkit**, gLite (EGEE), UNICORE, ARC; **HTCondor** (obliczenia wysokoprzepustowe, cycle scavenging).
- **Zastosowania**: fizyka wysokich energii – **WLCG** (Worldwide LHC Computing Grid: warstwy Tier-0 CERN → Tier-1 → Tier-2), bioinformatyka, klimatologia; w Polsce PL-Grid.
- **Obliczenia ochotnicze** (_volunteer computing_): **BOINC** (SETI@home, Einstein@home, Rosetta@home), **Folding@home** – serwer rozdziela jednostki pracy na komputery ochotników, wyniki weryfikowane redundancją (kilka niezależnych obliczeń).
- Porównanie z chmurą: [[35 Przetwarzanie w chmurze - modele, grid, skalowanie, standaryzacja, bezpieczeństwo#Chmura a grid]].

## Systemy peer-to-peer
**Idea**: każdy węzeł jest jednocześnie klientem i serwerem; zasoby (pliki, pasmo, CPU) dostarczają uczestnicy; węzły tworzą **sieć nakładkową** (_overlay_) nad Internetem.
- **Nieustrukturyzowane** (Napster, Gnutella, KaZaA) – losowy graf, wyszukiwanie zalewaniem/spacerami – [[48 Nieustrukturyzowane systemy P2P]].
- **Ustrukturyzowane** (Chord, Pastry, Kademlia, CAN) – DHT, deterministyczny routing $O(\log N)$ – [[49 Ustrukturyzowane systemy P2P i DHT]].
- **Dystrybucja plików**: BitTorrent – [[50 Protokół BitTorrent]].
- **Inne**: **IPFS** (adresowanie treścią, Kademlia), **blockchain/kryptowaluty** – [[54 Protokół Blockchain]], Tor (anonimizacja, onion routing), Skype (dawniej superwęzły), WebRTC, streaming P2P (PPLive), sieci CDN hybrydowe (Akamai NetSession).

## Systemy chmurowe i centra danych
**Idea**: skonsolidowana infrastruktura z wirtualizacją, udostępniana usługowo (IaaS/PaaS/SaaS) – [[35 Przetwarzanie w chmurze - modele, grid, skalowanie, standaryzacja, bezpieczeństwo]].

**Wzorce architektoniczne dużej skali**:
- **mikroserwisy** – małe, niezależnie wdrażane usługi komunikujące się przez API/komunikaty; service mesh (Istio), brama API ([[Microservices]]),
- **podział danych** (_sharding_) i **replikacja** – [[51 Big Data, NoSQL, CAP i PACELC - uzupełnienie]],
- **pamięci podręczne** (Memcached, Redis), **CDN** (Akamai, Cloudflare – serwery brzegowe z buforowaniem, anycast/DNS geolokalizacja),
- **kolejki i logi zdarzeń** (Kafka), **CQRS/Event Sourcing** (kontekst pracy magisterskiej – [[CQRS]], [[Event Sourcing]]),
- **bezstanowe usługi + zewnętrzny stan**, **autoskalowanie**, **wiele regionów** i stref dostępności.

**Systemy Google jako wzorcowe przykłady**: GFS → Colossus (pliki), MapReduce (przetwarzanie), BigTable (baza kolumnowa), Chubby (blokady, Paxos), Spanner (globalnie rozproszona baza z TrueTime), Borg (zarządzanie klastrem).
**Amazon**: Dynamo (klucz-wartość, spójność ostateczna), S3, SQS, EC2.
**Facebook/Meta**: Cassandra (pierwotnie), TAO (graf społecznościowy), Memcache na skalę, Haystack (zdjęcia).

## Systemy Big Data
**Idea**: przetwarzanie danych o dużej objętości, zmienności i szybkości napływu na klastrach z tanich maszyn – przenoszenie obliczeń do danych.
- Hadoop (HDFS + MapReduce + YARN), Spark – [[53 Przetwarzanie dużych danych - Hadoop i Apache Spark]],
- architektury Lambda/Kappa – [[Systemy Rozproszone Dużej Skali/Architektura Lambda]], [[Systemy Rozproszone Dużej Skali/Architektura Kappa]], [[52 Architektura systemów Big Data - narzędzia]].

## Edge / fog computing i IoT
**Idea**: miliardy urządzeń generują dane, a przesyłanie wszystkiego do chmury jest kosztowne i wolne → przetwarzanie **warstwowe**: urządzenia (czujniki) → **brzeg** (bramy, mikro-centra danych przy stacjach 5G – MEC) → **mgła** (węzły pośrednie) → **chmura** (analiza globalna, trenowanie modeli).
- **Zastosowania**: pojazdy autonomiczne, przemysł 4.0, inteligentne miasta, AR/VR, monitoring wideo z analizą lokalną.
- **Technologie**: MQTT, AWS IoT Greengrass, Azure IoT Edge, K3s/KubeEdge, federated learning (uczenie bez przesyłania surowych danych).
- **Wyzwania**: ograniczone zasoby, zawodna łączność, bezpieczeństwo fizyczne urządzeń, aktualizacje na dużą skalę.

## Systemy mobilne i sieci ad hoc
Węzły ruchome, łączność przerywana, ograniczona energia; sieci **MANET**, **DTN** (_delay-tolerant networks_ – store-and-forward), sieci czujnikowe (TinyOS, protokoły energooszczędne, agregacja danych w sieci).

## Zestawienie
| Klasa | Skala | Kontrola | Spójność typowa | Przykłady implementacji |
|---|---|---|---|---|
| Klaster HPC | 10²–10⁵ węzłów | jedna organizacja | silna | Slurm+MPI, Lustre |
| Grid | 10⁴–10⁶ rdzeni | federacja | zależna od zadań | Globus, WLCG, BOINC |
| Chmura | 10⁵–10⁶ serwerów | dostawca | silna lub ostateczna | AWS, Azure, GCP, OpenStack |
| P2P | 10⁶–10⁷ węzłów | brak | ostateczna / brak | BitTorrent, Kademlia, IPFS |
| Big Data | 10³–10⁴ węzłów | organizacja | zależna | Hadoop, Spark, Kafka |
| Edge/IoT | 10⁹ urządzeń | wiele | ostateczna | Greengrass, KubeEdge |
| Blockchain | 10⁴ węzłów pełnych | brak (konsensus) | probabilistyczna | Bitcoin, Ethereum |

## Wspólne techniki skalowania
- **podział** (dane – sharding/DHT, funkcje – mikroserwisy, geografia – regiony),
- **replikacja i buforowanie** (kompromis CAP/PACELC),
- **asynchroniczność** i kolejki, przetwarzanie wsadowe i strumieniowe,
- **decentralizacja** (brak globalnego stanu, plotkowanie – [[Algorytmy Rozproszone/Gossiping]], lokalne decyzje),
- **projektowanie pod awarie** (redundancja, detektory awarii, automatyczne odtwarzanie, _chaos engineering_),
- **spójne haszowanie** (równomierny podział i minimalne przemieszczenia przy zmianie liczby węzłów).
