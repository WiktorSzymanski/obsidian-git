---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 24
---
# 24. Niezawodne zatwierdzanie transakcji rozproszonych
---
> **Transakcja rozproszona** obejmuje operacje wykonywane przez wielu **uczestników** (węzły, bazy danych, zarządców zasobów). **Atomowe zatwierdzanie** (_atomic commitment_) zapewnia, że wszyscy uczestnicy podejmą **tę samą decyzję**: wszyscy zatwierdzą (**COMMIT**) albo wszyscy wycofają (**ABORT**) – także w obecności awarii procesów i kanałów.

Materiał źródłowy: [[Slajdy-FT-06_Commit.pdf]] (s. 1–2 transakcje, 3–5 blokady, 6 atomowe zatwierdzanie, 7–17 2PC, 18–23 3PC, 22 wait-freedom).

## Transakcje i ACID
- **Atomowość** (_Atomicity_) – wszystko albo nic.
- **Spójność** (_Consistency_) – transakcja przeprowadza system z jednego stanu spełniającego ograniczenia integralności do innego.
- **Izolacja** (_Isolation_) – współbieżne transakcje nie widzą swoich stanów pośrednich (efekt jak przy wykonaniu szeregowym – **serializowalność**).
- **Trwałość** (_Durability_) – po zatwierdzeniu zmiany przetrwają awarie (log w pamięci trwałej).

Uzupełnienie notatki [[ACID]].

## Kontrola współbieżności
### Konflikty żądań
Żądania są w **konflikcie**, jeśli dotyczą tego samego obiektu i co najmniej jedno go modyfikuje. Zarządca obiektu przy konflikcie może:
- **WAIT** – nowe żądanie kolejkowane,
- **REJECT** – nowe żądanie odrzucane (jego transakcja wycofywana),
- **PREEMPT** – obsługiwane żądanie anulowane (jego transakcja wycofywana).

### Blokowanie dwufazowe (2PL)
- **Faza wzrostu** – transakcja tylko **zakłada** blokady (współdzielone S dla odczytu, wyłączne X dla zapisu).
- **Faza zmniejszania** – po zwolnieniu pierwszej blokady transakcja **nie może założyć żadnej nowej**.
- **Twierdzenie**: 2PL gwarantuje **serializowalność** (punkt zablokowania wszystkich blokad wyznacza porządek szeregowy).
- **Ścisłe 2PL** (_strict_) – blokady **wyłączne** zwalniane dopiero po COMMIT/ABORT → brak kaskadowych wycofań (nikt nie przeczyta niezatwierdzonych danych).
- **Rygorystyczne 2PL** (_rigorous / strong strict_) – **wszystkie** blokady do końca transakcji.
- **Wady**: możliwe **zakleszczenia** (wykrywanie, timeouty, wait-die / wound-wait – [[06 Zakleszczenie w systemach rozproszonych]]); w systemie rozproszonym 2PL + 2PC → blokady trzymane do zakończenia zatwierdzania.
- Alternatywy: znaczniki czasowe (TO), optymistyczna kontrola współbieżności (walidacja przy zatwierdzeniu), MVCC.

Uzupełnienie notatki [[Systemy Wysokiej Niezawodności/2 Phase Locking]].

## Problem atomowego zatwierdzania
Każdy uczestnik głosuje **TAK** (może zatwierdzić – wszystkie zmiany zapisane w pamięci trwałej) lub **NIE**. Wymagania:
- **AC1 (zgodność)** – wszyscy, którzy podejmą decyzję, podejmują tę samą,
- **AC2** – decyzja jest nieodwołalna,
- **AC3 (ważność)** – COMMIT tylko, jeśli **wszyscy** zagłosowali TAK,
- **AC4** – jeśli brak awarii i wszyscy głosują TAK → decyzja COMMIT,
- **AC5 (terminacja)** – po naprawie awarii i przez wystarczająco długi czas bez nowych awarii wszyscy podejmą decyzję.

**Protokół nieblokujący** (_non-blocking_): poprawne procesy podejmują decyzję **mimo awarii innych**, bez czekania na ich naprawę.

---
## Protokół dwufazowego zatwierdzania (2PC, Gray 1978)
Uczestnicy: **koordynator** $P_0$ (zwykle inicjator transakcji) i **uczestnicy** $P_1 \dots P_n$.

### Przebieg
**Faza 1 – głosowanie (prepare)**
1. Koordynator zapisuje w logu `BEGIN_COMMIT` i wysyła **VOTE_REQUEST (PREPARE)** do wszystkich uczestników → stan **WAIT**.
2. Uczestnik:
   - może zatwierdzić → **wymusza zapis** (_force write_) rekordu `READY` (oraz redo/undo transakcji) do logu, odsyła **VOTE_COMMIT (TAK)** → stan **READY** (_prepared_, w wątpliwości),
   - nie może → zapisuje `ABORT`, odsyła **VOTE_ABORT (NIE)** i jednostronnie wycofuje → **ABORT**.

**Faza 2 – decyzja (commit/abort)**
3. Koordynator:
   - **wszystkie** głosy TAK → zapisuje `GLOBAL_COMMIT` w logu (**punkt zatwierdzenia** – od tej chwili decyzja jest nieodwołalna), wysyła **GLOBAL_COMMIT**,
   - choć jeden NIE lub timeout → zapisuje `GLOBAL_ABORT`, wysyła **GLOBAL_ABORT**.
4. Uczestnik wykonuje decyzję (zatwierdza/wycofuje, zwalnia blokady), zapisuje w logu, odsyła **ACK**.
5. Koordynator po wszystkich ACK zapisuje `END` (można usunąć stan transakcji).

### Automaty stanów
```
Koordynator:  INIT ──(wyślij VOTE_REQUEST)──> WAIT ──(wszystkie TAK)──> COMMIT
                                               └──(NIE lub timeout)──> ABORT

Uczestnik:    INIT ──(VOTE_REQUEST, głos TAK)──> READY ──(GLOBAL_COMMIT)──> COMMIT
                │                                   └───(GLOBAL_ABORT)───> ABORT
                └──(VOTE_REQUEST, głos NIE / timeout)───────────────────> ABORT
```

### Obsługa timeoutów (terminacja)
| Kto | W stanie | Timeout – czeka na | Działanie |
|---|---|---|---|
| uczestnik | **INIT** | VOTE_REQUEST | może **jednostronnie wycofać** (nie głosował) |
| koordynator | **WAIT** | głosy | **GLOBAL_ABORT** |
| koordynator | COMMIT/ABORT | ACK | ponawia wysłanie decyzji |
| uczestnik | **READY** | decyzję | **NIE może** decydować sam (głosował TAK – koordynator mógł już zatwierdzić) → **protokół terminacji kooperacyjnej** |

**Terminacja kooperacyjna**: uczestnik w READY pyta **innych uczestników** $P_j$ o stan:
| Stan $P_j$ | Decyzja pytającego |
|---|---|
| COMMIT | COMMIT |
| ABORT | ABORT |
| INIT | ABORT (koordynator nie mógł zatwierdzić – $P_j$ nie głosował) |
| READY | brak informacji – pytać dalej |

Jeśli **wszyscy** osiągalni uczestnicy są w **READY** → trzeba **czekać** na odtworzenie koordynatora → **2PC jest protokołem blokującym**.

### Odtwarzanie po awarii (na podstawie logu)
- **Koordynator** po restarcie:
  - brak `BEGIN_COMMIT` / decyzji w logu → ABORT (lub ponowne rozpoczęcie),
  - jest `GLOBAL_COMMIT`/`GLOBAL_ABORT`, brak `END` → ponownie rozsyła decyzję.
- **Uczestnik** po restarcie:
  - brak `READY` → ABORT (jednostronnie),
  - jest `READY`, brak decyzji → pyta koordynatora / innych uczestników (stan „w wątpliwości”, blokady z logu odtworzone!),
  - jest `COMMIT`/`ABORT` → redo/undo zgodnie z logiem.

### Problem blokowania
Awaria koordynatora **po** otrzymaniu głosów TAK, a **przed** rozesłaniem decyzji (lub awaria koordynatora i uczestnika, który już dostał decyzję) → pozostali uczestnicy w READY **trzymają blokady** do naprawy koordynatora. Dostępność systemu zależy od koordynatora.

### Koszt i optymalizacje
- $4n$ komunikatów (PREPARE, głos, decyzja, ACK), $2$ wymuszone zapisy u koordynatora, $2$ u każdego uczestnika.
- **Presumed abort** – brak informacji o transakcji w logu koordynatora ⇒ ABORT; nie trzeba wymuszać zapisu ABORT ani zbierać ACK dla ABORT.
- **Presumed commit** – odwrotnie (mniej ACK przy COMMIT, ale wymuszony zapis listy uczestników na starcie).
- **Uczestnik tylko do odczytu** – odpowiada `READ-ONLY` i wyłącza się z fazy 2.
- **Ostatni agent** / **1PC** – gdy jeden uczestnik.

## Protokół trójfazowego zatwierdzania (3PC, Skeen 1981)
Cel: **protokół nieblokujący** przy awariach procesów (fail-stop) w systemie **synchronicznym**, **bez podziału sieci**.

### Warunki nieblokowania (Skeen)
Protokół jest nieblokujący, jeśli w automacie każdego uczestnika:
1. nie istnieje stan, z którego można przejść **bezpośrednio** zarówno do COMMIT, jak i do ABORT (w 2PC tak jest dla READY),
2. nie istnieje stan **niezatwierdzalny** (_non-committable_), z którego można przejść bezpośrednio do COMMIT.

Rozwiązanie: nowy, **zatwierdzalny** stan pośredni **PRECOMMIT** między READY a COMMIT.

### Przebieg
1. **Faza 1 (głosowanie)** – jak w 2PC: VOTE_REQUEST → VOTE_COMMIT/VOTE_ABORT; koordynator → WAIT, uczestnik → READY.
2. **Faza 2 (przygotowanie do zatwierdzenia)**:
   - wszystkie TAK → koordynator wysyła **PREPARE_COMMIT** → stan **PRECOMMIT**; uczestnik przechodzi do **PRECOMMIT** i odsyła **READY_COMMIT (ACK)**,
   - choć jeden NIE / timeout → GLOBAL_ABORT (jak 2PC).
3. **Faza 3 (zatwierdzenie)** – po ACK od uczestników (tych, którzy nie ulegli awarii) koordynator wysyła **GLOBAL_COMMIT** → COMMIT.

```
Koordynator:  INIT → WAIT → PRECOMMIT → COMMIT          (WAIT → ABORT)
Uczestnik:    INIT → READY → PRECOMMIT → COMMIT          (INIT/READY → ABORT)
```

### Timeouty i protokół terminacji
| Kto | Stan | Timeout | Działanie |
|---|---|---|---|
| koordynator | WAIT | głosy | ABORT |
| koordynator | PRECOMMIT | ACK | **COMMIT** (wiadomo, że wszyscy głosowali TAK; uczestnicy, którzy ulegli awarii, dowiedzą się po odtworzeniu) |
| uczestnik | INIT | VOTE_REQUEST | ABORT |
| uczestnik | READY | PREPARE_COMMIT | **protokół terminacji** |
| uczestnik | PRECOMMIT | GLOBAL_COMMIT | **protokół terminacji** |

**Protokół terminacji** (po awarii koordynatora): wybierany jest **nowy koordynator** (elekcja – [[05 Algorytmy elekcji]]), który zbiera stany osiągalnych uczestników:
- ktoś w **COMMIT** → COMMIT,
- ktoś w **ABORT** → ABORT,
- ktoś w **PRECOMMIT** (a nikt w ABORT) → najpierw przeprowadza pozostałych przez PRECOMMIT (PREPARE_COMMIT), potem **COMMIT**,
- wszyscy w **READY** (lub INIT) → **ABORT** (żaden uczestnik nie mógł jeszcze zatwierdzić, bo nikt nie jest w PRECOMMIT).

Kluczowe: **stan uczestnika i koordynatora różni się co najwyżej o jedno przejście**, więc z READY nie da się „przeskoczyć” do COMMIT bez PRECOMMIT u pozostałych.

### Własności i ograniczenia 3PC
- ✔ **nieblokujący** przy awariach procesów – poprawni uczestnicy zawsze kończą.
- ✘ wymaga **systemu synchronicznego** (wiarygodne timeouty, fail-stop).
- ✘ **podział sieci**: jedna partycja (wszyscy w READY) zdecyduje ABORT, druga (ktoś w PRECOMMIT) COMMIT → **niespójność**. Nieblokujące zatwierdzanie przy podziałach jest niemożliwe (wniosek z FLP i problemu dwóch armii).
- ✘ więcej komunikatów ($6n$) i opóźnienie – rzadko stosowany w praktyce.
- **Wait-freedom**: 3PC gwarantuje poprawnym procesom zakończenie w skończonej liczbie kroków niezależnie od awarii innych (przy braku partycji), 2PC – nie.

## 2PC vs 3PC
| | 2PC | 3PC |
|---|---|---|
| Fazy | 2 | 3 |
| Komunikaty | $4n$ | $6n$ |
| Blokowanie | **blokujący** przy awarii koordynatora | **nieblokujący** przy awariach procesów |
| Model | asynchroniczny (bezpieczny) | synchroniczny, bez partycji |
| Podział sieci | bezpieczny (blokuje) | może naruszyć spójność |
| Zastosowanie | powszechny: XA/JTA, bazy rozproszone, WS-AtomicTransaction | głównie teoretyczny |

## Zatwierdzanie odporne na partycje – kworum
- **Paxos Commit** (Gray, Lamport 2004) – decyzja koordynatora uzgadniana konsensusem wśród replik → brak blokowania przy awarii mniejszości.
- **Spanner** (Google) – 2PC, w którym uczestnikami i koordynatorem są **grupy Paxos** (każda partycja danych replikowana).
- **Protokoły kworum** (Skeen): COMMIT wymaga kworum $V_C$, ABORT kworum $V_A$, $V_C + V_A > V$ – w partycji decyzję podejmuje tylko strona z kworum.

## Replikacja z głosowaniem
### Głosowanie statyczne – Gifford
Kworum odczytu $R$ i zapisu $W$ z $W > V/2$ i $R + W > V$ – [[Systemy Wysokiej Niezawodności/Algorytm Gifforda]]. Problem: po awariach lub podziale może zabraknąć kworum mimo działającej większości aktualnych replik.

### Głosowanie dynamiczne – Jajodia-Mutchler
Liczba głosów potrzebnych do kworum **dostosowuje się** do liczby replik uczestniczących w ostatniej aktualizacji. Każda replika przechowuje:
- **VN** – numer wersji (liczba aktualizacji),
- **RU** (_update sites cardinality_, SC) – liczba replik, które uczestniczyły w **ostatniej** aktualizacji,
- **DS** (_distinguished site_) – wyróżniona replika używana do rozstrzygania remisu, gdy RU jest **parzyste**.

**Algorytm aktualizacji** w partycji $P$:
1. Znajdź największy numer wersji $M = \max VN$ w partycji i zbiór $I$ replik z $VN = M$; odczytaj ich $RU = N$.
2. Aktualizacja **dozwolona**, jeśli:
   - $|I| > N/2$ (większość replik z ostatniej aktualizacji), **albo**
   - $|I| = N/2$ i $DS \in I$ (remis rozstrzygany przez wyróżnioną replikę).
3. Po aktualizacji wszystkie repliki w partycji dostają $VN := M+1$, $RU := |P|$ i nowy $DS$ (np. o największym id), gdy $|P|$ parzyste.

- Przykład: 5 replik, aktualizacja z 5 → partycja {A,B,C} aktualizuje (3 > 5/2) → RU=3; potem partycja {A,B} aktualizuje (2 > 3/2) → RU=2; potem {A} może aktualizować tylko jeśli A = DS (1 = 2/2).
- Uwagi: **DS nie istnieje, gdy RU jest nieparzyste** (remis niemożliwy); jeśli partycja zawierająca repliki z ostatniej aktualizacji zostanie rozdzielona tak, że żadna część nie ma większości – **system nie ma prawa postępu** ([[Dynamiczne głosowanie]]).
- **Wersja hybrydowa**: gdy $RU = 3$, algorytm przechodzi na głosowanie statyczne wśród tych 3 replik (większość 2 z 3), co zwiększa dostępność przy małej liczbie replik.
- Dostępność wyższa niż przy głosowaniu statycznym, ale konieczne przechowywanie i uzgadnianie dodatkowych metadanych.

Uzupełnienie notatki [[Systemy Wysokiej Niezawodności/Algorytm Jajodia-Mutchlera]].

## Zobacz też
- [[22 Wsteczne odtwarzanie stanu - uzupełnienie]] – output commit, logi
- [[23 Rozproszone uzgadnianie w środowisku zawodnym]]
- Wzorzec **Saga** (kompensacje zamiast 2PC) i Transactional Outbox – kontekst pracy magisterskiej: [[Transactional Outbox vs Event Sourcing]]
