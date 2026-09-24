---
tags:
  - obrona
  - odpowiedź
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 7
źródło: "[[NPR 07 Aspekty projektowe realizacji systemów rozproszonych]]"
---
# 7. Aspekty projektowe realizacji systemów rozproszonych
---
> Wypowiedź na obronę. Sekcja **Odpowiedź** to wersja na 2–3 minuty; **Rozwinięcia** to dalsze ciągi tej samej wypowiedzi — każde odpowiada jednemu pogrubionemu hasłu i można je wpleść w to miejsce. Całość czytana ciągiem to ok. 8 minut. Wersja skompresowana: [[NPR 07 Aspekty projektowe realizacji systemów rozproszonych]].

## Odpowiedź

Zagadnienie sprowadza się do pytania: co trzeba rozstrzygnąć, projektując mechanizm zdalnego dostępu. Rzeczy są cztery. **Przezroczystość dostępu**, czyli ukrycie komunikacji sieciowej przed aplikacją. **Gwarancja wykonania**, czyli ukrywanie błędów komunikacyjnych — co klient właściwie wie o wykonaniu operacji, kiedy coś poszło nie tak. **Specyfikacja interfejsu**, czyli sposób opisu sygnatur procedur zdalnych. I **obsługa sytuacji wyjątkowych**. Te cztery pozycje porządkują cały przedmiot: każde konkretne narzędzie — RPC, RMI, kolejki komunikatów, przestrzeń krotek — jest po prostu innym zestawem odpowiedzi na te same cztery pytania.

Przezroczystość dostępu realizuje się przez parę **namiastek**. Namiastka klienta udostępnia aplikacji procedurę lokalną o takiej samej sygnaturze co procedura zdalna i to ona wysyła dane oraz odbiera wyniki; namiastka serwera odbiera identyfikator procedury i parametry, a odsyła wyniki albo wyjątki. Dzięki temu kod aplikacyjny wygląda dokładnie tak, jakby wywołanie było lokalne. Namiastek się nie pisze — **generuje** je narzędzie z opisu interfejsu.

Drugi aspekt jest najważniejszy pojęciowo, bo dotyczy **semantyki błędu**. Są cztery możliwe gwarancje: **co najmniej raz**, **co najwyżej raz**, **dokładnie raz** i **ewentualnie**. Rzecz w tym, że „dokładnie raz" jest **nieosiągalna**, jeśli system jest narażony na awarie serwera albo łączy, a „ewentualnie" nie daje żadnej gwarancji — realnie implementuje się więc tylko dwie pierwsze. I trzeba dopowiedzieć rzecz kluczową: obie mówią cokolwiek **wyłącznie pod warunkiem, że odpowiedź dotarła**. Jeżeli odpowiedzi nie ma, klient nie wie nic.

Z tego wyrasta cała reszta. Wyróżnia się **pięć sytuacji błędnych** — nie można zlokalizować serwera, zaginęło żądanie, zaginęła odpowiedź, padł serwer, padł klient — i każda ma inną reakcję. Awaria klienta jest szczególna, bo prowadzi do **osierocenia obliczeń**.

Osobnym rozstrzygnięciem jest **wiązanie**: statyczne, gdzie klient ma adres serwera wpisany na stałe, albo dynamiczne, przez **łącznik**, u którego serwer się rejestruje — i to dopiero daje przezroczystość położenia.

Wreszcie klamrą całości są **idempotentność i bezstanowość**, bo to one decydują, czy wolno bezpiecznie retransmitować żądanie. Wzorcowym przykładem jest **NFS**, którego interfejs został świadomie przeprojektowany tak, żeby był idempotentny.

## Rozwinięcia

### Namiastki i generowanie kodu

Warto dopowiedzieć, co dokładnie robi generator. Z opisu interfejsu wytwarza nie tylko obie namiastki, ale też szkielet procedur zdalnych do wypełnienia przez programistę, przykładowy program klienta oraz pliki do zarządzania kompilacją. Sens jest taki, że **jedynym miejscem, w którym opisuje się interfejs, jest ten opis** — reszta jest z niego wyprowadzana, co eliminuje klasę błędów polegających na rozjechaniu się stron. W Javie idzie się jeszcze dalej: namiastki są generowane **dynamicznie w czasie wykonania**, więc nie ma osobnego kroku kompilacji interfejsu.

### Przekazywanie parametrów i konwersja danych

Skoro namiastka ma przesłać parametry przez sieć, pojawia się pytanie, jak je przekazać, i każdy ze sposobów ma swój problem. **Przez wartość** — problemem są różnice w reprezentacji danych między architekturami. **Przez referencję** — wskaźnik to liczba, która w innej przestrzeni adresowej nie znaczy nic. **Przez kopiowanie i odtwarzanie** — tu pojawia się **przetaczanie**, czyli konieczność opisania struktury danych na tyle dokładnie, żeby dało się zidentyfikować wszystkie jej składowe, obejść graf obiektów i odtworzyć go po drugiej stronie. Same różnice reprezentacji usuwa się dwojako. **Reprezentacja kanoniczna** wprowadza jeden wspólny format pośredni — kosztem jest dwukrotna konwersja przy każdej wymianie i możliwa utrata dokładności. **Konwersja bezpośrednia** tłumaczy wprost z formatu nadawcy na format odbiorcy, czyli jedna konwersja zamiast dwóch, ale liczba potrzebnych procedur rośnie kwadratowo z liczbą architektur i każdy musi rozpoznać architekturę partnera.

### Cztery semantyki błędu

Rozwińmy je po kolei. **Co najmniej raz** znaczy, że po uzyskaniu odpowiedzi klient ma pewność, że procedura wykonała się przynajmniej jeden raz — mogła jednak wykonać się kilka razy, jeśli po drodze doszło do retransmisji. **Co najwyżej raz** znaczy, że po uzyskaniu odpowiedzi klient wie, że wykonała się dokładnie raz. **Dokładnie raz** brzmi jak to, czego naprawdę chcemy, ale jest nieosiągalne w obecności awarii serwera lub łączy — zawsze da się skonstruować scenariusz, w którym klient nie jest w stanie odróżnić „wykonało się i zginęła odpowiedź" od „nie wykonało się w ogóle". **Ewentualnie** to brak jakiejkolwiek gwarancji. Praktyczny wniosek jest taki, że projektant wybiera między dwiema pierwszymi, a wybór ten przekłada się wprost na wymagania wobec operacji: „co najmniej raz" jest tanie, ale wymaga, żeby operacje były idempotentne; „co najwyżej raz" wymaga od serwera pamiętania obsłużonych żądań.

### Pięć sytuacji błędnych

**Nie można zlokalizować serwera** — jedyną sensowną reakcją jest zgłoszenie wyjątku, bo nie ma czego ponawiać. **Zaginione żądanie** — retransmisja po upłynięciu czasu oczekiwania. **Zaginiona odpowiedź** — i tu jest sedno: z perspektywy klienta wygląda **identycznie** jak zaginione żądanie, więc również prowadzi do retransmisji; serwer musi być na to przygotowany, czyli albo mieć procedury **idempotentne**, albo **numerować żądania i retransmitować zapamiętane odpowiedzi**. **Awaria serwera** — jeżeli padł przed podjęciem realizacji, wystarczy retransmisja, ale jeżeli padł po wykonaniu, pozostaje zgłoszenie wyjątku, bo klient nie ma jak stwierdzić, co się stało. **Awaria klienta** — nie boli klienta, tylko serwer, bo zostają na nim **obliczenia osierocone**.

### Obliczenia osierocone i metody ich usuwania

Metod są cztery i różnią się ceną. **Eksterminacja** — klient rejestruje swoje działania na nośniku niewrażliwym na awarie i po restarcie usuwa sieroty na podstawie dziennika; to najdroższe, bo wymaga pełnego logowania. **Reinkarnacja** — każdy restart klienta dostaje numer nowej epoki, a wszystko z epok poprzednich jest kasowane. **Łagodna reinkarnacja** — to samo, ale kasuje się z poprzedniej epoki tylko to, co nie ma już właściciela. **Wygaśnięcie** — serwer dostaje na wykonanie procedury ograniczony czas, po którym musi wystąpić o kolejny przydział; jeżeli klient przy restarcie odczeka ten czas, sieroty zakończą się same. Koszt rośnie od wygaśnięcia, które nie wymaga żadnego trwałego stanu, po eksterminację, wymagającą dziennika.

### Wiązanie i łącznik

**Wiązanie statyczne** oznacza, że klient ma na stałe wpisany identyfikator komunikacyjny serwera — proste, ale sztywne. **Wiązanie dynamiczne** wprowadza pośrednika, **łącznik**, u którego serwer się wcześniej rejestruje i u którego klient pyta o adres. To jest warunek **przezroczystości położenia**: serwer może zmienić maszynę albo port, a klient i tak go znajdzie. Warto zauważyć, że łącznik występuje we wszystkich omawianych narzędziach, tylko pod różnymi nazwami — `portmap` czy `rpcbind` w Sun RPC, `rmiregistry` w RMI, JNDI w świecie Javy EE.

### Stan, idempotentność i bezstanowość

Stan trzeba rozpatrywać na dwóch poziomach. **Stan postrzegany** to stan widziany z perspektywy klienta, przez odpowiedzi serwera; **stan integralny** to stan po stronie serwera, współtworzony przez niezależne zasoby. Rozróżnienie jest potrzebne po to, żeby móc sensownie powiedzieć, że coś jest bezstanowe **dla klienta**, mimo że serwer w środku oczywiście stan ma. Analogicznie idempotentność ma dwa sensy: **w sensie postrzegania**, gdy efekt wywołania jest jednoznacznie zdeterminowany wartościami parametrów, i **w sensie integralnym**, gdy jest zdeterminowany parametrami oraz stanem niezależnych zasobów; ten pierwszy jest mocniejszy. Serwer jest **bezstanowy** wtedy, gdy **wszystkie** jego operacje są idempotentne w odpowiednim sensie — czyli bezstanowość definiuje się przez idempotentność, a nie odwrotnie, i wystarczy jedna operacja nieidempotentna, żeby ją zniszczyć.

### NFS jako wzorzec projektowania pod idempotentność

Najlepiej widać to na NFS. Zwykły interfejs uniksowy idempotentny nie jest: odczyt czyta od bieżącej pozycji, więc drugie wywołanie z tymi samymi parametrami zwróci co innego. Interfejs NFS przeprojektowano więc tak, żeby idempotentny był. Po pierwsze, pozycja w pliku stała się **parametrem** operacji zamiast ukrytego stanu. Po drugie, deskryptor zastąpiono **samoopisującym się uchwytem**. Po trzecie, ścieżki rozwija klient **krok po kroku**, osobnymi operacjami wyszukania. Po czwarte, operacje otwarcia, zamknięcia i przesunięcia kursora **usunięto z interfejsu zdalnego** — kursor plikowy i tablica otwartych plików żyją po stronie klienta. Efekt jest taki, że przy zaginionej odpowiedzi wolno po prostu powtórzyć żądanie, i to domyka cały łańcuch: **semantyka błędu, retransmisja i idempotentność to jedno rozstrzygnięcie projektowe widziane z trzech stron**.
