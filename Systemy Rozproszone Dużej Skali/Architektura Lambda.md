---
tags:
  - SystemyRozproszoneDużejSkali
---
# Architektura Lambda
---
> Ma na celu rozwiązać problem opóźnień i starych wyników poprzez utworzenie **dwóch ścieżek dla danych**.

Wszystkie dane wchodzące do systemu przechodzą przez dwie ścieżki:
- *Hot path (speed layer)*  - analizowanie danych i podejmowanie decyzji z niepełnym zbiorem danych ale w **czasie rzeczywistym**. Zaprojektowany z myślą o **małym opóźnieniu** kosztem dokładności.
- *Cold path (batch layer)* - przechowuje wszystkie przychodzące dane w ich **surowej formie** i wykonuje *batch processing* na tych danych. Wyniki są przechowywane w postaci *batch view*. Pozwala na **dużą dokładność** na dużych zbiorach danych, co może być **czasochłonne**. 

![[SmartSelect_20241010_170658_Samsung Notes.jpg]]

Jeśli aplikacja klienta musi wyświetlać dane **najbardziej aktualne** jednak potencjalnie **mniej dokładne** dane w czasie rzeczywistym, zdobędzie je z *hot path*. W przeciwnym wypadku pobierze dane z *cold path* by wyświetlić dane mniej aktualne lecz **bardziej dokładne**.

*Hot path* ma dane przez relatywnie krótki czas, po którym są one zastąpione bardziej dokładnymi z *cold path*.

## Dane w Lambda
**Surowe dane** (ang. *raw data*) przechowywane w *batch layer* są **niemutowalne** (ang. *immutable*).**Przychodzące dane** zawsze są **dopisywane do istniejących danych**, a poprzednie dane nigdy nie są nadpisywane. **Wszelkie zmiany** pewnych wartości są przechowywane jako **nowy rekord** zdarzenia ze znacznikiem czasu. To pozwala na **ponowne przeliczenie w dowolnym momęcie w czasie** na historii gromadzonych danych. Umiejętność ponownego przeliczania *batch view* z oryginalnych surowych danych jest ważne, bo pozwala na tworzenie **nowych widoków** w ewolującym systemie.
## Zalety
- mamy dostęp do **najbardziej aktualnych danych**
- **ostatecznie** mamy **dokładne** dane
- przechowuje dane w **surowej formie**
## Wady
- duża **złożoność**
- utrzymywanie **dwóch** rodzajów systemu
- zależność od *framework*-u
- **przetwarzanie** tych samych danych **dwa razy**
- kosztowność *batch layer*

Dużą wadą **architektury Lambda** jest **złożoność**. **Logika** przetwarzania **występuje w dwóch różnych** miejscach - *cold* i *hot path* - z wykorzystaniem różnych *framework*-ów. To prowadzi do **duplikowania logiki przetwarzania** i złożoności **zarządzanej architektóry dla obu ścieżek**. To prowadziło do powstania [[Architektura Kappa|architektury Kappa]].