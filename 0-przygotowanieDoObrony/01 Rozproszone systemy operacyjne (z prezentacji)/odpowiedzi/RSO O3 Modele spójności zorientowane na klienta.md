---
tags:
  - obrona
  - odpowiedź
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 3
źródło: "[[RSO Z3 Modele spójności zorientowane na klienta]]"
---
# 3. Modele spójności zorientowane na klienta
---
> Wypowiedź na obronę. Sekcja **Odpowiedź** to wersja na 2–3 minuty; **Rozwinięcia** to dalsze ciągi tej samej wypowiedzi — każde odpowiada jednemu pogrubionemu hasłu i można je wpleść w to miejsce. Całość czytana ciągiem to ok. 7 minut. Warunki formalne: [[RSO Z3 Modele spójności zorientowane na klienta]].

## Odpowiedź

Modele zorientowane na klienta, nazywane też **gwarancjami sesji**, odwracają perspektywę w stosunku do modeli danocentrycznych. Te pierwsze opisują, co widzą wszystkie procesy naraz, i zakładają, że proces jest na stałe związany ze swoją repliką. Modele zorientowane na klienta nie mówią **nic o globalnym stanie replik** — gwarantują natomiast, że pojedynczy klient w ramach swojej sesji zobaczy dane zgodne z własną historią interakcji, nawet gdy przełącza się między serwerami.

**Motywacją są systemy ze spójnością ostateczną i klienci mobilni**. Klient łączy się kolejno z różnymi replikami, z których każda może być nieaktualna. Bez gwarancji sesji widziałby dane cofające się w czasie albo tracił własne zmiany — zmienił hasło, odświeżył stronę, trafił na inny serwer i widzi stare. Pomysł polega na tym, żeby zamiast synchronizować cały system, wymusić warunek **tylko na tym serwerze, który obsługuje bieżące żądanie**.

Aparat jest bardzo prosty. Każdy zapis ma globalnie unikalny identyfikator, a sesja klienta utrzymuje dwa zbiory: **write-set**, czyli identyfikatory zapisów wykonanych przez tego klienta, i **read-set**, czyli identyfikatory zapisów, które wpłynęły na wyniki jego odczytów. Cały mechanizm sprowadza się do jednego schematu: przed obsłużeniem żądania serwer sprawdza, czy zawiera już odpowiedni zbiór sesji, a po obsłużeniu żądania sesja jest aktualizowana.

Na tym aparacie definiuje się **cztery gwarancje**. **Read Your Writes** — klient zawsze widzi własne zapisy. **Monotonic Reads** — kolejne odczyty zwracają stan co najmniej tak samo aktualny, nigdy starszy. **Monotonic Writes** — zapisy tego samego klienta są na każdym serwerze wykonywane w kolejności ich zgłoszenia. I **Writes Follow Reads** — zapis trafia na każdym serwerze za tymi zapisami, które wpłynęły na poprzedzający go odczyt.

W tym jest ładna regularność: **RYW i MW pilnują write-setu, MR i WFR read-setu; RYW i MR sprawdza się przy odczycie, MW i WFR przy zapisie**. Wszystkie cztery są niezależne i można je dowolnie łączyć, ale trzeba od razu powiedzieć, że nawet komplet nie daje spójności sekwencyjnej — one dotyczą jednej sesji, a nie relacji między wieloma klientami.

W praktyce nie porównuje się zbiorów identyfikatorów, tylko **wektory wersji**, bo zbiory rosłyby bez ograniczeń. To są dokładnie te mechanizmy, które dziś spotyka się jako read-after-write w DynamoDB czy sesje przyczynowe w MongoDB.

## Rozwinięcia

### Skąd się wzięła osobna rodzina modeli

Historycznie gwarancje sesji sformułowali Terry i współpracownicy w systemie **Bayou**, w połowie lat dziewięćdziesiątych — a więc w kontekście replikacji leniwej, rozłączanych klientów i propagacji zmian przez anty-entropię. Założenie jest takie, że dane są replikowane na wielu serwerach, replikacja jest leniwa, czyli mamy spójność ostateczną, i że klient nie jest przywiązany do jednej repliki. W takim świecie modele danocentryczne albo są za drogie, albo nie mówią nic użytecznego, a to, co klienta realnie boli, to niespójność jego własnego doświadczenia. Stąd pomysł, żeby gwarancje sformułować **względem sesji**, a nie względem systemu.

### Write-set, read-set i warunek na serwerze

Każdy zapis ma identyfikator, w praktyce parę: serwer plus znacznik czasu. Przez zbiór zapisów zastosowanych już na danym serwerze oznacza się jego bazę. Sesja klienta trzyma **write-set** i **read-set**, a warunek dopuszczenia operacji to zawsze zawieranie jednego z tych zbiorów w bazie serwera. Potrzebne jest jeszcze jedno pojęcie pomocnicze — **zapisy istotne dla odczytu**, czyli te, które wpłynęły na jego wynik; to one po odczycie trafiają do read-setu.

### Cztery gwarancje i co je łamie

**Read Your Writes** mówi, że jeżeli klient wykonał zapis, to każdy jego późniejszy odczyt obsługuje serwer, który ten zapis już uwzględnia; warunek to zawieranie write-setu w bazie serwera. Złamanie tego widać, gdy po zmianie hasła albo edycji profilu odświeżenie strony trafia na inny serwer i pokazuje starą wartość. **Monotonic Reads** mówi, że jeżeli klient odczytał coś w pewnym stanie, to kolejne odczyty zwracają stan co najmniej tak samo aktualny; warunek dotyczy read-setu, a po odczycie read-set powiększa się o zapisy istotne. Złamanie: wiadomość widoczna w skrzynce znika po przełączeniu na inny serwer. **Monotonic Writes** mówi, że jeżeli klient wykonał najpierw jeden zapis, a potem drugi, to na każdym serwerze drugi zostanie wykonany po pierwszym — inaczej aktualizacja dokumentu mogłaby zostać zastosowana, zanim powstała jego wersja pierwsza. I **Writes Follow Reads**: jeżeli klient coś odczytał, a potem zapisał, to jego zapis jest na każdym serwerze umieszczony po wszystkich zapisach, które wpłynęły na ten odczyt — inaczej odpowiedź na forum pojawiłaby się na serwerze, na którym oryginalnego postu jeszcze nie ma.

### Realizacja wektorami wersji

Naiwna realizacja, w której klient przechowuje zbiory identyfikatorów, a serwer sprawdza zawieranie, ma oczywistą wadę: te zbiory rosną nieograniczenie. Dlatego w praktyce, już w Bayou, używa się **wektorów wersji**. Serwer trzyma wektor, którego składowa dla serwera $k$ to największy znacznik zapisu pochodzącego z tego serwera i zastosowanego lokalnie — działa to dlatego, że zapisy z danego serwera stosuje się w kolejności. Sesja reprezentuje swoje dwa zbiory analogicznymi wektorami, a warunek zawierania staje się zwykłą **dominacją po składowych**. Jeżeli serwer warunku nie spełnia, są trzy wyjścia: klient czeka, klient wybiera inny serwer, albo serwer sam pobiera brakujące zapisy od innych replik. Dodatkowo Monotonic Writes i Writes Follow Reads wymagają, żeby serwery stosowały zapisy w porządku zgodnym z zależnościami — w praktyce przyczynowym, na przykład przy użyciu znaczników Lamporta w propagacji.

### Zestawienie z modelami danocentrycznymi

Różnicę najprościej ująć tak: modele **danocentryczne** przyjmują perspektywę wszystkich procesów naraz, wiążą proces z repliką i płacą globalną synchronizacją. Modele **zorientowane na klienta** przyjmują perspektywę jednej sesji, zakładają klienta mobilnego, a płacą tylko porównaniem wektora wersji przy każdym żądaniu. To dlatego są tak popularne w systemach o spójności ostatecznej — dają użytkownikowi doświadczenie sensownej spójności za ułamek ceny modeli z [[RSO Z2 Danocentryczne modele spójności|zagadnienia 2]].
