---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
source: "Slajdy-FT-10_ByzantineFailures.pdf"
slajdy: "1–44"
zagadnienie: 23
---
# SWN 06. Awarie bizantyjskie
---
> Wykład o **najgorszym modelu awarii** — procesach zachowujących się arbitralnie (złośliwie). Stawia **problem bizantyjskich generałów**, dowodzi, że **nie ma rozwiązania dla $f \geqslant \frac{1}{3}N$** (nawet w systemie synchronicznym), pokazuje, że **przybliżone uzgodnienie nie jest łatwiejsze**, i podaje dwa algorytmy Lamporta-Shostaka-Pease'a: **OM** dla komunikatów ustnych ($\mathbb{S}^{Sync}\{\varnothing\}$) i **SM** dla komunikatów podpisanych ($\mathbb{S}^{Sync}\{C\}$). Ostatni znosi ograniczenie $N > 3f$ i redukuje złożoność z $\Omega(N^{f+1})$ do $O(N^2)$.

---
## Bizantyjscy generałowie
<sub>Slajdy-FT-10_ByzantineFailures.pdf, slajdy 1–3</sub>

Wykład zaczyna się od bajki (slajdy 1–3): armie otaczające twierdzę, ale **niektórzy generałowie są zdrajcami** — na slajdzie 3 zdrajca wysyła jednemu sąsiadowi „ATTACK", a drugiemu „RETREAT", myśląc „heh-heh".

### Protokół ad-hoc i istota problemu
<sub>slajdy 4 i 28 (slajd 28 to powtórka slajdu 4 z etykietą REPLAY)</sub>

**Protokół ad-hoc** (w środowisku bezawaryjnym):
- każdy generał $G_i$ podejmuje decyzję $v_i$ i ją rozgłasza,
- generałowie osiągają porozumienie na podstawie sumy zebranych opinii (większość ATTACK / większość RETREAT; a jeśli są podzieleni — *„hmm… kogo to obchodzi?"*),
- może wymagać kilku rund.

**Awarie bizantyjskie:**
- w czym problem? … **zdrajcy!**
- jeśli otrzymujesz $v_j$ od $G_j$, **nie możesz mu ufać**,
- musisz być ostrożny, bo **nie wiesz, komu ufać**,
- **intuicyjnie oczywiste jest, że dopóki większość węzłów nie jest uczciwa, nic użytecznego nie da się zrobić**.

> **Dymek ze slajdu 28 (REPLAY):** *traitors can forge messages* — zdolność zdrajcy do **fałszowania** komunikatów jest osią całego wykładu; jej odebranie (podpisy cyfrowe) zmienia wszystkie wyniki.

### Typy awarii i przemilczenia
<sub>slajd 5</sub>

| Typ awarii | Model |
|---|---|
| **crash failures** | \} **fail-stop model** |
| **omission failures** | \} |
| **Byzantine (malicious) failures** | — |

**Przemilczenia w modelu bizantyjskich generałów:**

> Ponieważ wadliwy generał (**zdrajca**) może odmówić wysłania komunikatu, niewadliwy generał może nigdy nie otrzymać oczekiwanego komunikatu. W takiej sytuacji zakładamy, że niewadliwy generał (**lojalny**) po prostu **wybiera dowolną wartość** i działa tak, jakby oczekiwany komunikat został odebrany.
>
> Oczywiście wymagamy, by takie przemilczenia **dało się wykryć** przez odbiorcę. W systemach synchronicznych, gdzie czas trwania każdej rundy jest znany, detekcja jest prosta — wszystkie oczekiwane komunikaty nieotrzymane do końca rundy **nie zostały wysłane** (*omitted*).

---
## Porozumienie bizantyjskie (BA)
<sub>slajd 6</sub>

**Dowodzący generał $G_0$ wysyła rozkaz do $N-1$ generałów-poruczników** i:

> **Agreement:** wszyscy lojalni porucznicy wykonują **ten sam** rozkaz.
> **Validity:** jeśli dowódca jest lojalny, to **każdy lojalny porucznik wykonuje rozkaz, który dowódca wysłał**.

Na razie pomijamy:
> *Termination: wszyscy lojalni porucznicy w końcu przyjmują rozkaz.*

> ⚠ **Uwaga ze slajdu:** jeśli $G_0$ jest lojalny, to **Agreement wynika bezpośrednio z Validity**. Trudność leży więc wyłącznie w przypadku **nielojalnego dowódcy**.

---
## Niemożliwość BA dla $f \geqslant \frac{1}{3}N$
<sub>slajdy 7–10</sub>

> **Claim.** Żadne rozwiązanie dla mniej niż $3f+1$ generałów nie poradzi sobie z $f$ zdrajcami (tj. rozwiązanie jest niemożliwe, jeśli $f \geqslant \frac{1}{3}N$ generałów to zdrajcy).

Najpierw pokazujemy, że **nie ma rozwiązania dla 3 generałów z 1 zdrajcą**.

![[swn-ft10-s08-trzej-generalowie-zdrajca-dowodca.png]]
<sub>Przypadek 1 (slajd 8): **dowódca jest zdrajcą** — wysyła „1 (ATTACK)" jednemu porucznikowi i „0 (RETREAT)" drugiemu; lojalny porucznik przekazuje „he said 0 (RETREAT)", a pierwszy porucznik nie wie, co robić (???).</sub>

![[swn-ft10-s09-trzej-generalowie-zdrajca-porucznik.png]]
<sub>Przypadek 2 (slajd 9): **dowódca jest lojalny** — wysyła „1 (ATTACK)" obu; ale zdrajca-porucznik kłamie: „he said 0 (RETREAT)". Lojalny porucznik znów widzi ??? — **dokładnie to samo, co w przypadku 1**.</sub>

Oba przypadki są dla lojalnego porucznika **nieodróżnialne**: w pierwszym musi zignorować rozkaz dowódcy (bo dowódca kłamie), w drugim musi go wykonać (bo dowódca jest lojalny). Nie da się spełnić jednocześnie Agreement i Validity.

### Wynik niemożliwości i strategia dowodu
<sub>slajd 10</sub>

> **Nie istnieje $f$-odporny algorytm Byzantine Agreement w $\mathbb{S}^{Sync}$ dla $f \geqslant \frac{1}{3}N$.**

**Strategia dowodu (przez symulację):**
- załóżmy, że mamy algorytm $A_f$ dla $3f$ generałów z $f > 1$ zdrajcami,
- konstruujemy algorytm $A_1$ dla 3 generałów i 1 zdrajcy za pomocą transformacji:
  - każdy generał **„symuluje" $f$ generałów** w $A_f$,
  - 1 zdrajca „symuluje" $f$ zdrajców,
  - 2 lojalnych generałów „symuluje" $2f$ lojalnych generałów,
  - zarówno **Agreement**, jak i **Validity** są spełnione,
- ale to jest **niemożliwe** (pokazane wyżej), więc nasze założenie jest błędne. $\blacksquare$

---
## Przybliżone uzgodnienie (AA)
<sub>slajdy 11–14</sub>

*Może trudność polega na wymaganiu **dokładnego** uzgodnienia?*

**Nowy problem:** $G_0$ wysyła **CZAS** ataku do $N-1$ poruczników:

> **Agreement:** wszyscy lojalni porucznicy atakują **w odstępie nie większym niż 10 minut** od siebie.
> **Validity:** jeśli dowódca jest lojalny, każdy lojalny porucznik atakuje **w ciągu 10 minut** od rozkazu dowódcy.

**Pytanie:** czy ten problem uzgadniania jest łatwiejszy?

> **AA Impossibility Result** (slajd 12): **Nie istnieje $f$-odporny algorytm Approximate Agreement w $\mathbb{S}^{Sync}$ dla $f \geqslant \frac{1}{3}N$.**

**Strategia dowodu:** załóżmy, że mamy algorytm AA dla 3 generałów z 1 zdrajcą; **przekształcimy go w algorytm BA** dla 3 generałów z 1 zdrajcą.

### Transformacja AA → BA
<sub>slajd 13</sub>

Niech $G_0$ wysyła: **1:00** oznacza ATTACK, **2:00** oznacza RETREAT. Każdy porucznik wykonuje:

**Faza 1:**
- uruchom protokół Approximate Agreement,
- jeśli uzgodniony czas jest **przed 1:10**, zdecyduj **ATTACK**,
- jeśli uzgodniony czas jest **po 1:50**, zdecyduj **RETREAT**.

**Faza 2:**
- jeśli nie doszedłeś do żadnej decyzji,
- zapytaj drugiego porucznika: *„Have you decided?"*,
- jeśli tak — **zrób to samo**,
- jeśli nie $\rightarrow$ **RETREAT**.

### Dowód poprawności transformacji
<sub>slajd 14</sub>

> **Claim.** Jeśli protokół Approximate Agreement działa, to działa również protokół Byzantine Agreement.

- jeśli $G_0$ jest lojalny, to **AA Validity** gwarantuje, że lojalny porucznik dostaje właściwy czas ataku, więc **BA Validity** jest spełnione (a z nim Agreement),
- jeśli $G_0$ jest zdrajcą, obaj porucznicy są lojalni; **AA Agreement** gwarantuje, że **nie mogą podjąć sprzecznych decyzji** w fazie 1,
- jeśli jeden zdecyduje w fazie 1, drugi zgadza się **najpóźniej w fazie 2**,
- jeśli żaden nie zdecyduje w fazie 1, **obaj wycofują się** w fazie 2.

Zatem Approximate Agreement jest **niemożliwe** dla 3 generałów z 1 zdrajcą. Metoda symulacji działa dla $3f$ generałów i $f$ zdrajców. $\blacksquare$

---
## Algorytmy BA — dwa rodzaje komunikatów
<sub>slajdy 15–16</sub>

> *Aby osiągnąć porozumienie, procesy muszą wymieniać swoje wartości i wielokrotnie przekazywać otrzymane wartości innym procesom. **Zdolność wadliwego procesu do zniekształcania tego, co otrzymuje od innych, w ogromnym stopniu zależy od typu komunikatów.***

- **komunikaty ustne** (*oral, non-authenticated*) — wadliwy proces może **sfałszować** komunikat i twierdzić, że otrzymał go od innego procesu, albo **zmienić zawartość** otrzymanego komunikatu przed przekazaniem go dalej. **Nie ma sposobu, by proces zweryfikował autentyczność** otrzymanego komunikatu.
- **komunikaty podpisane** (*signed, authenticated*) — wadliwy proces **nie może sfałszować** komunikatu ani zmienić zawartości otrzymanego przed przekazaniem dalej. Każdy proces może **zweryfikować autentyczność** otrzymanego komunikatu. **Wadliwe procesy wyrządzają mniej szkody.**

**Model systemu** (slajd 16): niech $\mathbb{S}$ będzie systemem ($\mathbb{S}^{Sync}$ = synchroniczny, $\mathbb{S}^{Async}$ = asynchroniczny). Przez $\mathbb{S}\{M\}$ oznaczamy system $\mathbb{S}$ **wzbogacony o dodatkowy mechanizm $M$**.
- komunikaty **ustne** są dostępne w $\mathbb{S}^{Sync}\{\varnothing\}$,
- komunikaty **podpisane** są dostępne w $\mathbb{S}^{Sync}\{C\}$, gdzie $C$ to kryptograficzny podpis cyfrowy.

---
## BA z komunikatami ustnymi — algorytm OM

### Założenia
<sub>slajd 17</sub>

Proste rozwiązanie z użyciem komunikatów „ustnych" w $\mathbb{S}^{Sync}\{\varnothing\}$, radzące sobie z $f$ zdrajcami, gdzie $\mathbf{f < \frac{1}{3}N}$.

**Sieć jest niezawodna =**
- **A1:** każdy wysłany komunikat jest dostarczony poprawnie,
- **A2:** odbiorca komunikatu wie, kto go wysłał,
- **A3:** brak komunikatu może zostać wykryty.

*A1 i A2 uniemożliwiają zdrajcy zakłócanie komunikacji między innymi (A2 = brak fałszywych komunikatów). A3 oznacza, że zdrajca nie może po prostu milczeć, aby zablokować postęp.*

### Szczegóły i funkcja *majority*
<sub>slajd 18</sub>

**Dodatkowe wymagania:**
- generałowie komunikują się **bezpośrednio** ze sobą (choć to można łatwo rozluźnić),
- jeśli komunikatu brakuje, **zakłada się, że mówi 0** (domyślny rozkaz to RETREAT).

**Funkcja *majority*:**
- jeśli więcej niż $\frac{n}{2}$ wartości $v_i$ równa się $v$, to $majority(v_1, \ldots, v_n) = v$,
- jeśli większość $v_i$ nie istnieje: $majority(v_1, \ldots, v_n) = 0$,
- $n$ to liczba procesów aktualnie osiągających porozumienie.

### Algorytm OM (Lamport-Shostak-Pease)
<sub>slajd 19</sub>

```
OM(0):
  1. G_c sends his value to every G_i
  2. Each lieutenant uses that value (or 0 if none)

OM(t), t>0:
  1. G_c sends his value to every G_i
  2. Let v_i be the value received by G_i
     G_i now
       - acts as commander for OM(t-1) with n-2 other lieutenants
       - acts as participant for n-2 instances of OM(t-1),
         with each G_j (i != j) as commander, deciding v_j (or 0)
  3. decide majority(v_1, ..., v_{n-1})
```

### Opis i złożoność
<sub>slajd 20</sub>

- algorytm dla $N$ procesów startuje od $OM(f)$,
- wykonanie $OM(f)$ wywołuje $N-1$ oddzielnych wykonań $OM(f-1)$, z których każde wywołuje $N-2$ wykonań $OM(f-2)$ i tak dalej,
- procesy są sukcesywnie dzielone na coraz mniejsze grupy $n$ członków (początkowo $n = N$), a porozumienie bizantyjskie jest **rekurencyjnie** osiągane w każdej grupie w kroku 2 $OM(t)$,
- w sumie jest $1 \cdot (N-1)(N-2)(N-3)\ldots(N-f)$ oddzielnych wykonań $OM(t)$ dla $t = f, f-1, f-2, \ldots, 0$,
- **złożoność komunikacyjna wynosi $\Omega(N^{f+1})$**,
- algorytm wymaga **$f+1$ rund** wymiany komunikatów,
- **$f+1$ jest dolnym ograniczeniem** liczby rund potrzebnych do osiągnięcia Byzantine Agreement w sieci w pełni połączonej z awariami procesów w $\mathbb{S}^{Sync}\{\varnothing\}$.

### Przykłady
<sub>slajdy 21–23</sub>

![[swn-ft10-s21-om-przyklad1.png]]
<sub>**Przykład 1** ($OM(1)$, $N=4$, zdrajcą jest **porucznik** $G_2$): w kroku 1 lojalny $G_0$ wysyła 0 do wszystkich; w kroku 2 każdy porucznik wymienia otrzymane wartości ($G_2$ kłamie, wysyłając 1 do $G_1$); w kroku 3 $G_1$ liczy $majority(0,1,0) = 0$, a $G_3$ liczy $majority(0,0,0) = 0$ — slajd 21</sub>

![[swn-ft10-s22-om-przyklad2.png]]
<sub>**Przykład 2** (zdrajcą jest **dowódca** $G_0$): wysyła 1 do $G_1$, 0 do $G_2$ i **nic** (null) do $G_3$ — brakujący komunikat liczy się jako 0. Po wymianie wszyscy trzej lojalni porucznicy liczą $majority(1,0,0) = 0$ — **uzgodnienie osiągnięte**, slajd 22</sub>

![[swn-ft10-s23-om-przyklad3-drzewo.png]]
<sub>**Przykład 3** ($OM(2)$, $N=7$, „gdzieś jest 2 zdrajców"): drzewo rekurencji $OM(2) \rightarrow OM(1) \rightarrow OM(0)$ — slajd 23</sub>

### Dowód poprawności — lemat
<sub>slajdy 24–25</sub>

> *Ponieważ nielojalny dowódca sprawia więcej problemów niż jakikolwiek nielojalny porucznik, rozważamy te przypadki osobno.*

> **Lemat.** Dla dowolnych $t$ i $f$, $OM(t)$ spełnia **Validity**, jeśli jest **więcej niż $2f + t$** generałów i **co najwyżej $f$** zdrajców.

**Dowód lematu:**
- rozważamy przypadek **lojalnego dowódcy**,
- $OM(t=0)$ trywialnie spełnia Validity (z założenia A1),
- indukcja dla $t > 0$: zakładamy lemat dla $OM(t-1)$ i dowodzimy dla $OM(t)$.

**Krok indukcyjny dla $OM(t)$, $t > 0$** (slajd 25):
- lojalny dowódca wysyła $v$ do $n-1$ poruczników,
- każdy lojalny porucznik używa $v$, wywołując $OM(t-1)$ z $n-2$ innymi porucznikami,
- z hipotezy lematu: $n > 2f + t$,
- zatem $(n-1) > 2f + (t-1)$,
- stosujemy **hipotezę indukcyjną**: $OM(t-1)$ spełnia Validity,
- gdy lojalny $G_i$ jest dowódcą w $OM(t-1)$,
- każdy lojalny porucznik $G_j$ dostaje to samo $v_i$ (od tego lojalnego $G_i$) i używa go w $OM(t)$,
- jest co najwyżej $f$ zdrajców,
- oraz $(n-1) > 2f + (t-1) \geqslant 2f$,
- zatem **większość** z $n-1$ poruczników w $OM(t)$ jest lojalna,
- każdy lojalny porucznik dostaje to samo $majority(v_1, \ldots, v_{n-1})$ w kroku 3 $OM(t)$. $\blacksquare$

### Twierdzenie
<sub>slajdy 26–27</sub>

> **Twierdzenie.** Dla dowolnego $f$, $OM(f)$ spełnia **Agreement** i **Validity**, jeśli jest **więcej niż $3f$** generałów i **co najwyżej $f$** zdrajców.

**Dowód (przez indukcję po $f$):**
- jeśli nie ma zdrajców — $OM(0)$ — łatwe,
- zakładamy tezę dla $OM(f-1)$, $f > 0$,
- **zakładamy lojalnego dowódcę:** bierzemy $t = f$ w poprzednim lemacie; $OM(f)$ spełnia Validity z lematu; Validity implikuje Agreement, gdy dowódca jest lojalny,
- **zakładamy, że dowódca jest zdrajcą** (slajd 27):
  - co najwyżej $f$ zdrajców, **włącznie z dowódcą**,
  - zatem co najwyżej $f-1$ zdrajców **wśród poruczników**,
  - skoro $n > 3f$, poruczników jest więcej niż $3f - 1 > 3(f-1)$,
  - stosujemy **hipotezę indukcyjną**: $OM(f-1)$ spełnia Agreement i Validity,
  - zatem dla wszystkich $i$, dowolni dwaj lojalni porucznicy $G_j$, $G_k$ dostają tę samą decyzję $v_i$ z $OM(f-1)$ na końcu kroku 2 $OM(f)$,
  - wynika to z **Agreement** $OM(f-1)$,
  - stąd dowolni dwaj lojalni porucznicy dostają ten sam wektor $v_1, \ldots, v_{n-1}$ w kroku 3 $OM(f)$,
  - zatem dowolni dwaj lojalni porucznicy dostają to samo $majority(v_1, \ldots, v_{n-1})$ z $OM(f)$. $\blacksquare$

---
## BA z komunikatami podpisanymi — algorytm SM

### Założenia
<sub>slajd 29</sub>

Mamy $\mathbb{S}^{Sync}\{C\}$, gdzie $C$ to narzędzie kryptograficzne zdolne **cyfrowo podpisywać** komunikaty. Zapis $v{:}i$ = wartość $v$ podpisana przez $G_i$.

Do A1–A3 dochodzi:
- **A4:** podpis cyfrowy $G_i$ **nie może zostać sfałszowany** (każda zmiana komunikatu może zostać wykryta), a **każdy może zweryfikować** autentyczność podpisu.

> **Pytanie ze slajdu:** *Czy w $\mathbb{S}^{Sync}\{C\}$ możemy uzyskać lepsze wyniki niż w $\mathbb{S}^{Sync}\{\varnothing\}$?*

### Algorytm SM
<sub>slajdy 30–31</sub>

- $G_0$ podejmuje decyzję $v$ i wysyła **podpisany rozkaz** $v{:}0$ (runda 0),
- każdy $G_i$ **dodaje własny podpis** do tego rozkazu i wysyła go do pozostałych poruczników (runda 1), którzy dodają swój podpis i wysyłają dalej (runda 2) itd.,
- **ważny rozkaz** (tj. poprawnie podpisany) w rundzie $k$ ma postać $v{:}0{:}i_1{:}\ldots{:}i_k$,
- każdy $G_i$ propaguje **tylko ważne rozkazy**,
- **$f+1$ rund** propagacji rozkazu: od fazy 0 do rundy $f$,
- w ostatniej rundzie: spośród $f+1$ podpisów w $v{:}0{:}i_1{:}\ldots{:}i_f$ **co najmniej jeden pochodzi od procesu poprawnego**.

Dla zbioru $V_i$ ważnych rozkazów:
$$\texttt{choice}(V_i) = \begin{cases} v & \text{jeśli } v \text{ jest jedyną decyzją w } V_i, \text{ tj. } V_i = \{v\} \\ 0 & \text{w przeciwnym razie (RETREAT)} \end{cases}$$

```
For G_i:
    V_i = {}
    round 0:
        G_0 broadcasts v:0 to all processes;
        receive v:0
        if v:0 is valid then V_i = V_i + {v}
    round k = 1..f:
        while v':0:j_1:...:j_{k-1} is a new value received in round k-1
            send v':0:j_1:...:j_{k-1}:i to every lieutenant except G_{j_1},...,G_{j_{k-1}}
        while receive v:0:j_1:...:j_k
            if v:0:j_1:...:j_k is valid then V_i = V_i + {v}
    v = choice(V_i)
```

### Przykład: $N=3$, $f=1$, zdrajcą jest $G_0$
<sub>slajdy 32–33</sub>

W rundzie 0 $G_0$ wysyła $0{:}0$ do $G_1$ i $1{:}0$ do $G_2$.

![[swn-ft10-s33-sm-runda1.png]]
<sub>Runda 1: $G_1$ przekazuje $0{:}0{:}1$, $G_2$ przekazuje $1{:}0{:}2$ — slajd 33</sub>

> $G_1$ i $G_2$ **wiedzą, że $G_0$ jest zdrajcą** — z A4 wynika, że **tylko on mógł wydać sprzeczne rozkazy**.

Wynik: $V_1 = V_2 = \{0, 1\}$, więc $\texttt{choice}(V_i) = 0$ — **uzgodnienie osiągnięte mimo $N = 3$ i $f = 1$**, co było niemożliwe dla komunikatów ustnych.

### Ile zdrajców?
<sub>slajd 34</sub>

- **formalnie dowolne $f < N$**,
- oczywiście rozsądne jest to tylko dla $\mathbf{N \geqslant f + 2}$ (potrzeba co najmniej **dwóch lojalnych** poruczników, żeby „wspólna decyzja" miała sens).

> [!important] To jest sedno przewagi podpisów
> Ograniczenie $N > 3f$ z algorytmu OM **znika**. Podpis cyfrowy odbiera zdrajcy zdolność fałszowania cudzych komunikatów, więc wystarczy, by **jeden** podpis w łańcuchu pochodził od procesu poprawnego.

### Czy naprawdę $f+1$ rund?
<sub>slajdy 35–40</sub>

Przykład dla $N = 4$, $f = 2$, zdrajcami są $G_0$ i $G_3$.

- **runda 0** (slajdy 35–36): $G_0$ wysyła $1{:}0$ do $G_1$ i $G_2$ oraz $0{:}0$ do $G_3$; stąd $V_1 = \{1\}$, $V_2 = \{1\}$, $V_3 = \{0\}$,

![[swn-ft10-s37-sm-fplus1-rund-runda1.png]]
<sub>**Runda 1** (slajd 37): $G_1$ rozsyła $1{:}0{:}1$, $G_2$ rozsyła $1{:}0{:}2$, $G_3$ wysyła $0{:}0{:}3$ **tylko do $G_2$** — *sends nothing to $G_1$*. Przypis: ***Here, if a message is missing, don't assume it says 0! Only signed orders matter!*** — slajd 37</sub>

- po rundzie 1 (slajd 38): $V_1 = \{1\}$, ale $V_2 = \{1, 0\}$ — **stan niezgodny**,
- **runda 2** (slajdy 39–40): $G_2$ przekazuje $0{:}0{:}3{:}2$ do $G_1$; teraz $V_1 = \{1,0\}$ i $V_2 = \{1,0\}$, więc $\texttt{choice}(V_i) = 0$ dla obu.

![[swn-ft10-s40-sm-runda2-decyzja.png]]
<sub>Runda 2 — po dostarczeniu $0{:}0{:}3{:}2$ obaj lojalni porucznicy mają $V = \{1,0\}$ i decydują 0 — slajd 40</sub>

**Odpowiedź:** tak, **$f+1$ rund jest konieczne** — po rundzie 1 (czyli $f$ rund dla $f = 2$… właściwie po drugiej z $f+1 = 3$ faz) stan był jeszcze niezgodny.

### Złożoność komunikacyjna
<sub>slajd 41</sub>

- **$f+1$ rund** ($f < N$),
- **$O(N^2)$**:
  - runda 0: $(N-1)$ komunikatów,
  - runda 1: $(N-1)(N-1)$ komunikatów,
  - runda 2: co najwyżej $(N-1)(N-2)$ komunikatów,
  - …
  - runda $f$: co najwyżej $(N-1)(N-f)$ komunikatów,
  - **w sumie: co najwyżej $(N-1) + f(N-1)^2$ komunikatów $\rightarrow O(N^2)$**.

### Optymalizacja
<sub>slajdy 42–43</sub>

> **$G_i$ przekazuje dalej tylko pierwsze dwa ważne komunikaty niosące różne wartości.**

**Uzasadnienie:**
- jeśli $G_0$ jest poprawny, wysłał **tę samą** podpisaną wartość $v$ do wszystkich procesów, które mogą ją tylko przekazywać (po dołączeniu swoich podpisów) — bo **żaden proces nie może sfałszować podpisu innego procesu**; stąd poprawny $G_i$ może odebrać **tylko jedną** ważną wartość $v$,
- jeśli $G_0$ jest wadliwy, mógł wysłać różne podpisane wartości do różnych procesów i żadnego komunikatu do innych — jeśli $G_i$ odbiera kilka **ważnych** komunikatów niosących różne wartości, **wie, że pochodzą od dowódcy $G_0$ i że $G_0$ jest wadliwy**. Aby uświadomić to innym poprawnym procesom, $G_i$ musi przekazać (po podpisaniu) **tylko dwa** komunikaty niosące różne wartości.

**Złożoność po optymalizacji** (slajd 43):
- nadal **$f+1$ rund** ($f < N$) i nadal $O(N^2)$, ale:
  - runda 0: $(N-1)$ komunikatów,
  - runda 1: $(N-1)(N-1)$ komunikatów,
  - rundy $2..f$: co najwyżej $(N-1)(N-?)$ komunikatów,
  - **w sumie co najwyżej $(N-1) + 2(N-1)^2$** komunikatów — jeśli dowódca jest wadliwy,
  - i **co najwyżej $(N-1) + (N-1)^2$** komunikatów, jeśli dowódca jest poprawny.

---
## Porównanie OM i SM

| | **OM** (komunikaty ustne) | **SM** (komunikaty podpisane) |
|---|---|---|
| Model | $\mathbb{S}^{Sync}\{\varnothing\}$ | $\mathbb{S}^{Sync}\{C\}$ |
| Warunek na $f$ | $f < \frac{1}{3}N$ (tj. $N > 3f$) | formalnie $f < N$; rozsądnie $N \geqslant f+2$ |
| Rundy | $f+1$ | $f+1$ |
| Komunikaty | $\Omega(N^{f+1})$ | $O(N^2)$; po optymalizacji $\leqslant (N-1) + 2(N-1)^2$ |
| Brakujący komunikat | traktowany jako **0** | **ignorowany** — liczą się tylko podpisane rozkazy |
| Decyzja | $majority(v_1,\ldots,v_{n-1})$ | $\texttt{choice}(V_i)$ |

---
## Bibliografia
<sub>slajd 44</sub>

1. L. Lamport, R. Shostak, M. Pease, *The Byzantine Generals Problem*, ACM Transactions on Programming Languages and Systems, 1982.
2. M. Raynal, *Fault-Tolerant Message-Passing Distributed Systems*, Springer, 2018, ch. 14.
3. A. D. Kshemkalyani, M. Singhal, *Distributed Computing — Principles, Algorithms, and Systems*, Cambridge University Press, 2008, ch. 14.

---
## Powiązania i braki

Poprzedni wykład: [[SWN 05 Problemy uzgadniania i wyniki niemożliwości]] (definicja BA i jego relacje z konsensusem). Dalej: [[SWN 07 Konsensus]] — konsensus procesów bizantyjskich (algorytm Phase-King).

> [!warning] Rozbieżność z notatką w vaultcie
> [[Systemy Wysokiej Niezawodności/Błąd Bizantyjski]] to **jedno zdanie**. Cały materiał tego wykładu (OM, SM, warunek $N > 3f$, *approximate agreement*, złożoności) w vaultcie **nie występuje** — ta notatka domyka poz. „Problem bizantyjskich generałów – Lamport-Shostak-Pease, algorytm OM(m), warunek N > 3f" z [[Braki w notatkach]].

> [!todo] Czego nie ma na slajdach tego wykładu
> - **Dowodu poprawności algorytmu SM** — podany jest tylko przykład i uzasadnienie optymalizacji.
> - **Dowodu dolnego ograniczenia $f+1$ rund** — slajd 20 podaje je jako fakt, slajdy 35–40 tylko ilustrują przykładem.
> - **Algorytmu Phase-King** — mimo że dotyczy konsensusu bizantyjskiego, pojawia się dopiero w [[SWN 07 Konsensus]]; por. [[Systemy Wysokiej Niezawodności/Algorytm Phase-King]].
> - **Wariantu asynchronicznego** — cały wykład operuje w $\mathbb{S}^{Sync}$; BA w $\mathbb{S}^{Async}$ nie jest omawiany.
