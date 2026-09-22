---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 23
---
# 23. Problemy rozproszonego uzgadniania w środowisku zawodnym
---
> Komplet pojęć zagadnienia. Algorytmy są nazwane i opatrzone informacją, jaki problem rozwiązują, ale ich przebieg pozostaje w [[SWN 05 Problemy uzgadniania i wyniki niemożliwości|SWN 05]]–[[SWN 08 Detektory awarii i replikacja procesu|SWN 08]].

## Model systemu

### System synchroniczny i rundy
**System synchroniczny** $\mathbb{S}^{Sync}$ opisuje się przez cztery założenia. Kanał między procesami może w dowolnej chwili przechowywać **co najwyżej jedną wiadomość**. **Funkcja tranzycji stanu** odwzorowuje deterministycznie bieżący stan procesu i wektor wiadomości przychodzących na stan nowy. **Funkcja generacji wiadomości** odwzorowuje stan procesu i jego sąsiadów na wiadomości do wysłania. Wykonanie całego systemu rozpoczyna się z procesami w dowolnych stanach początkowych i **wszystkimi kanałami pustymi**.

Procesy działają **krok w krok** (*lock-step*), powtarzając parę kroków nazywaną **rundą**. W pierwszym kroku każdy proces generuje zgodnie ze swoim stanem wiadomości do sąsiadów i umieszcza je w odpowiednich kanałach. W drugim stosuje funkcję tranzycji do swojego stanu i wiadomości przychodzących, przechodząc do stanu nowego, po czym kanały zostają opróżnione. Synchroniczność oznacza więc nie tyle wspólny zegar, ile **znane granice czasowe pozwalające podzielić wykonanie na rundy**.

### Rozszerzenia modelu
Zapis $\mathbb{S}\{M\}$ oznacza system $\mathbb{S}$ **wzbogacony o dodatkowy mechanizm** $M$. Notacja ta porządkuje całe zagadnienie, bo większość wyników brzmi „w tym modelu problem jest nierozwiązywalny, ale po dodaniu mechanizmu $M$ staje się rozwiązywalny". Najważniejsze warianty to $\mathbb{S}^{Sync}\{\varnothing\}$ — system synchroniczny bez dodatków, $\mathbb{S}^{Sync}\{C\}$ — z kryptograficznym podpisem cyfrowym, $\mathbb{S}^{Async}\{FD\}$ — system asynchroniczny z detektorem awarii, oraz $\mathbb{S}^{Async}\{\text{stable storage}\}$ i $\mathbb{S}^{Async}\{\text{Election}\}$, wykorzystywane przy omawianiu Paxosa.

### Rodzaje awarii
Rozważa się zarówno awarie procesów, jak i awarie łączy. **Awaria zatrzymania** (*stopping failure*) polega na tym, że proces przerywa wykonanie w dowolnym momencie — przed krokiem pierwszym lub drugim, po nim, a także **w środku kroku pierwszego**, umieszczając w kanałach jedynie **podzbiór** wiadomości, które miał wysłać. **Awaria bizantyjska** oznacza, że proces może wygenerować swój kolejny stan i kolejne wiadomości w **dowolny sposób**, niezależnie od funkcji tranzycji i funkcji generacji wiadomości. **Awaria łącza** polega na **gubieniu wiadomości**: proces próbuje umieścić wiadomość w kanale, ale uszkodzone łącze jej nie rejestruje.

### Przemilczenia
Ponieważ proces wadliwy może po prostu odmówić wysłania wiadomości, proces poprawny może nigdy nie doczekać się oczekiwanego komunikatu. Przyjmuje się wówczas, że odbiorca **wybiera dowolną wartość domyślną** i działa tak, jakby komunikat został odebrany. Wymaga to jednak, by **brak komunikatu dał się wykryć**, co w systemie synchronicznym jest proste: skoro czas trwania rundy jest znany, wszystkie komunikaty nieotrzymane do jej końca uznaje się za niewysłane. To założenie odróżnia później algorytm na komunikatach ustnych, gdzie brak komunikatu liczy się jako konkretna wartość, od algorytmu na komunikatach podpisanych, gdzie brak komunikatu jest po prostu ignorowany.

## Problemy uzgadniania

### Skoordynowany atak
**Skoordynowany atak** (*coordinated attack*) jest najprostszym problemem uzgadniania i punktem wyjścia dla pozostałych. Nieformalnie: kilku generałów planuje atak z różnych kierunków na wspólny cel, przy czym **jedyną drogą powodzenia jest zaatakowanie wszystkich naraz**; każdy ma początkową opinię, czy jego armia jest gotowa, a porozumiewają się wyłącznie przez posłańców, którzy mogą zostać zgubieni lub schwytani. Formalnie, przy oznaczeniach $1 = \text{atak}$ i $0 = \text{odwrót}$, wymaga się trzech własności. **Terminacja** (*liveness*) — wszystkie procesy w końcu decydują. **Zgodność** (*safety*) — żadne dwa procesy nie decydują o różnych wartościach. **Słaba ważność** (*Weak Validity*) — jeżeli wszystkie procesy startują z wartością 0, to jedyną możliwą decyzją jest 0; jeżeli wszystkie startują z wartością 1 **i wszystkie komunikaty zostaną dostarczone**, to jedyną możliwą decyzją jest 1.

Warunek ważności jest tu celowo osłabiony: jeśli choćby jeden proces startuje z jedynką, algorytm wolno zdecydować na 1, a jeśli wszystkie startują z jedynką, ale wszystkie komunikaty zostaną zgubione, wolno zdecydować na 0. Mimo tak słabego wymagania problem pozostaje nierozwiązywalny, o czym dalej.

### Konsensus
**Konsensus** jest bezpośrednim uogólnieniem skoordynowanego ataku — ten ostatni to w istocie konsensus binarny. Każdy proces rozgłasza swoją wartość początkową, przy czym wartości różnych procesów mogą się różnić. Wymaga się trzech własności: **terminacja** — każdy proces poprawny decyduje dokładnie jedną wartość; **zgodność** — wszystkie procesy poprawne decydują tę samą wartość; **ważność** — zdecydowana wartość została **zaproponowana przez pewien proces**. Nie ma znaczenia, którą konkretnie wartość procesy uzgodnią, dopóki nie łamie to ważności, ani na co zdecydują procesy wadliwe.

Odmianą wzmacniającą jest **jednolita zgodność** (*uniform agreement*), w której żadne dwa procesy — **także takie, które później ulegną awarii** — nie decydują różnie.

### Porozumienie bizantyjskie
W **porozumieniu bizantyjskim** (*Byzantine Agreement*, BA) występuje wyróżnione źródło: dowódca $P_s$ rozgłasza swoją wartość do pozostałych procesów, zwanych porucznikami. Wymaga się, by wszystkie procesy poprawne zdecydowały tę samą wartość (**zgodność**) oraz by — **jeżeli źródło jest poprawne** — była to wartość przez nie wysłana (**ważność**). Jeśli źródło jest wadliwe, procesy poprawne mogą uzgodnić dowolną wspólną wartość.

Istotna obserwacja: gdy dowódca jest lojalny, **zgodność wynika bezpośrednio z ważności**, bo wszyscy wykonują ten sam, autentyczny rozkaz. Cała trudność problemu leży więc w przypadku **dowódcy nielojalnego**.

### Spójność interaktywna
W **spójności interaktywnej** (*Interactive Consistency*, IC) każdy proces rozgłasza swoją wartość początkową, a procesy uzgadniają **cały wektor** $\langle v_1, v_2, \ldots, v_N \rangle$ wszystkich zaproponowanych wartości. Wymaga się, by wszystkie procesy poprawne uzgodniły **ten sam** wektor oraz by — jeśli proces $P_i$ jest poprawny — jego $i$-ta składowa była wartością przez niego zaproponowaną. Dla procesów wadliwych składowa może być dowolna, byle wspólna.

### Relacje między problemami
Wszystkie cztery problemy są ściśle powiązane i wzajemnie sprowadzalne. BA jest **szczególnym przypadkiem** IC, w którym interesuje nas wartość początkowa tylko jednego procesu. W drugą stronę, uruchomienie $N$ równoległych kopii protokołu BA — po jednej dla każdego procesu jako źródła — **rozwiązuje IC**. Mając rozwiązanie IC, można rozwiązać konsensus: procesy poprawne obliczają decyzję jako **wartość większościową** wspólnego wektora albo po prostu biorą jego **pierwszą składową**. Wreszcie mając konsensus, można rozwiązać BA w dwóch krokach — źródło wysyła swoją wartość do wszystkich procesów **łącznie z sobą samym**, po czym wszystkie uruchamiają algorytm konsensusu, traktując otrzymane wartości jako propozycje. Jeśli źródło jest poprawne, wszyscy dostaną tę samą wartość już w kroku pierwszym; jeśli wadliwe, mogą dostać różne, ale konsensus i tak je uzgodni.

Z tej wzajemnej sprowadzalności **nie wynika żaden porządek liniowy** między problemami: to, że jeden daje się wyrazić przez drugi, nie znaczy, że jest od niego słabszy.

### Dalsze warianty
Poza czterema podstawowymi problemami rozważa się warianty osłabiające wymagania na decyzję. **Konsensus $k$-zbiorowy** (*$k$-set consensus*) dopuszcza, by procesy uzgodniły **mały zbiór $k$ wartości** zamiast jednej. **Uzgodnienie przybliżone** (*approximate agreement*) wymaga jedynie, by zdecydowane wartości były **bliskie sobie nawzajem** — na przykład by wszyscy lojalni generałowie zaatakowali w odstępie nie większym niż dziesięć minut.

**Przemianowanie** (*renaming*) idzie w przeciwną stronę: wymaga, by wartości były **koniecznie różne**. Każdy proces otrzymuje nazwę $x_i$ z dziedziny $\mathbb{X}$, przy czym żądamy terminacji, przynależności nazwy do dziedziny, **różności nazw** procesów poprawnych oraz **anonimowości** — kod wykonywany przez proces **nie może zależeć od jego początkowego identyfikatora**. Problem przydaje się przy transformacji przestrzeni nazw, gdy procesy z różnych dziedzin muszą przypisać sobie różne nazwy z małej dziedziny, albo gdy identyfikatory mają służyć jako etykiety porządkujące.

**Elekcja** polega na wyborze jednego wyróżnionego procesu. W modelu procesów początkowo martwych jest trywialna i co więcej **każdy algorytm elekcji wybierający proces poprawny rozwiązuje zarazem konsensus** — wybrany lider rozgłasza swoją wartość początkową, a wszystkie procesy poprawne na nią decydują. W modelu fail-stop ta zależność zawodzi, bo lider może paść **przed** rozgłoszeniem wartości; elekcja nie jest zresztą rozwiązywalna przy awariach typu crash.

### Zadanie rozproszone
Wszystkie powyższe problemy uogólnia pojęcie **zadania rozproszonego**, opisanego zbiorami możliwych wartości wejściowych i wyjściowych oraz — być może częściową — funkcją $\mathcal{T}: \mathbb{In}^{N} \rightarrow \mathbb{Out}^{N}$. Jeśli wektor $I$ opisuje wejścia procesów, to $\mathcal{T}(I)$ jest **zbiorem legalnych wektorów decyzji**; częściowość funkcji oznacza, że nie każda kombinacja wejść jest dozwolona. Konsensus wyraża się w tym języku jako zadanie, w którym wszystkie decyzje muszą być równe, a elekcja — jako zadanie, w którym dokładnie jeden proces decyduje 1, a pozostałe 0.

Algorytm nazywa się **$f$-odpornym rozwiązaniem** zadania, jeżeli w każdym $f$-crash *fair execution* wszystkie procesy poprawne decydują (**terminacja**) oraz — gdy wszystkie procesy są poprawne — wektor decyzji należy do $\mathcal{T}(I)$ (**spójność**). Pokrewnym pojęciem jest **$f$-initially-dead fair execution**: wykonanie, w którym co najmniej $N-f$ procesów jest aktywnych, każdy aktywny proces jest poprawny, a każda wiadomość wysłana do procesu poprawnego zostaje dostarczona.

## Wyniki niemożliwości

### Niemożliwość przy awariach łączy
Dla grafu złożonego z **dwóch węzłów połączonych zawodną krawędzią nie istnieje algorytm rozwiązujący skoordynowany atak** — i to **nawet w modelu synchronicznym**. Przypadek dwóch węzłów implikuje niemożliwość dla grafów większych.

Dowód prowadzi się nie wprost, przez ciąg wykonań **nieodróżnialnych** dla poszczególnych procesów. Zaczyna się od wykonania, w którym oba procesy startują z jedynką i wszystkie wiadomości są dostarczane; z terminacji i ważności wynika, że oba decydują na 1. Następnie konstruuje się kolejne wykonania, w każdym gubiąc jedną wiadomość więcej. Każde z nich jest nieodróżnialne od poprzedniego dla tego procesu, którego dana wiadomość nie dotyczyła, więc ten proces decyduje tak samo jak wcześniej, a ze zgodności wynika, że drugi też. Powtarzając to rozumowanie, dochodzi się do wykonania, w którym **oba procesy startują z zerem, a mimo to decydują na 1** — co narusza ważność.

### Twierdzenie FLP
**Twierdzenie FLP'85** jest najważniejszym wynikiem całego zagadnienia: **nie istnieje deterministyczne rozwiązanie konsensusu w systemie asynchronicznym, jeżeli choćby jeden proces może ulec awarii typu crash**. Przyczyna jest prosta do wyrażenia — w systemie asynchronicznym **nie da się odróżnić procesu, który uległ awarii, od procesu bardzo wolnego lub odległego**, ponieważ nie ma żadnych granic czasowych, względem których można by orzec, że odpowiedź już nie nadejdzie. W systemie synchronicznym taka granica istnieje, co daje **doskonałą detekcję awarii**, a doskonała detekcja awarii czyni konsensus rozwiązywalnym. Asynchroniczność oznacza brak detekcji awarii, a brak detekcji — nierozwiązywalność konsensusu.

Istotne jest, że twierdzenie opiera się na **trzech warunkach jednocześnie**: rozwiązanie musi być deterministyczne, system asynchroniczny, a przynajmniej jeden proces podatny na crash. Osłabienie któregokolwiek z nich otwiera drogę do rozwiązania i właśnie stąd biorą się wszystkie obejścia opisane dalej.

### Niemożliwość porozumienia bizantyjskiego
**Nie istnieje $f$-odporny algorytm porozumienia bizantyjskiego w $\mathbb{S}^{Sync}$ dla $f \geqslant \frac{1}{3}N$**, czyli rozwiązanie wymaga więcej niż $3f$ generałów przy co najwyżej $f$ zdrajcach.

Podstawą dowodu jest przypadek **trzech generałów z jednym zdrajcą**. W pierwszym wariancie zdrajcą jest dowódca: wysyła jednemu porucznikowi rozkaz ataku, a drugiemu odwrotu; lojalny porucznik przekazuje dalej to, co otrzymał. W drugim wariancie dowódca jest lojalny i wysyła obu ten sam rozkaz, ale zdrajcą jest jeden z poruczników i przekazuje rozkaz przeciwny. **Dla lojalnego porucznika oba warianty są nieodróżnialne** — widzi dokładnie te same komunikaty — a jednak w pierwszym musi zignorować rozkaz dowódcy, a w drugim go wykonać. Nie da się więc spełnić zgodności i ważności naraz. Uogólnienie na $3f$ generałów przeprowadza się **przez symulację**: gdyby istniał algorytm dla $3f$ generałów z $f$ zdrajcami, każdy z trzech generałów mógłby symulować $f$ generałów, jeden zdrajca symulowałby $f$ zdrajców, a dwaj lojalni — $2f$ lojalnych, co dałoby rozwiązanie dla przypadku bazowego.

### Niemożliwość uzgodnienia przybliżonego
Naturalne pytanie brzmi, czy trudność nie bierze się z żądania **dokładnego** uzgodnienia. Odpowiedź jest przecząca: **dla uzgodnienia przybliżonego obowiązuje ten sam próg $f \geqslant \frac{1}{3}N$**. Dowodzi się tego, **przekształcając rozwiązanie AA w rozwiązanie BA**. Dowódca wysyła czas ataku, przy czym godzina 1:00 koduje atak, a 2:00 odwrót. W pierwszej fazie porucznicy uruchamiają protokół uzgodnienia przybliżonego i jeśli uzgodniony czas wypada przed 1:10, decydują atak, a jeśli po 1:50 — odwrót. W drugiej fazie ci, którzy nie podjęli decyzji, pytają drugiego porucznika, czy on zdecydował; jeśli tak, robią to samo, a jeśli nie — wycofują się. Skoro BA jest niemożliwe, niemożliwe jest i AA. Wniosek ma charakter ogólny: **samo osłabienie dokładności uzgodnienia nie ułatwia problemu**.

### Rozwiązania probabilistyczne
Osobną rodziną są algorytmy losowe, dzielone według tego, z czego rezygnują. Algorytm jest **Monte Carlo**, jeżeli **zawsze się kończy**, a prawdopodobieństwo uzyskania poprawnej konfiguracji końcowej jest większe od zera — poświęca więc poprawność. Algorytm jest **Las Vegas**, jeżeli kończy się z prawdopodobieństwem większym od zera, ale **wszystkie konfiguracje końcowe są poprawne** — poświęca więc terminację.

Dla obu rodzin obowiązują ograniczenia. **Nie istnieje 1-odporne fail-stop rozwiązanie Monte Carlo konsensusu.** **Nie istnieje $f$-odporne fail-stop rozwiązanie Las Vegas konsensusu dla $f \geqslant \frac{N}{2}$**, natomiast dla $f < \frac{N}{2}$ takie rozwiązanie **istnieje**. Oddzielnie dowodzi się też, że **nie istnieje 1-odporne fail-stop rozwiązanie konsensusu, które zawsze się kończy** — co jest formalnym uzasadnieniem konstrukcji Paxosa.

## Obejścia twierdzenia FLP

Ponieważ twierdzenie FLP opiera się na trzech warunkach naraz, każdy z nich wyznacza osobną drogę wyjścia. Mimo wyników niemożliwości wiele nietrywialnych problemów ma więc rozwiązania również w systemach asynchronicznych z awariami.

Pierwszą drogą jest **silniejsza synchronia** — przyjęcie modelu synchronicznego lub quasi-synchronicznego, w którym awarię da się wykryć przekroczeniem czasu oczekiwania. Drugą jest **słabszy model awarii**, w szczególności model **procesów początkowo martwych**. Trzecią jest **poświęcenie żywotności na rzecz bezpieczeństwa**, czyli osłabienie warunku terminacji do postaci „każdy proces poprawny w końcu decyduje **z prawdopodobieństwem 1**", co realizuje randomizacja, albo wręcz rezygnacja z terminacji, co realizuje Paxos. Czwartą, omówioną osobno na końcu, jest **rozszerzenie modelu o detektor awarii**, czyli przejście do $\mathbb{S}^{Async}\{FD\}$.

### Model procesów początkowo martwych
W **modelu procesów początkowo martwych** przyjmuje się, że **żaden proces nie może ulec awarii po wykonaniu jakiegokolwiek zdarzenia** — proces albo jest martwy od początku, albo pozostanie poprawny. Model ten jest słabszy od fail-stop i w nim konsensus oraz elekcja są osiągalne **deterministycznie**, dopóki $f < \frac{N}{2}$.

Kluczowa obserwacja brzmi: skoro procesy nie padają po wysłaniu wiadomości, to dla procesu $P_i$ jest **bezpieczne czekać** na wiadomość od $P_j$, o ile wie, że $P_j$ już cokolwiek wysłał. Proces początkowo martwy nie wysłał nic, więc w grafie relacji „odebrałem wiadomość od" tworzy **węzeł izolowany**, podczas gdy każdy proces poprawny ma odpowiednio wiele sąsiadów. Pozwala to wyodrębnić w tym grafie **jednoznaczny węzeł** (*knot*) złożony z procesów poprawnych i oprzeć na nim decyzję.

Problem ten rozwiązuje **[[SWN 05 Problemy uzgadniania i wyniki niemożliwości#Algorytm Fischera-Lyncha-Patersona|algorytm Fischera-Lyncha-Patersona]]**, który w dwóch fazach buduje graf, wyznacza w nim jednoznaczny knot procesów poprawnych i pozwala wszystkim procesom zdecydować **wyłącznie na podstawie propozycji pochodzących z tego knota**, zużywając $O(N^2)$ wiadomości.

## Komunikaty ustne i podpisane

### Dlaczego typ komunikatu zmienia wszystko
Aby osiągnąć porozumienie, procesy muszą wymieniać wartości i wielokrotnie przekazywać dalej to, co otrzymały od innych. Zdolność procesu wadliwego do **zniekształcania tego, co przekazuje**, zależy zaś od typu komunikatu i to właśnie ona przesądza o granicach możliwości.

Przy **komunikatach ustnych** (*oral*, nieuwierzytelnionych) proces wadliwy może **sfałszować** komunikat, twierdząc, że otrzymał go od kogoś innego, albo **zmienić zawartość** otrzymanego komunikatu przed przekazaniem dalej. Odbiorca nie ma żadnego sposobu, by zweryfikować autentyczność. Przy **komunikatach podpisanych** (*signed*, uwierzytelnionych) proces wadliwy **nie może sfałszować** komunikatu ani zmienić jego treści, a każdy może **zweryfikować autentyczność** — procesy wadliwe wyrządzają więc znacznie mniej szkody.

### Założenia o sieci
Dla komunikatów ustnych przyjmuje się trzy założenia: **A1** — każdy wysłany komunikat jest dostarczany poprawnie; **A2** — odbiorca komunikatu wie, kto go wysłał; **A3** — brak komunikatu może zostać wykryty. Założenia A1 i A2 uniemożliwiają zdrajcy zakłócanie komunikacji między innymi i wykluczają fałszywe komunikaty, a A3 sprawia, że zdrajca nie może zablokować postępu, po prostu milcząc. Dla komunikatów podpisanych dochodzi **A4** — podpis cyfrowy nie może zostać sfałszowany, każda zmiana komunikatu jest wykrywalna, a autentyczność podpisu może zweryfikować każdy.

### Rozstrzyganie wartości
Przy komunikatach ustnych decyzję podejmuje funkcja większościowa: $majority(v_1, \ldots, v_n)$ zwraca $v$, jeśli więcej niż $\frac{n}{2}$ wartości jest równych $v$, a w przeciwnym razie wartość domyślną 0. Ponieważ obowiązuje założenie A3, **brakujący komunikat traktuje się jako wartość 0**.

Przy komunikatach podpisanych decyduje funkcja $\texttt{choice}(V_i)$ działająca na zbiorze $V_i$ **ważnych**, czyli poprawnie podpisanych rozkazów: zwraca $v$, gdy $V_i = \{v\}$, a w przeciwnym razie 0, czyli odwrót. Zasadnicza różnica polega na tym, że tutaj **brakującego komunikatu nie wolno interpretować jako zera** — liczą się wyłącznie rozkazy faktycznie podpisane. Jeśli proces otrzyma kilka ważnych komunikatów o różnych wartościach, wie z tego samego faktu, że **pochodzą od dowódcy i że dowódca jest wadliwy**.

### Algorytmy i ich porównanie
Problem porozumienia bizantyjskiego na komunikatach ustnych rozwiązuje **[[SWN 06 Awarie bizantyjskie#BA z komunikatami ustnymi — algorytm OM|algorytm OM]]** Lamporta, Shostaka i Pease'a, działający rekurencyjnie i radzący sobie z $f$ zdrajcami, dopóki $N > 3f$. Ten sam problem przy komunikatach podpisanych rozwiązuje **[[SWN 06 Awarie bizantyjskie#BA z komunikatami podpisanymi — algorytm SM|algorytm SM]]**, którego istotą jest to, że podpis odbiera zdrajcy zdolność fałszowania cudzych komunikatów — wystarczy więc, by **jeden podpis w łańcuchu pochodził od procesu poprawnego**. Dzięki temu ograniczenie $N > 3f$ **znika**, a złożoność spada wykładniczo.

| | komunikaty ustne (OM) | komunikaty podpisane (SM) |
|---|---|---|
| model | $\mathbb{S}^{Sync}\{\varnothing\}$ | $\mathbb{S}^{Sync}\{C\}$ |
| warunek na $f$ | $f < \frac{1}{3}N$ | formalnie $f < N$; sensownie $N \geqslant f+2$ |
| liczba rund | $f+1$ | $f+1$ |
| liczba komunikatów | $\Omega(N^{f+1})$ | $O(N^2)$, po optymalizacji $\leqslant (N-1) + 2(N-1)^2$ |
| brakujący komunikat | traktowany jako 0 | ignorowany |
| decyzja | $majority(v_1, \ldots, v_{n-1})$ | $\texttt{choice}(V_i)$ |

Liczba **$f+1$ rund jest dolnym ograniczeniem** dla porozumienia bizantyjskiego w sieci w pełni połączonej — obowiązuje więc niezależnie od typu komunikatów.

## Algorytmy konsensusu

Trzy algorytmy odpowiadają trzem różnym obejściom twierdzenia FLP.

**[[SWN 07 Konsensus#Paxos|Paxos]]** rozwiązuje konsensus w systemie asynchronicznym przy awariach fail-stop i $f < \frac{N}{2}$, **rezygnując z warunku terminacji**. Opiera się na zasadzie, że propozycja zostaje wybrana, gdy zaakceptuje ją większość procesów, przy czym każda propozycja ma unikalny numer, wygrywa propozycja o numerze najwyższym, a raz zaakceptowana propozycja taką pozostaje. Bez dodatkowej elekcji algorytm może wpaść w zakleszczenie żywotne, gdy zwycięzca pada tuż przed zgłoszeniem propozycji albo gdy procesy w nieskończoność przebijają się nawzajem coraz wyższymi numerami. Opis również w notatce [[Systemy Wysokiej Niezawodności/Algorytm Paxos|Algorytm Paxos]].

**[[SWN 07 Konsensus#Algorytm Bracha-Touega — model i idea|Algorytm Brachy-Touega]]** rozwiązuje konsensus **binarny** w tym samym modelu i przy tym samym ograniczeniu $f < \frac{N}{2}$, ale drogą **randomizacji**: jest algorytmem typu Las Vegas, który **kończy się z prawdopodobieństwem 1**. Procesy losują wartość początkową i w kolejnych rundach aktualizują ją wraz z wagą przybliżającą liczbę głosów oddanych na tę wartość w rundzie poprzedniej. Opis również w notatce [[Systemy Wysokiej Niezawodności/Algorytm Bracha-Touega|Algorytm Bracha-Touega]].

**[[SWN 07 Konsensus#Konsensus procesów bizantyjskich — algorytm Phase-King|Algorytm Phase-King]]** Bermana i Garaya rozwiązuje konsensus **procesów bizantyjskich** w systemie synchronicznym, opierając się na **rotującym koordynatorze**: w każdej z $f+1$ faz innym liderem jest inny proces, a jego wartość większościowa służy jako głos rozstrzygający dla tych procesów, które nie zebrały wyraźnej większości. Ceną za **wielomianową** liczbę komunikatów jest ostrzejszy warunek $f < \frac{N}{4}$, podczas gdy algorytm OM osiąga optymalne $N > 3f$ kosztem złożoności wykładniczej. Opis również w notatce [[Systemy Wysokiej Niezawodności/Algorytm Phase-King|Algorytm Phase-King]].

> Algorytm Brachy-Touega omawiany w tym kursie to algorytm **konsensusu**, a nie detekcji zakleszczenia. Algorytm detekcji zakleszczenia tych samych autorów w prezentacjach nie występuje.

## Replikacja procesu

Replikacja jest praktycznym zastosowaniem konsensusu i właśnie ona pokazuje, dlaczego wyniki niemożliwości mają znaczenie inżynierskie.

### Replikacja aktywna
**Replikacja aktywna**, nazywana też **replikacją maszyny stanów**, polega na tym, że żądanie klienta trafia do **wszystkich replik**, a każda je przetwarza i odpowiada. Klient może wybrać strategię zbierania odpowiedzi — pierwszą, która nadejdzie, wszystkie istniejące albo ustaloną liczbę $R < n$ — i w zależności od $R$ może nawet tolerować **awarie bizantyjskie**, na przykład przy $R > 3f$.

Kluczowym wymaganiem spójności jest, by **żądania były całkowicie uporządkowane**, przy czym **konkretna relacja porządkująca nie ma znaczenia** — istotne jest wyłącznie, aby wszystkie repliki przetwarzały żądania **w tej samej kolejności**. Stąd bierze się zapotrzebowanie na rozgłaszanie totalne.

Replikacja aktywna **nie dopuszcza przetwarzania niedeterministycznego**, ponieważ mogłoby ono dać różne odpowiedzi na to samo żądanie. Co gorsza, takiego niedeterminizmu **nie da się odróżnić od awarii bizantyjskiej**, a głosowanie ani porozumienie bizantyjskie go nie ratują, bo każda odpowiedź może być inna. Nawet gdyby klient arbitralnie wybrał jedną odpowiedź, **stany poszczególnych replik zdążyły już się rozejść**. Zaletami są to, że klient nigdy nie musi ponawiać żądania oraz dopuszczalność awarii bizantyjskich; wadami — ograniczenie do operacji deterministycznych, większe zużycie zasobów i **brak skalowalności**, bo zwiększanie liczby replik nie podnosi przepustowości.

### Replikacja pasywna
**Replikacja pasywna**, czyli **primary-backup**, polega na tym, że żądanie obsługuje wyłącznie **replika główna**, która następnie rozsyła uaktualnienia do kopii zapasowych. To ona **narzuca całkowity porządek uaktualnień**, więc porządek nie musi być uzgadniany między replikami.

Ponieważ żądania przetwarza tylko jedna replika, **przetwarzanie niedeterministyczne jest dozwolone** — uaktualnienia i tak zapewniają globalną spójność grupy. Niezawodność zależy jednak od repliki głównej, a jej awaria pociąga cztery konsekwencje: operacja uaktualnienia musi być **atomowa**, trzeba **wybrać nową replikę główną**, klient może zaobserwować **opóźnienie** i może być zmuszony **ponowić żądanie**, co z kolei wymaga, by każde żądanie było **jednoznacznie identyfikowane**. Dodatkowo, przy wielu klientach, żądania muszą być **zsynchronizowane ze zmianami członkostwa** w grupie. Zaletami są niskie zużycie zasobów i dopuszczalność niedeterminizmu; wadami — brak zastosowań czasu rzeczywistego wobec możliwych dużych opóźnień oraz **brak tolerancji awarii bizantyjskich**, bo klient nie może otrzymać niepoprawnej odpowiedzi.

### Wywołania zagnieżdżone
Swoboda niedeterminizmu w replikacji pasywnej ma jeden wyjątek: **wywołania zagnieżdżone** (*nested invocations*). Scenariusz wygląda tak, że replika główna na podstawie decyzji niedeterministycznej wywołuje jeden serwer zaplecza, otrzymuje odpowiedź i **dopiero potem pada**. Nowa replika główna, podejmując **własną** decyzję niedeterministyczną, wywołuje **inny** serwer. W efekcie **oba serwery zaplecza zostały uaktualnione**, choć miał zostać wywołany tylko jeden — stan globalny jest niespójny.

## Komunikacja grupowa

Każdy typ replikacji wymaga innego rodzaju rozgłaszania: aktywna potrzebuje **rozgłaszania całkowicie uporządkowanego**, pasywna — **rozgłaszania synchronicznego względem widoków**. **Widokiem** grupy nazywa się przy tym jej skład członkowski, a każda jego zmiana, wywołana awarią lub odtworzeniem procesu, tworzy **nowy widok**.

### Rozgłaszanie niezawodne
**RBcast** (*Reliable Broadcast*) określają trzy własności. **Terminacja** — jeśli proces poprawny rozgłasza wiadomość, to w końcu sam ją dostarcza. **Zgodność** — jeśli **jakikolwiek** proces poprawny dostarcza wiadomość, to dostarczają ją **wszystkie** procesy poprawne. **Ważność** — wiadomość została rozgłoszona przez pewien proces i jest dostarczana **co najwyżej raz**. Realizuje to **[[SWN 08 Detektory awarii i replikacja procesu#Reliable Broadcast przez dyfuzję komunikatów|rozgłaszanie przez dyfuzję komunikatów]]**, rozwiązujące problem niezawodnego rozgłaszania w $\mathbb{S}^{Async}\{\varnothing\}$ przez to, że każdy odbiorca przy **pierwszym** odebraniu danej wiadomości retransmituje ją do wszystkich.

### Rozgłaszanie jednolite
W RBcast, jeśli nadawca uległ awarii, wiadomość jest dostarczana przez wszystkie procesy poprawne **albo przez żaden**. **UBcast** (*Uniform Broadcast*) wzmacnia warunek zgodności, obejmując nim także procesy wadliwe: jeśli **jakikolwiek proces — poprawny lub nie** — dostarczy wiadomość, to **wszystkie procesy poprawne w końcu ją dostarczą**. Różnica jest zasadnicza: RBcast wiąże tylko procesy poprawne, więc proces, który dostarczył wiadomość i natychmiast padł, nie zobowiązuje nikogo; w UBcast dostarczenie przez kogokolwiek jest już zobowiązaniem dla całej grupy.

### Rozgłaszanie totalne i synchroniczne względem widoków
**TOcast** (*totally ordered multicast*), nazywane też **rozgłaszaniem atomowym** (ABcast), to **RBcast uzupełniony o całkowity porządek dostarczania**. Wymaga go replikacja aktywna.

**VScast** (*view synchronous multicast*) zapewnia **spójny zbiór wiadomości dostarczanych mimo zmian członkostwa**. Dla dwóch kolejnych widoków $v_i$ i $v_{i+1}$ wymaga się, by wszystkie procesy poprawne należące do $v_i \cap v_{i+1}$ w końcu dostarczyły wiadomość (**zgodność**) oraz by wiadomość wysłana w widoku $v_i$ została dostarczona w każdym takim procesie **przed jakąkolwiek wiadomością z widoku $v_{i+1}$** (**ważność**). Wymaga go replikacja pasywna.

Wszystkie rodzaje rozgłaszania układają się w **trójwymiarową siatkę**: z RBcast przechodzi się do TOcast przez nałożenie **porządku totalnego**, w dół przez nałożenie porządku **FIFO** i dalej **przyczynowego**, a w trzecim wymiarze — do wariantów **jednolitych**.

### Równoważność z konsensusem
Centralnym wynikiem jest **$\text{TOcast} \cong C \cong \text{VScast}$**: mając rozwiązanie rozgłaszania totalnego, umiemy rozwiązać konsensus, a mając konsensus — umiemy zrealizować rozgłaszanie totalne. Pierwsza implikacja jest trywialna, bo wystarczy, by każdy proces zdecydował **pierwszą wartość dostarczoną w porządku totalnym**. Drugą realizuje **[[SWN 08 Detektory awarii i replikacja procesu#Rozwiązanie TOcast przy użyciu konsensusu|rozwiązanie TOcast przy użyciu konsensusu]]**, które problem zbudowania porządku totalnego sprowadza do **okresowego konsensusu nad podzbiorami wiadomości jeszcze niedostarczonych**, uzupełnionego o ustalony z góry deterministyczny porządek wewnątrz podzbioru. Analogicznie konsensus pozwala zrealizować VScast, przy czym uzgadnia się wtedy dodatkowo **skład następnego widoku**.

Konsekwencja tej równoważności jest dotkliwa: skoro z twierdzenia FLP konsensus jest w $\mathbb{S}^{Async}\{\varnothing\}$ nierozwiązywalny, to **w tym modelu nie da się zaimplementować ani TOcast, ani VScast, a więc ani replikacji aktywnej, ani pasywnej**.

## Detektory awarii

### Czym jest detektor awarii
Skoro przyczyną niemożliwości jest brak zdolności odróżnienia procesu martwego od wolnego, naturalnym wyjściem jest **dodanie takiej zdolności do modelu**. **Rozproszony detektor uszkodzeń** to zbiór modułów-wyroczni, z których **każda przyłączona jest do jednego procesu**, a jej zadaniem jest dostarczanie **listy procesów podejrzewanych o uszkodzenie**. Wyrocznie **mogą być omylne** — mogą podejrzewać proces poprawny albo nie podejrzewać uszkodzonego — ale obowiązuje zasada, że **błędne podejrzenia nie mogą powstrzymywać procesów poprawnych** przed zachowaniem zgodnym z ich specyfikacją.

### Zupełność i dokładność
Detektory klasyfikuje się według dwóch niezależnych rodzin własności. **Zupełność** (*completeness*) mówi o tym, czy awarie są wykrywane; **dokładność** (*accuracy*) — czy nie ma fałszywych podejrzeń.

W wariancie **silnej zupełności (SC)** w końcu **każdy** proces uszkodzony jest podejrzewany przez **każdy** proces poprawny. W wariancie **słabej zupełności (WC)** każdy uszkodzony jest podejrzewany przez **pewien** proces poprawny, przy czym każdy uszkodzony może być podejrzewany przez **inny** proces.

Dokładność ma cztery warianty. **Silna dokładność (SA)** — **żaden** proces poprawny **nigdy** nie jest podejrzewany. **Słaba dokładność (WA)** — **pewien** proces poprawny nigdy nie jest podejrzewany. **Ostateczna dokładność (EA)** — żaden proces poprawny **w końcu** nie jest podejrzewany. **Ostateczna słaba dokładność (EWA)** — pewien proces poprawny w końcu nie jest podejrzewany, przy czym musi to być **ten sam** proces dla wszystkich.

### Osiem klas detektorów
Kombinacje obu rodzin dają osiem klas, układających się w sześcian:

| | SA | WA | EA | EWA |
|---|---|---|---|---|
| **SC** | $\mathcal{P}$ (*perfect*) | $\mathcal{S}$ (*strong*) | $\Diamond\mathcal{P}$ (*eventually perfect*) | $\Diamond\mathcal{S}$ (*eventually strong*) |
| **WC** | $\mathcal{Q}$ | $\mathcal{W}$ (*weak*) | $\Diamond\mathcal{Q}$ | $\Diamond\mathcal{W}$ (*eventually weak*) |

### Dokładność ograniczona do podzbioru
Własności dokładności **nie da się osiągnąć przy podziale sieci** na rozłączne fragmenty, bo procesy po przeciwnych stronach podziału zawsze będą się nawzajem podejrzewać. Wprowadza się więc **$\Gamma$-dokładność**, dotyczącą jedynie procesów należących do pewnego podzbioru $\Gamma$. W wariancie **silnej $\Gamma$-dokładności** żaden poprawny proces ze zbioru $\Gamma$ nie jest podejrzewany przez inny proces z tego zbioru; w wariancie **słabej** pewien poprawny proces — niekoniecznie ze zbioru $\Gamma$ — nie jest podejrzewany przez żaden proces z $\Gamma$. Zachodzi przy tym monotoniczność: jeśli $\Gamma_1 \subset \Gamma_2$ i $\Gamma$-dokładność zachodzi w $\Gamma_2$, to zachodzi też w $\Gamma_1$.

### Redukcja i równoważność
Dwa detektory porównuje się za pomocą **algorytmu transformacji** $T_{\mathcal{D} \rightarrow \mathcal{D}'}$. Zapis $\mathcal{D} \succeq \mathcal{D}'$ czyta się „$\mathcal{D}$ emuluje $\mathcal{D}'$", równoważnie „$\mathcal{D}'$ jest słabszy niż $\mathcal{D}$". Jeśli redukcja zachodzi w obie strony, detektory są **równoważne**, co zapisuje się $\mathcal{D} \cong \mathcal{D}'$.

Że silna zupełność emuluje słabą, jest oczywiste — wystarczy pusta transformacja. Mniej oczywiste jest, że zachodzi również implikacja odwrotna, co pokazuje **[[SWN 08 Detektory awarii i replikacja procesu#Redukcja i równoważność|transformacja z WC na SC]]**, rozwiązująca problem wzmocnienia zupełności bez zmiany dokładności: każdy kontroler cyklicznie odpytuje swój moduł detektora i **rozgłasza listę podejrzeń**, a odbierając cudze listy, **sumuje je**, usuwając z wyniku nadawcę. Stąd $SC \cong WC$, a w konsekwencji $\mathcal{P} \cong \mathcal{Q}$, $\mathcal{S} \cong \mathcal{W}$, $\Diamond\mathcal{P} \cong \Diamond\mathcal{Q}$ i $\Diamond\mathcal{S} \cong \Diamond\mathcal{W}$.

Ponadto zachodzą ostre nierówności $\mathcal{P} \succ \mathcal{S}$, $\mathcal{P} \succ \Diamond\mathcal{P}$ oraz $\mathcal{S} \succ \Diamond\mathcal{S}$, natomiast **klasy leżące po przekątnej sześcianu są nieporównywalne** — dotyczy to par $\Diamond\mathcal{P}$ i $\mathcal{S}$, $\Diamond\mathcal{Q}$ i $\mathcal{W}$, $\Diamond\mathcal{P}$ i $\mathcal{W}$ oraz $\Diamond\mathcal{Q}$ i $\mathcal{S}$. W wariancie z dokładnością ograniczoną do podzbioru zachodzi $\Diamond\mathcal{S}(\Gamma) \cong \Diamond\mathcal{S}$, ale już $\Diamond\mathcal{W} \succ \Diamond\mathcal{W}(\Gamma)$ i $\mathcal{P} \succ \mathcal{P}(\Gamma)$.

### Konsensus z detektorem awarii
Konsensus rozwiązuje się odmiennie w zależności od siły dostępnego detektora. **[[SWN 07 Konsensus#Rozwiązanie konsensusu z użyciem dowolnego $\mathcal{D} \in \mathcal{S}$|Rozwiązanie dla detektora $\mathcal{S}$]]** wykorzystuje to, że silna dokładność w wariancie słabym gwarantuje istnienie **co najmniej jednego procesu nigdy nie podejrzewanego** — swoistego autorytetu. Algorytm przebiega w $N$ rundach: $N-1$ rund wymiany propozycji, rozgłoszenie ostatecznego zbioru propozycji, wyliczenie części wspólnej, o której de facto rozstrzyga autorytet, i decyzja na pierwszą pozostałą propozycję. Działa przy $f < N$ i wymaga kanałów ostatecznie niezawodnych.

**[[SWN 07 Konsensus#Rozwiązanie konsensusu z użyciem dowolnego $\mathcal{D} \in \Diamond\mathcal{S}$|Rozwiązanie dla detektora $\Diamond\mathcal{S}$]]** mierzy się z trudniejszą sytuacją, w której **na początku każdy proces może być podejrzewany**, więc żadnego autorytetu nie ma. Liczba rund nie jest tu ograniczona — algorytm działa do skutku — a jego istotą jest **rotujący koordynator**: kolejne procesy stają się koordynatorami rund, koordynator czeka na zebranie kworum większościowego, a decyzja rozgłaszana jest przez RBcast. Warunek to $f < \left\lceil \frac{N}{2} \right\rceil$, ponieważ kworum wynosi $\left\lfloor \frac{N}{2} \right\rfloor + 1$.

Dodatkowo obowiązuje wynik wzmacniający: **każdy protokół rozwiązujący konsensus przy użyciu $\Diamond\mathcal{S}$ rozwiązuje zarazem konsensus jednolity**.

### Rozgłaszanie niezawodne z terminacją
**TRBcast** (*Terminating Reliable Broadcast*) przypomina RBcast, ale wymaga, by **każdy proces poprawny zawsze dostarczył dokładnie jeden komunikat** — nawet wtedy, gdy wyróżniony nadawca jest wadliwy i padł przed rozgłoszeniem. W takim przypadku procesy dostarczają **specjalny komunikat $m_F$**, w istocie pustą wartość, która nie została faktycznie rozgłoszona i której dostarczenie **dowodzi awarii nadawcy**. Problem ten jest **równoważny porozumieniu bizantyjskiemu**.

Kluczowa różnica wobec RBcast polega na tym, że TRBcast **wymaga ostatecznego dostarczenia jakiegoś komunikatu**, a więc wymaga **rozpoznania awarii** — w przeciwieństwie do sytuacji, w której po prostu nic nie zostaje wysłane. Jest to zatem równoważne zdolności **odróżnienia procesu wolnego od procesu, który uległ awarii**, i stąd bierze się jego wysokie zapotrzebowanie na siłę detektora: TRBcast **da się rozwiązać przy dowolnej liczbie awarii typu crash, używając $\mathcal{P}$**, ale **nie da się go rozwiązać przy użyciu $\Diamond\mathcal{P}$, $\mathcal{S}$ ani $\Diamond\mathcal{S}$, nawet przy założeniu co najwyżej jednej awarii**. Detektor $\mathcal{P}$ jest przy tym **najsłabszym** wystarczającym do rozwiązania powtarzanych instancji TRBcast.

### Detektory w praktyce
Detektory awarii mają zalety, dla których wprowadza się je jako rozszerzenie modelu: są **naturalne**, bo stanowią autentyczne rozszerzenie modelu asynchronicznego; **minimalne**, bo dostarczają najmniejszej dodatkowej informacji wystarczającej do rozwiązania konsensusu; **proste** w zrozumieniu i użyciu; **przenośne**, bo pozwalają rozwiązać szeroki zakres problemów, od członkostwa w grupie przez elekcję po zatwierdzanie atomowe; wreszcie **efektywne dla małych wartości $f$**. Ich wadą są **skomplikowane algorytmy**, które muszą radzić sobie z błędnymi podejrzeniami.

Zasadnicze ograniczenie jest jednak takie, że detektory **nie są implementowalne w systemie asynchronicznym**, ponieważ ich własności musiałyby zachodzić **na zawsze**, ewentualnie od pewnego nieznanego, ale skończonego momentu. W praktyce implementuje się więc rozwiązania zbliżone, w których własności zachodzą **dostatecznie długo**, to znaczy na tyle długo, by rozwiązać problem. Formalizuje to pojęcie **algorytmu wyrozumiałego** (*indulgent*): taki algorytm **kończy się i produkuje poprawny wynik, jeśli detektor zachowuje się zgodnie ze swoją specyfikacją; jeśli detektor specyfikacji nie spełnia, algorytm może się nie zakończyć, ale jeśli się zakończy, wynik zawsze jest poprawny**. Zakłada się przy tym istnienie **okresów stabilności**, podczas których własności detektora zachodzą — i to właśnie w tych okresach możliwe są konsensus oraz replikacja, zarówno aktywna, jak i pasywna.

### Rozwiązywalność problemów a siła modelu
Całość zagadnienia podsumowuje uporządkowanie problemów według siły modelu potrzebnej do ich rozwiązania. **RBcast** wystarcza $\mathbb{S}^{Async}\{\varnothing\}$. **Konsensus, TOcast i VScast** wymagają $\mathbb{S}^{Async}\{\Diamond\mathcal{W}\}$. **Porozumienie bizantyjskie, równoważne mu TRBcast oraz nieblokujące zatwierdzanie atomowe** wymagają $\mathbb{S}^{Async}\{\mathcal{P}\}$. **Synchronizacja zegarów** wymaga już pełnej synchroniczności, czyli $\mathbb{S}^{Sync}\{\varnothing\}$.
