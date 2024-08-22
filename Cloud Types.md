---
up: 
class: MBP
---
# Typy Cloud-owe
---

Ich zadaniem jest aby program rozproszony i zrównoleglony, był pisany jak program sekwencyjny, bez umiejętności programowania równoległego.

`yield()` - jest jedyną częścią kodu która jest równoległa. Jest to uniwersalne polecenie do zaktualizowania stanu. Synchronizuje Cloudowe typy danych.

"Jak może to wykona, jak nie to idziemy dalej"

`flush` - wymaga ciągłego połaczenia z serwerem, gwarantuję **silną spójność**. Wymusza blokującą synchronizację z serwerem.
#### Przykład:
``` C
seat.assignedTo.setIfEmpty(customer);
flush;
if (seat.assignedTo.get() != customer) print("reservation failed");
```
