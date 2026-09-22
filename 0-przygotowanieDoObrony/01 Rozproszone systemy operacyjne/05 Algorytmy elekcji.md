---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 5
---
# 5. Algorytmy elekcji
---
> **Elekcja** (_leader election_) to wybór spośród procesów **jednego koordynatora (lidera)**, uznawanego przez wszystkie poprawne procesy. Typowo wybierany jest działający proces o **największym identyfikatorze**. Elekcję uruchamia się na starcie systemu lub po wykryciu awarii dotychczasowego koordynatora.

Zastosowania: koordynator w scentralizowanym wykluczaniu, sekwencer rozgłaszania totalnego, regeneracja żetonu, koordynator 2PC/3PC, lider w Paxos/Raft.

## Wymagania
- **Bezpieczeństwo** – każdy proces uczestniczący ma $elected = \bot$ lub $elected = P$, gdzie $P$ to ten sam, działający proces o największym id.
- **Żywotność** – wszystkie poprawne procesy w końcu ustalą $elected \neq \bot$.

Założenia: procesy mają **unikalne, porównywalne** identyfikatory; wiele procesów może uruchomić elekcję jednocześnie.

---
## Algorytm tyrana (_Bully_, Garcia-Molina, 1982)
**Założenia**: system **synchroniczny** (znane ograniczenia opóźnień → timeouty jako detektor awarii), każdy proces zna id wszystkich i może komunikować się z każdym, kanały niezawodne.

**Komunikaty**: $ELECTION$, $ANSWER$ (OK), $COORDINATOR$.

1. Proces $P$ wykrywa awarię koordynatora (brak odpowiedzi w czasie $T$) → wysyła $ELECTION$ do wszystkich procesów o **wyższym id**.
2. Jeśli nikt nie odpowie w czasie $T$ → $P$ zostaje koordynatorem, wysyła $COORDINATOR$ do wszystkich o niższym id.
3. Proces o wyższym id po odebraniu $ELECTION$ odsyła $ANSWER$ i **sam rozpoczyna elekcję** (jeśli jeszcze nie prowadzi).
4. Jeśli $P$ otrzymał $ANSWER$, czeka na $COORDINATOR$; gdy ten nie nadejdzie w czasie $T'$ → rozpoczyna elekcję od nowa.
5. Proces odtworzony po awarii rozpoczyna elekcję. Jeśli ma największe id, **„przejmuje władzę”** mimo działającego koordynatora – stąd nazwa.

- Komunikaty: najlepszy przypadek $N-2$ (awarię wykrył proces z drugim największym id), najgorszy $O(N^2)$ (wykrył proces z najmniejszym id).
- Wada: poprawność zależy od timeoutów – przy fałszywym podejrzeniu możliwych dwóch koordynatorów.

## Algorytm pierścieniowy Chang-Roberts (1979)
**Założenia**: procesy w **logicznym pierścieniu jednokierunkowym**, każdy zna tylko następnika, brak awarii w trakcie elekcji.

Każdy proces ma flagę $participant$ (początkowo $false$).
1. Inicjator: $participant := true$, wysyła $ELECTION(id)$ do następnika.
2. Odbiór $ELECTION(j)$ przez $P_i$:
   - $j > i$ → przekazuje $ELECTION(j)$ dalej, $participant := true$,
   - $j < i$ i $\neg participant$ → zastępuje własnym id: wysyła $ELECTION(i)$, $participant := true$,
   - $j < i$ i $participant$ → **odrzuca** komunikat (już przesłał większy lub równy),
   - $j = i$ → $P_i$ ma największe id: zostaje liderem, $participant := false$, wysyła $ELECTED(i)$.
3. Odbiór $ELECTED(j)$: $elected := j$, $participant := false$; jeśli $j \neq i$ przekazuje dalej.

- Komunikaty: najgorszy $O(N^2)$ (id malejące zgodnie z kierunkiem pierścienia), średni $O(N \log N)$, najlepszy $3N-1$.

## Algorytm LeLanna (1977)
Każdy inicjator wysyła komunikat ze swoim id wokół pierścienia (wszystkie komunikaty obiegają pełne koło, kanały FIFO). Proces, do którego wraca własny komunikat, zna już id wszystkich inicjatorów i wybiera największe.
Komunikaty $O(N^2)$ zawsze.

## Algorytm Hirschberga-Sinclaira (1980)
**Pierścień dwukierunkowy**. Działa w fazach $k = 0, 1, 2, \dots$: każdy aktywny proces wysyła sondę w **obie strony** na odległość $2^k$. Sonda jest odrzucana przez proces o większym id; jeśli wróci z obu stron, proces przechodzi do fazy $k+1$. Proces, którego sonda obiegnie cały pierścień, zostaje liderem.
Komunikaty $O(N \log N)$ w najgorszym przypadku – asymptotycznie optymalne dla pierścieni porównujących id.

## Pierścień z awariami (wariant Tanenbauma)
Komunikat $ELECTION$ gromadzi **listę id** procesów, przez które przeszedł. Jeśli następnik nie odpowiada, jest pomijany (proces zna dalszych następników). Po powrocie do inicjatora wybierany jest największy id z listy i rozsyłany $COORDINATOR(lista)$.

## Elekcja w dowolnej topologii – fala z wygaszaniem (_echo with extinction_)
Każdy inicjator uruchamia algorytm **echa** (fali) oznaczony swoim id. Proces uczestniczy tylko w fali o największym znanym id, a fale mniejsze **wygasza** (ignoruje). Tylko fala największego inicjatora dotrze z powrotem do źródła i zostaje on liderem. Koszt $O(N \cdot |E|)$.

## Elekcja z losowością – Raft (2014)
- Czasy **kadencji** (_term_), procesy w stanach follower / candidate / leader.
- Follower bez heartbeatu przez **losowy** timeout (np. 150–300 ms) zostaje kandydatem: zwiększa term, głosuje na siebie, wysyła $RequestVote$.
- Każdy proces głosuje w danym term co najwyżej raz (na kandydata z log-iem nie starszym niż własny).
- **Większość** głosów → lider, wysyła heartbeaty. Losowe timeouty minimalizują podział głosów.

## Elekcja a system asynchroniczny
- W systemie asynchronicznym z awariami elekcja jest nierozwiązywalna deterministycznie. Wiarygodna elekcja byłaby równoważna doskonałemu detektorowi awarii, a konsensus jest niemożliwy (FLP) – zob. [[23 Rozproszone uzgadnianie w środowisku zawodnym]].
- Stąd praktyczne algorytmy (Bully, Raft) zakładają **częściową synchroniczność** (timeouty) i dopuszczają chwilowo więcej niż jednego „lidera”. Bezpieczeństwo zapewnia wtedy mechanizm kadencji / numerów propozycji (Raft, [[Systemy Wysokiej Niezawodności/Algorytm Paxos|Paxos]]).

## Porównanie
| Algorytm | Topologia | Model | Komunikaty (najgorszy) |
|---|---|---|---|
| Bully | pełny graf | synchroniczny, awarie crash | $O(N^2)$ |
| Chang-Roberts | pierścień jednokier. | bez awarii | $O(N^2)$, średnio $O(N\log N)$ |
| LeLann | pierścień jednokier. | FIFO, bez awarii | $O(N^2)$ |
| Hirschberg-Sinclair | pierścień dwukier. | bez awarii | $O(N\log N)$ |
| Echo z wygaszaniem | dowolna | bez awarii | $O(N\cdot\lvert E\rvert)$ |
| Raft | pełny graf | częściowo synchroniczny | losowe, większość |
