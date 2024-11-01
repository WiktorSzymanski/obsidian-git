---
tags:
  - SystemyWysokiejNiezawodności
up: "[[Odtwarzanie]]"
---
# Awarie w systemie
---

| awarie              | przyczyny                                         | reakcje                                |
| ------------------- | ------------------------------------------------- | -------------------------------------- |
| **procesu**         | zakleszczenie, timeout, błąd ochrony, niespójność | abort, restart przetwarzania           |
| **węzła**           | błędy programowe lub sprzętowe, błędy zasilania   | stop i restart ze zdefiniowanego stanu |
| **pamięci masowej** | -                                                 | rekonstrukcja z archiwum               |
| **komunikacyjne**   | awaria medium lub urządzeń sieciowych             | naprawa, retransmisja                  |

## Typy awarii systemowych w modelu *fail-recovery*
- **przerwa** - system restartuje w tym samym stanie, który poprzedzał awarię
- **amnezja** - system restartuje w predefiniowaym stanie niezależnym od stanu w momencie wystąpienia awarii
- **częściowa amnezja** - system restartuje w stanie, którego część pokrywa się ze stanem w momencie wystąpienia awarii, a pozostała część jest predefiniowana. (np. awarie po których następuje restart serwera plików)