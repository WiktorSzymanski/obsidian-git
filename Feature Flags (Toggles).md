---
class: KUB2024
---
# Feature Flags (Toggles)
---
>Używają pewnego serwisu kontrolującego zmiany, aby decydować czy jakaś działająca funkcjonalność ma być udostępniana.

## Kiedy
- zmiana wymaga koordynacji na wielu serwisah
- wymagana duża elastyczność
- szybsze wycofywanie zmian
- wiele testów jednocześnie

## Zalety
- bez downtime
- najszybsze wdrożenia i rollbacki
- rozdzielenie deploymentu od realese-u

## Wady
- wymaga implementacji w kodzie
- wymaga serwisu do synchronizacji (FFS)
- wymaga koordynacji oraz dyscypliny