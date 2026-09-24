---
tags:
  - obrona
  - odpowiedź
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 5
źródło: "[[RSO Z5 Algorytmy elekcji]]"
---
# 5. Algorytmy elekcji
---
> Wypowiedź na obronę. Sekcja **Odpowiedź** to wersja na 2–3 minuty; **Rozwinięcia** to dalsze ciągi tej samej wypowiedzi — każde odpowiada jednemu pogrubionemu hasłu i można je wpleść w to miejsce. Całość czytana ciągiem to ok. 8 minut. Koszty i tabela porównawcza: [[RSO Z5 Algorytmy elekcji]].

## Odpowiedź

Elekcja to wybór spośród procesów **jednego koordynatora**, uznawanego za koordynatora przez wszystkie procesy poprawne. Typowo wybiera się działający proces o **największym identyfikatorze**, a samą elekcję uruchamia się na starcie systemu albo po wykryciu awarii dotychczasowego koordynatora.

Warto od razu powiedzieć, po co to jest, bo elekcja to cegła, na której stoi zaskakująco dużo innych mechanizmów: koordynator w scentralizowanym wzajemnym wykluczaniu, sekwencer w rozgłaszaniu totalnym, regeneracja utraconego żetonu, koordynator protokołów zatwierdzania dwu- i trójfazowego, wreszcie lider w Paxosie i Rafcie. Za każdym razem, gdy jakiś algorytm mówi „niech jeden wyróżniony proces zrobi X", ktoś musi tego jednego wskazać.

**Wymagania** stawiamy dwa. Bezpieczeństwo: każdy uczestniczący proces albo jeszcze nie zdecydował, albo wskazuje **ten sam** działający proces o największym identyfikatorze. Żywotność: wszystkie procesy poprawne ostatecznie coś wskażą. **Założenia** też dwa: procesy mają unikalne, porównywalne identyfikatory, a elekcję może rozpocząć jednocześnie wielu inicjatorów — algorytm musi to znosić.

Algorytmy dobiera się do **topologii i modelu synchroniczności**. Dla grafu pełnego i systemu synchronicznego klasyką jest **algorytm tyrana**, czyli Bully. Dla pierścienia mamy trzy klasyczne algorytmy: **Chang-Roberts**, **LeLann** i **Hirschberg-Sinclair**, przy czym ten ostatni schodzi do kosztu rzędu $N \log N$, asymptotycznie optymalnego. Dla dowolnego grafu stosuje się **echo z wygaszaniem**. A rozwiązaniem praktycznym, dzisiejszym, jest elekcja **z losowością**, tak jak w **Rafcie**.

Najważniejsza pointa jest jednak taka: w systemie **asynchronicznym z awariami elekcja jest nierozwiązywalna deterministycznie**, bo wiarygodna elekcja byłaby równoważna doskonałemu detektorowi awarii, a konsensus w tym modelu jest niemożliwy — to wynik FLP. Dlatego algorytmy praktyczne zakładają **częściową synchroniczność** i godzą się na to, że przez chwilę może istnieć więcej niż jedno domniemane kierownictwo; bezpieczeństwo ratuje wtedy mechanizm kadencji albo numerów propozycji. Algorytm elekcji nie jest więc samodzielny — jest tak dobry, jak model synchroniczności, w którym go uruchomiono.

## Rozwinięcia

### Algorytm tyrana

Bully, zaproponowany przez Garcíę-Molinę w 1982, działa w **grafie pełnym**: każdy proces zna identyfikatory wszystkich i może się z każdym skomunikować. Zakłada system **synchroniczny**, bo znane ograniczenia opóźnień pozwalają użyć **timeoutów jako detektora awarii**. Idea jest prosta: proces, który wykrył awarię koordynatora, zaczepia wyłącznie procesy o **wyższych** identyfikatorach. Jeżeli żaden nie odpowie, sam zostaje koordynatorem i ogłasza to niżej; jeżeli któryś odpowie, ten przejmuje elekcję i powtarza to samo wyżej. Nazwa bierze się stąd, że proces odtworzony po awarii rozpoczyna elekcję i — mając największy identyfikator — przejmuje władzę nawet wtedy, gdy działający koordynator już jest. Koszt waha się od $N-2$ komunikatów, gdy awarię wykrył proces o drugim największym identyfikatorze, do kwadratu z $N$, gdy wykrył ją proces najmniejszy. Słabość jest wprost konsekwencją założenia: poprawność opiera się na timeoutach, więc **fałszywe podejrzenie daje dwóch koordynatorów**.

### Algorytmy pierścieniowe

**Chang-Roberts** z 1979 działa w pierścieniu jednokierunkowym, w którym proces zna wyłącznie swojego następnika. W pierścieniu krąży komunikat z identyfikatorem, a reguła jest jedna: proces **przepuszcza dalej identyfikator większy od własnego, a mniejszy zastępuje swoim**; dodatkowa flaga uczestnictwa tłumi komunikaty zbędne. Proces, do którego wraca jego własny identyfikator, wie, że jest największy, i ogłasza się liderem. W najgorszym przypadku — gdy identyfikatory maleją zgodnie z kierunkiem pierścienia — kosztuje to kwadrat z $N$, średnio $N \log N$, a w najlepszym razie $3N-1$. **LeLann** z 1977 jest prostszy i droższy: każdy komunikat obiega pełne koło, więc proces, do którego wraca własny komunikat, zna identyfikatory wszystkich inicjatorów i po prostu wybiera największy; wymaga kanałów FIFO i kosztuje kwadrat z $N$ **zawsze**. **Hirschberg-Sinclair** z 1980 atakuje ten koszt kwadratowy, ale potrzebuje pierścienia dwukierunkowego: algorytm działa w **fazach**, w których aktywny proces wysyła sondy w obie strony na odległość dwa do potęgi numeru fazy. Sondę pochłania każdy proces o większym identyfikatorze, a do następnej fazy przechodzi tylko ten, któremu sondy wróciły z obu stron. Liderem zostaje proces, którego sonda obiegnie cały pierścień. To daje koszt rzędu $N \log N$ — asymptotycznie optymalny dla pierścieni porównujących identyfikatory.

### Wariant pierścieniowy odporny na awarie

Klasyczne algorytmy pierścieniowe zakładają, że nikt nie pada — a przecież elekcję uruchamia się właśnie po awarii. Dlatego stosuje się wariant, w którym komunikat elekcyjny **gromadzi listę identyfikatorów** procesów, przez które przeszedł, a niedostępny następnik jest pomijany, co wymaga, żeby proces znał także dalszych następników. Kiedy komunikat wróci do inicjatora, ten wybiera z listy największy identyfikator i rozsyła go jako decyzję. Zaleta jest taka, że jeden obieg załatwia zarówno zebranie informacji, jak i ominięcie procesów martwych.

### Echo z wygaszaniem

Kiedy topologia jest **dowolnym grafem**, a nie pierścieniem ani kliką, używa się schematu falowego. Każdy inicjator uruchamia **falę** — algorytm echa — oznaczoną swoim identyfikatorem. Reguła jest taka, że proces uczestniczy wyłącznie w fali o największym znanym mu identyfikatorze, a fale mniejsze **wygasza**. W efekcie do swojego źródła wróci tylko fala największego inicjatora i to on zostaje liderem. Kosztuje to iloczyn liczby procesów i liczby krawędzi, bo w pesymistycznym przebiegu każda fala może przejść po całym grafie, zanim zostanie wygaszona.

### Raft i rola losowości

Raft, z 2014, pokazuje, jak elekcję robi się dziś w praktyce. Czas porządkuje **kadencjami**, a procesy trzyma w trzech stanach: follower, candidate i leader. Follower, który przez **losowy** timeout nie dostał sygnału życia od lidera, zwiększa numer kadencji, głosuje na siebie i prosi pozostałych o głos. Każdy proces oddaje w danej kadencji **co najwyżej jeden głos**, i to tylko na kandydata, którego log nie jest starszy od własnego. Liderem zostaje ten, kto zbierze **większość**. Warto wskazać, po co jest tu każdy z tych elementów: losowość timeoutów minimalizuje podział głosów, czyli sytuację, w której kilku kandydatów startuje naraz i nikt nie zbiera większości; wymóg większości gwarantuje, że w jednej kadencji nie da się wybrać dwóch liderów, bo dwie większości zawsze się przecinają; a warunek o logu zapewnia, że liderem nie zostanie ktoś, kto zgubił zatwierdzone wpisy.

### Elekcja a model systemu

Na koniec rzecz, którą warto powiedzieć wprost, bo spina całe zagadnienie. W modelu **asynchronicznym z awariami** nie da się deterministycznie rozwiązać elekcji: gdybyśmy potrafili wiarygodnie wskazać „działający proces o największym identyfikatorze", mielibyśmy doskonały detektor awarii, a z nim rozwiązalibyśmy konsensus — którego, jak wiadomo z wyniku FLP, w tym modelu rozwiązać nie można. Stąd dwie strategie w praktyce. Pierwsza to **założyć synchroniczność** i oprzeć się na timeoutach, jak Bully, płacąc ryzykiem dwóch koordynatorów przy fałszywym podejrzeniu. Druga to **założyć częściową synchroniczność**, dopuścić przejściowo więcej niż jednego pretendenta i uratować bezpieczeństwo numeracją — kadencjami w Rafcie, numerami propozycji w Paxosie. Szczegóły tej granicy to [[RSO Z1 Komunikacja grupowa|zagadnienie 1]] po stronie rozgłaszania i [[23 Rozproszone uzgadnianie w środowisku zawodnym|zagadnienie 23]] po stronie konsensusu.
