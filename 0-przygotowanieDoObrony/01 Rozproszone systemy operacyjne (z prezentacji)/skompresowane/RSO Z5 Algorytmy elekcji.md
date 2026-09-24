---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 5
---
# 5. Algorytmy elekcji
---
> [!warning] Zagadnienie nieobecne w prezentacjach
> Cała treść tej notatki pochodzi **spoza slajdów** — z wersji pierwszej [[05 Algorytmy elekcji]] i notatek vaultu. W siedmiu prezentacjach (427 slajdów) elekcja **nie występuje ani razu**; koordynator pojawia się w nich wyłącznie jako **już wybrany**. Wynik weryfikacji fraz: [[RSO 05 Algorytmy elekcji|RSO 05]].

## Problem

**Elekcja** (*leader election*) to wybór spośród procesów **jednego koordynatora**, uznawanego za takiego przez wszystkie procesy poprawne. Typowo wybiera się działający proces o **największym identyfikatorze**. Elekcję uruchamia się na starcie systemu albo po wykryciu awarii dotychczasowego koordynatora.

Elekcja jest cegłą, na której stoi zaskakująco dużo innych mechanizmów:

- koordynator w [[RSO Z4 Algorytmy wzajemnego wykluczania#Podejście scentralizowane|scentralizowanym wzajemnym wykluczaniu]],
- sekwencer w [[RSO Z1 Komunikacja grupowa#Porządek globalny|rozgłaszaniu totalnym]],
- regeneracja utraconego żetonu,
- koordynator protokołów 2PC i 3PC,
- lider w Paxosie i Rafcie.

**Wymagania** są dwa:

- **Bezpieczeństwo** — każdy uczestniczący proces ma $elected = \bot$ albo $elected = P$, gdzie $P$ to **ten sam** działający proces o największym identyfikatorze.
- **Żywotność** — wszystkie procesy poprawne ostatecznie ustalą $elected \neq \bot$.

**Założenia**:

- procesy mają **unikalne, porównywalne identyfikatory**,
- elekcję może rozpocząć **jednocześnie wiele procesów** — algorytm musi więc znosić współbieżnych inicjatorów.

## Algorytm tyrana (Bully)

García-Molina, 1982. Rozwiązuje elekcję w **grafie pełnym** przy założeniu systemu **synchronicznego**: znane ograniczenia opóźnień pozwalają użyć **timeoutów jako detektora awarii**. Każdy proces zna identyfikatory wszystkich i może się z każdym komunikować.

Idea: proces, który wykrył awarię koordynatora, zaczepia **wyłącznie procesy o wyższych identyfikatorach**. Jeśli żaden nie odpowie, sam zostaje koordynatorem i ogłasza to niżej; jeśli któryś odpowie, ten przejmuje elekcję i powtarza to samo wyżej. Proces **odtworzony po awarii** rozpoczyna elekcję i — mając największy identyfikator — **przejmuje władzę mimo działającego koordynatora**; stąd nazwa algorytmu.

Koszt: $N-2$ komunikatów w najlepszym przypadku (awarię wykrył proces o drugim największym identyfikatorze), $O(N^2)$ w najgorszym (wykrył proces o najmniejszym). Wada: poprawność opiera się na timeoutach, więc **fałszywe podejrzenie daje dwóch koordynatorów**.

## Algorytmy pierścieniowe

**Chang-Roberts** (1979) rozwiązuje elekcję w **pierścieniu jednokierunkowym**, w którym proces zna wyłącznie swojego następnika. Idea: w pierścieniu krąży komunikat z identyfikatorem, a każdy proces **przepuszcza identyfikator większy od własnego, a mniejszy zastępuje swoim**; flaga uczestnictwa tłumi komunikaty zbędne. Proces, do którego wraca **jego własny** identyfikator, wie, że jest największy, i ogłasza się liderem. Koszt: $O(N^2)$ w najgorszym przypadku (identyfikatory malejące zgodnie z kierunkiem pierścienia), średnio $O(N \log N)$, najlepiej $3N-1$.

**LeLann** (1977) jest prostszy i droższy: **każdy** komunikat obiega **pełne koło**, więc proces, do którego wraca własny komunikat, zna identyfikatory wszystkich inicjatorów i wybiera największy. Wymaga kanałów FIFO, kosztuje $O(N^2)$ **zawsze**.

**Hirschberg-Sinclair** (1980) rozwiązuje problem **kosztu kwadratowego** w pierścieniu **dwukierunkowym**. Idea: działanie w **fazach**, w których aktywny proces wysyła sondy w obie strony na odległość $2^k$; sondę pochłania każdy proces o większym identyfikatorze, a do fazy $k+1$ przechodzi tylko ten, któremu sondy wróciły z obu stron. Liderem zostaje proces, którego sonda obiegnie cały pierścień. Koszt $O(N \log N)$ — **asymptotycznie optymalny** dla pierścieni porównujących identyfikatory.

**Wariant z awariami** (Tanenbaum): komunikat elekcyjny **gromadzi listę identyfikatorów** procesów, przez które przeszedł, a niedostępny następnik jest pomijany (proces zna dalszych następników). Po powrocie do inicjatora z listy wybierany jest największy identyfikator i rozsyłany jako decyzja.

## Elekcja w dowolnej topologii — echo z wygaszaniem

Rozwiązuje elekcję, gdy topologia jest **dowolnym grafem**. Każdy inicjator uruchamia **falę** (algorytm echa) oznaczoną swoim identyfikatorem, a proces uczestniczy wyłącznie w fali o **największym znanym identyfikatorze**, fale mniejsze **wygaszając**. Do źródła wróci więc tylko fala największego inicjatora i to on zostaje liderem. Koszt $O(N \cdot |E|)$.

## Elekcja z losowością — Raft

Raft (2014) porządkuje czas **kadencjami** (*term*), a procesy trzyma w trzech stanach:

- *follower*,
- *candidate*,
- *leader*.

Follower, który przez **losowy** timeout nie dostał heartbeatu, zwiększa kadencję, głosuje na siebie i prosi innych o głos; każdy proces oddaje w danej kadencji **co najwyżej jeden głos**, i to tylko na kandydata z logiem nie starszym niż własny. Liderem zostaje ten, kto zbierze **większość**. Losowość timeoutów istnieje po to, by **minimalizować podział głosów**, a wymóg większości gwarantuje, że w jednej kadencji nie da się wybrać dwóch liderów.

## Elekcja a model systemu

W systemie **asynchronicznym z awariami** elekcja jest **nierozwiązywalna deterministycznie**: wiarygodna elekcja byłaby równoważna doskonałemu detektorowi awarii, a konsensus jest w tym modelu niemożliwy — wynik **FLP**, zob. [[23 Rozproszone uzgadnianie w środowisku zawodnym]]. Dlatego algorytmy praktyczne (Bully, Raft) zakładają **częściową synchroniczność** i dopuszczają, by przez chwilę istniało więcej niż jedno domniemane kierownictwo; bezpieczeństwo ratuje wtedy mechanizm **kadencji lub numerów propozycji** (Raft, [[Systemy Wysokiej Niezawodności/Algorytm Paxos|Paxos]]). To najważniejsza pointa zagadnienia: **algorytm elekcji nie jest samodzielny — jest tak dobry, jak model synchroniczności, w którym go uruchomiono**.

| Algorytm | Topologia | Model | Komunikaty (najgorszy) |
|---|---|---|---|
| Bully | graf pełny | synchroniczny, awarie typu crash | $O(N^2)$ |
| Chang-Roberts | pierścień jednokierunkowy | bez awarii | $O(N^2)$, średnio $O(N \log N)$ |
| LeLann | pierścień jednokierunkowy | FIFO, bez awarii | $O(N^2)$ |
| Hirschberg-Sinclair | pierścień dwukierunkowy | bez awarii | $O(N \log N)$ |
| echo z wygaszaniem | dowolna | bez awarii | $O(N \cdot \lvert E \rvert)$ |
| Raft | graf pełny | częściowo synchroniczny | losowe timeouty, większość |

---
## Czego w prezentacjach nie ma

> [!todo] Całe zagadnienie
> Prezentacje nie zawierają **niczego** na temat elekcji — ani definicji problemu, ani żadnego z algorytmów (0 trafień fraz „elekcj", „Bully", „Chang", „Hirschberg", „LeLann", „Raft", „konsensus" w 427 slajdach). Materiał: [[05 Algorytmy elekcji]], [[Systemy Wysokiej Niezawodności/Replikacja Procesu]], [[Systemy Wysokiej Niezawodności/Algorytm Paxos]].
