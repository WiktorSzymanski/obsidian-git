---
class: SRDS
---
# Wstęp
---
>*Big Data* to zbiór danych tak duży, że jego przetwarzanie jest trudne dla relacyjnej bazy danych. *Big Data* to dane skalujące się, rozproszone i różnorodne, ich przetwarzanie wymaga specjalnej architektóry.

## Charakterystyka Big Data
- *Volume* - wielkość danych
- *Variety* - różnorodność danych
- *Velocity* - szybkość generowania danych
- *Veracity* - zaufanie do danych
- *Value* - wartość danych, czyli czy na podstawie tych danych możemy podjąć jakieś decyzje
- *Visualisation* - przedstawianie danych
- - czy pasują do aplikacji którą tworzymy
- - jak długo dane będą nam potrzebne

### Różnorodność danych
- *Structured*
- *Semi-structured*
- *Unstructured*
#TODO obrazek z prezki
## Skalowalność
- *Scale up* or *Scale vertically* - droższe ale prostrze w zarządaniu
- *Scale out* lub *Scale horizontaly* - tańcze, więcej wyzwań o zapewnienie tego skalowania, problem z komunikacją między węzłami

Im więcej maszyn tym więcej ukrytych problemów. Systemu nie da się skalować w nieskończoność.

Lepsze jest wiele małych, bez stanowych servisów
- *Stateless* - #TODO
- *Statefull* - przychodzące rządania polegają na poprzednim stanie, co za tym idzie wszystkie muszą trafić na ten sam węzeł, który zna jego stan

---
#TODO
- *Sharding* - dzielenie, porcjowanie danych na wielu węzłach
- *Caching* - 

**To wyżej to w sumie problemy BIG DATA**

# Warstwy Big Data
---
#TODO obrazek z prez
## Big Data Stack
#TODO obrazek z prez

## Architektura
#TODO obrazek z perz
### Komponenty
- *Data sources* -
- *Data storage* -
- *Batch processing* -
- *Real-time message ingestion* -
- *Stream processing* - 
- #TODO 

### Rodzaje architektury
- *Decoupled data bus
	- Drogie
- *Cloud-based infrastructure / Fog / Edge infrastractures*
	- Effektywność kosztów na średnim poziomie
- *Lambda & Kappa architecures*
	- 
#TODO 

## Przetwarzanie dużych zbiorów danych
- *Batch processing*
- *Stream processing*
### Real time
- *Macro Batch*
- *Micro Batch*
- *Near Real Time Decision Support*
- *Near Real Time Event Processing*
- *Real Time*
#TODO obrazek z prez

