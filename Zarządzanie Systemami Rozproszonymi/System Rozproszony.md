---
tags:
  - ZarządzanieSystemamiRozproszonymi
  - NarzędziaPrzetwarzaniaRozproszonego
up:
---

# System Rozproszony
---
> System rozproszony jest naturalnym krokiem ewolucyjnym przy skalowaniu aplikacji. System rozproszony wprowadza złożoność, która jest poważnym wyzwaniem przy utrzymaniu stabilności ciągle rozwijającej się aplikacji.

**Skalowalność:** Zwiększanie mocy obliczeniowej
**Dostępność:** Redundancja, zapobieganie awariom
**Wydajność:** Rozpraszanie obciążeń
#### Przykłady
- Netflix
- Amazon Web Services

[[Podstawowe Własności Systemu Rozproszonego]]
[[Paradygmat Interakcji Pomiędzy Zdalnymi Jednostkami]]
### Organizacja oprogramowania
#TODO obrazki z prezentacji


### Koncepcja dostępu do współdzielonych zasobów
- Użytkownik zasobu - modół (program, proces) zgłaszający zapotrzebowanie na zasób, żądający dostępu (wykonania operacji)
- Zarządca zasobu (ang. resource manager) - moduł oprogramowania odpowiedzialny za udostępnianie zasobu (koordynację i realizację operacji dostępu, żądanych przez użytkownika)
