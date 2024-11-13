---
tags:
  - SystemyWysokiejNiezawodności
up: "[[Odtwarzanie]]"
---
# Odtwarzanie Wsteczne 
---
>Rodzaj [[Odtwarzanie|odtwarzania]] polegającym na przywróceniu systyemu do stanu sprzed awarii, zanim wystąpił błąd. Jest to **uniwersalne** podejście nie zależne od rodzaju błędu. Może być stosowane dla każdego systemu i może być implementowane jako ogólny mechanizm.

## Wymagania
- wcześniejszy stan musi być poprawnie przywrócony
- przywrócony stan musi poprzedxzać wystąpienie uszkodzenia powodującego błąd. W przeciwnym razie **odtwarzanie wsteczne** jest nieefektywne ponieważ pewna operacja nadal będzie prowadzić do [[Awarie|awarii]].
## Cena
- **narzut** - odtwarzanie wsteczne może wprowadzać duży narzut
- **nawrót** - na ogół nie ma gwarancji, że awaria nie wystąpi po odtworzeniu stanu
- **niepowtarzalność** (niemożliwość wycofania) - nie wszystkie komponenty systemu są odtwarzalne, a szczególnie intearakcje zewnętrzne (np. wydanie przez bankomat pieniądze)

[[Odtwarzanie Wsteczne Węzła]]