---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 3
source: "brak"
slajdy: "—"
---
# RSO 03. Modele spójności zorientowane na klienta
---
> [!warning] Brak materiału w prezentacjach
> Zagadnienie **nie jest omówione** w żadnej z siedmiu prezentacji z katalogu `~/Documents/RozproszoneSystemyOperacyjne/`.

## Wynik weryfikacji

Przeszukano warstwę tekstową wszystkich 7 plików PDF (`rso_sum_01` – `rso_sum_07`, łącznie ok. 430 slajdów) pod kątem następujących fraz — **0 trafień** dla każdej:

| Szukana fraza | Trafienia |
|---|---|
| `RYW` / `Read Your Writes` | 0 |
| `monoton` (Monotonic Reads / Writes) | 0 |
| `WFR` / `Writes Follow Reads` | 0 |
| `sesji` (gwarancje sesji) | 0 |
| `zorientowan…na klienta` | 0 |
| `mobiln` w kontekście modeli spójności | patrz niżej |

**Jedyna wzmianka** w całym materiale znajduje się na slajdzie 26 prezentacji `rso_sum_06.pdf`:

> **Modele spójności nastawione na klienta**
> • uwzględnienie mobilności klienta

— czyli **jedno zdanie**, bez definicji, przykładów i implementacji. Zob. [[RSO 02 Danocentryczne modele spójności#Klasyfikacja modeli spójności replik]].

---
## Checklista do uzupełnienia z innych źródeł

Zakres wymagany przez listę zagadnień egzaminacyjnych (`egz_dypl_SRC_2024.pdf`, zag. 3):

- [ ] **TODO** Założenia modelu: klient mobilny, przełączanie między serwerami, zbiory zapisów WS i odczytów RS
- [ ] **TODO** **RYW** — _Read Your Writes_ (odczyt własnych zapisów)
- [ ] **TODO** **MR** — _Monotonic Reads_ (monotoniczne odczyty)
- [ ] **TODO** **MW** — _Monotonic Writes_ (monotoniczne zapisy)
- [ ] **TODO** **WFR** — _Writes Follow Reads_ (zapisy po odczytach)
- [ ] **TODO** Przykłady **naruszenia** każdego z czterech modeli (diagramy przestrzenno-czasowe)
- [ ] **TODO** Implementacja: wektory wersji / znaczniki sesji
- [ ] **TODO** Relacja do modeli danocentrycznych — czy modele klienckie są słabsze/silniejsze, czy ortogonalne

## Gdzie szukać

- **Wersja pierwsza tej notatki** (napisana z wiedzy ogólnej, nie ze slajdów): `0-przygotowanieDoObrony/01 Rozproszone systemy operacyjne/03 Modele spójności zorientowane na klienta.md`
- Notatka vaultu: [[Algorytmy Rozproszone/Model Spójności#Klasyfikacja]]
- Tanenbaum, van Steen, _Distributed Systems_ — rozdział o spójności zorientowanej na klienta (_client-centric consistency_)

---
## Powiązania
- Klasyfikacja modeli spójności replik (jedyna wzmianka w prezentacjach) → [[RSO 02 Danocentryczne modele spójności#Klasyfikacja modeli spójności replik]]
- Zwielokrotnianie i mobilność jako motywacja → [[RSO 02 Danocentryczne modele spójności#Zwielokrotnianie — motywacja]]
