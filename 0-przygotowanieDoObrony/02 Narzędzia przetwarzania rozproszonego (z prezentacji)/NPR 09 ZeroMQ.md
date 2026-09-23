---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 8
source: "slajdy–merged.pdf"
slajdy: "10–15"
---
# NPR 09. ZeroMQ
---
> Pierwsza z dwóch omawianych realizacji [[NPR 08 MOM i systemy kolejkowania komunikatów|MOM]] — i przypadek skrajny, bo **pozbawiony brokera**. ZeroMQ nie ma zarządcy kolejek ze slajdu 6; kolejkowanie jest **wbudowane w gniazdo**. Wykład pokazuje własności biblioteki, schemat komunikacji w siedmiu krokach oraz API w języku C: kontekst, gniazdo, wiązanie i łączenie, wysyłanie i odbiór.

---
## Własności
<sub>slajdy–merged.pdf, slajd 10</sub>

- **Mechanizm komunikacji ze zintegrowanym kolejkowaniem komunikatów** — **brak brokera**.
- **Interfejs wzorowany na gniazdach BSD.**
- Interfejs dostępny między innymi dla języków **C, C++, C#, Java, Python, Ruby, Ada**.
- Zdefiniowane **schematy komunikacji**: **REQ-REP, PUB-SUB, PUSH-PULL**.
- **Brak struktury komunikatu** — komunikat jest **sekwencją bajtów**, których typ/struktura zdefiniowana jest **na poziomie aplikacji**.

Brak brokera ma konsekwencję, której slajdy nie wypowiadają, ale która wynika wprost z modelu MOM: skoro kolejka żyje **w procesie**, a nie w osobnym, trwałym pośredniku, to **znika razem z procesem**. ZeroMQ nie realizuje więc komunikacji nieustannej — daje wygodę i wydajność kosztem gwarancji, które w JMS są podstawą.

Ostatni punkt jest lustrzanym odbiciem [[NPR 02 Sun RPC i standard XDR|XDR]]: tam cały wysiłek szedł w kanoniczną reprezentację typów, tu problem przerzucono na aplikację.

---
## Schemat komunikacji
<sub>slajdy–merged.pdf, slajd 11</sub>

![[npr-zmq-s11-schemat-komunikacji.png]]
<sub>Siedem kroków po obu stronach; różnica jest tylko w kroku 3 (połączenie vs związanie) i 4 (wysłanie vs odbiór). slajdy–merged.pdf, slajd 11</sub>

| Krok | Proces 1 | Proces 2 |
|---|---|---|
| 1 | utworzenie kontekstu | utworzenie kontekstu |
| 2 | utworzenie gniazda | utworzenie gniazda |
| 3 | **połączenie** gniazda | **związanie** gniazda |
| 4 | **wysłanie** komunikatu | **odbiór** komunikatu |
| 5 | inne operacje komunikacyjne | inne operacje komunikacyjne |
| 6 | zamknięcie gniazda | zamknięcie gniazda |
| 7 | usunięcie kontekstu | usunięcie kontekstu |

Symetria schematu jest istotna: **żadna ze stron nie jest serwerem**. To, która woła `bind`, a która `connect`, jest decyzją topologiczną, a nie rolą w protokole — w ZeroMQ strona łącząca się może równie dobrze być nadawcą, jak i odbiorcą.

---
## Kontekst
<sub>slajdy–merged.pdf, slajd 12</sub>

- **Kontekst jest zbiorem gniazd.**
- Tworzenie kontekstu:

```c
void *context = zmq_ctx_new();
```

- **Zalecenie: 1 kontekst na proces.**
- Usuwanie kontekstu:

```c
zmq_ctx_destroy(context);
// lub
zmq_ctx_term(context);
```

Usuwanie kontekstu obejmuje:
- **zakończenie wszystkich operacji blokujących**,
- **oczekiwanie na zakończenie wysyłania** komunikatów,
- **oczekiwanie na zamknięcie wszystkich gniazd**.

Kontekst jest tym, czym w systemie z brokerem jest **połączenie z brokerem** — z tą różnicą, że mieści się w tym samym procesie. Trzy czynności przy usuwaniu pokazują, gdzie naprawdę leżą kolejki: `zmq_ctx_term` **blokuje**, dopóki nie opróżni buforów wyjściowych.

---
## Gniazda
<sub>slajdy–merged.pdf, slajd 13</sub>

```c
void *zmq_socket(void *context, int type);
```

| Para typów | Schemat |
|---|---|
| **REQ** i **REP** | **naprzemienne** wysyłanie i odbiór |
| **PUB** i **SUB** | **publikowanie do wielu subskrybentów** |
| **PUSH** i **PULL** | **przetwarzanie potokowe** |

Typ gniazda **jest protokołem**: REQ wymusza naprzemienność (po wysłaniu trzeba odebrać, zanim wyśle się ponownie), PUB rozsyła bez potwierdzeń, PUSH rozdziela komunikaty między odbiorców metodą karuzelową. Te trzy pary odwzorowują trzy różne wzorce z [[NPR 08 MOM i systemy kolejkowania komunikatów|NPR 08]]: REQ-REP to odpowiednik wywołania zdalnego, PUB-SUB to publish/subscribe, a PUSH-PULL to kolejka punkt-punkt z wieloma konsumentami.

### Wiązanie i łączenie
<sub>slajd 14</sub>

```c
// wiązanie gniazda
void *gniazdo = zmq_socket(…, …);
zmq_bind(gniazdo, "tcp://*:5556");

// łączenie gniazd
void *gniazdo = zmq_socket(…, …);
zmq_connect(gniazdo, "tcp://*:5556");
```

**Łączyć można tylko „kompatybilne" gniazda** — to znaczy gniazda z tej samej pary (REQ do REP, PUB do SUB, PUSH do PULL). Próba połączenia REQ z SUB jest błędem.

---
## Wysyłanie i odbiór
<sub>slajdy–merged.pdf, slajd 15</sub>

```c
// wysyłanie
int zmq_send   (void *socket, void *buf, size_t len, int flags);
int zmq_sendmsg(void *socket, zmq_msg_t *msg, int flags);

// odbiór
int zmq_recv   (void *socket, void *buf, size_t len, int flags);
int zmq_recvmsg(void *socket, zmq_msg_t *msg, int flags);
```

Warianty `*msg` operują na obiekcie komunikatu `zmq_msg_t` zamiast na surowym buforze — pozwala to bibliotece uniknąć kopiowania danych.

> [!note] Uzupełnienie spoza slajdów
> Dwie praktyczne uwagi z notatek z laboratorium: przy `zmq_send` dane **trzeba wcześniej umieścić w pamięci** (funkcja bierze wskaźnik i długość, nie kopiuje z gniazda); **filtrowanie ma znaczenie przy subskrybowaniu** — gniazdo SUB **domyślnie nie odbiera niczego**, dopóki nie ustawi się filtra tematu. <sub>[[Narzędzia Przetwarzania Rozproszonego/ZMQ]]</sub>

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 10–15
> - **`zmq_setsockopt` i opcje gniazda** — w tym `ZMQ_SUBSCRIBE`, bez którego gniazdo SUB nie odbiera nic. Slajdy pokazują gniazda PUB-SUB, nie wspominając, jak ustawić filtr.
> - **Transporty inne niż `tcp://`** — `inproc://`, `ipc://`, `pgm://`.
> - **Komunikaty wieloczęściowe** (_multipart_) i ich rola w routowaniu — podstawa działania REQ-REP.
> - **Zachowanie przy przepełnieniu** (_high water mark_) i przy braku odbiorcy — kluczowe, skoro kolejka jest w procesie.
> - **Wzorce złożone** (broker jako urządzenie: `zmq_proxy`, ROUTER/DEALER) — slajdy wymieniają tylko trzy pary podstawowe.
> - Notatka [[Narzędzia Przetwarzania Rozproszonego/ZMQ]] w vaultcie jest **stubem** — niniejsza notatka ją zastępuje.
