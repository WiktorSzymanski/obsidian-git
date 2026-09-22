---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
source: "Slajdy-FT-09_Agreement.pdf"
slajdy: "1–39"
zagadnienie: 23
---
# SWN 05. Problemy uzgadniania i wyniki niemożliwości
---
> Wykład wprowadza **rodzinę problemów uzgadniania**: skoordynowany atak (*coordinated attack*), **konsensus**, **porozumienie bizantyjskie** (BA) i **spójność interaktywną** (IC) — pokazując, że wszystkie są wzajemnie sprowadzalne. Następnie podaje **dwa wyniki niemożliwości**: dla **awarii łączy** (skoordynowany atak jest nierozwiązywalny nawet w systemie synchronicznym) i dla **awarii procesów** (**FLP'85** — brak deterministycznego konsensusu w systemie asynchronicznym przy choćby jednej awarii). Kończy się trzema drogami obejścia FLP i szczegółowym omówieniem jednej z nich — modelu **procesów początkowo martwych** z algorytmem Fischera-Lyncha-Patersona.

---
## Problemy uzgadniania — motywacja
<sub>Slajdy-FT-09_Agreement.pdf, slajd 1</sub>

Problemy uzgadniania pojawiają się w wielu praktycznych zastosowaniach:
- uzgodnienie, czy **zatwierdzić, czy wycofać** wyniki rozproszonej akcji atomowej (np. transakcji bazodanowej),
- uzgodnienie **oszacowania wysokości samolotu** na podstawie odczytów wielu wysokościomierzy,
- uzgodnienie, czy **zaklasyfikować komponent systemu jako uszkodzony**, na podstawie wyników osobnych testów diagnostycznych wykonanych przez osobne procesy.

---
## Skoordynowany atak

### Scenariusz
<sub>slajdy 2–3</sub>

**Podstawowy problem uzgadniania** — scenariusz pola bitwy (nieformalnie):
- kilku generałów planuje **skoordynowany atak z różnych kierunków** na wspólny cel,
- **jedyną drogą powodzenia ataku jest zaatakowanie wszystkich naraz** (wyróżnione ramką na slajdzie),
- każdy generał ma **początkową opinię**, czy jego armia jest gotowa do ataku,
- generałowie komunikują się **wyłącznie przez posłańców**.

### Protokół ad-hoc i awarie łączy
<sub>slajd 4</sub>

**Protokół ad-hoc** (w środowisku **bezawaryjnym** — dymek na slajdzie):
- każdy generał wysyła swoją opinię do wszystkich pozostałych,
- osiągają porozumienie na podstawie sumy zebranych opinii: jeśli większość niezależnie decyduje **ATTACK**, to atak; jeśli większość **RETREAT**, to odwrót,
- głosowanie większościowe: *a jeśli są podzieleni, to… hmm… no… kogo to obchodzi?*,
- może wymagać kilku rund (**liczba rund = średnica grafu komunikacyjnego**).

**Awarie łączy:**
- posłańcy mogą zostać **zgubieni lub schwytani**,
- **protokół przestaje działać!**

> *In fact there is no algorithm that solves this problem correctly.*

### Definicja formalna
<sub>slajd 5</sub>

$1 = \text{attack} / \text{commit}$, $0 = \text{retreat} / \text{abort}$.

> **Termination** (*liveness*): wszystkie procesy w końcu decydują.
> **Agreement** (*safety*): żadne dwa procesy nie decydują o różnych wartościach.
> **Validity** (wersja **Weak Validity**):
> 1. jeśli wszystkie procesy startują z 0, to **0 jest jedyną możliwą** wartością decyzji,
> 2. jeśli wszystkie procesy startują z 1 **i wszystkie komunikaty zostaną dostarczone**, to **1 jest jedyną możliwą** wartością decyzji.

> *Validity jest dość słaba: jeśli choć jeden proces startuje z wartością 1, algorytm może zdecydować na 1; jeśli wszystkie procesy startują z 1, a wszystkie komunikaty zostaną zgubione, algorytm może zdecydować na 0.*
>
> *Nawet ta wersja Weak Validity problemu Coordinated Attack jest **niemożliwa do rozwiązania** w dowolnym systemie rozproszonym z dwoma lub więcej węzłami połączonymi zawodnymi kanałami. **Nawet w modelu systemu synchronicznego!***

---
## Model systemu

### System synchroniczny $\mathbb{S}^{Sync}$
<sub>slajd 6</sub>

- kanał (łącze) $c_{ij}$ w dowolnej chwili może przechowywać **co najwyżej jedną wiadomość** $\in \mathbb{M}$,
- **funkcja tranzycji stanu** odwzorowuje (deterministycznie) $states_i$ i wektory wiadomości przychodzących $\in \mathbb{M} \cup \{null\}$ na $states_i$,
- **funkcja generacji wiadomości** odwzorowuje $states_i \times$ sąsiedzi na elementy $\mathbb{M} \cup \{null\}$,
- wykonanie całego systemu rozpoczyna się z **wszystkimi procesami w dowolnych stanach początkowych i wszystkimi kanałami pustymi**,
- procesy **krok w krok** (*lock-step*) powtarzają dwa kroki:
  - **krok 1)** zgodnie z bieżącym stanem generują wiadomości do wysłania do sąsiadów; umieszczają te wiadomości w odpowiednich kanałach,
  - **krok 2)** stosują funkcję tranzycji do bieżącego stanu i wiadomości przychodzących, przechodząc do nowego stanu; usuwają wszystkie wiadomości z kanałów,
  - *kombinacja tych dwóch kroków nazywana jest **rundą***.

### Awarie
<sub>slajd 7</sub>

- ogólnie rozważać można zarówno **awarie procesów**, jak i **awarie łączy**,
- proces może wykazać **awarię zatrzymania** (*stopping failure*), zatrzymując się gdzieś w środku wykonania: może paść przed lub po kroku 1 lub 2, albo **w środku kroku 1** (umieszczając tylko **podzbiór** wiadomości wyjściowych),
- proces może wykazać **awarię bizantyjską**: może wygenerować swoje kolejne wiadomości i kolejny stan w **dowolny sposób, niezależnie** od funkcji generacji wiadomości i funkcji tranzycji stanu,
- łącze może zawieść przez **gubienie wiadomości** (proces może próbować umieścić wiadomość w kanale w kroku 1, ale uszkodzone łącze może jej nie zarejestrować).

> **Dymek ze slajdu:** w dalszej dyskusji Coordinated Attack rozważamy **wyłącznie awarie łączy**.

---
## Wynik niemożliwości dla awarii łączy
<sub>slajdy 8–12</sub>

> **Twierdzenie.** Niech $G$ będzie grafem złożonym z **2 węzłów połączonych zawodną krawędzią**. **Nie istnieje algorytm rozwiązujący problem Coordinated Attack na $G$.**

*Pokazujemy wynik niemożliwości dla najprostszego przypadku 2 węzłów połączonych 1 krawędzią. Ten przypadek implikuje niemożliwość dla dowolnego grafu o większej liczbie węzłów.*

**Szkic dowodu (nie wprost):**
1. Przypuśćmy, że rozwiązanie istnieje — algorytm $A$.
2. Bez utraty ogólności zakładamy: istnieje **tylko 1 stan początkowy** dla każdej wartości wejściowej (tj. dla danego wejścia i wzorca wiadomości możliwe jest tylko 1 wykonanie), a **oba procesy $P_1$ i $P_2$ wysyłają wiadomości w każdej rundzie** $A$.
3. Niech $\alpha$ będzie wykonaniem, w którym oba procesy startują z wartością 1 i **wszystkie wiadomości są dostarczane**.
4. Z **Termination** — oba decydują (załóżmy w rundzie $r$), a z **Validity** — na wartość 1.

![[swn-ft09-s09-wykonanie-alfa1.png]]
<sub>Wykonanie $\alpha_1$: takie samo jak $\alpha$, z tym że **po rundzie $r$ wszystkie wiadomości są gubione** (w $\alpha_1$ oba procesy również decydują na 1) — slajd 9</sub>

![[swn-ft09-s10-wykonanie-alfa2.png]]
<sub>Wykonanie $\alpha_2$: takie samo jak $\alpha_1$, z tym że **gubiona jest $m_1$** — slajd 10</sub>

- $\alpha_2$ jest **nieodróżnialne** od $\alpha_1$ dla $P_1$ (zapis: $\alpha_2 \sim^{1} \alpha_1$),
- skoro $P_1$ decyduje na 1 w $\alpha_1$, decyduje na 1 również w $\alpha_2$,
- z **Termination** i **Agreement** — $P_2$ również decyduje na 1.

Kontynuacja (slajd 11): $m_2$ jest gubiona w $\alpha_3 \sim^{2} \alpha_2$ — $P_2$ nadal decyduje na 1, więc musi też $P_1$; … itd.

![[swn-ft09-s12-naruszenie-validity.png]]
<sub>Finał dowodu — slajd 12</sub>

- $\alpha'$ = oba procesy startują z 1 i **żadna wiadomość nie jest dostarczana** — oba decydują na 1,
- $\alpha'' \sim^{1} \alpha'$ = $P_1$ startuje z 1, $P_2$ startuje z 0 — $P_1$ nadal decyduje na 1, więc musi też $P_2$,
- $\alpha''' \sim^{2} \alpha''$ = **oba startują z 0** — $P_2$ nadal decyduje na 1 — **Validity jest naruszone!** $\square$

---
## Pozostałe problemy uzgadniania

### Konsensus
<sub>slajd 13</sub>

> **Termination:** wszystkie $P_c$ (*correct* — dymek ze slajdu) decydują dokładnie 1 wartość $v_i$.
> **Agreement:** wszystkie $P_c$ decydują **to samo** $v_i$.
> **Validity:** $v_i$ zostało zaproponowane przez **pewien** $P_i$.

*Każdy proces rozgłasza swoją wartość początkową. Wartości początkowe różnych procesów mogą być różne. Wszystkie niewadliwe procesy uzgadniają **dowolną** wspólną wartość (nie obchodzi nas która, o ile warunek validity nie jest naruszony). Nieistotne jest, na jaką wartość uzgodnią się procesy wadliwe ani czy w ogóle się uzgodnią. Jest to **bezpośrednie uogólnienie problemu Coordinated Attack** (w istocie Coordinated Attack jest konsensusem binarnym).*

### Porozumienie bizantyjskie (Byzantine Agreement)
<sub>slajd 14</sub>

> **Termination:** wszystkie $P_c$ decydują dokładnie 1 wartość $v_s$.
> **Agreement:** wszystkie $P_c$ decydują **to samo** $v_s$.
> **Validity:** **jeśli wyróżnione źródło $P_s$ jest poprawne**, to $v_s$ jest wartością zaproponowaną przez $P_s$.

*Dowolnie wybrany proces $P_s$ (źródło) rozgłasza swoją wartość początkową do pozostałych procesów. Jeśli źródło jest wadliwe, wszystkie niewadliwe procesy uzgadniają **dowolną** wspólną wartość.*

### Spójność interaktywna (Interactive Consistency)
<sub>slajd 15</sub>

> **Termination:** wszystkie $P_c$ uzgadniają 1 wektor $\langle v_1, v_2, \ldots, v_N \rangle$.
> **Agreement:** wszystkie $P_c$ uzgadniają **ten sam** wektor.
> **Validity:** jeśli $P_i$ jest poprawny, $i$-ta wartość wektora $v_i$ została zaproponowana przez $P_i$.

*Każdy proces rozgłasza swoją wartość początkową. Procesy uzgadniają **wszystkie** zaproponowane wartości. Jeśli $i$-ty proces jest wadliwy, wszystkie niewadliwe procesy uzgadniają **dowolną** wspólną wartość dla $v_i$.*

### Relacje między problemami
<sub>slajdy 16–17</sub>

> **Dymek ze slajdu 16:** na razie rozważamy **tylko wykonania bezawaryjne**.

- wszystkie problemy uzgadniania są **ściśle powiązane**,
- **(IC $\rightarrow$ BA)** BA jest **szczególnym przypadkiem** IC, w którym interesuje nas wartość początkowa **tylko jednego** procesu,
- **(BA $\rightarrow$ IC)** jeśli każdy z $N$ procesów uruchomi kopię protokołu BA, problem IC zostaje rozwiązany,
- **(IC $\rightarrow$ C)** konsensus można rozwiązać, używając rozwiązania IC — wszystkie niewadliwe procesy mogą obliczyć decyzję na podstawie **wartości większościowej** wspólnego wektora albo wybierając **pierwszą** wartość takiego wektora,
- **(BA $\rightarrow$ IC $\rightarrow$ C)** zatem rozwiązania IC i konsensusu dają się wyprowadzić z rozwiązań BA,
- **ale** to nie znaczy, że BA jest słabszy niż IC ani że IC jest słabszy niż konsensus — **nie ma porządku liniowego**.

**(C $\rightarrow$ BA)** — BA można rozwiązać przy użyciu konsensusu (slajd 17):
1. źródło wysyła swoją wartość do wszystkich pozostałych procesów, **włącznie z sobą samym**,
2. wszystkie procesy uruchamiają algorytm konsensusu, używając wartości otrzymanych w kroku 1 jako swoich propozycji.

*(Jeśli źródło jest niewadliwe, wszystkie procesy otrzymają tę samą wartość w kroku 1 i wszystkie niewadliwe procesy uzgodnią ją w kroku 2. Jeśli źródło jest wadliwe, pozostałe procesy mogą nie otrzymać tej samej wartości w kroku 1, ale wszystkie niewadliwe procesy uzgodnią tę samą wartość w wyniku algorytmu konsensusu w kroku 2.)*

> **Dymek ze slajdu 17:** *Czy te transformacje nadal działają przy wykonaniach podatnych na awarie?*

---
## Wynik niemożliwości dla awarii procesów — FLP'85
<sub>slajdy 18–22 (slajd 18 to przerywnik „Application failures")</sub>

> **FLP'85 Impossibility Result**
> **Nie istnieje deterministyczne rozwiązanie konsensusu w systemie asynchronicznym, jeśli choćby jeden proces może ulec awarii typu crash.**

(FLP'85 = Fischer-Lynch-Paterson 1985 $\rightarrow$ [1])

**Dlaczego?** (slajd 20) — *w systemach asynchronicznych nie możemy odróżnić procesu, który uległ awarii, od procesu wolnego lub bardzo odległego.*

![[swn-ft09-s20-nierozroznialnosc-awarii.png]]
<sub>$P_2$ widzi $\langle v_1, v_2, v_3, null \rangle = \langle 1, 1, 0, null \rangle$ — nie wie, czy $P_4$ padł, czy jest tylko wolny; slajd 20. Slajd 21 pokazuje symetryczną sytuację z perspektywy $P_3$: $\langle null, 1, 0, 0 \rangle$.</sub>

**Trzy warunki twierdzenia** (slajd 22) — każdy z nich jest istotny:
1. rozwiązanie **deterministyczne**,
2. system **asynchroniczny**,
3. choćby jeden proces może ulec **crashowi**.

### Obejścia
<sub>slajd 23</sub>

> *Pomimo wyników niemożliwości wiele nietrywialnych problemów ma rozwiązania, nawet w systemach asynchronicznych z awariami.*

Wynik FLP'85 okazuje się **bardzo czuły na osłabienie założeń modelu**:
1. **Randomizacja** — osłabienie warunku terminacji:
   > **Termination:** każdy $P_c$ w końcu decyduje **z prawdopodobieństwem 1**

   tj. poświęcenie *liveness* na rzecz *safety* — np. **Paxos** $\rightarrow$ [2],
2. **Silniejsza synchronia** — model systemu synchronicznego (quasi-synchronicznego),
3. **Słabszy model awarii** — np. model **procesów początkowo martwych** (*initially dead-processes*).

---
## Model procesów początkowo martwych
<sub>slajdy 24–31</sub>

> *W modelu procesów początkowo martwych **żaden proces nie może ulec awarii po wykonaniu zdarzenia**.*

> **Def.** ***$f$-initially-dead fair execution*** $N$ procesów to wykonanie, w którym **co najmniej $N - f$ procesów jest aktywnych**, każdy aktywny proces jest poprawny i każda wiadomość wysłana do poprawnego procesu zostaje dostarczona.

> W ***$f$-initially-dead fair execution*** możemy rozwiązać konsensus, **dopóki $f < \frac{N}{2}$**.

**Ogólna idea:** ponieważ procesy **nie ulegają awarii po wysłaniu wiadomości**, dla $P_i$ jest bezpieczne czekać na odbiór wiadomości od $P_j$, jeśli wie, że $P_j$ **już wysłał co najmniej jedną wiadomość**.

### Algorytm Fischera-Lyncha-Patersona
<sub>slajd 25</sub>

**Struktury:**
- $Fellows_i$, $Active_i$, $Rcvd_i$ — zbiory identyfikatorów procesów; początkowo $\varnothing$,
- $M = \left\lceil \frac{N+1}{2} \right\rceil$ — **co najmniej $M$ procesów jest poprawnych**.

```
P_i:
    bcast(NAME,i)

    while (|Fellows_i| < M-1)                    -- first stage
        recv(NAME,j)
        Fellows_i := Fellows_i + {j}

    bcast(PROPOSE,i,v_i,Fellows_i)
    Active_i := Fellows_i

    while (Active_i not-subset-of Rcvd_i)        -- second stage
        recv(PROPOSE,j,v_j,Fellows_j)
        Active_i := Active_i + Fellows_j + {j}
        Rcvd_i   := Rcvd_i + {j}

    compute_knot()
```

### Faza pierwsza — budowa grafu $G$
<sub>slajdy 26–27</sub>

- procesy konstruują **graf skierowany $G$**, rozgłaszając swoją tożsamość,
- *fellows* procesu $P_i$ to takie $P_j$, od których $P_i$ odebrał wiadomość NAME: krawędź $\overline{(i,j)}$ w $G$ $\iff$ $recv(P_i, \overleftarrow{P_j}, \text{NAME})$,
- $P_i$ czeka na odbiór $M-1$ wiadomości — skoro jest co najmniej $M$ procesów poprawnych, **każdy poprawny proces $P_c$ odbierze dostatecznie wiele wiadomości**, by zakończyć tę pracę,
- proces **początkowo martwy nie wysłał żadnych wiadomości** — tworzy w $G$ **węzeł izolowany**,
- każdy $P_c$ ma $M-1$ *fellows* — **nie jest izolowany**.

![[swn-ft09-s26-graf-G-pierwsza-faza.png]]
<sub>Graf $G$; czerwone kółko po prawej to proces początkowo martwy (węzeł izolowany) — slajd 26</sub>

**Istnienie i jednoznaczność węzła (*knot*)** (slajd 27):
- istnieje w $G$ **knot zawierający procesy poprawne**, oznaczmy go $K$,
- ponieważ każdy $P_c$ ma stopień wyjściowy $M-1$, ten knot ma rozmiar **co najmniej $M$**,
- w konsekwencji istnieje **dokładnie 1 knot** (bo $2M > N$); **nie wszystkie $P_c$ muszą należeć do $K$**,
- ponieważ poprawny $P_i$ ma $M-1$ *fellows*, **co najmniej jeden fellow należy do $K$**, co implikuje, że **wszystkie procesy w $K$ są potomkami $P_i$**.

![[swn-ft09-s27-knot-K.png]]
<sub>Knot $K$ (obszar przerywany) i proces $P_i$ spoza niego — slajd 27</sub>

### Faza druga
<sub>slajd 28</sub>

- każdy $P_c$ konstruuje **własny obraz $G$**, odbierając zbiór *fellows* od każdego procesu, o którym wie, że jest poprawny (**nie powstaje zakleszczenie**, bo żaden $P_c$ nie może już ulec awarii),
- na końcu tej fazy każdy $P_c$ otrzymał zbiór *fellows* **każdego ze swoich potomków**, co pozwala **obliczyć jednoznaczny knot $K$** w $G$.

![[swn-ft09-s28-druga-faza.png]]
<sub>Druga faza — propagacja zbiorów *fellows*, slajd 28</sub>

### Konsensus i złożoność
<sub>slajdy 29–31</sub>

- skoro wszystkie $P_c$ uzgadniają knot $K$ procesów poprawnych,
- i każdy z nich rozgłasza swoją proponowaną wartość $v_i$ wraz ze swoimi *fellows*,
- po obliczeniu $K$ procesy **decydują o wartości w funkcji zebranych wartości od procesów z $K$** (większość wartości albo najmniejsza wartość itp.),
- **wymagane $O(N^2)$ wiadomości**.

> **Kluczowe (slajd 30):** *Consensus is computed **only among propositions from K**!*

![[swn-ft09-s31-inny-przyklad-grafu-G.png]]
<sub>Inny przykład grafu $G$ — knot $K$ nie obejmuje wszystkich procesów poprawnych, slajd 31</sub>

---
## Dalsze problemy uzgadniania

### $k$-set consensus, approximate agreement, renaming
<sub>slajd 32</sub>

- **$k$-set consensus** — procesy uzgadniają **mały zbiór $k$ wartości**,
- **approximate agreement** — procesy uzgadniają wartości **bliskie sobie nawzajem**,
- **renaming** — procesy uzgadniają wartości **koniecznie różne**.

### Problem przemianowania
<sub>slajd 33</sub>

Problem *renaming* przypisuje każdemu procesowi $P_i$ nazwę $x_i$ z dziedziny $\mathbb{X}$:

> **Termination:** każdy poprawny $P_i$ w końcu otrzymuje nazwę $x_i$.
> **Validity:** nazwa $x_i$ należy do $\mathbb{X}$.
> **Agreement:** dla procesów niewadliwych $P_i$ i $P_j$: $x_i \neq x_j$.
> **Anonymity:** kod wykonywany przez dowolny proces **nie może zależeć od jego początkowego identyfikatora**.

*Problem renaming jest użyteczny przy transformacji przestrzeni nazw — np. gdy procesy z różnych dziedzin muszą współpracować, ale najpierw muszą przypisać sobie różne nazwy z małej dziedziny. Innym przykładem jest sytuacja, gdy procesy muszą użyć swoich unikalnych nazw (ID) jako etykiet porządkujących (w kolejce priorytetowej).*

### Elekcja a konsensus
<sub>slajd 34</sub>

**Elekcja:**
- w modelu procesów początkowo martwych, skoro wszystkie procesy uzgadniają knot $K$ procesów poprawnych, **wybór procesu jest trywialny**,
- np. wybierany jest proces o **najmniejszym ID w $K$**.

**Elekcja i konsensus:**
- w modelu procesów początkowo martwych **każdy algorytm elekcji wybierający proces poprawny jako lidera rozwiązuje również problem konsensusu**: lider rozgłasza swoją wartość początkową i wszystkie poprawne procesy na nią decydują,
- **jednak w modelu fail-stop dostępność lidera nie pomaga** — lider może paść przed rozgłoszeniem swojej wartości,
- **w każdym razie elekcja nie jest rozwiązywalna przy awariach typu crash**.

---
## Uogólnienie — zadanie rozproszone
<sub>slajdy 35–38</sub>

> **Def.** ***Zadanie rozproszone*** (*distributed task*) jest opisane zbiorami $In$ i $Out$ możliwych wartości wejściowych i wyjściowych oraz (być może częściową) funkcją $\mathcal{T}: \mathbb{In}^{N} \rightarrow \mathbb{Out}^{N}$.

Interpretacja odwzorowania $\mathcal{T}$: jeśli wektor $I = \langle in_1, \ldots, in_N \rangle$ opisuje wejście procesów, to $\mathcal{T}(I)$ jest zbiorem **legalnych wyjść** algorytmu — wektorów decyzji $O = \langle out_1, \ldots, out_N \rangle$. Jeśli $\mathcal{T}$ jest funkcją częściową, **nie każda kombinacja wartości wejściowych $I$ jest dozwolona**.

![[swn-ft09-s36-distributed-task.png]]
<sub>Zadanie rozproszone jako odwzorowanie wejść na wyjścia — slajd 36</sub>

**Przykłady** (slajd 37):
- **konsensus** — wszystkie decyzje muszą być równe:
$$\mathbb{Out}^{N} = \{\langle 0,0,\ldots,0 \rangle, \langle 1,1,\ldots,1 \rangle, \ldots\}$$
gdzie $\mathcal{T}(\langle 0,0,\ldots,0 \rangle) = \{\langle 0,0,\ldots,0 \rangle\}$ oraz $\mathcal{T}(\langle \ldots,5,\ldots \rangle) = \{\ldots, \langle 5,5,\ldots,5 \rangle, \ldots\}$,
- **elekcja** — jeden proces decyduje 1, a pozostałe decydują 0:
$$\mathbb{Out}^{N} = \{\langle 1,0,\ldots,0 \rangle, \langle 0,1,\ldots,0 \rangle, \ldots, \langle 0,0,\ldots,1 \rangle\}$$

> **Def.** (slajd 38) Algorytm $A$ jest ***$f$-crash resilient solution*** dla zadania $\mathcal{T}$, jeżeli spełnia:
> **Termination:** w każdym $f$-crash *fair execution* wszystkie $P_c$ decydują.
> **Consistency:** jeśli wszystkie $P_i$ są poprawne, wektor decyzji $O$ należy do $\mathcal{T}(I)$.

---
## Bibliografia
<sub>slajd 39</sub>

1. M. J. Fischer, N. A. Lynch, M. S. Paterson, *Impossibility of distributed consensus with one faulty process*, Journal of the ACM no. 32, 1985, pp. 374–382.
2. L. Lamport, *Paxos made simple*, ACM SIGACT News no. 32, vol. 4, 2001, pp. 51–58.
3. M. Raynal, *Fault-Tolerant Message-Passing Distributed Systems*, Springer, 2018, ch. 10, 12.
4. G. Tel, *Introduction to Distributed Algorithms*, Cambridge University Press, 2000, ch. 14.

---
## Powiązania i braki

Kontynuacja tematu: [[SWN 06 Awarie bizantyjskie]] (algorytmy dla BA), [[SWN 07 Konsensus]] (Paxos, rozwiązania probabilistyczne), [[SWN 08 Detektory awarii i replikacja procesu]] (obejście FLP przez detektory awarii).

> [!note] Uzupełnienie spoza slajdów
> [[23 Rozproszone uzgadnianie w środowisku zawodnym]] (wersja pierwsza notatek) podaje **klasyfikację modeli awarii** (fail-stop, crash, fail-recovery, omission, timing, bizantyjskie, bizantyjskie z uwierzytelnianiem) oraz pojęcia **jednolitej zgodności** (*uniform agreement*) i **częściowej synchronii** (GST). Slajd 7 tego wykładu wymienia tylko trzy rodzaje awarii (stopping, bizantyjska, awaria łącza), a slajd 6 definiuje wyłącznie model synchroniczny.

> [!todo] Czego nie ma na slajdach tego wykładu
> - **Dowodu twierdzenia FLP** — podany jest tylko wynik i intuicja („nie odróżnimy procesu martwego od wolnego").
> - **Odpowiedzi** na pytanie ze slajdu 17 („czy te transformacje działają przy awariach?").
> - **Definicji knota** (*knot*) w grafie skierowanym — pojęcie używane od slajdu 27 bez definicji.
> - **Procedury `compute_knot()`** z algorytmu FLP — na slajdzie 25 jest tylko wywołanie.
> - **Złożoności** dla pozostałych problemów (podana tylko $O(N^2)$ dla algorytmu FLP).
