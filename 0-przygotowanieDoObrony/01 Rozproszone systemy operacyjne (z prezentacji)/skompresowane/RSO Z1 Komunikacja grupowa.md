---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 1
---
# 1. Komunikacja grupowa
---
> Komplet pojęć zagadnienia. Algorytmy są nazwane i opatrzone informacją, jaki problem rozwiązują, ale ich przebieg, pseudokod i zadania z prezentacji pozostają w [[RSO 01 Komunikacja grupowa|RSO 01]]. Materiał, którego prezentacje nie zawierają, jest oznaczony blokami `[!note]`.

## Czym jest komunikacja grupowa

**Komunikacja grupowa** (*group communication*) to mechanizm umożliwiający **rozsyłanie** (*multicast*) wiadomości przez organizowanie procesów — szerzej: obiektów — w **grupy**. Nieformalnie **rozgłaszanie** to abstrakcja komunikacyjna, za pomocą której proces wysyła wiadomość do grupy procesów; w mechanizmach **niezawodnych** dochodzą gwarancje utrzymywane **pomimo awarii**. Pozwala to modelować niezawodną komunikację przy założeniu, że zbiór procesów **dynamicznie się zmienia**. Typowe zastosowania to systemy z wieloma uczestnikami oraz **zwielokrotnianie** (replikacja).

Zagadnienie rozpada się na **dwa aspekty**: **zarządzanie grupami procesów** (usługa członkostwa, *membership service*) oraz **algorytmy niezawodnego rozsyłania wiadomości w grupie**. Cała reszta zagadnienia to rozwinięcie tych dwóch.

### Grupa i obraz grupy

**Grupa** to rzeczywisty zbiór procesów uczestniczących we wspólnym przetwarzaniu i komunikujących się przez przekazywanie wiadomości. **Obraz grupy** (*view*) to grupa **widziana w danej chwili** czasu rzeczywistego w pojedynczym procesie; obraz generuje usługa członkostwa. Rozróżnienie jest istotne, bo **różne procesy mogą mieć w tym samym momencie różne obrazy tej samej grupy** — po awarii jednego z członków jeden proces może już ją odnotować, a drugi jeszcze nie.

### Architektura i dwa poziomy przekazania wiadomości

Aplikacja rozmawia z **warstwą komunikacji grupowej** przez trzy operacje: **wyślij** (w dół), **odbierz** (w górę) i **zmiana_obrazu** (w górę); z zewnątrz do warstwy docierają sygnały **awaria** i **powrót**.

Stąd bierze się rozróżnienie, na którym opiera się cała reszta zagadnienia: **odebranie** wiadomości przez warstwę komunikacyjną to nie to samo co jej **dostarczenie** aplikacji. Wiadomość odebrana może zostać zabuforowana i przekazana w górę dopiero wtedy, gdy pozwoli na to wymagany porządek. **Porządek dostarczania może więc różnić się od porządku odbioru** — i to właśnie buforowanie realizuje wszystkie porządki opisane niżej.

Przykładowe systemy komunikacji grupowej (GCS): **ISIS** (pionierski), **Horus/Ensemble** (jego nowoczesna wersja), **JGroups** (dla Javy), **Transis**.

> [!note] Uzupełnienie spoza prezentacji — klasyfikacja grup
> Prezentacje definiują tylko grupę i obraz grupy. Lista egzaminacyjna wymaga ponadto podziałów: **grupa zamknięta** (wiadomości wysyłają wyłącznie jej członkowie, np. zbiór replik) vs **otwarta** (może do niej wysłać dowolny proces z zewnątrz, np. klient replikowanego serwisu); **płaska** (wszyscy równorzędni, brak pojedynczego punktu awarii, ale kosztowne uzgadnianie) vs **hierarchiczna** (jest koordynator, decyzje prostsze, ale koordynator jest SPoF); **statyczna** (stały skład) vs **dynamiczna** (procesy dołączają, odchodzą i ulegają awariom). Źródło: [[01 Komunikacja grupowa]].

## Usługa członkostwa i synchronizacja widoków

**Usługa członkostwa** utrzymuje aktualny skład grupy i wykrywa awarie, a każda zmiana składu to **zmiana widoku** (*view change*).

> [!note] Uzupełnienie spoza prezentacji — virtual synchrony
> **Synchronizacja widoków** (*virtual synchrony*) porządkuje zmiany widoku względem wiadomości: każda wiadomość jest dostarczana **w tym samym widoku** przez wszystkie procesy przechodzące do widoku następnego. Przy zmianie widoku wykonywany jest **flush** — procesy wymieniają między sobą wiadomości **niestabilne**, czyli niepotwierdzone jeszcze przez wszystkich, i dopiero potem instalują nowy widok. Realizacje: ISIS, Horus, JGroups, Spread. Źródło: [[01 Komunikacja grupowa]].

## Własności rozgłaszania niezawodnego

Specyfikacje wszystkich mechanizmów budowane są z tego samego zestawu własności.

**Ważność** (*validity*) — wiadomość rozgłoszona przez proces poprawny zostaje ostatecznie dostarczona. W wariancie **best-effort** dotyczy tylko par procesów poprawnych: jeżeli $P_i$ oraz $P_j$ są poprawne, to każda wiadomość rozgłoszona przez $P_i$ trafi ostatecznie do $P_j$.

**Brak powielania** (*no duplication*) — wiadomość dostarczona jest dostarczona **co najwyżej raz**.

**Brak samogeneracji** (*no creation*) — jeżeli wiadomość została dostarczona, to wcześniej została przez jakiś proces rozgłoszona. Dwie ostatnie własności bywają łącznie nazywane **integralnością**.

**Zgodność** (*agreement*) — jeżeli wiadomość odebrał pewien **poprawny** proces, to ostatecznie odbiorą ją **wszystkie procesy poprawne**.

**Jednolita zgodność** (*uniform agreement*) — jeżeli wiadomość odebrał **jakikolwiek** proces, poprawny **bądź niepoprawny**, to ostatecznie odbiorą ją wszystkie procesy poprawne.

Różnica między dwiema ostatnimi własnościami jest jedyną, która dzieli RB od URB, i warto ją umieć wypowiedzieć wprost: w **RB** wiadomość odebrana wyłącznie przez proces, który zaraz potem uległ awarii, **może przepaść** — pozostałe procesy nie muszą jej dostać. W **URB** takie odebranie już „zaraża": skoro ktokolwiek ją zobaczył, muszą ją zobaczyć wszyscy poprawni. URB jest potrzebne wszędzie tam, gdzie proces mógł na podstawie odebranej wiadomości wykonać **widoczny na zewnątrz** efekt, zanim padł.

## Hierarchia mechanizmów

| Skrót | Nazwa polska | Nazwa angielska |
|---|---|---|
| **BEB** | podstawowe rozgłaszanie niezawodne | *best-effort broadcast* |
| **RB** | zgodne rozgłaszanie niezawodne | *regular reliable broadcast* |
| **URB** | jednolite rozgłaszanie niezawodne | *uniform reliable broadcast* |
| **RFB** | zgodne rozgłaszanie z uporządkowaniem FIFO | *FIFO reliable broadcast* |
| **RCB** | zgodne rozgłaszanie z uporządkowaniem przyczynowym | *causal reliable broadcast* |
| **TO** | zgodne rozgłaszanie z uporządkowaniem globalnym | *total order reliable broadcast* |

Hierarchia narasta przez dokładanie własności: $\text{BEB} \xrightarrow{+\ \text{zgodność}} \text{RB} \xrightarrow{+\ \text{jednolitość}} \text{URB}$, a porządki są **ortogonalnym** dodatkiem do RB: $\text{RB}+\text{FIFO}=\text{RFB}$, $\text{RB}+\text{CO}=\text{RCB}$, $\text{RB}+\text{TO}=\text{TO}$.

### BEB

**BEB** zakłada **kanały niezawodne** (*perfect point-to-point links*) i **nie wymaga detektora awarii** — przyjmuje więc model z **ukrytymi awariami** (*fail-silent*). Nadawca po prostu wysyła wiadomość każdemu członkowi grupy, a odbiorca przekazuje ją w górę. Złożoność czasowa **1**, komunikacyjna **$n$**. Przebieg: [[RSO 01 Komunikacja grupowa#Algorytm|RSO 01]].

### RB — dwa algorytmy

**Algorytm pasywny** (*lazy reliable broadcast*) rozwiązuje problem **zgodności mimo awarii nadawcy**. Wymaga BEB oraz **doskonałego detektora awarii**. Idea: proces **nie retransmituje** wiadomości, dopóki jej nadawca żyje; zapamiętuje ją i „odkurza" dopiero wtedy, gdy detektor zgłosi awarię nadawcy. Stąd nazwa: leniwy. Złożoność optymistyczna $1$ / $n$, pesymistyczna $n$ / $n^2$. Przebieg: [[RSO 01 Komunikacja grupowa#Algorytm pasywny (_lazy reliable broadcast_)|RSO 01]].

**Algorytm aktywny** (*eager reliable broadcast*, *flooding*) rozwiązuje ten sam problem **bez detektora awarii**. Idea: każdy proces, który odbiera wiadomość **po raz pierwszy**, natychmiast rozgłasza ją dalej — niezależnie od tego, czy nadawca żyje. Ceną jest stały koszt $n^2$ komunikatów: złożoność optymistyczna $1$ / $n^2$, pesymistyczna $n$ / $n^2$. Przebieg: [[RSO 01 Komunikacja grupowa#Algorytm aktywny (_eager reliable broadcast_)|RSO 01]].

Zestawienie obu jest klasycznym kompromisem: **detektor awarii kupuje nam niższy koszt komunikacyjny w przypadku bezawaryjnym**.

### URB

**Algorytm z potwierdzeniami od wszystkich** (*all-ack uniform reliable broadcast*) rozwiązuje problem **jednolitej zgodności**. Wymaga BEB, niezawodnych kanałów i doskonałego detektora awarii. Idea: wiadomość **wolno dostarczyć aplikacji dopiero wtedy**, gdy potwierdziły ją — przez retransmisję BEB — **wszystkie procesy uznawane w tym momencie za poprawne**. Odłożenie dostarczenia do chwili, gdy wiadomość jest już u wszystkich, jest jedynym sposobem, by odebranie przez proces skazany na awarię nie było odebraniem „na wyłączność". Stąd dwa kroki zamiast jednego: złożoność optymistyczna $2$ / $n^2$, pesymistyczna $n+1$ / $n^2$. Przebieg: [[RSO 01 Komunikacja grupowa#Algorytm z potwierdzeniami od wszystkich (_all-ack uniform reliable broadcast_)|RSO 01]].

| Mechanizm | Czasowa opt. | Komunikacyjna opt. | Czasowa pes. | Komunikacyjna pes. |
|---|---|---|---|---|
| BEB | 1 | $n$ | — | — |
| RB pasywny | 1 | $n$ | $n$ | $n^2$ |
| RB aktywny | 1 | $n^2$ | $n$ | $n^2$ |
| URB all-ack | 2 | $n^2$ | $n+1$ | $n^2$ |

## Porządki dostarczania

### Porządek FIFO

**RFB** to RB plus warunek **FIFO order**: jeżeli proces $P_i$ rozgłosił $m_1$ **przed** $m_2$, to żaden proces nie odbierze $m_2$, jeśli nie odebrał wcześniej $m_1$. Porządek obowiązuje **tylko w obrębie jednego nadawcy** — wiadomości różnych nadawców pozostają nieuporządkowane.

### Porządek przyczynowy

Relacja $m_1 \rightarrow m_2$ („$m_1$ przyczynowo poprzedza $m_2$") zachodzi, gdy: **(a)** obie wiadomości rozgłosił ten sam proces, $m_1$ przed $m_2$; **(b)** $m_1$ została odebrana przez pewien proces $P_i$, a $m_2$ została rozgłoszona przez $P_i$ **po odebraniu** $m_1$; **(c)** istnieje $m_3$ takie, że dla pary $m_1, m_3$ oraz pary $m_3, m_2$ zachodzi (a) lub (b) — czyli przez **domknięcie przechodnie**.

**RCB** to RB plus warunek **causal order**: proces nie odbierze $m_2$, dopóki nie odebrał wszystkich $m_1$ takich, że $m_1 \rightarrow m_2$. Ponieważ punkt (a) definicji to dokładnie warunek FIFO, **RCB $\Rightarrow$ RFB**. Typowy kontrprzykład odwrotnej implikacji: $P_1$ rozgłasza $m_1$, $P_2$ po jej dostarczeniu rozgłasza odpowiedź $m_2$, a $P_3$ dostarcza $m_2$ przed $m_1$ — FIFO spełnione (różni nadawcy), przyczynowość naruszona.

### Porządek globalny

**TO** to RB plus **globalne uporządkowanie** (*total order*): jeżeli poprawne procesy $P_i$ i $P_j$ odbierają wiadomość $m$ i $P_i$ odebrał $m'$ przed $m$, to także $P_j$ odbiera $m'$ przed $m$. W wariancie **jednolitym** (*uniform total order*) znika ograniczenie do procesów poprawnych.

Porządek globalny mówi wyłącznie, że **wszyscy dostarczają w tej samej kolejności** — nie mówi, że jest to kolejność zgodna z FIFO czy z przyczynowością. Stąd osobne warianty **FIFO-total** i **causal-total**.

> [!note] Uzupełnienie spoza prezentacji — algorytmy realizacji porządków
> Prezentacje podają wyłącznie **specyfikacje** RFB, RCB i TO. Realizacje, wymagane przez listę zagadnień:
> - **FIFO — numery sekwencyjne.** Nadawca numeruje własne rozgłoszenia kolejno; odbiorca trzyma licznik oczekiwanego numeru od każdego nadawcy i bufor, dostarcza wiadomość dopiero gdy jej numer jest tym oczekiwanym.
> - **CO — zegary wektorowe** (Birman-Schiper-Stephenson, CBCAST). Wiadomość niesie wektor nadawcy; odbiorca dostarcza ją, gdy jest **kolejną** wiadomością tego nadawcy i gdy odbiorca dostarczył już wszystko, co nadawca widział w chwili wysłania. Pozostałe czekają w buforze.
> - **TO — sekwencer.** Wariant ze **stałym sekwencerem** (jeden proces nadaje globalne numery — prosty, ale SPoF i wąskie gardło) oraz z **ruchomym sekwencerem** (numeruje aktualny posiadacz tokenu, np. Totem).
> - **TO — ISIS** (Skeen, uzgadnianie priorytetów): odbiorcy proponują priorytety, nadawca wybiera maksimum jako ostateczny, kolejka jest sortowana, a dostarczane są wiadomości z czoła kolejki, gdy staną się dostarczalne. Koszt $3N$ komunikatów, **bez SPoF**.
> - **TO — przez konsensus** (Chandra-Toueg): procesy w rundach uzgadniają **zbiór** wiadomości do dostarczenia, a zbiór dostarczają w kolejności deterministycznej. Stąd fundamentalny fakt: **rozgłaszanie totalne jest równoważne konsensusowi** — zob. [[23 Rozproszone uzgadnianie w środowisku zawodnym]].
>
> Źródło: [[01 Komunikacja grupowa]].

## Podstawa formalna porządków

Porządek przyczynowy wiadomości opiera się na **relacji poprzedzania** i **zegarach logicznych**, które w tym przedmiocie omawia osobny wykład: **zegar skalarny Lamporta** daje implikację jednostronną $a \rightarrow b \Rightarrow C(a) < C(b)$, a **zegar wektorowy Matterna** — równoważność, dzięki czemu **wykrywa współbieżność**. Szczegóły: [[RSO 08 Czas wirtualny i złożoność algorytmów#Zegar skalarny|RSO 08]], [[RSO 08 Czas wirtualny i złożoność algorytmów#Zegar wektorowy|RSO 08]]. Środowisko zachowujące uporządkowanie przyczynowe: [[RSO 08 Czas wirtualny i złożoność algorytmów#Środowisko zachowujące uporządkowanie przyczynowe|RSO 08]].

Alternatywą probabilistyczną wobec deterministycznych mechanizmów rozgłaszania jest **rozgłaszanie epidemiczne** — dobrze skalowalne, bez gwarancji deterministycznych: [[Algorytmy Rozproszone/Gossiping]].

---
## Czego w prezentacjach nie ma

> [!todo] Zakres zagadnienia wykracza poza slajdy
> Prezentacje nie zawierają: **klasyfikacji grup** (otwarte/zamknięte, płaskie/hierarchiczne, statyczne/dynamiczne), szczegółów **usługi członkostwa**, **zmiany widoku** i **synchronizacji widoków**, **algorytmów** realizacji porządków FIFO/CO/TO (numery sekwencyjne, zegary wektorowe, sekwencer, ISIS) ani definicji **detektora awarii** i jego klas ($P$, $\Diamond P$, $S$, $\Diamond S$) — mimo że dwa z omawianych algorytmów zakładają detektor doskonały. Rejestr braków z opisem, co dokładnie sprawdzono: [[RSO 01 Komunikacja grupowa#Braki w prezentacjach]]. Materiał uzupełniający: [[01 Komunikacja grupowa]], [[Algorytmy Rozproszone/Zgodne Rozgłaszanie Niezawodne FIFO]].
