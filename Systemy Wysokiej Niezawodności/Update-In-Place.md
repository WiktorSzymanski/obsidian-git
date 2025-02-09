---
tags:
  - SystemyWysokiejNiezawodności
up: "[[Wycofanie operacji]]"
---
# Update-In-Place
---
> Każdy zapis (*update*) rejestruje jednocześnie krotkę `log=(OBJ, UNDO, REDO)` w logu, gdzie:
> - **OBJ** → identyfikator modyfikowanego obiektu
> - **UNDO** → stan obiektu sprzed modyfikacji
> - **REDO** → nowy stan obiektu 

Odtwarzalna operacja jest implementowana w postaci kolekcji operacji:
- *do*, wykonuje działanie (*update*) i rejestruje je (*log*)
- *undo*, wycofuje działanie *do* zgodnie z polem **UNDO**
- *redo*, odtwarza działanie *do* zgodnie z polem **REDO**

Przy tym rozwiązaniu występuje problem braku atomowości.