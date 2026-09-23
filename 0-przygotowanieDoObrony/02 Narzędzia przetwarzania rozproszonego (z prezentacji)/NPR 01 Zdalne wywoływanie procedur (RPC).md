---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 7
source: "slajdy–merged.pdf"
slajdy: "16–46"
---
# NPR 01. Zdalne wywoływanie procedur (RPC)
---
> Wykład wprowadza **RPC** jako próbę ukrycia komunikacji sieciowej pod składnią zwykłego wywołania procedury. Materiał dzieli się na dwie części zapowiedziane na slajdzie 16: **zagadnienia projektowe** (co trzeba rozstrzygnąć, żeby RPC w ogóle miało sens — przezroczystość dostępu, gwarancja wykonania, specyfikacja interfejsu, obsługa wyjątków) oraz **zagadnienia realizacyjne** (jak to zbudować — przetwarzanie interfejsu, wiązanie, obsługa komunikacji, semantyka błędu, osierocone obliczenia). Trzon stanowią **cztery semantyki błędu** i **trójwarstwowy protokół BLAST/CHAN/SELECT**.

---
## Od wywołania lokalnego do zdalnego
<sub>slajdy–merged.pdf, slajdy 17–19</sub>

Punktem wyjścia jest zwykły program lokalny — klient woła procedurę `zabij_proces`, która opakowuje uniksowe `kill`:

```c
main(int argc, char* argv[]) {
    int id, status;
    id = atoi(argv[1]);
    status = zabij_proces(id);
    exit(status);
}

int zabij_proces(int pid) {
    int stat;
    stat = kill(pid, 9);
    return stat;
}
```

**Idea RPC**: rozdzielić te dwa fragmenty na dwie maszyny tak, żeby **kod aplikacyjny się nie zmienił**. Interakcja sprowadza się do pary komunikatów — *żądanie klienta* w jedną stronę, *odpowiedź serwera* w drugą.

Po rozdzieleniu programista pisze nadal to samo `main` po stronie klienta i tę samą procedurę `zabij_proc` po stronie serwera; **cała reszta jest generowana przez system**:

![[npr-rpc-s19-namiastki-kod.png]]
<sub>Po lewej klient: aplikacja woła `zabij_proc`, ale ciało tej procedury zastępuje namiastka wysyłająca żądanie i czekająca na odpowiedź. Po prawej serwer: wygenerowane `main` odbiera żądanie, woła prawdziwą procedurę i odsyła wynik. slajdy–merged.pdf, slajd 19</sub>

---
## Zagadnienia projektowe
<sub>slajdy–merged.pdf, slajd 20</sub>

| Zagadnienie | Na czym polega |
|---|---|
| **Przezroczystość dostępu** | ukrycie komunikacji sieciowej przed aplikacją przez odpowiednie opakowanie funkcji komunikacyjnych **namiastką klienta** oraz **namiastką serwera** |
| **Gwarancja wykonania** | ukrywanie błędów komunikacyjnych |
| **Specyfikacja interfejsu** | sposób opisu sygnatur procedur zdalnych (nazwy, typy parametrów) |
| **Obsługa sytuacji wyjątkowych** | co zrobić, gdy wywołanie się nie powiedzie |

### Namiastki
<sub>slajd 21</sub>

| Pojęcie | Definicja |
|---|---|
| **Namiastka klienta** (ang. _client stub_) | udostępnienie aplikacji klienckiej **procedury lokalnej** odpowiedzialnej za przesłanie danych do serwera oraz odebranie wyników |
| **Namiastka serwera** (ang. _server stub_) | udostępnienie aplikacji po stronie serwera **procedury lokalnej** odpowiedzialnej za odebranie identyfikatora procedury zdalnej do wywołania i parametrów procedury, a odesłanie wyników lub zgłoszenie wyjątków |

Namiastka klienta ma dokładnie tę samą sygnaturę co procedura zdalna — to właśnie dzięki temu kod aplikacyjny nie musi wiedzieć, że wywołanie jest zdalne.

### Warstwy protokołu RPC
<sub>slajd 22</sub>

Protokół RPC rozpina się między namiastkami a protokołem warstwy sieciowej i składa się z **trzech warstw**, symetrycznych po obu stronach:

![[npr-rpc-s22-warstwy-protokolu.png]]
<sub>Warstwy protokołu RPC. slajdy–merged.pdf, slajd 22</sub>

Szczegóły każdej z warstw — zob. [[#Obsługa komunikacji klient-serwer]].

### Przebieg wywołania synchronicznego
<sub>slajd 23</sub>

W wariancie podstawowym klient **blokuje się** na czas całego wywołania: wysyła żądanie z argumentami, czeka, a wątek przetwarzania przenosi się na serwer.

![[npr-rpc-s23-przebieg-wywolania.png]]
<sub>Klient oczekuje przez cały czas lokalnego wykonania procedury na serwerze. slajdy–merged.pdf, slajd 23</sub>

---
## Przekazywanie parametrów
<sub>slajdy–merged.pdf, slajd 24</sub>

| Sposób | Ang. | Problem |
|---|---|---|
| **przez wartość** | _call-by-value_ | różnice w reprezentacji danych — kodowanie znaków, uporządkowanie bajtów, formaty liczb zmiennopozycyjnych |
| **przez referencję** | _call-by-reference_ | zinterpretowanie wartości wskaźnika w **innej przestrzeni adresowej** |
| **przez kopiowanie i odtwarzanie** | — | **problem przetaczania** — kwestia opisu struktur danych w celu prawidłowego zidentyfikowania wszystkich składowych |

Przekazywanie przez referencję w sensie dosłownym jest w RPC bezużyteczne: wskaźnik jest liczbą ważną wyłącznie w przestrzeni adresowej klienta. Stąd trzeci wariant — strukturę wskazywaną przez wskaźnik **kopiuje się w całości** do serwera, a po powrocie odtwarza po stronie klienta.

### Problem przetaczania
<sub>slajd 27</sub>

Żeby skopiować strukturę wskaźnikową, trzeba ją najpierw **obejść** i zidentyfikować wszystkie osiągalne składowe. To właśnie **przetaczanie** (ang. _marshalling_): graf obiektów połączonych wskaźnikami trzeba spłaszczyć do liniowej postaci nadającej się do przesłania, a po stronie odbiorczej odtworzyć — przy czym odtworzona struktura ma **inne adresy**, a współdzielone węzły mogą się zduplikować.

![[npr-rpc-s27-problem-przetaczania.png]]
<sub>Po lewej struktura oryginalna, w której jeden obiekt jest wskazywany z dwóch miejsc; po prawej struktura po odtworzeniu — współdzielenie zostało utracone. slajdy–merged.pdf, slajd 27</sub>

---
## Konwersja danych
<sub>slajdy–merged.pdf, slajdy 25–26</sub>

Problem różnic w reprezentacji danych da się rozwiązać na dwa sposoby.

### Reprezentacja kanoniczna
<sub>slajd 25</sub>

Ustala się **jeden wspólny format pośredni**. Każdy uczestnik konwertuje swoje dane do formatu kanonicznego przy wysyłaniu i z formatu kanonicznego przy odbiorze — nie musi znać architektury drugiej strony.

![[npr-rpc-s25-konwersja-kanoniczna.png]]
<sub>Wszystkie strony konwertują do i z jednego formatu. slajdy–merged.pdf, slajd 25</sub>

Kosztem jest **dwukrotna konwersja** przy każdej wymianie (nawet gdy obie strony mają tę samą architekturę) oraz możliwa **utrata dokładności**, gdy format kanoniczny jest uboższy od formatu natywnego. Tak działa [[NPR 02 Sun RPC i standard XDR|XDR]].

### Konwersja bezpośrednia
<sub>slajd 26</sub>

Każda para uczestników konwertuje **wprost** z formatu nadawcy na format odbiorcy — jedna konwersja zamiast dwóch, bez pośrednika.

![[npr-rpc-s26-konwersja-bezposrednia.png]]
<sub>Każda para konwertuje bezpośrednio między swoimi formatami. slajdy–merged.pdf, slajd 26</sub>

Kosztem jest **liczba procedur konwersji**: przy $n$ architekturach trzeba ich $n(n-1)$ zamiast $2n$, a każdy uczestnik musi **rozpoznać architekturę** partnera.

---
## Gwarancja wykonania — semantyka błędu
<sub>slajdy–merged.pdf, slajd 28</sub>

To centralne pojęcie całego wykładu. Pytanie brzmi: **jak należy zinterpretować przypadek wystąpienia błędu (wyjątku) w wywołaniu procedury zdalnej?** Odpowiedzi są cztery.

| Semantyka | Co wie klient |
|---|---|
| **Co najmniej raz** (_at-least-once_) | po uzyskaniu odpowiedzi (wyniku) od serwera klient ma pewność, że wywoływana procedura wykonała się **co najmniej raz** |
| **Co najwyżej raz** (_at-most-once_) | po uzyskaniu odpowiedzi (wyniku) od serwera klient wie, że wywoływana procedura wykonała się **dokładnie raz** |
| **Dokładnie raz** (_exactly-once_) | **niemożliwa do uzyskania**, jeśli system narażony jest na awarie (np. serwera lub łączy) |
| **Ewentualnie** (_maybe_) | **brak gwarancji** — procedura mogła się wykonać lub mogła się nie wykonać |

Kluczowe rozróżnienie: *co najwyżej raz* mówi coś o wykonaniu **tylko wtedy, gdy odpowiedź dotarła**. Gdy odpowiedź nie dotarła, klient nie wie nic. Dlatego „dokładnie raz" jako **bezwarunkowa** gwarancja jest nieosiągalna — potwierdzenie tego wyniku, przeniesione na grunt uzgadniania rozproszonego, to [[SWN 05 Problemy uzgadniania i wyniki niemożliwości|wyniki niemożliwości]].

> [!note] Uzupełnienie z Opracowania
> Slajd nie mówi, ile z tych semantyk jest w praktyce realizowanych. Opracowanie precyzuje: **dokładnie raz — wcale**, **ewentualnie — wcale**; realnie używa się wyłącznie *co najmniej raz* i *co najwyżej raz*. <sub>Opracowanie.pdf, s. 12</sub>

Semantyka *co najmniej raz* jest bezpieczna tylko dla **procedur idempotentnych** — zob. [[NPR 03 NFS, idempotentność i bezstanowość]].

Por. [[Narzędzia Przetwarzania Rozproszonego/Gwarancja wykonania (semantyka błędu)]].

---
## Zagadnienia realizacyjne
<sub>slajdy–merged.pdf, slajd 29</sub>

Pięć problemów do rozwiązania przy budowie mechanizmu RPC:
1. **Przetwarzanie interfejsu**
2. **Wiązanie klienta z serwerem**
3. **Obsługa komunikacji klient-serwer**
4. **Realizacja semantyki błędu**
5. **Problem osieroconych obliczeń**

### Przetwarzanie interfejsu
<sub>slajd 30</sub>

Z opisu interfejsu generuje się: **namiastkę klienta**, **namiastkę serwera**, **przykładowy program klienta** (_client sample_), **wzorzec do implementacji procedur zdalnych** (_template_) oraz **pliki do zarządzania kompilacją**. Konkretną realizację — narzędzie `rpcgen` — opisuje [[NPR 02 Sun RPC i standard XDR]].

### Wiązanie klienta z serwerem
<sub>slajdy 31–32</sub>

| Rodzaj | Definicja |
|---|---|
| **Wiązanie statyczne** | klient ma wprowadzony **na stałe** identyfikator komunikacyjny serwera (np. para: adres IP, nr portu) |
| **Wiązanie dynamiczne** | klient uzyskuje adres serwera za pośrednictwem **łącznika** (np. `portmap` lub `rpcbind` w Sun RPC) |

![[npr-rpc-s32-wiazanie-dynamiczne.png]]
<sub>Serwer rejestruje się w łączniku; klient pyta łącznik o identyfikator komunikacyjny, dostaje go i dopiero wtedy woła procedurę. slajdy–merged.pdf, slajd 32</sub>

Wiązanie dynamiczne jest warunkiem **przezroczystości położenia**: serwer może zmienić maszynę lub port, a klient nadal go znajdzie.

---
## Obsługa komunikacji klient-serwer
<sub>slajdy–merged.pdf, slajd 33</sub>

Trzy warstwy protokołu RPC, każda z osobnym zadaniem:

| Warstwa | Zadanie |
|---|---|
| **BLAST** | realizuje przesyłanie **dużych komunikatów** poprzez podział na mniejsze części, transmisję poszczególnych części i ponowne złożenie w jeden komunikat po stronie odbiorczej |
| **CHAN** | **synchronizuje** wymianę komunikatów z żądaniami wywołania procedur oraz odpowiedziami |
| **SELECT** | **rozdziela i przekazuje** komunikaty z żądaniami do odpowiednich procesów |

### BLAST
<sub>slajd 34</sub>

Nadawca dzieli komunikat na fragmenty i wysyła je kolejno; **ostatni fragment jest specjalnie oznaczony**. Odbiorca po otrzymaniu ostatniego fragmentu sprawdza kompletność i — jeśli czegoś brakuje — odsyła komunikat **SRR** z numerami brakujących fragmentów. Nadawca retransmituje wyłącznie te fragmenty.

![[npr-rpc-s34-blast.png]]
<sub>Fragmenty 3 i 5 zaginęły; po ostatnim fragmencie odbiorca wysyła SRR, dostaje brakujące fragmenty i potwierdza kolejnym SRR. slajdy–merged.pdf, slajd 34</sub>

> [!note] Uzupełnienie z Opracowania
> Slajd nie opisuje obsługi przypadku, w którym **ostatni fragment nie dotarł** ani polityki retransmisji SRR. Opracowanie uzupełnia: odbiorca uruchamia wtedy licznik czasu **RETRY** i wysyła SRR z brakującymi fragmentami; po każdym upłynięciu RETRY SRR jest wysyłany ponownie, ale **tylko dwa razy** — po trzecim upłynięciu odbiorca **zwalnia zgromadzone fragmenty** z pamięci. <sub>Opracowanie.pdf, s. 14–15</sub>

### CHAN — cztery warianty
<sub>slajdy 35–38</sub>

Warstwa CHAN rozwiązuje dwa naraz problemy: **synchronizację** żądań z odpowiedziami i **wykrycie, że druga strona jeszcze żyje**. Cztery warianty różnią się liczbą komunikatów sterujących.

**I. Wariant podstawowy** — każdy komunikat jest jawnie potwierdzany: żądanie → ACK → odpowiedź → ACK. Poprawny, ale generuje **zbyt wiele komunikatów**.

![[npr-rpc-s35-chan-podstawowy.png]]
<sub>CHAN, wariant podstawowy. slajdy–merged.pdf, slajd 35</sub>

**II. Domniemane potwierdzenia** — skoro odpowiedź sama dowodzi, że żądanie dotarło, ACK na żądanie jest zbędny. Serwer pomija potwierdzenie odbioru żądania; **kolejne żądanie klienta domyślnie potwierdza poprzednią odpowiedź**. Działa dobrze, gdy **czas realizacji żądania jest krótki** — klient zakłada, że odpowiedź przyjdzie w określonym czasie. Ostatnią wymianę klient domyka jawnym ACK.

![[npr-rpc-s36-chan-domniemane-potwierdzenia.png]]
<sub>CHAN — domniemane potwierdzenia. slajdy–merged.pdf, slajd 36</sub>

**III. Próbkowanie serwera** — gdy realizacja trwa długo, klient nie wie, czy serwer jeszcze pracuje, czy uległ awarii (albo czy nie nastąpiło partycjonowanie sieci); bez żadnego mechanizmu sprawdzającego czekałby **w nieskończoność**. Klient okresowo wysyła **PING**, a żywy serwer odpowiada **PONG**.

![[npr-rpc-s37-chan-probkowanie.png]]
<sub>CHAN — próbkowanie serwera. slajdy–merged.pdf, slajd 37</sub>

**IV. „Bicie serca"** — sytuacja odwrotna: to **serwer** z własnej inicjatywy wysyła okresowo komunikat „jeszcze żyję", uprzedzając przeterminowanie po stronie klienta, aż w końcu prześle właściwą odpowiedź.

![[npr-rpc-s38-chan-bicie-serca.png]]
<sub>CHAN — bicie serca. slajdy–merged.pdf, slajd 38</sub>

> [!note] Uzupełnienie z Opracowania
> Slajdy pokazują warianty III i IV jako diagramy bez komentarza. Opracowanie podaje ich sens: **oba** pozwalają wykryć **osierocenie obliczeń** — z jednej albo z drugiej strony, bo klient też może przestać istnieć, podczas gdy serwer nadal pracuje. <sub>Opracowanie.pdf, s. 16–17</sub>

### SELECT
<sub>slajd 42</sub>

| Strona | Zadanie |
|---|---|
| **klient** | odwzorowanie wywoływanej procedury na jej **identyfikator**, przekazywany do serwera |
| **serwer** | **zlokalizowanie** wywoływanej procedury na podstawie identyfikatora |

---
## Realizacja semantyki błędu
<sub>slajdy–merged.pdf, slajd 39</sub>

Pięć klas sytuacji błędnych i reakcje na nie:

| Sytuacja | Reakcja |
|---|---|
| **nie można zlokalizować serwera** | zgłoszenie wyjątku |
| **zaginione żądanie** | retransmisja żądania po upłynięciu ustalonego czasu oczekiwania |
| **zaginiona odpowiedź** | retransmisja żądania — a po stronie serwera: **stosowanie procedur idempotentnych** albo **numerowanie żądań i retransmisja odpowiedzi** |
| **awaria serwera** | **przed** podjęciem realizacji → retransmisja żądania; **po** wykonaniu → zgłoszenie wyjątku |
| **awaria klienta** | **osierocenie obliczeń** |

Istota problemu „zaginionej odpowiedzi": **z perspektywy klienta zaginiona odpowiedź i zaginione żądanie wyglądają identycznie** — w obu przypadkach upływa czas oczekiwania. Klient retransmituje, więc serwer musi umieć rozpoznać, czy to nowe żądanie, czy powtórzenie. Dla procedur nieidempotentnych jedynym wyjściem jest numerowanie żądań i logowanie odpowiedzi na potrzebę retransmisji.

> [!note] Uzupełnienie z Opracowania
> Slajd nie rozstrzyga przypadku awarii serwera **w trakcie** wykonania. Opracowanie: traktuje się go jak awarię po wykonaniu — zgłoszenie wyjątku — z zastrzeżeniem wykładowcy, że **nie wiadomo, co odbiorca wyjątku miałby z nim zrobić**, i że właściwym rozwiązaniem byłoby odtworzenie stanu żądania. <sub>Opracowanie.pdf, s. 18</sub>

---
## Usuwanie osieroconych obliczeń
<sub>slajdy–merged.pdf, slajdy 40–41</sub>

**Obliczenie osierocone** to obliczenie wykonywane przez serwer na rzecz klienta, który przestał istnieć. Zajmuje zasoby, a jego wynik nie ma już adresata. Cztery metody usuwania:

| Metoda | Na czym polega |
|---|---|
| **Eksterminacja** | rejestrowanie działań podejmowanych przez klienta na **nośniku niewrażliwym na awarie** i usuwanie na tej podstawie osieroconych obliczeń po restarcie klienta |
| **Reinkarnacja** | każdy restart klienta rozpoczyna **nową epokę** (identyfikowaną przez numer kolejny), po której usuwane są **wszystkie** obliczenia związane z poprzednią epoką |
| **Łagodna reinkarnacja** | reinkarnacja, w której usuwa się **tylko te** obliczenia rozpoczęte w starej epoce, **dla których nie ma właściciela** |
| **Wygaśnięcie** | przydział określonego czasu $T$ serwerowi na wykonanie procedury; jeśli wykonanie nie zakończy się w czasie $T$, serwer musi uzyskać **kolejny przydział**, pod warunkiem że obliczenia nie zostały osierocone. Jeśli klient odczeka czas $T$ przy restarcie, osierocone obliczenia **same się zakończą** |

Koszt rośnie od dołu tabeli w górę: wygaśnięcie nie wymaga żadnego trwałego stanu, reinkarnacja wymaga przechowywania numeru epoki na nośniku niewrażliwym na awarie, eksterminacja — pełnego dziennika działań klienta.

> [!note] Uzupełnienie z Opracowania
> Wykładowca zwrócił szczególną uwagę na **wygaśnięcie** jako rozwiązanie idealnie współgrające z wariantem CHAN opartym na **próbkowaniu**: ten sam licznik czasu obsługuje wykrywanie awarii i wygaszanie sierot. Opracowanie odnotowuje też, że numery epok muszą być przechowywane na nośniku niewrażliwym na awarie. <sub>Opracowanie.pdf, s. 19</sub>

---
## Warianty użycia RPC
<sub>slajdy–merged.pdf, slajdy 43–46</sub>

| Wariant | Definicja |
|---|---|
| **Wywołanie asynchroniczne** | klient **nie czeka** na wynik wykonania procedury zdalnej — wykonywanie procedury zdalnej odbywa się **równolegle** z przetwarzaniem po stronie klienta |
| **Wywołanie zwrotne** (ang. _callback_) | klient **udostępnia procedurę zdalną po swojej stronie** i przekazuje serwerowi informacje umożliwiające jej wywołanie — następuje **zamiana ról** pomiędzy klientem a serwerem |

**Wywołanie asynchroniczne** — klient wraca do pracy natychmiast po wysłaniu żądania i nigdy nie dostaje wyniku:

![[npr-rpc-s44-wywolanie-asynchroniczne.png]]
<sub>slajdy–merged.pdf, slajd 44</sub>

**Wywołanie asynchroniczne z potwierdzeniem odbioru** — klient wraca do pracy, ale dostaje krótkie potwierdzenie, że żądanie dotarło; to minimum potrzebne, by w ogóle mówić o jakiejkolwiek gwarancji:

![[npr-rpc-s45-asynchroniczne-z-potwierdzeniem.png]]
<sub>slajdy–merged.pdf, slajd 45</sub>

**Wywołanie zwrotne** — serwer w trakcie realizacji woła procedurę udostępnioną przez klienta; wynik wraca kanałem odwrotnym do pierwotnego:

![[npr-rpc-s46-wywolanie-zwrotne.png]]
<sub>slajdy–merged.pdf, slajd 46</sub>

Wywołanie zwrotne jest mechanizmem, który pozwala zbudować na RPC komunikację **push** — serwer przestaje być stroną wyłącznie reaktywną.

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 16–46
> - **Porównanie RPC z innymi paradygmatami interakcji** (komunikaty, obiekty zdalne, pamięć współdzielona, przestrzeń krotek) — zagadnienie 8 wymaga charakterystyki porównawczej; tabela zebrana jest w [[NPR 08 Podejścia do budowy systemów rozproszonych|skompresowane/NPR 08]] na podstawie wszystkich notatek tego katalogu.
> - **Przezroczystość inna niż dostępu** — slajd 20 wymienia wyłącznie przezroczystość dostępu. Pozostałe rodzaje (położenia, migracji, relokacji, replikacji, współbieżności, awarii, trwałości) w prezentacjach tego przedmiotu nie występują; częściowo pokrywa je [[Narzędzia Przetwarzania Rozproszonego/Przezroczystość]] i notatki z [[RSO 07 Model środowiska przetwarzania]].
> - **Protokół SELECT** jest opisany dwoma zdaniami (slajd 42) — brak omówienia, jak identyfikatory procedur są nadawane i czy są globalnie unikalne. Częściową odpowiedź daje trójwymiarowa identyfikacja w [[NPR 02 Sun RPC i standard XDR]].
> - **Brak analizy kosztu** (liczby komunikatów) poszczególnych wariantów CHAN — slajdy pokazują diagramy, ale nie zestawiają ich liczbowo.
> - **Brak omówienia relacji RPC do warstwy transportowej.** Opracowanie odnotowuje uwagę wykładowcy, że w praktyce RPC osadza się na gotowych protokołach transportowych, które rozwiązują znaczną część opisanych problemów, więc realne implementacje **odbiegają od modelu z wykładu**; problemy te pojawiają się w pełni dopiero przy projektowaniu RPC od zera. <sub>Opracowanie.pdf, s. 20</sub>
