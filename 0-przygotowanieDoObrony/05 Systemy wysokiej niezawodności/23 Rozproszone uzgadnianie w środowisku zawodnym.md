---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 23
---
# 23. Problemy rozproszonego uzgadniania w środowisku zawodnym
---
> **Rozproszone uzgadnianie** to grupa problemów, w których procesy muszą **wspólnie podjąć jedną decyzję** (wartość, lidera, porządek, zatwierdzenie transakcji), mimo że część procesów ulega awariom, a kanały mogą gubić lub opóźniać komunikaty. Podstawowy problem to **konsensus**, do którego sprowadzają się inne: rozgłaszanie totalne, zatwierdzanie atomowe, elekcja, członkostwo grupy, replikacja maszyn stanów.

Algorytmy szczegółowo: [[Systemy Wysokiej Niezawodności/Algorytm Paxos]], [[Systemy Wysokiej Niezawodności/Algorytm Phase-King]], [[Systemy Wysokiej Niezawodności/Algorytm Bracha-Touega]].

## Model systemu
### Synchroniczność
- **System synchroniczny** $S^{Sync}$ – znane górne ograniczenia czasu przetwarzania kroku, opóźnienia komunikatów i dryfu zegarów → wykonanie w **rundach**; awarię można wykryć timeoutem.
- **System asynchroniczny** $S^{Async}$ – brak jakichkolwiek ograniczeń czasowych → nie da się odróżnić procesu, który uległ awarii, od bardzo wolnego.
- **Częściowo synchroniczny** – ograniczenia istnieją, ale nie są znane, albo obowiązują dopiero od pewnego (nieznanego) momentu (GST – _global stabilization time_).

### Modele awarii procesów (od najłagodniejszego)
| Model | Opis |
|---|---|
| **fail-stop** | proces zatrzymuje się, a inne procesy **wiarygodnie wykrywają** awarię |
| **crash** (awaria zatrzymania) | proces zatrzymuje się na zawsze, bez gwarancji wykrycia |
| **fail-recovery** (crash-recovery) | proces zatrzymuje się i może zostać wznowiony (z pamięcią trwałą lub amnezją – [[Awarie]]) |
| **omission** (zaniechania) | proces lub kanał **gubi** wysyłane/odbierane komunikaty (_send/receive omission_) |
| **timing / performance** | odpowiedź poza dozwolonym przedziałem czasu (tylko w systemach synchronicznych) |
| **bizantyjskie** (arbitralne) | proces zachowuje się dowolnie: wysyła sprzeczne lub fałszywe wartości, zmawia się z innymi (błąd oprogramowania, atak); **fail-stop nie jest błędem bizantyjskim** ([[Systemy Wysokiej Niezawodności/Błąd Bizantyjski]]) |
| bizantyjskie z **uwierzytelnianiem** | jak wyżej, ale nie może podrobić podpisu poprawnego procesu |

### Kanały
niezawodne (każdy komunikat dostarczony dokładnie raz, bez modyfikacji), zawodne z utratą (_fair-lossy_), z duplikacją, bez uporządkowania / FIFO.

## Problem konsensusu
Każdy proces $P_i$ proponuje wartość $v_i$ i ma **zdecydować** o wartości $d_i$.
- **Zgodność** (_agreement_) – żadne dwa **poprawne** procesy nie decydują o różnych wartościach.
- **Ważność** (_validity_, integralność) – jeśli wszystkie procesy zaproponowały $v$, to decyzja to $v$ (wersja silniejsza: decyzja jest jedną z zaproponowanych wartości).
- **Terminacja** (_termination_) – każdy poprawny proces w końcu podejmuje decyzję.
- **Jednolita zgodność** (_uniform agreement_) – żadne dwa procesy (także takie, które później ulegną awarii) nie decydują różnie.

**Warianty**:
- **konsensus binarny** ($v \in \{0,1\}$),
- **spójność interaktywna** (_interactive consistency_) – uzgodnienie **wektora** wartości wszystkich procesów,
- **porozumienie bizantyjskie** (_Byzantine agreement_) – jeden nadawca (generał) i odbiorcy,
- **k-uzgodnienie** (_k-set agreement_) – co najwyżej $k$ różnych decyzji.

## Problem dwóch armii (dwóch generałów)
Dwie armie muszą **jednocześnie** zaatakować, a porozumiewają się posłańcami przez teren wroga (**zawodny kanał** – komunikat może zginąć), przy **poprawnych** procesach.
- Każde potwierdzenie wymaga potwierdzenia potwierdzenia… Ostatni komunikat może zginąć, więc jego nadawca nie może być pewny, że druga strona zaatakuje.
- **Twierdzenie**: nie istnieje deterministyczny protokół o skończonej liczbie komunikatów, który gwarantuje uzgodnienie przy możliwości utraty dowolnego komunikatu (dowód przez indukcję: usunięcie ostatniego komunikatu nie może zmieniać decyzji → w końcu protokół bez komunikatów).
- **Wniosek praktyczny**: w TCP (FIN/ACK) czy 2PC nie da się uzyskać **wspólnej wiedzy**, tylko prawdopodobieństwo lub timeouty.

## Problem bizantyjskich generałów (Lamport, Shostak, Pease, 1982)
**Generał-dowódca** wysyła rozkaz (atak/odwrót) do $n-1$ **poruczników**; część uczestników (w tym być może dowódca) to **zdrajcy** (błędy bizantyjskie). Kanały niezawodne, system synchroniczny, odbiorca zna nadawcę.

**Warunki**:
- **IC1** – wszyscy lojalni porucznicy wykonują **ten sam** rozkaz,
- **IC2** – jeśli dowódca jest lojalny, każdy lojalny porucznik wykonuje **jego** rozkaz.

### Niemożliwość dla 3 generałów i 1 zdrajcy
- Dowódca lojalny wysyła „atak”, zdrajca $L_2$ mówi $L_1$, że dostał „odwrót”.
- Dowódca-zdrajca wysyła $L_1$ „atak”, a $L_2$ „odwrót”; $L_2$ uczciwie przekazuje „odwrót”.
- $L_1$ widzi w obu przypadkach to samo, ale IC2 wymaga ataku w pierwszym przypadku, a IC1 zgodności z $L_2$ w drugim → sprzeczność.
- **Twierdzenie**: z komunikatami ustnymi (niepodpisanymi) problem jest rozwiązywalny **wtedy i tylko wtedy, gdy** $n \geq 3m + 1$ ($m$ – liczba zdrajców).

### Algorytm OM(m) – komunikaty ustne, $n \geq 3m+1$
- **OM(0)**:
  1. dowódca wysyła swoją wartość każdemu porucznikowi,
  2. porucznik używa otrzymanej wartości (lub domyślnej _odwrót_, gdy nic nie dostał).
- **OM(m)**, $m > 0$:
  1. dowódca wysyła wartość $v$ każdemu porucznikowi,
  2. każdy porucznik $i$ z otrzymaną wartością $v_i$ działa jako **dowódca** w **OM(m−1)**, rozsyłając $v_i$ do pozostałych $n-2$ poruczników,
  3. porucznik $i$ wybiera $majority(v_1, \dots, v_{n-1})$, gdzie $v_j$ to wartość uzyskana od $j$ w kroku 2 (a $v_i$ – od dowódcy).
- $m+1$ rund, złożoność komunikacyjna **wykładnicza** $O(n^{m+1})$.

### Algorytm SM(m) – komunikaty podpisane
Podpis uniemożliwia zdrajcy zmianę treści komunikatu lojalnego generała: porucznicy przekazują podpisane łańcuchy, a sprzeczne rozkazy od dowódcy są wykrywane. Rozwiązanie istnieje dla **dowolnego** $n \geq m + 2$.

### Inne rozwiązania
- **Phase-King** – $f < N/4$, $f+1$ faz, wielomianowy ([[Systemy Wysokiej Niezawodności/Algorytm Phase-King]]),
- **Phase-Queen** – $f < N/4$,
- **PBFT** (Castro-Liskov, 1999) – praktyczne, częściowo synchroniczne, $N \geq 3f+1$, fazy pre-prepare/prepare/commit, lider i zmiana widoku; podstawa blockchainów permissioned ([[54 Protokół Blockchain]]).

## Konsensus w systemie synchronicznym z awariami crash
**FloodSet**: każdy proces w każdej z $f+1$ rund rozsyła zbiór wszystkich znanych wartości; po $f+1$ rundach decyduje np. o minimum zbioru.
- Poprawność: w $f+1$ rundach jest co najmniej jedna runda bez awarii → wszystkie zbiory się wyrównują.
- **Dolna granica**: każdy algorytm konsensusu tolerujący $f$ awarii crash w systemie synchronicznym potrzebuje w najgorszym przypadku **$f+1$ rund**.

## Twierdzenie FLP (Fischer, Lynch, Paterson, 1985)
> W systemie **asynchronicznym** z niezawodnymi kanałami **nie istnieje deterministyczny** algorytm konsensusu, który zawsze kończy działanie, jeśli **choć jeden** proces może ulec awarii zatrzymania.

- Intuicja: istnieją konfiguracje **dwuwartościowe** (_bivalent_), z których możliwe są obie decyzje. Przeciwnik, opóźniając odpowiednie komunikaty (nieodróżnialne od awarii), może zawsze utrzymywać system w konfiguracji dwuwartościowej → brak terminacji.
- **Obejścia**:
  - **randomizacja** – terminacja z prawdopodobieństwem 1 (Ben-Or, [[Systemy Wysokiej Niezawodności/Algorytm Bracha-Touega|Bracha-Toueg]] – algorytm Las Vegas),
  - **częściowa synchroniczność** – bezpieczeństwo zawsze, terminacja po ustabilizowaniu (Paxos, Raft, PBFT),
  - **detektory awarii** (Chandra-Toueg),
  - słabsze problemy (k-set agreement, konsensus aproksymacyjny).

## Detektory awarii (Chandra, Toueg, 1996)
Moduł lokalny każdego procesu zwracający listę **podejrzewanych** procesów (może się mylić).

**Własności**:
- **Zupełność** (_completeness_):
  - **silna** – każdy proces, który uległ awarii, jest w końcu na stałe podejrzewany przez **każdy** poprawny proces,
  - **słaba** – przez **jakiś** poprawny proces.
- **Dokładność** (_accuracy_):
  - **silna** – żaden poprawny proces nie jest nigdy podejrzewany,
  - **słaba** – **jakiś** poprawny proces nie jest nigdy podejrzewany,
  - **ostatecznie silna / ostatecznie słaba** (◇) – warunek obowiązuje od pewnego momentu.

**Klasy** (przy silnej zupełności):
| | silna dokładność | słaba dokładność | ◇ silna | ◇ słaba |
|---|---|---|---|---|
| silna zupełność | **P** (doskonały) | **S** (silny) | **◇P** | **◇S** |

- Słaba zupełność da się przekształcić w silną (rozgłaszanie list podejrzanych).
- **◇S jest najsłabszym detektorem pozwalającym rozwiązać konsensus** w systemie asynchronicznym z $f < N/2$ (Chandra, Hadzilacos, Toueg).
- **P** (w praktyce system synchroniczny) pozwala tolerować $f < N$ awarii.
- Implementacja: heartbeaty z adaptacyjnymi timeoutami (zwiększanymi po każdym fałszywym podejrzeniu → ◇P przy częściowej synchroniczności), detektor φ-accrual (Cassandra).

### Algorytm Chandra-Toueg z ◇S ($f < N/2$)
Rotujący koordynator: w rundzie $r$ koordynatorem jest $c = (r \bmod N) + 1$. Każdy proces ma **oszacowanie** $est_i$ i znacznik $ts_i$ (runda ostatniej aktualizacji).
1. **Faza 1**: każdy wysyła do koordynatora $(est_i, ts_i)$.
2. **Faza 2**: koordynator czeka na $\lceil (N+1)/2 \rceil$ oszacowań, wybiera to z **największym** $ts$ i rozsyła jako propozycję.
3. **Faza 3**: każdy proces czeka na propozycję **lub** zaczyna podejrzewać koordynatora (◇S): propozycja → $est_i := v$, $ts_i := r$, odsyła **ACK**; podejrzenie → **NACK**.
4. **Faza 4**: koordynator czeka na większość odpowiedzi; jeśli wszystkie to ACK → **decyzja** $v$ rozgłaszana przez **Reliable Broadcast** ([[01 Komunikacja grupowa#Rozgłaszanie niezawodne (_Reliable Broadcast_)]]); w przeciwnym razie kolejna runda.

- **Bezpieczeństwo** zawsze (przecięcie większości gwarantuje, że zablokowana wartość z najwyższym $ts$ zostanie wybrana).
- **Terminacja** – ◇S gwarantuje, że w końcu istnieje koordynator, który nie jest podejrzewany, więc runda się powiedzie.

## Równoważności
- **Konsensus ⇔ rozgłaszanie totalne (atomowe)** – z konsensusu budujemy kolejne „partie” dostarczania; z rozgłaszania totalnego – decyzja = pierwsza dostarczona propozycja.
- **Replikacja maszyny stanów** (_state machine replication_) = rozgłaszanie totalne poleceń + deterministyczne repliki ([[Systemy Wysokiej Niezawodności/Replikacja Procesu]]).
- **Zatwierdzanie atomowe** (_non-blocking atomic commit_) jest pokrewne, ale **różne** od konsensusu: decyzja COMMIT jest możliwa tylko, gdy **wszyscy** zagłosowali TAK – [[24 Niezawodne zatwierdzanie transakcji rozproszonych]].

## Tolerowane awarie – podsumowanie
| Model | System | Warunek |
|---|---|---|
| crash | synchroniczny | $f < N$ (np. FloodSet, $f+1$ rund) |
| crash | asynchroniczny | niemożliwe deterministycznie (FLP) |
| crash | asynchr. + ◇S / częściowo synchr. | $f < N/2$ (Chandra-Toueg, Paxos, Raft) |
| crash | asynchr. + losowość | $f < N/2$ (Ben-Or, Bracha-Toueg) |
| bizantyjskie, bez podpisów | synchroniczny | $N \geq 3f + 1$ (OM, Phase-King: $N > 4f$) |
| bizantyjskie, z podpisami | synchroniczny | $N \geq f + 2$ (SM) |
| bizantyjskie | częściowo synchroniczny | $N \geq 3f + 1$ (PBFT) |
