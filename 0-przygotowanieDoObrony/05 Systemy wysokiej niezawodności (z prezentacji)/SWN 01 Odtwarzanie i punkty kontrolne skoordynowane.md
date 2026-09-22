---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
source: "Slajdy-FT-02_Recovery1.pdf"
slajdy: "1–36"
zagadnienie: 22
---
# SWN 01. Odtwarzanie stanu cz. I — punkty kontrolne i checkpointing skoordynowany
---
> Wykład otwiera zagadnienie **wstecznego odtwarzania stanu**: od klasyfikacji awarii i modelu węzła, przez odtwarzanie pojedynczego węzła (wycofywanie operacji vs przywracanie stanu), po **odtwarzanie w systemie rozproszonym** — gdzie pojawiają się wiadomości osierocone i utracone, efekt domino, ciągły restart oraz formalne pojęcia **spójnego punktu kontrolnego** $CP^\bullet$ i **linii odtwarzania** RL. Kończy się pierwszą techniką tworzenia punktów kontrolnych — **skoordynowaną (synchroniczną)**, na przykładzie **algorytmu Koo-Touega**.

---
## Awarie w systemie
<sub>Slajdy-FT-02_Recovery1.pdf, slajd 1</sub>

| Awaria              | Przyczyny                                         | Reakcje                                |
| ------------------- | ------------------------------------------------- | -------------------------------------- |
| **procesu**         | zakleszczenie, timeout, błąd ochrony, niespójność | abort, restart przetwarzania           |
| **węzła**           | błędy programowe lub sprzętowe, błędy zasilania   | stop i restart ze zdefiniowanego stanu |
| **pamięci masowej** | —                                                 | rekonstrukcja z archiwum               |
| **komunikacyjne**   | awaria medium lub urządzeń sieciowych             | naprawa, retransmisja                  |

## Typy awarii systemowych w modelu *fail-recovery*
<sub>slajd 2</sub>

- **przerwa** — system restartuje w **tym samym stanie**, który poprzedzał awarię,
- **amnezja** — system restartuje w **predefiniowanym stanie**, niezależnym od stanu w momencie wystąpienia awarii,
- **częściowa amnezja** — system restartuje w stanie, którego **część pokrywa się** ze stanem w momencie wystąpienia awarii, a pozostała część jest predefiniowana (np. awarie, po których następuje restart serwera plików).

## Odtwarzanie postępowe i wsteczne
<sub>slajdy 3–4</sub>

> **Odtwarzanie postępowe** (*forward recovery*) — jeśli natura błędu powodującego awarię pozwala na usunięcie błędów ze stanu systemu i system (proces) jest wyposażony w mechanizmy obsługujące dany błąd, to błąd ten można efektywnie wyeliminować (naprawić) i umożliwić postęp przetwarzania.

Przykład ze slajdu: całkowita utrata precyzji w wyniku operacji zmiennopozycyjnej może zostać usunięta przez podanie wyniku „zero" w obsłudze błędu (*arithmetic underflow*).

> **Odtwarzanie wsteczne** (*backward recovery*) — jeśli błąd jest nieprzewidywalny lub awaria nieodwracalna (np. *arithmetic overflow*), można jedynie **wymienić cały stan systemu** na wcześniej zarejestrowany, wolny od błędów („miejmy nadzieję" — komentarz ze slajdu).

**Ograniczenia odtwarzania postępowego** (slajd 4):
- zależy od skuteczności przewidywania i obsługiwania błędów, wymaga poprawnego przewidzenia konsekwencji błędów — na slajdzie cytat Nielsa Bohra: *„Trudno cokolwiek przewidzieć, a już szczególnie przyszłość"*,
- projektowane jest **pod konkretny system dla konkretnych błędów**,
- nie nadaje się do stosowania wobec błędów nieprzewidywalnych,
- **niemożliwe do zaimplementowania jako mechanizm systemowy**.

## Odtwarzanie wsteczne — własności, wymagania i cena
<sub>slajdy 5–7</sub>

Własności: **uniwersalne** (nie zależy od rodzaju błędu), stosowalne dla każdego systemu, implementowalne jako ogólny mechanizm.

**Wymagania:**
- wcześniejszy stan musi być poprawnie przywrócony, **oraz**
- przywrócony stan musi **poprzedzać wystąpienie uszkodzenia** powodującego błąd.

**Cena:**
- **narzut** — odtwarzanie wsteczne może wprowadzać duży narzut,
- **nawrót** — na ogół nie ma gwarancji, że awaria nie wystąpi po odtworzeniu stanu,
- **niepowtarzalność** (niemożliwość wycofania) — nie wszystkie komponenty systemu są odtwarzalne, a szczególnie **interakcje zewnętrzne** (np. wydane przez bankomat pieniądze).

> [!quote] Komentarz ze slajdu 6
> *No one cares if you can back up, only if you can recover!*

---
## Model węzła systemu
<sub>slajd 8</sub>

![[swn-ft02-s08-model-wezla.png]]
<sub>Model węzła systemu — Slajdy-FT-02_Recovery1.pdf, slajd 8</sub>

Węzeł zawiera **pamięć operacyjną** (*main memory*), w której rezydują **proces** $P_i$ i jego **kontroler** $C_i$, procesor (CPU) oraz dwa rodzaje pamięci zewnętrznej: **pamięć pomocniczą** (*secondary storage*) i **pamięć trwałą** (*stable storage*). Slajd odnotowuje kierunek rozwoju: $\rightarrow$ NVM: ReRAM, 3DXpoint, …

> [!important] Podział ról
> Rozróżnienie **proces $P_i$ / kontroler $C_i$** jest w tym wykładzie kluczowe: wszystkie algorytmy checkpointingu i odtwarzania formułowane są jako działania **kontrolerów**, nie procesów aplikacyjnych.

## Wsteczne odtwarzanie węzła
<sub>slajd 9</sub>

1. **Wycofywanie operacji** (*operation-based recovery*) — przechowywane w rejestrach (*log; audit trail*) niezbędne informacje o wykonywanych operacjach; strategie rejestrowania modyfikacji danych: **updating-in-place**, **write-ahead-log**.
2. **Przywracanie stanu** (*state-based recovery*) — przechowywany pełen stan procesu (pamięci, obiektu): **shadow pages**, **checkpointing**.
3. **mix 1+2**.

### Wycofywanie operacji — updating-in-place i write-ahead-log
<sub>slajd 10</sub>

> **Updating-in-place** — każdy zapis (*update*) rejestruje jednocześnie krotkę $log = \langle OBJ, UNDO, REDO \rangle$, gdzie **OBJ** = identyfikator modyfikowanego obiektu, **UNDO** = stan obiektu sprzed modyfikacji, **REDO** = nowy stan obiektu.

Odtwarzalna operacja jest implementowana w postaci kolekcji operacji:
- operacja **do** — wykonuje działanie (*update*) i rejestruje je (*log*),
- operacja **undo** — wycofuje działanie `do` zgodnie z polem UNDO,
- operacja **redo** — odtwarza działanie `do` zgodnie z polem REDO.

**Problem braku atomowości:** co jeśli awaria nastąpi pomiędzy dokonaniem zmian (*update*) a zapisaniem rejestru (*log*)?

> **Write-ahead-log** — `update` jest wykonywane zawsze **po** zapisaniu informacji UNDO, a **przed** zatwierdzeniem zmian zapisuje się informacje REDO.

### Przywracanie stanu — shadow pages i twin-page
<sub>slajd 11</sub>

![[swn-ft02-s11-shadow-pages-twin-page.png]]
<sub>Shadow pages i twin-page — Slajdy-FT-02_Recovery1.pdf, slajd 11</sub>

- **Shadow pages** — żądanie zapisu trafia do **kopii roboczej** strony X, a oryginał zachowywany jest jako **shadow page** X' (kopia zapasowa).
- **twin-page** (wariant 1.a) — utrzymywane są dwie kopie $X^1$, $X^2$; pierwszy zapis trafia do jednej, drugi do drugiej.
- **Checkpointing** — druga technika przywracania stanu, rozwijana w dalszej części wykładu.

> [!note] Uzupełnienie spoza slajdów
> [[Systemy Wysokiej Niezawodności/Shadow Pages]] dopowiada, że technika bywa realizowana **na poziomie sprzętowym** przy dostępie do pamięci (niektóre serwery mają ją wbudowaną): po pomyślnym zapisie kopii roboczej shadow page jest usuwany, a po niepowodzeniu — służy do przywrócenia stanu. **Wadą** jest zajmowanie dodatkowego miejsca podczas modyfikacji. W wariancie twin-page operacje na obu kopiach wykonywane są **naprzemiennie**, więc bliźniacza kopia zawsze przechowuje poprzedni stan.

---
## Punkt kontrolny
<sub>slajd 12</sub>

> **Punkt kontrolny** (*checkpoint*) = **stan** należący do wykonania $\alpha$, zachowany w **pamięci trwałej** w celu umożliwienia wznowienia przetwarzania od tego stanu w czasie odtwarzania wstecznego.

- **Lokalny punkt kontrolny** — punkt kontrolny $cp_i$ procesu $P_i$ (utworzony przez jego kontroler $C_i$).
- **Globalny punkt kontrolny** — wektor lokalnych punktów kontrolnych $CP = \langle cp_i : \text{dla każdego } P_i \rangle$.

![[swn-ft02-s12-punkt-kontrolny-lokalny-globalny.png]]
<sub>Lokalne punkty kontrolne trzech procesów $P_1/C_1$, $P_2/C_2$, $P_3/C_3$ — slajd 12</sub>

---
## Odtwarzanie w systemie rozproszonym

### Para punktów kontrolnych i wiadomość osierocona
<sub>slajd 13</sub>

> **Para punktów kontrolnych** = fragment globalnego punktu kontrolnego złożony z **2 lokalnych** punktów kontrolnych.

![[swn-ft02-s13-wiadomosc-osierocona.png]]
<sub>Para $(cp_i, cp_j)$ zawierająca wiadomość osieroconą $m$ — slajd 13</sub>

Para $(cp_i, cp_j)$ zawiera wiadomość **osieroconą** $m$, gdy
$$e_j^{\,l} = recv(P_j, P_i, m) \in cp_j \;\wedge\; e_i^{\,k} = send(P_i, \{P_j\}, m) \notin cp_i$$
czyli **odbiór jest w punkcie kontrolnym, a wysłanie nie**. Slajd stawia pytanie: czy taki niespójny obraz stanu przetwarzania jest akceptowalny dla odtwarzania?

### Semantyka wiadomości osieroconych
<sub>slajd 14</sub>

- w ogólnym przypadku **niespójny obraz stanu nie jest akceptowalny**,
- może prowadzić do naruszenia **własności bezpieczeństwa** przetwarzania rozproszonego — np. jeśli osierocona wiadomość $m$ przenosiła unikalny przywilej (żeton), po wznowieniu odtworzonego przetwarzania **oba procesy** posiadać będą przywilej,
- decydująca jest zatem możliwość określenia, czy osierocona wiadomość ma znaczenie dla poprawności przetwarzania, czy nie — czyli **znajomość semantyki wiadomości**.

### Determinizm i duplikacja
<sub>slajd 15</sub>

- wznowienie przetwarzania od $(cp_i, cp_j)$ wywoła ponownie zdarzenia $e_i^{\,k-1}$ oraz $e_j^{\,l+1}$, **o ile przetwarzanie jest deterministyczne**,
- jeśli zarówno $e_i^{\,k-1}$, jak i $e_i^{\,k}$ zachodzą deterministycznie, wówczas wiadomość $m$ — **już odebrana** przez $P_j$ — zostanie nadana ponownie $\rightarrow$ **duplikacja**,
- wymagany jest zatem dodatkowy mechanizm tolerujący duplikację wiadomości,
- gdy $e_i^{\,k-1}$ lub $e_i^{\,k}$ **nie** zachodzi deterministycznie, wynik lokalnego przetwarzania $P_i$ jest nieprzewidywalny.

### Wiadomości utracone
<sub>slajd 16</sub>

![[swn-ft02-s16-wiadomosc-utracona.png]]
<sub>Para $(cp_i^2, cp_j^1)$ zawierająca wiadomość utraconą $m$ — slajd 16</sub>

Para $(cp_i^2, cp_j^1)$ zawiera wiadomość **utraconą** $m$, gdy
$$e_i^{\,k} = send(P_i, \{P_j\}, m) \in cp_i^2 \;\wedge\; e_j^{\,l} = recv(P_j, P_i, m) \notin cp_j^1$$

- w ogólnym przypadku **utrata wiadomości nie prowadzi do niespójności obrazu stanu** — para ta odpowiada **spójnemu** obrazowi stanu przetwarzania,
- jednak dodatkowy mechanizm jest niezbędny dla odtworzenia/odzyskania (*replay*) utraconych wiadomości **znaczących** (np. żetonu) $\rightarrow$ *sliding window protocol*.

> [!important] Asymetria
> Wiadomość **osierocona** psuje spójność, wiadomość **utracona** — nie. To rozróżnienie wraca w definicji $CP^\bullet$ i $CP^!$ (slajd 22).

### Metoda kolejnych wycofań
<sub>slajd 17</sub>

Zamiast wybierać do wznowienia globalny punkt kontrolny zawierający kontrowersyjną parę $(cp_i^2, cp_j^1)$, można **wycofać się do pewnego innego** globalnego punktu kontrolnego zawierającego parę $(cp_i^{x<2}, cp_j^1)$, np. $(cp_i^1, cp_j^1)$. Podobnie postępuje się w przypadku wiadomości osieroconych oraz w dowolnym innym przypadku, gdy wybrany punkt kontrolny nie odpowiada obrazowi spójnemu (lub poprawnemu wg dowolnych innych kryteriów) — szuka się **wcześniejszego** punktu kontrolnego spełniającego wymogi poprawnego odtworzenia.

### Efekt domino
<sub>slajd 18</sub>

![[swn-ft02-s18-efekt-domino.png]]
<sub>Efekt domino — slajd 18</sub>

Niech $P_2$ ulegnie awarii po wysłaniu wiadomości $m$. Po wycofaniu $P_2$ do $cp_2^2$ wiadomość $m$ zostaje osierocona. Slajd pyta: **dokąd wycofujemy stan przetwarzania metodą kolejnych wycofań?** — a następnie: niech $P_3$ zostanie wycofany do $cp_3^2$. Kolejne wycofania wymuszają wycofania u sąsiadów, **kaskadowo**.

> [!note] Uzupełnienie spoza slajdów
> [[Systemy Wysokiej Niezawodności/Wycofanie operacji]] podaje zwięzłą definicję skutku: efekt domino to sytuacja, w której w procesie wybierania punktów kontrolnych **trafiamy na sam początek działania systemu**. Slajd 18 samej definicji nie formułuje — pokazuje mechanizm na przykładzie.

### Niespójność stanu kanałów
<sub>slajd 19</sub>

![[swn-ft02-s19-niespojnosc-stanu-kanalow.png]]
<sub>Niespójność stanu kanałów — slajd 19</sub>

- jeśli oba zdarzenia $e_i^{\,k-1}$ i $e_i^{\,k}$ zachodzą deterministycznie, wiadomość $m_1$ zostanie nadana ponownie $\rightarrow$ **duplikacja**,
- gdy $e_i^{\,k-1}$ lub $e_i^{\,k}$ nie zachodzi deterministycznie, po wznowieniu przetwarzania może zostać wysłana w jej miejsce **całkowicie inna** wiadomość $m_2$; wówczas $m_1$ **ciągle znajdująca się w kanale** stanie się osierocona w momencie odebrania, naruszając poprawność wznowionego przetwarzania.

### Ciągły restart
<sub>slajd 20</sub>

![[swn-ft02-s20-ciagly-restart.png]]
<sub>Ciągły restart (*recovery livelock*) — stan przed restartem i po restarcie, slajd 20</sub>

Prowadzić to może do problemu tzw. **ciągłego restartu** (*recovery livelock*): po każdym restarcie powstaje nowa wiadomość osierocona, wymuszająca kolejny restart.

---
## Spójność punktów kontrolnych
<sub>slajd 21</sub>

**Spójność pary punktów kontrolnych** zależy od wielu czynników:
- determinizmu zdarzeń / przetwarzania,
- semantyki wiadomości,
- dostępności dodatkowych mechanizmów (obsługujących duplikację lub utratę wiadomości, niespójności stanu kanałów itp.).

**Spójność globalnego punktu kontrolnego:**
- **CPP** = zbiór wszystkich par punktów kontrolnych,
- **CCPP** = zbiór wszystkich **spójnych** par punktów kontrolnych w CPP, $CCPP \subseteq CPP$,

$$CP = \langle cp_i : 1 \leqslant i \leqslant n \rangle \text{ jest spójnym punktem kontrolnym} \iff \bigwedge_{1 \leqslant i,j \leqslant n} (cp_i, cp_j) \in CCPP$$

### Spójny i silnie spójny punkt kontrolny
<sub>slajd 22</sub>

Dalej przyjmuje się definicje odpowiadające **spójnemu obrazowi stanu** przetwarzania rozproszonego:

> **Spójny punkt kontrolny** $CP^\bullet$: $(e \in CP \wedge e' \rightarrow e) \Rightarrow e' \in CP$
> (w szczególności: $recv(m) \in CP^\bullet \Rightarrow send(m) \in CP$)

> **Silnie spójny punkt kontrolny** $CP^{!}$: $CP^\bullet$ **+** $send(m) \in CP \Rightarrow recv(m) \in CP$

![[swn-ft02-s22-cp-spojny-i-silnie-spojny.png]]
<sub>Przykład: $CP^\bullet = \langle cp_1^2, cp_2^2, cp_3^2 \rangle$, $CP^{!} = \langle cp_1^1, cp_2^1, cp_3^1 \rangle$ — slajd 22</sub>

Różnica: $CP^\bullet$ dopuszcza wiadomości **utracone** (w kanale), $CP^{!}$ — nie; silnie spójny punkt kontrolny wymaga, by **kanały były puste**.

### Proste ustanawianie $CP^\bullet$ dla operacji atomowych
<sub>slajd 23</sub>

- **Ogólna idea:** każdy proces $P_i$ wyznacza punkt kontrolny $cp_i$ **bezpośrednio po wysłaniu** kolejnej wiadomości.
- **Założenie:** operacje `send` i `checkpoint` są nierozdzielne (atomowe).
- **Redukcja kosztu:** $cp_i$ wyznaczany jest po wysłaniu $K > 1$ wiadomości — slajd stawia pytanie *„jaki problem tu występuje?"* (przy $K>1$ pomiędzy punktami kontrolnymi mieszczą się wysłania, których odbiorca może już nie mieć w swoim $cp$ — wraca ryzyko osierocenia).

---
## Linia odtwarzania
<sub>slajd 24</sub>

- **GC** = zbiór wszystkich globalnych punktów kontrolnych,
- $GC^\bullet$ = zbiór wszystkich **spójnych** globalnych punktów kontrolnych, $GC^\bullet \subseteq GC$.

> **Linia odtwarzania** (*recovery line*) = spójny globalny punkt kontrolny: $RL = \langle cp_1, cp_2, \ldots, cp_n \rangle \in GC^\bullet$

> **Punkt odtwarzania** (*recovery point*) = $cp_i$ należący do jakiejś linii odtwarzania.

> **Najświeższa linia odtwarzania** (*current recovery line*): $RL^{*} :: \bigwedge_{RL \in GC^\bullet} RL \subseteq RL^{*}$

### Zawężenie linii odtwarzania
<sub>slajd 25</sub>

![[swn-ft02-s25-zawezenie-linii-odtwarzania.png]]
<sub>Zawężenie linii odtwarzania — slajd 25</sub>

Po awarii $P_3$ linia odtwarzania $RL = \{cp_1^3, cp_3^3\}$ — obejmuje tylko **część** procesów.

### Wirtualne punkty odtwarzania
<sub>slajd 26</sub>

![[swn-ft02-s26-wirtualne-punkty-odtwarzania.png]]
<sub>Wirtualne punkty odtwarzania — slajd 26</sub>

$P_2$ i $P_4$ uznają aktualny stan za **virtual checkpoint** ($vcp_2^5$ oraz $vcp_4^3$); wirtualna linia odtwarzania: $RL = \langle cp_1^3, vcp_2^5, cp_3^3, vcp_4^3 \rangle$.

---
## Output commit
<sub>slajd 27</sub>

**Problem interakcji ze światem zewnętrznym:** protokół odtwarzania **nie wycofa** operacji wykonanych na elementach spoza systemu.

**Algorytm *output commit*:**
- ma na celu zagwarantować, że stan, w którym wykonano takie operacje, **nigdy nie będzie musiał być wycofany**,
- może powodować dodatkowy narzut (blokowanie przetwarzania) przed każdą interakcją zewnętrzną,
- ponadto dane pobrane z zewnątrz muszą być składowane w pamięci trwałej, aby służyć do odtworzenia (*replay*),
- musi wchodzić w skład **każdego** protokołu odtwarzania stanu systemu z interakcjami zewnętrznymi — w dalszych rozważaniach przyjmuje się milcząco jego istnienie.

---
## Skoordynowane (synchroniczne) tworzenie punktów kontrolnych

### Algorytm Koo-Touega — idea i rodzaje punktów kontrolnych
<sub>slajd 28</sub>

**Sposób ustanawiania CP:** tak, aby zawsze był to $CP^\bullet$ (w efekcie $RL^{*}$).

> **Algorytm Koo-Touega [2]** wykorzystuje ideę **przetwarzania dyfuzyjnego** i **2PC**.

Dwa rodzaje punktów kontrolnych:
- **ostateczny** (*permanent checkpoint*) — należący do $CP^\bullet$; **tylko taki może być punktem odtwarzania**,
- **wstępny** (*tentative checkpoint*) — zanim stanie się *permanent*,
- punkty kontrolne obu rodzajów zapisywane są w pamięci trwałej,
- jednocześnie istnieje **co najwyżej jeden punkt ostateczny** dla każdego procesu.

### Model systemu
<sub>slajd 29</sub>

- przetwarzanie może być **niedeterministyczne**,
- model awarii *fail-recovery*: procesy podlegają tymczasowym awariom typu **zatrzymanie**; co więcej — pozostałe procesy w **skończonym czasie** dowiadują się o awariach,
- **niezawodne kanały FIFO**, w szczególności: sieć niepodzielna na rozłączne części, za retransmisję wiadomości odpowiada podsystem komunikacyjny,
- ewentualne interakcje ze światem zewnętrznym obsługuje dodatkowy protokół *output commit*.

**Dodatkowe założenia prezentacji:** tylko jeden proces rozpoczyna wykonanie algorytmu (**inicjator**); procesy nie ulegają awariom podczas wykonywania tego algorytmu.

### Algorytm tworzenia punktów kontrolnych
<sub>slajd 30</sub>

**Faza 1:** inicjator $C_i$ (kontroler procesu $P_i$) ustala **wstępny** $cp_i$ i rozsyła żądanie wyznaczenia $cp_j$ pozostałym $C_j$.

**Faza 2:** jeśli $C_i$ otrzymał **wszystkie potwierdzenia pozytywne**, rozsyła decyzję zamiany wstępnych punktów kontrolnych na ostateczne. $C_j$ **nie wysyła żadnych wiadomości** $P_j$, dopóki nie otrzyma decyzji.

**Wykorzystywane struktury:**
- $m.l$ (*m label*) — etykieta wiadomości $m$ (monotoniczny licznik wysyłanych wiadomości); minimalna etykieta $= \bot$, maksymalna $= \top$,
- $\text{last\_label\_rcvd}_i[j]$ — etykieta **ostatniej** wiadomości $m$ wysłanej przez $P_j$ i odebranej przez $P_i$ **od ostatniego punktu kontrolnego**:
$$\text{last\_label\_rcvd}_i[j] = \begin{cases} m.l & \text{jeśli } m \text{ istnieje} \\ \bot \end{cases}$$
- $\text{first\_label\_sent}_i[j]$ — etykieta **pierwszej** wiadomości $m$ wysłanej przez $P_i$ do $P_j$ **od ostatniego punktu kontrolnego**:
$$\text{first\_label\_sent}_i[j] = \begin{cases} m.l & \text{jeśli } m \text{ istnieje} \\ \bot \end{cases}$$

### Niepotrzebne punkty kontrolne — optymalizacja
<sub>slajd 31</sub>

![[swn-ft02-s31-niepotrzebne-punkty-kontrolne.png]]
<sub>Niepotrzebne punkty kontrolne: czy $cp_3^2$ jest potrzebny? — slajd 31</sub>

Jeśli $C_i$ wysyła do $C_j$ żądanie $\text{Take\_a\_tentative\_ckpt}(C_i, C_j, \text{last\_label\_rcvd}_i[j])$, to $C_j$ wyznacza wstępny punkt kontrolny procesu $P_j$ **tylko gdy**:
$$\text{last\_label\_rcvd}_i[j] \geqslant \text{first\_label\_sent}_j[i] > \bot$$

Żądanie jest wysyłane **tylko do** $C_j \in \text{ckpt\_cohort}_i = \{ C_j : \text{last\_label\_rcvd}_i[j] > \bot \}$.

### Algorytm odtwarzania
<sub>slajd 32</sub>

**Założenia:** tylko jeden inicjator; procedury *checkpointing* i *recovery* nie wykonują się jednocześnie; $C_j$ nie zgadza się na wycofanie $P_j$, jeśli już uczestniczy w którejś z tych procedur.

**Faza 1:** inicjator $C_i$ wysyła pytanie o **zgodę na wycofanie** do poprzedniego $cp_j$.

**Faza 2:** jeśli $C_i$ otrzymał wszystkie potwierdzenia pozytywne, rozsyła **decyzję wycofania** do wszystkich procesów. $C_j$ nie wysyła żadnych wiadomości, dopóki nie otrzyma decyzji.

### Niepotrzebne wycofanie — optymalizacja
<sub>slajdy 33–34</sub>

![[swn-ft02-s33-niepotrzebne-wycofanie.png]]
<sub>Niepotrzebne wycofanie: czy $cp_3^2$ musi być wycofany? — slajd 33</sub>

- $\text{last\_label\_sent}_i[j]$ — etykieta **ostatniej** wiadomości $m$ wysłanej przez $P_i$ do $P_j$ **przed** ostatnim punktem kontrolnym:
$$\text{last\_label\_sent}_i[j] = \begin{cases} m.l & \text{jeśli } m \text{ istnieje} \\ \top \end{cases}$$

Jeśli $C_i$ wysyła do $C_j$ żądanie $\text{Prepare\_to\_rollback}(C_i, C_j, \text{last\_label\_sent}_i[j])$, to $C_j$ wycofa $P_j$ do ostatniego punktu kontrolnego **tylko gdy**:
$$\text{last\_label\_rcvd}_j[i] > \text{last\_label\_sent}_i[j]$$

Spełnienie tego warunku wskazuje, że $P_i$ wycofuje się do stanu, w którym zostaną **anulowane zdarzenia wysłania** co najmniej jednej wiadomości **odebranej już** przez $P_j$.

$\text{roll\_cohort}_i = \{ C_j : P_i \text{ może wysyłać wiadomości do } P_j \}$.

> [!note] Uzupełnienie spoza slajdów
> [[Systemy Wysokiej Niezawodności/Algorytm Koo-Touega]] dodaje zachowanie odbiorcy żądania, którego na slajdach nie ma wprost: $C_j$ **od razu** zwraca potwierdzenie pozytywne, gdy nie musi tworzyć wstępnego punktu kontrolnego albo utworzył go wcześniej; jeśli okazuje się, że powinien go utworzyć, wykonuje proces **od fazy 1**, nie wysyłając jednak żądania do $C_i$, od którego żądanie dostał. To właśnie jest owo „przetwarzanie dyfuzyjne" ze slajdu 28.

### Wady skoordynowanego wyznaczania punktów kontrolnych
<sub>slajd 35</sub>

1. Wymagane dodatkowe wiadomości kontrolne.
2. Synchronizacja może powodować duży narzut (opóźnienie).
3. Przy małej ilości awarii prowadzi do zbędnego ponoszenia dużego kosztu.
4. Koszt ponosimy **nawet gdy w danym wykonaniu przetwarzania awarie w ogóle nie występują**.

> Te cztery wady są bezpośrednią motywacją dla technik z kolejnego wykładu — [[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów]].

---
## Literatura
<sub>slajd 36</sub>

1. E.N. Elnozahy, D.B. Johnson, Y.M. Wang, *A Survey of Rollback-Recovery Protocols in Message-Passing Systems*, Carnegie Mellon University technical report CMU-CS-96-181, 1996.
2. R. Koo, S. Toueg, *Checkpointing and Rollback-Recovery for Distributed Systems*, IEEE Transactions on Software Engineering, vol. 13 no. 1, pp. 23–31, 1987.
3. R. Chow, T. Johnson, *Distributed Operating Systems & Algorithms*, Addison Wesley Longman, 1997, ch. 13.
4. M. Raynal, *Distributed algorithms for message-passing systems*, Springer, 2013, ch. 8.

---
## Braki i uwagi

> [!todo] Czego nie ma na slajdach tego wykładu
> - **Definicji efektu domino** — slajd 18 pokazuje wyłącznie mechanizm na przykładzie.
> - **Analizy złożoności** algorytmu Koo-Touega (liczba wiadomości, liczba rund) — nie pojawia się w żadnym z 36 slajdów.
> - **Odpowiedzi** na pytanie ze slajdu 23 („jaki problem tu występuje?" przy $K > 1$) oraz ze slajdu 18 („dokąd wycofujemy stan?").
> - **Pojęcia *przetwarzanie dyfuzyjne*** — użyte na slajdzie 28 bez definicji; opis mechanizmu w [[Systemy Wysokiej Niezawodności/Algorytm Koo-Touega]], a formalnie (algorytm Dijkstry-Scholtena) w [[RSO 10 Detekcja zakończenia]].
