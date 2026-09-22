---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
source: "FT_Recovery-algorytmy.pdf"
strony: "39–56"
zagadnienie: 22
---
# SWN 04. Algorytmy odtwarzania — opis szczegółowy (Koo-Toueg, Wang-Fuchs, Manetho)
---
> Materiał pomocniczy do wykładów [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane|I]]–[[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny|III]]: rozdział **3.3.3 „Przegląd algorytmów realizujących technikę tolerowania uszkodzeń"** z pracy Baczyńskiego, Kędziory, Drewniaka i Stryczyńskiego *„Narzędzia uruchomieniowe programów rozproszonych z rozproszoną pamięcią współdzieloną o podwyższonej niezawodności"* (strony 39–56). Dokument podaje **pełny pseudokod** trzech algorytmów — **Koo-Touega**, **Wanga-Fuchsa** i **Manetho** — w szczegółowości, której slajdy nie mają.

> [!warning] O tym pliku
> `FT_Recovery-algorytmy.pdf` ma **uszkodzoną warstwę tekstową** (część runów przesunięta o +29 w ASCII, utracone polskie znaki diakrytyczne). Cała poniższa treść została odczytana **z obrazu stron**, nie z warstwy tekstowej.

> [!info] Czego tu nie ma
> Dokument omawia **tylko trzy** algorytmy. **Juanga-Venkatesana** i **Manivannana-Singhala** w nim nie ma — te są wyłącznie na slajdach ([[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Algorytm Juanga-Venkatesana]], [[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny#Checkpointing quasi-synchroniczny — algorytm Manivannana-Singhala]]).

---
## Algorytm Koo-Toueg
<sub>FT_Recovery-algorytmy.pdf, s. 39–47</sub>

> Koo i Toueg zaproponowali efektywny algorytm wstecznego odtwarzania stanu w **technice skoordynowanego tworzenia punktu kontrolnego**.

### Założenia
<sub>s. 39</sub>

- procesy komunikują się **tylko** poprzez wymianę komunikatów przez kanały,
- kanały są **niezawodne**: komunikaty nie mogą być duplikowane, tracone oraz są bezbłędne,
- kanały zapewniają dostarczenie komunikatów **z zachowaniem kolejności**, tj. są kanałami **FIFO**,
- procesy **zatrzymują się** podczas uszkodzenia; każdorazowo, gdy proces zostaje uszkodzony, wszystkie inne procesy są informowane o błędzie **w skończonym czasie**,
- uszkodzenie procesu **nie dzieli sieci** komunikacyjnej,
- obliczenia są w ogólności **niedeterministyczne**,
- w odniesieniu do spójności: wszystkie komunikaty są **znaczące** (powodują zmianę stanu), wszystkie **gubione komunikaty są akceptowalne** (istnieje mechanizm odnawiania i powtórnego przesłania utraconych komunikatów), jednak **w systemie nie występują komunikaty osierocone**,
- wszystkie działania mogą być przywrócone (wszystkie procesy mogą być cofnięte), tj. **nie ma interakcji ze światem zewnętrznym**.

> Algorytm ustanawiania punktu kontrolnego oraz algorytm odtworzenia oparte są na **przetwarzaniu dyfuzyjnym** i **dwufazowym protokole zatwierdzania** (*two-phase-commit protocol*).

> [!important] To domyka lukę ze slajdów
> Slajd 28 wykładu I mówi tylko „wykorzystuje ideę przetwarzania dyfuzyjnego i 2PC", bez wyjaśnienia. Blok **S4** poniżej pokazuje mechanizm dyfuzji wprost: kontroler, który przyjął żądanie, **sam rozsyła je dalej** do swojej kohorty i czeka na odpowiedzi, zanim odpowie żądającemu.

### Etykiety komunikatów
<sub>s. 40</sub>

Dla uproszczenia zakłada się, że jest **tylko jeden inicjujący kontroler**. Do każdej wiadomości $m$ wysyłanej przez proces aplikacyjny $P_i$ kontroler $C_i$ dołącza etykietę $send\_num_i$; konkatenacja komunikatu i etykiety oznaczana jest przez „$m * send\_num_i$". Etykiety są **inkrementowane przez kontrolery przed wysłaniem** komunikatu, więc $send\_num_i$ w każdej chwili reprezentuje **liczbę komunikatów aplikacyjnych wysłanych od $P_i$ do tej chwili**. Możliwe wartości etykiety zawierają się w przedziale **od 0 do MAX**.

### Działanie podczas bezbłędnej pracy
<sub>s. 40–41</sub>

Algorytm zapisuje w pamięci trwałej **dwa typy punktów kontrolnych**: **trwały** (*permanent*) i **wstępny** (*tentative*).
- **Trwały** punkt kontrolny jest lokalnym punktem kontrolnym procesu i jest **częścią spójnej linii odzysku** (*recovery line*) wykorzystywanej do odtworzenia stanu.
- **Wstępny** punkt kontrolny jest tymczasowy — staje się trwały, kiedy algorytm ustanawiania globalnego spójnego punktu kontrolnego **zakończy się pomyślnie**.
- Stan początkowy wszystkich procesów jest zapisywany jako trwały punkt kontrolny, aby utworzyć **początkową spójną linię odzysku**.

**Minimalizacja liczby procesów zapisujących punkt kontrolny:** kontroler procesu $P_i$ zapisuje wstępny punkt kontrolny **tylko wtedy**, gdy istnieje proces $P_j$, którego wstępny punkt kontrolny zarejestrował **otrzymanie** komunikatu od $P_i$, a ostatni **trwały** punkt kontrolny procesu $P_i$ **nie zarejestrował wysłania** tego komunikatu.

**Struktury kontrolne.** Dla każdych dwóch procesów $P_i$, $P_j$ niech $m$ będzie **ostatnim** komunikatem, jaki $P_i$ otrzymał od $P_j$ po zapisaniu przez $P_i$ trwałego lub wstępnego punktu kontrolnego:
$$last\_rcvd\_msg_i[j] = \begin{cases} send\_num_j & \text{gdy } m \text{ istnieje} \\ 0 & \text{w przeciwnym przypadku} \end{cases}$$

Niech $m$ będzie **pierwszym** komunikatem, który $P_i$ wysłał do $P_j$ po zapisaniu przez $P_i$ trwałego lub wstępnego punktu kontrolnego:
$$first\_sent\_msg_i[j] = \begin{cases} send\_num_j & \text{gdy } m \text{ istnieje} \\ 0 & \text{w przeciwnym przypadku} \end{cases}$$

**Warunek zapisania wstępnego punktu kontrolnego** (s. 41). Kiedy $C_i$ żąda od $C_j$ zapisania wstępnego punktu kontrolnego, wysyła komunikat razem z etykietą $last\_rcvd\_msg_i[j]$. W odpowiedzi $C_j$ zapisuje wstępny punkt kontrolny **tylko wtedy**, gdy:
$$last\_rcvd\_msg_i[j] \geqslant first\_sent\_msg_j[i] > 0$$
tj. kiedy **istnieje komunikat zapisany przez $C_i$ jako otrzymany, ale jeszcze nie zapisany przez $C_j$ jako wysłany**.

**Kohorta:**
$$ckpt\_cohort_i = \{C_j : last\_rcvd\_msg_i[j] > 0\}$$
Zbiór ten zawiera kontrolery wszystkich procesów $P_j$, od których $P_i$ otrzymał komunikat **po** zapisaniu przez $C_i$ ostatniego punktu kontrolnego.

**Dziedziczenie żądania.** Każdy kontroler $C_i$ przetrzymuje zmienną $agree\_to\_take\_ckpt_i$. Gdy $C_i$ otrzyma od innego kontrolera żądanie $m = \text{„}take\_tentative\_ckpt * last\_rcvd\_msg_j[i]\text{"}$, **dziedziczy** to żądanie pod warunkiem, że:
$$(agree\_to\_take\_ckpt_i) \wedge (last\_rcvd\_msg_j[i] \geqslant first\_sent\_msg_i[j] > 0)$$

Jeśli $C_i$ odziedziczy żądanie, **zapisuje wstępny punkt kontrolny** i rozpropagowuje żądanie $m = \text{„}take\_tentative\_ckpt * last\_rcvd\_msg_i[r]\text{"}$ do wszystkich $C_r \in ckpt\_cohort_i$, po czym czeka na wszystkie odpowiedzi. Po zebraniu odpowiedzi $C_i$ odsyła do $C_j$ wartość:
$$agree\_to\_take\_ckpt_i \wedge \Big( \bigwedge_{C_r \in ckpt\_cohort_i} agree\_to\_take\_ckpt_r \Big)$$

Jeśli $C_i$ **nie** odziedziczy żądania, odsyła natychmiast odpowiedź $m = \text{„}agree\_to\_take\_ckpt_i\text{"}$.

**Inicjator** (s. 42). Aby utworzyć globalny punkt kontrolny, kontroler $C_\alpha$ (inicjator) wysyła do wybranego $C_i$ żądanie $m = \text{„}take\_tentative\_ckpt * MAX\text{"}$. Zauważmy, że **jeśli $agree\_to\_take\_ckpt_i$ ma wartość *true*, to żądanie to jest zawsze dziedziczone** (bo $MAX \geqslant$ dowolnej etykiety).

Pierwsza faza kończy się, gdy inicjator zbierze wszystkie odpowiedzi. Jeśli wszystkie niosą *true*, inicjator decyduje o **przekształceniu tymczasowych punktów kontrolnych w trwałe**; w przeciwnym przypadku decyduje o **wycofaniu** wszystkich tymczasowych punktów kontrolnych. Gdy tymczasowy punkt kontrolny staje się trwały, **stary trwały punkt kontrolny może zostać odrzucony**, aby odzyskać przestrzeń.

> **Krytyczne dla poprawności:** wstrzymanie wysyłania jakichkolwiek wiadomości przez $P_i$ od momentu, kiedy $C_i$ zapisze wstępny punkt kontrolny, do czasu otrzymania decyzji od inicjatora.

### Bloki algorytmu tworzenia punktu kontrolnego
<sub>s. 42–43</sub>

```
S1: when an initiator C_alpha spontaneously decides to take another global checkpoint,
    C_alpha will do
        send (C_alpha, C_i, "take_tentative_ckpt * MAX");
        rec  (C_alpha, C_i, "agree_to_take_ckpt_i");
        if agree_to_take_ckpt
          then
            send (C_alpha, C_i, "make_tentative_ckpt_permanent");
          else
            send (C_alpha, C_i, "undo_tentative_ckpt");

S2: when P_i wants to send a message m to P_j, C_i will do
        wait (sending_enable_i);
        send_num_i[j] = send_num_i[j] + 1;
        append send_num_i[j] + 1;
        send (C_i, C_j, "m * send_num_i[j]");

S3: when P_i wants to receive a message m from P_j, C_j will do
        rec (C_i, C_j, "m * send_num_i[j]");
        extract send_num_i[j];
        return to P_i parameters of the receive operation: P_i, P_j and m;

S4: when C_i receives from C_j message m = "take_tentative_ckpt * last_rcvd_msg_j[i]",
    C_i will do
        if (agree_to_take_ckpt_i) and (last_rcvd_msg_j[i] >= first_sent_msg_i[j] > 0)
          then
            /* disable sending of messages from P_i */
            sending_enable_i := false;
            take tentative checkpoint of P_i;
            for all C_r in ckpt_cohort_i do
                send (C_i, C_r, "take_tentative_ckpt * last_rcvd_msg_i[r]");
            for all C_r in ckpt_cohort_i do
                rec (C_i, C_r, "agree_to_take_ckpt_r");
            agree_to_take_ckpt_i := (agree_to_take_ckpt_i)
                                     and (AND over C_r in ckpt_cohort_i: agree_to_take_ckpt_r)
            send (C_i, C_j, "agree_to_take_ckpt_i");

S5: when C_i receives from C_j message m, where
        m = "make_tentative_ckpt_permanent" or m = "undo_tentative_ckpt", C_i will do
        if m = "make_tentative_ckpt_permanent"
          then  make tentative checkpoint permanent;
          else  undo tentative checkpoint;
        for all C_r in ckpt_cohort_i do
            send (C_i, C_r, m);
        signal (sending_enable_i);
```

### Działanie podczas fazy *recovery*
<sub>s. 44–45</sub>

Algorytm monitoruje stan wszystkich procesów aplikacyjnych. Gdy jeden z procesów zostanie uszkodzony, inicjowana jest pierwsza faza wycofania: inicjator $C_\beta$ wysyła do wszystkich kontrolerów **żądanie przygotowania się do restartu** procesów aplikacyjnych od ich ostatniego zapisanego **trwałego** punktu kontrolnego i oczekuje na odpowiedzi. Kiedy $C_\beta$ otrzyma wszystkie odpowiedzi pozytywne ($agree\_to\_roll\_back_i$ = *true* dla wszystkich $i$), podejmuje decyzję o restarcie. Jeżeli jakiś proces **nie zaakceptuje** żądania, $C_\beta$ decyduje o **kontynuowaniu normalnej pracy**. Decyzja jest rozpropagowywana do wszystkich kontrolerów.

**Zbiory:**
- $roll\_back\_together_i$ — zbiór kontrolerów $C_j$ takich, że $P_i$ **wysłał komunikat** do $P_j$ po ostatnim utworzeniu punktu kontrolnego; jeżeli $P_i$ wymaga wycofania, $C_i$ powinien rozsyłać żądanie **tylko** do kontrolerów z tego zbioru,
- $roll\_cohort_i$ — zbiór wszystkich kontrolerów $C_j$ takich, że proces $P_j$ **mógł wysłać** komunikat do $P_i$ po ostatnim utworzeniu trwałego punktu kontrolnego; w rzeczywistości jest to **zbiór sąsiadów** węzła,
- zachodzi: $roll\_back\_together_i \subseteq roll\_cohort_i$.

**Struktura.** Dla dowolnej pary $P_i$, $P_j$ niech $m$ będzie **ostatnim** komunikatem, jaki $P_i$ wysłał do $P_j$ po ostatnim utworzeniu **trwałego** punktu kontrolnego:
$$last\_sent\_msg_i[j] = \begin{cases} send\_num_i & \text{gdy } m \text{ istnieje} \\ MAX & \text{w przeciwnym przypadku} \end{cases}$$

Zbiór $roll\_back\_together_i$ uszkodzonego procesu $P_i$ składa się z tych $C_j \in roll\_cohort_i$, dla których:
$$last\_rcvd\_msg_j[i] > last\_sent\_msg_i[j]$$

**Dziedziczenie żądania wycofania** (s. 45). $C_j$, który otrzyma $m = \text{„}prepare\_to\_roll\_back * last\_sent\_msg_i[j]\text{"}$, dziedziczy je pod warunkiem, że:
$$(agree\_to\_roll\_back_j) \wedge (last\_rcvd\_msg_j[i] > last\_sent\_msg_i[j]) \wedge (C_j \text{ nie ma jeszcze odziedziczonych dotąd innych żądań do wycofania})$$

Jeśli $C_j$ odziedziczy żądanie, przygotowuje do restartu $P_j$, rozsyła żądanie do wszystkich $C_r \in roll\_cohort_j$ i czeka na odpowiedzi. Po ich zebraniu ustawia własną odpowiedź:
$$agree\_to\_roll\_back_j = (agree\_to\_roll\_back_j) \wedge \Big( \bigwedge_{C_r \in roll\_cohort_j} agree\_to\_roll\_back_r \Big) \wedge \Big( \bigwedge_{C_r \in roll\_cohort_j} P_r \text{ nie jest uszkodzony} \Big)$$

Aby wycofać przetwarzanie, inicjator $C_\beta$ wysyła do kontrolera **uszkodzonego** procesu komunikat $m = \text{„}prepare\_to\_roll\_back * 0\text{"}$ (etykieta 0 sprawia, że warunek jest zawsze spełniony).

> Również w tym algorytmie krytyczne dla poprawności jest, aby **od momentu otrzymania żądania wycofania do czasu otrzymania decyzji od inicjatora zabronione było wysyłanie jakichkolwiek wiadomości przez $P_i$**.

### Bloki algorytmu odtwarzania
<sub>s. 46–47</sub>

```
S7: when an initiator C_beta decides to roll back computation due to fail_stop of P_i,
    C_beta will do
        send (C_beta, C_i, "prepare_to_roll_back * 0");
        rec  (C_beta, C_i, "agree_to_roll_back_i");
        if agree_to_roll_back_i
          then  send (C_beta, C_i, "roll_back");
          else  send (C_beta, C_i, "do_not_roll_back");

S8: when C_i receives from C_j message
    m = "prepare_to_roll_back * last_sent_msg_j[i]", C_i will do
        if (agree_to_roll_back_i)
           and (last_rcvd_msg_i[j] > last_sent_msg_j[i])
           and (first_msg_to_roll_back)
          then
            /* disable sending messages from P_i */
            sending_enable_i;
            first_msg_to_roll_back_i = false;
            for all C_r in roll_cohort_i do
                send (C_i, C_r, "prepare_to_roll_back * last_sent_msg_i[r]");
            for all C_r in roll_cohort_i do
                rec (C_i, C_r, "agree_to_roll_back_r");
            agree_to_roll_back_i = (agree_to_roll_back_j)
                                    and (AND over C_r in roll_cohort_i: agree_to_roll_back_r)
            send (C_i, C_j, "agree_to_roll_back_i =");

S9: when C_i receives from C_j message m = "roll_back" or m = "do_not_roll_back",
    C_i will do
        if m = "roll_back"
          then
            for all C_r in roll_cohort_i do
                send (C_i, C_r, "roll_back");
            roll_back_i;
          else
            for all C_r in roll_cohort_i do
                send (C_i, C_r, "do_not_roll_back");
            signal (sending_enable_i);
            first_msg_to_roll_back_i = true;
```

> [!note] Uwaga o numeracji bloków
> Dokument numeruje bloki **S1–S5** (checkpointing) i **S7–S9** (odtwarzanie), ponownie wykorzystując S2 i S3 (wysyłanie/odbiór). Bloku **S6 w tekście nie ma** — numeracja przeskakuje.

---
## Algorytm Wanga-Fuchsa
<sub>s. 48–50</sub>

### Założenia
<sub>s. 48</sub>

- procesy komunikują się tylko poprzez wymianę komunikatów,
- kanały komunikacyjne są **niezawodne** i w ogólności **nie muszą spełniać reguły FIFO**,
- awaria procesu polega na jego **zatrzymaniu**; zdarzenie to jest wykrywalne przez inne procesy, po czym może nastąpić wycofanie i wystartowanie zatrzymanego procesu,
- awarie **nie powodują rozdzielenia połączeń** pomiędzy węzłami,
- przetwarzanie jest **niedeterministyczne**,
- ze względu na spójność wszystkie wiadomości są **znaczące**,
- wszystkie zdarzenia w systemie mogą być **wycofane**.

> [!warning] Rozbieżność ze slajdem
> Slajd 21 wykładu II podaje **`deterministic processing (can be relaxed)`**, a ten dokument — **„przetwarzanie jest niedeterministyczne"**. Slajd jest bliższy oryginalnej pracy Wanga i Fuchsa (algorytm zakłada determinizm, który można rozluźnić przy dodatkowych mechanizmach); dokument opisuje wariant po rozluźnieniu. Na obronie bezpieczniej podać wersję ze slajdu z zaznaczeniem, że założenie daje się osłabić.

### Graf punktów kontrolnych i zależność
<sub>s. 48</sub>

Graf składa się z **węzłów** reprezentujących punkty kontrolne oraz **krawędzi** oznaczających relację zależności. $k$-ty punkt kontrolny procesu $P_i$ zapisujemy jako $ckpt_i^{k}$, gdzie $k \geqslant 1$, $1 \leqslant i \leqslant n$, $n$ — liczba procesów.

> **Interwał punktu kontrolnego** — czas pomiędzy dwoma punktami kontrolnymi tego samego procesu; czas pomiędzy $ckpt_i^{k}$ a $ckpt_i^{k+1}$ jest **$k$-tym interwałem** procesu $P_i$.

> Punkt kontrolny $ckpt_j^{\,l}$ jest **bezpośrednio zależny** od $ckpt_i^{k}$ wtedy i tylko wtedy, gdy:
> - $i = j$ oraz $l = k+1$, **lub**
> - $i \neq j$ i podczas $k$-tego interwału procesu $P_i$ została wysłana wiadomość $m$ odebrana przez proces $P_j$ w jego $l-1$ interwale.

### Struktury
<sub>s. 49</sub>

- $ckpt\_num_i$ — numer interwału procesu $P_i$,
- $send\_num_i$ — numer sekwencyjny wiadomości wysłanej przez $P_i$,
- $send\_log_i$ — struktura zawierająca pary $(send\_num_i, j)$, a więc informację **„jaka wiadomość wysłana została do jakiego procesu"**,
- $rec\_log_i$ — struktura zawierająca czwórki $(m, send\_num_j, ckpt\_num_j, j)$, a więc: **„odebrano wiadomość $m$ o numerze sekwencyjnym $send\_num_j$, wysłaną przez proces $P_j$ podczas jego $ckpt\_num_j$ interwału"**.

### Działanie podczas pracy bez awarii
<sub>s. 49</sub>

- do każdej wysyłanej przez $P_i$ wiadomości $m$ kontroler $C_i$ dołącza **zwiększony o jeden** numer sekwencyjny $send\_num_i$ oraz aktualny numer interwału $ckpt\_num_i$,
- kontroler odbiorcy **w sposób transparentny** dla procesu aplikacyjnego rozpakowuje i zapamiętuje przesyłane informacje kontrolne w powyższych strukturach,
- każdy kontroler **spontanicznie i niezależnie** od innych może zdecydować o zapisaniu kolejnego punktu kontrolnego $ckpt_i^{k}$; zapisywane są również struktury $rec\_log_i$ i $send\_log_i$ wraz z uaktualnionym $ckpt\_num_i$, po czym **$rec\_log_i$ i $send\_log_i$ zostają wyzerowane**.

### Działanie podczas awarii
<sub>s. 49–50</sub>

Kiedy kontroler wykryje awarię procesu aplikacyjnego, decyduje o rozpoczęciu fazy wycofania i odtwarzania. Aby prawidłowo wycofać i wznowić przetwarzanie, kontroler musi **skonstruować aktualny graf zależności punktów kontrolnych**. W tym celu rozsyła do wszystkich kontrolerów żądanie o przesłanie znanych im informacji o komunikacji. W odpowiedzi kontrolery procesów pracujących bezawaryjnie **dokonują zapisów punktów kontrolnych** tych procesów, po czym przesyłają zgromadzone informacje: $ckpt_i^{k}$, $rec\_log_i$, $send\_log_i$ oraz $ckpt\_num_i$.

Kontroler inicjujący fazę odtwarzania tworzy **globalny graf zależności**, który rozszerza się o tzw. **krawędzie rollback** (*rollback edges*) reprezentujące **wiadomości nie zapisane jeszcze w logu**. Szukając optymalnej linii odtwarzania, **węzły z wchodzącymi krawędziami rollback muszą zostać odrzucone**, jak również inne węzły osiągalne z tychże.

**Wyszukiwanie linii odtwarzania:** poszukiwanie rozpoczyna się od **jądra grafu** składającego się z węzłów reprezentujących **ostatnie** punkty kontrolne poszczególnych procesów. Jeśli istnieje krawędź łącząca dwa węzły z jądra, **późniejszy** z nich musi zostać zastąpiony w jądrze **wcześniejszym** punktem kontrolnym tego samego procesu. Krok ten jest powtarzany do momentu, w którym **nie ma żadnej krawędzi łączącej węzły w jądrze**. Węzły należące do jądra reprezentują punkty kontrolne tworzące **linię odtwarzania**.

Po wznowieniu wiadomości **przychodzące** są przetwarzane z logu aż do jego wyczerpania; wiadomości **wychodzące** wysyłane są ponownie, dlatego potrzebny jest dodatkowy **mechanizm odrzucający duplikaty**.

---
## Algorytm Manetho
<sub>s. 51–56</sub>

> Autorami są **Elmootazbellah N. Elnozahy** i **Willy Zwaenepoel**. Algorytm realizuje technikę **adaptacyjnego tworzenia punktu kontrolnego**.

### Cechy charakterystyczne
<sub>s. 51</sub>

- **niski narzut** w czasie normalnej pracy dzięki uniknięciu synchronicznego (pesymistycznego) zapisu na trwałym nośniku przez większość czasu,
- **ograniczone cofanie** procesów do punktów kontrolnych — cofane są **tylko procesy niesprawne** i tylko do **ostatnich** punktów kontrolnych,
- **redukcja opóźnienia zatwierdzenia operacji wejścia/wyjścia** (*output commit*) przez asynchroniczne wysyłanie komunikatów do świata zewnętrznego (bez koordynacji wielu procesów),
- **tolerowanie arbitralnej liczby awarii**, wliczając niesprawności, które mogłyby wystąpić **podczas odtwarzania**.

### Założenia
<sub>s. 51–52</sub>

- w przetwarzaniu uczestniczą **moduły odtwarzania** (*fail stop recovery units*), zwane dalej **RU**,
- RU zawierają **jeden lub więcej wątków**, które zmieniają stan RU,
- każdy RU ma dostęp do **trwałego nośnika**,
- po cofnięciu RU może być uruchamiany **na dowolnej dostępnej maszynie**,
- praca RU składa się z **sekwencji deterministycznych zmian stanów**, które rozpoczynają się zdarzeniem niedeterministycznym (odebraniem komunikatu, niedeterministycznym zdarzeniem wewnętrznym); zdarzenie $i$ w module $RU$ $p$ oznacza się $e_i^{\,p}$,
- komunikacja odbywa się tylko przez **asynchroniczną sieć**,
- komunikaty mogą być **gubione, duplikowane, przesyłane w niewłaściwej kolejności, opóźnione** powyżej czasu oczekiwania,
- sieć w przypadku uszkodzenia **nie dzieli się** na oddzielne, działające części,
- dane odebrane przez RU z urządzenia wejścia/wyjścia muszą być **zapisane na trwałym nośniku**, zanim RU wyśle komunikat do innego RU lub urządzenia.

### Graf poprzedzania
<sub>s. 52</sub>

> Graf poprzedzania zdarzenia $e_i^{\,p}$, $AG(e_i^{\,p})$, jest **grafem skierowanym acyklicznym**. Zawiera węzeł reprezentujący zdarzenie $e_i^{\,p}$ oraz węzły reprezentujące **wszystkie zdarzenia poprzedzające** $e_i^{\,p}$.

- w wyniku zajścia zdarzenia **odebrania komunikatu** odpowiedni węzeł posiada **dwie krawędzie wejściowe**: z węzła reprezentującego poprzednie zdarzenie w tym samym module oraz z węzła reprezentującego **zdarzenie nadania** tego komunikatu (zdarzenie nadania jest traktowane jako zapoczątkowane przez jakieś zdarzenie wewnętrzne),
- węzeł reprezentujący **niedeterministyczne zdarzenie wewnętrzne** posiada **tylko jedną krawędź** — z węzła reprezentującego poprzednie zdarzenie w tym samym module.

![[swn-rec-s52-rys8-przyklad-przetwarzania.png]]
<sub>Rysunek 8: przykładowe przetwarzanie rozproszone — FT_Recovery-algorytmy.pdf, s. 52</sub>

![[swn-rec-s52-rys9-graf-poprzedzania.png]]
<sub>Rysunek 9: odpowiadający mu graf poprzedzania — s. 52</sub>

**Zawartość węzłów** (s. 53): typ (zdarzenie odebrania komunikatu lub zdarzenie wewnętrzne), identyfikator odbiorcy, identyfikator nadawcy, indeks zdarzenia, unikalny identyfikator komunikatu. **Węzły nie zawierają kopii danych komunikatu** — graf niesie informacje użyteczne przy **powtarzaniu zdarzeń** podczas wstecznego odtwarzania.

### Działanie w czasie normalnej pracy
<sub>s. 53</sub>

- każdy RU utrzymuje **w pamięci ulotnej** graf AG bieżącego zdarzenia oraz **zapis historii komunikatów** (treść i nadawcę każdego odebranego komunikatu),
- przy nadaniu komunikatu RU przesyła **razem z komunikatem** graf AG bieżącego stanu,
- dla zmniejszenia ilości przesyłanej informacji stosuje się **przesłanie tylko pewnych części grafu** (*incremental piggybacking*): z definicji $AG(e_i^{\,p})$ jest poprawnym podgrafem $AG(e_{i+1}^{\,p})$; wszystkie RU $q$ komunikujące się z $p$ przesyłają z każdym komunikatem **tylko maksymalny indeks $j$** reprezentujący stan modułu $q$ we własnym grafie poprzedzania; gdy $p$ przesyła komunikat do $q$, będąc w stanie $e_i^{\,p}$, przesyła **tylko** $AG(e_i^{\,p}) - AG(e_j^{\,p})$,
- odbiorca odnotowuje nowe zdarzenie i **konstruuje nowy graf AG**, korzystając z poprzedniego i z informacji dostarczonych z komunikatem,
- **periodycznie** każdy RU zapisuje swój punkt kontrolny w pamięci nieulotnej — **bez żadnej koordynacji** z innymi modułami; zapisywane są też historia komunikatów i graf poprzedzania,
- graf poprzedzania jest zapisywany w pamięci trwałej **przed każdym wysłaniem komunikatu do świata zewnętrznego** (również bez koordynacji).

**Numer inkarnacji** (s. 53): z uwagi na niedeterministyczny czas przesyłania komunikatów RU $q$ może otrzymać komunikat od RU $p$ **po jego awarii i cofnięciu**, nie mogąc stwierdzić, czy komunikat pochodzi sprzed awarii, czy po cofnięciu. Wprowadza się więc **numer inkarnacji** przesyłany z każdym komunikatem; po awarii i cofnięciu moduł **zwiększa** numer inkarnacji i informuje o tym wszystkie inne moduły. **Komunikaty oznaczone nieaktualnym numerem inkarnacji są odrzucane.**

### Przebieg odtwarzania
<sub>s. 54</sub>

Odtwarzanie modułu $RU$ $p$ rozpoczyna się od odtworzenia z pamięci trwałej: stanu (z punktu kontrolnego), historii komunikatów, numeru inkarnacji oraz grafu poprzedzania $AG$. Numer inkarnacji $INCNUM$ jest **zwiększany o jeden i niezwłocznie zapisywany** w pamięci trwałej; w tablicy $INCVEC$ na pozycji $p$ zapisywany jest bieżący numer. Zmienna $G$ jest inicjowana odtworzonym grafem $AG$.

Następnie do wszystkich pozostałych modułów wysyłane jest żądanie przesłania ich numerów inkarnacji oraz grafów $AG(e_k^{\,p})$ takich, że $k$ jest **maksymalnym indeksem zdarzenia odebrania komunikatu od $p$** w grafie modułu $q$. Po otrzymaniu żądania każdy moduł $q$:
- zapisuje swój graf poprzedzania na stałym nośniku,
- szuka $k$, zapisuje je w tablicy $REJECTVEC$ na pozycji $p$ i wysyła żądane dane,
- **odrzuca komunikaty** zawierające graf $AG$, w których istnieje węzeł reprezentujący stan o indeksie **większym niż $k$**.

Graf $G$ jest sukcesywnie uzupełniany o przysłane grafy, a w $INCVEC$ zapisywane są przysłane numery inkarnacji. Kiedy wszystkie dane dotrą, $p$ rozsyła do wszystkich modułów tablicę $INCVEC$; po jej otrzymaniu $q$ uaktualnia własną tablicę, wybierając **większą** spośród odpowiadających sobie wartości, a następnie **unieważnia** liczbę na pozycji $p$ w $REJECTVEC$.

Moduł odtwarzany $p$ wykonuje następnie zdarzenia według grafu $G$, aby osiągnąć stan odpowiadający zdarzeniu o maksymalnym indeksie. Gdy odtwarzanym zdarzeniem jest **odebranie komunikatu**, $p$ żąda danych tego komunikatu **z historii komunikatów nadawcy**; w przeciwnym przypadku zdarzenie jest wykonywane od razu.

> Wszystkie komunikaty używane do komunikacji **w czasie odtwarzania nie są rejestrowane** tak jak komunikaty przetwarzania. Podczas odtwarzania wszystkie komunikaty związane z urządzeniami wejścia/wyjścia **nie są wysyłane**, ale zapisywane w pamięci ulotnej.

### Struktury danych
<sub>s. 54–55</sub>

| Struktura | Znaczenie |
|---|---|
| $N$ | liczba modułów programu rozproszonego |
| $c$ | indeks bieżącego stanu odtwarzanego modułu, odczytany z pamięci trwałej |
| $INCNUM$ | liczba inkarnacji |
| $INCVEC$ | tablica liczb inkarnacji wszystkich modułów (o wymiarze $N$) |
| $REJECTVEC$ | tablica liczb inkarnacji odrzucanych komunikatów dla każdego modułu (o wymiarze $N$) |
| $G$ | odtwarzany graf |
| $STATEINDEX$ | odtwarzany indeks stanu |

### Pseudokod algorytmu odtwarzania
<sub>s. 55–56</sub>

```
Initial state at all modules q:
    INCNUM        := 1;
    INCVEC[1..N]  := 1;
    REJECTVEC[1..N] := infinity;

At recovered module p:
    c := current state index;
    INCNUM := INCNUM + 1;
    save INCNUM on stable storage;
    INCVEC[p] := INCNUM;
    G := AG(e_c^p);

    for all modules q != p do
        send Get_AG(p);
        Upon receiving message Send_AG(INQ, AGQ)
            G := G union AGQ;
            INCVEC[q] := INQ;
    for all modules q != p do
        send Send_INC(p, INCVEC) message;

    m := max j such that e_j^p in G;
    STATEINDEX := c;
    while STATEINDEX <= m do
        execute up to next event without sending application messages;
        STATEINDEX := STATEINDEX + 1;
        if next event is a receive then
            request message from sender's log;
        else re-execute internal event;

At all modules q:
    Upon receiving message Get_AG(p)
        save AG on stable storage;
        k := max j such that e_j^p in G;
        REJECTVEC[p] := k;
        Send SendAG(INCNUM, AG(e_c^p)) message;

    Upon receiving message Send_INC(p, PINCVEC)
        for all i := 1..N do
            INCVEC[i] := max(INCVEC[i], PINCVEC[i]);
        REJECTVEC[p] := infinity;
```

---
## Zestawienie z wykładami

| Algorytm | Technika | Slajdy | Ten dokument |
|---|---|---|---|
| **Koo-Toueg** | checkpointing skoordynowany | `Slajdy-FT-02` 28–35 (fazy, optymalizacje) | s. 39–47 — **pełne bloki S1–S5, S7–S9** i mechanizm dyfuzji |
| **Wang-Fuchs** | checkpointing niezależny | `Slajdy-FT-03` 21–37 (z-dependency, krawędzie rollback, twierdzenia) | s. 48–50 — zwięzły opis słowny, **bez twierdzeń i bez garbage collection** |
| **Manetho** | checkpointing hybrydowy | `Slajdy-FT-04` 5–9 (model, AG, piggybacking) | s. 51–56 — **pełny pseudokod odtwarzania**, numer inkarnacji, `REJECTVEC` |

---
## Braki i uwagi

> [!todo] Czego nie ma w tym dokumencie
> - **Algorytmów Juanga-Venkatesana i Manivannana-Singhala** — są tylko na slajdach.
> - **Twierdzeń i dowodów** dla Wanga-Fuchsa (Theorem 1–3 ze slajdów wykładu II) oraz **garbage collection**.
> - **Analiz złożoności** żadnego z trzech algorytmów.
> - **Bloku S6** — numeracja bloków przeskakuje z S5 na S7.
