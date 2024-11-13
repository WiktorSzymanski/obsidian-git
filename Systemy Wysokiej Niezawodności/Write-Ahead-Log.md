---
up: "[[Wycofanie operacji]]"
tags:
  - SystemyWysokiejNiezawodności
---
# Write-Ahead-Log
---
> Jest rozwinięciem [[Update-In-Place]] mające na celu zapewnić **atomowość**. Polega na wprowadzaniu zmian stanu po wpisaniu informacji o tym do *log*-u.

Inaczej mówiąc:
- *update* jest wykonywany zawsze **po** zapisaniu informacji **UNDO**
- **przed** zatwierdzeniem zmian zapisuje się informacje **REDO**