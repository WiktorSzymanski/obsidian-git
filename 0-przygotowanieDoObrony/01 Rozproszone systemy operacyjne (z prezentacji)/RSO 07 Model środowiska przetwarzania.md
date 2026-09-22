---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
source: "rso_sum_01.pdf"
slajdy: "1–101"
---
# RSO 07. Model środowiska przetwarzania rozproszonego
---
> Wykład 1 buduje **aparat pojęciowy** używany we wszystkich pozostałych wykładach: definicję środowiska przetwarzania (węzły, łącza), model procesu i kanału, zdarzenia i funkcję tranzycji, procesy aktywne/pasywne wraz z warunkiem uaktywnienia, modele żądań oraz relację poprzedzania i diagramy przestrzenno-czasowe. Pierwsza część to klasyczne wprowadzenie do systemów rozproszonych (cechy, cele, przezroczystość, skalowalność, GRID, chmura).

---
## Część I — wprowadzenie

### Definicja i aspekty systemu rozproszonego
<sub>rso_sum_01.pdf, slajdy 3–6</sub>

> **System rozproszony** — zestaw niezależnych komputerów, sprawiający na jego użytkownikach wrażenie jednego, logicznie zwartego systemu.

Aspekty:
- **sprzęt**: maszyny są autonomiczne,
- **oprogramowanie**: wrażenie pojedynczego systemu.

**Cechy systemów rozproszonych**:
- ukrycie przed użytkownikami: różnic pomiędzy poszczególnymi komputerami, sposobów komunikowania się komputerów, wewnętrznej organizacji systemu rozproszonego,
- jednolity i spójny interfejs dla użytkownika — niezależnie od czasu i miejsca interakcji,
- łatwość rozszerzania (skalowania).

**Architektura systemu rozproszonego**: rozbudowa własności lokalnych systemów operacyjnych, usługi dla aplikacji rozproszonych, oprogramowanie warstwy pośredniej (_middleware_).

Postęp technologiczny ostatniego półwiecza (slajd 3): od 10 mln dolarów za 1 instrukcję/s do 1000 dolarów za 100 mln instrukcji/s — $12^{12}$ razy lepszy współczynnik cena/efektywność; sieci komputerowe LAN i WAN.

### Cele systemów rozproszonych
<sub>rso_sum_01.pdf, slajdy 7–8</sub>

- łatwe połączenie użytkownik-zasoby,
- ukrywanie faktu rozproszenia zasobów,
- otwartość,
- skalowalność.

**Łączenie użytkowników i zasobów**: ekonomia (współdzielony dostęp do zasobów jest tańszy — drukarki, specjalizowane komputery, szybkie pamięci, bazy danych, zdalne dokumenty) oraz bezpieczeństwo.

### Przezroczystość
<sub>rso_sum_01.pdf, slajdy 9–12</sub>

> **System przezroczysty** (transparentny, ang. _transparent_) sprawia wrażenie systemu scentralizowanego (dla użytkowników i aplikacji).

| Rodzaj | Ang. | Znaczenie |
|---|---|---|
| **dostępu** | _access transparency_ | ujednolicanie metod dostępu do danych i ukrywanie różnic w reprezentacji danych |
| **położenia** | _location transparency_ | użytkownicy nie mogą określić fizycznego położenia zasobu (np. na podstawie jego nazwy czy identyfikatora) |
| **wędrówki** | _migration transparency_ | można przenosić zasoby pomiędzy serwerami bez zmiany sposobu odwoływania się do nich |
| **przemieszczania** | _relocation transparency_ | zasoby mogą być przenoszone **nawet podczas ich używania** |
| **zwielokrotniania** | _replication transparency_ | ukrywanie przed użytkownikami faktu zwielokrotniania (replikacji) zasobów |
| **współbieżności** | _concurrency transparency_ | możliwość współbieżnego przetwarzania danych nie powodująca powstawania niespójności w systemie |
| **awarii** | _failure transparent_ | maskowanie przejściowych awarii poszczególnych komponentów systemu rozproszonego |
| **trwałości** | _persistence transparency_ | maskowanie sposobu przechowywania zasobu (pamięć ulotna, dysk) |

Istnieje **kompromis pomiędzy dużym stopniem przezroczystości a efektywnością systemu**.

### Otwartość i elastyczność
<sub>rso_sum_01.pdf, slajdy 13–14</sub>

**Otwartość**:
- usługi zgodne ze standardowymi regułami opisującymi ich składnię i semantykę (przykład: protokoły komunikacyjne w sieciach komputerowych),
- specyfikacja usług poprzez interfejsy — język opisu interfejsu (_Interface Definition Language_),
- specyfikacje muszą być **zupełne** (kompletne) i **neutralne** → zdolność do współdziałania (_interoperability_) i przenośność (_portability_).

**Systemy elastyczne**: łatwość konfiguracji, łatwość rekonfiguracji (np. wymiana poszczególnych komponentów).
**Organizacja systemu rozproszonego**: projekt monolityczny, logiczne wydzielenie składowych, podział na autonomiczne komponenty (koszt: wydajność), oddzielenie polityki od mechanizmu.

### Skalowalność
<sub>rso_sum_01.pdf, slajd 15</sub>

**Wymiary skalowalności**: pod względem rozmiaru (decentralizacja: danych, usług i algorytmów), geograficzna, pod względem administracyjnym.

**Algorytmy zdecentralizowane**:
- brak informacji globalnej,
- decyzje na podstawie informacji lokalnych,
- odporność na awarie pojedynczych maszyn,
- brak założeń dot. istnienia globalnego zegara.

### Problemy konstrukcji systemów rozproszonych
<sub>rso_sum_01.pdf, slajd 16</sub>

optymalne zrównoleglenie algorytmów przetwarzania · ocena poprawności i efektywności algorytmów rozproszonych · alokacja zasobów rozproszonych · synchronizacja procesów · ocena globalnego stanu przetwarzania · realizacja zaawansowanych modeli przetwarzania · niezawodność · bezpieczeństwo.

### Motywy i klasy zastosowań
<sub>rso_sum_01.pdf, slajdy 17–18</sub>

**Motywy**: różnorodność otwartych problemów związanych z konstrukcją i zarządzaniem systemami rozproszonymi; ogromne rzeczywiste zapotrzebowanie na systemy rozproszone; dostępność środków technicznych i praktyczne możliwości realizacji.

**Klasy zastosowań**: aplikacje wykorzystujące przetwarzanie rozproszone na wielu jednostkach obliczeniowych (_distributed supercomputing_) · wymagające dużej przepustowości (_high throughput_) · „na żądanie” (_on demand_) · intensywnie przetwarzające dane (_data intensive_) · umożliwiające współpracę (_collaborative_).

### GRID
<sub>rso_sum_01.pdf, slajdy 19–21</sub>

> „A computational grid is a hardware and software infrastructure that provides dependable, consistent, pervasive, and inexpensive access to high-end computational capabilities” / Ian Foster /

Cztery cechy z definicji:

| Cecha | Znaczenie |
|---|---|
| **usługa wiarygodna** (_dependable_) | użytkownicy żądają pewności, że otrzymają przewidywalny, nieprzerwany poziom wydajności dzięki różnym elementom tworzącym GRID |
| **usługa spójna** (_consistent_) | potrzebny jest standardowy serwis, dostępny poprzez standardowe interfejsy, pracujący ze standardowymi parametrami |
| **usługa powszechnie dostępna / wszechobecna** (_pervasive_) | usługa zawsze powinna być dostępna, niezależnie od tego gdzie znajduje się użytkownik tej usługi |
| **usługa relatywnie tania / opłacalna** (_inexpensive_) | dostęp do usługi powinien być relatywnie tani, tak by korzystanie z niej było atrakcyjne także z ekonomicznego punktu widzenia |

**Co GRID powinien** (slajd 21): umożliwiać rozproszenie geograficzne zasobów · obsługiwać heterogeniczność sprzętową i programową · być połączony poprzez heterogeniczną sieć · korzystać z ogólnie dostępnych, standardowych protokołów i interfejsów · być odporny na zawodny sprzęt · pozwalać na zmianę dynamiki dostępu do sprzętu · zrzeszać różne organizacje (wirtualne) z ich własnymi politykami bezpieczeństwa i dostępu do zasobów.

### Cloud computing
<sub>rso_sum_01.pdf, slajdy 22–24</sub>

> **Chmura obliczeniowa** (ang. _cloud_) to zbiór zasobów wraz z oprogramowaniem, zarządzanych i udostępnianych z użyciem usług przez dostawcę.

Typowa charakterystyka: **skalowalne**, dostępne **na żądanie** · napędzane **ekonomią**, nie standardami; **pay-as-you-go** · **łatwe** w użyciu · dostępne dla wielu użytkowników z pomocą **wirtualizacji**.

**Możliwości**: „płacisz za tyle ile używasz” · eliminacja kosztów inwestycyjnych — infrastruktury · **automatyzacja** zarządzania przez dostawcę, automatyzacja użycia przez użytkownika (skalowalność na żądanie itp.) · **iluzja dostępu do nieskończonej puli zasobów** dostępnych na żądanie, dowolne zastosowania.
**Ryzyko**: gdzie są moje dane? · nad czym mam kontrolę? · co jeśli jeden dostawca nie działa?

**Przykładowe projekty przetwarzania rozproszonego** (slajd 24): SETI, Cure Cancer, Fight Anthrax, Prime Numbers, Distributed.net, GIMPS, FreeDB.org, The Internet Movie Database, The Distributed Chess Project, Wikipedia, Dmoz – Open Directory Project, ClimatePrediction.net, Lifemapper.

---
## Część II — formalny model środowiska przetwarzania

### Rozproszony system informatyczny
<sub>rso_sum_01.pdf, slajdy 25–26</sub>

Rozproszony system informatyczny obejmuje:
- **środowisko przetwarzania rozproszonego** — węzły, łącza,
- **zbiór procesów rozproszonych** — zbiór procesów sekwencyjnych realizujących wspólne cele przetwarzania.

> **Środowisko przetwarzania rozproszonego** jest zbiorem $\mathcal{N}$ autonomicznych jednostek przetwarzających $N_i$ (węzłów), zintegrowanych siecią komunikacyjną (środowiskiem komunikacyjnym, łączami komunikacyjnymi, łączami transmisyjnymi).

Komunikacja między węzłami możliwa jest **tylko przez transmisję pakietów informacji** (wiadomości, komunikatów) łączami komunikacyjnymi <sub>(slajd 27)</sub>.

### Zegary
<sub>rso_sum_01.pdf, slajd 28</sub>

Jednostki przetwarzające realizują przetwarzanie z prędkością narzucaną przez **lokalne zegary**.
- zegary są **niezależne** $\rightarrow$ węzły działają **asynchronicznie**,
- zegary są **zsynchronizowane** lub istnieje **wspólny zegar globalny** dla wszystkich węzłów $\rightarrow$ węzły działają **synchronicznie**.

### Węzeł i łącze
<sub>rso_sum_01.pdf, slajdy 29–34</sub>

**Węzeł**: jednostka przetwarzająca $N_i \in \mathcal{N}$ obejmująca **procesor**, **lokalną pamięć operacyjną** i **interfejs komunikacyjny**.

**Łącze komunikacyjne** — element umożliwiający transmisję informacji między interfejsami odległych węzłów. Wyróżnia się łącza **jedno-** i **dwu-kierunkowe**.

- **Bufory łącza**: łącza są wyposażone w bufory o określonej pojemności (ang. _links capacity_). Łącze bez buforów (pojemność zero) to łącze **nie buforowane**, w przeciwnym razie — **buforowane**.
- **Kolejność odbierania**: **łącze FIFO** — kolejność odbierania komunikatów wysyłanych z danego węzła jest zgodna z kolejnością ich wysyłania; **łącze nonFIFO** — w przeciwnym przypadku.
- **Niezawodność**: łącza mogą gwarantować w sposób niewidoczny dla użytkownika, że żadna wiadomość nie jest tracona, duplikowana lub zmieniana — to tzw. **łącza niezawodne** (_reliable_, _lossless_, _duplicate free_, _error free_, _uncorrupted_, _no spurious_).
- **Czas transmisji** w łączu niezawodnym (_transmission delay_, _in-transit time_) może być ograniczony lub jedynie określony jako **skończony lecz nieprzewidywalny**.

### Struktura środowiska jako graf
<sub>rso_sum_01.pdf, slajdy 35–36</sub>

$$\mathcal{G} = \langle \mathcal{V}, \mathcal{A} \rangle$$

- wierzchołki $V_i \in \mathcal{V}$ reprezentują jednostki przetwarzające $N_i \in \mathcal{N}$,
- krawędzie $(V_i, V_j) \in \mathcal{A}$, $\mathcal{A} \subseteq \mathcal{V} \times \mathcal{V}$ grafu **niezorientowanego** lub łuki $\langle V_i, V_j \rangle \in \mathcal{A}$ grafu **zorientowanego** reprezentują odpowiednio łącza dwu- lub jedno-kierunkowe.

![[rso-w1-s36-przyklady-topologii.png]]
<sub>Przykłady topologii — rso_sum_01.pdf, slajd 36</sub>

### Przetwarzanie rozproszone i proces sekwencyjny
<sub>rso_sum_01.pdf, slajdy 37–39</sub>

> **Procesem rozproszonym** (przetwarzaniem rozproszonym) nazywamy **współbieżne** i **skoordynowane** (ang. _concurrent and coordinated_) wykonanie w środowisku rozproszonym zbioru $\mathcal{P}$ procesów sekwencyjnych $P_1, P_2, P_3, \ldots, P_n$ współdziałających w realizacji wspólnego celu przetwarzania.

Nieformalnie, każdy **proces sekwencyjny** jest działaniem wynikającym z wykonywania w pewnym środowisku (kontekście) programu sekwencyjnego (algorytmu sekwencyjnego), który składa się z ciągu operacji (instrukcji, wyrażeń) **atomowych** (nieprzerywalnych).

**Klasy operacji**:
- **wewnętrzne** (ang. _internal_) — odnoszą się tylko do zmiennych lokalnych programu,
- **komunikacyjne** (ang. _communication_) — odnoszą się do środowiska i dotyczą komunikatów (_messages_) oraz kanałów (_channels_).

### Kanał
<sub>rso_sum_01.pdf, slajdy 40–43</sub>

> **Kanał** jest obiektem (zmienną) skojarzonym z **uporządkowaną parą procesów** $\langle P_i, P_j \rangle$, modelującym jednokierunkowe łącze transmisyjne. Typem tego obiektu jest zbiór wiadomości, którego rozmiar nazywany jest **pojemnością kanału**.

Kanał skojarzony z parą $\langle P_i, P_j \rangle$ oznaczamy $C_{i,j}$ i nazywamy:
- **kanałem incydentnym** z procesem $P_i$ i z procesem $P_j$,
- **kanałem wyjściowym** procesu $P_i$,
- **kanałem wejściowym** procesu $P_j$.

Zbiory kanałów: $\mathcal{C}_i^{IN}$ (wejściowe), $\mathcal{C}_i^{OUT}$ (wyjściowe), $\mathcal{C}_i$ (wszystkie incydentne):
$$\mathcal{C}_i = \mathcal{C}_i^{IN} \cup \mathcal{C}_i^{OUT} \tag{2.1}$$

Zbiory procesów sąsiednich:
$$\mathcal{P}_i^{IN} = \{P_j : \langle P_j, P_i \rangle \in \mathcal{C}_i^{IN}\} \tag{2.2}$$
$$\mathcal{P}_i^{OUT} = \{P_j : \langle P_i, P_j \rangle \in \mathcal{C}_i^{OUT}\} \tag{2.3}$$

### Stan kanału i predykaty
<sub>rso_sum_01.pdf, slajdy 44–50</sub>

> Przez **stan $L_{i,j}$ kanału** $C_{i,j}$ rozumieć będziemy zbiór (lub uporządkowany zbiór) wiadomości wysłanych przez proces $P_i$, lecz jeszcze nie odebranych przez proces $P_j$.

W celu modelowania opóźnień komunikacyjnych w zbiorze $L_{i,j}$ wyróżnia się dwa **rozłączne** podzbiory:
- **zbiór wiadomości transmitowanych** $L_{i,j}^{T}$ (ang. _in-transit_),
- **zbiór wiadomości dostępnych** $L_{i,j}^{A}$ (ang. _available_, _arrived_, _ready_).

$$L_{i,j} = L_{i,j}^{T} \cup L_{i,j}^{A} \tag{2.4}$$

Predykaty opisujące stan kanału:

| Predykat | Definicja | Nr |
|---|---|---|
| $empty(C_{i,j})$ | $\equiv L_{i,j} = \varnothing$ | (2.5) |
| $\textit{in-transit}(C_{i,j})$ | $\equiv L_{i,j}^{T} \neq \varnothing$ | (2.6) |
| $available(C_{i,j})$ | $\equiv L_{i,j}^{A} \neq \varnothing$ | (2.7) |

### Operacje komunikacyjne
<sub>rso_sum_01.pdf, slajdy 51–54</sub>

**Indywidualne**:
- $send(P_i, P_j, M)$ — efektem jest umieszczenie wiadomości $M$ w kanale $C_{i,j}$:
  $$L_{i,j} := L_{i,j} \cup \{M\} \tag{2.8}$$
- $receive(P_i, P_j, inM)$ — jeżeli kanał $C_{i,j}$ nie jest pusty i pewna wiadomość $M$ jest bezpośrednio dostępna ($available(C_{i,j})$ ma wartość $True$), efektem jest pobranie wiadomości $M$ z kanału:
  $$L_{i,j} := L_{i,j} \setminus \{M\} \quad\text{oraz}\quad inM := M \tag{2.9}$$

**Grupowe**:
- $send(P_i, \mathcal{P}_i^{R}, M)$ — umieszczenie wiadomości we wszystkich kanałach $C_{i,j}$ takich, że $P_j \in \mathcal{P}_i^{R}$:
  $$L_{i,j} := L_{i,j} \cup \{M\} \tag{2.10}$$
- $receive(\mathcal{P}_j^{S}, P_j, sInM)$ — **atomowe** pobranie wiadomości $M_i$ od procesów $P_i \in \mathcal{P}_j^{S}$ i umieszczanie ich w $sInM$; dla każdego procesu $P_i \in \mathcal{P}_j^{S}$ wykonywane jest kolejno:
  $$L_{i,j} := L_{i,j} \setminus \{M_i\} \quad\text{oraz}\quad sInM := sInM \cup \{M_i\} \tag{2.11}$$

### Rodzaje komunikacji
<sub>rso_sum_01.pdf, slajdy 55–57</sub>

Kanały o niezerowej pojemności umożliwiają realizację operacji typów: **nieblokowanej** i **blokowanej**.
- **Komunikacja synchroniczna** — nadawca i odbiorca są blokowani aż odpowiedni odbiorca odczyta przesłaną do niego wiadomość (ang. _rendez-vous_).
- **Komunikacja asynchroniczna** — nadawca lub odbiorca komunikuje się w sposób nieblokowany.

### Stan procesu
<sub>rso_sum_01.pdf, slajdy 58–59</sub>

> **Stan $S_i(t)$ procesu** w chwili $t$ czasu lokalnego jest w ogólności zbiorem wartości wszystkich zmiennych lokalnych skojarzonych z procesem w chwili $t$ oraz ciągów wiadomości wysłanych (wpisanych) do incydentnych kanałów wyjściowych i ciągów wiadomości odebranych z incydentnych kanałów wejściowych do chwili $t$.

Dla każdego $t$, $S_i(t) \in \mathcal{S}_i$; dla uproszczenia notacji stan w pewnej chwili oznaczamy $S_i$.

Zbiór $\mathcal{S}_i^{0}$ to **zbiór stanów początkowych**, których wartości są zadawane wstępnie, bądź są wynikiem zajścia wyróżnionego **zdarzenia inicjującego** $E_i^{0}$.

### Zdarzenia
<sub>rso_sum_01.pdf, slajdy 60–65</sub>

> **Zdarzenie** $E_i^{k}$ odpowiada unikalnemu **wykonaniu operacji atomowej**, zmieniającemu stan $S_i$ procesu i ewentualnie stan incydentnych z procesem kanałów $C_{i,j}$ lub $C_{j,i}$. Jeżeli operacja odpowiadająca zdarzeniu została wykonana, to powiemy, że **zdarzenie zaszło**.

**Klasy zdarzeń**: $e\_send$, $e\_receive$, $e\_internal$.

- $e\_send(P_i, P_j, M)$ — zachodzi w procesie $P_i$ w wyniku wykonania przez ten proces operacji $send(P_i, P_j, M)$; analogicznie $e\_send(P_i, \mathcal{P}_i^{R}, M)$ dla operacji grupowej.
- $e\_receive(P_i, P_j, M)$ — zachodzi w procesie $P_j$, gdy $P_j$ wykonał operację $receive(P_i, P_j, inM)$, a odczytana do zmiennej lokalnej $inM$ wiadomość $M$ pochodziła od procesu $P_i$; analogicznie $e\_receive(\mathcal{P}_j^{S}, P_j, \mathcal{M}_j^{S})$ dla operacji grupowej.
- $e\_internal(P_i)$ — zachodzi gdy proces $P_i$ wykonał operację, która **nie zmienia stanu jego kanałów incydentnych**. Do zdarzeń lokalnych zalicza się m.in. $e\_init(P_i, S_i^{k})$ — nadające procesowi stan $S_i^{k}$ (w szczególności stan początkowy) oraz $e\_stop(P_i)$ — kończące wykonywanie procesu.

**Dostępność wiadomości** utożsamiać można z zajściem zdarzeń w środowisku komunikacyjnym: zdarzenia **dostarczenia** wiadomości $M$ — $e\_deliver(P_i, P_j, M)$ oraz zdarzenia **nadejścia** wiadomości $M$ — $e\_arrive(P_i, P_j, M)$. Przez $\mathcal{P}_j^{A}$ oznaczamy zbiór procesów, których wiadomości dotarły i są dostępne dla $P_j$.

### Funkcja tranzycji i predykaty zdarzeń
<sub>rso_sum_01.pdf, slajdy 66–69</sub>

> **Funkcja tranzycji** $\mathcal{F}_i \subseteq \mathcal{S}_i \times \mathcal{E}_i \times \mathcal{S}_i$ opisuje reguły zmiany stanu $S$ na $S'$ w wyniku zajścia zdarzenia $E$.

Elementy $\langle S, E, S_2 \rangle \in \mathcal{F}_i$ nazywamy **tranzycjami** lub **krokami**; w zależności od zachodzącego zdarzenia — **tranzycją wejścia, wyjścia** lub **lokalną**.

- **Zdarzenie dopuszczalne** (ang. _allowed_) w stanie $S$: takie, dla którego $\langle S, E, S_2 \rangle \in \mathcal{F}_i$. Predykat $allowed(E)$.
- **Zdarzenie gotowe / przygotowane** (ang. _ready_): może zajść ze względu na warunki zewnętrzne (stan kanałów). Predykat $ready(E)$.
- **Zdarzenie aktywne**: jednocześnie **gotowe i dopuszczalne**:
  $$enable(E) \equiv ready(E) \wedge allowed(E) \tag{2.14}$$

### Procesy aktywne i pasywne
<sub>rso_sum_01.pdf, slajdy 70–73</sub>

- Proces $P_i$ jest w **stanie końcowym** $S_i^{e}$, jeżeli zbiór zdarzeń dopuszczalnych w tym stanie jest **pusty**.
- Jeżeli niepusty zbiór zdarzeń dopuszczalnych zawiera **wyłącznie zdarzenia odbioru** i **żadne z nich nie jest aktywne (gotowe)**, to proces jest **wstrzymany** (**zablokowany**).
- Proces wstrzymany lub zakończony nazywamy **pasywnym**; **aktywny** to proces, który nie jest pasywny.

Stan procesu reprezentuje zmienna logiczna $passive_i$: $True$ gdy proces pasywny, $False$ gdy aktywny.

**Proces aktywny** ($passive_i = False$) może wysyłać i odbierać wiadomości, wykonywać tranzycje lokalne, a więc potencjalnie może również **spontanicznie (w dowolnej chwili) zmienić swój stan na pasywny**.

W stanie **pasywnym** ($passive_i = True$) dopuszczalne są co najwyżej zdarzenia odbioru. Zmiana stanu z pasywnego na aktywny uwarunkowana jest osiągnięciem gotowości przez choćby jedno z dopuszczalnych zdarzeń odbioru, czyli spełnieniem tak zwanego **warunku uaktywnienia**.

### Warunek uaktywnienia
<sub>rso_sum_01.pdf, slajdy 74–77</sub>

> **Warunek uaktywnienia** (ang. _activation condition_) procesu $P_i$ związany jest ze **zbiorem warunkującym** $\mathcal{D}_i$, zbiorem $\mathcal{P}_i^{A}$ oraz predykatem $activate_i(\mathcal{X})$.

> **Zbiór warunkujący** (ang. _dependent set_) jest sumą mnogościową zbiorów $\mathcal{P}_i^{S}$ wszystkich zdarzeń odbioru dopuszczalnych w danej chwili.

**Predykat $activate_i(\mathcal{X})$**:
1. jeżeli $\mathcal{X} = \mathcal{D}_i$, to $activate_i(\mathcal{X}) = True$
2. jeżeli $\mathcal{X} = \varnothing$, to $activate_i(\mathcal{X}) = False$
3. jeżeli $\mathcal{X} \subset \mathcal{D}_i$ i $\mathcal{X} \neq \varnothing$, to
   $$activate_i(\mathcal{X}) \equiv \exists \mathcal{X}' :: \mathcal{X}' \neq \varnothing \wedge \mathcal{X}' \subseteq \mathcal{X} \wedge (\mathcal{P}_i^{A} = \mathcal{X}' \Rightarrow (passive_i \rightsquigarrow \neg passive_i)) \tag{2.15}$$
   gdzie $passive_i \rightsquigarrow \neg passive_i$ oznacza, że pasywny proces $P_i$ zmieni swój stan na aktywny w skończonym, choć nieprzewidywalnym czasie.

**Predykat $ready$** — formalny zapis warunku uaktywnienia procesu $P_i$:
$$ready_i(\mathcal{X}) \equiv (\mathcal{P}_i^{A} \supseteq \mathcal{X}) \wedge activate_i(\mathcal{X}) \tag{2.17}$$

Gdy proces jest uaktywniany, wiadomości, których dostarczanie doprowadziło do spełnienia warunku uaktywnienia, są **atomowo** pobierane z buforów wejściowych i dalej przetwarzane.

### Modele żądań
<sub>rso_sum_01.pdf, slajdy 78–86</sub>

Wyróżnia się modele: **jednostkowy**, **AND**, **OR**, podstawowy **k spośród r**, **OR-AND**, dysjunkcyjny **k spośród r**, **predykatowy**.

| Model | Warunek uaktywnienia pasywnego procesu | Uwagi |
|---|---|---|
| **jednostkowy** | przybycie wiadomości od **jednego, ściśle określonego nadawcy**; $\lvert\mathcal{D}_i\rvert = 1$ dla każdego $i$, $1 \leqslant i \leqslant n$ | odpowiada klasie systemów, w których procesy żądają kolejno po jednym tylko zasobie |
| **AND** | dotarły wiadomości od **wszystkich** procesów tworzących zbiór warunkujący | zwany też **modelem zasobowym** |
| **OR** | wystarczy **jedna** wiadomość od któregokolwiek z procesów ze zbioru warunkującego | zwany też **modelem komunikacyjnym** |
| **podstawowy k spośród r** | wiadomości od co najmniej $k_i$ **różnych** procesów ze zbioru $\mathcal{D}_i$; $1 \leqslant k_i \leqslant \lvert\mathcal{D}_i\rvert$, $r_i = \lvert\mathcal{D}_i\rvert$ | — |
| **OR-AND** | $\mathcal{D}_i = \mathcal{D}_i^{1} \cup \mathcal{D}_i^{2} \cup \ldots \cup \mathcal{D}_i^{q_i}$, gdzie $\mathcal{D}_i^{u} \subseteq \mathcal{P}$ dla $1 \leqslant u \leqslant q_i$; proces staje się aktywny po otrzymaniu wiadomości od **każdego** z procesów tworzących zbiór $\mathcal{D}_i^{1}$ **lub** od każdego ze zbioru $\mathcal{D}_i^{2}$ **lub** … **lub** od każdego ze zbioru $\mathcal{D}_i^{q_i}$ | — |
| **predykatowy** | dla każdego pasywnego procesu $P_i$ ze zbiorem $\mathcal{D}_i$ określony jest predykat $activate_i(\mathcal{X})$, gdzie $\mathcal{X} \subseteq \mathcal{P}$ | stosownie definiując $activate_i(\mathcal{X})$ można uzyskać **wszystkie** wcześniej omówione modele żądań |

> [!tip] Powiązanie
> Modele **AND** i **OR** wracają w wykładzie 7 jako modele zakleszczenia — zob. [[RSO 06 Zakleszczenie w systemach rozproszonych]].

### Relacja poprzedzania zdarzeń
<sub>rso_sum_01.pdf, slajdy 87–89</sub>

**Relacja poprzedzania** $\mapsto$ (ang. _happen before_, _causal precedence_, _happened before_) zdefiniowana na zbiorze $\Lambda$:

$$E_i^{k} \mapsto E_j^{l} \iff \begin{cases} 1)\ i = j \wedge k < l,\ \text{lub} \\ 2)\ i \neq j\ \text{oraz}\ E_i^{k}\ \text{jest zdarzeniem}\ e\_send(P_i, P_j, M),\ \text{a}\ E_j^{l}\ \text{jest zdarzeniem}\ e\_receive(P_i, P_j, M)\ \text{odbioru tej samej wiadomości, lub} \\ 3)\ \text{istnieje sekwencja zdarzeń}\ E^{0}, E^{1}, E^{2}, \ldots, E^{s},\ \text{taka że}\ E^{0} = E_i^{k},\ E^{s} = E_j^{l}\ \text{i dla każdej pary}\ \langle E^{u}, E^{u+1}\rangle,\ 0 \leqslant u \leqslant s-1,\ \text{zachodzi 1) albo 2)} \end{cases}$$

**Relacja poprzedzania lokalnego** $\mapsto_i$: $E_i^{k} \mapsto_i E_j^{l}$ wtedy i tylko wtedy, gdy $i = j$ oraz $k < l$ (lub gdy $i = j$ oraz $E_i^{k} \mapsto E_j^{l}$).

Zdarzenia $E_i^{k}$ i $E_j^{l}$ nazywamy **przyczynowo-zależnymi**, jeżeli
$$E_i^{k} \mapsto E_j^{l} \quad\text{albo}\quad E_j^{l} \mapsto E_i^{k} \tag{3.9}$$
W przeciwnym razie nazywamy je **przyczynowo-niezależnymi** lub **współbieżnymi** (ang. _concurrent_, _causally independent_), co oznaczamy $E_i^{k} \parallel E_j^{l}$.

### Diagramy przestrzenno-czasowe
<sub>rso_sum_01.pdf, slajdy 90–91</sub>

Realizację przetwarzania rozproszonego można przedstawić graficznie w postaci **diagramu przestrzenno-czasowego** (ang. _space-time diagram_), w którym osie reprezentują upływ czasu globalnego, a punkty na osiach — zdarzenia.

![[rso-w1-s91-diagram-przestrzenno-czasowy.png]]
<sub>Przykładowy diagram — rso_sum_01.pdf, slajd 91</sub>

### Stany lokalne, stany współbieżne, graf stanów osiągalnych
<sub>rso_sum_01.pdf, slajdy 92–95</sub>

**Relacja poprzedzania stanów lokalnych** — przez analogię do relacji na zbiorze zdarzeń, częściowy porządek na zbiorze stanów wszystkich procesów $P_i \in \mathcal{P}$:
$$S_i^{k} \mapsto S_j^{l} \iff \begin{cases} E_i^{k+1} \mapsto E_j^{l},\ \text{lub} \\ E_i^{k+1} = E_j^{l} \end{cases} \tag{3.10}$$

Stany lokalne, dla których nie zachodzi ani relacja $S_i^{k} \mapsto S_j^{l}$, ani $S_j^{l} \mapsto S_i^{k}$, nazywamy **współbieżnymi**.

**Graf stanów osiągalnych**: zbiór częściowo uporządkowany $\langle \Lambda, \mapsto \rangle$ może być przedstawiony w postaci grafu zorientowanego, w którym wierzchołki odpowiadają stanom $\Sigma$, a łuki $\langle \Sigma^{k}, \Sigma^{l} \rangle$ oznaczają istnienie zdarzenia dopuszczalnego takiego, że
$$\langle \Sigma^{k}, E, \Sigma^{l} \rangle \in \Phi \tag{3.11}$$
Graf taki nazywamy **grafem stanów osiągalnych przetwarzania rozproszonego** lub **siatką obliczeń rozproszonych**.

![[rso-w1-s95-graf-stanow-osiagalnych.png]]
<sub>Przykład grafu stanów osiągalnych — rso_sum_01.pdf, slajd 95</sub>

### Niedeterminizm przetwarzania
<sub>rso_sum_01.pdf, slajdy 96–99</sub>

W kontekście grafu stanów osiągalnych każda realizacja przetwarzania rozproszonego jest pewną **ścieżką** w tym grafie. Istnienie wielu różnych ścieżek ilustruje **niedeterminizm** przetwarzania rozproszonego: dla danego stanu może istnieć wiele stanów następnych.

- **Niedeterministyczne zdarzenie lokalne** — gdy jego zajście może być zastąpione zajściem innego zdarzenia i wybór ten nie jest przewidywalny. Jeżeli przykładowo sekwencyjne wykonanie procesu może być w każdej chwili zmienione w wyniku zajścia **przerwania zewnętrznego**, to wszystkie zdarzenia tego procesu są niezdeterminowane.
- Przetwarzanie nazywamy **zdeterminowanym**, jeżeli wszystkie zdarzenia są zdeterminowane; w przeciwnym wypadku — **niedeterministycznym**.
- **Przetwarzanie quasi-deterministyczne** (ang. _quasi-deterministic_, _piece-wice deterministic_, _event-driven_) — podklasa przetwarzania niedeterministycznego, w której niedeterminizm jest **wyłącznie konsekwencją niedeterminizmu operacji odbioru**.

### Diagramy równoważne
<sub>rso_sum_01.pdf, slajdy 100–101</sub>

W ogólności istnieje wiele różnych diagramów przestrzenno-czasowych, którym odpowiada **taki sam zbiór częściowo uporządkowany** $\langle \Lambda, \mapsto \rangle$. Diagramy takie nazywa się **diagramami równoważnymi**.

![[rso-w1-s101-diagramy-rownowazne.png]]
<sub>Przykład diagramów równoważnych — rso_sum_01.pdf, slajd 101</sub>

---
## Powiązania
- Relacja poprzedzania i diagramy przestrzenno-czasowe są podstawą zegarów logicznych → [[RSO 08 Czas wirtualny i złożoność algorytmów]]
- Stan procesu, stan kanału i stany współbieżne są podstawą konstrukcji stanu globalnego → [[RSO 09 Stan globalny i migawki]]
- Procesy aktywne/pasywne i warunek uaktywnienia są podstawą detekcji zakończenia → [[RSO 10 Detekcja zakończenia]]
- Modele AND/OR wracają jako modele zakleszczenia → [[RSO 06 Zakleszczenie w systemach rozproszonych]]
- Operacje grupowe $send(P_i, \mathcal{P}_i^{R}, M)$ są podstawą mechanizmów rozgłaszania → [[RSO 01 Komunikacja grupowa]]
