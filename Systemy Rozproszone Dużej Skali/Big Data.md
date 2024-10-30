---
tags:
  - SystemyRozproszoneDużejSkali
---
# Big Data
---
>*Big Data* to zbiór danych tak duży, że jego przetwarzanie jest trudne dla relacyjnej bazy danych. *Big Data* to dane skalujące się, rozproszone i różnorodne, ich przetwarzanie wymaga specjalnej architektóry.

## Charakterystyka Big Data
### The 3 V's
- *Volume*
	- wielkość danych
	- znacząco rośnie z czasem
- *Variety*
	- heterogeniczność danych
	- różnorodność formatów, typów i struktur
- *Velocity*
	- szybkość generowania danych
	- szybkie przetwarzanie danych by zapewnić ich aktualność

### The 5 V's
- *Varacity*
	- prawdziwość/rzetelność danych, zaufanie do danych
- *Value*
	- wartość danych, czyli czy na podstawie tych danych możemy podjąć jakieś decyzje
### Różnorodność danych
- *Structured*
- *Semi-structured*
- *Unstructured*

![[SmartSelect_20241021_133304_Samsung Notes.jpg]]
## Skalowalność
- *Scale up* or *Scale vertically* - droższe ale prostrze w zarządaniu
- *Scale out* lub *Scale horizontaly* - tańcze, więcej wyzwań o zapewnienie tego skalowania, problem z komunikacją między węzłami

>Im więcej maszyn tym więcej ukrytych problemów.
Systemu nie da się skalować w nieskończoność.


Dla dobrej skalowalności zalecane jest wiele małych, bez stanowych servisów
- *Stateless* - każdy serwis jest wstanie spełnić przychodzące zapytanie, każde zapytanie jest traktowane niezależnie.
- *Statefull* - przychodzące rządania polegają na poprzednim stanie, co za tym idzie wszystkie muszą trafić na ten sam węzeł, który zna jego stan


- *Sharding* - dzielenie, porcjowanie danych (*shards*) na wielu procesach/węzłach (*workers*).
	- Procesy powinny być zaprojektowane by tworzyć wspólny i spójny serwis, pomimo *sharding*-u.
	- Właściwości bazujące na funkcji *hash*-ującej w celu równomiernego podziału danych na serwisach.
	- *Sharding* tworzy wyzwania "semantyczne" zwłaszcza w wypadku awarii.
- *Caching* - każdy serwis potrafi obsłużyć zapytanie *read-only* z *cache*-u, tylko zapytania aktualizujące muszą trafić do danego serwisu.
## Zastosowanie
Rozwiązania **big data** zazwyczaj obejmują co najmniej jeden z następujących typów obciążeń:
- *Batch processing*
- *Real-time processing*
- *Interactive exploration*
- *Predictive analytics and machine learning*
## Architektura
![[SmartSelect_20241021_143013_Samsung Notes.jpg]]
- *Data sources* - stąd przypływają dane do systemu 
- *Data storage* - dla przetwarzania danych w trybie *batch*, dane zwykle sa przechowywane w rozproszonym systemie plików, będącym wstanie przechowywać duże ich ilości. Zwykle nazywany *data lake*.
- *Batch processing* - zbiory danych potrafią być bardzo duże, stąd przetwarzanie ich do analizy może zająć dużo czasu. Wykorzystuje się do tego *long-running bach jobs*.
- *Real-time message ingestion* - w przypadku źródeł danych w czasie rzeczywistym konieczne jest uwzględnienie sposobu przechwytywania i przechowywania przychodzących wiadomości do celów przetwarzania strumieniowego
- *Stream processing* - po przechwyceniu danych strumieniowych rozwiązanie musi je przetworzyć. Często wykorzystuje *sliding window* do danych, które są przetwarzane
- *Analytical data store* - wiele rozwiązań **big data** przygotowuje dane do analizy i podaje je w bardziej ustrukturyzowanym formacie.
- *Analysis and reporting* - celem większości rozwiązań jest zwrócenie wglądu na dane poprzez analize i raporty
### Rodzaje architektur
- *Decoupled data bus
	- Drogie
- *Cloud-based infrastructure / Fog / Edge infrastractures*
	- Effektywność kosztów na średnim poziomie
- *[[Architektura Lambda|Lambda]] & [[Architektura Kappa|Kappa]] architecures*
## Przetwarzanie dużych zbiorów danych
- *Batch processing* - przetwarzanie całości zapisanych danych na raz
- *Stream processing* - przetwarzanie danych w momencie ich pojawienia się w systemie
### Real time
- *Macro Batch* >= 15 minut
- *Micro Batch* > 2 minut ale < 15 minut
- *Near Real Time Decision Support* >= 2 sekundy i < 2 minut
- *Near Real Time Event Processing* > 50 ms i < 2 sekund
- *Real Time* < 50 ms
![[SmartSelect_20241021_145202_Samsung Notes.jpg]]

