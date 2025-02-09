---
tags:
  - SystemyWysokiejNiezawodności
up:
---
# Algorytm Koo-Touega
---
> Metoda tworzenia **synchronicznych punktów kontrolnych** (_ang. Coordinated Checkpointing_) i **odtwarzania** z nich przetwarzania rozproszonego. Wykorzystuje ideę przetwarzania dyfuzyjnego i 2PC. #TODO co to to dyfuzyjne.
> 
> Stosuje dwa rodzaje punktów kontrolnych:
> - **ostateczny** (_ang. permanent checkpoint_) - należący do $CP^\bullet$, tylko taki może być punktem odtwarzania
> - **wstępny** (_ang. tentative checkpoint_) - punkt, który jeszcze nie stał się **ostatecznym**
> 
> Punkty kontrolne obu rodzajów zapisywane są do pamięci trwałej, a jednocześnie istnieje tylko jeden punkt **ostateczny** dla każdego procesu.

## Założenia
 - przetwarzanie może być niedeterministyczne
 - model awarii _fail-recovery_: procesy podlegają tymczasowym awariom typu zatrzymanie;
co więcej – pozostałe procesy w skończonym czasie dowiadują się o awariach
 - niezawodne kanały FIFO, w szczególności:
	 - sieć niepodzielna na rozłączne części
	 - za retransmisję wiadomości odpowiada podsystem komunikacyjny
 - ewentualne interakcje ze światem zewnętrznym obsługuje dodatkowy protokół _output commit_

## Wykorzystywane struktury
 - **$m.l$** ($m label$) – etykieta wiadomości $m$ (monotoniczny licznik wysyłanych wiadomości) minimalna etykieta = $\bot$ maksymalna = $\top$
 - **$last\_label\_rcvd_i[j]$** przechowuje etykietę ostatniej wiadomości $m$ wysłanej przez $P_j$ i odebranej przez $P_i$ od ostatniego punktu kontrolnego:
$$
\text{last\_label\_rcvd}_i[j] = \begin{cases} 
m.l & \text{jeśli m istnieje} \\
\bot & \\
\end{cases}
$$
- **$first\_label\_sent_i[j]$** przechowuje etykietę pierwszej wiadomości $m$ wysłanej przez $P_i$ do $P_j$ od ostatniego punktu kontrolnego:
$$
\text{first\_label\_sent}_i[j] = \begin{cases} 
m.l & \text{jeśli m istnieje} \\
\bot & \\
\end{cases}
$$
- **$last\_label\_sent_i[j]$** przechowuje etykietę ostatniej wiadomości $m$ wysłanej przez $P_i$ do $P_j$ **przed** ostatnim punktem kontrolnym:
$$
\text{last\_label\_sent}_i[j] = \begin{cases}
m.l & \text{jeśli m istnieje} \\
\top
\end{cases}
$$
## Algorytm tworzenia _checkpoint_-ów
**Faza 1**:
>inicjator $C_i$ (kontroler procesu $P_i$) ustala wstępny $cp_i$ i rozsyła żądanie wyznaczenia $cp_j$ pozostałym $C_j$.

**Faza 2**:
>jeśli $C_i$ otrzymał wszystkie potwierdzenia pozytywne, rozsyła decyzję zamiany wstępnych punktów kontrolnych na ostateczne
>$C_j$ nie wysyła żadnych wiadomości $P_j$ dopóki nie otrzyma decyzji

> [!info] Optymalizacja tworzenia punktów kontrolnych
> Jeśli $C_i$ wysyła do $C_j$ żądanie $take\_a\_tentative\_ckpt$($C_i, C_j, \text{last\_label\_rcvd}_i[j]$),
to $C_j$ wyznacza wstępny punkt kontrolny procesu $P_j$ tylko gdy:
>$$
\text{last\_label\_rcvd}_i[j] \geq \text{first\_label\_sent}_j[i] > \bot
>$$
>Żądanie jest wysyłane tylko do $C_j \in \text{ckpt\_cohort}_i = \{ C_j : \text{last\_label\_rcvd}_i[j] > \bot \}$. Inaczej mówiąc wysyła tylko do tych od których otrzymał jakąś wiadomość od czasu ostatniego _checkpoint_-u.

>[!tip]
>$C_j$ od razu zwraca potwierdzenie pozytywne gdy nie musi tworzyć **wstępnego** punktu kontrolnego lub utworzył go wcześniej. Jeśli okazuje się, że powinien utworzyć **wstępny** punkt kontrolny, wykonuje proces od **fazy 1**. Nie wysyła jednak żądania do $C_i$ od którego to żądanie dostał. 
## Algorytm odtwarzanie
**Faza 1**:
> inicjator $C_i$ wysyła pytanie o zgodę na wycofanie do poprzedniego $cp_j$

**Faza 2**:
> jeśli $C_i$ otrzymał wszystkie potwierdzenia pozytywne, rozsyła decyzję wycofania do wszystkich procesów.
> $C_j$ nie wysyła żadnych wiadomości dopóki nie otrzyma decyzji

> [!info] Optymalizacja odtwarzania
> Jeśli $C_i$ wysyła do $C_j$ żądanie $prepare\_to\_rollback$($C_i$, $C_j$, $last\_label\_sent_i[j]$),
to $C_j$ wycofa $P_j$ do ostatniego punktu kontrolnego tylko gdy:
>$$
\text{last\_label\_rcvd}_j[i] > \text{last\_label\_sent}_i[j]
>$$
>Spełnienie tego warunku wskazuje, że $P_i$ wycofuje się do stanu, w którym zostaną anulowane zdarzenia wysłania co najmniej jednej wiadomości odebranej już przez $P_j$.
>
$roll\_cohort_i$ = $C_j$ : $P_i$ może wysyłać wiadomości do $P_j$, potencjalnie każdego procesu z którym $P_i$ mógł się komunikować.