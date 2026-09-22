---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 53
---
# 53. Przetwarzanie dużych danych z wykorzystaniem platformy Hadoop i Apache Spark
---
> **Apache Hadoop** i **Apache Spark** to otwarte platformy do **rozproszonego przetwarzania bardzo dużych zbiorów danych** na klastrach tanich komputerów. Hadoop spopularyzował model **MapReduce** i rozproszony system plików **HDFS**, oparte na **przenoszeniu obliczeń do danych**. Spark zastąpił MapReduce jako silnik obliczeń dzięki przetwarzaniu **w pamięci** i bogatszemu modelowi programowania (**RDD**, DataFrame).

---
## Apache Hadoop
**Historia**: Google opublikował **GFS** (2003) i **MapReduce** (Dean, Ghemawat, OSDI 2004) → Doug Cutting i Mike Cafarella implementują je w projekcie wyszukiwarki Nutch → **Hadoop** (Yahoo!, 2006, Apache top-level 2008). Hadoop 2 (2013) – **YARN**; Hadoop 3 (2017) – erasure coding, wiele NameNode'ów.

**Założenia projektowe**:
- awarie sprzętu są **normą** → odporność realizowana programowo (replikacja, ponowne wykonanie),
- **przenoszenie obliczeń do danych** (lokalność) jest tańsze niż przesyłanie danych przez sieć,
- duże pliki, dostęp **sekwencyjny**, model **zapisz raz, czytaj wiele** (_write-once-read-many_),
- **skalowanie poziome** na commodity hardware.

**Główne moduły**: Hadoop Common, **HDFS** (składowanie), **YARN** (zarządzanie zasobami), **MapReduce** (przetwarzanie).

### HDFS – Hadoop Distributed File System
**Architektura mistrz–robotnicy**:
```
          ┌──────────────────────────┐
klient ──▶│ NameNode (mistrz)         │  metadane: przestrzeń nazw, pliki → bloki, bloki → DataNode
  │       │ fsimage + edit log        │
  │       └──────────────────────────┘
  │             ▲ heartbeat (3 s), raporty bloków
  │             │
  └──dane──▶ DataNode 1   DataNode 2   DataNode 3   ...   (bloki na lokalnych dyskach)
```
- **NameNode**:
  - przechowuje **przestrzeń nazw** (drzewo katalogów, uprawnienia) i **odwzorowanie plików na bloki** oraz **bloków na DataNode** – całość **w pamięci RAM** (szybkość; ograniczenie liczby plików – **problem małych plików**),
  - trwałość: **fsimage** (migawka przestrzeni nazw) + **edit log** (dziennik zmian), okresowo scalane (checkpoint – Secondary NameNode / Standby),
  - nie przesyła danych – klient komunikuje się z DataNode bezpośrednio,
  - lokalizacje bloków odtwarza z **raportów bloków** DataNode'ów (nie zapisuje ich trwale).
- **DataNode** – przechowuje **bloki** jako pliki w lokalnym SP, wysyła **heartbeat** (co 3 s) i okresowe raporty bloków, obsługuje odczyt/zapis klientów i replikację na polecenie NameNode.
- **Bloki**: duże – domyślnie **128 MiB** (Hadoop 1: 64 MiB) → mało metadanych, długie sekwencyjne odczyty, mniejszy udział czasu wyszukiwania na dysku; plik mniejszy niż blok nie zajmuje całego bloku.
- **Replikacja**: domyślny współczynnik **3**, **świadomość szaf** (_rack awareness_):
  - 1. replika – na węźle zapisującym (lub losowym),
  - 2. replika – na węźle w **innej szafie**,
  - 3. replika – w **tej samej szafie co 2.**, na innym węźle,
  → odporność na awarię całej szafy przy ograniczonym ruchu między szafami.
  Po awarii DataNode (brak heartbeatu przez ~10 min) NameNode zleca **ponowną replikację** brakujących bloków. Hadoop 3: **erasure coding** (np. RS-6-3 – narzut 1,5× zamiast 3×) dla danych zimnych.
- **Zapis** (potok):
  1. klient pyta NameNode o utworzenie pliku i przydział bloku → lista DataNode'ów,
  2. dane przesyłane pakietami **potokiem** (_pipeline_): klient → DN1 → DN2 → DN3,
  3. potwierdzenia wracają w odwrotnej kolejności,
  4. przy awarii węzła w potoku – potok przebudowany, blok później dosiany do pełnej replikacji.
  Semantyka: **pliki niemodyfikowalne** (tylko tworzenie i dopisywanie – _append_), **jeden pisarz**.
- **Odczyt**: klient pobiera od NameNode lokalizacje bloków (posortowane wg bliskości sieciowej) i czyta **bezpośrednio** z najbliższego DataNode; **sumy kontrolne** (CRC32C) weryfikują dane – przy błędzie odczyt z innej repliki i zgłoszenie uszkodzenia.
- **Wysoka dostępność NameNode** (Hadoop 2+): **Active** + **Standby** NameNode, współdzielony edit log na **JournalNode'ach** (kworum), automatyczny failover przez **ZooKeeper** (ZKFC), **fencing** starego aktywnego. **Federacja** – wiele niezależnych NameNode'ów dla różnych części przestrzeni nazw (skalowanie metadanych).
- Dostęp: `hdfs dfs -put/-ls/-cat`, Java API, WebHDFS (REST), montowanie NFS.

### YARN – Yet Another Resource Negotiator (Hadoop 2)
Oddzielenie **zarządzania zasobami** od **modelu programowania** (w Hadoop 1 JobTracker robił obie rzeczy – wąskie gardło przy ~4000 węzłów) → na YARN działają MapReduce, Spark, Tez, Flink.
- **ResourceManager** (globalny, na mistrzu):
  - **Scheduler** – przydziela zasoby (**kontenery**: pamięć, rdzenie wirtualne) wg polityki: **FIFO**, **Capacity Scheduler** (kolejki z gwarantowanymi udziałami – multi-tenancy), **Fair Scheduler** (sprawiedliwy podział między aplikacje/użytkowników),
  - **ApplicationsManager** – przyjmuje zgłoszenia aplikacji i uruchamia ich ApplicationMaster.
- **NodeManager** (na każdym węźle) – uruchamia i monitoruje **kontenery**, raportuje zasoby.
- **ApplicationMaster** (per aplikacja, sam w kontenerze) – negocjuje kontenery z ResourceManagerem, uwzględnia lokalność danych, uruchamia zadania na NodeManagerach, śledzi postęp i obsługuje awarie zadań.
- Przebieg: klient → RM (zgłoszenie) → RM uruchamia AM → AM prosi o kontenery → NM uruchamiają zadania → AM raportuje wynik → zwolnienie zasobów.

### MapReduce
**Model programowania** inspirowany programowaniem funkcyjnym: programista pisze tylko dwie funkcje, a platforma zajmuje się podziałem danych, zrównolegleniem, rozdziałem zadań, odpornością na awarie i komunikacją.
$$map: (k_1, v_1) \rightarrow list(k_2, v_2)$$
$$reduce: (k_2, list(v_2)) \rightarrow list(k_3, v_3)$$

**Przebieg zadania**:
1. **Podział wejścia** (_input splits_) – zwykle 1 split = 1 blok HDFS; `InputFormat` i `RecordReader` zamieniają dane na pary (klucz, wartość) – np. (offset linii, linia tekstu).
2. **Faza map** – dla każdego splitu jedno zadanie map, uruchamiane **lokalnie przy danych** (węzeł z blokiem → ta sama szafa → dowolny); wyniki pośrednie buforowane w pamięci, **dzielone na partycje** (jedna na reduktor): $partycja = hash(k_2) \bmod R$ (`Partitioner`), **sortowane** i zrzucane (_spill_) na **dysk lokalny**.
3. **Combiner** (opcjonalnie) – lokalna, wstępna agregacja wyników map na tym samym węźle (np. sumowanie liczników), zmniejszająca ruch sieciowy; musi być łączna i przemienna (często ta sama funkcja co reduce).
4. **Shuffle & sort** – reduktory **pobierają** swoje partycje od wszystkich maperów przez sieć, **scalają** posortowane strumienie → grupowanie wartości wg klucza.
5. **Faza reduce** – dla każdego klucza wywołanie `reduce(k2, [v2...])`; wynik zapisywany do HDFS (`part-r-00000`…).

**Przykład – WordCount**:
```java
public class WordCount {
  public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
    public void map(Object key, Text value, Context ctx) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) { word.set(itr.nextToken()); ctx.write(word, one); }
    }
  }
  public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    public void reduce(Text key, Iterable<IntWritable> values, Context ctx)
        throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable v : values) sum += v.get();
      ctx.write(key, new IntWritable(sum));
    }
  }
  public static void main(String[] args) throws Exception {
    Job job = Job.getInstance(new Configuration(), "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);        // combiner
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```
```
wejście: "ala ma kota", "kot ma ale"
map:     (ala,1) (ma,1) (kota,1) | (kot,1) (ma,1) (ale,1)
shuffle: ala→[1]  ale→[1]  kot→[1]  kota→[1]  ma→[1,1]
reduce:  (ala,1) (ale,1) (kot,1) (kota,1) (ma,2)
```
Inne typowe wzorce: filtrowanie, zliczanie i agregacje, sortowanie (TeraSort – partycjonowanie zakresowe), odwrócony indeks, złączenia (_reduce-side join_, _map-side / broadcast join_), obliczenia grafowe iteracyjne (PageRank – wiele zadań).

**Odporność na awarie**:
- zadanie, które uległo awarii (lub węzeł), jest **ponownie wykonywane** na innym węźle (dane wejściowe są w HDFS, wyniki map na dyskach lokalnych – po utracie węzła mapy powtarzane),
- **wykonanie spekulatywne** (_speculative execution_) – dla zadań znacznie wolniejszych od pozostałych (_stragglers_) uruchamiana jest kopia; wygrywa pierwsza,
- wymaganie: funkcje map/reduce **deterministyczne i bez efektów ubocznych**.

**Ograniczenia MapReduce**:
- **zapis wyników pośrednich na dysk** po każdej fazie → wolne dla **algorytmów iteracyjnych** (uczenie maszynowe, grafy: każda iteracja = osobne zadanie z odczytem i zapisem HDFS) i **zapytań interaktywnych**,
- sztywny, niskopoziomowy model dwufazowy (złożone przepływy wymagają łańcuchów zadań),
- wysokie opóźnienia, brak przetwarzania strumieniowego.

**Ekosystem Hadoop**: **Hive** (SQL → zadania), **Pig** (skrypty Pig Latin), **HBase** (baza kolumnowa na HDFS – BigTable), **ZooKeeper** (koordynacja), **Oozie** (przepływy zadań), **Sqoop**, **Flume**, **Tez** (DAG zamiast MapReduce), **Ambari**/Cloudera Manager (zarządzanie klastrem), **Ranger**, **Kerberos** (bezpieczeństwo), **Mahout** (ML).

---
## Apache Spark
**Historia**: AMPLab UC Berkeley (Matei Zaharia, 2009), artykuł o **RDD** (NSDI 2012), Apache top-level 2014, firma Databricks. Obecnie dominujący silnik przetwarzania wsadowego Big Data.

**Idea**: ogólny silnik obliczeń rozproszonych na **DAG operacji** z przechowywaniem danych pośrednich **w pamięci** (_in-memory_), jednolity dla przetwarzania wsadowego, SQL, strumieni, ML i grafów. Do **100×** szybszy od MapReduce dla zadań iteracyjnych w pamięci, ok. 10× na dysku.

### RDD – Resilient Distributed Dataset
> **Odporny rozproszony zbiór danych** – **niemutowalna**, **podzielona na partycje** kolekcja elementów przetwarzana równolegle, tworzona **deterministycznymi** operacjami ze stabilnego źródła (HDFS, S3) lub z innych RDD.

- **Partycje** – jednostka równoległości (1 zadanie na partycję), rozmieszczone na węzłach.
- **Linia rodowa** (_lineage_) – RDD pamięta **graf transformacji**, z których powstał. Utracona partycja jest **odtwarzana przez ponowne obliczenie** z przodków (tylko brakujące partycje), bez replikacji danych pośrednich. Stąd „resilient”.
- **Leniwe wartościowanie** – transformacje tylko budują plan; obliczenie następuje dopiero po **akcji** → optymalizacja całego potoku (łączenie operacji).
- **Utrwalanie** (_persist/cache_) – RDD wielokrotnie używany (w iteracjach) trzymany w pamięci: poziomy `MEMORY_ONLY`, `MEMORY_AND_DISK`, `DISK_ONLY`, serializowane, z replikacją (`_2`).
- Opcjonalnie: **partycjoner** (hash/range) i **preferowane lokalizacje** (lokalność danych HDFS).

### Transformacje i akcje
| Transformacje (leniwe, zwracają RDD) | Akcje (uruchamiają obliczenie) |
|---|---|
| `map`, `flatMap`, `filter`, `mapPartitions`, `sample`, `union`, `distinct` | `collect`, `count`, `first`, `take(n)`, `reduce`, `aggregate`, `foreach` |
| `groupByKey`, **`reduceByKey`**, `aggregateByKey`, `sortByKey`, `join`, `cogroup`, `repartition`, `coalesce` | `saveAsTextFile`, `saveAsParquet`, `countByKey` |

**Zależności między RDD**:
- **wąskie** (_narrow_) – każda partycja rodzica używana przez co najwyżej jedną partycję dziecka (`map`, `filter`, `union`): **potokowanie** w jednym zadaniu, tanie odtwarzanie,
- **szerokie** (_wide_) – partycja rodzica używana przez wiele partycji dziecka (`groupByKey`, `reduceByKey`, `join` bez współpartycjonowania): wymagają **shuffle** (przetasowania danych przez sieć, zapis plików shuffle na dysk).
- `reduceByKey` lepsze od `groupByKey` + `map` – agreguje lokalnie **przed** shuffle (jak combiner).

### Architektura wykonawcza
```
┌──────────── Driver ─────────────┐           ┌──── Cluster Manager ────┐
│ SparkSession / SparkContext      │◀────────▶ │ Standalone / YARN /     │
│ DAGScheduler → etapy (stages)    │           │ Kubernetes / (Mesos)    │
│ TaskScheduler → zadania (tasks)  │           └─────────────────────────┘
└───────────────┬──────────────────┘
                │ zadania, zmienne rozgłaszane
     ┌──────────┴───────────┬──────────────────────┐
 Executor 1 (węzeł)     Executor 2              Executor N
 [zadania w wątkach]    [cache partycji]        [bloki shuffle]
```
- **Driver** – proces z programem użytkownika: tworzy `SparkSession`, buduje DAG, planuje i nadzoruje wykonanie.
- **Cluster manager** – przydziela zasoby (YARN, Kubernetes, Standalone).
- **Executory** – procesy JVM na węzłach wykonujące **zadania** (w wątkach) i przechowujące **cache** oraz dane shuffle.
- **Planowanie**:
  1. akcja tworzy **zadanie** (_job_),
  2. **DAGScheduler** dzieli graf RDD na **etapy** (_stages_) na granicach **zależności szerokich** (shuffle); w obrębie etapu wąskie transformacje są potokowane,
  3. każdy etap = zbiór **zadań** (_tasks_), po jednym na partycję,
  4. **TaskScheduler** wysyła zadania do executorów z uwzględnieniem **lokalności danych** (PROCESS_LOCAL → NODE_LOCAL → RACK_LOCAL → ANY),
  5. awaria zadania → ponowienie; utrata wyników shuffle → ponowne wykonanie etapu nadrzędnego (z lineage); zadania spekulatywne.
- **Zmienne współdzielone**: **broadcast** (tylko do odczytu, wysyłane raz do executorów – np. mała tablica do złączenia), **akumulatory** (liczniki/sumy zbierane w driverze).

### Przykład – WordCount
```scala
val spark = SparkSession.builder.appName("WordCount").getOrCreate()
val counts = spark.sparkContext.textFile("hdfs:///dane/tekst.txt")   // RDD[String]
  .flatMap(line => line.split("\\s+"))                              // wąska
  .map(word => (word, 1))                                            // wąska
  .reduceByKey(_ + _)                                                // szeroka → shuffle (nowy etap)
counts.saveAsTextFile("hdfs:///wyniki/wordcount")                    // akcja → uruchomienie
```
```python
# PySpark DataFrame API
from pyspark.sql import SparkSession, functions as F
spark = SparkSession.builder.appName("wc").getOrCreate()
df = spark.read.text("s3://bucket/tekst/")
(df.select(F.explode(F.split("value", r"\s+")).alias("word"))
   .groupBy("word").count()
   .orderBy(F.desc("count"))
   .write.parquet("s3://bucket/wyniki/"))
```
Przykład iteracyjny (PageRank, regresja logistyczna): dane wejściowe **`cache()`** raz, a kolejne iteracje czytają je z pamięci zamiast z HDFS – główna przewaga nad MapReduce.

### Wyższe API i biblioteki
- **Spark SQL, DataFrame i Dataset** – dane tabelaryczne ze **schematem**, deklaratywne operacje (`select`, `filter`, `groupBy`, `join`) i SQL:
  - optymalizator **Catalyst**: plan logiczny → reguły optymalizacji (przepychanie predykatów i projekcji do źródła – Parquet, zwijanie stałych, wybór strategii złączeń: broadcast hash / sort-merge) → plan fizyczny (optymalizacja kosztowa),
  - silnik **Tungsten**: zarządzanie pamięcią poza stertą JVM, format binarny kolumn, **generowanie kodu** całych etapów (_whole-stage codegen_),
  - **Adaptive Query Execution** (Spark 3) – zmiana planu w trakcie na podstawie statystyk shuffle (łączenie małych partycji, obsługa skosu danych),
  - źródła: Parquet, ORC, JSON, CSV, JDBC, Hive, Delta Lake/Iceberg.
- **Structured Streaming** – strumień jako **nieskończenie rosnąca tabela**; to samo API co wsad; mikro-wsady (lub tryb ciągły), czas zdarzenia, **watermarki**, okna, exactly-once z punktami kontrolnymi i idempotentnymi ujściami; źródła Kafka, pliki.
- **MLlib** – ML na dużą skalę: potoki (_Pipelines_: Transformer, Estimator), klasyfikacja, regresja, klasteryzacja (k-means), rekomendacje (ALS), cechy.
- **GraphX / GraphFrames** – przetwarzanie grafów (PageRank, spójne składowe, Pregel API).
- **SparkR, PySpark, pandas API on Spark**.

---
## Hadoop MapReduce vs Spark
| Kryterium | Hadoop MapReduce | Apache Spark |
|---|---|---|
| Model | dwie fazy: map i reduce | dowolny **DAG** transformacji |
| Wyniki pośrednie | **dysk** (HDFS / lokalny) po każdej fazie | **pamięć** (z możliwością dysku) |
| Wydajność | niska dla iteracji i zapytań interaktywnych | do 100× szybszy w pamięci (iteracje, ML), ~10× na dysku |
| Odporność na awarie | ponowne wykonanie zadań, replikacja danych w HDFS | **lineage RDD** – odtwarzanie partycji, punkty kontrolne |
| API | niskopoziomowe (Java), dużo kodu | wysokopoziomowe: Scala, Java, **Python**, R, **SQL**, DataFrame |
| Wartościowanie | natychmiastowe (jawne zadania) | leniwe, optymalizacja całego planu (Catalyst) |
| Przetwarzanie | wsadowe | wsadowe, strumieniowe (mikro-wsady), interaktywne, ML, grafy |
| Opóźnienie | minuty–godziny | sekundy–minuty |
| Wymagania pamięci | niskie (dysk) | wysokie (RAM – koszt), ryzyko OOM |
| Składowanie | HDFS (część Hadoopa) | brak własnego – HDFS, S3, Cassandra, Kafka… |
| Zarządzanie zasobami | YARN | YARN, Kubernetes, Standalone |
| Zastosowanie dziś | legacy, bardzo duże zadania wsadowe na tanim sprzęcie | standard przetwarzania Big Data (ETL, analityka, ML) |

**Relacja**: Spark **nie zastępuje całego Hadoopa** – często działa **na** Hadoopie (dane w **HDFS**, zasoby z **YARN**), zastępując jedynie **MapReduce** jako silnik obliczeń. W chmurze coraz częściej Spark + magazyn obiektowy (S3) + Kubernetes, bez HDFS.

## Zobacz też
- [[52 Architektura systemów Big Data - narzędzia]], [[Systemy Rozproszone Dużej Skali/Architektura Lambda]]
- [[20 Rozproszone systemy plików#Google File System (2003) – dla porównania]]
