---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 8
---
# 8. Podejścia do budowy systemów rozproszonych (charakterystyka porównawcza)
---
> Zestawienie sześciu podejść omówionych w prezentacjach. Mechanizmy są tu nazwane i scharakteryzowane; ich szczegóły — w [[NPR 01 Zdalne wywoływanie procedur (RPC)|NPR 01]], [[NPR 02 Sun RPC i standard XDR|NPR 02]], [[NPR 04 Podejście obiektowe - Java RMI|NPR 04]], [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione|NPR 05]] i [[NPR 08 MOM i systemy kolejkowania komunikatów|NPR 08]]–[[NPR 11 Przestrzeń krotek - Linda i JavaSpaces|NPR 11]].

## Oś sporu

Wszystkie podejścia rozwiązują ten sam problem — jak dwa procesy na różnych maszynach mają ze sobą współpracować — i różnią się odpowiedzią na jedno pytanie: **czy ukrywać fakt rozproszenia, czy go ujawnić**.

Jedna rodzina ukrywa. **Zdalne wywoływanie procedur** i **zdalne wywoływanie metod** udają, że wywołanie jest lokalne: programista pisze zwykłe wywołanie, a namiastki chowają sieć. Cena jest stała — udawanie załamuje się przy awariach, bo lokalne wywołanie nie może „nie dojść", a zdalne może. Stąd cały aparat semantyk błędu.

Druga rodzina ujawnia. **Przekazywanie komunikatów**, **systemy kolejkowania** i **przestrzeń krotek** wymagają jawnych operacji wysłania i odebrania. Programista widzi, że coś jest przesyłane, i musi się z tym liczyć — ale w zamian dostaje rozłączenie stron w czasie i przestrzeni, którego pierwsza rodzina nie potrafi dać.

Osobno stoją **spotkania Ady**, które ujawniają rozproszenie, ale nie w celu rozłączenia stron — przeciwnie, w celu ich ścisłej synchronizacji.

## Sprzężenie w czasie i przestrzeni

Dwa wymiary, po których podejścia się różnią najostrzej.

**Sprzężenie w przestrzeni** to pytanie, czy nadawca musi znać odbiorcę. W RPC i RMI musi — potrzebuje jego identyfikatora komunikacyjnego, choćby uzyskanego z łącznika. W systemach kolejkowania nie musi: zna tylko nazwę **kolejki** albo **tematu**, a kto po drugiej stronie odbierze, jest jego sprawą. W przestrzeni krotek nie ma nawet nazwy skrzynki — jest **opis treści**, czyli identyfikacja **asocjacyjna**.

**Sprzężenie w czasie** to pytanie, czy obie strony muszą działać jednocześnie. W RPC, RMI i w spotkaniach Ady muszą. W systemach kolejkowania i w przestrzeni krotek nie muszą — komunikat przeżywa nieobecność obu stron, co nazywa się **komunikacją nieustanną** (w odróżnieniu od **przejściowej**, w której komunikat jest utrzymywany tylko dopóki żyją nadawca i odbiorca).

Im luźniejsze sprzężenie, tym trudniej wnioskować o globalnym przebiegu obliczenia — i tym łatwiej wymieniać, restartować i skalować poszczególne składniki.

## Sześć podejść

**RPC** jest zdalnym wywołaniem procedury. Interfejs opisuje się w osobnym języku (w Sun RPC — pliku `.x` przetwarzanym przez `rpcgen`), dane konwertuje kanonicznie (XDR), klienta wiąże z serwerem dynamicznie przez łącznik, a błędy interpretuje według wybranej semantyki. Jednostką jest **procedura**, wywołanie jest domyślnie synchroniczne, ale istnieją warianty asynchroniczne i wywołanie zwrotne.

**RMI** jest tym samym pomysłem przeniesionym na obiekty. Jednostką jest **obiekt zdalny**, a operacją — metoda zadeklarowana w interfejsie wywiedzionym z `Remote`. Interfejs jest opisany **w języku implementacji**, nie w osobnym IDL. Obiekty zwykłe przekazuje się **przez wartość** (wymagane `Serializable`), obiekty zdalne — **przez referencję** (wymagane `Remote`), przez co serwer dostaje namiastkę i może wołać z powrotem do klienta. Trwałość zapewnia mechanizm **obiektów aktywowalnych**, ale bez utrwalania stanu.

**Systemy kolejkowania komunikatów (MOM)** wprowadzają pośrednika. Nadawca umieszcza komunikat w **kolejce** (punkt-punkt, jeden konsument) albo publikuje go pod **tematem** (publish/subscribe, wielu konsumentów), a **zarządca kolejek** (broker) odpowiada za ich trwałe przechowanie. Komunikacja jest nieustanna, niezawodna i łatwo asynchroniczna; strony nie muszą się znać ani działać jednocześnie. **JMS** jest specyfikacją takiego systemu dla Javy, z transakcjami, trybami potwierdzeń, trwałością komunikatów, priorytetami, czasem życia i subskrypcjami trwałymi. **ZeroMQ** jest przypadkiem skrajnym — **nie ma brokera**, kolejkowanie jest wbudowane w gniazdo, interfejs wzorowany na gniazdach BSD, a komunikat jest nieinterpretowaną sekwencją bajtów; w zamian za wydajność traci się trwałość.

**Przestrzeń krotek** (Linda, JavaSpaces) zastępuje nazwę skrzynki **dopasowaniem treści**. Procesy umieszczają w niej **krotki** operacją `Output`, pobierają operacją `Input` i odczytują bez pobierania operacją `Read`; atrybuty krotki mogą pozostać bez wartości i wtedy pasują do czegokolwiek. Przestrzeń jest **zbiorem, nie kolejką** — nie ma uporządkowania, jest jedynie gwarancja żywotności. Operacje blokujące czynią z niej także mechanizm synchronizacji.

**Spotkania asymetryczne Ady** są komunikacją **synchroniczną między parą zadań**. Zadanie czynne (klient) woła **wejście** zadania biernego (serwera), a serwer przyjmuje wywołanie instrukcją `accept`; ta ze stron, która dotrze pierwsza, czeka. Nie ma tu żadnego pośrednika i żadnego buforowania — komunikacja i synchronizacja są **tym samym zdarzeniem**.

**Obiekty chronione Ady** to komunikacja **asynchroniczna między wieloma zadaniami** przez dane współdzielone, z wzajemnym wykluczaniem wbudowanym w język.

## Tabela porównawcza

| | **RPC** | **RMI** | **MOM / JMS** | **ZeroMQ** | **Przestrzeń krotek** | **Spotkania Ady** |
|---|---|---|---|---|---|---|
| **Jednostka** | procedura | obiekt (metoda) | komunikat | komunikat (bajty) | krotka | wejście (`entry`) |
| **Pośrednik** | brak (łącznik tylko do wiązania) | brak (rejestr tylko do wiązania) | **broker** (zarządca kolejek) | **brak** | **przestrzeń** | brak |
| **Identyfikacja odbiorcy** | adres serwera + nr programu/wersji/procedury | nazwa w rejestrze + interfejs | **nazwa kolejki lub tematu** | adres gniazda + typ | **asocjacyjna — przez treść** | **nazwa zadania + nazwa wejścia** |
| **Sprzężenie w przestrzeni** | ścisłe | ścisłe | **luźne** | średnie | **najluźniejsze** | ścisłe |
| **Sprzężenie w czasie** | ścisłe | ścisłe | **brak** (nieustanna) | ścisłe | **brak** (nieustanna) | **najściślejsze** (spotkanie) |
| **Synchroniczność** | synchroniczne (warianty async, callback) | synchroniczne | asynchroniczna; odbiór synchroniczny (`receive`) lub asynchroniczny (`onMessage`) | asynchroniczna | blokująca (`Input`/`Read`), warianty `Try_*` | **synchroniczne w obie strony** |
| **Liczba odbiorców** | 1 | 1 | 1 (kolejka) / wielu (temat) | wg typu gniazda (REQ-REP, PUB-SUB, PUSH-PULL) | 1 (`Input`) / wielu (`Read`) | 1 |
| **Opis interfejsu** | osobny język (`.x`, `rpcgen`) | **język implementacji** (`Remote`) | brak — liczy się treść | brak | brak — liczy się kształt krotki | **język implementacji** |
| **Reprezentacja danych** | kanoniczna (XDR) | serializacja Javy | typy wiadomości (`Text`, `Map`, `Bytes`, `Stream`, `Object`) | **nieinterpretowane bajty** | obiekty `Entry` (pola publiczne) | typy języka |
| **Semantyka błędu** | do wyboru: *co najmniej raz* / *co najwyżej raz* | *co najwyżej raz* (`RemoteException`) | tryby potwierdzeń i transakcje | brak gwarancji | żywotność („w końcu") | wyjątek w obu zadaniach |
| **Trwałość** | brak | brak (obiekty aktywowalne — bez stanu) | **`PERSISTENT` — dysk** | **brak** | **dzierżawa** (*lease*) | brak |
| **Przezroczystość dostępu** | **pełna** (namiastki) | **pełna** (stub/szkielet) | brak — jawne `send`/`receive` | brak | brak | brak — jawna składnia wejścia |

## Czego nie da się uzyskać naraz

Z tabeli wynika prawidłowość, którą warto umieć nazwać. **Przezroczystość dostępu i luźne sprzężenie wykluczają się.** Podejścia, które najlepiej ukrywają sieć (RPC, RMI), najmocniej wiążą strony w czasie i przestrzeni — bo żeby udawać wywołanie lokalne, trzeba mieć konkretnego, działającego rozmówcę. Podejścia, które strony rozłączają (MOM, przestrzeń krotek), muszą ujawnić, że komunikat jest gdzieś przekazywany.

Podobnie **trwałość kosztuje wydajność**: `PERSISTENT` w JMS oznacza zapis na dysk przy każdej wiadomości, a ZeroMQ jest szybkie dokładnie dlatego, że nie zapisuje nic.

## Szczegóły w notatkach

- [[NPR 01 Zdalne wywoływanie procedur (RPC)]] — zagadnienia projektowe i realizacyjne, semantyki błędu, BLAST/CHAN/SELECT, osierocone obliczenia
- [[NPR 02 Sun RPC i standard XDR]] — `rpcgen`, plik `.x`, potoki i filtry XDR
- [[NPR 03 NFS, idempotentność i bezstanowość]] — projektowanie interfejsu zdalnego pod bezstanowość
- [[NPR 04 Podejście obiektowe - Java RMI]] — architektura, rejestr, obiekty aktywowalne
- [[NPR 05 Ada 95 - zadania, spotkania i obiekty chronione]] — rendez-vous, `select`, obiekty chronione
- [[NPR 08 MOM i systemy kolejkowania komunikatów]] — cechy MOM, oba paradygmaty, model systemu
- [[NPR 09 ZeroMQ]] — kontekst, gniazda, schematy komunikacji
- [[NPR 10 JMS]] — model programistyczny, niezawodność, transakcje
- [[NPR 11 Przestrzeń krotek - Linda i JavaSpaces]] — krotki, dopasowanie, dzierżawa

---
## Czego w prezentacjach nie ma

> [!todo] Zakres zagadnienia wykracza poza slajdy
> Lista egzaminacyjna wymienia także **rozproszoną pamięć współdzieloną (DSM)** i **CORBA**, których prezentacje tego przedmiotu **nie omawiają wcale**. Nieobecna jest też **rozproszona Ada** (Annex E, partycje, PolyORB) — wykład o Adzie dotyczy wyłącznie współbieżności lokalnej. Modele spójności potrzebne przy DSM są w [[RSO 02 Danocentryczne modele spójności]] i [[RSO 03 Modele spójności zorientowane na klienta]]. **Tabela porównawcza powyżej nie pochodzi ze slajdów** — zestawiono ją z własności wymienionych osobno przy każdym z podejść.
