---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
source: "Slajdy-FT-06_Commit.pdf"
slajdy: "1–27"
zagadnienie: 24
---
# SWN 09. Transakcje, blokady i atomowe zatwierdzanie (2PC / 3PC)
---
> Wykład o **niezawodnym zatwierdzaniu transakcji rozproszonych**: od problemu współbieżności i izolacji, przez **2PL** i strategie rozstrzygania konfliktów (WAIT-DIE, WOUND-WAIT), po dwa protokoły zatwierdzania. **2PC** jest prosty, ale **blokujący** — awaria koordynatora zatrzymuje uczestników z założonymi blokadami. **3PC** dodaje stan *pre-commit*, który pozwala uczestnikowi podjąć **niezależną decyzję**. Wykład kończy się formalnym aparatem: **concurrency set**, własnościami **non-blocking** i **wait-freedom** oraz trzema twierdzeniami o niemożliwości.

---
## Transakcje

### Atomowa akcja i problemy
<sub>Slajdy-FT-06_Commit.pdf, slajd 1</sub>

> **Atomowa akcja** (*atomic action*) = zbiór niepodzielnych operacji.

**Problemy:**
1. **Współbieżność** — jak koordynować żądania współbieżnych atomowych akcji dotyczące operacji na tych samych obiektach?
2. **Izolacja** — wykonanie atomowej akcji musi być niezależne od innych atomowych akcji; tym bardziej **odtwarzanie** atomowej akcji musi być niezależne od innych atomowych akcji.

### Konflikty żądań
<sub>slajd 2</sub>

> **Współbieżne żądania są w konflikcie**, jeśli dotyczą **tego samego obiektu** i **przynajmniej jedno z nich wymaga jego modyfikacji**.

Rozwiązanie: **blokowanie dostępu** (*locking*).

Gdy **zarządca obiektu** otrzymuje żądanie będące w konflikcie z inną akcją już wykonywaną, może zastosować jedną z operacji:

| Operacja | Znaczenie |
|---|---|
| **WAIT** | nowe żądanie jest **kolejkowane** |
| **REJECT** | nowe żądanie jest **odrzucane** |
| **PREEMPT** | żądanie bieżąco obsługiwane jest **anulowane** |

W przypadku **REJECT** i **PREEMPT** odpowiednie atomowe akcje są **wycofywane** (*abort*). W przypadku **WAIT** atomowa akcja może być wycofana później.

---
## Blokady

### 2PL i zakleszczenie
<sub>slajd 3</sub>

> **2PL = 2 Phase Locking** — uszeregowanie zbioru atomowych akcji.

**Problem:** blokada **nie zawiera sama w sobie żadnych informacji** pomocnych w detekcji i/lub rozwiązywaniu konfliktów.

**Zakleszczenie!** Jeśli atomowa akcja oczekująca na założenie kolejnej blokady **może utrzymywać wcześniej uzyskane blokady**, może wystąpić zakleszczenie.

**Co wówczas?**
1. **detekcja zakleszczenia** i abort jednej lub kilku atomowych akcji,
2. **timeout** związany z blokadą:
   - a) na **utrzymywanie** blokady,
   - b) na **oczekiwanie** na założenie blokady,
   - ☹ timeout może prowadzić do **zbędnego wycofywania** atomowych akcji!

> [!note] Uzupełnienie spoza slajdów
> Slajdy **nie opisują faz** protokołu 2PL (wzrostu i zmniejszania blokad) ani wariantu *strict 2PL*. Notatka [[Systemy Wysokiej Niezawodności/2 Phase Locking]] zawiera wyłącznie rozwinięcie skrótu — to nadal **otwarty brak**, odnotowany w [[Braki w notatkach]].

### Transakcje rozproszone i etykiety czasowe
<sub>slajd 4</sub>

- protokół 2PL gwarantuje uszeregowanie atomowych akcji poprzez **dynamiczne uporządkowanie operacji**,
- alternatywnie porządek ten może być determinowany **a priori**, poprzez **globalnie jednoznaczne uszeregowanie żądań**,
- w systemie rozproszonym globalne uszeregowanie zapewnia mechanizm **etykiet czasowych** (*timestamps*):
$$TS = \langle clock(i), unique\_node\_ID(i) \rangle$$

### Strategie rozstrzygania konfliktów
<sub>slajd 5</sub>

1. **Zarządca obiektów zakłada blokady zgodnie z uszeregowaniem żądań.**
2. **Rozstrzyganie konfliktów zakładania blokad:**
   - **WAIT-DIE**: nowe żądanie $i$ **czeka** na blokadę posiadaną przez $j$, jeśli $TS_i < TS_j$; natomiast $TS_i \geqslant TS_j \Rightarrow$ **$i$ jest odrzucane** (akcja jest wycofywana),
   - **WOUND-WAIT**: $TS_i < TS_j \Rightarrow$ **$j$ jest odrzucane**; $TS_i \geqslant TS_j \Rightarrow$ **$i$ czeka**.

**Problem:** zbędne wycofywanie akcji (por. timeout); **niezawodność zegarów**.

---
## Atomowe zatwierdzanie
<sub>slajd 6</sub>

**Problem:** należy zagwarantować **globalną atomowość** rozproszonej transakcji:

> **Zgodność:** transakcję lokalnie zatwierdzają **wszystkie węzły, albo żaden**.
> **Poprawność:** jeśli wszystkie węzły zakończyły swoje operacje pomyślnie, transakcja **powinna zostać zatwierdzona**.

**Protokół zatwierdzania** (*commitment*):
- jego rolą jest zagwarantowanie **globalnej atomowości (i trwałości)**,
- każda składowa transakcja lokalnie dokonuje modyfikacji, **zapisując rejestry UNDO i REDO** — *w jakiej kolejności?* (dymek ze slajdu; odpowiedź: **write-ahead-log**, [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Wycofywanie operacji — updating-in-place i write-ahead-log]]).

---
## 2PC — dwufazowe zatwierdzanie

### Schemat komunikacyjny
<sub>slajdy 7–8</sub>

Slajd 7 to żartobliwy cytat z *Chatki Puchatka* („…rzecz, którą można z łatwością wyłożyć **dwa razy**, zanim ktokolwiek zrozumie, o co chodzi").

![[swn-ft06-s08-schemat-komunikacyjny-2pc.png]]
<sub>Schemat komunikacyjny 2PC: koordynator $P_0$ i uczestnicy $P_1 \ldots P_n$ — slajd 8</sub>

### Protokół 2PC (Gray [1])
<sub>slajd 9</sub>

| | **KOORDYNATOR** | **UCZESTNIK $P_i$** |
|---|---|---|
| **Faza 1** | 1. koordynator wysyła **agree_req** do wszystkich procesów — uczestników zatwierdzania<br>2. koordynator **czeka na odpowiedzi** od wszystkich procesów | 1. odebrawszy **agree_req** proces:<br>▸ jeśli lokalna część transakcji zakończyła się pomyślnie $\Rightarrow$ **zapisuje rejestry UNDO i REDO w pamięci trwałej** i odsyła odpowiedź **agree**<br>▸ w przeciwnym przypadku $\Rightarrow$ odsyła odpowiedź **abort** |
| **Faza 2** | 1. jeśli wszystkie procesy odpowiedziały **agree**, koordynator zapisuje **[COMMIT]** w rejestrze i wysyła do wszystkich procesów **commit**; w przeciwnym przypadku wysyła **rollback**<br>2. koordynator czeka na potwierdzenia **ack** od wszystkich procesów<br>3. gdy wszystkie procesy potwierdziły, koordynator zapisuje **[COMPLETE]** | 1. odebrawszy **commit**, proces zwalnia zasoby i blokady i odsyła potwierdzenie **ack**<br>2. odebrawszy **rollback**, proces wycofuje transakcję, zwalnia zasoby i blokady i odsyła potwierdzenie **ack** |

### Automaty 2PC
<sub>slajd 10</sub>

![[swn-ft06-s10-automaty-2pc.png]]
<sub>Automaty 2PC — koordynator $P_0$ ($q_0 \rightarrow w_0 \rightarrow \{a_0, c_0\} \rightarrow CT_0$) i uczestnik $P_i$ ($q_i \rightarrow w_i \rightarrow \{a_i, c_i\}$), slajd 10</sub>

**Koordynator $P_0$:** $q_0$ —(*time* / send **agree_req**)→ $w_0$; z $w_0$: (*not all $P_i$ agreed* / send **rollback**) → $a_0$, albo (*all $P_i$ agreed* / send **commit**) → $c_0$; z $c_0$: (*all ack received*) → $CT_0$.

**Uczestnik $P_i$:** $q_i$ —(**agree_req** received / send **agree** to $P_0$)→ $w_i$, albo (**agree_req** received → *vote abort* / send **abort** to $P_0$)→ $a_i$; z $w_i$: (**rollback** received / send **ack**)→ $a_i$, albo (**commit** received / send **ack**)→ $c_i$.

### Awarie
<sub>slajdy 11–13</sub>

**Poprawność (uściślenie, slajd 11):** *jeśli wszystkie węzły zakończyły swoje operacje pomyślnie **i wszystkie odpowiedzi węzłów zostały poprawnie dostarczone**, transakcja powinna zostać zatwierdzona.* Pytania: **kanały niezawodne? a co z awariami procesów?**

**Model fail-stop** — pięć punktów awarii (slajdy 12–13):

| Punkt | Sytuacja | *Recovery* |
|---|---|---|
| **K1** | koordynator **nie zapisał [COMMIT]** | koordynator wysyła **rollback**, UNDO; **procesy są zablokowane** aż do odebrania rollback |
| **K2** | awaria **pomiędzy [COMMIT] a [COMPLETE]** | koordynator wysyła **commit**; **procesy są zablokowane** aż do odebrania commit |
| **K3** | awaria **po [COMPLETE]** | ??? — *nie ma czego rozważać, pomijamy stan $CT_0$* |
| **P1** | koordynator **nie dostał odpowiedzi na agree_req** | **timeout** $\Rightarrow$ koordynator wysyła **rollback** |
| **P2** | $P_i$ zapisał UNDO i REDO log, ale koordynator **nie dostał ack** | $P_i$ **pyta o ostateczną decyzję** koordynatora; jeśli $P_i$ uległ awarii, nim zmodyfikował wszystkie obiekty $\Rightarrow$ **REDO**; alternatywnie koordynator może **powtarzać wysyłanie commit** |

### Automaty z przejściami F i T
<sub>slajdy 14–17</sub>

![[swn-ft06-s16-automaty-2pc-F-T.png]]
<sub>Automat koordynatora z przejściami **F** (*failure*) i **T** (*timeout*): z $q_0$ i $w_0$ przejście F prowadzi do $a_0$ (send **rollback**), a w $c_0$ pętla *not all ack* / T → **resend commit** — slajdy 15–16</sub>

![[swn-ft06-s17-automat-uczestnika-2pc.png]]
<sub>Automat uczestnika: ze stanu $w_i$ wychodzi **czerwona strzałka zakończona znakiem zapytania** — uczestnik w stanie oczekiwania **nie ma dokąd pójść samodzielnie**; to jest formalny obraz zablokowania 2PC, slajd 17</sub>

### Zablokowanie przetwarzania i concurrency set
<sub>slajdy 18–20</sub>

> **Jeśli koordynator ulegnie awarii, procesy zostają zablokowane aż do jego odtworzenia, przetrzymując zasoby (utrzymując blokady).**

> **Concurrency set.** Niech $s_i$ oznacza stan procesu $P_i$. Zbiór wszystkich stanów pozostałych procesów*, które mogą występować **współbieżnie** ze stanem $s_i$, nazywa się *concurrency set* stanu $s_i$: $\text{Cset}(s_i)$.
>
> \* *tu nie uwzględniamy tranzycji F i T*

$$\text{Cset}(q_i) = \{q_0, w_0, q_j, w_j, a_j\}$$
$$\text{Cset}(w_i) = \{w_0, \mathbf{a_0}, \mathbf{c_0}, q_j, w_j, \mathbf{a_j}, \mathbf{c_j}\}$$

> **Zablokowanie w 2PC może wystąpić, gdy dla dowolnego stanu $s$ zbiór $\text{Cset}(s)$ zawiera jednocześnie stany $a$ i $c$.**

**Dlaczego:** proces w stanie $w_i$ nie wie, czy reszta systemu jest w stanie *abort*, czy *commit* — obie możliwości są w jego concurrency set, a lokalnie ich nie odróżni.

---
## 3PC — trójfazowe zatwierdzanie

### Algorytm Skeena [2] — założenia
<sub>slajd 21</sub>

**Założenia:**
- kanały są niezawodne *uniform reliable reordering channels* (**nie gubią komunikatów**; kanał **dostarczy wysłany komunikat nawet jeśli jego nadawca uległ awarii**),
- koordynator używa rozgłaszania *uniform reliable broadcast* (**URBcast**),
- **sieć wykrywa awarię węzła**: awarię **nieomylnie (!)** wskazuje **timeout**,
- **co najwyżej jeden węzeł ulega awarii**.

**Stan buforowy automatu:**
- cel: **pozwolić procesom podjąć niezależną decyzję** (na podstawie stanu lokalnego).

### Protokół
<sub>slajd 22</sub>

**Faza 2** (koordynator): jeśli wszystkie procesy odpowiedziały **agree**, koordynator zapisuje **[PRECOMMIT]** w rejestrze, wysyła **URBcast** do wszystkich procesów **precommit** i czeka na potwierdzenie **precom_ok**; w przeciwnym przypadku wysyła **rollback**.

**Faza 3** (koordynator): gdy wszystkie procesy odpowiedzą **precom_ok**, koordynator zapisuje **[COMMIT]** w rejestrze i wysyła do wszystkich procesów **commit**.

### Automaty 3PC
<sub>slajd 23</sub>

![[swn-ft06-s23-automaty-3pc.png]]
<sub>Automaty 3PC — koordynator $q_0 \rightarrow w_0 \rightarrow \{a_0, p_0\} \rightarrow c_0$ oraz uczestnik $q_i \rightarrow w_i \rightarrow \{a_i, p_i\} \rightarrow c_i$; slajd 23</sub>

**Koordynator:** z $w_0$: (*not all $P_i$ agreed* / send **rollback**) → $a_0$, albo (*all $P_i$ agreed* / send **precommit**) → $p_0$; z $p_0$: (*all **precom_ok** received* / send **commit**) → $c_0$.

**Uczestnik:** z $w_i$: (**rollback** received) → $a_i$, albo (**precommit** received / send **precom_ok**) → $p_i$; z $p_i$: (**commit** received / send **ack**) → $c_i$.

### Rola stanu $p$
<sub>slajd 24</sub>

$$\text{Cset}(w_0) = \{q_i, w_i, a_i\}$$
$$\text{Cset}(w_i) = \{w_0, p_0, \mathbf{a_0}, q_j, w_j, \mathbf{a_j}, p_j\}$$

> **Jeśli $\text{Cset}(s_i)$ zawiera stan $c$, to wobec awarii koordynatora proces $P_i$ może lokalnie podjąć NIEZALEŻNĄ decyzję o przejściu do stanu $c_i$; w przeciwnym przypadku — do stanu $a_i$.**

Ponieważ w 3PC **żaden $\text{Cset}$ nie zawiera jednocześnie $a$ i $c$** (rozdziela je stan $p$), decyzja lokalna jest zawsze jednoznaczna — **3PC jest nieblokujący**.

> **Zadanie ze slajdu:** *Uzupełnić przejścia automatów 3PC.*

---
## Non-blocking i wait-freedom
<sub>slajd 25</sub>

> **Algorytm nieblokujący** (*non-blocking*) gwarantuje osiągnięcie zakończenia (*termination*) przez **co najmniej 1 proces**, jeśli **co najmniej 1 proces nie ulega awarii**.

> **Wait-freedom** to własność gwarantująca, że **każdy proces**, który nie ulega awarii, osiągnie zakończenie przetwarzania (*termination*) **bez względu na zachowanie innych procesów**.

Własność *wait-freedom* jest **silniejsza** niż *non-blocking*.

> **Pytanie ze slajdu:** *Którą z nich spełnia 2PC? A którą 3PC?*

### Wyniki niemożliwości
<sub>slajd 26</sub>

> **Twierdzenie 1** [3]: Nie istnieje nieblokujący protokół rozproszonego atomowego zatwierdzania, który byłby odporny na **arbitralne defekty 2 węzłów**.

> **Twierdzenie 2**: Nie istnieje nieblokujący protokół rozproszonego atomowego zatwierdzania, który byłby odporny na **rozdzielenie sieci** (separację na rozłączne podsieci — *partitioning*) przy możliwości gubienia komunikatów.

> **Twierdzenie 3**: Nie istnieje nieblokujący protokół rozproszonego atomowego zatwierdzania, który byłby odporny na **wielokrotne rozdzielenie sieci**. $\rightarrow$ [4]

> [!important] Dlaczego 3PC nie jest panaceum
> Założenia ze slajdu 21 — **co najwyżej jeden węzeł ulega awarii** i **timeout nieomylnie wskazuje awarię** — są bardzo mocne. Twierdzenia 1–3 pokazują, że po ich osłabieniu (2 awarie, partycjonowanie sieci) **nieblokujący protokół nie istnieje**. To bezpośrednio łączy się z [[SWN 07 Konsensus#Konsensus a CAP]], gdzie 2PC leży w części CA diagramu CAP.

---
## Literatura
<sub>slajd 27</sub>

1. J. N. Gray, *Notes on Database Operating Systems*, „Operating Systems. An Advanced Course", Lecture Notes in Computer Science, vol. 60, Springer, New York, 1979.
2. D. Skeen, *Nonblocking Commit Protocols*, ACM SIGMOD Conference on Management of Data, 1981.
3. D. Skeen, M. Stonebraker, *A Formal Model of Crash Recovery in a Distributed System*, IEEE Trans. Software Engineering, vol. 9, no. 3, 1983.
4. I. Keidar, D. Dolev, *Increasing the Resilience of Atomic Commit, at no Additional Cost*, 14th ACM Symposium on Principles of Database Systems, 1995.
5. R. Chow, T. Johnson, *Distributed Operating Systems & Algorithms*, Addison Wesley Longman, 1997, ch. 12.
6. K. P. Birman, *Reliable Distributed Systems*, Springer Science, 2005, ch. 14.
7. G. Coulouris et al., *Distributed Systems, Concepts and Design*, Addison Wesley, 2012, ch. 17.
8. A. Petrov, *Database Internals*, Springer Science, 2019, ch. 13.

---
## Porównanie 2PC i 3PC

| | **2PC** | **3PC** |
|---|---|---|
| Fazy | 2 (agree_req/agree, commit/rollback + ack) | 3 (dochodzi precommit / precom_ok) |
| Stany koordynatora | $q_0, w_0, a_0, c_0, CT_0$ | $q_0, w_0, a_0, p_0, c_0$ |
| Stany uczestnika | $q_i, w_i, a_i, c_i$ | $q_i, w_i, a_i, p_i, c_i$ |
| $\text{Cset}(w_i)$ zawiera $a$ **i** $c$? | **tak** $\rightarrow$ blokowanie | **nie** (rozdziela je $p$) |
| Blokujący? | **tak** — awaria koordynatora blokuje uczestników z blokadami | **nie** — przy założeniach ze slajdu 21 |
| Kanały | zwykłe | *uniform reliable reordering*, URBcast |
| Awarie | dowolna liczba (ale blokuje) | **co najwyżej 1 węzeł** |

---
## Braki i uwagi

> [!todo] Czego nie ma na slajdach tego wykładu
> - **Faz protokołu 2PL** (wzrostu i zmniejszania blokad), wariantu **strict 2PL** — slajd 3 podaje tylko rozwinięcie skrótu i problem zakleszczenia.
> - **Odpowiedzi** na pytanie ze slajdu 25 („którą własność spełnia 2PC, a którą 3PC?") — z materiału wynika, że **2PC nie jest nawet non-blocking**, a **3PC jest non-blocking, ale nie wait-free**.
> - **Rozwiązania zadania** ze slajdu 24 (uzupełnić przejścia automatów 3PC o F i T).
> - **Pełnych $\text{Cset}$ dla 3PC** — slajd 24 urywa listę wielokropkiem.
> - **Własności ACID** — wykład ich nie wymienia; [[ACID]] w vaultcie to same nagłówki.
> - **Protokołów terminacji i odtwarzania po restarcie uczestnika** w 3PC (dla 2PC jest to punkt P2 na slajdzie 13).
