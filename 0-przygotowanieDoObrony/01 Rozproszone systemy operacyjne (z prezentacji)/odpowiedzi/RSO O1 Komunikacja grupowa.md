---
tags:
  - obrona
  - odpowiedź
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 1
źródło: "[[RSO Z1 Komunikacja grupowa]]"
---
# 1. Komunikacja grupowa
---
> Wypowiedź na obronę. Sekcja **Odpowiedź** to wersja na 2–3 minuty; **Rozwinięcia** to dalsze ciągi tej samej wypowiedzi — każde odpowiada jednemu pogrubionemu hasłu i można je wpleść w to miejsce. Całość czytana ciągiem to ok. 8 minut. Szczegóły algorytmów: [[RSO Z1 Komunikacja grupowa]].

## Odpowiedź

Komunikacja grupowa to mechanizm, który pozwala procesowi wysłać jedną wiadomość do całego zbioru procesów zorganizowanych w **grupę**, i to z gwarancjami utrzymywanymi **pomimo awarii**. Sens tego zagadnienia polega na tym, że w systemie rozproszonym zbiór uczestników zmienia się dynamicznie — procesy padają, wracają, dołączają — a mimo to chcemy móc powiedzieć coś pewnego o tym, kto i w jakiej kolejności zobaczył daną wiadomość. Stosuje się to wszędzie tam, gdzie jest wielu uczestników i gdzie coś się replikuje.

Całość rozpada się na dwa aspekty. Pierwszy to **zarządzanie składem grupy**, czyli usługa członkostwa. Drugi to **algorytmy niezawodnego rozsyłania** w obrębie tej grupy. Reszta zagadnienia jest rozwinięciem tych dwóch.

Fundamentem dla drugiego aspektu jest jedno rozróżnienie: **odebranie wiadomości przez warstwę komunikacyjną to nie to samo co dostarczenie jej aplikacji**. Wiadomość odebraną można zabuforować i przekazać w górę dopiero wtedy, gdy pozwoli na to wymagany porządek — i to właśnie buforowanie realizuje wszystkie gwarancje kolejności.

Specyfikacje poszczególnych mechanizmów buduje się z jednego, powtarzalnego **zestawu własności**: ważność, brak powielania, brak samogeneracji, zgodność i jednolita zgodność. Dokładanie ich kolejno daje **hierarchię mechanizmów**. Najsłabszy jest **BEB**, czyli rozgłaszanie best-effort, które po prostu wysyła wiadomość każdemu. Dodanie zgodności daje **RB** — wiadomość odebrana przez proces poprawny trafi ostatecznie do wszystkich poprawnych. Dodanie jednolitości daje **URB**, gdzie wystarczy, że wiadomość zobaczył ktokolwiek, choćby proces, który zaraz potem padł, i już wszyscy poprawni muszą ją dostać.

Ortogonalnie do tej hierarchii dokłada się **porządki dostarczania**: **FIFO**, czyli zachowanie kolejności w obrębie jednego nadawcy, **przyczynowy**, czyli zachowanie relacji „ta wiadomość jest odpowiedzią na tamtą", i **globalny**, czyli identyczna kolejność u wszystkich odbiorców. Ten ostatni jest najdroższy i najciekawszy, bo **rozgłaszanie z porządkiem globalnym jest równoważne problemowi konsensusu** — a to znaczy, że dziedziczy wszystkie jego ograniczenia.

## Rozwinięcia

### Grupa, obraz grupy i rodzaje grup

Warto tu rozdzielić dwa pojęcia. **Grupa** to rzeczywisty zbiór procesów uczestniczących we wspólnym przetwarzaniu. **Obraz grupy**, czyli *view*, to grupa widziana w danej chwili przez pojedynczy proces. Rozróżnienie nie jest pedanterią: różne procesy mogą w tym samym momencie mieć różne obrazy tej samej grupy, bo po awarii jednego z członków jeden proces zdążył ją już odnotować, a drugi jeszcze nie. Same grupy dzieli się trojako — na **zamknięte**, gdzie wysyłać mogą tylko członkowie, jak w zbiorze replik, i **otwarte**, gdzie może wysłać dowolny klient z zewnątrz; na **płaskie**, gdzie wszyscy są równorzędni i nie ma pojedynczego punktu awarii, ale uzgadnianie jest kosztowne, i **hierarchiczne**, z koordynatorem, który upraszcza decyzje, ale sam jest punktem awarii; wreszcie na **statyczne** o stałym składzie i **dynamiczne**.

### Usługa członkostwa i synchronizacja widoków

Zarządzanie składem grupy realizuje **usługa członkostwa**: utrzymuje aktualny skład, wykrywa awarie, a każdą zmianę składu sygnalizuje aplikacji jako **zmianę widoku**. Samo to jednak nie wystarcza, bo zmiany widoku trzeba jakoś uporządkować względem strumienia wiadomości. Robi to **synchronizacja widoków**, czyli *virtual synchrony*: wymaga się, żeby każda wiadomość była dostarczona w tym samym widoku przez wszystkie procesy, które przechodzą do widoku następnego. Realizuje się to przez **flush** — przed zainstalowaniem nowego widoku procesy wymieniają między sobą wiadomości niestabilne, czyli takie, których nie potwierdzili jeszcze wszyscy. Tak działa ISIS, Horus, JGroups czy Spread.

### Dwa poziomy przekazania wiadomości

Architektonicznie aplikacja rozmawia z warstwą komunikacji grupowej przez trzy operacje: **wyślij** w dół oraz **odbierz** i **zmiana_obrazu** w górę; z zewnątrz do warstwy docierają jeszcze sygnały o awarii i powrocie procesu. I właśnie dlatego, że między odbiorem a dostarczeniem jest ta warstwa z buforem, **porządek dostarczania może różnić się od porządku odbioru**. Każdy z porządków, o których mówiłem, sprowadza się technicznie do reguły: „trzymaj wiadomość w buforze, dopóki nie dostarczyłeś wszystkiego, co według tej reguły ma ją poprzedzać".

### Własności rozgłaszania niezawodnego

Zestaw własności wygląda tak. **Ważność** mówi, że wiadomość rozgłoszona przez proces poprawny zostanie ostatecznie dostarczona; w wariancie best-effort dotyczy tylko par procesów poprawnych. **Brak powielania** — wiadomość jest dostarczana co najwyżej raz. **Brak samogeneracji** — jeżeli coś zostało dostarczone, to wcześniej ktoś to rozgłosił; te dwie razem nazywa się integralnością. **Zgodność** — jeżeli wiadomość odebrał pewien proces poprawny, to odbiorą ją ostatecznie wszystkie procesy poprawne. I wreszcie **jednolita zgodność**, gdzie zamiast „pewien proces poprawny" mamy „jakikolwiek proces, poprawny bądź nie". Ta ostatnia różnica to jedyne, co dzieli RB od URB, i warto ją wypowiedzieć wprost: w RB wiadomość odebrana wyłącznie przez proces, który zaraz potem uległ awarii, może przepaść. W URB takie odebranie już „zaraża" — skoro ktokolwiek ją zobaczył, muszą ją zobaczyć wszyscy poprawni. URB jest potrzebne wszędzie tam, gdzie proces mógł na podstawie odebranej wiadomości wykonać efekt widoczny na zewnątrz, zanim padł.

### Algorytmy i ich koszt

**BEB** zakłada kanały niezawodne i nie wymaga detektora awarii — to model fail-silent. Nadawca wysyła wiadomość każdemu członkowi, odbiorca przekazuje ją w górę; koszt to jedna jednostka czasu i $n$ komunikatów. **RB** ma dwie realizacje, które różnią się dokładnie tym, czy mamy detektor awarii. Algorytm **pasywny**, *lazy*, wymaga detektora doskonałego i nie retransmituje wiadomości, dopóki jej nadawca żyje — dopiero gdy detektor zgłosi awarię nadawcy, procesy „odkurzają" zapamiętaną wiadomość; w przypadku bezawaryjnym kosztuje tyle co BEB, czyli $n$ komunikatów, w pesymistycznym $n^2$. Algorytm **aktywny**, *eager*, radzi sobie bez detektora: każdy, kto odbiera wiadomość po raz pierwszy, natychmiast rozgłasza ją dalej — za cenę stałego kosztu $n^2$. To klasyczny kompromis: detektor awarii kupuje nam niższy koszt w przypadku bezawaryjnym. **URB** realizuje algorytm z potwierdzeniami od wszystkich: wiadomość wolno dostarczyć aplikacji dopiero wtedy, gdy potwierdziły ją wszystkie procesy uznawane w tej chwili za poprawne. Odłożenie dostarczenia jest jedynym sposobem, żeby odebranie przez proces skazany na awarię nie było odebraniem „na wyłączność" — stąd dwa kroki zamiast jednego.

### Porządek FIFO

Najsłabszy z porządków mówi, że jeżeli proces rozgłosił $m_1$ przed $m_2$, to żaden odbiorca nie dostanie $m_2$ bez wcześniejszego $m_1$. Obowiązuje wyłącznie w obrębie jednego nadawcy — wiadomości różnych nadawców pozostają wzajemnie nieuporządkowane. Realizuje się go najprościej, **numerami sekwencyjnymi**: nadawca numeruje swoje rozgłoszenia kolejno, a odbiorca trzyma dla każdego nadawcy licznik oczekiwanego numeru i bufor, i dostarcza wiadomość dopiero wtedy, gdy jej numer jest tym oczekiwanym.

### Porządek przyczynowy

Relacja „$m_1$ przyczynowo poprzedza $m_2$" zachodzi w trzech przypadkach: gdy obie rozgłosił ten sam proces, pierwszą przed drugą; gdy jakiś proces odebrał $m_1$, a potem rozgłosił $m_2$; oraz przez domknięcie przechodnie tych dwóch. Ponieważ pierwszy przypadek to dokładnie warunek FIFO, porządek przyczynowy jest silniejszy — implikuje FIFO, ale nie odwrotnie. Typowy kontrprzykład: $P_1$ rozgłasza pytanie, $P_2$ po jego dostarczeniu rozgłasza odpowiedź, a $P_3$ dostaje odpowiedź przed pytaniem — FIFO jest spełnione, bo to różni nadawcy, a przyczynowość naruszona. Realizuje się to **zegarami wektorowymi**, w algorytmie Birmana-Schipera-Stephensona: wiadomość niesie wektor nadawcy, a odbiorca dostarcza ją dopiero, gdy jest ona kolejną wiadomością tego nadawcy i gdy odbiorca dostarczył już wszystko, co nadawca widział w chwili wysłania.

### Porządek globalny i jego związek z konsensusem

Porządek globalny wymaga, żeby wszystkie procesy poprawne dostarczały wiadomości w tej samej kolejności — ale sam z siebie nie mówi, że jest to kolejność zgodna z FIFO czy z przyczynowością, stąd osobne warianty FIFO-total i causal-total. Realizacji jest kilka. Najprostsza to **sekwencer**: jeden proces nadaje globalne numery, co jest proste, ale daje wąskie gardło i pojedynczy punkt awarii; wariant z sekwencerem ruchomym numeruje aktualny posiadacz tokenu. Klasyczny algorytm **ISIS** uzgadnia priorytety — odbiorcy proponują swoje, nadawca wybiera maksimum jako ostateczny, kolejka jest sortowana i dostarcza się z jej czoła; kosztuje $3N$ komunikatów, ale nie ma pojedynczego punktu awarii. Trzecie podejście to realizacja **przez konsensus**, jak u Chandry i Touega: procesy w rundach uzgadniają zbiór wiadomości do dostarczenia, a potem dostarczają go w kolejności deterministycznej. To pokazuje, że oba problemy są równoważne — a więc porządek globalny dziedziczy niemożność FLP i wymaga takich samych założeń o detekcji awarii jak konsensus, o czym mówi [[23 Rozproszone uzgadnianie w środowisku zawodnym|zagadnienie 23]].
