---
up: "[[Systemy Plików]]"
tags: ZarządzanieSystemamiKomputerowymi
---
# OverlayFS
---

>to nowoczesny, warstwowy system plików, będący prostszą i efektywniejszą alternatywą do [[UnionFS]]. Działa poprzez łączenie dwóch systemów plików, jednej dolnej i jednej górnej warstwy, tworząc spójną przestrzeń plików.

**Dolna warstwa** jest tylko do odczytu, dane które zawiera są niezmienne. **Górna warstwa** jest pewnego rodzaju nakładką, gdzie wszystkie przez nas wprowadzone zmiany są zapisywane, bez naruszania warstwy dolnej. Ze względu na te właściwość OverlayFS jest wykorzystywany do konteneryzacji. Warto zauważyć, że jeśli plik w górnej warstwie nazywa się tak samo jak plik dolnej warstwy, to nie nadpisuje on pliku warstwy niższej, a raczej go ukrywa.