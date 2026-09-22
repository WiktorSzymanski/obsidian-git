---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 8
---
# 8. Podejścia do budowy systemów rozproszonych
---
> Podejście (paradygmat interakcji) określa, **jak jednostki przetwarzania wymieniają informacje i koordynują działanie**. Wybór podejścia wyznacza model systemu, poziom abstrakcji dla programisty, stopień powiązania komponentów oraz semantykę błędów.

Lista paradygmatów: [[Narzędzia Przetwarzania Rozproszonego/Paradygmat Interakcji Pomiędzy Zdalnymi Jednostkami]].

## Pojęcia pomocnicze
### Wymiary komunikacji
- **Synchroniczność**: nadawca blokowany do momentu (a) przekazania do warstwy komunikacyjnej, (b) odbioru przez odbiorcę, (c) otrzymania odpowiedzi – albo **asynchronicznie** kontynuuje pracę.
- **Trwałość**: **przejściowa** (_transient_) – komunikat istnieje tylko, gdy nadawca i odbiorca działają; **nieustanna** (_persistent_) – przechowywana w podsystemie komunikacyjnym do skutku ([[Narzędzia Przetwarzania Rozproszonego/Trwałość Komunikacji]]).
- Kombinacje: przejściowa asynchroniczna (UDP), przejściowa synchroniczna z potwierdzeniem odbioru / dostarczenia / z odpowiedzią (RPC), nieustanna asynchroniczna (poczta, MOM), nieustanna synchroniczna.
- **Powiązanie w przestrzeni** (czy nadawca zna odbiorcę) i **w czasie** (czy muszą działać jednocześnie).

---
## 1. Przekazywanie komunikatów (_message passing_)
**Model systemu**: procesy bez wspólnej pamięci połączone **łączami komunikacyjnymi**; operacje jawne `send(dest, msg)` / `receive(src, msg)`. Model: aplikacja → oprogramowanie systemowe → (serwer komunikacyjny) → sieć → … → odbiorca ([[model_komunikacji.excalidraw]], [[łącze_komunikacji.excalidraw]]).

**Istota**: programista sam kontroluje format, adresowanie, synchronizację i obsługę błędów. Najniższy poziom abstrakcji, największa elastyczność i wydajność.

**Warianty**: adresowanie bezpośrednie (proces) lub pośrednie (skrzynka / port), odbiór selektywny, komunikacja punkt-punkt lub grupowa ([[01 Komunikacja grupowa]]).

**Narzędzia**: gniazda BSD, **MPI** (`MPI_Send`/`MPI_Recv`, tryby: standardowy, buforowany `Bsend`, synchroniczny `Ssend`, gotowości `Rsend`; nieblokujące `Isend`/`Irecv`; kolektywne `Bcast`, `Scatter`, `Gather`, `Reduce`), PVM.

## 2. Middleware komunikatów (MOM) i publish/subscribe
**Model**: pośrednik (broker) z **kolejkami** (point-to-point) lub **tematami** (publish/subscribe). Komunikacja nieustanna i asynchroniczna.

**Istota**: luźne powiązanie w czasie i przestrzeni; dobra skalowalność i odporność na chwilową niedostępność odbiorcy; semantyki dostarczania at-most-once / at-least-once / exactly-once.

**Narzędzia**: JMS, RabbitMQ (AMQP), Apache Kafka, MQTT; bez brokera – **ZeroMQ** (gniazda z wzorcami: REQ/REP, PUB/SUB z filtrowaniem po prefiksie tematu, PUSH/PULL – potoki, DEALER/ROUTER – asynchroniczne req/rep; zob. [[Narzędzia Przetwarzania Rozproszonego/ZMQ]]).

## 3. Zdalne wywołanie procedur (RPC)
**Model systemu** (Birrell-Nelson 1984):
```
klient ── namiastek klienta (stub) ── runtime RPC ── sieć ── runtime RPC ── szkielet/stub serwera ── procedura
```
1. Klient wywołuje lokalnie procedurę-namiastek.
2. Namiastek **szereguje** (_marshalling_) parametry do wspólnej reprezentacji (XDR/NDR/Protobuf) i wysyła żądanie.
3. Stub serwera **rozpakowuje** parametry, wywołuje procedurę, szereguje wynik i odsyła.
4. Namiastek klienta rozpakowuje wynik i zwraca go klientowi.

**Istota**: **przezroczystość dostępu** – zdalne wywołanie wygląda jak lokalne ([[Narzędzia Przetwarzania Rozproszonego/Zdalne Wywoływanie Procedur]]).

**Elementy**:
- **specyfikacja interfejsu** (IDL, plik `.x` w ONC RPC, `.proto` w gRPC) i generator namiastków (`rpcgen` – [[Narzędzia Przetwarzania Rozproszonego/RPC]]),
- **wiązanie** (_binding_) – lokalizacja serwera: statyczne lub dynamiczne przez usługę katalogową (portmapper/rpcbind: program, wersja → port),
- przekazywanie parametrów: przez wartość, **copy/restore** zamiast przez referencję (brak wspólnej pamięci).

**Semantyki wykonania** (przy retransmisjach i awariach):
| Semantyka | Mechanizm | Gwarancja |
|---|---|---|
| **może** (_maybe_) | brak retransmisji | wykonanie 0 lub 1 raz, klient nie wie |
| **co najmniej raz** | retransmisje żądań | ≥ 1 wykonanie – dla operacji **idempotentnych** |
| **co najwyżej raz** | retransmisje + filtrowanie duplikatów + historia odpowiedzi | ≤ 1 wykonanie; po odpowiedzi – dokładnie raz |
| **dokładnie raz** | wymaga transakcji / trwałego logu | idealna, trudna do osiągnięcia |

Zob. [[Narzędzia Przetwarzania Rozproszonego/Gwarancja wykonania (semantyka błędu)]].
**Problemy**: awaria klienta → **sieroty** (_orphans_) na serwerze; usuwanie: eksterminacja, reinkarnacja (epoki), wygaśnięcie (_expiration_).
**Asynchroniczne RPC**: klient nie czeka (odroczone odpowiedzi, _callback_, _future_).

**Narzędzia**: ONC RPC (Sun), DCE RPC, XML-RPC, gRPC, Thrift, [[Technologie internetowe w przetwarzaniu rozproszonym/SOAP|SOAP]].

## 4. Obiekty rozproszone / zdalne wywołanie metod (RMI)
**Model**: obiekt zdalny z interfejsem; klient trzyma **referencję zdalną**, a lokalnie **pełnomocnika** (_proxy_). Po stronie serwera działa **szkielet** (_skeleton_). Rejestr (np. `rmiregistry`, CORBA Naming Service) wiąże nazwy z referencjami.

**Istota** (podejście obiektowe): hermetyzacja stanu i danych w obiekcie, dziedziczenie interfejsów, polimorfizm, **przekazywanie referencji do obiektów** jako parametrów (obiekty zdalne przez referencję, serializowalne przez wartość), rozproszone odśmiecanie (liczniki referencji / _leases_), dynamiczne wywołania (DII).

**Narzędzia**: Java RMI, CORBA (IDL, ORB, IIOP), DCOM, .NET Remoting.

## 5. Rozproszona pamięć współdzielona (DSM)
**Model**: procesy na różnych maszynach widzą **wspólną wirtualną przestrzeń adresową**; strony/obiekty migrują lub są replikowane, a chybienie strony wyzwala pobranie jej przez sieć.

**Istota**: najwyższa przezroczystość – programowanie jak na maszynie wieloprocesorowej (zmienne, blokady). Kosztem jest wydajność (_thrashing_) i konieczność wyboru **modelu spójności** pamięci ([[02 Danocentryczne modele spójności]]).

**Narzędzia**: Ivy, Munin, TreadMarks; na poziomie obiektów – Orca, JavaSpaces.

## 6. Przestrzenie krotek (Linda)
**Model**: globalna, asocjacyjna **przestrzeń krotek**; operacje `out(t)` (wstaw), `in(template)` (pobierz i usuń, blokująco), `rd(template)` (odczytaj), `eval` (utwórz aktywną krotkę).

**Istota**: **pełne rozprzężenie w czasie i przestrzeni** (generatywna komunikacja).

**Narzędzia**: Linda, JavaSpaces, GigaSpaces.

## 7. Komunikacja strumieniowa
Ciągły przepływ danych zależnych od czasu (audio, wideo). Wymaga gwarancji QoS: przepustowość, opóźnienie, jitter, synchronizacja strumieni.

## 8. Podejście językowe – spotkania (_rendezvous_, Ada)
Współbieżność wbudowana w język: **zadania** (`task`) komunikują się przez wywołania **wejść** (`entry`) przyjmowane instrukcją `accept`. Oba zadania czekają na siebie (synchroniczna komunikacja z odpowiedzią), a `select` pozwala na niedeterministyczny wybór spośród wejść ze strażnikami. Szczegóły: [[09 Wielozadaniowość i synchronizacja zadań i wątków]].

---
## Charakterystyka porównawcza
| Kryterium | Komunikaty | MOM / pub-sub | RPC | Obiekty zdalne | DSM | Przestrzeń krotek |
|---|---|---|---|---|---|---|
| Abstrakcja | niska | średnia | wysoka (procedura) | wysoka (obiekt) | najwyższa (pamięć) | wysoka |
| Przezroczystość dostępu | brak | częściowa | tak | tak | pełna | częściowa |
| Powiązanie w przestrzeni | tak (adres) | nie (broker / temat) | tak (serwer) | tak (referencja) | nie | nie |
| Powiązanie w czasie | tak | nie (trwałość) | tak | tak | tak | nie |
| Typowa synchroniczność | obie | asynchroniczna | synchroniczna | synchroniczna | – | blokujące `in` |
| Heterogeniczność | ręczne formaty | formaty wiadomości | IDL + marshalling | IDL / Java | trudna | średnia |
| Semantyka błędów | jawna w aplikacji | gwarancje brokera | maybe / at-least / at-most once | jak RPC + wyjątki | ukryte (trudne) | ukryte |
| Wydajność | najwyższa | średnia | średnia | średnia/niska | niska (thrashing) | średnia |
| Przykłady | gniazda, MPI | Kafka, JMS, ZeroMQ | ONC RPC, gRPC | Java RMI, CORBA | TreadMarks | JavaSpaces |

**Wnioski**:
- im wyższa przezroczystość, tym trudniej ukryć awarie i opóźnienia (RPC „wygląda lokalnie”, ale nie jest),
- luźne powiązanie (MOM, krotki) sprzyja skalowalności i odporności, utrudnia śledzenie przepływu,
- podejścia wysokopoziomowe zbudowane są zwykle na przekazywaniu komunikatów.

## Zobacz też
- [[Narzędzia Przetwarzania Rozproszonego/Przekazywanie wiadomości (komunikatów)]]
- [[Narzędzia Przetwarzania Rozproszonego/Synchroniczność komunikacji]]
- [[07 Aspekty projektowe realizacji systemów rozproszonych]]
