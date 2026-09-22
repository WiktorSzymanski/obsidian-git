---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 3
---
# 3. Modele spójności zorientowane na klienta (RYW, MR, MW, WFR)
---
> Modele **zorientowane na klienta** (_client-centric_, **gwarancje sesji** – Terry i in., system Bayou, 1994) nie gwarantują spójności globalnego stanu replik. Gwarantują natomiast, że **pojedynczy klient** w ramach swojej **sesji** zobaczy dane zgodne z własną historią interakcji z systemem, **nawet jeśli przełącza się między serwerami**.

Motywacja: systemy ze spójnością ostateczną i klienci mobilni. Klient łączy się z różnymi replikami, które mogą być nieaktualne, i bez gwarancji widziałby „cofające się w czasie” dane lub tracił własne zmiany.

## Założenia modelu
- Dane replikowane na wielu serwerach, replikacja **leniwa** (spójność ostateczna, np. anty-entropia).
- Każdy zapis ma globalnie unikalny identyfikator $WID$ (np. $\langle serwer, znacznik\ czasu\rangle$).
- $DB(S)$ – zbiór zapisów zastosowanych na serwerze $S$ (w pewnym porządku).
- Sesja klienta utrzymuje:
  - **write-set** $WS$ – identyfikatory zapisów wykonanych przez klienta,
  - **read-set** $RS$ – identyfikatory zapisów, które wpłynęły na wyniki odczytów klienta.
- $RelevantWrites(S, r)$ – zapisy w $DB(S)$, które wpływają na wynik odczytu $r$.

## Cztery gwarancje
### RYW – Read Your Writes (czytaj swoje zapisy)
> Jeśli klient wykonał zapis $w(x)$, to każdy późniejszy odczyt $r(x)$ tego klienta jest realizowany przez serwer, który **uwzględnia** $w(x)$.

Warunek: odczyt na $S$ dozwolony, gdy $WS \subseteq DB(S)$.
Naruszenie: zmiana hasła / edycja profilu, po odświeżeniu (inny serwer) widać starą wartość.

### MR – Monotonic Reads (monotoniczne odczyty)
> Jeśli klient odczytał $x$ w pewnym stanie, to kolejne odczyty $x$ zwracają stan **co najmniej tak samo aktualny** – nigdy starszy.

Warunek: odczyt na $S$ dozwolony, gdy $RS \subseteq DB(S)$. Po odczycie $RS := RS \cup RelevantWrites(S, r)$.
Naruszenie: skrzynka pocztowa – wiadomość widoczna, po przełączeniu na inny serwer znika.

### MW – Monotonic Writes (monotoniczne zapisy)
> Jeśli klient wykonał zapis $w_1$, a potem $w_2$, to na każdym serwerze $w_2$ jest wykonywany **po** $w_1$. Serwer może przyjąć zapis klienta tylko, jeśli zawiera wszystkie wcześniejsze zapisy tego klienta.

Warunek: zapis na $S$ dozwolony, gdy $WS \subseteq DB(S)$; dodatkowo $w_1$ poprzedza $w_2$ w $DB(S)$.
Naruszenie: aktualizacja dokumentu (wersja 2) zastosowana przed utworzeniem go (wersja 1); zmiana ceny zastosowana w złej kolejności.

### WFR – Writes Follow Reads (zapisy następują po odczytach)
> Jeśli klient wykonał odczyt $r$, a potem zapis $w$, to na każdym serwerze $w$ jest umieszczony **po wszystkich zapisach, które wpłynęły na** $r$.

Warunek: zapis na $S$ dozwolony, gdy $RS \subseteq DB(S)$; a na każdym serwerze $RelevantWrites(r)$ poprzedzają $w$.
Naruszenie: odpowiedź na post na forum pojawia się na serwerze, na którym oryginalnego postu jeszcze nie ma (lub przed nim).

## Podsumowanie
| Gwarancja | Sprawdzana przy | Warunek na serwerze | Aktualizacja sesji |
|---|---|---|---|
| RYW | odczycie | $WS \subseteq DB(S)$ | zapis: $WS \mathrel{+}= wid$ |
| MR | odczycie | $RS \subseteq DB(S)$ | odczyt: $RS \mathrel{+}= relevant$ |
| MW | zapisie | $WS \subseteq DB(S)$ | zapis: $WS \mathrel{+}= wid$ |
| WFR | zapisie | $RS \subseteq DB(S)$ | odczyt: $RS \mathrel{+}= relevant$ |

Gwarancje są niezależne i można je łączyć. Wszystkie cztery razem nie dają spójności sekwencyjnej (dotyczą jednej sesji, nie wielu klientów).

## Implementacja
- **Naiwnie**: klient przechowuje zbiory $WS$ i $RS$ (identyfikatory), serwer sprawdza zawieranie. Zbiory rosną nieograniczenie.
- **Wektory wersji** (Bayou): każdy serwer ma wektor $V_S[k]$ = największy znacznik zapisu z serwera $k$ zastosowany na $S$ (zapisy od $k$ stosowane w kolejności). Sesja reprezentuje $WS$ i $RS$ wektorami $V_{WS}$, $V_{RS}$; warunek $WS \subseteq DB(S)$ ⇔ $V_{WS} \leq V_S$ (dominacja po składowych).
- Gdy serwer nie spełnia warunku: klient **czeka**, wybiera **inny serwer**, albo serwer **pobiera brakujące zapisy** od innych replik.
- MW i WFR wymagają też, by serwery stosowały zapisy w porządku zgodnym z zależnościami (np. porządek przyczynowy / znaczniki Lamporta przy propagacji).

## Porównanie z modelami danocentrycznymi
| | data-centric | client-centric |
|---|---|---|
| Perspektywa | wszystkie procesy | pojedynczy klient/sesja |
| Klient | związany z „serwerem” (proces = replika) | mobilny, przełącza serwery |
| Koszt | wysoki (globalna synchronizacja) | niski (sprawdzenie wektora) |
| Przykłady | [[02 Danocentryczne modele spójności]] | Bayou, DynamoDB (read-after-write), MongoDB causal sessions |

## Zobacz też
- [[Algorytmy Rozproszone/Model Spójności#Klasyfikacja]]
- [[46 Ostateczna spójność - CRDT i typy chmurowe]]
