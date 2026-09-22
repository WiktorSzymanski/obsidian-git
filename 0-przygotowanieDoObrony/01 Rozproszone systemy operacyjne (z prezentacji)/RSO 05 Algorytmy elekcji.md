---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 5
source: "brak"
slajdy: "—"
---
# RSO 05. Algorytmy elekcji
---
> [!warning] Brak materiału w prezentacjach
> Zagadnienie **nie występuje w ogóle** w żadnej z siedmiu prezentacji z katalogu `~/Documents/RozproszoneSystemyOperacyjne/`. Jest to najbardziej jaskrawa luka w materiale wykładowym względem listy zagadnień egzaminacyjnych.

## Wynik weryfikacji

Przeszukano warstwę tekstową wszystkich 7 plików PDF (`rso_sum_01` – `rso_sum_07`, łącznie ok. 430 slajdów) — **0 trafień** dla każdej z fraz:

| Szukana fraza | Trafienia |
|---|---|
| `elekcj` (elekcja, elekcji) | 0 |
| `Bully` | 0 |
| `Chang` (Chang-Roberts) | 0 |
| `pierścien` w kontekście wyboru lidera | 0 (występuje tylko jako topologia w wykł. 2 i 4) |
| `Hirschberg` / `LeLann` | 0 |
| `Raft` / `Paxos` / `konsensus` | 0 |

**Jedyne wystąpienia słowa „koordynator"** dotyczą:
- `rso_sum_02.pdf` (6 trafień) — koordynator **bariery** w przykładzie liczenia złożoności (slajdy 50–53), zob. [[RSO 08 Czas wirtualny i złożoność algorytmów#Przykład — bariera]],
- `rso_sum_07.pdf` (2 trafienia) — koordynator w **scentralizowanym wzajemnym wykluczaniu** (slajdy 3–4), zob. [[RSO 04 Algorytmy wzajemnego wykluczania#Podejście scentralizowane]].

W obu przypadkach koordynator jest **założony jako już wybrany** — prezentacje **nie mówią, jak go wybrać**.

---
## Checklista do uzupełnienia z innych źródeł

Zakres wymagany przez listę zagadnień egzaminacyjnych (`egz_dypl_SRC_2024.pdf`, zag. 5):

- [ ] **TODO** Definicja problemu elekcji, wymagania (bezpieczeństwo, żywotność), założenia (unikalne identyfikatory, detekcja awarii)
- [ ] **TODO** **Algorytm tyrana (Bully)** — García-Molina; przebieg, złożoność $O(n^2)$
- [ ] **TODO** **Algorytmy pierścieniowe**: Chang-Roberts, LeLann, Hirschberg-Sinclair ($O(n \log n)$)
- [ ] **TODO** **Algorytm echa** (elekcja w grafie dowolnym)
- [ ] **TODO** Elekcja w **Raft** (kadencje, głosowanie większościowe)
- [ ] **TODO** Relacja do problemu konsensusu; wynik **FLP** (niemożliwość konsensusu w systemie w pełni asynchronicznym)
- [ ] **TODO** Zastosowania: wybór koordynatora dla scentralizowanego wzajemnego wykluczania, wybór inicjatora detekcji, wybór lidera replikacji

## Gdzie szukać

- **Wersja pierwsza tej notatki** (napisana z wiedzy ogólnej, nie ze slajdów): `0-przygotowanieDoObrony/01 Rozproszone systemy operacyjne/05 Algorytmy elekcji.md`
- Notatki vaultu z innego przedmiotu: [[Systemy Wysokiej Niezawodności/Algorytm Paxos]], [[Systemy Wysokiej Niezawodności/Replikacja Procesu]]
- Zagadnienie 23 (Systemy wysokiej niezawodności) dotyka rozproszonego uzgadniania — częściowo pokrywa tematykę

---
## Powiązania
- Koordynator w scentralizowanym wzajemnym wykluczaniu (zakłada istnienie wybranego koordynatora) → [[RSO 04 Algorytmy wzajemnego wykluczania#Podejście scentralizowane]]
- Koordynator bariery w analizie złożoności → [[RSO 08 Czas wirtualny i złożoność algorytmów#Przykład — bariera]]
- Inicjator $Q_\alpha$ w algorytmach migawek i detekcji (również zakładany jako dany) → [[RSO 09 Stan globalny i migawki]], [[RSO 10 Detekcja zakończenia]]
