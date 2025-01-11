---
tags:
  - KonstrukcjaSystemówChmurowych
up: "[[Wirtualizacja]]"
---
# Wirtualizacja w procesorach z rodziny x86
---
Normalnie działający system operacyjny:
- **Ring 0** -> bezpośredni dostęp do pamięci i urządzeń
- **Ring 3** -> aplikacje
- **Virtual Machine Monitor** (**VMM**) musi być poniżej systemu operacyjnego (**OS**)

![[Pasted image 20241228152847.png]]

#### Instrukcje uprzywilejowane
- wymagają trybu uprzywilejowanego (Ring 0, Ring 123 -> trap)
#### Instrukcje krytyczne
- wpływają na konfigurację zasobów w systemie lub ich zachowanie zależy od konfiguracji zasobów
#### Instrukcje wrażliwe
- krytyczne ale nie uprzywilejowane

## Warunki wirtualizacji
### Warunki Popka i Goldberga (1974)
- wszystkie **wrażliwe instrukcje** wykonywane w trybie użytkownika są przechwytywane i kierowane do **VMM**
	- procesory x86 Intela nie spełniają tego warunku
	- w IBM System/370 wszystkie instrukcje wrażliwe są uprzywilejowane (*mainframe 1970*)

### Metody obsługi instrukcji wrażliwych
- [[Pełna Wirtualizacja]]
- [[Parawirtualizacja]]
- [[Wirtualizacja Wspierana Sprzętowo|wsparcie sprzętowe]]