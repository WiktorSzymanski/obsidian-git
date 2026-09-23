---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 6
---
# 6. Zakleszczenie w systemach rozproszonych
---
> Komplet pojęć zagadnienia z notacją. Algorytmy są nazwane i opatrzone informacją, jaki problem rozwiązują, ale ich przebieg, pseudokod i przykłady na grafach pozostają w [[RSO 06 Zakleszczenie w systemach rozproszonych|RSO 06]]. Materiał, którego prezentacje nie zawierają, jest oznaczony blokami `[!note]`.

## Definicja

Procesy przetwarzania rozproszonego wymieniają komunikaty: jedne wysyłają **żądania przydziału zasobów**, inne odsyłają **potwierdzenia przydziału**. Może wtedy wystąpić sytuacja, w której **wszystkie procesy pewnego niepustego zbioru oczekują na wiadomości od innych procesów tego samego zbioru** — i to jest **zakleszczenie rozproszone** (*distributed deadlock*).

Formalnie przez $deadlock(\mathcal{B})$ oznacza się predykat stwierdzający, że w danej chwili $\tau$ niepusty zbiór procesów $\mathcal{B}$ jest zbiorem procesów zakleszczonych. Cała formalna część zagadnienia polega na rozpisaniu tego predykatu — inaczej dla każdego modelu żądań.

**Warunki konieczne** zakleszczenia są cztery: **wzajemne wykluczanie**; **istnienie procesu, który blokuje zasób i jednocześnie czeka na zasób blokowany przez inny proces** (przetrzymywanie i oczekiwanie); **brak wywłaszczania zasobów**; **czekanie cykliczne**.

## Reprezentacje graficzne

**Graf przydziału zasobów** wiąże procesy z zasobami i może być rozpięty na wielu stanowiskach. Do wykrywania zakleszczeń używa się jednak **grafu oczekiwania** (*Wait-For Graph*, WFG), w którym wierzchołkami są procesy, a łuk $P_i \rightarrow P_j$ oznacza, że **$P_i$ oczekuje na wiadomość od $P_j$**. Obowiązują dwie zasady odczytu: proces z łukiem wychodzącym jest **pasywny**, proces bez łuków wychodzących — **aktywny**.

WFG obejmujący cały system to **globalny graf oczekiwania**, a ograniczony do jednego stanowiska — **lokalny graf oczekiwania**. Sedno trudności: **globalny WFG nigdzie nie istnieje w całości** — każdy proces zna wyłącznie swoje krawędzie wychodzące, więc wykrycie zakleszczenia wymaga algorytmu rozproszonego.

## Strategie postępowania

Wobec zakleszczeń można: **nie dopuszczać** do nich, **dopuszczać i później usuwać** albo **ignorować** je. Niedopuszczanie dzieli się na **zapobieganie** i **unikanie**.

**Zapobieganie** w systemie rozproszonym realizuje się przez **priorytety** procesów przy dostępie do zasobów, a żeby nie zagłodzić procesów o niskich priorytetach — przez **znaczniki czasowe** i dwie strategie: **czekanie albo śmierć** (*wait-die*) oraz **zranienie albo czekanie** (*wound-wait*). Wadą obu są **niepotrzebne wywłaszczenia**: wycofuje się procesy, które wcale nie były zakleszczone.

## Stany procesu i warunek uaktywnienia

Formalizm detekcji opiera się na tym samym aparacie co model środowiska przetwarzania. Proces jest w każdej chwili **aktywny** albo **pasywny**: aktywny może realizować zdarzenia wewnętrzne i komunikacyjne, pasywny ($passive_i = \text{True}$) dopuszcza **co najwyżej zdarzenia odbioru**.

Przejście z pasywnego w aktywny wymaga spełnienia **warunku uaktywnienia** (*activation condition*), związanego ze **zbiorem warunkującym** $\mathcal{D}_i$ (*dependent set*) — sumą zbiorów procesów wszystkich dopuszczalnych w danej chwili zdarzeń odbioru — oraz z predykatem
$$ready_i(\mathcal{X}) \equiv (\mathcal{P}_i^{A} \supseteq \mathcal{X}) \wedge activate_i(\mathcal{X})$$
Wiadomości, które doprowadziły do spełnienia warunku, są pobierane z buforów **atomowo**. Pełna definicja: [[RSO 07 Model środowiska przetwarzania#Warunek uaktywnienia|RSO 07]].

To właśnie **postać predykatu $activate_i$ definiuje model żądań** — a model żądań definiuje, czym jest zakleszczenie.

## Model AND i model OR

W modelu **AND**, zwanym też **zasobowym**, proces pasywny staje się aktywny dopiero wtedy, gdy dotarły wiadomości **od wszystkich** procesów zbioru warunkującego:
$$
\begin{aligned}
deadlock(\mathcal{B}) \equiv\ & (\mathcal{B} \subseteq \mathcal{P}) \wedge (\mathcal{B} \neq \varnothing)\ \wedge \\
& (\forall P_i :: P_i \in \mathcal{B} :: (\, passive_i\ \wedge \\
& \quad (\exists P_j :: P_j \in \mathcal{D}_i \cap \mathcal{B} :: (\neg\, \textit{in-transit}_i[j] \wedge \neg\, available_i[j]))\,))
\end{aligned}
$$

W modelu **OR**, zwanym **komunikacyjnym**, wystarczy **jedna** wiadomość od któregokolwiek procesu zbioru warunkującego:
$$
\begin{aligned}
deadlock(\mathcal{B}) \equiv\ & (\mathcal{B} \subseteq \mathcal{P}) \wedge (\mathcal{B} \neq \varnothing)\ \wedge \\
& (\forall P_i :: P_i \in \mathcal{B} :: (\, passive_i\ \wedge\ \mathcal{D}_i \subseteq \mathcal{B}\ \wedge \\
& \quad (\forall P_j :: P_j \in \mathcal{D}_i :: (\neg\, \textit{in-transit}_i[j] \wedge \neg\, available_i[j]))\,))
\end{aligned}
$$

Różnica sprowadza się do dwóch miejsc. Kwantyfikator po $P_j$ jest w modelu AND **egzystencjalny** — wystarczy **jedna** brakująca wiadomość, by proces pozostał pasywny — a w modelu OR **ogólny**: brakować muszą **wszystkie**. Dodatkowo model OR żąda $\mathcal{D}_i \subseteq \mathcal{B}$, czyli **cały zbiór warunkujący musi być zakleszczony**, podczas gdy w AND wystarczy $P_j \in \mathcal{D}_i \cap \mathcal{B}$. **Zakleszczenie w modelu OR jest więc trudniejsze do wystąpienia**: pojedynczy aktywny proces w zbiorze warunkującym wystarczy, by proces nie był zakleszczony.

Przekłada się to wprost na warunek grafowy: w modelu AND zakleszczeniu odpowiada **cykl** w WFG, w modelu OR — **węzeł** (*knot*), czyli zbiór wierzchołków, z których osiągalne są wyłącznie wierzchołki tego zbioru. W modelu OR **cykl nie oznacza zakleszczenia**, bo proces może zostać odblokowany krawędzią prowadzącą poza cykl.

## Klasyfikacja problemów detekcji

Prezentacje rozróżniają cztery różne pytania, które potocznie nazywa się tak samo — „wykryć zakleszczenie":

| Problem | Predykat |
|---|---|
| **detekcja wystąpienia** zakleszczenia | $dE \equiv (\exists \mathcal{B} :: deadlock(\mathcal{B}))$ |
| **detekcja zakleszczenia procesu $P_i$** | $dP_i \equiv ((\exists \mathcal{B} :: deadlock(\mathcal{B})) \wedge P_i \in \mathcal{B})$ |
| **detekcja zbioru procesów** $\mathcal{B}^{*}$ | $deadlock(\mathcal{B}^{*}) \vee ((\mathcal{B}^{*} = \varnothing) \wedge (\nexists \mathcal{B} :: deadlock(\mathcal{B})))$ |
| **detekcja zbioru maksymalnego** | $(deadlock(\mathcal{B}^{*}) \vee \mathcal{B}^{*} = \varnothing) \wedge maxdead(\mathcal{B}^{*})$, gdzie $maxdead(\mathcal{B}^{*}) \equiv (\forall \mathcal{B} :: deadlock(\mathcal{B}) \Rightarrow (\mathcal{B} \subseteq \mathcal{B}^{*}))$ |

W modelu aplikacyjnym proces wysyła do procesów swojego zbioru warunkującego wiadomości **REQUEST**, a otrzymuje **GRANT** albo **CANCEL**; detekcja dokłada do tego własną warstwę komunikatów sterujących.

## Algorytmy Chandy-Misra-Haas

**CMH dla modelu AND** rozwiązuje detekcję zakleszczenia procesu techniką **pogoni za krawędziami** (*edge-chasing*): zablokowany proces rozsyła **sondę** ze swoim identyfikatorem wzdłuż krawędzi WFG, a **powrót sondy do inicjatora** ($\alpha_i = i$) dowodzi istnienia cyklu. Sondę propaguje się tylko wtedy, gdy proces jest **pasywny**, sonda tego inicjatora **jeszcze przez niego nie przeszła** (co zapobiega zapętleniu) i zasób od nadawcy **nie został przyznany**. Przy uaktywnieniu procesu znaczniki przejścia sond są czyszczone. Przebieg: [[RSO 06 Zakleszczenie w systemach rozproszonych#Algorytm Chandy-Misra-Haas dla modelu AND|RSO 06]].

**CMH dla modelu OR** rozwiązuje ten sam problem **przetwarzaniem dyfuzyjnym** (*query computation*). Zablokowany proces rozsyła **QUERY** do swojego zbioru warunkującego; proces **aktywny unieważnia** każde QUERY i REPLY, które do niego dotrze, i to właśnie milczenie procesu aktywnego „ratuje" inicjatora przed fałszywym wykryciem. Proces zablokowany propaguje **pierwsze** zapytanie danego inicjatora (*engaging query*), zapamiętując liczbę wysłanych zapytań w liczniku $num_k(i)$, a na zapytania kolejne odpowiada od razu, o ile jest zablokowany nieprzerwanie od pierwszego ($wait_k(i)$). **REPLY** odsyła się dopiero po zebraniu odpowiedzi na wszystkie własne zapytania, a **inicjator deklaruje zakleszczenie**, gdy jego licznik zejdzie do zera.

Poprawność ujmuje **twierdzenie 5.2** w dwóch częściach: jeżeli inicjator rozpoczyna detekcję, będąc zakleszczonym, to **stwierdzi to w skończonym czasie** (zakończenie), a jeżeli deklaruje zakleszczenie, to **należy do pewnego zbioru procesów zakleszczonych** (poprawność). Przebieg: [[RSO 06 Zakleszczenie w systemach rozproszonych#Detekcja zakleszczenia dla modelu OR|RSO 06]].

Struktura algorytmu OR jest **identyczna z algorytmem Dijkstry-Scholtena** detekcji zakończenia: *engaging query* odpowiada rodzicowi, $num_k(i)$ — licznikowi wysłanych wiadomości, a REPLY — sygnałowi. Zob. [[RSO 10 Detekcja zakończenia#Model przetwarzania dyfuzyjnego i algorytm Dijkstry-Scholtena|RSO 10]].

| | model AND | model OR |
|---|---|---|
| **Mechanizm** | sonda rozsyłana po WFG | przetwarzanie dyfuzyjne (QUERY/REPLY) |
| **Wykrycie** | sonda **wraca do inicjatora** | inicjator dostał REPLY na **wszystkie** QUERY |
| **Stan lokalny** | znaczniki przejścia sondy, przyznania zasobów | $num_k(i)$, $wait_k(i)$ |
| **Odpowiedzi zwrotne** | brak | REPLY wraca do rodzica |
| **Koszt** | $\leq \lvert E \rvert$ komunikatów | $2\lvert E \rvert$ komunikatów |

> [!note] Uzupełnienie spoza prezentacji — klasy algorytmów wykrywania
> Poza dwoma powyższymi wyróżnia się algorytmy **scentralizowane** (koordynator buduje globalny WFG, np. Ho-Ramamoorthy — ryzyko fantomów i SPoF), **przepychanie ścieżek** (*path-pushing*, Obermarck — węzły przesyłają sobie fragmenty ścieżek WFG) oraz oparte na **stanie globalnym** (migawka WFG i jej analiza). Źródło: [[06 Zakleszczenie w systemach rozproszonych]].

> [!note] Uzupełnienie spoza prezentacji — Bracha-Toueg
> Algorytm Brachy-Touega (1987) rozwiązuje detekcję w ogólnym modelu **$p$-z-$q$**, obejmującym AND ($p=q$) i OR ($p=1$). Idea: najpierw **migawka** spójnego stanu globalnego WFG, a potem **symulacja** rozdawania przydziałów — proces, który w symulacji zbierze wymagane $p$ przydziałów, sam staje się wolny i „rozdaje" przydziały dalej wstecz. Proces, którego symulacja nie odblokuje, jest zakleszczony; inicjator jest zakleszczony wtedy i tylko wtedy, gdy na koniec nie został uznany za wolnego. Cztery typy komunikatów (powiadomienie, przydział i ich potwierdzenia) dają koszt $O(\lvert E \rvert)$ na fazę. **Nie mylić** z probabilistycznym algorytmem konsensusu tych samych autorów: [[Systemy Wysokiej Niezawodności/Algorytm Bracha-Touega]]. Źródło: [[06 Zakleszczenie w systemach rozproszonych]].

> [!note] Uzupełnienie spoza prezentacji — fantomy i usuwanie zakleszczenia
> Algorytm wykrywania musi spełniać **postęp** (każde zakleszczenie wykryte w skończonym czasie) i **bezpieczeństwo**, czyli brak **zakleszczeń fantomowych** — fałszywych wykryć spowodowanych nieaktualnym, niespójnym obrazem WFG, gdy np. informacja o zwolnieniu zasobu jeszcze nie dotarła do detektora. **Usuwanie** polega na wyborze ofiary (najmłodszy proces, najmniejszy koszt wycofania), jej wycofaniu i zwolnieniu zasobów, a następnie na **usunięciu jej krawędzi z WFG u wszystkich zainteresowanych** — inaczej pozostałości sond powodują wykrycia fantomowe. Przy wielu inicjatorach naraz porządkuje się je priorytetem po identyfikatorze, żeby nie wycofać kilku procesów z tego samego cyklu. Źródło: [[06 Zakleszczenie w systemach rozproszonych]].

---
## Czego w prezentacjach nie ma

> [!todo] Zakres zagadnienia wykracza poza slajdy
> Prezentacje nie zawierają: algorytmu **Brachy-Touega** (0 trafień), **złożoności** algorytmów CMH, **dowodów** twierdzeń, problemu **zakleszczeń fantomowych**, **metod usuwania** zakleszczeń (wybór ofiary, wywłaszczenie, wycofanie) ani definicji zakleszczenia dla modeli żądań innych niż AND i OR (*$k$ spośród $r$*, OR-AND, predykatowy — wprowadzonych w [[RSO 07 Model środowiska przetwarzania#Modele żądań|RSO 07]]). Rejestr braków: [[RSO 06 Zakleszczenie w systemach rozproszonych#Braki w prezentacjach]]. Materiał uzupełniający: [[06 Zakleszczenie w systemach rozproszonych]], [[Synchronizacja]], [[Awarie]].
