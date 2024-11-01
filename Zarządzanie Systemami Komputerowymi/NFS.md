---
up: 
tags: ZarządzanieSystemamiKomputerowymi
---
# Network File System
---

>to rozproszony system plików stworzony przez firmę **Sun Microsystems**. Jest bezstanowym protokołem do tworzenia [[Bezstanowy Serwer Plików|bezstanowych serwerów plików]].

## Pamięć podręczna serwera NFS
wykorzystywana aby szybko obsłużyć żądania odczytu tych samych danych. Dodatkowo może być wykorzystywana aby szybciej przetwarzać zapis z punktu widzenia klienta.
W takiej sytuacji zwraca zakończenie zapisu w momencie gdy dane pojawią się w pamięci RAM. Taki tryb działania nazywa się `async`. Minusem tego podejścia jest utrata danych które nie zdążyły się zapisać do pamięci trwałej w momencie awarii systemu. Odwrotnie działa tryb `sync`. Zwraca pozytywne zakończenie operacji zapisu dopiero gdy wszystkie dane są zapisane na trwałym nośniku danych.