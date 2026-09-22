---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 52
---
# 52. Architektura systemów Big Data – pozyskiwanie, składowanie, przetwarzanie (uzupełnienie o technologie)
---
> Uzupełnienie [[Systemy Rozproszone Dużej Skali/Big Data#Architektura]] (warstwy logiczne), [[Systemy Rozproszone Dużej Skali/Architektura Lambda]] i [[Systemy Rozproszone Dużej Skali/Architektura Kappa]] o **konkretne technologie i zasady** w każdej warstwie potoku danych.

## Potok danych – przegląd
```
 ŹRÓDŁA ─▶ POZYSKIWANIE ─▶ SKŁADOWANIE ─▶ PRZETWARZANIE ─▶ SERWOWANIE / ANALIZA ─▶ UŻYTKOWNICY
 (logi,     (wsadowe,       (data lake,     (wsadowe,         (hurtownie, bazy NoSQL,  (BI, ML,
  IoT, BD,   strumieniowe,   HDFS, S3,       strumieniowe,     wyszukiwarki, cache)     aplikacje)
  API)       CDC)            hurtownie)      interaktywne, ML)
                  └────────── orkestracja, katalog metadanych, jakość danych, bezpieczeństwo ──────────┘
```

## 1. Pozyskiwanie dużych danych (_data ingestion_)
### Tryby
- **wsadowy** (_batch_) – okresowy import (co godzinę, dobę): eksport z baz, pliki CSV/Parquet, **ETL** (extract-transform-load) lub **ELT** (najpierw ładowanie surowych danych, transformacja w hurtowni),
- **strumieniowy** (_streaming / real-time_) – ciągły napływ zdarzeń z niskim opóźnieniem,
- **CDC** (_Change Data Capture_) – przechwytywanie zmian z logów transakcyjnych baz (binlog MySQL, WAL PostgreSQL) i publikacja jako strumień zdarzeń (Debezium).

### Narzędzia
| Narzędzie | Rola |
|---|---|
| **Apache Kafka** | rozproszony, trwały **log zdarzeń** (commit log): **tematy** (_topics_) podzielone na **partycje** (jednostka równoległości i uporządkowania), **offsety** (pozycja w logu), **grupy konsumentów** (każda partycja czytana przez jednego konsumenta w grupie – skalowanie), replikacja partycji (lider + ISR – in-sync replicas), retencja czasowa/rozmiarowa lub **kompakcja logu** (ostatnia wartość per klucz); semantyka at-least-once, **exactly-once** (idempotentny producent + transakcje); **Kafka Connect** (konektory źródeł/ujść), **Schema Registry** (Avro/Protobuf). Podstawa architektury Kappa i pracy magisterskiej ([[Magisterka/EDA]]) |
| Amazon Kinesis, Google Pub/Sub, Azure Event Hubs | zarządzane odpowiedniki Kafki |
| RabbitMQ, Apache Pulsar | kolejki komunikatów / pub-sub (Pulsar: warstwowe składowanie BookKeeper) |
| **Apache Flume** | zbieranie i przesyłanie logów do HDFS (źródło → kanał → ujście) |
| **Apache NiFi** | graficzne przepływy danych, śledzenie pochodzenia (_provenance_), transformacje |
| **Apache Sqoop** (wycofany) | import/eksport między bazami relacyjnymi a HDFS |
| **Logstash**, **Fluentd**, Fluent Bit, Beats | zbieranie i parsowanie logów – [[Logstash]], [[Zarządzanie Systemami Rozproszonymi/Fluentd]] |
| **Debezium** | CDC z baz do Kafki |
| MQTT (Mosquitto, EMQX) | pozyskiwanie z urządzeń IoT |
| Airbyte, Fivetran | ELT z API i SaaS |

### Wyzwania
wolumen i szczyty (buforowanie w logu), **kolejność i duplikaty** (idempotencja, klucze deduplikacji), **ewolucja schematu**, **opóźnione dane** (event time vs processing time, znaczniki wodne – _watermarks_), walidacja jakości, przeciwciśnienie, bezpieczeństwo w tranzycie.

## 2. Składowanie dużych danych
### Rodzaje magazynów
| Magazyn | Charakter | Technologie |
|---|---|---|
| **rozproszony system plików** | duże pliki, zapis sekwencyjny, lokalność danych dla obliczeń | **HDFS** – [[53 Przetwarzanie dużych danych - Hadoop i Apache Spark#HDFS – Hadoop Distributed File System]], Ceph, GlusterFS |
| **magazyn obiektowy** | nieograniczona skala, tani, niezależny od obliczeń (rozdzielenie _storage_ i _compute_) | **Amazon S3**, Azure Blob/ADLS, Google Cloud Storage, MinIO, Ceph RGW |
| **jezioro danych** (_data lake_) | **surowe dane** w natywnym formacie, schemat przy odczycie (_schema-on-read_) | HDFS/S3 + katalog metadanych (Hive Metastore, AWS Glue) |
| **hurtownia danych** (_data warehouse_) | dane oczyszczone, ustrukturyzowane, **schemat przy zapisie**, zoptymalizowane pod SQL i BI (kolumnowe, MPP) | Snowflake, Google BigQuery, Amazon Redshift, Teradata, ClickHouse |
| **lakehouse** | jezioro danych z **transakcjami ACID**, wersjonowaniem i schematem tabel na plikach w magazynie obiektowym | **Delta Lake**, **Apache Iceberg**, Apache Hudi (+ Spark, Trino) |
| **bazy NoSQL** | dostęp operacyjny o niskim opóźnieniu | HBase, Cassandra, MongoDB – [[51 Big Data, NoSQL, CAP i PACELC - uzupełnienie]] |
| **wyszukiwarki** | pełnotekstowe wyszukiwanie i analityka logów | Elasticsearch/OpenSearch (ELK – [[Zarządzanie Systemami Rozproszonymi/ELK Stack]]) |
| **bazy szeregów czasowych** | metryki, IoT | InfluxDB, TimescaleDB, Prometheus |

### Jezioro danych vs hurtownia
| | Data lake | Data warehouse |
|---|---|---|
| Dane | surowe, wszystkie typy | przetworzone, ustrukturyzowane |
| Schemat | przy odczycie | przy zapisie |
| Użytkownicy | data scientists, inżynierowie | analitycy biznesowi, BI |
| Koszt składowania | niski | wyższy |
| Ryzyko | **„bagno danych”** (_data swamp_) bez katalogu i jakości | mała elastyczność |

Warstwy jeziora (architektura medalionowa): **bronze** (surowe) → **silver** (oczyszczone, zintegrowane) → **gold** (agregaty biznesowe).

### Formaty plików
- **kolumnowe**: **Apache Parquet**, **ORC** – kompresja per kolumna (kodowanie słownikowe, RLE), **pomijanie danych** (statystyki min/max w blokach, _predicate pushdown_), odczyt tylko potrzebnych kolumn → wydajne zapytania analityczne,
- **wierszowe**: **Avro** (schemat w pliku, ewolucja schematu – dobre dla strumieni i zapisu), JSON/CSV (wymiana, słaba wydajność), SequenceFile,
- **kompresja**: Snappy, LZ4 (szybkie), Zstd, gzip (lepszy stopień); **dzielalność** plików (_splittable_) ważna dla równoległego przetwarzania.

### Zasady składowania
replikacja lub erasure coding, partycjonowanie danych (katalogi `rok=2024/miesiąc=05/`), retencja i warstwy (gorące/zimne/archiwalne), lokalność danych (Hadoop) lub rozdzielenie składowania od obliczeń (chmura), **katalog metadanych** i **lineage** (Apache Atlas, DataHub), zarządzanie dostępem (Apache Ranger, Lake Formation), szyfrowanie, zgodność z RODO (prawo do usunięcia – trudne w logach niemutowalnych).

## 3. Przetwarzanie dużych danych
### Wsadowe (_batch_)
Przetwarzanie **całości** zgromadzonych danych, wysokie opóźnienie (minuty–godziny), wysoka przepustowość i dokładność.
- **Hadoop MapReduce**, **Apache Spark** (w pamięci) – [[53 Przetwarzanie dużych danych - Hadoop i Apache Spark]],
- SQL na dużych danych: **Apache Hive** (SQL → MapReduce/Tez/Spark), Spark SQL, **Presto/Trino**, Impala (MPP, zapytania interaktywne na jeziorze), Apache Pig (skrypty – historyczne),
- Apache Tez, Apache Beam (jednolity model wsad/strumień, uruchamiany na Flink/Spark/Dataflow).

### Strumieniowe (_stream_)
Przetwarzanie **zdarzeń w chwili napływu** – niskie opóźnienie (ms–s).
- **Apache Flink** – prawdziwe przetwarzanie strumieniowe (zdarzenie po zdarzeniu), **stan zarządzany** z punktami kontrolnymi (algorytm Chandy-Lamport – asynchroniczne migawki barier, **exactly-once**), **czas zdarzenia** i **znaczniki wodne**, okna (przesuwne, kroczące, sesyjne),
- **Spark Structured Streaming** – mikro-wsady (lub tryb ciągły), API DataFrame jak dla wsadu,
- **Kafka Streams**, ksqlDB – przetwarzanie jako biblioteka w aplikacji,
- **Apache Storm** (topologie spouts/bolts, at-least-once – historyczny), Apache Samza, Google Dataflow.
- **Pojęcia**: okna czasowe, agregacje stanowe, złączenia strumieni, obsługa opóźnionych zdarzeń, semantyki dostarczenia (at-most-once / at-least-once / exactly-once), przeciwciśnienie.

### Interaktywne i analityczne
zapytania ad hoc (Trino, BigQuery, Druid, ClickHouse – OLAP w czasie zbliżonym do rzeczywistego), notatniki (Jupyter, Zeppelin, Databricks).

### Uczenie maszynowe
Spark MLlib, TensorFlow/PyTorch rozproszone, **Ray**, Dask, systemy cech (_feature stores_ – Feast), MLflow (cykl życia modeli).

### Grafy
Spark GraphX, Apache Giraph (model Pregel – „myśl jak wierzchołek”: supersteps z wymianą komunikatów), Neo4j.

### Architektury przetwarzania
| | Lambda | Kappa |
|---|---|---|
| Ścieżki | wsadowa (dokładna) + szybka (przybliżona) | jedna – strumieniowa |
| Ponowne przeliczenie | z surowych danych w warstwie wsadowej | odtworzenie strumienia z logu (Kafka) |
| Kod | dwie implementacje logiki | jedna |
| Technologie | Hadoop/Spark + Storm/Flink + warstwa serwująca (HBase, Druid) | Kafka + Flink/Kafka Streams |

Szczegóły: [[Systemy Rozproszone Dużej Skali/Architektura Lambda]], [[Systemy Rozproszone Dużej Skali/Architektura Kappa]]. Współcześnie także **architektura lakehouse** (wsad i strumień zapisują do tych samych tabel Delta/Iceberg) i **data mesh** (decentralizacja własności danych na domeny biznesowe, dane jako produkt).

## 4. Serwowanie, analiza, prezentacja
bazy serwujące wyniki (Cassandra, HBase, Redis, Druid), hurtownie i BI (Tableau, Power BI, Superset, Grafana), API danych, wyszukiwarki, dashboardy czasu rzeczywistego, modele ML w produkcji.

## 5. Warstwy przekrojowe
- **Orkestracja przepływów**: **Apache Airflow** (DAG zadań w Pythonie), Oozie (Hadoop), Dagster, Prefect, Argo Workflows,
- **Zarządzanie zasobami klastra**: YARN, Kubernetes, Mesos,
- **Koordynacja**: **Apache ZooKeeper** (konsensus ZAB – konfiguracja, elekcja, blokady; Kafka do 3.x, HBase), etcd,
- **Jakość danych**: Great Expectations, Deequ; **zarządzanie metadanymi i pochodzeniem danych**,
- **Bezpieczeństwo**: Kerberos (Hadoop), Ranger, szyfrowanie, maskowanie danych,
- **Monitorowanie**: metryki potoków, opóźnienia konsumentów Kafki (_consumer lag_), alerty.

## Przykładowy stos (referencyjny)
```
IoT/aplikacje ─▶ Kafka ─┬─▶ Flink (strumień: anomalie w czasie rzeczywistym) ─▶ Redis / Elasticsearch ─▶ Grafana
                        └─▶ Kafka Connect ─▶ S3 (Parquet, Iceberg: bronze/silver/gold)
                                                   │
                           Airflow ─▶ Spark (wsad: agregaty, ML) ─▶ Iceberg gold ─▶ Trino ─▶ Superset
```
