---
tags:
  - SystemyWysokiejNiezawodności
up: "[[Odtwarzanie Wsteczne Węzła]]"
---
## Wycofanie Operacji
---
> (*operation-based recovery*) to przechowywanie w rejestrach (*log: audit trail*) niezbędnych informacji o wykonywanych operacjach. Wyróżniamy dwie strategie rejestrowania modyfikacji danych:
> - **[[Update-In-Place]]**
> - **[[Write-Ahead-Log]]**

**Efekt domino** - sytiacja w której w procesie wybierania punktów kontrolnych trafiamy na sam początek działania systemu.