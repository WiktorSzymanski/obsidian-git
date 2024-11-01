---
tags:
  - SystemyRozproszoneDużejSkali
---
# Architektura Kappa
---
>Alternatywa do [[Architektura Lambda|architektury Lambda]], taki sam cel ale dane przechodzą przez **jedną ścieżkę** używając *stream processing system*. Podobne do *speed layer* z [[Architektura Lambda|lambdy]] w kwestii przetwarzania wszystkich zdarzeń na strumieniu wejściowym i przedstawianym jako *real-time view*.

![[SmartSelect_20241021_155141_Samsung Notes.jpg]]

Jeśli potrzeba wykonać operacje na **całym zbiorze danych** (*batch layer* w [[Architektura Lambda|lambdzie]]) wystarczy **odtworzyć strumień** zwykle używając zrównoleglenia w celu szybszego zakończenia operacji. W przeciwieństwie do [[Architektura Lambda|lambdy]] w tym rozwiązaniu **ponownie przetwarza** się dane tylko kiedy **zmienia się kod** i koniecznym jest ponowne przeliczenie wyników.

## Dane w Kappa
- Dane są **niemutowalne** (ang. *Immutable*) i ich **całość** jest zapisywana
- Dane są przyjmowane jako **strumień zdarzeń** do **rozproszonego, zunifikowanego i odpornego na błędy logu**.
- Zdarzenia są **sortowane**
- Odwrócenie działania zaistniałego już zdarzenia może być zmienione tylko przez **nowe zdarzenie**. 

## Zalety
- Ponowne przeliczanie konieczne tylko gdy zmienia się kod
- Może zostać wystawione ze **stałą ilością pamięci**
- Może być wykorzystywane do systemów skalujących się **horyzontalnie**
## Wady
- Błędy podczas przetwarzania danych z powodu braku *batch layer*

Podczas gdy [[Architektura Lambda|lambda]] zawsze potrzebuje dwóch działających instancji, **Kappa** wymaga drugiej tylko w przypadku ponownego **przeliczania wyników**. W przypadku architektóry **Kappa** wymagane jest **2x więczej miejsca na dane wynikowe** i wymaga by **baza danych wspierała wysokiej przepustowości odczyt** potrzebny przy ponownym przeliczaniu. 