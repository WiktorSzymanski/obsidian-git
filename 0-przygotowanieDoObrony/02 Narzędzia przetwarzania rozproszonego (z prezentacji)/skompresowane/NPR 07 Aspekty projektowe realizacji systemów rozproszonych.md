---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 7
---
# 7. Aspekty projektowe realizacji systemów rozproszonych
---
> Komplet pojęć zagadnienia w zakresie, w jakim pokrywają je prezentacje. Mechanizmy są nazwane, ale ich przebieg pozostaje w [[NPR 01 Zdalne wywoływanie procedur (RPC)|NPR 01]] i [[NPR 03 NFS, idempotentność i bezstanowość|NPR 03]].

## Cztery zagadnienia projektowe

Projektując mechanizm zdalnego dostępu, trzeba rozstrzygnąć cztery rzeczy. **Przezroczystość dostępu** to ukrycie komunikacji sieciowej przed aplikacją przez opakowanie funkcji komunikacyjnych namiastkami. **Gwarancja wykonania** to ukrywanie błędów komunikacyjnych — pytanie, co klient wie o wykonaniu operacji, gdy coś poszło nie tak. **Specyfikacja interfejsu** to sposób opisu sygnatur procedur zdalnych: ich nazw i typów parametrów. Czwarte to **obsługa sytuacji wyjątkowych**.

Te cztery pozycje porządkują cały przedmiot. Każde z narzędzi omawianych w [[NPR 08 Podejścia do budowy systemów rozproszonych|zagadnieniu 8]] jest w istocie innym zestawem odpowiedzi na nie.

## Przezroczystość dostępu i namiastki

Przezroczystość dostępu realizuje się przez parę **namiastek**. **Namiastka klienta** (*client stub*) udostępnia aplikacji klienckiej procedurę lokalną o takiej samej sygnaturze co procedura zdalna, odpowiedzialną za przesłanie danych do serwera i odebranie wyników. **Namiastka serwera** (*server stub*) udostępnia po stronie serwera procedurę lokalną odbierającą identyfikator procedury zdalnej i jej parametry, a odsyłającą wyniki lub zgłaszającą wyjątki. Dzięki temu kod aplikacyjny wygląda tak samo, jak gdyby wywołanie było lokalne.

Namiastek nie pisze się ręcznie — generuje je narzędzie z opisu interfejsu. Generator wytwarza także szkielet procedur zdalnych, przykładowy program klienta i pliki do zarządzania kompilacją. W Javie namiastki są generowane dynamicznie w czasie wykonania.

## Przekazywanie parametrów i konwersja danych

Parametry można przekazywać **przez wartość**, **przez referencję** albo **przez kopiowanie i odtwarzanie**. Każdy ze sposobów ma swój problem: przy wartości — różnice w reprezentacji danych między architekturami; przy referencji — wskaźnik jest liczbą bez sensu w innej przestrzeni adresowej; przy kopiowaniu i odtwarzaniu — **problem przetaczania**, czyli konieczność opisania struktury danych tak, by dało się zidentyfikować wszystkie jej składowe, obejść graf obiektów i odtworzyć go po drugiej stronie.

Różnice w reprezentacji usuwa konwersja. **Reprezentacja kanoniczna** wprowadza jeden wspólny format pośredni, do którego i z którego konwertuje każdy uczestnik — kosztem dwukrotnej konwersji przy każdej wymianie i możliwej utraty dokładności. **Konwersja bezpośrednia** tłumaczy wprost z formatu nadawcy na format odbiorcy — jedna konwersja zamiast dwóch, ale liczba potrzebnych procedur rośnie kwadratowo z liczbą architektur, a każdy musi rozpoznać architekturę partnera.

## Gwarancja wykonania: cztery semantyki błędu

Najważniejsze pojęcie zagadnienia. **Semantyka co najmniej raz** znaczy, że po uzyskaniu odpowiedzi klient ma pewność, że procedura wykonała się co najmniej raz. **Semantyka co najwyżej raz** — że po uzyskaniu odpowiedzi klient wie, że wykonała się dokładnie raz. **Semantyka dokładnie raz** jest **niemożliwa do uzyskania**, jeśli system narażony jest na awarie serwera lub łączy. **Semantyka ewentualnie** nie daje żadnej gwarancji — procedura mogła się wykonać lub nie.

Kluczowe jest to, że obie realizowalne semantyki mówią coś **wyłącznie pod warunkiem, że odpowiedź dotarła**. Gdy nie dotarła, klient nie wie nic. Realnie stosuje się tylko te dwie; „dokładnie raz" i „ewentualnie" nie są implementowane.

## Sytuacje błędne i reakcje na nie

Gdy **nie można zlokalizować serwera**, jedyną reakcją jest zgłoszenie wyjątku. **Zaginione żądanie** retransmituje się po upłynięciu czasu oczekiwania. **Zaginiona odpowiedź** wygląda z perspektywy klienta identycznie jak zaginione żądanie, więc również prowadzi do retransmisji — a serwer musi wtedy albo mieć procedury **idempotentne**, albo **numerować żądania i retransmitować odpowiedzi**. Przy **awarii serwera** przed podjęciem realizacji wystarczy retransmisja, a po wykonaniu pozostaje zgłoszenie wyjątku. **Awaria klienta** prowadzi do **osierocenia obliczeń**.

Obliczenia osierocone usuwa się czterema metodami. **Eksterminacja** rejestruje działania klienta na nośniku niewrażliwym na awarie i usuwa sieroty po restarcie. **Reinkarnacja** nadaje każdemu restartowi klienta numer nowej epoki i kasuje wszystko z epok poprzednich. **Łagodna reinkarnacja** kasuje z poprzedniej epoki tylko to, co nie ma właściciela. **Wygaśnięcie** przydziela serwerowi czas $T$ na wykonanie procedury, po którym musi on uzyskać kolejny przydział — a jeśli klient odczeka $T$ przy restarcie, sieroty zakończą się same. Koszt rośnie od wygaśnięcia, niewymagającego żadnego trwałego stanu, po eksterminację, wymagającą pełnego dziennika.

## Wiązanie

**Wiązanie statyczne** oznacza, że klient ma na stałe wpisany identyfikator komunikacyjny serwera. **Wiązanie dynamiczne** — że uzyskuje go za pośrednictwem **łącznika**, u którego serwer wcześniej się zarejestrował. Wiązanie dynamiczne jest warunkiem przezroczystości położenia: serwer może zmienić maszynę lub port, a klient nadal go znajdzie. Łącznik występuje we wszystkich omawianych narzędziach pod różnymi nazwami — `portmap`/`rpcbind`, `rmiregistry`, JNDI.

## Stan, idempotentność i bezstanowość

Stan systemu trzeba rozpatrywać na dwóch poziomach. **Stan postrzegany** (*observable state*) to stan widziany z perspektywy klienta, poprzez odpowiedzi serwera. **Stan integralny** (*inherent state*) to stan po stronie serwera, współtworzony przez niezależne zasoby. Rozróżnienie jest potrzebne, żeby móc powiedzieć, że coś jest bezstanowe **dla klienta**, mimo że serwer w środku oczywiście ma stan.

Operacja zdalna jest **idempotentna w sensie postrzegania**, jeśli jej wywołanie z określonymi wartościami parametrów zawsze daje ten sam efekt — efekt jest jednoznacznie zdeterminowany wartościami parametrów. Jest **idempotentna w sensie integralnym**, jeśli wynik jest zdeterminowany wartościami parametrów **oraz stanem niezależnych zasobów**. Wariant postrzegany jest mocniejszy.

Serwer jest **bezstanowy** — w sensie postrzegania albo integralnym — jeśli **wszystkie** jego operacje są idempotentne w odpowiednim sensie. Bezstanowość jest więc zdefiniowana przez idempotentność, a nie odwrotnie, i wystarczy **jedna** operacja nieidempotentna, by ją zniszczyć.

Praktycznym wzorcem jest NFS. Interfejs uniksowy nie jest idempotentny: `read(fd, …)` czyta od bieżącej pozycji, więc drugie wywołanie z tymi samymi parametrami zwróci co innego. Interfejs NFS został przeprojektowany tak, by nim był: pozycja jest **parametrem** (`read(fh, offset, count)`), deskryptor zastąpiono samoopisującym się uchwytem, ścieżki rozwija klient krok po kroku (`lookup`), a `open`, `close` i `lseek` **usunięto z interfejsu zdalnego** — kursor plikowy i tablica otwartych plików żyją po stronie klienta. To idempotentność pozwala bezpiecznie retransmitować żądanie przy zaginionej odpowiedzi.

---
## Czego w prezentacjach nie ma

> [!todo] Zakres zagadnienia wykracza poza slajdy
> Lista egzaminacyjna obejmuje także **rodzaje przezroczystości inne niż dostępu** (położenia, migracji, relokacji, replikacji, współbieżności, awarii, trwałości), **heterogeniczność**, **bezpieczeństwo**, **skalowalność**, **otwartość** oraz **organizację oprogramowania pośredniczącego** (*middleware*). Prezentacje tego przedmiotu wymieniają wyłącznie przezroczystość dostępu. Materiał uzupełniający: [[Narzędzia Przetwarzania Rozproszonego/Przezroczystość]], [[Narzędzia Przetwarzania Rozproszonego/Podstawowe Własności Systemu Rozproszonego]], [[RSO 07 Model środowiska przetwarzania]] (cechy i cele systemu rozproszonego, przezroczystość, otwartość, skalowalność).
