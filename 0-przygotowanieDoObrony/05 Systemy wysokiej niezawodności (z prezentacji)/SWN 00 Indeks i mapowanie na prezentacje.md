---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
source: "~/Documents/SystemyWysokiejNiezawodności/"
---
# SWN 00. Indeks i mapowanie na prezentacje
---
> Ten katalog to **druga wersja** notatek z Systemów Wysokiej Niezawodności. W odróżnieniu od wersji pierwszej (`0-przygotowanieDoObrony/05 Systemy wysokiej niezawodności/`), która powstała z **listy zagadnień egzaminacyjnych** i wiedzy ogólnej, tutaj **każda informacja pochodzi z oryginalnych prezentacji wykładowych** dr. Michała Szychowiaka (`~/Documents/SystemyWysokiejNiezawodności/`). Przerobione zostało **369 slajdów i stron** z **11 plików** pokrywających zagadnienia **22, 23 i 24**. Materiał z notatek w vaultcie jest wpleciony, ale **zawsze oznaczony** blokiem `> [!note] Uzupełnienie spoza slajdów`, a miejsca, gdzie notatka mówi co innego niż slajd — blokiem `> [!warning] Rozbieżność`.

---
## Mapowanie: zagadnienie egzaminacyjne → notatka

### Zagadnienie 22 — Wsteczne odtwarzanie stanu przetwarzania rozproszonego

| # | Notatka | Źródło | Slajdy | Pokrycie |
|---|---|---|---|---|
| 1 | [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane]] | `Slajdy-FT-02_Recovery1.pdf` | 1–36 | 🟢 **pełne** — awarie, odtwarzanie postępowe/wsteczne, model węzła, UIP/WAL, shadow pages, $CP^\bullet$/$CP^!$, linia odtwarzania, output commit, **Koo-Toueg** |
| 2 | [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów]] | `Slajdy-FT-03_Recovery2.pdf` | 1–45 | 🟢 **pełne** — **Juang-Venkatesan**, logowanie pesymistyczne/optymistyczne, relacja z-dependency, **Wang-Fuchs**, krawędzie rollback, garbage collection (Tw. 1–3) |
| 3 | [[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny]] | `Slajdy-FT-04_Recovery3.pdf` | 1–42 | 🟢 **pełne** — logowanie przyczynowe, **Manetho**, graf poprzedzania, **Manivannan-Singhal** z pełnym algorytmem odtwarzania i dowodem Tw. 1 |
| 4 | [[SWN 04 Algorytmy odtwarzania - opis szczegółowy]] | `FT_Recovery-algorytmy.pdf` | s. 39–56 | 🟢 **pełne** — pseudokod Koo-Touega (bloki S1–S5, S7–S9), Wang-Fuchs, **pełny pseudokod odtwarzania Manetho** |

### Zagadnienie 23 — Problemy rozproszonego uzgadniania w środowisku zawodnym

| # | Notatka | Źródło | Slajdy | Pokrycie |
|---|---|---|---|---|
| 5 | [[SWN 05 Problemy uzgadniania i wyniki niemożliwości]] | `Slajdy-FT-09_Agreement.pdf` | 1–39 | 🟢 **pełne** — coordinated attack + dowód niemożliwości, konsensus / BA / IC i ich redukcje, **FLP'85**, procesy początkowo martwe (algorytm FLP), renaming, zadanie rozproszone |
| 6 | [[SWN 06 Awarie bizantyjskie]] | `Slajdy-FT-10_ByzantineFailures.pdf` | 1–44 | 🟢 **pełne** — niemożliwość dla $f \geqslant \frac{1}{3}N$, *approximate agreement*, **OM** z lematem i twierdzeniem, **SM** z optymalizacją i złożonościami |
| 7 | [[SWN 07 Konsensus]] | `Slajdy-FT-12_Consensus.pdf` 1–45 + `FT_ConsensusFD.pdf` s. 1–2 | 45 + 2 | 🟢 **pełne** — **Paxos**, Monte Carlo / Las Vegas, **Bracha-Toueg**, **Phase-King** z dowodem, CAP; **+ pseudokody Chandry-Touega dla $\mathcal{S}$ i $\Diamond\mathcal{S}$** |
| 8 | [[SWN 08 Detektory awarii i replikacja procesu]] | `Slajdy-FT-11_ProcessReplication.pdf` | 1–44 | 🟢 **pełne** — replikacja aktywna/pasywna, RBcast/UBcast/TOcast/VScast, **TOcast $\cong$ C $\cong$ VScast**, **8 klas detektorów awarii**, redukcje, TRBcast |

### Zagadnienie 24 — Niezawodne zatwierdzanie transakcji rozproszonych

| # | Notatka | Źródło | Slajdy | Pokrycie |
|---|---|---|---|---|
| 9 | [[SWN 09 Transakcje i atomowe zatwierdzanie]] | `Slajdy-FT-06_Commit.pdf` | 1–27 | 🟢 **pełne** — konflikty, 2PL, WAIT-DIE/WOUND-WAIT, **2PC** z automatami i pięcioma punktami awarii, **concurrency set**, **3PC**, non-blocking vs wait-freedom, 3 twierdzenia o niemożliwości |
| 10 | [[SWN 10 Algorytmy głosowania]] | `Slajdy-FT-07_Voting.pdf` | 1–29 | 🟢 **pełne** — **Gifford** ($W \geqslant M$, $R+W>V$), partycjonowanie, CAP, **Jajodia-Mutchler** z prześledzonym przykładem, realokacja głosów (overthrow, alliance) |

---
## Mapowanie odwrotne: prezentacja → notatka

| Plik | Slajdy | Tytuł wykładu | Notatka |
|---|---|---|---|
| `Slajdy-FT-02_Recovery1.pdf` | 36 | Odtwarzanie stanu cz. I | [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane]] |
| `Slajdy-FT-03_Recovery2.pdf` | 45 | Odtwarzanie stanu cz. II | [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów]] |
| `Slajdy-FT-04_Recovery3.pdf` | 42 | Odtwarzanie stanu cz. III | [[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny]] |
| `FT_Recovery-algorytmy.pdf` | 18 s. | Przegląd algorytmów tolerowania uszkodzeń | [[SWN 04 Algorytmy odtwarzania - opis szczegółowy]] |
| `Slajdy-FT-09_Agreement.pdf` | 39 | Agreement | [[SWN 05 Problemy uzgadniania i wyniki niemożliwości]] |
| `Slajdy-FT-10_ByzantineFailures.pdf` | 44 | Byzantine failures | [[SWN 06 Awarie bizantyjskie]] |
| `Slajdy-FT-12_Consensus.pdf` | 45 | Consensus | [[SWN 07 Konsensus]] |
| `FT_ConsensusFD.pdf` | 2 s. | Chandra-Toueg, Figure 5 i 6 | [[SWN 07 Konsensus#Konsensus z detektorami awarii]] |
| `Slajdy-FT-11_ProcessReplication.pdf` | 44 | Process Replication | [[SWN 08 Detektory awarii i replikacja procesu]] |
| `Slajdy-FT-06_Commit.pdf` | 27 | Algorytmy zatwierdzania | [[SWN 09 Transakcje i atomowe zatwierdzanie]] |
| `Slajdy-FT-07_Voting.pdf` | 29 | Algorytmy głosowania | [[SWN 10 Algorytmy głosowania]] |
| **razem** | **369** | | **10 notatek + 89 rysunków** |

**Każdy z 369 slajdów/stron jest objęty odsyłaczem `<sub>` w którejś z notatek — żaden zakres nie został pominięty.**

---
## Rejestr rozbieżności

Miejsca, w których **notatka w vaultcie mówi co innego niż slajd**. Każda pozycja ma wskazany konkretny slajd jako dowód.

| Notatka w vaultcie | Co twierdzi | Co mówi slajd | Dowód |
|---|---|---|---|
| [[Systemy Wysokiej Niezawodności/Algorytm Manivannana-Singhala]] | *garbage collection*: usuwane są punkty kontrolne **przed** tym, do którego proces został przywrócony | `discard all the checkpoints **beyond** cp_i` — czyli te **za** punktem wycofania. Kasowanie punktów **poprzedzających RL** to osobny mechanizm | `FT-04` slajd 26 (beyond) vs slajd 41 (preceding) → [[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny#Garbage collection]] |
| [[Systemy Wysokiej Niezawodności/Algorytm Wanga-Fuchsa]] | przetwarzanie **deterministyczne** (bez zastrzeżenia) | `deterministic processing (**can be relaxed**)` | `FT-03` slajd 21 → [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Model systemu i idea]] |
| `FT_Recovery-algorytmy.pdf` (s. 48) | Wang-Fuchs: „przetwarzanie jest **niedeterministyczne**" | slajd: deterministyczne (można rozluźnić) | rozbieżność **wewnątrz materiałów kursu** → [[SWN 04 Algorytmy odtwarzania - opis szczegółowy#Założenia]] |
| [[Systemy Wysokiej Niezawodności/Algorytm Juanga-Venkatesana]] | dodaje założenie *piecewise deterministic* i logowanie optymistyczne „(PRAWDOPODOBNIE)" | slajd 4 podaje **tylko** kanały niezawodne FIFO o nieskończonej pojemności | `FT-03` slajd 4 → [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Założenia, idea i struktury]] |
| [[Systemy Wysokiej Niezawodności/Algorytm Bracha-Touega]] | (ostrzeżenie z [[Braki w notatkach]]) bywa mylony z algorytmem detekcji zakleszczenia | **potwierdzone**: w SWN Bracha-Toueg to algorytm **konsensusu binarnego typu Las Vegas**; wariant dla detekcji zakleszczenia w prezentacjach **nie występuje** | `FT-12` slajdy 12–31 → [[SWN 07 Konsensus#Algorytm Bracha-Touega — model i idea]] |
| [[Systemy Wysokiej Niezawodności/Wycofanie operacji]] | definiuje **efekt domino** w kontekście *operation-based recovery* | slajd stawia efekt domino w kontekście **kolejnych wycofań przy checkpointingu**, nie logowania operacji | `FT-02` slajd 18 → [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Efekt domino]] |
| [[Systemy Wysokiej Niezawodności/Replikacja Procesu]] | link `[[Reliable Broadcast]]` traktowany jako wiszący (wg [[Braki w notatkach]]) | **plik istnieje** w katalogu głównym vaultu (nieśledzony w gicie); definicja ze slajdu jest jednak pełniejsza (3 warunki + rozróżnienie RBcast/UBcast) | `FT-11` slajdy 14 i 16 → [[SWN 08 Detektory awarii i replikacja procesu#TOcast i RBcast]] |

---
## Odpowiedź na listę kontrolną z [[Braki w notatkach]]

Sekcja „Systemy wysokiej niezawodności" tej notatki wypisuje 17 braków. Stan po przerobieniu prezentacji:

### Zagadnienie 22
| Brak z listy | Stan |
|---|---|
| spójny stan globalny, linia odtwarzania, wiadomości osierocone i zagubione, **efekt domino** | 🟢 **domknięte** — [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Odtwarzanie w systemie rozproszonym]] (definicje formalne wiadomości osieroconej i utraconej, $CP^\bullet$/$CP^!$, RL, $RL^*$) |
| systematyka: CP skoordynowane / nieskoordynowane / wymuszone komunikacją vs logowanie pesymistyczne / optymistyczne / przyczynowe — **tabela** | 🟡 **częściowo** — wszystkie trzy techniki CP opisane ([[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane|I]], [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów|II]], [[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny|III]]) oraz logowanie pesymistyczne i optymistyczne; **logowania przyczynowego jako trybu logowania slajdy nie mają** (jest tylko jako *causal logging* w Manetho) |
| [[Systemy Wysokiej Niezawodności/Checkpoint]] `#TODO`; Manivannan-Singhal — pełen algorytm `#TODO` | 🟢 **domknięte** — definicja CP ze slajdu 12 wykł. I oraz **pełny algorytm QS z dowodem** w [[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny#Pełny algorytm odtwarzania QS]] |
| Koo-Toueg — wyjaśnić „przetwarzanie dyfuzyjne" | 🟢 **domknięte** — mechanizm dyfuzji widoczny wprost w bloku **S4** w [[SWN 04 Algorytmy odtwarzania - opis szczegółowy#Bloki algorytmu tworzenia punktu kontrolnego]] |
| output commit — definicja | 🟢 **domknięte** — [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Output commit]] |

### Zagadnienie 23
| Brak z listy | Stan |
|---|---|
| **definicja konsensusu**: zgodność, ważność, terminacja | 🟢 **domknięte** — [[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Konsensus]] |
| **twierdzenie FLP** | 🟢 **domknięte** (sam wynik + intuicja) — [[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Wynik niemożliwości dla awarii procesów — FLP'85]]; **dowodu brak w prezentacjach** |
| **problem bizantyjskich generałów** — Lamport-Shostak-Pease, OM(m), warunek $N > 3f$ | 🟢 **domknięte** — całe [[SWN 06 Awarie bizantyjskie]] |
| modele awarii (crash, omission, fail-stop, fail-recovery, bizantyjskie) — zebrać | 🟡 **częściowo** — slajd 7 wykł. 09 wymienia tylko *stopping*, *Byzantine* i awarię łącza; pełna klasyfikacja pozostaje w [[23 Rozproszone uzgadnianie w środowisku zawodnym]] |
| **detektory awarii ($\Diamond\mathcal{S}$, $\mathcal{P}$), konsensus z detektorem (Chandra-Toueg)** | 🟢 **domknięte** — [[SWN 08 Detektory awarii i replikacja procesu#Detektory awarii]] (8 klas, redukcje) + pseudokody w [[SWN 07 Konsensus#Konsensus z detektorami awarii]] |
| problem dwóch armii (uzgadnianie przy zawodnych kanałach) | 🟢 **domknięte** — jako *coordinated attack* z pełnym dowodem niemożliwości: [[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Wynik niemożliwości dla awarii łączy]] |
| reliable broadcast — wiszący link | 🟢 **domknięte** — [[SWN 08 Detektory awarii i replikacja procesu#TOcast i RBcast]] i [[SWN 08 Detektory awarii i replikacja procesu#Reliable Broadcast przez dyfuzję komunikatów]] |

### Zagadnienie 24
| Brak z listy | Stan |
|---|---|
| **2PC** — fazy, stany koordynatora/uczestników, obsługa timeoutów, blokowanie przy awarii koordynatora | 🟢 **domknięte** — [[SWN 09 Transakcje i atomowe zatwierdzanie#2PC — dwufazowe zatwierdzanie]] (tabela faz, automaty z przejściami F i T, punkty K1–K3 i P1–P2, concurrency set) |
| **3PC** — stan *pre-commit*, dlaczego nieblokujący, założenia | 🟢 **domknięte** — [[SWN 09 Transakcje i atomowe zatwierdzanie#3PC — trójfazowe zatwierdzanie]] |
| protokoły terminacji i odtwarzania (co robi uczestnik po restarcie) | 🟡 **częściowo** — dla 2PC punkt **P2** (slajd 13); dla **3PC slajdy tego nie podają** (jest tylko zadanie „uzupełnić przejścia automatów") |
| [[ACID]] — same nagłówki | 🔴 **nadal otwarte** — wykład **nie omawia ACID** |
| [[Systemy Wysokiej Niezawodności/2 Phase Locking]] — fazy wzrostu/zmniejszania, strict 2PL | 🔴 **nadal otwarte** — slajd 3 podaje tylko rozwinięcie skrótu i problem zakleszczenia |
| głosowanie dynamiczne, Jajodia-Mutchler (pusty plik) | 🟢 **domknięte** — [[SWN 10 Algorytmy głosowania#Dynamiczne głosowanie]] (protokół, struktury $VN$/$RU$/$DS$, prześledzony przykład na 5 procesach) |

---
## Zbiorcza lista TODO

### 🔴 Nadal otwarte — prezentacje w ogóle tego nie mają
- **Własności ACID** — brak w całym kursie → [[SWN 09 Transakcje i atomowe zatwierdzanie#Braki i uwagi]]
- **Fazy 2PL** (wzrostu / zmniejszania), **strict 2PL** → [[SWN 09 Transakcje i atomowe zatwierdzanie#2PL i zakleszczenie]]
- **Logowanie przyczynowe** jako trzeci tryb systematyki Elnozahy'ego → [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Braki i uwagi]]
- **Pełna klasyfikacja modeli awarii** (omission, timing, bizantyjskie z uwierzytelnianiem) → [[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Powiązania i braki]]
- **Algorytmy kworum oparte na strukturze** (siatka, drzewo, $\sqrt{N}$ Maekawy) → [[SWN 10 Algorytmy głosowania#Braki i uwagi]]
- **Algorytmy implementacji detektorów awarii** (heartbeat, ping-ack, adaptacyjne timeouty) → [[SWN 08 Detektory awarii i replikacja procesu#Powiązania i braki]]

### 🟡 Brakujące dowody i analizy złożoności
| Czego brakuje | Dotyczy | Notatka |
|---|---|---|
| dowód twierdzenia **FLP** | konsensus | [[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Powiązania i braki]] |
| dowód poprawności algorytmu **SM** | BA z podpisami | [[SWN 06 Awarie bizantyjskie#Powiązania i braki]] |
| dowód dolnego ograniczenia **$f+1$ rund** | BA | [[SWN 06 Awarie bizantyjskie#Powiązania i braki]] |
| pierwsza część dowodu Tw. 1 (Bracha-Toueg) | konsensus | [[SWN 07 Konsensus#Braki i uwagi]] |
| dowody niemożliwości Monte Carlo / Las Vegas | konsensus | [[SWN 07 Konsensus#Braki i uwagi]] |
| dowody twierdzeń o **TRBcast** | detektory awarii | [[SWN 08 Detektory awarii i replikacja procesu#Powiązania i braki]] |
| dowód, że **$\Diamond\mathcal{W}$ jest najsłabszym FD** dla konsensusu | detektory awarii | [[SWN 08 Detektory awarii i replikacja procesu#Powiązania i braki]] |
| złożoność **Koo-Touega**, **Wanga-Fuchsa**, **Manetho**, **Manivannana-Singhala** | odtwarzanie | [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Braki i uwagi]], [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Braki i uwagi]], [[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny#Braki i uwagi]] |
| złożoność **Paxosa**, **Brachy-Touega**, **Phase-Kinga** | konsensus | [[SWN 07 Konsensus#Braki i uwagi]] |
| złożoność **Gifforda** i **Jajodii-Mutchlera** | głosowanie | [[SWN 10 Algorytmy głosowania#Braki i uwagi]] |

### 🟡 Zadania i pytania bez odpowiedzi na slajdach
| Pytanie / zadanie | Slajd | Notatka |
|---|---|---|
| „jaki problem tu występuje?" przy $K>1$ | `FT-02` 23 | [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Proste ustanawianie $CP^\bullet$ dla operacji atomowych]] |
| „dokąd wycofujemy stan przetwarzania?" | `FT-02` 18 | [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Efekt domino]] |
| „nadmiar?" (logowanie po obu stronach) | `FT-03` 13 | [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Rejestrowanie po obu stronach]] |
| „gdzie jest RL w najgorszym przypadku?", „dlaczego nie rozszerzony graf?" | `FT-03` 39–41 | [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Punkty przeterminowane]] |
| „czy te transformacje działają przy awariach?" | `FT-09` 17 | [[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Relacje między problemami]] |
| **Uzupełnić przejścia automatów 3PC** | `FT-06` 24 | [[SWN 09 Transakcje i atomowe zatwierdzanie#Rola stanu $p$]] |
| „którą własność spełnia 2PC, a którą 3PC?" | `FT-06` 25 | [[SWN 09 Transakcje i atomowe zatwierdzanie#Non-blocking i wait-freedom]] |

### 🟡 Pojęcia użyte bez definicji
- **knot** w grafie skierowanym — używane od slajdu 27 wykł. 09 → [[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Powiązania i braki]]
- **procedura `compute_knot()`** — na slajdzie 25 jest samo wywołanie → tamże
- **przetwarzanie dyfuzyjne** — slajd 28 wykł. I; mechanizm dopiero w [[SWN 04 Algorytmy odtwarzania - opis szczegółowy#Bloki algorytmu tworzenia punktu kontrolnego]]
- **wariant $RU_i = 3$** protokołu Jajodii-Mutchlera — jawnie pominięty przypisem → [[SWN 10 Algorytmy głosowania#Założenia i struktury]]

---
## Materiał nieprzerobiony

Cztery prezentacje z katalogu **nie mapują się na zagadnienia 22–24** i zostały świadomie pominięte (decyzja uzgodniona przed przystąpieniem do pracy):

| Plik | Slajdy | Zawartość |
|---|---|---|
| `Slajdy-FT-00_HighAvailability.pdf` | 55 | Niezawodność, wysoka dostępność, tolerowanie uszkodzeń jako mapa kursu; **redundancja** — *redundancy at a glance*, redundancja połączeń, niezawodność przez replikację |
| `Slajdy-FT-01_Wprowadzenie.pdf` | 47 | **Wiarygodność** (niezawodny / dyspozycyjny / bezpieczny), definicje awarii, błędu i defektu, **naruszenia niezawodności**, **modele systemu**, automat I/O, **kanały komunikacyjne**, odporność na awarie, wysoka dostępność, **CAP conjecture**, zakres materiału |
| `Slajdy-FT-05_rDSM.pdf` | 41 | Rozproszona pamięć współdzielona, **backward recovery w DSM**, podejścia do odtwarzania w DSM, *lazy release consistency*, **spójność atomowa** |
| `Slajdy-FT-13_Stabilization.pdf` | 35 | **Samostabilizacja** — gwarancja, że system niezależnie od bieżącego stanu w skończonym czasie wraca do stanu poprawnego |

> [!important] Co z tego jest naprawdę potrzebne
> **`FT-01` zawiera modele systemu, definicje awarii/błędu/defektu, kanały komunikacyjne i CAP** — czyli podbudowę pojęciową dla zagadnień 22–24. Jeśli którakolwiek z prezentacji miałaby zostać dorobiona jako kolejna notatka, to **właśnie ta**. `FT-05` ma sekcję *backward recovery* dotykającą zagadnienia 22, ale w kontekście DSM, którego nie ma w liście zagadnień.

---
## Dwie warstwy materiału

- **`SWN 01`–`SWN 10`** — warstwa **do nauki**: pełne definicje, dowody, pseudokody, wycinki slajdów.
- **`skompresowane/`** — warstwa **do powtórki**: jeden plik na zagadnienie, same pojęcia z notacją, bez przebiegu algorytmów. Algorytmy są tam wyłącznie **nazwane** (z informacją, jaki problem rozwiązują) i zlinkowane tutaj:
  [[SWN 22 Wsteczne odtwarzanie stanu]] · [[SWN 23 Rozproszone uzgadnianie w środowisku zawodnym]] · [[SWN 24 Niezawodne zatwierdzanie transakcji]]

---
## Konwencje przyjęte w tym katalogu

- **Prefiks `SWN`** w nazwach plików — żeby nie kolidować z wikilinkami do wersji pierwszej, która jest linkowana z [[Mapa zagadnień]].
- **Odsyłacz do slajdu** pod każdym nagłówkiem: `<sub>Slajdy-FT-03_Recovery2.pdf, slajdy 12–15</sub>` — numeracja **slajdów** (każda strona PDF = 1 slajd; slajdy powtarzające się to animacje, oznaczone zakresem).
- **Rysunki** w podkatalogu `assets/`, nazwane `swn-ft<nr pliku>-s<nr slajdu>-<opis>.png`, osadzane przez `![[...]]` z podpisem `<sub>`. **89 wycinków**, każdy z dokładnym wskazaniem slajdu.
- **Definicje i twierdzenia** w blokach cytatu, wzory w LaTeX-u, pseudokody w blokach ` ``` `.
- **Braki** oznaczone blokiem `> [!todo]` z opisem, co dokładnie sprawdzono.
- **Materiał z notatek w vaultcie** — wyłącznie w blokach `> [!note] Uzupełnienie spoza slajdów` z linkiem do pliku źródłowego.
- **Rozbieżności** — w blokach `> [!warning] Rozbieżność z notatką w vaultcie`, zebrane w rejestrze wyżej.
- **Nic spoza slajdów** nie jest opisane jako treść wykładu.

> [!warning] Uszkodzone warstwy tekstowe
> `Slajdy-FT-03_Recovery2.pdf` ma dublowane litery i rozrywane słowa (`MES SSAGE LO OGGING`), `Slajdy-FT-11` gubi symbole (`◇S`), a `FT_Recovery-algorytmy.pdf` ma część runów przesuniętą o **+29 w ASCII** i utracone polskie znaki diakrytyczne. **Cały ten katalog powstał z odczytu obrazów stron**, nie z warstwy tekstowej — dlatego treść jest wierna temu, co widać na slajdzie.

---
## Wersja pierwsza

Katalog `0-przygotowanieDoObrony/05 Systemy wysokiej niezawodności/` — 3 notatki napisane z listy zagadnień egzaminacyjnych:
- [[22 Wsteczne odtwarzanie stanu - uzupełnienie]]
- [[23 Rozproszone uzgadnianie w środowisku zawodnym]]
- [[24 Niezawodne zatwierdzanie transakcji rozproszonych]]

**Zawiera materiał, którego nie ma w prezentacjach** (pełna klasyfikacja modeli awarii, jednolita zgodność, częściowa synchronia / GST, ACID, 2PL w szczegółach). **Obie wersje warto czytać razem**: tę dla zgodności z wykładem, tamtą dla uzupełnienia luk pojęciowych.

Notatki szczegółowe o algorytmach w katalogu `Systemy Wysokiej Niezawodności/` vaultu pozostają **nienaruszone** — ten katalog tylko je linkuje i cytuje.
